---
name: draft-paper
description: Draft a full empirical-economics manuscript from a research question and existing analysis outputs, then automatically run the complete QA/finalize pipeline (adversarial review, claim verification, proofreading, bibliography validation, disclosures) with no further prompting. Use when user says "draft the paper", "write up these results as a paper", "turn this analysis into a manuscript", "produce a publication-ready draft". NOT for reviewing an existing manuscript (use `/review-paper`) or running the underlying analysis (use `/data-analysis` or `/stata-replication` first).
argument-hint: "[research question or title]"
allowed-tools: ["Read", "Write", "Edit", "Grep", "Glob", "Bash", "Agent", "Task", "AskUserQuestion"]
context: fork
disable-model-invocation: true
effort: high
---

# Draft Paper (draft → automatic finalize)

Produces a full manuscript draft and then finalizes it end to end — the loop described in
[`orchestrator-protocol.md`](../../rules/orchestrator-protocol.md) — so the only thing left for
the user to touch afterward is author/institution names and any journal-specific choice the
Pre-Flight didn't already resolve. This is the skill that was missing: existing skills review or
QA an already-written manuscript; this one writes the manuscript.

## Phase 0: Pre-Flight (RUN_CONFIG — collect everything before drafting starts)

Per [`orchestration-schemas.md` §5](../../references/orchestration-schemas.md), gather every
interactive choice now, since nothing downstream can pause to ask:

```yaml
run_config:
  title_or_rq: [research question / working title]
  authors: [placeholder names — real names filled in by the user afterward]
  analysis_tool: R | Stata | Python           # where do _outputs/ tables/figures live
  outputs_path: scripts/R/_outputs/ | scripts/stata/_outputs/ | ...
  target_journal: [name, or "general working paper" if none]
  disclosure_policy: [from target_journal, or generic if none named]
  max_rounds: 5                                # loop-until-dry fallback cap, per orchestrator-protocol
```

If any required field is genuinely unresolvable from context, use `AskUserQuestion` **now** —
never mid-draft. Echo the resolved RUN_CONFIG back as a Pre-Flight Report before Phase 1.

## Phase 1: Draft

1. Copy [`templates/paper/paper-template.tex`](../../../templates/paper/paper-template.tex) and
   [`Preambles/paper-header.tex`](../../../Preambles/paper-header.tex) into place (`Paper/` by
   default, or wherever the Pre-Flight named).
2. Apply [`paper-writing-craft.md`](../../rules/paper-writing-craft.md) in full: literature
   density via the strand-by-strand test in §1 (run [`/lit-review`](../lit-review/SKILL.md) first
   if the literature list doesn't already exist), the structural checklist in §2, the register
   discipline in §3.
3. Every number in every table/figure comes from `\input{}`-ing `outputs_path` — never
   hand-transcribed. If the analysis hasn't been run yet, run it first via
   [`/data-analysis`](../data-analysis/SKILL.md) or [`/stata-replication`](../stata-replication/SKILL.md)
   (Stata logs/intermediates go to `scripts/stata/_log/` / `_temp/` per
   [`stata-code-conventions.md`](../../rules/stata-code-conventions.md); the paper only ever
   `\input{}`s `_outputs/`).
4. Compile (3-pass XeLaTeX + bibtex, mirroring `CLAUDE.md`'s existing LaTeX command block) and
   confirm a PDF is produced before moving to Phase 2 — a draft that doesn't compile isn't a
   draft.

## Phase 2: Auto-finalize (no user prompt between any of these steps)

Runs the existing critic-fixer and CoVe patterns from
[`orchestrator-protocol.md`](../../rules/orchestrator-protocol.md) — this skill composes them,
it does not reimplement verification. Loop-until-dry applies across the whole sequence: two
consecutive rounds adding zero new CRITICAL/MAJOR findings (by the deterministic
`sha1(file:line:locus)` id) ends the loop; `run_config.max_rounds` is the fallback cap.

1. **Content review** — `/review-paper --adversarial` (or `/seven-pass-review` for a long
   draft). This skill *is* the fixer: apply every CRITICAL/MAJOR finding directly, re-run the
   review, repeat until dry.
2. **Claim verification** — `/verify-claims` (CoVe, fresh-context `claim-verifier` fork, per
   `post-flight-verification.md`). Any HIGH-WARN (fabricated citation, numeric/directional
   contradiction) is fixed and the specific claim re-verified before moving on — the existing
   must-fix policy, not a new one.
3. **Bibliography** — `/validate-bib`. Fix every structural finding (missing/unused entries,
   malformed fields) directly.
4. **Proofreading** — `/proofread` stays read-only per its own design; this skill applies every
   reported fix directly (typos, grammar, overflow, consistency), then re-runs `/proofread`,
   looping until dry — the same critic→fixer→re-audit shape as `/qa-quarto`.
5. **Prose voice** — `/humanize` stays detect-only per its own design (`writing-with-ai.md`; the
   `[LEARN]` lesson that auto-rewriting these degrades quality). Any finding it tags `mechanical`
   is fixed directly; pure AI-voice tells (hedging stacks, boilerplate transitions, tricolon
   abuse, etc.) are **not** rewritten — collect them into the Final Report's punch-list instead.
6. **Disclosures** — `/submission-disclosures`, using the target journal / disclosure policy
   named in Pre-Flight (or a generic AI-use + data-availability block if none was named).

## Phase 3: Compile + verify

Re-run the 3-pass XeLaTeX + bibtex compile after all fixes land. Confirm: PDF builds with no
errors, no overfull-hbox warnings past this repo's usual tolerance, and — visually — no
`hyperref` link boxes (the `hidelinks` default in `paper-header.tex` should make this automatic;
if a caller has swapped in `colorlinks`, re-check the rendered PDF, not just the source).

## Final Report

State, in this order:

1. **What was fixed automatically**, grouped by category (content/CRITICAL, content/MAJOR,
   citations, bibliography, proofreading, mechanical-humanize, disclosures) with counts.
2. **The AI-voice punch-list** (if `/humanize` found anything non-mechanical) — the only items
   left requiring the user's own editorial judgment, per `writing-with-ai.md`.
3. **Remaining placeholders** — author names, institution, and anything the Pre-Flight left as
   "general working paper" that a named target journal would instead fix (margins, citation
   style, length limits).
4. Confirmation the PDF compiles cleanly as of this report.

## What this skill does not do

- Does not choose an estimator, diagnostic, or identification strategy — that's `/review-paper`'s
  job to *flag if wrong*, never this skill's job to *pick*. See
  [`paper-writing-craft.md`](../../rules/paper-writing-craft.md) §5 and the owner veto in
  [`meta-governance.md`](../../rules/meta-governance.md).
- Does not build a replication package (`/replication-package` is a separate deliverable — offer
  it as a next step, don't run it automatically, per `CLAUDE.md`'s Scope Discipline).
- Does not overwrite an existing manuscript without confirming — if `Paper/` (or the named
  target) already has a `.tex` file, ask before replacing it.

## Cross-references

- [`.claude/rules/paper-writing-craft.md`](../../rules/paper-writing-craft.md) — the content/structure rule this skill applies.
- [`.claude/rules/orchestrator-protocol.md`](../../rules/orchestrator-protocol.md) — the runtime (fan-out/reduce/judge/loop-until-dry) this skill's Phase 2 composes.
- [`templates/paper/paper-template.tex`](../../../templates/paper/paper-template.tex) / [`Preambles/paper-header.tex`](../../../Preambles/paper-header.tex) — the generic assets drafted into.
- [`.claude/skills/review-paper/SKILL.md`](../review-paper/SKILL.md), [`verify-claims`](../verify-claims/SKILL.md), [`proofread`](../proofread/SKILL.md), [`humanize`](../humanize/SKILL.md), [`validate-bib`](../validate-bib/SKILL.md), [`submission-disclosures`](../submission-disclosures/SKILL.md) — composed, not modified.
