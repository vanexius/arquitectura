# Plan detallado — Orquestación de IA multi-agente (Java / Spring)

Laboratorio de aprendizaje **paso a paso**. Cada fase termina en un **commit** y en algo
que funciona y puedes probar. El objetivo final: un **orquestador multi-agente** en
Spring Boot que usa DeepSeek.

## Decisiones tomadas

| Tema | Decisión |
|---|---|
| Tema | Agentes con LLM (multi-agente) |
| Lenguaje | Java 21 + Spring Boot 3 + Maven |
| Modo | Híbrido: primero el "motor" a mano, luego un framework |
| Modelo | DeepSeek — **ruta híbrida** |
| Local | Ollama con `deepseek-r1:1.5b` |
| Nube (si va lento) | API DeepSeek con `deepseek-chat` (V3) |

## Idea clave: mismo dialecto

Ollama y la API de DeepSeek son **compatibles con la API de OpenAI**. Cambiar de local a
nube será solo cambiar configuración, no código:

| | Endpoint | Modelo | Clave |
|---|---|---|---|
| Local | `http://localhost:11434/v1` | `deepseek-r1:1.5b` | — |
| Nube | `https://api.deepseek.com/v1` | `deepseek-chat` | `DEEPSEEK_API_KEY` |

## Cómo trabajaremos

- Cada fase = 1 rama/commit con mensaje `Fase N: ...`.
- Por fase: **objetivo → teoría → ejercicios (guiados + reto) → entregable → commit**.
- Los ejercicios con ⭐ son el núcleo de la orquestación.
- Verificamos siempre ejecutando, no solo leyendo.

---

## Fase 0 — Entorno y esqueleto

**Objetivo:** tener todo listo para escribir código.

**Teoría**
- Qué es servir un LLM en local (Ollama) y qué es un modelo "pensante" (`thinking`).
- Qué es la API compatible con OpenAI (endpoint, `messages`, `choices`).
- Estructura de un proyecto Spring Boot.

**Ejercicios**
1. Instalar Ollama (`winget install Ollama.Ollama`) y comprobar `ollama --version`.
2. Descargar el modelo: `ollama pull deepseek-r1:1.5b`.
3. Probarlo en consola: `ollama run deepseek-r1:1.5b "¿Quién eres?"`. Observa el bloque
   de razonamiento (thinking) antes de la respuesta.
4. Mirar la API a mano: abre `http://localhost:11434/api/tags` en el navegador.
5. Poner Maven en el `PATH` y comprobar `mvn -v`.
6. Crear el proyecto Spring Boot (Web) y arrancarlo. Debe responder en `http://localhost:8080/`.

**Entregable:** el proyecto arranca; Ollama responde.
**Commit:** `Fase 0: entorno y esqueleto Spring Boot`

---

## Fase 1 — "Hola DeepSeek" con HTTP crudo

**Objetivo:** hablar con el LLM desde Java **sin frameworks**, para entender el protocolo.

**Teoría**
- Estructura del request: `model`, `messages[]` con `role`/`content`, `temperature`.
- Estructura del response: `choices[0].message.content`, `usage` (tokens).

**Ejercicios**
1. Con `RestClient` (Spring 6.1), haz `POST` a `{base}/chat/completions` con un mensaje.
2. Imprime la respuesta y explora **el JSON completo** (usa `Map` o `JsonNode`).
3. Cambia `temperature` (0, 0.5, 1) y compara.
4. Envía un `system` + un `user`.
5. ⭐ Reto: envía un array `messages` con historial de dos turnos.

**Entregable:** `GET/POST /chat` que recibe texto y devuelve la respuesta del modelo.
**Commit:** `Fase 1: cliente HTTP crudo contra Ollama`

---

## Fase 2 — Capa de modelo intercambiable ⭐

**Objetivo:** poder cambiar local ↔ API sin tocar los agentes.

**Teoría**
- Inversión de dependencias e interfaces.
- DTOs de request/response.
- Perfiles de Spring y variables de entorno.

**Ejercicios**
1. Define `interface ModelClient { String chat(List<Message> msgs); }` y sus DTOs.
2. Implementa `OpenAiCompatibleModelClient` parametrizado por `base-url`, `model`, `api-key`.
3. Crea dos perfiles: `application-local.yml` (Ollama) y `application-api.yml` (DeepSeek).
4. Arranca el mismo endpoint con `--spring.profiles.active=local` y luego `=api`.
5. ⭐ Reto: que la respuesta indique **qué proveedor** contestó.

**Entregable:** `/chat` funciona con ambos perfiles; el cambio es solo configuración.
**Commit:** `Fase 2: capa de modelo intercambiable`

---

## Fase 3 — Spring AI

**Objetivo:** reemplazar el cliente a mano por Spring AI y comparar.

**Teoría**
- `ChatModel` / `ChatClient`, auto-configuración y starters.
- `base-url` para apuntar al endpoint compatible.

**Ejercicios**
1. Añade el starter de OpenAI (compatible) de Spring AI.
2. Configura `base-url` hacia Ollama.
3. Reescribe `/chat` con `ChatClient`.
4. Compara el código antes (Fase 1/2) y después: ¿menos líneas? ¿más claro?
5. ⭐ Reto: dos beans `ChatClient` (local y API) elegidos por perfil.

**Entregable:** `/chat` usando Spring AI.
**Commit:** `Fase 3: migración a Spring AI`

---

## Fase 4 — Prompting, roles y streaming

**Objetivo:** controlar el comportamiento y el formato de salida.

**Teoría**
- `system prompt`, *few-shot*, `temperature`, `top_p`, límites de tokens.
- Streaming (SSE) y cómo se ve el *thinking* de R1.

**Ejercicios**
1. Escribe un system prompt que fije persona, idioma y formato.
2. Prueba *few-shot* con dos ejemplos.
3. Pide salida **JSON** y valídala.
4. Implementa streaming: `/chat/stream` que emita trozos (`Flux` / SSE).
5. ⭐ Reto: mide latencia y tokens **con** y **sin** streaming.

**Entregable:** `/chat/stream` y `/chat/json`.
**Commit:** `Fase 4: prompting, JSON y streaming`

---

## Fase 5 — Tool calling

**Objetivo:** que el modelo invoque **métodos Java**.

**Teoría**
- Function/tool calling: JSON schema de la herramienta, `tool_calls`, mensaje `role=tool`.

**Ejercicios**
1. Define dos herramientas: calculadora (`sumar`, `multiplicar`) y `horaActual`.
2. Envía `tools` (JSON schema) **a mano** en el request.
3. Detecta `tool_calls`, ejecuta el método Java, devuelve el resultado como `role=tool`
   y vuelve a llamar al modelo.
4. Rehazlo con Spring AI (`@Tool` + `.tools()`) y compara.
5. ⭐ Reto: herramienta que **lea un archivo** de texto del proyecto.

**Entregable:** `/chat` resuelve "¿cuánto es 123*45?" usando la herramienta.
**Commit:** `Fase 5: tool calling`

---

## Fase 6 — Bucle agéntico (ReAct) ⭐

**Objetivo:** construir el **motor de agente** de verdad.

**Teoría**
- Patrón ReAct: *Thought → Action → Observation*.
- Límite de iteraciones, condición de parada, errores de herramienta.

**Ejercicios**
1. Implementa `Agent` con bucle: prompt → si hay `tool_calls` ejecuta y repite; si no,
   devuelve la respuesta final.
2. Añade límite de iteraciones y **traza** de cada paso.
3. Pregunta multi-paso: *"¿Cuánto es (12*8)+45 y qué hora es?"* (usa dos herramientas).
4. Haz que una herramienta falle y comprueba que el agente se recupera.
5. ⭐ Reto: fuerza el formato *Thought/Action/Observation* en el prompt y **parsea tú** la respuesta.

**Entregable:** `/agent` que resuelve tareas multi-paso con herramientas.
**Commit:** `Fase 6: bucle agéntico ReAct`

---

## Fase 7 — Memoria y estado

**Objetivo:** que el agente **recuerde** entre turnos.

**Teoría**
- Memoria a corto plazo (historial) vs *scratchpad* del agente.
- Compresión por resumen; memoria a largo plazo (vectorial / BD).

**Ejercicios**
1. Mantén el historial por `sessionId` en un mapa.
2. Pregunta de seguimiento: *"y multiplícalo por 3"* tras una operación.
3. Historial largo → implementa ventana + resumen.
4. (Opcional) Persiste en fichero o H2.
5. ⭐ Reto: separa **memoria de trabajo** (scratchpad) del **historial de chat**.

**Entregable:** conversaciones con memoria por sesión.
**Commit:** `Fase 7: memoria y estado`

---

## Fase 8 — Multi-agente y orquestador ⭐

**Objetivo:** varios especialistas coordinados por un orquestador.

**Teoría**
- Patrones **router / supervisor / handoff**.
- Agentes como herramientas; delegación; traza completa.

**Ejercicios**
1. Crea tres agentes: `Investigador` (herramientas de ficheros), `Calculador`
   (matemáticas) y `Redactor` (sin herramientas, formatea).
2. Orquestador **router**: clasifica la petición y elige agente.
3. Orquestador **supervisor**: delega y encadena (*research → calc → writer*).
4. Traza completa: quién hizo qué en cada paso.
5. ⭐ Reto: *handoff* — el investigador pasa datos al redactor.

**Entregable:** `/orchestrator` que resuelve una tarea que requiere varios agentes.
**Commit:** `Fase 8: orquestador multi-agente`

---

## Fase 9 — Framework y comparación

**Objetivo:** ver cómo lo hacen las librerías y contrastarlo con tu motor.

**Teoría**
- Spring AI *advisors* / LangChain4j *AI Services*; grafos tipo LangGraph.
- Qué se gana (velocidad, robustez) y qué se pierde (control, comprensión).

**Ejercicios**
1. Reimplementa el orquestador con Spring AI o LangChain4j.
2. Compara líneas de código, control y depuración.
3. Escribe una nota en `docs/` con la comparación.
4. ⭐ Reto: la misma tarea por los dos caminos; compara respuesta y latencia.

**Entregable:** implementación con framework + nota comparativa.
**Commit:** `Fase 9: orquestador con framework`

---

## Fase 10 — Producción

**Objetivo:** dejarlo presentable y medible.

**Teoría**
- API REST limpia, validación, *timeouts* y *retries*.
- Observabilidad (logs/trazas) y **evaluación** (evals).

**Ejercicios**
1. Endpoint REST con validación y manejo de errores.
2. Timeouts y reintentos contra el modelo.
3. Métricas: latencia, tokens y nº de iteraciones del agente.
4. Mini *eval*: 10 preguntas con respuesta esperada y un script que las corre.
5. ⭐ Reto: README final con diagrama + demo.

**Entregable:** demo funcional + README con arquitectura.
**Commit:** `Fase 10: API REST, observabilidad y evaluación`

---

## Estado de progreso

- [ ] Fase 0 — Entorno y esqueleto
- [ ] Fase 1 — "Hola DeepSeek" con HTTP crudo
- [ ] Fase 2 — Capa de modelo intercambiable
- [ ] Fase 3 — Spring AI
- [ ] Fase 4 — Prompting, roles y streaming
- [ ] Fase 5 — Tool calling
- [ ] Fase 6 — Bucle agéntico (ReAct)
- [ ] Fase 7 — Memoria y estado
- [ ] Fase 8 — Multi-agente y orquestador
- [ ] Fase 9 — Framework y comparación
- [ ] Fase 10 — Producción
