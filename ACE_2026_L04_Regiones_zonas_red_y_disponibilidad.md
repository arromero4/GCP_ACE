# ACE 2026 — Lección 04: Regiones, zonas, red y disponibilidad

**Fecha:** 24 de septiembre de 2026  
**Fecha de regeneración:** 24 de septiembre de 2026  
**Día del plan:** 4 de 74  
**Cobertura del plan:** Base; guía oficial ACE, apartados 1.1 y 2.3  
**Tema exacto del calendario:** Regiones, zonas, red y disponibilidad  
**Práctica exacta del calendario:** Comparar dos arquitecturas —una de una sola zona y otra de varias zonas— y explicar una decisión de ubicación  
**Idioma:** explicación en español; términos de Google Cloud y preguntas de examen en inglés  
**Duración sugerida:** 75–100 minutos, incluida la práctica y la evaluación  

> Recibir esta lección no demuestra dominio. El aprendizaje se comprueba al explicar las decisiones sin consultar apuntes, completar la práctica y justificar las respuestas.

> **Proyecto de práctica:** SiteOps Tracker es un caso completamente ficticio de portafolio para auditar infraestructura de TI en sedes ficticias. Usa React + TypeScript, Node.js + TypeScript y PostgreSQL; no representa ni reutiliza datos, arquitectura o requisitos de proyectos internos o privados.

---

## 1. Objetivo de aprendizaje

Al terminar esta lección podrás:

1. Explicar con tus propias palabras qué son una **region** y una **zone** en Google Cloud.
2. Diferenciar recursos **globales**, **regionales** y **zonales**.
3. Explicar por qué una VPC es global pero una subred es regional.
4. Relacionar la ubicación con latencia, disponibilidad, residencia de datos, disponibilidad de productos, capacidad y costo.
5. Distinguir una arquitectura de una sola zona, una arquitectura multizona y una arquitectura multirregión.
6. Elegir una ubicación a partir de requisitos, no por memoria ni por preferencia personal.
7. Consultar regiones, zonas, VPC, subredes e instancias mediante la consola y `gcloud`, sin crear recursos.
8. Detectar trampas frecuentes del examen ACE, como asumir que una VPC global convierte una VM zonal en un recurso global.

### Evidencia mínima de hoy

Al final debes producir:

- una tabla con al menos **dos arquitecturas comparadas**;
- un diagrama de una arquitectura de una sola zona;
- un diagrama de una arquitectura distribuida en al menos dos zonas;
- una decisión de ubicación en formato **requisito → elección → justificación → alternativa descartada**;
- el resultado de 10 preguntas en inglés;
- una ficha de errores, aunque hayas obtenido 10/10.

---

## 2. Relación con la guía oficial ACE

La guía oficial adjunta incluye, dentro de **Section 1.1 — Setting up cloud projects and accounts**:

- *Setting up cloud networking*.
- *Verifying product availability across geographical locations (for example, regions and zones)*.

También prepara la base de **Section 2.3 — Planning and implementing networking resources**, donde más adelante se estudian VPC, subredes, conectividad, firewalls y balanceadores.

La pregunta de examen rara vez será solamente “¿qué es una región?”. Lo normal es recibir un escenario con requisitos de continuidad, latencia, regulación, costo o disponibilidad de producto y tener que elegir la mejor ubicación.

---

## 3. Prerrequisitos explicados desde cero

### 3.1 Cuenta, proyecto y ubicación no son lo mismo

- Tu **Google account** identifica a una persona o identidad.
- Un **project** organiza recursos, APIs, permisos, cuotas y facturación.
- Una **region** o **zone** indica dónde se ejecuta o almacena un recurso que tiene alcance geográfico.

Un proyecto no “vive” en una sola región. Dentro del mismo proyecto puedes tener recursos en distintas ubicaciones, siempre que el producto, las políticas y los permisos lo permitan.

### 3.2 Consola y Cloud Shell

La **Google Cloud console** es la interfaz web. **Cloud Shell** proporciona una terminal autenticada con Google Cloud CLI instalada. Antes de ejecutar cualquier comando debes comprobar:

- cuenta activa;
- ID del proyecto activo;
- si existen valores predeterminados de región o zona;
- que estás en un proyecto de laboratorio autorizado.

Un comando correcto ejecutado en el proyecto equivocado sigue siendo un error operativo.

### 3.3 Permisos mínimos

Para consultar regiones y zonas, la documentación de Google indica que el rol predefinido **Compute Viewer** (`roles/compute.viewer`) contiene los permisos de lectura necesarios. No necesitas un rol de administrador para esta práctica.

Si el proyecto de Google Skills no te permite consultar algún recurso, no intentes ampliar tus propios privilegios. Usa la alternativa conceptual incluida más adelante.

### 3.4 Facturación y seguridad

La práctica principal es de solo lectura: no crea VMs, discos, direcciones IP, balanceadores ni bases de datos. Aun así:

- usa preferentemente un proyecto temporal de Google Skills;
- verifica el proyecto antes de cada bloque;
- no pegues tokens, claves privadas ni contraseñas en el archivo;
- recuerda que un presupuesto genera alertas, pero no detiene automáticamente el gasto.

---

## 4. Modelo mental: país, ciudad, edificio y red corporativa

Imagina una empresa con presencia internacional:

- Una **region** se parece a elegir un área metropolitana donde operar.
- Una **zone** se parece a elegir uno de varios campus de infraestructura dentro de esa área.
- Una **VPC network** se parece a la red corporativa lógica que conecta sedes.
- Una **subnet** se parece al segmento IP destinado a una sede o área geográfica.

La analogía ayuda, pero tiene límites: una zona es una abstracción lógica de infraestructura y no debes afirmar que equivale exactamente a un único edificio o centro de datos.

La idea clave es el **failure domain** o dominio de falla. Si todas las copias de una aplicación dependen de la misma zona, comparten ese dominio de falla. Distribuirlas entre zonas reduce el riesgo de que una sola falla zonal afecte a todas al mismo tiempo.

---

## 5. Regiones y zonas, paso a paso

### 5.1 Region

Una **region** es un área geográfica independiente formada por zonas. Ejemplos de nombres son `us-central1`, `europe-west1` o `northamerica-south1`; la lista disponible puede cambiar y siempre debe verificarse en la documentación vigente.

Una región influye en:

- dónde se almacenan o procesan datos para recursos regionales;
- distancia a usuarios y sistemas externos;
- latencia de red;
- disponibilidad de productos y características;
- precios y cargos de transferencia;
- requisitos legales o de residencia de datos;
- independencia ante fallas de otra región.

### 5.2 Zone

Una **zone** es un área de despliegue dentro de una región. El nombre de una zona estándar incluye el nombre de la región y un sufijo, por ejemplo:

```text
us-central1-a
└─ región ─┘└ zona dentro de la región
```

Una VM de Compute Engine es normalmente un recurso zonal. Un disco persistente zonal que se conectará a esa VM debe encontrarse en la misma zona.

### 5.3 ¿Qué protege cada distribución?

| Distribución | Ejemplo conceptual | Puede tolerar por diseño | No resuelve por sí sola |
|---|---|---|---|
| Una zona | Dos servidores en `region-a` | Falla de una VM, solo si hay redundancia real entre las dos | Falla completa de la zona; error compartido de aplicación o datos |
| Varias zonas de una región | Servidor A en `region-a` y B en `region-b` | Falla de una zona, si el tráfico y el estado pueden cambiar a la otra | Falla regional; datos no replicados; mala configuración común |
| Varias regiones | Servicio primario en una región y recuperación o servicio activo en otra | Puede reducir el impacto de una falla regional | No garantiza continuidad sin replicación, enrutamiento, pruebas y procedimientos |

**Regla de examen:** más copias no significan automáticamente alta disponibilidad. Deben evitar el mismo dominio de falla y existir mecanismos de detección, distribución de tráfico y manejo del estado.

---

## 6. Alcance global, regional y zonal

Antes de desplegar, pregunta: **¿cuál es el alcance del recurso?**

| Recurso o concepto | Alcance típico relevante hoy | Consecuencia práctica |
|---|---|---|
| VPC network | Global | La VPC no pertenece a una zona o región específica. |
| Rutas y reglas de firewall asociadas a una VPC | Globales dentro del alcance de la VPC | Pueden afectar recursos en distintas regiones de esa VPC, según sus coincidencias y prioridades. |
| Subnet | Regional | Una subred pertenece a una región y puede ser usada por recursos compatibles de sus zonas. |
| Compute Engine VM | Zonal | Debes identificar la zona al crearla o administrarla. |
| Zonal Persistent Disk | Zonal | Para conectarlo a una VM, ambos deben estar en la misma zona. |
| Static external IP address para Compute Engine | Regional | La VM que la use debe estar en la misma región. |
| Compute Engine image | Global | Puede utilizarse para crear recursos en distintas ubicaciones compatibles. |

### 6.1 La trampa de la VPC global

Una VPC global permite tener subredes en varias regiones bajo la misma red lógica. Eso **no** significa que:

- una VM zonal exista automáticamente en todas las zonas;
- los datos se repliquen automáticamente;
- una aplicación haga failover automáticamente;
- todos los productos estén disponibles en todas las regiones;
- el tráfico entre regiones sea gratuito o tenga la misma latencia que el tráfico local.

### 6.2 La subred regional

Una subred define rangos IP en una región. Recursos ubicados en diferentes zonas de esa misma región pueden usar una subred regional compatible. Por ello, no necesitas crear una subred diferente solo porque cambias de `zone-a` a `zone-b` dentro de la misma región.

### 6.3 Restricciones de acoplamiento

Piensa en “compatibilidad de alcance”:

- VM zonal + disco zonal: misma zona.
- VM zonal + dirección IP estática regional: misma región.
- VM en una zona + subred regional: la zona debe pertenecer a la región de la subred.

En preguntas ACE, una alternativa puede sonar correcta técnicamente pero fallar por incompatibilidad geográfica.

---

## 7. Disponibilidad, redundancia y recuperación

### 7.1 Redundancy

**Redundancy** significa disponer de más de una instancia o componente. Dos VMs idénticas en la misma zona ofrecen redundancia ante la falla de una VM, pero siguen expuestas a una falla zonal.

### 7.2 High availability

**High availability (HA)** busca mantener el servicio disponible durante fallas esperadas mediante redundancia, detección de salud y failover. No basta con duplicar máquinas.

Para una aplicación web multizona suelen ser necesarios, conceptualmente:

1. instancias en al menos dos zonas;
2. health checks que detecten backends no saludables;
3. un mecanismo que dirija tráfico únicamente a backends saludables;
4. una estrategia para sesiones y estado;
5. una capa de datos que también tolere la falla considerada;
6. monitoreo y pruebas.

### 7.3 Disaster recovery

**Disaster recovery (DR)** prepara la recuperación ante eventos de mayor alcance, como la pérdida de una región o corrupción grave de datos. Introduce dos términos útiles:

- **RTO — Recovery Time Objective:** cuánto tiempo puede tardar la recuperación.
- **RPO — Recovery Point Objective:** cuánto dato reciente se puede perder, medido en tiempo.

No es necesario memorizar una arquitectura universal. Primero interpreta el requisito. Si el escenario solo exige tolerar una falla zonal, una solución multirregión puede ser excesiva. Si exige continuar durante una falla regional, dos zonas de la misma región son insuficientes.

---

## 8. Cómo elegir una ubicación

Usa este orden de decisión:

### Paso 1 — Requisitos obligatorios

- residencia o soberanía de datos;
- regulación y políticas de la organización;
- contrato o requisito explícito de ubicación;
- separación obligatoria de dominios de falla.

Una región cercana no sirve si está fuera de la geografía permitida.

### Paso 2 — Disponibilidad del producto y de la característica

Verifica el producto exacto y, cuando corresponda, la característica, tipo de máquina, GPU, TPU o modalidad requerida. Que Compute Engine aparezca en una región no demuestra que todos los productos estén disponibles allí.

**Trampa ACE:** `gcloud compute regions list` enumera regiones de Compute Engine para el proyecto. No es un catálogo universal de disponibilidad de todos los servicios.

### Paso 3 — Latencia y cercanía

Ubica la carga cerca de los usuarios, fuentes de datos o sistemas con los que más se comunica. No confundas cercanía geográfica con una medición real: para una decisión de producción deben medirse rutas y latencias.

### Paso 4 — Resiliencia requerida

- Entorno desechable o desarrollo: una zona puede ser suficiente.
- Producción que debe tolerar una falla zonal: varias zonas de una región.
- Continuidad ante falla regional: arquitectura multirregión y un plan de DR probado.

### Paso 5 — Costo

Compara:

- cantidad de recursos duplicados;
- precio regional del producto;
- transferencia de datos entre zonas o regiones;
- almacenamiento y replicación;
- balanceo, monitoreo y operación;
- costo empresarial del tiempo de inactividad.

Elegir únicamente la región con menor precio puede aumentar la latencia o incumplir requisitos. Elegir siempre multirregión también puede ser un desperdicio.

### Paso 6 — Capacidad, cuotas y operación

La región o zona debe ofrecer capacidad para la carga y cuotas suficientes. El equipo también debe poder operar, monitorear y probar la solución.

### Paso 7 — Sostenibilidad, cuando no contradiga requisitos superiores

Google ofrece herramientas para comparar huella de carbono, precio y latencia. La sostenibilidad puede ser un criterio de desempate, pero no sustituye requisitos obligatorios de seguridad, disponibilidad o residencia.

---

## 9. Ejemplo: SiteOps Tracker

Este ejemplo es hipotético y no afirma requisitos de ningún proyecto real.

Supón que **SiteOps Tracker** recibe evidencias de auditoría de infraestructura desde distintas sedes ficticias. Los usuarios principales están en México y el equipo establece estos requisitos:

1. producción debe seguir atendiendo si falla una zona;
2. no se exige continuidad automática ante la pérdida de una región completa;
3. el producto seleccionado debe estar disponible en la región elegida;
4. desarrollo puede interrumpirse y debe minimizar costo;
5. el diseño debe evitar almacenar estado crítico únicamente en el disco local de una VM.

### 9.1 Opción A — Desarrollo en una sola zona

```text
Proyecto: siteops-dev
VPC global: siteops-vpc
└── Subred regional: app-subnet
    └── Zona A
        └── Una instancia de desarrollo
```

**Ventajas:** menor complejidad, menos recursos y costo potencialmente menor.  
**Riesgo aceptado:** una falla de la zona puede dejar el entorno no disponible.  
**Uso adecuado:** desarrollo desechable o laboratorio, si la interrupción es aceptable.

### 9.2 Opción B — Producción en varias zonas de una región

```text
Proyecto: siteops-prod
VPC global: siteops-vpc
└── Subred regional: app-subnet
    ├── Zona A → instancia de API saludable
    └── Zona B → instancia de API saludable
             ↑
      distribución de tráfico
      + health checks

Capa de datos: debe tener su propia estrategia de HA y respaldo
```

**Ventaja:** una falla de una zona no tiene que interrumpir la API si la otra zona, el tráfico y la capa de datos están preparados.  
**Límite:** ambas zonas siguen perteneciendo a la misma región.  
**Costo:** más capacidad, tráfico, monitoreo y operación que una sola instancia.

### 9.3 Opción C — Multirregión

```text
Región primaria                      Región secundaria
├── aplicación multizona             ├── aplicación o capacidad de recuperación
└── datos primarios  ──replicación──> └── datos de recuperación
                 enrutamiento/failover probado
```

Esta opción sería necesaria si el requisito cambiara a “continuar o recuperarse dentro de un RTO/RPO específico cuando una región completa no esté disponible”. No debe elegirse solo porque “más regiones es mejor”: aumenta costo y complejidad, y requiere resolver replicación, consistencia y failover.

### 9.4 Decisión documentada

```text
Requisito: producción debe tolerar la pérdida de una zona.
Elección: desplegar la capa de aplicación en al menos dos zonas de una misma región compatible.
Por qué: separa las réplicas entre dominios de falla zonales y mantiene proximidad regional.
Qué más hace falta: health checks, distribución de tráfico y HA de la capa de datos.
Alternativa descartada: una sola zona, porque no cumple el requisito de falla zonal.
Alternativa todavía innecesaria: multirregión, porque no existe un requisito de falla regional.
```

---

## 10. Comparación de decisiones y por qué no elegir alternativas

| Requisito principal | Elección razonable | Por qué | Por qué no la alternativa aparente |
|---|---|---|---|
| Laboratorio temporal; interrupción aceptable | Una zona | Menor complejidad y menor cantidad de recursos | Multizona agrega costo y operación sin satisfacer un requisito real adicional. |
| Continuar ante pérdida de una zona | Varias zonas de una región | Separa la aplicación entre dominios de falla zonales | Dos VMs en la misma zona siguen compartiendo la falla zonal. |
| Recuperarse ante pérdida de región | Otra región, con DR o servicio multirregión adecuado | Crea independencia regional | Dos o tres zonas de una misma región no protegen contra la región completa. |
| Datos obligatoriamente en una geografía | Región permitida que soporte el producto | Cumple restricción antes de optimizar latencia/costo | La región “más cercana” podría incumplir residencia de datos. |
| Producto no disponible en región candidata | Elegir otra región compatible o reevaluar producto | La ubicación debe soportar el servicio y la función necesarios | Cambiar solo de zona dentro de la misma región puede no cambiar disponibilidad regional. |
| Una aplicación usa zonas A y B de la misma región | Una subred regional compatible puede atender ambas | La subred tiene alcance regional | Crear una VPC por zona confunde el alcance y añade segmentación innecesaria. |
| Separación fuerte de entornos o administración | Proyectos y, si lo exige el diseño, redes separadas | El límite debe responder a gobierno y riesgo | No uses una región como sustituto de aislamiento IAM o de proyectos. |

### Regla rápida para preguntas ACE

1. Subraya el requisito que no puede violarse.
2. Identifica el dominio de falla mencionado: VM, zona o región.
3. Descarta soluciones que no cubran ese dominio.
4. Entre las restantes, elige la opción administrativamente más simple que cumpla todo.
5. No optimices algo que el escenario no pidió a costa de incumplir lo obligatorio.

---

## 11. Práctica guiada — Consola y CLI

### 11.1 Reglas de seguridad

- Usa un proyecto de laboratorio o uno donde tengas autorización de lectura.
- No ejecutes comandos `create`, `delete`, `update`, `set-iam-policy` ni `services enable` en esta práctica.
- Sustituye los marcadores solo con valores observados en tu proyecto.
- Si la API de Compute Engine no está habilitada y el laboratorio no autoriza habilitarla, usa la alternativa conceptual.
- Los comandos fueron contrastados con la documentación oficial vigente consultada el 24 de septiembre de 2026.

### 11.2 Parte A — Verificación inicial en Cloud Shell

Abre Cloud Shell desde la consola y ejecuta:

```bash
gcloud auth list --filter=status:ACTIVE --format="value(account)"
```

**Anota:** cuenta activa, ocultando parte del correo si compartes la evidencia públicamente.

Comprueba el proyecto:

```bash
gcloud config list project
```

Comprueba propiedades de Compute Engine configuradas en el cliente:

```bash
gcloud config list compute/
```

**Interpretación:** si `region` o `zone` aparecen vacíos, no es un error. Significa que la configuración activa de `gcloud` no tiene esos valores predeterminados. Si aparecen, recuerda que son valores del cliente; no convierten recursos zonales en regionales ni crean redundancia.

Registra:

```text
Cuenta activa:
Project ID:
Región predeterminada:
Zona predeterminada:
¿Proyecto de laboratorio autorizado? Sí / No
```

### 11.3 Parte B — Explorar regiones y zonas con CLI

Lista las regiones visibles para Compute Engine:

```bash
gcloud compute regions list
```

Lista las zonas:

```bash
gcloud compute zones list
```

Selecciona una región que aparezca con estado disponible en tu salida. No copies ciegamente el ejemplo. Describe la región reemplazando `REGION`:

```bash
gcloud compute regions describe REGION
```

Identifica en la respuesta:

- `name`;
- `status`;
- lista de `zones`;
- al menos una cuota regional.

Después describe una de sus zonas reemplazando `ZONE`:

```bash
gcloud compute zones describe ZONE
```

**Evidencia:** escribe una región real observada y al menos dos zonas pertenecientes a ella.

```text
Región elegida:
Estado observado:
Zona 1:
Zona 2:
Cuota regional observada:
```

> No concluyas que todos los productos están disponibles solo porque la región aparece en este comando. Confirma el servicio exacto en la página oficial de ubicaciones o en la documentación del servicio.

### 11.4 Parte C — Explorar VPC y subredes con CLI

Lista las VPC del proyecto:

```bash
gcloud compute networks list
```

Lista las subredes de todas las regiones:

```bash
gcloud compute networks subnets list
```

Si observaste `us-central1` y existe una subred allí, puedes limitar la consulta usando el ejemplo oficial:

```bash
gcloud compute networks subnets list --filter="region:( us-central1 )"
```

Si elegiste otra región, sustituye `us-central1` por esa región.

Contesta:

1. ¿Cuántas VPC aparecen?
2. ¿En qué regiones aparecen subredes?
3. ¿La salida muestra una zona para la VPC? ¿Por qué no debería necesitarla?
4. ¿La salida muestra una región para cada subred? ¿Qué demuestra?

Que la lista esté vacía no prueba que el comando falló: el proyecto puede no tener redes visibles o tu identidad puede carecer de permisos. Lee el mensaje y distingue salida vacía de error.

### 11.5 Parte D — Inventario de VMs, si el proyecto ya las contiene

Este comando solo consulta instancias existentes:

```bash
gcloud compute instances list
```

No crees VMs para completar esta lección. Si existen instancias, anota sus zonas y decide si están concentradas en un solo dominio de falla. Si no existen, registra “sin instancias” y continúa.

### 11.6 Parte E — Verificación en la consola

1. Confirma en el selector superior el mismo proyecto usado por `gcloud`.
2. Abre **Compute Engine → Zones** o la página de zonas enlazada en la documentación.
3. Localiza la región y las dos zonas anotadas.
4. Abre **VPC network → VPC networks**.
5. Observa una red y sus subredes, si existen.
6. Comprueba que la VPC se presenta como red y que las subredes tienen región.
7. Abre la página oficial **Cloud locations** y verifica el producto exacto que usarías; no te limites a la lista de Compute Engine.

No guardes cambios ni crees recursos.

### 11.7 Parte F — Comparación arquitectónica obligatoria

Completa esta tabla para SiteOps Tracker:

| Criterio | Arquitectura de una zona | Arquitectura de varias zonas |
|---|---|---|
| Número mínimo conceptual de zonas | | |
| ¿Tolera falla total de su única zona? | | |
| Distribución de tráfico necesaria | | |
| Tratamiento de sesiones/estado | | |
| Tratamiento de la base de datos | | |
| Complejidad | | |
| Costo relativo | | |
| Uso recomendado | | |

Dibuja las dos alternativas. Puedes usar este formato:

```text
ARQUITECTURA 1 — UNA ZONA
[Usuarios] -> [Entrada] -> [Aplicación en zona ___] -> [Datos]

Punto único de falla zonal:

ARQUITECTURA 2 — VARIAS ZONAS
                         -> [Aplicación en zona ___]
[Usuarios] -> [Entrada] -|
                         -> [Aplicación en zona ___]
                                      |
                           [Datos con estrategia HA]

Mecanismo de health check/failover:
```

### 11.8 Parte G — Decisión de ubicación

Responde sin usar “porque sí” ni “es más seguro”:

```text
Requisitos obligatorios del escenario:
Usuarios o sistemas cercanos:
Restricción de residencia de datos:
Producto y característica que deben verificarse:
Falla que debe tolerarse:
Región candidata:
Zonas candidatas:
Arquitectura elegida:
Justificación:
Alternativa descartada:
Por qué se descartó:
Riesgo residual:
```

### 11.9 Resultado esperado

La práctica está completa si puedes comprobar todos estos puntos:

- [ ] Verifiqué cuenta y proyecto antes de consultar recursos.
- [ ] Identifiqué una región y dos zonas de esa misma región.
- [ ] Expliqué que una región listada para Compute Engine no garantiza disponibilidad de todos los servicios.
- [ ] Identifiqué VPC como recurso global y subred como recurso regional.
- [ ] No creé ni modifiqué recursos.
- [ ] Comparé una arquitectura de una zona con otra multizona.
- [ ] Mencioné health checks, distribución de tráfico y estado/datos.
- [ ] Expliqué por qué multizona no equivale a multirregión.
- [ ] Documenté una decisión y descarté al menos una alternativa.

---

## 12. Alternativa conceptual completa sin cuenta ni crédito

Si no puedes acceder a Google Cloud, usa esta salida simulada:

```text
REGIONS
NAME              STATUS
example-region1   UP
example-region2   UP

ZONES
NAME                  REGION            STATUS
example-region1-a     example-region1   UP
example-region1-b     example-region1   UP
example-region1-c     example-region1   UP

NETWORKS
NAME           SUBNET_MODE
audit-vpc      CUSTOM

SUBNETS
NAME            REGION            NETWORK       RANGE
audit-app       example-region1   audit-vpc     10.10.0.0/24
audit-dr        example-region2   audit-vpc     10.20.0.0/24

INSTANCES
NAME            ZONE                  STATUS
audit-api-1     example-region1-a     RUNNING
audit-api-2     example-region1-b     RUNNING
```

Realiza estas tareas:

1. Marca cada recurso como global, regional o zonal.
2. Explica qué componente cubre dos zonas.
3. Explica por qué `audit-vpc` no vuelve globales a las instancias.
4. Decide si la aplicación toleraría una falla de `example-region1-a`, suponiendo que existe balanceo y una capa de datos con HA.
5. Decide si toleraría la pérdida completa de `example-region1`; justifica qué información falta.
6. Completa las secciones 11.7 y 11.8 usando estos datos.

Esta alternativa cubre íntegramente el objetivo conceptual, aunque no acredita experiencia práctica real con la consola.

---

## 13. Solución de problemas

### Problema 1 — `PERMISSION_DENIED`

**Causa probable:** la identidad no tiene permisos de lectura suficientes.  
**Acción:** confirma cuenta y proyecto; solicita al administrador el acceso de lectura apropiado. Para regiones y zonas, la documentación menciona `roles/compute.viewer`. No te otorgues roles elevados en un entorno ajeno.

### Problema 2 — La API de Compute Engine no está habilitada

**Síntoma:** el comando indica que `compute.googleapis.com` está deshabilitada.  
**Acción:** en un laboratorio, sigue solo las instrucciones del laboratorio. En un proyecto propio, revisa autorización y facturación antes de habilitarla. Si no está autorizado, realiza la alternativa conceptual. No habilites servicios en producción solo para completar una práctica.

### Problema 3 — La región o zona no existe

**Causa probable:** error de escritura, ubicación no disponible para el proyecto o nombre copiado de una referencia antigua.  
**Acción:** vuelve a ejecutar `gcloud compute regions list` y `gcloud compute zones list`; copia el nombre exacto de la salida vigente.

### Problema 4 — Región predeterminada y zona predeterminada no coinciden

Google permite configurar valores predeterminados que no sean coherentes entre sí. Comprueba `gcloud config list compute/`. Para el examen, recuerda que un valor predeterminado solo ayuda al cliente a construir solicitudes; no valida por sí solo el diseño.

### Problema 5 — No aparecen VPC, subredes o VMs

Puede ser correcto: el proyecto quizá no contenga esos recursos. Verifica proyecto y permisos. Distingue una tabla vacía de un mensaje de error.

### Problema 6 — Un producto no aparece en la región

No asumas que otra zona de la misma región resolverá el problema. Revisa la página de ubicaciones y la documentación específica del servicio. Después elige otra región permitida o reevalúa el producto.

### Refuerzo por un error compartido anteriormente — SSH timeout o backend `UNHEALTHY`

Una VM inaccesible por SSH o un backend no saludable no demuestra automáticamente que elegiste la zona equivocada.

Primero confirma:

1. proyecto, región y zona indicados en el comando;
2. que la instancia existe y está en ejecución;
3. método de acceso: IP externa, IAP u otra ruta autorizada;
4. rutas y reglas de firewall aplicables;
5. permisos de acceso y configuración de OS Login, cuando aplique;
6. para health checks: puerto, protocolo, ruta, proceso que escucha y reglas que permiten los rangos del verificador.

La geografía define alcance y dominios de falla; no sustituye el diagnóstico de conectividad. Este tema se retomará con profundidad en las lecciones de red, MIG y operación.

---

## 14. Impacto en costos y limpieza

### Impacto de esta práctica

Los pasos obligatorios son consultas de inventario. No crean recursos facturables como VMs, discos, IP reservadas, balanceadores o bases de datos. Pueden usar cuotas de API, pero el diseño evita generar consumo de infraestructura.

### Impacto de las arquitecturas estudiadas

- Una arquitectura multizona suele requerir más capacidad que una instancia única.
- Una arquitectura multirregión añade replicación, transferencia, almacenamiento y operación.
- El tráfico entre ubicaciones puede tener costos y latencias distintos.
- El costo debe compararse con el impacto de una interrupción y con los requisitos obligatorios.

### Limpieza

No se requiere borrar recursos porque la práctica no crea ninguno. Antes de cerrar:

```bash
gcloud compute instances list
```

Confirma que no creaste una VM por accidente. Si observas un recurso inesperado, no lo borres sin verificar propiedad, propósito y autorización. En un laboratorio temporal, usa el mecanismo de finalización del propio laboratorio.

---

## 15. Glosario bilingüe

| English term | Español | Explicación breve |
|---|---|---|
| Region | Región | Área geográfica independiente formada por zonas. |
| Zone | Zona | Área de despliegue dentro de una región. |
| Location | Ubicación | Término general para región, zona, multirregión u otro alcance admitido. |
| Failure domain | Dominio de falla | Conjunto de componentes que podrían verse afectados por una misma falla. |
| Zonal resource | Recurso zonal | Recurso ligado a una zona, como una VM. |
| Regional resource | Recurso regional | Recurso ligado a una región y utilizable según las reglas del producto. |
| Global resource | Recurso global | Recurso no ligado a una única región o zona. |
| VPC network | Red VPC | Red virtual global de Google Cloud. |
| Subnet / subnetwork | Subred | Segmento IP regional dentro de una VPC. |
| High availability (HA) | Alta disponibilidad | Diseño para mantener el servicio durante fallas previstas. |
| Redundancy | Redundancia | Existencia de componentes duplicados. |
| Failover | Conmutación por falla | Cambio de servicio o tráfico al componente disponible. |
| Health check | Comprobación de estado | Prueba que determina si un backend puede recibir tráfico. |
| Disaster recovery (DR) | Recuperación ante desastres | Plan para restaurar servicio y datos tras una interrupción grave. |
| Recovery Time Objective (RTO) | Objetivo de tiempo de recuperación | Tiempo máximo objetivo para restablecer el servicio. |
| Recovery Point Objective (RPO) | Objetivo de punto de recuperación | Pérdida de datos máxima aceptable expresada como tiempo. |
| Data residency | Residencia de datos | Requisito sobre la ubicación geográfica de los datos. |
| Latency | Latencia | Tiempo que tarda una comunicación en viajar y recibir respuesta. |
| Product availability | Disponibilidad del producto | Regiones o ubicaciones donde un servicio o función puede utilizarse. |
| Quota | Cuota | Límite de uso de un recurso o API. |
| Capacity | Capacidad | Recursos físicos/lógicos disponibles para atender una solicitud. |
| Data transfer | Transferencia de datos | Movimiento de datos entre componentes o ubicaciones. |
| Default region/zone | Región/zona predeterminada | Propiedad del cliente que completa solicitudes; no crea HA. |

---

## 16. Repaso activo de días hábiles previos

Responde primero sin mirar las soluciones.

### Lección 01 — Nube, proyectos y servicios

1. ¿Por qué un proyecto no es una región?
2. Ordena: recurso, proyecto, carpeta, organización.
3. ¿Qué responsabilidades agrupa un proyecto?

### Lección 02 — Consola, Cloud Shell y `gcloud`

4. ¿Qué dos valores debes confirmar antes de ejecutar un comando?
5. ¿Cuál es la diferencia operativa entre un comando de consulta y uno de mutación?
6. ¿Qué implica que `gcloud` tenga una zona predeterminada?

### Lección 03 — Costos, facturación y presupuesto

7. ¿Un presupuesto detiene automáticamente el gasto?
8. Menciona tres acciones para evitar costos inesperados al terminar un laboratorio.
9. ¿Por qué una arquitectura multirregión suele costar más que una de una zona?

### Recuperación a 7 y 21 días

- **Hace 7 días:** no aplica; el plan comenzó el 21/09/2026 y hoy es el día 4.
- **Hace 21 días:** no aplica; todavía no existen lecciones del plan con esa antigüedad.

No sustituyas estas ventanas por temas futuros. Empezarán a activarse cuando haya material realmente estudiado en esas fechas.

### Respuestas de comprobación

1. El proyecto es un contenedor lógico de recursos, APIs, IAM, cuotas y facturación; puede contener recursos en varias ubicaciones.
2. Organización → carpeta → proyecto → recurso.
3. Recursos, servicios/APIs, permisos, cuotas y asociación de facturación, entre otros.
4. Cuenta activa y Project ID; después revisa región/zona cuando el comando dependa de ellas.
5. Consulta lee estado; mutación crea, modifica o elimina estado y exige más cuidado/autorización.
6. El cliente puede usarla cuando omites `--zone`; no crea redundancia ni mueve recursos.
7. No. El presupuesto alerta; no es un límite duro de gasto.
8. Verificar recursos, detener/eliminar lo creado cuando esté autorizado, revisar discos/IP/balanceadores y completar la limpieza del laboratorio.
9. Por duplicación, replicación, transferencia de datos, almacenamiento, balanceo, monitoreo y complejidad operativa.

---

## 17. Preguntas originales de examen en inglés

**Instructions:** Choose the best answer unless the question says **Choose two**.

### Question 1

A production API serves users concentrated in one geography. The application must remain available if a single Google Cloud zone becomes unavailable, but it is not required to survive a full regional outage. What should the team do?

A. Run two VM instances in the same zone.  
B. Run application instances in at least two zones in the same region and use health-aware traffic distribution.  
C. Run one VM and create two VPC networks.  
D. Run one VM in the closest zone and take daily screenshots of its configuration.

### Question 2

Which statement correctly describes VPC networks and subnets in Google Cloud?

A. Both VPC networks and subnets are zonal resources.  
B. A VPC network is regional, while its subnets are global.  
C. A VPC network is global, while its subnets are regional.  
D. A VPC network becomes global only after two VMs are created in different regions.

### Question 3

A VM uses a zonal Persistent Disk. Which placement rule must be satisfied for the disk to be attached to the VM?

A. The disk and VM must be in the same zone.  
B. The disk can be in any region because disks are global.  
C. The disk and VM must be in different zones for high availability.  
D. Only the VPC name must match.

### Question 4

An engineer runs `gcloud compute regions list` and sees that a candidate region is `UP`. The application also requires a specific managed database feature. What should the engineer do next?

A. Assume the database feature is available because Compute Engine reports the region as `UP`.  
B. Verify the exact product and feature availability in the service-specific location documentation.  
C. Change the default zone; this enables every product in the region.  
D. Create the database in an unsupported region and wait for automatic migration.

### Question 5

A company requires application data to remain within an approved geography. Among compliant regions, it wants low latency for its users. What is the best selection process?

A. Select the globally cheapest region, regardless of geography.  
B. Select the closest zone first and review compliance after deployment.  
C. Filter to compliant locations that support the required services, then compare latency, resilience, and cost.  
D. Use any region because a global VPC makes all data global.

### Question 6

A disposable development environment can tolerate downtime and must minimize operational complexity and resource cost. Which placement is most appropriate?

A. A single-zone deployment, with the downtime risk documented.  
B. An active-active deployment across three regions.  
C. Two regions with synchronous replication, regardless of application needs.  
D. One separate VPC for every zone in Google Cloud.

### Question 7

An application must recover when its entire primary region is unavailable. Which design direction best addresses this requirement?

A. Place all replicas in different racks in one zone.  
B. Place replicas in two zones of the same region only.  
C. Add a secondary region and implement tested data replication and failover that meet the required RTO and RPO.  
D. Set a default region in `gcloud`.

### Question 8

An engineer configured a default zone in `gcloud`. What does this configuration provide?

A. Automatic multi-zone replication for every new resource.  
B. A client-side default used when supported commands omit an explicit zone.  
C. A guarantee that all products exist in that zone.  
D. Regional disaster recovery.

### Question 9 — Choose two

Which two commands are appropriate read-only starting points for confirming network scope in an existing project?

A. `gcloud compute networks list`  
B. `gcloud compute networks subnets list`  
C. `gcloud compute networks create audit-vpc`  
D. `gcloud compute networks delete default`

### Question 10 — Choose two

A backend in a managed instance group is reported as `UNHEALTHY`, and SSH to one VM times out. Which two actions are appropriate early diagnostic steps?

A. Confirm the active project and the VM's exact zone.  
B. Review access path, firewall rules, health-check port/path, and whether the service is listening.  
C. Move every resource to another region immediately without collecting evidence.  
D. Conclude that the VPC cannot connect resources because it is global.

---

## 18. Soluciones justificadas

### Answer 1 — B

**Why B is correct:** instances in multiple zones reduce dependence on one zonal failure domain; health-aware traffic distribution is needed to stop sending traffic to failed backends.

- **A is wrong:** two VMs in one zone still share the zonal failure.
- **C is wrong:** extra VPCs do not create application redundancy.
- **D is wrong:** documentation or screenshots do not keep the workload available.

### Answer 2 — C

**Why C is correct:** Google Cloud VPC networks are global resources, while subnets are regional resources.

- **A is wrong:** neither statement is correct.
- **B is wrong:** it reverses the scopes.
- **D is wrong:** the scope is a product property, not a result of creating VMs.

### Answer 3 — A

**Why A is correct:** a zonal Persistent Disk and the VM using it must be in the same zone.

- **B is wrong:** a zonal disk is not global.
- **C is wrong:** separating a VM from its zonal disk prevents that attachment; HA requires a different storage strategy.
- **D is wrong:** network identity does not override disk placement constraints.

### Answer 4 — B

**Why B is correct:** Compute Engine's region status does not prove availability of another product or a specific feature.

- **A is wrong:** it generalizes Compute Engine information to every service.
- **C is wrong:** a CLI default neither deploys nor enables products.
- **D is wrong:** unsupported placement should not be expected to migrate automatically.

### Answer 5 — C

**Why C is correct:** mandatory compliance and service availability must be satisfied before optimizing latency, resilience, and cost.

- **A is wrong:** it can violate data residency.
- **B is wrong:** compliance must be considered before deployment.
- **D is wrong:** VPC scope does not determine or erase data residency.

### Answer 6 — A

**Why A is correct:** when downtime is explicitly acceptable, a single-zone design can minimize resources and operational complexity; the accepted risk should be recorded.

- **B is wrong:** it is excessive for the stated requirement.
- **C is wrong:** synchronous cross-region replication adds complexity and may not be supported or required.
- **D is wrong:** creating VPCs per zone is not a valid availability requirement and adds unnecessary complexity.

### Answer 7 — C

**Why C is correct:** surviving a regional failure requires independence from the primary region, plus replication and tested failover aligned with RTO/RPO.

- **A is wrong:** one zone remains a single zonal and regional dependency.
- **B is wrong:** two zones protect against a zonal failure, not necessarily a regional failure.
- **D is wrong:** client configuration is not disaster recovery.

### Answer 8 — B

**Why B is correct:** the default helps supported client commands construct requests when an explicit zone is omitted.

- **A is wrong:** it does not replicate resources.
- **C is wrong:** product availability must be verified separately.
- **D is wrong:** a property value is not a recovery architecture.

### Answer 9 — A and B

**Why A and B are correct:** both commands list existing resources and let you observe the global network and regional subnet relationship.

- **C is wrong:** `create` mutates cloud state and is unnecessary for a read-only inventory.
- **D is wrong:** `delete` is destructive and could interrupt workloads.

### Answer 10 — A and B

**Why A and B are correct:** project/zone context and the actual network/application health path are high-value early checks for both targeting errors and connectivity failures.

- **C is wrong:** moving regions before diagnosis is risky, expensive, and unsupported by the evidence.
- **D is wrong:** a global VPC is designed to provide connectivity across its resources subject to routes, firewalls and product rules; “global” is not the cause of failure.

### Calificación

En preguntas **Choose two**, cuenta el punto solo si seleccionaste ambas respuestas correctas y ninguna incorrecta.

```text
Puntuación: ___ / 10
Porcentaje: ___ %
Tiempo: ___ minutos
Preguntas dudosas aunque acertadas:
```

Meta formativa de hoy: al menos 8/10 y capacidad de explicar cada distractor. Esta meta es del plan personal; no es una nota de aprobación oficial de Google.

---

## 19. Ficha de errores

Completa una fila por cada error y por cada acierto adivinado.

| Pregunta o paso | Mi respuesta/acción | Correcta | Tipo de error | Regla de decisión corregida | Cómo lo explicaré mañana |
|---|---|---|---|---|---|
| | | | Concepto / lectura / inglés / CLI / distracción | | |
| | | | | | |
| | | | | | |

Tipos de error sugeridos:

- confundí región con zona;
- confundí alcance de VPC y subred;
- ignoré una restricción obligatoria;
- elegí la solución más grande en vez de la mínima suficiente;
- asumí disponibilidad de producto;
- confundí redundancia con HA;
- olvidé la capa de datos o el failover;
- ejecuté o propuse una mutación innecesaria;
- interpreté mal una frase en inglés.

### Registro de error en una frase

```text
Antes pensaba que ____________________.
Ahora usaré la regla ____________________ porque ____________________.
```

---

## 20. Criterios de autoevaluación

Asigna 0, 1 o 2 puntos:

- **0:** no puedo hacerlo.
- **1:** puedo hacerlo con apuntes.
- **2:** puedo hacerlo sin apuntes y justificarlo.

| Criterio | 0–2 |
|---|---:|
| Defino región y zona sin confundirlas. | |
| Identifico el dominio de falla de una arquitectura. | |
| Clasifico VPC, subred, VM y disco zonal por alcance. | |
| Explico por qué VPC global no significa aplicación global. | |
| Elijo una región considerando requisitos obligatorios primero. | |
| Verifico producto y característica por ubicación. | |
| Distingo una solución multizona de una multirregión. | |
| Explico por qué dos VMs no bastan sin tráfico, salud y datos. | |
| Ejecuto consultas después de verificar cuenta y proyecto. | |
| Justifico por qué descarto alternativas. | |

```text
Total: ___ / 20
```

Interpretación personal:

- **17–20:** buen desempeño para continuar; registra dudas residuales.
- **13–16:** repasa las secciones con 0 o 1 y vuelve a explicar el caso SiteOps Tracker.
- **0–12:** repite la práctica conceptual y las preguntas antes de avanzar.

Esto no certifica dominio ni predice por sí solo el resultado del examen.

---

## 21. Cierre de recuperación activa

Sin mirar arriba, completa:

1. Una región es ________________________________________________.
2. Una zona es __________________________________________________.
3. VPC es ____________; una subred es ____________; una VM es normalmente ____________.
4. Para tolerar una falla zonal necesito __________________________.
5. Para una falla regional, varias zonas de la misma región son insuficientes porque __________________.
6. Antes de elegir ubicación verifico, en este orden: ______________.
7. Ver una región `UP` en Compute Engine no demuestra ______________.
8. Dos réplicas no crean HA si ____________________________________.

### Explicación oral de 60 segundos

Explica en voz alta, en español e incluyendo estos términos en inglés:

> “region”, “zone”, “failure domain”, “VPC network”, “subnet”, “high availability” y “failover”.

Si no puedes hacerlo sin leer, marca el tema para repaso; no lo conviertas en una afirmación de dominio.

---

## 22. Documentación oficial consultada

Documentación comprobada el **24 de septiembre de 2026**:

- [Associate Cloud Engineer exam guide](https://cloud.google.com/learn/certification/guides/cloud-engineer)
- [Compute Engine — Regions and zones](https://docs.cloud.google.com/compute/docs/regions-zones)
- [View available regions and zones](https://docs.cloud.google.com/compute/docs/regions-zones/viewing-regions-zones)
- [Changing the default region or zone](https://docs.cloud.google.com/compute/docs/regions-zones/changing-default-zone-region)
- [Google Cloud global locations and product availability](https://cloud.google.com/about/locations)
- [VPC networks](https://docs.cloud.google.com/vpc/docs/vpc)
- [`gcloud auth list`](https://docs.cloud.google.com/sdk/gcloud/reference/auth/list)
- [`gcloud config list`](https://docs.cloud.google.com/sdk/gcloud/reference/config/list)
- [`gcloud compute regions describe`](https://docs.cloud.google.com/sdk/gcloud/reference/compute/regions/describe)
- [`gcloud compute zones list`](https://docs.cloud.google.com/sdk/gcloud/reference/compute/zones/list)
- [`gcloud compute zones describe`](https://docs.cloud.google.com/sdk/gcloud/reference/compute/zones/describe)
- [`gcloud compute networks list`](https://docs.cloud.google.com/sdk/gcloud/reference/compute/networks/list)
- [`gcloud compute networks subnets list`](https://docs.cloud.google.com/sdk/gcloud/reference/compute/networks/subnets/list)
- [`gcloud compute instances list`](https://docs.cloud.google.com/sdk/gcloud/reference/compute/instances/list)

La disponibilidad de regiones, zonas, productos, características y precios cambia. Vuelve a consultar la documentación del servicio antes de una implementación real.

---

## 23. Registro de finalización

```text
Fecha y hora:
Proyecto de laboratorio o alternativa conceptual:
Práctica completada: Sí / Parcial / No
Puntuación: ___ / 10
Autoevaluación: ___ / 20
Tiempo total:
Concepto más claro:
Concepto que debo reforzar:
Error principal:
Regla de decisión que conservaré:
Pregunta para la siguiente sesión:
```

**Siguiente lección del plan:** Lección 05 — Método ACE: CLI, escenarios y vocabulario técnico en inglés.
