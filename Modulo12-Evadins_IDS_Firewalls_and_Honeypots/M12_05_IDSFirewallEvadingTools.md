# M12_05_IDSFirewallEvadingTools

## 1. Conceptos y definiciones

### IDS/Firewall Evasion Tools 🔴
Los **IDS/Firewall evading tools** son utilidades empleadas para **probar, modelar o generar tráfico** con el fin de analizar cómo reaccionan los controles perimetrales y, en contexto ofensivo, encontrar formas de **sortear reglas de filtrado, detección o inspección**. La lógica técnica no consiste solo en “mandar tráfico raro”, sino en **forzar al dispositivo de seguridad a interpretar el tráfico de forma distinta al host destino** o en **encapsular/parecer tráfico legítimo** para reducir la probabilidad de bloqueo.

En un escenario real, un firewall, un IDS o un IPS no toman decisiones en abstracto: aplican firmas, reglas de estado, normalización de paquetes, inspección de puertos, protocolos y, en equipos avanzados, análisis de capa 7. Un atacante usa estas herramientas para responder preguntas concretas:

- ¿Qué tipos de tráfico deja pasar el dispositivo?
- ¿Qué fragmentación, puertos o patrones no reensambla o no inspecciona correctamente?
- ¿Qué tráfico se considera “aplicación legítima” y cuál dispara una firma?
- ¿Puede generarse tráfico personalizado que llegue al objetivo pero no active la detección?

El examen CEH suele enfocar estas herramientas desde dos ópticas: **auditoría defensiva** y **abuso ofensivo**. La misma herramienta puede ser legítima para validar controles o maliciosa para diseñar evasiones. Lo importante es **qué genera o modifica**: tráfico de aplicación, paquetes personalizados, fragmentación o encapsulación.

### Traffic IQ Professional
**Traffic IQ Professional** es una herramienta de **auditoría y validación del comportamiento** de dispositivos de seguridad. Su función principal es **generar tráfico estándar de aplicación o tráfico de ataque entre dos máquinas virtuales** para comprobar cómo se comportan dispositivos como:

- firewalls de aplicación,
- IDS,
- IPS,
- routers,
- switches,
- y otros dispositivos de filtrado no proxy.

La lógica de la herramienta es sencilla pero potente: si puedes **reproducir de forma controlada tráfico normal y tráfico hostil**, puedes observar qué bloquea, qué registra y qué deja pasar un dispositivo. Desde el punto de vista ofensivo, esto permite **perfilar el perímetro** y ajustar el tráfico malicioso para que se parezca lo suficiente al tráfico permitido o para que explote huecos concretos en las reglas.

No es una herramienta “de explotación” en el sentido clásico; es una herramienta de **generación y validación de tráfico**, útil para ver si el dispositivo filtra por firma, por protocolo, por puerto o por comportamiento.

### Herramientas adicionales de evasión de IDS/firewall 🔴
El chunk enumera varias herramientas que CEH puede preguntar por **asociación funcional**, no por detalle operativo profundo. La trampa habitual es mezclar una herramienta de **escaneo/explotación general** con una de **túnel**, **fragmentación** o **tráfico encubierto**.

| Herramienta | Lógica principal en evasión | Idea de examen |
|---|---|---|
| **Nmap** | Escaneo y enumeración con opciones de evasión | Asociarla a **escaneo sigiloso**, manipulación de paquetes y reconocimiento |
| **Metasploit** | Framework de explotación con módulos y payloads | Asociarlo a **explotación y entrega de payloads**, no a simple fragmentación |
| **PingRAT** | Canal/actividad sobre ICMP | Asociarlo a **abuso de tráfico permitido** |
| **KoviD** | Herramienta vinculada a evasión/túneles | Asociarla a **encubrimiento de comunicaciones** |
| **Green Tunnel** | Túnel para sortear restricciones | Asociarlo a **bypass de censura/firewall** |
| **Hyperion** | Cripter/obfuscator usado para ocultar payloads | Asociarlo a **ofuscación y evasión AV/IDS**, no a fragmentación IP |

### Generadores de fragmentación de paquetes 🔴
Los **packet fragment generator tools** se usan para crear **paquetes fragmentados** o manipulados con el fin de comprobar si el firewall o IDS:

1. **reensambla correctamente** los fragmentos,
2. **inspecciona el contenido tras el reensamblado**,
3. o **toma decisiones parciales sobre fragmentos incompletos**.

La lógica técnica de la evasión por fragmentación es que algunos controles **no inspeccionan igual un paquete completo que varios fragmentos separados**, especialmente si el contenido malicioso queda distribuido entre fragmentos. Si el sistema de seguridad no reconstruye el flujo exactamente igual que el host destino, puede producirse una **desincronización de interpretación**: el firewall/IDS “ve” algo inocuo o insuficiente, pero el host final reensambla el contenido real.

No toda fragmentación implica evasión exitosa. Su valor depende de:

- cómo reensambla el dispositivo de seguridad,
- si hay inspección con estado,
- si existe normalización de tráfico,
- y si el sistema final reensambla de forma distinta.

### Colasoft Packet Builder
**Colasoft Packet Builder** es una herramienta para **crear paquetes de red personalizados** y **fragmentarlos**. Su valor en evasión está en que permite construir paquetes específicos en diferentes capas para comprobar si el dispositivo de seguridad detecta o bloquea tráfico anómalo, fragmentado o malformado.

El chunk indica que puede construir paquetes como:

- **Ethernet Packet**
- **ARP Packet**
- **IP Packet**
- **TCP Packet**
- **UDP Packet**

La lógica de examen aquí es clara: no es un escáner como Nmap ni un framework de explotación como Metasploit. Es un **constructor/generador de paquetes**. En un test, si la pregunta habla de **crear paquetes personalizados o fragmentados** para probar si un firewall los detecta, esta es la asociación correcta.

### Otras herramientas de generación de paquetes
El chunk menciona varias utilidades adicionales de esta familia. El objetivo no es memorizar todos sus detalles internos, sino reconocer que pertenecen al grupo de **packet crafting / packet generation / network testing**:

- **NetScanTools Pro**
- **CommView**
- **Ostinato**
- **WAN Killer**
- **WireEdit**

En examen, el criterio útil es este:

- si la pregunta habla de **crear, editar o generar tráfico/paquetes**, piensa en esta familia;
- si habla de **explotar un servicio vulnerable**, piensa en Metasploit;
- si habla de **escaneo y enumeración con opciones de evasión**, piensa en Nmap.

---

## 2. Exam Traps ⚠️

⚠️ **[Traffic IQ Professional]** El examen puede presentarla como si fuera exclusivamente una herramienta defensiva de auditoría. La confusión está en que, aunque se use para validar el comportamiento de firewalls, IDS e IPS, también puede **generar tráfico de ataque personalizado** y por eso puede ser utilizada por atacantes para estudiar y evadir controles perimetrales.

⚠️ **[Traffic IQ Professional vs Colasoft Packet Builder]** La trampa típica es intercambiar sus funciones. **Traffic IQ Professional** se centra en **auditar/validar el comportamiento de dispositivos de seguridad generando tráfico de aplicación o de ataque entre VMs**. **Colasoft Packet Builder** se centra en **crear y fragmentar paquetes personalizados**.

⚠️ **[Nmap]** CEH suele intentar que el alumno reduzca Nmap a “simple port scanner”. En preguntas de evasión, Nmap debe asociarse a **reconocimiento con capacidades de evasión**, no a framework de explotación ni a simple editor de paquetes.

⚠️ **[Metasploit]** Puede aparecer como distractor cuando la pregunta trata de evasión de firewall. La respuesta correcta solo será Metasploit si el foco está en **payloads, explotación o módulos de ataque**, no si el enunciado describe **fragmentación específica de paquetes** o **construcción manual de tráfico**.

⚠️ **[Packet Fragmentation]** El examen puede sugerir que fragmentar un paquete siempre evita el firewall. Incorrecto. La fragmentación solo ayuda cuando el dispositivo **no reensambla, no normaliza o no inspecciona correctamente** los fragmentos.

⚠️ **[Colasoft Packet Builder]** La pregunta puede hablar de crear **Ethernet, ARP, IP, TCP o UDP packets** y añadir como distractor herramientas de escaneo. La correcta será **Colasoft Packet Builder** porque su función es **packet crafting**, no enumeración.

⚠️ **[PingRAT]** Puede aparecer en una lista mezclada con herramientas de escaneo y packet crafting. La pista es que PingRAT se asocia con **abuso del tráfico ICMP** para canalizar actividad, no con generación genérica de fragmentos.

⚠️ **[Green Tunnel]** El examen puede meterla como si fuera una herramienta de explotación. El concepto correcto es **túnel para sortear restricciones/censura/firewall**, no un framework de explotación ni un generador de paquetes de bajo nivel.

⚠️ **[Hyperion]** Puede confundirse con herramientas de red. Su asociación más útil para CEH es **ofuscación/criptado de payloads** para reducir detección, no fragmentación IP ni auditoría de firewalls.

---

## 3. Nemotécnicos

### Para distinguir las familias de herramientas
**Traffic IQ = “IQ inspecciona el comportamiento”**  
Piensa en *IQ* como “análisis del comportamiento del dispositivo”. Sirve para **ver cómo responde** el firewall/IDS/IPS.

**Colasoft = “coloqué el paquete como quise”**  
Asócialo a **construcción manual de paquetes**: Ethernet, ARP, IP, TCP, UDP.

**Nmap = “mapear”**  
Si la idea central es **descubrir/sondear** el perímetro, piensa en Nmap.

**Metasploit = “meter el exploit”**  
Si la pregunta habla de **payload**, **módulos** o **explotación**, la asociación correcta suele ser Metasploit.

**Green Tunnel = túnel verde = paso permitido**  
Úsalo para recordar **bypass mediante túnel**.

**PingRAT = ping + RAT**  
Recuerda **ICMP usado como canal encubierto**.

### Lista de tipos de paquetes de Colasoft Packet Builder 🔴
**E-A-I-T-U**  
- **E**thernet  
- **A**RP  
- **I**P  
- **T**CP  
- **U**DP

Piensa: **“En ARP/IP/TCP/UDP está el paquete que edito”**.

### Regla de decisión rápida en examen
**¿Audita comportamiento? → Traffic IQ**  
**¿Crea/fragmenta paquetes? → Colasoft Packet Builder**  
**¿Escanea con evasión? → Nmap**  
**¿Explota/lanza payloads? → Metasploit**

---

## 4. Flashcards

**Q:** ¿Cuál es la finalidad principal de las herramientas de IDS/firewall evasion?  
**A:** Evaluar y explotar debilidades en firewalls, IDS e IPS generando tráfico, paquetes o patrones de comunicación que eviten su bloqueo o detección.

**Q:** ¿Qué hace Traffic IQ Professional en un contexto de auditoría?  
**A:** Genera tráfico estándar de aplicación o tráfico de ataque entre dos máquinas virtuales para validar el comportamiento de dispositivos de seguridad.

**Q:** ¿Qué tipos de dispositivos puede evaluar Traffic IQ Professional?  
**A:** Firewalls de aplicación, IDS, IPS, routers, switches y otros dispositivos de filtrado no proxy.

**Q:** En CEH, ¿por qué Traffic IQ Professional puede considerarse útil también para atacantes?  
**A:** Porque permite generar tráfico de ataque personalizado para perfilar el comportamiento del perímetro y ajustar técnicas de evasión.

**Q:** ¿Qué herramienta del chunk se asocia mejor con escaneo y enumeración con opciones de evasión?  
**A:** Nmap.

**Q:** ¿Qué herramienta del chunk se asocia mejor con explotación y payload delivery?  
**A:** Metasploit.

**Q:** ¿Qué herramienta del chunk se utiliza para crear paquetes personalizados y fragmentados?  
**A:** Colasoft Packet Builder.

**Q:** ¿Qué tipos de paquetes puede crear Colasoft Packet Builder según el chunk?  
**A:** Ethernet, ARP, IP, TCP y UDP.

**Q:** ¿Cuál es la lógica de la evasión por fragmentación de paquetes?  
**A:** Dividir el tráfico para que el firewall o IDS no lo reensamble o inspeccione correctamente, mientras el host destino sí reconstruye el contenido real.

**Q:** ¿La fragmentación garantiza por sí sola la evasión de un firewall?  
**A:** No; solo funciona si el dispositivo no normaliza, no reensambla o no inspecciona correctamente los fragmentos.

**Q:** ¿Qué herramienta del chunk se relaciona con el uso de ICMP como canal encubierto?  
**A:** PingRAT.

**Q:** ¿Qué herramienta del chunk se relaciona con túneles para sortear restricciones de red?  
**A:** Green Tunnel.

**Q:** ¿Qué herramienta del chunk se relaciona mejor con ofuscación de payloads para evitar detección?  
**A:** Hyperion.

**Q:** Si una pregunta describe validación del comportamiento de un IDS usando tráfico legítimo y malicioso entre dos VMs, ¿qué herramienta encaja mejor?  
**A:** Traffic IQ Professional.

**Q:** Si una pregunta describe creación manual de un paquete TCP fragmentado para probar un firewall, ¿qué herramienta encaja mejor?  
**A:** Colasoft Packet Builder.

**Q:** ¿Qué diferencia conceptual hay entre Traffic IQ Professional y Colasoft Packet Builder?  
**A:** Traffic IQ Professional valida el comportamiento de dispositivos generando tráfico de aplicación o ataque; Colasoft Packet Builder construye y fragmenta paquetes concretos.

**Q:** ¿Cuál es el error típico al confundir Nmap y Metasploit en CEH?  
**A:** Tratar Nmap como framework de explotación o Metasploit como simple escáner; Nmap se centra en reconocimiento y Metasploit en explotación/payloads.

**Q:** ¿Qué familia de herramientas incluye NetScanTools Pro, CommView, Ostinato, WAN Killer y WireEdit en este chunk?  
**A:** Herramientas de generación, edición o prueba de paquetes/tráfico de red.

**Q:** ¿Qué idea de examen suele esconder una pregunta sobre packet fragment generators?  
**A:** Que la evasión depende del manejo de fragmentos por el dispositivo de seguridad, no solo del hecho de fragmentar.

**Q:** ¿Qué significa que un dispositivo sea “non-proxy packet-filtering device” en el contexto del chunk?  
**A:** Que filtra tráfico sin actuar como proxy de aplicación, por lo que su comportamiento puede validarse mediante generación controlada de tráfico y ataques.

---

## 5. Confusión frecuente

### Traffic IQ Professional vs Colasoft Packet Builder
**Diferencia memorable:**  
Traffic IQ **prueba el comportamiento del control**. Colasoft **construye el paquete**.

**Criterio de decisión:**  
- Si el enunciado habla de **auditar o validar** cómo responde un firewall/IDS/IPS, elige **Traffic IQ Professional**.  
- Si habla de **crear o fragmentar paquetes concretos**, elige **Colasoft Packet Builder**.

### Nmap vs Metasploit
**Diferencia memorable:**  
Nmap **descubre**. Metasploit **explota**.

**Criterio de decisión:**  
- Si la pregunta trata de **escaneo, enumeración o reconocimiento con evasión**, elige **Nmap**.  
- Si trata de **payloads, módulos, explotación o post-explotación**, elige **Metasploit**.

### Packet fragmentation vs tunneling
**Diferencia memorable:**  
La **fragmentación** rompe el paquete en trozos; el **túnel** disfraza o encapsula la comunicación para atravesar restricciones.

**Criterio de decisión:**  
- Si la pregunta habla de **fragmentos IP o paquetes troceados**, piensa en **packet fragment generators**.  
- Si habla de **sortear restricciones transportando tráfico por otro canal**, piensa en **tunneling tools** como Green Tunnel.

### PingRAT vs Green Tunnel
**Diferencia memorable:**  
PingRAT **abusa de ICMP**. Green Tunnel **crea un túnel para pasar**.

**Criterio de decisión:**  
- Si la pista es **ping/ICMP**, elige **PingRAT**.  
- Si la pista es **túnel para bypass de restricciones**, elige **Green Tunnel**.

### Hyperion vs Colasoft Packet Builder
**Diferencia memorable:**  
Hyperion **oculta el payload**. Colasoft **manipula el paquete de red**.

**Criterio de decisión:**  
- Si la pregunta habla de **ofuscación o reducción de detección de malware**, elige **Hyperion**.  
- Si habla de **crear tramas o paquetes personalizados**, elige **Colasoft Packet Builder**.
