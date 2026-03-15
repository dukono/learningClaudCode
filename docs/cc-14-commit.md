# 14. /commit - Crear Commits con IA

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 14. /commit
[⬆️ Top](#14-commit---crear-commits-con-ia)

**¿Qué hace?**
Analiza los cambios en el repositorio git y crea un commit con un mensaje descriptivo y semánticamente correcto, generado automáticamente por Claude. Incluye co-autoría y respeta pre-commit hooks.

**Funcionamiento interno:**

```
Al invocar /commit, Claude ejecuta:

1. git status              → ver archivos modificados/sin trackear
2. git diff --staged       → ver cambios staged y unstaged
3. git log --oneline -5    → entender el estilo de commits del proyecto

Analiza los cambios:
→ Tipo: feat | fix | docs | refactor | test | chore
→ Scope: módulo afectado
→ Descripción: qué cambió y por qué (el "why", no el "what")

Stagea archivos relevantes:
git add archivo1.ts archivo2.ts  ← solo archivos relevantes, no todos

Crea el commit:
git commit -m "$(cat <<'EOF'
feat(auth): add JWT refresh token rotation

Implements automatic token rotation on refresh to prevent
token reuse after compromise.

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"

5. git status              → verificar que el commit fue exitoso
```

**Protocolo de seguridad git incorporado:**

```
Claude SIEMPRE sigue estas reglas:
✓ Crea commits NUEVOS (nunca --amend sin que lo pidas)
✓ Nunca usa --no-verify (respeta pre-commit hooks)
✓ Nunca usa --force push sin confirmación explícita
✓ Agrega archivos específicos, no "git add -A" (evita .env accidental)
✓ No incluye archivos con secretos o credenciales
✓ Confirma antes de push
```

**Formato de mensajes generados:**

```
[type]([scope]): [descripción corta]

[cuerpo opcional explicando el "por qué"]

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>

Tipos:
feat      → nueva funcionalidad
fix       → corrección de bug
docs      → solo documentación
refactor  → refactoring sin cambio de comportamiento
test      → agregar/arreglar tests
chore     → mantenimiento, build, dependencias
perf      → mejora de performance
style     → formato, sin cambio de lógica
```

**Uso práctico:**

```bash
/commit                     # Analiza cambios y crea commit

# Claude también puede hacer commits directamente cuando lo pides:
"Haz commit de los cambios que acabamos de hacer"
"Crea un commit con mensaje 'fix: corrige validación de email'"
```

**Pre-commit hooks:**

```bash
# Si el pre-commit hook falla:
# ── ejemplo: husky + lint-staged ──

git commit -m "feat: add login"
# → husky ejecuta lint-staged
# → lint-staged corre prettier + eslint
# → si hay errores: commit FALLA

# Claude maneja esto:
# 1. Ve que el hook falló
# 2. Lee el error del hook
# 3. Corrige el problema (formatea código, arregla lint)
# 4. Hace re-stage de los archivos corregidos
# 5. Crea un NUEVO commit (no amend)
```

**Casos de uso reales:**

```bash
# 1. Commit rápido después de feature
"Terminé el componente de búsqueda, haz commit"
→ Claude analiza cambios → crea mensaje descriptivo → commit

# 2. Commit con mensaje específico
"Haz commit con el mensaje: 'wip: experimento con nueva arquitectura de cache'"
→ Claude respeta el mensaje que diste pero sigue el protocolo de seguridad

# 3. Commit + Push
"Haz commit y push de los cambios"
→ Claude hace commit → te pide confirmación para push → push

# 4. Verificar antes de commit
"Antes de hacer commit, verifica que no hay errores de TypeScript"
→ Claude corre tsc --noEmit → si pasa → commit
```

**Mejores prácticas:**

```
✅ Dejar que Claude analice todos los cambios antes de stagear
✅ Revisar el mensaje propuesto antes de confirmar
✅ Usar /commit después de cada unidad lógica de trabajo
✅ Configurar CLAUDE.md con el formato de commits de tu proyecto

❌ No decirle a Claude que use --no-verify (bypassea las protecciones)
❌ No hacer commits de archivos .env o con credenciales
❌ No usar --amend en commits ya pusheados sin necesidad
```
