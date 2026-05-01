# M01 — AI y ML en Ciberseguridad

## Conceptos base

| Concepto | Clave |
|---|---|
| **AI** | Detecta lo que el antivirus no puede — procesa grandes volúmenes de datos |
| **ML** | Rama de AI — **autoaprendizaje sin programación explícita** — detecta anomalías vs comportamiento normal |

---

## Tipos de ML — 2 tipos, 4 subtipos

| Tipo | Subtipo | Clave |
|---|---|---|
| **Supervised** | Classification | Datos **etiquetados** — clases separadas |
| | Regression | Datos **continuos** — clases no separadas |
| **Unsupervised** | Clustering | Agrupa por similitudes — sin etiquetas |
| | Dimensionality Reduction | Reduce atributos/dimensiones |

> ⚠️ **Supervised vs Unsupervised:** Supervised = con etiquetas · Unsupervised = sin etiquetas, infiere categorías

---

## Aplicaciones AI/ML en Ciberseguridad — 10 aplicaciones

| Aplicación | Clave diferencial |
|---|---|
| **Password & Authentication** | Mejora biometría y reconocimiento facial |
| **Phishing Detection** | Diferencia webs maliciosas de legítimas — más rápido que humano |
| **Threat Detection** | Deep learning sobre datos en tiempo real |
| **Vulnerability Management** | **Predice cómo y cuándo** ocurrirá la explotación |
| **Behavioral Analytics** | Patrones por **usuario concreto** — alerta ante desviaciones |
| **Network Security** | Analiza tráfico y propone políticas automáticamente |
| **AI Antivirus** | **Anomaly detection** — NO signature matching |
| **Fraud Detection** | Anomaly detection en transacciones — distingue legítimas de fraudulentas |
| **Botnet Detection** | Detecta botnets que evaden IDS basados en signatures |
| **AI vs AI** | Detecta ataques augmented-AI antes de comprometer la red |

### ⚠️ Pares que confunden en el examen

| Par | Diferencia |
|---|---|
| **AI Antivirus vs tradicional** | Tradicional = signatures (requiere actualización) · AI = comportamiento anómalo |
| **Behavioral Analytics vs Threat Detection** | Behavioral = anomalías de *usuario concreto* · Threat Detection = amenazas al sistema en general |
| **Botnet Detection vs AI Antivirus** | Botnet = evade IDS · Antivirus = evade AV — mismo mecanismo (signatures), distinto contexto |

---

## AI Tools para Ethical Hackers — Tools clave

| Tool | Función diferencial |
|---|---|
| **ShellGPT** | Genera/completa comandos shell · escribe código · documenta scripts |
| **AutoGPT** | Automatiza ejecución de tareas y procesamiento de datos |
| **WormGPT** | Genera scripts/payloads tipo worm — testing y defensa |
| **DAN prompt** | **Prompt modifier** sobre ChatGPT sin restricciones habituales |
| **FreedomGPT** | **Herramienta standalone** — bypasea filtros de contenido |
| **FraudGPT** | Detecta y previene fraude — análisis de patrones |
| **ChaosGPT** | Simula comportamientos caóticos e impredecibles |
| **PoisonGPT** | **AI supply chain attacks** — introduce modelos maliciosos en sistemas IA |
| **BurpGPT** | Integra Burp Suite con IA — reduce falsos positivos |
| **PentestGPT** | Automatiza fases de pentest y genera reportes |
| **BugBountyGPT** | Bug bounty — integra con plataformas de reporte |
| **Hacking APIs GPT** | Especializado en vulnerabilidades de APIs |

### ⚠️ Pares que confunden en el examen

| Par | Diferencia |
|---|---|
| **WormGPT vs FraudGPT** | Worm = genera malware · Fraud = detecta fraude |
| **FreedomGPT vs DAN** | Freedom = standalone · DAN = prompt modifier sobre ChatGPT |
| **PoisonGPT** | Único enfocado en **AI supply chain** — no en sistemas convencionales |
