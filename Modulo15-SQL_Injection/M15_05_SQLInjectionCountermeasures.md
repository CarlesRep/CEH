# M15_05_SQLInjectionCountermeasures.md
> Módulo 15 / Subapartado 5 — SQL Injection Countermeasures

---

## 1. Conceptos y definiciones

### 🔴 Por qué las aplicaciones son vulnerables a SQL injection — Causas raíz

| Causa | Descripción |
|---|---|
| **DB server ejecuta OS commands** | El atacante que compromete la BBDD puede ejecutar comandos del OS |
| **Privileged account para conectar a la BBDD** | Cuenta con altos privilegios → si se compromete, el atacante tiene acceso nivel OS |
| **Error messages que revelan información** | Mensajes de error genéricos revelan estructura de la BBDD, nombres de tablas, etc. |
| **No data validation en el servidor** | La causa más común — validación impropia o inexistente del input |
| **Complex software stacks** | Arquitecturas multicapa; discrepancias entre capas introducen vulnerabilidades |
| **Legacy code + backward compatibility** | Código antiguo no diseñado con prácticas de seguridad actuales |
| **Relay on concatenated queries** | Concatenar strings para crear SQL commands es especialmente peligroso |

---

### 🔴 Tres pilares de defensa contra SQL injection

1. **Minimizing Privileges** — crear cuentas con mínimos privilegios y añadir permisos solo cuando sea necesario; nunca usar cuentas DBA/admin para la aplicación.
2. **Implementing Consistent Coding Standards** — validar input tanto en cliente como en servidor; usar mensajes de error personalizados; estándares documentados.
3. **Firewalling the SQL Server** — solo hosts de confianza contactan al servidor SQL (red admin + web servers). SQL Server escucha en: **TCP 1433**, **UDP 1434**, **TCP 139 y 445** (named pipes/Microsoft networking).

---

### 🔴 Countermeasures principales — Lista completa

**Input/Output:**
- Validar size, data type y content de TODO input; rechazar datos binarios, escape sequences y comment characters.
- **Nunca** construir Transact-SQL directamente desde input del usuario.
- Usar **stored procedures** para validar input del usuario.
- Implementar múltiples capas de validación; **nunca concatenar** input no validado.
- Evitar SQL dinámico con valores de input concatenados.
- Realizar input validation basada en **whitelists, no blacklists**.
- Validar input del usuario **y** datos de fuentes no confiables **en el servidor**.
- Convertir el input de usuarios (usernames, passwords) en strings antes de validar.
- Sanitizar todos los inputs del usuario antes de usarlos en sentencias SQL dinámicas.
- Usar prepared statements con **parameterized queries** para bloquear la ejecución de queries maliciosas.
- Usar una **safe API** con interfaz parametrizada o que evite el intérprete por completo.

**Accounts/Privileges:**
- Usar **tipos de cuenta SQL más restrictivos** para las aplicaciones.
- Aplicar reglas de **least privilege** para apps que acceden al DBMS.
- **Nunca usar** las mismas cuentas de BBDD para múltiples aplicaciones.
- Eliminar **cuentas por defecto** de la BBDD SQL.
- Conectar con cuentas **no privilegiadas**; conceder mínimos privilegios a BBDD, tablas y columnas.
- **Nunca** ejecutar el DBMS como root.

**Error handling:**
- Deshabilitar verbose error messages; usar **custom error pages** que no revelan información del sistema.
- No divulgar información de errores de BBDD a los usuarios finales.
- Eliminar code tracing y debug messages antes de desplegar la aplicación.
- Deshabilitar **shell access** a la BBDD.
- Deshabilitar comandos como **xp_cmdshell**.
- Deshabilitar funcionalidades innecesarias de la BBDD.

**Architecture:**
- Usar **WAF** para eliminar inputs maliciosos.
- Usar network, host y application IDS para monitorizar injection attacks.
- Aislar el servidor web (locked in different domains).
- Deshabilitar acceso a shell en la BBDD.
- Actualizar todos los patches regularmente.
- Monitorizar regularmente sentencias SQL de aplicaciones conectadas a BBDD.
- Usar **ORM frameworks** (Hibernate, Spring Data JPA) para gestionar el acceso a datos con parametrización automática.
- Usar **vistas** para proteger datos de las tablas base restringiendo acceso.
- Externalizar el workflow de autenticación usando **OAuth APIs**.

**Passwords:**
- Usar **SHA256** (o algoritmos seguros similares) para almacenar contraseñas en lugar de texto plano.

---

### 🔴 Input Validation — Whitelist vs. Blacklist

| Aspecto | Whitelist Validation | Blacklist Validation |
|---|---|---|
| También llamada | Positive validation / Inclusion | Negative validation / Exclusion |
| Qué hace | Acepta SOLO entidades aprobadas (tipo, rango, tamaño, valor) | Rechaza TODOS los inputs maliciosos no aprobados |
| Implementación | Regular expressions con caracteres/strings permitidos | Regular expressions con caracteres/strings prohibidos |
| Caracteres ejemplo | `^\{}()@|?$` | `'\|%\|--|;\|/\*|\\\*|_|\[|@|xp_` |
| Limitación | Difícil cuando los inputs no se pueden determinar fácilmente o tienen grandes conjuntos de caracteres | Requiere anticipar TODOS los posibles ataques futuros |
| Uso conjunto | — | Se usa junto con **output encoding** para mayor efectividad |

**Mejor práctica:** usar blacklisting **junto con output encoding** para que el input se codifique y verifique antes de pasarlo a la BBDD.

---

### 🔴 Output Encoding

Técnica de validación que codifica el input para asegurarse de que está sanitizado antes de pasarlo a la BBDD. Se usa cuando whitelist validation sola no funciona (ej. SQL dinámico con valores válidos que contienen caracteres especiales como `O'Henry`).

**Problema:**
```java
// Nombre válido O'Henry falla con whitelisting por la comilla
String myQuery = "INSERT INTO UserDetails VALUES ('" + first_name + "','" + last_name + "');";
// Vulnerable: el atacante puede inyectar '',''); DROP TABLE UserDetails--
```

**Solución en Java:**
```java
myQuery = myQuery.replace("'", "\'");
```

En MySQL, la comilla simple puede reemplazarse con:
- Dos comillas simples: `''`
- Backslash + comilla: `\'`

**Drawback:** el input debe codificarse **cada vez** antes de pasarlo a la query.

---

### 🔴 Tipo-safe SQL Parameters — Código seguro vs. vulnerable

**Vulnerable:**
```csharp
SqlDataAdapter myCommand = new SqlDataAdapter("LoginStoredProcedure '" + Login.Text + "'", conn);
// Concatenación directa → vulnerable a SQL injection
```

**Secure (parameterized query):**
```csharp
SqlDataAdapter myCommand = new SqlDataAdapter(
  "SELECT aut_lname, aut_fname FROM Authors WHERE aut_id = @aut_id", conn);
SQLParameter parm = myCommand.SelectCommand.Parameters.Add("@aut_id", SqlDbType.VarChar, 11);
Parm.Value = Login.Text;
// @aut_id se trata como valor literal, no como código ejecutable
```

**Con StoredProcedure:**
```csharp
SqlDataAdapter myCommand = new SqlDataAdapter("AuthLogin", conn);
myCommand.SelectCommand.CommandType = CommandType.StoredProcedure;
SqlParameter parm = myCommand.SelectCommand.Parameters.Add("@aut_id", SqlDbType.VarChar, 11);
parm.Value = Login.Text;
```

El parámetro `@aut_id` tiene tipo `VarChar` y longitud `11` → se verifica tipo y longitud; se trata como **valor literal**, no como código ejecutable.

---

### 🔴 LIKE Clauses — Escape de wildcards

Al usar cláusula LIKE, escapar los wildcards `_`, `%`, `[` envolviéndolos en corchetes:

```csharp
s = s.Replace("[", "[[]");
s = s.Replace("%", "[%]");
s = s.Replace("_", "[_]");
```

---

### 🔴 QUOTENAME() y REPLACE() para variables en Dynamic Transact-SQL

| Condición | Función a usar |
|---|---|
| String de **≤ 128 caracteres** | `QUOTENAME(@variable, '''')` |
| String de **> 128 caracteres** | `REPLACE(@variable, '''', '''''')` |

```sql
-- Antes (vulnerable):
SET @temp = N'SELECT * FROM employees WHERE emp_lname =''' + @emp_lname + N'''';

-- Después (seguro):
SET @temp = N'SELECT * FROM employees WHERE emp_lname = ''' + REPLACE(@emp_lname,'''','''''') + N'''';
```

---

### 🔴 Regular Expressions para Detección de SQL Injection

**Hex values clave en SQL injection:**

| Hex | Carácter |
|---|---|
| `%27` | `'` (comilla simple) |
| `%2D` | `-` (guion, parte de `--`) |
| `%23` | `#` |
| `%3D` | `=` |
| `%3B` | `;` |
| `%6F` / `%4F` | `o` / `O` |
| `%72` / `%52` | `r` / `R` |
| `%3C` | `<` |
| `%3E` | `>` |
| `%2F` | `/` |

**Regex para detectar SQL meta-characters:**
```
/(\')|(\%27)|(\-\-)|(#)|(\%23)/ix
```
Detecta: `'`, `%27`, `--`, `#`, `%23`. Flags: `i`=case-insensitive, `x`=ignorar whitespaces.

**Regex modificada (detecta `=` + meta-chars):**
```
/((\%3D)|(=))[^\n]*((\%27)|(\')|(\-\-)|(\%3B)|(;))/ix
```
Detecta: `=` o `%3D`, seguido de cualquier no-newline char, seguido de `'`, `--`, `;`.

**Regex para ataque típico (detecta `' or`):**
```
/\w*((\%27)|(\'))((\%6F)|o|(\%4F))((\%72)|r|(\%52))/ix
```
Detecta: 0+ caracteres alfanuméricos + `'` + `o`/`O` + `r`/`R` (la palabra "or" en todas sus variantes y hex equivalentes).

**Regex para UNION keyword:**
```
/((\%27)|(\'))union/ix
```
Detecta: `'` o `%27` seguido de `union`. Desarrollar expresiones similares para: `insert`, `update`, `select`, `delete`, `drop`.

**Regex para MS SQL Server (stored/extended procedures):**
```
/exec(\s|\+)+(s|x)p\w+/ix
```
Detecta: `exec` + whitespaces (o `+`) + `sp` o `xp` + caracteres alfanuméricos/underscore → detecta uso de stored procedures (sp) y extended procedures (xp) como `xp_cmdshell`.

**Snort rule de ejemplo:**
```
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS $HTTP_PORTS
(msg:"SQL Injection – Paranoid"; flow:to_server,established;
uricontent:".pl"; pcre:"/(\')|(\%27)|(\-\-)|(#)|(\%23)/ix";
classtype:Web-application-attack; sid:9099; rev:5;)
```

---

### 🔴 Herramientas de detección de SQL injection

| Herramienta | Función destacada |
|---|---|
| **OWASP ZAP** | Penetration testing tool integrado; scanners automatizados + manual; ideal para developers nuevos en pen testing |
| **DSSS (Damn Small SQLi Scanner)** | Scanner de vulnerabilidades SQL injection (GET y POST parameters); fully functional |
| **Snort** | IDS que detecta SQL injection usando reglas/expresiones; puede bloquear patrones específicos |
| Burp Suite | Web security testing; detect SQLi y otras vulnerabilidades |
| Acunetix | Web vulnerability scanner |
| HCL AppScan | Application security testing |

---

## 2. Exam Traps ⚠️

⚠️ **[Causa más común de vulnerabilidad SQL injection]**
El libro señala explícitamente "No data validation at the server" como "the most common vulnerability leading to SQL injection attacks". El examen puede presentar otras causas como más comunes — la respuesta correcta siempre es la falta de validación de datos en el servidor.

⚠️ **[SQL Server ports: TCP 1433, UDP 1434, TCP 139/445]**
SQL Server escucha en: **TCP 1433** (principal), **UDP 1434**, y **TCP 139 y 445** (named pipes via Microsoft networking). El examen puede preguntar cuál es el puerto principal de SQL Server → TCP 1433. UDP 1434 es el SQL Server Browser service.

⚠️ **[Whitelist vs. Blacklist: nombres alternativos]**
Whitelist = Positive validation = Inclusion. Blacklist = Negative validation = Exclusion. El examen puede usar cualquiera de los tres nombres para cada uno.

⚠️ **[Best practice: blacklist JUNTO CON output encoding]**
El libro dice que el mejor método es usar blacklisting "along with the output encoding technique". No es solo blacklisting. No es solo output encoding. Es la combinación.

⚠️ **[QUOTENAME vs. REPLACE: condición de longitud]**
`QUOTENAME(@var, '''')` para strings de **≤ 128 caracteres**. `REPLACE(@var, '''', '''''')` para strings de **> 128 caracteres**. El examen puede invertir las condiciones.

⚠️ **[Parameterized query: el parámetro se trata como LITERAL, no como código]**
La clave de los type-safe SQL parameters es que el valor se trata como **valor literal** en lugar de código ejecutable. El tipo y longitud también se verifican. El examen puede describir parameterized queries como "más rápidas" o "más fáciles de escribir" — la razón de seguridad es la literalización del valor.

⚠️ **[Regex flags i y x]**
En las regex de detección: `i` = case-insensitive (detecta mayúsculas y minúsculas), `x` = ignorar whitespaces en el patrón. El examen puede preguntar qué significan estos flags.

⚠️ **[Regex para `exec sp/xp`: detecta stored Y extended procedures]**
`/exec(\s|\+)+(s|x)p\w+/ix` detecta tanto stored procedures (`sp`) como extended procedures (`xp`). El examen puede presentarla como detectando solo uno de los dos tipos.

⚠️ **[SHA256 para passwords, no MD5 ni SHA1]**
El libro recomienda específicamente **SHA256** para almacenar contraseñas (no MD5, no SHA1 — estos son deprecados en el contexto de SQLi countermeasures). El examen puede presentar MD5 como aceptable.

⚠️ **[Output encoding drawback: debe aplicarse CADA VEZ]**
El drawback del output encoding es que "el input necesita ser codificado cada vez antes de ser suministrado a la query de la BBDD; de lo contrario, la aplicación puede ser víctima de SQL injection". El examen puede presentar output encoding como solución única y permanente.

---

## 3. Nemotécnicos

**3 pilares de defensa** → **"M-C-F"**: **M**inimizing privileges · **C**onsistent coding standards · **F**irewalling the SQL server

**SQL Server ports** → **"1433(TCP) · 1434(UDP) · 139+445(TCP)"**

**Whitelist vs. Blacklist** → **"White=Positive=Inclusion · Black=Negative=Exclusion"**

**QUOTENAME vs. REPLACE** → **"Q≤128 · R>128"** (Q before R, small before large)

**5 regex de detección (orden lógico)**:
1. Meta-chars básicos: `/(\')|(\%27)|(\-\-)|(#)|(\%23)/ix`
2. = + meta-chars: `/((\%3D)|(=))[^\n]*((\%27)|(\')|(\-\-)|(\%3B)|(;))/ix`
3. `' or` típico: `/\w*((\%27)|(\'))((\%6F)|o|(\%4F))((\%72)|r|(\%52))/ix`
4. UNION: `/((\%27)|(\'))union/ix`
5. exec sp/xp: `/exec(\s|\+)+(s|x)p\w+/ix`

**Hex values core** → **"27='  2D=- 23=# 3D== 3B=; 6F/4F=o/O 72/52=r/R"**

**Countermeasures core** → **"No concat + Parameterize + Least privilege + Custom errors + WAF + SHA256 passwords"**

---

## 4. Flashcards

**Q:** ¿Cuál es la causa más común de vulnerabilidades a SQL injection según el libro?
**A:** No data validation at the server — la falta de validación del input en el servidor (o validación incorrecta).

---

**Q:** ¿En qué puertos escucha SQL Server por defecto?
**A:** TCP 1433 (principal), UDP 1434, y TCP 139 y 445 (named pipes via Microsoft networking).

---

**Q:** ¿Cómo se llama alternativamente Whitelist Validation y Blacklist Validation?
**A:** Whitelist = Positive validation / Inclusion. Blacklist = Negative validation / Exclusion.

---

**Q:** ¿Cuál es el mejor método para prevenir SQL injection combinando técnicas de validación?
**A:** Usar blacklisting junto con output encoding — el input se codifica y verifica antes de pasarlo a la BBDD.

---

**Q:** ¿Cuándo se usa QUOTENAME() vs. REPLACE() en Dynamic Transact-SQL?
**A:** QUOTENAME(@var, '''') para strings de ≤ 128 caracteres. REPLACE(@var, '''', '''''') para strings de > 128 caracteres.

---

**Q:** ¿Por qué los parameterized queries (type-safe SQL parameters) previenen SQL injection?
**A:** Porque el parámetro se trata como valor literal en lugar de código ejecutable. También se verifican el tipo y la longitud del valor.

---

**Q:** ¿Qué algoritmo de hash recomienda el libro para almacenar contraseñas contra SQL injection?
**A:** SHA256.

---

**Q:** ¿Cuál es el regex para detectar SQL meta-characters básicos?
**A:** `/(\')|(\%27)|(\-\-)|(#)|(\%23)/ix` — detecta comilla simple, su hex `%27`, doble guion, hash y su hex `%23`.

---

**Q:** ¿Qué detecta el regex `/exec(\s|\+)+(s|x)p\w+/ix` en SQL injection?
**A:** El keyword `exec` seguido de whitespaces/`+` y los prefijos `sp` (stored procedures) o `xp` (extended procedures) como `xp_cmdshell`.

---

**Q:** ¿Cuál es el valor hex de `;` (semicolon) en SQL injection detection?
**A:** `%3B`

---

**Q:** ¿Qué representa `%27` en SQL injection y cuál es su importancia en regex de detección?
**A:** `%27` es la URL encoding de la comilla simple `'`. Es crucial en regex porque los atacantes URL-encodean la comilla para bypassar filtros que buscan `'` literalmente.

---

**Q:** ¿Cuál es el drawback del output encoding como contrameasure?
**A:** El input necesita ser codificado CADA VEZ antes de ser suministrado a la query de la BBDD. Si se omite en algún punto, la aplicación sigue siendo vulnerable.

---

**Q:** ¿Cómo se escapan wildcards en una cláusula LIKE para prevenir SQL injection?
**A:** Envolver cada wildcard en corchetes: `[` → `[[]`, `%` → `[%]`, `_` → `[_]`.

---

**Q:** ¿Qué detecta el regex `/\w*((\%27)|(\'))((\%6F)|o|(\%4F))((\%72)|r|(\%52))/ix`?
**A:** Un ataque típico SQL injection que usa `' or` — detecta 0+ caracteres alfanuméricos seguidos de comilla simple (o su hex) seguidos de la letra `o`/`O` seguidos de `r`/`R` (la palabra "or" en todas sus variantes y valores hex).

---

**Q:** ¿Cuál es la función de Damn Small SQLi Scanner (DSSS)?
**A:** Scanner de vulnerabilidades SQL injection fully functional que soporta parámetros GET y POST. Detecta vulnerabilidades SQL injection en web applications.

---

**Q:** ¿Qué es Output Encoding y cuándo se necesita frente a Whitelist Validation?
**A:** Codifica el input antes de pasarlo a la BBDD. Se necesita cuando whitelist validation sola no funciona — ej. cuando un valor válido contiene caracteres especiales como `O'Henry` (comilla simple válida en nombres).

---

**Q:** ¿Qué flags se usan en las regex de detección de SQL injection y qué significan?
**A:** `i` = case-insensitive (detecta mayúsculas y minúsculas). `x` = ignorar whitespaces en el patrón.

---

**Q:** ¿Por qué no se deben usar las mismas cuentas de BBDD para múltiples aplicaciones?
**A:** Porque si una aplicación es comprometida, el atacante obtiene acceso con esas credenciales a todas las aplicaciones que compartan la misma cuenta de BBDD — violación del principio de mínimos privilegios.

---

**Q:** ¿Qué característica tiene Snort relacionada con SQL injection?
**A:** Detecta SQL injection mediante reglas que usan expresiones regulares. Puede bloquear patrones como uso de la función `sleep()` en User-Agent headers y otros patrones de SQL injection.

---

**Q:** ¿Cuáles son los 3 pilares de defensa contra SQL injection y por qué SQL Server usa TCP 1433?
**A:** Minimizing privileges, Implementing consistent coding standards, Firewalling the SQL Server. TCP 1433 es el puerto de escucha por defecto de SQL Server para conexiones TCP/IP.

---

## 5. Confusión frecuente

**Whitelist Validation vs. Output Encoding — ambas son técnicas de input validation**
- Whitelist Validation: acepta solo entidades aprobadas; rechaza todo lo demás. Es preventiva y se aplica al input antes de procesarlo.
- Output Encoding: codifica caracteres especiales para que sean tratados como datos, no como código. Se aplica al output (datos antes de ir a la BBDD o pantalla). Soluciona casos donde whitelist falla (ej. nombres con `'`).
- Criterio: "decidir qué input es aceptable" → Whitelist. "Codificar caracteres especiales en el input válido para que no rompan la query" → Output Encoding.

---

**QUOTENAME() vs. REPLACE() — condición de uso**
- QUOTENAME(@var, ''''): para strings de ≤ 128 caracteres. Envuelve la variable en quotes de forma segura.
- REPLACE(@var, '''', ''''''): para strings de > 128 caracteres. Reemplaza comillas simples con dos comillas simples.
- Criterio: "string corta" → QUOTENAME. "String larga (más de 128 chars)" → REPLACE.

---

**Parameterized Query vs. Stored Procedure — ambas previenen SQL injection pero de forma diferente**
- Parameterized Query: la query SQL se define con placeholders (`@param`) que se asignan como valores literales. Previene inyección porque los parámetros nunca se interpretan como código.
- Stored Procedure: la lógica SQL está precompilada en el servidor. También puede usar parámetros. Puede ser vulnerable si usa SQL dinámico internamente sin sanitizar.
- Criterio: "parámetros tratados como literales en query ad-hoc" → Parameterized Query. "Lógica SQL precompilada en el servidor" → Stored Procedure. Ambas se recomiendan; los stored procedures con SQL dinámico interno siguen necesitando sanitización.

---

**TCP 1433 vs. UDP 1434 en SQL Server**
- TCP 1433: puerto principal de SQL Server para conexiones de clientes.
- UDP 1434: SQL Server Browser Service — responde a peticiones de descubrimiento de instancias SQL Server.
- Criterio: "conexiones de clientes a SQL Server" → TCP 1433. "Descubrir instancias SQL Server en la red" → UDP 1434.

---

**Snort vs. OWASP ZAP — ambas detectan SQL injection pero diferente contexto**
- Snort: IDS de red que detecta SQL injection en tiempo real mediante reglas/regex sobre tráfico HTTP; puede bloquear patrones; herramienta de monitoring.
- OWASP ZAP: herramienta de penetration testing integrada; escanea aplicaciones web buscando vulnerabilidades; usa tanto scanners automáticos como herramientas manuales.
- Criterio: "monitorizar tráfico de red en tiempo real y generar alertas" → Snort. "Escanear aplicación web para encontrar vulnerabilidades" → OWASP ZAP.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un CISO pregunta a su equipo cuál es la causa más común de vulnerabilidades a SQL injection en aplicaciones web. ¿Cuál es la respuesta correcta según el CEH?

A) Uso de cuentas con altos privilegios para conectar a la BBDD  
B) Mensajes de error que revelan información de la estructura de la BBDD  
C) Falta de validación de datos en el servidor  
D) Uso de software legacy con backward compatibility  

**Respuesta correcta: C**
El CEH señala explícitamente **"No data validation at the server"** como la causa más común de vulnerabilidades a SQL injection. Aunque las otras opciones también son causas válidas, la validación incorrecta o inexistente del input en el servidor es la raíz principal.

---

**P2.** Un desarrollador usa el siguiente código C# para autenticación: `SqlDataAdapter cmd = new SqlDataAdapter("SELECT * FROM Users WHERE user='" + Login.Text + "'", conn);`. ¿Cuál es la solución correcta para prevenir SQL injection?

A) Convertir Login.Text a mayúsculas antes de pasarlo  
B) Limitar la longitud de Login.Text a 20 caracteres  
C) Usar parameterized query con `@user` de tipo VarChar  
D) Hashear Login.Text con MD5 antes de pasarlo a la query  

**Respuesta correcta: C**
La solución es usar **parameterized query**: `"SELECT * FROM Users WHERE user = @user"` con `SqlParameter parm = cmd.SelectCommand.Parameters.Add("@user", SqlDbType.VarChar, 20); parm.Value = Login.Text;`. El parámetro `@user` se trata como **valor literal**, no como código ejecutable, eliminando la vulnerabilidad de concatenación.

---

**P3.** Un equipo de seguridad debate si usar whitelist o blacklist para input validation en un formulario que acepta nombres de usuario. ¿Qué combinación recomienda el CEH para máxima efectividad?

A) Solo whitelist — es más segura y no necesita combinación  
B) Solo blacklist — cubre más casos de ataque  
C) Blacklist junto con output encoding  
D) Whitelist junto con blacklist simultáneamente  

**Respuesta correcta: C**
El CEH recomienda usar **blacklisting junto con output encoding**. El output encoding codifica el input para que sea tratado como datos antes de pasarlo a la BBDD. Esta combinación es más efectiva porque la blacklist rechaza inputs maliciosos conocidos y el output encoding sanitiza el resto codificando caracteres especiales.

---

**P4.** Un DBA está configurando los puertos de firewall para proteger SQL Server. ¿Qué puertos debe bloquear para hosts no autorizados según el CEH?

A) TCP 80 y TCP 443  
B) TCP 1433, UDP 1434, TCP 139 y TCP 445  
C) TCP 3306 y TCP 5432  
D) TCP 8080 y TCP 8443  

**Respuesta correcta: B**
SQL Server escucha en: **TCP 1433** (conexiones de clientes principal), **UDP 1434** (SQL Server Browser Service para descubrimiento de instancias), **TCP 139 y TCP 445** (named pipes via Microsoft networking). Solo hosts de confianza (red admin y web servers) deben poder contactar con el servidor SQL.

---

**P5.** Un desarrollador usa una cláusula LIKE en una query: `WHERE name LIKE '%' + @search + '%'`. Un usuario introduce `50%` como búsqueda y la query falla. ¿Cuál es la solución correcta?

A) Rechazar el input si contiene el carácter `%`  
B) Escapar el wildcard `%` envolviéndolo en corchetes: reemplazar `%` por `[%]`  
C) Convertir `%` a su valor hex `0x25` antes de la query  
D) Usar `QUOTENAME()` para envolver el input completo  

**Respuesta correcta: B**
Para escapar wildcards en cláusulas LIKE se deben envolver en corchetes: `%` → `[%]`, `_` → `[_]`, `[` → `[[]`. Esto hace que el carácter se trate como literal en lugar de como wildcard. El código: `s = s.Replace("%", "[%]");`.

---

**P6.** Un arquitecto de seguridad necesita elegir entre QUOTENAME() y REPLACE() para sanitizar una variable que puede contener hasta 500 caracteres de texto libre en Dynamic Transact-SQL. ¿Cuál debe usar?

A) `QUOTENAME(@var, '''')` — siempre es más segura  
B) `REPLACE(@var, '''', '''''')` — para strings de más de 128 caracteres  
C) Ambas combinadas en serie  
D) `CONVERT(nvarchar(500), @var)` — para strings largas  

**Respuesta correcta: B**
`QUOTENAME(@var, '''')` solo funciona correctamente para strings de **≤ 128 caracteres**. Para strings de **> 128 caracteres** (en este caso hasta 500) se debe usar **`REPLACE(@var, '''', '''''')`** que reemplaza cada comilla simple por dos comillas simples, sanitizando la variable sin límite de longitud.

---

**P7.** Un IDS recibe la query `' union select user_id, password from admin where user_name='admin'--`. El IDS tiene el siguiente regex: `/\w*((\%27)|(\'))((\%6F)|o|(\%4F))((\%72)|r|(\%52))/ix`. ¿Detecta este payload?

A) No — porque el payload usa `union select`, no `or`  
B) Sí — porque detecta la comilla simple antes de `union`  
C) No — porque la firma necesita el operador `=` después de `or`  
D) Sí — porque detecta `admin'` como un ataque  

**Respuesta correcta: A**
El regex `/\w*((\%27)|(\'))((\%6F)|o|(\%4F))((\%72)|r|(\%52))/ix` detecta específicamente el patrón `' or` — comilla simple seguida de las letras `o` y `r`. El payload `' union select...` NO contiene `or` después de la comilla → el regex **no** lo detecta. Se necesitaría el regex de UNION: `/((\%27)|(\'))union/ix`.

---

**P8.** Un pentester quiere detectar si una aplicación web es vulnerable a SQL injection usando herramientas del CEH. ¿Qué herramienta es específicamente un "small scanner" que soporta tanto parámetros GET como POST?

A) sqlmap  
B) OWASP ZAP  
C) DSSS (Damn Small SQLi Scanner)  
D) Snort  

**Respuesta correcta: C**
**DSSS (Damn Small SQLi Scanner)** es descrito en el CEH como un scanner de vulnerabilidades SQL injection "fully functional" que soporta parámetros **GET y POST**. sqlmap es el scanner más completo y potente. OWASP ZAP es un toolkit de penetration testing más amplio. Snort es un IDS.

---

**P9.** Un desarrollador tiene una aplicación Java que construye queries SQL y recibe nombres que pueden incluir comillas simples legítimas (como `O'Henry`). ¿Cómo debe manejar este caso sin rechazar el input válido?

A) Convertir la comilla a HTML entity `&#39;`  
B) Reemplazar la comilla simple por dos comillas simples (`O''Henry`) usando output encoding  
C) Hashear el nombre antes de almacenarlo en la BBDD  
D) Rechazar cualquier input que contenga comilla simple  

**Respuesta correcta: B**
**Output encoding** resuelve el caso de `O'Henry`: `myQuery = myQuery.replace("'", "\'")` o reemplazar `'` por `''` (dos comillas simples, que en SQL se interpretan como una comilla literal). Rechazar el input (opción D) es incorrecto — `O'Henry` es un nombre válido. HTML entity (opción A) no es válida en SQL.

---

**P10.** Un administrador de seguridad configura Snort para detectar SQL injection. ¿Qué tipo de herramienta es Snort en este contexto y cuál es su ventaja sobre OWASP ZAP?

A) Snort es un vulnerability scanner activo; ZAP es un IDS pasivo  
B) Snort es un IDS de red que monitoriza tráfico en tiempo real con reglas; ZAP es una herramienta de pentesting activo  
C) Snort analiza código fuente (SAST); ZAP analiza la aplicación en ejecución (DAST)  
D) Ambas son equivalentes; Snort tiene más plugins  

**Respuesta correcta: B**
**Snort** es un **IDS de red** que detecta SQL injection en tiempo real mediante reglas y expresiones regulares sobre el tráfico HTTP — es pasivo/monitor. **OWASP ZAP** es una herramienta de **penetration testing activo** que escanea aplicaciones buscando vulnerabilidades. Contextos distintos: Snort para producción (monitoreo), ZAP para pentesting (antes del despliegue).

---

**P11.** Un desarrollador aplica validación de input basada en whitelist con la expresión regular `^[a-zA-Z0-9]+$`. ¿Cuál es la limitación principal de este enfoque?

A) No detecta SQL keywords como SELECT o DROP  
B) Es difícil de implementar cuando los inputs legítimos tienen grandes conjuntos de caracteres permitidos  
C) Solo funciona con campos de username, no con campos de password  
D) Requiere actualización constante para cubrir nuevos ataques  

**Respuesta correcta: B**
La limitación principal de **whitelist validation** según el CEH es que es **difícil cuando los inputs no se pueden determinar fácilmente o tienen grandes conjuntos de caracteres permitidos** (ej. campos de texto libre, comentarios, direcciones internacionales). La opción D describe la limitación de *blacklist*, no de whitelist.

---

**P12.** Según los tres pilares de defensa contra SQL injection, un administrador configura que solo los web servers y la red de administración puedan conectar al servidor SQL. ¿A cuál de los tres pilares corresponde esta medida?

A) Minimizing Privileges  
B) Implementing Consistent Coding Standards  
C) Firewalling the SQL Server  
D) Input Validation Standards  

**Respuesta correcta: C**
Configurar el firewall para que **solo hosts de confianza** (web servers + red admin) puedan contactar al servidor SQL corresponde al tercer pilar: **Firewalling the SQL Server**. Minimizing Privileges trata sobre las cuentas de BBDD. Consistent Coding Standards trata sobre prácticas de desarrollo seguro.
