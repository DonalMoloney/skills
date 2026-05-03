---
name: generate-hooks
description: Scan the current conversation for unwanted Claude behaviors and generate hookify rule files that block or warn on them automatically.
---

# Configure Hooks

## The Law
**Every recurring unwanted behavior must be captured in a hook rule file — never rely on a standing instruction or memory that vanishes when the session ends.**

## When to Use
- You have just corrected Claude for the second time on the same mistake
- You want to prevent a specific risky command (e.g. `rm -rf`, force-push) before it happens again
- A project requires hard guardrails that must survive session restarts
- **Never skip when:** the same misbehavior has happened more than once — a warning in chat will not stick, a hook will

## Process

### Phase 1: Surface the Behavior
1. If a specific behavior was supplied as an argument, note it and proceed to Phase 2.
2. If no argument was given, scan the last 20–30 user messages for:
   - Explicit corrections ("don't do X", "I said no console.log")
   - Reversions where the user undid Claude's action
   - Frustrated questions ("why did you do X?")
   - The same problem appearing more than once
3. For each candidate, record:
   - Which tool fired (Bash, Edit, Write, etc.)
   - The precise pattern or command
   - Why it caused a problem

### Phase 2: Confirm with the User
1. Present up to four detected behaviors as a multi-select list.
2. For each item the user picks, ask: **warn** (allow but notify) or **block** (prevent outright)?
3. Show the proposed regex pattern and let the user refine it before writing any file.

### Phase 3: Write Rule Files
1. Verify `.claude/` exists in the working directory; create it if not.
2. For each confirmed behavior, write `.claude/hookify.<rule-name>.local.md`.
   - Name format: action verb + subject in kebab-case (e.g. `block-dangerous-rm`, `warn-env-file-edit`)
   - Frontmatter fields: `name`, `enabled: true`, `event`, `pattern`, `action`
3. Single-condition rule template:
   ```markdown
   ---
   name: block-dangerous-rm
   enabled: true
   event: bash
   pattern: rm\s+-rf
   action: block
   ---

   This command was blocked because rm -rf without a specific target is too risky.
   Provide the exact path you intend to delete and proceed from there.
   ```
4. Multi-condition rule template (use when path and content must both match):
   ```markdown
   ---
   name: warn-env-file-edit
   enabled: true
   event: file
   conditions:
     - field: file_path
       operator: regex_match
       pattern: \.env$
     - field: new_text
       operator: contains
       pattern: SECRET
   ---

   Editing a secret value in a .env file — confirm this is intentional.
   ```
5. Python regex validation before saving:
   ```python
   import re
   pattern = r"rm\s+-rf"
   test_input = "rm -rf /tmp/test"
   assert re.search(pattern, test_input), "Pattern does not match expected input"
   ```

### Phase 4: Verify and Report
1. List the created files to confirm they landed in the project `.claude/` directory, not the plugin's.
2. Report each rule: name, event type, action, and the pattern it watches.
3. State clearly: rules are active on the next tool use — no restart required.

## Hook Trigger Reference

| Event | Fires On | Typical Use |
|-------|----------|-------------|
| `bash` | Any Bash tool invocation | Block dangerous shell commands |
| `file` | Edit, Write, MultiEdit | Guard source files or secrets |
| `stop` | Claude attempting to end the session | Require completion checks |
| `prompt` | Each user message submitted | Screen prompts for required context |
| `all` | Every tool use | Broad audit or logging rules |

## Red Flags — Stop Immediately
- You are about to write the rule file into the plugin's `.claude/` directory instead of the project's
- The regex pattern is so broad it will fire on routine operations
- The user has not confirmed the action (warn vs block) before you write the file
- You are creating a `block` rule for a pattern the user only asked to be warned about

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "I'll just remember not to do that" | Memory does not persist; hooks do |
| "The pattern might be too aggressive, I'll soften it" | A missed trigger is worse than a false positive — let the user decide |
| "I'll put it in the project root, not .claude/" | Files outside `.claude/` are not loaded by the hook system |
| "warn is fine, block is overkill" | Only the user can make that call — ask, don't decide |
| "No restart needed, so the rule is probably not active yet" | Rules activate on the very next tool call — they are live immediately |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Surface | Scan conversation for repeated corrections | Candidate behavior list ready |
| 2. Confirm | User picks behaviors and selects warn/block | Confirmed list + patterns approved |
| 3. Write | Create `.claude/hookify.<name>.local.md` | Files present in project `.claude/` |
| 4. Verify | List files, report rule summary | User sees "active immediately" confirmation |
