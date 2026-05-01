
## SNMP — Conceptos base

- Application-layer protocol sobre **UDP**
- Gestiona routers, hubs, switches, firewalls, printers, servers
- **Puerto 161** = agente recibe requests · **Puerto 162** = agente envía traps al manager

### 2 Componentes SNMP

| Componente | Rol |
|---|---|
| **SNMP Agent** | En el dispositivo de red — responde requests |
| **SNMP Management Station** | Envía requests y recibe traps |

### 2 Community Strings (contraseñas SNMP)

| Tipo | Acceso | Visibilidad |
|---|---|---|
| **Read Community String** | Solo lectura — ver configuración | **Public** |
| **Read/Write Community String** | Lectura y escritura — modificar configuración | **Private** |

> ⚠️ Si quedan en **default** → atacante puede leer/modificar configuración del dispositivo

---

## SNMP — Operaciones PDU

| Operación | Dirección | Función |
|---|---|---|
| **Get Request** | Manager → Agent | Recupera valor de variable específica |
| **GetNext Request** | Manager → Agent | Siguiente variable en el árbol MIB |
| **Set Request** | Manager → Agent | Modifica valor de variable en el agente |
| **GetBulk Request** | Manager → Agent | Gran volumen de datos en una sola request — **SNMPv2+** |
| **Response** | Agent → Manager | Responde a Get/Set/GetBulk |
| **Inform Request** | Agent → Manager | Info no solicitada sobre eventos — manager-to-manager |
| **Trap** | Agent → Manager | Alerta no solicitada — eventos críticos (reboot, link failure) |

> ⚠️ **Trap vs Inform:** ambos son no solicitados — Trap = unidireccional · Inform = confirmado

---

## MIB — Management Information Base

- Base de datos virtual con todos los objetos que SNMP gestiona
- Estructura **jerárquica** — elementos identificados por **OIDs** (Object Identifiers)
- OID = nombre numérico único en el árbol MIB

### MIBs principales de Windows

| MIB | Qué monitoriza |
|---|---|
| **DHCP.MIB** | Tráfico entre DHCP servers y hosts remotos |
| **HOSTMIB.MIB** | Recursos del host |
| **LNMIB2.MIB** | Servicios de workstation y server |
| **MIB_II.MIB** | Gestión de Internet TCP/IP |
| **WINS.MIB** | Windows Internet Name Service |

---

## SNMP Enumeration — Comandos

### SnmpWalk

```bash
snmpwalk -v1 -c public <Target IP>                              # SNMPv1 — ver todos los OIDs
snmpwalk -v2c -c public <Target IP>                             # SNMPv2c
snmpwalk -v2c -c public <Target IP> hrSWInstalledName           # Software instalado
snmpwalk -v2c -c public <Target IP> hrMemorySize                # RAM disponible
snmpwalk -v2c -c public <Target IP> <OID> <New Value>           # Cambiar valor de OID
snmpwalk -v2c -c public <Target IP> sysContact <New Value>      # Cambiar sysContact
```

### Nmap NSE para SNMP

```bash
nmap -sU -p 161 --script=snmp-processes <Target IP>    # Lista procesos SNMP + puertos
nmap -sU -p 161 --script=snmp-sysdescr <Target IP>     # Tipo de servidor y OS
nmap -sU -p 161 --script=snmp-win32-software <Target IP> # Apps en la máquina
```

> ⚠️ SNMP usa **UDP** → nmap necesita `-sU`

---

## Herramientas SNMP

| Herramienta | Función diferencial |
|---|---|
| **snmp-check** | Open-source GPL — output human-readable — Windows, Unix, appliances, printers |
| **SoftPerfect Network Scanner** | Ping, ports, shared folders, WMI, SNMP, HTTP, SSH, PowerShell — exporta XML/JSON |
| **OpUtils** | ManageEngine |
| **Network Performance Monitor** | SolarWinds |
| **PRTG Network Monitor** | Paessler |

---

## LDAP — Conceptos base

- Protocolo para acceder a **directory services distribuidos** (ej. Active Directory)
- Estructura jerárquica similar a organigrama empresarial
- Usa **DNS** para lookups rápidos
- Puerto **389** TCP/UDP (LDAP) · Puerto **636** TCP (LDAPS — secure)
- Cliente conecta a **DSA (Directory System Agent)** — transmisión en formato **BER**

> ⚠️ Un atacante puede hacer queries **anónimas** a LDAP para obtener: usernames, addresses, departamentos, nombres de servidores

---

## LDAP Enumeration — Comandos

### Manual con Python (ldap3)

```python
import ldap3
server = ldap3.Server('Target IP', get_info=ldap3.ALL, port=389)
connection = ldap3.Connection(server)
connection.bind()                                    # True = conexión OK
server.info                                          # Naming contexts, dominio
connection.search(search_base='DC=DOMAIN,DC=DOMAIN',
    search_filter='(&(objectClass=*))',
    search_scope='SUBTREE', attributes='*')          # Todos los objetos
connection.search(..., search_filter='(&(objectClass=person))',
    attributes='userPassword')                       # Dump de passwords
```

### ldapsearch (CLI)

```bash
ldapsearch -h <Target IP> -x                                    # Autenticación simple
ldapsearch -h <Target IP> -x -s base namingcontexts             # Naming contexts
ldapsearch -h <Target IP> -x -b "DC=htb,DC=local"              # Info del dominio
ldapsearch -h <Target IP> -x -b "DC=htb,DC=local" '(objectClass=Employee)'   # Clase Employee
ldapsearch -x -h <Target IP> -b "DC=htb,DC=local" "objectclass=*"            # Todos los objetos
ldapsearch -h <Target IP> -x -b "DC=htb,DC=local" \
    '(objectClass=Employee)' sAMAccountName sAMAccountType      # Usuarios de una clase
```

### Nmap NSE para LDAP

```bash
# Brute-force LDAP authentication
nmap -p 389 --script ldap-brute \
    --script-args 'ldap.base="cn=users,dc=CEH,dc=com"' <Target IP>
```

### Nmap para verificar puertos LDAP

```bash
# Verificar si LDAP está escuchando en 389 (LDAP) o 636 (LDAPS)
nmap -p 389,636 <Target IP>
```

---

## Herramientas LDAP

| Herramienta | Función |
|---|---|
| **Softerra LDAP Administrator** | GUI — AD, Novell, Netscape — enumera username, email, departamento |
| **ldapsearch** | CLI — búsquedas con filtros RFC 4515 |
| **AD Explorer** | Microsoft — explorar AD |
| **LDAP Admin Tool** | GUI multiplataforma |
| **LDAP Account Manager** | Gestión de cuentas LDAP |

---

## ⚠️ Distinciones SNMP vs LDAP

| | SNMP | LDAP |
|---|---|---|
| **Puerto** | UDP 161 (requests) / 162 (traps) | TCP/UDP 389 / 636 (secure) |
| **Protocolo** | UDP | TCP/UDP |
| **Gestiona** | Dispositivos de red | Directory services (AD) |
| **Auth default** | Community strings (public/private) | Anónima posible |

### 📝 Preguntas típicas CEH

> *"¿Qué versión de SNMP encripta passwords y mensajes?"*
> → **SNMPv3**

> *"¿Qué community string de SNMP permite modificar configuración de dispositivos?"*
> → **Read/Write Community String (private)**

> *"¿Qué puerto usa LDAP por defecto?"*
> → **389 TCP/UDP**

> *"Un atacante quiere listar el software instalado en un host vía SNMP. ¿Qué comando usa?"*
> → `snmpwalk -v2c -c public <IP> hrSWInstalledName`
