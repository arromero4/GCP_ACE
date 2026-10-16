# ACE 2026 - Lección 14: Organizaciones independientes y Workforce Identity Federation

> **Fecha:** 8 de octubre de 2026  
> **Dominio de la guía ACE:** 1.1 Setting up cloud projects and accounts  
> **Tema del plan:** Organizaciones independientes y Workforce Identity Federation  
> **Práctica del plan:** Resuelve dos casos: organización nueva e identidad externa de una persona.  
> **Duración sugerida:** 75-90 minutos  
> **Caso conductor:** SiteOps Tracker, proyecto ficticio de portafolio.

Esta lección enseña y ejercita una parte del temario. Recibir el archivo no demuestra dominio: compruébalo resolviendo los dos casos, justificando el tipo de organización y de identidad, y explicando cómo separar autenticación de autorización sin mirar las soluciones.

## 1. Objetivo

Al finalizar podrás:

1. explicar para qué sirve el recurso **Organization** en la jerarquía de Google Cloud;
2. distinguir una organización independiente (**standalone organization**) de una organización asociada con Cloud Identity o Google Workspace;
3. diferenciar **Organization Owner**, **Organization Administrator** y el rol básico **Owner**;
4. identificar cuándo una organización independiente se crea automáticamente y sus límites actuales;
5. explicar Workforce Identity Federation desde cero;
6. distinguir Workforce Identity Federation de Workload Identity Federation, Cloud Identity federation e Identity Platform;
7. separar **authentication** de **authorization**;
8. diseñar el acceso mínimo para una persona externa usando un principal federado;
9. inspeccionar una organización, workforce pools y providers con Console y Google Cloud CLI;
10. resolver los dos casos del plan con evidencia, costo, limpieza y controles de seguridad.

### Evidencia de aprendizaje

Conserva al terminar:

- la ficha de decisión del caso A, “organización nueva”;
- la ficha de decisión del caso B, “identidad externa de una persona”;
- una salida de `gcloud organizations list` o la alternativa conceptual;
- un inventario de workforce pools y providers, real o simulado;
- el identificador correcto de un principal individual de workforce;
- un flujo de autenticación y autorización en seis pasos;
- una lista de riesgos y controles;
- el registro de costo y limpieza;
- la puntuación de las diez preguntas y una ficha por cada error.

## 2. Alineación con la guía oficial

La guía oficial adjunta de Associate Cloud Engineer incluye, dentro de **1.1 Setting up cloud projects and accounts**:

- **Setting up standalone organizations**.
- **Configuring Workforce Identity Federation**.

La práctica de hoy sigue exactamente esos dos puntos y la fila 14 del calendario. La administración profunda de IAM, service accounts, impersonation y Workload Identity Federation se trabajará en las lecciones del dominio 4; hoy se construye la decisión inicial correcta.

## 3. Prerrequisitos explicados desde cero

### 3.1 Jerarquía de recursos

La jerarquía típica es:

```text
Organization
└── Folder (opcional)
    └── Project
        └── Resources
```

El recurso **Organization** es la raíz administrativa. Permite aplicar políticas y roles que pueden heredarse hacia carpetas, proyectos y recursos. Un proyecto sigue siendo el contenedor donde se habilitan APIs, se aplican cuotas y se atribuye consumo.

Para SiteOps Tracker, una organización podría contener carpetas `development`, `staging` y `production`, y proyectos separados para aplicación, datos y seguridad. La organización no sustituye a los proyectos; los gobierna desde arriba.

### 3.2 Identidad, autenticación y autorización

Son tres preguntas diferentes:

- **Identity:** ¿quién afirma ser la persona?
- **Authentication:** ¿cómo demuestra esa identidad?
- **Authorization:** después de autenticar, ¿qué puede hacer y sobre qué recurso?

En Workforce Identity Federation:

1. el proveedor de identidad externo autentica a la persona;
2. Google valida la credencial y entrega una credencial temporal;
3. las políticas IAM de Google Cloud autorizan o rechazan operaciones.

Un inicio de sesión correcto no implica acceso a ningún proyecto. La persona necesita un rol IAM en el alcance adecuado.

### 3.3 Proveedor de identidad externo

Un **Identity Provider (IdP)** administra identidades y credenciales. Workforce Identity Federation admite proveedores compatibles con **OpenID Connect (OIDC)** o **SAML 2.0**. Ejemplos comunes son Microsoft Entra ID y Okta, pero la decisión depende del IdP real y de sus capacidades.

El IdP emite afirmaciones o **claims** como identificador del usuario, grupos, tenant o departamento. Google Cloud mapea esas claims a atributos que puede usar para autenticar y autorizar.

### 3.4 IAM mínimo

Para configurar Workforce Identity Federation, la documentación recomienda **IAM Workforce Pool Admin** (`roles/iam.workforcePoolAdmin`) en la organización. Para conceder un rol sobre un proyecto a un principal federado también hacen falta permisos para modificar la allow policy del proyecto.

La persona externa no necesita el rol de administrador del pool para usar Google Cloud. Necesita únicamente el rol de recurso que cubra su tarea, por ejemplo acceso de solo lectura a inventario o logs.

No solicites `Owner` para evitar analizar permisos. Usa identidades administrativas separadas y roles de menor privilegio.

### 3.5 Herramientas

Para la inspección opcional necesitas:

- una cuenta autorizada;
- Google Cloud CLI actual o Cloud Shell;
- visibilidad de una organización;
- para la parte federada real, un pool y provider ya configurados por un administrador.

No necesitas crear una organización, comprar un dominio ni configurar un IdP para completar la alternativa conceptual.

## 4. Modelo mental: edificio, recepción y gafete temporal

Imagina que SiteOps Tracker ocupa un edificio:

- **Organization** es el edificio completo y sus reglas generales.
- **Folders** son pisos o áreas.
- **Projects** son oficinas con presupuestos, APIs y recursos.
- **Cloud Identity o Google Workspace** es un directorio corporativo propio del edificio.
- **External IdP** es la empresa que ya conoce y autentica a la persona externa.
- **Workforce identity pool** es la recepción que agrupa a una población externa confiable.
- **Provider** es el acuerdo técnico con el sistema de credenciales externo.
- **Attribute mapping** traduce los datos del gafete externo al formato de Google Cloud.
- **IAM allow policy** indica a qué oficina puede entrar y qué puede hacer.
- **Short-lived access token** es un gafete temporal, no una llave permanente.

La recepción puede confirmar la identidad sin autorizar ninguna oficina. Autenticar y autorizar son controles distintos.

## 5. Organización de Google Cloud paso a paso

### 5.1 Qué aporta el recurso Organization

Una organización:

- es el nodo raíz de carpetas y proyectos;
- permite centralizar IAM y Organization Policy;
- conserva la propiedad institucional de los proyectos;
- facilita separar ambientes con folders y projects;
- ofrece una vista de gobierno común.

La organización no concede automáticamente capacidad de crear folders, projects o recursos. Cada operación requiere permisos específicos.

### 5.2 Organización asociada con Cloud Identity o Google Workspace

En el modelo tradicional, una cuenta de Cloud Identity o Google Workspace se asocia con un recurso Organization. Un superadministrador del directorio ayuda a iniciar la configuración y delega la administración de Google Cloud.

Este modelo es apropiado cuando una empresa:

- posee y verifica un dominio;
- administra cuentas de usuarios y grupos;
- necesita servicios de Google adicionales;
- quiere controlar el ciclo de vida de identidades administradas.

Cloud Identity y Google Workspace comparten una plataforma de identidad. Google Workspace añade servicios de productividad; Cloud Identity se enfoca en identidad y administración de dispositivos según su edición.

### 5.3 Organización independiente

Una **standalone organization** no requiere Cloud Identity. Según la documentación vigente el 8 de octubre de 2026:

- está disponible para nuevos clientes de Free Trial de Google Cloud;
- se crea automáticamente cuando el usuario nuevo inicia sesión y acepta los términos;
- no está disponible para cuentas de Google Cloud existentes;
- solo se crea una por cuenta de usuario, aunque una persona puede ser invitada a administrar varias;
- el nombre predeterminado deriva del nombre de usuario y termina en `-org`;
- no se debe usar un dominio como nombre, porque los dominios se reservan para organizaciones de Cloud Identity o Google Workspace.

No incluyas datos personales ni información de seguridad en nombres de organización o recursos.

### 5.4 Roles iniciales de una organización independiente

El creador recibe:

- **Organization Owner** (`roles/cloudowner.admin`);
- **Organization Administrator** (`roles/resourcemanager.organizationAdmin`).

Aunque ambos llegan inicialmente a la misma persona, cumplen propósitos distintos.

| Rol | Modelo | Responsabilidad principal | No implica automáticamente |
|---|---|---|---|
| Organization Owner | Gestionado fuera de IAM para la organización independiente | Propietarios, recuperación y ciclo de vida de la organización | Crear todos los recursos o administrar cada servicio |
| Organization Administrator | Rol IAM | Allow/deny policies, concesión de roles y vista de jerarquía | Crear folders o projects sin roles adicionales |
| Owner (`roles/owner`) | Rol IAM básico sobre un recurso | Amplios permisos sobre el recurso donde se concede | Ser Organization Owner o controlar el ciclo de vida de la organización |

**Trampa de examen:** “Organization Owner” de una standalone organization no es lo mismo que el rol básico “Owner” de un proyecto.

### 5.5 Continuidad de administración

Una organización independiente debe conservar al menos un Google Account activo como Organization Owner. Un service account no puede recibir esa invitación. La documentación permite también propietarios que sean principals de un workforce pool, pero no se debe eliminar el último Google Account activo.

Buenas decisiones iniciales:

1. tener más de un propietario humano autorizado;
2. separar recuperación/ciclo de vida de administración diaria;
3. registrar Essential Contacts;
4. usar MFA o controles equivalentes;
5. revisar políticas base antes de desplegar;
6. evitar el uso diario de identidades altamente privilegiadas.

### 5.6 Políticas base de seguridad

Google Cloud aplica un conjunto de **security baseline constraints** a organizaciones nuevas. Entre los ejemplos documentados están:

- deshabilitar creación de claves persistentes de service accounts;
- deshabilitar carga de claves externas de service accounts;
- impedir la concesión automática del rol Editor a service accounts predeterminadas;
- exigir uniform bucket-level access para Cloud Storage.

No borres estas políticas solo porque bloquean un laboratorio. Primero entiende el control y usa una alternativa segura. En la práctica de hoy solo se listan.

### 5.7 Una organización o varias

Para la mayoría de departamentos o ambientes, usa una sola organización y separa con folders y projects. Múltiples organizaciones pueden ser correctas cuando se exige aislamiento administrativo independiente, pero agregan costos operativos:

- no existe visibilidad central por defecto;
- las políticas comunes deben replicarse;
- mover folders entre organizaciones no está soportado;
- las identidades y la administración se vuelven más complejas.

No crees una organización por sede de SiteOps Tracker solo porque existen varias ubicaciones físicas.

## 6. Workforce Identity Federation paso a paso

### 6.1 Qué problema resuelve

Workforce Identity Federation permite que empleados, partners o contractors administrados en un IdP externo accedan a recursos de Google Cloud mediante SSO. No es necesario sincronizar cada identidad a una cuenta administrada de Google.

Es una estrategia **sync-less**:

- la identidad permanece en el IdP externo;
- Google Cloud confía en claims verificadas;
- Security Token Service intercambia la credencial externa por una credencial temporal;
- IAM decide el acceso.

### 6.2 Componentes

| Componente | Función | Alcance |
|---|---|---|
| External IdP | Autentica a la persona y emite una credencial firmada | Sistema externo |
| Workforce identity pool | Agrupa identidades humanas federadas | Organización |
| Provider | Define la relación con un IdP OIDC o SAML | Dentro del pool |
| Attribute mapping | Traduce claims externas a atributos de Google | Provider |
| Attribute condition | Restringe qué credenciales se aceptan | Provider |
| IAM allow policy | Concede roles a un principal o principal set | Organization, folder, project o recurso admitido |
| STS | Intercambia credencial externa por token de Google de corta duración | Servicio de token |
| Console (federated) / gcloud | Punto de acceso de la persona | Solo productos compatibles |

### 6.3 Attribute mapping

El atributo obligatorio es:

- `google.subject`: identificador único de la persona.

Debe mapearse a una claim estable e inmutable. Si se usa un correo que puede cambiar, la persona podría perder acceso al cambiar de dirección. Un identificador interno estable del IdP suele ser mejor.

Otros atributos incluyen:

- `google.groups`: grupos relevantes;
- `google.display_name`: nombre para la interfaz;
- `google.email`: correo para integraciones compatibles, no para decisiones IAM;
- `attribute.KEY`: atributos personalizados como departamento o tipo de contrato.

No confíes en cualquier claim por conveniencia. Revisa quién puede modificarla en el IdP y si representa una propiedad de seguridad estable.

### 6.4 Attribute condition

Una **attribute condition** es una expresión CEL que restringe qué identidades puede aceptar Google Cloud. Es especialmente importante con IdPs multi-tenant que comparten issuer URI.

Ejemplo conceptual:

```text
assertion.tenant_id == "TENANT_ESPERADO"
```

La condición evita aceptar tokens válidos del mismo proveedor global pero emitidos para otro tenant. El nombre real de la claim depende del IdP.

### 6.5 Principal individual y principal sets

Una persona concreta se representa así:

```text
principal://iam.googleapis.com/locations/global/workforcePools/POOL_ID/subject/SUBJECT_ATTRIBUTE_VALUE
```

Conjuntos comunes:

```text
principalSet://iam.googleapis.com/locations/global/workforcePools/POOL_ID/group/GROUP_ID
principalSet://iam.googleapis.com/locations/global/workforcePools/POOL_ID/attribute.department/operations
principalSet://iam.googleapis.com/locations/global/workforcePools/POOL_ID/*
```

Para el caso de una sola persona externa y acceso excepcional, el principal individual limita el alcance. Para una función recurrente con varias personas, un grupo gobernado en el IdP suele ser más mantenible. Evita conceder acceso a todo el pool salvo que el requisito lo justifique.

### 6.6 Autenticación no es autorización

El flujo correcto es:

1. la persona abre el console (federated) o inicia `gcloud auth login` con una configuración autorizada;
2. el IdP autentica y emite una credencial firmada;
3. Google valida issuer, audience, firma, mapping y condition;
4. STS entrega una credencial temporal;
5. la persona llama a una API de Google Cloud;
6. IAM evalúa si el principal federado tiene el rol requerido sobre el recurso.

Si el paso 4 funciona y el paso 6 falla, el problema suele ser autorización, no autenticación.

### 6.7 Console federada

Las personas federadas usan el **Google Cloud Workforce Identity Federation console**, o console (federated). Muestra únicamente productos compatibles, y algunos productos pueden tener limitaciones de interfaz. No asumas que la Console normal y la federada tienen idéntica cobertura.

### 6.8 Seguridad para partners

Federar un partner crea una relación de confianza. Tu organización depende de que su IdP:

- aplique MFA según tus requisitos;
- emita claims correctas;
- retire usuarios oportunamente;
- gobierne la membresía de grupos;
- proteja sus claves y configuración.

Para administradores altamente privilegiados y cuentas de emergencia, las prácticas recomendadas favorecen Cloud Identity con controles fuertes, incluso si el resto de usuarios usa Workforce Identity Federation.

## 7. No confundir Workforce y Workload Identity Federation

| Pregunta | Workforce Identity Federation | Workload Identity Federation |
|---|---|---|
| ¿Quién se autentica? | Persona: empleado, contractor o partner | Aplicación, pipeline o workload |
| Ejemplo | Auditor externo abre Console | GitHub Actions despliega una imagen |
| Pool | Workforce identity pool | Workload identity pool |
| Principal | Humano federado | Workload federado |
| Interfaz típica | Console (federated), gcloud, APIs | SDK, API, pipeline, service account impersonation |
| Objetivo | SSO y acceso humano sin sincronizar cuentas | Evitar claves estáticas de service accounts para cargas externas |

La palabra “externo” no decide por sí sola. Pregunta si el actor es una persona o una carga.

## 8. Caso conductor: SiteOps Tracker

SiteOps Tracker es un proyecto ficticio para auditar infraestructura en sedes, registrar activos, hallazgos, estados, responsables, evidencias e historial. Su stack incluye React + TypeScript, Node.js + TypeScript, PostgreSQL y servicios de Google Cloud.

### Caso A - organización nueva

Una microempresa empieza una prueba de Google Cloud con una cuenta nueva. No tiene Cloud Identity ni un dominio preparado. Quiere separar desarrollo y producción de SiteOps Tracker y evitar que una sola persona sea el punto único de recuperación.

Decisión razonada:

1. si cumple las condiciones actuales de nuevo cliente de Free Trial, Google Cloud crea una standalone organization automáticamente;
2. confirmar el ID y nombre, sin datos sensibles;
3. conservar al menos un Google Account activo como Organization Owner;
4. agregar un segundo propietario humano autorizado;
5. separar Organization Owner de Organization Administrator cuando sea posible;
6. revisar security baseline constraints;
7. crear una jerarquía con folders/projects después de conceder los roles específicos;
8. si la empresa ya tiene un dominio y necesita directorio administrado, evaluar Cloud Identity o Google Workspace en vez de elegir standalone por inercia.

No crear una organización separada para cada sede. Las sedes son un concepto de la aplicación y, si hace falta, de red o ubicación; no necesariamente límites de administración raíz.

### Caso B - identidad externa de una persona

Una auditora externa debe consultar, durante dos semanas, el inventario del proyecto de desarrollo. Su empresa usa un IdP OIDC. No debe recibir una cuenta local de servicio, una clave persistente ni acceso a producción.

Diseño razonado:

1. usar Workforce Identity Federation porque el actor es humano;
2. usar un workforce pool de contractors o auditors en la organización;
3. configurar un provider OIDC para el IdP confiable;
4. mapear un identificador inmutable a `google.subject`;
5. restringir el tenant con una attribute condition;
6. representar a la auditora con un principal individual;
7. conceder únicamente los roles de lectura necesarios en el proyecto de desarrollo;
8. no conceder acceso a la carpeta o a toda la organización;
9. proporcionar console (federated) o configuración de login de gcloud;
10. retirar la asignación del IdP y el binding de IAM al finalizar.

Un ejemplo ficticio de principal es:

```text
principal://iam.googleapis.com/locations/global/workforcePools/siteops-auditors/subject/auditor-042
```

No uses una dirección real ni un identificador personal en esta práctica.

## 9. Glosario bilingüe

| English term | Término en español | Definición útil para ACE |
|---|---|---|
| organization resource | recurso de organización | Nodo raíz de la jerarquía de Google Cloud. |
| standalone organization | organización independiente | Organización sin requisito de Cloud Identity para nuevos clientes admitidos. |
| Organization Owner | propietario de la organización | Rol de ciclo de vida y recuperación de una standalone organization, gestionado fuera de IAM. |
| Organization Administrator | administrador de la organización | Rol IAM para políticas y concesión de acceso en la organización. |
| resource hierarchy | jerarquía de recursos | Relación Organization → Folder → Project → Resource. |
| identity | identidad | Representación de una persona o workload. |
| authentication | autenticación | Comprobación de quién es el principal. |
| authorization | autorización | Determinación de lo que puede hacer sobre un recurso. |
| workforce | personal | Usuarios humanos: empleados, partners o contractors. |
| workload | carga de trabajo | Aplicación, proceso o sistema no humano. |
| federation | federación | Confianza entre sistemas de identidad sin copiar necesariamente las cuentas. |
| identity provider (IdP) | proveedor de identidad | Sistema externo que autentica y emite claims. |
| workforce identity pool | pool de identidades de personal | Contenedor organizacional para identidades humanas federadas. |
| provider | proveedor | Configuración que relaciona el pool con un IdP. |
| claim / assertion | claim / afirmación | Dato emitido por el IdP acerca de la identidad. |
| attribute mapping | mapeo de atributos | Traducción de claims del IdP a atributos de Google. |
| attribute condition | condición de atributo | Regla CEL que limita las identidades aceptadas. |
| subject | sujeto | Identificador único de una persona federada. |
| principal | principal | Identidad que puede recibir permisos IAM. |
| principal set | conjunto de principals | Grupo definido por pool, grupo o atributo. |
| short-lived credential | credencial de corta duración | Token temporal emitido después del intercambio. |
| single sign-on (SSO) | inicio de sesión único | Autenticación mediante el IdP existente. |
| baseline constraint | restricción base | Organization Policy aplicada inicialmente para mejorar seguridad. |

## 10. Decisiones de servicio y por qué no elegir alternativas

| Requisito | Elegir | Por qué | Por qué no las alternativas |
|---|---|---|---|
| Nuevo cliente de Free Trial sin Cloud Identity que necesita raíz de gobierno | Standalone organization | Se crea automáticamente para clientes nuevos admitidos | Un proyecto sin organización pierde gobierno raíz; crear Cloud Identity solo por costumbre puede añadir administración innecesaria. |
| Empresa con dominio, usuarios administrados y servicios de Google | Cloud Identity o Google Workspace organization | Directorio y ciclo de vida administrados | Standalone no aporta el mismo directorio administrado ni sustituye las necesidades de Workspace. |
| Departamentos con políticas comunes | Una organización con folders/projects | Herencia y administración central | Varias organizaciones obligan a replicar políticas y reducen visibilidad central. |
| Persona externa que usa IdP corporativo | Workforce Identity Federation | SSO humano, sync-less y tokens temporales | Workload federation es para procesos; service account keys no representan a una persona. |
| Pipeline externo de CI/CD | Workload Identity Federation | El actor es una carga no humana | Workforce federation está diseñada para personas. |
| Usuarios que necesitan cuenta administrada y servicios amplios de Google | Cloud Identity federation | Existe una cuenta administrada correspondiente | Workforce federation solo da acceso a productos compatibles y no crea una cuenta de Google administrada. |
| Clientes finales que inician sesión en SiteOps Tracker | Identity Platform u otra solución CIAM | Son usuarios de la aplicación | Workforce federation es para personal que accede a recursos de Google Cloud. |
| Una persona externa, acceso excepcional | Principal individual por `subject` | Reduce el conjunto autorizado | Todo el pool es excesivo; un grupo es mejor si la función es recurrente y colectiva. |
| Equipo externo estable | Principal set por grupo gobernado | Alta/baja centralizada en el IdP | Bindings individuales escalan mal; todo el pool concede demasiado. |
| Administrador de emergencia | Identidad Cloud Identity protegida | Reduce dependencia total del IdP federado | Si el IdP falla o se compromete, una cuenta exclusivamente federada puede no servir para recuperación. |

### Regla de examen

1. ¿El actor es humano o workload?
2. ¿Necesita una cuenta administrada de Google o acceso sync-less?
3. ¿Cuál es el alcance mínimo: persona, grupo, atributo o pool?
4. ¿Dónde autentica y dónde se autoriza?
5. ¿Qué ocurre si el IdP falla o emite una claim incorrecta?
6. ¿Existe una identidad de emergencia fuera de esa dependencia?

## 11. Práctica guiada: resolver los dos casos

### 11.1 Alcance y seguridad

La práctica principal es de lectura y diseño. No crees ni elimines organizaciones, pools, providers o IAM bindings. Una configuración real requiere un IdP controlado, revisión de seguridad y autoridad de organización.

Los comandos y opciones se verificaron el 8 de octubre de 2026 contra documentación oficial vigente.

### 11.2 Parte A - preflight

```bash
gcloud auth list
gcloud version
gcloud organizations list
```

Registra:

```text
Cuenta activa: ______________________________________________________
Organizaciones visibles: ___________________________________________
¿La cuenta es humana o service account?: ___________________________
```

Una lista vacía no demuestra que Google Cloud no tenga organizaciones. Puede indicar cuenta equivocada, falta de acceso o, para ciertos service accounts, una lista incompleta.

### 11.3 Parte B - Console: caso A

Si tienes acceso autorizado a una organización:

1. Abre **IAM & Admin > Manage Resources** y confirma la raíz de la jerarquía.
2. Abre **My organizations** y registra el Organization ID.
3. Si es una standalone organization, abre **Organization details**.
4. Identifica Organization Owners y Organization Administrators sin modificar nada.
5. Abre **Organization Policies** y localiza políticas base relacionadas con claves de service accounts y uniform bucket-level access.
6. No cambies enforcement.
7. Dibuja la jerarquía propuesta de SiteOps Tracker con `development` y `production`.

Ficha de caso A:

| Pregunta | Respuesta y evidencia |
|---|---|
| ¿Es cuenta nueva admitida para standalone? |  |
| ¿Hay Cloud Identity/Workspace y dominio administrado? |  |
| ¿Cuál es el Organization ID? |  |
| ¿Quién controla ciclo de vida/recuperación? |  |
| ¿Quién administra IAM? |  |
| ¿Existe un segundo propietario humano? |  |
| ¿Qué baseline constraints están enforced? |  |
| ¿Cómo separar development y production? |  |
| Decisión final | standalone / Cloud Identity-Workspace / falta información |

### 11.4 Parte C - CLI: inspeccionar organización y políticas

Sustituye el identificador solo después de verificarlo:

```bash
export ORGANIZATION_ID="YOUR_ORGANIZATION_ID"

gcloud organizations describe "$ORGANIZATION_ID"

gcloud resource-manager org-policies list \
  --organization="$ORGANIZATION_ID"
```

Estas consultas no crean ni cambian recursos. Busca los nombres de restricciones, su estado y el nivel donde están aplicadas. No uses `gcloud org-policies delete` en esta lección.

Si no tienes organización, anota “sin acceso” y continúa con la alternativa conceptual.

### 11.5 Parte D - diseñar la jerarquía del caso A

Completa:

```text
Organization: siteops-demo-org (nombre ficticio, sin dominio ni PII)
├── Folder: development
│   ├── Project: app-dev
│   └── Project: data-dev
└── Folder: production
    ├── Project: app-prod
    └── Project: data-prod

Organization Owner(s): _____________________________________________
Organization Administrator(s): _____________________________________
Folder Creator / Project Creator: __________________________________
Essential Contacts: ________________________________________________
Razón para no usar varias organizaciones: __________________________
```

No asignes todos esos roles a la misma identidad sin justificarlo. El diagrama es una propuesta, no una orden de despliegue.

### 11.6 Parte E - Console: caso B

Si tu organización ya tiene Workforce Identity Federation configurado:

1. Ve a **IAM & Admin > Workforce Identity Federation**.
2. Localiza el workforce pool autorizado.
3. Abre el provider y registra protocolo, issuer, estado, mapping y condition sin copiar secretos.
4. Comprueba que `google.subject` deriva de una claim estable.
5. Comprueba que una condition restringe el tenant cuando aplique.
6. En el proyecto de desarrollo, abre **IAM** y busca el principal federado o principal set.
7. Verifica el rol y el alcance; no edites la policy.

Tabla de evidencia:

| Elemento | Valor no sensible | Control esperado |
|---|---|---|
| Pool ID |  | Único, estable y descriptivo |
| Provider ID |  | Relación con IdP aprobado |
| Protocol | OIDC / SAML | Firma y flujo documentados |
| `google.subject` | Claim, no valor personal | Única e inmutable |
| Attribute condition | Descripción, no secreto | Tenant/población autorizada |
| Principal | Formato | Persona exacta o grupo justificado |
| IAM role |  | Menor privilegio |
| Resource scope |  | Solo proyecto de desarrollo |
| Offboarding |  | Fecha, owner y verificación |

### 11.7 Parte F - CLI: inspeccionar pools y providers

Lista los pools de la organización:

```bash
gcloud iam workforce-pools list \
  --organization="$ORGANIZATION_ID" \
  --location="global"
```

Si existe un pool autorizado:

```bash
export WORKFORCE_POOL_ID="YOUR_WORKFORCE_POOL_ID"

gcloud iam workforce-pools describe "$WORKFORCE_POOL_ID" \
  --location="global"

gcloud iam workforce-pools providers list \
  --workforce-pool="$WORKFORCE_POOL_ID" \
  --location="global"
```

Si existe un provider autorizado:

```bash
export WORKFORCE_PROVIDER_ID="YOUR_PROVIDER_ID"

gcloud iam workforce-pools providers describe "$WORKFORCE_PROVIDER_ID" \
  --workforce-pool="$WORKFORCE_POOL_ID" \
  --location="global"
```

No pegues tokens, claves, certificados completos ni assertions en la ficha.

### 11.8 Parte G - principal de una persona externa

Para el caso ficticio, construye el principal sin ejecutarlo:

```bash
export EXTERNAL_SUBJECT="auditor-042"

export WORKFORCE_PRINCIPAL="principal://iam.googleapis.com/locations/global/workforcePools/${WORKFORCE_POOL_ID}/subject/${EXTERNAL_SUBJECT}"

printf '%s\n' "$WORKFORCE_PRINCIPAL"
```

Una propuesta de binding de proyecto tiene esta estructura:

```bash
# DISEÑO SOLAMENTE: no ejecutar en esta lección.
gcloud projects add-iam-policy-binding "SITEOPS_DEV_PROJECT_ID" \
  --role="roles/cloudasset.viewer" \
  --member="$WORKFORCE_PRINCIPAL"
```

Cloud Asset Inventory también requiere `serviceusage.services.use`; en un diseño real se concedería el rol mínimo que lo incluya, como `roles/serviceusage.serviceUsageConsumer`, en el mismo alcance si es necesario. Un administrador debe revisar y aprobar ambos bindings.

### 11.9 Parte H - login federado opcional, solo si ya está configurado

No ejecutes este bloque sin IDs proporcionados por el administrador y un provider real aprobado. El comando crea un archivo local de configuración; no crea un pool.

```bash
export LOGIN_CONFIG_FILE="$PWD/siteops-workforce-login.json"

gcloud iam workforce-pools create-login-config \
  "locations/global/workforcePools/${WORKFORCE_POOL_ID}/providers/${WORKFORCE_PROVIDER_ID}" \
  --output-file="$LOGIN_CONFIG_FILE"

gcloud auth login --login-config="$LOGIN_CONFIG_FILE"
gcloud auth list
```

El login puede cambiar la cuenta activa de la configuración de gcloud. Hazlo únicamente en un entorno o configuración dedicada. Protege el archivo contra modificación: aunque no contiene una contraseña, una persona maliciosa que cambie endpoints podría interceptar credenciales. No lo subas a Git.

### 11.10 Parte I - resolver el caso B

Completa:

```text
Actor: persona / workload
IdP y protocolo: ____________________________________________________
Pool: _______________________________________________________________
Provider: ___________________________________________________________
Claim para google.subject: _________________________________________
Por qué es inmutable: ______________________________________________
Attribute condition: _______________________________________________
Principal exacto: __________________________________________________
Rol(es) mínimo(s): __________________________________________________
Recurso y alcance: __________________________________________________
Método de acceso: console (federated) / gcloud / API
Control de MFA en IdP: _____________________________________________
Offboarding: ________________________________________________________
Identidad de emergencia: ___________________________________________
```

### 11.11 Alternativa conceptual completa sin cuenta, crédito, organización o IdP

Resuelve con estos datos ficticios.

#### Dataset del caso A

- Cuenta: nuevo cliente elegible para Free Trial.
- Cloud Identity/Workspace: no existe.
- Dominio verificado: no.
- Usuarios iniciales: fundadora y responsable técnico.
- Ambientes: development y production.
- Requisito: ningún punto único de recuperación.
- Políticas base: creación/carga de claves de service accounts deshabilitadas; uniform bucket-level access enforced.

Tareas:

1. Elige el tipo de organización.
2. Asigna Organization Owner y Organization Administrator con separación de funciones.
3. Dibuja folders y projects.
4. Explica por qué no borrarías las políticas base.
5. Indica qué cambiaría si ya existiera un dominio y un directorio corporativo administrado.

#### Dataset del caso B

- Actor: una auditora externa humana.
- IdP: OIDC, tenant `partner-tenant-77`.
- Claim estable: `sub=7f3a-042`.
- Correo: puede cambiar.
- Tarea: leer inventario del proyecto `app-dev` durante dos semanas.
- Acceso a producción: prohibido.
- MFA: exigida por el partner.
- Pool ficticio: `siteops-auditors`.
- Provider ficticio: `partner-oidc`.

Tareas:

1. Elige Workforce o Workload Identity Federation.
2. Define `google.subject`.
3. Escribe una condition conceptual para el tenant.
4. Construye el principal individual.
5. Elige el alcance y roles mínimos.
6. Describe autenticación y autorización por separado.
7. Define offboarding y evidencia de cierre.

Resultado conceptual esperado:

- Caso A: standalone organization automática; dos propietarios humanos autorizados, al menos uno con Google Account activo; administración diaria separada; folders `development` y `production`; baseline constraints preservadas.
- Si existe directorio corporativo y dominio administrado, evaluar Cloud Identity o Google Workspace.
- Caso B: Workforce Identity Federation, no Workload; `google.subject=assertion.sub`; condition que restrinja `partner-tenant-77`; principal `principal://iam.googleapis.com/locations/global/workforcePools/siteops-auditors/subject/7f3a-042`; roles de inventario solo en `app-dev`; retiro del binding y de la asignación del IdP al concluir.

## 12. Resultado esperado

### Si realizaste la inspección

Debes poder mostrar:

- identidad activa y organizaciones visibles;
- Organization ID y tipo de organización, cuando sea observable;
- separación entre Owner y Administrator;
- baseline constraints listadas, no modificadas;
- pools y providers listados o una justificación de ausencia de acceso;
- principal individual correctamente formado;
- análisis del alcance y roles mínimos;
- confirmación de que no se modificó IAM ni se crearon recursos.

### Si realizaste la alternativa conceptual

Debes entregar las once decisiones de los dos datasets, el flujo de seis pasos y una matriz de riesgos. La alternativa cubre íntegramente el razonamiento de la lección, aunque no sustituye la experiencia de una integración real con un IdP.

## 13. Solución de problemas

| Síntoma | Causa probable | Diagnóstico | Acción segura |
|---|---|---|---|
| No aparece una standalone organization | Cuenta existente o cliente no elegible | Compara la cuenta con los requisitos actuales | No intentes forzarla; evalúa organización con Cloud Identity/Workspace o usa la simulación. |
| `gcloud organizations list` no muestra la organización | Cuenta activa equivocada, falta de acceso o service account | Ejecuta `gcloud auth list`; revisa la cuenta | Cambia solo a una identidad autorizada; no concluyas que la organización no existe. |
| Organization Admin no puede crear folders | Ese rol no incluye la acción | Revisa permisos y rol de Folder Creator | Concede el rol específico al principal correcto; no uses Owner por comodidad. |
| Se confunden Organization Owner y `roles/owner` | Nombres parecidos | Revisa dónde se gestiona cada rol | Documenta ciclo de vida fuera de IAM frente a permisos IAM sobre recursos. |
| `PERMISSION_DENIED` al listar pools | Falta acceso IAM o organización incorrecta | Verifica Organization ID y rol | Solicita visibilidad mínima; usa la alternativa conceptual si no corresponde. |
| El pool no aparece | No existe, está en otra organización o fue eliminado | Lista con organización y ubicación correctas | No crees otro pool sin diseño y aprobación. |
| El provider rechaza el login | Issuer, audience, redirect URI, firma o estado | Compara IdP y provider | Corrige la relación en un entorno de prueba; no deshabilites validaciones. |
| Error de attribute mapping | Falta `google.subject` o la expresión no coincide | Revisa claims no sensibles y mapping | Usa una claim única y estable; no publiques tokens completos. |
| Usuario autentica pero recibe `PERMISSION_DENIED` | IAM no autoriza al principal mapeado | Compara subject real, principal y resource scope | Corrige el binding mínimo; no amplíes a todo el pool sin necesidad. |
| Usuario de otro tenant puede intentar autenticarse | IdP multi-tenant sin condition adecuada | Revisa issuer y tenant claim | Añade una attribute condition específica y probada. |
| El producto no aparece en console (federated) | Producto o función no compatible | Consulta la matriz oficial | Usa una interfaz compatible o arquitectura de identidad distinta; no asumas paridad. |
| Más de 400 grupos causan fallo de sign-in | Demasiadas membresías mapeadas | Revisa `google.groups` | Envía y mapea solo grupos de Google Cloud necesarios. |
| Archivo de login fue modificado | Riesgo de endpoints maliciosos | Compara con config aprobada | Descarta, regenera desde IDs confiables y restringe permisos del archivo. |
| Partner desactiva a la persona pero conserva acceso | Token vigente o offboarding incompleto | Revisa sesiones, IdP e IAM | Revoca asignación y binding; comprueba acceso después del cambio. |

### Diagnóstico en orden

1. actor humano o workload;
2. cuenta y organización;
3. pool y provider;
4. issuer, audience, firma y redirect;
5. attribute mapping;
6. attribute condition;
7. autenticación y token temporal;
8. principal resultante;
9. IAM role y resource scope;
10. soporte del producto y offboarding.

## 14. Impacto en costos y limpieza

### 14.1 Costos

- El uso de la API de IAM se ofrece sin cargo.
- Workforce Identity Federation se ofrece sin cargo.
- El detailed audit logging de Workforce Identity Federation usa Cloud Logging y puede generar cargos según volumen, almacenamiento y retención.
- El recurso Organization y la inspección de políticas no crean por sí solos cargas de cómputo.
- Los proyectos y servicios que se desplieguen debajo de la organización conservan sus precios normales.
- Una Free Trial y sus créditos tienen condiciones propias; no son necesarios para la alternativa conceptual.

Verifica las páginas oficiales de precios antes de habilitar logging detallado o desplegar recursos. No uses una respuesta generativa como fuente de precios.

### 14.2 Limpieza

La práctica principal es de solo lectura. Debes terminar con:

- organizaciones creadas: ninguna;
- pools/providers creados: ninguno;
- IAM bindings modificados: ninguno;
- políticas eliminadas: ninguna;
- recursos facturables creados: ninguno.

Si generaste `siteops-workforce-login.json` en la parte opcional:

1. cierra la sesión federada cuando corresponda;
2. verifica la ruta exacta del archivo;
3. elimina la copia local cuando ya no sea necesaria;
4. confirma que nunca se añadió a Git;
5. no borres el pool o provider compartido.

Limpia variables de esta sesión:

```bash
unset ORGANIZATION_ID WORKFORCE_POOL_ID WORKFORCE_PROVIDER_ID
unset EXTERNAL_SUBJECT WORKFORCE_PRINCIPAL LOGIN_CONFIG_FILE
```

Registro:

```text
Organizaciones creadas: ninguna
Pools/providers creados: ninguno
Bindings modificados: ninguno
Archivos locales creados: __________________________________________
Archivos locales eliminados o protegidos: __________________________
Costo observado/estimado: __________________________________________
Verificación final: _________________________________________________
```

## 15. Repaso activo espaciado

Responde sin mirar las lecciones anteriores. Después comprueba y corrige.

### Día hábil anterior - Lección 13: Observability, Asset Inventory y Cloud Assist

1. ¿Qué pregunta responde inventario y cuál responde telemetría?
2. ¿Cuándo debes usar la API específica del servicio en vez de Asset Inventory?
3. Distingue metric, log y trace con un ejemplo de SiteOps Tracker.
4. ¿Por qué una respuesta de Cloud Assist debe validarse?

### Dos días hábiles antes - Lección 12: red inicial, regiones y zonas

1. ¿Cuál es el alcance de una VPC y cuál el de una subnet?
2. ¿Por qué una VPC global no hace global a una aplicación?
3. Distingue product availability, capacity y quota.
4. ¿Qué relación tiene la organización con una VPC creada en un proyecto?

### Tres días hábiles antes - Lección 11: presupuestos y exportación

1. ¿Por qué un budget no detiene automáticamente el gasto?
2. ¿Qué permisos y alcances revisarías antes de configurar un presupuesto?
3. ¿Por qué BigQuery es útil para una exportación de facturación?
4. ¿Concederías acceso de facturación a una auditora de infraestructura por defecto? Justifica.

### Hace 7 días - Lección 9: APIs, cuotas y aumentos

1. Distingue API habilitada, IAM permission y quota disponible.
2. ¿Quién debe autorizar la habilitación de una API?
3. ¿Por qué un incremento de cuota no concede permisos?
4. ¿Qué APIs requiere una integración de identidad y cómo las verificarías antes de configurar?

### Hace 21 días - línea base previa al calendario

El plan comenzó el 21 de septiembre; el 17 de septiembre no había lección programada. Usa esta recuperación de línea base:

1. Define identity, authentication y authorization.
2. Explica por qué una credencial humana no debe reutilizarse como identidad de una aplicación.
3. Da una evidencia práctica más fuerte que “leí el tema”.

### Refuerzo breve de confusiones acumulativas

- **Organization Owner ≠ Project Owner.**
- **Organization Administrator ≠ Folder Creator ni Project Creator.**
- **Authentication success ≠ resource access.**
- **Workforce = personas; Workload = aplicaciones o procesos.**
- **API enabled ≠ permission granted ≠ quota available.**
- **Federation ≠ automatic authorization.**
- **Un correo mutable no siempre es un buen `google.subject`.**
- **Partner trust incluye confiar en su IdP y offboarding.**

## 16. Ficha de práctica

```text
Fecha y tiempo dedicado: ____________________________________________
Modalidad: organización real / inspección / alternativa conceptual

CASO A - ORGANIZACIÓN NUEVA
Datos clave: ________________________________________________________
Tipo de organización elegido: ______________________________________
Roles de recuperación y administración: ____________________________
Jerarquía propuesta: _______________________________________________
Baseline constraints conservadas: __________________________________
Por qué no elegí la alternativa: ___________________________________

CASO B - PERSONA EXTERNA
Actor: ______________________________________________________________
Pool/provider: ______________________________________________________
Subject y mapping: __________________________________________________
Condition: __________________________________________________________
Principal: __________________________________________________________
Rol y alcance: ______________________________________________________
Login: ______________________________________________________________
Offboarding: ________________________________________________________
Identidad de emergencia: ___________________________________________

Evidencia Console/CLI o simulada: __________________________________
Limpieza y costo: __________________________________________________
Puntuación: ____ / 10
Duda principal: ____________________________________________________
Siguiente acción concreta: _________________________________________
```

## 17. Ficha de errores

Completa una fila por respuesta fallada o decisión insegura.

| Pregunta o paso | Mi respuesta/acción | Evidencia que la refuta | Tipo de error | Regla corregida | Nueva explicación sin apuntes | +1 día | +7 días | +21 días |
|---|---|---|---|---|---|---|---|---|
|  |  |  | concepto / actor / alcance / claim / IAM / seguridad |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |

Tipos útiles:

- confundí Organization Owner con `roles/owner`;
- supuse que Organization Admin podía crear cualquier recurso;
- elegí Workload para una persona;
- confundí autenticación con autorización;
- usé una claim mutable como subject;
- omití la condition de tenant;
- concedí acceso a todo el pool o a toda la organización;
- ignoré soporte limitado de console (federated);
- olvidé offboarding o identidad de emergencia;
- propuse una clave persistente sin necesidad.

## 18. Criterios de autoevaluación

| Criterio | Aún no | Con apoyo | Sin apoyo y con evidencia |
|---|:---:|:---:|:---:|
| Explico Organization, Folder, Project y Resource | ☐ | ☐ | ☐ |
| Distingo standalone de Cloud Identity/Workspace organization | ☐ | ☐ | ☐ |
| Distingo Organization Owner, Organization Admin y Owner | ☐ | ☐ | ☐ |
| Elijo una o varias organizaciones con justificación | ☐ | ☐ | ☐ |
| Distingo Workforce de Workload Identity Federation | ☐ | ☐ | ☐ |
| Separo autenticación y autorización | ☐ | ☐ | ☐ |
| Diseño un subject estable y una condition de tenant | ☐ | ☐ | ☐ |
| Construyo el principal individual correcto | ☐ | ☐ | ☐ |
| Limito IAM al proyecto y tarea necesarios | ☐ | ☐ | ☐ |
| Documento offboarding, costo y limpieza | ☐ | ☐ | ☐ |

Meta personal para avanzar: al menos 8 de 10 preguntas correctas y evidencia “sin apoyo” en los criterios esenciales. Es una meta de estudio, no un umbral oficial del examen. Si no la alcanzas, registra los errores y repite los dos casos; no marques el tema como dominado por haber recibido el archivo.

## 19. Documentación oficial verificada

Consultada el 8 de octubre de 2026:

- [Associate Cloud Engineer certification](https://cloud.google.com/learn/certification/cloud-engineer)
- [Set up standalone organizations](https://docs.cloud.google.com/resource-manager/docs/standalone-organization-overview)
- [Set up a Google Cloud organization resource](https://docs.cloud.google.com/resource-manager/docs/creating-managing-organization)
- [Resource hierarchy](https://docs.cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy)
- [Manage multiple organization resources](https://docs.cloud.google.com/resource-manager/docs/managing-multiple-orgs)
- [Manage Google Cloud security baseline constraints](https://docs.cloud.google.com/resource-manager/docs/manage-baseline-constraints)
- [`gcloud version`](https://cloud.google.com/sdk/gcloud/reference/version)
- [`gcloud auth list`](https://cloud.google.com/sdk/gcloud/reference/auth/list)
- [`gcloud organizations list`](https://cloud.google.com/sdk/gcloud/reference/organizations/list)
- [`gcloud organizations describe`](https://cloud.google.com/sdk/gcloud/reference/organizations/describe)
- [Workforce Identity Federation overview](https://docs.cloud.google.com/iam/docs/workforce-identity-federation)
- [Configure Workforce Identity Federation](https://docs.cloud.google.com/iam/docs/configuring-workforce-identity-federation)
- [Best practices for Workforce Identity Federation](https://docs.cloud.google.com/iam/docs/best-practices-workforce-identity-federation)
- [Principal identifiers](https://docs.cloud.google.com/iam/docs/principal-identifiers)
- [User identities](https://docs.cloud.google.com/iam/docs/user-identities)
- [Identity federation supported products and limitations](https://docs.cloud.google.com/iam/docs/federated-identity-supported-services)
- [Set up console (federated)](https://docs.cloud.google.com/iam/docs/workforce-console-sso)
- [Sign in to gcloud with a federated identity](https://docs.cloud.google.com/iam/docs/workforce-log-in-gcloud)
- [`gcloud iam workforce-pools list`](https://cloud.google.com/sdk/gcloud/reference/iam/workforce-pools/list)
- [`gcloud iam workforce-pools describe`](https://cloud.google.com/sdk/gcloud/reference/iam/workforce-pools/describe)
- [`gcloud iam workforce-pools providers list`](https://cloud.google.com/sdk/gcloud/reference/iam/workforce-pools/providers/list)
- [`gcloud iam workforce-pools providers describe`](https://cloud.google.com/sdk/gcloud/reference/iam/workforce-pools/providers/describe)
- [`gcloud iam workforce-pools create-login-config`](https://cloud.google.com/sdk/gcloud/reference/iam/workforce-pools/create-login-config)
- [`gcloud auth login`](https://cloud.google.com/sdk/gcloud/reference/auth/login)
- [`gcloud projects add-iam-policy-binding`](https://cloud.google.com/sdk/gcloud/reference/projects/add-iam-policy-binding)
- [IAM pricing](https://cloud.google.com/iam/pricing)
- [Workforce Identity Federation pricing](https://cloud.google.com/workforce-identity-federation)

## 20. Preguntas de examen originales

Las preguntas son inéditas, basadas en escenarios y no proceden de dumps. Responde antes de leer la sección final.

### Question 1

A startup is a new Google Cloud Free Trial customer. It has no Cloud Identity account and no verified domain, but it needs an organization resource for governance. What should the startup expect?

A. It must create a service account and make it the organization owner.  
B. Google Cloud can automatically create a standalone organization for the eligible new account.  
C. It must buy Google Workspace Enterprise before creating any project.  
D. It should create one organization for each application environment.

### Question 2

In a standalone organization, which statement correctly distinguishes Organization Owner from Organization Administrator?

A. Organization Owner manages lifecycle and recovery outside IAM; Organization Administrator manages IAM policies and access.  
B. Organization Owner can only view billing; Organization Administrator automatically owns every project.  
C. They are aliases for the project-level basic Owner role.  
D. Organization Administrator is a service account reserved for Google-managed services.

### Question 3

SiteOps Tracker needs to give a contractor who signs in through an external OIDC identity provider temporary access to inspect a development project. Which identity solution is the best fit?

A. Workload Identity Federation  
B. A downloaded service account key  
C. Workforce Identity Federation  
D. Identity Platform for application customers

### Question 4

A federated user completes SSO successfully but receives `PERMISSION_DENIED` when listing assets. What is the most likely next area to inspect?

A. The project's IAM allow policy, principal identifier, role, and scope  
B. Whether the user has a permanent service account key  
C. The number of zones in the project's region  
D. The Cloud Billing budget threshold

### Question 5

An OIDC provider issues a permanent internal user ID in `sub` and a mutable email address in `email`. Which mapping is the safer choice for the required workforce subject?

A. `google.subject=assertion.sub`  
B. `google.subject=assertion.email` because emails are human-readable  
C. `google.subject=assertion.display_name`  
D. Omit `google.subject` and use only `google.groups`

### Question 6

The external IdP is multi-tenant and uses the same issuer URI for all customers. What additional control is most important?

A. An attribute condition that restricts tokens to the expected tenant  
B. Project Owner for every federated identity  
C. A longer organization display name  
D. Disabling token signature validation

### Question 7

Which principal identifier grants access to one workforce identity whose mapped subject is `auditor-042` in pool `siteops-auditors`?

A. `serviceAccount:auditor-042@siteops-auditors.iam.gserviceaccount.com`  
B. `principal://iam.googleapis.com/locations/global/workforcePools/siteops-auditors/subject/auditor-042`  
C. `principalSet://iam.googleapis.com/projects/siteops-auditors/group/auditor-042`  
D. `user:auditor-042`

### Question 8

An engineer has approved pool and provider IDs and needs a browser-based gcloud login configuration. Which command is correct?

A. `gcloud iam workforce-pools create-login-config locations/global/workforcePools/POOL/providers/PROVIDER --output-file=login.json`  
B. `gcloud auth activate-service-account --workforce-pool=POOL --provider=PROVIDER`  
C. `gcloud organizations create-login-config POOL/PROVIDER --key-file=login.json`  
D. `gcloud iam workload-identity-pools create-cred-config POOL --human-user`

### Question 9

An organization uses Workforce Identity Federation for most engineers. Which approach best protects emergency and highly privileged access?

A. Grant the whole workforce pool Organization Administrator.  
B. Keep protected Cloud Identity emergency and privileged accounts with strong MFA controls.  
C. Store a shared service account key in a team password manager.  
D. Disable attribute conditions to simplify recovery.

### Question 10

Which statement about cost and product support is accurate?

A. Workforce Identity Federation is charged per login and supports every Console product without limitations.  
B. Workforce Identity Federation is free, but detailed audit logging can incur Cloud Logging costs and the federated console supports only compatible products.  
C. Creating a workforce pool automatically creates billable VMs.  
D. IAM API calls require a paid Google Workspace edition.

## 21. Soluciones justificadas

### 1. Correct answer: B

**Why:** Eligible new Free Trial customers receive a standalone organization automatically without requiring Cloud Identity.

- **A is wrong:** A service account cannot be invited as Organization Owner, and at least one active Google Account must remain an owner.
- **C is wrong:** Google Workspace Enterprise is not a prerequisite for a standalone organization.
- **D is wrong:** Environments are usually separated with folders and projects inside one organization, not separate organizations by default.

### 2. Correct answer: A

**Why:** Organization Owner is the standalone-organization lifecycle and recovery role managed outside IAM. Organization Administrator is the IAM role used for policy and access administration.

- **B is wrong:** Neither statement reflects the documented responsibilities, and Organization Administrator doesn't automatically own projects.
- **C is wrong:** The project basic role `roles/owner` is a different role.
- **D is wrong:** Organization Administrator is an IAM role assignable to authorized principals, not a Google service account.

### 3. Correct answer: C

**Why:** The actor is a human contractor using an external IdP, which is the Workforce Identity Federation use case.

- **A is wrong:** Workload Identity Federation is for applications and other non-human workloads.
- **B is wrong:** A long-lived service account key is a workload credential and creates unnecessary security risk for a human user.
- **D is wrong:** Identity Platform is for customer identity in an application, not workforce access to Google Cloud resources.

### 4. Correct answer: A

**Why:** Successful SSO proves authentication. `PERMISSION_DENIED` on the resource points to authorization: mapped principal, IAM role, policy and resource scope.

- **B is wrong:** Workforce users don't need permanent service account keys.
- **C is wrong:** Zones don't determine whether the principal can list assets.
- **D is wrong:** A budget threshold doesn't grant IAM permissions.

### 5. Correct answer: A

**Why:** `sub` is described as a permanent internal ID, so mapping it to `google.subject` keeps the principal stable.

- **B is wrong:** The scenario says the email can change; changing the subject can break authorization.
- **C is wrong:** A display name is not necessarily unique or immutable.
- **D is wrong:** `google.subject` is required; groups don't replace it.

### 6. Correct answer: A

**Why:** With a shared issuer, an attribute condition must constrain accepted identities to the intended tenant and reduce confused-deputy risk.

- **B is wrong:** Project Owner is excessive and doesn't validate token origin.
- **C is wrong:** Display names don't secure token acceptance.
- **D is wrong:** Signature validation is fundamental and must not be disabled.

### 7. Correct answer: B

**Why:** A single workforce identity uses `principal://.../workforcePools/POOL_ID/subject/SUBJECT` and workforce pools use `locations/global`.

- **A is wrong:** This invents a service account instead of referencing the federated human.
- **C is wrong:** It uses a malformed project/group path and a principal set rather than one subject.
- **D is wrong:** `user:` identifies a Google Account in IAM, not this workforce subject.

### 8. Correct answer: A

**Why:** `gcloud iam workforce-pools create-login-config` accepts the provider audience and requires `--output-file`. The result is used with `gcloud auth login --login-config=...`.

- **B is wrong:** `activate-service-account` is not the browser-based workforce login flow.
- **C is wrong:** The command doesn't belong to the organizations group and doesn't create a key file.
- **D is wrong:** Workload credential configuration is a different flow for non-human identities.

### 9. Correct answer: B

**Why:** Protected Cloud Identity emergency and privileged accounts reduce complete dependence on the external IdP and support stronger recovery controls.

- **A is wrong:** Granting an entire pool Organization Administrator violates least privilege and magnifies IdP compromise risk.
- **C is wrong:** A shared long-lived key weakens attribution and introduces credential leakage risk.
- **D is wrong:** Removing attribute conditions broadens who can authenticate and makes recovery less secure.

### 10. Correct answer: B

**Why:** Workforce Identity Federation is a no-cost feature, while detailed audit logging uses Cloud Logging and may incur its normal charges. Console (federated) supports a limited set of products and features.

- **A is wrong:** There is no per-login charge stated, and product support isn't universal.
- **C is wrong:** A workforce pool doesn't create compute resources.
- **D is wrong:** IAM API use doesn't require a paid Workspace edition.
