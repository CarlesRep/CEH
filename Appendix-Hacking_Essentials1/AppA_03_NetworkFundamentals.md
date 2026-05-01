# AppA_03_NetworkFundamentals.md
# CEH v13 — Appendix A: Computer Network Fundamentals

---

## 1. Conceptos y definiciones

### Modelos de red: OSI vs TCP/IP 🔴

El **modelo OSI** es el modelo de referencia estándar para comunicación entre dos usuarios finales. Define 7 capas separando claramente servicios, interfaces y protocolos. Las **4 capas superiores** (Host Layers) gestionan la comunicación hacia/desde el usuario; las **3 inferiores** (Media Layers) gestionan el tránsito por la red.

| Capa OSI | Nº | PDU | Función clave |
|---|---|---|---|
| Application | 7 | Data | Proceso de red a aplicación |
| Presentation | 6 | Data | Representación, cifrado/descifrado, conversión de formato |
| Session | 5 | Data | Comunicación interhost, gestión de sesiones entre apps |
| Transport | 4 | Segments | Conexión extremo a extremo, fiabilidad, control de flujo |
| Network | 3 | Packet/Datagram | Determinación de ruta, direccionamiento lógico |
| Data Link | 2 | Frame | Direccionamiento físico |
| Physical | 1 | Bit | Transmisión de señal y binario |

El **modelo TCP/IP** tiene 4 capas y corresponde a implementación práctica:

| TCP/IP | Equivalencia OSI | Protocolos clave |
|---|---|---|
| Application | 5+6+7 | FTP, TFTP, SMTP, Telnet, DNS, SNMP, DHCP, HTTP, SSH, SOAP, SIP, RADIUS, TACACS+, RIP |
| Transport | 4 | TCP, UDP, SSL, TLS |
| Internet | 3 | IP, IPv6, IPsec, ICMP, ARP, IGRP, EIGRP, OSPF, HSRP, VRRP, BGP |
| Network Access | 1+2 | FDDI, Token Ring, CDP, VTP, PPP, WEP, WPA, WPA2, TKIP, EAP, LEAP, PEAP, STP |

Diferencia fundamental: OSI solo admite **comunicación orientada a conexión**; TCP/IP admite **comunicación con y sin conexión**. OSI define claramente servicios/interfaces/protocolos; TCP/IP no establece esa distinción.

---

### Tipos de red 🔴

| Tipo | Cobertura | Característica definitoria |
|---|---|---|
| **LAN** | Edificio / organización | Propiedad de organizaciones privadas; comparte recursos entre PCs/workstations |
| **WAN** | Países / continentes | Comunicación entre múltiples ubicaciones remotas; rápida, fiable, económica |
| **MAN** | Ciudad completa | Puede ser privada o de telecomunicaciones |
| **PAN** | Área de trabajo individual ("room-size") | Señales de radio y ópticas |
| **CAN** | Campus universitario | Área geográfica limitada |
| **GAN** | Ilimitada | Combinación de redes interconectadas; Internet es ejemplo |
| **WLAN** | Rango de punto de acceso | IEEE 802.11; señales RF |

---

### Estándares Wi-Fi 🔴

| Estándar | Frecuencia (GHz) | Vel. máx. | Modulación | Uso/Nota |
|---|---|---|---|---|
| **802.11** | 2,4 | 2 Mbps | DSSS, FHSS | El original |
| **802.11a** | 5 | 54 Mbps | OFDM | — |
| **802.11b** | 2,4 | 11 Mbps | DSSS | — |
| **802.11g** | 2,4 | 54 Mbps | OFDM | — |
| **802.11n** | 2,4 / 5 | 150 Mbps | MIMO-OFDM | — |
| **802.11ac** | 5 | 866,7 Mbps | MIMO-OFDM | — |
| **802.11ax** | 2,4 a 5 | 2400 Mbps | 1024-QAM | Wi-Fi 6 |
| **802.11be** | 2,4 / 5 / 6 | 3000 Mbps | QAM | Wi-Fi 7 |
| **802.11d** | — | — | — | Portabilidad global: variación de frecuencias, niveles de potencia y ancho de banda |
| **802.11e** | — | — | — | QoS: priorización de datos, voz y vídeo |
| **802.11i** | — | — | — | Cifrado mejorado para WLANs (802.11a/b/g) |
| **Bluetooth** | 2,4–2,485 | < 1 Mbps | FHSS | IEEE **802.15**; hasta 10 metros |

---

### Tecnologías inalámbricas adicionales

**WiMAX** (IEEE 802.16): alternativa inalámbrica a cable/DSL/T1/E1. Proporciona banda ancha a dispositivos móviles y fijos a varias millas.

**Microwave**: ondas de radio de alta frecuencia, comunicaciones **punto a punto**. Limitación: solo funciona en **línea de visión directa (line of sight)**.

**OWC (Optical Wireless Communication)**: transmisión no guiada con portadoras ópticas.
- **VLC**: banda visible (390–750 nm); LEDs pulsantes.
- **OWC punto a punto** (free space optical): frecuencias IR (750–1600 nm); láser; **10 Gbit/s por longitud de onda**.
- **UVC**: espectro solar blind UV (200–280 nm).

**Generaciones móviles** 🔴:

| Generación | Estándar | Datos descarga | Datos subida |
|---|---|---|---|
| **2G** (GSM) | Digital | — | — |
| **2.5G** (GPRS) | 2G + GPRS | 114 Kbit/s | 20 Kbit/s |
| **2.75G** (EDGE) | Enhanced GSM | 384 Kbit/s | 60 Kbit/s |
| **3G** HSPA (HSDPA+HSUPA) | UMTS | 7,2 Mbit/s | 2 Mbit/s |
| **3.5G** (HSPA+, 2008) | Evolved HSPA | 337 Mbit/s | 34 Mbit/s |
| **4G** (LTE) | IMT-Advanced | 100 Mbit/s (alta movilidad) / **1 Gbit/s** (baja movilidad) | — |

**TETRA**: estándar europeo para radio móvil profesional (PMR/PAMR). Usuarios: policía, militares, ambulancias, transportes. La baja frecuencia cubre grandes áreas con pocos transmisores.

---

### Topologías de red

| Topología | Descripción |
|---|---|
| **Bus** | Dispositivos conectados a un cable central (bus) |
| **Star** | Dispositivos conectados a un hub/switch central |
| **Ring** | Bucle cerrado; datos viajan nodo a nodo |
| **Mesh** | Cada dispositivo tiene enlace punto a punto con todos los demás |
| **Tree** | Híbrido bus+star; grupos star conectados a backbone bus lineal |
| **Hybrid** | Combinación de dos o más topologías (Star-Bus o Star-Ring son las más usadas) |

---

### Hardware de red

| Componente | Función |
|---|---|
| **NIC** | Conecta y comunica el equipo con la red |
| **Repeater** | Amplifica la señal entrante |
| **Hub** | Conecta segmentos LAN; **todos ven todos los paquetes** |
| **Switch** | Como hub pero los paquetes solo llegan al **nodo destino** |
| **Router** | Recibe paquetes de un segmento y los reenvía a otro |
| **Bridge** | Combina dos segmentos y gestiona el tráfico |
| **Gateway** | Habilita comunicación entre distintos entornos y protocolos |

---

### LAN Technologies 🔴

| Tecnología | Estándar IEEE | Velocidad | Distancia máx. |
|---|---|---|---|
| **Ethernet** | 802.3 | 10 Mbps | 100 m (10Base-T) |
| **Fast Ethernet** | 802.3u | 100 Mbps | 100 m (TX) / 2000 m (FX) |
| **Gigabit Ethernet** | 802.3z / 802.3-2008 | 1000 Mbps | Variable según medio |
| **10 Gigabit Ethernet** | 802.3ae-2002 | 10 Gbps | Fibra óptica obligatoria |

**10 Gigabit Ethernet** es el único que usa **exclusivamente fibra óptica**.

**Fast Ethernet — 3 tipos**: 100BASE-TX (UTP cat.5), 100BASE-FX (fibra óptica), 100BASE-T4 (UTP cat.3 + 2 hilos extra).

**Gigabit Ethernet**: también llamado **1000Base-T o "Gigabit-Ethernet-over-copper"**. 10× más rápido que 100Base-T.

**ATM (Asynchronous Transfer Mode)**: estándar de comunicación por celdas de tamaño fijo para voz/vídeo/datos. Opera en **capa de enlace de datos**. Usado en **redes privadas de larga distancia** (ISPs). Medio: fibra o par trenzado.

**PoE (Power over Ethernet)**: estándares **IEEE 802.3af y 802.3at**. Suministra energía a dispositivos de red a través del cable Ethernet existente. Roles: PSE (transmite energía) y PD (dispositivo alimentado).

---

### Cables 🔴

**Fibra óptica**: núcleo (core, mayor índice de refracción, transporta la señal) + cladding (menor índice) + buffer (protección) + jacket. Características: bajo coste, ancho de banda extremo, ligera, segura, sin interferencia electromagnética, sin cross-talk.

**Cable coaxial**: dos conductores separados por dieléctrico, configuración de cilindros concéntricos. 50 Ω (digital) / 75 Ω (analógico). Data rate: **10 Mbps**.

| Cable | Uso | Ancho de banda | Impedancia |
|---|---|---|---|
| **CAT 3** | Voz + 10BaseT (10 Mbps) | 16 MHz | 100 Ω |
| **CAT 4** | 10BaseT (10 Mbps) | 20 MHz | 100 Ω |
| **CAT 5** | 10/100/1000BASE-T; longitud máx. 100 m | 100 MHz | 100 Ω |
| **CAT 5e** | Fast Eth (100), Gigabit (1000), ATM 155 Mbps | 350 MHz | 100 Ω |
| **CAT 6** | Gigabit (1000) + 10 Gig (10000) | 250 MHz | 100 Ω |

**UTP Ethernet (BaseT)**:

| Estándar | Velocidad | Estándar IEEE | Cables usados | Nº pines |
|---|---|---|---|---|
| 10Base-T | 10 Mbps | 802.3i | CAT 3 o CAT 5 | 4 (pines 1,2,3,6) |
| 100Base-T | 100 Mbps | 802.3u | CAT 5 | 4 (pines 1,2,3,6) |
| 1000Base-T | 1000 Mbps | 802.3ab | CAT 5e | 8 (pines 1-8) |

---

### Protocolos de aplicación: puertos y características 🔴

| Protocolo | Puerto | Transporte | Característica clave |
|---|---|---|---|
| **SMTP** | TCP 25 | TCP | Email saliente; texto plano; solo ASCII 7 bits |
| **SFTP** | TCP 22 | TCP | Extensión SSH2; versión segura de FTP |
| **DHCP** | UDP 67/68 | UDP | Distribuye configuración TCP/IP a clientes |
| **DNS** | UDP/TCP 53 | UDP/TCP | Base de datos jerárquica distribuida; mapea URL → IP |
| **SNMP** | — | UDP | Gestión de redes TCP/IP |
| **RIP** | UDP 520 | UDP | Protocolo de enrutamiento; máx. 15 hops; distancia administrativa 120 |
| **Telnet** | TCP 23 | TCP | Acceso remoto en texto claro |
| **SSH** | TCP 22 | TCP | Acceso remoto cifrado; Linux/UNIX |

**FTP** usa **dos conexiones**:
- **Control**: puerto **21** (cliente → servidor); comandos y respuestas.
- **Datos**: puerto **20** en modo activo.

**Modos FTP**:
- **Activo**: control desde cliente; datos **desde servidor hacia cliente** (puerto 20). El firewall del cliente puede bloquear.
- **Pasivo**: ambas conexiones (control + datos) desde **cliente hacia servidor**. Rango de puertos negociado (ej. 50000-50009).

**TFTP**: protocolo lockstep. Solo lectura/escritura de ficheros remotos; sin listado, borrado, renombrado ni autenticación. Solo en **LAN**. Vulnerable a DoS y directory traversal.

**S-HTTP vs HTTPS**:
- **S-HTTP**: protocolo de capa de aplicación que cifra mensajes HTTP **individuales**. Alternativa a HTTPS. Requiere autenticación del usuario.
- **HTTPS**: cifra la **conexión completa** usando TLS/SSL. Protege contra MITM. Puede ser vulnerable a ataques **DROWN**.

**S/MIME**: firma digital con **RSA**; cifrado de mensajes con **DES**. Para email seguro.

**PGP**: privacidad criptográfica para email; firma digital + cifrado de ficheros almacenados. Usa clave aleatoria para cifrar el fichero, luego cifra esa clave con la clave pública del usuario.

**S/MIME vs PGP** (diferencias clave para el examen):

| | S/MIME v3 | OpenPGP |
|---|---|---|
| Cert. Format | X.509v3 | Basado en PGP anterior |
| Sym. Encryption | Triple DES (CBC) | Triple DES (CFB) |
| Signature Alg. | Diffie-Hellman con DSS o RSA | **ElGamal con DSS** |
| Hash | SHA-1 | SHA-1 |

**SOAP**: protocolo de mensajería basado en **XML** para servicios web. Independiente de plataforma y lenguaje. Equivalente funcional a RPC (como DCOM y CORBA).

**SNMP**: gestión de red TCP/IP en capa de aplicación. Gestiona routers, hubs, módems, impresoras, bridges, switches, servidores, workstations. Riesgos: DDoS y Remote Code Execution.

**NTP**: sincroniza relojes de equipos en red. Usa **UTC** como referencia. Vulnerable a DoS, DDoS amplification, intercepción y replay de paquetes.

**RPC**: permite comunicación inter-procesos sin conocer detalles de red. CVE notable: **CVE-2024-20678**. Puerto vulnerable: **111** (rpcbind).

**SMB**: protocolo de capa de aplicación para acceso compartido a ficheros, impresoras y puertos serie. Usa NetBIOS sobre TCP/IP (NBT). Versión mejorada: **CIFS** (Common Internet File System).

**SIP**: protocolo de señalización y control para sesiones multimedia (voz, vídeo, IM). Trabaja con SDP, RTP, SRTP y TLS. Determina ubicación, disponibilidad, capacidad y gestión de sesión del usuario.

**RADIUS**: AAA centralizado (Authentication, Authorization, Accounting) para servidores de acceso remoto. Flujo: Access-Request → Access-Accept/Reject (+ opcional Access-Challenge) → Accounting-Request → Accounting-Response.

**TACACS+**: AAA para dispositivos de red (switches, routers, firewalls). **Cifra toda la comunicación** incluyendo la contraseña. Modelo cliente-servidor. Vulnerabilidades: sin verificación de integridad, vulnerable a replay, contabilidad en texto claro, cifrado débil.

---

### Protocolos de transporte: TCP vs UDP 🔴

| | **TCP** | **UDP** |
|---|---|---|
| Tipo | Orientado a conexión | Sin conexión |
| Fiabilidad | Sí (ACK, retransmisión, windowing) | No (la app lo gestiona) |
| PDU | Segmentos | Datagramas |
| Protocolo IP nº | **6** | **17** |
| Protocolos que usan | FTP, HTTP, Telnet, SMTP | TFTP, SNMP, DHCP, DNS |

**Cabecera TCP** (20 bytes): Source Port (16b) + Dest Port (16b) + Seq Nº (32b) + ACK Nº (32b) + Header Length (4b) + Reserved (6b) + **flags: URG, ACK, PSH, RST, SYN, FIN** + Window Size (16b) + Checksum (16b) + Urgent Pointer (16b).

**Servicios TCP**: Simplex (una dirección), Half-duplex (dos direcciones, una a la vez), Full-duplex (dos direcciones simultáneas e independientes).

**Cabecera UDP** (8 bytes): Source Port (16b) + Dest Port (16b) + Length (16b) + Checksum (16b). Source Port es **opcional** en UDP.

---

### SSL y TLS

**SSL** (Netscape): protocolo de capa de aplicación para seguridad de transmisión. Requiere transporte fiable (**TCP**). Usa **RSA asimétrico** para cifrar datos.

**TLS**: establece conexión segura cliente-servidor. Usa **clave simétrica** para cifrado masivo, **asimétrica** para autenticación/intercambio de claves, y **MACs** para integridad. Algoritmo RSA con claves de **1024 y 2048 bits**. Protege contra tampering, forgery e intercepción.

---

### IP y ICMP

**IP**: protocolo de capa de red fundamental. Cabecera de **20 bytes** (fija). Campos clave: Version (4b), Header Length (4b), TOS (8b), Total Length (16b), ID (16b), Fragment Offset (13b), **TTL (8b)**, Protocol (8b), Checksum (16b), Src IP (32b), Dst IP (32b).

**IPv4 vs IPv6** 🔴:

| Característica | IPv4 | IPv6 |
|---|---|---|
| Tamaño | **32 bits** | **128 bits** |
| Formato | Decimal punteado | Hexadecimal con dos puntos |
| Total de dir. | 2^32 (~4,3 × 10^9) | 2^128 |
| Desplegado | 1981 | 1999 |
| Checksum en header | Sí | **No** |
| IPsec | Opcional | **Obligatorio** |
| ARP | Broadcast ARP request | **Multicast neighbor solicitation** |
| Autoconfiguración | Manual o DHCP | **Auto-configuración stateless** |
| Broadcast | Sí | **No** (usa multicast all-nodes) |
| Cabecera | 20 bytes | **40 bytes** |

**Mecanismos de transición IPv4 → IPv6**:
- **Dual Stack**: el nodo usa IPv4 o IPv6 según el valor DNS.
- **Tunneling**: encapsula paquetes IPv6 dentro de paquetes IPv4.
- **Translation**: NAT-PT y SIIT permiten que host IPv6 comunique con IPv4.

**ICMP**: aborda la limitación de IP (unreliable). **No corrige** problemas; solo **reporta errores** al origen. Los errores generados por ICMP **no generan nuevos mensajes ICMP**. Los mensajes ICMP se encapsulan en datagramas IP.

Tipos ICMP relevantes para el examen:
- **0**: Echo Reply
- **3**: Destination Unreachable (con 16 códigos de sub-error)
- **8**: Echo (ping)
- **11**: Time Exceeded (usado por traceroute)
- **5**: Redirect
- **30**: Traceroute

**ARP**: protocolo sin estado para resolver IP → MAC. Request = **broadcast**; Reply = **unicast**. El par IP/MAC se almacena en la **ARP cache**. En el paquete ARP: Operation 1 = Request, Operation 2 = Reply. En un ARP Request, la MAC destino es **null** (00:00:00:00:00:00).

---

### Protocolos de enrutamiento 🔴

| Protocolo | Tipo | AD | Hops máx. | Puerto/Protocolo | Actualizaciones |
|---|---|---|---|---|---|
| **RIP** | Distance Vector | 120 | **15** | UDP 520 | Cada **30 seg.** |
| **IGRP** | Distance Vector | 100 | **100** (ext. 255) | IP protocolo 9 | Cada **90 seg.** |
| **EIGRP** | Híbrido (DV+LS) | **90** (interno) / **170** (externo) | 100 (ext. 255) | Multicast 224.0.0.10 | Solo cambios |
| **OSPF** | Link-State | 110 | **Sin límite** | — | Solo cambios |
| **BGP** | Path Vector | — | — | TCP | Rutas entre AS |

**EIGRP** usa **DUAL** (Diffusing Update Algorithm). Es **classless** (soporta VLSMs). RTP para entrega fiable. Soporta IP, IPX y AppleTalk.

**OSPF** mantiene 3 tablas: neighbor table, topology table, routing table. Métrica: **cost**. Solo IP routing.

**HSRP** (Cisco): gateway por defecto tolerante a fallos. Comando de verificación: `show standby`. IP y MAC virtuales compartidas entre dos routers.

**VRRP**: asignación automática de IP routers a hosts participantes. Si el router activo falla, otro lo reemplaza automáticamente.

---

### Direccionamiento IP 🔴

**Clases IPv4**:

| Clase | Primeros bits | Rango decimal | Red/Host | Nº redes | Hosts/red | Uso |
|---|---|---|---|---|---|---|
| A | 0 | 1–126 | 8/24 | 126 | 16.277.214 | Organizaciones muy grandes |
| B | 10 | 128–191 | 16/16 | 16.384 | 65.534 | Organizaciones medianas |
| C | 110 | 192–223 | 24/8 | 2.097.152 | 254 | Organizaciones pequeñas |
| D | 1110 | 224–239 | N/A | N/A | N/A | Multicast |
| E | 1111 | 240–255 | N/A | N/A | N/A | Experimental |

**Máscaras de subred por defecto**:
- Clase A: 255.0.0.0 (/8)
- Clase B: 255.255.0.0 (/16)
- Clase C: 255.255.255.0 (/24)

**VLSM**: permite dos o más máscaras de subred distintas en la **misma red**.

**Subnetting**: toma bits del host ID para extender la máscara natural. Ejemplo: /27 en Clase C (3 bits extra) → **8 subredes posibles**.

**Supernetting (CIDR)**: combina bloques Clase C para crear superredes. Evita el agotamiento de IPs. La supernet mask es la **inversa** de la subnet mask.

**IANA — rangos de puertos**:
- **0–1023**: well-known ports (solo procesos root/privilegiados).
- **1024+**: puertos asignados dinámicamente o registrados para apps de usuario.

**NAT**: permite múltiples dispositivos compartir **una sola IP pública IPv4**. Los números de puerto TCP/UDP permanecen inalterados.

**PAT**: mapea múltiples dispositivos LAN a una sola IP pública usando distintos puertos. También llamado: port overloading, multiplexed NAT, single address NAT.

**VLAN**: redes lógicamente conectadas aunque físicamente dispersas. Configuradas por software, no hardware. Más barata que una red enrutada (switches < routers). Permite separación de tráfico, workgroups virtuales y seguridad.

---

### DHCP y DNS

**DHCP**: distribuye configuración TCP/IP (IP, máscara, gateway, DNS, lease time) mediante lease offer. Flujo: DHCPDISCOVER → DHCPOFFER → DHCPREQUEST → DHCPACK (IPv4); SOLICIT → ADVERTISE → REQUEST → REPLY (IPv6).

**DNS**: base de datos jerárquica distribuida que mapea URLs a IPs. Jerarquía: Root → TLD (.com, .org) → Second-level (domain.com) → Subdomain (about.domain.com) → Host.

**DNSSEC**: suite IETF que añade firmas digitales a registros DNS mediante criptografía de clave pública (asimétrica).
- **Garantiza**: autenticidad, integridad, no existencia de dominio/tipo.
- **No garantiza**: confidencialidad, protección contra DoS.
- El resolver verifica la firma digital; si no coincide con el valor en el registro, rechaza la respuesta.
- **DS Record (Delegation Signing)**: contiene la firma digital de una zona firmada. Hash de la clave pública = **DNSKEY**.
- **RRSIG**: firma del RRSET creada con clave privada.
- **NSEC**: cadena de nombres para probar no existencia.

---

### Protocolos de enlace y capa 2

**FDDI**: estándar óptico para LAN. Transfiere datos a **100 Mbps** hasta **200 km**. Dos anillos de fibra óptica: primary (activo) y secondary (backup). Soporta voz y multimedia (FDDI-2).

**CDP** (Cisco): protocolo propietario de **capa 2** (Data Link). MAC destino: **01.00.0c.cc.cc.cc**. Descubre dispositivos directamente conectados.

**VTP** (Cisco): protocolo de mensajería en **capa 2**. Distribuye configuración VLAN a todos los switches del mismo dominio. Almacena en VLAN database. Plug-and-play para VLANs nuevas. Vulnerabilidades: DoS, integer wrapping, buffer overflow.

**STP**: capa 2; corre en bridges y switches. Previene bucles. Vulnerable a: MITM, DNS Spoofing, DoS, session hijacking.

**PPP**: protocolo de capa de enlace para comunicación punto a punto (sin dispositivos intermedios). Autenticación: **PAP** y **CHAP**. Sin control de flujo propio → puede sobrecargar el receptor.

---

## 2. Exam Traps ⚠️

⚠️ **[OSI: capas superiores vs inferiores]**
Las 4 capas superiores (Application, Presentation, Session, Transport) son las **Host Layers** usadas cuando el mensaje va hacia/desde el usuario. Las 3 inferiores (Network, Data Link, Physical) son **Media Layers** usadas cuando el mensaje transita por el host. El examen puede invertirlos.

⚠️ **[OSI vs TCP/IP: comunicación orientada a conexión]**
OSI soporta **solo** comunicación orientada a conexión. TCP/IP soporta **ambas** (con y sin conexión). Trampa frecuente: presentar TCP/IP como solo orientado a conexión.

⚠️ **[Bluetooth — estándar IEEE]**
Bluetooth es **IEEE 802.15**, no 802.11 (que es Wi-Fi). Opera a 2,4–2,485 GHz y transfiere a **menos de 1 Mbps** hasta **10 metros**.

⚠️ **[802.11e vs 802.11i]**
- **802.11e**: QoS (priorización de datos, voz y vídeo).
- **802.11i**: seguridad/cifrado mejorado para WLANs (para 802.11a/b/g).
El examen puede confundir cuál es el de seguridad y cuál el de QoS.

⚠️ **[Hub vs Switch — visibilidad de paquetes]**
En un hub, **todos los nodos ven todos los paquetes**. En un switch, el paquete llega **solo al nodo destino**. Esta distinción es clásica en preguntas de seguridad (sniffing en hubs).

⚠️ **[10 Gigabit Ethernet — fibra obligatoria]**
Es el único estándar Ethernet de la tabla que usa **exclusivamente fibra óptica**. Los demás pueden usar cobre.

⚠️ **[FTP activo vs pasivo]**
- Activo: datos salen del **servidor** (puerto 20) hacia el cliente. El cliente puede bloquearlo con firewall.
- Pasivo: **ambas** conexiones las inicia el cliente. Útil detrás de NAT/firewall.

⚠️ **[SFTP — puerto y base]**
SFTP usa **puerto 22** y es extensión de **SSH2**, no de FTP estándar. No confundir con FTPS (FTP sobre SSL/TLS).

⚠️ **[S-HTTP vs HTTPS]**
S-HTTP cifra **mensajes individuales** (capa de aplicación). HTTPS cifra la **conexión completa** con TLS/SSL. Son distintos protocolos. No todos los navegadores soportan S-HTTP.

⚠️ **[S/MIME — algoritmos]**
RSA para firma digital; DES para cifrado del mensaje. El examen puede proponer AES o 3DES como respuesta para S/MIME — la respuesta del libro es DES.

⚠️ **[DNSSEC — qué no garantiza]**
DNSSEC **no garantiza** confidencialidad ni protección frente a DoS. Solo garantiza autenticidad e integridad. Trampa: el examen puede incluir "confidencialidad" en la lista de garantías.

⚠️ **[TCP protocolo nº 6 / UDP protocolo nº 17]**
El campo Protocol en la cabecera IP especifica el protocolo de transporte. TCP = **6**, UDP = **17**. Datos que el examen pregunta directamente.

⚠️ **[ICMP — no corrige errores]**
ICMP **reporta** el error al origen, pero **no lo corrige**. La fiabilidad debe ser provista por capas superiores (TCP o aplicación). Los mensajes ICMP no generan más mensajes ICMP.

⚠️ **[ARP Request: MAC destino null]**
En un ARP Request, la MAC destino es **null (00:00:00:00:00:00)**. El examen puede preguntar el valor del campo de hardware destino en un request ARP.

⚠️ **[RIP vs OSPF — hop count]**
RIP tiene máximo **15 hops** (el hop 16 es inaccesible). OSPF **no tiene límite de hops**. IGRP y EIGRP tienen máximo 100 (extensible a 255).

⚠️ **[EIGRP — distancia administrativa]**
EIGRP tiene **AD 90** para rutas internas y **AD 170** para rutas externas. RIP = 120, IGRP = 100, OSPF = 110. El más bajo gana la selección de ruta.

⚠️ **[Clase de IP 127.x.x.x — ausente en la tabla de clases]**
La Clase A va de 1 a **126** en el primer octeto. El rango **127.x.x.x está reservado para loopback** (127.0.0.1). El examen puede incluir 127 en el rango de Clase A — no es válido para hosts.

⚠️ **[Supernetting vs Subnetting]**
- Subnetting: **divide** una red en subredes más pequeñas (toma bits del host).
- Supernetting (CIDR): **combina** bloques Clase C en una superred. La supernet mask es la **inversa** de la subnet mask. Aplica principalmente a **Clase C**.

⚠️ **[TACACS+ vs RADIUS — cifrado]**
TACACS+ cifra **toda la comunicación** (incluida la contraseña). RADIUS cifra **solo la contraseña** en el paquete Access-Request. Esta diferencia es fundamental para preguntas de seguridad.

⚠️ **[NAT vs PAT]**
NAT mapea IPs internas a una IP pública (o pool). PAT (también llamado **port overloading**) mapea múltiples dispositivos a **una sola IP pública** usando diferentes puertos — es un caso especial de NAT.

---

## 3. Nemotécnicos

### OSI — 7 capas (de arriba a abajo)
**"All People Seem To Need Data Processing"**
- Application → Presentation → Session → Transport → Network → Data Link → Physical

### OSI — unidades de datos por capa
- Capas 5-7: **Data**
- Capa 4: **Segments**
- Capa 3: **Packets**
- Capa 2: **Frames**
- Capa 1: **Bits**

### Tipos de red por tamaño (de menor a mayor)
**PAN → LAN → CAN → MAN → WAN → GAN**
- PAN: room-size
- LAN: building
- CAN: campus
- MAN: city
- WAN: countries/continents
- GAN: unlimited (Internet)

### Clases IP — rangos del primer octeto
**"1-126 / 128-191 / 192-223 / 224-239 / 240-255"**
- A: 1–126 (0xxxxxxx)
- B: 128–191 (10xxxxxx)
- C: 192–223 (110xxxxx)
- D: 224–239 (1110xxxx) — Multicast
- E: 240–255 (1111xxxx) — Experimental
- Nota: **127** = loopback, no es clase válida para hosts.

### Distancias administrativas (de menor a mayor = más confiable)
**"90 - 100 - 110 - 120"**
- EIGRP interno: **90**
- IGRP: **100**
- OSPF: **110**
- RIP: **120**
- EIGRP externo: **170**

### Wi-Fi: las que tienen velocidades concretas para memorizar
- **802.11b**: 2,4 GHz, **11 Mbps** (DSSS)
- **802.11a/g**: 5/2,4 GHz, **54 Mbps** (OFDM)
- **802.11n**: **150 Mbps** (MIMO-OFDM)
- **802.11ax**: **2400 Mbps** (1024-QAM)
- **802.11be**: **3000 Mbps** (QAM)

### Generaciones móviles — velocidades de descarga
**"114K → 384K → 7.2M → 337M → 100M/1G"**
- 2.5G (GPRS): 114 Kbps
- 2.75G (EDGE): 384 Kbps
- 3G (HSPA): 7,2 Mbps
- 3.5G (HSPA+): 337 Mbps
- 4G (LTE): 100 Mbps (movilidad alta) / 1 Gbps (movilidad baja)

---

## 4. Flashcards

**Q:** ¿Cuántas capas tiene el modelo OSI y cuáles son las "Host Layers"?
**A:** 7 capas. Host Layers: Application (7), Presentation (6), Session (5), Transport (4).

---

**Q:** ¿Qué PDU (unidad de datos) corresponde a la capa Transport del modelo OSI?
**A:** Segments.

---

**Q:** ¿Qué diferencia fundamental existe entre OSI y TCP/IP respecto a los tipos de comunicación soportados?
**A:** OSI soporta solo comunicación orientada a conexión. TCP/IP soporta ambas: orientada a conexión y sin conexión.

---

**Q:** ¿Bajo qué estándar IEEE opera Bluetooth y cuáles son su frecuencia y alcance máximo?
**A:** IEEE 802.15. Frecuencia: 2,4–2,485 GHz. Alcance: hasta 10 metros. Velocidad: < 1 Mbps.

---

**Q:** ¿Cuál es la velocidad de descarga de 4G LTE en comunicación de alta movilidad vs baja movilidad?
**A:** Alta movilidad: 100 Mbit/s. Baja movilidad: 1 Gbit/s.

---

**Q:** ¿Qué estándar IEEE y velocidad tiene Gigabit Ethernet?
**A:** IEEE 802.3z (o 802.3-2008). 1000 Mbps. También llamado 1000Base-T.

---

**Q:** ¿Cuál es el único estándar Ethernet de la tabla que usa exclusivamente fibra óptica?
**A:** 10 Gigabit Ethernet (IEEE 802.3ae-2002).

---

**Q:** ¿Qué cable UTP se usa con 1000Base-T y cuántos pines utiliza?
**A:** CAT 5e. 8 pines (1-8).

---

**Q:** ¿En qué modos opera FTP y cuál es la diferencia en el origen de la conexión de datos?
**A:** Modo activo: la conexión de datos la establece el servidor (puerto 20) hacia el cliente. Modo pasivo: el cliente establece ambas conexiones (control y datos) hacia el servidor.

---

**Q:** ¿En qué versión de SSH2 se basa SFTP y en qué puerto opera?
**A:** Es extensión de SSH2. Opera en TCP puerto 22.

---

**Q:** ¿Qué garantiza DNSSEC y qué no garantiza?
**A:** Garantiza: autenticidad, integridad, no existencia de dominio/tipo. No garantiza: confidencialidad ni protección contra DoS.

---

**Q:** ¿Cuál es el número de protocolo IP para TCP y para UDP?
**A:** TCP: protocolo número 6. UDP: protocolo número 17.

---

**Q:** ¿Cuáles son los flags de la cabecera TCP?
**A:** URG, ACK, PSH, RST, SYN, FIN.

---

**Q:** ¿Qué diferencia hay entre un hub y un switch en cuanto a visibilidad de paquetes?
**A:** En un hub, todos los nodos ven todos los paquetes. En un switch, el paquete llega solo al nodo destino.

---

**Q:** ¿Cuál es el hop count máximo de RIP y cuál es su distancia administrativa?
**A:** Máximo 15 hops. Distancia administrativa: 120.

---

**Q:** ¿Cuál es la distancia administrativa de EIGRP para rutas internas y para rutas externas?
**A:** Internas: 90. Externas: 170.

---

**Q:** ¿OSPF tiene límite de hop count?
**A:** No. OSPF no tiene límite de hops. Su métrica es el coste.

---

**Q:** ¿Qué algoritmo usa EIGRP para determinar la mejor ruta libre de bucles?
**A:** DUAL (Diffusing Update Algorithm).

---

**Q:** ¿Cuál es la diferencia principal entre RADIUS y TACACS+ en cuanto a cifrado?
**A:** RADIUS cifra solo la contraseña. TACACS+ cifra toda la comunicación (incluida la contraseña).

---

**Q:** ¿Qué tipo de mensaje usa ARP para solicitar una MAC y qué tipo usa para responder?
**A:** Solicitud (Request): broadcast. Respuesta (Reply): unicast.

---

**Q:** ¿Cuál es el rango de puertos well-known según IANA?
**A:** 0–1023.

---

**Q:** ¿Qué algoritmos de firma y cifrado usa S/MIME?
**A:** Firma digital: RSA. Cifrado del mensaje: DES.

---

**Q:** ¿Cuáles son los 3 mecanismos de transición de IPv4 a IPv6?
**A:** Dual Stack, Tunneling, Translation (NAT-PT / SIIT).

---

**Q:** ¿Cuál es la diferencia entre NAT y PAT?
**A:** NAT mapea IPs internas a una IP pública. PAT (port overloading) mapea múltiples dispositivos a una sola IP pública usando distintos números de puerto.

---

## 5. Confusión frecuente

### Hub vs Switch
- **Hub**: dispositivo de capa 1. Todos los nodos del segmento LAN ven **todos** los paquetes → vulnerable a sniffing.
- **Switch**: similar al hub pero los paquetes llegan **solo al nodo destino** → más seguro y eficiente.
- **Criterio de decisión**: "todos ven los paquetes" → Hub. "solo el destino recibe" → Switch.

---

### TCP vs UDP — cuándo usa cada uno cada protocolo
- **TCP**: FTP, HTTP, Telnet, SMTP, SSH, SFTP, BGP. Cuando se necesita fiabilidad y orden.
- **UDP**: TFTP, SNMP, DHCP, DNS (consultas). Cuando la velocidad prima sobre la fiabilidad.
- **Criterio de decisión**: si el protocolo puede permitirse perder datos (DNS, DHCP, TFTP) → UDP. Si necesita integridad (transferencia de ficheros, web, email) → TCP.

---

### S-HTTP vs HTTPS
- **S-HTTP**: cifra **mensajes individuales** HTTP. Protocolo de capa de aplicación. Alternativa a HTTPS. Requiere autenticación del usuario. No todos los navegadores lo soportan.
- **HTTPS**: cifra la **conexión completa** usando TLS/SSL. El estándar de facto en la web.
- **Criterio de decisión**: "cifrado mensaje a mensaje" o "requiere autenticación de usuario" → S-HTTP. "conexión cifrada completa", "man-in-the-middle", "transacciones confidenciales" → HTTPS.

---

### RIP vs OSPF vs EIGRP — tipo y limitaciones
- **RIP**: Distance Vector, máx. 15 hops, UDP 520, updates cada 30 seg. Para redes pequeñas.
- **OSPF**: Link-State, sin límite de hops, métrica = coste. Para grandes AS.
- **EIGRP**: Híbrido (DV+LS), DUAL, classless, soporta VLSMs. Solo en redes Cisco.
- **Criterio de decisión**: "red pequeña" → RIP. "sin límite de hops" → OSPF. "híbrido con DUAL" → EIGRP.

---

### RADIUS vs TACACS+
- **RADIUS**: AAA para acceso remoto. Cifra solo contraseña. Combina autenticación y autorización en un solo proceso.
- **TACACS+**: AAA para dispositivos de red. Cifra **todo**. Separa autenticación, autorización y accounting en procesos distintos.
- **Criterio de decisión**: "solo contraseña cifrada" o "dial-in users" → RADIUS. "todo cifrado" o "switches/routers/firewalls" → TACACS+.

---

### Subnetting vs Supernetting (CIDR)
- **Subnetting**: divide una red Clase A/B/C en subredes más pequeñas. Toma bits del host ID. La máscara se **extiende**.
- **Supernetting**: combina bloques de Clase C en una red mayor. La supernet mask es la **inversa** de la subnet mask. También llamado CIDR.
- **Criterio de decisión**: "dividir" o "subredes" → Subnetting. "combinar Clase C", "CIDR", "evitar agotamiento IP" → Supernetting.

---

### NAT vs PAT
- **NAT**: traduce IPs internas a una IP pública (o pool de IPs). La traducción es de dirección IP.
- **PAT** (port overloading): un único caso de NAT donde múltiples IPs internas comparten **una sola IP pública** usando distintos puertos. También llamado: port overloading, multiplexed NAT, single address NAT.
- **Criterio de decisión**: "múltiples dispositivos con una sola IP pública" o "port overloading" → PAT. Traducción IP genérica → NAT.
