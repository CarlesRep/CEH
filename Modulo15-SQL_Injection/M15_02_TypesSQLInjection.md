# M15_02_TypesSQLInjection.md
> Módulo 15 / Subapartado 2 — Types of SQL Injection

---

## 1. Conceptos y definiciones

### 🔴 Tres categorías principales de SQL injection

| Categoría | Canal de ataque/resultados | Velocidad | Características |
|---|---|---|---|
| **In-band** | Mismo canal para ataque Y resultados | Rápida | Más común y fácil de explotar |
| **Blind/Inferential** | Mismo canal pero sin mensajes de error visibles | Lenta (bit a bit) | Resultados en forma booleana (TRUE/FALSE); no transmite datos visibles |
| **Out-of-band** | Canal **diferente** para ataque y resultados | Variable | Difícil de realizar; usa DNS, HTTP, email, file functions |

---

### 🔴 In-Band SQL Injection — Tipos

#### Error-based SQL Injection
El atacante introduce inputs incorrectos deliberadamente → la aplicación devuelve mensajes de error de la BBDD → el atacante analiza esos mensajes para construir queries maliciosas. Útil cuando no puede explotar otras técnicas directamente.

**Ejemplo Oracle 10g:**
```
http://www.example.com/product.php?id=10||UTL_INADDR.GET_HOST_NAME((SELECT user FROM DUAL))—
```
Resultado: `ORA-292257: host SCOTT unknown` → revela el nombre del usuario de la BBDD.

#### System Stored Procedure
El riesgo se dispara si la aplicación no sanitiza inputs usados para construir sentencias SQL dinámicas dentro de un stored procedure. El atacante inyecta inputs maliciosos en el stored procedure.

Ejemplo de stored procedure vulnerable:
```sql
Create procedure Login @user_name varchar(20), @password varchar(20) As
Declare @query varchar(250)
Set @query = 'Select 1 from usertable Where username = ' + @user_name + ' and password = ' + @password
exec(@query)
```
Input del atacante: `anyusername or 1=1'` / `anypassword` → login con cualquier contraseña.

#### Illegal/Logically Incorrect Query
El atacante envía queries incorrectas deliberadamente para obtener mensajes de error que revelan la estructura de la BBDD.

Ejemplo: `Username: 'Bob"` → query malformada → error: `"Incorrect Syntax near 'Bob'. Unclosed quotation mark after the character string ''"` → revela estructura interna.

#### 🔴 UNION SQL Injection
Usa `UNION SELECT` para unir una query forjada a la query original → accede a datos de otras tablas.

**Pasos para ejecutar UNION injection:**

1. **Determinar número de columnas** — usando ORDER BY incrementalmente:
   ```sql
   ORDER BY 10--
   ```
   Si ejecuta correctamente → 10+ columnas existen. Si error "Unknown column '10'" → menos de 10 columnas. Por ensayo y error se determina el número exacto.

2. **Determinar tipos de columnas** — usando UNION SELECT con null o valores conocidos:
   ```sql
   UNION SELECT 1,null,null--
   ```
   Si ejecuta → primera columna es de tipo integer.

3. **Ejecutar la inyección:**
   ```sql
   SELECT Name, Phone, Address FROM Users WHERE Id=1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable
   ```

**Detectar vulnerabilidad UNION:** añadir `'` al final de un comando `.php?id=`.

#### Tautology
Ataque basado en OR clause que hace que la condición WHERE siempre sea TRUE → bypassa autenticación.

```sql
SELECT * FROM users WHERE name = '' OR '1'='1';
```
`'1'='1'` es siempre TRUE.

#### End-of-Line Comment
El atacante usa `--` para comentar el resto de la query. Todo lo que sigue al `--` es ignorado por la BBDD.

```sql
SELECT * FROM members WHERE username = 'admin'--' AND password = 'password'
```
Resultado: login como admin sin contraseña.

#### In-line Comments
Integra múltiples inputs vulnerables en una sola query usando comentarios in-line (`/* */`). Permite: bypassear blacklisting, eliminar espacios, ofuscar queries, determinar versiones de la BBDD.

Ejemplo — dar privilegios de admin:
```
UserName = Attacker', 1, /*
Password = */'mypwd
```
Query resultante:
```sql
INSERT INTO Users (UserName, isAdmin, Password) VALUES('Attacker', 1, /*', 0, '*/'mypwd')
```
El atacante obtiene `isAdmin = 1`.

#### 🔴 Piggybacked Query (Stacked Queries)
El atacante inyecta una query maliciosa adicional a la query original usando **`;`** como delimitador. La query original se ejecuta sin modificar; el DBMS reconoce el `;` y ejecuta la query inyectada a continuación.

```sql
SELECT * FROM EMP WHERE EMP.EID = 1001 AND EMP.ENAME = 'Bob'; DROP TABLE DEPT;
```

**Nombre alternativo:** Stacked Queries Attack.

Objetivos: extraer, añadir, modificar, eliminar datos; ejecutar comandos remotos; DoS.

---

### 🔴 Blind/Inferential SQL Injection — Tipos

#### Principio general del Blind SQL injection
La aplicación no devuelve mensajes de error visibles (usa custom error pages). El atacante infiere información sobre la estructura de la BBDD a partir de respuestas TRUE/FALSE o tiempos de respuesta. No se transmiten datos visibles a través de la aplicación.

#### Boolean Exploitation (Inferential SQL Injection)
El atacante usa operaciones booleanas — TRUE/FALSE — para inferir información sobre la BBDD.

```
http://www.myshop.com/item.aspx?id=67 and 1=2
→ Query: SELECT ... WHERE ITEM_ID=67 AND 1=2 → FALSE → página en blanco
http://www.myshop.com/item.aspx?id=67 and 1=1
→ Query: SELECT ... WHERE ITEM_ID=67 AND 1=1 → TRUE → datos mostrados
```

Si `1=1` devuelve datos y `1=2` no → la página **es vulnerable** a SQL injection.

#### 🔴 Time-based SQL Injection (Time Delay)
Usa `WAITFOR DELAY` (SQL Server) o `BENCHMARK()` (MySQL) para evaluar tiempos de respuesta ante queries TRUE/FALSE.

```sql
-- SQL Server (Time delay 10 segundos si la tabla existe)
IF EXISTS(SELECT * FROM creditcard) WAITFOR DELAY '0:0:10'--
```

| Comando | DBMS | Sintaxis |
|---|---|---|
| `WAITFOR DELAY` | SQL Server | `WAITFOR DELAY '0:0:10'--` (horas:minutos:segundos) |
| `BENCHMARK()` | MySQL | `BENCHMARK(howmanytimes, do_this)` |

**Funcionamiento:** si la condición es TRUE → el servidor espera el tiempo especificado → el atacante detecta el delay y confirma la condición. Si FALSE → responde inmediatamente.

#### Heavy Query (Time-based sin funciones de delay)
Cuando el DBA deshabilita las funciones de time delay, el atacante usa queries que recuperan **cantidades masivas de datos** para generar el retraso. Se usan múltiples JOINs en tablas del sistema.

Ejemplo Oracle (query pesada):
```sql
SELECT count(*) FROM all_users A, all_users B, all_users C
```

Forma inyectada:
```sql
SELECT * FROM products WHERE id=1 AND 1 < SELECT count(*) FROM all_users A, all_users B, all_users C
```

Impacto: degradación severa del rendimiento del servidor.

---

### 🔴 Out-of-Band SQL Injection

Usa **canales de comunicación diferentes** para el ataque y la recuperación de resultados:
- Database email functionality
- File writing and loading functions
- **DNS requests**
- **HTTP requests**

Herramientas específicas por DBMS:

| DBMS | Función/Comando | Canal usado |
|---|---|---|
| **Microsoft SQL Server** | `xp_dirtree` | DNS requests al servidor del atacante |
| **Oracle Database** | `UTL_HTTP` package | HTTP requests desde SQL/PL/SQL al servidor del atacante |

Se usa cuando el atacante no puede usar in-band ni blind SQL injection (mismo canal bloqueado).

---

### Comparativa completa de tipos

| Tipo | Subtipo | Mecanismo core | Identificador único |
|---|---|---|---|
| **In-band** | Error-based | Mensajes de error de la BBDD | Análisis de errores |
| **In-band** | UNION | `UNION SELECT` para unir queries | Requiere coincidir columnas |
| **In-band** | Tautology | `OR '1'='1'` → siempre TRUE | Condition bypass |
| **In-band** | End-of-line comment | `--` comenta el resto | Password ignorado |
| **In-band** | In-line comments | `/* */` para ofuscar | Bypassa blacklisting |
| **In-band** | Piggybacked/Stacked | `;` separa queries adicionales | Stacked queries |
| **In-band** | Stored procedure | Input sin sanitizar en stored proc | exec() dinámico |
| **Blind** | Boolean-based | TRUE/FALSE comparando respuestas | `AND 1=1` vs `AND 1=2` |
| **Blind** | Time-based | WAITFOR/BENCHMARK mide tiempo | Delay como señal |
| **Blind** | Heavy query | JOINs masivos generan delay | Sin funciones de delay |
| **Out-of-band** | DNS/HTTP | Canal alternativo (xp_dirtree, UTL_HTTP) | Diferente canal de retorno |

---

## 2. Exam Traps ⚠️

⚠️ **[Blind SQL injection: no hay datos visibles — ni errores ni resultados]**
En blind injection **no** se ven ni mensajes de error ni datos de la BBDD. El atacante solo ve respuestas genéricas de la aplicación o diferencias de tiempo. El examen puede presentar "resultados parcialmente visibles" como descripción de blind injection → incorrecto.

⚠️ **[UNION injection: pasos en orden — columnas primero, tipos después]**
El orden es obligatorio: (1) determinar número de columnas con ORDER BY, (2) determinar tipos con UNION SELECT null, (3) ejecutar la inyección. El examen puede presentar el paso 2 antes del 1.

⚠️ **[ORDER BY para contar columnas: el error significa MENOS columnas]**
Si `ORDER BY 10--` genera error "Unknown column '10'" → hay **menos** de 10 columnas. Si ejecuta correctamente → hay 10 o más. El examen puede invertir la lógica.

⚠️ **[Piggybacked Query = Stacked Queries — son el mismo ataque]**
El libro usa ambos nombres para el mismo ataque. Delimitador: **`;`** (punto y coma). La query original NO se modifica — la maliciosa se añade después.

⚠️ **[Time-based SQL injection: WAITFOR DELAY es SQL Server, BENCHMARK es MySQL]**
El examen puede preguntar cuál función corresponde a cuál DBMS. `WAITFOR DELAY '0:0:10'--` → SQL Server. `BENCHMARK(n, expr)` → MySQL.

⚠️ **[Heavy Query: alternativa cuando el DBA deshabilita funciones de delay]**
Heavy query es la alternativa cuando **no se pueden usar** WAITFOR ni BENCHMARK. Usa múltiples JOINs en tablas del sistema. No es el método por defecto; es el fallback.

⚠️ **[Out-of-band: xp_dirtree (SQL Server) vs. UTL_HTTP (Oracle)]**
Dato específico de DBMS. `xp_dirtree` → DNS requests → SQL Server. `UTL_HTTP` → HTTP requests → Oracle. El examen puede intercambiarlos.

⚠️ **[In-line Comments: múltiples propósitos]**
In-line comments (`/* */`) no solo ofuscan — también permiten bypassear blacklisting, eliminar espacios y **determinar la versión de la BBDD**. El examen puede presentar In-line comments como solo ofuscación.

⚠️ **[Boolean exploitation: cómo confirmar vulnerabilidad]**
Si `id=67 AND 1=1` devuelve datos y `id=67 AND 1=2` no devuelve datos → la aplicación **es vulnerable**. El examen puede presentar el escenario y preguntar la conclusión.

⚠️ **[Tautology vs. End-of-Line Comment — ambos bypassean autenticación pero de forma diferente]**
Tautology: usa `OR '1'='1'` para hacer la condición siempre TRUE. End-of-line comment: usa `--` para ignorar el resto de la query (incluyendo el check de password). Mecanismos distintos, mismo objetivo.

---

## 3. Nemotécnicos

**3 categorías principales** → **"I-B-O"**: **I**n-band · **B**lind · **O**ut-of-band

**In-band subtypes (7+1)** → **"E-SP-IL-U-T-EL-IC-P"**:
- **E**rror-based · **S**tored **P**rocedure · **I**llegal query · **U**NION · **T**autology · **E**nd-of-**L**ine comment · **I**n-line **C**omments · **P**iggybacked (Stacked)

**Blind subtypes (3)** → **"B-T-H"**: **B**oolean · **T**ime-based · **H**eavy query

**UNION injection pasos (3)** → **"C-T-E"**: **C**ount columns (ORDER BY) · **T**ype columns (UNION SELECT null) · **E**xecute injection

**Time-based funciones** → **"W=SQL Server, B=MySQL"**:
- **W**AITFOR DELAY → SQL **S**erver
- **B**ENCHMARK() → MySQL

**Out-of-band herramientas** → **"xp_dirtree=MSSQL(DNS) · UTL_HTTP=Oracle(HTTP)"**

**Delimitadores clave en SQL injection:**
- `--` → end-of-line comment (ignora resto)
- `;` → query separator (piggybacked/stacked)
- `/* */` → in-line comment (ofuscación, bypassa blacklist)
- `'` → cierra string (detectar vulnerabilidad UNION)

---

## 4. Flashcards

**Q:** ¿Cuáles son las 3 categorías principales de SQL injection?
**A:** In-band (mismo canal, ataque + resultados), Blind/Inferential (sin mensajes de error visibles; respuestas booleanas/timing), Out-of-band (canal diferente para resultados).

---

**Q:** ¿Qué distingue a Blind SQL injection del in-band?
**A:** En blind injection no hay mensajes de error ni datos visibles. El atacante infiere información de la BBDD mediante respuestas TRUE/FALSE o tiempos de respuesta. No se transmiten datos visibles.

---

**Q:** ¿Cuáles son los pasos para ejecutar una UNION SQL injection?
**A:** (1) Determinar número de columnas con `ORDER BY n--` hasta generar error. (2) Determinar tipos de columnas con `UNION SELECT 1,null,null--`. (3) Ejecutar la inyección con los datos deseados.

---

**Q:** ¿Qué significa que `ORDER BY 10--` devuelva el error "Unknown column '10'"?
**A:** Que la tabla objetivo tiene **menos** de 10 columnas. Si la query se ejecuta correctamente, hay 10 o más columnas.

---

**Q:** ¿Qué es una Piggybacked Query y qué delimitador usa?
**A:** Una query maliciosa adicional inyectada después de la query original. Delimitador: `;` (punto y coma). También se llama Stacked Queries Attack. La query original no se modifica.

---

**Q:** ¿Qué hace una query de tautología como `OR '1'='1'`?
**A:** Hace que la condición WHERE sea siempre TRUE, permitiendo bypasear la autenticación y devolver todos los registros de la tabla.

---

**Q:** ¿Cuál es la función de time delay en SQL Server y cuál en MySQL?
**A:** SQL Server: `WAITFOR DELAY '0:0:10'--`. MySQL: `BENCHMARK(howmanytimes, do_this)`.

---

**Q:** ¿Cuándo se usa una Heavy Query en SQL injection?
**A:** Cuando el DBA ha deshabilitado las funciones de time delay (WAITFOR, BENCHMARK). El atacante genera retraso usando múltiples JOINs en tablas del sistema para recuperar cantidades masivas de datos.

---

**Q:** ¿Cuáles son las herramientas de out-of-band SQL injection en SQL Server y Oracle?
**A:** SQL Server: `xp_dirtree` → envía DNS requests. Oracle: `UTL_HTTP` → envía HTTP requests desde SQL/PL/SQL. Ambos dirigen las requests al servidor del atacante.

---

**Q:** ¿Cómo confirma un atacante que una aplicación es vulnerable a SQL injection usando Boolean-based blind injection?
**A:** Si `id=67 AND 1=1` devuelve datos y `id=67 AND 1=2` no devuelve datos → la aplicación es vulnerable (responde de forma diferente ante condiciones TRUE y FALSE).

---

**Q:** ¿Qué ventajas ofrece In-line Comments (`/* */`) en SQL injection?
**A:** Permite bypassear blacklisting, eliminar espacios, ofuscar queries, determinar versiones de la BBDD e integrar múltiples inputs vulnerables en una sola query.

---

**Q:** ¿Cuál es el payload para detectar una vulnerabilidad UNION SQL injection?
**A:** Añadir una comilla simple `'` al final de un parámetro como `.php?id=`. El tipo de mensaje de error obtenido indica si la BBDD es vulnerable.

---

**Q:** ¿Qué hace el attack End-of-Line Comment para bypasear la verificación de contraseña?
**A:** Usa `--` para comentar todo lo que sigue en la query, incluyendo la condición `AND password='xxx'`. Ejemplo: `admin'--` → `WHERE username='admin'--' AND password=...` → el password check queda comentado.

---

**Q:** ¿En qué se diferencia In-line Comments de End-of-Line Comment?
**A:** End-of-line comment (`--`) comenta TODO el resto de la query hasta el fin de línea. In-line comment (`/* */`) comenta solo la porción entre los marcadores, permitiendo integrar múltiples inputs vulnerables en una sola query.

---

**Q:** ¿Cuál es la sintaxis de time delay de SQL Server y qué significa el formato de tiempo?
**A:** `WAITFOR DELAY '0:0:10'--` — el formato es `horas:minutos:segundos`. `'0:0:10'` = 10 segundos de delay.

---

**Q:** ¿Qué tipo de SQL injection usa una stored procedure dinámica vulnerable?
**A:** System Stored Procedure injection — el riesgo aumenta cuando la aplicación no sanitiza los inputs que se usan para construir SQL dinámico dentro del stored procedure vía `exec()`.

---

**Q:** ¿Cuál es el nombre alternativo de Piggybacked Query?
**A:** Stacked Queries Attack.

---

**Q:** ¿Qué diferencia a Time-based blind SQL injection de Heavy Query injection?
**A:** Time-based usa funciones de delay explícitas (WAITFOR, BENCHMARK). Heavy Query genera el retraso con múltiples JOINs en tablas del sistema cuando las funciones de delay están deshabilitadas.

---

**Q:** ¿Qué información revela la función Oracle `UTL_INADDR.GET_HOST_NAME()` cuando se inyecta con `SELECT user FROM DUAL`?
**A:** Intenta devolver el hostname de la BBDD pasando el nombre del usuario actual como parámetro → genera un error ORA que contiene el nombre del usuario de la BBDD (ej. `ORA-292257: host SCOTT unknown`).

---

**Q:** ¿Por qué Out-of-band SQL injection es más difícil de realizar que In-band?
**A:** Porque el atacante necesita comunicarse con el servidor y determinar las características específicas del DBMS para usar el canal alternativo apropiado (DNS, HTTP), y necesita que el servidor tenga habilitadas esas funcionalidades.

---

## 5. Confusión frecuente

**In-band vs. Blind vs. Out-of-band — canal de comunicación**
- In-band: el mismo canal HTTP se usa para el ataque Y para ver los resultados (mensajes de error o datos devueltos visibles).
- Blind: el mismo canal HTTP pero SIN resultados visibles — el atacante infiere por comportamiento (TRUE/FALSE) o timing.
- Out-of-band: canales DIFERENTES — el ataque va por HTTP pero los resultados llegan por DNS, HTTP al servidor del atacante, email, etc.
- Criterio: "¿los resultados llegan por el mismo canal que el ataque?" Sí + visibles → In-band. Sí + no visibles → Blind. No → Out-of-band.

---

**Tautology vs. Boolean-based Blind**
- Tautology: técnica In-band. Usa `OR '1'='1'` para hacer la condición siempre TRUE y bypassear auth. Los resultados son visibles.
- Boolean-based Blind: técnica Blind. Usa `AND 1=1` / `AND 1=2` para inferir información a partir de diferencias en la respuesta (sin datos visibles).
- Criterio: "bypasear autenticación con OR clause visible" → Tautology. "Inferir estructura de BBDD sin mensajes visibles usando TRUE/FALSE" → Boolean blind.

---

**WAITFOR DELAY vs. BENCHMARK — DBMS diferente**
- WAITFOR DELAY: SQL Server. Sintaxis: `WAITFOR DELAY 'hh:mm:ss'`.
- BENCHMARK: MySQL. Sintaxis: `BENCHMARK(n_times, expression)`.
- Heavy Query: alternativa cuando ambas funciones están deshabilitadas. Usa JOINs masivos.
- Criterio: "delay en SQL Server" → WAITFOR. "delay en MySQL" → BENCHMARK. "delay sin funciones, DBA las ha bloqueado" → Heavy Query.

---

**Error-based vs. Illegal Query — ambas usan errores pero con objetivos distintos**
- Error-based: el atacante introduce inputs que generan errores de la BBDD para construir queries maliciosas más precisas. Es una técnica activa de explotación.
- Illegal/Logically Incorrect Query: el atacante envía queries incorrectas para obtener mensajes que revelen la estructura de la BBDD (nombres de columnas, tablas). Es una técnica de reconocimiento.
- Criterio: "construir exploit usando mensajes de error" → Error-based. "Obtener estructura de BBDD mediante errores intencionales" → Illegal query.

---

**Piggybacked vs. UNION — ambos añaden queries pero de forma diferente**
- UNION: une los resultados de dos queries en un solo conjunto de resultados. Requiere mismo número/tipos de columnas. La query original se ejecuta Y la inyectada también, mezclando los resultados.
- Piggybacked/Stacked: usa `;` para ejecutar una segunda query **completamente independiente** después de la original. No requiere coincidencia de columnas. La primera query se ejecuta y devuelve sus resultados; la segunda se ejecuta por separado.
- Criterio: "obtener datos de otra tabla en el mismo resultado" → UNION. "Ejecutar una operación completamente diferente (DROP, INSERT) además de la original" → Piggybacked/Stacked.

---

## 6. Preguntas de Práctica — Formato CEH

---

### Pregunta 1 — In-Band vs Blind vs Out-of-Band
Un pentester detecta que al introducir una comilla simple en un parámetro GET, la aplicación no muestra mensajes de error pero responde de forma diferente cuando el parámetro es `id=1 AND 1=1` (muestra datos) que cuando es `id=1 AND 1=2` (no muestra datos). ¿Qué tipo de SQL injection es y cuál es la técnica específica?

A) In-Band — Error-based injection  
B) Out-of-Band injection  
C) Blind — Boolean-based injection  
D) In-Band — UNION injection  

> **Respuesta correcta: C** — **Blind Boolean-based injection**: la aplicación no muestra mensajes de error ni datos visibles, pero responde de forma diferente a condiciones TRUE (`AND 1=1`) vs FALSE (`AND 1=2`). El atacante infiere información de la BBDD bit a bit observando estas diferencias en la respuesta.

---

### Pregunta 2 — WAITFOR DELAY
Un tester inyecta el siguiente payload en un parámetro de una aplicación con SQL Server: `1; IF EXISTS(SELECT * FROM creditcard) WAITFOR DELAY '0:0:10'--`. La respuesta tarda 10 segundos en llegar. ¿Qué confirma esto?

A) La tabla `creditcard` no existe en la BBDD  
B) La tabla `creditcard` existe en la BBDD (la condición fue TRUE y se ejecutó el delay)  
C) El servidor tiene bloqueadas las funciones de delay  
D) El parámetro no es inyectable  

> **Respuesta correcta: B** — En **Time-based blind SQL injection**, si la condición es TRUE, el servidor ejecuta `WAITFOR DELAY '0:0:10'` y la respuesta tarda 10 segundos. Si la respuesta llega inmediatamente, la condición es FALSE. Un delay de ~10 segundos confirma que la tabla `creditcard` **existe** en la BBDD y que el parámetro es inyectable.

---

### Pregunta 3 — Piggybacked Query
Un tester inyecta el siguiente input en un campo de búsqueda: `'; DROP TABLE Orders;--`. La aplicación ejecuta primero la búsqueda original y luego elimina la tabla. ¿Qué tipo de SQL injection es y cuál es el delimitador que permite la segunda query?

A) UNION injection; el delimitador es `UNION`  
B) Piggybacked Query (Stacked Queries); el delimitador es `;` (punto y coma)  
C) End-of-Line Comment injection; el delimitador es `--`  
D) Tautology injection; el delimitador es `OR`  

> **Respuesta correcta: B** — **Piggybacked Query (Stacked Queries)**: usa el **punto y coma `;`** como delimitador para ejecutar una query completamente independiente después de la original. La query original (`SELECT` de búsqueda) se ejecuta sin modificar, y la inyectada (`DROP TABLE Orders`) se ejecuta por separado a continuación.

---

### Pregunta 4 — Heavy Query
Un DBA ha deshabilitado las funciones `WAITFOR DELAY` y `BENCHMARK()` en su servidor de base de datos para prevenir ataques time-based. Sin embargo, un tester logra igualmente inferir información de la BBDD mediante delays. ¿Qué técnica usa?

A) Boolean-based blind injection con respuestas de tamaño diferente  
B) Heavy Query injection; usa múltiples JOINs en tablas del sistema para generar carga y delay artificial  
C) Out-of-band injection usando `xp_dirtree` para exfiltrar datos via DNS  
D) Error-based injection para revelar la estructura a través de mensajes de error  

> **Respuesta correcta: B** — **Heavy Query injection**: cuando las funciones de delay están deshabilitadas, el atacante genera el retraso haciendo queries que recuperan **cantidades masivas de datos** con múltiples JOINs en tablas del sistema (ej. `FROM all_users A, all_users B, all_users C`). La degradación del rendimiento del servidor actúa como señal.

---

### Pregunta 5 — Out-of-Band SQL Injection
Un tester descubre que una aplicación vulnerable usa Oracle Database. Quiere exfiltrar datos a su servidor externo pero el canal de respuesta HTTP está fuertemente filtrado. ¿Qué función de Oracle puede usar para enviar datos al servidor del atacante?

A) `xp_dirtree` — envía DNS requests  
B) `WAITFOR DELAY` — genera delays detectables  
C) `UTL_HTTP` — envía HTTP requests desde SQL/PL/SQL al servidor externo  
D) `BENCHMARK()` — ejecuta queries repetidamente  

> **Respuesta correcta: C** — En **Oracle**, la función `UTL_HTTP` (y su variante `UTL_FILE`) permite hacer **HTTP requests** desde código SQL/PL/SQL directamente a un servidor externo controlado por el atacante. En **SQL Server**, se usa `xp_dirtree` para enviar DNS requests. Son las funciones out-of-band específicas de cada DBMS en el CEH.

---

### Pregunta 6 — In-line Comments y Admin Bypass
Un atacante quiere bypassar la autenticación de una aplicación y además hacer que su cuenta tenga el campo `isAdmin` establecido en 1. Usa in-line comments para inyectar un INSERT malicioso. ¿Qué capacidad adicional ofrecen los in-line comments (`/* */`) que no ofrece el end-of-line comment (`--`)?

A) Los in-line comments permiten inyectar en múltiples parámetros simultáneamente  
B) Los in-line comments permiten eliminar espacios y bypassar blacklists que detectan palabras clave con espacios  
C) Los in-line comments permiten ejecutar procedimientos almacenados  
D) Los in-line comments permiten evadir el análisis de WAF automáticamente  

> **Respuesta correcta: B** — Los **in-line comments** (`/* */`) permiten: (1) **bypassar blacklists** que filtran palabras clave como `SELECT` o `UNION` detectando espacios — `SEL/**/ECT` evade el filtro de `SELECT`. (2) Eliminar espacios problemáticos. (3) Ofuscar queries. (4) Determinar versiones de BBDD. `--` solo comenta el resto de la línea sin estas capacidades de ofuscación.

---

### Pregunta 7 — ORDER BY para contar columnas
Un tester usa UNION injection y necesita determinar el número de columnas de la query original. Va probando `ORDER BY 1--`, `ORDER BY 2--`, etc. Cuando prueba `ORDER BY 6--` la aplicación devuelve error. ¿Cuántas columnas tiene la query original?

A) 6 columnas  
B) Más de 6 columnas  
C) 5 columnas  
D) Exactamente 7 columnas  

> **Respuesta correcta: C** — Si `ORDER BY 5--` funciona correctamente pero `ORDER BY 6--` devuelve error "Unknown column '6'", significa que la query tiene **5 columnas**. `ORDER BY N` funciona hasta el número de columnas; cuando N supera el número real de columnas, genera error. Por tanto: el mayor N sin error = número de columnas.

---

### Pregunta 8 — Categorías de SQL injection
Un auditor documenta los siguientes ataques en un informe de pentest: (1) Ataque que usa el mismo canal HTTP para el ataque y para ver los resultados directamente. (2) Ataque donde los resultados llegan a través de un servidor DNS del atacante. (3) Ataque donde el atacante infiere la respuesta midiendo el tiempo de respuesta. ¿Cómo se categorizan correctamente?

A) (1) Blind injection, (2) Out-of-band injection, (3) In-band injection  
B) (1) In-band injection, (2) Out-of-band injection, (3) Blind injection  
C) (1) In-band injection, (2) Blind injection, (3) Out-of-band injection  
D) Los tres son variantes de In-band injection  

> **Respuesta correcta: B** — (1) **In-band**: mismo canal HTTP para ataque y resultados visibles. (2) **Out-of-band**: canal diferente (DNS, HTTP al atacante, email) para los resultados. (3) **Blind**: mismo canal pero sin resultados visibles; el atacante infiere información por comportamiento (timing, diferencias de respuesta). Esta es la clasificación fundamental del CEH.

