# ot-prepare-issue

> 🧑‍💻 Interactive — acts once, may ask questions, hands control back

Creates a single, well-formed tracker issue from a brief without implementing anything. It first dedupes against existing issues and open PRs, then either links a covering spec (authoring one via `ot-auto-write-spec` on a design-only PR when a feature genuinely needs it) or embeds concrete, codebase-derived step-by-step guidance so a future run can pick the work up cold. User-provided screenshots are attached as issue evidence, and the SDLC labels (category, priority, risk) are applied on creation. Use it to file an issue for X or park an idea for later; to enrich issues that already exist, use `ot-auto-manage-issues` instead.

## Parameters

| Parameter | Required | Description |
|---|---|---|
| `brief` | Yes | Free-form description of the feature, fix, or task to capture. |
| `--priority <low\|medium\|high\|extreme>` | Optional | Override the inferred priority label. |
| `--risk <low\|medium\|high>` | Optional | Override the inferred risk label for the eventual change's blast radius. |
| `--assignee <login>` | Optional | Assign the issue; default is unassigned. |
| `images` | Optional | Screenshots or mockups (pasted or file paths) attached to the issue as evidence. |

## Works with

Emits one tracker issue (with SDLC labels), and on the spec-needed path also a design-only spec PR (emitting the `Spec:` and `PR:` reference lines) by delegating to [ot-auto-write-spec](ot-auto-write-spec.md). It hands off to [ot-spec-writing](ot-spec-writing.md) when a full spec is wanted, to [ot-auto-create-pr](ot-auto-create-pr.md) or [ot-auto-fix-issue](ot-auto-fix-issue.md) when the work should be done now, and to [ot-auto-manage-issues](ot-auto-manage-issues.md) for enriching existing issues.

---
*Source: [`skills/ot-prepare-issue/SKILL.md`](../../skills/ot-prepare-issue/SKILL.md)*
