# ACE 2026 - Lección 11: Presupuestos, alertas y exportación de facturación

> **Fecha:** 5 de octubre de 2026  
> **Dominio de la guía ACE:** 1.2 Managing billing configuration  
> **Tema del plan:** Presupuestos, alertas y exportación de facturación  
> **Práctica del plan:** Configurar o simular un presupuesto y una exportación a BigQuery; interpretar un reporte.  
> **Duración sugerida:** 90 minutos  
> **Caso conductor:** SiteOps Tracker, proyecto ficticio de portafolio.

Esta lección enseña y ejercita una parte del temario. Recibir el archivo no demuestra dominio: compruébalo ejecutando o simulando la práctica, explicando las decisiones sin apuntes y resolviendo los escenarios.

## 1. Objetivo

Al finalizar podrás:

1. explicar qué mide un **Cloud Billing budget** y qué no controla;
2. distinguir **actual spend** de **forecasted spend**;
3. definir alcance, periodo, importe y reglas de umbral de un presupuesto;
4. elegir entre correo basado en roles, canales de Cloud Monitoring y Pub/Sub;
5. distinguir las exportaciones **Standard usage cost**, **Detailed usage cost** y **Pricing data**;
6. crear o simular un presupuesto de alertas para SiteOps Tracker;
7. preparar o simular un dataset de BigQuery, habilitar la exportación desde Console e interpretar una consulta;
8. reconocer costos, permisos, latencia de datos y limpieza segura.

### Evidencia de aprendizaje

Al terminar conserva:

- una ficha del presupuesto con alcance, periodo, importe y umbrales;
- una decisión justificada sobre el tipo de exportación;
- una consulta o simulación que agrupe costos por proyecto y servicio;
- una interpretación de tres frases;
- el registro de limpieza o la indicación “alternativa conceptual”.

## 2. Alineación con la guía oficial

La guía oficial de Associate Cloud Engineer incluye, dentro de **1.2 Managing billing configuration**:

- **Establishing billing budgets and alerts**.
- **Setting up billing exports**.

La lección de hoy cubre exactamente esos dos puntos. El vínculo entre proyecto y cuenta de facturación se repasará solo como prerrequisito de la lección 10.

## 3. Prerrequisitos explicados desde cero

### 3.1 Proyecto y cuenta de facturación

Un proyecto contiene los recursos técnicos y registra su consumo. Una Cloud Billing account recibe los cargos de los proyectos vinculados.

- Una cuenta puede pagar varios proyectos.
- Un proyecto se vincula con una sola cuenta activa a la vez.
- El vínculo de pago no convierte a la cuenta en padre IAM del proyecto.

Para SiteOps Tracker, los proyectos ficticios de desarrollo y producción pueden generar costos por la API Node.js, la interfaz React, PostgreSQL administrado, almacenamiento de evidencias y transferencia de red.

### 3.2 Costo, crédito y neto

- **Cost:** cargo generado por el uso.
- **Credit:** ajuste negativo como un descuento o crédito promocional.
- **Net cost:** costo después de aplicar créditos.

En datos exportados, los créditos suelen representarse como importes negativos. No confundas costo bruto con costo neto al interpretar una consulta.

### 3.3 BigQuery

BigQuery es la plataforma analítica administrada de Google Cloud. Para esta práctica basta entender:

- un **project** contiene uno o más datasets;
- un **dataset** contiene tablas y vistas;
- una consulta GoogleSQL lee columnas y agrupa filas;
- almacenar y consultar datos puede generar cargos.

No necesitas experiencia previa en data engineering.

### 3.4 IAM mínimo

Crear un presupuesto a nivel de cuenta y configurar una exportación son operaciones distintas.

| Tarea | Acceso habitual |
|---|---|
| Crear o modificar un presupuesto de cuenta | Billing Account Costs Manager o Billing Account Administrator |
| Ver presupuestos | Permisos de lectura de presupuestos en la cuenta |
| Configurar exportación Standard/Detailed | Billing Account Costs Manager o Billing Account Administrator en la cuenta; BigQuery User en el proyecto del dataset |
| Configurar exportación Pricing | Billing Account Administrator en la cuenta; BigQuery Admin y permiso adicional indicado por la documentación en el proyecto |
| Crear canal de correo de Monitoring | Monitoring Editor en el proyecto del canal |

El acceso por proyecto para administrar presupuestos individuales existe con reglas específicas y algunas funciones en Preview. Para el flujo estable de CLI de esta lección se usa un presupuesto perteneciente a la cuenta de facturación.

## 4. Modelo mental: alarma, libro mayor y almacén analítico

Imagina la operación de varias sedes:

- el **presupuesto** es el plan mensual que el responsable financiero escribe en una pizarra;
- el **umbral** es una marca de advertencia: 50 %, 80 % o 100 %;
- la **alerta** es el mensaje que avisa que se cruzó una marca;
- la **exportación** es una copia continua del libro mayor;
- el **dataset de BigQuery** es el archivo analítico donde se investiga qué servicio, proyecto o recurso produjo el costo.

La alarma no apaga automáticamente el edificio. De igual manera, un presupuesto **alerts-only** informa, pero no detiene automáticamente el uso ni el gasto.

~~~mermaid
flowchart LR
  USE["SiteOps usage"] --> BILL["Cloud Billing account"]
  BILL --> BUDGET["Alerts-only budget"]
  BUDGET --> NOTICE["Email / Monitoring / Pub/Sub"]
  BILL --> EXPORT["Billing export"]
  EXPORT --> BQ["BigQuery dataset"]
~~~

## 5. Presupuestos paso a paso

### 5.1 Las cuatro decisiones

Todo presupuesto responde cuatro preguntas:

1. **Scope:** ¿qué consumo vigila?
2. **Time period:** ¿durante qué intervalo?
3. **Amount:** ¿contra qué importe se compara?
4. **Threshold rules:** ¿cuándo y por qué se envía una alerta?

### 5.2 Alcance

Un presupuesto alerts-only puede cubrir:

- toda una cuenta de facturación;
- organizaciones, carpetas o proyectos seleccionados pagados por esa cuenta;
- servicios concretos;
- recursos con una etiqueta determinada;
- combinaciones permitidas de filtros.

Ejemplo de SiteOps Tracker:

| Presupuesto | Alcance | Utilidad |
|---|---|---|
| SiteOps total | Cuenta completa | Visión financiera global |
| SiteOps desarrollo | Proyecto ficticio de desarrollo | Detectar pruebas abandonadas |
| PostgreSQL administrado | Servicio de base de datos en proyectos seleccionados | Vigilar el componente de datos |
| Equipo o entorno | Etiqueta consistente | Asignación interna de costos |

Un presupuesto con filtro de carpeta u organización solo considera proyectos que también sean pagados por la cuenta de facturación del presupuesto.

### 5.3 Periodo

Los presupuestos alerts-only permiten:

- **Monthly:** se reinicia cada mes;
- **Quarterly:** se reinicia cada trimestre natural;
- **Yearly:** se reinicia cada año;
- **Custom range:** intervalo no recurrente, con fecha inicial y fin opcional.

La documentación actual indica que los periodos comienzan a las 00:00 en la zona del Pacífico de Estados Unidos y Canadá. No asumas que el corte mensual coincide exactamente con la medianoche de Ciudad de México.

### 5.4 Importe

Dos estrategias frecuentes:

- **Specified amount:** cantidad fija en la moneda de la cuenta.
- **Last period amount:** el gasto del periodo anterior se convierte en referencia.

Cantidad fija es mejor cuando existe un objetivo aprobado. El periodo anterior es útil como línea base, pero conserva anomalías: un mes anormalmente caro produciría una referencia también alta.

### 5.5 Umbrales

Una regla combina:

- un porcentaje entre 0 y 1 en CLI;
- una base: gasto actual o gasto pronosticado.

Ejemplo con presupuesto de 1,000 unidades:

| Regla | Se activa cuando... | Interpretación |
|---|---|---|
| 50 % actual | El costo observado supera 500 | Aviso temprano basado en consumo ya registrado |
| 80 % forecast | El pronóstico supera 800 | Riesgo de terminar el periodo por encima de la trayectoria esperada |
| 100 % actual | El costo observado supera 1,000 | El objetivo ya fue rebasado |

**Actual** mira lo registrado hasta ahora. **Forecasted** estima el total del periodo a partir de la tendencia. Un forecast puede cruzar 80 % aunque el gasto actual aún sea 62 %.

### 5.6 Destinatarios y automatización

| Mecanismo | Cuándo elegirlo | Por qué no elegir otra opción |
|---|---|---|
| Correos basados en roles | Billing admins/users son quienes deben actuar | No exige canales adicionales |
| Cloud Monitoring email channels | Deben recibir avisos personas que no tienen roles de facturación | Darles un rol de facturación solo para recibir correo sería privilegio innecesario |
| Pub/Sub | Un sistema debe procesar el evento y ejecutar un flujo controlado | El correo no es una interfaz automatizable fiable |

Los canales personalizados no conceden automáticamente permiso para abrir el presupuesto. Recibir un enlace por correo y poder ver el recurso son autorizaciones separadas.

### 5.7 Alerts-only frente a spend cap

La regla principal para ACE es:

> Un presupuesto **alerts-only** no es un spending cap y no detiene automáticamente recursos.

La documentación actual también describe **spend cap budgets** en Preview para servicios elegibles y con limitaciones estrictas. Es una función distinta:

- se limita a un solo proyecto y un solo servicio elegible;
- solo algunos servicios API-based son compatibles;
- no cubre de forma general todos los cargos;
- recursos persistentes pueden seguir acumulando cargos;
- requiere intervención manual para reanudar el servicio.

No respondas una pregunta general sobre “budget alerts” como si siempre fuera un apagado automático. Solo elige un spend cap cuando el escenario lo mencione o satisfaga explícitamente sus condiciones.

## 6. Exportación de facturación a BigQuery

### 6.1 Para qué sirve

Los reportes integrados responden preguntas comunes. La exportación a BigQuery permite:

- consultas personalizadas;
- desglose por proyectos, servicios, SKUs, etiquetas y ubicaciones;
- unión con inventarios o centros de costo;
- históricos para dashboards;
- análisis repetible mediante SQL y vistas.

La exportación escribe datos automáticamente durante el día. No es una fotografía manual.

### 6.2 Tipos principales

| Exportación | Incluye | Elección para SiteOps Tracker | Por qué no elegir las otras |
|---|---|---|---|
| **Standard usage cost** | Cuenta, factura, proyecto, servicio, SKU, ubicación, costo, uso, créditos, etiquetas y moneda | Tendencias por proyecto y servicio | Detailed puede leer más datos y Pricing no contiene el uso real |
| **Detailed usage cost** | Todo Standard más datos a nivel de recurso para servicios compatibles | Encontrar una VM, disco u otro recurso concreto que impulsa el costo | Es más granular y sus consultas pueden procesar más bytes |
| **Pricing data** | Catálogo de precios aplicable a la cuenta, SKUs, unidades, moneda y niveles | Comparar precios y construir modelos | No reemplaza los registros de consumo |
| **FOCUS usage cost** | Datos normalizados conforme al estándar FinOps FOCUS | Comparación normalizada cuando el caso lo exige | Está en Preview y no es necesario para la práctica básica ACE |

Para hoy se recomienda **Standard usage cost**: basta para interpretar tendencias de los proyectos ficticios.

### 6.3 Tablas creadas

Tras habilitar cada exportación, Google crea tablas en el dataset:

| Tipo | Patrón de tabla |
|---|---|
| Standard | <code>gcp_billing_export_v1_BILLING_ACCOUNT_ID</code> |
| Detailed | <code>gcp_billing_export_resource_v1_BILLING_ACCOUNT_ID</code> |
| Pricing | <code>cloud_pricing_export</code> |

Usa el nombre exacto que aparezca en BigQuery Explorer. No publiques el ID real de la cuenta.

### 6.4 Ubicación y disponibilidad

- La ubicación del dataset se fija al crearlo y no puede cambiarse.
- La exportación acepta las multirregiones US y EU, además de regiones específicas compatibles.
- Con US o EU, la primera habilitación de Standard/Detailed puede incluir datos desde el inicio del mes anterior.
- El backfill retroactivo puede tardar hasta cinco días.
- En una región compatible, Standard/Detailed empieza desde la fecha de habilitación y no agrega datos anteriores.
- Pricing no se rellena retroactivamente.
- Al cambiar a otro proyecto o dataset, el histórico del dataset anterior no se copia automáticamente.

### 6.5 Servicio administrado

Al habilitar Standard o Detailed, Google agrega al dataset la service account administrada <code>billing-export-bigquery@system.gserviceaccount.com</code> con los permisos requeridos para escribir. No la elimines; hacerlo detiene la actualización y arriesga pérdida de datos.

### 6.6 Esquema cambiante

El esquema de las tablas exportadas puede añadir campos. Para consumidores estables:

1. crea una vista con las columnas que tu dashboard necesita;
2. hace que los reportes consulten la vista;
3. adapta la vista cuando el esquema de origen cambie.

## 7. Diseño para SiteOps Tracker

### 7.1 Arquitectura ficticia

| Componente | Servicio de ejemplo | Señal financiera |
|---|---|---|
| Frontend React + TypeScript | Hosting estático o servicio autorizado | Transferencia y almacenamiento |
| API Node.js + TypeScript | Cloud Run | Solicitudes, CPU y memoria |
| PostgreSQL | Cloud SQL for PostgreSQL | Instancia, almacenamiento y backups |
| Evidencias de auditoría | Cloud Storage | Capacidad, operaciones y red |
| Análisis financiero | BigQuery | Almacenamiento y bytes procesados |

### 7.2 Política propuesta

- Presupuesto mensual alerts-only de desarrollo.
- Importe fijo aprobado para el laboratorio.
- Umbrales: 50 % actual, 80 % forecast y 100 % actual.
- Destinatarios: equipo responsable mediante roles; Monitoring si necesita correo un operador sin acceso de facturación.
- Exportación Standard a un proyecto FinOps o de laboratorio separado.
- Vista o consulta por proyecto y servicio.
- Detailed solo si se necesita identificar un recurso concreto.

## 8. Decisiones de examen y distractores frecuentes

| Requisito del escenario | Mejor elección | Distractor y por qué falla |
|---|---|---|
| Avisar antes de superar objetivo | Budget con forecast threshold | Quota limita capacidad, no vigila costo total |
| Analizar tendencias por proyecto/servicio | Standard export | Pricing describe precios, no consumo |
| Identificar VM o disco costoso | Detailed export | Standard puede no aportar granularidad de recurso |
| Enviar evento a automatización | Pub/Sub | Correo depende de una persona y no es interfaz de programa |
| Agregar destinatario sin rol de Billing | Monitoring email channel | Billing Admin sería privilegio excesivo |
| Evitar interrupción y solo informar | Alerts-only budget | Spend cap puede pausar servicios elegibles |
| Proteger costo de una consulta | Dry run y maximum bytes billed | LIMIT no reduce necesariamente bytes leídos |
| Mantener consultas resistentes al esquema | Vista estable | Referenciar todas las columnas directamente aumenta fragilidad |

## 9. Práctica guiada

### 9.1 Reglas de seguridad

Realiza cambios solo en:

- un proyecto temporal de Google Skills;
- un proyecto personal de laboratorio;
- un proyecto expresamente autorizado.

No uses producción. No publiques IDs de facturación, correos, información fiscal o nombres privados. Si no tienes cuenta, permisos o crédito, realiza la alternativa conceptual de la sección 9.7: cubre íntegramente el objetivo.

### 9.2 Plan de la práctica

Crearás o simularás:

1. un presupuesto mensual alerts-only;
2. un dataset BigQuery;
3. una exportación Standard habilitada desde Console;
4. una consulta agregada por proyecto y servicio;
5. una interpretación y limpieza.

### 9.3 Preflight con Google Cloud CLI

Los comandos y opciones fueron comprobados en la documentación oficial vigente al 5 de octubre de 2026.

#### Paso 1: identidad y proyecto

~~~bash
gcloud auth list --filter=status:ACTIVE
gcloud config get-value project
~~~

Si no reconoces la identidad o el proyecto, detente.

#### Paso 2: define marcadores autorizados

~~~bash
export PROJECT_ID="YOUR_AUTHORIZED_PROJECT_ID"
export BILLING_ACCOUNT_ID="000000-000000-000000"
export DATASET_ID="siteops_billing_lab"
~~~

No pegues valores reales en una captura pública.

#### Paso 3: confirma el vínculo

~~~bash
gcloud billing projects describe "$PROJECT_ID"
gcloud billing accounts describe "$BILLING_ACCOUNT_ID"
~~~

Comprueba que:

- el proyecto está vinculado a la cuenta prevista;
- la cuenta está abierta;
- el proyecto es de laboratorio;
- tienes los permisos requeridos.

### 9.4 Presupuesto con Console

1. Abre **Billing** en Google Cloud Console.
2. Elige la cuenta autorizada.
3. Ve a **Cost management → Budgets & alerts**.
4. Selecciona **Create budget**.
5. Nombre: <code>siteops-lab-monthly</code>.
6. Time range: **Monthly**.
7. Scope:
   - proyecto de laboratorio;
   - todos los servicios;
   - todas las etiquetas, salvo que pruebes una etiqueta deliberada.
8. Amount: importe fijo pequeño, expresado en la moneda de la cuenta.
9. Thresholds:
   - 50 % de actual spend;
   - 80 % de forecasted spend;
   - 100 % de actual spend.
10. Mantén los destinatarios predeterminados del laboratorio.
11. Revisa el resumen antes de finalizar.
12. Guarda y registra el nombre, alcance, importe y umbrales.

No selecciones un spend cap para esta práctica.

### 9.5 Presupuesto con CLI

Esta ruta crea el mismo concepto mediante <code>gcloud</code>. No la ejecutes si ya creaste el presupuesto de Console con el mismo nombre.

#### Paso 1: comprueba o habilita la API solo en el laboratorio

~~~bash
gcloud services list --enabled --project="$PROJECT_ID" --filter="name:billingbudgets.googleapis.com"
gcloud services enable billingbudgets.googleapis.com --project="$PROJECT_ID"
~~~

La segunda línea modifica el proyecto. Ejecútala solo si la API no está habilitada y tienes autorización.

#### Paso 2: revisa presupuestos existentes

~~~bash
gcloud billing budgets list --billing-account="$BILLING_ACCOUNT_ID"
~~~

Evita duplicar un presupuesto del mismo alcance.

#### Paso 3: crea el presupuesto

~~~bash
gcloud billing budgets create --billing-account="$BILLING_ACCOUNT_ID" --display-name="siteops-lab-monthly" --budget-amount=10 --calendar-period=month --filter-projects="projects/$PROJECT_ID" --threshold-rule=percent=0.50 --threshold-rule=percent=0.80,basis=forecasted-spend --threshold-rule=percent=1.00
~~~

Notas:

- <code>10</code> usa la moneda asociada a la cuenta cuando no se especifica una divisa.
- <code>0.50</code> representa 50 %, no 0.5 %.
- <code>forecasted-spend</code> corresponde al umbral pronosticado.
- La salida incluye el identificador del presupuesto; consérvalo para la limpieza.

#### Paso 4: verifica

~~~bash
gcloud billing budgets list --billing-account="$BILLING_ACCOUNT_ID"
~~~

No esperes que la creación del presupuesto reduzca o detenga recursos.

### 9.6 Exportación Standard a BigQuery

#### Parte A: crea el dataset con Console

1. Abre **BigQuery**.
2. Selecciona el proyecto de laboratorio que contendrá los datos.
3. En Explorer, abre el menú del proyecto y elige **Create dataset**.
4. Dataset ID: <code>siteops_billing_lab</code>.
5. Para esta simulación, usa la multirregión **US** solo si cumple tus requisitos de residencia.
6. No configures expiración automática de tablas.
7. Conserva Google-managed encryption para el laboratorio.
8. Crea el dataset.

En un entorno real, ubicación, residencia, cifrado y retención se deciden antes de crearlo.

#### Parte B: alternativa CLI para crear el dataset

No la ejecutes si ya creaste el dataset en Console.

~~~bash
gcloud services enable bigquery.googleapis.com --project="$PROJECT_ID"
bq mk --dataset --location=US "$PROJECT_ID:$DATASET_ID"
~~~

La ubicación queda fijada al crear el dataset.

#### Parte C: habilita la exportación en Console

La configuración del vínculo de exportación se realiza en Cloud Billing Console; no inventes un comando <code>gcloud billing export</code>.

1. Abre **Billing → Billing export**.
2. Elige la cuenta autorizada.
3. Abre la pestaña **BigQuery export**.
4. En **Standard usage cost**, elige **Enable**.
5. Selecciona el proyecto que contiene el dataset.
6. Selecciona <code>siteops_billing_lab</code>.
7. Verifica que el proyecto esté vinculado a la misma cuenta cuyos datos exportarás.
8. Guarda.
9. Espera a que aparezca la tabla. Puede tomar varias horas; un backfill multirregional puede tardar hasta cinco días.

No confundas ausencia inmediata de filas con un fallo.

#### Parte D: inspecciona las tablas

~~~bash
bq ls "$PROJECT_ID:$DATASET_ID"
~~~

Identifica el nombre Standard que empiece con <code>gcp_billing_export_v1_</code>. Anonimízalo en tu evidencia.

#### Parte E: estima el costo de la consulta

Reemplaza los tres marcadores dentro del identificador de tabla antes de ejecutar:

~~~bash
bq query --use_legacy_sql=false --dry_run '
SELECT
  project.id AS project_id,
  service.description AS service,
  ROUND(SUM(cost), 2) AS gross_cost,
  currency
FROM `YOUR_PROJECT_ID.YOUR_DATASET_ID.YOUR_STANDARD_EXPORT_TABLE`
WHERE usage_start_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
GROUP BY project_id, service, currency
ORDER BY gross_cost DESC
LIMIT 20'
~~~

<code>--dry_run</code> valida y estima bytes sin ejecutar la consulta.

#### Parte F: ejecuta con límite de bytes

Solo si el dry run muestra un tamaño aceptable:

~~~bash
bq query --use_legacy_sql=false --maximum_bytes_billed=100000000 '
SELECT
  project.id AS project_id,
  service.description AS service,
  ROUND(SUM(cost), 2) AS gross_cost,
  currency
FROM `YOUR_PROJECT_ID.YOUR_DATASET_ID.YOUR_STANDARD_EXPORT_TABLE`
WHERE usage_start_time >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
GROUP BY project_id, service, currency
ORDER BY gross_cost DESC
LIMIT 20'
~~~

El máximo de 100,000,000 bytes hace que la consulta falle si necesita facturar más bytes. Ajustarlo requiere una decisión consciente.

Esta consulta muestra costo bruto. Para un reporte financiero completo debes incorporar créditos, ajustes, impuestos y la lógica de factura que requiera el caso.

### 9.7 Alternativa conceptual completa sin cuenta, crédito o permisos

Usa este caso:

- presupuesto mensual: 1,000 MXN;
- umbral 1: 50 % actual;
- umbral 2: 80 % forecast;
- umbral 3: 100 % actual;
- gasto actual: 620 MXN;
- pronóstico de cierre: 920 MXN.

Datos simulados del Standard export:

| project_id | service | gross_cost | currency |
|---|---|---:|---|
| siteops-dev | Cloud SQL | 310 | MXN |
| siteops-dev | Cloud Run | 180 | MXN |
| siteops-dev | Networking | 60 | MXN |
| siteops-dev | BigQuery | 45 | MXN |
| siteops-dev | Cloud Storage | 25 | MXN |
| **Total** |  | **620** | **MXN** |

Resuelve:

1. Porcentaje actual: 620 / 1,000 = **62 %**.
2. El umbral de 50 % actual ya se cruzó.
3. El de 100 % actual no se cruzó.
4. El forecast de 920 equivale a **92 %**, así que el umbral de 80 % forecast se cruzó.
5. Cloud SQL es el mayor contribuyente bruto del ejemplo.
6. Standard permite comparar servicios, pero no garantiza identificar la instancia concreta; para eso evaluarías Detailed.
7. El presupuesto no detiene automáticamente Cloud SQL ni Cloud Run.

Entrega el diagrama, la tabla de presupuesto y esta interpretación:

> “El gasto actual es 62 % del objetivo y el pronóstico es 92 %. Debieron activarse el aviso de 50 % actual y el de 80 % forecast. Cloud SQL concentra el mayor costo bruto. Antes de optimizar, validaría créditos y granularidad; el presupuesto alerts-only no apaga servicios.”

## 10. Resultado esperado

### Presupuesto

| Campo | Resultado esperado |
|---|---|
| Nombre | <code>siteops-lab-monthly</code> |
| Periodo | Monthly |
| Alcance | Un proyecto autorizado |
| Importe | Cantidad fija pequeña en moneda de la cuenta |
| Umbrales | 50 % actual, 80 % forecast, 100 % actual |
| Tipo | Alerts-only |

### Exportación

| Campo | Resultado esperado |
|---|---|
| Dataset | <code>siteops_billing_lab</code> |
| Tipo | Standard usage cost |
| Tabla | Patrón <code>gcp_billing_export_v1_...</code> |
| Consulta | Agrupa por proyecto y servicio |
| Seguridad de costo | Dry run y límite de bytes |

### Interpretación mínima

Escribe:

1. qué servicio concentra más costo;
2. si el resultado es bruto o neto;
3. qué decisión tomarías y qué dato adicional validarías.

## 11. Solución de problemas

| Síntoma | Causa probable | Verificación | Acción |
|---|---|---|---|
| No aparece la cuenta | Identidad equivocada o falta de acceso | <code>gcloud auth list</code> y lista de cuentas visible | Cambia a identidad autorizada o solicita acceso mínimo |
| <code>PERMISSION_DENIED</code> al crear budget | Falta permiso <code>billing.budgets.create</code> | Revisa rol en la cuenta | Solicita Billing Account Costs Manager; no pidas Admin sin necesidad |
| El budget incluye proyectos no deseados | Scope demasiado amplio | Revisa filtros de proyecto/carpeta/servicio | Corrige alcance antes de confiar en alertas |
| No llega correo | Destinatario o canal incorrecto; umbral aún no cruzado | Revisa reglas y recipients | Ajusta canal; no bajes artificialmente producción para “probar” |
| La alerta forecast aparece antes que actual | El pronóstico estima el cierre | Compara actual y forecast | Trátalo como señal preventiva, no error |
| Se cruzó 100 % y los recursos siguen activos | Es un budget alerts-only | Revisa tipo de presupuesto | Es comportamiento esperado |
| <code>bq mk</code> dice que el dataset existe | Ya fue creado | <code>bq ls "$PROJECT_ID"</code> | Reutiliza solo si ubicación y configuración son correctas |
| Invalid dataset region | Ubicación no compatible | Revisa región del dataset | Crea otro dataset en ubicación compatible; la ubicación no cambia |
| No aparece tabla tras habilitar export | Propagación inicial en curso | Revisa configuración y espera varias horas | No habilites y deshabilites repetidamente |
| Falta histórico | Dataset regional o export recién habilitada | Revisa fecha y ubicación | Acepta el límite; no inventes backfill |
| La exportación dejó de actualizarse | Service account administrada eliminada | Revisa IAM del dataset | Restaura el acceso o reconfigura según documentación |
| La consulta falla por bytes máximos | Supera <code>maximum_bytes_billed</code> | Ejecuta dry run | Reduce periodo/columnas o aumenta el límite conscientemente |
| El total no coincide con factura | Créditos, ajustes, impuestos, latencia o periodo | Revisa campos y fechas | Define si reportas bruto, neto o invoice total |
| Consulta falla tras cambio de esquema | Dependencia directa de columnas cambiantes | Revisa esquema de export | Usa una vista estable y actualízala |

## 12. Costos y limpieza

### 12.1 Impacto en costos

- Crear un budget no tiene cargo por sí mismo.
- Las alertas no eliminan el consumo que ya ocurrió.
- BigQuery puede cobrar almacenamiento y cómputo de consultas.
- La carga automática de datos al dataset no es el principal riesgo; las consultas que leen muchos bytes pueden serlo.
- Detailed suele procesar más datos que Standard.
- Usa dry run, filtros de fecha y <code>maximum_bytes_billed</code>.
- No interpretes <code>LIMIT 20</code> como garantía de bajo costo: BigQuery puede leer más datos antes de limitar filas.

### 12.2 Limpieza segura

Si solo hiciste la simulación, escribe “sin recursos creados”.

Si habilitaste la práctica:

1. En **Billing → Billing export**, deshabilita primero la exportación Standard del dataset de laboratorio.
2. Verifica que el budget que borrarás sea exactamente el del laboratorio.
3. Obtén su ID:

~~~bash
gcloud billing budgets list --billing-account="$BILLING_ACCOUNT_ID"
~~~

4. Declara el ID exacto y elimina con confirmación interactiva:

~~~bash
export BUDGET_ID="YOUR_LAB_BUDGET_ID"
gcloud billing budgets delete "$BUDGET_ID" --billing-account="$BILLING_ACCOUNT_ID"
~~~

5. Inspecciona el dataset:

~~~bash
bq ls "$PROJECT_ID:$DATASET_ID"
~~~

6. Solo si el dataset fue creado exclusivamente para esta práctica y no contiene datos necesarios, elimínalo. Se conserva la confirmación interactiva:

~~~bash
bq rm -r -d "$PROJECT_ID:$DATASET_ID"
~~~

7. No deshabilites BigQuery ni Billing Budget API en un proyecto compartido.
8. Registra qué se eliminó y qué quedó.

Eliminar el dataset destruye sus tablas. La exportación no reconstruye necesariamente el histórico eliminado.

## 13. Glosario bilingüe

| English | Español | Uso práctico |
|---|---|---|
| Billing budget | Presupuesto de facturación | Compara gasto con un objetivo |
| Alerts-only budget | Presupuesto solo de alertas | Informa sin detener automáticamente el uso |
| Spend cap budget | Presupuesto con tope de gasto | Función distinta y limitada que puede pausar servicios elegibles |
| Actual spend | Gasto actual | Costo observado hasta el momento |
| Forecasted spend | Gasto pronosticado | Estimación del cierre del periodo |
| Threshold rule | Regla de umbral | Condición que dispara una alerta |
| Budget scope | Alcance del presupuesto | Proyectos, servicios, carpetas, etiquetas u otros filtros |
| Calendar period | Periodo natural | Mes, trimestre o año recurrente |
| Custom period | Periodo personalizado | Intervalo no recurrente |
| Notification channel | Canal de notificación | Destino gestionado por Cloud Monitoring |
| Programmatic notification | Notificación programática | Mensaje procesable mediante Pub/Sub |
| Billing export | Exportación de facturación | Flujo automático de datos financieros |
| Standard usage cost | Costo de uso estándar | Datos suficientes para tendencias generales |
| Detailed usage cost | Costo de uso detallado | Añade granularidad por recurso compatible |
| Pricing data | Datos de precios | Información de SKUs, unidades y tarifas |
| Dataset | Conjunto de datos | Contenedor de tablas y vistas en BigQuery |
| Backfill | Relleno retroactivo | Carga de datos históricos disponible bajo ciertas condiciones |
| Gross cost | Costo bruto | Costo antes de créditos |
| Net cost | Costo neto | Costo después de créditos |
| Dry run | Ejecución de prueba | Valida y estima bytes sin ejecutar |
| Maximum bytes billed | Máximo de bytes facturados | Límite de seguridad para una consulta |

## 14. Repaso activo espaciado

Responde sin mirar las soluciones anteriores.

### Día hábil anterior - Lección 10

1. ¿Cuántas cuentas de facturación activas puede tener vinculadas un proyecto a la vez?
2. ¿Vincular una cuenta concede acceso a Cloud SQL?
3. ¿Qué dos superficies IAM intervienen al vincular proyecto y cuenta?
4. ¿Por qué el proyecto que contiene el dataset de exportación debe estar vinculado a la cuenta correcta?

### Dos días hábiles antes - Lección 9

1. Distingue API habilitada, cuota y presupuesto.
2. ¿Una cuota limita gasto total en dinero?
3. ¿Qué API verificarías antes de usar BigQuery?

### Tres días hábiles antes - Lección 8

1. Define principal, role y policy.
2. ¿Por qué no dar Billing Account Administrator solo para recibir alertas?
3. ¿Qué aplica mínimo privilegio: canal de Monitoring o rol de facturación amplio?

### Cuatro días hábiles antes - Lección 7

1. ¿Puede una Organization Policy sustituir una alerta de presupuesto?
2. ¿Conceder un rol IAM evita una restricción organizacional?
3. ¿Dónde investigarías una restricción que impide crear un dataset?

### Hace 7 días - Lección 6

El 28 de septiembre correspondió a jerarquía:

1. Dibuja Organization → Folder → Project → Resource.
2. Añade una Cloud Billing account sin convertirla en padre.
3. Añade el dataset BigQuery dentro de un proyecto.
4. Explica qué relación sirve para IAM y cuál para pagar.

### Hace 21 días - línea base

El 14 de septiembre fue anterior al inicio del plan; no se inventa una lección. Recupera la línea base:

1. ¿Qué pensabas que era un presupuesto cloud?
2. Explica ahora por qué “budget = apagado automático” es incompleto.
3. Escribe una pregunta de costos que BigQuery sí podría responder.

## 15. Ficha de práctica

| Campo | Registro |
|---|---|
| Modalidad | Console + CLI / simulación |
| Proyecto anonimizado |  |
| Budget creado o simulado |  |
| Alcance |  |
| Importe y moneda |  |
| Umbrales |  |
| Tipo de exportación |  |
| Dataset anonimizado |  |
| Mayor costo observado |  |
| Resultado bruto o neto |  |
| Limpieza completada |  |
| Duda principal |  |

## 16. Ficha de errores

| Nº | Mi respuesta o acción | Corrección | Tipo de error | Señal ignorada | Regla de decisión | Refuerzo |
|---:|---|---|---|---|---|---|
| 1 |  |  | Concepto / IAM / CLI / SQL / lectura |  |  |  |
| 2 |  |  | Concepto / IAM / CLI / SQL / lectura |  |  |  |
| 3 |  |  | Concepto / IAM / CLI / SQL / lectura |  |  |  |
| 4 |  |  | Concepto / IAM / CLI / SQL / lectura |  |  |  |
| 5 |  |  | Concepto / IAM / CLI / SQL / lectura |  |  |  |

Refuerzos rápidos:

- Si confundiste budget con cap, escribe: “alerts-only avisa; no detiene”.
- Si elegiste Pricing para consumo real, compara filas de uso frente a catálogo.
- Si elegiste Detailed sin requerir recurso, vuelve a leer el requisito de granularidad.
- Si una consulta fue cara, practica dry run y filtros de fecha.
- Si esperabas datos inmediatos, registra la latencia y el comportamiento de backfill.

## 17. Criterios de autoevaluación

Asigna 0, 1 o 2:

- **0:** aún no puedo explicarlo;
- **1:** lo explico con notas;
- **2:** lo explico y justifico sin notas.

| Criterio | 0-2 |
|---|---:|
| Explico por qué alerts-only no es spending cap |  |
| Distingo actual spend y forecasted spend |  |
| Defino scope, period, amount y thresholds |  |
| Elijo email, Monitoring o Pub/Sub con criterio |  |
| Distingo Standard, Detailed y Pricing |  |
| Explico ubicación y backfill |  |
| Creo o simulo budget sin duplicarlo |  |
| Creo o simulo dataset y export |  |
| Uso dry run y límite de bytes |  |
| Interpreto bruto frente a neto |  |
| Completo limpieza segura |  |
| Justifico distractores en inglés |  |
| **Total** | **/24** |

Interpretación:

- **20-24:** continúa; repasa cualquier distractor fallado;
- **15-19:** repite la consulta y la tabla de decisiones;
- **0-14:** rehace la simulación completa antes de avanzar.

La puntuación es una guía de estudio, no una afirmación de dominio ni un umbral oficial de Google.

## 18. Documentación oficial verificada

Consultada el 5 de octubre de 2026:

- [Associate Cloud Engineer exam guide](https://cloud.google.com/learn/certification/guides/cloud-engineer)
- [Create, edit, or delete budgets and budget alerts](https://docs.cloud.google.com/billing/docs/how-to/budgets)
- [Customize budget alert email recipients](https://docs.cloud.google.com/billing/docs/how-to/budgets-notification-recipients)
- [Set up programmatic budget notifications](https://docs.cloud.google.com/billing/docs/how-to/budgets-programmatic-notifications)
- [Manage spend cap budgets - Preview](https://docs.cloud.google.com/billing/docs/how-to/budgets-spend-caps)
- [gcloud billing budgets create](https://docs.cloud.google.com/sdk/gcloud/reference/billing/budgets/create)
- [gcloud billing budgets list](https://docs.cloud.google.com/sdk/gcloud/reference/billing/budgets/list)
- [gcloud billing budgets delete](https://docs.cloud.google.com/sdk/gcloud/reference/billing/budgets/delete)
- [Export Cloud Billing data to BigQuery](https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery)
- [Set up Cloud Billing data export to BigQuery](https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery-setup)
- [Understand Cloud Billing data tables](https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery-tables)
- [Create and manage BigQuery datasets](https://docs.cloud.google.com/bigquery/docs/managing-datasets)
- [Run queries and use dry runs](https://docs.cloud.google.com/bigquery/docs/running-queries)
- [Estimate and control BigQuery costs](https://docs.cloud.google.com/bigquery/docs/best-practices-costs)
- [gcloud services enable](https://docs.cloud.google.com/sdk/gcloud/reference/services/enable)

## 19. Preguntas de examen originales

Responde antes de abrir las soluciones. Son escenarios originales, no preguntas filtradas.

### Question 1

SiteOps Tracker has an alerts-only monthly budget. Actual spend reaches 110% of the budget amount, but the services continue running. What is the best explanation?

A. The budget is broken because every Cloud Billing budget stops resources at 100%.  
B. Alerts-only budgets notify recipients but do not automatically cap usage or spending.  
C. The billing account must be linked to two projects before enforcement works.  
D. BigQuery export must be enabled before a budget can stop resources.

### Question 2

The team wants an early warning when current spend is low but the projected month-end spend is likely to exceed 80% of the target. Which rule should they configure?

A. 80% actual-spend threshold  
B. 80% forecasted-spend threshold  
C. A BigQuery table expiration at 80 days  
D. A quota increase at 80%

### Question 3

Finance wants broad cost trends by project, service, SKU, and location. Resource-level identifiers are not required. Which export is the best initial choice?

A. Standard usage cost export  
B. Detailed usage cost export  
C. Pricing data export only  
D. Cloud Audit Logs export

### Question 4

An engineer must identify the specific virtual machine or disk driving an increase in cost. Which export should the engineer evaluate?

A. Standard usage cost only  
B. Detailed usage cost  
C. Pricing data  
D. Cloud Billing account list

### Question 5

A project manager needs budget alert emails but should not receive billing administration privileges. What should the cloud engineer configure?

A. Grant Billing Account Administrator  
B. Link a Cloud Monitoring email notification channel to the budget  
C. Make the project manager a Project Owner on every project  
D. Export pricing data to the manager's personal project

### Question 6

The operations team needs a system to receive budget events and invoke an approved workflow. Which notification mechanism should they use?

A. Pub/Sub programmatic notifications  
B. A screenshot of the Billing report  
C. A higher service quota  
D. A second billing account

### Question 7

The team enabled Standard usage cost export five minutes ago, but no table is visible. The configuration and permissions are correct. What should they do first?

A. Repeatedly disable and re-enable the export  
B. Wait for initial propagation and recheck later  
C. Delete the billing account  
D. Manually insert billing rows into a new table

### Question 8

An analyst wants to minimize the risk of an unexpectedly expensive BigQuery query. Which approach is best?

A. Add LIMIT 20 and assume only 20 rows are read  
B. Use a dry run, filter the time range, and set maximum bytes billed  
C. Use the Detailed export for every query  
D. Move the project to another billing account before querying

### Question 9

A team wants to analyze prices for SKUs and pricing tiers rather than its actual usage records. Which export should it enable?

A. Standard usage cost  
B. Detailed usage cost  
C. Pricing data  
D. VPC Flow Logs

### Question 10

A company enables Standard export into a supported regional BigQuery dataset and expects data from the previous month. No older rows appear. What is the best explanation?

A. Regional datasets generally start collecting from the date export is enabled; retroactive behavior differs from US/EU multi-region datasets.  
B. Standard export never includes project information.  
C. BigQuery cannot store Cloud Billing data in a regional dataset.  
D. Budget thresholds must reach 100% before export begins.

## 20. Soluciones justificadas

### 1. Correct answer: B

Un alerts-only budget dispara avisos; no detiene automáticamente uso o facturación.

- **A** generaliza incorrectamente. Los spend caps son una función distinta y limitada.
- **C** no tiene relación con el comportamiento del budget.
- **D** mezcla alertas con exportación analítica.

### 2. Correct answer: B

La base forecasted-spend compara el pronóstico del periodo con el porcentaje objetivo.

- **A** se activa solo cuando el gasto observado cruza 80 %.
- **C** controla retención de tablas, no trayectoria de gasto.
- **D** aumenta capacidad y no crea una alerta financiera.

### 3. Correct answer: A

Standard ofrece las dimensiones necesarias para tendencias generales y evita granularidad innecesaria.

- **B** es útil cuando se requieren recursos concretos y puede procesar más datos.
- **C** contiene precios, no el historial de consumo real.
- **D** registra eventos de auditoría, no sustituye la exportación de facturación.

### 4. Correct answer: B

Detailed añade datos a nivel de recurso para servicios compatibles.

- **A** puede no identificar la VM o disco concreto.
- **C** describe tarifas y SKUs, no qué recurso consumió.
- **D** solo enumera cuentas.

### 5. Correct answer: B

Un canal de correo de Cloud Monitoring permite personalizar destinatarios sin conceder administración de facturación.

- **A** viola mínimo privilegio.
- **C** concede control técnico amplio e innecesario.
- **D** no configura alertas ni es una práctica aceptable de gobierno.

### 6. Correct answer: A

Pub/Sub entrega mensajes programáticos que un flujo autorizado puede procesar.

- **B** es manual.
- **C** cambia capacidad, no notificaciones.
- **D** cambia la estructura financiera sin resolver la integración.

### 7. Correct answer: B

La creación y propagación inicial de tablas puede tardar varias horas; el backfill puede tardar más.

- **A** puede crear huecos y complica la configuración.
- **C** es destructivo y no resuelve latencia.
- **D** no debe hacerse: el proceso de exportación administra sus tablas.

### 8. Correct answer: B

Dry run estima bytes; el filtro reduce el alcance y maximum bytes billed impide superar el límite configurado.

- **A** no garantiza que solo se lean 20 filas.
- **C** suele aumentar la granularidad y bytes.
- **D** no controla el costo de la consulta.

### 9. Correct answer: C

Pricing data contiene SKUs, unidades, moneda, agregación y niveles de precio aplicables.

- **A** y **B** se centran en uso y costos.
- **D** registra tráfico de red.

### 10. Correct answer: A

Para una ubicación regional compatible, Standard/Detailed comienzan desde la habilitación; el backfill de mes actual y anterior corresponde a la primera habilitación en US o EU multi-region.

- **B** es falso: Standard incluye dimensiones de proyecto.
- **C** es falso: existen regiones compatibles.
- **D** inventa una dependencia entre thresholds y export.
