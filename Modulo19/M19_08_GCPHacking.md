# M19_08 — Google Cloud Platform (GCP) Hacking
**Módulo 19 / Subapartado 8 — Google Cloud Hacking**

---

## 1. Conceptos y definiciones

### 🔴 Herramientas de GCP Hacking — mapa general

| Herramienta | Función | Alcance |
|------------|---------|---------|
| **Google Cloud CLI** (`gcloud`, `gsutil`) | Enumeración nativa de organizaciones, proyectos, buckets, IAM, Compute, SQL | GCP nativo |
| **gcp_service_enum** | Script Python; descubre servicios GCP (Compute Engine, Cloud Storage); necesita service account key | GCP |
| **GCP Scanner** | Evalúa nivel de acceso de credenciales; soporta GCE, GCS, GKE, App Engine, Cloud SQL, BigQuery, Spanner, Pub/Sub, Cloud Functions, BigTable, CloudStore, KMS | GCP (amplio) |
| **cloud_enum** | Multi-cloud OSINT (AWS, Azure, GCP); enumera buckets GCP, Firebase, App Engine, Cloud Functions | Multi-cloud |
| **GCPBucketBrute** | Enumera buckets GCS; comprueba permisos y si son escalables a admin; usa `TestIamPermissions` API | GCS buckets |
| **GCP Privilege Escalation Scanner** | Script Python; identifica vulnerabilidades de escalada en políticas IAM; genera `privesc_methods.txt` y `setIamPolicy_methods.txt` | GCP IAM |
| **GCPGoat** | Infraestructura GCP vulnerable por diseño; práctica de XSS, SSRF, bucket misconfig, IAM escalada | Lab/practice |

---

### 🔴 Google Cloud CLI — comandos de enumeración por categoría

#### Organizaciones, proyectos y buckets

```bash
# Listar organizaciones accesibles
gcloud organizations list

# Ver carpetas dentro de una organización
gcloud resource-manager folders list --organization=<organization_id>

# Listar proyectos activos (donde el usuario tiene owner/editor/browser/viewer)
gcloud projects list

# Listar todos los buckets del proyecto por defecto
gsutil ls

# Listar buckets de un proyecto específico
gsutil ls -p <project_id>

# Obtener permisos (IAM policy) de un bucket específico
gsutil iam get gs://<bucket_name>

# Listar contenido de un bucket (objetos + subdirectorios)
gsutil ls gs://<bucket_name>

# Listar contenido de forma recursiva
gsutil ls -r gs://<bucket_name>
```

#### Service Accounts

```bash
# Listar service accounts del proyecto actual
gcloud iam service-accounts list

# Listar service accounts de un proyecto específico
gcloud iam service-accounts list --project <project_id>

# Obtener todos los roles asociados a una service account
gcloud projects get-iam-policy <project-id>

# Obtener access token de una service account
gcloud auth print-access-token --account=<service-account-email>
```

#### Compute Engine y Cloud SQL

```bash
# Listar todas las instancias Compute Engine del proyecto
gcloud compute instances list

# Detalles completos de una instancia en una zona específica
gcloud compute instances describe <instance> --zone <zone>

# Service accounts asociadas a una instancia Compute Engine
gcloud compute instances describe <INSTANCE> --zone=<zone> --format="table(serviceAccounts.scopes)"

# Listar instancias SQL del proyecto
gcloud sql instances list

# Enumerar bases de datos de una instancia SQL
gcloud sql databases list --instance=<instance_name>
```

#### IAM Roles y Políticas

```bash
# Listar roles predefinidos o custom (incluyendo eliminados con --show-deleted)
gcloud iam roles list [--show-deleted] [--organization=<org>] [--project=<project_id>]

# Metadata y permisos de un rol IAM específico
gcloud iam roles describe <role_id> [--organization=<org>] [--project=<project_id>]

# Obtener políticas IAM de una organización
gcloud organizations get-iam-policy <organization_id>

# Obtener políticas IAM de un proyecto
gcloud projects get-iam-policy <project_id>

# Obtener políticas IAM de una carpeta
gcloud resource-manager folders get-iam-policy <folder_id>
```

---

### Herramientas especializadas — sintaxis

#### gcp_service_enum

```bash
gcp_enum_services.py -f <service_account_key_file> --output-file <output_file>
```
Requiere: service account key file para autenticación.

#### GCP Scanner

```bash
python3 scanner.py -o <output_file> -g <gcloud_profile_path>
# -o: fichero de output
# -g: path al perfil gcloud con credenciales
```

**Credenciales soportadas por GCP Scanner:**
- GCP VM instance metadata
- User credentials de perfiles gcloud
- OAuth2 Refresh Tokens con cloud-platform scope
- GCP service account keys en JSON

**Recursos GCP soportados:** GCE, GCS, GKE, App Engine, Cloud SQL, BigQuery, Spanner, Pub/Sub, Cloud Functions, BigTable, CloudStore, KMS.

#### cloud_enum (Multi-cloud)

```bash
# Solo GCP (deshabilitando AWS y Azure para mayor velocidad)
cloud_enum.py -k <keyword> --disable-aws --disable-azure
```

Enumera: buckets GCP públicos, Firebase Realtime Databases, Google App Engine sites, Cloud Functions, Firebase apps abiertas.
Alternativa para buckets GCP: **GrayhatWarfare**.

#### GCP Privilege Escalation Scanner (3 pasos)

```bash
# Step 1: Listar permisos de todos los miembros del proyecto
python3 enumerate_member_permissions.py --project-id test-<project_ID>

# Step 2: Escanear vulnerabilidades de escalada con los permisos enumerados
python3 check_for_privesc.py
```

**Ficheros generados:**

| Fichero | Contenido |
|---------|----------|
| `all_org_folder_proj_sa_permissions.json` | Todos los miembros y sus permisos en el proyecto |
| `privesc_methods.txt` | Métodos identificados para escalar privilegios en GCP |
| `setIamPolicy_methods.txt` | Métodos que involucran `setIamPolicy` explotables para escalada |

#### GCPBucketBrute

Verifica permisos de buckets haciendo petición HTTP directa a:
```
https://www.googleapis.com/storage/v1/b/BUCKETNAME/iam
```

- Si `allUsers` o `allAuthenticatedUsers` pueden leer la política → respuesta válida (bucket público o semi-público)
- Si no → "access denied"

Usa la API **`TestIamPermissions`** de Google Storage: se pasa un bucket name + lista de permisos → devuelve los permisos que el atacante posee.

Si el atacante tiene suficientes permisos → GCPBucketBrute muestra que el bucket es **vulnerable a privilege escalation** hasta nivel administrator.

---

### Creación de backdoors con IAM en GCP

```bash
# Crear nuevo rol con permisos elevados (definidos en role-definition.yaml)
gcloud iam roles create <ROLE_NAME> --project=<PROJECT_ID> --file=role-definition.yaml

# Asignar el rol a una service account (persistencia)
gcloud projects add-iam-policy-binding <PROJECT_ID> \
  --member=serviceAccount:<SA>@<PROJECT_ID>.iam.gserviceaccount.com \
  --role=roles/<ROLE_NAME>
```

> Solo ejecutable tras escalar privilegios a nivel administrativo.

---

### GCPGoat — escenarios de práctica

| Escenario | Técnica |
|-----------|---------|
| SSRF | Obtener código fuente de Cloud Function y volcar DB → tomar cuenta admin |
| Misconfigured Storage Bucket | Usar bucket policies incorrectas → acceso admin al bucket |
| Lateral Movement | Credenciales de VM de bajo privilegio en dev bucket → acceder a Compute Instances de alto privilegio |

---

## 2. Exam Traps ⚠️

⚠️ **[gsutil vs. gcloud — namespacing]**
`gsutil` es la herramienta CLI para **Cloud Storage** (buckets). `gcloud` es la CLI general para todos los demás servicios GCP (Compute, IAM, SQL, etc.). El examen puede mezclarlos.

⚠️ **[gcloud projects list — permisos incluidos]**
Lista proyectos donde el usuario tiene **owner, editor, browser o viewer**. No lista proyectos sin ningún rol asignado. El examen puede presentarlo como lista completa de todos los proyectos de GCP.

⚠️ **[GCP Scanner — credenciales soportadas]**
GCP Scanner soporta **OAuth2 Refresh Tokens con cloud-platform scope**, además de service account keys, metadata de VM y perfiles gcloud. El examen puede omitir el OAuth2 token.

⚠️ **[cloud_enum — flags para GCP-only]**
Para enumerar solo GCP, se usan `--disable-aws` y `--disable-azure`. Sin estos flags, el tool escanea los tres clouds. El examen puede presentar un flag `--only-gcp` que no existe.

⚠️ **[GCPBucketBrute — API usada]**
GCPBucketBrute usa la API **`TestIamPermissions`** de Google Storage para verificar permisos. La URL de verificación directa es `https://www.googleapis.com/storage/v1/b/BUCKETNAME/iam`. El examen puede confundirla con una petición al IMDS.

⚠️ **[GCP Privilege Escalation Scanner — ficheros de output]**
Genera tres ficheros: `all_org_folder_proj_sa_permissions.json`, **`privesc_methods.txt`** y **`setIamPolicy_methods.txt`**. El examen puede preguntar qué contiene cada fichero.

⚠️ **[Backdoor GCP — privilegios requeridos]**
La creación de backdoors IAM en GCP **solo es posible tras escalar privilegios a nivel administrativo**. No es una técnica inicial. El examen puede presentarla como accesible con permisos básicos.

⚠️ **[gcloud iam roles list — flag --show-deleted]**
Por defecto `gcloud iam roles list` no muestra roles eliminados. Se necesita `--show-deleted` para incluirlos. Roles eliminados pueden revelar permisos históricos explotables.

⚠️ **[GCP Scanner vs. gcp_service_enum — autenticación]**
`gcp_service_enum` requiere un **service account key file** (flag `-f`). GCP Scanner usa un **perfil gcloud** (flag `-g`). El examen puede invertir los métodos de autenticación.

---

## 3. Nemotécnicos

### gsutil vs. gcloud — regla simple
**"gsutil = Storage (S)"** → solo para buckets GCS.
**"gcloud = todo lo demás"** → IAM, Compute, SQL, organizaciones, proyectos.

### Comandos gsutil para buckets — orden de enumeración
**"ls → iam get → ls bucket → ls -r"**
→ `gsutil ls` (lista buckets) → `gsutil iam get gs://<b>` (permisos) → `gsutil ls gs://<b>` (contenido) → `gsutil ls -r` (recursivo)

### GCP Privilege Escalation Scanner — 2 scripts + 3 outputs
- Scripts: `enumerate_member_permissions.py` (permisos) → `check_for_privesc.py` (escalada)
- Outputs: **permissions.json** | **privesc_methods.txt** | **setIamPolicy_methods.txt**

### GCP Scanner flags
**"-o output -g gcloud_profile"** → `-o` = output file | `-g` = gcloud profile path

### Herramientas por objetivo GCP
- **Organizaciones/proyectos/IAM**: `gcloud`
- **Buckets**: `gsutil` + GCPBucketBrute
- **Multi-cloud buckets**: cloud_enum (`--disable-aws --disable-azure`)
- **Credenciales comprometidas → nivel acceso**: GCP Scanner
- **Servicios expuestos**: gcp_service_enum
- **Escalada IAM**: GCP Privilege Escalation Scanner
- **Grafos y attack paths**: (GCP) → Cartography (del chunk anterior)

---

## 4. Flashcards

**Q:** ¿Qué herramienta CLI de GCP se usa específicamente para operar con Cloud Storage buckets?
**A:** `gsutil`. Para todos los demás servicios GCP se usa `gcloud`.

---

**Q:** ¿Qué comando lista todos los buckets GCS del proyecto por defecto?
**A:** `gsutil ls`

---

**Q:** ¿Qué comando obtiene la IAM policy de un bucket GCS específico?
**A:** `gsutil iam get gs://<bucket_name>`

---

**Q:** ¿Qué permisos mínimos necesita un usuario para aparecer en `gcloud projects list`?
**A:** Owner, editor, browser o viewer en al menos un proyecto.

---

**Q:** ¿Qué hace `gcloud iam roles list --show-deleted`?
**A:** Incluye en los resultados los roles IAM eliminados, que pueden revelar permisos históricos explotables.

---

**Q:** ¿Qué tipos de credenciales puede usar GCP Scanner para enumerar recursos?
**A:** GCP VM instance metadata, user credentials de perfiles gcloud, OAuth2 Refresh Tokens con cloud-platform scope, y service account keys en JSON.

---

**Q:** ¿Qué comando de cloud_enum enumera solo recursos GCP sin escanear AWS ni Azure?
**A:** `cloud_enum.py -k <keyword> --disable-aws --disable-azure`

---

**Q:** ¿Qué API usa GCPBucketBrute para verificar los permisos de un bucket?
**A:** La API `TestIamPermissions` de Google Storage. Verifica enviando bucket name + lista de permisos.

---

**Q:** ¿Qué indica GCPBucketBrute cuando un bucket es vulnerable a privilege escalation?
**A:** Muestra un mensaje indicando que el bucket es vulnerable a privilege escalation y que se puede elevar hasta nivel administrator.

---

**Q:** ¿Cuáles son los tres ficheros que genera el GCP Privilege Escalation Scanner?
**A:** `all_org_folder_proj_sa_permissions.json` (permisos de todos los miembros), `privesc_methods.txt` (métodos de escalada), `setIamPolicy_methods.txt` (métodos con setIamPolicy explotables).

---

**Q:** ¿Qué dos scripts componen el flujo del GCP Privilege Escalation Scanner?
**A:** `enumerate_member_permissions.py` (enumeración de permisos) seguido de `check_for_privesc.py` (detección de escalada).

---

**Q:** ¿Cómo se crea un backdoor de IAM persistente en GCP?
**A:** 1) `gcloud iam roles create` con un YAML de permisos elevados, 2) `gcloud projects add-iam-policy-binding` asignando ese rol a una service account. Solo ejecutable con privilegios admin.

---

**Q:** ¿Qué herramienta GCP enumera servicios (Compute Engine, Cloud Storage) usando un fichero de clave de service account?
**A:** `gcp_service_enum`. Sintaxis: `gcp_enum_services.py -f <key_file> --output-file <output>`.

---

**Q:** ¿Qué recursos GCP soporta GCP Scanner además de GCE y GCS?
**A:** GKE, App Engine, Cloud SQL, BigQuery, Spanner, Pub/Sub, Cloud Functions, BigTable, CloudStore y KMS.

---

**Q:** ¿Para qué se usa GCPGoat y cuáles son sus tres escenarios principales?
**A:** Práctica de ataques GCP en infraestructura vulnerable por diseño. Escenarios: SSRF (obtener fuente de Cloud Function + volcar DB), Misconfigured Storage Bucket (acceso admin), Lateral Movement (credenciales de VM de bajo privilegio → Compute Instances de alto privilegio).

---

**Q:** ¿Cómo verifica GCPBucketBrute si un bucket GCS es público?
**A:** Hace una petición HTTP directa a `https://www.googleapis.com/storage/v1/b/BUCKETNAME/iam`. Si `allUsers` o `allAuthenticatedUsers` pueden leer la policy → respuesta válida; si no → access denied.

---

## 5. Confusión frecuente

**gsutil vs. gcloud — cuándo usar cada uno**
- `gsutil`: exclusivamente para Cloud Storage (listar buckets, obtener políticas IAM de buckets, listar objetos).
- `gcloud`: todo lo demás: Compute Engine, Cloud SQL, IAM roles, organizaciones, proyectos, service accounts.
- Criterio: si el comando involucra `gs://` o buckets → `gsutil`. Si involucra IAM, VMs, SQL → `gcloud`.

---

**GCP Scanner vs. gcp_service_enum — autenticación y alcance**
- GCP Scanner: usa perfil `gcloud` (`-g`); alcance muy amplio (12+ servicios); evalúa nivel de acceso de credenciales comprometidas.
- gcp_service_enum: usa fichero de service account key (`-f`); más específico para descubrimiento de servicios expuestos.
- Criterio: si hay credenciales comprometidas y se quiere evaluar su alcance → GCP Scanner. Si se tiene una key y se quieren descubrir servicios → gcp_service_enum.

---

**cloud_enum vs. GCPBucketBrute — qué descubren**
- cloud_enum: multi-cloud OSINT; encuentra **buckets públicos GCP, Firebase, App Engine, Cloud Functions**. Usa keyword, no credenciales.
- GCPBucketBrute: se centra en **permisos y escalada de privilegios** en buckets GCS específicos. Usa `TestIamPermissions` API.
- Criterio: descubrimiento de recursos públicos sin credenciales → cloud_enum. Análisis de permisos y escalada en buckets conocidos → GCPBucketBrute.

---

**GCP Privilege Escalation Scanner — dos scripts, dos propósitos**
- `enumerate_member_permissions.py`: solo enumera; no detecta escalada. Genera la base de datos de permisos.
- `check_for_privesc.py`: analiza los permisos enumerados y detecta las rutas de escalada. Sin el primer script, el segundo no tiene datos.
- Criterio: si la pregunta es sobre enumeración de permisos → primer script. Si es sobre detección de escalada → segundo script (que genera privesc_methods.txt).

---

**GCPGoat vs. AWSGoat vs. AzureGoat — plataforma**
- GCPGoat: vulnerabilidades en IAM, storage buckets, Cloud Functions, Compute Engine. Escenarios: SSRF, bucket misconfig, lateral movement.
- AWSGoat: vulnerabilidades en S3, Lambda, IAM, EC2 metadata. Escenarios: SQLi, SSRF, IAM escalada, file upload.
- AzureGoat: vulnerabilidades en App Functions, CosmosDB, Storage, Automation, Identities. Escenarios: IDOR, SSRF, security misconfig, privilege escalation.
- Criterio: el examen puede preguntar qué plataforma usa cada GoaT lab o qué escenarios practica.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un pentester de GCP quiere descubrir buckets GCS públicos, instancias de Firebase, App Engine y Cloud Functions de una organización objetivo sin usar credenciales. ¿Qué herramienta debe usar?

A) GCPBucketBrute  
B) GCPTokenBrute  
C) cloud_enum  
D) Prowler  

**Respuesta correcta: C**
**cloud_enum** es una herramienta de **multi-cloud OSINT** que descubre recursos públicos en GCP (buckets GCS, Firebase, App Engine, Cloud Functions) usando solo una keyword, sin necesitar credenciales. GCPBucketBrute analiza permisos en buckets conocidos, no descubre nuevos recursos públicos.

---

**P2.** Un auditor usa el **GCP Privilege Escalation Scanner** y ejecuta primero `enumerate_member_permissions.py` y luego `check_for_privesc.py`. ¿Qué detecta el segundo script?

A) Enumera los roles IAM de todos los miembros del proyecto  
B) Analiza los permisos enumerados y detecta rutas de escalada de privilegios, generando `privesc_methods.txt`  
C) Exporta las Service Account keys de todos los proyectos  
D) Verifica si hay buckets GCS con acceso anónimo  

**Respuesta correcta: B**
`check_for_privesc.py` analiza los permisos previamente enumerados por `enumerate_member_permissions.py` y detecta las **rutas de escalada de privilegios**, generando el fichero `privesc_methods.txt`. Sin ejecutar el primer script primero, el segundo no tiene datos para analizar.

---

**P3.** Un atacante compromete una GCP Service Account con permisos `storage.objects.list` y `storage.objects.get` en un bucket GCS. ¿Qué puede hacer con estos permisos?

A) Subir nuevos ficheros al bucket  
B) Eliminar objetos del bucket  
C) Listar y descargar todos los objetos del bucket  
D) Modificar las IAM policies del bucket  

**Respuesta correcta: C**
`storage.objects.list` permite **listar** todos los objetos del bucket y `storage.objects.get` permite **descargar** los objetos. Para subir se necesita `storage.objects.create`; para eliminar `storage.objects.delete`; para modificar IAM se necesita `storage.buckets.setIamPolicy`.

---

**P4.** Un pentester GCP quiere enumerar todas las Service Accounts de un proyecto y sus permisos. ¿Cuál es el comando gcloud correcto para listar Service Accounts?

A) `gcloud iam service-accounts list --project <project_id>`  
B) `gcloud auth list --project <project_id>`  
C) `gcloud compute instances list --filter="serviceAccounts"`  
D) `gcloud projects get-iam-policy <project_id>`  

**Respuesta correcta: A**
`gcloud iam service-accounts list --project <project_id>` lista todas las **Service Accounts** de un proyecto GCP. `gcloud auth list` muestra las credenciales activas del CLI. `gcloud projects get-iam-policy` muestra la política IAM del proyecto completo (incluyendo todas las entidades, no solo service accounts).

---

**P5.** Un analista descubre que una Service Account GCP tiene el rol `roles/editor` a nivel de proyecto. ¿Por qué es esto un problema de seguridad?

A) `roles/editor` no permite acceder a GCS buckets  
B) `roles/editor` es un rol muy amplio que viola el principio de least privilege y puede usarse para escalar a `roles/owner`  
C) `roles/editor` solo funciona en producción, no en dev  
D) `roles/editor` requiere MFA adicional para operaciones sensibles  

**Respuesta correcta: B**
`roles/editor` es un **rol muy amplio** que otorga permisos de lectura/escritura sobre casi todos los recursos del proyecto. Viola el principio de **least privilege** y puede ser el punto de partida para escalar a `roles/owner`. Las Service Accounts deben tener solo los permisos mínimos necesarios para su función específica.

---

**P6.** Durante un pentest GCP, el atacante accede a una instancia Compute Engine y consulta el endpoint de metadatos para obtener el token de la Service Account. ¿Qué URL debe consultar?

A) `http://169.254.169.254/latest/meta-data/iam/security-credentials/`  
B) `http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token`  
C) `http://169.254.169.254/metadata/identity/oauth2/token`  
D) `http://metadata.gcp.internal/v1/token`  

**Respuesta correcta: B**
En **GCP**, el endpoint de metadatos de instancia es `metadata.google.internal` y la ruta para obtener el token de la Service Account es `/computeMetadata/v1/instance/service-accounts/default/token`. Requiere el header `Metadata-Flavor: Google`. La IP `169.254.169.254` es AWS (IMDS). Azure usa `$IDENTITY_ENDPOINT`.

---

**P7.** Un cloud security engineer usa **GCPBucketBrute** contra buckets GCS conocidos. ¿Cuál es el objetivo principal de esta herramienta a diferencia de cloud_enum?

A) Descubrir nuevos buckets públicos a partir de una keyword  
B) Analizar permisos IAM y rutas de escalada de privilegios en buckets conocidos usando la API `TestIamPermissions`  
C) Hacer brute-force del nombre de los buckets GCS  
D) Enumerar Service Accounts con acceso a los buckets  

**Respuesta correcta: B**
**GCPBucketBrute** usa la API **`TestIamPermissions`** para analizar qué permisos tiene el atacante (o el público) sobre buckets GCS conocidos y si puede escalar privilegios. A diferencia de cloud_enum (que descubre recursos sin credenciales), GCPBucketBrute analiza profundidad de permisos en buckets ya identificados.

---

**P8.** Un CISO pregunta cuál es el equivalente GCP de los Roles IAM de AWS. ¿Cómo se llaman las entidades que definen permisos en GCP?

A) Azure AD Service Principals  
B) GCP IAM Roles con primitivos (Owner/Editor/Viewer) y predefinidos  
C) GCP Security Groups  
D) GCP Access Policies  

**Respuesta correcta: B**
En **GCP IAM**, los roles definen los permisos. Los **roles primitivos** son: `roles/owner`, `roles/editor`, `roles/viewer` (muy amplios, evitar en producción). Los **roles predefinidos** son más granulares y específicos por servicio. Los **roles personalizados** permiten el mínimo privilegio exacto. Son el equivalente a los IAM Roles de AWS.
