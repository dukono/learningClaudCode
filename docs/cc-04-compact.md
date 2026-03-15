# 4. /compact - Comprimir Contexto

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 4. /compact
[⬆️ Top](#4-compact---comprimir-contexto)

**¿Qué hace?**
Comprime el historial de la conversación en un resumen conciso, liberando espacio en el context window sin perder el hilo de la tarea. Claude genera un resumen de lo más importante antes de borrar los mensajes detallados.

**Funcionamiento interno:**

```
Antes de /compact (35k tokens usados):
┌─────────────────────────────────────────┐
│ System Prompt                    3.7k   │
│ System Tools                    19.8k   │
│ Messages (historial completo)    9.5k   │
│   - Mensaje 1: pregunta          0.1k   │
│   - Mensaje 2: lectura archivo   2.1k   │
│   - Mensaje 3: análisis          1.3k   │
│   - Mensaje 4: código generado   3.2k   │
│   - Mensaje 5: iteraciones       2.8k   │
│ Free space                     142k    │
└─────────────────────────────────────────┘

Claude ejecuta internamente:
"Resume los puntos clave de esta conversación para continuar el trabajo"

Después de /compact (27k tokens usados):
┌─────────────────────────────────────────┐
│ System Prompt                    3.7k   │
│ System Tools                    19.8k   │
│ Messages (resumen comprimido)    1.5k   │
│   - Resumen: "Estábamos          1.5k   │
│     refactorizando auth.ts,              │
│     ya completamos login(),              │
│     falta logout() y tests"             │
│ Free space                     152k    │
└─────────────────────────────────────────┘
```

**Uso práctico:**

```bash
/compact                           # Comprime con resumen automático

/compact enfócate en los cambios   # Comprime con instrucción
          pendientes en el auth    # Claude prioriza eso en el resumen

/compact keep: lista de TODOs      # Instrucción para preservar algo específico
```

**Autocompact automático:**

```
Claude Code tiene un buffer de autocompact (≈33k tokens en el ejemplo).
Cuando el contexto se acerca al límite:

1. Claude avisa: "El contexto está casi lleno"
2. Si no haces /compact manualmente...
3. Se activa autocompact automáticamente
4. Igual que /compact pero sin que lo pidas

Para ver el buffer de autocompact:
/context
→ busca la línea "Autocompact buffer: Xk tokens"
```

**Cuándo usar /compact:**

```
Señales de que necesitas /compact:

✓ /context muestra >80% de uso
✓ Las respuestas se vuelven más lentas
✓ Claude empieza a olvidar cosas de antes en la sesión
✓ Ves el warning "contexto casi lleno"
✓ Llevas mucho tiempo en la misma tarea larga
```

**Casos de uso reales:**

```bash
# 1. Tarea larga de refactoring - comprimir para continuar
/compact
"Continuamos. ¿Qué falta por refactorizar?"

# 2. Compactar preservando lista de TODOs específica
/compact importante: aún falta migrar users.ts, orders.ts y tests de integración

# 3. Antes de cambiar de subtarea en el mismo proyecto
/compact
"Ahora vamos con los tests del módulo que refactorizamos"

# 4. Compactar con foco en errores pendientes
/compact prioriza: los 3 errores de TypeScript que encontramos
```

**Diferencia con /clear:**

```
/compact                    /clear
──────────────────          ──────────────────
Retiene resumen             Borra TODO
Continúa la tarea           Empieza de cero
Libera ~70-80% de tokens    Libera 100% de tokens
de la conversación          de la conversación
```

**Mejores prácticas:**

```
✅ Usar /compact antes de llegar al límite (no esperar el autocompact)
✅ Dar instrucciones específicas: /compact mantén la lista de archivos pendientes
✅ Usar en sesiones largas de debugging o refactoring
✅ /compact es reversible en concepto (los archivos no cambian)

❌ No usar /compact si prefieres empezar fresco (/clear es mejor)
❌ No asumir que Claude recordará detalles específicos post-compact
❌ No usar en medio de una operación multi-paso compleja sin verificar
```
