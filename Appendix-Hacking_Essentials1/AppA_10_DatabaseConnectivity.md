# AppA_10_DatabaseConnectivity.md
# CEH v13 — Appendix A: Database Connectivity

---

## 1. Conceptos y definiciones

### Conexión a SQL Server 🔴

Las aplicaciones web usan **tres métodos** para conectarse a SQL Server:
- Connection String
- OLE DB file (**UDL**)
- **ODBC Data Source Name (DSN)**

Para conectarse a SQL Server se necesita conocer: Server Name, Security Information, Database Name, Data Interface/API a usar, y Connection Procedure.

**Modos de autenticación en SQL Server**:

| Modo | Descripción | Cuándo se usa |
|---|---|---|
| **Windows Authentication Mode** | Modo de seguridad **por defecto**. Usuarios y grupos Windows son de confianza. Usa mensajes cifrados para autenticar. | Cuando la BD y la aplicación están en el **mismo servidor** |
| **Mixed Mode** | Credenciales mantenidas dentro del propio SQL Server | Cuando los usuarios se conectan desde **dominios no confiables** (aplicaciones de Internet) |

---

### Controles de datos para SQL Server 🔴

| Mecanismo | Descripción |
|---|---|
| **DAO** (Data Access Object) | Usa conexión JET; el más eficiente |
| **ADO** (ActiveX Data Object) | Establece connection string property + RecordSource property. Variantes: DSN, UDL, programático |
| **ADO Programmatically** | Declara objeto ADO connection → establece connection string → abre conexión → instancia recordset |
| **RDO** (Remote Data Object) | Similar a ADO; usa DSN o DSN-less connection strings |
| **ODBCDirect** | Usa RDO para conectividad con BD |
| **ODBC** | API para acceder a bases de datos |

---

### Conexión a MS Access

Requiere:
- **OLE DB connection manager**
- **Data provider**

Pasos: crear un OLE DB connection manager → seleccionar el data provider usando Connection Managers area en SSIS Designer o SQL Server Import and Export Wizard.

---

### Conectores MySQL 🔴

MySQL proporciona drivers estándar: **JDBC, ODBC, .NET y C nativo**.

| Driver (desarrollado por MySQL) | Protocolo/Lenguaje |
|---|---|
| Connector/NET | ADO.NET |
| Connector/ODBC | ODBC |
| Connector/J | JDBC |
| Connector/C++ | C++ |
| Connector/C | C |
| mysqlclient | C API |

Drivers de la comunidad incluyen: Connector/NET (ADO.NET), DBD::mysql (Perl), ruby-mysql (Ruby), MySQL++ (C++ Wrapper).

**MySQL Pluggable Authentication** — habilita:

| Concepto | Descripción |
|---|---|
| **External authentication** | Conecta a MySQL usando PAM, Windows login IDs, LDAP o Kerberos |
| **Proxy users** | El usuario externo actúa como proxy de un segundo usuario |
| **External user** | Usuario proxy que puede **impersonar** a otro usuario |
| **Second user** | Usuario proxied cuya identidad y privilegios son asumidos por el proxy |

---

### Conectores Oracle 🔴

| Driver Oracle | Función |
|---|---|
| **Oracle ODBC Driver** | Conecta aplicaciones ODBC en Windows, Linux, Solaris y AIX a Oracle |
| **ODP.NET** (Oracle Data Provider for .NET) | Habilita acceso ADO.NET a Oracle. Dos tipos: **Managed Driver** y **Unmanaged Driver** |
| **Oracle JDBC Driver** | Conectividad Java |
| **Oracle OCI8** | Extensión PHP de Oracle para conectar a Oracle Database |

---

## 2. Exam Traps ⚠️

⚠️ **[SQL Server — modo de autenticación por defecto]**
El modo por defecto es **Windows Authentication Mode**, no Mixed Mode. El examen puede preguntar cuál es el modo predeterminado de SQL Server.

⚠️ **[Windows Auth vs Mixed Mode — cuándo usar cada uno]**
Windows Auth: BD y app en el **mismo servidor**. Mixed Mode: usuarios desde **dominios no confiables o Internet**. El examen puede invertir los escenarios de uso.

⚠️ **[ADO vs DAO — el más eficiente]**
El más eficiente para conectar a SQL Server es **DAO con conexión JET**, no ADO. El examen puede presentar ADO como el más eficiente por ser el más completo.

⚠️ **[ODBCDirect — usa RDO, no ADO]**
ODBCDirect usa **RDO** (Remote Data Object) para conectividad con BD. No confundir con ADO ni con ODBC directo.

⚠️ **[MySQL — Connector/J es JDBC, no ODBC]**
Connector/J = **JDBC** (Java). Connector/ODBC = **ODBC**. El examen puede mezclar el sufijo del nombre con el protocolo.

⚠️ **[Oracle OCI8 — es extensión PHP]**
OCI8 es la **extensión PHP de Oracle** para conectar a Oracle Database. No es un driver ODBC ni JDBC. El examen puede presentarlo como driver Java.

⚠️ **[ODP.NET — dos tipos de driver]**
ODP.NET tiene dos variantes: **Managed Driver** y **Unmanaged Driver**. El examen puede preguntar cuántos tipos existen.

⚠️ **[MySQL proxy user vs external user]**
- **External user** = proxy user que puede **impersonar** a otro.
- **Second user** = el usuario proxied cuya identidad es asumida.
El examen puede invertir las definiciones.

---

## 3. Nemotécnicos

### SQL Server — métodos de conexión (3)
**"COD"**: Connection string, OLE DB (UDL), DSN (ODBC).

### SQL Server — Auth modes
- **Windows** = mismo servidor + cifrado = **default**.
- **Mixed** = Internet / dominios no confiables.
Regla: "Windows en casa, Mixed en la calle".

### ADO Programmatically — 4 pasos
**"DOSI"**: **D**eclara objeto → **O**pens connection (establece string) → **S**ets connection string → **I**nstantia recordset.
(Orden real: Declare → Set string → Open → Instantiate recordset)

### MySQL Connectors — sufijo = protocolo
- **/NET** → ADO.NET
- **/ODBC** → ODBC
- **/J** → **J**DBC (Java)
- **/C++** → C++
- **/C** → C API (mysqlclient)

### Oracle Drivers — "OJOO"
- **O**DBC Driver
- **J**DBC Driver
- **O**DP.NET (ADO.NET)
- **O**CI8 (PHP)

---

## 4. Flashcards

**Q:** ¿Cuáles son los tres métodos que usa una aplicación web para conectarse a SQL Server?
**A:** Connection String, OLE DB file (UDL), ODBC Data Source Name (DSN).

---

**Q:** ¿Cuál es el modo de autenticación por defecto en SQL Server?
**A:** **Windows Authentication Mode**.

---

**Q:** ¿Cuándo se usa Mixed Mode en SQL Server?
**A:** Cuando los usuarios se conectan desde dominios no confiables o aplicaciones de Internet.

---

**Q:** ¿Cuándo se usa Windows Authentication Mode en SQL Server?
**A:** Cuando la base de datos y la aplicación están en el **mismo servidor**.

---

**Q:** ¿Qué mecanismo de conexión a SQL Server es el más eficiente según el libro?
**A:** **DAO** (Data Access Object) usando una conexión JET.

---

**Q:** ¿Qué usa ODBCDirect para la conectividad con bases de datos?
**A:** **RDO** (Remote Data Object).

---

**Q:** ¿Qué dos componentes se necesitan para conectar una aplicación a MS Access?
**A:** OLE DB connection manager y Data provider.

---

**Q:** ¿Qué driver MySQL se usa para conectividad JDBC (Java)?
**A:** **Connector/J**.

---

**Q:** ¿Qué driver MySQL se usa para conectividad ODBC?
**A:** **Connector/ODBC**.

---

**Q:** ¿Qué permite la autenticación externa (External Authentication) de MySQL Pluggable Authentication?
**A:** Conectar a MySQL usando PAM, Windows login IDs, LDAP o Kerberos.

---

**Q:** ¿Qué es un "external user" en el contexto de MySQL Pluggable Authentication?
**A:** Un usuario proxy que puede **impersonar** a otro usuario (el segundo usuario/proxied user).

---

**Q:** ¿Qué driver Oracle habilita acceso ADO.NET a Oracle Database?
**A:** **ODP.NET** (Oracle Data Provider for .NET). Tiene dos variantes: Managed Driver y Unmanaged Driver.

---

**Q:** ¿Qué es Oracle OCI8?
**A:** Una extensión **PHP** de Oracle para conectar a Oracle Database.

---

**Q:** ¿En qué plataformas funciona el Oracle ODBC Driver?
**A:** Windows, Linux, Solaris y AIX.

---

## 5. Confusión frecuente

### Windows Authentication vs Mixed Mode
- **Windows Auth**: confianza en usuarios y grupos Windows; mensajes cifrados; BD y app en el mismo servidor; **modo por defecto**.
- **Mixed Mode**: credenciales dentro de SQL Server; para Internet y dominios no confiables.
- **Criterio de decisión**: "mismo servidor", "confianza en Windows" → Windows Auth. "Internet", "dominio no confiable" → Mixed Mode.

---

### ADO vs DAO vs RDO vs ODBC
- **DAO**: usa conexión JET; el más eficiente para SQL Server.
- **ADO** (ActiveX Data Object): connection string + RecordSource; variantes DSN, UDL, programático.
- **RDO** (Remote Data Object): similar a ADO; DSN o DSN-less.
- **ODBCDirect**: usa RDO para conectividad.
- **ODBC**: API genérica para acceder a BDs.
- **Criterio de decisión**: "ActiveX", "connection string + RecordSource" → ADO. "más eficiente" → DAO/JET. "RDO bajo el capó" → ODBCDirect.

---

### MySQL Connector/J vs Connector/ODBC vs Connector/NET
- **Connector/J** → **JDBC** (Java).
- **Connector/ODBC** → **ODBC**.
- **Connector/NET** → **ADO.NET**.
- **Criterio de decisión**: el sufijo del conector indica el protocolo. /J = Java/JDBC. /ODBC = ODBC. /NET = .NET/ADO.NET.

---

### Oracle OCI8 vs ODP.NET vs JDBC
- **OCI8**: extensión **PHP**. Para conectar PHP a Oracle.
- **ODP.NET**: driver **ADO.NET** (.NET). Managed y Unmanaged.
- **JDBC Driver**: conectividad **Java**.
- **Criterio de decisión**: "PHP" → OCI8. ".NET / ADO.NET" → ODP.NET. "Java" → JDBC.
