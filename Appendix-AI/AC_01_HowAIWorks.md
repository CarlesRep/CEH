# AC_01 — How AI Works: AI, ML, Deep Learning, and LLM

---

## 1. Conceptos y Definiciones

### Jerarquía AI → ML → Deep Learning → LLM 🔴

La relación es de **anidamiento por especialización**. Cada nivel es un subconjunto del anterior:

```
AI
└── Machine Learning (subset of AI)
    └── Deep Learning (subset of ML)
        └── LLM (specific application of Deep Learning)
```

| Nivel | Definición operativa | Característica diferencial |
|---|---|---|
| **AI** | Simulación de inteligencia humana en máquinas (percibir, razonar, aprender, interactuar) | Categoría más amplia; engloba todo |
| **ML** | Proporciona a la IA la **capacidad de aprender** desde datos sin programación explícita para cada tarea | Aprende de datos; hace predicciones/decisiones |
| **Deep Learning** | Clase de algoritmos de ML que usa **redes neuronales con múltiples capas** (deep neural networks) para aprender patrones complejos desde grandes volúmenes de datos | Habilita: reconocimiento de imagen, voz, NLP |
| **LLM** | Clase específica de modelos de deep learning entrenada sobre **vastas cantidades de texto** para comprender y generar lenguaje humano | Usa arquitectura transformer; GPT, BERT |

**Neural Networks**: componente fundamental de Deep Learning. Aprenden **representaciones jerárquicas** de los datos. No son sinónimo de Deep Learning; son la tecnología subyacente.

---

### Tecnologías de AI — tipos y funciones

| Tecnología | Función principal |
|---|---|
| **Cognitive Computing** | Simula procesos del pensamiento humano (percepción, razonamiento, toma de decisiones, aprendizaje desde la experiencia) |
| **Computer Vision** | Interpreta información visual, reconoce patrones, extrae insights de imágenes o vídeo |
| **Machine Learning** | Aprende y mejora automáticamente desde la experiencia sin programación explícita |
| **Deep Learning** | Enseña patrones intrincados desde datasets grandes y complejos; realiza tareas como reconocimiento de habla e imagen |
| **Neural Networks** | Componente fundamental de Deep Learning; aprende representaciones jerárquicas |
| **Natural Language Processing (NLP)** | Comunicación entre humanos y máquinas usando lenguajes humanos |

---

### Cómo funciona un LLM — pipeline completo 🔴

LLM usa una arquitectura de **transformer neural network** con parámetros extensos para procesar y comprender lenguaje humano.

**Fases del funcionamiento:**

1. **Training Data**: el modelo se entrena sobre vastas cantidades de texto de internet, libros, artículos, webs, etc. Aprende patrones de lenguaje, gramática, semántica y comprensión contextual.

2. **Tokenization**: el input del usuario (prompt/query) se descompone en unidades más pequeñas llamadas **tokens** (palabras o subpalabras) que el modelo puede procesar.

3. **Contextual Understanding**: el LLM analiza la secuencia de tokens y usa **mecanismos de atención (attention mechanisms)** para ponderar la importancia de cada token según su relevancia para el contexto global.

4. **Language Generation**: el LLM genera respuestas prediciendo la continuación o completado más probable del input, basándose en sus datos de entrenamiento.

5. **Fine-Tuning**: los LLMs pueden ajustarse para tareas o dominios específicos mediante entrenamiento adicional sobre un dataset más pequeño relacionado con la tarea (generación de código, traducción, resumen, etc.).

6. **Feedback Loop**: los LLMs mejoran su rendimiento a través de un bucle de retroalimentación, aprendiendo de interacciones y correcciones de usuarios para refinar comprensión y generación de lenguaje.

**Pipeline simplificado:**
```
Prompt/Input → Tokenización → Embedding/Representación matemática (Context Vector) → Generación de output
```

**Ejemplos de LLMs**: GPT (OpenAI) — Generative Pre-trained Transformer; BERT (Google) — Bidirectional Encoder Representations from Transformers.

---

### Aplicaciones de AI por sector

| Sector | Función de AI |
|---|---|
| Vehículos autónomos | Combina computer vision, ML y sensor fusion para navegación autónoma |
| Reconocimiento facial | Seguridad mediante autenticación facial |
| Diagnóstico médico | Algoritmos para diagnóstico preciso, detección temprana, planes de tratamiento personalizados |
| Atención al cliente | Chatbots/asistentes virtuales 24×7 |
| Manufactura | Predicción de fallos de equipos → mantenimiento preventivo |
| Recomendación de contenido | Siri, Alexa, plataformas de streaming, apps de navegación |
| Ciberseguridad | Análisis de tráfico de red, detección de anomalías, predicción de ataques |

---

### Aplicaciones de LLM

El libro enumera 18 aplicaciones. Las más relevantes para CEH/ciberseguridad:
- Detección de fraude, análisis de sentimiento, generación de código, procesamiento de lenguaje natural, chatbots, sistemas de búsqueda, clasificación, análisis de AI, respuesta a preguntas.

---

### 🔴 Desafíos de AI — lista completa

| Desafío | Descripción clave |
|---|---|
| **Computing Power** | Coste de supercomputadores y cloud → retrasa el desarrollo |
| **Trust Deficit** | Falta de transparencia en cómo los modelos llegan a sus outputs |
| **Limited Knowledge** | Falta de comprensión general sobre potencial y limitaciones de AI |
| **Human-level Performance** | Igualar consistentemente la precisión humana requiere datasets vastos y algoritmos ajustados |
| **Data Privacy and Security** | Los datasets masivos de entrenamiento generan preocupaciones sobre seguridad y uso indebido de datos personales |
| **Lack of Understanding** | Conceptos erróneos y expectativas poco realistas dificultan la adopción |
| **Unreliable Results** | Sesgos en datos + escenarios reales complejos → outputs inexactos |
| **Implementation Strategy** | Requiere planificación cuidadosa, infraestructura preparada y participación de stakeholders |
| **The Bias Problem** | Los sistemas AI **heredan sesgos** de los datos de entrenamiento → resultados discriminatorios |
| **Data Scarcity** | Acceso limitado a datos por privacidad y regulaciones → modelos sesgados |

Los **tres desafíos más interrelacionados** para el examen: Bias Problem + Unreliable Results + Data Scarcity forman un triángulo causal: escasez de datos → modelos entrenados con datos sesgados → resultados poco fiables.

---

## 2. Exam Traps ⚠️

⚠️ **[Jerarquía AI/ML/DL/LLM]** — El examen puede invertir la relación de subconjuntos. La secuencia correcta: ML es subconjunto de AI; Deep Learning es subconjunto de ML; LLM es aplicación específica de Deep Learning. Nunca al revés.

⚠️ **[Neural Networks vs. Deep Learning]** — Las redes neuronales son el **componente fundamental** de Deep Learning, no sinónimos. Deep Learning usa redes neuronales con **múltiples capas** (de ahí "deep"). Una red neuronal de una sola capa no es Deep Learning.

⚠️ **[LLM — arquitectura subyacente]** — LLMs usan arquitectura **transformer** con mecanismos de atención. El examen puede presentar otras arquitecturas (CNN, RNN) como base de los LLMs. La respuesta correcta es transformer.

⚠️ **[Tokenization — qué es un token]** — Un token puede ser una palabra o una **subpalabra** (subword), no necesariamente un carácter o una frase. El examen puede presentar "caracteres" o "frases" como definición de token.

⚠️ **[Fine-Tuning vs. Training]** — El training inicial usa **vastos datasets genéricos**. El fine-tuning usa un **dataset más pequeño y específico** para especializar el modelo. El examen puede confundir ambos procesos como equivalentes.

⚠️ **[Bias Problem — origen]** — El sesgo en AI proviene de los **datos de entrenamiento**, no del algoritmo en sí (aunque el algoritmo puede amplificarlo). Si el examen pregunta "¿de dónde heredan los sistemas AI sus sesgos?", la respuesta es: datos de entrenamiento.

⚠️ **[Trust Deficit — causa]** — El déficit de confianza en AI se debe a la **falta de transparencia** en cómo los modelos llegan a sus outputs (problema de "caja negra"), no a errores en los resultados per se.

⚠️ **[GPT vs. BERT — ambos son LLMs]** — GPT es de OpenAI (Generative Pre-trained Transformer). BERT es de Google (Bidirectional Encoder Representations from Transformers). Ambos son ejemplos de LLMs basados en arquitectura transformer, pero con orientaciones distintas (GPT = generativo; BERT = comprensión bidireccional).

---

## 3. Nemotécnicos

### Jerarquía de especialización
**"AI Madre, ML Hija, DL Nieta, LLM Bisnieta"**  
→ AI ⊃ ML ⊃ Deep Learning ⊃ LLM

### Pipeline LLM — "T-CAF-F"
**T**raining → **C**ontextual Understanding → **A**ttention Mechanisms → **F**ine-tuning → **F**eedback Loop  
(Con tokenización entre Training y Contextual)

### Desafíos AI — los 3 del triángulo causal
**"Escasez → Sesgo → Resultados poco fiables"**  
Data Scarcity → Bias Problem → Unreliable Results

### LLM ejemplos — "GPT-O, BERT-G"
- **GPT** = OpenAI = **G**enerative **P**re-trained **T**ransformer
- **BERT** = Google = **B**idirectional **E**ncoder **R**epresentations from **T**ransformers

### Tecnologías AI — "C²M-DNN"
**C**ognitive Computing · **C**omputer Vision · **M**achine Learning · **D**eep Learning · **N**eural Networks · **N**atural Language

---

## 4. Flashcards

**Q:** ¿Cuál es la relación jerárquica entre AI, ML, Deep Learning y LLM?  
**A:** ML es subconjunto de AI. Deep Learning es subconjunto de ML. LLM es una aplicación específica de Deep Learning. Es una jerarquía de especialización creciente.

---

**Q:** ¿Qué arquitectura de red neuronal usan los LLMs y qué mecanismo les permite ponderar la relevancia de cada token?  
**A:** Arquitectura **transformer**. Usan **attention mechanisms** (mecanismos de atención) para ponderar la importancia de cada token según su relevancia contextual.

---

**Q:** ¿Qué es la tokenización en el contexto de los LLMs?  
**A:** El proceso de descomponer el input del usuario (prompt) en unidades más pequeñas llamadas **tokens** — palabras o subpalabras — que el modelo puede procesar matemáticamente.

---

**Q:** ¿Cuál es la diferencia entre el training inicial de un LLM y el fine-tuning?  
**A:** El **training inicial** usa vastas cantidades de texto genérico (internet, libros, artículos) para aprender patrones generales de lenguaje. El **fine-tuning** usa un dataset más pequeño y específico para especializar el modelo en una tarea concreta (generación de código, traducción, etc.).

---

**Q:** ¿Qué tecnología AI combina computer vision, ML y sensor fusion para habilitar vehículos autónomos?  
**A:** La combinación de estas tres tecnologías constituye el sistema de navegación autónoma. No es una sola tecnología; es una **combinación de técnicas AI**.

---

**Q:** ¿De dónde heredan los sistemas AI sus sesgos y qué consecuencia tiene?  
**A:** Los sesgos se heredan de los **datos de entrenamiento**. Si los datos contienen sesgos (históricos, de selección, etc.), el modelo los aprende y puede producir **resultados discriminatorios**.

---

**Q:** ¿Qué desafío de AI se refiere específicamente a la falta de transparencia en cómo los modelos llegan a sus outputs?  
**A:** **Trust Deficit** (Déficit de confianza). Los usuarios no pueden entender el proceso interno del modelo → "caja negra" → dificultad para confiar en sus outputs.

---

**Q:** ¿Qué son GPT y BERT, y quién los desarrolló?  
**A:** Ambos son ejemplos de LLMs basados en arquitectura transformer. **GPT** (Generative Pre-trained Transformer) fue desarrollado por **OpenAI**. **BERT** (Bidirectional Encoder Representations from Transformers) fue desarrollado por **Google**.

---

**Q:** ¿Qué diferencia existe entre Neural Networks y Deep Learning?  
**A:** Las redes neuronales son el **componente fundamental** (tecnología base) de Deep Learning. Deep Learning usa redes neuronales con **múltiples capas** (deep neural networks). Una red neuronal de capa única no es Deep Learning; el "deep" hace referencia a la profundidad (número de capas).

---

**Q:** ¿Qué fase del pipeline LLM permite al modelo mejorar su rendimiento a partir de interacciones reales de usuarios?  
**A:** El **Feedback Loop** (bucle de retroalimentación). El modelo aprende de las correcciones e interacciones de usuarios para refinar su comprensión y generación de lenguaje.

---

**Q:** ¿Qué desafío de AI describe la situación donde el acceso limitado a datos por regulaciones de privacidad conduce a modelos sesgados?  
**A:** **Data Scarcity** (Escasez de datos). La falta de datos suficientes y representativos impide entrenar modelos equilibrados y lleva a sesgos.

---

**Q:** ¿Cuál es la aplicación de AI en manufactura mencionada explícitamente en el libro?  
**A:** **Predicción de fallos de equipos** (predict equipment failures) para mantenimiento preventivo y minimización del tiempo de inactividad.

---

**Q:** ¿Qué tecnología AI se enfoca específicamente en aprender representaciones jerárquicas de datos?  
**A:** **Neural Networks** (Redes Neuronales). Son el componente fundamental de Deep Learning y su característica definitoria es el aprendizaje de representaciones jerárquicas.

---

**Q:** ¿Cuál es el rol de AI en ciberseguridad según el libro?  
**A:** Detectar y mitigar amenazas de seguridad mediante análisis de tráfico de red, identificación de anomalías y predicción de ataques potenciales. Las herramientas de ciberseguridad potenciadas con AI mejoran las capacidades de detección y respuesta a amenazas.

---

**Q:** ¿Qué es Cognitive Computing y en qué se diferencia de Machine Learning?  
**A:** Cognitive Computing simula **procesos del pensamiento humano** en modelos computarizados — percepción, razonamiento, toma de decisiones, resolución de problemas y aprendizaje desde la experiencia. ML se centra en aprender desde datos para hacer predicciones. Cognitive Computing es más amplio e intenta replicar el proceso cognitivo completo; ML es el mecanismo de aprendizaje.

---

## 5. Confusión frecuente

### Machine Learning vs. Deep Learning
- **Machine Learning**: aprende desde datos usando algoritmos (regresión, árboles de decisión, SVM, etc.). No requiere redes neuronales necesariamente. Funciona bien con datasets moderados.
- **Deep Learning**: subconjunto de ML que usa exclusivamente **redes neuronales profundas (múltiples capas)**. Requiere grandes volúmenes de datos y alta capacidad computacional. Es la tecnología detrás de reconocimiento de imagen, voz y LLMs.
- **Criterio**: si la pregunta menciona "redes neuronales con múltiples capas" → Deep Learning. Si menciona "aprender desde datos sin programación explícita" en términos generales → ML.

---

### Deep Learning vs. Neural Networks
- **Neural Networks**: son la **tecnología/componente** — redes de neuronas artificiales que aprenden representaciones jerárquicas.
- **Deep Learning**: es el **paradigma/enfoque** que usa redes neuronales con muchas capas (deep = profundo = muchas capas ocultas).
- **Criterio**: si la pregunta pregunta por la "tecnología base" de Deep Learning → Neural Networks. Si pregunta por el "subconjunto de ML que usa redes profundas" → Deep Learning.

---

### Training vs. Fine-Tuning en LLMs
- **Training**: proceso inicial masivo sobre datasets genéricos y vastos. Enseña al modelo el lenguaje en general. Extremadamente costoso computacionalmente.
- **Fine-Tuning**: entrenamiento adicional sobre un dataset pequeño y específico para especializar el modelo en una tarea concreta. Mucho menos costoso que el training inicial.
- **Criterio**: si el escenario habla de "entrenar un modelo desde cero con millones de documentos" → training. Si habla de "adaptar un modelo existente para generación de código médico" → fine-tuning.

---

### Bias Problem vs. Unreliable Results vs. Trust Deficit
- **Bias Problem**: el modelo hereda sesgos de los **datos de entrenamiento** → produce resultados sistemáticamente discriminatorios o distorsionados.
- **Unreliable Results**: los outputs son **inexactos** por sesgos en datos O por la complejidad de escenarios reales. Más amplio que el Bias Problem.
- **Trust Deficit**: los usuarios **no confían** en el modelo porque no entienden cómo llega a sus outputs (transparencia, "caja negra"). No es sobre exactitud sino sobre comprensibilidad.
- **Criterio**: si la pregunta menciona "discriminatorio" o "datos sesgados" → Bias Problem. Si menciona "outputs incorrectos" sin especificar causa → Unreliable Results. Si menciona "falta de transparencia" o "caja negra" → Trust Deficit.

---

### LLM vs. Chatbot vs. Virtual Assistant
- **LLM**: modelo de Deep Learning entrenado sobre texto masivo para comprender y generar lenguaje. Es la tecnología base.
- **Chatbot**: aplicación que usa LLMs (u otras tecnologías) para conversación automatizada con usuarios. Puede estar alimentado por un LLM o por sistemas más simples basados en reglas.
- **Virtual Assistant**: categoría más amplia (Siri, Alexa) que puede usar LLMs + computer vision + otras tecnologías de AI.
- **Criterio**: si la pregunta menciona "modelo entrenado en texto" o "transformer" → LLM. Si menciona "soporte al cliente automatizado" → chatbot. Si menciona comandos de voz y múltiples capacidades → virtual assistant.
