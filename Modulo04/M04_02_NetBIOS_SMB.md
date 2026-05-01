# M04 — NetBIOS y SMB Enumeration

## NetBIOS — Conceptos base

- API original para acceder a recursos LAN — Windows lo usa para file/printer sharing
- Nombre NetBIOS = **16 caracteres ASCII únicos** — 15 = nombre del dispositivo · 16 = tipo de servicio/registro
- Microsoft **no soporta** NetBIOS name resolution para **IPv6**

### Puertos NetBIOS

| Puerto | Proto | Servicio |
|---|---|---|
| **137** | UDP (y TCP) | Name Services (NBNS/WINS) |
| **138** | UDP | Datagram Services |
| **139** | TCP | Session Services |

### NetBIOS Name Table — códigos clave

| Nombre | Código | Tipo | Descripción |
|---|---|---|---|
| `<hostname>` | **00** | UNIQUE | Hostname del equipo |
| `<domain>` | **00** | GROUP | Nombre del dominio |
| `<hostname>` | **03** | UNIQUE | Messenger service (equipo) |
| `<username>` | **03** | UNIQUE | Messenger service (usuario logado) |
| `<hostname>` | **20** | UNIQUE | **Server service** — clave para shares |
| `<domain>` | **1D** | GROUP | Master browser del subnet |
| `<domain>` | **1B** | UNIQUE | Domain master browser (PDC) |
| `<domain>` | **1E** | GROUP | Browser service elections |

> ⚠️ Para acceder a recursos compartidos, el sistema remoto debe tener el **código 20** (Server service)

---

## Nbtstat — Windows utility

**Uso:** troubleshooting de NETBIOS name resolution + enumeración

```
nbtstat [-a <remotename>] [-A <IPaddress>] [-c] [-n] [-r] [-R] [-RR] [-s] [-S]
```

### Parámetros clave

| Parámetro | Función |
|---|---|
| `-a <remotename>` | NetBIOS name table del host remoto por **nombre** |
| `-A <IPaddress>` | NetBIOS name table del host remoto por **IP** |
| `-c` | Contenido del **NetBIOS name cache** (nombres + IPs) |
| `-n` | Nombres registrados **localmente** |
| `-r` | Count de nombres resueltos por broadcast o WINS |
| `-R` | **Purga** la cache y recarga entradas #PRE del Lmhosts |
| `-RR` | Libera y re-registra nombres con el name server |
| `-s` | Sesiones NetBIOS — IPs a nombres NetBIOS |
| `-S` | Sesiones NetBIOS actuales con IPs |

> 🧠 **Mnemotécnia:** `-a` = por nombre · `-A` = por IP (mayúscula = IP)

---

## Herramientas NetBIOS Enumeration

| Herramienta | Función |
|---|---|
| **NetBIOS Enumerator** | Enumera nombres NetBIOS, usernames, dominios y MACs por rango de IPs |
| **Nmap** | Script `nbstat.nse` — recupera nombres NetBIOS y MACs |
| **Global Network Inventory** | Inventario de red |
| **Advanced IP Scanner** | Escaneo rápido |
| **Hyena** | Administración y auditoría |
| **Nsauditor** | Auditoría de seguridad de red |

```bash
# Nmap NetBIOS enumeration
nmap -sV -v --script nbstat.nse <target IP>
```

---

## PsTools — Enumeración de usuarios remotos

Suite de herramientas CLI para gestión y control remoto de sistemas Windows.

| Tool | Función |
|---|---|
| **PsExec** | Ejecuta procesos en sistemas remotos — reemplazo de Telnet |
| **PsFile** | Lista ficheros abiertos remotamente en el sistema local |
| **PsGetSid** | Traduce SIDs a nombres y viceversa — funciona en red |
| **PsKill** | Termina procesos en sistemas remotos — sin instalar cliente |
| **PsInfo** | Información del sistema (kernel, RAM, procesadores, OS) |
| **PsList** | Info de CPU y memoria de procesos |
| **PsLoggedOn** | Usuarios logados local y remotamente — usa HKEY_USERS + NetSessionEnum API |
| **PsLogList** | Vuelca contenido del Event Log local o remoto |
| **PsPasswd** | Cambia contraseñas en sistemas locales/remotos — **no envía en cleartext** |
| **PsShutdown** | Apaga o reinicia equipos locales/remotos — sin instalar cliente |

---

## Net View — Compartidos

```cmd
net view \\<computername>           # Recursos compartidos de un equipo
net view \\<computername> /ALL      # Incluye hidden shares
net view /domain                    # Todos los shares del dominio
net view /domain:<domain name>      # Shares de un dominio específico
```

---

## SMB Enumeration

**SMB** = Server Message Block — protocolo de transporte Windows para file/printer sharing

### Puertos SMB

| Puerto | Proto | Modo |
|---|---|---|
| **445** | TCP/UDP | SMB **directo** sobre TCP (Direct Host) — moderno |
| **137** | UDP | NetBIOS Name Service |
| **138** | UDP | NetBIOS Datagram |
| **139** | TCP | SMB **over NetBIOS** — legacy |

> ⚠️ **139 vs 445:** 139 = requiere NetBIOS (legacy) · 445 = SMB directo sobre TCP (moderno)

### Comandos Nmap para SMB

```bash
nmap -p 445 -A <target IP>                          # OS detection + version + scripts
nmap -p 445 --script smb-protocols <target IP>      # Protocolos y versiones SMB
nmap -p 139 --script smb-protocols <target IP>      # Igual via puerto 139
```

### Herramientas SMB

SMBMap · enum4linux · nullinux · SMBeagle · NetScanTools Pro · Nmap

### ⚠️ Puertos a bloquear (SMB countermeasures)

**TCP:** 88, 139, 445 · **UDP:** 88, 137, 138

### 📝 Pregunta típica CEH

> *"¿Qué opción de Nmap activa OS detection, version detection, scripts y traceroute en un solo comando?"*
> → `-A`

> *"¿Qué utilidad Windows permite ver la tabla de nombres NetBIOS de un host remoto por IP?"*
> → `nbtstat -A <IP>`

> *"¿Qué código NetBIOS indica que el Server service está corriendo en un host?"*
> → **20 (UNIQUE)**
