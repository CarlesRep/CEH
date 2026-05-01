# M10_02_Botnets.md
## CEH v13 — Módulo 10 | Botnets

---

## 1. Conceptos y definiciones

### Definiciones base

- **Bot**: contracción de "robot". Aplicación software que ejecuta tareas automatizadas en Internet.
- **Botnet**: contracción de "roBOT NETwork". Red de equipos infectados por bots. Un botnet de apenas **1.000 bots** tiene un ancho de banda combinado mayor que el de la mayoría de sistemas corporativos.
- Los bots tienen usos benignos (web spidering, data mining) y maliciosos.
- Tipos de bots: **Internet bots**, **IRC bots** (Cardinal, Sopel, Eggdrop, EnergyMech), **chatter bots**.

---

### 🔴 Jerarquía del crimen organizado cibernético

El crimen organizado cibernético opera con una estructura jerárquica y modelo de reparto de ingresos predefinido:

| Rol | Función |
|---|---|
| **Boss** | Jefe de la organización; actúa como empresario; **no comete crímenes directamente** |
| **Underboss** | Configura el servidor C&C y la base de datos crimeware toolkit; gestiona Trojans |
| **Campaign Managers** | Gestionan redes de afiliados para implementar ataques y robar datos |
| **Resellers** | Venden los datos robados |

**Ejemplo de operación coordinada**: ataque DDoS contra un banco para distraer al equipo de seguridad mientras se vacían cuentas con credenciales robadas.

---

### 🔴 Flujo de configuración de un botnet típico (Figure 10.5 — 12 pasos)

1. El atacante configura el C&C center y la base de datos crimeware toolkit.
2. Recluta afiliados.
3. Los afiliados aportan malware y lanzan el DDoS toolkit.
4. El atacante compromete webs legítimas o crea webs maliciosas nuevas.
5. Redirige víctimas a la web maliciosa mediante phishing/ingeniería social.
6. Los usuarios visitan la web maliciosa/comprometida.
7. La web maliciosa redirige al crimeware toolkit database.
8. El malware infecta los sistemas de los usuarios.
9. Los bots se conectan de vuelta al C&C center.
10. Los sistemas infectados buscan otros sistemas vulnerables y los infectan (expansión del botnet).
11. Los bots reciben instrucciones del C&C para atacar al primary target.
12. Atacan al objetivo principal.

**Flujo simplificado del bot (Image 2 — 6 pasos)**:
1. Atacante configura bot C&C handler.
2. Infecta una máquina (victim/bot).
3. El bot busca otros sistemas vulnerables y los infecta → crea botnet.
4. Los bots se conectan al C&C y esperan instrucciones.
5. El atacante envía comandos a los bots a través del C&C.
6. Los bots atacan el servidor objetivo.

---

### 🔴 Usos maliciosos de botnets

| Uso | Mecanismo |
|---|---|
| **DDoS attacks** | Generan tráfico masivo que consume ancho de banda y destruye conectividad |
| **Spamming** | Usan SOCKS proxy; harvestan emails de webs y otras fuentes |
| **Sniffing traffic** | Packet sniffer en la máquina comprometida captura tarjetas de crédito, contraseñas; permite robar información de otro botnet |
| **Keylogging** | Registra pulsaciones de teclado; objetivo típico: credenciales de PayPal y similares |
| **Spreading new malware** | Propagan nuevos bots |
| **Click fraud / AdSense abuse** | Automatizan clics en anuncios (Google AdSense) para generar ingresos fraudulentos |
| **IRC clone attacks** | Cada bot se clona miles de veces dentro de una red IRC, similar a DDoS |
| **Manipulating polls/games** | Cada bot tiene dirección única → manipulación de votaciones y juegos online |
| **Mass identity theft** | Envío masivo de emails suplantando entidades (p. ej., eBay) para robo de identidad |
| **Credential stuffing** | Intentos automatizados de login con credenciales robadas en múltiples webs |
| **Cryptocurrency mining** | Instalan software de minería en los bots usando su CPU sin conocimiento del propietario |

---

### 🔴 Ecosistema del botnet (Figure Botnet Ecosystem)

El botnet se articula en torno a dos componentes centrales: **Crimeware Toolkit Database** y **Trojan Command and Control Center**.

**Fuentes de entrada** (cómo se construye/alimenta el botnet):
- Zero-Day Market
- Botnet Market
- Scan & Intrusion
- Malware Market
- C&C del owner

**Salidas / actividades maliciosas**:
- DDoS · Extortion
- Data Theft
- Spam (Emails, Mass Mailing, Redirect)
- Phishing
- Client-Side Vulnerability → Malicious Site
- Financial Diversion (Credit Card, E-Commerce)
- Licenses (MP3, DivX)
- Stock Fraud · Scams · Adverts

---

### 🔴 Métodos de escaneo para encontrar máquinas vulnerables

| Método | Mecanismo | Característica clave |
|---|---|---|
| **Random Scanning** | Sondea IPs aleatorias del rango objetivo; al encontrar una vulnerable la infecta con el mismo código | Genera mucho tráfico; propagación rápida al inicio, se ralentiza al agotarse IPs nuevas |
| **Hit-list Scanning** | El atacante recopila previamente una lista de vulnerables; al infectar una, divide la lista por la mitad: el atacante sigue con una mitad y la nueva máquina con la otra | Número de comprometidos crece **exponencialmente**; todo el hit-list infectado en poco tiempo |
| **Topological Scanning** | Usa información de la máquina infectada (URLs en el disco duro) para encontrar nuevos objetivos | Resultados precisos; rendimiento similar al hit-list scanning |
| **Local Subnet Scanning** | Busca vulnerables en la **red local** de la máquina infectada, detrás del firewall | Se combina con otros mecanismos de escaneo |
| **Permutation Scanning** | Comparte una lista de permutación pseudoaleatoria de IPs generada con un **block cipher de 32 bits** y una clave preseleccionada; evita reinfecciones; reinicia desde punto aleatorio si encuentra máquina ya infectada | Evita reinfección; alta velocidad de escaneo; genera nueva clave de permutación cuando se agota la fase actual |

---

### 🔴 Técnicas de propagación del código malicioso

| Técnica | Origen del toolkit | Protocolo | Mecanismo |
|---|---|---|---|
| **Central Source Propagation** | Servidor central del atacante | HTTP, FTP, RPC | Al infectar nueva máquina, la fuente central transfiere una copia del toolkit; la nueva máquina busca más víctimas automáticamente |
| **Back-chaining Propagation** | Sistema del propio atacante | **TFTP** (Trivial File Transfer Protocol) | El sistema comprometido se conecta de vuelta al atacante (back-channel); el atacante transfiere el toolkit mediante port listeners o servidores web instalados con TFTP |
| **Autonomous Propagation** | El host atacante directamente | N/A (transferencia directa) | El host atacante transfiere el toolkit **en el momento exacto** en que compromete el nuevo sistema; no hay fuente externa |

**Distinción clave**:
- Central Source: fuente centralizada separada → toolkit copiado desde fuera.
- Back-chaining: el comprometido llama al atacante para recibir el toolkit (back-channel).
- Autonomous: el atacante transfiere el toolkit **él mismo** en tiempo real al comprometer la víctima.

---

## 2. Exam Traps ⚠️

⚠️ **[Boss — no comete crímenes directamente]** El examen puede preguntar qué rol de la jerarquía cybercrime comete los crímenes. El boss actúa como empresario y **no comete crímenes directamente**.

⚠️ **[Underboss — configura C&C y crimeware toolkit]** El que configura el servidor C&C no es el boss sino el **underboss**.

⚠️ **[Hit-list Scanning — crecimiento exponencial]** El examen puede preguntar qué técnica de escaneo causa un crecimiento exponencial de máquinas comprometidas. Respuesta: Hit-list Scanning (división sucesiva de la lista).

⚠️ **[Permutation Scanning — block cipher de 32 bits]** Dato técnico concreto: la lista pseudoaleatoria se genera con un block cipher de **32 bits** y una clave preseleccionada. El examen puede preguntar el tamaño del cipher.

⚠️ **[Back-chaining — protocolo TFTP]** El protocolo específico de Back-chaining Propagation es **TFTP** (Trivial File Transfer Protocol). Central Source usa HTTP/FTP/RPC. No son intercambiables.

⚠️ **[Autonomous Propagation — sin fuente externa]** La característica que lo distingue: el toolkit se transfiere **en el momento exacto** del compromiso, directamente desde el host atacante. No hay servidor central ni back-channel.

⚠️ **[Local Subnet Scanning — detrás del firewall]** Es la única técnica de escaneo que opera específicamente detrás de un firewall, en la red local de la máquina infectada.

⚠️ **[Random Scanning — tráfico excesivo]** Random Scanning genera mucho tráfico porque múltiples máquinas comprometidas sondean las mismas IPs simultáneamente. Es su debilidad principal.

⚠️ **[Botnet de 1.000 bots — ancho de banda]** El CEH especifica que un botnet de apenas 1.000 bots tiene más ancho de banda combinado que la mayoría de sistemas corporativos. Dato numérico preguntable.

⚠️ **[Sniffing en botnets — robo entre botnets]** El examen puede incluir la afirmación de que los sniffers de un botnet pueden robar información de **otro botnet**. Es correcto según el CEH.

⚠️ **[IRC Clone Attacks — también llamados clone attacks]** El examen puede usar el alias "clone attacks" para referirse a los ataques DDoS sobre redes IRC mediante botnets.

---

## 3. Nemotécnicos

### Jerarquía cybercrime — "B-U-C-R"
**"Boss Under-boss Campaign-managers Resellers"**
- **B**oss (no comete crímenes) → **U**nderboss (C&C + crimeware) → **C**ampaign managers (afiliados + ataques) → **R**esellers (venden datos)

### 5 métodos de escaneo — "R-H-T-L-P"
**"Robots Hitean Topologías Locales con Permutación"**
- **R**andom · **H**it-list · **T**opological · **L**ocal Subnet · **P**ermutation

### 3 técnicas de propagación — "C-B-A" + protocolos
**"Central usa HTTP/FTP/RPC · Back usa TFTP · Autonomous es directa"**
- **C**entral Source → HTTP/FTP/RPC
- **B**ack-chaining → TFTP
- **A**utonomous → transferencia directa en el momento del compromiso

### Usos maliciosos de botnets — "DDoS-SK-KSC-CMICC"
**"DDoS Spam Keylog Sniff Click Manipulate Identity Credential Crypto"**
- DDoS · Spam · Keylogging · Sniffing · Click fraud · Manipular polls · Identity theft · Credential stuffing · Cryptocurrency mining

---

## 4. Flashcards

**Q:** ¿Cuál es el ancho de banda combinado de un botnet de 1.000 bots según el CEH?
**A:** Mayor que el ancho de banda de la mayoría de sistemas corporativos.

**Q:** ¿Qué rol de la jerarquía del crimen organizado cibernético configura el servidor C&C y el crimeware toolkit database?
**A:** El underboss.

**Q:** ¿Qué rol de la jerarquía cybercrime no comete crímenes directamente?
**A:** El boss, que actúa como empresario.

**Q:** ¿Qué técnica de escaneo de botnets causa un crecimiento exponencial del número de máquinas comprometidas?
**A:** Hit-list Scanning — al infectar una máquina, la lista de objetivos se divide por la mitad y ambas mitades se escanean en paralelo.

**Q:** ¿Qué protocolo usa específicamente la técnica de Back-chaining Propagation?
**A:** TFTP (Trivial File Transfer Protocol).

**Q:** ¿Qué protocolos usa Central Source Propagation?
**A:** HTTP, FTP y RPC.

**Q:** ¿Cuál es la característica definitoria de Autonomous Propagation frente a las otras técnicas de propagación?
**A:** El host atacante transfiere el toolkit directamente al nuevo sistema comprometido en el momento exacto del compromiso, sin fuente centralizada ni back-channel.

**Q:** ¿Qué técnica de escaneo opera específicamente detrás de un firewall, en la red local de la máquina infectada?
**A:** Local Subnet Scanning.

**Q:** ¿Con qué se genera la lista pseudoaleatoria de IPs en Permutation Scanning?
**A:** Con un block cipher de 32 bits y una clave preseleccionada.

**Q:** ¿Qué ventajas ofrece Permutation Scanning frente a Random Scanning?
**A:** Evita la reinfección de objetivos ya comprometidos y garantiza alta velocidad de escaneo al seleccionar nuevos objetivos aleatoriamente.

**Q:** ¿Qué uso de botnet permite robar información de otro botnet?
**A:** Sniffing traffic — el sniffer en la máquina comprometida captura datos que pueden pertenecer a operaciones de otro botnet.

**Q:** ¿Cuáles son los 6 pasos del flujo de ataque botnet simplificado (Image 2)?
**A:** 1) Atacante configura C&C handler. 2) Infecta una máquina (bot). 3) El bot infecta otros sistemas vulnerables → botnet. 4) Bots se conectan al C&C y esperan instrucciones. 5) Atacante envía comandos a través del C&C. 6) Bots atacan el servidor objetivo.

**Q:** ¿Qué son los IRC Clone Attacks?
**A:** Ataques similares a DDoS en redes IRC donde el master agent instruye a cada bot para vincularse a miles de clones dentro de la red IRC, saturándola.

**Q:** ¿Qué es credential stuffing en el contexto de botnets?
**A:** Uso automatizado de credenciales robadas para intentar login masivo en múltiples webs, aprovechando los miles de IPs únicas del botnet.

---

## 5. Confusión frecuente

### Hit-list Scanning vs. Random Scanning
- **Random**: IPs elegidas aleatoriamente del rango objetivo; alto tráfico por solapamiento; propagación rápida al inicio y lenta al final.
- **Hit-list**: lista precompilada de vulnerables; división recursiva → crecimiento exponencial; todo el hit-list infectado en tiempo mínimo.
- **Criterio**: ¿hay una lista preconstruida de objetivos? → Hit-list. ¿Las IPs son completamente aleatorias? → Random.

### Topological Scanning vs. Local Subnet Scanning
- **Topological**: usa URLs encontradas en el disco duro de la máquina infectada para identificar nuevos objetivos; no limitado a la red local.
- **Local Subnet**: busca exclusivamente en la red local (LAN) de la máquina infectada; pensado para operar detrás de firewalls.
- **Criterio**: ¿el origen de la información son ficheros/URLs de la máquina infectada? → Topological. ¿El alcance es la red local/LAN? → Local Subnet.

### Central Source vs. Back-chaining vs. Autonomous Propagation
- **Central Source**: toolkit en servidor central separado → se copia al comprometido → HTTP/FTP/RPC.
- **Back-chaining**: el comprometido llama de vuelta al atacante para recibir el toolkit → TFTP.
- **Autonomous**: el atacante transfiere el toolkit él mismo en tiempo real en el momento del compromiso → sin protocolo de transferencia externo.
- **Criterio**: ¿hay un servidor central separado? → Central Source. ¿El comprometido llama al atacante (back-channel)? → Back-chaining. ¿El atacante lo transfiere directamente al comprometer? → Autonomous.

### Spamming vs. Mass Identity Theft (usos de botnet)
- **Spamming**: envío masivo de emails no solicitados; usa SOCKS proxy; objetivo: distribución de spam/malware.
- **Mass Identity Theft**: emails masivos suplantando entidades reputadas (eBay); objetivo específico: robo de identidad.
- **Criterio**: ¿el email suplanta una entidad específica para robar identidad? → Mass Identity Theft. ¿Es spam genérico o malware? → Spamming.
