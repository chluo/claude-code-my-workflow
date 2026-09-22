---
paths:
  - "scripts/stata/**/*.do"
  - "scripts/R/**/*.R"
  - "**/_outputs/*.png"
  - "**/_outputs/*.pdf"
  - "**/_output/*.png"
  - "**/_output/*.pdf"
---

# Figure Visual Quality (Stata / R statistical graphics)

**A figure that exports without error can still be unreadable.** `graph export` and `ggsave`
exit 0 whether or not the title text collides with the panel next to it — success in the log is
not success on the page. This rule exists because that exact failure shipped: a `graph combine`
small-multiple panel's per-subgraph title, sized for a full standalone graph, repeated and
visually overlapped across every panel once squeezed into a grid.

This rule covers Stata's native `graph`/`graph combine`/`graph export` and R's `ggplot2`
(`facet_wrap`/`facet_grid`, and combine tools like `patchwork`/`cowplot`/`gridExtra`) — the same
failure class hits both, for the same reason: the layout engine sizes text for the graph as
authored, not for the size it will actually render at once combined or shrunk into the paper.

## Why source review alone cannot catch this

Unlike TikZ ([`tikz-measurement.md`](tikz-measurement.md)), there is no accessible coordinate
system in a `.do` or `.R` file to compute label collisions from formulas — Stata's and ggplot2's
text layout is a black box that composes final placement only at render time. **The only
reliable check is looking at the rendered image.** A text-only review of the script (grep for
`title(`, `facet_wrap`) can flag risk, but cannot confirm or clear a finding — that requires
opening the actual `.png`/`.pdf` output, which is exactly what
[`figure-quality-reviewer`](../agents/figure-quality-reviewer.md) does and a source-only review
does not.

## Prevention — write combined/small-multiple graphs that can't collide

### Stata: `graph combine`

- **Suppress the redundant per-subgraph title before combining**, and set one title on the
  combine call instead: build each panel with `title("")`, then
  `graph combine g1 g2 g3 ..., title("Overall title") xsize(8) ysize(6)`. A per-panel title
  repeated across N panels at full-graph font size is the single most common cause of the exact
  overlap this rule exists to prevent.
- **Set text sizes explicitly, scaled to the actual panel size** — do not rely on Stata's
  default (sized for a full standalone graph). `graph combine ..., iscale(*.7)` scales every
  panel's text/marker/line elements together; a bare `iscale` omission is a common root cause.
- **Give panels enough size to hold their own labels**: `xsize()`/`ysize()` on the combine call,
  not left to Stata's auto-fit, which will shrink panels (and overlap their contents) before it
  drops a label.
- For a small-multiples series (one panel per group — industry, year bucket, etc.), prefer
  `by(group, ...)` on a single `graph twoway`/`graph bar` call over manually building and
  combining N separate graphs — Stata's `by()` sizing is designed for exactly this case and is
  less prone to the per-panel-title duplication bug than manual `graph combine`.

### R: `facet_wrap` / `facet_grid` and combine tools

- **Use faceting's own per-panel strip labels** (`facet_wrap(~group)`) rather than looping and
  building N separate `ggtitle()`-labeled plots and combining them — faceting sizes its own
  strip text to the panel automatically; manual combination (via `patchwork`/`cowplot`) requires
  you to size titles yourself and is where the duplication bug reappears in R's ecosystem.
- If manual combination is unavoidable (heterogeneous plot types that can't share one `facet_wrap`
  call), **set `theme(plot.title = element_text(size = ...))` explicitly, scaled to the final
  combined-figure dimensions** — never the default theme size meant for a full-page single plot.
- `theme(legend.position = "bottom")` (or `"none"` with a combined legend built separately) by
  default for faceted/combined figures — an inside-plot legend that was fine on one full-size
  panel routinely covers data once panels shrink.

## Detection — the mandatory visual check

**After every `graph export` (Stata) or `ggsave` (R) that produces a figure destined for the
paper, the rendered image itself must be opened and visually inspected before the figure is
considered done** — not just confirmed to exist on disk. This is not optional for a
"publication ready" claim (see [`/draft-paper`](../skills/draft-paper/SKILL.md)'s contract).
Use [`figure-quality-reviewer`](../agents/figure-quality-reviewer.md) for this: it greps
`.do`/`.R` sources for the risk patterns above to triage which figures need the closest look,
then actually opens each rendered `.png`/`.pdf` and reports overlapping or garbled text,
illegibly small text, clipped/cut-off labels, and a legend obscuring plotted data.

## Fixing a confirmed overlap

A visual finding here is fixed at the **source** (the `.do`/`.R` script), never by cropping or
hand-editing the exported image — the image is generated output, not the artifact of record.
Apply one of the prevention patterns above, re-run the script, re-export, and re-inspect the new
render before calling the finding resolved; a fix that wasn't re-verified against the actual
re-rendered image is a guess, not a fix.

## Cross-references

- [`.claude/agents/figure-quality-reviewer.md`](../agents/figure-quality-reviewer.md) — the visual-inspection reviewer this rule requires.
- [`.claude/rules/stata-code-conventions.md`](stata-code-conventions.md) §8 — the export mechanics (vector + raster, both formats) this rule adds a visual-quality gate on top of.
- [`.claude/rules/content-invariants.md`](content-invariants.md) INV-11/INV-12 — the R-figure invariants (transparent backgrounds, project theme) this rule complements, not replaces.
- [`.claude/rules/tikz-measurement.md`](tikz-measurement.md) / [`tikz-prevention.md`](tikz-prevention.md) — the analogous discipline for TikZ, where source-level formulas make prediction possible; this rule exists because that approach does not transfer to Stata/ggplot2's opaque layout engines.
- [`.claude/skills/draft-paper/SKILL.md`](../skills/draft-paper/SKILL.md) — where this check is wired into the auto-finalize loop for a manuscript.
