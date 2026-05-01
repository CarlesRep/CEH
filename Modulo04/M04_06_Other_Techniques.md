# M04 — Otras Técnicas de Enumeración

## IPsec Enumeration

**IPsec** = tecnología VPN más común (gateway-to-gateway y host-to-gateway)

**Componentes:** ESP (Encapsulating Security Payload) · AH (Authentication Header) · IKE (Internet Key Exchange)

**ISAKMP** = parte de IKE — establece, negocia, modifica y elimina SAs — **UDP 500**

### Proceso de enumeración IPsec

```bash
# 1. Scan inicial para detectar ISAKMP (presencia de VPN gateway)
nmap -sU -p 500 <target IP>

# 2. Fingerprinting con ike-scan para info detallada
ike-scan -M <target gateway IP>
```

### ike-scan — Capacidades

| Función | Descripción |
|---|---|
| **Discovery** | Hosts corriendo IKE en un rango IP |
| **Fingerprinting** | Implementación IKE — 2 métodos: UDP backoff + Vendor ID |
| **Transform enumeration** | Atributos IKE phase 1: encryption y hash algorithm |
| **User enumeration** | Usernames VPN válidos (en algunos sistemas) |
| **Pre-shared key cracking** | IKE Aggressive Mode — usa ike-scan + psk-crack |

**Info obtenible:** encryption algorithm · hashing algorithm · authentication type · key distribution algorithm · SA LifeDuration

---

## VoIP Enumeration

**VoIP** = usa infraestructura Internet para voz y video
**SIP (Session Initiation Protocol)** = protocolo principal de VoIP

### Puertos SIP

| Puerto | Proto | Uso |
|---|---|---|
| **5060** | TCP/UDP | SIP — tráfico sin cifrar |
| **5061** | TCP/UDP | SIP-TLS — tráfico **cifrado** |
| 2000, 2001 | TCP/UDP | Cisco CallManager |

### Info obtenible por VoIP enumeration

- VoIP gateway/servers
- IP-PBX systems
- User-Agent IP addresses
- User extensions (softphones/VoIP phones)

### Ataques derivados

DoS attacks · Session hijacking · Caller ID spoofing · Eavesdropping · SPIT (Spam over Internet Telephony) · **Vishing** (VoIP phishing)

### Herramientas VoIP

| Herramienta | Función |
|---|---|
| **Svmap** | Scanner open-source — identifica SIP devices y PBX servers — puertos default y no-default |
| **Metasploit SIP Username Enumerator** | Escanea extensiones numéricas de VoIP phones |

```bash
svmap <target network range/IP>    # Enumera SIP devices en la red
```

**Svmap capabilities:**
- Identifica SIP devices en puertos default y no-default
- Escanea grandes rangos
- Hace ring a todos los teléfonos simultáneamente (método INVITE)

---

## RPC Enumeration

**RPC (Remote Procedure Call)** = tecnología para programas distribuidos cliente/servidor

**Portmapper** = escucha en **TCP/UDP 111** — detecta endpoints RPC disponibles

> ⚠️ En redes con firewall, el portmapper suele estar filtrado → atacantes escanean rangos amplios de puertos para localizar RPC services directamente

### Comandos Nmap para RPC

```bash
nmap -sR <target IP/network>        # Scan de RPC services
nmap -T4 -A <target IP/network>     # Aggressive scan (OS + versión + scripts)
```

**Herramientas:** NetScanTools Pro (RPC Info tool — detecta portmapper en puerto 111)

---

## Unix/Linux User Enumeration — 3 comandos

| Comando | Función | Sintaxis |
|---|---|---|
| **rusers** | Lista usuarios logados en máquinas remotas de la red local (similar a `who` remoto) | `rusers [-a] [-l] [-u\|-h\|-i] [Host]` |
| **rwho** | Lista usuarios en todos los hosts de la red local — requiere rwho daemon | `rwho [-a]` |
| **finger** | Info detallada: login name, real name, terminal, idle time, login time, oficina | `finger [-l] [-m] [-p] [-s] [user@host]` |

### Opciones finger

| Opción | Qué muestra |
|---|---|
| `-s` | Login name, real name, terminal, idle time, login time, office, phone |
| `-l` | Todo lo de `-s` + home dir, shell, mail status, .plan, .project, .pgpkey, .forward |
| `-p` | Como `-l` pero **sin** mostrar .plan, .project, .pgpkey |
| `-m` | Previene matching de usernames |

> ⚠️ **rusers vs rwho:** rusers = similar a `who` pero para red · rwho = también lista red pero requiere rwho daemon

---

## Telnet y SSH Enumeration

### Telnet (puerto 23)
- **Inseguro** — transmite credenciales en **cleartext**
- Usado para: banner grabbing de SSH y SMTP · brute-force de credenciales · port-forwarding attacks
- Principalmente en redes privadas

### SSH (puerto 22)
- Alternativa segura a Telnet
- Atacantes pueden hacer: **brute-force de credenciales SSH**
- SFTP también usa puerto 22

---

## ⚠️ Resumen de técnicas y puertos

| Técnica | Puerto | Protocolo | Info clave |
|---|---|---|---|
| IPsec/IKE | **UDP 500** | ISAKMP | ike-scan para fingerprinting |
| VoIP/SIP | **5060/5061** | TCP/UDP | Svmap + Metasploit |
| RPC | **TCP/UDP 111** | Portmapper | nmap -sR |
| Unix users | — | rusers/rwho/finger | Comandos CLI |
| SMB | **TCP 445 / TCP 139** | TCP | nmap -p 445 -A |

### 📝 Preguntas típicas CEH

> *"¿Qué herramienta se usa para fingerprinting de IKE/IPsec VPN?"*
> → **ike-scan**

> *"¿Qué puerto usa ISAKMP para establecer SAs en una VPN?"*
> → **UDP 500**

> *"¿Qué herramienta open-source identifica SIP devices y PBX servers?"*
> → **Svmap**

> *"¿Qué comando Unix muestra info detallada de usuarios incluyendo .plan y shell?"*
> → `finger -l`

> *"¿En qué puerto escucha el portmapper de RPC?"*
> → **TCP/UDP 111**
