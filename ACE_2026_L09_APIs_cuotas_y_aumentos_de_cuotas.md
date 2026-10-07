# Google Cloud Associate Cloud Engineer (ACE) 2026

## Lección 09 - APIs, cuotas y aumentos de cuotas

**Fecha del plan:** 1 de octubre de 2026  
**Dominio de la guía oficial:** 1.1 - Setting up cloud projects and accounts  
**Duración sugerida:** 75-90 minutos  
**Práctica del plan:** habilitar una API en un laboratorio y localizar una cuota y su flujo de aumento  
**Caso transversal:** SiteOps Tracker, proyecto ficticio de portafolio  
**Documentación y comandos verificados:** 1 de octubre de 2026

> Esta lección enseña y propone evidencia verificable. Recibirla no demuestra dominio: el aprendizaje se comprueba al realizar la práctica, explicar las decisiones sin apuntes y corregir los errores.

---

## 1. Objetivo de aprendizaje

Al terminar deberías poder:

1. Explicar qué significa habilitar una API en un proyecto de Google Cloud.
2. Separar cuatro controles que suelen confundirse: API habilitada, IAM, cuota y facturación.
3. Distinguir una cuota ajustable de un límite del sistema.
4. Reconocer cuotas de tasa, de asignación y con dimensiones regionales o globales.
5. Habilitar de forma segura la **Cloud Quotas API** en un proyecto temporal.
6. Localizar una cuota en Google Cloud Console y con Google Cloud CLI.
7. Recorrer el flujo de solicitud de ajuste sin enviar una petición real.
8. Explicar los permisos, riesgos de costo y limpieza correspondientes.

### Evidencia que debes producir

Completa en este mismo archivo, al final de la práctica:

- ID del proyecto temporal, sin secretos.
- Estado de la API antes y después.
- Nombre del servicio: **cloudquotas.googleapis.com**.
- Una cuota observada: nombre, identificador, valor, dimensión y uso actual si aparece.
- Captura o transcripción sin datos sensibles de la pantalla o salida de consulta.
- Explicación de tres frases: por qué habilitar una API no concede acceso, por qué una cuota no es un presupuesto y por qué un aumento no está garantizado.
- Resultado de las 10 preguntas.
- Una ficha por cada error.

---

## 2. Prerrequisitos explicados desde cero

### 2.1 Proyecto de Google Cloud

Un **project** es el contenedor administrativo en el que se habilitan servicios, se aplican permisos, se acumulan muchas cuotas y se asocia el consumo. La misma API puede estar habilitada en un proyecto y deshabilitada en otro.

Para esta práctica usa uno de estos entornos, en este orden:

1. Proyecto temporal proporcionado por Google Skills.
2. Proyecto personal creado solo para laboratorio.
3. Alternativa conceptual completa de la sección 16.

No practiques cambios en un proyecto compartido, productivo o de otra persona.

### 2.2 Google Cloud Console y Cloud Shell

- **Google Cloud Console** es la interfaz web.
- **Cloud Shell** es una terminal administrada que ya incluye Google Cloud CLI.
- **gcloud** es la herramienta de línea de comandos principal.

Console y CLI son dos interfaces del mismo plano de control. Un cambio realizado por una debe ser observable desde la otra.

### 2.3 Identidad activa

La cuenta autenticada es quien intenta habilitar la API o consultar cuotas. Seleccionar un proyecto no concede permisos sobre él. Antes de cualquier cambio debes verificar:

- cuenta activa;
- proyecto activo;
- que el proyecto sea realmente de laboratorio;
- autorización para realizar la práctica.

### 2.4 Permisos mínimos

Para habilitar o deshabilitar servicios se necesita, entre otros, el permiso **serviceusage.services.enable** o **serviceusage.services.disable**. El rol predefinido **Service Usage Admin** (**roles/serviceusage.serviceUsageAdmin**) contiene esos permisos.

Para consultar cuotas intervienen permisos como:

- **resourcemanager.projects.get**;
- **monitoring.timeSeries.list**;
- **serviceusage.services.list**;
- **cloudquotas.quotas.get**.

Para cambiar cuotas se requieren:

- **serviceusage.quotas.update**;
- **cloudquotas.quotas.update**.

El rol **Cloud Quotas Viewer** (**roles/cloudquotas.viewer**) sirve para consulta. Para ajustes, **Cloud Quotas Admin** (**roles/cloudquotas.admin**) es más específico que conceder Owner. La selección exacta depende del alcance y la tarea. En el examen, evita roles básicos amplios cuando existe un rol predefinido más limitado.

### 2.5 Facturación y crédito

La práctica no crea máquinas, bases de datos ni buckets. Habilitar una API, por sí solo, normalmente no crea un recurso facturable. Sin embargo:

- el uso posterior de un servicio sí puede generar cargos;
- una cuota más alta permite potencialmente más consumo;
- algunos servicios exigen una cuenta de facturación habilitada;
- una solicitud de aumento puede requerir revisión y, en ciertos casos, pago anticipado.

Si el laboratorio no permite facturación o cambios de cuota, realiza las consultas y el recorrido conceptual sin enviar solicitudes.

### 2.6 Herramientas y versión

En Cloud Shell, gcloud ya está instalado. En un equipo local, usa una versión actual y comprueba:

~~~bash
gcloud version
~~~

Los comandos **gcloud quotas** utilizados en esta lección aparecen en la documentación oficial vigente. Si tu instalación antigua no reconoce el grupo, actualiza Google Cloud CLI o usa Cloud Shell.

---

## 3. Analogía: un edificio con servicios, llaves y aforo

Imagina que SiteOps Tracker ocupa una oficina:

- **Habilitar una API** es pedir que se conecte un servicio del edificio, como la electricidad del taller.
- **IAM** es entregar llaves a personas o aplicaciones autorizadas.
- **Quota** es el aforo o la capacidad autorizada: por ejemplo, cuántas operaciones por minuto o cuántos recursos puede solicitar la oficina.
- **System limit** es una restricción estructural no negociable, como el peso máximo que soporta un ascensor.
- **Billing** es la cuenta que paga el consumo.
- **Budget alert** es una alarma de gasto, no un interruptor automático.

Conectar la electricidad no entrega una llave. Tener una llave no aumenta el aforo. Elevar el aforo no paga la factura. Una alarma de gasto no limita necesariamente el consumo.

Esta separación es una regla central para ACE.

---

## 4. Modelo mental: cuatro puertas independientes

Para que una llamada a un servicio funcione, varias condiciones pueden ser necesarias:

| Control | Pregunta | Ejemplo de fallo | Solución típica |
|---|---|---|---|
| Service enablement | ¿La API está habilitada en el proyecto consumidor? | “API has not been used” o “service disabled” | Habilitar el servicio correcto |
| Authentication e IAM | ¿Quién llama y qué permisos tiene? | PERMISSION_DENIED | Autenticar y conceder el rol mínimo |
| Quota o límite | ¿Queda capacidad para esa métrica y dimensión? | 429, RATE_LIMIT_EXCEEDED o QUOTA_EXCEEDED | Reducir demanda, reintentar o solicitar ajuste |
| Billing y elegibilidad | ¿El servicio requiere facturación, términos o aprobación? | Billing disabled o prerequisite failed | Vincular facturación o cumplir el prerrequisito |

Una respuesta de examen suele fallar cuando resuelve la puerta equivocada.

### Ejemplo de SiteOps Tracker

El backend Node.js consulta recursos para mostrar un inventario ficticio de sedes.

1. El proyecto debe tener habilitada la API correspondiente.
2. La identidad de la carga debe tener permisos de solo lectura.
3. Las llamadas deben mantenerse dentro de la cuota.
4. Si el producto es facturable, el proyecto debe tener una configuración válida.

Dar Owner al backend no arregla una API deshabilitada ni una cuota agotada. Aumentar la cuota tampoco arregla un rol IAM ausente.

---

## 5. Qué es una API y qué significa habilitarla

Una **Application Programming Interface** expone operaciones que software, consola y CLI pueden invocar. El nombre visible del producto y el identificador técnico del servicio no siempre son iguales.

Ejemplos:

| Producto | Identificador de servicio |
|---|---|
| Cloud Quotas API | cloudquotas.googleapis.com |
| Compute Engine API | compute.googleapis.com |
| Cloud Run Admin API | run.googleapis.com |
| Artifact Registry API | artifactregistry.googleapis.com |

Habilitar un servicio registra que ese proyecto puede consumirlo. No significa:

- crear automáticamente todos sus recursos;
- autorizar a cualquier principal;
- eliminar la necesidad de autenticación;
- asegurar capacidad física;
- aumentar todas sus cuotas;
- hacer gratuito su uso.

### Proyecto consumidor

La habilitación pertenece al **consumer project**. Una aplicación puede leer un recurso en un proyecto y contabilizar ciertas llamadas contra otro proyecto de cuota. Por eso, ante un error, debes identificar:

- proyecto que contiene el recurso;
- proyecto en el que está habilitada la API;
- proyecto al que se atribuye la cuota de la solicitud;
- identidad que realiza la llamada.

No asumas que siempre son el mismo.

---

## 6. Qué son las cuotas

Una **quota** es un valor que limita el consumo de un recurso o la frecuencia de operaciones. Google Cloud las usa para proteger la capacidad compartida, prevenir picos accidentales y ayudar a gestionar el crecimiento.

### 6.1 Cuotas de tasa

Limitan operaciones durante un intervalo:

- solicitudes por minuto;
- escrituras por minuto;
- consultas por día;
- solicitudes por usuario.

Si se alcanza una cuota de tasa, esperar al reinicio de la ventana, aplicar reintentos con backoff y reducir llamadas puede ser mejor que pedir un aumento.

### 6.2 Cuotas de asignación

Limitan la cantidad de recursos que puede existir o reservarse:

- CPU virtuales por región;
- direcciones IP;
- discos;
- aceleradores.

Estas cuotas no se “reinician por minuto”. Debes liberar recursos, elegir otra región o solicitar capacidad adicional.

### 6.3 Dimensiones

Una cuota puede variar por:

- proyecto;
- región;
- zona;
- usuario;
- familia de GPU;
- red u otra dimensión específica del servicio.

Decir “el proyecto tiene cuota” es incompleto. Debes preguntar **para qué métrica, en qué ubicación y con qué dimensión**.

### 6.4 Valor predeterminado, concedido y preferido

- **Default value:** valor inicial definido por el servicio.
- **Granted value:** valor concedido actualmente.
- **Preferred value:** valor solicitado mediante una preferencia de cuota.
- **Current usage:** consumo observado, cuyo cálculo depende del tipo de cuota.

Una preferencia no equivale a aprobación. Mientras una solicitud se reconcilia, el valor concedido puede seguir siendo menor.

### 6.5 Cuota frente a límite del sistema

| Concepto | ¿Suele poder ajustarse? | Ejemplo mental |
|---|---:|---|
| Quota | A veces, mediante solicitud | Capacidad autorizada para un proyecto |
| System limit | Normalmente no | Restricción técnica fija del producto |

Antes de buscar el botón de aumento, confirma si la fila es realmente una cuota ajustable.

---

## 7. El flujo de un aumento de cuota

Una solicitud responsable sigue este razonamiento:

1. **Identificar el error exacto.** No toda falla de capacidad es una cuota.
2. **Identificar métrica y dimensión.** Servicio, quota ID, región y valor actual.
3. **Medir uso y tendencia.** Evitar solicitar un número arbitrario.
4. **Reducir consumo evitable.** Caché, batching, paginación, backoff, liberación de recursos.
5. **Evaluar alternativas.** Otra región, otra arquitectura o programación de carga.
6. **Estimar costo máximo.** Más capacidad puede permitir más gasto.
7. **Solicitar solo lo necesario.** Con justificación y contacto válidos.
8. **Esperar evaluación.** La aprobación no es automática ni inmediata.
9. **Verificar el valor concedido.** No desplegar suponiendo que ya cambió.
10. **Monitorear después.** Capacidad, errores y costos.

### Qué evalúa Google Cloud

Las solicitudes pueden pasar por revisión automatizada o humana. Entre los factores se encuentran disponibilidad de recursos, historial de uso y características de la petición. Los criterios completos no se publican y una petición puede ser rechazada.

### Cuándo no pedir aumento

No lo pidas como primera reacción cuando:

- un 429 se resuelve con backoff;
- el cliente envía solicitudes duplicadas;
- hay recursos olvidados que pueden eliminarse;
- la región equivocada está seleccionada;
- el error es IAM o API deshabilitada;
- se trata de un system limit;
- no se estimó el impacto financiero.

---

## 8. SiteOps Tracker: ejemplo paso a paso

### 8.1 Escenario

SiteOps Tracker es una aplicación ficticia con:

- React + TypeScript para la interfaz;
- Node.js + TypeScript para la API;
- PostgreSQL para activos, hallazgos, responsables e historial;
- servicios de Google Cloud para ejecución, observabilidad e inventario.

El equipo quiere consultar cuotas antes de ampliar el procesamiento nocturno de inventarios.

### 8.2 Diseño de control

| Necesidad | Decisión |
|---|---|
| Consultar metadatos de cuotas | Habilitar Cloud Quotas API en el proyecto de laboratorio |
| Leer cuotas | Identidad con permisos de lectura, no Owner |
| Pedir ajuste | Flujo separado, autorizado y con revisión |
| Evitar sobrecarga | Batch, caché y reintentos con exponential backoff |
| Evitar sorpresa de costo | Estimar consumo y configurar alertas de presupuesto aparte |
| Mantener historial | Registrar solicitud, motivo, valor anterior, valor pedido y resultado |

### 8.3 Historia de auditoría ficticia

La base de datos podría registrar:

| Campo | Ejemplo no identificable |
|---|---|
| event_type | QUOTA_REVIEWED |
| service | cloudquotas.googleapis.com |
| quota_id | valor observado en laboratorio |
| old_value | valor observado |
| requested_value | vacío, porque hoy no se envía solicitud |
| status | INSPECTED_ONLY |
| responsible_role | platform-operator |
| timestamp | fecha y hora del laboratorio |

No guardes tokens, credenciales ni información personal innecesaria.

---

## 9. Decisiones entre servicios

| Necesidad | Elige | Por qué | Por qué no las alternativas |
|---|---|---|---|
| Habilitar o deshabilitar una API | Service Usage | Gestiona el estado de consumo de servicios por proyecto | Service Management se orienta a administrar/configurar servicios publicados, no a habilitar su consumo |
| Consultar y solicitar ajustes de cuotas | Cloud Quotas | Expone QuotaInfo y QuotaPreference | IAM controla permisos; no cambia capacidad |
| Limitar gasto | Budgets and alerts, más controles técnicos | Advierte sobre gasto y apoya gobierno financiero | Una cuota técnica no equivale a presupuesto; un presupuesto no suele detener recursos |
| Autorizar a una persona o workload | IAM | Responde quién puede hacer qué y dónde | Habilitar la API no concede roles |
| Alertar por consumo cercano al límite | Cloud Monitoring y quota alerts | Observa tendencia y dispara alertas | Aumentar cuota no crea alerta |
| Exponer una API propia a clientes | API Gateway o Apigee, según requisitos | Gestiona entrada, autenticación, políticas y ciclo de una API propia | Habilitar una Google Cloud API no publica la API de tu aplicación |

### Regla de examen

Busca el verbo:

- **enable** → Service Usage;
- **authorize** → IAM;
- **increase capacity** → quota adjustment;
- **control cost** → billing controls y arquitectura;
- **observe threshold** → Monitoring;
- **publish/manage your own API** → API management.

---

## 10. Seguridad y mínimo privilegio

### 10.1 Para habilitar APIs

Prefiere **roles/serviceusage.serviceUsageAdmin** en el proyecto adecuado cuando la tarea exige administrar servicios. No concedas Owner solo por comodidad.

### 10.2 Para consultar cuotas

Prefiere permisos o un rol de consulta como **roles/cloudquotas.viewer**, junto con los permisos necesarios en el alcance.

### 10.3 Para solicitar ajustes

Separa la capacidad de lectura de la capacidad de cambio. **roles/cloudquotas.admin** es más específico que Owner, pero sigue siendo un rol sensible. Limita alcance y duración cuando sea posible.

### 10.4 Separación de funciones

Un diseño razonable para SiteOps Tracker:

- desarrolladores: ven cuotas relevantes;
- operador de plataforma: habilita servicios aprobados;
- responsable de costos: revisa impacto financiero;
- administrador de cuotas: presenta cambios autorizados;
- auditor: consulta historial sin poder modificarlo.

---

## 11. Práctica guiada segura - Google Cloud Console

### 11.1 Reglas antes de comenzar

- Usa un proyecto temporal.
- No envíes una solicitud real de aumento.
- No habilites productos distintos de Cloud Quotas API.
- No cambies cuotas ni overrides.
- Si la API ya estaba habilitada, registra ese estado y no la deshabilites al final.

### 11.2 Confirma el proyecto

1. Abre Google Cloud Console.
2. Revisa el selector de proyecto en la barra superior.
3. Confirma que el nombre e ID corresponden al laboratorio.
4. Anota el ID en “Registro de práctica”.

### 11.3 Registra el estado inicial

1. Ve a **APIs & Services > Enabled APIs & services**.
2. Busca **Cloud Quotas API**.
3. Registra uno de estos estados:
   - ya estaba habilitada;
   - no estaba habilitada;
   - no tienes permiso para comprobarlo.

### 11.4 Habilita Cloud Quotas API

Solo si no estaba habilitada:

1. Ve a **APIs & Services > API Library**.
2. Busca **Cloud Quotas API**.
3. Abre el resultado y confirma el identificador **cloudquotas.googleapis.com**.
4. Selecciona **Enable**.
5. Espera a que termine la operación.

No confundas Cloud Quotas API con un producto de facturación ni con Service Usage.

### 11.5 Verifica la habilitación

1. Regresa a **Enabled APIs & services**.
2. Localiza **Cloud Quotas API**.
3. Registra “ENABLED” y la hora aproximada.

### 11.6 Localiza una cuota

1. Ve a **IAM & Admin > Quotas & System Limits**.
2. Confirma de nuevo el proyecto.
3. Usa **Filter**.
4. Filtra **Service = Cloud Quotas API**.
5. Si no aparecen resultados, quita el filtro y busca “Cloud Quotas”; consulta también la sección de solución de problemas.
6. Elige una cuota visible, por ejemplo una de solicitudes de lectura o actualización.
7. Muestra la columna **Limit name** si está oculta.
8. Registra:
   - Service;
   - Quota;
   - Metric;
   - Limit name o quota ID;
   - Value;
   - Current usage, si aparece;
   - Dimensions o ubicación;
   - si es quota o system limit.

Los valores pueden variar. No copies un número de esta lección como si fuera el valor de tu proyecto.

### 11.7 Recorre el flujo de aumento sin enviar

1. Selecciona la casilla de una cuota ajustable.
2. Pulsa **Edit**.
3. Observa el valor actual, el campo **New value** y cualquier unidad.
4. Identifica si aparece **Apply for higher quota**.
5. No introduzcas un valor y no pulses **Submit request**.
6. Cierra el diálogo.
7. Abre la pestaña **Increase Requests** y comprueba que no generaste una petición.

Si el botón de edición no aparece, registra si se trata de falta de permisos, cuota no ajustable o system limit.

### 11.8 Evidencia de Console

Completa:

- Proyecto: ______________________________
- API antes: ENABLED / DISABLED / SIN PERMISO
- API después: ENABLED / SIN CAMBIO / SIN PERMISO
- Cuota elegida: _________________________
- Quota ID o Limit name: _________________
- Valor y dimensión: _____________________
- ¿Apareció Edit?: SÍ / NO
- ¿Se envió una solicitud?: **NO**

---

## 12. Práctica guiada segura - Cloud Shell y gcloud

Los comandos de esta sección fueron contrastados con documentación oficial vigente al 1 de octubre de 2026.

### 12.1 Verifica la cuenta

~~~bash
gcloud auth list --filter=status:ACTIVE
~~~

Resultado esperado: una sola cuenta activa o una cuenta claramente identificada como la del laboratorio.

### 12.2 Verifica el proyecto configurado

~~~bash
gcloud config get-value project
~~~

Asigna explícitamente el proyecto temporal. Sustituye el valor de ejemplo:

~~~bash
export PROJECT_ID="TU_PROJECT_ID_DE_LABORATORIO"
gcloud config set project "$PROJECT_ID"
gcloud projects describe "$PROJECT_ID"
~~~

No continúes si el proyecto no es el del laboratorio.

### 12.3 Lista servicios habilitados antes del cambio

~~~bash
gcloud services list --enabled --project="$PROJECT_ID"
~~~

Busca **cloudquotas.googleapis.com** en la salida y registra si ya estaba habilitada.

### 12.4 Habilita una API

Ejecuta solo en el proyecto temporal y solo si estaba deshabilitada:

~~~bash
gcloud services enable cloudquotas.googleapis.com \
  --project="$PROJECT_ID"
~~~

Resultado esperado:

~~~text
Operation finished successfully.
~~~

La redacción exacta puede variar.

### 12.5 Comprueba el estado

~~~bash
gcloud services list --enabled --project="$PROJECT_ID" \
  | grep -F "cloudquotas.googleapis.com"
~~~

Resultado esperado: una fila con el identificador y el título de Cloud Quotas API.

### 12.6 Lista información de cuotas

El siguiente comando usa como servicio consultado y como proyecto de cuota el laboratorio:

~~~bash
gcloud quotas info list \
  --service=cloudquotas.googleapis.com \
  --project="$PROJECT_ID" \
  --billing-project="$PROJECT_ID" \
  --limit=10
~~~

Busca en la salida:

- **quotaId**;
- **metric**;
- **dimensions**;
- **value** o información equivalente;
- ubicaciones aplicables, si las hay.

El flag **--billing-project** identifica el proyecto cuya cuota de la propia llamada a Cloud Quotas API se utiliza. Puede ser diferente del proyecto cuyos límites consultas. El nombre del flag no significa por sí solo que esta consulta cree un cargo.

### 12.7 Describe una cuota concreta

Copia exactamente un **quotaId** devuelto, sin inventarlo:

~~~bash
export QUOTA_ID="PEGA_AQUI_EL_QUOTA_ID_OBSERVADO"

gcloud quotas info describe "$QUOTA_ID" \
  --service=cloudquotas.googleapis.com \
  --project="$PROJECT_ID" \
  --billing-project="$PROJECT_ID"
~~~

Resultado esperado: metadatos de una cuota concreta, incluida su métrica y configuración.

### 12.8 Consulta preferencias existentes

Esto es de solo lectura:

~~~bash
gcloud quotas preferences list \
  --project="$PROJECT_ID" \
  --billing-project="$PROJECT_ID"
~~~

Para ver solo preferencias en reconciliación:

~~~bash
gcloud quotas preferences list \
  --project="$PROJECT_ID" \
  --billing-project="$PROJECT_ID" \
  --reconciling-only=true
~~~

Una lista vacía es válida: significa que no se encontraron preferencias con esos criterios.

### 12.9 Anatomía del comando de solicitud - no ejecutar hoy

La documentación oficial usa este patrón para crear una preferencia cuando aún no existe:

~~~bash
gcloud quotas preferences create \
  --project=PROJECT_ID_OR_NUMBER \
  --service=SERVICE_NAME \
  --quota-id=QUOTA_ID \
  --dimensions=DIMENSION_KEY=DIMENSION_VALUE \
  --preferred-value=PREFERRED_VALUE \
  --billing-project=BILLING_PROJECT_ID_OR_NUMBER \
  --email=CONTACT_EMAIL \
  --justification="BUSINESS_AND_TECHNICAL_REASON" \
  --preference-id=PREFERENCE_ID
~~~

**No lo ejecutes en esta práctica.** Crear una preferencia inicia un cambio real. Antes se necesita autorización, un valor derivado de demanda, dimensión exacta, estimación de costo y plan de seguimiento.

### 12.10 Guarda evidencia no sensible

Copia solo:

- el nombre de la cuenta o sustitúyelo por “[cuenta de laboratorio]”;
- el ID del proyecto de laboratorio;
- el nombre del servicio;
- un quota ID;
- valor y dimensión observados;
- resultado de consulta de preferencias.

No copies access tokens, cookies, claves, credenciales ni correos personales.

---

## 13. Resultado esperado

La práctica está completa si puedes demostrar:

1. Elegiste el proyecto correcto.
2. Determinaste el estado inicial de Cloud Quotas API.
3. La habilitaste solo si era necesario.
4. Confirmaste el estado con Console y CLI.
5. Localizaste una cuota real del proyecto.
6. Registraste quota ID, métrica, dimensión y valor.
7. Abriste el flujo de ajuste sin enviar una solicitud.
8. Explicaste qué permisos se necesitan.
9. Diferenciaste cuota, system limit, IAM, facturación y presupuesto.
10. Dejaste el entorno sin cambios innecesarios.

Una salida vacía, falta de permisos o restricción del laboratorio también puede ser evidencia válida si diagnosticas correctamente la causa y completas la alternativa conceptual.

---

## 14. Solución de problemas

### Caso 1 - “SERVICE_DISABLED” o la API no se ha usado

**Causa probable:** Cloud Quotas API no está habilitada en el proyecto consumidor.

**Diagnóstico:**

~~~bash
gcloud services list --enabled --project="$PROJECT_ID"
~~~

**Acción:** habilita **cloudquotas.googleapis.com** en el proyecto correcto. Si no tienes permiso, solicita el cambio al administrador; no te asignes Owner.

### Caso 2 - PERMISSION_DENIED al habilitar

**Causa probable:** falta **serviceusage.services.enable**.

**Acción:** solicita un rol adecuado, normalmente Service Usage Admin en el proyecto de laboratorio, o usa la alternativa conceptual.

### Caso 3 - PERMISSION_DENIED al consultar cuotas

**Causas posibles:**

- falta **cloudquotas.quotas.get**;
- falta **serviceusage.services.list**;
- falta acceso al proyecto;
- el proyecto usado por **--billing-project** no permite consumir la API.

**Acción:** verifica identidad, proyecto y permisos. Un error de permisos no se arregla aumentando cuota.

### Caso 4 - gcloud no reconoce “quotas”

**Causa probable:** CLI antigua o componentes incompletos.

**Acción:** usa Cloud Shell o actualiza Google Cloud CLI. Confirma con:

~~~bash
gcloud version
gcloud quotas --help
~~~

### Caso 5 - la lista no muestra cuotas

**Causas posibles:**

- API recién habilitada y propagación pendiente;
- filtro incorrecto;
- servicio sin filas visibles para ese alcance;
- permisos de Monitoring o Cloud Quotas insuficientes;
- proyecto equivocado.

**Acción:** espera unos minutos, quita filtros, confirma el servicio y usa **gcloud quotas info list**. No inventes un quota ID.

### Caso 6 - 429 TOO MANY REQUESTS

**Causa probable:** cuota de tasa.

**Acción inicial:** exponential backoff con jitter, reducción de concurrencia, batching y caché. Evalúa aumento solo si la demanda legítima y sostenida lo exige.

### Caso 7 - 403 QUOTA_EXCEEDED en Compute Engine

**Causa probable:** cuota de asignación o tasa específica.

**Acción:** lee la métrica y región del error; libera recursos, prueba una región autorizada o solicita un ajuste. No confundas esta respuesta con un 403 de IAM.

### Caso 8 - el valor existe globalmente, pero falla una región

**Causa probable:** la cuota está dimensionada por región.

**Acción:** consulta la fila de la región exacta. La capacidad en una región no implica capacidad en otra.

### Caso 9 - “Edit” está deshabilitado

**Causas posibles:** permiso insuficiente, system limit, cuota no ajustable o política del laboratorio.

**Acción:** registra la causa; no intentes evadir controles. Completa el recorrido conceptual.

### Caso 10 - solicitud pendiente

**Causa:** una **QuotaPreference** puede estar en estado de reconciliación.

**Acción:** consulta **Increase Requests** o:

~~~bash
gcloud quotas preferences list \
  --project="$PROJECT_ID" \
  --billing-project="$PROJECT_ID" \
  --reconciling-only=true
~~~

No despliegues suponiendo que el valor solicitado ya fue concedido.

### Caso 11 - la cuota aumentó pero el recurso sigue sin crearse

**Causa probable:** capacidad física no disponible, requisito adicional, región no compatible o permiso ausente.

**Acción:** lee el error exacto. Cuota autorizada no garantiza disponibilidad física del recurso.

### Caso 12 - deshabilitar la API falla por dependencias

**Causa:** otros servicios habilitados dependen de ella.

**Acción:** no uses opciones forzadas. Conserva la API habilitada y documenta el motivo, especialmente fuera de un laboratorio.

---

## 15. Impacto en costos y limpieza

### 15.1 Costos

- Habilitar Cloud Quotas API no crea una VM, base de datos o bucket.
- Las consultas de administración están sujetas a cuotas del API.
- Un ajuste de cuota no suele consumir recursos por sí mismo, pero amplía el techo de consumo potencial.
- Usar servicios facturables dentro de una cuota mayor puede elevar el gasto.
- Deshabilitar una API no garantiza que se eliminen datos ni que terminen cargos de almacenamiento.
- Una alerta de presupuesto avisa; no debe interpretarse como corte automático.

### 15.2 Riesgo de deshabilitar APIs

Google Cloud advierte que el efecto depende del servicio:

- algunos datos permanecen y pueden seguir generando cargos;
- algunos recursos pueden suspenderse;
- en ciertos servicios, recursos asociados pueden eliminarse;
- dependencias pueden impedir la deshabilitación.

Por eso se hace inventario antes de deshabilitar.

### 15.3 Limpieza segura

1. Confirma si Cloud Quotas API estaba habilitada antes.
2. Si ya estaba habilitada, **no la deshabilites**.
3. Si tú la habilitaste en un proyecto temporal y ninguna práctica posterior la necesita, revisa dependencias.
4. Solo entonces, en ese laboratorio:

~~~bash
gcloud services disable cloudquotas.googleapis.com \
  --project="$PROJECT_ID"
~~~

5. Verifica el resultado con:

~~~bash
gcloud services list --enabled --project="$PROJECT_ID"
~~~

6. Si el proyecto es efímero de Google Skills, finaliza el laboratorio según sus instrucciones.
7. Limpia variables locales:

~~~bash
unset PROJECT_ID QUOTA_ID
~~~

No fuerces la deshabilitación y no borres un proyecto que no creaste para esta práctica.

---

## 16. Alternativa conceptual completa sin cuenta, crédito o permisos

### 16.1 Escenario

SiteOps Tracker procesa inventarios ficticios. Su backend necesita consultar una API. Observas:

- proyecto: **siteops-lab-example**;
- servicio deseado: **cloudquotas.googleapis.com**;
- API inicialmente deshabilitada;
- identidad con permiso de lectura, pero sin **serviceusage.services.enable**;
- cuota ficticia de lectura: 1,200 solicitudes por minuto;
- demanda normal: 150 solicitudes por minuto;
- pico por duplicados: 1,450 solicitudes por minuto;
- error observado: HTTP 429;
- no existe autorización para solicitar un aumento.

### 16.2 Decide sin consola

Responde:

1. ¿Qué condición impide primero consultar Cloud Quotas?
2. ¿Qué rol específico solicitarías para habilitar el servicio?
3. ¿Un rol de IAM de lectura aumenta la cuota?
4. ¿Qué investigarías antes de pedir 2,000 solicitudes por minuto?
5. ¿Qué mitigaciones aplicarías al pico duplicado?
6. ¿La aprobación de 2,000 garantizaría disponibilidad de otro producto?
7. ¿Qué riesgo financiero existe?

### 16.3 Solución razonada

1. La API debe habilitarse en el proyecto consumidor.
2. Service Usage Admin es un rol predefinido apropiado para administrar habilitación, con alcance en el proyecto.
3. No. IAM autoriza acciones; no cambia el valor de cuota.
4. Origen de duplicados, métricas, ventana de tiempo, tendencia, dimensión, caché y costo.
5. Idempotencia, deduplicación, batching, caché, límites de concurrencia y exponential backoff con jitter.
6. No. Una cuota concedida no garantiza capacidad física ni resuelve otros prerrequisitos.
7. Un techo mayor permite más consumo potencial; deben estimarse cargos y vigilarse presupuestos.

### 16.4 Evidencia conceptual

Escribe una ficha con:

- servicio;
- identidad;
- permiso ausente;
- cuota y ventana;
- causa probable del pico;
- tres mitigaciones;
- valor que solicitarías solo después de medir;
- aprobador;
- métrica de costo que vigilarías.

Si puedes defender esas decisiones sin mirar la solución, la alternativa cubre el objetivo conceptual del laboratorio.

---

## 17. Glosario bilingüe

| English | Español | Definición para ACE |
|---|---|---|
| API | interfaz de programación de aplicaciones | Operaciones expuestas por un servicio |
| service enablement | habilitación de servicio | Autorización del proyecto para consumir una API |
| consumer project | proyecto consumidor | Proyecto contra el que se habilita o consume un servicio |
| resource project | proyecto de recursos | Proyecto que contiene los recursos |
| quota project | proyecto de cuota | Proyecto al que se atribuye la cuota de una llamada cliente |
| quota | cuota | Valor que restringe tasa o asignación |
| rate quota | cuota de tasa | Límite por intervalo de tiempo |
| allocation quota | cuota de asignación | Límite de cantidad de recursos |
| quota value | valor de cuota | Máximo aplicable a una cuota |
| quota ID | identificador de cuota | Identificador único dentro de un servicio |
| metric | métrica | Medida restringida por la cuota |
| dimension | dimensión | Atributo como región, zona o familia |
| quota preference | preferencia de cuota | Valor deseado expresado mediante Cloud Quotas |
| granted value | valor concedido | Capacidad aprobada actualmente |
| preferred value | valor preferido | Capacidad solicitada |
| quota adjustment | ajuste de cuota | Cambio solicitado al valor |
| increase request | solicitud de aumento | Petición de capacidad mayor |
| reconciling | en reconciliación | Solicitud todavía procesándose |
| system limit | límite del sistema | Restricción técnica normalmente no ajustable |
| exponential backoff | espera exponencial | Reintentos con intervalos crecientes |
| jitter | variación aleatoria | Evita que muchos clientes reintenten juntos |
| least privilege | mínimo privilegio | Solo permisos necesarios, por alcance mínimo |
| budget alert | alerta de presupuesto | Aviso financiero que no equivale a cuota |

### Frases clave en inglés

- **Enabling an API does not grant IAM permissions.**
- **A quota adjustment request is subject to review.**
- **A regional quota must be checked in the target region.**
- **A system limit is not the same as an adjustable quota.**
- **A higher quota can increase potential spending.**
- **Use exponential backoff for transient rate-limit errors.**

---

## 18. Preguntas originales estilo ACE - en inglés

Estas preguntas son originales y no proceden de dumps. No consultes las soluciones hasta terminar.

### Question 1

A developer has the Compute Viewer role on a project. The Compute Engine API is disabled. The developer tries to list VM instances and receives a service-disabled error. What should be done first?

A. Increase the regional CPU quota.  
B. Enable the Compute Engine API in the consumer project.  
C. Grant the developer the Owner role.  
D. Create a billing budget.

### Question 2

Your platform team must allow an operator to enable approved Google Cloud APIs in one project. Which predefined role is the most appropriate starting point?

A. Service Usage Admin  
B. Billing Account Administrator  
C. Cloud Quotas Viewer  
D. Organization Administrator

### Question 3

SiteOps Tracker receives HTTP 429 responses during a brief burst of duplicate read requests. Normal traffic is well below the rate quota. What should the team do first?

A. Request the largest possible quota increase.  
B. Grant the runtime service account Editor.  
C. Deduplicate requests and implement exponential backoff with jitter.  
D. Move the PostgreSQL database to another project.

### Question 4

A deployment requires 32 vCPUs in us-east4. The project has sufficient global quota, but the operation reports a regional quota error. What is the best next step?

A. Check the CPU quota and current usage specifically for us-east4.  
B. Enable Cloud Billing export.  
C. Add the user to a Cloud Identity group.  
D. Request an organization-wide IAM policy change.

### Question 5

An administrator submitted a preferred quota value of 100, but the current granted value remains 20 and the request is reconciling. What should the deployment pipeline assume?

A. It can immediately allocate 100 units.  
B. It should rely on the granted value of 20 until the adjustment is approved.  
C. The preferred value automatically overrides every regional limit.  
D. The API is now disabled.

### Question 6

Which statement best distinguishes a budget alert from a service quota?

A. A budget alert authorizes API calls, while a quota assigns IAM roles.  
B. A budget alert notifies about spend; a quota limits a technical rate or allocation.  
C. A quota always stops all billing, while a budget creates resources.  
D. They are two names for the same control.

### Question 7

An engineer can view a quota but cannot edit it. The organization wants least privilege for submitting authorized quota adjustments. Which action is best?

A. Grant Owner at the organization level.  
B. Grant a suitable quota administration role at the narrowest required scope.  
C. Disable and re-enable the API.  
D. Create a new billing account.

### Question 8 - Select two

Which TWO statements about enabling a Google Cloud API are correct?

A. The API is enabled for a specific consumer project.  
B. Enabling the API automatically grants all users permission to call it.  
C. Enabling the API alone does not necessarily create a billable resource.  
D. Enabling the API guarantees that every regional resource is available.  
E. Enabling the API automatically approves all quota increases.

### Question 9 - Select three

Before requesting a production quota increase, which THREE actions are most appropriate?

A. Identify the exact quota ID and dimension.  
B. Measure usage and estimate future demand.  
C. Estimate the cost impact of the higher capacity.  
D. Grant Editor to every developer.  
E. Assume approval and deploy immediately.

### Question 10

A quota adjustment is approved, but a new GPU VM still cannot be created in the selected zone because capacity is unavailable. What is the best explanation?

A. Quota approval and physical resource availability are separate constraints.  
B. IAM roles are no longer required after a quota increase.  
C. The budget alert deleted the VM.  
D. Cloud Quotas automatically disables Compute Engine.

---

## 19. Soluciones justificadas

### Answer 1 - B

**Why B is correct:** the explicit error says the service is disabled, so the first missing prerequisite is API enablement in the consumer project.

**Why the others are wrong:**

- A addresses capacity, not service state.
- C is excessive and still does not target the service-disabled condition.
- D monitors spending; it does not enable an API.

### Answer 2 - A

**Why A is correct:** Service Usage Admin includes permissions to enable, disable and inspect service states.

**Why the others are wrong:**

- B administers billing accounts, not service enablement.
- C is for viewing quotas and does not enable services.
- D is much broader and at the wrong administrative level.

### Answer 3 - C

**Why C is correct:** the burst is caused by duplicate requests and normal traffic fits the quota. Deduplication plus backoff addresses the cause and transient throttling.

**Why the others are wrong:**

- A may increase cost and does not remove duplicate traffic.
- B changes authorization, not rate.
- D is unrelated to the API request burst.

### Answer 4 - A

**Why A is correct:** many allocation quotas are regional. The target region and its current usage determine whether the request fits.

**Why the others are wrong:**

- B provides cost data, not regional CPU capacity.
- C manages membership, not quota.
- D changes access governance and is not indicated by the error.

### Answer 5 - B

**Why B is correct:** preferred value is requested capacity; granted value is the capacity currently available. Reconciling means processing is not complete.

**Why the others are wrong:**

- A assumes approval.
- C ignores dimensions and approved values.
- D has no relation to the preference state.

### Answer 6 - B

**Why B is correct:** budgets and quotas solve different problems: financial notification versus technical consumption limits.

**Why the others are wrong:**

- A reverses unrelated functions.
- C falsely treats quota as a universal billing cutoff.
- D collapses two separate controls.

### Answer 7 - B

**Why B is correct:** a specific administration role at the minimum scope follows least privilege.

**Why the others are wrong:**

- A is unnecessarily broad in role and scope.
- C does not grant quota update permissions.
- D does not address authorization.

### Answer 8 - A and C

**Why A is correct:** service enablement is attached to a consumer project.

**Why C is correct:** enablement alone normally does not create a VM, bucket or database, although later service use can be billable.

**Why the others are wrong:**

- B confuses enablement with IAM.
- D confuses service state with physical availability.
- E confuses enablement with reviewed quota adjustments.

### Answer 9 - A, B and C

**Why A is correct:** the exact quota and dimension prevent a request against the wrong constraint.

**Why B is correct:** demand should support the requested value.

**Why C is correct:** increased capacity can enable increased spending.

**Why the others are wrong:**

- D violates least privilege and does not justify capacity.
- E treats a non-guaranteed request as approved.

### Answer 10 - A

**Why A is correct:** quota is authorization up to a limit; it is not a reservation of physical inventory in a particular zone.

**Why the others are wrong:**

- B is false; IAM remains required.
- C invents behavior budgets do not provide by default.
- D invents behavior Cloud Quotas does not perform.

### Registro de puntuación

- Aciertos: ____ / 10
- Porcentaje: ______ %
- Tiempo: ______ minutos
- Preguntas dudosas: ______________________________
- Categoría principal de error: concepto / lectura / vocabulario / descarte / prisa

La meta personal del plan es 80 % en controles de dominio. No es un umbral oficial del examen.

---

## 20. Repaso activo espaciado

Responde sin mirar lecciones anteriores. Después verifica.

### D-1 hábil - Lección 08: IAM inicial y Cloud Identity

1. Completa: IAM responde “______ puede hacer ______ sobre ______”.
2. ¿Por qué un grupo suele ser mejor que asignar el mismo rol a diez usuarios?
3. ¿Qué diferencia hay entre habilitar una API y conceder un rol para usarla?

### Días hábiles anteriores - Lección 07: Organization Policy

1. ¿Una restricción de Organization Policy concede permisos?
2. Si una política se aplica en una carpeta, ¿qué recursos descendientes pueden heredarla?
3. Da un ejemplo de condición permitida por IAM pero bloqueada por una política de organización.

### Días hábiles anteriores - Lección 06: jerarquía

1. Ordena: recurso, proyecto, carpeta, organización.
2. ¿En qué nivel habilitas normalmente una API?
3. ¿Por qué separar desarrollo y producción en proyectos ayuda a cuotas y costos?

### Días hábiles anteriores - Lección 05: método ACE

1. Traduce: **least operational overhead**.
2. Antes de elegir un producto, ¿qué requisito del escenario elimina distractores?
3. ¿Qué debes registrar cuando una respuesta fue correcta por azar?

### D-7 - Lección 04: regiones, zonas, red y disponibilidad

1. Diferencia región y zona.
2. ¿Por qué una cuota regional obliga a identificar la ubicación?
3. ¿Una cuota suficiente garantiza que un acelerador exista físicamente en una zona?

### D-21

El 10 de septiembre de 2026 todavía no había comenzado este plan; por tanto, no existe una lección D-21 que recuperar. No se inventa contenido. Como sustitución basal, explica en 30 segundos la diferencia entre **project**, **service** y **resource**. La primera recuperación D-21 real ocurrirá cuando una lección del plan alcance esa antigüedad.

### Recuperación de 90 segundos

Sin mirar, completa:

1. API disabled → __________________________
2. PERMISSION_DENIED → _____________________
3. 429 rate limit → _________________________
4. Regional quota exceeded → _______________
5. System limit → ___________________________
6. Higher quota → mayor capacidad potencial y también __________________

---

## 21. Ficha de errores

Completa una ficha por cada pregunta o paso fallado.

| Campo | Tu registro |
|---|---|
| Fecha | 01/10/2026 |
| Pregunta o paso | |
| Mi respuesta o acción | |
| Respuesta o acción correcta | |
| Tipo de error | concepto / lectura / inglés / CLI / permisos / costo |
| Palabra clave que omití | |
| Por qué falló mi razonamiento | |
| Regla de decisión corregida | |
| Explicación sin mirar apuntes | |
| Nueva pregunta que evita memorizar | |
| Fecha de próximo repaso | |

### Errores críticos de hoy

Marca si ocurrió:

- [ ] Confundí API habilitada con IAM.
- [ ] Confundí cuota con presupuesto.
- [ ] Confundí quota con system limit.
- [ ] Ignoré región o dimensión.
- [ ] Supuse que preferred value era granted value.
- [ ] Propuse Owner sin justificarlo.
- [ ] Olvidé medir uso antes de pedir aumento.
- [ ] Supuse que cuota garantizaba capacidad física.
- [ ] Deshabilité una API que ya estaba habilitada.
- [ ] Copié un quota ID sin observarlo en mi proyecto.

### Regla personal

“Antes de actuar sobre una cuota, identificaré __________________, __________________ y __________________.”

---

## 22. Criterios de autoevaluación

Asigna 0, 1 o 2 puntos:

- **0:** no puedo hacerlo.
- **1:** puedo hacerlo con apuntes.
- **2:** puedo hacerlo sin apuntes y justificarlo.

| Criterio | 0 | 1 | 2 |
|---|---:|---:|---:|
| Explico API enablement, IAM, quota y billing por separado | [ ] | [ ] | [ ] |
| Distingo rate quota, allocation quota y system limit | [ ] | [ ] | [ ] |
| Identifico proyecto, servicio, quota ID y dimensión | [ ] | [ ] | [ ] |
| Habilito una API solo en el proyecto correcto | [ ] | [ ] | [ ] |
| Consulto cuotas con Console | [ ] | [ ] | [ ] |
| Consulto QuotaInfo con gcloud | [ ] | [ ] | [ ] |
| Recorro el flujo sin enviar cambios accidentales | [ ] | [ ] | [ ] |
| Selecciono roles más limitados que Owner | [ ] | [ ] | [ ] |
| Explico costo potencial y limpieza segura | [ ] | [ ] | [ ] |
| Obtengo al menos 8/10 y corrijo cada error | [ ] | [ ] | [ ] |

**Puntuación:** ____ / 20

### Interpretación

- **17-20:** base sólida; conserva fichas para repaso.
- **13-16:** repite las partes con 0 o 1.
- **0-12:** rehace la alternativa conceptual y la práctica de solo lectura.

La puntuación es un criterio personal de estudio, no una certificación de dominio ni una escala oficial de Google.

---

## 23. Registro de práctica

### Entorno

- Tipo: Google Skills / proyecto personal temporal / alternativa conceptual
- Proyecto: __________________________________
- Cuenta anonimizada: _________________________
- Inicio: ____________________________________
- Fin: _______________________________________

### API

- Servicio: cloudquotas.googleapis.com
- Estado inicial: _____________________________
- Acción: habilitada / ya habilitada / no autorizada
- Estado final: _______________________________

### Cuota observada

- Nombre: ____________________________________
- Quota ID: __________________________________
- Métrica: ___________________________________
- Valor: _____________________________________
- Dimensión o ubicación: ______________________
- Uso actual: _________________________________
- ¿Ajustable?: ________________________________

### Flujo de aumento

- ¿Se abrió Edit?: ____________________________
- Campo y unidad observados: __________________
- ¿Se mostró Apply for higher quota?: _________
- ¿Se envió solicitud?: **NO**
- Riesgo de costo explicado: __________________

### Limpieza

- ¿La API ya estaba habilitada?: ______________
- Acción de limpieza: _________________________
- Verificación final: _________________________

### Reflexión

- Lo que ya puedo explicar sin apuntes:
- Lo que debo repetir:
- Mi duda principal:

---

## 24. Resumen de decisiones para ACE

1. **API deshabilitada:** habilita el servicio en el proyecto consumidor.
2. **Acceso denegado:** corrige autenticación o IAM; no aumentes cuota.
3. **429 temporal:** backoff, jitter y reducción de demanda antes de pedir más.
4. **Cuota de asignación agotada:** libera recursos, revisa dimensión o solicita ajuste.
5. **System limit:** rediseña o usa una alternativa; no presupongas que es ajustable.
6. **Preferred value pendiente:** planifica con el granted value.
7. **Cuota regional:** consulta la región exacta.
8. **Aumento de cuota:** no garantiza aprobación ni capacidad física.
9. **Mayor cuota:** puede aumentar gasto potencial.
10. **Limpieza:** no deshabilites APIs preexistentes ni fuerces dependencias.

---

## 25. Documentación oficial consultada

Fuentes verificadas el 1 de octubre de 2026:

- [Associate Cloud Engineer certification exam guide](https://cloud.google.com/learn/certification/guides/cloud-engineer) - la guía adjunta incluye “Enabling APIs within projects” y “Assessing quotas and requesting increases” en la sección 1.1.
- [Enable and disable services](https://docs.cloud.google.com/service-usage/docs/enable-disable)
- [List services](https://docs.cloud.google.com/service-usage/docs/list-services)
- [gcloud services list](https://docs.cloud.google.com/sdk/gcloud/reference/services/list)
- [Cloud Quotas overview](https://docs.cloud.google.com/docs/quotas/overview)
- [Quota and system limit terminology](https://docs.cloud.google.com/docs/quotas/terminology)
- [View and manage quotas](https://docs.cloud.google.com/docs/quotas/view-manage)
- [Manage quotas using the gcloud CLI](https://docs.cloud.google.com/docs/quotas/gcloud-cli-examples)
- [gcloud quotas info describe](https://docs.cloud.google.com/sdk/gcloud/reference/quotas/info/describe)
- [Quota permissions](https://docs.cloud.google.com/docs/quotas/permissions)
- [Cloud Quotas roles and permissions](https://docs.cloud.google.com/iam/docs/roles-permissions/cloudquotas)
- [Service Usage roles and permissions](https://docs.cloud.google.com/iam/docs/roles-permissions/serviceusage)
- [Troubleshoot quota errors](https://docs.cloud.google.com/docs/quotas/troubleshoot)

La interfaz, los valores de cuota y la disponibilidad de ajustes pueden cambiar por servicio, proyecto, cuenta y fecha. Confirma siempre la documentación del servicio específico.

---

## 26. Cierre

La idea esencial de hoy es:

> Habilitar permite que el proyecto consuma una API; IAM autoriza al principal; la cuota limita capacidad; facturación cubre consumo; y ninguno sustituye a los demás.

Antes de la siguiente lección, completa la evidencia, corrige todas las preguntas falladas y explica en voz alta un caso de API deshabilitada, uno de IAM y uno de cuota regional.
