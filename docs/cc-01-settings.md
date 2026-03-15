# 1. settings.json - Configuración Global

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 1. settings.json
[⬆️ Top](#1-settingsjson---configuración-global)

**¿Qué hace?**
`settings.json` es el archivo de configuración de Claude Code. Controla el modelo activo, herramientas pre-aprobadas o bloqueadas, variables de entorno, hooks y más. Tiene dos ubicaciones con precedencia: la configuración local del proyecto sobreescribe la global.

**Funcionamiento interno:**

```
Ubicaciones y precedencia:

1. ~/.claude/settings.json         ← Global (siempre se carga)
2. .claude/settings.json           ← Local del proyecto (override)
3. .claude/settings.local.json     ← Local personal (no versionar en git)

Al iniciar Claude Code:
Global → merge con Local → resultado final activo

Ejemplo de merge:
Global:  { "model": "claude-sonnet-4-6", "allowedTools": ["Read"] }
Local:   { "allowedTools": ["Read", "Bash", "Write"] }
Final:   { "model": "claude-sonnet-4-6", "allowedTools": ["Read", "Bash", "Write"] }
```

**Estructura completa de settings.json:**

```json
{
  "model": "claude-sonnet-4-6",

  "allowedTools": [
    "Read",
    "Write",
    "Edit",
    "Bash",
    "Glob",
    "Grep",
    "mcp__ide__getDiagnostics"
  ],

  "blockedTools": [
    "WebSearch"
  ],

  "env": {
    "NODE_ENV": "development",
    "MY_API_KEY": "valor"
  },

  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint -- --fix"
          }
        ]
      }
    ]
  },

  "mcpServers": {
    "ide": {
      "type": "stdio",
      "command": "node",
      "args": ["~/.claude/mcp-servers/ide-server.js"]
    }
  }
}
```

**Opciones principales:**

```bash
# Ver configuración actual
/config
# o
/status

# Editar configuración global
code ~/.claude/settings.json

# Editar configuración local del proyecto
code .claude/settings.json
```

**Casos de uso reales:**

```json
// Caso 1: Proyecto con CI - pre-aprobar todas las herramientas
// .claude/settings.json
{
  "allowedTools": ["Read", "Write", "Edit", "Bash", "Glob", "Grep"]
}

// Caso 2: Proyecto sensible - bloquear escritura de archivos
// .claude/settings.json
{
  "blockedTools": ["Write", "Edit", "Bash"]
}

// Caso 3: Proyecto con variables de entorno específicas
// .claude/settings.local.json  (NO versionar en git)
{
  "env": {
    "DATABASE_URL": "postgresql://localhost/mydb",
    "API_KEY": "mi-clave-local"
  }
}

// Caso 4: Configuración personal global
// ~/.claude/settings.json
{
  "model": "claude-sonnet-4-6",
  "allowedTools": ["Read", "Glob", "Grep"]
}
```

**Herramientas disponibles para allowedTools/blockedTools:**

```
Read              - Leer archivos
Write             - Crear archivos nuevos
Edit              - Modificar archivos existentes
Bash              - Ejecutar comandos shell
Glob              - Buscar archivos por patrón
Grep              - Buscar contenido con regex
WebFetch          - Obtener contenido de URLs
WebSearch         - Buscar en internet
Agent             - Lanzar subagentes
NotebookEdit      - Editar Jupyter notebooks
mcp__*            - Herramientas de MCP servers
```

**Mejores prácticas:**

```
✅ Usar settings.local.json para credenciales (agregar a .gitignore)
✅ Versionar .claude/settings.json en git (sin secretos)
✅ Pre-aprobar herramientas en proyectos de confianza para mayor velocidad
✅ Bloquear herramientas destructivas en proyectos críticos

❌ No poner credenciales en settings.json versionado
❌ No usar --dangerously-skip-permissions en producción
❌ No bloquear Read/Glob/Grep (Claude los necesita para cualquier tarea)
```
