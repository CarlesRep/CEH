# M11_05_SessionHijackingCountermeasures.md
## CEH v13 — Módulo 11 | Session Hijacking Countermeasures

---

## 1. Conceptos y definiciones

### Síntomas de un ataque de session hijacking

- Ráfaga de actividad de red que reduce el rendimiento del sistema.
- Servidores ocupados por peticiones tanto del cliente legítimo como del hijacker.

---

### 🔴 Métodos de detección — Manual vs. Automático

| Método | Herramientas | Mecanismo |
|---|---|---|
| **Manual** | Wireshark · SteelCentral Packet Analyzer | Captura de paquetes en tránsito; análisis con herramientas de filtrado |
| **Automático** | IDS / IPS | Monitorización del tráfico entrante; si el paquete coincide con una firma → IDS genera alerta / IPS bloquea el tráfico |

#### Forced ARP Entry (parte del método manual)
Reemplaza la MAC de una máquina comprometida en el caché ARP del servidor por otra diferente para restringir el tráfico a esa máquina.

**Cuándo aplicar Forced ARP Entry**:
- Actualizaciones ARP repetidas.
- Frames entre cliente y servidor con MACs diferentes.
- ACK storms.

---

### 🔴 Contramedidas clave de session hijacking — Agrupadas

#### Gestión de sesiones (desarrollo web)
- **Regenerar el session ID después de cada login exitoso** → previene Session Fixation.
- Generar session IDs con strings largos o números aleatorios.
- **Aceptar solo session IDs generados por el servidor**.
- Evitar incluir el session ID en la URL o query string.
- Implementar **timeout absoluto** (independientemente de la actividad del usuario).
- Destruir sesiones en el servidor al hacer deauthenticación (no solo depender de la expiración).
- Reducir el life span de la sesión/cookie.
- Usar `Cache-Control: no-cache, no-store` y `Pragma: no-cache` en páginas sensibles.
- Implementar el atributo **SameSite** en cookies → previene CSRF.
- Habilitar **HTTPOnly** en cookies → prohíbe que scripts de usuario accedan a las cookies.
- Usar el flag **Secure** en cookies → envía cookies solo en peticiones HTTPS.

#### Cifrado y protocolos
- Usar **SSH** para crear canales de comunicación seguros.
- Pasar authentication cookies únicamente sobre **HTTPS**.
- Implementar **SSL/TLS** para cifrar toda la información en tránsito.
- Usar **IPsec** para cifrar información de sesión y comunicaciones IP.
- Usar **VPNs cifradas** (PPTP, L2PT) para conexiones remotas.
- Usar **SFTP, AS2, FTPS** para transferencia de ficheros con cifrado y certificados digitales.
- Habilitar **HSTS** → convierte automáticamente conexiones HTTP a HTTPS.
- Implementar **DNS over HTTPS (DoH)**.
- Deshabilitar los mecanismos de compresión de peticiones HTTP → contramedida CRIME.

#### Autenticación y acceso
- Implementar **MFA** para reducir el riesgo aunque se comprometa el session token.
- Usar autenticación fuerte como **Kerberos** o VPNs peer-to-peer.
- Implementar **Token Binding** → par de claves pública-privada por conexión.
- Vincular la sesión a la IP del usuario.
- Usar **behavioral biometrics** (ritmos de escritura, movimientos de ratón, patrones de navegación) para autenticación continua.
- Implementar **challenge-response** (CAPTCHA) al detectar actividad sospechosa.
- Implementar **risk-based authentication** antes de acceder a información sensible.
- Usar **zero-trust principles** (verificar a todos los usuarios, internos y externos).

#### Red e infraestructura
- Cambiar de hub a **switch** para reducir el riesgo de ARP spoofing.
- Usar **IDS/ARPwatch** para monitorizar ARP cache poisoning.
- Configurar **spoof rules** en gateways (internas y externas).
- Implementar **segmentación de red**.
- Activar **SMB signing** para firmar el tráfico (solución Microsoft).
- Implementar **WPA3** en redes inalámbricas.

---

### 🔴 HSTS (HTTP Strict Transport Security)

- Protege sitios HTTPS contra ataques **MITM**.
- Fuerza a los navegadores a interactuar con el servidor únicamente mediante **HTTPS**.
- Convierte automáticamente todas las conexiones HTTP inseguras en HTTPS.
- Garantiza que toda la comunicación está cifrada y que las respuestas provienen de un servidor autenticado.

---

### 🔴 Token Binding

- El cliente crea un **par de claves pública-privada por cada conexión** al servidor remoto.
- El cliente genera una **firma** usando la clave privada y la envía junto con la clave pública al servidor.
- El servidor verifica la firma con la clave pública del cliente.
- Aunque el atacante capture la firma, **no puede regenerarla ni reutilizarla** en otra conexión.
- Para cada nueva conexión se usa un nuevo par de claves.

---

### DNS over HTTPS (DoH)

- Versión mejorada del DNS que cifra las consultas DNS enviándolas por un **túnel HTTPS cifrado a través del puerto 443**.
- Se oculta en el tráfico HTTPS normal → indetectable por atacantes e ISPs.
- Envía solo un segmento del nombre de dominio (no el nombre completo) → mayor privacidad.
- Previene MITM y session hijacking en el proceso de DNS lookup.
- Mozilla adoptó DoH como protocolo por defecto en **2020** para sus clientes de EE.UU.

---

### 🔴 IPsec — Componentes

**Definición**: conjunto de protocolos del IETF para intercambio seguro de paquetes en la capa IP. Soporta IPv4 e IPv6.

**Servicios de seguridad**:
- Rechazo de paquetes repetidos (partial sequence integrity) · Data confidentiality (cifrado) · Access control · Connectionless integrity · Data origin authentication · Data integrity · Limited traffic-flow confidentiality · Network-level peer authentication · Replay protection.

**Componentes**:

| Componente | Función |
|---|---|
| **IPsec driver** | Cifrado/descifrado de paquetes a nivel de protocolo |
| **IKE (Internet Key Exchange)** | Genera las claves de seguridad para IPsec |
| **ISAKMP** | Permite comunicación cifrada entre dos equipos; establece Security Associations (SA) |
| **Oakley** | Usa el algoritmo **Diffie-Hellman** para crear master key y key por sesión |
| **IPsec Policy Agent** | Servicio Windows OS que aplica políticas IPsec a todas las comunicaciones del sistema |

---

### 🔴 Modos de IPsec — Transport vs. Tunnel

| | Transport Mode | Tunnel Mode |
|---|---|---|
| **Qué cifra** | Solo el **payload** (cabecera IP intacta) | Paquete IP **completo** (payload + cabecera) → nuevo IP header |
| **Seguridad** | Menor | Mayor |
| **Uso** | Comunicaciones end-to-end entre **dos hosts** | VPNs: network-to-network, host-to-network, host-to-host |
| **NAT** | Compatible (ESP) | Compatible + NAT traversal |
| **Uso habitual** | Entre dos hosts directamente | Entre dos gateways o host y gateway |

---

### 🔴 IPsec — AH vs. ESP

| | AH (Authentication Header) | ESP (Encapsulating Security Payload) |
|---|---|---|
| **Integridad** | ✅ | ✅ |
| **Autenticación de origen** | ✅ | ✅ |
| **Anti-replay** | ✅ (opcional) | ✅ |
| **Confidencialidad (cifrado)** | ❌ **No cifra** | ✅ Cifra |
| **Transport mode** | Protege payload + partes cabecera IP | Solo payload |
| **Tunnel mode** | Todo el paquete interior + campos cabecera exterior | Todo el paquete interior (payload + cabecera) |

**Regla clave**: AH = autenticación + integridad, **sin cifrado**. ESP = todo lo de AH + **cifrado**.

---

### 🔴 Arquitectura IPsec — Protocolos

| Protocolo | Función |
|---|---|
| **AH** | Integridad + autenticación de origen + anti-replay. Sin confidencialidad. |
| **ESP** | Todo lo de AH + confidencialidad (cifrado). |
| **IPsec DOI** | Define formatos de payload, tipos de intercambio y convenciones de nomenclatura para seguridad (algoritmos criptográficos, políticas). |
| **ISAKMP** | Protocolo clave que establece la seguridad para comunicaciones de gobierno, privadas y comerciales; combina autenticación, gestión de claves y security associations. |
| **Policy** | Define cuándo y cómo asegurar los datos y qué métodos de seguridad usar en cada nivel de red. |

---

### Herramientas de detección y prevención

| Herramienta | Tipo | Características clave |
|---|---|---|
| **USM Anywhere** (AT&T) | Detección | Threat detection, incident response, compliance; asset discovery, SIEM, IDS, threat intelligence, vulnerability assessment; cloud + on-premises + hybrid |
| **Wireshark** | Detección | Usa **Winpcap**; captura tráfico en: Ethernet, IEEE 802.11, PPP/HDLC, ATM, Bluetooth, USB, Token Ring, Frame Relay, FDDI |
| **Checkmarx One SAST** | Prevención | Análisis de código fuente estático; detecta MITM, Session Fixation, XSS; soporta CxOSA (open-source analysis) |
| **Fiddler** | Prevención | Web debugging proxy; descifra HTTPS con **técnica MITM de descifrado**; logs de todo el tráfico HTTP(S) |
| Quantum IPS · SolarWinds SEM · IBM Security Network IPS · LogRhythm | Detección adicional | IPS/SIEM |
| Nessus · Invicti · Wapiti | Prevención adicional | Vulnerability scanners |

---

## 2. Exam Traps ⚠️

⚠️ **[AH no cifra]** AH proporciona autenticación + integridad + anti-replay pero **NO confidencialidad (no cifra)**. ESP añade cifrado. Trampa más frecuente del bloque IPsec.

⚠️ **[Transport mode — solo payload]** En transport mode se cifra solo el payload; la cabecera IP queda intacta. En tunnel mode, todo el paquete original se encapsula y cifra.

⚠️ **[Tunnel mode — mayor seguridad]** Tunnel cifra cabecera + payload → mayor seguridad. El examen puede invertir esto.

⚠️ **[Oakley — Diffie-Hellman]** Oakley usa **Diffie-Hellman**. Si el examen pregunta "¿qué componente de IPsec usa Diffie-Hellman?" → Oakley.

⚠️ **[IKE vs. ISAKMP]** IKE **genera** claves. ISAKMP **establece** las SA y gestiona la negociación. Son complementarios, no iguales.

⚠️ **[Token Binding — par de claves por conexión, no reutilizable]** La firma capturada no puede reutilizarse en otra conexión porque la clave privada nunca sale del cliente.

⚠️ **[HTTPOnly ≠ Secure ≠ SameSite]** HTTPOnly → bloquea scripts (anti-XSS). Secure → solo HTTPS. SameSite → anti-CSRF.

⚠️ **[Regenerar session ID post-login → contramedida de Session Fixation]** Dato específico preguntado directamente.

⚠️ **[DNS over HTTPS — puerto 443]** DoH opera en el **puerto 443**, no en el 53 (DNS tradicional).

⚠️ **[Mozilla DoH por defecto — 2020, EE.UU.]** Dato numérico/temporal concreto preguntable.

⚠️ **[Wireshark — Winpcap]** Wireshark usa **Winpcap** → solo captura en redes soportadas por Winpcap.

⚠️ **[Checkmarx = SAST = prevención, no detección]** Checkmarx analiza código fuente estáticamente. No es un IDS ni un sniffer.

⚠️ **[Fiddler — técnica MITM de descifrado]** Fiddler descifra HTTPS usando una técnica MITM de descifrado. Es un proxy de debugging, no un escáner de vulnerabilidades.

---

## 3. Nemotécnicos

### AH vs. ESP — regla de bolsillo
- **AH** = **A**utenticación + **H**integridad (sin cifrado)
- **ESP** = **E**verything: autenticación + integridad + cifrado (**S**ecurity **P**lus)

### Transport vs. Tunnel — "T=Tráfico parcial, Tu=Todo"
- **T**ransport = solo **payload** (cabecera libre)
- **Tu**nnel = paquete **total** → nuevo IP header

### Componentes IPsec — "D-IKE-ISAKMP-O-P"
- **D**river · **IKE** (claves) · **ISAKMP** (SA) · **O**akley (DH) · **P**olicy Agent

### Cookies — 3 atributos — "H-S-Sa"
- **H**ttpOnly → sin scripts (anti-XSS)
- **S**ecure → solo HTTPS
- **Sa**meSite → sin cross-site (anti-CSRF)

### DoH — "443, no 53"
DNS sobre HTTPS = puerto **443** (HTTPS), no **53** (DNS clásico)

### Forced ARP Entry — cuándo aplicar — "ARP-F-ACK"
- **ARP** updates repetidas
- **F**rames con MACs diferentes
- **ACK** storms

---

## 4. Flashcards

**Q:** ¿Cuándo se debe aplicar una Forced ARP Entry?
**A:** Actualizaciones ARP repetidas · frames entre cliente y servidor con MACs diferentes · ACK storms.

**Q:** ¿Cuál es la diferencia funcional entre IDS e IPS en detección de session hijacking?
**A:** IDS detecta y genera alerta. IPS detecta y bloquea el tráfico.

**Q:** ¿Qué contramedida específica previene Session Fixation?
**A:** Regenerar el session ID después de cada login exitoso.

**Q:** ¿Qué atributo de cookie previene CSRF?
**A:** SameSite — impide que el navegador envíe cookies en peticiones cross-site.

**Q:** ¿Qué atributo de cookie prohíbe el acceso desde scripts JavaScript?
**A:** HTTPOnly.

**Q:** ¿Cuál es la diferencia entre AH y ESP en IPsec?
**A:** AH: autenticación + integridad + anti-replay, sin cifrado. ESP: todo lo de AH más confidencialidad (cifrado).

**Q:** ¿Qué cifra IPsec en Transport Mode y en Tunnel Mode?
**A:** Transport: solo el payload (cabecera IP intacta). Tunnel: paquete IP completo (payload + cabecera) → se convierte en payload de un nuevo paquete.

**Q:** ¿Qué componente de IPsec usa el algoritmo Diffie-Hellman?
**A:** Oakley. Lo usa para crear una master key y una key específica por sesión.

**Q:** ¿Qué es Token Binding y por qué protege contra session hijacking?
**A:** Crea un par de claves pública-privada único por cada conexión. El cliente firma con la clave privada; el servidor verifica con la pública. La firma capturada no puede reutilizarse porque la clave privada nunca sale del cliente.

**Q:** ¿Qué hace HSTS y contra qué protege?
**A:** Fuerza a los navegadores a usar solo HTTPS, convirtiendo automáticamente conexiones HTTP en HTTPS. Protege contra ataques MITM.

**Q:** ¿En qué puerto opera DNS over HTTPS y por qué es difícil de detectar?
**A:** Puerto 443. Se oculta dentro del tráfico HTTPS normal, siendo indetectable para atacantes e ISPs.

**Q:** ¿Qué herramienta de captura usa Wireshark y qué tipos de red soporta?
**A:** Winpcap. Soporta Ethernet, IEEE 802.11, PPP/HDLC, ATM, Bluetooth, USB, Token Ring, Frame Relay y FDDI.

**Q:** ¿Qué tipo de herramienta es Checkmarx One SAST y qué ataques ayuda a prevenir?
**A:** SAST (Static Application Security Testing) — análisis de código fuente. Previene MITM, Session Fixation y XSS.

**Q:** ¿Qué técnica usa Fiddler para descifrar tráfico HTTPS?
**A:** Técnica MITM de descifrado (MITM decryption technique).

**Q:** ¿Cuál es el uso habitual de Tunnel Mode vs. Transport Mode en IPsec?
**A:** Transport: comunicaciones end-to-end entre dos hosts. Tunnel: VPNs (gateway-to-gateway, host-to-gateway, host-to-host sobre Internet).

**Q:** ¿Qué año adoptó Mozilla DoH como protocolo por defecto y para qué clientes?
**A:** 2020, para clientes de EE.UU.

---

## 5. Confusión frecuente

### AH vs. ESP
- **AH**: autenticación + integridad + anti-replay. **Sin cifrado**.
- **ESP**: todo lo de AH + **cifrado**.
- **Criterio**: ¿el escenario requiere cifrado? → ESP. ¿Solo autenticación e integridad? → AH.

### Transport Mode vs. Tunnel Mode
- **Transport**: cabecera IP original visible; solo payload cifrado; host-to-host directo.
- **Tunnel**: todo cifrado + nuevo IP header; para VPNs entre gateways.
- **Criterio**: ¿la cabecera IP original debe ocultarse? → Tunnel. ¿Solo el contenido? → Transport.

### HTTPOnly vs. Secure vs. SameSite
- **HTTPOnly** → bloquea acceso desde JavaScript → previene XSS cookie theft.
- **Secure** → solo envía cookie en HTTPS → previene interceptación en claro.
- **SameSite** → previene envío de cookies en peticiones cross-site → previene CSRF.
- **Criterio**: ¿protección contra scripts? → HTTPOnly. ¿Contra transmisión en claro? → Secure. ¿Contra CSRF? → SameSite.

### IKE vs. ISAKMP vs. Oakley
- **IKE**: genera las claves de seguridad.
- **ISAKMP**: establece y gestiona las Security Associations (SA).
- **Oakley**: usa Diffie-Hellman para generar master key y keys de sesión.
- **Criterio**: ¿generación de claves? → IKE. ¿Gestión de SAs? → ISAKMP. ¿Diffie-Hellman? → Oakley.

### HSTS vs. Token Binding
- **HSTS**: fuerza HTTPS a nivel de transporte.
- **Token Binding**: vincula el token a la conexión específica mediante criptografía; opera a nivel de token de sesión.
- **Criterio**: ¿forzar HTTPS? → HSTS. ¿Vincular token a conexión para evitar reutilización? → Token Binding.
