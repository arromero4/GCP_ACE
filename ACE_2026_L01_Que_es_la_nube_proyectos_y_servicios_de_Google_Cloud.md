# ACE 2026 - Lección 01: Qué es la nube, proyectos y servicios de Google Cloud

**Fecha del plan:** 21 de septiembre de 2026  
**Fecha de regeneración:** 24 de septiembre de 2026  
**Guía oficial:** Base y sección 1.1, *Setting up cloud projects and accounts*  
**Práctica del calendario:** dibujar la relación entre organización, carpeta, proyecto y recurso; completar un diagnóstico original de 10 preguntas  
**Duración sugerida:** 60-90 minutos  
**Idioma:** explicación en español; términos de Google Cloud y preguntas de examen en inglés

> Esta es una reconstrucción desde cero. Completar la lectura o recibir una puntuación alta en el diagnóstico no demuestra por sí solo dominio del tema. Conserva evidencias y registra los errores para repasarlos.

> **Proyecto de práctica:** SiteOps Tracker es un caso completamente ficticio de portafolio para auditar infraestructura de TI en sedes ficticias. Usa React + TypeScript, Node.js + TypeScript y PostgreSQL; no representa ni reutiliza datos, arquitectura o requisitos de proyectos internos o privados.

---

## 1. Objetivo de aprendizaje

Al terminar esta lección podrás:

1. Explicar con tus propias palabras qué significa usar computación en la nube.
2. Diferenciar **service**, **API** y **resource**.
3. Dibujar la jerarquía **Organization > Folder > Project > Service resource**.
4. Explicar por qué el proyecto funciona como contenedor de servicios, límite de confianza, unidad de costos y punto de aplicación de políticas.
5. Distinguir **project name**, **project ID** y **project number**.
6. Identificar, de forma preliminar, servicios apropiados para SiteOps Tracker y descartar alternativas sobredimensionadas o incompatibles.
7. Inspeccionar proyectos y APIs habilitadas mediante la consola y comandos de solo lectura.
8. Establecer una línea base mediante 10 preguntas originales en inglés.

### Evidencia mínima del día

La lección queda trabajada cuando puedas mostrar estas cuatro evidencias:

- [ ] Un diagrama propio de la jerarquía de SiteOps Tracker.
- [ ] Una tabla que relacione al menos tres necesidades con un servicio y un recurso.
- [ ] La salida o simulación razonada de los comandos de solo lectura.
- [ ] El diagnóstico contestado y una ficha por cada error o respuesta insegura.

---

## 2. Prerrequisitos explicados desde cero

No necesitas experiencia previa en Google Cloud. Para la ruta práctica hay dos modalidades.

### Modalidad A: con acceso a Google Cloud

Necesitas:

- Una cuenta de Google Cloud o un proyecto temporal de Google Skills.
- Permiso para **ver** al menos un proyecto. No necesitas ser Owner ni crear recursos.
- Un navegador para abrir la Google Cloud console.
- Cloud Shell, que ya incluye Google Cloud CLI (`gcloud`). Google administra la VM temporal de Cloud Shell; no aparece como una VM dentro de tu proyecto. Cloud Shell es gratuito para usuarios con una cuenta de Google Cloud, sujeto a sus límites de uso.

### Modalidad B: sin cuenta, crédito o permisos

Puedes completar íntegramente el aprendizaje conceptual con el caso simulado de esta lección. No crees una cuenta de facturación ni introduzcas una tarjeta solo para realizar el Día 1.

### Tres palabras de terminal

- **Command / comando:** la acción principal, por ejemplo `gcloud projects list`.
- **Flag / opción:** modifica la acción y comienza normalmente con `--`, por ejemplo `--limit=5`.
- **Output / salida:** información que devuelve el comando. Leer una salida no significa modificar el entorno.

---

## 3. Qué es la computación en la nube

La computación en la nube permite consumir capacidad tecnológica a través de una red: procesamiento, almacenamiento, bases de datos, redes, observabilidad y otros servicios. En vez de comprar y operar todo el hardware físico, solicitas capacidades al proveedor y eliges cuánto control conservas.

### Analogía: de un centro de datos propio a un edificio administrado

Imagina que tu aplicación necesita oficinas:

- **On-premises:** compras el terreno, construyes el edificio, mantienes electricidad, seguridad, elevadores y oficinas.
- **Virtual machines en la nube:** Google mantiene el edificio físico, pero tú administras buena parte de tu oficina, incluido el sistema operativo y el software.
- **Managed service:** Google también administra más elementos de la plataforma. Tú configuras el servicio y tus datos.
- **Serverless application platform:** entregas código o un contenedor y el proveedor administra gran parte del aprovisionamiento y escalado de infraestructura.

"Nube" no significa "sin servidores". Significa que la propiedad y la operación de diferentes capas se distribuyen entre el proveedor y el cliente. Tus responsabilidades nunca desaparecen: siguen existiendo decisiones de identidad, permisos, datos, configuración, costos y arquitectura.

### Beneficios que debes relacionar con decisiones, no memorizar como publicidad

- **Elasticity / elasticidad:** aumentar o reducir capacidad ante cambios de demanda.
- **Pay for usage / pago por uso:** muchos servicios cobran por recursos o consumo, aunque cada producto tiene su propio modelo.
- **Managed operations / operación administrada:** delegar tareas repetitivas al proveedor.
- **Global infrastructure / infraestructura global:** elegir ubicaciones según latencia, disponibilidad, residencia de datos y oferta del producto.
- **Automation / automatización:** repetir operaciones con CLI, APIs e infraestructura como código.

### Responsabilidad compartida, versión inicial

Google protege y opera la infraestructura del servicio según el producto. El cliente sigue siendo responsable de sus usuarios, permisos, datos y configuraciones. Cuanto más administrado sea un servicio, más tareas operativas delegas; no delegas la decisión de quién debe acceder ni qué datos debe guardar la aplicación.

---

## 4. Service, API y resource: no son sinónimos

Esta distinción aparece continuamente en Google Cloud.

| Concepto | Pregunta que responde | Ejemplo |
|---|---|---|
| **Service / servicio** | ¿Qué capacidad ofrece Google Cloud? | Compute Engine ofrece infraestructura de máquinas virtuales. |
| **API** | ¿Qué interfaz se habilita en un proyecto para consumir o administrar esa capacidad? | Compute Engine API. |
| **Resource / recurso** | ¿Qué objeto concreto existe y administras? | Una VM llamada `audit-api-vm-01`. |

Otros ejemplos:

| Servicio | API o familia de API | Recurso posible |
|---|---|---|
| Cloud Run | Cloud Run Admin API | Un servicio HTTP desplegado |
| Cloud Storage | Cloud Storage APIs | Un bucket y sus objetos |
| Cloud SQL | Cloud SQL Admin API | Una instancia de PostgreSQL |
| Google Kubernetes Engine (GKE) | Kubernetes Engine API | Un clúster |

**Regla mental:** habilitar una API abre la puerta administrativa para un proyecto; no crea automáticamente una VM, un clúster ni una base de datos. Crear el recurso es otra acción y puede generar costos.

---

## 5. La jerarquía de recursos de Google Cloud

La estructura general es:

```text
Organization (raíz administrada)
└── Folder (opcional; puede contener subcarpetas)
    └── Project (unidad operativa fundamental)
        ├── Compute resource
        ├── Storage resource
        ├── Database resource
        └── Networking / observability / security resources
```

Todos los recursos salvo el nivel superior tienen exactamente un padre dentro de la jerarquía. Las carpetas son opcionales. Una cuenta personal puede tener un proyecto sin organización como nivel superior; una organización administrada permite usar carpetas y gobierno central.

### 5.1 Organization resource

Representa a una entidad, por ejemplo una empresa. Es la raíz administrada y permite gobierno central. Las políticas y asignaciones de acceso aplicadas en un ancestro pueden heredarse hacia sus descendientes.

Una organización normalmente se relaciona con Google Workspace o Cloud Identity. No confundas la **Organization resource** con una simple cuenta de usuario.

### 5.2 Folder resource

Es un agrupador opcional entre la organización y los proyectos. Puede representar departamentos, equipos, aplicaciones o entornos y puede contener otras carpetas.

Una carpeta es útil cuando una regla o responsabilidad debe abarcar varios proyectos. No es un contenedor para desplegar directamente una VM o una base de datos.

### 5.3 Project resource

Es la unidad operativa fundamental. Un proyecto funciona como:

1. **Service container:** allí se habilitan APIs y se crean la mayoría de los recursos de servicio.
2. **Trust boundary:** ayuda a aislar aplicaciones o entornos; de forma predeterminada, los recursos de un proyecto no obtienen acceso a los de otro solo por existir en la misma organización.
3. **Billing unit:** permite atribuir y separar costos.
4. **Policy attachment point:** puede recibir permisos y políticas, además de heredar configuraciones de sus ancestros.
5. **Quota context:** muchas cuotas y límites se evalúan por proyecto, servicio o ubicación; el detalle varía por producto.

### 5.4 Service resource

Es el objeto que realmente ejecuta o almacena algo: una VM, bucket, base de datos, red, clúster o servicio de Cloud Run. Muchos recursos pertenecen a un proyecto, aunque su alcance técnico puede ser global, regional o zonal.

### Importante: Billing account no es el padre del proyecto

Una **Cloud Billing account** se vincula a uno o más proyectos para pagar su consumo. No reemplaza a Organization o Folder y no forma la cadena de padres de Resource Manager.

```text
Jerarquía: Organization → Folder → Project → Resource
Facturación: Billing account ──vínculo de pago──> Project
```

---

## 6. Project name, project ID y project number

| Identificador | Quién lo define | ¿Cambia? | Uso principal |
|---|---|---|---|
| **Project name** | Persona o proceso creador | Sí, es editable | Lectura humana en la consola |
| **Project ID** | Se elige al crear; debe ser globalmente único | No después de la creación | CLI, APIs y nombres de muchos recursos |
| **Project number** | Google lo genera | No; es de solo lectura | Referencias internas, identidades y algunas configuraciones |

Ejemplo ficticio:

```text
Project name: SiteOps Tracker - Desarrollo
Project ID: siteops-tracker-dev-2026
Project number: 123456789012
```

No incluyas datos personales, secretos ni información de seguridad en nombres o IDs. El Project ID puede quedar visible en nombres y referencias de otros recursos.

### Error frecuente de examen

El nombre visible no es necesariamente el valor que acepta un comando. Cuando un comando solicita `PROJECT_ID`, no uses automáticamente el nombre descriptivo.

---

## 7. Caso aplicado: SiteOps Tracker

SiteOps Tracker registra sedes ficticias, activos de TI, auditorías, hallazgos, estados, responsables, evidencias e historial. Su frontend usa React + TypeScript, su API usa Node.js + TypeScript y sus datos operativos se modelan en PostgreSQL. Una jerarquía inicial razonable podría ser:

```text
Organization: empresa.example
└── Folder: Technology
    └── Folder: Site-Operations
        ├── Project: siteops-tracker-dev
        │   ├── Cloud Run service: audit-api-dev
        │   ├── Cloud SQL instance: audit-postgres-dev
        │   └── Cloud Storage bucket: audit-evidence-dev
        └── Project: siteops-tracker-prod
            ├── Cloud Run service: audit-api-prod
            ├── Cloud SQL instance: audit-postgres-prod
            └── Cloud Storage bucket: audit-evidence-prod
```

### ¿Por qué separar desarrollo y producción?

- Permite separar permisos y exposición al riesgo.
- Facilita atribuir costos por entorno.
- Reduce la probabilidad de que una prueba afecte producción.
- Permite administrar APIs, cuotas y configuración por proyecto.

No significa que dos proyectos jamás puedan comunicarse. Esa comunicación debe diseñarse y autorizarse explícitamente.

---

## 8. Decisión preliminar entre servicios

Hoy solo construirás el razonamiento inicial; los servicios se estudiarán con profundidad en lecciones posteriores.

| Necesidad de SiteOps Tracker | Elección inicial | Por qué encaja | Por qué no elegir las alternativas todavía |
|---|---|---|---|
| API Node.js/TypeScript en contenedor, tráfico variable y equipo pequeño | **Cloud Run service** | Plataforma de aplicaciones totalmente administrada; adecuada para endpoints HTTP y APIs; reduce administración de infraestructura. | **Compute Engine** exige administrar más elementos de la VM y el SO. **GKE** aporta Kubernetes y control de clúster, pero añade conceptos y operación innecesarios para una API sencilla. |
| Sedes, auditorías, hallazgos y relaciones con transacciones; PostgreSQL ya es parte del proyecto | **Cloud SQL for PostgreSQL** | Base de datos relacional administrada y compatible con PostgreSQL. | **Firestore** es una base documental y exigiría rediseñar el modelo. **Cloud Storage** almacena objetos, no sustituye una base relacional transaccional. **BigQuery** se orienta a análisis, no a la base operacional principal de esta aplicación. |
| Evidencias como PDF, imágenes y exportaciones | **Cloud Storage** | Almacenamiento de objetos durable para archivos. | Guardar archivos grandes como filas de la base puede aumentar complejidad y costo. **Persistent Disk** está ligado a patrones de VM y no es el servicio de objetos compartido que se necesita aquí. |

### Cuándo sí considerar las alternativas

- **Compute Engine:** cuando necesitas control del sistema operativo, software legado o configuración de host específica.
- **GKE:** cuando necesitas explícitamente Kubernetes, control de la plataforma de contenedores, patrones multi-servicio complejos o configuración detallada de red, escalado y seguridad.
- **Firestore:** cuando el modelo documental, sincronización para aplicaciones o escalado serverless encajan mejor que un esquema relacional.

La respuesta ACE no debe partir de "mi servicio favorito", sino de los requisitos: control operativo, tipo de datos, patrón de tráfico, disponibilidad, seguridad y costo.

---

## 9. Práctica guiada

### Reglas de seguridad antes de empezar

- Usa preferentemente un proyecto temporal de Google Skills o un proyecto donde tengas permiso de lectura.
- Confirma el proyecto seleccionado antes de cada comando.
- No habilites APIs, no crees recursos y no cambies IAM en esta lección.
- No copies tokens, claves, correos personales ni datos sensibles en tu evidencia.
- Los comandos siguientes fueron contrastados con la referencia oficial de Google Cloud disponible el 22 de septiembre de 2026.

### Parte A - Dibuja la jerarquía

En papel o Markdown, completa este molde:

```text
Organization o "No organization": __________________________
└── Folder o "No aplica": __________________________________
    ├── Project de desarrollo: ______________________________
    │   ├── Recurso de cómputo previsto: ____________________
    │   ├── Recurso de datos previsto: ______________________
    │   └── Recurso de objetos previsto: ____________________
    └── Project de producción: ______________________________
        ├── Recurso de cómputo previsto: ____________________
        ├── Recurso de datos previsto: ______________________
        └── Recurso de objetos previsto: ____________________
```

Debajo del dibujo, responde:

1. ¿Qué nivel usarías para separar desarrollo y producción?
2. ¿En qué nivel se habilita normalmente una API?
3. ¿Qué elemento se vincula al proyecto para pagar consumo, pero no es su padre jerárquico?
4. Si una regla debe abarcar todos los proyectos de Site-Operations, ¿qué ancestro sería un punto lógico para evaluarla?

### Parte B - Exploración de consola, solo lectura

1. Abre [Manage Resources](https://console.cloud.google.com/cloud-resource-manager).
2. Selecciona únicamente un proyecto que reconozcas y donde tengas autorización.
3. Identifica, si están disponibles, **Organization**, **Folder**, **Project name** y **Project ID**. Una cuenta personal puede mostrar `No organization`.
4. Abre el panel del proyecto y registra sin datos sensibles:
   - Project name
   - Project ID
   - Project number
   - Parent visible, si existe
5. Ve a **APIs & Services > Enabled APIs & services** y observa dos APIs habilitadas. No pulses **Enable**.
6. Anota un ejemplo con la forma: `servicio → API habilitada → recurso posible`.

La ubicación visual de menús puede cambiar. Usa el buscador de la consola si la etiqueta no coincide exactamente.

### Parte C - Exploración con CLI, solo lectura

Activa Cloud Shell desde la consola. Ejecuta cada bloque por separado y lee la salida antes de continuar.

#### 1. Ver el proyecto configurado en la CLI

```bash
gcloud config list project
```

Si aparece `(unset)`, no elijas un proyecto al azar. Continúa con la lista de proyectos accesibles.

#### 2. Listar hasta cinco proyectos visibles para la cuenta activa

```bash
gcloud projects list --sort-by=projectId --limit=5
```

Elige un `PROJECT_ID` que reconozcas. No uses el nombre descriptivo si es diferente.

#### 3. Definir una variable local y describir el proyecto

Reemplaza el valor ficticio antes de ejecutar el bloque:

```bash
export PROJECT_ID="replace-with-an-authorized-project-id"
gcloud projects describe "$PROJECT_ID"
```

Busca en la salida el ID, número, nombre, estado de ciclo de vida y parent, si tienes permiso para verlo.

#### 4. Listar hasta veinte servicios habilitados en ese proyecto

```bash
gcloud services list --enabled --project="$PROJECT_ID" --limit=20
```

`--enabled` consulta los servicios habilitados; no habilita nada. El comando puede requerir permisos de lectura que tu cuenta no tenga.

#### 5. Cerrar la variable local al terminar

```bash
unset PROJECT_ID
```

Esto solo elimina la variable de la sesión; no modifica Google Cloud.

### Parte D - Tabla de evidencia

Completa sin pegar información sensible:

| Comprobación | Evidencia observada o simulada | Qué demuestra |
|---|---|---|
| Proyecto configurado | | Diferencia entre contexto de CLI y existencia del proyecto |
| Proyecto accesible | | La cuenta solo lista proyectos para los que tiene visibilidad suficiente |
| Metadatos del proyecto | | Diferencia entre name, ID y number |
| API habilitada | | La API está disponible para el proyecto, pero no prueba que exista un recurso de ese servicio |
| Recurso posible | | Diferencia entre servicio y objeto concreto |

---

## 10. Alternativa conceptual completa, sin cuenta ni crédito

Usa este entorno ficticio:

```text
Organization: northstar.example
Folder: Engineering
Subfolder: Site-Operations
Project name: SiteOps Tracker - Dev
Project ID: siteops-tracker-dev-2026
Project number: 123456789012
Enabled services:
- run.googleapis.com
- sqladmin.googleapis.com
- storage.googleapis.com
Planned resources:
- Cloud Run service audit-api-dev
- Cloud SQL for PostgreSQL instance audit-db-dev
- Cloud Storage bucket audit-evidence-dev
```

Realiza estas tareas:

1. Dibuja la jerarquía y añade un proyecto de producción como hermano del proyecto de desarrollo.
2. Marca cuáles líneas son **service/API** y cuáles son **resources**.
3. Explica por qué `123456789012` no es el Project ID.
4. Explica por qué ver `run.googleapis.com` no demuestra que `audit-api-dev` exista.
5. Decide dónde vincularías conceptualmente la facturación y explica por qué no la dibujas como padre.
6. Completa la tabla de evidencia usando la palabra `simulada`.

Esta modalidad cubre íntegramente el objetivo del Día 1. La experiencia de consola y CLI se retomará en la Lección 2.

---

## 11. Resultado esperado

Al finalizar deberías poder producir una explicación parecida a esta, sin memorizarla literalmente:

> Google Cloud organiza el gobierno mediante una jerarquía. La Organization es la raíz administrada, las Folders agrupan y los Projects contienen servicios habilitados y la mayoría de los recursos. Un proyecto también ayuda a separar confianza, costos y políticas. La Billing account paga el consumo de proyectos, pero no es un padre de Resource Manager. Para SiteOps Tracker empezaría con proyectos distintos para desarrollo y producción; usaría Cloud Run para la API, Cloud SQL for PostgreSQL para datos relacionales y Cloud Storage para evidencias. Esa selección todavía debe validarse contra requisitos de disponibilidad, seguridad y costo.

Tu práctica es satisfactoria si el diagrama, la tabla y esa explicación no confunden los siguientes pares:

- Folder vs. Project
- Billing account vs. Organization
- Project name vs. Project ID
- API habilitada vs. recurso creado
- Servicio administrado vs. ausencia de responsabilidad del cliente

---

## 12. Solución de problemas

### `gcloud: command not found`

- Usa Cloud Shell, donde `gcloud` ya está instalado.
- Si trabajas localmente, instala Google Cloud CLI desde la documentación oficial; no descargues ejecutables de fuentes no verificadas.

### `project` aparece como `(unset)`

No es una falla del proyecto. Significa que la configuración activa de `gcloud` no tiene un proyecto predeterminado. Usa `--project="$PROJECT_ID"` de forma explícita en el comando de consulta.

### `PERMISSION_DENIED` o `The caller does not have permission`

- Confirma que el ID sea correcto.
- Confirma que la cuenta tenga permiso de lectura.
- No intentes otorgarte roles ni pedir Owner para completar esta práctica.
- Usa el caso conceptual si el proyecto es corporativo o de laboratorio y no tienes visibilidad.

### `NOT_FOUND` al describir el proyecto

Normalmente se escribió el Project name en vez del Project ID, existe un error tipográfico, el proyecto fue eliminado o la cuenta no puede verlo.

### No aparece una Organization o Folder

Puede ser normal en una cuenta personal o porque no tienes permiso para ver el ancestro. Escribe `No organization visible` o `Parent not visible`; no inventes un ID.

### La lista de proyectos parece incompleta

`gcloud projects list` devuelve proyectos activos visibles para la cuenta según sus permisos. Con ciertas formas de acceso mediante service accounts la lista también puede ser incompleta. No concluyas que un proyecto no existe solo porque tu cuenta no lo lista.

### `gcloud services list` no muestra lo mismo que esperabas

- Verifica `--project` y el Project ID.
- Comprueba que usaste `--enabled`, no `--available`.
- Puede faltar permiso para consultar Service Usage.
- La presencia de una API no implica que haya recursos desplegados.

### Cloud Shell no está disponible

Una organización puede deshabilitarlo. Completa la ruta de consola o la alternativa conceptual; no eludas las políticas corporativas.

---

## 13. Costos y limpieza de recursos

### Impacto de costos de esta práctica

- Los comandos indicados son consultas de solo lectura y no crean recursos facturables.
- Cloud Shell es gratuito para usuarios con una cuenta de Google Cloud, sujeto a límites.
- No se pide crear proyecto, vincular facturación, habilitar APIs ni desplegar servicios.
- Habilitar una API y consumir un servicio son acciones distintas; los costos dependen del uso y del modelo de precios de cada producto.

### Limpieza

1. Ejecuta `unset PROJECT_ID` si definiste la variable.
2. Cierra Cloud Shell cuando termines.
3. No elimines un proyecto corporativo ni de Google Skills como "limpieza".
4. Si te apartaste de las instrucciones y creaste un recurso, identifica primero el producto, propietario y proyecto. Elimínalo solo si estás autorizado y verifica después que dejó de existir. No borres todo el proyecto para corregir un recurso aislado.

**Resultado de limpieza esperado:** cero recursos nuevos y cero cambios de configuración en Google Cloud.

---

## 14. Glosario bilingüe

| English | Español | Significado práctico |
|---|---|---|
| Cloud computing | Computación en la nube | Consumo remoto de capacidades tecnológicas administradas en distintos niveles |
| Resource hierarchy | Jerarquía de recursos | Cadena de gobierno entre organización, carpetas, proyectos y recursos |
| Organization resource | Recurso de organización | Raíz administrada de una entidad |
| Folder resource | Recurso de carpeta | Agrupador opcional para gobierno y delegación |
| Project resource | Recurso de proyecto | Unidad operativa que contiene servicios y recursos |
| Service | Servicio | Capacidad o producto ofrecido por Google Cloud |
| API | Interfaz de programación de aplicaciones | Interfaz que permite consumir o administrar un servicio |
| Enable an API | Habilitar una API | Permitir que un proyecto use la interfaz de un servicio |
| Service resource | Recurso de servicio | Objeto concreto, como VM, bucket o base de datos |
| Parent | Padre | Ancestro inmediato de un recurso en la jerarquía |
| Inheritance | Herencia | Aplicación descendente de políticas o acceso desde ancestros |
| Trust boundary | Límite de confianza | Frontera de aislamiento y administración de acceso |
| Billing account | Cuenta de facturación | Entidad que paga el consumo de proyectos vinculados |
| Project name | Nombre del proyecto | Etiqueta humana editable |
| Project ID | ID del proyecto | Identificador global único e inmutable tras crear el proyecto |
| Project number | Número del proyecto | Identificador numérico generado por Google |
| Managed service | Servicio administrado | Producto donde Google opera más capas de la plataforma |
| IaaS | Infraestructura como servicio | Modelo con control de VM e infraestructura lógica; ejemplo: Compute Engine |
| Serverless | Sin administración directa de servidores | Modelo donde el proveedor abstrae gran parte del aprovisionamiento y escalado |
| Quota | Cuota | Límite de uso o capacidad aplicado según el servicio y alcance |
| Least privilege | Mínimo privilegio | Conceder solo los permisos necesarios |

---

## 15. Diagnóstico original - 10 preguntas en inglés

**Instrucciones:** responde sin consultar la clave. A menos que se indique lo contrario, selecciona una sola respuesta. No son preguntas filtradas ni copiadas del examen. Están diseñadas para medir tu línea base.

### Question 1

A company wants all of its Google Cloud projects to remain under company control when an employee leaves. It also wants a central point from which policies can be inherited by all projects. Which resource should be at the top of the hierarchy?

A. A Cloud Billing account  
B. A Compute Engine instance  
C. An Organization resource  
D. A Cloud Shell session

### Question 2

The SiteOps Tracker team needs development and production to have separate access controls, cost attribution, and failure boundaries. What is the best starting design?

A. Put both environments in one project and distinguish them only by VM names  
B. Use separate projects for development and production under an appropriate folder  
C. Put each environment in a different zone of the same project  
D. Use two billing accounts as the resource parents

### Question 3

Which statement correctly describes Google Cloud project identifiers?

A. The project name is globally unique and cannot be changed  
B. The project ID is generated as a read-only number  
C. The project number is selected by the user and can be edited later  
D. The project name is human-readable, the project ID is globally unique, and Google generates the project number

### Question 4

An engineer must see which services are enabled in project `audit-dev-123` without changing the project. Which command should the engineer run?

A. `gcloud services enable --project=audit-dev-123`  
B. `gcloud services list --enabled --project=audit-dev-123`  
C. `gcloud projects create audit-dev-123`  
D. `gcloud projects delete audit-dev-123`

### Question 5

The Compute Engine API is enabled in a project, but no VMs have been created. Which statement is correct?

A. Enabling the API automatically creates one default VM  
B. The API is a resource and Compute Engine is its parent folder  
C. The service can be administered through its API, but a VM is a separate resource that must be created  
D. The project must already contain a GKE cluster

### Question 6

A small team has a containerized Node.js HTTP API with variable traffic. The team wants to minimize infrastructure and cluster administration. Which service is the best initial choice?

A. Cloud Run  
B. Google Kubernetes Engine because every container requires a cluster  
C. Compute Engine because serverless platforms cannot run containers  
D. Cloud Storage because it executes HTTP application code

### Question 7

SiteOps Tracker stores sites, audits, findings, and status history with relationships and transactions. The application already uses PostgreSQL. Which managed service is the best initial fit for the operational database?

A. Cloud Storage  
B. Cloud SQL for PostgreSQL  
C. Google Kubernetes Engine  
D. Firestore solely because it is serverless

### Question 8 - Select two answers

Which two statements about the Google Cloud resource hierarchy are correct?

A. Every project must have a Folder as its immediate parent  
B. Folders are optional grouping resources and can contain projects  
C. Projects contain service resources such as VMs and buckets  
D. A Cloud Billing account is the hierarchical parent of all projects it pays for  
E. A resource can have several immediate parents at the same time

### Question 9

A developer has access to a project because an IAM role was granted on the project's parent folder. What concept best explains the effective access?

A. Policy inheritance  
B. Project ID mutability  
C. API auto-enablement  
D. Billing export

### Question 10

The application must store PDF reports and device photos as durable objects. The files are not relational rows and do not execute code. Which service is the best initial fit?

A. Cloud Storage  
B. Cloud SQL for PostgreSQL  
C. Compute Engine instance metadata  
D. GKE control plane

---

## 16. Soluciones justificadas

### Answer 1: C - An Organization resource

La Organization representa a la entidad y es la raíz desde la que se puede centralizar propiedad y herencia.

- **A is incorrect:** una Billing account paga consumo, pero no es la raíz de Resource Manager.
- **B is incorrect:** una VM es un recurso de servicio dentro de un proyecto.
- **D is incorrect:** Cloud Shell es una sesión de herramientas, no un nivel de gobierno.

### Answer 2: B - Separate projects

Los proyectos separados proporcionan límites claros para acceso, costos y cambios de cada entorno.

- **A is incorrect:** los nombres ayudan a identificar, pero no crean una frontera equivalente.
- **C is incorrect:** una zona es una ubicación de infraestructura, no el límite administrativo solicitado.
- **D is incorrect:** Billing accounts no son padres jerárquicos y no sustituyen el aislamiento por proyecto.

### Answer 3: D

El nombre es legible y editable; el Project ID es globalmente único e inmutable después de crear el proyecto; el Project number lo genera Google.

- **A is incorrect:** el nombre no tiene que ser único y puede cambiar.
- **B is incorrect:** confunde Project ID con Project number.
- **C is incorrect:** el número no lo selecciona el usuario ni es editable.

### Answer 4: B - `gcloud services list --enabled`

Es una consulta de servicios habilitados para el proyecto indicado.

- **A is incorrect:** `enable` cambia el estado del proyecto y además requiere especificar qué servicio habilitar.
- **C is incorrect:** crea un proyecto en vez de inspeccionar uno existente.
- **D is incorrect:** solicita la eliminación del proyecto; es destructivo y no responde a la necesidad.

### Answer 5: C

La API habilitada permite administrar o consumir el servicio; una instancia de VM es un recurso independiente que aún no existe.

- **A is incorrect:** habilitar la API no aprovisiona una VM predeterminada.
- **B is incorrect:** una API no es una carpeta padre de Compute Engine.
- **D is incorrect:** Compute Engine no requiere que exista un clúster GKE.

### Answer 6: A - Cloud Run

Cloud Run es una plataforma totalmente administrada apropiada para servicios HTTP en contenedores y reduce la operación de infraestructura.

- **B is incorrect:** un contenedor no exige por definición un clúster Kubernetes; GKE añade control y complejidad que el escenario no solicita.
- **C is incorrect:** Cloud Run ejecuta código y contenedores.
- **D is incorrect:** Cloud Storage almacena objetos; no ejecuta una API Node.js.

### Answer 7: B - Cloud SQL for PostgreSQL

El requisito es relacional, transaccional y compatible con PostgreSQL; Cloud SQL es un servicio administrado para ese motor.

- **A is incorrect:** Cloud Storage no ofrece una base relacional operacional.
- **C is incorrect:** GKE ejecuta contenedores; no es por sí mismo una base de datos administrada.
- **D is incorrect:** Firestore puede ser válido para modelos documentales, pero "serverless" no anula el requisito relacional ni justifica una migración de modelo.

### Answer 8: B and C

Folders son agrupadores opcionales y Projects contienen recursos de servicio.

- **A is incorrect:** un proyecto no siempre tiene una Folder; puede depender directamente de una Organization o ser el nivel superior en ciertos contextos.
- **D is incorrect:** la Billing account se vincula para pagar, no es un padre jerárquico.
- **E is incorrect:** un recurso tiene un solo padre inmediato en la jerarquía.

### Answer 9: A - Policy inheritance

Una asignación en la Folder puede heredarse por los proyectos descendientes.

- **B is incorrect:** el Project ID no se vuelve editable por herencia.
- **C is incorrect:** heredar un rol no habilita automáticamente todas las APIs.
- **D is incorrect:** la exportación de facturación es un flujo de datos de costos, no un mecanismo de acceso.

### Answer 10: A - Cloud Storage

Los PDF y las fotos son objetos, por lo que Cloud Storage es el ajuste inicial natural.

- **B is incorrect:** Cloud SQL administra datos relacionales; usarlo como repositorio principal de archivos no satisface el patrón descrito.
- **C is incorrect:** los metadatos de VM no son un almacén de evidencias de aplicación.
- **D is incorrect:** el control plane de GKE administra un clúster; no es almacenamiento de objetos.

### Puntuación diagnóstica

Asigna un punto por pregunta. En la pregunta 8, asigna el punto solo si elegiste **B y C**, sin opciones adicionales.

| Puntuación | Interpretación de línea base | Acción |
|---:|---|---|
| 0-4 | Los fundamentos necesitan construcción cuidadosa | Repite secciones 4-8 y rehace el diagrama sin mirar |
| 5-7 | Hay fundamentos parciales, con confusiones relevantes | Crea ficha para cada error o respuesta insegura |
| 8-10 | Buena línea base de esta introducción | Explica cada distractor; no lo tomes como dominio comprobado del ACE |

La puntuación de hoy no predice por sí sola el resultado del examen.

---

## 17. Repaso activo y calendario espaciado

### Días hábiles previos

No existen lecciones hábiles anteriores en este plan: esta es la Lección 1. Por tanto, hoy no corresponde inventar un tema previo.

### Hace 7 días

El 14 de septiembre de 2026 todavía no había empezado este plan. No hay contenido programado para recuperar.

### Hace 21 días

El 31 de agosto de 2026 todavía no había empezado este plan. No hay contenido programado para recuperar.

### Recuperación inicial de hoy

Cierra el archivo durante cinco minutos y responde de memoria:

1. ¿Cuál es la cadena jerárquica de cuatro niveles?
2. ¿Por qué una Billing account no debe dibujarse como padre de un Project?
3. ¿Cuál es la diferencia entre API habilitada y recurso creado?
4. ¿Qué identificador es editable y cuáles no?
5. ¿Por qué Cloud Run + Cloud SQL + Cloud Storage es una hipótesis razonable para SiteOps Tracker?

**Próximas recuperaciones de esta lección:**

- Próximo día hábil: breve recuerdo antes de la Lección 2.
- 28 de septiembre de 2026: recuperación de 7 días integrada en la Lección 6.
- 12 de octubre de 2026: recuperación de 21 días integrada en la Lección 16.

---

## 18. Ficha de errores

Completa una ficha por cada error y también por cada acierto con baja confianza.

```text
Fecha: 21/09/2026
Lección: L01
Pregunta o paso:
Mi respuesta:
Respuesta correcta o resultado esperado:
Tipo de error:
[ ] Concepto  [ ] Lectura en inglés  [ ] Confundí servicios
[ ] Confundí alcance/jerarquía  [ ] Comando/flag  [ ] Adiviné
Qué palabra del escenario ignoré:
Regla corregida en una frase:
Por qué fallan los distractores:
Ejemplo aplicado a SiteOps Tracker:
Acción de refuerzo:
Revisar: siguiente día hábil / 7 días / 21 días
```

### Errores prioritarios de esta lección

Marca si apareció alguno:

- [ ] Dibujé Billing account como padre del proyecto.
- [ ] Confundí Project name con Project ID.
- [ ] Supuse que habilitar una API crea un recurso.
- [ ] Elegí GKE solo porque la aplicación usa contenedores.
- [ ] Elegí una base documental ignorando requisitos relacionales.
- [ ] Interpreté falta de permiso como inexistencia del recurso.

---

## 19. Criterios de autoevaluación

Usa la escala 0-2 en cada criterio.

- **0:** no puedo hacerlo todavía.
- **1:** puedo hacerlo con notas o cometo imprecisiones.
- **2:** puedo explicarlo sin notas y justificarlo con un escenario.

| Criterio | 0 | 1 | 2 |
|---|---:|---:|---:|
| Defino computación en la nube sin decir que "no existen servidores" | | | |
| Distingo service, API y resource | | | |
| Dibujo Organization > Folder > Project > Resource | | | |
| Explico que Folder es opcional | | | |
| Explico los cuatro papeles principales del Project | | | |
| Distingo project name, ID y number | | | |
| Separo Billing account de la jerarquía | | | |
| Justifico tres servicios para SiteOps Tracker | | | |
| Explico por qué no elegir las alternativas | | | |
| Ejecuto o simulo correctamente las consultas de solo lectura | | | |

**Total máximo:** 20.

- **0-11:** repite el diagrama y los pares que confundiste.
- **12-16:** continúa, pero lleva los criterios con 0 o 1 a la ficha de errores.
- **17-20:** buena evidencia inicial; confirma la retención en los repasos posteriores.

Una puntuación alta no autoriza a saltar la siguiente lección ni demuestra dominio completo del dominio 1.

---

## 20. Actividad paralela de Google Skills

Continúa la **siguiente actividad pendiente**, sin saltar módulos, del path [Getting Started with Google Cloud](https://www.skills.google/paths/8).

Registra:

```text
Nombre exacto de la actividad:
Estado: no iniciada / en progreso / completada
Evidencia:
Concepto ACE relacionado:
Duda o error para reforzar mañana:
```

La ruta de Skills complementa esta lección; completar una actividad no sustituye la evidencia ni el diagnóstico.

---

## 21. Documentación oficial consultada

Referencias verificadas el **22 de septiembre de 2026**:

- [Associate Cloud Engineer Certification Exam Guide](https://cloud.google.com/learn/certification/guides/cloud-engineer)
- [About resource hierarchy](https://docs.cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy)
- [Create and understand projects](https://docs.cloud.google.com/resource-manager/docs/creating-managing-projects)
- [How Cloud Shell works](https://docs.cloud.google.com/shell/docs/how-cloud-shell-works)
- [Cloud Shell pricing](https://cloud.google.com/shell/pricing)
- [`gcloud config list`](https://docs.cloud.google.com/sdk/gcloud/reference/config/list)
- [`gcloud projects list`](https://docs.cloud.google.com/sdk/gcloud/reference/projects/list)
- [`gcloud projects describe`](https://docs.cloud.google.com/sdk/gcloud/reference/projects/describe)
- [`gcloud services list`](https://docs.cloud.google.com/sdk/gcloud/reference/services/list)
- [What is Cloud Run](https://docs.cloud.google.com/run/docs/overview/what-is-cloud-run)
- [Compute Engine overview](https://docs.cloud.google.com/compute/docs/overview)
- [GKE overview](https://docs.cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview)
- [Cloud SQL for PostgreSQL overview](https://docs.cloud.google.com/sql/docs/postgres/introduction)
- [Firestore overview](https://cloud.google.com/products/firestore)
- [Cloud Storage overview](https://docs.cloud.google.com/storage/docs/introduction)

---

## 22. Cierre de la lección

Antes de marcarla como trabajada, completa:

```text
Diagrama terminado: sí / no
Modalidad: cuenta real / Google Skills / conceptual
Diagnóstico: ___ / 10
Número de respuestas inseguras aunque fueran correctas: ___
Errores registrados: ___
Autoevaluación: ___ / 20
Recurso nuevo creado: no (resultado esperado)
Pregunta que todavía no puedo explicar:
```

**Idea esencial:** en Google Cloud, una buena decisión empieza por ubicar correctamente el alcance. Primero identifica Organization, Folder, Project y Resource; después elige y configura servicios.
