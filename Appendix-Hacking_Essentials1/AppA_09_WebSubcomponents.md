# AppA_09_WebSubcomponents.md
# CEH v13 — Appendix A: Web Subcomponents

---

## 1. Conceptos y definiciones

### Componentes primarios de una aplicación web

Toda aplicación web tiene **tres componentes primarios**:

| Componente | Función |
|---|---|
| **Web browser (cliente)** | Interfaz de usuario; gestiona la lógica de presentación; valida el input del usuario |
| **Web application server** | Recupera y procesa el fichero solicitado; genera el output al navegador |
| **Database server** | Almacena datos; proporciona lógica de negocio (stored procedures) |

---

### Thick, Thin y Smart Clients 🔴

| Característica | Thin Client | Thick Client | Smart Client |
|---|---|---|---|
| Procesamiento | En servidor central | En la **máquina cliente** | Combina ambos via web services |
| Hardware requerido | Mínimo (teclado + pantalla) | Completo (requiere SO + apps) | Dispositivo con conectividad Internet |
| Gestión | Centralizada | Descentralizada | — |
| Entorno ideal | **Público** (hoteles, aeropuertos) | **No** apto para entornos públicos | Múltiples plataformas y lenguajes |
| GUI | Básica | Rica (GUI y gráficos) | Rica (**rich GUI**) |
| Ejecución offline | No | Sí | **Sí** (sin Internet) |
| Personalizable | No | Sí | — |

**Smart Client (rich client)**: usa web services para comunicarse con aplicaciones en servidor. Puede ejecutarse **offline**. Diseñado para múltiples plataformas y lenguajes. Dispositivos: desktops, workstations, notebooks, tablet PCs, PDAs, móviles.

---

### Applet 🔴

Programa Java **embebido en una página web** que corre dentro del navegador en el **lado del cliente**. Contiene la API Java completa.

**Ventajas**: rendimiento rápido (corre en cliente), seguro, multiplataforma (Linux, Windows, Mac).
**Desventaja**: requiere **plugin** en el navegador para ejecutarse.

**Ciclo de vida del Applet** (en orden):

| Método | Cuándo se invoca |
|---|---|
| **init** | Inicializa el applet |
| **start** | Llamado automáticamente después de `init` |
| **paint** | Invocado inmediatamente después de `start()` |
| **stop** | Llamado automáticamente al salir de la página del applet |
| **destroy** | Llamado cuando el navegador se cierra normalmente |

---

### Servlet 🔴

Programa Java desplegado **en el servidor** que responde a peticiones del cliente y genera respuestas dinámicas. Robusto y escalable.

**Ventajas**: páginas web dinámicas, hereda características de Java, portable entre servidores web.
**Desventajas**: diseño difícil, reducción de rendimiento, lógica de negocio compleja difícil de implementar, requiere **Java Runtime Environment (JRE) en el servidor**.

**Ciclo de vida del Servlet** (en orden):

| Método | Función |
|---|---|
| **init()** | Inicializa la instancia del servlet |
| **service()** | Invocado en cada petición de servicio |
| **destroy()** | Elimina el servlet del servicio |

---

### ActiveX

Conjunto de tecnologías y servicios basado en el **Component Object Model (COM)**. Facilita la integración y reutilización de componentes. Usa **COM/DCOM** para permitir que los componentes ActiveX corran en cualquier lugar.

**ActiveX Controls**: controles manipulables visualmente con herramientas GUI. Java VM y Java Component son componentes ActiveX.
**ActiveX Scripting**: soporta cualquier lenguaje de scripting: VBScript, JScript, Perl, PowerScript, Tcl/Tk.

Elementos de ActiveX: Web Pages/Documents, Scripting (VBScript, JScript, Tcl/Tk), Controls/Applets (C++, Delphi, Java, VB), Components/Services, COM (empaquetado estándar), DCOM (computación distribuida). Plataformas: Windows, Macintosh, UNIX.

---

### Flash Application

Aplicaciones que proveen funcionalidad rica: animaciones, rich Internet apps, apps de escritorio, apps móviles, juegos móviles, reproductores de vídeo embebidos.

- **Lenguaje de desarrollo**: **ActionScript**.
- **Herramientas de diseño**: Adobe Animate, Adobe Flash Builder, Adobe Director, FlashDevelop, Adobe AIR, Apache Flex SDK.
- **Herramientas de visualización**: Flash Player (navegadores), AIR (escritorio/móvil), Scaleform (videojuegos).
- **Desventajas**: lenta carga, requiere Flash Player instalado, difícil de optimizar para SEO.

---

## 2. Exam Traps ⚠️

⚠️ **[Applet vs Servlet — dónde corren]**
**Applet**: corre en el **cliente** (navegador). **Servlet**: corre en el **servidor**. Esta distinción es la más explotada del chunk. El examen puede describir "programa Java que responde a peticiones del cliente" → Servlet. "programa Java embebido en página web que corre en el navegador" → Applet.

⚠️ **[Applet — ciclo de vida: orden de paint]**
El orden es: **init → start → paint → stop → destroy**. El examen puede proponer que `paint` ocurre antes de `start` o después de `stop`. `paint` se invoca **inmediatamente después de `start()`**.

⚠️ **[Servlet — requiere JRE en el servidor, no JDK]**
La desventaja del Servlet es que requiere **Java Runtime Environment (JRE)** en el servidor para ejecutar los servlets. El examen puede proponer JDK (Java Development Kit) como respuesta — el requisito de ejecución es el JRE.

⚠️ **[Thin Client — entorno público, no thick]**
Thin client es el **adecuado para entornos públicos** (hoteles, aeropuertos). Thick client **no** es apto para entornos públicos. El examen puede invertirlos.

⚠️ **[Smart Client — puede ejecutarse offline]**
A diferencia del thin client, el smart client puede ejecutarse **sin Internet** (offline). El examen puede describir esta capacidad y preguntar a qué tipo de cliente corresponde.

⚠️ **[ActiveX — basado en COM, no en Java]**
ActiveX se basa en **COM (Component Object Model)**. No confundir con tecnologías basadas en Java. Aunque Java VM y Java Component pueden ser componentes ActiveX, la base de ActiveX es COM.

⚠️ **[Flash — lenguaje de desarrollo es ActionScript]**
El lenguaje de programación para desarrollar aplicaciones Flash es **ActionScript**, no JavaScript ni Java. El examen puede proponer JavaScript como respuesta.

⚠️ **[Servlet — ciclo de vida de 3 métodos, no 5]**
El servlet tiene **3 métodos** en su ciclo de vida: `init()`, `service()`, `destroy()`. El applet tiene **5** (init, start, paint, stop, destroy). El examen puede preguntar el número o listar métodos de uno mezclados con los del otro.

---

## 3. Nemotécnicos

### Applet lifecycle — orden (5 pasos)
**"I Started Painting, Stopped, Destroyed"**
- **I**nit → **S**tart → **P**aint → **S**top → **D**estroy

### Servlet lifecycle — orden (3 pasos)
**"I Serve, then Die"**
- **I**nit() → **S**ervice() → **D**estroy()

### Applet vs Servlet — dónde corre
- **A**pplet → **C**lient (**A**C — "Apple Client")
- **S**ervlet → **S**erver (**S**S — doble S)

### Thin vs Thick vs Smart — tres diferencias clave
| | Thin | Thick | Smart |
|---|---|---|---|
| Procesamiento | Servidor | **Cliente** | Mixto |
| Entorno público | ✓ | ✗ | — |
| Offline | ✗ | ✓ | **✓** |

---

## 4. Flashcards

**Q:** ¿Cuáles son los tres componentes primarios de una aplicación web?
**A:** Web browser (cliente), Web application server, Database server.

---

**Q:** ¿Qué componente de una aplicación web proporciona la lógica de negocio mediante stored procedures?
**A:** El **Database server**.

---

**Q:** ¿En qué lado corre un Applet Java y en qué lado corre un Servlet Java?
**A:** Applet: lado del **cliente** (navegador). Servlet: lado del **servidor**.

---

**Q:** ¿Cuál es el ciclo de vida completo de un Applet en orden?
**A:** init → start → paint → stop → destroy.

---

**Q:** ¿Cuándo se invoca el método `paint` en el ciclo de vida del Applet?
**A:** Inmediatamente después del método `start()`.

---

**Q:** ¿Cuál es el ciclo de vida de un Servlet en orden?
**A:** init() → service() → destroy().

---

**Q:** ¿Cuándo se invoca `service()` en el ciclo de vida de un Servlet?
**A:** En cada petición de servicio (after every service request).

---

**Q:** ¿Qué requisito de entorno necesita un servidor para ejecutar Servlets?
**A:** **Java Runtime Environment (JRE)** instalado en el servidor.

---

**Q:** ¿Qué tipo de cliente es el adecuado para entornos públicos como hoteles y aeropuertos?
**A:** **Thin client**.

---

**Q:** ¿Qué diferencia un Smart Client de un Thin Client respecto a la ejecución offline?
**A:** El Smart Client puede ejecutarse **sin Internet** (offline). El Thin Client depende del servidor central y no puede operar offline.

---

**Q:** ¿En qué modelo de componentes (COM) se basa ActiveX?
**A:** En el **Component Object Model (COM)**. Usa COM/DCOM para ejecutar componentes en cualquier lugar.

---

**Q:** ¿Qué lenguaje de programación se usa para desarrollar aplicaciones Flash?
**A:** **ActionScript**.

---

**Q:** ¿Cuál es la desventaja principal de un Applet para el navegador cliente?
**A:** Requiere un **plugin** instalado en el navegador para ejecutarse.

---

**Q:** ¿Qué tecnología usa un Smart Client para comunicarse con aplicaciones en el servidor?
**A:** **Web services**.

---

**Q:** ¿Qué scripting soporta ActiveX Scripting?
**A:** Cualquier lenguaje de scripting: VBScript, JScript, Perl, PowerScript, Tcl/Tk.

---

## 5. Confusión frecuente

### Applet vs Servlet
- **Applet**: programa Java embebido en página web. Corre en el **navegador (cliente)**. Requiere plugin. Ciclo de vida: 5 métodos (init, start, paint, stop, destroy).
- **Servlet**: programa Java en el **servidor**. Responde a peticiones del cliente. Requiere JRE en servidor. Ciclo de vida: 3 métodos (init, service, destroy).
- **Criterio de decisión**: "cliente/navegador/browser/plugin" → Applet. "servidor/peticiones/dinámico en servidor" → Servlet.

---

### Thin Client vs Thick Client vs Smart Client
- **Thin**: procesamiento en servidor, hardware mínimo, gestión centralizada, apto para **entornos públicos**, no offline.
- **Thick**: procesamiento en cliente, hardware completo, personalizable, **no apto para entornos públicos**, GUI rica.
- **Smart (Rich)**: usa web services, ejecutable **offline**, multiplataforma, GUI rica.
- **Criterio de decisión**: "entorno público" → Thin. "procesamiento local, customizable" → Thick. "offline, multiplataforma, web services" → Smart.

---

### ActiveX vs Applet — componentes basados en navegador
- **ActiveX**: basado en **COM/DCOM**. Tecnología Microsoft. Soporta múltiples lenguajes (VBScript, JScript, Perl…). Corre en Windows, Mac, UNIX.
- **Applet**: basado en **Java**. Plataforma independiente (JVM). Solo Java. Requiere plugin Java en el navegador.
- **Criterio de decisión**: "COM", "DCOM", "VBScript" → ActiveX. "Java", "JVM", "API Java completa" → Applet.
