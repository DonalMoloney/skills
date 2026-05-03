---
name: fewer-prompts
description: Scan recent transcripts to find tool calls that repeatedly trigger permission prompts, then add a targeted allowlist to .claude/settings.json.
---

# Reduce Prompts

## The Law
**Only add to the allowlist what you have observed in transcripts — never guess at permissions or add broad patterns to avoid future prompts that haven't appeared yet.**

## When to Use
- Permission prompts are interrupting flow on every session for the same read-only operations
- Onboarding a team member whose clean environment will face constant approval requests
- After running improve-claude-md or project-start and wanting a smooth hands-off experience
- **Never skip when:** the same prompt has appeared three or more times in recent transcripts — that cadence will not improve on its own

## Process

### Phase 1: Collect Transcript Evidence
1. Locate Claude Code transcripts for this project:
   ```bash
   ls ~/.claude/projects/ | grep "$(basename $(pwd))"
   # or find by path hash
   find ~/.claude/projects/ -name "*.jsonl" -newer .claude/settings.json 2>/dev/null | head -20
   ```
2. Read the most recent transcript files (last 5–10 sessions).
3. Extract every tool call that was preceded by a permission prompt. Record:
   - Tool name (Bash, Read, Glob, etc.)
   - Exact command or path pattern
   - How many times it appeared across sessions

### Phase 2: Classify by Risk
Sort each observed call into one of three tiers:

| Tier | Description | Allow? |
|------|-------------|--------|
| Safe | Read-only, no side effects (cat, ls, grep, find, git status) | Yes — add to allowlist |
| Low-risk | Scoped writes with known targets (git commit, npm run test) | Yes — with specific pattern |
| Risky | Broad writes, installs, network calls, deletions | No — keep the prompt |

Anything in the Risky tier stays out of the allowlist regardless of frequency. The prompt is the protection.

### Phase 3: Generate the Allowlist
1. Build the allow array using observed patterns, not guesses:

```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(ls *)",
      "Bash(find . *)",
      "Bash(grep *)",
      "Bash(cat *.json)",
      "Bash(npm run test)",
      "Bash(pytest *)",
      "Read(*)"
    ],
    "deny": []
  }
}
```

Python/shell equivalent for generating the allow block from transcript data:
```python
import json, re
from collections import Counter

SAFE_PATTERNS = re.compile(
    r"^(git\s+(status|diff|log|show)|ls|find\s+\.|grep|cat\s+\S+\.(json|md|txt)|"
    r"npm\s+run\s+(test|lint|build)|pytest)"
)

def extract_safe_commands(transcript_lines: list[str]) -> list[str]:
    commands = []
    for line in transcript_lines:
        data = json.loads(line)
        if data.get("type") == "tool_use" and data.get("name") == "Bash":
            cmd = data.get("input", {}).get("command", "")
            if SAFE_PATTERNS.match(cmd.strip()):
                commands.append(cmd.strip())
    counts = Counter(commands)
    # Only add commands seen 3+ times
    return [f"Bash({cmd})" for cmd, n in counts.items() if n >= 3]
```

2. Order the allow list: most frequent patterns first, most specific before most general.
3. Prefer specific patterns over wildcards:
   - Prefer `Bash(npm run test)` over `Bash(npm *)`
   - Prefer `Bash(git diff *)` over `Bash(git *)`
   - Never use `Bash(*)` — that disables all Bash protection

### Phase 4: Apply to settings.json
1. Read the current `.claude/settings.json`.
2. Merge the new allow entries into the existing permissions block — do not overwrite entries that are already there.
3. Validate the JSON:
   ```bash
   python3 -m json.tool .claude/settings.json > /dev/null && echo "valid" || echo "INVALID"
   ```
4. Report to the user:
   - How many patterns were added
   - Which were excluded and why (tier classification)
   - That permissions take effect immediately — no restart needed

## Red Flags — Stop Immediately
- You are about to add `Bash(*)` or `Read(*)` as catch-all allowlists
- A command being added is destructive (rm, git push --force, pip install without a venv)
- You are adding permissions based on what you think Claude will need, not what transcripts show
- The JSON fails validation after your edit

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "Adding npm * is easier than listing each script" | A broad allow is a blank check — it will cover scripts that should stay gated |
| "I know what's safe, no need to check transcripts" | Transcripts are the evidence; intuition is not |
| "One more deletion command won't hurt" | Every destructive command that bypasses a prompt is a mistake waiting to happen |
| "I'll add it broadly now and tighten later" | "Later" never comes — set the right scope from the start |
| "This project is internal, security doesn't matter" | Allowlists scope what Claude runs automatically; scope matters on any machine |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Collect | Read transcripts, extract prompted tool calls | List of commands with frequency counts |
| 2. Classify | Sort into Safe / Low-risk / Risky | Each command has a tier and a decision |
| 3. Generate | Build allow array from Safe + Low-risk tiers only | Allowlist drafted, specificity confirmed |
| 4. Apply | Merge into settings.json, validate JSON, report | Valid file, user told what was added and what was excluded |
