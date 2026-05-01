# M09_03_ComputerBasedSocialEngineering.md
## CEH v13 — Módulo 09 | Computer-based Social Engineering Techniques

---

## 1. Conceptos y definiciones

### Marco general

Las técnicas computer-based usan sistemas informáticos e Internet como vector principal. No requieren interacción directa persona a persona. El objetivo sigue siendo el mismo: extraer credenciales, datos financieros o acceso a sistemas.

---

### 🔴 Phishing — Tipología completa

Phishing es el envío de comunicaciones falsas que suplantan a entidades legítimas para obtener información sensible. El atacante registra un dominio falso, construye un sitio lookalike y envía el enlace a las víctimas.

| Tipo | Objetivo | Característica diferencial |
|---|---|---|
| **Phishing (genérico)** | Masivo, sin objetivo específico | Miles de emails, baja personalización |
| **Spear Phishing** | Empleado concreto o grupo pequeño | Alta personalización, parece venir de alguien interno con autoridad |
| **Whaling** | CEO, CFO, políticos, celebrities | Email/web cuidadosamente diseñados; objetivo: datos corporativos críticos |
| **Pharming** | Usuarios de cualquier sitio web | Redirige tráfico sin necesidad de que la víctima haga clic en un enlace malicioso |
| **Spimming (SPIM)** | Usuarios de plataformas de mensajería instantánea | Spam vía IM usando bots; incluye malware como adjunto o hipervínculo |
| **Clone Phishing** | Destinatarios de emails legítimos previos | Réplica exacta de un email real con enlace/adjunto sustituido por malicioso |
| **E-wallet Phishing** | Usuarios de monederos digitales | Web falsa que imita al proveedor de e-wallet; captura credenciales en tiempo real |
| **Tabnabbing** | Usuarios con múltiples pestañas abiertas | La pestaña maliciosa cambia su contenido mientras el usuario está en otra pestaña |
| **Reverse Tabnabbing** | Usuarios que hacen clic en enlaces | El enlace cambia el contenido de la pestaña **original** del usuario a una página phishing |
| **Consent Phishing** | Usuarios de OAuth (Google, Facebook, Microsoft) | No roba credenciales; obtiene permisos OAuth mediante una app falsa |
| **Search Engine Phishing** | Usuarios que buscan en Google/Bing | Posiciona webs maliciosas en resultados de búsqueda (SEO poisoning) |
| **Angler Phishing** | Usuarios descontentos en redes sociales | Crea cuenta falsa de soporte de la empresa y responde a quejas públicas |

---

### Pharming — Dos vectores técnicos

**Pharming = "Phishing without a Lure"**: no requiere engañar al usuario para que haga clic. El tráfico se redirige a nivel DNS o de sistema operativo.

**Vector 1 — DNS Cache Poisoning:**
1. Atacante corrompe la caché del servidor DNS objetivo.
2. Modifica la IP de `www.targetwebsite.com` → IP de `www.hackerwebsite.com`.
3. Víctima escribe la URL legítima en el navegador.
4. El servidor DNS devuelve la IP falsa ya modificada.
5. La víctima llega al sitio del atacante sin saberlo.

**Vector 2 — Host File Modification:**
1. Atacante envía código malicioso como adjunto de email.
2. El usuario ejecuta el adjunto.
3. El código modifica el fichero `hosts` local del sistema.
4. Cuando la víctima escribe la URL legítima, el fichero hosts local redirige al sitio del atacante.

Pharming también puede ejecutarse mediante troyanos o gusanos.

---

### Tabnabbing vs. Reverse Tabnabbing

| | Tabnabbing | Reverse Tabnabbing |
|---|---|---|
| **Quién controla la pestaña maliciosa** | El atacante (web maliciosa que detecta cambio de pestaña) | El atacante modifica la pestaña **original** del usuario |
| **Flujo** | Usuario abre web maliciosa → cambia de pestaña → la web maliciosa se transforma en login legítimo | Usuario en web legítima → hace clic en enlace → la pestaña original cambia a phishing |
| **Por qué funciona** | El usuario vuelve a una pestaña que parece legítima | El usuario confía más en pestañas que él mismo abrió |

---

### Consent Phishing — OAuth

El atacante no roba contraseñas: explota el flujo OAuth. La víctima concede permisos a una app falsa, permitiendo al atacante acceder a:
- Direcciones de email, contactos, datos de perfil.
- Envío de spam desde la cuenta de la víctima.
- Acceso a otras cuentas vinculadas.

Vector de entrega posible: email de phishing, publicaciones en redes sociales, anuncios online.

---

### 🔴 Otras técnicas computer-based

| Técnica | Mecanismo | Objetivo |
|---|---|---|
| **Pop-up Windows** | Ventana emergente con aviso falso de error/infección → instalación de troyano/spyware | Credenciales, datos personales |
| **Hoax Letters** | Advertencia falsa sobre un virus inexistente → se propaga por reenvío | Pérdida de productividad, consumo de recursos de red |
| **Chain Letters** | Promesa de regalo/dinero si se reenvía a X destinatarios | Propagación masiva, recolección de emails |
| **Instant Chat Messenger** | Conversación informal para extraer fecha de nacimiento, nombre de soltera, etc. | Cracking de contraseñas de recuperación |
| **Spam Email** | Emails masivos con adjuntos maliciosos o petición de datos financieros | Credenciales bancarias, datos de red |
| **Scareware** | Pop-up que finge detectar malware; urgencia para descargar "antivirus" falso | Instalación de malware, dinero |

**Scareware vs. Pop-up**: Scareware es una categoría específica de malware que usa pop-ups con falsa alarma de infección. Todo scareware usa pop-ups, pero no todo pop-up es scareware.

---

### 🔴 Deepfake — Impersonation con IA

Un deepfake es la creación de contenido audiovisual falso de una persona real usando ML/IA, con el objetivo de engañar a otras personas haciéndoles creer que es auténtico.

**Técnicas subyacentes:**
- CNNs (Convolutional Neural Networks) y GANs (Generative Adversarial Networks).
- Proceso básico: vídeo fuente (cara a suplantar) + vídeo destino (vídeo objetivo) → sustitución facial.

**Usos maliciosos:** engaño/desinformación, chantaje reputacional, phishing avanzado (CEO fraud en vídeo).

**Herramienta principal del examen**: DeepFaceLab (deepfakevfx.com).
**Otras herramientas**: Deepfakesweb, Synthesia, DeepBrain AI, Vidnoz, Hoodem.

**Conocimientos requeridos** (dato examen): CNNs, GANs, edición de vídeo (Adobe Premiere Pro, Final Cut Pro, DaVinci Resolve), compositing, color grading, motion tracking, rotoscoping.

---

### Voice Cloning — Impersonation de voz con IA

El atacante replica la voz de una persona real para usarla en llamadas, mensajes de voz o grabaciones maliciosas.

**Fases:**
1. **Data collection**: grabaciones del objetivo (discursos, entrevistas, podcasts, redes sociales).
2. **Neural networks training**: CNNs o RNNs aprenden patrones de voz (tono, entonación, estilo).
3. **Create voice samples**: el modelo genera audio sintético a partir de texto (TTS con voz clonada).
4. **Impersonation**: llamadas, mensajes de voz, grabaciones fraudulentas.

**Herramienta principal del examen**: VEED.IO — clonación en tiempo real con una sola muestra de voz.
**Otras**: Murf.AI, Resemble.AI, ElevenLabs, PlayHT, voice.ai.

---

### AI para phishing — ChatGPT e impersonación de estilo

Los atacantes usan IA generativa para crear emails de phishing contextualmente relevantes, imitar el estilo de escritura de personas concretas (entrenando el modelo con textos públicos) y personalizar campañas masivas de spear phishing de forma automatizada.

**Ventaja de IA sobre humano**: consistencia, escalabilidad, adaptación rápida, sin fatiga. **Limitación**: carece de intuición y comprensión matizada.

**Herramienta principal del examen**: SET (Social Engineering Toolkit) — open-source, Python, pentesting via social engineering; categoriza ataques por vector (email, web, USB).
**ShellPhish**: phishing de credenciales en redes sociales (Instagram, Facebook, Twitter, LinkedIn) + recoge IP pública, hostname, geolocalización y datos del navegador de la víctima.

---

### 🔴 Angler Phishing vs. Catfishing

| | Angler Phishing | Catfishing |
|---|---|---|
| **Objetivo** | Usuarios descontentos con una empresa | Individuo específico (relación personal/laboral) |
| **Vector** | Cuenta falsa de soporte en redes sociales | Perfil falso de identidad atractiva |
| **Mecanismo** | Responde a quejas públicas o publica enlaces falsos de soporte | Construye relación romántica/personal falsa |
| **Fin** | Robo de credenciales o datos de cuenta | Extorsión económica, ingeniería social prolongada |

**Señales de Catfishing**: evita comunicación directa/webcam, una sola foto de perfil durante años, muchos amigos del género opuesto, peticiones de dinero urgentes con pretexto de emergencia.

---

### 🔴 Identity Theft — Tipos

| Tipo | Elemento robado / Uso |
|---|---|
| **Child Identity Theft** | SSN de menor → créditos, préstamos; puede pasar desapercibido durante años |
| **Criminal Identity Theft** | Usa identidad ajena al ser detenido para escapar de cargos |
| **Financial Identity Theft** | Cuenta bancaria o tarjeta → retiros, nuevas cuentas, préstamos |
| **Driver's License Identity Theft** | Carnet robado → tráfico, venta, multas a nombre de la víctima |
| **Insurance / Medical Identity Theft** | Datos médicos → tratamientos, seguros médicos en nombre de la víctima |
| **Tax Identity Theft** | SSN → declaración de renta fraudulenta, devolución a nombre ajeno |
| **Identity Cloning** | Suplantar identidad completa para ocultarse (inmigrantes ilegales, deudores) |
| **Synthetic Identity Theft** | Combina datos reales de distintas víctimas para crear una identidad nueva |
| **Social Security Identity Theft** | SSN → venderlo, obtener pasaporte, defraudar al Estado |

**Más peligroso**: Medical Identity Theft (diagnósticos erróneos, riesgo vital).
**Más sofisticado**: Synthetic Identity Theft (combina fragmentos de múltiples víctimas).
**Más fácil**: Driver's License Identity Theft (poca sofisticación).
**Pasa más tiempo desapercibido**: Child Identity Theft.

---

### Técnicas de obtención de identidad — Métodos clave

| Método | Descripción |
|---|---|
| **Skimming** | Dispositivo físico (skimmer/wedge) en lector de tarjeta para copiar banda magnética |
| **Wardriving** | Búsqueda de Wi-Fi abierta desde vehículo con portátil/smartphone |
| **Pretexting** | Impersonar ejecutivo de entidad financiera con "smooth-talking" |
| **Mail Theft & Rerouting** | Robo de correo postal con extractos bancarios, tarjetas, formularios |
| **Social Media Mining** | Extracción de datos personales de perfiles públicos |
| **Dark Web Data Trading** | Compra de SSN, tarjetas, credenciales en mercados del Dark Web |

---

### Riesgos corporativos de las redes sociales (11 amenazas)

Data Theft · Involuntary Data Leakage · Targeted Attacks · Network Vulnerability · Spam and Phishing · Modification of Content · Malware Propagation · Business Reputation damage · Infrastructure Costs · Loss of Productivity · Reconnaissance.

---

### Ley de referencia

**Identity Theft and Assumption Deterrence Act de 1998** (EE. UU.) — define el robo de identidad como el uso ilegal de la información de identificación de otra persona.

---

## 2. Exam Traps ⚠️

⚠️ **[Pharming — "Phishing without a Lure"]** El examen usa este alias directamente. Si ves "phishing sin señuelo" o "redirección sin clic" → Pharming.

⚠️ **[Pharming — dos vectores distintos]** DNS Cache Poisoning afecta al servidor DNS (todos los usuarios). Host File Modification afecta solo al equipo local infectado por adjunto. No confundirlos: diferente alcance y vector de entrega.

⚠️ **[Spear Phishing vs. Whaling]** Ambos son dirigidos. Whaling → exclusivamente altos ejecutivos (CEO, CFO, políticos, celebrities). Spear Phishing → cualquier empleado específico.

⚠️ **[Tabnabbing vs. Reverse Tabnabbing]** En Tabnabbing, la pestaña maliciosa (del atacante) se transforma. En Reverse Tabnabbing, la pestaña original del usuario (legítima) es la que cambia. El criterio: ¿qué pestaña se modifica?

⚠️ **[Consent Phishing — no roba contraseñas]** Obtiene permisos OAuth. El atacante accede a la cuenta sin conocer las credenciales de la víctima. Si el enunciado dice "sin obtener credenciales" → Consent Phishing.

⚠️ **[Scareware — malware, no solo técnica]** Es una categoría de malware, no solo una técnica de ingeniería social. El examen puede presentarlo en ambos contextos.

⚠️ **[Hoax Letters — sin daño técnico directo]** Solo producen pérdida de productividad y consumo de recursos de red. No borran datos, no infectan sistemas.

⚠️ **[Deepfake y Voice Cloning → computer-based]** Aunque el resultado sea una llamada de voz o un vídeo, el vector de generación es tecnológico (IA/ML). Clasificación: computer-based.

⚠️ **[SET — Python, open-source, pentesting]** No confundirlo con ShellPhish (específico de redes sociales). SET categoriza por vector (email, web, USB).

⚠️ **[Medical Identity Theft — más peligroso]** Sintético es el más sofisticado, pero Medical es el más peligroso por el riesgo vital. El examen distingue ambos calificativos.

⚠️ **[Child Identity Theft — no es el más peligroso]** Es el más duradero en su desdetección, no el más peligroso. Peligroso → Medical.

⚠️ **[Angler Phishing — target: usuarios descontentos]** El detalle diferencial es que el atacante se dirige específicamente a quienes han publicado quejas en redes sociales, no a usuarios al azar.

---

## 3. Nemotécnicos

### 12 tipos de Phishing — "SW-P-SPIM-CE-TR-ConSAC"
**"Spear Whales Pharming SPIM, Clone E-wallets, Tab Reverse, Consent Search, Angler Cats"**
Spear · Whaling · Pharming · SPIM · Clone · E-wallet · Tabnabbing · Reverse Tabnabbing · Consent · Search Engine · Angler · Catfishing

### Fases de Voice Cloning — "DCCS"
**D**ata collection → **C**NNs/RNNs training → **C**reate voice samples → **S**uplantación (impersonation)

### 9 tipos de Identity Theft — "CCFDI-TCSS"
**C**hild · **C**riminal · **F**inancial · **D**river's License · **I**nsurance · **T**ax · **C**loning · **S**ynthetic · **S**ocial Security

### Rangos de peligrosidad/sofisticación (datos clave del examen)
- **Más peligroso** → Medical
- **Más sofisticado** → Synthetic
- **Más fácil** → Driver's License
- **Más tiempo desapercibido** → Child

### Pharming — regla de los dos vectores
- **DNS** = servidor externo → todos los usuarios del DNS
- **HOST** = fichero local → solo el equipo infectado por adjunto

---

## 4. Flashcards

**Q:** ¿Por qué Pharming se denomina "Phishing without a Lure"?
**A:** Porque redirige el tráfico de la víctima sin necesidad de que haga clic en ningún enlace malicioso. La redirección ocurre a nivel de servidor DNS o de fichero hosts local.

---

**Q:** ¿Cuáles son los dos métodos técnicos de un ataque Pharming?
**A:** DNS Cache Poisoning (modifica la IP en el servidor DNS externo) y Host File Modification (modifica el fichero hosts local mediante un adjunto malicioso ejecutado por la víctima).

---

**Q:** ¿En qué se diferencia Spear Phishing de Whaling?
**A:** Spear Phishing se dirige a un empleado concreto o grupo pequeño. Whaling se dirige exclusivamente a altos ejecutivos (CEO, CFO, políticos, celebrities) con acceso a información crítica.

---

**Q:** ¿Qué protocolo explota el Consent Phishing y qué obtiene el atacante sin obtener contraseñas?
**A:** OAuth. El atacante obtiene permisos de acceso a la cuenta (emails, contactos, perfil, cuentas vinculadas) sin necesitar las credenciales de la víctima.

---

**Q:** ¿Qué diferencia Tabnabbing de Reverse Tabnabbing?
**A:** En Tabnabbing, la pestaña maliciosa (del atacante) se transforma mientras el usuario está en otra pestaña. En Reverse Tabnabbing, un enlace malicioso modifica la pestaña original del usuario (la legítima).

---

**Q:** ¿Qué es SPIM, qué plataformas usa y quién genera las campañas?
**A:** SPIM (Spam over Instant Messaging) es spam distribuido por mensajería instantánea. Lo generan spimmers usando bots que recopilan IDs de IM y envían mensajes masivos con adjuntos o hipervínculos maliciosos.

---

**Q:** ¿Qué tipo de Identity Theft es el más peligroso y por qué?
**A:** Medical Identity Theft. Genera entradas erróneas en el historial médico que pueden provocar diagnósticos falsos y decisiones médicas con riesgo para la vida.

---

**Q:** ¿Por qué Child Identity Theft es atractivo para los atacantes?
**A:** Porque puede pasar desapercibido durante muchos años, hasta que el menor intenta acceder a crédito o servicios siendo adulto.

---

**Q:** ¿Qué es Synthetic Identity Theft y qué lo hace el más sofisticado?
**A:** Combina datos reales de distintas víctimas (p. ej., SSN de una persona + nombre/fecha de nacimiento de otra) para crear una identidad nueva que no pertenece a ninguna persona real.

---

**Q:** ¿Qué herramienta del CEH se usa para phishing en redes sociales y qué datos adicionales recoge de la víctima?
**A:** ShellPhish. Recoge credenciales de redes sociales (Instagram, Facebook, Twitter, LinkedIn) además de IP pública, hostname, geolocalización e información del navegador.

---

**Q:** ¿Qué es SET, en qué lenguaje está escrito y cómo categoriza los ataques?
**A:** Social Engineering Toolkit. Open-source, Python. Categoriza los ataques por vector: email, web y USB.

---

**Q:** ¿Qué tecnologías de IA subyacen a la creación de deepfakes?
**A:** CNNs y GANs. El proceso combina un vídeo fuente (cara a suplantar) con un vídeo destino para sustituir la cara original. Requiere edición de vídeo (Adobe Premiere Pro, Final Cut Pro, DaVinci Resolve).

---

**Q:** ¿Cuáles son las fases de Voice Cloning en orden?
**A:** 1) Data collection (grabaciones del objetivo) → 2) Entrenamiento de CNNs/RNNs → 3) Generación de muestras de voz sintética → 4) Impersonation (llamadas/mensajes fraudulentos).

---

**Q:** ¿En qué se diferencia Angler Phishing de phishing genérico?
**A:** Angler Phishing se dirige específicamente a usuarios descontentos que han publicado quejas en redes sociales. El atacante crea una cuenta falsa de soporte y responde a esas quejas con enlaces maliciosos.

---

**Q:** ¿Qué es Skimming y cómo difiere de Pharming en el método de obtención de datos de tarjeta?
**A:** Skimming usa un dispositivo físico (skimmer/wedge) en un lector de tarjeta para copiar la banda magnética. Pharming redirige tráfico web de forma lógica (DNS o hosts). Skimming es físico; Pharming es lógico.

---

**Q:** ¿Qué ley estadounidense define el robo de identidad y en qué año se aprobó?
**A:** Identity Theft and Assumption Deterrence Act de 1998.

---

**Q:** ¿Qué es Clone Phishing y en qué se diferencia del phishing genérico?
**A:** El atacante crea una réplica casi idéntica de un email legítimo previamente enviado, sustituyendo el enlace o adjunto original por uno malicioso. La víctima reconoce el email y baja la guardia.

---

## 5. Confusión frecuente

### Spear Phishing vs. Whaling
- **Spear Phishing**: cualquier empleado o grupo pequeño.
- **Whaling**: exclusivamente altos ejecutivos con acceso completo a información crítica.
- **Criterio**: ¿el objetivo es un alto cargo con acceso total? Sí → Whaling. Empleado genérico → Spear Phishing.

---

### Pharming DNS vs. Pharming Host File
- **DNS Cache Poisoning**: afecta al servidor DNS externo; todos los usuarios de ese servidor son redirigidos.
- **Host File Modification**: afecta solo al equipo local infectado por adjunto malicioso.
- **Criterio**: ¿el vector es un adjunto de email ejecutado localmente? → Host File. ¿El ataque va al servidor DNS? → DNS Cache Poisoning.

---

### Tabnabbing vs. Reverse Tabnabbing
- **Tabnabbing**: la pestaña maliciosa (del atacante) se transforma.
- **Reverse Tabnabbing**: la pestaña original del usuario cambia mediante un enlace.
- **Criterio**: ¿qué pestaña cambia de contenido? La maliciosa → Tabnabbing. La legítima original del usuario → Reverse Tabnabbing.

---

### Scareware vs. Pop-up Windows
- **Pop-up Windows**: categoría genérica con cualquier pretexto (error de sistema, reconexión de red).
- **Scareware**: subcategoría específica que finge detectar un virus/malware para urgir la descarga de software malicioso.
- **Criterio**: ¿el pop-up finge detectar malware? → Scareware. Otro pretexto → Pop-up genérico.

---

### Hoax vs. Chain Letter
- **Hoax**: avisa de un virus inexistente → alarma y consumo de recursos.
- **Chain Letter**: promete beneficio o amenaza desgracia si no se reenvía → propagación y recolección de emails.
- **Criterio**: ¿hay promesa o amenaza personal? → Chain Letter. ¿Solo alerta de virus falso? → Hoax.

---

### Medical vs. Synthetic Identity Theft
- **Medical**: el más **peligroso** (riesgo de vida por errores médicos).
- **Synthetic**: el más **sofisticado** (identidad nueva creada de múltiples víctimas).
- **Criterio**: peligrosidad → Medical. Sofisticación → Synthetic.
