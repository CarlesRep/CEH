# M04 — Conceptos de Enumeración y Puertos

## ¿Qué es la Enumeración?

Proceso de extraer información mediante **conexiones activas** con el sistema y **consultas dirigidas**.

> ⚠️ Diferencia clave con scanning:
> - **Scanning** = detecta qué hay abierto (pasivo/semi-pasivo)
> - **Enumeración** = extrae información concreta mediante conexión activa (intrusivo)
> - Funciona en entornos **intranet** — puede ser ilegal sin autorización

**Info obtenible:**
- Network resources · Network shares · Routing tables
- Audit y service settings · SNMP y FQDN details
- Machine names · Users y groups · Applications y banners

---

## Técnicas de Enumeración

| Técnica | Cómo funciona |
|---|---|
| **Email IDs** | username@domainname → extrae username del dominio |
| **Default passwords** | Fabricantes asignan passwords por defecto — usuarios no las cambian |
| **Brute force Active Directory** | Error de diseño de AD: distintos mensajes de error según si el usuario existe → permite enumerar usuarios válidos |

> ⚠️ **Brute force AD:** si "logon hours" está habilitado → mensajes de error diferentes para usuarios válidos vs inválidos → enumerable

---

## Puertos y Servicios Enumerables — Tabla maestra

> 🧠 **Los más preguntados en el CEH — memorizar puerto + protocolo + servicio**

| Puerto | Proto | Servicio | Clave |
|---|---|---|---|
| **20/21** | TCP | FTP | 20=datos · 21=control |
| **22** | TCP | SSH / SFTP | SSH=gestión segura · SFTP=transferencia segura (mismo puerto) |
| **23** | TCP | Telnet | **Cleartext** — inseguro |
| **25** | TCP | SMTP | Email · también 2525 y 587 |
| **53** | TCP/UDP | DNS Zone Transfer | UDP=default · TCP=cuando excede 512 octetos |
| **69** | UDP | TFTP | **Connectionless** — sin garantía de entrega |
| **111** | TCP/UDP | RPC Portmapper | Portmapper — detecta endpoints RPC |
| **123** | UDP | NTP | Sincronización de tiempo |
| **135** | TCP/UDP | Microsoft RPC Endpoint Mapper | Vulnerabilidad DoS conocida |
| **137** | UDP | NetBIOS Name Service (NBNS/WINS) | Name resolution |
| **138** | UDP | NetBIOS Datagram Service | Datagramas |
| **139** | TCP | NetBIOS Session Service (SMB over NetBIOS) | File/printer sharing — null sessions |
| **161** | UDP | SNMP | Agente recibe requests |
| **162** | TCP/UDP | SNMP Trap | Agente envía notificaciones al manager |
| **179** | TCP | BGP | Routing tables ISP — misconfiguration → ataques de hijacking |
| **389** | TCP/UDP | LDAP | Directory services |
| **445** | TCP/UDP | SMB over TCP (Direct Host) | SMB sin NetBIOS |
| **500** | UDP | ISAKMP/IKE | VPN — negocia SAs y claves criptográficas |
| **636** | TCP | LDAPS | LDAP seguro (SSL) |
| **2049** | TCP | NFS | Remote file system |
| **3268** | TCP/UDP | Global Catalog Service | LDAP en Domain Controller — toda la org |
| **5060** | TCP/UDP | SIP | VoIP — sin cifrar |
| **5061** | TCP/UDP | SIP-TLS | VoIP — **cifrado con TLS** |

### ⚠️ Pares que confunden en el examen

| Par | Diferencia |
|---|---|
| **SNMP 161 vs 162** | 161 = agente **recibe** requests · 162 = agente **envía** traps/notificaciones |
| **139 vs 445** | 139 = SMB **over NetBIOS** · 445 = SMB **directo sobre TCP** |
| **53 TCP vs UDP** | UDP = default · TCP = failover cuando respuesta > 512 octetos |
| **5060 vs 5061** | 5060 = SIP sin cifrar · 5061 = SIP **con TLS** |
| **SSH vs SFTP** | Ambos en **puerto 22** — SFTP usa un solo puerto (ventaja vs FTP/S) |
| **FTP 20 vs 21** | 20 = datos · 21 = control |
| **Telnet vs SSH** | Telnet = **cleartext** · SSH = cifrado — SSH es el reemplazo seguro |

### 📝 Pregunta típica CEH

> *"¿Qué protocolo usan los agentes SNMP para enviar notificaciones de eventos al manager?"*
> → **SNMP Trap — UDP/TCP 162**

> *"¿Qué puerto usa NFS?"*
> → **TCP 2049**

> *"SFTP usa el mismo puerto que..."*
> → **SSH — puerto 22**
