# M04 — NTP y NFS Enumeration

## NTP — Conceptos base

- **Network Time Protocol** — sincroniza relojes de equipos en red
- Puerto: **UDP 123**
- Precisión en Internet: error < **10 ms** · En LAN: hasta **200 μs**

### Información obtenible por NTP

- Lista de hosts conectados al NTP server
- IPs de clientes, nombres de sistema y OSs
- **IPs internas** si el NTP server está en la **DMZ** ← dato crítico para el examen

---

## NTP Enumeration Commands — 4 comandos

> 🧠 **Mnemotécnia:** **N**o **N**ecesitas **N**ada **N**uevo → **ntpdate · ntptrace · ntpdc · ntpq**

| Comando | Función |
|---|---|
| **ntpdate** | Recopila muestras de tiempo de múltiples fuentes |
| **ntptrace** | Sigue la cadena de NTP servers hasta la fuente primaria — traza la ruta |
| **ntpdc** | Consulta estado y estadísticas del demonio ntpd — cada NTP server conectado |
| **ntpq** | Monitoriza operaciones del demonio ntpd — rendimiento y lista de peers |

### Parámetros comunes

| Parámetro | Función |
|---|---|
| `-4` | Fuerza resolución DNS a IPv4 |
| `-6` | Fuerza resolución DNS a IPv6 |
| `-n` | Muestra solo IPs (no hostnames) |
| `-p` | Lista de peers y resumen de estados |
| `-i` | Modo interactivo |
| `-d` | Modo debug |

### Ejemplo ntptrace

```bash
ntptrace
# localhost: stratum 4, offset 0.0019529...
# 10.10.0.1: stratum 2, offset 0.0114273...
# 10.10.1.1: stratum 1, offset 0.0017698...
```

> El **stratum** indica la distancia al reloj de referencia — stratum 1 = más preciso

---

## NTP Enumeration Tools

| Herramienta | Función |
|---|---|
| **PRTG Network Monitor** | Detalles del SNTP server, tiempo de respuesta, sensores activos, sincronización |
| **Nmap** | Scripts NSE para NTP |
| **Wireshark** | Captura y análisis de tráfico NTP |
| **udp-proto-scanner** | Escaneo de protocolos UDP |
| **NTP Server Scanner** | Bytefusion — escaneo de NTP servers |

> **Nota:** En muchas distribuciones Linux, ntpd se ha integrado con **Chrony (chronyd)**

---

## NFS — Conceptos base

- **Network File System** — usuarios acceden a ficheros remotos como si fueran locales
- Puerto: **TCP 2049**
- Usa **RPC (Remote Procedure Call)** para routing y procesamiento
- Proceso: **exporting** (servidor) → **mounting** (cliente)
- `/etc/exports` = lista de clientes autorizados en el servidor NFS
- Autenticación en NFS < v4 = solo **IP del cliente** (vulnerable a IP spoofing)

> ⚠️ Si NFS está mal configurado → atacante puede spoolear su IP y obtener acceso completo a los shares

### Información obtenible por NFS

- Directorios exportados
- Lista de clientes conectados con sus IPs
- Datos compartidos asociados a cada IP

---

## NFS Enumeration — Comandos

```bash
rpcinfo -p <Target IP>              # Escanea puerto NFS 2049 y servicios RPC
showmount -e <Target IP>            # Lista ficheros y directorios compartidos
```

---

## NFS Enumeration Tools

| Herramienta | Función |
|---|---|
| **RPCScan** | Comunica con servicios RPC — verifica misconfigurations en NFS shares |
| **SuperEnum** | Enumeración básica de cualquier puerto abierto — script ./superenum |

```bash
# RPCScan
python3 rpc-scan.py <Target IP> --rpc

# SuperEnum
./superenum → ingresa fichero Target.txt con IPs
```

---

## ⚠️ Distinciones NTP vs NFS

| | NTP | NFS |
|---|---|---|
| **Puerto** | UDP 123 | TCP 2049 |
| **Protocolo** | UDP | TCP |
| **Función** | Sincronización de tiempo | Sistema de ficheros remoto |
| **RPC usado** | No | Sí (portmapper 111) |

### 📝 Preguntas típicas CEH

> *"¿Qué comando NTP sigue la cadena de servidores hasta la fuente primaria de tiempo?"*
> → **ntptrace**

> *"¿Qué comando muestra la lista de directorios compartidos en un servidor NFS?"*
> → `showmount -e <IP>`

> *"¿Qué fichero en el servidor NFS contiene la lista de clientes autorizados?"*
> → **/etc/exports**

> *"¿Qué información crítica puede obtenerse de un servidor NTP ubicado en la DMZ?"*
> → **IPs internas de la red**
