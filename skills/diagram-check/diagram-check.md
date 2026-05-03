---
name: diagram-check
description: Validate Mermaid diagram syntax before rendering — use before including any Mermaid block in a document, PR, or design doc.
---

# Diagram Check

## The Law
**Never embed a Mermaid block in any document without validating it first — a broken diagram fails silently in some tools and corrupts output in others.**

## When to Use
- Any time a Mermaid block has been written or edited
- Before committing a markdown file that contains a Mermaid block
- After a diagram has been updated to reflect code changes
- When a diagram was copied from another source and its origin is unknown
- **Never skip when:** the diagram will be rendered in a tool you cannot preview locally (GitHub, Confluence, Notion) — silent failures in those tools are invisible until someone reports broken docs

## Process

### Phase 1: Structural Pre-Check
1. Confirm the block opens with a supported type keyword on its own line — no indentation, no inline text:
   ```
   flowchart TD
   sequenceDiagram
   erDiagram
   classDiagram
   stateDiagram-v2
   gantt
   graph LR
   pie
   mindmap
   timeline
   ```
2. Verify the code block is wrapped in triple backticks tagged `mermaid` in the markdown file:
   ````
   ```mermaid
   ...
   ```
   ````
3. Check that the block is not empty and contains at least one node or participant declaration.

### Phase 2: Syntax Rule Checks by Type

**flowchart / graph**
1. Every node ID uses only alphanumeric characters and underscores — no spaces, no hyphens in IDs.
2. Every node has a closing bracket that matches its opening: `[` → `]`, `(` → `)`, `{` → `}`, `([` → `])`.
3. Every edge is one of: `-->`, `---`, `--text-->`, `-.->`, `==>`, `<-->` — no invented operators.
4. Subgraph blocks opened with `subgraph` are closed with `end`.

**sequenceDiagram**
1. Every actor used in a message has been declared with `participant` or `actor` before its first use, or is the first actor mentioned.
2. Message arrows are one of: `->>`, `-->>`, `->`, `-->`, `-x`, `--x`.
3. `activate`/`deactivate` blocks are balanced — one `deactivate` per `activate`.
4. `loop`, `alt`, `opt`, `par` blocks all end with `end`.

**erDiagram**
1. Entity names contain no spaces — use underscores or PascalCase.
2. Relationship lines follow the form: `ENTITY_A ||--o{ ENTITY_B : "label"` — the label is always present and quoted.
3. Cardinality symbols used are only: `|o`, `||`, `}o`, `}|` on each side.

**classDiagram**
1. Class body uses `+`, `-`, `#`, `~` visibility markers before member names.
2. Relationships use one of: `<|--`, `*--`, `o--`, `-->`, `--`, `..>`, `..|>`.
3. Notes added with `note for ClassName` are on their own line.

**stateDiagram-v2**
1. Every state machine has `[*] -->` as its first transition.
2. Composite states opened with `state "Label" as ID {` are closed with `}`.
3. Fork/join use `state ID <<fork>>` and `state ID <<join>>` — not ad-hoc shapes.

**gantt**
1. `dateFormat` is declared before any `section` block.
2. Every task has a duration or explicit end date — no open-ended tasks.
3. Task IDs are unique within the chart.

### Phase 3: Label Safety Check
1. Flag any label containing special characters that Mermaid misparses: unescaped `"`, `(`, `)` inside square-bracket labels.
   - Fix: wrap the label text in quotes inside the bracket: `A["label (with parens)"]`
2. Flag labels longer than 40 characters — they overflow on most renderers.
3. Check that no two different node IDs share the same display label — this causes ambiguous diagrams.

### Phase 4: Render Verification
1. If the CLI tool `mmdc` (Mermaid CLI) is available, run:
   ```bash
   mmdc -i diagram.mmd -o /tmp/diagram-check.png
   ```
   A zero exit code means the diagram parsed successfully.
2. If `mmdc` is not available, use the Mermaid Live Editor at `https://mermaid.live` by pasting the source block.
3. Inspect the rendered output — confirm the layout matches the intent (direction, grouping, node count).
4. Do not accept "it parsed without error" as a substitute for visually checking the output.

## Red Flags — Stop Immediately
- The block opens with `graph` but has no direction keyword (`TD`, `LR`, `RL`, `BT`)
- A `subgraph`, `loop`, `alt`, `opt`, or composite state block has no closing `end`
- An ER relationship is missing its quoted label — the renderer will reject it
- Node IDs contain spaces — they will silently break edge resolution
- The gantt chart has no `dateFormat` line — it will render with wrong date handling
- You are about to skip Phase 4 because "the syntax looks right"

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "GitHub will show me if it's broken" | GitHub silently shows a blank box for broken Mermaid — no error message |
| "I only changed one label" | Label changes introduce bracket mismatches and special-char bugs more than structural edits |
| "The diagram parsed before, same structure" | Mermaid version differences between tools cause diagrams that parsed locally to fail in Confluence |
| "I'll fix it if someone reports it" | Broken docs erode trust; readers assume the content is also unreliable |
| "Validation takes too long" | Phase 1–3 takes under two minutes for any diagram under 30 nodes |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Structural | Confirm type keyword, fenced block tags, non-empty content | Block opens correctly, not empty |
| 2. Syntax Rules | Apply type-specific rule checklist | Zero rule violations for the diagram type |
| 3. Label Safety | Check special chars and label length | No long labels, no unescaped chars |
| 4. Render | Run mmdc or Mermaid Live; inspect output | Zero parse errors, layout matches intent |
