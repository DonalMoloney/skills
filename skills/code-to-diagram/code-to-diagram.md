---
name: code-to-diagram
description: Use to extract system architecture from code — trace dependencies, identify layers, draw data flow, and document structure as a diagram.
---

# Code to Diagram

## The Law
**A diagram that doesn't match the code is worse than no diagram. Extract diagrams from code, not from memory. Update diagrams when code changes.**

## When to Use
- Documenting system architecture
- Onboarding new team members
- Reviewing a complex module
- Verifying layering and dependencies
- **Never skip when:** the codebase is large — diagrams help everyone understand it

## Process

### Phase 1: Identify the Scope
1. What system are you diagramming?
   - Single service/module?
   - Multi-service architecture?
   - Data flow through a feature?
2. Define boundaries:
   - What's included?
   - What's external?
   - What's in-scope for the diagram?

### Phase 2: Trace Dependencies
1. Read the code and identify:
   - Main modules/services
   - How they import/depend on each other
   - What data flows between them
2. Draw a dependency graph:
   - Boxes for components
   - Arrows for dependencies
   - Labels for data types

### Phase 3: Identify Layers
1. Group components by layer:
   - **API/Router layer** — entry points
   - **Business logic layer** — core domain
   - **Data access layer** — database, cache
   - **External services** — third-party APIs
2. Draw layers as horizontal bands

### Phase 4: Document Data Flow
1. For a feature, trace data:
   - User input → API → Logic → Database
   - Response → Serialization → API → Client
2. Show which components touch which data
3. Note any transformations

### Phase 5: Create Diagram
1. Use a diagramming tool:
   - **Mermaid** (text-based, version-controllable)
   - **Lucidchart** (visual, collaboration)
   - **Draw.io** (free, exportable)
2. Keep it simple:
   - Boxes for components
   - Arrows for relationships
   - Labels for clarity
3. Aim for one-page readability

### Phase 6: Verify Against Code
1. Does the diagram match the code?
   - Run a quick trace: pick a feature, follow the boxes
   - Does the flow make sense?
2. Is anything missing?
   - Error paths, fallbacks?
   - External services?
3. Update the diagram if mismatches found

## Red Flags — Stop Immediately

- Diagram doesn't match the code — update it before sharing
- Diagram is so complex it needs explanation — simplify or break into multiple diagrams
- Diagram is missing critical components — add them
- You can't trace a feature through the diagram — redraw it

## Common Rationalizations

| Excuse | Why It's Wrong |
|--------|---|
| "The code IS the diagram" | True, but no one reads all the code to understand structure. Diagrams are faster. |
| "Diagrams are hard to keep up to date" | Diagrams that matter are worth keeping up to date. Make them part of the definition. |
| "We'll diagram this after release" | You won't. Diagram while the code is fresh. |
| "This is too complex to diagram" | Then it's too complex, period. Simplify the design. |

## Quick Reference

| Phase | Core Action | Done When |
|-------|---|---|
| **Scope** | Define what system and what boundaries | Scope is explicit |
| **Trace** | Read code; identify dependencies | Dependencies are clear |
| **Layers** | Group by API/Logic/Data/External | Layers are identified |
| **Flow** | Trace user request through components | Data flow is documented |
| **Diagram** | Draw boxes, arrows, labels; keep simple | Diagram is one-page readable |
| **Verify** | Check diagram against code; update if needed | Diagram matches code |
