# Reviewer Subagent Prompt Template

Use this template for the **single combined** spec-compliance + question-quality
review after each implementer task. (MCB experience: a two-stage review added
cost without proportional gain; collapsing into one pass with explicit
priorities works better.)

The controller has already verified counts mechanically (`grep` over the file)
before dispatch — so the reviewer focuses on **correctness** and **quality**.

```
Task tool (general-purpose, model: sonnet):
  description: "Review {file_id}: {filename}"
  prompt: |
    Review the quiz file `{output_path}/{filename}` ({N} MCQ in {LANGUAGE}
    about {topic_summary}). Be skeptical — verify yourself by reading the
    actual file and the source code/docs. Do NOT trust any summary.

    Work from: {repo_root}

    Counts already verified mechanically (N={N}; levels
    {N_basico}/{N_intermedio}/{N_avanzado}; categories
    {N_tec}/{N_func}/{N_mix}; letter balance; no all/none; sequential IDs).
    **Do not re-count.** Focus on the priorities below.

    ## Priority 1 — Snippet fidelity (highest risk)

    For AT LEAST 12 (Técnica)/(Mixta) questions, open the referenced file and
    confirm:
    - The snippet is REAL code that exists in the file (not invented or
      paraphrased), with correct symbol/entity/method names.
    - Where lines were omitted inside the snippet, a `// …` marker is present
      — non-contiguous lines must NOT appear as contiguous.
    - The `archivo:línea` ref is approximately right and points to the right
      file (±2 lines for formatter drift is acceptable).
    Flag any invented snippet, wrong file, misnamed symbol, or contiguous
    presentation of non-adjacent code.

    Source paths in scope: {list_of_code_paths}.

    ## Priority 2 — Answer correctness

    For ~10 questions across all levels, verify the marked answer is truly
    correct given the framework / domain semantics and the source. Spot-check
    the easy-to-get-wrong areas: framework annotations, draft/composition
    semantics, lifecycle transitions, factual values (e.g. batch sizes, retry
    counts, dependency declarations). Flag any wrong answer or misleading
    explanation.

    ## Priority 3 — Quality

    - Exactly ONE unambiguous correct answer per question (no "most correct"
      debates).
    - Distractors plausible and similar in style/length to the correct
      option; none absurd; no leak via grammar/specificity.
    - Pre-lecturas self-contained (could you answer from the pre-lectura
      alone?).
    - No two questions test the exact same fact (or its inverse).
    - Confluence/source links well-formed per the user-chosen template;
      do NOT "correct" the literal space/path supplied by the user.

    ## Report

    - ✅ Approved, or
    - ❌ Issues — grouped Critical (invented snippet / wrong answer / wrong
      factual value) / Important / Minor, each with question ID, the
      problem, and a concrete fix.

    List the technical snippets you verified (IDs + file) so the controller
    knows coverage. Be concise.
```

## How the controller uses the report

- **Critical** → dispatch a fix subagent (or fix inline) with precise
  instructions; re-verify mechanically; re-dispatch the reviewer only if
  uncertain.
- **Important** → fix inline if mechanical (≤3 small edits) or via fix
  subagent if more involved.
- **Minor** → fix inline; no re-review needed unless multiple minors compound.

Whether fixes happen via subagent or controller edit, the controller commits
afterwards.

## Anti-patterns

- Do NOT split into two reviewer subagents (spec then quality). One combined
  pass with explicit priorities is the lesson learned.
- Do NOT have the reviewer count totals — that's the controller's job
  (`grep -c '^### P-…'`).
- Do NOT have the reviewer "correct" user-specified conventions
  (Confluence space key, link templates). If the reviewer flags one as
  "wrong vs the actual source footer", the controller treats it as a false
  positive when the user explicitly chose that literal.
