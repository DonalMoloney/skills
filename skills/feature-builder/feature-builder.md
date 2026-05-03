---
name: feature-builder
description: Use when implementing a new feature end-to-end — triggers the Explorer → Architect → Builder → Reviewer workflow to prevent scope creep, missing edge cases, and untested code.
---

# Build a Feature

## The Law
**Understand before you touch code, plan before you write, and verify before you ship — skipping any phase produces features that technically work but break something else.**

## When to Use
- A new capability needs to be added to an existing codebase
- A user story or ticket is ready to implement
- A feature branch needs to be started from scratch
- **Never skip when:** the feature touches auth, payments, data persistence, or cross-service boundaries — these require all four phases without shortcutting

## Process

### Phase 1: Explorer
Goal — understand the terrain before committing to anything.

1. Read the relevant code paths end-to-end
   - Find the entry point (route, event handler, CLI command)
   - Trace the data flow through to the persistence layer
   - Identify all callers of the code you will change
2. Locate existing tests that cover adjacent behaviour
3. Check for abstractions you should extend rather than duplicate
4. Write a one-paragraph summary: what exists, what changes, what stays the same
5. **Gate:** confirm the summary with the user before continuing

### Phase 2: Architect
Goal — produce a plan that a reviewer can approve without seeing code.

1. List every file that will change and why
2. Define the public interface of the new feature (function signatures, API shape, event schema)
3. Identify edge cases: empty inputs, concurrent writes, auth failures, downstream unavailability
4. Choose the test strategy — unit, integration, or end-to-end — and name the specific cases
5. Note the rollback path if the feature ships broken

TypeScript — interface defined before implementation:
```typescript
// Defined in Phase 2, implemented in Phase 3
interface SubscriptionUpgradeService {
  upgrade(userId: string, targetPlan: PlanId): Promise<UpgradeResult>;
  preview(userId: string, targetPlan: PlanId): Promise<PricingPreview>;
}
```

Python — equivalent contract sketch:
```python
from abc import ABC, abstractmethod

class SubscriptionUpgradeService(ABC):
    @abstractmethod
    def upgrade(self, user_id: str, target_plan: str) -> UpgradeResult: ...

    @abstractmethod
    def preview(self, user_id: str, target_plan: str) -> PricingPreview: ...
```

6. **Gate:** present the plan and get explicit approval before writing any production code

### Phase 3: Builder
Goal — implement exactly what the plan describes, no more.

1. Write the failing test first (one test per edge case from Phase 2)
2. Implement the minimum code to make each test pass
3. Refactor once green — no behaviour changes during refactor
4. Run the full test suite; fix regressions before adding more code
5. Verify the feature manually against the original requirement

TypeScript — minimal passing implementation:
```typescript
export async function upgrade(
  userId: string,
  targetPlan: PlanId,
): Promise<UpgradeResult> {
  const user = await userRepo.findOrThrow(userId);
  const current = await planRepo.findByUser(userId);
  if (current.id === targetPlan) {
    throw new ConflictError("already on this plan");
  }
  return billingGateway.changeSubscription(user, targetPlan);
}
```

Python — matching implementation:
```python
def upgrade(self, user_id: str, target_plan: str) -> UpgradeResult:
    user = self.user_repo.find_or_raise(user_id)
    current = self.plan_repo.find_by_user(user_id)
    if current.id == target_plan:
        raise ConflictError("already on this plan")
    return self.billing_gateway.change_subscription(user, target_plan)
```

### Phase 4: Reviewer
Goal — validate the feature as if you didn't write it.

1. Re-read every changed file top-to-bottom
2. Confirm every edge case from Phase 2 has a corresponding test
3. Verify no unplanned files were modified
4. Check that error messages are safe for external consumers — no stack traces in API responses
5. Run linting and type checking to completion
6. Confirm the feature can be disabled or rolled back without a deploy if it misbehaves

## Red Flags — Stop Immediately
- Skipping Explorer because "it's a small change" — small changes in the wrong place still break things
- Writing code before the Phase 2 gate is approved
- Adding tests after the implementation is complete rather than driving with them
- Expanding scope mid-implementation without returning to Phase 2
- Merging with a failing test marked `skip` or `xfail`
- Shipping without verifying the rollback path

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "I know this codebase, I don't need to explore" | Confidence peaks just before the unexpected edge case appears |
| "The plan is obvious, I'll just code it" | Obvious plans produce the most surprising regressions |
| "Tests can be added after — this is urgent" | Post-hoc tests verify what was built, not what was required |
| "The scope is small, one phase is enough" | Small features in complex systems have non-local effects |
| "I'll refactor and add features at the same time" | Mixed changes make failures impossible to attribute |
| "It works in my manual test, ship it" | Manual testing has no memory; the automated suite does |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| Explorer | Read code, trace data flow, summarise | User confirms the summary |
| Architect | Define interface, list edge cases, name tests | Plan explicitly approved |
| Builder | Test-first, implement, green suite | All tests pass, no regressions |
| Reviewer | Re-read, verify coverage, confirm rollback | Every Phase 2 case has a test |
