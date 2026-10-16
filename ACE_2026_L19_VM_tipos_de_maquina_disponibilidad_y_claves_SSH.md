# ACE 2026 - Lección 19: VM: tipos de máquina, disponibilidad y claves SSH

**Fecha del plan:** 15 de octubre de 2026  
**Guía oficial ACE:** sección 2.1, Planning and implementing compute resources  
**Práctica del calendario:** crear o describir una VM y verificar metadatos, acceso y política de disponibilidad  
**Proyecto transversal:** SiteOps Tracker  
**Duración sugerida:** 75-90 minutos  
**Preguntas de examen:** 10 originales, en inglés

> Esta lección enseña y permite practicar el tema. Recibir el archivo no demuestra dominio: compruébalo con la práctica, el recuerdo activo y las preguntas sin consultar las soluciones.

## Objetivo

Al terminar deberías poder:

1. Explicar la diferencia entre **machine family**, **machine series** y **machine type**.
2. Elegir una forma de VM razonable a partir de CPU, memoria, arquitectura, costo y necesidad de control del sistema operativo.
3. Crear una VM Linux de laboratorio mediante Google Cloud Console o Google Cloud CLI.
4. Distinguir **live migration**, **automatic restart** y verdadera tolerancia a una caída zonal.
5. Explicar qué hacen la clave pública y la clave privada de SSH, y cómo Google Cloud administra claves al usar la consola o gcloud.
6. Consultar la configuración de la VM y el servidor de metadatos.
7. Eliminar los recursos del laboratorio y comprobar que no queden discos facturables.

## Prerrequisitos, explicados desde cero

### 1. Proyecto de Google Cloud

Un **project** es el contenedor de recursos, permisos, cuotas y cargos. Cada VM pertenece a un proyecto. Antes de crearla, verifica que estás en el proyecto de laboratorio correcto; un nombre visible puede repetirse, pero el **project ID** es único.

### 2. Facturación o proyecto temporal de Skills

Compute Engine consume recursos facturables. Usa preferentemente un proyecto temporal de Google Skills. Si trabajas en una cuenta propia, confirma facturación, presupuesto y limpieza. Un presupuesto envía alertas; no detiene por sí solo el consumo.

### 3. API de Compute Engine

La API **compute.googleapis.com** debe estar habilitada. Habilitar una API no crea una VM, pero permite que la consola y gcloud llamen al servicio.

### 4. Permisos

Para crear y administrar una VM se requieren permisos de Compute Engine. La documentación usa como referencia el rol predefinido **Compute Instance Admin (v1)**, aunque una organización puede otorgar un rol más limitado. Si se adjunta una service account también puede hacer falta permiso para usarla. En este laboratorio se elige **No service account** para reducir el alcance.

### 5. Red y subred

La interfaz de red de una VM vive en una **VPC** y una **subnet** regional. La VM, en cambio, es zonal. Si el proyecto no tiene la red **default**, selecciona la subred indicada por el laboratorio. No abras puertos HTTP o HTTPS: esta práctica solo necesita administración por SSH.

### 6. Cloud Shell o Google Cloud CLI

**Cloud Shell** ya incluye gcloud y recibe credenciales de tu sesión. En un equipo local, instala e inicializa Google Cloud CLI. El contexto de gcloud contiene cuenta, proyecto, región y zona; revisarlo evita crear recursos en el sitio equivocado.

### 7. Idea mínima de SSH

SSH usa criptografía de clave pública:

- La **private key** permanece en el cliente y nunca se comparte.
- La **public key** puede registrarse para la cuenta o la VM.
- El servidor comprueba que el cliente posee la clave privada correspondiente.
- La clave autentica; los permisos dentro de Google Cloud y Linux autorizan lo que puedes hacer.

## Caso de trabajo: SiteOps Tracker

SiteOps Tracker es una aplicación ficticia para auditar infraestructura en sedes, registrar activos, hallazgos, estados, responsables e historial. Su stack es React + TypeScript, Node.js + TypeScript y PostgreSQL.

Supón que una biblioteca nativa usada por el backend Node.js necesita una distribución Linux concreta, acceso a paquetes del sistema y control de procesos. Para aprender Compute Engine se crea una VM de desarrollo llamada **siteops-vm-l19**.

La VM no representa por sí sola una arquitectura de producción:

- El frontend React podría publicarse como contenido estático o servirse desde un servicio administrado.
- El API Node.js podría ejecutarse en Cloud Run si es stateless y no necesita control del host.
- PostgreSQL debería evaluarse en Cloud SQL o AlloyDB antes de autogestionarlo en la misma VM.
- Una sola VM es un punto único de falla ante errores zonales.

La razón pedagógica para usar Compute Engine hoy es practicar control del sistema operativo, forma de máquina, política de disponibilidad, metadatos y SSH.

## 1. Modelo mental: alquilar un taller

Imagina que una VM es un taller alquilado:

- La **family** es la clase de edificio: general, cómputo intensivo, memoria intensiva, almacenamiento intensivo o aceleradores.
- La **series** es la generación o línea de ese edificio.
- El **machine type** es el plano exacto: cuántos espacios de trabajo y cuánto almacén incluye.
- La **zone** es el parque industrial concreto.
- El **boot disk** es el almacén persistente.
- La **image** es el taller ya preparado con un sistema operativo.
- La **availability policy** indica qué hacer cuando Google mantiene el edificio físico.
- La clave SSH es la credencial criptográfica para abrir la puerta administrativa.

La decisión no empieza por el nombre del producto. Empieza por los requisitos de la carga.

## 2. Family, series y machine type

La documentación de Compute Engine organiza las opciones así:

| Nivel | Pregunta que responde | Ejemplo |
|---|---|---|
| Machine family | ¿Qué clase de carga voy a ejecutar? | General-purpose |
| Machine series | ¿Qué línea de CPU, arquitectura y capacidades necesito? | E2, N2, C4 |
| Machine type | ¿Cuántas vCPU y cuánta memoria tendrá esta VM? | e2-small, n2-standard-4 |

Por ejemplo, **n2-standard-4** pertenece a la serie N2, dentro de la familia general-purpose. La documentación indica que esa forma asigna 4 vCPU y 16 GB de memoria.

### Familias y señales de selección

| Familia o modalidad | Señal del requisito | Ejemplo en SiteOps Tracker | Por qué no elegirla automáticamente |
|---|---|---|---|
| General-purpose | CPU y memoria equilibradas | API Node.js, servidor de desarrollo, proceso auxiliar | Puede ser insuficiente para un cálculo muy intensivo o desperdiciar recursos si la carga es mínima |
| Shared-core | Carga pequeña, intermitente y no sensible a rendimiento sostenido | Demo personal con pocas solicitudes | Comparte tiempo de CPU; no es la opción para latencia estable o producción exigente |
| Compute-optimized | Alto uso sostenido de CPU por núcleo | Procesamiento CPU de fotografías de evidencias | Suele ser innecesaria para CRUD y tráfico bajo |
| Memory-optimized | Gran conjunto de datos activo en RAM | Analítica en memoria especializada | Es costosa y no soluciona por sí sola el diseño de PostgreSQL |
| Storage-optimized | Alta densidad o rendimiento local de almacenamiento | Procesamiento temporal especializado | No equivale a durabilidad de base de datos ni reemplaza backups |
| Network-optimized | Mucho ancho de banda y procesamiento de paquetes | Gateway especializado de gran escala | Excesiva para un API de portafolio |
| Accelerator-optimized | GPU para cómputo paralelo compatible | Inferencia o visión con GPU | Una GPU no acelera mágicamente cualquier código y eleva costo/operación |
| Custom machine type | Las formas predefinidas dejan CPU o memoria ociosa y existen mediciones | API con proporción medida poco común | Tiene restricciones por serie y la documentación señala una prima frente a una forma predefinida equivalente |

### Regla de examen

1. Busca primero una forma predefinida de propósito general.
2. Cambia de familia solo cuando el requisito lo justifique.
3. Usa una forma custom cuando mediciones reales demuestren una proporción CPU/memoria atípica.
4. Confirma que la serie y el tipo existen en la zona.
5. No confundas tamaño de VM con alta disponibilidad.

## 3. Cómo dimensionar sin adivinar

### Paso 1: identifica la unidad de trabajo

Para SiteOps Tracker, una solicitud típica podría:

1. autenticar al auditor;
2. consultar un activo;
3. registrar un hallazgo;
4. escribir historial en PostgreSQL;
5. devolver JSON.

Esto sugiere una carga equilibrada. No hay evidencia de que requiera GPU, decenas de GB de RAM ni CPU de alto rendimiento.

### Paso 2: separa desarrollo de producción

Una VM pequeña puede servir para una práctica o demo. No convierte esa misma forma en una recomendación de producción. En producción hacen falta métricas de CPU, memoria, latencia, concurrencia, crecimiento, recuperación y presupuesto.

### Paso 3: empieza pequeño y mide

Para esta práctica se usa **e2-small** como forma compartida de bajo costo. El objetivo es inspeccionar una VM, no ejecutar una prueba de rendimiento. Si el laboratorio no ofrece ese tipo en la zona, consulta los tipos disponibles y selecciona otro pequeño permitido.

### Paso 4: registra la decisión

Completa esta ficha antes de crear:

| Campo | Decisión de laboratorio |
|---|---|
| Carga | Backend Node.js de demostración |
| Necesidad de control del SO | Sí, para practicar paquetes, procesos y SSH |
| Machine family | General-purpose |
| Machine type | e2-small o equivalente permitido |
| Zona | La autorizada por el laboratorio |
| Sistema operativo | Debian 12 |
| Política de mantenimiento | MIGRATE |
| Reinicio automático | Activado |
| Alta disponibilidad zonal | No; es una sola VM |
| Service account | Ninguna para esta práctica |
| Datos persistentes importantes | Ninguno |

## 4. Disponibilidad: cuatro fallas distintas

La palabra “disponibilidad” puede ocultar problemas diferentes.

| Evento | Qué ocurre | Qué ayuda | Qué no basta |
|---|---|---|---|
| Mantenimiento del host físico | Google necesita intervenir el host | Live migration si la forma la admite | Reiniciar manualmente la aplicación |
| Falla del host o detención programada | La VM deja de ejecutarse | Automatic restart cuando es compatible | Una réplica en la misma VM |
| Caída del proceso Node.js | El SO sigue activo, pero el API murió | systemd, supervisor, contenedor, health check | Automatic restart de la VM |
| Caída de toda la zona | VM y recursos zonales pueden quedar inaccesibles | Réplicas en varias zonas, balanceo y diseño regional | Live migration de una sola VM |

### 4.1 Live migration

Con **onHostMaintenance = MIGRATE**, Compute Engine mueve una VM compatible a otro host durante mantenimiento con interrupción mínima. No todas las formas o configuraciones admiten migración en vivo. Las Spot VMs y los tipos que no la soportan usan **TERMINATE**.

### 4.2 Automatic restart

Con **automaticRestart = true**, Compute Engine intenta reiniciar la VM después de una falla del host o una detención programada compatible. No vigila la salud de tu API. Si Node.js termina pero Linux continúa, la política no lo detecta.

### 4.3 Por qué no es alta disponibilidad zonal

Ambas opciones actúan sobre eventos de host. Una caída zonal requiere recursos en más de una zona. En una arquitectura posterior, un **managed instance group** regional puede mantener instancias idénticas y distribuirlas; hoy solo debes reconocer que una VM no cubre ese requisito.

### 4.4 Decisión para SiteOps Tracker

Para la VM de laboratorio:

- **MIGRATE**: reduce la interrupción durante mantenimiento de host compatible.
- **Automatic restart: On**: recupera la VM ante ciertos fallos de infraestructura.
- **Una sola zona**: aceptable para la práctica, no para un objetivo de continuidad regional.

## 5. SSH y metadatos

### 5.1 Dos formas administradas por Google Cloud

| Método | Comportamiento de clave indicado por la documentación | Uso |
|---|---|---|
| SSH desde la consola | Crea una clave efímera | Acceso interactivo rápido |
| gcloud compute ssh | Crea y administra una clave persistente para el cliente | Acceso repetible desde Cloud Shell o terminal |

En ambos casos Google Cloud configura la conexión. No copies una clave privada al proyecto, a Git, al Markdown ni a la VM.

### 5.2 Metadata-based SSH keys

Si OS Login no está habilitado, las claves públicas pueden residir en metadatos del proyecto o de la instancia. El guest agent del sistema operativo transforma esa información en cuentas y claves autorizadas.

Riesgo importante: al establecer manualmente el valor **ssh-keys** mediante gcloud, debes conservar las claves existentes. Reemplazar el valor con un archivo que solo contiene una clave elimina las demás del valor de metadatos.

### 5.3 OS Login

OS Login asocia el acceso Linux con identidades y permisos IAM, en lugar de administrar largas listas de claves en metadatos. Se estudiará con detalle en la lección 22. Para el examen, reconoce la señal: acceso centralizado, revocación mediante IAM y menor administración manual de claves.

### 5.4 IAP frente a una IP externa

Una VM sin IP externa puede recibir conexiones administrativas mediante **IAP TCP forwarding**, si existen permisos IAM y una regla de firewall que permita SSH desde el rango de IAP. La opción de gcloud es **--tunnel-through-iap**.

No abras TCP 22 a **0.0.0.0/0** por comodidad. En un laboratorio propio, limita la fuente o usa IAP. El acceso por IAP no requiere una IP pública.

### 5.5 Servidor de metadatos

Cada VM de Compute Engine puede consultar un servidor local de metadatos. El encabezado **Metadata-Flavor: Google** es obligatorio. Entre otros datos, puedes leer:

- nombre de la instancia;
- tipo de máquina;
- zona;
- project ID;
- imagen de arranque;
- aviso de mantenimiento.

Los procesos dentro de la VM también pueden solicitar credenciales de una service account adjunta. Por eso no se debe entregar acceso SSH innecesario: un usuario dentro de la VM puede alcanzar los permisos de esa identidad. Esta práctica no adjunta una service account.

## 6. Decisión entre servicios

| Requisito | Elección principal | Por qué | Por qué no las alternativas |
|---|---|---|---|
| Control del SO, paquetes nativos y procesos | Compute Engine | Acceso a una VM completa | Cloud Run y functions abstraen el host; GKE añade Kubernetes |
| API stateless HTTP en contenedor, operación mínima y escalado a cero | Cloud Run | Plataforma administrada y orientada a solicitudes | Una VM requiere parches, capacidad y proceso de despliegue; GKE puede ser exceso |
| Muchas cargas que necesitan recursos y API de Kubernetes | GKE | Orquestación Kubernetes | Una VM aislada no ofrece el modelo del clúster; Cloud Run no expone toda la API de Kubernetes |
| Pequeño manejador de eventos | Cloud Run functions | Modelo de función y eventos | Una VM queda encendida y requiere administración |
| Varias VM idénticas con autoscaling y reparación | Managed instance group | Plantilla, escalado y health checks | Una VM manual no se reemplaza ni escala; Cloud Run sería mejor si no se necesita control del SO |
| PostgreSQL transaccional administrado | Cloud SQL para PostgreSQL, sujeto a requisitos | Reduce tareas de parcheo, backups y operación del motor | PostgreSQL en la VM mezcla aplicación y datos, aumenta el dominio de falla y la carga operativa |

### Decisión razonada para hoy

Se elige Compute Engine porque el ejercicio exige practicar una VM, su SO y SSH. Para el SiteOps Tracker real, si el backend puede empaquetarse como contenedor stateless y no necesita control del host, Cloud Run probablemente reduciría operación. La elección de hoy es didáctica, no una recomendación automática de producción.

## 7. Práctica guiada con Google Cloud Console

Los nombres de menús pueden variar ligeramente. No ejecutes a la vez la ruta de consola y la ruta CLI: elige una para crear una sola VM y usa la otra únicamente para inspeccionar si lo deseas.

### 7.1 Comprobaciones previas

1. Abre Google Cloud Console.
2. Confirma el **project ID** en el selector superior.
3. Ve a **APIs & Services > Enabled APIs & services** y confirma **Compute Engine API**.
4. Ve a **Compute Engine > VM instances**.
5. Si el proyecto de laboratorio define una región, zona, VPC o subred, usa exactamente esas opciones.

### 7.2 Crear

1. Selecciona **Create instance**.
2. Nombre: **siteops-vm-l19**.
3. Elige la región y zona autorizadas.
4. En **Machine configuration**:
   - familia: General-purpose;
   - serie: E2;
   - tipo: e2-small, o el tipo pequeño permitido.
5. En **OS and storage**, selecciona Debian 12 y un disco de arranque pequeño.
6. En **Management > Availability policies**:
   - On host maintenance: **Migrate VM instance**;
   - Automatic restart: **On**.
7. En **Security**, no agregues claves privadas. Usa el mecanismo administrado por la consola.
8. En **Identity and API access**, elige **No service account** si la política del laboratorio lo permite.
9. No marques **Allow HTTP traffic** ni **Allow HTTPS traffic**.
10. Revisa el estimado mostrado por la consola y selecciona **Create**.

### 7.3 Verificar

1. Espera a que el estado sea **Running**.
2. Abre el detalle de la VM y registra:
   - zona;
   - machine type;
   - imagen o disco de arranque;
   - IP interna;
   - IP externa, si existe;
   - On host maintenance;
   - Automatic restart;
   - service account.
3. Selecciona **SSH**. La consola crea una clave efímera para la sesión.
4. Dentro de la VM ejecuta las consultas de metadatos de la sección 8.6.
5. Sal con **exit**.

## 8. Práctica guiada con Google Cloud CLI

Los comandos y opciones de esta sección se contrastaron el 15 de octubre de 2026 con la documentación oficial de gcloud y Compute Engine enlazada al final.

### 8.1 Define variables

Sustituye el project ID y, si el laboratorio lo exige, la zona. No pegues un nombre de proyecto cuando se solicita un ID.

~~~bash
PROJECT_ID="TU_PROJECT_ID"
ZONE="us-central1-a"
VM_NAME="siteops-vm-l19"
MACHINE_TYPE="e2-small"
~~~

Comprueba los valores antes de crear:

~~~bash
printf 'PROJECT_ID=%s\nZONE=%s\nVM_NAME=%s\nMACHINE_TYPE=%s\n' \
  "$PROJECT_ID" "$ZONE" "$VM_NAME" "$MACHINE_TYPE"
~~~

### 8.2 Revisa identidad y contexto

~~~bash
gcloud auth list --filter=status:ACTIVE --format="value(account)"
gcloud config get-value project
gcloud config set project "$PROJECT_ID"
~~~

Resultado esperado: aparece tu cuenta activa y gcloud confirma el project ID elegido.

### 8.3 Habilita la API y valida el tipo

~~~bash
gcloud services enable compute.googleapis.com \
  --project="$PROJECT_ID"

gcloud compute machine-types describe "$MACHINE_TYPE" \
  --project="$PROJECT_ID" \
  --zone="$ZONE"
~~~

Si el segundo comando devuelve **NOT_FOUND**, no inventes un nombre: consulta los tipos disponibles en esa zona.

~~~bash
gcloud compute machine-types list \
  --project="$PROJECT_ID" \
  --zones="$ZONE" \
  --limit=20
~~~

### 8.4 Crea la VM

Este comando usa una imagen pública Debian 12, un disco de arranque pequeño, migración en vivo cuando es compatible, reinicio automático y ninguna service account.

~~~bash
gcloud compute instances create "$VM_NAME" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --machine-type="$MACHINE_TYPE" \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=10GB \
  --boot-disk-type=pd-balanced \
  --maintenance-policy=MIGRATE \
  --restart-on-failure \
  --no-service-account \
  --labels=app=siteops-tracker,env=lab
~~~

Notas:

- Si el proyecto no tiene red **default**, añade la opción **--subnet=NOMBRE_DE_SUBRED** con la subred autorizada.
- Si una política prohíbe IP externa, crea o usa la VM sin IP pública y conéctate mediante IAP ya configurado.
- No añadas reglas de firewall abiertas a Internet para “hacer que funcione”.

### 8.5 Inspecciona configuración y disponibilidad

~~~bash
gcloud compute instances describe "$VM_NAME" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --format="yaml(name,status,zone,machineType,scheduling,metadata.items,disks)"
~~~

La referencia oficial propone también aplanar el bloque de scheduling:

~~~bash
gcloud compute instances describe "$VM_NAME" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --flatten=scheduling
~~~

Busca:

- **status: RUNNING**;
- un machineType cuyo recurso termina en el tipo elegido;
- **onHostMaintenance: MIGRATE**;
- **automaticRestart: true**;
- disco de arranque con **autoDelete: true**.

### 8.6 Accede por SSH

~~~bash
gcloud compute ssh "$VM_NAME" \
  --project="$PROJECT_ID" \
  --zone="$ZONE"
~~~

gcloud puede pedir autorización para generar la pareja de claves. La clave privada permanece en el cliente. Una vez dentro:

~~~bash
curl "http://metadata.google.internal/computeMetadata/v1/instance/name" \
  -H "Metadata-Flavor: Google"

curl "http://metadata.google.internal/computeMetadata/v1/instance/machine-type" \
  -H "Metadata-Flavor: Google"

curl "http://metadata.google.internal/computeMetadata/v1/instance/zone" \
  -H "Metadata-Flavor: Google"

curl "http://metadata.google.internal/computeMetadata/v1/project/project-id" \
  -H "Metadata-Flavor: Google"
~~~

Comprueba además el sistema operativo:

~~~bash
uname -a
cat /etc/os-release
~~~

Sal de la VM:

~~~bash
exit
~~~

### 8.7 Ruta por IAP si no hay IP externa

Solo funciona si el proyecto ya tiene permisos IAM y firewall de IAP configurados:

~~~bash
gcloud compute ssh "$VM_NAME" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --tunnel-through-iap
~~~

IAP TCP forwarding requiere permitir el acceso autorizado y el tráfico TCP 22 desde el rango de IAP; no improvises una regla global en un proyecto administrado.

### 8.8 Evidencia del laboratorio

Completa sin incluir secretos:

| Evidencia | Tu registro |
|---|---|
| Project ID parcialmente oculto | |
| Zona | |
| Machine type | |
| Estado | |
| onHostMaintenance | |
| automaticRestart | |
| Nombre devuelto por metadata | |
| Tipo devuelto por metadata | |
| Método SSH: consola, gcloud o IAP | |
| Service account adjunta | |
| Disco autoDelete | |
| Hora de creación | |
| Hora de eliminación | |

No pegues:

- clave privada;
- contenido completo de tokens;
- cookies;
- credenciales;
- datos personales;
- nombres de proyectos internos o privados.

## 9. Resultado esperado

Al completar la ruta práctica debes poder demostrar:

1. Existe una sola VM de laboratorio llamada **siteops-vm-l19**.
2. El estado llega a **RUNNING**.
3. El tipo de máquina coincide con la forma validada en la zona.
4. La política muestra **MIGRATE** y **automaticRestart: true**, si la forma elegida los admite.
5. Puedes abrir una sesión SSH mediante el método permitido.
6. Las consultas al servidor de metadatos devuelven nombre, tipo, zona y project ID.
7. El disco de arranque tiene autoDelete activado o se elimina explícitamente.
8. Tras la limpieza, la VM ya no aparece y no queda un disco huérfano.

## 10. Alternativa conceptual completa sin cuenta ni crédito

Si no puedes crear recursos, realiza la siguiente simulación. Cubre el razonamiento del ejercicio aunque no sustituye experiencia práctica.

### 10.1 Revisa la solicitud

~~~text
Workload: SiteOps Tracker development API
Runtime: Node.js + TypeScript
Traffic: low and irregular
OS access: required for one native dependency
Persistent business data: stored outside the VM
Downtime target: lab only
Zone failure tolerance: not required for this exercise
Budget: minimize spend
~~~

### 10.2 Elige y justifica

Completa:

| Decisión | Tu respuesta | Requisito que la determina |
|---|---|---|
| Compute Engine o Cloud Run | | |
| General-purpose u optimizada | | |
| Predefined o custom | | |
| MIGRATE o TERMINATE | | |
| Automatic restart on/off | | |
| SSH por IP externa o IAP | | |

### 10.3 Interpreta este resultado simulado

~~~yaml
name: siteops-vm-l19
status: RUNNING
zone: projects/example/zones/us-central1-a
machineType: projects/example/zones/us-central1-a/machineTypes/e2-small
scheduling:
  automaticRestart: true
  onHostMaintenance: MIGRATE
  provisioningModel: STANDARD
disks:
  - boot: true
    autoDelete: true
metadata:
  items: []
~~~

Responde:

1. ¿Qué parte demuestra el tipo de máquina?
2. ¿Qué ocurriría durante mantenimiento de host compatible?
3. ¿Qué ocurriría si el proceso Node.js se detuviera pero Linux siguiera activo?
4. ¿Qué requisito no cubre esta única VM?
5. ¿Qué recurso desaparecería al borrar la VM según autoDelete?
6. ¿Dónde debería permanecer la clave privada?

### 10.4 Respuesta de contraste

- Compute Engine se justifica por la necesidad explícita de control del SO.
- General-purpose y una forma pequeña son razonables para una demo sin mediciones de demanda.
- MIGRATE atiende mantenimiento de host compatible.
- Automatic restart ayuda ante fallos de host, no ante un proceso Node.js caído.
- La única VM no tolera la pérdida de la zona.
- El disco de arranque se elimina con la VM porque autoDelete es true.
- La clave privada permanece en el cliente.

## 11. Solución de problemas

| Síntoma | Diagnóstico probable | Acción segura |
|---|---|---|
| API has not been used or is disabled | Compute Engine API no está habilitada o la activación aún se propaga | Habilita compute.googleapis.com, espera unos minutos y reintenta |
| PERMISSION_DENIED al crear | Falta compute.instances.create u otro permiso sobre red/disco | Revisa el proyecto activo y solicita el rol mínimo al administrador |
| Machine type not found | El tipo no existe en la zona o el nombre es incorrecto | Ejecuta machine-types describe/list para esa zona |
| RESOURCE_EXHAUSTED | Capacidad temporal o cuota insuficiente | Prueba otro tipo o zona permitidos; no eleves cuota sin justificación |
| Red default no encontrada | El proyecto no tiene VPC default | Usa la subred proporcionada por el laboratorio |
| SSH timeout | VM detenida, ruta o firewall ausente, IP incorrecta | Confirma RUNNING, revisa red y usa IAP si está configurado |
| Permission denied (publickey) | Clave, usuario, OS Login o metadatos no coinciden | Usa gcloud compute ssh y revisa si OS Login está habilitado |
| IAP connection failed | Falta rol de túnel o regla para el rango de IAP | Revisa IAM y firewall; no abras SSH a todo Internet |
| Metadata devuelve 403 | Falta el encabezado requerido | Añade Metadata-Flavor: Google exactamente |
| La política no puede cambiarse | El tipo o modelo no admite esa modificación | Consulta limitaciones; algunos tipos requieren recreación |
| El API no responde aunque la VM está RUNNING | El proceso Node.js falló | Usa un gestor de procesos o health check; la política de VM no vigila la aplicación |

Herramienta oficial de diagnóstico de SSH:

~~~bash
gcloud compute ssh "$VM_NAME" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --troubleshoot
~~~

Si diagnosticas IAP, añade **--tunnel-through-iap**. Algunas pruebas pueden omitirse si no tienes permisos de Network Management.

## 12. Impacto en costos

No se incluyen precios numéricos porque cambian por región, serie y fecha. Verifica la consola y la página oficial de precios antes de crear.

Los componentes que pueden generar cargos son:

- vCPU y memoria mientras la VM está en ejecución;
- tamaño y tipo del disco;
- dirección IP externa según su uso y estado;
- transferencia de datos saliente;
- imágenes de SO con licencia premium;
- recursos adicionales que no se usan en esta práctica.

Puntos de decisión:

- Una forma más grande cuesta más y no garantiza mejor arquitectura.
- Una custom machine puede ajustar la proporción CPU/memoria, pero la documentación indica un costo ligeramente superior frente a una forma predefinida equivalente.
- Detener una VM elimina el cargo de uso de la VM mientras está detenida, pero discos y direcciones estáticas pueden seguir generando cargos.
- El disco de arranque suele tener autoDelete activado; verifícalo, no lo supongas.
- Live migration y automatic restart no reemplazan un diseño multi-zona.

## 13. Limpieza de recursos

Primero confirma exactamente qué vas a borrar:

~~~bash
gcloud compute instances describe "$VM_NAME" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --format="yaml(name,status,zone,disks)"
~~~

Si es la VM desechable de esta práctica:

~~~bash
gcloud compute instances delete "$VM_NAME" \
  --project="$PROJECT_ID" \
  --zone="$ZONE"
~~~

Confirma la eliminación:

~~~bash
gcloud compute instances list \
  --project="$PROJECT_ID" \
  --filter="name=$VM_NAME"

gcloud compute disks list \
  --project="$PROJECT_ID" \
  --filter="name=$VM_NAME"
~~~

Resultado esperado: ninguna VM y ningún disco de laboratorio. Si queda un disco, verifica que no contenga datos necesarios y elimínalo desde **Compute Engine > Disks** o con el comando específico del recurso. No borres discos compartidos ni recursos cuyo propósito desconozcas.

Si reservaste una IP estática, revísala en **VPC network > IP addresses**. La práctica no requiere reservar una.

## 14. Glosario bilingüe

| English | Español | Definición útil |
|---|---|---|
| Virtual machine (VM) | Máquina virtual | Sistema de cómputo virtual con SO, CPU, memoria, disco y red |
| Machine family | Familia de máquinas | Categoría optimizada para un patrón de carga |
| Machine series | Serie de máquinas | Línea o generación dentro de una familia |
| Machine type | Tipo de máquina | Forma concreta de vCPU y memoria |
| Predefined machine type | Tipo predefinido | Forma ofrecida por Google con proporción fija |
| Custom machine type | Tipo personalizado | Combinación de vCPU y memoria dentro de límites de una serie |
| Shared-core | Núcleo compartido | Forma pequeña que comparte tiempo de CPU |
| vCPU | CPU virtual | Unidad de capacidad de procesamiento asignada |
| Boot disk | Disco de arranque | Disco persistente que contiene el SO |
| Public image | Imagen pública | Imagen de SO mantenida por Google o un proveedor |
| Image family | Familia de imágenes | Referencia a la imagen no obsoleta más reciente de una línea |
| Zone | Zona | Ubicación aislada dentro de una región |
| Host maintenance | Mantenimiento del host | Trabajo sobre la infraestructura física |
| Live migration | Migración en vivo | Movimiento de una VM compatible a otro host con mínima interrupción |
| Automatic restart | Reinicio automático | Reinicio de la VM tras ciertos fallos o detenciones |
| Availability policy | Política de disponibilidad | Comportamiento ante mantenimiento y fallos del host |
| Provisioning model | Modelo de aprovisionamiento | Condiciones como STANDARD o Spot |
| SSH key pair | Par de claves SSH | Clave pública y privada para autenticación |
| Public key | Clave pública | Parte compartible que se registra para verificar acceso |
| Private key | Clave privada | Secreto que permanece en el cliente |
| Instance metadata | Metadatos de instancia | Valores de configuración y datos accesibles para la VM |
| Metadata server | Servidor de metadatos | Endpoint local de Compute Engine dentro de la VM |
| OS Login | OS Login | Gestión de acceso Linux mediante identidades y permisos IAM |
| IAP TCP forwarding | Reenvío TCP de IAP | Túnel autorizado hacia una VM sin requerir IP pública |
| External IP address | Dirección IP externa | Dirección enrutable desde Internet |
| Internal IP address | Dirección IP interna | Dirección privada dentro de la VPC |
| Auto-delete | Eliminación automática | Propiedad que borra un disco junto con la VM |
| Least privilege | Mínimo privilegio | Solo los permisos necesarios para una tarea |

## 15. Repaso activo espaciado

Responde sin consultar las lecciones anteriores. Escribe una frase de decisión y una de justificación.

### Día hábil anterior: lección 18

1. A stateless Node.js API receives irregular HTTP traffic and needs no OS access. Which service is the default candidate, and why?
2. A workload requires full Kubernetes APIs, multiple controllers, and portable manifests. Which service is the better fit?
3. What requirement would make Compute Engine preferable to Cloud Run?

### Hace 7 días: lección 14

1. What problem does Workforce Identity Federation solve for external human users?
2. Why is issuing long-lived service account keys to employees a poor substitute?
3. When might a separate standalone organization be considered?

### Hace 21 días: lección 4

1. What is the difference between a region and a zone?
2. Does placing two VMs in the same zone protect against a zonal outage?
3. Why must you verify product and machine-type availability in a chosen location?

### Comprobación del repaso

- Lección 18: Cloud Run es candidato para HTTP stateless sin control del host; GKE para requisitos propios de Kubernetes; Compute Engine cuando se necesita control del SO o compatibilidad de VM.
- Lección 14: Workforce Identity Federation permite acceso de identidades externas sin convertir claves de service account en credenciales humanas permanentes; una organización independiente responde a límites administrativos o de identidad concretos.
- Lección 4: una región contiene zonas; dos VM en la misma zona comparten el riesgo zonal; productos y tipos no están disponibles en todas las ubicaciones.

Marca:

- [ ] Respondí las nueve preguntas sin apuntes.
- [ ] Expliqué el requisito que determina cada elección.
- [ ] Anoté cualquier duda en la ficha de errores.

## 16. Preguntas originales tipo examen

No son preguntas filtradas ni copiadas del examen. Resuélvelas antes de abrir la sección de soluciones.

### Question 1

SiteOps Tracker needs a small development VM for a low-traffic Node.js API. Engineers must install a native Linux package and access the operating system. Which option is the best starting point?

A. A small general-purpose predefined machine type  
B. A memory-optimized machine with the maximum available RAM  
C. An accelerator-optimized machine with multiple GPUs  
D. A storage-optimized bare metal instance

### Question 2

A batch component performs sustained CPU-bound transformation of audit images. Memory use is moderate, and the code does not support GPUs. Which machine family is the best fit?

A. Memory-optimized  
B. Compute-optimized  
C. Accelerator-optimized  
D. Shared-core

### Question 3

Monitoring shows that a VM consistently needs a CPU-to-memory ratio that is not available in an appropriate predefined type. What should the engineer evaluate next?

A. A custom machine type in a supported series  
B. A larger boot disk only  
C. A second external IP address  
D. Project-level SSH keys

### Question 4

A supported STANDARD VM should experience minimal disruption during planned host maintenance and should restart after a host failure. Which two settings should be configured? **Choose two.**

A. onHostMaintenance = MIGRATE  
B. onHostMaintenance = TERMINATE  
C. automaticRestart = true  
D. automaticRestart = false

### Question 5

SiteOps Tracker must remain available if an entire zone becomes unavailable. Which design best meets the requirement?

A. One VM with live migration enabled  
B. One VM with automatic restart enabled  
C. Multiple instances distributed across zones behind an appropriate load-balancing design  
D. One larger custom machine type in a single zone

### Question 6

An engineer connects to a Linux VM by using gcloud compute ssh for the first time. What does the documented workflow do?

A. It uploads the private key to instance metadata  
B. It can create and manage an SSH key pair for the client and publish the public key  
C. It converts the VM service account key into an SSH key  
D. It disables IAM checks permanently

### Question 7

An administrator uses gcloud compute instances add-metadata with metadata-from-file to update the ssh-keys value. The VM already has three instance-level keys. What must the administrator do?

A. Include the existing keys and the new key in the file  
B. Include only the private key for the new user  
C. Delete the VM service account first  
D. Change the host maintenance policy to TERMINATE

### Question 8

A VM has no external IP address. The organization has configured IAP TCP forwarding, the required firewall rule, and IAM permissions. Which command option should the engineer use to force SSH through IAP?

A. --internal-ip  
B. --tunnel-through-iap  
C. --maintenance-policy=MIGRATE  
D. --no-service-account

### Question 9

The Node.js process stops, but the VM remains RUNNING and the operating system is healthy. Why does automaticRestart not solve the incident?

A. automaticRestart only applies to Windows VMs  
B. automaticRestart responds to VM or host events, not application-process health  
C. automaticRestart requires a GPU  
D. automaticRestart only changes the boot disk type

### Question 10

An engineer stops a lab VM and plans to leave it stopped for a month. Which statement is correct?

A. All charges stop, including persistent disks and static IP addresses  
B. VM usage charges stop, but attached disks and static IP addresses can continue to incur charges  
C. Stopping automatically deletes the boot disk  
D. Stopping converts the VM into a Cloud Run service

## 17. Ficha de errores

Registra tanto fallos de preguntas como fallos del laboratorio.

| Nº | Pregunta o paso | Mi respuesta/acción | Respuesta correcta | Tipo de error | Por qué fallé | Regla de decisión recuperable | Fecha de nuevo intento |
|---:|---|---|---|---|---|---|---|
| 1 | | | | concepto / lectura / CLI / descuido | | | |
| 2 | | | | concepto / lectura / CLI / descuido | | | |
| 3 | | | | concepto / lectura / CLI / descuido | | | |

Plantilla de corrección:

1. **Señal omitida del escenario:**  
2. **Regla correcta en una frase:**  
3. **Por qué mi distractor parecía plausible:**  
4. **Nuevo ejemplo sin mirar apuntes:**  
5. **Comando o evidencia que lo confirma:**  

## 18. Criterios de autoevaluación

Puntúa cada criterio de 0 a 2:

- 0: no puedo explicarlo.
- 1: puedo reconocerlo con apoyo.
- 2: puedo decidir y justificar sin apoyo.

| Criterio | 0-2 |
|---|---:|
| Distingo family, series y machine type | |
| Elijo una familia desde requisitos, no desde nombres | |
| Explico MIGRATE frente a TERMINATE | |
| Explico automatic restart frente a salud de aplicación | |
| Identifico que una sola VM no tolera una caída zonal | |
| Explico clave pública frente a privada | |
| Distingo claves administradas, metadatos y OS Login | |
| Puedo crear y describir la VM con gcloud | |
| Puedo consultar metadatos desde dentro de la VM | |
| Puedo limpiar VM, disco e IP sin afectar otros recursos | |

Interpretación:

- **17-20:** buen desempeño de esta sesión; vuelve a comprobarlo sin notas en 24 horas.
- **13-16:** refuerza los criterios con 0 o 1 y repite las preguntas falladas.
- **0-12:** rehace la alternativa conceptual y luego el laboratorio guiado.

Meta personal del plan para este bloque: al menos **8/10** en las preguntas y evidencia completa de creación/inspección/limpieza. Esta meta es de estudio, no un umbral oficial del examen.

## 19. Enlaces oficiales

Fuentes consultadas y comandos verificados el 15 de octubre de 2026:

- [Associate Cloud Engineer certification](https://cloud.google.com/learn/certification/cloud-engineer)
- [Create and start a Compute Engine instance](https://docs.cloud.google.com/compute/docs/instances/create-start-instance)
- [Create a VM from a public image](https://docs.cloud.google.com/compute/docs/instances/create-vm-from-public-image)
- [Machine families resource and comparison guide](https://docs.cloud.google.com/compute/docs/machine-resource)
- [gcloud compute instances create reference](https://docs.cloud.google.com/sdk/gcloud/reference/compute/instances/create)
- [gcloud compute machine-types list reference](https://docs.cloud.google.com/sdk/gcloud/reference/compute/machine-types/list)
- [Set the host maintenance policy](https://docs.cloud.google.com/compute/docs/instances/setting-vm-host-options)
- [About host events](https://docs.cloud.google.com/compute/docs/instances/host-maintenance-overview)
- [Connect to Linux VMs](https://docs.cloud.google.com/compute/docs/connect/standard-ssh)
- [Add SSH keys to VMs](https://docs.cloud.google.com/compute/docs/connect/add-ssh-keys)
- [Troubleshoot SSH errors](https://docs.cloud.google.com/compute/docs/troubleshooting/troubleshooting-ssh-errors)
- [Use IAP for TCP forwarding](https://docs.cloud.google.com/iap/docs/using-tcp-forwarding)
- [View and query VM metadata](https://docs.cloud.google.com/compute/docs/metadata/querying-metadata)
- [Stop or restart a Compute Engine instance](https://docs.cloud.google.com/compute/docs/instances/stop-start-instance)
- [gcloud compute instances delete reference](https://docs.cloud.google.com/sdk/gcloud/reference/compute/instances/delete)
- [Compute Engine VM pricing](https://cloud.google.com/products/compute/pricing)

## 20. Soluciones justificadas

### Answer 1: A

**A is correct.** The workload is small and balanced, but it explicitly needs operating-system access. A small general-purpose predefined type is the simplest starting point.

- **B is incorrect:** no memory-intensive requirement is stated.
- **C is incorrect:** GPUs add cost and the scenario does not identify GPU-compatible work.
- **D is incorrect:** the workload does not require bare metal or storage density.

### Answer 2: B

**B is correct.** Sustained CPU-bound work with moderate memory and no GPU support is the signal for a compute-optimized family.

- **A is incorrect:** memory is not the bottleneck.
- **C is incorrect:** accelerator-optimized machines help only when the software can use the accelerator.
- **D is incorrect:** shared-core types are intended for small, non-intensive workloads and do not provide the required sustained CPU profile.

### Answer 3: A

**A is correct.** A custom machine type can match a measured, unusual CPU-to-memory ratio when a supported general-purpose series allows it.

- **B is incorrect:** boot disk capacity does not change CPU or RAM.
- **C is incorrect:** an IP address does not resize compute resources.
- **D is incorrect:** SSH keys control access, not resource shape.

### Answer 4: A and C

**A and C are correct.** MIGRATE requests live migration during supported host maintenance; automaticRestart=true requests recovery after relevant host failures or programmed stops.

- **B is incorrect:** TERMINATE stops the VM during the maintenance event and creates more disruption when migration is supported.
- **D is incorrect:** disabling restart works against the stated recovery requirement.

These settings still do not make one VM resilient to a complete zonal outage.

### Answer 5: C

**C is correct.** Zonal resilience requires independent capacity in more than one zone and a way to route traffic to healthy capacity.

- **A is incorrect:** live migration moves a VM between hosts; it does not move a zonal VM out of a failed zone.
- **B is incorrect:** automatic restart cannot start the same zonal resource while the zone is unavailable.
- **D is incorrect:** a larger VM remains a single zonal failure point.

### Answer 6: B

**B is correct.** gcloud compute ssh can generate a client key pair when needed and publish/manage the public key for the connection. The private key stays with the client.

- **A is incorrect:** private keys must never be placed in metadata.
- **C is incorrect:** service account credentials and SSH keys serve different purposes.
- **D is incorrect:** SSH does not permanently disable IAM or other access checks.

### Answer 7: A

**A is correct.** Updating the ssh-keys metadata value replaces that value. The file must contain the existing public keys plus the new public key.

- **B is incorrect:** metadata receives public keys, never the private key.
- **C is incorrect:** the VM service account is unrelated to preserving SSH metadata entries.
- **D is incorrect:** maintenance behavior has no relationship to SSH key-list replacement.

### Answer 8: B

**B is correct.** The documented option --tunnel-through-iap forces gcloud compute ssh to use IAP TCP forwarding.

- **A is incorrect:** --internal-ip tells gcloud to connect directly to the internal address and not use IAP; it requires network reachability from the client.
- **C is incorrect:** the maintenance policy controls host events.
- **D is incorrect:** removing a service account reduces workload identity exposure but does not create an SSH tunnel.

### Answer 9: B

**B is correct.** automaticRestart is an infrastructure scheduling property. It is not an application supervisor and does not know whether the Node.js process is healthy.

- **A is incorrect:** the setting is not limited to Windows.
- **C is incorrect:** GPU presence is irrelevant.
- **D is incorrect:** the setting does not change storage type.

Use a process manager and health checks; use multiple instances when the availability requirement demands redundancy.

### Answer 10: B

**B is correct.** A stopped VM does not incur normal VM usage charges, but retained disks and static IP addresses can continue to incur charges until deleted.

- **A is incorrect:** attached resources have independent lifecycles and pricing.
- **C is incorrect:** stopping retains the boot disk; deletion behavior is governed separately by autoDelete when the VM is deleted.
- **D is incorrect:** stopping does not change the product or deployment model.
