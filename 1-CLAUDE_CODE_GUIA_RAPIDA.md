# ⚡ Claude Code: Guía Rápida de Referencia

## 🚀 Instalación y Primer Arranque

```bash
# 1. Instalar (requiere Node.js 18+)
npm install -g @anthropic-ai/claude-code

# 2. Configurar API Key (obtener en console.anthropic.com)
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.bashrc && source ~/.bashrc

# 3. Abrir sesión interactiva en tu proyecto
cd /mi/proyecto && claude

# 4. Primera vez en un proyecto: generar CLAUDE.md
/init

# 5. Configurar atajos de teclado (solo una vez)
/terminal-setup
```

---

## 📋 Todos los Comandos Slash

| Comando | Qué hace | Cuándo usarlo |
|---------|----------|---------------|
| `/help` | Lista todos los comandos disponibles | Cuando no recuerdas algo |
| `/init` | Analiza el proyecto y genera CLAUDE.md | Primera vez en un proyecto |
| `/clear` | Borra TODO el contexto de la sesión | Al cambiar completamente de tarea |
| `/compact` | Comprime el contexto manteniendo el hilo | Sesiones largas (>1h o >80k tokens) |
| `/cost` | Tokens usados y costo estimado de la sesión | Control de gastos |
| `/model` | Cambiar el modelo (opus/sonnet/haiku) | Ajustar calidad vs. velocidad |
| `/review` | Code review del git diff actual | Antes de hacer commit |
| `/pr_comments <N>` | Cargar comentarios del PR #N de GitHub | Resolver revisiones de PRs |
| `/doctor` | Diagnóstico del entorno (versiones, conectividad) | Cuando algo no funciona |
| `/terminal-setup` | Configurar atajos de teclado en el terminal | Primera configuración |
| `/agent list` | Ver subagentes activos y su estado | Monitorear tareas paralelas |
| `/agent logs <id>` | Ver log detallado de un subagente | Debug de subagentes |
| `/agent stop <id>` | Detener un subagente en curso | Cancelar trabajo en progreso |

---

## 🎯 Fórmulas de Prompt Probadas

### Refactoring
```
Refactoriza [archivo] para [objetivo concreto].
No cambies [qué no tocar].
Después ejecuta [comando de tests] y verifica que pasan.
```

### Generar Tests
```
Genera tests para [clase/función] en [ruta destino].
Framework: [pytest/jest/junit].
Casos requeridos: [happy path] y [casos de error].
Mockear: [dependencias externas].
Patrón de nombres: [test_método_caso].
```

### Debugging con Stack Trace
```
Error en [función]:
[pegar error + stack trace completo]

Ocurre cuando: [condición].
1. Reproduce en un test que falle.
2. Corrígelo.
3. El test debe pasar.
```

### Análisis sin Cambios
```
Analiza [qué] y dime [qué información necesito].
NO modifiques ningún archivo. Solo lee y reporta.
```

### Migración Controlada
```
Migra [qué] a [nuevo framework/librería].
Hazlo paso a paso. Espera mi "ok" entre cada paso.
Primero muéstrame el plan completo.
```

### Plan Primero, Ejecución Después
```
[Describe la tarea]
IMPORTANTE: Muéstrame el plan detallado (qué archivos
tocas, cómo, qué tests ejecutas) antes de escribir
una sola línea de código. Espera mi aprobación.
```

---

## 🤖 Activar Subagentes

```
# Forma natural (Claude decide cómo dividir):
"Usa subagentes para procesar en paralelo los 8 módulos de src/services/"

# Forma explícita (tú controlas la división):
"Organiza el trabajo así en paralelo:
 - Subagente A: refactoriza src/auth/ (no toca interfaz pública)
 - Subagente B: refactoriza src/api/ (mantiene endpoints iguales)
 - Subagente C: actualiza tests/ para los cambios anteriores
 Cuando terminen los tres, ejecuta pytest tests/ -v"
```

**Usar subagentes cuando:**
- ✅ Las tareas son independientes entre sí
- ✅ Hay múltiples módulos a procesar
- ✅ El proyecto es grande y el contexto se llenaría
- ❌ Las tareas tienen dependencias entre sí
- ❌ Es una sola tarea pequeña y acotada

---

## 🔌 Configurar MCP (bases de datos y servicios externos)

```json
// .mcp.json en la raíz del proyecto
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://user:pass@localhost/db"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..."
      }
    }
  }
}
```

**MCP servers disponibles:** https://github.com/modelcontextprotocol/servers

---

## 🪝 Configurar Hooks Útiles

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [{
          "type": "command",
          "command": "if [[ '${file}' == *.py ]]; then ruff check '${file}' --fix; fi"
        }]
      }
    ],
    "Stop": [
      {
        "hooks": [{
          "type": "command",
          "command": "notify-send 'Claude Code' 'Tarea completada'"
        }]
      }
    ]
  }
}
```

---

## 🔒 Flags de Seguridad del CLI

```bash
# Solo lectura (máxima seguridad, exploración sin riesgos)
claude --allowedTools "Read"

# Leer y escribir (sin ejecución de comandos)
claude --allowedTools "Read,Write"

# Sin internet (trabajar offline)
claude --disallowedTools "WebSearch"

# Sin confirmaciones ⚠️ (solo CI/CD, nunca en máquina local)
claude --dangerously-skip-permissions --allowedTools "Read,Bash" --print "tarea"
```

---

## 💰 Cuándo Usar Cada Modelo

| ¿Qué necesitas hacer? | Modelo | Por qué |
|-----------------------|--------|---------|
| Pregunta rápida, explorar código | `haiku` | Rápido y muy barato |
| Refactoring, tests, debugging | `sonnet` | Mejor balance diario |
| Arquitectura, análisis profundo, código crítico | `opus` | Máxima calidad |

Cambiar: escribe `/model` dentro de la sesión.

---

## 📄 Plantilla CLAUDE.md para Proyecto Nuevo

```markdown
# CLAUDE.md — [Nombre del Proyecto]

## ¿Qué hace este proyecto?
[Una o dos frases describiendo el propósito]

## Stack Tecnológico
- Lenguaje: [Python 3.11 / Node.js 20 / Java 21...]
- Framework: [FastAPI / Express / Spring Boot...]
- Base de datos: [PostgreSQL / MongoDB / MySQL...]
- Tests: [pytest / Jest / JUnit...]
- Linting: [ruff+black / ESLint+Prettier...]

## Comandos Esenciales
- Tests:   [pytest tests/ -v]
- Lint:    [ruff check src/ --fix]
- Dev:     [uvicorn main:app --reload]
- Build:   [npm run build]

## Estructura del Proyecto
[directorio]/    ← qué hay aquí
[directorio]/    ← qué hay aquí

## Convenciones de Código
- [Convención 1: nombres, patrones, etc.]
- [Convención 2]

## Reglas de Arquitectura
- [Regla: "los routers no importan repositorios directamente"]
- [Regla: "los services no hacen queries SQL directamente"]

## Archivos NO Modificar Sin Aprobación
- [ruta/archivo_crítico.py]
- [ruta/migraciones/]

## Contexto de Negocio (para que Claude entienda las reglas)
[Breve descripción del dominio. Ej: "el stock nunca puede ser negativo"]
```

---

## 🔄 Flujo de Trabajo Recomendado

```
PRIMERA VEZ EN PROYECTO:
  cd /proyecto && claude
  /init → revisar y mejorar el CLAUDE.md generado

INICIO DE CADA SESIÓN:
  cd /proyecto && claude → CLAUDE.md se carga automáticamente

PARA CADA TAREA:
  1. Describe la tarea con contexto completo
  2. Si afecta >3 archivos: pide el PLAN primero
  3. Revisa el plan y da el "ok"
  4. Claude ejecuta
  5. Revisa cambios en git diff antes de commit

GESTIÓN DE LA SESIÓN:
  /cost → cuando quieres saber cuánto llevas
  /compact → si /cost muestra >80k tokens (misma tarea)
  /clear → si cambias completamente de tarea
```

---

## ⚠️ Revisar SIEMPRE Antes de Commit

```
🔴 SIEMPRE revisar (alto riesgo):
   Auth/autorización · Pagos · Migraciones DB
   Configs de seguridad · Datos personales (GDPR)

🟡 Revisar si hay tiempo (riesgo medio):
   Refactoring de módulos core · Cambios en modelo de datos

🟢 Revisión opcional (bajo riesgo):
   Tests unitarios · Docstrings · Linting · Scripts de utilidad
```

---

## 🔗 Recursos

| Recurso | URL |
|---------|-----|
| Documentación oficial | https://docs.anthropic.com/claude-code |
| Changelog y releases | https://github.com/anthropics/claude-code/releases |
| MCP Servers | https://github.com/modelcontextprotocol/servers |
| Precios API | https://www.anthropic.com/pricing |
| Console Anthropic | https://console.anthropic.com |

