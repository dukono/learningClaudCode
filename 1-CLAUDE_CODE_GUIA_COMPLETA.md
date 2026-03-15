# 📘 Claude Code: Guía Completa desde Cero
> Versión de referencia: Claude Code ~1.x · Fecha: Marzo 2026

## 📚 Tabla de Contenidos
1. [¿Qué es Claude Code?](#1-qué-es-claude-code)
2. [Instalación y Configuración Inicial](#2-instalación-y-configuración-inicial)
3. [Primera Sesión: Cómo Funciona en la Práctica](#3-primera-sesión-cómo-funciona-en-la-práctica)
4. [Comandos Slash: Control Total de la Sesión](#4-comandos-slash-control-total-de-la-sesión)
5. [El Arte del Prompting Efectivo](#5-el-arte-del-prompting-efectivo)
6. [CLAUDE.md: El Manual de Instrucciones de tu Proyecto](#6-claudemd-el-manual-de-instrucciones-de-tu-proyecto)
7. [Subagentes Especializados](#7-subagentes-especializados)
8. [MCP: Model Context Protocol](#8-mcp-model-context-protocol)
9. [Hooks: Automatizar el Comportamiento](#9-hooks-automatizar-el-comportamiento)
10. [Casos de Uso Prácticos Paso a Paso](#10-casos-de-uso-prácticos-paso-a-paso)
11. [Integración con el Ecosistema de Desarrollo](#11-integración-con-el-ecosistema-de-desarrollo)
12. [Buenas Prácticas y Patrones](#12-buenas-prácticas-y-patrones)
13. [Limitaciones, Seguridad y Privacidad](#13-limitaciones-seguridad-y-privacidad)
14. [Troubleshooting: Errores Comunes y Soluciones](#14-troubleshooting-errores-comunes-y-soluciones)
15. [Apéndices y Referencia Rápida](#15-apéndices-y-referencia-rápida)

---

## 1. ¿Qué es Claude Code?

### La Diferencia entre un Chatbot y un Agente

Para entender Claude Code, primero hay que entender qué lo hace diferente de usar Claude en el navegador.

Cuando usas Claude en **claude.ai** (versión web), el flujo es:
```
Tú escribes texto → Claude responde con texto → Fin
```
Claude no puede tocar tu computadora. Solo genera palabras. Si le pides que refactorice tu código, te dará el código reescrito como texto, pero **tú** tendrás que copiarlo y pegarlo en los archivos.

Con **Claude Code**, el flujo es radicalmente diferente:
```
Tú describes una tarea
         ↓
Claude lee tus archivos reales
         ↓
Claude razona sobre el código
         ↓
Claude edita los archivos directamente
         ↓
Claude ejecuta los tests en tu terminal
         ↓
Claude ve los resultados y corrige errores
         ↓
Claude repite hasta que funciona
         ↓
Te entrega el resultado final
```

La diferencia fundamental es que Claude Code tiene **acceso a herramientas reales** que actúan sobre tu sistema: puede leer archivos, escribirlos, ejecutar comandos en la terminal y navegar por la web. No solo "habla sobre código", lo **ejecuta**.

### Analogía del Desarrollador Senior

Imagina que puedes contratar a un desarrollador senior que:

- **Conoce todos los lenguajes y frameworks** — Python, Java, TypeScript, Rust, Go, frameworks de testing, ORMs, todo.
- **Lee todo el proyecto antes de tocar nada** — Antes de escribir una línea, inspecciona la estructura, lee los archivos existentes, entiende las convenciones del proyecto.
- **Ejecuta y verifica su propio trabajo** — Después de hacer un cambio, corre los tests para confirmar que no ha roto nada.
- **Itera hasta que funciona** — Si los tests fallan, analiza el error, corrige y vuelve a intentar.
- **Te explica cada decisión** — No hace cambios en silencio; te dice qué hizo y por qué.
- **Trabaja a la velocidad de la computadora** — Lo que a un desarrollador humano le llevaría 30 minutos, Claude lo hace en 2.

**Claude Code es ese desarrollador dentro de tu terminal.**

### ¿Para Qué Sirve Concretamente?

No es una herramienta abstracta. Aquí hay tareas reales que puedes delegar:

| Categoría | Tarea concreta |
|-----------|----------------|
| **Refactoring** | Convertir código síncrono a async/await en 20 archivos |
| **Testing** | Generar una suite completa de tests para un módulo sin cobertura |
| **Debugging** | Investigar un bug con stack trace y reproducirlo en un test |
| **Documentación** | Añadir docstrings a 50 funciones y generar README |
| **Migración** | Migrar de Flask a FastAPI, de SQLite a PostgreSQL |
| **Code Review** | Revisar todos los cambios antes de un PR |
| **Seguridad** | Buscar vulnerabilidades conocidas en el código |
| **Optimización** | Perfilar una función lenta e implementar mejoras |
| **Generación** | Crear una API REST completa desde especificación |

### Comparación con Herramientas Similares

Para saber cuándo usar Claude Code vs. otras herramientas:

| Herramienta | Qué es | Puntos fuertes | Limitaciones vs. Claude Code |
|-------------|--------|----------------|------------------------------|
| **GitHub Copilot** | Plugin de IDE | Autocompletado en tiempo real | No ejecuta código, solo sugiere mientras escribes |
| **Cursor** | IDE completo | UI integrada, chat en IDE | No tiene ejecución real de comandos del sistema |
| **Aider** | CLI para coding | Muy bueno con Git | Menos autónomo, requiere más guía manual |
| **ChatGPT + Code Interpreter** | Web + sandbox | Ejecuta código en sandbox | Sandbox aislado, no accede a TU proyecto real |
| **Devin** | Agente autónomo | Muy autónomo | Mucho más caro, menos control granular |
| **Claude Code** | CLI agente | Autónomo, acceso total, configurable | Requiere revisar cambios manualmente |

La elección no es "cuál es mejor" sino "cuál es el correcto para la tarea":
- Para autocompletado mientras codeas → **Copilot o Cursor**
- Para tareas grandes y autónomas → **Claude Code**
- Para pair programming interactivo → **Claude Code o Aider**

---

## 2. Instalación y Configuración Inicial

### Requisitos del Sistema

Antes de instalar, verifica que tu sistema cumple los requisitos:

| Requisito | Mínimo | Recomendado | Cómo verificar |
|-----------|--------|-------------|----------------|
| **Node.js** | v18.0 | v20 LTS | `node --version` |
| **npm** | v8.0 | v10.0 | `npm --version` |
| **Sistema Operativo** | macOS 12, Ubuntu 20.04, Windows WSL2 | macOS 14, Ubuntu 22.04 | — |
| **RAM** | 4 GB | 8 GB | — |
| **Conexión a internet** | Requerida siempre | — | — |
| **Cuenta Anthropic** | API Key activa | Plan Pro o Team | console.anthropic.com |

> **¿Por qué Node.js?** Claude Code está escrito en TypeScript/Node.js. El CLI es un proceso Node que se comunica con la API de Anthropic y actúa como intermediario entre tú y el modelo.

### Instalación en macOS

```bash
# Paso 1: Verificar que tienes Node.js 18+
node --version
# Si no tienes Node.js o es muy antiguo, instálalo con nvm:

# Instalar nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Recargar la shell
source ~/.zshrc  # o ~/.bashrc si usas bash

# Instalar Node.js 20 LTS
nvm install 20
nvm use 20
nvm alias default 20

# Paso 2: Instalar Claude Code globalmente
npm install -g @anthropic-ai/claude-code

# Paso 3: Verificar la instalación
claude --version
# Debería mostrar algo como: claude-code/1.x.x ...
```

### Instalación en Linux (Ubuntu/Debian)

```bash
# Paso 1: Instalar Node.js 20 via NodeSource
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verificar
node --version   # v20.x.x
npm --version    # 10.x.x

# Paso 2: Instalar Claude Code
npm install -g @anthropic-ai/claude-code

# Si hay error de permisos, la mejor solución es configurar
# un directorio global para npm sin sudo:
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
npm install -g @anthropic-ai/claude-code

# Paso 3: Verificar
claude --version
```

### Instalación en Windows (WSL2)

Claude Code **no funciona nativamente en Windows**. Necesita WSL2 (Windows Subsystem for Linux).

```powershell
# En PowerShell como administrador:

# Paso 1: Instalar WSL2 con Ubuntu
wsl --install -d Ubuntu

# Reiniciar el sistema si lo pide

# Paso 2: Abrir Ubuntu desde el menú de inicio
# Crear usuario y contraseña cuando lo pida

# Ya dentro de Ubuntu (WSL2), seguir los pasos de Linux:
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
npm install -g @anthropic-ai/claude-code
```

### Configurar la API Key

Claude Code necesita una API Key de Anthropic para funcionar. Sin ella no puede conectarse al modelo.

**Cómo obtener la API Key:**
1. Ve a https://console.anthropic.com
2. Regístrate o inicia sesión
3. Ve a "API Keys" en el menú lateral
4. Crea una nueva key → cópiala (solo se muestra una vez)

**Dónde configurarla:**

```bash
# OPCIÓN 1: Variable de entorno temporal (solo dura la sesión actual de terminal)
export ANTHROPIC_API_KEY="sk-ant-api03-..."

# OPCIÓN 2: Permanente en ~/.bashrc o ~/.zshrc (recomendado para desarrollo local)
echo 'export ANTHROPIC_API_KEY="sk-ant-api03-..."' >> ~/.bashrc
source ~/.bashrc
# A partir de ahora, cada vez que abras la terminal, la key estará disponible

# OPCIÓN 3: Dejar que Claude Code la pida
# Si ejecutas `claude` sin la variable, te pedirá la key y la guardará
# en ~/.claude/.credentials.json (cifrado localmente)

# VERIFICAR que está configurada:
echo $ANTHROPIC_API_KEY
# Debería mostrar: sk-ant-api03-...
```

> ⚠️ **SEGURIDAD CRÍTICA**: Nunca pongas la API Key en archivos de tu proyecto. Nunca la hagas commit en Git. Si accidentalmente la expones, ve inmediatamente a console.anthropic.com y revócala.

### Primera Verificación

```bash
$ claude --version

# Salida esperada:
claude-code/1.2.3 linux-x64 node-v20.11.0
```

Si ves esto, Claude Code está instalado correctamente.

### Configuración del Terminal con `/terminal-setup`

Cuando abres Claude Code por primera vez, ejecuta `/terminal-setup`. Este comando configura atajos de teclado que hacen el trabajo más eficiente:

```
> /terminal-setup

Configurando atajos de teclado...

Los siguientes atajos serán añadidos a tu configuración:

  Ctrl+← / Ctrl+→   Mover el cursor por palabras
  Alt+Backspace      Borrar la palabra anterior al cursor
  Ctrl+L             Limpiar la pantalla del terminal

¿Aplicar configuración? (s/n): s

✅ Configuración aplicada. Reinicia la terminal para que surta efecto.
```

---

## 3. Primera Sesión: Cómo Funciona en la Práctica

### Abrir Claude Code

Navega al directorio de tu proyecto y ejecuta `claude`:

```bash
cd /mi/proyecto
claude
```

Verás algo así:

```
╭──────────────────────────────────────────────────╮
│  Claude Code v1.2.3                              │
│                                                  │
│  Modelo:      claude-sonnet-4                    │
│  Directorio:  /mi/proyecto                       │
│  CLAUDE.md:   No encontrado                      │
│                                                  │
│  Escribe /help para ver comandos disponibles.    │
╰──────────────────────────────────────────────────╯

>
```

El prompt `>` indica que Claude está esperando tu tarea. **Simplemente escribe en lenguaje natural lo que quieres hacer.** No necesitas comandos especiales para empezar.

### ¿Debo usar `/init` antes de escribir tareas?

No es obligatorio, pero es muy recomendable para proyectos donde trabajarás regularmente. `/init` genera el archivo `CLAUDE.md` que da a Claude el contexto permanente del proyecto.

- Si es **la primera vez** que usas Claude Code en este proyecto → usa `/init`
- Si el proyecto ya tiene `CLAUDE.md` → puedes empezar directamente con tu tarea
- Si solo vas a hacer una tarea puntual y no repetirás → puedes saltarte `/init`

### Tu Primera Tarea: Exploración del Proyecto

La mejor primera tarea para cualquier proyecto nuevo es pedirle a Claude que lo explore:

```
> Explora la estructura de este proyecto y dime:
  1. Qué hace esta aplicación
  2. Qué stack tecnológico usa
  3. Cómo se organiza el código
  4. Cómo se ejecutan los tests
  No modifiques nada, solo observa y reporta.
```

Claude responderá leyendo archivos reales de tu proyecto:

```
Analizando el proyecto...

● Read(package.json)
● Read(README.md)
● Read(src/index.ts)
● Bash(find . -name "*.ts" | head -30)
● Read(tsconfig.json)

Aquí está lo que encontré:

**Qué hace:**
Es una API REST de gestión de usuarios. Permite crear, autenticar
y gestionar usuarios con roles y permisos.

**Stack:**
- Runtime: Node.js 20 + TypeScript 5
- Framework: Express 4
- Base de datos: PostgreSQL via Prisma ORM
- Tests: Jest + Supertest
- Lint: ESLint + Prettier

**Organización:**
src/
  controllers/    ← Manejo de requests HTTP
  services/       ← Lógica de negocio
  models/         ← Tipos TypeScript + Prisma schemas
  middleware/     ← Auth, validación, errores
  utils/          ← Funciones helper

**Tests:**
  npm test              → Todos los tests
  npm run test:watch    → Modo watch
  npm run test:coverage → Con cobertura (mínimo 80%)
```

### ¿Qué Pasa Internamente en Cada Solicitud?

Es importante entender esto para trabajar bien con Claude Code.

```
1. Tú escribes un mensaje en el prompt.

2. El CLI de Claude Code construye el "payload" para la API:
   {
     "system": [contenido de CLAUDE.md si existe],
     "messages": [historial completo de la conversación],
     "tools": [definiciones de Read, Write, Bash, etc.]
   }

3. Este payload se envía a la API de Anthropic vía HTTPS.

4. El modelo Claude recibe todo y decide:
   - ¿Necesito leer algún archivo? → usa herramienta Read
   - ¿Necesito ejecutar algo? → usa herramienta Bash
   - ¿Tengo suficiente info para responder? → responde directamente

5. Si Claude usa una herramienta, la respuesta viene como:
   {
     "tool_use": {
       "name": "Read",
       "input": { "path": "src/auth.py" }
     }
   }

6. El CLI ejecuta Read("src/auth.py") localmente en tu máquina
   y devuelve el contenido a la API.

7. Claude recibe el contenido del archivo y razona con él.

8. Este ciclo se repite (puede ser 5, 10, 20 veces)
   hasta que Claude tiene toda la información que necesita
   y puede completar la tarea.

9. Claude envía la respuesta final al usuario.
```

Esto explica por qué Claude Code puede tardar de segundos a minutos en tareas grandes: no es una sola llamada a la API, sino **muchas llamadas encadenadas**, cada una con su ida y vuelta.

### El Progreso en Tiempo Real

Claude Code muestra cada acción que realiza en tiempo real:

```
● Read(src/auth/service.py)          ← está leyendo este archivo
● Bash(pytest tests/ -v)             ← está ejecutando este comando
● Write(src/auth/service.py)         ← está escribiendo en este archivo
● WebSearch("bcrypt vs argon2 2025") ← está buscando en internet
```

El símbolo `●` con su color indica el estado:
- `●` normal → en progreso
- `●` verde → completado con éxito
- `●` rojo → falló (Claude lo verá y actuará en consecuencia)

### Interrumpir y Redirigir

Si Claude está haciendo algo incorrecto, presiona `Ctrl+C` para interrumpirlo inmediatamente. Después puedes redirigirlo:

```
(Claude está modificando tests/ cuando no debería)

Ctrl+C → interrumpido

> Para. No quiero que toques los tests todavía.
  Primero quiero que solo me muestres qué cambios planeas
  hacer en src/, sin modificar nada.
```

---

## 4. Comandos Slash: Control Total de la Sesión

Los comandos slash (`/comando`) son instrucciones para **controlar el comportamiento de Claude Code**, no para darle tareas de programación. Son meta-comandos: controlan la sesión, no el código.

**Diferencia clave:**
```
Tarea de programación → escribes en lenguaje natural
  "Refactoriza src/auth.py para usar bcrypt"

Control de la sesión → usas un comando slash
  /compact   (comprimir el contexto)
  /model     (cambiar el modelo)
  /cost      (ver cuánto cuesta)
```

### `/help` — El Punto de Partida

Siempre disponible. Lista todos los comandos con descripción breve.

```
> /help

Comandos disponibles:

  /agent          Gestionar subagentes (listar, ver logs, detener)
  /clear          Borrar todo el historial de la conversación actual
  /compact        Resumir el contexto de la conversación para reducir tokens
  /cost           Mostrar estadísticas de tokens y costo estimado
  /doctor         Diagnóstico del entorno: versiones, conectividad, permisos
  /help           Mostrar esta ayuda
  /init           Analizar el proyecto actual y generar/actualizar CLAUDE.md
  /model          Ver el modelo actual o cambiar a otro
  /pr_comments    Cargar comentarios sin resolver de un Pull Request de GitHub
  /review         Hacer code review de los cambios pendientes en git diff
  /terminal-setup Configurar atajos de teclado óptimos para Claude Code

Para empezar una tarea, simplemente escríbela en lenguaje natural.
```

### `/init` — Inicializar un Proyecto

`/init` es el primer comando que deberías ejecutar cuando empiezas a usar Claude Code en un proyecto. Realiza las siguientes acciones:

1. Lee la estructura de directorios del proyecto
2. Lee archivos clave de configuración: `package.json`, `pyproject.toml`, `pom.xml`, `Cargo.toml`, `go.mod`, `README.md`, etc.
3. Identifica el stack tecnológico (lenguaje, framework, DB, testing)
4. Detecta los comandos de build, test y lint
5. Genera el archivo `CLAUDE.md` con toda esa información estructurada

```
> /init

Analizando el proyecto en /mi/proyecto...

  ✓ Detectado lenguaje: Python 3.11
  ✓ Detectado framework: FastAPI 0.104
  ✓ Detectada base de datos: PostgreSQL (via SQLAlchemy 2.0)
  ✓ Detectado testing: pytest 7.4 + pytest-asyncio
  ✓ Detectado linting: ruff + black
  ✓ Leído: pyproject.toml
  ✓ Leído: README.md
  ✓ Analizada estructura src/

Generando CLAUDE.md...

  ✅ CLAUDE.md creado en /mi/proyecto/CLAUDE.md

¿Quieres que ajuste o amplíe alguna sección?
```

> **Nota:** `/init` no borra un `CLAUDE.md` existente. Si ya tienes uno, te preguntará si quieres sobreescribirlo o fusionar los nuevos hallazgos con el contenido existente.

### `/clear` — Limpiar el Contexto

El contexto en Claude Code es **acumulativo**: cada mensaje, cada archivo leído, cada output de comando se añade al contexto de la sesión. Esto es útil para que Claude recuerde lo que ha hecho, pero tiene un límite.

`/clear` borra **todo** el historial de la conversación actual:

```
> /clear

¿Estás seguro? Esto borrará toda la conversación actual.
El archivo CLAUDE.md se mantendrá en disco. (s/n): s

Contexto limpiado. Nueva conversación iniciada.
>
```

**¿Cuándo usar `/clear`?**
- Cuando terminas una tarea completamente diferente y empiezas otra nueva
- Cuando Claude parece "confundido" o mezcla conceptos de conversaciones anteriores
- Cuando el contexto está tan lleno que las respuestas se vuelven lentas o incoherentes
- Al comenzar un nuevo día de trabajo

**¿Qué se pierde con `/clear`?**
- Todo el historial de conversación de la sesión
- Los archivos que Claude había leído en memoria (tendrá que releerlos)
- El "razonamiento acumulado" sobre decisiones previas

**¿Qué NO se pierde con `/clear`?**
- El archivo `CLAUDE.md` (persiste en disco, se vuelve a leer al inicio)
- Los cambios que Claude hizo en tus archivos (ya están guardados en disco)
- Cualquier otro cambio real en el proyecto

### `/compact` — Comprimir sin Perder el Hilo

A diferencia de `/clear`, `/compact` no borra el contexto, lo **resume**. Claude lee toda la conversación actual y genera un resumen comprimido que captura lo esencial para poder continuar la tarea.

```
> /compact

Analizando la conversación actual para comprimirla...
  Longitud actual: 87,420 tokens

Generando resumen estructurado...

─────────────────────────────────────────────────────
RESUMEN COMPRIMIDO DE LA SESIÓN:

Objetivo: Migrar el módulo de autenticación de JWT simple a OAuth2.

Completado:
  ✅ Actualización de requirements.txt (python-jose → authlib)
  ✅ Nuevo OAuth2AuthService en src/auth/oauth2_service.py
  ✅ Tests unitarios para OAuth2AuthService (8 tests, todos pasan)

Pendiente:
  ⬜ Actualizar los endpoints en src/api/auth_routes.py
  ⬜ Actualizar la documentación en docs/auth.md

Contexto técnico clave:
  - Usamos python-jose para JWT (dependencia que se mantiene)
  - El client_id de OAuth está en .env como OAUTH_CLIENT_ID
  - Los tests usan pytest-asyncio con el fixture async_client
  - No romper la interfaz pública: login(), logout(), refresh()
─────────────────────────────────────────────────────

  Tokens antes del compact:   87,420
  Tokens después del compact:  2,840
  Reducción: 96.7%

Continuando con contexto comprimido. ¿Quieres que proceda con los pendientes?
```

**¿Cuándo usar `/compact`?**
- Cuando llevas más de 1 hora trabajando en la misma tarea
- Cuando `/cost` muestra más de 80,000-100,000 tokens
- Cuando las respuestas de Claude empiezan a ser más lentas
- Cuando necesitas continuar pero el contexto es demasiado grande

**La diferencia clave entre `/clear` y `/compact`:**

```
/clear                          /compact
─────────────────────────────   ────────────────────────────────
Antes:  [conversación 90k tok]  Antes:  [conversación 90k tok]
Después: [vacío]                Después: [resumen de ~3k tok]

Pierde todo el contexto         Mantiene lo esencial
Empieza desde cero              Claude recuerda qué estaba haciendo
Útil al cambiar de tarea        Útil para continuar la misma tarea
```

### `/cost` — Cuánto Está Costando Esto

Un control esencial para no llevarse sorpresas en la factura al final del mes.

```
> /cost

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ESTADÍSTICAS DE LA SESIÓN ACTUAL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Modelo:               claude-sonnet-4
  Duración sesión:      47 minutos

  Tokens de entrada:    134,820  → $0.40
  Tokens de salida:      22,340  → $0.34
  ───────────────────────────────────────
  Total estimado:                  $0.74

  Herramientas invocadas esta sesión:
    Read:       52 veces  (archivos leídos)
    Write:      18 veces  (archivos modificados/creados)
    Bash:       31 veces  (comandos ejecutados)
    WebSearch:   3 veces  (búsquedas en internet)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Interpretando las cifras:**

Los **tokens de entrada** incluyen todo lo que se envía a la API en cada request: el contenido de CLAUDE.md, el historial completo de conversación, el contenido de archivos leídos y los outputs de comandos ejecutados. Este número crece a lo largo de la sesión.

Los **tokens de salida** son las respuestas de Claude y el código que genera. Son más caros por token que los de entrada, pero suelen ser menos voluminosos.

Una sesión típica de 1 hora de trabajo intenso con `sonnet` cuesta entre **$0.30 y $2.00** dependiendo de cuántos archivos grandes se lean.

### `/model` — Elegir el Modelo Correcto

Claude Code puede usar diferentes modelos de Claude. Elegir el correcto tiene impacto directo en velocidad, calidad y costo.

```
> /model

Modelo actual: claude-sonnet-4

Modelos disponibles:

  1. claude-opus-4
     Calidad:    ████████████ Máxima
     Velocidad:  ████░░░░░░░░ Lenta (~3x más que sonnet)
     Costo:      ████████████ Alto (~5x más que sonnet)
     Mejor para: Diseño de arquitectura compleja, análisis
                 profundo de código, refactoring de sistemas
                 críticos donde la calidad es lo primero.

  2. claude-sonnet-4  ← ACTUAL
     Calidad:    █████████░░░ Alta
     Velocidad:  ████████░░░░ Normal
     Costo:      ████░░░░░░░░ Moderado
     Mejor para: Uso diario de desarrollo: refactoring,
                 generación de tests, debugging, la gran
                 mayoría de tareas cotidianas.

  3. claude-haiku-4
     Calidad:    ██████░░░░░░ Buena para tareas simples
     Velocidad:  ████████████ Muy rápida
     Costo:      █░░░░░░░░░░░ Muy barato (~10x menos que opus)
     Mejor para: Preguntas rápidas, exploración de código,
                 tareas simples de refactoring, explicaciones.

Selecciona (1-3) o Enter para mantener actual: _
```

**Regla práctica para elegir:**

```
¿Necesitas una explicación rápida o buscar algo en el código?
  → haiku (responde en segundos, muy barato)

¿Estás haciendo trabajo real de desarrollo: refactoring, tests, debugging?
  → sonnet (el mejor balance calidad/costo para el día a día)

¿Estás diseñando la arquitectura de un sistema, analizando todo
 un proyecto grande, o la decisión tiene consecuencias importantes?
  → opus (usa la máxima capacidad cuando realmente importa)
```

### `/review` — Code Review Antes del Commit

`/review` analiza los cambios actuales en tu repositorio Git (equivalente a inspeccionar `git diff`) y produce una revisión detallada con problemas categorizados por severidad.

```
> /review

Leyendo cambios pendientes en git diff...

Archivos modificados:
  M  src/auth/service.py      (+87, -23 líneas)
  M  src/auth/middleware.py   (+12, -4 líneas)
  A  tests/test_oauth.py      (+156 líneas, nuevo)
  M  requirements.txt         (+2, -1 líneas)

Analizando cambios en profundidad...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 RESULTADO DE LA REVISIÓN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

❌ BUG: src/auth/middleware.py, línea 67
   Si el header Authorization está mal formado (por ejemplo
   "Bearer" sin token), lanzará IndexError en parts[1].
   Se debe validar len(parts) == 2 antes de acceder.

⚠️  SEGURIDAD: src/auth/service.py, línea 134
   El refresh token generado no tiene tiempo de expiración.
   Un refresh token sin expiración es un riesgo de seguridad.
   Añadir: REFRESH_TOKEN_EXPIRE_DAYS = 30 en la configuración.

✅ src/auth/service.py (resto)
   Implementación de OAuth2 correcta y bien estructurada.

✅ tests/test_oauth.py
   Buena cobertura, uso correcto de pytest-asyncio.

⚠️  MEJORA: tests/test_oauth.py
   Falta test para el caso de refresh token expirado.
   Es un caso de error importante que debería estar cubierto.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RESUMEN: 1 bug crítico, 1 problema de seguridad, 1 mejora

¿Quieres que corrija los problemas encontrados? (s/n):
```

### `/doctor` — Diagnóstico del Entorno

Cuando algo no funciona bien, `/doctor` analiza el entorno completo y detecta problemas:

```
> /doctor

Ejecutando diagnóstico completo...

✅ Claude Code: v1.2.3 (última versión disponible)
✅ Node.js: v20.11.0 (cumple requisito >=18)
✅ API Key: Configurada y válida
✅ Conectividad con API de Anthropic: OK (latencia 234ms)
✅ Permisos de escritura en directorio actual: OK
✅ Git: v2.43.0 detectado
⚠️  Bash: /bin/bash versión 3.2 (recomendado 5.0+, algunas funciones pueden fallar)
❌ Herramienta 'ripgrep': No encontrada en PATH
   Claude Code usará grep como fallback, pero será más lento.
   Para instalar: 
     macOS:  brew install ripgrep
     Ubuntu: apt install ripgrep

Diagnóstico completado: 1 error no crítico, 1 advertencia.
```

### `/pr_comments` — Revisar Comentarios de un Pull Request

Si tienes un PR abierto en GitHub con comentarios de revisores sin resolver, Claude Code puede cargarlos y ayudarte a resolverlos:

```
> /pr_comments 247

Cargando comentarios del PR #247 en github.com/miorg/mirepo...
Autenticando con GitHub...

Se encontraron 4 comentarios sin resolver:

1. @john_reviewer en src/api/users.py:45
   "Este endpoint no valida que el usuario solo pueda
    ver su propio perfil, un usuario normal no debería
    poder ver los datos de otro usuario."

2. @john_reviewer en src/api/users.py:78
   "El password llega en texto plano y se guarda directamente.
    Debe hashearse con bcrypt antes de persistirse."

3. @maria_lead en tests/test_users.py:23
   "Falta test para el caso de usuario no encontrado (404)."

4. @maria_lead en README.md
   "La sección de Variables de Entorno está desactualizada,
    faltan OAUTH_CLIENT_ID y OAUTH_CLIENT_SECRET."

¿Quieres que resuelva estos comentarios? Puedo hacerlo todos
a la vez o uno por uno. (s/n):
```

---

## 5. El Arte del Prompting Efectivo

### Por Qué el Prompt Importa Tanto

Claude Code es poderoso, pero su efectividad depende **directamente** de la calidad de tu prompt. Un prompt vago produce resultados vagos. Un prompt preciso produce resultados precisos. Esta es, sin exageración, la habilidad más importante para ser productivo con Claude Code.

Veamos un ejemplo extremo de la diferencia:

```
❌ PROMPT VAGO: "arregla el código de auth"

¿Qué hace Claude?
  - Lee todos los archivos que tengan "auth" en el nombre
    (puede ser mucho contexto)
  - Toma decisiones arbitrarias sobre qué "arreglar"
    (puede cambiar cosas que funcionan y que no querías tocar)
  - Puede ignorar el problema real que te preocupa
  - Usa tokens en exploración innecesaria
  - El resultado puede ser correcto o completamente errado
```

```
✅ PROMPT PRECISO:
"En src/auth/service.py, la función verify_token() lanza
un KeyError sin mensaje claro cuando el campo 'exp' no existe
en el payload del JWT. Esto ocurre cuando alguien envía un
token JWT que fue generado por otro sistema.

Por favor:
1. Lee la función actual
2. Escribe un test en tests/test_auth.py que reproduzca este caso
   (el test debe fallar con el código actual)
3. Corrige la función añadiendo validación explícita que lance
   AuthenticationError('Token inválido: falta campo exp')
4. Verifica que el test pasa después del fix"

¿Qué hace Claude?
  - Va directamente al archivo correcto
  - Sabe exactamente qué función tiene el problema
  - Sabe qué excepción ocurre y en qué condición
  - Sabe dónde poner el test y qué debe comprobar
  - Sabe qué excepción debe lanzar el fix
  - Usa tokens eficientemente, sin exploración innecesaria
```

### La Anatomía de un Prompt Excelente

Un prompt de alta calidad tiene estos componentes. No todos son siempre necesarios, pero conocerlos te ayuda a saber qué incluir según la situación:

```
┌─────────────────────────────────────────────────────────┐
│                  COMPONENTES DE UN BUEN PROMPT          │
│                                                         │
│  1. CONTEXTO                                            │
│     ¿Dónde estamos? ¿Cuál es la situación actual?       │
│     "Tenemos una API FastAPI con auth basada en JWT..."  │
│                                                         │
│  2. TAREA ESPECÍFICA                                    │
│     ¿Qué hay que hacer exactamente?                     │
│     Verbo de acción + objeto concreto                   │
│     "Migra src/payments/client.py para usar httpx..."   │
│                                                         │
│  3. RESTRICCIONES                                       │
│     ¿Qué NO hay que tocar? ¿Qué límites existen?        │
│     "No cambies la firma de los métodos públicos..."    │
│                                                         │
│  4. CRITERIO DE ÉXITO                                   │
│     ¿Cómo sabremos que está hecho? ¿Cómo verificarlo?   │
│     "Todos los tests en tests/test_payments.py pasan"   │
│                                                         │
│  5. FORMATO DEL OUTPUT (opcional)                       │
│     ¿Cómo quiero recibir el resultado?                  │
│     "Muéstrame el plan antes de ejecutar"               │
└─────────────────────────────────────────────────────────┘
```

### Patrones de Prompt por Categoría de Tarea

#### REFACTORING

```
PATRÓN:
"Refactoriza [archivo específico] para [objetivo concreto].
Restricciones: [qué no cambiar].
Después de refactorizar, ejecuta [comando de tests] y 
asegúrate que todos pasan."

EJEMPLO REAL:
"Refactoriza src/services/user_service.py para extraer toda
la lógica de validación en una clase UserValidator separada
que irá en src/validators/user_validator.py.

Restricciones:
- Los métodos públicos de UserService (create_user, update_user,
  delete_user) no pueden cambiar su firma ni comportamiento
- UserService debe importar y usar UserValidator internamente

Después del refactoring ejecuta:
  pytest tests/test_user_service.py -v
y asegúrate que todos pasan."
```

#### GENERACIÓN DE TESTS

```
PATRÓN:
"Genera tests para [archivo/clase/función] en [ruta destino].
Framework: [pytest/jest/junit/etc].
Casos que deben cubrirse:
  - [caso happy path 1]
  - [caso de error 1]
  - [caso edge case 1]
Mockear: [dependencias externas que no deben ejecutarse realmente].
Patrón de nombres: [convención del proyecto]."

EJEMPLO REAL:
"Genera tests unitarios para la clase EmailService en
src/services/email_service.py.

Destino: tests/unit/test_email_service.py
Framework: pytest + unittest.mock

Casos que deben cubrirse:
  - Envío exitoso de email simple
  - Envío exitoso de email con attachment
  - Fallo de SMTP (simular SMTPException)
  - Email con dirección de destinatario inválida
  - Retry automático tras primer fallo (la clase tiene 3 reintentos)

Mockear: smtplib.SMTP (no enviar emails reales en tests)
Patrón de nombres: test_send_email_<caso_en_snake_case>"
```

#### DEBUGGING CON STACK TRACE

```
PATRÓN:
"Hay un bug en [función/endpoint]:

Error exacto: [mensaje de error]
Stack trace completo:
[pegar el stack trace]

Condición en que ocurre: [cuándo pasa exactamente]
Condición en que NO ocurre: [cuándo no pasa, si lo sabes]

Por favor:
1. Reproduce el bug en un test que falle
2. Corrige el bug
3. Verifica que el test pasa después del fix"

EJEMPLO REAL:
"Hay un bug en la API desde ayer en producción:

Error: KeyError: 'user_id'
Stack trace:
  File 'src/api/orders.py', line 89, in create_order
    user = context['user_id']
  File 'src/middleware/auth.py', line 45, in verify_request
    context = jwt.decode(token, SECRET_KEY)
  ...

Ocurre SOLO cuando la request viene del canal 'mobile' (header
X-Client-Type: mobile). Las requests desde 'web' funcionan bien.

1. Reproduce en tests/test_orders.py con una request simulada
   de canal mobile
2. Encuentra y corrige la causa
3. El test debe pasar"
```

#### ANÁLISIS SIN MODIFICACIONES

Cuando quieres explorar sin que Claude cambie nada:

```
PATRÓN:
"Analiza [qué] y dime [qué información necesito].
NO modifiques ningún archivo. Solo lee y reporta."

EJEMPLO REAL:
"Analiza la cobertura de tests de este proyecto sin modificar nada:
1. Lista todos los métodos públicos en src/ que NO tienen ningún test
2. Identifica qué módulos tienen estimativamente menos del 50% de cobertura
3. Ordena la lista por impacto (los más críticos para el negocio primero)

No modifiques ni crees ningún archivo. Solo el análisis."
```

### Técnicas Avanzadas de Prompting

#### Técnica 1: "Primero el Plan, Luego la Ejecución"

Esta es la técnica más valiosa. Pide a Claude que te muestre el plan completo antes de hacer **ningún** cambio. Así puedes corregir el rumbo antes de que Claude modifique archivos.

```
> Vamos a añadir rate limiting a todos los endpoints de la API.

  IMPORTANTE: Primero muéstrame el plan detallado:
  - Qué archivos vas a modificar
  - Qué librería vas a usar y por qué
  - Cómo lo implementarás en cada archivo
  - Qué tests añadirás

  Espera mi aprobación explícita antes de escribir
  una sola línea de código.
```

Cuando Claude presenta el plan, puedes ajustarlo:

```
> El plan está bien pero con estos cambios:
  1. Usa slowapi en lugar de flask-limiter (ya tenemos FastAPI)
  2. El rate limiting de /api/auth/ debe ser más estricto: 5/minuto
  3. No toques tests/integration/ por ahora, solo tests/unit/

  Con estos ajustes, ¿puedes ejecutar el plan?
```

#### Técnica 2: "Paso a Paso con Confirmación"

Para migraciones o cambios que tocan muchos archivos, divide el trabajo en pasos y pide confirmación entre cada uno:

```
> Vamos a migrar la base de datos de SQLite a PostgreSQL.
  Hazlo en estos pasos y ESPERA que yo diga "ok" después de cada uno:

  Paso 1: Solo muéstrame los cambios en requirements.txt
          (no los apliques todavía)
  Paso 2: Muéstrame los cambios en la configuración de conexión
  Paso 3: Muéstrame si algún modelo ORM necesita cambios
  Paso 4: Aplica todos los cambios y ejecuta los tests

  Empieza con el Paso 1.
```

#### Técnica 3: "Exploración Antes de Acción"

Cuando no conoces bien la zona del código que hay que tocar:

```
> Sin modificar nada, léete completamente el módulo de pagos
  (src/payments/) y explícame:
  1. Cómo fluye una transacción desde el endpoint hasta la DB
  2. Qué validaciones se hacen y en qué punto
  3. Qué casos de error existen y cómo se manejan actualmente
  4. Qué dependencias externas usa (Stripe, Redis, etc.)

  Necesito entender esto bien antes de pedirte que hagas cambios.
  No toques nada.
```

#### Técnica 4: "Contexto de Error Completo"

Al reportar un bug, incluye **siempre** el error completo. Claude no puede adivinar el stack trace.

```
> Tengo este error en producción desde esta mañana. Necesito que lo investigues:

  ERROR 2026-03-15 09:34:12 [payment-worker]
  Traceback (most recent call last):
    File "/app/src/workers/payment_worker.py", line 45, in process
      result = await self.payment_service.charge(order_id, amount)
    File "/app/src/services/payment_service.py", line 167, in charge
      charge = await stripe.charge.create(amount=amount_cents, ...)
    File "/app/vendor/stripe/api_resources/charge.py", line 23, in create
      return cls._static_request("post", cls.class_url(), **params)
  stripe.error.InvalidRequestError: Invalid integer: amount must be a
  positive integer. Got: 99.99

  Ocurre con ordenes donde el amount tiene decimales (ej: $9.99, $4.50).
  No ocurre con amounts que son múltiplos exactos de 100 (ej: $10.00, $20.00).

  1. Lee el código relevante
  2. Reproduce el bug en un test que falle
  3. Corrígelo
  4. El test debe pasar después del fix
```

### Anti-patrones: Qué NO Hacer

| Anti-patrón | Por qué es un problema | Cómo corregirlo |
|-------------|----------------------|-----------------|
| `"mejora el código"` | "Mejor" es subjetivo; Claude puede cambiar todo arbitrariamente | Define el criterio de mejora concreto: "mejora la legibilidad extrayendo las condicionales largas a métodos con nombres descriptivos" |
| `"arregla los tests que fallan"` | ¿Cuáles tests? ¿Por qué fallan? Claude no lo sabe | Pega el output completo del test fallido |
| `"refactoriza el proyecto entero"` | Demasiado amplio; Claude puede cambiar demasiadas cosas | Especifica un módulo: "refactoriza src/auth/" |
| `"como antes pero mejor"` | Claude no recuerda sesiones anteriores | Describe explícitamente lo que quieres |
| Prompt sin restricciones | Claude puede tocar archivos que no deberías cambiar | Siempre incluye "no toques X" |
| Error sin stack trace | Claude no puede reproducir ni debuggear el problema | Siempre incluye el error completo |

---

## 6. CLAUDE.md: El Manual de Instrucciones de tu Proyecto

### ¿Por Qué Existe CLAUDE.md?

Cada vez que abres Claude Code en un proyecto, la sesión empieza completamente "en blanco". Claude no recuerda:
- La sesión anterior
- El stack tecnológico del proyecto
- Tus convenciones de código
- Los comandos para ejecutar tests
- Qué archivos son críticos y no deben tocarse
- El contexto de negocio de la aplicación

Sin CLAUDE.md, tendrías que explicar todo esto desde cero en cada sesión. Con CLAUDE.md, le das a Claude toda esa información de forma **permanente y automática**.

> **CLAUDE.md es el único mecanismo de memoria persistente de Claude Code entre sesiones.**

### Cómo Funciona el Mecanismo

Cuando ejecutas `claude` en un directorio, lo primero que hace el CLI es buscar y leer todos los archivos CLAUDE.md relevantes. Su contenido se añade como contexto base antes de que empieces a escribir.

```
Orden de búsqueda (todos se leen si existen):
  
  ~/.claude/CLAUDE.md              ← Instrucciones globales del usuario
  /mi/proyecto/CLAUDE.md           ← Instrucciones del proyecto raíz
  /mi/proyecto/backend/CLAUDE.md   ← Instrucciones del subdirectorio
  
  (Si ejecutas claude en /mi/proyecto/backend/)
```

Si existen varios, todos se fusionan. Las instrucciones más específicas (del subdirectorio) tienen prioridad sobre las más generales en caso de conflicto.

### CLAUDE.md Global: Tus Preferencias Personales

El archivo `~/.claude/CLAUDE.md` aplica a **todos** tus proyectos. Es el lugar para preferencias personales que quieres en cualquier contexto:

```markdown
# ~/.claude/CLAUDE.md

## Idioma y Comunicación
- Responde siempre en español
- Cuando vayas a hacer algo significativo, explica brevemente por qué
- Si tienes dos opciones y ninguna es claramente mejor, pregúntame
  antes de elegir una arbitrariamente

## Mi Estilo de Código (General)
- Código explícito sobre código "mágico" o muy compacto
- Siempre añadir type hints/tipos estáticos
- Nombres de variables descriptivos (no abreviaciones)
- Comentarios solo cuando el código no es auto-explicativo

## Forma de Trabajar Conmigo
- Para cambios que afectan a más de 3 archivos: muéstrame el plan primero
- Después de cualquier cambio de código, ejecuta los tests del módulo afectado
- Si los tests fallan, corrígelo antes de continuar o pregúntame
- No hagas git commit automáticamente

## Seguridad
- Nunca hardcodear credenciales, API keys o secrets en el código
- Si encuentras credenciales hardcodeadas en código existente, avísame
```

### CLAUDE.md del Proyecto: La Guía Completa

Este es el más importante. Aquí describes todo lo que Claude necesita saber para este proyecto específico:

```markdown
# CLAUDE.md — Sistema de Gestión de Inventario

## ¿Qué es este proyecto?
API REST para gestión de inventario de retail. Permite a tiendas
gestionar stock, registrar movimientos y recibir alertas de reposición.
Es un servicio interno usado por ~50 tiendas con ~500 peticiones/minuto.

## Stack Tecnológico
- **Lenguaje**: Python 3.11
- **Framework web**: FastAPI 0.104
- **Base de datos**: PostgreSQL 15 via SQLAlchemy 2.0 (modo async)
- **Migraciones de BD**: Alembic
- **Tests**: pytest 7.4 + pytest-asyncio + httpx (para tests de endpoints)
- **Validación**: Pydantic v2
- **Autenticación**: JWT via python-jose
- **Caché**: Redis via redis-py (async)
- **Calidad de código**: ruff (linting) + black (formatting)
- **Contenedores**: Docker + docker-compose para dependencias de dev

## Estructura del Proyecto
```
src/
  api/            → Routers FastAPI (solo HTTP: request/response)
  services/       → Lógica de negocio (sin dependencia de HTTP ni DB directa)
  repositories/   → Acceso a base de datos (solo SQL, sin lógica de negocio)
  models/         → Modelos SQLAlchemy (mapeo a tablas)
  schemas/        → Schemas Pydantic (validación de request/response)
  core/           → Configuración, seguridad, excepciones, dependencias DI
tests/
  unit/           → Tests de services y repositories (mocks de DB)
  integration/    → Tests de endpoints completos (DB de test real)
  conftest.py     → Fixtures compartidos (client, db_session, factories)
```

## Comandos Esenciales

```bash
# Tests
pytest tests/unit/ -v                              # Solo tests unitarios
pytest tests/integration/ -v                       # Tests de integración
pytest tests/ --cov=src --cov-report=term-missing  # Con cobertura

# Servidor de desarrollo
docker-compose up -d           # Levanta PostgreSQL y Redis en Docker
uvicorn src.main:app --reload  # Servidor con hot-reload

# Base de datos
alembic upgrade head                              # Aplicar migraciones pendientes
alembic revision --autogenerate -m "descripción"  # Nueva migración automática

# Calidad de código
ruff check src/ tests/ --fix   # Linting con auto-corrección
black src/ tests/              # Formateo automático
```

## Arquitectura por Capas — IMPORTANTE

Esta es la regla más importante del proyecto:

```
HTTP Request
    ↓
  ROUTER (api/)          → Solo valida request y llama al service
    ↓                       NO importa repositories directamente
  SERVICE (services/)    → Contiene la lógica de negocio
    ↓                       Puede llamar a varios repositories
  REPOSITORY (repos/)    → Solo hace operaciones de base de datos
    ↓                       NO tiene lógica de negocio
  DATABASE
```

- Los **routers** no importan repositories. Solo llaman services.
- Los **services** no importan directamente SQLAlchemy ni hacen queries.
- Los **repositories** no contienen condicionales de negocio.

## Convenciones de Código

### Nomenclatura
- Clases: PascalCase → `UserService`, `ProductRepository`, `StockMovement`
- Funciones y variables: snake_case → `get_user_by_id`, `stock_level`
- Constantes: UPPER_SNAKE_CASE → `MAX_STOCK_LEVEL`, `DEFAULT_PAGE_SIZE`
- Archivos: snake_case → `user_service.py`, `stock_repository.py`
- URLs de endpoints: kebab-case → `/api/stock-movements/`, `/api/low-stock/`

### Tests
- Tests unitarios: usan `unittest.mock.AsyncMock` para repositories
- Tests de integración: usan la DB de test configurada en conftest.py
- Cobertura mínima requerida: 80%
- Patrón de nombres: `test_<método_o_endpoint>_<caso_de_uso>`
  Ejemplo: `test_reduce_stock_when_insufficient_raises_error`

### Manejo de Errores
- Todas las excepciones de negocio heredan de `AppError` (en `src/core/exceptions.py`)
- Los errores de validación de request los maneja Pydantic automáticamente
- Los errores de base de datos se capturan en el repository y se relanzam como `AppError`
- Nunca propagar `sqlalchemy.exc` fuera de los repositories

## Archivos Críticos: NO Modificar sin Aprobación Explícita
- `src/core/config.py` → Configuración central del sistema
- `src/core/security.py` → Toda la lógica de autenticación
- `alembic/versions/*.py` → Migraciones ya aplicadas en producción
- `docker-compose.yml` → Infraestructura de desarrollo compartida

## Contexto de Negocio (para entender las reglas)
Un "movimiento de stock" puede ser:
  - ENTRADA: recepción de mercancía de proveedor
  - SALIDA: venta o consumo
  - AJUSTE: corrección manual por inventario físico
  - TRANSFERENCIA: movimiento entre ubicaciones de la misma tienda

**Regla de negocio crítica**: El stock de un producto en una ubicación
NUNCA puede ser negativo. Si un movimiento causaría stock negativo,
debe lanzarse `InsufficientStockError` (no guardarse con valor negativo).
```

### Generar y Mantener CLAUDE.md

No tienes que escribir CLAUDE.md desde cero. Usa `/init` y luego refínalo:

```
# Generar el inicial
> /init

# Añadir lo que faltó
> Actualiza el CLAUDE.md con estas reglas adicionales que no detectaste:
  1. La arquitectura por capas: los routers no importan repositories
  2. La regla de negocio del stock negativo
  3. Los archivos que no deben tocarse sin aprobación

# Cuando algo cambia en el proyecto
> Hemos añadido Celery para tareas asíncronas. Actualiza CLAUDE.md:
  - Nueva sección en Stack: Celery 5.3 para tareas async
  - Nuevo directorio: src/tasks/ con sus tests en tests/unit/tasks/
  - Nuevo comando: celery -A src.tasks worker --loglevel=info
```

---

## 7. Subagentes Especializados

### El Problema que Resuelven

Imagina documentar un proyecto con 50 módulos usando Claude Code sin subagentes:

```
Sesión normal (sin subagentes):

Claude lee módulo 1 → escribe doc 1  [contexto: 5k tokens]
Claude lee módulo 2 → escribe doc 2  [contexto: 10k tokens]
Claude lee módulo 3 → escribe doc 3  [contexto: 15k tokens]
...
Claude lee módulo 30 → escribe doc 30 [contexto: 150k tokens ← casi lleno]
...
La calidad se degrada. La velocidad decrece. El costo sube.
```

Con subagentes:

```
Claude Principal divide el trabajo:
  ├── Subagente A: documenta módulos 1-10  [contexto limpio: 0-50k tokens]
  ├── Subagente B: documenta módulos 11-20 [contexto limpio: 0-50k tokens]
  ├── Subagente C: documenta módulos 21-30 [contexto limpio: 0-50k tokens]
  ├── Subagente D: documenta módulos 31-40 [contexto limpio: 0-50k tokens]
  └── Subagente E: documenta módulos 41-50 [contexto limpio: 0-50k tokens]

Los 5 trabajan en PARALELO.
Cada uno tiene contexto limpio y eficiente.
El principal recoge los 5 resultados y los integra.

Resultado: ~5x más rápido, mejor calidad, sin degradación.
```

### Qué es un Subagente Exactamente

Un subagente es una **instancia completamente separada de Claude** que:

- Tiene su **propio contexto independiente**: no comparte memoria con el agente principal ni con otros subagentes
- Puede usar las **mismas herramientas**: Read, Write, Bash, WebSearch, etc.
- Trabaja en una **tarea específica y acotada** asignada por el principal
- Es **creado mediante la herramienta `Agent`** internamente por Claude Code
- Cuando termina, **devuelve su resultado** al agente que lo creó

```
ANALOGÍA:

  Claude Principal = Director de proyecto
  Subagentes = Desarrolladores del equipo

  El director divide el trabajo, asigna tareas a cada desarrollador
  (con un briefing claro), y cuando todos terminan, integra los resultados.
  
  Cada desarrollador trabaja en su parte sin saber qué hacen los otros.
```

### Cómo Crear Subagentes

**La forma más simple es pedírselo en lenguaje natural.** Claude Code decide automáticamente cuándo y cómo dividir el trabajo:

```
> Necesito generar tests para todos los módulos en src/services/.
  Son 8 archivos. Usa subagentes para procesar varios en paralelo
  y ser más eficiente. El framework es pytest + pytest-asyncio.
  Los tests van en tests/unit/services/.
```

Claude detecta que las 8 tareas son independientes y puede crearlas en paralelo.

**Forma explícita para control total:**

```
> Organiza el trabajo con estos subagentes en paralelo:
  
  Subagente A:
    - Tarea: Generar tests para src/auth/ (AuthService, TokenService)
    - Destino: tests/unit/auth/
    - Contexto especial: usa pytest-asyncio, mockear la DB
  
  Subagente B:
    - Tarea: Generar tests para src/payments/ (PaymentService)
    - Destino: tests/unit/payments/
    - Contexto especial: mockear la API de Stripe, no hacer requests reales
  
  Subagente C:
    - Tarea: Generar tests para src/notifications/ (EmailService, SMSService)
    - Destino: tests/unit/notifications/
    - Contexto especial: mockear smtplib y twilio
  
  Cuando los tres terminen, ejecuta toda la suite:
  pytest tests/unit/ -v --cov=src
```

### Cuándo Usar Subagentes vs. Trabajar en Serie

La clave es si las subtareas son **independientes** entre sí:

```
INDEPENDIENTES → Paralelizables → Usar subagentes ✓
  - Documentar módulo A y módulo B (no dependen entre sí)
  - Generar tests para diferentes clases
  - Analizar diferentes partes del código
  - Migrar archivos que no se importan entre sí

DEPENDIENTES → Deben ser secuenciales → No usar subagentes ✗
  - Crear la clase User y luego crear UserService que la usa
  - Escribir una migración de DB y luego actualizar los modelos
  - Instalar dependencias y luego escribir código que las usa
  - El resultado de la tarea A es entrada de la tarea B
```

### El Comando `/agent`

Una vez que hay subagentes trabajando, puedes monitorearlos:

```
> /agent list

Subagentes activos (3):
  ID  Estado        Progreso  Tarea
  ─────────────────────────────────────────────────────────
   1  ✅ Completo   100%      Tests para src/auth/ (2 archivos)
   2  🔄 En curso    65%      Tests para src/payments/ (1 de 3 archivos)
   3  🔄 En curso    30%      Tests para src/notifications/ (1 de 2 archivos)


> /agent logs 2

Log del Subagente 2 (payments):
  [14:23:01] Iniciado
  [14:23:02] Read(src/payments/payment_service.py) → 312 líneas
  [14:23:04] Read(src/payments/stripe_client.py) → 145 líneas
  [14:23:06] Write(tests/unit/payments/test_payment_service.py) → 18 tests generados
  [14:23:08] Bash(pytest tests/unit/payments/test_payment_service.py -v) → 18 passed
  [14:23:10] Read(src/payments/refund_service.py) → en progreso...


> /agent stop 3

¿Detener el Subagente 3? Perderás su trabajo en curso. (s/n): n
```

### Limitaciones de los Subagentes

- **Sin comunicación entre subagentes**: Si el Subagente A descubre algo importante, el Subagente B no lo sabe. Solo el principal puede comunicarles información.
- **El resultado hay que integrarlo**: Si dos subagentes generan archivos que se solapan o contradicen, el principal tiene que resolver el conflicto.
- **Costo multiplicado por base**: Cada subagente tiene su propio overhead de tokens (CLAUDE.md se lee N veces, una por subagente). Sin embargo, al tener contextos más pequeños, el costo por subtarea suele ser menor que hacerlo todo junto.
- **No son hilos del sistema operativo**: Son llamadas a la API, no threads. La "paralelización" es en términos de tiempo de respuesta de la API, no de CPU local.

---

## 8. MCP: Model Context Protocol

### ¿Qué es MCP y Por Qué Cambia Todo?

**MCP (Model Context Protocol)** es un protocolo abierto creado por Anthropic que define un estándar universal para que los modelos de IA se conecten con herramientas y fuentes de datos externas.

**Sin MCP**, Claude Code solo puede:
- Leer/escribir archivos en tu máquina local
- Ejecutar comandos de terminal
- Buscar en internet públicamente

**Con MCP**, además puede:
- Hacer consultas a tu **base de datos** directamente (PostgreSQL, MySQL, MongoDB)
- Leer y escribir en **servicios externos** (GitHub Issues, Jira, Notion, Slack)
- Acceder a **APIs internas** de tu empresa
- Leer logs de **sistemas de monitoreo** (Datadog, CloudWatch)
- Controlar un **navegador web** (para scraping o testing E2E)
- Interactuar con **servicios cloud** (AWS S3, GCP Storage)

**Analogía:** MCP es a las integraciones de IA lo que USB es a los periféricos de computadora. Antes de USB, cada periférico tenía su propio puerto incompatible (puerto serial, paralelo, PS/2...). USB creó un estándar universal: cualquier periférico conecta con cualquier computadora. MCP hace lo mismo para los agentes de IA y las herramientas.

### Arquitectura de MCP

```
┌─────────────────────────────────────────────────────────┐
│              CLAUDE CODE (cliente MCP)                  │
│                                                         │
│  Cuando Claude necesita datos de la DB:                 │
│  "Quiero ver las órdenes pendientes de las últimas 48h" │
│                        │                               │
│                        ▼  llamada MCP estándar          │
└────────────────────────┼────────────────────────────────┘
                         │
         ┌───────────────┼──────────────────┐
         ▼               ▼                  ▼
┌──────────────┐ ┌──────────────┐  ┌──────────────┐
│  MCP Server  │ │  MCP Server  │  │  MCP Server  │
│  PostgreSQL  │ │    GitHub    │  │    Notion    │
│              │ │              │  │              │
│  Ejecuta     │ │  Llama a     │  │  Llama a     │
│  queries SQL │ │  GitHub API  │  │  Notion API  │
└──────┬───────┘ └──────┬───────┘  └──────┬───────┘
       │                │                  │
       ▼                ▼                  ▼
  Tu DB local       GitHub.com         Notion.so
```

Cada MCP server es un proceso ligero que:
1. Expone sus "herramientas" al cliente (Claude Code)
2. Recibe llamadas cuando Claude quiere usar una herramienta
3. Ejecuta la operación real (query SQL, llamada API, etc.)
4. Devuelve el resultado a Claude

### Configurar MCP en Claude Code

La configuración se hace en un archivo `.mcp.json` en la raíz del proyecto (para que aplique a ese proyecto) o en `~/.claude/settings.json` (para todos los proyectos).

**Ejemplo: Conectar Claude Code a PostgreSQL**

```json
// .mcp.json en la raíz del proyecto
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://user:password@localhost:5432/mi_base_de_datos"
      }
    }
  }
}
```

Una vez configurado, Claude Code tiene acceso directo a la base de datos:

```
> Muéstrame las últimas 10 órdenes con status 'pending' que
  fueron creadas hace más de 48 horas, ordenadas por antigüedad.

● MCP(postgres): 
  SELECT o.id, o.created_at, o.total, u.email
  FROM orders o
  JOIN users u ON o.user_id = u.id
  WHERE o.status = 'pending'
  AND o.created_at < NOW() - INTERVAL '48 hours'
  ORDER BY o.created_at ASC
  LIMIT 10

Resultado (7 registros):
  ID    | Creada              | Total   | Email usuario
  ──────|─────────────────────|─────────|──────────────────────
  10198 | 2026-03-12 14:45   | $89.99  | cliente1@ejemplo.com
  10201 | 2026-03-12 16:30   | $234.50 | cliente2@ejemplo.com
  10234 | 2026-03-13 09:12   | $45.00  | cliente3@ejemplo.com
  ...

¿Quieres que analice por qué están pendientes o que genere
código para un job que las procese automáticamente?
```

**Ejemplo: Conectar Claude Code a GitHub**

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxxxxxx"
      }
    }
  }
}
```

```
> Lista todos los issues abiertos con label 'bug' que no tienen
  asignado nadie y fueron creados hace más de 7 días.
  Para cada uno, busca en el código si hay algo relacionado con
  el error descrito.
```

**Ejemplo: Configurar múltiples MCP servers**

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://..."
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..."
      }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-...",
        "SLACK_TEAM_ID": "T0..."
      }
    }
  }
}
```

### MCP Servers Oficiales Disponibles

| Server | Qué conecta | Instalación (npx) |
|--------|-------------|-------------------|
| `server-postgres` | PostgreSQL | `@modelcontextprotocol/server-postgres` |
| `server-github` | GitHub API completa | `@modelcontextprotocol/server-github` |
| `server-filesystem` | Sistema de archivos con control fino | `@modelcontextprotocol/server-filesystem` |
| `server-brave-search` | Búsqueda web via Brave | `@modelcontextprotocol/server-brave-search` |
| `server-puppeteer` | Control de navegador Chrome | `@modelcontextprotocol/server-puppeteer` |
| `server-slack` | Slack (leer/enviar mensajes) | `@modelcontextprotocol/server-slack` |
| `server-google-drive` | Google Drive | `@modelcontextprotocol/server-gdrive` |
| `server-sqlite` | SQLite | `@modelcontextprotocol/server-sqlite` |

Lista completa y actualizada: https://github.com/modelcontextprotocol/servers

---

## 9. Hooks: Automatizar el Comportamiento

### ¿Qué son los Hooks y Para Qué Sirven?

Los **hooks** son scripts que se ejecutan automáticamente cuando ocurren eventos específicos en Claude Code. Piensa en ellos como los git hooks, pero para el ciclo de vida de las acciones de Claude.

Sin hooks, si quieres que después de cada cambio de código se ejecute el linter, tienes que pedírselo a Claude en cada prompt. Con hooks, eso sucede **automáticamente** sin que tengas que mencionarlo.

**Casos de uso típicos de hooks:**
- Ejecutar el linter automáticamente cada vez que Claude escribe un archivo de código
- Formatear el código después de cada escritura
- Recibir notificaciones del sistema cuando Claude termina una tarea larga
- Mantener un log de auditoría de todos los cambios que hace Claude
- Bloquear operaciones peligrosas en ciertos archivos o directorios

### Tipos de Eventos (Cuándo se Ejecutan)

| Tipo de Hook | Cuándo se ejecuta | Caso de uso típico |
|-------------|-------------------|--------------------|
| `PreToolUse` | Justo antes de que Claude use una herramienta | Validar, bloquear, loggear antes de la acción |
| `PostToolUse` | Justo después de que Claude usa una herramienta | Verificar resultado, ejecutar acciones consecuentes |
| `Notification` | Cuando Claude necesita atención del usuario | Notificaciones del sistema operativo |
| `Stop` | Cuando Claude termina de trabajar en la tarea | Ejecutar verificaciones finales, notificar |

### Configuración de Hooks

Los hooks se configuran en `~/.claude/settings.json` (para todos los proyectos) o en `.claude/settings.json` dentro del proyecto (solo para ese proyecto):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "ruff check ${file} --fix"
          }
        ]
      }
    ]
  }
}
```

**Variables disponibles en los comandos:**
- `${file}` → ruta del archivo afectado (en hooks de Write/Read)
- `${command}` → el comando ejecutado (en hooks de Bash)
- `${tool}` → nombre de la herramienta usada

### Ejemplos Prácticos de Hooks

#### Hook 1: Linting Automático después de Cada Escritura

Este es el hook más útil para proyectos Python. Cada vez que Claude escribe o modifica un archivo `.py`, se ejecuta el linter automáticamente:

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "if [[ '${file}' == *.py ]]; then ruff check '${file}' --fix && echo '✅ Lint OK'; fi"
          }
        ]
      }
    ]
  }
}
```

**¿Por qué es valioso?** Claude puede generar código perfectamente funcional que sin embargo viola las reglas de estilo del proyecto. Con este hook, el linting se aplica automáticamente sin que tengas que pedírselo en cada prompt ni Claude tenga que acordarse.

#### Hook 2: Formateo Automático (Python + TypeScript)

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "case '${file}' in *.py) black '${file}' ;; *.ts|*.tsx) npx prettier --write '${file}' ;; esac"
          }
        ]
      }
    ]
  }
}
```

#### Hook 3: Notificaciones cuando Claude Termina

Para tareas largas (10-30 minutos), recibir una notificación del sistema operativo cuando terminen:

```json
// macOS
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude ha terminado la tarea\" with title \"Claude Code\" sound name \"Glass\"'"
          }
        ]
      }
    ]
  }
}
```

```json
// Linux (requiere notify-send: sudo apt install libnotify-bin)
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Tarea completada ✅' --icon=dialog-information --expire-time=5000"
          }
        ]
      }
    ]
  }
}
```

#### Hook 4: Log de Auditoría Completo

Si necesitas registrar exactamente qué cambió Claude (útil en equipos o proyectos regulados):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$(date -u +%Y-%m-%dT%H:%M:%SZ) WRITE ${file}\" >> ~/.claude/audit.log"
          }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$(date -u +%Y-%m-%dT%H:%M:%SZ) BASH '${command}'\" >> ~/.claude/audit.log"
          }
        ]
      }
    ]
  }
}
```

#### Hook 5: Proteger Archivos Críticos

Si hay archivos que Claude nunca debe modificar (por ejemplo, archivos de configuración de producción):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "if echo '${file}' | grep -qE 'config/production|.env.prod|alembic/versions/'; then echo 'ERROR: Archivo protegido - requiere aprobación manual'; exit 1; fi"
          }
        ]
      }
    ]
  }
}
```

---

## 10. Casos de Uso Prácticos Paso a Paso

### Caso 1: Refactorizar Código Legacy con Control Total

**Situación:** `src/auth.py` tiene 400 líneas, sin tipos, sin tests, mezclando responsabilidades.

```
> Necesito refactorizar src/auth.py. Antes de tocar NADA:
  Lee el archivo y dime:
  1. Qué hace cada función
  2. Qué problemas tiene (tipos faltantes, responsabilidades mezcladas, etc.)
  3. Tu propuesta de cómo dividirlo en módulos más coherentes

  NO hagas cambios todavía.

─────────────────────────────────────────────────────
● Read(src/auth.py)

El archivo tiene 18 funciones. Análisis:

Problemas encontrados:
  ❌ Ninguna función tiene type hints
  ❌ hash_password usa MD5 (inseguro; debería usar bcrypt)
  ❌ Responsabilidades mezcladas: crypto, tokens JWT y
     operaciones de DB conviven en el mismo archivo
  ❌ verify_token no maneja TokenExpiredError

Propuesta de estructura:
  src/auth/
    __init__.py     ← re-exporta todo para compatibilidad
    crypto.py       ← hash_password(), verify_password() (con bcrypt)
    tokens.py       ← create_token(), verify_token(), refresh_token()
    service.py      ← AuthService: orquesta crypto y tokens + DB ops
─────────────────────────────────────────────────────

> El plan está bien. Una corrección: deja las operaciones de DB
  en service.py por ahora (no queremos un repository.py todavía).
  
  Y MUY IMPORTANTE: auth.py debe seguir funcionando igual
  hacia afuera (que importe de los nuevos módulos) para que
  nada se rompa.
  
  Procede con el refactoring.

─────────────────────────────────────────────────────
Ejecutando el plan ajustado...

● Read(src/auth.py)
● Write(src/auth/crypto.py)
● Write(src/auth/tokens.py)
● Write(src/auth/service.py)
● Write(src/auth/__init__.py)
● Write(src/auth.py)  ← ahora solo re-exporta de src/auth/

Verificando que los imports no se rompieron...
● Bash(grep -r "from auth import\|import auth" src/ tests/)
● Bash(python -c "from src.auth import hash_password, verify_token; print('imports OK')")

  imports OK ✅

Ejecutando tests existentes...
● Bash(pytest tests/test_auth.py -v)

  test_hash_password ........... PASSED
  test_verify_token ............. PASSED
  test_token_expired ............ PASSED
  (8 tests, 8 passed) ✅

Refactoring completado:
  - Creados 4 nuevos archivos en src/auth/
  - src/auth.py ahora solo re-exporta (compatibilidad total)
  - Añadidos type hints en todas las funciones
  - MD5 reemplazado por bcrypt en hash_password()
  - Añadido manejo de TokenExpiredError en verify_token()
─────────────────────────────────────────────────────
```

### Caso 2: Investigar y Corregir un Bug de Producción

```
> Tenemos este error en producción desde esta mañana:

  ERROR 2026-03-15 09:34:12
  stripe.error.InvalidRequestError: Invalid integer: amount must be
  a positive integer. Got: 99.99
  Traceback:
    File "src/services/payment_service.py", line 167, in charge
      charge = await stripe.charge.create(amount=amount_cents, ...)

  Ocurre con pagos de amounts con decimales ($9.99, $4.50).
  No ocurre con amounts que son múltiplos de 100 ($10.00, $20.00).

  1. Reproduce el bug en un test
  2. Corrígelo
  3. El test debe pasar después del fix

─────────────────────────────────────────────────────
● Read(src/services/payment_service.py)

Encontré el problema en la línea 164:

  amount_cents = order.total * 100

Si order.total es 9.99 (float de Python), entonces:
  9.99 * 100 = 999.9999999999999  ← no es un entero exacto
  
Stripe requiere un entero exacto. La conversión es incorrecta.
Fix: int(round(order.total * 100))

● Read(tests/test_payment_service.py)
● Write(tests/test_payment_service.py)

Añadido test que reproduce el bug:
  async def test_charge_with_decimal_amount_converts_correctly():
      """Regresión: $9.99 debe convertirse a 999 (int), no 999.999"""
      order = create_test_order(total=Decimal("9.99"))
      await payment_service.charge(order)
      # Verificar que Stripe recibió un entero
      call_args = mock_stripe.charge.create.call_args
      assert isinstance(call_args.kwargs["amount"], int)
      assert call_args.kwargs["amount"] == 999

● Bash(pytest tests/test_payment_service.py::test_charge_with_decimal_amount_converts_correctly -v)

  FAILED: stripe.error.InvalidRequestError: Invalid integer: 999.9999999
  ← El test reproduce el bug correctamente ✅

● Write(src/services/payment_service.py)
  Línea 164: amount_cents = order.total * 100
           → amount_cents = int(round(float(order.total) * 100))

● Bash(pytest tests/test_payment_service.py -v)

  test_charge_success ................................ PASSED
  test_charge_with_decimal_amount_converts_correctly  PASSED
  test_charge_stripe_error .......................... PASSED
  (7 tests, 7 passed) ✅

Bug corregido. La causa: conversión float→int sin redondeo explícito.
─────────────────────────────────────────────────────
```

### Caso 3: Generar Suite Completa de Tests

```
> src/services/inventory_service.py tiene 12 métodos públicos
  y CERO tests. Genera una suite completa.

  Requisitos:
  - Framework: pytest + pytest-asyncio
  - Mockear: InventoryRepository y NotificationService
  - Mínimo 2 casos por método (happy path + caso de error principal)
  - Caso especial obligatorio: test_reduce_stock_insufficient_raises_error
    debe verificar que NO se llama repository.save() cuando falla
  - Destino: tests/unit/services/test_inventory_service.py

─────────────────────────────────────────────────────
● Read(src/services/inventory_service.py)     ← 12 métodos públicos
● Read(src/repositories/inventory_repo.py)    ← interfaz del repo
● Read(src/core/exceptions.py)               ← excepciones disponibles
● Read(tests/unit/services/test_order_service.py) ← ver el estilo del proyecto

Generando tests...
● Write(tests/unit/services/test_inventory_service.py)

Generados 28 tests cubriendo los 12 métodos:
  get_stock:           2 tests
  add_stock:           3 tests  
  reduce_stock:        3 tests (incluye el caso especial)
  transfer_stock:      3 tests
  get_movements:       2 tests
  get_low_stock_items: 2 tests
  adjust_stock:        2 tests
  (más 11 tests para los 5 métodos restantes)

● Bash(pytest tests/unit/services/test_inventory_service.py -v)

  ............................
  28 passed in 1.23s ✅

● Bash(pytest tests/unit/services/test_inventory_service.py --cov=src/services/inventory_service.py)

  Name                               Stmts   Miss   Cover
  src/services/inventory_service.py    143      8     94%

  Líneas sin cobertura: 89-92 (logging de debug, no es lógica de negocio)
─────────────────────────────────────────────────────
```

---

## 11. Integración con el Ecosistema de Desarrollo

### Integración con Git

Claude Code entiende Git nativamente. Estos prompts funcionan sin configuración extra:

```bash
# Analizar el historial de cambios
> Analiza git log --oneline -30. ¿Qué módulos se han modificado más?
  ¿Hay algún patrón que sugiera un área problemática del código?

# Resolver conflictos de merge
> Hay conflictos en estos archivos tras el merge:
  [lista de archivos con conflictos]
  Resuélvelos. Si un cambio es ambiguo, pregúntame cuál mantener.

# Generar mensaje de commit semántico
> Revisa git diff --staged y genera un mensaje de commit
  siguiendo Conventional Commits:
  tipo(scope): descripción breve
  
  cuerpo opcional con más detalle

# Analizar impacto antes de cambiar algo
> Sin modificar nada: si elimino la clase UserValidator,
  ¿qué archivos se romperían? Lista todas las dependencias.
```

### Uso en Modo No Interactivo (Scripts y CI/CD)

El flag `--print` hace que Claude ejecute una tarea y salga, imprimiendo el resultado en stdout:

```bash
# Generar CHANGELOG desde los commits
claude --print "Analiza los commits desde el tag anterior hasta HEAD.
Genera un CHANGELOG.md con secciones: Added, Changed, Fixed, Removed.
Formato: markdown. Solo cambios relevantes para usuarios." > CHANGELOG_DRAFT.md

# Review automático de PR (output en JSON para procesarlo)
REVIEW_JSON=$(claude --print \
  --allowedTools "Read,Bash" \
  "Revisa git diff origin/main...HEAD. Lista problemas en JSON:
   [{\"file\": \"...\", \"line\": N, \"severity\": \"error|warning\",
     \"description\": \"...\"}].
   Si no hay problemas, devuelve []")

# Verificar cobertura y sugerir tests faltantes
claude --print \
  --allowedTools "Read,Bash" \
  "Ejecuta pytest con cobertura y lista los métodos públicos
   con menos del 70% de cobertura, con sugerencias de casos a añadir."
```

### Integración con GitHub Actions

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review en PRs

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write  # Para poder añadir comentarios al PR

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Necesario para ver el diff completo

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Instalar Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Ejecutar revisión con Claude
        run: |
          claude --print \
            --allowedTools "Read,Bash" \
            "Revisa los cambios en este PR (git diff origin/${{ github.base_ref }}...HEAD).
             Produce un reporte markdown con estas secciones (omite secciones vacías):
             ## ❌ Bugs Potenciales
             ## ⚠️ Problemas de Seguridad  
             ## 📝 Tests Faltantes
             ## 💡 Sugerencias de Mejora
             Si no encuentras ningún problema en una categoría, no incluyas esa sección." \
          > review.md
          echo "=== Revisión generada ==="
          cat review.md
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

      - name: Publicar revisión como comentario en el PR
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = fs.readFileSync('review.md', 'utf8');
            
            if (review.trim().length < 20) {
              console.log('Revisión sin problemas, no se comenta');
              return;
            }
            
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.payload.pull_request.number,
              body: `## 🤖 Revisión Automática (Claude Code)\n\n${review}\n\n---\n*Generado automáticamente. Revisar con criterio humano.*`
            });
```

### Git Hook: Verificación Pre-Push

```bash
#!/bin/bash
# .git/hooks/pre-push
# Ejecutar chmod +x .git/hooks/pre-push después de crear este archivo

echo "🤖 Verificando con Claude Code antes del push..."

ISSUES=$(claude --print \
  --allowedTools "Read,Bash" \
  "Analiza git diff origin/HEAD...HEAD (los commits que se van a pushear).
   Lista SOLO los problemas críticos: bugs que claramente fallarán en runtime
   o credenciales/secrets hardcodeados.
   Si no hay ninguno, responde exactamente la palabra: CLEAN")

if [ "$ISSUES" = "CLEAN" ]; then
    echo "✅ Sin problemas críticos. Procediendo con el push."
    exit 0
else
    echo ""
    echo "⚠️  Claude encontró posibles problemas:"
    echo "─────────────────────────────────────"
    echo "$ISSUES"
    echo "─────────────────────────────────────"
    echo ""
    read -p "¿Continuar con el push de todas formas? (s/N): " response
    [[ "$response" =~ ^[Ss]$ ]] && exit 0 || exit 1
fi
```

---

## 12. Buenas Prácticas y Patrones

### El Ciclo de Trabajo Recomendado

```
PRIMERA VEZ EN UN PROYECTO:
  1. cd /mi/proyecto && claude
  2. /init → genera CLAUDE.md
  3. Revisar y mejorar el CLAUDE.md (añadir convenciones, contexto de negocio)
  4. Primera tarea

CADA NUEVA SESIÓN:
  1. cd /mi/proyecto && claude
  2. Claude carga CLAUDE.md automáticamente
  3. Empezar con la tarea directamente

PARA CADA TAREA NUEVA:
  1. Describir la tarea con contexto completo
  2. Si afecta más de 3 archivos: pedir el PLAN primero
  3. Aprobar/ajustar el plan
  4. Claude ejecuta
  5. Revisar cambios en el IDE (git diff)
  6. Si hay problemas: describir y pedir corrección

GESTIÓN DE LA SESIÓN:
  1. /cost regularmente para controlar gastos
  2. /compact cuando el costo supere 80k tokens
  3. /clear al cambiar completamente de tarea
```

### Lo que SIEMPRE Debes Revisar Manualmente

```
🔴 REVISIÓN OBLIGATORIA (alto riesgo):
   - Cambios en autenticación o autorización
   - Código que procesa pagos o datos financieros
   - Migraciones de base de datos (pueden perder datos)
   - Cambios en configuración de seguridad
   - Código que maneja datos personales (GDPR/PII)
   - Cualquier cambio antes de deploy a producción

🟡 REVISIÓN RECOMENDADA (riesgo medio):
   - Refactoring de módulos core del sistema
   - Cambios en el modelo de datos o esquemas
   - Código que toca múltiples módulos a la vez
   - Nuevas dependencias añadidas

🟢 REVISIÓN OPCIONAL (bajo riesgo):
   - Tests unitarios generados por Claude
   - Docstrings y comentarios
   - Formateo y correcciones de linting
   - Scripts de utilidad internos
   - Archivos de configuración de desarrollo
```

### Gestionar el Costo Eficientemente

```
ESTRATEGIA 1: Modelo correcto para cada tarea
  Pregunta simple, exploración → haiku (~$0.01/respuesta)
  Trabajo diario de código    → sonnet (~$0.05-0.15/tarea)
  Arquitectura, análisis deep → opus (~$0.50-2.00/tarea)

ESTRATEGIA 2: Prompts específicos
  Específico = menos iteraciones = menos tokens = menos costo
  "Analiza src/auth.py" usa menos que "analiza el proyecto"

ESTRATEGIA 3: Gestionar el contexto
  Después de 1h de trabajo intenso → /compact
  Al cambiar de tarea → /clear
  Repo muy grande → especifica paths concretos

ESTRATEGIA 4: Subagentes para repos grandes
  Paradójicamente, los subagentes pueden ser más baratos:
  cada uno tiene contexto pequeño y limpio en lugar de
  un contexto enorme compartido que crece sin control.
```

---

## 13. Limitaciones, Seguridad y Privacidad

### Limitaciones Técnicas Reales

| Limitación | Qué significa en la práctica | Workaround |
|------------|------------------------------|------------|
| **Sin memoria entre sesiones** | Cada vez que abres `claude`, empieza desde cero | Mantener CLAUDE.md actualizado |
| **Ventana de contexto finita** | ~200k tokens; sesiones largas degradan la calidad | `/compact` regularmente |
| **Puede "alucinar"** | En código poco conocido, puede inventar APIs inexistentes | Siempre ejecutar y verificar el código generado |
| **Sin acceso a sistemas privados** | No puede conectarse a tu DB, APIs internas sin MCP | Configurar MCP servers |
| **Lento en repos enormes** | Miles de archivos = exploración costosa y lenta | Usar subagentes, ser específico con rutas |
| **No entiende el negocio** | No sabe las reglas específicas de tu dominio | Documentarlas en CLAUDE.md |
| **No toma decisiones de arquitectura** | Puede proponer opciones, pero la decisión es tuya | Usar para explorar, decidir tú |

### El Sistema de Confirmaciones de Seguridad

Claude Code implementa confirmaciones antes de operaciones potencialmente destructivas:

```
Operaciones que SIEMPRE piden confirmación:
  ❗ rm, rmdir (eliminar archivos)
  ❗ comandos con sudo
  ❗ git push, git reset --hard, git rebase
  ❗ operaciones de base de datos destructivas (DROP, DELETE masivo)

Operaciones que NO piden confirmación (consideradas seguras):
  ✓ Read (leer archivos)
  ✓ grep, find (buscar)
  ✓ pytest, jest, etc. (ejecutar tests)
  ✓ Write (escribir archivos nuevos o modificar existentes)
  ✓ git log, git diff, git status (solo lectura de git)
```

El diálogo de confirmación:

```
╭─────────────────────────────────────────────────────╮
│  Claude quiere ejecutar:                            │
│                                                     │
│    rm -rf dist/ build/ .pytest_cache/               │
│                                                     │
│  Esta acción eliminará directorios permanentemente. │
│                                                     │
│  [s] Sí, solo esta vez                             │
│  [n] No, cancela esta acción                       │
│  [S] Sí a este tipo de acción (esta sesión)        │
│  [N] No a este tipo de acción (esta sesión)        │
╰─────────────────────────────────────────────────────╯
```

> ⚠️ El flag `--dangerously-skip-permissions` desactiva **todas** las confirmaciones. Úsalo solo en entornos de CI/CD completamente controlados donde sabes exactamente qué va a ejecutar Claude. Nunca en tu máquina local de desarrollo.

### Privacidad del Código: ¿Qué Sabe Anthropic?

Esta es una pregunta crítica para proyectos con código propietario:

**¿Qué se envía a Anthropic en cada request?**
- El contenido completo de todos los archivos que Claude lee en la sesión
- El historial completo de la conversación
- Los outputs de todos los comandos ejecutados
- El contenido de CLAUDE.md

**¿Qué hace Anthropic con esos datos?**

| Plan | Uso para entrenamiento | Tiempo de retención |
|------|----------------------|---------------------|
| Free | Puede usarse para mejorar modelos | Según política de privacidad |
| Pro | Puede usarse para mejorar modelos | Según política de privacidad |
| Teams | ❌ No se usa para entrenamiento | 30 días |
| Enterprise | ❌ No se usa para entrenamiento | Configurable, puede ser 0 |

**Recomendación:** Para código propietario, con secretos de negocio o bajo obligaciones de confidencialidad, usa el plan **Teams** o **Enterprise**.

---

## 14. Troubleshooting: Errores Comunes y Soluciones

### Error: API Key No Configurada

```
AuthenticationError: API key is required. Set ANTHROPIC_API_KEY environment variable.
```

```bash
# Verificar si está configurada
echo $ANTHROPIC_API_KEY

# Si está vacía, configurarla
export ANTHROPIC_API_KEY="sk-ant-api03-..."

# Para que persista (añadir al .bashrc o .zshrc)
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.bashrc
source ~/.bashrc
```

### Error: Permisos al Instalar con npm

```
EACCES: permission denied, access '/usr/local/lib/node_modules'
```

```bash
# Solución: configurar npm para instalar en carpeta de usuario (sin sudo)
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
npm install -g @anthropic-ai/claude-code
```

### Problema: Claude No Responde / Se Queda Pensando

Si Claude lleva más de 2-3 minutos sin responder:

```bash
# 1. Presiona Ctrl+C para interrumpir

# 2. Verifica conectividad con la API
curl -s -o /dev/null -w "%{http_code}" https://api.anthropic.com/v1/models \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01"
# Debe devolver 200

# 3. Ejecuta el diagnóstico dentro de claude
/doctor

# 4. Si el problema persiste, prueba con un modelo diferente
/model
# Selecciona haiku (más rápido y menos propenso a timeouts)
```

### Problema: Contexto Demasiado Largo

```
Error: This request's context length exceeds the model's maximum context length.
```

```
# Dentro de la sesión de claude:

> /compact     ← Intenta primero comprimir

# Si sigue fallando:
> /clear       ← Limpia todo y empieza de nuevo
              (el trabajo en archivos ya está guardado)
```

### Problema: Claude Modificó Archivos que No Debería

```bash
# Opción 1: Revertir con Git (recomendado)
git diff --name-only    # Ver qué archivos cambió
git checkout -- src/archivo-que-no-debía-cambiar.py  # Revertir uno
git checkout -- .       # Revertir todos los cambios no commiteados

# Opción 2: Prevenir en el futuro
# Añadir en CLAUDE.md:
## Archivos PROTEGIDOS — NO MODIFICAR sin mi aprobación explícita
# - src/core/config.py
# - src/core/security.py
```

### Problema: Tests Generados que No Pasan

```
> Los tests que generaste en tests/test_inventory.py fallan.
  Aquí está el output completo:

  [pegar el output completo de pytest]

  Analiza los errores, corrígelos y ejecuta los tests
  de nuevo hasta que pasen todos.
```

Claude Code puede analizar sus propios errores y corregirlos. No tienes que debuggear manualmente los tests que él generó.

### Problema: Claude "Alucina" una API o Método

Síntoma: Claude escribe código que usa métodos o funciones que no existen en la librería.

```bash
# Verificar si el método realmente existe
python -c "import librería; help(librería.Clase)"

# Pedir a Claude que verifique
> El método que usaste (librería.Clase.método_inventado()) no parece existir.
  Por favor verifica la documentación oficial o el código fuente de
  la librería antes de usar ese método. Aquí está el error:
  
  AttributeError: 'Clase' object has no attribute 'método_inventado'
```

---

## 15. Apéndices y Referencia Rápida

### Cheatsheet Completo

```
╔═══════════════════════════════════════════════════════════════╗
║                CLAUDE CODE — REFERENCIA RÁPIDA                ║
╠═══════════════════════════════════════════════════════════════╣
║  INSTALACIÓN                                                  ║
║    npm install -g @anthropic-ai/claude-code                   ║
║    export ANTHROPIC_API_KEY="sk-ant-..."                      ║
╠═══════════════════════════════════════════════════════════════╣
║  INICIO                                                       ║
║    claude                   Sesión interactiva                ║
║    claude --print "tarea"   Sin interacción (scripts/CI)      ║
║    claude --version         Ver versión instalada             ║
╠═══════════════════════════════════════════════════════════════╣
║  COMANDOS SLASH (dentro de la sesión)                         ║
║    /help              Lista todos los comandos                ║
║    /init              Generar/actualizar CLAUDE.md            ║
║    /clear             Borrar TODO el contexto                 ║
║    /compact           Comprimir contexto (mantiene el hilo)   ║
║    /cost              Ver tokens usados y costo estimado      ║
║    /model             Ver modelo actual o cambiar             ║
║    /review            Code review del git diff actual         ║
║    /pr_comments <N>   Cargar comentarios del PR #N            ║
║    /doctor            Diagnóstico del entorno                 ║
║    /terminal-setup    Configurar atajos de teclado            ║
║    /agent list        Ver subagentes activos                  ║
║    /agent logs <id>   Ver log detallado de un subagente       ║
║    /agent stop <id>   Detener un subagente en curso           ║
╠═══════════════════════════════════════════════════════════════╣
║  FLAGS DEL CLI                                                ║
║    --print              Modo no interactivo (stdout)          ║
║    --model <nombre>     Forzar modelo específico              ║
║    --allowedTools X,Y   Solo permitir estas herramientas      ║
║    --disallowedTools X  Bloquear estas herramientas           ║
║    --dangerously-skip-permissions  ⚠️ Sin confirmaciones      ║
╠═══════════════════════════════════════════════════════════════╣
║  FLUJO RECOMENDADO                                            ║
║    1. cd /proyecto && claude                                  ║
║    2. /init (primera vez en el proyecto)                      ║
║    3. Describe tarea → pide plan → aprueba → ejecuta          ║
║    4. /compact cuando el costo > 80k tokens                   ║
║    5. /cost para controlar gastos                             ║
║    6. /clear al cambiar completamente de tarea                ║
╚═══════════════════════════════════════════════════════════════╝
```

### Fórmulas de Prompt por Tipo de Tarea

```
REFACTORING:
"Refactoriza [archivo] para [objetivo concreto].
 No cambies [restricción].
 Después ejecuta [tests] y verifica que pasan."

GENERAR TESTS:
"Genera tests para [clase] en [destino].
 Framework: [pytest/jest]. 
 Casos: [lista].
 Mockear: [dependencias externas]."

DEBUGGING:
"Error: [mensaje]
 Stack trace: [stack trace]
 Ocurre cuando: [condición]
 1. Reproduce en un test que falle.
 2. Corrígelo.
 3. El test debe pasar."

MIGRACIÓN CONTROLADA:
"Migra [qué] a [dónde/nuevo_framework].
 Hazlo paso a paso, esperando mi 'ok' entre pasos.
 Primero muéstrame el plan completo."

ANÁLISIS SIN CAMBIOS:
"Analiza [qué] y reporta [información].
 NO modifiques ningún archivo. Solo leer y reportar."
```

### Glosario

| Término | Definición completa |
|---------|---------------------|
| **Claude Code** | CLI (interfaz de línea de comandos) de Anthropic que convierte al modelo Claude en un agente de programación autónomo con acceso a herramientas del sistema |
| **Agente** | Sistema de IA que no solo genera texto sino que toma acciones reales: lee archivos, ejecuta comandos, modifica código, en un bucle hasta completar la tarea |
| **CLAUDE.md** | Archivo de instrucciones persistentes leído automáticamente al iniciar una sesión. Es la única "memoria" que persiste entre sesiones |
| **Subagente** | Instancia secundaria de Claude con contexto propio, creada por el agente principal para trabajar en una subtarea específica de forma independiente |
| **Orquestador** | El agente principal que divide una tarea grande en subtareas, crea y coordina subagentes, y luego integra sus resultados |
| **MCP (Model Context Protocol)** | Estándar abierto de Anthropic para conectar modelos de IA con herramientas externas (bases de datos, APIs, servicios) de forma estandarizada |
| **Hook** | Script configurado para ejecutarse automáticamente cuando ocurre un evento en Claude Code (ej: linter después de cada Write) |
| **Contexto** | Toda la información que Claude tiene "en memoria" en una sesión: CLAUDE.md + historial + archivos leídos + outputs de comandos |
| **Token** | Unidad de texto procesada por el modelo. Aproximadamente 4 caracteres en inglés o 3 en español. El costo se calcula por tokens. |
| **ReAct** | "Reason + Act": el patrón de funcionamiento de Claude Code. Razona sobre la situación → decide una acción → la ejecuta → observa el resultado → repite |
| **Tool / Herramienta** | Capacidad de Claude para interactuar con el sistema: `Read`, `Write`, `Bash`, `WebSearch`, `Agent` |
| **/compact** | Comprime el historial de conversación a un resumen, reduciendo tokens sin perder el hilo de la tarea actual |
| **/clear** | Borra completamente el contexto de la sesión. Solo persiste CLAUDE.md |
| **--print** | Flag del CLI para modo no interactivo. Claude ejecuta la tarea, imprime el resultado en stdout y termina. Ideal para scripts y CI/CD |

### Recursos Adicionales

| Recurso | URL | Qué encontrarás |
|---------|-----|-----------------|
| Documentación oficial | https://docs.anthropic.com/claude-code | Referencia completa y actualizada |
| Changelog de Claude Code | https://github.com/anthropics/claude-code/releases | Novedades de cada versión |
| MCP Servers oficiales | https://github.com/modelcontextprotocol/servers | Integraciones listas para usar |
| Especificación MCP | https://modelcontextprotocol.io | Para crear tus propios MCP servers |
| Precios de la API | https://www.anthropic.com/pricing | Precios actualizados por modelo |
| Console Anthropic | https://console.anthropic.com | Gestionar API keys, ver uso |

---

> 📌 **Nota de versión:** Esta guía documenta Claude Code a marzo de 2026. La herramienta evoluciona rápidamente. Para las últimas novedades, consulta https://docs.anthropic.com/claude-code. Cuando algo no funcione como se describe, ejecuta `/doctor` primero para verificar tu entorno.

