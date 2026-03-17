# Capítulo 11: Performance y Optimización

> **Nivel:** Requiere comprensión de los capítulos anteriores  
> **Prerequisito recomendado:** Todos los capítulos previos  
> **Objetivo:** Dominar las técnicas para hacer que Claude Code sea más rápido, más barato y produzca resultados de mayor calidad.

---

## Tabla de Contenidos

- [11.1 Las tres dimensiones de la performance](#111-las-tres-dimensiones)
- [11.2 Optimización de velocidad: reducir la latencia](#112-optimizar-velocidad)
- [11.3 Optimización de costo: gastar menos tokens](#113-optimizar-costo)
- [11.4 Optimización de calidad: obtener mejores resultados](#114-optimizar-calidad)
- [11.5 El CLAUDE.md como inversor de rendimiento](#115-el-claudemd-como-inversión)
- [11.6 Selección estratégica del modelo](#116-selección-del-modelo)
- [11.7 Prompts eficientes: el arte de ser específico](#117-prompts-eficientes)
- [11.8 Uso estratégico del sistema multi-agente](#118-multi-agente-estratégico)
- [11.9 La paradoja del costo en multi-agente](#119-paradoja-del-costo)
- [11.10 Métricas y monitoreo](#1110-métricas-y-monitoreo)
- [11.11 Resumen y glosario del capítulo](#1111-resumen-y-glosario)

---

## 11.1 Las tres dimensiones de la performance

Cuando hablamos de "performance" en Claude Code, en realidad hablamos de tres dimensiones distintas que frecuentemente están en tensión entre sí:

```
         CALIDAD
            ▲
            │
            │    Zona de
            │    sweet spot
            │      ●
            │
            └──────────────────► VELOCIDAD
           /
          /
         /
        COSTO
```

**Velocidad:** ¿Cuánto tiempo tarda Claude en completar la tarea?

**Costo:** ¿Cuántos tokens se consumen (y por tanto cuánto se paga)?

**Calidad:** ¿Qué tan buenos son los resultados?

### Las tensiones fundamentales

```
VELOCIDAD vs. CALIDAD:
  Mayor velocidad (Haiku) → menos capacidad de razonamiento
  Mayor calidad (Opus)    → más lento

VELOCIDAD vs. COSTO:
  Para la misma tarea, más rápido suele significar
  mismo costo o incluso más (modelos rápidos tienen precio/token similar)

COSTO vs. CALIDAD:
  Mejor calidad (Opus)  → costo más alto
  Menor costo (Haiku)   → menor calidad para tareas complejas
```

No existe una configuración que maximice las tres simultáneamente. El objetivo es encontrar el balance óptimo para cada tipo de tarea.

---

## 11.2 Optimizar velocidad: reducir la latencia

### Fuente 1: El tamaño del contexto

El factor individual más importante para la velocidad es el tamaño del contexto. La latencia hasta el primer token es aproximadamente proporcional al número de tokens de entrada.

**Estrategias para contextos más pequeños:**

```bash
# En lugar de leer archivos completos, usar rangos específicos
# LENTO: Leer 400 líneas para encontrar una función
Read("src/models/user.py")  # 400 líneas = ~4,000 tokens de contexto

# RÁPIDO: Localizar primero, luego leer solo lo necesario
Grep("class UserProfile", "src/models/user.py")  # → línea 87
Read("src/models/user.py", start_line=85, end_line=130)  # 45 líneas = ~450 tokens
```

**El impacto cuantitativo:**
```
Contexto 10k tokens  → ~1 segundo hasta primer token
Contexto 50k tokens  → ~3 segundos
Contexto 100k tokens → ~7 segundos
Contexto 150k tokens → ~15 segundos

Para 10 iteraciones de ReAct, la diferencia puede ser:
  Con 10k: 10 × 1s = 10 segundos total
  Con 100k: 10 × 7s = 70 segundos total (7× más lento)
```

### Fuente 2: El modelo elegido

```
Para tareas simples de análisis (leer, reportar, no modificar):
  Claude Haiku → respuesta en ~0.5-2 segundos

Para tareas de desarrollo estándar:
  Claude Sonnet → respuesta en ~1-4 segundos

Para tareas de arquitectura compleja:
  Claude Opus → respuesta en ~3-10 segundos
```

**Regla práctica:** Usa Haiku para análisis, Sonnet para desarrollo, Opus cuando realmente lo necesites.

### Fuente 3: La longitud del output esperado

Pedir outputs más cortos reduce el tiempo total de generación:

```
LENTO: "Explica en detalle cómo funciona bcrypt, incluyendo..."
→ Genera 2,000 tokens de respuesta = ~15 segundos

RÁPIDO: "Resume en 3 puntos por qué bcrypt es seguro"
→ Genera 200 tokens = ~2 segundos
```

Para tareas de código, esto se traduce en:

```
LENTO: "Escribe tests exhaustivos con muchos casos edge"
→ Output largo

RÁPIDO: "Escribe los 5 tests más importantes para hash_password"
→ Output controlado y más veloz
```

---

## 11.3 Optimizar costo: gastar menos tokens

### El precio de los tokens (referencia 2026)

```
Modelo          Input ($/1M tokens)    Output ($/1M tokens)
─────────────────────────────────────────────────────────────
Claude Haiku        $0.25                    $1.25
Claude Sonnet       $3.00                   $15.00
Claude Opus        $15.00                   $75.00

Con Prompt Caching (tokens de caché):
  Haiku cached:   $0.03/1M tokens (88% descuento)
  Sonnet cached:  $0.30/1M tokens (90% descuento)
  Opus cached:    $1.50/1M tokens (90% descuento)
```

### La estrategia del ahorro con caché

```
SESIÓN SIN OPTIMIZAR (Sonnet):
  
  Petición 1: 10,000 tokens input × $3/1M = $0.030
  Petición 2: 12,000 tokens input × $3/1M = $0.036
  Petición 3: 14,000 tokens input × $3/1M = $0.042
  ...
  Petición 20: 50,000 tokens input × $3/1M = $0.150
  
  Total input ≈ $1.00 para 20 peticiones

SESIÓN CON PROMPT CACHING (automático):

  Petición 1: 10,000 tokens sin caché × $3/1M = $0.030
  Petición 2: 10,000 en caché × $0.30/1M + 2,000 nuevos × $3/1M = $0.009
  Petición 3: 12,000 en caché + 2,000 nuevos = $0.010
  ...
  Petición 20: 48,000 en caché + 2,000 nuevos = $0.020
  
  Total input ≈ $0.25 (75% menos)
```

### El CLAUDE.md extenso es rentable

Dado que el CLAUDE.md se cachea desde la segunda petición, un CLAUDE.md de 2,000 tokens tiene un costo de solo $0.0006 por petición (con caché), mientras que le ahorra a Claude explorar el proyecto con Read y Glob en cada sesión.

**El ROI del CLAUDE.md:**
```
Sin CLAUDE.md:
  Claude lee 10 archivos para orientarse: 10 × 3,000 tokens = 30,000 tokens
  Costo: $0.09 (sin caché) por sesión

Con CLAUDE.md de 2,000 tokens:
  Claude ya tiene el contexto: 2,000 tokens
  Costo: $0.0006/petición después de caché
  
  Ahorro por sesión: ~$0.09 - $0.001 = $0.089
  Amortización del esfuerzo de escribir CLAUDE.md: ~10 sesiones
```

### Selección de modelo por tipo de tarea

```
TAREA                           MODELO ÓPTIMO     AHORRO VS OPUS
─────────────────────────────────────────────────────────────────
Añadir docstring a función      Haiku             97% ahorro
Formatear código                Haiku             97% ahorro
Buscar TODOs en el proyecto     Haiku             97% ahorro
Refactorizar función simple     Sonnet            80% ahorro
Escribir tests para una clase   Sonnet            80% ahorro
Depurar error de lógica         Sonnet            80% ahorro
Diseñar arquitectura            Opus              baseline
Detectar vulnerabilidades       Opus              baseline
Migración de sistema complejo   Opus              baseline
```

---

## 11.4 Optimizar calidad: obtener mejores resultados

### Los factores que más impactan la calidad

**1. El CLAUDE.md: el factor más importante**

Un CLAUDE.md bien escrito es la mejora de calidad más efectiva. Le dice a Claude exactamente:
- Cómo está organizado el proyecto.
- Qué convenciones seguir.
- Qué evitar y por qué.
- Qué comandos usar para qué.

Sin CLAUDE.md, Claude tiene que inferir todo esto. Con CLAUDE.md, puede ir directamente a trabajar con el contexto correcto.

**2. La especificidad del prompt**

```
BAJA CALIDAD (vago):
  "Mejora el módulo de autenticación"
  → Claude interpretará "mejora" según su criterio
  → El resultado puede ser algo que no querías

ALTA CALIDAD (específico):
  "En src/auth.py, mejora la función hash_password:
   1. Usa bcrypt con factor de coste 12
   2. Maneja el caso donde password es una cadena vacía
   3. Añade type hints y docstring
   4. Ejecuta los tests existentes para verificar"
  → Claude sabe exactamente qué hacer y cómo verificarlo
```

**3. El modelo correcto para la tarea**

La calidad de Haiku para tareas que requieren razonamiento complejo es notablemente inferior a Sonnet. Para análisis arquitectónico profundo, Opus produce resultados significativamente mejores.

**4. El contexto limpio**

Un contexto al 20% de uso produce resultados de mayor calidad que un contexto al 85%. Usar `/compact` regularmente mejora la calidad de las respuestas hacia el final de las sesiones largas.

---

## 11.5 El CLAUDE.md como inversión de rendimiento

### El CLAUDE.md mínimo vs. óptimo

```markdown
# CLAUDE.md MÍNIMO (ineficiente):
Mi proyecto Python.

# CLAUDE.md ÓPTIMO (mejora velocidad, costo y calidad):

## Proyecto: API de Autenticación
FastAPI + PostgreSQL 16 + Redis. Python 3.12.

## Estructura del proyecto
- src/auth/       → Módulo de autenticación (hash, JWT, OAuth2)
- src/api/v1/     → Endpoints REST de la API
- src/models/     → Modelos SQLAlchemy (User, Token, Session)
- src/services/   → Lógica de negocio
- tests/          → Pytest (cobertura mínima 80%)

## Comandos esenciales
- `pytest tests/ -v --cov=src` → Tests con cobertura
- `black src/ && isort src/`    → Formatear código
- `mypy src/`                   → Verificar tipos
- `docker-compose up -d`        → Iniciar BD y Redis locales

## Convenciones de código
- Type hints en TODAS las funciones públicas
- Docstrings en formato Google para funciones públicas
- Pydantic v2 para validación de inputs de la API

## Reglas de seguridad
- bcrypt con work factor 12 para hash de contraseñas
- JWT con RS256 (no HS256), expira en 24h
- NUNCA loggear contraseñas ni tokens en texto plano

## Archivos importantes
- src/auth/service.py → hash_password(), verify_password()
- src/auth/tokens.py  → create_token(), verify_token()
- src/core/config.py  → Configuración de la aplicación
- tests/conftest.py   → Fixtures de pytest compartidas

## Archivos que NO debes leer
- .env (secretos de producción)
- .env.prod
```

El CLAUDE.md óptimo:
- Evita que Claude explore el proyecto en cada sesión (~ahorra 20-30k tokens).
- Garantiza que Claude siga las convenciones del proyecto desde el primer mensaje.
- Reduce las preguntas de aclaración de Claude.

---

## 11.6 Selección estratégica del modelo

### El árbol de decisión para elegir modelo

```
¿La tarea requiere razonamiento arquitectónico profundo
o análisis de seguridad crítico?
           │
         SÍ │  NO
           │   │
           │   ▼
           │  ¿La tarea es puramente mecánica
           │  (formatear, añadir comentarios, buscar)?
           │   │
           │  SÍ │  NO
           │   │   │
           ▼   ▼   ▼
          OPUS HAIKU SONNET
```

### Cambiar el modelo durante la sesión

```bash
# Al inicio de sesión o cuando cambias de tipo de tarea:
> /model claude-haiku-4-20250514

# Para tareas complejas:
> /model claude-opus-4-20250514

# Volver al equilibrado:
> /model claude-sonnet-4-20250514
```

### Configurar el modelo por defecto en el proyecto

```json
// .claude/settings.json
{
  "model": "claude-sonnet-4-20250514"
}
```

```json
// ~/.claude/settings.json (global)
{
  "model": "claude-sonnet-4-20250514",
  "smallModel": "claude-haiku-4-20250514"
}
```

---

## 11.7 Prompts eficientes: el arte de ser específico

### Los principios de un prompt eficiente

**Principio 1: Scope acotado**

```
INEFICIENTE:
  "Revisa el proyecto y mejora lo que puedas"
  → Claude leerá decenas de archivos
  → El resultado es impredecible
  → Consume muchos tokens

EFICIENTE:
  "En src/auth/service.py, refactoriza hash_password() para:
   1. Usar bcrypt (ya instalado)
   2. Work factor 12
   3. Mantener compatibilidad con los tests existentes en tests/test_auth.py"
  → Claude lee 2 archivos
  → El resultado es predecible
  → Consume pocos tokens
```

**Principio 2: Criterio de éxito explícito**

```
SIN CRITERIO DE ÉXITO:
  "Optimiza las queries de la base de datos"
  → Claude no sabe cuándo ha terminado
  → Puede hacer demasiado o demasiado poco

CON CRITERIO DE ÉXITO:
  "Optimiza las queries lentas en src/db/queries.py.
   Criterio de éxito: todas las queries de la página de listado
   deben ejecutarse en menos de 100ms según los logs de desarrollo.
   Los tests actuales deben seguir pasando."
  → Claude sabe exactamente cuándo ha terminado
```

**Principio 3: Información de contexto relevante**

```
SIN CONTEXTO:
  "Corrige el bug del login"
  → Claude tiene que explorar para encontrar el problema

CON CONTEXTO:
  "Corrige el bug del login en src/auth/service.py.
   El error es: 'ValueError: invalid salt' en la línea 45.
   Se produce cuando el usuario tiene contraseñas migradas del sistema antiguo (MD5).
   Los tests relevantes están en tests/test_auth.py::test_legacy_login"
  → Claude va directamente al problema
```

### Técnicas avanzadas de prompting

**El prefijo de pensamiento:**
```
"Antes de hacer cualquier cambio, explica en 2-3 líneas tu plan de acción
y los riesgos potenciales. Luego procede."
```
→ Claude razona antes de actuar, lo que reduce errores.

**El formato de output:**
```
"Retorna el análisis en este formato:
ARCHIVOS_AFECTADOS: [lista]
RIESGO: [alto/medio/bajo]
CAMBIOS_PROPUESTOS: [descripción]
TESTS_NECESARIOS: [lista]"
```
→ El output estructurado es más fácil de procesar.

**La verificación explícita:**
```
"Después de cada cambio, ejecuta los tests relacionados.
Solo informa que has terminado cuando todos los tests pasen."
```
→ Claude no termina hasta que verificó su trabajo.

---

## 11.8 Uso estratégico del sistema multi-agente

### Cuándo el multi-agente mejora la performance

```
EL MULTI-AGENTE AYUDA CUANDO:
  ✓ La tarea involucra 10+ archivos independientes
  ✓ Las subtareas son paralelas (sin dependencias)
  ✓ La tarea completa consumiría >70% del contexto en un solo agente
  ✓ Necesitas calidad consistente en todas las partes

EL MULTI-AGENTE NO AYUDA CUANDO:
  ✗ La tarea es pequeña y simple
  ✗ Las subtareas tienen dependencias fuertes entre sí
  ✗ El overhead de inicialización supera el beneficio
```

### Diseñar subtareas para máxima paralelización

```
BUENA DIVISIÓN (tareas independientes):
  Sub A: src/auth/      (10 archivos, sin dependencias de otros módulos)
  Sub B: src/api/       (12 archivos, independiente de src/auth/)
  Sub C: src/models/    (8 archivos, independiente de src/api/)

MALA DIVISIÓN (subtareas dependientes):
  Sub A: "Crea el modelo User"
  Sub B: "Crea la API que usa User" ← Depende de A, no puede ir en paralelo
```

### El prompt del subagente: autosuficiencia total

Los subagentes no pueden pedir aclaraciones. Su prompt debe ser completamente autosuficiente:

```
PROMPT INCOMPLETO (causará problemas):
  "Documenta los archivos de auth"
  → ¿Qué estilo de docstring?
  → ¿Qué información incluir?
  → ¿Solo funciones públicas o también privadas?

PROMPT COMPLETO (funciona independientemente):
  "Documenta todos los archivos Python en src/auth/.
   
   ESTILO: Docstrings en formato Google (como en src/models/user.py).
   INCLUIR en cada función pública: Args, Returns, Raises, Example.
   EXCLUIR: Funciones privadas (empiezan con _).
   VERIFICAR: Lee src/models/user.py como referencia de estilo ANTES de empezar.
   HERRAMIENTAS DISPONIBLES: Read, Write, Glob (no Bash)."
```

---

## 11.9 La paradoja del costo en multi-agente

### Por qué el multi-agente puede costar MÁS

Cada subagente tiene costos adicionales:
- Costo de inicialización (system prompt × N subagentes).
- Tokens de output de cada subagente (no cacheables entre subagentes).

```
COSTO ESTIMADO: Documentar 30 módulos Python

ENFOQUE 1: Un solo agente
  Input tokens: ~100,000 (30 módulos × 3,000 tokens/módulo)
  Output tokens: ~30,000 (documentación generada)
  Costo (Sonnet): (100k × $3 + 30k × $15) / 1M = $0.75
  
  Con caché (80% del input en caché después de las primeras peticiones):
  Costo efectivo: ≈ $0.40

ENFOQUE 2: 3 subagentes de 10 módulos cada uno
  Input tokens por subagente: ~35,000 (system + 10 módulos + instrucciones)
  Input total: 3 × 35,000 = 105,000 tokens
  Output total: ~30,000 tokens
  
  SIN caché entre subagentes (cada subagente empieza fresco):
  Costo (Sonnet): (105k × $3 + 30k × $15) / 1M = $0.765
  
  El multi-agente cuesta ~2x más en este escenario.
```

### Cuándo el multi-agente SÍ reduce el costo

```
El multi-agente reduce el costo cuando el agente único habría
necesitado un modelo más caro (Opus) para mantener calidad,
y los subagentes pueden usar un modelo más barato (Sonnet o Haiku):

ENFOQUE 1: Un agente Opus para 80 módulos
  Input: ~240,000 tokens × $15/1M = $3.60
  Output: ~80,000 tokens × $75/1M = $6.00
  Total: $9.60

ENFOQUE 2: 8 subagentes Sonnet de 10 módulos cada uno (en paralelo)
  Input por subagente: ~35,000 × $3/1M = $0.105
  Output por subagente: ~10,000 × $15/1M = $0.150
  Total × 8 subagentes: ~$2.04
  
  Ahorro: ~79% menos costo
  Velocidad: 8× más rápido (paralelo)
  Calidad: consistente (cada subagente trabaja con contexto fresco)
```

### La regla práctica

```
Usa multi-agente cuando:
  - La calidad con un solo agente se degradaría (contexto lleno)
  - El modelo más barato + multi-agente sea más económico que
    el modelo más caro + agente único

No uses multi-agente cuando:
  - La tarea es pequeña (< 10 archivos, < 1 hora de trabajo)
  - La calidad con un solo agente es suficiente
```

---

## 11.10 Métricas y monitoreo

### Qué métricas monitorear

**Métricas por sesión:**
```bash
# Usar /cost al final de cada sesión para registrar:
> /cost

Sesión completada:
  Tiempo: 45 minutos
  Tokens de entrada: 127,450
  Tokens de entrada (caché): 98,230 (77%)
  Tokens de salida: 23,180
  Costo estimado: $0.67
  Tareas completadas: 3
  Costo por tarea: $0.22/tarea
```

**Indicadores de problemas de performance:**
```
SEÑAL DE ALARMA:                 ACCIÓN:
─────────────────────────────────────────────
Latencia > 15s por respuesta   → Usar /compact (contexto muy lleno)
Calidad baja en últimas tareas → Usar /compact o /clear
Costo > $1/tarea simple        → Revisar el modelo elegido
Costo > $5/sesión              → Revisar el uso de contexto
```

### El audit log de sesiones

Con el hook de audit log del Capítulo 8, puedes acumular métricas:

```bash
#!/bin/bash
# Analizar el uso de Claude Code en el último mes

AUDIT_DIR="$HOME/.claude/audit"
LOG_FILES="$AUDIT_DIR/$(date +%Y-%m)-*.jsonl"

echo "=== Resumen del último mes ==="
echo ""
echo "Herramientas más usadas:"
cat $LOG_FILES | jq -r '.tool' | sort | uniq -c | sort -rn | head -10

echo ""
echo "Proyectos con más actividad:"
cat $LOG_FILES | jq -r '.project' | sort | uniq -c | sort -rn | head -5
```

### Benchmarks de referencia

Para calibrar si el rendimiento es normal o hay algo inusual:

```
BENCHMARKS DE REFERENCIA (Claude Sonnet, sesión nueva):

Análisis de un archivo de 200 líneas:
  Tiempo: ~5-10 segundos
  Tokens: ~3,000-5,000

Refactorización de una función:
  Tiempo: ~15-30 segundos
  Tokens: ~5,000-10,000

Generar tests para una clase:
  Tiempo: ~20-40 segundos
  Tokens: ~8,000-15,000

Documentar un módulo de 10 archivos:
  Tiempo: ~3-5 minutos
  Tokens: ~30,000-50,000

Si tus tiempos son significativamente mayores:
  → El contexto probablemente está muy lleno (/compact)
  → O la red tiene latencia alta
```

---

## 11.11 Resumen y glosario

### Resumen

1. La performance tiene **tres dimensiones en tensión**: velocidad, costo y calidad. No se puede maximizar todo a la vez.
2. **El tamaño del contexto** es el factor más impactante para la velocidad. Mantener el contexto pequeño con lecturas por rango y `/compact` frecuente.
3. El **Prompt Caching** reduce el costo de input hasta en un 90% para sesiones largas. Es automático.
4. El **CLAUDE.md** es la inversión más rentable: ahorra tiempo de exploración en cada sesión y mejora la calidad.
5. **Seleccionar el modelo correcto** para cada tipo de tarea es fundamental: Haiku para mecánico, Sonnet para desarrollo, Opus para arquitectura.
6. **Los prompts específicos** con scope acotado, criterio de éxito y contexto relevante producen mejores resultados.
7. El **multi-agente** mejora velocidad y calidad en tareas masivas, pero puede aumentar el costo si no se usa correctamente.
8. La **paradoja del costo**: multi-agente con modelo barato puede ser más económico que agente único con modelo caro.

### Glosario

| Término | Definición |
|---------|------------|
| **Latencia hasta primer token** | El tiempo que tarda en aparecer el primer token de respuesta. Depende del tamaño del contexto. |
| **Prompt Caching** | Mecanismo automático que reutiliza tokens del prefijo del contexto entre peticiones. |
| **Modelo óptimo** | El modelo que ofrece el mejor balance calidad/costo para un tipo específico de tarea. |
| **Scope acotado** | La práctica de limitar explícitamente el alcance de una tarea para reducir el consumo de contexto. |
| **Overhead de inicialización** | El costo fijo de crear un subagente (tokens del system prompt, primera petición). |
| **ROI del CLAUDE.md** | El retorno de inversión de escribir un CLAUDE.md completo en términos de tokens ahorrados. |
| **Criterio de éxito** | Una condición explícita que le dice a Claude cuándo considera la tarea terminada. |
| **Sweet spot** | El balance óptimo entre velocidad, costo y calidad para un caso de uso específico. |

---

## Apéndice: Cheatsheet de optimización rápida

```
QUIERO MÁS VELOCIDAD:
  → Usar /compact si el contexto supera el 50%
  → Cambiar a Haiku para tareas simples (/model claude-haiku-4-20250514)
  → Limitar las lecturas: Grep primero, luego Read con rangos
  → Sesiones más cortas y focalizadas (1 tarea por sesión)

QUIERO MENOR COSTO:
  → El Prompt Caching es automático, pero un CLAUDE.md largo lo amplifica
  → Usar Haiku para tareas mecánicas (97% más barato que Opus)
  → Prompts específicos que eviten exploración innecesaria
  → Multi-agente con Sonnet para tareas grandes (vs. Opus single-agent)

QUIERO MAYOR CALIDAD:
  → Escribir un CLAUDE.md detallado con convenciones y contexto
  → Prompts específicos con criterio de éxito explícito
  → Usar /compact antes de tareas importantes (contexto fresco)
  → Usar Opus para decisiones arquitectónicas críticas
  → Añadir "explica tu plan antes de actuar" en prompts complejos

QUIERO LOS TRES:
  → Sesiones cortas y focalizadas (1 tarea, /clear al cambiar de tarea)
  → CLAUDE.md bien escrito
  → Selección inteligente del modelo por tipo de tarea
  → Multi-agente solo para tareas masivas independientes
```

---

## Ver también

- **[Capítulo 5](./cap-05-ventana-contexto.md):** Ventana de Contexto — fundamentos del Prompt Caching y `/compact`.
- **[Capítulo 6](./cap-06-sistema-multiagente.md):** Multi-Agente — cuándo y cómo usar múltiples agentes.
- **[Capítulo 9](./cap-09-seguridad-permisos.md):** Seguridad — performance en CI/CD.

---

> 📌 Has completado todos los capítulos. Vuelve al [Índice](./indice.md) para un repaso general.

