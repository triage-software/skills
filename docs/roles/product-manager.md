# 📋 Product Manager / Analyst

The pipeline turns your ideas into tracked, well-formed work and — when you want it — a written spec you can review before a single line of code exists. You describe the outcome; the skills dedupe against what already exists, link or author a covering spec, embed step-by-step guidance, and apply the full SDLC label set (category + priority + risk) so the backlog is ready for a developer to pick up. Nothing implements until you ask.

← Back to the [README](../../README.md#-workflows-by-role)

## Skills you'll use

| Skill | When | Example call | What you get |
|---|---|---|---|
| [`ot-discover`](../skills/ot-discover.md) | Establish the product context before any idea is weighed | `/ot-discover --mode client "Benefits portal"` | a `product-brief.md` built from your research folder, with every claim tagged by its evidence and every decision owned by a person — or a collection plan naming what to gather first |
| [`ot-brainstorm`](../skills/ot-brainstorm.md) | Think it through before any artifact exists | `/ot-brainstorm "should we build bulk-archive?"` | a routing decision with its reasoning, and a brief file the pipeline can run with |
| [`ot-prepare-issue`](../skills/ot-prepare-issue.md) | Park an idea as one clean issue | `/ot-prepare-issue "Bulk-archive orders from the grid"` | a deduped, SDLC-labeled issue with a linked spec or step-by-step guidance |
| [`ot-auto-manage-issues`](../skills/ot-auto-manage-issues.md) | Triage or enrich the backlog | `/ot-auto-manage-issues` | missing labels added, laconic issues clarified (screenshots analyzed), implementation-prep comment posted, feature issues without a spec flagged with a spec-required comment to their author (`--write-missing-specs` authors them instead) |
| [`ot-auto-manage-issues`](../skills/ot-auto-manage-issues.md) | Clean up one issue | `/ot-auto-manage-issues 123` | that issue relabeled and clarified in place |
| [`ot-auto-write-spec`](../skills/ot-auto-write-spec.md) | Review the plan before code | `/ot-auto-write-spec 123` | a spec-first PR; implementation left for a later run |
| [`ot-spec-writing`](../skills/ot-spec-writing.md) | Draft or review a spec directly | `/ot-spec-writing` | a staff-level spec, skeleton-first with an Open Questions gate |

## What happens automatically

- **SDLC labels on creation** — category, inferred priority, and risk are applied when the issue is filed.
- **Dedupe first** — [`ot-prepare-issue`](../skills/ot-prepare-issue.md) searches existing issues and open PRs before creating anything.
- **Spec linkage** — a covering spec in the repo or an open PR is linked; a feature that needs one gets a spec authored on a design-only PR.
- **Claim locks** — batch triage is idempotent and claim-aware, so it never clobbers work another agent is mid-flight on.
- **Implementation-prep comments** — [`ot-auto-manage-issues`](../skills/ot-auto-manage-issues.md) posts a read-only analysis as a comment; it never edits code.
- **Spec gate on feature issues** — triage checks spec coverage; a feature issue without a covering spec gets a spec-required comment addressed to its author (fill up the spec before implementation), unless you run with `--write-missing-specs` to have the specs authored on design-only PRs.
- **Spec PRs get a design review** — running [`ot-auto-review-pr`](../skills/ot-auto-review-pr.md) on a spec-only PR applies the specification lenses (what can go wrong, backward compatibility, what's missing, improvements, is it the simplest solution) instead of the code checklist — so ask for a review on the spec PR the same way you would on code.

## Tips

- Start with [`ot-brainstorm`](../skills/ot-brainstorm.md) when the idea is still fuzzy — its brief's resolved-unknowns table replaces the spec's autonomous defaults downstream, so the full-auto path runs on your answers instead of the agent's guesses.
- Use `ot-auto-write-spec <issue>` when you want to sign off on the approach before any implementation happens — it stops after the spec PR lands; a later [`ot-auto-fix-issue`](../skills/ot-auto-fix-issue.md) run picks the spec up and implements it on the same PR.
- Specs are **autonomous by default**: `ot-spec-writing --autonomous` posts its Open-Questions assumptions as a comment for you to override, rather than blocking. Pass `--interactive` (on [`ot-auto-fix-issue`](../skills/ot-auto-fix-issue.md) / [`ot-prepare-issue`](../skills/ot-prepare-issue.md)) when you'd rather answer the questions live.
- To override an autonomous assumption, just reply on the PR/issue comment — the defaults are posted precisely so you can correct them.
- Batch triage defaults to the last ~25 open issues, worst-described first; narrow it by state, label, author, or limit when you want a focused pass.
- [`ot-prepare-issue`](../skills/ot-prepare-issue.md) files work but never implements it — reach for [`ot-auto-fix-issue`](../skills/ot-auto-fix-issue.md) (or a developer) when you're ready to build.
