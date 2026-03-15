# 16. Hooks - Automatización de Eventos

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 16. Hooks
[⬆️ Top](#16-hooks---automatización-de-eventos)

**¿Qué hace?**
Los hooks son comandos shell que se ejecutan automáticamente en respuesta a eventos de Claude Code: antes/después de usar herramientas, cuando el usuario envía un mensaje, cuando Claude termina de responder. Permiten automatizar el workflow sin intervención manual.

**Funcionamiento interno:**

```
Sin hooks:
Usuario → Claude edita archivo → Claude termina → tú corres lint manualmente

Con hook PostToolUse en Write/Edit:
Usuario → Claude edita archivo → HOOK ejecuta automáticamente "npm run lint --fix"
                               → resultado del hook se muestra a Claude
                               → Claude puede reaccionar si el linter encontró errores

Ciclo con hooks:
┌──────────┐    ┌─────────────┐    ┌──────────────────┐    ┌──────────────┐
│ Mensaje  │───▶│  Claude     │───▶│  Tool Call       │───▶│  Hook        │
│ usuario  │    │  responde   │    │  (Write/Edit/...) │    │  PostToolUse │
└──────────┘    └─────────────┘    └──────────────────┘    └──────┬───────┘
                                                                    │
                                                         shell command ejecuta
                                                                    │
                                                         ┌──────────▼───────┐
                                                         │  resultado →     │
                                                         │  Claude lo ve    │
                                                         └──────────────────┘
```

**Eventos disponibles:**

```
PreToolUse      → Antes de que Claude ejecute una herramienta
                  → Puede BLOQUEAR la ejecución (exit code != 0)

PostToolUse     → Después de que Claude ejecute una herramienta
                  → Claude recibe el output como feedback

UserPromptSubmit → Cuando el usuario envía un mensaje
                  → Se ejecuta antes de que Claude procese el mensaje

Stop            → Cuando Claude termina de responder
                  → Para limpiar, notificar, etc.
```

**Configurar hooks en settings.json:**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint -- --fix \"$CLAUDE_TOOL_INPUT_FILE_PATH\""
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo '[Hook] Comando bash: '\"$CLAUDE_TOOL_INPUT_COMMAND\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude terminó' 'La respuesta está lista'"
          }
        ]
      }
    ]
  }
}
```

**Variables de entorno disponibles en hooks:**

```bash
# En hooks PostToolUse/PreToolUse:
$CLAUDE_TOOL_NAME              # Nombre de la herramienta: "Write", "Edit", "Bash"
$CLAUDE_TOOL_INPUT_FILE_PATH   # Ruta del archivo (para Write/Edit)
$CLAUDE_TOOL_INPUT_COMMAND     # Comando bash (para Bash tool)
$CLAUDE_TOOL_OUTPUT            # Output de la herramienta (PostToolUse)

# En hooks UserPromptSubmit:
$CLAUDE_USER_PROMPT            # El mensaje que escribió el usuario
```

**Casos de uso reales:**

```json
// Caso 1: Formatear automáticamente con Prettier después de cada escritura
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}

// Caso 2: Git add automático después de editar
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "git add \"$CLAUDE_TOOL_INPUT_FILE_PATH\""
          }
        ]
      }
    ]
  }
}

// Caso 3: Bloquear comandos peligrosos con PreToolUse
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$CLAUDE_TOOL_INPUT_COMMAND\" | grep -q 'rm -rf' && echo 'BLOQUEADO: rm -rf no permitido' && exit 1 || exit 0"
          }
        ]
      }
    ]
  }
}

// Caso 4: Log de todas las operaciones
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$(date): $CLAUDE_TOOL_NAME en $CLAUDE_TOOL_INPUT_FILE_PATH\" >> ~/.claude/activity.log"
          }
        ]
      }
    ]
  }
}

// Caso 5: Notificación desktop cuando Claude termina
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "which notify-send >/dev/null && notify-send '🤖 Claude' 'Terminó de responder'"
          }
        ]
      }
    ]
  }
}
```

**Bloquear operaciones con PreToolUse:**

```bash
# Un hook PreToolUse con exit code != 0 BLOQUEA la herramienta
# Claude recibe el mensaje de error y puede ajustar su acción

Ejemplo:
hook: "grep -q 'production' <<< \"$CLAUDE_TOOL_INPUT_COMMAND\" && echo 'ERROR: No ejecutar en producción' && exit 1 || exit 0"

Si Claude intenta: Bash("kubectl apply -f k8s/production.yaml")
→ El hook detecta "production"
→ exit 1 → herramienta bloqueada
→ Claude recibe: "ERROR: No ejecutar en producción"
→ Claude te informa y busca alternativa
```

**Mejores prácticas:**

```
✅ Usar PostToolUse para formatear/lint automáticamente después de escribir
✅ Usar Stop para notificaciones desktop cuando dejas a Claude trabajar solo
✅ Mantener hooks simples y rápidos (se ejecutan en cada tool call)
✅ Usar || true en hooks de formateo (no quieres que fallen si el archivo no aplica)
✅ Loggear operaciones en proyectos críticos

❌ No usar hooks para lógica compleja (úsalos para automatización simple)
❌ No hacer hooks que tarden mucho (ralentizan cada tool call)
❌ No bloques operaciones que Claude necesita para trabajar normalmente
❌ No usar hooks que modifiquen los mismos archivos que Claude está editando (conflictos)
```
