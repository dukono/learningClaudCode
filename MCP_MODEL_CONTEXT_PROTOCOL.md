# MCP — Model Context Protocol

> **Objetivo:** Entender qué es MCP, cómo funciona, cómo se define el "contrato", cómo son los mensajes JSON, qué campos son obligatorios, cómo responde el servidor y cómo se orquesta todo.

---

## 📚 Tabla de Contenidos

- [1. ¿Qué es MCP?](#1-qué-es-mcp)
- [2. ¿Dónde se ejecuta el modelo? ¿Y Copilot?](#2-dónde-se-ejecuta-el-modelo-y-copilot)
- [3. ¿Cómo busca ficheros y los modifica?](#3-cómo-busca-ficheros-y-los-modifica)
- [4. Claude Code vs Copilot](#4-claude-code-vs-copilot)
- [5. Llamadas a MongoDB desde un agente](#5-llamadas-a-mongodb-desde-un-agente)
- [6. ¿Qué es un servidor MCP?](#6-qué-es-un-servidor-mcp)
- [7. El "contrato" MCP — Protocolo JSON-RPC 2.0](#7-el-contrato-mcp--protocolo-json-rpc-20)
- [8. Campos obligatorios — Request](#8-campos-obligatorios--request)
- [9. Campos obligatorios — Response](#9-campos-obligatorios--response)
- [10. Múltiples llamadas: así recopila la información](#10-múltiples-llamadas-así-recopila-la-información)
- [11. Un servidor MCP es un servicio normal](#11-un-servidor-mcp-es-un-servicio-normal)
- [12. Ejemplo completo Spring Boot](#12-ejemplo-completo-spring-boot)
- [13. Resumen visual del flujo](#13-resumen-visual-del-flujo)

---

## 1. ¿Qué es MCP?

MCP (Model Context Protocol) es un **protocolo estándar abierto** creado por Anthropic para que modelos de IA puedan comunicarse con herramientas externas de forma estructurada.

- Define cómo el modelo **descubre** qué herramientas existen.
- Define cómo el modelo **llama** a esas herramientas.
- Define cómo el servidor **responde** con el resultado.

> Es un "contrato" basado en **JSON-RPC 2.0** sobre HTTP (o stdio).

---

## 2. ¿Dónde se ejecuta el modelo? ¿Y Copilot?

| Herramienta | ¿Dónde corre el modelo? | ¿Qué hace en local? |
|---|---|---|
| **Ollama** | En tu máquina (CPU/GPU local) | Todo: inferencia, memoria, carga | 
| **GitHub Copilot** | En servidores de Microsoft/OpenAI | Solo el plugin IDE que envía contexto |
| **Claude Code** | En servidores de Anthropic | Proceso local que orquesta herramientas |

**¿Por qué no ves saltos de CPU con Copilot?**
El modelo **nunca se ejecuta en tu máquina**. El plugin solo manda texto (fragmentos de código, contexto) a la API remota y recibe la respuesta. La inferencia ocurre en los servidores de Microsoft.

---

## 3. ¿Cómo busca ficheros y los modifica?

El modelo en remoto **no tiene acceso directo** a tu disco. El proceso local (plugin, CLI, agente) actúa como intermediario:

```
Tu disco  ←→  Proceso local (Copilot plugin / Claude Code CLI)  ←→  Modelo en remoto
```

**Búsqueda de ficheros:**
1. El proceso local ejecuta comandos del SO (`ls`, `find`, `grep`, lectura de ficheros).
2. Recoge el texto resultante.
3. Lo manda al modelo como parte del contexto (prompt).
4. El modelo responde con instrucciones (`editar línea X del fichero Y`).
5. El proceso local aplica los cambios en disco.

**El modelo nunca toca tu disco. Solo lee/escribe texto en el contexto.**

---

## 4. Claude Code vs Copilot

| Capacidad | GitHub Copilot | Claude Code |
|---|---|---|
| Completado de código inline | ✅ | ❌ (no inline en IDE) |
| Agente autónomo multi-paso | ⚠️ limitado | ✅ muy potente |
| Ejecutar comandos de terminal | ❌ | ✅ |
| Leer/escribir ficheros autónomamente | ⚠️ con Copilot Agent | ✅ |
| Llamar herramientas externas (MCP) | ⚠️ soporte básico | ✅ nativo |
| Funciona sin IDE (solo terminal) | ❌ | ✅ |
| Memoria de contexto larga | ⚠️ | ✅ (200k tokens) |
| Llamadas a bases de datos via MCP | ⚠️ | ✅ |

**Claude Code** es más un agente autónomo. **Copilot** es más asistente de escritura de código.

---

## 5. Llamadas a MongoDB desde un agente

**Flujo cuando dices: "busca los pedidos de la tienda X":**

```
Tú: "busca pedidos de la tienda principal"
        ↓
Modelo (remoto): entiende que "tienda principal" → storeId = "main-store"
        ↓
Modelo decide llamar tool: query_mongo({ collection: "orders", filter: { storeId: "main-store" } })
        ↓
Proceso local (Claude Code / MCP client): ejecuta la query real en Mongo
        ↓
Mongo devuelve: [ { orderId: 1, ... }, { orderId: 2, ... } ]  ← solo los resultados, no millones de docs
        ↓
Proceso local manda el RESULTADO (texto resumido) al modelo
        ↓
Modelo analiza y responde al usuario
```

**¿Se envían millones de datos al modelo?**
No. El proceso local ejecuta la query con **filtros y límites** (`LIMIT 100`, por ejemplo). Solo se manda al modelo el **resultado acotado**, no toda la base de datos.

**¿Necesito MCP para conectarme a Mongo?**
No es obligatorio. Claude Code puede ejecutar directamente un script Node.js o Python que conecte a Mongo. MCP es útil cuando quieres **exponer esa capacidad de forma reutilizable** para cualquier modelo/cliente.

---

## 6. ¿Qué es un servidor MCP?

Un servidor MCP es **cualquier servicio** (Spring Boot, Node.js, FastAPI, Go...) que:

1. **Expone** una lista de "tools" (herramientas) que puede ejecutar.
2. **Recibe** peticiones JSON-RPC con los parámetros de la herramienta a ejecutar.
3. **Responde** con el resultado en JSON-RPC.

> No es magia. Es HTTP + JSON con un formato concreto.

**Si lo despliegas en tu PC** → solo tú lo usas (o quien tenga acceso a tu red).  
**Si lo despliegas en un servidor público** → cualquier modelo/cliente compatible puede usarlo.

---

## 7. El "contrato" MCP — Protocolo JSON-RPC 2.0

MCP usa **JSON-RPC 2.0** como formato de mensajes. Hay 3 tipos de mensajes:

| Tipo | Descripción |
|---|---|
| `Request` | El cliente pide algo al servidor (tiene `id`) |
| `Response` | El servidor responde a un Request (tiene `id` + `result` o `error`) |
| `Notification` | Mensaje sin respuesta esperada (no tiene `id`) |

---

## 8. Campos obligatorios — Request

```json
{
  "jsonrpc": "2.0",        // OBLIGATORIO — siempre "2.0"
  "id": 1,                 // OBLIGATORIO en Request — número, string o null. Identifica la llamada
  "method": "tools/call",  // OBLIGATORIO — qué operación se quiere hacer
  "params": {              // OPCIONAL (pero casi siempre presente) — datos de entrada
    "name": "query_mongo",
    "arguments": {
      "collection": "orders",
      "filter": { "storeId": "main-store" },
      "limit": 10
    }
  }
}
```

### Métodos estándar MCP más importantes:

| Método | ¿Qué hace? |
|---|---|
| `initialize` | Handshake inicial — el cliente se presenta al servidor |
| `tools/list` | El modelo pregunta: "¿qué herramientas tienes?" |
| `tools/call` | El modelo llama a una herramienta concreta |
| `resources/list` | Lista recursos disponibles (ficheros, datos) |
| `resources/read` | Lee un recurso concreto |
| `prompts/list` | Lista prompts predefinidos que ofrece el servidor |
| `prompts/get` | Obtiene un prompt concreto |

---

## 9. Campos obligatorios — Response

### ✅ Respuesta exitosa (`result`):

```json
{
  "jsonrpc": "2.0",   // OBLIGATORIO
  "id": 1,            // OBLIGATORIO — mismo id que el Request
  "result": {         // OBLIGATORIO si éxito — NUNCA "params" en la respuesta
    "content": [
      {
        "type": "text",
        "text": "Encontrados 3 pedidos: [{ orderId: 1, total: 99.99 }, ...]"
      }
    ],
    "isError": false
  }
}
```

### ❌ Respuesta con error (`error`):

```json
{
  "jsonrpc": "2.0",   // OBLIGATORIO
  "id": 1,            // OBLIGATORIO — mismo id que el Request (o null si no se pudo parsear)
  "error": {          // OBLIGATORIO si fallo — NUNCA "result" y "error" a la vez
    "code": -32602,   // OBLIGATORIO — código numérico del error
    "message": "Parámetro 'collection' requerido",  // OBLIGATORIO
    "data": {         // OPCIONAL — info adicional del error
      "field": "collection"
    }
  }
}
```

### Códigos de error estándar JSON-RPC:

| Código | Significado |
|---|---|
| `-32700` | Parse error — JSON malformado |
| `-32600` | Invalid Request — falta `jsonrpc` o `method` |
| `-32601` | Method not found — el método no existe |
| `-32602` | Invalid params — parámetros incorrectos |
| `-32603` | Internal error — error interno del servidor |
| `-32000` a `-32099` | Errores de aplicación personalizados |

> **Regla de oro:** En la respuesta NUNCA hay `params`. Hay `result` (éxito) O `error` (fallo). Nunca los dos a la vez.

---

## 10. Múltiples llamadas: así recopila la información

**Una sola interacción del usuario puede generar VARIAS llamadas MCP.** El modelo decide cuántas y en qué orden. Ejemplo:

```
Usuario: "Dame un resumen de los pedidos pendientes de la tienda Madrid y compáralo con el mes anterior"

1ª llamada MCP:
  → tools/call: query_mongo({ collection: "orders", filter: { storeId: "madrid", status: "pending", month: "marzo" } })
  ← resultado: 45 pedidos, total 12.300€

2ª llamada MCP:
  → tools/call: query_mongo({ collection: "orders", filter: { storeId: "madrid", status: "pending", month: "febrero" } })
  ← resultado: 38 pedidos, total 9.800€

3ª llamada MCP (opcional):
  → tools/call: get_store_info({ storeId: "madrid" })
  ← resultado: { name: "Tienda Madrid Centro", manager: "Ana García" }

Modelo procesa los 3 resultados juntos y responde al usuario con el análisis.
```

**El modelo es el orquestador.** Decide qué herramientas llamar, en qué orden y con qué parámetros, basándose en lo que necesita para responder.

---

## 11. Un servidor MCP es un servicio normal

**Sí, exactamente.** Un servidor MCP es simplemente una aplicación que:

- Puede estar en **Spring Boot**, Node.js, FastAPI, Go, Rust...
- Recibe peticiones **HTTP POST** (o por stdio) con JSON-RPC
- Tiene lógica de negocio normal (conectar a Mongo, llamar a otra API, leer ficheros...)
- Devuelve JSON-RPC con el resultado

### Lo que el servidor MCP NO hace:
- No interpreta lenguaje natural.
- No decide qué herramienta llamar.
- No sabe nada del contexto del usuario.

### Lo que el modelo hace:
- Interpreta "tienda Madrid" → `storeId: "madrid"`.
- Decide qué herramienta llamar y con qué parámetros.
- Analiza los resultados y construye la respuesta.

**El servidor MCP solo recibe datos estructurados y devuelve datos estructurados.**

---

## 12. Ejemplo completo Spring Boot

### Paso 1 — El servidor describe sus herramientas (tools/list)

**Request del modelo:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list"
}
```

**Response del servidor Spring Boot:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [
      {
        "name": "query_orders",
        "description": "Consulta pedidos en MongoDB con filtros opcionales",
        "inputSchema": {
          "type": "object",
          "properties": {
            "storeId": {
              "type": "string",
              "description": "ID de la tienda"
            },
            "status": {
              "type": "string",
              "enum": ["pending", "completed", "cancelled"],
              "description": "Estado del pedido"
            },
            "limit": {
              "type": "integer",
              "description": "Máximo de resultados (default 20)",
              "default": 20
            }
          },
          "required": ["storeId"]
        }
      },
      {
        "name": "get_store_info",
        "description": "Obtiene información de una tienda por su ID",
        "inputSchema": {
          "type": "object",
          "properties": {
            "storeId": {
              "type": "string",
              "description": "ID de la tienda"
            }
          },
          "required": ["storeId"]
        }
      }
    ]
  }
}
```

### Paso 2 — El modelo llama a una herramienta (tools/call)

**Request del modelo:**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "query_orders",
    "arguments": {
      "storeId": "madrid",
      "status": "pending",
      "limit": 5
    }
  }
}
```

**Response del servidor:**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Encontrados 45 pedidos pendientes. Mostrando 5:\n- Pedido #1001: 199.99€\n- Pedido #1002: 49.50€\n- Pedido #1003: 320.00€\n- Pedido #1004: 15.99€\n- Pedido #1005: 89.00€\nTotal parcial: 674.48€"
      }
    ],
    "isError": false
  }
}
```

### Paso 3 — Controlador Spring Boot que recibe esto

```java
@RestController
@RequestMapping("/mcp")
public class McpController {

    @Autowired
    private OrderService orderService;

    @PostMapping
    public ResponseEntity<Map<String, Object>> handle(@RequestBody Map<String, Object> request) {
        String method = (String) request.get("method");
        Object id = request.get("id");

        return switch (method) {
            case "tools/list" -> ResponseEntity.ok(buildToolsList(id));
            case "tools/call" -> handleToolCall(id, request);
            default -> ResponseEntity.ok(buildMethodNotFound(id, method));
        };
    }

    private Map<String, Object> handleToolCall(Object id, Map<String, Object> request) {
        Map<String, Object> params = (Map<String, Object>) request.get("params");
        String toolName = (String) params.get("name");
        Map<String, Object> arguments = (Map<String, Object>) params.get("arguments");

        try {
            String resultText = switch (toolName) {
                case "query_orders" -> orderService.queryOrders(arguments);
                case "get_store_info" -> orderService.getStoreInfo(arguments);
                default -> throw new IllegalArgumentException("Tool no encontrada: " + toolName);
            };

            return Map.of(
                "jsonrpc", "2.0",
                "id", id,
                "result", Map.of(
                    "content", List.of(Map.of("type", "text", "text", resultText)),
                    "isError", false
                )
            );
        } catch (Exception e) {
            return Map.of(
                "jsonrpc", "2.0",
                "id", id,
                "error", Map.of(
                    "code", -32603,
                    "message", e.getMessage()
                )
            );
        }
    }
}
```

---

## 13. Resumen visual del flujo

```
┌─────────────┐      texto natural      ┌──────────────────┐
│   Usuario   │ ─────────────────────→  │  Modelo (remoto) │
│             │ ←─────────────────────  │  Claude/GPT/etc  │
└─────────────┘    respuesta en texto   └────────┬─────────┘
                                                 │
                                    JSON-RPC: tools/list
                                    JSON-RPC: tools/call x N
                                                 │
                                        ┌────────▼─────────┐
                                        │  Proceso local   │
                                        │  (Claude Code /  │
                                        │   MCP Client)    │
                                        └────────┬─────────┘
                                                 │
                                          HTTP POST JSON-RPC
                                                 │
                                        ┌────────▼─────────┐
                                        │  Servidor MCP    │
                                        │  (Spring Boot /  │
                                        │   Node / FastAPI)│
                                        └────────┬─────────┘
                                                 │
                                          query real
                                                 │
                                        ┌────────▼─────────┐
                                        │    MongoDB /     │
                                        │    PostgreSQL /  │
                                        │    API externa   │
                                        └──────────────────┘
```

**Flujo de una petición:**
1. Usuario escribe texto natural.
2. Modelo interpreta y llama `tools/list` para saber qué puede hacer.
3. Modelo llama `tools/call` **una o varias veces** con parámetros estructurados.
4. El proceso local (MCP client) reenvía esas llamadas al servidor MCP vía HTTP.
5. El servidor MCP ejecuta la lógica (query a Mongo, etc.) y responde con JSON-RPC.
6. El proceso local devuelve el resultado al modelo.
7. El modelo analiza todos los resultados y construye la respuesta final.
8. El usuario recibe la respuesta en lenguaje natural.

---

## Resumen final

| Concepto | Realidad |
|---|---|
| Servidor MCP | Es un servicio HTTP normal (Spring, Node, etc.) |
| El modelo interpreta el texto | Sí, el modelo decide qué tool llamar y con qué args |
| Número de llamadas | Puede ser 1 o N, el modelo decide |
| Se envían millones de docs al modelo | No, solo el resultado acotado de la query |
| Campos obligatorios en Request | `jsonrpc`, `id`, `method` |
| Campos obligatorios en Response OK | `jsonrpc`, `id`, `result` |
| Campos obligatorios en Response Error | `jsonrpc`, `id`, `error` |
| "params" en la respuesta | ❌ NUNCA. Solo en el Request |
| El servidor entiende lenguaje natural | ❌ Solo recibe/devuelve datos estructurados |

