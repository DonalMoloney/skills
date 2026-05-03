---
name: bdd-test
description: Use Behavior-Driven Development (BDD) for acceptance tests — write Given/When/Then scenarios in Gherkin, implement step definitions, verify user-facing behaviour.
---

# BDD Test

## The Law
**BDD tests describe behaviour in user language, not implementation language. A test written as "Given [context], When [action], Then [outcome]" can be understood by non-engineers.**

## When to Use
- Writing acceptance tests for features
- Documenting expected user behaviour
- Bridging requirements and tests
- **Never skip when:** stakeholders need to verify requirements are met

## Process

### Phase 1: Write Scenarios in Gherkin
1. Use Given/When/Then format:
   ```gherkin
   Feature: User Login
   
   Scenario: User logs in with valid credentials
     Given I am on the login page
     When I enter my email and password
     And I click the login button
     Then I should see the dashboard
     And I should be logged in
   ```
2. Keep scenarios simple and focused
3. Each scenario is one user journey

### Phase 2: Implement Step Definitions
1. **Python (behave):**
   ```python
   from behave import given, when, then
   
   @given('I am on the login page')
   def step_visit_login(context):
       context.browser.get('/login')
   
   @when('I enter my email and password')
   def step_enter_credentials(context):
       # Implementation
   ```

2. **JavaScript (cucumber.js):**
   ```javascript
   Given('I am on the login page', async function() {
       await this.goto('/login');
   });
   ```

### Phase 3: Run Scenarios
1. **Python:**
   ```bash
   behave
   ```

2. **JavaScript:**
   ```bash
   npm test -- --profile=bdd
   ```

### Phase 4: Verify User Behaviour
1. Run scenarios and watch them execute
2. Each step should be observable (not implementation details)
3. Scenarios should pass for correct behaviour, fail for incorrect

### Phase 5: Document Acceptance
1. Scenarios become living documentation
2. Keep scenarios up-to-date as behaviour changes
3. Scenarios define what "done" means

## Red Flags — Stop Immediately

- Scenario mentions implementation details (mocks, database, API) — make it user-focused
- Scenario has more than 5-6 steps — split it
- You can't implement a step definition — the scenario is too vague
- Scenarios are passing but the actual feature doesn't work — steps aren't testing real behaviour

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "BDD is too slow" | BDD is the speed. Writing tests after code is slower. |
| "We don't have time for scenario writing" | Scenarios are requirements. Writing them forces clarity upfront. |
| "Unit tests are enough" | Unit tests verify code works. BDD tests verify the feature works for the user. |
| "Business stakeholders won't read the scenarios" | When written well, they will. That's the point. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Gherkin** | Write scenarios in Given/When/Then format | Scenarios are clear and focused |
| **Steps** | Implement step definitions for each phrase | Each step is defined and executable |
| **Run** | Execute scenarios; watch them pass/fail | Scenarios run and show behaviour |
| **Verify** | Confirm scenarios test real user behaviour | Behaviour is observable and correct |
| **Document** | Scenarios become acceptance criteria | Feature is defined by scenarios |
