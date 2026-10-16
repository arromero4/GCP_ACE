# ACE 2026 - Lección 16: Repaso guiado del dominio 1: decisiones y errores comunes

> **Fecha:** 12 de octubre de 2026  
> **Dominio de la guía ACE:** 1.1 Setting up cloud projects and accounts / 1.2 Managing billing configuration  
> **Práctica exacta del plan:** Resuelve 20 escenarios; justifica y corrige cada respuesta.  
> **Duración sugerida:** 90-120 minutos  
> **Caso conductor:** SiteOps Tracker, proyecto ficticio de portafolio con React + TypeScript, Node.js + TypeScript, PostgreSQL y servicios de Google Cloud.  
> **Documentación y comandos verificados:** 12 de octubre de 2026.

Esta es una sesión de integración y recuperación activa. No añade un dominio nuevo: organiza lo aprendido para que puedas reconocer la decisión correcta bajo presión de examen. Recibir este archivo no demuestra dominio; la evidencia será explicar las decisiones sin apuntes, completar la práctica y corregir tus errores.

## 1. Objetivo

Al terminar la sesión podrás:

1. reconstruir el mapa del dominio 1 sin confundir organización, carpeta, proyecto y recurso;
2. distinguir **IAM**, **Organization Policy**, habilitación de APIs, cuotas y facturación;
3. elegir una identidad adecuada para personas internas, personal externo y cargas de trabajo;
4. relacionar ubicación, VPC, subred y disponibilidad de productos;
5. decidir entre Google Cloud Observability, Cloud Asset Inventory y la API específica de un servicio;
6. explicar qué resuelve un presupuesto y qué no resuelve;
7. inspeccionar de forma segura un entorno mediante Console y comandos de solo lectura;
8. resolver **20 escenarios originales en inglés**, justificar cada respuesta y corregir cada error.

### Evidencia mínima de hoy

Conserva lo siguiente:

- un mapa de memoria del dominio 1;
- la tabla de resultados de la práctica de Console/CLI o de la alternativa conceptual;
- una hoja de respuestas para los 20 escenarios;
- una justificación de una frase por respuesta antes de consultar las soluciones;
- una ficha por cada error o acierto inseguro;
- tu puntuación y el tiempo empleado.

No anotes correos reales, IDs reales, nombres internos, tokens, claves ni datos sensibles en un documento que vayas a compartir.

## 2. Alineación con la guía oficial ACE

La guía oficial adjunta asigna aproximadamente el 20 % del examen al dominio **Setting up a cloud solution environment**. Sus dos bloques son:

### 2.1 Setting up cloud projects and accounts

- resource hierarchy;
- organizational policies;
- IAM roles en proyectos;
- usuarios y grupos de Cloud Identity;
- habilitación de APIs;
- configuración inicial de Google Cloud Observability;
- cuotas y solicitudes de aumento;
- standalone organizations;
- cloud networking;
- disponibilidad geográfica de productos;
- Cloud Asset Inventory y Gemini Cloud Assist;
- Workforce Identity Federation.

### 2.2 Managing billing configuration

- cuentas de facturación;
- vinculación de proyectos;
- budgets and alerts;
- billing exports.

El examen suele combinar varias piezas en un mismo caso. Por ejemplo, una creación de VM puede fallar aunque el usuario tenga un rol correcto si la API está deshabilitada, la cuota es insuficiente, una Organization Policy lo prohíbe o el producto no está disponible en la ubicación elegida.

## 3. Prerrequisitos explicados desde cero

### 3.1 Vocabulario base

Antes de decidir, separa estas cinco preguntas:

1. **¿Dónde vive el recurso?** Organización, carpeta, proyecto y ubicación.
2. **¿Quién hace la solicitud?** Usuario, grupo, cuenta de servicio o identidad federada.
3. **¿Puede hacerlo?** IAM concede permisos mediante roles.
4. **¿Está permitido configurarlo así?** Organization Policy y otros controles pueden restringir la configuración.
5. **¿Puede consumirse el servicio?** API habilitada, cuota disponible, producto disponible y facturación válida cuando corresponda.

Una respuesta correcta a una de estas preguntas no responde automáticamente las demás.

### 3.2 Requisitos para la práctica con cuenta

- acceso a Google Cloud Console o Cloud Shell;
- un proyecto de laboratorio que tengas autorización para inspeccionar;
- Google Cloud CLI disponible;
- permisos de lectura suficientes para algunos recursos.

La práctica principal es de **solo lectura**. Un error `PERMISSION_DENIED` no implica que el recurso no exista: puede significar que tu principal carece del permiso requerido.

### 3.3 Si no tienes cuenta, permisos o crédito

Usa la alternativa conceptual completa de la sección 10. No necesitas crear una cuenta, introducir una tarjeta ni ejecutar comandos para completar los objetivos de decisión de hoy.

## 4. Recuperación activa espaciada

Responde primero sin mirar las notas. Después compara con la guía breve.

### 4.1 Días hábiles previos

#### Lección 15 - Laboratorio integral de entorno

Sin mirar, escribe el orden lógico de estas comprobaciones:

1. identidad y proyecto activos;
2. jerarquía y políticas;
3. IAM;
4. APIs y cuotas;
5. facturación;
6. red, ubicación e inventario;
7. evidencia y limpieza.

**Regla de recuperación:** no concluyas “es IAM” hasta comprobar también API, cuota, facturación, ubicación y restricciones.

#### Lección 14 - Standalone organizations y Workforce Identity Federation

Completa de memoria:

- una **standalone organization** sirve para ________;
- **Workforce Identity Federation** federa identidades de ________;
- **Workload Identity Federation** federa identidades de ________.

Respuesta guía: organización sin requerir Cloud Identity en los casos compatibles; personas externas; cargas de trabajo externas.

#### Lección 13 - Observability, Asset Inventory y Gemini Cloud Assist

Clasifica cada necesidad:

| Necesidad | Herramienta principal |
|---|---|
| Latencia, errores, métricas, logs y trazas de una aplicación | Google Cloud Observability |
| Inventario, metadatos, postura y búsqueda de recursos | Cloud Asset Inventory |
| Sugerencias asistidas y explicación contextual, siempre verificadas | Gemini Cloud Assist |

#### Lección 12 - Red y geografía

Recupera dos reglas:

- una **VPC network** es global;
- una **subnet** es regional.

Una zona es un dominio de fallo dentro de una región. La disponibilidad de un producto o una función debe verificarse; no se deduce solo por existir una región.

### 4.2 Hace 7 días - Lección 11

**Pregunta:** ¿qué diferencia hay entre un budget y una billing export?

- El **budget** compara gasto con un objetivo y dispara avisos o integraciones configuradas.
- La **billing export** envía datos de costos y uso a BigQuery para análisis.
- Un presupuesto **alerts-only** no establece automáticamente un límite duro de gasto.

### 4.3 Hace 21 días - Lección 1

Dibuja sin apuntes:

```text
Organization
└── Folder
    ├── Project: siteops-dev
    │   └── Resources
    └── Project: siteops-prod
        └── Resources
```

Después explica por qué un proyecto no es solo una carpeta visual: es un límite fundamental para APIs, IAM, cuotas, facturación y ciclo de vida.

## 5. Mapa mental del dominio 1

Usa la cadena **SCOPE - IDENTITY - PERMISSION - SERVICE - CAPACITY - MONEY - EVIDENCE**:

1. **Scope:** organización, carpeta, proyecto, recurso y ubicación.
2. **Identity:** quién o qué solicita la acción.
3. **Permission:** rol IAM y nivel donde está concedido.
4. **Service:** API habilitada y configuración básica.
5. **Capacity:** cuota y límites del sistema.
6. **Money:** cuenta de facturación, budget y exportación.
7. **Evidence:** inventario, métricas, logs, comandos y resultados.

### Analogía: abrir una nueva sede de SiteOps Tracker

- La **organización** es la empresa completa.
- Las **carpetas** son divisiones o áreas.
- Los **proyectos** son espacios operativos con presupuesto, accesos y ciclo de vida propios.
- IAM es la credencial que indica quién puede entrar y qué puede hacer.
- Organization Policy es el reglamento del edificio que nadie puede ignorar por tener una tarjeta poderosa.
- Habilitar una API es abrir el mostrador del servicio.
- La cuota es la capacidad máxima autorizada del mostrador.
- La facturación define quién paga el consumo.
- Un budget es la alarma de gasto, no necesariamente el interruptor general.
- Cloud Asset Inventory es el catálogo de activos; Observability es el tablero de salud en operación.

## 6. Conceptos paso a paso

### 6.1 Jerarquía: organización, carpeta, proyecto y recurso

La jerarquía crea un árbol de administración:

1. **Organization:** raíz para gobierno central.
2. **Folder:** agrupación opcional dentro de una organización; puede representar equipos, entornos o unidades de negocio.
3. **Project:** contenedor fundamental requerido para usar Google Cloud.
4. **Service resource:** VM, bucket, base de datos, red u otro recurso del servicio.

Las políticas aplicadas en niveles superiores pueden heredarse por los descendientes. Para SiteOps Tracker, separar desarrollo y producción en proyectos distintos permite:

- aislar cambios y fallos;
- separar permisos;
- analizar costos por entorno;
- manejar cuotas y APIs de forma explícita;
- eliminar un entorno temporal con menos riesgo para el otro.

**Error común:** usar labels como sustituto de proyectos. Las labels ayudan a clasificar y analizar; no crean por sí mismas un límite fuerte de IAM, facturación o ciclo de vida.

### 6.2 Organization Policy no es IAM

**IAM** responde: “¿quién puede hacer qué sobre cuál recurso?”.  
**Organization Policy** responde: “¿qué configuraciones están permitidas dentro de este ámbito?”.

Ejemplo de SiteOps Tracker:

- El grupo `network-auditors` recibe un rol de lectura: eso es IAM.
- La organización impide determinadas configuraciones de red no aprobadas: eso es Organization Policy.

Una persona con un rol amplio puede seguir bloqueada por una restricción de organización. Conceder `Owner` no es una solución general y suele empeorar el riesgo.

**Herencia IAM:** los allow policies de los ancestros se suman a los del recurso. Omitir un binding heredado en el proyecto no lo revoca. Debes corregir el binding en el nivel donde se otorgó, rediseñar el alcance o usar un mecanismo explícito apropiado, como una deny policy, si el requisito lo exige.

### 6.3 Principal, rol y recurso

Un grant IAM enlaza tres piezas:

- **principal:** persona, grupo o workload;
- **role:** colección de permisos;
- **resource:** nivel donde se concede.

Para SiteOps Tracker:

- auditores humanos: grupo de Cloud Identity con rol predefinido de lectura;
- API Node.js desplegada en Google Cloud: service account adjunta al runtime, sin claves descargadas;
- contratistas que autentican mediante su IdP externo: Workforce Identity Federation;
- pipeline externo no humano: Workload Identity Federation.

Prefiere grupos para acceso humano repetible y roles predefinidos de mínimo privilegio. Los basic roles (`Owner`, `Editor`, `Viewer`) son amplios; no son la primera elección cuando existe un rol predefinido más específico.

### 6.4 Workforce frente a Workload Identity Federation

La palabra decisiva es el sujeto:

- **workforce = personas**: empleados, socios o contratistas que usan un IdP externo;
- **workload = software**: CI/CD, aplicaciones o procesos fuera de Google Cloud.

Workforce Identity Federation usa los atributos del IdP para autenticar usuarios y puede autorizarse mediante IAM. No requiere sincronizar cada cuenta externa como una identidad administrada por Google.

### 6.5 APIs, IAM, cuotas y límites

Una acción puede requerir simultáneamente:

1. API habilitada en el proyecto consumidor;
2. permiso IAM para el principal;
3. cuota suficiente;
4. ausencia de una restricción que bloquee la configuración;
5. producto disponible en la ubicación;
6. facturación válida si el servicio es facturable.

**Error común:** aumentar cuota ante `PERMISSION_DENIED`. La cuota no concede permisos.  
**Error común:** conceder Owner ante `SERVICE_DISABLED`. IAM no habilita automáticamente una API.  
**Error común:** crear proyectos adicionales para evadir cuotas. Eso fragmenta gobierno y no sustituye una solicitud legítima de ajuste.

### 6.6 Facturación: cuenta, vínculo, budget y exportación

La **Cloud Billing account** paga costos de los proyectos vinculados, pero no es un proyecto y tiene sus propios roles IAM.

- Vincular un proyecto requiere permisos adecuados tanto sobre el proyecto como sobre la cuenta de facturación.
- Un **alerts-only budget** vigila tendencias y umbrales; no detiene automáticamente el uso o el cobro.
- Una **billing export** crea tablas en BigQuery para análisis detallado. El almacenamiento y las consultas de BigQuery pueden generar costo.

Para SiteOps Tracker, un budget de desarrollo puede avisar al equipo cuando el gasto real o previsto cruza umbrales. La exportación permite responder con SQL qué servicio, SKU o proyecto generó el costo.

### 6.7 Región, zona, VPC y subred

- Una **region** contiene zonas.
- Una **zone** es un dominio de fallo dentro de la región.
- Una **VPC network** es un recurso global.
- Una **subnet** es regional.

No elijas ubicación solo por cercanía. Considera latencia, residencia de datos, disponibilidad del producto y de la función, continuidad, conectividad y precio.

Ejemplo: SiteOps Tracker puede tener una sola VPC con subredes regionales para dos grupos de sedes. La existencia de la VPC global no convierte a una VM zonal en un recurso global ni garantiza alta disponibilidad.

### 6.8 Observability, Asset Inventory y la API del servicio

| Necesidad | Elección | Por qué no las alternativas |
|---|---|---|
| Latencia, errores, métricas, logs y trazas | Google Cloud Observability | Asset Inventory no es un sistema de telemetría operacional. |
| Buscar recursos y metadatos para auditoría | Cloud Asset Inventory | Monitoring no sustituye un catálogo de activos. |
| Estado operativo inmediato y autoritativo de una VM | Compute Engine API/CLI | Asset Inventory puede reflejar datos con retraso y no está diseñado para consultas en tiempo real que controlen producción. |
| Ayuda contextual y sugerencias | Gemini Cloud Assist | La IA no reemplaza permisos, documentación, evidencia ni revisión humana. |

### 6.9 Standalone organization

Una standalone organization ofrece un recurso de organización sin requerir Cloud Identity en los casos compatibles. Es útil para determinados clientes nuevos y entornos de aprendizaje o prueba que necesiten jerarquía y gobierno de organización.

No la confundas con:

- un proyecto sin organización;
- una carpeta;
- una segunda cuenta de facturación;
- Workforce Identity Federation.

La decisión debe considerar propiedad, recuperación, varios administradores y restricciones base. Evita un único propietario como punto único de fallo.

## 7. Tabla de decisiones: servicio correcto y alternativas

| Requisito del escenario | Primera elección | No elegir primero | Razón |
|---|---|---|---|
| Aplicar una restricción de configuración a muchos proyectos | Organization Policy en el ancestro adecuado | IAM | IAM concede permisos; no expresa por sí solo la configuración permitida. |
| Autorizar a auditores de red | Grupo + rol predefinido mínimo | Roles individuales o `Owner` | El grupo reduce mantenimiento; el rol específico reduce privilegios. |
| Personas externas con IdP corporativo | Workforce Identity Federation | Service account | Las service accounts representan workloads, no usuarios humanos. |
| Pipeline externo sin claves de larga duración | Workload Identity Federation | Workforce Identity Federation | El sujeto es una carga de trabajo. |
| Inventario de recursos en muchos proyectos | Cloud Asset Inventory | Cloud Monitoring | Inventory está diseñado para búsqueda y análisis de metadatos. |
| Estado actual de una VM durante un incidente | API/CLI de Compute Engine | Asset Inventory | La consulta operacional debe usar la fuente del servicio. |
| Tendencia de gasto y avisos | Budget | Quota | La cuota limita una métrica técnica; no es un presupuesto monetario. |
| Análisis SQL detallado de costos | Billing export a BigQuery | Budget | El budget alerta; no ofrece por sí solo un conjunto analítico detallado. |
| Despliegue en dos regiones dentro de una red | Una VPC global con subredes regionales | Una VPC por zona | Las VPC son globales y las subredes regionales. |
| Capacidad insuficiente antes de un despliegue | Solicitud planificada de ajuste de cuota | Crear proyectos para evadir el límite | La solicitud conserva gobierno y permite evaluación del proveedor. |

## 8. Glosario bilingüe

| English term | Español y significado práctico |
|---|---|
| organization resource | recurso de organización; raíz de la jerarquía |
| folder | carpeta; agrupación opcional dentro de una organización |
| project | proyecto; contenedor fundamental de servicios y recursos |
| resource hierarchy | jerarquía de recursos |
| ancestor / descendant | ancestro / descendiente |
| inheritance | herencia de políticas desde niveles superiores |
| principal | identidad que solicita acceso |
| permission | permiso; acción individual autorizable |
| role | rol; colección de permisos |
| allow policy | política de autorización que enlaza principales y roles |
| deny policy | política que niega permisos concretos aunque exista un grant aplicable |
| least privilege | mínimo privilegio |
| Organization Policy | políticas que restringen configuraciones en la jerarquía |
| constraint | restricción evaluada por Organization Policy |
| Cloud Identity | servicio para gestionar identidades y grupos |
| Workforce Identity Federation | federación de personas desde un IdP externo |
| Workload Identity Federation | federación de cargas de trabajo externas |
| service account | identidad para una aplicación o carga de trabajo |
| service/API enablement | habilitación de un servicio o API en un proyecto |
| quota | cuota; capacidad de consumo asignada y ajustable en ciertos casos |
| system limit | límite fijo que normalmente no puede aumentarse |
| billing account | cuenta de facturación que paga consumo elegible |
| billing link | vínculo entre proyecto y cuenta de facturación |
| budget alert | alerta al alcanzar un umbral de gasto real o previsto |
| billing export | exportación de datos de facturación a BigQuery |
| region / zone | región / zona |
| VPC network | red VPC global |
| subnet | subred regional |
| Cloud Asset Inventory | inventario global de metadatos de activos |
| Google Cloud Observability | conjunto de capacidades de Monitoring, Logging, Trace y otras señales |
| active account | cuenta que la CLI usa para autenticar solicitudes |
| configured project | proyecto predeterminado de la configuración activa de `gcloud` |

## 9. Práctica guiada con Console y CLI

### 9.1 Reglas de seguridad

1. Usa un proyecto de laboratorio autorizado.
2. Confirma identidad y proyecto antes de cada inspección.
3. No habilites APIs, no cambies IAM, no vincules facturación y no crees recursos para completar esta práctica.
4. Si un comando falla por permisos, registra el error y continúa con la alternativa conceptual.
5. Sustituye cualquier dato real por una etiqueta neutra en tus notas.

### 9.2 Recorrido en Google Cloud Console

#### Paso 1 - Contexto

1. Abre Google Cloud Console.
2. Usa el selector de proyecto.
3. Ve a **IAM & Admin > Settings** o al panel del proyecto.
4. Registra de forma anonimizada: project ID, project number y parent si es visible.

**Comprueba:** el display name puede cambiar; project ID y project number son identificadores distintos.

#### Paso 2 - IAM y Organization Policy

1. Ve a **IAM & Admin > IAM**.
2. Identifica un binding de lectura y anota principal, rol y nivel.
3. Ve a **IAM & Admin > Organization Policies** si tienes acceso.
4. Compara una restricción con un grant IAM.

**Pregunta:** ¿la evidencia muestra quién puede actuar o qué configuración está permitida?

#### Paso 3 - APIs y cuotas

1. Ve a **APIs & Services > Enabled APIs & services**.
2. Identifica una API habilitada.
3. Ve a **IAM & Admin > Quotas & System Limits**.
4. Elige una cuota y diferencia limit, current usage y dimensiones.

No solicites aumentos durante este repaso.

#### Paso 4 - Facturación

1. Ve a **Billing > Account management** si tienes permiso.
2. Verifica si el proyecto está vinculado.
3. Abre **Budgets & alerts** y observa el ámbito y los umbrales, sin crear ni editar.
4. Localiza **Billing export** y describe qué datos llegarían a BigQuery.

#### Paso 5 - Red e inventario

1. Ve a **VPC network > VPC networks**.
2. Identifica la VPC y las regiones de sus subredes.
3. Abre **Asset Inventory**.
4. Busca recursos del proyecto y revisa tipo, nombre y ubicación.
5. Abre **Monitoring > Overview** y explica por qué su función no es la misma que Asset Inventory.

### 9.3 Recorrido de Google Cloud CLI

Los siguientes comandos se verificaron en la referencia oficial vigente el 12 de octubre de 2026. Son de consulta.

#### A. Confirma identidad y proyecto

```bash
gcloud auth list --filter=status:ACTIVE --format="value(account)"
gcloud config list project
```

Define la variable solo después de confirmar visualmente el proyecto:

```bash
PROJECT_ID="TU_PROJECT_ID_DE_LABORATORIO"
gcloud projects describe "${PROJECT_ID}"
gcloud projects get-ancestors "${PROJECT_ID}"
```

**Resultado esperado:** una cuenta activa, un project ID conocido, metadatos del proyecto y, cuando existan y sean visibles, sus ancestros.

#### B. Inspecciona políticas, IAM y APIs

```bash
gcloud org-policies list --project="${PROJECT_ID}"
gcloud projects get-iam-policy "${PROJECT_ID}" --format=yaml
gcloud services list --enabled --project="${PROJECT_ID}"
```

**Interpretación:**

- una lista vacía de políticas establecidas directamente en el proyecto no demuestra ausencia de políticas heredadas;
- la política IAM del proyecto por sí sola puede no mostrar todo el acceso efectivo heredado;
- una API habilitada no demuestra que tu principal tenga el permiso para usarla.

#### C. Inspecciona red y activos

```bash
gcloud compute networks list --project="${PROJECT_ID}"
gcloud compute networks subnets list --project="${PROJECT_ID}"
gcloud asset search-all-resources \
  --scope="projects/${PROJECT_ID}" \
  --limit=20 \
  --format="table(assetType,displayName,location)"
```

**Resultado esperado:** redes globales, subredes con región y hasta 20 activos visibles para el principal. Si Cloud Asset Inventory y la API de Compute no están disponibles para tu identidad, registra el error; no habilites servicios sin autorización.

#### D. Inspecciona facturación y budgets

```bash
gcloud billing projects describe "${PROJECT_ID}"
```

Solo si el comando anterior muestra una cuenta y tienes permiso para consultarla, copia su ID en una variable local:

```bash
BILLING_ACCOUNT_ID="TU_BILLING_ACCOUNT_ID"
gcloud billing budgets list --billing-account="${BILLING_ACCOUNT_ID}"
```

No compartas el identificador real. Un fallo aquí puede deberse a permisos de Billing separados de los permisos del proyecto.

### 9.4 Matriz de evidencia

Completa esta tabla con datos anonimizados:

| Control | Evidencia observada | Decisión o riesgo | Estado |
|---|---|---|---|
| Cuenta activa |  | ¿Es la identidad esperada? | ☐ |
| Proyecto |  | ¿Es laboratorio y no producción? | ☐ |
| Jerarquía |  | ¿Qué hereda? | ☐ |
| Organization Policy |  | ¿Qué configuración restringe? | ☐ |
| IAM |  | ¿Cuál es el grant mínimo? | ☐ |
| APIs |  | ¿Qué servicio está habilitado? | ☐ |
| Cuota |  | ¿Hay capacidad suficiente? | ☐ |
| Facturación |  | ¿Existe vínculo y quién lo administra? | ☐ |
| Budget |  | ¿Alerta o limita? | ☐ |
| VPC/subred |  | ¿Global o regional? | ☐ |
| Asset Inventory |  | ¿Inventario o estado en tiempo real? | ☐ |
| Observability |  | ¿Qué señal operacional ofrece? | ☐ |

## 10. Alternativa conceptual íntegra sin cuenta ni crédito

Usa el siguiente estado ficticio de SiteOps Tracker:

| Campo | Valor ficticio |
|---|---|
| Organization | `organizations/100000000001` |
| Folder | `folders/200000000002` (`siteops`) |
| Project ID | `siteops-dev-demo` |
| Project number | `300000000003` |
| Active principal | `learner@example.invalid` |
| IAM directo | grupo `auditors@example.invalid` con Network Viewer |
| IAM heredado | grupo `platform-admins@example.invalid` con un rol administrativo desde la carpeta |
| Organization Policy | solo regiones aprobadas; aplicada en la carpeta |
| APIs | Service Usage, Cloud Asset y Compute habilitadas |
| Cuota CPU regional | límite 24, uso 8 |
| VPC | `siteops-dev-vpc`, modo custom |
| Subred | `siteops-mx`, región ficticia de ejemplo, CIDR `10.20.0.0/24` |
| Billing | vinculada y habilitada |
| Budget | MXN 500, alerts-only, umbrales 50/90/100 % |
| Billing export | dataset ficticio `billing_export` |

Completa estas tareas:

1. Dibuja la jerarquía y marca dónde se aplica cada política.
2. Explica por qué el grupo administrativo sigue teniendo acceso aunque no aparezca como binding directo del proyecto.
3. Calcula la capacidad de cuota disponible: `24 - 8 = 16` vCPU.
4. Decide qué revisar antes de solicitar 20 vCPU adicionales.
5. Explica por qué el budget no garantiza detener el gasto en MXN 500.
6. Elige Asset Inventory para un catálogo y Compute API para estado inmediato.
7. Indica qué cambiarías para producción sin copiar datos reales: proyecto separado, permisos mínimos, budget propio y diseño regional adecuado.
8. Escribe una evidencia equivalente para cada fila de la matriz de la sección 9.4.

### Resultado conceptual esperado

- jerarquía correctamente ordenada;
- IAM y Organization Policy diferenciados;
- 16 vCPU disponibles, insuficientes para una demanda adicional de 20 si todo debe ejecutarse a la vez;
- reconocimiento de que budget alerta y billing export analiza;
- VPC global y subred regional correctamente identificadas;
- distinción entre inventario y estado operacional;
- ninguna necesidad de crear recursos o usar crédito.

## 11. Resultado esperado de la práctica

Al terminar debes poder producir, sin consultar la solución, un diagnóstico como este:

> “Estoy usando la identidad y el proyecto correctos. El proyecto pertenece a un ámbito concreto y puede heredar IAM y Organization Policy. La API necesaria está habilitada, pero eso no sustituye permisos ni cuota. La VPC es global y la subred regional. El proyecto tiene un vínculo de facturación; el budget observado es de alertas y no un corte automático. Asset Inventory sirve para auditar metadatos, mientras que la API del servicio sirve para estado inmediato y Observability para señales operativas.”

Si no puedes explicar cada oración con una evidencia, marca el punto para repaso; no asumas dominio.

## 12. Solución de problemas

| Síntoma | Causa probable | Comprobación | Corrección segura |
|---|---|---|---|
| El comando consulta otro proyecto | Configuración activa incorrecta | `gcloud config list project` | Usa `--project` o cambia la configuración solo después de verificar el ID. |
| `PERMISSION_DENIED` | Falta de permiso o ámbito equivocado | Cuenta activa, rol y nivel del binding | Solicita el rol mínimo en el ámbito correcto; no concedas `Owner` por reflejo. |
| `SERVICE_DISABLED` | API deshabilitada | `gcloud services list --enabled` | Pide a un principal autorizado que evalúe y habilite la API si procede. |
| `RESOURCE_EXHAUSTED` o error de cuota | Cuota insuficiente o dimensión incorrecta | Quotas & System Limits, región y métrica | Reduce demanda, cambia diseño o solicita ajuste con anticipación. |
| No aparecen políticas | Solo miraste políticas directas | Ancestros y políticas efectivas | Revisa organización y carpetas; una lista directa vacía no prueba ausencia de herencia. |
| IAM del proyecto parece correcto, pero hay acceso inesperado | Grant heredado | `get-ancestors` y políticas de ancestros | Corrige el binding en su nivel de origen o rediseña el ámbito. |
| Billing muestra acceso denegado | Roles de Billing separados | Cuenta activa y permisos en billing account | Solicita acceso de solo lectura o usa la alternativa conceptual. |
| Asset Inventory no muestra un cambio inmediato | Permiso, API o consistencia del inventario | Repite consulta y contrasta con la API del servicio | Para decisiones operativas en tiempo real usa la API específica. |
| VPC aparece, pero no ves subredes | Proyecto, filtros o permisos incorrectos | Proyecto y `subnets list` sin filtros | Verifica alcance y permiso de lectura. |
| Console y CLI no coinciden | Proyecto, cuenta o tiempo de propagación diferentes | Selector de Console y configuración de CLI | Alinea identidad y proyecto; considera consistencia eventual. |

### Refuerzo breve de errores compartidos anteriormente

#### SSH timeout a una VM

No empieces concediendo `Owner` ni recreando la VM. Recorre:

1. proyecto, zona e instancia correctos;
2. ruta de acceso: IP externa, IAP, VPN o bastion;
3. VPC, subred, ruta y firewall;
4. puerto y origen permitidos;
5. IAM/OS Login y estado de la instancia;
6. evidencia en logs y diagnóstico del servicio.

#### Backend de MIG `UNHEALTHY`

Comprueba:

1. puerto y ruta del health check;
2. proceso de aplicación escuchando;
3. regla de firewall para los rangos de health checks aplicables;
4. named port y backend service;
5. plantilla e instancia correctas;
6. respuesta HTTP esperada y logs.

La conexión con el dominio 1 es el orden de diagnóstico: proyecto, ubicación, red, API, permisos, políticas y evidencia antes de hacer cambios amplios.

## 13. Impacto en costos y limpieza

### Costos

- La práctica indicada no crea recursos deliberadamente.
- Consultar configuración e inventario no debería producir una carga de infraestructura, pero respeta cuotas y revisa la documentación de precios si automatizas consultas a gran escala.
- Los recursos ya existentes siguen generando costos aunque cierres Cloud Shell.
- Consultar tablas de billing export en BigQuery puede generar costos de almacenamiento y procesamiento.
- Un alerts-only budget no impide automáticamente el gasto.

### Limpieza

Como la práctica es de solo lectura, no hay recursos que eliminar. Al finalizar:

```bash
unset PROJECT_ID
unset BILLING_ACCOUNT_ID
```

Si decidiste hacer cambios opcionales fuera de esta práctica, documenta cada cambio y revierte únicamente los recursos temporales que creaste. No deshabilites una API ni desvincules facturación de un proyecto compartido sin autorización: puede afectar cargas existentes.

## 14. Criterio rápido para responder escenarios ACE

Para cada pregunta, subraya:

1. **actor:** persona interna, persona externa o workload;
2. **verbo:** ver, crear, restringir, pagar, alertar, analizar;
3. **scope:** organización, carpeta, proyecto, recurso, región o zona;
4. **restricción:** mínimo privilegio, sin claves, menor operación, tiempo real;
5. **fallo:** autenticación, autorización, API, cuota, ubicación, billing o red;
6. **evidencia:** qué comando o producto confirma la hipótesis.

Descarta opciones que:

- conceden permisos más amplios de lo solicitado;
- cambian un componente no relacionado;
- confunden alerta con límite;
- confunden personas con workloads;
- asumen que todos los productos existen en todas las regiones;
- usan inventario eventual para una decisión operacional inmediata.

## 15. Documentación oficial

Fuentes consultadas y verificadas el 12 de octubre de 2026:

- [Associate Cloud Engineer certification](https://cloud.google.com/learn/certification/cloud-engineer)
- [Resource hierarchy](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy)
- [Organization Policy overview](https://cloud.google.com/resource-manager/docs/organization-policy/overview)
- [IAM overview](https://cloud.google.com/iam/docs/overview)
- [Enable and disable services](https://cloud.google.com/service-usage/docs/enable-disable)
- [View and manage quotas](https://cloud.google.com/docs/quotas/view-manage)
- [VPC networks](https://cloud.google.com/vpc/docs/vpc)
- [Google Cloud locations](https://cloud.google.com/about/locations)
- [Cloud Asset Inventory overview](https://cloud.google.com/asset-inventory/docs/overview)
- [Workforce Identity Federation](https://cloud.google.com/iam/docs/workforce-identity-federation)
- [Standalone organizations](https://cloud.google.com/resource-manager/docs/standalone-organization-overview)
- [Cloud Billing budgets and alerts](https://cloud.google.com/billing/docs/how-to/budgets)
- [Export Cloud Billing data to BigQuery](https://cloud.google.com/billing/docs/how-to/export-data-bigquery)
- [Cloud Monitoring overview](https://cloud.google.com/monitoring/docs/monitoring-overview)
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

La guía oficial adjunta `associate_cloud_engineer_exam_guide_english.pdf` es la referencia de cobertura. Los nombres, interfaces y opciones pueden cambiar; vuelve a confirmar la documentación antes de ejecutar cambios en un entorno real.

## 16. Evaluación: 20 escenarios originales en inglés

**Instructions:** Choose the best answer unless the question says **Choose two**. Record your answer and write one sentence explaining the requirement that determined it. Do not consult the solutions until all 20 are complete.

### Question 1

SiteOps Tracker needs separate development and production projects. A location restriction must apply to both environments, while each environment keeps independent project administration. What should the cloud engineer do?

A. Put all resources in one project and distinguish environments with labels.  
B. Create a SiteOps folder with development and production projects, then apply the organization policy to the folder.  
C. Create one VPC per environment and apply the organization policy to each VPC.  
D. Use two billing accounts because billing accounts are the parent of projects.

### Question 2

Security requires that project administrators cannot create disallowed external IP configurations, even if they can create virtual machines. Which control best enforces the requirement?

A. A Cloud Billing budget  
B. A broader IAM role for the administrators  
C. An Organization Policy constraint at the appropriate ancestor  
D. A Cloud Monitoring alert

### Question 3

An auditor can view a production project because `roles/viewer` was granted to the auditor's group on the parent folder. The project IAM page has no direct binding for that group. The auditor must lose access to this project. What should the engineer recognize first?

A. Removing the group from the project's local policy revokes the inherited grant.  
B. The inherited binding must be changed at its source or the hierarchy/access design must be adjusted.  
C. Disabling the Resource Manager API removes inherited access.  
D. Adding a second Viewer binding at the project overrides the folder binding.

### Question 4

Twenty network auditors need read-only access to VPC configuration in one project. Membership changes frequently. What is the most maintainable least-privilege approach?

A. Grant `roles/owner` to each auditor.  
B. Grant a suitable network viewer role to a managed group, then manage group membership.  
C. Share one service account key among all auditors.  
D. Grant Network Admin to every auditor at the organization level.

### Question 5

A developer has the required Compute Engine IAM role, but VM creation fails because the Compute Engine API is disabled. What is the best next action?

A. Ask an authorized principal to enable the Compute Engine API in the target project.  
B. Increase the regional CPU quota.  
C. Grant the developer Owner.  
D. Create a budget alert.

### Question 6

A planned rollout requires 300 regional vCPUs next week, but the project's applicable quota is 96. The architecture and demand are valid. What should the engineer do?

A. Create several projects only to bypass quota review.  
B. Request a quota adjustment in advance and track the request while validating fallback capacity.  
C. Add more IAM roles to the deployment service account.  
D. Move the billing account to another organization.

### Question 7

The team wants to deploy a managed database feature in a region selected for latency and data residency. What should the engineer do before finalizing the design?

A. Assume every Google Cloud product and feature is available in every region.  
B. Verify product and feature availability in the target location and evaluate availability requirements.  
C. Create a global VPC, which guarantees that every managed service is available.  
D. Increase all quotas in the project.

### Question 8

SiteOps Tracker needs private address ranges in two regions while keeping a single network administration boundary. Which design matches Google Cloud VPC behavior?

A. One global VPC network with a subnet in each required region  
B. One zonal VPC network for every zone  
C. One regional VPC network and one global subnet  
D. One billing account for each subnet

### Question 9

A project administrator must link a project to an existing billing account but receives a permission error. Which explanation is most accurate?

A. Project administration automatically includes every billing-account permission.  
B. Linking can require appropriate permissions on both the project and the billing account.  
C. The project must first be moved to the root of the organization.  
D. The Compute Engine API must be enabled before billing can be linked.

### Question 10

An alerts-only monthly budget reaches 100%. The finance team expects Google Cloud to stop all services automatically. What should the engineer explain?

A. Alerts-only budgets notify according to configured thresholds but do not automatically impose a hard spending cap.  
B. Every budget disables all APIs at 100%.  
C. A budget converts service quotas to zero.  
D. A budget deletes all project resources at the end of the month.

### Question 11

Finance needs itemized cost and usage data that analysts can query with SQL and join with project metadata. Which feature should be configured?

A. Cloud Billing export to BigQuery  
B. A single email budget alert  
C. A regional CPU quota  
D. Cloud Monitoring uptime checks

### Question 12

An auditor must find resources with a specific label across many accessible projects and review their metadata. Which service is the best fit?

A. Cloud Asset Inventory  
B. Cloud Trace  
C. Cloud DNS  
D. Cloud Billing budgets

### Question 13

During an incident, an automation must make a decision based on the current operational state of a Compute Engine VM. Which source should it use first?

A. A cached spreadsheet of assets  
B. Cloud Asset Inventory as the only real-time authority  
C. The Compute Engine API or CLI for the current VM state  
D. A billing export table

### Question 14

Users report intermittent latency in the SiteOps Tracker API. The team needs metrics, logs, and traces to investigate. Which product area should be used?

A. Google Cloud Observability  
B. Resource Manager folders  
C. Cloud Asset Inventory only  
D. Cloud Billing account management

### Question 15

External human contractors must sign in with their company's OIDC identity provider and access selected Google Cloud resources without creating a Google-managed account for each contractor. What should be used?

A. Workforce Identity Federation  
B. Workload Identity Federation  
C. A shared service account key  
D. A public firewall rule

### Question 16

An external CI pipeline needs short-lived access to deploy an application without storing a service account key. What should be used?

A. Workforce Identity Federation  
B. Workload Identity Federation  
C. A basic Owner role for all developers  
D. An alerts-only budget

### Question 17

A compatible new Google Cloud customer needs an organization resource for centralized hierarchy but does not use Cloud Identity. Which option should the engineer evaluate?

A. A standalone organization with properly managed owners  
B. A VPC network as the hierarchy root  
C. A billing account as the hierarchy root  
D. A Cloud Storage bucket with uniform access

### Question 18

The display name of a project changes, and an automation must continue addressing the same project. Which identifier should the automation use rather than the mutable display name?

A. The project ID  
B. The folder display name  
C. The billing account display name  
D. The default Compute Engine zone

### Question 19

Gemini Cloud Assist suggests granting Owner to resolve a permission error. What should the engineer do?

A. Apply the suggestion immediately because AI recommendations override IAM policy.  
B. Verify the active principal, required permission, scope, inherited policies, and official documentation, then grant only the minimum role if authorized.  
C. Disable all organization policies.  
D. Move the project to a different billing account.

### Question 20

An API is enabled and project billing is active, but an application request returns `PERMISSION_DENIED`. What should the engineer investigate next?

A. The active principal and whether IAM grants the required permission at an effective scope  
B. The monthly budget threshold only  
C. Regional vCPU quota only  
D. The project display name only

## 17. Hoja de respuestas y corrección

| # | Tu respuesta | Confianza 1-3 | Evidencia del escenario | Correcta | Regla corregida |
|---:|---|---:|---|---|---|
| 1 |  |  |  |  |  |
| 2 |  |  |  |  |  |
| 3 |  |  |  |  |  |
| 4 |  |  |  |  |  |
| 5 |  |  |  |  |  |
| 6 |  |  |  |  |  |
| 7 |  |  |  |  |  |
| 8 |  |  |  |  |  |
| 9 |  |  |  |  |  |
| 10 |  |  |  |  |  |
| 11 |  |  |  |  |  |
| 12 |  |  |  |  |  |
| 13 |  |  |  |  |  |
| 14 |  |  |  |  |  |
| 15 |  |  |  |  |  |
| 16 |  |  |  |  |  |
| 17 |  |  |  |  |  |
| 18 |  |  |  |  |  |
| 19 |  |  |  |  |  |
| 20 |  |  |  |  |  |

### Método de corrección

Para cada fallo o acierto con confianza 1:

1. copia la frase que determinaba la respuesta;
2. clasifica el error: concepto, lectura, comando, exceso de privilegio o confusión entre servicios;
3. escribe una regla de una línea;
4. explica por qué cada distractor es incorrecto;
5. vuelve a responder 24 horas después sin mirar.

## 18. Ficha de errores

| Fecha | Pregunta/tarea | Tu respuesta | Respuesta correcta | Tipo de error | Señal ignorada | Regla de decisión | Reprueba el |
|---|---|---|---|---|---|---|---|
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |

No registres solo “me confundí”. Ejemplo útil:

> Confundí Workforce con Workload. Señal ignorada: “human contractors”. Regla: workforce = personas; workload = software. Reprueba mañana con un escenario nuevo.

## 19. Criterios de autoevaluación

### Puntuación de escenarios

- **18-20:** rendimiento fuerte en este conjunto; revisa todavía los aciertos inseguros.
- **16-17:** objetivo personal mínimo del 80 % alcanzado para este repaso; corrige los fallos antes del control de dominio.
- **13-15:** cobertura parcial; repite las decisiones falladas y la práctica de inspección.
- **0-12:** reconstruye el mapa del dominio 1 antes de avanzar al control.

Este porcentaje es una meta del plan de estudio, no un umbral oficial de aprobación de Google.

### Evidencia práctica

Marca solo lo que puedas demostrar:

- [ ] Puedo dibujar la jerarquía sin apuntes.
- [ ] Puedo distinguir IAM de Organization Policy con un ejemplo.
- [ ] Puedo explicar API, cuota y billing como controles separados.
- [ ] Puedo distinguir Workforce de Workload Identity Federation.
- [ ] Puedo explicar VPC global y subred regional.
- [ ] Puedo elegir entre Observability, Asset Inventory y la API del servicio.
- [ ] Puedo explicar por qué un alerts-only budget no es un corte automático.
- [ ] Puedo ejecutar o interpretar los comandos de solo lectura.
- [ ] Justifiqué las 20 respuestas y corregí los distractores.

### Decisión de cierre

- Si obtuviste 16 o más y cumpliste al menos 7 de 9 evidencias, conserva las fichas y continúa con el siguiente día del plan.
- Si no, dedica 30-45 minutos al bloque más débil y vuelve a resolver solo los escenarios relacionados al día siguiente.
- No marques un tema como dominado únicamente por haber leído o descargado esta lección.

## 20. Soluciones justificadas

### 1. Correct answer: B

**Why:** A folder can group the development and production projects, and an Organization Policy applied to that folder can govern both descendants while project IAM remains independently manageable.

- **A is wrong:** Labels classify resources but do not replace project boundaries or hierarchical policy scope.
- **C is wrong:** Organization Policy is applied to Resource Manager hierarchy nodes, not to a VPC as the parent of projects.
- **D is wrong:** Billing accounts pay for linked projects; they are not parents in the resource hierarchy.

**Decision rule:** common guardrail + multiple projects = apply policy at the nearest common ancestor.

### 2. Correct answer: C

**Why:** Organization Policy constrains allowed resource configurations across a hierarchy, including administrators who otherwise have permissions to create resources.

- **A is wrong:** A budget monitors spend; it does not enforce network configuration.
- **B is wrong:** A broader role increases permissions and does not create the required guardrail.
- **D is wrong:** An alert detects a condition after or while it occurs; it is not the preventive configuration control requested.

**Decision rule:** “what may be configured” points to Organization Policy; “who may act” points to IAM.

### 3. Correct answer: B

**Why:** An allow-policy grant inherited from a folder remains effective in descendant projects. The binding must be changed at its source, or the resource/access design must be changed with an appropriate control.

- **A is wrong:** A child allow policy cannot subtract an inherited allow binding merely by omitting it.
- **C is wrong:** Disabling an API does not revoke the IAM grant and can disrupt unrelated operations.
- **D is wrong:** Another allow binding adds access; it does not override the inherited binding.

**Decision rule:** effective access includes applicable ancestor policies.

### 4. Correct answer: B

**Why:** A managed group centralizes changing membership, and a network-specific viewer role follows least privilege.

- **A is wrong:** Owner is far broader than read-only network inspection.
- **C is wrong:** Shared service account keys are insecure, hard to attribute and inappropriate for human identity.
- **D is wrong:** Network Admin permits modification and organization scope is broader than required.

**Decision rule:** frequent human membership changes = group; narrow task = narrow predefined role.

### 5. Correct answer: A

**Why:** The error explicitly states that the service API is disabled. An authorized identity must enable the API in the target project.

- **B is wrong:** Quota matters after the service can be consumed; it does not enable the API.
- **C is wrong:** Owner is excessive and still does not replace the explicit service-enablement step.
- **D is wrong:** A budget has no effect on API activation.

**Decision rule:** `SERVICE_DISABLED` = inspect Service Usage before changing IAM or quota.

### 6. Correct answer: B

**Why:** A planned, legitimate demand above quota should be handled through an advance quota-adjustment request, monitoring and a fallback plan.

- **A is wrong:** Project proliferation to evade quota review harms governance and might not solve service-level constraints.
- **C is wrong:** IAM permissions do not increase technical capacity.
- **D is wrong:** Moving a billing account does not raise the quota.

**Decision rule:** valid future demand + insufficient adjustable quota = request adjustment early.

### 7. Correct answer: B

**Why:** Product and feature availability vary by location, so the team must verify the exact managed feature and assess availability before committing.

- **A is wrong:** Not every product or feature is available in every region.
- **C is wrong:** A global VPC provides networking scope; it does not make every managed product globally available.
- **D is wrong:** Quota increases cannot create regional product availability.

**Decision rule:** location choice requires both business constraints and verified service availability.

### 8. Correct answer: A

**Why:** Google Cloud VPC networks are global resources, while subnetworks are regional. One VPC can contain regional subnets in multiple regions.

- **B is wrong:** VPC networks are not zonal.
- **C is wrong:** It reverses the actual scopes.
- **D is wrong:** Billing accounts are unrelated to subnet topology.

**Decision rule:** VPC = global; subnet = regional.

### 9. Correct answer: B

**Why:** Project and billing-account authorization are separate. Linking commonly requires suitable project-level billing management permission and permission to associate projects with the target billing account.

- **A is wrong:** Project administration does not automatically grant every billing-account permission.
- **C is wrong:** Hierarchy placement at the root is not a prerequisite for linking.
- **D is wrong:** Compute Engine API activation is unrelated to the billing link operation.

**Decision rule:** a cross-resource action requires permission on each participating resource.

### 10. Correct answer: A

**Why:** An alerts-only budget sends configured notifications and can feed programmatic workflows, but it does not automatically create a hard spending cap.

- **B is wrong:** APIs are not universally disabled at the threshold.
- **C is wrong:** Budget thresholds do not rewrite service quotas.
- **D is wrong:** Budgets do not delete project resources.

**Decision rule:** budget alert = visibility and notification, not automatic shutdown.

### 11. Correct answer: A

**Why:** Cloud Billing export writes structured billing data to BigQuery, where analysts can use SQL and combine it with other data.

- **B is wrong:** An email alert signals a threshold but does not supply the requested analytical dataset.
- **C is wrong:** CPU quota governs capacity, not cost-detail export.
- **D is wrong:** Uptime checks measure availability, not itemized billing usage.

**Decision rule:** itemized billing data + SQL = billing export to BigQuery.

### 12. Correct answer: A

**Why:** Cloud Asset Inventory supports resource discovery and metadata search across an accessible project, folder or organization scope.

- **B is wrong:** Cloud Trace analyzes request latency paths.
- **C is wrong:** Cloud DNS resolves names.
- **D is wrong:** Budgets monitor spend, not resource metadata.

**Decision rule:** cross-resource metadata inventory = Cloud Asset Inventory.

### 13. Correct answer: C

**Why:** A production decision requiring current VM state should use the service's operational API or CLI. Cloud Asset Inventory is not intended as the only real-time source for such a decision.

- **A is wrong:** A cached spreadsheet can be stale and unauthoritative.
- **B is wrong:** Asset Inventory can have freshness considerations and is designed for inventory and analysis.
- **D is wrong:** Billing export contains cost data, not authoritative VM runtime state.

**Decision rule:** immediate operational truth = service-specific API.

### 14. Correct answer: A

**Why:** Google Cloud Observability brings together operational signals such as metrics, logs and traces needed to diagnose latency.

- **B is wrong:** Folders organize resources; they do not provide telemetry.
- **C is wrong:** Asset Inventory catalogs metadata and does not replace application telemetry.
- **D is wrong:** Billing management concerns charges, not request latency diagnosis.

**Decision rule:** health, performance and telemetry = Observability.

### 15. Correct answer: A

**Why:** Workforce Identity Federation is for human users and groups from an external IdP, including contractors.

- **B is wrong:** Workload Identity Federation is for non-human workloads.
- **C is wrong:** A shared service account key is a long-lived secret and does not provide proper individual human identity.
- **D is wrong:** A firewall rule controls network traffic, not user authentication.

**Decision rule:** workforce = people.

### 16. Correct answer: B

**Why:** Workload Identity Federation gives external software a path to short-lived Google Cloud credentials without a stored service account key.

- **A is wrong:** Workforce Identity Federation targets human users.
- **C is wrong:** Owner is excessive and does not solve keyless external workload authentication.
- **D is wrong:** A budget is a cost-control signal, not an identity mechanism.

**Decision rule:** workload = software.

### 17. Correct answer: A

**Why:** For a compatible new customer, a standalone organization provides an organization resource without requiring Cloud Identity; ownership should be managed to avoid a single point of failure.

- **B is wrong:** A VPC is a networking resource, not the hierarchy root.
- **C is wrong:** A billing account is separate from the Resource Manager hierarchy.
- **D is wrong:** A bucket is a service resource inside a project.

**Decision rule:** organization governance without Cloud Identity in a supported case = evaluate standalone organization.

### 18. Correct answer: A

**Why:** The project ID identifies the project independently of its mutable display name and is accepted by common project-scoped commands and APIs.

- **B is wrong:** A folder display name identifies neither the project nor its API requests.
- **C is wrong:** A billing-account display name identifies a different resource.
- **D is wrong:** A default zone is configuration, not project identity.

**Decision rule:** do not automate against mutable display names.

### 19. Correct answer: B

**Why:** AI assistance is advisory. The engineer must validate context, permissions, scope, inheritance and official documentation, then apply the smallest authorized change.

- **A is wrong:** AI recommendations do not override policy, authorization or professional review.
- **C is wrong:** Disabling all policies is excessive and unrelated to a precise diagnosis.
- **D is wrong:** Billing-account placement does not usually resolve IAM permission errors.

**Decision rule:** use AI to accelerate investigation, never to bypass verification or least privilege.

### 20. Correct answer: A

**Why:** With the API enabled and billing active, `PERMISSION_DENIED` points first to the caller's identity, required permission and effective IAM scope.

- **B is wrong:** A budget threshold does not grant API authorization.
- **C is wrong:** CPU quota typically produces capacity or quota errors, not an IAM permission denial.
- **D is wrong:** The display name does not grant access.

**Decision rule:** read the error class: permission denial = principal + permission + effective policy.

### Clave compacta

| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|
| B | C | B | B | A | B | B | A | B | A |

| 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 |
|---|---|---|---|---|---|---|---|---|---|
| A | A | C | A | A | B | A | A | B | A |

Termina registrando tus errores y aciertos inseguros. El objetivo es construir reglas transferibles, no memorizar letras.
