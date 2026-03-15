# 11. /skills - Habilidades Personalizadas

[🏠 Volver al Índice](../../CLAUDE_CODE_COMANDOS_GUIA_PRACTICA.md)

---

## 11. /skills
[⬆️ Top](#11-skills---habilidades-personalizadas)

**¿Qué hace?**
Lista las habilidades (skills) disponibles en la sesión actual. Un skill es un prompt especializado que se invoca con `/nombre-del-skill` y carga instrucciones específicas para una tarea particular, sin consumir tokens permanentemente en el sistema.

**Funcionamiento interno:**

```
Sin skill activo:
System Prompt = instrucciones base de Claude Code (pequeño, eficiente)

Al invocar /commit (un skill):
System Prompt = instrucciones base + instrucciones del skill commit
               → "Crea un commit git analizando staged changes,
                  sigue el protocolo de seguridad, usa Co-Authored-By..."

Al terminar el skill:
System Prompt = vuelve a las instrucciones base

Ventaja: Los skills NO se cargan siempre, solo cuando se invocan.
/context muestra los tokens actuales de skills activos.
```

**Ubicación de archivos de skills:**

```
Carga en orden (later overrides earlier):
1. ~/.claude/skills/           ← skills personales globales
2. .claude/skills/             ← skills del proyecto
3. ~/.claude/plugins/cache/    ← skills de plugins instalados

Formato de un skill:
nombre-del-skill.md            ← el nombre = el comando /nombre-del-skill
```

**Crear un skill personalizado:**

```bash
# Crear directorio si no existe
mkdir -p ~/.claude/skills

# Crear el skill
cat > ~/.claude/skills/review-pr.md << 'EOF'
# Code Review de Pull Request

Cuando el usuario invoque este skill:

1. Pide el número de PR o la URL si no lo proporcionó
2. Usa `gh pr view [número] --json` para obtener los cambios
3. Analiza:
   - Corrección lógica del código
   - Posibles bugs o edge cases
   - Seguridad (inyección, XSS, validación)
   - Performance obvias
   - Legibilidad y mantenibilidad
4. Proporciona feedback estructurado:
   - 🔴 Bloqueantes (deben corregirse)
   - 🟡 Sugerencias (mejoras recomendadas)
   - 🟢 Positivos (lo que está bien)
5. Sugiere los cambios específicos con código cuando sea relevante
EOF

# Recargar para que esté disponible
/reload-plugins
```

**Skills built-in de Claude Code:**

```
/commit             → Crea commit git con mensaje generado por Claude
/review-pr          → Code review de PRs (si está instalado)
/compact            → Comprime contexto (built-in)
/clear              → Limpia conversación (built-in)
```

**Uso práctico:**

```bash
/skills                         # Lista todos los skills disponibles

# Invocar un skill:
/commit                         # → ejecuta el skill de commit
/review-pr 123                  # → ejecuta skill con argumento
/mi-skill arg1 arg2             # → skill personalizado con argumentos
```

**Diferencia Skill vs Agente:**

```
Skill                               Agente
──────────────────────────────────  ──────────────────────────────────
Prompt que modifica TU sesión       Subproceso separado con su contexto
Corre en la sesión actual           Corre en contexto aislado
Tú ves el progreso en tiempo real   Lo lanzas y esperas resultado
Para tareas interactivas            Para tareas autónomas largas
Ejemplo: /commit, /review-pr        Ejemplo: Agent tool con subagente
Tokens: se añaden al contexto       Tokens: contexto separado
```

**Mejores prácticas:**

```
✅ Crear skills para flujos que repites frecuentemente
✅ Mantener skills concisos y enfocados en una sola tarea
✅ Versionar skills del proyecto en .claude/skills/ con git
✅ Probar un skill nuevo antes de depender de él

❌ No crear skills para tareas que Claude ya hace bien sin instrucciones
❌ No hacer skills demasiado largos (consumen tokens cuando se activan)
❌ No duplicar lógica entre múltiples skills similares
```
