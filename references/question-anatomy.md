# Question Anatomy — Canonical Template

Every question in a `code-study-quiz` bank conforms to this anatomy. Deviating
from it breaks both human study (collapsible reveal) and the RUN-mode parser
(regex-driven).

## Header

```
### P-{f}.{n} · [Nivel] · (Categoría) · {tema corto}
```

- `{f}` — file index. For per-subsystem files use `1`, `2`, `3`, … and for a
  cross-cutting functional file use `0`.
- `{n}` — sequential question number within the file, starting at `1`. IDs are
  ordered **strictly Básico → Intermedio → Avanzado**; no interleaving.
- `[Nivel]` — one of `[Básico]` / `[Intermedio]` / `[Avanzado]` (Bloom).
- `(Categoría)` — one of `(Técnica)` / `(Funcional)` / `(Mixta)`.

## Pre-lectura

Always present. Always **self-contained** (the student must be able to answer
without opening any other file).

### Técnica

A real code fragment copied from the source file. Reference line:

````markdown
**Pre-lectura** — `path/to/file.ext:Lstart-Lend`

​`{lang}
{real code; insert "// …" wherever lines are omitted}
​`
````

### Funcional

Didactic prose rewritten from the source document (may be longer than the
original — its purpose is to **teach**, not to quote). The link to the source
is optional support, **never required to answer**.

```markdown
**Pre-lectura** _(reescrita para enseñar · [ver fuente]({SOURCE_URL}))_

> Párrafo(s) didácticos autocontenidos en prosa.
```

### Mixta

Both: a brief functional rewrite + the relevant code excerpt.

````markdown
**Pre-lectura funcional** _(reescrita · [ver fuente]({SOURCE_URL}))_

> Prosa breve que explica la regla / decisión de negocio.

**Pre-lectura técnica** — `path/to/file.ext:Lstart-Lend`
​``{lang}
{snippet real con `// …` para omisiones}
​``
````

## Stem and options

A single, clear question (not compound), followed by 4 options labelled A–D.

```markdown
{Pregunta en una sola oración interrogativa.}

- A) {opción}
- B) {opción}
- C) {opción}
- D) {opción}
```

Constraints:

- Exactly 4 options. No "todas/ninguna de las anteriores".
- All options of similar length / grammar / register. The correct one must not
  stand out by phrasing, specificity, length, or grammatical fit.
- No double negatives in the stem; no absolutes ("siempre", "nunca") unless
  factually grounded.

## Answer block

```markdown
<details><summary>Respuesta</summary>

**{Letra}.** {Por qué la correcta lo es y por qué falla cada distractor.}

</details>
```

The bold letter is the canonical position the RUN parser reads. The
explanation must touch each distractor, not just the correct option.

## Worked example (Mixta, Avanzado)

````markdown
### P-2.31 · [Avanzado] · (Mixta) · Edición de hijos de composición vía SAVE del raíz

**Pre-lectura funcional** _(reescrita · [ver en Confluence](https://syntax.atlassian.net/wiki/spaces/IOT/pages/1414660104/))_

> En MCB el planner edita un presupuesto como un borrador (draft). Añade y modifica
> activos y costes; los cambios no se confirman pieza a pieza, sino al guardar el
> presupuesto completo.

**Pre-lectura técnica** — `srv/planning/handlers/budgetAssetCrud.handler.ts:Lstart-Lend`
​`ts
// Los CRUD de hijos de composición se despachan en el SAVE del root (draft),
// no como CREATE/UPDATE directos sobre BudgetAssets.
// … (resto del handler)
​`

¿Por qué no se permite un `CREATE` directo contra `BudgetAssets` en planning?

- A) BudgetAssets es read-only
- B) Al ser entidad draft de composición, las mutaciones de hijos van por el SAVE del root
- C) Falta el rol BudgetPlanner para CREATE
- D) La proyección no expone BudgetAssets

<details><summary>Respuesta</summary>

**B.** En entidades draft, los hijos de composición se mutan a través del SAVE del root;
un CREATE directo no forma parte del flujo de draft. A/C/D no son la causa.

</details>
````

## Worked example (Funcional, Intermedio)

```markdown
### P-0.18 · [Intermedio] · (Funcional) · Desbloqueo de un presupuesto en revisión

**Pre-lectura** _(reescrita para enseñar · [ver en Confluence](https://syntax.atlassian.net/wiki/spaces/IOT/pages/1414660104/))_

> Un Cost Center Budget avanza por una máquina de estados. El planner lo edita
> mientras está en "In Process"; al enviarlo pasa a "Submitted" y se entrega al
> owner. En "Submitted" queda bloqueado para el planner: solo el owner decide
> la siguiente transición (aprobar → "Approved", o rechazar → vuelve a
> "In Process" con comentario).

¿Quién y cómo desbloquea un presupuesto en estado "Submitted"?

- A) El planner reenviándolo de nuevo
- B) El owner aprobándolo o rechazándolo
- C) El admin reactivando la versión
- D) Cualquier rol autenticado editándolo

<details><summary>Respuesta</summary>

**B.** Solo el owner decide la transición desde "Submitted". A está bloqueado,
C/D no participan en esa transición.

</details>
```
