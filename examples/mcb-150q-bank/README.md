# Example — MCB 150-question study bank

The MCB project is the first real bank generated with this skill (the pilot
that produced it). Rather than duplicating the 150 questions here, this
example points to the live bank, which lives in a separate sub-repo:

**Bank location:** `docs/local/quiz/` (sub-repo `docs/local`, branch `master`)

```
docs/local/quiz/
├── README.md                  rubric + index (Bloom levels, T/F/M categories)
├── 00-funcional.md            30 questions · 12 B / 11 I / 7 A · 100 % Funcional
├── 01-modelo-datos.md         40 questions · 16 B / 14 I / 10 A · 24 T / 10 F / 6 M
├── 02-servicios-negocio.md    40 questions · 16 B / 14 I / 10 A · 24 T / 10 F / 6 M
└── 03-sync-engine.md          40 questions · 16 B / 14 I / 10 A · 24 T / 10 F / 6 M
                                                            Total: 150
```

## What this example demonstrates

- **Scope A** (whole repo): 4 output files, one cross-cutting functional plus
  three per-subsystem (data model, business services, sync engine).
- **Mixed categories**: pure-functional file plus 60/25/15 split T/F/M in
  the subsystem files.
- **Self-contained pre-lecturas** with optional Confluence links built from
  the `IOT` space key + the `PAGE_ID` extracted from doc filenames.
- **Snippet fidelity** with `// …` markers wherever lines are omitted (added
  in iteration after the first reviewer caught condensed-as-contiguous
  presentations).
- **Letter-balanced answers** (A/B/C/D ≈ 25 % each per file after rebalancing
  iteration).
- **Controller-driven commits** in the nested sub-repo `docs/local`, with
  `dangerouslyDisableSandbox: true` only on `git commit`.

## Pilot history (for context)

The pilot was generated across one session via:
`superpowers:brainstorming` → `superpowers:writing-plans` →
`superpowers:subagent-driven-development`. The spec and plan are at:

- `docs/superpowers/specs/2026-05-27-quiz-estudio-codigo-mcb-design.md`
- `docs/superpowers/plans/2026-05-27-quiz-estudio-codigo-mcb.md`

The lessons that ended up baked into the skill (especially in
`references/quality-rubric.md`):

1. Answer-letter balance must be enforced from the first draft, not patched
   after review.
2. Snippets must use `// …` for any omission — non-contiguous code presented
   as contiguous is a fidelity defect.
3. Two questions phrased as inverse of each other (e.g. "in which state can
   you delete X?" / "which states block deletion of X?") test the same atomic
   fact — replace one.
4. Spec compliance + question quality can be reviewed in a single combined
   pass; splitting into two stages adds cost without proportional gain.
5. Implementer subagents must not run `git commit` — the controller commits
   with sandbox disabled, both to clear `.git/.../index.lock` EPERM and to
   handle nested sub-repos consistently.

## Re-running the pilot

To regenerate one file as a smoke test of the skill (without overwriting the
live bank), point BUILD at a temporary output directory and verify the
result matches the live file structurally.
