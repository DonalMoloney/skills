---
name: skill-spec
description: Use when authoring or editing a skill file, deciding what belongs in a skill versus CLAUDE.md, or verifying a skill is structured for reliable agent compliance
---

# Write Skill Rules

## The Law
**Every rule in a skill must address a rationalization that a real agent used — write for observed failures, not imagined ones.**

## When to Use
- You are writing a new skill and need guidance on structure, wording, and what to include
- A skill is failing to change agent behavior and you need to diagnose why
- You are deciding whether something should be a rigid discipline skill or a flexible technique guide
- **Never skip when:** you are about to add a rule that has not been tested — untested rules create false confidence

## Process

### Phase 1: Decide What Kind of Skill This Is
1. **Discipline skill** — enforces a rule agents routinely rationalize away (TDD order, verification before completion). These need explicit counters for every known excuse, a red flags list, and an iron-law statement. Wording must be unambiguous and leave no loopholes.
2. **Technique skill** — teaches an approach agents would apply correctly if they knew it existed (condition-based waiting, property testing). These need clear steps, a worked example, and guidance on when not to apply the technique.
3. **Reference skill** — documents an API, tool, or syntax agents cannot be expected to know from training. These need a quick-reference table and retrieval-friendly keywords.

The distinction matters: discipline skills need pressure-resistance; technique and reference skills need clarity and discoverability.

### Phase 2: Write the Description First
1. Start with "Use when..." followed by triggering conditions — symptoms, situations, contexts
2. Never summarize the skill's workflow or steps in the description
3. Keep it under 500 characters if possible; write in third person
4. Test the description mentally: would an agent load this skill for the right tasks and skip it for the wrong ones?

### Phase 3: Write the Skill Body
1. Open with the iron law (discipline skills) or a one-sentence core principle (technique/reference skills)
2. List when to use and when not to use — both matter
3. For discipline skills: add a rationalization table and a red flags list
4. For technique skills: add a worked example in Python and the primary language of the codebase

**Python example (technique skill):**

```python
import pytest
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_sort_is_idempotent(xs: list[int]) -> None:
    assert sorted(sorted(xs)) == sorted(xs)
```

**TypeScript example (technique skill):**

```typescript
import fc from "fast-check";

test("sort is idempotent", () => {
  fc.assert(
    fc.property(fc.array(fc.integer()), (xs) => {
      expect([...xs].sort().sort()).toEqual([...xs].sort());
    })
  );
});
```

5. Close with a Quick Reference table (Phase / Core Action / Done When)

### Phase 4: Apply the Checklist
1. Name uses only letters, numbers, and hyphens
2. YAML frontmatter has `name` and `description` — total under 1024 characters
3. Description is trigger-only; no workflow summary
4. Rationalization table has at most 8 rows
5. No flowcharts — numbered lists only
6. Every rule has a Python example if the source was TypeScript-only
7. No sentences carried over verbatim from any source material

## Red Flags — Stop Immediately
- The description explains what the skill does step-by-step rather than when to invoke it
- A rule exists that was not observed to be violated in any baseline run
- The rationalization table has more than 8 rows — merge duplicates
- There is a flowchart or dot diagram — replace with a numbered list
- The skill applies the same level of rigidity to a reference guide as to a discipline enforcer

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "This rule is obvious, no need to state it explicitly" | Obvious to the author means nothing to an agent under pressure — state it |
| "I'll add the Python example later" | Later means never; missing examples make skills TypeScript-only by default |
| "The description should explain what the skill does" | Descriptions that summarize workflows get used instead of the skill body |
| "More rows in the rationalization table means more thorough" | More rows dilutes focus; eight tight rows beat twenty loose ones |
| "A flowchart is clearer than a list here" | Flowcharts render inconsistently and cannot be copied as instructions — use lists |
| "Discipline and technique skills need the same structure" | They do not; conflating them weakens both — rigid counters in a reference guide are noise |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Classify | Identify skill type: discipline, technique, or reference | Type chosen; appropriate structure selected |
| 2. Description | Write trigger-only "Use when..." under 500 chars | No workflow steps appear in the description |
| 3. Body | Iron law or principle, steps, examples, rationalization table | All sections present; Python example included |
| 4. Checklist | Verify name, frontmatter, table limits, no flowcharts | Every checklist item passes |
