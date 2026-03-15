# CLAUDE CODE - GUÍA PRÁCTICA DE COMANDOS Y USO

> **Objetivo:** Guía completa y práctica de Claude Code CLI, sus comandos slash, configuración, herramientas y flujos de trabajo, con ejemplos del mundo real y mejores prácticas.

---

## 📚 Tabla de Contenidos

### CONFIGURACIÓN
0. [CLAUDE.md - Configuración del Proyecto](docs/cc-00-claude-md.md)
1. [settings.json - Configuración Global](docs/cc-01-settings.md)

### COMANDOS ESENCIALES
2. [/help - Ayuda y Referencia](docs/cc-02-help.md)
3. [/clear - Limpiar Conversación](docs/cc-03-clear.md)
4. [/compact - Comprimir Contexto](docs/cc-04-compact.md)
5. [/context - Ver Uso de Contexto](docs/cc-05-context.md)
6. [/status - Estado de Claude Code](docs/cc-06-status.md)
7. [/model - Cambiar Modelo](docs/cc-07-model.md)

### GESTIÓN DE CONTEXTO Y MEMORIA
8. [/add-dir - Agregar Directorios](docs/cc-08-add-dir.md)
9. [/memory - Gestión de Memoria Persistente](docs/cc-09-memory.md)

### HERRAMIENTAS Y EXTENSIONES
10. [/mcp - Model Context Protocol Servers](docs/cc-10-mcp.md)
11. [/skills - Habilidades Personalizadas](docs/cc-11-skills.md)
12. [/agents - Agentes Especializados](docs/cc-12-agents.md)

### INTEGRACIÓN
13. [/ide - Integración con IDE](docs/cc-13-ide.md)

### FLUJO DE TRABAJO GIT
14. [/commit - Crear Commits con IA](docs/cc-14-commit.md)

### SEGURIDAD Y PERMISOS
15. [Modos de Permisos y Seguridad](docs/cc-15-permisos.md)

### AUTOMATIZACIÓN
16. [Hooks - Automatización de Eventos](docs/cc-16-hooks.md)

---

## 📖 INTRODUCCIÓN

Esta guía cubre los **comandos y conceptos esenciales de Claude Code**, el CLI agentic de Anthropic para ingeniería de software. Cada sección incluye:

✅ **Funcionamiento interno** — qué hace Claude Code bajo el capó
✅ **Opciones y variantes** — uso básico a avanzado
✅ **Casos de uso reales** — ejemplos del mundo profesional
✅ **Mejores prácticas** — qué hacer y qué evitar
✅ **Troubleshooting** — problemas comunes y soluciones

---

## 🔗 Documentación Relacionada

- [CLAUDE_CODE_FUNCIONAMIENTO_INTERNO.md](CLAUDE_CODE_FUNCIONAMIENTO_INTERNO.md) — Arquitectura interna, LLM, context window, tool use
- [GIT_COMANDOS_GUIA_PRACTICA.md](GIT_COMANDOS_GUIA_PRACTICA.md) — Guía de Git (proyecto relacionado)
- Documentación oficial: https://docs.anthropic.com/claude-code

---

## 🚀 Inicio Rápido

```bash
# Instalar Claude Code
npm install -g @anthropic-ai/claude-code

# Iniciar en tu proyecto
cd mi-proyecto
claude

# Con modelo específico
claude --model claude-sonnet-4-6

# Sin confirmaciones (modo CI)
claude --dangerously-skip-permissions
```

---

## 📂 Estructura de Archivos de Configuración

```
~/.claude/
├── settings.json          # Configuración global
├── CLAUDE.md              # Instrucciones globales
├── skills/                # Skills personales
├── agents/                # Agentes personalizados
└── projects/
    └── [hash-proyecto]/
        └── memory/
            └── MEMORY.md  # Memoria persistente del proyecto

proyecto/
├── .claude/
│   ├── settings.json      # Config local (override global)
│   └── skills/            # Skills del proyecto
└── CLAUDE.md              # Instrucciones del proyecto
```
