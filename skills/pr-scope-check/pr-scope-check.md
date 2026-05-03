---
name: pr-scope-check
description: Invoke before opening a PR or when reviewing an incoming PR to assess its blast radius, identify structural risks, and recommend scope adjustments.
---

# Assess PR Scope

## The Law
**A PR must be reviewable by a human in one focused session — if the reviewer cannot hold the full change in their head, the PR must be split before it merges.**

## When to Use
- Before opening a PR on a branch with more than a few days of work
- When a PR review is taking longer than expected and reviewers are losing context
- When a PR description says "various improvements" or "refactor + feature"
- **Never skip when:** the diff crosses more than three distinct concerns, touches a shared utility used by many callers, or changes a public API surface

## Process

### Phase 1: Measure the Diff
1. Get the full diff against the base branch
   - `git diff main...HEAD --stat` for a file summary
   - `git diff main...HEAD` for the full line-level view
2. Record these counts before any qualitative assessment
   - Total files changed
   - Lines added and removed
   - Number of distinct directories touched
3. Apply the size thresholds:
   - **Small** (ideal): ≤10 files, ≤400 lines changed
   - **Medium** (acceptable): 11–25 files, 401–800 lines
   - **Large** (needs justification): 26–50 files, 801–1 500 lines
   - **Oversized** (must split): >50 files or >1 500 lines — no exceptions without explicit sign-off

### Phase 2: Identify Distinct Concerns
1. Group every changed file into a named concern
   - Example groupings: "data model change", "API endpoint", "UI component", "test suite", "config update"
2. Count the concerns
   - 1–2 concerns: focused PR, proceed
   - 3 concerns: borderline — document why they must ship together
   - 4+ concerns: split required — identify the minimal shippable unit and what can be deferred

### Phase 3: Blast Radius Assessment
For each changed file or module, answer:

1. **How many callers does this code have?**
   - Core utilities or shared services with many consumers carry high blast radius even with few changed lines

2. **Does a change here require changes elsewhere to stay consistent?**
   - Database schema migrations, API contract changes, shared type definitions

3. **Does this touch a critical path?**
   - Auth, payments, data persistence, background job queues — any failure here is production-impacting

4. **Is there test coverage for the changed paths?**
   - High blast radius + no tests = do not merge

5. **Is this reversible?**
   - Feature flags, soft deletes, backward-compatible API additions are lower risk than schema drops or breaking changes

Assign a blast radius rating:
- **Contained**: change is isolated to one module or domain, easy to roll back
- **Moderate**: touches a shared boundary but callers are known and tested
- **Wide**: changes a shared contract, a core utility, or a public API — requires extra scrutiny and staged rollout consideration

### Phase 4: Structural Risk Check
Walk the diff looking for these structural red flags:

1. A refactor mixed with a feature addition — impossible to tell which commit to revert if bugs surface
2. Test changes that lower assertion coverage rather than extend it
3. A migration with no rollback script
4. Config changes embedded in a feature PR — should be a separate, reviewable commit
5. Dependency version bumps bundled with application logic changes
6. Dead code removal bundled with new feature code — scope creep that obscures intent

### Phase 5: Deliver the Assessment
Write the assessment in this structure:

1. **Size**: state the measured counts and the size bucket (Small / Medium / Large / Oversized)
2. **Concerns**: list each distinct concern by name; state whether they must ship together and why
3. **Blast Radius**: state the rating and name the highest-risk files
4. **Structural Risks**: list any pattern matches from Phase 4
5. **Recommendation**: one of —
   - **Approve scope**: PR is focused, blast radius is understood, proceed to review
   - **Document coupling**: PR spans multiple concerns but they are genuinely inseparable — require a written justification in the PR description
   - **Split required**: name the minimal shippable unit and what moves to a follow-up PR

## Red Flags — Stop Immediately
- Diff exceeds 1 500 lines with no documented justification for the size
- A single PR mixes a database schema change with application feature code
- Blast radius is Wide but there are no tests for the changed paths
- The PR description does not explain why all the changed files belong together
- A breaking API change ships without a migration guide or deprecation notice
- Config or infrastructure changes are buried inside a feature PR

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "It's all one feature, splitting it would be artificial" | Features can always be broken into a data layer PR, a logic PR, and a UI PR — the split is never artificial |
| "It's faster to just ship it all at once" | A large PR blocks the team while it sits in review; split PRs merge faster in aggregate |
| "The refactor was necessary to make the feature work" | That is the definition of two separate PRs — the refactor first, the feature second |
| "Reviewers can just focus on the important parts" | Reviewers cannot reliably identify which parts matter without reviewing everything |
| "We'll clean it up in a follow-up" | Follow-up PRs for cleanup rarely land; the mess compounds |
| "Line count is misleading — most of it is auto-generated" | Auto-generated files still have blast radius; note them explicitly and exclude them from the count |
| "The tests cover it" | Coverage does not reduce blast radius — it reduces risk, which is different |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Measure | Count files, lines, directories | Size bucket assigned |
| 2. Concerns | Group files into named concerns | Count of distinct concerns determined |
| 3. Blast Radius | Callers, critical paths, reversibility | Rating: Contained / Moderate / Wide |
| 4. Structure | Refactor-feature mix, migrations, dep bumps | Red flag list complete |
| 5. Assessment | Approve / Document coupling / Split required | Written recommendation delivered |
