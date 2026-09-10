<p align="center">
  <a href="https://github.com/triage-software/open-triage">
    <img src="docs/open-triage.svg" alt="Open Triage logo" width="120" />
  </a>
</p>

<h1 align="center">Open Triage Skills</h1>

<p align="center">
  <b>🧠 plan · 🔨 implement · 🔍 review · ✅ QA gate · 🚢 merge</b><br/>
  Thirty-seven agent skills that run a full PR pipeline. Install them into any repo, with any coding agent.
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-green.svg" alt="License: MIT" /></a>
  <a href="https://skills.sh"><img src="https://img.shields.io/badge/install%20via-skills.sh-blue.svg" alt="Install via skills.sh" /></a>
  <a href="https://github.com/triage-software/skills/pulls"><img src="https://img.shields.io/badge/PRs-welcome-ff69b4.svg" alt="PRs welcome" /></a>
</p>

These skills ran a real product to completion. The workflow was proven inside the upstream [Open Mercato](https://github.com/open-mercato/skills) project, where it produced ~800k lines of code with zero hand-written lines, 1700+ merged PRs, 4000 unit tests, 730 integration tests, and weekly releases, with 100+ contributors working through it. This repository carries that pipeline into the [Open Triage](https://github.com/triage-software/open-triage) project, stripped of everything product-specific, so any team with a GitHub repo can run it.

## ⚡ 30-second quickstart

```bash
npx skills add triage-software/skills --skill '*'
```

Install all thirty-seven — the pipeline composes, and every skill is small until invoked. Drop `--skill '*'` to cherry-pick interactively. Skills install for 22+ coding agents (Claude Code, Cursor, Codex, and others) via [skills.sh](https://skills.sh).

Then, once per repository:

```
/ot-setup-agent-pipeline
```

It inspects your repo (default branch, validation scripts, GitHub labels), asks a few questions, writes `.ai/agentic.config.json`, and generates `SDLC.md` — your team's ticket-flow doc. Every other skill reads the config.

Then ship something:

```
/ot-auto-create-pr "add rate limiting to the login endpoint"
```

The agent drafts an execution plan, implements it phase by phase in an isolated worktree, runs your validation commands, reviews its own diff, and opens a labeled, reviewed PR.

## 🔄 Update an existing installation

Update project-installed skills from the project directory:

```bash
npx skills update -p
```

For skills installed globally, update the global installation instead:

```bash
npx skills update -g
```

This refreshes the installed skill files to their latest versions. It does not overwrite artifacts that the setup skill previously generated inside your repository, including `.ai/trackers/<tracker>.md` and `.ai/browsers/<provider>.md`. After updating, run:

```text
/ot-apply-upgrade-notes
```

That skill applies the relevant [UPGRADE_NOTES.md](UPGRADE_NOTES.md) migrations while preserving local edits.

ℹ️ A few skills drive a real browser through the configured browser provider — [`ot-prepare-test-env`](docs/skills/ot-prepare-test-env.md), [`ot-integration-tests`](docs/skills/ot-integration-tests.md), and [`ot-auto-qa-pr`](docs/skills/ot-auto-qa-pr.md). Because of that, skills.sh validation may flag them as **Medium** or **High** risk. We of course recommend reading any skill before you run it — but we use these exactly as shipped at Open Triage, with no issues so far.

## 🛠️ Local development

Working on the skills themselves? Skip the `npx skills add` round-trip and symlink this checkout straight into your agents' skill directories:

```bash
npm run install-skills
```

This links every skill in `skills/` into `~/.claude/skills` (Claude Code) and `~/.codex/skills` (Codex). Because they are symlinks, any edit you make in this repo is live on the next skill invocation — no reinstall needed.

Options:

```bash
npm run install-skills -- --agent claude   # only one agent (claude or codex)
npm run install-skills -- --force          # replace existing non-symlink installs
npm run uninstall-skills                   # remove only the links owned by this repo
```

The installer never touches skills it does not own: an existing real directory (e.g. installed earlier via `npx skills add`) is skipped with a warning unless you pass `--force`, and uninstall removes only symlinks that point into this checkout.

## 🔁 The pipeline

Three entry paths: hand the agent a task brief ([`ot-auto-create-pr`](docs/skills/ot-auto-create-pr.md)), a spec ([`ot-auto-write-spec`](docs/skills/ot-auto-write-spec.md) to author one, [`ot-auto-implement-spec`](docs/skills/ot-auto-implement-spec.md) to build one), or a GitHub issue ([`ot-auto-fix-issue`](docs/skills/ot-auto-fix-issue.md)). The issue path classifies first — a bug drives the autofix chain, a feature request gets its spec resolved (or autonomously written) and implemented on the same PR. All paths converge on the same review loop and the same QA gate. And when there is no artifact yet — just an idea or a question — [`ot-brainstorm`](docs/skills/ot-brainstorm.md) runs the conversation first and hands the pipeline a routing decision plus a brief. Before even that, [`ot-discover`](docs/skills/ot-discover.md) establishes the product context every later decision reads: a `product-brief.md` built from real material — interviews, data, documents — with every claim tagged by its evidence and every decision owned by a person.

The skills chain: every PR-producing skill ends with a `PR: #<number> (link: <url>)` reference line the next skill consumes, and every skill checks for a PR a previous skill already opened and continues on it instead of opening a duplicate. A completed autonomous run always leaves a **ready, fully labeled PR** (pipeline + category + priority + risk + QA meta) with a run-summary comment — and screenshots from the working app when the change is user-facing. Skills claim PRs and issues with an `in-progress` label, so concurrent agents back off instead of colliding.

```mermaid
flowchart LR
    discover["ot-discover<br/>(product context)"] --> brainstorm
    brainstorm["ot-brainstorm<br/>(conversation)"] -. "small task" .-> createPR
    brainstorm -. "feature" .-> writeSpec
    subgraph brief ["From a task brief"]
        createPR["ot-auto-create-pr"] --> reviewPR["ot-auto-review-pr"]
        reviewPR -- "changes requested" --> continuePR["ot-auto-continue-pr"]
        continuePR --> reviewPR
        reviewPR -- "approved" --> qaGate{"QA gate"}
        qaGate -- "skip-qa" --> mergePR["ot-merge-buddy /<br/>ot-approve-merge-pr"]
        qaGate -- "needs-qa" --> manualQA["manual QA"]
        manualQA -- "qa-approved" --> mergePR
    end
    subgraph issue ["From a GitHub issue: ot-auto-fix-issue classifies, then routes"]
        classify{"bug or FR?"}
        classify -- "bug" --> verifyStep["ot-verify-in-repo"]
        verifyStep --> rootCause["ot-root-cause"]
        rootCause --> applyFix["ot-fix"]
        applyFix --> openPR["ot-open-pr"]
        classify -- "feature request" --> specExists{"spec exists?"}
        specExists -- "no spec" --> writeSpec["ot-auto-write-spec<br/>(spec PR + mockups)"]
        writeSpec --> implementSpec["ot-auto-implement-spec"]
        specExists -- "spec exists" --> implementSpec
    end
    openPR --> reviewPR
    implementSpec --> reviewPR
```

## 📦 Skill catalog

📇 Per-skill cards with parameters: [docs/skills/](docs/skills/README.md)

### 🤖 Autonomous skills

**Naming convention:** the `ot-auto-*` prefix means **autonomous and non-interactive** — hand these a brief, an issue, or nothing at all and they run end-to-end without supervision: they claim their work with the `in-progress` lock so concurrent agents back off, work in isolated worktrees so your checkout stays untouched, run the validation gate, self-review, make the recommended most-reversible call themselves (documented for override) instead of stopping to ask, and finish with a PR, a review verdict, or a reconciled tracker. Safe to run on a schedule or in CI. Every skill **without** the `auto` prefix is interactive: it acts once, may ask you questions, reports, and hands control back.

| Skill | What it does autonomously |
|---|---|
| [`ot-auto-create-pr`](docs/skills/ot-auto-create-pr.md) | Takes a free-form task brief end-to-end: execution plan, isolated worktree, phase-by-phase commits, validation gate, self-review, labeled PR, then an autofix review loop until clean. Resumable. Hands runs whose plan exceeds the configured step threshold to [`ot-auto-create-pr-loop`](docs/skills/ot-auto-create-pr-loop.md) automatically. |
| [`ot-auto-create-pr-loop`](docs/skills/ot-auto-create-pr-loop.md) | Advanced ot-auto-create-pr for long spec implementations: run folder with PLAN/HANDOFF/NOTIFY, one commit per step, checkpoint verification every ~5 steps, plan-driven executor dispatch (per-step placement + model-tier hints in the plan), full gate at completion. |
| [`ot-auto-fix-issue`](docs/skills/ot-auto-fix-issue.md) | The single issue-to-PR entry point: classifies the issue first, then routes. A bug drives the autofix chain — triage gate, root-cause analysis, minimal fix with regression tests, a ready labeled PR, autofix review loop. A feature request takes the feature route — claims the issue, resolves its spec (autonomously written via [`ot-auto-write-spec`](docs/skills/ot-auto-write-spec.md) when none exists, implemented via [`ot-auto-implement-spec`](docs/skills/ot-auto-implement-spec.md)), and verifies the contract on the same PR — reviewed, UI-verified, fully labeled. For a spec without implementation, run `ot-auto-write-spec` directly. Stops cleanly when the issue is already solved or claimed. |
| [`ot-auto-write-spec`](docs/skills/ot-auto-write-spec.md) | Turns a brief or FR issue into a finished spec on a ready PR: autonomous Open-Questions defaults posted for override, UI mockups + current-app screenshots attached as PR evidence, full SDLC labels, chain markers for [`ot-auto-implement-spec`](docs/skills/ot-auto-implement-spec.md). |
| [`ot-auto-implement-spec`](docs/skills/ot-auto-implement-spec.md) | Implements an existing spec (by path, name, issue, or spec-PR number; clean stop when not found): reuses the spec PR's branch or runs [`ot-auto-create-pr`](docs/skills/ot-auto-create-pr.md), then the review autofix loop and UI verification with screenshots on the PR. |
| [`ot-auto-continue-pr`](docs/skills/ot-auto-continue-pr.md) | Resumes an in-progress PR from the first unchecked step in its tracking plan and carries it to completion — implementation, validation, review loop, summary comment. A PR with no plan (a human's, another tool's, a crashed run's) is adopted: the goal is reconstructed from its description, comments, review feedback, linked issues and diff, landed as a real plan, then executed. |
| [`ot-auto-continue-pr-loop`](docs/skills/ot-auto-continue-pr-loop.md) | Resumes runs started by [`ot-auto-create-pr-loop`](docs/skills/ot-auto-create-pr-loop.md): orients from HANDOFF.md, picks up at the first non-done Tasks-table row, keeps the per-step commit and checkpoint discipline to completion. |
| [`ot-auto-review-pr`](docs/skills/ot-auto-review-pr.md) | Reviews a PR by number in an isolated worktree, approves or requests changes, manages labels. On changes-requested, its autofix loop iterates fixes and re-review until merge-ready. A spec-only design PR gets a **specification review** instead of the code checklist: what can go wrong, backward compatibility, what's missing, how the spec can be improved, and whether it is the simplest possible solution — same severity scale and verdict rule, and the autofix loop amends the spec document (never adds implementation). |
| [`ot-auto-fix-pr`](docs/skills/ot-auto-fix-pr.md) | Drives one PR to merge-ready: merges the latest base in first, then loops review-autofix ([`ot-auto-review-pr`](docs/skills/ot-auto-review-pr.md)), its own CI-stabilization step (classify each failing check as real bug / test bug / flake / infra, fix the real ones with tests, never fake green), and UI QA ([`ot-auto-qa-pr`](docs/skills/ot-auto-qa-pr.md)), re-merging base whenever it advances. Files follow-up issues for non-blocking nits via [`ot-followup-issue-from-pr`](docs/skills/ot-followup-issue-from-pr.md), keeps the fork carry-forward supersede/credit rules, normalizes labels, and hands off to [`ot-approve-merge-pr`](docs/skills/ot-approve-merge-pr.md) — it never merges itself. A `--ci-only [--branch <name>]` mode drives a plain branch or no-PR change to green CI without the rest of the loop. |
| [`ot-pr-autopilot`](docs/skills/ot-pr-autopilot.md) | The single "just finish this PR" entry point: diagnoses what state one open PR is actually in — unfinished plan steps, missing review, unresolved conversations, red CI, base conflicts, missing labels or QA evidence, merge-ready — then maps that onto an ordered chain of the skills above and runs it, re-diagnosing between steps. Dispatch only: [`ot-auto-continue-pr`](docs/skills/ot-auto-continue-pr.md), [`ot-auto-fix-pr`](docs/skills/ot-auto-fix-pr.md), [`ot-auto-qa-pr`](docs/skills/ot-auto-qa-pr.md), [`ot-followup-issue-from-pr`](docs/skills/ot-followup-issue-from-pr.md) and [`ot-approve-merge-pr`](docs/skills/ot-approve-merge-pr.md) do the work. Never merges without `--allow-merge`; `--dry-run` diagnoses and mutates nothing. |
| [`ot-review-prs`](docs/skills/ot-review-prs.md) | Sweeps all unreviewed open PRs, newest first, through [`ot-auto-review-pr`](docs/skills/ot-auto-review-pr.md), respecting claim locks. |
| [`ot-close-fixed-issues`](docs/skills/ot-close-fixed-issues.md) | Post-merge housekeeping sweep: closes issues that merged PRs fix, comments on issues whose PRs were closed without merging. |

### 🧑‍💻 Interactive skills

Interactive helpers (no `auto` in the name — the other half of the naming convention): they act once, may ask you questions along the way, report, and hand control back to you.

| Skill | What it does |
|---|---|
| [`ot-setup-agent-pipeline`](docs/skills/ot-setup-agent-pipeline.md) | One-per-repo configurator. Inspects the repository, asks a few questions, writes `.ai/agentic.config.json`, installs tracker and browser-provider descriptors, generates `SDLC.md` and an `AGENTS.md` starter when missing. Verifies cross-skill coverage: if an installed skill references one that isn't installed, it prints the exact `npx skills add` command to fix it. |
| [`ot-apply-upgrade-notes`](docs/skills/ot-apply-upgrade-notes.md) | Post-upgrade migrator. Applies `UPGRADE_NOTES.md` to the repo: re-syncs installed tracker/browser descriptors while preserving local edits, reports custom-provider gaps, and checks the config against notable upgrades. |
| [`ot-merge-buddy`](docs/skills/ot-merge-buddy.md) | Scans open PRs and reports which can merge now and which are close but blocked, based on labels, reviews, CI, and mergeability. |
| [`ot-pipeline-retro`](docs/skills/ot-pipeline-retro.md) | Classifies runs the pipeline already finished — clean single pass, hard recovery, loop checkpoints, or a second pass with no recorded cause — and ranks the causes by the wall-clock hours they cost. Read-only; hands the top cause to `ot-prepare-issue`. |
| [`ot-approve-merge-pr`](docs/skills/ot-approve-merge-pr.md) | Approves and squash-merges a PR given only its number. Can file a follow-up issue at the same time. |
| [`ot-check-and-commit`](docs/skills/ot-check-and-commit.md) | Runs the configured validation gate on the current branch, fixes obvious drift, then commits and pushes when green. |
| [`ot-followup-issue-from-pr`](docs/skills/ot-followup-issue-from-pr.md) | Turns a PR or a PR comment into a tracked follow-up issue, assigned to the right person. |
| [`ot-discover`](docs/skills/ot-discover.md) | Product-level discovery and define, before there is anything to brainstorm about. Runs in three modes — existing product, client idea, own idea — and leaves one `product-brief.md`: problem and who has it, stakeholders, rules, flows, benchmark, success criteria, scope (now, later, not doing), non-goals, decisions with owners, riskiest assumptions with tests, open questions. Gathers real material first: a section with nothing behind it becomes a collection plan with capture templates, never prose; synthetic personas and assumptions are tagged and never count as evidence. [`ot-brainstorm`](docs/skills/ot-brainstorm.md), [`ot-spec-writing`](docs/skills/ot-spec-writing.md), and [`ot-prepare-issue`](docs/skills/ot-prepare-issue.md) read the brief when it exists, and its non-goals, business rules, and decisions become a contract the review skills enforce. |
| [`ot-brainstorm`](docs/skills/ot-brainstorm.md) | The conversation before any artifact exists: open questions one at a time, alternatives weighed (including building nothing), a challenger subagent attacks the conclusion, then the user confirms a routing decision — a machine-parsed `Next:` line plus a handoff brief that feeds [`ot-prepare-issue`](docs/skills/ot-prepare-issue.md), [`ot-auto-write-spec`](docs/skills/ot-auto-write-spec.md), [`ot-spec-writing`](docs/skills/ot-spec-writing.md), or [`ot-auto-create-pr`](docs/skills/ot-auto-create-pr.md). |
| [`ot-spec-writing`](docs/skills/ot-spec-writing.md) | Writes and reviews feature specs to staff-engineer standards: skeleton-first with a hard Open Questions gate, phased implementation breakdown that feeds [`ot-auto-create-pr`](docs/skills/ot-auto-create-pr.md), severity-ranked architectural reviews. |
| [`ot-prepare-issue`](docs/skills/ot-prepare-issue.md) | Files a single well-formed tracker issue for deferred work: dedupes against existing issues and PRs, links (or authors) a covering spec, otherwise embeds step-by-step guidance, and applies the SDLC labels on creation. |
| [`ot-auto-manage-issues`](docs/skills/ot-auto-manage-issues.md) | Brings existing issues up to standard, single or in bulk: applies missing SDLC labels, and for a laconic issue (one line + a screenshot) analyzes the screenshot with the terse text, clarifies the wording non-destructively, and posts the agent's understanding as a comment. Checks spec coverage for feature issues: when one lacks a covering spec it posts a spec-required comment to the issue author (fill up the spec before implementation), or authors the spec itself via [`ot-auto-write-spec`](docs/skills/ot-auto-write-spec.md) with `--write-missing-specs` (default off). Batch defaults to the last ~25 open, worst-described first, narrowable by state/label/author/limit. Idempotent and claim-aware. |
| [`ot-integration-tests`](docs/skills/ot-integration-tests.md) | Creates and runs integration/E2E tests by exploring the running app first — real locators, runtime fixtures, no hardcoded IDs — and reports failures with artifact-based per-test diagnosis. Reuses the shared [`ot-prepare-test-env`](docs/skills/ot-prepare-test-env.md) instance so QA and tests hit the same booted app. |
| [`ot-auto-qa-pr`](docs/skills/ot-auto-qa-pr.md) | QAs a change's UI in a real browser without merging. Checks the PR's review state first and runs [`ot-auto-review-pr`](docs/skills/ot-auto-review-pr.md) when the PR is still unreviewed, then boots the app via [`ot-prepare-test-env`](docs/skills/ot-prepare-test-env.md), derives a scenario from the diff, drives the configured browser provider with screenshots, and produces a pass/fail report. Posts evidence as a PR comment when a tracker is configured; otherwise saves screenshots + JSON/Markdown reports. |
| [`ot-auto-update-changelog`](docs/skills/ot-auto-update-changelog.md) | Drafts a CHANGELOG.md release entry for every PR merged since the last release — emoji categories, contributor credits resolved by the Supersede Credit Rule and verified against commit authorship so carry-forwards and umbrella merges credit the author, not the merger — then delegates to [`ot-auto-create-pr`](docs/skills/ot-auto-create-pr.md) to ship it as a docs PR. |

### 🤝 Skills invoke each other

The building blocks behind the autofix chain and the review loop. You can call them directly, but they mainly exist for the other skills to compose.

| Skill | What it does |
|---|---|
| [`ot-verify-in-repo`](docs/skills/ot-verify-in-repo.md) | Read-only triage gate: decides whether a GitHub issue is a real, still-unfixed defect, and stops the chain cleanly when there is nothing to do. |
| [`ot-root-cause`](docs/skills/ot-root-cause.md) | Read-only analysis: locates the bug and the minimal change surface so the fix step never re-explores the repo. |
| [`ot-fix`](docs/skills/ot-fix.md) | Implements the minimal change, adds regression tests, runs the validation gate. Does not commit or push. |
| [`ot-open-pr`](docs/skills/ot-open-pr.md) | The shared PR opener: commits, pushes, opens (or reuses) a ready PR with the unified body template, applies the full SDLC label set, posts the run summary, releases the claim lock, and emits the chain markers. |
| [`ot-code-review`](docs/skills/ot-code-review.md) | The review checklist behind [`ot-auto-review-pr`](docs/skills/ot-auto-review-pr.md): correctness, security, contract surfaces, plus your repo-local checklist when configured. |
| [`ot-prepare-test-env`](docs/skills/ot-prepare-test-env.md) | Boots the app for QA and tests, any stack: reuses the repo's own environment or generates portable bring-up scripts, then caches builds and validates warm reuse. It autonomously provisions the configured browser provider (agent-browser by default; Playwright supported), writes a shared environment descriptor, and works on macOS, Linux, WSL2, and Windows. |

## 👥 Workflows by role

Same pipeline, different entry points. Each role runs one or two commands; the skills chain the rest automatically. Deeper guides live under [docs/roles/](docs/roles/).

### 📋 Product Manager / Analyst

Turn ideas into well-formed, labeled work — and review the plan before any code is written.

| ▶️ You run | ⚙️ Runs automatically inside | 🎁 You get |
|---|---|---|
| `/ot-discover --mode client "Benefits portal for SMB clients"` | context gate over your research folder, interview rounds, a skeptic subagent, a quality gate against invented evidence | `product-brief.md` with tagged evidence and owned decisions, or a collection plan naming what still has to be gathered |
| `/ot-brainstorm "should we build bulk-archive?"` | read-only repo reading and tracker checks, a challenger subagent | a routing decision with its reasoning, and a brief file the pipeline can run with |
| `/ot-prepare-issue "Bulk-archive orders from the grid"` | dedupe search, [`ot-spec-writing`](docs/skills/ot-spec-writing.md) (when a feature needs a spec) | one well-formed issue with SDLC labels, a linked spec or step-by-step guidance |
| `/ot-auto-manage-issues` | claim-aware label sync, screenshot analysis, implementation-prep comment, spec-coverage check | the backlog triaged: missing labels added, laconic issues clarified, feature issues without a spec get a spec-required comment to their author (or a spec via `--write-missing-specs`) |
| `/ot-auto-write-spec 123` | `ot-spec-writing --autonomous`, [`ot-open-pr`](docs/skills/ot-open-pr.md) | a spec-first PR to review before implementation starts |

More: [docs/roles/product-manager.md](docs/roles/product-manager.md)

### 🎨 Designer

Get a written spec with visuals attached — mockups of the new layout next to screenshots of the current app.

| ▶️ You run | ⚙️ Runs automatically inside | 🎁 You get |
|---|---|---|
| `/ot-auto-write-spec "Redesign the checkout summary panel"` | `ot-spec-writing --autonomous`, [`ot-open-pr`](docs/skills/ot-open-pr.md), [`ot-prepare-test-env`](docs/skills/ot-prepare-test-env.md) + browser provider | a ready spec PR with UI mockups, current-app screenshots, and an assumptions comment |
| `/ot-auto-implement-spec 2026-07-18-checkout-redesign` | [`ot-auto-create-pr`](docs/skills/ot-auto-create-pr.md), [`ot-auto-review-pr`](docs/skills/ot-auto-review-pr.md), [`ot-auto-qa-pr`](docs/skills/ot-auto-qa-pr.md) | the built change with before/after screenshots from the working app |
| `/ot-auto-qa-pr 123` | [`ot-prepare-test-env`](docs/skills/ot-prepare-test-env.md), browser provider | fresh screenshots of a PR's UI to design-review, no source touched |
| `/ot-ux-setup` once, then `/ot-ux-review-pr 123` | [`ot-ux-setup`](docs/skills/ot-ux-setup.md) extracts the repo's design contract; [`ot-ux-review-pr`](docs/skills/ot-ux-review-pr.md) walks the PR in a real browser | a design review judged against your own design system: evidence-tagged findings with done-when criteria |
| `/ot-ux-shape "Quick-add flow for the people list"` | [`ot-ux-shape`](docs/skills/ot-ux-shape.md) | a decided direction before anything is drawn: scope, states, riskiest-assumption test |

💡 Tip — ask for visuals explicitly to force mockups: `/ot-auto-write-spec "Redesign the checkout summary panel — include mockups of the new layout and screenshots of the current one"`.

More: [docs/roles/designer.md](docs/roles/designer.md)

### 👩‍💻 Developer

Hand off a brief, a spec, or an issue number; get back a reviewed, labeled PR.

| ▶️ You run | ⚙️ Runs automatically inside | 🎁 You get |
|---|---|---|
| `/ot-auto-write-spec "CSV export for the orders grid"` | `ot-spec-writing --autonomous`, [`ot-open-pr`](docs/skills/ot-open-pr.md), browser provider for mockups | a ready spec PR with mockups + assumptions comment |
| `/ot-auto-implement-spec 2026-07-18-csv-export` | [`ot-auto-create-pr`](docs/skills/ot-auto-create-pr.md) / [`ot-auto-continue-pr`](docs/skills/ot-auto-continue-pr.md), [`ot-auto-review-pr`](docs/skills/ot-auto-review-pr.md), [`ot-auto-qa-pr`](docs/skills/ot-auto-qa-pr.md) | an implemented, reviewed PR with screenshots from the working app |
| `/ot-auto-fix-issue 123` | classifies then routes: bugs to the autofix chain, features to [`ot-auto-write-spec`](docs/skills/ot-auto-write-spec.md) + [`ot-auto-implement-spec`](docs/skills/ot-auto-implement-spec.md) | a finished, fully-labeled PR from an issue number |
| `/ot-auto-fix-issue 456` | [`ot-verify-in-repo`](docs/skills/ot-verify-in-repo.md), [`ot-root-cause`](docs/skills/ot-root-cause.md), [`ot-fix`](docs/skills/ot-fix.md), [`ot-open-pr`](docs/skills/ot-open-pr.md), [`ot-auto-review-pr`](docs/skills/ot-auto-review-pr.md) | a bug-fix PR with regression tests and a clean review |
| 🔁 `/ot-auto-create-pr-loop "Implement the multi-tenant billing spec"` | run folder (PLAN/HANDOFF/NOTIFY), per-step commits, checkpoint verification | a resumable, step-tracked PR for a large spec (continue with [`ot-auto-continue-pr-loop`](docs/skills/ot-auto-continue-pr-loop.md); plain runs escalate here on their own past the step threshold) |

More: [docs/roles/developer.md](docs/roles/developer.md)

### 🧪 QA

Boot the app once, verify UI changes in a real browser, and add integration coverage — without touching source.

| ▶️ You run | ⚙️ Runs automatically inside | 🎁 You get |
|---|---|---|
| `/ot-prepare-test-env` | app discovery, launch-script generation, browser-provider provisioning | a reusable booted app + shared test-env descriptor the other QA skills reuse |
| `/ot-auto-qa-pr 123` | [`ot-prepare-test-env`](docs/skills/ot-prepare-test-env.md), browser provider | screenshots + a pass/fail report posted on the PR (evidence only, no labels changed) |
| `/ot-auto-qa-pr 123 --self-qa-signoff` | same, plus label guards | `qa-approved` + `qa-self-verified` — only on a fully-green run with screenshots on a `needs-qa` PR |
| `/ot-integration-tests` | [`ot-prepare-test-env`](docs/skills/ot-prepare-test-env.md), browser provider | integration/E2E tests written against the live app, with artifact-based failure diagnosis |

More: [docs/roles/qa.md](docs/roles/qa.md)

### 🚀 Release Manager

Sweep open PRs, drive them to merge-ready, and ship — the QA gate stays a human decision.

| ▶️ You run | ⚙️ Runs automatically inside | 🎁 You get |
|---|---|---|
| `/ot-merge-buddy` | tracker scan of labels, reviews, CI, mergeability | a report of which PRs can merge now and which are close but blocked |
| `/ot-review-prs` | [`ot-auto-review-pr`](docs/skills/ot-auto-review-pr.md) per PR, claim-lock aware | every unreviewed open PR reviewed, newest first |
| `/ot-auto-fix-pr 123` | [`ot-auto-review-pr`](docs/skills/ot-auto-review-pr.md), its CI-stabilization step, [`ot-auto-qa-pr`](docs/skills/ot-auto-qa-pr.md), [`ot-followup-issue-from-pr`](docs/skills/ot-followup-issue-from-pr.md) | one PR driven to approvable, green, QA-evidenced — handed to [`ot-approve-merge-pr`](docs/skills/ot-approve-merge-pr.md), never self-merged |
| `/ot-auto-fix-pr 123 --ci-only` | tracker check status + failed-step logs | green CI from real fixes with tests, never by weakening checks |
| `/ot-pr-autopilot 123` | diagnosis of the PR's real state, then the matching chain of the skills above | one PR driven from wherever it is to merge-ready, with a summary comment covering every step — never merged unless `--allow-merge` |
| `/ot-auto-update-changelog` | [`ot-auto-create-pr`](docs/skills/ot-auto-create-pr.md) | a CHANGELOG release entry landed as a docs PR, with Supersede Credit |
| `/ot-approve-merge-pr 123` | approving review + squash-merge, QA-gate guard | the PR merged — refused when `needs-qa` lacks `qa-approved` or a blocking label is set |

More: [docs/roles/release-manager.md](docs/roles/release-manager.md)

## 🧰 Works with any stack

Nothing here assumes JavaScript, or any particular product. The base branch, the validation commands, the label taxonomy, and the working paths all come from one committed file, `.ai/agentic.config.json`, written by [`ot-setup-agent-pipeline`](docs/skills/ot-setup-agent-pipeline.md):

```json
{
  "version": 1,
  "baseBranch": "auto",
  "tracker": "github",
  "browser": { "provider": "agent-browser" },
  "validation": {
    "commands": ["pnpm typecheck", "pnpm test", "pnpm build"]
  },
  "labels": {
    "enabled": true,
    "pipeline": ["review", "changes-requested", "qa", "qa-failed", "merge-queue", "blocked", "do-not-merge"],
    "category": ["bug", "feature", "refactor", "security", "dependencies", "documentation"],
    "meta": ["needs-qa", "skip-qa", "qa-approved", "qa-self-verified", "in-progress"],
    "priority": ["priority-low", "priority-medium", "priority-high", "priority-extreme"],
    "risk": ["risk-low", "risk-medium", "risk-high"]
  },
  "qaGate": true,
  "paths": {
    "runs": ".ai/runs",
    "analysis": ".ai/analysis",
    "specs": ".ai/specs",
    "scripts": ".ai/scripts",
    "qa": ".ai/qa"
  },
  "reviewChecklist": null,
  "closeKeywords": []
}
```

A Rust repo puts `cargo test` and `cargo clippy` in `validation.commands`; a Go repo puts `go test ./...`. Skills run whatever you configure and treat any non-zero exit as a gate failure. A skill invoked in a repo without the config runs [`ot-setup-agent-pipeline`](docs/skills/ot-setup-agent-pipeline.md) first — interactively when you're there to answer its questions, with `--defaults` when running unattended — then continues with the freshly written config.

GitHub is the default tracker. Shipped split-provider templates also support Linear issues through `schpet/linear-cli` and Jira Cloud work items through Atlassian CLI, while GitHub continues to own PRs, reviews, and CI — see the tracker providers section below.

Agent-browser is the default browser automation provider for fresh setups. It
installs itself and Chrome for Testing when needed; existing repositories remain
on Playwright until their config makes a provider explicit.

## 🎨 Make it yours

Four layers of project fit, no forking:

- **Agent instructions** — skills read your `AGENTS.md` / `CLAUDE.md` before working, so project conventions apply from the first run. No such file? [`ot-setup-agent-pipeline`](docs/skills/ot-setup-agent-pipeline.md) offers a starter.
- **Generated project docs** — `SDLC.md` (the process doc), `CODE_REVIEW.md` (review rules, auto-applied by [`ot-code-review`](docs/skills/ot-code-review.md)), `BACKWARD_COMPATIBILITY.md` (protected contract surfaces — reviews flag violations, implementations warn you), and an `AGENTS.md` starter with a task-routing table. [`ot-setup-agent-pipeline`](docs/skills/ot-setup-agent-pipeline.md) derives each from your repository and only when the file is missing; existing docs are honored as-is.
- **Repo-local skills** — drop a skill with the same name into your repo at `.ai/skills/<skill-name>/SKILL.md` and it takes precedence over the installed one (details below).
- **Tracker descriptor** — every issue/PR/label command the skills run lives in one committed file, `.ai/trackers/<tracker>.md`, that you can edit or replace (details below).

## 🧩 Extending the skills

### How a skill is laid out

Each skill keeps its numbered main algorithm in `SKILL.md` and factors its repeatable procedures into per-skill `references/<step>.md` files under standard names — `agentic-setup.md`, `worktree-setup.md`, `claim-pr.md`, `pr-finalize.md`, `review-report.md`, `rules.md`. These standard step files are deliberately **duplicated inside every skill that uses them** rather than shared through cross-skill pointers, so each skill installs and runs standalone ([`ot-auto-create-pr`](docs/skills/ot-auto-create-pr.md) holds the canonical copy). The trade-off is intentional: standalone installability over DRY. When you edit a standard step file in one skill, sync the same change into the other skills that carry it — the collection's own contributor rule is to ask whether to propagate before doing so.

### Repo-local skill overrides

Every installed skill checks, right after loading the config, for a repo-local skill of the same name at `.ai/skills/<skill-name>/SKILL.md`. When present, the local skill wins — the installed one follows it instead of its own instructions. To *extend* rather than replace, the local skill just `@`-imports or references the installed skill and adds rules on top:

```markdown
<!-- .ai/skills/ot-auto-review-pr/SKILL.md -->
Follow the installed `ot-auto-review-pr` skill, plus:

- Also run `pnpm test:e2e` before approving PRs that touch `apps/web`.
- Our PR body template additionally requires a "Screenshots" section for UI changes.
```

Local rules win, but a local skill can never relax the installed skill's safety rules (no skipping tests, no `--no-verify`, no force-pushes). This convention is also what makes the collection a drop-in for repos that already keep specialized `ot-*` skills under `.ai/skills/`: the installed skills defer to them automatically.

### Project management (tracker) providers

No skill calls `gh` — or any tracker CLI — directly. Skills name **tracker operations** (**get-issue**, **create-pr**, **comment-pr**, **merge-pr**, …) and one committed descriptor file, `.ai/trackers/<tracker>.md`, defines how each operation is executed. [`ot-setup-agent-pipeline`](docs/skills/ot-setup-agent-pipeline.md) asks which tracker you use, sets the config's `tracker` field, and installs the matching descriptor into your repo.

That file is yours, which makes three things easy:

- **Extend or override GitHub behavior** — edit `.ai/trackers/github.md`: add flags, change the merge strategy, adjust comment conventions, extend the label taxonomy commands. Every skill picks it up on its next run.
- **Use a shipped split provider** — select `linear` for Linear issues through [`schpet/linear-cli`](https://github.com/schpet/linear-cli), or `jira` for Jira Cloud work items through [Atlassian CLI](https://developer.atlassian.com/cloud/acli/guides/introduction/). Setup installs the selected descriptor plus `github.md`, because the code host still owns PRs, reviews, CI, and PR labels. The templates document authentication, issue-label semantics, claim signals, identifier cross-links, and explicit post-merge issue transitions.
- **Bring your own tracker** — write `.ai/trackers/<name>.md` from the shipped `TEMPLATE.md` (in `ot-setup-agent-pipeline/references/trackers/`), implementing each operation with your tracker's CLI, MCP tools, or API, and set `"tracker": "<name>"` in the config. No skill changes needed — the descriptor is the whole integration surface.
- **Build another split setup** — implement issue operations against the project tracker and delegate repository/PR/review/CI/PR-label operations to its code-host companion. The template documents the pattern, including how identifiers cross-link (for example, an `ENG-123` ticket referenced from a GitHub PR).

The claim protocol (assignee + `in-progress` + 🤖 comment), the label guards (missing label ⇒ logged skip, `labels.enabled: false` ⇒ no label ops), and the QA gate semantics are part of the contract — a provider must express them, in whatever way its tracker allows.

### Browser automation providers

QA and integration-test skills select `browser.provider` from
`.ai/agentic.config.json` and execute the committed descriptor at
`.ai/browsers/<provider>.md`. Fresh setups use agent-browser; Playwright remains
available for existing repositories and teams that prefer it. The agent-browser
descriptor downloads its native release binary and Chrome for Testing itself,
then verifies a live headless launch — no Node runtime, project dependency, or
cloud-browser subscription is required.

Custom providers implement the operations in
`skills/ot-setup-agent-pipeline/references/browsers/TEMPLATE.md`. Repository E2E
suites remain authoritative; the provider controls agent-driven exploration,
assertions, and screenshots.

## 🏷️ Labels and the QA gate

Every PR carries at most one pipeline label (`review`, `changes-requested`, `merge-queue`, ...) plus additive category, meta, priority, and risk labels; priority says how urgent the work is, risk says how dangerous the change is to ship. The full taxonomy, and whether to use labels at all, lives in the config; [`ot-setup-agent-pipeline`](docs/skills/ot-setup-agent-pipeline.md) documents every group and creates missing labels for you.

The QA gate is the one hard rule: a PR labeled `needs-qa` cannot merge until a human adds `qa-approved`, no matter how green the checks are. Automated skills request QA; they never grant it.

## 🚀 Built with this workflow

<!-- PROOF: case studies land here before launch -->

Real production case studies are being added here.

---

Built by the [Open Triage](https://github.com/triage-software/open-triage) team, where these skills ship the product every week. We teach this way of working at [aitechleaders.pl](https://aitechleaders.pl) (an AI engineering course, in Polish).
