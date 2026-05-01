# Módulo 1 — Leyes y Estándares de Seguridad

## PCI DSS — 6 Requisitos de alto nivel

> Aplica a: merchants, processors, acquirers, issuers, service providers — cualquiera que **almacene, procese o transmita** datos de tarjeta

| Requisito | Puntos clave |
|---|---|
| **Build & Maintain Secure Network** | Firewall para datos de titulares · No usar defaults de vendor |
| **Protect Cardholder Data** | Proteger datos almacenados · Cifrar transmisión en redes públicas |
| **Vulnerability Management** | AV actualizado · Sistemas y apps seguros |
| **Strong Access Control** | Need-to-know · ID única por usuario · Restricción de acceso físico |
| **Monitor & Test Networks** | Monitorizar accesos · Testear sistemas regularmente |
| **Information Security Policy** | Política que cubra toda la organización |

> Incumplimiento → multas o cancelación de privilegios de procesamiento de tarjetas

---

## ISO/IEC Standards — número + foco

| Estándar | Foco |
|---|---|
| **27001:2022** | ISMS — framework completo de gestión de seguridad de la información |
| **27701:2019** | Extiende 27001 → privacidad — gestión de PII (PIMS) |
| **27002:2022** | Best practices y controles: acceso, criptografía, personal |
| **27005:2022** | Risk management de seguridad — soporta 27001 |
| **27018:2019** | PII en **entornos cloud públicos** |
| **27032:2023** | Relación Internet/web/red/ciberseguridad — stakeholders y roles |
| **27033-7:2023** | Seguridad en **virtualización de redes** |
| **27036-3:2023** | Seguridad en **cadena de suministro** (hardware, software, servicios) |
| **27040:2024** | Seguridad en **almacenamiento de datos** |

> **Regla de memorización:** 270**01** = base ISMS · 270**02** = controles · 270**05** = risk · 270**18** = cloud PII · 270**36** = supply chain · 270**40** = storage

---

## HIPAA — 6 Reglas

| Regla | Clave |
|---|---|
| **Electronic Transactions & Code Sets** | Estándar EDI para transacciones sanitarias electrónicas |
| **Privacy Rule** | Protege historial médico — derechos del paciente sobre su info |
| **Security Rule** | Salvaguardas administrativas, físicas y técnicas para **ePHI** |
| **Employer Identifier Standard** | Número nacional estándar por empleador |
| **NPI** | Identificador único de **10 dígitos** para proveedores sanitarios |
| **Enforcement Rule** | Sanciones civiles por incumplimiento |

---

## SOX — 11 Títulos (los más importantes)

| Título | Nombre | Clave |
|---|---|---|
| **I** | PCAOB | Supervisión independiente de auditoras |
| **II** | Auditor Independence | Evita conflictos de interés — prohíbe consultoría a clientes auditados |
| **III** | Corporate Responsibility | Ejecutivos **individualmente** responsables de informes financieros |
| **IV** | Enhanced Financial Disclosures | Controles internos sobre informes financieros |
| **VIII** | Corporate & Criminal Fraud Accountability | Penaliza destrucción/manipulación de registros — protege **whistleblowers** |
| **IX** | White-Collar Crime Penalty Enhancement | Aumenta penas por delitos financieros |
| **XI** | Corporate Fraud Accountability | Fraude como delito — congela transacciones inusuales |

---

## DMCA — 5 Títulos

| Título | Clave |
|---|---|
| **I** | Implementación tratados WIPO — prohíbe eludir protecciones de copyright |
| **II** | Limitación de responsabilidad de ISPs por infracción de usuarios |
| **III** | Permite copias de software para mantenimiento/reparación |
| **IV** | Disposiciones varias: educación a distancia, bibliotecas, webcasting |
| **V** | Protección de diseños de cascos de embarcaciones (VHDPA) |

---

## GDPR — 7 Principios (Art. 5.1-2)

> Aplica a **cualquier organización globalmente** que procese datos de ciudadanos de la UE · En vigor desde **25 mayo 2018**

1. **Lawfulness, fairness, transparency** — procesamiento legal, justo y transparente
2. **Purpose limitation** — solo para los fines especificados
3. **Data minimization** — solo los datos necesarios
4. **Accuracy** — datos exactos y actualizados
5. **Storage limitation** — solo el tiempo necesario
6. **Integrity & confidentiality** — seguridad apropiada (ej. cifrado)
7. **Accountability** — el **data controller** demuestra cumplimiento

---

## FISMA — Framework Federal USA

- Categorización de sistemas por impacto en misión
- Requisitos mínimos de seguridad
- Selección, evaluación y autorización de controles
- Obligatorio para **agencias federales** — incluye sistemas gestionados por terceros

---

## Cyber Laws por País

| País | Leyes destacadas |
|---|---|
| **USA** | Computer Fraud and Abuse Act · CCPA · FOIA · Privacy Act 1974 · FISMA · Electronic Communications Privacy Act |
| **Australia** | Cybercrime Act 2001 · Copyright Act 1968 · Patents Act 1990 |
| **UK** | Computer Misuse Act 1990 · Investigatory Powers Act 2016 · NIS Regulations 2018 |
| **China** | Trademark Law of the PRC (2001) |
| **India** | Information Technology Act · Patents Amendment Act 1999 · Copyright Act 1957 |
| **Germany** | §202a Data Espionage · §303a Alteration of Data · §303b Computer Sabotage |
| **Italy** | Penal Code Article 615 ter |
| **Japan** | Trademark Law No. 127 of 1959 |
| **Canada** | PIPEDA · Canadian Criminal Code §342.1 · Copyright & Trademarks Acts 1985 |
| **Singapore** | Computer Misuse Act |
| **South Africa** | Trademarks Act 194 of 1993 · Copyright Act 1978 |
| **South Korea** | Copyright Act (hasta 2023) · Industrial Design Protection Act |
| **Belgium** | Copyright Law 1994 · Computer Hacking law |
| **Brazil** | LGPD — Brazilian General Data Protection Law |
| **Hong Kong** | Article 139 of the Basic Law |
| **Philippines** | Republic Act No. 10175 |

> **Para el examen:** más preguntados — USA, UK, India (IT Act), Canadá (PIPEDA), Brasil (LGPD), Alemania (§202a/303a/303b)
