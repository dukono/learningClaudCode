# 7. /model - Cambiar Modelo

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 7. /model
[⬆️ Top](#7-model---cambiar-modelo)

**¿Qué hace?**
Cambia el modelo de IA activo para la sesión actual. Claude Code soporta los modelos de la familia Claude, con diferentes trade-offs entre velocidad, capacidad y costo.

**Modelos disponibles (familia Claude 4.x):**

```
Modelo                  ID                        Velocidad  Capacidad  Costo
──────────────────────  ────────────────────────  ─────────  ─────────  ─────
Claude Haiku 4.5        claude-haiku-4-5          ●●●●●      ●●○○○      $
Claude Sonnet 4.6       claude-sonnet-4-6         ●●●●○      ●●●●○      $$
Claude Opus 4.6         claude-opus-4-6           ●●●○○      ●●●●●      $$$

Vía AWS Bedrock (prefijo bedrock/):
bedrock/claude-sonnet-4-6
bedrock/claude-haiku-4-5-20251001
bedrock/claude-opus-4-6
```

**Uso práctico:**

```bash
/model                              # Muestra modelo actual y opciones

# Durante la sesión:
/model claude-haiku-4-5             # Cambiar a Haiku (más rápido)
/model claude-opus-4-6              # Cambiar a Opus (más capaz)
/model claude-sonnet-4-6            # Volver a Sonnet (balance)

# Al iniciar Claude Code:
claude --model claude-opus-4-6
claude --model claude-haiku-4-5

# En settings.json (persistente):
{
  "model": "claude-sonnet-4-6"
}
```

**Cuándo usar cada modelo:**

```
Claude Haiku 4.5 → Tareas simples y rápidas
  ✓ Buscar un archivo específico
  ✓ Responder preguntas sobre sintaxis
  ✓ Formatear o reformatear código simple
  ✓ Cuando la velocidad es prioritaria

Claude Sonnet 4.6 → Uso diario (recomendado)
  ✓ La mayoría de tareas de ingeniería
  ✓ Refactoring moderado
  ✓ Debugging estándar
  ✓ Balance ideal entre velocidad y calidad

Claude Opus 4.6 → Tareas complejas y críticas
  ✓ Arquitectura y diseño de sistemas
  ✓ Debugging de problemas muy complejos
  ✓ Análisis de código crítico de seguridad
  ✓ Cuando la calidad es más importante que la velocidad
```

**Mejores prácticas:**

```
✅ Usar Sonnet 4.6 como default para el día a día
✅ Cambiar a Haiku para exploración rápida de archivos
✅ Cambiar a Opus para decisiones arquitectónicas importantes
✅ Configurar el modelo en settings.json para persistir

❌ No usar Opus para todas las tareas (es más lento y costoso)
❌ No usar Haiku para análisis complejos (puede ser impreciso)
```
