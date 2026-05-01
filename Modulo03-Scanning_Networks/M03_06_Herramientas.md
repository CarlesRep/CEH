# M03 — Herramientas de Scanning

## Nmap — Opciones completas

```bash
nmap <options> <Target IP>
```

### Host Discovery
| Opción | Función |
|---|---|
| `-PR` | ARP ping (default en LAN) |
| `-PE` | ICMP ECHO ping |
| `-PP` | ICMP Timestamp ping |
| `-PM` | ICMP Address Mask ping |
| `-PS [puerto]` | TCP SYN ping (default 80) |
| `-PA [puerto]` | TCP ACK ping (default 80) |
| `-PU [puerto]` | UDP ping (default 40125) |
| `-PO` | IP Protocol ping |
| `-sn` | Solo host discovery — sin port scan |
| `--disable-arp-ping` | Deshabilita ARP ping por defecto |

### Port Scanning
| Opción | Técnica |
|---|---|
| `-sT` | TCP Connect/Full-Open |
| `-sS` | SYN/Stealth/Half-Open |
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

### Detection y OS
| Opción | Función |
|---|---|
| `-sV` | Service Version detection |
| `-O` | OS detection |
| `-sC` | NSE scripts default |
| `--script <nombre>` | Script específico NSE |
| `--script smb-os-discovery` | OS via SMB |
| `-A` | Aggressive: OS + versión + scripts + traceroute |
| `-6` | IPv6 |
| `-6 -O` | IPv6 OS fingerprinting |

### Evasión
| Opción | Función |
|---|---|
| `-D RND:N` | N decoys aleatorios |
| `-D d1,d2,ME,d4` | Decoys manuales |
| `-g` / `--source-port` | Source port manipulation |
| `--spoof-mac 0` | MAC aleatoria |
| `--spoof-mac [Vendor]` | MAC de vendor |
| `--spoof-mac [MAC]` | MAC manual |
| `--randomize-hosts` | Randomizar orden (grupos de 16384) |
| `--badsum` | Enviar bad checksums |
| `--ttl [time]` | TTL-based scan |

### Timing y Rendimiento
| Opción | Función |
|---|---|
| `-T<0-5>` | Timing (0=paranoid, 5=insane) |
| `-v` | Verbose (medir bandwidth) |
| `-p <puertos>` | Puertos específicos |

---

## Hping3 — Comandos completos

```bash
hping3 <options> <Target IP>
```

### Flags de protocolo
| Flag | Significado |
|---|---|
| `-1` | ICMP mode |
| `-2` / `--udp` | UDP mode |
| `-8` / `--scan` | Scan mode (rango de puertos) |
| `-9` | Listen mode |
| `-A` | ACK flag |
| `-S` | SYN flag |
| `-F` | FIN flag |
| `-P` | PUSH flag |
| `-U` | URG flag |
| `-Q` | Recopilar sequence numbers |
| `-a` | Spoofar IP origen |
| `--flood` | Envío lo más rápido posible |
| `--rand-dest` | Destino aleatorio |
| `-I` | Interfaz de red |
| `--tcp-timestamp` | Añadir TCP timestamp |

### Comandos clave con función

```bash
hping3 -1 10.0.0.25                              # ICMP ping
hping3 -A 10.0.0.25 -p 80                        # ACK scan en puerto 80
hping3 -2 10.0.0.25 -p 80                        # UDP scan en puerto 80
hping3 192.168.1.103 -Q -p 139                   # Recopilar TCP sequence numbers
hping3 -S 72.14.207.99 -p 80 --tcp-timestamp     # SYN + TCP timestamp (bypassear FW)
hping3 -8 50-60 -S 10.0.0.25 -V                  # SYN scan puertos 50-60
hping3 -F -P -U 10.0.0.25 -p 80                  # FIN+PUSH+URG scan en puerto 80
hping3 -1 10.0.1.x --rand-dest -I eth0           # ICMP ping sweep subred completa
hping3 -9 HTTP -I eth0                           # Interceptar tráfico HTTP (listen)
hping3 -S 192.168.1.1 -a 192.168.1.254 -p 22 --flood  # SYN flood (DoS) con IP spoofada
hping3 www.certifiedhacker.com -a 7.7.7.7        # IP spoofing
```

### Respuestas Hping por scan

| Scan | Puerto abierto | Puerto cerrado |
|---|---|---|
| **ACK scan** | RST response | Sin respuesta |
| **UDP scan** | Sin respuesta | ICMP port unreachable |
| **FIN/PUSH/URG** | Sin respuesta | RST response |

---

## Otras Herramientas

| Herramienta | Función |
|---|---|
| **Metasploit** | Framework pentest — exploits, payloads, pivoting, reporting |
| **NetScanTools Pro** | Investigación, troubleshooting y monitorización — IPv4/IPv6 |
| **Angry IP Scanner** | Ping sweep multithreaded — NetBIOS info — exporta CSV/TXT/XML |
| **ExtraHop** | Detección de scanning activo — descubre IoT — analiza SSL/TLS cifrado |
| **Snort** | IDS/IPS open source — signatures frecuentemente actualizadas |
| **sx** | Scanner rápido |
| **RustScan** | Scanner ultra-rápido en Rust |
| **Splunk Enterprise Security** | SIEM — correlación y detección |
| **Vectra Detect** | Detección de amenazas en red |
| **IBM Security QRadar XDR** | XDR integrado |

### 📝 Pregunta típica CEH

> *"¿Qué opción de Nmap randomiza el orden de los hosts en grupos de 16384?"*
> → `--randomize-hosts`

> *"¿Qué flag de Hping3 se usa para enviar un ICMP ping?"*
> → `-1`

> *"¿Qué flag de Hping3 activa el modo listen?"*
> → `-9`
