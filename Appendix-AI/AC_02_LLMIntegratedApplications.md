# AC_02 — LLM Integrated Applications

---

## 1. Conceptos y Definiciones

### Arquitectura de aplicaciones LLM integradas 🔴

Las aplicaciones que integran LLMs exponen a la organización a **web LLM attacks** que aprovechan el acceso del modelo a datos, APIs o información de usuarios que un atacante no puede acceder directamente. Este es el vector de riesgo central de las aplicaciones LLM integradas.

**Componentes del diagrama de arquitectura:**

| Componente | Rol |
|---|---|
| **User** | Origina la consulta |
| **App Frontend** | Interfaz de usuario; recibe y presenta |
| **LLM Orchestrator** | Coordina el flujo entre frontend, backend y el LLM; gestiona las llamadas |
| **App Backend** | Lógica de negocio, APIs, datos de la organización |
| **LLM** | Modelo de lenguaje que procesa la consulta y genera la respuesta |

**Flujo numerado (secuencia del diagrama):**
1. User → App Frontend: **Ask a Question**
2. App Frontend → LLM Orchestrator: **Deliver Question**
3. LLM Orchestrator → LLM: **Ask for code**
4. LLM → (proceso interno): **Research Code**
5. LLM → App Backend: **Execute Code**
6. App Backend → LLM Orchestrator: **Code Result**
7. LLM Orchestrator → App Frontend: **Deliver Question** (resultado al usuario)

El **LLM Orchestrator** es el componente clave de seguridad: gestiona qué llega al LLM y qué ejecuta el backend. Un atacante que comprometa el orquestador o manipule los prompts puede pivotar hacia el backend y las APIs.

---

### 🔴 Aplicaciones LLM reales — tabla de identificación

| Modelo/Herramienta | Categoría | Desarrollador | Función específica |
|---|---|---|---|
| **Claude** | Content generation | **Anthropic** | Asistente AI de propósito general |
| **ChatGPT** | Content generation | **OpenAI** | Genera texto a partir de prompts |
| **Falcon LLM** | Translation & localization | — | Razonamiento, programación, evaluación de habilidades y conocimiento |
| **NLLB-200** | Translation & localization | — | Traducción entre **200 idiomas** distintos |
| **Gemini** | Search & recommendation | **Google** | Chatbot AI de búsqueda y recomendación |
| **Alexa** | Virtual assistants | **Amazon** | Asistente de voz; control de dispositivos inteligentes, alarmas, podcasts, música |
| **Google Assistant** | Virtual assistants | **Google** | Dispositivos móviles y domótica; enviar mensajes, música, información meteorológica |
| **Codex** | Code development | **OpenAI** | Entrenado en código; genera snippets, explica código, asiste en escritura y comprensión |
| **Grammarly** | Sentiment analysis | — | Corrección gramatical, ortografía, puntuación, claridad, detección de plagio, sugerencias de reemplazo |
| **LLaMA** | Question answering | **Meta** | Predice y genera texto; comprende contexto; proporciona información precisa y relevante |
| **Brandwatch** | Market research | — | Plataforma de inteligencia digital del consumidor; analiza conversaciones online; insights de investigación de mercado |
| **Talkwalker** | Market research | — | Respuestas en tiempo real a preguntas de gestión crítica; testing de productos; feedback de clientes |

---

### Riesgo de seguridad en aplicaciones LLM integradas

El riesgo central: las apps LLM mejoran la experiencia de usuario pero **exponen a la organización a web LLM attacks**. El modelo tiene acceso a:
- Datos internos de la organización
- APIs del backend
- Información de usuarios

Un atacante que manipule el LLM (ej. mediante prompt injection) puede acceder indirectamente a recursos que no podría alcanzar directamente. El LLM actúa como **proxy involuntario** hacia el backend.

---

## 2. Exam Traps ⚠️

⚠️ **[Claude — desarrollador]** — Claude es desarrollado por **Anthropic**, no por OpenAI. ChatGPT es de OpenAI. El examen puede cruzar estos dos dado que ambos son asistentes de contenido generativo.

⚠️ **[Codex — desarrollador]** — Codex es de **OpenAI** (misma empresa que ChatGPT). Está entrenado específicamente en código. No confundir con LLaMA (Meta) ni con Gemini (Google).

⚠️ **[LLaMA — desarrollador]** — LLaMA es de **Meta**. El examen puede atribuirlo a OpenAI o Google dada la confusión entre grandes fabricantes de LLMs.

⚠️ **[Gemini — categoría]** — Gemini se clasifica en el libro como **Search and recommendation**, no como "content generation" aunque también genera contenido. Su categoría específica en el libro es búsqueda y recomendación.

⚠️ **[Grammarly — categoría]** — Grammarly está clasificada como **Sentiment analysis** en el libro, aunque su función visible es corrección gramatical. El examen puede presentar opciones como "content generation" o "grammar checking" que no corresponden a la categoría del libro.

⚠️ **[NLLB-200 — dato numérico]** — NLLB-200 traduce entre **200 idiomas**. El número está en el nombre. Si el examen pregunta "¿qué modelo soporta 200 idiomas?", la respuesta es NLLB-200.

⚠️ **[LLM Orchestrator — rol en seguridad]** — El orquestador no es solo un componente arquitectónico; es el punto crítico de seguridad. Un atacante que lo compromise puede pivotar hacia el backend y las APIs. El examen puede preguntar qué componente coordina las llamadas entre el frontend y el LLM.

⚠️ **[Alexa vs. Google Assistant — fabricante]** — Alexa es de **Amazon**; Google Assistant es de **Google**. Ambos son asistentes virtuales de voz pero de fabricantes distintos.

---

## 3. Nemotécnicos

### Fabricantes de LLMs principales — "AOGGM"
- **A**nthropologic → **Claude**
- **O**penAI → **ChatGPT** + **Codex**
- **G**oogle → **Gemini** + **Google Assistant**
- **G**oogle / Meta → separar: **LLaMA = Meta**
- **A**mazon → **Alexa**

### Regla de los tres "G" en asistentes
- **G**emini = Google (búsqueda/recomendación)
- **G**oogle Assistant = Google (asistente de voz/domótica)
- ≠ **A**lexa = Amazon (voz + smart home)

### NLLB-200 — "el nombre contiene el número"
200 idiomas → NLLB-**200**

### Categorías LLM — asociación herramienta↔categoría
- Código → **Codex** (OpenAI)
- Texto → **ChatGPT** (OpenAI) / **Claude** (Anthropic)
- Traducción → **Falcon** + **NLLB-200**
- Preguntas → **LLaMA** (Meta)
- Mercado → **Brandwatch** + **Talkwalker**
- Gramática/análisis → **Grammarly**

---

## 4. Flashcards

**Q:** ¿Qué componente de la arquitectura LLM integrada coordina el flujo entre el frontend, el backend y el modelo LLM?  
**A:** El **LLM Orchestrator**. Es el componente central que gestiona las llamadas y coordina qué información llega al LLM y qué operaciones ejecuta el backend.

---

**Q:** ¿Por qué las aplicaciones LLM integradas representan un riesgo de seguridad para las organizaciones?  
**A:** Exponen a la organización a **web LLM attacks** que aprovechan el acceso del modelo a datos internos, APIs y información de usuarios. Un atacante puede manipular el LLM para acceder indirectamente a recursos que no podría alcanzar directamente.

---

**Q:** ¿Qué asistente AI fue desarrollado por Anthropic?  
**A:** **Claude**.

---

**Q:** ¿Qué LLM está especializado en generación y comprensión de código, y quién lo desarrolló?  
**A:** **Codex**, desarrollado por **OpenAI**. Está entrenado en código de diversas fuentes y puede generar snippets, proporcionar explicaciones y asistir en escritura y comprensión de código.

---

**Q:** ¿Qué modelo LLM es capaz de traducir entre 200 idiomas diferentes?  
**A:** **NLLB-200** (No Language Left Behind). El número 200 forma parte del nombre del modelo.

---

**Q:** ¿Qué modelo LLM desarrolló Meta y para qué se usa principalmente?  
**A:** **LLaMA** (Large Language Model by Meta). Predice y genera texto, comprende contexto y proporciona información precisa y relevante. Categoría: Question Answering.

---

**Q:** ¿En qué categoría clasifica el libro a Grammarly y cuáles son sus funciones?  
**A:** **Sentiment analysis**. Sus funciones incluyen: corrección gramatical y ortográfica, puntuación, claridad, detección de plagio y sugerencias de reemplazo en textos en inglés.

---

**Q:** ¿Qué diferencia existe entre Brandwatch y Talkwalker en cuanto a su función de market research?  
**A:** **Brandwatch** analiza conversaciones online para proporcionar insights de investigación de mercado (plataforma de inteligencia digital del consumidor). **Talkwalker** ofrece respuestas en tiempo real a preguntas de gestión crítica, usado para testing de productos y feedback de clientes.

---

**Q:** ¿Qué asistente virtual es controlado por voz, desarrollado por Amazon, y permite controlar dispositivos inteligentes?  
**A:** **Alexa** (Amazon). Incluye: interacción por voz, configuración de alarmas, streaming de podcasts y música, control de dispositivos smart home.

---

**Q:** ¿En qué categoría se clasifica Gemini en el libro y quién lo desarrolló?  
**A:** Categoría: **Search and recommendation**. Desarrollado por **Google**.

---

**Q:** Nombra los pasos del flujo de una aplicación LLM integrada en el orden correcto del diagrama.  
**A:** (1) Ask a Question → (2) Deliver Question → (3) Ask for code → (4) Research Code → (5) Execute Code → (6) Code Result → (7) Deliver Question (resultado final al usuario).

---

**Q:** ¿Qué modelo LLM excele en razonamiento, programación, evaluación de habilidades y conocimiento, y se clasifica en traducción y localización?  
**A:** **Falcon LLM**.

---

**Q:** ¿En qué se diferencia Google Assistant de Alexa en cuanto a sus capacidades y contexto de uso?  
**A:** **Google Assistant** (Google): disponible en dispositivos móviles y domótica; puede enviar mensajes, reproducir música, proporcionar información meteorológica y controlar electrodomésticos inteligentes. **Alexa** (Amazon): principalmente controlado por voz; especializado en smart home, alarmas, streaming de audio. Fabricantes distintos: Google vs. Amazon.

---

**Q:** ¿Qué riesgo específico introduce el LLM Orchestrator como punto de ataque en aplicaciones LLM integradas?  
**A:** El orquestador gestiona el acceso del LLM al backend y las APIs. Si un atacante manipula los prompts o compromete el orquestador (ej. mediante prompt injection), puede usar el LLM como **proxy** para acceder a datos, APIs o información de usuarios del backend que no podría alcanzar directamente.

---

**Q:** ¿Qué dos herramientas de OpenAI están listadas en el libro y cuál es la diferencia de propósito?  
**A:** **ChatGPT**: generación de texto a partir de prompts, propósito general. **Codex**: generación y comprensión de código específicamente, entrenado en repositorios de código.

---

## 5. Confusión frecuente

### Claude vs. ChatGPT
- **Claude**: desarrollado por **Anthropic**. Asistente AI de propósito general centrado en seguridad y alineamiento.
- **ChatGPT**: desarrollado por **OpenAI**. Asistente de generación de texto de propósito general.
- Ambos se clasifican en "Content generation" en el libro.
- **Criterio**: si la pregunta menciona "Anthropic" → Claude. Si menciona "OpenAI" en contexto de chatbot de texto general → ChatGPT. Si menciona "OpenAI" + "código" → Codex.

---

### Alexa vs. Google Assistant
- **Alexa**: Amazon. Voz + smart home + entretenimiento (podcasts, música). Control centralizado de hogar.
- **Google Assistant**: Google. Móvil + domótica + información contextual (tiempo, mensajes, música). Más integrado con servicios de Google.
- **Criterio**: si la pregunta menciona "Amazon" o "Alexa" → Amazon. Si menciona "Google" en contexto de asistente de voz en móvil → Google Assistant. Si menciona "Google" en contexto de búsqueda/recomendación → Gemini.

---

### Gemini vs. Google Assistant
- **Gemini**: chatbot/modelo AI de Google para **búsqueda y recomendación**. Sucesor de Bard.
- **Google Assistant**: asistente de voz de Google para **dispositivos móviles y domótica**.
- Ambos son de Google pero con propósitos distintos y categorías distintas en el libro.
- **Criterio**: si la pregunta menciona "chatbot de búsqueda" o "recomendación" → Gemini. Si menciona "asistente de voz en móvil" o "smart home" → Google Assistant.

---

### LLaMA vs. Codex (modelos de código/texto)
- **LLaMA** (Meta): modelo de propósito general para **comprensión de texto y respuesta a preguntas**. No especializado en código.
- **Codex** (OpenAI): especializado exclusivamente en **generación y comprensión de código**.
- **Criterio**: si la pregunta pide un modelo para "generar código" → Codex. Si pide "comprensión de lenguaje natural y respuesta a preguntas" → LLaMA.

---

### Brandwatch vs. Talkwalker
- **Brandwatch**: análisis de **conversaciones online** para inteligencia del consumidor. Más orientado a escucha social.
- **Talkwalker**: respuestas **en tiempo real** a preguntas de gestión. Más orientado a testing de productos y feedback operativo.
- Ambos son "Market research" pero con distintos focos: Brandwatch = social listening; Talkwalker = real-time management insights.
- **Criterio**: si la pregunta menciona "conversaciones online" o "inteligencia del consumidor" → Brandwatch. Si menciona "tiempo real" o "testing de productos" → Talkwalker.
