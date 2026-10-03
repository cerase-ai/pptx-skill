# pptx-skill

A Cerase skill that has the assistant produce editable slides: a PowerPoint
file (`.pptx`), an OpenDocument presentation (`.odp`) or Google Slides. The
`deck` skill hands work to it when the person wants editable slides rather than
the HTML/PDF render. The caller passes the workspace path of `presentation.md`
(the draft `deck` wrote), the target format, the file name and an optional
theme.

## What the assistant does

- **`pptx`:** writes a script with `python-pptx` (a title slide, then content
  slides with a title and bullets), saves the file in the workspace and
  attaches it to the reply.
- **`odp`:** builds the `.pptx` first, then converts it with
  `cerase-office-converter.convert_pptx_to_odp`.
- **`gslide`:** calls `google-workspace.slides_create` with the title and the
  markdown, and returns the link.

Style rules: a cover with title and subtitle and no bullets; a title and at
most seven bullets per content slide, with at most one level of sub-bullets;
numbers instead of adjectives; one idea per slide; no decorative images, video
or audio; colour only for background and text; an overflowing slide is split,
never auto-fitted. A request for ten slides gets eight to twelve; when the
material needs more, the assistant asks before going further.

## Requirements

- Python with `python-pptx` wherever the assistant runs code, for `pptx` and as
  the first step of `odp`.
- The `cerase-office-converter` connector for `odp`.
- A Google Workspace connector exposing `slides_create` for `gslide`.

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
