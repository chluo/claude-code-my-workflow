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

## Continuity — read this before Phase 0

**This is one uninterrupted task from Pre-Flight to Final Report.** There are exactly two valid
places to end a turn: the `AskUserQuestion` in Phase 0 when a RUN_CONFIG field is genuinely
unresolvable, and the Final Report at the very end of Phase 3. Everything in between —
including running the underlying data analysis, and every step of Phase 2 — is a means to this
skill's end, never itself a stopping point.

**Run every sub-step in this skill inline, in your own context — never as a background hand-off
you wait on separately.** If Phase 1 needs `/data-analysis` or `/stata-replication` and no
analysis exists yet, that pipeline is not a hand-off you delegate and end your turn on: follow
its instructions yourself, in the current context, then continue immediately to the next step of
*this* skill. A sub-pipeline finishing and producing its own natural-sounding "here are your
results" conclusion is not this skill's conclusion — if the manuscript hasn't been drafted,
reviewed, verified, and compiled yet, the task is not done, regardless of how much work the last
step did.

**Never pause to ask permission before invoking the next sub-step.** Every skill this pipeline
composes — `/data-analysis`, `/stata-replication`, `/review-paper --adversarial`,
`/verify-claims`, `/proofread`, `/humanize`, `/validate-bib`, `/submission-disclosures` — is this
skill's own internal machinery, not an action that needs a separate go-ahead. "Should I now run
`/proofread`?" is not a valid thing to ask; just run it.

**Empirical-design and specification choices are your judgment call, not an escalation.**
[`credible-claims.md`](../../skills/credible-claims/SKILL.md)'s standing rule 3 says
identification, sample, and specification decisions always return to the researcher — this skill
**deliberately overrides that default**, on explicit user instruction, for the scope of
`/draft-paper` only (that standing rule is unchanged for every other skill). Sample
restrictions, functional form, control sets, clustering level, robustness checks run: decide
these yourself using ordinary best-practice defaults for the design at hand, exactly as
`/data-analysis` Phase 3 already does when run standalone. Do not stop to ask which
specification to run. The override is not silence, though — every non-obvious choice you make
goes in the Final Report's **Judgment calls** list (below), so the record survives even though
approval wasn't sought first.

**Reserve `AskUserQuestion` for two cases only:** (a) no reasonable default exists and the
candidate approaches would produce materially different, non-interchangeable papers (not "which
of these two defensible specifications" — that's a judgment call per above; more like "the
research question as stated is compatible with two entirely different datasets and there's no
way to infer which one is meant"), or (b) something makes proceeding practically impossible —
missing or unreadable data, no research question or topic given at all, a named target journal
that doesn't exist in `journal-profiles.md` and has no reasonable generic fallback. Everything
else: pick the most defensible option, state the assumption in the Pre-Flight Report or the
Judgment calls list, and keep going.

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

Resolve every field yourself where a reasonable default exists (see the Continuity section above
for exactly when `AskUserQuestion` is warranted instead — it's narrower than "any field is
unclear"). Typical defaults: no `target_journal` named → "general working paper"; `authors`
unspecified → placeholders; `analysis_tool` inferable from what's already in `scripts/` or
`data/` → infer it, don't ask. Echo the resolved RUN_CONFIG back as a Pre-Flight Report before
Phase 1, including a one-line note on any field you defaulted rather than were told.

## Phase 1: Draft

1. **Ensure the analysis outputs exist — this is a precondition, not a deliverable.** Check
   `outputs_path` from Pre-Flight. If it's empty or missing, run
   [`/data-analysis`](../data-analysis/SKILL.md) or [`/stata-replication`](../stata-replication/SKILL.md)
   yourself, inline, now (Stata logs/intermediates go to `scripts/stata/_log/` / `_temp/` per
   [`stata-code-conventions.md`](../../rules/stata-code-conventions.md); the paper only ever
   `\input{}`s `_outputs/`). **The moment that pipeline finishes, continue immediately to step 2
   below in the same turn — do not stop, summarize, or hand back to the user here.** No number in
   any table/figure is ever hand-transcribed; every one comes from `\input{}`-ing `outputs_path`.
2. Copy [`templates/paper/paper-template.tex`](../../../templates/paper/paper-template.tex) and
   [`Preambles/paper-header.tex`](../../../Preambles/paper-header.tex) into place (`Paper/` by
   default, or wherever the Pre-Flight named).
3. Apply [`paper-writing-craft.md`](../../rules/paper-writing-craft.md) in full: literature
   density via the strand-by-strand test in §1 (run [`/lit-review`](../lit-review/SKILL.md) first
   if the literature list doesn't already exist), the structural checklist in §2, the register
   discipline in §3. This is the actual manuscript prose — the deliverable step 1 was in service
   of.
4. Compile (3-pass XeLaTeX + bibtex, mirroring `CLAUDE.md`'s existing LaTeX command block) and
   confirm a PDF is produced before moving to Phase 2 — a draft that doesn't compile isn't a
   draft. Continue straight into Phase 2; a compiling PDF is progress, not completion.

## Phase 2: Auto-finalize (no user prompt between any of these steps)

A compiling draft from Phase 1 is not the deliverable — proceed into this phase automatically,
in the same turn, with no report back to the user in between. Runs the existing critic-fixer and CoVe patterns from
[`orchestrator-protocol.md`](../../rules/orchestrator-protocol.md) — this skill composes them,
it does not reimplement verification. Loop-until-dry applies across the whole sequence: two
consecutive rounds adding zero new CRITICAL/MAJOR findings (by the deterministic
`sha1(file:line:locus)` id) ends the loop; `run_config.max_rounds` is the fallback cap.

1. **Content-depth & structure review** — fork `content-depth-reviewer` (`Agent`, `context: fork`)
   against [`paper-writing-craft.md`](../../rules/paper-writing-craft.md)'s four sections
   (literature-engagement density, structural completeness, register discipline, typesetting
   checklist). This is the check that catches "thin draft" symptoms specifically — a drafter
   cannot be trusted to grade its own compliance with the rule it just applied, so this runs as
   an independent reviewer, not a self-check folded into Phase 1. Apply every CRITICAL/MAJOR
   finding directly (write the missing contribution/roadmap paragraph, add the second literature
   strand, fix the `hyperref` config, etc.), re-run the reviewer, repeat until dry.
2. **Content review** — `/review-paper --adversarial` (or `/seven-pass-review` for a long
   draft). This skill *is* the fixer: apply every CRITICAL/MAJOR finding directly, re-run the
   review, repeat until dry.
3. **Claim verification** — `/verify-claims` (CoVe, fresh-context `claim-verifier` fork, per
   `post-flight-verification.md`). Any HIGH-WARN (fabricated citation, numeric/directional
   contradiction) is fixed and the specific claim re-verified before moving on — the existing
   must-fix policy, not a new one.
4. **Bibliography** — `/validate-bib`. Fix every structural finding (missing/unused entries,
   malformed fields) directly.
5. **Proofreading** — `/proofread` stays read-only per its own design; this skill applies every
   reported fix directly (typos, grammar, overflow, consistency), then re-runs `/proofread`,
   looping until dry — the same critic→fixer→re-audit shape as `/qa-quarto`.
6. **Prose voice** — `/humanize` stays detect-only per its own design (`writing-with-ai.md`; the
   `[LEARN]` lesson that auto-rewriting these degrades quality). Any finding it tags `mechanical`
   is fixed directly; pure AI-voice tells (hedging stacks, boilerplate transitions, tricolon
   abuse, etc.) are **not** rewritten — collect them into the Final Report's punch-list instead.
7. **Disclosures** — `/submission-disclosures`, using the target journal / disclosure policy
   named in Pre-Flight (or a generic AI-use + data-availability block if none was named).

## Phase 3: Compile + verify

Re-run the 3-pass XeLaTeX + bibtex compile after all fixes land. Confirm: PDF builds with no
errors, no overfull-hbox warnings past this repo's usual tolerance, and — visually — no
`hyperref` link boxes (the `hidelinks` default in `paper-header.tex` should make this automatic;
if a caller has swapped in `colorlinks`, re-check the rendered PDF, not just the source).

## Final Report

State, in this order:

1. **What was fixed automatically**, grouped by category (content-depth, correctness/CRITICAL,
   correctness/MAJOR, citations, bibliography, proofreading, mechanical-humanize, disclosures)
   with counts.
2. **Judgment calls made autonomously** — every RUN_CONFIG field defaulted rather than confirmed,
   and every empirical-design/specification choice made without escalating (sample restrictions,
   functional form, controls, clustering, robustness checks run) per the Continuity section's
   override of `credible-claims.md` rule 3. This is the transparency the skipped escalation owes
   the user — not optional, even when nothing here is wrong.
3. **The AI-voice punch-list** (if `/humanize` found anything non-mechanical) — the only items
   left requiring the user's own editorial judgment, per `writing-with-ai.md`.
4. **Remaining placeholders** — author names, institution, and anything the Pre-Flight left as
   "general working paper" that a named target journal would instead fix (margins, citation
   style, length limits).
5. Confirmation the PDF compiles cleanly as of this report.

## What this skill does not do

- Makes concrete specification choices for the one paper at hand (Continuity section, above) —
  but never ships or asserts a generic "use estimator X" rule as content, and never treats its
  own choice as beyond challenge: Phase 2's `/review-paper` still judges whether *this specific*
  choice was sound, exactly as it would for a human-drafted paper. The distinction that matters:
  doing the work autonomously is not the same as this repository prescribing methodology to
  users in general, which stays vetoed per [`paper-writing-craft.md`](../../rules/paper-writing-craft.md) §5
  and [`meta-governance.md`](../../rules/meta-governance.md).
- Does not build a replication package (`/replication-package` is a separate deliverable — offer
  it as a next step, don't run it automatically, per `CLAUDE.md`'s Scope Discipline).
- Does not overwrite an existing manuscript without confirming — if `Paper/` (or the named
  target) already has a `.tex` file, ask before replacing it.

## Cross-references

- [`.claude/rules/paper-writing-craft.md`](../../rules/paper-writing-craft.md) — the content/structure rule this skill applies.
- [`.claude/agents/content-depth-reviewer.md`](../../agents/content-depth-reviewer.md) — the independent reviewer that checks Phase 1's draft actually complied with the rule above (Phase 2 step 1).
- [`.claude/rules/orchestrator-protocol.md`](../../rules/orchestrator-protocol.md) — the runtime (fan-out/reduce/judge/loop-until-dry) this skill's Phase 2 composes.
- [`templates/paper/paper-template.tex`](../../../templates/paper/paper-template.tex) / [`Preambles/paper-header.tex`](../../../Preambles/paper-header.tex) — the generic assets drafted into.
- [`.claude/skills/review-paper/SKILL.md`](../review-paper/SKILL.md), [`verify-claims`](../verify-claims/SKILL.md), [`proofread`](../proofread/SKILL.md), [`humanize`](../humanize/SKILL.md), [`validate-bib`](../validate-bib/SKILL.md), [`submission-disclosures`](../submission-disclosures/SKILL.md) — composed, not modified.
