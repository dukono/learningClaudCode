# 6. /status - Estado de Claude Code

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 6. /status
[⬆️ Top](#6-status---estado-de-claude-code)

**¿Qué hace?**
Muestra un panel con el estado actual de la configuración de Claude Code: modelo activo, modo de permisos, MCP servers conectados, directorio de trabajo y otras configuraciones relevantes.

**Uso práctico:**

```bash
/status         # Muestra panel de estado
/config         # Abre configuración interactiva (alternativa)
```

**Información mostrada:**

```
Estado típico de /status:

Model:           claude-sonnet-4-6
Provider:        bedrock (AWS)
Working dir:     /home/user/mi-proyecto
CLAUDE.md:       ✓ Loaded (247 tokens)
Permission mode: default (asks for approval)

MCP Servers:
  ✓ ide          connected
  ✗ github       not configured

Tools allowed:   (default - all with approval)
Tools blocked:   none

Hooks:           none configured
```

**Cuándo usar /status:**

```
✓ Al inicio de sesión para verificar que todo está configurado
✓ Para confirmar qué modelo está activo
✓ Para verificar si el IDE está conectado
✓ Para revisar modo de permisos antes de operaciones sensibles
✓ Para diagnosticar por qué Claude no puede usar cierta herramienta
```

**Casos de uso reales:**

```bash
# Verificar antes de tarea importante
/status
# Confirmar: modelo correcto, directorio correcto, CLAUDE.md cargado

# Diagnosticar problema con herramientas bloqueadas
/status
# Revisar si alguna tool está en blockedTools

# Verificar conexión con IDE
/status
# Ver si "ide" MCP server dice "connected" o error
```
