# AC_04 — Attacks on Machine Learning (OWASP ML Security Top Ten)

---

## 1. Conceptos y Definiciones

### 🔴 OWASP Machine Learning Security Top Ten — tabla maestra

| ID | Nombre | Mecanismo core | Diferenciador clave |
|---|---|---|---|
| **ML01** | Input Manipulation Attack | Altera el **input** para engañar al modelo en inferencia | Ataque en tiempo de inferencia; no afecta el entrenamiento |
| **ML02** | Data Poisoning Attack | Manipula los **datos de entrenamiento** para comportamiento incorrecto | Ataque en tiempo de entrenamiento; altera datos/etiquetas |
| **ML03** | Model Inversion Attack | Usa los **outputs del modelo** para hacer ingeniería inversa y extraer información | Extrae parámetros/arquitectura/datos del modelo a partir de sus predicciones |
| **ML04** | Membership Inference Attack | Usa un modelo entrenado + muestra de datos para determinar **si ese dato formó parte del entrenamiento** | Inferencia sobre el dataset de entrenamiento, no extracción del modelo |
| **ML05** | Model Theft | Obtiene acceso a los **parámetros del modelo** para clonarlo | Objetivo: replicar el modelo completo |
| **ML06** | AI Supply Chain Attack | Modifica o reemplaza una **librería o modelo ML** usada por el sistema | Ataque antes del despliegue; pasa desapercibido largo tiempo |
| **ML07** | Transfer Learning Attack | Explota el proceso de **transfer learning** (entrena en tarea A, fine-tunes en tarea B) para comprometer el modelo destino | Introduce backdoor en el modelo pre-entrenado que se transfiere |
| **ML08** | Model Skewing | Altera la **distribución de los datos de entrenamiento** para desplazar la frontera de clasificación | Sesga el límite entre "bueno" y "malo" del clasificador |
| **ML09** | Output Integrity Attack | **Modifica directamente el output** del modelo después de generarse | Ataque post-inferencia; no afecta el modelo en sí |
| **ML10** | Model Poisoning | Manipula los **parámetros del modelo** directamente (no los datos de entrenamiento) | Altera pesos/parámetros directamente → misclassification en subconjunto de muestras |

---

### ML01 — Input Manipulation Attack (Adversarial Attack)

El atacante altera deliberadamente los datos de input para engañar al modelo durante la inferencia. Las modificaciones pueden ser imperceptibles para un humano pero suficientes para cambiar la predicción del modelo radicalmente.

**Ejemplos canónicos:**
- Una imagen de un panda (57% de confianza) + ruido adversarial → el modelo lo clasifica como gibón (99.3% de confianza).
- Un barco → el modelo lo clasifica como coche (99.7% confianza).
- Un caballo → el modelo lo clasifica como rana (99.9% confianza).

**Ejemplo en ciberseguridad**: manipular el tráfico de red (IP origen/destino, payload) para que el IDS clasifique tráfico malicioso como benigno.

---

### ML02 — Data Poisoning Attack

El atacante compromete la integridad del modelo manipulando los datos de entrenamiento (muestras o etiquetas) para que haga predicciones incorrectas cuando se despliega.

**Ejemplos del libro:**
- **Spam classifier**: el atacante inyecta emails de spam etiquetados incorrectamente como "no spam" en el dataset de entrenamiento → el modelo aprende a clasificar spam como legítimo.
- **Traffic classification**: el atacante introduce tráfico de red con etiquetas incorrectas → el modelo aprende a clasificar ese tráfico en la categoría equivocada.
- **Visual**: un modelo de conducción autónoma confunde una señal de STOP con un límite de velocidad (imagen contaminada en el entrenamiento).

---

### ML03 — Model Inversion Attack

Usa los **outputs del modelo** para hacer ingeniería inversa y extraer información sensible (parámetros, arquitectura, datos de entrenamiento).

**Ejemplos:**
- **Bot detection bypass**: un anunciante entrena un modelo de deep learning de detección de bots y lo usa para modificar las predicciones del sistema de detección de la plataforma publicitaria, permitiendo que sus bots evadan la detección.
- **Face recognition personal data extraction**: el atacante entrena un modelo de reconocimiento facial, lo usa para atacar el modelo de reconocimiento de una empresa, introduce imágenes de 12 individuos y recupera información personal (nombre, dirección, SSN) a partir de las predicciones del modelo.

---

### ML04 — Membership Inference Attack

El atacante usa un modelo entrenado y una muestra de datos para determinar **si esa muestra específica fue usada para entrenar el modelo**. No extrae el modelo; infiere si ciertos datos estuvieron en el training set.

**Señal de inferencia**: si el modelo da una confianza muy alta (ej. 98%) para una muestra → probablemente fue parte del entrenamiento. Si da confianza baja y distribuida → probablemente no lo fue.

**Ejemplo**: el atacante entrena un modelo sobre registros financieros de una organización y luego consulta al modelo si el registro de un individuo específico estaba en los datos de entrenamiento → extrae información financiera sensible sin tener acceso directo al dataset.

---

### ML05 — Model Theft

El atacante accede a los parámetros del modelo para clonarlo. El proceso habitual:

**Flujo de Model Theft (MLaaS):**
1. El atacante consulta el modelo víctima (MLaaS) con inputs variados.
2. Obtiene pares query/etiqueta (datos etiquetados: ship, car, cat...).
3. Usa esos datos etiquetados para entrenar un **modelo clon**.
4. Usa el clon para lanzar ataques secundarios: adversarial attacks, membership inference, model inversion.

Método alternativo: **desensamblar el binario** del código del modelo o acceder a los datos de entrenamiento y al algoritmo directamente.

---

### ML06 — AI Supply Chain Attack

El atacante compromete librerías ML o modelos de terceros usados por el sistema. Estos ataques **pasan desapercibidos durante largo tiempo** porque la víctima no detecta que el paquete está comprometido.

**Ejemplo con librerías Python:**
1. El atacante modifica el código de un paquete del que depende el proyecto ML (ej. **NumPy** o **Scikit-learn**).
2. Sube la versión modificada a un repositorio público (**PyPI**).
3. La víctima descarga e instala el paquete → el código malicioso se instala con él.
4. El código malicioso puede: robar información, modificar resultados, hacer fallar el modelo.

**Flujo de ataque a chatbot bancario:**
Atacante sube modelo envenenado al Model Hub → banco descarga y despliega el chatbot → usuarios reciben desinformación.

---

### ML07 — Transfer Learning Attack

Explota el proceso de **transfer learning**: el atacante introduce un backdoor en un modelo pre-entrenado. Cuando ese modelo se usa como base y se hace fine-tuning para otra tarea, el backdoor se transfiere al modelo final.

**Flujo (Weight Poisoning):**
```
Clean model → Weight Poisoning → Poisoned model → Fine-tuning → Model with Backdoor
```

**Ejemplo**: el atacante entrena un modelo de reconocimiento facial con imágenes manipuladas y transfiere ese conocimiento envenenado al sistema de verificación de identidad de la víctima → el sistema hace predicciones incorrectas.

---

### ML08 — Model Skewing

El atacante contamina los datos de entrenamiento para **desplazar la frontera de clasificación** del modelo — es decir, modifica qué considera "bueno" y qué considera "malo".

**Diferencia respecto a ML02**: Data Poisoning busca comportamiento incorrecto general. Model Skewing busca específicamente **desplazar el límite de decisión** hacia un resultado favorable para el atacante.

**Ejemplo del libro**: el atacante quiere que le aprueben un préstamo → inyecta feedback falso en el sistema indicando que solicitantes de alto riesgo anteriores fueron aprobados → el modelo reajusta su frontera de riesgo → el atacante y perfiles similares pasan a clasificarse como bajo riesgo.

**Aplicación en ciberseguridad**: marcar binarios maliciosos específicos como benignos desplazando la frontera de clasificación.

---

### ML09 — Output Integrity Attack

El atacante **modifica directamente el output** del modelo después de que este lo genera, sin tocar el modelo ni sus datos de entrenamiento. Es un ataque post-inferencia.

**Ejemplo**: un atacante con acceso al output de un modelo de diagnóstico médico modifica las predicciones del modelo → los pacientes reciben diagnósticos incorrectos → tratamientos incorrectos → daño o muerte.

**Diferencia clave respecto a otros ataques de poisoning**: no se altera el modelo ni el entrenamiento; se intercepta y modifica el output ya generado.

---

### ML10 — Model Poisoning

El atacante manipula **directamente los parámetros del modelo** (no los datos de entrenamiento) para causar misclassification en un subconjunto específico de muestras de test.

**Diferencia con ML02 (Data Poisoning)**: ML02 ataca los datos de entrenamiento. ML10 ataca los parámetros/pesos del modelo directamente.

**Ejemplo del libro**: el atacante altera los parámetros de imagen de un modelo de compensación de cheques en un banco → el modelo identifica el carácter "7" como "1" → los importes de los cheques se procesan incorrectamente.

**Flujo**:
```
Modified Data + Incorrect Label → Poisoned Training Data → Training Algorithm → Poisoned ML Model → Incorrect output en subset de testing
```

---

## 2. Exam Traps ⚠️

⚠️ **[ML02 vs. ML10 — dónde ocurre el ataque]** — ML02 (Data Poisoning) ataca los **datos de entrenamiento** (muestras o etiquetas). ML10 (Model Poisoning) ataca los **parámetros/pesos del modelo** directamente. Ambos resultan en comportamiento incorrecto del modelo pero el punto de intervención es distinto.

⚠️ **[ML03 vs. ML05 — qué extrae el atacante]** — ML03 (Model Inversion) extrae **información** del modelo (arquitectura, datos de entrenamiento, PII) usando los outputs. ML05 (Model Theft) extrae los **parámetros del modelo** para clonarlo. ML03 busca información; ML05 busca el modelo en sí.

⚠️ **[ML04 — qué infiere exactamente]** — Membership Inference infiere **si una muestra específica estuvo en el dataset de entrenamiento**, no extrae el modelo ni los datos. La señal es la diferencia de confianza: alta confianza → probablemente en training; baja confianza → probablemente no.

⚠️ **[ML06 — pasa desapercibido]** — La característica definitoria de AI Supply Chain Attack es que pasa **desapercibido durante largo tiempo**. El atacante compromete NumPy/Scikit-learn/PyPI → la víctima instala sin saber. Si el examen pregunta qué caracteriza a este ataque, "difícil de detectar" es la respuesta.

⚠️ **[ML07 — qué explota]** — Transfer Learning Attack explota específicamente el **proceso de transfer learning** (fine-tuning de un modelo pre-entrenado). El backdoor se introduce en el modelo pre-entrenado y sobrevive al fine-tuning. No es un ataque durante el entrenamiento inicial; es un ataque sobre el proceso de reutilización de modelos.

⚠️ **[ML08 vs. ML02 — diferencia de objetivo]** — Ambos manipulan datos de entrenamiento, pero ML02 (Data Poisoning) busca comportamiento incorrecto general. ML08 (Model Skewing) busca específicamente **desplazar la frontera de decisión** entre clases (lo que el modelo considera "bueno" vs. "malo") hacia un resultado favorable para el atacante.

⚠️ **[ML09 — post-inferencia]** — Output Integrity Attack modifica el output **después de que el modelo lo genera**. No toca el modelo, los datos ni los parámetros. Es el único ataque del ML Top 10 que ocurre **después** de la inferencia.

⚠️ **[Input Manipulation vs. Data Poisoning — timing]** — ML01 (Input Manipulation) ocurre en **tiempo de inferencia** (cuando el modelo ya está desplegado). ML02 (Data Poisoning) ocurre en **tiempo de entrenamiento**. El timing del ataque es el diferenciador.

⚠️ **[Model Theft — métodos]** — ML05 puede realizarse por: (1) consultas masivas + análisis de pares query/output (MLaaS), (2) desensamblado del código binario, (3) acceso a datos de entrenamiento + algoritmo. El examen puede preguntar cuál es el método cuando el atacante solo tiene acceso externo a la API → respuesta: consultas masivas + construcción de clon.

⚠️ **[AI Supply Chain — librerías ejemplo]** — El libro cita **NumPy, Scikit-learn y PyPI** explícitamente. El examen puede preguntar por el repositorio comprometido (PyPI) o las librerías específicas.

---

## 3. Nemotécnicos

### OWASP ML Top 10 — orden y nombres
**"Input Data Model Member Theft Supply Transfer Skewing Output Poisoning"**  
ML01 **I**nput · ML02 **D**ata Poisoning · ML03 **M**odel Inversion · ML04 **M**embership · ML05 **T**heft · ML06 **S**upply Chain · ML07 **T**ransfer Learning · ML08 **S**kewing · ML09 **O**utput Integrity · ML10 **M**odel Poisoning

### Timing de los ataques ML — cuándo ocurren
| Fase | Ataques |
|---|---|
| **Tiempo de entrenamiento** | ML02 (Data Poisoning), ML08 (Skewing), ML10 (Model Poisoning), ML07 (Transfer Learning) |
| **Pre-despliegue** | ML06 (Supply Chain) |
| **Tiempo de inferencia** | ML01 (Input Manipulation) |
| **Post-inferencia** | ML09 (Output Integrity) |
| **Cualquier momento** | ML03 (Model Inversion), ML04 (Membership Inference), ML05 (Model Theft) |

### ML02 vs. ML10 — regla de diferenciación
**"02 = datos envenenados | 10 = parámetros envenenados"**

### ML03 vs. ML05 — regla de diferenciación
**"03 = extrae información del modelo | 05 = extrae el modelo"**

### ML06 — característica única
**"Supply Chain = no se nota durante mucho tiempo"** (goes unnoticed for a long time)

---

## 4. Flashcards

**Q:** ¿Cuál es la diferencia entre ML02 (Data Poisoning) y ML10 (Model Poisoning)?  
**A:** **ML02** ataca los **datos de entrenamiento** (muestras o etiquetas) para que el modelo aprenda comportamiento incorrecto. **ML10** ataca directamente los **parámetros/pesos del modelo** para causar misclassification en un subconjunto específico. El punto de intervención es distinto.

---

**Q:** Un atacante añade imágenes de señales de STOP con etiquetas incorrectas al dataset de entrenamiento de un sistema de conducción autónoma. ¿Qué tipo de ataque ML es?  
**A:** **ML02 — Data Poisoning Attack**. El atacante manipula las etiquetas del dataset de entrenamiento para comprometer la precisión del modelo desplegado.

---

**Q:** ¿Cuál es la característica definitoria de un AI Supply Chain Attack (ML06) que lo hace especialmente peligroso?  
**A:** Pasa **desapercibido durante largo tiempo** porque la víctima no detecta que el paquete/librería/modelo que ha instalado ha sido comprometido. El código malicioso se instala silenciosamente junto con la dependencia legítima.

---

**Q:** ¿Cuál es la diferencia entre ML03 (Model Inversion) y ML05 (Model Theft)?  
**A:** **ML03** usa los outputs del modelo para hacer ingeniería inversa y extraer **información** (datos de entrenamiento, PII, arquitectura). **ML05** accede a los parámetros del modelo para **clonarlo** completamente. ML03 busca datos; ML05 busca el modelo.

---

**Q:** Un atacante consulta repetidamente un modelo MLaaS con distintos inputs, recopila los pares query/etiqueta y usa esos datos para entrenar un modelo clon. ¿Qué ataque ML es?  
**A:** **ML05 — Model Theft**. El atacante usa las respuestas del modelo víctima como datos de entrenamiento para construir un clon funcional.

---

**Q:** ¿Qué infiere exactamente un Membership Inference Attack (ML04) y cuál es la señal que usa?  
**A:** Infiere si una muestra de datos específica **formó parte del dataset de entrenamiento** del modelo. La señal: si el modelo da confianza muy alta (~98%) para esa muestra → probablemente estaba en el training set; si da confianza baja y distribuida → probablemente no.

---

**Q:** ¿En qué fase del ciclo de vida del modelo ocurre un Input Manipulation Attack (ML01) y cómo se diferencia de Data Poisoning (ML02)?  
**A:** ML01 ocurre en **tiempo de inferencia** (modelo ya desplegado); el atacante altera los inputs que recibe el modelo. ML02 ocurre en **tiempo de entrenamiento**; el atacante manipula los datos con los que se entrena el modelo. Mismo objetivo (comportamiento incorrecto), distinto timing.

---

**Q:** ¿Cuál es la diferencia conceptual entre ML08 (Model Skewing) y ML02 (Data Poisoning)?  
**A:** Ambos manipulan datos de entrenamiento, pero con objetivos distintos. **ML02** busca comportamiento incorrecto general del modelo. **ML08** busca específicamente **desplazar la frontera de decisión** entre clases (lo que el modelo clasifica como "bueno" vs. "malo") hacia un resultado favorable para el atacante.

---

**Q:** ¿Qué hace un Transfer Learning Attack (ML07) y en qué se diferencia de un Data Poisoning Attack?  
**A:** **ML07** explota el proceso de **transfer learning**: introduce un backdoor en un modelo pre-entrenado mediante weight poisoning. Cuando ese modelo se usa como base para fine-tuning en otra tarea, el backdoor se transfiere al modelo final. **ML02** manipula datos de entrenamiento; ML07 manipula el modelo pre-entrenado antes del fine-tuning.

---

**Q:** ¿Por qué el Output Integrity Attack (ML09) es único respecto al resto del OWASP ML Top 10?  
**A:** Es el único ataque que ocurre **post-inferencia**: el atacante modifica el output **después de que el modelo lo genera**, sin tocar el modelo, sus parámetros ni sus datos de entrenamiento. Los demás ataques intervienen en el modelo o en su proceso de aprendizaje.

---

**Q:** ¿Qué librerías Python y qué repositorio público menciona el libro en el contexto de AI Supply Chain Attacks?  
**A:** Librerías: **NumPy** y **Scikit-learn**. Repositorio público: **PyPI** (Python Package Index). El atacante sube la versión modificada de la librería a PyPI y espera a que la víctima la instale.

---

**Q:** Describe el flujo de Weight Poisoning en un Transfer Learning Attack.  
**A:** `Clean model → Weight Poisoning → Poisoned model → Fine-tuning → Model with Backdoor`. El atacante envenena los pesos del modelo limpio; el backdoor sobrevive al proceso de fine-tuning y está presente en el modelo final desplegado por la víctima.

---

**Q:** En un Membership Inference Attack (ML04), ¿qué diferencia una muestra que formó parte del entrenamiento de una que no?  
**A:** El modelo muestra **mayor confianza** en sus predicciones para muestras vistas durante el entrenamiento. Si el modelo responde con clase 1 al 98% → probablemente en training. Si distribuye la confianza (clase 1 al 89%, otras clases con porcentajes más altos) → probablemente no estaba en el training set.

---

**Q:** ¿Cuál es el ejemplo de Model Poisoning (ML10) del libro y qué consecuencia práctica tiene?  
**A:** Un banco usa un modelo para procesar cheques automáticamente. El atacante altera los parámetros del modelo → el modelo identifica el carácter "7" como "1" → los importes de los cheques se procesan con valores incorrectos (fraude financiero).

---

**Q:** ¿Qué acciones puede realizar el código malicioso introducido en un AI Supply Chain Attack según el libro?  
**A:** Robar información sensible, modificar resultados del modelo, o hacer fallar completamente el modelo ML.

---

**Q:** ¿Qué ataques secundarios puede lanzar un atacante que ha robado exitosamente un modelo ML (ML05)?  
**A:** Con el modelo clonado, el atacante puede lanzar: **Adversarial Attacks (ML01)**, **Membership Inference Attacks (ML04)** y **Model Inversion Attacks (ML03)**. El modelo robado facilita estos ataques porque el atacante tiene acceso completo a la arquitectura.

---

## 5. Confusión frecuente

### ML02 (Data Poisoning) vs. ML10 (Model Poisoning) vs. ML08 (Model Skewing)
- **ML02 (Data Poisoning)**: manipula **datos de entrenamiento** (muestras o etiquetas) → comportamiento incorrecto general del modelo.
- **ML10 (Model Poisoning)**: manipula **parámetros del modelo** directamente → misclassification en subconjunto específico de muestras.
- **ML08 (Model Skewing)**: manipula **distribución de los datos de entrenamiento** para desplazar la **frontera de clasificación** → lo que era "malo" se convierte en "bueno" para el atacante.
- **Criterio**: "etiquetas incorrectas en dataset" → ML02. "Pesos/parámetros del modelo alterados directamente" → ML10. "Desplazar el límite entre bueno y malo del clasificador" → ML08.

---

### ML01 (Input Manipulation) vs. ML09 (Output Integrity)
- **ML01**: el atacante altera el **input** que recibe el modelo → la predicción sale incorrecta desde el inicio.
- **ML09**: el atacante intercepta y modifica el **output** que sale del modelo → la predicción era correcta pero se altera antes de llegar al destinatario.
- **Criterio**: si el ataque ocurre **antes** de que el modelo procese → ML01. Si ocurre **después** de que el modelo ha generado su respuesta → ML09.

---

### ML03 (Model Inversion) vs. ML04 (Membership Inference)
- **ML03**: usa outputs del modelo para extraer **datos o estructura** del modelo (PII de entrenamiento, arquitectura). Objetivo: obtener información del modelo.
- **ML04**: usa el modelo + una muestra para determinar **si esa muestra estaba en el training set**. Objetivo: confirmar si ciertos datos fueron usados para entrenar.
- **Criterio**: si el atacante quiere saber "¿qué datos tiene este modelo?" → ML03. Si quiere saber "¿estaba este registro específico en el training set?" → ML04.

---

### ML06 (AI Supply Chain) vs. ML07 (Transfer Learning)
- **ML06**: compromete **librerías, dependencias o modelos de terceros** antes de que se integren → el sistema víctima instala código malicioso sin saberlo.
- **ML07**: compromete específicamente el **proceso de transfer learning** → envenena el modelo pre-entrenado con backdoors que sobreviven al fine-tuning.
- **Criterio**: si el escenario menciona "librería PyPI", "NumPy", "Scikit-learn" o "modelo en repositorio público" → ML06. Si menciona "fine-tuning" o "modelo pre-entrenado con backdoor" → ML07.

---

### OWASP LLM Top 10 vs. OWASP ML Top 10 — solapamientos aparentes
- **LLM03 (Training Data Poisoning)** ≈ **ML02 (Data Poisoning)**: ambos manipulan datos de entrenamiento. La diferencia es el contexto: LLM03 es específico de Large Language Models; ML02 es para ML en general.
- **LLM05 (Supply Chain)** ≈ **ML06 (AI Supply Chain)**: misma categoría de ataque, distinto contexto (LLMs vs. ML general).
- **LLM10 (Model Theft)** ≈ **ML05 (Model Theft)**: igual concepto, distinto contexto.
- **Criterio**: si el escenario menciona LLMs, NLP, chatbots o arquitecturas transformer → usar OWASP LLM. Si menciona clasificadores, redes neuronales de propósito general o ML clásico → usar OWASP ML.
