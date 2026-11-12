# Style guide

The look of this repository, written down so slides and handouts can match.

The palette and type were read from the public Microsoft AI site
(<https://microsoft.ai>) on November 11, 2026, from its own style variables.
No Microsoft logo or wordmark is used anywhere here, and these materials are
not a Microsoft publication. See the note at the foot of the
[README](../README.md).

## Color

### Surfaces and ink

| Role | Hex | Where it is used |
| --- | --- | --- |
| Cream | `#FAF8F3` | The background of every graphic |
| Warm white | `#FEF9ED` | Text on dark, pills |
| Sand | `#F7ECD9` | The highlighted card in a set |
| Stone | `#F4F1EA` | Quiet boxes |
| Line | `#E1DBC6` | Borders and hairlines |
| Card | `#FFFFFF` | Cards on cream |
| Ink | `#191919` | Headings, dark blocks |
| Deep brown | `#3B230E` | Shadows |
| Body | `#4A413B` | Running text in graphics |
| Muted | `#5D524B` | Captions and secondary text |
| Taupe | `#96826F` | Decoration only. Too light for text |
| Bronze | `#A76F3E` | The one accent: dates, markers, numbered badges |

### The banner

A slate ground (`#6F8499` to `#5E7388`) with soft amber light
(`#F2B56B`, `#E9A66A`, `#F7DFA8`) crossing it on a diagonal, after the hero of
the Microsoft AI site. Text on it is cream.

### Chart colors

The four review bands. These are not brand colors: they were chosen to sit
with the palette and then checked for color-blind separation and for contrast
against cream, and all four pass.

| Band | Hex | Label inside the fill |
| --- | --- | --- |
| Confirmed | `#1B8A6B` | White |
| Worth a look | `#BF7F17` | Ink |
| Unclear text | `#3D7FC4` | White |
| Likely miscoded | `#B8432F` | White |

Always in that order, always with a legend, and never used for anything else.

### Badges

`labelColor=191919`, `color=A76F3E`, `style=flat-square`.

## Type

| Use | Stack |
| --- | --- |
| Headings in graphics | `'Segoe Serif', Georgia, 'Times New Roman', serif` |
| Everything else | `'Segoe Sans', 'Segoe UI', system-ui, -apple-system, Helvetica, Arial, sans-serif` |
| Code | `ui-monospace, 'Cascadia Mono', Consolas, Menlo, monospace` |

Segoe Serif and Segoe Sans are Microsoft's faces. On a machine without them
the graphics fall back to Georgia and the system sans, which keeps the same
character.

Headings are set in serif at regular weight, never bold. Numbers in the stat
tiles are sans, semibold. Small labels are sans, semibold, upper case, with
wide letter spacing.

## Shapes

- Graphics are 1200 units wide, on a cream card with a 24 unit corner radius
  and a one unit line border.
- Cards inside are white with a 16 unit radius. One card in a set may be sand
  with a bronze border, to mark the one that matters.
- Bars are 28 units tall with 4 unit corners and a 2 unit gap between
  segments.
- Screenshots sit on a cream card inside an ink window bar that names what is
  shown and says the data is invented.

## Screenshots

The application's own theme is dark, and it is shown as it is. Every
screenshot is of invented lines. The panel that picks a hospital is hidden,
because it names real ones.

## Files

| File | What it is |
| --- | --- |
| `images/banner.svg` | The title banner |
| `images/generate-vs-decide.svg` | The core idea, in two cards |
| `images/timeline.svg` | The three announcements |
| `images/architecture.svg` | A line's path from the database to a score |
| `images/score-bands.svg` | How a probability becomes a band |
| `images/stats.svg` | The year in five numbers |
| `images/results.svg` | How the model read the lines |
| `images/01` to `07` | Framed screenshots |

The graphics are plain SVG text and can be edited by hand.
