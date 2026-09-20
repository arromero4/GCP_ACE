# Plan integral de preparación ACE | 21 septiembre–31 diciembre 2026

**Objetivo:** reconstruir desde cero las destrezas del examen estándar Associate Cloud Engineer (ACE) tras un intento fallido, con práctica real, revisión de errores y dos simulacros antes de escoger fecha de examen. Aprobar no se puede garantizar; el plan busca una preparación comprobable.

**Fuente principal:** `associate_cloud_engineer_exam_guide_english.pdf` (guía oficial adjunta de 5 páginas). La guía agrupa los temas en **1. Entorno (~20 %), 2. Planificación e implementación (~30 %), 3. Operación (~30 %), 4. Acceso y seguridad (~20 %)**. La explicación diaria será en español desde cero, conservará los nombres de producto y conceptos en inglés y hará preguntas tipo examen en inglés. Los porcentajes son aproximados, no una fórmula para calcular la nota.

**Calendario:** lunes a viernes, 15:00, `America/Mexico_City`, del 21-09 al 31-12-2026, inclusive: **74 lecciones**. Descanso los sábados y domingos. Los festivos entre semana siguen en el calendario; podrán emplearse como día ligero de repaso si hace falta. **Hacia las 17:00** habrá un segundo recordatorio para la actividad pendiente de Google Skills. Duración sugerida: 60–90 minutos para la lección y la práctica; otros 60–90 minutos para Skills, con flexibilidad para laboratorios largos.

## Cómo se trabaja cada día

1. Leer **una lección nueva** y realizar su práctica en **un único archivo `.md` descargable**, titulado `ACE_2026_Lxx_Tema.md`. La lección debe contener objetivo, prerrequisitos, explicación gradual con analogía y ejemplo, glosario español/inglés, comparación de decisiones, pasos de consola/CLI verificables, resultado esperado, errores habituales, limpieza de recursos y referencias actuales a Google Cloud.
2. Resolver 8–12 preguntas originales en inglés, de escenarios y selección simple o múltiple. Incluir clave separada, justificación de la correcta y por qué fallan los distractores; no presentar preguntas filtradas del examen. En las fechas de control y simulacro usar los números indicados en la tabla.
3. Anotar en el mismo Markdown el resultado del laboratorio (o una alternativa de simulación si no hay crédito/cuenta), la puntuación, las dudas y una ficha de recuerdo activo. En el siguiente día hábil recuperar el tema anterior; a los 7 y 21 días recuperar al menos tres decisiones del tema (si coincide con fin de semana, mover al próximo día hábil), sin crear una segunda lección ese día.
4. Estudiar la siguiente actividad pendiente en el path de Skills indicado en el día. Guardar nombre de actividad, estado, evidencia y tema del examen que cubre. Si la actividad requiere más tiempo, continuarla al siguiente recordatorio, sin saltar el orden de la ruta. Si el path ya se completó, dedicar ese bloque a laboratorios del ACE o a los errores de ese dominio.
5. Para laboratorios, preferir proyectos temporales de Google Skills; si se utiliza una cuenta propia, verificar facturación, cuotas y recursos antes y después. Un presupuesto **alerta, no detiene automáticamente** el gasto. No incluir claves privadas ni secretos en los Markdown.

**Indicadores personales de avance:** mínimo 80 % en controles por dominio y 85 % en dos simulacros originales de 55 preguntas/120 minutos, sin un dominio por debajo de 75 %. Son metas de estudio propias, **no** umbrales oficiales publicados por Google. Si un control queda por debajo, usar el siguiente bloque de Skills y la fecha de corrección para reforzar la debilidad; ajustar la fecha del examen según los resultados, sin adivinarla ahora.

## Paths paralelos de Google Skills

| Fechas del plan | Path oficial | Carga orientativa indicada por ti | Función en la preparación |
|---|---|---:|---|
| Días 1–10 · 21 sep–2 oct | [Getting Started with Google Cloud](https://www.skills.google/paths/8) | 15 h 46 min | Primer contacto con consola, CLI, proyectos y laboratorios. |
| Días 11–18 · 5–14 oct | [Cloud Digital Leader Certification](https://www.skills.google/paths/9) | 8 h 25 min | Contexto de servicios y decisiones de negocio. Es base complementaria; no sustituye ACE. |
| Días 19–74 · 15 oct–31 dic | [Associate Cloud Engineer Certification](https://www.skills.google/paths/11) | 72 h 7 min | Prácticas específicas de implementación, operación y seguridad. |

La ruta de Skills puede cambiar sus actividades o duración. El recordatorio debe enlazar al path y solicitar **la siguiente actividad no completada** sin inventar nombres de módulos. Si un path termina antes, avanzar al siguiente; si se retrasa, conservar lo pendiente y usar los repasos del plan para alcanzar la cobertura ACE.

## Calendario diario y evidencia concreta

| # | Fecha | Guía | Lección | Práctica o evidencia |
|---:|---|---|---|---|
| 1 | 21/09/2026 | Base, 1.1 | Qué es la nube, proyectos y servicios de Google Cloud | Dibuja cómo se relacionan organización, carpeta, proyecto y recurso; realiza un diagnóstico de 10 preguntas. |
| 2 | 22/09/2026 | Base, 1.1 | Consola, Cloud Shell, gcloud y seguridad de laboratorio | Navega un proyecto de laboratorio; ejecuta comandos de consulta de proyecto, cuenta y configuración. |
| 3 | 23/09/2026 | Base, 1.2 | Costos, facturación y presupuesto sin sorpresas | Ubica la cuenta de facturación y prepara una lista de controles de costo y limpieza. |
| 4 | 24/09/2026 | Base, 1.1/2.3 | Regiones, zonas, red y disponibilidad | Compara dos arquitecturas de una y varias zonas y explica una decisión de ubicación. |
| 5 | 25/09/2026 | Base, todas | Método ACE: CLI, escenarios y vocabulario técnico en inglés | Resuelve 15 preguntas diagnósticas y crea registro de errores por dominio. |
| 6 | 28/09/2026 | 1.1 | Jerarquía: organizaciones, carpetas, proyectos y recursos | Diseña una jerarquía para Bitácora de Red con desarrollo y producción. |
| 7 | 29/09/2026 | 1.1 | Políticas de organización, restricciones y herencia | Decide en qué nivel aplicar tres restricciones y observa la diferencia frente a IAM. |
| 8 | 30/09/2026 | 1.1/4.1 | IAM inicial, miembros, Cloud Identity, usuarios y grupos | Crea una matriz de usuarios, grupos y permisos mínimos para tres equipos. |
| 9 | 01/10/2026 | 1.1 | APIs, cuotas y aumentos de cuotas | Habilita una API en un laboratorio y localiza una cuota y su flujo de aumento. |
| 10 | 02/10/2026 | 1.2 | Cuentas de facturación y vínculo con proyectos | Esquematiza un proyecto vinculado a una cuenta y explica quién puede administrarlos. |
| 11 | 05/10/2026 | 1.2 | Presupuestos, alertas y exportación de facturación | Configura o simula un presupuesto y una exportación a BigQuery; interpreta un reporte. |
| 12 | 06/10/2026 | 1.1 | Red inicial, regiones, zonas y disponibilidad de productos | Compara ubicaciones y configura o inspecciona una VPC y subredes. |
| 13 | 07/10/2026 | 1.1 | Google Cloud Observability, Cloud Asset Inventory y Gemini Cloud Assist | Encuentra recursos en inventario y plantea una consulta de análisis y observabilidad. |
| 14 | 08/10/2026 | 1.1 | Organizaciones independientes y Workforce Identity Federation | Resuelve dos casos: organización nueva e identidad externa de una persona. |
| 15 | 09/10/2026 | 1.1/1.2 | Laboratorio integral de entorno: proyecto, API, permisos, costos y red | Documenta pasos, resultados y limpieza de un entorno de laboratorio. |
| 16 | 12/10/2026 | 1.1/1.2 | Repaso guiado del dominio 1: decisiones y errores comunes | Resuelve 20 escenarios; justifica y corrige cada respuesta. |
| 17 | 13/10/2026 | 1.1/1.2 | Control de dominio 1 y práctica de recuperación | Completa una evaluación de 25 preguntas y vuelve a ejecutar dos tareas débiles. |
| 18 | 14/10/2026 | 2.1 | Elegir Compute Engine, GKE, Cloud Run, functions o Agent Runtime | Construye una tabla de decisión según operación, eventos y control requerido. |
| 19 | 15/10/2026 | 2.1 | VM: tipos de máquina, disponibilidad y claves SSH | Crea o describe una VM y verifica metadatos, acceso y política de disponibilidad. |
| 20 | 16/10/2026 | 2.1 | Discos de VM: Persistent Disk zonal/regional y Hyperdisk | Selecciona almacenamiento para tres cargas con rendimiento y recuperación distintos. |
| 21 | 19/10/2026 | 2.1 | Plantillas, managed instance groups, autoscaling y health checks | Dibuja y configura o simula una plantilla y MIG con comprobación de estado. |
| 22 | 20/10/2026 | 2.1 | OS Login, VM Manager, Spot VMs y máquinas personalizadas | Compara acceso, gestión de SO, interrupciones y costo en cuatro escenarios. |
| 23 | 21/10/2026 | 2.1 | Kubernetes básico: kubectl, GKE Autopilot, regional y privado | Consulta un clúster de laboratorio y compara sus tres modos de implementación. |
| 24 | 22/10/2026 | 2.1/3.1 | Contenedores, Artifact Registry y despliegue en GKE | Publica o identifica una imagen y despliega una app de prueba con kubectl. |
| 25 | 23/10/2026 | 2.1 | Cloud Run y Cloud Run functions: despliegue sin servidores | Despliega o simula un servicio y una función; explica escalado e invocación. |
| 26 | 26/10/2026 | 2.1 | Eventos con Pub/Sub, cambios de Cloud Storage y Eventarc | Traza un evento de origen a destino y selecciona el disparador apropiado. |
| 27 | 27/10/2026 | 2.1/3.1 | GPU, TPU y Agent Runtime en Gemini Enterprise Agent Platform | Escoge acelerador o servicio administrado para tres casos y explica límites. |
| 28 | 28/10/2026 | 2.2 | Cloud Storage: buckets y clases Standard, Nearline, Coldline, Archive | Elige clase de almacenamiento para cuatro patrones de acceso. |
| 29 | 29/10/2026 | 2.2 | Archivos: Filestore, NetApp Volumes y Managed Lustre | Selecciona servicio de archivos según protocolo, escala y carga. |
| 30 | 30/10/2026 | 2.2 | Bases relacionales: Cloud SQL, AlloyDB y Spanner | Elige base según compatibilidad SQL, escala, consistencia y operación. |
| 31 | 02/11/2026 | 2.2 | Análisis: BigQuery y Dataflow | Diseña carga analítica y flujo de procesamiento por lotes o en streaming. |
| 32 | 03/11/2026 | 2.2 | Firestore, Bigtable y Memorystore | Compara documentos, tablas de gran escala y caché en tres escenarios. |
| 33 | 04/11/2026 | 2.2 | Pub/Sub, Managed Service for Apache Kafka, carga y redundancia | Selecciona ingesta por CLI, Cloud Storage o Storage Transfer Service y diseña redundancia. |
| 34 | 05/11/2026 | 2.3 | VPC custom mode, subredes, Shared VPC y VPC Peering | Diseña red de dos proyectos y distingue compartir de conectar VPC. |
| 35 | 06/11/2026 | 2.3 | Firewall VPC, Cloud NGFW, secure Tags y service accounts | Decide reglas ingress/egress con acción, origen, destino, destino lógico y puertos. |
| 36 | 09/11/2026 | 2.3 | VPN, Interconnect, balanceadores y Network Service Tiers | Selecciona conexión híbrida, tipo de balanceo y tier en tres casos. |
| 37 | 10/11/2026 | 2.4 | Infraestructura como código e IA asistida | Compara Fabric FAST, Config Connector, Terraform y Helm; evalúa Gemini CLI, Antigravity, Cloud Assist y Application Design Center. |
| 38 | 11/11/2026 | 3.1 | Inventario de VM y acceso remoto con seguridad | Lista VM activas y explica diagnóstico paso a paso de acceso SSH fallido. |
| 39 | 12/11/2026 | 3.1 | Imágenes, snapshots y políticas de snapshots | Crea o simula snapshot, restaura y compara imagen frente a copia de disco. |
| 40 | 13/11/2026 | 3.1 | Inventario GKE y gestión de node pools | Consulta nodos, Pods y Services y diseña un cambio de pool con autoscaling. |
| 41 | 16/11/2026 | 3.1 | Pods, Services, StatefulSets, HPA y VPA | Interpreta manifiestos y ajusta horizontal y verticalmente una app. |
| 42 | 17/11/2026 | 3.1 | Autopilot Pod requests y acceso a Artifact Registry | Corrige un fallo de permisos de imagen y define solicitudes de recursos. |
| 43 | 18/11/2026 | 3.1 | Revisiones y autoscaling de Cloud Run; reparto de tráfico | Despliega revisión y calcula split para Cloud Run, functions y GKE. |
| 44 | 19/11/2026 | 3.1 | GPU/TPU, Agent Runtime, notebooks y Cloud Workstations | Elige dónde gestionar un agente, notebooks de Workbench/BigQuery y entorno de desarrollo. |
| 45 | 20/11/2026 | 3.2 | Objetos de Cloud Storage: permisos, protección y lifecycle | Define acceso mínimo y regla de transición/eliminación para un bucket. |
| 46 | 23/11/2026 | 3.2 | Consultas en bases de datos y estimación de costos | Ejecuta o interpreta consultas de Cloud SQL, BigQuery, Bigtable, Spanner, Firestore y AlloyDB; estima almacenamiento. |
| 47 | 24/11/2026 | 3.2 | Backups/restores y Database Center | Diseña recuperación de Cloud SQL, Firestore, Spanner, AlloyDB y Bigtable; inspecciona flota. |
| 48 | 25/11/2026 | 3.2 | Jobs de Dataflow/BigQuery y CMEK | Revisa un trabajo fallido y ubica cuándo configurar una clave administrada por cliente. |
| 49 | 26/11/2026 | 3.3 | Expandir rango IPv4 de subred, IP estática y rutas | Calcula capacidad de subred y propone IP interna/externa y ruta personalizada. |
| 50 | 27/11/2026 | 3.3 | Cloud DNS y Cloud NAT | Resuelve salida privada y nombre DNS para una aplicación. |
| 51 | 30/11/2026 | 3.3 | Operar y diagnosticar firewall VPC y Cloud NGFW | Identifica regla efectiva en dos fallos de conectividad ingress/egress. |
| 52 | 01/12/2026 | 3.4 | Cloud Monitoring: métricas estándar, personalizadas y alertas | Crea o simula alerta, métrica de aplicación y métrica basada en logs. |
| 53 | 02/12/2026 | 3.4 | Audit Logs, VPC Flow Logs, firewall logs, buckets y routers | Filtra evento en Logging, inspecciona detalle y diseña exportación a BigQuery/on-prem. |
| 54 | 03/12/2026 | 3.4 | Trace, Profiler, Query Insights e index advisor | Elige herramienta y causa probable para latencia y consulta lenta. |
| 55 | 04/12/2026 | 3.4 | Ops Agent, Prometheus, Service Health, Active Assist y Cloud Hub | Relaciona una señal con herramienta; usa Gemini Cloud Assist como ayuda de diagnóstico. |
| 56 | 07/12/2026 | 3.1/3.2/3.3/3.4 | Simulación operativa: aplicación lenta, VM inaccesible y datos en riesgo | Documenta hipótesis, consultas, corrección y 25 preguntas del dominio 3. |
| 57 | 08/12/2026 | 4.1 | IAM policies y roles básicos, predefinidos y personalizados | Lee una política y asigna permisos mínimos a tres personas. |
| 58 | 09/12/2026 | 4.1 | Herencia IAM y creación de roles personalizados | Calcula permisos efectivos desde organización a proyecto y diseña un rol. |
| 59 | 10/12/2026 | 4.1/1.1 | Cloud Identity, grupos y Workforce Identity Federation | Compara identidades humanas externas, grupos y alcance de políticas. |
| 60 | 11/12/2026 | 4.2 | Service accounts: creación, cuentas administradas y permisos mínimos | Asigna una cuenta a VM/servicio y diferencia permisos del recurso y de la cuenta. |
| 61 | 14/12/2026 | 4.2 | Impersonation y credenciales de corta duración | Traza quién suplanta a quién y selecciona token temporal en un escenario. |
| 62 | 15/12/2026 | 4.2 | Workload Identity Federation y cuentas para GKE | Conecta una carga externa o GKE a un servicio con permisos mínimos. |
| 63 | 16/12/2026 | 4.1/4.2 | Seguridad integrada: IAM, organización y secure Tags | Corrige una configuración insegura usando política, rol y regla de red. |
| 64 | 17/12/2026 | 3.2/4.1 | CMEK y controles de datos | Selecciona configuración de cifrado y aclara la división de responsabilidades. |
| 65 | 18/12/2026 | 4.1/4.2 | Laboratorio de IAM de principio a fin | Crea o simula roles, cuentas, vínculo a recurso e impersonation; revoca accesos. |
| 66 | 21/12/2026 | 4.1/4.2 | Casos de seguridad: humanos frente a workloads | Resuelve cinco casos y explica por qué los distractores son incorrectos. |
| 67 | 22/12/2026 | 4.1/4.2 | Evaluación de seguridad en inglés | Resuelve 25 preguntas de acceso/seguridad y revisa fallos por concepto. |
| 68 | 23/12/2026 | 1-4 | Diseño integral de Bitácora de Red en Google Cloud | Elige red, compute, datos, observabilidad, IAM y costos con justificación. |
| 69 | 24/12/2026 | 1-4 | Repaso espaciado de servicios parecidos | Compara 20 pares confundibles y rehace 10 preguntas falladas. |
| 70 | 25/12/2026 | 1-4 | Simulacro A: 55 preguntas originales en 120 minutos | Registra aciertos, tiempo y errores por 20/30/30/20. |
| 71 | 28/12/2026 | 1-4 | Corrección profunda del simulacro A | Explica cada error, repite un laboratorio débil y elabora fichas de decisión. |
| 72 | 29/12/2026 | 1-4 | Simulacro B: 55 preguntas originales en 120 minutos | Mide mejora por dominio y tipo de error sin reutilizar preguntas. |
| 73 | 30/12/2026 | 1-4 | Refuerzo dirigido al dominio más débil | Rehace práctica y 25 preguntas inéditas del dominio de menor rendimiento. |
| 74 | 31/12/2026 | 1-4 | Simulacro final y decisión de preparación | Resuelve 55 preguntas, contrasta metas de dominio y fija siguiente repaso/examen. |

## Mapa de cobertura de la guía adjunta

Este mapa permite comprobar que **cada apartado y grupo de productos** de la guía tiene una lección asignada. Los días de integración y simulacros vuelven a cubrir los cuatro dominios.

| Apartado de la guía | Cobertura prevista | Puntos de control |
|---|---|---|
| 1.1 Proyectos y cuentas | 1–2, 4, 6–9, 12–15, 59 | Jerarquía y políticas; IAM/Cloud Identity; APIs/cuotas; organización independiente; red/geografía; Observability, Asset Inventory y Cloud Assist; Workforce Identity Federation. |
| 1.2 Facturación | 3, 10–11, 15–17 | Cuentas, vínculo con proyecto, presupuestos/alertas y exportación. |
| 2.1 Compute | 18–27 | Decisiones VM/GKE/Cloud Run/functions/Agent Runtime; discos, SSH, MIG, OS Login, VM Manager, Spot/custom; kubectl, GKE, eventos, GPU/TPU. |
| 2.2 Almacenamiento y datos | 28–33 | Cloud Storage/clases, Filestore, NetApp Volumes, Managed Lustre; Cloud SQL, BigQuery, Firestore, Spanner, Bigtable, AlloyDB, Dataflow, Pub/Sub, Managed Kafka, Memorystore; carga/transferencia y redundancia. |
| 2.3 Red | 34–36 | VPC custom, Shared VPC y Peering; firewalls/NGFW, ingress/egress, secure Tags, service accounts; VPN, Interconnect, balanceadores y Network Service Tiers. |
| 2.4 Herramientas | 37 | Fabric FAST, Config Connector, Terraform, Helm; Gemini CLI, Antigravity, Cloud Assist, Application Design Center. |
| 3.1 Operación de compute | 38–44 | Acceso e inventario VM; snapshots/imágenes; inventario y pools GKE; Pods/Services/StatefulSets, HPA/VPA, Autopilot/Artifact Registry; revisiones/tráfico/escalado; aceleradores, agentes, notebooks y Workstations. |
| 3.2 Operación de datos | 45–48, 64 | Objetos/lifecycle; consultas y costos; backups/restores de motores enumerados; trabajos Dataflow/BigQuery, Database Center y CMEK. |
| 3.3 Operación de red | 49–51 | Subred IPv4, IP estática, rutas, Cloud DNS/NAT y mantenimiento de firewalls/NGFW. |
| 3.4 Monitoreo y logging | 52–56 | Métricas/alertas; VPC Flow Logs/audit/firewall logs, buckets/router/analytics/exportación; Trace/Profiler/Query Insights/index advisor; Personalized Service Health, Ops Agent, Prometheus, Active Assist, Cloud Hub y Gemini Cloud Assist. |
| 4.1 IAM | 8, 57–59, 63–69 | Políticas, tipos de roles, herencia y roles personalizados. |
| 4.2 Service accounts | 60–62, 65–69 | Cuentas Google-managed, permisos mínimos, asignación a recursos, IAM de cuentas, impersonation, credenciales temporales y Workload Identity Federation/GKE. |

## Cierre y preparación del examen

- Mantener un `registro_de_errores` dentro de cada Markdown: tema, razón del fallo, regla de decisión y nueva explicación sin mirar apuntes.
- En los simulacros, usar 55 preguntas inéditas y 120 minutos, con distribución orientativa 11/16/17/11 por dominios 1/2/3/4. Evitar memorizar respuestas; justificar el requisito del escenario que determina cada producto.
- Releer la guía adjunta al terminar los días 17, 37, 56, 69 y 74. Confirmar nombres y comportamiento de productos con documentación oficial vigente antes de cada lección.
- Google indica que el examen estándar dura 2 horas, tiene 50–60 preguntas y no exige prerrequisitos formales; recomienda 6+ meses de experiencia práctica. En diciembre, escoger fecha solo con base en resultados y disponibilidad personal: [página oficial de ACE](https://cloud.google.com/learn/certification/cloud-engineer).

**Material de referencia:** guía oficial adjunta `associate_cloud_engineer_exam_guide_english.pdf`; [página de certificación ACE](https://cloud.google.com/learn/certification/cloud-engineer); los tres paths enlazados en la tabla.
