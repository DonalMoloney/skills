---
name: deck-builder
description: Build a presentation deck — slide structure, chart selection, and export to PPTX or PDF — use when producing any formal slide presentation.
---

# Deck Builder

## The Law
**One slide, one point — a slide that tries to make three points makes none of them, and a chart on a slide that the presenter has to explain is a slide that failed.**

## When to Use
- A stakeholder meeting, review, or pitch needs a slide deck
- Technical findings need to be packaged for a non-technical audience
- A written report needs a condensed presentation companion
- An existing deck needs restructuring because audiences are losing the thread
- **Never skip when:** the audience will step through slides without a presenter — self-navigated decks need more structure and clearer labels than live-presented ones

## Process

### Phase 1: Define the Narrative Arc
1. Answer these three questions before writing a single slide:
   - What is the one thing the audience must leave knowing?
   - What decision or action does the deck need to trigger?
   - What objections will the audience bring, and which slide addresses each?
2. Write the slide titles in order as a sentence outline — each title should read as a claim, not a topic heading.
   - Good: "Costs rose 40% because deployments doubled in Q3"
   - Weak: "Cost analysis"
3. Target a maximum of 10 slides for a 20-minute slot; 15 for a 30-minute slot. Cut ruthlessly before adding slides.

### Phase 2: Apply Slide Structure
Build every deck with these structural slots in order:

1. **Title slide** — deck title, presenter name, date, and a single-sentence statement of purpose.
2. **Agenda** — 3–5 bullet points listing the major sections; no sub-bullets.
3. **Context** — one slide establishing why this topic matters right now; include the forcing function if one exists.
4. **Content slides** — the body of the deck; one claim per slide (see Phase 3).
5. **Summary / So What** — three bullets maximum; re-state the main claim and the required action.
6. **Appendix** — supporting data, methodology details, raw tables; never referenced live but available for Q&A.

### Phase 3: Write Each Content Slide
1. Write the slide title as the takeaway, not the topic.
2. Choose one of these layouts per slide — never mix:
   - **Single chart**: one chart, a title claim, and a two-sentence annotation below the chart explaining what the trend means.
   - **Bullet list**: title claim + 3–5 bullets; each bullet is a full sentence, not a noun phrase fragment.
   - **Two-column compare**: left column vs right column with a clear label on each side; use for before/after or option A/option B.
   - **Quote or callout**: large text + attribution + one-sentence context; use for customer voice or key metrics.
   - **Image + caption**: one image filling 60% of the slide; caption states what the image demonstrates.
3. Select the right chart type for the data being shown:
   - Change over time → **line chart**
   - Comparing categories at one point in time → **bar chart** (horizontal for long category names)
   - Part-to-whole composition → **stacked bar** or **pie** (pie only when there are fewer than 5 slices)
   - Relationship between two variables → **scatter plot**
   - Distribution shape → **histogram** or **box plot**
   - Single large number that defines the narrative → **big number callout** (not a chart)
4. Every chart must have: a title, axis labels with units, and a source citation.

### Phase 4: Export
**To PPTX (python-pptx):**
```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN

def build_title_slide(prs: Presentation, title: str, subtitle: str) -> None:
    layout = prs.slide_layouts[0]  # Title Slide layout
    slide = prs.slides.add_slide(layout)
    slide.shapes.title.text = title
    slide.placeholders[1].text = subtitle

def add_content_slide(prs: Presentation, title: str, body_lines: list[str]) -> None:
    layout = prs.slide_layouts[1]  # Title and Content layout
    slide = prs.slides.add_slide(layout)
    slide.shapes.title.text = title
    tf = slide.placeholders[1].text_frame
    tf.text = body_lines[0]
    for line in body_lines[1:]:
        p = tf.add_paragraph()
        p.text = line
        p.level = 0

prs = Presentation()
build_title_slide(prs, "Q3 Cost Review", "Engineering Leadership — 2026-05-03")
add_content_slide(prs, "Costs rose 40% because deployments doubled", [
    "Deploy frequency: 120 → 240 per month",
    "Cloud compute spend: $80k → $112k",
    "No corresponding increase in feature throughput",
])
prs.save("q3-cost-review.pptx")
```

**To PDF (LibreOffice headless, after saving PPTX):**
```bash
libreoffice --headless --convert-to pdf q3-cost-review.pptx
```

**To PDF (reportlab, pure Python):**
```python
from reportlab.pdfgen import canvas

def export_slide_as_pdf(title: str, bullets: list[str], output: str) -> None:
    c = canvas.Canvas(output, pagesize=(960, 540))
    c.setFont("Helvetica-Bold", 28)
    c.drawString(48, 460, title)
    c.setFont("Helvetica", 18)
    for i, bullet in enumerate(bullets):
        c.drawString(64, 400 - i * 36, f"• {bullet}")
    c.save()
```

### Phase 5: Quality Gate Before Sharing
1. Read every slide title in sequence — they should form a coherent argument without the slide body.
2. Check: does every chart have axis labels and a source?
3. Check: does every slide have exactly one main point?
4. Check: is the summary slide free of new information not mentioned earlier?
5. Share with one person outside the project and ask: "What is the deck asking you to do?" — if the answer is wrong, revise the summary slide.

## Red Flags — Stop Immediately
- A slide has more than 5 bullet points — consolidate or split
- A chart has no axis labels or source — the audience cannot evaluate it
- The summary slide introduces a point not covered in the body — it becomes a surprise, not a conclusion
- Two consecutive slides make the same point — one must go
- The deck has 20 slides for a 20-minute meeting — the audience will not finish it

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "More slides means more complete" | Audiences stop reading after slide 10 in a 20-slide deck; completeness lives in the appendix |
| "The chart is self-explanatory" | Every chart needs a title claim that states the insight, not just the metric name |
| "I'll fix the structure after I add the content" | Structure is load-bearing — adding content to a broken narrative arc makes the fix harder |
| "The presenter will explain the complex slide live" | Slides circulate after meetings; the slide must work without the presenter |
| "Pie charts are fine for 8 categories" | Above 5 slices a pie chart is unreadable; use a horizontal bar chart |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Narrative Arc | Write slide titles as claims; set slide count budget | Titles read as a coherent argument |
| 2. Structure | Apply Title → Agenda → Context → Content → Summary → Appendix | All structural slots filled |
| 3. Content Slides | One claim per slide; correct chart type; full chart labels | No slide makes more than one point |
| 4. Export | Generate PPTX via python-pptx or PDF via LibreOffice/reportlab | File opens and renders correctly |
| 5. Quality Gate | Title-only read-through; external sanity check | Audience can state the deck's ask |
