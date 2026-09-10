# ot-fix

> 🧑‍💻 Interactive — acts once, may ask questions, hands control back

Implements the minimal code change that a prior root-cause step identified, adds the regression tests that prove it, and runs the configured validation gate until it passes. Before touching anything it claims the tracker issue (assignee, in-progress label, claim comment) so other automation backs off. It stays deliberately narrow — no refactors, no scope creep — and it never commits, pushes, or opens a PR; that is handed to ot-open-pr. Use it as the fix step of an autofix chain once you already know what needs to change and where.

## Parameters

| Parameter | Required | Description |
|---|---|---|
| `issueId` | Yes | The tracker issue id to fix and claim. |
| `repo` | Optional | Repository as `owner/name`; inferred from the git remote when omitted. |

## Works with

Runs as step 3 of the autofix chain [ot-verify-in-repo](ot-verify-in-repo.md) → [ot-root-cause](ot-root-cause.md) → ot-fix → [ot-open-pr](ot-open-pr.md) → [ot-auto-review-pr](ot-auto-review-pr.md), usually driven by [ot-auto-fix-issue](ot-auto-fix-issue.md). It consumes the preceding `ot-root-cause` brief and emits a `Status: ready`/`Status: blocked` report plus a files-changed and tests summary that [ot-open-pr](ot-open-pr.md) parses to ship the work.

---
*Source: [`skills/ot-fix/SKILL.md`](../../skills/ot-fix/SKILL.md)*
