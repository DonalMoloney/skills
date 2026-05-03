---
name: claude-lint
description: Use to validate skill files and CLAUDE.md — check template compliance, frontmatter, The Law statement, process structure. Ensures consistency across custom skills.
---

# Claude Lint

## The Law
**Skill files follow a uniform template. Validate every skill against: frontmatter, The Law, When to Use, Process phases, Red Flags, Rationalizations, and Quick Reference. No deviations.**

## When to Use
- A new skill is ready for use
- A skill was modified and you need validation
- Auditing skill quality across the codebase
- **Never skip when:** a skill is about to be released — validation prevents poor quality

## Process

### Phase 1: Check Frontmatter
1. Verify frontmatter exists:
   ```yaml
   ---
   name: <skill-name>
   description: <one-line trigger>
   ---
   ```
2. Verify all fields are present:
   - `name` — lowercase, hyphenated, no spaces
   - `description` — one line, describes when to invoke
3. Verify `description` is NOT a placeholder

### Phase 2: Check The Law
1. Verify the skill has a "## The Law" section
2. The Law must be:
   - A single statement
   - Bold and imperative
   - Non-negotiable rule, not a guideline
3. Examples of good laws:
   - "Never use sleep(). Use condition-based waiting instead."
   - "Do not mark work done without verifying it works."

### Phase 3: Check When to Use
1. Verify "## When to Use" section exists
2. Should have 3-5 bullet points
3. Should include "Never skip when:" — describing rationalization scenarios
4. Should not have placeholders

### Phase 4: Check Process
1. Verify process is divided into 5-7 numbered phases
2. Each phase should be a clear section: `### Phase N: <Name>`
3. Each phase should have 1-3 action items
4. No vague steps like "implement X"

### Phase 5: Check Red Flags
1. Verify "## Red Flags — Stop Immediately" exists
2. Should have 5-8 bullet points
3. Each should be a real stop condition
4. Should match the Law and Process

### Phase 6: Check Rationalizations
1. Verify "## Common Rationalizations" exists
2. Should be a table: Excuse | Why It's Wrong
3. Should have 5-8 rows
4. Should match common objections to the Law

### Phase 7: Check Quick Reference
1. Verify "## Quick Reference" exists
2. Should be a table: Phase | Core Action | Done When
3. Each row summarizes one phase
4. Should be scannable in < 1 minute

### Phase 8: Validate File Quality
1. No placeholder text: "TBD", "TODO", "FIXME", "<insert here>"
2. No copy-pasted sentences from other skills
3. Consistent formatting and tone
4. No grammar errors

## Red Flags — Stop Immediately

- Frontmatter is missing or incomplete — cannot publish without it
- The Law is vague or phrased as a guideline — rewrite as imperative command
- Process has more than 7 phases — too complex, simplify
- Red Flags section is missing — cannot identify failure modes
- Rationalizations table is missing — cannot guide against common mistakes
- Placeholder text remains — remove all TBD/TODO before publishing
- Description field is a placeholder — write a clear trigger phrase

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "We can add the Quick Reference later" | Quick reference is essential. It's the skill in one glance. |
| "The Law section is too strict" | Strict laws prevent misuse. Loose guidelines are ignored. |
| "We don't need 5-8 rationalizations" | You do. People will rationalize every other way. Cover them. |
| "This skill is similar to X, we can copy" | Don't copy. Reword everything. Different voice feels custom, not derivative. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Frontmatter** | Verify name, description, no placeholders | All fields present and valid |
| **Law** | Check The Law is bold, imperative, non-negotiable | Law is clear and strict |
| **When** | Verify 3-5 triggers and "Never skip when" | Triggers are explicit |
| **Process** | Check 5-7 phases with 1-3 steps each | Each phase is actionable |
| **Red Flags** | Verify 5-8 stop-immediately conditions | Red Flags match the Law |
| **Rationalizations** | Check 5-8 excuse/reason pairs | Common objections are covered |
| **Reference** | Verify scannable table: Phase/Action/Done | Reference is <= 1 minute scan |
| **Quality** | No placeholders, no copies, grammar OK | File is publication-ready |
