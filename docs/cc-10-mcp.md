# 10. /mcp - Model Context Protocol Servers

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 10. /mcp
[⬆️ Top](#10-mcp---model-context-protocol-servers)

**¿Qué hace?**
Lista y gestiona los servidores MCP (Model Context Protocol) conectados. MCP es el protocolo estándar para extender las capacidades de Claude con herramientas personalizadas: acceso a bases de datos, APIs externas, servicios de empresa, etc.

**Funcionamiento interno:**

```
Sin MCP (herramientas built-in):
┌────────────────┐
│  Claude Code   │ → Bash, Read, Write, Edit, Glob, Grep, WebFetch
└────────────────┘

Con MCP servers:
┌────────────────┐     ┌──────────────────────────┐
│  Claude Code   │────▶│  MCP Server: ide          │ → getDiagnostics
└────────────────┘     └──────────────────────────┘
        │              ┌──────────────────────────┐
        └─────────────▶│  MCP Server: github       │ → listPRs, createIssue
                       └──────────────────────────┘
                       ┌──────────────────────────┐
        └─────────────▶│  MCP Server: postgres      │ → queryDB, listTables
                       └──────────────────────────┘

Las herramientas MCP aparecen en /context como "mcp__servidor__herramienta"
Ejemplo: mcp__ide__getDiagnostics
```

**Tipos de MCP servers:**

```
stdio (proceso local):
─────────────────────
Claude Code lanza un subproceso y se comunica via stdin/stdout.
Útil para: herramientas locales, acceso a filesystem, CLIs

HTTP/SSE (servidor remoto):
───────────────────────────
Claude Code se conecta a un servidor HTTP que implementa el protocolo MCP.
Útil para: APIs remotas, servicios cloud, servidores compartidos en equipo
```

**Configurar MCP en settings.json:**

```json
{
  "mcpServers": {
    "ide": {
      "type": "stdio",
      "command": "node",
      "args": ["/ruta/al/ide-mcp-server.js"]
    },
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "tu-token-aqui"
      }
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_URL": "postgresql://localhost/mi-db"
      }
    }
  }
}
```

**Uso práctico:**

```bash
/mcp                    # Lista servidores MCP y su estado

# Output típico:
# MCP Servers:
# ✓ ide          connected    (1 tool: getDiagnostics)
# ✗ github       error        (GITHUB_TOKEN not set)
# ✓ postgres     connected    (5 tools: query, list_tables, ...)
```

**MCP Servers populares:**

```bash
# IDE (IntelliJ/VSCode) - Diagnósticos de compilación
# Viene incluido con /ide

# Filesystem MCP - Acceso extendido a archivos
npx @modelcontextprotocol/server-filesystem /ruta/permitida

# GitHub MCP - Issues, PRs, repos
npx @modelcontextprotocol/server-github

# PostgreSQL MCP - Consultas a BD
npx @modelcontextprotocol/server-postgres

# Fetch MCP - HTTP requests mejorados
npx @modelcontextprotocol/server-fetch

# Brave Search MCP - Búsqueda web
npx @modelcontextprotocol/server-brave-search
```

**Casos de uso reales:**

```bash
# Con MCP de GitHub conectado:
"Lista los PRs abiertos y dime cuál tiene más comentarios pendientes"

# Con MCP de PostgreSQL:
"¿Cuántos usuarios se registraron esta semana?"
"Busca todos los pedidos pendientes de hace más de 7 días"

# Con MCP del IDE:
"¿Hay errores de TypeScript en el proyecto?"
→ Claude llama mcp__ide__getDiagnostics y lee los errores directamente
```

**Mejores prácticas:**

```
✅ Usar MCP del IDE para que Claude vea errores de compilación en tiempo real
✅ Configurar en settings.local.json (no versionar tokens en git)
✅ Verificar estado con /mcp antes de tareas que dependen del servidor

❌ No poner tokens de API directamente en settings.json versionado
❌ No conectar servidores MCP no confiables (tienen acceso a ejecutar código)
```
