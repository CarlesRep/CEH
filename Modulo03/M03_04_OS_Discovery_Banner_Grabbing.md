# M03 — OS Discovery y Banner Grabbing

## Banner Grabbing / OS Fingerprinting

**Objetivo:** identificar el OS del sistema remoto → exploits OS-específicos

### 2 Tipos

| Tipo | Cómo funciona | Detectabilidad |
|---|---|---|
| **Active** | Envía paquetes especiales/malformados → compara respuestas con base de datos | Detectable por IDS |
| **Passive** | Captura y analiza tráfico del target (sniffing) — sin enviar paquetes | Muy difícil de detectar |

---

## Active Banner Grabbing — 9 tests de Nmap

| Test | Qué envía | Puerto |
|---|---|---|
| **T1** | SYN + ECN-Echo | TCP abierto |
| **T2** | NULL (sin flags) | TCP abierto |
| **T3** | URG+PSH+SYN+FIN | TCP abierto |
| **T4** | ACK | TCP abierto |
| **T5** | SYN | TCP **cerrado** |
| **T6** | ACK | TCP **cerrado** |
| **T7** | URG+PSH+FIN | TCP **cerrado** |
| **T8 (PU)** | UDP | UDP **cerrado** → extrae ICMP port unreachable |
| **T9 (TSeq)** | 6 × SYN | TCP abierto → analiza ISN, IPID, TCP timestamp |

> ⚠️ T1-T4 = puertos **abiertos** · T5-T7 = puertos **cerrados** — respuestas diferentes revelan el OS

**Patrones de ISN (T9):**
- Traditional 64K → antiguas UNIX boxes
- Random increments → Solaris, IRIX, FreeBSD, Digital UNIX
- True random → Linux 2.0.x, OpenVMS, AIX
- Time-dependent → **Windows** (ISN incrementado en cantidad fija por tiempo)

---

## Passive Banner Grabbing

**Métodos:**
- **Banner desde mensajes de error** → tipo servidor, OS, SSL
- **Sniffing de tráfico** → captura paquetes del target
- **Extensiones de página** → `.aspx` → IIS + Windows platform

### 4 Parámetros clave para identificar OS

| Parámetro | Qué indica |
|---|---|
| **TTL** | Valor inicial según OS |
| **Window Size** | Tamaño de ventana TCP del OS |
| **DF bit** | Si el OS activa "Don't Fragment" |
| **TOS** | Type of Service — más dependiente del protocolo que del OS |

---

## TTL y Window Size por OS — MEMORIZAR

| OS | TTL | TCP Window Size |
|---|---|---|
| **Linux** | **64** | 5840 |
| **FreeBSD** | **64** | 65535 |
| **OpenBSD** | **255** | 16384 |
| **Windows** | **128** | 65,535 bytes – 1 GB |
| **Cisco Routers** | **255** | 4128 |
| **Solaris** | **255** | 8760 |
| **AIX** | **255** | 16384 |

> 🧠 **Regla TTL:** Linux/FreeBSD = **64** · Windows = **128** · Resto = **255**
> ⚠️ Para calcular hops: `hops = TTL_inicial - TTL_recibido`

---

## Herramientas de OS Discovery

| Herramienta | Método | Opción/Comando |
|---|---|---|
| **Nmap** | Active fingerprinting (9 tests) | `-O` |
| **Wireshark** | Passive — analiza TTL + Window Size del primer paquete TCP | Sniffing |
| **Unicornscan** | Observa TTL del resultado scan | `unicornscan <IP>` → TTL 128 = Windows |
| **Nmap NSE** | Script SMB | `--script smb-os-discovery` o `-sC` |
| **IPv6 Fingerprinting** | 18 probes IPv6 | `nmap -6 -O <target>` |

---

## IPv6 Fingerprinting — 18 probes (orden)

S1–S6 · IE1 · IE2 · NI · NS · U1 · TECN · T2–T7

`nmap -6 -O <target>`

---

## ⚠️ Distinciones clave

| Par | Diferencia |
|---|---|
| **Active vs Passive** | Active = envía paquetes al target · Passive = solo sniffing |
| **-O vs -sV** | `-O` = detecta OS · `-sV` = detecta versión del servicio |
| **Unicornscan vs Nmap** | Unicornscan = identifica OS por TTL observado · Nmap = múltiples probes + base de datos |

### 📝 Pregunta típica CEH

> *"Un analista captura tráfico y ve TTL=128 en los paquetes del target. ¿Qué OS es probablemente?"*
> → **Windows**

> *"Un analista captura tráfico y ve TTL=64. ¿Qué OS es probablemente?"*
> → **Linux o FreeBSD**

> *"¿Qué opción de Nmap realiza OS fingerprinting?"*
> → `-O`
