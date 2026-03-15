# 2. /help - Ayuda y Referencia

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 2. /help
[⬆️ Top](#2-help---ayuda-y-referencia)

**¿Qué hace?**
Muestra la ayuda de Claude Code: lista de slash commands disponibles, atajos de teclado y referencia rápida. También puedes preguntarle directamente a Claude sobre cómo usar cualquier funcionalidad.

**Uso práctico:**

```bash
/help                    # Muestra ayuda general
/help compact            # Ayuda sobre un comando específico
```

**Slash commands disponibles (referencia rápida):**

```
Conversación:
/clear          - Borra el historial de la conversación actual
/compact        - Comprime el historial en un resumen
/context        - Muestra uso de tokens por categoría

Configuración:
/config         - Abre la configuración de Claude Code
/status         - Muestra estado: modelo, permisos, MCP servers
/model          - Cambia el modelo activo en esta sesión

Contexto:
/add-dir        - Agrega un directorio adicional al workspace
/memory         - Gestiona archivos de memoria persistente

Extensiones:
/mcp            - Lista y gestiona MCP servers
/skills         - Lista skills disponibles
/agents         - Lista agentes personalizados

Integración:
/ide            - Conecta con el IDE (IntelliJ, VSCode)

Git:
/commit         - Crea un commit git con mensaje generado

Sistema:
/help           - Esta ayuda
/exit           - Cierra Claude Code
/reload-plugins - Recarga plugins sin reiniciar
/plugin         - Gestiona plugins habilitados/deshabilitados
```

**Atajos de teclado:**

```
Ctrl+C          - Interrumpir respuesta en curso
Ctrl+L          - Limpiar pantalla (equivale a /clear)
↑ / ↓           - Navegar historial de comandos
Tab             - Autocompletar slash commands
Esc             - Cancelar entrada actual
```

**Preguntar a Claude sobre sus capacidades:**

```
En lugar de /help, puedes preguntarle directamente:

"¿Qué herramientas tienes disponibles?"
"¿Cómo funciona el /compact?"
"¿Puedes buscar en internet?"
"¿Qué es un MCP server?"
```
