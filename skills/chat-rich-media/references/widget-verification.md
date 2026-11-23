# Verifying an inline widget

A widget is delivered only after it has been rendered, measured and clicked. Every step below caught a
real defect that reading the source did not.

## 1. Open it

```python
goto_url("file://$HERMES_HOME/widgets/<name>.html")   # expand the real absolute path
wait_for_load()
```

Every browser call can start on a blank or stale tab: call `goto_url` at the start of each check rather
than assuming the page from the previous call is still loaded — a bare `js()` then fails with
`Cannot read properties of null`.

## 2. Measure, don't eyeball

```python
js("""(() => ({ docW: document.documentElement.scrollWidth,
  docH: document.documentElement.scrollHeight,
  overflow: [...document.querySelectorAll('.card,.chip,.stat,.shot')]
              .filter(e => e.scrollWidth > e.clientWidth + 1).length,
  imgs: [...document.images].map(i => [i.currentSrc.split('/').pop(), i.naturalWidth, i.naturalHeight]),
  pos: [...document.images].map(i => getComputedStyle(i).objectPosition) }))()""")
```

Check: `docW` equals the viewport (no horizontal scroll), `overflow` is 0, every image has
`naturalWidth > 0` (a fallback URL that 404s still leaves a broken frame), and each portrait tile carries
the `object-position` you set.

## 3. Three widths, not one

```python
cdp('Emulation.setDeviceMetricsOverride', width=960, height=900, deviceScaleFactor=1, mobile=False)
js("new Promise(r=>requestAnimationFrame(()=>r(1)))")   # let it reflow before measuring
# ...measure...
cdp('Emulation.clearDeviceMetricsOverride')
```

Worth testing: **960** (frame maximum), **756** (the usual 2-column collapse), **380** (single column).
A layout that only survives its authoring width is a bug waiting for the reader's window size.

## 4. Prove alignment with numbers

Connector rails and grid centres drift by a few pixels when the bus margin ignores the gap. Compare rects:

```python
js("""(() => { const b = document.querySelector('.bus').getBoundingClientRect();
  const c = [...document.querySelectorAll('.branch')].map(x => {
    const r = x.getBoundingClientRect(); return Math.round(r.left + r.width/2); });
  return {bus: [Math.round(b.left), Math.round(b.right)], centers: c,
          deltaLeft: Math.round(b.left - c[0]), deltaRight: Math.round(b.right - c[c.length-1])}; })()""")
```

`deltaLeft`/`deltaRight` must be 0. Vision passes call a 2–8px offset "aligned" and invent defects that
are not there; the rects are the evidence.

## 5. Click it

```python
js("document.querySelector('[data-k=second]').click()")
js("document.querySelector('.panel h4').textContent")   # read back the text that should have changed
```

Assert the state you expect: exactly one selected card, the matching table row highlighted, the detail
panel carrying the new title. A handler bound to the wrong element looks perfect in a screenshot.

## 6. Screenshot + one vision pass

`capture_screenshot()` takes **no** `width` argument — passing one raises `TypeError: unexpected keyword
argument`. Resize with the CDP override instead, then screenshot, then ask a specific question: are heads
cropped, is text overlapping, is any label too low-contrast, does the caption cover the subject. That pass
reliably catches cropping and contrast; it is **not** evidence about who is in a photo (see
`image-sourcing.md`).

## 7. Report what was verified

Name the widths tested and what the interaction returned. "Rendered and checked" without numbers is the
claim that gets contradicted later.
