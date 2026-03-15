# 15. Modos de Permisos y Seguridad

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 15. Permisos y Seguridad
[⬆️ Top](#15-modos-de-permisos-y-seguridad)

**¿Qué hace?**
Claude Code implementa un sistema de permisos que controla qué herramientas puede ejecutar de forma autónoma y cuáles requieren aprobación explícita del usuario. El objetivo es balancear velocidad de trabajo con protección contra operaciones destructivas o irreversibles.

**Los tres modos de operación:**

```
Modo 1: Default (pide aprobación para herramientas peligrosas)
──────────────────────────────────────────────────────────────
claude                                 ← inicio normal

Claude evalúa cada tool call:
  Herramientas seguras → ejecuta automáticamente
  Herramientas potencialmente peligrosas → pide confirmación

Ejemplo:
  Read("archivo.ts")     → automático ✓
  Bash("npm test")       → automático ✓
  Bash("rm -rf dist/")   → pide confirmación ⚠️
  Write("archivo.ts")    → pide confirmación ⚠️ (primer uso)


Modo 2: Pre-aprobado (allowedTools en settings.json)
──────────────────────────────────────────────────────────────
settings.json:
{
  "allowedTools": ["Read", "Write", "Edit", "Bash", "Glob", "Grep"]
}

→ Las herramientas listadas se ejecutan SIN confirmación
→ Útil para proyectos de confianza donde el overhead molesta
→ Más rápido pero menos protección


Modo 3: Sin permisos (--dangerously-skip-permissions)
──────────────────────────────────────────────────────────────
claude --dangerously-skip-permissions

→ TODAS las herramientas se ejecutan sin confirmación
→ Solo para CI/CD o entornos aislados y controlados
→ NUNCA en producción o con datos reales importantes
```

**Blast radius assessment (cómo Claude evalúa riesgo):**

```
Claude clasifica operaciones antes de ejecutarlas:

SEGURO - Reversible y local:
✓ Leer archivos (Read, Glob, Grep)
✓ Buscar en internet (WebFetch, WebSearch)
✓ Ejecutar tests (Bash "npm test")
✓ Ver estado git (Bash "git status")
→ Ejecuta automáticamente

MODERADO - Irreversible pero local:
⚠ Escribir archivos nuevos (Write)
⚠ Modificar archivos existentes (Edit)
⚠ Ejecutar comandos de build
→ Pide confirmación en primer uso

PELIGROSO - Destructivo o afecta sistemas compartidos:
🛑 Eliminar archivos (Bash "rm")
🛑 Force push (Bash "git push --force")
🛑 Modificar base de datos en producción
🛑 Enviar emails, mensajes, webhooks
🛑 Modificar infraestructura (CI/CD, cloud)
→ SIEMPRE pide confirmación + describe consecuencias
```

**Configurar allowedTools:**

```json
// .claude/settings.json - proyecto de confianza
{
  "allowedTools": [
    "Read",
    "Write",
    "Edit",
    "Bash",
    "Glob",
    "Grep",
    "WebFetch"
  ]
}

// .claude/settings.json - proyecto sensible (solo lectura)
{
  "allowedTools": ["Read", "Glob", "Grep"],
  "blockedTools": ["Write", "Edit", "Bash"]
}

// .claude/settings.json - con herramientas MCP específicas
{
  "allowedTools": [
    "Read", "Glob", "Grep",
    "mcp__ide__getDiagnostics"
  ]
}
```

**Operaciones que Claude SIEMPRE confirma:**

```
Sin importar el modo de permisos, Claude verifica antes de:

Git destructivo:
  git push --force        → "Esto sobreescribirá el historial remoto"
  git reset --hard        → "Esto descartará todos los cambios locales"
  git checkout -- .       → "Esto revertirá todos los cambios del working tree"
  git branch -D           → "Esto borrará la rama permanentemente"

Filesystem:
  rm -rf                  → "Esto borrará X archivos/directorios"
  Borrar archivos con datos importantes

Sistemas externos:
  Push a repositorios remotos
  Crear/cerrar PRs o issues
  Enviar mensajes (Slack, email)
  Modificar infraestructura
```

**Casos de uso reales:**

```bash
# Caso 1: Desarrollo local fluido (pre-aprobar herramientas comunes)
# .claude/settings.json
{
  "allowedTools": ["Read", "Write", "Edit", "Bash", "Glob", "Grep"]
}
→ Claude trabaja sin interrupciones en tu máquina local

# Caso 2: Proyecto de cliente (máxima seguridad)
# .claude/settings.json
{
  "blockedTools": ["Bash", "Write", "Edit"]
}
→ Claude solo puede leer y analizar, no modificar nada

# Caso 3: Pipeline CI (automático)
claude --dangerously-skip-permissions "ejecuta los tests y crea un reporte"
→ Sin confirmaciones, perfecto para automatización

# Caso 4: Denegar una operación
# Si Claude pide permiso para algo que no quieres:
# → Click en "Deny" o "No"
# Claude buscará alternativas o te explicará por qué lo necesitaba
```

**Mejores prácticas:**

```
✅ Usar allowedTools para el workflow de tu proyecto (mejor UX)
✅ Revisar siempre operaciones destructivas antes de aprobarlas
✅ Usar --dangerously-skip-permissions solo en CI o entornos aislados
✅ Agregar settings.local.json a .gitignore para configs personales

❌ No usar --dangerously-skip-permissions en proyectos de producción
❌ No pre-aprobar operaciones destructivas en proyectos de clientes
❌ No aprobar ciegamente: leer siempre qué hace Claude antes de confirmar
❌ No bloquear Read/Glob/Grep (Claude los necesita para cualquier análisis)
```
