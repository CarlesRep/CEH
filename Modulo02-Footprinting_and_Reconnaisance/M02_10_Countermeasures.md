# M02 — Footprinting Countermeasures

## Por categoría

### Web y servidores
- Configurar web servers para evitar **information leakage**
- Deshabilitar **directory listings**
- Configurar **IIS** para evitar banner grabbing
- Prevenir que buscadores **cacheen** páginas web
- Usar **anonymous registration services**
- Usar **privacy services** en Whois lookup
- Sanitizar datos en registros de Internet — ocultar contacto directo
- Mantener **perfil del dominio privado**

### DNS
- Usar **Split DNS** — separar DNS interno del externo
- Restringir **zone transfers** solo a servidores autorizados

### Geolocalización y RRSS
- Deshabilitar **geo-tagging** en cámaras
- Desactivar geolocalización en móviles cuando no se necesite
- No revelar ubicación ni planes de viaje en RRSS
- Restringir acceso a RRSS desde la red corporativa
- Usar **pseudónimos** en blogs, grupos y foros

### Información corporativa
- No revelar info crítica en press releases, catálogos, informes anuales
- No exponer info estratégica en **tablones o paredes**
- Mantener documentos críticos **offline**
- Solicitar a **archive.org** eliminar historial del sitio

### Acceso y autenticación
- Implementar **MFA**
- Usar **VPN** o servidor detrás de proxy seguro
- Configurar mail servers para **ignorar emails anónimos**
- Deshabilitar/eliminar cuentas de **empleados que abandonaron** la org

### Técnicas y protocolos
- No habilitar protocolos innecesarios
- Usar **TCP/IP e IPsec filters** para defense in depth
- Implementar **captchas y rate limiting** en servicios públicos
- Desplegar **honeypots/honeynets** para atraer y detectar atacantes
- Cifrar y proteger con contraseña la info sensible

### Concienciación
- Formación periódica en **security awareness**
- Entrenamiento contra **social engineering**

---

## ⚠️ Conceptos clave para el examen

| Concepto | Dato clave |
|---|---|
| **Split DNS** | Separa DNS interno del externo — evita exposición de topología interna |
| **Honeypot** | Atrae footprinters y los aleja de sistemas críticos |
| **Banner grabbing** | Se mitiga configurando IIS — oculta versión del servidor |
| **archive.org** | Requiere **solicitud activa** para eliminar historial — NO automático |
| **Geo-tagging** | Deshabilitarlo en cámaras evita geolocalización por metadatos de fotos |
| **Zone transfer** | Restringir a servidores autorizados — si está abierto, el atacante obtiene toda la zona DNS |
