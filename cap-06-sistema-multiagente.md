# Capítulo 6: El Sistema Multi-Agente en Profundidad

> **Nivel:** Requiere comprensión básica de Claude Code  
> **Prerequisito recomendado:** [Cap. 4](./cap-04-sistema-herramientas.md) y [Cap. 5](./cap-05-ventana-contexto.md)  
> **Objetivo:** Entender cómo y por qué usar múltiples agentes, cuándo es útil, y cómo diseñar arquitecturas multi-agente efectivas.

---

## Tabla de Contenidos

- [6.1 Por qué un solo agente no siempre es suficiente](#61-por-qué-un-solo-agente-no-siempre-es-suficiente)
- [6.2 La arquitectura hub-and-spoke](#62-la-arquitectura-hub-and-spoke)
- [6.3 El aislamiento de contexto como ventaja fundamental](#63-el-aislamiento-de-contexto)
- [6.4 Cómo se crea un subagente técnicamente](#64-cómo-se-crea-un-subagente)
- [6.5 Ejecución paralela vs. secuencial](#65-paralela-vs-secuencial)
- [6.6 Patrones de comunicación entre agentes](#66-patrones-de-comunicación)
- [6.7 Agentes especializados con herramientas restringidas](#67-agentes-especializados)
- [6.8 Casos de uso reales del sistema multi-agente](#68-casos-de-uso-reales)
- [6.9 Limitaciones y antipatrones del multi-agente](#69-limitaciones-y-antipatrones)
- [6.10 Trust levels: herencia de permisos entre agentes](#610-trust-levels)
- [6.11 Resumen y glosario del capítulo](#611-resumen-y-glosario)

---

## 6.1 Por qué un solo agente no siempre es suficiente

### El límite del agente único

Como hemos visto en el Capítulo 5, la ventana de contexto del LLM es finita. En proyectos grandes, las sesiones largas o las tareas que involucran muchos archivos, un único agente puede encontrarse con problemas serios:

**Problema 1: El contexto se llena**
```
Tarea: Documentar 80 módulos Python de un proyecto grande

Con un único agente:
  Read(módulo_1.py)   → +3,000 tokens
  Read(módulo_2.py)   → +3,000 tokens
  ...
  Read(módulo_80.py)  → +3,000 tokens
  
  Solo en archivos leídos: 80 × 3,000 = 240,000 tokens
  → ¡Supera el límite de 200,000 tokens!
```

**Problema 2: La calidad se degrada**
```
Incluso antes de llegar al límite:
  Módulos 1-10:  documentados con alta calidad (contexto al 15%)
  Módulos 30-40: calidad media (contexto al 50%)
  Módulos 70-80: calidad baja (contexto lleno, "lost in the middle")
  
  La calidad es inconsistente.
```

**Problema 3: La velocidad es subóptima**
```
Con un solo agente (en serie):
  Documenta módulo_1 → módulo_2 → ... → módulo_80
  
  Si cada módulo tarda 30 segundos: 80 × 30 = 40 minutos total
```

### La solución: dividir para vencer

```
Con 8 subagentes (en paralelo):
  Subagente A: módulos 1-10   (30 segundos, contexto fresco)
  Subagente B: módulos 11-20  (30 segundos, contexto fresco)
  ...todos a la vez...
  
  Tiempo total: ~30 segundos
  Calidad: consistente (cada subagente tiene contexto fresco)
```

---

## 6.2 La arquitectura hub-and-spoke

### El modelo hub-and-spoke

La arquitectura de agentes en Claude Code sigue el modelo **hub-and-spoke** (cubo y radios, como una rueda de bicicleta):

```
                    AGENTE PRINCIPAL (Hub)
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
      Subagente A     Subagente B     Subagente C
      (radio 1)       (radio 2)       (radio 3)
      
      Documenta       Documenta       Documenta
      módulos 1-10    módulos 11-20   módulos 21-30
```

**El Hub (Agente Principal):**
- Recibe la tarea del usuario.
- Analiza y decide cómo dividirla.
- Crea los subagentes y les asigna subtareas.
- Espera a que los subagentes terminen.
- Integra los resultados.
- Informa al usuario.

**Los Radios (Subagentes):**
- Cada uno recibe una subtarea específica.
- Trabajan con su propio contexto aislado.
- No saben de la existencia de los otros subagentes.
- Cuando terminan, retornan su resultado al hub.

### Las restricciones del modelo

Este modelo tiene restricciones importantes:
- **No hay comunicación lateral:** Los subagentes no pueden hablar entre sí.
- **La jerarquía es plana:** Los subagentes no pueden crear sub-subagentes (aunque técnicamente es posible, no es una práctica habitual).
- **La comunicación es solo padre → hijo (al crear) e hijo → padre (al terminar).**

---

## 6.3 El aislamiento de contexto como ventaja fundamental

### ¿Por qué es una ventaja que los subagentes no compartan contexto?

A primera vista parece una limitación. En realidad, el aislamiento es una característica deliberada y beneficiosa.

### La paradoja del contexto compartido

Imagina que tienes un equipo de 5 desarrolladores trabajando en la misma sala, pero con una regla: todos deben leer en voz alta cada línea de código que escriben, cada archivo que abren, cada comando que ejecutan.

El resultado: el 90% del tiempo del equipo se va en escuchar lo que hacen los demás, y cada persona solo puede procesar una fracción de su propio trabajo.

Esto es exactamente lo que ocurre con un contexto compartido: añadir el trabajo de cada subagente al contexto de todos los demás lo llenaría exponencialmente.

### El modelo de memoria humana como analogía

Un equipo de personas que trabaja eficientemente no necesita que cada persona sepa exactamente lo que hacen todos los demás en tiempo real. El coordinador (el jefe del proyecto) sabe el estado de cada tarea. Cada trabajador se enfoca en su área. Al final del día, el coordinador integra los resultados.

Los agentes funcionan igual:
- El hub es el coordinador.
- Cada subagente es un especialista enfocado.
- El contexto aislado es la "habitación propia" donde cada especialista puede trabajar sin distracciones.

### La comparativa cuantitativa

```
TAREA: Documentar 30 módulos Python

ENFOQUE 1: Un solo agente
  Hacia el final de la tarea:
  [System: 5k] + [Historial con 30 archivos: 90k] + [Trabajo actual: 10k]
  = ~105,000 tokens por petición
  
  → "Lost in the Middle" severo para los últimos módulos
  → Calidad inconsistente (alta al inicio, baja al final)
  → Alta latencia (contexto lleno)

ENFOQUE 2: 3 subagentes de 10 módulos cada uno
  [System: 5k] + [Historial con 10 archivos: 30k] + [Trabajo actual: 5k]
  = ~40,000 tokens por petición
  
  → Sin "lost in the middle" (contexto al 20% del límite)
  → Calidad consistente para todos los módulos
  → Baja latencia (contexto pequeño)
  → Tiempo total dividido por 3 (paralelismo)
```

### La analogía del equipo de especialistas

Un equipo de personas que trabaja eficientemente no necesita que cada persona sepa exactamente lo que hacen todos los demás en tiempo real. El jefe del proyecto (el hub) sabe el estado de cada tarea. Cada trabajador (subagente) se enfoca en su área. Al final, el jefe integra los resultados.

El contexto aislado es la "habitación propia" donde cada especialista puede trabajar sin distracciones.

---

## 6.4 Cómo se crea un subagente técnicamente

### La herramienta Agent

Desde la perspectiva del LLM, crear un subagente es invocar la herramienta `Agent`, igual que invocar `Read` o `Bash`. Pero lo que ocurre internamente es fundamentalmente diferente.

### El JSON de invocación de la herramienta Agent

```json
{
  "type": "tool_use",
  "id": "toolu_agent_01",
  "name": "Agent",
  "input": {
    "task": "Documenta con docstrings completos (estilo Google) todos los archivos Python en src/auth/. Para cada función pública incluye: descripción, Args, Returns, Raises, y un Example.",
    "context": "El proyecto es una API FastAPI. El estilo de referencia está en src/models/user.py (ya documentado). Type hints en todas las funciones públicas.",
    "allowed_tools": ["Read", "Write", "Glob"],
    "model": "claude-sonnet-4-20250514"
  }
}
```

### Lo que ocurre al ejecutar la herramienta Agent

```
1. Tool Executor recibe la solicitud

2. Crea una nueva instancia del agente:
   - Contexto nuevo y vacío
   - Solo recibe: el task + el context del input
   - Las herramientas: solo las de allowed_tools
   - NO tiene el historial del agente principal
   - NO sabe que otros subagentes existen

3. El subagente ejecuta su propio ciclo ReAct:
   THOUGHT: "Necesito encontrar todos los archivos .py en src/auth/"
   ACTION: Glob("src/auth/**/*.py")
   OBSERVATION: ["src/auth/service.py", "src/auth/tokens.py", ...]
   
   THOUGHT: "Empezaré por service.py"
   ACTION: Read("src/auth/service.py")
   OBSERVATION: [contenido del archivo]
   
   ACTION: Edit("src/auth/service.py", old_string=..., new_string=...)
   ...

4. Cuando el subagente completa (stop_reason: end_turn):
   → Retorna su respuesta como tool_result al agente principal

5. El agente principal recibe el resultado y continúa su ciclo.
```

### Los parámetros de configuración del subagente

**`task` (obligatorio):** La instrucción específica. Debe ser **autosuficiente**: el subagente no puede preguntar dudas mientras trabaja.

**`context` (recomendado):** Información contextual: convenciones del proyecto, rutas relevantes, decisiones técnicas previas.

**`allowed_tools` (recomendado):** Lista de herramientas disponibles. Por principio de mínimo privilegio, dale solo las que necesita.

**`model` (opcional):** El modelo a usar. Puede ser diferente al del agente principal (útil para optimizar costos con Haiku para tareas simples).

---

## 6.5 Ejecución paralela vs. secuencial

### Ejecución paralela

Los subagentes se ejecutan en **paralelo** cuando el agente principal los crea todos en la misma respuesta:

```json
{
  "content": [
    {"type": "text", "text": "Voy a crear tres subagentes para documentar en paralelo."},
    {
      "type": "tool_use", "id": "toolu_A", "name": "Agent",
      "input": {"task": "Documenta src/auth/ (módulos de autenticación)"}
    },
    {
      "type": "tool_use", "id": "toolu_B", "name": "Agent",
      "input": {"task": "Documenta src/api/ (módulos de la API)"}
    },
    {
      "type": "tool_use", "id": "toolu_C", "name": "Agent",
      "input": {"task": "Documenta src/models/ (modelos de datos)"}
    }
  ],
  "stop_reason": "tool_use"
}
```

El Tool Executor los lanza simultáneamente. **Tiempo total ≈ tiempo del subagente más lento** (no la suma de todos).

### Cuándo es posible el paralelismo

```
INDEPENDIENTES (pueden ir en paralelo):
  ✓ Documentar src/auth/ y documentar src/api/ → sin dependencias
  ✓ Generar tests para módulo A y módulo B → archivos separados
  ✓ Analizar deuda técnica en frontend y backend → análisis separados

CON DEPENDENCIAS (deben ir en secuencia):
  ✗ Crear schema de DB → LUEGO crear modelos (los modelos necesitan el schema)
  ✗ Analizar errores → LUEGO escribir el informe (el informe necesita el análisis)
  ✗ Refactorizar función A → LUEGO actualizar sus tests (los tests necesitan ver A)
```

### Ejecución secuencial con paso de información

Cuando hay dependencias, el agente principal orquesta la secuencia:

```
PASO 1: Principal crea Subagente A
  Tarea: "Analiza src/auth/ e identifica todos los puntos donde
          se manejan contraseñas. Lista las funciones y líneas."

ESPERA el resultado de A.

A retorna:
  "4 puntos encontrados:
   1. auth.py:23 - hash_password() usa MD5
   2. auth.py:28 - verify_password() compara hashes MD5
   ..."

PASO 2: Principal crea Subagente B con el resultado de A como contexto:
  Tarea: "Migra el sistema de hash de contraseñas de MD5 a bcrypt.
          Los puntos a modificar son: [RESULTADO DE A]"

→ Subagente B tiene exactamente la información que necesita.
```

---

## 6.6 Patrones de comunicación entre agentes

### Las limitaciones de la comunicación

Los subagentes son instancias aisladas. Una vez creados, no pueden:
- Recibir mensajes del principal mientras trabajan.
- Enviar mensajes al principal hasta que terminen.
- Comunicarse con otros subagentes.

Toda la comunicación ocurre en dos momentos:
1. **Al crear el subagente:** El principal pasa `task` y `context`.
2. **Cuando el subagente termina:** Retorna su resultado al principal.

### Patrón 1: El informe estructurado

Para que el agente principal pueda procesar fácilmente el resultado, instruye al subagente a retornar datos en formato estructurado:

```
task: "Analiza src/auth/ y retorna un análisis en este formato JSON exacto:
{
  'archivos_analizados': ['lista'],
  'vulnerabilidades': [
    {'archivo': '...', 'linea': N, 'descripcion': '...', 'severidad': 'alta/media/baja'}
  ],
  'resumen': 'texto breve'
}"
```

### Patrón 2: El contexto heredado

Cuando los subagentes necesitan conocer las convenciones del proyecto, pásalas en el `context`:

```python
# El agente principal construye el contexto para cada subagente
context_comun = f"""
Convenciones del proyecto:
- Python 3.12 con FastAPI
- bcrypt con factor 12 para hashes
- Pydantic v2 para validación
- pytest con fixtures en conftest.py

Archivo de referencia para estilo:
- Docstrings: ver src/models/user.py (ya documentado)
- Tests: ver tests/test_models.py (ya completo)
"""
```

### Patrón 3: El pipeline de transformación

Para transformaciones complejas, los subagentes se encadenan:

```
Subagente A → Lista de problemas → Subagente B (con lista de A) → Parches → Subagente C (con parches de B) → Verifica
```

---

## 6.7 Agentes especializados con herramientas restringidas

### El principio de mínimo privilegio aplicado a agentes

Al crear un subagente, restricciones sus herramientas al mínimo necesario:

```json
// Subagente de análisis (solo lectura)
{ "allowed_tools": ["Read", "Glob", "Grep"] }
// → No puede modificar nada

// Subagente de documentación (lectura + escritura)
{ "allowed_tools": ["Read", "Write", "Edit", "Glob"] }
// → Puede modificar archivos, pero no ejecutar comandos

// Subagente de tests (lectura + escritura + ejecución)
{ "allowed_tools": ["Read", "Write", "Edit", "Bash", "Glob"] }
// → Puede hacer todo lo necesario para trabajar con tests

// Subagente de análisis de seguridad (lectura + web)
{ "allowed_tools": ["Read", "Glob", "Grep", "WebSearch"] }
// → Puede leer código y buscar vulnerabilidades en internet
```

### Subagentes con modelos más baratos

Para tareas mecánicas, usa Haiku para reducir el costo en ~80%:

```json
{
  "name": "Agent",
  "input": {
    "task": "Para cada archivo Python en src/, añade el encabezado de licencia si no lo tiene: '# Copyright 2026 Mi Empresa. All rights reserved.'",
    "allowed_tools": ["Read", "Edit", "Glob"],
    "model": "claude-haiku-4-20250514"
  }
}
```

---

## 6.8 Casos de uso reales del sistema multi-agente

### Caso 1: Documentar un proyecto grande (paralelo)

```
Proyecto legacy con 60 archivos Python sin documentación.

Agente Principal:
  → Analiza la estructura: identifica 6 módulos principales

Lanza 6 subagentes en paralelo:
  Sub A: Documenta src/auth/   (10 archivos)
  Sub B: Documenta src/api/    (12 archivos)
  Sub C: Documenta src/models/ (8 archivos)
  Sub D: Documenta src/services/ (15 archivos)
  Sub E: Documenta src/utils/ (10 archivos)
  Sub F: Documenta tests/     (activa docstrings)

Agente Principal: integra resultados y reporta

Resultado:
  - Tiempo: ~5 minutos (vs. ~45 minutos en serie)
  - Calidad: consistente en todos los módulos
```

### Caso 2: Refactorización con análisis previo (secuencial + paralelo)

```
FASE 1 (secuencial): Subagente A analiza toda la base de código
  → Identifica 23 puntos que usan el sistema de autenticación antiguo

FASE 2 (paralelo, con los resultados de A):
  Subagente B: Implementa el nuevo sistema JWT
  Subagente C: Migra los endpoints de la API
  Subagente D: Actualiza los tests de autenticación

FASE 3 (secuencial, después de B+C+D): Subagente E verifica
  → Ejecuta toda la suite de tests y reporta resultados
```

### Caso 3: Auditoría de seguridad (paralelo especializado)

```
4 subagentes especializados en paralelo:

Subagente A (Inyección):
  "Busca vulnerabilidades de inyección SQL y command injection"
  allowed_tools: ["Read", "Grep", "WebSearch"]

Subagente B (Autenticación):
  "Analiza el sistema de autenticación buscando debilidades"
  allowed_tools: ["Read", "Grep", "WebSearch"]

Subagente C (Datos sensibles):
  "Busca exposición de datos sensibles: logs, respuestas de error"
  allowed_tools: ["Read", "Grep"]

Subagente D (Dependencias):
  "Analiza las dependencias del proyecto buscando versiones con CVEs"
  allowed_tools: ["Read", "Bash", "WebSearch"]

Agente Principal: consolida hallazgos en informe de seguridad priorizado
```

---

## 6.9 Limitaciones y antipatrones del multi-agente

### Limitación 1: El overhead de inicialización

Cada subagente tiene un costo de inicialización (tiempo + tokens del system prompt + primera petición a la API). Para tareas pequeñas, este overhead supera el beneficio:

```
MAL USO: Lanzar subagente para "Añade un comentario a la línea 5 de auth.py"
  → El overhead es mayor que hacer el cambio directamente

BUEN USO: Lanzar subagente para "Documenta los 20 archivos de src/auth/"
  → La tarea es suficientemente grande para amortizar el overhead
```

### Limitación 2: Sin comunicación en tiempo real

Los subagentes no pueden pedir aclaraciones mientras trabajan. El contexto inicial debe ser **suficientemente completo** para resolver cualquier ambigüedad:

```
MAL: "Mejora el código de src/auth.py"
  → Ambiguo: ¿mejora de rendimiento? ¿seguridad? ¿legibilidad?
  → El subagente tomará la decisión que le parezca (puede no ser la que quieres)

BIEN: "Mejora la seguridad de src/auth.py específicamente:
       1. Cambia MD5 por bcrypt (factor 12)
       2. Añade rate limiting a la función login()
       3. Valida que el username no tenga caracteres especiales"
```

### Limitación 3: Conflictos al escribir los mismos archivos

Si dos subagentes modifican el mismo archivo simultáneamente, puede haber conflictos. La segunda escritura sobreescribe partes de la primera.

**Solución:** Dividir el trabajo para que no haya overlaps (archivos separados, directorios separados).

### Antipatrón: La cascada infinita

Subagentes que crean subagentes que crean más subagentes → costos exponenciales e impredecibles. Mantener la jerarquía plana: principal → subagentes (un solo nivel de profundidad).

### Antipatrón: Subagentes para todo

```
NO NECESITA MULTI-AGENTE:
  - "Corrige el bug en la línea 45 de auth.py"
  - "Escribe tests para hash_password"
  - "Añade logging a service.py"

SÍ SE BENEFICIA DEL MULTI-AGENTE:
  - "Documenta los 50 módulos del proyecto"
  - "Genera tests para todas las clases del módulo de modelos"
  - "Audita la seguridad de toda la aplicación"
```

---

## 6.10 Trust levels: herencia de permisos entre agentes

### El modelo de confianza

Los subagentes **heredan los permisos del agente que los crea**. Si el agente principal tiene permiso para ejecutar comandos Bash, sus subagentes también lo tienen (salvo restricción con `allowed_tools`).

```
Agente Principal (permisos: Read, Write, Bash, WebSearch)
  │
  ├── Subagente A (allowed_tools: ["Read", "Glob"])
  │   → RESTRINGIDO: solo puede leer
  │
  ├── Subagente B (sin allowed_tools especificado)
  │   → HEREDA todos los permisos del principal
  │
  └── Subagente C (allowed_tools: ["Read", "Write", "Bash"])
      → RESTRINGIDO: no tiene WebSearch
```

### Implicaciones de seguridad

1. Si el principal tiene `--dangerously-skip-permissions`, sus subagentes también.
2. Si quieres un subagente de "solo lectura" para mayor seguridad, debes especificarlo con `allowed_tools`.
3. No puedes dar a un subagente **más** permisos de los que tiene el principal.

### La práctica recomendada

```
BUENA PRÁCTICA: Principio de mínimo privilegio

Para cada subagente, pregúntate:
  - ¿Necesita leer archivos? → Añade: Read, Glob, Grep
  - ¿Necesita escribir archivos? → Añade: Write, Edit
  - ¿Necesita ejecutar comandos? → Añade: Bash
  - ¿Necesita internet? → Añade: WebSearch, WebFetch

Y dale SOLO lo que necesita.
```

---

## 6.11 Resumen y glosario

### Resumen

1. El **sistema multi-agente** resuelve los problemas de un agente único: contexto lleno, degradación de calidad, lentitud en tareas masivas.
2. La arquitectura es **hub-and-spoke**: el agente principal coordina; los subagentes trabajan con contextos aislados.
3. El **aislamiento de contexto** es una ventaja: cada subagente trabaja con contexto fresco, produciendo calidad consistente.
4. Los subagentes se crean con la herramienta **Agent**, pasando `task`, `context` y `allowed_tools`.
5. El **paralelismo** es posible cuando las subtareas son independientes; la **secuencia** se usa cuando hay dependencias.
6. La **comunicación** ocurre solo al crear el subagente y al terminar.
7. Los subagentes pueden usar **modelos más baratos** para tareas simples (optimización de costos).
8. Los subagentes **heredan los permisos** del principal; usa `allowed_tools` para restringirlos.

### Glosario

| Término | Definición |
|---------|------------|
| **Hub-and-spoke** | Arquitectura donde el principal coordina y los subagentes ejecutan. |
| **Subagente** | Instancia del agente Claude con contexto propio, creada por el agente principal. |
| **Aislamiento de contexto** | Cada subagente tiene su propia ventana de contexto independiente. |
| **Paralelismo** | Ejecución simultánea de múltiples subagentes cuando las tareas son independientes. |
| **Herramienta Agent** | La herramienta que invoca el LLM para crear un subagente. |
| **allowed_tools** | Lista de herramientas disponibles para un subagente (principio de mínimo privilegio). |
| **Overhead de inicialización** | Costo en tiempo y tokens de crear un subagente (amortizable en tareas grandes). |
| **Trust level** | Nivel de permisos heredado del agente padre. |

---

## Ver también

- **[Capítulo 4](./cap-04-sistema-herramientas.md):** Herramienta Agent — la herramienta que habilita el multi-agente.
- **[Capítulo 5](./cap-05-ventana-contexto.md):** Ventana de Contexto — por qué el aislamiento de contexto es beneficioso.
- **[Capítulo 9](./cap-09-seguridad-permisos.md):** Seguridad — trust levels y permisos en más detalle.
- **[Capítulo 11](./cap-11-performance-optimizacion.md):** Performance — la paradoja del costo en multi-agente.

---

> 📌 Siguiente capítulo: [Cap. 7 — MCP: Arquitectura del Protocolo](./cap-07-mcp-protocolo.md)
