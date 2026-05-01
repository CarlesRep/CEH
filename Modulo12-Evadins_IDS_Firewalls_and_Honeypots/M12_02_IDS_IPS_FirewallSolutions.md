# M12_02 — IDS, IPS and Firewall Solutions
> Módulo 12 / Subapartado 2 — IDS, IPS, and Firewall Solutions

---

## 1. Conceptos y definiciones

### YARA — Detección de malware basada en reglas

YARA es una herramienta de investigación de malware **multiplataforma** (Windows, macOS, Linux) que permite detectar y clasificar malware mediante reglas basadas en patrones de texto o binarios. Los analistas crean reglas que describen familias de malware; YARA analiza ficheros contra esas reglas y alerta si se produce una coincidencia.

**Estructura de una regla YARA — tres secciones obligatorias:**

| Sección | Función |
|---|---|
| `meta` | Información descriptiva de la regla (no afecta a la detección): descripción, nivel de amenaza, origen |
| `strings` | Define los patrones a buscar: hex strings (`{6A 40 68...}`), text strings (`"cadena"`), o expresiones regulares |
| `condition` | Expresión booleana que determina cuándo la regla se considera verdadera contra un fichero |

**Sintaxis básica:**
```
rule nombre_regla : tag {
    meta:
        descripcion = "..."
    strings:
        $a = {hex pattern}
        $b = "text string"
    condition:
        $a or $b
}
```

Cada regla comienza con la palabra clave `rule` seguida del nombre. La condición usa operadores booleanos (`or`, `and`, `not`) sobre las variables de strings definidas.

**Ejemplo del libro — `silent_banker`:**
- La regla detecta el troyano si cualquiera de las tres strings (`$a`, `$b`, `$c`) está presente en el fichero analizado.
- Combina patrones hex y texto plano en la misma regla.

**yarGen — generador de reglas YARA:**
- Genera reglas YARA automáticamente a partir de strings identificadas en ficheros de malware.
- Principio clave: **elimina strings que también aparecen en goodware** (ficheros legítimos), minimizando falsos positivos.
- Incluye una base de datos de strings de goodware y opcodes en formato ZIP (requiere extracción previa).
- Instalación de dependencias: `pip install -r requirements.txt`
- Ayuda: `python yarGen.py --help`

**Herramientas YARA adicionales:**
Vovk, Halogen, YARA Silly Silly, yara-forge, YaraRET.

---

### 🔴 Snort — Reglas y Arquitectura

Snort es un NIDS/NIPS open-source capaz de:
- Análisis de tráfico en tiempo real
- Packet logging
- Detección de: buffer overflows, stealth port scans, CGI attacks, SMB probes, OS fingerprinting

Usa **libpcap** (UNIX/Linux) o **WinPcap** (Windows) — la misma librería que tcpdump. Opera en **modo promiscuo** para capturar todos los paquetes de la red.

**Tres modos de uso de Snort:**
1. Sniffer de paquetes (como tcpdump)
2. Packet logger (para debug de tráfico)
3. Network IPS

#### Estructura de una regla Snort

Cada regla tiene dos secciones:

**1. Rule Header** — contiene:
- Acción (qué hacer)
- Protocolo
- IP origen + puerto origen
- Dirección del tráfico
- IP destino + puerto destino
- Bloque CIDR

**2. Rule Options** — contiene:
- Mensaje de alerta
- Información sobre la parte del paquete a inspeccionar

Regla de formato: **una sola línea** — ninguna regla puede extenderse más allá de una línea.

#### 🔴 Seis acciones de regla por defecto en Snort

| Acción | Comportamiento |
|---|---|
| `alert` | Genera alerta y registra el paquete |
| `log` | Registra el paquete sin alerta |
| `pass` | Ignora el paquete |
| `drop` | Bloquea y registra el paquete (IPS mode) |
| `reject` | Bloquea, registra y envía TCP reset/ICMP unreachable |
| `sdrop` | Bloquea sin registrar (silent drop) |

#### Protocolos soportados por Snort
- `tcp` — Transmission Control Protocol
- `udp` — User Datagram Protocol
- `icmp` — Internet Control Message Protocol

#### Operadores de dirección
- `->` : dirección única (origen → destino)
- `<>` : bidireccional — evalúa ambos sentidos (útil para sesiones telnet, POP3)
- **No existe** el operador `<-`

#### Especificación de IPs y puertos

**IPs:**
- `any` — cualquier dirección
- IP numérica con CIDR: `192.168.1.0/24`
- Negación con `!`: `!192.168.1.0/24` (cualquier IP excepto esa red)
- Lista de IPs: `[192.168.1.1,192.168.1.45,10.1.1.24]` — sin espacios, entre corchetes

**Puertos:**
- `any` — cualquier puerto
- Puerto estático: `80`
- Rango con `:` — operador de rango:
  - `:1024` = todos los puertos hasta 1024 (≤1024)
  - `1024:` = todos los puertos desde 1024 en adelante (≥1024)
  - `1:1024` = del 1 al 1024
- Negación: `!6000:6010` = todos excepto el rango 6000-6010

**Ejemplos de reglas con su interpretación:**

| Regla | Significado |
|---|---|
| `log udp any -> 192.168.1.0/24 1:1024` | Registra UDP desde cualquier puerto hacia puertos 1-1024 en la red |
| `log tcp any -> 192.168.1.0/24 :5000` | Registra TCP hacia puertos ≤5000 en la red |
| `log tcp any :1024 -> 192.168.1.0/24 400:` | Registra TCP desde well-known ports (≤1024) hacia puertos ≥400 |
| `log tcp any -> 192.168.1.0/24 !6000:6010` | Registra TCP hacia la red, excepto puertos 6000-6010 |
| `log tcp !192.168.1.0/24 any <> 192.168.1.0/24 23` | Bidireccional: todo TCP que involucre el puerto 23 y la red, excluyendo la propia red como origen |

---

### Suricata

Motor de detección de amenazas de red capaz de IDS, IPS inline, NSM (Network Security Monitoring) y procesamiento offline de pcap. Características diferenciales respecto a Snort:
- Soporte de scripting **Lua** para detección de amenazas complejas
- Salida en **YAML y JSON** — integración nativa con SIEMs, Splunk, Logstash/Elasticsearch, Kibana

---

### 🔴 Catálogo de herramientas por categoría

#### Herramientas de detección de intrusiones (IDS)
| Herramienta | Tipo | Característica clave |
|---|---|---|
| **Snort** | NIDS/NIPS | Open-source, reglas personalizables, libpcap/WinPcap |
| **Suricata** | NIDS/NIPS/NSM | Lua scripting, JSON/YAML output, integración SIEM |
| **Zeek** | NIDS | Análisis de red orientado a logs |
| **Samhain HIDS** | HIDS | Host-based |
| **OSSEC** | HIDS | Open-source, host-based |
| **Cisco Secure IPS** | NIPS | Solución enterprise Cisco |
| **Juniper IDP** | NIPS | Juniper/SolarWinds |

#### Herramientas de prevención de intrusiones (IPS)
| Herramienta | Fabricante |
|---|---|
| Trellix IPS | Trellix (ex-McAfee/FireEye) |
| Check Point Quantum IPS | Check Point |
| Atomic OSSEC | Atomicorp |
| McAfee Host IPS for Desktops | McAfee |
| Secure IPS (NGIPS) | Cisco |
| Palo Alto Advanced Threat Prevention | Palo Alto Networks |

**Trellix IPS — características específicas:**
- Detecta botnets, worms y ataques de reconocimiento
- Agrega flow data desde switches y routers
- Realiza análisis de comportamiento a nivel de red
- Cubre entornos on-premises, virtuales, SDDCs y nubes privadas/públicas

#### Firewalls — Network-based
| Producto | Fabricante |
|---|---|
| Cisco Secure Firewall ASA | Cisco |
| PA-7500 | Palo Alto Networks |
| FortiGate 7121F | Fortinet |
| Check Point 28000 Quantum Security Gateway | Check Point |
| Juniper SRX | Juniper |

#### Firewalls — Host-based
| Producto | Fabricante |
|---|---|
| Microsoft Defender Firewall | Microsoft |
| ZoneAlarm Pro Firewall | ZoneAlarm |
| Comodo Firewall | Comodo |
| Norton Smart Firewall | Norton |
| McAfee Firewall | McAfee |

#### Firewalls — Por mecanismo de trabajo
IPFire, pfSense, SonicWall TZ Series, BIG-IP Advanced Firewall Manager (F5), Sophos Firewall, WatchGuard Firebox, FortiProxy, SonicWall NSa 6700, ZYXEL VPN Firewall, DrayTek Vigor2765.

---

## 2. Exam Traps ⚠️

⚠️ **[YARA — partes de la regla]**
El examen puede preguntar cuántas partes tiene una regla YARA. Son **tres**: `meta`, `strings`, `condition`. La sección `meta` no afecta a la detección — es puramente informativa. Si la pregunta pregunta qué determina si una regla se cumple, la respuesta es `condition`.

⚠️ **[YARA — sintaxis de inicio de regla]**
Cada regla comienza con la palabra clave `rule` seguida del nombre. No confundir con otras palabras reservadas. El nombre puede ir seguido de `:` y una etiqueta (tag) opcional.

⚠️ **[yarGen — principio de funcionamiento]**
yarGen no sólo extrae strings de malware — su valor diferencial es **eliminar strings que también aparecen en goodware**. Si el examen pregunta el propósito principal de yarGen, la respuesta es ese filtrado, no la extracción bruta.

⚠️ **[Snort — número de acciones por defecto]**
Son exactamente **seis**: alert, log, pass, drop, reject, sdrop. El examen puede presentar cinco y pedir cuál falta, o incluir una falsa (ej. "deny") — no existe en Snort por defecto.

⚠️ **[Snort — drop vs reject vs sdrop]**
- `drop`: bloquea Y registra.
- `reject`: bloquea, registra Y envía TCP reset o ICMP unreachable al origen.
- `sdrop`: bloquea pero **no registra** (silent).
El examen puede preguntar cuál bloquea sin dejar traza: **sdrop**.

⚠️ **[Snort — operador de dirección inexistente]**
El operador `<-` **no existe** en Snort. Los operadores válidos son `->` y `<>`. Respuesta automática si el examen lo presenta como opción: incorrecto.

⚠️ **[Snort — librerías de captura]**
Snort usa **libpcap** en UNIX/Linux y **WinPcap** en Windows — las mismas que tcpdump. Si el examen pregunta qué librería usa Snort para sniffing, no confundir con otras librerías.

⚠️ **[Snort vs Suricata — diferenciador clave]**
Suricata añade soporte **Lua scripting** y salida nativa **JSON/YAML** para integración con SIEMs. Si el examen pregunta qué herramienta tiene integración nativa con Elasticsearch/Kibana/Splunk, es **Suricata**.

⚠️ **[Snort — regla de una sola línea]**
Ninguna regla Snort puede extenderse más allá de una sola línea. Si el examen presenta una regla multilínea como válida, es incorrecta.

⚠️ **[Snort — rango de puertos con `:` ]**
`:1024` significa puertos ≤1024 (hasta 1024). `1024:` significa puertos ≥1024 (desde 1024). No confundir la dirección del rango según dónde está el `:`.

⚠️ **[Trellix IPS — origen del producto]**
Trellix es el resultado de la fusión de McAfee Enterprise y FireEye. Si el examen referencia "McAfee IPS enterprise" en contexto moderno, puede estar refiriéndose a Trellix.

---

## 3. Nemotécnicos

**Seis acciones Snort — "ALPD-RS":**
**A**lert → **L**og → **P**ass → **D**rop → **R**eject → **S**drop

Regla de memoria para las tres que bloquean (D-R-S):
- **D**rop = bloquea + registra
- **R**eject = bloquea + registra + **R**esponde (TCP reset/ICMP)
- **S**drop = bloquea en **S**ilencio (sin registro)

**Protocolos Snort — TUI:**
**T**CP, **U**DP, **I**CMP — solo tres.

**Estructura regla YARA — MSC:**
**M**eta (información) → **S**trings (patrones) → **C**ondition (lógica booleana)

**Operadores de dirección Snort:**
- `->` : una dirección (flecha simple)
- `<>` : bidireccional (doble flecha)
- `<-` : NO EXISTE

**Rangos de puerto Snort — regla del dos puntos:**
- `:N` = "hasta N" (≤N) — el dos puntos va delante
- `N:` = "desde N" (≥N) — el dos puntos va detrás
- `N:M` = "de N a M"

**IDS tools open-source top 3:**
**S**nort + **S**uricata + **Z**eek = "SSZ"

---

## 4. Flashcards

**Q:** ¿Cuántas secciones tiene una regla YARA y cuáles son?
**A:** Tres: `meta` (información descriptiva, no afecta detección), `strings` (patrones a buscar: hex, texto, regex), `condition` (expresión booleana que determina si la regla se cumple).

---

**Q:** ¿Cuál es el principio diferencial de yarGen respecto a otros generadores de reglas YARA?
**A:** Elimina de las reglas generadas todas las strings que también aparecen en goodware (ficheros legítimos), reduciendo así los falsos positivos.

---

**Q:** ¿Qué comando instala las dependencias de yarGen?
**A:** `pip install -r requirements.txt`

---

**Q:** Enumera las seis acciones por defecto de las reglas Snort.
**A:** alert, log, pass, drop, reject, sdrop.

---

**Q:** ¿Qué diferencia hay entre `drop`, `reject` y `sdrop` en Snort?
**A:** `drop` bloquea y registra. `reject` bloquea, registra y envía TCP reset o ICMP unreachable al origen. `sdrop` bloquea sin registrar (silent drop).

---

**Q:** ¿Qué librerías usa Snort para captura de paquetes según el sistema operativo?
**A:** libpcap en UNIX/Linux; WinPcap en Windows. Las mismas que usa tcpdump.

---

**Q:** ¿Cuáles son los tres protocolos IP soportados en las reglas Snort?
**A:** TCP, UDP, ICMP.

---

**Q:** ¿Qué significa `:1024` como especificación de puerto en una regla Snort?
**A:** Todos los puertos menores o iguales a 1024 (hasta el puerto 1024 inclusive).

---

**Q:** ¿Existe el operador `<-` en Snort? ¿Qué operadores de dirección existen?
**A:** No existe `<-`. Los operadores válidos son `->` (dirección única, origen a destino) y `<>` (bidireccional).

---

**Q:** ¿Para qué sirve el operador bidireccional `<>` en Snort y cuándo es útil?
**A:** Evalúa el par dirección/puerto en ambos sentidos (origen o destino). Útil para registrar/analizar ambos lados de una conversación, como sesiones telnet o POP3.

---

**Q:** ¿Cuáles son las dos secciones lógicas de una regla Snort?
**A:** Rule header (acción, protocolo, IPs, puertos, CIDR, dirección) y rule options (mensaje de alerta, parte del paquete a inspeccionar).

---

**Q:** ¿Qué característica diferencia a Suricata de Snort?
**A:** Suricata soporta scripting Lua para detección de amenazas complejas y genera salida en YAML/JSON con integración nativa en SIEMs, Splunk, Logstash/Elasticsearch y Kibana.

---

**Q:** ¿Qué hace Trellix IPS que lo diferencia de un IPS convencional?
**A:** Agrega flow data desde switches y routers para realizar análisis de comportamiento a nivel de red, detectando botnets, worms y ataques de reconocimiento en entornos on-prem, virtuales, SDDCs y nubes.

---

**Q:** ¿Qué sintaxis usa Snort para especificar una lista de IPs en una regla?
**A:** Las IPs separadas por comas dentro de corchetes, sin espacios: `[192.168.1.1,192.168.1.45,10.1.1.24]`

---

**Q:** ¿Cómo se niega un rango de puertos en una regla Snort? Ejemplo concreto.
**A:** Con el operador `!` delante del rango. Ejemplo: `!6000:6010` significa todos los puertos excepto del 6000 al 6010.

---

**Q:** ¿Cuáles son los firewalls network-based asociados a Cisco y Palo Alto que menciona el CEH?
**A:** Cisco: Cisco Secure Firewall ASA. Palo Alto: PA-7500.

---

**Q:** ¿En qué modo opera Snort para capturar todos los paquetes de la red?
**A:** Modo promiscuo.

---

## 5. Confusión frecuente

**YARA `strings` vs `condition`**
- `strings`: define QUÉ patrones buscar (hex, texto, regex). Es el "qué".
- `condition`: define CUÁNDO la regla se considera cumplida usando las strings definidas. Es el "cuándo".
- **Criterio de decisión:** si la pregunta es "qué sección determina si una regla se activa", la respuesta es `condition`, no `strings`.

---

**Snort `drop` vs `reject`**
- `drop`: bloquea silenciosamente desde el punto de vista del origen — el paquete desaparece, no hay respuesta.
- `reject`: bloquea Y notifica al origen con TCP reset (para TCP) o ICMP port unreachable (para UDP).
- **Criterio de decisión:** si el escenario menciona "notificar al origen" o "enviar respuesta de rechazo", es `reject`.

---

**Snort `pass` vs `sdrop`**
- `pass`: ignora el paquete — lo deja pasar sin registrarlo. No bloquea.
- `sdrop`: bloquea el paquete sin registrarlo. No lo deja pasar.
- **Criterio de decisión:** si el paquete debe pasar sin registro → `pass`. Si debe bloquearse sin registro → `sdrop`.

---

**Snort vs Suricata — cuándo usar cada uno**
- Snort: el referente histórico open-source, reglas flexibles, bien documentado para el examen CEH.
- Suricata: más moderno, multithreading, integración SIEM nativa, Lua scripting.
- **Criterio de decisión en el examen:** si menciona integración con Elasticsearch/Kibana/Splunk out-of-the-box → Suricata. Si menciona libpcap/WinPcap o el NIDS open-source canónico → Snort.

---

**YARA vs Snort — propósito**
- YARA: análisis de **ficheros** para detección/clasificación de malware. Trabaja sobre binarios en disco o bases de datos.
- Snort: análisis de **tráfico de red** en tiempo real. Trabaja sobre paquetes.
- **Criterio de decisión:** si el escenario involucra analizar un fichero binario sospechoso → YARA. Si involucra tráfico de red → Snort.
