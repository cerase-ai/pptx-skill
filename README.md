# pptx-skill

A Cerase skill that has the assistant produce editable slides: a PowerPoint
file (`.pptx`), an OpenDocument presentation (`.odp`) or Google Slides. The
`deck` skill hands work to it when the person wants editable slides rather than
the HTML/PDF render. The caller passes the workspace path of `presentation.md`
(the draft `deck` wrote), the target format, the file name and an optional
template: a `.pptx` in the workspace whose look the slides take.

## What the assistant does

- **`pptx`:** reads `presentation.md` and writes a `<name>-slides.md` file in
  the workspace in the markdown shape the converter reads: a YAML block with
  title and subtitle, a `##` heading per slide, a `#` heading alone for a
  section slide, a columns block for side-by-side content, a pipe table in
  place of each chart, and a `notes` block for speaker notes. It converts that
  file with `cerase-office-converter.convert_md_to_pptx`, passing the template
  as `reference_doc_path` when there is one. The converter writes the file to
  `outputs/` in the workspace and returns its path, and the assistant attaches
  it with `[[attach: <path>]]`.
- **`odp`:** builds the `.pptx` first, then converts it with
  `cerase-office-converter.convert_pptx_to_odp`.
- **`gslide`:** builds the `.pptx`, uploads it to the person's Drive with
  `google-workspace.uploadFile` and `convertToGoogleFormat: true`, and gives
  the person the link of the new Google Slides file. When the Google Workspace
  connector is not among the assistant's connectors, it says the
  organisation's administrator has to assign it and sends the `.pptx`
  instead.

Style rules: a cover with title and subtitle and no bullets; at most six
bullets per content slide, with at most one level of sub-bullets; numbers
instead of adjectives; one point per slide; no video or audio; an overflowing
slide is split into two, never fitted by shrinking its text. A request for ten
slides gets eight to twelve; when the material needs more, the assistant asks
whether to deliver more slides or which part to leave out. The assistant's
container has no Python, LibreOffice or pandoc, so the assistant never builds
the file itself, and it never pastes file content or base64 in the chat.

## Requirements

- The `cerase-office-converter` connector for every format:
  `convert_md_to_pptx` builds the `.pptx`, and `convert_pptx_to_odp` converts
  it for `odp`.
- A Google Workspace connector exposing `uploadFile` for `gslide`.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The instructions the assistant loads: `name` and `description` frontmatter, then the method per format and the style rules. |
| `cerase.json` | Marketplace manifest: namespace `studio.guidance`, name `pptx`, display name, description, licence. |
| `i18n.yaml` | Italian display name and description for the Marketplace; not sent to the assistant. |
| `LICENSE` | MIT licence text. |

## Installation

Published in the Cerase Marketplace as `studio.guidance/pptx`
([marketplace page](https://marketplace.cerase.ai/en/p/studio.guidance/pptx)).
A Cerase appliance does not attach it by default: an administrator installs it
from the Marketplace.

## License

MIT. See [LICENSE](LICENSE).
