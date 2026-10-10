# DESIGN.md — DELTA

**File:** `studio/Website/design/DESIGN.delta.md`
**Author:** studio-designer · **Date:** 2026-10-06
**Form:** delta only. Nothing here is a rewrite; sections not named below are unchanged.
**Apply to:** the project `DESIGN.md` (the site's brand contract) and to
`studio/Website/wc-tokens.css`, which is already carrying two of these three tokens.

> **Reconciliation note.** `wc-tokens.css` shipped — before this delta landed — with
> `--red-text` and `--crane-accent` already defined, and its header comment says *"reconcile
> with DESIGN.md when the designer's value lands."* This document **is** that reconciliation.
> I re-measured every value the engineer shipped, independently, with the WCAG 2.1
> relative-luminance formula. **All four of their contrast claims verified to two decimal
> places.** I am ratifying their numbers rather than inventing competing ones, and adding the
> parts that were still missing: the selector-by-selector migration, the reason the hue drift
> is mandatory, the accent escalation ladder, and the handwriting register — including two
> violations that had already spread past `index.html`.

---

## D1 · NEW TOKEN — `--red-text` (small TEXT only)

### Status: **RATIFIED as shipped.** No value change.

```css
:root{
  --red:#E03030;          /* unchanged — rules, glyph marks, dots, pills, large display */
  --red-solid:#C22B2B;    /* unchanged — filled button / pill backgrounds */
  --red-text:#F2563F;     /* NEW · dark · small text only */
}
html[data-theme="light"]{
  --red:#C23B22;          /* unchanged */
  --red-text:#B93A20;     /* NEW · light · small text only */
}
```

### D1.1 Measured contrast (designer's independent re-measurement)

| Foreground | on `--black` `#1C1B18` | on `--surface` `#212121` | AA small (4.5) | AA large (3.0) |
|---|---|---|---|---|
| `--red` `#E03030` *(current, small labels)* | **3.80:1** | **3.55:1** | ✗ fail both | ✓ |
| **`--red-text` `#F2563F`** | **5.07:1** | **4.74:1** | ✓ **pass both** | ✓ |
| `--red-solid` `#C22B2B` | 3.01:1 | 2.82:1 | ✗ | borderline |

| Foreground | on light `--black` `#F5F2EC` | on light `--surface` `#EDE9E0` | AA small (4.5) |
|---|---|---|---|
| `--red` `#C23B22` *(current)* | **4.77:1** | **4.40:1** | ✓ on black / **✗ fail on surface** |
| **`--red-text` `#B93A20`** | **5.09:1** | **4.70:1** | ✓ **pass both** |

### D1.2 Direct answer: does light need a separate value? **Yes.**

The working assumption was that light `--red #C23B22` on `--black #F5F2EC` ≈ 4.8:1, therefore
light is already fine and only dark needs a token. That is true **on `--black`** — and it is
the wrong ground for the part of the site that matters most here.

`#work` and `#background` are `.band-surface`, which is `#EDE9E0` in light mode, not `#F5F2EC`.
On that ground light `--red` measures **4.40:1 — a fail for small text** — and the work-index
`.work-scan .scan-label` (10.5 px mono, `letter-spacing:.12em`, uppercase) sits precisely
there. Uppercase small-tracked mono at 10.5 px is among the least legible text on the page, on
the one band where the light-mode red does not clear AA.

So the split is **not** "dark is broken, light is fine." The split is:
**`--red` fails on `--surface` in both themes**, and `--surface` is where the work index lives.
Light needs its own small-text value for the same structural reason dark does. `#B93A20`
supplies it at 4.70:1.

### D1.3 Why `#F2563F` sits +3.3° warmer than `--red` — and why that is required

OKLCH decomposition of the reds:

| Token | L | C | H |
|---|---|---|---|
| `--red` `#E03030` | 0.100 | 0.196 | 12.5° |
| `--red-solid` `#C22B2B` | 0.063 | 0.121 | 12.6° |
| **`--red-text` `#F2563F`** | 0.160 | 0.251 | **15.8°** |
| light `--red` `#C23B22` | 0.070 | 0.120 | 15.4° |
| light `--red-text` `#B93A20` | 0.061 | 0.102 | 15.8° |

The dark text-red drifted from 12.5° to 15.8°. That looks like sloppiness. It is not — it is
forced. Lifting a red to 4.5:1 on `#212121` while holding its hue at 12.5° is not reachable in
sRGB; every hue-locked candidate that gets close on `#1C1B18` still dies on the work band:

| Hue-locked candidate (H≈12.5°) | on `#1C1B18` | on `#212121` | verdict |
|---|---|---|---|
| `#ED4537` | 4.52 ✓ | **4.22 ✗** | fails work band |
| `#EE4A3D` | 4.66 ✓ | **4.36 ✗** | fails work band |
| `#F04A3A` | 4.71 ✓ | **4.41 ✗** | fails work band |
| `#EF4E40` | 4.78 ✓ | **4.47 ✗** | fails work band |
| **`#F2563F` (shipped, H=15.8°)** | **5.07 ✓** | **4.74 ✓** | **passes both** |

**Rule to record:** *a small-text red that clears 4.5:1 on `#212121` must be ~3° warmer and
carries more chroma than the structural red. Accept the drift; it is the price of the
contrast floor on the work band. Do not "correct" `--red-text` back to the `--red` hue.*

3.3° is below the threshold at which a reader names two colours as different reds, and the
light pair already lives at 15.4°/15.8° — so `#F2563F` actually makes the dark theme's
structural/text relationship *match* the light theme's. The drift unifies the system.

### D1.4 Binding scope — the part that stops the token rotting

> `--red-text` may appear **only** in a `color:` declaration on live text smaller than
> 18.66 px bold / 24 px regular.
> It must **never** appear as `background`, `border-color`, `outline-color`, `fill`,
> `text-decoration-color`, `box-shadow` colour, or in any `::before`/`::after` mark.
> `--red` keeps 100% of those jobs. `--red-solid` keeps filled-button and pill grounds.

This is why the token is called `-text` and not `-accessible`: the moment it leaks onto a hairline
the palette has three reds again and the calibration is lost.

### D1.5 Migration table — every `color:var(--red)` in the live site

Type sizes read off `index.html` / `About.html`; large-text threshold = ≥18.66 px at ≥700 wt,
or ≥24 px at any weight.

**→ change to `var(--red-text)`** (all currently fail AA small text):

| Selector | Size / face | Band | dark now | light now |
|---|---|---|---|---|
| `.work-scan .scan-label` | 10.5 px mono | **surface** | 3.55 ✗ | **4.40 ✗** |
| `.ov-sign` | 12 px mono | black | 3.80 ✗ | 4.77 ✓ |
| `.hero-status` | 12 px mono | black | 3.80 ✗ | 4.77 ✓ |
| `.exp-meta` | 11.5 px mono | black | 3.80 ✗ | 4.77 ✓ |
| `.footer-tagline` | 11.5 px mono | black (footer) | 3.80 ✗ | 4.77 ✓ |
| `.coda-detail summary:hover` | 11.5 px mono | **surface** | 3.55 ✗ | **4.40 ✗** |

The first and last of these fail in **both** themes — they are the two that must not be
skipped if anyone does this work half-heartedly.

**→ keep `var(--red)`** (large text, or non-text):

- *Large text, 3:1 met:* `h1 .line:nth-child(2)` ("Creative", display) · `.lede-strong .accent`
  (30–40 px/700) · `a.project-title:hover` (22.4–28 px/600) · `.coda-detail[open] summary::before`
- *Decorative, `aria-hidden`, zero contrast obligation:* `.project-num`, `.exp-num`,
  `.project-glyph` (all `opacity:.14`) · `.ov-mark`
- *Non-text marks and decoration — 3:1 target, all met at 3.80:1:* `.eyebrow .dot` ·
  `.work-group .dot` · `.marquee-item .dia` · `.hero-status .pulse-dot` · `.type-caret` ·
  `.scroll-cue .mouse::after` · `.exp li::before` · `.exp-sub .sub::before` ·
  `.coda-cols h4::before` · `.ov-sign .ov-dia` · `.connect-link .link-arrow` ·
  `.nav-links a::after` · `.work-scan a:hover` border · all `text-decoration-color` ·
  `a:focus-visible / button:focus-visible` outline
- *Text on a red ground, not red text:* `.cat-pill.c-red` — `--paper-fixed #ECEAE4` on
  `--red-solid #C22B2B` = **4.75:1 ✓**, leave alone.
- `.connect-link .link-arrow` `--red` on `--maroon #2A1613` = 3.79:1, non-text, ✓ passes 3:1.

### D1.6 One adjacent defect found, not in scope — flagged for a decision

```css
::selection{background:var(--red);color:var(--paper-fixed)}   /* #ECEAE4 on #E03030 = 3.77:1 */
```

Selected copy drops to **3.77:1** (light: `#1E1C18` on `#C23B22` = 3.19:1). Selection is
transient and emphatic rather than reading copy, so reasonable people leave it — but it is the
one place the site puts body-size text on `--red` as a ground. If you want it fixed, the
cheapest correct move is `::selection{background:var(--red-solid)}` → 4.75:1, which also makes
selection match the CTA fill. Designer recommends the change; not applied here.

---

## D2 · TOKENIZE LOGO RED — `--crane-accent`

### Status: **RATIFIED as shipped** — `--crane-accent` resolves to `--red`, per theme.

```css
:root{                        --crane-accent:#E03030; }   /* = --red, dark  */
html[data-theme="light"]{     --crane-accent:#C23B22; }   /* = --red, light */
.crane-accent{fill:var(--crane-accent)}
```

**Binding rule (do not "simplify" this back):** the wingtip must carry
`class="crane-accent"`; `fill="var(--crane-accent)"` is **not** reliable as an SVG presentation
attribute. The fill belongs in CSS. No page, and no `icon.png` / `favicon.svg` /
`apple-touch-icon.png` export, may contain a literal `fill="#FF0000"`.
*Verified clean in `index.html` as of this delta.*

### D2.1 Match `--red` exactly, or stay deliberately hotter? — the actual argument

Measured, all on `--black #1C1B18`:

| Fill | OKLCH | contrast | reads as |
|---|---|---|---|
| `#FF0000` (current bug) | L 0.133 · **C 0.305** · H 11.7° | 4.31:1 | gamut-corner default |
| `--red #E03030` | L 0.100 · C 0.196 · H 12.5° | 3.80:1 | the calibrated brand red |
| `--red-solid #C22B2B` | L 0.063 · C 0.121 · H 12.6° | 3.01:1 | pressed / filled state |

The diagnostic that settles it: **`#FF0000` is only 0.8° away from `--red` in hue.** It is not a
different red — it is the *same* red at the maximum chroma the sRGB gamut can emit, with no
calibration applied. That is precisely why it sits badly: it is too close to the brand red to
read as a deliberate second accent, and far enough off in chroma and luminance to read as
unedited. It is a default, visibly on display next to two colours that were actually chosen.
Three near-reds that nobody differentiated on purpose read as drift; two reds with distinct
jobs read as a system.

> **Decision — `--crane-accent` matches `--red` exactly.** The mark's accent is the brand red,
> not a fourth red. "Deliberately hotter" only earns its keep when it is far enough away to be
> *legible as intent* (a 20–40° hue shift, or a switch of family entirely). A 0.8° nudge at
> 1.55× the chroma is the worst of both: unexplained, unfixed, and unrepeatable by anyone else.
> The crane's seven polygons are `currentColor`; the wingtip is the brand red. Two colours in
> the mark, two reds in the system.

### D2.2 The escalation ladder (pre-authorised, so nobody reaches for `#FF0000` again)

The one real argument for heat is size: the wingtip is 26 px in the nav and 18 px in the
footer, and a small saturated shape on charcoal can read slightly inert at `--red`. Measured,
that risk does not materialise — the wingtip is **non-text**, needs 3:1, and `#E03030` clears it
at 3.80:1; it is never the sole carrier of meaning (the "White Crane Labs" wordmark is always
beside it).

If visual review says the 18 px footer mark is too quiet, escalate **within the token**, never
back to a literal:

| Option | value | on `#1C1B18` | note |
|---|---|---|---|
| Locked default | `#E03030` | 3.80:1 | **ship this** |
| If punch needed | `#E8402F` | 4.27:1 | one notch hotter, still in family |
| Ceiling | `#EA4231` | 4.36:1 | do not go past here — next stop is `#FF0000` |
| Rejected | `#FF0000` | 4.31:1 | higher contrast, *worse* result — see D2.1 |

Note the trap this table documents: `#FF0000` has **better** raw contrast (4.31) than the
recommended `#E8402F` (4.27). Contrast alone does not justify it. Chroma discipline does.

### D2.3 Theme behaviour

`--crane-accent` tracks `--red` per theme, so the mark re-inks with the page. Two exceptions to
keep in mind — both currently correct:

- **`--maroon` `#2A1613` does not flip in light mode** (intentional, documented in
  `wc-tokens.css`). `#connect` therefore stays a dark closer, and no crane mark is placed on it.
  If a future crane mark is ever dropped onto `#connect`, it needs `--red`, not `--red-text`,
  and a `currentColor` body that is explicitly cream.
- **`@media print`** has no `data-theme`, so the mark prints with `#E03030` on white — fine for
  a logo. Do not add a print override; `wc-tokens.css` already forces text to black.

---

## D3 · HANDWRITING REGISTER — **REVERSED 2026-10-08.** Caveat is a human-trace motif and MAY be used at statement scale

> ### ⛔ DO NOT RE-MIGRATE. This section used to mandate the exact opposite of what the site now ships.
> The client, Ken Bai, owns this portfolio and overruled the demotion:
> *"go with option c for full revert of handwritten font. it adds a touch of human that fits with my concept."*
> His concept — **"human in the loop"** — outranks typographic discipline on his own site. Any agent
> tempted to "fix" Caveat back to Grotesk at statement scale is re-litigating a closed client decision.

### D3.1 The current rule (client reversal, 2026-10-08 — supersedes everything below it)

> **Caveat (`--fh`) is an intentional human-trace motif, not a banned display face.**
> It is aligned to the site's "human in the loop" positioning and is permitted in **two registers**:
>
> - **Statement scale** — `.ov-statement` (*"Great creative environments are designed with intent."*),
>   `.about-title` (*"Bridging creativity and technology…"*), `About.html` `.manifesto`
>   (*"AI is a tool. Humans still direct everything that matters."*) — at
>   `clamp(2.1rem,4vw,3rem)` ≈ **33.6–48 px**, `font-weight:600`, `line-height:1.25`, `letter-spacing:0`.
> - **Signature register** — `.ov-sign` · `.footer-tagline` · `.marginal` (≤16 px), 13 px, `font-weight:600`, `color:var(--red-text)`.
>
> **Hard rule that survives the reversal:** every Caveat instance **pins `font-weight:600` explicitly**.
> The `<link>` requests `family=Caveat:wght@600` — trimmed 2026-10-08 from `wght@500;600;700`
> because **600 is the only weight any rule on the site uses** (500/700 requests fetched two
> static faces nobody consumed). The trim was re-verified headless on both Caveat pages: the 600
> face still reports `status:"loaded"` and renders (proof: `studio/audit-out/icons-after.json`,
> `caveat.*`). An unpinned 400 remains *not in the loaded set* and would silently substitute a
> face that was never fetched. Do not remove the pin; if a future rule needs another weight,
> widen the `<link>` first — one or the other, never neither (D3.2 §1).

### D3.2 What was learned from the demote-then-revert cycle (guidance, not prohibition)

Kept short, because these are now *usage guidance*, not bans — the font stays; put it where it is strongest:

1. **The weight-400 substitution trap is real and silent** (it bit during the migration). Any future
   Caveat user must pin 600, or widen the `<link>` request — one or the other, never neither.
2. **Long handwritten sentences are the weak point.** Caveat at statement scale carries short,
   aphoristic one/two-line statements well (all three shipped statements are short). It degrades on
   multi-sentence prose — which lives in Inter/Space Grotesk and should stay there. If a future
   statement is long, shorten the sentence before assuming the font is wrong.
3. **Cascade trap (shipped defense):** `.ov-statement` also carries `.lede-strong` in the markup, and
   the display guardrail below re-asserts `.lede-strong{font-family:var(--fd)}`. The shipped rule is
   therefore written as `.lede-strong.ov-statement{…}` — specificity **0-2-0** — so Caveat wins the
   cascade on specificity, not source placement. Do not "simplify" that selector back to `.ov-statement`;
   at 0-1-0 it loses to the guardrail and silently renders Grotesk.
4. Scarcity was the original argument and it was not *wrong* — it just isn't the client's priority on
   his own site. Two display voices, one loosening every line, is now the accepted cost of the
   human-trace concept. Note this trade-off before applying the same reversal to a *client* project
   without their explicit sign-off.

### D3.3 Shipped CSS (this is what the code does — verified 2026-10-08)

```css
/* index.html — statement scale (restored from index v1/v2 originals) */
.about-title{font-family:var(--fh);font-weight:600;font-size:clamp(2.1rem,4vw,3rem);line-height:1.25;letter-spacing:0;text-align:center;max-width:26ch;margin:0 auto 1rem}
.lede-strong.ov-statement{border-left:none;padding-left:0;max-width:none;margin-bottom:0;font-family:var(--fh);font-weight:600;font-size:clamp(2.1rem,4vw,3rem);line-height:1.25;letter-spacing:0}

/* index.html — signature register (kept from the migration; --red-text + pinned 600 survive) */
.ov-sign{font-family:var(--fh);font-weight:600;font-size:13px;color:var(--red-text);…}
.footer-tagline{font-family:var(--fh);font-weight:600;font-size:13px;color:var(--red-text)}

/* About.html — same Caveat-at-large recipe, own alignment */
.manifesto{font-family:var(--fh);font-weight:600;color:var(--white);font-size:clamp(2.1rem,4vw,3rem);line-height:1.25;letter-spacing:0;text-align:left;max-width:26ch;margin:0 0 2.5rem}

/* Guardrail kept for TRUE display type — the two 0-2-0/0-1-0 Caveat statements out-rank it by design */
h1,h2,.lede-strong,.connect-headline,.project-title,.exp-role{font-family:var(--fd)}
```

`Green_Kite_Brand_Identity.html` — still does not load or use Caveat; the client brand world is
unaffected by either direction of this decision. Stale copies (`index v1/v2.html`, `archived/*`) are
out of scope and keep whatever register they were snapshotted with.

### D3.4 History — the superseded ruling (kept so nobody rediscovers it as "new")

2026-10-06, art director's ruling: Caveat demoted to signature-only (≤16 px), banned above 24 px;
three statement blocks migrated to Space Grotesk at the `.lede-strong` scale —
`index.html` `.ov-statement`, `index.html` `.about-title`, `About.html` `.manifesto`. Rationale then:
handwriting's meaning is proportional to scarcity; a signature used for headings is no longer a
signature. Two implementation traps surfaced (weight-400 substitution; the `.lede-strong` cascade
trap) — both live on in D3.2 above. **The ruling stood for two days and was reversed by the client;
the migration was fully reverted, including all three blocks.**

### D3.5 Current type register (for the DESIGN.md table)

| Register | Face | Where | Notes |
|---|---|---|---|
| Display | Space Grotesk `--fd` | `h1`–`h4` (incl. Work `h2`), `.lede-strong` (non-statement), `.connect-headline`, `.project-title`, numerals, buttons | guardrail enforces |
| Body | Inter `--fb` | `.lede`, `.project-desc`, `.exp li`, prose | — |
| Data | JetBrains Mono `--mono` | eyebrows, chips, pills, roles, meta, `.footer-copy` | — |
| **Statement (human trace)** | **Caveat `--fh` 600** | **`.ov-statement` · `.about-title` · `.manifesto`** | **33.6–48 px; short aphorisms only (D3.2 §2)** |
| **Signature** | **Caveat `--fh` 600** | **`.ov-sign` · `.marginal` · `.footer-tagline`** | **≤16 px, `--red-text`** |
| Client exception | Green Kite keeps its own stack (`--serif` Instrument Serif, `--mono` DM Mono) inside its own brand world — never inherits the White Crane register table |

### D3.6 Archived rationale for the demotion (preserved; no longer binding)

<details><summary>The 2026-10-06 argument, for context only — see D3.4.</summary>

Handwriting is a human trace, and its meaning is proportional to its scarcity. A single
sign-off in Caveat at the foot of a manifesto reads as a person having been in the room — the
one place in a machine-typeset page where a hand is visibly present, which is exactly the
claim the site is making when it says *human in the loop, always*. Put the same face at
33–48 px carrying the argument itself and the signal inverts: the gesture stops being a trace
and becomes a costume, handwriting reads as casual and decorative rather than authored, and —
the practical cost — Caveat and Space Grotesk end up competing for display duty with the two
faces pulling in opposite directions, one loosening every line it touches while the other is
trying to be the page's structural voice. Demoting Caveat to a register with a 16 px ceiling
does three things at once: it gives Space Grotesk the display slot outright, so the site has
one typographic voice instead of two; it keeps the handwriting rare enough that when it appears
you notice; and it makes the sign-off land as a signature rather than a headline that happens
to be scruffy. Scarcity is the whole mechanism — a signature used for headings is no longer a
signature.

</details>
