# M02 — Network y Email Footprinting, Traceroute

## Network Range

**Rangos IP privados IANA — memorizar los 3:**

| Rango | Prefijo |
|---|---|
| `10.0.0.0 – 10.255.255.255` | /8 |
| `172.16.0.0 – 172.31.255.255` | /12 |
| `192.168.0.0 – 192.168.255.255` | /16 |

> Para obtener network range: IP del servidor (de Whois) → **ARIN Whois database**

---

## Traceroute

**Cómo funciona:**
- Usa **protocolo ICMP** + campo **TTL** del header IP
- Cada router decrementa TTL en 1
- Cuando TTL = 0 → router devuelve **ICMP error message** → revela su IP
- Al llegar al destino → **ICMP reply normal**

**Qué revela:** IPs de routers intermedios · round-trip time · afiliación de red · geolocalización

### 3 Tipos de Traceroute

| Tipo | Protocolo | Comando | SO |
|---|---|---|---|
| **ICMP** | ICMP | `tracert <IP>` | Windows (default) |
| **TCP** | TCP (Layer 4) | `sudo tcptraceroute <host>` | Linux |
| **UDP** | UDP | `traceroute <host>` | Linux (default) |

> ⚠️ TCP/UDP traceroute = **Layer 4 traceroute** — cuando dispositivos bloquean ICMP
> ⚠️ `tracert` = Windows (ICMP) · `traceroute` = Linux (UDP por defecto)

### Herramientas Traceroute

| Herramienta | Diferencial |
|---|---|
| **NetScanTools Pro** | ICMP/UDP/TCP · localiza **país** de cada hop IPv4 |
| **PingPlotter** | ICMP/UDP/TCP · gráficos latencia y packet loss · detecta bottlenecks |
| **Traceroute NG** | Análisis histórico + reverse DNS |
| **tracert** | Nativa Windows |

---

## Email Tracking — Info obtenible

| Dato | Qué revela |
|---|---|
| **IP del destinatario** | Geolocalización del receptor |
| **Email received/read** | Cuándo se abre |
| **Read duration** | Tiempo leyendo |
| **Proxy detection** | Tipo de servidor del receptor |
| **OS y Browser** | Versión exacta → identificar vulnerabilidades |
| **Forward email** | Si fue reenviado a terceros |
| **Device type** | Desktop, móvil, laptop |
| **Path travelled** | Ruta por email transfer agents |
| **Links** | Si se hicieron clic |

---

## Email Header — Información contenida

- Servidor de correo del remitente
- Fecha/hora de recepción en servidores de origen
- Sistema de autenticación del servidor (DKIM, SPF)
- Fecha/hora de envío
- ID único del mensaje (asignado por mx.google.com)
- Nombre completo del remitente
- **IP del remitente** y dirección de envío

### Herramientas Email Tracking

| Herramienta | Función |
|---|---|
| **eMailTrackerPro** | Analiza headers → IP, geolocalización · guarda trazas históricas |
| **IP2LOCATION Email Header Tracer** | Traza ruta del email por IPs del header — open source |
| **MxToolbox** | Análisis de headers y DNS |
| **Holehe** | Comprueba si un email está registrado en servicios online |
| **Social Catfish** | Investigación de identidad por email |

> ⚠️ **Email Tracking vs Email Header:**
> - Tracking = monitoriza **comportamiento del receptor**
> - Header = info de **ruta y origen** del email

### 📝 Pregunta típica CEH

> *"Un atacante quiere saber si el destinatario abrió su email. ¿Qué usa?"*
> → **Email tracking tool** (ej. eMailTrackerPro)

> *"¿Qué información del header de un email permite rastrear la IP del remitente?"*
> → El campo **Received** y la **IP del remitente** en el header
