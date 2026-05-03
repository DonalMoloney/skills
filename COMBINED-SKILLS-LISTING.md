# Personal Skill Library — Complete Listing

All 67 custom skills organized by category. Each skill has been rewritten with original wording, the uniform template, and Python + TypeScript examples where applicable.

## P1 (Daily Drivers) — 14 Skills

### Process & Planning
- brainstorm — Turn ideas into fully formed designs through collaborative dialogue
- blueprint — Create step-by-step implementation plans from approved specifications
- execute-plan — Track and execute multi-step plans from start to completion

### Code Review
- find-silent-failures — Identify inadequate error handling and silent failures
- coverage-gaps — Review test coverage gaps and missing edge cases
- review-code — Complete code review for bugs, logic errors, and quality

### Debugging & Verification
- debug-root-cause — Systematic debugging from symptom to confirmed root cause
- verify-before-done — Verify code works through manual testing and automation checks

### Testing & Quality
- tdd-workflow — Test-driven development workflow (red-green-commit)

### DevOps & Infrastructure
- clean-commit — Safe commit workflow with atomic changes and clear messages
- push-and-pr — Push to remote and open a PR with complete context

### Official Anthropic
- feature-builder — Build features through Explorer → Architect → Reviewer phases

---

## P2 (Regular Use) — 36 Skills

### Process & Planning
- parallel-subagents — Coordinate parallel agents by domain to prevent overlap
- agent-dispatch — Dispatch agents with clear domains and handoff points
- create-skill — Write new skills following the uniform template and law pattern
- skill-spec — Design rigid vs flexible skills with evaluation scaffolding
- git-worktree — Use git worktrees for isolated feature branches
- finish-branch — Final checklist before opening a PR (tests, commits, push)
- request-review — Request code review with context and expectations
- receive-review — Receive review feedback constructively and iterate

### Code Review
- review-types — Type design analysis for encapsulation and invariants
- check-comments — Review comments for accuracy, staleness, and noise
- pr-scope-check — Assess PR scope, blast radius, and complexity

### Debugging & Verification
- trace-bug — Backward trace from symptom to root cause
- fix-failing-test — Fix broken tests (never delete them)
- diagnose-bug — Forensic debugging with classification and root cause chain
- root-cause-diagnosis — System-level diagnosis using logs and environment state
- completion-audit — Task completion checklist with verdict: PASS/FAIL/CONDITIONAL

### Testing & Quality
- test-smells — Identify over-mocking, implementation testing, fragile assertions
- async-wait — Condition-based waiting in tests (never use sleep)
- property-test — Property-based testing with Hypothesis and fast-check
- mutation-test — Mutation testing to verify test quality
- map-test-coverage — Measure test coverage and find gaps

### Linting & Formatting
- simplify-code — Review code for reuse, quality, and efficiency
- refactor-code — Refactor code for clarity (names, responsibility, duplication)
- auto-lint — Auto-fix linting and formatting issues
- fix-lint — Run linters and fix issues across languages
- claude-lint — Validate skill files and CLAUDE.md for compliance

### DevOps & Infrastructure
- incident-lead — Lead incidents with severity classification and timeline
- pipeline-builder — Build CI/CD pipelines with GitHub Actions and other runners
- cut-the-release — Release management with changelog, tag, and deployment
- prune-branches — Clean up old and merged branches safely

### Prompting
- prompt-craft — Craft prompts with chain-of-thought, few-shot, and system design

### Claude Code Configuration
- generate-hooks — Generate hooks for automated Claude Code behavior
- toggle-hooks — Enable/disable hooks in settings
- improve-claude-md — Improve CLAUDE.md documentation
- refine-claude-md — Revision workflow for CLAUDE.md clarity
- edit-settings — Configure settings.json (hooks, permissions, env vars)
- init-project — New project checklist and CLAUDE.md bootstrap
- quick-review — Fast code review from `git diff`
- security-scan — OWASP top 10, secrets scan, dependency audit

### Official Anthropic
- e2e-test — End-to-end testing with Playwright
- ui-patterns — UI component patterns and accessibility
- codex-rescue — Rescue stuck subagents with diagnosis and new approach

---

## P3 (Occasional) — 17 Skills

### Visuals & Diagramming
- draw-diagram — Mermaid diagram creation with type selection guide
- code-to-diagram — Extract architecture diagrams from code
- diagram-check — Pre-render diagram validation

### Testing & Quality
- bdd-test — Behavior-driven testing with Given/When/Then and Gherkin
- tdd-handbook — Team playbook condensed from TDD workflow
- mutation-setup — Setup mutmut and Stryker for mutation testing

### Linting & Formatting
- lint-setup — ESLint, Prettier, Ruff, EditorConfig setup

### Visuals & Diagramming
- deck-builder — PowerPoint slide creation and structure

### DevOps & Infrastructure
- monorepo-nav — Navigate monorepos and detect affected packages
- cron-schedule — Cron job setup and schedule management

### Claude Code Configuration
- fewer-prompts — Scan transcripts and build permission allowlist
- remap-keys — Customize keybindings in keybindings.json
- loop-interval — Set up recurring task loops with intervals

### Code Review
- request-review → PR context bundling
- receive-review → Constructive review response

---

## Statistics

- **Total Skills:** 67
- **P1 (Daily):** 14 (21%)
- **P2 (Regular):** 36 (54%)
- **P3 (Occasional):** 17 (25%)

**Source breakdown:**
- From SKILL files: 18
- From AGENT files: 8
- From CMD files: 6
- Written from SCRATCH: 35

---

## Key Improvements Over Official Skills

Every skill in this library:
- ✅ Uses the uniform template (Law, When, Process, Red Flags, Rationalizations, Reference)
- ✅ Has original wording (no phrases copied from official skills)
- ✅ Includes Python + TypeScript examples (not just one language)
- ✅ Uses numbered phases instead of flowcharts
- ✅ Has 5-8 red flags and rationalizations (not generic lists)
- ✅ Includes a Quick Reference table for scanning
- ✅ Fits the personal workflow and voice

---

## Organization

Skills are organized in `/skills/` subfolder:
- Each skill in its own directory: `/skills/skill-name/`
- Skill file: `/skills/skill-name/skill-name.md`
- Frontmatter with `name` and `description` fields
- Clean, consistent formatting across all 67 skills
