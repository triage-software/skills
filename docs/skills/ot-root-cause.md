# ot-root-cause

> 🧑‍💻 Interactive — acts once, may ask questions, hands control back

Read-only root-cause analysis for a tracker issue that has already been confirmed as a real defect. It traces the code path behind the reported behavior, pinpoints the smallest module or function that owns the bug, and defines the minimal change set — without editing, committing, or refactoring. The output is a tight report (summary, root cause, files to change, approach, risks) so the next agent can implement the fix without re-exploring the repo. Use it as the analysis step between triage and the actual fix.

## Parameters

| Parameter | Required | Description |
|---|---|---|
| `{issueId}` | Yes | The issue number in the tracker. |
| `{repo}` | Optional | `owner/name`; inferred from the git remote if omitted. |

## Works with

Step 2 of the autofix chain: it follows [ot-verify-in-repo](ot-verify-in-repo.md) (which confirms the defect is real) and hands its report to [ot-fix](ot-fix.md), which implements the proposed change. The whole chain is driven end-to-end by [ot-auto-fix-issue](ot-auto-fix-issue.md). Reads the issue via the tracker's read-only get-issue operation only.

---
*Source: [`skills/ot-root-cause/SKILL.md`](../../skills/ot-root-cause/SKILL.md)*
