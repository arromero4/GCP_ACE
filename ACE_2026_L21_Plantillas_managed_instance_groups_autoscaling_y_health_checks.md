# ACE 2026 · Lección 21 — Plantillas, managed instance groups, autoscaling y health checks

**Fecha:** 19 de octubre de 2026  
**Guía oficial ACE:** sección 2.1, planificación y configuración de recursos de cómputo  
**Práctica del plan:** dibujar y configurar o simular una plantilla y un managed instance group (MIG) con comprobación de estado  
**Proyecto transversal:** SiteOps Tracker  
**Duración sugerida:** 75–100 minutos  
**Estado al recibir este archivo:** material preparado; todavía debes realizar la práctica y comprobar tu comprensión.

---

## 1. Objetivo

Al finalizar la sesión podrás:

1. Explicar desde cero qué papel cumplen una **instance template**, un **managed instance group**, el **autoscaler**, un **health check** y la política de **autohealing**.
2. Distinguir una VM individual, un MIG zonal y un MIG regional, y justificar cuál elegir según disponibilidad, estado y operación.
3. Crear una plantilla inmutable y un MIG zonal pequeño para una simulación de la API de SiteOps Tracker.
4. Configurar escalado por CPU y una comprobación HTTP de estado sin confundir escalado, balanceo y reparación.
5. Diagnosticar un backend o una VM marcada como <code>UNHEALTHY</code>, incluso cuando el acceso SSH también presenta un timeout.
6. Estimar el impacto en costos y eliminar todos los recursos de la práctica.

### Evidencia mínima de hoy

Conserva en este archivo o en tus notas:

- un diagrama con plantilla, MIG, autoscaler, health check y VMs;
- la salida de <code>list-instances</code> o una captura equivalente de la consola;
- la política de autohealing y los límites mínimo/máximo del autoscaler;
- el resultado de la limpieza o la simulación conceptual completa;
- tu puntuación y tu ficha de errores.

---

## 2. Prerrequisitos explicados

### 2.1 Conocimientos previos

- **Proyecto de Google Cloud:** es el contenedor administrativo donde viven los recursos y se aplican facturación, cuotas e IAM.
- **Región y zona:** una región contiene varias zonas. Una zona es un dominio de fallo. Un MIG zonal coloca las VMs en una zona; un MIG regional puede distribuirlas entre varias zonas de la misma región.
- **VM, machine type e imagen:** el tipo de máquina define CPU y memoria; la imagen aporta el sistema operativo inicial.
- **Boot disk:** cada VM necesita un disco de arranque. En un MIG stateless debe tratarse como reemplazable; los datos de negocio no deben depender de ese disco.
- **VPC, subred y firewall:** una interfaz de red conecta la VM a una subred. Las reglas de firewall deciden qué tráfico puede llegar.
- **Cloud Shell y gcloud CLI:** Cloud Shell proporciona una terminal autenticada. <code>gcloud</code> envía solicitudes a las APIs de Google Cloud.

### 2.2 Cuenta, permisos y costo

Para ejecutar la práctica real se necesita:

- un proyecto con facturación habilitada o un proyecto temporal de laboratorio;
- Compute Engine API habilitada;
- permisos para crear redes, reglas de firewall, plantillas, health checks y MIGs;
- cuota para una o, durante un scale-out, dos VMs pequeñas.

No uses el rol Owner como atajo. En un entorno real, solicita únicamente los permisos necesarios mediante roles predefinidos adecuados. La práctica incluye una alternativa conceptual íntegra si no dispones de cuenta, crédito, permisos o cuota.

### 2.3 Seguridad antes de empezar

- Usa un proyecto de laboratorio, no uno de producción.
- No pegues claves, tokens ni secretos en scripts de inicio.
- Los nombres propuestos son genéricos y no corresponden a ningún proyecto privado.
- La regla de firewall de esta práctica acepta tráfico al puerto 80 **solo** desde los rangos de los sistemas de health checks de Google Cloud, no desde todo Internet.
- La base PostgreSQL de SiteOps Tracker se considera externa a las VMs. No se guardan registros de auditoría en el boot disk.

---

## 3. Modelo mental: una flota de inspección

Imagina una empresa que envía vehículos idénticos a inspeccionar sedes:

- La **instance template** es el plano homologado del vehículo: motor, equipo, radio y procedimientos de arranque.
- El **managed instance group** es el coordinador de la flota: mantiene el número deseado de vehículos con el mismo plano y reemplaza los que desaparecen.
- El **autoscaler** es el planificador de turnos: aumenta o reduce el número de vehículos según una señal de demanda.
- El **health check** es la inspección periódica: comprueba si cada vehículo responde correctamente.
- El **autohealing** es la decisión de retirar y sustituir un vehículo que falla repetidamente.
- Un **load balancer** sería el despachador que envía cada nueva solicitud a un vehículo sano. No se crea en esta práctica.

En SiteOps Tracker, cada VM ejecuta una copia intercambiable de una API Node.js. La API recibe solicitudes de activos y hallazgos, pero los datos persistentes viven en PostgreSQL. Si una VM se destruye, el historial no debe perderse.

~~~mermaid
flowchart TD
    T["Instance template<br/>configuración repetible"] --> M["Managed instance group<br/>tamaño deseado"]
    A["Autoscaler<br/>decide cuántas VMs"] --> M
    H["Health check<br/>señal por VM"] --> M
    M --> V1["VM A<br/>API SiteOps"]
    M --> V2["VM B<br/>API SiteOps"]
~~~

---

## 4. Conceptos paso a paso

### 4.1 Instance template: configuración repetible e inmutable

Una **instance template** guarda propiedades para crear VMs de manera uniforme, por ejemplo:

- machine type;
- imagen del sistema operativo;
- tamaño y tipo de boot disk;
- red y subred;
- etiquetas de red;
- service account;
- metadatos y startup scripts;
- opciones de programación y disponibilidad.

La plantilla no es una VM y no consume CPU por sí sola. Tampoco contiene el estado de ejecución de una instancia.

Las plantillas son **inmutables**: si debes cambiar la imagen, el script, el tipo de máquina o una etiqueta, creas una plantilla nueva, por ejemplo <code>sot-api-tpl-v2</code>, y actualizas el MIG. Esto permite saber exactamente qué configuración produce cada versión.

Hay plantillas:

- **globales**, reutilizables entre regiones;
- **regionales**, que reducen dependencias entre regiones y ayudan cuando existen requisitos de residencia o aislamiento regional.

Para SiteOps Tracker, una plantilla podría decir: “crea una VM Debian pequeña, instala el servidor de la API, escucha en el puerto 80, usa una service account mínima y aplica la etiqueta de red para health checks”.

### 4.2 Managed instance group: tratar varias VMs como una unidad

Un **managed instance group (MIG)** crea y administra VMs a partir de una plantilla. Mantiene un tamaño objetivo y reconcilia el estado real:

- si falta una instancia, crea otra;
- si una instancia se elimina, normalmente la reemplaza;
- puede aplicar una nueva plantilla mediante una actualización controlada;
- puede reparar VMs que fallen una comprobación de aplicación;
- puede aumentar o reducir el grupo con autoscaling.

Un **unmanaged instance group** solo agrupa VMs que administras individualmente. No ofrece la misma creación homogénea, actualización declarativa, autoscaling ni autohealing. Para réplicas intercambiables de una API, un MIG es la opción natural.

#### Stateless frente a stateful

La API de SiteOps Tracker debe ser **stateless**:

- una solicitud puede llegar a cualquier réplica;
- las sesiones no dependen de memoria local o se guardan en un servicio compartido;
- activos, hallazgos, responsables e historial viven en PostgreSQL;
- archivos de evidencia viven en almacenamiento persistente apropiado;
- logs se envían a Cloud Logging u otro destino central.

Un stateful MIG puede conservar ciertos discos o direcciones, pero añade complejidad. No es la primera opción para una API horizontalmente escalable.

### 4.3 MIG zonal frente a regional

| Opción | Distribución | Ventaja | Riesgo o costo | Uso razonable |
|---|---|---|---|---|
| MIG zonal | Una zona | Más simple para laboratorio | Una caída zonal afecta al grupo | Práctica, batch tolerante, entorno no crítico |
| MIG regional | Varias zonas de una región | Mejor tolerancia a fallo zonal | Más capacidad mínima y operación más cuidadosa | API de producción que debe seguir disponible |

La práctica usa un **MIG zonal** para limitar recursos. Una propuesta de producción para SiteOps Tracker normalmente evaluaría un MIG regional y al menos capacidad en más de una zona.

### 4.4 Autoscaling: decidir cuántas instancias

El autoscaler modifica el tamaño objetivo del MIG dentro de límites:

- **min replicas:** piso de capacidad y costo;
- **max replicas:** techo que protege cuota y presupuesto;
- **signal:** CPU, capacidad del load balancer, métricas de Cloud Monitoring o un horario;
- **target:** valor que el autoscaler intenta mantener;
- **initialization/cool-down period:** tiempo durante el cual una instancia nueva todavía se está preparando y sus métricas no deben interpretarse como estables;
- **stabilization:** ayuda a evitar una reducción demasiado rápida.

Con una meta de CPU de 0.60, el autoscaler intenta mantener cerca de 60 % la utilización promedio de vCPU del grupo. No significa “crear una VM cada vez que una muestra pasa de 60 %”. La decisión usa métricas, promedios, límites y periodos de estabilización.

En SiteOps Tracker:

- CPU puede ser suficiente si el procesamiento de reportes consume CPU de manera proporcional;
- capacidad del load balancer puede ser mejor si la demanda HTTP es la señal principal;
- una métrica personalizada puede ser mejor si lo determinante es una cola de auditorías pendientes;
- un horario puede anticipar visitas masivas conocidas.

### 4.5 Health check: comprobar la aplicación

Un health check define:

- protocolo, por ejemplo HTTP;
- puerto, por ejemplo 80;
- ruta, por ejemplo <code>/healthz</code>;
- intervalo entre comprobaciones;
- timeout;
- número de éxitos consecutivos para marcar healthy;
- número de fallos consecutivos para marcar unhealthy.

La ruta debe responder correctamente desde la interfaz de red de la VM. Que <code>curl localhost</code> funcione no demuestra que el probe remoto pueda atravesar el firewall ni que la aplicación escuche en la interfaz correcta.

#### Dos usos que no debes confundir

| Uso del health check | Acción principal |
|---|---|
| Load balancing health check | Deja de enviar tráfico a un backend no sano |
| MIG application health check con autohealing | Puede provocar la reparación o recreación de la VM |

Google recomienda que la comprobación de autohealing tolere más fallos que la de balanceo. Un reinicio innecesario es más disruptivo que retirar temporalmente una VM del tráfico. Puedes usar health checks separados con umbrales distintos.

### 4.6 Autohealing no es autoscaling

- **Autoscaling** responde a demanda agregada y cambia el número de VMs.
- **Autohealing** responde a la salud individual y repara una VM.

Una VM puede estar sana con CPU alta: quizá se necesita scale-out, no reemplazarla. Una VM puede tener CPU baja pero devolver HTTP 500: quizá necesita reparación, no scale-in.

### 4.7 Named ports: metadatos, no firewall

Un named port como <code>http:80</code> indica a ciertos load balancers que el servicio llamado “http” usa el puerto 80 en las instancias del grupo.

No hace ninguna de estas cosas:

- no abre el firewall;
- no inicia el servidor;
- no cambia el puerto real de Node.js o NGINX;
- no crea un health check;
- no garantiza que la ruta devuelva HTTP 200.

---

## 5. Decisión para SiteOps Tracker y alternativas

### Requisito supuesto

La API Node.js necesita un paquete de sistema y un agente de auditoría instalado en el SO. Debe mantener réplicas idénticas, escalar con demanda y recuperarse de una aplicación bloqueada. Los datos viven en PostgreSQL fuera de las VMs.

### Decisión

Para ese requisito, usaríamos:

- una instance template versionada;
- un MIG regional en producción;
- un MIG zonal de una VM en el laboratorio;
- autoscaling con límites conservadores;
- un health check HTTP dedicado al autohealing;
- datos y evidencias fuera del disco local;
- una service account dedicada de mínimos privilegios.

### Por qué no elegir automáticamente otra opción

| Alternativa | Cuándo sí elegirla | Por qué no es la opción de este escenario |
|---|---|---|
| VM individual | Servidor único, migración temporal o tarea administrativa | No ofrece réplicas homogéneas, autoscaling ni reparación coordinada |
| Unmanaged instance group | VMs heterogéneas que debes conservar y administrar una por una | No proporciona el modelo declarativo de una flota idéntica |
| Cloud Run | Contenedor HTTP stateless sin necesidad de administrar SO | Sería más simple para una API contenedorizada, pero no satisface el requisito supuesto de agente y control del sistema operativo |
| GKE Autopilot | Varios workloads Kubernetes y necesidad real de su API/ecosistema | Añade conceptos y operación de Kubernetes innecesarios para una sola API |
| Stateful MIG | Instancias que deben retener discos o identidad específica | SiteOps Tracker guarda su estado de negocio en PostgreSQL; acoplarlo a VMs dificulta reemplazo y escalado |
| MIG zonal en producción | Carga tolerante a la caída total de una zona | No protege la API frente a un fallo zonal |

La elección depende del requisito, no de que un servicio sea “más avanzado”.

---

## 6. Práctica guiada verificada: plantilla + MIG + autoscaling + health check

Los comandos y opciones de esta sección fueron contrastados con la documentación oficial de Google Cloud disponible el **19 de octubre de 2026**. Las etiquetas de la consola pueden cambiar ligeramente. Ejecuta la ruta CLI y usa la consola para inspeccionar; no crees una segunda copia de los mismos recursos.

### 6.1 Arquitectura de laboratorio

- Red VPC custom con una subred.
- Plantilla global basada en Debian 13 y una VM <code>e2-micro</code>.
- Startup script que instala NGINX y publica <code>/healthz</code>.
- MIG zonal con una instancia inicial.
- Health check HTTP en puerto 80.
- Autohealing con espera inicial.
- Autoscaling por CPU: mínimo 1 y máximo 2.
- Sin load balancer.

> **Importante:** un MIG puede llegar a dos VMs si la señal de CPU lo justifica. No generes carga artificial si deseas minimizar costo.

### 6.2 Preparación en Cloud Shell

Abre **Google Cloud Console → Activate Cloud Shell**. Define nombres:

~~~bash
PROJECT_ID="$(gcloud config get-value project)"
REGION="us-central1"
ZONE="us-central1-a"

NETWORK="ace-l21-vpc"
SUBNET="ace-l21-subnet"
TEMPLATE="ace-l21-sot-tpl-v1"
MIG="ace-l21-sot-mig"
HEALTH_CHECK="ace-l21-sot-hc"
FIREWALL="ace-l21-allow-hc"

gcloud auth list
gcloud config get-value project
gcloud services enable compute.googleapis.com --project="$PROJECT_ID"
~~~

**Comprobación:** <code>PROJECT_ID</code> no debe quedar vacío ni contener el marcador <code>(unset)</code>. Si es incorrecto, selecciona explícitamente el proyecto de laboratorio antes de continuar.

### 6.3 Crear la red de laboratorio

~~~bash
gcloud compute networks create "$NETWORK" \
  --project="$PROJECT_ID" \
  --subnet-mode=custom

gcloud compute networks subnets create "$SUBNET" \
  --project="$PROJECT_ID" \
  --network="$NETWORK" \
  --region="$REGION" \
  --range=10.21.0.0/24
~~~

La VPC custom evita depender de que exista la red <code>default</code>. La regla implícita de egress permite que una VM con IPv4 externa descargue paquetes; no se crea una regla pública de ingress.

### 6.4 Preparar el startup script

Este servidor representa únicamente el endpoint de salud de la API de SiteOps Tracker:

~~~bash
cat > /tmp/sot-startup.sh <<'STARTUP'
#!/bin/bash
set -euxo pipefail
apt-get update
apt-get install -y nginx
printf 'healthy\n' > /var/www/html/healthz
printf 'SiteOps Tracker API simulation\n' > /var/www/html/index.html
systemctl enable --now nginx
STARTUP
~~~

En producción, la imagen o el proceso de despliegue debería ser determinista y versionado. Un startup script que instala “lo último” en cada arranque puede producir resultados diferentes con el tiempo.

### 6.5 Crear la instance template

~~~bash
gcloud compute instance-templates create "$TEMPLATE" \
  --project="$PROJECT_ID" \
  --machine-type=e2-micro \
  --image-family=debian-13 \
  --image-project=debian-cloud \
  --boot-disk-type=pd-balanced \
  --boot-disk-size=10GB \
  --network="$NETWORK" \
  --subnet="$SUBNET" \
  --region="$REGION" \
  --tags=sot-health-check \
  --metadata-from-file=startup-script=/tmp/sot-startup.sh
~~~

Observa que:

- la plantilla es global porque no se indicó <code>--instance-template-region</code>;
- <code>--region</code> identifica la región de la subred usada por la interfaz;
- la etiqueta <code>sot-health-check</code> será el destino lógico de la regla de firewall;
- la plantilla define la VM, pero todavía no ejecuta ninguna.

Verifica:

~~~bash
gcloud compute instance-templates describe "$TEMPLATE" \
  --project="$PROJECT_ID" \
  --format="yaml(name,properties.machineType,properties.tags.items,properties.disks)"
~~~

### 6.6 Permitir exclusivamente los probes de health check

Los probes IPv4 relevantes para esta práctica proceden de <code>130.211.0.0/22</code> y <code>35.191.0.0/16</code>.

~~~bash
gcloud compute firewall-rules create "$FIREWALL" \
  --project="$PROJECT_ID" \
  --network="$NETWORK" \
  --direction=INGRESS \
  --action=ALLOW \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --target-tags=sot-health-check \
  --allow=tcp:80
~~~

No sustituyas los rangos por <code>0.0.0.0/0</code>. La etiqueta de la plantilla y el target tag de la regla deben coincidir exactamente.

### 6.7 Crear el MIG zonal

~~~bash
gcloud compute instance-groups managed create "$MIG" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --template="$TEMPLATE" \
  --size=1 \
  --base-instance-name=sot-api
~~~

Configura un named port útil para un futuro backend service:

~~~bash
gcloud compute instance-groups managed set-named-ports "$MIG" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --named-ports=http:80
~~~

Recuerda: el named port es metadato del grupo. El startup script inicia NGINX y la regla de firewall permite el probe; son responsabilidades distintas.

### 6.8 Crear el health check HTTP

~~~bash
gcloud compute health-checks create http "$HEALTH_CHECK" \
  --project="$PROJECT_ID" \
  --global \
  --port=80 \
  --request-path=/healthz \
  --check-interval=30s \
  --timeout=10s \
  --healthy-threshold=1 \
  --unhealthy-threshold=3
~~~

Estos umbrales toleran tres fallos consecutivos antes de declarar la VM no sana. Para producción debes ajustar tiempos según el arranque y el comportamiento real de la aplicación.

### 6.9 Asociar autohealing

~~~bash
gcloud compute instance-groups managed update "$MIG" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --health-check="$HEALTH_CHECK" \
  --initial-delay=180
~~~

Los 180 segundos dan tiempo para que Debian instale e inicie NGINX. Si el delay fuese menor que el arranque real, la VM podría entrar en un ciclo de reparación.

### 6.10 Activar autoscaling por CPU

~~~bash
gcloud compute instance-groups managed set-autoscaling "$MIG" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --min-num-replicas=1 \
  --max-num-replicas=2 \
  --target-cpu-utilization=0.60 \
  --cool-down-period=90
~~~

El límite máximo de dos controla la exposición de costo. No esperes un scale-out si la CPU permanece por debajo del objetivo.

### 6.11 Verificar por CLI

Espera varios minutos y ejecuta:

~~~bash
gcloud compute instance-groups managed describe "$MIG" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --format="yaml(name,targetSize,instanceTemplate,autoHealingPolicies,namedPorts)"

gcloud compute instance-groups managed list-instances "$MIG" \
  --project="$PROJECT_ID" \
  --zone="$ZONE"

gcloud compute health-checks describe "$HEALTH_CHECK" \
  --project="$PROJECT_ID" \
  --global \
  --format="yaml(name,type,httpHealthCheck,checkIntervalSec,timeoutSec,healthyThreshold,unhealthyThreshold)"
~~~

**Resultado esperado:**

- existe una plantilla global;
- el MIG tiene tamaño objetivo inicial de 1;
- la instancia termina en <code>RUNNING</code>;
- después del arranque y los probes, la salud debe llegar a <code>HEALTHY</code>;
- la política de autohealing muestra el health check y 180 segundos de initial delay;
- el named port muestra <code>http:80</code>;
- el autoscaler permite entre 1 y 2 instancias con meta de CPU 0.60.

Los tiempos y el formato exacto de la salida pueden variar. No concluyas que existe un error mientras la VM siga en inicialización.

### 6.12 Inspeccionar en la consola

Sin crear recursos adicionales:

1. Abre **Compute Engine → Instance groups**.
2. Selecciona <code>ace-l21-sot-mig</code>.
3. En **Overview/Instances**, revisa estado, acción actual y health check status.
4. En **Details**, identifica:
   - instance template;
   - autoscaling y sus límites;
   - autohealing y su initial delay;
   - named ports.
5. Abre **Compute Engine → Instance templates** y comprueba machine type, imagen, red, tags y metadata.
6. Abre **Network services → Health checks** y verifica puerto, ruta y umbrales.
7. Abre **VPC network → Firewall** y confirma origen, target tag y puerto.

### 6.13 Registro de evidencia

Completa:

| Evidencia | Tu resultado |
|---|---|
| Proyecto de laboratorio | |
| Plantilla creada o simulada | |
| MIG y alcance zonal/regional | |
| Tamaño actual | |
| Estado de VM | |
| Health state | |
| Min/max y señal de autoscaling | |
| Firewall: origen, target y puerto | |
| Tiempo invertido | |
| Duda principal | |

---

## 7. Alternativa conceptual íntegra sin cuenta, crédito o permisos

No necesitas ejecutar recursos para completar el objetivo conceptual. Haz esta simulación en papel o en una copia del Markdown.

### 7.1 Configuración simulada

| Elemento | Valor |
|---|---|
| Template | <code>sot-api-tpl-v1</code>, Debian 13, e2-micro, 10 GB, tag <code>sot-health-check</code> |
| MIG | Regional, tres zonas, min 2, max 6 |
| Aplicación | API Node.js stateless; datos en PostgreSQL externo |
| Health check | HTTP, puerto 8080, ruta <code>/healthz</code>, timeout 5 s, unhealthy threshold 3 |
| Initial delay | 120 s |
| Autoscaling | CPU objetivo 60 % |
| Named port | <code>http:8080</code> |

### 7.2 Predice antes de leer la comprobación

1. CPU promedio 25 % durante un periodo estable, tamaño actual 4: ¿qué tendencia esperas?
2. CPU promedio 85 % con tamaño 2: ¿qué tendencia esperas?
3. Una VM falla tres probes consecutivos, pero las demás están sanas: ¿qué componente actúa y sobre qué unidad?
4. Todas las VMs sirven <code>curl localhost:8080/healthz</code>, pero aparecen <code>UNHEALTHY</code>: ¿qué revisarías primero?
5. Se necesita cambiar el puerto del startup script: ¿se edita la plantilla v1?
6. Una zona completa deja de estar disponible: ¿por qué el MIG regional es relevante?

### 7.3 Comprobación de la simulación

1. El autoscaler tenderá a reducir instancias, nunca por debajo de 2 y respetando estabilización.
2. Tenderá a añadir instancias, nunca por encima de 6; la reacción no es instantánea.
3. El health check alimenta la política de autohealing; el MIG repara o recrea esa VM. No es un scale-out por demanda.
4. Revisa firewall desde los rangos de probe, target tag, puerto/ruta, escucha en <code>0.0.0.0</code> y tiempo de inicialización.
5. No. Crea <code>sot-api-tpl-v2</code> y actualiza el MIG de manera controlada.
6. Puede mantener réplicas en otras zonas de la región, sujeto a la distribución y capacidad configuradas.

Registra como evidencia tu diagrama, las seis predicciones y las correcciones.

---

## 8. Solución de problemas y refuerzo de errores observados

### 8.1 El MIG o el backend aparece UNHEALTHY

Sigue el diagnóstico en este orden:

1. **Proceso:** ¿NGINX o la API están ejecutándose?
2. **Escucha:** ¿el proceso escucha en el puerto correcto y en una interfaz accesible, no solo en <code>127.0.0.1</code>?
3. **Ruta:** ¿<code>/healthz</code> existe y devuelve un estado HTTP exitoso con rapidez?
4. **Health check:** ¿protocolo, puerto, ruta, scope y umbrales coinciden?
5. **Firewall:** ¿permite TCP al puerto desde <code>130.211.0.0/22</code> y <code>35.191.0.0/16</code>?
6. **Destino de firewall:** ¿el target tag coincide con el tag de la VM creada desde la plantilla?
7. **Named port:** si existe un backend service que usa un named port, ¿el nombre y puerto coinciden? Recuerda que esto no sustituye firewall ni proceso.
8. **Initial delay:** ¿la aplicación tarda más en arrancar que el tiempo concedido?
9. **Startup script:** ¿falló la descarga o instalación? Revisa serial output y logs de startup.
10. **Dependencias:** ¿el endpoint declara unhealthy porque PostgreSQL u otra dependencia temporal no responde? Evita que una dependencia remota inestable provoque una tormenta de recreaciones.

Para ver la salida serial de una instancia concreta:

~~~bash
gcloud compute instances get-serial-port-output INSTANCE_NAME \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --port=1
~~~

Sustituye <code>INSTANCE_NAME</code> por el nombre mostrado en <code>list-instances</code>.

### 8.2 SSH presenta timeout

Un timeout SSH y un health check fallido pueden coexistir, pero son rutas distintas:

- SSH normalmente usa TCP 22; el health check de esta práctica usa TCP 80.
- La regla de health check no concede SSH.
- OS Login, IAM, IAP, una IP externa y las reglas de firewall de SSH tienen requisitos propios.

No uses “no puedo entrar por SSH” como única prueba de que la aplicación está caída. Consulta primero el estado del MIG, la salida serial, los logs del startup script y la configuración de red. Tampoco abras TCP 22 a todo Internet solo para acelerar el diagnóstico.

### 8.3 Necesitas reconstruir una VM concreta del MIG

No la trates como una VM independiente ni cambies manualmente su configuración. Si la plantilla y la configuración actual del MIG ya son correctas, solicita al propio MIG que recree esa instancia:

~~~bash
gcloud compute instance-groups managed recreate-instances "$MIG" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --instances=INSTANCE_NAME
~~~

La VM subyacente se elimina y se crea de nuevo con la configuración vigente del grupo. Verifica después el estado con <code>list-instances</code>. Si todas las VMs necesitan la configuración nueva, usa una actualización gradual; recrearlas una por una no sustituye un rollout.

### 8.4 La VM entra en un ciclo de recreación

Posibles causas:

- initial delay demasiado corto;
- health check demasiado agresivo;
- error determinista dentro de la plantilla o startup script;
- ruta de salud dependiente de un servicio externo intermitente;
- firewall que bloquea todos los probes.

Para detener temporalmente las reparaciones mientras diagnosticas, puedes quitar la política de autohealing:

~~~bash
gcloud compute instance-groups managed update "$MIG" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --clear-autohealing
~~~

Esto no corrige la causa. Vuelve a asociar el health check después de reparar plantilla, aplicación o red.

### 8.5 El autoscaler no aumenta el grupo

Comprueba:

- la CPU promedio realmente supera el objetivo;
- el autoscaler está activo;
- <code>max-num-replicas</code> es mayor que el tamaño actual;
- hay cuota y capacidad en la zona;
- las nuevas VMs siguen dentro del periodo de inicialización;
- no estás interpretando una muestra breve como una tendencia.

### 8.6 “Quiero editar la plantilla”

No se modifica una plantilla existente. Crea una v2 y actualiza el MIG. En producción, elige una estrategia de actualización con límites de surge/unavailable, valida salud y conserva una ruta de rollback.

### 8.7 Permiso denegado o recurso no encontrado

- confirma proyecto, zona y región en cada comando;
- comprueba que la plantilla sea global o usa su URL regional correcta;
- no confíes únicamente en defaults de <code>gcloud</code>;
- solicita el permiso exacto que falta;
- revisa que Compute Engine API esté habilitada.

---

## 9. Costos, cuotas y limpieza

### 9.1 Qué puede generar costo

- VMs del MIG mientras estén ejecutándose;
- boot disks de cada VM;
- direcciones IPv4 externas;
- tráfico de red y salida a Internet;
- Cloud Logging si se supera la asignación sin costo;
- un load balancer si decidieras añadirlo, aunque esta práctica no lo crea.

La plantilla no ejecuta cómputo por sí sola. El autoscaler puede elevar el grupo hasta dos VMs, por lo que el límite máximo es también un control de costo. Un presupuesto genera alertas; no detiene automáticamente el consumo.

Los precios y los beneficios gratuitos cambian. Consulta la calculadora y las páginas oficiales antes de estimar dinero; no asumas que <code>e2-micro</code>, el disco o la IPv4 serán gratuitos en tu región y cuenta.

### 9.2 Limpieza verificada

Hazla en este orden. El primer comando detiene cambios automáticos de tamaño; el segundo desacopla el health check:

~~~bash
gcloud compute instance-groups managed stop-autoscaling "$MIG" \
  --project="$PROJECT_ID" \
  --zone="$ZONE"

gcloud compute instance-groups managed update "$MIG" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --clear-autohealing

gcloud compute instance-groups managed delete "$MIG" \
  --project="$PROJECT_ID" \
  --zone="$ZONE" \
  --quiet

gcloud compute health-checks delete "$HEALTH_CHECK" \
  --project="$PROJECT_ID" \
  --global \
  --quiet

gcloud compute firewall-rules delete "$FIREWALL" \
  --project="$PROJECT_ID" \
  --quiet

gcloud compute instance-templates delete "$TEMPLATE" \
  --project="$PROJECT_ID" \
  --quiet

gcloud compute networks subnets delete "$SUBNET" \
  --project="$PROJECT_ID" \
  --region="$REGION" \
  --quiet

gcloud compute networks delete "$NETWORK" \
  --project="$PROJECT_ID" \
  --quiet

rm -f /tmp/sot-startup.sh
~~~

Eliminar el MIG elimina sus VMs administradas. Antes de hacerlo en cualquier entorno real, verifica políticas de discos, datos y backups; una carga stateful exige un plan distinto.

### 9.3 Comprobación final

~~~bash
gcloud compute instance-groups managed list \
  --project="$PROJECT_ID" \
  --filter="name=$MIG"

gcloud compute instance-templates list \
  --project="$PROJECT_ID" \
  --filter="name=$TEMPLATE"

gcloud compute health-checks list \
  --project="$PROJECT_ID" \
  --filter="name=$HEALTH_CHECK"

gcloud compute networks list \
  --project="$PROJECT_ID" \
  --filter="name=$NETWORK"
~~~

Las cuatro consultas deben quedar sin coincidencias para los nombres del laboratorio.

---

## 10. Glosario bilingüe

| English term | Español y significado práctico |
|---|---|
| Instance template | Plantilla de instancia; definición inmutable para crear VMs uniformes |
| Managed instance group (MIG) | Grupo administrado de instancias; flota de VMs reconciliada por Compute Engine |
| Unmanaged instance group | Grupo no administrado; conjunto de VMs administradas individualmente |
| Stateless workload | Carga sin estado local imprescindible; una réplica puede sustituirse |
| Stateful workload | Carga que conserva datos o identidad ligados a una instancia |
| Zonal MIG | MIG cuyas VMs pertenecen a una zona |
| Regional MIG | MIG que puede distribuir VMs entre zonas de una región |
| Autoscaling | Ajuste automático del número de instancias |
| Scale out / scale in | Añadir / retirar capacidad horizontal |
| Target utilization | Utilización objetivo que guía el autoscaler |
| Initialization period | Tiempo concedido a una VM nueva para prepararse |
| Health check | Comprobación periódica de la capacidad de respuesta |
| Healthy / unhealthy | Sano / no sano según los umbrales del health check |
| Autohealing | Reparación automática de una VM no sana |
| Named port | Metadato nombre:puerto del servicio en un instance group |
| Startup script | Script ejecutado al arrancar una VM |
| Reconciliation | Acción de llevar el estado real al estado deseado |
| Rolling update | Sustitución gradual de VMs por una configuración nueva |
| Probe | Solicitud de sondeo enviada por el sistema de health checks |
| Backend | Recurso que atiende tráfico detrás de un balanceador |

---

## 11. Repaso activo espaciado

Responde sin mirar lecciones anteriores. Luego compara con la clave al final.

### 11.1 Día hábil anterior — L20: discos de VM

1. SiteOps Tracker necesita un disco de arranque para cada réplica stateless del MIG. ¿Qué propiedad evita conservarlo como fuente única de verdad?
2. Una base de datos autogestionada requiere un disco accesible tras fallo zonal. ¿Qué diferencia conceptual hay entre Persistent Disk zonal y regional?
3. ¿Por qué no eliges Hyperdisk solo porque es “más nuevo”?

### 11.2 Otro día hábil previo — L19: VMs, disponibilidad y SSH

1. ¿Qué tres restricciones del workload usas para seleccionar una machine family o machine type?
2. ¿Qué configuración de disponibilidad importa si una VM debe reiniciarse o migrarse durante mantenimiento?
3. ¿Por qué un timeout SSH no demuestra por sí solo que la aplicación esté unhealthy?

### 11.3 Hace 7 días — L16: repaso del dominio 1

1. ¿En qué proyecto y cuenta de facturación se crearán los recursos de laboratorio?
2. ¿Qué diferencia existe entre un budget alert y un límite duro de gasto?
3. ¿Qué revisas antes de elegir región: latencia, regulación, disponibilidad de producto o todas?

### 11.4 Hace 21 días — L6: jerarquía de recursos

1. Ordena: recurso, organización, proyecto y carpeta.
2. ¿En qué nivel aplicarías una política común para todos los proyectos de producción?
3. ¿Por qué separar desarrollo y producción en proyectos distintos ayuda a SiteOps Tracker?

---

## 12. Preguntas originales de examen

Responde en inglés. No mires las soluciones hasta terminar. Salvo que se indique **Choose two**, hay una sola respuesta correcta.

### Question 1

SiteOps Tracker runs a stateless Node.js API on Compute Engine. The team needs identical VMs, automatic replacement of failed application instances, and horizontal scaling. Which design best meets the requirements?

A. Create several unrelated VMs from the same public image and manage them manually.  
B. Create an instance template and use it with an autoscaled managed instance group and an application health check.  
C. Place existing VMs in an unmanaged instance group and enable a budget alert.  
D. Store all application data on each VM boot disk and take occasional snapshots.

### Question 2

A team must change the startup script and machine type used by every VM in a MIG. What should the team do?

A. Edit the existing instance template in place.  
B. Stop each VM, edit it manually, and restart it.  
C. Create a new instance template and roll the MIG to the new template.  
D. Change the named port because named ports control VM configuration.

### Question 3

An HTTP health check uses port 8080 and path <code>/healthz</code>. The application returns HTTP 200 when tested locally on every VM, but all instances are marked unhealthy. What should you check first?

A. Whether ingress from the Google Cloud health-check probe ranges is allowed to port 8080 for the VM targets.  
B. Whether the autoscaler's maximum replica count is high enough.  
C. Whether a billing budget has reached 100 percent.  
D. Whether the boot disks use Hyperdisk instead of Persistent Disk.

### Question 4

A MIG has CPU autoscaling with a target of 60%, a minimum of 2 instances, and a maximum of 8. Average CPU remains at 85% after the initialization period. What is the expected behavior?

A. Autohealing recreates every VM because CPU is above the target.  
B. The autoscaler tends to add instances, up to the configured maximum.  
C. The group immediately jumps to exactly 8 instances after one metric sample.  
D. The autoscaler reduces the group to the minimum.

### Question 5

SiteOps Tracker must remain available if one zone in a region fails. Which option is most appropriate?

A. A single VM with a larger machine type.  
B. A zonal MIG with one instance.  
C. A regional MIG with capacity distributed across multiple zones.  
D. An unmanaged instance group in one zone.

### Question 6

A load balancer should stop sending traffic quickly when a backend fails, but the team wants to avoid recreating VMs during brief application pauses. What is the best approach?

A. Use one very aggressive health check for both load balancing and autohealing.  
B. Disable all health checks and rely on SSH.  
C. Use separate health checks, with a more conservative policy for autohealing.  
D. Set the named port to <code>healthy:1</code>.

### Question 7

What does setting the named port <code>http:80</code> on a MIG accomplish?

A. It opens TCP port 80 in the VPC firewall.  
B. It starts the web server on every VM.  
C. It provides service-to-port metadata used by supported load balancers.  
D. It creates an HTTP health check at path <code>/</code>.

### Question 8 — Choose two

Which two practices make a Node.js API suitable for a stateless MIG?

A. Store audit history only on each VM's boot disk.  
B. Store persistent records in PostgreSQL outside the VM replicas.  
C. Make every request require the same specific VM.  
D. Send logs to a centralized service instead of relying only on local files.  
E. Give every VM a unique manually maintained startup configuration.

### Question 9

VM startup normally takes 150 seconds, but the MIG autohealing initial delay is 30 seconds. Instances repeatedly become unhealthy and are recreated. What is the best first correction?

A. Reduce the delay to zero so probes start immediately.  
B. Increase the initial delay to reflect real application initialization time and verify the startup process.  
C. Increase the CPU target to 100%.  
D. Convert the MIG to an unmanaged instance group.

### Question 10

The SiteOps Tracker VM needs to call one Google Cloud API with read-only access. The current template uses a broadly privileged default service account. What should the team do?

A. Grant Owner to the default service account so all calls succeed.  
B. Embed a long-lived service account key in the startup script.  
C. Create a dedicated least-privileged service account, create a new template that attaches it, and update the MIG.  
D. Remove the health check because health checks control IAM.

---

## 13. Ficha de errores

No registres solo “me equivoqué”. Convierte cada fallo en una regla reutilizable.

| Pregunta o paso | Mi respuesta/acción | Respuesta correcta o resultado | Causa del error | Señal decisiva del escenario | Regla para la próxima vez | Reintento D+1 / D+7 / D+21 |
|---|---|---|---|---|---|---|
| | | | | | | |
| | | | | | | |
| | | | | | | |
| | | | | | | |

### Refuerzo breve recomendado

Si tu error fue “backend UNHEALTHY”, formula de memoria esta cadena:

**proceso → interfaz/puerto → ruta/respuesta → firewall/origen → target tag → umbrales/delay → logs**.

Si tu error fue “SSH timeout”, separa las rutas:

**identidad/IAM/OS Login + red SSH** no es lo mismo que **salud HTTP de la aplicación**.

---

## 14. Criterios de autoevaluación

Marca únicamente lo que hayas comprobado:

- [ ] Puedo explicar plantilla, MIG, autoscaler, health check y autohealing sin leer.
- [ ] Distingo autoscaling de autohealing con un ejemplo.
- [ ] Sé por qué una plantilla no se edita y cómo introducir una v2.
- [ ] Puedo justificar zonal para laboratorio y regional para disponibilidad multizona.
- [ ] Configuré o simulé todos los elementos de la práctica.
- [ ] Puedo diagnosticar <code>UNHEALTHY</code> en un orden reproducible.
- [ ] Entiendo que named port, firewall, proceso y health check son configuraciones separadas.
- [ ] Eliminé los recursos o completé la alternativa conceptual.
- [ ] Obtuve al menos 8/10 en las preguntas y expliqué los fallos.

Interpretación sugerida:

- **9–10 correctas:** buen desempeño en esta práctica; conserva los repasos espaciados.
- **8 correctas:** objetivo de sesión razonablemente cubierto; revisa los distractores fallados.
- **0–7 correctas:** repite el modelo mental, la tabla de decisiones y el diagnóstico antes de avanzar.

La puntuación es una señal de estudio, no una afirmación de dominio ni un umbral oficial del examen.

---

## 15. Documentación oficial consultada

Comandos y conceptos verificados el 19 de octubre de 2026:

- [Create instance templates](https://cloud.google.com/compute/docs/instance-templates/create-instance-templates)
- [Instance templates overview](https://cloud.google.com/compute/docs/instance-templates)
- [Create a zonal managed instance group](https://cloud.google.com/compute/docs/instance-groups/create-zonal-mig)
- [Managed instance groups overview](https://cloud.google.com/compute/docs/instance-groups)
- [Set up application-based health checks and autohealing](https://cloud.google.com/compute/docs/instance-groups/autohealing-instances-in-migs)
- [Autoscaling groups of instances](https://cloud.google.com/compute/docs/autoscaler)
- [Scaling based on CPU utilization](https://cloud.google.com/compute/docs/autoscaler/scaling-cpu)
- [Use health checks and configure probe firewall rules](https://cloud.google.com/load-balancing/docs/health-checks)
- [View information about MIGs and managed instances](https://cloud.google.com/compute/docs/instance-groups/getting-info-about-migs)
- [Recreate selected instances in a MIG](https://cloud.google.com/compute/docs/instance-groups/working-with-managed-instances#recreating_instances)
- [Rolling updates for MIGs](https://cloud.google.com/compute/docs/instance-groups/rolling-out-updates-to-managed-instance-groups)
- [gcloud: create an instance template](https://cloud.google.com/sdk/gcloud/reference/compute/instance-templates/create)
- [gcloud: create an HTTP health check](https://cloud.google.com/sdk/gcloud/reference/compute/health-checks/create/http)
- [gcloud: set named ports](https://cloud.google.com/sdk/gcloud/reference/compute/instance-groups/managed/set-named-ports)
- [Compute Engine pricing](https://cloud.google.com/compute/vm-instance-pricing)
- Guía oficial adjunta: <code>associate_cloud_engineer_exam_guide_english.pdf</code>, sección 2.1.

---

## 16. Soluciones justificadas — sección final

### Solution 1 — B

**B is correct.** The template produces consistent VMs, the MIG manages desired state, autoscaling adjusts capacity, and an application health check can drive autohealing.

- **A** creates configuration drift and requires manual recovery.
- **C** does not give an unmanaged group MIG-style reconciliation, autoscaling, or autohealing; a budget is unrelated.
- **D** makes replaceable VMs hold irreplaceable state, which conflicts with horizontal scaling.

### Solution 2 — C

**C is correct.** Instance templates are immutable. Create a new version and update the MIG with a controlled rollout.

- **A** is impossible because an existing template cannot be edited in place.
- **B** introduces drift; the MIG can later recreate the manually changed instances from the old template.
- **D** changes only service-to-port metadata and does not alter machine type or startup logic.

### Solution 3 — A

**A is correct.** Local success proves the process answers locally, but probes still need a network path. Verify the firewall permits the documented health-check source ranges, the correct port, and the intended targets.

- **B** affects capacity, not probe reachability.
- **C** sends cost notifications and does not change health.
- **D** is unrelated to an HTTP connection failure.

Also verify that the service listens on a reachable interface, the path and protocol match, and the target tag is present.

### Solution 4 — B

**B is correct.** Sustained average CPU above the target causes the autoscaler to tend toward scale-out, subject to initialization, stabilization, quota, and the maximum.

- **A** confuses demand with health; high CPU is not automatically an application failure.
- **C** assumes an immediate deterministic jump after one sample, which is not how autoscaler decisions work.
- **D** is the opposite of the expected direction.

### Solution 5 — C

**C is correct.** A regional MIG can distribute managed instances across multiple zones in one region and is the appropriate choice for zonal failure tolerance.

- **A** remains a single-instance and single-zone failure point.
- **B** and **D** remain confined to one zone.
- An unmanaged group also lacks the desired managed lifecycle.

The application and its dependencies must also be designed for multizone availability; the MIG alone does not make PostgreSQL or other components resilient.

### Solution 6 — C

**C is correct.** Load balancing and autohealing have different consequences. A fast load-balancing check can remove a backend from traffic, while a more tolerant autohealing check avoids destructive recreation during a brief pause.

- **A** risks unnecessary repair loops.
- **B** removes automated traffic protection and makes SSH a poor health signal.
- **D** misunderstands named ports; they do not define health behavior.

### Solution 7 — C

**C is correct.** A named port is service-to-port metadata used by supported Application Load Balancers and proxy Network Load Balancers.

- **A** requires a separate firewall rule.
- **B** requires application or startup configuration.
- **D** requires a separate health-check resource and association.

### Solution 8 — B and D

**B and D are correct.** External PostgreSQL storage keeps business records independent of VM replacement, and centralized logs preserve operational evidence across instance lifecycles.

- **A** loses authoritative history when a boot disk is deleted.
- **C** creates session affinity or state coupling that prevents interchangeable replicas.
- **E** introduces configuration drift; a template should provide repeatable configuration.

### Solution 9 — B

**B is correct.** The initial delay must reflect real startup time, and the startup sequence should be verified. Otherwise probes can condemn a VM before the service is ready.

- **A** makes premature failure more likely.
- **C** changes a scaling target, not autohealing timing.
- **D** discards useful management features instead of fixing the configuration.

### Solution 10 — C

**C is correct.** A dedicated service account with only the required read permission follows least privilege. Because the template is immutable, create a new template and update the MIG.

- **A** grants excessive access.
- **B** creates a long-lived secret that can leak through metadata, logs, or the template.
- **D** confuses application health with IAM.

### Clave breve del repaso activo

**L20:** (1) La instancia es reemplazable y el estado autorizado vive fuera del boot disk; configura correctamente auto-delete y backups según el caso. (2) Un disco zonal depende de una zona; uno regional replica sincrónicamente entre dos zonas de una región y se elige por requisito de recuperación, no por costumbre. (3) Hyperdisk se elige por requisitos concretos de rendimiento, capacidad y compatibilidad; puede ser innecesario o más costoso.

**L19:** (1) CPU/memoria, arquitectura o aceleradores y patrón de costo/rendimiento. (2) On-host maintenance, automatic restart, provisioning model y tolerancia a interrupción. (3) SSH usa identidad y una ruta de red distinta a la comprobación HTTP de la aplicación.

**L16:** (1) Usa el proyecto de laboratorio correcto y confirma su vínculo de facturación. (2) Un budget alerta; no impone por sí mismo un corte duro. (3) Todas: latencia, regulación, disponibilidad de servicio, precio y resiliencia influyen.

**L6:** (1) Organización → carpeta → proyecto → recurso. (2) En una carpeta de producción o un ancestro común apropiado, respetando herencia y excepciones justificadas. (3) Separa IAM, cuotas, facturación, ciclo de vida y radio de impacto.
