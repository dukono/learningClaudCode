# 13. /ide - Integración con IDE

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 13. /ide
[⬆️ Top](#13-ide---integración-con-ide)

**¿Qué hace?**
Conecta Claude Code con el IDE activo (IntelliJ IDEA o VS Code), habilitando un servidor MCP que expone los diagnósticos del editor (errores de compilación, warnings de linting, errores de TypeScript) directamente a Claude.

**Funcionamiento interno:**

```
Sin /ide:
┌────────────────┐     ┌────────────────────┐
│  Claude Code   │     │  IntelliJ/VSCode    │
│                │     │  Error: "Property   │
│  (no sabe de  │     │  'foo' does not     │  ← Claude no puede ver esto
│   los errores)│     │  exist on type X"   │
└────────────────┘     └────────────────────┘

Con /ide:
┌────────────────┐     ┌────────────────────┐
│  Claude Code   │────▶│  IDE MCP Server    │
│                │     │  (plugin del IDE)  │
│  Puede llamar  │◀────│                    │
│  getDiagno-    │     │  Lee errores LSP   │
│  stics()       │     │  del editor        │
└────────────────┘     └────────────────────┘

Claude llama: mcp__ide__getDiagnostics(uri: "src/auth/login.ts")
Recibe:       Lista de errores exactos con línea, columna, mensaje
```

**Requisitos:**

```
IntelliJ IDEA:
  → Instalar plugin "Claude Code" desde Marketplace
  → Abrir proyecto en IntelliJ
  → Ejecutar /ide en Claude Code
  → Ejecutar /reload-plugins para activar

VS Code:
  → Instalar extensión "Claude Code" desde Extensions
  → Abrir proyecto en VS Code
  → Ejecutar /ide en Claude Code
```

**Uso práctico:**

```bash
/ide                    # Conecta con el IDE activo

# Verificar conexión:
/mcp
# Debe mostrar: ✓ ide   connected

# Usar la herramienta:
"¿Hay errores de TypeScript en el proyecto?"
# Claude llama getDiagnostics y reporta los errores

"Arregla todos los errores de compilación"
# Claude lee errores → abre archivos → corrige → verifica
```

**Herramienta expuesta:**

```
mcp__ide__getDiagnostics(uri?: string)

uri:    opcional - ruta del archivo a revisar
        sin uri: retorna diagnósticos de todos los archivos abiertos

Retorna:
[
  {
    "uri": "file:///proyecto/src/auth.ts",
    "severity": "error",    // error | warning | info | hint
    "message": "Property 'user' does not exist on type 'Request'",
    "range": {
      "start": { "line": 42, "character": 18 },
      "end":   { "line": 42, "character": 22 }
    },
    "source": "ts"          // "ts" | "eslint" | etc.
  }
]
```

**Casos de uso reales:**

```bash
# 1. Fix all TypeScript errors after refactoring
"Hay errores de TypeScript. Usa getDiagnostics, léelos y corrígelos todos."

# 2. Validar antes de hacer commit
"Antes de hacer commit, verifica con getDiagnostics que no hay errores"

# 3. Debugging de errores específicos de lint
"¿Qué reglas de ESLint están fallando?"
→ Claude llama getDiagnostics, filtra source "eslint", lista los problemas

# 4. Verificar después de cambios
"Modifiqué user.service.ts, ¿introduje nuevos errores?"
→ Claude llama getDiagnostics("src/user/user.service.ts")
```

**Mejores prácticas:**

```
✅ Conectar /ide al inicio de sesiones de debugging o refactoring
✅ Usar getDiagnostics ANTES de declarar que algo está arreglado
✅ Pedir a Claude que verifique con getDiagnostics después de modificaciones

❌ No asumir que Claude sabe de errores del IDE sin /ide conectado
❌ No usar /ide si el IDE no tiene el proyecto abierto (retornará vacío)
```
