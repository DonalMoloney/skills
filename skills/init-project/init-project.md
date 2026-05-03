---
name: init-project
description: Bootstrap a new Claude Code project with a CLAUDE.md, settings.json, and .gitignore entries so every future session starts with full context.
---

# Project Start

## The Law
**Every project gets a CLAUDE.md before any code is written — a session without one forces Claude to rediscover the same context every time.**

## When to Use
- Initializing a brand-new repository
- Bringing an existing codebase under Claude Code management for the first time
- Handing a project to a teammate who will use Claude Code on it
- **Never skip when:** the project will have more than one Claude session — the second session without a CLAUDE.md always wastes time re-establishing what was obvious in the first

## Process

### Phase 1: Gather Project Facts
1. Ask (or determine from context) the answers to these five questions:
   - What does this project do in one sentence?
   - What language(s) and framework(s) does it use?
   - What is the build/run/test command?
   - Are there any env vars or tools that must be installed first?
   - Are there any known gotchas or constraints Claude should know from the start?
2. If the project directory already has code, scan it to fill in gaps:
   ```bash
   ls -la
   cat package.json 2>/dev/null || cat pyproject.toml 2>/dev/null || cat Makefile 2>/dev/null
   ```

### Phase 2: Write CLAUDE.md
1. Create `CLAUDE.md` at the project root using only facts confirmed in Phase 1.
2. Start from this minimal bootstrap template — expand only where there is real content to add:

```markdown
# <Project Name>

## What This Is
<One sentence description.>

## Commands

```bash
# Development
<run command>

# Testing
<test command>

# Linting
<lint command>
```

## Architecture
<Two to four sentences on directory layout and key boundaries.>

## Environment
- Required tools: <list>
- Required env vars: <list or "none">

## Gotchas
- <First non-obvious thing Claude should know>
```

Python project CLAUDE.md example:
```markdown
# invoice-parser

## What This Is
Parses PDF invoices into structured JSON using pdfplumber and Pydantic.

## Commands

```bash
# Install
pip install -e ".[dev]"

# Testing
pytest -x -q

# Linting
ruff check . && ruff format --check .
```

## Architecture
src/invoice_parser/ contains the core extraction logic.
tests/ mirrors the src/ layout.
The models/ directory holds Pydantic schemas for each supported invoice format.

## Environment
- Required tools: Python 3.11+, poppler (for pdfplumber)
- Required env vars: none

## Gotchas
- pdfplumber coordinate system starts from bottom-left, not top-left
- Always run tests against the fixtures in tests/fixtures/ before adding new invoice types
```

### Phase 3: Create .claude/ Directory
1. Create `.claude/` if it does not exist:
   ```bash
   mkdir -p .claude
   ```
2. Write `.claude/settings.json` with the minimal working structure:
   ```json
   {
     "permissions": {
       "allow": [],
       "deny": []
     }
   }
   ```
3. Add `.claude/settings.local.json` to `.gitignore` — personal settings must never be committed:
   ```bash
   echo ".claude/settings.local.json" >> .gitignore
   echo ".claude.local.md" >> .gitignore
   ```

### Phase 4: New Project Checklist
Run through this list before declaring setup complete:

- [ ] `CLAUDE.md` exists at project root with at least: description, commands, and one gotcha
- [ ] Every documented command has been run and exits cleanly
- [ ] `.claude/settings.json` exists (even if minimal)
- [ ] `.claude/settings.local.json` and `.claude.local.md` are in `.gitignore`
- [ ] No secrets or personal paths are in any committed file
- [ ] If the project uses hooks, hookify rule files are in `.claude/`

### Phase 5: First-Session Handoff
Tell the user two things:
1. Press `#` at any point during a session to have Claude incorporate what it just learned into CLAUDE.md automatically.
2. Use `improve-claude-md` after the first real working session to fill in gaps that only emerge from actual use.

## Red Flags — Stop Immediately
- CLAUDE.md contains a command that failed when you ran it to verify
- A secret key or personal path is about to be written into the committed `settings.json`
- The CLAUDE.md was populated with guesses rather than verified facts
- `.claude/settings.local.json` is not in `.gitignore` before being created

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "I'll add CLAUDE.md after I understand the project better" | Every session before it has to rediscover the same facts from scratch |
| "The commands are obvious, no need to document them" | Obvious to you today; not obvious to Claude or a teammate next month |
| "I'll put personal settings in the shared settings.json" | Personal API keys or paths in a committed file are a security incident waiting to happen |
| "The project is too small to need this" | Small projects grow; retrofitting CLAUDE.md is harder than starting with one |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Gather | Confirm description, commands, env, gotchas | Five facts answered and verified |
| 2. Write | Create CLAUDE.md from template with real content | File exists, every command verified |
| 3. Structure | Create `.claude/`, write settings.json, update .gitignore | Directory present, local files excluded from git |
| 4. Checklist | Verify all boxes | All items checked |
| 5. Handoff | Communicate `#` shortcut and improve-claude-md next step | User knows how to keep the file current |
