# ACE 2026 · Lección 18

## Elegir Compute Engine, GKE, Cloud Run, functions o Agent Runtime

**Fecha:** 14 de octubre de 2026  
**Guía oficial ACE:** 2.1 *Planning and implementing compute resources*  
**Práctica del plan:** construir una tabla de decisión según operación, eventos y control requerido  
**Caso transversal:** SiteOps Tracker, proyecto ficticio para auditar infraestructura en sedes, registrar activos, hallazgos, estados, responsables e historial  
**Stack del caso:** React + TypeScript, Node.js + TypeScript, PostgreSQL y servicios de Google Cloud  
**Duración sugerida:** 75-100 minutos

> Recibir esta lección no demuestra dominio. La evidencia será poder elegir un servicio a partir de requisitos, explicar por qué y descartar alternativas con precisión.

---

## Objetivo

Al terminar podrás:

1. Distinguir el nivel de abstracción y la carga operativa de Compute Engine, GKE, Cloud Run, Cloud Run functions y Agent Runtime.
2. Elegir una opción a partir de cinco señales: unidad de despliegue, patrón de invocación, control requerido, orquestación y especialización del workload.
3. Evitar decisiones basadas solo en popularidad, lenguaje o “serverless”.
4. Construir y defender una tabla de decisión para seis workloads de SiteOps Tracker.
5. Reconocer en preguntas de examen palabras que apuntan a VM, Kubernetes, contenedor administrado, función orientada a eventos o agente de IA.

## Prerrequisitos explicados desde cero

### 1. Workload

Un **workload** es el conjunto de código y procesos que realiza una función de negocio. En SiteOps Tracker, la API Node.js, un importador de evidencias y un agente que resume hallazgos serían workloads diferentes.

### 2. Máquina virtual

Una **virtual machine (VM)** presenta CPU, memoria, disco y sistema operativo. Tú eliges la imagen, instalas software, aplicas parches y decides cómo escalar. Esa libertad aumenta el trabajo operativo.

### 3. Contenedor

Un **container** empaqueta una aplicación y sus dependencias. Comparte el kernel del host y suele ser más pequeño que una VM. Una imagen de contenedor no decide por sí sola dónde ejecutarse: puede correr en GKE, Cloud Run, una VM u otra plataforma.

### 4. Kubernetes

**Kubernetes** orquesta contenedores mediante objetos declarativos como Deployments, Pods y Services. GKE es el servicio administrado de Kubernetes de Google Cloud. “Usamos contenedores” no significa automáticamente “necesitamos Kubernetes”.

### 5. Serverless

En este contexto, **serverless** significa que Google administra la infraestructura subyacente y el escalado de la plataforma. Los servidores siguen existiendo; tu equipo deja de administrarlos directamente. Cloud Run y Cloud Run functions son ejemplos.

### 6. Evento

Un **event** describe algo que ocurrió: se creó un objeto en Cloud Storage, llegó un mensaje a Pub/Sub o cambió un recurso. Eventarc puede enrutar eventos hacia destinos compatibles, incluidas aplicaciones en Cloud Run.

### 7. Agente de IA

Un **AI agent** no es solo una API que llama a un modelo. Es una aplicación agentic capaz de razonar sobre una tarea, usar herramientas, mantener contexto o sesiones y ejecutar pasos. **Agent Runtime** en Gemini Enterprise Agent Platform está especializado en desplegar, operar y escalar este tipo de aplicación.

---

## Repaso activo espaciado

Responde primero sin mirar las pistas.

### Día hábil anterior · 13/10 · Lección 17

1. ¿Qué dos grupos obtuvieron menor puntuación en tu control del dominio 1?
2. ¿Qué dos tareas de recuperación elegiste y qué evidencia obtuviste?
3. Explica en una frase la diferencia entre Organization Policy e IAM.

**Pistas:** usa tu ficha real; no inventes resultados. Organization Policy restringe configuraciones; IAM determina quién puede realizar qué acción sobre qué recurso.

### Días hábiles previos · 12/10 y 09/10 · Lecciones 16 y 15

1. Antes de desplegar compute, ¿qué contexto debes confirmar en CLI?
2. Ordena estas comprobaciones: cuenta activa, proyecto, API, IAM, cuota, ubicación.
3. ¿Por qué un presupuesto no sustituye la limpieza de un laboratorio?

**Pistas:** confirma identidad y proyecto antes de operar. Una alerta de presupuesto no borra recursos ni garantiza detener el gasto.

### Hace 7 días · 07/10 · Lección 13

1. ¿Qué servicio responde “qué recursos existen”?
2. ¿Qué herramientas responden “cómo se comporta la aplicación” y “qué ocurrió”?
3. ¿Por qué una recomendación de Gemini Cloud Assist debe validarse antes de aplicarse?

**Pistas:** Cloud Asset Inventory; Cloud Monitoring y Cloud Logging; la persona operadora conserva responsabilidad sobre requisito, seguridad, costo y reversión.

### Hace 21 días · 23/09 · Lección 3

1. ¿Qué diferencia hay entre precio, costo y presupuesto?
2. ¿Qué recursos pueden seguir cobrando aunque una aplicación no reciba tráfico?
3. ¿Qué evidencia revisarías al terminar un laboratorio de compute?

**Pistas:** una tarifa se convierte en costo según consumo; discos, IP, nodos, VMs, instancias mínimas y almacenamiento de artefactos pueden generar cargos. Revisa inventario y Billing, no solo la pantalla de la aplicación.

---

## La pregunta central: ¿qué quieres administrar tú?

Usa la analogía de transportar equipos entre sedes:

- **Compute Engine** es comprar y conducir una camioneta: máximo control, también mantenimiento completo.
- **GKE Standard** es dirigir una flota con un sistema logístico: controlas nodos y reglas de Kubernetes.
- **GKE Autopilot** es usar la misma logística, pero Google opera la infraestructura de nodos.
- **Cloud Run** es contratar transporte por viaje para un contenedor: entregas la carga y defines condiciones; la plataforma escala.
- **Cloud Run functions** es solicitar un envío automático cuando ocurre un evento concreto: aportas una función y un trigger.
- **Agent Runtime** es contratar una plataforma especializada para un operador digital que razona, usa herramientas y mantiene sesiones.

La opción “más administrada” no siempre es mejor. La opción correcta es la de menor operación que todavía satisface todos los requisitos.

## Paso 1 · Identifica la unidad de despliegue

| La unidad real es... | Primera candidata |
|---|---|
| Sistema operativo, driver, daemon o aplicación heredada | Compute Engine |
| Conjunto de objetos Kubernetes, charts, operators o CRDs | GKE |
| Contenedor HTTP, worker o job sin necesidad de clúster | Cloud Run |
| Una función pequeña desde código fuente | Cloud Run functions |
| Una aplicación agentic con herramientas y sesiones | Agent Runtime |

Esto solo genera una candidata. Todavía debes comprobar eventos, control, estado y operación.

## Paso 2 · Identifica cómo empieza el trabajo

### Solicitud HTTP o gRPC

Una API Node.js que recibe solicitudes y puede ser stateless encaja naturalmente en un **Cloud Run service**. GKE también puede servir HTTP, pero solo se justifica si existen requisitos de Kubernetes. Compute Engine puede hacerlo, pero obliga a administrar VM, parches, health checks y escalado.

### Evento

Si una acción pequeña comienza al llegar un objeto o mensaje, una **Cloud Run function** con un trigger de Eventarc es una candidata fuerte. Si el procesamiento ya es una aplicación de contenedor compleja, el evento también puede dirigirse a un **Cloud Run service**.

### Trabajo finito o programado

Un proceso que inicia, completa y termina puede ser un **Cloud Run job**. No elijas una función solo porque “corre ocasionalmente”; compara duración, empaquetado, reintentos y complejidad.

### Proceso permanente

Un daemon que requiere control de host puede necesitar **Compute Engine**. Un conjunto de controladores y workloads persistentes con semántica Kubernetes puede necesitar **GKE**. Cloud Run ofrece services, jobs y worker pools, pero no concede administración del host.

### Solicitud agentic

Si el workload planifica pasos, invoca herramientas y conserva sesiones como agente, considera **Agent Runtime**. No lo elijas para una API CRUD ordinaria solo porque la aplicación usa Gemini en una ruta.

## Paso 3 · Decide cuánto control necesitas

### Compute Engine

Elige Compute Engine cuando el requisito exige control de VM, sistema operativo, software de bajo nivel, drivers, imágenes específicas, procesos heredados o configuraciones que una plataforma más administrada no expone.

**Tú operas:** sistema operativo, parches, proceso, disponibilidad de instancias y buena parte del escalado. Puedes usar managed instance groups y VM Manager para reducir trabajo, pero sigues trabajando en el nivel de VM.

**Ejemplo SiteOps Tracker:** un colector heredado que necesita una interfaz de red especial, un paquete del sistema y un daemon permanente.

**No elegirlo cuando:** una API stateless estándar cabe en un contenedor de Cloud Run y no necesita acceso al host. En ese caso, la VM añade mantenimiento sin aportar valor.

### Google Kubernetes Engine (GKE)

Elige GKE cuando el requisito es Kubernetes: manifiestos, portabilidad de Kubernetes, CRDs, operators, políticas de scheduling, topologías complejas o un ecosistema de microservicios que el equipo sabe operar.

- **Autopilot:** Google gestiona la infraestructura de nodos, escalado y varias configuraciones predeterminadas. Es la primera opción de GKE para la mayoría de workloads compatibles.
- **Standard:** Google gestiona el control plane, mientras tu equipo conserva mayor control de nodos y node pools. Úsalo cuando ese control sea un requisito, no por costumbre.

**Ejemplo SiteOps Tracker:** varios servicios ya distribuidos como Helm charts, con un operator, sidecars y políticas Kubernetes.

**No elegirlo cuando:** solo tienes una API Node.js y una base PostgreSQL. Un clúster agrega objetos, permisos, upgrades, observabilidad y capacidad que quizá no necesitas.

### Cloud Run

Cloud Run es una plataforma totalmente administrada para ejecutar código, funciones o contenedores. Para una aplicación contenerizada, sus formas principales incluyen:

- **Service:** recibe solicitudes y mantiene revisiones; apropiado para API o web.
- **Job:** ejecuta tareas hasta completarlas.
- **Worker pool:** ejecuta workloads continuos sin endpoint HTTP de entrada, cuando el modelo sea compatible.

**Ejemplo SiteOps Tracker:** API Node.js stateless que recibe tráfico irregular y guarda datos duraderos en PostgreSQL externo al contenedor.

**No elegirlo cuando:** necesitas modificar el kernel, administrar el host, usar un operator de Kubernetes o depender de disco local como fuente duradera de verdad.

### Cloud Run functions

Cloud Run functions ofrece un modelo centrado en una función HTTP o CloudEvents. El despliegue desde código fuente usa buildpacks y Cloud Build; Artifact Registry almacena artefactos. Eventarc puede conectar eventos de otros servicios.

**Ejemplo SiteOps Tracker:** al cargar una evidencia en Cloud Storage, extraer metadatos y registrar una tarea de validación.

**No elegirlo cuando:** el sistema es una API con muchas rutas, un proceso complejo con varios componentes o un contenedor que ya tiene su propio servidor. Un Cloud Run service suele dar un modelo más claro.

### Agent Runtime

Agent Runtime en **Gemini Enterprise Agent Platform** - evolución de Vertex AI Agent Engine en la guía - es un runtime administrado y especializado para aplicaciones agentic. Puede desplegar agentes desde imagen, Dockerfile, fuentes u otros flujos admitidos, y ofrece capacidades de operación para agentes.

**Ejemplo SiteOps Tracker:** un agente que recibe un hallazgo, consulta herramientas autorizadas, propone una clasificación, mantiene sesión y devuelve una explicación trazable para revisión humana.

**No elegirlo cuando:** solo necesitas servir React, ejecutar una API CRUD Node.js o procesar un evento determinista. Cloud Run, functions, GKE o Compute Engine son opciones de cómputo generales más directas.

---

## Tabla comparativa de decisión

| Criterio | Compute Engine | GKE Autopilot | GKE Standard | Cloud Run service/job | Cloud Run functions | Agent Runtime |
|---|---|---|---|---|---|---|
| Unidad | VM / bare metal | Pods y objetos Kubernetes | Pods, objetos y nodos Kubernetes | Contenedor | Función desde source | Aplicación agentic |
| Infraestructura | Tú administras VM y SO | Google administra nodos | Tú administras nodos; Google, control plane | Google administra plataforma | Google administra plataforma y build | Google administra runtime agentic |
| Control del host | Alto | Bajo | Medio/alto sobre nodos | No | No | No es el objetivo |
| Kubernetes API | No nativa | Sí | Sí | No | No | No es el criterio central |
| HTTP | Sí, autogestionado | Sí | Sí | Natural para services | Sí para HTTP functions | Solicitudes a agentes |
| Eventos | Debes integrar | Integración posible | Integración posible | Eventarc/Pub/Sub, según diseño | CloudEvents/Eventarc es natural | Eventos pueden iniciar lógica agentic, según diseño |
| Batch finito | Scripts/VM | Jobs de Kubernetes | Jobs de Kubernetes | Cloud Run jobs | Posible si encaja como función | No es un batch genérico |
| Escala a cero | No de forma natural para VM | Depende del workload/capacidad | Depende de configuración | Sí en modelos compatibles | Sí en modelos compatibles | Gestionada según runtime y configuración |
| Carga operativa relativa | Alta | Media | Alta | Baja | Baja | Baja de infraestructura, especializada en agentes |
| Mejor señal de examen | SO, driver, VM, legado | Kubernetes sin operar nodos | Control de node pools/Kubernetes | Contenedor stateless o job | Handler pequeño orientado a evento | Agente, tools, sessions, reasoning |

## Árbol de decisión rápido

1. **¿Es un agente de IA con tools/sessions y ciclo agentic?** Considera Agent Runtime.
2. **¿Exige Kubernetes API, operator, CRD o scheduling Kubernetes?** Considera GKE.
3. **¿Exige control de SO, host, driver o software heredado?** Considera Compute Engine.
4. **¿Es un handler pequeño activado por HTTP o CloudEvent?** Considera Cloud Run functions.
5. **¿Es un contenedor HTTP, job o worker compatible sin requisitos anteriores?** Considera Cloud Run.
6. Si dos opciones sirven, elige la que cumpla requisitos con menor carga operativa y costo total justificable.

## Señales trampa del examen

- “Containerized” no obliga a GKE.
- “Event-driven” no obliga a functions; un evento también puede llegar a Cloud Run.
- “Full control” sin especificar qué control no justifica una VM.
- “Microservices” no obliga a Kubernetes; busca requisitos de Kubernetes.
- “Uses Gemini” no convierte automáticamente la aplicación en agente.
- “Serverless” no significa sin costo, sin límites o sin observabilidad.
- “Existing Kubernetes manifests/operators” sí es una señal fuerte hacia GKE.
- “Custom OS package/driver/kernel” es una señal fuerte hacia Compute Engine.

## Glosario bilingüe

| English | Español | Uso en decisión |
|---|---|---|
| workload | carga de trabajo | Código y procesos que resuelven una función |
| virtual machine | máquina virtual | Instancia con sistema operativo administrable |
| container image | imagen de contenedor | Paquete inmutable para ejecutar una aplicación |
| managed platform | plataforma administrada | El proveedor opera más infraestructura |
| control plane | plano de control | Componentes que administran el estado de Kubernetes |
| node pool | grupo de nodos | Conjunto de máquinas worker de GKE Standard |
| manifest | manifiesto | Declaración YAML de un objeto Kubernetes |
| custom resource definition (CRD) | definición de recurso personalizado | Extiende la API de Kubernetes |
| request-driven | orientado a solicitudes | Trabajo iniciado por HTTP/gRPC |
| event-driven | orientado a eventos | Trabajo iniciado por un evento |
| trigger | disparador | Regla que conecta un evento con un destino |
| service | servicio | Aplicación Cloud Run que recibe solicitudes |
| job | trabajo | Ejecución finita hasta completar tareas |
| function | función | Handler pequeño HTTP o CloudEvents |
| scale to zero | escalar a cero | No mantener instancias activas sin demanda, si el modelo lo permite |
| cold start | arranque en frío | Latencia al iniciar una instancia bajo demanda |
| agentic application | aplicación agentic | Agente que razona, usa tools y ejecuta pasos |
| operational burden | carga operativa | Trabajo de parches, escalado, upgrades y diagnóstico |

---

## Práctica: construye la tabla de decisión

La práctica no crea recursos. Primero inspeccionarás superficies de compute y luego completarás una matriz para SiteOps Tracker.

### Parte A · Verificación segura de contexto

Los comandos siguientes fueron verificados en la documentación oficial vigente consultada el 14/10/2026.

```bash
export PROJECT_ID="your-lab-project-id"
export REGION="us-central1"

gcloud auth list --filter=status:ACTIVE --format="value(account)"
gcloud config list project
gcloud projects describe "$PROJECT_ID"
```

Detente si la cuenta o el proyecto no corresponden a un laboratorio autorizado.

### Parte B · Inspección en Google Cloud console

Usa el buscador de la consola; las etiquetas de navegación pueden evolucionar.

1. Selecciona el proyecto de laboratorio.
2. Abre **Compute Engine → VM instances**. Registra qué control observarías: tipo de máquina, zona, disco e imagen de SO.
3. Abre **Kubernetes Engine → Clusters**. Registra si un clúster visible usa Autopilot o Standard; no lo modifiques.
4. Abre **Cloud Run** y revisa las pestañas disponibles para services, jobs, worker pools o functions.
5. Busca **Agent Runtime** en Gemini Enterprise Agent Platform. Si tu proyecto no tiene acceso o API habilitada, usa la documentación; no habilites nada solo para completar esta lección.
6. Anota para cada superficie qué administra Google y qué administraría tu equipo.

### Parte C · Inspección con CLI

Todas son consultas. Una lista vacía es un resultado válido.

```bash
gcloud compute instances list \
  --project="$PROJECT_ID"

gcloud container clusters list \
  --project="$PROJECT_ID"

gcloud run services list \
  --project="$PROJECT_ID" \
  --region="$REGION"

gcloud functions list \
  --project="$PROJECT_ID" \
  --regions="$REGION" \
  --v2
```

Interpreta, no solo copies:

- ¿El inventario está vacío o falló la consulta?
- ¿El recurso tiene alcance zonal, regional o multirregional?
- ¿Qué superficie operativa aparece para cada opción?
- ¿Qué comando requiere una región explícita y cuál consulta todas las ubicaciones por defecto?

### Parte D · Tabla de decisión obligatoria

Completa primero la tabla sin mirar la propuesta posterior.

| Workload de SiteOps Tracker | Inicio: request/event/schedule/continuous | Unidad: VM/Kubernetes/container/function/agent | Control requerido | Operación aceptable | Elección | Por qué no las otras |
|---|---|---|---|---|---|---|
| API Node.js stateless con tráfico irregular |  |  |  |  |  |  |
| Colector heredado que necesita paquete de SO y daemon |  |  |  |  |  |  |
| Suite de microservicios con Helm, operator y CRD |  |  |  |  |  |  |
| Procesar evidencia al cargarse en Cloud Storage |  |  |  |  |  |  |
| Reconciliación nocturna que termina al completar |  |  |  |  |  |  |
| Agente que analiza hallazgos, usa tools y mantiene sesión |  |  |  |  |  |  |

### Propuesta razonada para comparar

| Workload | Elección probable | Motivo principal | Alternativa cercana y por qué no |
|---|---|---|---|
| API Node.js stateless, tráfico irregular | Cloud Run service | Contenedor HTTP, plataforma administrada, escalado automático | GKE agrega clúster sin requisito Kubernetes; VM agrega SO y parcheo |
| Colector con paquete de SO y daemon | Compute Engine | Necesita control de host y proceso permanente | Cloud Run no expone host; functions no modela un daemon |
| Helm + operator + CRD | GKE | El requisito explícito es Kubernetes | Cloud Run ejecuta contenedores, pero no expone Kubernetes API/CRDs |
| Evidencia de Cloud Storage | Cloud Run function + Eventarc | Handler pequeño orientado a evento | VM mantiene infraestructura; GKE es excesivo para un handler aislado |
| Reconciliación nocturna finita | Cloud Run job | Contenedor batch que termina | Service espera solicitudes; function puede ser demasiado estrecha para un proceso complejo |
| Agente con tools y sesiones | Agent Runtime | Runtime especializado en ciclo de vida agentic | Cloud Run podría alojar código, pero tendrías que construir más capacidades operativas de agente |

### Parte E · Defensa oral de 90 segundos

Elige dos filas y usa esta estructura:

> “The workload is triggered by ___. Its deployment unit is ___. It requires ___ control but does not require ___. I would choose ___ because ___. I would not choose ___ because ___.”

Grábate o léelo en voz alta. El objetivo no es pronunciación perfecta, sino una decisión técnica clara en inglés.

## Alternativa conceptual completa sin cuenta o crédito

Si no tienes acceso a Google Cloud:

1. Copia la tabla obligatoria en una hoja.
2. Completa sus seis filas usando el árbol de decisión.
3. Para cada fila, escribe un costo operativo oculto de la alternativa descartada.
4. Dibuja cinco tarjetas: VM, Kubernetes, Container, Function, Agent.
5. Coloca cada workload en una tarjeta y permite solo una tarjeta principal por workload.
6. Defiende dos decisiones con la plantilla en inglés.
7. Resuelve las diez preguntas y registra errores.

Esta alternativa cubre íntegramente el objetivo de seleccionar servicio; no simula una implementación que no realizaste.

## Resultado esperado

Al finalizar debes tener:

- una tabla de seis workloads con una elección defendible por fila;
- al menos una razón explícita para no elegir la alternativa más cercana;
- evidencia de cuatro consultas de inventario o la alternativa conceptual completa;
- dos defensas orales en inglés;
- una ficha de errores con reglas de decisión reutilizables.

## Solución de problemas

| Síntoma | Interpretación | Comprobación | Acción segura |
|---|---|---|---|
| `PERMISSION_DENIED` | Falta permiso en ese servicio/proyecto | Confirma cuenta, proyecto, comando y recurso | Solicita permiso mínimo; no pidas `Owner` por reflejo |
| API disabled | La API del producto no está habilitada | Confirma proyecto y mensaje exacto | Documenta; habilita solo en laboratorio autorizado |
| Lista vacía | No hay recursos visibles en ese alcance | Verifica región, filtros y proyecto | Registra “sin recursos”; no lo llames error sin evidencia |
| Cloud Run no muestra servicios | Región incorrecta o sin despliegues | Prueba la región documentada del laboratorio | No recorras regiones creando recursos |
| Functions no coincide con consola | Generación/API o región distinta | Usa `--v2` y `--regions` como en la práctica | Compara nombre, región y generación |
| GKE tarda o parece complejo | Estás evaluando operación, no solo despliegue | Pregunta si Kubernetes es un requisito | Si no lo es, compara Cloud Run |
| Agent Runtime no aparece | Acceso, disponibilidad o API no configurada | Consulta documentación oficial actual | Completa la evaluación conceptual; no habilites por curiosidad |

### Refuerzo de errores compartidos anteriormente

- **SSH timeout:** si el diseño no necesita control de SO, Cloud Run puede eliminar la administración de SSH; pero migrar de plataforma no sustituye entender red, firewall/IAP, estado de VM e identidad. Si el requisito exige VM, diagnostica por capas.
- **MIG backend `UNHEALTHY`:** Compute Engine con MIG exige que health check, puerto, ruta, proceso y firewall coincidan. Cloud Run administra más infraestructura, pero la aplicación todavía debe iniciar y responder según su contrato. No elijas servicio solo para ocultar un error de aplicación.

## Impacto en costos y limpieza

### Modelo mental de costo

| Servicio | Fuentes típicas de costo que debes revisar |
|---|---|
| Compute Engine | Tiempo de VM, máquina, discos, snapshots, IP y red; una VM encendida puede cobrar sin tráfico |
| GKE | Cómputo de workloads/nodos, gestión aplicable, discos, balanceo y red; Standard puede mantener capacidad ociosa |
| Cloud Run | CPU/memoria/instancias según configuración, solicitudes, red, builds e imágenes; `min instances` puede evitar escala a cero |
| Cloud Run functions | Ejecución, builds, Artifact Registry, Eventarc y red según uso/configuración |
| Agent Runtime | Compute agentic, modelos/APIs, herramientas, almacenamiento, red y componentes de plataforma según configuración |

La opción con menor precio unitario no siempre tiene menor costo total. Incluye horas de operación, parches, upgrades, incidentes y especialización del equipo.

### Limpieza

La práctica es de solo lectura y no crea recursos. Al terminar:

```bash
unset PROJECT_ID
unset REGION
```

Si hiciste pruebas adicionales en una cuenta propia, revisa antes de borrar: VMs y discos, clústeres y node pools, Cloud Run services/jobs/functions, Artifact Registry, Cloud Build, triggers de Eventarc e instancias mínimas. Elimina solo recursos que tú creaste y cuyo nombre, proyecto y región confirmaste.

---

## Preguntas originales tipo examen

Responde sin consultar las soluciones. Todas las preguntas son originales y no proceden de dumps.

### Question 1

SiteOps Tracker must run a legacy Windows service that installs a specific OS driver and requires administrators to manage the operating system. Which compute option is the best fit?

A. Cloud Run functions  
B. Compute Engine  
C. GKE Autopilot  
D. Agent Runtime

### Question 2

A platform team already operates Kubernetes. A new workload requires a custom operator, CRDs, and direct control over node pools. Which option should the team choose?

A. GKE Standard  
B. Cloud Run service  
C. Cloud Run function  
D. Compute Engine without Kubernetes

### Question 3

A team needs Kubernetes APIs and manifests but wants Google to manage node infrastructure, scaling, and many security settings. The workload is compatible with the managed constraints. Which option is most appropriate?

A. GKE Autopilot  
B. GKE Standard with manually managed nodes  
C. A single Compute Engine VM  
D. Cloud Run function

### Question 4

The SiteOps Tracker Node.js API is containerized, stateless, receives irregular HTTPS traffic, and stores durable data in PostgreSQL. The team does not need Kubernetes. Which option minimizes operational burden?

A. Compute Engine  
B. GKE Standard  
C. Cloud Run service  
D. Agent Runtime

### Question 5

Whenever a PDF is uploaded to Cloud Storage, a small TypeScript handler must extract metadata and publish a result. Which design is the best starting point?

A. A permanently running VM that polls the bucket  
B. A Cloud Run function triggered through Eventarc  
C. A GKE Standard cluster with one Pod  
D. Agent Runtime with a conversational session

### Question 6

A containerized reconciliation process runs every night, processes all open findings, and exits when complete. It does not serve requests. Which option best matches the execution model?

A. Cloud Run job  
B. Cloud Run service with a public URL  
C. A Cloud Run HTTP function kept warm  
D. A permanent Compute Engine VM solely to wait for the schedule

### Question 7

SiteOps Tracker needs an AI component that reasons over findings, calls approved tools, maintains sessions, and is governed as an enterprise agent. Which specialized runtime should be evaluated first?

A. Agent Runtime on Gemini Enterprise Agent Platform  
B. A Cloud Run function solely because it uses Python  
C. A Compute Engine VM solely because models use compute  
D. Cloud DNS

### Question 8 — Choose two

Which two requirements are strong signals for choosing GKE instead of a Cloud Run service?

A. The application is packaged as a container  
B. The workload depends on Kubernetes CRDs and operators  
C. The team needs explicit Kubernetes scheduling and node-pool control  
D. The application receives HTTPS requests  
E. The team wants automatic scaling

### Question 9

A standard CRUD API calls a Gemini model in one endpoint but has no planning loop, tools, or sessions. Which statement is most accurate?

A. It must use Agent Runtime because it calls Gemini  
B. It can use a general application platform such as Cloud Run; model usage alone does not make it an agentic workload  
C. It must use GKE Standard  
D. It must run on a Windows VM

### Question 10

A network collector requires a long-running daemon, a custom system package, and host-level packet capture. The team previously experienced SSH timeouts and wants to avoid VMs. What should the engineer do?

A. Choose Cloud Run functions because operational discomfort overrides technical requirements  
B. Choose Agent Runtime because it is fully managed  
C. Validate whether the host-level requirements can change; if they cannot, use Compute Engine and design secure operations  
D. Use GKE Autopilot because every container workload fits its restrictions

---

## Ficha de errores

| Q | Mi respuesta | Correcta | Confianza | Señal que ignoré | Regla corregida | Próxima revisión |
|---:|---|---|:---:|---|---|---|
|  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |

Clasifica el error: concepto, lectura del requisito, confusión de producto, exceso de control, subestimación operativa o inglés.

## Criterios de autoevaluación

- [ ] Puedo explicar el continuo VM → Kubernetes → contenedor administrado → función.
- [ ] Puedo distinguir GKE Autopilot de Standard sin decir que Autopilot “no usa nodos”.
- [ ] Puedo elegir Cloud Run service, job o function según la forma de iniciar y terminar trabajo.
- [ ] No elijo GKE solo porque hay contenedores.
- [ ] No elijo Agent Runtime solo porque hay una llamada a un modelo.
- [ ] Mi tabla contiene seis elecciones y seis descartes razonados.
- [ ] Ejecuté las consultas de inventario o completé toda la alternativa conceptual.
- [ ] Puedo defender dos decisiones en inglés en 90 segundos.
- [ ] Obtuve al menos 8/10 y corregí también los aciertos de baja confianza.
- [ ] No afirmo dominio hasta repetir la decisión sin consultar apuntes.

**Interpretación personal:**

- **9-10/10:** continúa, pero revisa respuestas de baja confianza.
- **8/10:** cumple la meta diaria; repite la regla del error.
- **6-7/10:** rehace el árbol y dos filas de la tabla en 24 horas.
- **0-5/10:** vuelve a los cinco pasos y explica una elección por servicio antes de avanzar.

Estos rangos son criterios de estudio, no una puntuación oficial de Google.

## Documentación oficial

Consultada el 14/10/2026:

- [Associate Cloud Engineer certification](https://cloud.google.com/learn/certification/cloud-engineer)
- [Compute Engine overview](https://cloud.google.com/compute/docs/overview)
- [GKE overview](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview)
- [GKE Autopilot overview](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview)
- [Choose a GKE mode of operation](https://cloud.google.com/kubernetes-engine/docs/concepts/choose-cluster-mode)
- [What is Cloud Run](https://cloud.google.com/run/docs/overview/what-is-cloud-run)
- [Create Cloud Run jobs](https://cloud.google.com/run/docs/create-jobs)
- [Cloud Run functions overview](https://cloud.google.com/run/docs/functions/overview)
- [Cloud Run function triggers](https://cloud.google.com/run/docs/function-triggers)
- [Create triggers with Eventarc](https://cloud.google.com/run/docs/triggering/trigger-with-events)
- [Gemini Enterprise Agent Platform overview](https://cloud.google.com/gemini-enterprise-agent-platform/overview)
- [Agent Runtime](https://cloud.google.com/gemini-enterprise-agent-platform/build/runtime)
- [Agent Platform name changes](https://cloud.google.com/gemini-enterprise-agent-platform/vertex-ai-name-changes)
- [`gcloud auth list`](https://cloud.google.com/sdk/gcloud/reference/auth/list)
- [`gcloud config list`](https://cloud.google.com/sdk/gcloud/reference/config/list)
- [`gcloud projects describe`](https://cloud.google.com/sdk/gcloud/reference/projects/describe)
- [`gcloud compute instances list`](https://cloud.google.com/sdk/gcloud/reference/compute/instances/list)
- [`gcloud container clusters list`](https://cloud.google.com/sdk/gcloud/reference/container/clusters/list)
- [`gcloud run services list`](https://cloud.google.com/sdk/gcloud/reference/run/services/list)
- [`gcloud functions list`](https://cloud.google.com/sdk/gcloud/reference/functions/list)

---

## Soluciones justificadas

### Solution 1 — B

**Compute Engine** provides the required OS and driver control. **A** offers a function abstraction with no host administration. **C** manages infrastructure under Autopilot constraints and is not the direct choice for a Windows service requiring OS administration. **D** is specialized for agentic applications, not legacy Windows services.

### Solution 2 — A

**GKE Standard** matches explicit Kubernetes features plus node-pool control. **B** runs containers without exposing Kubernetes CRDs/operators. **C** is a function model and does not satisfy the Kubernetes requirement. **D** omits the requested Kubernetes orchestration.

### Solution 3 — A

**GKE Autopilot** preserves Kubernetes APIs while Google manages node infrastructure and many operational settings. **B** adds node management that the team wants to avoid. **C** supplies one VM, not a Kubernetes platform. **D** does not expose Kubernetes manifests and APIs.

### Solution 4 — C

A **Cloud Run service** directly fits a stateless container serving irregular HTTPS traffic and minimizes infrastructure operation. **A** adds OS maintenance and VM scaling. **B** adds a cluster without a Kubernetes requirement. **D** is for agentic workloads, not an ordinary API.

### Solution 5 — B

A **Cloud Run function triggered through Eventarc** fits a small handler activated by a Cloud Storage event. **A** wastes capacity polling and requires VM operation. **C** introduces cluster overhead for one handler. **D** is inappropriate because the task is deterministic event processing, not agentic reasoning.

### Solution 6 — A

A **Cloud Run job** is designed for a containerized task that runs to completion. **B** models request-serving traffic. **C** forces a function/HTTP abstraction and warm-instance idea that the requirement does not need. **D** leaves a VM running merely to wait, increasing operation and potential idle cost.

### Solution 7 — A

**Agent Runtime** is the specialized managed runtime for enterprise agentic applications with tools and sessions. **B** chooses by language instead of workload behavior. **C** chooses raw compute without addressing the agent lifecycle. **D** provides DNS, not compute or agent operation.

### Solution 8 — B and C

CRDs/operators and explicit Kubernetes scheduling/node-pool control are strong GKE requirements. **A** is insufficient because Cloud Run also runs containers. **D** is supported by many platforms, including Cloud Run. **E** is also available across managed platforms and does not by itself require Kubernetes.

### Solution 9 — B

A general application platform such as **Cloud Run** can host a CRUD API that happens to call a model. **A** confuses model use with an agentic architecture. **C** invents a Kubernetes requirement. **D** invents an operating-system requirement.

### Solution 10 — C

The engineer should first challenge the requirements; if host-level packet capture, a system package, and a daemon remain mandatory, **Compute Engine** is the direct fit and must be operated securely. **A** lets discomfort override hard constraints. **B** is unrelated to a network daemon. **D** falsely assumes every container is compatible with Autopilot and ignores host-level restrictions.
