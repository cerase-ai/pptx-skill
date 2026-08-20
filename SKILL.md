---
name: pptx
description: "Creates a PowerPoint presentation (.pptx) with editable slides, or an ODP / Google Slides variant. Delegated to by `deck` (Path B) when the requested format is editable PPTX."
---
# PPTX — PowerPoint / editable slides

Create `.pptx` PowerPoint slides (or .odp / Google Slides variant) from structured content. Invoked by `deck` (Path B) when the user wants editable slides rather than the fast HTML/PDF path.

## Inputs

From the caller:
- **draft**: workspace path of `presentation.md` (format-agnostic markdown produced by `deck` Stage 2)
- **target format**: `pptx` (PowerPoint), `odp` (LibreOffice), `gslide` (Google Slides)
- **filename**: e.g. `q3-results-presentation.pptx`
- **theme**: optional — default "clean light" if unset

## Backend per format

### `pptx` (native, fast)

Use python-pptx in the workspace:

```python
from pptx import Presentation
from pptx.util import Inches, Pt
prs = Presentation()
# Title slide
slide = prs.slides.add_slide(prs.slide_layouts[0])
slide.shapes.title.text = '<title>'
slide.placeholders[1].text = '<subtitle>'
# Content slide
slide = prs.slides.add_slide(prs.slide_layouts[1])
slide.shapes.title.text = '<heading>'
tf = slide.placeholders[1].text_frame
for i, bullet in enumerate(bullets):
    p = tf.paragraphs[0] if i == 0 else tf.add_paragraph()
    p.text = bullet
    p.level = 0
prs.save('<filename>.pptx')
```

Write to workspace. Attach to reply.

### `odp` (via cerase-office-converter)

Create `.pptx` natively, then:

```
call_recipe("cerase-office-converter.convert_pptx_to_odp", {input_b64: <base64 of .pptx>})
```

### `gslide` (via google-workspace MCP)

```
call_recipe("google-workspace.slides_create", {
  title: "<filename without ext>",
  slides_markdown: <full file contents>,
})
```

Returns `{deck_id, deck_url}`.

## Style rules

- **Cover slide**: title (40-44pt) + subtitle (18-22pt). No bullet list on cover.
- **Content slides**: title at top (28-32pt), ≤ 7 bullets in body (16-18pt). Use `level` 0 for top bullets, `level` 1 for sub-bullets (max).
- **Numbers > adjectives**: "€2.3M revenue Q3" beats "strong revenue this quarter".
- **One concept per slide**: if you find yourself cramming 2 distinct points, split into 2 slides.
- **No clipart / decorative images**: PoC slides are content-first. Photos go in workspace separately if uploaded.

## Slide count discipline

If the brief said "10 slides", produce 8-12 (10% wiggle room). If the source content overflows, propose: "il materiale è denso, ti consegno 14 slide o ne lascio fuori X?". Don't silently produce 25 slides when asked for 10.

## Don't

- Don't try to embed videos / audio in PPTX — fragile across viewers.
- Don't use color outside title (background) + bullet (text) — themes break otherwise.
- Don't auto-fit text: when a slide overflows, split it. Auto-fit produces unreadable 8pt slides.
