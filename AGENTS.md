# Repository Guidelines & Agents

## Project Structure

This repository is documentation-first, centered on skill authoring.

- Root docs: `CLAUDE.md` (project intent), `plan.md` (build workflow), `COMBINED-SKILLS-LISTING.md` (source catalog)
- Agent area: `.claude/agents/` — reusable agent prompts for building and reviewing skills
- New skill files live at repository root
- Use `docs/` for longer-form reference material

## Available Agents

### skill-writer
**File:** `.claude/agents/skill-writer.md`
**What it does:** Reads the official source skill, applies the uniform template, rewrites all wording in our own voice, adds Python examples, saves the result to the project root, and updates `plan.md`.

**Trigger:**
```
Build hunt-the-root.md
Write the red-green-commit skill
```

### skill-reviewer
**File:** `.claude/agents/skill-reviewer.md`
**What it does:** Checks five quality gates — template compliance, originality, Python examples, iron law integrity, token trim — and returns a structured pass/fail report with a final verdict.

**Trigger:**
```
Review hunt-the-root.md
Check red-green-commit.md is ready
```

### Recommended Build Workflow
```
skill-writer  →  skill-reviewer  →  fix issues  →  mark done in plan.md
```

---

## Style & Naming Conventions

- File names: kebab-case, `.md` extension (e.g. `shape-before-build.md`)
- Headings: concise and descriptive, sentence case in body text
- Examples: concrete paths and exact commands, no generic placeholders
- Template structure: follow `plan.md` exactly when creating or editing skills

## Validation Commands

```bash
rg --files                          # list all tracked files
rg "pattern" *.md                   # check wording consistency, find duplicated phrases
npx markdownlint-cli "**/*.md"      # lint Markdown before committing
git status                          # review pending changes
```

## Commit Guidelines

- Style: `type(scope): imperative summary` — e.g. `docs(skills): rewrite hunt-the-root wording`
- One logical change per commit
- PRs: include purpose, files changed, validation steps, and follow-up work
- Add before/after snippets when wording or structure changes materially

## Agent-Specific Notes

When adding agents under `.claude/agents/`:
- Keep each file single-purpose and deterministic
- Align with conventions in `plan.md` and `CLAUDE.md`
- Document the agent here in the Available Agents section
