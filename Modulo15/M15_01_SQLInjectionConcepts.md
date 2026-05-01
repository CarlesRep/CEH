# M15_01_SQLInjectionConcepts.md
> Módulo 15 / Subapartado 1 — SQL Injection Concepts

---

## 1. Conceptos y definiciones

### SQL Injection — Definición y base técnica

SQL injection = técnica que explota vulnerabilidades de input no sanitizado para pasar comandos SQL a través de una aplicación web para ejecución por la BBDD backend. El fallo reside en la **aplicación web**, no en la BBDD ni en el servidor web.

**Comandos SQL básicos:** INSERT · SELECT · UPDATE · DELETE

Los ataques funcionan porque la aplicación **no valida correctamente el input** antes de pasarlo a una sentencia SQL.

---

### 🔴 Tipos de ataques habilitados por SQL injection

| Tipo de ataque | Descripción |
|---|---|
| **Authentication Bypass** | Login sin credenciales válidas → acceso con privilegios de administrador |
| **Authorization Bypass** | Altera información de autorización almacenada en la BBDD |
| **Information Disclosure** | Obtiene información sensible de la BBDD |
| **Compromised Data Integrity** | Desfigura páginas web, inserta contenido malicioso, altera contenido de la BBDD |
| **Compromised Availability of Data** | Elimina información de la BBDD, logs o información de auditoría |
| **Remote Code Execution** | Compromete el OS del host |

---

### 🔴 SQL Injection Query — Mecanismo core

**Input normal (legítimo):**
```sql
SELECT Count(*) FROM Users WHERE UserName='smith' AND Password='simpson';
```

**Input con SQL injection (payload: `Blah' or 1=1 --`):**
```sql
SELECT Count(*) FROM Users WHERE UserName='Blah' or 1=1 --' AND Password='Pe***&4**';
```

**Por qué funciona:**
- `OR 1=1` evalúa siempre como **TRUE** → la condición WHERE es siempre verdadera
- `--` indica el inicio de un **comentario en SQL** → todo lo que sigue (incluyendo `AND Password=...`) es ignorado
- La query simplificada efectiva: `SELECT Count(*) FROM Users WHERE UserName='blah' Or 1=1`
- Resultado: autenticación exitosa sin credenciales válidas

**Código vulnerable (código de servidor):**
```csharp
string strQry = "SELECT Count(*) FROM Users WHERE UserName='" + txtUser.Text 
              + "' AND Password='" + txtPassword.Text + "'";
```
El problema: **concatenación directa** del input del usuario en la query SQL sin validación.

---

### 🔴 UNION-based SQL Injection — Técnica avanzada

El operador **UNION** une los resultados de una query a otra (splicing). Requisito: el número y tipos de columnas deben coincidir con la query original.

**Obtener nombres de tablas de usuario** (desde `sysobjects` de SQL Server):
```sql
UNION SELECT id, name, '', 0 FROM sysobjects WHERE xtype='U' --
```
- `xtype='U'` filtra tablas de usuario (User tables)
- Las tablas de sistema en SQL Server: **sysobjects, syscolumns, sysindexes**

**Obtener usernames y passwords de la tabla Users:**
```sql
UNION SELECT 0, UserName, Password, 0 FROM Users --
```

---

### 🔴 Tabla de ejemplos de SQL injection

| Objetivo | Payload del atacante | Query ejecutada |
|---|---|---|
| **Updating Table** | `blah'; UPDATE jb-customers SET jb-email='info@certifiedhacker.com' WHERE email='jason@springfield.com; --` | SELECT + UPDATE en cadena |
| **Adding New Records** | `blah'; INSERT INTO jb-customers (...) VALUES (...);--` | SELECT + INSERT en cadena |
| **Identifying Table Name** | `blah' AND 1=(SELECT COUNT(*) FROM mytable); --` | Verifica si la tabla existe por conteo |
| **Deleting a Table** | `blah'; DROP TABLE Creditcard; --` | SELECT + DROP TABLE en cadena |
| **Returning More Data** | `OR 1=1` | `SELECT * FROM User_Data WHERE Email_ID='blah' OR 1=1` → devuelve todos los registros |

---

### HTTP POST vs. GET en contexto de SQL injection

| Método | Datos | Visibilidad | Uso en SQL injection |
|---|---|---|---|
| **GET** | En la URL (query string) | Visible en barra de dirección | Input directamente manipulable en URL |
| **POST** | En el body del mensaje | No visible en URL | Más seguro pero igualmente vulnerable; ideal para XML web services |

El POST se considera más seguro que GET pero **no elimina la vulnerabilidad** de SQL injection — la validación del input es la solución real.

---

### Server-side technologies vulnerables

ASP · ASP.NET · Cold Fusion · JSP · PHP · Python · Ruby on Rails

**Bases de datos relacionales target:** Microsoft SQL Server · Oracle · IBM DB2 · MySQL (open-source)

Los ataques no explotan vulnerabilidades del software específico — explotan aplicaciones que **no siguen prácticas de codificación segura**.

---

### Sistema de tablas de metadatos en SQL Server

SQL Server almacena metadatos en tablas del sistema:
- **sysobjects** — nombres de todos los objetos de la BBDD (tablas, vistas, stored procedures)
- **syscolumns** — columnas de cada tabla
- **sysindexes** — índices

Parámetro clave: `xtype='U'` en sysobjects filtra específicamente **tablas de usuario**.

---

## 2. Exam Traps ⚠️

⚠️ **[Dónde reside el fallo de SQL injection]**
El fallo está en la **aplicación web**, no en la BBDD ni en el servidor web. El examen puede presentar "vulnerabilidad del servidor SQL" como causa → incorrecto. Es siempre una falta de validación de input en la aplicación.

⚠️ **[OR 1=1: por qué siempre es TRUE]**
`1=1` es una expresión matemáticamente siempre verdadera. Combinado con OR, hace que toda la condición WHERE sea verdadera. El examen puede preguntar por qué esta expresión bypassa la autenticación.

⚠️ **[-- (double hyphen): función en SQL]**
El doble guion `--` marca el **inicio de un comentario** en SQL. Todo lo que sigue en la línea es ignorado por el intérprete. Es el mecanismo que permite ignorar la verificación de la contraseña.

⚠️ **[UNION: requisito de columnas]**
Para que una inyección UNION funcione, el número y tipos de datos de las columnas en la query inyectada deben **coincidir exactamente** con la query original. El examen puede presentar UNION como funcionando sin esta restricción.

⚠️ **[sysobjects xtype='U']**
`xtype='U'` filtra tablas de **usuario**. El examen puede preguntar qué valor de xtype se usa para tablas de usuario o qué tabla de sistema contiene los nombres de objetos de la BBDD → sysobjects.

⚠️ **[GET vs. POST: ambos son vulnerables a SQL injection]**
POST se considera más seguro que GET pero ambos son igualmente vulnerables a SQL injection. La seguridad adicional de POST es solo respecto a la visibilidad del input en la URL, no respecto a la posibilidad de inyección.

⚠️ **[Remote Code Execution como objetivo de SQL injection]**
El libro lista RCE explícitamente como uno de los seis objetivos de SQL injection — el atacante puede comprometer el OS del host. El examen puede presentarlo como no relacionado con SQL injection.

⚠️ **[Authentication Bypass vs. Authorization Bypass]**
Authentication Bypass: login sin credenciales válidas. Authorization Bypass: altera información de **autorización** (permisos, roles) almacenada en la BBDD. Son dos ataques distintos. El examen los puede intercambiar.

---

## 3. Nemotécnicos

**6 tipos de ataques SQL injection** → **"A-A-I-D-D-R"**:
- **A**uthentication Bypass · **A**uthorization Bypass · **I**nformation Disclosure · **D**ata Integrity Compromise · **D**ata Availability Compromise · **R**emote Code Execution

**Mecánica del payload `' or 1=1 --`**:
- `'` → cierra el string del username
- `or 1=1` → condición siempre TRUE
- `--` → comenta el resto (elimina verificación de password)
- Resultado: la query devuelve todos los registros → autenticación sin credenciales

**UNION attack requirements** → **"Match Columns + Match Types"**

**SQL Server system tables** → **"SYS-OCI"**: **SYS**objects · **SYS**columns · **SYS**indexes

**Tabla de ejemplos de injection** → **"U-I-I-D-R"**: **U**pdate · **I**nsert · **I**dentify table · **D**rop table · **R**eturn more data (OR 1=1)

**Comandos SQL básicos** → **"SUID"**: **S**ELECT · **U**PDATE · **I**NSERT · **D**ELETE

---

## 4. Flashcards

**Q:** ¿Dónde reside el fallo que permite los ataques de SQL injection?
**A:** En la aplicación web — específicamente en la falta de validación del input antes de pasarlo a una sentencia SQL. No es un fallo de la BBDD ni del servidor web.

---

**Q:** ¿Qué hace la expresión `OR 1=1` en un SQL injection?
**A:** Hace que la condición WHERE sea siempre verdadera (1=1 es siempre TRUE), permitiendo que la query devuelva todos los registros independientemente del valor del username.

---

**Q:** ¿Qué función cumple `--` (doble guion) en una inyección SQL?
**A:** Marca el inicio de un comentario en SQL — todo lo que sigue en la línea es ignorado por el intérprete, eliminando efectivamente la verificación de contraseña.

---

**Q:** ¿Cuál es el requisito para que funcione una inyección basada en UNION?
**A:** El número y los tipos de datos de las columnas en la query inyectada deben coincidir exactamente con los de la query original.

---

**Q:** ¿Cuáles son los 6 tipos de ataques habilitados por SQL injection?
**A:** Authentication Bypass, Authorization Bypass, Information Disclosure, Compromised Data Integrity, Compromised Availability of Data, Remote Code Execution.

---

**Q:** ¿Cuál es la diferencia entre Authentication Bypass y Authorization Bypass en SQL injection?
**A:** Authentication Bypass: login sin credenciales válidas → acceso admin. Authorization Bypass: altera la información de autorización (permisos, roles) almacenada en la BBDD.

---

**Q:** ¿Qué tablas del sistema de SQL Server almacenan metadatos de la BBDD?
**A:** sysobjects (nombres de objetos), syscolumns (columnas), sysindexes (índices).

---

**Q:** ¿Qué valor de xtype en sysobjects filtra las tablas de usuario?
**A:** `xtype='U'` — filtra específicamente User tables.

---

**Q:** ¿Cuál es la query SQL injection para obtener nombres de tablas de usuario en SQL Server?
**A:** `UNION SELECT id, name, '', 0 FROM sysobjects WHERE xtype='U' --`

---

**Q:** ¿Qué payload SQL injection se usa para devolver todos los registros de una tabla?
**A:** `OR 1=1` — `SELECT * FROM User_Data WHERE Email_ID='blah' OR 1=1` devuelve todos los registros.

---

**Q:** ¿Qué payload SQL injection elimina una tabla llamada Creditcard?
**A:** `blah'; DROP TABLE Creditcard; --`

---

**Q:** ¿Cómo puede un atacante verificar si existe una tabla llamada "mytable"?
**A:** `blah' AND 1=(SELECT COUNT(*) FROM mytable); --` — si la query no da error, la tabla existe.

---

**Q:** ¿Por qué el método HTTP POST no protege contra SQL injection?
**A:** POST oculta los parámetros de la URL pero el input sigue siendo enviado al servidor y procesado sin validación — la vulnerabilidad no está en la visibilidad del parámetro sino en la falta de validación del input antes de construir la query SQL.

---

**Q:** ¿Qué característica del código hace que una aplicación sea vulnerable a SQL injection?
**A:** La concatenación directa del input del usuario en la query SQL sin validación, como: `"SELECT ... WHERE UserName='" + txtUser.Text + "'"`.

---

**Q:** ¿Cuáles son las bases de datos relacionales que menciona el libro como target de SQL injection?
**A:** Microsoft SQL Server, Oracle, IBM DB2, MySQL (open-source).

---

**Q:** ¿Qué server-side technologies menciona el libro como potencialmente vulnerables a SQL injection?
**A:** ASP, ASP.NET, Cold Fusion, JSP, PHP, Python, Ruby on Rails.

---

**Q:** ¿Qué query SQL injection actualiza el email de un registro específico?
**A:** `blah'; UPDATE jb-customers SET jb-email='info@certifiedhacker.com' WHERE email='jason@springfield.com; --`

---

**Q:** ¿Cuál es el objetivo del ataque "Compromised Availability of Data" mediante SQL injection?
**A:** El atacante elimina información de la BBDD, logs o información de auditoría — ataque a la disponibilidad de los datos.

---

**Q:** ¿Qué query SQL injection obtiene usernames y passwords de una tabla Users?
**A:** `UNION SELECT 0, UserName, Password, 0 FROM Users --`

---

**Q:** ¿Cuál es el objetivo del Remote Code Execution a través de SQL injection?
**A:** Comprometer el sistema operativo del host (host OS) — el nivel más grave de compromiso posible mediante SQL injection.

---

## 5. Confusión frecuente

**SQL injection flaw: aplicación vs. BBDD vs. servidor web**
- El fallo está exclusivamente en la **aplicación web** — específicamente en la falta de validación del input.
- La BBDD funciona correctamente; simplemente ejecuta las queries que recibe (incluyendo las maliciosas).
- El servidor web también funciona correctamente; pasa las peticiones a la aplicación.
- Criterio: "¿dónde se produce la vulnerabilidad?" → en la aplicación, por concatenar input del usuario sin validar en queries SQL.

---

**Authentication Bypass vs. Authorization Bypass**
- Authentication Bypass: el atacante **entra** al sistema sin credenciales válidas (bypassa el login). OR 1=1 es el ataque típico.
- Authorization Bypass: el atacante ya está dentro pero altera sus **permisos o roles** en la BBDD para acceder a recursos restringidos.
- Criterio: "no tiene credenciales y entra" → Authentication Bypass. "Ya está dentro pero modifica sus privilegios en la BBDD" → Authorization Bypass.

---

**OR 1=1 vs. UNION SELECT**
- `OR 1=1`: bypassa la condición WHERE → devuelve todos los registros de la tabla consultada. Ideal para authentication bypass.
- `UNION SELECT`: une resultados de una query diferente a la original → permite extraer datos de otras tablas. Requiere coincidencia de columnas.
- Criterio: "devolver todos los registros de la misma tabla" → OR 1=1. "Extraer datos de otras tablas (ej. Users desde una búsqueda de Products)" → UNION SELECT.

---

**-- (doble guion) vs. payload OR 1=1**
- `--` es el mecanismo de comentario — cancela el resto de la query (ej. la verificación de password).
- `OR 1=1` es la condición de bypass — hace que el WHERE sea siempre verdadero.
- En el payload típico `Blah' or 1=1 --` se usan juntos: OR 1=1 bypassa el username, `--` cancela el password check.
- Criterio: "¿qué hace que se ignore el resto de la query?" → `--`. "¿qué hace que la condición siempre sea verdadera?" → `OR 1=1`.

---

**sysobjects vs. syscolumns vs. sysindexes**
- sysobjects: nombres de todos los objetos (tablas, vistas, stored procedures). Filtro para tablas de usuario: `xtype='U'`.
- syscolumns: columnas de cada tabla (qué campos tiene cada tabla).
- sysindexes: índices de las tablas.
- Criterio: "obtener nombres de tablas" → sysobjects con `xtype='U'`. "Obtener columnas de una tabla conocida" → syscolumns.

---

## 6. Técnica Adicional: SQLMap 🔴

**SQLMap** es la herramienta de referencia del CEH para automatizar la detección y explotación de vulnerabilidades SQL injection. Es open-source y detecta automáticamente el tipo de BBDD y la técnica de inyección aplicable.

### Comandos SQLMap esenciales para el examen

```bash
# Test básico de vulnerabilidad
sqlmap -u "http://target.com/page.php?id=1"

# Enumerar bases de datos disponibles
sqlmap -u "http://target.com/page.php?id=1" --dbs

# Enumerar tablas de una BBDD específica
sqlmap -u "http://target.com/page.php?id=1" -D dbname --tables

# Enumerar columnas de una tabla específica
sqlmap -u "http://target.com/page.php?id=1" -D dbname -T tablename --columns

# Volcar datos de una tabla
sqlmap -u "http://target.com/page.php?id=1" -D dbname -T tablename --dump

# Forzar uso de técnica específica (B=boolean, T=time-based, U=union, E=error, S=stacked, Q=inline)
sqlmap -u "http://target.com/page.php?id=1" --technique=U

# Test sobre formulario POST (desde fichero de request)
sqlmap -r request.txt -p "username"

# Evadir WAF con tamper scripts
sqlmap -u "http://target.com/page.php?id=1" --tamper=space2comment

# Obtener shell del sistema operativo
sqlmap -u "http://target.com/page.php?id=1" --os-shell

# Nivel de profundidad y riesgo (1-5)
sqlmap -u "http://target.com/page.php?id=1" --level=3 --risk=2
```

### Flags clave de SQLMap para el examen
| Flag | Función |
|------|---------|
| `--dbs` | Enumerar todas las BBDD disponibles |
| `-D <db>` | Especificar la BBDD objetivo |
| `--tables` | Enumerar tablas de la BBDD |
| `-T <table>` | Especificar la tabla objetivo |
| `--columns` | Enumerar columnas de la tabla |
| `--dump` | Volcar los datos |
| `--os-shell` | Obtener shell del OS (si el DBMS lo permite) |
| `--tamper` | Script para evadir WAF/IDS |
| `--technique` | Especificar técnica de inyección |
| `-r` | Leer petición HTTP desde fichero |

> **Nota examen:** SQLMap puede obtener `--os-shell` en SQL Server usando `xp_cmdshell` y en MySQL usando `SELECT INTO OUTFILE`. Es una de las capacidades más valoradas en el examen.

---

## 7. Preguntas de Práctica — Formato CEH

---

### Pregunta 1 — Localización del fallo SQL injection
Un auditor de seguridad identifica que una aplicación web permite SQL injection. El servidor SQL Server está completamente parchado y actualizado. ¿Dónde reside el fallo que permite el ataque?

A) En el servidor SQL Server, que no valida correctamente las queries  
B) En el servidor web Apache, que no filtra el tráfico HTTP  
C) En la aplicación web, que concatena el input del usuario directamente en las queries SQL sin validación  
D) En el firewall de red, que permite conexiones al puerto 1433  

> **Respuesta correcta: C** — El fallo de SQL injection **reside siempre en la aplicación web**, no en la BBDD ni en el servidor web. La BBDD ejecuta correctamente lo que recibe; el servidor web transmite correctamente lo que la aplicación le envía. La causa raíz es la **concatenación directa del input del usuario** en las queries SQL sin sanitización ni uso de prepared statements.

---

### Pregunta 2 — Mecanismo del payload `' OR 1=1 --`
Un atacante introduce `admin' OR 1=1 --` en el campo de username de una aplicación de login con SQL Server. ¿Cuál es la query SQL que se ejecuta realmente?

A) `SELECT * FROM Users WHERE Username='admin' OR 1=1` (con el password check activo)  
B) `SELECT * FROM Users WHERE Username='admin' OR 1=1 --' AND Password='...'`  (password ignorado)  
C) `SELECT * FROM Users WHERE Username='admin' OR Username='1=1'`  
D) La query falla con error de sintaxis  

> **Respuesta correcta: B** — La comilla simple `'` cierra el string del username. `OR 1=1` hace la condición siempre verdadera. `--` comenta todo lo que sigue, incluyendo `AND Password='...'`. El resultado es una query que siempre devuelve TRUE independientemente de la contraseña introducida, otorgando acceso.

---

### Pregunta 3 — SQLMap: obtener bases de datos
Un pentester usa SQLMap para atacar un parámetro vulnerable. Ha confirmado que el parámetro `id` es inyectable. ¿Cuál es el comando para enumerar todas las bases de datos disponibles en el servidor?

A) `sqlmap -u "http://target.com/page.php?id=1" --tables`  
B) `sqlmap -u "http://target.com/page.php?id=1" --dbs`  
C) `sqlmap -u "http://target.com/page.php?id=1" --schema`  
D) `sqlmap -u "http://target.com/page.php?id=1" --enumerate`  

> **Respuesta correcta: B** — `--dbs` enumera todas las **bases de datos** disponibles en el servidor. `--tables` lista las tablas (requiere especificar la BBDD con `-D`). `--schema` muestra el esquema completo. El orden correcto en SQLMap es: `--dbs` → `-D dbname --tables` → `-D dbname -T table --columns` → `--dump`.

---

### Pregunta 4 — Tipos de ataques SQL injection
Un pentester quiere actualizar el email de un usuario específico en la base de datos usando SQL injection, además de ejecutar la query original de la aplicación. ¿Qué tipo de SQL injection debe usar y cuál es el delimitador clave?

A) UNION injection con `UNION SELECT` para combinar resultados  
B) Tautology injection con `OR 1=1` para acceder a todos los registros  
C) Piggybacked Query (Stacked Queries) con `;` para ejecutar una segunda query independiente  
D) Time-based blind injection con `WAITFOR DELAY` para confirmar la modificación  

> **Respuesta correcta: C** — **Piggybacked Query / Stacked Queries** usa `;` para añadir una query completamente nueva e independiente después de la original. La query original se ejecuta sin modificar, y la inyectada (`UPDATE users SET email=...`) se ejecuta por separado. UNION solo combina resultados SELECT; no permite UPDATE/INSERT/DELETE.

---

### Pregunta 5 — Comandos SQL básicos (SUID)
¿Cuáles son los 4 comandos SQL básicos que los ataques de SQL injection pueden utilizar?

A) CREATE, READ, UPDATE, DELETE  
B) SELECT, INSERT, UPDATE, DELETE  
C) GET, POST, PUT, DELETE  
D) SELECT, FETCH, MODIFY, REMOVE  

> **Respuesta correcta: B** — Los 4 comandos SQL básicos relevantes para SQL injection son **SELECT** (obtener datos), **UPDATE** (modificar datos), **INSERT** (añadir datos) y **DELETE** (eliminar datos). El nemotécnico del libro es **SUID** (Select, Update, Insert, Delete). No confundir con los verbos HTTP (GET, POST, PUT, DELETE).

---

### Pregunta 6 — Objetivo: Remote Code Execution
Un atacante ha realizado una SQL injection exitosa en una aplicación con Microsoft SQL Server. Usando SQLMap, ejecuta `sqlmap --os-shell`. ¿Qué capacidad de SQL Server permite esto?

A) La función `EXEC xp_cmdshell` permite ejecutar comandos del sistema operativo desde SQL Server  
B) La función `SELECT INTO OUTFILE` escribe ficheros en el disco del servidor  
C) El módulo `xp_dirtree` permite listar directorios del sistema  
D) La función `UTL_HTTP` permite hacer peticiones HTTP al exterior  

> **Respuesta correcta: A** — En **SQL Server**, el procedimiento almacenado extendido `xp_cmdshell` permite ejecutar comandos del sistema operativo directamente desde T-SQL. SQLMap activa esta función (deshabilitada por defecto desde SQL Server 2005) para obtener RCE. En MySQL, se usa `SELECT INTO OUTFILE` para escribir webshells. En Oracle, `UTL_HTTP` hace peticiones HTTP.

---

### Pregunta 7 — sysobjects xtype
Un atacante usa SQL injection para obtener los nombres de todas las tablas de usuario de una base de datos SQL Server. ¿Cuál es la query inyectada correcta?

A) `UNION SELECT name FROM sysobjects WHERE xtype='T' --`  
B) `UNION SELECT id, name, '', 0 FROM sysobjects WHERE xtype='U' --`  
C) `UNION SELECT name FROM information_schema.tables --`  
D) `UNION SELECT table_name FROM syscolumns WHERE type='U' --`  

> **Respuesta correcta: B** — En SQL Server, `sysobjects` almacena todos los objetos de la BBDD. `xtype='U'` filtra específicamente las **tablas de usuario** (User tables). El número de columnas debe coincidir con la query original (4 columnas en este caso). `information_schema.tables` también funciona pero no es la sintaxis que usa el libro CEH.

---

### Pregunta 8 — Authentication Bypass vs Authorization Bypass
Escenario A: Un atacante introduce `admin' --` como username sin contraseña y accede al sistema como administrador. Escenario B: Un atacante modifica el campo `isAdmin` de su cuenta de base de datos de 0 a 1 usando SQL injection. ¿Qué tipo de SQL injection representa cada escenario?

A) A = SQL Injection básica; B = Advanced SQL Injection  
B) A = Authentication Bypass; B = Authorization Bypass  
C) A = Tautology Attack; B = Piggybacked Query  
D) Ambos son Authentication Bypass  

> **Respuesta correcta: B** — **Authentication Bypass** (escenario A): el atacante accede al sistema **sin credenciales válidas**, bypass del mecanismo de login. **Authorization Bypass** (escenario B): el atacante ya tiene acceso pero altera su **nivel de autorización/permisos** en la BBDD para obtener privilegios elevados. Son objetivos distintos de SQL injection.

