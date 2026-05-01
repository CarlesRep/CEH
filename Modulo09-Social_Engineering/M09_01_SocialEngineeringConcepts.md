# M09_01_SocialEngineeringConcepts.md
## CEH v13 — Módulo 09 | Social Engineering Concepts

---

## 1. Conceptos y definiciones

### Qué es la ingeniería social

La ingeniería social es la manipulación psicológica de personas para que divulguen información sensible o realicen acciones que beneficien al atacante. A diferencia de los ataques técnicos, no explota vulnerabilidades de red ni de software: explota la naturaleza humana — confianza, autoridad, miedo, codicia.

El proceso sigue una lógica en tres tiempos:
1. **Recopilación previa**: webs corporativas (IDs, nombres, emails), anuncios y medios (productos, estructura), blogs y foros (información personal y organizativa).
2. **Selección y establecimiento de relación**: el atacante elige objetivos de mayor accesibilidad (empleados descontentos, recepcionistas, help-desk) y construye credibilidad.
3. **Explotación**: extrae credenciales, información financiera, tecnología en uso o planes futuros.

El éxito radica en que **las víctimas no son conscientes del fallo de seguridad**: responder preguntas de un desconocido o contestar un spam se percibe como algo inocuo.

---

### 🔴 Objetivos frecuentes (Common Targets)

| Objetivo | Por qué es valioso para el atacante |
|---|---|
| Recepcionistas / Help-desk | Acceso fácil, predisposición a ayudar, pueden revelar contraseñas o extensiones |
| Ejecutivos de soporte técnico | Se les puede manipular haciéndose pasar por directivos, clientes o proveedores |
| Administradores de sistemas | Tienen versiones de SO, contraseñas de admin, topología de red |
| Usuarios y clientes | Objetivo fácil si el atacante finge ser soporte técnico |
| Proveedores | Conocen infraestructura interna y procesos de la organización |
| Altos directivos (CxOs, RRHH, Finanzas) | Acceso a información económica crítica y decisiones estratégicas |

---

### 🔴 Comportamientos vulnerables — Los 8 principios de influencia

Estos principios son el núcleo del examen. Cada uno describe un mecanismo psicológico explotable:

| Principio | Mecanismo | Ejemplo de explotación |
|---|---|---|
| **Authority (Autoridad)** | Las personas obedecen a figuras de poder | Llamada fingiendo ser el administrador de red pidiendo credenciales |
| **Intimidation (Intimidación)** | El miedo a consecuencias fuerza la acción | "El director va a dar una presentación y necesita los archivos ahora mismo" |
| **Consensus / Social Proof** | Hacemos lo que hacemos los demás | Webs con testimonios falsos que legitiman rogueware |
| **Scarcity (Escasez)** | Lo escaso genera urgencia de decisión | Phishing con "último iPhone disponible, haz clic ya" |
| **Urgency (Urgencia)** | Presión temporal elimina el pensamiento crítico | Ransomware con cuenta atrás; "oferta por tiempo limitado" |
| **Familiarity / Liking** | Confiamos más en quien nos cae bien | Shoulder surfing o tailgating facilitado por simpatía |
| **Trust (Confianza)** | Relación preestablecida reduce la guardia | Falso experto de seguridad que "detectó errores" en tu sistema |
| **Greed (Codicia)** | Promesa de beneficio ilícito | Competidor falso que ofrece recompensa económica por datos internos |

**Distinción Scarcity vs. Urgency**: Scarcity crea la percepción de que el recurso se agota (stock, plazas). Urgency crea la percepción de que el tiempo se agota (cuenta atrás, oferta expirada). Pueden coexistir pero son mecanismos distintos.

---

### Factores que hacen vulnerables a las organizaciones

- **Formación de seguridad insuficiente**: empleados que desconocen las técnicas de ingeniería social.
- **Acceso no regulado a la información**: sin control de acceso ni vigilancia sobre quién toca qué datos.
- **Múltiples unidades geográficas**: dificulta la gestión centralizada y amplía la superficie de ataque humano.
- **Falta de políticas de seguridad**: sin política de cambio de contraseñas, de compartición de información, de privilegios de acceso o de identificación de usuario único.

---

### Por qué la ingeniería social sigue siendo efectiva

- No existe hardware ni software específico que la contrarreste.
- No hay método que garantice protección total.
- Es barata (o gratuita) y fácil de ejecutar.
- Los intentos son difíciles de detectar.
- Los humanos son el eslabón más susceptible: no se puede parchear la psicología humana.

---

### 🔴 Fases de un ataque de ingeniería social

| Fase | Acción |
|---|---|
| 1. Research | Dumpster diving, web corporativa, perfiles de empleados, naturaleza del negocio |
| 2. Select Target | Preferencia por empleados descontentos (más manipulables) |
| 3. Develop Relationship | Construir credibilidad y confianza con el objetivo |
| 4. Exploit Relationship | Extraer credenciales, finanzas, tecnología, planes futuros |

---

### 🔴 Tipos de ingeniería social — Clasificación en 3 categorías

#### Human-based (interacción humana directa)
> Impersonation · Vishing · Eavesdropping · Shoulder Surfing · Dumpster Diving · Reverse Social Engineering · Piggybacking · Tailgating · Diversion Theft · Honey Trap · Baiting · Bait and Switch · Quid Pro Quo · Elicitation

#### Computer-based (sistemas informáticos e Internet)
> Phishing · Spam mail · Instant chat messenger · Pop-up window attacks · Scareware · Deepfake Videos · Voice Cloning

#### Mobile-based (aplicaciones móviles)
> Malicious apps · Repackaging legitimate apps · QRLJacking · Fake security applications · SMiShing (SMS Phishing)

**Regla de clasificación**: si hay interacción cara a cara o por teléfono directo → human-based. Si el vector es un ordenador/web/email → computer-based. Si el vector es un dispositivo móvil o app → mobile-based.

---

### Consecuencias de los ataques de ingeniería social

- Pérdida económica directa
- Daño a la reputación (goodwill)
- Pérdida de privacidad de stakeholders y clientes
- Riesgo de terrorismo (blueprints de instalaciones)
- Demandas y arbitrajes (publicidad negativa)
- Cierre temporal o permanente del negocio

---

## 2. Exam Traps ⚠️

⚠️ **[No existe defensa técnica]** El examen puede ofrecer "firewall", "IDS" o "antivirus" como respuesta a "¿qué protege mejor contra la ingeniería social?". La respuesta correcta es siempre **formación y concienciación de empleados (security awareness training)**.

⚠️ **[Scarcity vs. Urgency]** Son principios distintos. Scarcity = recurso limitado. Urgency = tiempo limitado. El ransomware con cuenta atrás es **Urgency**, no Scarcity. Si el enunciado dice "stock agotado" o "últimas plazas" → Scarcity.

⚠️ **[Consensus / Social Proof — rogueware]** El examen asocia este principio específicamente con **webs con testimonios falsos de antimalware (rogueware)**. Si ves "fake testimonials" o "fake reviews" → Consensus/Social Proof.

⚠️ **[Select Target — empleados descontentos]** El examen pregunta qué tipo de empleado es el objetivo preferido. La respuesta es **disgruntled employees** (empleados descontentos), no los mejor informados ni los de mayor rango.

⚠️ **[Clasificación de técnicas]** Vishing es **human-based**, no computer-based, aunque use el teléfono. SMiShing es **mobile-based**, no computer-based. QRLJacking es **mobile-based**.

⚠️ **[Voice Cloning y Deepfake]** Ambos son **computer-based**, no mobile-based, aunque el resultado se use en llamadas. El criterio es el vector técnico de generación, no el canal de entrega.

⚠️ **[Objetivo: System Administrator]** El examen puede preguntar quién tiene información sobre versiones de SO y contraseñas de administrador. La respuesta es **system administrator**, no el help-desk.

⚠️ **[Fases: orden estricto]** Las cuatro fases son Research → Select Target → Develop Relationship → Exploit Relationship. El examen puede desordénarlas y pedir la secuencia correcta.

⚠️ **[Greed vs. Trust]** Ambos pueden implicar una promesa. La diferencia: **Greed** implica beneficio económico o material para la víctima. **Trust** implica construir credibilidad técnica o profesional previa (sin necesariamente prometer algo).

---

## 3. Nemotécnicos

### Los 8 principios de influencia — "A-I-C-S-U-F-T-G"
**"A Intelligent Cat Suddenly Understood Five Tricks Greedily"**
- **A**uthority
- **I**ntimidation
- **C**onsensus / Social Proof
- **S**carcity
- **U**rgency
- **F**amiliarity / Liking
- **T**rust
- **G**reed

### Las 4 fases del ataque — "RSDE"
**"Ratas Seleccionadas Desarrollan Engaños"**
- **R**esearch
- **S**elect Target
- **D**evelop Relationship
- **E**xploit Relationship

### Tipos de técnicas human-based — 14 técnicas
**"I Very Eagerly Should Dance, Reverse Pirates Tailgate Directly, Honey Baits Bring Quiet Excitement"**
- **I**mpersonation · **V**ishing · **E**avesdropping · **S**houlder Surfing · **D**umpster Diving · **R**everse SE · **P**iggybacking · **T**ailgating · **D**iversion Theft · **H**oney Trap · **B**aiting · **B**ait & Switch · **Q**uid Pro Quo · **E**licitation

### Mobile-based — 5 técnicas
**"MR QFS"** → Malicious apps · Repackaging · QRLJacking · Fake security apps · SMiShing

### Computer-based — 7 técnicas
**"PSI Pop Scare Deep Voice"** → Phishing · Spam · Instant chat · Pop-up · Scareware · Deepfake · Voice Cloning

---

## 4. Flashcards

**Q:** ¿Cuál es la definición técnica de ingeniería social según el CEH?
**A:** El arte de manipular a personas para que divulguen información sensible o realicen acciones que permitan al atacante ejecutar un ataque o cometer fraude.

---

**Q:** ¿Qué tipo de empleado es el objetivo preferido en la fase "Select Target"?
**A:** Empleados descontentos (disgruntled employees), porque son más fáciles de manipular.

---

**Q:** Un atacante llama a una víctima, afirma ser experto en seguridad de XYZ Company, dice haber detectado errores en su sistema y le envía un fichero malicioso para "corregirlos". ¿Qué principio de influencia aplica?
**A:** Trust (Confianza). El atacante construye credibilidad profesional antes de ejecutar el ataque.

---

**Q:** El ransomware muestra una cuenta atrás indicando que los ficheros se borrarán si no se paga en 24h. ¿Qué principio de ingeniería social aplica?
**A:** Urgency (Urgencia). Presión temporal para forzar una decisión sin reflexión.

---

**Q:** Un atacante crea una web con testimonios falsos que recomiendan un antimalware que en realidad es rogueware. ¿Qué principio aplica?
**A:** Consensus / Social Proof (Consenso / Prueba social).

---

**Q:** ¿Cuáles son las tres categorías de ataques de ingeniería social?
**A:** Human-based, Computer-based, Mobile-based.

---

**Q:** ¿A qué categoría pertenece el Vishing?
**A:** Human-based (aunque use el teléfono, implica interacción humana directa).

---

**Q:** ¿A qué categoría pertenece el SMiShing?
**A:** Mobile-based.

---

**Q:** ¿A qué categoría pertenecen Deepfake Videos y Voice Cloning?
**A:** Computer-based.

---

**Q:** ¿A qué categoría pertenece QRLJacking?
**A:** Mobile-based.

---

**Q:** ¿Cuál es la única contramedida efectiva contra la ingeniería social según el CEH?
**A:** La formación y concienciación de los empleados (security awareness training). No existe hardware ni software que la contrarreste completamente.

---

**Q:** Un atacante llama a la recepcionista y dice: "El director necesita sus ficheros ahora mismo o la presentación fracasará". ¿Qué principio aplica?
**A:** Intimidation (Intimidación). Se usa presión emocional y una figura de autoridad para forzar la acción.

---

**Q:** ¿Cuáles son las 4 fases de un ataque de ingeniería social en orden?
**A:** 1. Research the Target Company → 2. Select a Target → 3. Develop a Relationship → 4. Exploit the Relationship.

---

**Q:** ¿Qué objetivo de ingeniería social tiene acceso a versiones de SO y contraseñas de administrador?
**A:** System Administrator (Administrador de sistemas).

---

**Q:** Un atacante se hace pasar por un competidor y ofrece una recompensa económica a un empleado para que revele información interna. ¿Qué principio aplica?
**A:** Greed (Codicia).

---

**Q:** ¿Qué factor organizativo hace más fácil el acceso a información sensible por parte de un atacante?
**A:** Unregulated access to information (acceso no regulado a la información) y la falta de políticas de seguridad.

---

**Q:** Un empleado permite que un desconocido entre tras él por una puerta de seguridad porque le cae bien. ¿Qué principio de influencia facilita este ataque?
**A:** Familiarity / Liking (Familiaridad / Simpatía).

---

**Q:** ¿Qué fuentes usa el atacante en la fase de Research de un ataque de ingeniería social?
**A:** Web corporativa, anuncios en medios, blogs/foros de empleados, dumpster diving y perfil de empleados.

---

## 5. Confusión frecuente

### Piggybacking vs. Tailgating
- **Piggybacking**: el atacante entra a una zona restringida **con el conocimiento y consentimiento** (aunque engañado) de un empleado autorizado.
- **Tailgating**: el atacante sigue de cerca a un empleado autorizado **sin su conocimiento**, aprovechando que la puerta está abierta.
- **Criterio de decisión**: ¿la persona autorizada sabe que el atacante entra? Sí → Piggybacking. No → Tailgating.

---

### Scarcity vs. Urgency
- **Scarcity**: el recurso es limitado ("quedan 2 unidades", "últimas plazas").
- **Urgency**: el tiempo es limitado ("oferta válida 1 hora", cuenta atrás del ransomware).
- **Criterio de decisión**: ¿la presión viene del agotamiento del objeto o del agotamiento del tiempo? Objeto → Scarcity. Tiempo → Urgency.

---

### Authority vs. Intimidation
- **Authority**: el atacante se presenta como una figura con poder legítimo (administrador, directivo) y pide colaboración.
- **Intimidation**: el atacante usa amenazas o presión emocional para forzar la acción. A menudo implica consecuencias negativas si no se actúa.
- **Criterio de decisión**: ¿hay amenaza o consecuencia negativa implícita? Sí → Intimidation. Solo rango/poder → Authority.

---

### Phishing vs. SMiShing vs. Vishing
- **Phishing**: vector email/web → computer-based.
- **SMiShing**: vector SMS → mobile-based.
- **Vishing**: vector llamada de voz → human-based.
- **Criterio de decisión**: el canal determina la categoría, no el objetivo del ataque.

---

### Trust vs. Consensus / Social Proof
- **Trust**: el atacante construye credibilidad **con la víctima directamente** (relación uno a uno).
- **Consensus**: el atacante usa **la opinión de terceros** (testimonios, reviews) para influir en la víctima.
- **Criterio de decisión**: ¿la credibilidad viene de una relación directa o de referencias de otros? Directa → Trust. Terceros → Consensus.
