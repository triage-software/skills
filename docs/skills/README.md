# Skill cards

One card per skill, with its parameters and the companion skills it works with. Each card links back to the skill's `SKILL.md` source.

**Naming convention:** the `ot-auto-*` prefix marks a skill as 🤖 **autonomous** — hand it a brief, an issue, or nothing at all and it runs end-to-end without supervision (isolated worktree, validation gate, self-review, claim locks). Every other skill is 🧑‍💻 **interactive**: it acts once, may ask you questions, reports, and hands control back.

| Skill | Type | What it does |
|---|---|---|
| [ot-auto-create-pr](ot-auto-create-pr.md) | 🤖 | Takes a free-form task brief end-to-end to a labeled, self-reviewed PR. Resumable. |
| [ot-auto-create-pr-loop](ot-auto-create-pr-loop.md) | 🤖 | Advanced create-pr for long spec builds: run folder, one commit per step, checkpoint verification. |
| [ot-auto-continue-pr](ot-auto-continue-pr.md) | 🤖 | Resumes an in-progress PR from the first unchecked step of its tracking plan. |
| [ot-auto-continue-pr-loop](ot-auto-continue-pr-loop.md) | 🤖 | Resumes a create-pr-loop run from the first non-done Tasks-table row. |
| [ot-auto-fix-issue](ot-auto-fix-issue.md) | 🤖 | The issue-to-PR entry point: classifies the issue, then drives the bug or feature route. |
| [ot-auto-fix-pr](ot-auto-fix-pr.md) | 🤖 | Drives one PR to merge-ready: base merge, review-autofix, CI stabilization, UI QA. |
| [ot-auto-write-spec](ot-auto-write-spec.md) | 🤖 | Turns a brief or feature-request issue into a finished spec on a ready PR with mockups. |
| [ot-auto-implement-spec](ot-auto-implement-spec.md) | 🤖 | Implements an existing spec and ships a reviewed, UI-verified PR. |
| [ot-auto-review-pr](ot-auto-review-pr.md) | 🤖 | Reviews or re-reviews a PR by number, with an autofix loop until merge-ready. |
| [ot-auto-qa-pr](ot-auto-qa-pr.md) | 🤖 | QAs a PR's UI in a real browser and posts screenshot evidence — no source touched. |
| [ot-auto-manage-issues](ot-auto-manage-issues.md) | 🤖 | Brings existing issues up to standard: label sync, screenshot analysis, spec-coverage checks. |
| [ot-auto-update-changelog](ot-auto-update-changelog.md) | 🤖 | Drafts a CHANGELOG release entry for merged PRs and ships it as a docs PR. |
| [ot-pr-autopilot](ot-pr-autopilot.md) | 🤖 | Diagnoses what state one open PR is really in, then runs the matching chain of the skills above. Dispatch only. |
| [ot-review-prs](ot-review-prs.md) | 🧑‍💻 | Sweeps every unreviewed open PR, newest first, through the review skill. |
| [ot-close-fixed-issues](ot-close-fixed-issues.md) | 🧑‍💻 | Post-merge housekeeping: closes issues merged PRs fixed, comments on closed-unmerged PRs. |
| [ot-merge-buddy](ot-merge-buddy.md) | 🧑‍💻 | Reports which open PRs can merge now and which are close but blocked. |
| [ot-pipeline-retro](ot-pipeline-retro.md) | 🧑‍💻 | Classifies finished runs and ranks what second passes cost, in wall-clock hours. |
| [ot-approve-merge-pr](ot-approve-merge-pr.md) | 🧑‍💻 | Approves and squash-merges a PR by number, honoring the QA gate. |
| [ot-setup-agent-pipeline](ot-setup-agent-pipeline.md) | 🧑‍💻 | One-per-repo configurator: writes the config, installs descriptors, generates project docs. |
| [ot-apply-upgrade-notes](ot-apply-upgrade-notes.md) | 🧑‍💻 | Applies UPGRADE_NOTES.md after an upgrade, preserving local edits. |
| [ot-check-and-commit](ot-check-and-commit.md) | 🧑‍💻 | Runs the validation gate on the branch, fixes obvious drift, commits and pushes when green. |
| [ot-discover](ot-discover.md) | 🧑‍💻 | Product-level discovery and define in three modes; leaves a product-brief.md built from real material, with tagged evidence and owned decisions. |
| [ot-brainstorm](ot-brainstorm.md) | 🧑‍💻 | Divergent conversation before any artifact exists; converges on which skill runs next, plus a handoff brief. |
| [ot-prepare-issue](ot-prepare-issue.md) | 🧑‍💻 | Files one well-formed, labeled tracker issue from a brief without implementing it. |
| [ot-spec-writing](ot-spec-writing.md) | 🧑‍💻 | Writes and reviews feature specs to staff-engineer standards. |
| [ot-ux-review-pr](ot-ux-review-pr.md) | 🧑‍💻 | Design-judgment review of a PR's UI: walks screens in a real browser, posts evidence-tagged findings with done-when criteria. |
| [ot-ux-setup](ot-ux-setup.md) | 🧑‍💻 | Extracts the repo's design contract (tokens, components, archetypes, conventions) into committed files. Once per repo. |
| [ot-ux-shape](ot-ux-shape.md) | 🧑‍💻 | Turns a vague feature idea into a decided direction: scope, interaction contract, validation plan; AI-necessity gate included. |
| [ot-followup-issue-from-pr](ot-followup-issue-from-pr.md) | 🧑‍💻 | Turns a PR or PR comment into a tracked follow-up issue. |
| [ot-prepare-test-env](ot-prepare-test-env.md) | 🧑‍💻 | Boots the app for QA and tests on any stack and provisions the browser provider. |
| [ot-integration-tests](ot-integration-tests.md) | 🧑‍💻 | Creates and runs integration/E2E tests by exploring the running app first. |
| [ot-create-skill](ot-create-skill.md) | 🧑‍💻 | Authors a new OM skill, or splits an oversized SKILL.md into layered references. |
| [ot-verify-in-repo](ot-verify-in-repo.md) | 🧑‍💻 | Read-only triage gate: decides whether an issue is a real, still-unfixed defect. |
| [ot-root-cause](ot-root-cause.md) | 🧑‍💻 | Read-only analysis: locates the bug and the minimal change surface. |
| [ot-fix](ot-fix.md) | 🧑‍💻 | Implements the minimal change with regression tests and runs the validation gate. |
| [ot-open-pr](ot-open-pr.md) | 🧑‍💻 | The shared PR opener: commits, pushes, opens or reuses a labeled PR, emits chain markers. |
| [ot-code-review](ot-code-review.md) | 🧑‍💻 | The review checklist behind ot-auto-review-pr: correctness, security, contract surfaces. |
