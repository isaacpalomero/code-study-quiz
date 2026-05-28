---
name: code-study-quiz
description: >
  Use when the user explicitly asks to build a study quiz bank from a codebase,
  a specific development (Jira ticket, current branch, spec doc, PR/MR), or a
  free-form topic; or when the user asks to RUN an existing quiz bank
  interactively. Triggers: "quiz de estudio", "study quiz", "build quiz on this
  repo", "quiz on this ticket", "quiz on this branch", "ejecútame el quiz",
  "run quiz NN".
when_to_use: >
  "code-study-quiz", "study quiz", "quiz de estudio", "banco de preguntas",
  "ejecútame el quiz", "run quiz", "build a quiz", "quiz on ticket",
  "quiz on branch", "quiz on spec", "MCQ study bank", "pre-lectura"
disable-model-invocation: true
---

# code-study-quiz

Build and run **self-study MCQ banks** grounded in a real codebase and its
functional documentation. Two modes: **BUILD** (orchestrates generation of one
or more `.md` quiz files) and **RUN** (asks an existing bank interactively and
scores).

This skill is a **thin orchestrator** over the superpowers stack. It encodes
the _quiz domain_ (anatomy, levels, categories, quality rubric, lessons
learned) and delegates the _process_ to existing skills.

## When to use

| Situation                                                      | Mode            |
| -------------------------------------------------------------- | --------------- |
| User wants a study bank on a whole repo or several subsystems  | BUILD (scope A) |
| User wants a quiz scoped to a Jira ticket / branch / spec / PR | BUILD (scope B) |
| User describes a free-form topic to be quizzed on              | BUILD (scope C) |
| User says _"ejecútame el quiz NN"_ / _"run quiz X"_            | RUN             |

Do **not** auto-trigger — `disable-model-invocation: true`. Invoke only when
the user explicitly asks.

## Mandatory rules (non-negotiable)

All BUILD outputs must comply with `references/quality-rubric.md`:

- Only MCQ, 4 options, exactly ONE unambiguous correct answer.
- Pre-lectura **autocontenida** (answerable without opening source/docs).
- (Técnica) snippets are **real code** with `// …` marking any omission of
  non-contiguous lines.
- Correct-letter distribution balanced (≥⌊N/4⌋−1 of each in files of N≥40).
- No "todas/ninguna de las anteriores"; no double negatives; no absolutes.
- No two questions testing the same fact (even inverted).
- Levels grouped strictly Básico → Intermedio → Avanzado; sequential IDs.

Full anatomy template: `references/question-anatomy.md`.

## BUILD — orchestration

**REQUIRED SUB-SKILL:** Use superpowers:brainstorming, then
superpowers:writing-plans, then superpowers:subagent-driven-development.

1. **Scoping** — invoke `superpowers:brainstorming` injecting the questionnaire
   from `references/build-questionnaire.md`. The FIRST question is the
   **scope** (A: whole repo · B: specific development · C: free-form topic);
   subsequent questions depend on it. Output: spec at
   `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`.

2. **Planning** — invoke `superpowers:writing-plans` using
   `references/implementer-prompt.md` as the per-task template. Produces a
   plan with one task per quiz file + a scaffold task (README + dir) + a
   final QA task.

3. **Execution** — invoke `superpowers:subagent-driven-development`. Per task:
   - Implementer subagent generates the file using
     `references/implementer-prompt.md`. **It does NOT commit.**
   - The controller commits in the target repo / nested sub-repo, using
     `dangerouslyDisableSandbox: true` only on `git commit` (sandbox blocks
     `.git/.../index.lock` otherwise).
   - Reviewer subagent uses `references/reviewer-prompt.md` for a **combined**
     spec-compliance + question-quality review (one pass, not two — lesson
     learned in MCB).
   - Fix loop until approved.

4. **Closure** — update the bank's `README.md` index with real counts.

Defaults (override in brainstorming if needed): language Spanish; per
subsystem-file 40 questions = 16 Básico / 14 Intermedio / 10 Avanzado, with
24 Técnica / 10 Funcional / 6 Mixta; cross-cutting functional file ~30 (in
scope A only). Scopes B and C usually produce a single file of 15–25
questions.

## RUN — interactivity

**REQUIRED SUB-PROTOCOL:** Follow `references/run-protocol.md`.

Given a `.md` bank (produced by BUILD or any file conforming to the anatomy):

1. Parse the file (regex over the anatomy: header line, options A–D, the
   correct letter inside `<details>`).
2. For each selected question, show the pre-lectura inline, **randomize the
   position** of the correct option (deterministic: `q_index × 7 mod 4` →
   A/B/C/D), call `AskUserQuestion` with 4 options, record the response.
3. At the end: show `X/N`, list each question with ✓/✗ and the explanation
   from `<details>`. If score < 80 %, suggest a deeper dive on the missed
   topics.

Supports filters: `--level=básico|intermedio|avanzado`,
`--category=técnica|funcional|mixta`, `--from=N --to=M`.

## Quick reference

| Need                               | File                                |
| ---------------------------------- | ----------------------------------- |
| Question template / worked example | `references/question-anatomy.md`    |
| Non-negotiable quality rules       | `references/quality-rubric.md`      |
| BUILD scoping questions            | `references/build-questionnaire.md` |
| Implementer subagent prompt        | `references/implementer-prompt.md`  |
| Reviewer subagent prompt           | `references/reviewer-prompt.md`     |
| RUN protocol                       | `references/run-protocol.md`        |
| Real-world example (this repo)     | `examples/mcb-150q-bank/README.md`  |

## Common mistakes

| Pitfall                                             | Fix                                                              |
| --------------------------------------------------- | ---------------------------------------------------------------- |
| Subagent commits → sandbox blocks `.git/index.lock` | Controller commits; subagent only writes files                   |
| Snippet pastes non-contiguous code as if contiguous | Insert `// …` at every omission; verify `archivo:línea`          |
| Correct letter clusters on B/C                      | Reorder options to balance A/B/C/D (content unchanged)           |
| Two questions test the same fact (or its inverse)   | Replace one with a genuinely different fact from the docs        |
| Pre-lectura forces opening Confluence/code          | Rewrite as didactic prose; the source link is _optional_ support |
