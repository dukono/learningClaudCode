# 0. CLAUDE.md - Configuración del Proyecto

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 0. CLAUDE.md
[⬆️ Top](#0-claudemd---configuración-del-proyecto)

**¿Qué hace?**
`CLAUDE.md` es el archivo de instrucciones persistentes que Claude Code carga automáticamente al iniciar cada sesión. Es la "memoria de largo plazo" del proyecto: convenciones, comandos importantes, arquitectura, preferencias del equipo. Claude lo lee antes de responder cualquier mensaje.

**Funcionamiento interno:**

```
Al iniciar claude en /mi-proyecto:

1. Busca y carga en orden:
   ~/CLAUDE.md                    ← instrucciones globales personales
   /mi-proyecto/CLAUDE.md         ← instrucciones del proyecto
   /mi-proyecto/src/CLAUDE.md     ← instrucciones del subdirectorio (si existe)

2. Todo el contenido se inyecta al inicio del System Prompt

3. Claude "ya sabe" las convenciones antes de tu primer mensaje

Jerarquía:
┌─────────────────────────────────┐
│  ~/CLAUDE.md                    │  ← Global: tus preferencias personales
│  (cargado siempre)              │
├─────────────────────────────────┤
│  proyecto/CLAUDE.md             │  ← Proyecto: convenciones del repo
│  (cargado si existe)            │
├─────────────────────────────────┤
│  subdirectorio/CLAUDE.md        │  ← Local: contexto específico
│  (cargado si existe)            │
└─────────────────────────────────┘
```

**Uso práctico - Qué incluir:**

```markdown
# Mi Proyecto - CLAUDE.md

## Comandos esenciales
\`\`\`bash
npm run dev        # Iniciar servidor de desarrollo
npm test           # Correr tests
npm run build      # Build de producción
npm run lint       # Linting
\`\`\`

## Arquitectura
- Frontend: React + TypeScript en src/
- Backend: Express en server/
- DB: PostgreSQL con Prisma ORM
- Tests: Jest + React Testing Library

## Convenciones de código
- Usar arrow functions, no function declarations
- Imports absolutos desde src/ (no relativos)
- Nombres de componentes en PascalCase
- Hooks personalizados con prefijo use

## Estilo de commits
feat: nueva funcionalidad
fix: corrección de bug
docs: documentación
refactor: refactoring sin cambio de comportamiento

## Lo que NO hacer
- No usar any en TypeScript
- No hacer console.log en producción
- No modificar package-lock.json manualmente
```

**Casos de uso reales:**

```bash
# 1. Crear CLAUDE.md básico para proyecto nuevo
cat > CLAUDE.md << 'EOF'
## Stack
- Node.js 20, TypeScript 5
- Run tests: npm test
- Linting: npm run lint -- --fix

## Convenciones
- Español para comentarios y docs
- camelCase para variables, PascalCase para clases
EOF

# 2. CLAUDE.md global para preferencias personales
cat > ~/.claude/CLAUDE.md << 'EOF'
## Preferencias personales
- Responder siempre en español
- Preferir soluciones simples sobre ingeniería prematura
- Siempre correr tests antes de decir que algo funciona
EOF

# 3. CLAUDE.md en subdirectorio para contexto específico
# src/api/CLAUDE.md
cat > src/api/CLAUDE.md << 'EOF'
## API Layer
- Todos los endpoints en kebab-case
- Validación con Zod
- Errores con formato: { error: string, code: string }
EOF
```

**Diferencia entre CLAUDE.md y /memory:**

```
CLAUDE.md                          /memory (MEMORY.md)
─────────────────────────────────  ──────────────────────────────────
Escrito por humanos                Escrito por Claude automáticamente
Instrucciones del proyecto         Aprendizajes de sesiones anteriores
Versionado en git                  NO versionado (fuera del repo)
Compartido con el equipo           Personal de cada desarrollador
Convenciones, comandos, stack      Patrones descubiertos, bugs resueltos
```

**Mejores prácticas:**

```
✅ Incluir comandos de build/test/lint exactos
✅ Describir la arquitectura del proyecto brevemente
✅ Listar convenciones que Claude no puede inferir del código
✅ Incluir qué NO hacer (igual de importante)
✅ Mantenerlo conciso (Claude lo lee en cada sesión → tokens)
✅ Versionar CLAUDE.md en git junto con el código

❌ No duplicar lo que ya está obvio en el código
❌ No hacer listas interminables de reglas
❌ No incluir secretos o credenciales
❌ No explicar conceptos básicos de las tecnologías usadas
```

**Troubleshooting:**

```bash
# Verificar que Claude está leyendo tu CLAUDE.md
# Pregúntale: "¿Qué dice mi CLAUDE.md sobre los tests?"

# Si no lo lee, verificar ubicación:
ls -la CLAUDE.md           # Debe estar en el directorio donde ejecutas claude
ls -la ~/.claude/CLAUDE.md # Global

# Ver qué contexto tiene Claude actualmente
/context
```
