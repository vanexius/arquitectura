# AGENTS.md — Traspaso del proyecto a otro agente

> **LÉEME PRIMERO.** Este archivo es la fuente de verdad para que otro agente (o una
> persona) retome el trabajo sin contexto previo. Léelo completo antes de tocar nada,
> y después `docs/plan-detallado.md` y `README.md`.

---

## 1. Qué es este proyecto

Laboratorio de aprendizaje: **orquestación de IA multi-agente** en **Java + Spring**,
usando **DeepSeek** como modelo, en dos modos intercambiables:

- **Local:** Ollama con `deepseek-r1:1.5b` (gratis, por CPU).
- **Nube:** API de DeepSeek con `deepseek-chat` (si lo local va lento).

La meta final (Fase 8) es un **orquestador multi-agente** en Spring Boot.
El plan tiene 11 fases (0–10), cada una con teoría y ejercicios.

## 2. Repositorio

| Elemento | Valor |
|---|---|
| Remoto | `https://github.com/vanexius/arquitectura` |
| Ruta local | `C:\VANE\MisProyectos` |
| Rama de trabajo | `main` |
| `origin` | `https://vanexius@github.com/vanexius/arquitectura.git` |
| Identidad git **local** de este repo | `vanexius <212634073+vanexius@users.noreply.github.com>` |
| Autenticación | Git Credential Manager (ya autenticado para `vanexius`) |

### ⚠️ Reglas de git importantes

- La configuración **global** del PC pertenece a **otra cuenta** (`Maxi-Montenegro`).
  **NO** modificar la config global. Este repo usa config **local** (ya está puesta).
- **Nunca** commitear secretos: claves de API, `.env`, tokens. `.env` ya está en `.gitignore`.

## 3. Hardware y limitaciones (¡importante!)

| Recurso | Valor | Implicación |
|---|---|---|
| RAM | **7,4 GB** | Justo: usar modelos pequeños (1.5b–3b). No correr Ollama + IDE + navegador a la vez |
| GPU | Intel HD 4600 | **Sin CUDA** → Ollama corre en **CPU**, lento |
| Disco libre | 85 GB | Suficiente |
| SO | Windows 10 (PowerShell) | Los comandos de ejemplo son PowerShell |

## 4. Entorno (versiones y rutas)

| Herramienta | Versión | Ruta / nota |
|---|---|---|
| Java | **`jdk-25.0.4.1` (25 LTS)** operativo | `C:\Program Files\Java\jdk-25.0.4.1` |
| `JAVA_HOME` (usuario) | `C:\Program Files\Java\jdk-25.0.4.1` | ✔ configurado |
| Maven | **3.10.0** | `C:\VANE\apache-maven-3.10.0-bin\apache-maven-3.10.0\bin` (en `PATH` de usuario) |
| `MAVEN_HOME` (usuario) | `C:\VANE\apache-maven-3.10.0-bin\apache-maven-3.10.0` | |
| Ollama | **0.40.5** | `C:\Users\Mkjdf983\AppData\Local\Programs\Ollama\ollama.exe` |
| Modelo | `deepseek-r1:1.5b` (1,1 GB) | **verificado: responde** |

> **Nota:** los cambios en variables de entorno (usuario) solo se ven en una
> **terminal nueva**. En PowerShell, `mvn -v` o `java -version` escriben en *stderr*;
> ver el `$LASTEXITCODE` real, no el "Exited with code 1" de PowerShell.

## 5. Decisiones tomadas (y por qué)

| Tema | Decisión | Motivo |
|---|---|---|
| Tema | Agentes con LLM (multi-agente) | Elección del usuario |
| Lenguaje | Java 21/25 + Spring Boot + Maven | Encaja con sus otros proyectos |
| Modo | Híbrido: motor a mano, luego framework | Aprender de verdad |
| Modelo | DeepSeek, ruta híbrida (local → API) | Gratis primero |
| Local | Ollama `deepseek-r1:1.5b` | Cabe en 7,4 GB de RAM |
| Nube | API `deepseek-chat` (V3) | Si lo local es lento |
| Java 25 | Instalar y usar | Compatible (ver §6) |

## 6. Stack y versiones clave

- **Spring Boot 4.1.1** — *«requires at least Java 17 and is compatible with versions up
  to and including Java 26»* → **Java 21 y 25 valen**.
- **Spring AI 2.0.1** — depende exactamente de Spring Boot 4.1.1. Se usa en la Fase 3.
- **Spring Boot 4 renombra starters:** `spring-boot-starter-web` → **`webmvc`**;
  los de test son `spring-boot-starter-webmvc-test` y `-actuator-test`.
- **Ollama y la API de DeepSeek son compatibles con OpenAI** → cambiar de local a nube
  es solo cambiar `base-url` / `model` / `api-key`:

| | Endpoint | Modelo | Clave |
|---|---|---|---|
| Local | `http://localhost:11434/v1` | `deepseek-r1:1.5b` | — |
| Nube | `https://api.deepseek.com/v1` | `deepseek-chat` | `DEEPSEEK_API_KEY` |

## 7. Estado actual

### Historial (usar `git log` para los hashes)

- `Commit inicial` — README + .gitignore
- `docs: plan detallado de aprendizaje (11 fases)` — `docs/plan-detallado.md`
- `Fase 0: proyecto Spring Boot 4.1.1 (Java 21, Maven)` — esqueleto
- `Fase 0: Java 25 + documentación de traspaso` — cierre de la Fase 0

### Fase 0 — Completada ✅

- [x] Ollama instalado y funcionando (`0.40.5`)
- [x] Modelo `deepseek-r1:1.5b` descargado y **probado** (respondió en ~16 s la 1ª vez)
- [x] Proyecto Spring Boot 4.1.1 creado, **compila y pasa tests**
- [x] Maven 3.10.0 instalado y en `PATH`
- [x] **Java 25 instalado, configurado y verificado** (`release 25`, BUILD SUCCESS)
- [x] `pom.xml` con `<java.version>25</java.version>`

### Fases 1–10

Todas pendientes. Ver `docs/plan-detallado.md`.

## 8. Cómo verificar que todo funciona

```powershell
# 1) Modelo local
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" list
# Debe mostrar: deepseek-r1:1.5b

# 2) API de Ollama
Invoke-RestMethod http://localhost:11434/api/tags

# 3) Proyecto
cd C:\VANE\MisProyectos
$env:JAVA_HOME = "C:\Program Files\Java\jdk-25.0.4.1"
.\mvnw.cmd -B -ntp clean package                   # BUILD SUCCESS
```

## 9. Cosas aprendidas (evitan repetir errores)

1. **Initializr devolvió `bootVersion=4.1.1.RELEASE`**, que **no existe** en Maven Central
   (Spring quitó el sufijo `.RELEASE`). Hay que usar **`4.1.1`**.
2. **`deepseek-r1` es un modelo "pensante"** (*thinking*): razona antes de responder →
   en CPU es lento. Para bucles de agente (Fases 6–8) considerar la API.
3. **La primera llamada tarda** porque Ollama carga el modelo en RAM (~16 s);
   las siguientes son rápidas.
4. PowerShell marca como "error" la salida de `git`/`mvn` por *stderr*;
   **mirar siempre `$LASTEXITCODE`**.
5. **`deepseek-r1` sí soporta herramientas** (badges `tools` + `thinking` en Ollama).

## 10. Próximos pasos inmediatos

1. **Fase 1 — "Hola DeepSeek" con HTTP crudo** (siguiente tarea):
   - Cliente `RestClient` que haga `POST` a `http://localhost:11434/v1/chat/completions`.
   - Endpoint `/chat` que reciba texto y devuelva la respuesta del modelo.
   - Detalle y ejercicios en `docs/plan-detallado.md` (Fase 1).
2. Al cerrar **cada fase**: actualizar el checklist del `README.md` **y** este archivo,
   y hacer `git commit` + `git push`.

## 11. Convenciones de trabajo

- **Commits:** `Fase N: descripción breve` (una fase = uno o más commits + `git push`).
- **Idioma:** español en docs, comentarios y mensajes de commit.
- **Estructura:** `docs/` para documentación; el código Spring Boot en la raíz (`src/`).
- **Antes de borrar o sobrescribir:** confirmar con el usuario.
- **No** añadir dependencias sin explicar para qué.

## 12. Prompt de arranque (copiar y pegar al siguiente agente)

```text
Estamos retomando un proyecto en C:\VANE\MisProyectos (repo GitHub vanexius/arquitectura).
Lee primero AGENTS.md, luego README.md y docs/plan-detallado.md, y sigue el plan.

Contexto: laboratorio de orquestación multi-agente en Java/Spring con DeepSeek.
Modelo local: Ollama con deepseek-r1:1.5b (ya instalado y funcionando).
Stack: Spring Boot 4.1.1, Spring AI 2.0.1 (Fase 3), Java 21/25, Maven 3.10.0.

Reglas: NO tocar la config global de git (es de otra cuenta); la config local ya está.
No commitear secretos. Mantener todo en español. Un commit por fase.

Punto de retomada: terminar la Fase 0 (configurar Java 25 y actualizar el pom),
actualizando AGENTS.md y el checklist del README al cerrar cada fase.
```
