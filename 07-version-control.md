# 🌳 Version Control — Production Engineering Guide

> How production teams use version control as the backbone of the SDLC: Git concepts, the Git 3.0 transition, repository strategy (monorepo vs polyrepo), branching strategies, commit standards (Conventional Commits), pull requests and code review, merge strategies and merge queues, branch protection and rulesets, tags/releases/changelogs, hotfixes, large files, secrets in history, history hygiene and recovery, versioning everything (IaC, migrations, docs, data), compliance, and scenario playbooks.
>
> **Setup** (install, config, SSH keys, signing, hosts, CLI) lives in **[42 — Git, GitHub, GitLab & Bitbucket Setup](./42-Git-GitHub-GitLab-Bitbucket-Setup.md)**. This chapter covers **strategy, workflow, and governance**.
>
> Related: [02 SDLC](./02-software-development-lifecycle.md) · [06 Coding Standards](./06-coding-standards.md) · [16 DevOps & CI/CD](./16-devops-and-ci-cd.md)

---

## 📚 Table of Contents

1. [Why Version Control Is the Backbone](#1-why-version-control-is-the-backbone)
2. [Git Mental Model](#2-git-mental-model)
3. [Git Versions and the Git 3.0 Transition](#3-git-versions-and-the-git-30-transition)
4. [Repository Strategy — Monorepo vs Polyrepo](#4-repository-strategy--monorepo-vs-polyrepo)
5. [Repository Structure and Mandatory Files](#5-repository-structure-and-mandatory-files)
6. [Branching Strategies](#6-branching-strategies)
7. [Branch Naming](#7-branch-naming)
8. [Commits and Commit Messages](#8-commits-and-commit-messages)
9. [Pull Requests / Merge Requests](#9-pull-requests--merge-requests)
10. [Code Review](#10-code-review)
11. [Merge Strategies and Merge Queues](#11-merge-strategies-and-merge-queues)
12. [Branch Protection and Rulesets](#12-branch-protection-and-rulesets)
13. [Tags, Releases, and Changelogs](#13-tags-releases-and-changelogs)
14. [Hotfix Process](#14-hotfix-process)
15. [Large Files and Large Repositories](#15-large-files-and-large-repositories)
16. [Secrets and Sensitive Data in Git](#16-secrets-and-sensitive-data-in-git)
17. [History Hygiene](#17-history-hygiene)
18. [Recovery and Debugging with Git](#18-recovery-and-debugging-with-git)
19. [Sharing Code Between Repositories](#19-sharing-code-between-repositories)
20. [Versioning Everything Else](#20-versioning-everything-else)
21. [Security, Compliance, and Audit](#21-security-compliance-and-audit)
22. [Scenario Playbooks](#22-scenario-playbooks)
23. [Command Reference — Workflow](#23-command-reference--workflow)
24. [Checklists](#24-checklists)
25. [References](#25-references)

---

## 1. Why Version Control Is the Backbone

Everything in a modern SDLC hangs off the repository:

```text
Requirement ID → branch → commits → pull request → review → CI → merge → tag → release → deploy
                                    (traceability, audit trail, rollback point at every step)
```

| Capability | Provided by |
|---|---|
| Collaboration without overwriting | Branches, merges |
| Quality gates | Pull requests, required checks |
| Audit trail | Immutable commit history, signed commits, PR records |
| Reproducibility | Tags, lockfiles, IaC in Git |
| Rollback | Revert commits, redeploy previous tags |
| Traceability | IDs in branches/commits/PRs (chapter 03 §13) |
| Automation | Webhooks → CI/CD, GitOps |

> **Rule: if it isn't in version control, it doesn't exist.** Code, tests, infrastructure, pipelines, configuration (non-secret), database migrations, documentation, ADRs, dashboards-as-code, and runbooks all belong in Git.

---

## 2. Git Mental Model

### 2.1 Core objects

| Object | What it is |
|---|---|
| **Blob** | File contents |
| **Tree** | Directory listing (names → blobs/trees) |
| **Commit** | Snapshot (root tree) + parent(s) + author/committer + message |
| **Tag** (annotated) | Named, signed-able pointer to a commit with metadata |

Each object is identified by a hash of its content (SHA-1 today by default; SHA-256 coming as the default — §3).

### 2.2 References

| Ref | Meaning |
|---|---|
| Branch (`refs/heads/main`) | Movable pointer to a commit |
| Remote-tracking branch (`refs/remotes/origin/main`) | Last known state of a remote branch |
| Tag (`refs/tags/v1.4.0`) | Fixed pointer to a commit |
| `HEAD` | What you have checked out |

### 2.3 The three areas

```text
Working tree ──git add──► Index (staging) ──git commit──► Repository (history)
      ▲                                                        │
      └──────────────── git restore / git switch ◄─────────────┘
```

### 2.4 History is a DAG

```text
A──B──C──F──G      main
       \     /
        D──E       feature (merged with a merge commit G)
```

Understanding that commits are immutable snapshots and branches are just pointers makes every "scary" Git operation predictable.

---

## 3. Git Versions and the Git 3.0 Transition

- Git is in the **2.5x** release series (Git 2.55 was released 29 June 2026; Git 2.56 has since been released). Keep developer machines and CI runners on a recent 2.x release — install/upgrade instructions are in **42 §1** and **41**.
- **Git 3.0** is a planned *breaking* release. The Git project documents upcoming breaking changes and says **no release date is set**; community reporting points to a target around the end of 2026.

### 3.1 Changes planned for Git 3.0

| Change | What it means | Prepare now |
|---|---|---|
| **Default branch `main`** for new repos | `git init` without config will create `main` instead of `master` | Set `git config --global init.defaultBranch main` (already the default on GitHub, GitLab, Bitbucket) |
| **SHA-256 default** for new repos | New repositories use SHA-256 object IDs; existing SHA-1 repos keep working; interoperability work is ongoing | Audit tools/scripts that assume 40-char hashes; test with `git init --object-format=sha256` |
| **Reftable** as default ref storage | Faster, more scalable ref storage format; better behaviour on case-insensitive filesystems | Test tooling that reads `.git/refs` directly — use `git` plumbing commands instead |
| **Rust** becomes a build requirement for Git itself | Affects people building Git from source / distro packagers | Usually no action for users of packaged Git |

> ⚠️ Hosting support for SHA-256 repositories is the main ecosystem dependency. Do not create SHA-256 production repositories until your Git host and all tooling (CI, code review, scanners) support them.

```bash
# Hygiene that is future-proof regardless of Git 3.0 timing
git config --global init.defaultBranch main
git rev-parse --show-object-format     # sha1 or sha256 for the current repo
git rev-parse --short=12 HEAD          # never hard-code hash lengths in scripts
```

Official list of breaking changes: https://git-scm.com/docs/BreakingChanges

---

## 4. Repository Strategy — Monorepo vs Polyrepo

### 4.1 Comparison

| Dimension | Monorepo | Polyrepo |
|---|---|---|
| Atomic cross-project changes | ✅ One PR | ❌ Coordinated PRs/releases |
| Code sharing | ✅ Direct | Packages + versioning |
| Consistent tooling/standards | ✅ One config | Must be replicated (templates, shared configs) |
| Access control granularity | Path-based (CODEOWNERS, rulesets) | Per repo |
| CI scale | Needs affected-only builds, caching | Naturally scoped |
| Clone size / performance | Needs sparse checkout, partial clone | Small repos |
| Team autonomy | Shared conventions | Higher |
| Dependency drift | Low (single version policy possible) | Higher |

### 4.2 Decision guide

```text
Prefer MONOREPO when:
  - Many services/packages share libraries and change together
  - You want one set of standards and atomic refactors
  - You can invest in build tooling (affected builds, remote cache)
Prefer POLYREPO when:
  - Teams/products are truly independent, with different release cycles or access needs
  - Open-source components need their own repos
  - Build tooling investment isn't feasible
Hybrid is common: one monorepo per product/domain + separate repos for infra/platform/OSS.
```

### 4.3 Monorepo tooling

| Ecosystem | Tools |
|---|---|
| JS/TS | **Nx**, **Turborepo**, pnpm workspaces, Rush |
| JVM | **Gradle** multi-project (+ build cache), Maven multi-module |
| Polyglot / large scale | **Bazel**, **Pants**, Buck2 |
| .NET | Solution files + Directory.Build.props, Nx (.NET plugin) |
| Go | Go workspaces (`go.work`), multiple modules |
| Python | uv workspaces, Pants |

Must-haves: **affected-only builds/tests**, **remote build cache**, **CODEOWNERS per path**, **sparse checkout** for large repos.

---

## 5. Repository Structure and Mandatory Files

### 5.1 Service repository layout (example)

```text
orders-service/
├── .github/ or .gitlab/           # CI workflows, issue/PR templates
│   ├── workflows/
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── CODEOWNERS
├── docs/
│   ├── adr/
│   ├── runbooks/
│   └── architecture/
├── src/
├── tests/
├── deploy/                        # Helm/Kustomize/Terraform for this service (or separate infra repo)
├── .editorconfig
├── .gitattributes
├── .gitignore
├── .pre-commit-config.yaml
├── .git-blame-ignore-revs
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE                        # if distributed / open source
├── README.md
└── SECURITY.md
```

### 5.2 Mandatory files

| File | Purpose | Status |
|---|---|:---:|
| `README.md` | What, why, how to run/test/deploy, owners | 🔴 |
| `.gitignore` | Exclude build outputs, deps, secrets, IDE files | 🔴 |
| `.gitattributes` | Line endings, binary handling, LFS patterns, diff drivers | 🔴 |
| `CODEOWNERS` | Required reviewers per path | 🔴 |
| `CONTRIBUTING.md` | Branch, commit, PR, and review conventions | 🟠 |
| `SECURITY.md` | Vulnerability reporting process | 🟠 (🔴 for public repos) |
| `CHANGELOG.md` | Human-readable release history | 🟠 |
| PR/MR template | Consistent PR descriptions | 🟠 |
| `LICENSE` | Legal terms | 🔴 for open source |

Templates for `.gitignore`: https://github.com/github/gitignore

---

## 6. Branching Strategies

### 6.1 Strategies compared

| Strategy | Long-lived branches | Release model | Best for |
|---|---|---|---|
| **Trunk-based development** | `main` only (+ optional short-lived release branches) | Continuous delivery/deployment; feature flags | High-performing teams, SaaS, CI/CD (strongly correlated with DORA performance) |
| **GitHub Flow** | `main` | Deploy from `main` after merge (or from PR branches) | Web apps with continuous deployment |
| **GitLab Flow** | `main` + environment branches (`staging`, `production`) or release branches | Promotion via merges | Teams needing explicit environment promotion |
| **Release branches** | `main` + `release/x.y` | Scheduled releases, multiple supported versions | Mobile apps, on-prem/shipped software, libraries |
| **Git Flow** (Driessen, 2010) | `main` + `develop` (+ release/hotfix/feature) | Versioned releases | Legacy/shipped software with infrequent releases — the author himself recommends simpler models for continuously delivered web apps |

### 6.2 Recommendation

```text
Default for production teams:  TRUNK-BASED DEVELOPMENT
  - Short-lived feature branches (hours to ≤ 2 days), merged via PR into main
  - main is always releasable (green CI, protected)
  - Incomplete work hidden behind feature flags
  - Release branches only when you must support/patch released versions (mobile, on-prem)
Avoid:  long-lived feature branches, develop branches that drift, environment branches with cherry-picks
```

### 6.3 Trunk-based with short-lived branches

```text
main ──●──●──●──●──●──●──●──●──►   (every merge: CI green, deployable)
         \  /    \   /   \  /
          ●       ●─●     ●         short-lived branches (≤ 1–2 days)
```

### 6.4 Release branches for shipped software

```text
main ──●──●──●──●──●──●──●──●──►
              \           \
               release/2.4 ──●(v2.4.0)──●(v2.4.1 hotfix)
                           \
                            release/2.5 ──●(v2.5.0)
Rules: fix on main first, then cherry-pick to supported release branches (`git cherry-pick -x`).
```

### 6.5 Keeping branches short-lived

| Technique | How |
|---|---|
| Feature flags | Merge incomplete code switched off (chapter 05 §18) |
| Branch by abstraction | Introduce an abstraction, swap implementations gradually |
| Parallel change (expand/contract) | For APIs and schemas |
| Small vertical slices | Split stories (chapter 03 §8.4) |
| Stacked PRs | Chain small dependent PRs (§9.5) |

---

## 7. Branch Naming

```text
<type>/<ticket-id>-<short-kebab-description>

feat/PAY-142-retry-alt-payment
fix/PAY-151-null-currency
chore/PLAT-88-upgrade-node-24
docs/PAY-160-retry-runbook
refactor/ORD-77-extract-pricing
perf/SRCH-12-cache-facets
release/2.5
hotfix/2.4.1-PAY-170-duplicate-charge
```

| Rule | Status |
|---|:---:|
| Lowercase, kebab-case, ASCII only | 🔴 |
| Include the ticket ID for traceability | 🔴 |
| Type prefix aligned with Conventional Commit types | 🟠 |
| Delete branches after merge (enable auto-delete) | 🔴 |
| Enforce patterns with rulesets/push rules where supported | 🟡 |

---

## 8. Commits and Commit Messages

### 8.1 Commit principles

| Principle | Status |
|---|:---:|
| **Atomic**: one logical change per commit; builds and tests pass at every commit on `main` | 🔴 |
| Separate refactoring/formatting commits from behaviour changes | 🔴 |
| Never commit secrets, generated artifacts, or large binaries (use LFS/artifact stores) | 🔴 |
| Commit early and often on your branch; tidy before merging (§17) | 🟠 |
| Sign commits (SSH or GPG) — setup in **42 §4** | 🟠 (🔴 in regulated environments) |

### 8.2 Conventional Commits 1.0.0

Specification: https://www.conventionalcommits.org/en/v1.0.0/

```text
<type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

| Type | Use | SemVer effect (with release tooling) |
|---|---|---|
| `feat` | New feature | MINOR |
| `fix` | Bug fix | PATCH |
| `feat!` / `BREAKING CHANGE:` footer | Breaking API change | MAJOR |
| `docs` | Documentation only | none |
| `style` | Formatting, no logic change | none |
| `refactor` | Neither fix nor feature | none |
| `perf` | Performance improvement | PATCH (configurable) |
| `test` | Tests only | none |
| `build` | Build system, dependencies | none |
| `ci` | CI configuration | none |
| `chore` | Maintenance | none |
| `revert` | Reverts a previous commit | depends |

> In the spec itself only `feat` and `fix` (and breaking changes) carry SemVer meaning; the other types are widely used conventions (from the Angular convention / commitlint config-conventional).

### 8.3 Good commit messages

```text
feat(payments): offer alternative methods after card decline

When the issuer declines with a non-fraud reason, the checkout now offers
UPI and netbanking within the same session. Fraud blocks still end the
flow with a support message. Attempts are capped at 3 per order.

Refs: PAY-142
```

```text
fix(orders)!: return 409 instead of 400 for duplicate idempotency key

BREAKING CHANGE: clients relying on 400 for duplicate keys must handle 409.
Refs: ORD-201
```

Message rules (from the long-standing Git community convention):

```text
🔴 Subject ≤ ~72 characters, imperative mood ("add", not "added"/"adds"), no trailing period
🔴 Blank line between subject and body
🟠 Body explains WHAT and WHY (the diff shows HOW); wrap at ~72 chars
🟠 Footers: Refs/Closes ticket IDs, BREAKING CHANGE, Co-authored-by, Signed-off-by (DCO)
```

### 8.4 Enforcing commit conventions

```bash
# commitlint (Node-based) with conventional config
npm install -D @commitlint/cli @commitlint/config-conventional
echo "export default { extends: ['@commitlint/config-conventional'] };" > commitlint.config.js
# run in a commit-msg hook (Husky/Lefthook/pre-commit) and in CI for PR titles when squash-merging
```

When using **squash merges**, the **PR title** becomes the commit subject on `main` — validate PR titles in CI.

### 8.5 Developer Certificate of Origin (DCO)

Some open-source projects require `Signed-off-by:` lines (`git commit -s`) certifying the contributor's right to submit the code (https://developercertificate.org/). This is different from cryptographic commit signing.

---

## 9. Pull Requests / Merge Requests

### 9.1 PR size

| Size (changed lines, excluding generated/lockfiles) | Guidance |
|---|---|
| < 200 | ✅ Ideal — fast, thorough review |
| 200–400 | 🟠 Acceptable |
| > 400 | 🔴 Split it (or explain why it can't be split) |

Small PRs get reviewed faster and more carefully, and correlate with shorter lead time and lower change-fail rate.

### 9.2 PR template

```markdown
## What
Short description of the change.

## Why
Problem / ticket: PAY-142. Link to design doc/ADR if any.

## How
Key implementation notes, trade-offs, alternatives considered.

## Testing
- [ ] Unit tests
- [ ] Integration / contract tests
- [ ] Manual verification steps (screenshots/recordings for UI)

## Risk & rollout
- Risk level: low / medium / high
- Feature flag: `checkout-payment-retry` (default off)
- Migration: none / expand step only
- Rollback: revert PR or disable flag

## Checklist
- [ ] Follows coding standards (chapter 06); CI green
- [ ] Docs/runbooks/ADRs updated
- [ ] Observability added (logs/metrics/traces)
- [ ] Security considered (authz, input validation, secrets)
- [ ] No breaking API changes (or versioned and announced)
```

### 9.3 PR lifecycle

```text
Draft PR (early feedback, CI runs)
  → Ready for review (self-review done, description complete)
  → Review (comments resolved; approvals per CODEOWNERS/rules)
  → Required checks green; branch up to date (or merge queue)
  → Merge (strategy per §11) → branch auto-deleted
  → Deploy → verify → close linked ticket
```

### 9.4 Author responsibilities

```text
🔴 Self-review the diff before requesting review
🔴 Keep the PR focused; no unrelated changes
🔴 Respond to every comment (fix, discuss, or explain)
🟠 Annotate tricky parts with PR comments to guide reviewers
🟠 Rebase/merge main regularly to keep the branch current
```

### 9.5 Stacked PRs

Break a large change into a chain of small dependent PRs (each reviewable alone):

```text
main ← PR1: add PaymentGateway port
        ← PR2: PSP adapter implementing the port
            ← PR3: retry use case
                ← PR4: UI
```

Tools: GitHub/GitLab native stacking support where available, Graphite, `git town`, `spr`, or manual `--update-refs` rebases (`git rebase --update-refs`).

---

## 10. Code Review

### 10.1 The standard (Google Engineering Practices)

> Approve a change once it **definitely improves the overall code health** of the system, even if it isn't perfect. There is no "perfect" code — only better code.

Source: https://google.github.io/eng-practices/review/

### 10.2 What to look for (in priority order)

| # | Area | Questions |
|---:|---|---|
| 1 | **Design** | Does it belong here? Fits the architecture? Simpler approach? |
| 2 | **Functionality / correctness** | Does it do what's intended? Edge cases, concurrency, error paths? |
| 3 | **Security** | AuthZ on every resource? Input validation? Secrets? Injection? |
| 4 | **Tests** | Meaningful, deterministic, covering failure paths? Would they fail if the code broke? |
| 5 | **Operability** | Logs/metrics/traces? Timeouts/retries? Migration safety? Rollback? |
| 6 | **Complexity** | Can a future reader understand it quickly? Over-engineering? |
| 7 | **Naming & readability** | Clear names, comments explain *why* |
| 8 | **Consistency** | Follows standards (most should be automated — don't nitpick what tools catch) |
| 9 | **Documentation** | README, API docs, ADRs, changelog updated |

### 10.3 Comment labels (Conventional Comments)

```text
blocking: this query isn't scoped by tenant_id — IDOR risk.
issue: retry here isn't idempotent; duplicate charges possible on timeout.
suggestion: extract this into a PaymentRetryPolicy for testability.
question: why 5s timeout here when the SLO budget is 2s?
nit: typo in log message.
praise: nice use of a sealed interface here 👍
```

Spec: https://conventionalcomments.org/

### 10.4 Review service levels

| Item | Target |
|---|---|
| First response to a review request | Within **1 business day** (ideally a few hours) |
| Small PR complete review | Same day |
| Reviewer load | Spread via CODEOWNERS teams + auto-assignment; avoid single-person bottlenecks |
| Disagreement | Resolve in conversation; escalate to tech lead after 2 rounds; record decisions |

### 10.5 Approvals policy (example)

| Change type | Approvals |
|---|---|
| Standard change | 1 approval from code owner |
| Security-sensitive paths (auth, crypto, payments) | 2 approvals incl. security champion/owner |
| Infrastructure / CI / IaC | 1 platform owner |
| Database migrations | 1 DB-savvy reviewer |
| Docs only | 1 approval (any team member) |

### 10.6 CODEOWNERS

```text
# .github/CODEOWNERS (GitHub) — last matching pattern wins
*                           @shopnow/backend-team
/apps/web/                  @shopnow/frontend-team
/services/payments/         @shopnow/payments-team @shopnow/security-champions
/infra/                     @shopnow/platform-team
/db/migrations/             @shopnow/dba-reviewers
/.github/workflows/         @shopnow/platform-team
/docs/adr/                  @shopnow/architects
```

GitLab supports `CODEOWNERS` with sections and approval rules; Bitbucket Cloud supports default reviewers and code owners via its own configuration — see **42** for platform specifics.

### 10.7 Review anti-patterns

| Anti-pattern | Fix |
|---|---|
| Rubber-stamp approvals | Require meaningful review of risky paths; rotate reviewers |
| Nitpicking formatting | Automate with formatters/linters |
| Huge PRs reviewed superficially | Enforce size guidance; stacked PRs |
| Review as gatekeeping/ego | Kind, specific, actionable comments about code, not people |
| Slow reviews blocking flow | Review SLAs; make review time part of the team's priorities |

---

## 11. Merge Strategies and Merge Queues

### 11.1 Strategies

| Strategy | Result on `main` | Pros | Cons | Use |
|---|---|---|---|---|
| **Squash merge** | One commit per PR | Clean linear history; PR title = commit message | Loses intermediate commits; harder with stacked branches | ✅ Default for most teams |
| **Rebase merge** | PR commits replayed linearly | Linear history, keeps atomic commits | Requires disciplined commit hygiene | Teams that curate commits |
| **Merge commit** | Merge commit with two parents | Preserves full branch topology | Noisy history | Release branch merges, long-lived integration |

```text
Recommended: squash merge for feature PRs + require linear history on main.
Release branches: merge or cherry-pick -x for traceability.
```

### 11.2 Merge queues

A merge queue tests each PR **against the latest `main` plus PRs ahead of it** before merging, preventing "green PR, red main" when two compatible-looking PRs conflict semantically.

| Platform | Feature |
|---|---|
| GitHub | Merge queue (ruleset/branch protection setting; CI must trigger on `merge_group` events) |
| GitLab | Merge trains |
| Others | Bors-style bots, Mergify, Aviator |

```yaml
# GitHub Actions: run required checks for merge queue entries too
on:
  pull_request:
  merge_group:
```

---

## 12. Branch Protection and Rulesets

### 12.1 Baseline for `main` (and `release/*`)

| Rule | Status |
|---|:---:|
| No direct pushes; changes via PR only | 🔴 |
| Required status checks (build, lint, tests, security scans) | 🔴 |
| Required approvals (≥ 1; ≥ 2 for sensitive paths) + code-owner review | 🔴 |
| Dismiss stale approvals when new commits are pushed | 🔴 |
| Require conversation resolution before merge | 🟠 |
| Require linear history | 🟠 |
| Require signed commits | 🟠 (🔴 regulated) |
| Block force pushes and deletion | 🔴 |
| Apply rules to administrators too (no bypass, or audited bypass only) | 🔴 |
| Merge queue for busy repos | 🟠 |
| Tag protection for `v*` (only release automation can create) | 🟠 |

### 12.2 Organisation-level rulesets

Manage rules centrally (GitHub organisation rulesets, GitLab group-level settings/compliance frameworks, Bitbucket workspace-level permissions) so every repo inherits the baseline instead of being configured by hand.

### 12.3 Push protection and server-side checks

- Enable **secret scanning with push protection** (GitHub Advanced Security / secret protection; GitLab secret push protection; Bitbucket secret scanning) — blocks pushes containing detected secrets.
- Restrict file sizes and file types server-side where supported.

---

## 13. Tags, Releases, and Changelogs

### 13.1 Tags

```bash
git tag -s v2.5.0 -m "Release 2.5.0"     # signed annotated tag (setup: 42 §4)
git push origin v2.5.0
git tag -v v2.5.0                         # verify signature
git describe --tags --always              # human-readable version from history
```

```text
🔴 Use annotated (preferably signed) tags for releases; never move or delete a published release tag
🔴 Tag format: v<MAJOR>.<MINOR>.<PATCH>[-prerelease] (SemVer 2.0.0)
🟠 Releases are created by automation from CI, not by hand
```

### 13.2 Release automation tools

| Tool | Approach |
|---|---|
| **release-please** (Google) | Opens a release PR from Conventional Commits; merging it tags and publishes |
| **semantic-release** | Fully automated version, changelog, tag, publish on merge |
| **Changesets** | Contributors add changeset files; great for JS monorepos |
| **GoReleaser** | Go binaries, archives, checksums, signing |
| **git-cliff** | Changelog generation from commits (any language) |

### 13.3 Changelog format (Keep a Changelog)

```markdown
# Changelog
All notable changes to this project are documented here.
The format follows Keep a Changelog, and this project adheres to Semantic Versioning.

## [Unreleased]

## [2.5.0] - 2026-10-02
### Added
- Alternative payment methods after card decline (PAY-142)
### Fixed
- Currency missing in refund emails (PAY-151)
### Security
- Upgrade xyz-lib to 4.2.1 (CVE-…)

[2.5.0]: https://git.example.com/shop/orders/compare/v2.4.1...v2.5.0
```

Sections: **Added, Changed, Deprecated, Removed, Fixed, Security**. https://keepachangelog.com/

---

## 14. Hotfix Process

### 14.1 Trunk-based (continuously deployed)

```text
1. Branch from main: fix/PAY-170-duplicate-charge
2. Minimal fix + regression test; expedited review (still ≥ 1 approval)
3. Merge → pipeline → canary → full rollout
4. If main contains unreleasable work: fix main, then release from a tag-based release branch (below)
5. Postmortem if customer impact (chapter 21)
```

### 14.2 Release-branch (shipped versions)

```bash
# Fix lands on main first (default), then is backported
git switch main && git pull
# ... merge fix PR to main (commit abc1234) ...
git switch release/2.4
git cherry-pick -x abc1234          # -x records the original commit ID
git tag -s v2.4.1 -m "Hotfix 2.4.1: prevent duplicate charge"
git push origin release/2.4 v2.4.1
```

```text
🔴 Fix forward on main first, then backport — never fix only on the release branch
🔴 A hotfix still needs a test proving the bug and its fix
🟠 Document the hotfix in the changelog and incident record
```

Emergency changes that bypass normal gates must be **logged and reviewed retrospectively** (break-glass process).

---

## 15. Large Files and Large Repositories

### 15.1 What not to commit

```text
❌ Build outputs (dist/, target/, bin/, obj/)       → CI artifacts / registries
❌ Dependencies (node_modules/, vendor/ for most)  → package managers + lockfiles
❌ Large binaries (videos, datasets, model weights) → Git LFS, object storage, DVC, artifact registries
❌ Secrets, .env files, private keys                → secret managers
❌ Personal IDE settings (except shared, intentional ones)
```

### 15.2 Git LFS

```bash
git lfs install
git lfs track "*.psd" "*.mp4" "assets/models/*.onnx"
git add .gitattributes
git commit -m "build: track design and media assets with Git LFS"
git lfs ls-files
```

> LFS has storage/bandwidth quotas on hosted platforms; check your plan. Configure LFS **before** adding large files — migrating afterwards requires history rewriting (`git lfs migrate`).

### 15.3 Scaling large repositories

| Technique | Command / note |
|---|---|
| Partial clone (blobless) | `git clone --filter=blob:none <url>` |
| Shallow clone (CI) | `git clone --depth=1 <url>` (avoid when tools need history, e.g. versioning) |
| Sparse checkout | `git sparse-checkout set apps/web libs/ui` |
| Scalar (large-repo defaults) | `scalar clone <url>` / `scalar register` — ships with Git |
| File system monitor | `git config core.fsmonitor true` (built-in daemon on macOS/Windows; recent Git releases add Linux support) |
| Background maintenance | `git maintenance start` |
| Commit graph / multi-pack index | Enabled by maintenance tasks |

Performance config snippets: **42 §5**.

---

## 16. Secrets and Sensitive Data in Git

### 16.1 Prevention (layers)

```text
1. .gitignore for .env, *.pem, *.key, credentials files
2. Pre-commit secret scanning (gitleaks, trufflehog, detect-secrets)
3. Server-side push protection (platform secret scanning)
4. CI secret scanning on full history (periodic)
5. Secrets only in secret managers; apps read them at runtime
```

### 16.2 If a secret is committed

```text
1. ROTATE/REVOKE THE SECRET IMMEDIATELY — assume it is compromised.
   (Removing it from history does NOT un-leak it: forks, clones, caches, CI logs may have it.)
2. Check access logs for misuse; follow the incident process (chapter 21).
3. Remove from history if required (policy/compliance), using git-filter-repo:
     git filter-repo --replace-text expressions.txt     # or --path secret.env --invert-paths
4. Force-push the rewritten history (coordinate with the team; everyone re-clones).
5. Ask the hosting provider to purge cached views/PR refs if needed (platform-specific support process).
6. Add a detection rule to prevent recurrence.
```

> The Git project recommends **git-filter-repo** over the older `git filter-branch`. https://github.com/newren/git-filter-repo

### 16.3 Personal data

Personal data (customer records, production DB dumps, logs with PII) must never be committed. Test fixtures use synthetic data. If personal data is committed, treat it as a **data incident** (privacy/legal obligations under GDPR, India DPDP Act, etc.).

---

## 17. History Hygiene

### 17.1 Golden rules

```text
🔴 Never rewrite history on shared branches (main, release/*)
🔴 On your own branch, rewriting is fine; push with --force-with-lease (never plain --force)
🟠 Tidy commits before merge if using rebase-merge (squash merge makes this less critical)
🟠 Keep main linear and every commit buildable (helps git bisect)
```

### 17.2 Fixup workflow

```bash
git commit --fixup=<sha>                 # mark a fix for an earlier commit on your branch
git rebase -i --autosquash origin/main   # automatically squash fixups into their targets
git push --force-with-lease
```

### 17.3 Updating your branch

```bash
git fetch origin
git rebase origin/main            # linear history on your branch (resolve conflicts commit by commit)
# or
git merge origin/main             # preserves branch history; simpler for long-lived shared branches
git config --global rerere.enabled true   # reuse recorded conflict resolutions
```

### 17.4 Rebase vs merge (for updating branches)

| | Rebase | Merge |
|---|---|---|
| History | Linear | Includes merge commits |
| Rewrites commits | Yes (your branch only) | No |
| Best for | Personal short-lived branches | Shared branches |

---

## 18. Recovery and Debugging with Git

| Situation | Command |
|---|---|
| Undo uncommitted changes in a file | `git restore <file>` |
| Unstage | `git restore --staged <file>` |
| Undo last commit, keep changes | `git reset --soft HEAD~1` |
| Undo a pushed commit on a shared branch | `git revert <sha>` (creates an inverse commit) |
| Revert a merge commit | `git revert -m 1 <merge-sha>` |
| Find "lost" commits after reset/rebase | `git reflog` → `git branch rescue <sha>` |
| Recover a deleted branch | `git reflog` → `git switch -c <name> <sha>` |
| Find the commit that introduced a bug | `git bisect start` / `bad` / `good v2.4.0` / `git bisect run ./test.sh` |
| Who changed this line and why | `git blame -w -C <file>` (+ `.git-blame-ignore-revs`) |
| Search history for a string | `git log -S "functionName" --oneline` |
| Search history with regex | `git log -G "regex" -p` |
| Show file at a past version | `git show v2.4.0:src/app.ts` |
| Temporarily shelve work | `git stash push -m "wip retry"` / `git stash pop` |
| Work on two branches simultaneously | `git worktree add ../hotfix release/2.4` |

### Automated bisect

```bash
git bisect start
git bisect bad HEAD
git bisect good v2.4.0
git bisect run npm test -- --run tests/payments/retry.test.ts
git bisect reset
```

---

## 19. Sharing Code Between Repositories

| Mechanism | How | Pros | Cons | Use |
|---|---|---|---|---|
| **Packages** (npm, Maven, PyPI, NuGet, Go modules…) | Publish versioned artifacts to a registry | Clear versioning, standard tooling | Release overhead | ✅ Default for shared libraries |
| **Monorepo** | Shared code in the same repo | Atomic changes | Requires monorepo tooling | Tightly coupled code |
| **Git submodules** | Pin another repo at a commit | Exact pinning | Easy to misuse; extra commands; detached HEADs | Vendored third-party code, rare cases |
| **Git subtree** | Copy another repo's history into a subdirectory | No extra tooling for consumers | Complex syncing | Occasional vendoring |
| **Copy-paste** | — | — | Divergence | ❌ Avoid |

Registry setup per ecosystem: **37-official-package-registries.md**; private registries: chapter 16.

---

## 20. Versioning Everything Else

| Artifact | Approach |
|---|---|
| **Infrastructure** | IaC (Terraform/OpenTofu, Pulumi, Bicep, CloudFormation) in Git; plan in PR, apply from CI (chapter 17) |
| **Kubernetes / deployments** | GitOps (Argo CD, Flux): the Git repo is the desired state |
| **CI/CD pipelines** | Pipeline-as-code in the repo (`.github/workflows`, `.gitlab-ci.yml`, `bitbucket-pipelines.yml`) |
| **Database schema** | Versioned migrations (Flyway, Liquibase, Alembic, Prisma Migrate, EF Core migrations, golang-migrate) — forward-only, reviewed (chapter 10) |
| **API contracts** | OpenAPI/AsyncAPI/proto files versioned with code; breaking-change checks in CI |
| **Configuration** | Non-secret config in Git; secrets referenced, not stored |
| **Documentation** | Docs-as-code (Markdown + MkDocs/Docusaurus) (chapter 22) |
| **Dashboards & alerts** | As code (Grafana provisioning/Jsonnet/Terraform providers) |
| **Datasets & ML models** | DVC, LFS, or model registries (MLflow); never raw large files in Git |
| **Prompts (LLM apps)** | Versioned prompt templates in Git with eval results per version |

---

## 21. Security, Compliance, and Audit

| Control | Implementation |
|---|---|
| **Strong authentication** | SSO + MFA for Git hosting; SSH keys (Ed25519) or fine-grained tokens with expiry (setup: 42 §3) |
| **Least privilege** | Teams-based access; write access only where needed; CI tokens scoped per repo/job |
| **Signed commits and tags** | SSH/GPG signing; verified badges; required signatures on protected branches |
| **Provenance** | CI produces build provenance (SLSA) linking artifacts to commit SHA and workflow |
| **Separation of duties** | Author cannot approve own PR; protected branches for all including admins |
| **Audit logs** | Platform audit logs streamed to SIEM; retention per policy |
| **Secret scanning** | Push protection + historical scans |
| **Dependency review** | Dependency review on PRs; SCA (chapter 09) |
| **Repository hygiene** | Archive unused repos; review collaborators/deploy keys quarterly |
| **Retention / legal hold** | Repos and PR records retained per regulatory requirements |

Regulated-industry evidence that Git provides automatically: PR = change request, approvals = review evidence, CI checks = test evidence, signed tag = release record, deploy logs = implementation evidence.

---

## 22. Scenario Playbooks

### 22.1 Small startup team (2–6 engineers)

```text
- One repo per product (monorepo for web + API + shared libs)
- Trunk-based; squash merge; 1 approval; CI required
- Continuous deployment from main with feature flags
- release-please or semantic-release for versioning/changelog
```

### 22.2 Enterprise SaaS (many teams)

```text
- Org-level rulesets: protected main, required checks, code owners, signed commits
- Monorepo per domain with Nx/Bazel/Gradle affected builds, or polyrepo with shared templates
- Merge queues on busy repos
- Secret push protection org-wide; SSO-enforced access; audit log streaming
- Service catalogue links repos ↔ owners ↔ SLOs
```

### 22.3 Mobile apps

```text
- Trunk-based with release branches per app-store release (release/5.12)
- Fixes on main, cherry-pick -x to release branch
- Version = marketing SemVer + monotonically increasing build number from CI
- Tags per store submission; phased rollout tracked in release notes
```

### 22.4 Libraries / SDKs (open source or internal)

```text
- SemVer strictly; Conventional Commits; automated releases
- Maintain release branches for supported majors (e.g. 3.x, 4.x)
- Public CHANGELOG with migration guides for breaking changes
- DCO or CLA for external contributions; SECURITY.md with disclosure process
```

### 22.5 Regulated environment (banking, health, government)

```text
- Signed commits and tags required; 2 approvals; segregation of duties
- Every PR linked to an approved change request / ticket
- CI evidence archived (test reports, scan results, SBOM, provenance)
- Release tags immutable; break-glass changes logged and reviewed
```

### 22.6 Migrating from Git Flow to trunk-based

```text
1. Make main protected and always releasable; strengthen CI
2. Introduce feature flags
3. Stop creating new long-lived feature branches; cap branch age (e.g. 2 days)
4. Merge develop into main; retire develop
5. Keep release branches only where you support shipped versions
6. Track DORA lead time and change fail rate before/after
```

---

## 23. Command Reference — Workflow

```bash
# Start work
git switch main && git pull --ff-only
git switch -c feat/PAY-142-retry-alt-payment

# Commit
git add -p                                   # stage hunks interactively
git commit -m "feat(payments): offer alternative methods after decline" -m "Refs: PAY-142"

# Stay current
git fetch origin && git rebase origin/main
git push -u origin HEAD
git push --force-with-lease                  # after rebasing your own branch

# Inspect
git status -sb
git log --oneline --graph --decorate -20
git diff origin/main...HEAD                  # what this branch changes
git range-diff origin/main@{1} origin/main HEAD   # compare before/after rebase

# Clean up after merge
git switch main && git pull --ff-only
git branch -d feat/PAY-142-retry-alt-payment
git fetch --prune

# Platform CLIs (setup in 42)
gh pr create --fill --draft                  # GitHub
gh pr checks --watch
glab mr create --fill --draft                # GitLab
```

---

## 24. Checklists

### Repository setup
- [ ] Default branch `main`; protected (no direct pushes, no force push, no deletion)
- [ ] Required checks, approvals, code-owner review, stale-approval dismissal
- [ ] Merge strategy chosen (squash default); linear history; auto-delete branches
- [ ] CODEOWNERS, PR template, CONTRIBUTING, SECURITY.md, README
- [ ] `.gitignore`, `.gitattributes`, `.editorconfig`, pre-commit hooks
- [ ] Secret scanning + push protection enabled
- [ ] Dependency update bot enabled
- [ ] Tag protection for release tags; release automation configured
- [ ] Access via teams; SSO/MFA enforced

### Daily developer practice
- [ ] Short-lived branch named `<type>/<ticket>-<desc>`
- [ ] Small, atomic commits with Conventional Commit messages
- [ ] Rebase/merge main at least daily
- [ ] PR < ~400 lines, template complete, self-reviewed
- [ ] `--force-with-lease` only on own branches

### Release
- [ ] Release created by automation from a green main (or release branch)
- [ ] Signed annotated tag; changelog generated and reviewed
- [ ] Artifacts linked to the tag (SBOM, provenance)
- [ ] Hotfixes landed on main first and backported with `cherry-pick -x`

### Git 3.0 readiness
- [ ] `init.defaultBranch=main` everywhere (dev machines, CI images, scripts)
- [ ] No scripts assume 40-character hashes or read `.git/refs` directly
- [ ] Track your Git host's SHA-256 support before creating SHA-256 repos

---

## 25. References

### Git
- Git documentation: https://git-scm.com/doc
- Pro Git book (free): https://git-scm.com/book/en/v2
- Git breaking changes for 3.0: https://git-scm.com/docs/BreakingChanges
- GitHub blog — Git release highlights: https://github.blog/ (search "Highlights from Git")
- git-filter-repo: https://github.com/newren/git-filter-repo
- Git LFS: https://git-lfs.com/
- Scalar: https://git-scm.com/docs/scalar

### Workflows and conventions
- Trunk-Based Development: https://trunkbaseddevelopment.com/
- DORA capability — trunk-based development: https://dora.dev/capabilities/trunk-based-development/
- GitHub Flow: https://docs.github.com/get-started/using-github/github-flow
- Git Flow original post (with 2020 note): https://nvie.com/posts/a-successful-git-branching-model/
- Conventional Commits 1.0.0: https://www.conventionalcommits.org/en/v1.0.0/
- Conventional Comments: https://conventionalcomments.org/
- Semantic Versioning: https://semver.org/
- Keep a Changelog: https://keepachangelog.com/
- Developer Certificate of Origin: https://developercertificate.org/
- Google Engineering Practices — Code Review: https://google.github.io/eng-practices/review/

### Platforms
- GitHub rulesets: https://docs.github.com/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets
- GitHub merge queue: https://docs.github.com/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue
- GitHub CODEOWNERS: https://docs.github.com/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners
- GitLab protected branches & merge trains: https://docs.gitlab.com/
- Bitbucket Cloud docs: https://support.atlassian.com/bitbucket-cloud/

### Release tooling
- release-please: https://github.com/googleapis/release-please
- semantic-release: https://semantic-release.gitbook.io/
- Changesets: https://github.com/changesets/changesets
- git-cliff: https://git-cliff.org/
- commitlint: https://commitlint.js.org/

---

**Previous:** [06 — Coding Standards](./06-coding-standards.md) · **Next:** [08 — Testing & Quality](./08-testing-and-quality.md)