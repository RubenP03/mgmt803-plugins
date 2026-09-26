# shoji

A Quarto reveal.js theme built from the PowerPoint design of the same name —
plum `#595460`, pale gray `#EBEDEB`, dusty blue `#97A7B8`, bold letter-spaced
titles, a Mondrian-ish grid of rectangles. The frame is drawn tight and the type
set small, so each slide carries a good deal of text or code.

## The frame

On a 1280×720 canvas:

| | |
|---|---|
| Rules and block edges | 4px |
| Title band height | 108px |
| Left seam | 72px (5.6%) |
| Base rule | 660px (92%) |
| Text margins, left / right / foot | 116 / 76 / 80 |
| Root font size | 28px |
| Figure height cap | 470px |

That leaves a text area of 1088×504. The foot strip splits at 38% and the picture
layouts meet at the panel's midpoint.

## Install

```
/plugin marketplace add kerryback/mgmt803-plugins
/plugin install shoji@mgmt803
```

Then ask for a deck, or invoke it with `/shoji`.

## Requirements

- Quarto, to render.
- Node plus decktape, only if you want a PDF.

## Layouts

| Class | Layout |
|---|---|
| (none) | Title in a plum band across the top |
| `.band-bottom` | Title in a plum band along the foot, blue block beside it |
| `.plain-title` | No band; plum title on the white panel |
| `.no-title` | Heading hidden, its space reclaimed |
| `.image-left` / `.image-right` | Picture fills half the panel, text beside it |

Vary them — three slides running with the band in the same place is the thing to
avoid. Components (cards, callouts, stats, numbered steps, compared columns,
picture layouts) are documented in
[`skills/shoji/references/components.md`](skills/shoji/references/components.md).

## The one authoring trap

Never put a markdown heading inside a fenced div. Pandoc fuses the two into a
`<section>`, which reveal reads as a vertical stack: the frame doubles up and
forward navigation jumps back to slide 1. Use `[Title]{.card-title}` inside a
card and `[Label]{.eyebrow}` inside any other div.

## Notes

The deck's front matter needs `width: 1280`, `height: 720`, `margin: 0` and
`max-scale: 5`. The grid is specified in px, which are canvas units and scale
with the deck — but only at that canvas size; 1280×720 is 16:9, so the canvas is
the shape of the screen rather than being letterboxed inside it; `max-scale`
lifts reveal's default 2× cap, which would otherwise stop the deck growing at
2560px; and reveal's default 10% margin scales the canvas by 0.9, which leaves
1px slivers along the seams.
