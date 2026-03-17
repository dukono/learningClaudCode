# CLAUDE CODE: ARQUITECTURA Y FUNCIONAMIENTO TÉCNICO
## Guía Completa — Índice Maestro

> Esta guía está organizada como un libro técnico. Cada capítulo puede leerse de forma independiente, aunque se recomienda seguir el orden para una comprensión progresiva. No se asumen conocimientos previos sobre agentes de IA o arquitecturas de LLM.

---

## 📚 CAPÍTULOS

| Nº | Título | Descripción breve |
|----|--------|-------------------|
| [01](./cap-01-que-es-claude-code.md) | ¿Qué es Claude Code Internamente? | Del chatbot al agente autónomo. Conceptos fundamentales. |
| [02](./cap-02-patron-react.md) | El Patrón ReAct: Reason + Act | El ciclo de razonamiento y acción que impulsa al agente. |
| [03](./cap-03-arquitectura-componentes.md) | Arquitectura de Componentes | Los módulos internos y cómo se comunican. |
| [04](./cap-04-sistema-herramientas.md) | El Sistema de Herramientas (Tools) | Cada herramienta, cómo funciona y cuándo usarla. |
| [05](./cap-05-ventana-contexto.md) | La Ventana de Contexto | Tokens, memoria de trabajo y gestión del contexto. |
| [06](./cap-06-sistema-multiagente.md) | El Sistema Multi-Agente | Paralelismo, subagentes y arquitecturas complejas. |
| [07](./cap-07-mcp-protocolo.md) | MCP: Arquitectura del Protocolo | Cómo Claude se conecta a cualquier fuente de datos. |
| [08](./cap-08-sistema-hooks.md) | El Sistema de Hooks | Automatización y eventos del ciclo de vida. |
| [09](./cap-09-seguridad-permisos.md) | Seguridad: Permisos y Confianza | Capas de seguridad, prompt injection y CI/CD seguro. |
| [10](./cap-10-ciclo-vida-sesion.md) | Ciclo de Vida de una Sesión | De `claude` a Ctrl+D, paso a paso. |
| [11](./cap-11-performance-optimizacion.md) | Performance y Optimización | Velocidad, costos y estrategias avanzadas. |

---

## 🗺️ MAPA DE DEPENDENCIAS ENTRE CAPÍTULOS

```
Cap 01 (¿Qué es?)
    └──► Cap 02 (ReAct)
              └──► Cap 03 (Componentes)
                        ├──► Cap 04 (Herramientas)
                        │         └──► Cap 06 (Multi-Agente)
                        │         └──► Cap 07 (MCP)
                        ├──► Cap 05 (Contexto)
                        │         └──► Cap 11 (Performance)
                        └──► Cap 10 (Ciclo de Vida)
                                  ├──► Cap 08 (Hooks)
                                  └──► Cap 09 (Seguridad)
```

---

## 💡 CÓMO USAR ESTA GUÍA

- **Si eres nuevo en Claude Code:** lee los capítulos del 01 al 05 en orden.
- **Si quieres entender el multi-agente:** lee el 06, que requiere el 04 y el 05.
- **Si quieres extender Claude con MCP:** lee el 07 directamente.
- **Si quieres automatizar con hooks:** lee el 08.
- **Si quieres usar Claude en producción/CI:** lee el 09 y el 10.
- **Si quieres reducir costos:** lee el 11.

---

> 📌 Referencia: Marzo 2026. Documentación oficial: https://docs.anthropic.com/claude-code

