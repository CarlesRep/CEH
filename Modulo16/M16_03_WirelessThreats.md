# M16_03_WirelessThreats.md
> Módulo 16 / Subapartado 3 — Wireless Threats

---

## 1. Conceptos y definiciones

### 🔴 Las 5 categorías de ataques wireless

| Categoría | Objetivo |
|---|---|
| **Access Control** | Evadir medidas de control de acceso WLAN (MAC filters, Wi-Fi port access controls) |
| **Integrity** | Alterar datos durante la transmisión; enviar frames forjados |
| **Confidentiality** | Interceptar información confidencial (cleartext o cifrada) |
| **Availability** | Obstruir la entrega de servicios wireless a usuarios legítimos |
| **Authentication** | Robar identidad, credenciales, login de clientes Wi-Fi |

---

### 🔴 Access Control Attacks

| Ataque | Descripción | Herramientas |
|---|---|---|
| **MAC Spoofing** | Reconfigura la MAC address para aparecer como AP autorizado al host de una red de confianza | SMAC |
| **AP Misconfiguration** | Configuración incorrecta de APs que expone la red; difícil de detectar porque el AP es un dispositivo legítimo; no dispara alertas en IDS | — |
| **Ad Hoc Associations** | Fuerza a la red a usar modo ad-hoc (sin AP); inherentemente inseguro, sin autenticación ni cifrado robustos; un atacante en la red puede comprometer también la LAN cableada | — |
| **Promiscuous Client** | Coloca AP cercano con SSID común y señal más fuerte → atrae a clientes hacia AP del atacante; similar al evil-twin attack | — |
| **Client Mis-association** | El cliente se conecta (intencionalmente o accidentalmente) a un AP fuera de la red legítima; el atacante configura un rogue AP fuera del perímetro corporativo; lanza MITM, EAP dictionary o Metasploit attacks | — |
| **Unauthorized Association** | Asociación maliciosa con **soft APs** (laptop con NIC configurada como AP falso) para acceder a la red; puede llevarse a cabo mediante virus que activan soft APs | — |

**Elementos clave en AP Misconfiguration:**
- SSID broadcast: APs con SSID por defecto → vulnerables a brute-force dictionary attacks
- WEP con SSID no cifrado → broadcast de contraseña en plaintext
- Weak password: usar SSID como contraseña básica
- Configuration error: instalación incorrecta, políticas inconsistentes, errores humanos

---

### 🔴 Integrity Attacks

| Ataque | Descripción | Herramientas |
|---|---|---|
| **Data-Frame Injection** | Construye y envía frames 802.11 forjados | Airpwn-ng, Wperf |
| **WEP Injection** | Construye y envía claves de cifrado WEP forjadas | WEP cracking + injection tools |
| **Bit-Flipping Attacks** | Captura frame, invierte bits aleatorios en el payload, modifica el ICV y lo reenvía al usuario | — |
| **Extensible AP Replay** | Captura protocolos EAP 802.1X (EAP Identity, Success, Failure) para posterior replay | — |
| **Data Replay** | Captura frames de datos 802.11 para posterior replay (modificado) | — |
| **Initialization Vector Replay** | Deriva el keystream enviando un mensaje en plaintext | — |
| **RADIUS Replay** | Captura mensajes RADIUS Access-Accept o Reject para posterior replay | — |
| **Wireless Network Viruses** | Los virus pueden comprometer APs; método simple para comprometer la red | — |

---

### 🔴 Confidentiality Attacks

| Ataque | Descripción | Herramientas |
|---|---|---|
| **Eavesdropping** | Captura y decodifica tráfico de aplicaciones no protegido | Wireshark, Ettercap, Kismet, analizadores comerciales |
| **Traffic Analysis** | Infiere información observando características externas del tráfico | Wireshark, Ettercap, Snort |
| **Cracking WEP Key** | Captura datos para recuperar clave WEP con brute force o **FMS (Fluhrer-Mantin-Shamir) cryptanalysis** | Aircrack-ng, WEPCrack |
| **Evil Twin AP** | Finge ser AP autorizado haciendo beacon del SSID de la WLAN → atrae usuarios | Hostapd, EvilTwinFramework, Wifiphisher |
| **Honeypot AP** | Configura SSID del AP igual al de un AP legítimo; señal más fuerte que APs legítimos → NIC conecta al rogue AP | Manipulating SSID |
| **Session Hijacking** | Manipula la red para que el host del atacante aparezca como el destino deseado | Ethernet capture + injection tools entre AP y auth server; wireless capture + injection tools entre cliente y AP |
| **Masquerading** | Finge ser usuario autorizado para acceder al sistema | — |
| **MITM Attack** | Ejecuta herramientas MITM convencionales en un evil-twin AP para interceptar sesiones TCP o túneles SSL/SSH | Capture + injection tools |

---

### 🔴 Availability Attacks

| Ataque | Descripción | Herramientas |
|---|---|---|
| **Access Point Theft** | Extrae físicamente el AP de su ubicación instalada | Stealth and/or speed |
| **Disassociation Attacks** | Destruye la conectividad entre AP y cliente | Destruction of connectivity |
| **EAP-Failure** | Observa intercambio EAP 802.1X válido y luego envía al cliente un mensaje EAP-Failure forjado | Airtool Pi |
| **Beacon Flood** | Genera miles de beacons 802.11 falsos → dificulta a clientes encontrar AP legítimo | AirJack |
| **Denial-of-Service** | Explota el mecanismo **CSMA/CA CCA (Clear Channel Assessment)** para hacer que el canal parezca ocupado | Adaptador que soporta CW Tx mode con utilidad de bajo nivel |
| **De-authenticate Flood** | Inunda clientes con de-authenticates o disassociates forjados → desconecta usuarios del AP | AirJack |
| **Routing Attacks** | Distribuye información de routing maliciosa usando protocolos como RIP, AODV, DSR mediante ataques wormhole y sinkhole | dsniff, Ettercap, aLTEr attack |
| **Authenticate Flood** | Envía authenticates/associates forjados desde MACs aleatorias → llena la tabla de asociación del AP objetivo | — |
| **ARP Cache Poisoning** | Crea múltiples vectores de ataque | — |
| **Power Saving Attacks** | Transmite TIM (Traffic Indication Map) o DTIM falso a cliente en modo power-saving → vulnerable a DoS | — |
| **TKIP MIC Exploit** | Genera datos TKIP inválidos para superar el umbral de errores MIC del AP objetivo → suspende el servicio WLAN | — |

---

### 🔴 Authentication Attacks

| Ataque | Descripción | Herramientas |
|---|---|---|
| **PSK Cracking** | Recupera WPA PSK de frames de handshake capturados usando dictionary attack | Cowpatty, Fern Wifi Cracker |
| **LEAP Cracking** | Recupera credenciales de paquetes LEAP 802.1X capturados crackeando NT password hash | Asleap, THC-LEAPcracker |
| **VPN Login Cracking** | Obtiene credenciales VPN (PPTP password o IPsec pre-shared key) con brute-force | ike_scan + IKECrack (IPsec); Anger + THC-pptp-bruter (PPTP) |
| **Domain Login Cracking** | Recupera credenciales Windows crackeando NetBIOS password hashes | John the Ripper, L0phtCrack, THC-Hydra |
| **Key Reinstallation Attack (KRACK)** | Explota el four-way handshake del protocolo WPA2 | Técnica de reutilización de nonce |
| **Identity Theft** | Captura identidades de usuario desde paquetes cleartext de 802.1X Identity Response | Packet capturing tools |
| **Shared Key Guessing** | Intenta autenticación 802.11 shared key con claves WEP por defecto del vendor o crackeadas | WEP cracking tools, Wifite |
| **Password Speculation** | Intenta repetidamente autenticación 802.1X usando identidad capturada para adivinar contraseña | Password dictionary |
| **Application Login Theft** | Captura credenciales de aplicaciones desde protocolos en cleartext | Ace Password Sniffer, dsniff, Wi-Jacking Attack |

---

### 🔴 Honeypot AP Attack

Un atacante configura un AP no autorizado con:
- **Alta ganancia** (antenas de alta potencia)
- **Mismo SSID** que la red objetivo
- **Señal más fuerte** que los APs legítimos → las NICs que buscan la señal más fuerte se conectan al rogue AP

Cuando un usuario autorizado se conecta al honeypot AP, el atacante puede robar: identidad, username y password.

---

### 🔴 Wormhole Attack

**Objetivo:** explotar protocolos de routing dinámico (**DSR** y **AODV**) en redes de sensores inalámbricos.

**Mecanismo:**
1. El atacante se posiciona estratégicamente para interceptar transmisiones
2. Anuncia que el nodo malicioso tiene la **ruta más corta** al destino
3. Crea un **túnel** entre nodo fuente (S) y destino (D) via nodo malicioso (M)
4. Snifea RREQ de S → lo reenvía a D antes que el original
5. Snifea RREP de D → lo reenvía a S antes que el original
6. Establece un **enlace directo falso** entre S y D via M
7. Controla el flujo de datos y lanza otros ataques

**Protocolos involucrados:** RREQ (Route Request) y RREP (Route Reply) de AODV/DSR.

**Impacto:** afecta confidencialidad, integridad y disponibilidad de los datos de la red.

---

### Sinkhole Attack

Variante del **selective forwarding attack**. El atacante:
1. Coloca nodo malicioso cerca de la **base station**
2. Anuncia al nodo malicioso como la **ruta más corta** a la base station
3. Atrae a todos los nodos vecinos con información de routing falsa
4. Realiza **data forging** sobre todas las transmisiones
5. Puede combinarse con wormhole attack

**Característica:** complejo de detectar; afecta aplicaciones de capas superiores del modelo OSI.

---

### Inter-Chip Privilege Escalation / Wireless Co-Existence Attack

**Mecanismo:** explota vulnerabilidades en chips combinados (combo chips) que manejan Bluetooth y Wi-Fi simultáneamente. Un chip Bluetooth puede:
- Capturar credenciales u otros datos sensibles del chip Wi-Fi
- Manipular el tráfico que pasa por el chip Wi-Fi

**Resultado:** **escalada de privilegios en los límites del chip (chip boundaries)** → también llamado **wireless co-existence attack**.

---

## 2. Exam Traps ⚠️

⚠️ **[Evil Twin AP vs. Honeypot AP — distinción sutil]**
Evil Twin AP: finge ser AP autorizado haciendo beacon del SSID de la WLAN para atraer usuarios (categoría: Confidentiality). Honeypot AP: configura el SSID igual al de un AP legítimo Y usa señal más fuerte; el concepto específico de honeypot implica señal superior para atraer pasivamente. Son similares pero con énfasis diferente; el examen los puede presentar como equivalentes o distintos.

⚠️ **[De-authenticate Flood vs. Disassociation Attack]**
De-authenticate Flood: inunda con frames de **de-autenticación o disasociación** forjados → desconecta usuarios. Disassociation Attack: destruye la conectividad entre AP y cliente. Son similares pero el De-authenticate Flood usa flooding masivo de frames falsos, mientras que Disassociation puede ser más quirúrgico.

⚠️ **[Beacon Flood vs. Authenticate Flood — ambas floods pero diferentes objetivos]**
Beacon Flood: miles de beacons falsos → dificulta encontrar AP legítimo (ataca la discovery). Authenticate Flood: autenticaciones forjadas desde MACs aleatorias → llena la tabla de asociación del AP (ataca la capacidad del AP).

⚠️ **[Promiscuous Client vs. Client Mis-association — diferencia de mecanismo]**
Promiscuous Client: el atacante crea AP con señal más fuerte → los clientes se conectan activamente buscando la mejor señal. Client Mis-association: el cliente se conecta accidentalmente o intencionalmente a un AP fuera del perímetro; el atacante configura un rogue AP y atrae con SSID spoofado.

⚠️ **[Wormhole vs. Sinkhole — routing attacks en redes de sensores]**
Wormhole: crea túnel artificial entre dos nodos → falsifica la ruta más corta mediante RREQ/RREP manipulation. Sinkhole: nodo malicioso cerca de la base station → se anuncia como ruta más corta a la base station. El sinkhole puede combinarse con wormhole.

⚠️ **[PSK Cracking: herramientas Cowpatty y Fern Wifi Cracker]**
Para PSK Cracking, las herramientas son Cowpatty y Fern Wifi Cracker (dictionary attack sobre frames de handshake capturados). El examen puede usar Aircrack-ng para PSK Cracking — Aircrack-ng se menciona para WEP Cracking (no PSK específicamente en estas tablas).

⚠️ **[LEAP Cracking: herramientas Asleap y THC-LEAPcracker]**
LEAP es un protocolo **propietario de Cisco**. Las herramientas para crackearlo son Asleap y THC-LEAPcracker (dictionary attack sobre NT password hash). El examen puede confundir LEAP con WPA PSK.

⚠️ **[CSMA/CA CCA para DoS]**
El DoS wireless que explota **CSMA/CA CCA (Clear Channel Assessment)** hace que el canal **parezca ocupado** → los dispositivos no transmiten. No es un flood de paquetes; es hacer que el canal parezca permanentemente ocupado. El examen puede confundirlo con Beacon Flood.

⚠️ **[TKIP MIC Exploit: supera umbral de errores MIC → suspende WLAN]**
El TKIP MIC Exploit genera datos TKIP inválidos para superar el **umbral de errores MIC** del AP → el AP suspende el servicio WLAN como medida de protección. El examen puede preguntar qué ocurre cuando se supera el umbral MIC → suspensión del servicio.

⚠️ **[Unauthorized Association: Soft AP vs. Accidental Association]**
Unauthorized Association tiene dos formas: (1) **maliciosa** via soft APs (laptop configurada como AP falso), (2) **accidental** (conectarse al AP de organización vecina sin conocimiento). El examen puede presentar solo una de las dos formas.

---

## 3. Nemotécnicos

**5 categorías de ataques** → **"ACICA"**: **A**ccess Control · **I**ntegrity · **C**onfidentiality · **A**vailability · **A**uthentication

**Access Control attacks (6)** → **"MAC-AP-Ad-Pro-Mis-Unauth"**:
- **MAC** Spoofing · **AP** Misconfiguration · **Ad** Hoc · **Pro**miscuous Client · **Mis**-association · **Unauth**orized Association

**Confidentiality attacks tools:**
- Eavesdropping/Traffic Analysis: **Wireshark + Ettercap + Kismet**
- WEP Cracking: **Aircrack-ng + WEPCrack**
- Evil Twin: **Hostapd + EvilTwinFramework + Wifiphisher**

**Authentication attacks tools:**
- PSK → **Cowpatty + Fern Wifi Cracker**
- LEAP → **Asleap + THC-LEAPcracker**
- Domain → **John the Ripper + L0phtCrack + THC-Hydra**
- VPN IPsec → **ike_scan + IKECrack**
- VPN PPTP → **Anger + THC-pptp-bruter**

**Wormhole vs. Sinkhole:**
- Wormhole: **túnel S↔D vía M** usando RREQ/RREP manipulation
- Sinkhole: **nodo malicioso near base station** → ruta más corta falsa

**Availability attacks** → **"AP-Dis-EAP-Beacon-DoS(CCA)-DeAuth-Route-Auth-ARP-Power-TKIP"**

---

## 4. Flashcards

**Q:** ¿Cuáles son las 5 categorías de ataques wireless?
**A:** Access Control, Integrity, Confidentiality, Availability, Authentication.

---

**Q:** ¿Cuál es la diferencia entre un Evil Twin AP y un Honeypot AP?
**A:** Evil Twin AP: finge ser AP autorizado haciendo beacon del SSID de la WLAN. Honeypot AP: usa el mismo SSID que un AP legítimo y transmite señal más fuerte para atraer dispositivos que buscan la señal más potente.

---

**Q:** ¿Cuáles son las herramientas para PSK Cracking en autenticación wireless?
**A:** Cowpatty y Fern Wifi Cracker — realizan dictionary attack sobre frames de key handshake capturados.

---

**Q:** ¿Qué herramientas se usan para LEAP Cracking y qué crackean?
**A:** Asleap y THC-LEAPcracker — realizan dictionary attack sobre el NT password hash extraído de paquetes LEAP capturados.

---

**Q:** ¿Qué hace el Beacon Flood attack?
**A:** Genera miles de beacons 802.11 falsos para dificultar a los clientes encontrar un AP legítimo. Herramienta: AirJack.

---

**Q:** ¿Qué explota el DoS wireless basado en CSMA/CA CCA?
**A:** Explota el mecanismo CSMA/CA Clear Channel Assessment para hacer que el canal parezca permanentemente ocupado, impidiendo que los dispositivos transmitan.

---

**Q:** ¿Qué hace un Authenticate Flood attack?
**A:** Envía authenticates/associates forjados desde MACs aleatorias para llenar la tabla de asociación del AP objetivo.

---

**Q:** ¿Cuál es el mecanismo del Wormhole Attack?
**A:** El atacante crea un túnel entre nodo fuente (S) y destino (D) vía nodo malicioso (M), interceptando y reenviando RREQ y RREP antes que los originales, estableciendo un enlace falso y tomando control del flujo de datos.

---

**Q:** ¿En qué se diferencia un Sinkhole Attack de un Wormhole Attack?
**A:** Sinkhole: nodo malicioso cerca de la base station se anuncia como ruta más corta → atrae todo el tráfico hacia sí para manipularlo. Wormhole: crea un túnel artificial entre dos nodos específicos manipulando RREQ/RREP. El sinkhole puede combinarse con wormhole.

---

**Q:** ¿Qué es un Wireless Co-Existence Attack / Inter-Chip Privilege Escalation?
**A:** Explota vulnerabilidades en combo chips (Bluetooth+Wi-Fi) para que un chip (ej. Bluetooth) capture datos sensibles o manipule el tráfico del otro chip (Wi-Fi), resultando en escalada de privilegios a nivel de chip (chip boundaries).

---

**Q:** ¿Qué hace el TKIP MIC Exploit como availability attack?
**A:** Genera datos TKIP inválidos para superar el umbral de errores MIC del AP objetivo → el AP suspende el servicio WLAN como medida de protección.

---

**Q:** ¿Qué es un Ad Hoc Association attack?
**A:** Fuerza a la red a operar en modo ad-hoc (sin AP), que es inherentemente inseguro sin autenticación ni cifrado robustos. El atacante puede conectarse fácilmente a un cliente en modo ad-hoc y comprometer también la LAN cableada.

---

**Q:** ¿Qué herramientas se usan para Eavesdropping y Traffic Analysis en ataques de confidencialidad wireless?
**A:** Wireshark, Ettercap, Kismet y analizadores comerciales.

---

**Q:** ¿Cuáles son las dos formas de Unauthorized Association attack?
**A:** (1) Maliciosa: mediante soft APs (laptop con NIC configurada como AP falso). (2) Accidental: conectarse al AP de una organización vecina sin conocimiento.

---

**Q:** ¿Qué ataque captura mensajes RADIUS Access-Accept o Reject para posterior replay?
**A:** RADIUS Replay — tipo de integrity attack.

---

**Q:** ¿Qué herramientas se usan para crear un Evil Twin AP?
**A:** Hostapd, EvilTwinFramework, Wifiphisher.

---

**Q:** ¿Qué hace el Power Saving Attack?
**A:** Transmite un TIM (Traffic Indication Map) o DTIM falso a un cliente en modo power-saving, haciéndolo vulnerable a un ataque DoS.

---

**Q:** ¿Qué herramientas se usan para crackear credenciales de login de dominio Windows en ataques de autenticación wireless?
**A:** John the Ripper, L0phtCrack, THC-Hydra.

---

**Q:** ¿Cuáles son las herramientas para VPN Login Cracking distinguiendo IPsec de PPTP?
**A:** IPsec: ike_scan + IKECrack. PPTP: Anger + THC-pptp-bruter.

---

**Q:** ¿Qué hace un De-authenticate Flood attack?
**A:** Inunda clientes con frames forjados de de-autenticación o disasociación para desconectarlos del AP. Herramienta: AirJack.

---

## 5. Confusión frecuente

**Evil Twin AP vs. Honeypot AP vs. Promiscuous Client**
- Evil Twin AP: el AP falso transmite el SSID exacto de la red legítima activamente (beacon WLAN's SSID). Categoría: confidentiality.
- Honeypot AP: similar al evil twin pero con énfasis en la señal más fuerte para atraer pasivamente dispositivos buscando la mejor señal. También puede tener el mismo SSID.
- Promiscuous Client: el atacante no crea el AP para suplantar — crea AP con SSID común y señal más fuerte aprovechando que las tarjetas 802.11 buscan la señal más intensa. Categoría: access control.
- Criterio: "beacon activo del SSID de la red real" → Evil Twin. "Señal más fuerte + mismo SSID" → Honeypot. "Señal irresistiblemente fuerte con SSID genérico" → Promiscuous Client.

---

**Beacon Flood vs. Authenticate Flood — ambas floods de disponibilidad**
- Beacon Flood: miles de beacons falsos → clientes no pueden encontrar APs legítimos (ataca la discovery/visibilidad).
- Authenticate Flood: autenticaciones forjadas desde MACs aleatorias → llena la tabla de asociación del AP (ataca la capacidad del AP para nuevas conexiones).
- Criterio: "clientes no pueden encontrar el AP legítimo" → Beacon Flood. "AP no puede aceptar nuevas conexiones" → Authenticate Flood.

---

**Wormhole vs. Sinkhole — routing attacks similares**
- Wormhole: crea túnel artificial entre dos nodos específicos (S y D); manipula RREQ/RREP; afecta una ruta específica.
- Sinkhole: el nodo malicioso se posiciona cerca de la base station y atrae TODO el tráfico anunciando ser la ruta más corta; puede combinarse con wormhole.
- Criterio: "túnel entre dos nodos específicos con manipulation de route discovery" → Wormhole. "Atracción de todo el tráfico de la red hacia un nodo malicioso central" → Sinkhole.

---

**LEAP vs. WPA PSK Cracking — herramientas diferentes**
- LEAP Cracking (Asleap, THC-LEAPcracker): crackea el NT password hash del protocolo LEAP propietario de Cisco.
- PSK Cracking (Cowpatty, Fern Wifi Cracker): dictionary attack sobre frames del WPA four-way handshake capturados.
- Criterio: "protocolo Cisco, NT hash" → LEAP cracking. "WPA handshake frames" → PSK cracking.

---

**Disassociation Attack vs. De-authenticate Flood**
- Disassociation Attack: destruye la conectividad entre un AP y un cliente específico (targeted).
- De-authenticate Flood: inunda múltiples clientes con frames forjados de de-autenticación/disasociación (masivo, volumétrico). Herramienta: AirJack.
- Criterio: "romper conectividad de un cliente específico" → Disassociation. "Desconectar masivamente usuarios con flood de frames" → De-authenticate Flood.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un atacante configura un Access Point no autorizado con el mismo SSID que la red corporativa y una antena de alta ganancia que le proporciona señal más fuerte que los APs legítimos. Los dispositivos de la zona se conectan automáticamente al AP del atacante. ¿Qué tipo de ataque wireless representa este escenario?

A) Evil Twin AP, que finge ser AP autorizado haciendo beacon del SSID corporativo
B) Honeypot AP, que usa señal superior para atraer pasivamente a dispositivos hacia el AP falso
C) Rogue AP, que proporciona acceso no autorizado a la red corporativa desde dentro del perímetro
D) Client Mis-association, donde el cliente se conecta accidentalmente a un AP externo

**Respuesta correcta: B**
El Honeypot AP se caracteriza por configurar el mismo SSID que un AP legítimo Y usar señal más fuerte que los APs legítimos, atrayendo pasivamente a dispositivos que buscan la señal más potente. El Evil Twin también imita el SSID pero su énfasis está en hacerse pasar por AP autorizado con KARMA; el Honeypot se enfoca en la señal superior para atracción pasiva.

---

**P2.** Un atacante observa que la red corporativa usa IEEE 802.1X con EAP. Decide capturar el intercambio EAP válido entre un cliente legítimo y el servidor de autenticación, y luego enviar al cliente un mensaje EAP-Failure forjado para interrumpir el servicio. ¿En qué categoría de ataques wireless se clasifica este ataque y qué herramienta se usa?

A) Authentication attack → herramienta: Asleap para crackear credenciales EAP capturadas
B) Integrity attack → herramienta: Airpwn-ng para inyectar frames EAP forjados
C) Availability attack tipo EAP-Failure → herramienta: Airtool Pi para enviar EAP-Failure forjado
D) Confidentiality attack → herramienta: Wireshark para capturar y decodificar intercambio EAP

**Respuesta correcta: C**
El ataque EAP-Failure es un Availability attack: observa un intercambio EAP 802.1X válido y luego envía al cliente un mensaje EAP-Failure forjado, interrumpiendo la conectividad del usuario. La herramienta específica mencionada en el temario CEH es Airtool Pi. No es un integrity attack (no modifica los datos EAP) ni authentication (no intenta robar credenciales).

---

**P3.** Un atacante tiene acceso a la red corporativa wireless e intenta capturar credenciales de otros usuarios. Configura un entorno para que los clientes legítimos lo consideren el destino final de sus peticiones. ¿Qué tipo de ataque de confidencialidad wireless describe este escenario?

A) Eavesdropping: captura y decodifica tráfico de aplicaciones no cifrado con Wireshark
B) Session Hijacking: manipula la red para que el host del atacante aparezca como el destino deseado
C) Traffic Analysis: infiere información observando patrones externos del tráfico sin descifrarlo
D) Masquerading: el atacante finge ser un usuario autorizado para acceder al sistema

**Respuesta correcta: B**
Session Hijacking en wireless implica manipular la red para que el host del atacante aparezca como el destino deseado, interceptando las sesiones. Se diferencia del eavesdropping (pasivo, solo captura) y del masquerading (fingir ser usuario autorizado para acceder, no para interceptar). Requiere herramientas de capture e injection entre AP y servidor de autenticación.

---

**P4.** Un analista detecta que un atacante está realizando un Wormhole Attack en la red de sensores inalámbricos de una instalación industrial. ¿Cuál es el mecanismo específico que hace efectivo este ataque en redes que usan AODV o DSR?

A) El atacante retransmite todas las peticiones de routing con TTL incrementado para atraer el tráfico
B) El atacante anuncia que su nodo tiene la ruta más corta, crea un túnel entre S y D via su nodo, y reenvía RREQ/RREP antes que los originales
C) El atacante inunda la red con RREQ falsos para saturar las tablas de routing de los nodos legítimos
D) El atacante bloquea físicamente las comunicaciones directas entre nodos y fuerza el enrutamiento por su nodo

**Respuesta correcta: B**
El Wormhole Attack explota RREQ (Route Request) y RREP (Route Reply) de AODV/DSR. El atacante se posiciona estratégicamente, anuncia que tiene la ruta más corta, reenvía RREQ de S→D y RREP de D→S antes que los originales, estableciendo un enlace directo falso vía su nodo malicioso. Esto le da control del flujo de datos entre origen y destino.

---

**P5.** Un ingeniero de seguridad wireless quiere entender la diferencia entre un Beacon Flood y un Authenticate Flood. ¿Cuál es la distinción correcta entre ambos ataques de disponibilidad?

A) Beacon Flood afecta solo a nuevas conexiones; Authenticate Flood afecta a conexiones establecidas
B) Beacon Flood genera miles de beacons falsos dificultando encontrar el AP legítimo; Authenticate Flood envía auth/associate desde MACs aleatorias llenando la tabla de asociación del AP
C) Beacon Flood usa herramienta AirJack; Authenticate Flood usa Airtool Pi; son equivalentes en impacto
D) Beacon Flood y Authenticate Flood son el mismo ataque con diferentes nombres según el vendor

**Respuesta correcta: B**
La distinción es el vector: Beacon Flood genera miles de beacons 802.11 falsos → dificulta a clientes encontrar el AP legítimo (ataca la discovery). Authenticate Flood envía authenticates/associates forjados desde MACs aleatorias → llena la tabla de asociación del AP objetivo (ataca la capacidad del AP). Ambas usan AirJack pero con impactos diferentes.

---

**P6.** Durante un análisis forense wireless, un analista identifica que el atacante realizó un "Bit-Flipping Attack" contra la red WEP. ¿Qué hace este ataque y por qué es posible en WEP específicamente?

A) El atacante fuerza IVs débiles repitiendo el mismo IV en múltiples paquetes para analizar patrones
B) El atacante captura un frame, invierte bits aleatorios en el payload, modifica el ICV y lo reenvía; posible porque CRC-32 no es hash criptográfico
C) El atacante usa fuerza bruta sobre el campo IV para reconstruir el keystream RC4 completo
D) El atacante inyecta bits adicionales en el stream de datos para crear colisiones en la función de hash WEP

**Respuesta correcta: B**
El Bit-Flipping Attack en WEP captura un frame, invierte bits aleatorios en el payload, ajusta el ICV (Integrity Check Value) correspondientemente y reenvía el frame modificado. Es posible porque WEP usa CRC-32 para integridad, que no es un hash criptográfico — al ser una función matemática predictable, el atacante puede calcular el ICV correcto para el payload modificado.

---

**P7.** Un analista identifica que el atacante usó un Inter-Chip Privilege Escalation attack. ¿Cuál es el mecanismo único de este ataque y qué lo diferencia de otros ataques wireless?

A) Explota vulnerabilidades en el chipset Wi-Fi para escalar privilegios en el sistema operativo del dispositivo víctima
B) Explota chips combinados (combo chips) donde un chip Bluetooth puede capturar datos o manipular tráfico del chip Wi-Fi
C) Usa la antena compartida entre Bluetooth y Wi-Fi para inyectar frames 802.11 con identidad Bluetooth
D) Aprovecha la coexistencia de 2.4 GHz de Bluetooth y Wi-Fi para crear interferencias dirigidas

**Respuesta correcta: B**
El Inter-Chip Privilege Escalation (también llamado Wireless Co-Existence Attack) explota vulnerabilidades en chips combinados (combo chips) que manejan Bluetooth y Wi-Fi simultáneamente. Un chip Bluetooth comprometido puede capturar credenciales u otros datos sensibles del chip Wi-Fi o manipular su tráfico, cruzando los límites del chip (chip boundaries). Es único porque explota la arquitectura de hardware combinado, no el protocolo wireless en sí.

---

**P8.** Un atacante ha comprometido un AP legítimo de una red corporativa inyectando un virus. El AP comprometido ahora envía frames 802.11 forjados que contienen claves WEP falsas. ¿En qué categoría de ataque wireless se clasifica esta técnica?

A) Access Control attack → Unauthorized Association usando el AP como soft AP comprometido
B) Confidentiality attack → Evil Twin AP que se hace pasar por el AP legítimo con firma WEP
C) Integrity attack → WEP Injection que construye y envía claves de cifrado WEP forjadas
D) Authentication attack → Shared Key Guessing usando las claves WEP generadas por el virus

**Respuesta correcta: C**
WEP Injection es un Integrity attack que construye y envía claves de cifrado WEP forjadas. Los Integrity attacks se centran en alterar datos durante la transmisión y enviar frames forjados. El hecho de que el AP esté comprometido por un virus es el vector de acceso inicial, pero el ataque en sí (inyectar claves WEP falsas) es de integridad.

---

**P9.** Un atacante realiza un PSK Cracking attack sobre una red WPA2-Personal. Ha capturado el four-way handshake. ¿Qué herramientas específicas se usan para este ataque y qué técnica emplean?

A) Aircrack-ng y WEPCrack → análisis estadístico de IVs débiles capturados
B) Asleap y THC-LEAPcracker → crackeado del NT password hash del protocolo LEAP
C) Cowpatty y Fern Wifi Cracker → dictionary attack sobre los frames del handshake capturado
D) Reaver y Pixie Dust → brute-force del PIN WPS para recuperar la PSK indirectamente

**Respuesta correcta: C**
Para PSK Cracking (recuperar WPA/WPA2 PSK), las herramientas específicas del temario CEH son Cowpatty y Fern Wifi Cracker, usando dictionary attacks sobre los frames del handshake capturado. Aircrack-ng es para WEP; Asleap para LEAP (protocolo propietario Cisco); Reaver para WPS PIN attacks. La confusión Aircrack-ng vs Cowpatty para PSK es una trampa clásica del examen.

---

**P10.** Un analista detecta actividad de un atacante que está capturando mensajes RADIUS Access-Accept para usarlos posteriormente. ¿Qué tipo de ataque de integridad wireless describe este escenario?

A) RADIUS Injection: el atacante modifica los mensajes RADIUS antes de enviarlos al servidor
B) Extensible AP Replay: el atacante captura protocolos EAP 802.1X para posterior replay
C) RADIUS Replay: el atacante captura mensajes RADIUS Access-Accept o Reject para posterior replay
D) Identity Theft: el atacante captura identidades de usuario de paquetes cleartext de 802.1X

**Respuesta correcta: C**
RADIUS Replay es el ataque de Integrity que específicamente captura mensajes RADIUS Access-Accept o Access-Reject para posterior replay. Se diferencia del Extensible AP Replay (que captura protocolos EAP 802.1X) y del Identity Theft (que roba identidades de paquetes cleartext). Ambos RADIUS Replay y EAP Replay son ataques de integridad porque permiten falsificar el resultado de la autenticación.
