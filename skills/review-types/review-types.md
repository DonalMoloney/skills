---
name: review-types
description: Invoke when introducing a new type, before opening a PR that adds data models, or when refactoring types to strengthen their guarantees.
---

# Review Types

## The Law
**A type must make illegal states unrepresentable — if invalid data can be constructed without bypassing the type's own interface, the type is not doing its job.**

## When to Use
- A new domain model, data class, or interface is being introduced
- A PR adds or modifies type definitions across multiple files
- Existing types are being refactored and their invariants are in question
- **Never skip when:** the type carries business rules (e.g. a monetary amount, a validated email, a bounded range) — these invariants must be enforced by the type itself, not by its callers

## Process

### Phase 1: Identify All Invariants
Before scoring anything, enumerate every constraint the type is supposed to enforce:

1. Data consistency requirements — fields that must be in sync with each other
2. Value range or format constraints — a percentage must be 0–100, an email must contain `@`
3. Valid state transitions — which combinations of field values are legal?
4. Relational constraints — field A is only meaningful when field B has a certain value
5. Business rules encoded in the type — a `Price` is never negative, a `UserId` is never empty

Write down the invariant list explicitly. A type that has no invariants is a data bag — score it accordingly.

### Phase 2: Score Encapsulation (1–10)
Ask whether the type's internals are protected from external interference:

1. Can a caller set fields to values that violate an invariant without going through a validated constructor or setter?
2. Are internal fields exposed as mutable public attributes when they should not be?
3. Is the interface minimal — does it expose exactly what callers need and nothing more?

Python examples:
```python
# WEAK encapsulation — caller can set negative price directly
@dataclass
class Price:
    amount: float   # nothing stops Price(amount=-5.0)

# STRONG encapsulation — invariant enforced at construction
from pydantic import BaseModel, field_validator

class Price(BaseModel):
    amount: float

    @field_validator("amount")
    @classmethod
    def must_be_non_negative(cls, v: float) -> float:
        if v < 0:
            raise ValueError(f"Price cannot be negative, got {v}")
        return v

    model_config = {"frozen": True}   # prevents post-construction mutation
```

TypeScript contrast:
```typescript
// WEAK — nothing prevents invalid construction
interface Price {
  amount: number;
}

// STRONGER — private constructor forces the factory path
class Price {
  private constructor(private readonly amount: number) {}

  static of(amount: number): Price {
    if (amount < 0) throw new Error(`Price cannot be negative: ${amount}`);
    return new Price(amount);
  }
}
```

### Phase 3: Score Invariant Expression (1–10)
Ask whether the invariants are visible from the type definition alone:

1. Would a developer reading the type for the first time understand its constraints without reading the rest of the codebase?
2. Are constraints enforced at compile time (type-level) rather than only at runtime?
3. Does the type use specific subtypes or branded types to distinguish semantically different values of the same primitive?

```python
# WEAK expression — both IDs are bare strings; easy to swap them by mistake
user_id: str
org_id: str

# STRONG expression — types are distinct; the compiler prevents swapping
from typing import NewType

UserId = NewType("UserId", str)
OrgId = NewType("OrgId", str)

user_id: UserId
org_id: OrgId
```

### Phase 4: Score Invariant Usefulness (1–10)
Ask whether the invariants prevent real bugs in this codebase:

1. Do the invariants reflect actual business requirements, or are they theoretical constraints no caller would ever violate?
2. Are the invariants neither too tight (rejecting valid inputs) nor too loose (accepting invalid ones)?
3. Do the invariants make the code easier to reason about for the next engineer?

### Phase 5: Score Invariant Enforcement (1–10)
Ask whether the invariants actually hold at runtime:

1. Are invariants checked at construction — not deferred to a validate() method a caller might forget to call?
2. Are all mutation points guarded — including setters, `__setattr__`, and any method that changes state?
3. Is it impossible to create an instance in an invalid state through any normal code path?

```python
# DEFECT: validation is opt-in, callers will forget
@dataclass
class EmailAddress:
    value: str

    def validate(self) -> None:
        if "@" not in self.value:
            raise ValueError("Not a valid email")

# FIX: validation is mandatory at construction
from pydantic import BaseModel, EmailStr

class EmailAddress(BaseModel):
    value: EmailStr   # Pydantic validates format automatically on instantiation
```

### Phase 6: Flag Common Anti-Patterns
Check explicitly for each of these:

1. **Anemic model** — type has fields but no behaviour; all logic lives in external service classes
2. **Stringly-typed API** — `status: str` instead of `status: Literal["active", "suspended", "deleted"]`
3. **Any-typed field** — `data: Any` or `config: object` with no narrowing
4. **Invariant by convention** — constraint documented in a comment rather than enforced by the type
5. **Inconsistent enforcement** — some mutation methods validate, others do not
6. **God type** — a single type that carries responsibilities belonging to three separate concepts
7. **Missing construction boundary** — no validated constructor; callers assemble the type field by field

### Phase 7: Deliver the Assessment
For each type under review, report in this structure:

**Type: `ClassName`**

1. Invariants identified — the list from Phase 1
2. Ratings — Encapsulation / Expression / Usefulness / Enforcement, each with a one-sentence justification
3. Strengths — what the type does well
4. Concerns — specific issues with file path and line references
5. Recommended improvements — concrete and proportionate; note when a simpler approach is better than a sophisticated one

## Red Flags — Stop Immediately
- A type accepts `Any` or `object` for a field that represents a domain concept
- A numeric field representing money, percentage, or count has no range validation
- A string field representing an identifier, email, or URL has no format constraint
- Invariants are documented only in comments, not enforced in code
- The type exposes a public mutable field that, if set to an arbitrary value, would violate the type's stated contract
- A `validate()` or `check()` method exists — callers will skip it

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "Callers know not to pass invalid values" | Callers change, new callers arrive, and knowledge is lost — the type must enforce the rule itself |
| "Runtime validation is enough, we don't need type-level constraints" | Runtime-only checks find the bug after the data is already corrupted; type-level checks prevent corruption |
| "Using `Any` is fine here, it's just an internal detail" | `Any` spreads — once one field is `Any`, the surrounding code loses type safety |
| "A validate() method is cleaner than construction-time checks" | Optional validation is skipped validation; construction-time is the only reliable moment |
| "Branded types are overkill for a project this size" | The cost of a type alias is one line; the cost of swapping two IDs in production is much higher |
| "The type does too many things because the domain is complex" | Complexity in the domain is a reason to split types, not a reason to accept a god type |
| "Perfect invariants are the enemy of shipping" | Enforce the invariants that prevent real bugs; skip theoretical ones — this is a judgement call, not an excuse to skip all of them |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Invariants | List every constraint the type must enforce | Invariant list written out explicitly |
| 2. Encapsulation | Can callers bypass the type's own interface? | Score 1–10 with justification |
| 3. Expression | Are invariants visible from the definition? | Score 1–10 with justification |
| 4. Usefulness | Do invariants prevent real bugs? | Score 1–10 with justification |
| 5. Enforcement | Are invariants checked at construction? | Score 1–10 with justification |
| 6. Anti-patterns | Anemic, stringly-typed, Any, no boundary, etc. | Each pattern explicitly checked |
| 7. Report | Invariants, ratings, strengths, concerns, fixes | Assessment delivered per type |
