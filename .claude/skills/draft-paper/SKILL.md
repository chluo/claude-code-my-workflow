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

**Empirical-design and specification decisions escalate to the user by default** — per
[`credible-claims.md`](../../skills/credible-claims/SKILL.md) standing rule 3, identification,
sample, and specification choices always return to the researcher, and this skill does not
override that. When one of these comes up, ask via `AskUserQuestion` as usual, but **always
include a waiver option among the choices**, worded along the lines of *"Use your judgment on
this and don't escalate design decisions like this for the rest of the session."* If the user
selects it, treat every subsequent design decision as a judgment call for the remainder of this
conversation — decide it yourself using ordinary best-practice defaults, do not ask again, and
log it in the Final Report's **Judgment calls** list instead. This waiver is scoped to design
decisions specifically; it does not also cover the general-clarification waiver below, and a
waiver granted in an earlier turn of this same conversation still applies to a later
`/draft-paper` invocation within it (forked runs inherit conversation history) — but check
whether it was actually granted before assuming so.

**Use `AskUserQuestion` whenever a RUN_CONFIG field or other input is genuinely unresolvable
from context — never mid-draft.** Same pattern: the question's options must always include a
waiver — *"Use your judgment on choices like this and stop asking for the rest of the
session."* If granted, resolve every subsequent genuinely-ambiguous field yourself for the rest
of this conversation (pick the most defensible option), disclosing the assumption in the
Pre-Flight Report or the Judgment calls list rather than asking again. This waiver is separate
from the design-decision one above — granting one does not grant the other.

## LaTeX toolchain — install automatically if missing, never ask

Before either compile step (Phase 1 step 4, Phase 3), confirm a XeLaTeX toolchain is present
(`xelatex --version`; also need `bibtex`). **If it's missing, install one yourself, right now,
with no confirmation step** — do not ask "should I install LaTeX?", and do not pause between
individual install commands for separate approval; run the sequence straight through, the same
"never pause to ask permission" policy as the rest of this skill's composed sub-steps.

Detect the platform and use a non-interactive install path — adapt exact flags/package names to
what's actually detected; the goal is a working, silent, unattended install, not a fixed
transcript:

- **Windows** — `winget install --id MiKTeX.MiKTeX -e --silent --accept-package-agreements --accept-source-agreements`;
  then configure MiKTeX to auto-install missing packages rather than prompt for them
  (`initexmf --set-config-value [MPM]AutoInstall=1`, or whatever the current MiKTeX CLI calls
  this — check `--help` if the exact flag has moved). If `winget` isn't available, fall back to
  MiKTeX's basic installer with its documented unattended flag.
- **macOS** — `brew install --cask basictex` (small, fast); upgrade to `mactex-no-gui` if a
  compile still fails on a package `tlmgr` can't resolve. Open a fresh shell (or re-source the
  path helper) so `xelatex`/`tlmgr` land on `PATH` within the same session; `sudo tlmgr install
  <package>` for anything `basictex` didn't ship.
- **Linux (Debian/Ubuntu)** — `sudo apt-get update && sudo apt-get install -y texlive-xetex
  texlive-latex-extra texlive-fonts-recommended texlive-bibtex-extra` (`-y` keeps it
  non-interactive).

**Escalate only if a good-faith install attempt still leaves no working `xelatex`** — no package
manager available and no viable direct-download path in this environment, or a required step
demands interactive credentials this session genuinely cannot supply (e.g., a `sudo` password
prompt with no way to answer it). Only then stop and tell the user what was tried, what failed,
and what they'd need to do manually. (This governs this skill's own conversational behavior —
it does not itself change the surrounding session's tool-permission mode; if that mode is
configured to require approval on every `Bash` call regardless of skill content, that's a
harness-level setting outside what this file can override.)

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
never mid-draft — including the general-clarification waiver option described in Continuity
above (unless that waiver was already granted earlier this conversation, in which case resolve
it yourself and note the assumption instead). Echo the resolved RUN_CONFIG back as a Pre-Flight
Report before Phase 1.

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
4. Compile (3-pass XeLaTeX + bibtex, mirroring `CLAUDE.md`'s existing LaTeX command block —
   see "LaTeX toolchain" above if `xelatex` isn't found) and confirm a PDF is produced before
   moving to Phase 2 — a draft that doesn't compile isn't a draft. Continue straight into
   Phase 2; a compiling PDF is progress, not completion.

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

Re-run the 3-pass XeLaTeX + bibtex compile after all fixes land (toolchain should already be
installed from Phase 1 — re-check "LaTeX toolchain" above if not). Confirm: PDF builds with no
errors, no overfull-hbox warnings past this repo's usual tolerance, and — visually — no
`hyperref` link boxes (the `hidelinks` default in `paper-header.tex` should make this automatic;
if a caller has swapped in `colorlinks`, re-check the rendered PDF, not just the source).

## Final Report

State, in this order:

1. **What was fixed automatically**, grouped by category (content-depth, correctness/CRITICAL,
   correctness/MAJOR, citations, bibliography, proofreading, mechanical-humanize, disclosures)
   with counts.
2. **Judgment calls made autonomously, if either waiver in Continuity was granted this
   conversation** — every RUN_CONFIG field defaulted and every design/specification choice made
   without escalating (sample restrictions, functional form, controls, clustering, robustness
   checks run) after the corresponding waiver was given. This is the transparency the skipped
   escalation owes the user — not optional, even when nothing here is wrong. Omit this section
   entirely if no waiver was granted and every genuinely ambiguous point escalated normally.
3. **The AI-voice punch-list** (if `/humanize` found anything non-mechanical) — the only items
   left requiring the user's own editorial judgment, per `writing-with-ai.md`.
4. **Remaining placeholders** — author names, institution, the acknowledgment footnote if there
   was no genuine content for it (never filled with a "working draft" disclaimer instead — see
   `paper-writing-craft.md` §4), and anything the Pre-Flight left as "general working paper"
   that a named target journal would instead fix (margins, citation style, length limits).
5. Confirmation the PDF compiles cleanly as of this report.

## What this skill does not do

- By default, does not choose an estimator, diagnostic, or identification strategy — those
  decisions escalate to the user per `credible-claims.md` rule 3, exactly as they would outside
  this skill (Continuity, above). If the user has granted the design-decision waiver earlier in
  the conversation, Phase 1 makes these choices autonomously for the rest of it instead, and
  discloses each one in the Final Report's Judgment-calls list; `/review-paper` still judges
  whether the specific choice made was sound, exactly as it would for a human-drafted paper.
  Either way, this repository never ships the choice as generic prescriptive content — see
  [`paper-writing-craft.md`](../../rules/paper-writing-craft.md) §5 and
  [`meta-governance.md`](../../rules/meta-governance.md).
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
