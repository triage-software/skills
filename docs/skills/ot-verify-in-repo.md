# ot-verify-in-repo

> 🧑‍💻 Interactive — acts once, may ask questions, hands control back

A fast, read-only triage gate that decides whether a tracker issue is a real, still-unfixed defect on the current branch before any fixing work begins. It checks whether the issue is already closed, already claimed by someone else, already covered by an open or merged PR, or simply not a bug — and stops the chain cleanly with a `NO_ACTION_NEEDED` verdict when so. It never edits, commits, claims, or pushes. Use it as the first gate before spending effort on root-causing and fixing.

## Parameters

| Parameter | Required | Description |
|---|---|---|
| `{issueId}` | Yes | The GitHub issue number, e.g. `1234`. |
| `{repo}` | Optional | `owner/name`; inferred from the current git remote if omitted. |

## Works with

Step 1 of the autofix chain: when it says go, [ot-root-cause](ot-root-cause.md) reads the code, then [ot-fix](ot-fix.md) makes edits and [ot-open-pr](ot-open-pr.md) opens the PR; when it emits `NO_ACTION_NEEDED`, none of that runs. The chain is driven end-to-end by [ot-auto-fix-issue](ot-auto-fix-issue.md). It only reads claim signals via read-only tracker operations — it never claims the issue itself.

---
*Source: [`skills/ot-verify-in-repo/SKILL.md`](../../skills/ot-verify-in-repo/SKILL.md)*
