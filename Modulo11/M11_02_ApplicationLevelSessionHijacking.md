# M11_02_ApplicationLevelSessionHijacking.md
## CEH v13 — Módulo 11 | Application-Level Session Hijacking

---

## 1. Conceptos y definiciones

### Marco general

El application-level session hijacking opera sobre sesiones HTTP. El atacante roba o predice un session token válido para obtener acceso no autorizado a un servidor web o crear nuevas sesiones no autorizadas. Los métodos de obtención son tres:

- **Stealing**: robo mediante acceso físico, sniffing (Wireshark, Riverbed Packet Analyzer Plus) o extracción de ficheros/memoria.
- **Guessing**: análisis de variables observables en el session ID para identificar patrones; solo efectivo contra algoritmos de generación débiles.
- **Brute forcing**: prueba sistemática de todas las permutaciones posibles; con DSL el atacante puede generar hasta **1.000 session IDs por segundo**; más efectivo cuando el algoritmo es no aleatorio.

> **Session prediction attack**: nombre específico del brute-force cuando el rango de valores predicables del session ID es muy pequeño.

---

### 🔴 Métodos de compromiso del session token — 12 técnicas

| Técnica | Mecanismo principal |
|---|---|
| **Session Sniffing** | Interceptar tráfico HTTP con Wireshark/Riverbed para extraer session IDs |
| **Predictable Session Token** | Analizar patrones en tokens (secuenciales, timestamp, PRNG débil) para predecir el siguiente |
| **MITM Attack** | Dividir la conexión TCP en dos (cliente-atacante / atacante-servidor); leer, modificar e insertar datos |
| **Man-in-the-Browser (MitB)** | Troyano instalado entre el navegador y sus mecanismos de seguridad; modifica DOM y transacciones |
| **Cross-site Scripting (XSS)** | Inyectar JavaScript malicioso en páginas dinámicas para robar cookies/session IDs |
| **Cross-site Request Forgery (CSRF)** | Explotar la confianza del servidor en el navegador del usuario para ejecutar acciones no autorizadas |
| **Session Replay Attack** | Capturar el authentication token y reenviarlo al servidor para suplantar al usuario |
| **Session Fixation** | El atacante establece previamente un session ID y obliga a la víctima a usarlo; ataque antes del login |
| **CRIME Attack** | Explotar la compresión DEFLATE de cookies en TLS/HTTPS/SPDY para deducir el valor de la cookie |
| **Forbidden Attack** | Reutilización de nonce criptográfico en AES-GCM durante el TLS handshake para generar claves y hacer MITM |
| **Session Donation Attack** | El atacante dona su propio session ID a la víctima; la víctima introduce sus datos en la cuenta del atacante |
| **PetitPotam Hijacking** | (cubierto en chunks posteriores) |

---

### Session Sniffing

- Herramientas: **Wireshark**, **Riverbed Packet Analyzer Plus**.
- El session token puede estar en: **cookie** (header HTTP), **URL**, **body** del request HTTP.
- El atacante sniffa el tráfico cliente-servidor → extrae el session ID → se hace pasar por la víctima enviando ese session ID al servidor antes que la víctima.

---

### 🔴 Predictable Session Token — Patrones de análisis

| Tipo de patrón | Mecanismo | Ejemplo |
|---|---|---|
| **Sequential tokens** | Los tokens se emiten de forma secuencial (001, 002, 003…); el atacante observa la serie y predice el siguiente sumando 1 | Token 1003 → siguiente: 1004 |
| **Timestamp-based tokens** | El token incluye un timestamp predecible | `user123-20240611T1234` → predecible con fecha y hora actuales |
| **Small token space** | Espacio de tokens pequeño → brute force factible | Tokens 0000–9999 = 10.000 valores posibles |
| **No rate limiting** | Sin límite de intentos → brute force indefinido | 1.000 tokens/min sin bloqueo |
| **Predictable PRNG** | PRNG mal sembrado o con semilla predecible (p. ej., tiempo actual) → el atacante replica el proceso de generación | PRNG con semilla = timestamp → replicable |

**Flujo de ataque para predecir session token**:
1. Adquirir el session ID actual y conectar a la app.
2. Aplicar brute-force o calcular el siguiente session ID.
3. Modificar el valor en cookie/URL/hidden form field y asumir la identidad del siguiente usuario.

---

### MITM Attack (en contexto de session hijacking)

- Divide la conexión TCP en dos: cliente→atacante y atacante→servidor.
- El atacante puede leer, modificar e insertar datos fraudulentos en la comunicación interceptada.
- Target en HTTP: la conexión TCP entre cliente y servidor.

---

### 🔴 Man-in-the-Browser (MitB) Attack

**Diferencia clave con MITM**: MitB usa un **Troyano** instalado previamente entre el navegador y sus mecanismos de seguridad (no intercepta el tráfico de red, actúa dentro del navegador).

**Objetivo principal**: robo financiero manipulando transacciones de banca online.

**Por qué bypasea SSL/PKI/2FA**: porque actúa dentro del navegador, donde los controles de seguridad ya han sido completados. Todo parece funcionar con normalidad para el usuario.

**Flujo del ataque MitB (14 pasos)**:
1. El Troyano infecta el SO o la aplicación.
2. Instala código malicioso (extension files) en la configuración del navegador.
3. Al reiniciar el navegador, las extension files se cargan.
4. Las extensions registran un handler para cada visita a una página web.
5. Al cargar una página, la extension compara la URL con una lista de sitios objetivo.
6. El usuario hace login de forma segura en el sitio web.
7. Al detectar una carga de página específica, la extension registra un event handler para el botón.
8. Cuando el usuario hace clic en el botón, la extension usa el **DOM interface** para extraer todos los datos de los campos del formulario y modifica los valores.
9. El navegador envía el formulario con los valores modificados al servidor.
10. El servidor recibe los valores modificados sin poder distinguirlos de los originales.
11. El servidor ejecuta la transacción y genera un recibo.
12. El navegador recibe el recibo de la transacción modificada.
13. El navegador muestra el recibo con los detalles **originales** (no los modificados).
14. El usuario cree que la transacción original llegó al servidor sin interceptación.

---

### XSS Attack (Cross-site Scripting) — En contexto de session hijacking

- El atacante inyecta JavaScript malicioso en una página dinámica vulnerable.
- Lenguajes/tecnologías explotables: JavaScript, VBScript, ActiveX, HTML, Flash.
- La página ejecuta el script en el sistema del usuario → captura cookies/session IDs → los envía al atacante.

**Script ejemplo del CEH**:
```html
<SCRIPT>alert(document.cookie);</SCRIPT>
```
Este script muestra el session ID actual del usuario; versiones más sigilosas lo envían directamente al atacante sin generar ninguna alerta visible.

**Diferencia XSS vs. CSRF**:
- XSS explota la **confianza del usuario en el sitio web**.
- CSRF explota la **confianza del sitio web en el navegador del usuario**.

---

### CSRF (Cross-site Request Forgery) — "Session Riding" / "One-click Attack"

- Alias: **one-click attack**, **session riding**.
- El atacante crea un formulario malicioso y lo envía al usuario autenticado.
- El usuario completa el formulario creyendo que es legítimo y lo envía al servidor real.
- El servidor acepta los datos porque vienen de un usuario autenticado.

**Flujo**:
1. Atacante aloja una web con formulario que parece legítimo pero ya contiene la petición del atacante.
2. El usuario introduce login/password.
3. El formulario se envía al sitio real.
4. El servidor acepta la petición como si fuera del usuario.

---

### Session Replay Attack

- El atacante captura el **authentication token** escuchando la conversación usuario-servidor.
- Reenvía el authentication token capturado al servidor para suplantar al usuario.

**Flujo**:
1. Usuario establece conexión con el servidor.
2. Servidor pide autenticación.
3. Usuario envía authentication tokens → atacante los captura por eavesdropping.
4. Atacante reenvía (replays) el authentication token al servidor → acceso no autorizado.

---

### 🔴 Session Fixation Attack

**Definición clave**: el atacante **fija el session ID antes de que el usuario haga login**, en lugar de robar el ID generado aleatoriamente tras el login. La víctima "se loguea en la sesión del atacante".

**Condición de posibilidad**: la app web permite usar un session ID existente en lugar de generar uno nuevo en el momento del login.

**Vectores para fijar el session ID**:
- Session token en el **argumento URL**.
- Session token en un **hidden form field**.
- Session ID en una **cookie**.

**3 fases de la Session Fixation**:
1. **Session set-up phase**: el atacante establece una conexión legítima con el servidor y obtiene un session ID válido (trap session ID). Si el servidor tiene idle session timeout, el atacante debe mantener el session ID activo enviando peticiones periódicas.
2. **Fixation phase**: el atacante introduce el session ID trampa en el navegador de la víctima.
3. **Entrance phase**: el atacante espera a que la víctima haga login con el trap session ID y luego entra en su sesión.

**Ejemplo concreto del CEH**:
- Servidor emite session ID `0D6441FEA4496C2` al atacante.
- Atacante envía a la víctima: `http://citibank.com/?SID=0D6441FEA4496C2`.
- Víctima hace clic → el servidor detecta el session ID ya activo → no genera uno nuevo.
- Víctima introduce credenciales → el servidor le da acceso.
- Atacante accede también a `http://citibank.com/?SID=0D6441FEA4496C2` con las mismas credenciales.

---

### CRIME Attack

**Nombre completo**: Compression Ratio Info-Leak Made Easy.

- Explota la característica de **compresión de datos** en SSL/TLS, SPDY y HTTPS.
- Algoritmo de compresión explotado: **DEFLATE** (lossless).
- Baja probabilidad de mitigación → especialmente peligroso.

**Flujo**:
1. El atacante usa ingeniería social para que la víctima haga clic en un enlace malicioso.
2. Si la víctima ya tiene una conexión HTTPS activa, el atacante sniffa el tráfico HTTPS (p. ej., con ARP spoofing).
3. El atacante captura el valor de la cookie de las peticiones HTTPS y envía múltiples peticiones con la cookie precedida de caracteres aleatorios.
4. Monitoriza el tráfico para obtener el valor comprimido y cifrado de la cookie.
5. Analiza la longitud de la cookie → predice el valor real de la authentication cookie.
6. Usa la cookie para suplantar a la víctima y acceder a la aplicación segura.

**Herramienta de detección**: **CrimeCheck** — detecta si un servidor web tiene TLS o HTTP compression habilitados (vulnerabilidad CRIME).

---

### Forbidden Attack

- Tipo de **MITM attack**.
- Se ejecuta cuando un **nonce criptográfico es reutilizado** durante el establecimiento de una sesión HTTPS.
- Según la especificación TLS, los nonces deben usarse **una sola vez**.
- La vulnerabilidad: la implementación TLS reutiliza el mismo nonce al cifrar datos con **AES-GCM** durante el TLS handshake.
- El atacante usa el nonce repetido para **generar las claves criptográficas de autenticación** y así hacer MITM.

**Flujo**:
1. Atacante monitoriza la conexión y sniffa el nonce del TLS handshake.
2. Genera claves de autenticación usando el nonce → hijackea la conexión.
3. Todo el tráfico fluye a través del atacante.
4. Inyecta JavaScript o campos web en la transmisión → víctima revela contraseñas, SSN, datos bancarios.

---

### 🔴 Session Donation Attack

**Diferencia clave con Session Fixation**: en Session Donation el atacante **ya ha logueado** en el servicio con una cuenta propia; le "dona" su session ID a la víctima. La víctima introduce sus datos que quedan vinculados a la **cuenta del atacante**, no a la suya propia.

**Flujo**:
1. Atacante hace login, establece conexión legítima y borra la información almacenada de su cuenta.
2. Servidor emite session ID `0D6441FEA4496C2` al atacante.
3. Atacante envía `http://citibank.com/?SID=0D6441FEA4496C2` a la víctima.
4. Víctima hace clic → introduce su información en el formulario → la información queda vinculada a la cuenta del atacante.
5. El atacante hace login como sí mismo y accede a la información de la víctima.

**Vectores para enviar el session ID**: cross-site cooking · MITM attack · session fixation.

---

## 2. Exam Traps ⚠️

⚠️ **[Session Prediction Attack — nombre específico del brute-force]** Cuando el rango de valores predicables es muy pequeño, el brute-force se llama **session prediction attack**. El examen puede presentar ambos términos como si fueran distintos.

⚠️ **[Brute force DSL — 1.000 session IDs/segundo]** Dato numérico concreto preguntable: con DSL se pueden generar **1.000 session IDs por segundo**.

⚠️ **[MitB bypasea SSL/PKI/2FA]** A diferencia de MITM, MitB actúa dentro del navegador, después de que todos los controles de seguridad han operado. Por eso bypasea SSL, PKI y 2FA.

⚠️ **[MitB — usa DOM interface]** El mecanismo técnico específico que usa la extension maliciosa para extraer y modificar datos del formulario es el **DOM (Document Object Model) interface**.

⚠️ **[XSS vs. CSRF — quién confía en quién]** XSS explota la confianza del **usuario en el sitio web**. CSRF explota la confianza del **sitio web en el navegador del usuario**. El examen invierte esta relación deliberadamente.

⚠️ **[CSRF aliases: one-click attack y session riding]** Si el examen menciona "one-click attack" o "session riding" → CSRF.

⚠️ **[Session Fixation — ataque antes del login]** Session Fixation se inicia **antes** de que el usuario haga login (el atacante fija el session ID de antemano). Session Hijacking clásico ocurre **después** del login.

⚠️ **[Session Fixation — 3 fases: Setup → Fixation → Entrance]** El examen puede desordenar las fases.

⚠️ **[CRIME — algoritmo DEFLATE]** El algoritmo de compresión específico explotado en CRIME es **DEFLATE**. No es el cifrado el que se rompe — es la compresión.

⚠️ **[CRIME — protocolo explotado: SSL/TLS, SPDY, HTTPS]** CRIME explota la compresión en estos tres protocolos. CrimeCheck es la herramienta de detección.

⚠️ **[Forbidden Attack — nonce + AES-GCM]** Los dos datos técnicos específicos: **nonce reutilizado** + cifrado **AES-GCM** durante el TLS handshake. Si el examen menciona nonce reuse → Forbidden Attack.

⚠️ **[Session Donation vs. Session Fixation]** En Donation el atacante ya tiene una cuenta legítima y dona su session ID. En Fixation el atacante solo tiene el session ID trampa pero no una cuenta activa con información de la víctima. El resultado es distinto: Donation vincula los datos de la víctima a la cuenta del atacante.

---

## 3. Nemotécnicos

### 12 técnicas de compromiso del session token — "S-P-MITM-MitB-X-C-R-F-CRIME-For-Don-Petit"
**"Sniff Predict, MITM-Browser XSS-CSRF Replay-Fixation CRIME-Forbidden-Donation-Petit"**
- **S**niffing · **P**redictable token · **MITM** · **Mit**B · **X**SS · **C**SRF · **R**eplay · **F**ixation · **CRIME** · **For**bidden · **Don**ation · **Petit**Potam

### Session Fixation — 3 fases — "Set-Fix-Enter"
- **Set**up: obtener trap session ID
- **Fix**ation: introducir el ID en el navegador de la víctima
- **Enter**: esperar login de la víctima y entrar

### MitB vs. MITM — diferencia clave
- **MITM**: intercepta el **canal de red** (fuera del navegador).
- **MitB**: actúa **dentro del navegador** mediante Troyano; modifica DOM; bypasea SSL/2FA.

### XSS vs. CSRF — "X=Usuario confía en web; C=Web confía en navegador"
- **X**SS → el **u**suario confía en el sitio
- **C**SRF → el sitio confía en el **navegador** del usuario

### CRIME — "Compresión DEFLATE en TLS/SPDY/HTTPS"
**C**ompression **R**atio **I**nfo-Leak **M**ade **E**asy → DEFLATE → TLS, SPDY, HTTPS

### Datos numéricos clave
- Brute force con DSL: **1.000 session IDs/segundo**
- Session Fixation idle timeout: el atacante debe mantener vivo el trap session ID enviando peticiones periódicas
- Forbidden Attack: nonce usado **una sola vez** según spec TLS; la vulnerabilidad es la reutilización

---

## 4. Flashcards

**Q:** ¿Cuántos session IDs por segundo puede generar un atacante con DSL en un brute-force de session IDs?
**A:** Hasta 1.000 session IDs por segundo.

**Q:** ¿Cuándo se llama "session prediction attack" a un ataque de brute-force de session IDs?
**A:** Cuando el rango de valores predicables del session ID es muy pequeño.

**Q:** ¿Cuál es la diferencia fundamental entre MITM y Man-in-the-Browser?
**A:** MITM intercepta el tráfico de red (fuera del navegador). MitB usa un Troyano instalado dentro del navegador, entre el navegador y sus mecanismos de seguridad, y modifica las transacciones a nivel de DOM.

**Q:** ¿Por qué el MitB bypasea SSL, PKI y 2FA?
**A:** Porque actúa dentro del navegador, después de que todos los controles de seguridad han operado correctamente. Todo parece funcionar con normalidad tanto para el usuario como para el servidor.

**Q:** ¿Qué interface técnica usa la extension maliciosa en un MitB para extraer y modificar datos del formulario?
**A:** El DOM (Document Object Model) interface.

**Q:** ¿Cuál es la diferencia entre XSS y CSRF en términos de qué confianza explotan?
**A:** XSS explota la confianza del usuario en el sitio web. CSRF explota la confianza del sitio web en el navegador del usuario.

**Q:** ¿Cuáles son los dos alias de CSRF?
**A:** One-click attack y session riding.

**Q:** ¿Qué hace único al Session Fixation respecto al Session Hijacking clásico en cuanto al momento del ataque?
**A:** Session Fixation se inicia antes de que el usuario haga login (el atacante fija el session ID de antemano). El hijacking clásico ocurre después del login.

**Q:** ¿Cuáles son las 3 fases de un Session Fixation Attack?
**A:** Session set-up phase (obtener trap session ID) → Fixation phase (introducir el ID en el navegador de la víctima) → Entrance phase (esperar login y entrar en la sesión).

**Q:** ¿Qué protocolo de compresión explota el CRIME Attack y en qué protocolos opera?
**A:** Algoritmo DEFLATE (lossless). Opera en SSL/TLS, SPDY y HTTPS.

**Q:** ¿Qué herramienta detecta si un servidor es vulnerable al CRIME Attack?
**A:** CrimeCheck.

**Q:** ¿Qué vulnerabilidad técnica explota el Forbidden Attack?
**A:** La reutilización de un nonce criptográfico al cifrar con AES-GCM durante el TLS handshake. Según la especificación TLS, el nonce debe usarse solo una vez.

**Q:** ¿En qué se diferencia Session Donation de Session Fixation?
**A:** En Session Donation el atacante ya tiene una cuenta legítima activa y dona su session ID; la víctima introduce sus datos que quedan vinculados a la cuenta del atacante. En Session Fixation el atacante solo obtiene un session ID trampa para que la víctima lo use; el objetivo es acceder a la sesión autenticada de la víctima, no capturar sus datos en la propia cuenta del atacante.

**Q:** ¿En qué consiste un Session Replay Attack?
**A:** El atacante captura el authentication token escuchando la conversación usuario-servidor y luego reenvía (replays) ese mismo token al servidor para suplantar al usuario y obtener acceso no autorizado.

**Q:** ¿Qué script JavaScript usa el CEH como ejemplo para robar session IDs mediante XSS?
**A:** `<SCRIPT>alert(document.cookie);</SCRIPT>` — muestra el session ID actual del usuario (versiones más sigilosas lo envían al atacante sin generar alertas).

---

## 5. Confusión frecuente

### Session Fixation vs. Session Donation
- **Fixation**: el atacante obtiene un trap session ID (sin cuenta activa) y lo impone a la víctima para poder entrar en su sesión autenticada.
- **Donation**: el atacante tiene una cuenta legítima propia, dona su session ID a la víctima para que los datos de ésta se guarden en la cuenta del atacante.
- **Criterio**: ¿el objetivo es acceder a la sesión de la víctima? → Fixation. ¿El objetivo es que la víctima introduzca sus datos en la cuenta del atacante? → Donation.

### XSS vs. CSRF
- **XSS**: el atacante inyecta código en el sitio web que se ejecuta en el navegador del usuario; explota que el usuario confía en el sitio; requiere que el sitio sea vulnerable a inyección.
- **CSRF**: el atacante usa el estado de autenticación existente del usuario en el sitio; explota que el sitio confía en el navegador del usuario; no requiere que el sitio sea vulnerable a inyección.
- **Criterio**: ¿hay inyección de código en el sitio? → XSS. ¿Se usa la sesión activa del usuario para ejecutar acciones no autorizadas? → CSRF.

### MITM vs. MitB
- **MITM**: interceptación a nivel de red/transporte; requiere posicionarse entre cliente y servidor en la red.
- **MitB**: interceptación a nivel de aplicación dentro del navegador; requiere Troyano preinstalado; bypasea SSL/2FA; modifica DOM.
- **Criterio**: ¿el ataque opera sobre el tráfico de red? → MITM. ¿Opera dentro del navegador modificando transacciones? → MitB.

### CRIME vs. Forbidden Attack
- **CRIME**: explota la **compresión** (DEFLATE) en TLS/HTTPS/SPDY para deducir el valor de la cookie; no rompe el cifrado directamente.
- **Forbidden Attack**: explota la **reutilización de nonce** en AES-GCM durante el TLS handshake para generar claves y hacer MITM.
- **Criterio**: ¿el vector es la compresión de datos? → CRIME. ¿Es la reutilización de nonce en el cifrado? → Forbidden.

### Session Replay vs. Session Sniffing
- **Sniffing**: el atacante solo captura el session ID para hacerse pasar por la víctima en tiempo real.
- **Replay**: el atacante captura el authentication token completo y lo reenvía posteriormente al servidor para repetir una autenticación ya realizada.
- **Criterio**: ¿se reutiliza un authentication token ya enviado? → Replay. ¿Se roba el session ID para usarlo en tiempo real? → Sniffing.
