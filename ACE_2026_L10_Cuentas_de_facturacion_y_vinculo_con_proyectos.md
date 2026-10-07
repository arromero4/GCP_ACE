# ACE 2026 — Lección 10: Cuentas de facturación y vínculo con proyectos

> **Fecha:** 2 de octubre de 2026  
> **Dominio de la guía ACE:** 1.2 Managing billing configuration  
> **Tema del plan:** Cuentas de facturación y vínculo con proyectos  
> **Práctica del plan:** Esquematizar un proyecto vinculado a una cuenta y explicar quién puede administrarlos.  
> **Duración sugerida:** 75–90 minutos  
> **Caso conductor:** SiteOps Tracker, proyecto ficticio de portafolio.

Esta lección prepara una parte del examen; recibirla no demuestra dominio. El dominio se comprueba explicando las relaciones sin apuntes, resolviendo los escenarios y justificando cada permiso.

## 1. Objetivo

Al terminar la sesión podrás:

1. distinguir un **Google Cloud project**, una **Cloud Billing account** y un **Google payments profile/account**;
2. explicar la cardinalidad del vínculo: una cuenta de facturación puede pagar varios proyectos, mientras cada proyecto se vincula con una sola cuenta de facturación activa a la vez;
3. separar la jerarquía de recursos y permisos de la relación de pago;
4. elegir los roles mínimos para consultar, crear, vincular, cambiar o administrar una cuenta de facturación;
5. inspeccionar, sin modificar, la configuración de facturación desde Google Cloud Console y Google Cloud CLI;
6. dibujar el esquema solicitado para SiteOps Tracker y explicar quién puede administrar cada recurso.

### Evidencia de aprendizaje

Produce al final:

- un diagrama con dos tipos de relación claramente rotulados: **ownership/IAM hierarchy** y **payment link**;
- una tabla de responsabilidades con al menos cuatro roles;
- una explicación oral de dos minutos sobre por qué vincular un proyecto no concede acceso a sus recursos;
- el resultado anonimizado de los comandos de inspección, o la alternativa conceptual completa si no tienes cuenta o crédito.

## 2. Prerrequisitos explicados desde cero

### 2.1 Proyecto de Google Cloud

Un proyecto es el contenedor básico donde se crean y administran recursos como una instancia de Compute Engine, un bucket de Cloud Storage o una base de datos de Cloud SQL. Tiene:

- un **project ID** único y estable;
- un **project number** asignado por Google;
- políticas de Identity and Access Management, o **IAM**;
- APIs habilitadas, cuotas, recursos y consumo medible.

Piensa en un proyecto como una sede operativa de SiteOps Tracker: allí viven la API Node.js, la interfaz React, la base PostgreSQL administrada y sus registros.

### 2.2 Jerarquía de recursos

Los proyectos pueden pertenecer a una organización y, opcionalmente, a carpetas. Esa jerarquía sirve para organización, políticas e IAM:

**Organization → Folder → Project → Resource**

La cuenta de facturación no sustituye a la organización ni se convierte en padre IAM del proyecto.

### 2.3 IAM

IAM responde: **¿quién puede hacer qué sobre cuál recurso?**

- **Principal:** usuario, grupo, service account u otra identidad.
- **Role:** conjunto de permisos.
- **Policy:** unión entre principal y rol sobre un recurso.

Para el tema de hoy hay dos superficies de autorización:

1. permisos sobre el **proyecto**;
2. permisos sobre la **cuenta de facturación**.

Una operación de vínculo cruza ambas superficies, por lo que normalmente requiere autorización en las dos.

### 2.4 Google Cloud CLI

Google Cloud CLI proporciona el comando <code>gcloud</code>. En esta lección solo se usa para leer configuración. Antes de ejecutar comandos:

- confirma la identidad activa;
- confirma el proyecto seleccionado;
- usa únicamente un proyecto personal, de laboratorio o autorizado;
- reemplaza los marcadores de ejemplo;
- no pegues en el documento IDs reales, nombres de personas ni datos de pago.

## 3. Modelo mental: tres objetos distintos

Imagina el sistema financiero de una empresa:

- el **proyecto** es una sucursal que consume electricidad;
- la **Cloud Billing account** es el centro de costo que recibe y agrupa los cargos;
- el **Google payments profile/account** identifica a la entidad legal y sus instrumentos de pago.

La sucursal no pasa a ser propiedad del centro de costo por recibir allí su factura. De la misma forma, el vínculo de facturación no convierte a la cuenta de facturación en padre IAM del proyecto.

| Objeto | Pregunta que responde | Contiene o administra | No implica |
|---|---|---|---|
| Google Cloud project | ¿Dónde viven y se controlan los recursos? | APIs, cuotas, recursos, IAM del proyecto y consumo | Autoridad sobre la cuenta de facturación |
| Cloud Billing account | ¿A qué cuenta se cargan los costos? | Relación de pago con proyectos, acceso de facturación, moneda e historial de costos | Propiedad IAM de los recursos del proyecto |
| Google payments profile/account | ¿Quién es la entidad pagadora y con qué medio paga? | Datos legales, fiscales y métodos de pago según el tipo de cuenta | Permisos para desplegar recursos en Google Cloud |

### 3.1 Cardinalidad del vínculo

- **Una Cloud Billing account puede pagar uno o muchos proyectos.**
- **Un proyecto puede estar vinculado con una sola Cloud Billing account a la vez.**
- Cambiar el vínculo mueve el cobro futuro del proyecto a otra cuenta válida y activa; no mueve el proyecto en la jerarquía.
- Una cuenta cerrada se comporta, para el proyecto, como facturación deshabilitada.

### 3.2 El vínculo no es herencia

Estas relaciones deben dibujarse por separado:

~~~mermaid
flowchart TB
  ORG["Organization (IAM parent)"] --> DEV["siteops-dev project"]
  ORG --> PROD["siteops-prod project"]
  DEV -. "payment link" .-> BILL["Cloud Billing account"]
  PROD -. "payment link" .-> BILL
  BILL --> PAY["Google payments profile/account"]
~~~

Las flechas sólidas superiores representan pertenencia o asociación administrativa. Las flechas punteadas representan quién paga. Ninguna flecha punteada concede, por sí sola, permisos para leer activos, corregir hallazgos o desplegar la API.

## 4. Aplicación a SiteOps Tracker

SiteOps Tracker es una aplicación ficticia para:

- registrar sedes y activos;
- documentar hallazgos de auditoría;
- asignar responsables;
- cambiar estados;
- conservar un historial de acciones.

Su stack de ejemplo es React + TypeScript, Node.js + TypeScript, PostgreSQL y servicios de Google Cloud.

### 4.1 Separación propuesta

| Elemento ficticio | Propósito | Relación de facturación |
|---|---|---|
| <code>siteops-dev</code> | Desarrollo y pruebas del frontend, API y esquema PostgreSQL | Vinculado a la cuenta central de laboratorio |
| <code>siteops-prod</code> | Servicio de producción y datos operativos simulados | Vinculado a la misma cuenta o a una cuenta separada si la gobernanza lo exige |
| Cloud Billing account | Agrupa los cargos de los proyectos vinculados | Paga el consumo; no administra los activos de SiteOps |
| Payments profile/account | Entidad legal pagadora e instrumentos de pago | Se asocia con la cuenta de facturación |

Los nombres son marcadores ficticios. En una captura o entrega, reemplaza cualquier identificador real por valores como <code>PROJECT_A</code> y <code>BILLING_ACCOUNT_X</code>.

### 4.2 Flujo paso a paso

1. Un desarrollador despliega la API de SiteOps Tracker en un proyecto autorizado.
2. El servicio consume CPU, almacenamiento, red y base de datos.
3. Los medidores atribuyen ese consumo al proyecto.
4. El proyecto tiene un vínculo de pago con una Cloud Billing account.
5. Los cargos aparecen en esa cuenta de facturación.
6. La entidad y el método de pago se gestionan mediante el sistema de Google payments.
7. IAM del proyecto sigue decidiendo quién puede ver o cambiar los recursos de la aplicación.
8. IAM de facturación decide quién puede ver costos, vincular proyectos o administrar la cuenta.

## 5. Quién puede administrarlos

### 5.1 Roles principales

| Rol predefinido | Alcance habitual | Puede hacer | No debe confundirse con |
|---|---|---|---|
| **Billing Account Creator** — <code>roles/billing.creator</code> | Organización | Crear nuevas cuentas de facturación | Billing Account Administrator; crear no equivale a administrar todos los proyectos |
| **Billing Account Administrator** — <code>roles/billing.admin</code> | Cuenta de facturación | Administrar la cuenta, su IAM, vínculos y configuración de facturación | Project Owner; no concede administración de recursos del proyecto |
| **Billing Account User** — <code>roles/billing.user</code> | Cuenta de facturación | Usar esa cuenta como destino al vincular un proyecto | Billing Account Viewer; ver no permite vincular |
| **Billing Account Viewer** — <code>roles/billing.viewer</code> | Cuenta de facturación | Consultar información y costos permitidos | Billing Account User; no crea la asociación |
| **Project Billing Manager** — <code>roles/billing.projectManager</code> | Proyecto | Vincular o desvincular la facturación del proyecto | Editor/Owner; no administra los recursos de la aplicación |
| **Project Owner** — <code>roles/owner</code> | Proyecto | Incluye permisos amplios, entre ellos permisos del lado del proyecto para facturación | Permiso automático sobre cualquier cuenta de facturación |

### 5.2 Patrón de mínimo privilegio para vincular

Una combinación común y deliberadamente limitada es:

- **Project Billing Manager** sobre el proyecto; y
- **Billing Account User** más visibilidad necesaria sobre la cuenta de facturación de destino.

La documentación actual también indica permisos de lectura necesarios para el flujo de Console. En términos de roles, una persona puede requerir:

- en el proyecto: **Project Billing Manager + Project Browser + Service Usage Viewer**, o **Project Owner**;
- en la cuenta destino: **Billing Account User + Billing Account Viewer**, o **Billing Account Administrator**.

La combinación exacta depende de si se usa Console o API/CLI y de la operación. Para cambiar desde una cuenta actual a otra, también se necesita capacidad de quitar la asociación vigente, ya sea del lado de la cuenta actual o del proyecto.

### 5.3 Permisos atómicos que explican la operación

| Acción | Permiso relevante | Recurso donde se evalúa |
|---|---|---|
| Ver la asociación | <code>billing.resourceAssociations.list</code> | Cuenta de facturación |
| Leer el proyecto | <code>resourcemanager.projects.get</code> | Proyecto |
| Crear asociación con la cuenta destino | <code>billing.resourceAssociations.create</code> | Cuenta destino |
| Crear asignación de facturación | <code>resourcemanager.projects.createBillingAssignment</code> | Proyecto |
| Quitar asociación desde la cuenta actual | <code>billing.resourceAssociations.delete</code> | Cuenta actual |
| Quitar asignación desde el proyecto | <code>resourcemanager.projects.deleteBillingAssignment</code> | Proyecto |
| Consultar servicios para el flujo de Console | <code>serviceusage.services.list</code> y <code>serviceusage.effectivepolicy.get</code> | Proyecto |

### 5.4 Separación de funciones para SiteOps Tracker

| Persona ficticia | Acceso recomendado | Motivo |
|---|---|---|
| Responsable financiero | Billing Account Viewer; Admin solo si administra la cuenta | Puede revisar costos sin desplegar ni alterar recursos |
| Operador de plataforma | Project Billing Manager en el proyecto + Billing Account User en la cuenta permitida | Puede crear el vínculo sin recibir privilegios generales de Project Owner |
| Desarrollador de SiteOps | Roles técnicos mínimos sobre servicios del proyecto | No necesita modificar la relación de pago |
| Administrador de facturación | Billing Account Administrator | Administra IAM y configuración de la cuenta, pero no obtiene acceso automático a los datos de SiteOps |

## 6. Decisiones de servicio y por qué no elegir alternativas

### 6.1 ¿Una o varias cuentas de facturación?

| Necesidad | Elección razonable | Por qué | Por qué no la alternativa |
|---|---|---|---|
| Un solo pagador, moneda y equipo financiero para dev y prod | Una cuenta para varios proyectos | Simplifica pagos, permisos y consolidación | Varias cuentas añaden administración sin una frontera real |
| Entidades legales o pagadores distintos | Cuentas separadas | Reflejan responsabilidades de pago diferentes | Una sola cuenta mezclaría responsabilidades financieras |
| Solo separar costos de dev y prod | Proyectos separados, etiquetas y exportación/agrupación de costos | La separación analítica no exige otra cuenta | Crear cuentas adicionales solo para “carpetas de costos” suele ser innecesario |
| Aislamiento fuerte exigido por política financiera | Cuentas separadas y permisos independientes | Establece una frontera administrativa de facturación | Solo etiquetas no separan quién administra el pago |

### 6.2 Billing account frente a alternativas

| Si necesitas… | Usa… | No uses como sustituto… | Razón |
|---|---|---|---|
| Definir quién paga el consumo | Cloud Billing account | Organization/Folder | La jerarquía organiza y gobierna; no es el destino de cobro |
| Definir la entidad legal y método de pago | Google payments profile/account | Project IAM | Son sistemas de acceso distintos |
| Limitar consumo técnico | Quotas y límites de servicio | Billing account | Facturación atribuye costos; no es un límite de capacidad |
| Recibir avisos de gasto | Budgets and alerts | Desvincular el proyecto preventivamente | Un presupuesto avisa; desvincular puede detener servicios y causar pérdida |
| Dar capacidad de vínculo sin administrar recursos | Project Billing Manager + Billing Account User | Project Owner | Owner es demasiado amplio para esa tarea |

### 6.3 Budget no es spending cap

Un presupuesto de Cloud Billing sirve para seguimiento y alertas; no detiene automáticamente el consumo. Tampoco reemplaza el vínculo del proyecto, las cuotas o controles automatizados explícitos. La configuración detallada de presupuestos corresponde a otra práctica del plan.

## 7. Práctica guiada: inspeccionar, esquematizar y explicar

### 7.1 Alcance y seguridad

La práctica obligatoria es **de solo lectura**. No cambies ni deshabilites la facturación. Hazla únicamente en un proyecto personal, de laboratorio o para el que tengas autorización explícita.

No expongas:

- el identificador real de la cuenta de facturación;
- datos fiscales o medios de pago;
- correos de usuarios;
- nombres de proyectos privados.

En tu evidencia utiliza marcadores anonimizados.

### 7.2 Ruta A — Google Cloud Console

La navegación fue contrastada con la documentación oficial vigente al 2 de octubre de 2026.

1. Abre **Google Cloud Console**.
2. En el selector superior, elige tu proyecto de laboratorio.
3. Ve a **Billing**.
4. Si aparece la vista de cuenta, abre **My Projects**.
5. Localiza el proyecto y observa:
   - nombre o ID anonimizado;
   - estado de facturación;
   - cuenta de facturación asociada;
   - acciones visibles, sin ejecutarlas.
6. Si tienes acceso a la cuenta, abre **Account management**.
7. Observa qué proyectos aparecen vinculados y abre el panel de información de IAM solo para reconocer roles.
8. Cierra cualquier cuadro de cambio sin guardar.

Registra la observación así:

| Campo | Valor anonimizado |
|---|---|
| Proyecto | <code>PROJECT_A</code> |
| Facturación habilitada | Sí / No |
| Cuenta asociada | <code>BILLING_ACCOUNT_X</code> / Ninguna |
| Mi acceso visible | Rol o “no visible” |
| Cambio realizado | Ninguno |

### 7.3 Ruta B — Google Cloud CLI

Los comandos y opciones de facturación fueron verificados en la referencia oficial actual de <code>gcloud</code>. Ejecuta una línea a la vez.

#### Paso 1: confirma identidad y contexto

~~~bash
gcloud auth list --filter=status:ACTIVE
gcloud config get-value project
~~~

Si la identidad o proyecto no son los esperados, detente y selecciona el contexto autorizado antes de continuar.

#### Paso 2: declara un marcador local y lee el proyecto

~~~bash
export PROJECT_ID="YOUR_LAB_PROJECT_ID"
gcloud projects describe "$PROJECT_ID"
~~~

#### Paso 3: describe la relación de facturación

~~~bash
gcloud billing projects describe "$PROJECT_ID"
~~~

Busca conceptualmente estos campos:

- <code>projectId</code>;
- <code>billingAccountName</code>;
- <code>billingEnabled</code>.

Si se muestra un ID real, no lo pegues en una entrega pública.

#### Paso 4: lista únicamente las cuentas activas que tu identidad puede ver

~~~bash
gcloud billing accounts list --filter=open=true
~~~

Que una cuenta no aparezca no demuestra que no exista; puede significar que tu identidad no tiene permiso para verla.

#### Paso 5: describe una cuenta visible, sin cambiarla

Solo si el paso anterior mostró una cuenta autorizada:

~~~bash
export BILLING_ACCOUNT_ID="OBSERVED_ACCOUNT_ID"
gcloud billing accounts describe "$BILLING_ACCOUNT_ID"
~~~

#### Sintaxis que debes reconocer, pero no ejecutar hoy

El comando oficial para vincular o mover un proyecto es:

~~~bash
gcloud billing projects link "$PROJECT_ID" --billing-account="$BILLING_ACCOUNT_ID"
~~~

No lo ejecutes como parte de esta práctica. Requiere permisos en el proyecto y la cuenta destino, y puede cambiar quién paga los consumos futuros.

El comando de desvinculación es:

~~~bash
gcloud billing projects unlink "$PROJECT_ID"
~~~

Tampoco lo ejecutes. Puede detener servicios facturables y afectar la disponibilidad o conservación de recursos.

### 7.4 Ruta C — alternativa conceptual completa sin cuenta ni crédito

Puedes completar íntegramente el objetivo sin una cuenta activa.

#### Datos del caso

- <code>siteops-dev</code> pertenece a una organización ficticia.
- <code>siteops-prod</code> pertenece a la misma organización.
- <code>BILLING-X</code> está abierta y puede pagar ambos proyectos.
- Ana tiene Project Billing Manager en <code>siteops-dev</code>.
- Bruno tiene Billing Account User en <code>BILLING-X</code>.
- Carla tiene ambos roles anteriores.
- Diego tiene Billing Account Viewer en <code>BILLING-X</code>.
- Elena tiene Billing Account Administrator en <code>BILLING-X</code>, pero ningún rol en los proyectos.

#### Tareas

1. Redibuja el diagrama de la sección 3.2.
2. Marca las flechas de jerarquía y pago con estilos distintos.
3. Decide quién puede crear el vínculo por sí solo.
4. Explica por qué Ana y Bruno, por separado, no completan toda la operación.
5. Explica qué puede ver Diego y por qué no puede vincular.
6. Explica por qué Elena no obtiene acceso a la base PostgreSQL ni al historial de SiteOps.

#### Resultado del caso

Carla reúne permisos de ambos lados y es la única de la lista que puede completar por sí sola el vínculo descrito. Ana solo tiene el permiso del lado del proyecto; Bruno solo tiene el permiso del lado de la cuenta; Diego puede consultar; Elena administra la cuenta, pero la operación sobre el proyecto aún depende de los permisos requeridos allí.

### 7.5 Entregable de la práctica

Completa esta plantilla:

~~~text
Proyecto: PROJECT_A
Padre IAM: ORGANIZATION_A o “sin organización”, según el caso
Cuenta de facturación: BILLING_ACCOUNT_X
Tipo de relación proyecto → cuenta: payment link
¿La cuenta es padre IAM del proyecto?: No
Administrador de la cuenta: principal con Billing Account Administrator
Operador mínimo de vínculo: principal con permisos requeridos en proyecto y cuenta destino
Evidencia usada: Console / CLI / caso conceptual
Cambio realizado: Ninguno
~~~

Luego explica en voz alta:

> “El proyecto contiene los recursos y su IAM. La cuenta de facturación recibe sus cargos, pero no es su padre IAM. Para vincularlos se necesitan permisos del lado del proyecto y de la cuenta de destino.”

## 8. Resultado esperado

La práctica está completa si puedes mostrar, sin información sensible:

- el esquema del proyecto y la cuenta;
- la dirección correcta del vínculo de pago;
- una cuenta que puede pagar varios proyectos;
- un solo vínculo activo por proyecto;
- la separación entre Cloud Billing IAM, Project IAM y Google payments;
- los roles mínimos de vínculo;
- evidencia de inspección sin cambios.

Ejemplo de conclusión válida:

> SiteOps Tracker usa proyectos separados para desarrollo y producción. Ambos pueden cargar consumo a una cuenta central, pero esta relación solo define el pago. El equipo financiero administra la cuenta; el equipo técnico administra recursos mediante IAM del proyecto. Un operador de vínculo necesita permisos explícitos en ambos lados.

## 9. Solución de problemas

| Síntoma | Causa probable | Comprobación segura | Acción recomendada |
|---|---|---|---|
| <code>gcloud billing accounts list</code> no muestra nada | No hay acceso visible, identidad equivocada o cuentas cerradas | Revisa <code>gcloud auth list</code> y elimina temporalmente el filtro solo si está autorizado | Solicita Billing Account Viewer o User según la tarea; no asumas que no existe una cuenta |
| <code>billingEnabled: false</code> | Proyecto sin vínculo válido o cuenta cerrada | Describe el proyecto y verifica el estado de la cuenta | Escala al administrador; no vincules una cuenta personal por conveniencia |
| <code>PERMISSION_DENIED</code> al vincular | Falta permiso en el proyecto, la cuenta destino o ambos | Separa la operación en sus dos superficies IAM | Solicita el rol mínimo que falta |
| Falta <code>createBillingAssignment</code> | No hay permiso de asignación en el proyecto | Revisa el rol del principal sobre el proyecto | Project Billing Manager puede ser suficiente sin otorgar Owner |
| La cuenta aparece, pero no se puede elegir | Puede faltar User/Viewer, la cuenta está cerrada o existe una restricción | Describe la cuenta y revisa permisos con un administrador | Usa una cuenta abierta y autorizada |
| No se puede mover el proyecto | Falta quitar el vínculo actual o crear el nuevo | Revisa permisos sobre cuenta actual, destino y proyecto | Concede solo el permiso faltante; no desvincules primero salvo necesidad |
| “Change billing” no está disponible | IAM insuficiente, bloqueo del vínculo o contexto incorrecto | Verifica proyecto seleccionado y consulta al administrador | No intentes evadir el bloqueo |
| Un servicio dejó de funcionar después de desvincular | Se deshabilitó la facturación necesaria | Confirma el estado de facturación y revisa el servicio | Restaura el vínculo autorizado; algunos servicios tardan hasta 24 h o requieren reinicio |
| Los cargos siguen apareciendo tras desvincular | Uso acumulado aún no procesado | Revisa las fechas de uso y reporte | Espera el procesamiento; los cargos anteriores siguen siendo pagaderos |
| Se confundió nombre con ID de cuenta | Se usó un display name donde se esperaba account ID | Compara la salida de <code>accounts list</code> | Usa el identificador esperado por el comando, sin publicarlo |
| Un Billing Admin no puede abrir la base de datos | Billing IAM no concede acceso a recursos | Revisa IAM del proyecto y del servicio | Asigna solo el rol técnico necesario, si corresponde |

## 10. Costos, riesgo y limpieza

### 10.1 Impacto en costos

- Consultar páginas de configuración y ejecutar los comandos de lectura de esta práctica no crea recursos facturables.
- Vincular un proyecto no crea por sí solo una VM, una base de datos ni un bucket, pero permite que el consumo facturable del proyecto se cargue a la cuenta.
- Cambiar el vínculo afecta dónde se atribuyen cargos futuros; los cargos ya acumulados pueden tardar en reflejarse.
- Deshabilitar facturación no es un mecanismo seguro de control presupuestario. Puede detener servicios y algunos recursos podrían eliminarse o no recuperarse completamente.
- Los presupuestos generan seguimiento y alertas, pero no son un límite automático de gasto.

### 10.2 Limpieza

La ruta recomendada es solo lectura, por lo que **no hay recursos que limpiar**.

Si, fuera de esta práctica y con autorización, se modificó un proyecto temporal:

1. documenta la cuenta original antes de hacer cualquier otro cambio;
2. elimina primero los recursos facturables que ya no sean necesarios siguiendo el procedimiento de cada servicio;
3. restaura el vínculo autorizado mediante el proceso formal;
4. verifica el estado con <code>gcloud billing projects describe</code>;
5. nunca desvincules un proyecto de producción como “limpieza”.

No borres un proyecto ni cierres una cuenta de facturación solo para completar esta lección.

## 11. Glosario bilingüe

| English | Español | Significado práctico |
|---|---|---|
| Billing account | Cuenta de facturación | Recurso de Google Cloud que recibe cargos de proyectos vinculados |
| Billing account ID | ID de cuenta de facturación | Identificador usado por API y CLI |
| Billing enabled | Facturación habilitada | El proyecto tiene un vínculo válido con una cuenta activa |
| Billing disabled | Facturación deshabilitada | No existe un vínculo utilizable para cobrar servicios |
| Billing account administrator | Administrador de cuenta de facturación | Rol amplio para administrar la cuenta y su IAM |
| Billing account user | Usuario de cuenta de facturación | Puede usar una cuenta permitida para vincular proyectos |
| Billing account viewer | Visualizador de cuenta de facturación | Puede consultar, pero no crear vínculos |
| Billing account creator | Creador de cuentas de facturación | Puede crear cuentas, normalmente a nivel de organización |
| Project Billing Manager | Administrador de facturación del proyecto | Administra la asignación de facturación del proyecto sin administrar todos sus recursos |
| Link | Vincular | Asociar un proyecto con la cuenta que pagará su consumo |
| Unlink | Desvincular | Quitar la asociación de pago; puede interrumpir servicios |
| Payment link | Vínculo de pago | Relación financiera, no relación padre-hijo de IAM |
| Resource hierarchy | Jerarquía de recursos | Organization, folders, projects y recursos |
| Google payments profile | Perfil de pagos de Google | Identidad legal y datos de pago asociados |
| Principal | Principal o identidad | Usuario, grupo o service account al que se asigna un rol |
| Role | Rol | Conjunto de permisos |
| Least privilege | Mínimo privilegio | Conceder solo los permisos necesarios |
| Accrued charges | Cargos acumulados | Costos ya generados aunque todavía no se reflejen |
| Closed account | Cuenta cerrada | Cuenta que no puede pagar consumo nuevo |
| Spending cap | Tope de gasto | Límite que detiene consumo; un presupuesto de Cloud Billing no lo es |

## 12. Repaso activo espaciado

Responde sin consultar notas. Después compara con tus lecciones anteriores.

### Día hábil anterior — Lección 9: APIs y cuotas

1. ¿Qué diferencia hay entre habilitar una API y tener cuota disponible?
2. Si una API está habilitada pero falla por cuota, ¿por qué cambiar la cuenta de facturación no garantiza resolverlo?
3. Relaciona cada concepto con su pregunta:
   - API habilitada;
   - cuota;
   - vínculo de facturación.

   Preguntas: “¿puedo llamar al servicio?”, “¿cuánta capacidad se permite?”, “¿quién paga el consumo?”

### Dos días hábiles antes — Lección 8: IAM

1. Define principal, role y policy con un ejemplo de SiteOps Tracker.
2. ¿Por qué Billing Account Administrator no equivale a Project Owner?
3. Diseña la combinación de mínimo privilegio para que una persona vincule un proyecto sin desplegar recursos.

### Tres días hábiles antes — Lección 7: Organization Policy

1. ¿Qué diferencia existe entre una restricción organizacional y un permiso IAM?
2. Si una política bloquea una acción, ¿conceder Owner necesariamente la permite? Explica.
3. ¿Cómo investigarías un botón de cambio de facturación deshabilitado sin evadir controles?

### Cuatro días hábiles antes — Lección 6: jerarquía de recursos

1. Dibuja Organization → Folder → Project → Resource.
2. Añade una cuenta de facturación sin convertirla en padre del proyecto.
3. Explica qué se hereda por la jerarquía y qué no se hereda por el vínculo de pago.

### Hace 7 días — Lección 5: método ACE, CLI, escenarios y vocabulario

1. En una pregunta ACE, subraya objetivo, restricciones y criterio de optimización.
2. Antes de ejecutar <code>gcloud</code>, ¿qué dos datos de contexto verificas?
3. Traduce: **least privilege**, **link a project**, **billing account**, **accrued charges**.

### Hace 21 días — línea base

El 11 de septiembre de 2026 fue anterior al inicio de este plan, por lo que no se inventa una lección. Usa una recuperación de línea base:

1. Explica con tus palabras qué es un proyecto de Google Cloud.
2. Nombra tres motivos por los que conviene separar desarrollo y producción.
3. Escribe qué creías que significaba “billing account” antes de esta lección y qué corregirías ahora.

## 13. Ficha de errores

Registra cada error después de responder las preguntas, no antes.

| Nº | Mi respuesta | Respuesta correcta | Tipo de error | Regla que faltó | Señal del escenario | Acción de refuerzo |
|---:|---|---|---|---|---|---|
| 1 |  |  | Concepto / IAM / lectura / prisa |  |  |  |
| 2 |  |  | Concepto / IAM / lectura / prisa |  |  |  |
| 3 |  |  | Concepto / IAM / lectura / prisa |  |  |  |
| 4 |  |  | Concepto / IAM / lectura / prisa |  |  |  |
| 5 |  |  | Concepto / IAM / lectura / prisa |  |  |  |

Usa estas reglas de reparación:

- Si confundiste pago con jerarquía, redibuja las dos relaciones.
- Si elegiste Owner sin necesidad, vuelve a construir la solución con roles mínimos.
- Si olvidaste un lado de la autorización, escribe “proyecto + cuenta destino”.
- Si trataste budget como tope, escribe “alerta, no apagado automático”.
- Si propusiste unlink como primer paso de un cambio, recuerda que <code>link</code> puede mover directamente el proyecto a otra cuenta autorizada.

## 14. Criterios de autoevaluación

Asigna 0, 1 o 2 puntos a cada criterio:

- **0:** no puedo explicarlo todavía;
- **1:** lo explico con apoyo;
- **2:** lo explico y justifico sin apuntes.

| Criterio | 0–2 |
|---|---:|
| Distingo project, billing account y payments profile/account |  |
| Explico uno-a-muchos y una cuenta activa por proyecto |  |
| Separo jerarquía IAM de vínculo de pago |  |
| Selecciono roles mínimos para vincular |  |
| Identifico los dos lados de permisos |  |
| Interpreto <code>billingEnabled</code> |  |
| Uso Console sin realizar cambios |  |
| Uso los comandos de inspección con el contexto correcto |  |
| Explico riesgos de unlink y cargos acumulados |  |
| Justifico mis respuestas en inglés o español técnico claro |  |
| **Total** | **/20** |

Interpretación:

- **17–20:** continúa, pero revisa cualquier distractor fallado;
- **13–16:** repite las explicaciones de IAM y el diagrama;
- **0–12:** rehace la alternativa conceptual y vuelve a intentar las preguntas.

Esta puntuación es una autoevaluación, no una certificación de dominio.

## 15. Documentación oficial verificada

Consultada el 2 de octubre de 2026:

- [Associate Cloud Engineer exam guide](https://cloud.google.com/learn/certification/guides/cloud-engineer)
- [Cloud Billing concepts](https://docs.cloud.google.com/billing/docs/concepts)
- [Enable, disable, or change billing for a project](https://docs.cloud.google.com/billing/docs/how-to/modify-project)
- [Overview of Cloud Billing access control](https://docs.cloud.google.com/billing/docs/how-to/billing-access)
- [Manage access to Cloud Billing accounts](https://docs.cloud.google.com/billing/docs/how-to/grant-access-to-billing)
- [Create custom roles for Cloud Billing](https://docs.cloud.google.com/billing/docs/how-to/custom-roles)
- [gcloud billing projects describe](https://docs.cloud.google.com/sdk/gcloud/reference/billing/projects/describe)
- [gcloud billing projects link](https://docs.cloud.google.com/sdk/gcloud/reference/billing/projects/link)
- [gcloud billing projects unlink](https://docs.cloud.google.com/sdk/gcloud/reference/billing/projects/unlink)
- [gcloud billing accounts list](https://docs.cloud.google.com/sdk/gcloud/reference/billing/accounts/list)
- [gcloud billing accounts describe](https://docs.cloud.google.com/sdk/gcloud/reference/billing/accounts/describe)

## 16. Preguntas de examen originales

Responde antes de abrir la sección de soluciones. Todas las preguntas son originales y se basan en escenarios; no son dumps del examen.

### Question 1

SiteOps Tracker uses separate development and production projects. Finance wants one payer and consolidated billing administration, while engineers must retain separate project IAM policies. What should the cloud engineer do?

A. Move both workloads into one project because a billing account can be linked to only one project.  
B. Link both projects to the same Cloud Billing account and keep project IAM separate.  
C. Make the Cloud Billing account the IAM parent of both projects.  
D. Link each project to two billing accounts for redundancy.

### Question 2

A platform operator must link an existing project to an approved billing account but must not receive broad permissions over application resources. Which role pairing best follows least privilege?

A. Project Owner on the project and Billing Account Administrator on the account  
B. Project Billing Manager on the project and Billing Account User on the billing account  
C. Billing Account Viewer on both the project and the billing account  
D. Organization Administrator and Project Editor

### Question 3

A finance analyst has Billing Account Viewer on a billing account. The analyst can inspect costs but cannot link a project. What is the best explanation?

A. Viewer is read-only and does not grant the permission to create a billing association.  
B. A billing account can never be linked after it is created.  
C. The analyst must become Project Owner only; billing account permissions are irrelevant.  
D. Projects can be linked only through the Google payments profile.

### Question 4

An engineer is Project Owner of a laboratory project but has no access to the target billing account. The engineer attempts to link the project and receives a permission error. What is missing?

A. A role on the target billing account that permits using it, such as Billing Account User  
B. Compute Admin on the project  
C. Organization Policy Administrator on the organization  
D. Service Account Token Creator on the default service account

### Question 5

The billing administrator for SiteOps Tracker is granted Billing Account Administrator on the account. Which statement is correct?

A. The administrator automatically receives access to PostgreSQL data in all linked projects.  
B. The administrator becomes the IAM parent of every linked project.  
C. The administrator can manage the billing account, but project-resource access remains controlled separately.  
D. The administrator can bypass organization policies in linked projects.

### Question 6

A project is already linked to Billing Account A. An authorized engineer must move it to Billing Account B. Which CLI operation expresses the intended change directly?

A. <code>gcloud billing projects link PROJECT_ID --billing-account=BILLING_ACCOUNT_B</code>  
B. <code>gcloud projects move PROJECT_ID --billing-account=BILLING_ACCOUNT_B</code>  
C. <code>gcloud billing accounts attach BILLING_ACCOUNT_B --project=PROJECT_ID</code>  
D. <code>gcloud iam projects inherit PROJECT_ID BILLING_ACCOUNT_B</code>

### Question 7

An engineer wants to verify whether billing is enabled for a project without changing any configuration. Which command is most appropriate?

A. <code>gcloud billing projects describe PROJECT_ID</code>  
B. <code>gcloud billing projects unlink PROJECT_ID</code>  
C. <code>gcloud billing accounts close ACCOUNT_ID</code>  
D. <code>gcloud projects delete PROJECT_ID</code>

### Question 8

A team plans to unlink production from its billing account whenever monthly spend approaches a target. What is the best response?

A. Approve the plan because unlinking has no effect on running services.  
B. Use a billing budget for alerts and design explicit cost controls; do not treat unlinking as a routine spending cap.  
C. Create a second link to another billing account before unlinking the first.  
D. Grant all developers Billing Account Administrator so anyone can unlink quickly.

### Question 9

An organization needs a principal to create new Cloud Billing accounts, but not to manage every project. Which role is designed for this purpose?

A. Billing Account Creator  
B. Billing Account Viewer  
C. Project Billing Manager  
D. Project Editor

### Question 10

A company wants to separate application ownership from payment administration. Which design best meets the requirement?

A. Give finance Project Owner on every linked project.  
B. Give developers Billing Account Administrator and remove project IAM.  
C. Manage project resources with project IAM and manage payment relationships with Cloud Billing IAM.  
D. Use the Google payments profile as the project resource hierarchy.

## 17. Soluciones justificadas

### 1. Correct answer: B

Una cuenta de facturación puede pagar varios proyectos y cada proyecto conserva su propia política IAM.

- **A** es incorrecta: una cuenta puede estar vinculada con múltiples proyectos.
- **C** es incorrecta: el vínculo de pago no convierte la cuenta en padre IAM.
- **D** es incorrecta: cada proyecto tiene una sola cuenta de facturación activa a la vez.

### 2. Correct answer: B

Project Billing Manager aporta la capacidad del lado del proyecto y Billing Account User permite usar la cuenta aprobada. Es el patrón que mejor satisface mínimo privilegio.

- **A** funcionaría con permisos adicionales, pero concede mucho más acceso del necesario.
- **C** no crea la asociación; Viewer es de lectura.
- **D** mezcla privilegios amplios e irrelevantes.

### 3. Correct answer: A

Billing Account Viewer permite consultar información, no crear vínculos.

- **B** es falsa: una cuenta abierta y válida puede recibir proyectos autorizados.
- **C** ignora que la operación requiere autorización del lado de la cuenta.
- **D** confunde el sistema de Google payments con Cloud Billing IAM.

### 4. Correct answer: A

Ser Project Owner cubre ampliamente el lado del proyecto, pero no concede por sí mismo permiso sobre la cuenta destino.

- **B** administra Compute Engine, no el vínculo de pago.
- **C** administra políticas organizacionales, no sustituye el permiso de la cuenta.
- **D** permite crear tokens de service accounts y no resuelve la asociación.

### 5. Correct answer: C

Cloud Billing IAM y Project IAM son sistemas separados. El administrador de facturación no hereda acceso a los recursos o datos del proyecto.

- **A** concedería acceso a datos que el rol de facturación no incluye.
- **B** transforma erróneamente un vínculo financiero en jerarquía.
- **D** es falsa: IAM de facturación no evade Organization Policy.

### 6. Correct answer: A

El comando <code>gcloud billing projects link</code> vincula un proyecto con una cuenta válida; si ya estaba vinculado, expresa el cambio a la nueva cuenta.

- **B**, **C** y **D** no son comandos válidos para esta operación.
- Desvincular primero no es necesario para expresar el cambio y puede introducir una interrupción evitable.

### 7. Correct answer: A

<code>describe</code> consulta la información de facturación del proyecto sin modificarla.

- **B** cambia el estado y puede detener servicios.
- **C** sería destructivo para la capacidad de pago y no responde por el proyecto.
- **D** elimina el contenedor del proyecto y es totalmente desproporcionado.

### 8. Correct answer: B

Los presupuestos y alertas sirven para seguimiento; los controles de costo deben diseñarse explícitamente. Desvincular producción puede detener servicios y poner recursos en riesgo.

- **A** niega un impacto documentado.
- **C** es imposible como estado simultáneo: un proyecto no mantiene dos cuentas activas.
- **D** viola mínimo privilegio y aumenta el riesgo.

### 9. Correct answer: A

Billing Account Creator está diseñado para crear cuentas de facturación, normalmente en el alcance de organización.

- **B** solo consulta.
- **C** gestiona el vínculo de facturación de proyectos.
- **D** administra recursos del proyecto y no crea cuentas de facturación.

### 10. Correct answer: C

La separación correcta conserva IAM del proyecto para recursos y Cloud Billing IAM para la relación financiera.

- **A** concede a finanzas privilegios técnicos innecesarios.
- **B** concede a desarrolladores control financiero amplio y elimina la separación.
- **D** confunde Google payments con la jerarquía de recursos de Google Cloud.
