# M03 — Conceptos Base de Network Scanning

## Tipos de Scanning — 3 tipos

| Tipo | Qué descubre |
|---|---|
| **Port Scanning** | Puertos abiertos y servicios en ejecución |
| **Network Scanning** | Hosts activos e IPs en la red |
| **Vulnerability Scanning** | Vulnerabilidades conocidas — usa catálogo de exploits + scanning engine |

## Objetivos del Network Scanning

- Descubrir hosts activos, IPs y puertos abiertos
- Identificar **OS y arquitectura** del sistema (fingerprinting)
- Identificar **servicios y versiones** en ejecución
- Detectar vulnerabilidades explotables
- Mapear topología: dispositivos, routers, switches

---

## TCP Flags — 6 flags (1 bit cada una, total 6 bits)

> 🧠 **Mnemotécnia:** **S**iempre **A**ctivo **P**ush **U**rgente **F**in **R**eset → **S·A·P·U·F·R**

| Flag | Función clave |
|---|---|
| **SYN** | Inicia conexión — notifica nuevo número de secuencia |
| **ACK** | Confirma recepción — identifica siguiente secuencia esperada |
| **PSH** | Empuja datos al buffer de la aplicación — se activa al inicio y fin de transferencia |
| **URG** | Datos urgentes — procesamiento inmediato, detiene todo lo demás |
| **FIN** | Fin de transmisión — cierra la conexión establecida por SYN |
| **RST** | Error en conexión — aborta; usado por atacantes para identificar puertos abiertos |

> ⚠️ **SYN scanning** usa principalmente: **SYN, ACK y RST**

---

## TCP Three-Way Handshake

```
Cliente                            Servidor
  |--- SYN (SEQ#10) ------------->|
  |<-- SYN+ACK (ACK#11, SEQ#142) -|
  |--- ACK (ACK#143, SEQ#11) ---->|
  |         CONEXIÓN ABIERTA      |
```

**Terminación:** Sender → FIN o RST · Receiver → ACK + FIN · Conexión cerrada

---

## Puertos críticos — memorizar para el examen

> 🧠 Los más preguntados:

| Puerto | Servicio | Puerto | Servicio |
|---|---|---|---|
| **21/tcp** | FTP | **110/tcp** | POP3 |
| **22/tcp** | SSH | **111/tcp,udp** | RPC portmapper |
| **23/tcp** | Telnet | **119/tcp** | NNTP |
| **25/tcp** | SMTP | **123/udp** | NTP |
| **53/tcp,udp** | DNS | **137/tcp,udp** | NETBIOS Name Service |
| **67/udp** | BOOTP Server | **138/tcp,udp** | NETBIOS Datagram |
| **68/udp** | BOOTP Client | **139/tcp,udp** | NETBIOS Session |
| **69/udp** | TFTP | **143/tcp,udp** | IMAP |
| **80/tcp,udp** | HTTP | **161/tcp,udp** | SNMP |
| **88/tcp,udp** | Kerberos | **162/tcp,udp** | SNMP Trap |
| **443/tcp** | HTTPS | **389/tcp,udp** | LDAP |
| **445/tcp,udp** | Microsoft DS (SMB) | **500/udp** | ISAKMP/IKE |
| **514/tcp** | Shell (rsh) | **1080/tcp,udp** | SOCKS Proxy |
| **514/udp** | Syslog | **1433/tcp,udp** | MS SQL Server |
| **515/tcp,udp** | Printer (lpd) | **1723/tcp,udp** | PPTP |
| **194/tcp,udp** | IRC | **2049/tcp,udp** | NFS |
| **5060/tcp,udp** | SIP | **6000-6063/tcp,udp** | X Window System |
| **6667/tcp** | IRC | **3389/tcp** | RDP (Remote Desktop) |
