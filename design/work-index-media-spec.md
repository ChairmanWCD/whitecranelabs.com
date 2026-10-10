# Work-Index Media Spec

**File:** `studio/Website/design/work-index-media-spec.md`
**Author:** studio-designer · **Date:** 2026-10-06
**Owner of this doc:** designer. **Owner of the edits it describes:** studio-engineer.
**Applies to:** `studio/Website/index.html`, section `#work` (the six `.project` rows).

This is written-direction deliverable. The designer did **not** edit `index.html` or any
case-study HTML. Every CSS/HTML block below is copy-paste ready.

Companion token work (small-red text, crane accent, handwriting register) lives in
`DESIGN.md` — see *Delta* section at the root of this repo. It is referenced here but not
repeated.

---

## 0. Two traps before you type anything

Read these first; both will bite silently.

### Trap A — `.project` is shared with `#background`

`#background` re-uses the `.project` class for its **Education** and **Skills** rows
(`index.html` ≈ lines 474 and 492). Those rows carry `.project-glyph`, never
`.project-media`. If you change bare `.project { grid-template-columns: … }` to three
columns, the Education degrees and the entire Skills chip cloud get pushed into the
260 px media track.

→ **All new rules are scoped `#work .project`.** Not `.project`. Not `.work-index .project`
(`#background` wraps its rows in `.work-index` too).

### Trap B — the `#work` scope out-ranks the 640 px collapse

The existing responsive rule is:

```css
@media (max-width:640px){.project,.exp{grid-template-columns:1fr}…}   /* specificity 0-1-0 */
```

`#work .project{grid-template-columns:84px 260px 1fr}` is specificity **1-1-0**. The ID wins,
so the 640 px `1fr` collapse **silently stops applying to the work rows** and mobile stays
three-column forever.

→ Every breakpoint override in this spec repeats `#work`. Do not "tidy" it back to
`.project`.

---

## 1. Grid — two columns → three

### 1.1 Geometry

| Token | Value |
|---|---|
| Desktop breakpoint | **900 px** (three-column layout applies at ≥900 px) |
| Columns, ≥900 px | `84px 260px 1fr` |
| Gap | `clamp(1rem, 3vw, 2.25rem)` — **unchanged** from the current `.project` rule |
| Column order (DOM = visual) | `1 = .project-num` · `2 = .project-media` · `3 = .project-main` |

`.project-num` stays exactly as it is (84 px track, `clamp(2.6rem,5vw,3.8rem)` ghost numeral
at `opacity:.14`). The 84 px track is already tight at the 3.8 rem top size — do not shrink it
to make room for media.

### 1.2 Why 260 px, and why the breakpoint is 900 px

`.wrap` is `max-width:1080px` with `padding:0 clamp(1.25rem,4vw,3rem)`, so the content box at
the cap is **984 px**. At the desktop gap of 36 px:

```
84 (num) + 36 + 260 (media) + 36 + main = 984   →  main = 568px
```

`.project-desc` is capped at `62ch` ≈ 494 px at 14.5 px Inter, so the text column never gets
clamped at desktop. Comfortable.

Narrowing the viewport, the text column is:

```
main(V) ≈ 0.86·V − 344        (V in 640…1080, where 4vw and 3vw are both unclamped)
```

| Viewport | media | main |
|---|---|---|
| 1080 | 260 | 587 |
| 900 | 260 | **430** |
| 860 | 260 | 396 |
| 760 | 260 | 310 ✗ |
| 641 | 260 | 207 ✗ |

`main` stops being a comfortable measure at about **V ≈ 890 px**. **900 px** is the breakpoint
because it is the round number above that crossing, not because it is a device width.

### 1.3 CSS — replace the one existing `.project` rule

Find (≈ `index.html` line 160):

```css
.project{padding:2.2rem 0;border-top:1px solid var(--border-lo);display:grid;grid-template-columns:84px 1fr;gap:clamp(1rem,3vw,2.25rem);align-items:start}
```

Replace with:

```css
.project{padding:2.2rem 0;border-top:1px solid var(--border-lo);display:grid;grid-template-columns:84px 1fr;gap:clamp(1rem,3vw,2.25rem);align-items:start}
/* Work-index media column. Scoped to #work — see Trap A. Keep the base .project
   rule above untouched so #background Education/Skills rows stay two-column. */
#work .project{grid-template-columns:84px 260px 1fr}
```

`.exp` has its own separate rule (`grid-template-columns:84px 1fr`) and is **not** touched.

### 1.4 Tablet band, 641–899 px — media sits above the copy, in the text column

Three columns do not survive below 900 px (§1.2). Drop back to the existing two-column
`84px 1fr` and place the thumb as a hanging tile at the head of the text column. Column 1 row 2
is left empty — harmless, because nothing in the work rows paints a background or border on a
cell, and `.project-num` is a `opacity:.14` ghost.

Insert **immediately before** the existing `@media (max-width:640px)` block (≈ line 224):

```css
@media (max-width:899px){
  #work .project{grid-template-columns:84px 1fr}
  #work .project-num{grid-column:1;grid-row:1}
  #work .project-media{grid-column:2;grid-row:1;max-width:220px}
  #work .project-main{grid-column:2;grid-row:2}
}
```

The `aspect-ratio` on the media (below) keeps 4:3 at 220 px → 165 px tall. Ratio never changes;
only the width does.

### 1.5 ≤640 px — degrades gracefully with `.project-num`

The existing block already hides `.project-num`, `.exp-num` and `.project-glyph`. Add the
`#work`-scoped collapse and let the thumb go full content width (Trap B):

```css
@media (max-width:640px){.project,.exp{grid-template-columns:1fr}.project-num,.exp-num,.project-glyph{display:none}.coda-cols{grid-template-columns:1fr}
  #work .project{grid-template-columns:1fr}
  #work .project-media{grid-column:1;grid-row:1;max-width:none}
  #work .project-main{grid-column:1;grid-row:2}
}
```

With the number gone, source order (`media` before `main`) already puts the thumb on top —
the explicit `grid-row` values are there so the placement does not depend on the numeral
staying hidden.

---

## 2. One aspect ratio for all six thumbs

> ### **Aspect ratio: `4 / 3` (1.3333). Every thumb, no exceptions.**

```css
#work .project-media{
  display:block;
  aspect-ratio:4/3;
  border:1px solid var(--border);
  border-radius:2px;
  overflow:hidden;
  background:var(--border-lo);
  min-width:0;
}
#work .project-media img{
  display:block;
  width:100%;
  aspect-ratio:4/3;
  object-fit:cover;
  object-position:center;
}
```

`aspect-ratio` is declared on **both** the frame and the `img`. On the `img` it derives the
height from the 100% width so no percentage-height resolution is needed; on the frame it holds
the box open if the image 404s, so a missing asset costs no layout shift.

### 2.1 Why 4:3 and not 3:2 or 16:10

Measured native ratios of the five sources that already exist:

| Asset | Native ratio | Crop to 4:3 | Crop to 3:2 |
|---|---|---|---|
| `assets/heopet/master.png` | 1.333 (1024×768) | **none** | 12% off the width |
| `assets/white_crane_studio/ui.png` | 1.451 (1908×1315) | 8% off the width | none |
| `assets/white_crane_studio/hero.png` | 1.730 (1024×592) | 23% off the width | 13% |
| `assets/critique/header.png` | 1.333 (1600×1200) | **none** | 12% |
| `assets/critique/ui.png` | 2.048 (3781×1846) | 35% off the width | 27% |

4:3 leaves two sources untouched and the rest lightly cropped, and it buys 22 px of extra
height at the 260 px slot (**260×195** vs 260×173). That height matters because three of the
six thumbs are screenshots — legibility at 260 px is the brief's actual acceptance test, and
height is what buys it.

**Legibility check at 260 px wide:** 4:3 → 195 px tall. A screenshot cropped to that box still
resolves 11 px mono at ~1.4× downscale, and the cat-pill / chip rows stay readable. This is the
size each generated asset was art-directed against.

---

## 3. Theme survival — the `#work` band is two different bands

`#work` is `.band-surface`:

| | `--black` | `--surface` | `--border` |
|---|---|---|---|
| **dark** | `#1C1B18` | **`#212121`** | `#3A3A3A` (solid) |
| **light** | `#F5F2EC` | **`#EDE9E0`** | `rgba(30,28,24,.14)` |

The thumbs are **content, not chrome**. They must not be re-tinted per theme and they must not
be tuned for one of them.

### Rules

1. **Each asset keeps its own ground colour.** The HeoPet thumb stays teal `#34707F`, Green
   Kite stays paper `#F2F6F2`, the shop thumb stays warm grey. There is no "work-band
   background" baked into any asset, and nothing gets recoloured per theme.
2. **Every asset is fully opaque. No alpha.** No transparency-dependent artwork, no cut-out
   PNGs, no `mix-blend-mode`, no `opacity` on the wrapper. The frame's
   `background:var(--border-lo)` is a *loading* placeholder only — because the assets are
   opaque it is never visible in normal use, and it must never be load-bearing.
3. **Every thumb gets the same 1 px hairline frame, `1px solid var(--border)`, `border-radius:2px`.**
   This is not decoration, it is the theme bridge, and it is the reason one rule works for both
   modes. Measured perimeter luminance of the real assets against both band colours:

   | Asset | perimeter avg | vs `#212121` | vs `#EDE9E0` | needs the hairline |
   |---|---|---|---|---|
   | `heopet/master.png` | `#34707F` | 2.89:1 | 0.22:1 (lighter) | yes — light band |
   | `white_crane_studio/ui.png` | `#0A0A10` | **0.82:1** | 0.06:1 | **critical** — near-invisible on dark |
   | `white_crane_studio/hero.png` | `#381F5A` | 1.16:1 | 0.09:1 | **critical** |
   | `critique/ui.png` | `#B1AAA6` | 7.03:1 | 0.53:1 | yes — dark band |
   | `critique/header.png` | `#C9C3BF` | 9.23:1 | 0.69:1 | yes — dark band |

   `white_crane_studio/ui.png` at **0.82:1** against `#212121` is the case the frame exists for:
   without it the thumbnail simply is not there in dark mode, and adding a drop shadow or a
   theme-swapped ground would both break the "content, not chrome" rule.

4. **`var(--border)` is the correct token in both themes** — solid `#3A3A3A` on dark reads as a
   crisp edge; `rgba(30,28,24,.14)` on light reads as a whisper. Do not substitute
   `--border-lo` for the frame (too weak on dark), and do not use `--red` at rest (the site
   reserves `--red` for interaction and glyph marks).
5. **No dark-only or light-only artwork.** Reject any asset whose composition depends on the
   band behind it — no white-keyed logos, no black-background marketing art intended to bleed
   into a dark page, no "inverse" variants. `white_crane_studio/hero.png` and `ui.png` are
   near-black and are tolerated *only* because the hairline frames them; they are not
   precedent for adding more near-black art to this band.
6. **Do not add a `box-shadow`.** The band already carries `border-top/bottom: 1px var(--border-lo)`
   and the site's depth language is reserved for `.nav-pill` and `.btn-primary`.

---

## 4. Hover treatment

The site's motion vocabulary, read off the existing rules: `--ease-out:
cubic-bezier(0.16,1,0.3,1)`, ≤0.35 s, transform/opacity only, no scale-bounce. Precedents:
`.link-primary:hover{gap:10px}`, `.connect-link:hover .link-arrow{transform:translate(3px,-3px)}`,
`.nav-links a:hover::after{width:100%}`, `.work-scan a:hover{border-color:var(--red)}`.

So: **no scale, no lift, no parallax, no crossfade, no filter.** One opacity step on the image
plus the hairline going to the interaction colour. Note that `border-color` is already inside
the site's hover vocabulary (`.work-scan a` transitions `border-color .2s`), so the frame shift
is in-idiom; the opacity step is the only change that touches the image itself.

```css
#work .project-media{transition:border-color .28s var(--ease-out)}
#work .project-media img{transition:opacity .28s var(--ease-out)}
#work .project:hover  .project-media,
#work .project:focus-within .project-media{border-color:var(--red)}
#work .project:hover  .project-media img,
#work .project:focus-within .project-media img{opacity:.84}
```

- `:focus-within` is not optional — keyboard users tabbing to `a.project-title` must get the
  same affordance as a mouse hover. `.project` wraps the title link, so it works for free.
- 0.28 s, single value, `--ease-out`. Inside the ≤0.35 s ceiling.
- `opacity:.84` is a *dim*, not a flash: it lowers rather than raises luminance, which keeps
  `white_crane_studio/ui.png` from glaring against `#212121` on hover.
- Do **not** transition `border-width` (reflow) or `object-position` (reads as a pan).
- Do not wrap this in `transform:scale()`; a scale inside `overflow:hidden` at 260 px is
  exactly the scale-bounce the site avoids.

### 4.1 Reduced-motion override

```css
@media (prefers-reduced-motion:reduce){
  #work .project-media,
  #work .project-media img{transition:none}
}
```

The end states are kept — a `prefers-reduced-motion` user still gets the red hairline and the
dim, instantly. The global `@media (prefers-reduced-motion:reduce){*{transition-duration:.01ms!important}}`
already at line 244 covers this in practice; this rule is explicit so the intent is legible at
the component and survives someone tightening the global rule later.

---

## 5. Per-row asset assignment

DOM insertion for every row: **between `.project-num` and `.project-main`.**

```html
<div class="project pj-blue" data-stagger="1">
  <div class="project-num">01</div>
  <div class="project-media"><img src="…" alt="" width="1152" height="864" loading="lazy" decoding="async"></div>
  <div class="project-main"> … </div>
</div>
```

**`alt=""`.** The adjacent `a.project-title` already carries the accessible name one node
away; a descriptive alt makes screen readers announce "HeoPet … HeoPet mobile app screenshot".
These thumbs are decorative restatements. If you later make the thumb itself the link, move the
name onto the link and keep the image `alt=""`.

**`loading="lazy" decoding="async"` on all six.** `#work` sits below a `100vh` hero, so nothing
here is LCP. Do **not** set `fetchpriority="high"` on any thumb.

**`width`/`height` attributes = the intrinsic size of the file you ship**, so the box is right
before CSS lands.

### Row 01 — HeoPet (`assets/heopet/master.png`)
- **Native 1024×768 = exactly 4:3. No crop. No resize.** Ship as-is, `object-position:center`.
- Ground `#34707F` teal — sits comfortably on both bands. The most theme-safe asset on the page.
- ⚠️ `assets/heopet/master.png` is **byte-identical to `assets/repo-cover.png`**
  (SHA-256 `9835FF05CA6C9027…`). Same picture, two paths, 693,781 bytes each; `repo-cover.png`
  is the `og:image` on `index.html` and `About.html`. See §7.

### Row 02 — WCD Creative Studio (`assets/white_crane_studio/hero.png`)
- **Client direction 2026-10-09: ship the hero art, not the UI screenshot.**
  `hero.png` is 1024×592 (1.730) → 4:3 keeps full height and loses **23% of the
  width** (centre crop to 789×592 via `object-fit:cover; object-position:center`
  in the browser — no file edit required). Same violet-ground engine art as the
  case-study hero (`WCD_Creative_Studio.html:445`), so the index tile and the
  case study open on the same image.
- ⚠️ Ground `#381F5A` measures **1.16:1 against `#212121`**. Dark-on-dark: the §3
  hairline is mandatory on this row. Supersedes the earlier `ui.png` call
  (near-black `#0A0A10` at 0.82:1, 8% crop) — `ui.png` remains in `assets/`,
  unused by the index.
- Naming trap: this directory is **row 02** (the studio/engine case study). It is *not* the
  shop. Row 06 must not reuse it — see §7.

### Row 03 — The Critique Council (`assets/critique/header.png`)
- **Native 1600×1200 = exactly 4:3. No crop.** `object-position:center`. Ground `#C9C3BF`
  (top-left `#F6F6F6`) — light, so it holds on the dark band and needs only the hairline on
  light.
- Rejected alternative: `assets/critique/ui.png` is 3781×1846 (**2.048**) and would lose **35%
  of its width** to reach 4:3, which is a re-composition, not a crop. Prefer `header.png`.
- ⚠️ `header.png` also exists as `Critique Council Header.png` in the site **root** —
  byte-identical (`4DAEC794335B0B3B…`), 2,400,530 bytes, and referenced (with a space-in-URL)
  by `Critique_Council_Live.html` and `Critique_Council_Thesis.html`. See §7.

### Row 04 — Pratt's AI Infrastructure — **no asset exists. Do not fabricate one.**
There is no Pratt artwork anywhere in `assets/`, and generating invented "institutional" art
for an employer's case study is not a call the designer or engineer should make unilaterally.

**Recommended fallback — a CSS glyph tile, zero new assets, zero hallucinated imagery:**

```html
<div class="project-media project-media--glyph" aria-hidden="true"></div>
```

```css
#work .project-media--glyph{display:flex;align-items:center;justify-content:center;
  background:color-mix(in srgb,var(--gold) 14%,var(--black))}
#work .project-media--glyph::after{content:'';width:38%;aspect-ratio:1;
  border:2px solid var(--gold);opacity:.5;transform:rotate(45deg)}
```

- Reuses the `pj-gold` accent the row already owns and the site's existing rotated-diamond
  motif (`.work-group .dot`, `.marquee-item .dia`, `.coda-detail summary::before`).
- `color-mix(… ,var(--black))` is an **opaque** computed ground in both themes, so rule §3.2
  holds; the tile reads as a deliberate institutional placeholder rather than a broken image.
- It degrades correctly at 641–899 px and ≤640 px with no extra CSS.
- **Treat this as a slot, not a solution.** The real fix is a screenshot of a Pratt pipeline
  or a document Ken can approve. Flagged in the report as the one row still owed artwork.

### Row 05 — Green Kite (`assets/green-kite/hero.png`)
- **Client direction 2026-10-09: ship the case-study hero, not the generated
  thumb.** `hero.png` is 1024×1024 (square) → 4:3 keeps full width and loses
  **25% of the height** (centre crop to 1024×768 via `object-fit:cover;
  object-position:center` in the browser — no file edit). Provenance:
  byte-identical extraction of the `.hero-right` artwork embedded (base64) in
  `Green_Kite_Brand_Identity.html` — no re-encode, no regeneration — so the
  index tile and the case study open on the same image. The generated
  `index-thumb.png` / `.webp` (1152×864, §9.3–§9.4) remain in `assets/`, unused
  by the index.
- Brand-world rule stands: **this thumbnail must read as Green Kite, not as
  White Crane** — it is a client brand world. Do not recolour it, do not let it
  inherit `--surface`, and do not "harmonise" it with the charcoal/red system.
  Its difference is the point.
- ⚠️ **Row 05 is the row where the §3 hairline is load-bearing.** Green Kite's authentic paper
  is a near-white, and *any* near-white ground sits only **1.11:1** from the light band
  `#EDE9E0`. That is not a defect of this asset — it is unavoidable for a paper-ground identity
  mark — but it means in **light** mode the frame, not luminance separation, is what stops the
  thumbnail dissolving into the band. Do not drop the border on this row, and do not "fix" the
  apparent emptiness by adding a background or a shadow.

### Row 06 — White Crane Design Shop (`assets/wcd-shop/index-thumb.png` + `.webp`) — **NEW, generated**
- 1152×864, exactly 4:3. `object-position:center`. No crop.
- Photographic garment flat-lay. It is intentionally the only photograph in the index; it
  signals *product*, not *AI art*.

### Generation provenance
Both new assets were generated on the local Z-Image server. Full prompts, negative prompts,
seeds, sizes, rejected rolls and the objective QA gates are recorded in **§9** of this
document. Reproduce or re-roll from §9.

---

## 6. Markup — the six `<img>` tags, ready to paste

Ship the `<picture>` form (WebP with PNG fallback). Order matters: most-specific source first.

```html
<!-- 01 HeoPet — existing, no crop -->
<div class="project-media">
  <img src="assets/heopet/master.png" alt="" width="1024" height="768" loading="lazy" decoding="async">
</div>

<!-- 02 WCD Creative Studio — hero art, 23% centre crop via object-fit (client call 2026-10-09) -->
<div class="project-media">
  <img src="assets/white_crane_studio/hero.png" alt="" width="1024" height="592" loading="lazy" decoding="async">
</div>

<!-- 03 The Critique Council — existing, no crop -->
<div class="project-media">
  <img src="assets/critique/header.png" alt="" width="1600" height="1200" loading="lazy" decoding="async">
</div>

<!-- 04 Pratt — CSS glyph placeholder, see §5 Row 04 -->
<div class="project-media project-media--glyph" aria-hidden="true"></div>

<!-- 05 Green Kite — case-study hero, 25% centre-height crop via object-fit (client call 2026-10-09) -->
<div class="project-media">
  <img src="assets/green-kite/hero.png" alt="" width="1024" height="1024" loading="lazy" decoding="async">
</div>

<!-- 06 White Crane Design Shop — new -->
<div class="project-media">
  <picture>
    <source type="image/webp" srcset="assets/wcd-shop/index-thumb.webp">
    <img src="assets/wcd-shop/index-thumb.png" alt="" width="1152" height="864" loading="lazy" decoding="async">
  </picture>
</div>
```

`.project-media img` targets the `img` correctly inside `<picture>` (the `<source>` element
generates no box), so the `aspect-ratio`/`object-fit` rules above need no `<picture>`-specific
variant.

---

## 7. Asset hygiene found while working in `assets/`

Flagged, not fixed — none of these are files the designer owns.

1. **`Draft image.png` — already fixed concurrently.** `index.html` shipped
   `poster="assets/White%20Crane%20Lab%20Hero/Draft%20image.png"` on the hero transition
   `<video>`. Midway through this session the engineer renamed it to
   `transition-poster.png` (same 678,307 bytes) and re-pointed `index.html`. **`index.html` is
   clean.**
   → **Still broken: `index v2.html`** retains the dead `Draft%20image.png` reference.
2. **`index v1.html` / `index v2.html` are stray copies at the site root.** `index v2.html` is
   byte-size-identical to `index.html` (61,397 B) and now stale/broken. `archived/` already
   exists for exactly this. Recommend removing both from the deployed root.
3. **Two byte-identical duplicate assets, 3.1 MB combined:**
   - `assets/repo-cover.png` ≡ `assets/heopet/master.png`
   - `Critique Council Header.png` (root) ≡ `assets/critique/header.png`
   → Point `Critique_Council_Live.html` / `Critique_Council_Thesis.html` at
   `assets/critique/header.png` and delete the root copy. For `repo-cover.png`, keep one and
   alias, or keep both only if the `og:image` is deliberately pinned separate from the HeoPet
   case study — but then name it `og-cover.png` so nobody assumes it is unique art.
4. **Filenames with literal spaces** still force `%20` encoding in `index.html`
   (`White%20Crane%20Lab%20Hero/Dark%20Mode.png`, `Light%20Mode.png`, `Transition.mp4`,
   `Transition-reverse.mp4`) and in the two Critique case studies. Combined with
   `assets/White Crane Lab Hero/` this is the source of the `Draft image.png` class of bug.
   → Recommend `assets/hero/` with `dark-mode.png`, `light-mode.png`, `transition.mp4`,
   `transition-reverse.mp4`, `transition-poster.png`.
5. **Three directory naming conventions** now coexist: `white_crane_studio` (snake),
   `White Crane Lab Hero` (spaced CamelCase), and the new `green-kite` / `wcd-shop` (kebab,
   as specified in the brief). The kebab-case new dirs are the ones to converge on. Recorded,
   not enforced.
6. **`assets/heopet/*.gif` are 0.9–1.2 MB each** (six files, ~6 MB). Unrelated to this spec,
   but if the HeoPet row ever animates in the index, budget accordingly.

---

## 8. Handoff checklist for studio-engineer

- [ ] §1.3 base rule — `#work .project` three-column added, bare `.project` untouched
- [ ] §1.4 641–899 px band block inserted **above** the existing 640 block
- [ ] §1.5 `#work` overrides added **inside** the existing 640 block (Trap B)
- [ ] §2 `.project-media` + `img` rules; `aspect-ratio:4/3` on both
- [ ] §3 hairline `1px var(--border)` / `border-radius:2px` on all six, opaque assets only
- [ ] §4 hover + `:focus-within`, 0.28 s `--ease-out`, no scale
- [ ] §4.1 reduced-motion override
- [ ] §5 DOM order `num → media → main` in all six rows
- [ ] §5 Row 04 glyph tile (no invented Pratt imagery)
- [ ] `alt=""` on every thumb; `loading="lazy"` on all six
- [ ] Visually verify in **both** themes, especially rows 02, 03 and 05 (see report)
- [ ] §7.1 confirm `index v2.html` dead poster is removed with the stale copies

**Designer's open items:** row 04 artwork (Pratt) is still owed; rows 05 and 06 need the
art director's eye before they are considered approved — see the report.

---

## 9. Generation log — full provenance (reproduce or re-roll from here)

**Server:** White Crane Studio Inference Server, `http://192.168.1.202:8100` (self-hosted, no key)
**Model:** `z_image_turbo` · **Endpoint used:** `POST /submit` → `GET /status/{id}` → `GET /result/{id}`
**Common params:** `image_size:"landscape_4_3"` · `num_images:2` · `output_format:"png"` ·
`sync_mode:false` · `enable_safety_checker:true` · `guidance_scale` and `num_inference_steps`
left at server defaults (**resolved to 9 steps**, `guidance_scale` default 0.0)

> ### ⚠️ Preset gotcha — `landscape_4_3` is **not** 4:3
> The preset named `landscape_4_3` returns **1152 × 896**, which is **1.2857 (9:7)**, not 1.3333.
> Every generated asset therefore needed a deterministic centre-crop to reach the §2 ratio.
> Do not assume the preset honours the ratio in §2 — crop explicitly, or request a custom
> `image_size:{width:1152,height:864}` if you want to skip the crop step.

### 9.1 Roll GK-1 — Green Kite, **REJECTED** (do not ship)

| | |
|---|---|
| `request_id` | `cdb5087c-b19d-4f2b-8018-94c8e0d0a8ea` |
| **seed** | **20261006** |
| size | `landscape_4_3` → 1152 × 896 |
| time | 442.41 s (2 images, 9 steps) |
| outputs | `…/output/471065137dd0443d996bc92a435284d0_batch_0.png` (roll A)<br>`…/output/7981b8de478b459d94d436c200152e4c_batch_1.png` (roll B) |

Prompt (verbatim):

> Flat vector brand identity mark for a community sustainability social enterprise. A single
> abstract diamond kite drawn as clean geometric flat shapes, tilted and lifting diagonally
> toward the upper right, with one slender curved tail line trailing below it. Strictly limited
> palette: deep leaf green #2B8A54 for the main kite body, a lighter mid green #4CAF78 for one
> faceted wing panel, on a **flat warm pale paper ground** #F2F6F2 that fills the entire frame.
> … *(full text is in the server `/result` record for this request_id)*

Negative prompt:

> text, letters, words, typography, watermark, logo, gradient, 3d render, bevel, drop shadow,
> glow, photograph, sky, clouds, person, human, hands, face, cluttered, busy background, frame,
> mockup, noise, grain

**Measured on both rolls — why they were rejected:**

| | roll A | roll B | Green Kite target |
|---|---|---|---|
| perimeter ground | `#FBF5E7` | `#FDF6E9` | `#F2F6F2` |
| ground hue (HSV) | **42.0°** | **39.0°** | **120.0°** |
| ground saturation | 0.080 | 0.079 | 0.016 |
| red/magenta pixels | 0.00 % | 0.00 % | — |
| mark (green) coverage | 7.25 % | 6.47 % | — |

> **My prompt was the bug.** I wrote *"flat **warm** pale paper ground #F2F6F2"* — but Green
> Kite's `#F2F6F2` is a *cool*, near-neutral mint-white (hue 120°, sat 0.016). The word "warm"
> contradicted the hex, and the model resolved the conflict toward the word: both rolls came
> back warm ivory, hue 39–42°.
>
> **The precise reason this must be rejected is brand world, not theme survival.** White Crane's
> own light surfaces are warm at hue 40–45° (`--black #F5F2EC` = 40°, `--paper-fixed #ECEAE4` =
> 45°, light `--surface #EDE9E0` = 41.5°). Roll A's ground at **hue 42.0°** is therefore the
> *same hue as White Crane's light background* and sits just **1.028:1** from `#F5F2EC`.
> The brief's one hard rule for this row was *must read as Green Kite, NOT as White Crane* —
> and a hue-matched ground does the exact opposite of that.
>
> *Correction I owe the record:* my first instinct was "ivory will vanish on the light band."
> That argument is wrong — Green Kite's **authentic** `#F2F6F2` is also only **1.110:1** from
> `#EDE9E0`, statistically the same as the rejected ivory (1.114:1). No near-white ground can
> separate from the light band; that is what the §3 hairline is for. Reject on **hue**, not on
> contrast.

### 9.2 Roll WC-1 — White Crane Design Shop, **ACCEPTED**

| | |
|---|---|
| `request_id` | `ed4ff036-18cb-4914-9567-764689876663` |
| **seed** | **20261007** |
| size | `landscape_4_3` → 1152 × 896 → cropped to **1152 × 864** |
| time | 386.63 s (2 images, 9 steps) |
| outputs | **roll A (chosen)** `…/output/9fb0381a58f74dc5a2849515b214f85c_batch_0.png`<br>roll B (alternate) `…/output/39a795d5ce09476581a92e206724eeee_batch_1.png` |

Prompt (verbatim, abridged only by ellipsis — full text in the server record):

> High-end apparel e-commerce product photograph, straight top-down flat-lay of one heavyweight
> boxy-crop hoodie neatly folded, centred on a smooth seamless pale warm-grey studio surface.
> Garment is deep charcoal cotton fleece with a soft brushed hand-feel, visible fine fabric
> weave, natural soft cotton creases and realistic seam and ribcuff detail. Across the chest
> panel a large abstract generative print: a flowing parametric line field breaking into angular
> fragmented shards, printed in bone white with a single hot coral-red accent, anime-inspired
> linework rhythm but completely non-representational. Soft diffused north window light from the
> upper left, gentle realistic contact shadow under the folded garment, true-to-life neutral
> colour, entire garment tack sharp. Shot on 85mm macro at f8, premium apparel catalogue
> quality. One garment only. No props, no mannequin, no person, no hands, no hanger, no
> packaging, no tags, no face, no character, no figure, no illustration of any person, no text,
> no letters, no words, no typography, no numbers, no brand marks, no logos, no watermark, no
> collage, no montage. Landscape 4 by 3.

Negative prompt:

> text, letters, words, typography, watermark, logo, face, person, human, hands, mannequin,
> extra limbs, distorted anatomy, melted features, anime character, collage, montage, cluttered,
> busy background, mockup frame, oversaturated, plastic sheen

**Why this prompt is written the way it is — read before re-rolling.** The brief forbids shipping
garbled text or melted anatomy. Both failure modes were designed *out* rather than hoped away:
the frame contains **no human** (a folded flat-lay, not a model → no anatomy to melt) and the
chest print is specified as **abstract generative linework, explicitly non-representational**
(no anime face → no features to garble, no lettering to smear). "Anime-inspired" is kept as a
*linework-rhythm* instruction only. If you re-roll, **keep those two constraints** — asking this
model for actual anime character art re-introduces the exact defect we are not allowed to ship.

**Measured — passes every objective gate:**

| metric | roll A (shipped) | roll B | gate |
|---|---|---|---|
| mean saturation | **0.041** | 0.033 | < 0.15 → photographic, not AI-collage |
| low-saturation pixels | 95.9 % | 94.9 % | high = neutral studio palette ✓ |
| perimeter ground | `#CDCDC9` | `#CECECD` | pale warm-grey studio surface ✓ |
| ground vs dark band `#212121` | **10.22:1** | — | unmistakable object ✓ |
| ground vs light band `#EDE9E0` | **1.30:1** | — | reads as a tile, frame assists ✓ |
| red-family pixels | 10.1 % | 12.5 % | the single coral accent, controlled ✓ |
| near-black / near-white | 1.03 % / 0.00 % | 0.76 % / 0.01 % | no clipping ✓ |
| background sd | 43.3 | — | gradient falloff → **reads as a photograph** |

Roll A chosen over roll B on hue coherence: A's hue mass sits 68 % in the two warm-neutral bins
(consistent single lighting), B spreads 30 % into green/blue bins (more chromatic noise). **This
is a metrics-based tie-break, not an aesthetic judgement — see §9.5.**

### 9.3 Roll GK-2 — Green Kite, **ACCEPTED after deterministic ground correction**

| | |
|---|---|
| `request_id` | `26b00ed0-5f18-4c59-a6bf-db8738ed29bf` |
| **seed** | **20261016** |
| size | `landscape_4_3` → 1152 × 896 → cropped to **1152 × 864** |
| time | 399.93 s (2 images, 9 steps) |
| outputs | **roll A (chosen)** `…/output/1a0186b312924f379978d44130d1fc52_batch_0.png`<br>roll B (alternate) `…/output/be36ca2254314929b2885885dd7287b6_batch_1.png` |

Prompt changes from GK-1 — **"warm" deleted everywhere**, and the paper described negatively:

> …COLOUR IS STRICTLY CONTROLLED. The entire background is a flat, cool, pale mint-white paper,
> hex #F2F6F2, with a faint cool green cast. It is emphatically **NOT cream, NOT ivory, NOT
> beige, NOT kraft, NOT eggshell, NOT warm yellow-white**. It is a cool near-white with green in
> it, the colour of pale mint. … The kite is **LARGE, filling roughly sixty percent of the frame
> height** … Only these three colours exist anywhere in the image. …
> *(full text in the server record for this request_id)*

Negative prompt adds: `cream background, ivory background, beige background, kraft paper, warm
white, eggshell, sepia, yellow tint, … tiny mark, low contrast`

**Result — improved but still not exact:**

| | GK-1 A/B | **GK-2 A (shipped)** | GK-2 B | target |
|---|---|---|---|---|
| ground hue | 42.0° / 39.0° | **78.8°** | 60.0° | **120.0°** |
| ground sat | 0.080 / 0.079 | 0.063 | 0.055 | 0.016 |
| mark coverage | 7.3 % / 6.5 % | 10.3 % | 9.1 % | — |
| ground sd | — | **0.67 / 0.69 / 0.92** | 0.54 / 0.38 / 0.41 | flat |
| darkest mark green | — | `#1D8751` (h 149°) | `#198758` | `#2B8A54` |

Reading: the roll moved **off** White Crane's 40–45° warm band (WC-collision distance improved
1.028 → 1.069) and the **mark's green is essentially on-brand** — `#1D8751` vs client `#2B8A54`
is Δ(14, 3, 3). But the paper field was still 41° of hue short: pale lime rather than pale mint.

### 9.4 Deterministic post-processing (not a re-roll) — and why it was safe

A diffusion model will not land an exact hex on a large flat field. Rather than burn another
7 minutes on a fourth roll, I used the fact that **the ground in GK-2 is a genuinely flat colour
field** — measured sd **0.67 / 0.69 / 0.92** across 9 360 probes, i.e. under 1/255 of variation.
On a field that flat, swapping the ground colour cannot band, cannot posterise and cannot reveal
a gradient, because there is no gradient there to break.

Two steps, both deterministic and reversible:

1. **Centre crop** 1152 × 896 → **1152 × 864** (exact 4:3), trimming 32 px of height as
   16 top / 16 bottom. Symmetric, so the optically centred mark does not shift.
2. **Ground snap** `#F7FCEC` → `#F2F6F2`, i.e. Δ(R,G,B) = **(−5, −6, +6)**, applied with a
   **smoothstep weight ramp on luma 140 → 217**:
   - ground (luma ≈ 250) → weight 1.0 → exact `#F2F6F2`
   - mark (luma ≈ 109) → weight 0.0 → **untouched**
   - anti-aliased edge pixels → proportional weight, so the blend tapers with the field and
     **no halo is produced**
   - 895 329 pixels full-weight, 46 839 edge-ramp pixels

**Verification after the fact — the correction landed and the mark is provably intact:**

| | before | **after** | target |
|---|---|---|---|
| ground colour | `#F7FCEC` | **`#F2F6F2`** | `#F2F6F2` ✓ exact |
| ground hue | 78.8° | **120.0°** | 120.0° ✓ exact |
| ground sd | 0.67/0.69/0.92 | 0.71/0.65/0.85 | still flat, no banding ✓ |
| red/magenta pixels | 0.00 % | **0.01 %** | not-White-Crane ✓ |
| alpha | 255 | **255** | fully opaque per §3.2 ✓ |
| dimensions | 1152×896 | **1152×864 = 1.3333** | 4:3 per §2 ✓ |

The WCD shop thumb received **the crop only** — no colour intervention of any kind. It is a
photograph and it stays faithful.

Uncorrected originals of all four chosen/alternate rolls are parked at
`studio/dist/work-index-thumbs/` (`*_ROLL-A_uncropped-original.png`,
`*_ROLL-B_uncropped-original.png`) so the art director can compare the pre-snap roll and reject
the correction if they disagree with it.

### 9.5 What these gates do **not** prove — read before shipping

I have no image input in this session; **I never looked at a single pixel of any roll.** Every
accept/reject above is a *measured* claim about colour distributions, coverage, opacity,
dimensions and hue mass — those are solid and repeatable. They cannot test the two things the
brief actually bans:

| gate | automated? | status |
|---|---|---|
| palette is Green Kite, not White Crane | ✅ measured | **pass, decisively** (0.01 % red hue mass, ground hue exact) |
| dimensions / ratio / opacity / no clipping | ✅ measured | **pass** |
| "reads as real product photography, not AI collage" | ◐ proxy (mean saturation 0.041, 95.9 % neutral, background gradient sd 43) | **strong signal, not proof** |
| garbled text anywhere in frame | ❌ not testable here | **unverified** — prompt bans all text; needs eyes |
| melted anatomy | ❌ not testable here | **structurally excluded** (no human in frame), still needs eyes |
| mark reads as a *kite* at 260 px | ❌ not testable here | **unverified** — needs eyes |
| composition / optical centring / general taste | ❌ not testable here | **unverified** — needs eyes |

**Therefore: rows 05 and 06 are METRICALLY CLEARED, NOT APPROVED.** The art director must open
both at ~260 px, in both themes, before they go live. Re-roll instructions: §9.1–§9.3 hold the
exact prompts and seeds; change only the seed, and for Green Kite **never restore the word
"warm."**
