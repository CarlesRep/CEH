# M10_05_DoSDDoSCountermeasures.md
## CEH v13 — Módulo 10 | DoS/DDoS Attack Countermeasures

---

## 1. Conceptos y definiciones

### 🔴 Técnicas de detección — 3 tipos

| Técnica | Mecanismo | Indicador de ataque |
|---|---|---|
| **Activity Profiling** | Analiza la tasa media de paquetes por flujo de red (flujos con cabeceras similares: IP origen/destino, puertos, protocolos) | Aumento del nivel de actividad en clusters + aumento del número de clusters distintos (DDoS). Usa **cálculo de entropía** para medir aleatoriedad; si aumenta la entropía → posible ataque |
| **Sequential Change-Point Detection** | Filtra tráfico por IP, puertos y protocolos; almacena flujo en gráfico tasa vs. tiempo | Cambio drástico en la tasa de flujo de tráfico; usa el algoritmo **CUSUM** (Cumulative Sum); también detecta actividades de escaneo de gusanos de red |
| **Wavelet-Based Signal Analysis** | Analiza tráfico en términos de **componentes espectrales** (frecuencias); separa cada ventana espectral | Componentes de alta frecuencia no habituales → tráfico anómalo. Tráfico normal = baja frecuencia. Durante un ataque, aumentan los componentes de alta frecuencia |

---

### Estrategias de contramedida DoS/DDoS — 3 niveles de respuesta

| Estrategia | Descripción | Desventaja |
|---|---|---|
| **Absorbing the Attack** | Capacidad adicional para absorber el ataque; requiere planificación previa | Coste de recursos adicionales incluso cuando no hay ataques |
| **Degrading Services** | Mantener solo los servicios críticos operativos; identificar servicios críticos y recortar los no críticos | No garantiza disponibilidad total |
| **Shutting Down Services** | Apagar todos los servicios hasta que el ataque cese | Opción extrema; puede no ser la ideal |

---

### 🔴 Contramedidas DDoS — Marco de 6 objetivos

1. **Protect Secondary Victims** — evitar que los zombies participen en el ataque.
2. **Detect and Neutralize Handlers** — identificar y deshabilitar los handlers del C&C.
3. **Prevent Potential Attacks** — Egress/Ingress filtering, TCP Intercept, Rate Limiting.
4. **Deflect Attacks** — honeypots.
5. **Mitigate Attacks** — Load Balancing, Throttling, Drop Requests.
6. **Post-Attack Forensics** — Traffic Pattern Analysis, Packet Traceback, Event Log Analysis.

---

### Protect Secondary Victims

- Instalar y actualizar antivirus y anti-troyanos.
- Aplicar parches de software regularmente.
- Deshabilitar servicios innecesarios y desinstalar aplicaciones en desuso.
- Escanear todos los ficheros recibidos de fuentes externas.
- Configurar y actualizar los mecanismos defensivos integrados en hardware/software.
- Los ISPs pueden usar **dynamic pricing** para incentivar a los usuarios a proteger sus sistemas.

---

### Detect and Neutralize Handlers

- Analizar protocolos de comunicación y patrones de tráfico entre handlers, clientes y agentes.
- Identificar nodos infectados por handlers → deshabilitarlos.
- Razonamiento clave: el número de **handlers es mucho menor que el número de agentes** → neutralizar pocos handlers puede inutilizar muchos agentes.
- Identificar **IPs de origen spoofadas** que no corresponden a subredes válidas.

---

### 🔴 Prevent Potential Attacks — Técnicas específicas

#### Egress Filtering
- Escanea las cabeceras de los paquetes IP que **salen** de la red.
- Requiere que los paquetes legítimos que salgan tengan una IP de origen cuya porción de red coincida con la red interna.
- Evita que el servidor establezca conexiones de vuelta al atacante (especialmente útil en zero-day attacks).
- Restringe el tráfico saliente al necesario → limita la capacidad del atacante de conectarse a otros sistemas.

#### Ingress Filtering
- Técnica de filtrado de paquetes usada por **ISPs** para prevenir el **source address spoofing** del tráfico entrante.
- Hace el tráfico de Internet trazable hasta su origen real.
- Protege contra flooding attacks que se originan desde prefijos IP válidos.

#### TCP Intercept *(Cisco IOS)*
- Protege servidores TCP contra **SYN flooding** actuando como intermediario.
- **Modo intercept (por defecto)**: el router intercepta los paquetes SYN, responde con SYN-ACK en nombre del servidor, espera el ACK del cliente y luego conecta con el servidor real. Combina las dos medias conexiones transparentemente.
- **Modo watch (pasivo)**: las peticiones pasan al servidor pero si no se establece la conexión en **30 segundos**, el software envía un reset al servidor.

**Comandos Cisco IOS para activar TCP Intercept**:
```
Step 1: access-list <number> {deny|permit} tcp any <destination> <wildcard>
Step 2: ip tcp intercept list <access-list-number>
Modo:   ip tcp intercept mode {intercept | watch}
```
La access list identifica la dirección del servidor/red a proteger. La fuente se define como "any" (no importa desde dónde).

#### Rate Limiting
- Controla la tasa de tráfico entrante/saliente en una interfaz de red.
- Especialmente importante en appliances hardware.
- Se configura para limitar la tasa de peticiones en las **capas 4 y 5 del modelo OSI**.

---

### Deflect Attacks — Honeypots

- Sistemas con seguridad limitada intencionalmente para atraer a atacantes DDoS.
- Permiten capturar información sobre atacantes, técnicas y herramientas.
- Los atacantes instalan handlers o código de agente en el honeypot → protegen los sistemas reales.
- Se usa un enfoque **defense-in-depth con IPsec** para desviar tráfico DoS sospechoso hacia honeypots.
- **Tipos**: low-interaction honeypots y high-interaction honeypots.
- **Honeynet**: ejemplo de high-interaction honeypot; simula la topología completa de una red con equipos y aplicaciones reales.
- **Blumira**: honeypot software que detecta acceso no autorizado y movimiento lateral; bloquea la IP origen a nivel de switch o firewall en cuanto detecta un evento.

---

### Mitigate Attacks

| Técnica | Mecanismo |
|---|---|
| **Load Balancing** | Aumentar ancho de banda en conexiones críticas; usar modelo de servidores replicados para distribuir carga y aumentar failsafe |
| **Throttling** | Configurar routers con lógica para regular el tráfico entrante a niveles seguros para el servidor; usa "Min-max fair server-centric router"; puede generar falsos positivos o dejar pasar tráfico malicioso |
| **Drop Requests** | Descartar paquetes cuando aumenta la carga; el sistema induce al solicitante a resolver un puzzle difícil (memoria/CPU intensa) antes de continuar → degrada el rendimiento de los zombies |

---

### Post-Attack Forensics

| Técnica | Función |
|---|---|
| **Traffic Pattern Analysis** | Almacena datos post-ataque; identifica características únicas del tráfico de ataque; mejora los filtros de load balancing y throttling; ayuda a evitar que el servidor sea usado como plataforma DDoS |
| **Packet Traceback** | Traza el paquete de ataque hacia atrás hasta su fuente real (reverse engineering); permite desarrollar técnicas de filtrado y bloqueo de futuros ataques |
| **Event Log Analysis** | Análisis forense mediante logs de routers, firewalls e IDS; ayuda a identificar el tipo de ataque y rastrear la IP del atacante con ayuda de ISPs y fuerzas del orden |

---

### 🔴 Técnicas de defensa contra botnets — 4 técnicas

| Técnica | Mecanismo |
|---|---|
| **RFC 3704 Filtering** | ACL básico que bloquea tráfico con IPs spoofadas; usa una **"bogon list"** (IPs no asignadas o reservadas que no deberían proceder de Internet); si la IP de origen está en la bogon list → el paquete es de origen spoofado → se descarta. La bogon list cambia regularmente |
| **Cisco IPS Source IP Reputation Filtering** | Usa **Cisco Global Correlation** (Cisco IPS 7.0) + **Cisco SensorBase Network** (base de datos de amenazas: botnets, malware, dark nets, botnet harvesters); filtra tráfico DoS antes de que alcance activos críticos |
| **Black Hole Filtering** | Nodos de red donde el tráfico entrante se **descarta sin notificar al origen**; usa **RTBH (Remotely Triggered Black Hole) filtering** con el ISP; emplea rutas BGP host para redirigir tráfico a un next hop "null0" |
| **ISP/Cloud DDoS Prevention** | El ISP "scruba/limpia" el tráfico antes de permitir que entre al enlace del usuario; el servicio opera en la nube → los ataques no saturan los enlaces. Cisco: **IP Source Guard** filtra tráfico basado en DHCP snooping binding database |

---

### Protección DoS/DDoS a nivel de ISP

- Los ISPs ofrecen un SLA de **"clean pipes"**: garantizan un ancho de banda de tráfico genuino, no de todo el tráfico.
- Muchos ISPs bloquean todo el tráfico durante un DDoS (incluido el legítimo) si no tienen clean-pipes.
- Si el ISP no ofrece clean-pipes → usar **subscription services de cloud providers** que filtran y pasan solo conexiones de confianza.
- Vendors: **Imperva** y **VeriSign** para protección cloud contra DoS.
- Los administradores pueden pedir al ISP que bloquee la IP afectada y mover el sitio a otra IP tras **DNS propagation**.

---

### Herramientas y appliances de protección DDoS

| Herramienta / Servicio | Descripción clave |
|---|---|
| **Anti DDoS Guardian** | Monitoriza cada paquete entrante/saliente en tiempo real; limita flujo de red, ancho de banda de cliente, conexiones TCP concurrentes, tasa de conexión TCP, ancho de banda UDP, tasa de conexión UDP y tasa de paquetes UDP |
| **FortiDDoS** | Arquitectura ML masivamente paralela; inspecciona paquetes L3, L4 y L7 inbound y outbound de menor tamaño; latencia mínima |
| **Quantum DDoS Protector (Check Point)** | Protección multicapa; behavioral baselining; signatures automáticas; challenge/response; respuesta en segundos; integrado con Check Point Security Management |
| **Huawei AntiDDoS1000** | Big Data analytics; modela **60+ tipos** de tráfico de red; defiende contra **100+ tipos de ataques**; respuesta en segundos; in-line mode |
| **Cloudflare** | Red de **100 Tbps**; bloquea media de **87 billion threats/día**; mitigación en **3 segundos**; protección BGP + integración L7 |
| **Akamai DDoS Protection** | Detiene ataques en la nube antes de que alcancen aplicaciones/data centers; DNS siempre disponible |
| **A10 Thunder TPS** | Detecta y bloquea DDoS y otras ciberamenazas antes de que causen outages |

---

## 2. Exam Traps ⚠️

⚠️ **[CUSUM — Sequential Change-Point Detection]** El algoritmo específico de Sequential Change-Point Detection es **CUSUM (Cumulative Sum)**. El examen puede preguntar qué algoritmo usa esta técnica.

⚠️ **[Wavelet — alta frecuencia = ataque]** Tráfico normal = baja frecuencia. Durante un ataque, aumentan los **componentes de alta frecuencia**. El examen puede invertir esta lógica como trampa.

⚠️ **[Activity Profiling — entropía]** El método de **cálculo de entropía** mide la aleatoriedad en los niveles de actividad. Más entropía → más sospechoso. El examen puede preguntar qué calcula este método.

⚠️ **[TCP Intercept modo watch — 30 segundos]** En el modo watch pasivo, el timeout es **30 segundos**. No confundir con los 75 segundos del listen queue en SYN flood.

⚠️ **[TCP Intercept — modo por defecto: intercept (activo)]** El modo por defecto de TCP Intercept en Cisco IOS es el **modo intercept (activo)**, no el watch (pasivo).

⚠️ **[Egress vs. Ingress Filtering]** Egress filtra paquetes que **salen** de la red interna. Ingress filtra paquetes que **entran** (usado por ISPs para prevenir spoofing). El examen los confunde deliberadamente.

⚠️ **[Black Hole Filtering — RTBH + BGP + null0]** Los tres elementos técnicos: RTBH (Remotely Triggered Black Hole) · BGP host routes · next hop "null0". El examen puede preguntar el protocolo de enrutamiento usado → BGP.

⚠️ **[RFC 3704 — bogon list]** La bogon list contiene IPs no asignadas o reservadas. Un paquete con IP de origen en la bogon list → IP spoofada → se descarta. La bogon list cambia regularmente.

⚠️ **[Cisco SensorBase — botnets, malware, dark nets, botnet harvesters]** La base de datos que usa Cisco Global Correlation es **Cisco SensorBase Network**. El examen puede preguntar el nombre específico.

⚠️ **[Huawei AntiDDoS1000 — 60+ tipos de tráfico, 100+ tipos de ataques]** Datos numéricos específicos que el examen puede preguntar directamente.

⚠️ **[Cloudflare — 100 Tbps, 87 billion threats/day, 3 segundos]** Tres datos numéricos del servicio Cloudflare presentes en el libro.

⚠️ **[Honeypot — high-interaction = honeynet]** El ejemplo específico de high-interaction honeypot es el **honeynet**, que simula la topología completa de una red con equipos y aplicaciones reales.

⚠️ **[Throttling — falsos positivos]** La limitación principal de Throttling es que puede generar **falsos positivos** (dejar pasar tráfico malicioso o bloquear tráfico legítimo).

---

## 3. Nemotécnicos

### 3 técnicas de detección — "A-S-W"
**"Activity Sequentially Wavelets"**
- **A**ctivity Profiling → entropía
- **S**equential Change-Point Detection → CUSUM
- **W**avelet-Based Signal Analysis → componentes espectrales / alta frecuencia

### 6 objetivos de contramedida — "P-D-P-D-M-P"
**"Protect Detect Prevent Deflect Mitigate Post-forensics"**
- **P**rotect secondary victims → **D**etect handlers → **P**revent (egress/ingress/TCP/rate) → **D**eflect (honeypots) → **M**itigate (LB/throttle/drop) → **P**ost-attack forensics

### 4 técnicas anti-botnet — "RFC-CISCO-BLACK-ISP"
- **RFC** 3704 Filtering (bogon list)
- **CISCO** IPS Source IP Reputation (SensorBase)
- **BLACK** Hole Filtering (RTBH + BGP + null0)
- **ISP**/Cloud DDoS Prevention (clean pipes / IP Source Guard)

### TCP Intercept modos — "Intercept activo 30s watch"
- **Intercept** (por defecto): activo, el router completa el handshake.
- **Watch** (pasivo): pasivo, timeout de **30 segundos**.

### Cloudflare datos numéricos — "100T-87B-3s"
- **100** Tbps red · **87** billion threats/día · **3** segundos de mitigación

---

## 4. Flashcards

**Q:** ¿Qué algoritmo usa la técnica Sequential Change-Point Detection y qué calcula?
**A:** Algoritmo CUSUM (Cumulative Sum). Calcula desviaciones en la tasa de tráfico real versus la media local esperada en la serie temporal de tráfico.

**Q:** ¿Qué indica un aumento de los componentes de alta frecuencia en el análisis Wavelet-Based?
**A:** Tráfico anómalo / posible ataque DDoS. El tráfico normal es de baja frecuencia; durante un ataque aumentan los componentes de alta frecuencia.

**Q:** ¿Qué mide el cálculo de entropía en Activity Profiling?
**A:** La aleatoriedad en los niveles de actividad de la red. Un aumento de entropía indica que la red puede estar bajo ataque.

**Q:** ¿Cuál es el modo por defecto de TCP Intercept en Cisco IOS?
**A:** El modo intercept (activo). El router intercepta SYNs y completa el handshake en nombre del servidor.

**Q:** ¿Cuál es el timeout en el modo watch pasivo de TCP Intercept?
**A:** 30 segundos. Si la conexión no se establece en ese tiempo, el software envía un reset al servidor.

**Q:** ¿Cuál es la diferencia entre Egress Filtering e Ingress Filtering?
**A:** Egress filtra los paquetes que **salen** de la red interna (evita que salgan IPs spoofadas). Ingress filtra los paquetes que **entran**, usado por ISPs para prevenir source address spoofing.

**Q:** ¿Qué es la "bogon list" en el contexto de RFC 3704 Filtering?
**A:** Lista de todas las IPs no asignadas o reservadas que no deberían proceder de Internet. Un paquete con IP de origen en la bogon list es de fuente spoofada y debe descartarse.

**Q:** ¿Qué protocolo de enrutamiento usa Black Hole Filtering para redirigir tráfico a null0?
**A:** BGP (Border Gateway Protocol) — rutas host BGP que dirigen el tráfico al next hop "null0".

**Q:** ¿Cuántos tipos de tráfico de red modela y cuántos tipos de ataques defiende Huawei AntiDDoS1000?
**A:** 60+ tipos de tráfico de red; defiende contra 100+ tipos de ataques.

**Q:** ¿Cuáles son los tres datos numéricos clave del servicio Cloudflare DDoS Protection?
**A:** Red de 100 Tbps · bloquea 87 billion threats/día · mitigación en 3 segundos.

**Q:** ¿Qué es la estrategia "Absorbing the Attack" y cuál es su principal desventaja?
**A:** Usar capacidad adicional para absorber el ataque (requiere planificación previa). Desventaja: coste de los recursos adicionales incluso cuando no hay ataques en curso.

**Q:** ¿Por qué neutralizar handlers es especialmente eficiente para detener un ataque DDoS?
**A:** Porque el número de handlers es mucho menor que el número de agentes → neutralizar pocos handlers puede inutilizar muchos agentes simultáneamente.

**Q:** ¿Cuál es la limitación principal del Throttling como contramedida DDoS?
**A:** Puede generar falsos positivos: dejar pasar tráfico malicioso o descartar tráfico legítimo.

**Q:** ¿Qué diferencia un honeynet de un honeypot estándar?
**A:** Un honeynet es un high-interaction honeypot que simula la topología completa de una red con equipos y aplicaciones reales. Un honeypot estándar puede ser low-interaction (simulación parcial).

**Q:** ¿Qué hace IP Source Guard en dispositivos Cisco en el contexto de la defensa DDoS?
**A:** Filtra el tráfico basándose en la DHCP snooping binding database o en IP source bindings para evitar que los bots envíen paquetes spoofados.

---

## 5. Confusión frecuente

### Egress Filtering vs. Ingress Filtering
- **Egress**: filtra paquetes que **salen** de la red interna; evita que IPs internas spoofadas lleguen a Internet; protege a las víctimas externas de ataques lanzados desde dentro.
- **Ingress**: filtra paquetes que **entran** desde Internet; usado por ISPs; hace el tráfico trazable.
- **Criterio**: ¿el filtro es para tráfico saliente? → Egress. ¿Para tráfico entrante en el ISP? → Ingress.

### TCP Intercept modo intercept vs. modo watch
- **Intercept (activo, por defecto)**: el router completa el handshake con el cliente antes de conectar al servidor; los paquetes falsos nunca llegan al servidor.
- **Watch (pasivo)**: las peticiones pasan al servidor; si no se establece en **30 segundos** → reset.
- **Criterio**: ¿el router completa el handshake en nombre del servidor? → Intercept. ¿Las peticiones llegan al servidor con monitorización? → Watch.

### Activity Profiling vs. Sequential Change-Point vs. Wavelet
- **Activity Profiling**: analiza **tasas de paquetes por flujo**; usa **entropía**.
- **Sequential Change-Point**: analiza **tasas de tráfico en el tiempo**; usa **CUSUM**; detecta worms.
- **Wavelet**: analiza **componentes espectrales/frecuencias**; alta frecuencia = anómalo.
- **Criterio**: ¿el análisis es sobre flujos y aleatoriedad? → Activity Profiling. ¿Sobre cambios en tasas temporales? → Sequential. ¿Sobre frecuencias espectrales? → Wavelet.

### Black Hole Filtering vs. Rate Limiting
- **Black Hole**: descarta el tráfico sin notificar al origen; usa BGP + null0; operación remota con ISP.
- **Rate Limiting**: no descarta todo el tráfico; limita la tasa a niveles seguros en las capas 4 y 5 del OSI; permite cierto tráfico.
- **Criterio**: ¿el tráfico se descarta completamente? → Black Hole. ¿Se limita la velocidad pero se permite algo? → Rate Limiting.
