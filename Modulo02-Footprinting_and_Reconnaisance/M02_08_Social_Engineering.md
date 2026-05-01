# M02 — Social Engineering en Footprinting

## Definición

Proceso **no técnico** — explota la naturaleza confiada de las personas para obtener información confidencial. La víctima **no sabe** que está siendo engañada.

**Info obtenible:** credenciales · SSNs · tarjetas de crédito · IPs · servidores · layout de red · OS/software · productos de seguridad

---

## 7 Técnicas principales

| Técnica | Clave | Dato diferencial |
|---|---|---|
| **Eavesdropping** | Intercepta comunicaciones (audio, vídeo, texto, fax, IM) | Pasivo — sin consentimiento |
| **Shoulder Surfing** | Observa pantalla/teclado desde detrás | Efectivo en lugares concurridos |
| **Dumpster Diving** | Busca info en basura física | También: *trashing* |
| **Impersonation** | Se hace pasar por técnico, mensajero, cliente, conserje | Puede combinar con shoulder surfing |
| **Tailgating** | Sigue a persona autorizada físicamente | **Sin consentimiento** del autorizado |
| **Piggybacking** | Igual que tailgating | **Con consentimiento** del autorizado |
| **Reverse Social Engineering** | Se presenta como experto para que la víctima le busque | La víctima inicia el contacto |

---

## Dumpster Diving — ¿Qué se puede encontrar?

Facturas telefónicas · info de contacto · info financiera · código fuente impreso · info operacional · números de cuenta ATM · sticky notes con contraseñas

---

## ⚠️ Pares que confunden en el examen

| Par | Diferencia CLAVE |
|---|---|
| **Tailgating vs Piggybacking** | Tailgating = **sin** consentimiento · Piggybacking = **con** consentimiento |
| **Shoulder Surfing vs Eavesdropping** | Shoulder = observación **visual** · Eavesdropping = interceptación de **comunicaciones** |
| **Dumpster Diving vs Impersonation** | Dumpster = info física en basura · Impersonation = engaño de identidad activo |
| **RRSS Footprinting vs Social Engineering** | RRSS = info *pública disponible* · Social Eng = engañar activamente para obtener más |

### 📝 Pregunta típica CEH

> *"Un atacante se hace pasar por técnico de mantenimiento para acceder a un edificio. ¿Qué técnica es?"*
> → **Impersonation**

> *"Un empleado mantiene abierta la puerta a alguien que dice olvidó su tarjeta. La persona accede sin que el empleado lo sepa. ¿Qué técnica es?"*
> → **Tailgating** (sin consentimiento consciente)

> *"Un atacante entra al edificio porque el empleado autorizado le abre la puerta sabiendo que no tiene acceso. ¿Qué técnica es?"*
> → **Piggybacking** (con consentimiento)
