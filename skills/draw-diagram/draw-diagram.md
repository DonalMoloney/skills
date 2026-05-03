---
name: draw-diagram
description: Generate a Mermaid diagram from a description, codebase, or concept — use when any flow, relationship, or structure needs to be visualised.
---

# Diagram It

## The Law
**Choose the diagram type that matches the structure of the information, not the one you already know how to write — wrong type, wrong diagram.**

## When to Use
- User asks to visualise a process, architecture, or data model
- A written explanation has become too dense and a picture would clarify it
- A PR description needs a flow or sequence diagram to explain the change
- Onboarding docs need a structural overview
- **Never skip when:** the audience is non-technical and will read the output without a code walkthrough

## Process

### Phase 1: Identify Diagram Type
1. Determine the nature of the thing to be drawn:
   - Steps that execute in order with decisions → **flowchart**
   - Messages exchanged between actors over time → **sequenceDiagram**
   - Tables, columns, and foreign-key relationships → **erDiagram**
   - Classes, methods, and inheritance → **classDiagram**
   - States an entity moves between on events → **stateDiagram-v2**
   - Task durations and dependencies → **gantt**
   - How components depend on each other at the package level → **graph LR** (dependency graph)
2. When two types could fit, pick the one that answers the most important question the reader has.
3. Never mix types in one diagram block — split into two separate diagrams instead.

### Phase 2: Draft the Mermaid Source
1. Open the block with the type keyword on its own line:
   ```
   flowchart TD
   sequenceDiagram
   erDiagram
   classDiagram
   stateDiagram-v2
   gantt
   graph LR
   ```
2. Apply the relevant syntax rules:
   - **flowchart / graph**: nodes in `A[Label]`, `A(Label)`, `A{Decision}` form; edges as `-->`, `--Label-->`, `-.->`, `==>`.
   - **sequenceDiagram**: use `participant`, `->>`/`-->>`; wrap async returns in `activate`/`deactivate` blocks.
   - **erDiagram**: entity names in ALL_CAPS; relationships as `||--o{`, `}o--||`, etc.; add quoted relationship labels.
   - **classDiagram**: `class Foo { +method() }`, inheritance as `Foo <|-- Bar`.
   - **stateDiagram-v2**: `[*] --> State`; label transitions with `: EventName`.
   - **gantt**: always set `dateFormat YYYY-MM-DD`; use `section` blocks; mark criticals with `crit`.
3. Keep labels short — 40 characters maximum; break long labels with `<br/>`.
4. Add a `%%` comment at the top naming the diagram purpose.

### Phase 3: Validate Before Output
1. Run the Mermaid source through the diagram-check skill before including it in any document.
2. Fix every error the validator reports — do not pass a known-broken diagram downstream.
3. Wrap the final block in a markdown fenced code block tagged `mermaid`.

### Phase 4: Deliver with Context
1. Place a one-sentence caption directly below the diagram block explaining what it shows.
2. If the diagram is for a design doc, also provide a plain-English paragraph summarising the same information — diagrams are not self-contained for all readers.

## Red Flags — Stop Immediately
- You are writing a flowchart for a set of database tables — that is an ER diagram
- Labels exceed 60 characters — the renderer will clip or overlap them
- A single diagram has more than 20 nodes — split it
- The diagram has no direction keyword (TD, LR, etc.) on a flowchart
- You are about to use `graph` without specifying direction

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "A flowchart works for everything" | Sequence diagrams show time order; ER diagrams show cardinality — the reader misses both when the wrong type is used |
| "I'll validate later" | Broken diagrams fail silently in some renderers and corrupt docs in others |
| "The label is long but the reader needs every word" | A long label is a sign the node itself is doing too much — split the diagram or shorten to a noun phrase |
| "I don't need a caption" | A diagram without a caption forces the reader to reverse-engineer what they are looking at |
| "I'll use one big diagram to show everything" | Large diagrams are unreadable; split by concern and cross-reference |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Identify Type | Match structure to diagram type from the lookup list | One type chosen, alternatives ruled out |
| 2. Draft Source | Write valid Mermaid syntax for that type | Block opens with correct keyword, labels under 40 chars |
| 3. Validate | Run diagram-check skill | Zero errors reported |
| 4. Deliver | Add caption and plain-English summary | Output is usable without a code walkthrough |
