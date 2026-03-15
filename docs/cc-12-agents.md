# 12. /agents - Agentes Especializados

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 12. /agents
[⬆️ Top](#12-agents---agentes-especializados)

**¿Qué hace?**
Lista los agentes personalizados disponibles. Un agente es una instancia de Claude con un system prompt especializado, conjunto de herramientas propio y contexto aislado, lanzado por la sesión principal mediante la herramienta `Agent`.

**Funcionamiento interno:**

```
Sesión principal (tú):
┌─────────────────────────────────┐
│  Claude (Sonnet 4.6)            │
│  Contexto: tu conversación      │
│                                 │
│  Agent tool → lanza subagente   │
└──────────┬──────────────────────┘
           │
    ┌──────▼──────────────────────────────────────────┐
    │  Subagente: unit-test-fixer                      │
    │  Modelo: Sonnet 4.6                              │
    │  Herramientas: Read, Write, Edit, Bash, Glob...  │
    │  Contexto: solo el prompt que le pasaste         │
    │  Trabaja de forma AUTÓNOMA                       │
    │  Puede lanzar sus propios subagentes             │
    └─────────────────────────────────────────────────┘
           │
    Resultado regresa a la sesión principal
```

**Tipos de agentes built-in:**

```
general-purpose     → Agente de propósito general. Default si no especificas tipo.
                      Herramientas: todas disponibles

Explore             → Exploración rápida de codebases. Solo lectura.
                      Herramientas: Glob, Grep, Read, WebFetch, WebSearch
                      Uso: buscar archivos, entender arquitectura

Plan                → Arquitecto de software. Diseña planes de implementación.
                      Herramientas: igual que Explore (no escribe código)
                      Uso: planificar features antes de implementar

unit-test-fixer     → Especializado en crear y arreglar tests.
                      Uso: cuando tests fallan o faltan tests

engineering-solution-architect → Soluciones basadas en best practices.
                      Uso: cuando quieres la solución "correcta", no la rápida

pre-change-analyzer → Analiza riesgos antes de cambios grandes.
                      Uso: antes de refactorings o cambios de producción
```

**Crear un agente personalizado:**

```bash
mkdir -p ~/.claude/agents

cat > ~/.claude/agents/security-reviewer.md << 'EOF'
---
name: security-reviewer
description: Revisa código en busca de vulnerabilidades de seguridad OWASP Top 10
tools: Read, Glob, Grep
model: claude-opus-4-6
---

Eres un experto en seguridad de aplicaciones. Cuando se te asigna código:

1. Busca vulnerabilidades OWASP Top 10:
   - Inyección SQL/NoSQL
   - XSS (Cross-Site Scripting)
   - Broken Authentication
   - Sensitive Data Exposure
   - Broken Access Control
   - Security Misconfiguration
   - Insecure Deserialization

2. Para cada vulnerabilidad encontrada:
   - Archivo y línea específica
   - Descripción del riesgo
   - Ejemplo de explotación
   - Solución recomendada con código

3. Califica severidad: CRÍTICA / ALTA / MEDIA / BAJA

Solo lee código, NO modifiques archivos.
EOF
```

**Uso práctico - Agent tool en sesión:**

```
# La sesión principal lanza subagentes con la herramienta Agent:

"Lanza el agente Explore para encontrar todos los archivos que usan la función deprecated calcPrice()"

"Usa el agente unit-test-fixer para crear tests para el archivo src/auth/login.ts"

"Lanza 3 agentes en paralelo para analizar independientemente:
 - src/auth/
 - src/api/
 - src/db/"
```

**Foreground vs Background:**

```
Foreground (default):
─────────────────────
Agente = Claude Code espera el resultado antes de continuar.
Úsalo cuando necesitas el resultado para tu siguiente paso.

"Usa el agente Explore para encontrar el bug" → esperas → recibes resultado → actúas

Background:
───────────
Agente corre en paralelo mientras tú haces otras cosas.
Recibes notificación cuando termina.

"Lanza 3 agentes en background para analizar los 3 módulos"
→ Lanzas los 3 → sigues trabajando → te notifican cuando terminan
```

**Worktrees con agentes:**

```bash
# Agentes pueden trabajar en worktrees aislados
# (no interfieren con tu rama actual)

Agent tool con isolation: "worktree"
→ Crea un git worktree temporal
→ Agente trabaja en copia aislada del repo
→ Si hace cambios: worktree persiste (puedes revisar)
→ Si no hace cambios: worktree se elimina automáticamente
```

**Mejores prácticas:**

```
✅ Usar agentes para tareas autónomas largas (no bloquean tu sesión)
✅ Dar contexto específico en el prompt del agente (no hereda tu sesión)
✅ Usar Explore para búsquedas antes de implementar
✅ Usar Plan para diseñar antes de codificar
✅ Lanzar múltiples agentes en paralelo para tareas independientes

❌ No lanzar agentes para tareas de 2 minutos (overhead no vale la pena)
❌ No asumir que el agente tiene tu contexto (debes dárselo explícitamente)
❌ No lanzar agentes que editen los mismos archivos en paralelo (conflictos)
```
