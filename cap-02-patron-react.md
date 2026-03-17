# Capítulo 2: El Patrón ReAct — Reason + Act

> **Nivel:** Sin conocimientos previos requeridos  
> **Prerequisito recomendado:** [Capítulo 1](./cap-01-que-es-claude-code.md)  
> **Objetivo:** Entender en profundidad el ciclo de razonamiento y acción que gobierna cada tarea de Claude Code.

---

## Tabla de Contenidos

- [2.1 El problema que ReAct resuelve](#21-el-problema-que-react-resuelve)
- [2.2 El origen académico del patrón](#22-el-origen-académico-del-patrón)
- [2.3 Los tres elementos: Thought, Action, Observation](#23-los-tres-elementos-thought-action-observation)
- [2.4 Traza completa de una tarea real](#24-traza-completa-de-una-tarea-real)
- [2.5 Cómo el modelo decide cuándo ha terminado](#25-cómo-el-modelo-decide-cuándo-ha-terminado)
- [2.6 ReAct vs. Chain-of-Thought: la diferencia clave](#26-react-vs-chain-of-thought-la-diferencia-clave)
- [2.7 El concepto de grounding: anclar el razonamiento en datos reales](#27-el-concepto-de-grounding-anclar-el-razonamiento-en-datos-reales)
- [2.8 Cuándo el patrón ReAct falla y por qué](#28-cuándo-el-patrón-react-falla-y-por-qué)
- [2.9 Implicaciones prácticas para el usuario](#29-implicaciones-prácticas-para-el-usuario)
- [2.10 Resumen y glosario del capítulo](#210-resumen-y-glosario-del-capítulo)

---

## 2.1 El problema que ReAct resuelve

Antes de explicar qué es ReAct, es importante entender el problema que intenta resolver.

### El problema de la información incompleta

Imagina que eres un detective al que le piden resolver un caso. Tienes dos opciones:

**Opción A — Detective de escritorio:** Alguien te da un resumen del caso y tú, sin salir de tu oficina, tienes que deducir quién lo hizo basándote solo en ese resumen. Si la persona que te dio el resumen omitió algo importante, tu deducción será incorrecta, y nunca lo sabrás.

**Opción B — Detective de campo:** Visitas la escena del crimen, entrevistas a testigos, solicitas registros, analiza evidencia física. Cada acción te da nueva información que guía la siguiente. Si descubres algo inesperado, ajustas tu hipótesis.

Los LLMs tradicionales eran como el detective de escritorio. Se les daba un prompt (el resumen del caso) y generaban una respuesta sin poder explorar más allá de lo que se les había dado.

### El resultado práctico del problema

```
PREGUNTA AL CHATBOT:
  "¿Hay algún bug en mi código de autenticación?"

PROBLEMA DEL CHATBOT:
  - No sabe qué código tienes
  - No puede leerlo
  - Inventará una respuesta genérica sobre bugs comunes
  - Si hay un bug específico de tu implementación, no lo encontrará

RESPUESTA TÍPICA (inexacta):
  "Los bugs más comunes en autenticación son: 1) No usar bcrypt
   para contraseñas, 2) No validar tokens correctamente, 3)..."
```

ReAct soluciona esto dándole al modelo la capacidad de **explorar** antes de responder.

---

## 2.2 El origen académico del patrón

### El paper de 2022

ReAct fue introducido en el paper **"ReAct: Synergizing Reasoning and Acting in Language Models"** (Yao et al., 2022), publicado en la conferencia ICLR 2023.

Los autores partieron de una observación simple: los LLMs son muy buenos razonando (como en Chain-of-Thought), y los agentes con herramientas son muy buenos actuando. ¿Por qué no combinar ambas capacidades?

La hipótesis: si el modelo puede **razonar sobre qué acción tomar**, ejecutar esa acción y luego **razonar sobre el resultado** antes de decidir la siguiente acción, el rendimiento debería mejorar significativamente.

### Los experimentos del paper

El paper probó ReAct en varias tareas:
- Responder preguntas de trivia que requerían buscar en Wikipedia.
- Navegar por sitios web para completar tareas (comprar productos, encontrar información).
- Razonar sobre secuencias de operaciones.

En todos los casos, ReAct superó tanto a los modelos de solo razonamiento (Chain-of-Thought) como a los agentes de solo acción (sin razonamiento explícito entre acciones).

### De paper académico a Claude Code

Claude Code es una implementación del patrón ReAct con algunas diferencias clave respecto al paper original:

- El paper usaba Wikipedia como fuente de información. Claude Code usa el sistema de archivos, la terminal y la web.
- El paper probaba en tareas de preguntas y respuestas. Claude Code trabaja en tareas de ingeniería de software.
- El paper demostraba el concepto. Claude Code es un producto que lo aplica en producción.

---

## 2.3 Los tres elementos: Thought, Action, Observation

El ciclo ReAct tiene exactamente tres elementos que se repiten en secuencia:

### Thought (Pensamiento)

El **Thought** es el razonamiento interno del modelo. Es donde Claude analiza la situación actual y decide qué hacer a continuación.

Características:
- **No es visible para el usuario** en el output normal (aunque puede activarse con flags de debug).
- Se basa en toda la información disponible en ese momento: la tarea original, el historial de acciones anteriores, y los resultados de esas acciones.
- Puede incluir hipótesis, planes, razonamientos sobre incertidumbre, y justificaciones de por qué se elige una herramienta sobre otra.

Ejemplo de Thought:
```
"El usuario quiere que migre la autenticación a bcrypt.
 Antes de hacer cualquier cambio, necesito ver el código actual.
 Voy a leer src/auth.py para entender qué hay implementado.
 También debería comprobar si hay tests que deba actualizar."
```

### Action (Acción)

La **Action** es la invocación de una herramienta. Claude "llama" a una herramienta especificando el nombre y los parámetros necesarios.

Ejemplos de acciones:
```
Read("src/auth.py")              → leer un archivo
Write("src/auth.py", contenido)  → escribir un archivo
Bash("pytest tests/ -v")         → ejecutar tests
WebSearch("bcrypt python docs")   → buscar en internet
Bash("grep -r 'hash_password' .") → buscar en el código
```

La acción es un mecanismo preciso: Claude debe especificar exactamente qué herramienta usar y con qué parámetros. No puede ser vago.

### Observation (Observación)

La **Observation** es el resultado de la acción. Puede ser:
- El contenido de un archivo leído.
- El output de un comando ejecutado (stdout + stderr).
- Los resultados de una búsqueda web.
- Un mensaje de error si algo salió mal.

La Observation se incorpora al historial de la conversación y está disponible para el siguiente Thought. Es lo que "ancla" el razonamiento en la realidad del sistema.

### El ciclo completo

```
┌────────────────────────────────────────────────────────────────┐
│                     CICLO REACT                                │
│                                                                │
│   ┌──────────┐    ┌──────────┐    ┌──────────────┐            │
│   │          │    │          │    │              │            │
│   │ THOUGHT  │───►│  ACTION  │───►│ OBSERVATION  │            │
│   │          │    │          │    │              │            │
│   │ "Necesito│    │ Read(    │    │ "def auth(): │            │
│   │  leer el │    │  auth.py)│    │   return md5 │            │
│   │  archivo"│    │          │    │   ..."       │            │
│   └──────────┘    └──────────┘    └──────┬───────┘            │
│        ▲                                 │                     │
│        └─────────────────────────────────┘                     │
│                  (la observación informa                       │
│                   el siguiente thought)                        │
│                                                                │
│   Este ciclo se repite hasta que:                              │
│   - La tarea está completa, O                                  │
│   - No hay más acciones útiles que tomar                       │
└────────────────────────────────────────────────────────────────┘
```

---

## 2.4 Traza completa de una tarea real

Para entender el patrón en profundidad, vamos a seguir una tarea real de principio a fin. La tarea es:

> *"En src/auth.py hay un bug con el hash de passwords: está usando MD5, que es inseguro. Investígalo y corrígelo correctamente."*

### Iteración 1: Exploración inicial

```
═══════════════════════════════════════════════════════════════
ITERACIÓN 1
═══════════════════════════════════════════════════════════════

THOUGHT:
  "El usuario dice que hay un bug con MD5 en src/auth.py.
   Para entender el problema completamente, primero necesito
   leer el archivo y ver el código actual. No puedo hacer
   nada sin ver qué hay implementado."

ACTION:
  Herramienta: Read
  Parámetros: { "path": "src/auth.py" }

OBSERVATION:
  """
  import hashlib
  
  def hash_password(password: str) -> str:
      return hashlib.md5(password.encode()).hexdigest()
  
  def verify_password(password: str, hashed: str) -> bool:
      return hash_password(password) == hashed
  
  def create_user(username: str, password: str) -> dict:
      return {
          "username": username,
          "password_hash": hash_password(password),
          "created_at": datetime.now().isoformat()
      }
  """
```

### Iteración 2: Exploración de tests

```
═══════════════════════════════════════════════════════════════
ITERACIÓN 2
═══════════════════════════════════════════════════════════════

THOUGHT:
  "Confirmado: hash_password usa MD5 (hashlib.md5).
   MD5 es criptográficamente roto — no debe usarse para
   contraseñas porque:
   - Es vulnerable a ataques rainbow table
   - Es muy rápido (malo para contraseñas: permite fuerza bruta)
   - No usa salt por defecto
   
   La solución correcta es bcrypt, que:
   - Es lento por diseño (dificulta fuerza bruta)
   - Incluye salt automáticamente
   - Es el estándar de la industria para contraseñas
   
   Antes de cambiar el código, necesito ver si hay tests
   que podrían verse afectados por el cambio."

ACTION:
  Herramienta: Bash
  Parámetros: { "command": "find . -name 'test_*.py' -o -name '*_test.py' | xargs grep -l 'hash_password\\|verify_password' 2>/dev/null" }

OBSERVATION:
  "tests/test_auth.py"
```

### Iteración 3: Leer los tests existentes

```
═══════════════════════════════════════════════════════════════
ITERACIÓN 3
═══════════════════════════════════════════════════════════════

THOUGHT:
  "Hay tests en tests/test_auth.py que prueban las funciones
   de hash. Necesito leerlos para entender qué asumen sobre
   el comportamiento actual antes de hacer cambios."

ACTION:
  Herramienta: Read
  Parámetros: { "path": "tests/test_auth.py" }

OBSERVATION:
  """
  import pytest
  from src.auth import hash_password, verify_password
  
  def test_hash_password_returns_md5():
      result = hash_password('test123')
      assert result == 'cc03e747a6afbbcbf8be7668acfebee5'  # MD5 de 'test123'
      assert len(result) == 32  # MD5 siempre tiene 32 caracteres hex
  
  def test_verify_password_correct():
      hashed = hash_password('mysecret')
      assert verify_password('mysecret', hashed) == True
  
  def test_verify_password_wrong():
      hashed = hash_password('mysecret')
      assert verify_password('wrongpassword', hashed) == False
  """
```

### Iteración 4: Verificar dependencias

```
═══════════════════════════════════════════════════════════════
ITERACIÓN 4
═══════════════════════════════════════════════════════════════

THOUGHT:
  "Los tests asumen que:
   1. hash_password retorna exactamente el hash MD5 de 'test123'
   2. La longitud es 32 caracteres
   
   Cuando cambie a bcrypt, estos tests fallarán porque:
   - bcrypt genera hashes de 60 caracteres (no 32)
   - bcrypt incluye salt, así que el mismo input da diferente output
     en cada llamada (no se puede comparar con un valor fijo)
   
   Necesito:
   1. Añadir bcrypt a requirements.txt
   2. Actualizar hash_password y verify_password
   3. Actualizar los tests para que no dependan del formato MD5
   
   Primero verifico que bcrypt no esté ya instalado."

ACTION:
  Herramienta: Bash
  Parámetros: { "command": "cat requirements.txt" }

OBSERVATION:
  """
  flask==3.0.0
  sqlalchemy==2.0.23
  pytest==7.4.3
  """
  (bcrypt no está listado)
```

### Iteración 5: Añadir bcrypt a requirements.txt

```
═══════════════════════════════════════════════════════════════
ITERACIÓN 5
═══════════════════════════════════════════════════════════════

THOUGHT:
  "bcrypt no está en requirements.txt. Necesito añadirlo.
   Usaré bcrypt==4.1.2, que es la versión estable más reciente."

ACTION:
  Herramienta: Write
  Parámetros: {
    "path": "requirements.txt",
    "content": "flask==3.0.0\nsqlalchemy==2.0.23\npytest==7.4.3\nbcrypt==4.1.2\n"
  }

OBSERVATION:
  "Archivo escrito correctamente."
```

### Iteración 6: Instalar la dependencia

```
═══════════════════════════════════════════════════════════════
ITERACIÓN 6
═══════════════════════════════════════════════════════════════

THOUGHT:
  "He añadido bcrypt a requirements.txt. Ahora necesito
   instalarlo para poder usarlo y para que los tests funcionen."

ACTION:
  Herramienta: Bash
  Parámetros: { "command": "pip install bcrypt==4.1.2" }

OBSERVATION:
  """
  Successfully installed bcrypt-4.1.2
  """
```

### Iteración 7: Actualizar el código de auth.py

```
═══════════════════════════════════════════════════════════════
ITERACIÓN 7
═══════════════════════════════════════════════════════════════

THOUGHT:
  "Con bcrypt instalado, puedo actualizar src/auth.py.
   Los cambios necesarios:
   - Eliminar el import de hashlib
   - Importar bcrypt
   - hash_password: usar bcrypt.hashpw() con salt automático
   - verify_password: usar bcrypt.checkpw() para comparar
   
   IMPORTANTE: con bcrypt, hash_password retorna bytes, no string.
   Debo decidir si retornar bytes o string. Retornaré string
   decodificando con .decode('utf-8') para compatibilidad."

ACTION:
  Herramienta: Write
  Parámetros: {
    "path": "src/auth.py",
    "content": "import bcrypt\nfrom datetime import datetime\n\n\ndef hash_password(password: str) -> str:\n    \"\"\"Genera un hash seguro de la contraseña usando bcrypt.\n    \n    bcrypt incluye salt automáticamente y es lento por diseño,\n    lo que dificulta ataques de fuerza bruta.\n    \n    Returns:\n        str: El hash de la contraseña (formato bcrypt, 60 caracteres)\n    \"\"\"\n    salt = bcrypt.gensalt()\n    hashed = bcrypt.hashpw(password.encode('utf-8'), salt)\n    return hashed.decode('utf-8')\n\n\ndef verify_password(password: str, hashed: str) -> bool:\n    \"\"\"Verifica si una contraseña coincide con su hash bcrypt.\n    \n    Returns:\n        bool: True si la contraseña es correcta, False en caso contrario\n    \"\"\"\n    return bcrypt.checkpw(\n        password.encode('utf-8'),\n        hashed.encode('utf-8')\n    )\n\n\ndef create_user(username: str, password: str) -> dict:\n    return {\n        'username': username,\n        'password_hash': hash_password(password),\n        'created_at': datetime.now().isoformat()\n    }\n"
  }

OBSERVATION:
  "Archivo escrito correctamente."
```

### Iteración 8: Actualizar los tests

```
═══════════════════════════════════════════════════════════════
ITERACIÓN 8
═══════════════════════════════════════════════════════════════

THOUGHT:
  "El código está actualizado. Ahora debo actualizar los tests.
   
   El test test_hash_password_returns_md5 es incorrecto por dos razones:
   1. Ya no usamos MD5
   2. bcrypt produce hashes diferentes cada vez (por el salt)
   
   Los tests correctos para bcrypt deben verificar:
   1. Que el hash tiene el formato correcto de bcrypt (empieza con '$2b$')
   2. Que verify_password funciona correctamente (estos tests ya son válidos)
   3. Que dos llamadas a hash_password con el mismo input dan resultados
      diferentes (confirmar que el salt es aleatorio)"

ACTION:
  Herramienta: Write
  Parámetros: {
    "path": "tests/test_auth.py",
    "content": "import pytest\nfrom src.auth import hash_password, verify_password\n\n\ndef test_hash_password_uses_bcrypt_format():\n    \"\"\"El hash debe tener el formato de bcrypt: empieza con $2b$\"\"\"\n    result = hash_password('test123')\n    assert result.startswith('$2b$'), f'Expected bcrypt format, got: {result[:10]}'\n    assert len(result) == 60, f'Expected 60 chars, got: {len(result)}'\n\n\ndef test_hash_password_generates_unique_hashes():\n    \"\"\"bcrypt debe generar hashes diferentes por el salt aleatorio\"\"\"\n    hash1 = hash_password('test123')\n    hash2 = hash_password('test123')\n    assert hash1 != hash2, 'bcrypt should generate unique hashes due to random salt'\n\n\ndef test_verify_password_correct():\n    \"\"\"verify_password debe retornar True con la contraseña correcta\"\"\"\n    hashed = hash_password('mysecret')\n    assert verify_password('mysecret', hashed) == True\n\n\ndef test_verify_password_wrong():\n    \"\"\"verify_password debe retornar False con contraseña incorrecta\"\"\"\n    hashed = hash_password('mysecret')\n    assert verify_password('wrongpassword', hashed) == False\n\n\ndef test_verify_password_empty():\n    \"\"\"verify_password debe manejar contraseña vacía correctamente\"\"\"\n    hashed = hash_password('')\n    assert verify_password('', hashed) == True\n    assert verify_password('notempty', hashed) == False\n"
  }

OBSERVATION:
  "Archivo escrito correctamente."
```

### Iteración 9: Verificar con los tests

```
═══════════════════════════════════════════════════════════════
ITERACIÓN 9 (VERIFICACIÓN FINAL)
═══════════════════════════════════════════════════════════════

THOUGHT:
  "He actualizado el código y los tests. Ahora ejecuto los tests
   para confirmar que todo funciona correctamente."

ACTION:
  Herramienta: Bash
  Parámetros: { "command": "pytest tests/test_auth.py -v" }

OBSERVATION:
  """
  ========================= test session starts ==========================
  collected 5 items
  
  tests/test_auth.py::test_hash_password_uses_bcrypt_format PASSED  [ 20%]
  tests/test_auth.py::test_hash_password_generates_unique_hashes PASSED  [ 40%]
  tests/test_auth.py::test_verify_password_correct PASSED  [ 60%]
  tests/test_auth.py::test_verify_password_wrong PASSED  [ 80%]
  tests/test_auth.py::test_verify_password_empty PASSED  [100%]
  
  ========================== 5 passed in 0.34s ===========================
  """

THOUGHT:
  "Todos los tests pasan. La tarea está completa. Puedo resumir
   los cambios realizados para el usuario."

→ FIN DEL CICLO REACT
→ Claude responde al usuario con el resumen de los cambios.
```

### Lo que este ejemplo demuestra

Observa cuántas iteraciones fueron necesarias para una tarea aparentemente simple: **9 ciclos Thought → Action → Observation**. Cada iteración descubrió algo nuevo que informó la siguiente acción:

- Iteración 1: Ver el código actual (necesario para saber qué cambiar).
- Iteración 2: Encontrar los tests (necesario para no romperlos).
- Iteración 3: Leer los tests (necesario para saber qué actualizar).
- Iteración 4: Verificar dependencias (necesario antes de instalar).
- Iteraciones 5-6: Añadir e instalar bcrypt.
- Iteraciones 7-8: Actualizar código y tests.
- Iteración 9: Verificar que todo funciona.

Un chatbot podría haber dado el código actualizado de `auth.py`, pero:
- No habría actualizado los tests (no los conocía).
- No habría añadido bcrypt a `requirements.txt`.
- No habría instalado la dependencia.
- No habría verificado que los tests pasan.

---

## 2.5 Cómo el modelo decide cuándo ha terminado

Una pregunta natural es: ¿cómo sabe Claude cuándo parar de ejecutar acciones?

### El mecanismo técnico: stop_reason

Cada respuesta de la API de Anthropic incluye un campo `stop_reason` que indica por qué el modelo dejó de generar texto:

```json
{
  "content": [...],
  "stop_reason": "tool_use"  // quiere usar una herramienta
}

{
  "content": [...],
  "stop_reason": "end_turn"  // ha terminado
}
```

Cuando `stop_reason` es `"end_turn"`, Claude Code sabe que el modelo considera la tarea completa y muestra la respuesta al usuario.

### Los criterios de finalización del modelo

El modelo aprende (durante el entrenamiento) a reconocer cuándo una tarea está completa. Los indicadores son:

1. **La tarea solicitada se ha completado:** Si el usuario pidió "corrige el bug", el ciclo termina cuando el bug está corregido y los tests pasan.

2. **No hay más acciones útiles:** Si después de explorar, el modelo no puede determinar qué acción tomarte llevaría más cerca del objetivo, responde con lo que encontró.

3. **Se alcanza un bloqueador:** Si hay un error que impide continuar (por ejemplo, un archivo no existe, o hay un permiso denegado), el modelo lo comunica al usuario.

4. **La información obtenida es suficiente para responder:** Para preguntas de comprensión (no de modificación), cuando el modelo tiene suficiente información para dar una respuesta completa.

### El límite de iteraciones

Claude Code tiene un límite interno de iteraciones para evitar bucles infinitos. Si una tarea requiere más iteraciones de las esperadas, Claude puede:
- Simplificar el enfoque.
- Pedir al usuario que divida la tarea.
- Informar de lo que pudo completar y lo que no.

---

## 2.6 ReAct vs. Chain-of-Thought: la diferencia clave

### Qué es Chain-of-Thought (CoT)

**Chain-of-Thought** (Cadena de Pensamiento) es una técnica anterior a ReAct donde el modelo genera pasos intermedios de razonamiento antes de dar la respuesta final. Pero estos pasos son **solo texto**: el modelo razona en su cabeza sin tomar acciones reales.

Ejemplo de Chain-of-Thought para el mismo problema:

```
PREGUNTA:
  "Hay un bug en mi código de hash de passwords, ¿cómo lo corrijo?"

CHAIN-OF-THOUGHT (solo razonamiento):
  "Vamos a pensar paso a paso:
   1. Los bugs comunes en hash de passwords incluyen usar MD5 o SHA1.
   2. La solución moderna es usar bcrypt o Argon2.
   3. En Python, se puede usar la librería bcrypt.
   4. El código sería:
      import bcrypt
      def hash_password(p): return bcrypt.hashpw(p.encode(), bcrypt.gensalt())
   
   Por lo tanto, la solución es reemplazar el hash actual por bcrypt."

RESPUESTA:
  "Usa bcrypt así: [código genérico]"
```

### La limitación de Chain-of-Thought

Chain-of-Thought razona **sobre lo que cree que es el código**, no sobre el código real. El modelo puede:
- Asumir que el código usa MD5 cuando en realidad usa SHA256.
- No saber que ya hay tests que habría que actualizar.
- No saber si bcrypt ya está instalado o no.
- Dar código que asume una estructura del proyecto que no existe.

### La ventaja de ReAct

ReAct **observa la realidad antes de actuar**:

```
COMPARATIVA:

CHAIN-OF-THOUGHT:
  Problema → [razonar] → Solución hipotética
  
  Trabaja con suposiciones sobre el código.
  La solución puede ser incorrecta para el contexto específico.
  No puede verificar que la solución funciona.

REACT:
  Problema → [razonar] → Leer código real → [razonar] → 
  Leer tests reales → [razonar] → Implementar → 
  Ejecutar tests → [razonar] → ¿Tests pasan? → Reportar

  Trabaja con el código y el contexto reales.
  La solución está adaptada al proyecto específico.
  Verifica que la solución funciona.
```

### La analogía del médico

- **Chain-of-Thought:** Un médico que escucha los síntomas y da un diagnóstico basado solo en lo que el paciente describe. Puede ser correcto, pero trabaja con información de segunda mano.
- **ReAct:** Un médico que escucha los síntomas, pide análisis de sangre, solicita una radiografía, y luego diagnostica basándose en datos reales. Puede tardar más, pero el diagnóstico es más preciso.

---

## 2.7 El concepto de grounding: anclar el razonamiento en datos reales

### Qué es grounding

**Grounding** (anclaje) es el proceso de conectar el razonamiento del LLM con información real del mundo, en lugar de dejar que el modelo trabaje solo con lo que "sabe" de su entrenamiento.

Los LLMs tienen un conocimiento del mundo que puede estar:
- **Desactualizado:** El entrenamiento tiene una fecha de corte. Claude no sabe qué versión de una librería es la más reciente hoy.
- **Genérico:** El modelo sabe cómo se usa bcrypt en general, no cómo está integrado en tu proyecto específico.
- **Potencialmente incorrecto:** Especialmente para información muy específica o reciente, los LLMs pueden "alucinar" (generar información falsa con apariencia de verdadera).

### Cómo ReAct provee grounding

Cada vez que Claude Code lee un archivo real, ejecuta un comando real o busca en internet, está "anclando" su razonamiento en datos reales. Esto elimina la dependencia del conocimiento interno del modelo para los aspectos específicos del proyecto.

```
SIN GROUNDING (chatbot):
  "Tu proyecto probablemente usa Flask y SQLAlchemy, así que..."
  → Suposición. Puede ser incorrecta.

CON GROUNDING (ReAct):
  Lee requirements.txt → Confirma que usa Flask y SQLAlchemy
  Lee la estructura del proyecto → Sabe exactamente qué hay
  Lee el código actual → Entiende la implementación real
  → Todo basado en datos reales, no suposiciones.
```

### Grounding y la fiabilidad del output

El grounding es la razón por la que Claude Code produce resultados más fiables que los chatbots para tareas de programación:

- **No inventa dependencias:** Lee `requirements.txt` y sabe exactamente qué está disponible.
- **No inventa estructura de archivos:** Explora el proyecto antes de hacer referencias a rutas.
- **No asume convenciones:** Lee `CLAUDE.md` y el código existente para adaptarse al estilo del proyecto.
- **No reporta éxito sin verificar:** Ejecuta los tests para confirmar que los cambios funcionan.

---

## 2.8 Cuándo el patrón ReAct falla y por qué

ReAct es poderoso, pero no es infalible. Hay situaciones donde el patrón puede producir resultados subóptimos.

### Problema 1: El bucle infinito conceptual

Puede ocurrir que Claude entre en un ciclo donde:
- Intenta acción A → falla
- Intenta acción B (alternativa) → falla
- Vuelve a intentar acción A → falla

**Causa:** El modelo no reconoce que ya ha intentado algo que no funcionó.  
**Síntoma:** Claude hace lo mismo repetidamente sin progreso.  
**Solución:** Interrumpir (Ctrl+C) y re-enfocar la tarea con más contexto específico.

### Problema 2: La explosión del contexto

En proyectos grandes, el ciclo ReAct puede generar muchísimas iteraciones, cada una añadiendo contenido al contexto (archivos leídos, outputs de comandos). Eventualmente, el contexto puede llenarse.

**Causa:** Cada Observation añade tokens al contexto.  
**Síntoma:** Claude empieza a "olvidar" información del principio de la sesión, las respuestas se degradan en calidad.  
**Solución:** Usar `/compact` para comprimir el historial (Capítulo 5), o dividir la tarea en subtareas más pequeñas.

### Problema 3: La cadena de errores

Si una acción temprana produce un error que Claude no detecta como error, las acciones siguientes se basarán en una premisa incorrecta.

**Ejemplo:**
```
Iteración 1: Lee auth.py → Observación: (archivo vacío por algún bug)
Iteración 2: "El archivo está vacío, crearé la implementación completa"
→ Sobreescribe el archivo con código que el usuario no quería.
```

**Causa:** El modelo interpretó incorrectamente la Observation.  
**Solución:** Revisar los cambios antes de hacer `git commit`. Usar `git diff` para ver exactamente qué cambió.

### Problema 4: Scope demasiado amplio

Si el prompt inicial es demasiado vago, Claude puede explorar en direcciones inesperadas.

**Ejemplo de prompt problemático:** "Mejora el código"  
**Resultado posible:** Claude lee todos los archivos, hace cambios en 20 sitios distintos, algunos deseados y otros no.

**Solución:** Ser específico en el scope. "Mejora la función hash_password en src/auth.py" produce resultados mucho más predecibles que "mejora el código".

---

## 2.9 Implicaciones prácticas para el usuario

Entender el patrón ReAct tiene consecuencias directas en cómo usar Claude Code de forma efectiva.

### Implicación 1: Sé específico en los objetivos, no en los pasos

Con un chatbot, a veces necesitas describir los pasos exactos porque el modelo no puede explorar. Con ReAct, Claude puede descubrir los pasos necesarios por sí mismo.

```
MENOS EFECTIVO (describir pasos):
  "Lee src/auth.py, luego mira si hay tests en tests/test_auth.py,
   luego cambia la función hash_password para usar bcrypt..."

MÁS EFECTIVO (describir el objetivo):
  "La función de hash de passwords en src/auth.py usa MD5, que es
   inseguro. Migra a bcrypt, actualiza los tests y asegúrate de
   que todo sigue funcionando."
```

Claude descubrirá los pasos necesarios a través del ciclo ReAct.

### Implicación 2: Espera múltiples acciones para tareas complejas

Una tarea compleja puede requerir 10-20 iteraciones. Es normal ver a Claude leer varios archivos, ejecutar comandos, hacer cambios y verificar. No es señal de que algo va mal; es el patrón funcionando correctamente.

### Implicación 3: Los errores en la ejecución son información

Si Claude ejecuta un test y falla, eso no es un problema del agente: es información valiosa que el agente puede usar para corregir su enfoque. El ciclo ReAct está diseñado para manejar errores como parte del proceso.

### Implicación 4: Puedes interrumpir y redirigir

Si ves que Claude está tomando una dirección incorrecta (por ejemplo, está modificando archivos que no querías tocar), puedes interrumpir con Ctrl+C y reorientar con instrucciones más específicas.

---

## 2.10 Resumen y glosario del capítulo

### Resumen

1. **ReAct** (Reasoning + Acting) es el patrón que gobierna cómo Claude Code aborda cada tarea.
2. El ciclo tiene tres elementos: **Thought** (razonamiento), **Action** (invocar herramienta), **Observation** (resultado).
3. El ciclo se **repite** hasta que la tarea está completa o no hay más acciones útiles.
4. ReAct supera a **Chain-of-Thought** porque trabaja con datos reales, no con suposiciones.
5. El **grounding** (anclaje en datos reales) es lo que hace que los outputs de ReAct sean más fiables.
6. El patrón puede fallar en bucles infinitos, explosión de contexto, o scopes demasiado amplios.
7. La implicación práctica: describe **qué quieres lograr**, no los pasos; Claude descubre los pasos con ReAct.

### Glosario

| Término | Definición |
|---------|------------|
| **ReAct** | Reasoning + Acting. El patrón de ciclos Thought→Action→Observation. |
| **Thought** | El razonamiento interno de Claude entre acciones (no siempre visible). |
| **Action** | La invocación de una herramienta con parámetros específicos. |
| **Observation** | El resultado de ejecutar una herramienta: contenido de archivo, output de comando, etc. |
| **Chain-of-Thought** | Técnica donde el modelo razona en pasos intermedios pero sin tomar acciones reales. |
| **Grounding** | El proceso de anclar el razonamiento del LLM en datos reales del sistema. |
| **stop_reason** | Campo en la respuesta de la API que indica si Claude quiere usar una herramienta (`tool_use`) o ha terminado (`end_turn`). |
| **Alucinación** | Cuando un LLM genera información falsa con apariencia de verdadera (mitigado por grounding). |

---

## Ver también

- **[Capítulo 1](./cap-01-que-es-claude-code.md):** Qué es Claude Code — conceptos fundamentales.
- **[Capítulo 3](./cap-03-arquitectura-componentes.md):** Arquitectura de Componentes — cómo se implementa el ciclo ReAct internamente.
- **[Capítulo 4](./cap-04-sistema-herramientas.md):** El Sistema de Herramientas — cada herramienta disponible para las Actions.
- **[Capítulo 5](./cap-05-ventana-contexto.md):** La Ventana de Contexto — cómo gestionar el crecimiento del contexto en ciclos largos.

---

> 📌 Siguiente capítulo: [Cap. 3 — Arquitectura de Componentes](./cap-03-arquitectura-componentes.md)

