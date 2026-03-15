# 3. /clear - Limpiar Conversación

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 3. /clear
[⬆️ Top](#3-clear---limpiar-conversación)

**¿Qué hace?**
Borra completamente el historial de mensajes de la sesión actual. Claude olvida todo lo que se dijo en la conversación, pero **no modifica ningún archivo** y no afecta CLAUDE.md ni los archivos de memoria.

**Funcionamiento interno:**

```
Antes de /clear:
┌────────────────────────────────────┐
│ System Prompt (CLAUDE.md, tools)   │ ← se mantiene
│ Message 1: "lee el archivo X"      │
│ Message 2: (contenido del archivo) │
│ Message 3: "modifica la función Y" │ ← todo esto
│ Message 4: (código modificado)     │    se elimina
│ Message 5: "ahora añade tests"     │
└────────────────────────────────────┘

Después de /clear:
┌────────────────────────────────────┐
│ System Prompt (CLAUDE.md, tools)   │ ← intacto
│ [conversación vacía]               │
└────────────────────────────────────┘

→ Los archivos modificados en disco NO se revierten
→ CLAUDE.md se sigue cargando en la próxima interacción
→ MEMORY.md no se borra
```

**Uso práctico:**

```bash
/clear          # Limpia el historial y empieza sesión fresca

# Equivalente con atajo de teclado:
Ctrl+L
```

**Cuándo usar /clear vs /compact:**

```
/clear                              /compact
──────────────────────────────────  ──────────────────────────────────
Borra TODO el historial             Resume el historial en un resumen
Claude olvida el contexto           Claude retiene lo esencial
Empieza completamente de cero       Continúa con contexto comprimido
Útil cuando la sesión se desvió     Útil cuando el contexto crece mucho
Libera tokens al máximo             Libera tokens parcialmente

Úsalo cuando:                       Úsalo cuando:
- Cambias de tarea completamente    - Sigues con la misma tarea
- La conversación se confundió      - Solo necesitas más espacio
- Quieres contexto completamente    - Quieres mantener el hilo
  limpio para nueva tarea             de la conversación
```

**Casos de uso reales:**

```bash
# 1. Cambiar de tarea: terminé con el backend, ahora frontend
/clear
"Ahora necesito crear el componente de login en React"

# 2. Claude se confundió y está dando respuestas incorrectas
/clear
"Empecemos de nuevo. El problema es: [descripción clara]"

# 3. Antes de una tarea importante que necesita contexto limpio
/clear
"Lee el archivo CLAUDE.md y luego revisa src/auth/"
```

**Mejores prácticas:**

```
✅ Usar /clear al cambiar completamente de área de trabajo
✅ Usar /clear si Claude empieza a confundirse o alucinr
✅ Es seguro: no modifica ningún archivo

❌ No usar /clear si quieres mantener el contexto de la tarea actual
❌ No confundir con deshacer cambios en archivos (no los revierte)
```
