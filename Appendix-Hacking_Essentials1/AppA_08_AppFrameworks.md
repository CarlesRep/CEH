# AppA_08_AppFrameworks.md
# CEH v13 — Appendix A: Application Development Frameworks and Vulnerabilities

---

## 1. Conceptos y definiciones

### .NET Framework 🔴

Arquitectura basada en **CLR (Common Language Runtime), FCL (Framework Class Library) y tecnología JIT**. Características: multi-lenguaje, multiplataforma.

Lenguajes soportados: VB, C++, C#, JScript (entre otros). Stack: Common Language Specification → ASP.NET / Windows Forms / Data and XML → Base Class Library → CLR → Windows / COM+ Services.

**Vulnerabilidades de .NET**:

| Vulnerabilidad | Descripción |
|---|---|
| **Remote Code Execution** | Ejecución de código remoto a través de documento o aplicación maliciosa |
| **DoS** | Envío de peticiones web manipuladas que deniegan acceso al servicio .NET a usuarios legítimos |
| **Feature Bypass** | Elude etiquetado Enhanced Security Usage presentando un certificado inválido para un uso específico |
| **.NET Assembly Tampering** | Manipulación de los DLL del framework para modificar la implementación |

---

### J2EE Framework

Entorno independiente de plataforma para diseñar y desarrollar aplicaciones web Java. Modelo de aplicación **multi-capa y distribuido**.

Componentes: Application Client, Web Client, JSP Pages/Servlets, EJB, Database.
Capas: Client Tier → Web Tier → Business Tier → EIS Tier.

**Vulnerabilidades de J2EE**:
- **XSS Bypass**: elude protecciones XSS usando caracteres Unicode "overlong" no canónicos con `%00` (null byte codificado) en lugar de los caracteres en lista negra.
- **Ejecución de programas arbitrarios**: el componente PointBase 4.6 de J2EE 1.4 RI permite ejecutar programas arbitrarios mediante sentencias SQL.
- **DoS**: mismo vector (PointBase 4.6 + SQL).
- **Divulgación de información sensible**: mismo vector (PointBase 4.6 + SQL).

---

### ColdFusion

Plataforma de desarrollo web rápido (**rapid web application development**). Construida sobre **Java**, usa el contenedor J2EE **Apache Tomcat**.

**Vulnerabilidades**:
- Directory Traversal
- Unvalidated Browser Input
- CSRF
- CFFILE, CFFTP, CFPOP Vulnerability
- DoS

---

### Ruby on Rails 🔴

Framework de aplicaciones web **server-side**. Implementa el patrón **MVC (Model-View-Controller)**:

| Componente MVC | Nombre Rails | Función |
|---|---|---|
| **Model** | ActiveRecord | Mantiene la relación entre objetos y base de datos |
| **View** | ActionView | Responsable de la presentación de datos (JSP, ASP, PHP como referencia) |
| **Controller** | ActionController | Dirige el tráfico: consulta modelos para datos específicos y los organiza en la vista |

**Vulnerabilidades de Ruby on Rails**:

| Vulnerabilidad | Mecanismo |
|---|---|
| **Remote Code Execution** | Cualquier app Rails con el **parser XML habilitado** es vulnerable; facilita recuperación de BD al ejecutar código vulnerable |
| **Authentication Bypass** | La autenticación básica **no usa algoritmo de tiempo constante** para verificar credenciales → medición de diferencias de timing permite bypass |
| **DoS** | Caché y consumo de memoria excesivos por uso de rutas wildcard; cabecera HTTP Accept manipulada provoca consumo de memoria |
| **Directory Traversal** | ActionView permite leer ficheros arbitrarios usando `..` (dot dot) en un pathname cuando se usa el método `render` sin restricciones |
| **XSS** | ActionView permite inyectar scripts/HTML arbitrario en valores de atributos de tag handlers cuando el texto se declara como "HTML safe" |

---

### AJAX 🔴

Frameworks usados para crear aplicaciones web con **vínculo dinámico entre cliente y servidor**.

Tecnologías que combina AJAX:

| Tecnología | Función en AJAX |
|---|---|
| HTML/XHTML + CSS | Presentación |
| **DOM** | Display dinámico e interacción con datos |
| JSON, XML | Intercambio de datos |
| XSLT | Manipulación |
| **XMLHttpRequest** | **Comunicación asíncrona** |
| JavaScript | Integración de todas las tecnologías |

Flujo: evento en navegador → crea objeto XMLHttpRequest → envía HttpRequest → servidor procesa y responde → JavaScript procesa la respuesta → actualiza contenido de página.

**Vulnerabilidades de AJAX** 🔴:

| Vulnerabilidad | Descripción |
|---|---|
| **Increased Attack Surface** | Más llamadas ocultas = más vectores; múltiples endpoints dispersos |
| **Browser-based attacks** | El modelo de seguridad del navegador es insuficiente para el modelo Ajax; JavaScript es vulnerable a ataques basados en navegador |
| **XSS** | Construcción dinámica del DOM; ejecución dinámica de JavaScript; datos controlados por el usuario en más lugares; XSS autopropagante; streams JSON/XML pueden ser maliciosos |
| **Mashup/Widget Hacks** | Mashup = XSS auto-infectado; falta de límites de seguridad claros; widgets heredan los mismos derechos que el sitio que los ejecuta; APIs de terceros diseñadas para facilidad de uso, no seguridad; peticiones GET que recuperan JSON son vulnerables |
| **CSRF** | El workaround de acceso cross-domain permite construir un vector de ataque CSRF dinámico basado en AJAX |
| **XML/JSON Injection** | Inyecciones en streams JSON, XML, SOAP; inyección de malware JavaScript |
| **SQLi** | Inyección de ficheros maliciosos; inyecciones pueden ocurrir en JSON, XML, SOAP |
| **XPath Injection** | Vector adicional en AJAX |

---

## 2. Exam Traps ⚠️

⚠️ **[.NET Assembly Tampering — vector de ataque]**
La manipulación de los **DLL del framework** permite modificar la implementación del core. No es una vulnerabilidad de la aplicación sino del propio framework. El examen puede preguntar qué vector permite alterar el comportamiento de .NET a nivel de runtime.

⚠️ **[J2EE XSS bypass — mecanismo específico]**
El bypass usa caracteres Unicode **"overlong" no canónicos** junto con `%00` (null byte codificado). El examen puede omitir el detalle del null byte o del Unicode overlong — ambos son parte del vector.

⚠️ **[ColdFusion — base tecnológica]**
ColdFusion está construido sobre **Java** y usa **Apache Tomcat** como contenedor J2EE. No confundir con una tecnología Microsoft. El examen puede listar tecnologías base y preguntar cuál corresponde a ColdFusion.

⚠️ **[Ruby on Rails — MVC y nombres Rails]**
El examen puede dar el nombre Rails (ActiveRecord, ActionView, ActionController) y pedir a qué componente MVC corresponde, o viceversa. Memorizar las tres parejas.

⚠️ **[Rails Authentication Bypass — algoritmo de tiempo constante]**
La vulnerabilidad es que Rails **no usa algoritmo de tiempo constante** para verificar credenciales → timing attack. El examen puede describir un ataque por canal lateral de tiempo sin mencionar "constant-time" explícitamente.

⚠️ **[Rails RCE — condición: parser XML habilitado]**
La vulnerabilidad de Remote Code Execution en Rails requiere que el **parser XML esté habilitado**. No es una vulnerabilidad universal — el examen puede preguntar la condición necesaria.

⚠️ **[AJAX — XMLHttpRequest para comunicación asíncrona]**
El componente clave de AJAX para la comunicación asíncrona es el objeto **XMLHttpRequest**. DOM es para display dinámico. No confundir.

⚠️ **[AJAX — Mashup = XSS auto-infectado]**
Un mashup es una forma de **XSS auto-infectado**. Los widgets heredan los mismos derechos que el sitio que los ejecuta — ampliando el vector. El examen puede preguntar qué característica de los mashups los hace peligrosos.

⚠️ **[AJAX — inyecciones en múltiples streams]**
Las inyecciones AJAX pueden ocurrir en **JSON, XML, SOAP y otros streams**, no solo en parámetros HTTP tradicionales. El examen puede presentar solo SQL injection en formularios — en AJAX el vector es más amplio.

---

## 3. Nemotécnicos

### .NET Framework vulnerabilidades — "RDFA"
- **R**emote Code Execution
- **D**oS
- **F**eature Bypass
- **A**ssembly Tampering

### Rails MVC — parejas exactas
**"M=ActiveRecord, V=ActionView, C=ActionController"**
- **M**odel → **Active**Record → BD
- **V**iew → **Action**View → presentación
- **C**ontroller → **Action**Controller → tráfico/lógica

### AJAX tecnologías — "HC-DJ-XX-J"
- **H**TML/CSS → presentación
- **D**OM → display dinámico
- **J**SON/XML → datos
- **X**SLT → manipulación
- **X**MLHttpRequest → comunicación asíncrona
- **J**avaScript → integración

### Rails vulnerabilidades — "RADDT+X"
Remote Code Execution, Authentication Bypass, DoS, Directory Traversal, XSS

---

## 4. Flashcards

**Q:** ¿Sobre qué tres tecnologías se basa la arquitectura del .NET Framework?
**A:** CLR (Common Language Runtime), FCL (Framework Class Library) y tecnología JIT.

---

**Q:** ¿Qué vulnerabilidad .NET permite modificar la implementación del framework manipulando sus DLLs?
**A:** **.NET Assembly Tampering** (Modifying the Framework Core).

---

**Q:** ¿Qué modelo de aplicación usa J2EE?
**A:** Multi-capa y distribuido (multi-tiered, distributed application model).

---

**Q:** ¿Qué mecanismo específico usa el XSS bypass en J2EE?
**A:** Usa caracteres Unicode "overlong" no canónicos con `%00` (null byte codificado) en lugar de los caracteres en lista negra.

---

**Q:** ¿Sobre qué tecnología está construida ColdFusion y qué contenedor usa?
**A:** Construida sobre **Java**. Usa el contenedor J2EE **Apache Tomcat**.

---

**Q:** ¿Qué patrón arquitectónico implementa Ruby on Rails?
**A:** **MVC** (Model-View-Controller).

---

**Q:** ¿Cómo se llaman los componentes MVC en Rails?
**A:** Model = **ActiveRecord**, View = **ActionView**, Controller = **ActionController**.

---

**Q:** ¿Cuál es la condición necesaria para la vulnerabilidad de Remote Code Execution en Ruby on Rails?
**A:** Que el **parser XML esté habilitado** en la aplicación Rails.

---

**Q:** ¿Por qué es posible el Authentication Bypass en Ruby on Rails?
**A:** Porque la autenticación básica **no usa un algoritmo de tiempo constante** para verificar credenciales → las diferencias de timing permiten bypass.

---

**Q:** ¿Qué permite la Directory Traversal en Ruby on Rails (ActionView)?
**A:** Leer ficheros arbitrarios usando `..` (dot dot) en un pathname cuando el método `render` se usa sin restricciones.

---

**Q:** ¿Cuál es el componente AJAX responsable de la comunicación asíncrona?
**A:** El objeto **XMLHttpRequest**.

---

**Q:** ¿Qué tecnología AJAX es responsable del display dinámico e interacción con datos?
**A:** **DOM** (Document Object Model).

---

**Q:** ¿Qué es un Mashup en el contexto de vulnerabilidades AJAX?
**A:** Una forma de **XSS auto-infectado** que carece de límites de seguridad claros. Los widgets en un mashup heredan los mismos derechos que el sitio que los ejecuta.

---

**Q:** ¿En qué streams pueden ocurrir inyecciones en aplicaciones AJAX?
**A:** En **JSON, XML, SOAP** y otros streams.

---

**Q:** ¿Qué vulnerabilidad AJAX resulta del workaround de acceso cross-domain?
**A:** **CSRF** — permite construir un vector de ataque CSRF dinámico basado en AJAX.

---

## 5. Confusión frecuente

### .NET Assembly Tampering vs Feature Bypass
- **Assembly Tampering**: el atacante modifica los **DLL del framework** para cambiar su comportamiento a nivel de implementación. Afecta al framework en sí.
- **Feature Bypass**: elude mecanismos de seguridad específicos (etiquetado Enhanced Security) presentando un certificado inválido. Afecta a una función de seguridad de la aplicación.
- **Criterio de decisión**: "modificar DLL del framework" → Assembly Tampering. "eludir verificación de certificado" → Feature Bypass.

---

### Rails ActiveRecord vs ActionView vs ActionController
- **ActiveRecord** (Model): gestiona la **base de datos** y las relaciones entre objetos.
- **ActionView** (View): gestiona la **presentación** de los datos al usuario.
- **ActionController** (Controller): gestiona el **flujo**: consulta el modelo y organiza los datos en la vista.
- **Criterio de decisión**: "BD" / "objetos y datos" → ActiveRecord. "HTML / presentación" → ActionView. "lógica de negocio / tráfico" → ActionController.

---

### AJAX XSS vs CSRF vs Mashup Hack
- **XSS**: inyección de scripts arbitrarios via DOM dinámico, texto declarado "HTML safe" o streams JSON/XML maliciosos.
- **CSRF**: abusa del workaround de acceso cross-domain para generar peticiones fraudulentas desde el navegador de la víctima.
- **Mashup Hack**: XSS auto-infectado; el widget hereda derechos del sitio → escala el alcance del ataque.
- **Criterio de decisión**: "inyectar script en la página" → XSS. "petición fraudulenta cross-domain" → CSRF. "widget/tercero que hereda derechos del sitio" → Mashup.

---

### J2EE PointBase vulnerabilities — distinción entre las tres
El libro describe los tres vectores (ejecución de programas, DoS, divulgación de información) con el mismo mecanismo base: **PointBase 4.6 database component + SQL statements**. La distinción es el **impacto**: ejecución de código arbitrario vs denegación de servicio vs fuga de información. El examen puede dar el impacto y pedir el nombre de la vulnerabilidad o viceversa.
