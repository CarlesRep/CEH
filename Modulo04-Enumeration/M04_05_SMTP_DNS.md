# M04 — SMTP y DNS Enumeration

## SMTP — Conceptos base

- Puerto principal: **TCP 25** · también: **2525** y **587**
- Usado con POP3 e IMAP para email
- Usa **MX records** (DNS) para dirigir el correo

### 3 Comandos SMTP para enumerar usuarios

| Comando | Función | Respuesta válido | Respuesta inválido |
|---|---|---|---|
| **VRFY** | Valida si un usuario existe | `250 user@server` | `550 User unknown` |
| **EXPN** | Muestra direcciones reales de aliases/listas | `250 user@server` | `550 User unknown` |
| **RCPT TO** | Define destinatarios del mensaje | `250 Recipient ok` | `550 User unknown` |

> ⚠️ Los servidores SMTP responden **diferente** para usuarios válidos e inválidos → permite enumeración

### Telnet a SMTP (proceso manual)

```bash
telnet 192.168.168.1 25
HELO x
VRFY Jonathan          # 250 = existe · 550 = no existe
EXPN Jonathan          # 250 = alias real · 550 = no existe
MAIL FROM:Jonathan
RCPT TO:Ryder          # 250 = válido · 550 = inválido
```

---

## SMTP Enumeration — Herramientas y comandos

### Nmap NSE para SMTP

```bash
nmap -p 25,365,587 --script=smtp-commands <Target IP>    # Lista comandos SMTP disponibles
nmap -p 25 --script=smtp-open-relay <Target IP>          # Identifica open relays
nmap -p 25 --script=smtp-enum-users <Target IP>          # Enumera usuarios del SMTP
```

### Metasploit — SMTP enumeration

```bash
# Módulo: auxiliary/scanner/smtp/smtp_enum
use auxiliary/scanner/smtp/smtp_enum
set RHOST <target IP>
set USER_FILE /usr/share/metasploit-framework/data/wordlists/unix_users.txt
run
# Usa VRFY para validar usuarios del wordlist contra el servidor
```

### smtp-user-enum

```bash
smtp-user-enum.pl [options] (-u username|-U file) (-t host|-T file-of-targets)

# Opciones clave:
# -M mode: EXPN, VRFY, o RCPT TO (default: VRFY)
# -u user: usuario único a verificar
# -U file: fichero con lista de usuarios
# -t host: host SMTP objetivo
# -p port: puerto TCP (default: 25)
# -D dom: dominio para crear email addresses
```

### Herramientas SMTP

| Herramienta | Función |
|---|---|
| **NetScanTools Pro** | SMTP Email Generator — extrae parámetros de header, flags, log de sesión |
| **smtp-user-enum** | Enumera usuarios OS-level en Solaris via VRFY/EXPN/RCPT TO |
| **Metasploit** | Módulo smtp_enum — wordlist-based |
| **Nmap** | NSE scripts smtp-commands, smtp-open-relay, smtp-enum-users |

---

## DNS Zone Transfer Enumeration

### Concepto

- **Zone Transfer** = proceso de copiar la zona DNS del servidor primario al secundario
- Si el servidor permite zone transfers a cualquiera → atacante obtiene: nombres DNS, hostnames, IPs, aliases, usernames

### Comandos

```bash
# dig (Linux)
dig ns <target domain>                               # Obtiene name servers del dominio
dig @<nameserver domain> <target domain> axfr        # Intenta zone transfer

# nslookup (Windows)
nslookup
set querytype=soa
<target domain>                                       # Obtiene SOA (Start of Authority)
/ls -d <nameserver domain>                           # Intenta zone transfer

# DNSRecon
dnsrecon -t axfr -d <target domain>                  # Prueba todos los NS servers
# -t = tipo de enumeración · axfr = zone transfer · -d = dominio objetivo
```

---

## DNS Cache Snooping

**Objetivo:** determinar qué sitios han visitado recientemente los usuarios del sistema

### 2 Métodos

| Método | Cómo | Indica visita si |
|---|---|---|
| **Non-recursive** | RD bit = 0 (`+norecurse`) | Registro presente en cache → `NOERROR` |
| **Recursive** | RD bit = 1 (`+recurse`) | TTL recibido < TTL inicial → ya estaba en cache |

```bash
# Non-recursive
dig @<DNS server IP> <target domain> A +norecurse

# Recursive
dig @<DNS server IP> <target domain> A +recurse
```

---

## DNSSEC Zone Walking

**Objetivo:** obtener registros internos si la zona DNS no está bien configurada

- Explota vulnerabilidad de **zone enumeration** en DNSSEC
- **NSEC3** = versión mejorada que usa **hashes criptográficos** para prevenir la enumeración

### Herramientas DNSSEC Zone Walking

| Herramienta | Función |
|---|---|
| **LDNS (ldns-walk)** | Enumera zona DNSSEC y obtiene registros DNS |
| **DNSRecon** | Zone enumeration + NSEC zone enumeration |
| **nsec3map** | Zone walking sobre NSEC3 |
| **nsec3walker** | Zone walking |
| **DNSwalk** | Zone walking |

```bash
# LDNS
ldns-walk @<DNS server IP> <target domain>

# DNSRecon zone walking
dnsrecon -d <target domain> -z
```

---

## OWASP Amass — DNS Enumeration

```bash
amass enum -d <Target Domain>                                    # Enumera DNS + subdominios completo
amass enum -passive -d <Target Domain> -src                      # Solo pasivo
amass enum -active -d <Target Domain> /usr/share/wordlists/amass/all.txt  # Activo + brute-force
amass track -config /root/amass/config.ini -dir amass4owasp -d <Target> -last 2  # Comparar últimos 2 scans
amass db -dir amass4owasp -list                                  # Mostrar resultados de DB
amass viz -d3 -dir amass4owasp                                   # Gráfico d3-force HTML
```

---

## Nmap para DNS y DNSSEC

```bash
nmap --script=broadcast-dns-service-discovery <Target>           # Servicios disponibles
nmap -T4 -p 53 --script dns-brute <Target>                       # Subdominios + IPs (*A*=IPv4, *AAAA*=IPv6)
nmap -Pn -sU -p 53 --script=dns-recursion 192.168.1.150         # Verifica si recursión habilitada
nmap -sU -p 53 --script dns-nsec-enum \
    --script-args dns-nsec-enum.domains=eccouncil.org <target>   # DNSSEC NSEC enumeration
```

---

## ⚠️ Distinciones clave

| Par | Diferencia |
|---|---|
| **VRFY vs EXPN** | VRFY = valida un usuario · EXPN = muestra dirección real de alias/lista |
| **Zone Transfer vs Cache Snooping** | Zone Transfer = copia toda la zona · Cache Snooping = detecta sitios visitados recientemente |
| **NSEC vs NSEC3** | NSEC = vulnerable a zone walking · NSEC3 = usa hashes criptográficos para prevenirlo |
| **Non-recursive vs Recursive snooping** | Non-recursive = bit RD=0 · Recursive = bit RD=1, examina TTL |

### 📝 Preguntas típicas CEH

> *"¿Qué comando SMTP muestra las direcciones reales de listas de correo?"*
> → **EXPN**

> *"¿Qué herramienta de Metasploit enumera usuarios SMTP?"*
> → `auxiliary/scanner/smtp/smtp_enum`

> *"¿Qué opción de dig realiza una zone transfer?"*
> → `axfr`

> *"¿Qué versión de DNSSEC previene la enumeración con hashes criptográficos?"*
> → **NSEC3**
