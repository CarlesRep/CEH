# M02 — Whois, DNS Footprinting y Reverse DNS

## Whois — Conceptos base

- Protocolo query/response — consulta bases de datos de recursos de Internet
- **Puerto: 43 TCP**
- Mantenido por **RIRs (Regional Internet Registries)**

### 3 Modelos de almacenamiento

| Modelo | Clave |
|---|---|
| **Thick Whois** (Distributed) | Almacena info **completa** de todos los registrars |
| **Thin Whois** (Centralized) | Solo almacena el **nombre del servidor** Whois del registrar |
| **Decentralized Whois** | Info completa en **múltiples entidades independientes** |

> ⚠️ **Thick vs Thin:** Thick = datos completos en el propio servidor · Thin = solo apunta al registrar

### Información que devuelve Whois

Nombre de dominio · Registrar · Contacto del propietario · Name servers · NetRange · Fecha de creación/expiración · Última actualización · Estado del dominio · Info de IP

**Herramientas:** whois.domaintools.com · tamos.com · Active Whois · Batch IP Converter (soporta IDNs y IPv6)

---

## RIRs — 5 Registros Regionales

> 🧠 **Mnemotécnia:** **A**mérica **A**frica **A**sia **R**ipa **L**atino
> → **ARIN** · **AFRINIC** · **APNIC** · **RIPE NCC** · **LACNIC**

| RIR | Región |
|---|---|
| **ARIN** | América del Norte |
| **AFRINIC** | África |
| **APNIC** | Asia-Pacífico |
| **RIPE NCC** | Europa |
| **LACNIC** | Latinoamérica y Caribe |

---

## IP Geolocation

**Info obtenible:** País · Región/estado · Ciudad · Lat/Long · ZIP · Zona horaria · ISP · Dominio · Velocidad de conexión · Código IDD · Operador móvil · Elevación

**Uso ofensivo:** social engineering localizado · phishing geolocalizado · servidor comprometido cerca de la víctima · malware específico por área geográfica

**Herramientas:** IP2Location · IP Location Finder · IP Address Geographical Location Finder

> ⚠️ **Whois vs IP Geolocation:** Whois = info de *registro* del dominio/IP · Geolocation = *ubicación física* del target

---

## DNS Footprinting

**Qué revela:** nombres de dominio · nombres de equipos · IPs · hosts clave de la red

### Tipos de registros DNS — memorizar todos

| Registro | Dirección | Descripción |
|---|---|---|
| **A** | nombre → IPv4 | IP del host |
| **AAAA** | nombre → IPv6 | IP IPv6 del host |
| **MX** | — | Servidor de correo del dominio |
| **NS** | — | Name server del host |
| **CNAME** | alias → nombre real | Alias canónico |
| **SOA** | — | Autoridad sobre la zona DNS |
| **SRV** | — | Registros de servicio |
| **PTR** | IP → nombre | DNS inverso |
| **RP** | — | Persona responsable |
| **HINFO** | — | CPU y OS del host |
| **TXT** | — | Texto sin estructura |

> ⚠️ **PTR vs A:** A = nombre→IP · PTR = IP→nombre (inversas)
> ⚠️ **NS vs SOA:** NS = servidor de nombres · SOA = autoridad sobre la zona DNS
> ⚠️ **MX vs SRV:** MX = solo correo · SRV = cualquier servicio

### Herramientas DNS

| Herramienta | Diferencial |
|---|---|
| **SecurityTrails** | Registros DNS **históricos** + actuales (A, AAAA, NS, MX, SOA, TXT) + subdominios por fuerza bruta |
| **Fierce** | Reconocimiento activo + subdominios + IPs no contiguas + conexiones HTTP |
| **DNSdumpster** | Subdominios, IPs, DNS servers |
| **DNSChecker** | Propagación DNS |
| **zdns** | Resolución DNS rápida a gran escala |

> ⚠️ **SecurityTrails vs Fierce:** SecurityTrails = histórico + mapa DNS · Fierce = activo + IPs no contiguas

### Fierce — Comandos clave

```bash
fierce --domain certifiedhacker.com                              # Scan básico
fierce --domain certifiedhacker.com --subdomains write admin mail # Subdominios específicos
fierce --domain certifiedhacker.com --subdomains mail --traverse 10 # IPs contiguas (rango 10)
fierce --domain certifiedhacker.com --subdomains mail --connect  # Probar HTTP
fierce --domain certifiedhacker.com --wide                       # Scan completo
```

---

## Reverse DNS Lookup

| | DNS Lookup | Reverse DNS Lookup |
|---|---|---|
| **Dirección** | Nombre → IP | IP → Nombre |
| **Registro** | A / AAAA | **PTR** |

**Uso ofensivo:** rango de IPs → todos los dominios asociados → mapear infraestructura

### Herramientas

| Herramienta | Uso |
|---|---|
| **DNSRecon** | Reverse lookup por rango de IPs — brute force |
| **Reverse Lookup** (mxtoolbox) | Lookup individual de IP → PTR |
| **puredns** | Resolución y brute force DNS |

```bash
# Reverse DNS lookup sobre rango de IPs
dnsrecon -r 162.241.216.0-162.241.216.255
# -r = rango de IPs (primera-última)
```

### 📝 Pregunta típica CEH

> *"¿Qué tipo de registro DNS se usa en un Reverse DNS Lookup?"*
> → **PTR**

> *"¿En qué puerto escucha el protocolo Whois?"*
> → **Puerto 43 TCP**
