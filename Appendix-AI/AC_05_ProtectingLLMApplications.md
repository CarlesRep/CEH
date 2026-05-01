# AC_05 — Protecting LLM Applications: Countermeasures

---

## 1. Conceptos y Definiciones

### 🔴 Contramedidas por vulnerabilidad OWASP LLM — tabla maestra

| Vulnerabilidad | Contramedidas clave |
|---|---|
| **LLM01 Prompt Injection** | Privilege control (RBAC); human approval para operaciones sensibles; segregación de contenido (filtrado + sanitización + separación por capas de confianza); tratar el LLM como componente no confiable |
| **LLM02 Insecure Output Handling** | Zero-trust: tratar el output del LLM como si fuera input de usuario; seguir OWASP ASVS; output encoding (HTML entity, URL, base64) |
| **LLM03 Training Data Poisoning** | Verificación de supply chain de datos; registro MLhOM de fuentes; validación de calidad/precisión/relevancia; modelos separados por caso de uso |
| **LLM04 Model DoS** | Input validation (tipo, longitud, formato); content filtering; resource caps (CPU, memoria, disco); API rate limits (por cuenta o IP); queue management; resource monitoring |
| **LLM05 Supply Chain** | Evaluación de proveedores; testing de plugins antes de integrar; actualizar y parchear componentes; inventario de componentes; code signing |
| **LLM06 Sensitive Info Disclosure** | Data scrubbing (eliminar/enmascarar PII en training); input validation; fine-tuning con salvaguardas y cifrado; data access controls + autenticación + cifrado |
| **LLM07 Insecure Plugin Design** | Type checks + validation layer (parameter control); OWASP ASVS; SAST/DAST/IAST testing; least privilege (ASVS); OAuth2 + API Keys; confirmación de usuario para acciones sensibles |
| **LLM08 Excessive Agency** | Limitar funciones de plugins a esenciales; scope control; granular functionality; mínimos permisos; autenticación de usuario robusta; **human-in-the-loop**; downstream authorization |
| **LLM09 Overreliance** | Monitor + validate outputs; cross-check con fuentes confiables; fine-tuning específico por tarea; auto-validación contra hechos conocidos; task segmentation; comunicar limitaciones del LLM; interfaces con warnings |
| **LLM10 Model Theft** | Autenticación fuerte para acceso a ficheros del modelo; segmentación de red; monitorización de logs; MLOps automation (cifrar modelo + código, DLP, ofuscación de código, seguridad física) |

---

### Mitigación de Prompt Injection — cuatro pilares

**1. Privilege Control**: RBAC para limitar acceso al LLM → solo usuarios/entidades autorizadas pueden ejecutar acciones privilegiadas.

**2. Human Approval**: operaciones sensibles requieren revisión y autorización humana antes de ejecutarse.

**3. Segregation of Content**: separar contenido no confiable de los prompts de usuario mediante:
- Filtrado y sanitización de inputs.
- Separación en capas/categorías por nivel de confianza.
- Políticas estrictas de separación de contenido.

**4. Trust Boundaries**: tratar el LLM como componente **no confiable**; mostrar advertencias visuales cuando los outputs se consideren sospechosos; el usuario debe verificar antes de actuar.

---

### Best Practices contra Prompt Injection — lista completa 🔴

1. La interacción usuario↔LLM es un **trust boundary bidireccional**: ni el input del usuario ni el output del LLM deben ser confiados.
2. Asegurarse de que el LLM **no tenga acceso a información secreta**.
3. **Restringir acceso a plugins** que puedan ser secuestrados.
4. Eliminar etiquetas especializadas de los inputs.
5. Instruir al LLM sobre prompt injections mediante **meta prompts**.
6. **Registrar inputs y outputs** para detectar prompt injection, data leakage y comportamiento indeseable.
7. Implementar **IAM y Authorization** con principio de mínimo privilegio (least privilege).
8. **Escanear modelos** con herramientas como Model Scan para identificar intentos de code injection.
9. **Cifrar modelos en reposo** para prevenir lectura/escritura tras una infiltración exitosa.
10. **Cifrar modelos en tránsito con TLS o mTLS** para todas las conexiones HTTP/TCP → protección contra MITM.
11. Almacenar y verificar **checksum** al cargar modelos para garantizar la integridad del fichero.
12. Mantener integridad y autenticidad del modelo mediante **firma criptográfica**.
13. Asegurar que los modelos ML almacenados tienen **acceso autenticado**.

---

### Prevención de Insecure Output Handling

**Zero-Trust para outputs LLM**: tratar el output del LLM exactamente igual que se trataría input de un usuario no confiable — validar y sanitizar antes de cualquier procesamiento o display.

**Output Encoding** (contra XSS y otros riesgos):
- **HTML entity encoding**: `<` → `&lt;` etc.
- **URL encoding**
- **Base64 encoding**

Seguir **OWASP ASVS** (Application Security Verification Standard) para validación y sanitización.

---

### Prevención de Model DoS — cinco mecanismos

1. **Input Validation**: tipo de dato, límites de longitud, adherencia a formato.
2. **Content Filtering**: detectar y filtrar inputs maliciosos o malformados.
3. **Resource Caps**: limitar CPU, memoria, disco I/O por request.
4. **API Rate Limits**: por cuenta de usuario o dirección IP → previene flooding.
5. **Queue Management**: priorizar tareas críticas; prevenir sobrecarga por peticiones concurrentes.
6. **Resource Monitoring**: monitorización continua de uso de recursos y métricas de rendimiento.

---

### Prevención de Insecure Plugin Design

Herramientas de testing para plugins:
- **SAST** (Static Application Security Testing)
- **DAST** (Dynamic Application Security Testing)
- **IAST** (Interactive Application Security Testing)

Autenticación en plugins: **OAuth2 + API Keys**.

---

### Prevención de Excessive Agency — diferenciadores clave

El concepto de **Human-in-the-Loop** es la contramedida específica de LLM08: requiere aprobación humana para acciones del LLM, añadiendo una capa de supervisión y control para operaciones críticas o sensibles.

**Downstream Authorization**: implementar mecanismos de autorización en los sistemas downstream para garantizar que las acciones iniciadas por el LLM estén autorizadas y alineadas con las políticas organizacionales.

---

### Herramientas de seguridad LLM 🔴

| Herramienta | Función | Instalación/URL |
|---|---|---|
| **LLM Guard** | Toolkit para seguridad LLM en producción: evaluación de input/output, sanitización, detección de contenido dañino, prevención de data leakage, protección contra prompt injection y jailbreak | `pip install llm-guard` / llm-guard.com |
| **Lakera** (Chrome extension) | Previene compartir información sensible con ChatGPT: detecta credit card numbers, nombres, emails, teléfonos, direcciones, SSN, secret keys | lakera.ai |
| **Rebuff** | Protección contra prompt injection | rebuff.ai |
| **Lasso Security** | Seguridad LLM | lasso.security |
| **BurpGPT** | Extensión de Burp Suite para testing de seguridad LLM | burpgpt.app |
| **Garak** | Seguridad LLM | garak.ai |
| **Whylabs** | Monitorización y seguridad LLM | whylabs.ai |
| **Prompt Security** | Protección de prompts | prompt.security |

**LLM Guard — uso con código Python:**
```python
# Input scanner (ej. bloquear temas de violencia)
from llm_guard.input_scanners import BanTopics
scanner = BanTopics(topics=["violence"], threshold=0.5)
sanitized_prompt, is_valid, risk_score = scanner.scan(prompt)

# Output scanner (ej. detectar sesgo)
from llm_guard.output_scanners import Bias
scanner = Bias(threshold=0.5)
sanitized_output, is_valid, risk_score = scanner.scan(prompt, model_output)
```

**Lakera** detecta y bloquea: números de tarjeta de crédito, nombres angloparlantes, emails, teléfonos, direcciones postales de EE.UU., SSN de EE.UU., secret keys.

---

### MLOps — contramedidas para Model Theft

Conjunto de medidas dentro de MLOps automation:
- Cifrar datos y código del modelo.
- Seguridad física del entorno de almacenamiento.
- **DLP (Data Loss Prevention)**: evitar que usuarios no autorizados transfieran ficheros del modelo.
- **Code obfuscation**: ocultar parámetros críticos del modelo.
- Segmentación de red: aislar el LLM de recursos y APIs innecesarios.

---

## 2. Exam Traps ⚠️

⚠️ **[LLM02 — zero-trust para outputs]** — La contramedida central de LLM02 es tratar el output del LLM **como si fuera input de usuario no confiable**. No basta con validar el input; el output del LLM también debe validarse antes de usarse. El examen puede preguntar "¿qué principio se aplica al output del LLM?" → zero-trust.

⚠️ **[Output Encoding — tres tipos]** — Las técnicas de output encoding son **HTML entity encoding, URL encoding y base64 encoding**. El examen puede añadir "AES encryption" como trampa — el cifrado no es output encoding para prevenir XSS.

⚠️ **[LLM04 Rate Limits — granularidad]** — Los API rate limits se pueden aplicar **por cuenta de usuario o por dirección IP**. El examen puede preguntar la granularidad disponible.

⚠️ **[LLM07 — herramientas de testing]** — El plugin testing usa **SAST, DAST e IAST**. Son tres métodos distintos de análisis de seguridad de aplicaciones. El examen puede omitir IAST o incluir RAST como trampa.

⚠️ **[LLM07 — autenticación]** — La autenticación para plugins usa **OAuth2 + API Keys**, no certificados SSL ni LDAP. Si el examen pregunta el mecanismo de autenticación para LLM agents, la respuesta es OAuth2 + API Keys.

⚠️ **[LLM08 — human-in-the-loop]** — Es la contramedida distintiva de Excessive Agency. No es solo "limitar permisos"; es añadir supervisión humana activa para operaciones críticas del LLM. Si el examen pregunta qué contramedida añade "oversight" humano sobre acciones del LLM → human-in-the-loop.

⚠️ **[LLM03 — MLhOM records]** — Los registros de fuentes de datos de entrenamiento se llaman **MLhOM records** en el libro. Dato específico que puede aparecer en preguntas de identificación de término.

⚠️ **[LLM05 — code signing]** — La contramedida de autenticidad/integridad en Supply Chain es **code signing**. No confundir con checksum (que es para LLM01 best practice nº 11) aunque ambos verifican integridad.

⚠️ **[LLM10 — DLP como contramedida]** — DLP (Data Loss Prevention) se menciona explícitamente como contramedida para Model Theft. Si el examen pregunta qué tecnología previene la transferencia no autorizada de ficheros del modelo → DLP.

⚠️ **[Lakera — categorías de datos detectados]** — Lakera detecta: credit card numbers, nombres angloparlantes, emails, teléfonos, direcciones postales de EE.UU., SSN de EE.UU. y secret keys. El examen puede preguntar qué tipo de dato detecta o no detecta.

⚠️ **[LLM Guard — instalación]** — `pip install llm-guard`. Evalúa tanto inputs como outputs. Usa `input_scanners` y `output_scanners` como módulos distintos.

---

## 3. Nemotécnicos

### Cuatro pilares de mitigación de Prompt Injection
**"PHST"** = **P**rivilege Control · **H**uman Approval · **S**egregation of Content · **T**rust Boundaries

### LLM04 DoS — seis contramedidas
**"IVCRQM"** = **I**nput Validation · **C**ontent Filtering · **R**esource Caps · Rate limits (API) · **Q**ueue Management · **M**onitoring

### Best Practices Prompt Injection — los tres que usan cifrado (nº 9, 10, 11)
**"Reposo-Tránsito-Checksum"**  
→ Cifrar en reposo (9) · TLS/mTLS en tránsito (10) · Verificar checksum al cargar (11)

### LLM07 testing — "SDI"
**S**AST (static) · **D**AST (dynamic) · **I**AST (interactive)

### LLM Guard — dos módulos
`input_scanners` (BanTopics, etc.) + `output_scanners` (Bias, etc.)

### Herramientas LLM Security — "LLBGWP" + Lakera
**L**LM Guard · **L**asso · **B**urpGPT · **G**arak · **W**hylabs · **P**rompt Security + **R**ebuff + **Lakera** (Chrome)

---

## 4. Flashcards

**Q:** ¿Cuál es el principio de seguridad central para mitigar LLM02 (Insecure Output Handling)?  
**A:** **Zero-trust**: tratar el output del LLM exactamente como si fuera input de un usuario no confiable — validar y sanitizar antes de cualquier procesamiento o display posterior.

---

**Q:** ¿Qué tres técnicas de output encoding se recomiendan para prevenir XSS en LLM02?  
**A:** **HTML entity encoding, URL encoding y base64 encoding**. Se usan para sanitizar y escapar caracteres especiales, scripts y contenido potencialmente dañino en el output del LLM.

---

**Q:** ¿Cuáles son los cuatro pilares de mitigación de Prompt Injection (LLM01)?  
**A:** (1) **Privilege Control** (RBAC); (2) **Human Approval** para operaciones sensibles; (3) **Segregation of Content** (filtrado + sanitización + separación por capas de confianza); (4) **Trust Boundaries** (tratar el LLM como componente no confiable + alertas visuales).

---

**Q:** ¿Qué granularidad tienen los API rate limits como contramedida contra Model DoS (LLM04)?  
**A:** Por **cuenta de usuario** o por **dirección IP**. Controlan la frecuencia y volumen de peticiones al LLM para prevenir flooding.

---

**Q:** ¿Qué herramienta de seguridad LLM se instala con `pip install llm-guard` y qué dos tipos de controles ofrece?  
**A:** **LLM Guard**. Ofrece **Input Controls** (sanitización, detección de contenido dañino, protección contra prompt injection y jailbreak) y **Output Controls** (prevención de data leakage, detección de sesgo).

---

**Q:** ¿Qué tecnología de prevención de Model Theft (LLM10) impide que usuarios no autorizados transfieran ficheros del modelo?  
**A:** **DLP (Data Loss Prevention)**. Es parte del conjunto de medidas MLOps Automation para proteger el modelo.

---

**Q:** ¿Cuáles son las tres herramientas de testing recomendadas para prevenir Insecure Plugin Design (LLM07)?  
**A:** **SAST** (Static Application Security Testing), **DAST** (Dynamic Application Security Testing) e **IAST** (Interactive Application Security Testing).

---

**Q:** ¿Qué mecanismo de autenticación se recomienda específicamente para LLM agents en el contexto de LLM07?  
**A:** **OAuth2 + API Keys** para autenticación y autorización de usuarios y aplicaciones que acceden a los LLM agents.

---

**Q:** ¿Qué contramedida específica de LLM08 (Excessive Agency) añade supervisión humana activa sobre acciones críticas del LLM?  
**A:** **Human-in-the-Loop**: requiere aprobación humana para acciones del LLM en operaciones críticas o sensibles, permitiendo revisar, validar e intervenir antes de la ejecución.

---

**Q:** ¿Qué son los MLhOM records en el contexto de prevención de Training Data Poisoning (LLM03)?  
**A:** Registros de fuentes de datos, transformaciones y pasos de preprocesamiento usados en el entrenamiento del LLM. Permiten trazar y verificar la procedencia e integridad de los datos de entrenamiento.

---

**Q:** ¿Qué tipos de datos sensibles detecta la extensión de Chrome Lakera para proteger contra LLM06?  
**A:** Números de tarjeta de crédito, nombres angloparlantes, emails, teléfonos, direcciones postales de EE.UU., SSN de EE.UU. y secret keys.

---

**Q:** ¿Cuáles son las dos contramedidas de cifrado incluidas en las best practices contra Prompt Injection del libro?  
**A:** (1) **Cifrar modelos en reposo** para prevenir lectura/escritura tras infiltración; (2) **Cifrar modelos en tránsito con TLS o mTLS** en todas las conexiones HTTP/TCP para proteger contra MITM.

---

**Q:** ¿Cuál es la contramedida de autenticidad/integridad recomendada en Supply Chain (LLM05)?  
**A:** **Code signing** — verificar la autenticidad e integridad de modelos LLM y código mediante firmas criptográficas. Diferente del checksum (que verifica integridad del fichero al cargarlo, best practice nº 11 de Prompt Injection).

---

**Q:** ¿Qué contramedida de LLM08 implementa autorización en sistemas externos para validar acciones del LLM?  
**A:** **Downstream Authorization**: implementar mecanismos de autorización en los sistemas downstream para garantizar que las acciones iniciadas por el LLM estén autorizadas y alineadas con políticas organizacionales.

---

**Q:** ¿Cuáles son las seis contramedidas contra Model DoS (LLM04) en orden?  
**A:** (1) Input Validation, (2) Content Filtering, (3) Resource Caps (CPU/memoria/disco), (4) API Rate Limits (por usuario o IP), (5) Queue Management (priorizar tareas críticas), (6) Resource Monitoring (monitorización continua).

---

## 5. Confusión frecuente

### Human Approval (LLM01) vs. Human-in-the-Loop (LLM08)
- **Human Approval (LLM01)**: revisión humana de prompts o operaciones sensibles **antes de ejecutarlos** — es un control de entrada.
- **Human-in-the-Loop (LLM08)**: supervisión humana activa sobre las **acciones que el LLM ejecuta** — es un control sobre la agencia del sistema. Añade una capa de oversight en tiempo de ejecución de acciones.
- **Criterio**: si el escenario habla de "revisar un prompt antes de enviarlo al LLM" → Human Approval (LLM01). Si habla de "aprobar las acciones que el LLM toma en nombre del usuario" → Human-in-the-Loop (LLM08).

---

### Code Signing vs. Checksum — ambos en supply chain/integridad
- **Code Signing** (LLM05): firma criptográfica del código/modelo → verifica **autenticidad** (quién lo firmó) + integridad. Se usa en el proceso de distribución.
- **Checksum** (best practice nº 11 de LLM01): valor hash del fichero → verifica **integridad** al cargar el modelo. No garantiza autenticidad (quién lo creó).
- **Criterio**: si la pregunta menciona "verificar quién publicó el modelo" → Code Signing. Si menciona "verificar que el fichero no se ha corrompido al cargarlo" → Checksum.

---

### LLM Guard vs. Lakera — propósito y contexto
- **LLM Guard**: toolkit para desarrolladores/equipos de seguridad; protege en **producción** con input/output scanners; instalable via pip; detecta temas prohibidos, sesgo, prompt injection, jailbreak.
- **Lakera**: extensión de Chrome para **usuarios finales** de ChatGPT; detecta datos sensibles que el usuario podría compartir inadvertidamente (PII, tarjetas, SSN, keys).
- **Criterio**: si el escenario es sobre proteger una aplicación LLM en producción → LLM Guard. Si es sobre proteger al usuario de compartir datos con ChatGPT → Lakera.

---

### SAST vs. DAST vs. IAST en LLM07
- **SAST** (Static): analiza el código **sin ejecutarlo** — detecta vulnerabilidades en el código fuente del plugin.
- **DAST** (Dynamic): analiza la aplicación **en ejecución** — simula ataques desde fuera.
- **IAST** (Interactive): combina ambos — análisis desde dentro de la aplicación **durante la ejecución** con agentes instrumentados.
- **Criterio**: "análisis de código sin ejecutar" → SAST. "Simular ataques contra la app en ejecución" → DAST. "Agentes dentro de la aplicación durante las pruebas" → IAST.

---

### Resource Caps vs. API Rate Limits (LLM04)
- **Resource Caps**: limitan los **recursos del sistema** (CPU, memoria, disco) que una sola petición puede consumir — previenen agotamiento de recursos por peticiones complejas.
- **API Rate Limits**: limitan la **frecuencia y volumen** de peticiones — previenen flooding con muchas peticiones en poco tiempo.
- **Criterio**: si el ataque es "una sola petición que consume todos los recursos" → Resource Caps. Si el ataque es "miles de peticiones en pocos segundos" → API Rate Limits.
