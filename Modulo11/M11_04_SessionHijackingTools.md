# M11_04_SessionHijackingTools.md
## CEH v13 — Módulo 11 | Session Hijacking Tools

---

## 1. Conceptos y definiciones

### 🔴 Herramientas de Session Hijacking — Tabla consolidada

| Herramienta | Tipo | Características clave para el examen |
|---|---|---|
| **Hetty** | HTTP toolkit (security research) | MITM HTTP proxy con logs y búsqueda avanzada · HTTP client para crear/editar peticiones manualmente y repetir peticiones proxied · intercepta requests y responses para revisión manual (editar, enviar/recibir, cancelar) |
| **Caido** | Web security auditing toolkit | Intercepta y visualiza HTTP requests en tiempo real durante la navegación · personalización y testing contra wordlists grandes · modificación automática de peticiones entrantes con **Regex rules** · reenvío manual de peticiones para testing de endpoints |
| **bettercap** | Portable framework (escrito en **Go**) | Reconocimiento y ataques sobre: redes **Wi-Fi**, dispositivos **Bluetooth Low Energy (BLE)**, dispositivos **wireless HID**, redes **IPv4/IPv6** · orientado a: security researchers, red teamers, reverse engineers |
| **Burp Suite** | Web security testing proxy | Suite completa de auditoría web; intercepción y modificación de tráfico HTTP/HTTPS |
| **OWASP ZAP** | Web application security scanner | Scanner de seguridad web open-source; proxy de intercepción |
| **WebSploit Framework** | Exploitation framework | Framework de explotación web |
| **sslstrip** | SSL stripping tool | Degradación de HTTPS a HTTP para capturar credenciales |
| **JHijack** | Session hijacking tool | Herramienta específica de session hijacking |

---

### Herramientas principales — Detalle

#### Hetty
- **URL**: github.com
- **Función central**: MITM HTTP proxy.
- Permite crear y editar HTTP requests manualmente.
- Permite repetir (replay) peticiones previamente proxied.
- Intercepta tanto requests como responses → permite revisión manual completa (editar, enviar/recibir, cancelar).

#### Caido
- **URL**: caido.io
- Visualización de HTTP requests en tiempo real.
- Testing contra **wordlists grandes** (para fuzzing y brute force de parámetros).
- Modificación automática de peticiones entrantes mediante **Regex rules**.
- Reenvío manual de peticiones para testing de endpoints específicos.

#### bettercap
- **URL**: bettercap.org
- Escrito en **Go** → portabilidad.
- **Ámbitos de ataque**: Wi-Fi · BLE (Bluetooth Low Energy) · Wireless HID · IPv4/IPv6.
- **Perfiles de usuario**: security researchers · red teamers · reverse engineers.

---

## 2. Exam Traps ⚠️

⚠️ **[bettercap — escrito en Go]** Dato técnico específico: bettercap está escrito en el lenguaje **Go**. El examen puede preguntar el lenguaje de implementación.

⚠️ **[bettercap — BLE no es WiFi genérico]** bettercap opera sobre Wi-Fi, BLE, wireless HID e IPv4/IPv6. El examen puede presentar "Bluetooth clásico" como opción — incorrecto; bettercap trabaja con **BLE (Bluetooth Low Energy)**.

⚠️ **[Hetty — MITM HTTP proxy, no HTTPS scanner]** Hetty es específicamente un **MITM HTTP proxy** para investigación de seguridad. No confundir con un escáner web activo.

⚠️ **[Caido — Regex rules para modificación automática]** El dato diferenciador de Caido frente a otras herramientas es la modificación automática de peticiones mediante **Regex rules**.

⚠️ **[sslstrip — SSL stripping]** sslstrip degrada conexiones HTTPS a HTTP para capturar tráfico en claro. No es un proxy genérico.

⚠️ **[OWASP ZAP — open source]** El examen puede preguntar qué herramienta de session hijacking/web security es open-source → OWASP ZAP.

---

## 3. Nemotécnicos

### Herramientas principales — "H-C-B"
**"Hetty Caido Bettercap"**
- **H**etty → MITM HTTP proxy + replay
- **C**aido → Regex rules + wordlists + tiempo real
- **B**ettercap → Go + Wi-Fi + BLE + HID + IPv4/IPv6

### bettercap — ámbitos — "W-B-H-I"
**"Wi-Fi Bluetooth HID Internet"**
- **W**i-Fi · **B**LE · **H**ID wireless · **I**Pv4/IPv6

### Herramientas adicionales — "B-O-W-S-J"
**"Burp OWASP WebSploit SSLstrip JHijack"**

---

## 4. Flashcards

**Q:** ¿En qué lenguaje de programación está escrito bettercap?
**A:** Go.

**Q:** ¿Sobre qué tipos de redes y dispositivos puede operar bettercap?
**A:** Redes Wi-Fi · dispositivos Bluetooth Low Energy (BLE) · dispositivos wireless HID · redes IPv4/IPv6.

**Q:** ¿Cuál es la función principal de Hetty?
**A:** MITM HTTP proxy con logs y búsqueda avanzada; permite crear, editar, interceptar y repetir peticiones HTTP manualmente.

**Q:** ¿Qué característica diferencia a Caido de otras herramientas de session hijacking?
**A:** La modificación automática de peticiones entrantes mediante Regex rules, y el testing contra wordlists grandes.

**Q:** ¿Qué herramienta de session hijacking es específicamente un SSL stripping tool?
**A:** sslstrip.

**Q:** ¿Qué herramienta de web security testing de la lista es open-source y ampliamente conocida como escáner web?
**A:** OWASP ZAP.

**Q:** ¿Para qué perfiles de usuario está diseñado bettercap?
**A:** Security researchers, red teamers y reverse engineers.

**Q:** ¿Qué permite hacer Hetty con peticiones previamente proxied?
**A:** Repetirlas (replay de peticiones proxied).

---

## 5. Confusión frecuente

### Hetty vs. Caido vs. Burp Suite
- **Hetty**: MITM HTTP proxy orientado a investigación de seguridad; replay de peticiones; revisión manual.
- **Caido**: auditoría web con Regex rules para modificación automática; testing contra wordlists; tiempo real.
- **Burp Suite**: suite completa de web security testing; la más amplia en funcionalidades; proxy + scanner + intruder + repeater.
- **Criterio**: ¿el enunciado menciona Regex automáticas o wordlists? → Caido. ¿Es MITM proxy con replay manual? → Hetty. ¿Suite completa de auditoría web? → Burp Suite.

### bettercap vs. otras herramientas de red
- **bettercap**: multiprotocolo (Wi-Fi + BLE + HID + IPv4/IPv6); escrito en Go; orientado a red y entornos inalámbricos.
- **OWASP ZAP / Burp Suite**: orientados a aplicaciones web HTTP/HTTPS; no trabajan con Wi-Fi ni BLE directamente.
- **Criterio**: ¿el ataque es sobre redes inalámbricas o Bluetooth? → bettercap. ¿Es sobre tráfico HTTP/HTTPS de aplicación web? → OWASP ZAP / Burp Suite / Hetty / Caido.
