# Implementer Subagent Prompt Template

Use this template when the controller dispatches a subagent to generate ONE
quiz file. Fill the `{…}` placeholders from the spec/plan; the rest is fixed.

````
Task tool (general-purpose, model: sonnet):
  description: "Implement {file_id}: {filename}"
  prompt: |
    You are creating `{output_path}/{filename}`: a bank of exactly
    {N} multiple-choice study questions in {LANGUAGE} about {topic_summary}.

    Work from: {repo_root}

    ## Question anatomy

    See `.claude/skills/code-study-quiz/references/question-anatomy.md` for
    the canonical template and two worked examples. Replicate them exactly.

    ## Quality rules (non-negotiable)

    See `.claude/skills/code-study-quiz/references/quality-rubric.md`. The
    rules that catch agents most often:

    - **Snippet fidelity:** real code from the file; insert `// …` wherever
      lines are omitted; never present non-contiguous lines as contiguous.
      Reference as `archivo:línea`.
    - **Correct-letter balance:** distribute roughly evenly across A/B/C/D.
      For N≥40, target ≥8 of each (≥8 D and ≥8 A obligatory). Do NOT cluster
      on B/C.
    - **No duplicate / inverted facts** across questions.
    - **Pre-lectura autocontenida:** answerable from the snippet/prose
      alone; the link or `archivo:línea` is optional support.
    - No "todas/ninguna de las anteriores"; no double negatives.

    ## Counts (STRICT)

    - Total: {N}.
    - Levels: {N_basico} [Básico] · {N_intermedio} [Intermedio] ·
      {N_avanzado} [Avanzado], grouped Básico → Intermedio → Avanzado,
      IDs `P-{f}.1`…`P-{f}.{N}` sequential.
    - Categories: {N_tec} (Técnica) · {N_func} (Funcional) ·
      {N_mix} (Mixta).
    - ≥{N_cap_critical} framework-critical questions inside Técnica/Mixta
      (drafts, composition SAVE handlers, projections-with-filter,
      `@flow.status`, authorization, etc.).

    ## Sources to READ

    Technical (read these, extract real snippets):
    {list_of_code_paths}

    Functional (for Funcional / Mixta), link convention
    `{LINK_TEMPLATE}` with `{PAGE_ID}` = trailing number of the source
    filename:
    {list_of_doc_paths}

    Also read:
    - `{output_path}/README.md` (the bank rubric).
    - The spec at `{spec_path}`.

    If unsure about fine framework points, you MAY consult the
    `sap-cap-agent` / `sap-btp-expert-agent` / `sap-advisor-agent` skills
    for validation — anchor every question to the real code/docs of the
    target project.

    ## Topic coverage (must touch ALL)

    {bullet_list_of_topics}

    ## Document header

    ```markdown
    # {file_id} · {file_title}

    > Quiz de estudio · {N} preguntas · niveles Bloom · categorías T/F/M.
    > Lee la pre-lectura y despliega **Respuesta**. Rúbrica en `README.md`.

    ---
    ```
    Separate questions with `\n\n---\n\n`.

    ## Process

    1. Read the rubric (`question-anatomy.md`, `quality-rubric.md`), the
       spec, and ALL the source files listed.
    2. Draft {N} questions respecting counts and topic coverage.
    3. Verify every Técnica snippet against the real file (content + line;
       add `// …` for omissions).
    4. Self-check: counts, letter balance, 4 options each, no all/none,
       IDs sequential, ≥{N_cap_critical} framework-critical, pre-lecturas
       autocontained.
    5. Write `{output_path}/{filename}`.

    ## IMPORTANT — do NOT commit

    Do NOT run `git add` or `git commit`. The controller handles commits
    (nested-repo + sandbox reasons). Just write the file.

    ## Report back

    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT.
    - Counts per level and per category.
    - Correct-answer letter tally (A/B/C/D).
    - IDs of framework-critical questions.
    - Any snippet whose line you could not pin down (use `// …` and approx).
    - Concerns (e.g. doc gaps you noticed).
````

## Placeholder cheat-sheet

| Placeholder                                           | Source                                                    |
| ----------------------------------------------------- | --------------------------------------------------------- |
| `{file_id}`                                           | Task number (e.g. "Task 3")                               |
| `{filename}`                                          | E.g. `01-modelo-datos.md`                                 |
| `{output_path}`                                       | E.g. `docs/local/quiz`                                    |
| `{repo_root}`                                         | Absolute path to target repo                              |
| `{N}`, `{N_basico}`, `{N_intermedio}`, `{N_avanzado}` | From the spec                                             |
| `{N_tec}`, `{N_func}`, `{N_mix}`                      | From the spec                                             |
| `{N_cap_critical}`                                    | Usually 3 for ~40-question subsystem files; lower for B/C |
| `{LANGUAGE}`                                          | Spanish by default                                        |
| `{topic_summary}`                                     | One-line summary of the file's subject                    |
| `{list_of_code_paths}`                                | Exact paths to read for snippets                          |
| `{list_of_doc_paths}`                                 | Exact functional doc paths                                |
| `{LINK_TEMPLATE}`                                     | URL template with `{PAGE_ID}` literal                     |
| `{bullet_list_of_topics}`                             | From the spec's topic coverage                            |
| `{spec_path}`                                         | `docs/superpowers/specs/YYYY-MM-DD-…md`                   |
