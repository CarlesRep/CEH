# M10_04_DoSDDoSAttackTechniques.md
## CEH v13 — Módulo 10 | DoS/DDoS Attack Techniques

---

## 1. Conceptos y definiciones

### 🔴 Las 3 categorías de vectores de ataque DoS/DDoS

| Categoría | Recurso agotado | Unidad de medida | Ejemplos de técnicas |
|---|---|---|---|
| **Volumetric** | Ancho de banda (red) | **bps** (bits per second) | UDP flood, ICMP flood, Smurf, NTP amplification, PoD, Pulse wave |
| **Protocol** | Connection state tables (firewalls, LBs, app servers) | **pps** (packets/s) o **cps** (connections/s) | SYN flood, Fragmentation, Spoofed session flood, ACK flood, SYN-ACK flood |
| **Application Layer** | Recursos de aplicación | **rps** (requests per second) | HTTP flood, Slowloris, UDP app layer flood, DDoS extortion |

**Subtipo de Volumetric**:
- **Flood attack**: zombies envían grandes volúmenes de tráfico directamente al sistema víctima.
- **Amplification attack**: el atacante/zombies envían mensajes a una dirección IP broadcast; el tráfico se amplifica al responder múltiples hosts y consume el ancho de banda de la víctima.

**Protocolos objetivo típicos de ataques volumétricos**: NTP, DNS, SSDP — son stateless y sin mecanismos de congestión integrados.

---

### 🔴 Técnicas de ataque — Detalle por técnica

#### UDP Flood Attack *(Volumetric)*
- El atacante envía paquetes UDP spoofed a **puertos aleatorios** del servidor víctima con un gran rango de IPs de origen.
- El servidor verifica repetidamente si hay aplicaciones en esos puertos → no las encuentra → responde con ICMP "**Destination Unreachable**".
- Resultado: agotamiento de recursos de red y ancho de banda hasta que el sistema queda offline.

#### ICMP Flood Attack *(Volumetric)*
- Envío masivo de **ICMP echo request packets** directamente o a través de redes de reflexión.
- El sistema víctima responde a cada petición, saturando su ancho de banda.
- **Umbral de protección por defecto**: **1.000 paquetes/s** → al superarse, el router rechaza ICMP echo requests de todas las direcciones en la misma zona de seguridad durante el segundo actual y el siguiente.

#### Ping of Death (PoD) *(Volumetric)*
- Envío de paquetes malformados o de **tamaño excesivo** mediante ping.
- El límite RFC 791 para paquetes IP es **65.535 bytes**.
- El PoD envía paquetes de **65.538 bytes** (3 bytes por encima del límite).
- El proceso de reensamblado en el sistema receptor puede causar crash.
- La identidad del atacante se puede falsificar fácilmente; solo necesita la IP del objetivo.

#### Smurf Attack *(Volumetric / Amplification)*
- El atacante falsifica la IP de origen con la IP de la víctima.
- Envía una gran cantidad de **ICMP ECHO requests** a una dirección **IP broadcast**.
- Todos los hosts de la red broadcast responden → respuestas van a la IP víctima (spoofed).
- Resultado: flood masivo de tráfico hacia la víctima → crash.

#### Pulse Wave DDoS Attack *(Volumetric)*
- Patrón **periódico** (no continuo): pulsos de ataque cada **~10 minutos**.
- Duración de la sesión: aproximadamente **una hora o varios días**.
- Cada pulso: **300 Gbps o más** → suficiente para saturar completamente un enlace de red.
- Recuperación muy difícil y a veces imposible.

#### Zero-Day DDoS Attack *(Volumetric)*
- Explota vulnerabilidades de DDoS **sin parche disponible** y sin mecanismos defensivos efectivos.
- El atacante bloquea todos los recursos de la víctima y roba datos hasta que se identifica la estrategia y se despliega un parche.
- No existe un enfoque universal de protección actualmente.

#### NTP Amplification Attack *(Volumetric / Amplification)*
- Usa una botnet para enviar **grandes paquetes UDP** con IP spoofed (IP de la víctima) a servidores NTP.
- Requiere que el servidor NTP tenga el comando **monlist** habilitado.
- Cada paquete UDP activa una petición `monlist` → el servidor NTP genera **respuestas masivas** enviadas a la IP de la víctima (spoofed).
- Resultado: la IP víctima es inundada con respuestas grandes → congestión de red, agotamiento de ancho de banda, memoria y energía.

**Comando nmap para recuperar monlist**:
```
nmap -sU -pU:123 -Pn -n --script=ntp-monlist <target>
```

**Contramedidas**: endurecer configuración del servidor NTP · deshabilitar monlist · limitar flow control · monitorizar tráfico · implementar zero-trust · firewalls para filtrar peticiones NTP.

#### SYN Flood Attack *(Protocol)*
- El atacante envía múltiples **SYN requests** con IPs de origen falsas al servidor víctima.
- El servidor responde con **SYN/ACK** y espera el **ACK final** (three-way handshake).
- El ACK final nunca llega → las conexiones parciales quedan en la **listen queue**.
- El servidor mantiene cada conexión parcialmente abierta durante **75 segundos**.
- Cuando la listen queue se llena, el servidor no puede aceptar nuevas conexiones legítimas.

**Contramedidas**: packet filtering · reducir timeout en estado "SYN RECEIVED" · reducir retransmisiones · **SYN cookies** · **SynAttackProtect**.

#### SYN-ACK Flood Attack *(Protocol)*
- Similar al SYN flood, pero explota la **segunda fase** del three-way handshake.
- El atacante envía masivamente paquetes **SYN-ACK** para agotar recursos del servidor.

#### ACK y PUSH ACK Flood Attack *(Protocol)*
- Envío masivo de paquetes **ACK** y **PUSH ACK** spoofed durante una sesión TCP activa.
- Deja al servidor no funcional.

#### Fragmentation Attack *(Protocol)*
- Envío masivo de paquetes fragmentados de **1.500+ bytes** a una tasa de paquetes relativamente baja.
- Los paquetes fragmentados generalmente no se inspeccionan al pasar por routers, firewalls e IDS/IPS (el protocolo permite la fragmentación).
- El reensamblado e inspección de estos paquetes grandes consume recursos excesivos.
- El contenido de los fragmentos está **aleatorizado por el atacante**, lo que complica aún más el reensamblado y consume más recursos hasta crashear el sistema.

#### Spoofed Session Flood Attack *(Protocol)*
- Crea sesiones TCP falsas con combinaciones de paquetes SYN, ACK y RST/FIN.
- Objetivo: eludir firewalls y agotar recursos de red.
- Subtipos:
  - **Multiple SYN-ACK**: múltiples SYN + múltiples ACK + uno o más RST/FIN.
  - **Multiple ACK**: omite completamente los SYN → usa solo ACK + RST/FIN. El firewall no detecta el ataque porque los filtros basados en SYN no se activan.

#### HTTP GET/POST Attack *(Application Layer — Layer 7)*
- **HTTP GET**: cabecera HTTP con retardo temporal → el servidor mantiene la conexión abierta esperando la petición completa que nunca llega → agotamiento de conexiones.
- **HTTP POST**: cabecera completa pero **cuerpo incompleto** → el servidor espera el resto del cuerpo → web server/app inaccesible.
- No usa paquetes malformados, spoofing ni técnicas de reflexión.
- **Requiere menos ancho de banda** que otros ataques.
- Todos los parámetros de red parecen normales → difícil detección.

**Subtipos HTTP flood**:
- **Single-Session HTTP Flood**: múltiples peticiones en una sola sesión HTTP (explota HTTP 1.1).
- **Single-Request HTTP Flood**: múltiples peticiones HTTP ocultas dentro de un único paquete HTTP → anonimato total.
- **Recursive GET Flood**: simula navegación legítima por páginas/imágenes mientras ejecuta flood → engaña al firewall.
- **Random Recursive GET Flood**: variante para foros/blogs con páginas en secuencia; usa números de página aleatorios del rango válido para simular un usuario legítimo.

#### Slowloris Attack *(Application Layer — Layer 7)*
- Envía **peticiones HTTP parciales** al servidor → el servidor abre múltiples conexiones y espera a que se completen.
- Las peticiones permanecen incompletas indefinidamente → el pool máximo de conexiones concurrentes se llena → nuevas conexiones denegadas.
- Usa **tráfico HTTP legítimo** → extremadamente difícil de detectar.
- Distinto de otros ataques: no requiere alto volumen de tráfico.

#### UDP Application Layer Flood Attack *(Application Layer)*
- Aunque UDP flood es típicamente volumétrico, algunos protocolos de aplicación basados en UDP pueden usarse para ataques de capa de aplicación.
- **Protocolos UDP explotables**: CHARGEN · SNMPv2 · QOTD · RPC · SSDP · CLDAP · TFTP · NetBIOS · NTP · Quake Network Protocol · Steam Protocol · VoIP.

#### Multi-Vector Attack
- Combinación de ataques volumétricos, de protocolo y de capa de aplicación **simultáneos o secuenciales**.
- El atacante cambia rápidamente entre vectores (p. ej., SYN → Layer 7).
- Objetivo: confundir al equipo de TI, dispersar recursos de defensa y desviar el foco.

#### Peer-to-Peer Attack
- Explota bugs en **servidores peer-to-peer** que usan el protocolo **DC++ (Direct Connect)**.
- **No usa botnets**. El atacante no necesita comunicarse con los clientes subvertidos.
- Instruye a clientes de grandes hubs P2P a desconectarse de la red P2P y conectarse al sitio víctima.
- Miles de equipos intentan conectar al objetivo simultáneamente → degradación del servicio.
- **Fácilmente identificable** por firmas.
- **Contramedida**: especificar puertos para comunicación P2P (p. ej., deshabilitar P2P en puerto 80).

#### Permanent DoS (PDoS) — "Phlashing"
- Apunta exclusivamente al **hardware**: causa daño irreversible que requiere **reemplazo o reinstalación** del hardware.
- También conocido como **"bricking"** del sistema.
- Explota flaws de administración remota en interfaces de dispositivos (impresoras, routers, dispositivos de red).
- Vector: emails, IRC chats, tweets o vídeos con **actualizaciones de firmware fraudulentas** (corruptas o con vulnerabilidades).
- No requiere gran cantidad de recursos (a diferencia del DDoS).
- Más rápido y destructivo que los DoS convencionales.

#### TCP SACK Panic Attack
- Ataque remoto específico contra **sistemas Linux**.
- Explota una vulnerabilidad de **integer overflow** en el **Linux Socket Buffer (SKB)**.
- Utiliza el mecanismo **TCP SACK** (Selective Acknowledgment): el emisor solo retransmite los paquetes no confirmados.
- El socket buffer de Linux puede almacenar un máximo de **17 segmentos**.
- El atacante envía paquetes SACK con **MSS mínimo (48 bytes)** → aumenta el número de segmentos TCP → el socket buffer excede 17 segmentos → integer overflow → **kernel panic** → DoS.
- También efectivo contra **contenedores y máquinas virtuales** (la vulnerabilidad está en el kernel stack).
- **Contramedidas**: parchear la vulnerabilidad · implementar regla de firewall para bloquear paquetes con MSS mínimo.

#### DRDoS (Distributed Reflection DoS) — "Spoofed Attack"
- Usa múltiples **intermediaries (zombies)** y **secondary victims (reflectors)**.
- Flujo:
  1. Atacante ordena a zombies enviar paquetes **TCP SYN** con la IP del objetivo como IP de origen hacia los reflectors.
  2. Los reflectors envían **SYN/ACK** a la IP del objetivo (creyendo que éste inició la conexión).
  3. El objetivo descarta los SYN/ACK (no envió los SYN).
  4. Los reflectors reenvían SYN/ACK hasta timeout → flood sostenido.
- **Por qué es más efectivo que DDoS**: los reflectors (no comprometidos) generan ancho de banda masivo. El atacante real es prácticamente imposible de rastrear.
- **Contramedida**: desactivar el servicio **CHARGEN**.

#### DDoS Extortion / Ransom DDoS (RDDoS)
- El atacante amenaza con un ataque DDoS y exige un **rescate** (en criptomoneda).
- Puede ejecutar un **ataque DDoS de muestra** para demostrar capacidad.
- El email de rescate incluye: importe, plazo, método de pago, amenazas sobre vulnerabilidades o datos expuestos.
- **Contramedidas**: implementar defensa DDoS efectiva · reportar inmediatamente a fuerzas del orden · evaluar activos regularmente · implementar **BGP/DNS swing** y protección **always-on**.

---

### 🔴 Herramientas de ataque DoS/DDoS

| Herramienta | Descripción |
|---|---|
| **HOIC** (High Orbit Ion Cannon) | Herramienta DDoS; usada en el escenario de volunteers |
| **LOIC** (Low Orbit Ion Cannon) | Herramienta DDoS de uso común |
| **Slowloris** | Herramienta de Layer-7 DDoS; usa HTTP legítimo; ataca infraestructura web |
| **HULK** | Herramienta de DDoS HTTP |
| **ISB** (I'm so bored) | HTTP, UDP, TCP e ICMP flood; incluye comandos de red (WHOIS, netstat, traceroute, ping) |
| **UltraDDOS-v2** | GUI simple; configura IP destino, puerto y número de paquetes |
| **UFONet** | DDoS tool |
| **Packet Flooder Tool** | Herramienta de flood de paquetes |

---

## 2. Exam Traps ⚠️

⚠️ **[Unidades de medida por categoría]** El examen pregunta directamente: Volumetric → **bps**. Protocol → **pps/cps**. Application Layer → **rps**. No confundirlos.

⚠️ **[PoD — límite RFC 791: 65.535 bytes; ataque: 65.538 bytes]** El examen puede preguntar el tamaño del paquete en un PoD. El límite es 65.535; el PoD supera ese límite (p. ej., 65.538).

⚠️ **[ICMP Flood — umbral por defecto: 1.000 paquetes/s]** Dato numérico concreto: el router activa la protección anti-ICMP flood cuando se superan **1.000 paquetes/s** y bloquea durante el segundo actual y el siguiente.

⚠️ **[SYN Flood — listen queue: 75 segundos]** El servidor mantiene conexiones parcialmente abiertas durante **75 segundos**. El examen puede preguntar este valor.

⚠️ **[Smurf Attack — IP broadcast + ICMP + spoofing]** El Smurf es un ataque de **amplificación** que usa una red broadcast como amplificador. No confundir con ICMP flood directo.

⚠️ **[Pulse Wave — cada 10 minutos, 300 Gbps+]** Datos numéricos específicos: pulsos cada **~10 minutos**, cada pulso de **300 Gbps o más**.

⚠️ **[NTP Amplification — comando monlist]** El ataque requiere que el servidor NTP tenga habilitado el comando **monlist**. Es el detalle técnico discriminante.

⚠️ **[Fragmentation Attack — paquetes 1.500+ bytes, no inspeccionados]** Los paquetes fragmentados no se inspeccionan al pasar por routers/firewalls/IDS porque el protocolo permite la fragmentación. El contenido está aleatorizado.

⚠️ **[Multiple ACK Spoofed Session Flood — sin SYN]** Es la variante que **omite completamente los SYN**. Como los firewalls usan filtros SYN para detectar tráfico anormal, esta variante tiene una tasa de detección muy baja.

⚠️ **[HTTP GET vs. HTTP POST flood]** GET: cabecera retrasada, la petición nunca se completa. POST: **cabecera completa pero cuerpo incompleto**. El servidor espera el resto del cuerpo en ambos casos.

⚠️ **[Slowloris — tráfico HTTP legítimo]** Slowloris es "distinctly different" porque usa tráfico HTTP perfectamente legítimo. No hay spoofing, malformed packets ni reflexión.

⚠️ **[Peer-to-Peer Attack — no usa botnets]** Punto crítico: P2P attack **no usa botnets**. Usa el protocolo **DC++**. El examen puede intentar asociarlo con botnets.

⚠️ **[PDoS — "phlashing" y "bricking"]** El PDoS tiene dos alias: **phlashing** (nombre técnico del ataque) y **bricking** (el método específico de daño al hardware mediante firmware corrupto).

⚠️ **[TCP SACK Panic — MSS mínimo: 48 bytes, límite socket buffer: 17 segmentos]** Dos datos numéricos específicos del examen. MSS mínimo = **48 bytes**. Límite socket buffer = **17 segmentos**.

⚠️ **[DRDoS — contramedida: deshabilitar CHARGEN]** La contramedida específica para DRDoS es desactivar el servicio **CHARGEN (Character Generator Protocol)**.

⚠️ **[DDoS Extortion contramedida: BGP/DNS swing]** Si el examen pregunta contramedidas específicas de RDDoS → **BGP/DNS swing** y **always-on protection service**.

---

## 3. Nemotécnicos

### 3 categorías + unidades — "V-P-A → bps-pps-rps"
**"Very Protocol Applications" → bps · pps/cps · rps**

### Técnicas volumétricas — "U-I-P-S-PW-Z-NTP"
**"Under Ice Polar Seas Pulse Zeros, NTP"**
- **U**DP flood · **I**CMP flood · **P**oD · **S**murf · **P**ulse **W**ave · **Z**ero-day · **NTP** amplification

### Técnicas de protocolo — "SYN-F-SS-ACK-SYNACK"
**"SYNs Fragment Spoofed Sessions, ACKs SYN-ACKs"**
- **SYN** flood · **F**ragmentation · **S**poofed **S**ession · **ACK** flood · **SYN-ACK** flood

### Técnicas de aplicación — "HTTP-SL-UDP-EXT"
**"HTTP Slowly Used for Extortion"**
- **HTTP** flood · **SL**owloris · **UDP** app layer · **EXT**ortion

### Datos numéricos críticos — tabla rápida
| Técnica | Número clave |
|---|---|
| PoD | 65.535 bytes (límite RFC 791); 65.538 bytes (tamaño ataque) |
| ICMP Flood | 1.000 paquetes/s (umbral por defecto) |
| SYN Flood | 75 segundos (tiempo en listen queue) |
| Pulse Wave | ~10 min entre pulsos; 300 Gbps+ por pulso |
| TCP SACK Panic | MSS = 48 bytes; socket buffer = 17 segmentos |
| NTP Amplification | Puerto UDP 123; comando monlist |

### PDoS — "Ph-Br"
- **Ph**lashing = nombre del ataque
- **Br**icking = método de ejecución (firmware corrupto)

---

## 4. Flashcards

**Q:** ¿En qué unidades se mide la magnitud de cada categoría de ataque DoS/DDoS?
**A:** Volumetric → bps. Protocol → pps o cps. Application Layer → rps.

**Q:** ¿Cuál es el tamaño máximo de paquete IP según RFC 791 y qué tamaño usa el PoD para superarlo?
**A:** RFC 791 establece 65.535 bytes. El PoD usa paquetes de 65.538 bytes (3 bytes por encima del límite).

**Q:** ¿Cuál es el umbral por defecto que activa la protección anti-ICMP flood en un router?
**A:** 1.000 paquetes/s. Al superarse, el router rechaza ICMP echo requests de todas las direcciones en la misma zona de seguridad durante el segundo actual y el siguiente.

**Q:** ¿Cuánto tiempo mantiene un servidor una conexión parcialmente abierta en el listen queue durante un SYN flood?
**A:** 75 segundos.

**Q:** ¿Cómo funciona el Smurf Attack?
**A:** El atacante falsifica la IP de la víctima y envía ICMP ECHO requests a una dirección IP broadcast. Todos los hosts de la red broadcast responden a la IP de la víctima (spoofed), generando un flood masivo de tráfico.

**Q:** ¿Cuál es el patrón temporal de un Pulse Wave DDoS y qué magnitud tiene cada pulso?
**A:** Pulsos periódicos cada ~10 minutos, cada uno de 300 Gbps o más. La sesión puede durar una hora o varios días.

**Q:** ¿Qué función del servidor NTP debe estar habilitada para que sea vulnerable a un ataque NTP Amplification?
**A:** El comando monlist.

**Q:** ¿Qué comando nmap se usa para recuperar el monlist de un servidor NTP?
**A:** `nmap -sU -pU:123 -Pn -n --script=ntp-monlist <target>`

**Q:** ¿Por qué la variante "Multiple ACK Spoofed Session Flood" tiene una tasa de detección muy baja?
**A:** Porque omite completamente los paquetes SYN. Los firewalls usan filtros de SYN para detectar tráfico anormal, por lo que sin SYN el ataque pasa desapercibido.

**Q:** ¿En qué se diferencia HTTP POST flood de HTTP GET flood?
**A:** En GET flood la cabecera está retrasada y la petición nunca se completa. En POST flood la cabecera está completa pero el cuerpo del mensaje está incompleto; el servidor espera el resto del cuerpo indefinidamente.

**Q:** ¿Por qué Slowloris es "distinctly different" de otras herramientas DDoS?
**A:** Porque usa tráfico HTTP perfectamente legítimo para tomar el servidor. No emplea paquetes malformados, spoofing ni técnicas de reflexión.

**Q:** ¿Qué protocolo de red usa el Peer-to-Peer attack y usa botnets?
**A:** Usa el protocolo DC++ (Direct Connect). No usa botnets.

**Q:** ¿Cuáles son los dos alias del Permanent DoS (PDoS)?
**A:** Phlashing (nombre del ataque) y bricking (el método de ejecución mediante firmware corrupto).

**Q:** ¿Cuáles son los dos datos numéricos clave del TCP SACK Panic Attack?
**A:** MSS mínimo = 48 bytes (usado por el atacante). Límite del socket buffer de Linux = 17 segmentos. Al superarse los 17 segmentos se produce integer overflow → kernel panic.

**Q:** ¿Cuál es la contramedida específica para el ataque DRDoS?
**A:** Desactivar el servicio CHARGEN (Character Generator Protocol).

**Q:** ¿Qué diferencia un DRDoS de un DDoS estándar en términos de efectividad y trazabilidad?
**A:** DRDoS es más efectivo porque los reflectors (no comprometidos) generan gran ancho de banda adicional. Es prácticamente imposible rastrear al atacante real porque los reflectors parecen ser los atacantes directos.

---

## 5. Confusión frecuente

### Volumetric vs. Protocol vs. Application Layer
- **Volumetric**: agota el ancho de banda; medido en **bps**; afecta la red.
- **Protocol**: agota las connection state tables de dispositivos intermedios (LB, firewall); medido en **pps/cps**.
- **Application Layer**: agota recursos de la aplicación; medido en **rps**; ocurre después de establecer conexión.
- **Criterio**: ¿se satura la red? → Volumetric. ¿Se saturan las tablas de conexión del firewall/LB? → Protocol. ¿Se satura la aplicación con peticiones aparentemente legítimas? → Application Layer.

### SYN Flood vs. SYN-ACK Flood vs. ACK Flood
- **SYN Flood**: explota la fase 1 del handshake; envía SYNs sin completar; llena la listen queue.
- **SYN-ACK Flood**: explota la fase 2; envía SYN-ACK masivos al servidor.
- **ACK/PUSH ACK Flood**: envía ACKs spoofed durante sesiones TCP activas.
- **Criterio**: ¿qué paquete se usa masivamente? SYN → SYN Flood. SYN-ACK → SYN-ACK Flood. ACK solo → ACK Flood.

### Smurf Attack vs. ICMP Flood
- **Smurf**: usa **IP broadcast** como amplificador; IP de origen spoofed con IP de la víctima; todos los hosts de la red responden a la víctima.
- **ICMP Flood**: envío directo de ICMP echo requests a la víctima; sin amplificación por broadcast.
- **Criterio**: ¿hay una red broadcast como intermediaria? → Smurf. ¿Es envío directo? → ICMP Flood.

### PDoS vs. DoS/DDoS
- **PDoS**: daño permanente al hardware (firmware corrupto); requiere reemplazo físico; no necesita muchos recursos; más rápido y destructivo.
- **DoS/DDoS**: interrupción temporal del servicio; el sistema se puede recuperar; requiere tráfico masivo (DDoS) o un único vector (DoS).
- **Criterio**: ¿el daño es físico/permanente y requiere reemplazo? → PDoS. ¿Es interrupción recuperable? → DoS/DDoS.

### HTTP GET Flood vs. Slowloris
- **HTTP GET Flood**: peticiones GET completas o incompletas a alta velocidad para agotar recursos.
- **Slowloris**: peticiones HTTP **parciales** mantenidas abiertas indefinidamente; no requiere alto volumen; usa tráfico legítimo; llena el pool de conexiones concurrentes.
- **Criterio**: ¿el ataque satura por volumen de peticiones? → HTTP GET Flood. ¿Mantiene conexiones parciales abiertas con bajo volumen? → Slowloris.
