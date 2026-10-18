# arquitectura

Laboratorio de aprendizaje: **orquestación de IA multi-agente** con Java y Spring.

## Stack

- **Java 25** (LTS) + **Spring Boot 4.1.1** + **Maven 3.10.0**
- **Spring AI** (fases avanzadas)
- **DeepSeek** como modelo, en dos modos intercambiables:
  - Local: **Ollama** con `deepseek-r1:1.5b`
  - Nube: **API DeepSeek** con `deepseek-chat` (V3)
- **LangChain4j** (solo para comparar, Fase 9)

## Idea clave

Ollama y la API de DeepSeek hablan el **mismo dialecto** (compatible con OpenAI).
Cambiar de local a nube es cambiar configuración, no código.

| | Endpoint | Modelo | Clave |
|---|---|---|---|
| Local | `http://localhost:11434/v1` | `deepseek-r1:1.5b` | — |
| Nube | `https://api.deepseek.com/v1` | `deepseek-chat` | `DEEPSEEK_API_KEY` |

## Hoja de ruta

11 fases, con teoría y ejercicios en cada una: **[docs/plan-detallado.md](docs/plan-detallado.md)**

> **Para retomar el trabajo con otro agente o persona:** leer **[AGENTS.md](AGENTS.md)**
> (contexto, entorno, decisiones, estado y próximos pasos).

| # | Fase |
|---|---|
| 0 | Entorno y esqueleto |
| 1 | "Hola DeepSeek" con HTTP crudo |
| 2 | Capa de modelo intercambiable |
| 3 | Spring AI |
| 4 | Prompting, roles y streaming |
| 5 | Tool calling |
| 6 | Bucle agéntico (ReAct) |
| 7 | Memoria y estado |
| 8 | Multi-agente y orquestador |
| 9 | Framework y comparación |
| 10 | Producción |

## Requisitos

- **Java 25** (LTS) — `java -version` (también funciona con Java 21)
- **Maven 3.10.0** — `mvn -v` (o usar el wrapper `.\mvnw.cmd`)
- **Ollama** con el modelo: `ollama pull deepseek-r1:1.5b`

## Cómo ejecutar

```powershell
# con el wrapper (recomendado, no requiere Maven instalado)
.\mvnw.cmd spring-boot:run

# o con Maven instalado
mvn spring-boot:run
```

Comprobar que arranca: `http://localhost:8080/actuator/health` → `{"status":"UP"}`

## Estado

| # | Fase | Estado |
|---|---|---|
| 0 | Entorno y esqueleto | ✅ |
| 1 | "Hola DeepSeek" con HTTP crudo | ⬜ |
| 2 | Capa de modelo intercambiable | ⬜ |
| 3 | Spring AI | ⬜ |
| 4 | Prompting, roles y streaming | ⬜ |
| 5 | Tool calling | ⬜ |
| 6 | Bucle agéntico (ReAct) | ⬜ |
| 7 | Memoria y estado | ⬜ |
| 8 | Multi-agente y orquestador | ⬜ |
| 9 | Framework y comparación | ⬜ |
| 10 | Producción | ⬜ |
