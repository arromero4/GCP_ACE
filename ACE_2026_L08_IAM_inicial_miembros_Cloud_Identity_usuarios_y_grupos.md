# Google Cloud Associate Cloud Engineer (ACE) 2026

## Lección 08 - IAM inicial, miembros, Cloud Identity, usuarios y grupos

**Fecha del plan:** 30 de septiembre de 2026  
**Cobertura de la guía oficial:** 1.1 *Setting up cloud projects and accounts* y 4.1 *Managing IAM*  
**Práctica obligatoria del calendario:** crear una matriz de usuarios, grupos y permisos mínimos para tres equipos  
**Proyecto transversal:** SiteOps Tracker, proyecto ficticio de portafolio  
**Duración sugerida:** 90-120 minutos  
**Nivel:** desde cero  
**Idioma:** explicación en español; términos de Google Cloud y preguntas tipo examen en inglés

> Esta lección usa únicamente identidades, dominios, proyectos y recursos ficticios. No copies correos, IDs ni datos reales en una práctica pública.

---

## 1. Objetivo de aprendizaje

Al terminar la práctica podrás:

1. Explicar con tus palabras la diferencia entre **autenticación** y **autorización**.
2. Distinguir **Cloud Identity** de **Identity and Access Management (IAM)**.
3. Leer la relación fundamental de IAM: **principal + role + resource scope**.
4. Diferenciar **permission**, **role**, **role binding** y **allow policy**.
5. Reconocer usuarios, grupos y otros tipos de principal en una política.
6. Explicar por qué Google recomienda conceder roles a grupos en vez de repetir concesiones individuales.
7. Comparar roles básicos, predefinidos y personalizados.
8. Calcular acceso efectivo básico considerando la herencia de organización, carpeta y proyecto.
9. Diseñar una matriz de acceso mínimo para tres equipos de SiteOps Tracker.
10. Inspeccionar una política IAM con Google Cloud Console y `gcloud` sin modificar recursos.
11. Realizar, solo si cuentas con autorización, una concesión temporal de laboratorio y retirarla.
12. Resolver preguntas originales en inglés basadas en escenarios ACE.

### Evidencia que debes producir

Completa dentro de este archivo:

- una matriz de tres equipos;
- la justificación del principal, rol y alcance de cada concesión;
- la salida o descripción de al menos tres consultas de solo lectura;
- el resultado de la verificación posterior a la limpieza;
- las respuestas a diez preguntas en inglés;
- una ficha de errores y una autoevaluación.

La recepción de este archivo no demuestra dominio. El dominio se comprueba cuando puedes justificar las decisiones y repetir la práctica sin depender de la solución.

---

## 2. Prerrequisitos explicados desde cero

### 2.1 Cuenta, identidad y sesión

Para usar Google Cloud necesitas una identidad que pueda autenticarse. Puede ser una cuenta de usuario administrada por Cloud Identity o Google Workspace, una cuenta de Google individual u otro tipo de principal admitido.

- **Autenticación:** demostrar quién eres.
- **Autorización:** determinar qué acciones puedes realizar después de autenticarte.

Iniciar sesión correctamente no significa que tengas permiso para modificar un proyecto.

### 2.2 Jerarquía de recursos

Recuerda el orden general:

```text
Organization
└── Folder
    └── Project
        └── Resource
```

Una concesión IAM colocada en un nivel superior puede heredarse en sus descendientes. Por eso el alcance es tan importante como el rol.

### 2.3 Identificador de proyecto

En los comandos usarás `PROJECT_ID`, no el nombre visible del proyecto. Un proyecto también tiene un número, pero no debes intercambiar ambos valores a ciegas.

### 2.4 Acceso recomendado para la práctica

La ruta principal es de solo lectura. Idealmente necesitas permiso para:

- ver el proyecto;
- consultar su allow policy;
- describir roles predefinidos.

La ruta opcional de escritura requiere autorización para cambiar la política IAM de un proyecto de laboratorio. No la ejecutes en producción, en un proyecto ajeno ni en un laboratorio administrado por terceros que prohíba esos cambios.

### 2.5 Si no tienes organización, dominio o crédito

No necesitas crear una organización ni pagar recursos para completar el objetivo. La sección **Alternativa conceptual completa** permite diseñar la matriz, interpretar una política y calcular acceso efectivo sin cuenta, dominio, facturación o crédito.

---

## 3. Analogía: edificio, directorio y llaves

Imagina que SiteOps Tracker funciona dentro de un edificio:

- **Cloud Identity** es el directorio de personal. Registra a las personas y los grupos a los que pertenecen.
- **IAM** es el sistema que decide qué puertas puede abrir cada identidad.
- Un **principal** es la persona o grupo que intenta entrar.
- Un **permission** es una acción concreta, por ejemplo, leer una métrica.
- Un **role** es un llavero que agrupa permisos.
- Un **role binding** entrega un llavero a uno o más principales.
- El **resource scope** indica dónde funciona ese llavero: una sala, un piso o todo el edificio.
- Una **allow policy** es la lista de asignaciones pegada al recurso.

Esta analogía evita tres errores comunes:

1. Registrar a una persona en el directorio no le abre automáticamente ninguna puerta.
2. Tener un llavero no implica que funcione en todo el edificio.
3. Una regla de construcción, como "ninguna sala puede tener una salida pública", no es una llave: se parece a **Organization Policy**, no a IAM.

---

## 4. Modelo mental de IAM: quién puede hacer qué y dónde

La pregunta central de IAM puede escribirse así:

```text
WHO          CAN DO WHAT                 ON WHICH RESOURCE
principal + role (permissions) + resource scope
```

Ejemplo ficticio:

```text
group:siteops-auditors@example.org
        + roles/logging.viewer
        + project siteops-prod
```

Lectura: el grupo ficticio `siteops-auditors@example.org` recibe las acciones incluidas en `roles/logging.viewer` dentro del proyecto ficticio `siteops-prod`.

### 4.1 Permission

Un **permission** representa una acción concreta sobre un tipo de recurso. Suele tener una forma parecida a:

```text
servicio.recurso.accion
```

No se conceden permisos sueltos directamente a un principal. Se conceden roles que contienen permisos.

### 4.2 Role

Un **role** es una colección de permisos. Ejemplos verificados en la documentación oficial:

- `roles/monitoring.viewer`: lectura de Monitoring.
- `roles/logging.viewer`: lectura de los registros accesibles para ese rol.
- `roles/run.developer`: lectura y escritura de recursos de Cloud Run.
- `roles/resourcemanager.projectIamAdmin`: administración de allow policies en proyectos.

Un rol no es lo mismo que un puesto laboral. "Desarrollador" en el organigrama de una empresa no significa que deba recibir automáticamente `roles/editor`.

### 4.3 Principal

Un **principal** es una identidad a la que se puede conceder acceso. Puede ser un usuario, un grupo, una cuenta de servicio u otro tipo admitido.

### 4.4 Role binding

Un **role binding** asocia un rol con uno o más principales. En forma conceptual:

```yaml
role: roles/monitoring.viewer
members:
  - group:siteops-auditors@example.org
```

La documentación actual usa **principal**, pero el campo de política y la opción de CLI siguen mostrando con frecuencia la palabra `members` o `--member`. En el examen, interpreta ambas en contexto.

### 4.5 Allow policy

Una **allow policy**, antes denominada con frecuencia *IAM policy*, contiene una lista de bindings y metadatos. Se adjunta a un recurso. Cada recurso admite como máximo una allow policy propia, pero su acceso efectivo también puede incluir políticas heredadas.

### 4.6 Effective access

El acceso efectivo no siempre coincide con lo que ves en la política directa del proyecto. Debes considerar:

- bindings del recurso;
- bindings heredados desde proyectos, carpetas u organización según el tipo de recurso;
- condiciones IAM, si existen;
- deny policies y otros controles avanzados;
- pertenencia actual del usuario a grupos.

Para esta lección, concentra el cálculo en allow policies e herencia. Los controles avanzados se estudian con mayor profundidad más adelante.

---

## 5. Cloud Identity frente a IAM

### 5.1 Cloud Identity

Cloud Identity es un servicio de **Identity as a Service (IDaaS)** para administrar centralmente usuarios y grupos. Responde preguntas como:

- ¿Existe la cuenta de esta persona?
- ¿Está activa o suspendida?
- ¿A qué grupos pertenece?
- ¿Quién administra el grupo?

La administración suele realizarse desde Google Admin console y requiere privilegios del dominio.

### 5.2 IAM

IAM controla el acceso a recursos de Google Cloud. Responde preguntas como:

- ¿Qué principal recibió un rol?
- ¿Sobre qué recurso se concedió?
- ¿Qué permisos contiene el rol?
- ¿La concesión es directa o heredada?

### 5.3 Cómo colaboran

Flujo recomendado:

1. Cloud Identity crea o sincroniza al usuario.
2. Cloud Identity coloca al usuario en un grupo por función laboral.
3. IAM concede roles al grupo sobre recursos de Google Cloud.
4. Al cambiar de puesto, se modifica la pertenencia al grupo.
5. Al salir de la organización, se suspende o elimina su cuenta según la política de identidad.

IAM no crea la cuenta humana. Cloud Identity no sustituye las políticas de acceso de Google Cloud.

### 5.4 Ejemplo SiteOps Tracker

Una analista ficticia se incorpora al equipo de auditoría:

1. Se crea `analyst-a@example.org` en Cloud Identity.
2. Se agrega al grupo `siteops-auditors@example.org`.
3. El grupo ya tiene roles de lectura en los proyectos autorizados.
4. La analista obtiene ese acceso mediante la pertenencia al grupo.

No fue necesario añadir su cuenta individual a cada proyecto.

---

## 6. Tipos de principal y sintaxis frecuente

| Tipo | Forma frecuente en un binding | Uso | Precaución |
|---|---|---|---|
| Usuario | `user:persona@example.org` | Acceso humano individual | Difícil de mantener si se repite en muchos recursos. |
| Grupo | `group:equipo@example.org` | Acceso por función | Preferido para equipos; controla bien propietarios y membresía. |
| Service account | `serviceAccount:nombre@proyecto.iam.gserviceaccount.com` | Identidad de workload | No es una cuenta humana; aplica mínimo privilegio. |
| Dominio | `domain:example.org` | Todos los usuarios de un dominio | Alcance muy amplio; rara vez es mínimo privilegio. |
| `allAuthenticatedUsers` | literal, sin prefijo | Cualquier identidad autenticada con cuenta Google | No significa "solo mi organización". |
| `allUsers` | literal, sin prefijo | Cualquier persona en internet | Puede hacer público un recurso; úsalo solo por requisito explícito. |

### Regla ACE

Si muchas personas realizan la misma función y la membresía cambia con el tiempo, normalmente conviene:

```text
usuarios -> grupo funcional -> rol mínimo -> alcance mínimo
```

Evita resolverlo con:

```text
cada usuario -> muchos bindings individuales -> muchos proyectos
```

---

## 7. Usuarios y grupos en Cloud Identity

### 7.1 Usuario

Un usuario representa a una persona administrada. La cuenta permite autenticarse, pero el acceso a Google Cloud depende de las concesiones IAM.

### 7.2 Grupo

Un grupo reúne identidades que comparten una función. Google recomienda conceder roles a grupos cuando sea posible porque la membresía puede cambiar sin editar cada allow policy.

### 7.3 Grupo funcional, no grupo por persona

Buen patrón:

```text
siteops-auditors@example.org
siteops-app-deployers-dev@example.org
siteops-project-access-admins@example.org
```

Patrón deficiente:

```text
grupo-de-ana@example.org
grupo-temporal-2@example.org
usuarios-varios@example.org
```

El nombre debe revelar la función y, cuando sea útil, el entorno o alcance.

### 7.4 Ciclo joiner-mover-leaver

| Evento | Acción de identidad | Resultado esperado en IAM |
|---|---|---|
| Joiner: alta | Crear usuario y agregarlo a grupos aprobados | Hereda acceso del grupo. |
| Mover: cambio de función | Quitar grupos anteriores y agregar los nuevos | Cambia el acceso sin reescribir todos los proyectos. |
| Leaver: salida | Suspender o retirar la identidad según el proceso | Deja de usar las concesiones del grupo. |

### 7.5 Propietarios de grupo

Cada grupo de acceso debe tener responsables, revisión periódica y reglas sobre miembros externos. Un grupo sin propietario operativo puede acumular acceso innecesario.

### 7.6 Administración manual frente a automatizada

La guía ACE menciona administración manual y automatizada de usuarios y grupos:

- **Manual:** un administrador crea la cuenta o cambia la membresía desde Google Admin console. Es comprensible para una práctica pequeña, pero no escala bien.
- **Automatizada:** una organización sincroniza identidades desde su fuente autorizada o usa APIs y procesos de aprovisionamiento. Reduce trabajo repetitivo, pero exige gobernanza, permisos y manejo de errores.

Para esta lección no configures sincronización. Solo recuerda la decisión: la fuente de identidad administra el ciclo de vida; IAM consume usuarios y grupos como principales.

---

## 8. Tipos de roles

### 8.1 Basic roles

Los roles básicos son:

- `roles/viewer`
- `roles/editor`
- `roles/owner`

Son amplios y pueden incluir permisos de muchos servicios. La documentación de seguridad recomienda no usarlos en producción cuando exista una alternativa más limitada.

### 8.2 Predefined roles

Los roles predefinidos son administrados por Google y suelen estar orientados a un servicio o función. Ejemplos:

- `roles/monitoring.viewer`
- `roles/logging.viewer`
- `roles/run.developer`
- `roles/artifactregistry.reader`

Son la primera opción habitual porque equilibran precisión y mantenimiento.

### 8.3 Custom roles

Un rol personalizado contiene una lista elegida de permisos. Úsalo cuando ningún rol predefinido satisfaga el requisito de mínimo privilegio.

No crees un custom role solo porque su nombre parece más elegante. Debes mantenerlo, revisar permisos compatibles y actualizarlo cuando cambie el requisito.

### 8.4 Orden de decisión

```text
1. Define la tarea exacta.
2. Busca el rol predefinido más limitado que la permita.
3. Ajusta el alcance al recurso más pequeño viable.
4. Usa un custom role solo si queda una brecha real.
5. Evita basic roles en producción cuando haya alternativa.
```

---

## 9. Alcance, herencia y acceso efectivo

### 9.1 Una concesión se hereda hacia abajo

Si un grupo recibe un rol en una carpeta, los proyectos descendientes pueden heredar la concesión. Si se concede en la organización, el posible alcance es todavía mayor.

```text
Organization: example.org
└── Folder: SiteOps
    ├── Project: siteops-dev
    └── Project: siteops-prod
```

Una concesión en `Folder: SiteOps` puede llegar a `siteops-dev` y `siteops-prod`.

### 9.2 Las concesiones se acumulan

Para allow policies, piensa en una unión de permisos permitidos:

```text
permisos efectivos = concesiones directas + concesiones heredadas aplicables
```

Ejemplo:

- El grupo obtiene `roles/monitoring.viewer` en la carpeta.
- El mismo grupo obtiene `roles/logging.viewer` en el proyecto de producción.
- En producción puede tener ambos conjuntos de permisos.

### 9.3 Un binding hijo no cancela un binding padre

No puedes retirar en el proyecto una concesión heredada simplemente omitiéndola de la política del proyecto. Debes cambiar la concesión en su origen o emplear controles diseñados para limitar acceso, según el caso.

### 9.4 Alcance mínimo

Pregunta siempre:

> ¿Esta función necesita el rol en toda la organización, en una carpeta, en un proyecto o en un recurso concreto?

Conceder un rol limitado en toda la organización puede ser más peligroso que conceder un rol algo más amplio sobre un único recurso. El rol y el alcance se evalúan juntos.

### 9.5 Política directa frente a acceso efectivo

`gcloud projects get-iam-policy` devuelve la política adjunta al proyecto. No demuestra por sí sola todas las concesiones heredadas de sus ancestros. Para un diagnóstico completo, revisa la jerarquía y las políticas de los niveles superiores con los permisos adecuados.

---

## 10. Principio de mínimo privilegio

**Least privilege** significa conceder solo los permisos necesarios, sobre los recursos necesarios y durante el tiempo necesario.

Evalúa cuatro dimensiones:

| Dimensión | Pregunta |
|---|---|
| Principal | ¿Debe recibirlo una persona, un grupo o un workload? |
| Rol | ¿Cuál es el rol más limitado que permite la tarea? |
| Alcance | ¿Cuál es el recurso más pequeño viable? |
| Tiempo | ¿Debe ser permanente o temporal? |

### Señales de exceso de privilegio

- se concede `Owner` para resolver un error de lectura;
- se usa `Editor` porque no se conoce el rol del servicio;
- se asigna un rol en organización para trabajar en un proyecto;
- cada usuario conserva acceso después de cambiar de equipo;
- se concede `allUsers` cuando solo debía acceder un grupo;
- un deployer puede actuar como una service account con privilegios excesivos.

### Frase de examen

> Grant the narrowest predefined role at the lowest practical resource level, preferably to a group for human users.

---

## 11. Diseño de identidad para SiteOps Tracker

### 11.1 Contexto ficticio

SiteOps Tracker registra sedes, activos, hallazgos, estados, responsables e historial. Su stack de portafolio incluye:

- React + TypeScript;
- Node.js + TypeScript en Cloud Run;
- PostgreSQL en Cloud SQL;
- imágenes de contenedor en Artifact Registry;
- métricas y registros en Cloud Monitoring y Cloud Logging.

Los identificadores siguientes son exclusivamente didácticos:

```text
Organization: example.org
Folder: SiteOps
Projects: siteops-dev, siteops-prod
Repository: siteops-containers
Runtime service account: siteops-runtime@siteops-dev.iam.gserviceaccount.com
```

### 11.2 Requisitos de tres equipos

#### Equipo A - auditoría técnica

Necesita:

- leer registros estándar disponibles;
- consultar métricas;
- documentar hallazgos operativos;
- no modificar aplicaciones, identidades, políticas ni recursos.

#### Equipo B - despliegue de aplicación en desarrollo

Necesita:

- administrar recursos de Cloud Run en `siteops-dev`;
- leer la imagen que se desplegará desde un repositorio autorizado;
- actuar como una service account de runtime concreta;
- no administrar IAM del proyecto ni producción.

#### Equipo C - administración de acceso del proyecto

Necesita:

- administrar allow policies de los proyectos aprobados;
- no recibir automáticamente permisos para editar workloads o datos;
- trabajar mediante un grupo pequeño y revisado.

### 11.3 Matriz propuesta de mínimo privilegio

| Equipo | Usuarios ficticios | Grupo ficticio | Rol | Alcance | Motivo |
|---|---|---|---|---|---|
| Auditoría técnica | `auditor-a@example.org`, `auditor-b@example.org` | `siteops-auditors@example.org` | `roles/logging.viewer` | Proyectos autorizados | Leer registros estándar accesibles para el rol. |
| Auditoría técnica | mismos | mismo | `roles/monitoring.viewer` | Proyectos autorizados | Consultar métricas y Monitoring en modo lectura. |
| Despliegue dev | `developer-a@example.org`, `developer-b@example.org` | `siteops-app-deployers-dev@example.org` | `roles/run.developer` | Proyecto `siteops-dev` o recursos Cloud Run admitidos | Crear y actualizar recursos de Cloud Run, sin usar `Editor`. |
| Despliegue dev | mismos | mismo | `roles/artifactregistry.reader` | Repositorio `siteops-containers` | Descargar la imagen a desplegar; no requiere borrar artefactos. |
| Despliegue dev | mismos | mismo | `roles/iam.serviceAccountUser` | Solo la runtime service account aprobada | Permitir `actAs` durante el despliegue; no sobre todas las cuentas de servicio. |
| Administración de acceso | `access-admin-a@example.org`, `access-admin-b@example.org` | `siteops-project-access-admins@example.org` | `roles/resourcemanager.projectIamAdmin` | Solo proyectos autorizados | Administrar allow policies del proyecto sin recibir `Owner`. |

### 11.4 Matices importantes

1. Si el requisito se amplía a listar recursos y revisar sus allow policies, evalúa `roles/iam.securityReviewer` por separado. Es amplio y se superpone con diversos permisos de lectura; revisa su contenido antes de agregarlo a otros roles.
2. Si los desarrolladores también crean o suben imágenes, `roles/artifactregistry.reader` no basta; evalúa `roles/artifactregistry.writer` sobre el repositorio concreto. No lo concedas si solo despliegan una imagen existente.
3. `roles/iam.serviceAccountUser` permite actuar como la service account. La service account de runtime también debe tener privilegios mínimos, porque un deployer podría ejecutar código con su identidad.
4. `roles/resourcemanager.projectIamAdmin` permite cambiar allow policies del proyecto. Es un rol sensible y debe revisarse; no concede por sí mismo administración funcional de todos los servicios.
5. Para producción puede ser más seguro que los humanos no desplieguen directamente y que lo haga una identidad de CI/CD controlada. Esa arquitectura se estudiará en lecciones posteriores.

### 11.5 Tu matriz obligatoria

Completa esta versión antes de consultar la propuesta anterior de nuevo:

| Equipo | Tarea exacta | Usuarios ficticios | Grupo | Rol mínimo | Alcance mínimo | ¿Por qué no `Owner`/`Editor`? | Revisión |
|---|---|---|---|---|---|---|---|
| Auditoría |  |  |  |  |  |  |  |
| Despliegue dev |  |  |  |  |  |  |  |
| Administración de acceso |  |  |  |  |  |  |  |

---

## 12. Qué servicio o mecanismo elegir

| Necesidad | Elección | Por qué | Por qué no la alternativa |
|---|---|---|---|
| Crear y suspender cuentas humanas | Cloud Identity o Google Workspace | Administra el ciclo de vida de identidades. | IAM no crea cuentas de usuario. |
| Dar el mismo acceso a varias personas | Grupo + IAM | Centraliza membresía y concesiones. | Bindings por usuario multiplican cambios y auditoría. |
| Autorizar acciones sobre recursos | IAM allow policy | Vincula principal, rol y recurso. | Cloud Identity solo no concede acceso a recursos. |
| Restringir una configuración para todos | Organization Policy | Controla qué configuraciones se permiten. | IAM decide quién puede actuar, no sustituye una restricción organizacional. |
| Permitir solo lectura de métricas | `roles/monitoring.viewer` | Rol predefinido de lectura del servicio. | `roles/viewer` es mucho más amplio; `roles/editor` permite cambios. |
| Administrar allow policies de un proyecto | `roles/resourcemanager.projectIamAdmin` | Está diseñado para ese alcance. | `roles/owner` mezcla administración IAM con acceso amplio. |
| Cubrir una brecha que ningún rol predefinido resuelve | Custom role revisado | Puede ajustarse a permisos concretos. | Crear un custom role sin brecha aumenta mantenimiento. |
| Dar acceso a toda persona en internet | `allUsers`, solo por requisito explícito | Representa acceso público. | Un grupo o usuario no hace público el recurso; `allAuthenticatedUsers` también es más amplio que una organización. |

---

## 13. Práctica guiada segura - Google Cloud Console

### 13.1 Reglas antes de comenzar

1. Usa un proyecto de laboratorio autorizado.
2. Mantén la primera parte en modo de solo lectura.
3. No agregues `Owner`, `Editor`, `allUsers` ni `allAuthenticatedUsers`.
4. No modifiques una organización o carpeta real.
5. Si el laboratorio pertenece a Google Skills, sigue sus instrucciones y no crees identidades fuera del alcance permitido.
6. Registra el estado anterior y posterior a cualquier cambio opcional.

### 13.2 Confirma el proyecto

1. Abre Google Cloud Console.
2. Usa el selector superior de proyecto.
3. Anota:
   - nombre visible;
   - `Project ID`;
   - organización o carpeta, si tienes permiso para verla.
4. Confirma que no estás en producción.

### 13.3 Inspecciona IAM

1. Ve a **IAM & Admin > IAM**.
2. Observa las columnas de principal, roles y herencia disponibles en tu vista.
3. Localiza al menos:
   - un usuario;
   - un grupo, si existe;
   - una service account.
4. No confundas una service account con una persona.
5. Anota si cada concesión observada parece directa o heredada.

### 13.4 Inspecciona un rol

1. Ve a **IAM & Admin > Roles**.
2. Busca **Monitoring Viewer**.
3. Confirma el ID `roles/monitoring.viewer`.
4. Revisa descripción, etapa y permisos incluidos.
5. Compara con el rol básico **Viewer** y explica cuál tiene menor superficie para ver únicamente Monitoring.

### 13.5 Observa Cloud Identity, solo si eres administrador autorizado

1. Abre Google Admin console con una cuenta administrativa.
2. Revisa **Directory > Users** y **Directory > Groups**.
3. Identifica la diferencia entre:
   - crear una cuenta;
   - agregarla a un grupo;
   - conceder al grupo un rol IAM.
4. No crees usuarios ni grupos reales para esta lección.

Si no tienes dominio verificado o privilegios de administrador, usa la alternativa conceptual. Esta limitación no impide completar el aprendizaje.

### 13.6 Cambio opcional y controlado

Solo si tienes un grupo de práctica real, autorización y un proyecto temporal:

1. En **IAM & Admin > IAM**, selecciona **Grant access**.
2. Introduce el grupo de laboratorio.
3. Selecciona **Monitoring Viewer**.
4. Guarda.
5. Comprueba que el binding aparece.
6. Retira esa concesión al terminar.

Registra:

```text
Proyecto temporal:
Grupo de laboratorio:
Rol:
Hora de concesión:
Hora de retirada:
Verificación final:
```

---

## 14. Práctica guiada segura - Cloud Shell y gcloud

Los comandos siguientes se contrastaron con la documentación oficial vigente el 30 de septiembre de 2026.

### 14.1 Confirma la identidad activa

```bash
gcloud auth list --filter='status:ACTIVE' --format='value(account)'
```

Resultado esperado: una cuenta activa. Si no reconoces la cuenta, detente.

### 14.2 Confirma el proyecto configurado

```bash
gcloud config get-value project
```

Guarda el ID únicamente cuando confirmes que es el proyecto de laboratorio:

```bash
export PROJECT_ID="REEMPLAZA_CON_PROJECT_ID_AUTORIZADO"
```

Verifica el proyecto:

```bash
gcloud projects describe "$PROJECT_ID" \
  --format='yaml(projectId,projectNumber,name,parent)'
```

### 14.3 Obtén la allow policy directa del proyecto

```bash
gcloud projects get-iam-policy "$PROJECT_ID" --format=json
```

Busca:

- `bindings`;
- `role`;
- `members`;
- `etag`;
- `version`.

Este comando es de lectura. La política directa del proyecto puede no mostrar todas las concesiones heredadas desde ancestros.

### 14.4 Muestra bindings en una tabla

```bash
gcloud projects get-iam-policy "$PROJECT_ID" \
  --flatten='bindings' \
  --format='table(bindings.role,bindings.members)'
```

### 14.5 Filtra grupos

```bash
gcloud projects get-iam-policy "$PROJECT_ID" \
  --flatten='bindings' \
  --filter='bindings.members:group:' \
  --format='table(bindings.role,bindings.members)'
```

Si no devuelve filas, no concluyas que la organización no usa grupos. Puede no haber bindings de grupo directos en ese proyecto o puede existir acceso heredado.

### 14.6 Describe roles predefinidos

```bash
gcloud iam roles describe roles/monitoring.viewer \
  --format='yaml(name,title,description,stage,includedPermissions)'
```

```bash
gcloud iam roles describe roles/resourcemanager.projectIamAdmin \
  --format='yaml(name,title,description,stage,includedPermissions)'
```

Compara los permisos. El nombre del rol no sustituye la revisión de su contenido.

### 14.7 Simula la matriz sin escribir

Crea en tus notas estas variables ficticias, sin ejecutar una concesión:

```bash
AUDIT_GROUP='group:siteops-auditors@example.org'
DEPLOY_GROUP='group:siteops-app-deployers-dev@example.org'
ACCESS_GROUP='group:siteops-project-access-admins@example.org'
```

Relaciona cada valor con los roles y alcances de la matriz. Estas direcciones son ejemplos y no tienen que existir.

### 14.8 Concesión opcional de laboratorio

Ejecuta esta sección solo si `LAB_GROUP` es un grupo real de práctica, tienes autorización y registraste el estado previo:

```bash
export LAB_GROUP='REEMPLAZA_CON_GRUPO_AUTORIZADO'
```

```bash
gcloud projects add-iam-policy-binding "$PROJECT_ID" \
  --member="group:${LAB_GROUP}" \
  --role='roles/monitoring.viewer'
```

La sintaxis oficial de `gcloud projects add-iam-policy-binding` requiere proyecto, `--member` y `--role`; una condición es opcional.

### 14.9 Verifica el cambio opcional

```bash
gcloud projects get-iam-policy "$PROJECT_ID" \
  --flatten='bindings' \
  --filter="bindings.members:group:${LAB_GROUP}" \
  --format='table(bindings.role,bindings.members)'
```

### 14.10 Retira la concesión opcional

```bash
gcloud projects remove-iam-policy-binding "$PROJECT_ID" \
  --member="group:${LAB_GROUP}" \
  --role='roles/monitoring.viewer'
```

Verifica de nuevo:

```bash
gcloud projects get-iam-policy "$PROJECT_ID" \
  --flatten='bindings' \
  --filter="bindings.members:group:${LAB_GROUP}" \
  --format='table(bindings.role,bindings.members)'
```

Resultado esperado: no debe quedar ese binding directo. Si el grupo conserva acceso, investiga concesiones heredadas o bindings adicionales antes de concluir que la retirada falló.

### 14.11 Limpia variables locales

```bash
unset PROJECT_ID LAB_GROUP AUDIT_GROUP DEPLOY_GROUP ACCESS_GROUP
```

`unset` limpia variables de la sesión; no cambia recursos de Google Cloud.

---

## 15. Alternativa conceptual completa sin cuenta o permisos

### 15.1 Escenario

Considera esta jerarquía ficticia:

```text
Organization: example.org
└── Folder: SiteOps
    ├── Project: siteops-dev
    └── Project: siteops-prod
```

Pertenencias:

```text
user:analyst-a@example.org
  -> group:siteops-auditors@example.org

user:developer-a@example.org
  -> group:siteops-app-deployers-dev@example.org

user:access-admin-a@example.org
  -> group:siteops-project-access-admins@example.org
```

Bindings:

```yaml
folder_siteops:
  - role: roles/monitoring.viewer
    members:
      - group:siteops-auditors@example.org

project_siteops_prod:
  - role: roles/logging.viewer
    members:
      - group:siteops-auditors@example.org

project_siteops_dev:
  - role: roles/run.developer
    members:
      - group:siteops-app-deployers-dev@example.org
  - role: roles/resourcemanager.projectIamAdmin
    members:
      - group:siteops-project-access-admins@example.org
```

### 15.2 Calcula el acceso efectivo

Responde sin mirar la solución:

1. ¿Puede la analista consultar Monitoring en desarrollo?
2. ¿Puede consultar Monitoring en producción?
3. ¿Puede consultar Logging en producción?
4. ¿Puede modificar Cloud Run en producción?
5. ¿Puede la desarrolladora modificar Cloud Run en desarrollo?
6. ¿Puede la administradora de acceso administrar la allow policy de desarrollo?
7. ¿El último rol le permite automáticamente editar la aplicación?

### 15.3 Solución

1. Sí. `roles/monitoring.viewer` se concede al grupo en la carpeta y se hereda.
2. Sí, por la misma razón.
3. Sí. Existe un binding directo en `siteops-prod`.
4. No según los datos dados. No tiene `roles/run.developer` ni otro rol equivalente.
5. Sí. Su grupo tiene `roles/run.developer` directamente en `siteops-dev`.
6. Sí. Su grupo tiene `roles/resourcemanager.projectIamAdmin` en el proyecto.
7. No. Administrar la allow policy no equivale a administrar los recursos de Cloud Run.

### 15.4 Cambio de grupo

Si `analyst-a@example.org` sale de `siteops-auditors@example.org`, pierde el acceso recibido solo por ese grupo cuando el cambio se propague. Si además tenía un binding directo como usuario, ese binding permanece. Esta distinción aparece con frecuencia en preguntas de examen.

---

## 16. Hoja de decisión de la práctica

Completa una fila por cada concesión propuesta:

| Requisito | Principal elegido | Rol | Alcance | Herencia esperada | Riesgo | Alternativa descartada | Decisión final |
|---|---|---|---|---|---|---|---|
| Ver métricas |  |  |  |  |  |  |  |
| Leer registros |  |  |  |  |  |  |  |
| Desplegar Cloud Run en dev |  |  |  |  |  |  |  |
| Descargar imagen |  |  |  |  |  |  |  |
| Actuar como runtime SA |  |  |  |  |  |  |  |
| Administrar IAM de proyecto |  |  |  |  |  |  |  |

### Explicaciones obligatorias

Escribe en español:

1. Por qué un grupo reduce la carga administrativa frente a bindings individuales.
2. Por qué `roles/viewer` no es la mejor elección para una persona que solo necesita Monitoring.
3. Por qué `roles/resourcemanager.projectIamAdmin` no debe concederse a toda la organización.
4. Qué acceso se hereda desde la carpeta SiteOps.
5. Qué cambiarías para impedir despliegues humanos directos en producción.

### Respuesta corta en inglés

Completa y lee en voz alta:

> For SiteOps Tracker, I grant roles to functional groups instead of individual users. I choose the narrowest predefined role and the lowest practical resource scope. Cloud Identity manages users and group membership, while IAM authorizes actions on Google Cloud resources.

---

## 17. Resultado esperado

Al finalizar debes tener:

- una matriz con tres equipos, usuarios ficticios, grupos, roles y alcances;
- una justificación de mínimo privilegio por binding;
- distinción clara entre Cloud Identity e IAM;
- la salida o descripción de `get-iam-policy`;
- la descripción de al menos dos roles con `gcloud iam roles describe`;
- evidencia de que no dejaste una concesión temporal;
- siete respuestas correctas del cálculo de acceso efectivo;
- respuestas a las diez preguntas de examen;
- ficha de errores completada.

Ejemplo de conclusión válida:

> Cloud Identity controla las cuentas y la pertenencia a grupos. IAM concede roles a esos principales sobre un alcance. Para SiteOps Tracker usaría grupos funcionales, roles predefinidos y bindings en proyectos o recursos concretos. Revisaría la herencia antes de concluir que una retirada en el proyecto eliminó todo el acceso.

Conclusión insuficiente:

> IAM da permisos y ya.

---

## 18. Solución de problemas

### Caso 1 - `PERMISSION_DENIED` al consultar la política

**Causa probable:** tu cuenta no tiene permiso para obtener la allow policy o estás en el proyecto equivocado.

**Diagnóstico:**

```bash
gcloud auth list --filter='status:ACTIVE' --format='value(account)'
gcloud config get-value project
```

Solicita acceso de lectura adecuado; no te autoasignes `Owner`.

### Caso 2 - el principal tiene un rol, pero la acción sigue fallando

Comprueba:

- recurso y proyecto correctos;
- rol concedido y permisos incluidos;
- alcance del binding;
- condición IAM;
- deny policy u otra restricción;
- propagación de membresía del grupo;
- API necesaria habilitada.

No asumas que el título del rol contiene exactamente el permiso requerido.

### Caso 3 - quitaste un binding del proyecto y aún hay acceso

Puede existir:

- un binding heredado de carpeta u organización;
- otro rol que contiene el mismo permiso;
- una concesión directa al usuario además de la del grupo;
- una condición diferente con otro binding.

Revisa el acceso efectivo y los ancestros.

### Caso 4 - el grupo no aparece en la salida filtrada

La política directa puede no contener grupos. También puede haber acceso heredado. Ejecuta primero la tabla sin filtro y revisa el nivel correcto.

### Caso 5 - no puedes abrir Google Admin console

No eres administrador del dominio o no existe un tenant de Cloud Identity/Workspace disponible. Completa la alternativa conceptual; no necesitas crear un dominio para la lección.

### Caso 6 - `INVALID_ARGUMENT` con `--member`

Verifica el prefijo:

```text
user:
group:
serviceAccount:
domain:
```

`allUsers` y `allAuthenticatedUsers` no llevan esos prefijos.

### Caso 7 - elegiste `roles/editor` porque el rol predefinido no funcionó

No escales directamente. Identifica el permiso faltante, revisa documentación del servicio y elige uno o varios roles predefinidos limitados. Considera un custom role solo si existe una brecha demostrada.

### Caso 8 - el deployer de Cloud Run recibe error relacionado con la service account

Además de permisos sobre Cloud Run y el repositorio, un despliegue con una service identity requiere que el deployer pueda actuar como esa service account, normalmente mediante `roles/iam.serviceAccountUser` sobre la cuenta concreta.

### Caso 9 - una persona salió del grupo pero conserva acceso

Busca bindings directos a `user:correo`, pertenencia a otros grupos y concesiones heredadas. Quitar un camino de acceso no elimina los demás.

### Caso 10 - la política cambió mientras la editabas

Las políticas incluyen `etag` para control de concurrencia. Evita reemplazar una política completa basada en una copia antigua. Para un único binding, usa los comandos específicos y vuelve a leer el estado.

---

## 19. Impacto en costos y limpieza

### 19.1 Costos

- Consultar IAM y describir roles no crea VMs, bases de datos ni almacenamiento.
- Los bindings IAM por sí mismos no ejecutan workloads.
- Un rol puede permitir crear recursos facturables; el costo proviene de esos recursos, no del texto del binding.
- Cloud Identity tiene ediciones y requisitos distintos. No contrates ni habilites una edición de pago solo para completar esta lección.
- Google Admin, la creación de un dominio y licencias pueden depender de la configuración de la organización.

### 19.2 Riesgo operativo

El principal impacto de una práctica IAM incorrecta es de seguridad:

- acceso excesivo;
- modificación o eliminación de recursos;
- exposición pública;
- escalamiento mediante service accounts;
- pérdida de acceso administrativo.

### 19.3 Lista de limpieza

- [ ] Retiré el binding temporal de `roles/monitoring.viewer`.
- [ ] Confirmé que el grupo de laboratorio ya no aparece en la política directa.
- [ ] No agregué `allUsers` ni `allAuthenticatedUsers`.
- [ ] No concedí `Owner` ni `Editor`.
- [ ] No creé usuarios, grupos o dominios innecesarios.
- [ ] No guardé correos reales, tokens, credenciales o IDs privados en este archivo.
- [ ] Ejecuté `unset` para limpiar variables locales.
- [ ] Anoté cualquier acceso heredado que siga vigente.

---

## 20. Glosario bilingüe

| English | Español | Definición útil para ACE |
|---|---|---|
| Authentication | Autenticación | Verificación de la identidad. |
| Authorization | Autorización | Decisión sobre acciones permitidas. |
| Principal | Principal / identidad | Entidad que puede recibir acceso. |
| Member | Miembro | Término aún visible en bindings y CLI para una identidad asociada. |
| User | Usuario | Cuenta humana administrada. |
| Group | Grupo | Conjunto de identidades administrado como unidad. |
| Service account | Cuenta de servicio | Identidad de un workload o aplicación. |
| Permission | Permiso | Acción concreta sobre un recurso. |
| Role | Rol | Colección de permisos. |
| Basic role | Rol básico | Rol amplio: Viewer, Editor u Owner. |
| Predefined role | Rol predefinido | Rol administrado por Google para una función o servicio. |
| Custom role | Rol personalizado | Rol definido por el cliente con permisos seleccionados. |
| Role binding | Vinculación de rol | Asociación entre un rol y uno o más principales. |
| Allow policy | Política de permisos | Conjunto de bindings adjunto a un recurso. |
| Resource scope | Alcance del recurso | Nivel o recurso donde aplica la concesión. |
| Policy inheritance | Herencia de políticas | Aplicación de concesiones de ancestros a descendientes. |
| Effective access | Acceso efectivo | Resultado de todas las políticas y controles aplicables. |
| Least privilege | Mínimo privilegio | Solo los permisos, alcance y tiempo necesarios. |
| Cloud Identity | Cloud Identity | Servicio central de usuarios y grupos. |
| Google Admin console | Consola de administración de Google | Interfaz para administrar el dominio y las identidades. |
| Direct grant | Concesión directa | Binding en el recurso inspeccionado. |
| Inherited grant | Concesión heredada | Binding procedente de un ancestro. |
| Public access | Acceso público | Acceso concedido a `allUsers` o configuración equivalente. |
| Policy etag | ETag de política | Valor usado para detectar actualizaciones concurrentes. |

### Frases clave en inglés

- **Grant the role to a group rather than to each user individually.**
- **Use the principle of least privilege.**
- **Choose the lowest practical resource scope.**
- **The project inherits the role binding from its parent folder.**
- **Cloud Identity manages users and groups; IAM manages authorization.**
- **Removing a user from one group does not remove direct grants.**
- **A basic role is broader than a service-specific predefined role.**

---

## 21. Preguntas originales estilo ACE - en inglés

No consultes las soluciones hasta responder las diez preguntas.

### Question 1

A company onboards and offboards SiteOps auditors frequently. All auditors need the same read-only access to several Google Cloud projects. What should the administrator do?

A. Grant the Owner role to each auditor at the organization level.  
B. Create a functional group, grant the required roles to the group, and manage user membership in Cloud Identity.  
C. Share one user account among all auditors.  
D. Create a service account key for each auditor.

### Question 2

A user only needs read-only access to Cloud Monitoring for one project. Which choice best follows least privilege?

A. Grant `roles/owner` on the project.  
B. Grant `roles/editor` on the project.  
C. Grant `roles/monitoring.viewer` on the project.  
D. Grant `roles/viewer` on the organization.

### Question 3

Which statement correctly describes Cloud Identity and IAM?

A. Cloud Identity manages Google Cloud resource permissions, while IAM creates user accounts.  
B. Cloud Identity centrally manages users and groups, while IAM grants roles on Google Cloud resources.  
C. Cloud Identity and IAM are two names for the same allow policy.  
D. IAM must create a group before Google Admin console can use it.

### Question 4

The group `siteops-auditors@example.org` receives `roles/monitoring.viewer` on a folder that contains two projects. No deny policy or condition applies. What is the expected result?

A. The role applies only to the folder object and never to projects.  
B. The role is inherited by resources in both descendant projects where applicable.  
C. Each user must also receive the same role directly.  
D. The role automatically becomes `roles/editor` in child projects.

### Question 5

An administrator must manage allow policies for one project but should not automatically receive broad access to its workloads. Which predefined role is the most appropriate starting point?

A. `roles/owner`  
B. `roles/editor`  
C. `roles/resourcemanager.projectIamAdmin`  
D. `roles/run.admin`

### Question 6

Which command reads the allow policy attached directly to a project?

A. `gcloud projects get-iam-policy PROJECT_ID`  
B. `gcloud projects add-iam-policy-binding PROJECT_ID`  
C. `gcloud iam roles delete PROJECT_ID`  
D. `gcloud organizations set-iam-policy PROJECT_ID`

### Question 7

A developer proposes granting `allAuthenticatedUsers` access because "only employees will be authenticated." What is the key problem?

A. `allAuthenticatedUsers` represents only service accounts.  
B. `allAuthenticatedUsers` can include any authenticated Google account, not just identities in the company domain.  
C. `allAuthenticatedUsers` is identical to a Cloud Identity group.  
D. `allAuthenticatedUsers` can be used only on folders.

### Question 8 - Select three

A human deployer must deploy an existing container image to Cloud Run by using a specific runtime service account. Based on the documented Cloud Run deployment model, which three role grants are typically required?

A. Cloud Run Developer on the Cloud Run resource or appropriate project scope.  
B. Artifact Registry Reader on the repository containing the image.  
C. Service Account User on the runtime service account.  
D. Owner on the organization.  
E. Billing Account Administrator on the billing account.

### Question 9 - Select two

An audit group only needs to read standard project logs and Monitoring data. Which two roles directly match those requirements?

A. `roles/logging.viewer`  
B. `roles/monitoring.viewer`  
C. `roles/editor`  
D. `roles/resourcemanager.projectIamAdmin`

### Question 10

Mina receives `roles/monitoring.viewer` through a group and also has the same role through a direct user binding. She is removed from the group. What should you expect after group membership changes propagate?

A. All Monitoring access is removed because group membership always overrides direct bindings.  
B. Her direct binding can continue to grant Monitoring access.  
C. Her direct binding is automatically deleted by Cloud Identity.  
D. She automatically receives `roles/viewer` instead.

---

## 22. Soluciones justificadas

### Answer 1 - B

**Correct:** B. A functional group centralizes access, while Cloud Identity membership handles joiners, movers and leavers.

- A is incorrect: Owner at organization level is excessive in role and scope.
- C is incorrect: shared accounts destroy individual accountability and are unsafe.
- D is incorrect: service account keys are not the right identity model for human auditors.

### Answer 2 - C

**Correct:** C. `roles/monitoring.viewer` directly matches read-only Monitoring access at the required project scope.

- A is incorrect: Owner is far broader and can administer access.
- B is incorrect: Editor permits many modifications unrelated to the requirement.
- D is incorrect: Viewer at organization scope is both broader in permissions and vastly broader in scope.

### Answer 3 - B

**Correct:** B. Cloud Identity manages identity objects and group membership; IAM authorizes principals on Google Cloud resources.

- A reverses the responsibilities.
- C is incorrect: they are separate systems that integrate.
- D is incorrect: groups are managed through identity administration; IAM consumes the group as a principal.

### Answer 4 - B

**Correct:** B. Descendant projects inherit applicable allow-policy role bindings from their parent folder.

- A ignores policy inheritance.
- C is incorrect: group members receive access through the group; duplicate user bindings are unnecessary.
- D is incorrect: inheritance preserves the role; it does not upgrade it.

### Answer 5 - C

**Correct:** C. Project IAM Admin is designed to administer project allow policies.

- A is incorrect: Owner is unnecessarily broad and can administer much more than the stated task.
- B is incorrect: Editor focuses on broad resource modification and is not the precise IAM-administration choice.
- D is incorrect: Cloud Run Admin manages Cloud Run, not the project's general allow policy.

### Answer 6 - A

**Correct:** A. `get-iam-policy` is the read operation for the policy attached to the project.

- B changes the policy and also lacks required `--member` and `--role` arguments.
- C is unrelated and would target a role, not read the project policy.
- D is a write operation for an organization and uses the wrong resource type.

### Answer 7 - B

**Correct:** B. `allAuthenticatedUsers` is not restricted to a company's Cloud Identity domain.

- A is incorrect: it is not limited to service accounts.
- C is incorrect: it is a special principal, not a managed company group.
- D is incorrect: the core issue is its broad audience, not an alleged folder-only rule.

### Answer 8 - A, B and C

**Correct:**

- A supplies permissions to create or update the Cloud Run resource.
- B permits access to the existing container image in Artifact Registry.
- C permits the deployer to act as the selected runtime service account.

**Distractors:**

- D is incorrect: organization Owner is drastically excessive.
- E is incorrect: billing administration is unrelated to deploying the service.

The exact resource scope can vary, but the three access surfaces remain important: Cloud Run, the image repository and the service identity.

### Answer 9 - A and B

**Correct:**

- A matches read-only access to standard logs available to Logs Viewer.
- B matches read-only Monitoring access.

**Distractors:**

- C is far broader and permits changes across services.
- D permits policy administration, not merely reading logs and metrics.

### Answer 10 - B

**Correct:** B. Allow-policy grants accumulate. Removing one access path does not remove a separate direct binding.

- A is incorrect: group removal does not override an independent allow binding.
- C is incorrect: Cloud Identity changes membership; it does not automatically delete unrelated project bindings.
- D is incorrect: IAM does not substitute another role automatically.

### Registro de puntuación

| Resultado | Acción recomendada |
|---:|---|
| 9-10 | Explica oralmente las diez decisiones y repite la matriz sin apuntes. |
| 8 | Revisa el distractor que elegiste y escribe la regla de decisión. |
| 6-7 | Repite secciones 4-12 y vuelve a resolver al día siguiente. |
| 0-5 | Reconstruye el modelo principal-rol-alcance y realiza la alternativa conceptual completa. |

**Puntuación:** ____ / 10  
**Tiempo:** ____ minutos  
**Preguntas a revisar:** ____

---

## 23. Repaso activo espaciado

Responde sin abrir las lecciones anteriores. Después compara con tus apuntes.

### D-1 hábil - Lección 07: Organization Policy

1. Completa: IAM decide **quién puede hacer qué**; Organization Policy limita __________.
2. ¿Una Organization Policy concede un rol a un grupo?
3. ¿Qué diferencia existe entre una policy configurada y una effective policy?
4. ¿Para qué sirve dry-run mode?
5. Explica por qué una persona puede tener permiso IAM y aun así encontrar una operación bloqueada.

**Respuesta breve:** Organization Policy restringe configuraciones o acciones permitidas; no concede identidades. La política efectiva considera la jerarquía. Dry-run registra posibles infracciones sin aplicar el bloqueo de esa configuración de prueba.

### Días hábiles anteriores - Lección 06: jerarquía

1. Ordena: recurso, organización, proyecto, carpeta.
2. ¿Por qué separar desarrollo y producción en proyectos?
3. ¿Qué propiedad tiene un recurso respecto de sus padres en la jerarquía?

**Respuesta breve:** organización -> carpeta -> proyecto -> recurso. Los proyectos crean fronteras de administración, cuota, facturación y aislamiento. Un recurso tiene un único padre inmediato dentro de la jerarquía.

### Días hábiles anteriores - Lección 05: método ACE

1. En un escenario, ¿qué palabras indican mínimo privilegio?
2. ¿Por qué debes justificar los distractores y no solo memorizar la letra correcta?
3. Escribe una regla de decisión en inglés usando *least privilege*.

### D-7 - Lección 03: costos, facturación y presupuestos

El 23 de septiembre se estudió costos y facturación.

1. ¿Un budget detiene automáticamente el gasto?
2. ¿Cuál es la diferencia entre una billing account y un project?
3. Enumera tres acciones de limpieza que previenen costos imprevistos.
4. Relaciona IAM con facturación: ¿tener permiso para crear un recurso significa que su costo está bloqueado por un presupuesto?

**Respuesta breve:** un presupuesto alerta, no detiene automáticamente el gasto. La cuenta de facturación paga cargos de proyectos vinculados; el proyecto agrupa recursos y configuración. IAM puede permitir crear un recurso facturable, mientras que el budget por sí solo no impide crearlo.

### D-21

El plan comenzó el 21 de septiembre de 2026; no existe una lección programada el 9 de septiembre dentro de esta secuencia. Registra `N/A` y no inventes contenido retroactivo.

### Recuperación de 90 segundos

Sin mirar, completa:

```text
Cloud Identity sirve para:
IAM sirve para:
Principal significa:
Role significa:
Scope significa:
Un grupo es preferible cuando:
Una concesión heredada:
Un basic role debe evitarse cuando:
```

---

## 24. Ficha de errores

Completa una fila por cada fallo en la práctica o las preguntas.

| # | Tema | Mi respuesta o acción | Respuesta correcta | Por qué fallé | Regla de decisión | Cómo lo explicaré sin apuntes | Fecha de repaso |
|---:|---|---|---|---|---|---|---|
| 1 |  |  |  |  |  |  |  |
| 2 |  |  |  |  |  |  |  |
| 3 |  |  |  |  |  |  |  |

### Errores críticos de hoy

Marca cualquiera que haya ocurrido:

- [ ] Confundí Cloud Identity con IAM.
- [ ] Confundí autenticación con autorización.
- [ ] Elegí un usuario cuando convenía un grupo.
- [ ] Elegí un basic role sin revisar roles predefinidos.
- [ ] Ignoré el alcance del binding.
- [ ] Olvidé la herencia desde un ancestro.
- [ ] Supuse que quitar una concesión elimina todas las demás.
- [ ] Confundí una service account con un usuario humano.
- [ ] Confundí `allAuthenticatedUsers` con el dominio corporativo.
- [ ] Dejé un cambio temporal sin retirar.

### Regla personal para el siguiente intento

```text
Antes de elegir una respuesta IAM, identificaré:
1. quién necesita acceso;
2. qué acción exacta necesita;
3. sobre qué recurso;
4. durante cuánto tiempo;
5. si existe herencia o más de un camino de acceso.
```

---

## 25. Criterios de autoevaluación

Marca solo lo que puedas demostrar:

- [ ] Explico autenticación frente a autorización con un ejemplo propio.
- [ ] Distingo Cloud Identity de IAM sin leer la definición.
- [ ] Construyo la cadena principal-rol-alcance.
- [ ] Explico permission, role, binding y allow policy.
- [ ] Identifico `user:`, `group:` y `serviceAccount:`.
- [ ] Explico por qué `allAuthenticatedUsers` no representa solo a mi organización.
- [ ] Diferencio basic, predefined y custom roles.
- [ ] Defiendo el uso de grupos funcionales.
- [ ] Calculo acceso directo y heredado en el escenario conceptual.
- [ ] Explico por qué un binding hijo no cancela un binding padre.
- [ ] Diseño tres equipos con permisos mínimos.
- [ ] Ejecuto consultas de solo lectura sin cambiar la política.
- [ ] Si hice un cambio opcional, demuestro que fue retirado.
- [ ] Obtengo al menos 8/10 en las preguntas.
- [ ] Justifico por qué cada distractor es incorrecto.

### Interpretación

- **13-15 criterios:** buen avance; repite el diseño sin mirar y conserva evidencia.
- **10-12 criterios:** comprensión parcial; refuerza herencia, alcance y grupos.
- **7-9 criterios:** vuelve al modelo mental y repite la práctica conceptual.
- **0-6 criterios:** no avances por memoria; reconstruye los conceptos desde la analogía.

Esta escala es una herramienta de estudio personal, no un umbral oficial de Google.

---

## 26. Registro de práctica

```text
Fecha:
Hora de inicio:
Hora de fin:
Duración total:

Modalidad:
[ ] Google Cloud Console
[ ] Cloud Shell / gcloud
[ ] Alternativa conceptual

Proyecto de laboratorio autorizado:
Cuenta activa verificada:
¿Hubo cambio IAM?:
¿Se retiró el cambio?:

Equipo 1 diseñado:
Equipo 2 diseñado:
Equipo 3 diseñado:

Comandos ejecutados:
1.
2.
3.

Resultado observado:

Puntuación del cuestionario:
Tiempo del cuestionario:

Duda principal:
Concepto a reforzar:
Próxima fecha de repaso:
```

---

## 27. Resumen de decisiones para ACE

1. Cloud Identity administra usuarios y grupos; IAM autoriza acciones sobre recursos.
2. Un principal recibe un rol sobre un alcance.
3. Los roles contienen permisos; los permisos no se conceden sueltos directamente.
4. Concede roles a grupos funcionales cuando varias personas comparten una función.
5. Prefiere roles predefinidos limitados sobre basic roles amplios.
6. Usa custom roles solo si un requisito real no cabe en roles predefinidos.
7. Evalúa rol y alcance juntos.
8. Las allow policies se heredan y las concesiones aplicables se acumulan.
9. Quitar un usuario de un grupo no elimina bindings directos u otros caminos de acceso.
10. `allUsers` y `allAuthenticatedUsers` requieren extrema cautela.
11. Administrar IAM no equivale automáticamente a administrar workloads.
12. Para desplegar Cloud Run debes considerar permisos sobre el servicio, la imagen y la service identity.

---

## 28. Documentación oficial consultada

Consultada y contrastada el **30 de septiembre de 2026**:

- [Associate Cloud Engineer certification](https://cloud.google.com/learn/certification/cloud-engineer)
- [IAM overview](https://cloud.google.com/iam/docs/overview)
- [IAM principals](https://cloud.google.com/iam/docs/principals-overview)
- [Understanding allow policies](https://cloud.google.com/iam/docs/allow-policies)
- [Roles overview](https://cloud.google.com/iam/docs/roles-overview)
- [Choose which type of role to use](https://cloud.google.com/iam/docs/choose-role-type)
- [Use IAM securely](https://cloud.google.com/iam/docs/using-iam-securely)
- [Best practices for using Google groups](https://cloud.google.com/iam/docs/groups-best-practices)
- [Using resource hierarchy for access control](https://cloud.google.com/iam/docs/resource-hierarchy-access-control)
- [Manage access to projects, folders, and organizations](https://cloud.google.com/iam/docs/granting-changing-revoking-access)
- [Overview of Cloud Identity](https://cloud.google.com/identity/docs/overview)
- [Create Cloud Identity user accounts](https://cloud.google.com/identity/docs/how-to/create-cloud-identity-user-accounts)
- [Set up Cloud Identity as a Google Cloud administrator](https://cloud.google.com/identity/docs/how-to/set-up-cloud-identity-admin)
- [Cloud Run IAM roles](https://cloud.google.com/run/docs/reference/iam/roles)
- [Artifact Registry access control](https://cloud.google.com/artifact-registry/docs/access-control)
- [`gcloud projects get-iam-policy`](https://cloud.google.com/sdk/gcloud/reference/projects/get-iam-policy)
- [`gcloud projects add-iam-policy-binding`](https://cloud.google.com/sdk/gcloud/reference/projects/add-iam-policy-binding)
- [`gcloud projects remove-iam-policy-binding`](https://cloud.google.com/sdk/gcloud/reference/projects/remove-iam-policy-binding)
- [`gcloud iam roles describe`](https://cloud.google.com/sdk/gcloud/reference/iam/roles/describe)
- [Cloud Logging access control](https://cloud.google.com/logging/docs/access-control)
- [Cloud Monitoring access control](https://cloud.google.com/monitoring/access-control)
- [Resource Manager roles and permissions](https://cloud.google.com/iam/docs/roles-permissions/resourcemanager)

Además se utilizó la guía oficial adjunta `associate_cloud_engineer_exam_guide_english.pdf`, especialmente:

- Section 1.1: granting members IAM roles within a project;
- Section 1.1: managing users and groups in Cloud Identity;
- Section 4.1: viewing and creating IAM policies;
- Section 4.1: attaching roles and policy inheritance;
- Section 4.1: managing role types.

---

## 29. Cierre

La decisión IAM correcta no empieza con un nombre de rol. Empieza con un requisito:

```text
¿Quién necesita realizar qué acción, sobre qué recurso y durante cuánto tiempo?
```

Después eliges un principal administrable, el rol predefinido más limitado y el alcance más pequeño viable. En SiteOps Tracker, los grupos funcionales desacoplan la rotación de personas de las políticas de los proyectos. Esa separación entre identidad y autorización es el núcleo de la lección.

Antes de cerrar, confirma:

- [ ] matriz de tres equipos completa;
- [ ] práctica o alternativa conceptual terminada;
- [ ] limpieza verificada;
- [ ] diez preguntas respondidas;
- [ ] soluciones revisadas después del intento;
- [ ] ficha de errores actualizada;
- [ ] repaso D-1 y D-7 realizado;
- [ ] D-21 registrado como no aplicable.
