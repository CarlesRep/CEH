# M02 — Social Networking, LinkedIn y Email Harvesting

## RRSS Footprinting vs Social Engineering

> ⚠️ **Distinción crítica para el examen:**
> - **RRSS Footprinting** = recopila info *disponible públicamente* en redes sociales
> - **Social Engineering** = *engaña* activamente a personas para que revelen información

---

## Información por actividad en RRSS

### Usuarios individuales

| Actividad | Info obtenida |
|---|---|
| Mantener perfil | Contacto, ubicación, info personal |
| Conectar/chatear | Lista de amigos, info de contactos |
| Compartir fotos/vídeos | Identidad de familiares, intereses |
| Jugar/grupos | Intereses del target |
| Crear eventos | Actividades futuras |

### Organizaciones

| Actividad | Info obtenida |
|---|---|
| Encuestas | Estrategias de negocio |
| Promocionar productos | Perfil de producto |
| Soporte | Vectores de social engineering |
| Reclutamiento | Plataformas y tecnologías usadas |

**Táctica de fake profile:** crear cuenta → solicitud de amistad → si acepta → acceso a páginas restringidas

---

## LinkedIn Footprinting + theHarvester

```bash
# Enumerar empleados de empresa en LinkedIn
theHarvester -d microsoft -l 200 -b linkedin

# Extraer emails de un dominio (Baidu como fuente)
theHarvester -d microsoft.com -l 200 -b baidu
```

| Parámetro | Función |
|---|---|
| `-d` | Dominio o nombre de empresa |
| `-l` | Límite de resultados |
| `-b` | Fuente: linkedin, google, bing, baidu… |

> ⚠️ **theHarvester LinkedIn vs Email:**
> - LinkedIn mode = enumera **empleados y cargos**
> - Email mode = extrae **direcciones de correo** del dominio

---

## Herramientas de Análisis de RRSS

| Herramienta | Función clave |
|---|---|
| **BuzzSumo** | Contenido más compartido por tema/autor/dominio — Twitter, Facebook, LinkedIn, Pinterest |
| **Sherlock** | Busca **username** en múltiples RRSS → devuelve URL completa por plataforma |
| **Social Searcher** | Búsqueda en tiempo real + analytics — URLs, posts, info personal |
| **Google Trends** | Tendencias de búsqueda del target |
| **Hashatit** | Búsqueda por hashtags |
| **Ubersuggest** | Keywords y presencia en RRSS |

### ⚠️ Distinciones clave

| Par | Diferencia |
|---|---|
| **Sherlock vs Social Searcher** | Sherlock = busca por **username** · Social Searcher = busca **contenido** con analytics |
| **BuzzSumo vs Sherlock** | BuzzSumo = contenido más compartido del dominio · Sherlock = localiza usuario por username |

---

## Email Harvesting

**Objetivo:** obtener emails corporativos → vector de ataque para fases posteriores

**Uso ofensivo:** social engineering · brute force · spear phishing

**Herramientas:** theHarvester · Email Spider

**Fuentes:** Google · Bing · Yahoo · Baidu

### 📝 Pregunta típica CEH

> *"Un atacante quiere obtener emails corporativos de microsoft.com usando Baidu. ¿Qué comando usa?"*
> → `theHarvester -d microsoft.com -l 200 -b baidu`
