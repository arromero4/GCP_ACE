# Inglés para Software Engineer — Sesión 21: paginar colecciones tipadas y comunicar límites

> **Etapa actual de la ruta:** TypeScript profundo.  
> **Lección técnica de referencia:** **Lección 2 — Arrays y tuplas** del archivo técnico `Ruta_Full_Stack_TypeScript_Lecciones_01_a_06.md`.  
> **Sesión de inglés:** **21**. El número de sesión de inglés y el número de lección técnica son independientes.  
> **Tema de refuerzo y evaluación diagnóstica:** dividir un array readonly de tuples en páginas mediante `slice`, calcular metadatos y comunicar índices, límites y resultados vacíos sin mutar el input.  
> **Objetivo profesional:** explicar una función de paginación durante una entrevista, un code review, una actualización de equipo o una investigación de debugging, incluyendo contrato, flujo, complejidad y casos límite.  
> **Meta estimada de práctica oral:** **15–20 minutos**, ajustables.  
> **Estado:** sesión preparada para practicar. No hay una respuesta posterior registrada que demuestre práctica o dominio de la sesión 20. Por eso esta sesión propone una actividad nueva dentro de **Arrays y tuplas** y no avanza todavía a objetos.

---

## Cómo usar esta sesión

1. Lee el repaso, el vocabulario y la explicación técnica.
2. Para una práctica de 15–20 minutos realiza `R1`, `P1`, `L1`, `C1`, `I1` y una actividad entre `D1`, `CR1` o `B1`.
3. Antes de hablar, escribe entre tres y siete palabras clave; no redactes un discurso completo.
4. Haz una primera toma con la estructura de apoyo y una segunda con menos notas.
5. Adapta los modelos a tu forma de comunicar. El objetivo es producir respuestas naturales, no memorizar.
6. Envía en este chat un audio o texto con la etiqueta del ejercicio, por ejemplo: `Sesión 21 — C1`.
7. En el chat se corregirán gramática, vocabulario, ortografía, fluidez y precisión técnica. La pronunciación solo se evaluará si existe audio realmente accesible.
8. La existencia de este archivo no significa que la sesión esté completada ni que la Lección técnica 2 esté dominada.

**Ruta oral sugerida:** `R1` 1 minuto; `P1` 1 minuto; `L1` 1 minuto; `C1` 2–3 minutos; `I1` 2 minutos; una actividad profesional 1 minuto; segunda toma 3–5 minutos.

---

## 1. Continuidad de la ruta

La sesión 20 trabajó una agregación por sede: recorría tuples, acumulaba conteos y calculaba promedios sin tratar mediciones ausentes como cero. Esta sesión no presupone que ese contenido ya fue practicado. Recupera tres ideas:

- un array readonly comunica que la función no debe modificar el input;
- una tuple establece un contrato posicional fijo;
- un resultado pequeño no demuestra que el conjunto completo sea pequeño.

La nueva pregunta es:

> How can we return one page from a readonly typed collection while preserving the input order and making boundary behavior explicit?

El contexto continúa siendo **SiteOps Tracker**, un proyecto ficticio de portafolio para auditar infraestructura en sedes, registrar activos, hallazgos, estados, responsables e historial. No representa un proyecto interno y no contiene clientes, logros ni métricas inventadas.

La práctica combina:

- **TypeScript:** arrays readonly, tuples readonly, unions con `undefined`, `length`, `slice` y retorno tipado;
- **algoritmos:** conversión entre números de página e índices, rango semiabierto, costo proporcional a los elementos copiados y casos fuera de rango;
- **inglés profesional:** explicar páginas, límites, orden, resultados vacíos, decisiones de contrato y evidencia limitada.

Variables usadas en complejidad:

- `n`: cantidad total de registros;
- `k`: cantidad de registros copiados a la página devuelta;
- `p`: tamaño de página solicitado.

---

## 2. Repaso breve del lenguaje anterior

| Intención | Expresión reutilizable |
| --- | --- |
| Hablar de input readonly | `The function receives a readonly collection and preserves it.` |
| Definir una tuple | `Each tuple follows a fixed positional contract.` |
| Hablar de orden | `The output preserves the order of the provided input.` |
| Limitar una conclusión | `The result describes only the supplied data.` |
| Hablar de ausencia | `An empty result is not the same as an invalid request.` |
| Distinguir evidencia | `The function does not prove why a record is missing.` |
| Hablar de complejidad | `The cost depends on the number of copied items.` |
| Hablar de validación | `TypeScript types do not validate external data at runtime.` |

### R1 — Recuperación activa (45–60 segundos)

**Qué debes intentar decir:** resume qué protegía `readonly` en la sesión anterior, qué no garantiza en runtime y por qué un resumen o una página no representa necesariamente todos los datos.

- **Estructura reutilizable:** `A readonly input communicates that … . It helps prevent … during development, but it does not … at runtime. A summary or page contains …, so it does not necessarily show … .`
- **Palabras y frases útiles:** `preserve caller data`, `compile-time contract`, `runtime freeze`, `subset of records`, `complete collection`.
- **Conectores:** `but`, `however`, `so`, `therefore`.
- **Patrones de oración:** `X communicates that Y`; `X helps prevent Y, but it does not Z`; `X contains only …`.
- **Vocabulario técnico:** `readonly`, `input`, `mutation`, `runtime`, `subset`, `collection`.
- **Por qué funcionan:** separan intención de diseño, garantía de TypeScript y alcance real del resultado.

**Consigna:** habla durante 45–60 segundos e incluye una frase con `compile time` y otra con `runtime`.

---

## 3. Vocabulario con uso real

| Expresión | Significado en contexto | Ejemplo profesional |
| --- | --- | --- |
| `pagination` | división de una colección en páginas | `Pagination returns a manageable subset of records.` |
| `page number` | número visible de página | `The page number is one-based in this contract.` |
| `page size` | cantidad máxima por página | `The requested page size is three.` |
| `page item` | elemento incluido en una página | `The second page contains three items.` |
| `total item count` | total de elementos del input | `The total item count is seven.` |
| `total page count` | número de páginas calculadas | `The total page count is three.` |
| `start index` | primera posición incluida | `The start index for page two is three.` |
| `end index` | límite que no se incluye | `The end index is exclusive.` |
| `zero-based index` | índice que comienza en cero | `JavaScript arrays use zero-based indexes.` |
| `one-based page number` | página que comienza en uno | `Users normally request a one-based page number.` |
| `offset` | cantidad de posiciones omitidas | `The offset is calculated from the page number.` |
| `boundary` | límite de un rango | `The final page tests a collection boundary.` |
| `inclusive` | que sí incluye el límite | `The start index is inclusive.` |
| `exclusive` | que no incluye el límite | `The end index is exclusive.` |
| `out of range` | fuera de las páginas disponibles | `Page four is out of range for this data set.` |
| `remaining items` | elementos que quedan | `The last page contains one remaining item.` |
| `partial page` | página con menos del máximo | `The final page is a partial page.` |
| `empty page` | página válida sin elementos | `This contract returns an empty page beyond the data.` |
| `stable order` | orden que se mantiene entre operaciones | `Reliable pagination requires a stable input order.` |
| `preserve order` | conservar el orden recibido | `Slice preserves the relative order of the selected records.` |
| `shallow copy` | copia del contenedor, no copia profunda | `Slice creates a shallow copy of the selected array segment.` |
| `mutating method` | método que cambia el array | `Splice is a mutating method.` |
| `ceil` / `round up` | redondear hacia arriba | `We round up to calculate the final partial page.` |
| `invalid argument` | parámetro que viola el contrato | `A page size of zero is an invalid argument.` |

### Contrastes importantes

```text
page number 1          ≠ array index 1
start index            ≠ page number
inclusive start        ≠ exclusive end
empty page             ≠ invalid arguments
valid number           ≠ available page
slice                   ≠ splice
readonly type           ≠ runtime freeze
preserved order         ≠ guaranteed stable source
page items              ≠ every item
```

### V1 — Definiciones profesionales (45–60 segundos)

**Qué debes intentar decir:** define `page number`, `page size`, `start index`, `exclusive end` y `partial page` con relación al ejemplo técnico.

- **Estructura reutilizable:** `The page number identifies … . The page size sets … . The start index is …, while the end index is … . A partial page occurs when … .`
- **Palabras y frases útiles:** `requested segment`, `maximum number of items`, `first included position`, `not included`, `fewer remaining records`.
- **Conectores:** `while`, `because`, `when`, `in this contract`.
- **Patrones de oración:** `X identifies Y`; `X sets the maximum …`; `X occurs when Y`.
- **Vocabulario técnico:** `page number`, `page size`, `index`, `inclusive`, `exclusive`, `partial page`.
- **Por qué funcionan:** permiten definir el contrato con palabras precisas antes de explicar la implementación.

**Consigna:** incluye la frase `The start index is inclusive, while the end index is exclusive.`

---

## 4. Pronunciación guiada en texto

La sílaba en MAYÚSCULAS marca el acento aproximado. Esta guía no sustituye una evaluación de audio.

| Término | Guía aproximada | Frase breve |
| --- | --- | --- |
| `pagination` | pa-yi-NÉI-shon | `pagination logic` |
| `page size` | PÉICH sáiz | `a page size of three` |
| `boundary` | BÁUN-da-ri | `a page boundary` |
| `inclusive` | in-KLÚ-siv | `an inclusive start` |
| `exclusive` | eks-KLÚ-siv | `an exclusive end` |
| `offset` | ÓF-set | `calculate the offset` |
| `integer` | ÍN-ti-yer | `a positive integer` |
| `remaining` | ri-MÉI-ning | `the remaining item` |
| `partial` | PÁR-shal | `a partial page` |
| `ceiling` | SÍ-ling | `use the ceiling value` |
| `preserve` | pri-ZÉRV | `preserve the order` |
| `stable` | STÉI-bol | `a stable order` |
| `shallow` | SHÁ-lou | `a shallow copy` |
| `mutate` | MIÚ-teit | `do not mutate the input` |
| `range` | RÉINCH | `out of range` |
| `slice` | SLÁIS | `slice the array` |
| `splice` | SPLÁIS | `splice mutates the array` |

### P1 — Cadena de pronunciación y significado (30–45 segundos)

**Qué debes intentar decir:** describe una solicitud de página, el rango de índices y la cantidad devuelta.

- **Estructura reutilizable:** `For page … with a page size of …, the start index is … and the exclusive end index is … . The function returns … items and preserves … .`
- **Palabras y frases útiles:** `page two`, `page size of three`, `start index three`, `exclusive end index six`, `relative order`.
- **Conectores:** `for`, `and`, `so`, `while`.
- **Patrones de oración:** `For page X with size Y, …`; `The function returns N items`; `X is inclusive while Y is exclusive`.
- **Vocabulario técnico:** `pagination`, `start index`, `exclusive end`, `slice`, `order`.
- **Por qué funcionan:** practican los números y contrastes que suelen causar confusión en un walkthrough.

**Consigna:** repite la secuencia dos veces. En la segunda contrasta `slice` con `splice`.

---

## 5. Gramática en contexto

### 5.1 Números ordinales y cardinales

```text
The second page contains three records.
Page two contains three records.
The page size is three.
The third page contains one remaining record.
```

- `second` y `third` son ordinales;
- `two` y `three` son cardinales;
- en interfaces técnicas es común decir `page two`, no `the page two`.

### 5.2 Rangos inclusivos y exclusivos

```text
The range starts at index three and stops before index six.
The start index is inclusive.
The end index is exclusive.
The function copies indexes three through five.
```

Una forma especialmente clara:

> It copies records from index three up to, but not including, index six.

### 5.3 `From`, `to`, `through` y `up to`

```text
from index three to index five
indexes three through five
from index three up to, but not including, index six
```

Para código, la tercera versión comunica con precisión un rango semiabierto.

### 5.4 `Within`, `beyond` y `out of`

```text
Page three is within the available range.
Page four is beyond the available pages.
The requested page is out of range.
```

### 5.5 `Fewer`, `remaining` y `at most`

```text
The final page contains fewer items.
Only one item remains.
A page contains at most three items.
```

- `fewer` se usa con elementos contables;
- `at most` establece un máximo;
- `remaining` describe lo que queda después de páginas anteriores.

### 5.6 Condicionales para límites

```text
If the page number is zero, the function returns undefined.
If the page is beyond the data, the function returns an empty item array.
If seven items are divided into pages of three, the final page contains one item.
```

### 5.7 Certeza y alcance

```text
The empty page confirms that this range contains no records.
It does not prove that the complete collection is empty.
The function preserves the supplied order.
It does not guarantee that the source order stays stable between calls.
```

### G1 — Regla, ejemplo y límite (60 segundos)

**Qué debes intentar decir:** explica cómo se calculan los límites de la página, qué incluye `slice` y qué no prueba una página vacía.

- **Estructura reutilizable:** `For page … and size …, the function calculates … . The range starts at … and stops before … . It therefore returns … . If the result is empty, that means … under this contract, but it does not prove … .`
- **Palabras y frases útiles:** `one-based page number`, `zero-based index`, `inclusive start`, `exclusive end`, `selected range`, `complete collection`.
- **Conectores:** `for`, `therefore`, `if`, `but`.
- **Patrones de oración:** `The range starts at X and stops before Y`; `If X, that means Y but not Z`.
- **Vocabulario técnico:** `page`, `index`, `range`, `slice`, `empty result`, `contract`.
- **Por qué funcionan:** presentan cálculo, resultado y alcance sin confundir ausencia local con ausencia total.

**Consigna:** usa `up to, but not including` al menos una vez.

---

## 6. Explicación técnica exacta y breve

### 6.1 Modelo de datos

```ts
type AssetStatus = "online" | "offline" | "maintenance";

type AssetRecord = readonly [
  assetId: string,
  siteId: string,
  status: AssetStatus
];

type AssetPage = readonly [
  items: readonly AssetRecord[],
  pageNumber: number,
  pageSize: number,
  totalItems: number,
  totalPages: number
];
```

Responsabilidades:

- `AssetRecord`: tuple readonly con tres posiciones fijas;
- `AssetPage`: tuple readonly que combina los elementos seleccionados y los metadatos;
- `items`: array readonly de tuples `AssetRecord`.

La etiqueta `items` describe la posición, pero en runtime sigue siendo la posición `0` de la tuple.

### 6.2 Datos de ejemplo

```ts
const records: readonly AssetRecord[] = [
  ["asset-101", "site-north", "online"],
  ["asset-102", "site-north", "offline"],
  ["asset-103", "site-north", "maintenance"],
  ["asset-201", "site-south", "online"],
  ["asset-202", "site-south", "offline"],
  ["asset-301", "site-west", "online"],
  ["asset-302", "site-west", "maintenance"]
];
```

Hay siete registros. El orden es parte del input proporcionado.

### 6.3 Función de paginación

```ts
function paginateAssets(
  records: readonly AssetRecord[],
  pageNumber: number,
  pageSize: number
): AssetPage | undefined {
  const hasInvalidPageNumber =
    !Number.isInteger(pageNumber) || pageNumber < 1;

  const hasInvalidPageSize =
    !Number.isInteger(pageSize) || pageSize < 1;

  if (hasInvalidPageNumber || hasInvalidPageSize) {
    return undefined;
  }

  const totalItems = records.length;
  const totalPages = Math.ceil(totalItems / pageSize);
  const startIndex = (pageNumber - 1) * pageSize;
  const endIndex = startIndex + pageSize;
  const pageItems = records.slice(startIndex, endIndex);

  return [
    pageItems,
    pageNumber,
    pageSize,
    totalItems,
    totalPages
  ];
}
```

### 6.4 Resultado para página 2 y tamaño 3

```ts
const result = paginateAssets(records, 2, 3);
```

Cálculo:

```text
totalItems = 7
totalPages = Math.ceil(7 / 3) = 3
startIndex = (2 - 1) × 3 = 3
endIndex = 3 + 3 = 6
slice(3, 6) copies indexes 3, 4 and 5
```

Resultado:

```ts
[
  [
    ["asset-201", "site-south", "online"],
    ["asset-202", "site-south", "offline"],
    ["asset-301", "site-west", "online"]
  ],
  2,
  3,
  7,
  3
]
```

### 6.5 Conversión entre página e índice

El contrato usa:

- número de página visible: comienza en `1`;
- índice de array: comienza en `0`.

Por eso:

```ts
const startIndex = (pageNumber - 1) * pageSize;
```

Tabla:

| Página | Tamaño | Inicio incluido | Fin excluido | Índices copiados |
| ---: | ---: | ---: | ---: | --- |
| 1 | 3 | 0 | 3 | 0, 1, 2 |
| 2 | 3 | 3 | 6 | 3, 4, 5 |
| 3 | 3 | 6 | 9 | 6 |
| 4 | 3 | 9 | 12 | ninguno |

### 6.6 Por qué `Math.ceil`

Con siete registros y páginas de tres:

```text
7 / 3 = 2.333...
```

Hay dos páginas completas y una página parcial. `Math.ceil` redondea hacia arriba:

```text
Math.ceil(7 / 3) = 3
```

`Math.floor` produciría `2` y ocultaría la página parcial.

### 6.7 Contrato de argumentos

La función acepta:

- `pageNumber`: entero mayor o igual que `1`;
- `pageSize`: entero mayor o igual que `1`.

Devuelve `undefined` cuando cualquiera de esos parámetros:

- no es entero;
- es cero;
- es negativo.

Ejemplos:

```ts
paginateAssets(records, 0, 3);   // undefined
paginateAssets(records, 1.5, 3); // undefined
paginateAssets(records, 1, 0);   // undefined
```

Este guard valida únicamente los argumentos de paginación. No valida en runtime la estructura de cada `AssetRecord` recibido desde una fuente externa.

### 6.8 Página fuera de rango

Este contrato distingue entre:

- **argumento inválido:** `pageNumber` o `pageSize` no cumple las reglas → `undefined`;
- **página numéricamente válida pero más allá de los datos:** devuelve una tuple con `items` vacío.

```ts
paginateAssets(records, 4, 3);
```

Resultado conceptual:

```ts
[
  [],
  4,
  3,
  7,
  3
]
```

La página solicitada es positiva y entera, pero supera `totalPages`.

Otra API podría:

- devolver `undefined`;
- limitar la página a la última disponible;
- lanzar un error;
- devolver una union con un código de estado.

Ninguna opción debe asumirse. El equipo necesita acordar y documentar el contrato.

### 6.9 Input vacío

```ts
paginateAssets([], 1, 3);
```

Resultado conceptual:

```ts
[
  [],
  1,
  3,
  0,
  0
]
```

Bajo este contrato:

- la solicitud usa argumentos válidos;
- no existen elementos;
- `totalPages` es `0`;
- `items` está vacío.

La frase precisa es:

> The request is valid, but the collection contains no available items.

### 6.10 `slice` no es `splice`

```ts
const pageItems = records.slice(startIndex, endIndex);
```

`slice`:

- devuelve un nuevo array;
- no modifica `records`;
- incluye `startIndex`;
- excluye `endIndex`;
- conserva el orden relativo de los elementos seleccionados.

`splice` modifica el array sobre el que se ejecuta y no corresponde a este contrato.

`slice` crea una copia superficial del segmento. En este ejemplo, cada tuple contiene valores primitivos y está tipada como readonly. No se afirma que `slice` realice una copia profunda.

### 6.11 Orden estable

La función preserva el orden recibido:

```text
input order → selected segment in the same relative order
```

Pero no ordena ni estabiliza la fuente. Si el array cambia de orden entre dos solicitudes, un registro puede pasar de una página a otra.

Una explicación cuidadosa:

> The function preserves the current input order, but reliable pagination across separate calls also requires the source order to remain stable.

### 6.12 Complejidad

Para una página con `k` elementos:

- leer `length`, calcular índices y metadatos: `O(1)`;
- copiar el segmento con `slice`: `O(k)`;
- espacio adicional para la página: `O(k)`.

Por tanto:

```text
Time:  O(k)
Space: O(k)
```

Como `k ≤ pageSize`, también puede describirse como costo proporcional al tamaño real de la página.

La función no recorre los `n` registros completos para obtener una página. Esta afirmación corresponde al array ya disponible en memoria y a esta implementación concreta; no describe automáticamente una consulta a base de datos.

### 6.13 Qué garantiza y qué no

La función garantiza, si recibe valores que cumplen el contrato estático:

- cálculo de índices según página basada en uno;
- rango de `slice` con inicio incluido y fin excluido;
- conservación del input;
- metadatos coherentes con `records.length`;
- orden relativo preservado.

No garantiza:

- que cada tuple externa haya sido validada en runtime;
- que el orden de origen permanezca igual entre llamadas;
- que no existan IDs duplicados;
- que el conjunto esté actualizado;
- que una página vacía signifique que el conjunto completo está vacío;
- una política distinta para páginas fuera de rango.

---

## 7. Lenguaje para explicar código paso a paso

### Inputs

```text
The function receives a readonly array of asset tuples.
It also receives a one-based page number and a positive page size.
```

### Variables

```text
totalItems stores the input length.
totalPages stores the rounded-up page count.
startIndex stores the first included position.
endIndex stores the first excluded position.
pageItems stores the copied segment.
```

### Estructuras de datos

```text
Each asset is a readonly tuple.
The input is a readonly array of those tuples.
The output is a readonly tuple containing a nested readonly array and metadata.
```

### Control de flujo

```text
First, the function checks the pagination arguments.
If either argument is invalid, it returns undefined early.
Otherwise, it calculates the indexes and slices the array.
Finally, it returns the page tuple.
```

### Condiciones

```text
The page number and page size must be positive integers.
A page beyond the available range is not treated as an invalid argument in this contract.
```

### Salida

```text
The output contains the page items, requested page number, page size, total item count, and total page count.
```

### Complejidad

```text
The time and additional space are O(k), where k is the number of copied page items.
```

### Casos límite

```text
I would test an empty input, the first page, a full middle page, a partial final page, an out-of-range page, zero, negative values, and non-integer arguments.
```

### C1 — Walkthrough completo del código (75–90 segundos)

**Qué debes intentar decir:** explica inputs, variables, estructuras de datos, guard, cálculo, rango, output, complejidad y casos límite; después realiza tu propia explicación sin leer el modelo.

- **Estructura reutilizable:** `The function receives … . First, it checks … . If …, it returns … . Otherwise, it calculates … . The start index …, while the end index … . Slice then … . Finally, the function returns … . The complexity is … because … . I would test … .`
- **Palabras y frases útiles:** `readonly array`, `positive integer`, `early return`, `one-based page`, `zero-based index`, `exclusive end`, `copied segment`, `partial page`.
- **Conectores:** `first`, `if`, `otherwise`, `then`, `finally`, `because`.
- **Patrones de oración:** `The function receives X and returns Y`; `If X, it returns Y`; `X stores Y`; `The complexity is X because Y`.
- **Vocabulario técnico:** `input`, `variable`, `tuple`, `array`, `control flow`, `condition`, `output`, `complexity`, `edge case`.
- **Por qué funcionan:** cubren todas las dimensiones esperadas en una explicación técnica y mantienen una secuencia fácil de seguir.

**Consigna:** explica el código durante 75–90 segundos. Debes mencionar explícitamente inputs, variables, estructuras de datos, control de flujo, condiciones, output, complejidad y al menos tres casos límite.

### C2 — Trazado de página 2 (60–75 segundos)

**Qué debes intentar decir:** traza `paginateAssets(records, 2, 3)` desde los argumentos hasta la tuple de retorno.

- **Estructura reutilizable:** `The request asks for … . Both arguments are …, so … . The total item count is … and the total page count is … . The start index is calculated as … . The exclusive end is … . Slice copies … . The returned tuple therefore contains … .`
- **Palabras y frases útiles:** `page two`, `size three`, `valid positive integers`, `seven total items`, `three total pages`, `indexes three through five`.
- **Conectores:** `so`, `because`, `then`, `therefore`.
- **Patrones de oración:** `X is calculated as …`; `Slice copies indexes …`; `The tuple contains …`.
- **Vocabulario técnico:** `argument`, `total count`, `start index`, `exclusive end`, `slice`, `return tuple`.
- **Por qué funcionan:** convierten fórmulas e índices en una narrativa verificable sin saltos.

**Consigna:** habla sin mirar la tabla y comprueba al final que nombraste exactamente tres IDs: `asset-201`, `asset-202` y `asset-301`.

---

## 8. Expresiones habituales para trabajo diario

### Actualización de equipo

```text
I implemented the pagination contract for the in-memory typed collection.
The function now returns page items together with total counts.
I kept page numbers one-based and array indexes zero-based.
The remaining question is how we want to handle pages beyond the available range.
```

### Debugging

```text
I reproduced the issue on the second page.
The symptom is that the first record of each page is skipped.
The current formula uses the page number directly as an array index.
The start index should subtract one before multiplying by the page size.
```

### Code review

```text
The use of slice preserves the input, which matches the readonly contract.
Could we name the exclusive end index explicitly?
That would make the range easier to verify.
I would also add tests for an empty input and a partial final page.
```

### Comunicación con producto o equipo

```text
Should an out-of-range page return an empty page or an error result?
Do we need the input order to remain stable across requests?
Is the page size fixed, configurable, or capped?
Should an empty collection report zero pages or one empty page?
```

### Comunicación con reclutadores

```text
I am strengthening my TypeScript fundamentals through small, typed collection exercises.
I can explain not only the implementation but also the contract, edge cases, and trade-offs.
My operations background helps me communicate boundaries and avoid unsupported conclusions.
```

### S1 — Actualización de stand-up (45–60 segundos)

**Qué debes intentar decir:** comunica qué implementaste, una decisión tomada, una pregunta abierta y el siguiente paso.

- **Estructura reutilizable:** `Yesterday, I worked on … . I implemented … and chose … because … . The open question is whether … . Today, I will … and verify … .`
- **Palabras y frases útiles:** `pagination helper`, `one-based page number`, `exclusive end`, `out-of-range behavior`, `edge-case tests`.
- **Conectores:** `yesterday`, `because`, `however`, `today`, `before`.
- **Patrones de oración:** `I implemented X`; `I chose X because Y`; `The open question is whether …`; `I will X before Y`.
- **Vocabulario técnico:** `pagination`, `contract`, `slice`, `boundary`, `test case`.
- **Por qué funcionan:** separan trabajo completado, decisión técnica y requisito todavía no confirmado.

**Consigna:** no excedas 60 segundos y formula la política fuera de rango como una pregunta, no como un hecho.

### D1 — Actualización de debugging (60 segundos)

**Qué debes intentar decir:** describe un error que omite el primer elemento de cada página, la reproducción, la causa confirmada y la corrección.

- **Estructura reutilizable:** `The symptom is … . I reproduced it with page … and size … . I confirmed that the formula … . Because page numbers are … while indexes are …, the start index should … . My next step is to … .`
- **Palabras y frases útiles:** `skips the first item`, `off-by-one error`, `uses pageNumber directly`, `subtract one`, `boundary test`.
- **Conectores:** `when`, `because`, `while`, `therefore`, `next`.
- **Patrones de oración:** `I reproduced X with Y`; `I confirmed that …`; `X should be Y rather than Z`.
- **Vocabulario técnico:** `symptom`, `reproduction`, `off-by-one`, `index`, `formula`, `regression test`.
- **Por qué funcionan:** conectan síntoma, evidencia, causa y prevención sin especular sobre datos no observados.

**Consigna:** usa las expresiones `I reproduced`, `I confirmed` y `off-by-one error`.

### CR1 — Comentario de code review (45–60 segundos)

**Qué debes intentar decir:** reconoce una decisión correcta, señala una ambigüedad y propone una prueba verificable.

- **Estructura reutilizable:** `Using … is a good fit because … . One ambiguity is … . Could we …? I would also add a test where … so that … .`
- **Palabras y frases útiles:** `slice preserves the input`, `out-of-range contract`, `name the exclusive end`, `partial final page`, `expected metadata`.
- **Conectores:** `because`, `however`, `also`, `so that`.
- **Patrones de oración:** `X is a good fit because Y`; `One ambiguity is …`; `Could we + base verb …?`; `That would make … explicit`.
- **Vocabulario técnico:** `readonly contract`, `boundary`, `metadata`, `test coverage`, `behavior`.
- **Por qué funcionan:** mantienen el comentario colaborativo, específico y basado en un resultado comprobable.

**Consigna:** solicita una prueba para siete registros con tamaño tres sin escribir la solución.

### RC1 — Explicación breve para reclutador (30–45 segundos)

**Qué debes intentar decir:** explica qué habilidad estás desarrollando y por qué es relevante sin recitar código.

- **Estructura reutilizable:** `I am currently strengthening … . One exercise involves … . It helps me practice …, …, and … . I can apply the same communication approach when … .`
- **Palabras y frases útiles:** `TypeScript fundamentals`, `typed collections`, `pagination contract`, `edge cases`, `technical communication`, `code review`.
- **Conectores:** `currently`, `for example`, `because`, `when`.
- **Patrones de oración:** `I am strengthening X through Y`; `It helps me practice A, B, and C`; `I apply this when …`.
- **Vocabulario técnico:** `TypeScript`, `typed collection`, `contract`, `debugging`, `review`.
- **Por qué funcionan:** muestran aprendizaje transferible y comunicación profesional sin presentar el ejercicio como experiencia de producción.

**Consigna:** menciona que SiteOps Tracker es un proyecto ficticio de portafolio.

---

## 9. Comprensión auditiva viable en el chat

No se genera un archivo de audio. Puedes usar la función de lectura en voz alta del chat o pedir a otra persona que lea el guion una vez sin mostrarte el texto.

### Guion L1

> The collection contains seven asset records. The request asks for page two with a page size of three. Because page numbers start at one but array indexes start at zero, the start index is three. The exclusive end index is six. Slice therefore copies indexes three, four, and five, returning three records. The total page count is three because the final record needs a partial third page. A request for page four uses valid numeric arguments, but it is beyond the available data. Under this contract, it returns an empty item array rather than undefined. That empty page does not mean the complete collection has no records.

### Preguntas

1. How many records are in the complete collection?
2. What page and page size were requested?
3. Why is the start index three?
4. Which indexes does `slice` copy?
5. Why are there three total pages?
6. What does page four return under this contract?
7. What does the empty page fail to prove?

### L1 — Resumen oral (45–60 segundos)

**Qué debes intentar decir:** resume la solicitud, el rango, la página parcial y el significado limitado de la página vacía.

- **Estructura reutilizable:** `The input contains … . The request asks for … . The calculated range starts at … and stops before …, so … . There are … total pages because … . Page four returns … under this contract, but that does not mean … .`
- **Palabras y frases útiles:** `seven records`, `page two`, `page size three`, `indexes three through five`, `partial final page`, `empty item array`.
- **Conectores:** `because`, `so`, `under this contract`, `but`.
- **Patrones de oración:** `The range starts at X and stops before Y`; `There are N pages because …`; `X does not mean Y`.
- **Vocabulario técnico:** `input`, `page`, `range`, `slice`, `partial page`, `empty result`.
- **Por qué funcionan:** convierten comprensión auditiva en una explicación cuantitativa y limitada.

**Consigna:** responde primero las siete preguntas y luego resume sin repetir el guion palabra por palabra.

---

## 10. Pregunta técnica de entrevista

> How would you paginate a readonly array of typed tuples without mutating the input, and how would you define the boundary behavior?

### I1 — Respuesta técnica (75–90 segundos)

**Qué debes intentar decir:** define input, output, validación de argumentos, fórmula, `slice`, resultado fuera de rango, complejidad y decisiones pendientes.

- **Estructura reutilizable:** `I would receive … and return … . First, I would verify … . Then, I would calculate … using … . Because pages are … and indexes are …, the start index would … . I would use slice because … . Under this contract, an out-of-range page would …, while invalid arguments would … . The complexity would be … . I would confirm … before production use.`
- **Palabras y frases útiles:** `readonly array of tuples`, `page metadata`, `positive integers`, `round up`, `inclusive start`, `exclusive end`, `empty page`, `stable order`.
- **Conectores:** `first`, `then`, `because`, `while`, `under this contract`, `before`.
- **Patrones de oración:** `I would receive X and return Y`; `I would use X because Y`; `X would return A, while Y would return B`.
- **Vocabulario técnico:** `pagination`, `tuple`, `slice`, `boundary`, `complexity`, `edge case`, `contract`.
- **Por qué funcionan:** muestran dominio del algoritmo y capacidad de hacer explícitas las decisiones que una implementación por sí sola no resuelve.

**Consigna:** incluye `O(k) time`, `O(k) space`, `one-based`, `zero-based` y una pregunta sobre orden estable.

### Modelo adaptable

> I would receive a readonly array of asset tuples, a one-based page number, and a positive page size. I would return a readonly tuple containing the selected records and the pagination metadata. First, I would verify that both numeric arguments are positive integers. Then, I would round up the total item count divided by the page size to calculate the total pages. Because user-facing pages start at one while array indexes start at zero, the start index would be the page number minus one, multiplied by the page size. I would use `slice` with an inclusive start and an exclusive end because it returns a new array without mutating the input. Under this contract, invalid arguments return undefined, while a page beyond the available data returns an empty item array. The time and extra space are O(k), where k is the number of copied records. I would confirm the out-of-range policy and stable-order requirement before production use.

---

## 11. Presentación profesional de 60–90 segundos

### PP1 — Presentación adaptable

**Qué debes intentar decir:** resume experiencia, transición, stack, ejercicio de paginación y valor transferible hacia un puesto de Software Engineer.

- **Estructura reutilizable:** `I have more than … in … . Over time, I became interested in … . I am currently focusing on … . In my portfolio work, I use … to … . One recent exercise involved … . It helped me practice … . My background helps me … . I am looking for … .`
- **Palabras y frases útiles:** `more than ten years`, `IT operations and infrastructure`, `transitioning into software development`, `React and TypeScript`, `Node.js`, `PostgreSQL`, `testing`, `Docker`, `typed collections`, `boundary behavior`.
- **Conectores:** `over time`, `currently`, `for example`, `because`, `now`.
- **Patrones de oración:** `My background helps me + base verb`; `I am focusing on + noun/-ing`; `I use X to Y`; `One exercise involved …`.
- **Vocabulario técnico:** `full-stack development`, `TypeScript fundamentals`, `pagination`, `debugging`, `data boundaries`.
- **Por qué funcionan:** construyen una transición profesional coherente, conectan experiencia real con habilidades nuevas y dejan claro que es práctica de portafolio.

**Respuesta modelo:**

> I have more than ten years of experience in IT operations, infrastructure, networking, and technical troubleshooting. Over time, I became increasingly interested in building software that makes operational information easier to use and explain. I am currently focusing on full-stack development with React, TypeScript, Node.js, PostgreSQL, testing, and Docker. In my portfolio work, I use a fictional project called SiteOps Tracker to practice typed data modeling and technical communication. One recent exercise involved paginating a readonly collection of asset tuples, calculating page boundaries, and defining what an empty or out-of-range result means. It helped me practice TypeScript contracts, edge cases, and clear code walkthroughs. My operations background helps me investigate unexpected behavior, communicate limitations, and avoid unsupported conclusions. I am looking for a Software Engineer or Full-Stack role where I can combine that experience with strong development practices.

**Consigna:** graba 60–90 segundos con seis notas: `background`, `transition`, `stack`, `portfolio`, `pagination`, `target role`. Cambia al menos dos expresiones del modelo para que suene natural.

---

## 12. Práctica conductual y de proyecto

### B1 — Pregunta conductual STAR (75–90 segundos)

> Tell me about a time when you had to narrow a large set of operational information to what the team needed first.

**Qué debes intentar decir:** usa una experiencia real; explica situación, audiencia, criterio de selección, acción, resultado real y aprendizaje, sin revelar nombres privados ni inventar métricas.

- **Estructura reutilizable:** `The situation was … . The team needed … first because … . My responsibility was … . I organized the information by … and selected … . I communicated what was included and what remained outside the current scope. The actual result was … . I learned to … .`
- **Palabras y frases útiles:** `large set of records`, `immediate priority`, `relevant subset`, `define the scope`, `remaining items`, `follow-up`, `actual outcome`.
- **Conectores:** `at first`, `because`, `then`, `however`, `as a result`.
- **Patrones de oración:** `The team needed X before Y`; `I selected X based on Y`; `I made it clear that …`; `The actual result was …`.
- **Vocabulario técnico:** `scope`, `priority`, `subset`, `operational data`, `communication`, `follow-up`.
- **Por qué funcionan:** transforman experiencia operativa real en evidencia de priorización, comunicación y manejo de alcance.

**Consigna:** usa siete notas: `situation`, `audience`, `priority`, `selection`, `communication`, `result`, `learning`. No uses nombres identificables ni cifras que no recuerdes con certeza.

### PJ1 — Pregunta de proyecto (60–75 segundos)

> What did you learn from the pagination exercise in SiteOps Tracker?

**Qué debes intentar decir:** aclara que es práctica ficticia, describe problema, diseño, trade-off y requisito pendiente.

- **Estructura reutilizable:** `In this fictional portfolio exercise, I wanted to … . I modeled … as … because … . The function calculates … and uses … . One important distinction is … . The trade-off is … . Before extending it, I would confirm … .`
- **Palabras y frases útiles:** `readonly tuple`, `nested typed collection`, `one-based page`, `zero-based index`, `slice`, `out-of-range behavior`, `stable ordering`.
- **Conectores:** `because`, `then`, `however`, `while`, `before`.
- **Patrones de oración:** `I modeled X as Y because Z`; `The function uses X to Y`; `One distinction is A versus B`.
- **Vocabulario técnico:** `pagination`, `contract`, `boundary`, `mutation`, `complexity`, `requirement`.
- **Por qué funcionan:** presentan aprendizaje auténtico sin convertir un ejercicio ficticio en experiencia de producción.

**Consigna:** incluye la diferencia entre argumento inválido y página vacía, además de `O(k)`.

---

## 13. Ejercicio guiado — añadir navegación de página

Amplía la tuple de salida:

```ts
type NavigableAssetPage = readonly [
  items: readonly AssetRecord[],
  pageNumber: number,
  pageSize: number,
  totalItems: number,
  totalPages: number,
  hasPreviousPage: boolean,
  hasNextPage: boolean
];

function paginateAssetsWithNavigation(
  records: readonly AssetRecord[],
  pageNumber: number,
  pageSize: number
): NavigableAssetPage | undefined {
  // 1. Reutiliza el contrato de argumentos positivos y enteros.
  // 2. Calcula totalItems y totalPages.
  // 3. Calcula startIndex y endIndex.
  // 4. Obtén pageItems sin mutar records.
  // 5. Define hasPreviousPage.
  // 6. Define hasNextPage para una página dentro del rango.
  // 7. Decide y documenta los flags para input vacío
  //    y para una página más allá de totalPages.

  return undefined;
}
```

### Restricciones

- no usar `any`;
- no usar assertions para silenciar errores;
- no usar `splice`;
- conservar el orden recibido;
- no modificar `records`;
- probar input vacío;
- probar primera, intermedia, última y fuera de rango;
- documentar la política de flags sin asumirla.

No se anticipa la solución completa.

### E1 — Diseño antes de programar (60–75 segundos)

**Qué debes intentar decir:** define las posiciones de la tuple, reglas para `hasPreviousPage` y `hasNextPage`, casos ambiguos, complejidad y pruebas.

- **Estructura reutilizable:** `The output tuple would contain … . For a page within range, hasPreviousPage would … and hasNextPage would … . The ambiguous cases are … . I would define … before implementing … . The complexity would remain … because … . I would test … .`
- **Palabras y frases útiles:** `navigation flags`, `first page`, `last available page`, `empty input`, `out-of-range page`, `explicit policy`, `copied items`.
- **Conectores:** `for`, `while`, `however`, `before`, `because`.
- **Patrones de oración:** `X would be true when Y`; `The ambiguous case is …`; `I would confirm X before Y`.
- **Vocabulario técnico:** `tuple position`, `boolean flag`, `boundary`, `contract`, `complexity`, `test case`.
- **Por qué funcionan:** obligan a definir semántica y casos límite antes de convertirlos en booleanos.

**Consigna:** habla antes de programar. No presentes como hecho la política para página fuera de rango; formula tu decisión y justifícala.

---

## 14. Ejercicio libre integrador

Diseña una función para paginar hallazgos tipados de SiteOps Tracker.

Tuple inicial sugerida:

```ts
type FindingSeverity = "low" | "medium" | "high";

type FindingRecord = readonly [
  findingId: string,
  siteId: string,
  severity: FindingSeverity,
  isResolved: boolean
];
```

El resultado debe comunicar:

```text
elementos de la página
número solicitado
tamaño
total de registros
total de páginas
cantidad de hallazgos high en la página
```

Decide y documenta:

- página basada en uno o en cero;
- rango inclusivo/exclusivo;
- comportamiento para argumentos inválidos;
- comportamiento fuera de rango;
- input vacío;
- tamaño máximo permitido, si lo hubiera;
- conservación del orden;
- posible necesidad de ordenar antes de paginar;
- efecto de cambios entre solicitudes;
- complejidad;
- validación runtime futura.

No se incluye solución completa.

### F1 — Explicación libre (75–90 segundos)

**Qué debes intentar decir:** presenta contrato, cálculo, selección, metadatos, conteo local, límites, complejidad y una pregunta para el equipo.

- **Estructura reutilizable:** `The function would receive … . I would define page numbers as … . First, it would … . Then, it would calculate … and select … . The output would include … . Invalid arguments would …, while an out-of-range page would … . The time complexity would be … because … . One requirement I would confirm is … .`
- **Palabras y frases útiles:** `finding tuple`, `page metadata`, `high-severity count`, `selected page`, `stable input order`, `maximum page size`.
- **Conectores:** `first`, `then`, `while`, `because`, `however`, `before`.
- **Patrones de oración:** `The function would receive X and return Y`; `X would happen when Y`; `I would confirm whether …`.
- **Vocabulario técnico:** `input contract`, `pagination`, `severity`, `count`, `boundary`, `complexity`, `runtime validation`.
- **Por qué funcionan:** integran arrays, tuples, cálculo de página y comunicación de requisitos sin depender de una solución memorizada.

**Consigna:** prepara diez notas: `input`, `page base`, `guard`, `start`, `end`, `slice`, `metadata`, `high count`, `complexity`, `question`.

---

## 15. Plan de pruebas

| Caso | Entrada conceptual | Qué verificar |
| --- | --- | --- |
| primera página completa | 7 elementos, página 1, tamaño 3 | índices 0–2 y metadatos |
| página intermedia completa | 7, página 2, tamaño 3 | índices 3–5 |
| última página parcial | 7, página 3, tamaño 3 | un elemento |
| página fuera de rango | 7, página 4, tamaño 3 | `items` vacío bajo este contrato |
| input vacío | 0, página 1, tamaño 3 | cero elementos y cero páginas |
| página cero | 7, página 0, tamaño 3 | `undefined` |
| página negativa | 7, página -1, tamaño 3 | `undefined` |
| página decimal | 7, página 1.5, tamaño 3 | `undefined` |
| tamaño cero | 7, página 1, tamaño 0 | `undefined` |
| tamaño negativo | 7, página 1, tamaño -3 | `undefined` |
| tamaño decimal | 7, página 1, tamaño 2.5 | `undefined` |
| tamaño mayor al total | 7, página 1, tamaño 10 | siete elementos, una página |
| orden | tuples conocidas | mismo orden relativo |
| no mutación | conservar referencia de input | input sin cambios |
| IDs duplicados | dos tuples con mismo ID | ambos permanecen; paginar no deduplica |
| fuente cambiante | orden diferente entre llamadas | documentar desplazamiento de elementos |
| datos externos inválidos | estructura inesperada | requiere validación runtime aparte |

### T1 — Explicación del plan (60–75 segundos)

**Qué debes intentar decir:** agrupa los casos por ruta normal, límites, argumentos inválidos, integridad y estabilidad.

- **Estructura reutilizable:** `First, I would test … to verify the normal path. Then, I would test … to cover the final boundary. For invalid arguments, I would include … . I would also verify … because … . Finally, I would document … .`
- **Palabras y frases útiles:** `full page`, `partial page`, `out-of-range page`, `positive integer`, `input remains unchanged`, `stable order`.
- **Conectores:** `first`, `then`, `also`, `because`, `finally`.
- **Patrones de oración:** `I would test X to verify Y`; `This case distinguishes A from B`; `I would document …`.
- **Vocabulario técnico:** `normal path`, `edge case`, `boundary`, `invalid argument`, `mutation`, `ordering`.
- **Por qué funcionan:** organizan el plan por riesgos y explican qué defecto detectaría cada prueba.

**Consigna:** menciona al menos seis casos y explica la razón de cada uno, no la sintaxis de un framework.

---

## 16. Errores frecuentes

| Evita | Usa |
| --- | --- |
| `The function paginate the records.` | `The function paginates the records.` |
| `The page have three items.` | `The page has three items.` |
| `Page two start in index three.` | `Page two starts at index three.` |
| `The end index is included.` | `The end index is exclusive.` |
| `It copies from three until six included.` | `It copies from index three up to, but not including, index six.` |
| `The last page has less items.` | `The last page has fewer items.` |
| `The page is out of rank.` | `The page is out of range.` |
| `Seven divided between three.` | `Seven divided by three.` |
| `Math.ceil rounds to down.` | `Math.ceil rounds up.` |
| `Slice changes the original array.` | `Slice returns a new array and preserves the original.` |
| `Splice and slice are the same.` | `Splice mutates; slice copies a segment.` |
| `Readonly freezes the data.` | `Readonly is a compile-time restriction, not a runtime freeze.` |
| `An empty page means no records exist.` | `An empty page means that the selected range contains no records.` |
| `Undefined means the page is empty.` | `Undefined means the pagination arguments are invalid in this contract.` |
| `The complexity is O(n).` | `The complexity is O(k) for k copied items.` |
| `The function sorts the records.` | `The function preserves the supplied order; it does not sort.` |
| `The order is always stable.` | `The function requires a stable source order across separate requests.` |
| `Types validate the API response.` | `Types alone do not validate external data at runtime.` |

### Práctica de corrección

Corrige:

1. `The function paginate seven record in three page.`
2. `Page two start in the index three and finish in six included.`
3. `The last page has less items because seven divide between three.`
4. `Slice mutate the input but readonly freeze it in runtime.`
5. `A page out of rank always return undefined.`
6. `An empty result prove that the collection has not records.`
7. `The algorithm is O(n) because pagination inspect all records.`
8. `The function guarantee a stable order between requests.`

Envía como `Sesión 21 — Corrección`.

---

## 17. Mini simulación integradora

Preguntas:

1. Why does the formula subtract one from the page number?
2. Which boundary is inclusive and which is exclusive?
3. Why does the function use `Math.ceil`?
4. What is the difference between invalid arguments and an out-of-range page?
5. Why is `slice` compatible with the readonly input contract?
6. What are the time and space complexities?
7. What does the function assume about order?
8. What does an empty page fail to prove?

### MI1 — Simulación oral (75–90 segundos)

**Qué debes intentar decir:** responde las ocho preguntas como una explicación conectada, no como respuestas aisladas.

- **Estructura reutilizable:** `The formula subtracts one because … . The range uses … . Math.ceil is necessary because … . Under this contract, invalid arguments …, while an out-of-range page … . Slice is appropriate because … . The function runs in … and uses … . It preserves … but assumes … . Finally, an empty page does not prove … .`
- **Palabras y frases útiles:** `one-based page`, `zero-based index`, `inclusive start`, `exclusive end`, `partial page`, `new array`, `stable source order`, `complete collection`.
- **Conectores:** `because`, `while`, `therefore`, `however`, `finally`.
- **Patrones de oración:** `X is necessary because Y`; `X returns A, while Y returns B`; `The function preserves X but assumes Y`.
- **Vocabulario técnico:** `formula`, `boundary`, `slice`, `contract`, `complexity`, `ordering`, `evidence`.
- **Por qué funcionan:** integran cálculo, contrato, mutabilidad, rendimiento y límites de la evidencia en una sola respuesta.

**Consigna:** responde sin leer el código y limita tu explicación a 90 segundos.

---

## 18. Checklist de autoevaluación

Marca únicamente después de practicar:

- [ ] Puedo explicar que la sesión 21 corresponde a la Lección técnica 2.
- [ ] Puedo diferenciar sesión de inglés 21 de lección técnica 2.
- [ ] Puedo definir `pagination`, `page size`, `boundary` y `partial page`.
- [ ] Puedo usar números ordinales y cardinales con páginas.
- [ ] Puedo diferenciar número de página basado en uno e índice basado en cero.
- [ ] Puedo explicar la fórmula de `startIndex`.
- [ ] Puedo explicar por qué el inicio es inclusivo.
- [ ] Puedo explicar por qué el final es exclusivo.
- [ ] Puedo usar `up to, but not including`.
- [ ] Puedo explicar por qué `Math.ceil` conserva la página parcial.
- [ ] Puedo describir la tuple `AssetPage`.
- [ ] Puedo trazar página 2 con tamaño 3.
- [ ] Puedo identificar los tres registros de la segunda página.
- [ ] Puedo diferenciar `slice` de `splice`.
- [ ] Puedo explicar que `slice` hace una copia superficial.
- [ ] Puedo explicar que `readonly` no congela datos en runtime.
- [ ] Puedo diferenciar argumentos inválidos de página fuera de rango.
- [ ] Puedo explicar el resultado del input vacío.
- [ ] Puedo explicar que una página vacía no demuestra que todo el conjunto esté vacío.
- [ ] Puedo justificar `O(k)` de tiempo.
- [ ] Puedo justificar `O(k)` de espacio.
- [ ] Puedo explicar la necesidad de orden estable entre solicitudes.
- [ ] Puedo formular una pregunta sobre la política fuera de rango.
- [ ] Puedo dar una actualización de debugging sobre un error off-by-one.
- [ ] Puedo formular un comentario de code review colaborativo.
- [ ] Puedo responder la pregunta técnica durante 75–90 segundos.
- [ ] Puedo adaptar mi presentación profesional sin memorizarla.
- [ ] Puedo convertir una experiencia real en una respuesta STAR sin revelar datos privados.
- [ ] Completé el ejercicio guiado sin `any`, assertions injustificadas ni `splice`.
- [ ] Envié al chat al menos una respuesta oral o escrita.

---

## 19. Resumen de progreso esperado y próximo tema

### Después de practicar deberías poder

- explicar paginación con arrays y tuples tipadas;
- convertir una página basada en uno a índices basados en cero;
- describir un rango con inicio incluido y fin excluido;
- usar `slice` sin mutar el input;
- diferenciar `slice` de `splice`;
- calcular páginas totales con `Math.ceil`;
- distinguir página parcial, vacía y fuera de rango;
- separar argumentos inválidos de resultado vacío;
- trazar una solicitud de página paso a paso;
- justificar `O(k)` de tiempo y espacio;
- comunicar la necesidad de orden estable;
- limitar conclusiones derivadas de una página;
- hablar en stand-ups, debugging y code review;
- responder una pregunta técnica de entrevista;
- adaptar experiencia real a una respuesta conductual;
- recordar que TypeScript no sustituye la validación runtime.

### Frases que conviene conservar

```text
The page number is one-based, while array indexes are zero-based.
The start index is inclusive and the end index is exclusive.
Slice returns a new array without mutating the input.
Math.ceil keeps the final partial page.
Invalid arguments return undefined under this contract.
An out-of-range page returns an empty item array under this contract.
An empty page does not prove that the complete collection is empty.
The time and additional space are O(k) for k copied items.
The function preserves the current input order.
Reliable pagination across calls requires a stable source order.
```

### Próximo tema condicionado a práctica real

Si no envías práctica ni confirmas que realizaste esta sesión, la siguiente continuará dentro de **Lección técnica 2 — Arrays y tuplas** con otra actividad nueva y no idéntica. Crear otro archivo no contará como dominio.

Después de practicar y recibir correcciones, la siguiente decisión podrá ser:

- una evaluación integradora final de arrays, tuples y typed collections; o
- avanzar a **Lección técnica 3 — Objetos y modelado básico de datos**.

### Cómo solicitar correcciones

Envía audio o texto en este chat con una etiqueta:

```text
Sesión 21 — R1
Sesión 21 — V1
Sesión 21 — P1
Sesión 21 — G1
Sesión 21 — C1
Sesión 21 — C2
Sesión 21 — S1
Sesión 21 — D1
Sesión 21 — CR1
Sesión 21 — RC1
Sesión 21 — L1
Sesión 21 — I1
Sesión 21 — PP1
Sesión 21 — B1
Sesión 21 — PJ1
Sesión 21 — E1
Sesión 21 — F1
Sesión 21 — T1
Sesión 21 — MI1
Sesión 21 — Corrección
```

Primero se atenderá tu respuesta. Se identificarán errores importantes de gramática, vocabulario, ortografía, fluidez y comunicación técnica; se ofrecerá una versión natural; se explicarán brevemente las correcciones clave; y se darán de tres a cinco expresiones reutilizables. La pronunciación solo se comentará cuando el audio sea realmente accesible.
