# ACE 2026 — Lección 03: Costos, facturación y presupuesto sin sorpresas

**Fecha del plan:** 23 de septiembre de 2026  
**Fecha de regeneración:** 24 de septiembre de 2026  
**Día:** 3 de 74  
**Guía oficial:** Base y sección 1.2 — *Managing billing configuration*  
**Práctica del calendario:** ubicar la cuenta de facturación y preparar una lista de controles de costo y limpieza  
**Duración sugerida:** 90–120 minutos  
**Idioma:** explicación en español; términos de Google Cloud y preguntas de examen en inglés

> Recibir o leer esta lección no demuestra dominio. La evidencia será tu explicación sin apuntes, la inspección o simulación de facturación, la lista de controles terminada y la corrección razonada de las preguntas.

> **Regla de seguridad:** hoy no vincules ni desvincules proyectos, no crees cuentas de facturación, no introduzcas una tarjeta, no habilites APIs y no borres recursos. La ruta práctica usa consultas de solo lectura. Trabaja únicamente con una cuenta y un proyecto propios o asignados por Google Skills.

> **Proyecto de práctica:** SiteOps Tracker es un caso completamente ficticio de portafolio para auditar infraestructura de TI en sedes ficticias. Usa React + TypeScript, Node.js + TypeScript y PostgreSQL; no representa ni reutiliza datos, arquitectura o requisitos de proyectos internos o privados.

---

## 1. Objetivo de aprendizaje

Al terminar la sesión podrás:

1. explicar la relación entre **Cloud Billing account**, **payment profile**, **project**, **service**, **SKU**, **usage** y **cost**;
2. reconocer que una cuenta de facturación determina quién paga y puede financiar varios proyectos;
3. distinguir estimación, atribución, observación, alerta y control de consumo;
4. explicar por qué un presupuesto de alertas normalmente **avisa**, pero no detiene por sí solo el uso ni el gasto;
5. elegir entre Pricing Calculator, Reports, Cost table, budgets, quotas y billing export según el objetivo;
6. identificar los principales generadores de costo de una aplicación como SiteOps Tracker;
7. localizar, sin modificar nada, la configuración de facturación de un proyecto con consola y `gcloud`;
8. elaborar controles concretos antes, durante y después de un laboratorio;
9. responder diez escenarios originales en inglés y justificar por qué los distractores fallan.

### Evidencia mínima del día

- [ ] Identificaste o simulaste el vínculo `project → billing account`.
- [ ] Interpretaste las salidas de los comandos de solo lectura.
- [ ] Completaste la lista de controles de costo y limpieza de la sección 12.
- [ ] Resolviste las diez preguntas sin consultar las soluciones.
- [ ] Registraste cada error o respuesta insegura en la ficha de errores.
- [ ] Explicaste en voz alta: “A budget alert is not a general spending cap.”

---

## 2. Prerrequisitos explicados desde cero

### 2.1 Conceptos recuperados de los días anteriores

De la Lección 01 necesitas estas ideas:

- un **project** agrupa recursos, APIs, permisos, cuotas y datos de facturación;
- un **service** es una capacidad de Google Cloud;
- una **API** permite administrar o consumir esa capacidad;
- un **resource** es el objeto concreto, por ejemplo una VM o una instancia de base de datos.

De la Lección 02 necesitas estas comprobaciones:

- **quién:** cuenta activa;
- **dónde:** proyecto objetivo;
- **cómo:** configuración activa de `gcloud`.

La facturación agrega una cuarta pregunta: **¿quién paga el consumo?**

### 2.2 Modalidad A: con acceso a Google Cloud

Necesitas:

- navegador y Google Cloud console;
- Cloud Shell o Google Cloud CLI;
- permiso para ver un proyecto autorizado;
- de manera opcional, permiso de lectura sobre su información de facturación.

Poder ver un proyecto no garantiza poder ver todos los costos de su cuenta de facturación. Google Cloud separa el acceso a recursos del acceso financiero. No solicites `Owner` para resolver una consulta de lectura.

### 2.3 Modalidad B: sin cuenta, crédito o permiso de facturación

Puedes completar íntegramente la lección con:

- las salidas simuladas de la sección 11;
- el caso de SiteOps Tracker;
- la lista de controles;
- las preguntas y la ficha de errores.

No abras una cuenta de pago ni agregues una tarjeta solo para esta práctica. La ausencia de acceso financiero no impide aprender el modelo ni practicar las decisiones del examen.

### 2.4 Protección de datos

Una cuenta de facturación puede revelar nombres de proyectos, importes y estructura organizacional. En tus notas:

- reemplaza el ID real por `XXXXXX-XXXXXX-XXXXXX`;
- no copies métodos de pago, correos privados ni información fiscal;
- no publiques salidas de una organización sin autorización;
- conserva solo la evidencia necesaria para estudiar.

---

## 3. Relación con la guía oficial ACE

La sección **1.2, Managing billing configuration**, incluye cuatro capacidades:

1. crear una o más cuentas de facturación;
2. vincular proyectos con una cuenta de facturación;
3. establecer presupuestos y alertas;
4. configurar exportaciones de facturación.

Hoy construirás la base conceptual y harás una inspección segura. Los días 10 y 11 profundizarán respectivamente en el vínculo de proyectos, y en presupuestos, alertas y exportación a BigQuery. No se adelantan esas modificaciones en esta práctica.

### Lo que suele medir un escenario de examen

El examen no solo pregunta dónde hacer clic. Normalmente presenta un requisito y espera que identifiques:

- el **alcance**: cuenta, organización, carpeta, proyecto, servicio o recursos etiquetados;
- el **momento**: estimar antes, observar durante o reconciliar después;
- la **acción**: informar, limitar capacidad, automatizar una respuesta o eliminar recursos;
- el **permiso mínimo** necesario;
- la alternativa con menor complejidad operacional que satisfaga el requisito.

---

## 4. Modelo mental: el edificio, el pagador, los medidores y la alarma

Imagina una empresa con varios departamentos:

- La **Cloud Billing account** es el centro de costos que acepta los cargos y define quién paga.
- Cada **project** es un sobre administrativo que contiene recursos y atribuye su consumo.
- Cada recurso tiene **medidores**: segundos de CPU, GiB-mes almacenados, operaciones, solicitudes o GiB transferidos.
- Una **SKU** es una unidad comercial específica con una descripción y precio aplicable.
- El **billing report** es un tablero para analizar tendencias.
- El **Cost table** ayuda a reconciliar el detalle con una factura o estado de cuenta.
- Un **budget alert** es una alarma de humo: advierte; no es, por defecto, una válvula que corte toda la energía.
- La **limpieza** es cerrar llaves y retirar equipos que ya no necesitas.

La analogía evita cuatro errores frecuentes:

1. crear un presupuesto no crea crédito;
2. llegar al 100 % del presupuesto no garantiza que los servicios se detengan;
3. borrar una VM no elimina necesariamente sus discos, snapshots o direcciones reservadas;
4. una cuota limita cierta capacidad técnica, pero no representa un límite monetario global.

---

## 5. La cadena de facturación, paso a paso

### 5.1 Payment profile frente a Cloud Billing account

El **Google payments profile** contiene la relación de pagos, como entidad y métodos de pago. La **Cloud Billing account** acumula y calcula costos de uso y los dirige al perfil de pagos asociado.

Son sistemas relacionados, pero sus permisos no son idénticos. Un rol sobre la cuenta de facturación no debe interpretarse automáticamente como permiso total sobre el perfil de pagos.

### 5.2 Billing account frente a project

La relación fundamental es:

```text
Payment profile
       │
       ▼
Cloud Billing account
       ├── Project: siteops-dev
       ├── Project: siteops-test
       └── Project: siteops-prod
                    ├── Cloud Run service
                    ├── Cloud SQL instance
                    └── Cloud Storage bucket
```

Reglas mentales para el examen:

- una cuenta de facturación puede estar vinculada a varios proyectos;
- un proyecto solo puede estar vinculado a una cuenta de facturación a la vez;
- cambiar el vínculo cambia qué cuenta paga el consumo posterior;
- el acceso al proyecto y el acceso a facturación se administran por separado;
- muchos servicios necesitan que la facturación esté habilitada, incluso cuando parte del uso pueda quedar cubierta por crédito o Free Tier.

### 5.3 De uso técnico a costo

Una simplificación útil es:

```text
Costo bruto ≈ Σ (cantidad utilizada por SKU × precio aplicable)
Costo neto ≈ costo bruto − créditos − descuentos + ajustes e impuestos aplicables
```

No uses esta expresión como calculadora contractual. Los precios pueden incluir niveles, mínimos, redondeo, compromisos, moneda, región y reglas específicas por producto. Úsala para razonar: **el costo nace de una cantidad medida bajo una SKU**.

### 5.4 Ejemplos de medidores

| Recurso o servicio | Posibles factores de costo | Riesgo habitual |
|---|---|---|
| Compute Engine | vCPU, memoria, sistema operativo, disco, GPU, red | dejar VM, disco o IP reservada después del laboratorio |
| Cloud Run | cómputo, solicitudes y red según configuración y uso | configurar instancias mínimas innecesarias o tráfico inesperado |
| Cloud SQL | instancia, almacenamiento, backups y red | mantener una instancia encendida cuando el laboratorio terminó |
| Cloud Storage | datos almacenados, operaciones, recuperación y transferencia | conservar objetos o elegir una clase sin atender el patrón de acceso |
| GKE | nodos o modo de cobro, almacenamiento, balanceo y red | dejar clústeres y balanceadores activos |
| Networking | transferencia de datos, balanceadores, IP y otros componentes | olvidar que el tráfico entre ubicaciones o hacia internet puede cobrar |

La lista no sustituye la página de precios de cada producto. Su propósito es enseñarte a buscar **todos los componentes**, no solo el recurso principal.

---

## 6. El ciclo para evitar sorpresas

### Paso 1: estimar antes de desplegar

Define:

- servicio y región;
- tamaño o capacidad;
- horas de uso;
- volumen de datos;
- operaciones o solicitudes;
- transferencia de red;
- crecimiento esperado.

Usa la **Google Cloud Pricing Calculator** para una estimación previa. Una estimación no es una factura garantizada porque el uso real puede cambiar.

### Paso 2: atribuir con una estructura clara

Separa, cuando tenga sentido:

- desarrollo, pruebas y producción;
- aplicaciones o centros de costo;
- recursos temporales y permanentes.

Los proyectos crean límites administrativos claros. Las **labels** complementan la atribución dentro de esos límites, por ejemplo:

```text
app=siteops-tracker
environment=dev
owner=site-operations-team
expiration=2026-09-23
```

Una etiqueta de expiración no elimina el recurso por sí sola; necesita un proceso que la revise y actúe.

### Paso 3: observar el consumo

En **Cloud Billing Reports** puedes filtrar y agrupar costos, por ejemplo por proyecto, servicio, SKU, ubicación o etiqueta. Esto responde preguntas como:

- ¿qué proyecto aumentó su costo?;
- ¿qué servicio explica el cambio?;
- ¿en qué región se generó?;
- ¿cuál es la tendencia del mes?

### Paso 4: alertar temprano

Un presupuesto tiene:

- **scope / alcance**;
- **amount / monto**;
- **threshold rules / umbrales**;
- **actual** o **forecasted spend**;
- **recipients / destinatarios** o notificaciones programáticas.

Una estrategia didáctica podría alertar al 50 %, 80 % y 100 %, además de un umbral de pronóstico. Es un ejemplo, no una regla universal; los umbrales dependen del riesgo, ritmo de gasto y tiempo de respuesta.

### Paso 5: responder

Una alerta solo es útil si existe un responsable y un procedimiento:

1. confirmar cuenta, proyecto y periodo;
2. abrir Reports y agrupar por proyecto y servicio;
3. comparar con el comportamiento esperado;
4. identificar recursos activos o crecimiento de uso;
5. detener, redimensionar o eliminar únicamente recursos autorizados;
6. documentar causa, acción y verificación.

Las notificaciones a Pub/Sub pueden activar automatización. Sin embargo, una automatización que deshabilita facturación puede provocar pérdida de servicio y no garantiza costo exactamente igual al presupuesto debido a retardos de medición y notificación. Debe diseñarse y probarse como una operación de producción, no asumirse como comportamiento automático del presupuesto.

### Paso 6: limpiar y verificar

Al terminar un laboratorio:

- elimina el recurso según las instrucciones del laboratorio;
- revisa dependencias que puedan permanecer;
- vuelve a consultar el inventario;
- comprueba la página de Billing después del tiempo de actualización correspondiente;
- conserva evidencia de cierre.

**Detener no siempre equivale a dejar de pagar por todo.** Una VM detenida puede conservar discos o direcciones; una instancia borrada puede dejar snapshots; un managed instance group puede recrear una VM eliminada individualmente.

---

## 7. Una distinción crítica: presupuesto, cuota y límite de gasto

| Mecanismo | Pregunta que responde | Lo que sí hace | Lo que no debes asumir |
|---|---|---|---|
| **Alerts-only budget** | ¿El gasto real o previsto cruzó un umbral? | monitoriza y notifica; admite alcances y umbrales | no detiene de forma general los servicios al llegar al monto |
| **Quota** | ¿Cuánta capacidad o cuántas solicitudes permite un servicio? | protege capacidad y uso según la métrica concreta | no fija un límite monetario total ni cubre todos los servicios |
| **Pricing Calculator** | ¿Cuánto podría costar una arquitectura prevista? | estima antes de desplegar | no observa el uso real ni genera alertas |
| **Reports** | ¿Cómo evoluciona el costo y qué lo explica? | analiza tendencias y agrupa/filtra datos | no es la herramienta principal para reconciliar una factura línea por línea |
| **Cost table** | ¿Cómo se compone la factura o estado de cuenta? | presenta detalle tabular reconciliable y descargable | no sustituye la alerta preventiva |
| **Billing export** | ¿Cómo analizo datos de costos con consultas propias? | exporta datos para análisis detallado, por ejemplo en BigQuery | no evita costos y el análisis de BigQuery puede tener costo |
| **Automated response** | ¿Qué acción programada se toma ante una señal? | puede ejecutar una política diseñada por ti | no es automática solo por crear un presupuesto y puede afectar disponibilidad |

La documentación vigente también describe **spend cap budgets** en vista previa para servicios compatibles. No generalices esa capacidad a todos los servicios ni confundas una función de vista previa con el comportamiento estable de un presupuesto de alertas. En un escenario que diga simplemente *budget alerts*, la inferencia segura es **notificación, no corte automático**.

---

## 8. Elegir la herramienta correcta y descartar alternativas

| Requisito del escenario | Elección preferida | Por qué no las alternativas cercanas |
|---|---|---|
| Estimar una arquitectura antes de crearla | Pricing Calculator y precios oficiales | Reports y Cost table necesitan consumo real; un budget no estima componentes técnicos |
| Encontrar el proyecto o servicio que causó un aumento este mes | Reports con filtros y agrupación | Cost table se orienta a reconciliación por periodo de factura; Calculator es predictiva |
| Reconciliar cargos con una factura | Cost table | Reports es mejor para tendencias; un budget solo compara contra un monto |
| Recibir aviso de gasto real o previsto | Budget con umbrales adecuados | Una quota no representa dinero; una label no envía alertas |
| Analizar costos de forma personalizada con SQL | Billing export a BigQuery | El reporte integrado puede ser suficiente para análisis simples y evita operar consultas propias |
| Delegar lectura financiera sin administrar la cuenta | Billing Account Viewer | Billing Account Administrator concede facultades innecesarias; Project Viewer no equivale a ver todos los costos de la cuenta |
| Limitar una métrica técnica de un servicio | Quota apropiada | Un budget no bloquea esa operación; bajar cuotas sin estudiar el tráfico puede romper la aplicación |
| Atribuir costos por ambiente | proyectos separados y/o labels consistentes | nombres informales de recursos son débiles para filtros y gobierno |

### Principio ACE: la solución mínima que satisface el requisito

Si el enunciado solo solicita **ver** costos, no elijas un rol de administración. Si solo necesita una alerta, no diseñes primero una función que deshabilite facturación. Si el reporte integrado responde la pregunta, no exportes a BigQuery sin necesidad.

---

## 9. Decisión de servicios con enfoque de costo: SiteOps Tracker

Para el análisis de costos, supón que SiteOps Tracker tiene:

- frontend React;
- API Node.js/TypeScript en contenedor;
- PostgreSQL;
- pocos usuarios internos al inicio;
- tráfico irregular;
- archivos de evidencia;
- requerimiento de separar desarrollo y producción.

### 9.1 Cómputo para la API

| Opción | Cuándo puede encajar | Por qué no elegirla en el caso inicial |
|---|---|---|
| **Cloud Run** | API HTTP stateless, contenedor, tráfico variable y deseo de reducir operación | es candidata inicial; debe validarse conexión a base, concurrencia, latencia y mínimos de instancia |
| **Compute Engine** | necesitas control del sistema operativo o software específico | una VM siempre aprovisionada añade operación y posible capacidad ociosa si el tráfico es bajo |
| **GKE** | necesitas Kubernetes, múltiples cargas y control de orquestación | para una sola API pequeña puede añadir complejidad y recursos que no justifican el requisito |

**Decisión inicial razonada:** evaluar Cloud Run para la API porque el requisito no pide administrar SO ni Kubernetes. No se elige solo porque “serverless es barato”; se elige porque cumple el patrón operacional y permite alinear capacidad con tráfico. El costo real se estima y se mide.

### 9.2 Base de datos

| Opción | Encaje | Decisión |
|---|---|---|
| **Cloud SQL for PostgreSQL** | modelo relacional existente y compatibilidad PostgreSQL | candidata natural para la primera arquitectura |
| **Firestore** | documentos y patrones de acceso NoSQL | no es reemplazo directo de un esquema PostgreSQL solo para reducir operación |
| **Spanner** | consistencia relacional y escala global exigente | sobredimensionado si el caso no requiere esa escala o disponibilidad global |

### 9.3 Archivos de evidencia

Los archivos grandes no deben guardarse automáticamente como columnas binarias en PostgreSQL. **Cloud Storage** puede ser más adecuado para objetos, mientras PostgreSQL guarda metadatos y referencias. Se deben evaluar permisos, retención, operaciones, almacenamiento y transferencia.

### 9.4 Estructura financiera propuesta

```text
Cloud Billing account: Engineering-Lab
├── project: siteops-tracker-dev
│   └── budget scope: proyecto de desarrollo
└── project: siteops-tracker-prod
    └── budget scope: proyecto de producción
```

Controles:

- presupuestos separados para detectar comportamiento anómalo por ambiente;
- labels consistentes en recursos compatibles;
- responsables y fecha de revisión;
- inventario de recursos antes y después de laboratorios;
- ningún secreto ni método de pago en documentación del proyecto.

---

## 10. Práctica guiada con consola — solo lectura

### Resultado de aprendizaje de la práctica

Debes poder responder:

1. ¿Qué proyecto seleccioné?
2. ¿Tiene facturación habilitada?
3. ¿Qué cuenta paga, si tengo permiso para verla?
4. ¿Hay presupuestos visibles?
5. ¿Qué controles aplicaré antes y después del siguiente laboratorio?

### Paso 0: confirma el alcance

- Usa un proyecto propio o temporal asignado.
- No trabajes en producción.
- Si el proyecto pertenece a Google Skills, sigue las instrucciones del lab y no modifiques la facturación.
- Si ves información de otros equipos, detente y usa la alternativa conceptual.

### Paso 1: identifica el proyecto

1. Abre Google Cloud console.
2. Observa el selector superior de proyecto.
3. Registra solo un alias seguro, por ejemplo `siteops-lab`.
4. Confirma que la cuenta de la esquina superior derecha sea la autorizada.

### Paso 2: localiza el estado de facturación

1. Abre **Navigation menu → Billing**.
2. Si se muestra una cuenta vinculada, registra enmascarados su nombre y el final del ID.
3. Si aparece que la facturación está deshabilitada o no tienes acceso, no intentes corregirlo hoy.
4. No pulses opciones para cambiar, vincular, desvincular o cerrar la cuenta.

### Paso 3: inspecciona Reports, si tienes acceso

1. En Billing, abre **Cost management → Reports**.
2. Selecciona un periodo visible y documenta cuál usaste.
3. Agrupa primero por **Project** y después por **Service**.
4. Si hay datos, anota el servicio con mayor costo sin copiar importes confidenciales.
5. Si no hay datos, registra “sin uso visible o sin permiso”; no generes consumo para llenar el reporte.

### Paso 4: inspecciona Budgets & alerts

1. Abre **Cost management → Budgets & alerts**.
2. No crees ni edites un presupuesto.
3. Para uno existente, si está autorizado, identifica:
   - alcance;
   - periodo;
   - monto;
   - umbrales reales o previstos;
   - destinatarios o canal de notificación.
4. Si no existe ninguno, diseña en papel uno para `siteops-tracker-dev`; no lo guardes hoy.

### Paso 5: distingue Cost table

Si tu cuenta tiene factura o estado de cuenta disponible:

1. abre **Cost management → Cost table**;
2. observa que el objetivo es el detalle de un periodo de factura;
3. no descargues ni compartas datos financieros para esta práctica.

Si no hay factura, basta con explicar la diferencia respecto de Reports.

---

## 11. Práctica guiada con CLI — comandos verificados de solo lectura

Los comandos se comprobaron contra la referencia oficial vigente el **23 de septiembre de 2026**. Los IDs son marcadores; no copies literalmente `000000-000000-000000` como si fuera una cuenta real.

### Paso 1: preflight — quién y dónde

```bash
gcloud auth list --filter='status:ACTIVE' --format='value(account)'
gcloud config get-value project
gcloud config configurations list
```

**Detente si:** la cuenta no es la esperada, el proyecto es producción o no reconoces la configuración.

### Paso 2: guarda y valida el proyecto actual

```bash
PROJECT_ID="$(gcloud config get-value project 2>/dev/null)"

if [ -z "$PROJECT_ID" ] || [ "$PROJECT_ID" = "(unset)" ]; then
  echo "No hay proyecto activo. No continúes con la práctica real."
else
  printf 'Proyecto activo: %s\n' "$PROJECT_ID"
fi
```

Este bloque solo crea una variable local de shell. No cambia Google Cloud.

### Paso 3: consulta el vínculo de facturación del proyecto

```bash
gcloud billing projects describe "$PROJECT_ID"
```

Busca campos equivalentes a:

```yaml
billingAccountName: billingAccounts/XXXXXX-XXXXXX-XXXXXX
billingEnabled: true
name: projects/siteops-tracker-lab/billingInfo
projectId: siteops-tracker-lab
```

Interpretación:

- `billingEnabled: true`: el proyecto tiene facturación habilitada;
- `billingAccountName`: identifica la cuenta vinculada, si tu permiso permite verla;
- `billingEnabled: false`: no intentes vincular una cuenta en esta lección.

### Paso 4: lista únicamente cuentas abiertas visibles para la identidad activa

```bash
gcloud billing accounts list --filter='open=true'
```

Una salida conceptual puede contener:

```text
ACCOUNT_ID                NAME                 OPEN
XXXXXX-XXXXXX-XXXXXX      Engineering-Lab      True
```

Que la lista esté vacía puede significar que no existe una cuenta accesible o que tu identidad carece de permisos. No prueba que el proyecto no tenga facturación.

### Paso 5: inspecciona una cuenta autorizada

Sustituye el marcador solo si el ID apareció en tu salida y estás autorizado:

```bash
BILLING_ACCOUNT_ID="000000-000000-000000"
gcloud billing accounts describe "$BILLING_ACCOUNT_ID"
```

No uses el nombre descriptivo en lugar del ID.

### Paso 6: lista proyectos vinculados, si tu rol lo permite

```bash
gcloud billing projects list \
  --billing-account="$BILLING_ACCOUNT_ID" \
  --limit=20
```

Este comando puede mostrar proyectos distintos al proyecto activo. Ejecútalo solo en una cuenta propia o de laboratorio y no copies nombres internos a tus notas.

### Paso 7: lista presupuestos visibles

```bash
gcloud billing budgets list \
  --billing-account="$BILLING_ACCOUNT_ID"
```

El flag `--billing-account` es obligatorio. Cero resultados no es lo mismo que un error de permisos.

Si el comando solicita habilitar una API, no aceptes automáticamente en un proyecto organizacional o de laboratorio. Usa la consola o la alternativa simulada y registra la limitación.

### Paso 8: limpia las variables locales

```bash
unset PROJECT_ID BILLING_ACCOUNT_ID
```

La práctica no crea recursos en Google Cloud y no requiere comandos destructivos.

### Resultado esperado

Completa esta tabla sin publicar información sensible:

| Comprobación | Resultado real o simulado | Interpretación |
|---|---|---|
| Cuenta activa | `a***@example.com` | identidad autorizada |
| Proyecto | `siteops-tracker-lab` | destino de inspección |
| Billing enabled | `true`, `false` o `sin permiso` | estado observado, no modificado |
| Billing account | `XXXXXX-XXXXXX-123ABC` o `no visible` | pagador vinculado o falta de acceso |
| Budgets visible | cantidad o `sin permiso` | inventario, no protección garantizada |
| Acción posterior | por ejemplo, “crear propuesta de umbrales” | no ejecutar cambios hoy |

---

## 12. Entregable principal: controles de costo y limpieza

Copia y completa esta lista para SiteOps Tracker o el próximo laboratorio.

### Antes de desplegar

- [ ] Confirmé cuenta activa, configuración y proyecto.
- [ ] Verifiqué que el proyecto es temporal o está autorizado.
- [ ] Identifiqué al propietario técnico y financiero.
- [ ] Anoté la hora o fecha de finalización.
- [ ] Identifiqué todos los recursos que la arquitectura creará.
- [ ] Revisé precio, región y medidores relevantes.
- [ ] Preparé una estimación razonable; documenté supuestos.
- [ ] Revisé cuotas como límites técnicos, sin tratarlas como presupuesto.
- [ ] Definí labels de aplicación, ambiente, propietario y expiración cuando el servicio las soporte.
- [ ] Definí presupuesto, umbrales, responsables y procedimiento de respuesta.

### Durante la práctica

- [ ] No cambié de proyecto sin volver a hacer preflight.
- [ ] No concedí `Owner` para resolver un problema de lectura.
- [ ] Registré recursos creados y su ubicación.
- [ ] Evité tamaños, réplicas, GPUs y almacenamiento superiores al requisito.
- [ ] Revisé si el servicio crea dependencias: discos, IP, balanceadores, snapshots o buckets.
- [ ] Consulté Reports o el panel del laboratorio cuando era apropiado.

### Al finalizar

- [ ] Seguí la limpieza indicada por Google Skills o por la práctica.
- [ ] Verifiqué que un controlador no recreará recursos eliminados, por ejemplo un MIG.
- [ ] Comprobé VMs, grupos administrados, discos y direcciones reservadas.
- [ ] Comprobé clústeres, servicios serverless, bases de datos y balanceadores.
- [ ] Comprobé buckets, objetos, snapshots y backups que deban eliminarse.
- [ ] Cerré Cloud Shell y eliminé de mis notas cualquier dato sensible.
- [ ] Volví a consultar el inventario y guardé evidencia de “cero recursos” o de los recursos permanentes autorizados.
- [ ] Programé una revisión posterior del reporte porque el costo no siempre aparece de inmediato.

### Control especial por el problema de SSH/MIG ya observado

Si una VM no acepta SSH o el health check de un managed instance group aparece `UNHEALTHY`, eso no significa que el recurso dejó de generar costo.

- No dependas de SSH para limpiar infraestructura.
- Revisa el recurso desde el plano de control: consola o `gcloud`, con la identidad correcta.
- No borres solo una instancia administrada esperando que desaparezca: el MIG puede recrearla.
- En un proyecto temporal propio, elimina o escala el recurso controlador según la práctica autorizada.
- En Google Skills, usa **End Lab** y el procedimiento del laboratorio; no borres recursos compartidos fuera de sus instrucciones.
- Un presupuesto no resolverá el estado `UNHEALTHY` ni apagará automáticamente el grupo.

Este refuerzo conecta un error real con la lección de hoy sin sustituir el futuro módulo de diagnóstico de MIG y SSH.

---

## 13. Impacto en costos y limpieza de esta práctica

### Costo esperado

- Los comandos indicados son de consulta y no crean recursos facturables.
- La navegación por Billing no genera infraestructura.
- La práctica no crea presupuestos, exportaciones ni notificaciones.
- Consultar no detiene costos que ya estén generando recursos existentes.

### Riesgos indirectos

- Activar una API sin autorización modifica el proyecto.
- Crear una exportación a BigQuery puede producir almacenamiento y consultas facturables.
- Automatizar la desactivación de facturación puede interrumpir servicios.
- Descargar datos financieros puede crear un riesgo de exposición.

### Limpieza concreta

Solo ejecuta:

```bash
unset PROJECT_ID BILLING_ACCOUNT_ID
```

No hay nada más que borrar por esta práctica. Si descubriste recursos anteriores, anótalos y sigue su procedimiento autorizado; no los elimines a ciegas desde esta lección.

---

## 14. Solución de problemas

### `PERMISSION_DENIED`

**Causa probable:** tu cuenta puede ver el proyecto, pero no la cuenta de facturación o los presupuestos.

**Acción:** confirma la cuenta activa, registra qué operación falló y solicita el rol mínimo al administrador. Para lectura general de costos de una cuenta, **Billing Account Viewer** suele ser la referencia apropiada. No te concedas `Owner`.

### `billingEnabled: false`

**Significado:** el proyecto no tiene facturación activa. No significa que debas vincular una tarjeta o cuenta durante la práctica.

**Acción:** usa la alternativa conceptual o un proyecto de laboratorio autorizado.

### `gcloud billing accounts list` no devuelve filas

**Posibilidades:** no hay cuenta abierta visible, la cuenta activa es incorrecta o faltan permisos.

**Acción:** vuelve al preflight. No inventes un ID ni uses uno de otra organización.

### `INVALID_ARGUMENT` al usar la cuenta

**Causa común:** se proporcionó el nombre descriptivo o el prefijo completo en un lugar que espera el ID.

**Acción:** revisa la sintaxis del comando. El ID tiene un formato semejante a `XXXXXX-XXXXXX-XXXXXX`.

### El listado de presupuestos devuelve cero elementos

Eso puede ser un resultado válido. Confirma que consultaste la cuenta correcta y distingue una lista vacía de un error de permisos.

### La consola muestra cero costo, pero hubo actividad

Puede existir retardo de procesamiento, filtros demasiado estrechos, créditos o un periodo equivocado.

**Acción:** revisa rango temporal, proyecto, servicio, créditos y filtros. No crees consumo adicional para “probar”.

### Llegó una alerta después de superar el umbral

Los datos de costo y las notificaciones no son un sistema de corte instantáneo. Un umbral debe dar margen para la latencia y el tiempo humano de respuesta.

### Borraste la VM, pero siguen apareciendo recursos o cargos

Revisa:

- discos persistentes;
- snapshots;
- IP estática reservada;
- balanceador;
- base de datos;
- almacenamiento;
- grupo administrado que recreó la instancia;
- transferencia de red registrada con retraso.

### El proyecto de Google Skills tiene opciones bloqueadas

Es normal que un laboratorio use permisos limitados. Completa los pasos autorizados, conserva la evidencia conceptual y termina el laboratorio. No intentes ampliar IAM ni cambiar la cuenta de facturación.

---

## 15. Glosario bilingüe

| English | Español | Definición práctica |
|---|---|---|
| Cloud Billing account | cuenta de facturación de Cloud | recurso que acumula costos y determina quién paga |
| Google payments profile | perfil de pagos de Google | entidad y configuración de pago asociada |
| linked project | proyecto vinculado | proyecto cuyo consumo se carga a una cuenta de facturación |
| billing enabled | facturación habilitada | estado que permite usar servicios que requieren facturación |
| payer | pagador | cuenta o entidad responsable del pago |
| service | servicio | capacidad de Google Cloud, como Compute Engine |
| resource | recurso | objeto concreto desplegado |
| SKU | unidad de mantenimiento de existencias | elemento comercial medido con precio específico |
| usage | uso o consumo | cantidad medida de un servicio |
| gross cost | costo bruto | costo antes de créditos o descuentos aplicables |
| net cost | costo neto | costo resultante tras ajustes aplicables |
| credit | crédito | importe que reduce cargos bajo condiciones determinadas |
| discount | descuento | reducción de precio según reglas o compromisos |
| budget | presupuesto | monto de referencia para monitorizar gasto |
| budget alert | alerta de presupuesto | notificación al cruzar un umbral |
| threshold | umbral | porcentaje o punto que activa una señal |
| actual spend | gasto real | costo registrado hasta el momento |
| forecasted spend | gasto previsto | proyección basada en la tendencia disponible |
| scope | alcance | recursos o dimensiones cubiertos por una configuración |
| billing report | reporte de facturación | vista de tendencias con filtros y agrupaciones |
| Cost table | tabla de costos | detalle tabular para reconciliar factura o estado de cuenta |
| billing export | exportación de facturación | envío de datos de costos a un destino analítico |
| cost attribution | atribución de costos | asignación del gasto a proyecto, equipo o aplicación |
| label | etiqueta | par clave-valor para organizar y filtrar recursos compatibles |
| quota | cuota | límite técnico de una métrica de servicio |
| notification channel | canal de notificación | destino para recibir avisos |
| egress | salida de red | datos transferidos desde un origen a otro destino |
| idle resource | recurso inactivo u ocioso | recurso aprovisionado sin utilidad suficiente |
| orphaned resource | recurso huérfano | dependencia que permanece después de borrar el recurso principal |
| cleanup | limpieza | eliminación o cierre verificado de recursos temporales |
| least privilege | privilegio mínimo | conceder solo permisos necesarios |
| cost anomaly | anomalía de costo | comportamiento de gasto inesperado |

---

## 16. Repaso activo espaciado

Contesta sin mirar lecciones anteriores. Marca `S` si respondes con seguridad, `D` si dudas y `N` si no puedes responder.

### 16.1 Día hábil anterior — Lección 02

1. ¿Qué tres elementos debes confirmar antes de un cambio con `gcloud`?
2. ¿Cuál es la diferencia entre cuenta activa, configuración activa y proyecto predeterminado?
3. ¿Por qué `gcloud config get-value project` no demuestra que tienes permiso para modificar ese proyecto?
4. ¿Cuándo elegirías consola y cuándo CLI para una tarea repetible?

### 16.2 Días hábiles previos — Lección 01

5. Diferencia **service**, **API** y **resource** con un ejemplo.
6. Ordena: resource, folder, organization, project.
7. Distingue project name, project ID y project number.
8. ¿Por qué habilitar una API no crea automáticamente un recurso facturable?

### 16.3 Recuperación de hace 7 días

No corresponde todavía: el plan comenzó el 21 de septiembre de 2026. La primera recuperación de siete días se moverá al día hábil aplicable según el calendario. No inventes una lección previa.

### 16.4 Recuperación de hace 21 días

No corresponde todavía por la misma razón. Se activará cuando existan al menos 21 días de separación dentro de la ruta.

### Respuestas de control

1. Cuenta, proyecto y configuración activa; además, desde hoy, quién paga.
2. La cuenta firma la solicitud; la configuración agrupa valores de CLI; el proyecto predeterminado es el destino usado por comandos que no reciben otro explícito.
3. Es un valor local de configuración; IAM decide la autorización real.
4. Consola para exploración visual u operación ocasional; CLI para pasos reproducibles, filtrables o repetidos.
5. Servicio: capacidad; API: interfaz habilitada; recurso: objeto creado. Ejemplo: Compute Engine, Compute Engine API y una VM.
6. Organization → Folder → Project → Resource.
7. Nombre legible y modificable; ID globalmente único usado en muchas llamadas; número asignado por Google.
8. Habilitar la interfaz administrativa no aprovisiona por sí solo una VM, base o clúster.

Si fallaste 1–4, vuelve al preflight antes de ejecutar la práctica. Si fallaste 5–8, repasa el modelo de proyecto antes de estudiar la cadena de facturación.

---

## 17. Preguntas originales tipo examen — en inglés

Responde antes de consultar la sección 18. Salvo que se indique **Choose two**, hay una sola mejor respuesta.

### Question 1

Your team creates a monthly alerts-only Cloud Billing budget of USD 100. Actual spend reaches USD 100. What should the team expect by default?

A. Google Cloud immediately stops all services in scope.  
B. The configured notifications are triggered, but usage and charges can continue.  
C. The linked projects are automatically deleted.  
D. All service quotas are reduced to zero.

### Question 2

Which statement correctly describes the relationship between Google Cloud projects and Cloud Billing accounts?

A. A project can be linked to several billing accounts at the same time.  
B. A billing account can pay for only one project.  
C. A billing account can pay for multiple projects, while a project is linked to one billing account at a time.  
D. Every resource must be linked directly to a billing account, bypassing the project.

### Question 3

Before deploying SiteOps Tracker, the team wants an estimate based on expected Cloud Run, Cloud SQL, storage, and network usage. Which tool should they use first?

A. Google Cloud Pricing Calculator  
B. Cloud Billing Cost table  
C. Cloud Audit Logs  
D. VPC Flow Logs

### Question 4

A billing administrator notices that this month's costs increased. They need to identify which project and service caused the change by using built-in filters and grouping. What should they use?

A. Cloud Billing Reports  
B. Cloud Shell Editor  
C. IAM Policy Troubleshooter  
D. Google Cloud Pricing Calculator

### Question 5

The finance team needs a detailed tabular view that reconciles costs to an invoice or statement for a selected invoice month. Which feature is the best fit?

A. Budgets & alerts  
B. Cost table  
C. Service quotas  
D. Error Reporting

### Question 6

A finance analyst must view cost information for all projects linked to a billing account but must not manage the billing account. Which predefined role follows least privilege?

A. Billing Account Administrator  
B. Project Owner  
C. Billing Account Viewer  
D. Organization Administrator

### Question 7 — Choose two

A development project has unpredictable test usage. The team wants early warning and clear attribution without stopping production automatically. Which two actions are most appropriate?

A. Create budget thresholds for actual and forecasted spend.  
B. Apply consistent project and resource labels where supported.  
C. Grant every developer Billing Account Administrator.  
D. Assume the Free Tier prevents all charges.  
E. Set every service quota to its maximum.

### Question 8

Why is a service quota not a reliable replacement for a Cloud Billing budget?

A. Quotas apply only to IAM permissions.  
B. Quotas limit specific technical usage, not total monetary spend across all services.  
C. Quotas can be viewed only after an invoice is issued.  
D. Quotas always delete resources when reached.

### Question 9

SiteOps Tracker has a stateless containerized HTTP API, low and irregular traffic, and no Kubernetes requirement. The team wants to minimize infrastructure operations and avoid paying for unnecessary always-on capacity. Which compute option should they evaluate first?

A. A manually managed Compute Engine VM  
B. A multi-node GKE Standard cluster  
C. Cloud Run  
D. A dedicated bare-metal server

### Question 10

After a lab, SSH to a VM times out and a managed instance group reports unhealthy instances. The team wants to prevent unnecessary cost. What is the best next action?

A. Wait for the monthly budget to stop the resources automatically.  
B. Delete only one managed VM and assume it cannot return.  
C. Verify the authorized project from the control plane, then clean up the MIG or temporary lab project according to the lab procedure.  
D. Increase the VM machine type so SSH becomes cheaper.

---

## 18. Soluciones justificadas

### Question 1 — Correct answer: B

An alerts-only budget triggers configured notifications when its rules are met; it does not generally cap usage or spending.

- **A is wrong:** stopping services is not the default behavior of an alerts-only budget.
- **C is wrong:** budgets do not delete projects.
- **D is wrong:** budgets and quotas are separate controls.

### Question 2 — Correct answer: C

One billing account can pay for multiple linked projects. A project has one billing account link at a time.

- **A is wrong:** simultaneous links to multiple billing accounts are not the model.
- **B is wrong:** a billing account can aggregate several projects.
- **D is wrong:** resources are organized within projects; they are not each linked directly to the billing account.

### Question 3 — Correct answer: A

Pricing Calculator is designed for predeployment estimation using assumed resources and usage.

- **B is wrong:** Cost table analyzes incurred costs associated with an invoice or statement.
- **C is wrong:** Audit Logs records administrative and data access events according to log type, not a price estimate.
- **D is wrong:** VPC Flow Logs provides network flow telemetry, not a forecast.

### Question 4 — Correct answer: A

Reports supports trend analysis, filters, and grouping by dimensions such as project and service.

- **B is wrong:** an editor does not provide billing analysis.
- **C is wrong:** Policy Troubleshooter evaluates access decisions.
- **D is wrong:** the calculator estimates planned usage rather than explaining incurred cost.

### Question 5 — Correct answer: B

Cost table provides detailed tabular costs associated with an invoice or statement and can be downloaded when authorized.

- **A is wrong:** budgets compare spend to thresholds.
- **C is wrong:** quotas restrict technical metrics.
- **D is wrong:** Error Reporting groups application errors.

### Question 6 — Correct answer: C

Billing Account Viewer is intended for read access to billing cost information without account administration.

- **A is wrong:** it grants management capabilities beyond the stated need.
- **B is wrong:** project ownership is broad and does not express read-only access to all billing-account costs.
- **D is wrong:** organization administration is far broader than necessary.

### Question 7 — Correct answers: A and B

Budgets provide early signals, and consistent project/label structure improves attribution.

- **C is wrong:** Billing Account Administrator violates least privilege.
- **D is wrong:** Free Tier is conditional and is not a universal guarantee against charges.
- **E is wrong:** maximum quotas increase possible consumption and do not create useful cost governance.

### Question 8 — Correct answer: B

A quota covers a defined service metric. It does not represent a single monetary ceiling for every SKU and service.

- **A is wrong:** quotas govern capacity or rate, not only IAM.
- **C is wrong:** quotas exist independently of invoice issuance.
- **D is wrong:** reaching a quota typically rejects or limits additional usage; it does not generally delete resources.

### Question 9 — Correct answer: C

Cloud Run is the first option to evaluate for a stateless container, variable traffic and low desired operational overhead. “Evaluate first” still requires checking networking, database connections, minimum instances and cost assumptions.

- **A is wrong:** it adds OS and capacity management without a stated requirement.
- **B is wrong:** Kubernetes adds unnecessary complexity for the described single workload.
- **D is wrong:** it conflicts with the managed-cloud and low-operations requirements.

### Question 10 — Correct answer: C

SSH is not required to manage or delete infrastructure. The control plane can inspect and clean the authorized MIG or temporary project. A managed group can recreate an individually deleted VM.

- **A is wrong:** a normal budget alert does not stop resources automatically.
- **B is wrong:** the MIG controller may recreate the instance.
- **D is wrong:** a larger machine does not resolve cost governance or necessarily fix SSH.

### Registro de resultado

| Métrica | Resultado |
|---|---|
| Aciertos | `/10` |
| Respuestas seguras | `/10` |
| Respuestas correctas pero inseguras |  |
| Errores por concepto |  |
| Errores por leer deprisa |  |
| Tiempo total |  |

Una respuesta correcta con duda también merece ficha: el objetivo es poder justificarla, no adivinarla.

---

## 19. Ficha de errores

Completa una fila por cada error y por cada acierto inseguro.

| Pregunta o práctica | Mi respuesta | Respuesta correcta | Tipo de error | Regla de decisión | Explicación nueva sin mirar | Fecha de repaso |
|---|---|---|---|---|---|---|
| Ejemplo: Q1 | A | B | confundí alerta con límite | alerts-only budget notifica; no corta por defecto | “La alarma informa y necesita un procedimiento de respuesta” | próximo día hábil |
|  |  |  | concepto / inglés / lectura / proceso |  |  |  |
|  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |

### Clasificación recomendada

- **Concepto:** no conocía la regla.
- **Inglés:** interpreté mal una palabra como *forecasted*, *linked* o *scope*.
- **Lectura:** ignoré “least privilege”, “before deployment” o “by default”.
- **Proceso:** elegí una herramienta válida, pero para otra fase del ciclo.
- **Sobreingeniería:** propuse automatización o permisos superiores sin requisito.

---

## 20. Criterios de autoevaluación

Puntúa cada criterio de 0 a 2:

- **0:** no puedo explicarlo;
- **1:** lo explico con ayuda o dudo;
- **2:** lo explico y lo aplico sin apuntes.

| Criterio | 0–2 |
|---|---:|
| Relaciono payment profile, billing account y project |  |
| Distingo presupuesto, quota y límite de gasto |  |
| Elijo Calculator, Reports o Cost table según el momento |  |
| Interpreto `billingEnabled` y una cuenta vinculada |  |
| Aplico privilegio mínimo a una necesidad financiera |  |
| Identifico recursos huérfanos y el riesgo de un MIG |  |
| Completo la lista antes/durante/después |  |
| Justifico al menos 8 de 10 preguntas en inglés |  |

**Total:** `/16`

### Criterio de avance de estudio

- **13–16:** continúa, pero conserva las fichas de respuestas inseguras.
- **9–12:** repite la práctica conceptual y explica las decisiones en voz alta.
- **0–8:** vuelve a las secciones 4–8 antes de avanzar.

Error crítico que exige repaso aunque el total sea alto: afirmar que un presupuesto de alertas detiene automáticamente todos los servicios al llegar al 100 %.

Estos criterios son metas personales de estudio, no un umbral oficial del examen.

---

## 21. Cierre de la lección

Completa estas frases sin mirar:

1. A Cloud Billing account ____________________________________________.
2. A project can be linked to ________________________________________.
3. An alerts-only budget _____________________________________________.
4. A quota differs from a budget because ______________________________.
5. Reports is preferable when ________________________________________.
6. Cost table is preferable when ______________________________________.
7. After deleting a VM, I must also check ______________________________.
8. If a MIG is unhealthy, cost can continue because ____________________.

### Explicación de 60 segundos

Graba o pronuncia una respuesta a este escenario:

> “Explain how you would prevent cost surprises in a temporary Google Cloud lab.”

Incluye estas expresiones:

- “First, I verify the active account and project.”
- “I estimate the main cost drivers before deployment.”
- “A budget alert does not automatically stop spending.”
- “I inspect dependent resources during cleanup.”
- “Finally, I verify that no temporary resources remain.”

---

## 22. Documentación oficial consultada

Verificada el **23 de septiembre de 2026**:

- Guía oficial adjunta: `associate_cloud_engineer_exam_guide_english.pdf`, página 1, sección 1.2.
- [Associate Cloud Engineer certification](https://cloud.google.com/learn/certification/cloud-engineer)
- [Create, edit, or delete budgets and budget alerts](https://docs.cloud.google.com/billing/docs/how-to/budgets)
- [Analyze billing data and cost trends with Reports](https://docs.cloud.google.com/billing/docs/how-to/reports)
- [View and download cost details with Cost table](https://docs.cloud.google.com/billing/docs/how-to/cost-table)
- [Cloud Billing access control and permissions](https://docs.cloud.google.com/billing/docs/how-to/billing-access)
- [Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator)
- [`gcloud billing accounts list`](https://docs.cloud.google.com/sdk/gcloud/reference/billing/accounts/list)
- [`gcloud billing accounts describe`](https://docs.cloud.google.com/sdk/gcloud/reference/billing/accounts/describe)
- [`gcloud billing projects describe`](https://docs.cloud.google.com/sdk/gcloud/reference/billing/projects/describe)
- [`gcloud billing projects list`](https://docs.cloud.google.com/sdk/gcloud/reference/billing/projects/list)
- [`gcloud billing budgets list`](https://docs.cloud.google.com/sdk/gcloud/reference/billing/budgets/list)

Los precios, nombres de productos, funciones en vista previa y opciones de CLI pueden cambiar. Para una práctica futura, vuelve a consultar la documentación del día en lugar de memorizar una versión.

---

## 23. Registro personal de finalización

**Fecha y hora de estudio:**  
**Modalidad:** real / simulada  
**Proyecto usado (alias seguro):**  
**Puntuación:** `/10`  
**Concepto más débil:**  
**Acción de refuerzo:**  
**Evidencia de limpieza:**  
**Duda para la próxima lección:**  

### Próxima lección del calendario

**Lección 04 — Regiones, zonas, red y disponibilidad.** No la adelantes si todavía confundes presupuesto, cuota y límite de gasto.
