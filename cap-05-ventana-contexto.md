# Capítulo 5: La Ventana de Contexto — Memoria, Tokens y Gestión

> **Nivel:** Sin conocimientos previos requeridos  
> **Prerequisito recomendado:** [Cap. 1](./cap-01-que-es-claude-code.md) y [Cap. 3](./cap-03-arquitectura-componentes.md)  
> **Tiempo de lectura estimado:** 50-60 minutos  
> **Objetivo:** Entender qué es la ventana de contexto, qué la llena, por qué se degrada la calidad cuando se acerca al límite, y todas las estrategias disponibles para gestionarla eficientemente.

---

## Tabla de Contenidos

- [5.1 La mesa de trabajo del carpintero: modelo mental esencial](#51-la-mesa-de-trabajo-del-carpintero-modelo-mental-esencial)
- [5.2 Qué es un token: explicación completa desde cero](#52-qué-es-un-token-explicación-completa-desde-cero)
- [5.3 Los 200,000 tokens: qué significa en la práctica real](#53-los-200000-tokens-qué-significa-en-la-práctica-real)
- [5.4 Anatomía del contexto: qué ocupa qué y cuánto](#54-anatomía-del-contexto-qué-ocupa-qué-y-cuánto)
- [5.5 El crecimiento del contexto durante una sesión real](#55-el-crecimiento-del-contexto-durante-una-sesión-real)
- [5.6 El fenómeno "Lost in the Middle"](#56-el-fenómeno-lost-in-the-middle)
- [5.7 Prompt Caching: el mecanismo de ahorro automático](#57-prompt-caching-el-mecanismo-de-ahorro-automático)
- [5.8 El comando /compact: comprimir sin perder el hilo](#58-el-comando-compact-comprimir-sin-perder-el-hilo)
- [5.9 El comando /clear: empezar desde cero](#59-el-comando-clear-empezar-desde-cero)
- [5.10 Estrategias proactivas de gestión del contexto](#510-estrategias-proactivas-de-gestión-del-contexto)
- [5.11 Optimización del costo: input vs output tokens](#511-optimización-del-costo-input-vs-output-tokens)
- [5.12 Resumen y glosario del capítulo](#512-resumen-y-glosario-del-capítulo)

---

## 5.1 La mesa de trabajo del carpintero: modelo mental esencial

### La analogía fundamental

Antes de los números y los términos técnicos, una analogía que captura perfectamente la esencia de la ventana de contexto:

Imagina que trabajas en un banco de carpintería de tamaño fijo. Para hacer una pieza de madera, necesitas tener a la vista y al alcance de la mano varios elementos simultáneamente: los planos del diseño, las herramientas que estás usando, las piezas ya terminadas del ensamblaje, los materiales pendientes, y las notas sobre decisiones que tomaste a lo largo del trabajo.

Cuando la mesa está vacía al inicio de la jornada, tienes todo el espacio del mundo. Puedes extender los planos, dejar las herramientas organizadas, y trabajar con comodidad. Conforme avanza el trabajo, la mesa se llena: los planos ya revisados, las piezas terminadas temporalmente apoyadas, el serrín y los recortes de materiales ya usados.

Eventualmente, la mesa se llena completamente. Y aquí está el problema crítico: **cuando la mesa está completamente llena, el carpintero empieza a trabajar peor**. Ya no puede ver bien los planos del principio porque están cubiertos por cosas. Tarda más en encontrar las herramientas. Su trabajo se vuelve menos preciso.

**La ventana de contexto de Claude es exactamente esa mesa de trabajo.** Tiene un tamaño fijo de aproximadamente 200,000 tokens. Todo lo que ocurre en la sesión ocupa espacio en esa mesa:
- Cada mensaje tuyo.
- Cada respuesta de Claude.
- Cada archivo que Claude lee (completo, en el historial).
- Cada output de comando que Claude ejecuta.
- Las instrucciones del CLAUDE.md.

Cuando la mesa se llena, Claude empieza a "perder de vista" el trabajo de hace horas.

**`/compact`** es como quitar de la mesa los materiales ya usados y guardar solo las notas importantes de lo que se hizo: "corté las piezas A, B y C, ahora toca ensamblar D y E". La mesa queda casi vacía, pero conservas lo esencial.

**`/clear`** es limpiar la mesa completamente para empezar una tarea nueva desde cero.

### Por qué la "mesa" no puede hacerse más grande

La pregunta natural es: ¿por qué no hacer la ventana de contexto más grande — 1 millón de tokens, 10 millones?

La respuesta técnica: procesar el contexto tiene un costo computacional que crece aproximadamente de forma **cuadrática** con el número de tokens (debido a la atención en los Transformers). Doblar el contexto no dobla el tiempo de procesamiento: lo multiplica por cuatro. Los 200,000 tokens ya son una de las ventanas de contexto más grandes de la industria — mantenerla abierta durante horas ya es un logro de ingeniería enorme.

La respuesta práctica: para proyectos reales, 200,000 tokens es suficiente si se gestiona bien. Un archivo de código típico tiene 100-300 líneas, lo que son 1,000-3,000 tokens. La ventana puede contener decenas de archivos simultáneamente. El problema no es el tamaño absoluto, sino la acumulación descontrolada.

---

## 5.2 Qué es un token: explicación completa desde cero

### Por qué los LLMs no usan caracteres ni palabras

Para procesar texto de forma eficiente, los LLMs necesitan una unidad de representación intermedia: no tan granular como un carácter (ineficiente, demasiados pasos), no tan grande como una palabra (distribución muy desigual, palabras raras no vistas en entrenamiento).

La solución es el **token**: un fragmento de texto de longitud variable que corresponde a los patrones más frecuentes del corpus de entrenamiento.

### Byte-Pair Encoding: cómo se crean los tokens

El algoritmo que genera los tokens se llama **Byte-Pair Encoding (BPE)**. Funciona así:

```
PROCESO BPE (simplificado)
═══════════════════════════════════════════════════════════════

PASO 1: Empieza con todos los caracteres individuales como tokens.

PASO 2: Encuentra el par de caracteres que aparece con más frecuencia
        en todo el corpus de entrenamiento.

PASO 3: Fusiona ese par en un nuevo token.

PASO 4: Repite hasta tener el tamaño de vocabulario deseado (~50,000-100,000).

RESULTADO PARA TEXTO EN INGLÉS:
  - Palabras muy comunes como "the", "is", "of" → tokens únicos
  - Palabras frecuentes como "program", "function" → tokens únicos
  - Palabras técnicas largas se dividen: "authentication" → "authen"+"tication"
  - Símbolos comunes: ".", ",", "{", "}", "=>" → tokens únicos
```

### Ejemplos concretos de tokenización

```
INGLÉS (muy eficiente — el modelo fue entrenado principalmente con inglés):
  "Hello, world!"    → "Hello" | "," | " world" | "!"         = 4 tokens
  "programming"      → "programm" | "ing"                     = 2 tokens

ESPAÑOL (menos eficiente — ~10-20% más tokens):
  "¡Hola, mundo!"    → "¡" | "Hola" | "," | " mundo" | "!"   = 5 tokens
  "programación"     → "program" | "aci" | "ón"               = 3 tokens
  "autenticación"    → "autent" | "icaci" | "ón"              = 3 tokens

CÓDIGO PYTHON (muy eficiente):
  "def hello():"     → "def" | " hello" | "()" | ":"          = 4 tokens
  "import bcrypt"    → "import" | " bc" | "rypt"              = 3 tokens
  "return True"      → "return" | " True"                     = 2 tokens
  "    " (4 espacios)→ "    "                                  = 1 token

CÓDIGO JSON (menos eficiente por las comillas y llaves):
  '"name": "Alice"'  → '"' | "name" | '"' | ":" | ' "' | "Alice" | '"' = 7 tokens
  
NÚMEROS:
  "42"               → "42"                                    = 1 token
  "12345"            → "123" | "45"                           = 2 tokens
  "3.14159"          → "3" | "." | "14" | "159"              = 4 tokens
  
MARKDOWN:
  "## Heading"       → "##" | " Heading"                     = 2 tokens
  "**bold text**"    → "**" | "bold" | " text" | "**"        = 4 tokens
```

### La tabla de estimación práctica

| Tipo de contenido | Regla de estimación |
|-------------------|---------------------|
| Texto en inglés | 1 token ≈ 4 caracteres |
| Texto en español | 1 token ≈ 3.5 caracteres |
| Código Python/JavaScript | 1 token ≈ 3-4 caracteres |
| JSON / YAML con muchas comillas | 1 token ≈ 3 caracteres |
| 1 línea de código (80 chars) | ≈ 20-25 tokens |
| 1 párrafo (100 palabras inglés) | ≈ 130-150 tokens |
| 1 página (500 palabras) | ≈ 650-750 tokens |
| 1 archivo Python de 100 líneas | ≈ 1,200-2,500 tokens |
| 1 archivo Python de 500 líneas | ≈ 6,000-12,000 tokens |
| 1 archivo JSON de configuración (2KB) | ≈ 500-700 tokens |
| Un README.md típico (2 páginas) | ≈ 1,000-1,500 tokens |

### Los tres impactos de los tokens que cambian tu forma de trabajar

**1. El costo: pagas por cada token**

La API de Anthropic cobra por token. Hay dos tipos con precios diferentes:
- **Input tokens:** Todo lo que envías (mensajes, historial, resultados de herramientas, CLAUDE.md). Precio más bajo.
- **Output tokens:** Todo lo que genera el modelo (sus respuestas, las solicitudes de herramientas). Precio más alto (generalmente 3-5x los input tokens).

En una sesión larga, los input tokens dominan el costo porque el historial acumulado crece y se envía completo con cada petición.

**2. La velocidad: más tokens = más lento**

El tiempo hasta el primer token de respuesta escala con el tamaño del contexto. Una petición con 5,000 tokens de contexto puede responder en 0.5 segundos. Una con 150,000 tokens puede tardar 10-15 segundos para el mismo output.

**3. La calidad: el fenómeno "Lost in the Middle"**

Los LLMs no atienden uniformemente a todo el contexto. La información en el centro de un contexto muy largo recibe menos atención que la del principio y el final. Se desarrolla en la sección 5.6.

---

## 5.3 Los 200,000 tokens: qué significa en la práctica real

### La escala en perspectiva

```
200,000 TOKENS SON APROXIMADAMENTE:
═══════════════════════════════════════════════════════════════

En texto:
  ~150,000 palabras en inglés
  ~130,000 palabras en español
  ~600 páginas de libro en formato A4
  El texto completo de "El Señor de los Anillos" (trilogía)
  5-6 novelas de longitud media

En código:
  ~50,000-80,000 líneas de código Python
  Equivalente a un proyecto mediano completo

En trabajo con Claude Code (sesión típica):
  50-100 archivos leídos completamente (100-200 líneas cada uno)
  200-500 iteraciones del ciclo ReAct
  30-60 minutos de trabajo muy intenso con muchos archivos grandes
  ó 2-4 horas de trabajo normal con archivos medianos
```

### Por qué el contexto se llena más rápido de lo esperado

Hay una trampa psicológica: 200,000 tokens parece enorme, pero en la práctica el contexto se agota mucho más rápido de lo intuitivo. Las razones:

**Acumulación de resultados de herramientas:**
Cada archivo que Claude lee no desaparece del contexto cuando "termina" de usarlo. El contenido del archivo permanece en el historial de la sesión. Si Claude lee 20 archivos durante la sesión, los 20 archivos siguen en el contexto, aunque Claude ya no los necesite activamente.

**Los outputs de comandos son verbosos:**
Un `pytest tests/ -v` con una suite mediana puede generar 3,000-8,000 tokens de output. Un `npm install` con muchas dependencias puede generar 5,000+ tokens. `git log --all` en un repositorio con historial largo puede generar decenas de miles de tokens.

**El historial conversacional crece:**
Cada mensaje tuyo + cada respuesta de Claude se añade al historial. En una sesión de trabajo intenso con mensajes largos, esto puede ser 1,000-2,000 tokens adicionales por cada intercambio.

**Las respuestas largas de Claude también cuentan:**
Cuando Claude genera código extenso o documentación detallada, esos tokens se añaden al historial como tokens del "assistant". Una respuesta de 2,000 tokens son 2,000 tokens más en el contexto.

---

## 5.4 Anatomía del contexto: qué ocupa qué y cuánto

### El mapa de ocupación de la ventana

```
VENTANA DE CONTEXTO DE 200,000 TOKENS
═══════════════════════════════════════════════════════════════
┌──────────────────────────────────────────────────────────────┐
│ SISTEMA (fijo en toda la sesión)                             │
│  • System prompt base de Claude Code         ~800 tokens     │
│  • ~/.claude/CLAUDE.md (instrucciones globales) ~200-2,000   │
│  • ./CLAUDE.md (instrucciones del proyecto)  ~300-3,000      │
│  Total sistema: ~1,300 - 5,800 tokens                        │
├──────────────────────────────────────────────────────────────┤
│ HISTORIAL ACUMULADO (crece con cada intercambio)             │
│                                                              │
│  Intercambio 1:                                              │
│  • Mensaje usuario: "Lee auth.py y dime el bug"  ~50 tokens  │
│  • tool_use: Read(auth.py)                       ~20 tokens  │
│  • tool_result: [contenido de auth.py 100 líneas] ~1,200 tok │
│  • Respuesta Claude: "El bug es que usa MD5..."  ~300 tokens  │
│  Subtotal intercambio 1:                         ~1,570 tokens│
│                                                              │
│  Intercambio 2:                                              │
│  • Mensaje usuario: "Corrígelo"                  ~15 tokens  │
│  • tool_use: Write(auth.py)                      ~200 tokens │
│  • tool_result: "archivo escrito"                ~5 tokens   │
│  • tool_use: Bash(pytest tests/ -v)              ~30 tokens  │
│  • tool_result: [output pytest, 3,000 tokens]   ~3,000 tokens│
│  • Respuesta Claude: "Corregido, tests pasan"    ~200 tokens │
│  Subtotal intercambio 2:                         ~3,450 tokens│
│                                                              │
│  ... 8 intercambios más similares ...            ~20,000 tok │
│                                                              │
│  Total historial después de 10 intercambios:    ~50,000 tok  │
├──────────────────────────────────────────────────────────────┤
│ MENSAJE ACTUAL (el que acabas de enviar)          ~50-500 tok │
└──────────────────────────────────────────────────────────────┘
TOTAL USADO: ~52,000-56,000 tokens (26-28% de 200,000)
```

### Los grandes consumidores: lo que dispara el consumo

**1. Archivos grandes leídos completamente:**
```
Ejemplo con un archivo grande:
  src/models.py — 600 líneas Python
  Estimación: 600 líneas × 20 tokens/línea = ~12,000 tokens

  Si en una sesión Claude lee 8 archivos similares:
  8 × 12,000 = 96,000 tokens solo en archivos leídos
  (esto es el 48% del contexto total disponible)
```

**2. Outputs de comandos verbosos:**
```
pytest tests/ -v (suite mediana, 50 tests):   ~3,000-6,000 tokens
pytest tests/ -v --cov=src (con cobertura):   ~5,000-10,000 tokens
npm install (muchas dependencias):             ~2,000-8,000 tokens
docker compose up -d (logs de inicio):        ~2,000-5,000 tokens
git log --all --oneline (repo con historial): ~variable, puede ser enorme
```

**3. Respuestas largas de Claude:**
```
Claude generando 20 docstrings completos:     ~5,000-8,000 tokens
Claude generando un módulo Python nuevo:      ~3,000-6,000 tokens
Claude explicando una arquitectura compleja:  ~2,000-4,000 tokens
```

### La regla del 70%

Cuando `/cost` muestra más del 70% de uso del contexto:
- La latencia aumenta notablemente (contexto grande = más tiempo de procesamiento).
- El fenómeno "Lost in the Middle" empieza a afectar la calidad.
- Estás a pocas horas de alcanzar el límite.

**Acción recomendada al llegar al 70%:** Usa `/compact` de forma preventiva.

---

## 5.5 El crecimiento del contexto durante una sesión real

### La progresión típica en una sesión de trabajo

```
PROGRESIÓN DEL CONTEXTO EN UNA SESIÓN DE 3 HORAS
═══════════════════════════════════════════════════════════════

T=0:00  INICIO
  Tokens: ~3,000 (system + CLAUDE.md)
  Mesa: casi vacía
  Latencia por petición: ~0.5-1s

T=0:15  EXPLORACIÓN INICIAL (15 minutos)
  Claude lee: README, estructura, 3-4 archivos clave
  Tokens: ~15,000-25,000
  Mesa: 8-12% ocupada
  Latencia: ~1-2s
  Estado: excelente

T=1:00  TRABAJO ACTIVO (1 hora)
  Claude ha leído 10-15 archivos, ejecutado 15-20 comandos
  Tokens: ~60,000-80,000
  Mesa: 30-40% ocupada
  Latencia: ~2-4s
  Estado: bueno, sin problemas notables

T=1:30  SEÑAL DE ALERTA (/cost muestra 50-60%)
  Tokens: ~100,000-120,000
  Latencia: ~4-7s
  Estado: funcional pero con latencia notable
  ACCIÓN RECOMENDADA: /compact si la sesión va a continuar

T=2:00  ZONA DE RIESGO (sin /compact)
  Tokens: ~130,000-150,000
  Latencia: ~7-12s
  Estado: calidad puede degradarse, "Lost in the Middle" activo
  ACCIÓN URGENTE: /compact o /clear

T=3:00  LÍMITE CERCANO (sin acciones tomadas)
  Tokens: ~180,000-195,000
  Latencia: ~15-25s
  Estado: calidad degradada, riesgo de error por contexto excedido
  CRISIS: /compact inmediato o Claude tirará error en próxima petición
```

### Señales de que el contexto necesita atención (lista práctica)

```
SEÑALES TEMPRANAS (actuar pronto):
  □ /cost muestra >60% de uso
  □ La latencia es notablemente mayor que al inicio
  □ La sesión lleva más de 1-1.5 horas de trabajo activo

SEÑALES URGENTES (actuar ahora):
  □ /cost muestra >75% de uso
  □ Claude responde "no recuerdo" a cosas discutidas hace 30 min
  □ Las respuestas contienen inconsistencias con lo discutido antes
  □ Latencia >15 segundos por respuesta

SEÑAL CRÍTICA (demasiado tarde, /compact ya):
  □ Error "context_length_exceeded" de la API
  □ Claude Code muestra una advertencia de contexto lleno
```

---

## 5.6 El fenómeno "Lost in the Middle"

### El problema científicamente documentado

En 2023, un paper de Nelson Liu et al. (Stanford/MIT) documentó un problema sistemático en los LLMs: los modelos tienen dificultades para utilizar eficientemente información que aparece en el **centro** de un contexto largo.

El paper, titulado *"Lost in the Middle: How Language Models Use Long Contexts"*, demostró que:
- Los modelos prestan mucha atención al inicio del contexto (el system prompt, los primeros mensajes).
- Los modelos prestan mucha atención al final del contexto (los mensajes más recientes).
- Los modelos prestan significativamente menos atención al **centro** del contexto.

### La curva de atención visualizada

```
ATENCIÓN DEL LLM SEGÚN POSICIÓN EN EL CONTEXTO
═══════════════════════════════════════════════════════════════

Atención
  Alta  ████                                            ████
        ████                                            ████
        ██████                                        ██████
          ████████                              ████████
              █████████████████████████████████
  Baja
        ←── INICIO ────────── CENTRO ─────────── FINAL ──►
           (system,           (información          (mensajes
           CLAUDE.md)          de hace              recientes)
                               1-2 horas)

ZONA PROBLEMÁTICA: El "centro" del contexto, donde está
información leída hace 1-2 horas, recibe menos atención.
```

### Consecuencias prácticas en Claude Code

```
EJEMPLO REAL:
═══════════════════════════════════════════════════════════════

HORA 1: Claude lee src/auth.py
  → Aprende que la función hash_password usa bcrypt con factor 12
  → Esta información queda en el INICIO del historial

HORA 2: 30 mensajes después, contexto casi lleno
  → hash_password ahora está en el CENTRO del contexto
  → Claude puede "olvidar" que usa factor 12

  → Tú preguntas: "¿qué factor de bcrypt usamos?"
  → Claude puede responder: "No recuerdo el factor exacto, habría
    que revisar auth.py" (aunque la información SÍ está en el contexto,
    solo que en el centro)
  
  → O peor: Claude asume un factor diferente al hacer cambios

SÍNTOMAS TÍPICOS:
  • Claude contradice algo que estableció hace 1-2 horas
  • Claude "redescubre" decisiones ya tomadas
  • Cambios que rompen convenciones establecidas al inicio de la sesión
```

### Cómo mitigarlo

**Técnica 1: `/compact` frecuente.** Al resumir el historial, la información importante pasa del "centro" al "inicio" (el resumen), donde recibe máxima atención.

**Técnica 2: Repetir información crítica.** Si hay algo muy importante que Claude necesita recordar, menciónalo de nuevo: "Recuerda que usamos bcrypt con factor 12..."

**Técnica 3: CLAUDE.md para información permanente.** Lo que pongas en el CLAUDE.md siempre está en el inicio del contexto (máxima atención) y nunca "se pierde en el centro".

---

## 5.7 Prompt Caching: el mecanismo de ahorro automático

### Qué es el Prompt Caching

**Prompt Caching** es una funcionalidad de la API de Anthropic que permite almacenar en caché (guardar en memoria temporal) el prefijo del contexto entre peticiones. Cuando la siguiente petición envía el mismo prefijo, en lugar de procesarlo de nuevo, lo lee desde la caché.

Este mecanismo se activa automáticamente — no requiere configuración.

### El ahorro que produce: ejemplo numérico real

```
SESIÓN SIN CACHING (para ilustrar el costo):
═══════════════════════════════════════════════════════════════

Petición 1: Contexto = 10,000 tokens → procesa 10,000 tokens
Petición 2: Contexto = 12,000 tokens → procesa 12,000 tokens
Petición 3: Contexto = 14,500 tokens → procesa 14,500 tokens
...
Petición 20: Contexto = 55,000 tokens → procesa 55,000 tokens

Total tokens procesados: ~350,000 tokens de input
Costo estimado (Sonnet): ~$1.05

SESIÓN CON CACHING (lo que realmente ocurre):
═══════════════════════════════════════════════════════════════

Petición 1: Procesa 10,000 → cachea los primeros 10,000 tokens
Petición 2: Lee 10,000 desde caché + procesa 2,000 nuevos
Petición 3: Lee 12,000 desde caché + procesa 2,500 nuevos
...
Petición 20: Lee 53,000 desde caché + procesa 2,000 nuevos

Total tokens procesados (sin caché): ~45,000
Tokens leídos de caché: ~305,000 (coste ~10% del normal)

Ahorro: ~75% del costo total de input tokens
```

### Qué se cachea específicamente

```
PREFIJOS QUE SE CACHEAN:
═══════════════════════════════════════════════════════════════

1. El system prompt (CLAUDE.md + instrucciones base)
   → Siempre el mismo en toda la sesión → siempre en caché

2. El historial de conversación hasta el punto estable
   → Petición N cachea los mensajes 1..N-1
   → Petición N+1 lee 1..N-1 desde caché y solo procesa el mensaje N
   
3. Definiciones de herramientas
   → Siempre las mismas → siempre en caché después de la 1ª petición
```

### Ver el ahorro por caché con /cost

```
> /cost

DESGLOSE DE TOKENS EN ESTA SESIÓN:
────────────────────────────────────────────
  Tokens input procesados:           4,830
  Tokens input desde caché:         48,210
  Tokens output generados:           9,120
  ─────────────────────────────────────────
  Total tokens input:               53,040
  
  Costo estimado (Sonnet):
  • Sin caché: $0.159
  • Con caché: $0.036  ← 77% de ahorro

  Tokens usados vs máximo:          53,040 / 200,000 (26.5%)
```

### Implicación práctica: el CLAUDE.md es "casi gratis"

Dado que el CLAUDE.md se cachea después de la primera petición, su costo real es mínimo. Vale la pena tener un CLAUDE.md detallado y completo: el overhead es básicamente nulo después de la primera petición de la sesión.

---

## 5.8 El comando /compact: comprimir sin perder el hilo

### Qué hace exactamente /compact

`/compact` es el comando más importante para gestionar sesiones largas. Hace lo siguiente:

```
PROCESO DE /compact PASO A PASO
═══════════════════════════════════════════════════════════════

ESTADO ANTES de /compact:
  Contexto: ~120,000 tokens (60% de 200k)
  Historial: 25 intercambios de mensajes, archivos leídos, outputs

PASO 1: Claude Code construye un payload especial:
  {
    "system": "Eres un asistente que resume conversaciones técnicas",
    "messages": [HISTORIAL COMPLETO DE 120,000 TOKENS],
    "user": "Resume esta conversación de trabajo en Claude Code.
             Incluye:
             1. Estado actual del proyecto (qué se completó, qué está en progreso)
             2. Archivos modificados y qué cambios se hicieron
             3. Decisiones técnicas importantes tomadas y por qué
             4. Tareas pendientes identificadas
             5. Contexto técnico crítico (stack, convenciones relevantes)
             
             Sé conciso pero completo. Máximo 3,000 tokens."
  }

PASO 2: La API genera el resumen (~3,000-5,000 tokens)

PASO 3: Claude Code REEMPLAZA el historial completo con el resumen
  El resumen se convierte en el ÚNICO mensaje "previo" del historial

ESTADO DESPUÉS de /compact:
  Contexto: ~8,000-12,000 tokens (4-6% de 200k)
  Historial: 1 mensaje de resumen que contiene lo esencial
```

### Qué preserva y qué pierde /compact

| Información | Se preserva ✓ | Se pierde ✗ |
|-------------|--------------|-------------|
| Estado actual del proyecto | ✓ | |
| Archivos modificados y qué cambios | ✓ | |
| Decisiones técnicas tomadas | ✓ | |
| Tareas completadas y pendientes | ✓ | |
| El texto exacto de los archivos leídos | | ✗ |
| Los outputs exactos de los comandos | | ✗ |
| El razonamiento detrás de cada decisión menor | | ✗ |
| Por qué se rechazaron opciones alternativas | | ✗ (a menos que lo pidas) |

### /compact con instrucciones personalizadas

Puedes añadir instrucciones específicas para el resumen:

```
/compact Asegúrate de incluir en el resumen:
- La decisión exacta sobre bcrypt vs argon2id y la justificación técnica
- Los parámetros exactos de bcrypt que elegimos (factor 12, UTF-8)
- El estado de cada archivo en tests/ (cuáles se actualizaron)
- Todos los errores de tests que encontramos y cómo los resolvimos
```

### Cuándo usar /compact vs. no usarlo

```
USA /compact CUANDO:                     NO USES /compact CUANDO:
  ✓ /cost > 65-70% de uso                 ✗ Necesitas el texto exacto de
  ✓ La sesión lleva >1h de trabajo           algo discutido hace poco
  ✓ Latencia notablemente mayor           ✗ Estás a punto de terminar
  ✓ Vas a continuar la misma tarea        ✗ La tarea está completada
  ✓ Claude empieza a "olvidar" cosas      ✗ Cambias de tarea completamente
                                            (usa /clear mejor)
```

---

## 5.9 El comando /clear: empezar desde cero

### Qué hace /clear exactamente

`/clear` borra **todo el historial de conversación** de la sesión actual. El contexto queda completamente vacío, exactamente igual que al inicio de la sesión.

**Qué NO borra /clear:**
- Los archivos que Claude modificó en disco → siguen modificados.
- El CLAUDE.md → sigue en disco y se recargará.
- Los commits de git realizados → permanentes.
- La configuración de la sesión (modelo elegido).

`/clear` es destructivo para la memoria de la sesión, pero no para el trabajo realizado en archivos.

### El flujo de decisión entre /compact y /clear

```
¿Vas a continuar trabajando en la MISMA TAREA?
                │
        ┌───────┴────────┐
        │                │
       SÍ               NO
        │                │
        ▼                ▼
    /compact           /clear
   (preserva         (borra todo,
   el hilo y        comienzo limpio
   el estado)       para nueva tarea)
```

**Usa /clear cuando:**
- Completaste la tarea actual y empiezas una completamente diferente.
- Claude tomó una dirección muy errónea y el historial "envenena" las siguientes respuestas.
- Inicio de una nueva jornada laboral (el historial del día anterior raramente es relevante).
- La tarea requiere que Claude "olvide" lo que aprendió (ej. análisis imparcial).

**Usa /compact cuando:**
- Continúas con la misma tarea pero el contexto está llegando al límite.
- Quieres liberar espacio sin perder el hilo de lo que estabas haciendo.
- Las respuestas se degradas en calidad y quieres "refrescar" la atención del LLM.

---

## 5.10 Estrategias proactivas de gestión del contexto

### Estrategia 1: Acotar el scope desde el principio

El scope del prompt determina cuánto contexto va a consumir Claude en exploración:

```
CONSUME MUCHO CONTEXTO — scope amplio:
  "Analiza el proyecto y sugiere mejoras generales"
  → Claude leerá README, estructura, docenas de archivos, ejecutará análisis...
  → Fácilmente 40,000-80,000 tokens solo en exploración

CONSUME POCO CONTEXTO — scope acotado:
  "Analiza la función hash_password en src/auth.py y sugiere
   mejoras de seguridad específicas"
  → Claude lee 1 archivo, posiblemente consulta 1-2 referencias
  → ~2,000-5,000 tokens total
```

La especificidad en los prompts no solo mejora la precisión de la respuesta, también reduce dramáticamente el consumo de contexto.

### Estrategia 2: Leer por rangos en lugar de archivos completos

Cuando necesitas información de un archivo grande, busca primero y luego lee solo lo necesario:

```python
# EN LUGAR DE:
Read("src/models/user_repository.py")  # 600 líneas → ~7,000 tokens

# MEJOR:
Grep("class UserRepository", "src/models/user_repository.py")
# → "Encontrado en línea 87"

Read("src/models/user_repository.py", start_line=85, end_line=150)
# → Solo 65 líneas → ~780 tokens
# AHORRO: ~6,200 tokens (88% menos)
```

### Estrategia 3: Limitar el output de comandos verbosos

```bash
# EN LUGAR DE (puede generar 50,000+ tokens):
git log --all

# MEJOR (controlado):
git log --all --oneline -20          # Solo los 20 últimos commits
git log --since="1 week ago" --oneline  # Solo la última semana

# EN LUGAR DE (output completo, verbose):
pytest tests/ -v

# MEJOR (resumido cuando solo te importa si pasa):
pytest tests/ -q                     # --quiet: mucho menos output
pytest tests/ --tb=short            # solo traceback corto en fallos
```

### Estrategia 4: Dividir tareas grandes en sesiones pequeñas

```
ENFOQUE DE SESIÓN ÚNICA (antipatrón):
  Sesión de 3 horas para "refactorizar todo el módulo de autenticación"
  → Contexto se llena, calidad se degrada, los últimos cambios son peores

ENFOQUE MULTI-SESIÓN (mejor práctica):
  Sesión 1 (~45 min): "Migra hash_password y verify_password a bcrypt"
                       → /clear al terminar
  Sesión 2 (~45 min): "Actualiza todos los tests relacionados con auth"
                       → /clear al terminar
  Sesión 3 (~30 min): "Actualiza la documentación del módulo de auth"
                       → /clear al terminar
  
  Cada sesión empieza con contexto limpio y máxima calidad.
  El trabajo es el mismo; la calidad de cada parte es mejor.
```

### Estrategia 5: Usar subagentes para tareas masivas

Para tareas que requieren procesar muchos archivos, los subagentes mantienen contextos independientes:

```
SIN SUBAGENTES (problema de contexto):
  Tarea: "Documenta los 40 módulos del proyecto"
  → Claude lee los 40 archivos → contexto: ~150,000 tokens
  → Módulos 30-40 reciben documentación de menor calidad (contexto casi lleno)

CON SUBAGENTES (calidad uniforme):
  Agente principal divide: 4 subagentes × 10 módulos cada uno
  → Subagente A: módulos 1-10, contexto fresco (~30,000 tokens)
  → Subagente B: módulos 11-20, contexto fresco (~30,000 tokens)
  → Subagente C: módulos 21-30, contexto fresco (~30,000 tokens)
  → Subagente D: módulos 31-40, contexto fresco (~30,000 tokens)
  
  Calidad uniforme para todos los módulos.
  (Ver Capítulo 6 para el sistema multi-agente completo)
```

### Estrategia 6: El CLAUDE.md como alternativa al historial

Todo lo que necesitas que Claude "recuerde" entre sesiones debería estar en el CLAUDE.md, no solo en el historial:

```markdown
# CLAUDE.md — Lo que siempre necesitas saber

## Decisiones técnicas importantes
- Auth: bcrypt con rounds=12 (decidido 2026-01-15, más seguro que rounds=10)
  Ver: docs/security/decisions.md para justificación completa
- DB: PostgreSQL 15 con pgvector para búsqueda semántica
- API: FastAPI con Pydantic v2 (NO v1 — incompatible)

## Convenciones del equipo
- Sin abreviaciones en nombres de variables/funciones
- Tests: 1 clase de test por módulo, fixtures en conftest.py
- Commits: Conventional Commits (feat: fix: docs: etc.)

## Estado del proyecto
- Módulos completos: auth, models, api/v1
- En progreso: api/v2 (endpoint de búsqueda semántica)
- Pendiente: módulo de analytics, integración con Stripe
```

Al tener esta información en CLAUDE.md, Claude la tiene disponible desde el inicio de cada sesión con máxima atención, sin necesidad de "explorar" o que tú repitas el contexto.

---

## 5.11 Optimización del costo: input vs output tokens

### La diferencia de precio entre input y output

En los modelos de Anthropic (a modo ilustrativo):
- **Input tokens (lo que envías):** Precio base.
- **Output tokens (lo que genera Claude):** 3-5x más caro que los input tokens.

Esta asimetría tiene implicaciones importantes para cómo trabajas con Claude Code.

### Dónde va realmente el costo en una sesión típica

```
DESGLOSE DE TOKENS EN SESIÓN TÍPICA DE 1 HORA
═══════════════════════════════════════════════════════════════

INPUT TOKENS:
  System prompt + CLAUDE.md (repetido cada petición): ~50,000
    → Mayormente en caché: costo real ~5,000 equivalentes
  Historial acumulado (archivos, outputs):            ~80,000
    → Mayormente en caché: costo real ~10,000 equivalentes
  Mensajes tuyos (prompts):                            ~2,000
  ─────────────────────────────────────────────────────
  Total input real (sin caché):                       ~17,000 tokens

OUTPUT TOKENS:
  Respuestas de Claude (texto):                        ~8,000
  Solicitudes de herramientas (tool_use JSONs):        ~2,000
  ─────────────────────────────────────────────────────
  Total output:                                        ~10,000 tokens

COSTO APROXIMADO:
  Input: 17,000 × $3/M tokens = $0.051
  Output: 10,000 × $15/M tokens = $0.15
  ─────────────────────────────────────────────────────
  TOTAL: ~$0.20 por hora de trabajo activo
```

*(Los precios son aproximados e ilustrativos; verificar precios actuales en console.anthropic.com)*

### Estrategias para reducir el costo

**Optimizar output:** Las respuestas largas son caras. Para tareas de ejecución (no de explicación), pide respuestas concisas:

```
EN LUGAR DE: "Corrige el bug en auth.py y explica detalladamente cada cambio"
MEJOR: "Corrige el bug en auth.py y haz un resumen breve de qué cambió"
```

**Usar el modelo correcto:**
- Haiku para tareas simples: 10-20x más barato que Opus.
- Sonnet para trabajo normal: 3-5x más barato que Opus.
- Opus solo cuando el problema realmente lo requiere.

**Evitar outputs grandes innecesarios:** Si Claude va a generar un archivo largo (ej. 500 líneas de código), ese archivo son output tokens caros. Asegúrate de que la tarea lo requiere.

---

## 5.12 Resumen y glosario del capítulo

### Los puntos fundamentales

1. **La ventana de contexto** (200,000 tokens) es la "mesa de trabajo" del LLM. Tiene tamaño fijo y todo lo de la sesión ocupa espacio en ella.

2. **Un token** es ~4 caracteres en inglés. Los archivos leídos, outputs de comandos, y respuestas de Claude se acumulan en el historial y permanecen en el contexto.

3. **El contexto se llena más rápido de lo intuitivo** porque los archivos leídos no "desaparecen" — permanecen en el historial toda la sesión.

4. **"Lost in the Middle"** es un fenómeno real: los LLMs prestan menos atención a información en el centro del contexto. La solución es `/compact` frecuente.

5. **Prompt Caching** reduce el costo en 70-90% automáticamente. El CLAUDE.md es "casi gratis" gracias al caching.

6. **`/compact`** resume el historial (~90% reducción de tokens) preservando el hilo. Úsalo a partir del 65-70% de uso del contexto.

7. **`/clear`** borra todo el historial. Para cambio completo de tarea o inicio de jornada.

8. Las mejores estrategias son: prompts acotados, lectura por rangos, sesiones pequeñas, subagentes para tareas masivas, y CLAUDE.md completo.

### Glosario del capítulo

| Término | Definición |
|---------|------------|
| **Ventana de contexto** | Cantidad máxima de tokens que el LLM puede procesar en una sola petición (~200k en Claude). |
| **Token** | Unidad básica de procesamiento del LLM. Aprox. 4 chars en inglés, 3.5 en español. |
| **BPE** | Byte-Pair Encoding. El algoritmo que convierte texto en tokens identificando patrones frecuentes. |
| **"Lost in the Middle"** | Fenómeno donde los LLMs prestan menos atención a información en el centro del contexto. |
| **Prompt Caching** | Mecanismo de Anthropic que cachea el prefijo del contexto entre peticiones para reducir costo. |
| **Input tokens** | Los tokens enviados al modelo (mensajes, historial, resultados de herramientas). Precio base. |
| **Output tokens** | Los tokens generados por el modelo (respuestas, solicitudes de herramientas). 3-5x más caros. |
| **/compact** | Comando que resume el historial de la sesión liberando ~90% del espacio de contexto. |
| **/clear** | Comando que borra todo el historial. Para cambio de tarea o inicio de jornada. |
| **/cost** | Comando que muestra el desglose de tokens y costo estimado de la sesión. |
| **Stateless** | Característica de los LLMs: sin memoria entre peticiones. El historial se envía explícitamente. |
| **70% rule** | Regla práctica: usar /compact cuando el contexto supera el 70% de la ventana disponible. |

---

## Ver también

- **[Capítulo 2](./cap-02-patron-react.md):** ReAct — cada iteración del ciclo añade tokens al contexto.
- **[Capítulo 3](./cap-03-arquitectura-componentes.md):** Arquitectura — el Context Manager que gestiona el historial.
- **[Capítulo 6](./cap-06-sistema-multiagente.md):** Multi-Agente — usar subagentes para mantener contextos frescos.
- **[Capítulo 11](./cap-11-performance-optimizacion.md):** Performance — estrategias avanzadas de optimización de costo y velocidad.

---

> 📌 Siguiente capítulo: [Cap. 6 — El Sistema Multi-Agente](./cap-06-sistema-multiagente.md)

### La analogía

Imagina que trabajas en un banco de carpintería. Tu mesa de trabajo tiene un tamaño fijo. Para hacer una pieza, necesitas tener a la vista los materiales, las herramientas, los planos y las piezas que ya tienes hechas.

Cuando la mesa está vacía (inicio de sesión), tienes todo el espacio del mundo. Conforme avanza el trabajo, la mesa se llena: los planos ya revisados, las piezas terminadas, los materiales usados.

Eventualmente, la mesa se llena. Y aquí está el problema: **cuando la mesa está llena, el carpintero empieza a perder de vista partes del trabajo**. Los planos de la primera hora quedan cubiertos bajo los materiales nuevos.

La **ventana de contexto** de Claude es esa mesa de trabajo. Tiene un tamaño fijo (200,000 tokens). Todo lo que ocurre en la sesión ocupa espacio. Cuando se llena, Claude empieza a "perder de vista" el trabajo hecho horas atrás.

`/compact` es como quitar materiales ya usados de la mesa y guardar solo las notas importantes de lo que se hizo. `/clear` es limpiar la mesa completamente para empezar una tarea nueva.

### Por qué no hacer la mesa más grande

La respuesta técnica: procesar el contexto tiene un costo computacional **cuadrático** (no lineal). Doblar el contexto no dobla el tiempo: lo multiplica por cuatro. La ventana de 200k tokens ya es una de las más grandes del mercado. Gestionarla bien es la clave para un uso eficiente.

---

## 5.2 Qué es un token: explicación detallada con ejemplos

### El problema de trabajar con caracteres o palabras

Los LLMs no procesan texto carácter a carácter (ineficiente) ni palabra a palabra (distribución desigual). Necesitan unidades de significado más grandes y frecuentes.

### La solución: Byte-Pair Encoding (BPE)

El algoritmo **BPE** analiza un corpus masivo de texto y encuentra los **substrings más frecuentes**, que se convierten en tokens. El resultado es un vocabulario de ~50,000-100,000 tokens que cubre palabras comunes, partes de palabras, símbolos y puntuación.

### Ejemplos concretos de tokenización

```
INGLÉS (muy eficiente, entrenado con más inglés):
  "Hello, world!"   → "Hello" | "," | " world" | "!"      = 4 tokens

ESPAÑOL (algo menos eficiente por tildes y ñ):
  "¡Hola, mundo!"   → "¡" | "Hola" | "," | " mundo" | "!" = 5 tokens

CÓDIGO PYTHON (muy eficiente):
  "def hash(p):"    → "def" | " hash" | "(p)" | ":"        = 4 tokens

PALABRAS TÉCNICAS LARGAS:
  "authentication"  → "auth" | "ent" | "ication"           = 3 tokens
  "bcrypt"          → "bc" | "rypt"                        = 2 tokens
  "cryptocurrency"  → "crypt" | "o" | "currency"           = 3 tokens

NÚMEROS:
  "123456789"       → "123" | "456" | "789"                = 3 tokens
```

### Reglas prácticas para estimar tokens

| Tipo de contenido | Ratio aproximado |
|-------------------|------------------|
| Texto en inglés | 1 token ≈ 4 caracteres |
| Texto en español | 1 token ≈ 3.5 caracteres |
| Código Python/JS/TS | 1 token ≈ 3-4 caracteres |
| JSON / YAML | 1 token ≈ 3 caracteres (muchos símbolos) |
| 1 línea de código (80 chars) | ≈ 20-25 tokens |
| 1 página de texto (500 palabras) | ≈ 700 tokens |
| 1 archivo Python de 200 líneas | ≈ 2,000-4,000 tokens |

### Por qué importan los tokens

Los tokens son la unidad de medida de **todo** lo que hace el LLM:
1. **La ventana de contexto** (la "memoria" de Claude) se mide en tokens: ~200,000 en Claude.
2. **El costo** se mide en tokens: pagas por cada token enviado (input) y recibido (output).
3. **La velocidad** depende de los tokens: más tokens en el contexto = más latencia por petición.

---

## 5.3 Los 200,000 tokens: qué significa en la práctica

### La escala en perspectiva

```
200,000 tokens es aproximadamente:

  150,000 palabras en inglés
  130,000 palabras en español
  600 páginas de libro en formato A4
  1,000 archivos Python de 100 líneas cada uno
  El texto completo de "El Señor de los Anillos" (trilogía completa)
  5-6 novelas cortas
```

En teoría, 200,000 tokens debería ser más que suficiente. En la práctica, el contexto se llena más rápido de lo esperado.

### Por qué el contexto se llena más rápido de lo esperado

**Los resultados de herramientas se acumulan:**
Cada archivo que Claude lee, cada output de comando que ejecuta, se añade al historial. No solo el archivo "en uso": todos los archivos que Claude ha leído en la sesión siguen en el contexto.

**El historial crece linealmente:**
Cada mensaje tuyo + cada respuesta de Claude + cada resultado de herramienta se añade. En una sesión de trabajo intenso, esto puede ser decenas de miles de tokens por hora.

**El sistema prompt es constante:**
El CLAUDE.md y las instrucciones base ocupan espacio en cada petición, aunque no crecen.

### El contador de tokens en tiempo real

```
> /cost

Uso de tokens en esta sesión:
  Tokens de entrada: 45,230
  Tokens de entrada (desde caché): 38,100
  Tokens de salida: 8,120
  Total: 53,350 / 200,000 (26.7%)
  
  Costo estimado hasta ahora: $0.27
```

---

## 5.4 Anatomía del contexto: qué ocupa qué

### El mapa de ocupación en una sesión típica

```
VENTANA DE CONTEXTO (200,000 tokens)
┌──────────────────────────────────────────────────────────────────┐
│ SISTEMA (estático — igual en cada petición)          ~1k-6k tok  │
│  - System prompt base de Claude Code                             │
│  - ~/.claude/CLAUDE.md (instrucciones globales)                  │
│  - ./CLAUDE.md (instrucciones del proyecto)                      │
├──────────────────────────────────────────────────────────────────┤
│ HISTORIAL DE CONVERSACIÓN (crece con cada intercambio)           │
│  - Mensaje usuario #1                              ~50-500 tok   │
│  - Respuesta Claude #1 (texto)                     ~100-800 tok  │
│  - tool_use: Read(auth.py)                          ~20-50 tok   │
│  - tool_result: [contenido de auth.py]          ~1,000-5,000 tok │
│  - tool_use: Bash(pytest tests/)                    ~20-30 tok   │
│  - tool_result: [output del pytest]              ~500-3,000 tok  │
│  - ...se repite por cada intercambio...                          │
├──────────────────────────────────────────────────────────────────┤
│ MENSAJE ACTUAL                                       ~50-500 tok │
└──────────────────────────────────────────────────────────────────┘
```

### Los grandes consumidores de contexto

**1. Archivos grandes leídos completamente**
```
Un archivo Python de 500 líneas:
  ~50 chars/línea × 500 = 25,000 chars ÷ 4 ≈ 6,250 tokens

Si Claude lee 10 archivos así: 10 × 6,250 = 62,500 tokens
(solo en archivos leídos)
```

**2. Outputs de comandos verbosos**
```
pytest con cobertura en proyecto mediano:  ~3,000-8,000 tokens
npm install con muchas dependencias:        ~2,000-5,000 tokens
git log --all --oneline (historial largo):  variable, puede ser enorme
```

**3. Respuestas largas de Claude**
```
Claude documentando 10 funciones:
  ~500 tokens/función × 10 = 5,000 tokens
  (añadidos al historial en rol "assistant")
```

---

## 5.5 Crecimiento del contexto durante una sesión real

### La progresión típica en una sesión de trabajo intenso

```
INICIO DE SESIÓN
  Tokens: ~3,000 (solo system + CLAUDE.md)
  Estado: mesa vacía, todo el espacio disponible

DESPUÉS DE 15 MINUTOS (exploración inicial)
  Claude lee: README, package.json, estructura del proyecto
  Tokens: ~15,000-25,000
  Estado: cómodo, sin problemas de rendimiento

DESPUÉS DE 1 HORA (trabajo activo)
  Claude ha leído 10-15 archivos, ejecutado 20 comandos
  Tokens: ~60,000-80,000
  Estado: bien, pero la latencia empieza a aumentar levemente

DESPUÉS DE 2 HORAS (sesión larga)
  Claude ha leído 25+ archivos, muchos intercambios
  Tokens: ~120,000-150,000
  Estado: latencia notable, calidad puede degradarse levemente
  → ACCIÓN RECOMENDADA: usar /compact

DESPUÉS DE 3 HORAS SIN /compact
  Tokens: ~170,000-190,000
  Estado: cerca del límite, calidad degradada
  RIESGO: error por exceder el límite en la siguiente petición
```

### Señales de que el contexto necesita atención

1. **Latencia notablemente mayor:** Las respuestas tardan el doble de lo habitual.
2. **Claude "olvida" cosas del inicio de la sesión:** Le preguntas sobre algo que se discutió hace 1 hora y dice no recordarlo.
3. **Respuestas menos coherentes:** Aparecen inconsistencias o repeticiones.
4. **`/cost` muestra >70% de uso.**
5. **Mensaje de advertencia de Claude Code:** "El contexto está cerca del límite."

---

## 5.6 Lost in the Middle: el fenómeno de atención

### El problema de atención en contextos largos

Investigaciones académicas han demostrado que los LLMs tienen dificultades para mantener atención uniforme en contextos muy largos. El problema se conoce como **"Lost in the Middle"** (perdido en el medio).

### La curva de atención del LLM

```
ATENCIÓN DEL LLM A LO LARGO DEL CONTEXTO:

Alta
 ▲
 │████                                                      ████
 │████                                                      ████
 │  ████                                                  ████
 │    ██████                                          ██████
 │          ████████████████████████████████████████
 │
 └─────────────────────────────────────────────────────────────►
    Inicio                    CENTRO                        Final
  (alta atención)          (baja atención                (alta atención)
                         = "lost in the middle")
```

El LLM presta más atención a:
- El **principio** del contexto (el system prompt, el CLAUDE.md).
- El **final** del contexto (los mensajes más recientes).
- Presta menos atención al **"medio"**: información añadida hace un rato pero que ya no es lo más reciente.

### Consecuencias prácticas

```
SESIÓN DE 2 HORAS:

  Hora 1: Claude lee src/auth.py (IMPORTANTE, contiene el bug)
  Hora 2: Claude lee 15 archivos más

  Al llegar a la hora 2, src/auth.py está en el "medio" del contexto.
  Claude puede "olvidar" detalles de auth.py aunque siga en el contexto.

  Síntoma: Claude hace un cambio que contradice algo que vio en auth.py
  porque ya no presta la misma atención a esa parte del contexto.
```

### La solución

El fenómeno "Lost in the Middle" es una razón más para usar `/compact` regularmente. Al comprimir el historial, se crea un resumen donde la información importante está **concentrada al principio**, donde el LLM presta máxima atención.

---

## 5.7 Prompt Caching: el mecanismo de ahorro automático

### Qué es

**Prompt Caching** es un mecanismo de Anthropic que permite **cachear** (guardar en memoria) el prefijo del contexto entre peticiones, evitando reprocesarlo.

### El ahorro que produce

Sin prompt caching, cada petición reprocesa todos los tokens anteriores:
```
Petición 1: Procesar 10,000 tokens → costo completo X
Petición 2: Procesar 12,000 tokens → costo completo X + algo
Petición 3: Procesar 14,000 tokens → costo completo X + más
```

Con prompt caching:
```
Petición 1: Procesar 10,000 tokens → costo completo X
            → Anthropic cachea los primeros 10,000 tokens

Petición 2: 10,000 tokens desde caché (coste ~10% de X) + 2,000 nuevos
Petición 3: 12,000 tokens desde caché + 2,000 nuevos
→ Ahorro del 80-90% en tokens de entrada para sesiones largas
```

### Cómo funciona técnicamente

Anthropic cachea automáticamente los prefijos del contexto que se repiten entre peticiones:
- El system prompt (siempre igual).
- El CLAUDE.md (siempre igual en una sesión).
- El historial de conversación (el prefijo estable crece petición a petición).

**El caching se activa automáticamente.** No requiere configuración por parte del usuario.

### Implicación para el CLAUDE.md

Dado que el CLAUDE.md se cachea, su contenido es **muy barato** en tokens después de la primera petición. Merece la pena invertir en un CLAUDE.md completo y detallado: el costo se amortiza rápidamente.

### Ver el uso del caché con /cost

```
> /cost

  Tokens de entrada sin caché:    2,340
  Tokens de entrada desde caché: 41,560
  Tokens de salida:               8,120
  
  Ahorro por caché: ~$0.37 (83% de los tokens de entrada desde caché)
```

---

## 5.8 El comando /compact: comprimir sin perder el hilo

### Qué hace exactamente

`/compact` es el comando más importante para gestionar el contexto en sesiones largas:

```
ANTES DEL /compact:
  Historial: [msg1][resp1+tools1][msg2][resp2+tools2]...[msg20][resp20]
  Tokens: ~90,000

PROCESO DE /compact:
  1. Claude Code envía todo el historial a la API con instrucción:
     "Resume esta conversación preservando:
      - Tareas completadas y su estado
      - Estado actual del proyecto y archivos modificados
      - Tareas pendientes
      - Decisiones técnicas importantes y justificación
      - Contexto técnico crítico (stack, convenciones)"

  2. El LLM genera el resumen (~3,000-5,000 tokens)

  3. El historial completo SE REEMPLAZA con ese resumen

DESPUÉS DEL /compact:
  Historial: [RESUMEN: "Hemos migrado auth.py a bcrypt. Archivos
              modificados: src/auth.py, tests/test_auth.py,
              requirements.txt. Todos los tests pasan.
              Pendiente: actualizar la documentación."]
  Tokens: ~6,000
```

### Qué se preserva y qué se pierde

| Se preserva | Se pierde |
|-------------|-----------|
| Estado del proyecto (qué archivos se modificaron) | Detalle de cada acción individual |
| Decisiones técnicas tomadas | Outputs exactos de los comandos |
| Contexto de la tarea | Texto exacto de los archivos que se leyeron |
| Tareas pendientes | Por qué se descartaron ciertas opciones |

### Cuándo usar /compact

```
USA /compact CUANDO:
  ✓ /cost muestra >70% de uso del contexto
  ✓ La sesión lleva más de 1 hora de trabajo activo
  ✓ Las respuestas se vuelven más lentas o menos cohesivas
  ✓ Quieres continuar trabajando en la misma tarea

NO USES /compact CUANDO:
  ✗ Necesitas que Claude tenga el texto exacto de algo discutido antes
  ✗ Estás a punto de terminar (no merece la pena)
  ✗ Cambias de tarea completamente (usa /clear)
```

### /compact con instrucciones personalizadas

```
/compact Asegúrate de incluir en el resumen: la razón exacta por la 
que decidimos usar bcrypt en lugar de argon2id, y los parámetros de 
configuración de bcrypt que elegimos.
```

---

## 5.9 El comando /clear: empezar desde cero

### Qué hace /clear

`/clear` borra **todo el historial de conversación** de la sesión actual. El contexto queda completamente vacío.

**Qué NO borra /clear:**
- Los archivos que Claude modificó (siguen modificados en disco).
- El CLAUDE.md (sigue en disco, se recargará).
- La configuración de la sesión (modelo elegido, etc.).
- Los commits de git que se hicieron.

### /compact vs. /clear: cuándo usar cada uno

```
¿Quieres continuar la MISMA TAREA?
       │
  SÍ ──┤── NO
       │         │
       ▼         ▼
   /compact    /clear
  (preserva  (borra todo,
  el hilo)    empezar nuevo)
```

**Prefiere /clear cuando:**
- Cambio completo de tarea (pasas de autenticación a base de datos).
- Claude tomó una dirección muy errónea y el historial contaminaría el nuevo trabajo.
- Inicio de nueva jornada laboral (el contexto del día anterior rara vez es relevante).

---

## 5.10 Estrategias proactivas de gestión del contexto

### Estrategia 1: Prompts acotados para reducir exploración

```
CONSUME MUCHO CONTEXTO (exploración amplia):
  "Analiza el proyecto y sugiere mejoras"
  → Claude leerá docenas de archivos = miles de tokens

CONSUME POCO CONTEXTO (scope acotado):
  "Analiza específicamente la función hash_password en src/auth.py
   y sugiere cómo mejorar su seguridad"
  → Claude lee 1 archivo = pocos tokens
```

### Estrategia 2: Leer solo lo necesario con rangos de líneas

```bash
# En lugar de leer 400 líneas:
Read("src/models/user.py")

# Busca primero para localizar, luego lee solo el fragmento:
Grep("class UserProfile", "src/models/user.py") → línea 87
Read("src/models/user.py", start_line=85, end_line=120)
```

### Estrategia 3: Dividir sesiones largas en tareas pequeñas

```
En lugar de: sesión de 3 horas para refactorizar todo el módulo de auth

Sesión 1 (45 min): Migrar hash_password a bcrypt → /clear
Sesión 2 (45 min): Actualizar tests de autenticación → /clear
Sesión 3 (30 min): Actualizar la documentación → /clear

Cada sesión empieza con contexto limpio, calidad consistente en todas.
```

### Estrategia 4: Usar subagentes para tareas masivas

```
Sin subagentes:
  Claude lee 50 archivos → contexto: ~150,000 tokens → calidad degradada

Con subagentes (Capítulo 6):
  Subagente A: archivos 1-10  → contexto fresco (~30k tokens)
  Subagente B: archivos 11-20 → contexto fresco (~30k tokens)
  ...calidad consistente para todos los archivos
```

### Estrategia 5: El CLAUDE.md como alternativa al historial

Si hay información que Claude necesita en cada sesión, ponla en el CLAUDE.md en lugar de repetirla:

```markdown
# CLAUDE.md

## Comandos frecuentes
- Tests: pytest tests/ -v --cov=src
- Lint: black src/ && flake8 src/
- Tipos: mypy src/

## Archivos importantes
- Modelos: src/models/ (User, Token, Session)
- Auth: src/auth.py (hash + verify con bcrypt factor 12)
- API: src/api/v1/

## Decisiones de arquitectura
- Usamos bcrypt con factor 12 (no argon2id — ver decisión 2026-01)
- JWT para sesiones (expira en 24h)
- Redis para caché de tokens revocados
```

Esto evita que Claude tenga que "explorar" al inicio de cada sesión, ahorrando tokens y tiempo.

---

## 5.11 Resumen y glosario

### Resumen

1. La **ventana de contexto** es la "mesa de trabajo" del LLM: 200,000 tokens de tamaño fijo.
2. Un **token** es ~4 caracteres en inglés. El código es eficiente; el YAML y el JSON algo menos.
3. El contexto se llena por la acumulación de historial de conversación, archivos leídos y outputs de comandos.
4. El fenómeno **"Lost in the Middle"** reduce la atención del LLM a información en el centro del contexto.
5. El **Prompt Caching** de Anthropic cachea el prefijo del contexto, reduciendo el costo en 80-90% para sesiones largas.
6. **`/compact`** resume el historial preservando lo esencial (~90% de reducción de tokens). Para continuar la misma tarea.
7. **`/clear`** borra todo el historial. Para cambio completo de tarea o inicio de jornada.
8. Las mejores estrategias: prompts acotados, lectura por rangos, sesiones pequeñas, subagentes para tareas masivas.

### Glosario

| Término | Definición |
|---------|------------|
| **Ventana de contexto** | La cantidad máxima de tokens que el LLM puede procesar en una sola petición (~200k). |
| **Token** | Unidad básica de procesamiento del LLM. ~4 caracteres en inglés. |
| **BPE** | Byte-Pair Encoding. El algoritmo que divide el texto en tokens. |
| **"Lost in the Middle"** | Fenómeno donde el LLM presta menos atención a información en el centro del contexto. |
| **Prompt Caching** | Mecanismo de Anthropic que cachea el prefijo del contexto para reducir costo y latencia. |
| **/compact** | Comando que resume el historial de la sesión, reduciendo tokens sin perder el hilo. |
| **/clear** | Comando que borra todo el historial de la sesión. |
| **/cost** | Comando que muestra el uso de tokens y costo estimado de la sesión. |
| **Stateless** | Propiedad de los LLMs: sin memoria entre peticiones. El historial se envía explícitamente. |

---

## Ver también

- **[Capítulo 2](./cap-02-patron-react.md):** ReAct — cada iteración del ciclo añade tokens al contexto.
- **[Capítulo 6](./cap-06-sistema-multiagente.md):** Multi-Agente — usar subagentes para mantener contextos frescos.
- **[Capítulo 11](./cap-11-performance-optimizacion.md):** Performance — estrategias avanzadas de optimización.

---

> 📌 Siguiente capítulo: [Cap. 6 — El Sistema Multi-Agente](./cap-06-sistema-multiagente.md)
