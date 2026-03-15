# CLAUDE CODE: ARQUITECTURA Y FUNCIONAMIENTO TÉCNICO PROFUNDO

## 📚 TABLA DE CONTENIDOS

1. [¿Qué es Claude Code Internamente?](#1-qué-es-claude-code-internamente)
2. [El Patrón ReAct: Reason + Act](#2-el-patrón-react-reason--act)
3. [Arquitectura de Componentes](#3-arquitectura-de-componentes)
4. [El Sistema de Herramientas (Tools)](#4-el-sistema-de-herramientas-tools)
5. [La Ventana de Contexto: Cómo Funciona la Memoria](#5-la-ventana-de-contexto-cómo-funciona-la-memoria)
6. [El Sistema Multi-Agente en Profundidad](#6-el-sistema-multi-agente-en-profundidad)
7. [MCP: Arquitectura del Protocolo](#7-mcp-arquitectura-del-protocolo)
8. [El Sistema de Hooks: Eventos y Automatización](#8-el-sistema-de-hooks-eventos-y-automatización)
9. [Seguridad: Modelo de Permisos y Confianza](#9-seguridad-modelo-de-permisos-y-confianza)
10. [Ciclo de Vida Completo de una Sesión](#10-ciclo-de-vida-completo-de-una-sesión)
11. [Performance y Optimización Técnica](#11-performance-y-optimización-técnica)

---

## 1. ¿QUÉ ES CLAUDE CODE INTERNAMENTE?

### Más Allá de "Un Chatbot de Código"

Claude Code no es un chatbot que "sabe de código". Es un **agente autónomo** con arquitectura fundamentalmente diferente a un chatbot.

**La diferencia es estructural:**

```
CHATBOT TRADICIONAL (ej: claude.ai web):
┌───────────────────────────────────────────────────┐
│                                                   │
│  Usuario ──── texto ────► Modelo LLM              │
│                                │                  │
│                                ▼                  │
│  Usuario ◄─── texto ────── Respuesta              │
│                                                   │
│  El modelo SOLO genera texto.                     │
│  No puede tocar nada fuera de esta caja.          │
└───────────────────────────────────────────────────┘


AGENTE (Claude Code):
┌───────────────────────────────────────────────────────┐
│                                                       │
│  Usuario ──── texto ────► Agente                      │
│                             │                         │
│                             ▼                         │
│                    ┌─── ¿Necesito más info? ───┐      │
│                    │                           │      │
│                    ▼ Sí                        │ No   │
│             Llama herramienta                  │      │
│          (Read/Write/Bash/Web)                 │      │
│                    │                           │      │
│                    ▼                           │      │
│             Obtiene resultado                  │      │
│                    │                           │      │
│                    └─────────────► Razona con  │      │
│                                   nueva info   │      │
│                                       │        │      │
│                                       ▼        │      │
│                               ¿Tarea completa?─┘      │
│                                       │               │
│                                       ▼               │
│  Usuario ◄─── respuesta ─────── Respuesta final       │
│                                                       │
│  El agente ACTÚA en el sistema: lee archivos reales,  │
│  ejecuta comandos reales, modifica código real.       │
└───────────────────────────────────────────────────────┘
```

### La Definición Técnica

Claude Code es un **agente de programación autónomo** que implementa el paradigma **LLM + Tools + Feedback Loop**:

- **LLM**: El modelo Claude (Sonnet, Opus, Haiku) como motor de razonamiento
- **Tools**: Herramientas que permiten actuar sobre el sistema (Read, Write, Bash, etc.)
- **Feedback Loop**: Cada acción produce un resultado que se retroalimenta al modelo para la siguiente decisión

Lo que hace que esto sea poderoso: el modelo no solo "sabe" qué hay que hacer, sino que **puede hacerlo** y **verificar que lo hizo bien**.

---

## 2. EL PATRÓN REACT: REASON + ACT

### ¿Qué es ReAct?

**ReAct** (Reasoning + Acting) es el patrón arquitectónico que usa Claude Code. Fue introducido en el paper "ReAct: Synergizing Reasoning and Acting in Language Models" (2022).

La idea central es que un modelo LLM alternará entre:
1. **Razonar** (Thought): pensar sobre la situación y decidir qué hacer
2. **Actuar** (Action): ejecutar una herramienta
3. **Observar** (Observation): ver el resultado de la acción
4. Volver al paso 1 con la nueva información

Este ciclo se repite hasta que la tarea está completa.

### El Ciclo Completo en Detalle

Tomemos una tarea real: *"En src/auth.py hay un bug con el hash de passwords, investígalo y corrígelo"*

```
═══════════════════════════════════════════════════════════════
ITERACIÓN 1: Exploración inicial
═══════════════════════════════════════════════════════════════

THOUGHT (razonamiento interno del modelo):
  "Para investigar el bug del hash de passwords, primero
   necesito leer el archivo src/auth.py y ver el código actual."

ACTION (llamada a herramienta):
  tool: Read
  input: { "path": "src/auth.py" }

OBSERVATION (resultado de la herramienta):
  "def hash_password(password: str) -> str:
       return hashlib.md5(password.encode()).hexdigest()
   
   def verify_password(password: str, hashed: str) -> bool:
       return hash_password(password) == hashed"

═══════════════════════════════════════════════════════════════
ITERACIÓN 2: Análisis del problema
═══════════════════════════════════════════════════════════════

THOUGHT:
  "Encontré el problema: hash_password usa MD5. MD5 es
   criptográficamente roto (vulnerable a rainbow tables y 
   ataques de fuerza bruta modernos). Debo cambiarlo a bcrypt,
   que es el estándar actual para passwords.
   
   Antes de modificar, voy a ver si hay tests para esta función."

ACTION:
  tool: Bash
  input: { "command": "grep -r 'hash_password\|verify_password' tests/" }

OBSERVATION:
  "tests/test_auth.py:    assert hash_password('test123') == 'cc03e...'
   tests/test_auth.py:    assert verify_password('test123', expected_hash)"

═══════════════════════════════════════════════════════════════
ITERACIÓN 3: Planificación
═══════════════════════════════════════════════════════════════

THOUGHT:
  "Hay tests que verifican el hash con un valor MD5 hardcodeado.
   Si cambio a bcrypt, los tests van a fallar porque bcrypt genera
   hashes diferentes cada vez (incluye salt). Necesito:
   1. Actualizar hash_password para usar bcrypt
   2. Actualizar verify_password para verificar con bcrypt
   3. Actualizar los tests para no comparar con valores hardcodeados
   
   Primero verifico que bcrypt está disponible como dependencia."

ACTION:
  tool: Bash
  input: { "command": "cat requirements.txt | grep bcrypt" }

OBSERVATION:
  ""  ← vacío, bcrypt no está en requirements.txt

═══════════════════════════════════════════════════════════════
ITERACIÓN 4: Resolver dependencia
═══════════════════════════════════════════════════════════════

THOUGHT:
  "bcrypt no está en requirements.txt. Necesito añadirlo."

ACTION:
  tool: Write
  input: {
    "path": "requirements.txt",
    "content": "[contenido actual] + bcrypt==4.1.2\n"
  }

OBSERVATION:
  "Archivo escrito exitosamente"

═══════════════════════════════════════════════════════════════
ITERACIÓN 5, 6, 7: Implementar el fix y actualizar tests
...
═══════════════════════════════════════════════════════════════

ITERACIÓN FINAL: Verificación
═══════════════════════════════════════════════════════════════

THOUGHT:
  "He actualizado hash_password, verify_password y los tests.
   Necesito ejecutar los tests para verificar que todo funciona."

ACTION:
  tool: Bash
  input: { "command": "pytest tests/test_auth.py -v" }

OBSERVATION:
  "test_hash_password_uses_bcrypt PASSED
   test_verify_password_correct PASSED  
   test_verify_password_wrong PASSED
   3 passed in 0.45s"

THOUGHT:
  "Todos los tests pasan. La tarea está completa."

→ Claude responde al usuario con el resumen de lo que hizo.
```

### ¿Por Qué Este Patrón es Poderoso?

```
GENERACIÓN DE TEXTO TRADICIONAL:          REACT:
  
  Input → Modelo → Output                  Input → Modelo → Acción
                                                     ↑          ↓
  El modelo trabaja con información        Observación ← Sistema
  incompleta. Solo sabe lo que le
  das en el prompt inicial.               El modelo puede DESCUBRIR
                                          información que no tenía.
  Si no tienes toda la info               Puede VERIFICAR su propio trabajo.
  en el prompt, el resultado              Puede ITERAR hasta que funciona.
  puede ser incorrecto.
```

El resultado práctico: Claude Code puede resolver problemas que requieren exploración, donde no se sabe de antemano exactamente qué información se necesita o qué cambios serán necesarios.

---

## 3. ARQUITECTURA DE COMPONENTES

### Vista de Alto Nivel

```
┌────────────────────────────────────────────────────────────────┐
│                        TU MÁQUINA LOCAL                        │
│                                                                │
│   ┌────────────────────────────────────────────────────────┐   │
│   │                   CLAUDE CODE CLI                      │   │
│   │                                                        │   │
│   │  ┌──────────────┐         ┌─────────────────────────┐ │   │
│   │  │     REPL     │         │    AGENTE PRINCIPAL     │ │   │
│   │  │              │────────►│                         │ │   │
│   │  │  Interfaz    │         │  ┌──────────────────┐   │ │   │
│   │  │  de usuario  │◄────────│  │   Planificador   │   │ │   │
│   │  │  (prompt >)  │         │  │   (ReAct loop)   │   │ │   │
│   │  └──────────────┘         │  └────────┬─────────┘   │ │   │
│   │                           │           │              │ │   │
│   │  ┌──────────────┐         │  ┌────────▼─────────┐   │ │   │
│   │  │   CONTEXT    │         │  │   Context        │   │ │   │
│   │  │   MANAGER    │◄───────►│  │   Manager        │   │ │   │
│   │  │              │         │  └────────┬─────────┘   │ │   │
│   │  └──────────────┘         │           │              │ │   │
│   │                           │  ┌────────▼─────────┐   │ │   │
│   │  ┌──────────────┐         │  │  Tool Executor   │   │ │   │
│   │  │  MCP CLIENT  │◄───────►│  │                  │   │ │   │
│   │  │              │         │  └──────────────────┘   │ │   │
│   │  └──────────────┘         └─────────────────────────┘ │   │
│   └───────────────────────────────────────┬────────────────┘   │
│                                           │                    │
│   ┌──────────────────┐    ┌───────────────▼──────────────────┐ │
│   │  TU PROYECTO     │    │          SISTEMA                  │ │
│   │                  │◄──►│  (filesystem, bash, red, etc.)   │ │
│   │  src/            │    │                                  │ │
│   │  tests/          │    └──────────────────────────────────┘ │
│   │  CLAUDE.md       │                                         │
│   └──────────────────┘                                         │
└───────────────────────────────┬────────────────────────────────┘
                                │ HTTPS (TLS 1.3)
                    ┌───────────▼────────────────┐
                    │     ANTHROPIC API          │
                    │                            │
                    │  claude-opus-4             │
                    │  claude-sonnet-4           │
                    │  claude-haiku-4            │
                    └────────────────────────────┘
```

### Descripción de Cada Componente

**REPL (Read-Eval-Print Loop)**
La interfaz de usuario: el prompt `>` donde escribes. Lee tu input, lo pasa al agente, recibe la respuesta y la muestra. También gestiona el historial de la sesión, los atajos de teclado (configurados con `/terminal-setup`) y la visualización del progreso.

**Agente Principal**
El "cerebro". Implementa el bucle ReAct: recibe una tarea, llama a la API de Anthropic con el contexto completo, recibe la respuesta (que puede ser una llamada a herramienta o una respuesta final), ejecuta la herramienta si es necesario, y repite hasta completar la tarea.

**Context Manager**
Gestiona qué información entra en cada request a la API. Se encarga de:
- Incluir el contenido de CLAUDE.md
- Mantener el historial de conversación
- Añadir los resultados de herramientas previas
- Monitorear el tamaño del contexto y avisar cuando se acerca al límite

**Tool Executor**
Ejecuta las herramientas que solicita Claude. Para Read/Write, opera el filesystem. Para Bash, lanza un proceso. Gestiona las confirmaciones de seguridad antes de operaciones peligrosas. Captura outputs y errores para devolverlos al modelo.

**MCP Client**
Gestiona las conexiones con los MCP Servers configurados. Lanza los servidores MCP como procesos hijos, comunica con ellos via el protocolo MCP, y expone sus herramientas al agente principal como si fueran herramientas nativas.

---

## 4. EL SISTEMA DE HERRAMIENTAS (TOOLS)

### ¿Cómo Funciona una Herramienta Técnicamente?

Las herramientas se definen como JSON Schema y se envían a la API de Anthropic junto con el mensaje. El modelo Claude sabe que puede "llamar" a estas herramientas y genera una respuesta especial cuando quiere usarlas.

**Lo que se envía a la API (simplificado):**
```json
{
  "model": "claude-sonnet-4",
  "system": "[contenido de CLAUDE.md]",
  "messages": [...historial...],
  "tools": [
    {
      "name": "Read",
      "description": "Lee el contenido de un archivo del sistema de archivos",
      "input_schema": {
        "type": "object",
        "properties": {
          "path": {
            "type": "string",
            "description": "Ruta del archivo a leer"
          }
        },
        "required": ["path"]
      }
    },
    {
      "name": "Write",
      "description": "Escribe contenido en un archivo",
      "input_schema": { ... }
    },
    {
      "name": "Bash",
      "description": "Ejecuta un comando bash",
      "input_schema": { ... }
    }
  ]
}
```

**Lo que devuelve la API cuando Claude quiere usar una herramienta:**
```json
{
  "content": [
    {
      "type": "text",
      "text": "Necesito leer el archivo para entender el problema."
    },
    {
      "type": "tool_use",
      "id": "tool_abc123",
      "name": "Read",
      "input": {
        "path": "src/auth.py"
      }
    }
  ],
  "stop_reason": "tool_use"
}
```

El CLI detecta que el `stop_reason` es `tool_use`, extrae el nombre y los parámetros, ejecuta la operación localmente, y envía de vuelta el resultado:

```json
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "tool_abc123",
      "content": "def hash_password(password: str) -> str:\n    return hashlib.md5(password.encode()).hexdigest()\n..."
    }
  ]
}
```

### Herramientas Disponibles y Cuándo se Usan

#### `Read` — Leer Archivos

La herramienta más usada. Lee el contenido de archivos de texto.

```
Cuándo la usa Claude:
  - Para entender código antes de modificarlo
  - Para leer configuraciones y dependencias
  - Para ver tests existentes antes de generar nuevos
  - Para leer documentación del proyecto

Limitaciones:
  - Solo archivos de texto (no binarios: imágenes, PDFs, ejecutables)
  - El contenido completo se añade al contexto (archivos muy grandes
    consumen muchos tokens)
  - No tiene acceso a archivos fuera del directorio de trabajo
    a menos que se especifique una ruta absoluta explícita
```

#### `Write` — Escribir/Modificar Archivos

Crea un archivo nuevo o sobreescribe el contenido de uno existente.

```
Características importantes:
  - Sobreescribe el archivo COMPLETO, no hace edits parciales
  - Claude debe leer el archivo antes de modificarlo para no
    perder contenido existente
  - Crea directorios intermedios si no existen

Cuándo pide confirmación:
  - Nunca pide confirmación para Write (se considera seguro
    porque los cambios se pueden revertir con git)
  - PERO los hooks PostToolUse sí se ejecutan después
```

#### `Bash` — Ejecutar Comandos

La herramienta más poderosa y la más peligrosa. Ejecuta cualquier comando en el shell.

```
Casos de uso:
  - Ejecutar tests: pytest tests/ -v
  - Instalar dependencias: pip install bcrypt
  - Operaciones de git: git log, git diff
  - Buscar en código: grep -r "TODO" src/
  - Compilar: npm run build, cargo build
  - Cualquier operación del sistema operativo

Cuándo pide confirmación:
  - rm, rmdir (eliminar)
  - comandos con sudo
  - git push, git reset --hard
  - Comandos que el sistema detecta como potencialmente destructivos
  
Timeout:
  - Los comandos tienen un timeout (por defecto varios minutos)
  - Si un comando tarda demasiado, Claude puede intentar interrumpirlo
```

#### `WebSearch` — Búsqueda en Internet

Permite a Claude buscar información actualizada en internet.

```
Cuándo lo usa Claude:
  - Buscar documentación de una librería desconocida
  - Investigar patrones de código actuales
  - Verificar si una API ha cambiado
  - Buscar soluciones a errores específicos

Limitaciones:
  - No es un navegador completo (no ejecuta JavaScript)
  - Solo accede a información públicamente disponible
  - Los resultados se incluyen en el contexto (consumen tokens)
```

#### `Agent` — Crear Subagentes

La herramienta que permite el sistema multi-agente.

```
Cómo funciona:
  - El agente principal llama a la herramienta Agent con:
    - La tarea que debe realizar el subagente
    - El contexto relevante para esa tarea
    - Las herramientas que puede usar el subagente
  
  - Se crea una nueva instancia de Claude con su propio contexto
  - El subagente ejecuta su tarea (también con ReAct)
  - Cuando termina, devuelve su resultado al agente principal
  - El agente principal recibe el resultado y continúa

Diferencia con los otros tools:
  - Los otros tools (Read, Write, Bash) son síncronos: Claude espera
    el resultado inmediato de la operación del sistema
  - Agent es diferente: puede lanzar múltiples subagentes y esperar
    a que todos completen (paralelismo)
```

### Limitar las Herramientas Disponibles

Puedes controlar qué herramientas puede usar Claude con los flags del CLI:

```bash
# Solo lectura (modo de auditoría o exploración segura)
claude --allowedTools "Read"
# Claude puede leer archivos pero no puede escribir nada ni ejecutar comandos

# Lectura y escritura (sin ejecución de comandos)
claude --allowedTools "Read,Write"
# Útil si no quieres que Claude ejecute código, solo que modifique archivos

# Sin búsqueda en internet
claude --disallowedTools "WebSearch"
# Claude no puede buscar en internet (útil para trabajar offline)

# Sin subagentes (trabajo en modo single-agent)
claude --disallowedTools "Agent"
# Claude no puede crear subagentes, trabaja todo en secuencia
```

---

## 5. LA VENTANA DE CONTEXTO: CÓMO FUNCIONA LA MEMORIA

### El Concepto Fundamental

La **ventana de contexto** es la "memoria de trabajo" de Claude. Es todo lo que el modelo puede "ver" en un momento dado. En Claude 3+ esta ventana es de aproximadamente **200,000 tokens**.

Un token es aproximadamente 4 caracteres en inglés o 3 en español. Para referencia:
- 200,000 tokens ≈ 150,000 palabras ≈ 600 páginas de libro

Parece mucho, pero en una sesión de trabajo real, este espacio se llena sorprendentemente rápido.

### Qué Ocupa el Contexto

```
┌────────────────────────────────────────────────────────────────┐
│              VENTANA DE CONTEXTO (200k tokens)                 │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ SYSTEM PROMPT                               ~500-2k tok  │  │
│  │ (instrucciones base de Claude Code)                      │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │ CLAUDE.md                                   ~500-2k tok  │  │
│  │ (instrucciones del proyecto)                             │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │ HISTORIAL DE CONVERSACIÓN                 ~variable tok  │  │
│  │ - Tus mensajes anteriores                               │  │
│  │ - Respuestas de Claude                                  │  │
│  │ - Llamadas a herramientas y sus resultados              │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │ ARCHIVOS LEÍDOS (en resultados de herramientas)         │  │
│  │ - Cada archivo Read() añade su contenido al historial   │  │
│  │ - Un archivo de 300 líneas Python ≈ 3,000 tokens        │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │ OUTPUTS DE COMANDOS (en resultados de herramientas)     │  │
│  │ - pytest output completo: 2,000-10,000 tokens           │  │
│  │ - grep results: variable                                │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │ TU MENSAJE ACTUAL                             ~100-500k  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                │
│  ← Todo esto se envía a la API en CADA request               │
└────────────────────────────────────────────────────────────────┘
```

### El Crecimiento del Contexto Durante una Sesión

```
Inicio de sesión:
[SYSTEM][CLAUDE.md]
Total: ~2,000 tokens

Después del primer mensaje y exploración:
[SYSTEM][CLAUDE.md][Tu mensaje][Respuesta][Read(auth.py)=3k][Read(tests.py)=2k]
Total: ~8,000 tokens

Después de 30 minutos de trabajo:
[SYSTEM][CLAUDE.md][10 intercambios][15 archivos leídos][8 outputs de tests]
Total: ~60,000 tokens

Después de 1 hora:
[SYSTEM][CLAUDE.md][25 intercambios][30 archivos leídos][20 outputs]
Total: ~120,000 tokens ← comenzando a ser problemático

Después de 2 horas sin /compact:
Total: ~180,000 tokens ← cerca del límite, calidad degradada
```

### Qué Pasa Cuando el Contexto se Llena

Cuando el contexto se acerca al límite (200k tokens), varios problemas aparecen:

1. **Respuestas más lentas**: La API tarda más en procesar contextos grandes
2. **Costo mayor**: Cada request envía más tokens de entrada
3. **Degradación de calidad**: El modelo "presta menos atención" a información al inicio del contexto (conocido como "lost in the middle")
4. **Eventualmente, error**: Si se supera el límite, la API devuelve un error

### Las Estrategias de Gestión: `/compact` y `/clear`

**`/compact` — Comprimir Manteniendo el Hilo:**

```
ANTES del /compact:
  [Mensaje 1][Respuesta 1][Herramientas 1][Mensaje 2][Respuesta 2]...[Mensaje 20][...]
  Total: 90,000 tokens

QUÉ HACE /compact:
  1. Envía todo el historial a Claude con instrucción:
     "Resume esta conversación manteniendo: tareas completadas,
      estado actual, pendientes, y contexto técnico clave."
  2. Claude genera un resumen conciso
  3. El historial se reemplaza por ese resumen

DESPUÉS del /compact:
  [Resumen comprimido: "Hemos migrado auth.py a OAuth2. Completado:
   X, Y, Z. Pendiente: actualizar endpoints. Contexto: python-jose..."]
  Total: ~3,000 tokens
```

**`/clear` — Borrar Todo:**

```
ANTES del /clear:
  [Mensaje 1][Respuesta 1]...[Mensaje 20]...
  Total: 90,000 tokens

QUÉ HACE /clear:
  Borra TODOS los mensajes del historial.
  CLAUDE.md sigue en disco (se releerá al inicio de la siguiente sesión).
  Los archivos modificados siguen modificados.

DESPUÉS del /clear:
  [vacío]
  Total: ~0 tokens (solo CLAUDE.md en el siguiente arranque)
```

**La diferencia clave:**

```
/compact = resumir          /clear = borrar
  ↓                           ↓
Contexto con esencia        Contexto vacío
Claude recuerda la tarea    Claude no recuerda nada
Útil: misma tarea continúa  Útil: nueva tarea diferente
```

---

## 6. EL SISTEMA MULTI-AGENTE EN PROFUNDIDAD

### La Arquitectura Técnica

```
┌──────────────────────────────────────────────────────────────┐
│                    AGENTE PRINCIPAL                          │
│                                                              │
│  Contexto:                                                   │
│  [CLAUDE.md + historial completo + tu prompt]                │
│                                                              │
│  Decide crear subagentes:                                    │
│  tool: Agent                                                 │
│  input: {                                                    │
│    "task": "Documentar src/auth/ con docstrings",           │
│    "context": "Proyecto FastAPI, convenciones del proyecto",│
│    "tools": ["Read", "Write"]                               │
│  }                                                           │
└──────────────────────────────┬───────────────────────────────┘
                               │ Crea instancia nueva
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                    SUBAGENTE 1                               │
│                                                              │
│  Contexto:                                                   │
│  [instrucciones del principal + tarea específica]            │
│                                                              │
│  SU PROPIO bucle ReAct:                                      │
│  Read(src/auth/service.py) → analiza → Write(docstrings)     │
│  Read(src/auth/tokens.py) → analiza → Write(docstrings)      │
│  ...                                                         │
│                                                              │
│  NO SABE qué hacen otros subagentes.                         │
│  NO COMPARTE contexto con nadie más.                         │
│  Cuando termina → devuelve resultado al principal.           │
└──────────────────────────────────────────────────────────────┘
```

### Por Qué los Contextos Separados son una Ventaja

```
PROBLEMA DEL CONTEXTO COMPARTIDO:

  Agente único documentando 50 archivos:
  
  Iteración 1: Read(módulo_1.py) → contexto: +3k tokens
  Iteración 2: Read(módulo_2.py) → contexto: +3k tokens
  ...
  Iteración 20: Read(módulo_20.py) → contexto: +3k tokens
  
  Total del contexto: 60k tokens solo de archivos leídos.
  Más el historial de todas las conversaciones anteriores.
  El modelo cada vez "presta menos atención" a los primeros archivos.
  La calidad de documentación para el módulo_20 es peor que para el módulo_1.

SOLUCIÓN CON SUBAGENTES:

  Subagente A: solo ve módulos 1-10  → contexto máximo: 30k tokens
  Subagente B: solo ve módulos 11-20 → contexto máximo: 30k tokens
  Subagente C: solo ve módulos 21-30 → contexto máximo: 30k tokens
  
  Cada subagente trabaja con contexto fresco y acotado.
  La calidad es consistente para todos los módulos.
```

### El Modelo de Ejecución: Paralelo vs. Secuencial

Los subagentes pueden ejecutarse en paralelo o en secuencia dependiendo de las dependencias:

```
PARALELO (cuando las tareas son independientes):

  Principal ──┬──► Subagente A (trabaja en módulo auth/)
              ├──► Subagente B (trabaja en módulo api/)    ← al mismo tiempo
              └──► Subagente C (trabaja en módulo db/)
  
  Principal espera que TODOS terminen, luego integra.
  Tiempo total ≈ tiempo del subagente más lento (no la suma de todos)

SECUENCIAL (cuando hay dependencias):

  Principal ──► Subagente A ──resultado──► Principal ──► Subagente B
  
  El principal espera A, usa su resultado para configurar B.
  Tiempo total = tiempo de A + tiempo de B
```

### Comunicación entre el Principal y los Subagentes

El agente principal puede pasar información a los subagentes en el momento de crearlos, pero los subagentes **no pueden comunicarse entre sí ni con el principal una vez que están trabajando**:

```
Correcto: Principal pasa contexto al crear el subagente
  ✓ "Subagente: documenta src/auth/ siguiendo el estilo
      del proyecto (usa las convenciones del CLAUDE.md)"

Incorrecto: Subagente A comunica a subagente B en tiempo real
  ✗ No es posible. Los subagentes son instancias aisladas.

Si necesitas que un subagente use el resultado de otro:
  → El principal espera a que el primero termine
  → Extrae la información relevante del resultado
  → La pasa como contexto al segundo subagente

Ejemplo:
  Subagente A: "Analiza la arquitectura de src/ y crea un mapa
                de dependencias entre módulos"
  
  Principal recibe el mapa de dependencias.
  
  Subagente B: "Genera tests para src/auth/, teniendo en cuenta
                estas dependencias: [mapa del Subagente A]"
```

---

## 7. MCP: ARQUITECTURA DEL PROTOCOLO

### El Problema que Resuelve MCP

Antes de MCP, si una herramienta de IA quería integrarse con una base de datos, tenía que implementar su propio conector específico. Si una nueva herramienta de IA quería hacer lo mismo, implementaba otro conector diferente. El resultado: un ecosistema fragmentado donde cada herramienta y cada fuente de datos tenían que conectarse de forma ad-hoc.

MCP define un protocolo estándar: **cualquier cliente MCP** (Claude Code, y en el futuro otros) puede conectarse a **cualquier servidor MCP** (PostgreSQL, GitHub, Jira, etc.) usando el mismo protocolo.

### Arquitectura en Detalle

```
┌──────────────────────────────────────────────────────────────┐
│                    CLAUDE CODE (Host MCP)                    │
│                                                              │
│  Al iniciar, lee .mcp.json y lanza los MCP servers          │
│  configurados como procesos hijos.                           │
│                                                              │
│  Para cada servidor MCP:                                     │
│    1. Ejecuta el comando especificado en .mcp.json           │
│    2. Se comunica via stdin/stdout o TCP con el proceso      │
│    3. Pide al servidor la lista de herramientas disponibles  │
│    4. Añade esas herramientas al modelo (junto con Read,      │
│       Write, Bash, etc.)                                     │
│                                                              │
│  Desde la perspectiva del modelo Claude, las herramientas    │
│  MCP son indistinguibles de las nativas.                     │
└───────────────────────────────────────────┬──────────────────┘
                                            │
                 ┌──────────────────────────▼──────────────────┐
                 │                MCP TRANSPORT                │
                 │                                             │
                 │  Protocolo: JSON-RPC 2.0                    │
                 │  Transport: stdio (predeterminado) o SSE    │
                 │  Seguridad: proceso local (no red)          │
                 └───────────────┬─────────────────────────────┘
                                 │
          ┌──────────────────────▼───────────────────────────┐
          │                MCP SERVER                        │
          │         (ej: @modelcontextprotocol/server-postgres)│
          │                                                  │
          │  Expone:                                         │
          │  - Lista de herramientas disponibles             │
          │  - Esquema JSON de los parámetros de cada tool   │
          │  - Implementación de cada herramienta            │
          │                                                  │
          │  Ejemplo (server-postgres):                      │
          │  tools:                                          │
          │    - query(sql: string) → resultados             │
          │    - list_tables() → lista de tablas             │
          │    - describe_table(name: string) → schema       │
          └──────────────────────────────────────────────────┘
```

### El Flujo de una Llamada MCP

```
1. Claude decide que necesita consultar la base de datos:
   
   THOUGHT: "Para entender el bug, necesito ver los datos
             reales en la tabla orders."

2. Claude genera una llamada a la herramienta MCP:
   
   {
     "type": "tool_use",
     "name": "mcp_postgres_query",
     "input": {
       "sql": "SELECT * FROM orders WHERE status='error' LIMIT 5"
     }
   }

3. El CLI recibe la llamada y la redirige al MCP Server:
   
   CLI ──► MCP Client ──► (stdin) ──► postgres-mcp-server
   
   El mensaje es JSON-RPC:
   {
     "jsonrpc": "2.0",
     "method": "tools/call",
     "params": {
       "name": "query",
       "arguments": { "sql": "SELECT * FROM orders..." }
     }
   }

4. El MCP Server ejecuta la query en PostgreSQL y responde:
   
   postgres-mcp-server ──► (stdout) ──► MCP Client ──► CLI
   
   {
     "jsonrpc": "2.0",
     "result": {
       "content": [
         {
           "type": "text",
           "text": "id | status | error_msg | created_at\n...[resultados]..."
         }
       ]
     }
   }

5. El CLI envía el resultado a la API de Anthropic como
   resultado de herramienta, y Claude continúa su análisis.
```

---

## 8. EL SISTEMA DE HOOKS: EVENTOS Y AUTOMATIZACIÓN

### Cuándo se Ejecutan los Hooks en el Flujo

```
FLUJO DE UNA ACCIÓN CON HOOKS:

  Claude decide usar la herramienta Write(src/auth.py, contenido)
                              │
                              ▼
                    ┌─────────────────┐
                    │  PreToolUse     │ ← Hook ejecutado ANTES
                    │  hooks          │   Si retorna error → acción cancelada
                    └────────┬────────┘   Si retorna OK → continúa
                             │
                             ▼
                    ┌─────────────────┐
                    │  Tool Executor  │
                    │  Write(archivo) │ ← La acción real
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  PostToolUse    │ ← Hook ejecutado DESPUÉS
                    │  hooks          │   Puede hacer cualquier acción
                    └────────┬────────┘   El resultado no afecta al modelo
                             │
                             ▼
                  Resultado devuelto al modelo
```

### El Formato de Configuración

```json
{
  "hooks": {
    "<EventType>": [
      {
        "matcher": "<ToolName o '*'>",
        "hooks": [
          {
            "type": "command",
            "command": "<comando bash>",
            "timeout": 30000
          }
        ]
      }
    ]
  }
}
```

**Campos del hook:**
- `type`: actualmente solo `"command"` está disponible
- `command`: el comando bash a ejecutar
- `timeout`: tiempo máximo de espera en ms (por defecto 30 segundos)

**Variables disponibles en `command`:**
- `${file}` → ruta completa del archivo afectado (para Write/Read)
- `${command}` → el comando bash ejecutado (para Bash hooks)
- `${tool}` → nombre de la herramienta (`Write`, `Read`, `Bash`, etc.)

**El `matcher`:**
- `"Write"` → solo para operaciones Write
- `"Read"` → solo para operaciones Read
- `"Bash"` → solo para operaciones Bash
- `"*"` → para cualquier herramienta

### Lógica de los Hooks PreToolUse

Los hooks `PreToolUse` pueden **cancelar** la acción de Claude si el script retorna un código de error (exit code != 0):

```bash
# Este hook en PreToolUse cancelará la operación Write
# si el archivo es uno de los archivos protegidos

#!/bin/bash
PROTECTED_PATHS="config/production.yml .env.prod alembic/versions/"
for path in $PROTECTED_PATHS; do
  if echo "${file}" | grep -q "$path"; then
    echo "ERROR: ${file} es un archivo protegido."
    echo "Requiere aprobación manual antes de modificar."
    exit 1  # Código de error → Claude Code cancela la acción Write
  fi
done
exit 0  # OK → Claude Code procede con la acción Write
```

Si el hook cancela la acción, Claude recibe un mensaje de error como resultado de herramienta y puede actuar en consecuencia (intentar un enfoque diferente, o comunicarle al usuario que la acción fue bloqueada).

---

## 9. SEGURIDAD: MODELO DE PERMISOS Y CONFIANZA

### Las Tres Capas de Seguridad

```
CAPA 1: PERMISOS DEL SISTEMA OPERATIVO
  Claude Code solo puede hacer lo que tu usuario del SO puede hacer.
  Si tu usuario no puede borrar /etc/passwd, Claude tampoco puede.
  Esta es la capa más fundamental y no configurable.

CAPA 2: CONFIRMACIONES DE CLAUDE CODE
  Claude Code implementa un sistema de confirmaciones para
  operaciones que considera potencialmente peligrosas.
  Esta capa es configurable (--dangerously-skip-permissions).

CAPA 3: HOOKS PreToolUse
  Puedes implementar lógica custom que bloquea operaciones
  específicas basándote en cualquier criterio que necesites.
  Esta es la capa más flexible y potente para casos específicos.
```

### Qué Operaciones Requieren Confirmación

Claude Code clasifica las operaciones por riesgo:

```
RIESGO ALTO → Confirmación obligatoria:
  ┌────────────────────────────────────────────────────────┐
  │  Bash:                                                 │
  │    rm, rmdir, unlink (eliminar archivos)               │
  │    sudo (elevación de privilegios)                     │
  │    git push (enviar cambios a remoto)                  │
  │    git reset --hard (potencial pérdida de trabajo)     │
  │    dd, mkfs (operaciones de disco)                     │
  │    cualquier pipe a rm o a operaciones destructivas    │
  └────────────────────────────────────────────────────────┘

RIESGO BAJO → Sin confirmación:
  ┌────────────────────────────────────────────────────────┐
  │  Read (leer archivos) → reversible (no hace nada)      │
  │  Write (escribir archivos) → reversible con git        │
  │  Bash: pytest, jest, etc. → ejecuta tests, no destruye │
  │  Bash: grep, find, ls → solo lee el sistema            │
  │  Bash: git log, git diff, git status → solo lectura    │
  │  Bash: npm install, pip install → instala, no destruye │
  └────────────────────────────────────────────────────────┘
```

### El Flag `--dangerously-skip-permissions`

Este flag desactiva **todas** las confirmaciones de la Capa 2. Es necesario en entornos de CI/CD donde no hay un humano para responder a las confirmaciones.

```
CUÁNDO USARLO (apropiado):
  ✓ En un contenedor Docker desechable de CI/CD
  ✓ En GitHub Actions con un usuario de servicio dedicado
  ✓ En scripts automatizados donde el scope está perfectamente
    acotado con --allowedTools

CUÁNDO NO USARLO (peligroso):
  ✗ En tu máquina local de desarrollo
  ✗ En servidores de producción
  ✗ Cuando no sabes exactamente qué va a hacer Claude
  ✗ En repositorios con archivos críticos sin protección adicional
```

**Si necesitas usarlo en CI/CD, combínalo con `--allowedTools`:**

```bash
# En CI/CD: sin confirmaciones PERO limitando las herramientas disponibles
claude --dangerously-skip-permissions \
       --allowedTools "Read,Bash" \
       --print "Ejecuta los tests y reporta fallos"

# Esto es razonablemente seguro porque:
# - No puede escribir archivos (--allowedTools no incluye Write)
# - No puede crear subagentes (no incluye Agent)
# - Solo puede leer y ejecutar, no modificar el código
```

---

## 10. CICLO DE VIDA COMPLETO DE UNA SESIÓN

### Desde `claude` hasta Cerrar la Sesión

```
FASE 1: INICIALIZACIÓN
─────────────────────────────────────────────────────────────
$ claude

El CLI:
  1. Lee la configuración de ~/.claude/settings.json
  2. Busca y lee CLAUDE.md (global + proyecto + subdirectorio)
  3. Lanza los MCP Servers configurados en .mcp.json (si existe)
  4. Pregunta a cada MCP Server qué herramientas ofrece
  5. Muestra el banner de bienvenida
  6. Abre el prompt >

FASE 2: BUCLE INTERACTIVO
─────────────────────────────────────────────────────────────
Para cada tarea que escribes:

  1. Lees el input del usuario
  2. Construye el payload para la API:
     {
       system: [system prompt + CLAUDE.md],
       messages: [historial completo + tu nuevo mensaje],
       tools: [Read, Write, Bash, WebSearch, Agent, + MCP tools]
     }
  3. Envía el payload a la API de Anthropic (HTTPS)
  4. Recibe la respuesta:
     - Si es "tool_use" → ejecuta la herramienta, vuelve al paso 2
     - Si es "end_turn" → muestra la respuesta al usuario, espera siguiente input
  5. Ejecuta hooks correspondientes (PreToolUse, PostToolUse)
  6. Actualiza el Context Manager con los nuevos mensajes y resultados

FASE 3: COMANDOS SLASH
─────────────────────────────────────────────────────────────
Los comandos /slash se procesan LOCALMENTE, no se envían a la API:

  /clear   → Context Manager borra el historial en memoria
  /compact → Envía una request especial a la API para resumir,
             luego reemplaza el historial con el resumen
  /cost    → Context Manager cuenta los tokens y los muestra
  /model   → Cambia el modelo en la configuración local

FASE 4: CIERRE
─────────────────────────────────────────────────────────────
Ctrl+C o Ctrl+D:
  - Ejecuta hooks de Stop si están configurados
  - Cierra conexiones con MCP Servers
  - La sesión (historial en memoria) se pierde
  - CLAUDE.md y los cambios en archivos persisten en disco
```

---

## 11. PERFORMANCE Y OPTIMIZACIÓN TÉCNICA

### Factores que Afectan la Velocidad

```
VELOCIDAD DE RESPUESTA DE CLAUDE CODE:
Más lento ◄────────────────────────────────► Más rápido

Modelo:    opus          sonnet          haiku
           (máx calidad) (equilibrado)   (máx velocidad)

Contexto:  200k tokens   100k tokens     10k tokens
           (lleno)       (medio)         (pequeño)

Archivos:  muchos y      pocos y         ninguno
leídos:    grandes       medianos        (solo historial)

Complejidad: tarea muy   tarea           tarea simple
tarea:       compleja    mediana

Latencia red: pobre       normal          buena
              (VPN, etc.) 
```

### Cuándo Cada Estrategia de Optimización es Correcta

```
ESTRATEGIA 1: Elegir el modelo correcto
  
  Haiku es correcto cuando:
    - La respuesta es simple y directa
    - No requiere razonamiento complejo
    - La velocidad importa más que la calidad máxima
    
  Sonnet es correcto para el 80% de los casos:
    - Refactoring, debugging, generación de tests
    - El balance calidad/costo es óptimo para trabajo diario
    
  Opus es correcto cuando:
    - La tarea es genuinamente compleja: arquitectura de sistemas,
      análisis de código profundo, refactoring de alto riesgo
    - La calidad del output justifica el costo adicional

ESTRATEGIA 2: Gestionar el contexto activamente
  
  Usar /compact cuando:
    - /cost muestra >80k tokens
    - La sesión lleva >1 hora de trabajo continuo
    - Las respuestas se sienten "lentas" o "dispersas"
    - Quieres continuar la misma tarea
  
  Usar /clear cuando:
    - Cambias completamente de tarea
    - Quieres que Claude "olvide" una dirección errónea que tomó
    - Inicio de nueva jornada de trabajo

ESTRATEGIA 3: Ser específico en los prompts
  
  "Analiza el proyecto" vs "Analiza src/auth/service.py"
    El primero: Claude leerá muchos archivos → muchos tokens
    El segundo: Claude lee un archivo → pocos tokens
  
  Cuanto más específico seas con las rutas y el scope,
  menos exploración hace Claude y menos tokens consume.

ESTRATEGIA 4: Usar subagentes para repos grandes
  
  Paradoja: Los subagentes a veces son MÁS BARATOS que trabajar
  en serie con un solo agente, porque:
  
  Sin subagentes: contexto crece hasta 150k tokens (mucho overhead)
  Con subagentes: 3 agentes × 50k tokens cada uno = 150k tokens total
  
  Pero la calidad de los subagentes es mejor (contextos limpios)
  y el tiempo real es menor (trabajan en paralelo).
```

### Estimación de Costos por Tipo de Tarea

```
Tarea                          Modelo      Tokens aprox.   Costo aprox.
─────────────────────────────────────────────────────────────────────────
Pregunta rápida                haiku       1,000 tok       $0.001
Explorar un archivo            haiku       5,000 tok       $0.005
Refactorizar 1 archivo         sonnet     20,000 tok       $0.10
Generar tests para 1 clase     sonnet     30,000 tok       $0.15
Debuggear un bug con logs      sonnet     40,000 tok       $0.20
Migrar 1 módulo a nuevo fw     sonnet     60,000 tok       $0.30
Documentar 10 módulos          sonnet    100,000 tok       $0.50
Análisis de arquitectura       opus       50,000 tok       $1.50
Migración completa de fw       opus      200,000 tok       $6.00
─────────────────────────────────────────────────────────────────────────
Precios orientativos para marzo 2026. Verificar en anthropic.com/pricing
```

---

> 📌 Este documento explica el funcionamiento interno de Claude Code. Los detalles de implementación pueden cambiar entre versiones. Para la referencia más actual, consulta https://docs.anthropic.com/claude-code. Fecha de referencia: Marzo 2026.

