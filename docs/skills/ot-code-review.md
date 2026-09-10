# ot-code-review

> 🧑‍💻 Interactive — acts once, may ask questions, hands control back

Reviews a diff, branch, or PR against correctness, security, breaking-change, and quality standards. It runs the configured validation gate (every failure is a blocker), applies a built-in review checklist plus any repo-local `reviewChecklist`, `CODE_REVIEW.md`, and `BACKWARD_COMPATIBILITY.md` rules, and produces severity-ranked findings (blocker / major / minor / nit) with a mechanical approve or request-changes verdict. It is the shared review engine used across the pipeline. Use it whenever you want a rigorous review of a change unit.

## Parameters

This skill takes no parameters.

## Works with

Accepts one review unit — a PR number, a branch, an explicit range/diff, or nothing (defaulting to the current branch's diff). It is the review engine invoked by [ot-auto-review-pr](ot-auto-review-pr.md) and [ot-review-prs](ot-review-prs.md), and by the self-review steps of [ot-auto-create-pr](ot-auto-create-pr.md) and [ot-auto-continue-pr](ot-auto-continue-pr.md), which consume its verdict and blocker/major findings.

---
*Source: [`skills/ot-code-review/SKILL.md`](../../skills/ot-code-review/SKILL.md)*
