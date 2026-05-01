# M08 — Sniffing

---

## 1. Conceptos Fundamentales de Sniffing

### ¿Qué es el Packet Sniffing?
Proceso de monitorizar y capturar todos los paquetes de datos que pasan por una red usando un software o dispositivo hardware. El sniffer opera en **modo promiscuo**: desactiva el filtro de la NIC Ethernet que normalmente descarta el tráfico no dirigido al host, permitiendo capturar todo el tráfico del segmento.

**Información capturada:** contraseñas en texto plano, emails, números de tarjetas de crédito, transacciones financieras, tráfico SMTP/POP/IMAP, HTTP Basic, Telnet, SQL, SMB, NFS, FTP, syslog, router configuration, DNS traffic.

### Hub vs Switch: diferencia clave
- **Hub:** transmite datos a todos los puertos (sin mapeo de línea). Todo el tráfico del segmento llega a todos los hosts. El sniffing en hub es **pasivo y difícil de detectar**.
- **Switch:** mantiene una tabla CAM (Content Addressable Memory) que mapea MACs a puertos físicos y envía datos solo al puerto correcto. El sniffing en switch requiere técnicas activas.

### Sniffing Pasivo vs Activo 🔴

**Pasivo:** solo captura y monitoriza sin enviar paquetes. Funciona en redes con hubs (common collision domain). No transmite datos → difícil de detectar.

**Activo:** inyecta paquetes en la red para manipular switches. Técnicas de active sniffing:
- MAC flooding
- DNS poisoning
- ARP poisoning
- DHCP attacks
- Switch port stealing
- Spoofing attack

### Protocolos vulnerables a sniffing
Todos transmiten en texto plano: **Telnet** (puerto 23), **rlogin**, **HTTP**, **SNMPv1/v2**, **SMTP**, **NNTP**, **POP**, **FTP**, **IMAP**, **TFTP** (sin autenticación ni cifrado).

### Sniffing en el Modelo OSI
Los sniffers operan en la **capa 2 (Data Link)**. Las capas superiores del modelo OSI no son conscientes del sniffing porque las capas están diseñadas para trabajar independientemente. El Data Link layer usa la dirección MAC del destino (no IP) en la cabecera Ethernet.

---

## 2. Estructura de Direcciones MAC y CAM

### MAC Address
48 bits divididos en dos secciones de 24 bits:
- **Primeros 24 bits (6 dígitos hex):** OUI (Organizationally Unique Identifier) = identifica al fabricante.
- **Siguientes 24 bits (6 dígitos hex):** NIC specific = número de serie del adaptador.

Ejemplo: `D4-BE-D9-14-C8-29` → `D4BED9` = Dell, Inc.; `14C829` = serial del adaptador.

### CAM Table (Content Addressable Memory)
Tabla dinámica de tamaño fijo que almacena MACs disponibles en puertos físicos junto con parámetros VLAN. Columnas: VLAN, MAC Address, Type (Dynamic), Learn (Yes), Age, Ports.

**Limitación crítica:** tamaño fijo. Si se llena, el switch entra en **fail-open mode** y empieza a funcionar como un hub, broadcasting todo el tráfico a todos los puertos.

---

## 3. Técnica: MAC Attacks

### MAC Flooding
El atacante bombarda el switch con miles de MACs falsas hasta llenar la tabla CAM. Al desbordarse, el switch entra en **fail-open mode** (actúa como hub). El atacante pone su NIC en modo promiscuo y puede ver todo el tráfico.

**Herramienta: macof** (parte de la suite dsniff, Unix/Linux). Inunda la red con MACs e IPs aleatorias. Velocidad: **131,000 entradas por minuto**. Cuando la tabla CAM se llena, el switch convierte su operación a modo hub.

### Switch Port Stealing
El atacante spoofea la MAC del host objetivo como fuente en paquetes ARP gratuitos (gratuitous ARP) y los inyecta desde su propio puerto. Crea una **race condition** entre el tráfico legítimo del host y los paquetes falsificados del atacante. Si el atacante envía paquetes más frecuentemente que el host legítimo, el switch asocia la MAC del objetivo con el puerto del atacante. El atacante puede lanzar un DoS contra el host objetivo para ralentizar sus respuestas y ganar la race condition.

### Defensa contra MAC Attacks: Port Security
**Port Security** identifica y limita las MACs que pueden acceder al puerto. Si se supera el máximo de MACs seguras o se detecta una MAC no autorizada → violación de seguridad.

Comandos Cisco para configurar Port Security:
```
switchport mode access
switchport port-security
switchport port-security maximum <1-3072>     # Default: 1
switchport port-security violation {restrict | shutdown}
switchport port-security mac-address sticky   # Aprende la primera MAC automáticamente
switchport port-security aging time 2         # Aging time en minutos
switchport port-security aging type inactivity
snmp-server enable traps port-security trap-rate 5  # Controla la tasa de traps SNMP
```

**Efecto:** limita ataques de MAC flooding. Cuando se detecta flooding, bloquea el puerto y envía un **SNMP trap**.

---

## 4. Técnica: DHCP Attacks

### ¿Cómo funciona DHCP?
DHCP es un protocolo cliente-servidor que asigna IPs, gateway por defecto y máscara de subred. Flujo:
1. Cliente broadcast **DHCPDISCOVER**.
2. DHCP-relay agent captura la petición y la unicasta a los servidores DHCP.
3. Servidor unicasta **DHCPOFFER** con MACs del cliente y servidor.
4. Relay agent broadcast el DHCPOFFER en la subred del cliente.
5. Cliente broadcast **DHCPREQUEST** solicitando configuración.
6. Servidor unicasta **DHCPACK** con la configuración IP.

Mensajes DHCPv4 → DHCPv6 equivalentes:
- DHCPDiscover → Solicit
- DHCPOffer → Advertise
- DHCPRequest → Request/Confirm/Renew/Rebind
- DHCPAck → Reply
- DHCPRelease → Release
- DHCPDecline → Decline

### DHCP Starvation Attack
El atacante inunda el servidor DHCP con peticiones DHCP usando **MACs falsas (spoofed)**, agotando el pool de IPs disponibles → ataque DoS. Los usuarios legítimos no pueden obtener ni renovar IPs.

**Herramientas:** Yersinia, Hyenae, Gobbler, dhcpStarvation.py, DHCPig.

### Rogue DHCP Server Attack
Tras agotar el pool del servidor DHCP legítimo, el atacante despliega un **servidor DHCP rogue** que responde a las peticiones DHCP de los clientes antes que el servidor legítimo. El rogue server asigna como default gateway la IP del atacante → todo el tráfico del cliente pasa por el atacante → MITM.

El cliente acepta la primera respuesta que llega. Si el rogue server responde antes, el cliente usa su configuración.

### Defensa contra DHCP Starvation: Port Security
Limitar el número máximo de MACs en el puerto del switch (`switchport port-security maximum 1`).

### Defensa contra Rogue DHCP Server: DHCP Snooping
**DHCP Snooping** es una característica del switch que no permite que otros puertos respondan a paquetes DHCP Discover de clientes. Solo el puerto marcado como **trusted** (donde está el servidor DHCP legítimo) puede enviar respuestas DHCP.

**Por defecto, todos los puertos de la VLAN son untrusted.**

Comandos Cisco para DHCP Snooping:
```
ip dhcp snooping                           # Habilitar globalmente
ip dhcp snooping vlan 4,104               # Habilitar en VLANs específicas
ip dhcp snooping trust                     # Marcar interfaz como trusted
ip dhcp snooping limit rate <pps>         # Limitar paquetes DHCP por segundo
show ip dhcp snooping                      # Ver estado
show ip dhcp snooping binding             # Ver tabla de bindings (MAC, IP, VLAN, interfaz)
no ip dhcp snooping information option   # Deshabilitar inserción del campo option-82
```

---

## 5. Técnica: ARP Poisoning

### ¿Qué es ARP?
Protocolo TCP/IP **sin estado (stateless)** que mapea IPs a MACs. ARP es **stateless** → una máquina puede enviar un ARP reply sin que nadie lo haya solicitado, y los sistemas lo aceptan. Esta es la vulnerabilidad fundamental que permite ARP spoofing.

Proceso ARP: el host origen genera ARP request (con su MAC, su IP y la IP destino) → el switch hace broadcast → el host destino responde con su MAC → el origen actualiza su tabla ARP.

### ARP Spoofing / ARP Poisoning
El atacante envía gran cantidad de **ARP replies falsos** con IPs y MACs falsas. La víctima acepta ciegamente el ARP reply y actualiza su tabla ARP con la entrada envenenada. Una vez la tabla ARP está envenenada, el switch entra en modo forwarding y el atacante intercepta todo el tráfico.

**Herramienta: arpspoof** (`arpspoof –i [Interface] –t [Target Host]`): reemplaza el MAC address en la tabla ARP de la víctima con el del atacante. El tráfico de la víctima al gateway se redirige al atacante.

### Amenazas del ARP Poisoning 🔴
- **Packet Sniffing:** sniffea el tráfico de la red.
- **Session Hijacking:** roba información de sesión válida.
- **VoIP Call Tapping:** usa port mirroring para monitorizar llamadas VoIP.
- **Manipulating Data:** captura y modifica datos o detiene el flujo de tráfico.
- **Man-in-the-Middle:** el atacante se sitúa entre víctima y servidor.
- **Data Interception:** intercepta IPs, MACs y VLANs conectadas al switch.
- **Connection Hijacking:** manipula la conexión del cliente para tomar control completo.
- **Connection Resetting:** transmisión de información de routing incorrecta.
- **Stealing Passwords:** engaña a los hosts para que envíen credenciales.
- **DoS Attack:** asocia múltiples IPs con una sola MAC del host objetivo, sobrecargándolo.

### Defensa contra ARP Poisoning: DAI (Dynamic ARP Inspection)
**DAI** valida paquetes ARP usando la **DHCP snooping binding table** (que contiene MACs, IPs, VLANs e interfaces obtenidos escuchando exchanges DHCP). Por defecto, cuando DAI está activo en una VLAN, **todos los puertos son untrusted**.

**Requisito:** DHCP snooping debe estar habilitado ANTES de habilitar DAI. Si los hosts tienen IPs estáticas (no DHCP), se debe usar **static mapping** (asociación IP-MAC manual).

Comandos Cisco para DAI:
```
ip arp inspection vlan 10                    # Habilitar DAI en VLAN
ip arp inspection vlan 10,11,12             # Múltiples VLANs
ip arp inspection vlan 10-13               # Rango de VLANs
show ip arp inspection                      # Ver estado de ARP inspection
ip arp inspection validate                  # Habilitar validaciones adicionales
```

---

## 6. Técnica: Spoofing Attacks

### MAC Spoofing / Duplicating
El atacante sniffea la red para obtener MACs de clientes legítimos asociados al switch port y luego **duplica (spoofea) esa MAC**. Si tiene éxito, recibe todo el tráfico destinado al cliente legítimo.

**Nota:** esta técnica se puede usar para **bypassar el filtrado MAC de puntos de acceso wireless**.

Métodos de MAC spoofing en Windows 11:
- **Método 1:** Control Panel → Network → Ethernet Properties → Configure → Advanced → Network Address → introducir nueva MAC (sin ':').
- **Método 2:** Regedit → `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-...}` → añadir valor "NetworkAddress" (REG_SZ) con la nueva MAC.

### IRDP Spoofing
IRDP (ICMP Router Discovery Protocol) permite a un host descubrir IPs de routers activos escuchando mensajes de advertisement. **No requiere autenticación** → el atacante puede añadir entradas de ruta por defecto remotamente spoofando mensajes de router advertisement.

El atacante establece alto nivel de preferencia y lifetime de la ruta para que los hosts la elijan como ruta preferida. **Requisito:** el atacante debe estar en la misma red que la víctima.

Ataques posibles via IRDP: passive sniffing, MITM, DoS (añadir entradas de ruta incorrectas).

**Defensa:** deshabilitar IRDP en los hosts si el OS lo permite.

### VLAN Hopping
Permite acceder al tráfico de VLANs que deberían ser inaccesibles. Dos métodos:

**Switch Spoofing:** el atacante conecta un switch rogue que engaña al switch legítimo para crear un **trunk link** entre ellos. El trunk permite recibir tráfico de múltiples VLANs. Solo funciona si el switch legítimo está configurado para negociar trunk (`dynamic auto`, `dynamic desirable` o `trunk` mode).

**Defensa contra Switch Spoofing:**
```
switchport mode access         # Configurar puertos explícitamente como access
switchport mode nonegotiate   # Asegurar que no negocie trunks
```

**Double Tagging:** el atacante añade dos etiquetas 802.1Q al frame Ethernet. La etiqueta exterior = native VLAN del atacante. La etiqueta interior = VLAN objetivo. El switch strips la etiqueta exterior (native VLAN) y reenvía el frame con la etiqueta interior a todas las interfaces trunk. Solo funciona si los puertos usan **native VLANs**.

**Defensa contra Double Tagging:**
```
switchport access vlan 2          # Especificar VLAN por defecto (no native)
switchport trunk native vlan 999  # Cambiar native VLANs a un ID no usado
vlan dot1q tag native             # Etiquetar explícitamente native VLANs
```

### STP Attack
En un ataque STP (Spanning Tree Protocol), el atacante conecta un **switch rogue con prioridad menor** que cualquier otro switch de la red. Esto hace que el switch rogue sea elegido como **root bridge**. El root bridge recibe todo el tráfico de la red, permitiendo al atacante sniffearlo todo.

Los BPDUs (Bridge Protocol Data Units) contienen un BID = Bridge Priority + MAC Address. **Valor por defecto del Bridge Priority: 32769**.

**Defensas contra STP Attacks:**

**BPDU Guard:** habilitar en puertos que nunca deben recibir BPDUs. Si se recibe un BPDU, el puerto se pone en **errdisable mode** (shut down).
```
spanning-tree portfast bpduguard
```

**Root Guard:** protege el root bridge. Si un puerto con root guard recibe un BPDU superior, lo convierte en **loop inconsistent state** (no errdisable).
```
spanning-tree guard root
```

**Loop Guard:** mejora la estabilidad previniendo bridging loops por switches defectuosos.
```
spanning-tree guard loop
```

**UDLD (Unidirectional Link Detection):** detecta enlaces unidireccionales que pueden causar loops STP.
```
udld { enable | disable | aggressive }
```

---

## 7. Técnica: DNS Poisoning

### ¿Qué es DNS Poisoning?
El atacante manipula las entradas de la tabla DNS para que las peticiones DNS resuelvan a IPs falsas (del servidor del atacante). También llamado **DNS Spoofing**.

### Técnicas de DNS Poisoning

**Intranet DNS Spoofing:** el atacante debe estar conectado a la LAN y poder sniffear tráfico. Usa ARP poisoning (`arpspoof`/`dnsspoof`) para redirigir peticiones DNS al atacante. Cuando el cliente envía una petición DNS, el router (envenenado) la envía al atacante, que responde con una IP falsa antes que el servidor DNS real.

**Internet DNS Spoofing (Remote DNS Poisoning):** el atacante configura un servidor DNS rogue con IP estática. Usa un Trojan para cambiar la entrada DNS primaria de la víctima por la IP del atacante. El tráfico de la víctima se redirige al sistema del atacante. Puede afectar a víctimas en cualquier parte del mundo.

**Proxy Server DNS Poisoning:** el atacante configura un proxy server con un DNS fraudulento como entrada DNS primaria. Un Trojan cambia la configuración del proxy server de la víctima (en Internet Explorer). El proxy redirige al sitio falso del atacante.

**DNS Cache Poisoning:** el atacante modifica o añade entradas falsas en la **caché del DNS resolver**. El resolver usa la caché para responder peticiones rápidamente sin consultar el servidor autoritativo. Si la caché está envenenada, todas las consultas a ese dominio se redirigen al sitio malicioso.

**SAD DNS (Side-channel Attack on DNS):** variante moderna de DNS cache poisoning que explota side channels y vulnerabilidades en implementaciones DNS (dnsmasq, unbound, BIND) para inyectar registros DNS maliciosos.

### Defensa contra DNS Poisoning 🔴
- Implementar **DNSSEC** (Domain Name System Security Extensions).
- Usar **SSL** para asegurar el tráfico.
- Configurar el DNS resolver para usar un **nuevo source port aleatorio** en cada query saliente.
- **No permitir tráfico saliente en UDP puerto 53 como source port por defecto**.
- Restringir **DNS zone transfers** a un conjunto limitado de IPs.
- Usar **DNS-over-HTTPS (DoH)** o **DNS-over-TLS (DoT)** para cifrar queries DNS.
- Usar **DNS Cookie RFC 7873** o desactivar paquetes ICMP salientes para prevenir SAD DNS attacks.
- Usar **0x20 encoding** y DNS cookies como seguridad adicional.
- **Aleatorizar source y destination IPs, query IDs y capitalización en name requests**.
- Resolver todas las queries DNS a un servidor DNS local. Bloquear peticiones DNS a servidores externos.
- Restringir el servicio DNS recursivo a usuarios autorizados.
- Usar **RNDC keys** si las respuestas deben hacerse por el puerto 53.
- Asegurar que el software DNS use **generación de números aleatorios segura** para transaction IDs.

---

## 8. SPAN Port, Wiretapping y Lawful Interception

### SPAN Port (Switched Port Analyzer)
Característica Cisco (también conocida como **port mirroring**) que envía una copia de cada paquete que pasa por el switch a un puerto destino específico para análisis. Puede tener uno o más puertos fuente (incluyendo toda una VLAN) pero **solo un puerto destino**. Útil para monitorizar, depurar y detectar accesos no autorizados.

### Wiretapping
Monitorización de conversaciones telefónicas o de Internet por un tercero. Dos tipos:
- **Active Wiretapping:** permite monitorizar, registrar Y **alterar/inyectar datos** en la comunicación (MITM).
- **Passive Wiretapping:** solo monitoriza y registra tráfico (snooping/eavesdropping).

El wiretapping sin orden judicial o consentimiento es un delito criminal en la mayoría de países.

### Lawful Interception (LI)
Interceptación legalmente sancionada de comunicaciones por operadores de red o ISPs para vigilancia. Llevada a cabo por agencias de cumplimiento de la ley (LEAs). Solo para monitorizar comunicaciones en canales sospechosos.

---

## 9. Herramientas de Sniffing

### Wireshark
Herramienta principal de análisis de tráfico de red. Captura tráfico en tiempo real de Ethernet, IEEE 802.11, PPP/HDLC, ATM, Bluetooth, USB, Token Ring, Frame Relay, FDDI.

**Follow TCP Stream:** muestra datos TCP de la misma forma que la capa de aplicación. Útil para encontrar contraseñas en sesiones Telnet.

**Display Filters clave para el examen:**
```
arp, http, tcp, udp, dns, ip      # Filtrar por protocolo
tcp.port==23                       # Filtrar por puerto específico
ip.addr==192.168.1.100            # Filtrar por IP
ip.addr == 10.0.0.4 or ip.addr == 10.0.0.5  # Múltiples IPs
tcp.flags.reset==1                 # Todos los TCP resets
http.request                       # Todos los HTTP GET requests
tcp.analysis.retransmission       # Todas las retransmisiones
!(arp or icmp or dns)             # Enmascarar protocolos no deseados
tcp.port == 4000                   # Paquetes TCP con puerto 4000 (src o dst)
tcp.port eq 25 or icmp            # SMTP e ICMP
ip.src==192.168.0.0/16 and ip.dst==192.168.0.0/16  # Solo tráfico LAN
```

### Hardware Protocol Analyzers
Dispositivos que interpretan el tráfico de red sin alterar el segmento. Ventajas sobre software analyzers: capturan más datos sin pérdida de paquetes en sobrecarga, opciones de conexión LAN/WAN/wireless, timestamps precisos, visualizan bus states y eventos de bajo nivel. **Desventaja:** más costosos.

Ejemplos: Xgig 1000 (VIAVI), SierraNet M1288 (Teledyne LeCroy).

### Otras herramientas de sniffing
- **macof** (dsniff suite): MAC flooding, 131,000 entradas/minuto.
- **arpspoof:** ARP poisoning y sniffing.
- **Habu:** toolkit que incluye ARP poisoning, DHCP starvation, análisis TCP.
- **Yersinia:** DHCP starvation attacks.
- **Capsa Portable Network Analyzer:** análisis y diagnóstico de red, detección de ARP poisoning/flooding.
- **OmniPeek:** análisis de red en tiempo real con Google Maps plugin.
- **DerpNSpoof:** DNS poisoning tool.
- **Ettercap:** multi-purpose: DHCP attacks, ARP spoofing, DNS poisoning.

---

## 10. Contramedidas Generales contra Sniffing

- Usar **SSH** en lugar de Telnet; **SCP** en lugar de FTP; **SSL** para email.
- Usar **HTTPS** en lugar de HTTP.
- Usar **SFTP** en lugar de FTP.
- Usar **SNMPv3** en lugar de SNMPv1/v2.
- Usar **switch** en lugar de hub.
- Añadir permanentemente la **MAC del gateway a la caché ARP**.
- Usar **IPs estáticas y tablas ARP estáticas**.
- Usar **IPv6** en lugar de IPv4 (IPsec es opcional en IPv4, **obligatorio en IPv6**).
- Recuperar la MAC address **directamente desde la NIC**, no desde el OS.
- Usar **PGP, S/MIME, VPN, IPSec, SSL/TLS, SSH, OTPs**.
- Usar **WPA2 o WPA3** para el tráfico wireless.
- Usar **POP2 o POP3** en lugar de POP para descargar emails.
- Restringir el acceso físico a los medios de red.
- Usar **ACLs** para permitir acceso solo a IPs confiables.
- Usar **IDS/IPS** para detectar actividades de sniffing.
- Usar **VLANs y segmentación de red** para limitar el alcance de los sniffers.

---

## 11. Detección de Sniffers

Los sniffers operan en **modo promiscuo** y no transmiten datos → difíciles de detectar. Para detectarlos, buscar sistemas en modo promiscuo.

### Métodos de Detección

**Ping Method:** enviar un ping request a la máquina sospechosa con su **IP correcta pero MAC incorrecta**. La NIC normal lo rechaza por la MAC incorrecta. Un sistema con sniffer **responde** porque no rechaza paquetes con MAC diferente. Esta respuesta identifica el sniffer.

**DNS Method (Reverse DNS Lookup):** los sniffers usando reverse DNS lookup aumentan el tráfico de red. Este aumento puede indicar la presencia de un sniffer. Enviar una petición ICMP a una IP **inexistente**: si un sistema responde, está realizando reverse DNS lookups y probablemente ejecuta un sniffer.

**ARP Method:** enviar un **ARP no-broadcast** a todos los nodos de la red. El nodo en modo promiscuo cachea la dirección ARP local. Luego enviar un ping broadcast con la IP local pero diferente MAC. Solo el nodo que tiene la MAC cacheada puede responder. Una máquina en modo promiscuo responde al ping porque tiene la información correcta en su caché.

### Herramientas de Detección de Modo Promiscuo

**Nmap:**
```bash
nmap --script=sniffer-detect [Target IP/Range]
```

**NetScanTools Pro:** incluye Promiscuous Mode Scanner para escanear la subred con paquetes ARP modificados e identificar dispositivos respondiendo a cada tipo de paquete ARP.

---

## 3. Exam Traps ⚠️

⚠️ **[Sniffing pasivo vs activo: qué técnicas son activas]** — El sniffing pasivo NO envía paquetes (solo captura). El activo SÍ envía paquetes. Técnicas activas: MAC flooding, DNS poisoning, ARP poisoning, DHCP attacks, switch port stealing, spoofing attack. El examen puede preguntar qué técnicas son pasivas o activas.

⚠️ **[macof: velocidad exacta]** — macof inunda el switch a **131,000 entradas por minuto**. Dato específico que puede aparecer en el examen.

⚠️ **[DHCP Snooping: todos los puertos son untrusted por defecto]** — Con DHCP snooping habilitado, TODOS los puertos de la VLAN son **untrusted por defecto**. Solo el puerto del servidor DHCP legítimo se configura como trusted. El examen puede invertir esto.

⚠️ **[DAI requiere DHCP Snooping previo]** — DAI (Dynamic ARP Inspection) **requiere que DHCP Snooping esté habilitado antes**. Si se habilita DAI sin DHCP Snooping en una red con IPs estáticas, puede ocurrir un auto-imposed DoS en la VLAN.

⚠️ **[ARP es stateless]** — ARP es **stateless**: una máquina puede enviar un ARP reply sin haber recibido una petición, y otros sistemas lo aceptan. Esta es la vulnerabilidad fundamental que permite ARP spoofing. El examen puede preguntar por qué ARP es vulnerable.

⚠️ **[STP: Bridge Priority por defecto = 32769]** — El valor por defecto del Bridge Priority es **32769**. Para hacerse root bridge, el atacante introduce un switch con prioridad MENOR (ej. 0 o 1). El examen puede preguntar qué hace que un switch sea elegido root bridge.

⚠️ **[BPDU Guard: errdisable vs Root Guard: loop inconsistent]** — BPDU Guard pone el puerto en **errdisable mode** (lo desactiva). Root Guard lo convierte en **loop inconsistent state** (no errdisable). El examen puede confundir los estados.

⚠️ **[Double Tagging: solo en native VLANs]** — El ataque de double tagging solo es posible si los puertos del switch están configurados para usar **native VLANs**. La defensa es cambiar la native VLAN a un ID no usado (ej. 999).

⚠️ **[DNS Poisoning: UDP puerto 53 como source port]** — La defensa específica: "Do not allow outgoing traffic to use **UDP port 53** as a default source port". Esto es contra-intuitivo pero es una medida específica de defensa contra DNS poisoning.

⚠️ **[SAD DNS: DNS Cookie RFC 7873]** — La defensa específica contra SAD DNS attacks es usar **DNS Cookie RFC 7873** o desactivar paquetes ICMP salientes, y reducir el timeout period para queries pendientes.

⚠️ **[IPv6 vs IPv4: IPsec]** — IPsec es **opcional en IPv4** pero **obligatorio en IPv6**. El examen puede preguntar por qué se recomienda IPv6 como contramedida contra sniffing.

⚠️ **[Ping Method: IP correcta, MAC incorrecta]** — Para detectar un sniffer con el método Ping, se envía el ping con la **IP correcta pero MAC incorrecta**. Un sistema normal rechaza el paquete (MAC no coincide). Un sistema con sniffer responde. El examen puede invertir cuál rechaza y cuál responde.

⚠️ **[Switch Port Stealing vs MAC Flooding]** — MAC Flooding: inunda la CAM table con MACs falsas para que el switch actúe como hub. Switch Port Stealing: spoofea la MAC del objetivo en gratuitous ARP para asociar esa MAC con el puerto del atacante. Son ataques distintos contra el switch.

⚠️ **[SPAN port: un solo destino]** — Un SPAN port puede tener uno o más puertos fuente pero **solo un puerto destino**. El examen puede presentar múltiples destinos como opción válida.

---

## 4. Nemotécnicos

**6 técnicas de active sniffing:**
> **"MAC-DNS-ARP-DHCP-SwitchPort-Spoof"**
> MAC flooding → DNS poisoning → ARP poisoning → DHCP attacks → Switch port stealing → Spoofing

**4 técnicas de DNS poisoning:**
> **"Intra-Inter-Proxy-Cache"**
> Intranet → Internet (Remote) → Proxy Server → Cache Poisoning

**Defensas de switch (emparejadas con ataques):**
> MAC Flooding → **Port Security**
> DHCP Starvation → **Port Security**
> Rogue DHCP Server → **DHCP Snooping**
> ARP Poisoning → **DAI (requiere DHCP Snooping)**
> Switch Spoofing → **`switchport mode nonegotiate`**
> Double Tagging → **Cambiar native VLAN a ID no usado (999)**
> STP Attack → **BPDU Guard / Root Guard / Loop Guard / UDLD**

**Métodos de detección de sniffers:**
> **"Ping(IP ok, MAC wrong) → DNS(reverse lookup) → ARP(non-broadcast)"**

**Protocolos seguros vs inseguros:**
> Telnet → SSH | FTP → SFTP/SCP | HTTP → HTTPS | SNMPv1/v2 → SNMPv3 | POP → POP2/POP3

---

## 5. Flashcards

**Q:** ¿Qué es el sniffing pasivo y en qué entorno funciona?
**A:** Captura y monitoriza paquetes sin enviar ninguno. Funciona en redes con hubs (common collision domain). Difícil de detectar porque no transmite.

**Q:** ¿Qué ocurre cuando la tabla CAM de un switch se llena?
**A:** El switch entra en fail-open mode y actúa como hub, broadcasting todo el tráfico a todos los puertos.

**Q:** ¿Qué herramienta realiza MAC flooding y a qué velocidad?
**A:** macof (parte de la suite dsniff). Inunda el switch con 131,000 entradas por minuto.

**Q:** ¿Por qué ARP es vulnerable a spoofing?
**A:** Porque es stateless: una máquina puede enviar un ARP reply sin que nadie lo haya pedido, y los sistemas aceptan ARP replies sin verificar su autenticidad.

**Q:** ¿Qué debe configurarse ANTES de habilitar DAI?
**A:** DHCP Snooping. DAI usa la DHCP snooping binding table para validar los paquetes ARP.

**Q:** ¿Cuál es el estado por defecto de los puertos cuando DHCP Snooping está habilitado?
**A:** Todos los puertos de la VLAN son untrusted por defecto.

**Q:** ¿Cuál es el valor por defecto del Bridge Priority en STP?
**A:** 32769.

**Q:** ¿Qué estado activa BPDU Guard cuando recibe un BPDU no autorizado?
**A:** Errdisable mode (el puerto se desactiva completamente).

**Q:** ¿Qué estado activa Root Guard cuando recibe un BPDU superior?
**A:** Loop inconsistent state (no errdisable).

**Q:** ¿Solo el método Double Tagging VLAN hopping funciona con qué condición?
**A:** Solo si los puertos del switch están configurados para usar native VLANs.

**Q:** ¿Qué protocolo de seguridad es obligatorio en IPv6 pero opcional en IPv4?
**A:** IPsec.

**Q:** ¿Cómo funciona el método Ping para detectar un sniffer?
**A:** Se envía un ping a la máquina sospechosa con su IP correcta pero MAC incorrecta. Un sistema normal lo rechaza; un sistema con sniffer responde porque no filtra por MAC.

**Q:** ¿Cómo defiende DHCP Snooping contra servidores DHCP rogue?
**A:** Solo el puerto marcado como trusted puede enviar respuestas DHCP. Los puertos untrusted no pueden responder a DHCP Discover packets.

**Q:** ¿Cuáles son las 4 técnicas de DNS Poisoning?
**A:** Intranet DNS Spoofing, Internet DNS Spoofing (Remote), Proxy Server DNS Poisoning, DNS Cache Poisoning.

**Q:** ¿Qué contramedida específica usa DNSSEC para proteger contra DNS Poisoning?
**A:** Implementar Domain Name System Security Extensions (DNSSEC), que añade firmas criptográficas a los registros DNS para validar su autenticidad.

**Q:** ¿Qué comando Nmap detecta NICs en modo promiscuo?
**A:** `nmap --script=sniffer-detect [Target IP/Range]`

**Q:** ¿Qué diferencia SPAN port de wiretapping?
**A:** SPAN port es una característica legítima del switch (port mirroring) para monitorización de red. Wiretapping es la interceptación covert de conversaciones por terceros, generalmente con intenciones maliciosas.

**Q:** ¿Qué hace el comando `switchport mode nonegotiate`?
**A:** Impide que el puerto negocie trunks, previniendo ataques de switch spoofing en VLAN hopping.

---

## 6. Confusión frecuente

**MAC Flooding vs Switch Port Stealing** → MAC Flooding: inunda la CAM table con MACs falsas para que el switch actúe como hub. Switch Port Stealing: spoofea la MAC del objetivo con gratuitous ARP para que el switch asocie esa MAC con el puerto del atacante. MAC Flooding es masivo; Switch Port Stealing es quirúrgico.

**ARP Poisoning vs DNS Poisoning** → ARP Poisoning: falsifica asociaciones IP-MAC para redirigir tráfico de capa 2. DNS Poisoning: falsifica asociaciones dominio-IP para redirigir peticiones a nivel de resolución de nombres. ARP opera en capa 2; DNS opera en capa de aplicación.

**DHCP Starvation vs Rogue DHCP** → DHCP Starvation: agota el pool de IPs del servidor DHCP (DoS). Rogue DHCP: configura un servidor DHCP falso para asignar configuraciones maliciosas a clientes (MITM). Son ataques complementarios: la starvation facilita el rogue server.

**BPDU Guard vs Root Guard** → BPDU Guard: protege puertos que no deben recibir BPDUs (access ports) → errdisable. Root Guard: protege el root bridge de ser desplazado por switches no autorizados → loop inconsistent. BPDU Guard es más drástico.

**Intranet DNS Spoofing vs Internet DNS Spoofing** → Intranet: requiere acceso físico a la LAN, usa ARP poisoning. Internet: usa un Trojan para cambiar la DNS primaria de la víctima remotamente, puede afectar víctimas en cualquier parte del mundo.

**DHCP Snooping vs DAI** → DHCP Snooping: filtra mensajes DHCP no confiables y construye la binding table (MAC, IP, VLAN, interfaz). DAI: usa esa binding table para validar paquetes ARP. DHCP Snooping es la base; DAI es la defensa contra ARP poisoning que la aprovecha.

**Active vs Passive Wiretapping** → Active: monitoriza, registra Y puede alterar/inyectar datos (= MITM). Passive: solo monitoriza y registra (= snooping/eavesdropping). Active es más peligroso porque puede modificar la comunicación.

**Port Security vs DHCP Snooping** → Port Security: limita el número de MACs en un puerto (defiende contra MAC flooding y DHCP starvation). DHCP Snooping: filtra mensajes DHCP por confiabilidad del puerto (defiende contra rogue DHCP servers). Son defensas complementarias en la misma capa.
