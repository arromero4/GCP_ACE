# ACE 2026 - Lección 20: Discos de VM: Persistent Disk zonal/regional y Hyperdisk

**Fecha:** 16 de octubre de 2026  
**Día del plan:** 20 de 74  
**Dominio de la guía ACE:** 2.1 - Planning and implementing compute resources  
**Práctica del plan:** seleccionar almacenamiento para tres cargas con rendimiento y recuperación distintos  
**Caso transversal:** SiteOps Tracker, proyecto ficticio de portafolio  
**Duración sugerida:** 75-90 minutos  
**Documentación y comandos verificados:** 16 de octubre de 2026

> Esta lección enseña y permite practicar. Recibirla no demuestra dominio: compruébalo con la práctica, las preguntas y los criterios de autoevaluación.

## Objetivo

Al terminar la sesión podrás:

1. Explicar la diferencia entre almacenamiento de bloques duradero y almacenamiento efímero.
2. Distinguir un Persistent Disk zonal de uno regional sin confundir réplica con copia de seguridad.
3. Elegir entre `pd-standard`, `pd-balanced`, `pd-ssd` y las familias principales de Hyperdisk según IOPS, throughput, latencia, disponibilidad y costo.
4. Reconocer que el rendimiento de Persistent Disk suele crecer con la capacidad, mientras que Hyperdisk permite aprovisionar capacidad y rendimiento por separado dentro de sus límites.
5. Crear e inspeccionar un disco zonal en la consola, un Persistent Disk regional y un Hyperdisk con Google Cloud CLI.
6. Justificar tres decisiones de almacenamiento para SiteOps Tracker y limpiar todos los recursos creados.

## Prerrequisitos, explicados desde cero

### 1. Proyecto y facturación

Un **project** es el contenedor administrativo de los recursos. Para crear discos reales, el proyecto debe tener facturación habilitada o ser un proyecto temporal de Google Skills. Un disco empieza a generar cargos cuando se crea, aunque no esté conectado a una VM.

Si no tienes cuenta, crédito o permisos, usa la alternativa conceptual completa de esta lección. Permite practicar la misma decisión sin crear recursos.

### 2. Compute Engine API

La API `compute.googleapis.com` debe estar habilitada. Habilitar una API permite llamar al servicio; no crea por sí solo una VM ni un disco.

### 3. Permisos mínimos para el laboratorio

Para crear y usar un disco intervienen, entre otros, estos permisos:

- `compute.disks.create` para crearlo.
- `compute.disks.use` para conectarlo.
- `compute.instances.attachDisk` para conectarlo a una VM.

En un laboratorio, un rol como **Compute Instance Admin (v1)** puede contener los permisos necesarios, pero en producción se debe conceder el rol mínimo que corresponda al trabajo real. No otorgues Owner solo para evitar analizar permisos.

### 4. Región, zona y alcance

- Una **region** es un área geográfica, por ejemplo `us-central1`.
- Una **zone** es una ubicación de despliegue dentro de una región, por ejemplo `us-central1-a`.
- Un disco **zonal** vive en una zona y normalmente se conecta a una VM de esa zona.
- Un disco **regional** mantiene réplicas síncronas en dos zonas de la misma región y se conecta a una VM ubicada en una de esas zonas de réplica.

### 5. Tres medidas de rendimiento

- **IOPS:** operaciones de entrada/salida por segundo. Importa mucho cuando hay muchas lecturas y escrituras pequeñas y aleatorias, como índices de una base de datos.
- **Throughput:** cantidad de datos transferidos por segundo, normalmente MiB/s. Importa cuando se leen o escriben archivos grandes de forma secuencial.
- **Latency:** tiempo que tarda una operación individual. Importa en transacciones sensibles al retraso.

Más IOPS no significa automáticamente más throughput, y un disco rápido no puede superar el límite de la VM a la que está conectado.

### 6. RPO y RTO

- **Recovery Point Objective (RPO):** cuánto dato reciente se puede perder, medido en tiempo.
- **Recovery Time Objective (RTO):** cuánto tiempo puede permanecer indisponible el servicio.

Un disco regional ayuda a reducir RPO y RTO ante la pérdida de una zona, pero no reemplaza una estrategia de recuperación completa.

## Caso de trabajo: SiteOps Tracker

SiteOps Tracker es una aplicación ficticia que usa:

- React + TypeScript para la interfaz.
- Node.js + TypeScript para la API.
- PostgreSQL para sedes, activos, hallazgos, estados, responsables e historial.
- Servicios de Google Cloud para cómputo, datos, observabilidad y seguridad.

Hoy se analizan tres cargas hipotéticas en Compute Engine:

1. **Entorno de desarrollo:** archivos temporales de importación y pruebas; una interrupción zonal es aceptable y se puede reconstruir el contenido.
2. **PostgreSQL autogestionado de producción:** conserva hallazgos e historial; el negocio requiere que los datos confirmados sobrevivan a la falla de una zona y acepta un procedimiento de failover.
3. **Reindexación intensiva:** procesa grandes lotes de historial y exige más IOPS y throughput sin inflar innecesariamente la capacidad.

La lección no recomienda autogestionar PostgreSQL por defecto. Se usa ese escenario para aprender discos. En una arquitectura real también se compararía con Cloud SQL o AlloyDB.

## 1. Modelo mental: casilleros, dos sucursales y un motor regulable

Imagina el almacenamiento como el archivo físico de una empresa:

- **Persistent Disk zonal:** un casillero resistente dentro de una sola sede. Si cambias la computadora, el casillero sigue existiendo; si toda la sede queda inaccesible, también queda inaccesible el casillero.
- **Persistent Disk regional:** dos casilleros sincronizados en dos sedes de la misma ciudad. Cada escritura confirmada llega a ambas copias. Si una sede falla, los documentos permanecen en la otra, pero todavía debes trasladar el trabajo a una máquina funcional.
- **Hyperdisk:** un casillero cuyo tamaño y velocidad de atención se contratan por separado. Puedes necesitar poco espacio pero muchas operaciones por segundo.
- **Snapshot:** una fotografía puntual del casillero. Sirve para recuperar una versión anterior; no atiende escrituras en tiempo real ni sustituye el segundo casillero sincronizado.
- **Local SSD:** una mesa de trabajo muy rápida unida al equipo físico. Es excelente para datos temporales, pero no debe albergar la única copia de datos duraderos.

La idea decisiva es separar cuatro preguntas:

1. ¿Los datos deben sobrevivir a la VM?
2. ¿Deben seguir disponibles si falla una zona?
3. ¿Necesito muchas operaciones pequeñas o gran transferencia secuencial?
4. ¿Necesito recuperar una versión anterior por borrado, corrupción o error humano?

## 2. Qué es el almacenamiento de bloques duradero

Persistent Disk y Hyperdisk ofrecen **durable block storage** conectado por la red de Google. El sistema operativo los ve como dispositivos de bloque, parecidos a discos físicos, pero su ciclo de vida puede ser independiente del de la VM.

Consecuencias prácticas:

- Detener una VM no elimina el disco.
- Un disco desasociado sigue existiendo y sigue generando cargos.
- Un disco de datos puede separarse de una VM y conectarse a otra compatible.
- Un disco nuevo y vacío debe formatearse y montarse antes de que Linux pueda guardar archivos en él.
- El cifrado en reposo y en tránsito se aplica de forma predeterminada; la elección de claves administradas por Google o por el cliente es otro eje de diseño.

### Bloque no es objeto ni archivo compartido

| Necesidad | Opción habitual | Por qué |
|---|---|---|
| Disco de arranque o volumen de una VM | Persistent Disk o Hyperdisk | El sistema operativo necesita un dispositivo de bloques. |
| Objetos servidos por HTTP, evidencias y archivos estáticos | Cloud Storage | API de objetos, gran durabilidad y clases de almacenamiento; no se monta como un disco POSIX normal por defecto. |
| Sistema de archivos NFS compartido | Filestore | Varios clientes consumen un servicio de archivos administrado. |
| Datos temporales de muy baja latencia ligados al host | Local SSD | Alto rendimiento local, pero datos efímeros. |
| PostgreSQL administrado | Cloud SQL o AlloyDB | Google gestiona gran parte de parches, backups y alta disponibilidad; no eliges un disco de VM directamente. |

## 3. Persistent Disk zonal

Un Persistent Disk zonal almacena datos con redundancia dentro de una zona. Su recurso tiene alcance zonal y debe estar en la misma zona que la VM a la que se conecta.

### Tipos principales

| Tipo | Nombre CLI | Señal de uso | No elegirlo cuando... |
|---|---|---|---|
| Standard Persistent Disk | `pd-standard` | Capacidad económica y acceso principalmente secuencial | La carga necesita IOPS altos o latencia tipo SSD. |
| Balanced Persistent Disk | `pd-balanced` | Uso general con equilibrio entre precio y rendimiento | Hay un requisito intenso y medido de IOPS que justifica SSD o Hyperdisk. |
| SSD Persistent Disk | `pd-ssd` | Cargas sensibles a latencia y muchas operaciones aleatorias | La carga es fría, secuencial y el costo domina. |
| Extreme Persistent Disk | `pd-extreme` | IOPS aprovisionadas para cargas compatibles muy exigentes | Hyperdisk es compatible y ofrece mejor ajuste o la carga no justifica el costo. |

En Persistent Disk, el rendimiento depende del tipo, del tamaño total de los discos del mismo tipo conectados y de los límites de la VM. Aumentar capacidad puede aumentar el rendimiento, pero pagar capacidad sin necesitarla es una señal para evaluar Hyperdisk.

### Cuándo encaja en SiteOps Tracker

Para el entorno de desarrollo, un `pd-balanced` zonal de tamaño pequeño es razonable si:

- el contenido puede reconstruirse;
- no hay requisito de continuidad ante la caída de una zona;
- se crean snapshots solo cuando se necesite conservar un punto recuperable;
- se limpia el disco al terminar el laboratorio.

## 4. Persistent Disk regional

Un Persistent Disk regional replica cada escritura de forma síncrona entre dos zonas de una misma región.

### Lo que sí resuelve

- Mantiene datos del disco disponibles ante la pérdida de una de las dos zonas.
- Reduce el RPO frente a depender de snapshots periódicos.
- Facilita conectar o forzar la conexión del disco a una VM de la otra zona durante un failover controlado.

### Lo que no resuelve

- No crea automáticamente una VM secundaria.
- No cambia por sí mismo DNS, balanceadores ni tráfico.
- No detecta si la aplicación está sana.
- No protege contra un `DELETE`, corrupción lógica o ransomware replicado: la escritura errónea también se replica.
- No protege frente a una pérdida regional completa.
- No sustituye snapshots, backups coherentes con la aplicación ni pruebas de restauración.

### Regla de examen

Si el requisito dice **“survive a zonal failure with minimal data loss”**, piensa en un disco regional o en replicación de aplicación. Si dice **“recover data deleted yesterday”**, piensa en backup o snapshot. Son fallas diferentes.

### Costo y rendimiento

La réplica síncrona ocupa dos zonas. Google documenta que los discos regionales cuestan el doble por byte que sus equivalentes zonales, porque mantienen dos réplicas. Además, la escritura síncrona puede introducir una compensación de rendimiento. La disponibilidad de datos no equivale a disponibilidad completa del servicio.

## 5. Hyperdisk

Hyperdisk es la oferta de bloques duraderos recomendada por Google para las series de máquina compatibles más recientes. Su diferencia conceptual clave es que permite ajustar capacidad y rendimiento por separado dentro de límites definidos.

### Familias y señales

| Familia | Nombre CLI | Señal dominante |
|---|---|---|
| Hyperdisk Balanced | `hyperdisk-balanced` | Uso general, bases de datos y aplicaciones empresariales; capacidad, IOPS y throughput ajustables. |
| Hyperdisk Balanced High Availability | `hyperdisk-balanced-high-availability` | Réplica síncrona entre dos zonas con rendimiento aprovisionable y necesidad de RPO muy bajo. |
| Hyperdisk Extreme | `hyperdisk-extreme` | Máximo IOPS para bases de datos de gama alta. |
| Hyperdisk Throughput | `hyperdisk-throughput` | Mucha transferencia secuencial y capacidad; no prioriza IOPS elevados. |
| Hyperdisk ML | `hyperdisk-ml` | Lectura de muy alto throughput para entrenamiento o inferencia distribuida. |

Para la mayoría de las cargas compatibles, Google recomienda **Hyperdisk Balanced**. No obstante, la elección depende de la serie de máquina, la zona, los límites de tamaño/rendimiento y el costo.

### Capacidad y rendimiento no son lo mismo

Un Hyperdisk Balanced de 10 GiB puede configurarse, dentro de sus límites actuales, con hasta 5,000 IOPS. Con 5,000 IOPS, un throughput de 500 MiB/s se encuentra dentro de la relación permitida. Esto permite practicar la idea de “poco espacio, más rendimiento” sin crear un volumen grande.

La cifra aprovisionada es un **máximo**, no una promesa de que la aplicación siempre lo alcance. También limitan:

- la serie y el tamaño de la VM;
- el patrón de I/O y el tamaño de bloque;
- el sistema de archivos;
- el número de discos que comparten los límites de la VM;
- las cuotas del proyecto.

### No elijas Hyperdisk solo porque suene más rápido

No es la respuesta automática si:

- la serie de máquina no lo admite;
- la región o zona no ofrece el tipo requerido;
- la carga no justifica rendimiento aprovisionado adicional;
- un servicio administrado como Cloud SQL satisface mejor el requisito operacional;
- el requisito real es almacenamiento de objetos o archivos compartidos, no bloques.

## 6. Durabilidad, disponibilidad, backup y recuperación

| Concepto | Pregunta que responde | Ejemplo |
|---|---|---|
| Durabilidad | ¿El dato persiste más allá de la VM o un fallo de hardware? | Persistent Disk o Hyperdisk. |
| Disponibilidad zonal | ¿El dato puede seguir usándose si una zona falla? | Persistent Disk regional o Hyperdisk Balanced HA, más un plan de failover. |
| Backup/snapshot | ¿Puedo recuperar un punto anterior? | Snapshot programado o backup de aplicación. |
| Alta disponibilidad de la aplicación | ¿El servicio completo continúa atendiendo? | VM/instancia secundaria, health checks, failover y componentes regionales. |
| Recuperación regional | ¿Puedo operar si la región completa falla? | Replicación o backup en otra región y un runbook probado. |

Un error frecuente del examen es elegir un disco regional para corregir borrados accidentales. La réplica reproduce el borrado; hace falta una copia histórica independiente.

## 7. Decisión para las tres cargas de SiteOps Tracker

### Matriz de decisión

| Carga | Requisito dominante | Elección | Justificación | Alternativas descartadas |
|---|---|---|---|---|
| Importaciones de desarrollo | Costo bajo; recuperación mediante reconstrucción | `pd-balanced` zonal | Bloque duradero, sencillo y suficiente para uso general en una sola zona. | Regional duplica almacenamiento sin un RTO/RPO que lo exija; Hyperdisk añade capacidad operativa innecesaria; Local SSD perdería datos con el host. |
| PostgreSQL autogestionado de producción | Datos confirmados deben sobrevivir a una falla zonal | `pd-ssd` regional o, si la serie es compatible y se necesita rendimiento ajustable, Hyperdisk Balanced HA | Réplica síncrona en dos zonas. SSD favorece I/O aleatoria de base de datos. Requiere VM secundaria y runbook de failover. | Zonal no cubre la falla de zona; snapshots tienen RPO según horario; Cloud Storage no es volumen de bloques. En un diseño real se debe comparar primero Cloud SQL HA o AlloyDB. |
| Reindexación intensiva | 10 GiB de datos, 5,000 IOPS y 500 MiB/s sin comprar capacidad sobrante | `hyperdisk-balanced` zonal | Permite aprovisionar rendimiento y tamaño por separado y encaja con carga general exigente. | Aumentar un Persistent Disk solo para obtener rendimiento desperdicia capacidad; Hyperdisk Extreme sería excesivo; Hyperdisk Throughput prioriza transferencia sobre IOPS. |

### Regla de decisión de seis pasos

1. Clasifica el acceso: bloques, objetos o archivos compartidos.
2. Decide si los datos son temporales o duraderos.
3. Fija RPO y RTO frente a falla de VM, zona y región.
4. Describe el patrón: IOPS, throughput, latencia, lectura/escritura y tamaño.
5. Verifica compatibilidad de zona, serie de máquina, tamaño y cuotas.
6. Compara costo de capacidad, rendimiento, réplica y operación; añade backups cuando corresponda.

## 8. Práctica guiada con Google Cloud Console

> Usa un proyecto temporal de Skills si está disponible. Si usas una cuenta propia, revisa facturación y ejecuta la limpieza al terminar.

### 8.1 Comprobaciones previas

1. Abre **Google Cloud Console**.
2. Confirma el project ID visible en el selector superior.
3. Ve a **Billing** y confirma que entiendes qué cuenta pagará los recursos.
4. Ve a **APIs & Services > Enabled APIs & services** y confirma que **Compute Engine API** está habilitada.
5. Ve a **Compute Engine > Storage > Disks**.

### 8.2 Crea un Persistent Disk zonal

1. Pulsa **Create disk**.
2. En **Name**, escribe `sot-l20-zonal-pd`.
3. En **Location**, selecciona **Zonal**.
4. Elige una zona permitida por el laboratorio; anótala como `ZONE_A`.
5. En **Disk source type**, selecciona **Blank disk**.
6. En **Disk type**, selecciona **Balanced persistent disk**.
7. En **Size**, usa `10 GB`.
8. Mantén el cifrado predeterminado administrado por Google para este laboratorio.
9. Pulsa **Create**.

### 8.3 Inspecciona y registra evidencia

Abre el disco y registra:

- nombre;
- alcance zonal;
- zona;
- tipo `pd-balanced`;
- capacidad;
- estado;
- si está conectado a alguna VM;
- política de eliminación, si aparece en el flujo de conexión a una VM.

No lo conectes solo para completar una casilla. El objetivo de esta parte es reconocer el recurso y su alcance. Déjalo creado hasta completar la comparación por CLI y después elimínalo.

## 9. Práctica guiada con Google Cloud CLI

Los comandos siguientes se verificaron contra la documentación oficial de Google Cloud disponible el 16 de octubre de 2026.

### 9.1 Define variables seguras

Abre Cloud Shell y sustituye los valores de ejemplo por los de tu laboratorio.

```bash
export ACE_PROJECT_ID="TU_PROJECT_ID"
export ACE_REGION="us-central1"
export ACE_ZONE_A="us-central1-a"
export ACE_ZONE_B="us-central1-b"
export ACE_PD_ZONAL="sot-l20-zonal-pd"
export ACE_PD_REGIONAL="sot-l20-regional-pd"
export ACE_HD="sot-l20-hyperdisk"
```

Verifica que las dos zonas sean distintas y pertenezcan a la misma región. Si Google Skills asigna otra región, usa exclusivamente las ubicaciones autorizadas por el laboratorio.

### 9.2 Verifica identidad, proyecto y API

```bash
gcloud auth list --filter=status:ACTIVE
gcloud config get-value project
gcloud projects describe "$ACE_PROJECT_ID" \
  --format='table(projectId,name,lifecycleState)'
gcloud services list \
  --project="$ACE_PROJECT_ID" \
  --enabled \
  --filter='config.name:compute.googleapis.com' \
  --format='value(config.name)'
```

Si la API no aparece y tienes permiso para habilitarla:

```bash
gcloud services enable compute.googleapis.com \
  --project="$ACE_PROJECT_ID"
```

### 9.3 Verifica zonas y tipos disponibles

```bash
gcloud compute zones list \
  --project="$ACE_PROJECT_ID" \
  --filter="name~'^${ACE_REGION}-'" \
  --format='table(name,status,region.basename())'

gcloud compute disk-types list \
  --project="$ACE_PROJECT_ID" \
  --filter="zone:${ACE_ZONE_A}" \
  --format='table(name,zone.basename(),defaultDiskSizeGb)'
```

Confirma que `pd-balanced` exista. Crea Hyperdisk solo si `hyperdisk-balanced` aparece en la zona. Si no aparece, conserva la práctica conceptual; no cambies de ubicación fuera de las reglas del laboratorio.

### 9.4 Comprueba el disco zonal creado en consola

```bash
gcloud compute disks describe "$ACE_PD_ZONAL" \
  --project="$ACE_PROJECT_ID" \
  --zone="$ACE_ZONE_A" \
  --format='yaml(name,zone,type,sizeGb,status,users)'
```

Si la consola creó el disco en otra zona, actualiza `ACE_ZONE_A`. Después verifica que `ACE_ZONE_B` siga perteneciendo a la misma región y sea distinta.

### 9.5 Crea un Persistent Disk regional

```bash
gcloud compute disks create "$ACE_PD_REGIONAL" \
  --project="$ACE_PROJECT_ID" \
  --size=10GB \
  --type=pd-balanced \
  --region="$ACE_REGION" \
  --replica-zones="$ACE_ZONE_A,$ACE_ZONE_B"
```

La opción `--replica-zones` requiere exactamente dos zonas distintas dentro de la región indicada.

Inspecciona el resultado:

```bash
gcloud compute disks describe "$ACE_PD_REGIONAL" \
  --project="$ACE_PROJECT_ID" \
  --region="$ACE_REGION" \
  --format='yaml(name,region,replicaZones,type,sizeGb,status,users)'
```

Comprueba que `replicaZones` contenga ambas zonas. No interpretes esto como un backup: ambas réplicas mantienen el mismo estado actual.

### 9.6 Crea un Hyperdisk Balanced con rendimiento explícito

Ejecuta esta parte solo si `hyperdisk-balanced` está disponible en `ACE_ZONE_A` y tu cuota lo permite.

```bash
gcloud compute disks create "$ACE_HD" \
  --project="$ACE_PROJECT_ID" \
  --zone="$ACE_ZONE_A" \
  --type=hyperdisk-balanced \
  --size=10GB \
  --provisioned-iops=5000 \
  --provisioned-throughput=500
```

Inspección:

```bash
gcloud compute disks describe "$ACE_HD" \
  --project="$ACE_PROJECT_ID" \
  --zone="$ACE_ZONE_A" \
  --format='yaml(name,zone,type,sizeGb,provisionedIops,provisionedThroughput,status,users)'
```

Interpreta la evidencia:

- `sizeGb: 10` representa capacidad.
- `provisionedIops: 5000` representa el máximo de operaciones aprovisionado.
- `provisionedThroughput: 500` representa el máximo de throughput aprovisionado en MiB/s.
- `users: []` o un valor vacío indica que todavía no está conectado.

### 9.7 Conexión opcional a una VM compatible

No necesitas crear una VM para cumplir la práctica. Si el proyecto ya tiene una VM de laboratorio compatible, puedes estudiar los comandos sin ejecutarlos:

```bash
# Disco zonal: VM y disco deben estar en la misma zona.
gcloud compute instances attach-disk VM_NAME \
  --project="$ACE_PROJECT_ID" \
  --zone="$ACE_ZONE_A" \
  --disk="$ACE_PD_ZONAL" \
  --device-name="$ACE_PD_ZONAL"

# Disco regional: la VM debe estar en una de las zonas de réplica.
gcloud compute instances attach-disk VM_NAME \
  --project="$ACE_PROJECT_ID" \
  --zone="$ACE_ZONE_A" \
  --disk="$ACE_PD_REGIONAL" \
  --disk-scope=regional \
  --device-name="$ACE_PD_REGIONAL"
```

Un disco nuevo no se puede usar como sistema de archivos hasta formatearlo y montarlo. Formatear borra el contenido del dispositivo elegido; verifica siempre el nombre antes de ejecutar `mkfs`.

### 9.8 Evidencia del laboratorio

Completa esta tabla antes de limpiar:

| Carga | Recurso observado/creado | Scope | Tipo | Capacidad | IOPS/throughput | RPO/RTO razonado | Evidencia |
|---|---|---|---|---:|---|---|---|
| Desarrollo |  |  |  |  |  |  |  |
| PostgreSQL autogestionado |  |  |  |  |  |  |  |
| Reindexación |  |  |  |  |  |  |  |

Escribe además tres frases:

1. “Elegí un disco zonal para ___ porque ___.”
2. “Elegí un disco regional para ___ porque ___; todavía necesito ___ para que la aplicación sea altamente disponible.”
3. “Elegí Hyperdisk para ___ porque permite ___ sin ___.”

## 10. Resultado esperado

Al completar la ruta práctica deberías tener, antes de limpiar:

- un `pd-balanced` zonal de 10 GB creado desde la consola;
- un `pd-balanced` regional de 10 GB con dos zonas de réplica;
- si la zona lo admite, un `hyperdisk-balanced` zonal de 10 GB con 5,000 IOPS y 500 MiB/s;
- una matriz que justifique tres cargas distintas;
- evidencia de `describe` que distinga `zone`, `region`, `replicaZones`, capacidad y rendimiento aprovisionado;
- ninguna VM nueva requerida;
- todos los recursos del laboratorio eliminados al final.

No marques la práctica como concluida solo porque los comandos devolvieron `DONE`: explica por qué cada alcance y tipo satisface un requisito distinto.

## 11. Alternativa conceptual completa sin cuenta ni crédito

### 11.1 Inventario simulado

Supón que `disk-types list` devuelve:

| Zona | Tipos relevantes |
|---|---|
| `us-central1-a` | `pd-standard`, `pd-balanced`, `pd-ssd`, `hyperdisk-balanced` |
| `us-central1-b` | `pd-standard`, `pd-balanced`, `pd-ssd`, `hyperdisk-balanced` |

Supón también que la VM de producción puede utilizar Persistent Disk regional y que una serie moderna de VM admite Hyperdisk Balanced.

### 11.2 Tres tickets

**Ticket A - desarrollo**

- 50 GiB.
- Datos reconstruibles en cuatro horas.
- 1,500 IOPS son suficientes.
- No hay requisito de supervivencia zonal.

**Ticket B - base de producción**

- 500 GiB.
- Escrituras aleatorias y sensibles a latencia.
- RPO objetivo frente a falla zonal: cercano a cero.
- RTO objetivo: 15 minutos mediante un runbook de failover.
- Se requieren copias diarias para errores humanos.

**Ticket C - reindexación**

- 10 GiB.
- 5,000 IOPS y 500 MiB/s.
- Puede volver a empezar si la zona falla.
- Se ejecuta durante seis horas y después se elimina.

### 11.3 Solución razonada de la simulación

- **A:** `pd-balanced` zonal. La réplica regional no tiene requisito que la justifique.
- **B:** `pd-ssd` regional, más VM secundaria/runbook y snapshots o backup coherente con PostgreSQL. Si la serie y el presupuesto lo permiten, Hyperdisk Balanced HA es una alternativa para ajustar rendimiento.
- **C:** Hyperdisk Balanced zonal de 10 GiB con 5,000 IOPS y 500 MiB/s. El rendimiento se configura sin aumentar el disco solo para ganar IOPS.

### 11.4 Evidencia equivalente

Sin ejecutar Google Cloud, entrega:

1. La tabla de decisión completa.
2. Los tres comandos `gcloud compute disks create` que usarías.
3. Una explicación de por qué B necesita backup además de réplica.
4. Un cálculo cualitativo de costo: zonal < regional para el mismo tipo/capacidad; rendimiento de Hyperdisk por encima de la línea base puede sumar cargo.
5. Los comandos exactos de limpieza.

## 12. Solución de problemas

| Síntoma | Causa probable | Diagnóstico y corrección |
|---|---|---|
| `PERMISSION_DENIED` al crear | Falta `compute.disks.create` o el proyecto es incorrecto | Revisa cuenta activa y project ID; solicita el rol mínimo. No uses Owner como atajo. |
| `API not enabled` | Compute Engine API deshabilitada | Verifica `gcloud services list`; habilita `compute.googleapis.com` si tienes autorización. |
| `Invalid value for replicaZones` | Las zonas son iguales, pertenecen a regiones distintas o no coinciden con `--region` | Lista zonas y elige exactamente dos distintas dentro de la misma región. |
| El tipo no aparece | No está disponible en la zona o el proyecto | Ejecuta `gcloud compute disk-types list`; usa una ubicación permitida que sí lo admita o completa la alternativa conceptual. |
| IOPS/throughput inválidos | Combinación fuera de límites para tamaño o familia | Para el ejercicio usa 10 GiB, 5,000 IOPS y 500 MiB/s en Hyperdisk Balanced; vuelve a revisar límites actuales. |
| El disco no se conecta | Incompatibilidad de scope, zona o serie de máquina | Zonal exige misma zona; regional exige una zona de réplica y `--disk-scope=regional`; Hyperdisk exige serie compatible. |
| El disco no aparece como carpeta en Linux | Es nuevo, no tiene sistema de archivos o no está montado | Identifica `/dev/disk/by-id/google-*`, formatea solo el disco vacío correcto y monta por UUID. |
| El rendimiento real es menor al aprovisionado | La VM, patrón de I/O o configuración limita | Comprueba límites de máquina, tamaño de bloques, concurrencia y sistema de archivos. Aprovisionado significa máximo, no rendimiento garantizado. |
| No se puede eliminar | El disco sigue conectado | Desmonta dentro del SO, separa con `detach-disk`, verifica `users`, y luego elimina. |
| Tras una falla zonal el servicio no vuelve solo | El disco regional preserva datos, no toda la aplicación | Implementa VM secundaria, health checks, fencing y runbook de failover probado. |

## 13. Impacto en costos

### Qué genera cargos

- Capacidad aprovisionada de Persistent Disk o Hyperdisk desde la creación hasta la eliminación, esté conectado o no.
- Dos réplicas para un disco regional; Google documenta un costo por byte aproximadamente doble frente al equivalente zonal.
- IOPS y throughput aprovisionados de Hyperdisk por encima de la línea base incluida, según la familia.
- Snapshots y su almacenamiento incremental.
- VM, red y otros componentes si decides realizar la conexión opcional.

### Qué no debes suponer

- Detener una VM no deja de cobrar sus discos duraderos.
- Eliminar una VM no garantiza que se eliminen todos los discos de datos; depende de la política de eliminación.
- Un presupuesto envía alertas, pero no corta automáticamente el gasto.
- Un disco regional no elimina la necesidad de backups.

### Controles de costo para esta práctica

1. Usa tamaños mínimos compatibles.
2. No crees una VM nueva solo para observar metadatos de disco.
3. Crea los recursos, captura evidencia y elimina en la misma sesión.
4. Revisa la página **Disks** al final para detectar discos desasociados.
5. Consulta precios de tu región en la documentación oficial o la calculadora; no extrapoles una tarifa de otra ubicación.

## 14. Limpieza de recursos

> Eliminar discos borra los datos del laboratorio. Antes de ejecutar, confirma nombre, proyecto y que no contengan información necesaria.

### 14.1 Si conectaste un disco

Primero desmonta el sistema de archivos dentro de la VM:

```bash
sudo umount /dev/disk/by-id/google-DEVICE_NAME
```

Después separa el disco zonal:

```bash
gcloud compute instances detach-disk VM_NAME \
  --project="$ACE_PROJECT_ID" \
  --zone="$ACE_ZONE_A" \
  --disk="$ACE_PD_ZONAL"
```

Para un disco regional, añade `--disk-scope=regional`:

```bash
gcloud compute instances detach-disk VM_NAME \
  --project="$ACE_PROJECT_ID" \
  --zone="$ACE_ZONE_A" \
  --disk="$ACE_PD_REGIONAL" \
  --disk-scope=regional
```

### 14.2 Elimina los discos del laboratorio

```bash
gcloud compute disks delete "$ACE_PD_ZONAL" \
  --project="$ACE_PROJECT_ID" \
  --zone="$ACE_ZONE_A" \
  --quiet

gcloud compute disks delete "$ACE_PD_REGIONAL" \
  --project="$ACE_PROJECT_ID" \
  --region="$ACE_REGION" \
  --quiet

gcloud compute disks delete "$ACE_HD" \
  --project="$ACE_PROJECT_ID" \
  --zone="$ACE_ZONE_A" \
  --quiet
```

Si no creaste Hyperdisk, el último comando devolverá que no existe; no es necesario crearlo para poder eliminarlo.

### 14.3 Verificación final

```bash
gcloud compute disks list \
  --project="$ACE_PROJECT_ID" \
  --filter='name~^sot-l20-' \
  --format='table(name,location.basename(),type.basename(),sizeGb,users)'
```

Resultado esperado: ninguna fila. Revisa también **Compute Engine > Storage > Disks** y confirma que no queden discos `sot-l20-*`.

## 15. Glosario bilingüe

| English term | Término en español | Explicación breve |
|---|---|---|
| Block storage | Almacenamiento de bloques | Volumen que el SO trata como dispositivo de disco. |
| Durable storage | Almacenamiento duradero | Persiste más allá del ciclo de vida de una VM. |
| Zonal disk | Disco zonal | Recurso cuyo alcance es una sola zona. |
| Regional disk | Disco regional | Volumen replicado síncronamente entre dos zonas de una región. |
| Replica zone | Zona de réplica | Una de las dos zonas que conserva una copia sincronizada. |
| Synchronous replication | Replicación síncrona | Confirma la escritura después de escribir en ambas réplicas. |
| Persistent Disk | Persistent Disk | Familia de discos duraderos cuyo rendimiento suele escalar con capacidad. |
| Hyperdisk | Hyperdisk | Bloque duradero con rendimiento aprovisionable por separado. |
| IOPS | Operaciones de E/S por segundo | Cantidad de operaciones que puede procesar el volumen. |
| Throughput | Rendimiento de transferencia | Cantidad de datos transferidos por segundo. |
| Latency | Latencia | Tiempo de respuesta de una operación. |
| Provisioned performance | Rendimiento aprovisionado | Máximo configurado de IOPS o throughput. |
| Baseline performance | Rendimiento base | Rendimiento incluido antes de cargos adicionales, cuando aplica. |
| Snapshot | Instantánea | Copia puntual usada para restaurar un estado anterior. |
| Failover | Conmutación por error | Traslado de la carga a un recurso de respaldo. |
| Force attach | Conexión forzada | Conexión de un disco regional a otra VM cuando la separación normal no es posible durante una falla. |
| Recovery Point Objective (RPO) | Objetivo de punto de recuperación | Máxima antigüedad de datos que se acepta perder. |
| Recovery Time Objective (RTO) | Objetivo de tiempo de recuperación | Tiempo máximo aceptable para restaurar el servicio. |
| Device name | Nombre de dispositivo | Identificador estable usado para reconocer el disco dentro del SO. |
| Mount | Montar | Hacer accesible un sistema de archivos en una ruta. |
| Unattached disk | Disco desasociado | Disco que existe y se factura, aunque no esté conectado a una VM. |

## 16. Repaso activo espaciado

Responde sin mirar las soluciones ni lecciones anteriores. Después explica cada respuesta en voz alta.

### Día hábil anterior: lección 19 - VM, tipos de máquina, disponibilidad y SSH

1. ¿Qué dos políticas ayudan a una VM individual ante mantenimiento del host o terminación no iniciada por el usuario, pero no crean alta disponibilidad multizona?
2. ¿Por qué una máquina con más vCPU puede cambiar el rendimiento máximo observable de un disco?
3. ¿Qué diferencia de control de acceso existe entre OS Login y claves SSH guardadas en metadata?

### Hace 7 días: lección 15 - laboratorio integral de entorno

1. Ordena estas comprobaciones antes de crear recursos: proyecto, facturación, permisos, API, ubicación y costo.
2. ¿Por qué habilitar una API no equivale a desplegar un recurso?
3. ¿Qué evidencia mínima guardarías para demostrar creación, verificación y limpieza de un recurso?

### Hace 21 días: lección 5 - método ACE

1. Convierte este enunciado en requisitos: “Use the most cost-effective durable storage and tolerate a zonal outage with minimal data loss.”
2. ¿Qué palabra del enunciado descarta Local SSD? ¿Qué frase descarta un disco zonal?
3. Explica el método: requisito dominante -> scope -> servicio -> operación -> costo -> limpieza.

### Comprobación del repaso

- **Lección 19:** `onHostMaintenance=MIGRATE` y `automaticRestart=true` ayudan a una VM individual; no crean una réplica en otra zona. El rendimiento del disco también está limitado por la VM. OS Login centraliza acceso mediante IAM, mientras que las claves en metadata requieren gestión explícita y pueden heredarse según el alcance.
- **Lección 15:** primero confirma contexto y autoridad; habilitar una API abre el plano de control, pero no crea recursos. La evidencia debe mostrar identidad del recurso, configuración, resultado y ausencia tras la limpieza.
- **Lección 5:** “durable” elimina almacenamiento efímero; “tolerate a zonal outage” exige redundancia entre zonas o replicación de aplicación. El requisito determina la decisión, no el nombre más llamativo del producto.

Si no pudiste explicar dos o más respuestas sin mirar, anótalo como refuerzo. No reemplaza la lección de hoy.

## 17. Preguntas originales tipo examen

Todas las preguntas son originales y están basadas en escenarios. No son preguntas filtradas del examen.

### Question 1

SiteOps Tracker runs a development VM. Its imported test files can be recreated in four hours, and the team wants durable general-purpose block storage at the lowest reasonable operational complexity. A zonal outage does not need to be tolerated. What should the team choose?

A. Zonal Balanced Persistent Disk  
B. Regional SSD Persistent Disk  
C. Hyperdisk Balanced High Availability  
D. Cloud Storage mounted as the boot disk

### Question 2

A self-managed PostgreSQL database must preserve committed data if either of two zones in the same region fails. The application team will maintain a standby VM and a tested failover procedure. Which storage option best addresses the stated storage requirement?

A. Zonal Standard Persistent Disk with automatic restart  
B. Regional SSD Persistent Disk replicated across the two zones  
C. Local SSD on both VMs without application replication  
D. A daily snapshot of a zonal disk

### Question 3

A workload needs only 50 GiB of capacity but requires substantially more IOPS than a small Persistent Disk provides. The selected machine series supports Hyperdisk. What is the primary reason to choose Hyperdisk Balanced?

A. It stores objects through an HTTP API  
B. It automatically creates a standby VM  
C. Capacity and performance can be provisioned independently within supported limits  
D. It is always free when unattached

### Question 4 - Choose two

Which two statements about a regional Persistent Disk are correct?

A. It synchronously replicates disk data across two zones in one region.  
B. It automatically restores data deleted by an administrator yesterday.  
C. It still requires an application and VM failover design for service availability.  
D. It protects the workload from a complete regional outage without any other architecture.

### Question 5

An engineer creates an empty data disk and attaches it to a Linux VM. The disk appears in the Google Cloud console but no directory is available to the application. What should the engineer do next?

A. Enable the Cloud Storage API  
B. Format the correct empty device and mount its filesystem  
C. Convert the disk to a snapshot  
D. Increase the VM's boot disk size

### Question 6

The team must recover from accidental deletion of audit history that happened 18 hours ago. The current contents of both replicas of a regional disk already reflect the deletion. What should the team use?

A. Force-attach the secondary replica  
B. Automatic restart  
C. A suitable snapshot or application-consistent backup from before the deletion  
D. A larger regional disk

### Question 7

An engineer runs the following command:

```text
gcloud compute disks create findings-regional \
  --region=us-central1 \
  --replica-zones=us-central1-a,europe-west1-b \
  --type=pd-balanced \
  --size=100GB
```

Why should the command fail?

A. Regional disks require exactly three replica zones.  
B. Both replica zones must belong to the specified region.  
C. `pd-balanced` cannot be regional.  
D. Regional disks must be at least 1 TiB.

### Question 8

A disk is detached from every VM but remains listed in the project. Which statement is correct?

A. It stops incurring storage charges immediately because it is unattached.  
B. It is deleted automatically after 24 hours.  
C. Its provisioned capacity continues to incur charges until the disk is deleted.  
D. It becomes a free snapshot.

### Question 9 - Choose two

A team selected a Hyperdisk volume with 20,000 provisioned IOPS, but the benchmark reaches much less. Which two checks are most relevant?

A. Verify that the VM machine type supports the required disk performance.  
B. Verify the I/O pattern, block size, concurrency, and filesystem configuration.  
C. Add a second region to the VPC firewall rule.  
D. Grant Browser to the service account.

### Question 10

SiteOps Tracker stores user-uploaded evidence files that are accessed through the application by object name and do not need POSIX block-device semantics. Which option should be evaluated before adding another VM disk?

A. Cloud Storage  
B. Hyperdisk Extreme  
C. Local SSD  
D. Regional boot disk

## 18. Ficha de errores

Completa una fila por cada pregunta fallada o comando que no pudiste explicar.

| Campo | Registro |
|---|---|
| Fecha | 2026-10-16 |
| Pregunta o paso |  |
| Mi respuesta/acción |  |
| Respuesta/acción correcta |  |
| Requisito que ignoré |  |
| Tipo de error | Concepto / lectura / CLI / scope / costo / distracción |
| Por qué elegí el distractor |  |
| Regla de decisión corregida |  |
| Nueva explicación sin apuntes |  |
| Revisión programada | Próximo día hábil / 7 días / 21 días |

Refuerzos frecuentes para esta lección:

- **Réplica no es backup.** La réplica conserva el estado actual; el backup conserva un punto recuperable.
- **Disco regional no es aplicación regional.** Hace falta compute, detección y failover.
- **Rendimiento aprovisionado no es rendimiento garantizado.** La VM y el patrón de I/O también limitan.
- **Desasociado no significa gratuito.** El cargo termina al eliminar el recurso.

No se registraron lagunas específicas del alumno en el contexto disponible; usa la ficha para convertir cualquier fallo de hoy en refuerzo concreto.

## 19. Criterios de autoevaluación

Asigna 0, 1 o 2 puntos a cada criterio:

- **0:** no puedo hacerlo todavía.
- **1:** puedo hacerlo con notas o cometo una omisión relevante.
- **2:** puedo hacerlo sin notas y justifico la decisión.

| Criterio | 0 | 1 | 2 |
|---|:---:|:---:|:---:|
| Distingo bloques, objetos, archivos compartidos y almacenamiento efímero. |  |  |  |
| Explico la diferencia entre disco zonal y regional. |  |  |  |
| Distingo réplica síncrona de snapshot/backup. |  |  |  |
| Uso IOPS, throughput y latencia para describir una carga. |  |  |  |
| Explico por qué Hyperdisk desacopla capacidad y rendimiento. |  |  |  |
| Elijo almacenamiento para las tres cargas de SiteOps Tracker. |  |  |  |
| Interpreto `zone`, `region`, `replicaZones` y `users` en `describe`. |  |  |  |
| Identifico costos de capacidad, réplica y rendimiento. |  |  |  |
| Ejecuto o explico la limpieza sin dejar discos huérfanos. |  |  |  |
| Obtengo al menos 8/10 en las preguntas y justifico los distractores. |  |  |  |

**Interpretación personal, no oficial:**

- 17-20: preparación sólida para este tema; repásalo según el calendario.
- 13-16: repite las decisiones y el comando que causó duda.
- 0-12: vuelve a las secciones 3-7 y completa la alternativa conceptual antes de continuar.

Una puntuación alta no garantiza aprobar el examen; únicamente muestra evidencia de esta sesión.

## 20. Enlaces oficiales

- [Associate Cloud Engineer certification](https://cloud.google.com/learn/certification/cloud-engineer)
- [Choose a disk type](https://cloud.google.com/compute/docs/disks)
- [Persistent Disk overview](https://cloud.google.com/compute/docs/disks/persistent-disks)
- [Create a new Persistent Disk volume](https://cloud.google.com/compute/docs/disks/add-persistent-disk)
- [Create and manage regional disks](https://cloud.google.com/compute/docs/disks/regional-persistent-disk)
- [About synchronous disk replication](https://cloud.google.com/compute/docs/disks/about-regional-persistent-disk)
- [Build highly available services with regional disks](https://cloud.google.com/compute/docs/disks/high-availability-regional-persistent-disk)
- [Hyperdisk overview](https://cloud.google.com/compute/docs/disks/hyperdisks)
- [About Hyperdisk Balanced](https://cloud.google.com/compute/docs/disks/hd-types/hyperdisk-balanced)
- [Create a new Hyperdisk volume](https://cloud.google.com/compute/docs/disks/add-hyperdisk)
- [Persistent Disk performance](https://cloud.google.com/compute/docs/disks/performance)
- [Format and mount a non-boot disk on Linux](https://cloud.google.com/compute/docs/disks/format-mount-disk-linux)
- [`gcloud compute disks create`](https://cloud.google.com/sdk/gcloud/reference/compute/disks/create)
- [`gcloud compute disks describe`](https://cloud.google.com/sdk/gcloud/reference/compute/disks/describe)
- [`gcloud compute instances attach-disk`](https://cloud.google.com/sdk/gcloud/reference/compute/instances/attach-disk)
- [`gcloud compute instances detach-disk`](https://cloud.google.com/sdk/gcloud/reference/compute/instances/detach-disk)
- [`gcloud compute disks delete`](https://cloud.google.com/sdk/gcloud/reference/compute/disks/delete)
- [Disk and image pricing](https://cloud.google.com/compute/disks-image-pricing)
- [Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator)

## 21. Soluciones justificadas

### Answer 1: A

**A is correct.** A zonal Balanced Persistent Disk is durable, general purpose, and avoids paying for cross-zonal replication that the requirement does not need.

- **B** adds regional replication and SSD performance without a stated need.
- **C** adds high availability and provisioned-performance capabilities that are unnecessary here.
- **D** is invalid because Cloud Storage is object storage, not a VM boot block device.

### Answer 2: B

**B is correct.** Regional SSD Persistent Disk synchronously replicates across two zones and fits random database I/O. The standby VM and runbook address compute/application failover.

- **A** remains zonal; automatic restart does not create cross-zonal disk availability.
- **C** relies on ephemeral local storage and provides no valid replication.
- **D** has an RPO determined by snapshot timing and does not preserve every committed write.

### Answer 3: C

**C is correct.** Hyperdisk lets the team configure capacity and performance independently within supported ranges.

- **A** describes object storage such as Cloud Storage.
- **B** confuses storage with compute failover orchestration.
- **D** is false; disk capacity can be billed while unattached, and provisioned performance can add cost.

### Answer 4: A and C

**A and C are correct.** A regional disk synchronously maintains data in two zones, but application availability still needs compute, health checks and failover behavior.

- **B** is false because a deletion is replicated; recovery of old data needs a snapshot or backup.
- **D** is false because both replicas remain in one region.

### Answer 5: B

**B is correct.** A blank disk is only a block device. Linux needs a filesystem and mount point before the application can use it.

- **A** is unrelated.
- **C** creates a recovery artifact but does not make the blank disk usable.
- **D** changes a different volume.

### Answer 6: C

**C is correct.** The required state is historical. Restore from a snapshot or application-consistent backup created before the deletion.

- **A** exposes the same replicated current state.
- **B** restarts compute and cannot reverse data changes.
- **D** changes capacity, not history.

### Answer 7: B

**B is correct.** Both replica zones must be distinct zones in the region supplied by `--region`; `europe-west1-b` is not in `us-central1`.

- **A** is false: exactly two replica zones are used.
- **C** is false: `pd-balanced` supports regional disks.
- **D** is false: regional Balanced Persistent Disk supports much smaller sizes than 1 TiB.

### Answer 8: C

**C is correct.** Durable disk capacity is billed from creation until deletion, including when detached.

- **A** confuses compute attachment with the storage resource lifecycle.
- **B** invents an automatic deletion period.
- **D** confuses a disk with a separate snapshot resource.

### Answer 9: A and B

**A and B are correct.** The VM's machine-type limits and the actual I/O pattern can prevent a disk from reaching its provisioned maximum.

- **C** changes networking and does not raise block-storage performance.
- **D** changes an IAM viewer role and is unrelated to data-plane I/O.

### Answer 10: A

**A is correct.** Cloud Storage matches object-by-name access and avoids treating evidence files as a VM block volume.

- **B** is expensive high-IOPS block storage and does not match object semantics.
- **C** is ephemeral local block storage.
- **D** is intended for boot/block use and cross-zonal data availability, not an object repository.
