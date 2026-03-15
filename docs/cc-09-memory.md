# 9. /memory - Gestión de Memoria Persistente

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 9. /memory
[⬆️ Top](#9-memory---gestión-de-memoria-persistente)

**¿Qué hace?**
Gestiona los archivos de memoria persistente de Claude. A diferencia de la conversación (que se borra con /clear), la memoria persiste entre sesiones. Claude puede actualizar sus propios archivos de memoria para recordar cosas en conversaciones futuras.

**Funcionamiento interno:**

```
Directorio de memoria (por proyecto):
~/.claude/projects/[hash-del-path-del-proyecto]/memory/

Archivos típicos:
├── MEMORY.md          ← Cargado automáticamente en cada sesión
├── debugging.md       ← Notas de debugging del proyecto
├── patterns.md        ← Patrones de código descubiertos
└── decisions.md       ← Decisiones arquitectónicas tomadas

MEMORY.md se inyecta en el system prompt automáticamente.
Límite efectivo: primeras 200 líneas (el resto se trunca).

Ciclo de uso:
┌──────────────────┐
│  Sesión 1        │ → Claude aprende que "siempre usar pnpm, no npm"
│  Claude trabaja  │ → Escribe en MEMORY.md: "# Package manager: pnpm"
└──────────────────┘

┌──────────────────┐
│  Sesión 2        │ → MEMORY.md se carga automáticamente
│  Claude ya sabe  │ → "Recuerdo que usas pnpm, no npm"
└──────────────────┘
```

**Uso práctico:**

```bash
/memory                    # Abre el archivo MEMORY.md en el editor

# Claude actualiza la memoria automáticamente cuando:
# - El usuario pide "recuerda que..."
# - Claude aprende algo estable sobre el proyecto
# - Se resuelve un bug recurrente con solución específica
```

**Ejemplo de MEMORY.md bien estructurado:**

```markdown
# Proyecto: mi-api

## Stack confirmado
- Node 20 + TypeScript 5 + Express
- PostgreSQL + Prisma ORM
- pnpm (NO usar npm ni yarn)
- Tests: Vitest

## Comandos clave
- Dev: pnpm dev
- Tests: pnpm test --run
- Build: pnpm build
- Lint+fix: pnpm lint --fix

## Patrones del proyecto
- Validación con Zod en todos los endpoints
- DTOs en src/types/, no inline
- Errores: throw new AppError(message, statusCode)

## Bugs resueltos anteriormente
- Prisma + Jest: necesita `--testEnvironment node` en vitest.config
- Hot reload no funciona con nodemon en WSL2, usar tsx watch

## Preferencias del usuario
- Respuestas concisas, sin explicar lo obvio
- Siempre ejecutar lint antes de decir que algo está listo
```

**Diferencia MEMORY.md vs CLAUDE.md:**

```
MEMORY.md                          CLAUDE.md
─────────────────────────────────  ─────────────────────────────────
Escrito por Claude                 Escrito por el equipo/usuario
Aprendizajes automáticos           Instrucciones explícitas
Fuera del repo (personal)          Dentro del repo (compartido)
~/.claude/projects/[hash]/memory/  ./CLAUDE.md o ~/CLAUDE.md
Se actualiza solo                  Se actualiza manualmente
```

**Casos de uso reales:**

```bash
# Pedirle a Claude que recuerde algo
"Recuerda que en este proyecto siempre hacemos squash merge, nunca merge commit"
# → Claude actualiza MEMORY.md automáticamente

# Pedirle que olvide algo incorrecto
"Olvida lo que tienes en memoria sobre el puerto del servidor, ahora es 8080"

# Ver qué tiene en memoria
/memory
# → abre MEMORY.md

# Limpiar memoria obsoleta
/memory
# → editar manualmente el archivo

# Pedir a Claude que organice su memoria
"Revisa y reorganiza tu MEMORY.md, hay cosas desactualizadas"
```

**Mejores prácticas:**

```
✅ Pedirle a Claude que recuerde decisiones importantes: "recuerda que..."
✅ Revisar /memory periódicamente para limpiar información obsoleta
✅ MEMORY.md debe ser conciso (límite efectivo: 200 líneas)
✅ Guardar en MEMORY.md: comandos específicos, bugs resueltos, preferencias

❌ No guardar secretos o tokens en MEMORY.md
❌ No dejar que MEMORY.md crezca sin control (se trunca en línea 200)
❌ No duplicar lo que ya está en CLAUDE.md
```
