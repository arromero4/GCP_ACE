# Inglés para Software Engineer — Sesión 18: comparar snapshots tipados y comunicar cambios

> **Etapa actual de la ruta:** TypeScript profundo.  
> **Lección técnica de referencia:** **Lección 2 — Arrays y tuplas** del archivo técnico `Ruta_Full_Stack_TypeScript_Lecciones_01_a_06.md`.  
> **Sesión de inglés:** **18**. El número de sesión de inglés no es el número de lección técnica.  
> **Tema:** comparar dos snapshots representados como arrays readonly de tuples, detectar cambios de estado sin mutar las entradas y explicar supuestos, complejidad y límites de la evidencia.  
> **Objetivo profesional:** comunicar de forma precisa cómo se comparan estados anteriores y actuales durante una entrevista, un code review, una investigación técnica o una actualización de incidente.  
> **Meta estimada de práctica oral:** **15–20 minutos**, ajustables.  
> **Estado:** sesión preparada para practicar. No hay una respuesta posterior registrada que demuestre práctica o dominio de la sesión 17. Por eso esta sesión ofrece una actividad nueva dentro de **Arrays y tuplas** y no avanza todavía a objetos.

---

## Cómo usar esta sesión

1. Lee el repaso, el vocabulario y la explicación técnica.
2. Para una práctica de 15–20 minutos realiza `R1`, `P1`, `C1`, `I1` y una actividad entre `D1`, `CR1` o `B1`.
3. Antes de hablar, escribe solo entre tres y siete palabras clave.
4. Haz una primera toma con la estructura de apoyo y una segunda con menos notas.
5. Adapta los modelos a tu forma de hablar; no los memorices palabra por palabra.
6. Envía aquí un audio o texto con la etiqueta del ejercicio, por ejemplo: `Sesión 18 — C1`.
7. En el chat se corregirán gramática, vocabulario, ortografía, fluidez y precisión técnica. La pronunciación solo se evaluará si existe audio realmente accesible.
8. La existencia de este archivo no significa que la sesión esté completada ni que la Lección técnica 2 esté dominada.

**Ruta oral sugerida:** `R1` 1 minuto; `P1` 1 minuto; `L1` 2 minutos; `C1` 2–3 minutos; `I1` 2 minutos; una actividad profesional 2–3 minutos; segunda toma 3–5 minutos.

---

## 1. Continuidad de la ruta

Las sesiones 6–17 han trabajado la **Lección técnica 2 — Arrays y tuplas** desde varios ángulos:

- arrays frente a tuples;
- readonly collections;
- transformaciones y búsquedas;
- `filter`, `find`, `map`, `reduce`, `some`, `every` e `includes`;
- recorridos con `for...of`;
- casos límite;
- selección y priorización sin mutación;
- explicación de complejidad y alcance de los datos.

La sesión 17 preguntaba cuál era el mejor candidato dentro de una sola colección. Esta sesión plantea un problema distinto:

> How can we compare a previous snapshot with a current snapshot and report only the status changes?

El contexto es **SiteOps Tracker**, un proyecto ficticio de portafolio para auditar infraestructura en sedes, registrar activos, hallazgos, estados, responsables e historial. No representa un sistema privado ni contiene clientes, logros o métricas inventadas.

La práctica combina:

- **TypeScript:** arrays readonly, tuples readonly, desestructuración, `find`, `push`, unions y tipos de retorno;
- **algoritmos:** recorrido exterior, búsqueda interior, complejidad `O(n × m)` y espacio proporcional al número de cambios;
- **inglés profesional:** describir estado anterior, estado actual, transición, ausencia, supuestos y evidencia limitada.

---

## 2. Repaso breve del lenguaje anterior

| Intención | Expresión reutilizable |
| --- | --- |
| Describir una colección | `The array represents a variable-length collection of records.` |
| Describir una tuple | `Each tuple has four fixed positions.` |
| Hablar de no mutación | `The function reads both inputs without modifying them.` |
| Explicar una búsqueda | `The inner search looks for a record with the same identifier.` |
| Hablar de ausencia | `Undefined means that no matching record was found.` |
| Limitar una conclusión | `The result is based only on the two snapshots provided.` |
| Explicar complejidad | `The nested search may compare each current record with many previous records.` |
| Distinguir requisito | `Whether missing records should be reported is a product decision.` |

### R1 — Recuperación activa (45–60 segundos)

**Qué debes intentar decir:** explica la diferencia entre un array y una tuple, define qué significa `readonly` para los inputs y describe qué representa `undefined` al usar `find`.

- **Estructura reutilizable:** `An array represents …, while a tuple represents … . Readonly indicates that … . When find returns undefined, it means … .`
- **Palabras y frases útiles:** `variable-length collection`, `fixed positional structure`, `read-only input`, `matching record`, `return undefined`.
- **Conectores:** `while`, `in this case`, `when`, `however`.
- **Patrones de oración:** `X represents …`; `X indicates that …`; `When X happens, it means …`.
- **Vocabulario técnico:** `array`, `tuple`, `readonly`, `find`, `return value`.
- **Por qué funcionan:** recuperan los conceptos mínimos necesarios para explicar la comparación sin saltar a una etapa posterior de la ruta.

**Consigna:** habla durante 45–60 segundos. En una segunda toma añade que `readonly` de TypeScript no valida datos externos en runtime.

---

## 3. Vocabulario con uso real

| Expresión | Significado en contexto | Ejemplo profesional |
| --- | --- | --- |
| `snapshot` | representación del estado en un momento | `The previous snapshot was captured before the maintenance window.` |
| `previous state` | estado registrado anteriormente | `The previous state was online.` |
| `current state` | estado del snapshot más reciente | `The current state is offline.` |
| `matching record` | registro con la misma identidad | `The search found a matching record by asset ID.` |
| `status transition` | cambio de un estado a otro | `The function reports each status transition.` |
| `changed from X to Y` | cambió de X a Y | `The asset changed from online to offline.` |
| `remained unchanged` | permaneció sin cambio | `The second asset remained unchanged.` |
| `no longer present` | ya no aparece | `The asset is no longer present in the current snapshot.` |
| `newly detected` | detectado por primera vez en la comparación | `This asset was newly detected in the current snapshot.` |
| `reconcile` | conciliar dos conjuntos de registros | `The function reconciles the previous and current snapshots.` |
| `baseline` | punto de comparación | `The previous snapshot acts as the baseline.` |
| `identifier` | valor usado para reconocer una entidad | `The asset ID is the stable identifier.` |
| `duplicate identifier` | identificador repetido | `A duplicate identifier makes the match ambiguous.` |
| `missing record` | registro ausente en una colección | `A missing record does not automatically mean that the asset was removed.` |
| `assumption` | condición aceptada para el diseño | `The implementation assumes unique asset IDs.` |
| `scope of the result` | alcance de lo que concluye la función | `The scope of the result is limited to status changes.` |
| `nested lookup` | búsqueda dentro de un recorrido | `The nested lookup affects time complexity.` |
| `worst case` | escenario de mayor costo | `In the worst case, every lookup scans the previous array.` |

### Contrastes importantes

```text
previous snapshot      ≠ verified historical truth
current snapshot       ≠ live state forever
missing record         ≠ confirmed deletion
no reported changes    ≠ no operational problem exists
readonly TypeScript    ≠ runtime immutability
matching identifier    ≠ validated identity
```

### V1 — Definición profesional (45–60 segundos)

**Qué debes intentar decir:** define `snapshot`, `baseline`, `matching record` y `status transition` sin traducir palabra por palabra.

- **Estructura reutilizable:** `A snapshot is … . In this comparison, the previous snapshot acts as … . A matching record is … . A status transition occurs when … .`
- **Palabras y frases útiles:** `state at a specific time`, `comparison baseline`, `same stable identifier`, `different status value`.
- **Conectores:** `in this comparison`, `when`, `for example`, `therefore`.
- **Patrones de oración:** `X is a representation of …`; `X occurs when …`; `X acts as …`.
- **Vocabulario técnico:** `snapshot`, `baseline`, `identifier`, `record`, `transition`.
- **Por qué funcionan:** producen definiciones breves que sirven en entrevistas, documentación y conversaciones de diseño.

**Consigna:** incluye un ejemplo de una transición de `online` a `offline` y una advertencia sobre registros ausentes.

---

## 4. Pronunciación guiada en texto

La sílaba en MAYÚSCULAS marca el acento aproximado. Esta guía no sustituye una evaluación de audio.

| Término | Guía aproximada | Frase breve |
| --- | --- | --- |
| `previous` | PRÍ-vi-es | `the previous snapshot` |
| `current` | KÉ-rent | `the current state` |
| `snapshot` | SNÁP-shot | `compare two snapshots` |
| `matching` | MÁ-ching | `a matching record` |
| `transition` | tran-ZÍ-shon | `a status transition` |
| `changed` | CHÉINJD | `the status changed` |
| `unchanged` | an-CHÉINJD | `it remained unchanged` |
| `identifier` | ai-DÉN-ti-fai-er | `a stable identifier` |
| `duplicate` | DIÚ-pli-ket | `a duplicate identifier` |
| `reconcile` | RÉ-kon-sail | `reconcile two snapshots` |
| `comparison` | com-PÁ-ri-son | `the comparison result` |
| `complexity` | com-PLÉK-sa-ti | `time complexity` |
| `quadratic` | cua-DRÁ-tik | `quadratic behavior` |
| `assumption` | a-SÁMP-shon | `an explicit assumption` |

### Terminación `-ed` en `changed`

`Changed` termina con el sonido consonántico `/d/`; no añadas una sílaba completa:

```text
changed from online to offline
```

En cambio, `detected` sí tiene una sílaba adicional aproximada:

```text
de-TEK-tid
```

### P1 — Cadena de pronunciación y significado (30–45 segundos)

**Qué debes intentar decir:** describe una coincidencia, una transición y un registro que no cambió.

- **Estructura reutilizable:** `The current record matches … . Its status changed from … to … . The next record remained unchanged, so … .`
- **Palabras y frases útiles:** `current record`, `previous snapshot`, `changed from`, `remained unchanged`, `not included`.
- **Conectores:** `because`, `so`, `while`.
- **Patrones de oración:** `X matches Y`; `X changed from A to B`; `X remained unchanged`.
- **Vocabulario técnico:** `record`, `snapshot`, `status`, `transition`, `result array`.
- **Por qué funcionan:** practican las combinaciones de palabras que aparecerán en el walkthrough y en una actualización de incidente.

**Consigna:** repite la secuencia dos veces. En la segunda cambia `online to offline` por `maintenance to online`.

---

## 5. Gramática en contexto

### 5.1 Estado anterior frente a estado actual

Usa pasado para el snapshot anterior y presente para el actual:

```text
The asset was online in the previous snapshot.
It is offline in the current snapshot.
```

También puedes condensar la transición:

```text
The asset changed from online to offline.
```

### 5.2 `changed`, `has changed` y `was changed`

```text
The status changed from online to offline.
```

Describe el evento de cambio.

```text
The status has changed since the previous snapshot.
```

Conecta el cambio pasado con el estado actual.

```text
The status was changed by the synchronization process.
```

Es voz pasiva y afirma que un proceso modificó el estado. No la uses si solo observaste dos valores distintos y no conoces la causa.

### 5.3 `still`, `no longer` y `remained`

```text
The asset is still online.
The asset is no longer present in the current snapshot.
The status remained unchanged.
```

- `still` indica continuidad.
- `no longer` indica que una condición dejó de ser cierta.
- `remained unchanged` compara dos momentos sin afirmar qué ocurrió entre ellos.

### 5.4 `from`, `to` y `between`

```text
The status changed from maintenance to online.
There is a difference between the previous and current values.
```

Evita:

```text
Incorrect: changed of online to offline
Natural:   changed from online to offline
```

### 5.5 Grados de certeza

```text
The two snapshots show different status values.
This suggests that the asset changed state.
The comparison does not identify the cause of the change.
The current snapshot may be incomplete.
```

La primera frase describe datos observables. La segunda es una inferencia. La tercera limita la conclusión.

### 5.6 Requisitos frente a supuestos

```text
The requirement says to report status changes for matching IDs.
The implementation assumes that each asset ID is unique within a snapshot.
```

Si la unicidad no está confirmada, conviértela en pregunta:

```text
Can we assume that asset IDs are unique within each snapshot?
```

### G1 — Cambio observable, inferencia y límite (60 segundos)

**Qué debes intentar decir:** describe un valor anterior, un valor actual, la transición aparente y algo que la comparación no demuestra.

- **Estructura reutilizable:** `In the previous snapshot, … was … . In the current snapshot, it is … . The data therefore shows … . This may indicate …; however, the comparison does not prove … .`
- **Palabras y frases útiles:** `previous value`, `current value`, `status transition`, `may indicate`, `does not prove the cause`.
- **Conectores:** `therefore`, `however`, `while`, `because`.
- **Patrones de oración:** `X was A and is now B`; `This may indicate …`; `The comparison does not prove …`.
- **Vocabulario técnico:** `snapshot`, `state`, `transition`, `evidence`, `cause`.
- **Por qué funcionan:** separan observación, inferencia y conclusión; esa separación mejora la precisión durante debugging e incidentes.

**Consigna:** usa `changed from`, `may` y `does not prove` al menos una vez.

---

## 6. Explicación técnica exacta y breve

### 6.1 Modelo de datos

```ts
type AssetStatus = "online" | "offline" | "maintenance";

type SnapshotRecord = readonly [
  assetId: string,
  siteId: string,
  status: AssetStatus,
  latencyMs: number
];

type StatusChange = readonly [
  assetId: string,
  previousStatus: AssetStatus,
  currentStatus: AssetStatus
];
```

`SnapshotRecord` tiene cuatro posiciones fijas:

1. `assetId`: identidad usada para buscar la contraparte;
2. `siteId`: sede asociada al registro;
3. `status`: estado observado;
4. `latencyMs`: latencia del snapshot.

`StatusChange` tiene tres posiciones:

1. identificador;
2. estado anterior;
3. estado actual.

La práctica compara únicamente `status`. Un cambio de latencia no se incluye porque no forma parte del contrato actual.

### 6.2 Datos de ejemplo

```ts
const previousSnapshot: readonly SnapshotRecord[] = [
  ["asset-101", "site-north", "online", 18],
  ["asset-102", "site-north", "maintenance", 0],
  ["asset-201", "site-south", "online", 24]
];

const currentSnapshot: readonly SnapshotRecord[] = [
  ["asset-101", "site-north", "offline", 0],
  ["asset-102", "site-north", "online", 20],
  ["asset-201", "site-south", "online", 26],
  ["asset-301", "site-west", "online", 14]
];
```

### 6.3 Función de comparación

```ts
function findStatusChanges(
  previous: readonly SnapshotRecord[],
  current: readonly SnapshotRecord[]
): readonly StatusChange[] {
  const changes: StatusChange[] = [];

  for (const currentRecord of current) {
    const [assetId, , currentStatus] = currentRecord;

    const previousRecord = previous.find(
      ([previousAssetId]) => previousAssetId === assetId
    );

    if (previousRecord === undefined) {
      continue;
    }

    const [, , previousStatus] = previousRecord;

    if (previousStatus === currentStatus) {
      continue;
    }

    changes.push([
      assetId,
      previousStatus,
      currentStatus
    ]);
  }

  return changes;
}
```

Resultado para los datos anteriores:

```ts
[
  ["asset-101", "online", "offline"],
  ["asset-102", "maintenance", "online"]
]
```

### 6.4 Contrato de la función

**Inputs**

- `previous`: snapshot usado como baseline;
- `current`: snapshot que se compara con el baseline.

**Criterio de coincidencia**

- dos registros coinciden cuando tienen el mismo `assetId`.

**Criterio de cambio**

- existe un cambio reportable cuando el registro aparece en ambos snapshots y sus valores de `status` son distintos.

**Output**

- un nuevo array readonly de tuples `StatusChange`;
- el array puede estar vacío cuando no se encuentran cambios reportables.

**Fuera del alcance actual**

- activos nuevos;
- activos ausentes en el snapshot actual;
- cambios de sede;
- cambios de latencia;
- la causa de una transición.

### 6.5 Trazado paso a paso

| Registro actual | Coincidencia anterior | Comparación | Acción |
| --- | --- | --- | --- |
| `asset-101 / offline` | `asset-101 / online` | distinto | agregar `[asset-101, online, offline]` |
| `asset-102 / online` | `asset-102 / maintenance` | distinto | agregar `[asset-102, maintenance, online]` |
| `asset-201 / online` | `asset-201 / online` | igual | omitir |
| `asset-301 / online` | no existe | no aplicable | omitir según el contrato actual |

### 6.6 No mutación

La función no modifica `previous` ni `current`:

- no ordena los inputs;
- no elimina ni agrega elementos a los inputs;
- no reasigna posiciones dentro de sus tuples;
- crea un array de salida independiente llamado `changes`.

Usar `changes.push(...)` modifica el array local de salida, no los arrays de entrada. Por eso estas dos afirmaciones pueden ser ciertas al mismo tiempo:

```text
The function mutates its local result array.
The function does not mutate either input array.
```

### 6.7 Complejidad

Sea:

- `n` = cantidad de registros en `current`;
- `m` = cantidad de registros en `previous`;
- `k` = cantidad de cambios encontrados.

Para cada registro actual, `find` puede recorrer gran parte o la totalidad de `previous`.

- **Tiempo en el peor caso:** `O(n × m)`.
- Si ambos arrays tienen un tamaño similar `N`, puede expresarse como `O(N²)`.
- **Espacio de salida:** `O(k)` porque el resultado crece con la cantidad de cambios.
- **Espacio auxiliar adicional:** `O(1)` sin contar el array de salida.

No digas simplemente que es `O(n)` porque hay un solo `for...of` visible. La llamada a `find` también recorre una colección.

### 6.8 ¿Por qué no optimizar inmediatamente?

Una estructura indexada podría reducir búsquedas repetidas, pero antes se debe confirmar:

- el tamaño esperado de los snapshots;
- si los IDs son únicos;
- si se necesita conservar el orden actual;
- qué hacer con duplicados;
- si claridad o rendimiento es la prioridad principal.

Una respuesta profesional:

> The current implementation favors clarity. If snapshot size makes the nested lookup expensive, I would consider indexing the previous records after confirming the uniqueness rules.

No es necesario introducir a fondo otra estructura de datos en esta sesión. El objetivo continúa siendo dominar arrays y tuples.

### 6.9 Casos límite y decisiones pendientes

| Caso | Comportamiento actual | Pregunta necesaria |
| --- | --- | --- |
| `current` vacío | devuelve `[]` | ¿es un snapshot válido o un fallo de captura? |
| `previous` vacío | devuelve `[]` | ¿todos los actuales deberían marcarse como nuevos? |
| mismo ID, mismo estado | no reporta cambio | ¿también importan latencia o sede? |
| ID solo en `current` | se omite | ¿debe reportarse como `added`? |
| ID solo en `previous` | no se visita | ¿debe reportarse como `missing` o `removed`? |
| ID duplicado en `previous` | `find` usa la primera coincidencia | ¿los IDs deben ser únicos? |
| orden diferente | no afecta la coincidencia por ID | ¿el resultado debe seguir el orden actual? |
| datos externos inválidos | el tipo no los detecta en runtime | ¿dónde ocurre la validación? |

### 6.10 TypeScript no valida snapshots externos

Aunque una variable se anote como:

```ts
readonly SnapshotRecord[]
```

una API, archivo o consulta externa puede entregar valores inválidos. Los tipos se eliminan al ejecutar JavaScript. La validación runtime pertenece a una etapa posterior de la ruta.

Frase clave:

> TypeScript describes the expected structure during development, but external snapshot data still requires runtime validation.

---

## 7. Lenguaje para explicar código paso a paso

| Parte | Pregunta mental | Lenguaje útil |
| --- | --- | --- |
| Propósito | ¿Qué resuelve? | `The function compares two snapshots and returns status transitions.` |
| Inputs | ¿Qué recibe? | `It receives a previous array and a current array.` |
| Estructura | ¿Qué significa cada tuple? | `Each tuple stores an asset ID, a site ID, a status, and a latency value.` |
| Variable local | ¿Qué acumula? | `Changes stores the transitions found so far.` |
| Recorrido | ¿Qué se visita? | `The outer loop examines each current record.` |
| Búsqueda | ¿Cómo empareja? | `Find searches the previous snapshot for the same asset ID.` |
| Condición 1 | ¿Qué ocurre si falta? | `If no match exists, the loop skips the current record.` |
| Condición 2 | ¿Qué ocurre si no cambió? | `If both statuses are equal, no transition is added.` |
| Salida | ¿Qué devuelve? | `The function returns a new array of status-change tuples.` |
| Complejidad | ¿Cuánto cuesta? | `The nested lookup leads to O(n × m) time in the worst case.` |
| Edge cases | ¿Qué falta definir? | `Duplicate and missing identifiers require explicit business rules.` |
| Evidencia | ¿Qué no prueba? | `The result does not identify why a status changed.` |

### C1 — Walkthrough principal (75–90 segundos)

**Qué debes intentar decir:** explica propósito, inputs, tuples, variable local, recorrido, búsqueda, condiciones, output, no mutación, complejidad y un límite.

- **Estructura reutilizable:** `The function receives … and returns … . Each input contains … . It initializes … . For each …, it searches … . If …, it skips … . Otherwise, it adds … . The function preserves … . In the worst case, it runs in … because … . The result does not … .`
- **Palabras y frases útiles:** `previous snapshot`, `current snapshot`, `matching asset ID`, `status differs`, `result array`, `preserve the inputs`, `nested lookup`.
- **Conectores:** `first`, `for each`, `if`, `otherwise`, `because`, `however`, `finally`.
- **Patrones de oración:** `The function receives X and returns Y`; `It searches X for Y`; `If X, it skips Y`; `This leads to …`.
- **Vocabulario técnico:** `input`, `tuple`, `destructuring`, `find`, `loop`, `condition`, `return type`, `complexity`.
- **Por qué funcionan:** cubren el código por responsabilidades y flujo de datos en vez de traducir cada línea literalmente.

**Consigna:** habla durante 75–90 segundos. Debes usar las frases `matching record`, `changed from`, `does not mutate` y `in the worst case`.

### Modelo adaptable de walkthrough

> The function receives a previous snapshot and a current snapshot, both represented as readonly arrays of fixed-position tuples. It creates a separate result array for status changes. The outer loop examines each current record and destructures its asset ID and current status. Then, `find` searches the previous snapshot for a record with the same ID. If no matching record exists, the function skips that record because newly detected assets are outside the current contract. If a match exists but both statuses are equal, it also skips the record. Otherwise, it adds a tuple containing the asset ID, the previous status, and the current status. The function does not mutate either input. In the worst case, the nested lookup takes O(n times m) time. The result reports value differences, but it does not explain why a status changed.

### C2 — Trazado de un registro (45–60 segundos)

**Qué debes intentar decir:** traza `asset-102` desde la lectura del registro hasta la tuple de salida.

- **Estructura reutilizable:** `The loop reads … from the current snapshot. It extracts … . The search finds … in the previous snapshot. The previous status was …, while the current status is … . Because …, the function adds … .`
- **Palabras y frases útiles:** `reads the tuple`, `extracts the identifier`, `finds the match`, `statuses differ`, `adds a transition`.
- **Conectores:** `then`, `while`, `because`, `therefore`.
- **Patrones de oración:** `The previous value was X, while the current value is Y`; `Because X differs from Y, …`.
- **Vocabulario técnico:** `iteration`, `destructuring`, `lookup`, `comparison`, `output tuple`.
- **Por qué funcionan:** muestran que comprendes el cambio de valores y el control de flujo, no solo la firma de la función.

**Consigna:** termina diciendo exactamente qué tres valores se agregan a la salida, sin leer el código.

---

## 8. Expresiones para trabajo diario

### Reunión diaria

```text
Yesterday, I implemented the first version of the snapshot comparison.
Today, I am reviewing duplicate IDs and missing-record behavior.
The main open question is whether new assets belong in the same result.
```

### Debugging

```text
I reproduced the unexpected transition with a minimal pair of snapshots.
The comparison is matching by array position instead of asset ID.
I confirmed the input order is not stable.
I have not confirmed whether duplicate IDs are valid.
```

### Code review

```text
Could we make the uniqueness assumption explicit?
This implementation preserves the inputs, but the nested lookup may become expensive for large snapshots.
Would it be clearer to separate added assets from status changes?
```

### Incidente

```text
The current snapshot shows two status transitions.
One asset changed from online to offline.
This comparison confirms a value difference, not the cause of the outage.
We are validating the source data before drawing a broader conclusion.
```

### Reclutador o hiring manager

```text
I am building a TypeScript portfolio project that applies my infrastructure background to software problems.
One exercise compares typed infrastructure snapshots and communicates changes without overstating the evidence.
```

### S1 — Actualización de stand-up (45–60 segundos)

**Qué debes intentar decir:** comunica avance, tarea actual, riesgo y siguiente acción.

- **Estructura reutilizable:** `Yesterday, I … . Today, I am … . I found that … . The main open question is … . Next, I will … .`
- **Palabras y frases útiles:** `implemented`, `comparison logic`, `duplicate IDs`, `missing records`, `confirm the requirement`.
- **Conectores:** `yesterday`, `today`, `however`, `before`, `next`.
- **Patrones de oración:** `I found that …`; `The main open question is whether …`; `I will X before Y`.
- **Vocabulario técnico:** `snapshot`, `lookup`, `edge case`, `requirement`, `test case`.
- **Por qué funcionan:** ofrecen una actualización breve y accionable sin afirmar que el trabajo está terminado.

**Consigna:** no excedas 60 segundos y menciona al menos una pregunta pendiente.

### D1 — Actualización de debugging (60 segundos)

**Qué debes intentar decir:** describe síntoma, reproducción, causa confirmada o hipótesis, evidencia pendiente y próximo paso.

- **Estructura reutilizable:** `The symptom is … . I reproduced it with … . I confirmed that … / My current hypothesis is … . I have not confirmed … . Next, I will … .`
- **Palabras y frases útiles:** `unexpected transition`, `minimal snapshots`, `match by position`, `unstable order`, `verify uniqueness`.
- **Conectores:** `when`, `because`, `however`, `so`, `next`.
- **Patrones de oración:** `I reproduced X by Y`; `I confirmed that …`; `I have not confirmed whether …`.
- **Vocabulario técnico:** `symptom`, `reproduction`, `root cause`, `hypothesis`, `input order`, `identifier`.
- **Por qué funcionan:** separan hechos comprobados de hipótesis y hacen visible el siguiente paso.

**Consigna:** usa una frase con `I confirmed` y otra con `I have not confirmed`.

### CR1 — Comentario de code review (45–60 segundos)

**Qué debes intentar decir:** reconoce la intención, señala un riesgo concreto y propone una acción colaborativa.

- **Estructura reutilizable:** `The current approach clearly … . One concern is … because … . Could we …? That would make … explicit.`
- **Palabras y frases útiles:** `clear comparison flow`, `duplicate identifier`, `first match`, `document the assumption`, `add a test`.
- **Conectores:** `however`, `because`, `if`, `so that`.
- **Patrones de oración:** `One concern is …`; `Could we + base verb …?`; `That would make X explicit`.
- **Vocabulario técnico:** `implementation`, `assumption`, `ambiguity`, `test coverage`, `contract`.
- **Por qué funcionan:** critican el comportamiento observable sin convertir la revisión en un juicio personal.

**Consigna:** comenta el riesgo de IDs duplicados y pide una prueba o aclaración.

### I1 — Actualización de incidente (45–60 segundos)

**Qué debes intentar decir:** comunica dos cambios observados, el alcance de la evidencia y el siguiente paso.

- **Estructura reutilizable:** `The latest comparison shows … . Asset … changed from … to … . This confirms …, but it does not establish … . We are now … .`
- **Palabras y frases útiles:** `latest snapshot`, `status transition`, `value difference`, `underlying cause`, `validate the source`.
- **Conectores:** `while`, `but`, `before`, `now`.
- **Patrones de oración:** `The data shows X`; `This confirms X, but not Y`; `We are now + -ing`.
- **Vocabulario técnico:** `incident`, `snapshot`, `transition`, `evidence`, `validation`.
- **Por qué funcionan:** mantienen la comunicación útil y evitan convertir correlación temporal en causalidad.

**Consigna:** no inventes impacto, duración, causa ni métricas. Usa únicamente los datos del ejemplo.

---

## 9. Comprensión auditiva viable en el chat

No se genera un archivo de audio. Puedes usar la función de lectura en voz alta del chat, pedir que se lea el siguiente guion o pedirle a otra persona que lo lea una vez sin mostrarte el texto.

### Guion L1

> I compared the latest asset snapshot with the previous one. Two matching assets have different status values. Asset one-oh-one changed from online to offline, and asset one-oh-two changed from maintenance to online. Asset two-oh-one remained online, so it was not included in the result. Asset three-oh-one has no matching record in the previous snapshot. The current implementation skips it because newly detected assets are outside the agreed scope. The comparison does not identify the cause of either transition.

### Preguntas de comprensión

1. How many matching assets changed status?
2. Which asset remained unchanged?
3. Why was `asset-301` skipped?
4. What does the comparison not identify?
5. Which statement describes a requirement and which describes evidence?

No revises el guion antes del primer intento. Responde con palabras clave y después escucha o lee una segunda vez.

### L1 — Resumen oral (45–60 segundos)

**Qué debes intentar decir:** resume los cambios, el registro sin cambio, el registro sin coincidencia y el límite de la conclusión.

- **Estructura reutilizable:** `The update reports … . The first asset …, while the second … . Another asset remained … . One current record was skipped because … . The comparison does not … .`
- **Palabras y frases útiles:** `two transitions`, `remained online`, `no previous match`, `outside the scope`, `does not identify the cause`.
- **Conectores:** `while`, `also`, `because`, `however`.
- **Patrones de oración:** `X changed from A to B`; `X was skipped because …`; `The result does not …`.
- **Vocabulario técnico:** `matching record`, `status value`, `snapshot`, `scope`, `cause`.
- **Por qué funcionan:** convierten comprensión auditiva en una actualización técnica concisa.

**Consigna:** resume sin repetir el guion palabra por palabra. Si no recuerdas un ID, di `one asset` y conserva la precisión del estado.

---

## 10. Pregunta técnica de entrevista

### Pregunta

> How would you compare two arrays of typed tuples without mutating them, and what is the time complexity of your approach?

### Estructura de respuesta

1. define la identidad usada para emparejar;
2. explica el recorrido del snapshot actual;
3. explica la búsqueda en el snapshot anterior;
4. define cuándo se produce una tuple de salida;
5. explica no mutación;
6. justifica complejidad de tiempo y espacio;
7. menciona supuestos y una posible optimización condicionada.

### I2 — Respuesta técnica (75–90 segundos)

**Qué debes intentar decir:** responde la pregunta completa con un enfoque actual y un posible refinamiento.

- **Estructura reutilizable:** `I would use … as the stable identifier. For each …, I would search … . When …, I would add … to a new result array. This preserves … . With an array lookup inside the loop, the worst-case time complexity is … . The output uses … space. I would confirm … . If the inputs are large, I would consider … .`
- **Palabras y frases útiles:** `stable identifier`, `matching record`, `new result array`, `worst-case complexity`, `unique IDs`, `index the baseline`.
- **Conectores:** `for each`, `when`, `because`, `before`, `if`.
- **Patrones de oración:** `I would use X to Y`; `This preserves …`; `The complexity is X because Y`.
- **Vocabulario técnico:** `readonly array`, `tuple`, `lookup`, `nested iteration`, `output-sensitive space`, `index`.
- **Por qué funcionan:** muestran razonamiento, corrección y conciencia de trade-offs sin optimizar antes de conocer las restricciones.

**Consigna:** no digas solo `O(n)`. Explica el costo de `find` dentro del recorrido.

### Modelo adaptable

> I would use the asset ID as the stable identifier. For each record in the current snapshot, I would search the previous snapshot for a tuple with the same ID. If no match exists, I would follow the agreed rule for newly detected assets. If a match exists and the statuses differ, I would add a new status-change tuple to a separate result array. This approach reads both inputs without modifying them. Because the array lookup is inside the outer loop, the worst-case time complexity is O(n times m), and the output requires O(k) space for k changes. I would first confirm that identifiers are unique and clarify how missing records should be handled. If the snapshots are large, I would consider indexing the previous records to avoid repeated linear searches.

---

## 11. Presentación profesional de 60–90 segundos

Esta actividad conecta tu experiencia real con tu transición a software sin inventar cargos, métricas o resultados.

### PP1 — Presentación adaptable

**Qué debes intentar decir:** resume tu experiencia en operaciones e infraestructura, tu enfoque actual en desarrollo Full Stack, las tecnologías principales y cómo SiteOps Tracker convierte conocimiento operativo en práctica de software.

- **Estructura reutilizable:** `I have more than … in … . Over time, I became interested in … . I am currently focusing on … . In my portfolio work, I use … to … . My operations background helps me … . I am looking for … .`
- **Palabras y frases útiles:** `more than ten years`, `IT operations and infrastructure`, `transitioning into software development`, `React and TypeScript`, `Node.js`, `PostgreSQL`, `testing`, `Docker`, `troubleshooting mindset`.
- **Conectores:** `over time`, `currently`, `for example`, `because`, `now`.
- **Patrones de oración:** `My background helps me + base verb`; `I am currently focusing on + noun/-ing`; `I use X to Y`.
- **Vocabulario técnico:** `full-stack development`, `typed data`, `debugging`, `reliability`, `infrastructure`, `software engineering`.
- **Por qué funcionan:** construyen una narrativa coherente entre experiencia previa y objetivo profesional sin presentar un proyecto ficticio como experiencia laboral.

**Respuesta modelo basada en tu experiencia:**

> I have more than ten years of experience in IT operations, infrastructure, networking, and technical troubleshooting. Over time, I became increasingly interested in building software that makes operational work clearer and more reliable. I am now focusing on full-stack development with React, TypeScript, Node.js, PostgreSQL, testing, and Docker. In my portfolio work, I use a fictional project called SiteOps Tracker to practice typed data modeling, debugging, and clear technical communication. For example, I can compare infrastructure snapshots and explain status changes without overstating what the data proves. My operations background helps me investigate problems systematically, communicate during incidents, and think about edge cases. I am looking for a Software Engineer or Full-Stack role where I can combine that experience with strong software development practices.

**Consigna:** graba una toma de 60–90 segundos con cinco notas: `background`, `transition`, `stack`, `example`, `target role`. Cambia al menos dos expresiones para que suene natural para ti.

---

## 12. Práctica conductual y de proyecto

### Pregunta conductual

> Tell me about a time when new evidence changed your understanding of a technical issue.

No inventes una historia. Elige una experiencia real de operaciones, infraestructura, soporte o desarrollo. Puedes omitir datos confidenciales y sustituir nombres por descripciones generales.

### B1 — Respuesta STAR (75–90 segundos)

**Qué debes intentar decir:** describe la situación, la interpretación inicial, la evidencia nueva, la acción que tomaste, el resultado real y el aprendizaje.

- **Estructura reutilizable:** `The situation was … . Initially, the available evidence suggested … . My responsibility was … . Then, I found …, which changed … . I responded by … . The result was … . I learned to … .`
- **Palabras y frases útiles:** `initial evidence`, `new signal`, `reassess the hypothesis`, `verify the source`, `communicate the change`, `document the finding`.
- **Conectores:** `initially`, `then`, `because`, `as a result`, `since then`.
- **Patrones de oración:** `The evidence suggested X`; `New information showed Y`; `I changed my approach by …`.
- **Vocabulario técnico:** `incident`, `hypothesis`, `evidence`, `diagnosis`, `verification`, `root cause`.
- **Por qué funcionan:** convierten experiencia operativa real en evidencia transferible para ingeniería de software.

**Consigna:** prepara seis notas: `situation`, `initial view`, `new evidence`, `action`, `real result`, `learning`. Si no recuerdas una métrica exacta, no la inventes.

### Pregunta de proyecto

> What did you learn from comparing snapshots in SiteOps Tracker?

### PJ1 — Respuesta de proyecto (60–75 segundos)

**Qué debes intentar decir:** aclara que es práctica de portafolio, describe el problema, una decisión técnica, un edge case y un aprendizaje.

- **Estructura reutilizable:** `In this portfolio exercise, I wanted to … . I modeled … as … because … . One important decision was … . An edge case is … . The exercise reinforced that … .`
- **Palabras y frases útiles:** `fictional portfolio project`, `readonly arrays`, `fixed-position tuples`, `match by identifier`, `duplicate IDs`, `limit the conclusion`.
- **Conectores:** `because`, `however`, `for example`, `therefore`.
- **Patrones de oración:** `I modeled X as Y because Z`; `One edge case is …`; `The result shows X but not Y`.
- **Vocabulario técnico:** `modeling`, `contract`, `comparison`, `complexity`, `validation`.
- **Por qué funcionan:** explican aprendizaje auténtico sin presentar la práctica como un sistema desplegado o un logro laboral.

**Consigna:** incluye una frase sobre `O(n × m)` y otra sobre runtime validation.

---

## 13. Ejercicio guiado — detectar activos nuevos

Amplía el análisis con una función separada:

```ts
type AddedAsset = readonly [
  assetId: string,
  currentStatus: AssetStatus
];

function findAddedAssets(
  previous: readonly SnapshotRecord[],
  current: readonly SnapshotRecord[]
): readonly AddedAsset[] {
  const added: AddedAsset[] = [];

  for (const currentRecord of current) {
    // 1. Extrae assetId y currentStatus.
    // 2. Busca el mismo assetId en previous.
    // 3. Si no existe, agrega una tuple a added.
  }

  return added;
}
```

### Restricciones

- no mutar `previous` ni `current`;
- no usar `any`;
- no usar `as` para silenciar el compilador;
- conservar el orden del snapshot actual;
- no mezclar todavía activos nuevos con cambios de estado;
- explicar qué significa un resultado vacío.

### Casos que debes probar

1. un activo nuevo;
2. ningún activo nuevo;
3. `previous` vacío;
4. `current` vacío;
5. orden diferente pero IDs coincidentes;
6. ID duplicado y requisito todavía no definido.

No se anticipa la solución completa. Completa los `TODO` y envía tu código como `Sesión 18 — Ejercicio guiado`.

### E1 — Diseño antes de programar (60 segundos)

**Qué debes intentar decir:** explica contrato, condición para considerar un activo nuevo, output, no mutación, complejidad y ambigüedad de duplicados.

- **Estructura reutilizable:** `The function receives … . A current asset is considered new when … . The output tuple contains … . The implementation preserves … by … . Its worst-case time complexity is … because … . Duplicate IDs would …, so I need to confirm … .`
- **Palabras y frases útiles:** `no previous match`, `added asset`, `separate result array`, `nested lookup`, `ambiguous match`.
- **Conectores:** `when`, `because`, `if`, `so`.
- **Patrones de oración:** `X is considered Y when …`; `The output contains …`; `I need to confirm whether …`.
- **Vocabulario técnico:** `contract`, `condition`, `output tuple`, `complexity`, `duplicate`.
- **Por qué funcionan:** obligan a definir comportamiento antes de convertirlo en código.

**Consigna:** habla antes de implementar. Después del código, repite la explicación y corrige cualquier diferencia entre diseño e implementación.

---

## 14. Ejercicio libre integrador

Diseña, sin escribir de inmediato la solución completa, una comparación que pueda distinguir tres categorías:

```text
added
missing
status-changed
```

Debes decidir:

- si el resultado será un solo array o tres arrays separados;
- qué tuples representan cada categoría;
- cómo identificar registros;
- cómo manejar IDs duplicados;
- si el orden del resultado importa;
- qué significa `missing` sin afirmar que el activo fue eliminado;
- qué cambios quedan fuera de alcance;
- qué complejidad aceptas;
- qué validación requerirían datos externos;
- qué preguntas harías a producto o al equipo.

No se incluye una solución completa anticipada. El objetivo es que hagas explícito el contrato.

### F1 — Explicación libre de diseño (75–90 segundos)

**Qué debes intentar decir:** define inputs, categorías, tuples de salida, reglas de coincidencia, ausencia, duplicados, no mutación, complejidad y límites.

- **Estructura reutilizable:** `I would receive … . I would separate the result into … because … . A record would be classified as added when …, missing when …, and changed when … . Each output tuple would contain … . I would preserve … . The main edge case is … . The expected complexity is … . The result would not prove … . Before implementation, I would confirm … .`
- **Palabras y frases útiles:** `comparison categories`, `stable identifier`, `separate output`, `ambiguous duplicate`, `reported as missing`, `limited evidence`.
- **Conectores:** `because`, `when`, `while`, `however`, `before`.
- **Patrones de oración:** `X is classified as Y when …`; `I would separate X from Y`; `The result would not prove …`.
- **Vocabulario técnico:** `input contract`, `tuple`, `classification`, `lookup`, `edge case`, `runtime validation`.
- **Por qué funcionan:** integran modelado, algoritmo, comunicación y aclaración de requisitos en una sola respuesta.

**Consigna:** prepara nueve notas: `inputs`, `identity`, `added`, `missing`, `changed`, `output`, `duplicates`, `complexity`, `question`.

---

## 15. Plan de pruebas sin soluciones anticipadas

Considera estas pruebas para `findStatusChanges`:

| Caso | Qué debes definir o verificar |
| --- | --- |
| dos cambios | se devuelven dos tuples en orden actual |
| mismo estado | no se agrega una tuple |
| activo nuevo | se aplica la regla actual de omisión |
| activo ausente | no aparece porque el recorrido parte de `current` |
| arrays vacíos | se devuelve un array vacío |
| orden diferente | se empareja por ID, no por índice |
| estado anterior `maintenance` | la unión acepta el valor |
| ID duplicado | se documenta la ambigüedad |
| snapshot externo inválido | no se atribuye validación runtime a TypeScript |

### T1 — Explicación del plan de pruebas (60–75 segundos)

**Qué debes intentar decir:** presenta tres pruebas normales, dos edge cases y el límite de TypeScript.

- **Estructura reutilizable:** `First, I would test … to verify … . I would also test … . For edge cases, I would include … because … . Finally, I would not assume …; external data still requires … .`
- **Palabras y frases útiles:** `expected transition`, `unchanged status`, `empty input`, `duplicate ID`, `runtime validation`.
- **Conectores:** `first`, `also`, `for edge cases`, `because`, `finally`.
- **Patrones de oración:** `I would test X to verify Y`; `This case checks whether …`; `I would not assume …`.
- **Vocabulario técnico:** `test case`, `expected output`, `edge case`, `duplicate`, `validation`.
- **Por qué funcionan:** organizan el plan por intención verificable y no por una lista desordenada de inputs.

**Consigna:** no describas la implementación del test; explica qué riesgo cubre cada caso.

---

## 16. Errores frecuentes que debes evitar

| Evita | Usa | Motivo |
| --- | --- | --- |
| `The status changed of online to offline.` | `The status changed from online to offline.` | La combinación correcta es `from … to …`. |
| `The asset is in offline.` | `The asset is offline.` | `Offline` funciona como adjetivo. |
| `The previous status is online.` | `The previous status was online.` | El snapshot anterior se describe en pasado. |
| `It continue online.` | `It remained online.` / `It is still online.` | Expresiones naturales de continuidad. |
| `There are two status change.` | `There are two status changes.` | Se necesita plural. |
| `The function compare two arrays.` | `The function compares two arrays.` | Tercera persona singular. |
| `Find return undefined.` | `Find returns undefined.` | Tercera persona singular. |
| `It searches the ID on the array.` | `It searches for the ID in the array.` | Preposiciones naturales. |
| `The function does not modifies the inputs.` | `The function does not modify the inputs.` | Después de `does not`, verbo base. |
| `Push mutates the inputs.` | `Push mutates the local result array, not the inputs.` | Precisa cuál array cambia. |
| `The complexity is O(n) because there is one loop.` | `The worst-case complexity is O(n × m) because each iteration may scan the previous array.` | Incluye el recorrido de `find`. |
| `No changes means all assets are healthy.` | `No changes means that no reportable status difference was found.` | Evita una conclusión excesiva. |
| `The missing asset was deleted.` | `The asset is missing from the current snapshot.` | Ausencia no demuestra eliminación. |
| `TypeScript validates the API response.` | `TypeScript checks declared types during development; external data needs runtime validation.` | Distingue compilación de runtime. |
| `I need to confirm if IDs are unique.` | `I need to confirm whether IDs are unique.` | `Whether` expresa una decisión pendiente con claridad. |

### Práctica de corrección

Corrige sin mirar la tabla:

1. `The function compare the current snapshot with the previous.`
2. `Asset one-oh-one changed of online at offline.`
3. `Find search the ID on the previous array.`
4. `The function does not mutates none of the inputs.`
5. `The complexity is O(n) because only has a for loop.`
6. `An empty result prove that all the site is healthy.`
7. `The missing record means the asset was deleted.`

No se anticipan las respuestas completas. Envíalas como `Sesión 18 — Corrección`.

---

## 17. Mini simulación de entrevista de cinco minutos

### Q1 — Modelo de datos (30–45 segundos)

**Qué debes intentar decir:** justifica el array exterior y las tuples interiores.

- **Estructura reutilizable:** `The outer arrays represent …, while each tuple represents … . The fixed positions are appropriate because … .`
- **Palabras y frases útiles:** `variable number of records`, `fixed positional contract`, `compact transition`.
- **Conectores:** `while`, `because`, `in this exercise`.
- **Patrones de oración:** `X represents Y`; `X is appropriate because …`.
- **Vocabulario técnico:** `array`, `tuple`, `position`, `contract`, `record`.
- **Por qué funcionan:** vinculan la estructura de tipos con el significado del dato.

### Q2 — Flujo (45–60 segundos)

**Qué debes intentar decir:** explica cómo se empareja y cuándo se agrega una transición.

- **Estructura reutilizable:** `For each …, the function … . If …, it … . When …, it adds … .`
- **Palabras y frases útiles:** `current record`, `same asset ID`, `matching tuple`, `different statuses`.
- **Conectores:** `for each`, `if`, `when`, `otherwise`.
- **Patrones de oración:** `It searches X for Y`; `It adds X when Y`.
- **Vocabulario técnico:** `iteration`, `lookup`, `condition`, `output`.
- **Por qué funcionan:** resumen el control de flujo sin leer línea por línea.

### Q3 — Complejidad (45–60 segundos)

**Qué debes intentar decir:** justifica `O(n × m)` y `O(k)`.

- **Estructura reutilizable:** `The outer loop processes … . For each iteration, find may scan … . Therefore … . The result stores …, so … .`
- **Palabras y frases útiles:** `worst case`, `entire previous array`, `number of changes`, `output space`.
- **Conectores:** `for each`, `therefore`, `because`, `so`.
- **Patrones de oración:** `X may happen for every Y`; `This leads to …`.
- **Vocabulario técnico:** `time complexity`, `space complexity`, `nested lookup`, `k changes`.
- **Por qué funcionan:** derivan la complejidad de las operaciones reales.

### Q4 — Edge cases (45–60 segundos)

**Qué debes intentar decir:** menciona activos nuevos, ausentes y duplicados como decisiones distintas.

- **Estructura reutilizable:** `A current-only record could mean …, while a previous-only record could mean … . A duplicate ID makes … ambiguous. I would confirm … before … .`
- **Palabras y frases útiles:** `newly detected`, `missing from the snapshot`, `ambiguous match`, `business rule`.
- **Conectores:** `while`, `however`, `before`, `rather than`.
- **Patrones de oración:** `X could mean …`; `I would confirm whether …`.
- **Vocabulario técnico:** `edge case`, `duplicate`, `missing`, `requirement`.
- **Por qué funcionan:** evitan convertir supuestos en comportamiento confirmado.

### Q5 — Evidencia (30–45 segundos)

**Qué debes intentar decir:** explica qué prueba y qué no prueba el resultado.

- **Estructura reutilizable:** `The result confirms … within … . It does not establish … . If the data comes from …, we still need … .`
- **Palabras y frases útiles:** `different values`, `provided snapshots`, `underlying cause`, `external source`, `runtime validation`.
- **Conectores:** `within`, `however`, `if`, `still`.
- **Patrones de oración:** `X confirms Y, not Z`; `We still need …`.
- **Vocabulario técnico:** `evidence`, `scope`, `cause`, `validation`.
- **Por qué funcionan:** mantienen precisión técnica y operativa.

---

## 18. Checklist de autoevaluación

Marca únicamente después de practicar:

- [ ] Puedo explicar que la sesión 18 sigue vinculada a la Lección técnica 2.
- [ ] Puedo diferenciar el número de sesión de inglés del número de lección técnica.
- [ ] Puedo definir `snapshot`, `baseline` y `status transition`.
- [ ] Puedo describir las cuatro posiciones de `SnapshotRecord`.
- [ ] Puedo describir las tres posiciones de `StatusChange`.
- [ ] Puedo explicar por qué se empareja por `assetId` y no por índice.
- [ ] Puedo explicar qué hace el `for...of` exterior.
- [ ] Puedo explicar qué hace `find`.
- [ ] Puedo decir qué significa `undefined` en la búsqueda.
- [ ] Puedo explicar cuándo se agrega una tuple al resultado.
- [ ] Puedo describir un registro que permaneció sin cambios.
- [ ] Puedo usar `changed from … to …` correctamente.
- [ ] Puedo distinguir estado anterior en pasado y estado actual en presente.
- [ ] Puedo explicar por qué un activo nuevo se omite en el contrato actual.
- [ ] Puedo diferenciar un registro ausente de una eliminación confirmada.
- [ ] Puedo explicar que `push` modifica el resultado local, no los inputs.
- [ ] Puedo justificar `O(n × m)` de tiempo.
- [ ] Puedo justificar `O(k)` de espacio de salida.
- [ ] Puedo explicar el problema de IDs duplicados.
- [ ] Puedo formular al menos una pregunta de requisito con `whether`.
- [ ] Puedo explicar que un array vacío no prueba que todo esté saludable.
- [ ] Puedo explicar que la comparación no identifica la causa.
- [ ] Puedo explicar que TypeScript no valida datos externos en runtime.
- [ ] Puedo dar una actualización de debugging en menos de 60 segundos.
- [ ] Puedo redactar un comentario de code review colaborativo.
- [ ] Puedo responder la pregunta técnica durante 75–90 segundos.
- [ ] Puedo adaptar mi presentación profesional sin memorizarla.
- [ ] Puedo convertir una experiencia real en una respuesta STAR.
- [ ] Completé el ejercicio guiado sin `any` ni assertions injustificadas.
- [ ] Envié al chat al menos una respuesta `C1`, `I1`, `I2`, `E1`, `B1`, `PJ1`, `F1` o `T1`.

---

## 19. Resumen de progreso esperado y próximo tema

### Después de practicar deberías poder

- comparar verbalmente un snapshot anterior y uno actual;
- explicar arrays readonly de tuples readonly;
- describir emparejamiento por identificador;
- trazar una transición de estado paso a paso;
- diferenciar registros cambiados, sin cambio, nuevos y ausentes;
- explicar no mutación con precisión;
- justificar `O(n × m)` por el `find` anidado;
- separar espacio auxiliar y espacio de salida;
- usar pasado y presente para comparar estados;
- comunicar observación, inferencia y límite de evidencia;
- hacer preguntas sobre unicidad, ausencia y alcance;
- conectar experiencia operativa real con razonamiento de software;
- recordar que TypeScript no sustituye validación runtime.

### Frases que conviene conservar

```text
The previous snapshot acts as the comparison baseline.
The function matches records by asset ID rather than array position.
The asset changed from online to offline.
The status remained unchanged, so no transition was added.
The function reads both inputs without modifying them.
The nested lookup leads to O(n times m) time in the worst case.
A missing record does not prove that the asset was deleted.
The comparison confirms a value difference, not the cause.
I need to confirm whether asset IDs are unique within each snapshot.
External snapshot data still requires runtime validation.
```

### Próximo tema condicionado a práctica real

Si no envías práctica ni confirmas que realizaste esta sesión, la siguiente continuará dentro de **Lección técnica 2 — Arrays y tuplas** con una actividad nueva y no idéntica. Crear otro archivo no contará como dominio.

Después de practicar y recibir correcciones, la siguiente decisión podrá ser:

- una evaluación integradora final de arrays, tuples y typed collections; o
- avanzar a **Lección técnica 3 — Objetos y modelado básico de datos**.

### Cómo solicitar correcciones

Envía audio o texto en este chat con una etiqueta:

```text
Sesión 18 — R1
Sesión 18 — V1
Sesión 18 — P1
Sesión 18 — G1
Sesión 18 — C1
Sesión 18 — C2
Sesión 18 — S1
Sesión 18 — D1
Sesión 18 — CR1
Sesión 18 — I1
Sesión 18 — L1
Sesión 18 — I2
Sesión 18 — PP1
Sesión 18 — B1
Sesión 18 — PJ1
Sesión 18 — E1
Sesión 18 — F1
Sesión 18 — T1
Sesión 18 — Corrección
```

Primero se atenderá tu respuesta. Se identificarán los errores importantes de gramática, vocabulario, ortografía, fluidez y comunicación técnica; se ofrecerá una versión natural; se explicarán brevemente las correcciones clave; y se darán de tres a cinco expresiones reutilizables. La pronunciación solo se comentará cuando el audio sea realmente accesible.
