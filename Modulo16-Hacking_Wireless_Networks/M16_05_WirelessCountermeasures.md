# M16_05_WirelessCountermeasures.md
> Módulo 16 / Subapartado 5 — Wireless Attack Countermeasures

---

## 1. Conceptos y definiciones

### 🔴 6 Capas de seguridad wireless

| Capa | Nombre | Descripción |
|---|---|---|
| **1** | Wireless Signal Security | Monitorización continua del espectro RF con WIDS; detecta dispositivos no autorizados, rogue APs, interferencias RF, aumento de uso de bandwidth |
| **2** | Connection Security | Autenticación per frame/packet → protege contra MITM; previene sniffing entre usuarios legítimos |
| **3** | Device Security | Vulnerability management y patch management |
| **4** | Data Protection | Algoritmos de cifrado: WPA3, WPA2, AES |
| **5** | Network Protection | Autenticación robusta → solo usuarios autorizados acceden a la red |
| **6** | End-User Protection | Personal firewalls en sistemas de usuario final de la WLAN → incluso si el atacante se asocia con APs, no puede acceder a ficheros |

---

### 🔴 Defensa contra WPA/WPA2/WPA3 Cracking

| Medida | Detalle |
|---|---|
| **Contraseñas fuertes** | Mínimo 12-16 caracteres; mayúsculas, minúsculas, números y caracteres especiales |
| **WPA2: AES/CCMP únicamente** | Usar solo AES/CCMP; validar servidor; especificar dirección del servidor; regenerar claves para cada nueva conexión |
| **Deshabilitar TKIP** | Solo AES en la configuración del router |
| **MAC Address Filtering** | Solo dispositivos con MACs específicas pueden conectarse |
| **Upgrade a WPA3** | Mejor protección contra brute-force; previene explotación de dispositivos conectados |
| **Deshabilitar Remote Management** | Evitar ataques externos al router |
| **Deshabilitar WPS** | WPS tiene vulnerabilidades conocidas que permiten brute-force del PIN |
| **Actualizar firmware del router** | Parchar vulnerabilidades conocidas regularmente |
| **Reducir rango de señal** | Limitar potencia de transmisión; colocar router centralmente |
| **Monitorizar actividad de red** | Detectar actividad inusual y dispositivos no autorizados |
| **Habilitar WPA3-SAE** | Protección contra offline dictionary attacks; forward secrecy |
| **Deshabilitar Transition Mode** | Si todos los dispositivos soportan WPA3, deshabilitar modo de transición WPA2/WPA3 |
| **VPN** | Remote access VPN, extranet VPN, intranet VPN |
| **IPsec y SSL/TLS** | Para comunicación segura |
| **NAC/NAP** | Network Access Control/Protection para control adicional sobre conectividad de usuarios |

---

### 🔴 Defensa contra KRACK Attacks

- Actualizar routers y dispositivos Wi-Fi con últimos patches de seguridad
- Activar auto-updates en todos los dispositivos inalámbricos; parchear firmware
- Evitar redes Wi-Fi públicas
- Solo navegar en sitios HTTPS seguros; no acceder a recursos sensibles en redes no protegidas
- Habilitar extensión **HTTPS Everywhere**
- Habilitar **two-factor authentication**
- Usar **VPN** para proteger información en tránsito
- Usar **WPA3** en redes wireless
- **Deshabilitar fast roaming y repeater mode** para mejorar la mitigación de KRACK
- Emplear **EAPOL-key replay counter** para que el AP reconozca solo el valor del contador más reciente
- Usar conexión cableada (Ethernet) o datos móviles cuando se detecta vulnerabilidad KRACK
- Routers de terceros en lugar de routers del ISP si no proporcionan patches de seguridad suficientes
- **Segmentación de red** para separar partes críticas del acceso general de usuarios
- **Deshabilitar el protocolo 802.11r** (susceptible a KRACK) si no se necesita para roaming
- Usar **802.1X authentication** + RADIUS server para redes enterprise

---

### 🔴 Defensa contra aLTEr Attacks

**Método principal recomendado:** cifrar queries DNS con estándares de seguridad adecuados.
**Herramienta de Cisco:** "Cisco Security Connector" (desarrollada con Apple) + Cisco Umbrella (intelligence block).

| Medida | Descripción |
|---|---|
| Cifrar queries DNS | Usar solo DNS resolvers de confianza |
| DNS over HTTPS | Resolver queries DNS via protocolo HTTPS |
| Acceder solo a HTTPS | No navegar en sitios HTTP en redes no seguras |
| **DNS over TLS / DTLS** | Cifrado e integridad para tráfico DNS |
| **RFC 7858 / RFC 8310** | Previene DNS spoofing; mejora cifrado y políticas de resolución de nombres |
| MAC en paquetes de user plane | Añadir código de autenticación de mensaje |
| **DNSCrypt** | Autentica comunicación entre cliente DNS y resolver DNS |
| **HSTS** | Parámetros HTTPS correctos para evitar redirección a sitios maliciosos |
| VPN con integridad | Túnel virtual con protección de integridad y autenticación de endpoints |
| **Actualizar a 5G** | Mejor seguridad que 4G/LTE |
| **eSIM technology** | Mejora autenticación y cifrado |
| **DNSSEC** | Asegura el proceso de DNS lookup; garantiza autenticidad de datos de respuesta |
| AES-256 | Cifrado extremo a extremo en redes LTE |
| **Mutual authentication** | Verificar identidad de ambas partes (UE y red) |
| SIM cards seguras | Con features como OTA updates y almacenamiento seguro |
| Location-based access controls | Restringir acceso según ubicación geográfica |
| Seguridad física | Proteger infraestructura de red (vigilancia, control de acceso, sellos anti-manipulación) |

---

### 🔴 Detección y Bloqueo de Rogue APs

#### Detección

| Método | Descripción |
|---|---|
| **RF Scanning** | APs re-propositados como RF sensors (solo captura y análisis de paquetes) distribuidos por la red cableada; alertan al admin WLAN sobre dispositivos wireless no autorizados |
| **AP Scanning** | APs con funcionalidad de detectar APs vecinos → expone datos vía MIBS e interfaz web |
| **Wired Side Inputs** | Software de gestión de red detecta dispositivos en la LAN usando Telnet, SNMP y CDP (Cisco Discovery Protocol) |
| **Comparison with authorized AP list** | Comparar APs detectados con lista de APs autorizados; herramientas: AirMagnet WiFi Analyzer |
| **Signal Strength Analysis** | Identificar APs físicamente cercanos pero no autorizados; herramienta: Ekahau Survey for Wi-Fi Planning and Analysis |
| **MAC Address Filtering** | Monitorizar MACs de APs autorizados conocidos; herramienta: Cisco Wireless LAN Controllers (built-in rogue AP detection) |

#### Bloqueo

- Lanzar ataque **DoS** sobre el rogue AP para denegar servicio a nuevos clientes
- **Bloquear el puerto del switch** al que está conectado el AP, o localizarlo físicamente y eliminarlo
- Usar **WIPS** para monitorización continua y acciones automatizadas de bloqueo
- **ACLs** para restringir acceso a MACs autorizadas
- **802.1X authentication** para controlar acceso a la red
- **Segmentación de red** para aislar recursos críticos del acceso wireless general
- Deshabilitar broadcasting de SSIDs abiertos
- Whitelist de MACs autorizadas en el wireless controller

---

### 🔴 Best Practices — Configuración

- Cambiar el **SSID por defecto** después de configurar la WLAN
- Configurar password del router y habilitar protección por firewall
- **Deshabilitar SSID broadcast**
- Deshabilitar remote router login y wireless administration
- Habilitar **MAC address filtering** en APs/routers
- Habilitar cifrado en APs y cambiar passphrases frecuentemente
- Cerrar todos los puertos no usados
- **Segregar la red** (invitados no acceden a la red privada)
- Usar closed networks (proporcionar SSID a empleados, no broadcast)
- **Deshabilitar DHCP** y usar IPs estáticas
- **Deshabilitar SNMP** o configurar con mínimos privilegios
- Cambiar la IP por defecto de la consola del router
- Usar **WPA3** si está soportado; si no, WPA2 con AES
- **Deshabilitar WPS** en el router
- Usar VLANs o SSIDs separados para segmentar tipos de tráfico
- Ajustar potencia de transmisión del router
- Configurar **red de invitados separada** con acceso restringido
- Usar el firewall integrado del router

---

### Best Practices — SSID Settings

- Usar **SSID cloaking** para evitar broadcast de mensajes wireless por defecto
- No usar SSID, nombre de empresa, nombre de red o strings fáciles de adivinar en passphrases
- Colocar **firewall o packet filter** entre AP e Intranet corporativa
- Limitar la fuerza de la red wireless para que no sea detectable fuera de la organización
- Verificar regularmente dispositivos wireless por problemas de configuración
- Implementar **IPsec over wireless** para cifrado adicional de tráfico
- Modificar el SSID con caracteres únicos (no usar SSID por defecto del fabricante)
- Usar **SSID separado para usuarios invitados**
- Separar la red en múltiples zonas con sus propios SSIDs
- **Mantener SSID broadcast en modo oculto** para dispositivos de la organización
- Proteger cada SSID con WPA3 o WPA2+AES como mínimo
- **Cambiar SSIDs y contraseñas asociadas periódicamente**

---

### Best Practices — Authentication

- Habilitar **WPA3** para el nivel de seguridad más alto
- Si WPA3 no está soportado → WPA2 con AES; **evitar WPA o TKIP**
- Usar **802.1X con RADIUS server** para redes enterprise (credenciales individuales por usuario)
- Implementar **MFA** donde sea posible
- Gestión adecuada de certificados digitales para 802.1X
- Deshabilitar la red cuando no se requiera
- Colocar APs en ubicaciones seguras
- Mantener drivers de equipos wireless actualizados
- Usar servidor centralizado para autenticación
- Habilitar **server verification en el cliente** con 802.1X para prevenir MITM attacks
- Habilitar **two-factor authentication**
- Desplegar **rogue-AP detection** o **WIDS/WIPS**

---

### 🔴 WIPS — Wireless Intrusion Prevention System

**Definición:** dispositivo de red que monitoriza el espectro de radio para detectar APs no autorizados; puede implementar contramedidas automáticamente.

**Componentes del despliegue WIPS (Cisco):**

| Componente | Función |
|---|---|
| **APs in monitor mode** | Escaneo continuo de canales; detección de ataques; captura de paquetes |
| **Mobility Services Engine (MSE)** | Punto central de agregación de alarmas de todos los controllers y APs en monitor mode; almacena alarmas y ficheros forenses |
| **Local mode AP(s)** | Proporciona servicio wireless a clientes + rogue scanning y location scanning |
| **Wireless LAN Controller(s)** | Reenvía información de ataques de los APs en monitor mode a la MSE; distribuye parámetros de configuración a los APs |
| **Wireless Control System** | Configura el servicio wireless IPS en la MSE; envía configuraciones WIPS a los controllers; pone APs en monitor mode; permite ver alarmas, forensics y reporting |

**Herramientas WIPS principales:**
- Cisco Adaptive Wireless IPS
- WatchGuard Wi-Fi Cloud WIPS
- Extreme AirDefense
- Arista WIPS
- SonicWall Wireless Network Manager

---

## 2. Exam Traps ⚠️

⚠️ **[6 capas de seguridad wireless: orden y nombres exactos]**
Las 6 capas en orden: (1) Wireless Signal Security, (2) Connection Security, (3) Device Security, (4) Data Protection, (5) Network Protection, (6) End-User Protection. El examen puede reordenarlas o cambiar nombres.

⚠️ **[Connection Security: autenticación per frame/packet → protege contra MITM]**
La capa de Connection Security usa autenticación per frame/packet específicamente para proteger contra MITM attacks. No es cifrado de datos (eso es Data Protection). El examen puede confundir las capas.

⚠️ **[EAPOL-key replay counter para mitigación KRACK]**
El EAPOL-key replay counter asegura que el AP reconoce solo el valor del contador más reciente, mitigando replay attacks en el contexto de KRACK. El examen puede preguntar qué mecanismo específico contrarresta KRACK a nivel de protocolo.

⚠️ **[802.11r susceptible a KRACK → deshabilitar si no se necesita]**
El protocolo 802.11r (fast roaming) es susceptible a KRACK. Se recomienda deshabilitarlo si no es necesario. El examen puede no asociar 802.11r con KRACK.

⚠️ **[aLTEr: la solución principal es cifrar queries DNS; herramienta de Cisco = Cisco Security Connector + Umbrella]**
El método principal recomendado para aLTEr es cifrar queries DNS. La herramienta específica de Cisco es "Cisco Security Connector" (desarrollada con Apple) que usa Cisco Umbrella. El examen puede pedir la herramienta específica.

⚠️ **[RFC 7858 / RFC 8310 para prevenir DNS spoofing en defensa contra aLTEr]**
Estos RFC específicos se implementan para prevenir DNS spoofing. Son datos específicos y preguntables directamente. El examen puede presentar RFCs diferentes como opciones.

⚠️ **[RF Scanning usa APs re-propositados como sensores en la red cableada]**
En RF Scanning, los APs re-propositados se "distribuyen por toda la red cableada" (not wireless) como sensores para detectar dispositivos wireless no autorizados. El examen puede presentar esto como una solución puramente wireless.

⚠️ **[Wired Side Inputs: usa Telnet, SNMP y CDP]**
Wired Side Inputs usa específicamente Telnet, SNMP y Cisco Discovery Protocol (CDP) para detectar rogue APs. El examen puede preguntar qué protocolos usa esta técnica.

⚠️ **[Best practice: deshabilitar DHCP + usar IPs estáticas]**
Deshabilitar DHCP y usar IPs estáticas es una best practice específica de configuración wireless. El examen puede presentarlo como "poco práctico para redes grandes" — desde perspectiva de seguridad, el libro lo recomienda.

⚠️ **[WIPS vs. WIDS: WIDS solo detecta; WIPS detecta Y contrarresta automáticamente]**
WIDS = Wireless Intrusion Detection System (solo detecta). WIPS = Wireless Intrusion Prevention System (detecta Y puede implementar contramedidas automáticamente). El examen puede confundirlos.

---

## 3. Nemotécnicos

**6 capas de seguridad wireless** → **"S-C-D-D-N-E"**:
- **S**ignal (wireless signal) · **C**onnection (per-frame auth) · **D**evice (vuln/patch mgmt) · **D**ata (WPA3/AES) · **N**etwork (strong auth) · **E**nd-user (personal firewalls)

**Defensa KRACK — puntos clave** → **"Patch + WPA3 + No-public-WiFi + HTTPS + VPN + No-802.11r + 802.1X + EAPOL-counter + Segmentation"**

**Defensa aLTEr — puntos clave** → **"DNS-cifrado + DNS-over-TLS + HTTPS + RFC7858/8310 + DNSSEC + 5G + eSIM + HSTS"**

**Detección rogue APs (6 métodos)** → **"RF-AP-Wired-List-Signal-MAC"**:
- **RF** Scanning · **AP** Scanning · **Wired** Side Inputs · Authorized **List** Comparison · **Signal** Strength · **MAC** Address Filtering

**Bloqueo rogue APs** → **"DoS + Port-block + WIPS + ACL + 802.1X + Segment + SSIDs"**

**WIPS Cisco componentes (5)** → **"Monitor-APs + MSE + Local-APs + Controllers + Control-System"**

**Best practices configuración** → reglas de oro:
1. Cambiar SSID por defecto
2. Deshabilitar SSID broadcast
3. Deshabilitar WPS
4. Deshabilitar DHCP → IPs estáticas
5. Deshabilitar SNMP (o mínimos privilegios)
6. WPA3 (o WPA2+AES como mínimo)
7. MAC filtering habilitado
8. Red de invitados separada

---

## 4. Flashcards

**Q:** ¿Cuáles son las 6 capas de seguridad wireless en orden?
**A:** (1) Wireless Signal Security, (2) Connection Security, (3) Device Security, (4) Data Protection, (5) Network Protection, (6) End-User Protection.

---

**Q:** ¿Qué capa de seguridad wireless usa autenticación per frame/packet para proteger contra MITM?
**A:** Capa 2 — Connection Security.

---

**Q:** ¿Qué capa de seguridad wireless emplea personal firewalls en sistemas de usuario final?
**A:** Capa 6 — End-User Protection.

---

**Q:** ¿Cuál es el mecanismo específico de KRACK que se recomienda emplear para que el AP reconozca solo el contador más reciente?
**A:** EAPOL-key replay counter.

---

**Q:** ¿Qué protocolo 802.11 se recomienda deshabilitar para mitigar KRACK y para qué se usa normalmente?
**A:** 802.11r — normalmente se usa para fast roaming (seamless roaming); es susceptible a KRACK.

---

**Q:** ¿Cuál es el método principal recomendado para defenderse contra aLTEr attacks?
**A:** Cifrar queries DNS con estándares de seguridad adecuados.

---

**Q:** ¿Qué herramienta desarrolló Cisco (con Apple) para defenderse contra aLTEr y qué servicio usa?
**A:** "Cisco Security Connector" — usa Cisco Umbrella (intelligence block) para cifrar queries DNS y validarlas.

---

**Q:** ¿Qué estándares RFC se implementan para prevenir DNS spoofing en la defensa contra aLTEr?
**A:** RFC 7858 y RFC 8310.

---

**Q:** ¿Qué técnica de detección de rogue APs re-propósita APs como sensores distribuidos por la red cableada?
**A:** RF Scanning — los APs re-propositados como RF sensors se distribuyen por toda la red cableada para detectar y alertar al admin WLAN sobre dispositivos wireless no autorizados.

---

**Q:** ¿Qué protocolos usa la técnica de "Wired Side Inputs" para detectar rogue APs?
**A:** Telnet, SNMP y Cisco Discovery Protocol (CDP).

---

**Q:** ¿Cuál es la diferencia entre WIDS y WIPS?
**A:** WIDS (Wireless Intrusion Detection System): solo detecta y alerta sobre amenazas wireless. WIPS (Wireless Intrusion Prevention System): detecta amenazas Y puede implementar contramedidas automáticamente para bloquearlas.

---

**Q:** ¿Cuáles son los 5 componentes del despliegue WIPS de Cisco?
**A:** APs in monitor mode, Mobility Services Engine (MSE), Local mode AP(s), Wireless LAN Controller(s), Wireless Control System.

---

**Q:** ¿Qué función tiene el Mobility Services Engine (MSE) en un despliegue WIPS?
**A:** Es el punto central de agregación de alarmas de todos los controllers y sus APs en monitor mode; almacena información de alarmas y ficheros forenses.

---

**Q:** ¿Por qué se recomienda deshabilitar WPS como best practice?
**A:** WPS tiene vulnerabilidades conocidas que permiten ataques de brute-force al PIN para obtener acceso a la red.

---

**Q:** ¿Por qué se recomienda deshabilitar DHCP en configuración de seguridad wireless?
**A:** Para reducir el riesgo de que dispositivos no autorizados obtengan configuración de red automáticamente; usar IPs estáticas hace más difícil para un atacante conectarse.

---

**Q:** ¿Qué es SSID cloaking y por qué no es suficiente como medida de seguridad?
**A:** Mantiene el SSID en modo oculto (no broadcast). No es suficiente como única medida porque el SSID puede obtenerse de paquetes de probe requests/responses y frames de datos.

---

**Q:** ¿Qué configuración de autenticación se recomienda para redes enterprise wireless?
**A:** 802.1X con RADIUS server para proporcionar credenciales individuales por usuario.

---

**Q:** ¿Cuál es la recomendación de cifrado si WPA3 no está disponible?
**A:** WPA2 con AES/CCMP. Evitar WPA o TKIP.

---

**Q:** ¿Qué tecnología de 5G se menciona para mejorar autenticación y cifrado en defensa contra aLTEr?
**A:** eSIM technology — proporciona autenticación y cifrado mejorados.

---

**Q:** ¿Cuál es la diferencia entre DNSCrypt y DNSSEC en la defensa contra aLTEr?
**A:** DNSCrypt: autentica la comunicación entre cliente DNS y resolver DNS. DNSSEC: asegura el proceso de DNS lookup garantizando la autenticidad de los datos de respuesta (firma criptográfica de registros DNS).

---

## 5. Confusión frecuente

**WIDS vs. WIPS — detección vs. prevención**
- WIDS: Wireless Intrusion Detection System. Solo detecta y alerta sobre actividad wireless no autorizada mediante análisis del espectro RF. No actúa automáticamente.
- WIPS: Wireless Intrusion Prevention System. Detecta Y puede implementar contramedidas automáticamente (bloquear dispositivos, lanzar DoS contra rogue APs, etc.).
- Criterio: "detectar y alertar" → WIDS. "Detectar y bloquear automáticamente" → WIPS.

---

**RF Scanning vs. AP Scanning vs. Wired Side Inputs — tres métodos de detección de rogue APs**
- RF Scanning: APs re-propositados distribuidos en la red CABLEADA como sensores; detectan cualquier señal wireless no autorizada en el área.
- AP Scanning: APs normales con funcionalidad adicional de detectar APs vecinos; expone datos via MIBS e interfaz web.
- Wired Side Inputs: software de gestión de red usa protocolos de red (Telnet, SNMP, CDP) para detectar dispositivos conectados a la LAN.
- Criterio: "sensores RF en la red cableada" → RF Scanning. "APs detectando otros APs" → AP Scanning. "Software usando Telnet/SNMP/CDP" → Wired Side Inputs.

---

**DNSCrypt vs. DNS over TLS vs. DNSSEC**
- DNSCrypt: protocolo para autenticar comunicación entre cliente DNS y resolver (anti-spoofing de resolver).
- DNS over TLS: cifra el canal de comunicación de las queries DNS usando TLS.
- DNSSEC: firma criptográficamente los registros DNS para garantizar su autenticidad (anti-cache poisoning).
- Criterio: "autenticar el resolver" → DNSCrypt. "Cifrar el canal DNS" → DNS over TLS. "Autenticar los registros DNS" → DNSSEC.

---

**Best Practice: deshabilitar WPS vs. deshabilitar SSID broadcast**
- Deshabilitar WPS: elimina el vector de ataque del PIN WPS (brute-force). WPS no está relacionado con el broadcast del SSID.
- Deshabilitar SSID broadcast: el SSID no aparece en listas de redes disponibles (SSID cloaking). No elimina la posibilidad de detectar la red por otros medios.
- Criterio: "prevenir brute-force del PIN" → deshabilitar WPS. "Ocultar la red de listas visibles" → deshabilitar SSID broadcast. Ambas son recomendadas pero independientes.

---

**Connection Security (capa 2) vs. Data Protection (capa 4)**
- Connection Security (capa 2): autenticación per frame/packet → protege la integridad de la conexión contra MITM; previene que el atacante intercepte la comunicación entre dos usuarios.
- Data Protection (capa 4): cifrado de los datos transmitidos con WPA3/WPA2/AES; protege la confidencialidad del contenido.
- Criterio: "autenticar cada frame para prevenir MITM" → Connection Security. "Cifrar datos transmitidos" → Data Protection.
