# AppA_04_NetworkTroubleshooting.md
# CEH v13 — Appendix A: Network Troubleshooting

---

## 1. Conceptos y definiciones

### Mensajes ICMP de error: los que el examen explota 🔴

Este chunk completa la operativa de ICMP iniciada en el chunk anterior. Los mensajes que el examen usa con más frecuencia:

| Tipo ICMP | Nombre | Cuándo se genera |
|---|---|---|
| **0** | Echo Reply | Respuesta a ping |
| **3** | Destination Unreachable | Host/red/puerto inaccesible; fragmentación necesaria pero prohibida; servicios IP no disponibles |
| **5** | Redirect | Existe una mejor ruta hacia el destino (código 0–3) |
| **8** | Echo (Request) | Ping enviado |
| **11** | Time Exceeded | TTL del datagrama llegó a 0 (usado por traceroute) |
| **12** | Parameter Problem | Error en la cabecera IP que impide procesar el datagrama |

**ICMP Echo / Echo Reply** — el campo Protocol en la cabecera IP vale **1** cuando el payload es ICMP. El ping usa Type 8 (Request) y recibe Type 0 (Reply). Code = 0 en ambos casos.

**Destination Unreachable (Type 3)** se genera en tres escenarios distintos:
1. Host, red o puerto no alcanzable.
2. Fragmentación requerida pero el bit "Don't Fragment" está activado — el router no puede reenviar el paquete y notifica al origen.
3. Servicios IP-relacionados no disponibles (FTP, web).

El caso de fragmentación ocurre típicamente al reenviar de **Token Ring a Ethernet** (MTU diferente).

**Time Exceeded (Type 11)**: cada router decrementa el TTL en 1. Cuando llega a 0, el paquete se descarta y el router notifica al origen con Type 11. **Traceroute se basa en este mecanismo** — envía paquetes con TTL incremental (1, 2, 3…) y recolecta los Type 11 de cada hop.

**Redirect (Type 5)**: el gateway envía este mensaje cuando detecta que existe una **mejor ruta** hacia la red destino. Condiciones para enviarlo: el router debe estar configurado para enviar redirects; la ruta alternativa no puede ser otro redirect ni la ruta por defecto; el datagrama no puede ser source-routed; la interfaz de entrada y salida del paquete es la misma.

**Parameter Problem (Type 12)**: errores en la cabecera IP que no están relacionados con el estado del host ni de la red destino, pero impiden procesar el datagrama. Se envía al origen.

**Mensajes de control ICMP** (a diferencia de los de error): no se generan por pérdida de paquetes ni errores. Informan a los hosts sobre condiciones como **congestión de red** o la **existencia de una mejor ruta** (gateway alternativo).

---

### Problemas típicos de red

| Tipo de problema | Causa |
|---|---|
| **Physical** | Cables sueltos o defectuosos |
| **Connectivity** | Fallos de red o configuración de puertos/interfaces en LAN/WAN |
| **Configuration** | Mala configuración de DHCP, DNS o enrutamiento |
| **Software** | Software incompatible o versiones distintas |
| **Traffic Overload** | Tráfico que supera la capacidad de los dispositivos |
| **Network IP** | Configuración errónea de IP, máscara de subred o enrutamiento |

---

### Pasos de troubleshooting 🔴

El libro define **6 pasos** en este orden:
1. Troubleshooting IP Problems
2. Troubleshooting Local Connectivity Issues
3. Troubleshooting Physical Connectivity Issues
4. Troubleshooting Routing Problems
5. Troubleshooting Upper-layer Faults
6. Troubleshooting Wireless Network Connection Issues

**Lógica del orden**: primero se verifica la configuración lógica (IP), luego la conectividad local (ping en misma subred), luego la física (cables, puertos), luego el enrutamiento (traceroute), después los servicios de capa superior (firewall, autenticación, versiones), y por último el caso específico inalámbrico.

**Troubleshooting local connectivity**:
- Misma subred → ping al destino directo.
- Distinta subred → ping al gateway del router.
- Si falla → verificar tabla de enrutamiento.
- IP duplicada → desconectar el dispositivo sospechoso y hacer ping; si responde otro equipo, hay IP duplicada.

**Troubleshooting routing**:
- Usar **traceroute** para localizar el hop problemático.
- Acceder al router problemático con **telnet** y hacer ping desde allí.
- Verificar rutas configuradas con máscara de subred correcta.
- Comprobar bucles de enrutamiento.

**Upper-layer faults** — tabla de problemas frecuentes:

| Problema | Solución |
|---|---|
| Firewall bloquea tráfico | Mover el host para evitar el firewall |
| Servidor o servicio caído | Reemplazar con servidor temporal |
| Problemas de autenticación | Usar software para verificar la autenticación |
| Incompatibilidad de versiones | Actualizar dispositivos a la misma versión |

---

### Herramientas de troubleshooting 🔴

| Herramienta | Plataforma | Función principal |
|---|---|---|
| **ping** | Win/Linux | Verifica accesibilidad de IP/host; usa ICMP Echo Type 8 |
| **tracert** (Windows) / **traceroute** (Linux) | Win/Linux | Traza la ruta de paquetes hop a hop usando TTL incremental |
| **ipconfig** | Windows | Muestra configuración TCP/IP (IP, máscara, gateway) |
| **ipconfig /all** | Windows | Información detallada de todos los adaptadores |
| **ifconfig** | Linux | Equivalente a ipconfig en Linux |
| **nslookup** | Win/Linux | Resuelve nombres DNS a IPs; diagnostica problemas de resolución DNS |
| **netstat** | Win/Linux | Muestra conexiones TCP/IP entrantes y salientes activas |
| **netstat -e** | Windows | Muestra estadísticas de protocolos |
| **pathping** | Windows | Combina ping + tracert; mide pérdida de paquetes en cada hop (test de 25 segundos) |
| **pathping -n** | Windows | Muestra IPs numéricas en lugar de nombres DNS |
| **mtr** | Linux | Equivalente a pathping en Linux |
| **route** | Win/Linux | Muestra y modifica la tabla de enrutamiento activa |
| **PuTTY** | Windows | Cliente SSH/SFTP; genera hashes de contraseñas |
| **Tera Term** | Windows | Automatización de tareas para conexiones remotas; soporta Telnet y SSH |
| **Subnet/IP Calculator** | Web/App | Calcula subredes IPv4/IPv6, rangos de broadcast, hosts disponibles |
| **Speedtest.net** | Web | Mide el ancho de banda disponible (descarga, subida, latencia) |

**Nslookup** se usa específicamente cuando un recurso es accesible por IP pero **no por nombre DNS** — apunta a un fallo de resolución de nombres.

**Netstat** no solo muestra conexiones: sirve para **identificar servicios asociados a puertos** definidos por el usuario, lo que es relevante para detección de backdoors.

**Pathping** funciona en dos fases: primero traza la ruta (como tracert), luego ejecuta un **test de 25 segundos** recopilando la tasa de pérdida de datos en cada router.

**Route** — sintaxis de modificación:
```
route [-p] [dest] [mask subnet] gateway [-if interface]
```
Útil cuando el host tiene múltiples IPs o gateways.

---

## 2. Exam Traps ⚠️

⚠️ **[ICMP Echo: Type 8 vs Type 0]**
El **ping enviado** (request) es Type **8**. La **respuesta** (reply) es Type **0**. El examen puede invertirlos. Echo = 8, Echo Reply = 0.

⚠️ **[ICMP en cabecera IP: Protocol Field = 1]**
Cuando el payload de IP es ICMP, el campo Protocol vale **1** (no 6 como TCP ni 17 como UDP). El examen puede preguntar el valor del campo Protocol para un ping.

⚠️ **[Destination Unreachable por fragmentación]**
Si un paquete tiene el bit **Don't Fragment** activado y necesita fragmentarse para atravesar un enlace (ej. Token Ring → Ethernet), el router lo descarta y envía ICMP Type 3 al origen. No es un fallo de host ni de red — es un fallo de **tamaño de MTU + DF bit**.

⚠️ **[Time Exceeded = TTL llegó a 0, no que expiró el tiempo de sesión]**
ICMP Type 11 no indica que una sesión haya tardado demasiado. Indica que el **TTL del datagrama llegó a cero** en algún router intermedio. Traceroute explota este mensaje enviando paquetes con TTL=1, TTL=2, etc.

⚠️ **[Parameter Problem Type 12 — no es fallo de red]**
Type 12 se envía cuando hay un error en la **cabecera IP** del datagrama que impide procesarlo. No está relacionado con el estado del host destino ni de la red. El examen puede confundirlo con Destination Unreachable.

⚠️ **[ICMP Redirect Type 5 — condiciones]**
No cualquier router envía redirects. Condiciones: el router debe estar **configurado para enviarlos**, la ruta alternativa no puede ser otro redirect ni la ruta por defecto, y el datagrama no puede ser source-routed.

⚠️ **[tracert vs traceroute]**
**tracert** es el comando en **Windows**. **traceroute** es el equivalente en **Linux/Unix**. El examen puede pedir el comando para un SO específico.

⚠️ **[ipconfig vs ipconfig /all]**
`ipconfig` muestra configuración básica. `ipconfig /all` muestra información detallada de **todos los adaptadores**. La distinción importa en preguntas de diagnóstico.

⚠️ **[ifconfig — solo Linux]**
`ifconfig` es el equivalente de `ipconfig` en **Linux**. En Windows no existe (o es obsoleto). El examen puede preguntar cuál comando usar en un sistema Linux para ver la configuración de red.

⚠️ **[nslookup — cuándo usarlo]**
Se usa cuando un recurso es accesible **por IP pero no por nombre DNS**. No sirve para diagnosticar conectividad de red general — eso es ping.

⚠️ **[pathping — test de 25 segundos]**
Pathping espera **25 segundos** recopilando datos de pérdida en cada hop. Es más lento que tracert pero da información de calidad del enlace. El examen puede preguntar qué herramienta mide la pérdida de paquetes en cada hop — pathping (Windows) o mtr (Linux).

⚠️ **[PuTTY — función en el libro]**
El libro describe PuTTY como herramienta para **FTP/SFTP** y generación de hashes de contraseñas. En la práctica es un cliente SSH/Telnet/SCP, pero el libro lo asocia con SFTP. Responder según el libro en el examen.

⚠️ **[Orden de los 6 pasos de troubleshooting]**
El examen puede presentar los pasos desordenados. El orden correcto: IP → Local → Physical → Routing → Upper-layer → Wireless.

---

## 3. Nemotécnicos

### ICMP Types para el examen
**"0-Echo Reply, 3-Unreachable, 5-Redirect, 8-Echo, 11-Exceeded, 12-Parameter"**
Regla de par/impar: **0 y 8** son el par ping (Reply=0, Request=8). **3** = Destino inaccesible. **5** = Redirect. **11** = TTL=0. **12** = Error de cabecera.

### ICMP Protocol Field en IP
Ping → IP Protocol = **1** (ICMP). TCP = 6. UDP = 17.
Secuencia: **1, 6, 17** → ICMP, TCP, UDP.

### Herramientas Windows vs Linux
| Windows | Linux | Función |
|---|---|---|
| ipconfig | ifconfig | Config de red |
| tracert | traceroute | Traza ruta |
| pathping | mtr | Ruta + pérdida |
| — | — | (ping = igual en ambos) |

### 6 pasos de troubleshooting
**"IP Local Physical Routing Upper Wireless"**
→ **"I Like Pigs Resting Under Water"** (poco elegante pero funcional)

---

## 4. Flashcards

**Q:** ¿Qué tipo ICMP corresponde a Echo Request (ping enviado) y cuál a Echo Reply?
**A:** Echo Request = Type **8**. Echo Reply = Type **0**.

---

**Q:** ¿Qué valor tiene el campo Protocol en la cabecera IP cuando el payload es ICMP?
**A:** **1**.

---

**Q:** ¿En qué circunstancia genera un router un ICMP Destination Unreachable relacionado con fragmentación?
**A:** Cuando un datagrama necesita fragmentarse para atravesar un enlace (ej. Token Ring → Ethernet) pero tiene el bit **Don't Fragment** activado. El router lo descarta y notifica al origen.

---

**Q:** ¿Qué mecanismo ICMP usa traceroute para identificar cada hop?
**A:** Envía paquetes con TTL incremental (1, 2, 3…). Cuando el TTL llega a 0 en un router, éste descarta el paquete y devuelve un **ICMP Time Exceeded (Type 11)** al origen.

---

**Q:** ¿Cuándo se genera un ICMP Parameter Problem (Type 12)?
**A:** Cuando hay un error en la **cabecera IP** del datagrama que impide procesarlo, sin que el problema esté relacionado con el estado del host ni de la red destino.

---

**Q:** ¿Cuándo envía un router un ICMP Redirect (Type 5)?
**A:** Cuando detecta que existe una mejor ruta hacia el destino, siempre que: esté configurado para enviar redirects, la ruta alternativa no sea otro redirect ni la ruta por defecto, y el datagrama no sea source-routed.

---

**Q:** ¿Cuál es el orden correcto de los 6 pasos de troubleshooting de red según el libro?
**A:** 1. IP Problems → 2. Local Connectivity → 3. Physical Connectivity → 4. Routing Problems → 5. Upper-layer Faults → 6. Wireless.

---

**Q:** Al hacer troubleshooting local, ¿a qué IP se hace ping si origen y destino están en distintas subredes?
**A:** Al **gateway IP del router**.

---

**Q:** ¿Cómo se detecta una IP duplicada en la red durante el troubleshooting?
**A:** Desconectando el dispositivo sospechoso y haciendo ping a esa IP. Si otro dispositivo responde, hay duplicado → cambiar la IP.

---

**Q:** ¿Qué herramienta Windows combina ping y tracert para medir pérdida de paquetes en cada hop?
**A:** **pathping**. Ejecuta un test de 25 segundos tras trazar la ruta.

---

**Q:** ¿Cuál es el equivalente Linux de pathping?
**A:** **mtr**.

---

**Q:** ¿Cuándo se usa nslookup en lugar de ping para diagnosticar un problema?
**A:** Cuando un recurso es accesible **por IP pero no por nombre DNS** — indica un fallo de resolución DNS, no de conectividad.

---

**Q:** ¿Cuál es el equivalente de `ipconfig` en Linux?
**A:** `ifconfig`.

---

**Q:** ¿Qué muestra `ipconfig /all` que no muestra `ipconfig` solo?
**A:** Información detallada de **todos los adaptadores** de red del sistema.

---

**Q:** ¿Para qué sirve `netstat -e`?
**A:** Muestra estadísticas de varios protocolos (no solo las conexiones activas).

---

**Q:** ¿Qué herramienta se usa para ver y modificar la tabla de enrutamiento en el host?
**A:** `route`. Sintaxis: `route [-p] dest [mask subnet] gateway [-if interface]`.

---

**Q:** ¿Qué herramienta usa el libro para acceder al router problemático durante el troubleshooting de enrutamiento?
**A:** **telnet** (para acceder al router) + **ping** (para verificar conectividad desde él).

---

**Q:** ¿Qué herramienta se usa para diagnosticar un problema de Wi-Fi en Windows según el libro?
**A:** **Windows Network Diagnostics** (Settings → Network & Internet → Wi-Fi).

---

## 5. Confusión frecuente

### tracert vs traceroute
- **tracert**: comando en **Windows**.
- **traceroute**: comando en **Linux/Unix**.
- Ambos funcionan igual: TTL incremental + ICMP Time Exceeded para mapear cada hop.
- **Criterio de decisión**: si el escenario menciona Windows → tracert. Linux/Unix → traceroute.

---

### ping vs nslookup — cuándo usar cada uno
- **ping**: verifica **conectividad de red** (accesibilidad de IP/host). Usa ICMP Echo.
- **nslookup**: diagnostica **resolución DNS** (nombre → IP). Se usa cuando la IP funciona pero el nombre no resuelve.
- **Criterio de decisión**: "¿está el host activo?" → ping. "¿resuelve el nombre correctamente?" → nslookup.

---

### pathping vs tracert — qué aporta cada uno
- **tracert**: solo traza la ruta (identifica hops). Rápido.
- **pathping**: traza la ruta + mide la **tasa de pérdida de paquetes** en cada hop (test de 25 segundos). Más lento pero más informativo.
- **Criterio de decisión**: "localizar el hop problemático" → tracert. "medir calidad del enlace / pérdida de paquetes" → pathping.

---

### ICMP Type 3 vs Type 11 vs Type 12 — tres errores distintos
- **Type 3** (Destination Unreachable): el destino **no puede alcanzarse** (host, red, puerto) o hay problema de fragmentación/DF.
- **Type 11** (Time Exceeded): el **TTL llegó a 0** en un router intermedio. El problema está en la ruta, no en el destino.
- **Type 12** (Parameter Problem): hay un **error en la cabecera IP**. El problema está en el paquete en sí.
- **Criterio de decisión**: destino inaccesible → Type 3. TTL agotado / traceroute → Type 11. Error de formato de cabecera → Type 12.

---

### ICMP mensajes de error vs mensajes de control
- **Mensajes de error** (Type 3, 11, 12): resultado de paquetes perdidos o errores durante la transmisión.
- **Mensajes de control** (Type 5 Redirect): no son errores; informan sobre condiciones de red como congestión o mejor ruta disponible.
- **Criterio de decisión**: si la pregunta dice "resultado de paquete perdido" → mensaje de error. Si dice "informar de mejor ruta" o "congestión" → mensaje de control.
