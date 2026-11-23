# Seven layout recipes

Each pattern lists what it is for, the structure that works, and the detail that is easy to get wrong.
Copy the matching file from `../templates/` when one exists; the sketches below are the parts worth
remembering when you build from scratch.

---

## 1. Gallery + key facts (subject profile)

**For:** a person, company, place, product — anything with a face, a place and a handful of facts.

```
header (name · one-line descriptor · badge)
gallery: 3 tiles, fixed height, object-fit: cover, caption strip, license corner
stat strip: 4 small cards (label / value)
timeline strip: 5–6 nodes on one rail
footnote: sources + license
```

- Tile height 150–180px, `grid-template-columns: repeat(3, minmax(0, 1fr))`, collapse to 1 column < 720px.
- Caption sits in a gradient overlay at the bottom of the tile; **it will cover the lower third of the
  photo**, so anchor portraits at the top (`object-position: center top`) or the face ends up under text.
- License label goes top-right, small, translucent — never on top of the subject's face.

## 2. Mind map

**For:** taxonomy, architecture, a concept with 3–5 clusters.

```
root card (accent border)
rail (vertical line, 24px) + bus (horizontal line across the outer column centres)
N branch cards, each with a 14px drop line into the bus
chips inside each branch = leaves
```

- The bus margin is the whole trick: `margin: 0 calc((100% - (N-1)*gap) / (2N))` for N equal
  columns. Anything else drifts 2–8px off the outer card centres.
- Below ~760px the horizontal axis stops making sense: switch to 2 columns, hide the bus, keep the rail.
- Cap 5 branches. A sixth branch means the map is really a table.

## 3. Timeline

**For:** history, releases, an incident, a process — anything where order is the message.

```
right-aligned time labels | rail with dots | cards
each card: title · kind badge · caret
detail inside the card: grid-template-rows 0fr → 1fr (height animation, no JS measuring)
footer: what happens next
```

- The `grid-template-rows: 0fr → 1fr` transition is the only reliable way to animate to auto height.
  Inside it must be a wrapper with `overflow: hidden`.
- One card open at a time; a filter that dims non-matching rows (`opacity` + `grayscale`) beats removing
  them, because the rail stays continuous.
- Parallel time labels (≈22:17) are honest; rounding to a tidy grid is not.

## 4. Comparison grid

**For:** 3–5 options the reader must choose between.

- Cards if each option has a personality; a table if it is purely attribute-by-attribute.
- Never both at full size — pick one, and put the *recommendation* in prose, not in the grid.
- Highlight the selected column with `color-mix(... 10%, transparent)`, not a border, so the grid does not
  reflow when selection moves.

## 5. Diagram grid (SVG shapes)

**For:** connection patterns, flow shapes, arrangement of parts — where drawing beats describing.

- Hand-write the SVG: ~8 lines of `line`/`circle` per shape at `viewBox="0 0 120 66"`, `stroke:
  currentColor` with `stroke-opacity`, so the shape inherits the theme.
- Grid of 4–5 cards, click one → detail panel below updates (pros / costs / typical use). Update the text
  in JS from a data object; do not pre-render five hidden panels.
- Keep one shared `<table>` with a highlighted row for the numeric comparison — it is the fastest thing to
  scan, and it survives narrow widths better than five cards.

## 6. Stat board

**For:** "what did it cost / how big / how fast" — three to six numbers.

- One number per card, big; the unit and the comparison in small caps underneath.
- Always give the comparison (`2266 → 1888 bytes`, `−17%`); a bare number is decoration.
- Do not add icons unless they are drawn in the same stroke weight as the rest.

## 7. Chart

**For:** a series of numbers where the trend is the message. Small inline SVG, no library:

```html
<svg viewBox="0 0 300 80" preserveAspectRatio="none" style="width:100%;height:80px">
  <polyline fill="none" stroke="var(--accent)" stroke-width="2"
    points="0,60 50,52 100,55 150,38 200,30 250,22 300,18"/>
</svg>
```

- Bars: `<rect>` per value with `height` and `y` computed from a max you calculate in JS.
- Never fake an axis. If the numbers are not comparable, use the stat board instead.
- Label the last point; that is the one the reader is looking for.
