---
name: edit-settings
description: Use to modify Claude Code settings — configure hooks, permissions, environment variables, and other harness behavior through settings.json.
---

# Edit Settings

## The Law
**Settings configuration is code. Document every hook, permission, and env var. Preserve existing settings while adding new ones. Test that settings take effect.**

## When to Use
- Adding automated hooks to Claude Code (before/after actions)
- Configuring permissions allowlist
- Setting environment variables for a project
- Changing harness behavior (timeouts, model preferences)
- **Never skip when:** automating frequent actions — hooks save time multiplicatively

## Process

### Phase 1: Understand Settings Structure
1. Locate settings file:
   - `.claude/settings.json` (project-specific)
   - `~/.claude/settings.json` (user-wide)
2. Review current structure:
   - `hooks` — automated shell scripts
   - `permissions` — allowlist for tools
   - `env` — environment variables
   - Other harness config

### Phase 2: Identify What to Configure
1. What behavior needs to be automated?
   - Running tests on save?
   - Linting before commit?
   - Deploying on merge?
2. What permissions are missing?
   - Does the user need to approve every bash command?
   - Can specific commands run automatically?

### Phase 3: Add Hooks (if needed)
1. Hooks run shell commands on events:
   ```json
   {
     "hooks": {
       "on-before-commit": "npm test"
     }
   }
   ```
2. Common events:
   - `on-before-commit` — before staging files
   - `on-after-success` — after successful task
   - `on-error` — when something fails
3. Keep hook commands simple and fast

### Phase 4: Add Permissions (if needed)
1. Allowlist reduces permission prompts:
   ```json
   {
     "permissions": {
       "read": ["**/*.md", "**/*.json"],
       "bash": ["npm", "git", "yarn"]
     }
   }
   ```
2. Common safe commands to allowlist:
   - `npm`, `yarn`, `pip` (package managers)
   - `git` (version control)
   - `node`, `python` (interpreters)

### Phase 5: Add Env Vars (if needed)
1. Environment variables for the project:
   ```json
   {
     "env": {
       "NODE_ENV": "development",
       "DEBUG": "1"
     }
   }
   ```
2. Do NOT put secrets here — use `.env.local`

### Phase 6: Test Settings
1. Reload Claude Code
2. Verify hooks run on events
3. Verify permissions work as expected
4. Verify env vars are set:
   ```bash
   echo $NODE_ENV
   ```

## Red Flags — Stop Immediately

- Putting secrets in settings.json — use `.env.local` instead
- A hook command is slow or blocking — make it fast or remove it
- A hook fails silently — test it before committing
- Overwriting existing settings without reading them — always merge
- Settings file has invalid JSON — it won't parse

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "We'll add hooks later" | Hooks automate routine work. Add them when you notice repetition. |
| "The permission allowlist is too restrictive" | Better to be restrictive and add as needed than to allowlist everything. |
| "Environment variables in settings are fine" | Only non-secret config. Secrets go in `.env.local` or secrets manager. |
| "The settings are too complex" | Document them. Clear comments prevent confusion. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Understand** | Locate settings file; review structure | Structure is clear |
| **Identify** | What behavior needs automation? | Automation goals are listed |
| **Hooks** | Add hook for event (e.g., before-commit) | Hook is configured and documented |
| **Permissions** | Allowlist safe commands to reduce prompts | Permissions are explicit |
| **Env** | Add non-secret environment variables | Env vars are set |
| **Test** | Reload harness; verify hooks and perms work | Settings are active and working |
