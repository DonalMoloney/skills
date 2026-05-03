---
name: ui-patterns
description: Use when building frontend components, pages, or UI systems — guides design direction, component patterns, and accessibility requirements for production-grade interfaces.
---

# UI Patterns

## The Law
**Every interface must commit to a clear aesthetic direction and meet WCAG 2.1 AA accessibility standards — a component that looks good but excludes users is not finished.**

## When to Use
- Building a new component, page, or complete UI from scratch
- Reviewing an existing interface for design consistency or accessibility gaps
- Establishing a component pattern to be reused across a product
- **Never skip when:** the interface will be used by the public, is part of an authenticated product, or handles form input of any kind — accessibility is mandatory, not optional

## Process

### Phase 1: Design Direction
Goal — commit to an intentional aesthetic before touching markup.

1. Identify the context: who uses this, what action do they take, what emotion should the interface evoke
2. Choose a deliberate tone — pick one and execute it precisely:
   - Refined minimalism: generous whitespace, restrained palette, single display font
   - Editorial density: layered type scales, strong grid, high contrast
   - Utilitarian: data-first layout, monospace accents, zero decoration
   - Expressive: unexpected colour, motion-driven hierarchy, bespoke illustration
3. Select typography intentionally — pair a distinctive display face with a legible body face; avoid defaults like Inter, Roboto, and Arial
4. Define a colour system using CSS custom properties — dominant colour, accent, surface, and semantic tokens (error, success, warning)
5. Decide on spacing scale (e.g. 4px base unit) before writing a single margin

CSS custom properties foundation:
```css
:root {
  --colour-surface: #0f0f0f;
  --colour-text: #f2f0eb;
  --colour-accent: #c8ff00;
  --colour-error: #ff4d4d;
  --font-display: "Bebas Neue", sans-serif;
  --font-body: "Lora", serif;
  --space-unit: 4px;
}
```

### Phase 2: Component Patterns
Goal — build reusable, self-contained components.

#### Button
```tsx
type ButtonProps = {
  intent: "primary" | "secondary" | "danger";
  loading?: boolean;
  children: React.ReactNode;
} & React.ButtonHTMLAttributes<HTMLButtonElement>;

export function Button({ intent, loading, children, ...props }: ButtonProps) {
  return (
    <button
      data-intent={intent}
      aria-busy={loading}
      disabled={loading || props.disabled}
      {...props}
    >
      {loading ? <span aria-hidden>...</span> : children}
    </button>
  );
}
```

Python (Jinja2 server-rendered equivalent):
```html
{# button.html #}
<button
  class="btn btn--{{ intent }}"
  {% if loading %}aria-busy="true" disabled{% endif %}
  {% if disabled %}disabled{% endif %}
>
  {% if loading %}<span aria-hidden="true">...</span>{% else %}{{ label }}{% endif %}
</button>
```

#### Form Field with Error State
```tsx
type FieldProps = {
  id: string;
  label: string;
  error?: string;
  children: React.ReactNode;
};

export function Field({ id, label, error, children }: FieldProps) {
  const errorId = `${id}-error`;
  return (
    <div data-field={id}>
      <label htmlFor={id}>{label}</label>
      {React.cloneElement(children as React.ReactElement, {
        id,
        "aria-describedby": error ? errorId : undefined,
        "aria-invalid": error ? true : undefined,
      })}
      {error && (
        <p id={errorId} role="alert">
          {error}
        </p>
      )}
    </div>
  );
}
```

#### Loading State Skeleton
```tsx
export function Skeleton({ width, height }: { width: string; height: string }) {
  return (
    <div
      role="status"
      aria-label="Loading"
      style={{ width, height }}
      className="skeleton"
    />
  );
}
```

CSS:
```css
.skeleton {
  background: linear-gradient(
    90deg,
    var(--colour-surface-muted) 25%,
    var(--colour-surface-subtle) 50%,
    var(--colour-surface-muted) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.4s infinite;
  border-radius: 4px;
}

@keyframes shimmer {
  from { background-position: 200% 0; }
  to   { background-position: -200% 0; }
}
```

#### Motion — high-impact moments only
```css
/* Page load: staggered reveal */
.card {
  opacity: 0;
  transform: translateY(16px);
  animation: slide-up 0.4s ease forwards;
}

.card:nth-child(1) { animation-delay: 0ms; }
.card:nth-child(2) { animation-delay: 80ms; }
.card:nth-child(3) { animation-delay: 160ms; }

@keyframes slide-up {
  to { opacity: 1; transform: none; }
}

/* Respect user preference */
@media (prefers-reduced-motion: reduce) {
  .card { animation: none; opacity: 1; transform: none; }
}
```

### Phase 3: Accessibility Checklist (WCAG 2.1 AA)
Goal — ship nothing that excludes users.

**Perceivable**
1. All images have descriptive `alt` text; decorative images use `alt=""`
2. Colour contrast meets 4.5:1 for body text, 3:1 for large text and UI components
3. Information is not conveyed by colour alone (use icons, labels, or patterns alongside)
4. Content reflows at 320px viewport width without horizontal scroll

**Operable**
5. Every interactive element is reachable and usable by keyboard alone
6. Focus order matches the visual reading order
7. Focus indicators are visible — never remove `:focus-visible` without a replacement
8. No keyboard trap: pressing Escape or Tab can always leave a modal or menu
9. Moving content (carousels, auto-play) can be paused or stopped

**Understandable**
10. Labels are attached to every form field — no placeholder-only inputs
11. Errors identify the field, explain what is wrong, and suggest how to fix it
12. `lang` attribute is set on `<html>` and on any inline foreign-language text

**Robust**
13. Interactive components use appropriate ARIA roles (`button`, `dialog`, `listbox`, etc.)
14. ARIA `aria-expanded`, `aria-selected`, and `aria-checked` states update on interaction
15. Test with at least one screen reader (VoiceOver on macOS / NVDA on Windows)

Quick contrast check during development:
```bash
# Install axe-core CLI
npm install -g @axe-core/cli
axe http://localhost:3000 --tags wcag2a,wcag2aa
```

Python (pytest + axe-playwright):
```python
from axe_playwright_python.sync_playwright import Axe

def test_homepage_accessibility(page):
    page.goto("/")
    results = Axe().run(page)
    violations = [v for v in results.violations if v["impact"] in ("critical", "serious")]
    assert violations == [], f"Accessibility violations: {violations}"
```

### Phase 4: Review
Goal — confirm the interface is coherent, complete, and accessible before handoff.

1. Open the component in isolation (Storybook, standalone HTML) — does it look intentional at every breakpoint?
2. Tab through every interactive element — is the focus order logical?
3. Disable CSS — does the page still convey its structure through semantic markup alone?
4. Run the axe scan — zero critical or serious violations
5. Check that every animation respects `prefers-reduced-motion`
6. Verify the colour palette passes contrast ratios for all text sizes used

## Red Flags — Stop Immediately
- Placeholder text used as the only label for an input field
- `cursor: pointer` on a `<div>` with a click handler — use `<button>` instead
- Animation with no `prefers-reduced-motion` override
- Colour contrast below 4.5:1 on body text
- `aria-label` overriding visible text — the two must match or the label adds no value
- Inline `style` for layout decisions that belong in the design token system

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "Accessibility is a nice-to-have" | WCAG 2.1 AA is a legal requirement in most jurisdictions |
| "Screen reader users are a tiny minority" | Accessibility improvements benefit all users, including keyboard power users |
| "The designer didn't spec focus states" | Every interactive element needs a visible focus state — spec or not |
| "We'll fix contrast in the polish pass" | Contrast is a design constraint, not a cosmetic tweak |
| "Inter looks fine here" | Fine is not memorable; choose fonts that serve the context specifically |
| "We'll add ARIA later" | ARIA retrofitted onto non-semantic markup is harder than building semantic from the start |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| Design Direction | Choose tone, fonts, colour tokens | CSS custom properties defined before markup |
| Component Patterns | Build with semantic HTML and ARIA | Components work keyboard-only |
| Accessibility Checklist | Run axe, check contrast, test focus | Zero critical/serious violations |
| Review | Breakpoints, motion prefs, semantic structure | Passes every checklist item |
