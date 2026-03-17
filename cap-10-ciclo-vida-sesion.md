# Capítulo 10: Ciclo de Vida de una Sesión

> **Nivel:** Requiere comprensión de los capítulos anteriores  
> **Prerequisito recomendado:** Capítulos 1-5  
> **Objetivo:** Entender exactamente qué ocurre desde que se ejecuta `claude` hasta que termina la sesión, con todos los estados y transiciones.

---

## Tabla de Contenidos

- [10.1 Las fases de una sesión](#101-las-fases-de-una-sesión)
- [10.2 Fase de inicialización](#102-fase-de-inicialización)
- [10.3 El modo interactivo: el REPL en funcionamiento](#103-el-modo-interactivo)
- [10.4 El modo no interactivo: --print y automatización](#104-modo-no-interactivo)
- [10.5 El modo headless: integraciones y pipelines](#105-modo-headless)
- [10.6 La gestión del historial durante la sesión](#106-gestión-del-historial)
- [10.7 Las transiciones de estado](#107-transiciones-de-estado)
- [10.8 La fase de cierre](#108-la-fase-de-cierre)
- [10.9 Continuación de sesiones: --resume](#109-continuación-de-sesiones)
- [10.10 Sesiones en entornos con IDE](#1010-sesiones-con-ide)
- [10.11 Resumen y glosario del capítulo](#1011-resumen-y-glosario)

---

## 10.1 Las fases de una sesión

Una sesión de Claude Code pasa por las siguientes fases:

```
INICIO
  │
  ▼
┌─────────────────┐
│  INICIALIZACIÓN │  Lee configuración, carga CLAUDE.md, lanza MCP servers
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   MODO ACTIVO   │  Interactivo (REPL) o No interactivo (--print)
│                 │
│  ┌───────────┐  │
│  │  Tarea 1  │  │  Bucle ReAct para cada tarea
│  └───────────┘  │
│  ┌───────────┐  │
│  │  Tarea 2  │  │
│  └───────────┘  │
│       ...       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│     CIERRE      │  Hooks Stop, limpieza, MCP servers terminan
└─────────────────┘
         │
         ▼
FIN
```

Cada fase tiene comportamientos específicos y puntos donde puedes intervenir.

---

## 10.2 Fase de inicialización

### Lo que ocurre al ejecutar `claude`

```bash
$ claude
# O con opciones:
$ claude --model claude-opus-4 --allowedTools "Read,Write,Bash"
```

**1. Parseo de argumentos de línea de comandos:**
```
--model          → Sobreescribe el modelo por defecto
--allowedTools   → Lista de herramientas permitidas
--disallowedTools → Lista de herramientas bloqueadas
--print          → Modo no interactivo
--dangerously-skip-permissions → Desactiva confirmaciones
--resume <id>    → Continúa una sesión anterior
```

**2. Lectura de configuración (en orden de prioridad):**
```
1. Argumentos de línea de comandos    (mayor prioridad)
2. Variables de entorno               (ANTHROPIC_API_KEY, etc.)
3. .claude/settings.json              (configuración del proyecto)
4. ~/.claude/settings.json            (configuración global)
5. Valores por defecto del CLI        (menor prioridad)
```

**3. Validación de la API key:**
```
- Si ANTHROPIC_API_KEY está definida → usar
- Si no → buscar en el keyring del sistema
- Si no → mostrar error e instrucciones de configuración
```

**4. Carga de CLAUDE.md:**
```
Orden de búsqueda (todos se cargan, en orden):
1. ~/.claude/CLAUDE.md            (instrucciones globales)
2. {workspace}/CLAUDE.md          (instrucciones del proyecto)
3. {workspace}/{subdirectorio}/CLAUDE.md  (instrucciones del módulo)

Los tres se concatenan en el system prompt.
```

**5. Inicialización de MCP Servers:**
```
Para cada servidor en .mcp.json:
  1. Verificar que el comando existe (npx, node, python, etc.)
  2. Lanzar el proceso hijo
  3. Establecer comunicación stdio
  4. Ejecutar handshake (initialize)
  5. Obtener lista de herramientas (tools/list)
  6. Registrar herramientas en el Tool Executor
  
  Si algún servidor falla:
    - Mostrar advertencia
    - Continuar sin ese servidor
    - Las herramientas de ese servidor no estarán disponibles
```

**6. Inicio del REPL (modo interactivo) o ejecución directa (modo --print).**

### Los mensajes de inicio

```
╭──────────────────────────────────────────────────────────────╮
│ ✻ Welcome to Claude Code!                                    │
│   claude-sonnet-4-20250514                                   │
│   /help for help, /status for token count, /cost for costs  │
│                                                              │
│   MCP: postgres ✓ | github ✓ | mi-servidor ✗ (error)        │
╰──────────────────────────────────────────────────────────────╯

>
```

---

## 10.3 El modo interactivo: el REPL en funcionamiento

### La anatomía del REPL

El modo interactivo es el modo predeterminado cuando ejecutas `claude` sin `--print`. La sesión permanece abierta hasta que el usuario la cierra explícitamente.

```
ESTADO DEL REPL:

  ESPERA_INPUT
       │
       │ Usuario escribe y presiona Enter
       ▼
  PROCESANDO
       │
       ├── Envía petición a la API
       ├── Streaming de la respuesta
       ├── Ejecuta herramientas si tool_use
       └── Repite hasta end_turn
       │
       ▼
  ESPERA_INPUT  ← Vuelve al inicio
```

### Los indicadores visuales durante el procesamiento

Mientras Claude trabaja, el REPL muestra indicadores:

```
> Refactoriza src/auth.py para usar bcrypt

⠋ Pensando...

⊕ Read src/auth.py
  → Leyendo archivo (234 líneas)

⊕ Read requirements.txt
  → Leyendo dependencias

⊕ Bash: pip show bcrypt
  → bcrypt 4.1.2 instalado ✓

⊕ Edit src/auth.py
  → Reemplazando función hash_password

⊕ Edit src/auth.py
  → Reemplazando función verify_password

⊕ Bash: pytest tests/test_auth.py -v
  → 5 passed, 0 failed ✓

He refactorizado src/auth.py para usar bcrypt:
- hash_password() ahora usa bcrypt.hashpw() con salt
- verify_password() ahora usa bcrypt.checkpw()
- Todos los tests pasan (5/5)

>
```

### Los comandos del REPL (comandos /slash)

Los comandos que empiezan con `/` son procesados localmente:

```
/help      → Lista todos los comandos disponibles
/status    → Estado de la sesión (modelo, contexto, MCP servers)
/cost      → Uso de tokens y costo estimado
/model     → Ver o cambiar el modelo actual
/compact   → Comprimir el historial (con petición a la API)
/clear     → Borrar todo el historial (operación local)
/add-dir   → Añadir un directorio adicional al workspace
/exit      → Cerrar la sesión
/quit      → Alias de /exit
```

### Interrumpir una operación en curso

Si Claude está procesando y quieres interrumpirlo:

```
Ctrl+C → Interrumpe la operación actual y vuelve al prompt >
         (el historial hasta ese punto se conserva)

Ctrl+C+C → Dos veces rápido: sale de Claude Code
           (equivale a /exit)

Ctrl+D   → En línea vacía: cierra Claude Code (señal EOF)
```

---

## 10.4 Modo no interactivo: --print y automatización

### Qué es el modo --print

El modo `--print` ejecuta Claude Code de forma **no interactiva**: recibe una tarea, la procesa, imprime la respuesta y termina. No hay REPL, no hay prompts al usuario.

```bash
# Modo interactivo (predeterminado):
claude
# → Muestra el REPL, espera input del usuario

# Modo no interactivo:
claude --print "¿Cuántos archivos Python hay en src/?"
# → Ejecuta la tarea, imprime la respuesta, termina
```

### Cómo se usa en scripts

```bash
#!/bin/bash
# analizar-codigo.sh

OUTPUT=$(claude \
  --dangerously-skip-permissions \
  --allowedTools "Read,Glob,Grep" \
  --print "Analiza src/ buscando funciones sin docstrings.
           Lista solo las funciones sin documentar, con su archivo y línea.
           Formato: archivo:línea - nombre_funcion()")

echo "Funciones sin documentar:"
echo "$OUTPUT"

# Contar cuántas hay
COUNT=$(echo "$OUTPUT" | grep -c "^src/")
echo "Total: $COUNT funciones sin documentar"
```

### El exit code de --print

Claude Code en modo `--print` retorna diferentes exit codes:

```
0  → Tarea completada sin errores
1  → Error de configuración (API key inválida, etc.)
2  → Error durante la ejecución de la tarea
```

Esto permite usar Claude Code en pipelines condicionales:

```bash
if claude --print "Verifica que todos los tests pasan" --dangerously-skip-permissions; then
  echo "✓ Tests OK, procediendo con el deploy"
  ./deploy.sh
else
  echo "✗ Tests fallaron, abortando deploy"
  exit 1
fi
```

### Pasar tareas desde stdin

```bash
# Pasar la tarea desde un archivo
cat tarea.txt | claude --print --dangerously-skip-permissions

# Pasar la tarea desde otra herramienta
git diff HEAD~1 | claude --print \
  "Este es un diff de los últimos cambios. 
   Revisa si hay algún problema de seguridad introducido."
```

---

## 10.5 Modo headless: integraciones y pipelines

### Qué es el modo headless

El modo "headless" es similar al modo `--print`, pero diseñado específicamente para integraciones donde la salida se procesa programáticamente.

```bash
# Formato de salida JSON para procesamiento programático
claude --print --output-format json "Analiza src/auth.py"
```

El output JSON tiene la estructura:
```json
{
  "response": "El análisis de src/auth.py revela...",
  "model": "claude-sonnet-4-20250514",
  "usage": {
    "input_tokens": 4521,
    "output_tokens": 287
  },
  "stop_reason": "end_turn",
  "session_id": "sess_01ABC123"
}
```

### Integración con herramientas de CI/CD

**GitHub Actions:**
```yaml
- name: Claude Code Analysis
  id: claude-analysis
  run: |
    RESULT=$(claude --print --output-format json \
      --dangerously-skip-permissions \
      --allowedTools "Read,Glob,Grep" \
      "Analiza los cambios en el PR")
    
    echo "result=$(echo $RESULT | jq -r .response)" >> $GITHUB_OUTPUT

- name: Comment on PR
  uses: actions/github-script@v7
  with:
    script: |
      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: `**Análisis de Claude Code:**\n\n${{ steps.claude-analysis.outputs.result }}`
      });
```

---

## 10.6 Gestión del historial durante la sesión

### El crecimiento del historial

Durante una sesión activa, el historial crece con cada intercambio:

```
t=0  Inicio: historial = []
              Tokens: ~3,000 (solo system)

t=5m  Primer intercambio:
       historial = [msg1, resp1, tool_results1]
       Tokens: ~15,000

t=30m  Múltiples intercambios:
        historial = [msg1..msg8, resp1..resp8, tools1..tools8]
        Tokens: ~60,000

t=1h  Sesión larga:
       historial = [...]  (muchos intercambios)
       Tokens: ~100,000+
       → Considerar /compact
```

### El impacto en la latencia

El tamaño del contexto afecta directamente a la latencia de cada petición:

| Tokens en el contexto | Latencia aproximada hasta primer token |
|----------------------|----------------------------------------|
| 1,000 - 5,000 | 0.5 - 1 segundo |
| 5,000 - 20,000 | 1 - 3 segundos |
| 20,000 - 50,000 | 2 - 5 segundos |
| 50,000 - 100,000 | 4 - 10 segundos |
| 100,000 - 150,000 | 8 - 20 segundos |

### Cuándo intervenir

```
/cost muestra 30-50% → Todo bien, sin acción necesaria
/cost muestra 50-70% → Considerar /compact pronto
/cost muestra 70-85% → USAR /compact ahora
/cost muestra >85%   → USO URGENTE de /compact o riesgo de error
```

### El proceso de /compact en detalle

```
Usuario ejecuta: /compact

REPL:
  → "Comprimiendo historial..."

Context Manager:
  1. Obtiene el historial completo (puede ser 80,000+ tokens)
  2. Construye petición especial:
     {
       "model": modelo_actual,
       "messages": [historial_completo],
       "system": "Eres un asistente que crea resúmenes concisos de
                  conversaciones de programación. Preserva: tareas 
                  completadas, archivos modificados, decisiones técnicas,
                  tareas pendientes y contexto del proyecto."
     }
  3. Envía la petición a la API
  4. Recibe el resumen (~3,000-5,000 tokens)
  5. REEMPLAZA el historial completo con el resumen

REPL:
  → "Historial comprimido: 82,340 tokens → 4,120 tokens (95% reducción)"
  → Vuelve al prompt >
```

---

## 10.7 Transiciones de estado durante una tarea

### El diagrama de estados durante una tarea

```
REPL en ESPERA_INPUT
         │
         │ Usuario envía mensaje
         ▼
CONSTRUYENDO_PAYLOAD
         │
         │ Payload listo
         ▼
ENVIANDO_A_API ──────────────────────────────────────────────────┐
         │                                                        │
         │ stop_reason = "end_turn"           stop_reason = "tool_use"
         ▼                                         │
MOSTRANDO_RESPUESTA              EJECUTANDO_HERRAMIENTA(S)
         │                                    │    │
         │                               OK   │    │  Cancelado por usuario
         ▼                                    │    │  o hook PreToolUse
REPL en ESPERA_INPUT            AÑADIENDO_RESULTADO           │
                                         │                    │
                                         │    ◄───────────────┘
                                         │ Vuelve a ENVIANDO_A_API
                                         │ con historial actualizado
                                         └──────────────────────────►
                                                                     │
                                                         CONSTRUYENDO_PAYLOAD
                                                         (siguiente iteración)
```

### Los estados especiales

**ESPERANDO_CONFIRMACION:**
Ocurre cuando Claude quiere ejecutar una operación que requiere confirmación del usuario. El REPL muestra el prompt y espera input:

```
Claude quiere ejecutar:
  rm -rf tmp/

¿Permitir? [y/N/always] _
```

El usuario puede responder:
- `y` → La herramienta se ejecuta, continúa el ciclo.
- `n` → La herramienta retorna un error, Claude puede intentar otra cosa.
- `always` → Se permite esta vez y todas las siguientes del mismo tipo (en esta sesión).

**STREAMING:**
Durante la generación de texto, cada token llega de forma incremental y el REPL lo muestra en tiempo real. Este estado puede interrumpirse con Ctrl+C.

---

## 10.8 La fase de cierre

### Cómo termina una sesión

Una sesión puede terminar de varias formas:

**1. Por decisión del usuario:**
```bash
> /exit     # Comando explícito de salida
> /quit     # Alias de /exit
Ctrl+D      # Señal EOF en línea vacía
Ctrl+C+C    # Dos interrupciones rápidas
```

**2. Por finalización natural en modo --print:**
Claude Code termina automáticamente cuando la tarea está completa.

**3. Por error irrecuperable:**
```
- La API key ha expirado o es inválida
- Error de conexión persistente
- Límite de contexto superado sin posibilidad de recuperación
```

### Lo que ocurre durante el cierre

```
1. El REPL recibe la señal de cierre

2. Se dispara el evento Stop:
   → Se ejecutan los hooks Stop configurados
   → (Si hay hook de auto-commit, se ejecuta aquí)
   → (Si hay hook de informe de sesión, se genera aquí)

3. MCP Client cierra los servidores MCP:
   → Envía notificación de cierre a cada servidor
   → Espera a que los procesos hijos terminen (o les envía SIGTERM)

4. El Context Manager libera la memoria del historial

5. El proceso de Claude Code termina
```

### El hook Stop en la práctica

```bash
#!/bin/bash
# ~/.claude/hooks/on-stop.sh

PROJECT="${CLAUDE_PROJECT_DIR}"
SESSION="${CLAUDE_SESSION_ID:0:8}"

echo "[$(date +%H:%M:%S)] Sesión $SESSION terminada en $PROJECT"

# Si hay cambios sin commitear, mostrar un recordatorio
if git -C "$PROJECT" diff --quiet 2>/dev/null; then
  echo "  ✓ Sin cambios pendientes"
else
  echo "  ⚠ Hay cambios sin commitear:"
  git -C "$PROJECT" diff --stat 2>/dev/null
fi

exit 0
```

---

## 10.9 Continuación de sesiones: --resume

### El problema que resuelve

Por defecto, cada vez que ejecutas `claude`, empiezas con un contexto vacío. Si estabas trabajando en una tarea larga y necesitas pausar (reiniciar el equipo, terminar el día laboral), al volver tienes que reconstruir el contexto.

`--resume` permite continuar una sesión anterior:

```bash
# Iniciar sesión normal
$ claude
# [trabajas en una tarea, identificas el ID de sesión con /status]
# > /status
# Sesión ID: sess_01ABC123XYZ

# Más tarde, continuar esa sesión
$ claude --resume sess_01ABC123XYZ
# → El historial de la sesión anterior se carga automáticamente
# → Claude "recuerda" todo lo que se discutió
```

### Cuándo es útil

- **Jornada laboral larga:** Pausa para comer y retoma donde lo dejaste.
- **Interrupciones:** Reinicio del sistema, necesidad de hacer otra cosa.
- **Tareas complejas multi-día:** El trabajo en una refactorización grande que se extiende varios días.

### Cuándo NO es útil

- Si han pasado días y el contexto ya no es relevante (usa `/clear` en su lugar).
- Si la sesión anterior terminó con errores o en un estado inconsistente.
- Si cambias de tarea completamente.

---

## 10.10 Sesiones con IDE: integración con editores

### Claude Code como extensión del IDE

Claude Code tiene integraciones con IDEs populares que añaden capacidades adicionales:

**VS Code / Cursor:**
- Claude puede leer el archivo actualmente abierto en el editor.
- Claude puede ver las selecciones de texto del editor.
- Los cambios que Claude hace se reflejan en tiempo real en el editor.

**JetBrains (IntelliJ IDEA, PyCharm, etc.):**
- Integración similar a VS Code.
- Claude puede acceder a la estructura del proyecto del IDE.

### Cómo funciona la integración

Cuando Claude Code se lanza desde un IDE (o el IDE tiene el plugin instalado), tiene acceso a:

```
Variables de entorno adicionales:
  CLAUDE_IDE_FILE          → Archivo actualmente abierto en el IDE
  CLAUDE_IDE_SELECTION     → Texto seleccionado en el editor
  CLAUDE_IDE_LINE          → Línea actual del cursor
  CLAUDE_IDE_PROJECT       → Ruta del proyecto en el IDE
```

Claude puede usar esta información contextual sin que el usuario tenga que especificarla explícitamente:

```
> Explica la función en la que está el cursor

  → Claude lee CLAUDE_IDE_FILE y CLAUDE_IDE_LINE
  → Localiza la función relevante
  → Explica sin que el usuario tenga que copiar/pegar el código
```

---

## 10.11 Resumen y glosario

### Resumen

1. Una sesión tiene tres fases: **inicialización** (lee configuración, lanza MCP), **modo activo** (REPL o --print), y **cierre** (hooks Stop, limpieza).
2. La **inicialización** lee configuración en orden de prioridad: args > env vars > .claude/settings.json > ~/.claude/settings.json.
3. El **modo interactivo** (REPL) permanece abierto hasta que el usuario lo cierra; el **modo --print** termina automáticamente.
4. El historial crece durante la sesión y afecta a la **latencia** y la **calidad**. Gestionar con `/compact` o `/clear`.
5. Durante cada tarea, el agente pasa por estados: `CONSTRUYENDO_PAYLOAD` → `ENVIANDO_A_API` → `EJECUTANDO_HERRAMIENTA` → de vuelta a `ENVIANDO_A_API`.
6. El **cierre** dispara los hooks `Stop`, cierra los MCP servers y libera la memoria.
7. **`--resume`** permite continuar sesiones anteriores cargando su historial.
8. La **integración con IDE** añade contexto extra (archivo abierto, selección) disponible via variables de entorno.

### Glosario

| Término | Definición |
|---------|------------|
| **REPL** | Read-Eval-Print Loop. El modo interactivo de Claude Code que permanece abierto. |
| **Modo --print** | Modo no interactivo: ejecuta una tarea, imprime la respuesta y termina. |
| **Fase de inicialización** | La fase al arrancar Claude Code: lee configuración, carga CLAUDE.md, lanza MCP servers. |
| **Fase de cierre** | La fase al terminar la sesión: ejecuta hooks Stop, cierra MCP servers, libera memoria. |
| **--resume** | Flag para continuar una sesión anterior cargando su historial. |
| **Modo headless** | Modo de ejecución sin interfaz de usuario, diseñado para integraciones programáticas. |
| **Streaming** | Estado durante el que los tokens de respuesta llegan progresivamente del API. |
| **Señal EOF** | End of File. En Claude Code (Ctrl+D), indica el fin de la sesión interactiva. |

---

## Ver también

- **[Capítulo 3](./cap-03-arquitectura-componentes.md):** Arquitectura — los componentes que forman la sesión.
- **[Capítulo 5](./cap-05-ventana-contexto.md):** Ventana de Contexto — gestión del historial durante la sesión.
- **[Capítulo 8](./cap-08-sistema-hooks.md):** Hooks — los eventos Stop y Notification de la sesión.
- **[Capítulo 11](./cap-11-performance-optimizacion.md):** Performance — optimizar el rendimiento durante la sesión.

---

> 📌 Siguiente capítulo: [Cap. 11 — Performance y Optimización](./cap-11-performance-optimizacion.md)

