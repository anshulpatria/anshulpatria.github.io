# anshulpatria.github.io

The index for eight live browser builds. One self contained HTML file, no build step,
no dependencies beyond two Google Fonts.

---

## Deploy

These files go in the **root** of the `anshulpatria.github.io` repository.

```bash
git clone https://github.com/anshulpatria/anshulpatria.github.io.git
cd anshulpatria.github.io

# copy the contents of this folder in, replacing the existing index.html
cp -r /path/to/site/. .

git add -A
git commit -m "New index: ASCII odyssey intro and live project previews"
git push
```

GitHub Pages serves the root of `main` for a `<user>.github.io` repository, so it
publishes within a minute or two. Settings, Pages, Source should read
**Deploy from a branch**, branch `main`, folder `/ (root)`.

**Your project subpaths are separate repositories.** `/brandstudio/`,
`/gravitypong/`, `/linevisualiser/`, `/3D-pin-art/` and `/genaiafterhourspub/`
each live in their own repo with their own Pages setting. Nothing here touches them.

### Files

| file | why it is there |
|---|---|
| `index.html` | the whole site |
| `og.png` | 1200x630 social card, generated from the site's own ASCII |
| `favicon.svg` | the blue cursor dot |
| `404.html` | you are adrift at sea, and the raft follows your cursor |
| `robots.txt` | points crawlers at the sitemap |
| `sitemap.xml` | the index plus your five GitHub Pages projects |
| `.nojekyll` | tells Pages to serve files as they are, no Jekyll pass |

The footer is a three track grid, so **Reach out** is genuinely centred rather
than wherever `space-between` leaves it. Both it and **Replay the intro** use
`.gpill`, a glass capsule that shares the same tokens as the rail and the
cursor, lifts 2px on hover and squishes to 0.965 on press.

---

## Glass morphs

Four states that change shape rather than just fade.

**The rail.** Full bleed with a hairline at the top of the index. Once you are
90px into the projects it draws in to a floating capsule: margins open, corners
round to a pill, the hairline goes transparent and a shadow appears. Section 11.
The threshold is measured from the top of the index, not the document, so the
intro's scroll track cannot put it in the tight state before you arrive.

**The hovered row.** Lifts into a slab of glass, inset from the gutters and
rounded. The tint is a gradient on purpose: translucent in the top and bottom
bands so the ground refracts where you can see it, near opaque through the
middle so type stays legible over moving characters. A flat 80% would hide the
refraction completely. The row separator retracts so nothing cuts through it.

**The skip control.** An icon at rest, a capsule reading "Skip to work" on
hover, with the label fading in 100ms behind the width so it does not appear
before there is room for it. It also responds to keyboard focus.

**The preview, inside the row.** There is no floating tile any more. `#art`
sits at z-index 1, below `.shell`, so the hovered row's glass slab paints over
it and the slab's backdrop *is* the preview. The glass genuinely refracts the
image rather than sitting next to it.

`#art` is fixed and moved by transform rather than living inside the row,
because reparenting an iframe reloads it. One image and one iframe serve all
eight rows. `.row` must not have `isolation: isolate`: that forms a backdrop
root and the slab would never see anything outside the row.

The cursor is a plain glass bead and the row glass is a plain panel using the
same material: same specular radial, same rim stack, same refraction, no tint.
Only the hovered row carries a filter.

**A metaball merge was tried and removed.** It worked by shaping one element
with an SVG mask whose contents ran through a blur and a threshold, which does
produce a real union with a gooey neck. The problem is what it forced onto a
single composited layer: `mask`, `backdrop-filter` and `filter: drop-shadow`
together. In Chrome that tears, throwing black triangular artifacts, and
because that element was also the cursor, the cursor vanished with it. Do not
reintroduce it without splitting those three across separate layers, and note
that splitting is not trivial either, since a `filter` on an ancestor forms a
backdrop root and kills the refraction underneath it.

**Clipping the preview.** `#art` carries both `overflow: hidden` and
`contain: paint`, and nothing inside it may use a 3D transform. `overflow:
hidden` alone does not reliably clip a **composited** descendant, and an iframe
is always composited; give that iframe a `translate3d` and it gets its own
layer and escapes the clip, which is how a 1280 wide page ended up covering
most of the window. The child transforms are deliberately 2D.

**Two traps worth recording.** `#art` must sit outside `.shell`. `.shell` is a
stacking context, so a preview inside it cannot be painted under the row glass,
and `.rise` on each row carries a `transform`, which makes `position: fixed`
resolve against a row rather than the viewport. That combination is what let a
1280x1000 iframe spill across the whole page. `#art` also declares `width: 0;
height: 0` so a failure to size it leaves it invisible rather than enormous.

Second: slicing markup out by string boundaries left two orphaned `</div>`
tags once, which closed `.shell` early and wrecked the tree. Check tag balance
after any structural edit.

**On frame rate.** The worst offender was `getBoundingClientRect` inside the
`pointermove` handler, forcing a synchronous layout on every pointer event. Rects
are cached per hover and refreshed on scroll and resize. Two separate three
channel filters, one for the bead and one for the row, are now one. The ambient
ground went from 157,320 cell evaluations a second to 71,820.

Each project aims its preview with `focus` and `focusX`, 0 to 1 across the page.
The artwork is masked from 40% of the row rightward, so without a horizontal aim
the tools show their canvas and hide the control column. At `focusX: 0.11` the
band opens on page x 0. **These values are guesses and should be tuned by eye.**

Both footer pills are **seated, not floating**: a tight contact shadow directly
underneath with no offset, and no lift on hover, the way a paperweight meets
paper. Reach out is blue with a white label.

---

## Responsive

Breakpoints at 900px, 720px and 480px, plus `hover:none` for touch.

| width | changes |
|---|---|
| under 900px | project rows collapse from three columns to one, the legend hides |
| under 720px | the rail readout hides, long URLs truncate, the intro shortens to 820vh |
| under 480px | the dev panel goes full width, the intro shortens to 640vh |
| touch | no hover preview, no glass bead, no click ripple, all pointer only effects |

The ASCII ground drops from 184 columns to 92 below 640px, and the intro from
150 to 70, so the glyphs stay legible rather than turning to grit.

Nothing has a fixed width wider than the smallest viewport. The one exception
is the preview iframe at 1280px, which is deliberate: it renders at desktop
width and is scaled down by transform so framed sites lay out correctly.

---

## The two switches

Both are near the top of their sections in `index.html`.

```js
const INTRO = true;      // the ASCII odyssey intro. false loads the site directly
const PREVIEW = "live";  // "live" iframes, "off" no hover panel
```

`INTRO = false` removes the intro and its scroll track from the document
entirely, so the site loads immediately. Useful when you are editing the index
and do not want to scroll through the odyssey each reload.

Add `?intro=0` to any URL to skip the intro once, `?intro=1` to force it.
Pressing skip, or Escape, suppresses it for that browser tab only.

---

## Custom domain

If you point `artboard42.com` at this repo:

1. Add a file called `CNAME` containing just `artboard42.com`
2. At your registrar, `CNAME` on `www` to `anshulpatria.github.io`, and four
   `A` records on the apex to `185.199.108.153`, `185.199.109.153`,
   `185.199.110.153`, `185.199.111.153`
3. In Settings, Pages, set the custom domain and tick Enforce HTTPS
4. Update the four absolute URLs in `index.html` (`canonical`, `og:url`,
   `og:image`, `twitter:image`) and the two in `sitemap.xml` and `robots.txt`

Note that the site currently lists Artboard 42 as one of the seven projects
pointing at `artboard42.com`. If that domain becomes this site, change that row.

---

## Editing the projects

The list lives in one array near the top of the script. Order in the array is
the order on the page, and it follows the voyage.

```js
{ ord:"I", plate:"lotus", field:4,
  name:"Gravity Pong", url:"...", type:"Game",
  myth:"You meant to leave an hour ago.",
  desc:"Pong with gravity wells in the court...",
  thumb:null }
```

- `thumb` wins over the live iframe. Set it when a site refuses to be framed,
  which is why Field uses its `og.png`. Leave it `null` to boot the real page.
  Orchestrator uses a YouTube thumbnail for the same reason; if `maxresdefault`
  is missing on a video the panel falls back to `hqdefault`, which always exists.
- `field` picks the background: 0 contour, 1 metaballs, 2 line grid, 3 pinscreen,
  4 gravity wells, 5 flow dashes, 6 fluid, 7 registration marks, 8 agent graph.
- `ord` is no longer rendered. It is kept on each project in case the voyage
  order ever wants numbering again; today the left column is the input label.
- `plate` picks the intro creature: `lotus`, `eye`, `pig`, `song`, `bottle`,
  `raft`, `throne`.

---

## Glass dev panel

Type `ddsd` anywhere on the page. Nine live sliders for roundness, refraction,
chromatic spread, frost, saturation, brightness, tint, edge light and shadow.
They drive the rail and the preview tiles together. **Copy CSS** puts the current
values on your clipboard so you can paste them back into the stylesheet.

The panel is inert until you type the sequence, so it is safe to ship.

---

## Known limits

- **Framing.** GitHub Pages allows embedding, so those five preview live.
  `field-8yc.pages.dev` refuses, which is why it falls back to its image.
  If you ever add a project that refuses framing, give it a `thumb`.
- **Fonts.** Bricolage Grotesque and Instrument Sans come from Google Fonts.
  On a network that blocks them the loader still completes on its 7 second
  ceiling and the page renders in the fallback stack.
- **Cost.** The ASCII ground redraws about 10,000 cells at 15fps in JavaScript.
  Comfortable on a laptop. If a low powered machine struggles, lower the column
  count in the background renderer.

---

## License

Copyright (c) 2026 Anshul Patria. All rights reserved.

This is **not** open source. No permission is granted to copy, reuse, modify,
redistribute or deploy any part of this repository or the site it publishes, for
any purpose, including personal and non-commercial ones. You are welcome to look
at it and to link to it. See [LICENSE](LICENSE) for the terms, and ask first if
you want to use something.
