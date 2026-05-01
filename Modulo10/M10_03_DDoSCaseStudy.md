# M10_03_DDoSCaseStudy.md
## CEH v13 — Módulo 10 | DDoS Case Study

---

## 1. Conceptos y definiciones

### Escenario DDoS con HOIC — "Volunteers"

Un vector de DDoS poco convencional: el atacante no necesita comprometer máquinas. En su lugar:

1. El atacante aloja la herramienta **HOIC (High Orbit Ion Cannon)** en un servidor propio o comprometido.
2. Publica el enlace de descarga en redes sociales y buscadores (Twitter, Facebook, Google).
3. Usuarios que **voluntariamente** desean participar en el ataque descargan HOIC → se les llama **"volunteers"**.
4. Los volunteers se conectan al atacante a través de un **canal IRC** y esperan instrucciones.
5. El atacante ordena saturar el servidor objetivo (p. ej., PayPal, MasterCard, PAYBACK) con múltiples peticiones.
6. El servidor objetivo queda saturado e inaccesible para usuarios legítimos.

**Distinción clave**: a diferencia del modelo zombie clásico, los volunteers actúan conscientemente. No son víctimas involuntarias — son participantes activos que descargaron la herramienta por propia voluntad.

---

### Distribución de botnets — Métodos de reclutamiento

Los atacantes expanden sus botnets anunciando enlaces de descarga maliciosos a través de:
- Blogs · motores de búsqueda · redes sociales · emails.
- **Fake updates** y **alertas de seguridad falsas** para engañar a la víctima.

Objetivo: aumentar el tamaño de la red de ataque de forma rápida y efectiva.

---

### 🔴 Dispositivos móviles Android como botnets

Los dispositivos Android no seguros se han convertido en objetivo primario para ampliar botnets debido a su alta vulnerabilidad a malware.

**Vectores de infección**:
- Apps maliciosas en Google Play Store.
- **Drive-by downloads** (descarga al visitar una web).
- Third-party app stores.

**Flujo de ataque**:
1. El atacante vincula un servidor malicioso al fichero **APK** (Android Package).
2. Cifra el APK y elimina permisos y funcionalidades innecesarias.
3. Distribuye el paquete malicioso en third-party app stores.
4. La víctima descarga e instala la app.
5. El dispositivo de la víctima queda bajo control del atacante e integrado en el **mobile botnet**.
6. El mobile botnet se usa para DDoS y **web injections**.

**Tipos de malware móvil**: Trojans · bots · RATs (Remote Access Trojans).

---

### 🔴 Caso Real: HTTP/2 'Rapid Reset' Attack — Google Cloud (2023)

#### Datos clave (preguntables en examen)

| Dato | Valor |
|---|---|
| **Pico del ataque** | **398 millones de requests per second (rps)** |
| **Ataque previo (referencia)** | 46 millones rps |
| **Factor de escala** | **7,5 veces** mayor que el ataque anterior |
| **Duración pico** | **2–3 minutos** |
| **Periodo del ataque** | Finales de agosto – finales de septiembre **2023** |
| **CVE asignado** | **CVE-2023-44487** |
| **Severidad CVSS** | **7.5 / 10** (High severity) |
| **Protocolo explotado** | **HTTP/2** |
| **Técnica** | **Rapid Reset** mediante stream multiplexing |

#### Mecanismo técnico

HTTP/2 soporta **multiplexing**: gestión de hasta **100 streams activos simultáneos** a través de una única conexión TCP. El atacante explotó esta capacidad para:
1. Iniciar series consecutivas de solicitudes de stream.
2. Resetear cada stream inmediatamente (**RST flags**).
3. Generar un flujo masivo de paquetes TCP con RST que saturan y resetean el servidor objetivo.

Resultado: disrupción significativa de servicios sin necesidad de mantener conexiones abiertas de larga duración.

#### Respuesta de Google

- Mitigación en el **perímetro de la red** de Google mediante capacidad edge masiva.
- Actualización de proxies y sistemas de defensa con contramedidas específicas.
- La misma infraestructura hardware/software usada para los servicios de Google se proporcionó a los clientes de **Google Cloud Application Load Balancer** y **Cloud Armor**.
- **Respuesta colaborativa**: coordinación con otros cloud providers y mantenedores de software que usan HTTP/2.
- Compartición de inteligencia de amenazas en tiempo real.
- Resultado: parches y técnicas de mitigación adoptadas por múltiples proveedores de infraestructura.
- **Responsible disclosure coordinada** de las nuevas metodologías de ataque.

---

## 2. Exam Traps ⚠️

⚠️ **[398 millones rps — dato numérico exacto]** El examen puede preguntar el pico del ataque HTTP/2 Rapid Reset. La respuesta exacta es **398 millones de requests per second**. No confundir con 46 millones (el ataque anterior de referencia).

⚠️ **[7,5 veces mayor — factor de escala]** El ataque fue 7,5 veces más grande que el anterior récord (46 rps → 398 rps). El examen puede presentar otros multiplicadores como distractores.

⚠️ **[CVE-2023-44487 — CVSS 7.5]** El examen puede preguntar el CVE o la puntuación CVSS de la vulnerabilidad HTTP/2 Rapid Reset. CVE-2023-44487, severidad **High**, CVSS **7.5/10**.

⚠️ **[HTTP/2 — 100 streams por conexión TCP]** El detalle técnico que hace posible el ataque: HTTP/2 permite hasta **100 streams activos simultáneos** en una sola conexión TCP. El examen puede preguntar este número.

⚠️ **[Volunteers vs. Zombies]** Los volunteers en el escenario HOIC actúan **conscientemente y voluntariamente**. No son víctimas de malware. El examen puede presentarlos como zombies — incorrecto.

⚠️ **[HOIC — canal de coordinación IRC]** La coordinación entre el atacante y los volunteers se realiza a través de un **canal IRC**. Si el examen pregunta el canal de comunicación en el escenario HOIC → IRC.

⚠️ **[Android botnet — APK + cifrado]** El flujo específico: el atacante vincula el servidor malicioso al APK, lo **cifra**, elimina permisos innecesarios y lo distribuye en third-party stores. El cifrado es un paso explícito en el CEH.

⚠️ **[Web injections — uso adicional del mobile botnet]** Además de DDoS, el mobile botnet se usa para **web injections**. El examen puede omitir este uso como distractor.

⚠️ **[Duración del pico — 2-3 minutos]** El ataque fue de larga duración (agosto–septiembre 2023) pero el pico de intensidad duró solo **2–3 minutos**.

---

## 3. Nemotécnicos

### HTTP/2 Rapid Reset — datos numéricos clave
**"398 = 7.5 × 46 · CVE-2023-44487 · CVSS 7.5 · 100 streams · 2-3 min"**
- **398** millones rps (pico)
- **7,5** veces el anterior
- **46** millones rps (referencia anterior)
- **CVE-2023-44487**
- **CVSS 7.5**
- **100** streams por conexión TCP
- Pico: **2–3** minutos

### Flujo HOIC con volunteers — "H-A-V-IRC-F"
**"Hacker Anuncia, Volunteers se unen al IRC, Flood"**
- **H**OIC alojado en servidor
- **A**nuncio en redes sociales/buscadores
- **V**olunteers descargan HOIC
- Conectan via **IRC**
- **F**lood al objetivo

### Flujo Android botnet — "APK-C-D-I-M"
**"APK Cifrado Distribuido → Instalado → Mobile botnet"**
- Malware vinculado al **APK**
- **C**ifrado + eliminación de permisos
- **D**istribuido en third-party store
- Víctima lo **I**nstala
- Dispositivo integrado en **M**obile botnet

---

## 4. Flashcards

**Q:** ¿Qué son los "volunteers" en el escenario de DDoS con HOIC?
**A:** Usuarios que conscientemente y voluntariamente descargan la herramienta HOIC y participan en el ataque DDoS siguiendo instrucciones del atacante. No son víctimas de malware — actúan deliberadamente.

**Q:** ¿A través de qué canal se coordinan el atacante y los volunteers en el escenario HOIC?
**A:** Un canal IRC.

**Q:** ¿Cuál fue el pico máximo de requests per second en el ataque HTTP/2 Rapid Reset contra Google Cloud?
**A:** 398 millones de requests per second (rps).

**Q:** ¿Cuántas veces mayor fue el ataque HTTP/2 Rapid Reset respecto al ataque previo de referencia?
**A:** 7,5 veces mayor (398 millones rps vs. 46 millones rps anteriores).

**Q:** ¿Qué CVE fue asignado a la vulnerabilidad HTTP/2 Rapid Reset y cuál es su puntuación CVSS?
**A:** CVE-2023-44487, clasificado como High Severity con CVSS de 7.5/10.

**Q:** ¿Cuántos streams simultáneos permite HTTP/2 por conexión TCP y por qué es relevante para el ataque Rapid Reset?
**A:** Hasta 100 streams activos simultáneos. El atacante los iniciaba y reseteaba consecutivamente con RST flags, generando un flood masivo de paquetes TCP.

**Q:** ¿Cuánto duró el pico del ataque HTTP/2 Rapid Reset?
**A:** 2–3 minutos.

**Q:** ¿Qué pasos específicos sigue el atacante para infectar dispositivos Android y añadirlos a un mobile botnet?
**A:** 1) Vincula servidor malicioso al APK. 2) Cifra el APK y elimina permisos innecesarios. 3) Distribuye en third-party app store. 4) Víctima descarga e instala → dispositivo integrado en mobile botnet.

**Q:** Además de DDoS, ¿para qué otro uso específico se emplea el mobile botnet según el CEH?
**A:** Web injections.

**Q:** ¿Qué métodos usa el atacante para distribuir botnets y reclutar nuevas máquinas?
**A:** Blogs, motores de búsqueda, redes sociales, emails, fake updates y alertas de seguridad falsas con enlaces de descarga maliciosos.

**Q:** ¿Cómo respondió Google al ataque HTTP/2 Rapid Reset?
**A:** Mitigación en el perímetro mediante capacidad edge; actualización de proxies y defensas; respuesta colaborativa con otros cloud providers y mantenedores HTTP/2; compartición de inteligencia en tiempo real; responsible disclosure coordinada; parches distribuidos a múltiples proveedores.

---

## 5. Confusión frecuente

### Volunteers (HOIC) vs. Zombies (botnet clásico)
- **Volunteers**: participantes conscientes y voluntarios; descargaron HOIC deliberadamente; coordinados por IRC; no están comprometidos por malware.
- **Zombies**: equipos infectados sin conocimiento del propietario; controlados por C&C; propietario es víctima involuntaria.
- **Criterio**: ¿la persona sabe que está participando en el ataque? Sí → Volunteer. No → Zombie.

### 398 millones rps vs. 46 millones rps
- **398 millones rps**: pico del ataque HTTP/2 Rapid Reset de septiembre 2023 (el récord).
- **46 millones rps**: el ataque de referencia anterior que fue superado 7,5 veces.
- **Criterio**: el examen puede presentar ambos números. El ataque en cuestión (Rapid Reset) → 398. El anterior → 46.

### HTTP/2 Rapid Reset vs. DDoS genérico por volumen
- **Rapid Reset**: explota la característica de multiplexing de HTTP/2 (100 streams/conexión); usa RST flags para resetear streams consecutivamente; efecto devastador con conexiones relativamente pocas pero muy densas.
- **DDoS genérico por volumen**: satura mediante puro volumen de tráfico/conexiones sin explotar una característica específica del protocolo.
- **Criterio**: si el enunciado menciona HTTP/2, multiplexing o RST flags → Rapid Reset.
