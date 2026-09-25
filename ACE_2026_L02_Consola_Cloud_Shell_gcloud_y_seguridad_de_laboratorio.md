# ACE 2026 — Lección 02: Consola, Cloud Shell, `gcloud` y seguridad de laboratorio

**Fecha:** 22 de septiembre de 2026  
**Fecha de regeneración:** 24 de septiembre de 2026  
**Ruta del plan:** Base y sección 1.1 — *Setting up cloud projects and accounts*  
**Práctica del calendario:** Navegar un proyecto de laboratorio y ejecutar consultas de proyecto, cuenta y configuración.  
**Duración sugerida:** 90–120 minutos  
**Idioma:** explicación en español; términos de producto y preguntas de examen en inglés.

> Esta lección usa únicamente consultas de lectura. No habilites APIs, no cambies IAM, no crees recursos y no borres proyectos. Trabaja solo en un proyecto que te pertenezca o que el laboratorio te haya asignado.

> **Proyecto de práctica:** SiteOps Tracker es un caso completamente ficticio de portafolio para auditar infraestructura de TI en sedes ficticias. Usa React + TypeScript, Node.js + TypeScript y PostgreSQL; no representa ni reutiliza datos, arquitectura o requisitos de proyectos internos o privados.

---

## 1. Objetivo de aprendizaje

Al terminar la sesión podrás:

1. distinguir **Google Cloud console**, **Cloud Shell** y **Google Cloud CLI (`gcloud`)**;
2. explicar la diferencia entre cuenta activa, configuración activa y proyecto predeterminado;
3. leer la estructura de un comando `gcloud`;
4. comprobar de forma segura **quién eres**, **qué configuración usas** y **en qué proyecto estás** antes de operar;
5. consultar los proyectos visibles y describir uno autorizado sin modificarlo;
6. elegir la herramienta apropiada para una tarea de examen;
7. reconocer prácticas peligrosas en un laboratorio y detenerte antes de afectar el proyecto equivocado.

La recepción del archivo no demuestra dominio. El dominio se comprueba al ejecutar o simular la práctica, explicar los resultados y justificar las decisiones.

---

## 2. Conexión con la guía oficial ACE

La guía oficial define al Associate Cloud Engineer como alguien que ejecuta tareas comunes de plataforma para desplegar, proteger, supervisar y mantener soluciones. Esta lección construye la herramienta de trabajo necesaria para la sección **1.1, Setting up cloud projects and accounts**.

Hoy no crearás la jerarquía ni asignarás roles. Primero aprenderás a inspeccionar el contexto con seguridad. Esta comprobación previa reduce errores cuando más adelante debas:

- trabajar con proyectos y cuentas;
- habilitar APIs;
- configurar observabilidad;
- revisar cuotas, redes o disponibilidad regional;
- operar recursos mediante consola o CLI.

**Idea de examen:** antes de ejecutar una acción, confirma identidad, destino y alcance. Muchas respuestas incorrectas saltan directamente a otorgar `Owner`, cambiar el proyecto o usar una herramienta más compleja de la necesaria.

---

## 3. Prerrequisitos, explicados desde cero

### 3.1 Una cuenta de Google

Es la identidad con la que inicias sesión. Puede ser una cuenta personal, corporativa o temporal de laboratorio. La cuenta responde a **“¿quién hace la solicitud?”**.

No confundas una cuenta de usuario con un proyecto. Una misma cuenta puede ver varios proyectos y un proyecto puede permitir el acceso de varias identidades.

### 3.2 Un proyecto autorizado

Un **project** es un contenedor administrativo para recursos, APIs, permisos, cuotas y facturación. Para esta práctica basta un proyecto de laboratorio, un sandbox o un proyecto propio sin datos sensibles.

Necesitarás reconocer:

- **Project name:** etiqueta legible y modificable; no es la referencia más segura para comandos.
- **Project ID:** identificador globalmente único que aparece con frecuencia en CLI y APIs.
- **Project number:** número único asignado por Google, usado por algunos servicios e identidades administradas.

### 3.3 Un navegador moderno

La consola es una aplicación web. Cloud Shell se abre desde ella, así que no necesitas instalar `gcloud` localmente para completar la práctica.

### 3.4 Permisos de lectura

Los comandos de hoy necesitan que la cuenta pueda ver el proyecto. Si recibes `PERMISSION_DENIED`, no significa que debas darte `Owner`; significa que debes revisar la cuenta, el proyecto y el permiso mínimo necesario.

### 3.5 Si no tienes cuenta, proyecto o crédito

Usa la **alternativa conceptual íntegra** de la sección 10. Podrás interpretar salidas simuladas, detectar un contexto inseguro y responder todas las comprobaciones sin iniciar sesión ni gastar dinero.

---

## 4. Mapa mental: tres superficies, un mismo entorno

Imagina que Google Cloud es un gran centro de operaciones:

- La **Google Cloud console** es el tablero visual con menús, filtros y formularios.
- **Cloud Shell** es una estación de trabajo temporal que Google prepara para ti.
- **`gcloud`** es el lenguaje de órdenes que puedes usar desde esa estación o desde una instalación local.

No son tres nubes distintas. Son tres maneras de interactuar con los servicios de Google Cloud.

### 4.1 Google Cloud console

La consola web es adecuada para:

- descubrir servicios;
- observar relaciones visuales;
- revisar pocos recursos;
- completar una tarea ocasional con un formulario;
- confirmar visualmente el proyecto seleccionado.

Ventajas: es guiada y muestra contexto. Limitaciones: repetir muchos clics es lento, difícil de auditar y propenso a variaciones humanas.

### 4.2 Cloud Shell

Cloud Shell proporciona una VM temporal con Linux basado en Debian y herramientas preinstaladas, incluido `gcloud`. Google posee y administra esa VM; no aparece como una VM dentro de tus proyectos.

Hechos que conviene memorizar:

- la sesión y la VM son efímeras;
- tras una hora de inactividad, la sesión termina y la VM se descarta;
- el directorio `$HOME` estándar dispone de **5 GB gratuitos y persistentes** entre sesiones y proyectos;
- la organización puede impedir el uso de Cloud Shell;
- la primera llamada que requiera credenciales puede pedir autorización;
- al iniciar Cloud Shell, el proyecto activo de la consola se propaga a la configuración de `gcloud` y a `GOOGLE_CLOUD_PROJECT`;
- que el proyecto se propague **no significa** que la VM de Cloud Shell pertenezca a ese proyecto.

**Analogía:** la VM es un escritorio prestado que se desmonta; `$HOME` es un casillero personal que permanece. No dejes secretos en el escritorio ni en el casillero.

### 4.3 Google Cloud CLI y el comando `gcloud`

Google Cloud CLI es el conjunto de herramientas de línea de comandos. `gcloud` es su interfaz principal para administrar autenticación, configuración y numerosos recursos.

Es adecuada para:

- repetir operaciones de forma consistente;
- filtrar y formatear resultados;
- documentar pasos reproducibles;
- automatizar tareas mediante scripts;
- trabajar con muchos recursos.

En Cloud Shell ya está instalada. La documentación oficial consultada el **22 de septiembre de 2026** mostraba la versión actual **586.0.0**; la versión seguirá cambiando, por lo que debes comprobarla con `gcloud version` y no memorizarla para el examen.

---

## 5. El triángulo de seguridad: quién, dónde y cómo

Antes de ejecutar una modificación, responde tres preguntas.

| Pregunta | Concepto | Ejemplo | Comprobación de hoy |
|---|---|---|---|
| ¿Quién soy? | Active account | `student-...@example.com` | `gcloud auth list` |
| ¿Dónde operaré? | Project | `siteops-tracker-lab` | `gcloud config list project` |
| ¿Con qué perfil y valores? | Active configuration | `default` o `lab` | `gcloud config configurations list` y `gcloud config list` |

### 5.1 Cuenta activa

`gcloud` puede almacenar credenciales de varias cuentas, pero utiliza una sola cuenta activa a la vez. Esa identidad hace la solicitud y sus permisos determinan qué puede ver o modificar.

**Analogía:** la cuenta es el gafete de la persona. Tener acceso al edificio no significa tener acceso a todas las salas.

### 5.2 Proyecto predeterminado

La propiedad `core/project` indica el proyecto que muchos comandos usarán cuando no se especifique uno. No otorga permisos y no crea una relación de propiedad.

**Analogía:** es la dirección escrita en el formulario. Cambiar la dirección no cambia tu identidad ni te entrega las llaves del lugar.

### 5.3 Configuración activa

Una **configuration** es un conjunto nombrado de propiedades de `gcloud`; funciona como un perfil. Puede guardar, entre otras cosas, cuenta, proyecto, región o zona predeterminados.

Una configuración:

- no es un proyecto;
- no es una cuenta;
- no concede IAM;
- solo reúne valores que influyen en los comandos.

**Analogía:** es una carpeta de trabajo que contiene el gafete elegido, la dirección y preferencias. Si abres la carpeta equivocada, puedes apuntar al entorno equivocado aunque el comando esté bien escrito.

### 5.4 Opciones por invocación

Una opción global como `--project=PROJECT_ID` puede reemplazar la propiedad de proyecto para **esa ejecución**. No modifica permanentemente la configuración activa.

Esto es útil para hacer explícito el destino, pero no sustituye la verificación de la cuenta ni de los permisos.

---

## 6. Cómo leer un comando `gcloud`

Patrón general:

```text
gcloud [RELEASE_LEVEL] GROUP [SUBGROUP] COMMAND [POSITIONAL_ARGUMENTS] [FLAGS]
```

Ejemplo:

```bash
gcloud projects list --sort-by=projectId --limit=5
```

Despiece:

| Parte | Valor | Función |
|---|---|---|
| Herramienta | `gcloud` | Invoca Google Cloud CLI. |
| Grupo | `projects` | Selecciona el área de proyectos. |
| Comando | `list` | Solicita una lista. |
| Flag | `--sort-by=projectId` | Ordena por el campo `projectId`. |
| Flag | `--limit=5` | Limita la salida a cinco elementos. |

Segundo ejemplo:

```bash
gcloud projects describe siteops-tracker-lab
```

Aquí `siteops-tracker-lab` es un **positional argument**: identifica la entidad sobre la que opera `describe`.

### 6.1 `list` frente a `describe`

- `list` responde “¿qué elementos puedo enumerar?”.
- `describe` responde “¿qué metadatos tiene este elemento concreto?”.

Que un proyecto no aparezca en `list` no prueba por sí solo que no exista. Puede haber límites de permisos o casos particulares de identidades de servicio. `describe` también falla si el proyecto no existe o si la cuenta activa no puede verlo.

### 6.2 `--filter`, `--format`, `--limit` y `--sort-by`

- `--filter` selecciona registros según una condición.
- `--format` cambia la presentación de la salida.
- `--limit` limita cuántos registros devuelve el comando.
- `--sort-by` ordena por uno o más campos.

Ejemplo seguro para mostrar solo la cuenta activa:

```bash
gcloud auth list --filter=status:ACTIVE --format="value(account)"
```

### 6.3 Por qué hoy no usarás `--quiet`

`--quiet` desactiva preguntas interactivas y acepta valores predeterminados cuando existen. Es útil en automatización controlada, pero durante el aprendizaje puede ocultar una última oportunidad de detectar una acción peligrosa.

Regla de laboratorio: **no añadas `--quiet` a un comando que modifica o elimina recursos hasta comprender exactamente el efecto**.

---

## 7. Ejemplo guiado: SiteOps Tracker

Una empresa ficticia mantiene **SiteOps Tracker**, una aplicación de portafolio para auditar infraestructura de TI en sedes ficticias y conservar hallazgos, estados, evidencias e historial. Tiene dos proyectos de Google Cloud:

- `siteops-tracker-dev`: pruebas y desarrollo;
- `siteops-tracker-prod`: producción.

Ana recibe una cuenta temporal de laboratorio para inspeccionar `siteops-tracker-dev`. En la consola ve `siteops-tracker-prod`, porque lo consultó ayer. Abre Cloud Shell y está a punto de pegar un comando de creación.

El flujo correcto es:

1. detenerse antes de ejecutar;
2. confirmar la cuenta activa;
3. confirmar la configuración activa;
4. confirmar el proyecto configurado;
5. describir el ID autorizado;
6. comparar el ID devuelto con las instrucciones del laboratorio;
7. solo entonces considerar una acción posterior.

El flujo incorrecto sería asumir que “si Cloud Shell abrió, todo está bien”. Cloud Shell hereda el proyecto de la consola al inicio, pero un profesional verifica el contexto, especialmente cuando existen entornos `dev` y `prod` parecidos.

### Registro mínimo de preflight

| Campo | Valor de ejemplo enmascarado |
|---|---|
| Cuenta activa | `student-***@example.com` |
| Configuración activa | `default` |
| Proyecto esperado | `siteops-tracker-dev` |
| Proyecto observado | `siteops-tracker-dev` |
| Comando previsto | Consulta de metadatos |
| ¿Produce cambios? | No |
| Decisión | Continuar con la consulta |

Nunca pegues en una ficha pública correos completos, tokens, claves, cookies o salidas sensibles.

---

## 8. Decisión entre herramientas y servicios

| Necesidad | Elección recomendada | Por qué | Por qué no las alternativas |
|---|---|---|---|
| Explorar visualmente un proyecto por primera vez | Google Cloud console | Ofrece navegación, formularios y contexto visual. | `gcloud` exige conocer grupos y campos; REST sería innecesariamente manual. |
| Usar CLI sin instalar nada | Cloud Shell + `gcloud` | Entorno temporal preconfigurado y autenticado. | Instalación local requiere mantenimiento; una VM propia agrega costo y administración. |
| Repetir una consulta o documentar pasos | `gcloud` | Comandos reproducibles, filtrables y aptos para scripts. | Clics de consola son más difíciles de repetir exactamente. |
| Integrar llamadas dentro de una aplicación | Client library o REST API | Diseñadas para código de aplicación, control de errores y autenticación programática. | Automatizar una app invocando `gcloud` como subproceso suele ser menos robusto. |
| Declarar infraestructura reproducible | Terraform u otra herramienta IaC aprobada | Describe estado deseado y permite revisión. | Crear muchos recursos a mano en consola o con comandos aislados aumenta deriva. |
| Consultar identidad, configuración y proyecto hoy | `gcloud` en Cloud Shell | Es directo, gratuito y de solo lectura. | Terraform, Kubernetes y Gemini no aportan valor a esta inspección básica. |

### Trampa frecuente de examen

“Automatizable” no significa automáticamente “usa Terraform”. Para una consulta rápida y repetible, `gcloud` suele ser suficiente. Terraform es apropiado cuando debes administrar estado deseado de infraestructura, no para mostrar qué cuenta está activa.

---

## 9. Práctica principal: inspección segura con consola y CLI

### 9.1 Alcance y regla de parada

Esta práctica solo consulta información. Detente si:

- el proyecto visible no coincide con el proyecto autorizado;
- la cuenta activa no es la cuenta del laboratorio;
- un comando propone crear, actualizar o eliminar algo;
- la guía del laboratorio prohíbe un paso;
- aparece una solicitud de compartir un token o una clave.

### 9.2 Parte A — Consola

1. Abre [Google Cloud console](https://console.cloud.google.com/).
2. Inicia sesión únicamente con la cuenta autorizada para el laboratorio.
3. Usa el selector de proyecto de la barra superior.
4. Selecciona el proyecto asignado, no un proyecto corporativo ni de producción.
5. Registra localmente y de forma enmascarada:
   - nombre del proyecto;
   - Project ID;
   - Project number, si aparece;
   - cuenta con la que iniciaste sesión.
6. Compara el Project ID con las instrucciones del laboratorio.
7. Selecciona **Activate Cloud Shell** en la consola. Se abrirá un panel de terminal.
8. Si la primera llamada a Google Cloud solicita autorización, lee el cuadro y autoriza solo si la cuenta y el proyecto son correctos.

> No ejecutes `gcloud auth login` dentro de un laboratorio que ya te entregó credenciales temporales, salvo que sus instrucciones lo pidan expresamente. Podrías reemplazar el contexto previsto con tu cuenta personal.

### 9.3 Parte B — Comprobar la herramienta

Ejecuta:

```bash
gcloud version
```

**Observa:** versión de Google Cloud CLI y componentes instalados. No es necesario que coincida para siempre con la versión mencionada en esta lección.

### 9.4 Parte C — Comprobar quién eres

Primero muestra las cuentas con credenciales disponibles:

```bash
gcloud auth list
```

Después muestra únicamente la cuenta activa:

```bash
gcloud auth list --filter=status:ACTIVE --format="value(account)"
```

**Control:** compara el resultado con la cuenta del laboratorio. Enmascara el correo antes de guardarlo o compartirlo.

### 9.5 Parte D — Comprobar el perfil y sus propiedades

Lista las configuraciones nombradas:

```bash
gcloud config configurations list
```

La columna activa identifica el perfil en uso. Después consulta sus propiedades establecidas:

```bash
gcloud config list
```

Consulta solamente la propiedad de proyecto:

```bash
gcloud config list project
```

**Control:** el valor de `core/project` debe ser el Project ID autorizado. Si está vacío o es distinto, detente. No lo cambies en esta lección; documenta la discrepancia.

### 9.6 Parte E — Ver proyectos accesibles

Ejecuta la consulta oficial de ejemplo con límite:

```bash
gcloud projects list --sort-by=projectId --limit=5
```

**Observa:** el comando lista proyectos activos visibles para la cuenta bajo las condiciones descritas en la referencia. No concluyas que tienes permisos de edición solo porque ves un proyecto.

### 9.7 Parte F — Describir exactamente un proyecto autorizado

Asigna el ID autorizado a una variable local. Sustituye solo el texto de ejemplo; no uses `<` ni `>`:

```bash
export PROJECT_ID="replace-with-authorized-project-id"
```

Comprueba lo que escribiste:

```bash
printf '%s\n' "$PROJECT_ID"
```

Si coincide con las instrucciones, consulta sus metadatos:

```bash
gcloud projects describe "$PROJECT_ID"
```

Busca campos equivalentes a Project ID, Project number, nombre, estado del ciclo de vida y relación superior si tu permiso permite verla.

Al terminar, elimina solo la variable de esta sesión:

```bash
unset PROJECT_ID
```

`unset` no elimina el proyecto. Solo retira la variable de la shell actual.

### 9.8 Evidencia de la práctica

Completa sin copiar secretos:

| Evidencia | Tu registro |
|---|---|
| Versión de `gcloud` observada | |
| Cuenta activa enmascarada | |
| Configuración activa | |
| Project ID esperado, enmascarado si es sensible | |
| Project ID configurado | |
| ¿Coinciden? | Sí / No |
| Resultado de `projects describe` | Éxito / Error |
| Si hubo error, categoría | Cuenta / Proyecto / Permiso / Red / Otro |
| Una duda para revisar | |

---

## 10. Alternativa conceptual completa, sin cuenta ni crédito

Lee esta salida simulada:

```text
$ gcloud auth list
Credentialed Accounts
ACTIVE: *
ACCOUNT: student-***@example.com

$ gcloud config configurations list
NAME: default
IS_ACTIVE: True
ACCOUNT: student-***@example.com
PROJECT: siteops-tracker-prod

$ gcloud config list project
[core]
project = siteops-tracker-prod

El laboratorio autoriza: siteops-tracker-dev
```

Responde:

1. ¿Cuál es la cuenta activa? `student-***@example.com`.
2. ¿Cuál es la configuración activa? `default`.
3. ¿Cuál es el proyecto predeterminado? `siteops-tracker-prod`.
4. ¿Coincide con el proyecto autorizado? No.
5. ¿Debe ejecutarse una creación? No; hay que detenerse y corregir o aclarar el contexto según las instrucciones.
6. ¿Cambiar de proyecto concedería permisos? No. Los permisos dependen de IAM para la identidad.
7. ¿La VM de Cloud Shell pertenece a `siteops-tracker-prod`? No; Google administra la VM temporal y no aparece en ese proyecto.

Ahora interpreta esta consulta:

```text
$ gcloud projects describe siteops-tracker-dev
ERROR: (gcloud.projects.describe) User [...] does not have permission to access projects instance [...] or it may not exist
```

Diagnóstico seguro:

1. verificar que el ID esté escrito exactamente;
2. verificar la cuenta activa;
3. confirmar que el laboratorio siga vigente;
4. solicitar únicamente el permiso necesario al administrador o reiniciar el laboratorio según sus instrucciones;
5. no asignarse `Owner`, no probar cuentas ajenas y no crear otro proyecto para “evitar” el error.

Esta alternativa cubre el mismo objetivo de lectura, diagnóstico y decisión aunque no produzca una salida real.

---

## 11. Resultado esperado

Al completar la práctica debes poder producir una explicación semejante a esta:

> “Abrí Cloud Shell desde el proyecto autorizado. Verifiqué la versión de `gcloud`, la cuenta activa, la configuración activa y `core/project`. Consulté hasta cinco proyectos accesibles y describí por ID solo el proyecto de laboratorio. No habilité APIs, no cambié configuración persistente y no creé recursos.”

Resultados válidos:

- todas las comprobaciones coinciden y `describe` devuelve metadatos;
- detectas una discrepancia y te detienes antes de operar;
- recibes un error, lo clasificas correctamente y aplicas el camino de solución sin elevar privilegios de forma excesiva;
- completas la alternativa conceptual si no cuentas con acceso.

Un resultado “sin errores” no prueba por sí solo que comprendiste. Debes poder explicar qué respondió cada comando y qué no garantiza.

---

## 12. Solución de problemas

### 12.1 Cloud Shell no aparece o está deshabilitado

**Causa posible:** la organización controla el acceso a Cloud Shell.  
**Acción:** confirma la política con el administrador o usa una instalación local aprobada. No intentes eludir la restricción.

### 12.2 Aparece una solicitud de autorización

**Causa:** la primera llamada a una API o herramienta con credenciales puede requerir que autorices Cloud Shell.  
**Acción:** verifica cuenta y proyecto, lee el alcance y autoriza solo en el entorno esperado.

### 12.3 `gcloud auth list` no muestra una cuenta activa

**Causa posible:** la autenticación no se completó o la sesión expiró.  
**Acción:** en un laboratorio, sigue sus instrucciones o reinícialo. En un entorno propio, usa el método de autenticación aprobado. No introduzcas una cuenta personal en un entorno corporativo sin autorización.

### 12.4 El proyecto de `gcloud config list project` está vacío

**Significado:** no hay un proyecto predeterminado establecido en la configuración activa.  
**Acción:** para hoy, registra el hallazgo y usa el ID explícito autorizado solo en la consulta `describe`. En una tarea posterior, podrás establecer el proyecto cuando el plan lo indique.

### 12.5 La consola muestra un proyecto y `gcloud` otro

**Causa posible:** cambiaste el selector después de abrir Cloud Shell, activaste otra configuración o abriste una sesión con contexto anterior.  
**Acción:** detente, vuelve a verificar ambos contextos y sigue las instrucciones del laboratorio. No asumas sincronización continua.

### 12.6 `PERMISSION_DENIED`

**Causas posibles:** cuenta incorrecta, proyecto incorrecto, laboratorio expirado o permiso insuficiente.  
**Acción:** verifica identidad e ID; después solicita el rol mínimo requerido. `Owner` no es una solución predeterminada.

### 12.7 `NOT_FOUND`

**Causas posibles:** Project ID mal escrito, proyecto eliminado o mensaje que oculta falta de visibilidad.  
**Acción:** copia el ID desde la fuente autorizada, distingue nombre de ID y confirma con el administrador del laboratorio.

### 12.8 El proyecto esperado no aparece en `projects list`

**Causa posible:** permisos, identidad activa o condiciones especiales de una cuenta de servicio.  
**Acción:** no concluyas inmediatamente que el proyecto no existe. Verifica la cuenta e intenta `projects describe` con el ID autorizado.

### 12.9 La versión local de `gcloud` es antigua

**Acción:** consulta la guía oficial de instalación y actualización. En Cloud Shell, la imagen se actualiza periódicamente. No fuerces una actualización durante un laboratorio cronometrado salvo que sea requisito.

### 12.10 Compartiste una salida con correo completo

**Acción:** elimina o edita la publicación si es posible y enmascara la identidad. Si expusiste un token o una clave, trátalo como incidente: revoca o rota la credencial según el procedimiento de seguridad. Un correo no es un secreto, pero sigue siendo dato personal.

---

## 13. Seguridad de laboratorio: reglas que evitan incidentes

1. Usa solo la cuenta temporal o autorizada.
2. Lee el paso completo antes de copiar un comando.
3. Verifica cuenta, configuración y proyecto antes de cada cambio.
4. Prefiere primero consultas como `list`, `describe` y `get`.
5. No compartas tokens, claves privadas, cookies, contraseñas ni archivos de credenciales.
6. No ejecutes `gcloud auth print-access-token` para “demostrar” acceso.
7. No otorgues `Owner` para resolver un error de permisos.
8. No uses `sudo` en Cloud Shell salvo necesidad documentada; el privilegio sobre la VM temporal no concede privilegios sobre Google Cloud.
9. Evita `--quiet` mientras aprendes acciones con impacto.
10. No uses proyectos de producción para practicar.
11. No borres el proyecto al limpiar un laboratorio salvo instrucción explícita del proveedor.
12. Enmascara IDs internos y correos antes de compartir capturas o salidas.

---

## 14. Costos y limpieza

### Impacto en costos

- Los comandos de consulta de esta práctica no crean recursos facturables.
- El uso estándar de Cloud Shell y sus 5 GB de `$HOME` se ofrece sin costo según la documentación consultada.
- Ver proyectos y metadatos no significa que un proyecto no tenga otros recursos facturables; hoy no los crearás ni inspeccionarás en detalle.
- Instalar herramientas, habilitar APIs o crear recursos queda fuera de esta práctica y podría cambiar el costo.

### Limpieza

1. Ejecuta `unset PROJECT_ID` si aún no lo hiciste.
2. Cierra Cloud Shell cuando termines; la VM temporal se descarta tras la inactividad indicada por Google.
3. No elimines el proyecto.
4. No desactives APIs.
5. Si creaste notas locales en `$HOME`, recuerda que pueden persistir entre sesiones; elimina únicamente tus archivos de práctica si contienen datos sensibles.
6. Cierra la sesión del laboratorio según las instrucciones del proveedor.

No existe un recurso de nube creado por esta lección que debas borrar.

---

## 15. Glosario bilingüe

| English term | Término en español | Explicación breve |
|---|---|---|
| Google Cloud console | Consola de Google Cloud | Interfaz web para administrar Google Cloud. |
| Cloud Shell | Cloud Shell | Entorno de shell temporal y preconfigurado administrado por Google. |
| Google Cloud CLI | CLI de Google Cloud | Conjunto de herramientas de línea de comandos. |
| `gcloud` | comando `gcloud` | Interfaz principal para numerosos servicios y la configuración del CLI. |
| Active account | Cuenta activa | Identidad que `gcloud` usa para las solicitudes. |
| Credentialed account | Cuenta con credenciales | Cuenta cuyas credenciales conoce el CLI; no necesariamente está activa. |
| Configuration | Configuración / perfil | Conjunto nombrado de propiedades de `gcloud`. |
| Property | Propiedad | Valor persistente en una configuración, como `core/project`. |
| Project | Proyecto | Contenedor administrativo de recursos, IAM, APIs, cuotas y facturación. |
| Project ID | ID de proyecto | Identificador globalmente único usado habitualmente por CLI y APIs. |
| Project number | Número de proyecto | Identificador numérico único asignado por Google. |
| Command group | Grupo de comandos | Categoría jerárquica, por ejemplo `projects`. |
| Command | Comando | Acción, por ejemplo `list` o `describe`. |
| Positional argument | Argumento posicional | Entidad o valor identificado por su posición en el comando. |
| Flag / option | Opción / bandera | Modifica el comportamiento, por ejemplo `--limit`. |
| Filter | Filtro | Condición que selecciona resultados. |
| Output format | Formato de salida | Presentación solicitada con `--format`. |
| Read-only query | Consulta de solo lectura | Solicitud que inspecciona sin modificar el recurso. |
| Least privilege | Mínimo privilegio | Conceder solo los permisos necesarios durante el tiempo necesario. |
| Ephemeral VM | VM efímera | Máquina temporal que se descarta al terminar su ciclo. |
| Persistent home directory | Directorio personal persistente | `$HOME` de Cloud Shell que sobrevive a las sesiones estándar. |
| Authorization | Autorización | Permiso para que una herramienta use credenciales y realice solicitudes. |
| Authentication | Autenticación | Proceso de demostrar la identidad. |
| `PERMISSION_DENIED` | Permiso denegado | La identidad no tiene autorización suficiente para la solicitud. |
| `NOT_FOUND` | No encontrado | El recurso no existe, el ID puede ser incorrecto o no es visible. |

---

## 16. Repaso activo espaciado

Responde sin mirar la clave. Explica en voz alta y después comprueba.

### 16.1 Día hábil anterior — Lección 01

1. Ordena: recurso, proyecto, carpeta, organización.
2. ¿La cuenta de facturación es el padre jerárquico de un proyecto?
3. Distingue Project name, Project ID y Project number.
4. Explica la diferencia entre un servicio/API habilitado y un recurso creado.
5. ¿Por qué separar `dev` y `prod` en proyectos puede reducir riesgo?

**Clave breve:**

1. Organización → carpeta opcional → proyecto → recurso.
2. No. Se vincula para facturación, pero no ocupa ese lugar en la jerarquía de recursos.
3. Nombre: legible y modificable; ID: globalmente único; número: identificador numérico asignado.
4. Habilitar una API permite usarla en el proyecto; crear un recurso instancia algo administrable y potencialmente facturable.
5. Mejora aislamiento de permisos, cuotas, facturación, políticas y radio de impacto.

### 16.2 Hace 7 días — 15 de septiembre de 2026

El plan todavía no había comenzado; no existe una lección previa que repasar. Sustitución deliberada: repite de memoria el triángulo de hoy — **cuenta, configuración, proyecto**— y di qué comando consulta cada elemento.

### 16.3 Hace 21 días — 1 de septiembre de 2026

El plan todavía no había comenzado; no se inventa contenido. Sustitución deliberada: explica la diferencia entre consola, Cloud Shell y `gcloud` con una frase para cada uno.

---

## 17. Ficha de errores

Completa una fila por error real o por respuesta fallada. No escribas “entendido” sin evidencia.

| Fecha | Tarea o pregunta | Mi respuesta/acción | Respuesta/acción correcta | Causa raíz | Regla para la próxima vez | Fecha de repaso |
|---|---|---|---|---|---|---|
| 22/09/2026 | Ejemplo: confundí configuración con proyecto | | | Concepto / lectura / prisa / comando | | 23/09/2026 |
| | | | | | | 29/09/2026 |
| | | | | | | 13/10/2026 |

Etiquetas recomendadas para la causa raíz:

- **Concepto:** no distinguía dos términos.
- **Lectura:** ignoré una condición del escenario.
- **Contexto:** no verifiqué cuenta o proyecto.
- **CLI:** confundí grupo, comando, argumento u opción.
- **Seguridad:** elegí privilegios excesivos o expuse datos.
- **Prisa:** respondí antes de eliminar distractores.

---

## 18. Criterios de autoevaluación

Puntúa cada criterio con `0 = aún no`, `1 = con ayuda`, `2 = sin ayuda`.

| Criterio | 0–2 |
|---|---:|
| Explico consola, Cloud Shell y `gcloud` sin confundirlos. | |
| Distingo cuenta, configuración y proyecto. | |
| Identifico grupo, comando, argumento y flags en un comando. | |
| Verifico la cuenta activa con el comando apropiado. | |
| Verifico la configuración y el proyecto predeterminado. | |
| Interpreto una discrepancia entre consola y CLI. | |
| Describo un proyecto autorizado sin modificarlo. | |
| Rechazo compartir tokens o usar `Owner` como atajo. | |
| Explico el ciclo efímero de la VM y la persistencia de `$HOME`. | |
| Justifico cuándo usar consola, Cloud Shell o CLI. | |
| **Total** | **/20** |

Interpretación:

- **17–20:** continúa, pero registra cualquier pregunta fallada.
- **13–16:** repite los comandos de comprobación y las preguntas falladas.
- **0–12:** relee las secciones 4–6, completa la alternativa conceptual y vuelve a intentar el examen corto.

No conviertas esta puntuación en una afirmación automática de dominio. Busca evidencia: explicación propia, práctica correcta y recuerdo al día siguiente.

---

## 19. Documentación oficial verificada

Comandos y opciones contrastados el **22 de septiembre de 2026**:

- [Google Cloud console](https://console.cloud.google.com/)
- [How Cloud Shell works](https://docs.cloud.google.com/shell/docs/how-cloud-shell-works)
- [Launch Cloud Shell](https://docs.cloud.google.com/shell/docs/launching-cloud-shell)
- [gcloud CLI overview](https://docs.cloud.google.com/sdk/gcloud)
- [`gcloud auth list`](https://docs.cloud.google.com/sdk/gcloud/reference/auth/list)
- [`gcloud config list`](https://docs.cloud.google.com/sdk/gcloud/reference/config/list)
- [`gcloud config configurations list`](https://docs.cloud.google.com/sdk/gcloud/reference/config/configurations/list)
- [`gcloud projects list`](https://docs.cloud.google.com/sdk/gcloud/reference/projects/list)
- [`gcloud projects describe`](https://docs.cloud.google.com/sdk/gcloud/reference/projects/describe)
- [Associate Cloud Engineer certification](https://cloud.google.com/learn/certification/cloud-engineer)

La guía adjunta **associate_cloud_engineer_exam_guide_english.pdf** es la fuente de cobertura del examen para esta ruta. Las interfaces, versiones y referencias pueden cambiar; vuelve a comprobar la documentación antes de practicar en otra fecha.

---

## 20. Preguntas originales basadas en escenarios

Elige **una** respuesta por pregunta. Intenta responder en 15–20 minutos sin mirar las soluciones.

### Question 1

An engineer is using a locked-down laptop and must run preinstalled Google Cloud CLI commands without installing software. The organization permits the standard browser-based environment. What should the engineer use?

A. Create a Compute Engine VM in the production project.  
B. Activate Cloud Shell from the Google Cloud console.  
C. Create a GKE cluster and connect to one of its nodes.  
D. Download a service account key to the laptop.

### Question 2

You need to print only the account currently active in the gcloud CLI. Which command should you run?

A. `gcloud config list project`  
B. `gcloud projects list --limit=1`  
C. `gcloud auth list --filter=status:ACTIVE --format="value(account)"`  
D. `gcloud config configurations list --filter=project`

### Question 3

The Google Cloud console project selector shows `site-operations-prod`, but `gcloud config list project` shows `site-operations-dev`. You are about to create a resource. What should you do first?

A. Run the create command with `--quiet` so the configured default is used.  
B. Stop and verify the authorized account, configuration, and target project.  
C. Grant yourself the Owner role on both projects.  
D. Assume the console always overrides the gcloud configuration in an existing session.

### Question 4

A gcloud command supports the global `--project=PROJECT_ID` option. What is its effect?

A. It permanently changes the active configuration for all future commands.  
B. It grants the active account access to the specified project.  
C. It overrides the project property for that command invocation.  
D. It moves resources from the configured project to the specified project.

### Question 5

Which statement best describes a named gcloud configuration?

A. It is an IAM role that grants permissions across multiple projects.  
B. It is a profile containing gcloud CLI properties such as account and project.  
C. It is a billing account linked to the active project.  
D. It is a Cloud Shell virtual machine that persists indefinitely.

### Question 6

Which statement about the standard Cloud Shell environment is correct?

A. Its VM is created inside the selected customer project and incurs Compute Engine charges.  
B. Its VM and `$HOME` directory are both discarded after one hour of inactivity.  
C. Google manages a temporary VM, while 5 GB in `$HOME` persists across standard sessions.  
D. It can be used even when an organization has disabled Cloud Shell access.

### Question 7

An engineer must document a repeatable query that lists up to five accessible projects sorted by project ID. Which option is best?

A. Manually capture five console screenshots.  
B. Run `gcloud projects list --sort-by=projectId --limit=5`.  
C. Run `gcloud projects describe --all`.  
D. Provision Terraform state and import every project.

### Question 8

A teammate asks you to paste an access token into a shared chat to prove that your lab account works. What should you do?

A. Share the token because lab credentials are temporary.  
B. Share only the first half of the token.  
C. Refuse to share the token and verify access with non-secret, read-only evidence.  
D. Grant the teammate Owner so the token is unnecessary.

### Question 9

`gcloud projects describe site-operations-lab` returns `PERMISSION_DENIED`. What is the best next step?

A. Verify the active account and project ID, then request only the required access if needed.  
B. Create another project with the same name.  
C. Add `--quiet` to bypass the permission check.  
D. Authenticate with any personal Google account available on the machine.

### Question 10

In `gcloud projects list --limit=5`, which mapping is correct?

A. `projects` is the command, `list` is the project ID, and `--limit` is an IAM role.  
B. `projects` is a command group, `list` is a command, and `--limit=5` is a flag.  
C. `gcloud projects` is a configuration, and `list` creates five projects.  
D. `--limit=5` changes the project quota to five.

---

## 21. Soluciones justificadas

### Answer 1 — B

**B is correct.** Cloud Shell supplies a browser-accessible, preconfigured environment with `gcloud`, so no local installation or customer VM is required.

- **A is incorrect:** creating a production VM adds cost, administration and risk for a simple CLI requirement.
- **C is incorrect:** a GKE cluster is far more complex and creates billable resources.
- **D is incorrect:** downloading a service account key creates a credential-management risk and is unnecessary.

### Answer 2 — C

**C is correct.** `gcloud auth list` lists credentialed accounts; the filter selects `ACTIVE`, and the format prints only the `account` value.

- **A is incorrect:** it prints the project property, not the active identity.
- **B is incorrect:** it lists a project, which does not identify the active account reliably.
- **D is incorrect:** it lists configurations and the filter shown does not implement the requested identity query.

### Answer 3 — B

**B is correct.** A mismatch is a stop signal. Verify identity, active configuration and authorized target before any mutation.

- **A is incorrect:** `--quiet` suppresses prompts and can increase risk; it does not resolve the mismatch.
- **C is incorrect:** `Owner` is excessive and does not determine which target is intended.
- **D is incorrect:** the console project is propagated when Cloud Shell starts, but you must not assume continuous synchronization or override behavior.

### Answer 4 — C

**C is correct.** A command-line option takes precedence for that invocation; it does not permanently rewrite the property.

- **A is incorrect:** persistent change requires changing configuration, for example through an authorized `gcloud config set` workflow.
- **B is incorrect:** project selection never grants IAM permissions.
- **D is incorrect:** the option selects request context; it does not migrate resources.

### Answer 5 — B

**B is correct.** A named configuration is a profile of `gcloud` properties, which can include account, project, region and zone.

- **A is incorrect:** IAM roles belong to authorization policies, not CLI profiles.
- **C is incorrect:** billing accounts are separate resources; a configuration is local CLI context.
- **D is incorrect:** a configuration is not a VM, and Cloud Shell VMs are temporary.

### Answer 6 — C

**C is correct.** Google owns and manages the temporary VM; standard Cloud Shell provides 5 GB of persistent `$HOME` storage across sessions.

- **A is incorrect:** the Cloud Shell VM does not appear in the selected customer project.
- **B is incorrect:** the VM is discarded after inactivity, but standard `$HOME` persists subject to Google's retention conditions.
- **D is incorrect:** an organization can control or disable access to Cloud Shell.

### Answer 7 — B

**B is correct.** The command is the documented, reproducible query using sorting and limiting flags.

- **A is incorrect:** screenshots are manual, harder to reproduce and may expose identifiers.
- **C is incorrect:** `projects describe` needs one project ID or number; the shown `--all` use is invalid for this purpose.
- **D is incorrect:** Terraform state is unnecessary for a read-only inventory query.

### Answer 8 — C

**C is correct.** Tokens are bearer credentials and must not be shared. Demonstrate access with redacted, non-secret query output.

- **A is incorrect:** short lifetime does not make a credential safe to disclose.
- **B is incorrect:** partial disclosure is unnecessary and can still violate security policy.
- **D is incorrect:** granting `Owner` is a much larger security failure and violates least privilege.

### Answer 9 — A

**A is correct.** First validate identity and spelling; if access is genuinely required, obtain the minimum suitable permission through the approved process.

- **B is incorrect:** names and IDs differ, and creating another project neither fixes authorization nor follows the lab scope.
- **C is incorrect:** `--quiet` suppresses prompts; it cannot bypass IAM.
- **D is incorrect:** using an arbitrary personal account can violate policy and still may lack access.

### Answer 10 — B

**B is correct.** `projects` is the group, `list` is the action, and `--limit=5` changes how many results are returned.

- **A is incorrect:** it misclassifies every component.
- **C is incorrect:** listing is read-only and does not create projects.
- **D is incorrect:** `--limit` limits output records, not service quota.

**Registro del intento:** ____ / 10  
**Preguntas para la ficha de errores:** ______________________________  
**Próximo repaso:** 23/09/2026, luego 29/09/2026 y 13/10/2026.
