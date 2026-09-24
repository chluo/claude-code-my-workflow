---
name: draft-paper
description: Draft a full empirical-economics manuscript from a research question and existing analysis outputs, then automatically run the complete QA/finalize pipeline (adversarial review, claim verification, proofreading, bibliography validation, disclosures) and assemble a DCAS-compliant replication package, with no further prompting. Use when user says "draft the paper", "write up these results as a paper", "turn this analysis into a manuscript", "produce a publication-ready draft". NOT for reviewing an existing manuscript (use `/review-paper`) or running the underlying analysis (use `/data-analysis` or `/stata-replication` first).
argument-hint: "[research question or title]"
allowed-tools: ["Read", "Write", "Edit", "Grep", "Glob", "Bash", "Agent", "Task", "AskUserQuestion"]
context: fork
disable-model-invocation: true
effort: high
---

# Draft Paper (draft → automatic finalize)

Produces a full manuscript draft, finalizes it end to end — the loop described in
[`orchestrator-protocol.md`](../../rules/orchestrator-protocol.md) — and assembles its
replication package, so the only thing left for the user to touch afterward is
author/institution names and any journal-specific or deposit choice the Pre-Flight and Phase 4
didn't already resolve. This is the skill that was missing: existing skills review or QA an
already-written manuscript; this one writes the manuscript and packages it.

## Continuity — read this before Phase 0

**Invoking this skill is itself the check-in — no earlier standing preference pauses it
mid-run.** If something earlier in this project set a general expectation to check in between
phases of work while the user is still learning the workflow (e.g., "for the first few sessions,
check in with me a bit more often," or "get approval before moving to a new phase") — that
preference governs **which tasks get their own plan-and-approval cycle in the first place.** It
does not mean execution should pause partway through a task the user has already explicitly
requested by invoking `/draft-paper`. This skill's own phase structure — Pre-Flight through
Phase 4 and the exit checkpoint below — **is** the complete, single unit of work that preference
already approved by name; there is no "config adaptation" phase versus "data pipeline" phase
inside it that independently warrants its own pause. If project config (CLAUDE.md placeholders,
environment setup) genuinely needs adaptation before this flow can proceed, that is a Phase 0/1
precondition to absorb inline — the same discipline as the LaTeX toolchain and the
analysis-outputs precondition below — never a separate task with its own plan, approval, and
stopping point. If a standing "check in more often" preference and an explicit invocation of
this skill seem to conflict, **the explicit invocation wins for the scope of this run.**

**This is one uninterrupted task from Pre-Flight to Final Report.** There are exactly two valid
places to end a turn: the `AskUserQuestion` escalations described below, and the Final Report at
the very end of Phase 4 — which itself must end with the permission check described immediately
below. Everything in between — including running the underlying data analysis, and every step of
Phase 2 through Phase 4 — is a means to this skill's end, never itself a stopping point.

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
`/verify-claims`, `/proofread`, `/humanize`, `/validate-bib`, `/submission-disclosures`,
`/replication-package` (which itself composes `/audit-reproducibility`) — is this skill's own
internal machinery, not an action that needs a separate go-ahead. "Should I now run
`/proofread`?" is not a valid thing to ask; just run it.

**Empirical-design and specification decisions escalate to the user by default** — per
[`credible-claims.md`](../../skills/credible-claims/SKILL.md) standing rule 3, identification,
sample, and specification choices always return to the researcher, and this skill does not
override that. When one of these comes up, ask via `AskUserQuestion` as usual, but **always
include a waiver option among the choices**, worded along the lines of *"Use your judgment on
this and don't escalate design decisions like this for the rest of the session"* — **plus the
broader "switch to full contractor mode" option described below**, since a user facing this
escalation may want both waivers at once, not just this one. If the user selects the narrower
waiver, treat every subsequent design decision as a judgment call for the remainder of this
conversation — decide it yourself using ordinary best-practice defaults, do not ask again, and
log it in the Final Report's **Judgment calls** list instead. This waiver is scoped to design
decisions specifically; it does not also cover the general-clarification waiver below (contractor
mode covers both at once — see below), and a waiver granted in an earlier turn of this same
conversation still applies to a later `/draft-paper` invocation within it (forked runs inherit
conversation history) — but check whether it was actually granted before assuming so.

**Use `AskUserQuestion` whenever a RUN_CONFIG field or other input is genuinely unresolvable
from context — never mid-draft.** Same pattern: the question's options must always include a
waiver — *"Use your judgment on choices like this and stop asking for the rest of the
session"* — **plus the "switch to full contractor mode" option described below**. If the
narrower waiver is granted, resolve every subsequent genuinely-ambiguous field yourself for the
rest of this conversation (pick the most defensible option), disclosing the assumption in the
Pre-Flight Report or the Judgment calls list rather than asking again. This waiver is separate
from the design-decision one above (contractor mode covers both at once — see below) — granting
one alone does not grant the other.

## Contractor mode — one switch for both waivers

The two waivers above are independent by design, so getting full autonomy normally means
triggering and granting each separately as it happens to come up. **Contractor mode is a single,
explicit toggle that grants both at once** — named after how some users frame full autonomy at
their own project's kickoff: *"switch to contractor mode — coordinate everything autonomously
and only come back to me when there's ambiguity or a decision to make."* That framing isn't part
of this template's own content; the name just recognizes it as a phrase worth recognizing when a
user says it, generically, on any project.

- **How it's invoked.** The user says something to this effect — "switch to contractor mode,"
  "full contractor mode," "go full autonomous" — at any point: before invoking `/draft-paper`,
  in the Pre-Flight Report exchange, or mid-run. It is also always offered as an explicit third
  option alongside the narrower waiver in every design-decision and general-clarification
  `AskUserQuestion` (see the two waiver paragraphs above) — worded along the lines of *"Switch to
  full contractor mode — decide this and everything like it for the rest of this run, only
  escalating genuine blockers,"* so a user answering the first escalation of either kind doesn't
  have to wait for the second kind to also opt in.
- **What it grants.** Immediately treats both waivers above as granted for the remainder of the
  conversation — every subsequent design/specification decision and every subsequent genuinely-
  ambiguous field are resolved autonomously, using the same defensible-default judgment either
  waiver alone would apply, with the same disclosure obligation (every such decision still goes
  in the Final Report's Judgment calls list — contractor mode changes who decides, never whether
  it's reported).
- **What it does not touch.** Contractor mode governs the two waivers only. It does **not**
  disable the Exit Checkpoint's own final permission ask (below) — that mechanism exists to
  catch silent premature stopping, a different failure mode than escalation frequency, and stays
  in force regardless of how much autonomy has been granted elsewhere. It also does not touch
  genuine blockers — missing/unreadable data, no research question given, a target journal with
  no reasonable fallback — those still halt and ask, contractor mode or not, because there is no
  defensible default to fall back on.
- **Discoverability.** Mention contractor mode as an available option in the Pre-Flight Report
  itself (one line is enough) so a user doesn't have to already know the phrase from an earlier
  session to use it.

## Exit checkpoint — never end this skill's turn without asking first

**This is the backstop for everything above, added because the failure mode above was observed
in practice, twice, in two different shapes.** First: a run stopped silently after Phase 1's data
analysis, with no manuscript ever drafted, requiring a fresh invocation to notice and finish. The
"keep going, do not stop" instruction added after that was necessary but was exactly the
instruction that had just failed — not sufficient alone. Second, even after that fix: a run
treated "adapting project config" as a complete phase in its own right and stopped to check in
before "the next phase" (per a standing, general check-in-cadence preference from earlier in the
project — see the precedence rule above), never reaching Phase 0's own Pre-Flight at all. This
rule is the guarantee layered on top of both:

**You may never end this skill's turn — for any reason — without an explicit permission check.**
This applies uniformly: to the legitimate completion after Phase 4, to any escalation elsewhere
in this skill, and to any other point where you find yourself about to stop — including a stop
that feels justified by a general standing preference to check in often. **A standing check-in
preference is not, by itself, a genuine blocker** (see the precedence rule above) — it is not a
reason this rule's step 2 treats as license to stop. Before ending the turn, always:

1. State exactly what is done and what is not, against the phase list (Pre-Flight; Phase 1
   steps 1–4; each Phase 2 sub-step, converged or not; Phase 3 compile; Phase 4 package).
2. If anything on that list is incomplete and there is no genuine blocker forcing a stop, **do
   not stop** — the honest self-audit itself is what prevents a silent premature exit; go back
   and finish it instead of asking permission to abandon it.
3. Only once the audit confirms the list is genuinely complete (or a genuine blocker was already
   escalated per the rules above), end the turn by asking — via `AskUserQuestion` — whether to
   consider the run finished, rather than declaring it finished unilaterally.

A silent stop is indistinguishable, from the user's side, between "finished" and "broken." This
rule makes that distinction visible in the same turn — a user who sees the permission check fire
after only Phase 1 immediately knows something is wrong and can say "continue," recovering
without a fresh invocation — instead of the ambiguous silence that caused the original failure.

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
  analysis_tool: R | Stata | Python           # where do the final tables/figures live
  outputs_path: scripts/R/_outputs/ | scripts/stata/_output/ | ...   # naming differs by language — see each language's own convention
  target_journal: [name, or "general working paper" if none]
  disclosure_policy: [from target_journal, or generic if none named]
  max_rounds: 5                                # loop-until-dry fallback cap, per orchestrator-protocol
  contractor_mode: false                       # true if the user invoked it before/during this Pre-Flight — see Continuity
```

If any required field is genuinely unresolvable from context, use `AskUserQuestion` **now** —
never mid-draft — including the general-clarification waiver option described in Continuity
above (unless that waiver was already granted earlier this conversation, in which case resolve
it yourself and note the assumption instead). Echo the resolved RUN_CONFIG back as a Pre-Flight
Report before Phase 1, and include one line noting contractor mode is available on request (per
Continuity) if it hasn't already been invoked for this run.

## Phase 1: Draft

1. **Ensure the analysis outputs exist — this is a precondition, not a deliverable.** Check
   `outputs_path` from Pre-Flight. If it's empty or missing, run
   [`/data-analysis`](../data-analysis/SKILL.md) or [`/stata-replication`](../stata-replication/SKILL.md)
   yourself, inline, now (Stata logs/intermediates go to `scripts/stata/_log/` / `_temp/` per
   [`stata-code-conventions.md`](../../rules/stata-code-conventions.md); the paper only ever
   `\input{}`s `outputs_path`, never a log or intermediate file). **The moment that pipeline finishes, continue immediately to step 2
   below in the same turn — do not stop, summarize, or hand back to the user here.** No number in
   any table/figure is ever hand-transcribed; every one comes from `\input{}`-ing `outputs_path`.

   **Check what the supplied data actually contains before treating any field as given.** If the
   Pre-Flight research question centers on a *measurement construction* (the contribution
   paragraph will say so — "we construct X," not "we apply X"), and the raw data already
   contains a precomputed version of that central quantity, decide whether to (a) derive it from
   the more primitive inputs the data also contains, writing up that derivation for
   `paper-writing-craft.md` §2 item 8's appendix, or (b) treat the supplied field as authoritative
   and validate it internally (e.g., check the field against its own stated formula from other
   supplied components) rather than re-deriving from scratch. This is exactly the kind of
   specification decision Continuity's design-decision escalation governs — by default, ask;
   under the design-decision waiver, decide it yourself and disclose the choice (and why) in
   Judgment calls. **Do not silently default to (b) just because it's less work** — (b) caps how
   deep the paper's own methodology section and appendix can honestly go, since there is no
   construction left to explain if the paper never performed one; that gap is a common,
   avoidable reason an otherwise-rigorous draft reads as thin next to a paper that built the
   measure from primitives itself.
2. Copy [`templates/paper/paper-template.tex`](../../../templates/paper/paper-template.tex) and
   [`Preambles/paper-header.tex`](../../../Preambles/paper-header.tex) into place (`Paper/` by
   default, or wherever the Pre-Flight named).
3. Apply [`paper-writing-craft.md`](../../rules/paper-writing-craft.md) in full: literature
   density via the strand-by-strand test in §1 — extended, per that section, to every assumed
   parameter and construction choice wherever it appears, not just the introduction — the
   structural checklist in §2 (including item 8's technical appendix, when step 1 above decided
   this paper's contribution needs one), and the register discipline in §3. This is the actual
   manuscript prose — the deliverable step 1 was in service of.
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
2. **Figure visual quality** — fork `figure-quality-reviewer` (`Agent`, `context: fork`) against
   every figure the manuscript includes, per [`figure-visual-quality.md`](../../rules/figure-visual-quality.md).
   Unlike the other Phase 2 checks, a confirmed finding here is fixed at the *generating*
   `.do`/`.R` script (suppress per-panel titles, set explicit text sizing on combined/faceted
   graphs, move an obscuring legend), not the manuscript source — re-run the script, re-export
   the figure, then re-run the reviewer against the new render before calling it dry. A `graph
   export`/`ggsave` exiting 0 is not evidence the figure is fine; only opening the rendered image
   is.
3. **Content review** — `/review-paper --adversarial` (or `/seven-pass-review` for a long
   draft). This skill *is* the fixer: apply every CRITICAL/MAJOR finding directly, re-run the
   review, repeat until dry.
4. **Claim verification** — `/verify-claims` (CoVe, fresh-context `claim-verifier` fork, per
   `post-flight-verification.md`). Any HIGH-WARN (fabricated citation, numeric/directional
   contradiction) is fixed and the specific claim re-verified before moving on — the existing
   must-fix policy, not a new one.
5. **Bibliography** — `/validate-bib`. Fix every structural finding (missing/unused entries,
   malformed fields) directly.
6. **Proofreading** — `/proofread` stays read-only per its own design; this skill applies every
   reported fix directly (typos, grammar, overflow, consistency), then re-runs `/proofread`,
   looping until dry — the same critic→fixer→re-audit shape as `/qa-quarto`.
7. **Prose voice** — `/humanize` stays detect-only per its own design (`writing-with-ai.md`; the
   `[LEARN]` lesson that auto-rewriting these degrades quality). Any finding it tags `mechanical`
   is fixed directly; pure AI-voice tells (hedging stacks, boilerplate transitions, tricolon
   abuse, etc.) are **not** rewritten — collect them into the Final Report's punch-list instead.
8. **Disclosures** — `/submission-disclosures`, using the target journal / disclosure policy
   named in Pre-Flight (or a generic AI-use + data-availability block if none was named).

## Phase 3: Compile + verify

Re-run the 3-pass XeLaTeX + bibtex compile after all fixes land (toolchain should already be
installed from Phase 1 — re-check "LaTeX toolchain" above if not). Confirm: PDF builds with no
errors, no overfull-hbox warnings past this repo's usual tolerance, and — visually — no
`hyperref` link boxes (the `hidelinks` default in `paper-header.tex` should make this automatic;
if a caller has swapped in `colorlinks`, re-check the rendered PDF, not just the source).

## Phase 4: Replication package

Build the deposit as part of this flow — proceed automatically, same as Phase 2/3, no separate
go-ahead. Run [`/replication-package`](../replication-package/SKILL.md) against the now-finalized
manuscript and `outputs_path` from Pre-Flight (a finalized manuscript is a precondition here —
this phase runs *after* Phase 3, not before, since the package's Table/Figure → script:line map
is read from the manuscript as it will actually ship, not a draft still being revised).

`/replication-package` itself calls `/audit-reproducibility` (its own Phase 3) and can block:

- **Audit-reproducibility FAIL (a manuscript number doesn't reproduce within tolerance).** This
  skill is still the fixer: find why (almost always a stale number that didn't get updated after
  the table/figure it cites changed), fix the manuscript to match the actual `_output`/`_outputs`
  value, re-run `/replication-package`, repeat until dry — same discipline as every Phase 2 step,
  not a new one. Never "fix" the direction by editing the script's output to match a wrong
  manuscript claim.
- **Restricted/confidential data detected with no access note (its Phase 5).** This is not a
  specification judgment call — whether data is genuinely restricted, and what the access/DUA
  process is, is a factual question about the data's provenance, not a defensible-default choice.
  Route it through Continuity's design-decision escalation (ask, with the usual session-waiver
  option) unless the source and license are already stated unambiguously elsewhere in the
  manuscript's Data section, in which case fill in the access note directly and disclose the
  choice in Judgment calls.

On a clean run (all DCAS checklist items PASS or `[FILL]`, audit PASS/EXPLAINED-only), the
package lives at `replication_package/` per that skill's own tree — report its location and any
remaining `[FILL]` items in the Final Report rather than treating an open `[FILL]` as a failure;
some fields (license choice, deposit target) are the author's call, not this skill's to invent.

## Final Report

State, in this order:

1. **What was fixed automatically**, grouped by category (content-depth, figure-quality,
   correctness/CRITICAL, correctness/MAJOR, citations, bibliography, proofreading,
   mechanical-humanize, disclosures) with counts.
2. **Judgment calls made autonomously, if either waiver (or contractor mode) was granted this
   conversation** — state which was active (design-decision waiver, general-clarification
   waiver, or full contractor mode covering both) and when it was granted, then every RUN_CONFIG
   field defaulted and every design/specification choice made without escalating (sample
   restrictions, functional form, controls, clustering, robustness checks run) as a result. This
   is the transparency the skipped escalation owes the user — not optional, even when nothing
   here is wrong. Omit this section entirely if nothing was granted and every genuinely ambiguous
   point escalated normally.
3. **The AI-voice punch-list** (if `/humanize` found anything non-mechanical) — the only items
   left requiring the user's own editorial judgment, per `writing-with-ai.md`.
4. **Remaining placeholders** — author names, institution, the acknowledgment footnote if there
   was no genuine content for it (never filled with a "working draft" disclaimer instead — see
   `paper-writing-craft.md` §4), and anything the Pre-Flight left as "general working paper"
   that a named target journal would instead fix (margins, citation style, length limits).
5. Confirmation the PDF compiles cleanly as of this report.
6. **Replication package** (Phase 4) — its location, the DCAS checklist summary, and any open
   `[FILL]` items left for the author (license choice, deposit target, restricted-data access
   note if that was escalated rather than resolved autonomously).
7. **The exit checkpoint's permission check** (see above) — end the report by asking whether to
   consider the run finished, per the self-audit against the phase list. Do not end the report
   with a bare statement of completion; end it with that question.

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
- Does not overwrite an existing manuscript without confirming — if `Paper/` (or the named
  target) already has a `.tex` file, ask before replacing it.

## Cross-references

- [`.claude/rules/paper-writing-craft.md`](../../rules/paper-writing-craft.md) — the content/structure rule this skill applies.
- [`.claude/agents/content-depth-reviewer.md`](../../agents/content-depth-reviewer.md) — the independent reviewer that checks Phase 1's draft actually complied with the rule above (Phase 2 step 1).
- [`.claude/rules/figure-visual-quality.md`](../../rules/figure-visual-quality.md) / [`.claude/agents/figure-quality-reviewer.md`](../../agents/figure-quality-reviewer.md) — the rendered-image check for overlapping/illegible figure text (Phase 2 step 2).
- [`.claude/rules/orchestrator-protocol.md`](../../rules/orchestrator-protocol.md) — the runtime (fan-out/reduce/judge/loop-until-dry) this skill's Phase 2 composes.
- [`templates/paper/paper-template.tex`](../../../templates/paper/paper-template.tex) / [`Preambles/paper-header.tex`](../../../Preambles/paper-header.tex) — the generic assets drafted into.
- [`.claude/skills/review-paper/SKILL.md`](../review-paper/SKILL.md), [`verify-claims`](../verify-claims/SKILL.md), [`proofread`](../proofread/SKILL.md), [`humanize`](../humanize/SKILL.md), [`validate-bib`](../validate-bib/SKILL.md), [`submission-disclosures`](../submission-disclosures/SKILL.md), [`replication-package`](../replication-package/SKILL.md) — composed, not modified.
