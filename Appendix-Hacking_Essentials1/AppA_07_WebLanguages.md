# AppA_07_WebLanguages.md
# CEH v13 — Appendix A: Web Markup and Programming Languages

---

## 1. Conceptos y definiciones

### Lenguajes de marcado

**HTML (HyperText Markup Language)**: lenguaje de marcado principal para crear páginas web y contenido visualizable en navegadores. Usa **tags y atributos** para definir estructura y layout del documento web.

**XML (Extensible Markup Language)**: lenguaje de marcado derivado de **SGML (Standard Generalized Markup Language)**. Define reglas para convertir datos en formato legible por máquinas y humanos. Diseñado para **almacenar y transportar datos** — no para presentarlos.
- Características: extensible, transporta datos sin presentarlos, estándar público.
- Usos: intercambio de información entre organizaciones, carga/descarga de BDs, combinación con hojas de estilo.

---

### Lenguajes de servidor 🔴

**Java**: lenguaje orientado a objetos desarrollado por **Sun Microsystems**, diseñado para entornos **distribuidos**. Puede construir módulos pequeños (**applets**) para páginas web.
- Características: independiente de plataforma, multihilo, red integrada, garbage collection automático, ejecuta código remoto de forma segura, manejo de excepciones, portabilidad.
- **Java Security Platform** = Core Java Security Architecture + Java Cryptography Architecture.
  - Core: Byte Code Verifier, Security Manager, Class Loader, Access Controller, Access Rights, Policy Description Tools.
  - Criptografía: Digital Signatures, RSA, DSA, AES, Triple DES, SHA, RC2/4, PKCS#5, Key Generators, MACs.
  - Extensiones: **JCE, JSSE, JAAS**.
  - Base: **JVM + Sandbox**.

**JSP (Java Server Pages)**: tecnología Java para desarrollar **páginas web dinámicas**. Corre en un componente servidor llamado **JSP container**. Similar a ASP y PHP pero usa Java.
- Tags fundamentales: `<%...%>` (scriptlets), `<%!...%>` (declarative), `%@...%` (directive), `<%= ...%>` (expression).
- Desventaja clave: difícil de depurar porque las páginas JSP se **convierten en servlets y luego se compilan**.

**ASP (Active Server Pages)**: framework de Microsoft para páginas web dinámicas.
- Ventajas: arquitectura 3 capas, compatible con **~55 lenguajes**, seguridad directa integrada.
- Desventajas: código interpretado y de tipado débil, mezcla layout (HTML) y lógica (scripting), sin gestión de estado real.

**PHP (PHP: Hypertext Preprocessor)**: lenguaje de scripting **server-side open source** para páginas web dinámicas e interactivas.
- Soporta programación procedural **y** orientada a objetos.
- Módulo de conexión a BD integrado.
- Desventaja: no adecuado para aplicaciones a gran escala por falta de modularidad; código fuente visible al ser open source.

**Perl (Practical Extraction and Report Language)**: lenguaje dinámico, de alto nivel, interpretado, multiplataforma. Diseñado para **edición de texto**; también para **creación y manipulación de imágenes**. Usado en desarrollo web, especialmente para **pasarelas de pago**. No necesita compilación → menor tiempo de ejecución. Soporta procedural y OOP.

**CGI (Common Gateway Interface)**: forma estándar en que un **servidor web** se conecta con aplicaciones externas. Recoge datos del navegador, los pasa al programa externo, y devuelve el output al navegador. Independiente del lenguaje (Perl, C, C++ son los más usados).

Flujo CGI: usuario rellena formulario → envío → servidor pasa datos a la aplicación CGI → CGI genera HTML → servidor envía la página al navegador.

---

### Lenguajes de cliente y scripting 🔴

**JavaScript**: lenguaje de scripting dinámico que funciona en **todos los navegadores principales**. Uso: mejorar diseño, validar formularios, detectar navegadores, crear cookies.
- Ventajas: menos interacción con servidor, feedback inmediato, mayor interactividad.
- Desventajas: sin multithreading/multiprocesador; **no puede usarse para aplicaciones de red**.

**Bash Scripting**: entorno de scripting incluido en distribuciones Linux. Muy útil para **automatizar acciones durante pentesting**. Extensión de fichero: `.sh`.

**PowerShell**: shell de línea de comandos **orientada a objetos** y lenguaje de scripting desarrollado por Microsoft para administración de sistemas. Construido sobre el **Common Language Runtime del .NET Framework**. Acepta y devuelve tanto texto como **objetos .NET Framework**. Incluye **cmdlets** (command-lets) que realizan funciones individuales.
- Ejecuta 4 tipos de comandos: funciones PowerShell, programas ejecutables, cmdlets, scripts PowerShell.

---

### Lenguajes compilados 🔴

**C**: lenguaje orientado a **procedimientos**. Lenguaje de **nivel medio** (combina elementos de alto nivel con funcionalidad de ensamblador). Usado en sistemas científicos, SOs y microcontroladores. Portabilidad: corre en cualquier compilador con mínimas modificaciones.

**C++**: lenguaje orientado a objetos, superconjunto de C. Soporta **polimorfismo estático y dinámico**.

| Característica C++ | Descripción |
|---|---|
| **Classes** | Tipos de datos definidos por el usuario |
| **Inheritance** | Un tipo adquiere propiedades de otro |
| **Data Abstraction** | Representa características clave sin detalles de implementación |
| **Encapsulation** | Envuelve datos en una única entidad |
| **Polymorphism** | Una interfaz, múltiples implementaciones |
| **Dynamic Binding** | Enlaza llamada a procedimiento con código en tiempo de ejecución |
| **Message Passing** | Objetos se comunican pasando mensajes |
| **Function Overloading** | Misma función con distintos tipos de argumento |
| **Operator Overloading** | Añade propiedades a operadores para nuevos tipos |

---

### .NET Framework

**Microsoft .NET**: arquitectura de programación de software de Microsoft para aplicaciones basadas en Internet. Implementaciones: **C#, VB.Net, ASP.Net, ADO.Net**.

Componentes fundamentales:
- **CLR (Common Language Runtime)**: entorno de ejecución que gestiona el código en ejecución y provee servicios.
- **Class Libraries**: colección de clases, interfaces y tipos de valor reutilizables para acceder a funcionalidad del sistema.
- **Assembly**: bloques de construcción de aplicaciones .NET; usados para despliegue, versionado y seguridad.

**C#**: lenguaje orientado a objetos y **type-safe**, combina productividad de lenguajes RAD con potencia de C++.

---

## 2. Exam Traps ⚠️

⚠️ **[XML — no presenta datos, solo los transporta]**
XML **lleva datos pero no los presenta**. La presentación se delega a hojas de estilo. El examen puede describir XML como "lenguaje para mostrar datos en el navegador" — es HTML, no XML.

⚠️ **[XML — derivado de SGML, no de HTML]**
XML deriva de **SGML (Standard Generalized Markup Language)**, no de HTML. HTML también deriva de SGML, pero son ramas distintas. El examen puede proponer "derivado de HTML" como trampa.

⚠️ **[Java — desarrollado por Sun Microsystems, no Microsoft]**
Java fue desarrollado por **Sun Microsystems**. .NET y C# son de Microsoft. El examen puede mezclar los dos ecosistemas.

⚠️ **[JSP — dificultad de debugging]**
La desventaja principal de JSP en el examen es que las páginas se **convierten en servlets y se compilan**, lo que dificulta el debugging. No es una limitación de rendimiento sino de mantenimiento.

⚠️ **[ASP — compatible con ~55 lenguajes]**
ASP es compatible con aproximadamente **55 lenguajes**. El examen puede proponer "solo VBScript" o un número diferente.

⚠️ **[PHP — limitación de escala]**
PHP no es adecuado para aplicaciones a gran escala por falta de **modularidad**. El examen puede proponer "por bajo rendimiento" como causa — el libro especifica que es por falta de modularidad.

⚠️ **[Perl — no necesita compilación]**
Perl es **interpretado**, no compilado. Esto es una ventaja (menor tiempo de ejecución). No confundir con Java (que compila a bytecode).

⚠️ **[JavaScript — no sirve para aplicaciones de red]**
JavaScript **no puede usarse para aplicaciones de red** y carece de multithreading. El examen puede presentarlo como un lenguaje válido para networking — no lo es según el libro.

⚠️ **[PowerShell — sobre .NET CLR, no sobre Win32]**
PowerShell está construido sobre el **CLR del .NET Framework**, no sobre la API Win32 tradicional. Acepta y devuelve objetos .NET, no solo texto.

⚠️ **[C — nivel medio, no alto nivel]**
C es un lenguaje de **nivel medio** (combina alto nivel + ensamblador). No es un lenguaje de alto nivel puro. El examen puede clasificarlo erróneamente.

⚠️ **[C++ es superconjunto de C]**
Todo código C válido es (casi) válido en C++. C++ añade OOP sobre C. El examen puede preguntar la relación entre ambos: C++ es el **superconjunto**.

⚠️ **[CGI — independiente del lenguaje]**
CGI es **independiente del lenguaje**. Perl, C y C++ son los más usados, pero no son los únicos ni son exclusivos. El examen puede presentar CGI como "solo funciona con Perl".

⚠️ **[Java Security Platform — dos partes]**
La plataforma de seguridad Java tiene **dos partes**: Core Java Security Architecture y Java Cryptography Architecture. La base incluye **JVM y Sandbox**. Las extensiones son JCE, JSSE y JAAS.

---

## 3. Nemotécnicos

### Lenguajes server-side vs client-side
- **Server-side**: JSP, ASP, PHP, Perl, CGI → el código corre en el servidor.
- **Client-side**: JavaScript, HTML → el código corre en el navegador.
- **Ambos**: Java (applets en cliente, servlets en servidor).

### Java Security Platform — base
**"JVM + Sandbox"** → la base de todo. Las extensiones son **JCE** (cifrado), **JSSE** (SSL/TLS), **JAAS** (autenticación).

### .NET componentes — "CCA"
- **C**LR: ejecuta el código.
- **C**lass Libraries: reutilización.
- **A**ssembly: despliegue, versioning, seguridad.

### C++ features OOP — "CIADPME"
Classes, Inheritance, Abstraction, Dynamic Binding, Polymorphism, Message Passing, Encapsulation (+ Function/Operator Overloading).

### Lenguajes y sus características definitivas para el examen
| Lenguaje | Característica clave |
|---|---|
| XML | Transporta datos, no los presenta; derivado de SGML |
| Java | Sun Microsystems; distribuido; JVM + Sandbox |
| PHP | Open source; server-side; no escala (no modular) |
| Perl | Interpretado; texto + imágenes; pasarelas de pago |
| JavaScript | Client-side; no networking; no multithreading |
| PowerShell | Microsoft; .NET CLR; objetos .NET; cmdlets |
| C | Nivel medio; procedural; SOs/microcontroladores |
| C++ | OOP; superconjunto de C; polimorfismo estático y dinámico |

---

## 4. Flashcards

**Q:** ¿De qué lenguaje de marcado deriva XML?
**A:** De **SGML** (Standard Generalized Markup Language).

---

**Q:** ¿Para qué está diseñado XML: presentar datos o transportarlos?
**A:** Para **transportar** datos. XML lleva datos pero no los presenta.

---

**Q:** ¿Quién desarrolló Java y para qué entornos fue diseñado?
**A:** **Sun Microsystems**. Diseñado para entornos **distribuidos**.

---

**Q:** ¿Cuáles son las dos partes de la Java Security Platform?
**A:** Core Java Security Architecture y Java Cryptography Architecture. Base: JVM + Sandbox.

---

**Q:** ¿Cuáles son las tres extensiones de seguridad de Java?
**A:** **JCE** (cifrado), **JSSE** (SSL/TLS), **JAAS** (autenticación y autorización).

---

**Q:** ¿En qué componente servidor corre JSP?
**A:** En un **JSP container**.

---

**Q:** ¿Cuál es la principal desventaja de depurar JSP?
**A:** Las páginas JSP se **convierten en servlets y luego se compilan**, lo que dificulta el debugging.

---

**Q:** ¿Con cuántos lenguajes es compatible ASP?
**A:** Aproximadamente **55 lenguajes**.

---

**Q:** ¿Por qué PHP no es adecuado para aplicaciones a gran escala?
**A:** Por falta de **modularidad**.

---

**Q:** ¿Qué tipo de lenguaje es Perl: compilado o interpretado?
**A:** **Interpretado** — no necesita compilación, lo que reduce el tiempo de ejecución.

---

**Q:** ¿Para qué uso especial menciona el libro a Perl además de desarrollo web?
**A:** Para **creación y manipulación de imágenes** y uso en **pasarelas de pago**.

---

**Q:** ¿Cuáles son las dos limitaciones principales de JavaScript según el libro?
**A:** Carece de capacidades **multithreading/multiprocesador** y **no puede usarse para aplicaciones de red**.

---

**Q:** ¿Sobre qué framework está construido PowerShell y qué devuelve además de texto?
**A:** Sobre el **CLR del .NET Framework**. Devuelve tanto texto como **objetos .NET Framework**.

---

**Q:** ¿Qué nivel de lenguaje es C y por qué?
**A:** Lenguaje de **nivel medio**: combina elementos de lenguajes de alto nivel con funcionalidad de lenguaje ensamblador.

---

**Q:** ¿Qué relación tienen C y C++?
**A:** C++ es el **superconjunto de C**. Añade OOP sobre C y soporta polimorfismo estático y dinámico.

---

**Q:** ¿Qué es CGI y de qué lenguaje depende?
**A:** Common Gateway Interface — forma estándar para que un servidor web se conecte con aplicaciones externas. Es **independiente del lenguaje** (Perl, C y C++ son los más usados).

---

**Q:** ¿Cuáles son los 4 tipos de comandos que ejecuta PowerShell?
**A:** Funciones PowerShell, programas ejecutables, cmdlets, scripts PowerShell.

---

**Q:** ¿Cuáles son los 3 componentes fundamentales del .NET Framework?
**A:** CLR (Common Language Runtime), Class Libraries, Assembly.

---

**Q:** ¿Qué hace el Assembly en .NET?
**A:** Es el bloque de construcción de apps .NET. Usado para **despliegue, versionado y seguridad**.

---

## 5. Confusión frecuente

### XML vs HTML
- **HTML**: define la **presentación** del contenido en el navegador (tags fijos, layout visual).
- **XML**: define reglas para **transportar datos** en formato legible por máquinas y humanos (tags extensibles, sin presentación).
- **Criterio de decisión**: "mostrar en navegador" / "estructura de página web" → HTML. "transportar datos entre sistemas" / "almacenar datos" → XML.

---

### Java vs JavaScript
- **Java**: lenguaje compilado (a bytecode), orientado a objetos, desarrollado por Sun Microsystems. Corre en JVM. Diseñado para entornos distribuidos. Applets para web.
- **JavaScript**: lenguaje de scripting interpretado, client-side, corre en el navegador. Sin relación directa con Java más allá del nombre.
- **Criterio de decisión**: "JVM", "Sun Microsystems", "distribuido", "applet" → Java. "navegador", "validar formularios", "client-side scripting" → JavaScript.

---

### JSP vs ASP vs PHP — server-side
- **JSP**: Java-based, JSP container, convierte a servlets (debugging difícil).
- **ASP**: Microsoft, ~55 lenguajes, arquitectura 3 capas, tipado débil.
- **PHP**: open source, server-side scripting, no escala (no modular).
- **Criterio de decisión**: "Java en el servidor" → JSP. "Microsoft, múltiples lenguajes" → ASP. "open source, scripts de servidor" → PHP.

---

### C vs C++
- **C**: procedural, nivel medio, control total sobre hardware, usado en SOs y microcontroladores.
- **C++**: OOP, superconjunto de C, clases, herencia, polimorfismo. Más abstracción.
- **Criterio de decisión**: "procedural", "nivel medio", "SOs/microcontroladores" → C. "OOP", "clases", "polimorfismo" → C++.

---

### Bash vs PowerShell
- **Bash**: shell de Linux, scripting para automatizar tareas de **pentesting**, ficheros `.sh`.
- **PowerShell**: shell de Microsoft sobre **.NET CLR**, devuelve objetos .NET, cmdlets, administración de sistemas Windows.
- **Criterio de decisión**: "Linux", "pentesting", ".sh" → Bash. "Windows", ".NET", "cmdlets", "objetos" → PowerShell.
