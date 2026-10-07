# ACE 2026 - Lección 13: Google Cloud Observability, Cloud Asset Inventory y Gemini Cloud Assist

> **Fecha:** 7 de octubre de 2026  
> **Dominio de la guía ACE:** 1.1 Setting up cloud projects and accounts  
> **Tema del plan:** Google Cloud Observability, Cloud Asset Inventory y Gemini Cloud Assist  
> **Práctica del plan:** Encuentra recursos en inventario y plantea una consulta de análisis y observabilidad.  
> **Duración sugerida:** 75-90 minutos  
> **Caso conductor:** SiteOps Tracker, proyecto ficticio de portafolio.

Esta lección enseña y ejercita una parte del temario. Recibir el archivo no demuestra dominio: compruébalo realizando o simulando la práctica, explicando qué fuente usarías para cada pregunta operativa y justificando las respuestas sin mirar las soluciones.

## 1. Objetivo

Al finalizar podrás:

1. distinguir **inventario**, **telemetría** y **asistencia generativa**;
2. explicar las funciones de métricas, logs y trazas dentro de Google Cloud Observability;
3. buscar recursos con Cloud Asset Inventory en Console y con `gcloud asset search-all-resources`;
4. escribir una consulta de inventario para detectar activos sin etiquetas de gobierno;
5. escribir y ejecutar una consulta de Cloud Logging para investigar señales operativas;
6. reconocer que Cloud Asset Inventory usa un modelo de consistencia eventual y no sustituye a la API del servicio cuando se necesita estado en vivo;
7. usar Gemini Cloud Assist como ayuda contextual y validar sus afirmaciones con consultas deterministas;
8. elegir el servicio correcto según la pregunta y descartar alternativas razonables pero inadecuadas;
9. estimar el impacto económico de inventario y observabilidad;
10. documentar evidencia, problemas, limpieza y errores de razonamiento.

### Evidencia de aprendizaje

Conserva al terminar:

- una tabla de activos obtenida de un proyecto o del conjunto ficticio;
- una consulta de Cloud Asset Inventory que encuentre recursos sin `env` u `owner`;
- una consulta de Logging que muestre eventos `WARNING` o superiores;
- una hipótesis que conecte un activo con una señal de observabilidad, sin afirmar causalidad no demostrada;
- un prompt seguro para Gemini Cloud Assist y una lista de comprobaciones independientes;
- el registro de costo y limpieza;
- la puntuación de las diez preguntas y una ficha por cada error.

## 2. Alineación con la guía oficial

La guía oficial adjunta de Associate Cloud Engineer incluye, dentro de **1.1 Setting up cloud projects and accounts**:

- **Provisioning and setting up products in Google Cloud Observability**.
- **Configuring Cloud Asset Inventory and using Gemini Cloud Assist to analyze resources**.

La práctica de hoy cubre exactamente esos puntos: localizar activos, formular una consulta de inventario, observar eventos y usar la asistencia generativa con verificación. La creación de alertas, métricas personalizadas, Audit Logs, VPC Flow Logs, Trace y Ops Agent se estudia con mayor profundidad en el dominio 3.4; hoy se construye el modelo mental inicial.

## 3. Prerrequisitos explicados desde cero

### 3.1 Proyecto y alcance

Un **project** es el contenedor administrativo donde se habilitan APIs, se crean recursos, se aplican cuotas y se registra consumo. Para una búsqueda de inventario, el **scope** indica el contenedor que se quiere consultar:

- `projects/PROJECT_ID` busca en un proyecto;
- `folders/FOLDER_ID` busca en una carpeta y sus descendientes admitidos;
- `organizations/ORGANIZATION_ID` busca en una organización.

El alcance no amplía tus permisos. Pedir el ámbito de una organización no permite ver recursos para los que tu identidad carece de acceso.

### 3.2 API habilitada, IAM y datos disponibles son condiciones distintas

Una consulta puede fallar o regresar cero filas por razones diferentes:

1. la API necesaria no está habilitada;
2. la identidad no tiene el permiso requerido;
3. el alcance es incorrecto;
4. no existen activos o señales que coincidan;
5. el dato aún no se indexó o está sujeto a consistencia eventual;
6. el filtro elimina todos los resultados.

No concluyas “el recurso no existe” solo porque una herramienta devolvió una lista vacía.

### 3.3 IAM mínimo para la práctica

Para ver metadatos de activos, la documentación recomienda:

- **Cloud Asset Viewer** (`roles/cloudasset.viewer`);
- **Service Usage Consumer** (`roles/serviceusage.serviceUsageConsumer`).

Todas las llamadas de Cloud Asset Inventory requieren `serviceusage.services.use`; `SearchAllResources` requiere `cloudasset.assets.searchAllResources`. Para consultar logs normalmente se usa **Logs Viewer** (`roles/logging.viewer`), y para consultar datos de Monitoring, **Monitoring Viewer** (`roles/monitoring.viewer`). Gemini Cloud Assist requiere su configuración administrativa y el rol **Gemini Cloud Assist User** (`roles/geminicloudassist.user`), además de acceso a las fuentes que deba inspeccionar.

Solicitar `Owner` para evitar comprender los permisos es una mala práctica. Si no puedes obtener el rol mínimo en un proyecto autorizado, realiza la alternativa conceptual completa.

### 3.4 Cloud Shell y Google Cloud CLI

Puedes usar Cloud Shell o una instalación actual de Google Cloud CLI. Antes de consultar:

- verifica la cuenta activa;
- fija explícitamente el proyecto;
- no pegues secretos, tokens, datos personales ni contenido de producción en la terminal, en Cloud Assist o en esta ficha;
- usa un proyecto de laboratorio, no un entorno crítico.

### 3.5 Telemetría

**Telemetry** son señales que describen el comportamiento de un sistema. Las tres señales principales son:

- **metrics:** valores numéricos a lo largo del tiempo;
- **logs:** registros de eventos con marca temporal;
- **traces:** recorrido de una solicitud a través de componentes y spans.

Una aplicación vacía o recién creada quizá no tenga telemetría útil. Eso no significa que la consulta sea incorrecta.

## 4. Modelo mental: directorio, sensores y copiloto

Imagina la operación de varias sedes de SiteOps Tracker:

- **Cloud Asset Inventory** es el directorio maestro de infraestructura: qué recursos se conocen, de qué tipo son, dónde están y qué metadatos tienen.
- **Cloud Monitoring** es el tablero de sensores: series numéricas como latencia, uso de CPU o conteo de solicitudes.
- **Cloud Logging** es la bitácora: eventos detallados como un error de conexión a PostgreSQL o una denegación de acceso.
- **Cloud Trace** es el rastreo de un paquete: muestra por qué servicios pasó una solicitud y cuánto tardó cada tramo.
- **Gemini Cloud Assist** es un copiloto que interpreta contexto y propone explicaciones o consultas, pero puede equivocarse.

La analogía revela una regla de examen: el directorio no reemplaza a los sensores y el copiloto no reemplaza a la evidencia.

## 5. Tres capas que no deben confundirse

### 5.1 Capa 1: estado y metadatos de recursos

Cloud Asset Inventory es un servicio global de inventario de metadatos. Permite ver, buscar, exportar, supervisar y analizar activos de Google Cloud. Sus usos típicos incluyen:

- descubrimiento de recursos;
- auditorías de seguridad y costo;
- revisión de ubicación, etiquetas, estado y fechas;
- historial de cambios de metadatos;
- exportaciones para análisis;
- feeds de cambios.

Cloud Asset Inventory conserva hasta **35 días** de historial de creación, actualización y eliminación de activos. Si un activo no cambió durante más tiempo, una consulta histórica puede devolver el estado más reciente disponible.

### 5.2 Capa 2: comportamiento operativo

Google Cloud Observability responde preguntas como:

- ¿subió la latencia del API?
- ¿cuántos errores ocurrieron en la última hora?
- ¿qué dependencia consumió más tiempo en una solicitud?
- ¿se está acercando una métrica a un umbral?

No basta con saber que existe un servicio de Cloud Run o una instancia de Cloud SQL. Para operarlo necesitas señales sobre su comportamiento.

### 5.3 Capa 3: interpretación asistida

Gemini Cloud Assist puede usar el proyecto, la organización, la página visible de Console y otras fuentes autorizadas como contexto. Puede ayudar a:

- explicar productos y prácticas;
- inspeccionar recursos y cambios;
- proponer consultas y comandos;
- analizar rendimiento si Monitoring está habilitado y la identidad tiene permisos;
- orientar una investigación.

La oferta consultada es **Preview**. Google advierte que la salida puede parecer plausible y ser incorrecta. Debes validar cada afirmación antes de ejecutar una acción.

## 6. Google Cloud Observability paso a paso

### 6.1 Métricas

Una **metric** contiene puntos numéricos ordenados por tiempo. Ejemplos para SiteOps Tracker:

- solicitudes por segundo al API Node.js;
- latencia p95 del endpoint de hallazgos;
- utilización de CPU;
- conexiones activas de PostgreSQL;
- tamaño de una cola.

Usa una métrica cuando importa la tendencia, el umbral, una agregación o una alerta. Una métrica no suele explicar por sí sola el detalle exacto de un fallo individual.

### 6.2 Logs

Un **log entry** representa un evento. Puede incluir `timestamp`, `severity`, tipo de recurso, nombre del log y payload estructurado o textual. Ejemplos:

- `ERROR`: la API no pudo escribir un hallazgo;
- `WARNING`: se agotó un pool de conexiones;
- Audit Log: una identidad cambió una política;
- request log: una llamada regresó HTTP 500.

Usa logs cuando necesitas el evento, el mensaje, la identidad, el código o el contexto de una operación.

### 6.3 Traces

Una **trace** agrupa spans que representan tramos de una solicitud. Para una acción “cerrar hallazgo” podrías observar:

1. React envía una solicitud;
2. la API Node.js valida la identidad;
3. la API consulta PostgreSQL;
4. la API registra el cambio de estado;
5. la respuesta vuelve al navegador.

Usa Trace cuando la pregunta es “¿en qué tramo se consumió el tiempo?”. Un log puede decir que hubo lentitud; una traza ayuda a localizarla dentro de una petición distribuida.

### 6.4 Correlación no es causalidad

Si ves un cambio de configuración a las 14:00 y errores a las 14:02, existe correlación temporal. Aún debes comprobar:

- qué cambió exactamente;
- si afectó al recurso que emite errores;
- si el error empezó después del cambio;
- si existen otras causas compatibles;
- si una reversión controlada o evidencia adicional confirma la hipótesis.

En el examen, la mejor primera acción suele ser recopilar la señal más directa y reducir el alcance antes de aplicar un cambio destructivo.

## 7. Cloud Asset Inventory paso a paso

### 7.1 Qué es un asset

Un **asset** es la representación de un recurso, una política o determinados metadatos compatibles. El inventario puede incluir campos como:

- `name`;
- `assetType`;
- `project`;
- `displayName`;
- `location`;
- `labels` y `tags`;
- `createTime` y `updateTime`;
- `state`.

La disponibilidad exacta de campos depende del tipo de recurso.

### 7.2 Búsqueda y listado no son idénticos

- `gcloud asset search-all-resources` busca recursos por alcance y por campos indexados, y permite `--query`, `--asset-types`, `--order-by` y `--read-mask`.
- `gcloud asset list` lista activos de un proyecto, carpeta u organización y permite escoger un `--content-type`, por ejemplo `resource` o `iam-policy`.

Para la práctica, **search-all-resources** es preferible porque la evidencia pedida es localizar y filtrar recursos. `asset list` sería útil si necesitas la representación de contenido de activos en un momento determinado.

### 7.3 Sintaxis de búsqueda

Una búsqueda combina campo, operador y valor:

- exacta: `location=us-central1-a`;
- parcial: `location:us-central1`;
- existencia de etiqueta: `labels.env:*`;
- negación: `NOT labels.env:*`;
- combinación: `NOT labels.env:* OR NOT labels.owner:*`.

El operador `=` exige coincidencia exacta y distingue mayúsculas/minúsculas. `:` tokeniza y permite coincidencia parcial. La verificación de existencia con `*` se admite para `labels`.

### 7.4 Consistencia eventual

El inventario puede tardar en reflejar cambios. Por tanto:

- úsalo para descubrimiento, gobierno, auditoría y análisis;
- no lo uses como única fuente para una decisión que pueda romper disponibilidad o seguridad en tiempo real;
- consulta la API específica del servicio cuando necesitas estado vivo.

Ejemplo: para una auditoría de activos detenidos, Asset Inventory es apropiado. Para decidir en este instante si una VM puede recibir una operación crítica, consulta la API de Compute Engine.

### 7.5 Alcance y permisos

Una búsqueda de organización es valiosa para gobierno central, pero requiere permisos en el nivel adecuado. Si solo administras el proyecto de desarrollo de SiteOps Tracker, usa el scope del proyecto. Principio de menor privilegio también significa limitar el alcance de observación.

## 8. Gemini Cloud Assist con una disciplina de verificación

### 8.1 Qué aporta

Cloud Assist puede convertir una pregunta natural en una explicación, consulta o flujo guiado. En el panel de Console, el contexto de la página actual puede hacer la respuesta más específica. La documentación indica que, para preguntas sobre recursos, también puede proporcionar una consulta equivalente para verificar resultados.

### 8.2 Prompt de calidad para SiteOps Tracker

Un buen prompt limita alcance, salida y acciones:

> Using only the resources visible in the selected project, summarize assets by service and location. Identify resources that appear to lack the `env` or `owner` label. For every claim, include the resource name and the query I can run to verify it. State uncertainty explicitly. Do not make changes.

Este prompt es mejor que “arregla mi proyecto” porque:

- fija el proyecto visible como alcance;
- define los campos deseados;
- exige nombres verificables;
- pide la consulta equivalente;
- prohíbe cambios;
- obliga a declarar incertidumbre.

### 8.3 Ciclo de verificación

1. Lee la respuesta como una hipótesis.
2. Identifica cada afirmación factual.
3. Ejecuta la consulta determinista propuesta o una equivalente.
4. Comprueba proyecto, scope, identidad y momento.
5. Revisa el comando con `gcloud ... --help` y la referencia oficial.
6. Evalúa permisos, costo y reversibilidad.
7. Solo entonces ejecuta una acción autorizada.

### 8.4 Límites y seguridad

- No envíes contraseñas, tokens, claves privadas ni datos personales.
- La página visible de Console puede formar parte del contexto; abre solo el recurso apropiado.
- No aceptes recomendaciones de IAM, red o borrado sin revisión humana.
- Cloud Assist no es la fuente apropiada para confirmar precios; usa la página oficial del producto.
- Si existe requisito de residencia de datos o CMEK, revisa la advertencia de configuración antes de habilitar Cloud Assist.
- La disponibilidad, etapa de lanzamiento y características pueden cambiar.

## 9. SiteOps Tracker: ejemplo integrado

SiteOps Tracker registra sedes, activos auditados, hallazgos, estados, responsables e historial. Su aplicación usa React + TypeScript, Node.js + TypeScript, PostgreSQL y servicios de Google Cloud.

Supón que una auditoría detecta que el panel tarda más en abrir hallazgos:

1. **Inventario:** confirma qué servicio ejecuta la API, en qué ubicación vive la base de datos y si ambos tienen etiquetas `env`, `owner` y `site`.
2. **Métricas:** comprueba si subió la latencia o el número de solicitudes.
3. **Logs:** filtra `WARNING` y `ERROR` del recurso que sirve el API.
4. **Trazas:** si la aplicación está instrumentada, identifica el span lento.
5. **Cloud Assist:** pide un resumen o una consulta, pero valida con las fuentes anteriores.
6. **PostgreSQL de la aplicación:** consulta el historial funcional del hallazgo si la pregunta trata del cambio de estado de negocio.

No confundas estos historiales:

| Historial | Qué responde | No responde por sí solo |
|---|---|---|
| PostgreSQL de SiteOps Tracker | Quién cambió un hallazgo, estado o responsable en la aplicación | Cambios de infraestructura de Google Cloud |
| Cloud Asset Inventory | Cambios de metadatos y configuración de activos compatibles | Cada request y cada error de aplicación |
| Cloud Logging / Audit Logs | Eventos operativos y administrativos registrados | Una vista completa y normalizada de todos los activos |
| Cloud Trace | Camino y latencia de una solicitud instrumentada | Inventario o gobierno de etiquetas |

## 10. Glosario bilingüe

| English term | Término en español | Definición útil para ACE |
|---|---|---|
| asset | activo | Recurso o metadato compatible representado en Cloud Asset Inventory. |
| asset type | tipo de activo | Identificador del producto y recurso, por ejemplo `compute.googleapis.com/Instance`. |
| inventory | inventario | Catálogo consultable de activos y metadatos. |
| scope | alcance | Proyecto, carpeta u organización sobre la que se ejecuta una búsqueda. |
| query | consulta | Expresión que filtra datos. |
| read mask | máscara de lectura | Lista de campos que se pide devolver. |
| eventual consistency | consistencia eventual | Un cambio puede tardar en aparecer en la vista consultada. |
| live state | estado en vivo | Estado actual obtenido de la API del servicio. |
| telemetry | telemetría | Señales que describen comportamiento operativo. |
| metric | métrica | Serie de valores numéricos a lo largo del tiempo. |
| log entry | entrada de log | Evento con marca temporal y contexto. |
| trace | traza | Recorrido de una solicitud. |
| span | tramo | Unidad de trabajo dentro de una traza. |
| severity | gravedad | Nivel de una entrada, como `WARNING` o `ERROR`. |
| observability | observabilidad | Capacidad de comprender el estado del sistema mediante señales. |
| prompt | instrucción o solicitud | Texto enviado a un sistema generativo. |
| grounding / context | fundamentación / contexto | Información que ayuda a producir una respuesta pertinente. |
| hallucination | alucinación | Salida plausible pero incorrecta o no sustentada. |
| least privilege | menor privilegio | Conceder solo los permisos y el alcance necesarios. |
| label | etiqueta | Par clave-valor de metadatos para organización y gobierno. |
| finding | hallazgo | Problema o condición registrada por SiteOps Tracker; no es un producto de Google Cloud en este ejemplo. |

## 11. Decisiones de servicio y por qué no elegir alternativas

| Necesidad | Elegir | Por qué | Por qué no las alternativas |
|---|---|---|---|
| Descubrir recursos de varios servicios por metadatos | Cloud Asset Inventory | Vista global y filtro por tipo, ubicación, etiquetas y estado | Logging registra eventos; Monitoring mide series; ninguna es un inventario normalizado. |
| Confirmar el estado actual antes de una acción crítica | API específica del servicio | Fuente viva y específica | Asset Inventory puede tener retraso de indexación. |
| Saber cuántos errores ocurrieron por minuto | Cloud Monitoring o métrica basada en logs | Serie temporal y agregación | Una búsqueda de assets no mide eventos; leer miles de logs no es la mejor vista de tendencia. |
| Conocer el mensaje de un error concreto | Cloud Logging | Conserva el evento y sus campos | Una métrica agregada pierde detalle; Trace se centra en el camino de solicitudes. |
| Localizar el tramo lento de una petición distribuida | Cloud Trace | Descompone la latencia por spans | Los logs aislados pueden no revelar la ruta completa. |
| Obtener una explicación conversacional o una consulta sugerida | Gemini Cloud Assist | Usa lenguaje natural y contexto autorizado | No sustituye queries deterministas ni revisión humana. |
| Auditar cambios funcionales de un hallazgo | Historial de la aplicación en PostgreSQL | Es el sistema de registro del dominio | Asset Inventory no conoce por sí mismo estados de negocio. |
| Exportar inventario para SQL y análisis prolongado | Asset Inventory a BigQuery | Permite análisis relacional y reportes | La búsqueda interactiva es mejor para exploración breve; exportar agrega costo y complejidad. |

### Regla de examen

Clasifica primero la pregunta:

1. **What exists and how is it configured?** → Asset Inventory.
2. **What is happening over time?** → Monitoring.
3. **What event occurred and with what context?** → Logging.
4. **Where did one request spend time?** → Trace.
5. **What is the live state now?** → service-specific API.
6. **Can AI summarize or propose a query?** → Cloud Assist, seguido de verificación.

## 12. Práctica guiada: inventario, consulta y observabilidad

### 12.1 Alcance y seguridad

Realiza la práctica en un proyecto de laboratorio autorizado. Los pasos principales son de lectura. Habilitar la API de Cloud Asset Inventory es opcional y requiere autorización. No concedas roles, no crees sinks, buckets, dashboards ni alertas, y no modifiques recursos para fabricar datos.

Define antes de comenzar:

```text
Proyecto de laboratorio: ______________________________
Cuenta autorizada: ___________________________________
Scope previsto: projects/_____________________________
Ventana temporal de logs: 1 día
Etiquetas de gobierno: env, owner
```

### 12.2 Parte A - Console: encontrar recursos

1. Abre Google Cloud Console y selecciona el proyecto de laboratorio.
2. Ve a **IAM & Admin > Asset Inventory**.
3. En la pestaña **Resource**, empieza sin filtro. La documentación recomienda observar primero el inventario y luego refinar.
4. Registra hasta diez filas con `asset type`, nombre, ubicación, estado y etiquetas.
5. Aplica un filtro de ubicación o tipo que exista en tu resultado.
6. Busca recursos que tengan la etiqueta `env`.
7. Cambia el criterio para identificar activos que parezcan carecer de `env` u `owner`.
8. No cambies etiquetas: la evidencia de hoy es descubrir y plantear una consulta.

Tabla de evidencia:

| # | Asset type | Resource name | Location | State | `env` | `owner` | Observación |
|---:|---|---|---|---|---|---|---|
| 1 |  |  |  |  |  |  |  |
| 2 |  |  |  |  |  |  |  |
| 3 |  |  |  |  |  |  |  |
| 4 |  |  |  |  |  |  |  |
| 5 |  |  |  |  |  |  |  |

### 12.3 Parte B - CLI: preflight

Los comandos y opciones de esta práctica se comprobaron el 7 de octubre de 2026 contra la documentación oficial de Google Cloud CLI.

```bash
export PROJECT_ID="YOUR_PROJECT_ID"

gcloud auth list
gcloud config set project "$PROJECT_ID"
gcloud config list
gcloud services list --enabled --project="$PROJECT_ID"
```

Comprueba visualmente que la cuenta y el proyecto sean correctos. En la lista de servicios busca `cloudasset.googleapis.com`.

Si Cloud Asset Inventory API no está habilitada y estás autorizado para habilitar servicios:

```bash
gcloud services enable cloudasset.googleapis.com \
  --project="$PROJECT_ID"
```

Habilitar una API no concede permisos para usarla.

### 12.4 Parte C - CLI: inventario inicial

Ejecuta primero una búsqueda sin `--query`:

```bash
gcloud asset search-all-resources \
  --scope="projects/$PROJECT_ID" \
  --limit=25 \
  --read-mask="name,assetType,displayName,location,state,labels" \
  --format="table(assetType,displayName,location,state,name)"
```

Qué demuestra cada opción:

- `--scope` limita el contenedor consultado;
- `--limit` limita filas devueltas, no el universo de activos;
- `--read-mask` pide solo campos necesarios;
- `--format` cambia la presentación local, no el filtro del servidor.

Guarda el número de filas visibles y al menos tres tipos de activo. Si no hay 25, no es un error: el proyecto puede contener menos.

### 12.5 Parte D - CLI: consulta de gobierno

Busca activos que carezcan de al menos una de las etiquetas esperadas:

```bash
gcloud asset search-all-resources \
  --scope="projects/$PROJECT_ID" \
  --query='NOT labels.env:* OR NOT labels.owner:*' \
  --limit=50 \
  --read-mask="name,assetType,displayName,location,state,labels" \
  --format="table(assetType,displayName,location,state,labels,name)"
```

Después, comprueba la búsqueda positiva:

```bash
gcloud asset search-all-resources \
  --scope="projects/$PROJECT_ID" \
  --query='labels.env:*' \
  --limit=50 \
  --read-mask="name,assetType,displayName,location,state,labels" \
  --format="table(assetType,displayName,location,state,labels,name)"
```

Interpreta los resultados con cautela:

- algunos tipos de activo quizá no admitan etiquetas;
- una etiqueta ausente no prueba una infracción hasta comparar con la política aplicable;
- una búsqueda vacía puede deberse a permisos, scope, datos o indexación;
- no añadas etiquetas sin revisar impacto, propietario y procedimiento de cambio.

### 12.6 Parte E - Console: consulta de observabilidad

1. Ve a **Logging > Logs Explorer**.
2. Selecciona el proyecto de laboratorio.
3. Usa una ventana de 24 horas.
4. Ejecuta una consulta amplia:

```text
severity >= "WARNING"
```

5. Si conoces el tipo de recurso de la API de SiteOps Tracker, agrega un filtro. Para un servicio de Cloud Run, un ejemplo es:

```text
resource.type = "cloud_run_revision"
AND severity >= "WARNING"
```

6. Inspecciona una entrada y registra `timestamp`, `severity`, `resource.type`, `logName` y un campo del payload.
7. Si no hay resultados, amplía primero la ventana o retira el filtro de tipo. No generes errores deliberadamente en un proyecto compartido.
8. Abre **Monitoring > Metrics Explorer** y observa una métrica existente de un recurso encontrado. Registra nombre de métrica, recurso, alineación y periodo; no crees un dashboard.

Consulta e hipótesis:

```text
Pregunta operativa: _________________________________________________
Consulta de Logs Explorer: __________________________________________
Métrica observada: __________________________________________________
Hipótesis sustentada: _______________________________________________
Dato que todavía falta: _____________________________________________
```

### 12.7 Parte F - CLI: leer logs

Ejecuta la misma idea desde CLI:

```bash
gcloud logging read \
  'severity >= "WARNING"' \
  --project="$PROJECT_ID" \
  --freshness=1d \
  --order=desc \
  --limit=25 \
  --format="table(timestamp,severity,resource.type,logName)"
```

Si el proyecto contiene Cloud Run y deseas reducir el alcance:

```bash
gcloud logging read \
  'resource.type = "cloud_run_revision" AND severity >= "WARNING"' \
  --project="$PROJECT_ID" \
  --freshness=1d \
  --order=desc \
  --limit=25 \
  --format="table(timestamp,severity,resource.type,logName)"
```

`--freshness` limita la antigüedad cuando se usa orden descendente sin una marca temporal explícita. `--order=desc` devuelve primero las entradas más recientes.

### 12.8 Parte G - Gemini Cloud Assist, solo si ya está disponible

1. Mantén seleccionado el proyecto de laboratorio.
2. Abre el panel **Gemini Cloud Assist** desde la barra de Console.
3. Envía el prompt de la sección 8.2, adaptado únicamente con nombres ficticios o metadatos no sensibles.
4. Copia la consulta verificable que proponga; no ejecutes comandos de escritura.
5. Compara cada recurso citado con Asset Inventory.
6. Marca cada afirmación como **confirmada**, **refutada** o **no comprobada**.
7. Registra una limitación observada.

Si Cloud Assist no aparece, no intentes evadir políticas. Un administrador debe configurarlo. La documentación vigente indica que para un proyecto se habilitan APIs requeridas y se conceden al menos `roles/geminicloudassist.user` y `roles/cloudasset.viewer`; el acceso adicional depende de las fuentes que se consulten.

### 12.9 Alternativa conceptual completa sin cuenta, crédito o permisos

Usa este inventario ficticio:

| Asset type | Display name | Location | State | Labels |
|---|---|---|---|---|
| `run.googleapis.com/Service` | `siteops-api-dev` | `us-central1` | ACTIVE | `env=dev`, `owner=app-team` |
| `sqladmin.googleapis.com/Instance` | `siteops-db-dev` | `us-central1` | RUNNABLE | `env=dev`, `owner=platform` |
| `storage.googleapis.com/Bucket` | `siteops-evidence-dev` | `US` | ACTIVE | `env=dev` |
| `compute.googleapis.com/Instance` | `legacy-worker` | `us-east1-b` | TERMINATED | sin etiquetas |
| `pubsub.googleapis.com/Topic` | `finding-events` | `global` | ACTIVE | `owner=app-team` |

Y estos eventos ficticios:

| Time | Resource type | Severity | Message |
|---|---|---|---|
| 14:01 | `cloud_run_revision` | INFO | `GET /findings 200 in 180 ms` |
| 14:04 | `cloud_run_revision` | WARNING | `PostgreSQL pool utilization reached 90%` |
| 14:05 | `cloud_run_revision` | ERROR | `POST /findings failed: connection timeout` |
| 14:06 | `cloudsql_database` | WARNING | `Active connections near configured limit` |

Completa sin Console:

1. Predice qué activos devuelve `NOT labels.env:* OR NOT labels.owner:*`.
2. Escribe una consulta para encontrar solo activos que sí tengan `env`.
3. Escribe una consulta de Logging para `cloud_run_revision` con `WARNING` o superior.
4. Formula una hipótesis que conecte los eventos, usando “podría” y no “demuestra”.
5. Indica dos datos necesarios para confirmar la causa.
6. Redacta un prompt para Cloud Assist que no permita cambios.
7. Explica qué consultarías en la API específica si necesitas el estado actual de `legacy-worker`.

Resultado conceptual esperado:

- La consulta de etiquetas ausentes incluye `siteops-evidence-dev`, `legacy-worker` y `finding-events`.
- La consulta positiva es `labels.env:*`.
- La consulta de logs puede ser `resource.type = "cloud_run_revision" AND severity >= "WARNING"`.
- La hipótesis razonable es que el agotamiento del pool o del límite de conexiones podría contribuir al timeout; la secuencia no prueba causalidad.
- Conviene comprobar métricas de conexiones, configuración del pool, latencia y errores de Cloud SQL, despliegues recientes y trazas.
- Para el estado en vivo de la VM se consulta Compute Engine, no se confía únicamente en el inventario.

## 13. Resultado esperado

### Si ejecutaste la práctica

Debes poder mostrar:

- proyecto, cuenta y scope verificados;
- inventario inicial sin filtro;
- resultados de la consulta de etiquetas;
- una búsqueda de logs en Console y su equivalente en CLI;
- una métrica existente inspeccionada o la explicación de por qué no había datos;
- una hipótesis limitada por evidencia;
- validación independiente de la respuesta de Cloud Assist, si estaba disponible;
- confirmación de que no se crearon recursos de cómputo, almacenamiento ni red.

### Si realizaste la alternativa conceptual

Debes entregar las siete respuestas de la sección 12.9, una tabla que relacione pregunta con herramienta y la lista de verificación del prompt. La alternativa cubre íntegramente la decisión técnica, aunque no sustituye la experiencia de permisos, Console y CLI.

## 14. Solución de problemas

| Síntoma | Causa probable | Diagnóstico | Acción segura |
|---|---|---|---|
| `API ... not enabled` | Cloud Asset API deshabilitada | Revisa `gcloud services list --enabled` | Pide autorización o usa `gcloud services enable cloudasset.googleapis.com --project=...`. |
| `PERMISSION_DENIED` en Asset Inventory | Falta permiso de búsqueda o uso del servicio | Confirma identidad, scope y roles | Solicita `roles/cloudasset.viewer` y `roles/serviceusage.serviceUsageConsumer` en el alcance necesario. |
| La consulta devuelve cero activos | Scope incorrecto, proyecto vacío, filtro estricto o indexación | Ejecuta sin `--query`; verifica proyecto y Console | Refina desde una búsqueda amplia; espera si el recurso acaba de cambiar. |
| El estado del inventario contradice Console | Consistencia eventual o fuentes con tiempos distintos | Registra timestamps y consulta API del servicio | Usa la API específica para la decisión en vivo. |
| `Invalid read mask` | Campo no admitido o mal escrito | Compara con la referencia `ResourceSearchResult` | Usa campos válidos: `name`, `assetType`, `project`, `displayName`, `location`, `labels`, `tags`, `createTime`, `updateTime`, `state`. |
| No aparecen logs | Ventana, filtro, permisos o ausencia de telemetría | Quita el filtro de recurso y amplía ventana | Confirma `roles/logging.viewer`; no fabriques errores en producción. |
| La consulta de Logging falla | Sintaxis o comillas alteradas por shell | Revisa el filtro como un solo argumento | Usa comillas simples externas y dobles dentro del filtro. |
| No aparece Cloud Assist | No está configurado, falta rol o no está disponible | Revisa Admin for Gemini y permisos con un administrador | Usa la alternativa conceptual; no evadas restricciones. |
| Cloud Assist inventa un activo | Salida generativa incorrecta o contexto equivocado | Ejecuta la consulta equivalente y revisa scope | Marca la afirmación como refutada; no actúes sobre ella. |
| Cloud Assist no puede analizar métricas | Monitoring API o `roles/monitoring.viewer` ausente | Revisa requisitos de la función | Solicita configuración mínima o inspecciona Metrics Explorer directamente. |
| Configuración de Cloud Assist falla dentro de VPC Service Controls | Limitación documentada del flujo de Console | Verifica si el proyecto está en un perímetro | Un administrador debe seguir la ruta CLI documentada y excluir la API no compatible indicada; no improvises. |

### Diagnóstico en orden

1. identidad;
2. proyecto y scope;
3. API habilitada;
4. permisos;
5. existencia de datos;
6. sintaxis y ventana temporal;
7. consistencia o retraso de indexación;
8. cuota o limitación del producto.

## 15. Impacto en costos y limpieza

### 15.1 Cloud Asset Inventory

La página oficial vigente indica que el uso de Cloud Asset Inventory es sin costo. Sí pueden generar cargos los destinos de una exportación, como Cloud Storage o BigQuery. Esta práctica no exporta datos ni crea feeds.

### 15.2 Google Cloud Observability

Los cargos dependen del volumen y uso. En la página de precios consultada el 7 de octubre de 2026:

- Cloud Logging cobra por almacenamiento que supere las asignaciones aplicables; la consulta y el análisis de logs almacenados no tienen un cargo adicional de Logging;
- retener ciertos logs más allá del periodo predeterminado puede generar costo;
- Monitoring puede cobrar por métricas facturables ingeridas y por determinadas lecturas de la API después de la asignación gratuita;
- Trace se cobra por spans ingeridos después de su asignación gratuita.

La práctica solo lee un volumen pequeño. No crees sinks, buckets, métricas personalizadas, checks o alertas para completarla. Verifica siempre la página de precios actual, porque tarifas y asignaciones pueden cambiar.

### 15.3 Gemini Cloud Assist

La documentación de configuración consultada presenta la oferta Preview y un flujo para habilitar Cloud Assist “at no cost”. Otras funciones de Gemini for Google Cloud, soporte o ediciones empresariales pueden tener condiciones distintas. Consulta la página de precios oficial y no uses la respuesta del chat como fuente de precios.

### 15.4 Limpieza

Esta práctica no debe crear recursos facturables. Por tanto:

- no borres activos preexistentes;
- no deshabilites `cloudasset.googleapis.com` en un proyecto compartido: otro proceso puede depender de ella;
- cierra Cloud Shell si terminaste;
- elimina solo archivos locales de notas que contengan datos sensibles; idealmente no los generes;
- deja intactos dashboards, alertas, sinks y políticas.

Puedes limpiar la variable de la sesión:

```bash
unset PROJECT_ID
```

Registro:

```text
Recursos creados: ninguno / detallar ________________________________
APIs habilitadas: ninguna / cloudasset.googleapis.com
Recursos eliminados: ninguno
Elementos preexistentes modificados: ninguno
Costo observado o estimado: ________________________________________
Verificación final: _________________________________________________
```

## 16. Repaso activo espaciado

Responde sin mirar las lecciones anteriores. Después comprueba y corrige.

### Día hábil anterior - Lección 12: red inicial, regiones, zonas y disponibilidad

1. ¿Cuál es el alcance de una VPC y cuál el de una subnet?
2. ¿Por qué una VPC global no vuelve multirregional a una aplicación?
3. Ordena estas comprobaciones: disponibilidad del producto, cuota y capacidad.
4. Para SiteOps Tracker, ¿qué requisito duro podría descartar una región cercana?

### Dos días hábiles antes - Lección 11: presupuestos, alertas y exportación

1. ¿Por qué un budget no detiene automáticamente el gasto?
2. ¿Qué destino permite analizar exportaciones de facturación con SQL?
3. ¿Qué diferencia hay entre umbral real y forecasted?
4. Relaciona una anomalía de costo con el inventario: ¿qué consulta harías primero?

### Tres días hábiles antes - Lección 10: cuentas de facturación y vínculo

1. ¿Qué relación existe entre proyecto y Cloud Billing account?
2. ¿Administrar un proyecto implica poder vincularlo a cualquier cuenta de facturación?
3. ¿Qué dos lados del vínculo requieren permisos?
4. ¿Qué ocurre con recursos facturables si el proyecto pierde una cuenta de facturación válida?

### Hace 7 días - Lección 8: IAM inicial, miembros, Cloud Identity, usuarios y grupos

1. Explica principal, role y policy con una frase cada uno.
2. ¿Por qué es preferible asignar un rol a un grupo estable y no a muchas personas individualmente?
3. ¿Qué rol mínimo de Asset Inventory pedirías para esta práctica y qué rol adicional aporta `serviceusage.services.use`?
4. ¿Por qué Cloud Assist no debe recibir más permisos que su usuario necesita para el análisis?

### Hace 21 días - línea base previa al calendario

El plan comenzó el 21 de septiembre; el 16 de septiembre no había una lección programada. Usa esta recuperación de línea base:

1. Define con tus palabras cloud, project, service y resource.
2. Distingue configuración, operación y acceso/seguridad.
3. Da un ejemplo de evidencia práctica que sea más fuerte que “leí el tema”.

### Refuerzo breve de confusiones acumulativas

- **API enabled ≠ permission granted ≠ quota available ≠ data present.**
- **Inventory ≠ monitoring.** El primero responde qué existe; el segundo, cómo se comporta.
- **Global scope ≠ global workload availability.**
- **Budget alert ≠ spending cap.**
- **Gemini response ≠ verified fact.**
- **Temporal correlation ≠ proven root cause.**

## 17. Ficha de práctica

```text
Fecha y tiempo dedicado: ____________________________________________
Modalidad: proyecto real / laboratorio / alternativa conceptual
Proyecto o dataset ficticio: _______________________________________

Inventario inicial:
- Scope: ____________________________________________________________
- Tipos de activo observados: ______________________________________
- Número de filas visibles: ________________________________________

Consulta de gobierno:
____________________________________________________________________
Resultado e interpretación:
____________________________________________________________________

Consulta de observabilidad:
____________________________________________________________________
Resultado e interpretación:
____________________________________________________________________

Hipótesis:
____________________________________________________________________
Evidencia que falta:
____________________________________________________________________

Cloud Assist:
- Prompt usado: ____________________________________________________
- Afirmaciones confirmadas: ________________________________________
- Afirmaciones refutadas/no comprobadas: ___________________________

Limpieza y costo:
____________________________________________________________________

Puntuación de preguntas: ____ / 10
Duda principal: ____________________________________________________
Siguiente acción concreta: _________________________________________
```

## 18. Ficha de errores

Completa una fila por respuesta fallada o inferencia no sustentada.

| Pregunta o paso | Mi respuesta/acción | Evidencia que la refuta | Tipo de error | Regla de decisión corregida | Nueva explicación sin apuntes | Revisión +1 día | +7 días | +21 días |
|---|---|---|---|---|---|---|---|---|
|  |  |  | concepto / lectura / comando / alcance / IAM / costo |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |

Tipos útiles:

- confundí inventario con telemetría;
- elegí una vista eventual para una decisión en vivo;
- ignoré scope o proyecto;
- confundí API habilitada con permiso;
- acepté una salida generativa sin validarla;
- elegí la señal equivocada: metric, log o trace;
- atribuí causalidad a una coincidencia temporal;
- olvidé costo, reversibilidad o limpieza.

## 19. Criterios de autoevaluación

Marca una opción por criterio:

| Criterio | Aún no | Con apoyo | Sin apoyo y con evidencia |
|---|:---:|:---:|:---:|
| Distingo inventario, métricas, logs y trazas | ☐ | ☐ | ☐ |
| Selecciono el scope correcto | ☐ | ☐ | ☐ |
| Ejecuto una búsqueda inicial de activos | ☐ | ☐ | ☐ |
| Escribo una query de etiquetas con `NOT` y `*` | ☐ | ☐ | ☐ |
| Explico consistencia eventual y cuándo usar la API del servicio | ☐ | ☐ | ☐ |
| Escribo un filtro de Logging con severidad | ☐ | ☐ | ☐ |
| Formulo una hipótesis sin confundirla con una conclusión | ☐ | ☐ | ☐ |
| Diseño un prompt seguro y verificable | ☐ | ☐ | ☐ |
| Valido una salida de Cloud Assist | ☐ | ☐ | ☐ |
| Explico costos y confirmo que no creé recursos | ☐ | ☐ | ☐ |

Meta personal para avanzar: al menos 8 de 10 preguntas correctas y evidencia “sin apoyo” en los criterios esenciales. Es una meta de estudio, no un umbral oficial del examen. Si no la alcanzas, registra los errores y repite las consultas; no marques el tema como dominado por haber recibido el archivo.

## 20. Documentación oficial verificada

Consultada el 7 de octubre de 2026:

- [Cloud Asset Inventory overview](https://docs.cloud.google.com/asset-inventory/docs/asset-inventory-overview)
- [Search for resources](https://docs.cloud.google.com/asset-inventory/docs/searching-resources)
- [Search query syntax](https://docs.cloud.google.com/asset-inventory/docs/search-query-syntax)
- [`gcloud asset search-all-resources`](https://cloud.google.com/sdk/gcloud/reference/asset/search-all-resources)
- [`gcloud asset list`](https://cloud.google.com/sdk/gcloud/reference/asset/list)
- [`gcloud auth list`](https://cloud.google.com/sdk/gcloud/reference/auth/list)
- [`gcloud config set`](https://cloud.google.com/sdk/gcloud/reference/config/set)
- [`gcloud config list`](https://cloud.google.com/sdk/gcloud/reference/config/list)
- [`gcloud services list`](https://cloud.google.com/sdk/gcloud/reference/services/list)
- [`gcloud services enable`](https://cloud.google.com/sdk/gcloud/reference/services/enable)
- [Cloud Asset Inventory roles and permissions](https://docs.cloud.google.com/asset-inventory/docs/roles-permissions)
- [Cloud Asset Inventory pricing](https://cloud.google.com/asset-inventory/pricing)
- [Google Cloud Observability overview](https://cloud.google.com/stackdriver/docs)
- [Logging query language](https://cloud.google.com/logging/docs/view/logging-query-language)
- [`gcloud logging read`](https://cloud.google.com/sdk/gcloud/reference/logging/read)
- [Google Cloud Observability pricing](https://cloud.google.com/products/observability/pricing)
- [Gemini Cloud Assist overview](https://docs.cloud.google.com/cloud-assist/overview)
- [Set up Gemini Cloud Assist](https://docs.cloud.google.com/cloud-assist/set-up-gemini)
- [Use Gemini Cloud Assist in Console](https://docs.cloud.google.com/cloud-assist/chat-panel)
- [How Gemini products in Google Cloud use your data](https://docs.cloud.google.com/gemini/docs/discover/data-governance)
- [Google Cloud with Gemini and responsible AI](https://docs.cloud.google.com/gemini/docs/discover/responsible-ai)
- [Gemini for Google Cloud pricing](https://cloud.google.com/products/gemini/pricing)
- [Associate Cloud Engineer certification](https://cloud.google.com/learn/certification/cloud-engineer)

## 21. Preguntas de examen originales

Las preguntas son inéditas, basadas en escenarios y no proceden de dumps. Responde antes de leer la sección final.

### Question 1

SiteOps Tracker's operations team needs a cross-service list of resources that don't have an `owner` label. The list can tolerate a short indexing delay. Which service should they use?

A. Cloud Trace  
B. Cloud Asset Inventory  
C. Cloud Monitoring  
D. Cloud Logging

### Question 2

An engineer must determine where a single user request spent most of its time across a Node.js API and a PostgreSQL call. Which signal is the most direct choice?

A. An asset export  
B. A billing report  
C. A distributed trace and its spans  
D. An organization policy

### Question 3

An automation will immediately stop production VMs that are reported as idle. Missing or stale state could cause an outage. What should the engineer do?

A. Use only Cloud Asset Inventory because it provides one global inventory.  
B. Ask Gemini Cloud Assist and execute its first command automatically.  
C. Query the Compute Engine API for live state and separately use telemetry to prove the idle condition.  
D. Search Cloud Logging for the VM name and assume no results means the VM is idle.

### Question 4

Which command correctly searches up to 25 resources in project `siteops-lab` and returns selected metadata fields?

A. `gcloud asset search-all-resources --scope="projects/siteops-lab" --limit=25 --read-mask="name,assetType,location,state"`  
B. `gcloud asset search-all-resources --project="projects/siteops-lab" --max-results=25 --fields="name,type,zone,status"`  
C. `gcloud resources list --organization="siteops-lab" --limit=25`  
D. `gcloud asset list --scope="projects/siteops-lab" --query="*" --read-mask="name"`

### Question 5

The team wants the most recent warning and error log entries from Cloud Run revisions in a project. Which Logging filter is appropriate?

A. `assetType = "run.googleapis.com/Service" AND state = "WARNING"`  
B. `resource.type = "cloud_run_revision" AND severity >= "WARNING"`  
C. `metric.type = "cloud_run_revision" OR severity = "WARNING"`  
D. `labels.service:* AND logLevel < "WARNING"`

### Question 6

You ask Gemini Cloud Assist to identify unlabeled resources. It reports a Cloud SQL instance that doesn't appear in your Asset Inventory results. What should you do next?

A. Delete the reported instance to remove the inconsistency.  
B. Grant Cloud Assist Owner so it can correct the inventory.  
C. Treat the response as a hypothesis, verify project and scope, and run a deterministic asset or service query.  
D. Assume Cloud Asset Inventory is always wrong because it is eventually consistent.

### Question 7

`gcloud asset search-all-resources` returns no rows immediately after a resource was created. Select two actions that are appropriate.

A. Verify the account, project scope, API, permissions, and query.  
B. Conclude that the resource creation failed and recreate it repeatedly.  
C. Allow for indexing delay and query the service-specific API if live state is required.  
D. Grant the querying user Project Owner permanently.

### Question 8

Which statement best describes the cost impact of today's read-only practice?

A. Every Cloud Asset Inventory search is billed per returned asset.  
B. Cloud Asset Inventory use is free, but exported data destinations and observability ingestion or retention can generate charges.  
C. Cloud Logging queries always double the storage charge.  
D. Enabling any API automatically creates a billable VM.

### Question 9

An analyst needs to search resource metadata but must not modify assets. Which predefined access pattern follows least privilege most closely?

A. Project Owner only  
B. Editor plus Billing Account Administrator  
C. Cloud Asset Viewer plus Service Usage Consumer at the necessary scope  
D. Organization Administrator plus Security Admin

### Question 10

SiteOps Tracker shows a latency spike at 14:05. Asset history shows a configuration change at 14:00, and logs show database connection warnings at 14:04. What is the best next step?

A. State that the configuration change caused the incident and close the investigation.  
B. Delete the changed resource to restore service quickly.  
C. Correlate metrics, logs, traces, the exact change, and current service state before choosing a reversible remediation.  
D. Use only the Asset Inventory result because it is the most centralized source.

## 22. Soluciones justificadas

### 1. Correct answer: B

**Why:** Cloud Asset Inventory is designed to search cross-service resource metadata, including labels, and the scenario tolerates eventual consistency.

- **A is wrong:** Trace analyzes request paths and latency, not a resource inventory.
- **C is wrong:** Monitoring stores time-series telemetry, not a normalized list of assets and labels.
- **D is wrong:** Logging records events; it is not the authoritative cross-service metadata search requested.

### 2. Correct answer: C

**Why:** A distributed trace decomposes one request into spans, making per-component latency visible.

- **A is wrong:** An asset export describes infrastructure metadata, not the request path.
- **B is wrong:** Billing data explains consumption and cost, not latency within a request.
- **D is wrong:** Organization policy constrains resources; it doesn't measure execution time.

### 3. Correct answer: C

**Why:** The automation needs live service state and evidence of idleness. The Compute Engine API supplies current VM state; metrics establish utilization. This avoids making a production decision from an eventually consistent inventory.

- **A is wrong:** Asset Inventory is useful for discovery but explicitly not intended as the sole source when stale data could break availability.
- **B is wrong:** Generative output must be reviewed and must not trigger destructive actions automatically.
- **D is wrong:** Absence of matching logs does not prove idleness; collection, filtering, or permissions may explain the absence.

### 4. Correct answer: A

**Why:** `search-all-resources` accepts `--scope`, `--limit`, and `--read-mask`; a project scope has the form `projects/PROJECT_ID`.

- **B is wrong:** The command does not use `--project` as a substitute for the search scope, nor `--max-results` or `--fields` with those meanings.
- **C is wrong:** `gcloud resources list` is not the documented Cloud Asset Inventory command.
- **D is wrong:** `gcloud asset list` uses exactly one of `--project`, `--folder`, or `--organization`; it is not invoked with this search syntax.

### 5. Correct answer: B

**Why:** Cloud Logging filters on monitored-resource fields such as `resource.type` and on `severity`; `>= "WARNING"` includes warnings and more severe entries.

- **A is wrong:** `assetType` and `state` belong to an inventory-style question, not the Logging query shown.
- **C is wrong:** `metric.type` is not the field for selecting Cloud Run log resources, and `OR` would broaden the wrong conditions.
- **D is wrong:** It uses an unspecified label and reverses the severity requirement.

### 6. Correct answer: C

**Why:** Cloud Assist output is a hypothesis. Verify the selected project, scope, permissions and timing, then use a deterministic query or the Cloud SQL API.

- **A is wrong:** Deleting an unverified resource is destructive and unsupported by evidence.
- **B is wrong:** Owner violates least privilege and doesn't make generative output inherently accurate.
- **D is wrong:** Eventual consistency is a limitation to consider, not proof that every inventory result is wrong.

### 7. Correct answers: A and C

**Why:** First verify identity, scope, API, permissions and syntax. Then account for indexing delay; if the decision requires live state, query the service-specific API.

- **B is wrong:** Repeated recreation can cause duplicates, cost and additional confusion.
- **D is wrong:** Permanent Owner is unnecessary and violates least privilege.

### 8. Correct answer: B

**Why:** Cloud Asset Inventory itself is offered without charge, while exports can incur destination costs. Logging, Monitoring and Trace can incur ingestion, storage, retention, read or usage charges depending on the signal and volume.

- **A is wrong:** The pricing page does not charge each search per returned asset.
- **C is wrong:** Querying stored logs does not automatically double storage charges.
- **D is wrong:** Enabling an API does not create a VM.

### 9. Correct answer: C

**Why:** Cloud Asset Viewer supplies read access to asset metadata, and Service Usage Consumer supplies the service-use permission required by Cloud Asset Inventory calls. Scope the roles only where needed.

- **A is wrong:** Owner is far broader than read-only inventory access.
- **B is wrong:** Editor and billing administration add unrelated mutation and financial privileges.
- **D is wrong:** Organization and security administration are excessive for this task.

### 10. Correct answer: C

**Why:** The timestamps create a useful hypothesis, not a proven cause. Correlating the exact change with metrics, logs, traces and live state narrows the diagnosis before a reversible remediation.

- **A is wrong:** Temporal proximity alone does not prove causality.
- **B is wrong:** Deletion is destructive and premature.
- **D is wrong:** Asset Inventory gives valuable configuration context but not all operational signals or live state.
