---
name: toggle-hooks
description: Enable or disable existing hookify rules interactively by toggling the `enabled` field in each rule file.
---

# Hook Setup

## The Law
**Never delete a rule to disable it — toggle `enabled: false` so it can be re-activated without rewriting the pattern from scratch.**

## When to Use
- A hookify rule is blocking work that is currently safe to do (temporary disable)
- A previously disabled rule needs to be switched back on
- You want an overview of which rules are active before starting a risky task
- **Never skip when:** a rule is misfiring — disable it properly rather than deleting it and losing the configuration

## Process

### Phase 1: Discover Existing Rules
1. Glob for all rule files matching `.claude/hookify.*.local.md` in the project directory.
2. If no files are found, stop and report: "No rules exist yet. Run configure-hooks to create the first one."
3. For each file, read the frontmatter and extract:
   - `name` — the rule identifier
   - `enabled` — current state (`true` or `false`)
   - `event` — what it watches
   - `action` — warn or block
   - The rule message body (first sentence) for the description

### Phase 2: Present the Toggle List
1. Build a multi-select list where each option shows:
   - Label: `<rule-name> (currently <enabled|disabled>)`
   - Description: event type, action, and the first line of the rule message
2. Ask: "Which rules do you want to toggle?"
3. If the user selects nothing, report "No changes made" and stop.

### Phase 3: Apply Toggles
1. For each selected rule, flip its `enabled` field:
   - `enabled: true` becomes `enabled: false`
   - `enabled: false` becomes `enabled: true`
2. Use a targeted Edit — replace only the `enabled` line, leave everything else untouched.

Example settings.json hook block for reference (this is what loads the rules, not what you edit here):
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "hookify-runner bash"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "hookify-runner file"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "hookify-runner stop"
          }
        ]
      }
    ]
  }
}
```

Python snippet to verify a rule file's current enabled state before toggling:
```python
import re

def get_enabled_state(filepath: str) -> bool:
    with open(filepath) as f:
        content = f.read()
    match = re.search(r"^enabled:\s*(true|false)", content, re.MULTILINE)
    if not match:
        raise ValueError(f"No enabled field found in {filepath}")
    return match.group(1) == "true"
```

### Phase 4: Report Changes
1. List three groups: Enabled (newly on), Disabled (newly off), Unchanged.
2. Remind the user: changes apply on the next tool use — no restart needed.
3. Suggest running configure-hooks if the user wants to add new rules rather than just toggling existing ones.

## settings.json Hook Configuration

A working hookify setup in `settings.json` (or `settings.local.json`) looks like this:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "hookify-runner bash" }]
      },
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [{ "type": "command", "command": "hookify-runner file" }]
      }
    ],
    "PostToolUse": [],
    "Stop": [
      {
        "hooks": [{ "type": "command", "command": "hookify-runner stop" }]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [{ "type": "command", "command": "hookify-runner stop" }]
      }
    ]
  }
}
```

Key points:
- `PreToolUse` hooks run before the tool executes — use for blocking rules
- `PostToolUse` hooks run after — use for audit/logging rules
- `Stop` fires when Claude ends a task — use for completion-check rules
- `matcher` is a regex string matched against the tool name

## Red Flags — Stop Immediately
- You are about to delete a rule file instead of setting `enabled: false`
- The glob found files outside the project's `.claude/` directory — check the path before editing
- You are editing the settings.json hook wiring rather than the individual rule files (wrong tool for this job)
- The user asked to "remove" a rule — clarify whether they mean disable or permanently delete before acting

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "I'll just delete it, it's easy to recreate" | The regex pattern and message took time to tune — keep it |
| "I can remember which rules are active" | The list changes; the file is the source of truth |
| "The rule fires too much, I'll loosen the pattern while I'm here" | Scope creep — toggle only; pattern changes need user review |
| "No restart needed is probably wrong for this one" | It is always correct — the runner reads files on every invocation |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Discover | Glob `.claude/hookify.*.local.md` | Full list of rules with current state |
| 2. Present | Multi-select toggle list | User has selected which rules to flip |
| 3. Toggle | Edit `enabled` field only | Files updated, no lines deleted |
| 4. Report | Enabled / Disabled / Unchanged summary | User sees "active on next tool use" |
