---
name: shoji
description: >-
  Build a Quarto reveal.js slide deck in the Shoji style — a plum, pale-gray and
  dusty-blue theme derived from the PowerPoint design of the same name, with a
  slim frame and small type so a slide holds a good deal of text or code: thin
  rules, a shallow title band, tight margins, cards, callouts, stats, steps, and
  half-bleed picture layouts, rendered to HTML and exported to PDF. Use when the
  user invokes /shoji, asks for a shoji deck by name, or is editing a deck
  already built with this theme. For a general "make me slides" with no style
  named, prefer the pptx skill.
---

# Shoji

Author a presentation as a Quarto reveal.js deck: Markdown in a `.qmd`, a
vendored `.scss` theme, `quarto render` to a self-contained `.html`, decktape to
PDF. The theme does the visual work — the frame, the palette, the tracking, and
a small set of components — so slides look designed without hand-placed boxes.

The look comes from Microsoft's Shoji PowerPoint theme: plum `#595460`, pale gray
`#EBEDEB`, dusty blue `#97A7B8`, bold titles with wide letter spacing, and a
Mondrian-ish grid of rectangles filling the canvas. Every rectangle and rule in
this theme lands on one of a few shared lines, so the shapes line up both within
a slide and from slide to slide. Each layout is a different selection of
rectangles from those lines, which is why the layout classes exist: the source
deck moves its title band and blocks around, and so should a deck built with
this.

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

That leaves a text area of 1088×504. The foot strip splits at 38% and the
picture layouts meet at the panel's midpoint.

## When to use this vs. other deck skills

- Use shoji when the user asks for it by name, or when editing a deck whose
  front matter already points at `shoji.scss`.
- Use the `pptx` skill for a general request with no style named, and always
  when the user needs a natively editable PowerPoint.
- Shoji suits text- and code-heavy academic decks that were fighting the
  panel's edges: same quiet palette, appreciably more canvas.

## Prerequisites

- Quarto, to render. `quarto --version`.
- decktape plus Node, only for PDF export. `npm install -g decktape`.

Render first; only chase a missing tool if the render actually fails.

## Workflow

### 1 — Scaffold the deck folder

Work in a folder that will be the deliverable, and put the theme beside the
`.qmd` so the relative `theme:` resolves:

- Copy `assets/shoji.scss` into the deck folder.
- Copy `assets/starter.qmd` and rename it, or write fresh front matter:

```
---
title: "Your title"
subtitle: "Optional subtitle"
author: "Kerry Back"
date: today
format:
  revealjs:
    theme: shoji.scss
    width: 1280
    height: 720
    margin: 0
    max-scale: 5
    slide-number: c/t
    footer: "Course or talk name"
    highlight-style: github
---
```

`width: 1280`, `height: 720`, `margin: 0` and `max-scale: 5` are load bearing.

- The grid is specified in px, which are canvas units and scale with the deck —
  but only if the canvas is that size.
- 1280×720 is 16:9. Reveal scales the canvas but never reshapes it, so a canvas
  that is not the screen's shape is letterboxed: the frame's rectangles sit in a
  fixed region while the pale viewport background fills the rest of the window.
- `margin: 0` matters twice over: reveal's default 10% margin scales the canvas
  by 0.9, which puts every edge on a fractional device pixel and leaves 1px
  slivers of the wrong colour along the seams.
- `max-scale: 5` overrides reveal's default cap of 2×. Without it the deck stops
  growing at 2560px and just sits in the middle of any larger window.

### 2 — Write the slides

`##` starts a slide; `#` starts a section divider, which the theme renders as a
plum band automatically. Body content is plain Markdown plus the theme's classes
— read `references/components.md` and build with those rather than ad-hoc CSS.

No speaker notes. The presenter narrates; a `::: {.notes}` block is a place for
cut text to hide instead of being cut.

### 3 — Prune what you drafted

A slide is a visual aid for someone talking over it. Cut whole sentences and
whole bullets, not words inside kept sentences: anything that explains what the
slide already states, narrates what the audience can see, foreshadows a later
slide, or expands a card's own title. Keep the claim and the specifics nobody
can reconstruct from hearing them once — a number, a name, a path, a command,
a line of code.

### 4 — Hand the prose to the editor subagent

You drafted the prose; you do not edit it down yourself. Launch the
`slide-prose-editor` subagent and wait for it:

```
Agent({
  subagent_type: "slide-prose-editor",
  run_in_background: false,
  description: "Edit slide prose",
  prompt: "Edit the prose in <folder>/<name>.qmd. Apply your edits directly."
})
```

Wait for it rather than backgrounding it: the next step depends on its edits,
because cutting text changes whether a slide still fits the panel.

It applies edits, it does not propose them. Do not re-edit its output and do
not restore anything it cut.

Do not relay its report, quote its edits, or mention that it ran. The user
wants the finished deck, not the steps that produced it. If it flags a factual
claim as suspect, fix the fact and stay silent about the rest.

The authority it works from is
`plugins/shoji/skills/shoji/references/what-not-to-do.md` in the user's
skills repo — the catalogue of text that has been cut from these decks, with
the replacement for each. Read it yourself before step 2 so there is less for
the subagent to find.

### 5 — Render and look at every slide

```
quarto render <name>.qmd
```

Reading the `.qmd` will not tell you whether a slide fits, and this theme does
not shrink text to fit: content longer than the panel runs straight past the
bottom rule. The slimmer frame buys room, it does not buy a safety net — and the
smaller type makes it easy to keep adding until a slide is a wall. Serve the deck
and screenshot each slide, then look at them.

```
python3 -m http.server 8712 &
```

Reveal decks need HTTP; they misbehave from `file://`. Walk the deck with
`window.Reveal.next()` between screenshots rather than jumping by hash — id-based
hash navigation can bounce back to slide 1.

Fix an overflowing slide by splitting it or cutting content.

### 6 — Say what would make it better

Once the draft is whole, read it as a sequence and tell the user, unprompted,
what would raise it: layouts that repeat, claims the deck asserts but could show,
a real number or screenshot instead of four bullets. Name the slide, name the
change, offer to make it.

### 7 — Export (optional)

```
decktape reveal http://127.0.0.1:8712/<name>.html <name>.pdf --size 1280x720
```

decktape hangs on `file://` URLs — always give it the served URL. The exported
PDF keeps the frame, the footer, and the slide numbers.

## Conventions (hard rules)

- Never put a markdown heading inside a fenced div. Pandoc fuses the heading and
  the div into a `<section>`, reveal reads that as a vertical stack, the frame
  doubles up and forward navigation jumps back to slide 1. Inside a `.card` use
  `[Title]{.card-title}`; inside any other div use `[Label]{.eyebrow}`. The
  slide's own `##` heading is fine and gives the slide a usable id.
- `##` starts a slide. No `---` rules between slides; they create blank ones.
- Keep the title's letter spacing. The tracking is what makes this design read
  the way it does — don't override `letter-spacing` on headings.
- Relative paths for images and assets; the deck folder moves.
- No long inline code inside a card, a stat, or a compared column. Inline code
  never wraps, so one wide backticked string forces its column open and collapses
  the rest of the row. Put commands and paths in a `.note`, a `.lead`, or a
  fenced block, where the full panel width is available.
- Preserve the user's wording when restyling an existing deck. Change classes
  and layout, not prose, unless asked.
- Read `references/what-not-to-do.md` before writing slide prose, and check the
  finished deck against it. It is the running catalogue of writing that has been
  cut from these decks, with the replacement in each case; add to it whenever
  something else gets cut.
- No commentary, and above all no knowing asides. A knowing aside is a clause
  that implies first-hand experience of how people in organizations behave —
  "and people are paid on those numbers" tacked onto a point about reported cost.
  Someone who had consulted for twenty years could write that and it would carry
  their authority. Written into an instructor's deck it puts a claim about the
  world into their mouth that they are not making, which is a misattribution
  rather than a matter of taste. The slide carries the analysis; what to add from
  personal experience is the instructor's to decide, standing in the room.
- Out on the same grounds: verdicts about what matters most ("usually the most
  useful number in the model"), aphoristic closers, rhetorical dismissals
  ("answers a question nobody asked"), dramatizing vocabulary ("ruinous" for
  "far off"), and commentary on the session itself ("twice today", "take your
  time on this one"). An `.eyebrow` carries a factual locator — a place, a date,
  a running time, a source — not a cue about significance.
- The test for any sentence: if it states a fact about the project, the method or
  the tax code, keep it. If it says how to feel about that fact, how much it
  matters, or what kind of people are involved, cut it.
- Ask questions that can be answered by reasoning from the material, not by
  guessing the answer already in your head. A question naming the fact it wants
  ("under a freeze, what does the requisition count measure?") or carrying the
  answer inside it ("is it 7 percent? why not?") only works for someone who has
  already worked the case. Ask what each party is claiming, what would have to be
  true for each to be right, what the student assumed and how much rests on it.
  Slide titles follow the same rule — a title that states the finding spends it
  before anyone has read the slide.

## Authoring guidance

- One idea per slide; let the panel's whitespace carry the rest.
- Vary the slide layout, not just the components. The banded default is the
  workhorse; move the band to the foot with `.band-bottom` every few slides, drop
  it with `.plain-title` when the slide already carries a lot (code, a wide
  table, a figure), and break the run with a picture layout. Three consecutive
  slides with the band in the same place is the thing to avoid — on a projector
  the audience sees one silhouette for the whole session.
- Vary the components too. Cards are the easy default and so the one to ration —
  no single one on more than about a third of the content slides.
  `references/components.md` has stats, steps, compared columns, and picture
  layouts for exactly this reason.
- Prefer cards to bullet lists once items run past a phrase each; prefer a
  picture layout to a fifth card slide.
- Open each major part with a `#` heading — the plum band is the deck's rhythm.
- The palette is three colors. `.card-sage` and `.card-sand` exist for the rare
  fourth category; reaching for them often turns a quiet design loud.
- Charts and diagrams as SVG or high-dpi PNG from matplotlib. The canvas is only
  1280×720 CSS pixels but is presented full-screen. Figures may run to 470px
  tall.
- The extra room is for breathing space and for figures, not a licence to fill
  the slide. Body type is 28px on a canvas shown at a distance; a slide that uses
  every pixel of the wider panel is a slide nobody at the back can read.
