# ACE 2026 — Lección 05

## Método ACE: CLI, escenarios y vocabulario técnico en inglés

| Dato | Valor |
|---|---|
| Fecha del plan | 25 de septiembre de 2026 |
| Día | 5 de 74 |
| Cobertura de la guía | Base y visión transversal de las secciones 1–4 |
| Proyecto de práctica | **SiteOps Tracker** — caso ficticio de portafolio |
| Evidencia obligatoria | Diagnóstico original de 15 preguntas y registro de errores por dominio |
| Duración sugerida | 80–105 minutos |

> Esta lección construye un método de resolución y obtiene una línea base. Una puntuación diagnóstica no demuestra dominio ni predice por sí sola el resultado del examen.

---

## 1. Objetivo

Al terminar esta lección podrás:

1. Descomponer una pregunta de escenario del Associate Cloud Engineer (ACE) en requisito, estado actual, restricción y acción.
2. Reconocer palabras clave en inglés sin decidir únicamente por una palabra aislada.
3. Leer la estructura de un comando `gcloud`, consultar su ayuda y distinguir operaciones de lectura de operaciones que cambian recursos.
4. Usar la consola y la CLI para verificar, sin crear recursos, la cuenta activa, el proyecto, sus metadatos y las APIs habilitadas.
5. Comparar servicios por ajuste al requisito, carga operativa, seguridad, disponibilidad y costo.
6. Resolver 15 preguntas diagnósticas originales en inglés.
7. Registrar cada error con una regla de decisión reutilizable.

---

## 2. Prerrequisitos explicados desde cero

No necesitas dominar Google Cloud todavía. Sí conviene reconocer estas ideas de las cuatro lecciones anteriores:

### 2.1 Proyecto

Un **project** es una frontera de administración. Agrupa recursos, habilita APIs, recibe políticas de IAM, maneja cuotas y se vincula con una cuenta de facturación.

**Analogía:** en SiteOps Tracker, un proyecto es como un expediente operativo separado. Desarrollo y producción pueden tener expedientes distintos para reducir el riesgo de mezclar permisos, cuotas y recursos.

### 2.2 Recurso

Un **resource** es una entidad administrable: una VM, un bucket, una base de datos o un servicio de Cloud Run, por ejemplo. Casi toda pregunta ACE pide crear, configurar, consultar, operar o proteger uno o más recursos.

### 2.3 Cuenta de facturación, presupuesto y cuota

- Una **billing account** permite pagar el consumo de proyectos vinculados.
- Un **budget** compara gasto real o previsto contra umbrales y puede enviar alertas.
- Una **quota** limita la cantidad o la tasa de uso de un recurso o API.

Un presupuesto no es un interruptor automático de gasto. Una cuota tampoco sustituye un control financiero: su propósito principal es controlar capacidad o tasa.

### 2.4 Configuración y credenciales de la CLI

- La **configuration** activa de `gcloud` contiene propiedades como proyecto, cuenta, región o zona predeterminados.
- Las **credentials** prueban quién ejecuta la acción.
- Tener una cuenta autenticada no significa que esa identidad tenga permisos en todos los proyectos.

### 2.5 Región y zona

- Una **region** es un área geográfica.
- Una **zone** es un dominio de despliegue dentro de una región.

Distribuir recursos entre zonas puede proteger contra el fallo de una zona. Una sola zona sigue siendo un único dominio de fallo.

### 2.6 Herramientas necesarias

Elige una opción:

- **Opción A:** Google Cloud Console con Cloud Shell y acceso de solo lectura a un proyecto de laboratorio.
- **Opción B:** la práctica conceptual completa de esta lección, sin cuenta ni crédito.

No necesitas habilitar APIs, crear recursos ni proporcionar una tarjeta para completar la práctica conceptual.

---

## 3. Qué evalúa el ACE

La guía oficial adjunta organiza el examen estándar en cuatro dominios:

| Dominio | Peso aproximado | Pregunta mental |
|---|---:|---|
| 1. Setting up a cloud solution environment | 20 % | ¿Cómo preparo proyectos, cuentas, red, APIs, cuotas y facturación? |
| 2. Planning and implementing a cloud solution | 30 % | ¿Qué servicio y configuración implementan mejor el requisito? |
| 3. Ensuring the successful operation of a cloud solution | 30 % | ¿Cómo observo, mantengo, recupero o corrijo lo desplegado? |
| 4. Configuring access and security | 20 % | ¿Qué identidad y permisos mínimos necesita cada actor? |

A fecha del 25 de septiembre de 2026, Google publica para el examen estándar:

- duración de 2 horas;
- 50–60 preguntas de opción múltiple y selección múltiple;
- ausencia de prerrequisitos formales;
- recomendación de 6 o más meses de experiencia práctica.

Los porcentajes son aproximados. No permiten deducir una nota de aprobación oficial.

### 3.1 La pregunta no suele pedir “el mejor producto” en abstracto

Pide el producto o la acción que satisface **ese conjunto concreto de restricciones**. El mismo servicio puede ser correcto en un escenario e incorrecto en otro.

Ejemplo:

- “Run a stateless HTTP container with minimal operational overhead” apunta normalmente a Cloud Run.
- “Run custom Kubernetes controllers and use Kubernetes-native APIs” apunta a GKE.
- “Install a kernel module and control the guest OS” apunta a Compute Engine.

La decisión cambia porque cambia el requisito, no porque un producto sea universalmente superior.

---

## 4. Método CLAVE para resolver escenarios

Usaremos la palabra **CLAVE** como procedimiento repetible.

### C — Clasifica el dominio y el estado actual

Pregunta:

- ¿El caso trata de entorno, implementación, operación o seguridad?
- ¿Qué existe ya?
- ¿Qué falta?

Palabras que señalan el dominio:

- **set up, organization, project, API, quota, billing** → entorno;
- **choose, deploy, create, implement** → planificación e implementación;
- **monitor, troubleshoot, restore, scale, inspect** → operación;
- **grant, role, principal, service account, least privilege** → acceso y seguridad.

No elijas aún una respuesta. Primero ubica el problema.

### L — Localiza la instrucción y las restricciones

Subraya mentalmente:

1. El verbo principal: `choose`, `configure`, `identify`, `troubleshoot`.
2. El modificador: `first`, `most appropriate`, `most cost-effective`.
3. Las restricciones no negociables: `without downtime`, `least privilege`, `minimal operational overhead`.
4. El número de respuestas: `Choose one` o `Choose two`.

Ejemplo:

> The team must deploy a stateless container, scale to zero, and minimize operational overhead.

Las tres restricciones importan. “Container” por sí sola no obliga a elegir GKE.

### A — Aísla la acción exacta

Convierte el enunciado en una frase corta:

> “Necesito ejecutar un contenedor HTTP sin administrar servidores y con escalado a cero.”

Si no puedes resumir el problema, todavía no conviene comparar opciones.

### V — Valida cada opción contra los requisitos

Prueba cada opción con cinco filtros:

1. **Funcionalidad:** ¿resuelve la necesidad?
2. **Alcance:** ¿actúa sobre el recurso o nivel correcto?
3. **Seguridad:** ¿aplica mínimo privilegio y evita credenciales de larga duración?
4. **Operación:** ¿cumple la carga administrativa, disponibilidad y recuperación pedidas?
5. **Costo:** ¿evita capacidad o servicios innecesarios sin sacrificar requisitos?

Una opción puede ser técnicamente posible y aun así no ser la **best answer**.

### E — Elimina distractores y explica la decisión

Descarta respuestas que:

- resuelven un problema distinto;
- conceden más permisos de los necesarios;
- ignoran una palabra como `existing`, `first` o `without downtime`;
- requieren administrar infraestructura que el escenario desea evitar;
- añaden costo o complejidad sin beneficio requerido;
- usan un comando destructivo para una consulta.

Antes de confirmar, completa:

> Elijo ___ porque satisface ___ y ___; no elijo ___ porque incumple ___.

---

## 5. Cómo leer inglés técnico de examen

### 5.1 Modificadores que cambian la respuesta

| Inglés | Sentido en el escenario | Consecuencia |
|---|---|---|
| must | requisito obligatorio | Una opción que no lo cumple queda descartada. |
| should | recomendación esperada | Busca la práctica preferida de Google Cloud. |
| first | primera acción | No saltes directamente a una corrección irreversible. |
| existing | ya existe | Evita proponer recreación si puede configurarse lo actual. |
| least privilege | mínimo privilegio | Prefiere el rol y alcance más estrechos suficientes. |
| minimal operational overhead | mínima carga operativa | Favorece servicios administrados cuando cumplen el requisito. |
| without downtime | sin interrupción | Descarta acciones que requieran detener el servicio. |
| most cost-effective | mejor relación costo/requisito | No significa simplemente el precio nominal más bajo. |
| highly available | alta disponibilidad | Busca tolerancia al dominio de fallo relevante. |
| durable | duradero | Se refiere a conservar datos, no necesariamente a disponibilidad de cómputo. |
| scalable | escalable | Debe adaptarse al crecimiento especificado. |
| immediately | de inmediato | Prefiere la acción con efecto y tiempo adecuados. |
| Choose two | selecciona dos | Dos opciones forman la respuesta; elegir una queda incompleto. |

### 5.2 Diferencias útiles

- **deploy**: poner una carga o versión en ejecución.
- **provision**: asignar o crear la capacidad o infraestructura necesaria.
- **configure**: ajustar propiedades o comportamiento.
- **manage**: operar a lo largo del tiempo.
- **monitor**: observar mediante métricas, logs, alertas o estado.
- **troubleshoot**: investigar sistemáticamente una falla.

---

## 6. Gramática de `gcloud`

La forma general es:

~~~text
gcloud GROUP SUBGROUP COMMAND POSITIONAL_ARGUMENTS --flags
~~~

Ejemplo:

~~~bash
gcloud services list --enabled --project=PROJECT_ID
~~~

Desglose:

- `gcloud`: herramienta.
- `services`: grupo.
- `list`: comando.
- `--enabled`: modo de la consulta.
- `--project=PROJECT_ID`: flag global que fija explícitamente el proyecto de esa invocación.

### 6.1 Verbos de lectura y verbos de cambio

| Normalmente consultan | Normalmente cambian estado |
|---|---|
| `list`, `describe`, `get-iam-policy`, `help` | `create`, `delete`, `update`, `set`, `add`, `remove`, `enable`, `disable` |

La tabla es una guía de seguridad, no una sustitución de la ayuda del comando. Lee `--help` antes de ejecutar una operación desconocida.

### 6.2 Comandos para descubrir la sintaxis

~~~bash
gcloud help
gcloud services list --help
gcloud topic filters
gcloud topic formats
gcloud cheat-sheet
~~~

Buenas prácticas:

- Usa primero la versión estable del comando. Recurre a `alpha` o `beta` solo si el requisito lo exige y comprendes sus limitaciones.
- Usa un `--project` explícito cuando haya riesgo de operar en el proyecto equivocado.
- No copies literalmente `PROJECT_ID`: es un marcador de posición.
- `--quiet` no hace un comando más seguro; elimina solicitudes de confirmación. Evítalo durante el aprendizaje, especialmente con comandos que cambian estado.
- `--format` transforma la presentación de la salida.
- `--filter` selecciona los recursos que cumplen una expresión.

### 6.3 Consola frente a CLI

| Necesidad | Consola | CLI |
|---|---|---|
| Explorar por primera vez | Visual y útil para descubrir servicios | Requiere conocer grupo/comando |
| Repetir exactamente una tarea | Más difícil de reproducir | Fácil de documentar y automatizar |
| Auditar la acción | Puede requerir registrar varios clics | El comando puede conservarse en un runbook |
| Procesar muchas filas | Limitado | `--filter` y `--format` ayudan |
| Evitar proyecto equivocado | Revisar selector superior | Usar `--project=PROJECT_ID` |

Para SiteOps Tracker, la consola funciona como el panel visual de la sala de operaciones; la CLI funciona como un runbook reproducible.

---

## 7. Mapa inicial para decidir entre servicios

Estas comparaciones son una orientación. Cada servicio tendrá una lección específica más adelante.

| Requisito dominante | Elección probable | Por qué | Por qué no elegir la alternativa de inmediato |
|---|---|---|---|
| Contenedor HTTP stateless, escalado automático, mínima operación | Cloud Run | Plataforma administrada y orientada a servicios/contenedores | GKE añade control y operación que el caso no pide; una VM exige administrar SO |
| APIs de Kubernetes, controladores, objetos y ecosistema Kubernetes | GKE | Proporciona Kubernetes administrado | Cloud Run no expone el control Kubernetes requerido |
| Control del sistema operativo, paquetes o kernel | Compute Engine | Da control de la VM y del guest OS | Cloud Run abstrae el SO |
| Función pequeña activada por evento | Cloud Run functions | Ajuste directo a una función basada en evento | Una VM o clúster agrega operación innecesaria |
| PostgreSQL transaccional administrado | Cloud SQL for PostgreSQL | Compatibilidad relacional y operación administrada | BigQuery es analítico; Firestore es documental |
| Analítica sobre grandes volúmenes | BigQuery | Motor analítico administrado | Cloud SQL no es la primera opción para un almacén analítico masivo |
| Objetos como evidencias, imágenes o exportaciones | Cloud Storage | Almacenamiento de objetos con clases y lifecycle | Persistent Disk es almacenamiento de bloque para cómputo; Cloud SQL almacena datos relacionales |
| Desacoplar productor y consumidor de eventos | Pub/Sub | Mensajería asíncrona y absorción de picos | Cloud DNS resuelve nombres; una réplica de base no crea una cola |
| Avisar por gasto | Cloud Billing budget and alerts | Observa umbrales de gasto | Una cuota limita uso/capacidad, no implementa el mismo control financiero |
| Dar lectura de logs a una persona | Rol predefinido específico en IAM | Reduce permisos al conjunto necesario | Owner o Editor son excesivos |
| Dar acceso a una aplicación | Service account adjunta al runtime | Identidad de workload sin usar la cuenta humana | Una clave JSON de larga duración aumenta riesgo y gestión |

### 7.1 Ejemplo completo con SiteOps Tracker

Escenario:

> SiteOps Tracker has a stateless Node.js API packaged as a container. Traffic is unpredictable. The team wants automatic scaling and does not want to manage servers or a Kubernetes cluster.

Aplicación de CLAVE:

1. **Clasifica:** dominio 2, compute.
2. **Localiza:** `stateless container`, `unpredictable traffic`, `does not want to manage servers or a Kubernetes cluster`.
3. **Aísla:** ejecutar HTTP en contenedor con escalado administrado.
4. **Valida:** Cloud Run satisface contenedor y baja operación.
5. **Elimina:** Compute Engine obliga a administrar VM; GKE añade Kubernetes que el escenario rechaza; Cloud SQL no ejecuta la API.

Decisión: **Cloud Run**.

---

## 8. Práctica guiada — Consola y CLI de solo lectura

### 8.1 Propósito

Construir un inventario mínimo antes de operar:

- identidad activa;
- configuración activa;
- proyecto correcto;
- estado del proyecto;
- APIs habilitadas;
- regiones y zonas visibles, únicamente si Compute Engine API ya está habilitada.

### 8.2 Reglas de seguridad

1. Usa un proyecto de laboratorio autorizado.
2. No pegues tokens, claves, correos reales ni resultados sensibles en documentos públicos.
3. No habilites APIs para esta práctica.
4. No asignes roles.
5. No crees ni borres recursos.
6. Si un comando pide activar una API, cancela y usa la alternativa conceptual.

Los comandos y flags de esta práctica se verificaron en la documentación oficial vigente consultada el 25 de septiembre de 2026.

### 8.3 Parte A — Verificación en la consola

1. Abre [Google Cloud Console](https://console.cloud.google.com/).
2. Revisa el selector de proyecto de la barra superior.
3. Anota por separado:
   - **Project name**: nombre visible;
   - **Project ID**: identificador único usado por la CLI;
   - **Project number**: identificador numérico generado.
4. Busca **APIs & Services** y abre **Enabled APIs & services**.
5. Observa, sin cambiar nada, qué APIs aparecen habilitadas.
6. Si tienes permiso, abre **Billing** y confirma únicamente si el proyecto está vinculado. No modifiques el vínculo.
7. Activa **Cloud Shell** desde la barra superior.

Evidencia:

| Comprobación | Resultado del alumno |
|---|---|
| Project ID revisado | |
| Project number localizado | |
| ¿La API de Compute Engine ya estaba habilitada? | Sí / No / Sin permiso |
| ¿Se pudo ver la facturación? | Sí / No / Sin permiso |

### 8.4 Parte B — Descubrir antes de ejecutar

Comprueba la versión y consulta la ayuda:

~~~bash
gcloud --version
gcloud help
gcloud services list --help
~~~

Resultado esperado:

- la versión instalada de Google Cloud CLI;
- la estructura general de `gcloud`;
- la sintaxis de `gcloud services list`, incluidos `--enabled`, `--available`, `--filter` y `--format`.

### 8.5 Parte C — Verificar identidad y configuración

~~~bash
gcloud auth list \
  --filter="status:ACTIVE" \
  --format="value(account)"

gcloud config list
~~~

Interpretación:

- el primer comando muestra la cuenta activa;
- el segundo muestra propiedades de la configuración activa, incluido el proyecto si está definido;
- no publiques esa salida sin anonimizarla.

Captura de evidencia:

| Pregunta | Respuesta |
|---|---|
| ¿Existe una cuenta activa? | |
| ¿Existe un proyecto configurado? | |
| ¿Hay región predeterminada? | |
| ¿Hay zona predeterminada? | |

### 8.6 Parte D — Guardar y validar el Project ID

~~~bash
PROJECT_ID="$(gcloud config get project)"
printf 'PROJECT_ID=%s\n' "$PROJECT_ID"
~~~

Si la salida está vacía o muestra `(unset)`, detente. No adivines un ID y no uses un proyecto real ajeno. Puedes seguir con la alternativa conceptual.

Si el valor es correcto:

~~~bash
gcloud projects describe "$PROJECT_ID" \
  --format="yaml(projectId,projectNumber,name,lifecycleState)"
~~~

Resultado esperado:

- `projectId` coincide con el proyecto autorizado;
- `projectNumber` es numérico;
- `lifecycleState` normalmente muestra `ACTIVE` para un proyecto utilizable.

### 8.7 Parte E — Consultar APIs habilitadas

~~~bash
gcloud services list \
  --enabled \
  --project="$PROJECT_ID" \
  --format="table(config.name,title)"
~~~

Para comprobar específicamente Compute Engine API:

~~~bash
gcloud services list \
  --enabled \
  --project="$PROJECT_ID" \
  --filter="config.name=compute.googleapis.com" \
  --format="value(config.name)"
~~~

Interpretación:

- si aparece `compute.googleapis.com`, la API ya está habilitada;
- si no aparece, no la habilites para esta práctica;
- una lista vacía no demuestra por sí sola un fallo: revisa proyecto, permisos y filtro.

### 8.8 Parte F — Consulta opcional de regiones y zonas

Ejecuta esta parte solo si `compute.googleapis.com` ya apareció en el paso anterior:

~~~bash
gcloud compute regions list \
  --project="$PROJECT_ID" \
  --limit=5 \
  --format="table(name,status)"

gcloud compute zones list \
  --project="$PROJECT_ID" \
  --filter="status=UP" \
  --limit=10 \
  --format="table(name,status)"
~~~

Resultado esperado:

- una muestra de regiones;
- una muestra de zonas cuyo estado coincide con el filtro;
- ningún recurso nuevo.

### 8.9 Evidencia final de práctica

Completa sin copiar datos sensibles:

| Elemento | Evidencia |
|---|---|
| Versión de CLI | |
| Proyecto validado | Sí / No |
| Estado del proyecto | |
| Cantidad aproximada de APIs habilitadas | |
| Compute Engine API ya habilitada | Sí / No |
| Consulta de regiones/zonas ejecutada | Sí / No / No aplicaba |
| Comando que volverías a consultar con `--help` | |
| Duda principal | |

---

## 9. Alternativa íntegra sin cuenta ni crédito

Analiza estas salidas ficticias:

~~~text
$ gcloud config list
[core]
account = learner@example.invalid
project = siteops-lab-123

$ gcloud projects describe siteops-lab-123 \
    --format="yaml(projectId,projectNumber,name,lifecycleState)"
lifecycleState: ACTIVE
name: SiteOps Lab
projectId: siteops-lab-123
projectNumber: '123456789012'

$ gcloud services list --enabled \
    --project=siteops-lab-123 \
    --format="table(config.name,title)"
NAME                               TITLE
cloudresourcemanager.googleapis.com Cloud Resource Manager API
iam.googleapis.com                  Identity and Access Management API
run.googleapis.com                  Cloud Run Admin API
~~~

Responde:

1. ¿Cuál es el Project ID?
2. ¿Cuál es el Project number?
3. ¿Qué identidad está activa?
4. ¿El proyecto parece utilizable según `lifecycleState`?
5. ¿Compute Engine API aparece habilitada?
6. ¿Sería correcto ejecutar `gcloud compute zones list` como parte obligatoria de esta práctica?
7. ¿Qué flag usarías para evitar depender del proyecto predeterminado?

### Solución de la alternativa

1. `siteops-lab-123`.
2. `123456789012`.
3. `learner@example.invalid`.
4. Sí, el estado mostrado es `ACTIVE`.
5. No aparece `compute.googleapis.com`.
6. No. Se debe omitir la parte opcional; no se habilita una API solo para completar una consulta de esta lección.
7. `--project=siteops-lab-123`.

---

## 10. Resultado esperado

Al finalizar debes tener:

- una explicación de CLAVE con tus propias palabras;
- un inventario mínimo de cuenta, configuración, proyecto y APIs, o su equivalente conceptual;
- las respuestas a 15 preguntas;
- puntuación total y por dominio;
- un registro de errores, incluidos los aciertos con baja confianza;
- al menos una regla de decisión para cada error.

No es necesario tener recursos desplegados.

---

## 11. Solución de problemas

### `gcloud: command not found`

Causa probable: la CLI no está instalada o no está en `PATH`.

Acción:

- usa Cloud Shell, que incluye Google Cloud CLI; o
- sigue la guía oficial de instalación para tu sistema.

No descargues binarios desde sitios no oficiales.

### No hay cuenta activa

Síntoma: el comando indica que no existe una cuenta activa.

Acción:

- en un laboratorio, sigue exactamente el inicio de sesión indicado por el proveedor;
- no autentiques una cuenta personal dentro de un laboratorio temporal salvo que las instrucciones lo pidan.

### El proyecto aparece como `(unset)`

Acción:

- verifica el Project ID en el selector de la consola;
- para esta práctica, puedes continuar usando `--project=PROJECT_ID` de forma explícita;
- no confundas Project name, Project ID y Project number.

### `PERMISSION_DENIED`

Posibles causas:

- cuenta activa incorrecta;
- proyecto incorrecto;
- falta el permiso de lectura requerido.

Acción:

1. Comprueba `gcloud auth list`.
2. Comprueba `gcloud config list`.
3. Comprueba el Project ID.
4. Solicita el rol mínimo necesario al administrador del laboratorio.

No te concedas Owner o Editor como solución rápida.

### `NOT_FOUND`

Posibles causas:

- Project ID mal escrito;
- recurso eliminado;
- referencia al proyecto equivocado.

Comprueba el identificador antes de suponer un problema de red.

### Compute Engine API no está habilitada

Omite la consulta de regiones y zonas. Habilitar una API cambia el estado del proyecto y está fuera de esta práctica.

### La salida es demasiado extensa

Usa `--limit` para reducir filas y `--format` para mostrar solo campos útiles. Usa `--filter` únicamente después de identificar el nombre real del campo.

### El filtro devuelve cero filas

Revisa:

1. proyecto;
2. permisos;
3. ortografía y mayúsculas del valor;
4. estructura de la salida sin filtro;
5. documentación de `gcloud topic filters`.

---

## 12. Costos y limpieza

### Impacto en costos

Los comandos de esta práctica son de consulta y no crean VMs, bases de datos, buckets, clústeres ni servicios. Por ello, la práctica no debería generar consumo de esos recursos.

Esto no significa que el proyecto completo no tenga costos preexistentes. Revisa la página de facturación solo si tienes autorización.

### Limpieza

Como no se crea ningún recurso, no hay recursos de nube que borrar.

Al terminar:

~~~bash
unset PROJECT_ID
~~~

Después:

- cierra Cloud Shell si ya no lo usarás;
- no deshabilites APIs preexistentes “para limpiar”;
- no elimines proyectos ni recursos que no creaste;
- no compartas capturas con correos, IDs o datos sensibles sin anonimizar.

---

## 13. Repaso activo espaciado

Responde sin mirar las soluciones. Explica cada respuesta en voz alta.

### 13.1 Días hábiles previos

#### Día 1 — Nube, proyectos y servicios

1. Ordena de mayor a menor alcance: recurso, carpeta, organización, proyecto.
2. ¿Por qué un proyecto es más que una carpeta visual?
3. ¿Qué diferencia hay entre un servicio de Google Cloud y un recurso creado con ese servicio?

#### Día 2 — Consola, Cloud Shell y `gcloud`

4. ¿Qué dos cosas debes verificar antes de ejecutar un comando?
5. ¿Cloud Shell y Cloud Console son lo mismo?
6. ¿Por qué `--project` puede reducir errores?

#### Día 3 — Costos y facturación

7. ¿Un presupuesto detiene automáticamente todo el gasto?
8. Explica la diferencia entre billing account, budget y quota.

#### Día 4 — Regiones, zonas y disponibilidad

9. ¿Dos VMs en la misma zona protegen frente a la pérdida completa de esa zona?
10. Menciona tres factores para elegir región además de la distancia.

### 13.2 Recuperación de hace 7 días

El plan comenzó el 21 de septiembre de 2026. El 18 de septiembre todavía no existía una lección programada en esta secuencia, así que no se inventa una recuperación.

Actividad de línea base: explica en 60 segundos qué esperas que haga un Associate Cloud Engineer y compara tu respuesta con los cuatro dominios de la guía.

### 13.3 Recuperación de hace 21 días

El 4 de septiembre de 2026 tampoco pertenece a esta secuencia. No hay una lección anterior que recuperar.

Actividad de línea base: escribe tres errores que deseas evitar durante esta nueva preparación. Para cada uno, define una conducta observable, por ejemplo: “Antes de ejecutar, verificaré cuenta y Project ID”.

### 13.4 Soluciones breves del repaso

1. Organización → carpeta → proyecto → recurso.
2. Es frontera para recursos, APIs, IAM, cuotas y facturación.
3. El servicio es la capacidad o API; el recurso es una instancia administrable creada mediante ella.
4. Identidad activa y proyecto/alcance objetivo.
5. No. La consola es interfaz web; Cloud Shell es un entorno de terminal accesible desde ella.
6. Fija el proyecto de la invocación y reduce dependencia de una configuración predeterminada.
7. No; alerta. No debe tratarse como tope automático.
8. Billing account paga; budget compara gasto y alerta; quota limita cantidad o tasa.
9. No.
10. Latencia, residencia de datos, disponibilidad del producto, costo, capacidad y diseño de recuperación son ejemplos válidos.

---

## 14. Glosario bilingüe

| English | Español | Uso |
|---|---|---|
| account | cuenta | Identidad autenticada o cuenta de facturación, según contexto |
| active configuration | configuración activa | Conjunto de propiedades que usa `gcloud` |
| availability | disponibilidad | Capacidad de permanecer accesible |
| billing account | cuenta de facturación | Entidad que paga proyectos vinculados |
| budget alert | alerta de presupuesto | Aviso al alcanzar umbrales de gasto |
| command | comando | Acción concreta de una herramienta |
| constraint | restricción | Condición que limita las opciones válidas |
| credential | credencial | Prueba usada para autenticar una identidad |
| deploy | desplegar | Poner una carga o versión en ejecución |
| describe | describir/consultar detalle | Mostrar metadatos de un recurso |
| distractor | distractor | Opción plausible pero incorrecta |
| downtime | tiempo de inactividad | Periodo sin servicio |
| enabled API | API habilitada | API disponible para uso en el proyecto |
| failure domain | dominio de fallo | Alcance que puede fallar de forma conjunta |
| flag | opción/indicador | Modificador como `--project` |
| high availability | alta disponibilidad | Diseño que reduce interrupciones |
| least privilege | mínimo privilegio | Solo permisos suficientes y necesarios |
| lifecycle state | estado de ciclo de vida | Estado actual de un proyecto o recurso |
| list | listar | Mostrar una colección de recursos |
| managed service | servicio administrado | Servicio cuya operación asume en parte Google |
| operational overhead | carga operativa | Trabajo recurrente de administrar la plataforma |
| principal | principal/identidad | Usuario, grupo, service account u otra identidad |
| project ID | ID de proyecto | Identificador global usado por APIs y CLI |
| project number | número de proyecto | Identificador numérico asignado por Google |
| provision | aprovisionar | Crear o asignar capacidad |
| quota | cuota | Límite de cantidad o tasa |
| requirement | requisito | Condición que la solución debe cumplir |
| resource | recurso | Entidad administrable en Google Cloud |
| role | rol | Conjunto de permisos IAM |
| service account | cuenta de servicio | Identidad para una carga de trabajo |
| troubleshoot | diagnosticar | Investigar una falla de manera sistemática |
| workload | carga de trabajo | Aplicación, proceso o servicio que se ejecuta |
| zone | zona | Dominio de despliegue dentro de una región |

---

## 15. Diagnóstico ACE — 15 preguntas originales

### Instrucciones

- Tiempo sugerido: 25 minutos.
- No consultes notas.
- Elige una respuesta salvo que se indique **Choose two**.
- Registra confianza: 1 = adivinanza, 2 = parcial, 3 = segura.
- Las preguntas son originales y no proceden de dumps.

### Hoja de respuestas

| # | Respuesta | Confianza 1–3 |
|---:|---|---:|
| 1 | | |
| 2 | | |
| 3 | | |
| 4 | | |
| 5 | | |
| 6 | | |
| 7 | | |
| 8 | | |
| 9 | | |
| 10 | | |
| 11 | | |
| 12 | | |
| 13 | | |
| 14 | | |
| 15 | | |

### Domain 1 — Setting up a cloud solution environment

#### Question 1

SiteOps Tracker needs separate development and production environments. Production access must be tightly controlled, and accidental changes in development must not consume production quotas. What should the team do?

A. Put all resources in one project and distinguish them only with labels.  
B. Use separate projects for development and production.  
C. Put development and production VMs in different zones of one project.  
D. Create two dashboards in Cloud Monitoring.

#### Question 2 — Choose two

Before running read-only inventory commands, an engineer wants to confirm the active identity and the active `gcloud` configuration. Which two commands should the engineer run?

A. `gcloud auth list`  
B. `gcloud config list`  
C. `gcloud projects create siteops-audit`  
D. `gcloud services enable compute.googleapis.com`  
E. `gcloud compute instances delete INSTANCE_NAME`

#### Question 3

The finance team wants email notifications when monthly Google Cloud spending reaches 50%, 80%, and 100% of a planned amount. They understand that resources should continue running. What should they configure?

A. A Cloud Billing budget with alert thresholds  
B. A Compute Engine CPU quota  
C. A VPC firewall rule  
D. A Cloud Storage lifecycle rule

### Domain 2 — Planning and implementing a cloud solution

#### Question 4

The SiteOps Tracker API is a stateless Node.js container that receives HTTPS requests. Traffic is unpredictable, and the team wants automatic scaling with minimal infrastructure management. Which service is the best fit?

A. Cloud Run  
B. Compute Engine with one manually managed VM  
C. GKE Standard with a manually managed node pool  
D. Cloud SQL

#### Question 5

A team must run custom Kubernetes controllers and use Kubernetes Deployments, Services, and StatefulSets. Which compute option should it choose?

A. Cloud Run  
B. GKE  
C. Cloud Run functions  
D. BigQuery

#### Question 6

SiteOps Tracker stores assets, findings, responsible users, and status history. The application requires PostgreSQL transactions, joins, and foreign keys. The team wants a managed database. Which service is the most appropriate starting choice?

A. Cloud SQL for PostgreSQL  
B. BigQuery  
C. Firestore  
D. Cloud Storage

#### Question 7

SiteOps Tracker stores evidence images. Files are frequently viewed for 30 days and rarely viewed afterward. The team wants object storage and automatic transitions based on age. What should it use?

A. Cloud Storage with an object lifecycle policy  
B. A boot Persistent Disk attached to a VM  
C. Cloud SQL binary columns as the primary object store  
D. Cloud DNS

#### Question 8

The API must accept audit events quickly while a separate worker processes them asynchronously. Traffic arrives in bursts, and the producer must be decoupled from the consumer. Which service should connect them?

A. Pub/Sub  
B. Cloud DNS  
C. A Cloud SQL read replica  
D. Cloud VPN

### Domain 3 — Ensuring the successful operation of a cloud solution

#### Question 9

The operations team wants a notification when the API's p95 latency remains above a threshold for five minutes. What should it configure?

A. A Cloud Monitoring alerting policy based on a latency metric  
B. A Cloud Billing budget  
C. A snapshot schedule  
D. An organization policy

#### Question 10

A new Cloud Run revision should initially receive 5% of production traffic while the current revision receives 95%. Clients must continue using the same URL. What should the team configure?

A. Cloud Run traffic splitting  
B. A different public DNS name for every client  
C. A managed instance group  
D. A second billing account

#### Question 11

The team wants selected application logs to be available for centralized SQL analysis in BigQuery. Which Google Cloud logging capability should it use?

A. A Log Router sink with BigQuery as the destination  
B. A VPC Flow Logs configuration only  
C. A Compute Engine machine image  
D. A billing export only

#### Question 12

Before a risky schema change, the team wants a recoverable copy of its Cloud SQL database. What is the most direct action?

A. Create an on-demand Cloud SQL backup  
B. Create a Compute Engine machine image  
C. Add a custom static route  
D. Change the Cloud Storage class of unrelated files

### Domain 4 — Configuring access and security

#### Question 13

An auditor needs to view logs in one project but must not modify resources or IAM. What is the best approach?

A. Grant the predefined Logs Viewer role at the project level  
B. Grant the Owner role at the organization level  
C. Grant the Editor basic role at the project level  
D. Create and email a service account key to the auditor

#### Question 14

A Cloud Run service needs read-only access to objects in one evidence bucket. Which approach best follows least privilege?

A. Attach a dedicated service account to the service and grant it object-viewer access on that bucket  
B. Grant Owner to the default compute service account  
C. Store a downloaded service account JSON key inside the container image  
D. Make the bucket public

#### Question 15

External contractors must use their existing corporate identities to access Google Cloud resources. The company does not want to create and maintain a separate Google identity for every contractor. Which capability is designed for this human workforce scenario?

A. Workforce Identity Federation  
B. Workload Identity Federation for an application workload  
C. Long-lived service account keys shared by the contractors  
D. Anonymous public access

---

## 16. Soluciones justificadas

### Question 1 — Correct answer: B

**Por qué:** proyectos separados crean fronteras claras para IAM, cuotas, APIs, facturación y ciclo de vida entre desarrollo y producción.

- A es incorrecta: las etiquetas ayudan a organizar, pero no crean esas fronteras.
- B es correcta.
- C es incorrecta: una zona es ubicación, no una frontera completa de administración.
- D es incorrecta: un dashboard observa; no aísla recursos ni permisos.

**Regla:** cuando el escenario pide aislamiento administrativo entre entornos, piensa primero en proyectos separados.

### Question 2 — Correct answers: A and B

**Por qué:** `gcloud auth list` permite ver cuentas con credenciales y cuál está activa; `gcloud config list` muestra propiedades de la configuración activa.

- A es correcta.
- B es correcta.
- C es incorrecta: crea un proyecto y cambia estado.
- D es incorrecta: habilita una API y cambia estado.
- E es incorrecta y destructiva: elimina una VM.

**Regla:** antes de operar, valida identidad y configuración con consultas, no con acciones de creación o eliminación.

### Question 3 — Correct answer: A

**Por qué:** un presupuesto con umbrales está diseñado para alertar sobre gasto. El escenario aclara que las cargas deben continuar.

- A es correcta.
- B es incorrecta: una cuota de CPU limita capacidad, no representa el presupuesto mensual.
- C es incorrecta: controla tráfico de red.
- D es incorrecta: administra la transición o eliminación de objetos.

**Regla:** budget alerts notifican; no supongas que detienen automáticamente los recursos.

### Question 4 — Correct answer: A

**Por qué:** Cloud Run ejecuta contenedores orientados a solicitudes con infraestructura administrada y escalado automático, apropiado para el requisito de baja operación.

- A es correcta.
- B es incorrecta: una VM única requiere administración y no satisface por sí sola el escalado pedido.
- C es incorrecta: GKE puede ejecutar el contenedor, pero añade administración Kubernetes que no se necesita.
- D es incorrecta: Cloud SQL es una base de datos, no el runtime de la API.

**Regla:** contenedor HTTP stateless + mínima operación suele favorecer Cloud Run.

### Question 5 — Correct answer: B

**Por qué:** el requisito explícito de controladores y objetos Kubernetes hace que GKE sea el ajuste directo.

- A es incorrecta: Cloud Run abstrae Kubernetes.
- B es correcta.
- C es incorrecta: una función no ofrece el plano Kubernetes requerido.
- D es incorrecta: BigQuery es un servicio analítico.

**Regla:** si el requisito depende de la API y los objetos de Kubernetes, elige GKE.

### Question 6 — Correct answer: A

**Por qué:** Cloud SQL for PostgreSQL conserva el modelo relacional solicitado y reduce la administración de la base.

- A es correcta.
- B es incorrecta: BigQuery está orientado a analítica, no a la base transaccional primaria de la aplicación.
- C es incorrecta: Firestore es documental y no proporciona el modelo PostgreSQL solicitado.
- D es incorrecta: Cloud Storage almacena objetos, no ofrece transacciones y joins PostgreSQL.

**Regla:** compatibilidad PostgreSQL transaccional administrada apunta primero a Cloud SQL for PostgreSQL.

### Question 7 — Correct answer: A

**Por qué:** Cloud Storage almacena objetos y sus reglas de lifecycle pueden cambiar clase o eliminar por edad conforme a la política definida.

- A es correcta.
- B es incorrecta: un boot disk depende de una VM y no es el servicio de objetos adecuado.
- C es incorrecta: guardar archivos grandes en la base transaccional añade una responsabilidad que Cloud Storage cubre directamente.
- D es incorrecta: Cloud DNS gestiona nombres.

**Regla:** objetos + política por edad = Cloud Storage + lifecycle.

### Question 8 — Correct answer: A

**Por qué:** Pub/Sub desacopla productor y consumidor y admite procesamiento asíncrono de mensajes.

- A es correcta.
- B es incorrecta: DNS resuelve nombres.
- C es incorrecta: una read replica escala lecturas de base; no es un sistema de mensajería.
- D es incorrecta: VPN proporciona conectividad, no una cola de eventos.

**Regla:** eventos asíncronos y desacoplamiento suelen apuntar a Pub/Sub.

### Question 9 — Correct answer: A

**Por qué:** Cloud Monitoring puede evaluar una métrica durante una ventana y activar notificaciones mediante una política de alerta.

- A es correcta.
- B es incorrecta: un presupuesto observa gasto, no latencia.
- C es incorrecta: un snapshot protege datos de disco.
- D es incorrecta: una organization policy impone restricciones de configuración.

**Regla:** métrica + umbral + duración + notificación = alerting policy.

### Question 10 — Correct answer: A

**Por qué:** el reparto de tráfico de Cloud Run permite distribuir porcentajes entre revisiones detrás del mismo servicio.

- A es correcta.
- B es incorrecta: obliga a los clientes a usar otra dirección y no es el mecanismo nativo solicitado.
- C es incorrecta: un MIG administra VMs, no revisiones de Cloud Run.
- D es incorrecta: la facturación no reparte tráfico.

**Regla:** canary entre revisiones de Cloud Run = traffic splitting.

### Question 11 — Correct answer: A

**Por qué:** Log Router usa sinks para dirigir entradas seleccionadas a destinos compatibles, incluido BigQuery.

- A es correcta.
- B es incorrecta: VPC Flow Logs registra flujos de red, pero no exporta por sí solo todos los logs de aplicación requeridos.
- C es incorrecta: una machine image captura estado de VM.
- D es incorrecta: una billing export contiene datos de facturación, no logs de aplicación.

**Regla:** exportación selectiva de logs a BigQuery = sink de Log Router.

### Question 12 — Correct answer: A

**Por qué:** un backup de Cloud SQL es la copia recuperable directamente relacionada con la instancia de base de datos.

- A es correcta.
- B es incorrecta: una machine image corresponde a Compute Engine, no a una instancia administrada de Cloud SQL.
- C es incorrecta: una ruta modifica networking.
- D es incorrecta: la clase de objetos no respalda la base.

**Regla:** protege el recurso con su mecanismo nativo de backup antes de un cambio riesgoso.

### Question 13 — Correct answer: A

**Por qué:** el rol predefinido Logs Viewer entrega permisos de lectura de logs sin conceder administración general del proyecto.

- A es correcta.
- B es incorrecta: Owner a nivel organización viola mínimo privilegio de forma extrema.
- C es incorrecta: Editor permite modificar numerosos recursos.
- D es incorrecta: una persona debe usar una identidad humana; distribuir una clave de service account crea riesgo y mala trazabilidad.

**Regla:** elige un rol predefinido específico y el alcance más estrecho que satisfaga la tarea.

### Question 14 — Correct answer: A

**Por qué:** una service account dedicada representa la carga, y el permiso se limita al bucket y a lectura de objetos.

- A es correcta.
- B es incorrecta: Owner es excesivo y usar una identidad predeterminada compartida reduce el aislamiento.
- C es incorrecta: incrustar una clave de larga duración en una imagen expone credenciales.
- D es incorrecta: hacer público el bucket elimina el control requerido.

**Regla:** workload → service account adjunta → rol mínimo sobre el recurso mínimo.

### Question 15 — Correct answer: A

**Por qué:** Workforce Identity Federation está orientado a personas externas que conservan identidades de un proveedor externo.

- A es correcta.
- B es incorrecta: Workload Identity Federation está orientado a workloads, no a usuarios humanos en este escenario.
- C es incorrecta: claves compartidas son de larga duración, difíciles de atribuir y de alto riesgo.
- D es incorrecta: acceso anónimo no autentica ni autoriza a cada contratista.

**Regla:** workforce = personas; workload = aplicaciones o cargas.

---

## 17. Puntuación y análisis por dominio

### 17.1 Distribución

| Dominio | Preguntas | Aciertos | Porcentaje |
|---|---|---:|---:|
| 1. Entorno | 1–3 | /3 | % |
| 2. Planificación e implementación | 4–8 | /5 | % |
| 3. Operación | 9–12 | /4 | % |
| 4. Acceso y seguridad | 13–15 | /3 | % |
| **Total** | **1–15** | **/15** | **%** |

Fórmula:

~~~text
porcentaje = (aciertos / preguntas) × 100
~~~

### 17.2 Lectura prudente del resultado

| Total | Interpretación para el plan |
|---:|---|
| 0–7 | Línea base inicial: registra conceptos desconocidos y sigue la secuencia sin saltar temas |
| 8–11 | Base en desarrollo: identifica los dos patrones de error más frecuentes |
| 12–13 | Buena línea base: revisa cada baja confianza y cada distractor dudoso |
| 14–15 | Diagnóstico alto: conserva el registro; no equivale a dominio completo ni a garantía de aprobar |

Con solo 3–5 preguntas por dominio, un porcentaje de dominio es muy sensible a un solo error. Úsalo para orientar el estudio, no para etiquetarte.

---

## 18. Ficha de errores

Registra también los aciertos con confianza 1: una adivinanza correcta sigue siendo una laguna.

Categorías recomendadas:

- **K:** knowledge gap — desconocimiento del concepto;
- **R:** reading — lectura incompleta de la consigna;
- **D:** distractor — opción plausible no eliminada;
- **C:** command — confusión de sintaxis o alcance;
- **P:** product confusion — servicios parecidos;
- **T:** time/rushing — prisa o cambio injustificado.

| Pregunta | Dominio | Mi respuesta | Correcta | Confianza | Categoría | Restricción que omití | Regla de decisión nueva | Acción y fecha |
|---:|---|---|---|---:|---|---|---|---|
| 1 | 1 | | B | | | | | |
| 2 | 1 | | A, B | | | | | |
| 3 | 1 | | A | | | | | |
| 4 | 2 | | A | | | | | |
| 5 | 2 | | B | | | | | |
| 6 | 2 | | A | | | | | |
| 7 | 2 | | A | | | | | |
| 8 | 2 | | A | | | | | |
| 9 | 3 | | A | | | | | |
| 10 | 3 | | A | | | | | |
| 11 | 3 | | A | | | | | |
| 12 | 3 | | A | | | | | |
| 13 | 4 | | A | | | | | |
| 14 | 4 | | A | | | | | |
| 15 | 4 | | A | | | | | |

### Ejemplo de corrección útil

| Campo | Ejemplo |
|---|---|
| Error | Elegí GKE en la pregunta 4 |
| Causa | Asocié “container” con Kubernetes y omití “minimal operational overhead” |
| Regla | Un contenedor no implica GKE; primero identifico cuánto control operativo exige el escenario |
| Recuperación | Explicar Cloud Run vs GKE sin notas y resolver dos escenarios nuevos |

Evita escribir solo “me confundí”. La ficha debe indicar **qué pista faltó** y **cómo decidirás la próxima vez**.

---

## 19. Criterios de autoevaluación

Marca cada criterio:

| Criterio | Sí | Parcial | No |
|---|:---:|:---:|:---:|
| Puedo aplicar CLAVE sin consultar la explicación | | | |
| Distingo `first`, `best`, `least privilege` y `Choose two` | | | |
| Puedo explicar la anatomía de un comando `gcloud` | | | |
| Verifico cuenta y proyecto antes de operar | | | |
| Distingo comandos de consulta de comandos de cambio | | | |
| Sé que `--quiet` elimina confirmaciones y no añade seguridad | | | |
| Puedo justificar Cloud Run vs GKE vs Compute Engine en un caso básico | | | |
| Puedo diferenciar budget y quota | | | |
| Puedo diferenciar identidad humana y service account | | | |
| Registré todos los errores y aciertos de baja confianza | | | |
| Puedo explicar por qué cada distractor diagnóstico falla | | | |

### Criterio de cierre de la lección

La lección queda trabajada cuando:

1. ejecutaste la práctica segura o completaste la alternativa conceptual;
2. resolviste las 15 preguntas antes de mirar las respuestas;
3. calculaste resultados por dominio;
4. registraste errores y baja confianza;
5. formulaste al menos una regla de decisión por error.

Recibir el archivo no implica haber completado estos pasos.

---

## 20. Tarjetas de recuerdo activo

Completa primero la respuesta y luego comprueba:

1. **Frente:** ¿Qué significa `least privilege`?  
   **Reverso:** Conceder solo los permisos suficientes, a la identidad correcta y en el alcance necesario.

2. **Frente:** ¿Qué compruebo antes de una acción CLI?  
   **Reverso:** Identidad activa, proyecto, alcance, verbo del comando y si cambia estado.

3. **Frente:** ¿Cuál es la diferencia entre `--filter` y `--format`?  
   **Reverso:** `--filter` selecciona elementos; `--format` cambia qué campos o representación se muestran.

4. **Frente:** ¿Un budget detiene el consumo?  
   **Reverso:** No por sí mismo; alerta según umbrales.

5. **Frente:** ¿Qué palabra obliga a ordenar acciones?  
   **Reverso:** `first`.

6. **Frente:** ¿Qué frase suele favorecer servicios administrados?  
   **Reverso:** `minimize operational overhead`.

7. **Frente:** ¿Qué diferencia a Workforce de Workload Identity Federation?  
   **Reverso:** Workforce representa personas externas; Workload representa aplicaciones/cargas.

8. **Frente:** ¿Qué significa un acierto con confianza 1?  
   **Reverso:** Posible laguna; debe registrarse y explicarse.

---

## 21. Próxima conexión con SiteOps Tracker

La siguiente lección diseñará la jerarquía de recursos para separar desarrollo y producción de SiteOps Tracker. Lleva estas decisiones:

- qué debe aislarse entre ambientes;
- qué políticas deberían heredarse;
- qué identidades necesitan acceso;
- qué costos y cuotas conviene separar;
- qué datos nunca deben publicarse en un portafolio.

---

## 22. Documentación oficial

Consultada y verificada el **25 de septiembre de 2026**:

- [Associate Cloud Engineer certification](https://cloud.google.com/learn/certification/cloud-engineer)
- [Associate Cloud Engineer — Standard Exam Guide (PDF)](https://services.google.com/fh/files/misc/associate_cloud_engineer_exam_guide_english.pdf)
- [gcloud CLI overview](https://docs.cloud.google.com/sdk/gcloud)
- [gcloud command reference](https://docs.cloud.google.com/sdk/gcloud/reference)
- [gcloud config list](https://docs.cloud.google.com/sdk/gcloud/reference/config/list)
- [gcloud config get](https://docs.cloud.google.com/sdk/gcloud/reference/config/get)
- [gcloud auth list](https://docs.cloud.google.com/sdk/gcloud/reference/auth/list)
- [gcloud projects describe](https://docs.cloud.google.com/sdk/gcloud/reference/projects/describe)
- [gcloud services list](https://docs.cloud.google.com/sdk/gcloud/reference/services/list)
- [gcloud compute regions list](https://docs.cloud.google.com/sdk/gcloud/reference/compute/regions/list)
- [gcloud compute zones list](https://docs.cloud.google.com/sdk/gcloud/reference/compute/zones/list)
- [gcloud topic filters](https://docs.cloud.google.com/sdk/gcloud/reference/topic/filters)
- [gcloud topic formats](https://docs.cloud.google.com/sdk/gcloud/reference/topic/formats)
- [Use Cloud Shell](https://docs.cloud.google.com/shell/docs/using-cloud-shell)

---

## 23. Registro personal de cierre

| Campo | Respuesta |
|---|---|
| Tiempo total | |
| Práctica realizada | Consola/CLI / Conceptual |
| Resultado total | /15 |
| Dominio más débil hoy | |
| Patrón de error principal | |
| Regla de decisión más útil | |
| Comando que puedo explicar sin notas | |
| Duda pendiente | |
| Próxima acción concreta | |
