---
paths:
  - "Paper/**/*.tex"
  - "**/manuscript*.tex"
  - "templates/paper/**"
---

# Paper-Writing Craft (genre expectations for an empirical economics manuscript)

**This rule governs exposition, not method.** It answers "what does a thorough first draft
look like" — literature density, narrative structure, typesetting. It never answers "which
estimator, diagnostic, or identification strategy to use" — that content is vetoed for the
owner's own field per [`meta-governance.md`](meta-governance.md)'s current, dated ruling.
Anything below that could be mistaken for a methods prescription should instead point at
[`/review-paper`](../skills/review-paper/SKILL.md) or [`inference-robustness.md`](inference-robustness.md),
which already own that ground.

## Why this rule exists

A comparison of an AI-drafted paper against an expert-drafted paper on the *same dataset and
research question* found the statistical content comparable or stronger in the AI draft
(more explicit hypothesis tests, more robustness checks) — the gap was entirely in **how much
context the paper builds around its own results** before and after presenting them. A model
left to its own judgment tends to under-cite, under-motivate, and under-typeset relative to a
human expert who has internalized a field's unwritten expectations for a "real" submission.
This rule writes those expectations down so a fresh clone of this template doesn't have to
rediscover them by falling short once.

## 1. Literature-engagement density

**The test, not a count.** A fixed "cite N papers" rule is a cargo-cult metric a model can
satisfy with padding. The actual bar: **a reader who has never seen this paper before must be
able to state, after the introduction alone, which prior claims this paper agrees with, which
it revises, and specifically why its approach differs.** That requires engaging multiple named
strands of prior work, not one — a paper that argues against "the literature" in the abstract
and then cites a single source in the introduction has not done this.

Concretely, for each strand the introduction touches:

1. Name the specific paper(s) or approach (not "prior work" or "some studies").
2. State what they find or claim, in enough detail that a reader could distinguish it from a
   different strand.
3. State in one clause how this paper's approach differs, extends, or contests it.

Use [`/lit-review`](../skills/lit-review/SKILL.md) to build this list before drafting the
introduction — do not draft the lit engagement from memory of what "sounds like" the field;
verify each citation is real and says what you're about to claim it says (this is exactly what
[`/verify-claims`](../skills/verify-claims/SKILL.md) checks after the fact — using `/lit-review`
first means fewer citations need correcting later).

## 2. Structural checklist

A manuscript earns the label "thorough" when it has all of the following, in this order. None
of these prescribe a method — they prescribe that the method, whatever it is, gets properly
framed:

1. **Title + abstract.** The abstract states the question, the approach in one sentence, the
   headline finding with its direction and (if central to the claim) magnitude, and what the
   finding implies — not just "we find X is significant."
2. **Introduction:**
   - An opening frame (a motivating fact, tension, or — where it genuinely clarifies rather than
     decorates — an epigraph). Optional; never required padding.
   - The gap: what existing measurement/approach misses or gets wrong.
   - The approach: what this paper does differently, in plain language before any notation.
   - **The contribution paragraph** — an explicit, first-person statement of what is new (not
     "we contribute to the literature" — name the contribution).
   - **The roadmap paragraph** — one sentence per remaining section, naming what question that
     section answers.
3. **A conceptual/framework section**, in prose, that explains *why* the approach measures what
   it claims to measure — before the technical estimation section, not folded into it. Where an
   abstract quantity is hard to grasp on first read (a rate, an index, a residual), a **worked
   numerical toy example** (two or three illustrative cases, concrete numbers) that shows the
   quantity behaving as claimed is a high-value device — it is frequently the single highest-ROI
   paragraph in the paper for reader comprehension, and its absence is a common reason an
   otherwise-rigorous draft reads as thin. Use it whenever it would clarify; do not force one
   where the quantity is already transparent.
4. **Data section** — source, sample construction, and every restriction applied, each with the
   observation count it costs (a reader must be able to reconstruct the analysis sample from
   this section alone).
5. **Results**, presented as an answer to the specific question the roadmap paragraph promised
   for that section — not a tour of every table produced.
6. **Discussion / limitations** — before the conclusion, not omitted. States what the evidence
   does *not* show, and what would overturn the headline claim. This is exposition of existing
   findings, not a request to run new diagnostics.
7. **Conclusion** — restates the contribution and its scope; does not introduce new claims.

## 3. Register and reporting-language discipline

Match claim strength to evidence strength — the same discipline
[`credible-claims`](../skills/credible-claims/SKILL.md) applies to research execution applies to
the prose itself: "significant" is a statistical term, not a rhetorical intensifier;
"suggests" and "shows" are not interchangeable; a null result is reported as informative, not
buried. See [`writing-with-ai.md`](writing-with-ai.md) for the broader human-readable prose
standard this sits inside.

## 4. Typesetting — match professional convention, not just "compiles"

Use [`Preambles/paper-header.tex`](../../Preambles/paper-header.tex) and
[`templates/paper/paper-template.tex`](../../templates/paper/paper-template.tex) as the starting
point for any manuscript. They encode the defaults below so a fresh draft doesn't ship the
tells of an unpolished LaTeX default:

- **No visible hyperlink boxes.** `hyperref`'s un-configured default draws a colored frame
  around every citation and cross-reference — a working-paper-in-progress does not look like
  this, and it is the single most visible tell of an unfinished draft. Use `hidelinks` (matches
  a plain print convention) or `colorlinks=true` with subdued, non-garish colors — never the
  bare default.
- **A real title/author/affiliation block** — not a single flush-left author line when there
  are multiple authors; a centered grid, one column per author with their affiliation beneath.
- **A formal abstract environment** — centered bold "Abstract" header, justified body,
  distinguishable from the introduction that follows.
- **`booktabs` tables** — `\toprule`/`\midrule`/`\bottomrule`, never hand-drawn `\hline` grids —
  and every table's numbers come from `\input{}`-ing the analysis script's output, never
  hand-transcribed (same discipline as [`stata-code-conventions.md`](stata-code-conventions.md) §4
  and [`r-code-conventions.md`](r-code-conventions.md)).
- **`natbib`-style citations** with a real bibliography style, not raw `\cite{}` rendered as a
  bracketed number with no author-year context in text.
- **Never write a "working draft" disclaimer, in any wording** — "Working title and working
  draft," "Comments welcome," "All errors are our own," and equivalents are genre-standard on a
  paper an author is actively circulating for feedback, but a skill that reports its own output
  as publication-ready must not manufacture that disclaimer itself; it is not a placeholder to
  invent, it is a statement about the author's own relationship to the draft. If there is no
  genuine acknowledgment content to put in the `\thanks{}` footnote (no real names, no real
  funding source), leave it as an explicit bracketed placeholder — `[Acknowledgments — fill in
  before circulating]` — the same way author names stay placeholders, never substitute
  boilerplate disclaimer text to make the footnote look filled-in.

## 5. Where this rule stops

- It does not choose an estimator, a clustering level, a bandwidth, a specification, or a
  robustness battery — [`inference-robustness.md`](inference-robustness.md) and
  [`/review-paper`](../skills/review-paper/SKILL.md) already own that, and
  [`meta-governance.md`](meta-governance.md) vetoes new prescriptive content here.
- It does not judge whether the identification strategy is sound — that is a `/review-paper`
  question, explicitly declined here per the same owner ruling.
- It is silent on which specific journal style to match; for a named target, see
  [`journal-profiles.md`](../references/journal-profiles.md) and
  [`/submission-disclosures`](../skills/submission-disclosures/SKILL.md).

## Enforcement

Applying this rule is the drafter's job; **checking** it is a separate, independent step — a
drafter cannot be trusted to grade its own compliance with the rule it just applied (the same
principle as [`review-fencing.md`](review-fencing.md)). [`content-depth-reviewer`](../agents/content-depth-reviewer.md)
is the read-only agent whose entire job is this rule's four sections; `/draft-paper` fans out to
it in Phase 2 as an independent check, not a rubber stamp on Phase 1's own drafting pass.

## Cross-references

- [`.claude/skills/draft-paper/SKILL.md`](../skills/draft-paper/SKILL.md) — the skill that applies this rule end to end, then auto-runs QA.
- [`.claude/agents/content-depth-reviewer.md`](../agents/content-depth-reviewer.md) — the independent reviewer that checks compliance with this rule (all four sections above).
- [`.claude/skills/lit-review/SKILL.md`](../skills/lit-review/SKILL.md) — build the citation list this rule requires, before drafting.
- [`.claude/rules/writing-with-ai.md`](writing-with-ai.md) — prose register and the human-readable standard.
- [`.claude/rules/inference-robustness.md`](inference-robustness.md) — the statistical-rigor side this rule deliberately does not touch.
- [`.claude/rules/meta-governance.md`](meta-governance.md) — the owner veto this rule stays inside.
- [`.claude/rules/stata-code-conventions.md`](stata-code-conventions.md) / [`r-code-conventions.md`](r-code-conventions.md) — the `\input{}` discipline tables must follow.
