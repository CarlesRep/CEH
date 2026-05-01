# M12_01 — IDS, IPS and Firewall Concepts
> Módulo 12 / Subapartado 1 — IDS, IPS, and Firewall Concepts

---

## 1. Conceptos y definiciones

### IDS — Intrusion Detection System

El IDS monitoriza tráfico entrante y saliente buscando coincidencias con firmas conocidas de ataques o comportamientos anómalos. Cuando detecta una coincidencia, lanza una alerta al equipo de seguridad. Es un sistema **pasivo**: detecta pero no bloquea. Opera como sniffer de paquetes, capturando e inspeccionando paquetes a nivel TCP/IP.

**Posicionamiento en red:** La práctica recomendada es **layered defense**: un IDS delante del firewall (monitoriza amenazas externas antes de que lleguen al firewall) y otro detrás (detecta tráfico que lo haya superado). Si sólo se despliega uno, cerca del DMZ es el emplazamiento preferido.

**Flujo de decisión del IDS:**
1. El sensor captura el paquete.
2. Se compara contra la base de firmas → si coincide, se ejecutan acciones predefinidas (terminar conexión, bloquear IP, descartar paquete, lanzar alarma) y **se salta la detección de anomalías**.
3. Si no coincide con ninguna firma, el sensor analiza patrones de tráfico buscando anomalías.
4. Si pasa todos los controles, el paquete se reenvía a la red.

---

### IPS — Intrusion Prevention System

El IPS es un IDS **activo**: detecta **y** previene intrusiones. Se coloca **inline** en la red (entre origen y destino), lo que le permite descartar paquetes maliciosos en tiempo real sin intervención humana. Suele situarse **detrás del firewall** como capa adicional. Opera con reglas y políticas configuradas que determinan qué tráfico bloquear, registrar o permitir.

| Característica | IDS | IPS |
|---|---|---|
| Modo de operación | Pasivo | Activo (inline) |
| Puede bloquear tráfico | No | Sí |
| Posición en red | Fuera del flujo directo | En línea entre origen y destino |
| Genera alertas | Sí | Sí |
| Registra logs en tiempo real | Sí | Sí |
| Previene ataques directos | No | Sí |

---

### 🔴 Métodos de detección de intrusiones

#### 1. Signature Recognition (Misuse Detection)
Compara paquetes entrantes/salientes contra **firmas binarias** de ataques conocidos. Sólo detecta ataques conocidos. Los problemas clave son:
- Requiere una cantidad masiva de firmas; a más firmas, mayor riesgo de falsos positivos y mayor consumo de ancho de banda.
- Cambiar un solo bit en una cadena de ataque invalida la firma — requiere firmas enteramente nuevas.
- Ejemplos actuales que requieren múltiples firmas por ataque: **URSNIF** y **VIRLOCK**.
- Un paquete inofensivo con la misma firma puede generar un **falso positivo**.

#### 2. Anomaly Detection (Not-Use Detection)
Establece una línea base del comportamiento normal del tráfico. Cualquier evento que supere el **umbral de tolerancia** se considera ataque. El mayor reto es construir un modelo preciso de tráfico "normal" dado que existe variabilidad estadística inherente en redes reales. Puede detectar ataques desconocidos (zero-day) pero genera más falsos positivos que la detección por firma.

#### 3. Protocol Anomaly Detection
Analiza el tráfico en busca de desviaciones respecto a los estándares de protocolo definidos. Flujo de trabajo:
1. **Baseline:** aprender la estructura, secuencia, tiempos y contenido esperado del tráfico de protocolo.
2. **Anomaly identification:** detectar estructuras de paquetes inusuales, órdenes de secuencia inesperados, tiempos de respuesta anómalos, violaciones de protocolo.
3. **Detection rules:** reglas basadas en especificaciones del protocolo que guían la detección.

---

### 🔴 Tipos de alertas IDS

| Tipo | Ataque real | Alerta generada | Significado |
|---|---|---|---|
| **True Positive** | ✅ Sí | ✅ Sí | IDS detecta correctamente un ataque real |
| **False Positive** | ❌ No | ✅ Sí | IDS alerta sin ataque real — actividad legítima clasificada como ataque |
| **False Negative** | ✅ Sí | ❌ No | IDS no detecta un ataque real — **el fallo más peligroso** |
| **True Negative** | ❌ No | ❌ No | IDS ignora correctamente actividad legítima — comportamiento esperado |

---

### Indicadores generales de intrusión

**Indicadores en sistema de ficheros:**
- Nuevos ficheros o programas desconocidos
- Cambios en permisos de ficheros (ej. read-only → write tras escalada de privilegios)
- Modificaciones inexplicables en tamaño, propietario o permisos
- Presencia de ficheros **suid/sgid** en Linux no presentes en la lista maestra
- Ficheros ejecutables con extensiones dobles o extrañas
- Ficheros desaparecidos
- Consumo anómalo de espacio en disco
- Reducción de ancho de banda disponible por consumo de recursos

**Indicadores de red:**
- Aumento repentino de consumo de ancho de banda
- Sondeos repetidos de servicios disponibles
- Solicitudes de conexión desde IPs fuera del rango de red
- Intentos de login repetidos desde hosts remotos
- Influx repentino de datos de log (posible DoS/DDoS)
- Cambios inesperados en configuración de red o reglas de firewall
- Conexiones salientes inusuales hacia dominios maliciosos

**Indicadores de sistema:**
- Logs cortos, incompletos, con permisos incorrectos o desaparecidos
- Rendimiento inusualmente lento
- Modificaciones en software del sistema o ficheros de configuración
- Procesos no familiares en ejecución
- Instalación de software no autorizado
- Presencia de artefactos de herramientas de atacante (shell history files, ficheros temporales)
- Gaps en la contabilidad del sistema (system accounting)

---

### Tipos de IDS

**NIDS — Network-Based IDS:**
Captura e inspecciona todo el tráfico de red. Opera en modo promiscuo. Genera alertas a nivel IP o aplicación. Más distribuido que el HIDS. Identifica anomalías a nivel de router y host. Asigna un nivel de amenaza a cada paquete malicioso. Detecta DoS, port scans, intentos de intrusión. Se coloca como "caja negra" en la red.

**HIDS — Host-Based IDS:**
Analiza el comportamiento de un sistema individual. Instalable en cualquier sistema (PC de escritorio hasta servidor). Más versátil que el NIDS. Eficaz para detectar modificaciones de ficheros no autorizadas y amenazas internas. Más centrado en Windows aunque existen versiones UNIX. Genera mayor overhead al monitorizar cada evento del sistema. No muy común precisamente por ese overhead.

| | NIDS | HIDS |
|---|---|---|
| Scope | Red completa | Sistema individual |
| Detecta modificaciones de ficheros | Limitado | Sí |
| Overhead | Bajo | Alto |
| Modo de escucha | Promiscuo | Eventos de sistema |
| Distribución | Mayor | Menor |

---

### Firewall

Sistema software o hardware en el gateway de red que protege recursos privados de acceso no autorizado. Examina todo el tráfico entrante y saliente y bloquea lo que no cumple los criterios de seguridad definidos. Opera en la unión entre dos redes (típicamente privada e Internet).

**Funciones clave:**
- Filtrado de paquetes por dirección IP origen/destino y tipo de tráfico
- Enrutamiento de datos entre redes según reglas
- Logging de todos los intentos de acceso para auditoría
- Actuación como "phone tap" activo para detectar intentos de intrusión a módems
- Configuración para restringir servicios específicos (POP, SMTP, FTP, Telnet)

**Limitaciones del firewall:**
- No previene ataques internos (backdoors, empleados maliciosos)
- No protege contra ingeniería social ni ataques data-driven (malicious links/emails)
- No detecta dispositivos externos ya infectados conectados a la red (laptops, USB)
- Incapaz de proteger contra zero-day de forma completa
- Crea un punto único de fallo y puede generar bottleneck
- No es sustituto de antivirus/antimalware
- No bloquea ataques a capas superiores del protocolo stack
- No previene ataques desde puertos/aplicaciones comunes
- No bloquea ataques desde conexiones dial-in
- No entiende tráfico tunnelizado

---

### 🔴 Arquitectura de Firewall

#### Bastion Host
Sistema diseñado y configurado específicamente para defender la red. Actúa de mediador entre redes interna y externa. Dos interfaces:
- **Interfaz pública:** conectada directamente a Internet
- **Interfaz privada:** conectada a la intranet

#### Screened Subnet (DMZ)
Red protegida creada con un firewall de dos o tres interfaces (two/three-homed) detrás de un screening firewall. En un **three-homed firewall**:
- Interfaz 1 → Internet
- Interfaz 2 → DMZ
- Interfaz 3 → Intranet

La DMZ responde peticiones públicas sin permitir acceso a la intranet. **Vulnerabilidad crítica:** si el firewall three-homed se ve comprometido, tanto DMZ como intranet quedan expuestas. Solución más segura: múltiples firewalls separados (Internet ↔ DMZ ↔ Intranet).

#### Multi-homed Firewall
Nodo con múltiples NICs conectado a dos o más redes. Cada interfaz se conecta a un segmento de red distinto (lógica y físicamente). Más de tres interfaces permite subdivisiones adicionales según objetivos de seguridad. El modelo más seguro es el **back-to-back firewall**.

#### DMZ
Zona neutra entre red interna (trusted) y red externa (untrusted). Actúa como buffer. Servicios colocables en DMZ: email, web, FTP. **No colocar en DMZ:** servidores web que se comunican con servidores de bases de datos (darían acceso directo a datos sensibles desde el exterior).

---

### 🔴 Tipos de Firewall por mecanismo de trabajo

#### Packet Filtering Firewall
- Opera en: **capa de red OSI** / **capa internet TCP/IP**
- Compara cada paquete contra un conjunto de criterios antes de reenviar
- Analiza: IP origen, IP destino, puerto TCP/UDP origen, puerto TCP/UDP destino, TCP flag bits, protocolo, dirección, interfaz
- No mantiene estado — cada paquete se evalúa de forma independiente

#### Circuit-Level Gateway
- Opera en: **capa de sesión OSI** / **capa de transporte TCP/IP**
- No filtra paquetes individuales — permite o bloquea **flujos de datos (sesiones)**
- Verifica el **TCP handshake** para determinar si una sesión es válida
- El tráfico parece originarse desde el propio gateway (oculta IPs internas)
- Relativamente barato; oculta información de la red privada

#### Application-Level Firewall (Proxy)
- Opera en: **capa de aplicación OSI/TCP/IP**
- Filtra a nivel de aplicación: puede bloquear comandos específicos HTTP (GET/POST), FTP, Telnet, gopher
- Dos modos:
  - **Activo:** examina peticiones activamente contra vulnerabilidades conocidas (SQLi, XSS, cookie tampering) y rechaza las sospechosas
  - **Pasivo:** igual que activo pero no rechaza peticiones — funciona como IDS
- Puede detectar worms con código malicioso en protocolos legítimos (lo que los stateful firewalls no pueden)
- Característica adicional: **content-caching proxy** — cachea datos frecuentemente solicitados

#### Stateful Multilayer Inspection Firewall
- Combina packet filtering + circuit-level + application-level
- Filtra en capa de red para verificar paquetes de sesión Y evalúa contenido en capa de aplicación
- Puede recordar paquetes anteriores y tomar decisiones sobre futuros paquetes
- Realiza **deep packet inspection**
- Ejemplo: **Cisco PIX**
- Trackea y loguea slots/translations

#### Application Proxy
- Actúa como servidor proxy específico por servicio/protocolo
- Ejemplo: FTP proxy → sólo permite tráfico FTP, bloquea todo lo demás
- Interfaz entre workstation y red
- Tipos especiales: **caching proxy** (almacena copias de datos solicitados frecuentemente)
- Ventaja principal: **transparencia** — el usuario cree que habla con el servidor real; el servidor cree que habla directamente con el usuario

#### NAT (Network Address Translation)
- Separa direcciones IP en dos conjuntos (internas y externas)
- Oculta la topología de la red interna
- Modifica IPs origen/destino y puede cambiar puertos origen/destino
- Actúa como filtro: permite sólo conexiones iniciadas desde la red interna
- Esquemas de traducción:
  1. Mapeo fijo 1:1 interno↔externo (sin ahorro de espacio de direcciones)
  2. Asignación dinámica sin modificar puertos (limita hosts simultáneos al número de IPs externas disponibles)
  3. Mapeo fijo con **port mapping** (múltiples máquinas internas comparten una IP externa)
  4. Asignación dinámica de IP externa + par de puertos por conexión (uso más eficiente)
- **Desventaja clave:** interfiere con sistemas de cifrado y autenticación

#### VPN Firewall
- Proporciona acceso seguro a red privada a través de Internet mediante **encapsulación y cifrado**
- Principios de operación: cifrar tráfico → verificar integridad → encapsular nuevos paquetes → enviar → desencapsular → verificar integridad → descifrar
- Realiza cifrado/descifrado fuera del perímetro de packet-filtering para permitir inspección de paquetes
- VPN no es tecnología de firewall per se, pero los firewalls son convenientes para añadir características VPN
- **Desventaja:** el usuario es vulnerable a ataques en la red de destino

#### Next-Generation Firewall (NGFW)
- Combina: deep packet inspection + application awareness/control + IPS integrado + threat intelligence en la nube
- Opera en múltiples capas OSI simultáneamente
- Puede detectar y bloquear: zero-day exploits, ransomware, APTs
- Desventajas: más caro, más complejo de configurar, introduce latencia

---

### 🔴 Tabla OSI: Firewall Technologies por capa

| Capa OSI | Tecnologías |
|---|---|
| Aplicación | VPN, Application Proxies |
| Presentación | VPN |
| Sesión | VPN, Circuit-Level Gateways |
| Transporte | VPN, Packet Filtering |
| Red | VPN, NAT, Packet Filtering, Stateful Multilayer Inspection |
| Enlace de datos | VPN, Packet Filtering |
| Física | No aplicable |

---

## 2. Exam Traps ⚠️

⚠️ **[IDS vs IPS — pasivo vs activo]**
El examen puede presentar "active IDS" como sinónimo de IPS. Un IDS pasivo sólo detecta; un IPS (active IDS) detecta Y previene. Si la pregunta menciona "bloqueo" o "prevención inline", la respuesta es IPS.

⚠️ **[False Negative — el fallo más peligroso]**
El examen pregunta cuál es el tipo de alerta más peligroso. La respuesta es **False Negative** (ataque real sin alerta), no False Positive. Un FP genera ruido; un FN significa que el ataque pasa desapercibido.

⚠️ **[False Positive — efecto secundario]**
El examen puede preguntar el efecto de demasiados falsos positivos. La respuesta correcta: hacen a los usuarios **insensibles a las alarmas** (alarm fatigue), debilitando la reacción ante intrusiones reales.

⚠️ **[Anomaly Detection vs Signature Recognition — qué detecta cada uno]**
Signature recognition sólo detecta ataques **conocidos**. Anomaly detection puede detectar ataques **desconocidos** (zero-day) pero genera más falsos positivos. El examen puede preguntar cuál detecta zero-days: **anomaly detection**.

⚠️ **[URSNIF/VIRLOCK — relevancia para firmas]**
Estos malware han "driven the need for multiple signatures for a single attack". Si el examen pregunta por qué la detección por firmas es insuficiente en ataques modernos, la razón es que cambiar un solo bit invalida la firma — se necesitan firmas completamente nuevas.

⚠️ **[IDS placement — best practice]**
El examen puede ofrecer como opción "poner el IDS sólo dentro del firewall cerca del DMZ". Esto es subóptimo. La práctica recomendada es **layered defense: uno delante y otro detrás del firewall**.

⚠️ **[Circuit-Level Gateway — qué verifica]**
No filtra paquetes individuales. Verifica el **TCP handshake** para validar sesiones. El tráfico aparece como originado desde el gateway (no desde el host real). Opera en **capa de sesión OSI**.

⚠️ **[Application-Level Firewall activo vs pasivo]**
El examen puede confundir el modo pasivo con un IDS. **Modo pasivo:** examina peticiones y detecta vulnerabilidades pero **no las rechaza** (funciona como IDS). **Modo activo:** detecta Y rechaza. Si la pregunta menciona que "examina pero no bloquea", es modo pasivo de application-level firewall.

⚠️ **[Stateful Multilayer Inspection — qué combina]**
Combina los TRES tipos anteriores: packet filtering + circuit-level + application-level. El ejemplo canónico es **Cisco PIX**. No confundir con NGFW, que añade IPS integrado y threat intelligence.

⚠️ **[Three-homed firewall — vulnerabilidad]**
Si el three-homed firewall es comprometido, **tanto la DMZ como la intranet** quedan expuestas. La alternativa más segura es usar múltiples firewalls separados.

⚠️ **[NAT — limitación con cifrado]**
NAT interfiere con sistemas de **cifrado y autenticación**. Si el examen pregunta por qué NAT puede ser problemático en entornos con IPsec o similares, esta es la razón.

⚠️ **[Firewall — lo que NO puede hacer]**
El examen pregunta frecuentemente limitaciones del firewall. Recordar especialmente: NO previene ataques internos/backdoors, NO protege de ingeniería social, NO entiende tráfico tunnelizado, NO es sustituto de antivirus.

⚠️ **[DMZ — qué NO colocar]**
Servidores web que se comunican con bases de datos **NO deben estar en la DMZ**, pues darían a usuarios externos acceso directo a datos sensibles.

⚠️ **[Packet Filtering — capa OSI]**
Opera en capa de **red (OSI)** o capa **internet (TCP/IP)**. El examen puede confundir con capa de transporte porque analiza puertos TCP/UDP; pero los puertos son datos del header que se inspeccionan, no la capa en que opera el firewall.

⚠️ **[Signature-based IDS — rendimiento con muchas firmas]**
A mayor número de firmas en la base de datos, mayor riesgo de **dropping de paquetes**. El examen puede preguntar el efecto de una base de datos de firmas muy grande: degradación de rendimiento y posible descarte de paquetes.

---

## 3. Nemotécnicos

**Tipos de alertas IDS — 2×2 matrix "ATAQUE / ALERTA":**
```
         ALERTA SÍ    ALERTA NO
ATAQUE   True+        False-  ← MÁS PELIGROSO
NO ATAQ  False+       True-
```
Regla: **"False Negative = Fallo Fatal"** — el atacante pasa sin ser detectado.

**Capas OSI de los tipos de firewall (de más bajo a más alto):**
- **Red** → Packet Filtering
- **Sesión** → Circuit-Level Gateway (recuerda: "Circuit = Conexión = Sesión")
- **Aplicación** → Application-Level Firewall / Proxy

**Three-homed firewall — interfaces:**
**I-D-I**: Internet → DMZ → Intranet (de afuera hacia adentro, 1-2-3)

**Métodos de detección IDS — SPA:**
- **S**ignature recognition (firmas conocidas)
- **P**rotocol anomaly detection (desviaciones de protocolo)
- **A**nomaly detection (comportamiento fuera del umbral)

**Arquitectura firewall — BMS:**
- **B**astion Host (un sistema, dos interfaces)
- **M**ulti-homed (múltiples NICs, múltiples segmentos)
- **S**creened Subnet / DMZ (zona neutra entre dos redes)

**NAT esquemas de traducción — orden de eficiencia (menor a mayor):**
1. Mapeo fijo 1:1 (ningún ahorro)
2. Dinámico sin modificar puertos (limitado por IPs externas)
3. Port mapping con IP fija (varios internos, una IP externa)
4. Dinámico IP+puerto (máxima eficiencia)

---

## 4. Flashcards

**Q:** ¿Cuál es la diferencia fundamental entre un IDS y un IPS?
**A:** El IDS es pasivo — sólo detecta y alerta. El IPS es activo e inline — detecta y previene/bloquea tráfico malicioso en tiempo real.

---

**Q:** ¿Cuál es el tipo de alerta IDS más peligroso y por qué?
**A:** False Negative — el IDS no detecta un ataque real. El ataque pasa sin generar alerta, comprometiendo la red sin que el equipo de seguridad lo sepa.

---

**Q:** ¿Qué ocurre en el flujo de detección del IDS cuando una firma coincide con un paquete?
**A:** Se ejecutan acciones predefinidas (terminar conexión, bloquear IP, descartar paquete, lanzar alarma) y se salta la detección de anomalías.

---

**Q:** ¿Qué método de detección IDS puede detectar ataques zero-day?
**A:** Anomaly detection — detecta cualquier comportamiento que supere el umbral de tolerancia del tráfico normal, incluyendo ataques desconocidos.

---

**Q:** ¿Por qué URSNIF y VIRLOCK son relevantes para la detección por firmas?
**A:** Porque han generado la necesidad de múltiples firmas por un solo ataque. Cambiar un solo bit en una cadena de ataque invalida la firma existente y requiere firmas completamente nuevas.

---

**Q:** ¿En qué capa OSI opera un Circuit-Level Gateway y qué verifica?
**A:** Capa de sesión (OSI). Verifica el TCP handshake entre paquetes para determinar si una sesión es válida. No filtra paquetes individuales.

---

**Q:** ¿En qué capa OSI opera un Packet Filtering Firewall?
**A:** Capa de red (OSI) / capa internet (TCP/IP).

---

**Q:** ¿Cuál es la best practice de posicionamiento del IDS en la red?
**A:** Layered defense: un IDS delante del firewall y otro detrás. Si sólo hay uno, idealmente cerca del DMZ dentro del firewall.

---

**Q:** Un firewall three-homed se ve comprometido. ¿Qué redes quedan expuestas?
**A:** Tanto la DMZ como la intranet. La alternativa más segura es usar múltiples firewalls independientes para separar Internet↔DMZ y DMZ↔intranet.

---

**Q:** ¿Qué tipo de tráfico NO puede ser colocado en la DMZ aunque parezca razonable?
**A:** Servidores web que se comunican con servidores de bases de datos — darían a usuarios externos acceso directo a datos sensibles.

---

**Q:** ¿Cuál es la diferencia entre el modo activo y pasivo de un Application-Level Firewall?
**A:** Activo: examina peticiones y **rechaza** las que parecen ataques. Pasivo: examina peticiones pero **no las rechaza** — funciona como un IDS.

---

**Q:** ¿Qué tecnología de firewall combina packet filtering + circuit-level + application-level? ¿Ejemplo concreto?
**A:** Stateful Multilayer Inspection Firewall. Ejemplo: Cisco PIX.

---

**Q:** ¿Por qué NAT puede interferir con sistemas de seguridad?
**A:** Porque interfiere con sistemas de cifrado y autenticación, ya que modifica cabeceras IP (y posiblemente puertos) rompiendo la integridad de los paquetes que dichos sistemas verifican.

---

**Q:** ¿Qué característica diferencia un NGFW de un Stateful Multilayer Inspection Firewall?
**A:** El NGFW añade IPS integrado, application awareness/control y threat intelligence en la nube. Ambos hacen deep packet inspection, pero el NGFW opera en múltiples capas OSI simultáneamente y puede bloquear APTs, ransomware y zero-days.

---

**Q:** Enumera cinco limitaciones del firewall que el examen CEH suele preguntar.
**A:** (1) No previene ataques internos/backdoors; (2) no protege contra ingeniería social; (3) no entiende tráfico tunnelizado; (4) no es sustituto de antivirus; (5) no previene ataques desde dispositivos externos ya infectados conectados a la red.

---

**Q:** ¿Cuál es la ventaja principal de un Application Proxy en términos de experiencia de usuario?
**A:** Transparencia — el usuario cree que habla directamente con el servidor real; el servidor cree que habla directamente con el usuario. El proxy gestiona toda la comunicación de forma invisible.

---

**Q:** ¿Qué esquema NAT hace el uso más eficiente de las IPs externas disponibles?
**A:** Asignación dinámica de par IP externa + puerto por cada conexión iniciada desde el interior.

---

**Q:** ¿En qué se diferencia NIDS de HIDS respecto al overhead generado?
**A:** HIDS genera mayor overhead al monitorizar cada evento del sistema individual. NIDS opera de forma pasiva en modo promiscuo sobre el tráfico de red sin impactar los sistemas monitorizados.

---

## 5. Confusión frecuente

**IDS (pasivo) vs IPS (activo)**
- IDS: monitoriza y alerta. No bloquea tráfico. Situado fuera del flujo de datos.
- IPS: inline en la red, bloquea y descarta paquetes maliciosos automáticamente.
- **Criterio de decisión:** si la pregunta menciona "bloquear", "descartar paquetes" o "prevenir ataques", es IPS.

---

**False Positive vs False Negative**
- False Positive: alerta sin ataque real. Genera ruido y fatiga de alertas, pero el sistema está protegido.
- False Negative: ataque real sin alerta. El sistema queda comprometido sin saberlo — el peor escenario.
- **Criterio de decisión:** el examen pregunta cuál es "más peligroso" → siempre False Negative.

---

**Signature Recognition vs Anomaly Detection**
- Signature: conoce el ataque de antes (firmas conocidas). No detecta zero-days.
- Anomaly: detecta comportamientos fuera del umbral. Detecta zero-days pero genera más falsos positivos.
- **Criterio de decisión:** si el escenario menciona zero-day o ataque desconocido → Anomaly Detection.

---

**Circuit-Level Gateway vs Packet Filtering**
- Packet Filtering: capa de red, evalúa paquetes individuales (IP, puerto, protocolo, flags).
- Circuit-Level Gateway: capa de sesión, evalúa sesiones completas verificando el TCP handshake. No filtra paquetes individuales.
- **Criterio de decisión:** si menciona "TCP handshake" o "sesión" → Circuit-Level Gateway.

---

**Application-Level Firewall (modo pasivo) vs IDS**
- Ambos examinan tráfico sin bloquear activamente.
- Application-Level Firewall pasivo: específicamente analiza tráfico de aplicación contra vulnerabilidades conocidas (SQLi, XSS).
- IDS: monitorización más amplia, firma + anomalía.
- **Criterio de decisión:** si el escenario menciona análisis a nivel de aplicación (HTTP, FTP) sin bloqueo → modo pasivo de application-level firewall.

---

**Bastion Host vs Multi-homed Firewall**
- Bastion Host: sistema con DOS interfaces (pública e intranet), diseñado para resistir ataques.
- Multi-homed Firewall: MÁS DE DOS NICs, permite subdividir la red en múltiples segmentos.
- **Criterio de decisión:** si menciona subdivisión de múltiples segmentos de red → Multi-homed Firewall.

---

**NIDS vs HIDS**
- NIDS: monitoriza el tráfico de red completo, modo promiscuo, menos overhead.
- HIDS: monitoriza un sistema individual, más versátil, detecta modificaciones de ficheros, mayor overhead.
- **Criterio de decisión:** si el escenario menciona "modificación de ficheros" o "actividad de un sistema específico" → HIDS.
