---
name: chat-rich-media
description: "Use when answering visually: widgets, galleries, diagrams."
version: 1.1.0
author: Hermes Agent
license: GPL-3.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [chat, widget, preview, rich-media, gallery, images, mindmap, timeline, visualization]
    related_skills: [claude-design, architecture-diagram, baoyu-infographic]
---

# Chat rich media

How to answer **visually inside the chat surface**: inline HTML widgets, framed image galleries,
diagrams, mind maps, timelines, comparison grids, small charts.

This skill is about the presentation layer only. It says nothing about what the answer should contain —
it says what shape the answer should take, and how to build that shape so it survives the chat frame.

## When to Use

Reach for a visual whenever the content is **structural** rather than narrative:

| The content is… | Show it as |
|---|---|
| a subject with a face, a place or a product (person, company, city, device) | gallery + key-fact strip |
| a taxonomy, architecture, org chart, concept map | mind map |
| a sequence: history, releases, incident, process | timeline |
| a handful of options to choose between | comparison grid |
| a system's connection pattern | diagram grid (SVG) |
| numbers that carry the point | small chart (bars / sparkline) |
| a catalogue of objects | gallery |

Also use it when the user asks directly for “思维导图 / 时间线 / 拓扑 / 图表 / mind map / diagram”.

Do **not** decorate: legal, medical, sexual or grief topics stay text. No pictures there, no diagrams
that make a serious answer look like marketing.

## What Renders Where

| Form | Mechanism | Notes |
|---|---|---|
| table / code / `$LaTeX$` / task list / quote | native markdown | streamed by the model, nothing to build |
| inline HTML widget | write the file, then `::preview{file="/abs/path.html"}` **alone on its own line** | frame spans the chat column to ~960px; height follows content to ~1200px; theme vars injected |
| remote image | `![alt](https://…)` | needs the host to allow hotlinking; can 429 later |
| local file | `MEDIA:/abs/path` | images/audio/video render inline, other types become a card |
| local path inside markdown | — | blocked; must go through `MEDIA:` |

## The Widget Contract

- **No page chrome.** `body` carries no `background`, no `font-family`, no margin. The app injects the
  theme variables and the app font; only a standalone page (mockup, poster, game) overrides them.
- **Colour from variables only**: `--foreground`, `--muted-foreground`, `--accent`, `--border`, `--card`.
  Tint with `color-mix(in srgb, var(--accent) 12%, transparent)` so one file works in light and dark.
- **Fluid and flush left.** No centring wrapper. Grid tracks as `repeat(N, minmax(0, 1fr))` — a bare `1fr`
  overflows on long unbroken strings instead of shrinking.
- **Breakpoints** at ~820 / 720 / 560 / 420: collapse columns, drop connector lines, never take body text
  below ~11px.
- **Height budget ~1200px.** Cap image tiles and let inner regions scroll; do not let the frame scroll
  forever.
- **One interaction minimum.** A widget that only displays is half-done. Expand / filter / select /
  replay — something the reader can poke.
- **Talkback.** `data-hermes-send="<prompt>"` on any clickable element turns that click into a hidden user
  turn. Answer it by **rewriting the widget file**, not with a paragraph of prose.
- **Images inside a widget**: relative `src="img/x.jpg"` plus an `onerror` swap to a remote URL. The
  frame's loader may not resolve the base path that a local `file://` check did.
- **Portraits in fixed-height tiles**: `object-position: center top`. A centred crop cuts the top of the
  head and reads as a bug in a face card.
- **Connector rails across grid columns**: `margin: 0 calc((100% - (N-1)*gap) / (2N))` lands exactly on the
  outer column centres. Off by one gap and the lines drift 2–8px, which the eye reads as sloppy.

## Working Procedure

1. Decide the form from the table above. If two fit, pick the one with fewer moving parts.
2. Write the HTML into a **persistent** directory (`~/.hermes/widgets/<name>.html`), images beside it in
   `~/.hermes/widgets/img/`. A scratch directory gets pruned and leaves the reader with a dead preview.
3. Copy the closest template from `templates/`, keep the layout, replace the copy.
4. Verify: render, measure, resize to three widths, click the interaction, screenshot, one vision pass.
   Full loop in `references/widget-verification.md`.
5. Answer text first, then the directive on its own line, then only what the picture cannot say.

## Seven Patterns

Layout recipes with structure sketches — gallery, mind map, timeline, comparison grid, diagram grid, stat
board, chart: `references/patterns.md`.

## Imagery

Where to search, which license is safe, and how to tell you actually have the right person:
`references/image-sourcing.md`.

## Pitfalls

- `::preview` pointing at a file that was never written renders nothing. Write, then reference.
- Do not re-narrate what the widget already shows. The prose carries the *reading* of it.
- A gallery of mismatched aspect ratios reads as scraped, not curated — fixed tile height, `object-fit:
  cover`, one caption style.
- A single vision pass is not an identity check; see `image-sourcing.md` for the evidence hierarchy.
- Search results are user-uploaded: a file named after a company's "headquarters" can be an interior
  showroom. Caption the frame, not the hope.
- Long CJK strings break naive flex layouts. Test with the real language of the answer.
- Do not answer in a wall of text because the question looks simple — three lines plus one diagram beats
  twenty lines of prose for anything with parts.
