# M04 — Enumeration Countermeasures

## SNMP Countermeasures

### Medidas principales
- **Eliminar el agente SNMP** o desactivar el servicio si no es necesario
- **Cambiar community strings por defecto** (public/private → valores complejos y únicos)
- **Actualizar a SNMPv3** — encripta passwords y mensajes
- **Bloquear puertos TCP/UDP 161** (y 162)
- No usar modo **"NoAuthNoPriv"** — no cifra comunicaciones
- Usar modo **"AuthNoPriv"** — usa MD5 y SHA para protección adicional
- Restringir acceso solo a IPs/redes legítimas mediante **ACLs**
- Mantener SNMP en **VLAN separada** y segura
- Implementar **RBAC** para comunidades SNMP
- No misconfigrar con permisos **read-write** cuando solo se necesita read
- Auditar logs de tráfico SNMP regularmente

> ⚠️ **Versiones:** SNMPv1/v2c = community strings sin cifrar · SNMPv3 = autenticación + cifrado

---

## LDAP Countermeasures

- Usar **SSL o STARTTLS** para cifrar tráfico LDAP (por defecto va sin cifrar)
- Usar username diferente al email + habilitar **account lockout**
- Restringir acceso a AD mediante software como **Citrix**
- Implementar **NTLM, Kerberos** u autenticación básica
- Desactivar **anonymous binds** al directorio LDAP
- Desplegar **canary accounts** y **decoy groups con "Admin"** en el nombre
- Habilitar **MFA** para acceder a directorios LDAP
- Configurar **ACLs** para limitar visibilidad según credenciales
- Log de todas las queries y modificaciones LDAP
- Colocar LDAP servers en **segmento de red seguro**
- Forzar **strong password policies** para cuentas con acceso LDAP

> ⚠️ **Decoy groups:** crear grupos con "Admin" en el nombre — atacantes los buscan específicamente

---

## NFS Countermeasures

- Implementar permisos apropiados en file systems exportados (read/write restringido)
- **Bloquear puerto NFS 2049** con firewall
- Configurar correctamente `/etc/smb.conf`, `/etc/exports`, `/etc/hosts.allow`
- Mantener **root_squash ON** en `/etc/exports` — requests como root del cliente no son confiables
- Implementar **NFS tunneling a través de SSH** para cifrar tráfico
- No ejecutar **suid y sgid** en file systems exportados
- Implementar **autenticación Kerberos** para NFS
- Migrar a **NFSv4** — soporta Kerberos, cifrado y autenticación mejorada
- Log de requests de acceso al NFS server
- Aplicar principio de **least privilege**

> ⚠️ **root_squash** = opción crítica — evita que un cliente con root pueda actuar como root en el servidor

---

## SMTP Countermeasures

- **Ignorar mensajes a destinatarios desconocidos**
- **Deshabilitar EXPN, VRFY, y RCPT TO** o restringirlos a usuarios autenticados
- **Deshabilitar el open relay**
- No compartir info de IPs internas o mail relay en respuestas
- Implementar **SPF, DKIM, y DMARC**
- Limitar número de conexiones aceptadas desde una fuente (anti brute-force)
- Usar **TLS** para cifrar comunicación SMTP
- Configurar **rate limiting**
- Usar **ML solutions** para identificar spammers
- Configurar SMTP para dar **información mínima en errores**
- Usar **ACLs** para restringir ciertos comandos SMTP

> 🧠 **SPF + DKIM + DMARC** = tríada de autenticación de email — memorizar los 3 juntos

---

## SMB Countermeasures

- **Bloquear puertos:** TCP 88, 139, 445 · UDP 88, 137, 138
- Deshabilitar SMB en servidores de Internet (bastion hosts) desactivando:
  - "Client for Microsoft Networks"
  - "File and Printer Sharing for Microsoft Networks"
- Configurar **RestrictNullSessAccess = 1** en el registro:
  `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters`
- Usar **SMBv3 o superior** — evitar SMBv1 (obsoleto y vulnerable)
- Implementar **firmas digitales** en transmisión SMB
- Configurar **ACLs** para restringir acceso a shares SMB
- Habilitar **Windows Firewall** o endpoint protection
- Usar **VPNs seguras** para acceso remoto
- Monitorizar con **NGFWs** y sistemas de análisis de comportamiento

> ⚠️ **SMBv1** = obsoleto y vulnerable — siempre usar SMBv3+

---

## DNS Countermeasures

### Medidas principales
- **Restringir zone transfers** solo a slave nameservers autorizados
- **Deshabilitar DNS recursion** — mitiga amplification y poisoning attacks
- **Audit DNS zones** regularmente
- **Usar servidores separados** para funciones authoritative y resolving
- **Usar DNS aislados** — no alojar DNS junto a application servers
- Implementar **DNSSEC** — solo acepta requests firmadas digitalmente
- Usar **DoH (DNS-over-HTTPS) o DoT (DNS-over-TLS)** para cifrar queries
- Randomizar **source ports** (no siempre UDP 53) y query IDs
- Implementar **Split DNS architecture** — DNS interno separado del externo
- Usar **DNS change lock** para restringir cambios no autorizados
- Habilitar **rate limiting** para queries por IP
- Activar **DNS logging y monitoring**
- Implementar **anomaly detection** para patrones inusuales

### Medidas adicionales
- No publicar hosts privados en DNS zones públicas
- Eliminar/purgar registros DNS antiguos o no usados
- Usar `/etc/hosts` para subdomains de desarrollo en lugar de registros DNS
- Desplegar **DNS Firewalls** con threat intelligence
- Usar **premium DNS registration** para ocultar HINFO del público

> 🧠 **DoH vs DoT:** ambos cifran DNS — DoH = sobre HTTPS (puerto 443) · DoT = sobre TLS (puerto 853)

---

## Tabla resumen de countermeasures por protocolo

| Protocolo | Medida clave #1 | Medida clave #2 | Puerto a bloquear |
|---|---|---|---|
| **SNMP** | Actualizar a SNMPv3 | Cambiar community strings | TCP/UDP 161 |
| **LDAP** | Usar SSL/STARTTLS | Desactivar anonymous binds | — |
| **NFS** | root_squash ON | Bloquear puerto 2049 | TCP 2049 |
| **SMTP** | Deshabilitar VRFY/EXPN | Implementar SPF+DKIM+DMARC | — |
| **SMB** | Usar SMBv3+ | Bloquear TCP 139/445 | TCP 139, 445 · UDP 137, 138 |
| **DNS** | Restringir zone transfers | Deshabilitar recursion | — |

### 📝 Preguntas típicas CEH

> *"¿Qué versión de SNMP debe usarse para solucionar problemas de seguridad de community strings?"*
> → **SNMPv3**

> *"¿Qué opción debe estar activa en /etc/exports para que los requests como root del cliente no sean confiables?"*
> → **root_squash**

> *"¿Qué tres estándares de autenticación de email deben implementarse juntos?"*
> → **SPF + DKIM + DMARC**

> *"¿Qué valor del registro Windows restringe acceso anónimo en SMB?"*
> → **RestrictNullSessAccess = 1**
