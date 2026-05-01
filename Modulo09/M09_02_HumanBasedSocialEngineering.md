# M09_02_HumanBasedSocialEngineering.md
## CEH v13 — Módulo 09 | Human-based Social Engineering Techniques

---

## 1. Conceptos y definiciones

### Marco general

Las técnicas human-based implican interacción directa persona a persona (presencial, telefónica o VoIP). El atacante explota mecanismos psicológicos como la reciprocidad, el respeto a la autoridad, la cortesía o la curiosidad. El vector es siempre la conducta humana, no un sistema informático.

---

### 🔴 Impersonation — Tipos y subtipos

Impersonation es la técnica raíz de la mayoría de ataques human-based. El atacante finge ser alguien legítimo para obtener acceso o información.

| Tipo de impersonation | Mecanismo psicológico explotado | Ejemplo clave |
|---|---|---|
| Legitimate end-user | Reciprocidad ("un favor merece otro") | "Soy John de finanzas, olvidé mi contraseña" |
| Important user | No cuestionar la autoridad + intimidación | "Soy la secretaria del CFO, necesito tu ayuda urgente" |
| Technical support agent | Confianza técnica + ignorancia del usuario | "Soporte técnico: tuvimos un crash, dame tu ID y password" |
| Internal employee / client / vendor | Acceso físico no vigilado | Entra con ropa de trabajo, busca contraseñas en escritorios |
| Repairman | Personas de mantenimiento no se cuestionan | Electricista / técnico que implanta dispositivos de escucha |
| Third-party authorization | Credibilidad delegada + verificación imposible | "John me dijo que me darías la info mientras está de vacaciones" |
| Trusted authority figure | Máxima efectividad: autoridad + urgencia | Auditor, inspector, director de ventas con clientes en el coche |

**Reciprocación**: regla social por la que un favor genera obligación de devolver otro favor, incluso si el favor original no fue solicitado. Los ingenieros sociales la explotan sistemáticamente en entornos corporativos.

---

### Vishing (Voice/VoIP Phishing)

Vishing es impersonation por voz. El atacante usa VoIP y **caller ID spoofing** para falsificar la identificación. Los vectores concretos son:

- Mensajes pregrabados que imitan a entidades financieras legítimas.
- SMS o email previo que induce a la víctima a llamar a un número controlado por el atacante.
- Llamada directa del atacante.

El objetivo típico: nombre completo, fecha de nacimiento, número de seguridad social, número de cuenta bancaria, número de tarjeta de crédito, credenciales (usuario/contraseña).

**Subvariante — Abusing help desk over-helpfulness**: el personal de help desk está entrenado para ser útil y frecuentemente resetea contraseñas sin verificar la identidad del solicitante. El atacante solo necesita conocer el nombre del empleado a suplantar.

**Subvariante — Third-party authorization por vishing**: más efectiva cuando la figura de autoridad referenciada está de vacaciones o viajando, haciendo imposible la verificación inmediata.

**Subvariante — Tech support vishing**: el atacante finge ser soporte técnico del proveedor de software de la organización, solicita credenciales para resolver un problema de red ficticio.

**Subvariante — Trusted authority figure**: el más efectivo. Imita a un auditor externo, inspector de seguridad contra incendios, superintendente. Usar terminología técnica específica (p. ej., "HVAC") añade credibilidad suficiente para acceder a zonas restringidas.

---

### Eavesdropping

Escucha no autorizada de conversaciones o lectura de mensajes ajenos. Puede ser:
- **Audio**: conversaciones telefónicas, reuniones.
- **Vídeo**: captación visual de información.
- **Escrito**: emails, mensajería instantánea, faxes.

Información obtenible: contraseñas, planes de negocio, números de teléfono, direcciones.

---

### Shoulder Surfing

Observar a una persona mientras introduce información sensible en un dispositivo. Técnicas de apoyo: prismáticos, dispositivos ópticos, cámaras de pequeño tamaño instaladas para grabar las acciones del sistema.

Información objetiva: contraseñas, PINs, números de cuenta, datos de autenticación.

---

### 🔴 Dumpster Diving

Búsqueda de información sensible en cubos de basura físicos. Toda la información descartada sin destrucción adecuada es recuperable.

| Tipo de material | Información extraíble |
|---|---|
| Listas de teléfonos | Nombres y contactos de empleados |
| Organigramas | Estructura corporativa, salas de servidores, zonas restringidas |
| Emails impresos, notas, faxes, memos | Contraseñas, operaciones internas, contactos |
| Manuales de política | Procedimientos de empleo, uso de sistemas, operaciones |
| Calendarios, notas de eventos, logs | Horarios de login/logout → ventana óptima de ataque |
| Estados de cuenta, nóminas, previsiones de ventas | Información financiera crítica |
| Diagramas de red | Topología de red, direccionamiento |
| Código fuente | Lógica de aplicaciones internas |

**Combinación habitual**: el atacante usa un pretexto (fingir ser técnico, limpiador, reparador) para justificar su presencia en la zona de basura o para tener acceso físico al edificio.

---

### Reverse Social Engineering

El atacante invierte el flujo: en lugar de pedir información, **se posiciona como el experto al que la víctima acude voluntariamente**. Requiere más preparación y habilidad que el resto de técnicas.

Fases:
1. **Sabotage**: el atacante provoca o simula un incidente técnico (corrompe una estación de trabajo o la hace parecer defectuosa).
2. **Marketing**: se asegura de que la víctima lo llame a él. Deja tarjetas de visita en la oficina o coloca su número de contacto en el propio mensaje de error.
3. **Support**: aunque ya tiene la información, continúa prestando asistencia para no levantar sospechas.

---

### 🔴 Piggybacking vs. Tailgating

| | Piggybacking | Tailgating |
|---|---|---|
| **Definición** | Entrada con el **consentimiento** (aunque engañado) del empleado autorizado | Entrada **sin consentimiento**, siguiendo físicamente a alguien |
| **Mecanismo** | Cortesía forzada: "olvidé mi tarjeta, ¿me abres?" | El empleado abre la puerta y el atacante se cuela detrás |
| **Conciencia de la víctima** | La víctima sabe que deja pasar al atacante | La víctima no sabe que está dejando pasar al atacante |

---

### Diversion Theft

También conocido como **"Round the Corner Game"** o **"Cornet Game"**. El objetivo es redirigir una entrega legítima a una ubicación incorrecta. El vector físico habitual es el conductor de reparto; el vector online implica persuadir a alguien para enviar ficheros confidenciales a un destinatario no autorizado.

---

### Honey Trap

El atacante crea una identidad online atractiva y establece una relación romántica o de amistad falsa con un insider de la organización objetivo. El objetivo es extraer información confidencial de alguien que ya tiene acceso legítimo a ella.

---

### 🔴 Baiting

El atacante explota **curiosidad + codicia** dejando un dispositivo físico (generalmente un USB) en lugares de alto tráfico (aparcamientos, ascensores, aseos). El dispositivo lleva el logotipo de una empresa legítima y una etiqueta atractiva (p. ej., "Employee Salary Information 2024").

Flujo del ataque:
1. Víctima encuentra el USB.
2. Lo conecta a su sistema (curiosidad/codicia).
3. Se descarga y ejecuta un fichero malicioso.
4. El atacante obtiene acceso remoto o instala malware.

**Distinción clave Baiting vs. Quid Pro Quo**: en Baiting, el gancho es un objeto físico o promesa unilateral. En Quid Pro Quo, el atacante ofrece activamente un servicio a cambio de credenciales.

---

### Quid Pro Quo

Traducción literal: "algo a cambio de algo". El atacante llama a números aleatorios de una organización fingiendo ser del departamento de TI. Busca empleados con problemas técnicos reales y les ofrece resolverlos a cambio de credenciales o ejecutando comandos maliciosos en su sistema.

Diferencia con Baiting: en Quid Pro Quo el intercambio es **activo y bidireccional** (servicio a cambio de información). En Baiting el gancho es **pasivo y unilateral** (objeto dejado para que la víctima lo recoja).

---

### Elicitation

Extracción de información mediante conversaciones aparentemente inocentes y socialmente normales. Requiere habilidades sociales avanzadas. El atacante no pide información directamente, sino que guía la conversación de forma que la víctima la revele voluntariamente sin darse cuenta.

---

### Bait and Switch

El atacante captura la atención de la víctima con una oferta atractiva (precio bajo, producto deseado) a través de un enlace o descarga. Al hacer clic, el atacante ejecuta su objetivo real: instalar malware, robar datos o comprometer la privacidad.

**Objetivo principal**: clientes de e-commerce.

**Diferencia con Baiting**: Bait and Switch opera online y culmina con una transacción o descarga engañosa. Baiting usa dispositivos físicos (USB) y explota la curiosidad/codicia de forma offline.

---

## 2. Exam Traps ⚠️

⚠️ **[Vishing — clasificación]** Vishing es **human-based**, no computer-based, aunque use VoIP o teléfono. El criterio es la interacción humana directa por voz.

⚠️ **[Piggybacking vs. Tailgating]** El examen confunde deliberadamente ambos. Criterio único: ¿la víctima sabe que deja pasar al atacante? Sí (aunque engañada) → Piggybacking. No → Tailgating.

⚠️ **[Reverse Social Engineering — quién pide información]** En todas las demás técnicas, el atacante pide información a la víctima. En Reverse Social Engineering, **la víctima acude al atacante** a pedir ayuda. Es el único caso con este flujo invertido.

⚠️ **[Dumpster Diving — es human-based]** Aunque no implique conversación, dumpster diving está clasificado como human-based en el CEH. No es computer-based.

⚠️ **[Baiting — vector físico]** El USB es el ejemplo canónico del examen. El ataque no requiere interacción verbal con la víctima. El examen puede presentar Baiting como "USB en el aparcamiento" → identificarlo correctamente.

⚠️ **[Quid Pro Quo — números aleatorios]** El detalle que lo distingue en el examen: el atacante llama a **números aleatorios** hasta encontrar alguien con un problema real. No hay targeting previo del individuo.

⚠️ **[Third-party Authorization — condición de efectividad]** Es más efectivo cuando la figura de autoridad referenciada **está de vacaciones o de viaje**, porque la verificación inmediata es imposible. El examen puede preguntar cuándo es más probable que funcione.

⚠️ **[Diversion Theft — nombre alternativo]** También llamado "Round the Corner Game" o "Cornet Game". Si el examen presenta este nombre, hace referencia a Diversion Theft.

⚠️ **[Honey Trap — insider]** El objetivo del Honey Trap es siempre un **insider** de la organización que ya posee información crítica. No es un ataque a sistemas externos.

⚠️ **[Bait and Switch — objetivo principal]** El examen puede preguntar el objetivo principal de Bait and Switch: **clientes de e-commerce**. No es un ataque genérico de phishing.

⚠️ **[Trusted Authority Figure — terminología técnica]** El uso de términos técnicos específicos (p. ej., HVAC) es suficiente para añadir credibilidad y conseguir acceso físico. El examen puede preguntar por qué esta subvariante es la más efectiva.

---

## 3. Nemotécnicos

### 14 técnicas human-based — Orden de memoria
**"I Eavesdrop Shoulders Dumping Reverse Pirates Tailgating Divert Honey Bees Quit Every Battle"**
- **I**mpersonation · **E**avesdropping · **S**houlder Surfing · **D**umpster Diving · **R**everse SE · **P**iggybacking · **T**ailgating · **D**iversion Theft · **H**oney Trap · **B**aiting · **Q**uid Pro Quo · **E**licitation · **B**ait & Switch

*(Nota: Vishing está dentro de Impersonation como subtipo)*

### Fases de Reverse Social Engineering — "SaMS"
**S**abotage → **M**arketing → **S**upport

### Subtipos de Vishing/Impersonation — "LAT 3T"
- **L**egitimate end-user
- **A**uthority / Important user  
- **T**ech support (presencial)
- **T**hird-party authorization
- **T**ech support (vishing)
- **T**rusted authority figure

### Piggybacking vs. Tailgating — regla de bolsillo
**"Piggy = Permission (engañado). Tail = sin permiso (a escondidas)"**

### Materiales en dumpster diving — "POEM-CSD"
- **P**hone lists · **O**rg charts · **E**mails/notes/faxes · **M**anuals · **C**alendars · **S**ource code · **D**iagrams de red

---

## 4. Flashcards

**Q:** ¿Qué mecanismo psicológico explota específicamente la variante "Legitimate End-User" de impersonation?
**A:** La reciprocación: la regla social por la que un favor genera obligación de devolver otro, incluso si el favor original no fue solicitado.

---

**Q:** Un atacante llama a números aleatorios de una empresa fingiendo ser del departamento de TI y, cuando encuentra a alguien con un problema real, le ofrece ayuda a cambio de credenciales. ¿Qué técnica es?
**A:** Quid Pro Quo.

---

**Q:** ¿Cuál es el nombre alternativo de Diversion Theft?
**A:** "Round the Corner Game" o "Cornet Game".

---

**Q:** ¿En qué se diferencia Piggybacking de Tailgating?
**A:** En Piggybacking la víctima sabe que deja pasar al atacante (aunque engañada). En Tailgating, la víctima no sabe que el atacante la sigue y entra sin su conocimiento.

---

**Q:** Un atacante deja un USB en el aparcamiento con la etiqueta "Employee Salary Information 2024" y el logo de la empresa. ¿Qué técnica es y qué principios psicológicos explota?
**A:** Baiting. Explota la curiosidad y la codicia del empleado.

---

**Q:** ¿Cuáles son las tres fases de Reverse Social Engineering en orden?
**A:** Sabotage → Marketing → Support.

---

**Q:** ¿Por qué la técnica Third-party Authorization es especialmente efectiva?
**A:** Porque es más efectiva cuando la figura de autoridad referenciada está de vacaciones o de viaje, haciendo imposible la verificación inmediata.

---

**Q:** Un atacante crea un perfil online atractivo y establece una relación falsa con un empleado de la empresa objetivo para extraer información confidencial. ¿Qué técnica es?
**A:** Honey Trap.

---

**Q:** ¿Qué diferencia Elicitation del resto de técnicas de extracción de información?
**A:** En Elicitation el atacante no pide información directamente; la extrae mediante conversaciones aparentemente inocentes y normales, sin que la víctima sea consciente de que está revelando datos sensibles.

---

**Q:** ¿A qué categoría de ingeniería social pertenece el Dumpster Diving?
**A:** Human-based.

---

**Q:** ¿Cuál es el objetivo principal de los ataques Bait and Switch?
**A:** Clientes de e-commerce.

---

**Q:** ¿Qué tecnología usa el Vishing para falsificar la identidad del llamante?
**A:** Caller ID spoofing sobre VoIP (Voice over IP).

---

**Q:** Un atacante fingiendo ser auditor externo se presenta en la empresa y dice tener 10 minutos para comprobar los procedimientos de recuperación ante desastres. ¿Qué subtipo de impersonation es?
**A:** Trusted Authority Figure (figura de autoridad de confianza).

---

**Q:** ¿Qué información es recuperable mediante Dumpster Diving? Cita al menos 6 tipos.
**A:** IDs y contraseñas, diagramas de red, números de cuenta, estados bancarios, datos de nóminas, código fuente, previsiones de ventas, listas de teléfonos, organigramas, calendarios, manuales de política.

---

**Q:** ¿En qué se diferencia Bait and Switch de Baiting?
**A:** Bait and Switch opera online (enlace/descarga) y tiene como objetivo clientes de e-commerce. Baiting usa dispositivos físicos (USB) y explota curiosidad/codicia de empleados.

---

**Q:** Un atacante se hace pasar por técnico de reparación, entra en la empresa y planta dispositivos de escucha mientras finge revisar el sistema eléctrico. ¿Qué técnica es?
**A:** Impersonation — variante Repairman.

---

**Q:** ¿Cuál es la diferencia entre Shoulder Surfing pasivo y con ayuda técnica?
**A:** El pasivo es la simple observación directa por encima del hombro. Con ayuda técnica se usan prismáticos, dispositivos ópticos o cámaras de pequeño tamaño instaladas para grabar las acciones en el sistema.

---

## 5. Confusión frecuente

### Baiting vs. Quid Pro Quo
- **Baiting**: el atacante deja un objeto/oferta de forma **pasiva y unilateral**. La víctima actúa sola (coge el USB, hace clic). No hay interacción directa.
- **Quid Pro Quo**: el atacante **contacta activamente** con la víctima y ofrece un servicio a cambio de credenciales. Hay intercambio bidireccional explícito.
- **Criterio**: ¿hay interacción directa atacante-víctima con oferta explícita de servicio? Sí → Quid Pro Quo. No → Baiting.

---

### Baiting vs. Bait and Switch
- **Baiting**: vector **físico** (USB, dispositivo), explota curiosidad/codicia, objetivo: empleados.
- **Bait and Switch**: vector **online** (enlace, descarga), explota deseo de oferta ventajosa, objetivo: clientes de e-commerce.
- **Criterio**: ¿el gancho es un objeto físico? → Baiting. ¿Es un enlace/oferta online? → Bait and Switch.

---

### Vishing vs. Phishing
- **Vishing**: canal voz/VoIP → **human-based**.
- **Phishing**: canal email/web → **computer-based**.
- **Criterio**: el canal determina la categoría. Teléfono/voz → Vishing/human. Email/web → Phishing/computer.

---

### Eavesdropping vs. Shoulder Surfing
- **Eavesdropping**: escucha/lectura de **comunicaciones** (audio, vídeo, mensajes). La información viaja por un canal.
- **Shoulder Surfing**: observación directa de **lo que la víctima introduce** en un dispositivo. La información está siendo tecleada, no transmitida.
- **Criterio**: ¿la información está en tránsito (comunicación)? → Eavesdropping. ¿Está siendo introducida en un dispositivo? → Shoulder Surfing.

---

### Reverse Social Engineering vs. Impersonation
- **Impersonation**: el atacante **se acerca a la víctima** y solicita información directamente.
- **Reverse Social Engineering**: el atacante **espera a que la víctima acuda a él**. Crea un problema para que le busquen.
- **Criterio**: ¿quién inicia el contacto en busca de ayuda? El atacante → Impersonation. La víctima → Reverse Social Engineering.
