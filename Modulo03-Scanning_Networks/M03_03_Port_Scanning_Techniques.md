# M03 — Técnicas de Port Scanning

## Árbol de clasificación

```
TCP
├── Open TCP:    TCP Connect/Full-Open        (-sT)
├── Stealth TCP:
│   ├── Half-Open/SYN Scan                   (-sS)
│   ├── Inverse TCP Flag:
│   │   ├── FIN Scan                         (-sF)
│   │   ├── NULL Scan                        (-sN)
│   │   ├── Xmas Scan                        (-sX)
│   │   └── Maimon Scan                      (-sM)
│   └── ACK Flag Probe:
│       ├── TTL-Based                        (--ttl)
│       ├── Window-Based                     (-sW)
│       └── ACK Scan (firewall check)        (-sA)
└── Spoofed TCP: IDLE/IPID Scan              (-sI)

UDP:             UDP Scan                    (-sU)
SCTP:
├──              SCTP INIT Scan              (-sY)
└──              SCTP COOKIE ECHO Scan       (-sZ)
Otros:
├──              SSDP Scan
├──              List Scan                   (-sL)
└──              IPv6 Scan                   (-6)
```

---

## TCP Connect / Full-Open Scan (`-sT`)

- Completa el **three-way handshake** completo → envía RST al cerrar
- Puerto **abierto** → conexión exitosa
- Puerto **cerrado** → error de conexión
- **Más detectable** — queda registrado en logs
- No requiere privilegios de root

---

## Half-Open / SYN / Stealth Scan (`-sS`)

- Envía **SYN** → recibe SYN+ACK → envía **RST** (interrumpe antes de completar)
- **No queda en logs** → stealth
- También llamado: SYN scan

| Respuesta | Estado |
|---|---|
| SYN+ACK | Puerto **abierto** |
| RST | Puerto **cerrado** |

---

## Inverse TCP Flag Scans — RFC 793

> **Regla RFC 793:** Puerto cerrado → responde **RST/ACK** · Puerto abierto → **sin respuesta**
> ⚠️ **Solo efectivo en UNIX/BSD — NO en Windows** (ignora RFC 793)

| Scan | Flags enviadas | Opción |
|---|---|---|
| **FIN Scan** | FIN | `-sF` |
| **NULL Scan** | Ninguna (0 flags) | `-sN` |
| **Xmas Scan** | FIN + URG + PSH | `-sX` |

**Ventajas:** evita IDS y logging · muy stealth
**Desventajas:** requiere root · solo efectivo en BSD/UNIX

---

## TCP Maimon Scan (`-sM`)

- Probe: **FIN/ACK**
- Puerto **cerrado** → RST
- Puerto **abierto** → paquete descartado (en BSD)
- Sin respuesta → `open|filtered`
- ICMP unreachable (type 3, code 1,2,3,9,10,13) → `filtered`

---

## ACK Flag Probe Scans

### TTL-Based (`--ttl`)
- Envía ACK → analiza **TTL del RST recibido**
- **TTL < 64** → puerto **abierto**
- **TTL > 64** → puerto cerrado

### Window-Based (`-sW`)
- Envía ACK → analiza **Window del RST recibido**
- **Window ≠ 0** → puerto **abierto**
- **Window = 0** → puerto cerrado
- Usar cuando todos los paquetes tienen el mismo TTL

### ACK para detectar firewalls (`-sA`)
- Sin respuesta → puerto **filtrado** (firewall stateful)
- RST → puerto **no filtrado** (sin firewall)

**Ventajas:** evade IDS en la mayoría de casos
**Desventajas:** muy lento · solo en BSD TCP/IP stack vulnerable

---

## IDLE / IPID Header Scan (`-sI`) — Escaneo anónimo

**Concepto:** usa un **zombie** como intermediario → IP real oculta

### Los 3 pasos

**Paso 1:** Sondear IPID del zombie
```
Atacante → SYN+ACK → Zombie → RST (con IPID = X, ej. 31337)
```

**Paso 2:** Escanear target con IP zombie spoofada
```
Si puerto ABIERTO:  Target → SYN+ACK → Zombie → RST (IPID = X+1)
Si puerto CERRADO: Target → RST → Zombie (queda idle, IPID sin cambio)
```

**Paso 3:** Volver a sondear IPID del zombie
```
IPID aumentó +2 (X+2) → puerto ABIERTO
IPID aumentó +1 (X+1) → puerto CERRADO
```

> ⚠️ El zombie debe incrementar IPID **globalmente** para ser útil

---

## UDP Scan (`-sU`)

- No hay handshake
- Puerto **cerrado** → **ICMP port unreachable** (ICMP_PORT_UNREACH)
- Puerto **abierto** → sin respuesta (tráfico perdido = abierto)
- Lento por limitación de rate de ICMP errors (RFC 1812)
- Requiere privilegios root
- Complementar con `-sV` o `-O` para más info

---

## SCTP INIT Scan (`-sY`)

**SCTP four-way handshake:** INIT → INIT-ACK → COOKIE-ECHO → COOKIE-ACK

| Respuesta | Estado |
|---|---|
| INIT+ACK | Puerto **abierto** |
| ABORT | Puerto **cerrado** |
| Sin respuesta / ICMP unreachable | **filtered** |

**Ventaja:** diferencia claramente open / closed / filtered

---

## SCTP COOKIE ECHO Scan (`-sZ`)

- Envía **COOKIE ECHO**
- Puerto **abierto** → sin respuesta (paquete descartado)
- Puerto **cerrado** → **ABORT**
- No bloqueado por firewalls no-stateful

> ⚠️ **INIT vs COOKIE ECHO:** INIT = más detectable · COOKIE ECHO = más sigiloso pero no diferencia open de filtered → `open|filtered`

---

## List Scan (`-sL`)

- Solo genera lista de IPs/nombres — **no escanea ni hace ping**
- Hace reverse DNS por defecto
- Muestra todos como "not scanned"
- Útil para verificar IPs incorrectas antes de un scan activo

---

## IPv6 Scan (`-6`)

- Espacio de **128 bits** (vs 32 IPv4)
- 2⁶⁴ direcciones por subred → escaneo completo = **~5 mil millones de años**
- Requiere recopilar IPs IPv6 de tráfico, logs, headers de email

---

## Nmap — Resumen de opciones de scanning

| Opción | Técnica |
|---|---|
| `-sT` | TCP Connect |
| `-sS` | SYN/Stealth |
| `-sX` | Xmas |
| `-sF` | FIN |
| `-sN` | NULL |
| `-sM` | Maimon |
| `-sA` | ACK Flag Probe |
| `-sW` | Window-Based ACK |
| `-sI` | IDLE/IPID |
| `-sU` | UDP |
| `-sY` | SCTP INIT |
| `-sZ` | SCTP COOKIE ECHO |
| `-sL` | List Scan |
| `-sV` | Service Version |
| `-O` | OS Detection |
| `-6` | IPv6 |

---

## ⚠️ Tabla maestra de respuestas por estado de puerto

| Scan | Puerto Abierto | Puerto Cerrado | Puerto Filtrado |
|---|---|---|---|
| **TCP Connect** | Conexión OK | Error | Timeout |
| **SYN/Stealth** | SYN+ACK | RST | Sin respuesta |
| **FIN/NULL/Xmas** | Sin respuesta | RST/ACK | Sin respuesta |
| **UDP** | Sin respuesta | ICMP port unreachable | ICMP unreachable error |
| **SCTP INIT** | INIT+ACK | ABORT | Sin respuesta |
| **SCTP COOKIE ECHO** | Sin respuesta | ABORT | Sin respuesta |
| **ACK (`-sA`)** | RST | RST | Sin respuesta |

### 📝 Pregunta típica CEH

> *"Un atacante quiere escanear sin que quede en logs. ¿Qué técnica usa?"*
> → **SYN/Half-Open Scan (`-sS`)**

> *"¿Qué técnica usa un zombie para ocultar la IP real del atacante?"*
> → **IDLE/IPID Scan (`-sI`)**

> *"Inverse TCP Flag scan no funciona contra. ¿Qué OS?"*
> → **Microsoft Windows** (ignora RFC 793)
