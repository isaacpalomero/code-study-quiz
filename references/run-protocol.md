# RUN Protocol — Interactive Quiz

RUN mode plays an existing quiz bank `.md` interactively, scoring at the end.

## Inputs

- **Bank path**: a `.md` file conforming to `question-anatomy.md`.
- **Optional filters**:
  - `--level=básico|intermedio|avanzado` — only questions of that level.
  - `--category=técnica|funcional|mixta` — only questions of that category.
  - `--from=N --to=M` — question-index range (inclusive, 1-based).

## 1. Parse the bank

Use line-based regex over the canonical anatomy:

| Element                             | Pattern                                                                                            |
| ----------------------------------- | -------------------------------------------------------------------------------------------------- |
| Question header                     | `^### (P-\d+\.\d+) · \[(Básico\|Intermedio\|Avanzado)\] · \((Técnica\|Funcional\|Mixta)\) · (.+)$` |
| Option line                         | `^- ([A-D])\) (.+)$`                                                                               |
| Correct letter (inside `<details>`) | `^\*\*([A-D])\.` on the first non-empty line after `<summary>Respuesta</summary>`                  |

Capture per question: `{id, level, category, topic, pre_lectura_block, options{A,B,C,D}, correct_letter, explanation}`.

`pre_lectura_block` = everything between the header and the first `- A)` line.

If the file does not parse cleanly (e.g. fewer than 4 options, no correct
letter), STOP and report the offending ID — do not silently skip.

## 2. Filter and order

Apply `--level`, `--category`, `--from`, `--to`. Preserve original order
(Básico → Intermedio → Avanzado).

## 3. Ask the questions

For each question `q` at filtered index `i` (starting from 0):

1. **Show the pre-lectura** as plain text in the chat (not inside an
   `AskUserQuestion` option). Include the topic header so the user knows
   which question it is.
2. **Randomize the option order.** Use the deterministic shuffle from
   `code-comprehension-quiz`:
   - Compute `target_position = (i × 7) mod 4` → 0 → A, 1 → B, 2 → C, 3 → D.
   - Place the correct option at `target_position`; place the other three at
     the remaining slots in any stable order.
   - Record the mapping `{user_letter → original_letter}` for scoring.
3. **Call `AskUserQuestion`** with the question's stem and the 4 shuffled
   options. One question per `AskUserQuestion` call.
4. **Record** the user's chosen letter and whether it matches the correct
   position. If the user skips or selects "Other", count as incorrect.

## 4. Score and review

At the end, output the score and a per-question review.

```
## Quiz {bank_name}: X/N

### {id} — {topic}    ✓
Tu respuesta: B   (correcta)

**Explicación:** {original explanation from <details>}

---

### {id} — {topic}    ✗
Tu respuesta: A   (correcta: C)

**Explicación:** {original explanation from <details>}
```

If score < 80 %, append:

```
Áreas a repasar: {list of topics from missed questions}
¿Quieres que profundicemos en alguna?
```

If 100 %, congratulate and offer the next file (if any) or a harder filter.

## Stop conditions

- File does not parse → report and stop; do not partial-run.
- User asks to stop mid-session → score what was answered so far and review.
- User does not respond → mark as skipped and continue.

## Implementation note

The protocol is **stateless across questions** from the user's perspective
(one `AskUserQuestion` per question). The controller keeps the running tally
internally. After completing one bank, do NOT auto-start another — wait for
an explicit request.

The shuffle method must be deterministic per `(bank, index)` so that re-runs
of the same bank in the same order produce the same letter positions
(reproducible study sessions). If the user wants fresh randomization, accept
an explicit `--reshuffle=<seed>` and use `(i × 7 + seed) mod 4`.
