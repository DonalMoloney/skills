# Personal Skill Library — Build Plan

> **Skill source list:** See `COMBINED-SKILLS-LISTING.md`
> **Agents for building:** See `AGENTS.md`

---

## Standard Changes Applied to Every Skill

These apply to all 67 skills regardless of source:

| Change | Detail |
|---|---|
| Uniform template | Apply section order: Law → When to Use → Process → Red Flags → Rationalizations → Quick Reference |
| Original wording | Every sentence rewritten — no lifted phrases from source |
| Light token trim | Remove repeated phrases and duplicate examples only; never cut rules |
| Python examples | Add Python/pytest example alongside any TypeScript-only code block |
| No flowcharts | Replace dot/graphviz diagrams with numbered phase lists |
| Rationalization table | Max 8 rows; merge duplicates, reword all |

---

## Full Skill Inventory

All 67 skills — custom name, priority, source type, source path, and **skill-specific changes** beyond the standard set.

**Priority:** `P1` daily drivers · `P2` regular use · `P3` occasional
**Source:** `SKILL` = SKILL.md · `AGENT` = agent file · `CMD` = command file · `SCRATCH` = write from description

---

### Process & Planning

| Source Skill | Our Name | P | Src | Skill-Specific Changes |
|---|---|---|---|---|
| `brainstorming` | `brainstorm.md` | P1 | SKILL | Condense visual companion section; make hard gate more prominent; trim checklist prose |
| `writing-plans` | `blueprint.md` | P1 | SKILL | Tighten scope-check section; compress bite-sized task examples |
| `executing-plans` | `execute-plan.md` | P1 | SKILL | Clarify checkpoint gates; tighten rollback instructions |
| `subagent-driven-development` | `parallel-subagents.md` | P2 | SKILL | Trim coordination protocol; add Python subprocess/async examples |
| `dispatching-parallel-agents` | `agent-dispatch.md` | P2 | SKILL | Consolidate merge-back steps; add failure-isolation notes |
| `skill-creator` | `create-skill.md` | P2 | SKILL | Reframe as personal-first workflow; trim eval scaffolding |
| `writing-skills` | `skill-spec.md` | P2 | SKILL | Tighten rigid vs flexible skill distinction; add quick checklist |
| `using-git-worktrees` | `git-worktree.md` | P2 | SKILL | Add cleanup reminder; compress command examples |
| `finishing-a-development-branch` | `finish-branch.md` | P2 | SKILL | Tighten merge-readiness checklist; compress PR checklist |
| `requesting-code-review` | `request-review.md` | P3 | SKILL | Condense context-bundle steps |
| `receiving-code-review` | `receive-review.md` | P3 | SKILL | Reframe from defensive to constructive angle |

**Source base path:** `~/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.7/skills/<name>/SKILL.md`

---

### Code Review

| Source Skill | Our Name | P | Src | Skill-Specific Changes |
|---|---|---|---|---|
| `silent-failure-hunter` | `find-silent-failures.md` | P1 | AGENT | Convert agent → skill format; add Python try/except patterns |
| `pr-test-analyzer` | `coverage-gaps.md` | P1 | AGENT | Convert agent → skill format; add pytest coverage examples |
| `code-reviewer` | `review-code.md` | P1 | AGENT | Convert agent → skill format; add CLAUDE.md compliance check step |
| `type-design-analyzer` | `review-types.md` | P2 | AGENT | Convert agent → skill format; add Python dataclass/Pydantic examples |
| `comment-analyzer` | `check-comments.md` | P2 | AGENT | Convert agent → skill format; add docstring staleness check |
| `pr-review-expert` | `pr-scope-check.md` | P2 | SCRATCH | Write from scratch: PR scope assessment, blast-radius checklist, size recommendations |

**Source base path:** `~/.claude/plugins/cache/claude-plugins-official/pr-review-toolkit/unknown/agents/<name>.md`

---

### Debugging & Verification

| Source Skill | Our Name | P | Src | Skill-Specific Changes |
|---|---|---|---|---|
| `systematic-debugging` | `debug-root-cause.md` | P1 | SKILL | Remove flowchart; add Python pdb/pytest --pdb commands; trim rationalization table 15→8 rows |
| `verification-before-completion` | `verify-before-done.md` | P1 | SKILL | Add Python verification command examples; tighten common-failures table |
| `root-cause-tracing` | `trace-bug.md` | P2 | SCRATCH | Write from scratch: backward trace technique, call-stack reading, Python traceback examples |
| `test-fixing` | `fix-failing-test.md` | P2 | SCRATCH | Write from scratch: diagnose-then-fix workflow, never-delete rule, pytest failure patterns |
| `sleuth` | `diagnose-bug.md` | P2 | SKILL | Forensic debugger; classifies bug type; builds root cause chain; fix strategy before code |
| `traceroot` | `root-cause-diagnosis.md` | P2 | SKILL | System-level symptom tracer; environment snapshot; log excavation; hypothesis ranking |
| `verdict` | `completion-audit.md` | P2 | SKILL | Completion auditor; passes/fails tasks against checklists; blocks vs highs vs lows triage |

**Source base path:** `~/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.7/skills/<name>/SKILL.md`

---

### Testing & Quality

| Source Skill | Our Name | P | Src | Skill-Specific Changes |
|---|---|---|---|---|
| `test-driven-development` | `tdd-workflow.md` | P1 | SKILL | Add pytest examples alongside TypeScript; cut code examples 4→1 pair; compress "Why Order Matters" to 4 bullets |
| `testing-anti-patterns` | `test-smells.md` | P2 | SCRATCH | Write from scratch: over-mocking, test implementation not behaviour, fragile assertions |
| `condition-based-waiting` | `async-wait.md` | P2 | SCRATCH | Write from scratch: asyncio.wait_for (Python) + waitFor (JS) patterns; no sleep rule |
| `property-based-testing` | `property-test.md` | P2 | SCRATCH | Write from scratch: Hypothesis (Python) + fast-check (TS) side-by-side |
| `mutation-testing` | `mutation-test.md` | P2 | SCRATCH | Write from scratch: mutmut (Python) + Stryker (TS); ≥80% score gate |
| `test-coverage` | `map-test-coverage.md` | P2 | SCRATCH | Write from scratch: pytest-cov + Istanbul; gap-finding workflow |
| `atdd` | `bdd-test.md` | P3 | SCRATCH | Write from scratch: Given/When/Then, Gherkin, behave (Python) |
| `tdd-guide` | `tdd-handbook.md` | P3 | SCRATCH | Write from scratch: team playbook condensed from tdd-workflow |
| `add-mutation-testing` | `mutation-setup.md` | P3 | SCRATCH | Write from scratch: setup steps for mutmut + Stryker in existing repo |

**Source base path:** `~/.claude/plugins/cache/claude-plugins-official/superpowers/5.0.7/skills/<name>/SKILL.md` (where available)

---

### Linting & Formatting

| Source Skill | Our Name | P | Src | Skill-Specific Changes |
|---|---|---|---|---|
| `simplify` | `simplify-code.md` | P2 | AGENT | Convert agent → skill format; add Python refactoring patterns |
| `reshape` | `refactor-code.md` | P2 | SKILL | Code clarity refactorer; names, responsibility, duplication, dead code, complexity, types |
| `Universal Code Linter` | `auto-lint.md` | P2 | SCRATCH | Write from scratch: language detection logic, tool matrix (ruff/eslint/etc) |
| `lint-fix` | `fix-lint.md` | P2 | SCRATCH | Write from scratch: Black + isort + Ruff + ESLint --fix + Prettier command sequence |
| `cclint` | `claude-lint.md` | P2 | SCRATCH | Write from scratch: CLAUDE.md + skill file validation rules |
| `ln-741-linter-configurator` | `lint-setup.md` | P3 | SCRATCH | Write from scratch: ESLint + Prettier + Ruff + EditorConfig setup steps |

**Source base path:** `~/.claude/plugins/cache/claude-plugins-official/pr-review-toolkit/unknown/agents/code-simplifier.md` (simplify only)

---

### Visuals & Diagramming

| Source Skill | Our Name | P | Src | Skill-Specific Changes |
|---|---|---|---|---|
| `mermaid-skill` | `draw-diagram.md` | P3 | SCRATCH | Write from scratch: diagram type selection guide, syntax quick ref |
| `design-doc-mermaid` | `code-to-diagram.md` | P3 | SCRATCH | Write from scratch: code → diagram extraction workflow |
| `mermaid-syntax-skill` | `diagram-check.md` | P3 | SCRATCH | Write from scratch: pre-render validation steps |
| `pptx` | `deck-builder.md` | P3 | SCRATCH | Write from scratch: slide structure, chart types, export steps |

---

### DevOps & Infrastructure

| Source Skill | Our Name | P | Src | Skill-Specific Changes |
|---|---|---|---|---|
| `commit-commands:commit` | `clean-commit.md` | P1 | CMD | Convert command → skill format; add co-author line; add safety protocol |
| `commit-commands:commit-push-pr` | `push-and-pr.md` | P1 | CMD | Convert command → skill format; add PR template; add confirmation gate before push |
| `incident-commander` | `incident-lead.md` | P2 | SCRATCH | Write from scratch: severity classification, comms template, timeline log |
| `ci-cd-pipeline-builder` | `pipeline-builder.md` | P2 | SCRATCH | Write from scratch: GitHub Actions + other runners; Python + Node support |
| `release-manager` | `cut-the-release.md` | P2 | SCRATCH | Write from scratch: changelog, tag, deploy sequence |
| `commit-commands:clean_gone` | `prune-branches.md` | P2 | CMD | Convert command → skill format; add dry-run step before delete |
| `monorepo-navigator` | `monorepo-nav.md` | P3 | SCRATCH | Write from scratch: workspace navigation, affected-package detection |
| `schedule` | `cron-schedule.md` | P3 | SCRATCH | Write from scratch: cron agent setup, schedule management |

**Source base path:** `~/.claude/plugins/cache/claude-plugins-official/commit-commands/unknown/commands/<name>.md`

---

### Prompting

| Source Skill | Our Name | P | Src | Skill-Specific Changes |
|---|---|---|---|---|
| `codex:gpt-5-4-prompting` | `prompt-craft.md` | P2 | SCRATCH | Write from scratch: model-agnostic prompt patterns; chain-of-thought, few-shot, system prompt design |

---

### Official Anthropic

| Source Skill | Our Name | P | Src | Skill-Specific Changes |
|---|---|---|---|---|
| `feature-dev` | `feature-builder.md` | P1 | SCRATCH | Write from scratch: Explorer → Architect → Reviewer phases with clear handoff gates |
| `webapp-testing` | `e2e-test.md` | P2 | SCRATCH | Write from scratch: Playwright setup, selector strategy, assertion patterns |
| `frontend-design` | `ui-patterns.md` | P2 | SKILL | Tighten component pattern examples; add accessibility checklist |
| `codex:rescue` | `codex-rescue.md` | P2 | AGENT | Convert agent → skill format; clarify handoff criteria |

**Source base paths:**
- frontend-design: `~/.claude/plugins/cache/claude-plugins-official/frontend-design/unknown/skills/frontend-design/SKILL.md`
- codex:rescue: `~/.claude/plugins/cache/openai-codex/codex/1.0.4/agents/codex-rescue.md`

---

### Claude Code Configuration

| Source Skill | Our Name | P | Src | Skill-Specific Changes |
|---|---|---|---|---|
| `hookify` | `generate-hooks.md` | P2 | CMD | Convert command → skill format; add hook trigger reference table |
| `hookify:configure` | `toggle-hooks.md` | P2 | CMD | Convert command → skill format; add settings.json examples |
| `claude-md-management:claude-md-improver` | `improve-claude-md.md` | P2 | SKILL | Tighten improvement criteria; add structure checklist |
| `claude-md-management:revise-claude-md` | `refine-claude-md.md` | P2 | SCRATCH | Write from scratch: revision workflow, clarity checks, accuracy verification |
| `update-config` | `edit-settings.md` | P2 | SCRATCH | Write from scratch: settings.json structure, hooks config, permissions |
| `init` | `init-project.md` | P2 | SCRATCH | Write from scratch: new project checklist, CLAUDE.md bootstrap |
| `review` | `quick-review.md` | P2 | SCRATCH | Write from scratch: fast code review trigger, git diff workflow |
| `security-review` | `security-scan.md` | P2 | SCRATCH | Write from scratch: OWASP top 10 checklist, secrets scan, dep audit |
| `fewer-permission-prompts` | `fewer-prompts.md` | P3 | SCRATCH | Write from scratch: transcript scan → allowlist generation |
| `keybindings-help` | `remap-keys.md` | P3 | SCRATCH | Write from scratch: keybindings.json structure, chord binding syntax |
| `loop` | `loop-interval.md` | P3 | SCRATCH | Write from scratch: interval setup, loop prompt design |

**Source base paths:**
- hookify: `~/.claude/plugins/cache/claude-plugins-official/hookify/unknown/commands/<name>.md`
- claude-md-improver: `~/.claude/plugins/cache/claude-plugins-official/claude-md-management/1.0.0/skills/claude-md-improver/SKILL.md`

---

## Build Summary

| Tier | Count | Done |
|---|---|---|
| P1 | 14 | 14 |
| P2 | 36 | 36 |
| P3 | 17 | 17 |
| **Total** | **67** | **67** |

Source breakdown: 18 SKILL · 8 AGENT · 6 CMD · 35 SCRATCH

---

## Uniform Template

```markdown
---
name: <custom-name>
description: <one-line trigger — when Claude should invoke this skill>
---

# <Title>

## The Law
**[Single iron-law statement — bold, imperative, non-negotiable]**

## When to Use
- Trigger scenario 1
- **Never skip when:** [scenarios people rationalize away]

## Process

### Phase 1: <Name>
1. Step
   - Sub-detail

## Red Flags — Stop Immediately
- ...

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
```

---

## Build Checklist (per skill)

- [ ] Identify source type from inventory table
- [ ] Read source file — or write from description if SCRATCH
- [ ] Apply uniform template
- [ ] Apply skill-specific changes listed in the table above
- [ ] Rewrite all wording — no copy-pasted sentences
- [ ] Add Python examples where source is TypeScript-only
- [ ] Light trim — remove redundancy, keep effectiveness
- [ ] Save to project root as `<our-name>.md`
- [ ] Update "Done" count in Build Summary above

---

## Agents Available

| Agent | File | Purpose |
|---|---|---|
| `skill-writer` | `.claude/agents/skill-writer.md` | Reads source, applies changes, saves skill file |
| `skill-reviewer` | `.claude/agents/skill-reviewer.md` | Checks template, originality, Python examples |
