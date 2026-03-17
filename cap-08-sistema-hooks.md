# Capítulo 8: El Sistema de Hooks — Eventos y Automatización

> **Nivel:** Requiere comprensión básica de Claude Code  
> **Prerequisito recomendado:** [Cap. 3](./cap-03-arquitectura-componentes.md) y [Cap. 4](./cap-04-sistema-herramientas.md)  
> **Objetivo:** Entender qué son los hooks, cómo se configuran, cuándo se ejecutan y cómo usarlos para automatizar tareas en el ciclo de trabajo.

---

## Tabla de Contenidos

- [8.1 Qué son los hooks y el problema que resuelven](#81-qué-son-los-hooks)
- [8.2 Los cuatro tipos de evento](#82-los-cuatro-tipos-de-evento)
- [8.3 El flujo de ejecución con hooks](#83-flujo-de-ejecución)
- [8.4 La anatomía de un hook: configuración completa](#84-la-anatomía-de-un-hook)
- [8.5 Variables de entorno disponibles en los hooks](#85-variables-de-entorno)
- [8.6 La lógica de cancelación en PreToolUse](#86-cancelación-en-pretooluse)
- [8.7 Hooks globales vs. hooks de proyecto](#87-globales-vs-proyecto)
- [8.8 Casos de uso reales con código completo](#88-casos-de-uso-reales)
- [8.9 El evento Notification: Claude pide atención](#89-el-evento-notification)
- [8.10 El evento Stop: al finalizar la sesión](#810-el-evento-stop)
- [8.11 Depurar hooks que fallan](#811-depurar-hooks)
- [8.12 Resumen y glosario del capítulo](#812-resumen-y-glosario)

---

## 8.1 Qué son los hooks y el problema que resuelven

### La analogía de los webhooks

Si conoces los webhooks de GitHub, los hooks de Claude Code funcionan de forma muy similar.

En GitHub, cuando ocurre un evento (alguien hace un push, crea un PR, cierra un issue), GitHub puede enviar automáticamente una notificación HTTP a una URL que tú configures. Esa URL puede hacer cualquier cosa: ejecutar CI, enviar notificaciones, actualizar documentación.

Los **hooks de Claude Code** son el equivalente: cuando Claude realiza ciertas acciones (lee un archivo, escribe código, ejecuta un comando, termina la sesión), se ejecuta automáticamente un script que tú defines.

### El problema que resuelven

Sin hooks, Claude Code opera en una "caja negra": Claude hace sus cosas y tú no puedes interceptar ni reaccionar a cada acción individual.

Los hooks te dan **control granular** sobre el ciclo de trabajo:

1. **Antes de que Claude modifique algo** (PreToolUse): validar que el cambio es seguro, bloquear archivos de producción.
2. **Después de que Claude modifique algo** (PostToolUse): formatear automáticamente el código, ejecutar linters.
3. **Cuando Claude necesita tu atención** (Notification): recibir una notificación en el móvil o en Slack.
4. **Cuando la sesión termina** (Stop): hacer un commit automático, enviar un informe de sesión.

### La diferencia entre manual y automático

```
MANUAL (sin hooks):
  Tú: "Claude, modifica auth.py"
  Claude: [modifica auth.py]
  Tú: (recuerdas) "Ah, tengo que ejecutar el formatter..."
  Tú: black src/auth.py

AUTOMÁTICO (con hooks PostToolUse):
  Tú: "Claude, modifica auth.py"
  Claude: [modifica auth.py]
  Hook PostToolUse: [automáticamente ejecuta black src/auth.py]
  → El código siempre queda formateado, sin que nadie lo recuerde.
```

---

## 8.2 Los cuatro tipos de evento

### PreToolUse

Se ejecuta **antes** de que Claude ejecute una herramienta. El script puede:
- **Cancelar la acción** (si retorna exit code != 0).
- Realizar validaciones previas.
- Registrar la intención de la acción.

**Disponible para:** Read, Write, Edit, MultiEdit, Bash, y todas las herramientas MCP.

**Caso de uso típico:** Bloquear modificaciones a archivos de configuración de producción.

### PostToolUse

Se ejecuta **después** de que Claude ejecute una herramienta. El script:
- **No puede cancelar** la acción (ya ocurrió).
- Puede realizar acciones secundarias.
- Su resultado **NO afecta al modelo** (Claude no ve el output del hook).

**Caso de uso típico:** Formatear automáticamente código después de que Claude lo escribe.

### Notification

Se ejecuta cuando Claude Code necesita enviar una notificación al usuario. Ocurre cuando:
- Claude ha completado una tarea larga y quiere avisar.
- Claude ha encontrado un problema que requiere atención humana.
- Una operación de larga duración ha terminado.

**Caso de uso típico:** Enviar una notificación push al móvil o un mensaje a Slack.

### Stop

Se ejecuta cuando la sesión de Claude Code termina.

**Caso de uso típico:** Hacer un commit automático con los cambios de la sesión, enviar un informe.

---

## 8.3 Flujo de ejecución con hooks

### Diagrama de flujo completo (PreToolUse)

```
Claude decide usar Write(src/auth.py, nuevo_contenido)
                      │
                      ▼
          ¿Hay hooks PreToolUse para Write?
                      │
              NO ─────┤───── SÍ
                      │              │
                      │              ▼
                      │    Ejecuta el script del hook
                      │              │
                      │    ¿Exit code == 0?
                      │              │
                      │    OK ───────┤──── ERROR
                      │              │              │
                      │              │         Cancela la acción
                      │              │         Claude recibe error
                      ▼              ▼
              Tool Executor ejecuta Write(src/auth.py, contenido)
                              │
                              ▼
              ¿Hay hooks PostToolUse para Write?
                              │
                  NO ─────────┤──────── SÍ
                              │                  │
                              │                  ▼
                              │    Ejecuta el script del hook
                              │    (resultado no afecta a Claude)
                              │
                              ▼
              Resultado devuelto a Claude
```

### Puntos clave del flujo

1. **PreToolUse se ejecuta antes**: Claude ya decidió qué hacer; el hook puede impedirlo pero no cambiarlo.
2. **PostToolUse se ejecuta después**: La acción ya ocurrió; el hook puede reaccionar pero no deshacerla.
3. **El resultado de PostToolUse no llega a Claude**: Los hooks son transparentes para el LLM.
4. **Los hooks tienen timeout**: Por defecto 30 segundos. Si el script tarda más, se cancela.

---

## 8.4 La anatomía de un hook: configuración completa

### La estructura de configuración en settings.json

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "/ruta/al/script-de-validacion.sh",
            "timeout": 10000
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "cd '${CLAUDE_PROJECT_DIR}' && python -m black '${CLAUDE_FILE_PATH}' -q 2>/dev/null; true",
            "timeout": 30000
          }
        ]
      },
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$(date): ${CLAUDE_TOOL_NAME}\" >> ~/.claude/audit.log",
            "timeout": 5000
          }
        ]
      }
    ],
    "Notification": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' '${CLAUDE_NOTIFICATION_MESSAGE}' 2>/dev/null || true",
            "timeout": 5000
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "/ruta/al/script-de-cierre.sh",
            "timeout": 60000
          }
        ]
      }
    ]
  }
}
```

### El campo `matcher`: para qué herramienta se activa

```
"Write"      → Solo para operaciones Write
"Edit"       → Solo para operaciones Edit
"MultiEdit"  → Solo para operaciones MultiEdit
"Bash"       → Solo para operaciones Bash
"Read"       → Solo para operaciones Read
"Glob"       → Solo para operaciones Glob
"Grep"       → Solo para operaciones Grep
"Agent"      → Solo para crear subagentes
"*"          → Todas las herramientas (wildcard)
```

### El campo `timeout`

Tiempo máximo de espera en **milisegundos**:

```
1000  → 1 segundo   (log, notify rápidos)
5000  → 5 segundos  (format, lint)
30000 → 30 segundos (predeterminado)
60000 → 1 minuto    (operaciones más lentas)
```

---

## 8.5 Variables de entorno disponibles en los hooks

Los hooks reciben información sobre la acción a través de variables de entorno:

### Variables disponibles en todos los hooks

```bash
CLAUDE_TOOL_NAME      # "Write", "Bash", "Read", etc.
CLAUDE_SESSION_ID     # Identificador único de la sesión
CLAUDE_PROJECT_DIR    # Directorio del proyecto (donde se ejecutó claude)
```

### Variables específicas por herramienta

**Para Write, Edit, MultiEdit, Read:**
```bash
CLAUDE_FILE_PATH      # Ruta completa: /home/user/proyecto/src/auth.py
CLAUDE_FILE_RELATIVE  # Ruta relativa al proyecto: src/auth.py
```

**Para Bash:**
```bash
CLAUDE_BASH_COMMAND   # El comando completo: "pytest tests/ -v"
```

**Para Notification:**
```bash
CLAUDE_NOTIFICATION_MESSAGE  # El texto del mensaje
CLAUDE_NOTIFICATION_LEVEL    # "info", "warning", "error"
```

### Usar las variables en comandos inline

```json
{
  "type": "command",
  "command": "echo 'Claude escribió: ${CLAUDE_FILE_RELATIVE}' >> ~/log.txt"
}
```

O en scripts externos (las variables están disponibles como variables de entorno normales):

```bash
#!/bin/bash
# hook-post-write.sh
echo "Archivo modificado: $CLAUDE_FILE_PATH"
echo "Sesión: $CLAUDE_SESSION_ID"
echo "Proyecto: $CLAUDE_PROJECT_DIR"
```

---

## 8.6 Cancelación en PreToolUse

### Cómo cancelar una acción

En un hook `PreToolUse`, cancelas la acción de Claude haciendo que tu script salga con un **exit code distinto de 0**:

```bash
#!/bin/bash
# pre-write-validator.sh

PROTECTED_FILES=(
  "config/production.yml"
  ".env.prod"
  "alembic/versions/"
)

for protected in "${PROTECTED_FILES[@]}"; do
  if [[ "$CLAUDE_FILE_RELATIVE" == *"$protected"* ]]; then
    echo "ERROR: '$CLAUDE_FILE_RELATIVE' es un archivo protegido."
    echo "Las modificaciones a este archivo requieren revisión manual."
    exit 1  # ← Exit code 1: Claude Code CANCELA la acción Write
  fi
done

exit 0  # ← Exit code 0: Claude Code PERMITE la acción Write
```

### Qué recibe Claude cuando se cancela una acción

```
tool_result: {
  isError: true,
  content: "La acción fue cancelada por un hook PreToolUse:
             ERROR: 'config/production.yml' es un archivo protegido.
             Las modificaciones a este archivo requieren revisión manual."
}
```

Claude puede entonces intentar un enfoque alternativo o informar al usuario.

### El output del script en la cancelación

El texto que imprime tu script (stdout y stderr) se incluye en el mensaje de error que recibe Claude. Úsalo para dar información útil:

```bash
# Malo: mensaje sin contexto
exit 1

# Bueno: mensaje que Claude puede entender y comunicar al usuario
echo "CANCELADO: No se puede modificar 'config/production.yml'."
echo "ALTERNATIVA: Modifica 'config/staging.yml' o 'config/development.yml' en su lugar."
exit 1
```

---

## 8.7 Hooks globales vs. hooks de proyecto

### La diferencia

**Hooks globales** (`~/.claude/settings.json`):
- Se aplican en **todos tus proyectos**.
- Contienen comportamientos que quieres en cualquier contexto.
- Ejemplos: notificaciones, audit log global, formateo universal.

**Hooks de proyecto** (`.claude/settings.json` en el directorio del proyecto):
- Se aplican **solo en ese proyecto**.
- Contienen reglas específicas del proyecto.
- Ejemplos: protección de archivos, linters específicos del stack, tests automáticos.

### El orden de ejecución

Cuando hay hooks en ambos lugares, se ejecutan en orden:
1. Hooks globales (`~/.claude/settings.json`)
2. Hooks de proyecto (`.claude/settings.json`)

Si un hook global cancela la acción (PreToolUse con exit 1), los hooks de proyecto **no se ejecutan** para esa acción.

### Ejemplo de separación global/proyecto

```json
// ~/.claude/settings.json (GLOBAL: para todos los proyectos)
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "echo \"$(date -Iseconds) ${CLAUDE_TOOL_NAME}: ${CLAUDE_FILE_RELATIVE:-${CLAUDE_BASH_COMMAND:-n/a}}\" >> ~/.claude/global-audit.log",
        "timeout": 3000
      }]
    }],
    "Notification": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "notify-send 'Claude Code' '${CLAUDE_NOTIFICATION_MESSAGE}' 2>/dev/null || true",
        "timeout": 3000
      }]
    }]
  }
}
```

```json
// proyecto/.claude/settings.json (PROYECTO: solo para este proyecto)
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Write",
      "hooks": [{
        "type": "command",
        "command": "/bin/bash ${CLAUDE_PROJECT_DIR}/.claude/hooks/check-protected-files.sh",
        "timeout": 5000
      }]
    }],
    "PostToolUse": [{
      "matcher": "Write",
      "hooks": [{
        "type": "command",
        "command": "cd '${CLAUDE_PROJECT_DIR}' && [[ '${CLAUDE_FILE_RELATIVE}' == *.py ]] && python -m black '${CLAUDE_FILE_PATH}' -q 2>/dev/null; true",
        "timeout": 15000
      }]
    }]
  }
}
```

---

## 8.8 Casos de uso reales con código completo

### Caso 1: Formatter automático al escribir Python

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [{
          "type": "command",
          "command": "if [[ '${CLAUDE_FILE_RELATIVE}' == *.py ]]; then cd '${CLAUDE_PROJECT_DIR}' && python -m black '${CLAUDE_FILE_PATH}' -q 2>/dev/null && python -m isort '${CLAUDE_FILE_PATH}' -q 2>/dev/null; fi; true",
          "timeout": 20000
        }]
      }
    ]
  }
}
```

### Caso 2: Proteger archivos de producción

```bash
#!/bin/bash
# .claude/hooks/check-protected-files.sh

PROTECTED_PATTERNS=(
  "config/production"
  ".env.prod"
  ".env.production"
  "alembic/versions/"
  "k8s/production"
)

FILE="${CLAUDE_FILE_RELATIVE}"

for pattern in "${PROTECTED_PATTERNS[@]}"; do
  if [[ "$FILE" == *"$pattern"* ]]; then
    echo "ARCHIVO PROTEGIDO: '$FILE'"
    echo ""
    echo "Este archivo está en la lista de archivos de producción."
    echo "Modifícalo manualmente con revisión explícita."
    echo ""
    echo "Alternativas disponibles:"
    echo "  - config/staging.yml"
    echo "  - config/development.yml"
    exit 1
  fi
done

exit 0
```

### Caso 3: Audit log completo de la sesión

```bash
#!/bin/bash
# ~/.claude/hooks/audit-log.sh

LOG_FILE="$HOME/.claude/audit/$(date +%Y-%m-%d).jsonl"
mkdir -p "$(dirname "$LOG_FILE")"

TIMESTAMP=$(date -Iseconds)
SESSION="$CLAUDE_SESSION_ID"
TOOL="$CLAUDE_TOOL_NAME"
PROJECT="${CLAUDE_PROJECT_DIR##*/}"

echo "{\"timestamp\":\"$TIMESTAMP\",\"session\":\"${SESSION:0:8}\",\"project\":\"$PROJECT\",\"tool\":\"$TOOL\",\"file\":\"${CLAUDE_FILE_RELATIVE:-}\",\"command\":\"${CLAUDE_BASH_COMMAND:-}\"}" >> "$LOG_FILE"

exit 0
```

### Caso 4: Ejecutar tests relacionados automáticamente

```bash
#!/bin/bash
# .claude/hooks/run-related-tests.sh

FILE="$CLAUDE_FILE_RELATIVE"

# Solo para archivos Python en src/ (no para tests mismos)
if [[ "$FILE" != src/*.py ]] || [[ "$FILE" == tests/* ]]; then
  exit 0
fi

# Derivar el path del test correspondiente
MODULE_FILE="$(basename "$FILE" .py)"
SIMPLE_TEST="tests/test_${MODULE_FILE}.py"
TESTS_DIR="$CLAUDE_PROJECT_DIR"

if [ -f "$TESTS_DIR/$SIMPLE_TEST" ]; then
  echo "Ejecutando tests relacionados: $SIMPLE_TEST"
  cd "$TESTS_DIR"
  python -m pytest "$SIMPLE_TEST" -q --tb=short 2>&1 | tail -20
else
  echo "No se encontraron tests específicos para $FILE"
fi

exit 0  # No cancelar incluso si los tests fallan (esto es PostToolUse)
```

---

## 8.9 El evento Notification: Claude pide atención

### Cuándo se dispara Notification

Claude Code envía una notificación cuando necesita que el usuario sepa algo:

1. **Tarea larga completada:** "He terminado de documentar los 50 archivos."
2. **Se requiere entrada del usuario:** "He encontrado un conflicto de merge que requiere decisión."
3. **Error irrecuperable:** "No puedo continuar sin acceso a la base de datos."
4. **Advertencia importante:** "El contexto está al 90%. Considera usar /compact."

### Ejemplo: Notificación en el escritorio (Linux)

```bash
#!/bin/bash
# ~/.claude/hooks/desktop-notify.sh

MSG="${CLAUDE_NOTIFICATION_MESSAGE}"
LEVEL="${CLAUDE_NOTIFICATION_LEVEL:-info}"

case "$LEVEL" in
  "error")   URGENCY="critical" ;;
  "warning") URGENCY="normal" ;;
  *)         URGENCY="low" ;;
esac

# notify-send (Linux con libnotify)
notify-send \
  --urgency="$URGENCY" \
  --app-name="Claude Code" \
  "Claude Code" \
  "$MSG" 2>/dev/null

exit 0
```

### Ejemplo: Notificación por Slack

```bash
#!/bin/bash
# ~/.claude/hooks/slack-notify.sh

MSG="${CLAUDE_NOTIFICATION_MESSAGE}"
PROJECT="${CLAUDE_PROJECT_DIR##*/}"
SLACK_WEBHOOK="${SLACK_WEBHOOK_URL}"

if [ -z "$SLACK_WEBHOOK" ]; then
  exit 0
fi

curl -s -X POST "$SLACK_WEBHOOK" \
  -H 'Content-type: application/json' \
  --data "{\"text\": \"*Claude Code* en _${PROJECT}_: ${MSG}\"}" > /dev/null

exit 0
```

---

## 8.10 El evento Stop: al finalizar la sesión

### Cuándo se dispara Stop

El evento `Stop` se dispara cuando la sesión de Claude Code termina:
- El usuario presiona Ctrl+C o Ctrl+D.
- El comando `exit` o `quit`.
- Cuando Claude termina su trabajo en modo `--print` (no interactivo).

### Ejemplo: Informe de sesión al finalizar

```bash
#!/bin/bash
# .claude/hooks/session-report.sh

PROJECT_DIR="$CLAUDE_PROJECT_DIR"
REPORT_DIR="$PROJECT_DIR/.claude/session-reports"
mkdir -p "$REPORT_DIR"

REPORT_FILE="$REPORT_DIR/$(date +%Y-%m-%d_%H-%M-%S).md"

cd "$PROJECT_DIR" || exit 0

{
  echo "# Informe de sesión Claude Code"
  echo ""
  echo "**Fecha:** $(date)"
  echo "**Sesión:** $CLAUDE_SESSION_ID"
  echo "**Proyecto:** ${PROJECT_DIR##*/}"
  echo ""
  echo "## Archivos modificados en esta sesión"
  echo ""
  if git rev-parse --git-dir > /dev/null 2>&1; then
    git diff --name-status HEAD
  else
    echo "(No es un repositorio git)"
  fi
} > "$REPORT_FILE"

echo "Informe guardado en: $REPORT_FILE"
exit 0
```

---

## 8.11 Depurar hooks que fallan

### Problema 1: El hook no se ejecuta

**Diagnóstico:**
```bash
# 1. Verificar que el JSON de configuración es válido
jq . .claude/settings.json

# 2. Verificar que el matcher es correcto (sensible a mayúsculas)
# Correcto:   "Write", "Bash", "Read"
# Incorrecto: "write", "bash", "read"

# 3. Verificar que el evento está bien nombrado:
# Correcto:   "PreToolUse", "PostToolUse", "Notification", "Stop"
```

### Problema 2: El hook falla silenciosamente (PostToolUse)

El output del hook en PostToolUse no se muestra al usuario por defecto:

```bash
# Añadir logging explícito al hook
echo "$(date): Hook ejecutado para $CLAUDE_FILE_RELATIVE" >> /tmp/hook-debug.log
black "$CLAUDE_FILE_PATH" 2>> /tmp/hook-debug.log
echo "Exit code: $?" >> /tmp/hook-debug.log

# Revisar el log
tail -f /tmp/hook-debug.log
```

### Problema 3: El hook cancela acciones que no debería

```bash
#!/bin/bash
# Versión de debug del hook PreToolUse

LOG="/tmp/pre-tool-debug.log"
echo "=== Hook PreToolUse ===" >> "$LOG"
echo "Tool: $CLAUDE_TOOL_NAME" >> "$LOG"
echo "File: $CLAUDE_FILE_RELATIVE" >> "$LOG"

if [[ "$CLAUDE_FILE_RELATIVE" == *"production"* ]]; then
  echo "BLOQUEADO" >> "$LOG"
  echo "Archivo de producción bloqueado: $CLAUDE_FILE_RELATIVE"
  exit 1
fi

echo "PERMITIDO" >> "$LOG"
exit 0
```

### Problema 4: El hook tarda demasiado y se cancela por timeout

```bash
# Para PostToolUse, lanzar operaciones lentas en background
black "$CLAUDE_FILE_PATH" &
disown
# El hook termina inmediatamente, black sigue en background
exit 0
```

---

## 8.12 Resumen y glosario

### Resumen

1. Los **hooks** ejecutan scripts automáticamente en respuesta a las acciones de Claude Code.
2. Hay **cuatro tipos de evento**: `PreToolUse` (antes, puede cancelar), `PostToolUse` (después), `Notification` (aviso), `Stop` (cierre).
3. El **matcher** especifica para qué herramienta se activa: `"Write"`, `"Bash"`, `"*"` (todas), etc.
4. Los hooks reciben información via **variables de entorno**: `CLAUDE_FILE_PATH`, `CLAUDE_BASH_COMMAND`, etc.
5. Los hooks `PreToolUse` **cancelan acciones** con exit code != 0. El mensaje de error llega a Claude.
6. Los hooks `PostToolUse` **no pueden cancelar** la acción ni afectar a Claude.
7. Existen **hooks globales** (`~/.claude/settings.json`) y **de proyecto** (`.claude/settings.json`); se ejecutan en ese orden.
8. Casos de uso típicos: formatter automático, protección de archivos, audit log, notificaciones, informe de sesión.

### Glosario

| Término | Definición |
|---------|------------|
| **Hook** | Script que se ejecuta automáticamente en respuesta a un evento de Claude Code. |
| **PreToolUse** | Evento antes de ejecutar una herramienta. Puede cancelar la acción con exit code != 0. |
| **PostToolUse** | Evento después de ejecutar una herramienta. No puede cancelar ni afecta a Claude. |
| **Notification** | Evento que Claude dispara para comunicar algo al usuario. |
| **Stop** | Evento que ocurre al finalizar la sesión de Claude Code. |
| **Matcher** | Patrón que especifica para qué herramienta se activa el hook (ej: `"Write"`, `"*"`). |
| **Exit code** | Código de retorno de un proceso. 0 = éxito (permite la acción en PreToolUse), otro = error (cancela). |
| **Hook global** | Hook definido en `~/.claude/settings.json`, activo en todos los proyectos. |
| **Hook de proyecto** | Hook definido en `.claude/settings.json`, activo solo en ese proyecto. |

---

## Ver también

- **[Capítulo 3](./cap-03-arquitectura-componentes.md):** Arquitectura — cómo el Tool Executor gestiona la ejecución de hooks.
- **[Capítulo 9](./cap-09-seguridad-permisos.md):** Seguridad — los hooks como capa de protección adicional.
- **[Capítulo 10](./cap-10-ciclo-vida-sesion.md):** Ciclo de vida — cuándo exactamente se invocan los eventos.

---

> 📌 Siguiente capítulo: [Cap. 9 — Seguridad: Permisos y Confianza](./cap-09-seguridad-permisos.md)

