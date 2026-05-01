# M03 — Network Scanning Countermeasures

## Ping Sweep Countermeasures

- Configurar firewalls para **bloquear ICMP echo requests** de fuentes desconocidas
- Usar **IDS/IPS** (ej. Snort) para detectar ping sweeps
- **Terminar conexión** con hosts que envíen más de **10 ICMP ECHO requests**
- En **DMZ:** permitir solo `ICMP ECHO_REPLY`, `HOST UNREACHABLE`, `TIME EXCEEDED`
- Limitar tráfico ICMP con **ACLs** a IPs específicas del ISP
- Implementar **rate limiting** para paquetes ICMP
- Segmentar red en **subredes aisladas**
- Usar **rangos privados + NAT** para ocultar IPs internas

---

## Port Scanning Countermeasures

- Firewall debe **inspeccionar datos del paquete** — no solo TCP header
- Ejecutar port scanning **contra la propia red** para verificar detección
- Mantener **firmware actualizado** de router, IDS y firewall
- Mantener **mínimo de puertos abiertos** — bloquear específicamente: **135-159, 256-258, 389, 445, 1080, 1745, 3268**
- Bloquear **mensajes ICMP type-3 salientes** en border routers
- Bloquear técnicas de **source routing**
- Usar **TCP wrappers** para limitar acceso por dominio/IP
- Usar **proxy servers** para bloquear paquetes fragmentados o malformados
- Redirigir port scans a **honeypots**
- Usar **IPS** para blacklistear IPs con intentos de port scan
- Implementar **port knocking** para ocultar puertos abiertos
- Usar **NAT** para ocultar IPs internas
- Implementar **VLANs** para aislar tipos de tráfico
- Implementar **egress filtering** para controlar tráfico saliente

---

## Banner Grabbing Countermeasures

### Deshabilitar/cambiar banners

- Mostrar **banners falsos**
- Desactivar servicios innecesarios
- Eliminar extensiones de archivo que revelan tecnología (`.asp`, `.aspx`)

**Configuración Apache:**
- `ServerSignatureOff` en `httpd.conf`
- Cambiar `ServerTokens` de `Full` a **`Prod`**
- Usar `mod_headers` para cambiar banner

**Configuración IIS:**
- Cambiar `RemoveServerHeader` de `0` a `1` en `UrlScan.ini`
- Modificar `AlternateServerName` a valor falso

**Desactivar HTTP methods:** `Connect`, `Put`, `Delete`, `Options`

### Otras medidas

- Filtrar puertos con **packet filtering**
- Usar **IDS/IPS** para detectar banner grabbing
- Reemplazar HTTP, FTP, Telnet con **HTTPS, SFTP, SSH**
- Usar **TLS** para cifrar banner durante el handshake

---

## IP Spoofing Detection — 3 técnicas

| Técnica | Cómo | Cuándo |
|---|---|---|
| **Direct TTL Probes** | TTL respuesta ≠ TTL esperado = spoofado | Atacante en subred diferente |
| **IP Identification Number** | IPID respuesta no próxima al probe = spoofado | Incluso en misma subred |
| **TCP Flow Control** | Datos tras window=0 = spoofado | Durante handshake |

---

## IP Spoofing Countermeasures

- No usar **autenticación solo por IP**
- **Ingress filtering** — ACLs que bloquean IPs fuera del rango
- **Egress filtering** — bloquea salida de IPs externas
- Usar **ISNs aleatorios**
- **IPSec** — autenticación, integridad, confidencialidad
- Migrar a **IPv6** · Usar **VPN** en redes públicas

---

## ⚠️ Conceptos clave para el examen

| Concepto | Dato clave |
|---|---|
| **Port knocking** | Oculta puertos abiertos — el puerto solo se abre tras secuencia correcta |
| **Honeypot** | Atrae scanners y los aleja de sistemas reales |
| **ServerTokens Prod** | En Apache — oculta versión del servidor en banners |
| **TCP wrappers** | Limita acceso a servicios por dominio/IP |
| **Egress filtering** | Controla tráfico saliente — identifica hosts internos maliciosos |
| **Puertos a bloquear** | 135-159, 256-258, 389, 445, 1080, 1745, 3268 |
