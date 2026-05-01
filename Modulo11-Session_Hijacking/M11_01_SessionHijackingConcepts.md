# M11_01_SessionHijackingConcepts.md
## CEH v13 — Módulo 11 | Session Hijacking Concepts

---

## 1. Conceptos y definiciones

### Qué es Session Hijacking

Un servidor web envía un **session identification token** al cliente web tras una autenticación exitosa. Estos tokens diferencian las múltiples sesiones que el servidor mantiene con distintos clientes.

**Session hijacking**: ataque en el que el atacante toma el control de una sesión TCP activa y válida entre dos equipos. Como la mayoría de los mecanismos de autenticación solo operan al **inicio** de la sesión TCP, el atacante puede acceder al sistema mientras la sesión está en curso, sin necesidad de autenticarse.

El atacante puede:
- Adivinar o robar un **session ID** válido (que identifica a usuarios autenticados).
- Usar ese session ID para establecer una sesión con el servidor.
- El servidor responde como si hablara con el usuario autenticado legítimo.

**Ataques que habilita**: MITM (Man-in-the-Middle) · DoS · robo de identidad · fraude · robo de información.

---

### 🔴 Por qué el Session Hijacking tiene éxito — 6 factores

| Factor | Mecanismo de explotación |
|---|---|
| **Sin account lockout para session IDs inválidos** | El atacante prueba múltiples session IDs (brute-force) sin que el servidor emita advertencias ni bloquee la cuenta |
| **Algoritmo de generación débil o session IDs cortos** | Los sitios usan algoritmos lineales basados en tiempo o IP; el atacante analiza el patrón secuencial y estrecha el espacio de búsqueda; un ID corto es fácilmente determinable aunque el algoritmo sea fuerte |
| **Manejo inseguro de session IDs** | El atacante engaña al navegador para visitar otro sitio y recupera la información del session ID almacenado; vectores: DNS poisoning, XSS, bugs del navegador |
| **Session timeout indefinido** | IDs sin fecha de expiración dan tiempo ilimitado al atacante; explotable con la opción "remember me"; también explotable si el atacante compromete un proxy que loguea o cachea session IDs |
| **TCP/IP vulnerable por diseño** | Todos los equipos con TCP/IP son vulnerables por los flaws inherentes al diseño del protocolo |
| **Contramedidas ineficaces sin cifrado** | Es fácil sniffar session IDs en redes planas si no hay transport security, incluso con SSL en la aplicación web; peor si el session ID contiene credenciales reales |

---

### 🔴 Proceso de Session Hijacking — 3 fases

| Fase | Descripción |
|---|---|
| **1. Tracking the Connection** | El atacante usa un sniffer o Nmap para identificar a la víctima y capturar los números de secuencia y acknowledgment (SEQ/ACK); usa estos números para construir paquetes |
| **2. Desynchronizing the Connection** | El atacante altera el SEQ/ACK del servidor para crear un estado desincronizado; envía datos nulos al servidor para avanzar los contadores SEQ/ACK sin que el cliente lo registre; alternativa: enviar RST + SYN con diferente número de secuencia al mismo puerto para que el servidor cierre la conexión actual y abra una nueva |
| **3. Injecting the Attacker's Packet** | Una vez interrumpida la conexión, el atacante inyecta datos en la red o actúa como MITM, pasando datos entre cliente y servidor mientras lee e inyecta a voluntad |

#### Desincronización — Detalle técnico
- El atacante envía **datos nulos** al servidor → el SEQ/ACK del servidor avanza → el cliente no registra el incremento → desincronización.
- Alternativa: el atacante envía **RST + SYN** (mismo puerto, diferente SEQ) al servidor → el servidor cierra la conexión y abre una nueva con diferente SEQ.
- El atacante detecta (no intercepta) el SYN/ACK del servidor al cliente → envía ACK al servidor → el servidor queda en estado **established**.
- Resultado: tanto servidor como cliente están desincronizados pero en estado established.

**ACK Storm**: si el atacante usa un flag **FIN** en lugar de RST, el servidor responde con ACK, creando un bucle sin fin de paquetes ACK (ACK storm). Esto revela el ataque. Ocurre porque los paquetes inaceptables generan ACKs, creando tráfico de red excesivo. Como estos paquetes no llevan datos, no se retransmiten si se pierden; un único paquete perdido termina la conversación no deseada.

**Requisitos para hijack de TCP no cifrado**:
1. Tráfico de sesión TCP sin cifrar.
2. Capacidad de reconocer números SEQ/ACK del cliente para predecir el NSN.
3. Capacidad de falsificar MAC o IP para recibir comunicaciones no dirigidas al atacante.
4. Si el atacante está en el segmento local: puede sniffar y predecir **ISN + 1** y redirigir el tráfico envenenando los cachés ARP de ambos hosts legítimos.

---

### Packet Analysis — Local Session Hijack

Para hijack local:
1. Sniffar el tráfico de red para monitorizar la sesión.
2. Encontrar un paquete ACK → determinar el **NSN (Next Sequence Number)** a partir del ACK.
3. Usar ese NSN antes de que el cliente lo use → desincronización y control de la sesión.

El método alternativo (transmitir datos con SEQ guessados) no es fiable.

---

### 🔴 Tipos de Session Hijacking — Activo vs. Pasivo

| | Passive Hijacking | Active Hijacking |
|---|---|---|
| **Objetivo** | Observar y registrar el tráfico | Tomar el control de la sesión existente |
| **Técnica** | Sniffer en la red | MITM, desincronización + inyección |
| **Información obtenida** | User IDs, contraseñas | Control completo de la sesión |
| **Detección** | Difícil (pasiva) | Más fácil de detectar (ACK storms, anomalías) |
| **Contramedida básica** | One-time passwords (S/KEY), Kerberos | Cifrado de sesión completa |

**Nota**: las contramedidas de passive hijacking (one-time passwords, Kerberos) **no protegen** contra active hijacking si los datos no están cifrados o no llevan firma digital.

---

### 🔴 Session Hijacking en el modelo OSI — 2 niveles

| Nivel | Descripción | Características |
|---|---|---|
| **Network-Level Hijacking** | Interceptación de paquetes TCP/UDP entre cliente y servidor | No requiere modificación por aplicación web; ataca el flujo de datos del protocolo compartido por todas las webs; proporciona información para atacar sesiones de aplicación |
| **Application-Level Hijacking** | Control de sesiones HTTP mediante obtención de session IDs | Permite crear nuevas sesiones no autorizadas con datos robados; normalmente ocurren juntos según el sistema atacado |

---

### 🔴 Spoofing vs. Hijacking — Diferencias clave

| | Spoofing | Session Hijacking |
|---|---|---|
| **Sesión existente** | Inicia una **nueva** sesión con credenciales robadas | **Toma el control** de una sesión activa existente |
| **Autenticación** | Usa credenciales capturadas | Explota el session ID de una sesión ya autenticada |
| **Dificultad** | Más sencillo (IP spoofing simple) | Más difícil (requiere conocer SEQ numbers, estado de la sesión, método de desplazar al usuario legítimo) |
| **Requisito de SEQ** | No necesita conocer SEQ si no hay sesión abierta con esa IP | Necesita conocer el SEQ en uso en el momento del hijack (ISN + nº de paquetes intercambiados) |

**Blind Hijacking**: el atacante predice los SEQ numbers sin poder ver las respuestas (no hay ruta de vuelta para los paquetes hacia su IP). Para esto:
- No puede usar ARP cache poisoning (los routers no hacen broadcast ARP por Internet).
- Debe predecir las respuestas de la víctima y evitar que el host legítimo envíe TCP/RST.
- Útil para explotar relaciones de confianza entre usuarios y máquinas remotas.

**Source Routing**: en session hijacking, el tráfico solo vuelve al atacante si se usa **source routing** (el emisor especifica la ruta del paquete IP hacia el destino). El atacante realiza source routing y sniffa el tráfico al pasar por su sistema.

**Condiciones que hacen imposible el hijacking/spoofing**:
- Per-packet integrity checking activado.
- Cifrado de sesión con SSL o PPTP (el atacante no puede participar en el key exchange).

---

## 2. Exam Traps ⚠️

⚠️ **[Autenticación solo al inicio]** El examen puede preguntar por qué el session hijacking es posible. La respuesta es que la autenticación solo se verifica al **inicio** de la sesión TCP, no en cada paquete.

⚠️ **[Passive vs. Active Hijacking]** Passive = solo observa. Active = toma control. El examen puede describir un escenario y preguntar si es activo o pasivo. Sniffing de contraseñas = passive. MITM = active.

⚠️ **[ACK Storm — causa y efecto]** El ACK storm revela el ataque. Se produce cuando el atacante usa **FIN** en lugar de RST, generando un bucle de paquetes ACK. La pérdida de un solo paquete termina el storm (TCP usa IP, sin retransmisión de paquetes sin datos).

⚠️ **[Network-Level vs. Application-Level]** Network-level: TCP/UDP, no requiere modificación por aplicación. Application-level: HTTP session IDs. El examen puede preguntar cuál es más común entre atacantes → Network-level (no requiere adaptar el ataque por aplicación).

⚠️ **[Blind Hijacking — sin ARP por Internet]** El atacante no puede usar ARP cache poisoning en blind hijacking porque los routers no hacen broadcast ARP por Internet. Es el motivo por el que debe predecir SEQ numbers en lugar de sniffar.

⚠️ **[Source Routing — condición para recibir tráfico]** En session hijacking, el tráfico solo vuelve al atacante si se usa **source routing**. Sin source routing, el tráfico va al destino legítimo.

⚠️ **[SSL/PPTP impide hijacking]** Si la sesión usa SSL o PPTP, el hijacking es imposible porque el atacante no puede participar en el intercambio de claves.

⚠️ **[ISN + 1 para hijack local]** Para hijack local, el atacante predice el NSN como **ISN + 1** envenenando los cachés ARP de ambos hosts legítimos. Dato específico del examen.

⚠️ **[Session Spoofing usa credenciales capturadas; Hijacking usa session ID activo]** El examen puede confundir ambos. Spoofing = credenciales robadas + nueva sesión. Hijacking = session ID activo + sesión existente.

---

## 3. Nemotécnicos

### 6 factores de éxito del session hijacking — "No-Weak-Insecure-Indefinite-TCP-No-Enc"
**"No Weak IS, Indefinite TCP, No Encryption"**
- **No** account lockout
- **Weak** session ID algorithm / short IDs
- **Insecure** session ID handling
- **Indefinite** session timeout
- **TCP/IP** vulnerable by design
- **No** encryption for countermeasures

### 3 fases del proceso — "T-D-I"
**"Tracks, Desync, Injects"**
- **T**racking the connection
- **D**esynchronizing the connection
- **I**njecting the attacker's packet

### Condiciones para hijack TCP no cifrado — "T-S-M"
- Tráfico **T**CP sin cifrar
- Capacidad de leer **S**EQ/ACK (NSN)
- Capacidad de falsificar **M**AC o IP

### Spoofing vs. Hijacking — regla de bolsillo
**"Spoof = Nueva sesión con credenciales robadas. Hijack = Sesión activa robada"**

---

## 4. Flashcards

**Q:** ¿Por qué la autenticación al inicio de la sesión TCP hace posible el session hijacking?
**A:** Porque la autenticación solo se verifica al inicio. Una vez establecida la sesión, el atacante puede tomarla sin necesidad de autenticarse, ya que el servidor no vuelve a verificar la identidad en cada paquete.

**Q:** ¿Cuáles son las 3 fases del proceso de session hijacking?
**A:** 1) Tracking the Connection (sniffar SEQ/ACK). 2) Desynchronizing the Connection (alterar SEQ/ACK del servidor con datos nulos o RST+SYN). 3) Injecting the Attacker's Packet (inyectar datos o actuar como MITM).

**Q:** ¿Qué es un ACK Storm y cuándo se produce?
**A:** Un bucle interminable de paquetes ACK que revela el ataque. Se produce cuando el atacante usa un flag FIN en lugar de RST, porque el servidor responde con ACK, creando un ciclo. Un solo paquete perdido lo termina (no hay retransmisión porque los paquetes no llevan datos).

**Q:** ¿Cuál es la diferencia fundamental entre Session Hijacking y Session Spoofing?
**A:** Hijacking toma el control de una sesión activa ya existente usando el session ID en curso. Spoofing inicia una nueva sesión usando credenciales robadas de la víctima.

**Q:** ¿Cuáles son los dos niveles de session hijacking en el modelo OSI?
**A:** Network-Level (intercepción de paquetes TCP/UDP) y Application-Level (control de sesiones HTTP mediante session IDs).

**Q:** ¿Cuál es la diferencia entre passive y active session hijacking?
**A:** Passive: el atacante solo observa y registra el tráfico (sniffing). Active: el atacante toma control de la sesión existente (MITM, inyección).

**Q:** ¿Por qué el blind hijacking no puede usar ARP cache poisoning?
**A:** Porque los routers no hacen broadcast ARP por Internet, por lo que no existe una ruta para que los paquetes vuelvan a la IP del atacante.

**Q:** ¿Qué tres requisitos son necesarios para hijackear una sesión TCP no cifrada?
**A:** Tráfico TCP sin cifrar · capacidad de reconocer SEQ/ACK y predecir el NSN · capacidad de falsificar MAC o IP para recibir comunicaciones.

**Q:** ¿Cómo previenen SSL y PPTP el session hijacking?
**A:** Porque cifran la sesión completa; el atacante no puede participar en el intercambio de claves (key exchange) y por tanto no puede descifrar ni inyectar tráfico.

**Q:** ¿Qué hace el atacante para predecir el NSN en un hijack local?
**A:** Sniffa el tráfico, encuentra un paquete ACK, calcula el NSN como ISN + 1 y envenena los cachés ARP de ambos hosts legítimos para redirigir el tráfico hacia su sistema.

**Q:** ¿Qué factor de éxito del session hijacking está relacionado con la opción "remember me"?
**A:** Indefinite session timeout — los session IDs sin fecha de expiración pueden ser explotados indefinidamente, y la opción "remember me" es un ejemplo directo de este mecanismo.

**Q:** ¿Por qué los atacantes prefieren el network-level hijacking sobre el application-level?
**A:** Porque no requiere modificar el ataque por cada aplicación web; ataca el flujo de datos del protocolo compartido por todas las aplicaciones.

---

## 5. Confusión frecuente

### Session Hijacking vs. Session Spoofing
- **Hijacking**: sesión existente y activa + session ID en uso → toma del control; requiere desplazar al usuario legítimo; más difícil.
- **Spoofing**: nueva sesión + credenciales robadas → inicia conexión nueva; no requiere sesión previa; más sencillo.
- **Criterio**: ¿había una sesión activa en progreso? Sí → Hijacking. No → Spoofing.

### Passive Hijacking vs. Active Hijacking
- **Passive**: solo observa (sniffing de passwords); no interfiere en la sesión; contramedida: one-time passwords, Kerberos.
- **Active**: toma el control (MITM, inyección, desincronización); contramedida: cifrado de sesión completa.
- **Criterio**: ¿el atacante interfiere en la sesión o solo la observa? Interfiere → Active. Solo observa → Passive.

### Network-Level vs. Application-Level Hijacking
- **Network-Level**: protocolo TCP/UDP; independiente de la aplicación; proporciona base para ataques de nivel superior.
- **Application-Level**: protocolo HTTP; session IDs específicos de la aplicación; permite crear nuevas sesiones no autorizadas.
- **Criterio**: ¿el ataque opera sobre paquetes TCP/UDP? → Network-Level. ¿Opera sobre session IDs HTTP? → Application-Level.

### ACK Storm vs. desincronización normal
- **Desincronización normal** (RST): el servidor cierra la conexión y abre una nueva; el atacante controla la sesión sin revelar el ataque.
- **ACK Storm** (FIN): genera un bucle de ACKs que revela el ataque por exceso de tráfico de red.
- **Criterio**: ¿se usa RST para desincronizar? → Sin ACK storm. ¿Se usa FIN? → ACK storm + ataque revelado.
