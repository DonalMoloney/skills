---
name: remap-keys
description: Customize Claude Code keyboard shortcuts by editing ~/.claude/keybindings.json — single keys, chords, and multi-step sequences.
---

# Rebind Keys

## The Law
**Always read `~/.claude/keybindings.json` before writing to it — and always validate that the binding you are adding does not conflict with an existing one.**

## When to Use
- A default keybinding conflicts with a terminal emulator or editor shortcut
- You want a faster chord for a frequently used slash command
- A team member needs a custom binding documented so others can adopt it
- **Never skip when:** a user says a key "doesn't work" — check for a conflict in keybindings.json before assuming a bug

## Process

### Phase 1: Read the Current Bindings
1. Read `~/.claude/keybindings.json`. If it does not exist, note that and start from an empty array `[]`.
2. List every binding currently defined: key, command, and any conditions.
3. Identify if the key the user wants to bind is already taken.

### Phase 2: Understand the File Structure
keybindings.json is a JSON array of binding objects:

```json
[
  {
    "key": "ctrl+shift+r",
    "command": "/review"
  },
  {
    "key": "ctrl+k ctrl+b",
    "command": "/blueprint"
  },
  {
    "key": "ctrl+shift+h",
    "command": "/hookify"
  },
  {
    "key": "ctrl+shift+t",
    "command": "/tdd-workflow"
  }
]
```

**Key format rules:**
- Modifiers: `ctrl`, `shift`, `alt`, `meta` (always lowercase)
- Separator: `+` between modifiers and key, space between chord steps
- Regular keys: lowercase letter or named key (`enter`, `escape`, `tab`, `f1`–`f12`)
- Chord (two-step sequence): two `"key"` strings separated by a space: `"ctrl+k ctrl+b"`

**Command field:** any slash command string, including arguments (e.g. `"/loop 5m /review"`)

### Phase 3: Check for Conflicts
Before adding a new binding:
1. Scan the existing bindings for the exact key string.
2. Check against common terminal defaults that Claude Code inherits:

| Key | Typical Terminal Use | Safe to Rebind? |
|-----|---------------------|-----------------|
| `ctrl+c` | Interrupt / copy | No |
| `ctrl+z` | Suspend process | No |
| `ctrl+l` | Clear screen | Use with caution |
| `ctrl+shift+*` | Rarely claimed | Generally yes |
| `ctrl+k ctrl+*` | Chord — rarely claimed | Generally yes |
| `alt+*` | Terminal meta sequences | Varies by terminal |

3. If there is a conflict, propose an alternative binding before writing.

### Phase 4: Write the Binding
1. Append the new binding object to the array — do not replace the whole file.
2. Single key binding example:
   ```json
   {
     "key": "ctrl+shift+d",
     "command": "/debug-root-cause"
   }
   ```
3. Chord binding example:
   ```json
   {
     "key": "ctrl+k ctrl+d",
     "command": "/debug-root-cause"
   }
   ```
4. Binding with arguments example:
   ```json
   {
     "key": "ctrl+shift+l",
     "command": "/loop 10m /review"
   }
   ```
5. Validate the JSON after writing:
   ```bash
   python3 -m json.tool ~/.claude/keybindings.json > /dev/null && echo "valid" || echo "INVALID"
   ```

### Phase 5: Confirm and Document
1. Read the file back and show the user the final binding list.
2. Note that keybindings take effect after restarting the Claude Code session.
3. If the user has a CLAUDE.md with a workflow section, suggest adding a one-line note about the custom binding so teammates can adopt it.

Example CLAUDE.md note to suggest:
```markdown
## Custom Keybindings
- `ctrl+shift+r` — `/review` (run code review)
- `ctrl+k ctrl+b` — `/blueprint` (write a plan)
```

## Red Flags — Stop Immediately
- You are about to overwrite `~/.claude/keybindings.json` without reading it first
- The key being added is already in the file — writing it again creates a duplicate, not an override
- A chord's first step (`ctrl+k`) conflicts with an existing single-key binding
- The JSON is invalid after your edit and you have not fixed it before reporting success

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "ctrl+c is unused in Claude Code" | It is the terminal interrupt — never rebind it |
| "I'll just overwrite the file, it's simpler" | Any existing custom bindings the user has are silently deleted |
| "Duplicates don't matter, last one wins" | Undefined behavior — validate and deduplicate instead |
| "Chords are confusing, just use a single key" | Chords avoid all single-key conflicts; they are the safer choice for power users |
| "Keybindings are active immediately" | They require a session restart — always tell the user this |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Read | Open `~/.claude/keybindings.json` | Current bindings listed, file state known |
| 2. Structure | Understand key format and command field | Desired binding is correctly formed |
| 3. Conflicts | Check existing bindings and terminal defaults | No conflict found, or alternative proposed |
| 4. Write | Append binding, validate JSON | File is valid, new entry confirmed |
| 5. Confirm | Show final list, note restart requirement | User knows binding is pending restart |
