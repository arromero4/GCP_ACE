# ACE 2026 · Lección 17

## Control de dominio 1 y práctica de recuperación

**Fecha:** 13 de octubre de 2026  
**Cobertura de la guía oficial:** 1.1 *Setting up cloud projects and accounts* y 1.2 *Managing billing configuration*  
**Práctica exigida por el plan:** evaluación de 25 preguntas y repetición de **exactamente dos** tareas débiles  
**Caso transversal:** SiteOps Tracker, proyecto ficticio de portafolio con React + TypeScript, Node.js + TypeScript, PostgreSQL y servicios de Google Cloud  
**Tiempo sugerido:** 15 minutos de activación, 50 minutos de control, 35–60 minutos por cada una de las dos tareas de recuperación y 15 minutos de corrección.

> Este control mide el desempeño de hoy; recibir o leer la lección no demuestra dominio. La meta personal del plan es al menos 80 %, es decir, 20 de 25 respuestas correctas. No es un umbral oficial de Google.

---

## Objetivo

Al terminar podrás:

1. Resolver 25 escenarios originales en inglés sobre la configuración del entorno de una solución en Google Cloud.
2. Justificar decisiones sobre jerarquía, Organization Policy, IAM, identidad, APIs, cuotas, geografía, red, inventario, observabilidad y facturación.
3. Identificar tus dos grupos más débiles con evidencia, sin asumir de antemano cuáles son.
4. Volver a ejecutar exactamente dos tareas de recuperación con Google Cloud console y Google Cloud CLI, o completar sus alternativas conceptuales si no tienes cuenta o crédito.
5. Registrar errores, causa raíz, regla de decisión y siguiente comprobación.

## Prerrequisitos explicados desde cero

No necesitas memorizar comandos ni tener recursos de pago. Sí necesitas comprender estas bases:

- **Google Cloud console:** interfaz web para administrar recursos.
- **Cloud Shell:** terminal temporal autenticada desde la consola; incluye Google Cloud CLI.
- **Google Cloud CLI (`gcloud`):** conjunto de comandos para consultar o administrar Google Cloud.
- **Proyecto (*project*):** frontera frecuente de recursos, APIs, cuotas, IAM y atribución de costos.
- **Principal:** identidad a la que se puede conceder acceso: usuario, grupo, cuenta de servicio o identidad federada.
- **Rol (*role*):** colección de permisos. IAM une principal + rol + recurso.
- **Cuenta de facturación (*Cloud Billing account*):** instrumento que paga el consumo de proyectos vinculados; no es un proyecto.
- **Región y zona:** una región es un área geográfica; una zona es un dominio de implementación dentro de una región.

Para la práctica con cuenta necesitas permiso de lectura en un proyecto de laboratorio. Algunas consultas requieren permisos adicionales sobre Organization Policy, Cloud Asset Inventory o Cloud Billing. Un error `PERMISSION_DENIED` es evidencia útil y no una invitación a pedir `Owner`.

## Alineación con la guía oficial

La guía oficial asigna aproximadamente 20 % del examen al dominio 1. Este control cubre:

- jerarquía de recursos y Organization Policy;
- roles IAM, usuarios y grupos de Cloud Identity;
- habilitación de APIs, cuotas y aumentos;
- organización independiente, red y disponibilidad geográfica;
- Google Cloud Observability, Cloud Asset Inventory y Gemini Cloud Assist;
- Workforce Identity Federation;
- cuentas de facturación, vínculo de proyectos, presupuestos, alertas y exportación.

Los porcentajes de la guía son aproximados. Las 25 preguntas de hoy están equilibradas en cinco grupos de cinco para diagnosticar debilidades; esa distribución es una herramienta de estudio, no la fórmula de puntuación del examen.

---

## Activación: repaso espaciado antes del control

Responde sin consultar apuntes. Después compara con las pistas. Dedica como máximo 90 segundos por bloque.

### Día hábil anterior · 12/10 · Lección 16

1. ¿Qué pregunta separa Organization Policy de IAM?
2. ¿Qué tres comprobaciones harías antes de culpar a la red por un fallo de API?
3. ¿Por qué un presupuesto de alertas no equivale a un freno automático del gasto?

**Pistas:** Organization Policy restringe *qué configuraciones pueden existir*; IAM decide *quién puede hacer qué*. Para una API, verifica proyecto activo, servicio habilitado y permiso/cuota antes de la conectividad. Un presupuesto de tipo *alerts-only* notifica; no impide por sí solo el uso o la facturación.

### Días hábiles recientes · 09/10 y 08/10 · Lecciones 15 y 14

1. Ordena el diagnóstico del laboratorio integral: identidad, proyecto, API, IAM, cuota, ubicación, recurso.
2. ¿Qué federación corresponde a una persona externa y cuál a un proceso externo?
3. ¿Qué riesgo reduce federar identidades en lugar de distribuir credenciales permanentes?

**Pistas:** confirma primero contexto e identidad y luego avanza hacia el recurso. **Workforce Identity Federation** es para personas externas; **Workload Identity Federation** es para cargas externas. La federación evita depender de claves de larga duración y permite credenciales de corta duración.

### Hace 7 días · 06/10 · Lección 12

1. ¿Una VPC es global o regional?
2. ¿Una subred es global o regional?
3. ¿Dos zonas de la misma región toleran la pérdida de una zona? ¿Y la pérdida de toda la región?

**Pistas:** la VPC es global; las subredes son regionales. Dos zonas reducen el riesgo zonal, pero no sustituyen un diseño multirregional cuando el requisito es sobrevivir a la caída de una región.

### Hace 21 días · 22/09 · Lección 2

1. ¿Qué dos comandos revelan la cuenta activa y el proyecto configurado?
2. ¿Por qué debes verificar el selector de proyecto antes de ejecutar un comando?
3. ¿Qué dato nunca debes pegar en una ficha de estudio?

**Pistas:** `gcloud auth list --filter=status:ACTIVE --format="value(account)"` y `gcloud config list project`. El proyecto incorrecto puede causar cambios o costos fuera del laboratorio. Nunca guardes contraseñas, tokens, claves privadas ni secretos.

---

## Modelo mental paso a paso

Imagina que SiteOps Tracker administra auditorías de sedes como si Google Cloud fuera una ciudad:

1. **Organization** es el ayuntamiento: es la raíz de gobierno.
2. **Folders** son distritos: agrupan proyectos y permiten heredar políticas y acceso.
3. **Projects** son edificios con dirección propia: contienen recursos, APIs, cuotas e IAM.
4. **Resources** son las habitaciones y equipos: redes, bases de datos, servicios y máquinas.
5. **Organization Policy** es el reglamento de construcción: puede prohibir ubicaciones o configuraciones, aunque alguien tenga una llave.
6. **IAM** es el sistema de llaves: decide quién puede abrir una puerta y qué acción puede realizar.
7. **APIs** son oficinas que deben estar abiertas para atender solicitudes.
8. **Quotas** son límites de capacidad del servicio; una cuota no es un presupuesto.
9. **VPC y subredes** son la red vial: la VPC conecta globalmente; cada subred ocupa una región.
10. **Cloud Asset Inventory** es el padrón de bienes; **Monitoring** y **Logging** son el tablero y la bitácora operativa.
11. **Cloud Billing account** es la cuenta que paga los servicios del edificio; un presupuesto de alertas es el aviso de consumo.

### Ejemplo integrado de SiteOps Tracker

La empresa ficticia separa `development` y `production`. Los hallazgos de auditoría viven en PostgreSQL; la API Node.js se despliega en Google Cloud y el frontend React consume esa API.

- Una carpeta por entorno permite heredar controles distintos.
- Proyectos separados reducen el radio de impacto y distinguen costos/cuotas.
- Un grupo de Cloud Identity recibe un rol predefinido mínimo, en vez de dar permisos a cada persona.
- Una política organizacional restringe ubicaciones aprobadas; IAM no reemplaza esa restricción.
- Cloud Asset Inventory ayuda a descubrir recursos desplegados; Monitoring y Logging muestran su comportamiento.
- La exportación de facturación a BigQuery permite analizar costo por proyecto, servicio o etiqueta.

## Decisiones que suelen confundirse

| Necesidad | Elección principal | Por qué | Por qué no las alternativas |
|---|---|---|---|
| Agrupar y heredar gobierno | Organization → folder → project | La jerarquía transmite políticas y bindings heredados | Las labels clasifican y ayudan con costos, pero no crean una frontera de herencia |
| Impedir una configuración | Organization Policy | Aplica restricciones a configuraciones de recursos | IAM controla acciones de identidades; no expresa por sí solo una ubicación permitida |
| Dar acceso a un equipo | Grupo de Cloud Identity + rol mínimo | Simplifica altas/bajas y conserva consistencia | Grants individuales escalan mal; `Owner` es excesivo |
| Personas de un IdP externo | Workforce Identity Federation | Está diseñada para usuarios humanos externos | Workload Identity Federation es para software/cargas, no para personas |
| CI/CD externo sin claves | Workload Identity Federation | Entrega credenciales de corta duración a una carga | Una clave JSON de cuenta de servicio es un secreto de larga duración |
| Descubrir recursos a escala | Cloud Asset Inventory | Busca inventario y metadatos en proyecto, carpeta u organización | Monitoring mide señales; la API específica del servicio ofrece estado operativo detallado |
| Latencia, errores y tendencias | Monitoring + Logging | Métricas, alertas y registros operativos | Cloud Asset Inventory no es un sistema de telemetría en tiempo real |
| Avisar sobre gasto | Budget alerts | Compara costo real/pronosticado contra umbrales | Una cuota limita consumo técnico; una alerta de presupuesto no es un límite automático |
| Analizar costo con SQL | Billing export to BigQuery | Conserva datos exportados para consulta y combinación | El informe incorporado es útil, pero no sustituye análisis SQL repetible |
| Diseñar red por ubicación | VPC global + subred regional | Separa conectividad global de rangos regionales | Una zona no define una subred; una VPC no se limita a una sola región |

## Glosario bilingüe

| English | Español | Regla breve |
|---|---|---|
| resource hierarchy | jerarquía de recursos | Organization, folders, projects y resources |
| organization policy | política de organización | Restringe configuraciones permitidas |
| allow policy / IAM policy | política de permisos IAM | Vincula principals con roles |
| inherited binding | binding heredado | Acceso recibido desde un ancestro |
| principal | principal / identidad | Usuario, grupo, service account o identidad federada |
| least privilege | mínimo privilegio | Solo permisos necesarios, alcance mínimo |
| enabled API | API habilitada | Servicio disponible para consumo en el proyecto |
| quota | cuota | Límite ajustable de consumo en muchos casos |
| system limit | límite del sistema | Límite fijo que normalmente no puede aumentarse |
| region | región | Área geográfica independiente |
| zone | zona | Dominio de despliegue dentro de una región |
| VPC network | red VPC | Red virtual global |
| subnet | subred | Rango IP regional |
| asset inventory | inventario de activos | Catálogo consultable de recursos y políticas |
| observability | observabilidad | Métricas, logs, trazas y señales de salud |
| billing account | cuenta de facturación | Paga proyectos vinculados |
| alerts-only budget | presupuesto solo de alertas | Notifica; no bloquea automáticamente el gasto |
| billing export | exportación de facturación | Datos de costos enviados a BigQuery |
| workforce | fuerza laboral | Personas que acceden a recursos |
| workload | carga de trabajo | Software o proceso que accede a recursos |

---

## Control de dominio 1 · 25 preguntas originales

### Instrucciones

1. Programa 50 minutos y cierra la sección de soluciones.
2. Todas las preguntas están en inglés. Lee primero el requisito, luego el escenario.
3. Si dice **Choose two**, marca exactamente dos respuestas; la pregunta solo cuenta si ambas son correctas y no marcas otra.
4. Registra respuesta, confianza (`H`, `M`, `L`) y tiempo aproximado.
5. No uses consola, documentación ni asistente durante el primer intento.

### Hoja de respuestas

| Q | Answer | Confidence | Time | Q | Answer | Confidence | Time |
|---:|:---:|:---:|---:|---:|:---:|:---:|---:|
| 1 |  |  |  | 14 |  |  |  |
| 2 |  |  |  | 15 |  |  |  |
| 3 |  |  |  | 16 |  |  |  |
| 4 |  |  |  | 17 |  |  |  |
| 5 |  |  |  | 18 |  |  |  |
| 6 |  |  |  | 19 |  |  |  |
| 7 |  |  |  | 20 |  |  |  |
| 8 |  |  |  | 21 |  |  |  |
| 9 |  |  |  | 22 |  |  |  |
| 10 |  |  |  | 23 |  |  |  |
| 11 |  |  |  | 24 |  |  |  |
| 12 |  |  |  | 25 |  |  |  |
| 13 |  |  |  |  |  |  |  |

### Question 1

SiteOps Tracker needs separate development and production environments. Production must inherit stricter policies, and each environment needs separate quotas and cost attribution. What is the best resource hierarchy?

A. One project with labels for development and production  
B. One organization, environment folders, and separate projects under each folder  
C. Two unrelated billing accounts with all resources in one project  
D. One VPC network for each environment, without separate projects

### Question 2

The security team must prevent new resources from being created outside approved locations, even by project administrators. Which control should be used?

A. A budget threshold  
B. An IAM role binding  
C. An Organization Policy constraint applied at the appropriate hierarchy level  
D. A Cloud Monitoring alert

### Question 3

A user has the Compute Admin role on a project, but an inherited organization policy rejects a VM configuration. What should the cloud engineer conclude?

A. The IAM role automatically overrides the organization policy  
B. The user needs the Owner role  
C. IAM authorization and organization policy evaluation are separate; the restricted configuration remains blocked  
D. Enabling the Compute Engine API removes the inherited restriction

### Question 4 — Choose two

A project is about to be moved from a development folder to a production folder. Which two effects should be assessed before the move?

A. Changes to inherited IAM bindings  
B. Changes to effective organization policies  
C. Automatic conversion of all zonal resources to regional resources  
D. Automatic replacement of the linked billing account  
E. Automatic deletion of project labels

### Question 5

SiteOps Tracker needs to group costs by fictitious site code, but site code must not change IAM or policy inheritance. What should the engineer use?

A. Labels on supported resources  
B. A separate organization for every site  
C. Basic Owner roles named after each site  
D. A new zone for every site

### Question 6

Twenty internal auditors need read-only access to selected SiteOps Tracker projects, and team membership changes frequently. What is the most maintainable least-privilege approach?

A. Grant Owner to every auditor  
B. Add each auditor directly to every project  
C. Put auditors in a Cloud Identity group and grant the group an appropriate predefined viewer role at the narrowest useful scope  
D. Share a service account key with the team

### Question 7

External partner employees must use their existing corporate identities to access Google Cloud resources without receiving Google-managed user accounts. Which capability is designed for this case?

A. Workforce Identity Federation  
B. Workload Identity Federation  
C. Compute Engine service account keys  
D. VPC Network Peering

### Question 8 — Choose two

An external CI system needs short-lived access to deploy the SiteOps Tracker backend. Which two choices best meet the requirement?

A. Configure Workload Identity Federation  
B. Grant only the required deployment permissions  
C. Download and store a long-lived service account key in the CI system  
D. Configure Workforce Identity Federation for the CI process  
E. Grant the basic Owner role so deployment cannot fail

### Question 9

An engineer removes a direct project-level role from a user, but the user still has the same access. What should be checked next?

A. Inherited IAM bindings from parent folders or the organization  
B. The project's budget thresholds  
C. The subnet's primary IP range  
D. The enabled API list only

### Question 10

A company uses Cloud Identity for its domain but has no Google Cloud organization resource. It now needs centralized hierarchy and organization-level policies. What should the administrator set up?

A. A standalone organization associated with the existing identity setup  
B. A second default VPC in every project  
C. A billing export dataset  
D. A regional managed instance group

### Question 11

A developer has the required IAM permission, but a request fails with a message that the service API has not been used or is disabled in the project. What is the most direct next action?

A. Enable the required API in the correct project, if authorized  
B. Increase the CPU quota  
C. Create a new billing account  
D. Grant the Owner role

### Question 12

A deployment fails because the requested regional CPU count exceeds the project's quota. What should the engineer do first?

A. Review current quota usage and request an increase or reduce the request  
B. Create more IAM groups  
C. Disable Cloud Monitoring  
D. Move the billing account to another organization

### Question 13

The selected Google Cloud product is not offered in the region required by the proposed architecture. What is the best response?

A. Assume all products exist in every region once billing is enabled  
B. Verify the product's location availability and select a supported location or an alternative design  
C. Add the Editor role to the deployment account  
D. Create a budget alert for that region

### Question 14

Which statement about VPC networks and subnetworks in Google Cloud is correct?

A. Both VPC networks and subnetworks are zonal  
B. VPC networks are regional and subnetworks are global  
C. VPC networks are global and subnetworks are regional  
D. Both VPC networks and subnetworks are tied to a billing account rather than a project

### Question 15 — Choose two

The SiteOps Tracker API must continue running if one zone fails, but surviving a whole regional outage is not required. Which two design decisions directly support the requirement?

A. Deploy instances across at least two zones in the same region  
B. Use a health-aware regional distribution mechanism appropriate to the compute platform  
C. Place every instance in one zone with a larger machine type  
D. Create a budget alert at 50 percent  
E. Use labels instead of health checks

### Question 16

An auditor needs to search for resources across many projects in an organization and view common resource metadata from one inventory service. Which service is the best fit?

A. Cloud Asset Inventory  
B. Cloud Monitoring uptime checks  
C. Cloud Billing budgets  
D. Cloud DNS

### Question 17

The operations team needs latency, request count, and error-rate trends for the SiteOps Tracker API. What should it use primarily?

A. Cloud Monitoring metrics and dashboards  
B. Cloud Asset Inventory resource search  
C. Organization Policy  
D. Cloud Identity groups

### Question 18

The team must inspect individual failed application requests and their structured context. Which capability is the best starting point?

A. Cloud Logging  
B. A billing export  
C. A quota increase request  
D. A folder-level IAM binding

### Question 19

Gemini Cloud Assist proposes a change to a SiteOps Tracker resource. What is the safest operational response?

A. Apply every suggestion automatically because it is generated by Google Cloud  
B. Validate the recommendation against requirements, permissions, policy, cost, and rollback before applying it  
C. Grant Owner to Gemini Cloud Assist  
D. Disable audit logging before making the change

### Question 20

An engineer needs the exact current operational state and service-specific configuration of one database instance. Which source should be treated as the most direct authority?

A. The database service's console or API  
B. A budget alert email  
C. A Cloud Identity group description  
D. An organization folder name

### Question 21 — Choose two

An administrator must link an existing project to a Cloud Billing account. Which two authorization scopes generally need to be considered?

A. Permission to manage billing association on the project  
B. Permission to associate projects with the target billing account  
C. Permission to edit every VPC firewall rule  
D. Permission to administer Cloud DNS in the project  
E. Permission to create organization policies at the organization root

### Question 22

SiteOps Tracker has an **alerts-only** monthly budget. Actual spend crosses 100 percent of the budget. What happens by default?

A. All project resources are immediately deleted  
B. Billing is automatically disabled for every linked project  
C. Configured notifications are sent, but usage and charges can continue  
D. All quotas are set to zero

### Question 23

The finance team wants repeatable SQL analysis of detailed Google Cloud costs and usage. Which configuration best supports this requirement?

A. Export Cloud Billing data to BigQuery  
B. Enable VPC Flow Logs only  
C. Store screenshots of the Billing report  
D. Increase project quotas

### Question 24

An engineer disables an API after a lab and assumes all associated costs stop. Why is this assumption unsafe?

A. Disabling an API doesn't necessarily delete billable resources that were already created  
B. Disabled APIs always double the cost of existing resources  
C. APIs and resources are the same object  
D. Billing accounts cannot be linked to projects with disabled APIs

### Question 25 — Choose two

The SiteOps Tracker team needs to allocate cost by environment and fictitious site and then run monthly SQL reports. Which two elements form the best foundation?

A. Consistent project/resource labels where supported  
B. Cloud Billing export to BigQuery  
C. One shared Owner account for all environments  
D. A separate zone for every cost center  
E. Disabling Cloud Logging

---

## Corrección y selección de las dos tareas débiles

No mires solo la puntuación total. Clasifica cada pregunta así:

| Grupo | Preguntas | Tema | Aciertos / 5 |
|---|---|---|---:|
| A | 1–5 | Jerarquía y Organization Policy |  |
| B | 6–10 | IAM e identidad |  |
| C | 11–15 | APIs, cuotas, red y geografía |  |
| D | 16–20 | Observabilidad, inventario y asistencia |  |
| E | 21–25 | Facturación y costos |  |

### Regla objetiva

1. Elige los **dos grupos con menor puntuación**.
2. Si hay empate, elige primero el grupo con más respuestas de confianza baja.
3. Si continúa el empate, elige el grupo con mayor tiempo total.
4. Ejecuta exactamente las dos tareas con la misma letra. No hagas las cinco hoy.

Anota la selección:

- **Tarea débil 1:** ___ · Evidencia: ___
- **Tarea débil 2:** ___ · Evidencia: ___

---

## Práctica de recuperación: ejecuta exactamente dos tareas

### Seguridad y preparación común

Los comandos siguientes son de consulta. Fueron comprobados en la referencia oficial vigente consultada el 13/10/2026. Usa un proyecto de laboratorio; no pegues secretos.

En Cloud Shell:

```bash
export PROJECT_ID="your-lab-project-id"

gcloud auth list --filter=status:ACTIVE --format="value(account)"
gcloud config list project
gcloud projects describe "$PROJECT_ID"
```

**Punto de control:** la cuenta y el proyecto deben ser los esperados. Si no lo son, detente; no cambies una cuenta o un proyecto ajeno.

### Tarea A · Jerarquía y Organization Policy

**Consola**

1. Abre el selector de proyecto y elige el proyecto de laboratorio.
2. Ve a **IAM & Admin → Manage resources** y localiza su parent.
3. Ve a **IAM & Admin → Organization Policies**.
4. Elige una restricción visible y registra si el valor es local o heredado. No edites nada.

**CLI**

```bash
gcloud projects get-ancestors "$PROJECT_ID"
gcloud org-policies list --project="$PROJECT_ID"
```

**Resultado esperado:** puedes dibujar `organization/folder → project → resource` e identificar que una política efectiva puede venir de un ancestro. Una lista vacía de políticas configuradas directamente no demuestra que no existan reglas heredadas.

**Alternativa sin cuenta:** dibuja una organización ficticia con carpetas `development` y `production`; coloca dos proyectos debajo. Añade una restricción de ubicación en `production` y explica por qué dar `Owner` en el proyecto no debe anularla.

### Tarea B · IAM e identidad

**Consola**

1. Ve a **IAM & Admin → IAM** en el proyecto de laboratorio.
2. Identifica un principal, su rol y si el acceso aparece como heredado.
3. Localiza la vista de grupos/identidades solo si tu laboratorio lo permite; no agregues miembros.
4. Escribe una recomendación de mínimo privilegio sin aplicarla.

**CLI**

```bash
gcloud projects get-iam-policy "$PROJECT_ID" --format=yaml
```

**Resultado esperado:** distingues principal, rol, binding, condición y herencia. Puedes explicar por qué un grupo es preferible a veinte grants individuales y por qué Workforce y Workload Identity Federation resuelven actores diferentes.

**Alternativa sin cuenta:** crea una matriz para tres perfiles ficticios: auditor humano interno, auditor humano externo y proceso CI externo. Asigna respectivamente grupo de Cloud Identity, Workforce Identity Federation y Workload Identity Federation; anota alcance y rol mínimo.

### Tarea C · APIs, cuotas, red y geografía

**Consola**

1. Ve a **APIs & Services → Enabled APIs & services** y registra una API habilitada.
2. Ve a **IAM & Admin → Quotas & System Limits** y localiza una cuota regional. No solicites aumento.
3. Ve a **VPC network → VPC networks** e identifica red, subred y región.
4. Comprueba en la página oficial de ubicaciones del producto si la región propuesta está soportada.

**CLI**

```bash
gcloud services list --enabled --project="$PROJECT_ID"
gcloud compute networks list --project="$PROJECT_ID"
gcloud compute networks subnets list --project="$PROJECT_ID"
```

**Resultado esperado:** separas cuatro causas: API deshabilitada, permiso insuficiente, cuota agotada y producto no disponible en la ubicación. También reconoces VPC global y subred regional.

**Alternativa sin cuenta:** recibe estos cuatro mensajes ficticios y asigna la primera verificación: `SERVICE_DISABLED`, `PERMISSION_DENIED`, `QUOTA_EXCEEDED`, `unsupported region`. Explica por qué ninguna de las cuatro se corrige automáticamente con `Owner`.

### Tarea D · Cloud Asset Inventory y observabilidad

**Consola**

1. Busca **Asset Inventory** y abre la búsqueda de recursos.
2. Limita el alcance al proyecto de laboratorio y observa tipo, nombre y ubicación.
3. Abre **Monitoring → Dashboards** y **Logging → Logs Explorer**.
4. Escribe qué pregunta responde cada vista: “¿qué existe?”, “¿cómo se comporta?” o “¿qué ocurrió?”. No crees alertas.

**CLI**

```bash
gcloud asset search-all-resources \
  --scope="projects/${PROJECT_ID}" \
  --limit=20 \
  --format="table(assetType,displayName,location)"
```

**Resultado esperado:** Cloud Asset Inventory devuelve recursos accesibles dentro del alcance; Monitoring ofrece métricas y Logging eventos. Un resultado vacío puede significar que no hay recursos visibles o que falta permiso.

**Alternativa sin cuenta:** para SiteOps Tracker, clasifica estas preguntas: “¿Qué buckets existen?”, “¿Cuál es la latencia p95?”, “¿Por qué falló la solicitud 8f2?” y “¿Cuál es la configuración operativa exacta de la instancia SQL?”. Respuestas: Asset Inventory, Monitoring, Logging y la API/consola de Cloud SQL.

### Tarea E · Facturación y costos

**Consola**

1. Ve a **Billing** desde el proyecto de laboratorio.
2. Registra si el proyecto está vinculado a una cuenta de facturación, sin copiar datos sensibles.
3. Revisa **Budgets & alerts** y **Billing export** solo si tienes acceso.
4. Explica la diferencia entre notificar gasto y exportar datos para análisis.

**CLI**

```bash
gcloud billing projects describe "$PROJECT_ID"
```

Si el comando muestra un identificador de cuenta y tienes permiso para listar presupuestos, introdúcelo manualmente:

```bash
export BILLING_ACCOUNT_ID="000000-000000-000000"
gcloud billing budgets list --billing-account="$BILLING_ACCOUNT_ID"
```

**Resultado esperado:** identificas el vínculo proyecto-cuenta. Una lista vacía puede significar que no hay presupuestos visibles; `PERMISSION_DENIED` significa que debes revisar acceso, no pedir `Owner` al proyecto por reflejo.

**Alternativa sin cuenta:** dibuja una cuenta de facturación que paga dos proyectos. Añade un presupuesto de alertas a 50/90/100 % y una exportación a BigQuery. Explica qué componente notifica, cuál permite SQL y cuál no detiene automáticamente recursos.

---

## Resultado esperado de la práctica

Completa esta evidencia para cada una de las dos tareas elegidas:

| Evidencia | Tarea 1 | Tarea 2 |
|---|---|---|
| Grupo y puntuación |  |  |
| Ruta de consola usada |  |  |
| Comando ejecutado o simulación |  |  |
| Resultado observado |  |  |
| Decisión corregida |  |  |
| Duda pendiente |  |  |
| Próxima recuperación |  |  |

La práctica está completa cuando puedes explicar en voz alta la decisión corregida y predecir el resultado antes de repetir la consulta. Ejecutar un comando sin interpretarlo no basta.

## Solución de problemas

| Síntoma | Causa probable | Comprobación segura | Respuesta adecuada |
|---|---|---|---|
| `PERMISSION_DENIED` | Falta un permiso en el alcance consultado | Identifica comando, recurso y principal activo | Pide el rol mínimo al administrador; no `Owner` por defecto |
| `SERVICE_DISABLED` o API no usada | API deshabilitada en el proyecto objetivo | Verifica proyecto y lista de APIs habilitadas | Habilita solo si la práctica lo autoriza; hoy basta documentar |
| `QUOTA_EXCEEDED` | Cuota agotada o solicitud sobredimensionada | Consulta uso, límite, región y recurso | Reduce solicitud o tramita aumento con justificación |
| La lista está vacía | Sin recursos/políticas visibles o filtro/alcance equivocado | Revisa proyecto, scope, filtros y permisos | Corrige el contexto; no inventes que “no existe nada” |
| No aparece facturación | Proyecto sin vínculo o acceso insuficiente | Ejecuta `gcloud billing projects describe` | Distingue ausencia de vínculo de ausencia de permiso |
| Campo o menú cambió | La interfaz web evoluciona | Usa la búsqueda de la consola y la referencia oficial | Sigue el concepto y confirma el nombre actual |

### Refuerzo breve de errores ya observados

- **SSH timeout:** antes de cambiar IAM al azar, separa ruta de red, firewall/IAP, estado de VM, IP y método de autenticación. Un timeout suele indicar que ni siquiera se estableció la conexión; no prueba por sí solo un error de clave.
- **MIG backend `UNHEALTHY`:** confirma health check, puerto, ruta, proceso escuchando y regla de firewall desde los rangos de comprobación. Aumentar permisos IAM no vuelve saludable una aplicación que no responde.

Estos dos recordatorios no sustituyen el tema de hoy; sirven para reforzar la disciplina de diagnosticar por capas.

## Impacto en costos y limpieza

Las tareas propuestas son de lectura y no crean recursos. Aun así:

- Cloud Shell no cobra por la sesión estándar, pero los recursos que consultes pueden seguir generando cargos.
- Consultar Cloud Asset Inventory, IAM o listas de red no elimina nada.
- Un presupuesto de tipo *alerts-only* no detiene automáticamente el gasto. La documentación actual también describe *spend cap budgets* para servicios compatibles; no asumas que están disponibles para todo servicio o proyecto.
- La exportación de facturación a BigQuery puede producir almacenamiento y consultas cobrables; hoy solo se inspecciona o simula.
- Deshabilitar una API no equivale a borrar los recursos creados con ella.

Al terminar:

```bash
unset PROJECT_ID
unset BILLING_ACCOUNT_ID
```

Si usaste un proyecto temporal de Google Skills, sigue sus instrucciones de finalización. Si usaste una cuenta propia, revisa Billing, Compute Engine, VPC, Cloud SQL, Cloud Storage y cualquier otro servicio del laboratorio; elimina únicamente recursos que tú hayas creado y cuyo nombre/alcance hayas confirmado.

## Ficha de errores

Registra cada fallo, incluso si acertaste por adivinanza:

| Q | Mi respuesta | Correcta | Confianza | Tipo de error | Evidencia que ignoré | Regla corregida | Próxima revisión |
|---:|---|---|:---:|---|---|---|---|
|  |  |  |  | Concepto / lectura / inglés / alcance / prisa |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |

Plantilla de recuerdo:

> Cuando el requisito dice **___**, primero comparo **___** con **___**. Elijo **___** porque **___**. Descarto **___** porque **___**.

## Criterios de autoevaluación

| Estado | Evidencia mínima | Acción |
|---|---|---|
| Verde | 20–25/25, ningún grupo por debajo de 4/5 y explicaciones sin consultar | Conserva las dos recuperaciones elegidas y revisa errores de baja confianza |
| Amarillo | 17–19/25 o un grupo en 3/5 | Repite las dos tareas más débiles y recupera sus reglas en 24–48 horas |
| Rojo | 0–16/25 o algún grupo en 0–2/5 | Completa las dos tareas, reestudia las decisiones de esos grupos y prepara un mini-control nuevo |

Además, marca:

- [ ] Respondí las 25 preguntas sin mirar soluciones.
- [ ] Corregí también los aciertos de baja confianza.
- [ ] Elegí las dos tareas por evidencia, no por preferencia.
- [ ] Ejecuté o simulé exactamente dos tareas.
- [ ] Puedo distinguir Organization Policy de IAM.
- [ ] Puedo distinguir Workforce de Workload Identity Federation.
- [ ] Puedo separar API, permiso, cuota y disponibilidad regional.
- [ ] Puedo distinguir Asset Inventory, Monitoring, Logging y API específica.
- [ ] Puedo explicar presupuesto, vínculo de facturación y exportación.
- [ ] No guardé secretos ni afirmé dominio sin evidencia.

## Documentación oficial

Consultada el 13/10/2026:

- [Associate Cloud Engineer certification](https://cloud.google.com/learn/certification/cloud-engineer)
- [Resource hierarchy](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy)
- [Organization Policy overview](https://cloud.google.com/resource-manager/docs/organization-policy/overview)
- [IAM overview](https://cloud.google.com/iam/docs/overview)
- [Workforce Identity Federation](https://cloud.google.com/iam/docs/workforce-identity-federation)
- [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation)
- [Cloud Asset Inventory overview](https://cloud.google.com/asset-inventory/docs/overview)
- [Cloud Monitoring overview](https://cloud.google.com/monitoring/docs/monitoring-overview)
- [Geography and regions](https://cloud.google.com/docs/geography-and-regions)
- [Cloud Billing budgets and alerts](https://cloud.google.com/billing/docs/how-to/budgets)
- [Export Cloud Billing data to BigQuery](https://cloud.google.com/billing/docs/how-to/export-data-bigquery)
- [`gcloud auth list`](https://cloud.google.com/sdk/gcloud/reference/auth/list)
- [`gcloud config list`](https://cloud.google.com/sdk/gcloud/reference/config/list)
- [`gcloud projects describe`](https://cloud.google.com/sdk/gcloud/reference/projects/describe)
- [`gcloud projects get-ancestors`](https://cloud.google.com/sdk/gcloud/reference/projects/get-ancestors)
- [`gcloud org-policies list`](https://cloud.google.com/sdk/gcloud/reference/org-policies/list)
- [`gcloud projects get-iam-policy`](https://cloud.google.com/sdk/gcloud/reference/projects/get-iam-policy)
- [`gcloud services list`](https://cloud.google.com/sdk/gcloud/reference/services/list)
- [`gcloud compute networks list`](https://cloud.google.com/sdk/gcloud/reference/compute/networks/list)
- [`gcloud compute networks subnets list`](https://cloud.google.com/sdk/gcloud/reference/compute/networks/subnets/list)
- [`gcloud asset search-all-resources`](https://cloud.google.com/sdk/gcloud/reference/asset/search-all-resources)
- [`gcloud billing projects describe`](https://cloud.google.com/sdk/gcloud/reference/billing/projects/describe)
- [`gcloud billing budgets list`](https://cloud.google.com/sdk/gcloud/reference/billing/budgets/list)

---

## Soluciones justificadas

### Solution 1 — B

Environment folders plus separate projects provide inherited governance and separate quota/cost boundaries. **A** labels within one project do not separate quotas or IAM/policy boundaries. **C** billing accounts do not replace resource hierarchy. **D** VPC separation alone does not provide the required project boundaries.

### Solution 2 — C

Organization Policy is designed to constrain permitted resource configurations at hierarchy levels. **A** monitors cost. **B** controls who can act, not which configurations are globally permitted. **D** detects operational conditions after or during operation.

### Solution 3 — C

IAM answers whether the principal may request an action; Organization Policy can still reject a forbidden configuration. **A** is false because IAM does not override the constraint. **B** adds excessive access and still does not remove the constraint. **D** enables a service but does not change policy inheritance.

### Solution 4 — A and B

Moving a project can change inherited IAM bindings and effective organization policies because the ancestor path changes. **C** resource location does not automatically change. **D** a move does not automatically replace billing association. **E** project labels are not automatically deleted merely because the parent changes.

### Solution 5 — A

Labels are appropriate metadata for grouping/filtering supported resources and allocating costs without changing hierarchy. **B** creates needless governance roots. **C** misuses broad IAM roles. **D** zones are deployment locations, not cost categories.

### Solution 6 — C

A group plus an appropriate predefined role centralizes membership and supports least privilege. **A** is overprivileged. **B** creates repetitive, error-prone grants. **D** shares a sensitive workload credential with humans and destroys accountability.

### Solution 7 — A

Workforce Identity Federation gives external people federated access using an external identity provider. **B** is for software workloads. **C** keys represent service accounts and are long-lived secrets. **D** connects networks, not identities.

### Solution 8 — A and B

Workload Identity Federation supplies short-lived credentials to external software, while minimum deployment permissions limit impact. **C** introduces a long-lived secret. **D** targets human workforces. **E** violates least privilege.

### Solution 9 — A

Access can remain through a binding inherited from a folder or organization, or through another group membership; inherited bindings are the best next check. **B** affects cost notifications. **C** is networking. **D** API enablement does not explain an unchanged IAM grant.

### Solution 10 — A

A standalone organization lets an eligible identity setup establish an organization resource and central governance. **B** is a network setting. **C** analyzes costs. **D** is a compute availability construct.

### Solution 11 — A

The error explicitly identifies a disabled service; verify the project and enable that API if authorized. **B** applies to capacity failures. **C** is unrelated. **D** is excessive and does not itself enable the API.

### Solution 12 — A

First determine usage and limit, then reduce the request or justify a quota increase. **B** changes identity management. **C** removes visibility. **D** does not solve a regional resource quota.

### Solution 13 — B

Product availability must be verified by location; choose a supported location or redesign. **A** is false: billing does not make every product global. **C** permissions do not create regional availability. **D** cost alerts do not change the service catalog.

### Solution 14 — C

Google Cloud VPC networks are global and their subnetworks are regional. **A** and **B** reverse or misstate scopes. **D** is false because these network resources belong to projects even though project consumption can be billed.

### Solution 15 — A and B

Spreading across zones and using health-aware regional distribution removes a single-zone dependency and routes away from failed instances. **C** preserves the single-zone failure domain. **D** monitors spend, not availability. **E** labels do not test health.

### Solution 16 — A

Cloud Asset Inventory is designed to search resource metadata across a chosen accessible scope. **B** tests endpoint availability rather than cataloging all resources. **C** monitors spend. **D** provides name resolution.

### Solution 17 — A

Cloud Monitoring stores and visualizes operational metrics such as latency, request count, and errors. **B** catalogs assets. **C** enforces configuration constraints. **D** manages identities.

### Solution 18 — A

Cloud Logging is the starting point for individual structured log entries and request context. **B** contains cost records. **C** changes capacity. **D** grants access and provides no application event details.

### Solution 19 — B

AI assistance accelerates analysis, but the engineer remains responsible for validation, least privilege, policy, cost, and rollback. **A** removes necessary review. **C** grants excessive access without a justified principal/use. **D** reduces auditability exactly when it is needed.

### Solution 20 — A

The resource-specific service console or API is the direct source for current operational configuration. **B** describes costs. **C** describes group identity. **D** is organizational metadata, not database state.

### Solution 21 — A and B

Linking crosses two resources: the project and the target billing account, so authorization must cover both sides. **C** and **D** are unrelated service permissions. **E** is broader governance access and is not inherently required for a billing association.

### Solution 22 — C

An alerts-only budget sends configured notifications but does not automatically stop usage or charges. **A**, **B**, and **D** describe destructive or limiting actions that are not the default behavior of an alerts-only budget. Some supported services now offer separately configured spend-cap budgets; that does not change the stated scenario.

### Solution 23 — A

Billing export to BigQuery supports repeatable SQL over cost and usage data. **B** captures network flow metadata. **C** is manual and not queryable. **D** increases technical capacity and does not export costs.

### Solution 24 — A

API enablement controls whether the project can call a service, while existing resources have their own lifecycle and may remain billable. **B** is invented. **C** confuses a control-plane service with its resources. **D** is not a general billing rule.

### Solution 25 — A and B

Consistent labels provide allocation dimensions, and a BigQuery billing export supplies queryable records. **C** violates accountability and least privilege. **D** misuses geography as accounting structure. **E** removes useful operational evidence and does not improve cost allocation.
