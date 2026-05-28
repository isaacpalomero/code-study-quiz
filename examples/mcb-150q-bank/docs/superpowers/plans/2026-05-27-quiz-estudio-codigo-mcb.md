# Quiz de estudio del código MCB — Plan de implementación

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Generar un banco de quiz markdown (~150 preguntas MCQ, en español, con pre-lectura autocontenida) que enseñe el backend MCB a nivel técnico y funcional.

**Architecture:** Cinco documentos en `docs/local/quiz/` (sub-repo git propio): un README con la rúbrica y un archivo por subsistema. Cada archivo es independiente y lo genera un subagente que lee sus fuentes (código real + Confluence) y verifica cada pre-lectura contra ellas. Dos ejes por pregunta: nivel Bloom (Básico/Intermedio/Avanzado) y categoría (Técnica/Funcional/Mixta).

**Tech Stack:** Markdown con `<details>` para respuestas. Fuentes: repo `mcb-backend` (`db/*.cds`, `srv/**`) y `docs/local/Confluence/**/*.ES.md`. Skills de apoyo: `sap-cap-agent`, `sap-btp-expert-agent`, `sap-advisor-agent`.

**Spec:** `docs/superpowers/specs/2026-05-27-quiz-estudio-codigo-mcb-design.md`

---

## Convenciones comunes (todas las tareas)

**Ubicación de salida:** `docs/local/quiz/` (sub-repo git; commits se hacen DENTRO de ese sub-repo: `cd docs/local/quiz` o `cd docs/local` según dónde esté el `.git` — verificar con `git rev-parse --show-toplevel`).

**Anatomía de cada pregunta** (copiar exactamente esta forma):

````markdown
### P-{archivo}.{n} · [Nivel] · (Categoría) · {tema corto}

**Pre-lectura** — `{ref}`
{contenido autocontenido: snippet de código, o prosa didáctica reescrita}

{enunciado en forma de pregunta}

- A) {opción}
- B) {opción}
- C) {opción}
- D) {opción}

<details><summary>Respuesta</summary>

**{Letra}.** {por qué es correcta y por qué fallan los distractores}
</details>
````

**Reglas no negociables:**
- Solo MCQ, 4 opciones. Nunca "todas/ninguna de las anteriores", sin dobles negaciones.
- **Pre-lectura autocontenida:** se responde sin abrir Confluence ni el código.
  - Técnica → snippet de código real + ref `archivo:línea`.
  - Funcional → prosa didáctica reescrita (puede ser más larga que el original) + enlace Confluence.
  - Mixta → prosa funcional **+** snippet de código del mismo mecanismo.
- **Enlace Confluence:** `PAGE_ID` = número final del nombre del `.md`/carpeta fuente →
  `https://syntax.atlassian.net/wiki/spaces/IOT/pages/{PAGE_ID}/`.
- Distractores plausibles, de longitud/estilo similar a la correcta.
- Verificar cada pre-lectura contra su fuente real ANTES de escribir la pregunta.
- IDs `P-{archivo}.{n}` correlativos por archivo, ordenados Básico → Intermedio → Avanzado.
- Para puntos finos de CAP/BTP/SAP, consultar la skill correspondiente para validar (no inventar).

**Cuotas por archivo de subsistema (01/02/03), ~40 preguntas:**
- Nivel: 16 Básico · 14 Intermedio · 10 Avanzado.
- Categoría: 24 Técnica · 10 Funcional · 6 Mixta.
- Lente CAP-crítico: al menos 3 preguntas (dentro de Técnica/Mixta) sobre mecanismos CAP imprescindibles (draft flows, SAVE handler de composición, `@flow.status`, proyecciones de seguridad).

---

## Task 1: Scaffold + README con rúbrica

**Files:**
- Create: `docs/local/quiz/README.md`

- [ ] **Step 1: Crear el directorio**

Run: `mkdir -p docs/local/quiz`

- [ ] **Step 2: Escribir `docs/local/quiz/README.md`**

Contenido (literal, ajustar solo el índice si cambian nombres):

```markdown
# Quiz de estudio — Backend MCB

Banco de preguntas para estudiar el código y la funcionalidad del backend MCB.
Diseño: `docs/superpowers/specs/2026-05-27-quiz-estudio-codigo-mcb-design.md`.

## Cómo estudiarlo
- **Modo autoguiado:** lee la pre-lectura de cada pregunta, piensa la respuesta y despliega
  `Respuesta` para comprobarla. Las pre-lecturas son autocontenidas: no necesitas abrir el código
  ni Confluence para responder (los enlaces y refs `archivo:línea` son apoyo opcional).
- **Modo interactivo:** pídele al asistente *"ejecútame el quiz NN"*. Leerá ese archivo y te
  preguntará de a una, aleatorizando la posición de la opción correcta, puntuará y repasará los fallos.

## Niveles (Bloom)
- **[Básico]** — recordar: qué es, dónde está, qué tipo.
- **[Intermedio]** — comprender/aplicar: cómo y por qué.
- **[Avanzado]** — analizar/evaluar: trade-offs, fallos, concurrencia.

## Categorías
- **(Técnica)** — sobre el código real.
- **(Funcional)** — sobre regla de negocio/workflow (doc de Confluence).
- **(Mixta)** — cómo el código materializa la regla de negocio.

## Índice
| Archivo | Tema | Nº |
|---------|------|----|
| `00-funcional.md` | Dominio y flujos de negocio (transversal) | ~30 |
| `01-modelo-datos.md` | Esquemas CDS (`db/*.cds`) | ~40 |
| `02-servicios-negocio.md` | admin, planning, reviews, config, masters | ~40 |
| `03-sync-engine.md` | Motor de sync + APIs SAP | ~40 |
```

- [ ] **Step 3: Commit (dentro del sub-repo)**

```bash
cd docs/local/quiz 2>/dev/null; ROOT=$(git rev-parse --show-toplevel); cd "$ROOT"
git add "$(git rev-parse --show-prefix)README.md" 2>/dev/null || git add **/quiz/README.md
git commit -m "docs(quiz): scaffold quiz bank with README rubric"
```

---

## Task 2: `00-funcional.md` — Dominio y flujos de negocio (~30, 100% Funcional)

**Files:**
- Create: `docs/local/quiz/00-funcional.md`

**Cuotas:** 12 Básico · 11 Intermedio · 7 Avanzado. Todas categoría **Funcional** (1–2 Mixtas opcionales si aportan).

**Fuentes (Confluence `.ES.md`, ruta base `docs/local/Confluence/Mining- Maintenance Cost Budgeting--920485900/`):**
- `MCB - Internal documentation--1666940935/MCB - Updated Functional Documentation--1414660104/...ES.md`
- `MCB - Architecture--1227587588/...ES.md`
- `MCB - UATs--1453981700/MCB - UAT-001 - Budget Administrator Flow--1454702593/...ES.md`
- `MCB - Governance--1635549223/MCB - Epics Definition--1353744386/...ES.md`
- `MCB - Data Model--1653309441/MCB - Current Data Model--1635516444/...ES.md`
- `docs/MCB-PROJECT.md` (resumen de workflow y glosario, como apoyo).

**Cobertura mínima de temas:** ciclo de vida de BudgetVersion (New→InProcess→Computed→Submitted→Approved→Closed), roles (Admin/Planner/Owner) y qué puede cada uno, flujo de aprobación/rechazo, reforecast, qué es un Cost Center Budget / Budget Asset / componente de coste (Primary/Secondary/NonCapital), año fiscal y periodos, glosario (YTD, burn rate).

- [ ] **Step 1: Leer spec, README (Task 1) y las fuentes listadas.** Extraer PAGE_ID de cada doc usado.

- [ ] **Step 2: Redactar 30 preguntas** respetando cuotas y cobertura. Cada pre-lectura = prosa didáctica autocontenida + enlace Confluence del doc fuente.

- [ ] **Step 3: Verificar** que cada afirmación funcional coincide con su doc fuente; corregir discrepancias.

- [ ] **Step 4: Checklist de calidad** (autocontención, distractores, sin all/none, IDs correlativos, conteo por nivel).

Ejemplo trabajado (formato a replicar):

````markdown
### P-0.18 · [Avanzado] · (Funcional) · Reenvío de un presupuesto en revisión

**Pre-lectura** _(reescrita para enseñar · [ver en Confluence](https://syntax.atlassian.net/wiki/spaces/IOT/pages/1414660104/))_

> En MCB un Cost Center Budget avanza por una máquina de estados. El planner lo edita mientras está
> en "In Process"; al enviarlo pasa a "Submitted" y se entrega al owner. En "Submitted" queda
> bloqueado para el planner: no puede editarlo ni reenviarlo. Solo el owner decide la siguiente
> transición (aprobar → "Approved", o rechazar → vuelve a "In Process" con comentario).

¿Quién y cómo desbloquea un presupuesto que está en "Submitted"?

- A) El planner, reenviándolo de nuevo
- B) El owner, aprobándolo o rechazándolo
- C) El admin, reactivando la versión
- D) Cualquiera con rol autenticado, editándolo

<details><summary>Respuesta</summary>

**B.** Solo el owner decide la transición desde "Submitted". A está bloqueado, C/D no participan en
esa transición.
</details>
````

- [ ] **Step 5: Escribir** `docs/local/quiz/00-funcional.md`.

- [ ] **Step 6: Commit**

```bash
cd "$(git -C docs/local/quiz rev-parse --show-toplevel)"
git add "$(git -C docs/local/quiz rev-parse --show-prefix)00-funcional.md" 2>/dev/null || git add **/quiz/00-funcional.md
git commit -m "docs(quiz): add 00-funcional question bank"
```

---

## Task 3: `01-modelo-datos.md` — Esquemas CDS (~40)

**Files:**
- Create: `docs/local/quiz/01-modelo-datos.md`

**Cuotas:** 16 Básico · 14 Intermedio · 10 Avanzado · (24 Técnica / 10 Funcional / 6 Mixta) · ≥3 CAP-crítico.

**Fuentes técnicas:** `db/budget.schema.cds`, `db/sync.schema.cds`, `db/config.schema.cds`, `db/common.schema.cds`, `db/reporting.schema.cds`, `db/budget.schema.change-tracking.cds`. (Excluir `db/anomaly.schema.cds` — fuera de alcance.)
**Fuentes funcionales:** `MCB - Data Model--1653309441/MCB - Current Data Model--1635516444/...ES.md`; `docs/MCB-PROJECT.md` (sección Domain Model).
**Skill de apoyo:** `sap-cap-agent` para mecánica de CDS (aspects, `@assert`, composición/asociación, `@flow.status`).

**Cobertura mínima:** aspect `Managed` (createdAt/By, modifiedAt/By + `@cds.on.insert/update`), tipos custom con `@assert.format`/`@assert.range` (Year, CurrencyCode, Percentage, MonetaryAmount), Composition vs Association (assets vs costCenter) y cascade-delete, `@assert.unique` (compound keys), `@mandatory`/`not null`, enums de estado con `@flow.status`, `key UUID`/CUID, code lists.

- [ ] **Step 1: Leer** spec, README, los `db/*.cds` en alcance y la doc de modelo de datos. Consultar `sap-cap-agent` para confirmar la semántica exacta de cualquier anotación dudosa.

- [ ] **Step 2: Redactar 40 preguntas** con las cuotas. Para Técnicas, copiar el snippet CDS real con `archivo:línea`.

- [ ] **Step 3: Verificar** cada snippet contra el `.cds` (línea y contenido). Corregir refs.

- [ ] **Step 4: Checklist de calidad** + conteos por nivel y categoría.

Ejemplo trabajado:

````markdown
### P-1.22 · [Avanzado] · (Técnica) · Cascade-delete por composición

**Pre-lectura** — `db/budget.schema.cds` (entidad `CostCenterBudgets`)
​```cds
entity CostCenterBudgets : cuid, managed {
  assets : Composition of many BudgetAssets on assets.budget = $self;
  costCenter : Association to CostCenters not null @mandatory;
}
​```

Si se borra un `CostCenterBudgets`, ¿qué ocurre con sus `assets` y su `costCenter`?

- A) Se borran los assets; el cost center permanece
- B) Se borran los assets y el cost center
- C) Falla por la FK del cost center
- D) Quedan assets huérfanos; el cost center se borra

<details><summary>Respuesta</summary>

**A.** `Composition` implica cascade-delete (los assets mueren con el budget); `Association` a un
maestro independiente no borra el cost center. B/C/D malinterpretan composición vs asociación.
</details>
````

- [ ] **Step 5: Escribir** `docs/local/quiz/01-modelo-datos.md`.

- [ ] **Step 6: Commit**

```bash
cd "$(git -C docs/local/quiz rev-parse --show-toplevel)"
git add "$(git -C docs/local/quiz rev-parse --show-prefix)01-modelo-datos.md" 2>/dev/null || git add **/quiz/01-modelo-datos.md
git commit -m "docs(quiz): add 01-modelo-datos question bank"
```

---

## Task 4: `02-servicios-negocio.md` — Servicios de negocio (~40)

**Files:**
- Create: `docs/local/quiz/02-servicios-negocio.md`

**Cuotas:** 16 Básico · 14 Intermedio · 10 Avanzado · (24 Técnica / 10 Funcional / 6 Mixta) · ≥3 CAP-crítico (drafts/composición).

**Fuentes técnicas:**
- admin: `srv/admin/admin.srv.cds`, `srv/admin/admin.srv.auth.cds`, `srv/admin/admin.srv.ts`, `srv/admin/handlers/{versionCrud,budgetCrud,confirmVersion,activateVersion,closeVersion,reforecastVersion}.handler.ts`, `srv/admin/domains/costCenterBudget.domain.ts`.
- planning: `srv/planning/planning.srv.{cds,auth.cds,ts}`, `srv/planning/handlers/{computeBudget,submitBudget,editBudget,budgetAssetCrud,budgetAssetUtilizationCrud,getBudgetMaterials}.handler.ts`, `srv/planning/domains/budgetVersion.domain.ts`.
- reviews: `srv/reviews/reviews.srv.{cds,auth.cds,ts}`, `srv/reviews/handlers/{reviewBudget,reviewCommentCrud}.handler.ts`.
- config: `srv/config/config.srv.{cds,auth.cds,ts}`, `srv/config/handlers/*.ts`.
- masters: `srv/masters/masters.srv.{cds,auth.cds,ts}`, `srv/masters/handlers/getCostCentersForFiscalYear.handler.ts`.

**Fuentes funcionales:** UAT-001 (`...1454702593...ES.md`), Updated Functional Documentation (`...1414660104...ES.md`).
**Skill de apoyo:** `sap-cap-agent` para drafts, SAVE handlers de composición, `@requires`/`@restrict`.

**Cobertura mínima:** autorización por servicio (`@requires` roles Admin/Planner/Owner) y por entidad (`@restrict` con `where`), proyecciones con filtro (PlanningService oculta `status = New`), acciones bound/unbound (createBudgetVersion, computeBudget, submitBudget, approve/rejectBudget, activate/close/reforecast), RolePolicy + `assertForUser`, máquina de estados `@flow.status`, **drafts** (por qué CREATE/UPDATE de hijos de composición pasa por el SAVE de la raíz), `getOrThrow` como precondición.

- [ ] **Step 1: Leer** spec, README, los ficheros de servicio listados y la doc funcional. Consultar `sap-cap-agent` para el modelo de drafts/composición.

- [ ] **Step 2: Redactar 40 preguntas** con cuotas. Mixtas: enlazar regla de negocio (p. ej. "solo el owner aprueba") con el handler/RolePolicy real.

- [ ] **Step 3: Verificar** snippets contra los `.ts/.cds` reales (archivo:línea).

- [ ] **Step 4: Checklist de calidad** + conteos.

Ejemplo trabajado:

````markdown
### P-2.31 · [Avanzado] · (Mixta) · Por qué editar un asset pasa por el SAVE de la raíz

**Pre-lectura funcional** _(reescrita · [ver en Confluence](https://syntax.atlassian.net/wiki/spaces/IOT/pages/1414660104/))_
> El planner edita un presupuesto como un borrador (draft): añade/modifica activos y costes y, al
> terminar, guarda. Los cambios no se confirman pieza a pieza, sino al guardar el presupuesto completo.

**Pre-lectura técnica** — `srv/planning/handlers/budgetAssetCrud.handler.ts`
​```ts
// Los CRUD de hijos de composición se despachan en el SAVE de la raíz (draft),
// no como CREATE/UPDATE directos sobre BudgetAssets.
​```

¿Por qué no se permite un `CREATE` directo contra `BudgetAssets` en planning?

- A) BudgetAssets es read-only
- B) Al ser entidad draft de composición, las mutaciones de hijos van por el SAVE de la raíz
- C) Falta el rol BudgetPlanner para CREATE
- D) La proyección no expone BudgetAssets

<details><summary>Respuesta</summary>

**B.** En entidades draft, los hijos de composición se mutan a través del SAVE del root; un CREATE
directo no forma parte del flujo de draft. A/C/D no son la causa.
</details>
````

- [ ] **Step 5: Escribir** `docs/local/quiz/02-servicios-negocio.md`.

- [ ] **Step 6: Commit**

```bash
cd "$(git -C docs/local/quiz rev-parse --show-toplevel)"
git add "$(git -C docs/local/quiz rev-parse --show-prefix)02-servicios-negocio.md" 2>/dev/null || git add **/quiz/02-servicios-negocio.md
git commit -m "docs(quiz): add 02-servicios-negocio question bank"
```

---

## Task 5: `03-sync-engine.md` — Motor de sync + APIs SAP (~40)

**Files:**
- Create: `docs/local/quiz/03-sync-engine.md`

**Cuotas:** 16 Básico · 14 Intermedio · 10 Avanzado · (24 Técnica / 10 Funcional / 6 Mixta) · ≥3 CAP/BTP-crítico.

**Fuentes técnicas:** `srv/sync/syncers/base.syncer.ts`, `srv/sync/syncers/batch.syncer.ts`, `srv/sync/lib/{relations,resilience,retry,syncSummary,translations,utils}.ts`, `srv/sync/handlers/{syncEntity,syncAll,syncNow,syncScheduleCrud}.handler.ts`, `srv/sync/domains/sync.domain.ts`, `srv/sync/jobScheduler/jobScheduler.client.ts`, y un par de syncers concretos como ejemplo (`costCenter`, `equipment`, `actualCosts`: client+mapper+syncer), `db/sync.schema.cds`.
**Fuentes funcionales:** Architecture (`...1227587588...ES.md`), Job scheduler (`...1464205316...ES.md`), API compatibility / Private Cloud (`...1745420292...ES.md`), `docs/MCB-PROJECT.md` (External SAP APIs).
**Skills de apoyo:** `sap-btp-expert-agent` (destinos, conectividad, job scheduler en BTP), `sap-advisor-agent` (semántica de las APIs OData de SAP).

**Cobertura mínima:** patrón BaseSyncer (upsert, `syncInBatch`, troceo en lotes de 1000 y por qué), grafo de dependencias entre syncers (CostCenters→[ControllingAreas,CompanyCodes]; Equipment→[FunctionalLocations,EquipmentTypes]; ActualCosts→[Equipment,CostCenters]), separación client/mapper/syncer, paginación (`fetchWithPagination`), resiliencia (retry con backoff, 3 intentos/1s), delta vs full sync, `generateId`/`asRelationKey`, mappers puros, las 16 APIs (qué entidad sincroniza cada una).

- [ ] **Step 1: Leer** spec, README, los ficheros de sync listados y la doc de arquitectura/scheduler. Consultar `sap-btp-expert-agent`/`sap-advisor-agent` para detalles BTP/OData.

- [ ] **Step 2: Redactar 40 preguntas** con cuotas.

- [ ] **Step 3: Verificar** snippets contra los `.ts` reales (archivo:línea) y la tabla de APIs.

- [ ] **Step 4: Checklist de calidad** + conteos.

Ejemplo trabajado:

````markdown
### P-3.09 · [Intermedio] · (Técnica) · Tamaño de lote en BaseSyncer

**Pre-lectura** — `srv/sync/syncers/base.syncer.ts` (método de sincronización por lotes)
​```ts
for (const chunk of chunks(ids, 1000)) {
  await UPSERT.into(this.entity).entries(await this.fetchPage(chunk))
}
​```

¿Cuál es la razón principal de procesar en lotes de 1000?

- A) Es el límite de filas de SQLite
- B) Acotar memoria y tamaño de payload por página de la API OData
- C) Es el máximo de UPSERT de CAP
- D) Coincide con el número de cost centers

<details><summary>Respuesta</summary>

**B.** El troceo limita memoria y respeta el tamaño de página configurado para la API OData. A/C/D
no son límites reales del sistema.
</details>
````

- [ ] **Step 5: Escribir** `docs/local/quiz/03-sync-engine.md`.

- [ ] **Step 6: Commit**

```bash
cd "$(git -C docs/local/quiz rev-parse --show-toplevel)"
git add "$(git -C docs/local/quiz rev-parse --show-prefix)03-sync-engine.md" 2>/dev/null || git add **/quiz/03-sync-engine.md
git commit -m "docs(quiz): add 03-sync-engine question bank"
```

---

## Task 6: Cierre — índice y QA global

**Files:**
- Modify: `docs/local/quiz/README.md` (actualizar nº reales por archivo en el índice)

- [ ] **Step 1: Contar** preguntas reales por archivo y por nivel/categoría:

Run: `for f in docs/local/quiz/0*.md; do echo "$f: $(grep -c '^### P-' "$f")"; done`
Expected: 00≈30, 01≈40, 02≈40, 03≈40.

- [ ] **Step 2: Verificar IDs únicos y correlativos** por archivo (sin saltos ni duplicados).

Run: `for f in docs/local/quiz/0*.md; do echo "== $f =="; grep -o 'P-[0-9]*\.[0-9]*' "$f" | sort -t. -k2 -n | uniq -d; done`
Expected: sin salida (ningún duplicado).

- [ ] **Step 3: Verificar autocontención y enlaces:** que ninguna pregunta dependa de abrir el código/Confluence para responder; que cada enlace Confluence tenga la forma `…/pages/{PAGE_ID}/`.

- [ ] **Step 4: Actualizar** la tabla de índice del README con los conteos reales.

- [ ] **Step 5: Commit**

```bash
cd "$(git -C docs/local/quiz rev-parse --show-toplevel)"
git add "$(git -C docs/local/quiz rev-parse --show-prefix)README.md" 2>/dev/null || git add **/quiz/README.md
git commit -m "docs(quiz): finalize index and global QA"
```

---

## Notas de ejecución

- Tareas 2–5 son **independientes** entre sí (cada una crea un archivo distinto) → aptas para subagentes en paralelo tras la Task 1.
- Cada subagente debe leer: este plan (sus convenciones comunes + su Task), el spec, y el README (Task 1).
- Los commits van al **sub-repo `docs/local`** (no al repo `mcb-backend` padre).
