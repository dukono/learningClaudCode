# Capítulo 4: El Sistema de Herramientas (Tools)

> **Nivel:** Sin conocimientos previos requeridos  
> **Prerequisito recomendado:** [Cap. 3](./cap-03-arquitectura-componentes.md)  
> **Tiempo de lectura estimado:** 55-70 minutos  
> **Objetivo:** Entender en profundidad cada herramienta disponible en Claude Code — qué hace, cómo funciona internamente, cuándo Claude la usa, y cómo puedes controlar qué herramientas están disponibles en cada situación.

---

## Tabla de Contenidos

- [4.1 Qué es una herramienta y cómo se define](#41-qué-es-una-herramienta-y-cómo-se-define)
- [4.2 El ciclo completo de invocación de una herramienta](#42-el-ciclo-completo-de-invocación-de-una-herramienta)
- [4.3 Herramienta Read: leer el sistema de archivos](#43-herramienta-read-leer-el-sistema-de-archivos)
- [4.4 Herramienta Write: escribir y crear archivos](#44-herramienta-write-escribir-y-crear-archivos)
- [4.5 Herramienta Edit y MultiEdit: modificar sin reescribir](#45-herramienta-edit-y-multiedit-modificar-sin-reescribir)
- [4.6 Herramienta Bash: el poder completo del terminal](#46-herramienta-bash-el-poder-completo-del-terminal)
- [4.7 Herramientas Glob y Grep: exploración eficiente](#47-herramientas-glob-y-grep-exploración-eficiente)
- [4.8 Herramientas WebSearch y WebFetch: acceso a internet](#48-herramientas-websearch-y-webfetch-acceso-a-internet)
- [4.9 Herramienta Agent: crear subagentes paralelos](#49-herramienta-agent-crear-subagentes-paralelos)
- [4.10 Cómo Claude decide qué herramienta usar](#410-cómo-claude-decide-qué-herramienta-usar)
- [4.11 Patrones de uso avanzados con herramientas](#411-patrones-de-uso-avanzados-con-herramientas)
- [4.12 Controlar las herramientas disponibles](#412-controlar-las-herramientas-disponibles)
- [4.13 Resumen y glosario del capítulo](#413-resumen-y-glosario-del-capítulo)

---

## 4.1 Qué es una herramienta y cómo se define

### La definición técnica

En el contexto de los LLMs, una **herramienta** (tool) es una función que el modelo puede invocar durante su proceso de razonamiento. El modelo no ejecuta la función directamente — es un modelo de lenguaje que solo genera texto — pero puede **solicitar** que se ejecute especificando el nombre y los parámetros en su respuesta.

Esta es la distinción fundamental: el LLM en los servidores de Anthropic **describe** qué herramienta quiere usar y con qué parámetros, pero la ejecución real ocurre en **tu computadora** a través del Tool Executor.

La analogía más clara: imagina que eres un arquitecto (el LLM) que trabaja de forma remota y tienes un asistente de construcción local (el Tool Executor). Tú no puedes tocar físicamente los materiales desde tu oficina, pero puedes decirle: *"Lee los planos del edificio en el archivo 'planos_norte.pdf' y dime el grosor de las paredes"*. El asistente lo hace y te da la información. Luego tú decides qué instrucciones dar.

### Por qué este diseño es tan poderoso

Este diseño de "el LLM decide, el agente ejecuta" es extremadamente flexible porque:

1. **El LLM siempre puede ver el resultado:** Todo lo que la herramienta retorna se añade al contexto, disponible para el siguiente ciclo de razonamiento.

2. **Las herramientas pueden combinarse infinitamente:** Puedes buscar un patrón con Grep, leer los archivos que encuentres con Read, modificarlos con Edit, y verificar con Bash — todo en una sola tarea.

3. **Las herramientas se pueden extender:** Mediante MCP puedes añadir herramientas personalizadas que Claude usa igual que las nativas (Capítulo 7).

4. **Las herramientas se pueden restringir:** Puedes dar a Claude solo las herramientas de solo lectura para auditorías, o solo herramientas de análisis para entornos de producción.

### El JSON Schema: el contrato formal de cada herramienta

Las herramientas se definen usando **JSON Schema**, un estándar para describir la estructura de objetos JSON. Cada herramienta tiene exactamente tres elementos:

```json
{
  "name": "Read",

  "description": "Lee el contenido completo de un archivo de texto del
    sistema de archivos local. Úsala cuando necesites ver el contenido
    de un archivo antes de modificarlo, para entender el código existente,
    o para leer archivos de configuración. Solo funciona con archivos de
    texto plano (código, JSON, YAML, Markdown, etc.) — no puede leer
    archivos binarios como imágenes, PDFs o ejecutables.",

  "input_schema": {
    "type": "object",
    "properties": {
      "path": {
        "type": "string",
        "description": "Ruta al archivo. Puede ser relativa al workspace o absoluta."
      },
      "start_line": {
        "type": "integer",
        "description": "Número de línea desde el que empezar a leer (1-indexado, opcional)."
      },
      "end_line": {
        "type": "integer",
        "description": "Número de línea hasta el que leer, inclusivo (opcional)."
      }
    },
    "required": ["path"]
  }
}
```

### Por qué la descripción importa más que el código

La descripción es lo **único** que el LLM tiene para decidir si debe usar una herramienta y cuándo. El LLM no puede ver la implementación interna del Tool Executor. Solo puede leer la descripción en lenguaje natural.

Esta es la razón por la que Claude a veces elige una herramienta "subóptima": si la descripción de Grep dice "busca texto en archivos" y la de Bash no menciona grep explícitamente, Claude puede preferir Grep aunque `Bash("grep ...")` haría lo mismo.

Esto también es importante al crear herramientas MCP personalizadas: una descripción clara y precisa es la diferencia entre que Claude use tu herramienta correctamente o la ignore.

### La lista completa de herramientas nativas

Claude Code viene con las siguientes herramientas predefinidas:

| Herramienta | Propósito | ¿Requiere confirmación? |
|-------------|-----------|------------------------|
| **Read** | Leer archivos de texto | No |
| **Write** | Crear o sobreescribir archivos | No (pero aconsejable revisar) |
| **Edit** | Reemplazar texto en archivo existente | No |
| **MultiEdit** | Múltiples reemplazos en un archivo | No |
| **Bash** | Ejecutar comandos de terminal | Solo si son destructivos |
| **Glob** | Buscar archivos por patrón | No |
| **Grep** | Buscar texto en archivos | No |
| **WebSearch** | Buscar en internet | No |
| **WebFetch** | Descargar contenido de URL | No |
| **Agent** | Crear subagente independiente | No |
| **TodoRead** | Leer lista de tareas del proyecto | No |
| **TodoWrite** | Actualizar lista de tareas | No |

---

## 4.2 El ciclo completo de invocación de una herramienta

### El flujo de extremo a extremo

Cuando Claude decide usar una herramienta, se desencadena una secuencia precisa de eventos. Entenderla te permite depurar problemas, optimizar costos y diseñar mejores prompts.

```
CICLO DE INVOCACIÓN DE HERRAMIENTA — TRAZA DETALLADA
═══════════════════════════════════════════════════════════════

PASO 1: El LLM genera la solicitud de herramienta
─────────────────────────────────────────────────
La API de Anthropic retorna:
{
  "id": "msg_01XYZ...",
  "stop_reason": "tool_use",        ← señal de que quiere una herramienta
  "content": [
    {
      "type": "text",
      "text": "Voy a leer el archivo para entender la implementación."
    },
    {
      "type": "tool_use",           ← la solicitud de herramienta
      "id": "toolu_01ABC",          ← ID único para este uso de herramienta
      "name": "Read",               ← nombre de la herramienta
      "input": {                    ← parámetros según el JSON Schema
        "path": "src/auth.py"
      }
    }
  ]
}

PASO 2: Claude Code reconoce el stop_reason "tool_use"
──────────────────────────────────────────────────────
El Agente Principal:
  • Añade la respuesta del asistente al historial
  • Extrae todas las solicitudes tool_use de la respuesta
  • Pasa cada solicitud al Tool Executor

PASO 3: El Tool Executor ejecuta la herramienta
────────────────────────────────────────────────
Para Read("src/auth.py"):
  1. Valida que el path existe y es un archivo
  2. Verifica que el usuario tiene permisos de lectura
  3. Abre el archivo y lee su contenido
  4. Retorna el contenido como string

Si hay error (archivo no existe, permisos, etc.):
  • is_error = true
  • El contenido es el mensaje de error
  • Claude ve el error y puede decidir qué hacer a continuación

PASO 4: El resultado se añade al historial como tool_result
────────────────────────────────────────────────────────────
Context Manager construye:
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01ABC",   ← vinculado al tool_use original
      "content": "import hashlib\n\ndef hash_password(password: str) -> str:\n    return hashlib.md5(password.encode()).hexdigest()\n\ndef verify_password(password: str, hashed: str) -> bool:\n    return hash_password(password) == hashed\n",
      "is_error": false
    }
  ]
}

PASO 5: El historial actualizado se envía a la API
───────────────────────────────────────────────────
El payload ahora incluye:
  • El mensaje original del usuario
  • La respuesta del asistente con tool_use
  • El tool_result con el contenido del archivo
  
El LLM recibe toda esta información y continúa el razonamiento.

PASO 6: El LLM decide si necesita más herramientas o si terminó
─────────────────────────────────────────────────────────────────
Con el contenido de auth.py en contexto, el LLM puede:
  • Invocar otra herramienta (Read otro archivo, Bash para tests, etc.)
  • Generar la respuesta final (stop_reason: "end_turn")
```

### Los IDs de herramientas: vinculando solicitudes con resultados

Cada `tool_use` genera un ID único (ej. `toolu_01ABC`). El `tool_result` correspondiente debe incluir el mismo ID en `tool_use_id`. Esto permite que el LLM vincule cada resultado con la herramienta que lo generó, especialmente importante cuando se usan varias herramientas en paralelo.

### Múltiples herramientas en paralelo

El LLM puede solicitar varias herramientas en la misma respuesta:

```json
{
  "stop_reason": "tool_use",
  "content": [
    { "type": "tool_use", "id": "toolu_01", "name": "Read", "input": {"path": "src/auth.py"} },
    { "type": "tool_use", "id": "toolu_02", "name": "Read", "input": {"path": "tests/test_auth.py"} },
    { "type": "tool_use", "id": "toolu_03", "name": "Read", "input": {"path": "requirements.txt"} }
  ]
}
```

En este caso, el Tool Executor ejecuta las tres lecturas y las retorna juntas como tres `tool_result` en el mismo mensaje `user`. Esto **no es paralelismo real** (los archivos se leen secuencialmente), pero es más eficiente que tres iteraciones separadas porque reduce el número de viajes a la API.

---

## 4.3 Herramienta Read: leer el sistema de archivos

### Qué hace y por qué es la más usada

`Read` lee el contenido completo de un archivo de texto y lo retorna como string. Es, con diferencia, la herramienta más frecuentemente invocada en una sesión típica de Claude Code.

Esto es así porque la filosofía de Claude Code es **"ver antes de tocar"**: antes de modificar cualquier archivo, Claude lo lee. Esto evita sobreescribir contenido sin haberlo visto, y proporciona el contexto necesario para hacer cambios correctos.

### Cuándo Claude la usa

```
USO PRINCIPAL — Antes de modificar:
  1. Leer auth.py antes de editar las funciones
  2. Leer tests/test_auth.py antes de añadir tests
  3. Leer requirements.txt antes de añadir dependencias

USO SECUNDARIO — Para entender el contexto:
  4. Leer README.md para entender el proyecto
  5. Leer CLAUDE.md si necesita recordar instrucciones
  6. Leer .env.example para entender las variables de entorno
  7. Leer docker-compose.yml para entender la infraestructura

USO TERCIARIO — Para verificar:
  8. Releer un archivo después de escribirlo para confirmar
  9. Leer los logs después de ejecutar tests
```

### Parámetros completos con ejemplos

```json
// Leer archivo completo (más común)
{ "path": "src/auth.py" }

// Leer solo las primeras 30 líneas (para contexto rápido)
{ "path": "src/auth.py", "end_line": 30 }

// Leer una función específica (entre líneas conocidas)
{ "path": "src/auth.py", "start_line": 45, "end_line": 80 }

// Leer desde el final (para ver el final del archivo)
{ "path": "src/auth.py", "start_line": 150 }
```

### El impacto en el contexto: la trampa de los archivos grandes

Cada archivo leído con Read añade su contenido completo al historial de la sesión. Esos tokens permanecen en el contexto para el resto de la sesión:

```
IMPACTO EN EL CONTEXTO
═══════════════════════════════════════════════════════════════

Archivo leído              Tokens añadidos    % de ventana 200k
─────────────────────────────────────────────────────────────
README.md (2 KB)          ~500 tokens        0.25%
requirements.txt (50 líneas) ~300 tokens    0.15%
auth.py (100 líneas)      ~1,200 tokens      0.6%
models.py (500 líneas)    ~6,000 tokens      3%
una_clase_grande.java (2000 líneas) ~24,000  12%
un_repositorio_completo (10k líneas) ~120,000 60%
```

Leer archivos grandes innecesariamente puede agotar el contexto rápidamente. La buena práctica es **leer solo lo necesario**:

```
EN LUGAR DE: Read("src/models/big_module.py")  (2000 líneas)

MEJOR:
  1. Grep("class UserModel", "src/models/big_module.py")
     → Resultado: "UserModel está en la línea 234"
  2. Read("src/models/big_module.py", start_line=234, end_line=300)
     → Solo la clase que necesitas
```

### Archivos que Read puede leer vs. los que no puede

```
PUEDE LEER (texto plano):             NO PUEDE LEER (binario):
  ✓ .py, .js, .ts, .go, .rs            ✗ .pdf
  ✓ .json, .yaml, .toml                ✗ .jpg, .png, .gif
  ✓ .md, .txt, .rst                    ✗ .docx, .xlsx
  ✓ .html, .css, .xml                  ✗ .zip, .tar
  ✓ .sh, .bash, .zsh                   ✗ ejecutables compilados
  ✓ .sql, .graphql                     ✗ archivos de base de datos SQLite
  ✓ .env, .gitignore                   ✗ .pyc (Python bytecode)
  ✓ Makefile, Dockerfile               ✗ .class (Java bytecode)
```

Para leer PDFs o documentos Word, hay que convertirlos primero:
```bash
# Convertir PDF a texto
pdf2txt documento.pdf > documento_texto.txt
# Luego Claude puede leer documento_texto.txt
```

---

## 4.4 Herramienta Write: escribir y crear archivos

### El comportamiento crítico: sobreescritura total

El punto más importante de `Write` es que **sobreescribe TODO el contenido del archivo**. No es un append, no es un merge — es una reescritura completa.

```
DEMOSTRACIÓN DEL COMPORTAMIENTO DE WRITE
═══════════════════════════════════════════════════════════════

ESTADO INICIAL DE src/utils.py:
  line 1:  def suma(a, b): return a + b
  line 2:  def resta(a, b): return a - b
  line 3:  def multiplica(a, b): return a * b
  line 4:  def divide(a, b): return a / b if b != 0 else None

Claude invoca:
  Write("src/utils.py", "def suma(a, b):\n    return a + b\n")

ESTADO FINAL DE src/utils.py:
  line 1:  def suma(a, b):
  line 2:      return a + b
  
¡Las funciones resta(), multiplica(), y divide() han desaparecido!
Si Claude no leyó el archivo antes, ha perdido tres funciones.
```

**Por eso Claude siempre lee antes de escribir.** El flujo correcto:

```
FLUJO CORRECTO PARA MODIFICAR UN ARCHIVO CON WRITE:
═══════════════════════════════════════════════════════════════

1. Read("src/utils.py")
   → Claude ve todo el contenido actual

2. Claude construye mentalmente el nuevo contenido completo:
   def suma(a, b):
       return a + b
   def resta(a, b):
       return a - b
   def multiplica(a, b):
       return a * b
   def divide(a, b):
       if b == 0:
           raise ValueError("No se puede dividir por cero")
       return a / b

3. Write("src/utils.py", <nuevo_contenido_completo>)
   → El archivo se sobreescribe con el contenido correcto que incluye TODO

La clave: Claude nunca pierde las funciones existentes porque las vio en el paso 1.
```

### Creación automática de directorios

`Write` crea automáticamente los directorios intermedios si no existen:

```
Write("src/modules/auth/tokens/jwt_handler.py", contenido)

Si src/modules/auth/tokens/ no existe:
  → Claude Code crea src/, luego src/modules/, luego src/modules/auth/,
    luego src/modules/auth/tokens/, y finalmente escribe el archivo.
  → El comportamiento es equivalente a: mkdir -p src/modules/auth/tokens/
```

Esto es útil cuando Claude está creando un módulo nuevo desde cero con varios archivos en una estructura de directorios.

### Cuándo usar Write vs. Edit

**Usa Write cuando:**
- Creas un archivo nuevo (ninguna alternativa).
- El archivo es pequeño (< 50 líneas) y los cambios son extensos.
- Estás rehaciendo el archivo completamente (refactoring total).
- Los cambios afectan a más del 60-70% del contenido.

**Usa Edit cuando:**
- El archivo es grande (> 100 líneas) y los cambios son localizados.
- Solo cambias 1-5 fragmentos específicos.
- Quieres minimizar el riesgo de perder contenido no relacionado.

---

## 4.5 Herramienta Edit y MultiEdit: modificar sin reescribir

### El problema que resuelve Edit

Imagina un archivo Python de 800 líneas. Quieres cambiar solo una función de 20 líneas. Con Write:
- Claude tiene que incluir las 800 líneas completas en el payload.
- ~9,600 tokens adicionales en el contexto.
- Si Claude comete cualquier error al reconstruir las 780 líneas que no cambia, las pierde.

Edit resuelve esto con un mecanismo de **buscar-y-reemplazar preciso**:

### Cómo funciona Edit

```json
{
  "name": "Edit",
  "input": {
    "path": "src/auth.py",
    "old_string": "def hash_password(password: str) -> str:\n    return hashlib.md5(password.encode()).hexdigest()",
    "new_string": "def hash_password(password: str) -> str:\n    \"\"\"Genera un hash seguro de la contraseña usando bcrypt.\"\"\"\n    salt = bcrypt.gensalt()\n    return bcrypt.hashpw(password.encode('utf-8'), salt).decode('utf-8')"
  }
}
```

El Tool Executor:
1. Abre el archivo `src/auth.py`.
2. Busca la cadena exacta en `old_string`.
3. Si la encuentra exactamente (incluyendo espacios y saltos de línea): la reemplaza con `new_string`.
4. Si no la encuentra: retorna error (Claude puede ajustar).
5. Guarda el archivo.

**Solo las líneas que cambian se incluyen en el payload** — eficiencia máxima.

### La precisión es clave: old_string debe ser único

El `old_string` debe identificar de forma única el fragmento a cambiar. Si el mismo texto aparece en múltiples lugares del archivo, hay ambigüedad:

```python
# PROBLEMA: Este texto aparece 3 veces en el archivo
old_string = "return None"

# MEJOR: Incluir contexto suficiente para que sea único
old_string = "def get_user_by_email(email):\n    if not email:\n        return None"
```

La regla práctica: incluye al menos 3 líneas de contexto antes y después del cambio real para asegurar la unicidad.

### MultiEdit: múltiples cambios atómicos en un archivo

Cuando necesitas hacer varios cambios en el mismo archivo, `MultiEdit` es más eficiente que varias llamadas a `Edit`:

```json
{
  "name": "MultiEdit",
  "input": {
    "path": "src/auth.py",
    "edits": [
      {
        "old_string": "import hashlib",
        "new_string": "import bcrypt"
      },
      {
        "old_string": "def hash_password(password: str) -> str:\n    return hashlib.md5(password.encode()).hexdigest()",
        "new_string": "def hash_password(password: str) -> str:\n    \"\"\"Hash seguro con bcrypt.\"\"\"\n    salt = bcrypt.gensalt()\n    return bcrypt.hashpw(password.encode(), salt).decode()"
      },
      {
        "old_string": "def verify_password(password: str, hashed: str) -> bool:\n    return hash_password(password) == hashed",
        "new_string": "def verify_password(password: str, hashed: str) -> bool:\n    \"\"\"Verifica contraseña contra hash bcrypt.\"\"\"\n    return bcrypt.checkpw(password.encode(), hashed.encode())"
      }
    ]
  }
}
```

Las ediciones se aplican de forma **atómica** (todas se aplican o ninguna, si hay error en alguna). Las ediciones se aplican en orden, de arriba abajo.

### La guía de decisión completa: Read vs Write vs Edit

```
ÁRBOL DE DECISIÓN PARA MODIFICAR UN ARCHIVO
═══════════════════════════════════════════════════════════════

¿Existe el archivo?
│
├── NO → Write (crear nuevo)
│
└── SÍ → ¿Cuánto del archivo cambia?
          │
          ├── > 60% del archivo → Write (más claro)
          │
          ├── 10-60% del archivo → depende del tamaño:
          │    ├── Archivo < 50 líneas → Write (simple)
          │    └── Archivo > 50 líneas → Edit/MultiEdit
          │
          └── < 10% del archivo → Edit/MultiEdit (más eficiente)
               │
               └── ¿Cuántos cambios?
                    ├── 1 cambio → Edit
                    └── 2+ cambios → MultiEdit
```

---

## 4.6 Herramienta Bash: el poder completo del terminal

### La herramienta más versátil

`Bash` ejecuta un comando de terminal y retorna stdout, stderr y exit code. Su potencia viene de la versatilidad de la línea de comandos: cualquier cosa que puedas hacer en una terminal, Claude puede hacerlo con Bash.

Esta versatilidad la convierte en la herramienta más poderosa y la que requiere más comprensión para usarla correctamente.

### Anatomía completa del resultado de Bash

```
Claude invoca: Bash("pytest tests/ -v 2>&1")

Tool Executor ejecuta el proceso y captura:

RESULTADO:
{
  "stdout": "========================= test session starts ==========================\n...\n5 passed in 0.34s\n",
  "stderr": "",
  "exit_code": 0,
  "timed_out": false
}

Claude ve este resultado completo y puede:
  - Analizar si los tests pasaron (exit_code == 0)
  - Ver cuáles tests fallaron (en stdout/stderr)
  - Decidir qué hacer a continuación
```

### Casos de uso por categoría

**Testing y verificación:**
```bash
pytest tests/ -v                          # Tests Python
pytest tests/ -v --cov=src --cov-report=term-missing  # Con cobertura
npm test                                  # Tests JavaScript
cargo test                                # Tests Rust
go test ./...                             # Tests Go
mvn test                                  # Tests Maven
```

**Gestión de dependencias:**
```bash
pip install bcrypt==4.1.2                 # Instalar paquete Python
pip install -r requirements.txt           # Instalar todas las deps
npm install lodash                        # Instalar paquete Node
npm ci                                    # Instalar limpio (CI)
cargo add tokio                           # Añadir dep Rust
```

**Búsqueda y análisis de código:**
```bash
grep -rn "hash_password" src/            # Buscar texto en archivos
grep -rn "TODO\|FIXME\|HACK" . --include="*.py"  # Buscar comentarios
find . -name "*.test.ts" -newer src/auth.ts  # Archivos test más nuevos
git log --oneline -20                    # Últimos 20 commits
git diff HEAD~1 --stat                   # Qué cambió en el último commit
git blame src/auth.py                    # Quién cambió cada línea
```

**Compilación y construcción:**
```bash
npm run build                            # Build JavaScript/TypeScript
tsc --noEmit                             # Solo type-checking TypeScript
cargo build --release                   # Build optimizado Rust
go build ./...                           # Build completo Go
python -m compileall src/               # Compilar Python a bytecode
```

**Herramientas de calidad:**
```bash
black src/ --check                       # Verificar formato Python
black src/                               # Aplicar formato Python
flake8 src/ --max-line-length=100        # Lint Python
eslint src/ --ext .ts,.tsx              # Lint TypeScript
prettier --check "src/**/*.ts"          # Verificar formato TS
rustfmt --check src/main.rs             # Verificar formato Rust
```

**Git y control de versiones:**
```bash
git status                               # Estado del repo
git add -p src/auth.py                  # Añadir cambios interactivamente
git commit -m "feat: migrate MD5 to bcrypt"  # Commit
git push origin main                    # Push (requiere confirmación)
git stash                               # Guardar cambios temporalmente
```

**Docker y entornos:**
```bash
docker compose up -d                    # Levantar servicios
docker compose ps                       # Ver estado de servicios
docker exec -it app_1 bash              # Entrar en contenedor
kubectl get pods                        # Ver pods Kubernetes
```

### El exit code: señal de éxito o fallo

El `exit_code` es una señal crítica que Claude usa para tomar decisiones:

```
exit_code = 0   → Éxito. El comando se completó sin errores.
exit_code = 1   → Error genérico. Algo salió mal.
exit_code = 2   → Uso incorrecto (argumentos erróneos).
exit_code = 127 → Comando no encontrado.
exit_code = 130 → Interrumpido por el usuario (Ctrl+C).

EJEMPLO DE DECISIÓN BASADA EN EXIT CODE:
  Claude ejecuta: pytest tests/
  
  Si exit_code = 0:
    → "Todos los tests pasan. La migración está completa."
  
  Si exit_code = 1:
    → Analiza stdout/stderr para ver qué tests fallaron
    → Identifica la causa del fallo
    → Hace los cambios necesarios y vuelve a ejecutar
```

### La no-persistencia del shell: detalle técnico importante

**Cada llamada a Bash es un proceso shell completamente nuevo**. El estado (variables de entorno, directorio actual, aliases) no persiste entre llamadas:

```bash
# PROBLEMA — Las llamadas son procesos separados:

# Llamada 1:
cd /opt/proyecto && source .venv/bin/activate
# → exit_code: 0, pero estos cambios se pierden al salir del proceso

# Llamada 2:
python -c "import bcrypt; print('OK')"
# → ERROR: bcrypt no instalado (el venv no está activado en este proceso)

# SOLUCIÓN — Encadenar en una llamada:
# Llamada única:
source .venv/bin/activate && python -c "import bcrypt; print('OK')"
# → Funciona porque todo está en el mismo proceso
```

Claude generalmente maneja esto correctamente, pero si ves errores de "módulo no encontrado" o "comando no encontrado" que no tienen sentido, esta es la causa probable.

### El timeout de Bash

Los comandos tienen un timeout configurable. Por defecto es varios minutos. Para comandos que sabes que tardan:

```bash
# Comandos potencialmente lentos
timeout 300 npm run build    # Máximo 5 minutos para el build
timeout 120 pytest tests/ -v # Máximo 2 minutos para los tests
```

Si el comando excede el timeout, el Tool Executor lo mata y retorna `timed_out: true`. Claude puede reintentar o reportar el timeout al usuario.

---

## 4.7 Herramientas Glob y Grep: exploración eficiente

### Por qué existen si Bash puede hacer lo mismo

Podrías hacer todo lo que hacen Glob y Grep con Bash:
- `find . -name "*.py"` en vez de Glob
- `grep -rn "pattern"` en vez de Grep

Entonces, ¿por qué existen estas herramientas separadas?

**Razón 1: Resultados estructurados.** Glob y Grep retornan resultados en un formato JSON estructurado, fácil de procesar. Bash retorna texto plano que Claude tiene que parsear.

**Razón 2: Abstracción de plataforma.** La sintaxis de `find` y `grep` varía entre Linux, macOS y Windows. Glob y Grep son consistentes en todas las plataformas.

**Razón 3: Eficiencia de descripción.** Las descripciones de las herramientas orientan mejor al LLM hacia su uso cuando es apropiado.

### Herramienta Glob en detalle

`Glob` encuentra archivos cuya ruta coincida con un patrón glob. Los patrones glob son la sintaxis que también usan los `.gitignore` y los shells Unix.

**Patrones más útiles:**

```
PATRONES BÁSICOS:
  *.py          → todos los .py en el directorio actual (no subdirectorios)
  **/*.py       → todos los .py en cualquier profundidad
  src/*.py      → .py en src/ (no en subdirectorios de src/)
  src/**/*.py   → .py en src/ y cualquier subdirectorio
  
PATRONES CON NOMBRE:
  test_*.py     → archivos que empiezan por test_
  *_test.py     → archivos que terminan en _test.py
  *auth*        → archivos que contienen "auth" en el nombre
  
PATRONES POR EXTENSIÓN:
  **/*.{py,js}  → .py o .js en cualquier lugar
  **/*.test.*   → cualquier archivo de test (test.py, test.ts, etc.)
  
EXCLUSIONES:
  !**/node_modules/**   → excluir node_modules
  !**/__pycache__/**    → excluir caché Python
  !**/.git/**           → excluir directorio git
```

**Ejemplo de uso real:**

```json
{
  "name": "Glob",
  "input": {
    "pattern": "**/*.py",
    "exclude": [
      "**/node_modules/**",
      "**/__pycache__/**",
      "**/venv/**",
      "**/.venv/**"
    ]
  }
}
```

Resultado:
```
[
  "src/__init__.py",
  "src/auth.py",
  "src/models/user.py",
  "src/models/product.py",
  "tests/__init__.py",
  "tests/test_auth.py",
  "tests/test_models.py"
]
```

### Herramienta Grep en detalle

`Grep` busca un patrón de texto en uno o más archivos y retorna las líneas que coinciden con contexto.

**Parámetros:**

```json
{
  "name": "Grep",
  "input": {
    "pattern": "hash_password",        // cadena o regex
    "paths": ["src/", "tests/"],       // dónde buscar
    "file_pattern": "*.py",            // filtro por extensión (opcional)
    "context_lines": 3,                // líneas de contexto antes/después
    "case_sensitive": true             // distinguir mayúsculas (por defecto true)
  }
}
```

**Resultado estructurado:**

```json
[
  {
    "file": "src/auth.py",
    "line_number": 3,
    "line": "def hash_password(password: str) -> str:",
    "context_before": ["import hashlib", ""],
    "context_after": ["    \"\"\"Hash the password using MD5.\"\"\"", "    return hashlib.md5(password.encode()).hexdigest()"]
  },
  {
    "file": "src/auth.py",
    "line_number": 11,
    "line": "    return hash_password(password) == hashed",
    "context_before": ["def verify_password(password: str, hashed: str) -> bool:", ""],
    "context_after": ["", "def create_user(username: str, password: str) -> dict:"]
  },
  {
    "file": "tests/test_auth.py",
    "line_number": 7,
    "line": "    result = hash_password('test123')",
    "context_before": ["def test_hash_password():", ""],
    "context_after": ["    assert len(result) == 32"]
  }
]
```

**Casos de uso prácticos:**

```
ANTES DE REFACTORIZAR UNA FUNCIÓN:
  Grep("hash_password", ["src/", "tests/"])
  → "Veo que hash_password se usa en 3 lugares: auth.py líneas 3 y 11,
     test_auth.py línea 7. Necesito actualizar los tres."

BUSCAR TODOS LOS TODOS DEL PROYECTO:
  Grep("TODO|FIXME|HACK|XXX", ["."], file_pattern="*.py")
  → Lista completa de deuda técnica pendiente

VERIFICAR QUE UN IMPORT SE USA:
  Grep("from src.auth import", ["src/", "tests/"])
  → "La función hash_password se importa en api.py y test_auth.py"

ENCONTRAR DÓNDE SE USA UNA CLASE:
  Grep("UserRepository(", ["src/"])
  → "UserRepository se instancia en services/user_service.py y api/users.py"
```

---

## 4.8 Herramientas WebSearch y WebFetch: acceso a internet

### Por qué necesita Claude acceder a internet

El modelo Claude fue entrenado con datos hasta cierta fecha (la "fecha de corte"). Después de esa fecha, no sabe qué ha cambiado:
- Una librería puede haber lanzado una versión nueva con una API diferente.
- Una práctica que antes era estándar puede haberse deprecado.
- Un servicio puede haber cambiado su endpoint o su formato de autenticación.

WebSearch y WebFetch resuelven este problema dando a Claude acceso a información actualizada en tiempo real.

### WebSearch: búsqueda en internet

`WebSearch` realiza una búsqueda web y retorna una lista de resultados con título, URL, y fragmento:

```json
{
  "name": "WebSearch",
  "input": {
    "query": "bcrypt python migration MD5 security best practices 2025",
    "num_results": 5
  }
}
```

Resultado:
```json
[
  {
    "title": "Migrating from MD5 to bcrypt for password hashing in Python",
    "url": "https://docs.example.com/security/password-migration",
    "snippet": "...when migrating from MD5 to bcrypt, you need to handle existing hashed passwords..."
  },
  {
    "title": "bcrypt 4.1.x changelog - PyPI",
    "url": "https://pypi.org/project/bcrypt/",
    "snippet": "bcrypt 4.1.3 released 2025-01-15. Added support for Python 3.13..."
  }
]
```

**Cuándo Claude la usa:**
- Verificar la última versión de un paquete antes de añadirlo a requirements.txt.
- Buscar soluciones a mensajes de error muy específicos.
- Encontrar si una API ha cambiado su autenticación recientemente.
- Buscar documentación de una herramienta poco conocida.

### WebFetch: leer el contenido de una URL específica

`WebFetch` descarga el contenido completo de una URL:

```json
{
  "name": "WebFetch",
  "input": {
    "url": "https://pypi.org/project/bcrypt/",
    "extract_text": true
  }
}
```

Con `extract_text: true`, se extrae el texto legible de la página HTML, descartando scripts y estilos.

**Cuándo Claude la usa:**
- Después de WebSearch para leer en detalle una página de resultados.
- Para leer directamente la documentación oficial de una librería.
- Para obtener el contenido de un issue de GitHub o una entrada de changelog.

### Limitaciones críticas de WebFetch

**No ejecuta JavaScript:** Las páginas que cargan contenido dinámicamente (SPAs, documentación generada por React, etc.) pueden aparecer vacías o incompletas porque WebFetch solo obtiene el HTML inicial, sin ejecutar scripts.

```
Páginas que funcionan bien con WebFetch:
  ✓ pypi.org (HTML estático)
  ✓ docs.python.org (HTML estático)
  ✓ GitHub (HTML estático con server-side rendering)
  ✓ StackOverflow (HTML estático)

Páginas que pueden dar resultados vacíos:
  ✗ Documentación de muchas herramientas modernas (Vite, React, etc.)
    → Generadas con frameworks JavaScript que requieren ejecución
```

**El contenido cuenta como tokens:** Una página web puede tener mucho texto. WebFetch puede añadir 5,000-20,000 tokens al contexto por una sola llamada. Úsalo con moderación en sesiones largas.

---

## 4.9 Herramienta Agent: crear subagentes paralelos

### Qué es un subagente y por qué es diferente

La herramienta `Agent` es cualitativamente diferente de todas las demás. En lugar de interactuar con el sistema operativo, **crea una nueva instancia completa del agente Claude Code** que trabaja de forma completamente independiente.

```
DIFERENCIA FUNDAMENTAL
═══════════════════════════════════════════════════════════════

HERRAMIENTA READ:
  • Claude llama a Read("archivo.py")
  • El Tool Executor lee el archivo
  • El resultado vuelve al contexto de Claude
  • Todo ocurre en el mismo agente, el mismo contexto
  
HERRAMIENTA AGENT:
  • Claude llama a Agent(prompt="Documenta los archivos en src/")
  • Se crea UN NUEVO PROCESO DE CLAUDE CODE completamente independiente
  • Este subagente tiene su PROPIO contexto (empieza desde cero)
  • El subagente tiene sus PROPIAS herramientas y ejecuta su propio ciclo ReAct
  • Cuando el subagente termina, retorna su resultado al agente principal
  • El resultado se añade al contexto del agente principal
```

### Por qué el contexto independiente es la ventaja clave

Un subagente con contexto independiente tiene dos ventajas importantes:

**1. El contexto del agente principal no se llena:** Si el agente principal está documentando un proyecto con 50 archivos, en lugar de leer los 50 archivos (llenando su contexto), puede crear 10 subagentes, cada uno documentando 5 archivos con su propio contexto fresco.

**2. Calidad consistente:** Un subagente que empieza con contexto fresco tiene toda su capacidad cognitiva disponible para su subtarea. Un agente con 180,000 tokens de contexto puede ser menos preciso en su razonamiento.

### Invocación completa con todos los parámetros

```json
{
  "name": "Agent",
  "input": {
    "task": "Documenta todos los archivos en src/auth/ con docstrings estilo Google para cada función pública. Incluye: descripción, Args con tipos y descripción, Returns con tipo y descripción, y un ejemplo de uso. NO cambies la lógica, solo añade documentación.",
    
    "context": "El proyecto usa Python 3.12. Ver src/models/user.py como referencia del estilo de docstrings que queremos.",
    
    "allowed_tools": ["Read", "Write", "Edit", "Glob", "Bash"],
    
    "model": "claude-haiku-4-5"
  }
}
```

**Parámetros importantes:**

- **`task`:** La descripción completa de la subtarea. Cuanto más específica, mejor el resultado.
- **`context`:** Contexto adicional que el subagente necesita (no tiene acceso al historial del agente principal).
- **`allowed_tools`:** Qué herramientas puede usar el subagente. Es buena práctica limitar al mínimo necesario.
- **`model`:** Puede ser diferente del modelo del agente principal (ej. Haiku para tareas simples).

### Patrones comunes de uso de Agent

**Patrón 1: Procesamiento en paralelo**
```
Agente principal divide una tarea grande en subtareas:
  ├── Agent("Documenta src/auth/")
  ├── Agent("Documenta src/models/")
  ├── Agent("Documenta src/api/")
  └── Agent("Documenta src/utils/")
Los cuatro trabajan en paralelo (o en serie según implementación).
```

**Patrón 2: Especialización**
```
Agente principal orquesta especialistas:
  ├── Agent("Analiza vulnerabilidades de seguridad en src/")
  ├── Agent("Genera tests para las funciones sin cobertura")
  └── Agent("Actualiza el README con los cambios recientes")
Cada subagente es experto en su dominio.
```

**Patrón 3: Exploración independiente**
```
Agente principal pregunta a un subagente:
  Agent("Investiga cómo funciona la autenticación JWT en este proyecto y retorna un resumen de: qué archivos están involucrados, qué librerías se usan, y cómo fluye la autenticación")
El subagente explora sin contaminar el contexto del principal.
```

El sistema multi-agente se desarrolla completamente en el **Capítulo 6**.

---

## 4.10 Cómo Claude decide qué herramienta usar

### El proceso de decisión del LLM

El LLM no tiene una tabla de reglas explícitas ni un árbol de decisión codificado. La decisión emerge del razonamiento del modelo sobre:
1. La descripción de la tarea actual.
2. La información que ya tiene en contexto.
3. Las descripciones de las herramientas disponibles.
4. Los patrones aprendidos durante el entrenamiento (RLHF).

### El árbol de prioridades típico

Aunque no es explícito, Claude tiende a seguir este orden de preferencia:

```
PARA EXPLORAR EL PROYECTO:
  1. Glob           → Cuando necesita encontrar archivos por patrón
  2. Grep           → Cuando necesita encontrar texto específico
  3. Bash + find    → Para búsquedas más complejas o filtros específicos
  4. Read + Bash    → Para análisis detallado de archivos específicos

PARA ENTENDER EL CÓDIGO:
  1. Read (fragmento) → Si sabe qué líneas necesita (con start/end_line)
  2. Read (completo)  → Para archivos pequeños o cuando necesita todo
  3. Grep             → Para encontrar dónde está algo específico

PARA MODIFICAR EL CÓDIGO:
  1. Edit/MultiEdit  → Para cambios localizados en archivos existentes
  2. Write           → Para archivos nuevos o cambios extensos

PARA VERIFICAR:
  1. Bash + test     → Para ejecutar tests
  2. Bash + lint     → Para verificar calidad del código
  3. Read            → Para releer el archivo modificado y verificar

PARA OBTENER INFORMACIÓN EXTERNA:
  1. WebSearch       → Para búsquedas generales
  2. WebFetch        → Para leer una URL específica ya identificada
```

### El efecto de las descripciones en la elección

Un ejemplo de cómo la descripción afecta la elección:

```
Si el usuario dice: "Encuentra todos los archivos Python del proyecto"

Claude puede usar:
  1. Glob("**/*.py")  → La descripción de Glob menciona "encontrar archivos por patrón"
  2. Bash("find . -name '*.py'")  → La descripción de Bash menciona "ejecutar comandos del terminal"

En la mayoría de los casos, Claude elige Glob porque:
  - La descripción de Glob es más específica para esta tarea
  - Glob retorna resultados estructurados
  - La descripción de Glob fue escrita para guiar exactamente hacia este uso
```

---

## 4.11 Patrones de uso avanzados con herramientas

### Patrón 1: Read parcial + Grep para archivos grandes

Para archivos de más de 200-300 líneas, leer el archivo completo puede ser costoso. La estrategia eficiente:

```
TAREA: "Encuentra y corrige el bug en la clase UserRepository"

INEFICIENTE:
  Read("src/repositories/user_repository.py")  → 600 líneas, 7k tokens

EFICIENTE:
  1. Grep("class UserRepository", "src/repositories/user_repository.py")
     → "UserRepository está en línea 45"
  2. Grep("def get_user_by_email", "src/repositories/user_repository.py")
     → "get_user_by_email está en línea 87"
  3. Read("src/repositories/user_repository.py", start_line=85, end_line=120)
     → Solo 35 líneas relevantes
```

### Patrón 2: Bash para verificar antes de modificar

Antes de hacer cambios que podrían romper algo:

```bash
# Verificar el estado limpio del repositorio
git status
git diff --stat

# Solo si no hay cambios no commiteados, proceder con las modificaciones
# (si los hubiera, Claude puede avisar al usuario)
```

### Patrón 3: Grep para impacto de refactoring

Antes de renombrar una función o cambiar su firma:

```
Tarea: "Renombra hash_password() a create_password_hash()"

1. Grep("hash_password", ["."], file_pattern="*.py")
   → Encuentra todos los usos: auth.py, api/users.py, tests/test_auth.py
   
2. Para cada archivo con usos, usa Edit para hacer el cambio
   → auth.py: 2 cambios (definición + uso)
   → api/users.py: 1 cambio (importación + uso)
   → tests/test_auth.py: 3 cambios (en las funciones de test)
   
3. Verificación: Grep("hash_password", ["."])
   → Si retorna vacío, todos los usos fueron actualizados
   
4. Bash("pytest tests/ -v")
   → Confirmar que los tests siguen pasando
```

### Patrón 4: Uso de TodoRead/TodoWrite para tareas largas

Las herramientas `TodoRead` y `TodoWrite` permiten a Claude mantener una lista de tareas para proyectos largos:

```
INICIO DE SESIÓN LARGA:
  TodoRead()
  → Ver qué quedó pendiente de la sesión anterior

DURANTE LA SESIÓN:
  Completar una tarea → TodoWrite() para marcarla como completada
  Encontrar un subtask → TodoWrite() para añadirlo a la lista

AL FINAL DE LA SESIÓN:
  TodoRead() → Ver qué queda pendiente para la próxima sesión
```

---

## 4.12 Controlar las herramientas disponibles

### Por qué controlar las herramientas es importante

No siempre quieres que Claude tenga todas las herramientas disponibles:

- **Auditorías de seguridad:** Solo lectura, sin posibilidad de modificar archivos.
- **Entornos de producción:** Muy restringido, sin escritura de archivos ni comandos peligrosos.
- **CI/CD:** Solo las herramientas necesarias para el análisis o la tarea específica.
- **Demostraciones:** Limitar para mayor predecibilidad.

### `--allowedTools`: lista blanca (solo estas herramientas)

```bash
# MODO SOLO LECTURA: para auditorías, análisis, revisiones
claude --allowedTools "Read,Glob,Grep,WebSearch"

# MODO ANÁLISIS + EJECUCIÓN: puede leer y ejecutar, pero no modificar
claude --allowedTools "Read,Glob,Grep,Bash,WebSearch,WebFetch"

# MODO COMPLETO SIN INTERNET: sin acceso a la web
claude --allowedTools "Read,Write,Edit,MultiEdit,Bash,Glob,Grep,Agent"

# MODO COMPLETO SIN SUBAGENTES: single-agent explícito
claude --allowedTools "Read,Write,Edit,MultiEdit,Bash,Glob,Grep,WebSearch,WebFetch"
```

### `--disallowedTools`: lista negra (todo excepto estas)

```bash
# SIN INTERNET: útil en entornos air-gapped o privacidad estricta
claude --disallowedTools "WebSearch,WebFetch"

# SIN SUBAGENTES: single-agent para mayor control
claude --disallowedTools "Agent"

# SIN ESCRITURA: análisis sin riesgo de modificar archivos
claude --disallowedTools "Write,Edit,MultiEdit"

# SIN BASH: sin ejecución de comandos (solo manipulación de archivos)
claude --disallowedTools "Bash"
```

### En el archivo de configuración (persistente por proyecto)

Para configuración permanente en lugar de flags de línea de comandos:

```json
// .claude/settings.json
{
  "allowedTools": ["Read", "Write", "Edit", "Bash", "Glob", "Grep"],
  "disallowedTools": ["WebSearch", "WebFetch", "Agent"]
}
```

### Para CI/CD: modo completamente no interactivo

```bash
#!/bin/bash
# Script de CI para análisis de código con Claude

claude \
  --dangerously-skip-permissions \
  --allowedTools "Read,Bash,Glob,Grep" \
  --print "Revisa todos los archivos Python en src/ y lista:
           1. Posibles vulnerabilidades de seguridad
           2. Patrones de código que violan las convenciones del proyecto
           3. Tests con cobertura insuficiente
           
           Formato de salida: JSON con arrays 'security', 'conventions', 'coverage'"
```

`--dangerously-skip-permissions` deshabilita los prompts de confirmación (útil en CI donde no hay usuario para responder).

`--print` ejecuta Claude sin el REPL interactivo, imprime el resultado en stdout y termina. El script captura la salida con `$()`.

---

## 4.13 Resumen y glosario del capítulo

### Los puntos fundamentales

1. **Las herramientas son el puente** entre el LLM (que solo genera texto) y el sistema real. El LLM *decide* qué herramienta usar; el Tool Executor *ejecuta* la acción.

2. **El ciclo completo** es: LLM genera `tool_use` → Tool Executor ejecuta → resultado se añade como `tool_result` → LLM continúa. Cada ciclo es una petición HTTP completa.

3. **Read** es la herramienta más usada. Lee archivos de texto completos o por rango de líneas. Cada lectura añade tokens al contexto.

4. **Write** sobreescribe el archivo completo. Claude siempre lee antes de escribir para no perder contenido.

5. **Edit/MultiEdit** hacen cambios quirúrgicos (buscar-y-reemplazar). Más eficientes para cambios localizados en archivos grandes.

6. **Bash** ejecuta cualquier comando. El estado del shell NO persiste entre llamadas. Algunas operaciones destructivas requieren confirmación del usuario.

7. **Glob y Grep** son herramientas nativas para exploración eficiente del filesystem. Preferibles a Bash + find/grep por sus resultados estructurados.

8. **WebSearch/WebFetch** dan acceso a información de internet actualizada. WebFetch no ejecuta JavaScript.

9. **Agent** crea subagentes con contextos completamente independientes. Fundamental para tareas masivas y para no agotar el contexto.

10. **`--allowedTools` y `--disallowedTools`** permiten controlar precisamente qué puede hacer Claude. Esencial para CI/CD y entornos restrictivos.

### Glosario del capítulo

| Término | Definición |
|---------|------------|
| **Herramienta (Tool)** | Función que el LLM puede invocar para interactuar con el sistema real. El LLM decide, el Tool Executor ejecuta. |
| **JSON Schema** | Estándar para describir la estructura de objetos JSON. Define los parámetros de cada herramienta. |
| **tool_use** | Tipo de respuesta del LLM que indica que quiere invocar una herramienta. |
| **tool_result** | Mensaje que contiene el resultado de ejecutar una herramienta, enviado de vuelta al LLM. |
| **exit_code** | Código de retorno de un proceso Unix (0 = éxito, otro valor = error). |
| **Glob pattern** | Patrón para encontrar archivos por su ruta (ej. `**/*.py` busca todos los .py). |
| **old_string** | En Edit: el fragmento exacto a reemplazar. Debe ser único en el archivo. |
| **new_string** | En Edit: el texto que reemplaza a old_string. |
| **--allowedTools** | Flag de Claude Code para especificar lista blanca de herramientas disponibles. |
| **--disallowedTools** | Flag para especificar lista negra de herramientas no disponibles. |
| **--print** | Flag para ejecutar Claude en modo no interactivo (para scripts y CI/CD). |
| **--dangerously-skip-permissions** | Flag que deshabilita los prompts de confirmación (solo para CI/CD automatizado). |
| **RLHF** | Reinforcement Learning from Human Feedback. Técnica de entrenamiento que enseñó al LLM cuándo usar cada herramienta. |
| **Subagente** | Instancia de Claude Code creada por la herramienta Agent con su propio contexto independiente. |
| **TodoRead/TodoWrite** | Herramientas para gestionar una lista de tareas persistente entre iteraciones. |

---

## Ver también

- **[Capítulo 3](./cap-03-arquitectura-componentes.md):** Arquitectura — el Tool Executor que implementa las herramientas.
- **[Capítulo 5](./cap-05-ventana-contexto.md):** Ventana de Contexto — cómo el uso de herramientas afecta al contexto.
- **[Capítulo 6](./cap-06-sistema-multiagente.md):** Multi-Agente — el uso avanzado de la herramienta Agent.
- **[Capítulo 7](./cap-07-mcp-protocolo.md):** MCP — cómo añadir herramientas personalizadas más allá de las nativas.
- **[Capítulo 9](./cap-09-seguridad-permisos.md):** Seguridad — las confirmaciones y el modelo de permisos de Bash.

---

> 📌 Siguiente capítulo: [Cap. 5 — La Ventana de Contexto](./cap-05-ventana-contexto.md)

```json
{
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Voy a leer el archivo para entender la implementación actual."
    },
    {
      "type": "tool_use",
      "id": "toolu_01ABC123",
      "name": "Read",
      "input": { "path": "src/auth.py" }
    }
  ],
  "stop_reason": "tool_use"
}
```

**Paso 2: El Tool Executor ejecuta la herramienta localmente**

```
Extrae:  name = "Read", input = { "path": "src/auth.py" }
Ejecuta: Lee el archivo /proyecto/src/auth.py
Resultado: "import hashlib\n\ndef hash_password..."
```

**Paso 3: El resultado se añade al historial como `tool_result`**

```json
{
  "role": "user",
  "content": [{
    "type": "tool_result",
    "tool_use_id": "toolu_01ABC123",
    "content": "import hashlib\n\ndef hash_password(password):\n    return hashlib.md5(password.encode()).hexdigest()\n"
  }]
}
```

**Paso 4: El LLM recibe el resultado y continúa el ciclo ReAct**

Con el resultado en contexto, el LLM puede invocar otra herramienta o generar la respuesta final.

---

## 4.3 Herramienta Read: leer el sistema de archivos

### Qué hace

`Read` lee el contenido completo de un archivo de texto y lo retorna como string. Es la herramienta más frecuentemente usada.

### Cuándo Claude la usa

- **Antes de modificar cualquier archivo:** Para no sobreescribir contenido sin haberlo visto.
- **Para entender el contexto del proyecto:** Leer `requirements.txt`, `package.json`, `Makefile`.
- **Para ver tests existentes** antes de generar nuevos (para no duplicar ni contradecir).
- **Para leer configuraciones:** `.env.example`, `docker-compose.yml`, `pyproject.toml`.
- **Para entender la arquitectura:** Leer archivos de definición de modelos, schemas, etc.

### Parámetros

```
path (obligatorio): Ruta al archivo (relativa o absoluta).
start_line (opcional): Línea desde la que empezar (1-indexado).
end_line (opcional): Línea hasta la que leer (inclusivo).
```

### Limitaciones importantes

**Solo archivos de texto.** Los archivos binarios (imágenes, PDFs, ejecutables) darán garbage o error.

**Impacto en el contexto.** Cada archivo leído añade su contenido completo al historial. Un archivo de 500 líneas Python puede añadir ~5,000 tokens.

**Estrategia para archivos grandes:**

```
INEFICIENTE: Read("src/models/user.py")
  → 400 líneas, 4000 tokens consumidos

EFICIENTE:
  1. Grep("class UserProfile", "src/models/user.py")
     → Resultado: línea 87
  2. Read("src/models/user.py", start_line=85, end_line=120)
     → Solo 35 líneas relevantes, ~350 tokens
```

---

## 4.4 Herramienta Write: escribir y crear archivos

### El comportamiento de sobreescritura completa

Este es el punto más importante de `Write`: **sobreescribe TODO el contenido del archivo**.

```
ESTADO INICIAL src/utils.py:
  def suma(a, b): return a + b
  def resta(a, b): return a - b
  def multiplica(a, b): return a * b

Claude hace: Write("src/utils.py", "def suma(a, b):\n    return a + b\n")

ESTADO FINAL:
  def suma(a, b):
      return a + b

¡Las funciones resta() y multiplica() han desaparecido!
```

**Por eso Claude siempre lee antes de escribir.** El flujo correcto:
1. `Read` el archivo completo.
2. Construir el nuevo contenido completo en el razonamiento del LLM.
3. `Write` el archivo completo modificado.

### Creación automática de directorios

`Write` crea automáticamente los directorios intermedios si no existen:
```
Write("src/new_module/utils.py", contenido)
→ Si src/new_module/ no existe, lo crea automáticamente
```

### Write vs. Edit

- **Write:** Para archivos nuevos o cuando los cambios afectan a >50% del archivo.
- **Edit/MultiEdit:** Para modificaciones localizadas en archivos existentes (ver sección 4.5).

---

## 4.5 Herramienta Edit y MultiEdit: modificar sin reescribir

### El problema que resuelven

`Write` reescribe el archivo completo. Para cambiar una sola función en un archivo de 500 líneas, hay que incluir las 500 líneas completas en el payload y riesgo de perder partes del archivo si el LLM comete un error.

`Edit` resuelve esto con un **buscar-y-reemplazar preciso**:

```json
{
  "name": "Edit",
  "input": {
    "path": "src/auth.py",
    "old_string": "return hashlib.md5(password.encode()).hexdigest()",
    "new_string": "salt = bcrypt.gensalt()\n    return bcrypt.hashpw(password.encode('utf-8'), salt).decode('utf-8')"
  }
}
```

El Tool Executor:
1. Abre el archivo.
2. Busca exactamente la cadena `old_string`.
3. La reemplaza con `new_string`.
4. Guarda el archivo.

Si `old_string` no se encuentra, retorna un error y Claude puede ajustar.

### MultiEdit: múltiples cambios atómicos

`MultiEdit` aplica varias ediciones en el mismo archivo con una sola invocación:

```json
{
  "name": "MultiEdit",
  "input": {
    "path": "src/auth.py",
    "edits": [
      {
        "old_string": "import hashlib",
        "new_string": "import bcrypt"
      },
      {
        "old_string": "return hashlib.md5(password.encode()).hexdigest()",
        "new_string": "salt = bcrypt.gensalt()\n    return bcrypt.hashpw(password.encode(), salt).decode()"
      },
      {
        "old_string": "return hash_password(password) == hashed",
        "new_string": "return bcrypt.checkpw(password.encode(), hashed.encode())"
      }
    ]
  }
}
```

Las ediciones se aplican de forma **atómica** (todas o ninguna) y en una sola llamada.

### Cuándo usar Edit vs. Write

| Situación | Herramienta recomendada |
|-----------|------------------------|
| Creando un archivo nuevo | **Write** |
| Archivo pequeño (< 30 líneas) | **Write** |
| Cambios en < 50% del archivo | **Edit / MultiEdit** |
| Archivo grande (> 100 líneas) | **Edit / MultiEdit** |
| Cambios en > 70% del archivo | **Write** |

> 📌 Siguiente capítulo: [Cap. 5 — La Ventana de Contexto](./cap-05-ventana-contexto.md)

### Casos de uso principales

```bash
# Ejecutar tests
pytest tests/ -v
jest --coverage
cargo test

# Gestionar dependencias
pip install bcrypt==4.1.2
npm install lodash

# Buscar en código
grep -r "hash_password" src/ --include="*.py"
git diff HEAD~1
git log --oneline -10

# Compilar o construir
npm run build
cargo build --release
```

### El exit code es información crítica

Claude usa el exit code para determinar si el comando tuvo éxito:
- `exit_code: 0` → éxito
- `exit_code: 1+` → error

Esto le permite a Claude reaccionar ante fallos de tests, errores de compilación, etc.

### Operaciones que requieren confirmación del usuario

```
SIEMPRE PIDEN CONFIRMACIÓN (destructivas):
  rm, rmdir, unlink         → eliminar archivos
  sudo                      → elevación de privilegios
  git push                  → enviar cambios al remoto
  git reset --hard          → posible pérdida de trabajo

NUNCA PIDEN CONFIRMACIÓN (seguras/reversibles):
  cat, ls, find, grep       → solo lectura
  pytest, jest, cargo test  → ejecutar tests
  git status, diff, log     → git de solo lectura
  pip install, npm install  → instalar dependencias
  black, eslint             → formateo/lint
```

### La persistencia del estado del shell

**Importante:** El estado del shell **no persiste** entre llamadas a Bash. Cada comando se ejecuta en un subshell nuevo.

```bash
# Llamada 1: cd /tmp && export MY_VAR="hello"
# Llamada 2: echo $MY_VAR  → vacío (MY_VAR no persiste)
#            pwd           → directorio original (no /tmp)

# SOLUCIÓN: concatenar operaciones o usar rutas absolutas
cd /tmp && ls -la && cat file.txt   # Todo en una sola llamada
ls /tmp/file.txt                    # Ruta absoluta
```

### Los timeouts

Los comandos tienen un timeout de varios minutos. Para comandos que se sabe que tardan mucho:

```bash
# Añadir timeout explícito
timeout 60 pytest tests/  # Máximo 60 segundos
```

---

## 4.7 Herramientas Glob y Grep: exploración eficiente

### Herramienta Glob: encontrar archivos por patrón

`Glob` encuentra archivos cuya ruta coincida con un patrón glob:

```
Patrones más útiles:
  *.py          → todos los .py en el directorio actual
  **/*.py       → todos los .py en cualquier subdirectorio
  src/**/*.ts   → todos los .ts bajo src/
  tests/test_*  → archivos que empiezan por test_ en tests/
```

```json
{
  "name": "Glob",
  "input": {
    "pattern": "**/*.py",
    "exclude": ["**/node_modules/**", "**/__pycache__/**", "**/venv/**"]
  }
}
```

Resultado estructurado:
```
src/auth.py
src/models/user.py
tests/test_auth.py
tests/test_models.py
```

**Cuándo usarla:**
- Explorar la estructura del proyecto sin leer archivos.
- Encontrar todos los archivos de un tipo específico.
- Antes de hacer cambios que afectan a múltiples archivos.

### Herramienta Grep: buscar texto en archivos

`Grep` busca un patrón de texto (o regex) en uno o más archivos:

```json
{
  "name": "Grep",
  "input": {
    "pattern": "hash_password",
    "paths": ["src/", "tests/"],
    "file_pattern": "*.py",
    "context_lines": 3
  }
}
```

Resultado:
```
src/auth.py:3:def hash_password(password: str) -> str:
src/auth.py:8:    return hash_password(password) == hashed
tests/test_auth.py:5:    result = hash_password('test123')
```

**Cuándo usarla:**
- Encontrar todos los usos de una función antes de refactorizarla.
- Buscar TODO/FIXME comments.
- Verificar que un import se usa en todos los archivos que debería.
- Encontrar dónde se instancia una clase.

### Ventaja sobre Bash con find/grep

Las herramientas nativas Glob y Grep están optimizadas y retornan resultados estructurados. No requieren conocer la sintaxis exacta de `find` o `grep` del sistema.

---

## 4.8 WebSearch y WebFetch: acceso a internet

### WebSearch: búsqueda en internet

`WebSearch` realiza una búsqueda y retorna títulos, URLs y fragmentos:

```json
{
  "name": "WebSearch",
  "input": {
    "query": "bcrypt python best practices 2025",
    "num_results": 5
  }
}
```

**Cuándo Claude la usa:**
- Buscar documentación de una librería que no conoce bien.
- Verificar si una API ha cambiado desde su fecha de entrenamiento.
- Buscar soluciones a mensajes de error específicos.
- Encontrar la versión más reciente de un paquete.

### WebFetch: leer una URL específica

`WebFetch` descarga el contenido de una URL:

```json
{
  "name": "WebFetch",
  "input": {
    "url": "https://pypi.org/project/bcrypt/",
    "extract_text": true
  }
}
```

**Limitaciones importantes:**
- **No ejecuta JavaScript.** Las páginas que cargan contenido dinámicamente pueden aparecer vacías.
- **El contenido se añade al contexto.** Una página web puede ser enorme; usar con moderación.
- **Solo información pública.** No puede acceder a páginas que requieran autenticación.

### Desactivar el acceso a internet

```bash
claude --disallowedTools "WebSearch,WebFetch"
```

---

## 4.9 Herramienta Agent: crear subagentes

La herramienta `Agent` es cualitativamente diferente de las demás. En lugar de interactuar con el sistema operativo, **crea una nueva instancia del agente Claude Code** que trabaja de forma independiente.

```json
{
  "name": "Agent",
  "input": {
    "task": "Documenta todos los archivos en src/auth/ con docstrings completos siguiendo el estilo Google. Para cada función pública, incluye: descripción, parámetros con tipos, valor de retorno, y un ejemplo de uso.",
    "context": "El proyecto usa Python 3.12. El estilo de docstring de referencia está en src/models/user.py.",
    "allowed_tools": ["Read", "Write", "Glob"],
    "model": "claude-sonnet-4-20250514"
  }
}
```

**Características clave:**
- El subagente tiene su **propio contexto aislado** (independiente del agente principal).
- Ejecuta su propio ciclo ReAct hasta completar la tarea.
- Cuando termina, retorna el resultado al agente principal.
- Los subagentes son completamente aislados entre sí.

La ventaja principal: **cada subagente trabaja con contexto fresco**, produciendo resultados de calidad consistente incluso para tareas que involucran muchos archivos.

Este sistema se desarrolla en profundidad en el **Capítulo 6**.

---

## 4.10 Cómo Claude decide qué herramienta usar

### El mecanismo de decisión

El LLM no tiene una tabla de reglas explícitas. La decisión emerge del razonamiento sobre la **descripción de cada herramienta** en el contexto de la tarea. El modelo fue entrenado con RLHF (Reinforcement Learning from Human Feedback) para elegir las herramientas apropiadas.

### El orden de preferencia típico

**Para exploración del proyecto:**
```
1. Glob  → Para encontrar archivos por patrón (más eficiente)
2. Grep  → Para buscar texto específico en archivos
3. Bash con find/grep → Si necesita lógica más compleja
```

**Para leer contenido:**
```
1. Read                         → Para leer archivos completos
2. Read con start/end_line      → Para fragmentos de archivos grandes
3. Bash con cat/head/tail       → Para casos especiales con filtros
```

**Para modificar archivos:**
```
1. Edit/MultiEdit  → Para cambios localizados en archivos existentes
2. Write           → Para archivos nuevos o cambios extensos
```

**Para obtener información de internet:**
```
1. WebSearch → Para buscar información general o documentación
2. WebFetch  → Para leer una URL específica ya identificada
```

---

## 4.11 Controlar las herramientas disponibles

### `--allowedTools`: lista blanca (solo estas herramientas)

```bash
# Solo lectura (análisis, auditoría sin modificar nada)
claude --allowedTools "Read,Glob,Grep"

# Lectura + ejecución de tests, sin escritura de archivos
claude --allowedTools "Read,Glob,Grep,Bash"

# Sin subagentes (modo single-agent explícito)
claude --allowedTools "Read,Write,Edit,MultiEdit,Bash,Glob,Grep,WebSearch"
```

### `--disallowedTools`: lista negra (todo excepto estas)

```bash
# Sin internet
claude --disallowedTools "WebSearch,WebFetch"

# Sin subagentes
claude --disallowedTools "Agent"

# Sin modificar archivos (solo análisis)
claude --disallowedTools "Write,Edit,MultiEdit"
```

### Para uso en CI/CD

```bash
# Ejecutar tests y reportar fallos, sin modificar nada
claude \
  --dangerously-skip-permissions \
  --allowedTools "Read,Bash,Glob,Grep" \
  --print "Ejecuta toda la suite de tests y reporta los fallos con sus stack traces"
```

`--print` ejecuta Claude en modo no interactivo (sin REPL), imprime la respuesta y termina. Ideal para scripts de CI/CD.

### En el archivo de configuración (persistente)

```json
// .claude/settings.json
{
  "allowedTools": ["Read", "Write", "Edit", "Bash", "Glob", "Grep"],
  "disallowedTools": ["WebSearch", "Agent"]
}
```

---

## 4.12 Resumen y glosario

### Resumen

1. Las **herramientas** se definen con JSON Schema: nombre, descripción y parámetros. El LLM usa la descripción para decidir cuándo usarlas.
2. El ciclo: LLM genera `tool_use` → Tool Executor ejecuta localmente → resultado se añade como `tool_result` → LLM continúa.
3. **Read** lee archivos de texto (o por rango de líneas). Impacta el tamaño del contexto.
4. **Write** sobreescribe el archivo completo. Claude siempre lee antes de escribir.
5. **Edit/MultiEdit** hacen cambios quirúrgicos (buscar-y-reemplazar); más eficientes para cambios localizados.
6. **Bash** ejecuta cualquier comando del sistema. El estado del shell no persiste entre llamadas. Algunos comandos piden confirmación.
7. **Glob** y **Grep** son herramientas nativas para exploración eficiente del filesystem y búsqueda de texto.
8. **WebSearch** y **WebFetch** permiten acceder a información de internet.
9. **Agent** crea subagentes con contextos independientes para tareas paralelas (Capítulo 6).
10. Las herramientas se controlan con `--allowedTools` y `--disallowedTools`.

### Glosario

| Término | Definición |
|---------|------------|
| **Herramienta (Tool)** | Función que el LLM puede invocar para interactuar con el sistema real. |
| **JSON Schema** | Estándar para describir la estructura de datos JSON. Usado para definir herramientas. |
| **tool_use** | Tipo de respuesta del LLM cuando quiere invocar una herramienta. |
| **tool_result** | Mensaje que contiene el resultado de ejecutar una herramienta. |
| **exit_code** | Código de retorno de un proceso (0 = éxito, otro = error). |
| **Glob** | Patrón para encontrar archivos por su ruta (ej. `**/*.py`). |
| **RLHF** | Reinforcement Learning from Human Feedback. Técnica de entrenamiento del LLM. |
| **--allowedTools** | Flag de Claude Code para especificar lista blanca de herramientas. |
| **--disallowedTools** | Flag para especificar lista negra de herramientas. |
| **--print** | Flag para ejecutar Claude en modo no interactivo (para scripts). |

---

## Ver también

- **[Capítulo 3](./cap-03-arquitectura-componentes.md):** Arquitectura — el Tool Executor que implementa las herramientas.
- **[Capítulo 6](./cap-06-sistema-multiagente.md):** Multi-Agente — el uso avanzado de la herramienta Agent.
- **[Capítulo 7](./cap-07-mcp-protocolo.md):** MCP — cómo añadir herramientas personalizadas.
- **[Capítulo 9](./cap-09-seguridad-permisos.md):** Seguridad — las confirmaciones y el modelo de permisos.

---

> 📌 Siguiente capítulo: [Cap. 5 — La Ventana de Contexto](./cap-05-ventana-contexto.md)
