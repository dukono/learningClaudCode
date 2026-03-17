# Capítulo 12: CLAUDE.md y el Sistema de Memoria

> **Nivel:** Sin conocimientos previos requeridos  
> **Objetivo:** Dominar el sistema completo de memoria de Claude Code: instrucciones persistentes con CLAUDE.md, memoria de aprendizaje con MEMORY.md, y cómo Claude actualiza su propia memoria entre sesiones.

---

## Tabla de Contenidos

- [12.1 El problema fundamental: Claude olvida todo al cerrar](#121-el-problema-fundamental)
- [12.2 Las dos capas de memoria persistente](#122-las-dos-capas-de-memoria)
- [12.3 CLAUDE.md — La memoria de instrucciones](#123-claudemd-la-memoria-de-instrucciones)
- [12.4 Jerarquía de CLAUDE.md: global, proyecto, subdirectorio](#124-jerarquía-de-claudemd)
- [12.5 Qué escribir en un CLAUDE.md óptimo](#125-qué-escribir-en-un-claudemd-óptimo)
- [12.6 MEMORY.md — La memoria de aprendizaje](#126-memorymd-la-memoria-de-aprendizaje)
- [12.7 El comando /memory: gestión manual](#127-el-comando-memory)
- [12.8 Cómo Claude actualiza su propia memoria](#128-cómo-claude-actualiza-su-propia-memoria)
- [12.9 Diferencias clave: CLAUDE.md vs MEMORY.md](#129-diferencias-clave)
- [12.10 Casos de uso reales y plantillas](#1210-casos-de-uso-reales)
- [12.11 Resumen y glosario del capítulo](#1211-resumen-y-glosario)

---

## 12.1 El problema fundamental: Claude olvida todo al cerrar

Cuando cierras Claude Code y lo vuelves a abrir, el historial de conversación desaparece por completo. Esta es una propiedad fundamental de los LLMs: son **stateless** (sin estado permanente). Cada sesión empieza desde cero.

Esto significa que:
- Claude no sabe que en la sesión anterior acordasteis usar `pnpm` en vez de `npm`.
- No sabe que el proyecto usa Python 3.12, no 3.9.
- No sabe que la función `hash_password` fue migrada a bcrypt en el sprint anterior.
- No recuerda que preferís los commits en inglés aunque el resto del código esté en español.

Sin un mecanismo de memoria, en cada sesión perderías tiempo explicando el contexto del proyecto desde cero.

### La solución: archivos que persisten entre sesiones

Claude Code resuelve este problema con dos archivos de texto que se leen automáticamente al inicio de cada sesión:

```
SESIÓN 1:
  Claude trabaja → aprende cosas → escribe en MEMORY.md
  Tú defines convenciones → las escribes en CLAUDE.md
  Sesión termina → historial se borra

SESIÓN 2:
  Claude Code arranca → lee CLAUDE.md + MEMORY.md
  Claude "sabe" todo lo del CLAUDE.md desde el primer mensaje
  Claude "recuerda" lo que aprendió en sesiones anteriores

El truco: no es memoria real → es contexto leído al inicio
```

---

## 12.2 Las dos capas de memoria persistente

```
┌──────────────────────────────────────────────────────────────┐
│                  SISTEMA DE MEMORIA DE CLAUDE CODE            │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  CLAUDE.md  (instrucciones de proyecto)             │    │
│  │  ─────────────────────────────────────              │    │
│  │  • Escrito por HUMANOS                              │    │
│  │  • Convenciones, stack, comandos, reglas            │    │
│  │  • Versionado en git → compartido con el equipo     │    │
│  │  • Se actualiza cuando el proyecto cambia           │    │
│  └─────────────────────────────────────────────────────┘    │
│                          +                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  MEMORY.md  (aprendizajes de sesiones previas)      │    │
│  │  ─────────────────────────────────────              │    │
│  │  • Escrito por CLAUDE automáticamente               │    │
│  │  • Bugs resueltos, patrones descubiertos            │    │
│  │  • Personal (fuera del repositorio git)             │    │
│  │  • Se enriquece con cada sesión                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                          =                                   │
│  SYSTEM PROMPT de la sesión nueva                           │
│  (Claude ya "conoce" el proyecto antes del primer mensaje)  │
└──────────────────────────────────────────────────────────────┘
```

---

## 12.3 CLAUDE.md — La memoria de instrucciones

### Qué es

`CLAUDE.md` es un archivo Markdown que Claude Code lee automáticamente al arrancar cualquier sesión. Su contenido se inyecta en el **system prompt** antes de que llegue el primer mensaje del usuario.

El resultado: Claude ya conoce el proyecto antes de que le digas nada. No tiene que explorar el repositorio para orientarse. Sabe exactamente qué comandos usar, qué convenciones respetar, qué evitar.

### Cuándo se lee

```
Al ejecutar: claude

  1. Claude Code detecta el directorio de trabajo
  2. Busca CLAUDE.md en la jerarquía de directorios
  3. Concatena todos los encontrados
  4. Los inyecta al inicio del system prompt

  El primer mensaje ya incluye el contexto completo de CLAUDE.md
```

### Dónde vive

```bash
# Tres ubicaciones posibles:
~/.claude/CLAUDE.md              # Global: aplica a TODOS tus proyectos
/tu-proyecto/CLAUDE.md           # Proyecto: aplica a este repositorio
/tu-proyecto/src/modulo/CLAUDE.md # Subdirectorio: contexto específico del módulo
```

### Impacto en tokens

El contenido de CLAUDE.md se incluye en **cada petición a la API**. Sin embargo, gracias al **Prompt Caching** de Anthropic, después de la primera petición los tokens del CLAUDE.md se sirven desde caché al ~10% del coste normal.

Un CLAUDE.md de 2,000 tokens añade solo ~$0.0006 por petición (con caché), pero ahorra a Claude leer 10+ archivos en cada sesión (~$0.09 de exploración). **El ROI es enorme desde la primera sesión.**

---

## 12.4 Jerarquía de CLAUDE.md: global, proyecto, subdirectorio

### El orden de carga

```
Cuando Claude Code arranca en /proyecto/src/auth/:

PASO 1: Lee ~/.claude/CLAUDE.md (si existe)
        → Tus preferencias personales: idioma, estilo, herramientas favoritas

PASO 2: Lee /proyecto/CLAUDE.md (si existe)
        → Convenciones del equipo: stack, comandos, arquitectura

PASO 3: Lee /proyecto/src/CLAUDE.md (si existe)
        → Contexto del módulo src/ (opcional)

PASO 4: Lee /proyecto/src/auth/CLAUDE.md (si existe)
        → Contexto específico del módulo de autenticación

TODOS se concatenan en ese orden en el system prompt.
Si hay conflictos, prevalece el más específico (el último).
```

### El CLAUDE.md global (`~/.claude/CLAUDE.md`)

Este archivo aplica a **absolutamente todos tus proyectos**. Es el lugar para tus preferencias personales:

```markdown
# ~/.claude/CLAUDE.md — Preferencias personales de [Tu Nombre]

## Idioma y comunicación
- Responde siempre en español, aunque el código esté en inglés
- Sé conciso: prefiero respuestas cortas y directas
- Cuando algo es ambiguo, pregunta antes de actuar

## Filosofía de desarrollo
- Prefiere la solución simple sobre la ingeniería prematura
- YAGNI: no añadir funcionalidad que no se necesita ahora
- Siempre ejecutar tests antes de declarar que algo funciona

## Git y commits
- Commits en inglés siguiendo Conventional Commits
- NUNCA hacer git push sin que yo lo pida explícitamente
- Siempre hacer git status antes de cualquier operación git

## Herramientas preferidas
- Node.js: usar pnpm, nunca npm ni yarn
- Python: usar uv, nunca pip directamente
- Editor mental: asumir VS Code como IDE
```

### El CLAUDE.md de proyecto

El archivo más importante para el trabajo diario. Debe estar en la raíz del repositorio y versionarse en git:

```markdown
# /mi-proyecto/CLAUDE.md

## Proyecto: API de Autenticación v2

### Stack técnico
- Runtime: Node.js 22, TypeScript 5.4
- Framework: Fastify 4 (NO Express)
- Base de datos: PostgreSQL 16 con Drizzle ORM
- Tests: Vitest + Supertest
- Linting: ESLint 9 (flat config) + Prettier

### Estructura del proyecto
- src/routes/    → Rutas HTTP (un archivo por entidad)
- src/services/  → Lógica de negocio
- src/db/        → Schema Drizzle y queries
- src/middleware → Middleware de autenticación y validación
- tests/         → Tests de integración (estructura espejo de src/)

### Comandos esenciales
- `pnpm dev`          → Servidor de desarrollo (puerto 3000)
- `pnpm test`         → Todos los tests
- `pnpm test:watch`   → Tests en modo watch
- `pnpm lint`         → Lint + format check
- `pnpm lint:fix`     → Fix automático de lint
- `pnpm db:push`      → Sincronizar schema con la BD
- `pnpm db:studio`    → Abrir Drizzle Studio (explorador BD)

### Convenciones de código
- Todos los archivos con nombres en kebab-case
- Exports nombrados, nunca default exports
- Zod para validación de inputs en rutas
- Errores: siempre lanzar AppError (src/utils/errors.ts)
- No usar any en TypeScript → usar unknown y hacer type guards

### Seguridad (OBLIGATORIO)
- bcrypt con cost factor 12 para contraseñas (NO MD5, NO SHA1)
- JWT: RS256, expira en 24h, refresh tokens con rotación
- NUNCA loggear contraseñas, tokens o datos personales
- Validar y sanitizar TODOS los inputs con Zod

### Lo que NO hacer
- No modificar src/db/schema.ts sin migración de Drizzle
- No cambiar los endpoints públicos de la API sin actualizar docs/api.md
- No hacer console.log en producción (usar logger de Pino)
- No instalar dependencias sin actualizar CLAUDE.md con el stack

### Variables de entorno requeridas
Ver .env.example para la lista completa.
NUNCA hardcodear valores; siempre usar process.env.NOMBRE_VARIABLE
```

### El CLAUDE.md de subdirectorio

Para módulos complejos con reglas específicas:

```markdown
# /mi-proyecto/src/db/CLAUDE.md

## Módulo de Base de Datos

### IMPORTANTE: Reglas de migrations
- NUNCA modificar archivos en migrations/ existentes
- Para cambios de schema: crear nueva migration con `pnpm drizzle-kit generate`
- Las migrations se aplican en producción de forma ordenada

### Queries y transacciones
- Todas las operaciones que modifiquen datos deben ir en transacción
- Usar prepared statements para queries que se ejecutan frecuentemente
- El pool de conexiones está en src/db/client.ts (no crear conexiones nuevas)

### Convención de nombres en la BD
- Tablas: snake_case plural (users, refresh_tokens)
- Columnas: snake_case (created_at, user_id)
- Índices: idx_tabla_columna (idx_users_email)
```

---

## 12.5 Qué escribir en un CLAUDE.md óptimo

### La estructura recomendada

Un CLAUDE.md efectivo responde a estas preguntas que Claude tendría que resolver explorando el proyecto:

```
1. ¿Qué hace este proyecto? (1-2 líneas)
2. ¿Cuál es el stack exacto? (runtime, framework, BD, herramientas)
3. ¿Cómo arranco el proyecto y corro los tests?
4. ¿Cómo está organizado el código? (estructura de directorios)
5. ¿Qué convenciones debo seguir?
6. ¿Qué debo evitar absolutamente?
7. ¿Qué archivos son especialmente sensibles o peligrosos?
```

### Qué incluir ✅

```markdown
## Incluir en CLAUDE.md

✅ Comandos exactos de build, test, lint, dev
   Razón: Claude no puede saber si usas "npm test" o "vitest run"

✅ Stack técnico con versiones específicas
   Razón: La forma de hacer las cosas cambia entre versiones

✅ Estructura de directorios y responsabilidad de cada uno
   Razón: Ahorra a Claude leer decenas de archivos para orientarse

✅ Convenciones que no son obvias del código
   Razón: Si está en el código, Claude lo ve. Si no está, necesita que se lo digas.

✅ Reglas de seguridad obligatorias
   Razón: Un error aquí puede tener consecuencias graves

✅ Qué NO hacer (lista negra)
   Razón: Igual de importante que el "qué hacer"

✅ Archivos/directorios que nunca deben modificarse sin cuidado
   Razón: Previene accidentes en archivos críticos
```

### Qué NO incluir ❌

```markdown
## NO incluir en CLAUDE.md

❌ Secretos, API keys, contraseñas
   Razón: CLAUDE.md puede estar en git y viajar a la API de Anthropic

❌ Tutoriales de las tecnologías usadas
   Razón: Claude ya sabe qué es React, PostgreSQL, etc.

❌ Listas interminables de reglas obvias
   Razón: Si Claude ya lo hace bien por defecto, no hace falta decírselo

❌ Comentarios sobre Claude Code en sí
   Razón: "Recuerda que puedes usar Bash" → Claude ya lo sabe

❌ Contexto histórico del proyecto que ya no es relevante
   Razón: Ocupa tokens sin aportar valor

❌ Información que cambia muy frecuentemente
   Razón: CLAUDE.md desactualizado confunde más que ayuda
```

---

## 12.6 MEMORY.md — La memoria de aprendizaje

### Qué es

`MEMORY.md` es el complemento de `CLAUDE.md`, pero con una diferencia fundamental: **lo escribe Claude, no tú**. Es el diario de aprendizaje de Claude sobre tu proyecto específico.

### Dónde vive

```bash
# Directorio de memoria (creado automáticamente por Claude):
~/.claude/projects/[hash-del-path-del-proyecto]/memory/

# El archivo principal:
~/.claude/projects/[hash]/memory/MEMORY.md

# Puedes crear archivos adicionales de memoria:
~/.claude/projects/[hash]/memory/debugging.md
~/.claude/projects/[hash]/memory/patterns.md
~/.claude/projects/[hash]/memory/decisions.md
```

El hash del path es determinista: siempre que trabajes con `/home/usuario/mi-proyecto`, Claude usará el mismo directorio de memoria. Así tu MEMORY.md persiste entre sesiones aunque uses distintos terminales.

### Qué contiene típicamente

```markdown
# MEMORY.md (generado y actualizado por Claude)

## Aprendizajes de sesiones anteriores

### Configuración especial detectada
- El proyecto usa una versión fork de Express en vendor/express-patched/ (no la versión oficial)
- El puerto de desarrollo es 4200, no 3000 (configured in vite.config.ts)
- Las variables de entorno locales se leen de .env.local, no de .env

### Bugs conocidos y soluciones
- 2026-02-15: Bug de race condition en AuthService.refreshToken()
  Solución: usar distributed lock con Redis. Ver: src/services/auth.service.ts:156
  
- 2026-03-01: Los tests de integración fallan si Redis no está arriba
  Solución: `docker-compose up -d redis` antes de `pnpm test`

### Patrones de código del proyecto
- Las respuestas de error siguen: { success: false, error: { code, message, details? } }
- Los DTOs de request usan sufijo Request (CreateUserRequest, UpdateProfileRequest)
- Los test fixtures están en tests/fixtures/, no en tests/__fixtures__

### Decisiones técnicas tomadas durante sesiones
- 2026-03-10: Decidimos NO usar GraphQL, mantener REST. Ver ADR-003 en docs/adr/
- 2026-03-12: Migración a pnpm workspaces completada. Comandos cambiaron.
```

### El límite de MEMORY.md

Claude Code lee automáticamente las **primeras 200 líneas** de MEMORY.md. El contenido posterior se trunca. Por eso es importante mantener MEMORY.md conciso y eliminar aprendizajes obsoletos.

---

## 12.7 El comando /memory

### Qué hace

`/memory` abre el archivo `MEMORY.md` del proyecto actual en el editor de texto configurado en el sistema.

```bash
> /memory

# Efecto: abre ~/.claude/projects/[hash]/memory/MEMORY.md
# en $EDITOR (nano, vim, VS Code, etc.)
```

### Para qué sirve

Permite **revisar y editar manualmente** la memoria que Claude ha ido acumulando:

```bash
# Ver qué recuerda Claude sobre el proyecto:
> /memory
# Abre el archivo y puedes leerlo

# Corregir un aprendizaje incorrecto:
> /memory
# Editas directamente y eliminas o corriges la entrada equivocada

# Añadir manualmente información importante:
> /memory
# Escribes directamente en el archivo
# Claude la leerá en la próxima sesión
```

### Cómo verificar que Claude lee la memoria

```bash
# Pregunta directa:
> ¿Qué información tienes en tu memoria sobre este proyecto?
# Claude listará lo que tiene en MEMORY.md

# O:
> /context
# Muestra el system prompt completo, incluyendo MEMORY.md
```

---

## 12.8 Cómo Claude actualiza su propia memoria

### Cuándo Claude escribe en MEMORY.md

Claude actualiza `MEMORY.md` automáticamente en situaciones específicas:

**1. Cuando el usuario lo pide explícitamente:**
```bash
> Recuerda que en este proyecto siempre usamos pnpm, nunca npm

# Claude escribe en MEMORY.md:
# - Gestor de paquetes: pnpm (NUNCA npm ni yarn)
```

**2. Cuando resuelve un bug con una solución no obvia:**
```bash
# Claude encuentra que los tests fallan porque Redis no está arriba
# Resuelve el problema
# Escribe en MEMORY.md:
# - Tests requieren Redis activo. Solución: docker-compose up -d redis
```

**3. Cuando descubre configuración especial:**
```bash
# Claude lee package.json y descubre que el puerto de dev es 8080, no 3000
# Escribe en MEMORY.md:
# - Puerto de desarrollo: 8080 (configurado en .env.local como PORT=8080)
```

**4. Cuando se toman decisiones arquitectónicas:**
```bash
> Hemos decidido migrar de REST a tRPC para la API interna

# Claude escribe en MEMORY.md:
# - 2026-03-15: Migración a tRPC iniciada. Solo afecta a src/internal/.
#   La API pública sigue siendo REST.
```

### El ciclo de memoria completo

```
┌─────────────────────────────────────────────────────────────┐
│  CICLO DE MEMORIA ENTRE SESIONES                            │
│                                                             │
│  Sesión N:                                                  │
│    Lee CLAUDE.md + MEMORY.md                               │
│    ─→ Trabaja con ese contexto                              │
│    ─→ Aprende cosas nuevas                                  │
│    ─→ Escribe en MEMORY.md                                  │
│    Sesión cierra                                            │
│                                                             │
│  Sesión N+1:                                                │
│    Lee CLAUDE.md + MEMORY.md (ahora más completo)          │
│    ─→ Ya "sabe" lo aprendido en sesiones anteriores        │
│    ─→ Puede enfocarse en la tarea sin reexplorar            │
│    ─→ Aprende más → actualiza MEMORY.md                    │
└─────────────────────────────────────────────────────────────┘
```

---

## 12.9 Diferencias clave: CLAUDE.md vs MEMORY.md

| Aspecto | CLAUDE.md | MEMORY.md |
|---------|-----------|-----------|
| **Quién escribe** | Humanos (desarrolladores) | Claude automáticamente |
| **Qué contiene** | Instrucciones, convenciones, stack | Aprendizajes, bugs, patrones descubiertos |
| **Versionado en git** | ✅ Sí, debe versionarse | ❌ No, está fuera del repositorio |
| **Compartido con el equipo** | ✅ Sí | ❌ No, es personal de cada dev |
| **Actualización** | Manual, cuando el proyecto cambia | Automática por Claude en cada sesión |
| **Ubicación** | `proyecto/CLAUDE.md` | `~/.claude/projects/[hash]/memory/` |
| **Límite efectivo** | Sin límite estricto | ~200 líneas (el resto se trunca) |
| **Persistencia** | Mientras exista el archivo | Mientras exista el directorio de usuario |

---

## 12.10 Casos de uso reales y plantillas

### Plantilla: CLAUDE.md para proyecto Python/FastAPI

```markdown
# CLAUDE.md — Backend API

## Stack
- Python 3.12, FastAPI 0.111
- PostgreSQL 16 con SQLAlchemy 2.0 + Alembic
- Redis 7 (caché y sesiones)
- Tests: pytest + pytest-asyncio

## Comandos esenciales
- `uv run fastapi dev`        → Servidor dev (puerto 8000)
- `uv run pytest tests/ -v`   → Tests
- `uv run alembic upgrade head` → Aplicar migrations
- `uv run alembic revision --autogenerate -m "desc"` → Nueva migration
- `uv run ruff check . --fix`  → Lint + fix

## Estructura
- app/routers/     → Rutas FastAPI (un archivo por dominio)
- app/services/    → Lógica de negocio
- app/models/      → Modelos SQLAlchemy
- app/schemas/     → Schemas Pydantic
- app/core/        → Config, DB, seguridad
- tests/           → Tests (estructura espejo de app/)

## Convenciones
- Type hints obligatorios en todas las funciones
- Docstrings en formato Google
- Async para operaciones I/O, sync para lógica pura
- Schemas Pydantic para request/response (no dicts)

## Seguridad
- Contraseñas: bcrypt con rounds=12 (app/core/security.py)
- JWT: algoritmo HS256, secreto en env SECRET_KEY
- CORS configurado en app/core/config.py (NO *):
  Solo dominios de ALLOWED_ORIGINS

## Migrations (CRÍTICO)
- NUNCA modificar migrations existentes en alembic/versions/
- SIEMPRE generar nueva migration para cambios de schema
- Revisar el SQL generado antes de hacer upgrade

## Lo que NO hacer
- No usar sync I/O en rutas async (usar asyncpg, aioredis)
- No poner lógica de negocio en los routers (va en services/)
- No hardcodear URLs ni credenciales
```

### Plantilla: CLAUDE.md global (preferencias personales)

```markdown
# ~/.claude/CLAUDE.md — Preferencias de desarrollo

## Comunicación
- Siempre en español (código en inglés, explicaciones en español)
- Respuestas concisas: no repetir lo que acabo de decir
- Si algo es ambiguo, preguntar en lugar de asumir

## Filosofía
- Solución más simple que funcione (no ingeniería prematura)
- Siempre correr tests después de un cambio antes de decir "listo"
- Explicar el "por qué" de cambios importantes, no solo el "qué"

## Git
- Commits en inglés, formato Conventional Commits
- NUNCA hacer push sin confirmación explícita mía
- NUNCA hacer --force-push sin confirmación explícita

## Herramientas preferidas
- JavaScript/TypeScript: pnpm (nunca npm o yarn)
- Python: uv (nunca pip directamente)
- Docker: docker compose v2 (no docker-compose v1)
```

### Gestionar memoria de forma proactiva

```bash
# Al inicio de un proyecto nuevo, crear memoria inicial:
> Recuerda los siguientes hechos sobre este proyecto:
  1. Usamos pnpm, no npm
  2. Los tests requieren que MongoDB esté activo en localhost:27017
  3. La rama principal es "main", no "master"

# Claude los escribirá en MEMORY.md

# Verificar periódicamente y limpiar:
> /memory
# Revisa el archivo, elimina entradas obsoletas

# Limpiar memoria que ya no es relevante:
> Limpia de tu memoria la información sobre la migración de
  Redux a Zustand, ya está completada y no es relevante.
```

---

## 12.11 Resumen y glosario

### Resumen

1. **Claude olvida todo al cerrar la sesión.** CLAUDE.md y MEMORY.md son la solución a este problema.
2. **CLAUDE.md** contiene instrucciones del proyecto escritas por humanos. Se versiona en git y se comparte con el equipo. Aplica en tres niveles: global, proyecto, subdirectorio.
3. **MEMORY.md** contiene aprendizajes escritos por Claude automáticamente entre sesiones. Es personal (no en git). Se lee en cada inicio de sesión.
4. El **Prompt Caching** hace que el costo de CLAUDE.md sea mínimo (~10% del normal después de la primera petición).
5. El comando **`/memory`** abre el archivo MEMORY.md para revisión y edición manual.
6. Claude actualiza MEMORY.md cuando el usuario se lo pide, cuando resuelve bugs no obvios, y cuando descubre configuración especial.
7. El límite efectivo de MEMORY.md es ~200 líneas; el resto se trunca.

### Glosario

| Término | Definición |
|---------|------------|
| **CLAUDE.md** | Archivo de instrucciones del proyecto leído automáticamente por Claude en cada sesión. |
| **MEMORY.md** | Archivo de aprendizajes escrito por Claude automáticamente. Personal, no versionado en git. |
| **System prompt** | El mensaje de instrucciones que establece el comportamiento de Claude. CLAUDE.md se inyecta aquí. |
| **Prompt Caching** | Mecanismo de Anthropic que reutiliza el prefijo del contexto entre peticiones al 10% del coste. |
| **/memory** | Comando del REPL que abre MEMORY.md en el editor para revisión/edición. |
| **Jerarquía de CLAUDE.md** | Global (~/.claude/CLAUDE.md) → Proyecto → Subdirectorio. Los específicos sobreescriben los generales. |
| **Hash del proyecto** | Identificador único derivado del path del proyecto. Determina el directorio de MEMORY.md. |

---

## Ver también

- **[Capítulo 5](./cap-05-ventana-contexto.md):** Cómo el contenido de CLAUDE.md afecta al tamaño del contexto.
- **[Capítulo 13](./cap-13-skills-plugins.md):** Skills y Plugins — otra forma de extender las instrucciones de Claude.
- **[Capítulo 16](./cap-16-configuracion-settings.md):** Settings.json — configuración técnica del entorno.

---

> 📌 Siguiente capítulo: [Cap. 13 — Skills y Plugins](./cap-13-skills-plugins.md)

