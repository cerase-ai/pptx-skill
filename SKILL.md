---
name: pptx
description: "Creates a PowerPoint presentation (.pptx) with editable slides, or an OpenDocument (.odp) or Google Slides version of it. Delegated to by `deck` when the format asked for is editable slides."
---
# PPTX — editable slides

You write the slides as markdown in your workspace, and the office converter turns them into PowerPoint. Your container runs no Python, so you never build the file yourself: the converter does. Invoked by `deck` when the person wants slides they can edit rather than the HTML or PDF deck.

## Inputs

From the caller:
- **draft**: the workspace path of `presentation.md`, the deck `deck` wrote
- **format**: `pptx` (PowerPoint), `odp` (LibreOffice) or `gslide` (Google Slides)
- **file name**: such as `q3-results.pptx`
- **template**: optional, a .pptx in the workspace whose look the slides take

## Step 1 — write the slides file

`presentation.md` is written for the HTML and PDF renderer, and part of its syntax means nothing to PowerPoint. Read it and write `<name>-slides.md` in the workspace, in this shape:

```
---
title: Q3 for the Northwind board
subtitle: Revenue, costs, and the hire we ask for
---

# Part 1 — Results

## Revenue grew 18% on the quarter

- €2.3M in Q3, up from €1.95M
- Three new accounts signed

## Two regions carried the growth

:::::: {.columns}
::: {.column}
North: 12 new accounts
:::
::: {.column}
South: 4 new accounts
:::
::::::

## The hire costs €58,000 a year

| Line | Cost |
|---|---:|
| Salary | €55,000 |
| Tools | €3,000 |

::: notes
What the presenter says on this slide.
:::
```

How each part of `presentation.md` carries over:
- the `+++` block and the cover's `# ` title and line under it → the YAML block's `title` and `subtitle`
- each `## ` slide → a `## ` slide, its title unchanged
- a `:::chapter` act divider → a `# ` heading alone, which becomes a section slide
- `:::columns` with its `:::col` parts → the `{.columns}` block above, one `{.column}` per column
- a `:::chart` → its table, as a pipe table: PowerPoint gets the numbers, not the drawing
- the `---` lines between slides can stay or go: a new `## ` starts a slide either way

## Step 2 — convert

```
call_recipe("cerase-office-converter.convert_md_to_pptx", {"path": "<name>-slides.md", "output_filename": "<name>.pptx"})
```

It answers `{path, filename, size_bytes}`: the deck is in your workspace at `path`, which is `outputs/<name>.pptx`. With a template, add `"reference_doc_path": "<template>.pptx"`.

## Other formats

- **odp**: `call_recipe("cerase-office-converter.convert_pptx_to_odp", {"path": "outputs/<name>.pptx", "output_filename": "<name>.odp"})`
- **gslide** (Google Slides): make the .pptx, then upload it converted, with the upload call below, and only when the person asked for a Google file: the upload puts the content in their Drive. The answer carries the new file's `Link:`; give the person that link. If the Google Workspace connector is not among your connectors, say in their language that Google Slides needs that connector, which the organisation's admin assigns, and send the .pptx instead.

The upload that makes the Google Slides file:

```
call_recipe("google-workspace.uploadFile", {"localPath": "outputs/<name>.pptx", "name": "<title>", "mimeType": "application/vnd.openxmlformats-officedocument.presentationml.presentation", "convertToGoogleFormat": true})
```

These calls are the complete set. Do not invent others.

## Deliver

Attach the file: `[[attach: outputs/<name>.pptx]]`. Never paste its content or any base64 in the chat.

## Style rules

- **Cover**: title and subtitle only, no bullets.
- **Content slides**: at most 6 bullets, one level of sub-bullet at most.
- **Numbers over adjectives**: "€2.3M revenue in Q3" beats "strong revenue this quarter".
- **One point per slide**: two distinct points are two slides.

## Slide count

If the brief said 10 slides, deliver 8 to 12. When the material overflows, ask in their language whether to deliver more slides or which part to leave out; never deliver 25 when 10 were asked for.

## Don't

- Don't write Python, or call `libreoffice` or `pandoc` from bash: none of them is in your container.
- Don't embed video or audio.
- Don't fit an overflowing slide by shrinking its text: split it into two.
