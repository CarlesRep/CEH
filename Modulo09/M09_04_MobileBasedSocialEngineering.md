# M09_04_MobileBasedSocialEngineering.md
## CEH v13 — Módulo 09 | Mobile-based Social Engineering Techniques

---

## 1. Conceptos y definiciones

### Marco general

Las técnicas mobile-based usan aplicaciones móviles o el canal SMS/QR como vector primario. El atacante explota la confianza del usuario en los markets oficiales y en comunicaciones aparentemente legítimas por móvil.

---

### 🔴 Las 5 técnicas mobile-based

| Técnica | Mecanismo | Vector principal |
|---|---|---|
| **Publishing Malicious Apps** | App maliciosa subida a markets oficiales con nombre popular | App store oficial |
| **Repackaging Legitimate Apps** | App legítima descargada, reempaquetada con malware y subida a tienda de terceros | Third-party app store |
| **Fake Security Applications** | App falsa de seguridad distribuida tras infectar primero el PC de la víctima | App store del atacante + malware previo en PC |
| **SMiShing** | SMS fraudulento que induce a acción inmediata (llamada, descarga, web maliciosa) | SMS |
| **QRLJacking** | Clonación de QR Code de login para secuestrar sesión | QR Code + phishing web |

---

### Publishing Malicious Apps

El atacante publica una app en markets oficiales (Google Play, App Store) usando nombres populares y características atractivas (p. ej., juego). La víctima la descarga creyendo que es genuina. Una vez instalada, el malware extrae credenciales, contactos y otros datos y los envía al atacante.

**Diferencia con Repackaging**: en Publishing Malicious Apps, la app es creada desde cero por el atacante. En Repackaging, el atacante parte de una app legítima existente.

---

### Repackaging Legitimate Apps

Flujo:
1. Un desarrollador legítimo publica una app en un market oficial.
2. El atacante descarga la app legítima.
3. La reempaqueta con malware.
4. La sube a una **tienda de terceros** (third-party app store).
5. El usuario descarga la versión maliciosa creyendo que es la original.
6. El malware recopila información del usuario y la envía al atacante.

**Clave**: el vector de distribución es una **tienda de terceros**, no el market oficial.

---

### Fake Security Applications

Ataque en dos fases:
1. **Fase 1 — PC**: el atacante infecta primero el ordenador de la víctima con malware.
2. **Fase 2 — Móvil**: cuando la víctima accede a su banca online, el malware del PC muestra un pop-up indicando que debe descargar una app de seguridad en el móvil para recibir el segundo factor de autenticación.
3. La víctima descarga la app desde el store del atacante.
4. El atacante obtiene credenciales bancarias (usuario/contraseña) y el SMS del segundo factor de autenticación → acceso completo a la cuenta bancaria.

**Característica específica**: es el único ataque mobile-based que requiere infectar el PC de la víctima previamente como condición de éxito. Su objetivo concreto es **bypassear la autenticación de dos factores (2FA) por SMS**.

---

### SMiShing (SMS Phishing)

Uso del sistema de mensajería SMS para inducir acción inmediata: descargar malware, visitar una web maliciosa o llamar a un número fraudulento. Los mensajes están diseñados para provocar urgencia.

**Vectores de engaño típicos**:
- Alerta urgente del banco (llamar al número del SMS para verificar cuenta).
- Premio o selección como ganador aleatorio (pagar tasa mínima + compartir datos).

**Diferencia con Vishing**: SMiShing usa **SMS** como canal inicial → mobile-based. Vishing usa **voz/VoIP** → human-based.

---

### 🔴 QRLJacking — Flujo detallado

QRLJacking explota el método **QR Code Login** de aplicaciones web para secuestrar la sesión de la víctima.

**Flujo completo del ataque**:
1. El atacante obtiene QR codes legítimos del servicio web objetivo haciéndose pasar por usuario legítimo.
2. Clona los QR codes.
3. Construye una web de phishing que incluye el QR code clonado.
4. Implementa un **QR code refreshing script** para renovar automáticamente el QR code clonado cuando expira.
5. Envía la URL de la web de phishing a la víctima.
6. La víctima escanea el QR code con su app móvil.
7. El device ID y las credenciales de la víctima se envían al servidor del atacante.
8. El atacante se autentica en la aplicación objetivo en nombre de la víctima.

**Información adicional obtenida durante el ataque**: ubicación GPS actual, device ID, IMEI, datos de la tarjeta SIM.

**Usos posteriores**: transacciones ilegales, impersonation, envío de spam.

**Herramienta principal**: **QR TIGER** — genera duplicados de QR codes estáticos y dinámicos; permite tracking de escaneos (número, hora, ubicación, dispositivo).

**Herramientas adicionales**: QR Code Generator · Soti MobiControl · QR Code KIT.

---

### Comparativa global de las 5 técnicas mobile-based

| Técnica | ¿Requiere acción previa en PC? | Vector de distribución | Objetivo principal |
|---|---|---|---|
| Publishing Malicious Apps | No | Market oficial | Credenciales + contactos |
| Repackaging Legitimate Apps | No | Third-party store | Información del usuario |
| Fake Security Apps | **Sí** (malware previo en PC) | App store del atacante | Credenciales bancarias + bypass 2FA |
| SMiShing | No | SMS | Acción inmediata (llamada/descarga/datos) |
| QRLJacking | No | Web de phishing + QR | Secuestro de sesión de login |

---

## 2. Exam Traps ⚠️

⚠️ **[Publishing Malicious Apps vs. Repackaging]** El examen distingue el origen de la app. Si la app es creada desde cero por el atacante → Publishing Malicious Apps. Si parte de una app legítima modificada → Repackaging. El segundo siempre usa third-party stores, no el market oficial.

⚠️ **[Fake Security Apps — dos fases]** El examen puede describir solo la fase del pop-up en el PC y preguntar qué técnica es. La respuesta es Fake Security Applications (mobile-based), aunque la primera fase ocurra en un PC.

⚠️ **[Fake Security Apps — objetivo real]** No es solo robar credenciales: el objetivo específico es **bypassear 2FA por SMS** capturando el segundo factor en la app móvil falsa.

⚠️ **[SMiShing — mobile-based, no human-based]** Aunque el vector final sea una llamada telefónica (la víctima llama al número del SMS), el iniciador del ataque es un SMS → mobile-based.

⚠️ **[QRLJacking — QR code refreshing script]** Detalle técnico específico que el examen puede preguntar: el atacante necesita un script de renovación del QR code porque los QR codes de login tienen caducidad.

⚠️ **[QRLJacking — información adicional]** Además de las credenciales, el ataque obtiene GPS, device ID, IMEI y datos de SIM. El examen puede preguntar qué información extra se obtiene.

⚠️ **[QR TIGER — herramienta de QRLJacking]** Si el examen pregunta por la herramienta de clonación de QR codes asociada a QRLJacking → QR TIGER.

---

## 3. Nemotécnicos

### Las 5 técnicas mobile-based — "PuReFaSmQ"
**"Puré Frío, Salsa Muy Quemada"**
- **Pu**blishing Malicious Apps
- **Re**packaging Legitimate Apps
- **Fa**ke Security Applications
- **Sm**iShing
- **Q**RLJacking

### QRLJacking — flujo en 4 pasos — "OCRS"
**"Obtén, Clona, Refresca, Secuestra"**
- **O**btener QR code legítimo
- **C**lonar QR code
- **R**efrescar con script automático
- **S**ecuestrar sesión cuando la víctima escanea

### Fake Security Apps — "PC → Pop-up → App → 2FA"
Infecta PC → Muestra pop-up en banca online → Víctima descarga app falsa → Atacante captura 2FA por SMS

---

## 4. Flashcards

**Q:** ¿Cuál es la diferencia entre Publishing Malicious Apps y Repackaging Legitimate Apps?
**A:** Publishing crea la app maliciosa desde cero y la sube a markets oficiales. Repackaging toma una app legítima existente, le añade malware y la redistribuye en tiendas de terceros.

---

**Q:** ¿A través de qué tipo de store se distribuyen las apps reempaquetadas (Repackaging)?
**A:** Third-party app stores (tiendas de aplicaciones de terceros, no markets oficiales).

---

**Q:** ¿Cuál es el objetivo específico de las Fake Security Applications, más allá del robo de credenciales?
**A:** Bypassear la autenticación de dos factores (2FA) por SMS, capturando el código de segundo factor en la app móvil falsa instalada por la víctima.

---

**Q:** ¿Qué hace único a las Fake Security Applications respecto al resto de técnicas mobile-based?
**A:** Es la única que requiere infectar previamente el PC de la víctima como condición necesaria para ejecutar el ataque.

---

**Q:** ¿Qué es SMiShing y qué acciones busca provocar en la víctima?
**A:** SMS Phishing. Busca provocar una acción inmediata: descargar malware, visitar una web maliciosa o llamar a un número fraudulento para divulgar información personal o financiera.

---

**Q:** ¿Cuál es el mecanismo técnico clave que permite a QRLJacking mantener activo el QR code malicioso?
**A:** Un QR code refreshing script que renueva automáticamente el QR code clonado cuando caduca.

---

**Q:** ¿Qué información adicional puede obtener un atacante durante un ataque QRLJacking, además de las credenciales?
**A:** Ubicación GPS actual, device ID, IMEI y datos de la tarjeta SIM.

---

**Q:** ¿Qué herramienta de clonación de QR codes está asociada a QRLJacking en el CEH?
**A:** QR TIGER. Permite clonar QR codes estáticos y dinámicos y hacer tracking de escaneos (número, hora, ubicación, dispositivo).

---

**Q:** Un atacante escanea el QR code de login de WhatsApp Web, lo clona, construye una página de phishing y envía la URL a la víctima. Cuando la víctima escanea el código, el atacante accede a su cuenta. ¿Qué técnica es?
**A:** QRLJacking.

---

**Q:** ¿A qué categoría pertenece SMiShing y por qué no es human-based?
**A:** Mobile-based. Aunque el canal final pueda ser una llamada, el vector de iniciación del ataque es un SMS, lo que lo clasifica como mobile-based.

---

**Q:** Nombra las 5 técnicas mobile-based del CEH v13.
**A:** Publishing Malicious Apps · Repackaging Legitimate Apps · Fake Security Applications · SMiShing · QRLJacking.

---

## 5. Confusión frecuente

### Publishing Malicious Apps vs. Repackaging Legitimate Apps
- **Publishing**: app creada desde cero por el atacante; distribuida en **markets oficiales** con nombre popular.
- **Repackaging**: app legítima modificada con malware; distribuida en **tiendas de terceros**.
- **Criterio**: ¿el atacante creó la app o la tomó de otra parte? Creada → Publishing. Modificada → Repackaging. ¿Market oficial o terceros? Oficial → Publishing. Terceros → Repackaging.

---

### SMiShing vs. Vishing
- **SMiShing**: el vector inicial es un **SMS** → mobile-based.
- **Vishing**: el vector es una llamada de **voz/VoIP** → human-based.
- **Criterio**: ¿el primer contacto del atacante es un mensaje de texto? → SMiShing. ¿Es una llamada de voz? → Vishing.

---

### Fake Security Apps vs. Publishing Malicious Apps
- **Fake Security Apps**: requiere infección previa del PC; objetivo específico es el bypass de 2FA bancario; el pop-up en el navegador es el detonante.
- **Publishing Malicious Apps**: no requiere infección previa; la víctima descarga la app de forma completamente voluntaria desde el market.
- **Criterio**: ¿hay malware previo en el PC que desencadena la descarga? → Fake Security Apps. ¿La víctima descarga la app por su cuenta? → Publishing.

---

### QRLJacking vs. Phishing genérico
- **QRLJacking**: explota específicamente el mecanismo de **login por QR code**; el objetivo es el secuestro de sesión; requiere clonación activa del QR y script de refresco.
- **Phishing**: engaño genérico mediante web/email falso para robar credenciales directamente.
- **Criterio**: ¿el ataque explota el escaneo de un QR code para autenticación? → QRLJacking. ¿Es un formulario de login falso? → Phishing.
