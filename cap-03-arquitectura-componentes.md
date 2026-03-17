# Capítulo 3: Arquitectura de Componentes

> **Nivel:** Sin conocimientos previos requeridos  
> **Prerequisito recomendado:** [Cap. 1](./cap-01-que-es-claude-code.md) y [Cap. 2](./cap-02-patron-react.md)  
> **Tiempo de lectura estimado:** 50-65 minutos  
> **Objetivo:** Entender qué módulos internos forman Claude Code, qué responsabilidad tiene cada uno, cómo se comunican entre sí y cómo colaboran para transformar tu instrucción en texto en una serie de acciones reales sobre tu sistema.

---

## Tabla de Contenidos

- [3.1 Por qué importa entender la arquitectura](#31-por-qué-importa-entender-la-arquitectura)
- [3.2 Vista general del sistema: los seis componentes](#32-vista-general-del-sistema-los-seis-componentes)
- [3.3 El REPL: la interfaz de usuario desde dentro](#33-el-repl-la-interfaz-de-usuario-desde-dentro)
- [3.4 El Agente Principal: el director de orquesta](#34-el-agente-principal-el-director-de-orquesta)
- [3.5 El Context Manager: la memoria de trabajo](#35-el-context-manager-la-memoria-de-trabajo)
- [3.6 El Tool Executor: del JSON a la acción real](#36-el-tool-executor-del-json-a-la-acción-real)
- [3.7 El MCP Client: extensibilidad por diseño](#37-el-mcp-client-extensibilidad-por-diseño)
- [3.8 La comunicación con la API de Anthropic](#38-la-comunicación-con-la-api-de-anthropic)
- [3.9 El sistema de archivos de configuración](#39-el-sistema-de-archivos-de-configuración)
- [3.10 Cómo fluye la información: trazas completas](#310-cómo-fluye-la-información-trazas-completas)
- [3.11 Fallos comunes y cómo diagnosticarlos](#311-fallos-comunes-y-cómo-diagnosticarlos)
- [3.12 Resumen y glosario del capítulo](#312-resumen-y-glosario-del-capítulo)

---

## 3.1 Por qué importa entender la arquitectura

### No necesitas saber cómo funciona el motor para conducir un coche... pero ayuda

Puedes usar Claude Code sin entender su arquitectura interna. Sin embargo, hay muchas situaciones en las que entender cómo está construido marca la diferencia entre resolver un problema en dos minutos o perder una hora:

- **Cuando Claude "olvida" información:** Si entiendes el Context Manager, sabes exactamente por qué ocurre y cómo solucionarlo.
- **Cuando una tarea es muy lenta:** Si entiendes el ciclo de peticiones, puedes optimizar para reducir el número de viajes a la API.
- **Cuando el contexto se llena:** Si entiendes cómo se acumula el contexto, puedes gestionar sesiones largas de forma proactiva.
- **Cuando quieres extender Claude con MCP:** El MCP Client tiene una arquitectura muy específica que necesitas entender para configurarlo correctamente.
- **Cuando hay un error y no sabes de dónde viene:** Saber qué componente maneja cada operación te permite diagnosticar rápidamente.

### La arquitectura como mapa mental

Piensa en este capítulo como el plano de un edificio. No necesitas conocer cada tubo y cada cable, pero saber dónde están las paredes principales, dónde está la cocina y dónde está el cuadro eléctrico te hace capaz de vivir en él con mucha más autonomía.

Al final de este capítulo, tendrás un modelo mental claro de **qué ocurre internamente desde el momento en que presionas Enter hasta que Claude devuelve su respuesta**, con todos los pasos intermedios.

---

## 3.2 Vista general del sistema: los seis componentes

### La analogía de la orquesta

Antes de ver cada componente en detalle, una analogía ayuda a entender el conjunto como un todo cohesionado:

Imagina que Claude Code es una **orquesta interpretando una partitura** (tu tarea):

- **El REPL** es el escenario y el sistema de sonido: lo que el público (tú) escucha y ve.
- **El Agente Principal** es el director de orquesta: coordina todo, decide el tempo y qué sección toca cuándo.
- **El Context Manager** es el atril con la partitura completa: contiene toda la información de qué se ha tocado ya y qué queda por tocar.
- **El Tool Executor** son los músicos: ejecutan las notas concretas cuando el director lo indica.
- **El MCP Client** son los músicos invitados de otras orquestas: traen capacidades especiales que la orquesta no tiene por sí sola.
- **La API de Anthropic** es el compositor remoto que envía las instrucciones de qué tocar a continuación, basándose en todo lo que ha escuchado.

### El diagrama de arquitectura completo

```
════════════════════════════════════════════════════════════════
TU COMPUTADORA (proceso local)
════════════════════════════════════════════════════════════════
┌───────────────────────────────────────────────────────────────┐
│  CLAUDE CODE CLI                                              │
│                                                               │
│  ┌─────────────┐       ┌─────────────────────────────────┐   │
│  │    REPL     │◄─────►│       AGENTE PRINCIPAL          │   │
│  │             │       │                                 │   │
│  │  > prompt   │       │  ┌─────────────────────────┐   │   │
│  │  Streaming  │       │  │   Bucle ReAct           │   │   │
│  │  /commands  │       │  │   Thought→Action→Obs.   │   │   │
│  └─────────────┘       │  └──────────────┬──────────┘   │   │
│                        │                 │              │   │
│                        │  ┌──────────────▼──────────┐   │   │
│                        │  │    Context Manager      │   │   │
│                        │  │  • Historial sesión     │   │   │
│                        │  │  • CLAUDE.md cargado    │   │   │
│                        │  │  • Resultados tools     │   │   │
│                        │  │  • Contador de tokens   │   │   │
│                        │  └──────────────┬──────────┘   │   │
│                        │                 │              │   │
│  ┌─────────────┐       │  ┌──────────────▼──────────┐   │   │
│  │  MCP CLIENT │◄─────►│  │    Tool Executor        │   │   │
│  │             │       │  │  • Read / Write / Edit  │   │   │
│  │  Gestiona   │       │  │  • Bash (subprocess)   │   │   │
│  │  procesos   │       │  │  • WebSearch           │   │   │
│  │  MCP hijos  │       │  │  • Agent (subagente)   │   │   │
│  └──────┬──────┘       │  │  • MCP tools           │   │   │
│         │              │  └─────────────────────────┘   │   │
└─────────┼──────────────┴──────────────────────────────────┘   │
          │                                                       │
    ┌─────▼──────────────────┐   ┌──────────────────────────┐   │
    │     MCP SERVERS        │   │     TU PROYECTO          │   │
    │  (procesos separados)  │   │                          │   │
    │  npx @mcp/postgres     │   │  src/    tests/          │   │
    │  npx @mcp/github       │   │  CLAUDE.md  .mcp.json    │   │
    │  npx @mcp/filesystem   │   │  .claude/settings.json   │   │
    └────────────────────────┘   └──────────────────────────┘
                                          │
                        ┌─────────────────▼────────────────┐
                        │    ANTHROPIC API (NUBE)          │
                        │    api.anthropic.com/v1/messages │
                        │    claude-opus/sonnet/haiku      │
                        │    HTTPS TLS 1.3 + SSE streaming │
                        └──────────────────────────────────┘
```

### Por qué cada componente existe

Cada componente resuelve un problema específico:

| Componente | Problema que resuelve |
|------------|----------------------|
| **REPL** | Necesitamos una interfaz interactiva donde el usuario escribe y ve resultados |
| **Agente Principal** | Alguien tiene que orquestar el ciclo Thought→Action→Observation |
| **Context Manager** | Los LLMs no tienen estado; hay que llevar el historial con cada petición |
| **Tool Executor** | Las decisiones del LLM (JSON) tienen que convertirse en acciones del SO |
| **MCP Client** | Las herramientas nativas no cubren todos los casos; necesitamos extensibilidad |
| **API de Anthropic** | La inteligencia real (el LLM) vive en la nube; necesitamos un canal de comunicación |

---

## 3.3 El REPL: la interfaz de usuario desde dentro

### Qué es un REPL y por qué Claude Code usa uno

**REPL** son las siglas de **Read-Eval-Print Loop** (Leer-Evaluar-Imprimir-Repetir). Es el patrón de interfaz de los lenguajes de programación interactivos (Python, Node.js, Ruby tienen todos su REPL). La idea es simple: lees una entrada, la evalúas, imprimes el resultado, y vuelves a leer.

Claude Code adoptó este patrón porque encaja perfectamente con el modelo de trabajo: writes una instrucción, Claude la "evalúa" (ejecuta el ciclo ReAct), imprime el resultado, y vuelves a escribir.

### Lo que ves cuando ejecutas `claude`

```
╭─────────────────────────────────────────────────────────────╮
│  ✻ Welcome to Claude Code research preview!                 │
│    /help for help, /status for token count                  │
╰─────────────────────────────────────────────────────────────╯

> _    ← aquí el cursor espera tu input
```

### El ciclo interno del REPL en detalle

```
CICLO REPL
═══════════════════════════════════════════════════════════════

READ (Leer):
  • Espera en el prompt >
  • Acepta tu texto hasta que presionas Enter
  • Para mensajes multilínea: Shift+Enter añade nueva línea
  • Detecta si el input empieza con / (comando slash)

EVAL (Evaluar):
  • Si es comando /slash → lo procesa localmente, sin llamar a la API
  • Si es mensaje normal → lo pasa al Agente Principal
  • El Agente ejecuta el ciclo completo (potencialmente muchos pasos)
  • Durante la ejecución, el REPL muestra progreso en tiempo real:
    ⊕ Reading src/auth.py...
    ⊕ Executing: pytest tests/ -v
    ⊕ Writing src/auth.py...

PRINT (Imprimir):
  • Muestra el streaming de la respuesta token a token
  • Cuando termina, añade salto de línea y estadísticas
  • Muestra: "Cost: ~$0.003 | Tokens: 4,521 in / 287 out"

LOOP (Repetir):
  • Vuelve al prompt >
  • La sesión persiste hasta Ctrl+D o /exit
```

### Los comandos /slash: operaciones locales sin costo

Los comandos que empiezan con `/` son procesados **completamente por el REPL**, sin enviar ninguna petición a la API de Anthropic (con una excepción: `/compact`):

| Comando | Dónde se procesa | Qué hace | ¿Consume tokens? |
|---------|-----------------|----------|-----------------|
| `/help` | Local (REPL) | Muestra lista de comandos | No |
| `/clear` | Local (REPL) | Borra historial de RAM | No |
| `/status` | Local (REPL) | Muestra tokens usados en la sesión | No |
| `/model` | Local (REPL) | Muestra o cambia el modelo activo | No |
| `/cost` | Local (REPL) | Desglose detallado de costos | No |
| `/compact` | API | Envía petición para resumir el historial | Sí (una petición) |
| `/add-dir` | Local (REPL) | Añade un directorio al workspace | No |
| `/exit` | Local (REPL) | Cierra Claude Code | No |

### La visualización de progreso en tiempo real

Una característica importante del REPL es que muestra cada acción de Claude mientras ocurre, no al final. Esto sirve para:

1. **Verificar que Claude está en la dirección correcta** sin esperar a que termine.
2. **Detectar si Claude está explorando archivos inesperados** (señal de que el prompt fue ambiguo).
3. **Interrumpir con Ctrl+C** si Claude está haciendo algo que no querías.

```
EJEMPLO DE VISUALIZACIÓN DE PROGRESO:
═══════════════════════════════════════════════════════════════

> Migra auth.py de MD5 a bcrypt

⊕ Reading src/auth.py...                    ← acción 1
⊕ Searching: grep -r "hash_password" .      ← acción 2
⊕ Reading tests/test_auth.py...             ← acción 3
⊕ Reading requirements.txt...              ← acción 4
⊕ Writing src/auth.py...                   ← acción 5
⊕ Writing tests/test_auth.py...            ← acción 6
⊕ Writing requirements.txt...             ← acción 7
⊕ Executing: pip install bcrypt...         ← acción 8
⊕ Executing: pytest tests/test_auth.py -v  ← acción 9

He completado la migración a bcrypt. Todos los tests pasan (5/5).
Los cambios fueron:
- src/auth.py: reemplazado hashlib.md5 por bcrypt.hashpw/checkpw
- tests/test_auth.py: actualizado para verificar formato bcrypt
- requirements.txt: añadido bcrypt==4.1.2

> _    ← de vuelta al prompt
```

### El modo no interactivo del REPL

Además del modo interactivo, el REPL soporta un modo "headless" para scripts y automatización:

```bash
# Modo -p (print): ejecuta una tarea y sale
claude -p "Genera el changelog de los últimos 10 commits"
# Claude ejecuta la tarea, imprime el resultado, y el proceso termina

# Útil en CI/CD, scripts, o pipelines automatizados
```

---

## 3.4 El Agente Principal: el director de orquesta

### La responsabilidad del Agente Principal

El Agente Principal es el módulo central que orquesta todo el flujo de trabajo. Sus responsabilidades son:

1. Recibir la tarea del REPL.
2. Coordinar con el Context Manager para construir el payload.
3. Enviar la petición a la API de Anthropic.
4. Recibir y analizar la respuesta.
5. Si la respuesta pide una herramienta: delegarla al Tool Executor.
6. Si la respuesta es final: devolver el resultado al REPL.
7. Repetir el ciclo hasta que la tarea esté completa.

Lo que el Agente Principal **no hace** es decidir qué acción tomar. Esa decisión la toma el LLM en Anthropic. El Agente Principal simplemente ejecuta lo que el LLM pide y continúa el ciclo.

### El bucle central en pseudocódigo detallado

Aunque el código real está en TypeScript y es más complejo, la lógica esencial del bucle es:

```python
def agent_loop(user_message, context_manager, tool_executor, max_iterations=50):
    """
    Implementa el ciclo ReAct principal.
    
    max_iterations: límite de seguridad para evitar bucles infinitos.
    """
    context_manager.add_user_message(user_message)
    iterations = 0
    
    while iterations < max_iterations:
        iterations += 1
        
        # 1. Construir el payload con TODO el historial actual
        payload = {
            "model": config.model,
            "max_tokens": config.max_tokens,
            "system": context_manager.get_system_prompt(),  # CLAUDE.md + instrucciones base
            "messages": context_manager.get_history(),      # historial completo
            "tools": tool_executor.get_tool_definitions()   # definiciones JSON de herramientas
        }
        
        # 2. Enviar a Anthropic (petición HTTP)
        response = anthropic_api.post("/v1/messages", payload)
        
        # 3. Añadir la respuesta del asistente al historial
        context_manager.add_assistant_response(response.content)
        
        # 4. Decidir qué hacer según stop_reason
        if response.stop_reason == "end_turn":
            # El LLM ha terminado — devolver respuesta al REPL
            return response.content
        
        elif response.stop_reason == "tool_use":
            # El LLM quiere ejecutar una o más herramientas
            tool_calls = response.get_tool_calls()
            
            for tool_call in tool_calls:
                # Mostrar al usuario qué está haciendo Claude
                repl.show_progress(f"⊕ {tool_call.name}: {tool_call.input}")
                
                # Ejecutar la herramienta (puede requerir confirmación)
                tool_result = tool_executor.execute(
                    name=tool_call.name,
                    params=tool_call.input
                )
                
                # Añadir el resultado al historial
                context_manager.add_tool_result(
                    tool_use_id=tool_call.id,
                    result=tool_result,
                    is_error=tool_result.is_error
                )
            
            # Continuar el bucle con el historial actualizado
            continue
        
        elif response.stop_reason == "max_tokens":
            # Claude agotó los tokens de output — respuesta truncada
            context_manager.add_warning("Respuesta truncada por límite de tokens")
            return response.content
    
    # Se alcanzó el límite de iteraciones
    return "Se alcanzó el límite máximo de iteraciones. Tarea incompleta."
```

### Por qué la "inteligencia" no está en el bucle

Una observación crucial: el bucle del Agente Principal es **mecánicamente simple**. No hay lógica de planificación, no hay algoritmos complejos de toma de decisiones. El bucle simplemente:
- Envía lo que tiene al LLM
- Ejecuta lo que el LLM pide
- Repite

Toda la "inteligencia" — decidir qué herramienta usar, cómo interpretar los resultados, cuándo está completa la tarea — vive en el modelo de Anthropic. El Agente Principal es el "ejecutor", no el "pensador".

Esta separación de responsabilidades es un diseño inteligente: significa que mejorar las capacidades de Claude Code no requiere cambiar el código del agente, solo mejorar el modelo de Anthropic.

### El límite de iteraciones: protección contra bucles infinitos

El parámetro `max_iterations` existe por seguridad. Sin él, un modelo que entrara en un ciclo (intentando algo que siempre falla y volviendo a intentarlo) podría ejecutarse indefinidamente, consumiendo tokens y dinero.

En la práctica, las tareas normales raramente necesitan más de 20-30 iteraciones. Tareas de gran escala (migrar un módulo completo, refactorizar un sistema) pueden necesitar 50-100.

---

## 3.5 El Context Manager: la memoria de trabajo

### El problema fundamental que resuelve

Los LLMs son intrínsecamente **stateless** (sin estado). Cada petición HTTP a la API es completamente independiente de la anterior. Desde la perspectiva del servidor de Anthropic, cada vez que recibes una petición es como si fuera la primera vez que hablas con ese usuario.

Esta es una limitación fundamental de la arquitectura de los LLMs: el modelo no tiene memoria entre llamadas. El Context Manager resuelve este problema de una forma elegante aunque costosa en tokens: **envía el historial completo con cada petición**.

### Cómo funciona: el historial como ilusión de memoria

```
SESIÓN CON 3 INTERCAMBIOS — LO QUE VIAJA EN CADA PETICIÓN
═══════════════════════════════════════════════════════════════

PETICIÓN 1 (primer mensaje):
  messages: [
    { role: "user", content: "Lee src/auth.py" }
  ]
  Tokens enviados: ~50

PETICIÓN 2 (segundo mensaje):
  messages: [
    { role: "user",      content: "Lee src/auth.py" },
    { role: "assistant", content: [Read tool + resultado] },
    { role: "user",      content: [tool_result: contenido auth.py] },
    { role: "assistant", content: "He leído el archivo. Usa MD5..." },
    { role: "user",      content: "Cambia MD5 por bcrypt" }  ← nuevo
  ]
  Tokens enviados: ~3,000 (incluye el archivo leído)

PETICIÓN 3 (tercer mensaje):
  messages: [
    ... todo lo anterior ...
    { role: "assistant", content: [Write tool] },
    { role: "user",      content: [tool_result: "archivo escrito"] },
    { role: "assistant", content: "He actualizado auth.py" },
    { role: "user",      content: "Ejecuta los tests" }  ← nuevo
  ]
  Tokens enviados: ~6,000 (todo el historial acumulado)

El LLM "recuerda" porque TODO el historial se envía con cada petición.
No es memoria real: es un historial que viaja de vuelta a la API en cada llamada.
```

### La estructura interna del payload que construye el Context Manager

El Context Manager construye un objeto JSON muy específico que la API de Anthropic espera recibir:

```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 8192,
  "system": "Eres Claude Code, un asistente de programación...\n\n# Contenido de CLAUDE.md\n## Mi Proyecto\n...",
  "messages": [
    {
      "role": "user",
      "content": "Migra auth.py de MD5 a bcrypt"
    },
    {
      "role": "assistant",
      "content": [
        {
          "type": "text",
          "text": "Voy a leer el archivo auth.py primero."
        },
        {
          "type": "tool_use",
          "id": "toolu_01abc",
          "name": "Read",
          "input": { "path": "src/auth.py" }
        }
      ]
    },
    {
      "role": "user",
      "content": [
        {
          "type": "tool_result",
          "tool_use_id": "toolu_01abc",
          "content": "import hashlib\n\ndef hash_password(p):\n    return hashlib.md5(p.encode()).hexdigest()"
        }
      ]
    }
  ],
  "tools": [
    { "name": "Read", "description": "...", "input_schema": {...} },
    { "name": "Write", "description": "...", "input_schema": {...} }
  ]
}
```

### Cómo el Context Manager carga CLAUDE.md

Al iniciar Claude Code, el Context Manager busca y carga automáticamente los archivos `CLAUDE.md` en un orden específico, de más genérico a más específico:

```
ORDEN DE CARGA DE CLAUDE.md
═══════════════════════════════════════════════════════════════

1. ~/.claude/CLAUDE.md
   Instrucciones globales que aplican a todos tus proyectos.
   Ejemplo: "Siempre usa inglés para los nombres de variables",
   "Prefiero código funcional sobre orientado a objetos"

2. {raíz del proyecto}/CLAUDE.md
   Instrucciones específicas de este proyecto.
   Ejemplo: "Este proyecto usa FastAPI y pytest",
   "El servidor de tests es: pytest tests/ -v --cov=src"

3. {subdirectorio actual}/CLAUDE.md (si existe)
   Instrucciones para un módulo específico.
   Ejemplo: En tests/CLAUDE.md: "Siempre usa fixtures de pytest,
   no objetos mock directos"

REGLA DE PRECEDENCIA:
  Si hay conflictos entre CLAUDE.md global y CLAUDE.md del proyecto,
  prevalece el del proyecto (más específico gana).
```

El contenido combinado se añade al campo `system` de cada petición, antes de las instrucciones base de Claude Code.

### El contador de tokens: monitoreo en tiempo real

El Context Manager mantiene un conteo actualizado de tokens en todo momento. Puedes verlo con `/status`:

```
SALIDA DE /status
═══════════════════════════════════════════════════════════════

Sesión actual:
  Modelo:           claude-sonnet-4-20250514
  Tokens input:     45,231  (en el contexto actual)
  Tokens output:    2,847   (generados en esta sesión)
  Ventana máxima:   200,000 tokens
  Uso del contexto: 22.6% de la ventana disponible
  Costo estimado:   ~$0.047
```

### La compresión de contexto: qué hace /compact

Cuando el contexto se acerca al límite, el comando `/compact` activa una operación especial:

```
/compact FUNCIONA ASÍ:
═══════════════════════════════════════════════════════════════

ANTES de /compact:
  Contexto: 150,000 tokens (75% de 200k)
  Historial: 80 intercambios de mensajes

1. Context Manager envía UNA petición especial a la API:
   "Resume el historial de esta conversación en ~2,000 tokens,
    preservando los detalles técnicos más importantes"

2. La API responde con el resumen (~2,000 tokens)

3. Context Manager REEMPLAZA el historial completo por el resumen

DESPUÉS de /compact:
  Contexto: ~5,000 tokens (2.5% de 200k)
  Historial: 1 mensaje de resumen

PÉRDIDA: Detalles específicos de los intercambios anteriores.
GANANCIA: El contexto "renace" con mucho espacio libre.
```

---

## 3.6 El Tool Executor: del JSON a la acción real

### La responsabilidad del Tool Executor

El Tool Executor es el módulo que convierte las decisiones del LLM (expresadas como JSON) en acciones reales que afectan a tu sistema de archivos, tu terminal, y el mundo exterior.

Cuando el LLM dice "ejecuta pytest tests/", el LLM no ejecuta nada directamente. Es el Tool Executor en tu computadora el que recibe esa instrucción, la valida, y lanza el proceso real.

### El flujo de ejecución de una herramienta

```
FLUJO DE EJECUCIÓN DE UNA HERRAMIENTA
═══════════════════════════════════════════════════════════════

1. LLM RESPONDE CON:
   {
     "type": "tool_use",
     "id": "toolu_01",
     "name": "Bash",
     "input": { "command": "pytest tests/test_auth.py -v" }
   }

2. TOOL EXECUTOR RECIBE EL JSON

3. VALIDACIÓN:
   ¿El nombre de herramienta existe? → Sí (Bash)
   ¿Los parámetros son válidos? → Sí (command es string)
   ¿Requiere confirmación del usuario? → ¿Es destructivo? → No (solo ejecuta tests)

4. EJECUCIÓN:
   subprocess.run(
     ["sh", "-c", "pytest tests/test_auth.py -v"],
     cwd="/ruta/al/proyecto",
     capture_output=True,
     timeout=120
   )

5. CAPTURA DE RESULTADO:
   stdout: "========================= test session starts..."
   stderr: ""
   exit_code: 0

6. RETORNO AL CONTEXT MANAGER:
   {
     "type": "tool_result",
     "tool_use_id": "toolu_01",
     "content": "========================= test session starts...\n5 passed in 0.34s",
     "is_error": false
   }
```

### Cómo funciona la herramienta Bash internamente

La herramienta Bash es la más versátil y la que más cuidado requiere entender:

```
EJECUCIÓN DE BASH
═══════════════════════════════════════════════════════════════

PROCESO PADRE: Claude Code CLI (PID: 1234)
    │
    └── PROCESO HIJO: sh -c "pytest tests/ -v" (PID: 1235)
         │
         ├── stdout → CAPTURADO por Claude Code
         ├── stderr → CAPTURADO por Claude Code
         ├── exit code → 0 (éxito) / 1+ (error)
         └── timeout: 120 segundos por defecto (configurable)

Cuando el proceso hijo termina:
  - stdout + stderr se concatenan
  - El resultado completo se añade al contexto como tool_result
  - Claude puede ver EXACTAMENTE lo que vería un humano en la terminal
```

### La no-persistencia del estado del shell

Esta es una trampa importante que confunde a muchos usuarios. El shell **no persiste** su estado entre llamadas Bash:

```bash
# PROBLEMA: Estas dos llamadas Bash son procesos separados

# Llamada 1 a Bash:
cd /tmp && export MY_VAR="hello" && echo "Variable configurada"
# Resultado: "Variable configurada"

# Llamada 2 a Bash (proceso sh NUEVO):
echo $MY_VAR
# Resultado: "" (vacío — MY_VAR no existe en el nuevo proceso)

# TAMBIÉN:
# Llamada 1: cd /tmp
# Llamada 2: pwd
# Resultado: /ruta/original (no /tmp — el cd no persiste)
```

**Solución práctica:** Claude generalmente lo sabe y maneja esto correctamente, pero si necesitas ejecutar múltiples comandos relacionados, es más confiable encadenarlos en una sola llamada:

```bash
# Correcto: todo en una llamada
cd /tmp && ls -la && cat resultado.txt

# En vez de tres llamadas separadas
```

### La herramienta Read: cómo lee archivos el Tool Executor

```
HERRAMIENTA READ
═══════════════════════════════════════════════════════════════

INVOCACIÓN:
  { "name": "Read", "input": { "path": "src/auth.py" } }

PROCESO INTERNO:
  1. Validar que la ruta existe
  2. Verificar que es un archivo (no un directorio)
  3. Verificar que el usuario tiene permisos de lectura
  4. Leer el contenido completo con open(path, 'r', encoding='utf-8')
  5. Si el archivo es binario o tiene caracteres no-UTF8: error

RESULTADO:
  El contenido completo del archivo como string UTF-8.
  Incluye TODOS los caracteres: espacios, tabulaciones, saltos de línea.

LÍMITE:
  No hay límite de tamaño explícito en Read, pero leer archivos
  muy grandes consume muchos tokens. Un archivo de 5,000 líneas
  podría consumir 50,000+ tokens solo en el read.
```

### La herramienta Write vs Edit: cuándo usar cada una

```
WRITE vs EDIT
═══════════════════════════════════════════════════════════════

WRITE — Reemplaza el archivo completo:
  Uso: Crear archivos nuevos o reescritura total
  Ventaja: Simple y directo
  Riesgo: Si el archivo tiene 500 líneas y solo cambia 5,
          se envían las 500 líneas en el payload de la herramienta

EDIT — Reemplaza un fragmento específico:
  Uso: Cambios quirúrgicos en archivos grandes
  Ventaja: Solo se envían las líneas que cambian
  Mecanismo: Especifica el string exacto a reemplazar + el nuevo string
  Riesgo: Si el string a reemplazar aparece múltiples veces en el archivo,
          puede haber ambigüedad (necesita más contexto)

CUÁNDO CLAUDE ELIGE CADA UNA:
  - Archivo < 100 líneas → Write (simple)
  - Cambio < 10% del archivo → Edit (eficiente)
  - Crear archivo nuevo → Write (siempre)
  - Refactoring masivo → Write (más claro)
```

### Las operaciones que requieren confirmación del usuario

El Tool Executor tiene una lista de operaciones que considera potencialmente destructivas y muestra un prompt de confirmación antes de ejecutar:

```
OPERACIONES QUE PIDEN CONFIRMACIÓN:
═══════════════════════════════════════════════════════════════

SIEMPRE piden confirmación:
  • rm -rf (eliminar directorio recursivamente)
  • git push (enviar cambios al repositorio remoto)
  • git reset --hard (descartar cambios)
  • DROP TABLE, DELETE FROM (operaciones destructivas de BD)
  • sudo (escalación de privilegios)

NUNCA piden confirmación (operaciones seguras):
  • cat, ls, find, grep (solo lectura)
  • pytest, jest, go test (ejecutar tests)
  • git status, git diff, git log (git de solo lectura)
  • pip install, npm install (instalar dependencias)
  • black, prettier (formateo de código)
  • Write/Edit en archivos del proyecto (reversible con git)

CONFIGURABLE en settings.json:
  Puedes añadir o quitar comandos de la lista de confirmación
  según las necesidades de tu proyecto.
```

---

## 3.7 El MCP Client: extensibilidad por diseño

### Por qué existe el MCP Client

Las herramientas nativas de Claude Code (Read, Write, Bash, WebSearch) son muy poderosas, pero hay situaciones donde no son suficientes:

- **Necesitas consultar PostgreSQL directamente** para verificar datos antes de hacer cambios en el código que los procesa.
- **Necesitas crear un PR en GitHub** sin abandonar la sesión de Claude.
- **Necesitas leer conversaciones de Slack** para entender el contexto de una decisión de diseño.
- **Necesitas interactuar con una API interna** de tu empresa que no tiene un SDK instalado.

El **MCP Client** resuelve esto permitiendo conectar Claude Code con cualquier fuente de datos o servicio externo a través del **Model Context Protocol (MCP)**.

### La arquitectura del MCP Client en detalle

```
ARQUITECTURA MCP CLIENT
═══════════════════════════════════════════════════════════════

Claude Code (proceso principal)
    │
    ├── MCP Client (módulo interno)
    │       │
    │       ├── Al arrancar: Lee .mcp.json
    │       │
    │       ├── Para cada servidor configurado:
    │       │       │
    │       │       ├── Lanza el proceso hijo (npx @mcp/postgres...)
    │       │       ├── Realiza handshake (protocolo MCP)
    │       │       ├── Solicita lista de herramientas disponibles
    │       │       └── Registra herramientas en Tool Executor
    │       │
    │       └── Durante la sesión:
    │               │
    │               ├── Cuando LLM invoca mcp_postgres_query:
    │               │       └── Envía mensaje al proceso MCP Server
    │               │           y espera el resultado
    │               │
    │               └── Cuando Claude Code cierra:
    │                       └── Termina todos los procesos MCP hijos
    │
    └── MCP SERVERS (procesos hijos separados)
            │
            ├── @mcp/postgres (PID: 5678)
            │       Acepta mensajes JSON vía stdin/stdout
            │       Ejecuta queries en PostgreSQL
            │
            └── @mcp/github (PID: 5679)
                    Acepta mensajes JSON vía stdin/stdout
                    Interactúa con la API de GitHub
```

### Configuración del MCP Client

La configuración se hace en `.mcp.json` en la raíz del proyecto:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://user:password@localhost:5432/mydb"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..."
      }
    },
    "mi-servidor-interno": {
      "command": "python",
      "args": ["/tools/mcp-server-interno.py"],
      "env": {
        "API_BASE_URL": "https://api.interna.empresa.com"
      }
    }
  }
}
```

Después de configurar esto, en la sesión de Claude puedes hacer cosas como:

```
> Consulta la tabla de usuarios en la base de datos y dime cuántos 
  tienen el email verificado vs los que no

Claude ejecuta: mcp_postgres_query("SELECT email_verified, COUNT(*) FROM users GROUP BY email_verified")
```

---

## 3.8 La comunicación con la API de Anthropic

### El protocolo de comunicación

Claude Code se comunica con Anthropic usando tres tecnologías específicas:

**1. HTTPS con TLS 1.3**

Toda la comunicación viaja cifrada. Esto garantiza que:
- Tu código no puede ser interceptado en tránsito
- Las respuestas de Anthropic no pueden ser manipuladas
- La API key no puede ser robada en tránsito

**2. REST API (HTTP POST)**

El endpoint principal es:
```
POST https://api.anthropic.com/v1/messages
Content-Type: application/json
x-api-key: sk-ant-api03-...
anthropic-version: 2023-06-01
```

**3. Server-Sent Events (SSE) para streaming**

En lugar de esperar a que la respuesta completa esté lista, la API envía los tokens a medida que se generan. Cada token llega como un evento SSE:

```
data: {"type":"content_block_delta","delta":{"type":"text_delta","text":"Voy"}}
data: {"type":"content_block_delta","delta":{"type":"text_delta","text":" a"}}
data: {"type":"content_block_delta","delta":{"type":"text_delta","text":" leer"}}
data: {"type":"message_delta","delta":{"stop_reason":"tool_use"}}
data: [DONE]
```

Claude Code lee estos eventos y los muestra en tiempo real en el REPL.

### Latencia: qué factores la controlan

La latencia total de una iteración del ciclo ReAct depende de varios factores:

```
FACTORES DE LATENCIA
═══════════════════════════════════════════════════════════════

LATENCIA DE RED (fija, depende de tu conexión):
  Latencia base a api.anthropic.com: ~50-200ms

TIEMPO DE PROCESAMIENTO DEL LLM (variable):
  Contexto de   5,000 tokens → ~0.5-1s hasta primer token
  Contexto de  50,000 tokens → ~2-4s
  Contexto de 150,000 tokens → ~8-15s
  
  Nota: El tamaño del CONTEXTO impacta más que el tamaño de la RESPUESTA

TIEMPO DE GENERACIÓN (proporcional a la respuesta):
  Cada 100 tokens de output ≈ 0.5-1 segundo adicional
  Respuesta de  200 tokens ≈ 1-2s adicionales
  Respuesta de 2,000 tokens ≈ 10-15s adicionales

DIFERENCIA ENTRE MODELOS:
  Haiku:  ×1.0 (base)
  Sonnet: ×1.5-2.0
  Opus:   ×3.0-5.0
  
EJEMPLO PRÁCTICO:
  Contexto: 30,000 tokens, respuesta: 500 tokens, modelo: Sonnet
  Latencia total estimada: ~3-6 segundos por iteración
```

### El campo usage: cómo se mide el costo

Cada respuesta de la API incluye información precisa de tokens usados:

```json
{
  "id": "msg_01...",
  "stop_reason": "tool_use",
  "usage": {
    "input_tokens": 4521,
    "output_tokens": 287,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 4100
  }
}
```

- **input_tokens:** Tokens en el payload enviado (mensajes + sistema + herramientas)
- **output_tokens:** Tokens generados por el modelo
- **cache_read_input_tokens:** Tokens que se leyeron desde la caché de Anthropic (más baratos)

Esto es lo que acumula el Context Manager para mostrar en `/status` y `/cost`.

---

## 3.9 El sistema de archivos de configuración

### La raíz del workspace: dónde se ejecuta claude

Claude Code opera en el directorio desde el que se ejecuta el comando `claude`. Ese directorio se convierte en la **raíz del workspace** y afecta a todo:

```bash
# Si ejecutas claude aquí:
cd /home/usuario/mi-proyecto
claude

# Entonces:
# - La raíz del workspace es /home/usuario/mi-proyecto
# - CLAUDE.md se busca en /home/usuario/mi-proyecto/CLAUDE.md
# - .mcp.json se busca en /home/usuario/mi-proyecto/.mcp.json
# - Read("src/auth.py") lee /home/usuario/mi-proyecto/src/auth.py
# - Bash("ls") lista /home/usuario/mi-proyecto/
```

### Mapa completo de archivos de configuración

```
ARCHIVOS DE CONFIGURACIÓN
═══════════════════════════════════════════════════════════════

GLOBALES (afectan a todos los proyectos):
  ~/.claude/
  ├── CLAUDE.md               → Instrucciones globales para Claude
  └── settings.json           → Configuración global
      {
        "model": "claude-sonnet-4-20250514",
        "theme": "dark",
        "defaultPermissions": ["read", "write", "bash"],
        "confirmationRequired": ["git push", "rm -rf"]
      }

DEL PROYECTO (solo para este proyecto):
  {raíz del proyecto}/
  ├── CLAUDE.md               → Instrucciones del proyecto para Claude
  ├── .mcp.json               → Servidores MCP del proyecto
  └── .claude/
      ├── settings.json       → Configuración local del proyecto
      │   {
      │     "model": "claude-opus-4-5",   ← override del modelo global
      │     "hooks": { "pre_commit": "./.claude/hooks/check.sh" }
      │   }
      └── hooks/              → Scripts de hooks (ver Capítulo 8)
          ├── pre_tool_use.sh
          └── post_tool_use.sh
```

### El CLAUDE.md bien estructurado: plantilla completa

Un `CLAUDE.md` efectivo tiene secciones específicas:

```markdown
# [Nombre del Proyecto] — Contexto para Claude Code

## 1. Descripción del proyecto
Sistema de [qué hace]. Desarrollado en [stack].
Objetivo principal: [para qué sirve].

## 2. Stack tecnológico
- Lenguaje: Python 3.12 (o Node 20, Go 1.22, etc.)
- Framework: FastAPI (o Express, Gin, etc.)
- Base de datos: PostgreSQL 15
- Testing: pytest con coverage mínimo 80%
- Linting: black + flake8

## 3. Estructura del proyecto
src/
├── auth/           → módulo de autenticación
├── models/         → modelos SQLAlchemy
├── api/            → routers FastAPI
└── utils/          → utilidades compartidas
tests/              → tests con pytest
docs/               → documentación técnica

## 4. Comandos esenciales
- Tests:         pytest tests/ -v --cov=src
- Lint:          black src/ && flake8 src/ --max-line-length=100
- Servidor dev:  uvicorn main:app --reload --port 8000
- Migraciones:   alembic upgrade head

## 5. Convenciones y reglas
- Commits en Conventional Commits: feat:, fix:, docs:, refactor:
- NUNCA hacer commit directamente a main — siempre crear PR
- NUNCA usar MD5 o SHA1 para contraseñas — usar bcrypt
- Los nombres de funciones y variables en inglés
- Los comentarios en español (equipo hispanohablante)

## 6. Estado actual (actualizar con cada sesión importante)
Fecha: 2026-03-15
Trabajando en: módulo de facturación
Pendiente: integrar con Stripe API, tests del módulo de reportes
Bloqueadores: esperando acceso a las credenciales de Stripe staging
```

---

## 3.10 Cómo fluye la información: trazas completas

### Traza de inicio de sesión

Esto es lo que ocurre en los primeros milisegundos cuando ejecutas `claude`:

```
TRAZA: INICIO DE SESIÓN
═══════════════════════════════════════════════════════════════

t=0ms     CLI arranca el proceso principal

t=5ms     Context Manager:
           • Busca ~/.claude/CLAUDE.md → lo carga
           • Busca ./CLAUDE.md → lo carga
           • Inicializa historial vacío

t=10ms    MCP Client:
           • Lee .mcp.json
           • Para cada servidor: lanza proceso hijo con spawn()
           • Realiza handshake JSON-RPC 2.0
           • Solicita lista de tools disponibles
           • Registra tools en Tool Executor

t=50ms    REPL:
           • Muestra pantalla de bienvenida
           • Muestra prompt >
           • Espera input del usuario

t=∞       (esperando al usuario)
```

### Traza de una tarea completa

```
TRAZA: "Encuentra y corrige el bug de seguridad en auth.py"
═══════════════════════════════════════════════════════════════

t=0ms     REPL recibe el input del usuario
           Context Manager: añade mensaje al historial

t=1ms     Agente Principal: construye payload
           { model, system: [CLAUDE.md], messages: [tarea], tools: [...] }
           Tamaño del payload: ~2,500 tokens

t=2ms     HTTP POST → api.anthropic.com/v1/messages

t=300ms   Primer token SSE llega desde Anthropic
          REPL muestra: streaming del text "Voy a revisar..."

t=800ms   LLM solicita tool_use: Read("src/auth.py")
          REPL muestra: "⊕ Reading src/auth.py..."

t=805ms   Tool Executor ejecuta: open("src/auth.py").read()
          (operación local, ~5ms)
          Resultado: contenido del archivo (2,000 tokens)

t=806ms   Context Manager: añade tool_result al historial
          Historial ahora: ~4,500 tokens

t=807ms   HTTP POST → api.anthropic.com (historial actualizado)

t=1,500ms LLM solicita tool_use: Bash("grep -r 'hash_password' .")
          REPL muestra: "⊕ Executing: grep..."
          Tool Executor: subprocess + captura stdout
          Resultado: lista de archivos que usan la función

...       [continúan iteraciones similares para tests, requirements, etc.]

t=12,000ms LLM solicita tool_use: Bash("pytest tests/test_auth.py -v")
            REPL muestra: "⊕ Executing: pytest..."
            Tool Executor: proceso hijo ejecuta pytest (~2,000ms)
            Resultado: "5 passed in 0.34s"

t=14,500ms LLM genera stop_reason: "end_turn"
            Texto final streamed token a token al REPL

t=15,000ms REPL muestra respuesta completa al usuario
            Muestra estadísticas: "Cost: ~$0.032 | 28k tokens"
            Muestra prompt > de nuevo

TOTAL: ~15 segundos, ~9 iteraciones, ~28,000 tokens
```

---

## 3.11 Fallos comunes y cómo diagnosticarlos

### Fallo 1: "Context length exceeded"

```
ERROR: "Your request exceeded the maximum context length (200,000 tokens)"

CAUSA:
  El historial acumulado + todos los archivos leídos + respuesta
  supera el límite de la ventana de contexto.

DIAGNÓSTICO:
  /status → ver cuántos tokens usa la sesión actual

SOLUCIONES:
  1. /compact → comprimir el historial (pierde detalles pero libera espacio)
  2. /clear → reiniciar sesión completamente (pierde todo el contexto)
  3. Dividir la tarea en partes más pequeñas
  4. Evitar leer archivos grandes innecesariamente
```

### Fallo 2: Claude "olvida" información mencionada antes

```
SÍNTOMA:
  Claude contradice algo que "acordó" hace 20 mensajes,
  o no recuerda una restricción que dijiste al inicio.

CAUSA:
  Con contextos muy largos, el LLM puede tener dificultades
  para mantener coherencia entre la información al inicio
  y la información al final del contexto.

SOLUCIONES:
  1. Mencionar de nuevo la restricción en el mensaje actual
  2. Añadir restricciones críticas al CLAUDE.md (persiste siempre)
  3. Usar /compact para "refrescar" el contexto
```

### Fallo 3: MCP Server no responde

```
ERROR: "MCP server 'postgres' failed to start" o timeout en herramienta MCP

DIAGNÓSTICO:
  1. Verificar que el proceso está instalado: npx @modelcontextprotocol/server-postgres --help
  2. Verificar las credenciales en .mcp.json
  3. Verificar que el servicio (PostgreSQL) está corriendo

SOLUCIONES:
  1. Corregir la configuración en .mcp.json
  2. Reiniciar Claude Code (los procesos MCP se relanzarán)
  3. Ejecutar el servidor MCP manualmente para ver el error completo
```

### Fallo 4: Bash no encuentra un comando

```
ERROR: "bash: pytest: command not found"

CAUSA:
  El entorno de shell de Claude Code puede no tener las mismas
  variables de entorno que tu terminal (por ejemplo, el PATH).
  
SOLUCIONES:
  1. Usar rutas absolutas: /home/usuario/.venv/bin/pytest
  2. Activar el entorno virtual explícitamente:
     source .venv/bin/activate && pytest tests/
  3. Configurar el PATH en CLAUDE.md:
     "Siempre activa el entorno virtual con: source .venv/bin/activate"
```

---

## 3.12 Resumen y glosario del capítulo

### Los puntos fundamentales

1. **Seis componentes colaboran** para transformar tu instrucción en acciones: REPL, Agente Principal, Context Manager, Tool Executor, MCP Client, y la API de Anthropic.

2. **El REPL** maneja la interfaz de usuario: lee input, muestra progreso en tiempo real, procesa comandos `/slash` localmente sin costo, y soporta modo no interactivo para automatización.

3. **El Agente Principal** implementa el bucle ReAct: construye payloads, envía a la API, ejecuta herramientas, repite. La inteligencia real está en el LLM, no en el bucle.

4. **El Context Manager** resuelve la naturaleza stateless de los LLMs enviando el historial completo con cada petición. Carga `CLAUDE.md` automáticamente. Mantiene el contador de tokens.

5. **El Tool Executor** convierte JSON del LLM en acciones reales: procesos hijos para Bash, lectura de archivos para Read, escritura para Write/Edit. Las operaciones destructivas requieren confirmación.

6. **El MCP Client** lanza servidores externos como procesos hijos y expone sus herramientas al LLM, extendiendo capacidades más allá de las nativas.

7. **La comunicación** usa HTTPS + SSE para streaming. La latencia depende del tamaño del contexto, el modelo, y la red.

8. **El sistema de archivos** tiene dos capas: global (`~/.claude/`) y por proyecto (`.claude/`). `CLAUDE.md` es la única memoria persistente entre sesiones.

### Glosario del capítulo

| Término | Definición |
|---------|------------|
| **REPL** | Read-Eval-Print Loop. La interfaz interactiva de Claude Code que recibe input y muestra output. |
| **Agente Principal** | Módulo que implementa el bucle ReAct y coordina todos los demás componentes. |
| **Context Manager** | Módulo que mantiene el historial de sesión y construye los payloads JSON para cada petición a la API. |
| **Tool Executor** | Módulo que convierte solicitudes de herramientas (JSON del LLM) en acciones reales del sistema operativo. |
| **MCP Client** | Módulo que lanza y comunica con servidores MCP externos, añadiendo herramientas personalizadas. |
| **Payload** | El objeto JSON completo enviado a la API en cada petición (modelo + sistema + historial + herramientas). |
| **stop_reason** | Campo de la respuesta API: `"tool_use"` (Claude quiere ejecutar una herramienta) o `"end_turn"` (ha terminado). |
| **SSE** | Server-Sent Events. Protocolo para recibir los tokens de respuesta de forma progresiva (streaming). |
| **Stateless** | Sin estado. Los LLMs no tienen memoria entre peticiones; el historial se envía explícitamente en cada llamada. |
| **Workspace** | El directorio del proyecto desde el que se ejecutó `claude`. Raíz de todas las operaciones de archivos. |
| **CLAUDE.md** | Archivo de texto que Claude lee automáticamente al inicio de cada sesión para entender el contexto del proyecto. |
| **/compact** | Comando que envía el historial al LLM para comprimirlo, liberando espacio en la ventana de contexto. |
| **Proceso hijo** | Proceso del SO lanzado por Claude Code para ejecutar comandos Bash o servidores MCP. |
| **Handshake MCP** | El intercambio inicial de mensajes entre el MCP Client y un servidor MCP para establecer la conexión. |

---

## Ver también

- **[Capítulo 2](./cap-02-patron-react.md):** El Patrón ReAct — el ciclo que implementa el Agente Principal.
- **[Capítulo 4](./cap-04-sistema-herramientas.md):** El Sistema de Herramientas — cada herramienta del Tool Executor en profundidad.
- **[Capítulo 5](./cap-05-ventana-contexto.md):** La Ventana de Contexto — cómo gestiona el Context Manager la memoria.
- **[Capítulo 7](./cap-07-mcp-protocolo.md):** MCP — el protocolo completo que usa el MCP Client.
- **[Capítulo 8](./cap-08-sistema-hooks.md):** Hooks — cómo extender el Tool Executor con lógica propia.

---

> 📌 Siguiente capítulo: [Cap. 4 — El Sistema de Herramientas](./cap-04-sistema-herramientas.md)

Imagina que Claude Code es una **orquesta ejecutando una partitura** (tu tarea):

- **El REPL** es el público: escucha y ve el resultado.
- **El Agente Principal** es el director de orquesta: decide qué hacer en cada momento.
- **El Context Manager** es el atril con la partitura: contiene toda la información relevante.
- **El Tool Executor** son los músicos: ejecutan las acciones concretas.
- **El MCP Client** son los músicos invitados: añaden capacidades especiales.
- **La API de Anthropic** es el compositor: decide la siguiente nota basándose en todo lo que ha escuchado.

### El diagrama de arquitectura completo

```
TU MÁQUINA LOCAL
┌────────────────────────────────────────────────────────────┐
│  CLAUDE CODE CLI                                           │
│                                                            │
│  ┌──────────┐       ┌──────────────────────────────────┐   │
│  │  REPL    │◄─────►│       AGENTE PRINCIPAL           │   │
│  │          │       │                                  │   │
│  │ Muestra  │       │  ┌────────────────────────────┐  │   │
│  │ prompt > │       │  │   Planificador ReAct       │  │   │
│  │ Streaming│       │  │  (Thought→Action→Obs.)     │  │   │
│  └──────────┘       │  └────────────┬───────────────┘  │   │
│                     │               │                  │   │
│                     │  ┌────────────▼───────────────┐  │   │
│                     │  │     Context Manager        │  │   │
│                     │  │  Historial + CLAUDE.md     │  │   │
│                     │  │  Tool results + Tokens     │  │   │
│                     │  └────────────┬───────────────┘  │   │
│                     │               │                  │   │
│  ┌──────────┐       │  ┌────────────▼───────────────┐  │   │
│  │  MCP     │◄─────►│  │     Tool Executor          │  │   │
│  │  CLIENT  │       │  │  Read/Write/Edit/Bash       │  │   │
│  └──────┬───┘       │  │  WebSearch/Agent/MCP tools  │  │   │
│         │           │  └────────────────────────────┘  │   │
└─────────┼───────────┴──────────────────────────────────┘   │
          │                                                    │
   ┌──────▼────────────┐    ┌─────────────────────────────┐   │
   │   MCP SERVERS     │    │    TU PROYECTO              │   │
   │  @mcp/postgres    │    │  src/  tests/  CLAUDE.md    │   │
   │  @mcp/github      │    │  .mcp.json  .claude/        │   │
   └───────────────────┘    └─────────────────────────────┘   │
                                       │ HTTPS TLS 1.3         │
                         ┌─────────────▼──────────────────┐   │
                         │     ANTHROPIC API (NUBE)       │   │
                         │  claude-opus/sonnet/haiku      │   │
                         └────────────────────────────────┘   │
```

---

## 3.2 El REPL: la interfaz de usuario

### Qué es un REPL

**REPL** son las siglas de **Read-Eval-Print Loop** (Leer-Evaluar-Imprimir-Repetir). Es la interfaz interactiva que ves cuando ejecutas `claude`:

```
╭────────────────────────────────────────────────────────────╮
│ ✻ Welcome to Claude Code!                                  │
│   /help for help, /status for token count                 │
╰────────────────────────────────────────────────────────────╯

> _
```

El `>` espera tu input. Cuando escribes y presionas Enter, el REPL:
1. Lee tu mensaje completo.
2. Lo pasa al Agente Principal para procesarlo.
3. Muestra progreso en tiempo real: `⊕ Read src/auth.py`, `⊕ Bash: pytest tests/ -v`
4. Muestra el streaming de la respuesta, token a token.
5. Cuando el agente termina, vuelve a mostrar el `>`.

### Los comandos /slash son operaciones locales

Los comandos que empiezan con `/` se procesan **localmente por el REPL** sin enviarlos a la API:

| Comando | Qué hace |
|---------|----------|
| `/clear` | Borra el historial en memoria |
| `/compact` | Envía a la API para resumir, luego reemplaza el historial |
| `/cost` | Cuenta y muestra los tokens usados |
| `/model` | Cambia el modelo en la configuración local |
| `/help` | Muestra la lista de comandos disponibles |
| `/status` | Muestra el estado actual de la sesión |

Ninguno consume tokens extra (excepto `/compact`, que envía una petición de resumen).

---


> 📌 Siguiente capítulo: [Cap. 4 — El Sistema de Herramientas](./cap-04-sistema-herramientas.md)
