# Capítulo 1: ¿Qué es Claude Code Internamente?

> **Nivel:** Absolutamente sin conocimientos previos — se parte de cero  
> **Tiempo de lectura estimado:** 45-60 minutos  
> **Objetivo del capítulo:** Comprender, desde los fundamentos más básicos, qué hace que Claude Code sea diferente de cualquier otra herramienta de IA, cómo funciona por dentro, y por qué esa diferencia cambia radicalmente lo que es posible hacer.

---

## Tabla de Contenidos

- [1.1 El punto de partida: qué es realmente un chatbot](#11-el-punto-de-partida-qué-es-realmente-un-chatbot)
- [1.2 Los límites fundamentales del chatbot](#12-los-límites-fundamentales-del-chatbot)
- [1.3 De chatbot a agente: qué cambia exactamente](#13-de-chatbot-a-agente-qué-cambia-exactamente)
- [1.4 El motor detrás de todo: qué es un LLM](#14-el-motor-detrás-de-todo-qué-es-un-llm)
- [1.5 Los tokens: la unidad atómica de todo](#15-los-tokens-la-unidad-atómica-de-todo)
- [1.6 Cómo fluye realmente una conversación](#16-cómo-fluye-realmente-una-conversación)
- [1.7 Las herramientas: el puente entre el lenguaje y el sistema real](#17-las-herramientas-el-puente-entre-el-lenguaje-y-el-sistema-real)
- [1.8 Acceso al sistema real: la diferencia que lo cambia todo](#18-acceso-al-sistema-real-la-diferencia-que-lo-cambia-todo)
- [1.9 Los modelos disponibles y cuándo usar cada uno](#19-los-modelos-disponibles-y-cuándo-usar-cada-uno)
- [1.10 Capacidades completas de Claude Code](#110-capacidades-completas-de-claude-code)
- [1.11 Limitaciones y lo que Claude Code no puede hacer](#111-limitaciones-y-lo-que-claude-code-no-puede-hacer)
- [1.12 Comparativa con otras herramientas del ecosistema](#112-comparativa-con-otras-herramientas-del-ecosistema)
- [1.13 Los primeros pasos: instalación y primera sesión](#113-los-primeros-pasos-instalación-y-primera-sesión)
- [1.14 Resumen completo y glosario del capítulo](#114-resumen-completo-y-glosario-del-capítulo)

---

## 1.1 El punto de partida: qué es realmente un chatbot

### Antes de hablar de Claude Code, hay que entender qué es un chatbot

Para entender qué hace especial a Claude Code, es necesario entender muy bien qué es — y qué no es — un chatbot de inteligencia artificial. Aunque muchas personas los usan a diario, pocos entienden realmente cómo funcionan por dentro. Sin ese entendimiento, la diferencia con Claude Code no queda del todo clara.

Un chatbot de IA, en su versión más básica, es un programa que recibe texto y produce texto. No importa si es ChatGPT, el Claude de la web, Gemini de Google, o cualquier otro: todos comparten el mismo patrón fundamental.

Imagina que envías una carta a alguien que vive muy lejos. Tú escribes tu mensaje, la carta viaja por correo, la persona la lee, escribe su respuesta, y la respuesta llega de vuelta. Lo que caracteriza a esta interacción es que **todo son palabras en papel**. La persona que responde no puede ver tu casa, no puede salir a comprobar nada, no puede tomar acciones en el mundo físico. Solo puede leer lo que tú escribiste y escribir su respuesta.

Un chatbot funciona exactamente igual, pero de forma instantánea y a través de internet. Tú escribes un mensaje en tu navegador, ese mensaje viaja por internet a los servidores de Anthropic, OpenAI, o Google, el modelo de IA lo lee y genera una respuesta de texto, y esa respuesta viaja de vuelta a tu pantalla.

### El proceso interno de un chatbot: paso a paso

Veamos el proceso con más detalle, porque cada paso importa:

```
═══════════════════════════════════════════════════════════════
CÓMO FUNCIONA UN CHATBOT PASO A PASO
═══════════════════════════════════════════════════════════════

PASO 1: Tú escribes un mensaje
        "¿Cómo puedo ordenar una lista en Python?"
              │
              ▼
PASO 2: El navegador empaqueta el mensaje en JSON
        {
          "model": "claude-3-5-sonnet",
          "messages": [
            {"role": "user", "content": "¿Cómo puedo ordenar..."}
          ]
        }
              │
              ▼ viaja por HTTPS (internet cifrado)
              │
PASO 3: El servidor de Anthropic recibe el mensaje
              │
              ▼
PASO 4: El modelo de IA procesa el texto
        (este proceso ocurre en grandes servidores con GPUs
         especializados en operaciones matemáticas masivas)
              │
              ▼
PASO 5: El modelo genera su respuesta, token por token
        "Puedes ordenar una lista en Python de varias formas:
         1. list.sort() para ordenar in-place..."
              │
              ▼ viaja por HTTPS de vuelta a ti
              │
PASO 6: Tu navegador muestra la respuesta
```

Este proceso es, conceptualmente, un sistema de **entrada de texto → salida de texto**. El modelo recibe texto, procesa ese texto usando complejos cálculos matemáticos, y produce más texto como respuesta.

### La clave que muchos no notan

Lo que muchas personas no se detienen a pensar es que en el proceso anterior, el modelo de IA **nunca sale de ese ciclo de texto**. No puede abrir archivos. No puede ejecutar código. No puede acceder a internet por su cuenta. No puede hacer nada más allá de recibir texto y producir texto.

Cuando le preguntas al chatbot web "¿cómo funciona este código?" y pegas el código en el chat, el chatbot solo puede leer esas palabras que escribiste. No puede ir a ver el archivo original. No puede ejecutar el código para ver qué produce. No puede verificar si su respuesta es correcta en el contexto de tu proyecto real.

Este es el límite fundamental. No es un defecto de diseño que se vaya a arreglar: es la naturaleza de un sistema de solo texto.

### El chatbot en la práctica del desarrollo de software

Cuando usas un chatbot para resolver un problema de programación real, el flujo concreto es así:

```
FLUJO REAL CON UN CHATBOT — Corrección de un bug
═══════════════════════════════════════════════════════════════

El proyecto tiene esta estructura:
  mi-proyecto/
  ├── src/
  │   ├── auth.py       ← tiene un bug de seguridad
  │   ├── models.py
  │   └── api.py
  └── tests/
      └── test_auth.py

PARA USAR EL CHATBOT NECESITAS:

1. Abrir auth.py en tu editor
2. Seleccionar todo el código (Ctrl+A)
3. Copiarlo (Ctrl+C)
4. Ir al chatbot, pegar el código
5. Escribir una descripción del problema
6. Recibir la solución propuesta
7. Copiar la solución
8. Volver a tu editor
9. Pegar la solución (reemplazando el código)
10. Guardar
11. Ir a la terminal y ejecutar los tests
12. Si los tests fallan, copiar el error, volver al chat, pegar el error...
13. Repetir desde el paso 6.

TIEMPO TOTAL: 30-90 minutos para un bug "simple"
TÚ HACES TODO EL TRABAJO MANUAL ENTRE LAS HERRAMIENTAS
═══════════════════════════════════════════════════════════════
```

El problema no es que el chatbot sea "malo". El problema es **estructural**: el chatbot no puede actuar. Solo puede hablar. Toda la acción la tienes que hacer tú.

---

## 1.2 Los límites fundamentales del chatbot

### El problema de la información de segunda mano

Cuando usas un chatbot para resolver un problema de programación, todo lo que el chatbot sabe sobre tu proyecto es lo que tú le cuentas. Eso significa que el chatbot trabaja con **información de segunda mano**.

Considera este escenario real:

Tienes un bug en `auth.py`. Abres el chatbot y describes el problema. El chatbot no puede ver:
- Cómo está estructurado el resto del proyecto
- Qué dependencias tienes instaladas
- Qué tests existen y cómo están escritos
- El historial reciente de cambios en git
- Cómo se llama la función problemática desde otros archivos

La información que proporcionas siempre es incompleta, porque **tú no sabes qué información es relevante hasta después de encontrar el bug**.

### Lo que el chatbot no puede detectar por sí solo

Hay toda una categoría de problemas que un chatbot no puede detectar porque requieren explorar el sistema:

```
PROBLEMAS INVISIBLES PARA EL CHATBOT:
═══════════════════════════════════════════════════════════════

❌ "¿Esta librería está instalada?"
   → El chatbot no puede ver tu requirements.txt ni tu entorno

❌ "¿Esta función se llama desde algún otro archivo?"
   → El chatbot no puede hacer grep en tu proyecto

❌ "¿Los tests pasan con este cambio?"
   → El chatbot no puede ejecutar pytest

❌ "¿Este bug es nuevo o existía en versiones anteriores?"
   → El chatbot no puede ver git log ni git diff

❌ "¿Hay algún archivo de configuración que afecte a esto?"
   → El chatbot no puede buscar en tu sistema de archivos

❌ "¿Hay código duplicado en otro archivo que también necesita cambiar?"
   → El chatbot no puede buscar patrones en el código base
```

### La imposibilidad de verificar

La tercera limitación fundamental es que **el chatbot no puede verificar que sus soluciones funcionen**. Da código que parece correcto basándose en su entrenamiento, pero no puede ejecutarlo para saberlo con certeza.

Esto crea el ciclo más frustrante del desarrollo asistido por IA:

```
El chatbot da una solución
        │
        ▼
Tú la implementas y ejecutas los tests
        │
        ▼
Los tests fallan (por una razón diferente al bug original)
        │
        ▼
Copias el error, lo pegas en el chat
        │
        ▼
El chatbot da otra solución (sin contexto de tu implementación real)
        │
        ▼
Vuelves a implementar y ejecutar...
        │
        ▼
[repite 3-7 veces]
```

Este ciclo de prueba y error con información incompleta es exactamente lo que Claude Code elimina.

---

## 1.3 De chatbot a agente: qué cambia exactamente

### La definición de "agente autónomo"

Un **agente autónomo** es un sistema de IA que puede percibir su entorno, tomar decisiones, y ejecutar acciones en ese entorno de forma independiente para alcanzar un objetivo dado.

La diferencia con un chatbot se resume en una sola palabra: **acción**.

Un chatbot puede razonar sobre qué hacer, pero no puede hacerlo.  
Un agente puede razonar sobre qué hacer **y hacerlo**.

Claude Code es un agente. Cuando le dices *"corrige el bug en auth.py"*, Claude no te explica cómo corregirlo. Claude **lo corrige**. Lee el archivo, analiza el código, identifica el problema, escribe la solución, ejecuta los tests, y te dice si todo funciona.

### La diferencia en la práctica: el mismo bug, dos herramientas

Veamos el mismo escenario de antes — corregir el bug de hashing — pero esta vez con Claude Code:

```
CON CLAUDE CODE — Corrección del mismo bug
═══════════════════════════════════════════════════════════════

TÚ ESCRIBES:
  > Hay un bug de seguridad en src/auth.py con el hash
    de contraseñas. Corrígelo correctamente.

CLAUDE TRABAJA SOLO:
  1. Lee src/auth.py                    ← ve que usa MD5 (inseguro)
  2. Busca tests relacionados           ← encuentra tests/test_auth.py
  3. Lee tests/test_auth.py             ← ve qué asumen los tests
  4. Lee requirements.txt               ← bcrypt no está instalado
  5. Actualiza auth.py con bcrypt       ← implementación correcta
  6. Actualiza tests/test_auth.py       ← tests adaptados a bcrypt
  7. Añade bcrypt a requirements.txt    ← documenta la dependencia
  8. Ejecuta: pip install bcrypt        ← instala la dependencia
  9. Ejecuta: pytest tests/test_auth.py ← verifica que todo funciona

CLAUDE TE RESPONDE:
  "He migrado auth.py de MD5 a bcrypt:
   - Actualicé hash_password() para usar bcrypt.hashpw()
   - Actualicé verify_password() para usar bcrypt.checkpw()
   - Añadí bcrypt==4.1.2 a requirements.txt
   - Actualicé los tests para verificar formato bcrypt
   Los 5 tests pasan correctamente."

TIEMPO TOTAL: 3-8 minutos
TÚ NO HICISTE NADA MANUAL
═══════════════════════════════════════════════════════════════
```

### El ciclo del agente en detalle

El ciclo de funcionamiento de un agente es más complejo que el del chatbot:

```
╔═══════════════════════════════════════════════════════════════╗
║                  EL CICLO DEL AGENTE                         ║
╚═══════════════════════════════════════════════════════════════╝

┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   Tú das una tarea:                                         │
│   "Corrige el bug de hashing en src/auth.py"                │
│                                    │                        │
│                                    ▼                        │
│   ┌─────────────────────────────────────────────────────┐   │
│   │  THOUGHT (Razonamiento interno)                     │   │
│   │  "Para corregir este bug necesito primero ver        │   │
│   │   el código actual. Voy a leer auth.py"             │   │
│   └──────────────────────┬──────────────────────────────┘   │
│                          │                                  │
│                          ▼                                  │
│   ┌─────────────────────────────────────────────────────┐   │
│   │  ACTION (Acción: usa una herramienta)               │   │
│   │  → Leer el archivo src/auth.py                     │   │
│   └──────────────────────┬──────────────────────────────┘   │
│                          │                                  │
│                          ▼                                  │
│   ┌─────────────────────────────────────────────────────┐   │
│   │  OBSERVATION (Resultado de la acción)               │   │
│   │  → Contenido del archivo: usa hashlib.md5()         │   │
│   └──────────────────────┬──────────────────────────────┘   │
│                          │                                  │
│                          ▼                                  │
│          ¿La tarea está completa?                           │
│            │                │                               │
│           NO               SÍ                               │
│            │                │                               │
│            ▼                ▼                               │
│        Volver a         Responder                           │
│        THOUGHT          al usuario                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Este ciclo se llama **ReAct** (Reasoning + Acting) y es el corazón del funcionamiento de Claude Code. Se desarrolla en profundidad en el **Capítulo 2**.

### La analogía del contratista vs el consultor telefónico

Para hacer la diferencia muy concreta, considera estas dos situaciones:

**Situación A: El consultor telefónico (chatbot)**

Tienes una gotera en el techo. Llamas a un consultor que te asesora por teléfono:

*Tú:* "Tengo una gotera en el techo, ¿qué hago?"  
*Consultor:* "Las goteras suelen ocurrir por tejas rotas o por problemas con el sellado. ¿Puedes describir dónde está la gotera exactamente? ¿Qué tipo de techo tienes?"  
*Tú:* "Está en la esquina noreste del salón, el techo es de tejas de cerámica"  
*Consultor:* "Probablemente hay una teja rota. Necesitarías quitar las tejas circundantes, aplicar impermeabilizante, y reemplazar las tejas dañadas."  
*Tú:* "Vale, lo intenté pero ahora llueve más dentro"  
*Consultor:* "Oh, quizás el problema es más profundo. ¿Me puedes describir qué viste cuando quitaste las tejas?"

Y así indefinidamente. El consultor puede estar muy bien informado y darte consejos muy buenos, pero **todo el trabajo manual lo haces tú**.

**Situación B: El contratista en persona (agente)**

Llamas al contratista, viene a tu casa, sube al techo, examina físicamente el estado, identifica tres tejas rotas y un problema de sellado en la cumbre, las repara, baja y te dice:

*Contratista:* "Había tres tejas rotas en la zona noreste y el sellado de la cumbre estaba deteriorado. He reemplazado las tejas y he aplicado impermeabilizante. El techo está reparado. He tomado fotos antes y después por si las necesitas para la garantía."

Claude Code es el contratista. No te dice qué hacer. **Lo hace**.

### Qué acciones puede ejecutar Claude Code

Cuando hablamos de "acciones", nos referimos concretamente a:

| Tipo de acción | Qué hace | Ejemplo |
|----------------|----------|---------|
| **Leer archivos** | Abre y lee el contenido de cualquier archivo de texto | Lee `src/auth.py` |
| **Escribir archivos** | Crea o modifica archivos en tu proyecto | Actualiza `requirements.txt` |
| **Ejecutar comandos** | Corre comandos en la terminal | Ejecuta `pytest tests/ -v` |
| **Buscar en código** | Busca patrones en archivos del proyecto | Encuentra todos los usos de `hash_password` |
| **Gestionar git** | Hace commits, crea ramas, revisa diffs | `git diff HEAD~1` |
| **Buscar en web** | Obtiene información actualizada de internet | Busca la última versión de bcrypt |
| **Crear subagentes** | Delega partes de la tarea a otros agentes paralelos | Un agente para tests, otro para la implementación |

---

## 1.4 El motor detrás de todo: qué es un LLM

### Por qué es importante entender los LLMs

Claude Code está impulsado por los modelos de lenguaje de Anthropic (LLMs). Para usar Claude Code de forma efectiva, es crucial entender qué son los LLMs, cómo "razonan", y cuáles son sus fortalezas y debilidades. Sin este entendimiento, es difícil saber por qué Claude toma ciertas decisiones o por qué a veces comete errores.

### Qué es un LLM: explicación desde cero

**LLM** son las siglas de **Large Language Model** (Modelo de Lenguaje Grande).

Para entender qué es un LLM, empezamos por una propiedad del lenguaje: las palabras no aparecen al azar. Si alguien dice "El gato persiguió al...", sabes con alta probabilidad que la siguiente palabra es "ratón", "pájaro", o algo relacionado con una presa. No va a ser "democracia" ni "refrigerador".

Esta propiedad del lenguaje — que las palabras y las frases siguen patrones — es la base de los LLMs. Un LLM es un programa entrenado para **predecir qué texto viene a continuación** dado un texto de entrada. Pero el entrenamiento se hace con cantidades astronómicas de texto, y el modelo aprende patrones tan complejos y sutiles que emerge una capacidad que nadie esperaba: la capacidad de razonar.

### El proceso de entrenamiento paso a paso

El entrenamiento de un LLM funciona así conceptualmente:

```
PROCESO DE ENTRENAMIENTO (simplificado)
═══════════════════════════════════════════════════════════════

DATOS DE ENTRENAMIENTO (enormes cantidades de texto):
  - Miles de millones de páginas de internet
  - Millones de libros digitalizados
  - Repositorios completos de GitHub (código abierto)
  - Artículos científicos, manuales técnicos, foros
  - Conversaciones, debates, tutoriales, documentación
  Total: cientos de miles de millones de palabras

PROCESO DE APRENDIZAJE:
  Para cada fragmento de texto:
  
  1. El modelo lee: "def calcular_suma(a, b):"
  2. Intenta predecir: ¿qué viene después?
  3. El modelo genera una predicción: "    pass"
  4. Compara con el texto real:          "    return a + b"
  5. Calcula el error entre predicción y realidad
  6. Ajusta internamente miles de millones de parámetros
     para reducir ese error
  
  Repite este proceso TRILLONES de veces.
  
RESULTADO:
  Un modelo con miles de millones de parámetros ajustados
  que ha "aprendido" los patrones del lenguaje, del código,
  del razonamiento lógico, y del conocimiento humano.
```

### Por qué el LLM puede "razonar"

Después de este entrenamiento masivo, el modelo desarrolla capacidades que sorprenden a sus propios creadores. No porque haya "magia", sino porque durante el entrenamiento vio millones de ejemplos de:

- Personas resolviendo problemas paso a paso
- Código correcto y sus errores corregidos
- Explicaciones técnicas en diferentes niveles de detalle
- Debates y argumentos con sus contra-argumentos
- Debugging sessions documentadas en foros

Al haber "visto" todos estos patrones, el modelo aprende a replicarlos. Cuando le pides que depure un bug, puede hacerlo porque aprendió los patrones de cómo se depuran bugs de millones de ejemplos reales.

### Las fortalezas del LLM de Claude

Los modelos Claude están especialmente bien entrenados para:

```
FORTALEZAS DEL MODELO CLAUDE:
═══════════════════════════════════════════════════════════════

COMPRENSIÓN DE CÓDIGO:
  ✓ Puede leer y entender código en 50+ lenguajes de programación
  ✓ Detecta bugs lógicos, de seguridad, y de rendimiento
  ✓ Entiende patrones de diseño y arquitecturas
  ✓ Puede razonar sobre el comportamiento de código complejo

SEGUIMIENTO DE INSTRUCCIONES:
  ✓ Puede seguir instrucciones con múltiples condiciones
  ✓ Recuerda restricciones dadas al principio de la conversación
  ✓ Adapta el nivel técnico de respuesta al interlocutor
  ✓ Puede trabajar con especificaciones ambiguas y pedir clarificación

RAZONAMIENTO TÉCNICO:
  ✓ Evalúa trade-offs entre diferentes soluciones
  ✓ Puede identificar consecuencias de cambios en el código
  ✓ Razona sobre rendimiento, escalabilidad, y mantenibilidad
  ✓ Puede seguir el hilo de razonamientos largos y complejos
```

### Las debilidades del LLM que debes conocer

Es igual de importante entender cuándo el LLM puede equivocarse:

**1. Alucinación (información falsa con apariencia de verdadera)**

Los LLMs a veces generan información falsa con completa confianza. Por ejemplo:
- Pueden inventar que una función de una librería existe cuando no existe.
- Pueden citar un estudio científico con título, autores, y año, que nunca se publicó.
- Pueden describir cómo configurar una herramienta de forma incorrecta.

Esto ocurre porque el modelo genera texto **plausible**, no texto **verificado**. La distinción es crucial. Claude Code mitiga esto al poder verificar: puede buscar si una función realmente existe ejecutando código, puede buscar en internet para verificar información.

**2. Conocimiento con fecha de corte**

El entrenamiento tiene una fecha de corte. Claude no sabe qué ocurrió después de esa fecha. No sabe cuál es la última versión de una librería hoy. Claude Code mitiga esto con la búsqueda web.

**3. Aritmética compleja**

Los LLMs son notoriamente malos con cálculos numéricos largos. Para eso, Claude Code puede escribir y ejecutar código Python que haga los cálculos con precisión exacta.

**4. Pérdida de coherencia en contextos extremadamente largos**

Aunque Claude puede manejar contextos de 200,000 tokens, en conversaciones muy largas puede "perder el hilo" de información mencionada hace mucho. Se desarrolla en el **Capítulo 5**.

### Los modelos disponibles y cuándo usar cada uno

Claude existe en varias versiones con diferentes características:

```
COMPARATIVA DE MODELOS CLAUDE
═══════════════════════════════════════════════════════════════

CAPACIDAD DE RAZONAMIENTO
    ▲ mayor capacidad / más lento / más caro
    │
    │  Claude Opus
    │  ─────────────────────────────────────────────────────
    │  Modelo de máxima capacidad. Para tareas que requieren
    │  razonamiento muy profundo y coherencia en análisis
    │  extendidos. Diseño de arquitecturas complejas,
    │  análisis de seguridad profundos, decisiones técnicas
    │  de alto impacto.
    │
    │  Claude Sonnet  ◄── EL MÁS USADO EN LA PRÁCTICA
    │  ─────────────────────────────────────────────────────
    │  El equilibrio óptimo entre capacidad, velocidad y
    │  costo. Maneja el 85-90% del trabajo diario de
    │  desarrollo: debugging, features, refactoring, tests,
    │  documentación, code review.
    │
    │  Claude Haiku
    │  ─────────────────────────────────────────────────────
    │  El más rápido y económico. Para tareas simples y
    │  repetitivas donde el costo importa: CI/CD de gran
    │  volumen, tareas de baja complejidad, pruebas rápidas.
    │
    ▼ menor capacidad / más rápido / más barato
```

| Modelo | Cuándo usarlo | Ejemplo de uso |
|--------|---------------|----------------|
| **Haiku** | Tareas simples, CI/CD, bajo costo | Añadir comentarios, renombrar variables |
| **Sonnet** | 90% del trabajo diario | Debugging, nuevas features, tests |
| **Opus** | Decisiones complejas de alto impacto | Diseño de arquitectura, análisis de seguridad |

---

## 1.5 Los tokens: la unidad atómica de todo

### Por qué necesitas entender los tokens

Los tokens son absolutamente fundamentales para entender cómo funciona Claude Code. Ignorarlos es como usar un coche sin entender que necesita gasolina. Los tokens determinan:

- **El costo:** pagas por cada token procesado
- **La velocidad:** más tokens = más tiempo de procesamiento
- **La memoria:** la ventana de contexto se mide en tokens
- **Las limitaciones:** cuando el contexto se llena, Claude empieza a "olvidar"

### Qué es exactamente un token

Los LLMs no procesan texto letra a letra ni palabra a palabra. Trabajan con unidades intermedias llamadas **tokens**, que son fragmentos de texto de longitud variable.

El sistema que convierte texto en tokens se llama **tokenizador**. El algoritmo usado es **Byte-Pair Encoding (BPE)**, que durante el entrenamiento identificó cuáles fragmentos de texto aparecen con más frecuencia y los convirtió en tokens individuales.

Una intuición: en inglés, palabras comunes como "the", "is", "of" son tokens individuales. Palabras largas o poco frecuentes se dividen en sub-palabras. Los símbolos de código como `{`, `}`, `=>` son tokens individuales.

### Visualizando la tokenización en diferentes contextos

```
EJEMPLOS DETALLADOS DE TOKENIZACIÓN
═══════════════════════════════════════════════════════════════

TEXTO EN INGLÉS (eficiente — el modelo fue entrenado principalmente en inglés):
  "Hello world"          → [Hello][ world]                    = 2 tokens
  "programming"          → [programm][ing]                    = 2 tokens
  "authentication error" → [authen][tication][ error]         = 3 tokens

TEXTO EN ESPAÑOL (menos eficiente — ~10-20% más tokens que inglés):
  "Hola mundo"           → [Hola][ mundo]                     = 2 tokens
  "autenticación"        → [autent][icaci][ón]                = 3 tokens
  "implementación"       → [implement][aci][ón]               = 3 tokens

CÓDIGO PYTHON:
  "def hello():"         → [def][ hello][()][:]               = 4 tokens
  "import hashlib"       → [import][ hash][lib]               = 3 tokens
  "return True"          → [return][ True]                    = 2 tokens
  "bcrypt.hashpw(p, s)"  → [bcrypt][.][hash][pw][(][p][,][ s][)] = 9 tokens

NÚMEROS:
  "42"                   → [42]                               = 1 token
  "12345"                → [123][45]                          = 2 tokens
  "3.14159"              → [3][.][14][159]                    = 4 tokens

SÍMBOLOS Y PUNTUACIÓN:
  "{"                    → [{]                                = 1 token
  "=>"                   → [=>]                               = 1 token
  "!="                   → [!=]                               = 1 token
```

### La regla práctica para estimar tokens

Aunque no necesitas calcular tokens exactamente, tener una intuición es muy útil para planear el trabajo:

| Tipo de contenido | Aproximación |
|-------------------|--------------|
| Texto en inglés | 1 token ≈ 4 caracteres |
| Texto en español | 1 token ≈ 3.5 caracteres |
| Código Python/JS | 1 token ≈ 3-4 caracteres |
| 1 línea de código (80 chars) | ≈ 20-25 tokens |
| 1 párrafo de texto (100 palabras) | ≈ 130-160 tokens |
| 1 página de texto (500 palabras) | ≈ 650-800 tokens |
| 1 archivo Python de 100 líneas | ≈ 1,500-2,500 tokens |
| 1 archivo Python de 500 líneas | ≈ 7,000-12,000 tokens |
| Un proyecto completo de 10k líneas | ≈ 100,000-200,000 tokens |

### Los tres impactos críticos de los tokens

**Impacto 1: La ventana de contexto — la "memoria de trabajo"**

La ventana de contexto es la cantidad máxima de tokens que Claude puede procesar en una sola sesión. En Claude, esta ventana es de aproximadamente **200,000 tokens**.

Pero ¿qué ocupa espacio en la ventana de contexto? Todo:

```
CONTENIDO DE LA VENTANA DE CONTEXTO
═══════════════════════════════════════════════════════════════

 [Instrucciones base del sistema]        ≈  2,000 tokens
 [Contenido de CLAUDE.md]                ≈  1,000-5,000 tokens
 [Historial de mensajes de la sesión]    ≈ variable (crece)
 [Archivos leídos en la sesión]          ≈ variable (crece)
 [Outputs de comandos ejecutados]        ≈ variable (crece)
 [Respuesta actual de Claude]            ≈ variable
 ─────────────────────────────────────────
 TOTAL MÁXIMO                            ≈ 200,000 tokens
```

Cuando el total se acerca al límite, Claude empieza a "olvidar" la información más antigua de la sesión. Esto se desarrolla en profundidad en el **Capítulo 5**.

**Impacto 2: El costo — pagas por tokens**

El servicio de API de Anthropic se factura por tokens. Hay dos tipos:

- **Input tokens (tokens de entrada):** Todo lo que envías al modelo — tu mensaje, el historial, el contenido de archivos leídos, los resultados de herramientas. Se cobran a menor precio.
- **Output tokens (tokens de salida):** Todo lo que el modelo genera — su respuesta y planificación. Se cobran a mayor precio (generalmente 3-5 veces más caros que los input).

Una sesión típica de desarrollo puede consumir 50,000-200,000 tokens. El costo varía según el modelo y el plan, pero tener conciencia de esto ayuda a usar Claude Code de forma eficiente.

**Impacto 3: La velocidad — más tokens = más tiempo**

El procesamiento de tokens lleva tiempo. Una sesión con contexto pequeño responde en 2-5 segundos. Una sesión donde el contexto ya tiene 150,000 tokens de historial puede tardar 20-40 segundos por iteración.

Esto es especialmente relevante en sesiones largas donde se han leído muchos archivos grandes.

### La temperatura: un parámetro que Claude Code gestiona por ti

Hay un parámetro llamado **temperatura** que controla la aleatoriedad del LLM al generar texto:
- **Temperatura 0.0:** El modelo siempre elige la opción más probable. Respuestas completamente deterministas. Ideal para código.
- **Temperatura 1.0:** Introduce variación estadística. Respuestas más creativas pero menos predecibles.

Claude Code usa automáticamente temperatura baja para tareas de programación. No necesitas configurar esto manualmente.

---

## 1.6 Cómo fluye realmente una conversación

### El viaje completo de un mensaje

Cuando escribes algo en Claude Code, ocurre una cadena de eventos sofisticada. Entender esta cadena es importante para entender las latencias, los costos, y por qué algunas operaciones son más caras que otras.

```
═══════════════════════════════════════════════════════════════
FLUJO COMPLETO DE UNA INTERACCIÓN CON CLAUDE CODE
═══════════════════════════════════════════════════════════════

TU COMPUTADORA — proceso local
────────────────────────────────────────────────────────────
PASO 1: Tú escribes en la terminal:
        > Migra auth.py de MD5 a bcrypt

PASO 2: Claude Code CLI lee tu input y lo procesa

PASO 3: El Context Manager construye el payload JSON completo:
        {
          "model": "claude-sonnet-4-20250514",
          "max_tokens": 8192,
          "system": "[Contenido de CLAUDE.md]\n[Instrucciones base]",
          "messages": [
            ...todos los mensajes anteriores de la sesión...,
            {
              "role": "user",
              "content": "Migra auth.py de MD5 a bcrypt"
            }
          ],
          "tools": [
            {"name": "Read",      "description": "Lee el contenido de un archivo"},
            {"name": "Write",     "description": "Escribe o crea un archivo"},
            {"name": "Edit",      "description": "Edita parte de un archivo"},
            {"name": "Bash",      "description": "Ejecuta un comando de terminal"},
            {"name": "WebSearch", "description": "Busca en internet"},
            {"name": "Agent",     "description": "Crea un subagente"}
          ]
        }

                    │
                    ▼ HTTPS POST /v1/messages
                      Headers: x-api-key: sk-ant-api03-...
                      (~50-200ms de latencia de red)
                    │
                    ▼

SERVIDORES DE ANTHROPIC — nube
────────────────────────────────────────────────────────────
PASO 4: El servidor autentica la petición (verifica la API key)

PASO 5: El modelo Claude recibe y procesa el payload completo
        (todo el contexto hasta 200,000 tokens entra a la vez
         en las GPUs de Anthropic)

PASO 6: El modelo genera su respuesta decidiendo:

        ┌── ¿Necesita información antes de actuar?
        │   → Respuesta: {"stop_reason": "tool_use",
        │                  "content": [{"type": "tool_use",
        │                               "name": "Read",
        │                               "input": {"path": "src/auth.py"}}]}
        │
        └── ¿Ha terminado y puede responder al usuario?
            → Respuesta: {"stop_reason": "end_turn",
                           "content": [{"type": "text",
                                         "text": "He completado la migración..."}]}

                    │
                    ▼ Respuesta JSON en streaming (token por token)
                      (~100ms - varios segundos según complejidad)
                    │
                    ▼

TU COMPUTADORA — de vuelta
────────────────────────────────────────────────────────────
PASO 7: Claude Code recibe la respuesta

        SI stop_reason = "tool_use":
          → Extrae herramienta y parámetros del JSON
          → Ejecuta la herramienta LOCALMENTE en tu computadora
          → Captura el resultado completo (stdout, stderr, contenido)
          → Añade el resultado al historial de mensajes
          → Vuelve al PASO 3 con el historial actualizado

        SI stop_reason = "end_turn":
          → Muestra la respuesta final al usuario
          → Espera el siguiente mensaje del usuario

═══════════════════════════════════════════════════════════════
```

### Lo que esto implica: cada acción es un viaje de ida y vuelta

Cuando Claude necesita leer 5 archivos, instalar una dependencia, y ejecutar los tests, eso implica **múltiples viajes de ida y vuelta** a los servidores de Anthropic:

```
EJEMPLO: Tarea que requiere 6 acciones
═══════════════════════════════════════════════════════════════

Petición 1: → Anthropic      Claude decide: "necesito leer auth.py"
            ← Tu máquina     Ejecutas: Read("src/auth.py")
Petición 2: → Anthropic      Claude decide: "necesito leer los tests"
            ← Tu máquina     Ejecutas: Read("tests/test_auth.py")
Petición 3: → Anthropic      Claude decide: "necesito ver requirements.txt"
            ← Tu máquina     Ejecutas: Read("requirements.txt")
Petición 4: → Anthropic      Claude decide: "actualizo auth.py"
            ← Tu máquina     Ejecutas: Write("src/auth.py", nuevo_contenido)
Petición 5: → Anthropic      Claude decide: "instalo bcrypt"
            ← Tu máquina     Ejecutas: Bash("pip install bcrypt")
Petición 6: → Anthropic      Claude decide: "verifico con tests"
            ← Tu máquina     Ejecutas: Bash("pytest tests/test_auth.py -v")
Petición 7: → Anthropic      Claude dice: "completado, aquí el resumen"

TOTAL: 7 viajes a Anthropic + 6 ejecuciones locales
TIEMPO TOTAL: 30 segundos - 3 minutos (según latencia y tamaño)
```

### Por qué las herramientas se ejecutan localmente

Un punto crucial: cuando Claude "lee un archivo" o "ejecuta un comando", esas operaciones ocurren **en tu computadora**, no en los servidores de Anthropic.

- **Lo que ocurre en Anthropic:** el "pensamiento" — decidir qué herramienta usar y con qué parámetros.
- **Lo que ocurre en tu máquina:** la "acción" — leer el archivo real, ejecutar el comando real.

Esto tiene implicaciones importantes:
1. **Privacidad parcial:** el contenido de tus archivos sí viaja a Anthropic (como resultado de la herramienta), pero tu sistema de archivos no es accedido directamente por Anthropic.
2. **Los permisos son los tuyos:** Claude solo puede hacer lo que tu usuario del SO puede hacer.
3. **El trabajo real ocurre localmente:** no dependes de la velocidad de Anthropic para ejecutar comandos.

### El streaming: por qué el texto aparece gradualmente

Cuando ves la respuesta de Claude aparecer poco a poco en la terminal, eso es **streaming**. La API envía los tokens de la respuesta a medida que se generan, no esperando a que esté completa.

Esto mejora enormemente la experiencia: ves el progreso inmediatamente en lugar de esperar en silencio 30 segundos y recibir todo de golpe.

---

## 1.7 Las herramientas: el puente entre el lenguaje y el sistema real

---

### Qué son las herramientas en el contexto de Claude Code

El elemento que transforma a Claude de un chatbot en un agente es el **sistema de herramientas**. Una herramienta, en este contexto, es una capacidad de acción que Claude puede invocar especificando parámetros concretos.

Conceptualmente, es como un contrato entre Claude y tu computadora: Claude dice "quiero ejecutar la herramienta X con los parámetros Y" y tu computadora ejecuta esa acción y devuelve el resultado.

### Las herramientas principales: guía completa

**Herramienta Read — Leer archivos**

Lee el contenido completo de cualquier archivo de texto.

```
FUNCIONAMIENTO:
  Claude invoca:    Read("src/auth.py")
  Tu computadora:   Abre el archivo y lee su contenido completo
  Devuelve a Claude: Todo el texto del archivo como string

CUÁNDO LA USA CLAUDE:
  - Antes de modificar cualquier archivo (para entender qué hay)
  - Para entender la estructura del proyecto
  - Para leer documentación, README, CLAUDE.md
  - Para verificar el resultado de un Write previo

ARCHIVOS QUE PUEDE LEER:
  ✓ Código fuente (.py, .js, .ts, .java, .go, .rs, .cpp...)
  ✓ Configuración (.json, .yaml, .toml, .ini, .env...)
  ✓ Documentación (.md, .rst, .txt...)
  ✓ Datos (.csv, .xml, .sql...)
  ✗ Binarios (.pdf, .jpg, .png, .docx, .exe...)
```

**Herramienta Write — Escribir archivos**

Crea un archivo nuevo o sobreescribe completamente uno existente.

```
FUNCIONAMIENTO:
  Claude invoca:    Write("src/auth.py", "import bcrypt\n\ndef hash_password...")
  Tu computadora:   Escribe el contenido en el archivo (crea o sobreescribe)
  Devuelve a Claude: Confirmación de que se escribió correctamente

CUÁNDO LA USA CLAUDE:
  - Para crear archivos nuevos
  - Para reemplazar completamente un archivo con nueva versión
  - Cuando los cambios son tan extensos que editar es ineficiente

IMPORTANTE:
  Write SOBREESCRIBE el archivo completo. Si el archivo tiene
  1000 líneas y Claude solo quiere cambiar una, debería usar Edit.
```

**Herramienta Edit — Edición quirúrgica**

Reemplaza un fragmento específico de un archivo sin reescribir todo.

```
FUNCIONAMIENTO:
  Claude invoca:    Edit("src/auth.py",
                        old_string="import hashlib",
                        new_string="import bcrypt")
  Tu computadora:   Localiza el fragmento exacto y lo reemplaza
  Devuelve a Claude: Confirmación del cambio aplicado

VENTAJA SOBRE WRITE:
  - Más eficiente para cambios pequeños en archivos grandes
  - Menor riesgo de perder contenido no relacionado
  - Muestra claramente qué cambió (el "diff" es visible)
```

**Herramienta Bash — Ejecutar comandos de terminal**

Ejecuta cualquier comando de terminal y captura su output completo.

```
FUNCIONAMIENTO:
  Claude invoca:    Bash("pytest tests/test_auth.py -v")
  Tu computadora:   Ejecuta el comando en la shell
  Devuelve a Claude: stdout + stderr completos del proceso

EJEMPLOS DE USO:
  Bash("pytest tests/ -v")              → ejecutar tests
  Bash("pip install bcrypt")            → instalar dependencias
  Bash("git diff HEAD~1")               → ver cambios recientes
  Bash("grep -r 'hash_password' .")     → buscar en el código
  Bash("find . -name '*.py' | sort")   → encontrar archivos
  Bash("npm run build")                 → compilar/construir
  Bash("docker compose up -d")         → levantar servicios
  Bash("cat requirements.txt")         → leer archivos rápido

LÍMITES DE SEGURIDAD:
  Claude solo puede ejecutar comandos que tu usuario del SO
  puede ejecutar. Si tu usuario no puede hacer sudo, Claude tampoco.
```

**Herramienta WebSearch — Búsqueda en internet**

Busca información actualizada en internet.

```
FUNCIONAMIENTO:
  Claude invoca:    WebSearch("bcrypt python latest version 2025")
  Internet:         Devuelve resultados de búsqueda
  Devuelve a Claude: Títulos, snippets y URLs relevantes

CUÁNDO LA USA CLAUDE:
  - Verificar la versión más reciente de una librería
  - Buscar soluciones a errores específicos (StackOverflow, GitHub Issues)
  - Obtener información sobre cambios recientes en APIs
  - Verificar documentación cuando no está seguro
```

**Herramienta Agent — Crear subagentes**

Crea un proceso de Claude independiente para trabajar en una subtarea.

```
FUNCIONAMIENTO:
  Claude invoca:    Agent(prompt="Revisa todos los tests en tests/
                                 y mejora la cobertura al 80%")
  Subagente:        Trabaja independientemente usando sus propias herramientas
  Devuelve a Claude: Resultado completo de la subtarea

CUÁNDO LA USA CLAUDE:
  - Tareas que pueden hacerse en paralelo
  - Proyectos muy grandes que requieren dividir el trabajo
  - Cuando una parte de la tarea es suficientemente independiente

DESARROLLADO EN: Capítulo 6 (Sistema Multi-Agente)
```

### Cómo Claude "sabe" qué herramienta usar

Cuando el payload se envía a la API, incluye la definición de todas las herramientas en formato JSON (sus nombres, descripciones, y esquemas de parámetros). Durante el entrenamiento, el modelo aprendió cuándo es apropiado usar cada herramienta.

El modelo infiere la herramienta correcta del contexto de la tarea, igual que un desarrollador humano infiere que necesita abrir un archivo antes de modificarlo, o que necesita ejecutar los tests para verificar un cambio.

---

## 1.8 Acceso al sistema real: la diferencia que lo cambia todo

### Comparativa de flujos de trabajo: números reales

```
MIGRAR AUTENTICACIÓN DE MD5 A BCRYPT
═══════════════════════════════════════════════════════════════

CON CHATBOT WEB:
  [5 min]  Abres auth.py, copias el código, lo pegas en el chat
  [2 min]  El chatbot da una versión actualizada de auth.py
  [3 min]  Copias, pegas en tu editor, guardas
  [2 min]  Ejecutas los tests → fallan (los tests verificaban MD5)
  [3 min]  Copias el error y lo pegas en el chat
  [1 min]  "Ah, también tienes que actualizar los tests"
  [5 min]  Copias test_auth.py, lo pegas en el chat
  [2 min]  El chatbot da tests actualizados
  [3 min]  Copias, pegas en tu editor
  [2 min]  Ejecutas → error: bcrypt no está instalado
  [3 min]  Copias el error, lo pegas en el chat
  [1 min]  "Necesitas pip install bcrypt y actualizar requirements.txt"
  [3 min]  Instalas, actualizas requirements.txt
  [2 min]  Ejecutas → los tests pasan
  [5 min]  Revisas todo para asegurarte de que tiene sentido
  ─────────────────────────────────────────────────────────────
  TOTAL: ~43 minutos
  ACCIONES MANUALES: ~15 operaciones de copiar/pegar

CON CLAUDE CODE:
  [30 seg] Escribes: "Migra la autenticación de MD5 a bcrypt"
  [4 min]  Claude trabaja autónomamente:
            • Lee auth.py
            • Examina tests/test_auth.py
            • Verifica requirements.txt
            • Actualiza auth.py con bcrypt
            • Actualiza los tests correctamente
            • Añade bcrypt a requirements.txt
            • Instala bcrypt
            • Ejecuta los tests para verificar
  [30 seg] Lees el resumen de cambios de Claude
  ─────────────────────────────────────────────────────────────
  TOTAL: ~5 minutos
  ACCIONES MANUALES: 1 (escribir el prompt)
═══════════════════════════════════════════════════════════════
```

### La exploración autónoma del contexto

Una de las capacidades más valiosas de Claude Code es que puede explorar la estructura de tu proyecto sin que le digas exactamente dónde está cada cosa.

Cuando le dices "arregla el bug de autenticación", Claude puede:

```bash
# Entender qué tipo de proyecto es
cat README.md
cat package.json   # o setup.py, Cargo.toml, go.mod...

# Encontrar todos los archivos relacionados con autenticación
grep -r "auth\|login\|password\|hash" --include="*.py" . | head -30

# Ver la estructura completa del proyecto
find . -name "*.py" | grep -v __pycache__ | sort

# Revisar cambios recientes relevantes
git log --oneline -15
git diff HEAD~3 -- src/auth.py
```

Esta exploración autónoma significa que Claude puede desarrollar una comprensión real del proyecto, no una aproximación basada en lo que tú crees que es relevante.

### La verificación como parte natural del proceso

Después de hacer cambios, Claude típicamente verifica su propio trabajo:

```
CICLO DE VERIFICACIÓN AUTOMÁTICA
═══════════════════════════════════════════════════════════════

Claude hace cambios en src/auth.py
        │
        ▼
Claude ejecuta: pytest tests/test_auth.py -v
        │
        ├── TODOS LOS TESTS PASAN
        │     └── Claude informa: "Completado. 5/5 tests pasan."
        │
        └── ALGÚN TEST FALLA
              └── Claude analiza el error
                    └── Claude hace corrección
                          └── Claude vuelve a ejecutar tests
                                └── (ciclo hasta que pasan)
```

Esta verificación automática es lo que hace que los cambios de Claude Code sean mucho más confiables que los de un chatbot. Claude no "supone" que funcionó — **lo verifica**.

---

## 1.9 Los modelos disponibles y cuándo usar cada uno

---

### Los modelos de Claude Code en profundidad

**Claude Haiku — El veloz y económico**

Haiku es el modelo más rápido y el menos costoso. Responde en 1-3 segundos para la mayoría de las tareas. Es ideal cuando:

- El costo por token es crítico (pipelines de CI/CD que procesan cientos de PRs)
- La tarea es simple y bien definida (añadir un comentario, reformatear código, renombrar una función)
- Necesitas alta velocidad para muchas operaciones pequeñas en paralelo
- Estás explorando o experimentando y no quieres gastar presupuesto

La limitación: puede fallar en tareas que requieren razonamiento de varios pasos, mantener coherencia en contextos muy largos, o tomar decisiones técnicas complejas.

**Claude Sonnet — El equilibrio (usa este por defecto)**

Sonnet es el modelo que Anthropic recomienda para el uso general. Tiene una capacidad de razonamiento muy alta con velocidades y costos razonables. Responde en 3-10 segundos para la mayoría de las tareas.

Casos de uso:
- Desarrollo de nuevas features
- Debugging de bugs complejos
- Refactoring de módulos
- Escritura y mejora de tests
- Documentación técnica
- Code review automatizado
- Análisis de código existente

El 85-90% del trabajo diario de desarrollo encaja perfectamente en Sonnet.

**Claude Opus — La potencia máxima**

Opus es el modelo más capaz de Anthropic. Es significativamente más lento y más caro que Sonnet, pero puede manejar los problemas más complejos con mayor coherencia.

Úsalo cuando:
- Diseñas la arquitectura de un sistema desde cero y necesitas considerar muchas dimensiones simultáneamente
- Refactorizas un sistema legacy con muchas interdependencias complejas
- Necesitas análisis de seguridad profundo con múltiples vectores de ataque
- Tomas una decisión técnica de alto impacto que es costosa de revertir
- Necesitas mantener coherencia perfecta en razonamientos muy extendidos

### Cómo cambiar el modelo en la práctica

```bash
# Al iniciar una sesión con un modelo específico
claude --model claude-opus-4-5
claude --model claude-haiku-4-5

# Dentro de una sesión activa
/model claude-opus-4-5
/model claude-haiku-4-5
/model                    # muestra el modelo actual
```

---

## 1.10 Capacidades completas de Claude Code

### Gestión de código y archivos

```
CAPACIDADES DE ARCHIVOS:
═══════════════════════════════════════════════════════════════
✓ Leer cualquier archivo de texto (código, config, docs, logs)
✓ Crear archivos nuevos
✓ Modificar archivos existentes (edición quirúrgica o completa)
✓ Buscar archivos por nombre (find, glob)
✓ Buscar contenido en archivos (grep, ripgrep)
✓ Ver metadatos de archivos (permisos, tamaño, fecha)
✓ Gestionar permisos (chmod si tiene permisos para ello)
```

### Ejecución y verificación

```
CAPACIDADES DE EJECUCIÓN:
═══════════════════════════════════════════════════════════════
✓ Ejecutar tests (pytest, jest, cargo test, go test, etc.)
✓ Compilar código (gcc, tsc, cargo build, go build, etc.)
✓ Instalar dependencias (pip, npm, cargo, gem, etc.)
✓ Ejecutar scripts de migración
✓ Ejecutar linting y formateo (eslint, flake8, rustfmt, etc.)
✓ Ejecutar análisis de seguridad (bandit, semgrep, etc.)
✓ Iniciar y parar servicios (docker, systemctl, etc.)
✓ Desplegar aplicaciones
```

### Gestión de versiones con git

```
CAPACIDADES DE GIT:
═══════════════════════════════════════════════════════════════
✓ Ver estado del repositorio (git status)
✓ Ver cambios sin commitear (git diff)
✓ Ver historial de commits (git log)
✓ Crear commits con mensajes descriptivos
✓ Crear y cambiar de ramas (git branch, git checkout)
✓ Hacer merge y rebase
✓ Ver quién cambió qué línea (git blame)
✓ Comparar versiones de un archivo (git diff HEAD~N -- archivo)
✓ Crear y aplicar stash
```

### Búsqueda e investigación

```
CAPACIDADES DE INVESTIGACIÓN:
═══════════════════════════════════════════════════════════════
✓ Buscar en internet (versiones, documentación, soluciones)
✓ Leer documentación técnica de URLs
✓ Verificar versiones de librerías en tiempo real
✓ Buscar soluciones a errores específicos
✓ Verificar mejores prácticas actualizadas
```

### Trabajo avanzado con multi-agente y MCP

```
CAPACIDADES AVANZADAS:
═══════════════════════════════════════════════════════════════
✓ Crear subagentes para trabajar en paralelo
✓ Dividir proyectos grandes en partes manejables
✓ Conectarse a bases de datos via MCP
✓ Interactuar con APIs externas via MCP
✓ Gestionar recursos cloud via MCP
✓ Integrarse con GitHub, Jira, Notion via MCP
```

---

## 1.11 Limitaciones y lo que Claude Code no puede hacer

### Limitaciones técnicas fundamentales

**No puede leer archivos binarios**

Claude Code puede leer archivos de texto, pero no puede interpretar binarios: imágenes (JPG, PNG, GIF), PDFs, documentos de Word/Excel, ejecutables compilados, archivos de base de datos binarios.

Para PDFs, se puede usar una herramienta de conversión primero:
```bash
# Convertir PDF a texto antes de que Claude lo lea
pdf2txt documento.pdf > documento.txt
```

**No recuerda sesiones anteriores**

Cada nueva sesión de Claude Code empieza completamente desde cero. No existe memoria persistente entre sesiones.

```
SESIÓN DEL LUNES (termina cuando cierras Claude):
  Has pasado 2 horas trabajando en el módulo de autenticación.
  Claude ha leído todos los archivos, entiende el contexto.
  Cierras Claude.

SESIÓN DEL MARTES (nueva sesión = nuevo contexto):
  Claude no sabe nada de lo que ocurrió el lunes.
  No sabe qué archivos modificaste.
  No sabe en qué punto quedó la tarea.
  
LA SOLUCIÓN:
  Mantener CLAUDE.md actualizado con el estado del proyecto.
  (Desarrollado en el Capítulo 12)
```

**No puede controlar interfaces gráficas**

Claude Code no puede:
- Hacer clic en botones en el navegador
- Interactuar con aplicaciones de escritorio
- Ver lo que hay en tu pantalla
- Controlar el ratón o el teclado

Solo puede trabajar con archivos y comandos de terminal.

**No puede acceder a redes privadas sin configuración**

Si tus bases de datos o servicios internos están en una red privada sin acceso desde internet, Claude Code no puede acceder a ellos directamente. La solución es configurar un servidor MCP dentro de esa red (Capítulo 7).

**No garantiza corrección absoluta**

Claude puede cometer errores. Puede:
- Malinterpretar un requerimiento ambiguo
- Generar código correcto en sintaxis pero con bug lógico
- Hacer una suposición incorrecta sobre el comportamiento esperado
- Tomar una decisión técnica que tú habrías tomado diferente

Siempre conviene revisar los cambios con `git diff` antes de hacer commit, especialmente en código crítico de negocio o de seguridad.

### Limitaciones de permisos del sistema operativo

Esta es la primera y más importante capa de seguridad de Claude Code:

```
REGLA FUNDAMENTAL:
  Claude Code solo puede hacer lo que TU USUARIO del SO puede hacer.

EJEMPLOS:

  Tu usuario NO tiene acceso a /etc/shadow
  → Claude Code tampoco puede leerlo

  Tu usuario NO puede escribir en /var/log/
  → Claude Code tampoco puede escribir ahí

  Tu usuario NO puede hacer sudo sin contraseña
  → Claude Code no puede escalar privilegios

  Ejecutas Claude Code en contenedor Docker con usuario sin privilegios
  → Claude Code también estará limitado a esos permisos
```

Esto significa que la configuración de permisos de tu entorno es tu principal herramienta de control sobre lo que Claude puede y no puede hacer.

### Limitaciones de la ventana de contexto

En sesiones muy largas con proyectos grandes:

- El contexto puede llenarse (~200,000 tokens)
- Las respuestas pueden degradarse en calidad
- Claude puede "olvidar" información del inicio de la sesión
- Es necesario usar `/compact` para comprimir el historial

Se desarrolla completamente en el **Capítulo 5**.

---

## 1.12 Comparativa con otras herramientas del ecosistema

### Claude Code vs. GitHub Copilot

```
GITHUB COPILOT:                    CLAUDE CODE:
════════════════════════════       ════════════════════════════
Plugin para el editor IDE          Herramienta de terminal

Sugiere código inline mientras     Ejecuta tareas completas
escribes (autocompletado)          bajo tu dirección

Alcance: la función o archivo      Alcance: proyecto completo
actual donde estás trabajando      con exploración autónoma

No puede ejecutar código           Puede ejecutar código,
No puede hacer commits             hacer commits, tests, etc.
No puede instalar dependencias     Puede instalar dependencias

Respuesta: milisegundos            Respuesta: segundos-minutos

Ideal para: "el siguiente bloque   Ideal para: "implementa esta
de código que necesito ahora       feature completa incluyendo
mismo"                             tests y documentación"
```

**¿Son competidores?** No — son complementarios. Copilot para el flujo de escritura de código fluido. Claude Code para tareas completas que requieren coordinación a nivel de proyecto.

### Claude Code vs. Claude.ai (chatbot web)

```
CLAUDE WEB (claude.ai):            CLAUDE CODE:
════════════════════════════       ════════════════════════════
Solo texto entra y sale            Puede acceder a tu sistema

No puede leer tus archivos         Lee, escribe, modifica archivos
No puede ejecutar código           Ejecuta cualquier comando
No puede verificar resultados      Verifica con tests reales

Interfaz visual en navegador       Terminal (más potente)

Gratis o suscripción mensual       Pago por tokens (API)
Sin configuración necesaria        Requiere API key y configuración

Ideal para: aprender, explorar     Ideal para: trabajo de desarrollo
ideas, preguntas conceptuales,     real que requiere acceso al
documentación genérica             proyecto y verificación
```

### Claude Code vs. Cursor / Windsurf

**Cursor** y **Windsurf** son editores de código basados en VS Code con IA integrada profundamente.

```
CURSOR / WINDSURF:                 CLAUDE CODE:
════════════════════════════       ════════════════════════════
IDE completo con GUI               Solo terminal, sin GUI

Integración visual con código      Todo por línea de comandos
(ves los cambios en el editor)     (los cambios están en archivos)

Más familiar para usuarios         Más potente para scripts,
que vienen de VS Code              CI/CD, servidores remotos

IA limitada al contexto del IDE    Acceso total al sistema
                                   operativo

Multi-agente limitado              Multi-agente completo
                                   con subagentes paralelos
```

**¿Cuándo preferir Claude Code?**
- En servidores remotos sin GUI
- En pipelines de CI/CD automatizados
- Cuando necesitas el sistema multi-agente completo
- Cuando necesitas integrar con herramientas MCP personalizadas
- Cuando el trabajo requiere scripts de automatización complejos

---

## 1.13 Los primeros pasos: instalación y primera sesión

### Requisitos previos

Para usar Claude Code necesitas:

```
REQUISITOS:
═══════════════════════════════════════════════════════════════
1. Node.js versión 18 o superior
   → Verificar: node --version  (debe mostrar v18.x o superior)
   → Instalar: https://nodejs.org

2. Una cuenta en Anthropic con plan de API activo
   → Registro: https://console.anthropic.com
   → Necesitas añadir créditos o tener una suscripción

3. Una API key de Anthropic
   → Formato: sk-ant-api03-...
   → Obtener en: https://console.anthropic.com/settings/keys
   → IMPORTANTE: trátala como una contraseña, nunca la publiques

4. Terminal (bash, zsh, PowerShell en Windows)
```

### Instalación

```bash
# Instalar Claude Code globalmente via npm
npm install -g @anthropic-ai/claude-code

# Verificar que se instaló correctamente
claude --version
# Output esperado: claude/X.X.X ...
```

### Configurar la API key

```bash
# Opción 1: Variable de entorno en la sesión actual
export ANTHROPIC_API_KEY="sk-ant-api03-TU_CLAVE_AQUI"

# Opción 2: Permanente en ~/.bashrc o ~/.zshrc
echo 'export ANTHROPIC_API_KEY="sk-ant-api03-TU_CLAVE_AQUI"' >> ~/.bashrc
source ~/.bashrc

# Opción 3: Claude te la pide interactivamente la primera vez
claude   # si no hay API key configurada, te la pedirá
```

### Tu primera sesión interactiva

```bash
# Navega a tu proyecto
cd ~/mi-proyecto

# Inicia Claude Code
claude

# Verás la pantalla de bienvenida:
# ╭──────────────────────────────────────────────╮
# │ ✻ Welcome to Claude Code!                   │
# │   /help for help, /status for token count  │
# ╰──────────────────────────────────────────────╯
# >
```

### Los comandos esenciales para empezar

```
COMANDOS DENTRO DE UNA SESIÓN ACTIVA:
═══════════════════════════════════════════════════════════════

INFORMACIÓN Y AYUDA:
  /help           → Ver todos los comandos disponibles
  /status         → Ver cuántos tokens has usado en la sesión
  /model          → Ver el modelo activo / cambiarlo

GESTIÓN DE LA SESIÓN:
  /clear          → Limpiar el historial de conversación (nueva sesión)
  /compact        → Comprimir el historial (cuando el contexto está lleno)

INTERRUPCIONES:
  Ctrl+C          → Interrumpir la respuesta/acción actual
  Ctrl+D          → Salir de Claude Code completamente
  Ctrl+Z          → Suspender el proceso (bg para recuperarlo)

PRIMERAS TAREAS PARA EXPLORAR:
  > ¿Qué hace este proyecto?
  (Claude leerá README, estructura de archivos, archivos clave)

  > Lista todos los archivos Python y explícame su propósito
  (Claude explorará y creará un mapa del proyecto)

  > ¿Hay tests? ¿Qué cobertura tienen?
  (Claude encontrará los tests y los analizará)

  > Encuentra todos los TODO, FIXME y HACK en el código
  (Claude buscará en todos los archivos)

  > Ejecuta los tests y dime si pasan
  (Claude ejecutará el test suite completo)
```

### Modo no interactivo para automatización

Claude Code puede usarse en scripts y pipelines de CI/CD sin interacción:

```bash
# Ejecutar una tarea directamente y salir
claude -p "Ejecuta los tests y reporta el resultado"

# En un script bash
RESULTADO=$(claude -p "Genera el changelog del último sprint" 2>&1)
echo "Changelog generado: $RESULTADO"

# En un pipeline de CI (GitHub Actions, GitLab CI, etc.)
- name: Code Review con Claude
  run: |
    claude -p "Revisa los cambios en este PR y busca bugs de seguridad"
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

# Leer instrucciones desde un archivo
cat instrucciones.md | claude -p "Ejecuta estas instrucciones"
```

### El archivo CLAUDE.md: cómo darle contexto persistente

Una de las primeras cosas que deberías hacer en cualquier proyecto es crear un archivo `CLAUDE.md` en la raíz. Claude lo lee automáticamente al inicio de cada sesión.

```markdown
# Mi Proyecto — Contexto para Claude

## Qué es este proyecto
Sistema de gestión de inventario para e-commerce.
Backend: Python 3.12 + FastAPI. Base de datos: PostgreSQL 15.
Frontend: React 18 + TypeScript.

## Comandos importantes
- Ejecutar tests: pytest tests/ -v
- Iniciar servidor de desarrollo: uvicorn main:app --reload
- Migraciones de BD: alembic upgrade head

## Convenciones del proyecto
- Usa black para formateo de código Python
- Tests en pytest con fixtures en conftest.py
- Commits en formato conventional: feat:, fix:, docs:, etc.
- Nunca hacer commit directamente a main — siempre PR

## Estado actual (actualizar regularmente)
- Trabajando en: módulo de facturación automática
- Pendiente: integración con Stripe, tests del módulo de reportes
```

Se desarrolla completamente en el **Capítulo 12**.

---

## 1.14 Resumen completo y glosario del capítulo

### Los puntos fundamentales

**1. El chatbot vs. el agente — la diferencia que lo cambia todo**

Un chatbot solo hace entrada/salida de texto. No puede acceder a tu sistema, no puede ejecutar código, no puede verificar resultados. Claude Code es un agente: puede leer archivos, ejecutar comandos, y verificar resultados. Esta diferencia transforma completamente lo que es posible: de 43 minutos de trabajo manual a 5 minutos de supervisión.

**2. El LLM como motor de razonamiento**

Claude es un Large Language Model entrenado en billones de textos. Su "razonamiento" emerge de patrones aprendidos. Es poderoso — puede entender código complejo, detectar bugs, diseñar soluciones — pero no infalible: puede alucinar, puede equivocarse, y no conoce información posterior a su fecha de corte de entrenamiento.

**3. Los tokens son la moneda fundamental**

Todo en Claude Code se mide en tokens: el costo (pagas por tokens), la velocidad (más tokens = más lento), y la memoria (ventana de contexto de ~200k tokens). Un archivo de 100 líneas de código Python consume aproximadamente 2,000 tokens.

**4. El ciclo petición-herramienta**

Cada acción que Claude toma implica un viaje de ida y vuelta a los servidores de Anthropic. Las herramientas se ejecutan localmente, pero la decisión de qué herramienta usar la toma el modelo en la nube. Leer 5 archivos = 5+ viajes a Anthropic.

**5. Las herramientas como puente**

Read, Write, Edit, Bash, WebSearch, y Agent son las herramientas que convierten al chatbot en agente. Sin ellas, Claude sería solo un generador de texto sofisticado. Con ellas, puede explorar, actuar, y verificar.

**6. Las limitaciones son igual de importantes que las capacidades**

Claude no recuerda sesiones pasadas, no puede leer binarios, no puede controlar GUIs, está limitado por los permisos del SO, y puede cometer errores. Conocer estas limitaciones es esencial para usarlo correctamente.

**7. Los modelos tienen niveles para diferentes necesidades**

Haiku para CI/CD y tareas simples. Sonnet para el 90% del trabajo diario. Opus para decisiones de arquitectura complejas de alto impacto.

### Glosario completo del capítulo

| Término | Definición |
|---------|------------|
| **LLM** | Large Language Model. Modelo de IA entrenado en enormes cantidades de texto para predecir y generar lenguaje. El "cerebro" de Claude. |
| **Agente autónomo** | Sistema de IA que puede tomar acciones reales (leer archivos, ejecutar comandos) además de generar texto. A diferencia del chatbot, puede actuar. |
| **Chatbot** | Sistema de IA que solo hace entrada/salida de texto. No puede acceder a archivos ni ejecutar código. |
| **Token** | Unidad básica de procesamiento del LLM. Fragmento de texto (aprox. 4 chars en inglés). Todo se mide en tokens: costo, velocidad, memoria. |
| **Ventana de contexto** | Espacio máximo de memoria de trabajo del LLM, medido en tokens (~200k en Claude). Todo lo que está en la conversación activa ocupa espacio aquí. |
| **API** | Application Programming Interface. El punto de acceso HTTP al modelo Claude en los servidores de Anthropic. |
| **API key** | Clave de autenticación (`sk-ant-api03-...`). Necesaria para acceder a la API. Tratarla como contraseña. |
| **Herramienta (Tool)** | Capacidad de acción que el modelo puede invocar: Read (leer), Write (escribir), Bash (comandos), WebSearch (internet), Agent (subagentes). |
| **Input tokens** | Tokens enviados al modelo (mensajes, historial, resultados de herramientas). Menor precio. |
| **Output tokens** | Tokens generados por el modelo (respuestas). Mayor precio (3-5x los input tokens). |
| **Streaming** | Los tokens de respuesta llegan progresivamente, no todos juntos. Por eso el texto "aparece" gradualmente. |
| **stop_reason** | Campo en la respuesta de la API: `"tool_use"` (Claude quiere ejecutar una herramienta) o `"end_turn"` (ha terminado). |
| **BPE** | Byte-Pair Encoding. Algoritmo para dividir texto en tokens. Identifica fragmentos frecuentes en el texto de entrenamiento. |
| **Temperatura** | Parámetro que controla la aleatoriedad (0 = determinista, 1 = creativo). Claude Code usa temperatura baja para código. |
| **Alucinación** | Cuando un LLM genera información falsa con apariencia de verdadera. Mitigado en Claude Code por la verificación con herramientas reales. |
| **Grounding** | Anclar el razonamiento del LLM en datos reales del sistema (archivos, outputs de comandos) en vez de suposiciones. |
| **ReAct** | Reasoning + Acting. El patrón de ciclos Thought→Action→Observation que gobierna cómo Claude aborda cada tarea. Ver Capítulo 2. |
| **CLAUDE.md** | Archivo en el proyecto que Claude lee automáticamente al inicio de cada sesión para entender el contexto. Ver Capítulo 12. |
| **MCP** | Model Context Protocol. Sistema para extender Claude con herramientas personalizadas (BDs, APIs, etc.). Ver Capítulo 7. |
| **Claude Haiku** | El modelo más rápido y económico. Para tareas simples y CI/CD. |
| **Claude Sonnet** | El modelo equilibrado. Recomendado para el 90% del trabajo diario. |
| **Claude Opus** | El modelo más capaz. Para decisiones técnicas complejas de alto impacto. |

---

## Ver también

- **[Capítulo 2](./cap-02-patron-react.md):** El Patrón ReAct — el ciclo de razonamiento y acción en detalle con ejemplos completos.
- **[Capítulo 4](./cap-04-sistema-herramientas.md):** El Sistema de Herramientas — cada herramienta explicada en profundidad con todos sus parámetros.
- **[Capítulo 5](./cap-05-ventana-contexto.md):** La Ventana de Contexto — cómo gestionar tokens, memoria y contexto en sesiones largas.
- **[Capítulo 9](./cap-09-seguridad-permisos.md):** Seguridad — permisos, capas de protección, y uso seguro en producción.
- **[Capítulo 12](./cap-12-claude-md-memoria.md):** CLAUDE.md — cómo dar contexto persistente y configurar el comportamiento de Claude.

---

> 📌 Siguiente capítulo: [Cap. 2 — El Patrón ReAct: Reason + Act](./cap-02-patron-react.md)
