# ACE 2026 - Lección 12: Red inicial, regiones, zonas y disponibilidad de productos

> **Fecha:** 6 de octubre de 2026  
> **Dominio de la guía ACE:** 1.1 Setting up cloud projects and accounts  
> **Tema del plan:** Red inicial, regiones, zonas y disponibilidad de productos  
> **Práctica del plan:** Comparar ubicaciones y configurar o inspeccionar una VPC y subredes.  
> **Duración sugerida:** 75-90 minutos  
> **Caso conductor:** SiteOps Tracker, proyecto ficticio de portafolio.

Esta lección enseña y ejercita una parte del temario. Recibir el archivo no demuestra dominio: compruébalo explicando el alcance de cada recurso, realizando o simulando la práctica y justificando las decisiones sin mirar las soluciones.

## 1. Objetivo

Al finalizar podrás:

1. distinguir **location**, **region** y **zone**;
2. explicar por qué una VPC es global y una subnet es regional;
3. diferenciar recursos zonales, regionales, multirregionales y globales;
4. comprobar la disponibilidad geográfica de un producto antes de diseñar;
5. elegir una ubicación considerando latencia, residencia de datos, disponibilidad, costo y continuidad;
6. decidir entre una VPC en modo **custom** y una en modo **auto**;
7. crear o inspeccionar una VPC custom con dos subredes regionales mediante Console y Google Cloud CLI;
8. reconocer errores de permisos, API, región, CIDR y dependencias;
9. limpiar los recursos de la práctica y explicar su impacto económico.

### Evidencia de aprendizaje

Conserva al terminar:

- una tabla comparativa de dos ubicaciones candidatas;
- un diagrama o tabla que muestre una VPC global y dos subredes regionales;
- la salida de inspección de la red y sus subredes, o la alternativa conceptual completa;
- una justificación de cinco frases para la ubicación elegida para SiteOps Tracker;
- el registro de limpieza;
- la puntuación de las diez preguntas y una ficha por cada error.

## 2. Alineación con la guía oficial

La guía oficial de Associate Cloud Engineer incluye, dentro de **1.1 Setting up cloud projects and accounts**:

- **Setting up cloud networking**.
- **Verifying product availability across geographical locations (e.g., regions, zones)**.

La práctica de hoy cubre exactamente esos puntos. Las reglas detalladas de firewall, Shared VPC, VPC Network Peering, VPN, balanceadores y operación de red se estudian más adelante; aquí solo se introduce el modelo que permite entenderlos.

## 3. Prerrequisitos explicados desde cero

### 3.1 Proyecto

Un **project** es el contenedor administrativo en el que se crean recursos, se habilitan APIs, se aplican cuotas y se registra consumo. La VPC de la práctica vive en un proyecto concreto, aunque su alcance sea global dentro de ese proyecto.

Antes de ejecutar comandos, confirma el proyecto activo. Un comando técnicamente correcto ejecutado en el proyecto equivocado sigue siendo un error operativo.

### 3.2 Compute Engine API

La API `compute.googleapis.com` expone recursos de Compute Engine y VPC. Si está deshabilitada, los comandos de red fallarán aunque la identidad tenga permisos.

Habilitar una API y tener una cuota disponible son condiciones distintas:

- **API enabled:** el servicio puede recibir solicitudes para ese proyecto.
- **Quota:** límite administrativo o de capacidad aplicable al uso.
- **Product availability:** el producto o característica existe en la ubicación elegida.

Una condición no sustituye a las otras.

### 3.3 IAM mínimo

Para crear, modificar y borrar redes y subredes suele ser apropiado **Compute Network Admin** (`roles/compute.networkAdmin`). Este rol no concede administración general de VM ni administración de reglas de firewall. Para habilitar servicios, el permiso puede provenir de **Service Usage Admin** (`roles/serviceusage.serviceUsageAdmin`) u otro rol que incluya `serviceusage.services.enable`.

No solicites Owner solo para evitar analizar permisos. En un laboratorio temporal quizá ya tengas privilegios amplios; en un entorno real usa el rol mínimo que cubra la tarea.

### 3.4 Dirección IP y CIDR

Una subred reserva un bloque de direcciones. La notación `10.20.0.0/24` significa:

- `10.20.0.0` es la dirección inicial del bloque;
- `/24` fija los primeros 24 bits como prefijo de red;
- el bloque contiene 256 direcciones, aunque no todas son utilizables por cargas;
- dos rangos primarios o secundarios de subred de la misma VPC deben ser bloques CIDR válidos y únicos.

Para esta práctica no necesitas calcular subredes avanzadas. Basta reconocer que `10.20.0.0/24` y `10.30.0.0/24` no se superponen, mientras que dos subredes con `10.20.0.0/24` sí se superponen.

### 3.5 Cloud Shell y Google Cloud CLI

Puedes usar Cloud Shell o una instalación local de `gcloud`. Cloud Shell ya inicia con una identidad autenticada, pero aun así debes verificar cuenta y proyecto. No pegues secretos, claves privadas ni tokens en la terminal o en esta ficha.

## 4. Modelo mental: país, ciudad, edificio y plano vial

Imagina una empresa con sedes:

- una **region** es parecida a un área metropolitana elegida para operar;
- una **zone** es un área de despliegue dentro de esa región, como un edificio o campus independiente;
- una **VPC network** es el plano vial privado que conecta las sedes autorizadas;
- una **subnet** es una colonia o distrito de ese plano, situada en una región y con un rango de direcciones;
- una VM es una oficina concreta situada en una zona.

La analogía tiene un límite importante: una VPC global no mueve ni replica automáticamente aplicaciones o datos. El plano puede abarcar regiones; las cargas siguen teniendo el alcance y la estrategia de disponibilidad de cada producto.

### 4.1 Alcance de ubicaciones

| Alcance | Pregunta que responde | Ejemplo conceptual | Fallo que puede afectar |
|---|---|---|---|
| Zonal | ¿En qué zona exacta vive el recurso? | Una VM individual | Una zona |
| Regional | ¿En qué región vive o se replica el recurso? | Una subnet | La región, según diseño del servicio |
| Multirregional | ¿En qué conjunto definido de regiones guarda o sirve datos? | Opción de ubicación de algunos productos | El conjunto y la arquitectura del producto |
| Global | ¿El recurso se administra sin asociarlo a una sola región o zona? | Una VPC y sus reglas/rutas asociadas | No implica que todas las cargas sean globales |

No todos los productos ofrecen los cuatro alcances. Las opciones son específicas de cada servicio.

### 4.2 Cómo leer nombres

- `us-central1` es una región.
- `us-central1-a` es una zona de esa región.
- La letra final identifica la zona; no representa un nivel de servicio superior.
- Una región puede tener varias zonas, pero debes consultar la documentación y el listado actual, no memorizar una cantidad fija.

## 5. Regiones y zonas paso a paso

### 5.1 Región

Una región contiene zonas y representa una ubicación geográfica para recursos regionales. Elegir región afecta dónde se almacenan o procesan datos, la latencia hacia usuarios y sistemas, la disponibilidad de productos, el precio y el diseño de continuidad.

### 5.2 Zona

Una zona es un área de despliegue dentro de una región. Distribuir cargas entre zonas puede reducir la dependencia de una sola zona. Sin embargo, dos zonas de la misma región no protegen por sí solas frente a una interrupción regional.

### 5.3 Alta disponibilidad no es un atributo automático

Estas afirmaciones son diferentes:

1. “La VPC es global”.
2. “La aplicación se ejecuta en dos zonas”.
3. “La base de datos tiene alta disponibilidad regional”.
4. “Existe recuperación en una segunda región”.

La primera describe alcance de red. Las otras describen decisiones de arquitectura. En el examen, identifica primero qué nivel de fallo exige el escenario:

- fallo de instancia;
- fallo de zona;
- fallo de región;
- pérdida o corrupción de datos;
- indisponibilidad de un producto en la ubicación deseada.

No elijas una solución multirregional si el requisito solo pide tolerancia zonal y el costo importa. Tampoco ofrezcas dos zonas como respuesta a un requisito explícito de continuidad ante pérdida regional.

## 6. Disponibilidad geográfica de productos

### 6.1 No existe una “región universal”

Una región puede admitir Compute Engine pero no una característica concreta de otro servicio, un tipo de máquina, un motor de base de datos o una aceleradora. Incluso si el producto existe, una edición o función particular puede tener otra matriz de disponibilidad.

Antes de elegir ubicación:

1. enumera todos los productos y funciones obligatorios;
2. abre la página general de ubicaciones y la página de ubicaciones de cada producto;
3. comprueba región, edición y característica, no solo el nombre del servicio;
4. confirma requisitos de residencia o soberanía de datos;
5. revisa cuotas y disponibilidad de capacidad;
6. compara precio y transferencia de datos;
7. documenta una segunda opción viable.

### 6.2 Matriz de decisión para SiteOps Tracker

SiteOps Tracker usa React + TypeScript, Node.js + TypeScript, PostgreSQL y servicios de Google Cloud. Antes de desplegar, llena una matriz como esta con datos oficiales actuales:

| Criterio | Región candidata A | Región candidata B | Regla de descarte |
|---|---|---|---|
| Latencia hacia sedes y usuarios | Medición o estimación | Medición o estimación | Rechazar si incumple el objetivo |
| Servicio para API Node.js | Disponible/no disponible | Disponible/no disponible | Debe estar disponible |
| PostgreSQL administrado y función requerida | Disponible/no disponible | Disponible/no disponible | Debe estar disponible |
| Almacenamiento de evidencias | Disponible/no disponible | Disponible/no disponible | Debe estar disponible |
| Residencia de datos | Cumple/no cumple | Cumple/no cumple | Debe cumplir política |
| Zonas y diseño de continuidad | Opción documentada | Opción documentada | Debe cubrir el fallo requerido |
| Precio y transferencia | Estimación | Estimación | Debe caber en presupuesto |
| Plan de recuperación regional | Opción documentada | Opción documentada | Obligatorio solo si el requisito lo pide |

**Orden recomendado:** primero requisitos duros; después optimización. Una región cercana que no ofrece el componente obligatorio no es candidata válida.

### 6.3 Capacidad frente a disponibilidad de producto

- **Product availability:** la documentación indica que el producto o función se ofrece en esa ubicación.
- **Capacity availability:** en ese momento existe capacidad para la configuración solicitada.
- **Quota:** tu proyecto está autorizado para solicitar cierta cantidad.

Un error `ZONE_RESOURCE_POOL_EXHAUSTED` puede señalar capacidad temporal, no que el producto esté ausente de toda la región. Un error de cuota tampoco prueba ausencia del producto.

## 7. VPC y subredes paso a paso

### 7.1 VPC network

Una **Virtual Private Cloud network** es una red virtual dentro de la infraestructura de Google. En Google Cloud:

- la VPC es un recurso global;
- sus rutas y reglas de firewall asociadas también son globales;
- un proyecto puede tener varias VPC;
- la VPC no define por sí sola una única franja CIDR global;
- la conectividad efectiva también depende de rutas y reglas de firewall.

### 7.2 Subnet

Una subnet es un recurso regional dentro de una VPC:

- tiene un rango IP primario;
- puede tener rangos secundarios para casos como alias IP;
- puede ser usada por interfaces de recursos ubicados en zonas de la misma región;
- sus rangos no deben superponerse con otros rangos de subred de la misma VPC.

Una misma VPC puede tener subredes en distintas regiones. No necesitas crear una VPC por región solo por ese motivo.

### 7.3 Auto mode

Al crear una VPC en modo **auto**, Google crea una subred con un rango predefinido en cada región admitida y agrega subredes cuando aparecen regiones nuevas.

Puede ser útil para una prueba rápida, pero tiene desventajas para una producción planificada:

- crea más subredes de las necesarias;
- usa rangos predefinidos;
- complica evitar solapamientos con redes externas;
- reduce el control sobre regiones e IPv6.

### 7.4 Custom mode

Al crear una VPC en modo **custom**, no se crea ninguna subred automáticamente. Tú eliges regiones y rangos.

Para SiteOps Tracker es la opción didáctica y normalmente la opción de producción preferible porque permite:

- crear solo las regiones justificadas;
- reservar rangos no superpuestos;
- documentar crecimiento;
- integrar con redes externas de forma deliberada.

La conversión de auto a custom es de una sola dirección; una VPC custom no vuelve a auto.

### 7.5 Default network

Si una política de organización no lo impide, un proyecto nuevo puede incluir la red `default`. Es una VPC auto mode con reglas de firewall preconfiguradas. No la uses por inercia en producción:

- puede crear un alcance más amplio de lo necesario;
- sus reglas y rangos quizá no sigan tu modelo de seguridad;
- “ya existe” no equivale a “cumple requisitos”.

Para un laboratorio aislado, crea una VPC custom con nombres inequívocos y elimínala al terminar.

### 7.6 Dynamic routing mode

El modo de enrutamiento dinámico (`regional` o `global`) controla cómo se comportan los Cloud Routers y las rutas dinámicas aprendidas. No cambia el alcance de una subnet y no replica cargas. En esta práctica se conserva `regional` porque no se configura conectividad híbrida.

## 8. Diseño ficticio de SiteOps Tracker

Supón estos requisitos:

- usuarios principales cerca de una región candidata primaria;
- evidencia y datos sujetos a una política de ubicación que debe verificarse;
- API Node.js y PostgreSQL deben estar disponibles en la región elegida;
- el laboratorio solo necesita demostrar red, sin crear VM, NAT ni base de datos;
- se quiere reservar una región secundaria para estudiar recuperación, no activar servicios aún.

~~~mermaid
flowchart TB
  VPC["siteops-vpc-lab (global)"]
  VPC --> S1["siteops-primary\n10.20.0.0/24\nregion A"]
  VPC --> S2["siteops-secondary\n10.30.0.0/24\nregion B"]
  S1 --> Z1["Workloads in zones\nof region A"]
  S2 --> Z2["Recovery candidates\nin region B"]
~~~

Este diagrama muestra alcance, no una arquitectura activa. Tener `siteops-secondary` vacía no constituye recuperación ante desastres: faltan datos, cómputo, balanceo, DNS, procedimientos y pruebas de failover.

## 9. Glosario bilingüe

| Español | English | Significado operativo |
|---|---|---|
| ubicación | location | Ámbito geográfico ofrecido por un producto |
| región | region | Ubicación geográfica que contiene zonas |
| zona | zone | Área de despliegue dentro de una región |
| recurso zonal | zonal resource | Recurso ligado a una zona |
| recurso regional | regional resource | Recurso ligado a una región |
| recurso global | global resource | Recurso no ligado a una sola región o zona |
| multirregión | multi-region | Conjunto definido de regiones administrado por un producto |
| nube privada virtual | Virtual Private Cloud (VPC) | Red virtual global de un proyecto |
| subred | subnet / subnetwork | Recurso regional con uno o más rangos IP |
| rango primario | primary IP range | Bloque base de direcciones de una subnet |
| rango secundario | secondary IP range | Bloque adicional usado para alias IP y casos específicos |
| bloque CIDR | CIDR block | Notación de red como `10.20.0.0/24` |
| solapamiento | overlap | Dos rangos comparten direcciones, situación no válida dentro de la VPC |
| modo automático | auto mode | Crea subredes regionales automáticamente |
| modo personalizado | custom mode | Permite elegir cada región y rango |
| ruta | route | Camino que puede seguir el tráfico saliente |
| ingreso | ingress | Tráfico que entra a un recurso |
| egreso | egress | Tráfico que sale de un recurso o ubicación |
| residencia de datos | data residency | Requisito sobre dónde se almacenan o procesan datos |
| conmutación por error | failover | Cambio a una alternativa tras una falla |
| recuperación ante desastres | disaster recovery | Capacidad y proceso para restaurar el servicio tras un evento grave |
| disponibilidad de producto | product availability | Oferta documentada de un servicio o función en una ubicación |
| disponibilidad de capacidad | capacity availability | Capacidad física actualmente utilizable |

## 10. Decisiones de servicio y por qué no elegir alternativas

| Requisito | Elección recomendada | Por qué | Por qué no la alternativa |
|---|---|---|---|
| Producción con dos regiones concretas y rangos controlados | VPC custom | Control explícito de regiones y CIDR | Auto crea subredes predefinidas en todas las regiones |
| Prueba rápida sin integración externa | Auto puede ser aceptable | Menos configuración inicial | No usar si necesitas control de direccionamiento o crecimiento |
| Tolerar caída de una zona | Recursos en varias zonas o servicio regional con HA documentada | Elimina la dependencia de una zona | Dos instancias en la misma zona comparten el mismo ámbito de fallo |
| Tolerar caída de una región | Segunda región con datos, cómputo y failover probado | Cubre el ámbito regional | Varias zonas de una misma región no cubren pérdida regional |
| Menor latencia para usuarios | Región cercana que también cumpla todos los requisitos | Optimiza ida y vuelta sin romper restricciones | Elegir solo por distancia puede omitir producto, residencia o costo |
| Residencia obligatoria | Ubicación admitida por política y servicio | Cumple requisito duro | Multirregión o región extranjera puede violar la restricción |
| Producto ausente en la región preferida | Otra región admitida o alternativa funcional evaluada | Parte de una matriz real de disponibilidad | No asumir que “Google Cloud global” significa disponibilidad universal |
| Usar una VPC en dos regiones | Una VPC global con una subnet por región | Es el modelo nativo de Google Cloud | Dos VPC añaden aislamiento y complejidad sin necesidad en este caso |

### Regla de examen

Subraya primero el requisito decisivo:

- **lowest latency**: cercanía, pero con disponibilidad del stack;
- **survive a zonal failure**: varias zonas o servicio regional apropiado;
- **survive a regional failure**: segunda región y failover;
- **data must remain in...**: ubicación compatible con la política;
- **minimize operational overhead**: servicio administrado y alcance suficiente, sin sobrediseñar;
- **avoid overlapping IPs**: VPC custom y plan de CIDR.

## 11. Práctica guiada: comparar ubicaciones y crear o inspeccionar una VPC

### 11.1 Alcance y seguridad

La práctica crea únicamente una VPC custom y dos subredes IPv4. No crea VM, reglas de firewall explícitas, IP pública, Cloud NAT, VPN, balanceador ni base de datos.

Usa un proyecto de laboratorio temporal. No ejecutes la creación en producción. Si solo tienes permisos de lectura, realiza la inspección y después la alternativa conceptual.

Comandos y opciones contrastados con la documentación oficial el **6 de octubre de 2026**:

- `gcloud services enable`;
- `gcloud compute regions list` y `gcloud compute zones list`;
- `gcloud compute networks create`, `list`, `describe` y `delete`;
- `gcloud compute networks subnets create`, `list`, `describe` y `delete`.

### 11.2 Parte A - comparación de ubicaciones

1. Abre [Cloud locations](https://cloud.google.com/about/locations).
2. Elige dos regiones candidatas cercanas a los usuarios ficticios de SiteOps Tracker.
3. Para cada producto previsto, abre su página de ubicaciones. Como mínimo revisa el servicio elegido para la API Node.js, PostgreSQL administrado y almacenamiento de evidencias.
4. Registra “sí/no” para cada producto y función obligatoria.
5. Añade una restricción ficticia de residencia de datos y un objetivo de latencia.
6. Elimina cualquier región que incumpla un requisito duro.
7. Justifica la ganadora con cinco frases: usuarios, producto, datos, disponibilidad y costo.

No copies una lista de regiones a memoria. La habilidad examinable es saber verificar y decidir.

### 11.3 Parte B - inspección inicial en Console

1. Abre Google Cloud Console y selecciona el proyecto de laboratorio.
2. Ve a **VPC network > VPC networks**.
3. Observa las redes existentes y su **Subnet creation mode**.
4. Abre una red que puedas leer.
5. Identifica nombre, modo de subred, MTU, modo de enrutamiento dinámico y subredes.
6. En cada subnet visible, registra región y rango IP.
7. No edites ni borres una red existente que no haya sido creada para esta práctica.

Si puedes crear recursos, continúa con una VPC nueva. Si no, salta a 11.8.

### 11.4 Parte C - preflight con CLI

En Cloud Shell, sustituye `YOUR_PROJECT_ID` por el ID real del proyecto de laboratorio:

~~~bash
export PROJECT_ID="YOUR_PROJECT_ID"
export NETWORK="siteops-vpc-lab"
export SUBNET_PRIMARY="siteops-primary"
export SUBNET_SECONDARY="siteops-secondary"
export REGION_PRIMARY="us-central1"
export REGION_SECONDARY="us-east1"

gcloud auth list
gcloud config set project "$PROJECT_ID"
gcloud config list
gcloud projects describe "$PROJECT_ID" --format="value(projectId,lifecycleState)"
~~~

Confirma que:

- la cuenta activa es la esperada;
- el proyecto coincide exactamente;
- `lifecycleState` es `ACTIVE`;
- las regiones de ejemplo aparecen como `UP` en el listado actual.

Consulta regiones y zonas:

~~~bash
gcloud compute regions list
gcloud compute zones list
~~~

Habilita la API solo si la práctica y tus permisos lo permiten:

~~~bash
gcloud services enable compute.googleapis.com
~~~

Si la organización exige otras regiones, cambia ambas variables y vuelve a comprobar disponibilidad. No uses dos valores iguales para esta comparación.

### 11.5 Parte D - crear la VPC custom

~~~bash
gcloud compute networks create "$NETWORK" \
  --subnet-mode=custom \
  --bgp-routing-mode=regional \
  --mtu=1460
~~~

Inspecciona el recurso:

~~~bash
gcloud compute networks describe "$NETWORK" \
  --format="yaml(name,autoCreateSubnetworks,routingConfig.routingMode,mtu)"
~~~

Debes observar `autoCreateSubnetworks: false`. La ausencia de subredes en este punto es normal: custom mode no las crea automáticamente.

### 11.6 Parte E - crear dos subredes regionales

~~~bash
gcloud compute networks subnets create "$SUBNET_PRIMARY" \
  --network="$NETWORK" \
  --range="10.20.0.0/24" \
  --region="$REGION_PRIMARY" \
  --description="SiteOps primary lab subnet"

gcloud compute networks subnets create "$SUBNET_SECONDARY" \
  --network="$NETWORK" \
  --range="10.30.0.0/24" \
  --region="$REGION_SECONDARY" \
  --description="SiteOps secondary lab subnet"
~~~

Los bloques no se superponen. La segunda subnet no convierte la aplicación en multirregional; solo reserva una red regional adicional.

### 11.7 Parte F - inspeccionar y demostrar el alcance

~~~bash
gcloud compute networks list \
  --filter="name=$NETWORK"

gcloud compute networks subnets list \
  --network="$NETWORK" \
  --format="table(name,region,ipCidrRange,privateIpGoogleAccess)"

gcloud compute networks subnets describe "$SUBNET_PRIMARY" \
  --region="$REGION_PRIMARY" \
  --format="yaml(name,region,network,ipCidrRange,gatewayAddress,privateIpGoogleAccess,purpose,stackType)"
~~~

Responde con tus propias palabras:

1. ¿Qué campo prueba que la VPC es custom?
2. ¿Qué campos muestran que cada subnet es regional?
3. ¿Por qué los dos CIDR pueden convivir?
4. ¿En qué zonas puede una VM usar `siteops-primary`?
5. ¿Qué falta para afirmar que SiteOps Tracker tiene recuperación regional?

### 11.8 Alternativa conceptual completa sin cuenta, crédito o permisos

Esta alternativa cubre el mismo objetivo sin crear recursos.

#### Escenario autocontenido

Usa esta matriz **ficticia**, creada solo para practicar la decisión; no describe disponibilidad real de Google Cloud:

| Criterio | Región Alfa | Región Beta | Región Gamma |
|---|---:|---:|---:|
| Latencia estimada a usuarios | 18 ms | 42 ms | 24 ms |
| API Node.js administrada | Sí | Sí | Sí |
| PostgreSQL administrado con HA requerida | Sí | Sí | No |
| Almacenamiento de evidencias | Sí | Sí | Sí |
| Cumple residencia de datos | Sí | Sí | Sí |
| Índice de costo relativo | 1.08 | 1.00 | 0.94 |
| Segunda región de recuperación autorizada | Beta | Alfa | Beta |

Requisitos:

- latencia menor de 50 ms;
- los tres productos obligatorios en la misma región primaria;
- HA de PostgreSQL;
- menor latencia como desempate;
- una segunda región autorizada para recuperación.

Resultado esperado:

1. Descarta Gamma porque no ofrece la función obligatoria de PostgreSQL.
2. Alfa y Beta cumplen los requisitos duros.
3. Elige Alfa como primaria por la regla de desempate de latencia.
4. Reserva Beta para diseñar recuperación, pero aclara que una subnet vacía no implementa recuperación.

#### Configuración simulada

Completa esta ficha:

| Recurso | Alcance | Región | CIDR | Estado simulado |
|---|---|---|---|---|
| `siteops-vpc-lab` | Global | No aplica | No aplica | Creada, custom |
| `siteops-primary` | Regional | Alfa | `10.20.0.0/24` | Creada |
| `siteops-secondary` | Regional | Beta | `10.30.0.0/24` | Creada |

Escribe las tres validaciones esperadas:

- `autoCreateSubnetworks` es `false`;
- las subredes muestran regiones diferentes;
- los rangos no se superponen.

Después redacta un procedimiento de limpieza: borrar primero las dos subredes y luego la VPC. Con esto completas la evidencia sin una cuenta activa.

## 12. Resultado esperado

### Si ejecutaste la práctica

- existe una sola VPC llamada `siteops-vpc-lab` en modo custom;
- la inspección muestra `autoCreateSubnetworks: false`;
- existen `siteops-primary` y `siteops-secondary` en regiones distintas;
- sus rangos son `10.20.0.0/24` y `10.30.0.0/24`;
- no se creó ninguna VM ni servicio de datos;
- puedes explicar que VPC global no significa aplicación global;
- al terminar, la limpieza deja de mostrar esos tres recursos.

### Si realizaste la alternativa conceptual

- descartaste una región por ausencia de una función obligatoria;
- elegiste una región primaria con una regla explícita;
- clasificaste correctamente VPC y subredes por alcance;
- diseñaste CIDR sin solapamiento;
- escribiste la secuencia de inspección y limpieza.

### Justificación breve de SiteOps Tracker

Completa sin mirar notas:

> Elegí __________ como región primaria porque __________. Verifiqué que __________, __________ y __________ están disponibles con las funciones requeridas. La VPC es global, pero la subnet primaria es regional. Para tolerar una caída de zona usaría __________. Para tolerar una caída regional necesitaría además __________. No afirmo que la subnet secundaria por sí sola proporcione recuperación porque __________.

## 13. Solución de problemas

| Síntoma | Causa probable | Diagnóstico | Corrección segura |
|---|---|---|---|
| `PERMISSION_DENIED` al crear red | Falta permiso de red | Revisa cuenta activa y política IAM | Solicita `roles/compute.networkAdmin` o permisos equivalentes mínimos |
| Error al habilitar API | Falta permiso Service Usage | Verifica si la API ya está habilitada | Pide a un administrador habilitarla; no solicites Owner por defecto |
| API no habilitada | `compute.googleapis.com` desactivada | Revisa APIs del proyecto | Habilita la API con autorización |
| Recurso ya existe | Nombre repetido en el proyecto | Lista redes o subredes | Inspecciona el existente o usa un nombre inequívoco |
| Región inválida | Nombre incorrecto o región no disponible | Ejecuta `gcloud compute regions list` | Copia un nombre válido con estado adecuado |
| CIDR se superpone | El rango comparte direcciones con otra subnet | Lista subredes y compara `ipCidrRange` | Elige un bloque único dentro del plan IP |
| `Invalid value for field 'resource.IPCidrRange'` | CIDR mal formado o prohibido | Revisa dirección y prefijo | Usa un CIDR válido, por ejemplo `10.30.0.0/24` |
| La VM no tendría ingreso | Custom VPC sin regla explícita y deny ingress implícito | Inspecciona reglas de firewall | Es el comportamiento esperado; no abras puertos para esta práctica |
| No puedes borrar la subnet | Hay un recurso que la usa | Lista recursos dependientes | Elimina solo dependencias del laboratorio, luego la subnet |
| No puedes borrar la VPC | Persisten subredes o referencias | Revisa subredes, reglas, rutas personalizadas, peering, VPN, routers o conectores | Elimina dependencias autorizadas; no fuerces borrado de recursos ajenos |
| Servicio ausente de la región | La característica no se ofrece allí | Revisa página de ubicaciones del producto | Cambia de región o evalúa otro servicio con los requisitos |
| Error de capacidad | Capacidad temporal insuficiente | Distingue capacidad, cuota y disponibilidad geográfica | Prueba otra zona/configuración o sigue el procedimiento de capacidad |

### Diagnóstico en orden

1. identidad;
2. proyecto;
3. API;
4. IAM;
5. nombre de región o zona;
6. disponibilidad del producto o función;
7. cuota;
8. capacidad temporal;
9. dependencias y políticas de organización.

Este orden evita cambiar arquitectura cuando el problema real es de contexto o permisos.

## 14. Impacto en costos y limpieza

### 14.1 Impacto en costos

La práctica no crea cargas de cómputo ni datos. Una VPC básica y sus subredes no tienen un cargo horario independiente en la tabla de precios de red. Sin embargo, otros componentes y tráfico sí pueden generar cargos:

- transferencia entre zonas o regiones;
- transferencia de salida a internet;
- direcciones IPv4 externas en ciertos estados;
- Cloud NAT;
- Cloud VPN o Cloud Interconnect;
- balanceadores;
- VPC Flow Logs, incluida ingestión y almacenamiento de logs;
- VM, bases de datos y otros servicios conectados.

La transferencia entrante suele no tener cargo de red, pero el recurso que procesa los datos sí puede cobrar. Los precios cambian; consulta siempre la página oficial antes de estimar.

No habilites VPC Flow Logs solo para observar esta práctica vacía. Tampoco crees una VM “para probar” si el objetivo puede demostrarse mediante inspección de configuración.

### 14.2 Limpieza por Console

1. Ve a **VPC network > VPC networks**.
2. Abre `siteops-vpc-lab`.
3. Verifica que solo contiene recursos de esta práctica.
4. Elimina primero `siteops-secondary` y `siteops-primary`.
5. Elimina después `siteops-vpc-lab`.
6. Recarga la lista y confirma que ya no aparecen.

### 14.3 Limpieza por CLI

Ejecuta solo si las variables aún identifican exactamente los recursos que creaste:

~~~bash
gcloud compute networks subnets delete "$SUBNET_SECONDARY" \
  --region="$REGION_SECONDARY" \
  --quiet

gcloud compute networks subnets delete "$SUBNET_PRIMARY" \
  --region="$REGION_PRIMARY" \
  --quiet

gcloud compute networks delete "$NETWORK" --quiet
~~~

Verifica:

~~~bash
gcloud compute networks list --filter="name=$NETWORK"
gcloud compute networks subnets list
~~~

El resultado no debe contener esos nombres. No deshabilites Compute Engine API como limpieza rutinaria de un proyecto compartido: otros recursos pueden depender de ella.

### 14.4 Registro de limpieza

| Elemento | Creado | Eliminado | Evidencia |
|---|---|---|---|
| `siteops-vpc-lab` | Sí / No | Sí / No | Salida o captura |
| `siteops-primary` | Sí / No | Sí / No | Salida o captura |
| `siteops-secondary` | Sí / No | Sí / No | Salida o captura |
| Recursos adicionales | Debe ser No | No aplica | Confirmación |

## 15. Repaso activo espaciado

No leas las respuestas anteriores hasta intentar cada recuperación.

### Día hábil anterior - Lección 11

Tema: presupuestos, alertas y exportación de facturación.

1. Explica por qué un presupuesto no es un límite automático de gasto.
2. Distingue actual spend de forecasted spend.
3. ¿Qué aporta exportar datos de facturación a BigQuery?
4. Relaciona el tema de hoy: ¿cómo puede la elección de región afectar transferencia y costo?

### Dos días hábiles antes - Lección 10

Tema: cuentas de facturación y vínculo con proyectos.

1. ¿Cuántas cuentas de facturación activas puede tener vinculadas un proyecto a la vez?
2. ¿Por qué el vínculo de facturación no es un padre de la jerarquía IAM?
3. Antes de crear una VPC de laboratorio, ¿qué proyecto y cuenta de facturación debes verificar?

### Tres días hábiles antes - Lección 9

Tema: APIs, cuotas y aumentos de cuotas.

1. Distingue API habilitada, cuota y capacidad regional.
2. ¿Qué comando habilita Compute Engine API?
3. ¿Por qué un error de cuota no demuestra que el producto no exista en la región?

### Hace 7 días - Lección 7

Tema: políticas de organización, restricciones y herencia.

Recupera tres decisiones:

1. ¿En qué nivel aplicarías una restricción que debe afectar todos los proyectos?
2. ¿Qué diferencia una Organization Policy de una política IAM?
3. ¿Cómo podría una restricción heredada impedir IPv6, una ubicación o la creación de recursos aunque tu rol IAM sea suficiente?

### Hace 21 días - línea base

El plan comenzó el 21 de septiembre; el 15 de septiembre todavía no había una lección programada. Para no inventar historial, usa esta recuperación fundacional:

1. Define recurso, proyecto y servicio.
2. Explica por qué “global” puede describir el alcance administrativo de un recurso sin garantizar alta disponibilidad de la aplicación.
3. Escribe una pregunta que debes hacer antes de elegir cualquier región.

### Refuerzo breve de confusiones acumulativas

- Un presupuesto avisa; no detiene automáticamente el gasto.
- IAM autoriza identidades; Organization Policy limita configuraciones permitidas.
- API habilitada, cuota, capacidad y disponibilidad geográfica son comprobaciones diferentes.
- VPC global no significa que datos y cómputo se repliquen globalmente.

## 16. Ficha de práctica

Completa al terminar:

| Campo | Registro |
|---|---|
| Fecha | 6 de octubre de 2026 |
| Modalidad | Console + CLI / Solo inspección / Alternativa conceptual |
| Proyecto de laboratorio | |
| Región primaria candidata | |
| Región secundaria candidata | |
| Productos verificados | |
| VPC | |
| Subredes y CIDR | |
| Resultado observado | |
| Limpieza confirmada | |
| Tiempo dedicado | |
| Duda principal | |

## 17. Ficha de errores

Crea una fila por error de práctica o pregunta. No escribas solo “me confundí”.

| Campo | Tu registro |
|---|---|
| Escenario o comando | |
| Mi respuesta o acción | |
| Resultado correcto | |
| Tipo de error | Concepto / lectura / alcance / CLI / IAM / API / CIDR / costo |
| Pista ignorada | |
| Regla de decisión nueva | |
| Explicación sin apuntes | |
| Reintento y fecha | |

Ejemplo de regla útil:

> Si el escenario exige sobrevivir una falla regional, varias zonas de una sola región no bastan; debo evaluar una segunda región y el mecanismo de failover de cada componente.

## 18. Criterios de autoevaluación

Asigna un punto por cada criterio demostrado, no por haberlo leído:

| Criterio | Punto |
|---|---:|
| Distingo región de zona con un ejemplo | 1 |
| Clasifico VPC como global y subnet como regional | 1 |
| Explico por qué global no equivale a HA global | 1 |
| Verifico disponibilidad por producto y función | 1 |
| Elijo ubicación con requisitos duros antes de optimizar | 1 |
| Diferencio auto, custom y default | 1 |
| Diseño dos CIDR no superpuestos | 1 |
| Creo/inspecciono la configuración o completo la simulación | 1 |
| Explico costo y limpio de forma segura | 1 |
| Obtengo al menos 8/10 y corrijo cada error | 1 |

Interpretación personal:

- **9-10:** listo para combinar este tema con reglas de firewall y servicios regionales; repasa errores igualmente.
- **7-8:** repite la comparación de ubicaciones y la inspección de alcance.
- **0-6:** vuelve al modelo mental, realiza la alternativa conceptual y después reintenta las preguntas.

Esta escala es una meta de estudio, no un umbral oficial de Google. No concluyas dominio solo por recibir o leer la lección.

## 19. Documentación oficial verificada

Consultada el **6 de octubre de 2026**:

- [Associate Cloud Engineer certification](https://cloud.google.com/learn/certification/cloud-engineer)
- [VPC networks](https://cloud.google.com/vpc/docs/vpc)
- [Create and manage VPC networks](https://cloud.google.com/vpc/docs/create-modify-vpc-networks)
- [Subnets](https://cloud.google.com/vpc/docs/subnets)
- [Regions and zones](https://cloud.google.com/compute/docs/regions-zones)
- [Cloud locations and products available by region](https://cloud.google.com/about/locations)
- [`gcloud compute networks create`](https://cloud.google.com/sdk/gcloud/reference/compute/networks/create)
- [`gcloud compute networks list`](https://cloud.google.com/sdk/gcloud/reference/compute/networks/list)
- [`gcloud compute networks describe`](https://cloud.google.com/sdk/gcloud/reference/compute/networks/describe)
- [`gcloud compute networks subnets create`](https://cloud.google.com/sdk/gcloud/reference/compute/networks/subnets/create)
- [`gcloud compute networks subnets list`](https://cloud.google.com/sdk/gcloud/reference/compute/networks/subnets/list)
- [`gcloud compute networks subnets describe`](https://cloud.google.com/sdk/gcloud/reference/compute/networks/subnets/describe)
- [`gcloud compute regions list`](https://cloud.google.com/sdk/gcloud/reference/compute/regions/list)
- [`gcloud compute zones list`](https://cloud.google.com/sdk/gcloud/reference/compute/zones/list)
- [Enable and disable services](https://cloud.google.com/service-usage/docs/enable-disable)
- [Compute Engine roles and permissions](https://cloud.google.com/iam/docs/roles-permissions/compute)
- [Network pricing](https://cloud.google.com/vpc/network-pricing)

## 20. Preguntas de examen originales

Responde antes de abrir la sección de soluciones. Todas las preguntas son originales y están basadas en escenarios; no son dumps del examen.

### Question 1

A company needs one private network for workloads in two Google Cloud regions. Which statement is correct?

A. A VPC network is zonal, so the company needs one VPC per zone.  
B. A VPC network is regional, so the company needs one VPC per region.  
C. A VPC network is global, and it can contain regional subnets in both regions.  
D. A subnet is global, so one subnet can be attached to resources in any region.

### Question 2

SiteOps Tracker must keep regulated data in an allowed geography and use a required PostgreSQL feature. Users also need low latency. What should the engineer do first? **Select two answers.**

A. Verify that the required products and specific features are available in each candidate location.  
B. Select the region with the shortest name to reduce configuration errors.  
C. Eliminate locations that violate the data residency requirement.  
D. Select any global Google Cloud service and assume all dependent products are globally available.  
E. Choose the least expensive region before checking mandatory requirements.

### Question 3

A production project needs subnets only in two approved regions and must use organization-planned, non-overlapping CIDR ranges. Which VPC configuration is most appropriate?

A. A custom mode VPC with explicitly created subnets.  
B. An auto mode VPC because it creates a subnet in every region.  
C. The default network without reviewing its rules.  
D. One auto mode VPC per approved region.

### Question 4

An engineer has created `10.20.0.0/24` as a primary subnet range in a VPC. Which range can the engineer use for another subnet in the same VPC?

A. `10.20.0.0/24`  
B. `10.20.0.128/25`  
C. `10.30.0.0/24`  
D. `10.20.0.0/25`

### Question 5

The API tier must remain available if one zone fails, but the company does not require protection from a full regional outage. Which design best matches the requirement without unnecessary regional complexity?

A. Run every API instance in one zone and rely on the global VPC.  
B. Distribute the API across multiple zones in the same region, using an appropriate regional service or load-balancing design.  
C. Create an empty subnet in a second region.  
D. Change the VPC dynamic routing mode to global.

### Question 6

The recovery requirement changes: SiteOps Tracker must continue after a complete regional outage. What is required?

A. Two zones in the primary region only.  
B. A global VPC only.  
C. A second region with the necessary application and data recovery components plus a tested failover mechanism.  
D. A larger primary subnet.

### Question 7

Which command correctly inspects one subnet named `siteops-primary` in `us-central1`?

A. `gcloud compute networks describe siteops-primary --zone=us-central1-a`  
B. `gcloud compute networks subnets describe siteops-primary --region=us-central1`  
C. `gcloud compute subnets get siteops-primary --global`  
D. `gcloud networks regions describe siteops-primary --region=us-central1`

### Question 8

The preferred region is close to users, but a mandatory database feature is not offered there. What should the engineer do?

A. Deploy anyway because Google Cloud products are globally available.  
B. Create a subnet in the region; that makes the database feature available.  
C. Select another supported region or evaluate a suitable alternative service, then document latency, residency, cost, and availability trade-offs.  
D. Increase the project quota in the preferred region.

### Question 9

An engineer sets `--bgp-routing-mode=global` when creating a VPC. Which outcome should the engineer expect?

A. Every subnet becomes a global resource.  
B. All workloads are replicated across regions.  
C. Cloud Router dynamic route behavior changes, but subnets remain regional and workloads are not replicated automatically.  
D. Product availability becomes identical in all regions.

### Question 10

Which statement best describes cost for this networking lab?

A. Every VPC and subnet has an hourly charge, even with no workloads.  
B. The basic empty VPC and subnets do not introduce the workload charges of VMs or databases, but traffic and services such as NAT, VPN, load balancing, external IP addresses, and logging can cost money.  
C. All inter-zone and inter-region traffic is free inside one VPC.  
D. Deleting the VPC automatically deletes every resource in the project.

## 21. Soluciones justificadas

### 1. Correct answer: C

- **C is correct:** a Google Cloud VPC is global and can contain subnets in multiple regions; each subnet is regional.
- **A is incorrect:** a VPC is not zonal.
- **B is incorrect:** the VPC is not limited to one region.
- **D is incorrect:** a subnet is regional and is used by resources in zones of that region.

**Decision rule:** classify the network and the subnet separately: global VPC, regional subnet.

### 2. Correct answers: A and C

- **A is correct:** availability must be verified for the exact product and required feature.
- **C is correct:** data residency is a hard constraint, so noncompliant locations must be removed before optimization.
- **B is incorrect:** a region name has no architectural value.
- **D is incorrect:** a global resource does not make every dependent product globally available.
- **E is incorrect:** cost optimization comes after mandatory requirements are satisfied.

**Decision rule:** filter by hard constraints first; compare latency and cost only among valid candidates.

### 3. Correct answer: A

- **A is correct:** custom mode provides explicit control over regions and CIDR ranges.
- **B is incorrect:** auto mode creates more regional subnets than required and uses predefined ranges.
- **C is incorrect:** the default network is auto mode and includes preconfigured firewall rules that must not be accepted without review.
- **D is incorrect:** VPCs are global; one VPC per region is unnecessary for this requirement.

**Decision rule:** choose custom mode when production address planning and regional control matter.

### 4. Correct answer: C

- **C is correct:** `10.30.0.0/24` does not overlap `10.20.0.0/24`.
- **A is incorrect:** it is the identical range.
- **B is incorrect:** it is contained within the existing `/24`.
- **D is incorrect:** it is also contained within the existing `/24`.

**Decision rule:** primary and secondary subnet ranges in one VPC must be unique, valid CIDR blocks.

### 5. Correct answer: B

- **B is correct:** multiple zones in one region match a zonal-failure requirement without adding a second region unnecessarily.
- **A is incorrect:** the global scope of the VPC does not protect a workload placed in one zone.
- **C is incorrect:** an empty subnet contains no serving or recovery capability.
- **D is incorrect:** dynamic routing mode changes Cloud Router behavior, not application placement or replication.

**Decision rule:** match the architecture to the stated failure domain.

### 6. Correct answer: C

- **C is correct:** regional continuity needs components in another region, protected data, traffic switching and a tested procedure.
- **A is incorrect:** two zones still share the regional failure domain.
- **B is incorrect:** a global network alone does not replicate application or data resources.
- **D is incorrect:** subnet size changes address capacity, not failure-domain coverage.

**Decision rule:** “survive a regional outage” requires a second region, not merely another zone or a global control plane.

### 7. Correct answer: B

- **B is correct:** `gcloud compute networks subnets describe` requires the subnet name and its region.
- **A is incorrect:** it targets a VPC network command and supplies a zone, not a subnet region.
- **C is incorrect:** that command group and `--global` form are not the documented syntax.
- **D is incorrect:** that command hierarchy is not valid.

**Decision rule:** subnet commands live under `gcloud compute networks subnets`, and a subnet is identified with its region.

### 8. Correct answer: C

- **C is correct:** the mandatory feature must exist; choose a supported region or an alternative that satisfies the full requirement, then record trade-offs.
- **A is incorrect:** Google Cloud's global infrastructure does not imply uniform product-feature availability.
- **B is incorrect:** creating a subnet does not add a managed database feature to a region.
- **D is incorrect:** quota cannot make an unavailable regional feature exist.

**Decision rule:** distinguish product availability from quota and network configuration.

### 9. Correct answer: C

- **C is correct:** global dynamic routing affects how Cloud Routers advertise and learn dynamic routes; subnets remain regional and workloads remain where deployed.
- **A is incorrect:** routing mode does not change the scope of subnets.
- **B is incorrect:** it performs no application or data replication.
- **D is incorrect:** networking configuration does not change product availability matrices.

**Decision rule:** the word “global” must be interpreted in the context of the specific feature.

### 10. Correct answer: B

- **B is correct:** this lab avoids VM and database charges, while attached services and data transfer can still be billable.
- **A is incorrect:** the basic empty VPC and subnet objects are not billed as hourly compute resources.
- **C is incorrect:** some cross-zone and cross-region data transfer is billable even within one VPC.
- **D is incorrect:** a VPC with dependencies cannot be deleted until those references are removed, and deleting it does not delete the whole project.

**Decision rule:** estimate the traffic path and attached services, not merely the existence of the network object.
