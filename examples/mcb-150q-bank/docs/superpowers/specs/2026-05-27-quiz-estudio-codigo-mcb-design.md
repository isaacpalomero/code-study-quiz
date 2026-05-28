# Quiz de estudio del código MCB — Diseño

> **Estado:** Diseño en revisión (v3) · **Fecha:** 2026-05-27 · **Autor:** Isaac Palomero (con Claude)
> **Objetivo:** Material de estudio para ganar conocimiento profundo del backend MCB —técnico **y
> funcional**— mediante preguntas de opción múltiple con pre-lectura, en español.

---

## 1. Propósito y criterios de éxito

Crear un **banco de preguntas versionado** que sirva como material de autoestudio del repositorio
`mcb-backend` y, simultáneamente, como fuente para sesiones de quiz interactivas.

**Éxito =** una persona que complete el banco entiende, con respaldo en el código real **y en la
documentación funcional**:

- Cómo está modelado el dominio en CDS (entidades, tipos, restricciones, workflows de estado).
- Cómo funcionan los servicios de negocio (handlers, acciones, proyecciones, autorización, estados).
- Cómo opera el motor de sincronización con las APIs OData de SAP.
- **El "qué" y el "por qué" funcional**: ciclo de vida del presupuesto, roles, aprobación, governance.
- Los **mecanismos de CAP críticos** para entender el código (p. ej. flujos de draft, SAVE handlers
  de composición), cuando son imprescindibles.

**No-objetivos (YAGNI):**

- No enseña programación genérica ni SAP CAP en abstracto: cada pregunta es específica del MCB.
- No cubre (en esta iteración) el motor de anomalías, `srv/lib` utilidades, apps Fiori ni testing.
  Quedan como extensiones futuras (`05-…`, `06-…`).

## 2. Alcance

| Incluido | Subsistema | Fuentes principales |
|----------|-----------|---------------------|
| ✅ | Funcional / dominio (transversal) | `docs/local/Confluence/…` (ES); UATs, Updated Functional Documentation, Architecture, Governance |
| ✅ | Modelo de datos (CDS) | `db/*.cds` (budget, sync, config, common, reporting) + Data Model en Confluence |
| ✅ | Servicios de negocio | `srv/admin`, `srv/planning`, `srv/reviews`, `srv/config`, `srv/masters` |
| ✅ | Motor de sync + APIs SAP | `srv/sync/` (BaseSyncer, clients, mappers, syncers, orquestación) |
| ❌ | Motor de anomalías | `srv/analytics/anomaly` (extensión futura) |
| ❌ | Utilidades / Fiori / Testing | `srv/lib`, `app/`, `*.test.ts` (extensión futura) |

## 3. Entregables y estructura de archivos

Ruta: **`docs/local/quiz/`** (sub-repo git propio).

```
docs/local/quiz/
├── README.md                  # índice, leyenda de niveles/categorías, instrucciones modo interactivo
├── 00-funcional.md            # ~30 preguntas funcionales transversales (flujos cross-subsistema)
├── 01-modelo-datos.md         # ~40 preguntas
├── 02-servicios-negocio.md    # ~40 preguntas
└── 03-sync-engine.md          # ~40 preguntas
```

Total objetivo: **~150 preguntas** (~120 en subsistemas + ~30 funcionales transversales).

## 4. Dos ejes ortogonales por pregunta

Cada pregunta lleva **dos etiquetas independientes**: su **nivel** (Bloom) y su **categoría** (fuente/enfoque).

### 4.1 Nivel — taxonomía de Bloom

| Nivel | Bloom | Evalúa |
|-------|-------|--------|
| **[Básico]** | Recordar | Qué es, dónde está, qué tipo/aspecto |
| **[Intermedio]** | Comprender / Aplicar | Cómo y por qué funciona; aplicar a un caso |
| **[Avanzado]** | Analizar / Evaluar | Trade-offs, modos de fallo, concurrencia, alternativas |

**Distribución por archivo:** 40 % Básico · 35 % Intermedio · 25 % Avanzado.

### 4.2 Categoría — fuente/enfoque

| Categoría | Pre-lectura (autocontenida) | Enfoque |
|-----------|-----------------------------|---------|
| **Técnica** | Fragmento de código real (`archivo:línea`) | Cómo está implementado |
| **Funcional** | Prosa didáctica reescrita de la doc + enlace Confluence | Regla de negocio, workflow, decisión de dominio |
| **Mixta** | Prosa didáctica funcional **+** código real | Cómo el código materializa la regla de negocio |

**Distribución por archivo de subsistema (01/02/03):** 60 % Técnica · 25 % Funcional · 15 % Mixta.
El archivo `00-funcional.md` es 100 % Funcional (con algún Mixto si aporta), repartido en los 3 niveles Bloom.

### 4.3 Lente "CAP-crítico"

Dentro de las categorías Técnica/Mixta, el agente generador **identifica activamente** los mecanismos
de SAP CAP cuya comprensión es imprescindible para entender el código del MCB (p. ej. **flujos de
draft**, bloqueo de CREATE/UPDATE en hijos de composición y su SAVE handler raíz, `@flow.status`,
proyecciones con filtro de seguridad) y crea preguntas sobre ellos. No es un archivo aparte: es una
lente que se aplica donde el código lo requiera.

## 5. Anatomía de cada pregunta

- **Solo MCQ**, 4 opciones (A–D). Nunca preguntas abiertas.
- **Pre-lectura obligatoria y AUTOCONTENIDA:** el estudiante debe poder responder leyendo solo el
  documento de quiz, **sin abrir Confluence ni el código fuente**.
  - **Técnica:** fragmento de código real, con `archivo:línea` como referencia.
  - **Funcional:** **prosa didáctica reescrita** de la sección relevante (puede ser más extensa que
    el original, orientada a enseñar, no un copy-paste), **+ enlace a Confluence** como referencia.
  - Nunca inventada: toda pre-lectura se ancla a una fuente real (código o documento).
- **Enlace Confluence:** se construye con el ID (número final del nombre del `.md`/carpeta fuente):
  `https://syntax.atlassian.net/wiki/spaces/IOT/pages/{PAGE_ID}/`.
- **Respuesta colapsable** (`<details>`) con: letra correcta, por qué lo es y por qué fallan los distractores.
- **Cabecera con identificador + nivel + categoría**: `P-{archivo}.{n} · [Nivel] · (Categoría) · tema`.

Plantilla canónica (ejemplo de pregunta **Mixta**):

```markdown
### P-2.07 · [Avanzado] · (Mixta) · Submit de presupuesto y máquina de estados

**Pre-lectura funcional** _(reescrita para enseñar · [ver en Confluence](https://syntax.atlassian.net/wiki/spaces/IOT/pages/1414660104/))_

> En MCB, el presupuesto de un centro de coste (Cost Center Budget) avanza por una máquina de
> estados. El **planner** trabaja sobre él mientras está en **"In Process"**: ahí puede añadir
> activos, costes y usos. Cuando lo considera listo, ejecuta la acción de **envío a aprobación**,
> que mueve el estado a **"Submitted"** y lo entrega al **owner** para su revisión. Desde
> "Submitted", el budget queda **bloqueado para edición** del planner y el envío no puede repetirse:
> la siguiente transición válida la decide el owner (aprobar/rechazar). Esta regla evita que un
> presupuesto en revisión se altere o se reenvíe por error.

**Pre-lectura técnica** — `srv/planning/planning.srv.ts` (acción `submitBudget`)
​```ts
await CostCenterBudgetRolePolicy.for(budget).canBeSubmitted.assertForUser(req.user)
await UPDATE(CostCenterBudgets, id).with({ status_code: 'Submitted' })
​```

¿Qué garantiza que un planner no pueda reenviar un budget ya "Submitted"?

- A) El frontend oculta el botón de enviar
- B) La RolePolicy `canBeSubmitted` rechaza el envío si el estado no es "In Process"
- C) La proyección de PlanningService oculta los budgets Submitted
- D) Una restricción @assert.unique en el estado

<details><summary>Respuesta</summary>

**B.** La transición se valida en servidor vía RolePolicy contra el estado actual; A es cosmético,
C afecta a visibilidad pero no impediría la acción directa, D no aplica a transiciones de estado.
</details>
```

## 6. Fuentes, agentes y skills

- **Código:** el repo `mcb-backend` (la pre-lectura técnica se extrae y verifica del código actual).
- **Funcional:** `docs/local/Confluence/Mining- Maintenance Cost Budgeting--920485900/` — preferir las
  versiones **`.ES.md`** (quiz en español). Documentos clave: *Updated Functional Documentation*,
  *Architecture*, *Current Data Model*, *Configuration*, *Governance*, *UATs*. La pre-lectura se
  **reescribe en prosa didáctica autocontenida** (no copy-paste) y se enlaza a Confluence usando el
  ID (número final del nombre): `https://syntax.atlassian.net/wiki/spaces/IOT/pages/{PAGE_ID}/`.
- **Skills de apoyo** para casos muy específicos de SAP/BTP/CAP (redactar y **validar** la pregunta,
  no inventar): `/genai-studio-agents:sap-cap-agent` (mecánica CAP/CDS), `/genai-studio-agents:sap-btp-expert-agent`
  (BTP, MTA, destinos, XSUAA), `/genai-studio-agents:sap-advisor-agent` (módulos/comportamiento SAP).
- **Modo interactivo:** al pedir *"ejecútame el quiz NN"*, el asistente lee ese `.md` y formula las
  preguntas de a una vía `AskUserQuestion`, **aleatorizando la posición de la opción correcta**,
  puntúa y revisa los fallos al final. El markdown es siempre la fuente de la verdad.

## 7. Criterios de calidad

- Distractores plausibles, de longitud y estilo gramatical similar a la respuesta correcta.
- Sin "todas/ninguna de las anteriores", sin dobles negaciones, sin absolutos engañosos.
- Cada pre-lectura verificada contra su fuente real (código o documento) en el momento de generar.
- **Pre-lectura autocontenida:** se puede responder sin abrir Confluence ni el código; el enlace y la
  referencia `archivo:línea` son apoyo opcional, no requisito para responder.
- Preguntas Mixtas: el extracto funcional y el de código deben referirse al mismo mecanismo.
- Cada pregunta enseña algo concreto y verificable del MCB.
- Cobertura amplia: repartir entre los ficheros/conceptos del subsistema, no concentrar en uno solo.

## 8. Riesgos y mitigaciones

| Riesgo | Mitigación |
|--------|-----------|
| Referencias `archivo:línea` se desactualizan al evolucionar el código | Citar también el símbolo (entidad/función); la línea es orientativa |
| Doc de Confluence desactualizada vs código | En preguntas Mixtas, el código manda; señalar discrepancias si las hubiera |
| Distractores triviales que delatan la respuesta | Checklist de calidad (§7); homogeneizar longitud/estilo |
| Volumen alto (~150) → generación larga | Generar por archivo (00→01→02→03), revisable incrementalmente |
| Skills SAP/BTP devuelven info genérica | Usarlas solo para validar puntos finos; anclar siempre la pregunta al MCB |
```
