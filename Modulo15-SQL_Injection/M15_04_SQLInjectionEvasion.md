# M15_04_SQLInjectionEvasion.md
> Módulo 15 / Subapartado 4 — SQL Injection Evasion Techniques

---

## 1. Conceptos y definiciones

### Contexto: IDS y evasión de SQL injection

Un IDS basado en firmas analiza strings de input contra una base de datos de patrones conocidos (regular expressions). Si hay coincidencia → alarma. Los atacantes evitan la detección transformando las queries para que no coincidan con ninguna firma.

El IDS sensor se coloca en el servidor de BBDD para inspeccionar sentencias SQL. Las técnicas de evasión de firma incluyen: encoding diferente, fragmentación de paquetes, expresiones equivalentes, manipulación de espacios.

---

### 🔴 Catálogo completo de técnicas de evasión (13)

| Técnica | Mecanismo core |
|---|---|
| **In-line Comment** | Inserta `/* */` entre keywords SQL para romper patrones detectables |
| **Char Encoding** | Usa función CHAR() para representar caracteres sin necesidad de quotes |
| **String Concatenation** | Rompe keywords SQL en fragmentos y los concatena (`+` MSSQL, `||` Oracle) |
| **Obfuscated Code** | Envuelve (wrapping) o cifra/hashea las strings, las descifra en runtime |
| **Manipulating White Spaces** | Elimina o añade espacios/tabs/CR/LF entre keywords sin cambiar la ejecución |
| **Hex Encoding** | Representa strings como valores hexadecimales (ej. `SELECT` → `0x73656c656374`) |
| **Sophisticated Matches** | Alternativas equivalentes a `OR 1=1` (ej. `OR 'x'='x'`, `OR 7>1`) |
| **URL Encoding** | Reemplaza caracteres con `%` + código ASCII hex (ej. `'` → `%27`) |
| **Null Byte** | Prepende `%00` a la query para terminar la string en C/C++ y bypassar IDS |
| **Case Variation** | Mezcla mayúsculas y minúsculas (`UnIoN sEleCt`) explotando la insensibilidad de SQL |
| **Declare Variables** | Almacena la query SQL en una variable y la ejecuta con EXEC() |
| **IP Fragmentation** | Divide el paquete IP en fragmentos pequeños → el IDS no puede reconstruir el ataque |
| **Variations** | Equivalentes matemáticos o de string a `OR 1=1` (ej. `OR 2=2`, `OR 1+1=2`) |

---

### 🔴 In-line Comment — Detalle

Inserta `/* */` entre keywords SQL para:
1. Romper patrones reconocibles por el IDS
2. Escribir SQL sin espacios en blanco
3. Dividir keywords internamente

```sql
-- Sin spaces, usando /* */ entre palabras
'/**/UNION/**/SELECT/**/password/**/FROM/**/Users/**/WHERE/**/username/**/LIKE/**/'admin'--

-- Dividir keywords internamente
'/**/UN/**/ION/**/SEL/**/ECT/**/password/**/FR/**/OM/**/Users/**/WHE/**/RE/**/username/**/LIKE/**/'admin'--
```

---

### 🔴 Char Encoding — Detalle

La función `CHAR()` convierte valores decimales (o hexadecimales) en caracteres. Permite representar strings sin usar quotes → bypassa firmas que buscan strings literales.

```sql
-- Load files: string '/etc/passwd' = char(47,101,116,99,47,112,97,115,115,119,100)
' union select 1,(load_file(char(47,101,116,99,47,112,97,115,115,119,100))),1,1,1;

-- Inject without quotes: string '%'
' or username like char(37);

-- Inject without quotes: string 'root' = char(114,111,111,116)
' union select * from users where login = char(114,111,111,116);

-- Check existing file: string 'n.ext'
' and 1=(if((load_file(char(110,46,101,120,116))<>char(39,39)),1,0));
```

Valores ASCII clave: `37='%'` · `39='\''` · `47='/'` · `114,111,111,116 = 'root'`

---

### 🔴 String Concatenation — Detalle y sintaxis por DBMS

Rompe keywords SQL en fragmentos para que la firma no los detecte. La sintaxis varía por DBMS:

| DBMS | Operador de concatenación | Ejemplo |
|---|---|---|
| **MSSQL** | `+` | `'; EXEC ('DRO'+'P T'+'AB'+'LE')` |
| **Oracle** | `||` | `'; EXECUTE IMMEDIATE 'SEL' \|\| 'ECT US' \|\| 'ER'` |
| **MySQL** | `CONCAT()` | `'; EXECUTE CONCAT('INSE','RT US','ER')` |

Ejemplo evasión OR 1=1: `' OR 'Simple' = 'Sim'+'ple'`

Las firmas del IDS comparan strings en ambos lados del `=` → una concatenación hace que la comparación no coincida con ninguna firma conocida.

---

### Obfuscated Code — Detalle

**Wrapping:** usa una utilidad wrap para ofuscar la query maliciosa → el IDS no la reconoce.

**SQL string obfuscation:** combina concatenación, cifrado/hashing y descifrado en runtime.

```sql
-- Obfuscación de 'qwerty':
Reverse(concat(if(1,char(121),2),0x74,right(left(0x567210,2),1),lower(mid('TEST',2,1)),replace(0x7074,'pt','w'),char(instr(123321,33)+110)))

-- Bypassing signatures con parentheses y mixed case:
/?id=(1)unIon(selEct(1),mid(hash,1,32)from(test.users))
/?id=1+union+(sELect'1',concat(login,hash)from+test.users)
/?id=(1)union(((((((select(1),hex(hash)from(test.users))))))))
```

---

### Manipulating White Spaces — Detalle

Añadir o eliminar espacios sin afectar la ejecución SQL:

- Añadir caracteres especiales: **tab**, **carriage return (CR)**, **line feed (LF)**
- La firma `UNION SELECT` es diferente de `UNION\nSELECT` → IDS no detecta
- Eliminar espacios: `'OR'1'='1'` (sin espacios) → ejecuta igual

```sql
-- UNION con newline entre palabras
UNION
SELECT password FROM Users

-- Sin spaces: equivalente a ' OR '1'='1'
'OR'1'='1'
```

---

### 🔴 Hex Encoding — Detalle

Representa strings como valores hexadecimales. La mayoría de IDS no reconocen encoding hex.

```sql
-- Hex de 'SELECT': 0x73656c656374
; declare @x varchar(80); set @x = X73656c656374 20404076657273696f6e; EXEC(@x)
-- No usa comillas simples
```

Conversiones clave:
- `SELECT @@version` = `0x73656c656374204040...`
- `DROP Table CreditCard` = `0x44524f50205461626c652043726564697443617264`
- `INSERT into USERS ('certifiedhacker','qwerty')` = `0x494e5345525420696e746f...`

---

### 🔴 Sophisticated Matches — Alternativas a `OR 1=1`

Las firmas del IDS capturan variaciones de `OR 1=1` pero hay alternativas equivalentes:

```sql
OR 'john' = 'john'           -- string comparison
' OR 'microsoft' = 'micro'+'soft'  -- concatenation
' OR 'movies' = N'movies'    -- Unicode prefix N
' OR 'software' like 'soft%' -- LIKE wildcard
' OR 7 > 1                   -- numeric comparison
' OR 'best' > 'b'            -- string comparison
' OR 'whatever' IN ('whatever')  -- IN clause
' OR 5 BETWEEN 1 AND 7       -- BETWEEN clause
```

El truco de la `N'string'`: añadir el prefijo `N` convierte a Unicode → evade sistemas avanzados.

**SQL injection characters clave:**

| Carácter | Función |
|---|---|
| `'` o `"` | String indicators |
| `--` o `#` | Single-line comment |
| `/* ... */` | Multi-line comment |
| `+` | Addition / concatenation / space en URL |
| `||` | Concatenate (Oracle) |
| `%` | Wildcard |
| `@variable` | Local variable |
| `@@variable` | Global variable |
| `waitfor delay '0:0:10'` | Time delay |

---

### 🔴 URL Encoding — Detalle

Reemplaza caracteres con `%` + código ASCII en hexadecimal.

```sql
-- Comilla simple: ASCII 0x27 → URL encoded: %27
-- Espacio: → %20
-- = → %3D
-- ' UNION SELECT Password FROM Users_Data WHERE name='Admin'--
-- URL encoded:
%27%20UNION%20SELECT%20Password%20FROM%20Users_Data%20WHERE%20name%3D%27Admin%27%E2%80%94
```

**Double URL Encoding:** cuando el filtro decodifica una vez, el resultado sigue encoded:
- `'` → `%27` (simple URL encoding)
- `%27` → `%2527` (double: `%` = `%25`, entonces `%` → `%25` y `27` queda igual)

```
Double-encoded query: %2527%2520UNION%2520SELECT...
```

---

### Null Byte — Detalle

**Problema:** aplicaciones web usan PHP/ASP (donde NULL no termina la string) junto con C/C++ (donde `\0` sí termina la string). Esta discrepancia es explotable.

```sql
-- Query normal (bloqueada por IDS):
' UNION SELECT Password FROM Users WHERE UserName='admin'--

-- Con null byte prepended (bypassa IDS):
%00' UNION SELECT Password FROM Users WHERE UserName='admin'--
```

El `%00` hace que el IDS lea la string como terminada antes del payload → no detecta la inyección.

---

### Case Variation — Detalle

SQL es **case insensitive** por defecto en la mayoría de DBMS. Las firmas basadas en expresiones regulares con opción case-insensitive también son bypasseables mezclando cases:

```sql
-- Firma detecta estas dos:
union select user_id, password from admin where user_name='admin'--
UNION SELECT USER_ID, PASSWORD FROM ADMIN WHERE USER_NAME='ADMIN'--

-- Bypassa la firma:
UnIoN sEleCt UsEr_iD, PaSSwOrd fROm aDmiN wHeRe UseR_NamE='AdMIn'--
```

---

### Declare Variables — Detalle

El atacante almacena la query maliciosa en una variable y la ejecuta mediante EXEC():

```sql
; declare @sqlvar nvarchar(70); 
set @sqlvar = (N'UNI' + N'ON' + N' SELECT' + N'Password'); 
EXEC(@sqlvar)
```

La variable `@sqlvar` nunca contiene la string completa detectada por el IDS — se construye mediante concatenación y se ejecuta dinámicamente.

---

### IP Fragmentation — Detalle

El atacante divide el paquete IP en fragmentos pequeños. El IDS necesita **reassemblar** los fragmentos para detectar el ataque; si no puede, el payload pasa desapercibido.

Variantes de fragmentación para complicar el reassembly:
- Pausar entre fragmentos (esperar timeout del IDS)
- Enviar en orden inverso
- Enviar en orden correcto excepto el primer fragmento (enviado último)
- Enviar en orden correcto excepto el último fragmento (enviado primero)
- Enviar en orden aleatorio

---

### Variation — Detalle

El objetivo: tener una cláusula WHERE que **siempre evalúe como TRUE**. Cualquier comparación matemática o de string que sea siempre verdadera funciona:

```sql
-- Estas queries devuelven el mismo resultado:
SELECT * FROM accounts WHERE userName='Bob' OR 1=1 --
SELECT * FROM accounts WHERE userName='Bob' OR 2=2 --
SELECT * FROM accounts WHERE userName='Bob' OR 1+1=2 --
SELECT * FROM accounts WHERE userName='Bob' OR "evade"="ev"+"ade" --
```

Hay **infinitas posibilidades** de variation → las firmas no pueden cubrir todas.

---

## 2. Exam Traps ⚠️

⚠️ **[In-line Comment: dos usos distintos]**
In-line comments (`/* */`) se usan en dos contextos distintos del CEH: (1) como técnica de evasión IDS (fragmentar keywords SQL para romper firmas) y (2) como tipo de SQL injection en-band (dar privilegios admin). El examen puede mezclar los contextos. La diferencia: en evasión, el objetivo es no ser detectado; en injection type, el objetivo es manipular la query.

⚠️ **[CHAR() para MySQL sin double quotes]**
El libro especifica que `CHAR()` puede usarse para SQL injection en MySQL **sin double quotes**. El examen puede presentar CHAR() como técnica solo de MSSQL → es válida en múltiples DBMS.

⚠️ **[String Concatenation: sintaxis diferente por DBMS]**
MSSQL usa `+`, Oracle usa `||`, MySQL usa `CONCAT()`. El examen puede intercambiar los operadores entre DBMS. Regla: `+` = MSSQL; `||` = Oracle; `CONCAT()` = MySQL.

⚠️ **[URL Encoding: ' = %27, double encoding de ' = %2527]**
Simple: `'` → `%27`. Double: `%27` se vuelve a encodear como `%2527` (porque `%` = `%25`). El examen puede presentar `%2527` como simple encoding o preguntar qué representa `%27` → es comilla simple.

⚠️ **[Null Byte: mecanismo exacto]**
El `%00` funciona porque C/C++ interpreta NULL como fin de string, mientras PHP/ASP no. Esto hace que el IDS vea la string como terminada antes del payload. El examen puede preguntar por qué funciona el null byte → es la discrepancia entre C/C++ y lenguajes de alto nivel.

⚠️ **[Case Variation: SQL es case insensitive pero las FIRMAS del IDS también]**
La razón por la que case variation funciona es que las expresiones regulares de las firmas IDS también son case-insensitive. Paradójicamente, esto es lo que crea la vulnerabilidad de evasión mediante mixing de cases. El examen puede presentar "SQL es case sensitive" → incorrecto.

⚠️ **[Hex Encoding: no usa comillas simples]**
La versión hex con `declare @x` no usa comillas simples (`'`). Este es el dato diferenciador del hex encoding respecto a otros métodos.

⚠️ **[Sophisticated Matches: el prefix N' para Unicode]**
`N'string'` es la versión Unicode de la string. Añadir `N` a la segunda comparación (`OR 'john'=N'john'`) es especialmente útil para evadir sistemas avanzados. El examen puede preguntar qué hace el prefijo N en el contexto de evasión.

⚠️ **[IP Fragmentation: el IDS necesita reassemblar antes de detectar]**
El IDS analiza cada fragmento individualmente → no puede encontrar coincidencia con la firma porque el payload está dividido. Para detectar el ataque, el IDS necesita primero reassemblar los fragmentos. El examen puede presentar IP fragmentation como efectiva contra todos los IDS → solo contra los que no reassemblan.

⚠️ **[Variation: infinitas posibilidades implica que las firmas no pueden ser completas]**
La razón por la que Variation es efectiva como evasión es precisamente que hay infinitas combinaciones matemáticas o de string que son siempre TRUE. El examen puede preguntar por qué las firmas no pueden detectar todas las variaciones de `OR 1=1`.

---

## 3. Nemotécnicos

**13 técnicas de evasión** → **"IC-CE-SC-OC-WS-HE-SM-UE-NB-CV-DV-IPF-V"**:
**I**n-line **C**omment · **C**har **E**ncoding · **S**tring **C**oncat · **O**bfuscated **C**ode · **W**hite **S**paces · **H**ex **E**ncoding · **S**ophisticated **M**atches · **U**RL **E**ncoding · **N**ull **B**yte · **C**ase **V**ariation · **D**eclare **V**ariables · **IP F**ragmentation · **V**ariation

**Concatenación por DBMS** → **"M+=, O=||, MySQL=CONCAT()"**:
- **M**SSQL = `+` · **O**racle = `||` · **MySQL** = `CONCAT()`

**URL encoding claves:**
- `'` = `%27`
- Espacio = `%20`
- `=` = `%3D`
- `%` = `%25` (doble encoding: `'` → `%27` → `%2527`)

**Char() valores clave:**
- `37` = `%` (wildcard)
- `39` = `'` (single quote)
- `47` = `/`
- `114,111,111,116` = `root`

**Hex encoding de keywords:**
- `SELECT` = `0x73656c656374`

**Sophisticated Matches alternativas** → **"string=str+ing · N'str' · LIKE · > · IN · BETWEEN · 1+1=2"**

**IP Fragmentation variantes (5)** → **"Pause · Reverse · First-Last · Last-First · Random"**

---

## 4. Flashcards

**Q:** ¿Cuáles son las 13 técnicas de evasión de SQL injection?
**A:** In-line Comment, Char Encoding, String Concatenation, Obfuscated Code, Manipulating White Spaces, Hex Encoding, Sophisticated Matches, URL Encoding, Null Byte, Case Variation, Declare Variables, IP Fragmentation, Variation.

---

**Q:** ¿Qué hace el In-line Comment como técnica de evasión IDS?
**A:** Inserta `/* */` entre keywords SQL para fragmentarlas y romper los patrones detectables por las firmas. Permite además escribir SQL sin espacios en blanco, dividiendo keywords internamente (ej. `UN/**/ION`).

---

**Q:** ¿Para qué se usa la función CHAR() como técnica de evasión y en qué DBMS funciona sin double quotes?
**A:** Convierte valores decimales/hexadecimales en caracteres → permite representar strings sin quotes literales, evadiendo firmas que buscan strings. En MySQL puede usarse para SQL injection sin double quotes.

---

**Q:** ¿Cuál es el operador de concatenación en MSSQL, Oracle y MySQL para evasión?
**A:** MSSQL: `+`. Oracle: `||`. MySQL: `CONCAT()`.

---

**Q:** ¿Qué representa `%27` en URL encoding?
**A:** La comilla simple `'` (ASCII 0x27). `%2527` es el double URL encoding de `'` (porque `%` se encodea como `%25`).

---

**Q:** ¿Por qué funciona el Null Byte (`%00`) para evadir IDS?
**A:** Porque en C/C++ el carácter NULL termina la string, mientras que PHP/ASP no lo interpretan así. El IDS (implementado en C/C++) ve la string terminada antes del payload → no detecta la inyección.

---

**Q:** ¿Qué representan `0x73656c656374` y `SELECT @@version` en hex encoding?
**A:** `0x73656c656374` = `SELECT`. La statement `; declare @x varchar(80); set @x = 0x73656c656374...; EXEC(@x)` no usa comillas simples.

---

**Q:** ¿Cuál es la representación hex de `DROP Table CreditCard`?
**A:** `0x44524f50205461626c652043726564697443617264`

---

**Q:** ¿Cómo puede un atacante evadir la firma `OR 'john'='john'` en sistemas avanzados?
**A:** Usando el prefijo Unicode N: `OR 'john'=N'john'` — convierte la segunda string a Unicode, evadiendo sistemas de firma avanzados.

---

**Q:** ¿Por qué Case Variation funciona como evasión?
**A:** Porque SQL es case insensitive por defecto en la mayoría de DBMS, por lo que `UnIoN sEleCt` ejecuta igual que `UNION SELECT`. Las firmas IDS basadas en regex con opción case-insensitive también fallan en detectar el mixing.

---

**Q:** ¿Cómo funciona Declare Variables como evasión IDS?
**A:** El atacante almacena la query maliciosa en una variable usando concatenación de fragmentos (`N'UNI'+N'ON'+N' SELECT'`) y la ejecuta con `EXEC()`. El IDS nunca ve la string completa → no puede hacer matching de firma.

---

**Q:** ¿Qué variantes de IP fragmentation se usan para complicar el reassembly del IDS?
**A:** Pausar entre fragmentos, enviar en orden inverso, enviar en orden correcto excepto el primer fragmento (enviado último), enviar en orden correcto excepto el último (enviado primero), enviar en orden aleatorio.

---

**Q:** ¿Por qué la técnica de Variation hace que las firmas IDS sean incompletas?
**A:** Porque hay infinitas posibilidades matemáticas y de string que evalúan como TRUE: `OR 2=2`, `OR 1+1=2`, `OR "evade"="ev"+"ade"`, etc. Las firmas no pueden cubrir todas las posibles combinaciones.

---

**Q:** ¿Qué es el Sophisticated Match `OR 'movies' = N'movies'` y qué lo hace diferente de `OR 1=1`?
**A:** Es una alternativa equivalente que usa prefijo Unicode N en la segunda string. Mientras `OR 1=1` es una comparación numérica clásica que las firmas detectan, `N'movies'` convierte a Unicode evadiendo firmas avanzadas.

---

**Q:** ¿Cuál es el resultado de char(114,111,111,116) y para qué se usa en SQL injection?
**A:** Representa la string `root`. Se usa para inyección sin quotes: `' union select * from users where login = char(114,111,111,116);`

---

**Q:** ¿Qué hace Manipulating White Spaces como técnica de evasión?
**A:** Elimina o añade espacios/tabs/CR/LF entre keywords SQL sin afectar la ejecución. `UNION SELECT` y `UNION\nSELECT` son detectados diferente por el IDS pero ejecutan igual.

---

**Q:** ¿Qué carácter se usa como wildcard en SQL y cuál es su valor en CHAR()?
**A:** `%` es el wildcard. Su valor en CHAR() es 37: `char(37)` = `%`. Ejemplo: `' or username like char(37);` — busca cualquier username.

---

**Q:** ¿Por qué Hex Encoding es efectivo contra la mayoría de IDS?
**A:** Porque la mayoría de IDS no reconocen hex encodings. La string `SELECT` representada como `0x73656c656374` no coincide con ninguna firma de texto plano. Además, la versión hex no requiere comillas simples.

---

**Q:** ¿Cuáles son los comentarios de SQL que los atacantes pueden explotar para evasión?
**A:** `--` (single-line comment), `#` (single-line en MySQL), `/* ... */` (multi-line/in-line comment).

---

**Q:** ¿Qué combinaciones de true statements se pueden usar como variación de `OR 1=1`?
**A:** `OR 2=2`, `OR 1+1=2`, `OR "evade"="ev"+"ade"`, `OR 5 BETWEEN 1 AND 7`, `OR 7 > 1`, `OR 'software' like 'soft%'`, `OR 'whatever' IN ('whatever')`. Todas evalúan como TRUE.

---

## 5. Confusión frecuente

**Char Encoding vs. Hex Encoding — ambas codifican caracteres pero de forma diferente**
- Char Encoding: usa la función `CHAR(valor_decimal)` de SQL para representar caracteres. Trabaja dentro de la query SQL. Ej: `char(47)` = `/`.
- Hex Encoding: convierte strings completas a su representación hexadecimal con `0x` prefix. La string se pasa directamente al engine. Ej: `0x73656c656374` = `SELECT`.
- Criterio: "función SQL que convierte número a carácter" → Char Encoding. "String representada como 0x..." → Hex Encoding.

---

**URL Encoding vs. Null Byte — ambas modifican caracteres en la URL**
- URL Encoding: reemplaza caracteres especiales con `%HH`. El filtro puede no parsear la query encoded. Simple: `%27`=`'`. Double: `%2527`=encoded de `%27`.
- Null Byte: prepende `%00` al payload para terminar la string antes del payload en lenguajes C/C++. No modifica el payload en sí, solo añade el terminador al inicio.
- Criterio: "codificar caracteres especiales para evadir filtros de parsing" → URL Encoding. "Terminar prematuramente la string en el IDS usando el terminador C/C++" → Null Byte.

---

**String Concatenation vs. Declare Variables — ambas evitan escribir keywords completos**
- String Concatenation: rompe la keyword directamente en la query: `'DRO'+'P'`. Simple y directo.
- Declare Variables: almacena la query completa (construida por concatenación) en una variable de BBDD y la ejecuta con EXEC(). Más sofisticado; la query nunca aparece como string en la sentencia SQL directamente.
- Criterio: "keyword dividida directamente en la query" → String Concatenation. "Query almacenada en variable y ejecutada por EXEC()" → Declare Variables.

---

**Sophisticated Matches vs. Variation — ambas son alternativas a OR 1=1**
- Sophisticated Matches: comparaciones de strings o números que son siempre verdaderas como alternativas exactas a `OR 1=1`. Ej: `OR 'john'='john'`, `OR 5 BETWEEN 1 AND 7`.
- Variation: el concepto más amplio — cualquier expresión matemática o de string que siempre evalúa TRUE. Incluye `OR 2=2`, `OR 1+1=2`, `OR "evade"="ev"+"ade"`. Hay infinitas variaciones.
- Ambas solapan en función pero el libro las presenta como técnicas distintas. Criterio en examen: "alternativas específicas de string como `OR 'x'=N'x'`" → Sophisticated Matches. "Concepto general de infinitas variaciones matemáticas" → Variation.

---

**In-line Comment como evasión vs. como tipo de injection**
- Como evasión IDS: `/* */` fragmenta keywords para que no coincidan con firmas. El objetivo es no ser detectado.
- Como tipo de injection (M15_02): `/* */` permite dar privilegios admin inyectando `isAdmin=1` entre comentarios. El objetivo es manipular la lógica de la aplicación.
- Criterio: "¿el objetivo es bypassar el IDS?" → evasión. "¿el objetivo es manipular la query de la aplicación?" → tipo de injection.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un IDS monitoriza queries SQL buscando la firma `UNION SELECT`. Un pentester inyecta: `'/**/UNION/**/SELECT/**/password/**/FROM/**/Users--`. El IDS no genera alerta. ¿Qué técnica de evasión usa?

A) Case Variation  
B) In-line Comment  
C) Hex Encoding  
D) URL Encoding  

**Respuesta correcta: B**
**In-line Comment** (`/* */`) inserta comentarios entre keywords SQL para fragmentar los patrones detectables por el IDS. La firma `UNION SELECT` no coincide con `UNION/**/SELECT` aunque SQL ejecuta la query correctamente porque los comentarios son ignorados por el motor.

---

**P2.** Un pentester necesita inyectar el path `/etc/passwd` en una query MySQL sin usar comillas simples para evadir los filtros del WAF. ¿Qué técnica usa?

A) `CHAR(47,101,116,99,47,112,97,115,115,119,100)`  
B) `0x2f6574632f706173737764`  
C) `%2F%65%74%63%2F%70%61%73%73%77%64`  
D) `concat('/','etc','/','passwd')`  

**Respuesta correcta: A**
**Char Encoding** convierte la string `/etc/passwd` usando la función `CHAR()` con valores decimales: `CHAR(47,101,116,99,47,112,97,115,115,119,100)` = `/etc/passwd` donde 47='/'. Permite representar strings **sin comillas** literales. Opción B es Hex Encoding (también válida pero no usa CHAR). La opción A es la que específicamente usa la función CHAR() con valores decimales.

---

**P3.** Un atacante usa la siguiente query en Oracle para evadir un IDS que busca la keyword completa `EXECUTE`: `'; EXECUTE IMMEDIATE 'SEL' || 'ECT US' || 'ER'`. ¿Qué técnica usa?

A) Case Variation con Oracle functions  
B) String Concatenation usando el operador `||` de Oracle  
C) Declare Variables con EXECUTE IMMEDIATE  
D) In-line Comment fragmentando la keyword  

**Respuesta correcta: B**
**String Concatenation** en Oracle usa el operador `||` para romper keywords SQL en fragmentos. `'SEL' || 'ECT US' || 'ER'` construye `SELECT USER` en tiempo de ejecución. La firma del IDS no encuentra ningún fragmento que coincida con la keyword completa. MSSQL usa `+`, MySQL usa `CONCAT()`.

---

**P4.** Un IDS tiene configuradas firmas case-insensitive para detectar `union select`. Un pentester inyecta `UnIoN sEleCt UsEr_iD, PaSSwOrd fROm aDmiN--`. El IDS no detecta el ataque. ¿Por qué funciona esta técnica?

A) Porque SQL distingue mayúsculas de minúsculas  
B) Porque el IDS case-insensitive solo detecta todo en mayúsculas o todo en minúsculas  
C) Porque SQL es case insensitive y ejecuta la query, pero el mixing de cases crea un patrón diferente a las firmas del IDS  
D) Porque las comillas simples bypassan el motor de regex del IDS  

**Respuesta correcta: C**
**Case Variation** funciona porque SQL es case insensitive (ejecuta `UnIoN` como `UNION`) pero las firmas IDS basadas en regex que no contemplan el mixing de cases no detectan `UnIoN sEleCt`. Las firmas case-insensitive detectan `union select` y `UNION SELECT` pero fallan con mezclas no anticipadas.

---

**P5.** Un pentester inyecta `%00' UNION SELECT Password FROM Users WHERE UserName='admin'--`. El IDS no detecta el ataque pero la inyección se ejecuta correctamente. ¿Qué técnica de evasión usa y por qué funciona?

A) Double URL Encoding — decodifica dos veces  
B) Null Byte — termina prematuramente la string en el IDS (implementado en C/C++)  
C) Case Variation — el %00 actúa como separador de mayúsculas  
D) In-line Comment — %00 es equivalente a `/* */`  

**Respuesta correcta: B**
**Null Byte** (`%00`) funciona porque el IDS está implementado en C/C++ donde `\0` termina la string. Al preprender `%00`, el IDS lee la string como terminada antes del payload malicioso. PHP/ASP (donde corre la aplicación web) no interpreta NULL como fin de string, por lo que la inyección se ejecuta normalmente.

---

**P6.** Un atacante inyecta `; declare @sqlvar nvarchar(70); set @sqlvar = (N'UNI' + N'ON' + N' SELECT' + N'Password'); EXEC(@sqlvar)` en un campo vulnerable. ¿Qué técnica de evasión usa?

A) String Concatenation directa en la query  
B) In-line Comment con NULL bytes  
C) Declare Variables — almacena la query en variable y la ejecuta con EXEC()  
D) Hex Encoding con variables de MSSQL  

**Respuesta correcta: C**
**Declare Variables** almacena la query maliciosa en una variable (`@sqlvar`) construida por concatenación de fragmentos y la ejecuta mediante `EXEC()`. La firma del IDS nunca ve la string completa `UNION SELECT` — la variable se construye en tiempo de ejecución y ningún fragmento individual coincide con la firma.

---

**P7.** Un IDS bloquea queries que contienen `OR 1=1` y sus variantes directas. Un pentester usa `' OR 5 BETWEEN 1 AND 7--` y el IDS no lo detecta. ¿Qué técnica usa y por qué hay infinitas posibilidades de evasión?

A) Sophisticated Matches — número limitado de alternativas documentadas  
B) Variation — hay infinitas expresiones matemáticas/de string que siempre evalúan como TRUE  
C) In-line Comment — los números están separados por comentarios  
D) URL Encoding — los números se representan en hex  

**Respuesta correcta: B**
**Variation** explota el hecho de que hay **infinitas posibilidades** de expresiones que siempre evalúan como TRUE (cualquier comparación que sea siempre verdadera funciona como alternativa a `OR 1=1`). `OR 5 BETWEEN 1 AND 7`, `OR 2=2`, `OR 1+1=2`, `OR "text"="text"` — las firmas no pueden cubrir todas las variantes posibles.

---

**P8.** Un analista de seguridad captura el siguiente payload URL: `%2527%2520UNION%2520SELECT%2520password%2520FROM%2520users`. ¿Qué técnica de evasión representa y cómo se descifra?

A) Simple URL Encoding — `%25` = espacio, `%27` = `'`  
B) Double URL Encoding — `%2527` = `%27` = `'`, `%2520` = `%20` = espacio  
C) Hex Encoding con prefijo 0x  
D) Char Encoding con valores ASCII  

**Respuesta correcta: B**
**Double URL Encoding**: `%25` es la URL encoding de `%`. Por tanto `%2527` = `%25` + `27` = `%27` = `'` (comilla simple). `%2520` = `%25` + `20` = `%20` = espacio. Cuando el filtro decodifica una vez, obtiene `%27 UNION SELECT...` (aún encoded), permitiendo que pase. El motor SQL decodifica de nuevo y obtiene la query final.

---

**P9.** Un IDS analiza tráfico packet por packet y no puede detectar un ataque SQL injection porque los datos están distribuidos en múltiples fragmentos IP enviados en orden aleatorio. ¿Qué técnica usa el atacante?

A) URL Encoding fragmentada  
B) IP Fragmentation  
C) HTTP Parameter Fragmentation (HPF)  
D) CRLF Injection  

**Respuesta correcta: B**
**IP Fragmentation** divide el paquete IP en fragmentos pequeños. El IDS necesita reassemblar los fragmentos antes de analizar el contenido, pero si los fragmentos llegan en orden aleatorio, fuera de orden o con pausas, el IDS no puede reconstruir el payload completo y el ataque pasa desapercibido.

---

**P10.** Un pentester quiere inyectar la string `SELECT @@version` en un campo de SQL Server sin usar comillas y sin que las firmas del IDS detecten la keyword `SELECT`. ¿Cuál es el hex encoding correcto de `SELECT`?

A) `0x53454c454354`  
B) `0x73656c656374`  
C) `0x44524f50`  
D) `0x494e53455254`  

**Respuesta correcta: B**
`0x73656c656374` es el hex encoding de `SELECT` (en minúsculas: s=73, e=65, l=6c, e=65, c=63, t=74). El **Hex Encoding** no usa comillas simples y la mayoría de IDS no reconocen las representaciones hex. La opción A es `SELECT` en mayúsculas. `0x44524f50` = `DROP`. `0x494e53455254` = `INSERT`.

---

**P11.** Una firma IDS detecta el patrón `OR 'x'='x'`. Un pentester lo evade con `OR 'john'=N'john'`. ¿Qué diferencia hace que la segunda versión evada sistemas avanzados?

A) El uso de comillas dobles en lugar de simples  
B) El prefijo N convierte la segunda string a Unicode, creando un patrón no cubierto por la firma  
C) La función N() normaliza la comparación  
D) `john` es un username real que hace la comparación más difícil de detectar  

**Respuesta correcta: B**
El prefijo **N** antes de una string en SQL convierte esa string a **Unicode (nvarchar)**. `N'john'` no coincide con la firma `'john'` porque el tipo de datos es diferente. Esta técnica de **Sophisticated Matches** con prefijo N es especialmente efectiva para evadir sistemas de firma avanzados que no contemplan comparaciones Unicode.

---

**P12.** Un WAF bloquea queries con `UNION SELECT` en texto plano. Un pentester inyecta `/?a=1+union/*&b=*/select+1,password+from+users+where+id=1`. El WAF no detecta el ataque. ¿Qué técnica WAF bypass es esta?

A) HPP — HTTP Parameter Pollution  
B) HPF — HTTP Parameter Fragmentation  
C) Buffer Overflow del WAF  
D) Normalization bypass  

**Respuesta correcta: B**
**HPF (HTTP Parameter Fragmentation)** fragmenta la query SQL entre múltiples parámetros usando `/*` y `*/` como delimitadores: el parámetro `a` contiene `1+union/*` y `b` contiene `*/select+1,password...`. El WAF analiza cada parámetro por separado y no ve `UNION SELECT` completo. Cuando el servidor concatena los parámetros, la query se reconstrye completa.
