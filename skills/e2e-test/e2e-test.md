---
name: e2e-test
description: Use when writing or fixing browser-based end-to-end tests — covers Playwright setup, selector strategy, assertion patterns, and parallel execution in both JavaScript and Python.
---

# Playwright Test

## The Law
**Every browser test must target stable, semantic selectors and assert on observable user outcomes — not implementation details — so that refactors don't silently break the test suite.**

## When to Use
- Adding end-to-end coverage for a new user flow
- A brittle test keeps failing after UI refactors that didn't change behaviour
- Setting up Playwright in a project for the first time
- **Never skip when:** a user-facing flow involves auth, checkout, form submission, or multi-step wizards — these must have at least one passing Playwright spec before the feature ships

## Process

### Phase 1: Setup
Goal — get Playwright installed and a browser context running.

JavaScript / TypeScript:
```bash
npm init playwright@latest
# Choose: TypeScript, tests/ directory, GitHub Actions CI
npx playwright install --with-deps chromium
```

Python (pytest-playwright):
```bash
pip install pytest-playwright
playwright install --with-deps chromium
```

Minimal config — `playwright.config.ts`:
```typescript
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests",
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry",
  },
  projects: [{ name: "chromium", use: { ...devices["Desktop Chrome"] } }],
});
```

Python `conftest.py`:
```python
import pytest

@pytest.fixture(scope="session")
def base_url():
    return "http://localhost:3000"
```

### Phase 2: Selector Strategy
Goal — pick selectors that survive UI refactors.

Priority order (use the first one that applies):

1. `data-testid` — explicit, refactor-proof, your first choice
2. ARIA role + accessible name — `getByRole("button", { name: "Submit" })`
3. Label text — `getByLabel("Email address")`
4. Placeholder — `getByPlaceholder("search...")`
5. Visible text — `getByText("Confirm")` — only for stable copy
6. CSS class or XPath — last resort, document why

Add `data-testid` to markup when no stable semantic target exists:
```html
<button data-testid="checkout-submit">Place order</button>
```

TypeScript — preferred locator patterns:
```typescript
// Role-based (most readable)
await page.getByRole("button", { name: "Place order" }).click();

// data-testid (most stable)
await page.getByTestId("checkout-submit").click();

// Chained scope — narrow before interacting
const form = page.getByTestId("shipping-form");
await form.getByLabel("Postcode").fill("SW1A 1AA");
```

Python — same patterns:
```python
# Role-based
await page.get_by_role("button", name="Place order").click()

# data-testid
await page.get_by_test_id("checkout-submit").click()

# Chained scope
form = page.get_by_test_id("shipping-form")
await form.get_by_label("Postcode").fill("SW1A 1AA")
```

### Phase 3: Test Structure
Goal — write tests that read like user stories.

TypeScript — full spec:
```typescript
import { test, expect } from "@playwright/test";

test.describe("checkout flow", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/cart");
  });

  test("completes purchase with valid card", async ({ page }) => {
    await page.getByTestId("checkout-submit").click();
    await page.getByLabel("Card number").fill("4242424242424242");
    await page.getByLabel("Expiry").fill("12/26");
    await page.getByLabel("CVC").fill("123");
    await page.getByRole("button", { name: "Pay now" }).click();
    await expect(page.getByTestId("order-confirmation")).toBeVisible();
  });

  test("shows error on declined card", async ({ page }) => {
    await page.getByTestId("checkout-submit").click();
    await page.getByLabel("Card number").fill("4000000000000002");
    await page.getByRole("button", { name: "Pay now" }).click();
    await expect(page.getByTestId("payment-error")).toContainText("declined");
  });
});
```

Python pytest-playwright — same test:
```python
import pytest
from playwright.sync_api import Page, expect

class TestCheckoutFlow:
    def test_completes_purchase_with_valid_card(self, page: Page):
        page.goto("/cart")
        page.get_by_test_id("checkout-submit").click()
        page.get_by_label("Card number").fill("4242424242424242")
        page.get_by_label("Expiry").fill("12/26")
        page.get_by_label("CVC").fill("123")
        page.get_by_role("button", name="Pay now").click()
        expect(page.get_by_test_id("order-confirmation")).to_be_visible()

    def test_shows_error_on_declined_card(self, page: Page):
        page.goto("/cart")
        page.get_by_test_id("checkout-submit").click()
        page.get_by_label("Card number").fill("4000000000000002")
        page.get_by_role("button", name="Pay now").click()
        expect(page.get_by_test_id("payment-error")).to_contain_text("declined")
```

### Phase 4: Assertion Patterns
Goal — assert on what the user sees, not on what the DOM contains.

Prefer web-first assertions — they auto-wait:
```typescript
// Correct — waits for element to appear
await expect(page.getByTestId("success-banner")).toBeVisible();
await expect(page.getByRole("heading")).toHaveText("Order confirmed");
await expect(page.getByTestId("item-count")).toHaveText("3 items");

// Wrong — brittle; doesn't auto-wait
const text = await page.textContent(".banner");
expect(text).toBe("Order confirmed");
```

Python equivalents:
```python
# Correct
expect(page.get_by_test_id("success-banner")).to_be_visible()
expect(page.get_by_role("heading")).to_have_text("Order confirmed")
expect(page.get_by_test_id("item-count")).to_have_text("3 items")
```

Never use fixed `sleep` or `waitForTimeout` — always wait on a condition:
```typescript
// Wrong
await page.waitForTimeout(2000);

// Correct
await page.waitForSelector("[data-testid='results-loaded']");
// or simply assert — Playwright's expect retries automatically
await expect(page.getByTestId("results-loaded")).toBeVisible();
```

### Phase 5: CI Integration
Goal — run tests in parallel without flakiness.

GitHub Actions:
```yaml
- name: Run Playwright tests
  run: npx playwright test
  env:
    CI: true

- uses: actions/upload-artifact@v4
  if: failure()
  with:
    name: playwright-report
    path: playwright-report/
```

Python CI:
```bash
pytest tests/e2e/ --tracing=retain-on-failure -n auto
```

## Red Flags — Stop Immediately
- Selectors that include generated class names (`.css-1a2b3c`) or array indexes
- `waitForTimeout` / `time.sleep` anywhere in test code
- Assertions on internal state (Redux store, component props) rather than visible UI
- Tests that pass only when run in isolation but fail in the full suite
- Hardcoded absolute URLs that break in CI environments

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "XPath is faster to write" | XPath breaks on any structural change; `data-testid` doesn't |
| "I'll add retries to fix flakiness" | Retries hide root causes; fix the wait strategy instead |
| "The test is too slow, I'll skip mobile" | Mobile viewports catch real layout bugs desktop misses |
| "I'll use `page.evaluate()` to check state" | Testing internal state doesn't verify the user experience |
| "This test only needs to run locally" | Tests not in CI are not real tests — they decay immediately |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| Setup | Install Playwright, configure baseURL | `npx playwright test` runs without errors |
| Selector strategy | Use data-testid / ARIA role / label | No CSS class or XPath selectors in specs |
| Test structure | beforeEach navigates, test asserts outcome | Each test is independent and readable |
| Assertions | Web-first `expect` calls only | No `textContent()` + manual assert patterns |
| CI | Parallel run with artifact upload on failure | Tests run on every PR without manual steps |
