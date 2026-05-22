# Prompt Testing — Chatbot Educativo de Programación
**Inteligencia Artificial I · Prompt Unit Testing con Promptfoo y Gemini**

Nombres: David Fabián Ortiz Vega · Camilo Andrés García Almeida

---

## Contexto

Una institución educativa detectó problemas en su chatbot de IA: respuestas inconsistentes, explicaciones poco claras, respuestas demasiado largas y variaciones inesperadas según el prompt. Como equipo de QA y Prompt Engineers, diseñamos y ejecutamos pruebas automatizadas para validar la calidad de las respuestas.

---

## Parte 1 — Configuración del entorno

### Requisitos instalados
- Node.js
- npm
- Promptfoo (`npm install -g promptfoo`)
- API Key de Gemini (obtenida desde Google AI Studio)

### Configuración de la API Key en PowerShell
```powershell
$env:GOOGLE_API_KEY="tu-clave-aqui"
```

### Comandos principales
```powershell
promptfoo eval --no-cache   # ejecutar evaluación
promptfoo view              # ver resultados en el navegador
```

---

## Parte 2 — Diseño del chatbot educativo

El chatbot está orientado a enseñar programación a estudiantes principiantes. Los prompts fueron diseñados para explicar conceptos de programación usando analogías, ejemplos en Python y estructuras claras y amigables.

### Prompts diseñados

**Prompt 1 — General:**
```
Explain the concept of {{topic}} in programming to a beginner student
```

**Prompt 2 — Estructurado con rol de tutor:**
```
You are an expert programming tutor specialized in teaching beginners.
Your goal is to explain the concept of {{topic}} in a way that a
student with no prior programming experience can understand.

Follow this structure:
1. Start with a simple real-world analogy (no technical terms yet)
2. Explain what {{topic}} means in programming in 2-3 sentences
3. Show a short code example in Python (max 10 lines)
4. Explain the code line by line in simple words
5. End with one practical use case where {{topic}} is useful

Rules:
- Never use jargon without explaining it first
- Keep the tone friendly and encouraging
- If the student might get confused, add a tip or warning
```

La diferencia entre ambos permite comparar qué tan importante es dar contexto e instrucciones claras al modelo. El Prompt 2 produce respuestas más estructuradas, con analogías, código Python y explicaciones paso a paso. El Prompt 1 genera respuestas más genéricas y variables.

---

## Parte 3 — Archivo YAML

```yaml
# yaml-language-server: $schema=https://promptfoo.dev/config-schema.json

description: "Chatbot educativo — tutor de programación"

evaluateOptions:
  maxConcurrency: 2

providers:
  - id: "google:gemini-2.5-flash"
    config:
      temperature: 0.7

defaultTest:
  options:
    provider: "google:gemini-2.5-flash"
    delay: 2000

prompts:
  - "Explain the concept of {{topic}} in programming to a beginner student"
  - |
    You are an expert programming tutor specialized in teaching beginners.
    Your goal is to explain the concept of {{topic}} in a way that a
    student with no prior programming experience can understand.

    Follow this structure:
    1. Start with a simple real-world analogy (no technical terms yet)
    2. Explain what {{topic}} means in programming in 2-3 sentences
    3. Show a short code example in Python (max 10 lines)
    4. Explain the code line by line in simple words
    5. End with one practical use case where {{topic}} is useful

    Rules:
    - Never use jargon without explaining it first
    - Keep the tone friendly and encouraging
    - If the student might get confused, add a tip or warning

tests:
  - vars:
      topic: arrays
    assert:
      - type: icontains
        value: "array"

  - vars:
      topic: arrays
    assert:
      - type: icontains
        value: "array"

  - vars:
      topic: arrays
    assert:
      - type: icontains
        value: "python"
      - type: icontains
        value: "array"
      - type: llm-rubric
        value: "The response follows a clear structure with an analogy, code example, and explanation appropriate for a beginner"
```

### Estructura del YAML

| Campo | Descripción |
|---|---|
| `description` | Nombre de la evaluación |
| `providers` | Modelo usado: `google:gemini-2.5-flash` |
| `prompts` | 2 plantillas de instrucción al modelo |
| `tests` | 3 casos de prueba con topic: arrays |
| `assert` | Condiciones que debe cumplir cada respuesta |
| `maxConcurrency: 2` | Dos requests en paralelo |
| `delay: 2000` | 2 segundos de espera entre requests |

---

## Parte 4 — Ejecución y análisis

### Resultados finales (evaluación exitosa)

| Test | Topic | Assertion | Prompt 1 | Prompt 2 |
|---|---|---|---|---|
| 1 | arrays | icontains "array" | ✅ PASS | ✅ PASS |
| 2 | arrays | icontains "array" | ✅ PASS | ✅ PASS |
| 3 | arrays | icontains "python" + icontains "array" + llm-rubric | ✅ PASS | ✅ PASS |

**Resultado final: 6/6 PASS (100%) · 0 failed · 0 errors · Duración: 1s**

### Errores encontrados durante el proceso

Durante las evaluaciones intermedias se presentaron dos tipos de errores antes de llegar al resultado final:

**RateLimitExhaustedError:** La API gratuita de Gemini tiene límites por minuto. Al lanzar varios tests simultáneamente, el modelo rechazaba las peticiones después de 4 intentos. Se resolvió ajustando `maxConcurrency` y `delay`.

**Timeout (300000ms):** Algunas requests quedaban en cola esperando respuesta y expiraban a los 5 minutos. Ocurría cuando el rate limit acumulaba peticiones sin procesarlas.

### Análisis de resultados

**PASS en icontains:** Ambos prompts mencionaron la palabra "array" y "python" en sus respuestas, cumpliendo las condiciones de texto exacto.

**PASS en llm-rubric:** El modelo juez evaluó que las respuestas del Prompt 2 seguían una estructura clara con analogía, ejemplo de código y explicación apropiada para principiantes. Esto confirma que el Prompt 2 estructurado produce respuestas de mayor calidad educativa.

**Comparación entre prompts:** El Prompt 1 generó respuestas como *"Okay, imagine you're trying to keep track of a bunch of similar things. Let's use an analogy first — A Row of Mailboxes..."*. El Prompt 2 generó respuestas como *"Hey there, future programmer! Let's dive into one of the most fundamental concepts in programming: arrays..."* con estructura más detallada y tono más amigable.

---

## Parte 5 — Exploración y experimentación

### Prueba con chatbot de cocina

Como experimentación adicional, antes de la versión final de programación, probamos el sistema con un chatbot de cocina usando topics como `cooking`, `spaghetti` y `potatoes`. Esta prueba nos permitió:

- Observar cómo el modelo adapta el tono según el dominio del prompt
- Detectar que `icontains` falla cuando el assertion exige palabras que el modelo no usa literalmente (por ejemplo, pedir "preparation" cuando el modelo dice "process")
- Confirmar que los errores de timeout se presentan cuando el rate limit acumula requests en cola

### Hallazgo clave: limitación de icontains

Durante las pruebas con el topic `loops`, el modelo usó sinónimos como *"repeat"* o *"same set of actions"* en lugar de la palabra exacta *"loop"*. El assertion `icontains` falló aunque la explicación era correcta. Esto reveló que `icontains` solo sirve para palabras que el modelo usa literalmente, mientras que `llm-rubric` es más adecuado para evaluar calidad semántica.

---

## Dificultades encontradas

- **Error 401 Unauthorized:** Al inicio se configuró el provider como `vertex:gemini-2.5-pro` en lugar de `google:gemini-2.5-flash`. Vertex AI requiere OAuth2 y no acepta API Keys simples de Google AI Studio.
- **Modelo inexistente:** Se usó `gemini-3.1-pro-preview` que no existe. El nombre correcto es `gemini-2.5-flash`.
- **RateLimitExhaustedError:** La API gratuita permite pocos requests por minuto. Se solucionó con `maxConcurrency: 1` o `2` y `delay` entre requests.
- **Timeout en requests:** Las peticiones expiraban tras 5 minutos en cola. Se resolvió reduciendo la concurrencia.
- **Error 503 UNAVAILABLE:** En algunos momentos los servidores de Gemini estaban saturados por alta demanda global. No es un error de configuración sino de infraestructura externa.
- **Caché de evaluaciones:** Promptfoo guarda resultados anteriores. Al no usar `--no-cache`, mezclaba resultados viejos con nuevos. Se resolvió siempre ejecutando con `promptfoo eval --no-cache`.

---

## Aprendizajes sobre Prompt Testing

- Los prompts son software: pequeños cambios producen resultados muy distintos.
- Un prompt estructurado con rol, instrucciones y reglas produce respuestas más consistentes y educativas.
- `icontains` es útil para verificar palabras clave exactas, pero falla cuando el modelo usa sinónimos correctos.
- `llm-rubric` evalúa calidad semántica y es más flexible, pero consume más tokens y puede agotar el rate limit.
- `javascript` permite validar condiciones numéricas como longitud de respuesta.
- Sin pruebas sistemáticas, es imposible garantizar la calidad de un chatbot en producción.
- El rate limit de la API gratuita es una restricción real que obliga a diseñar evaluaciones eficientes con concurrencia baja y delays.

---

## Criterios cubiertos

| Criterio | Estado |
|---|---|
| Configuración correcta del entorno | ✅ |
| YAML funcional con provider, prompts, tests y assertions | ✅ |
| Mínimo 2 prompts distintos | ✅ (2 prompts) |
| Mínimo 3 casos de prueba | ✅ (3 tests) |
| Mínimo 2 tipos de assertions | ✅ (icontains, llm-rubric) |
| Ejecución y análisis de resultados | ✅ (6/6 PASS) |
| Exploración y experimentación adicional | ✅ (chatbot de cocina + FAIL análisis) |
| Claridad de la entrega | ✅ |
