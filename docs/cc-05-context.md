# 5. /context - Ver Uso de Contexto

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 5. /context
[⬆️ Top](#5-context---ver-uso-de-contexto)

**¿Qué hace?**
Muestra el uso actual de tokens del context window, desglosado por categoría. Permite ver exactamente cuánto espacio está consumiendo cada parte: system prompt, herramientas, mensajes, etc.

**Funcionamiento interno:**

```
El context window de claude-sonnet-4-6: 200,000 tokens

Categorías que consumen tokens:
┌─────────────────────────────────────────────────────┐
│  System prompt    │ Instrucciones base del CLI        │
│  System tools     │ Definiciones de todas las tools   │
│  MCP tools        │ Tools de servidores MCP           │
│  Custom agents    │ Definiciones de agentes propios   │
│  Skills           │ Skills cargados actualmente       │
│  Messages         │ Historial de la conversación      │
│  Free space       │ Tokens disponibles                │
│  Autocompact buf. │ Reserva para autocompact          │
└─────────────────────────────────────────────────────┘
```

**Output de /context:**

```
Context Usage
⛁ ⛁ ⛁ ⛁ ⛁ ⛁ ⛁ ⛁ ⛶ ⛶   bedrock/claude-sonnet-4.6 · 35k/200k tokens (18%)

Estimated usage by category:
⛁  System prompt:    3.7k tokens (1.8%)
⛁  System tools:    19.8k tokens (9.9%)
⛁  MCP tools:          61 tokens (0.0%)
⛁  Custom agents:    1.3k tokens (0.7%)
⛁  Skills:           237 tokens (0.1%)
⛶  Free space:       142k (70.9%)
⛝  Autocompact buffer: 33k tokens (16.5%)
```

**Interpretar las barras:**

```
⛁  = tokens usados
⛶  = tokens libres
⛝  = buffer de autocompact (reservado)

Colores:
───────
Gris oscuro  = system prompt + tools (fijo, no cambia)
Celeste      = MCP tools
Lila         = custom agents
Amarillo     = skills
Verde/Blanco = messages (tu conversación)
```

**Qué hacer según el uso:**

```
0-50%   → Todo bien, trabaja normalmente
50-70%  → Considera /compact pronto si la tarea es larga
70-85%  → Usa /compact ahora
85%+    → Autocompact inminente, usa /compact o /clear

El autocompact buffer (16.5%) es el espacio que Claude reserva
para poder ejecutar un /compact de emergencia automático.
```

**Cómo reducir el uso:**

```bash
# 1. Comprimir la conversación
/compact

# 2. Empezar de cero
/clear

# 3. Deshabilitar plugins/skills que no uses
/plugin            # → deshabilita los innecesarios
/reload-plugins

# 4. Reducir agentes personalizados cargados
# Revisar ~/.claude/agents/ y eliminar los que no usas

# 5. Usar modelos con mayor contexto si está disponible
/model claude-opus-4-6
```

**Casos de uso reales:**

```bash
# Verificar contexto antes de una tarea larga
/context
# Si hay <50% libre, hacer /compact primero

# Diagnosticar por qué Claude parece "olvidar" cosas
/context
# Si messages usa mucho → /compact
# Si skills usa mucho → deshabilitar skills no usados

# Optimizar carga de skills
/context
# Ver cuántos tokens usan los skills
# Si son muchos y no los usas → /plugin para deshabilitar
```

**Mejores prácticas:**

```
✅ Revisar /context al inicio de sesiones largas
✅ Hacer /compact cuando messages > 30% del total
✅ Deshabilitar plugins/skills que no uses en el proyecto actual
✅ El autocompact buffer (16%) es sagrado, no lo cuentes como disponible

❌ No ignorar el aviso de contexto casi lleno
❌ No asumir que 200k tokens es infinito para proyectos grandes
```
