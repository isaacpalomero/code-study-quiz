# BUILD Questionnaire — Scoping the Bank

When BUILD mode invokes `superpowers:brainstorming`, this is the questionnaire
the orchestrator must drive. Ask the questions **one at a time** (the
brainstorming skill enforces this), preferring multiple-choice form via
`AskUserQuestion`.

## Question 0 — Scope of the quiz (ALWAYS FIRST)

This answer reshapes every subsequent question. The three scopes are:

- **A) Repositorio completo / amplio.** Whole codebase or a broad set of
  subsystems. Default to multiple output files (one per subsystem) plus an
  optional cross-cutting functional file. Example: the MCB pilot (150
  questions across 4 files).
- **B) Desarrollo concreto.** Quiz scoped to a specific piece of work, done
  or in progress. Possible sources, prompted depending on what the user has
  on hand:
  - A **Jira / GitLab ticket** — retrieve content via
    `mcp__claude_ai_Atlassian__getJiraIssue` /
    `mcp__plugin_gitlab_gitlab__get_issue`. Use the ticket text, links,
    acceptance criteria, comments.
  - The **current branch** — use `git log main...HEAD`, `git diff main...HEAD`,
    PR/MR description (`mcp__plugin_gitlab_gitlab__get_merge_request`).
  - A **spec / design doc** — e.g. `docs/superpowers/specs/*.md`,
    `docs/local/MCB1-*/SPEC.md`, or any path the user names.
  - A **PR / MR** — diff + description.
    Output is typically a single `.md` of 15–25 questions.
- **C) Tema libre.** The user describes in free text what they want to be
  quizzed on. The orchestrator then asks what files / directories / docs the
  user knows are relevant, and lets the implementer subagent discover the
  rest.

## Questions 1+ — variable by scope

Ask only the ones that apply.

### For scope A (whole repo)

1. Codebase root path (default: the current working directory).
2. Subsystems to cover (multi-select; expressed as paths or globs).
3. Subsystems explicitly **out of scope** (mention them in the spec as
   exclusions, like MCB excluded the anomaly engine in iteration 1).
4. Whether to include a cross-cutting functional file (`00-…`).

### For scope B (specific development)

1. The identifier(s): Jira/GitLab ticket key(s), branch name, spec file path,
   PR/MR id — collect all that apply.
2. Companion sources the user wants included (commits, related docs,
   internal Confluence pages).

### For scope C (free-form)

1. Free-text description (the quiz topic).
2. Known relevant files / directories / docs.

### Common to all scopes

1. **Functional documentation** location (directory of `.md` files exported
   from a wiki, internal docs, etc.) and the URL convention used to link back
   to the source. Example for Confluence:
   `https://syntax.atlassian.net/wiki/spaces/{SPACE}/pages/{PAGE_ID}/` where
   `{PAGE_ID}` is the trailing number in each filename. The user provides the
   literal `{SPACE}`; the skill never assumes it.
2. **Output language** (default: Spanish). Affects question wording,
   pre-lectura prose, and explanations.
3. **Volume and distribution.**
   - Scope A defaults: 40 per file (16 Básico / 14 Intermedio / 10 Avanzado;
     24 Técnica / 10 Funcional / 6 Mixta); cross-cutting functional file
     ~30 (12/11/7, 100 % Funcional).
   - Scopes B and C defaults: 15–25 in a single file (proportions per
     `quality-rubric.md` minimums).
4. **Output directory** (where the `.md` files land) and **commit strategy**:
   - Main repo (single git root) → controller commits in main.
   - Nested sub-repo (the path has its own `.git`) → controller commits in
     the sub-repo. **Sandbox warning:** `git commit` must use
     `dangerouslyDisableSandbox: true` to avoid `.git/.../index.lock` EPERM.
   - No commit → just write files (useful for ephemeral / draft work).

## Output of the questionnaire

The orchestrator writes the spec at
`docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` (per
`superpowers:brainstorming` convention). The spec captures:

- The chosen scope (A/B/C) and its sources.
- The file structure (one row per `.md` to generate, with counts and source
  refs).
- Counts and distributions (filled with the chosen volume).
- Language, output directory, commit strategy.
- Explicit out-of-scope items.
- A pointer to `references/question-anatomy.md` and
  `references/quality-rubric.md` (those are the contract every generated
  question must meet).
