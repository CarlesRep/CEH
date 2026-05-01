# M03 — Host Discovery y Ping Scans

## Concepto

**Host Discovery** = primer paso del network scanning — identifica sistemas activos antes de escanear puertos

> ⚠️ ARP ping scan es el **default** de Nmap
> `-sn` = desactiva port scan (solo host discovery)
> `--disable-arp-ping` = desactiva el ARP ping por defecto

---

## Resumen de técnicas — tabla maestra

| Técnica | Opción Nmap | Protocolo | Puerto/Default | Cuando usarlo |
|---|---|---|---|---|
| **ARP Ping Scan** | `-PR` | ARP | — | Default en redes locales — más eficiente |
| **UDP Ping Scan** | `-PU` | UDP | **40,125** | Detrás de firewalls con filtrado TCP estricto |
| **ICMP ECHO Ping** | `-PE` | ICMP | — | UNIX/Linux/BSD — **NO funciona en Windows** |
| **ICMP Timestamp Ping** | `-PP` | ICMP | — | Cuando admin bloquea ICMP ECHO |
| **ICMP Address Mask Ping** | `-PM` | ICMP | — | Cuando admin bloquea ICMP ECHO |
| **TCP SYN Ping** | `-PS` | TCP | **80** | Sin logs — paralelo |
| **TCP ACK Ping** | `-PA` | TCP | **80** | Bypasea firewalls que bloquean SYN |
| **IP Protocol Ping** | `-PO` | Multi | ICMP(1)+IGMP(2)+IP-in-IP(4) | Cualquier respuesta indica host online |

---

## ARP Ping Scan (`-PR`)

- Envía paquetes **ARP** → ARP response = host activo
- Muestra **MAC address** del dispositivo
- Maneja retransmisiones y timeouts automáticamente
- El más eficiente para redes locales grandes

---

## UDP Ping Scan (`-PU`)

- Puerto default: **40,125**
- Respuesta UDP = host activo
- Si offline: host/network unreachable o TTL exceeded
- Ventaja: detecta hosts detrás de firewalls con filtrado TCP

---

## ICMP ECHO Ping (`-PE`) y Ping Sweep

- ICMP ECHO request → si activo → ICMP ECHO reply
- **Funciona en UNIX/Linux/BSD — NO en Windows** (no responde a ICMP en broadcast)
- **Ping Sweep** = envío a rango de IPs simultáneamente
- Paquete ping = **64 bytes** (56 datos + 8 cabecera)
- Método más **antiguo y lento**

---

## ICMP Timestamp Ping (`-PP`) y Address Mask Ping (`-PM`)

- **Timestamp (`-PP`):** consulta timestamp → para sincronización de tiempo
- **Address Mask (`-PM`):** consulta máscara de subred
- Ambos: respuesta **condicional** según configuración del admin
- Útiles cuando el admin **bloquea ICMP ECHO** tradicional

---

## TCP SYN Ping (`-PS`)

- Envía **SYN vacío** → host activo responde con **ACK** → atacante envía **RST**
- Puerto default: **80** — admite rangos: `-PS22-25,80,113`
- Ventajas: escaneo paralelo (sin timeout) · **no crea conexión completa → sin logs**

---

## TCP ACK Ping (`-PA`)

- Envía **ACK vacío** directamente
- Host responde con **RST** → host activo
- Puerto default: **80**
- Ventaja: bypasea firewalls que bloquean SYN

> ⚠️ **SYN vs ACK Ping:** SYN = sin logs · ACK = bypasea firewalls que bloquean SYN

---

## Ping Sweep Tools

| Herramienta | Diferencial |
|---|---|
| **Angry IP Scanner** | Escanea rangos + puertos — **multithreaded** — exporta CSV/TXT/XML — NetBIOS info |
| **SolarWinds Engineer's Toolset** | Suite completa |
| **NetScanTools Pro** | Investigación de red |
| **Colasoft Ping Tool** | Ping sweep |
| **Advanced IP Scanner** | Escaneo rápido |
| **OpUtils** | ManageEngine |

### 📝 Pregunta típica CEH

> *"Un atacante quiere descubrir hosts en una red sin dejar logs. ¿Qué ping scan usa?"*
> → **TCP SYN Ping (`-PS`)** — no crea conexión completa, sin logs

> *"¿Qué ping scan usa Nmap por defecto en redes locales?"*
> → **ARP Ping Scan (`-PR`)**

> *"¿Qué ping scan funciona en UNIX pero NO en Windows?"*
> → **ICMP ECHO Ping (`-PE`)**
