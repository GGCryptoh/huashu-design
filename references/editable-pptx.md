# Editable PPTX Export: HTML Hard Constraints + Sizing Decisions + Common Errors

This doc covers the path of **using `scripts/html2pptx.js` + `pptxgenjs` to translate HTML element-by-element into real, editable PowerPoint text frames** — the only path `export_deck_pptx.mjs` supports.

> **Core prerequisite**: to go this route, the HTML must be written under the 4 constraints below from line one. **Not "write first, convert later"** — after-the-fact patching triggers 2–3 hours of rework (verified the hard way on the 2026-04-20 Options private board project).
>
> If visual freedom comes first (animation / web component / CSS gradients / complex SVG), switch to the PDF path (`export_deck_pdf.mjs` / `export_deck_stage_pdf.mjs`). **Don't** expect pptx export to give you both visual fidelity and editability — this is a physical constraint of the PPTX file format itself (see "Why the 4 Constraints Aren't a Bug But a Physical Constraint" at the end).

---

## Canvas Size: Use 960×540pt (LAYOUT_WIDE)

PPTX units are **inches** (physical size), not pixels. Decision rule: the body's computedStyle size must **match the presentation layout's inch dimensions** (±0.1", enforced by `html2pptx.js`'s `validateDimensions`).

### 3 Candidate Sizes Compared

| HTML body | Physical size | Matching PPT layout | When to pick |
|---|---|---|---|
| **`960pt × 540pt`** | **13.333″ × 7.5″** | **pptxgenjs `LAYOUT_WIDE`** | ✅ **Default recommendation** (modern PowerPoint 16:9 standard) |
| `720pt × 405pt` | 10″ × 5.625″ | Custom | Only when the user specifies the "legacy PowerPoint Widescreen" template |
| `1920px × 1080px` | 20″ × 11.25″ | Custom | ❌ Non-standard; fonts look unusually small once projected |

**Don't think of HTML dimensions as resolution.** PPTX is a vector document; body size determines **physical size**, not sharpness. An oversized body (20″×11.25″) won't make text crisper — it just makes pt sizes look smaller relative to the canvas, hurting projection and print.

### Three Equivalent Ways to Write `body`

```css
body { width: 960pt;  height: 540pt; }    /* clearest, recommended */
body { width: 1280px; height: 720px; }    /* equivalent, for px folks */
body { width: 13.333in; height: 7.5in; }  /* equivalent, for inch intuition */
```

Matching pptxgenjs code:

```js
const pptx = new pptxgen();
pptx.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch, no custom layout needed
```

---

## 4 Hard Constraints (Violations Throw Immediately)

`html2pptx.js` translates the HTML DOM element-by-element into PowerPoint objects. PowerPoint's format constraints projected onto HTML = the 4 rules below.

### Rule 1: No Bare Text Inside a DIV — Wrap It in `<p>` or `<h1>`-`<h6>`

```html
<!-- ❌ Wrong: text directly inside a div -->
<div class="title">Q3 revenue up 23%</div>

<!-- ✅ Right: text inside <p> or <h1>-<h6> -->
<div class="title"><h1>Q3 revenue up 23%</h1></div>
<div class="body"><p>New users are the main driver</p></div>
```

**Why**: PowerPoint text must live inside a text frame, and text frames map to HTML paragraph-level elements (p/h*/li). A bare `<div>` has no corresponding text container in PPTX.

**You also can't use `<span>` to carry primary text** — span is inline, it can't independently align as a text frame. Span is only allowed **nested inside p/h\*** for local styling (bold, color swap).

### Rule 2: No CSS Gradients — Solid Colors Only

```css
/* ❌ Wrong */
background: linear-gradient(to right, #FF6B6B, #4ECDC4);

/* ✅ Right: solid color */
background: #FF6B6B;

/* ✅ If you must have multi-color stripes, use flex children each with their own solid color */
.stripe-bar { display: flex; }
.stripe-bar div { flex: 1; }
.red   { background: #FF6B6B; }
.teal  { background: #4ECDC4; }
```

**Why**: PowerPoint's shape fill supports only solid and gradient-fill, but pptxgenjs's `fill: { color: ... }` only maps to solid. Going through PowerPoint's native gradient requires a different structure — the toolchain doesn't support it today.

### Rule 3: Background / Border / Shadow Only on DIVs, Never on Text Tags

```html
<!-- ❌ Wrong: <p> with a background color -->
<p style="background: #FFD700; border-radius: 4px;">Key point</p>

<!-- ✅ Right: outer div carries background/border, <p> only carries text -->
<div style="background: #FFD700; border-radius: 4px; padding: 8pt 12pt;">
  <p>Key point</p>
</div>
```

**Why**: in PowerPoint, a shape (rectangle / rounded rectangle) and a text frame are two separate objects. An HTML `<p>` only translates to a text frame; backgrounds, borders, and shadows belong to a shape — they must live on the **div that wraps the text**.

### Rule 4: No `background-image` on a DIV — Use an `<img>` Tag

```html
<!-- ❌ Wrong -->
<div style="background-image: url('chart.png')"></div>

<!-- ✅ Right -->
<img src="chart.png" style="position: absolute; left: 50%; top: 20%; width: 300pt; height: 200pt;" />
```

**Why**: `html2pptx.js` only extracts image paths from `<img>` elements; it doesn't parse CSS `background-image` URLs.

---

## Path A HTML Template Skeleton

One independent HTML file per slide; scopes isolated (avoiding CSS pollution that single-file decks suffer from).

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    width: 960pt; height: 540pt;           /* ⚠️ matches LAYOUT_WIDE */
    font-family: system-ui, -apple-system, "PingFang SC", sans-serif;
    background: #FEFEF9;                    /* solid color, no gradients */
    overflow: hidden;
  }
  /* DIVs carry layout / background / border */
  .card {
    position: absolute;
    background: #1A4A8A;                    /* background on the DIV */
    border-radius: 4pt;
    padding: 12pt 16pt;
  }
  /* Text tags carry only font styling — no background, no border */
  .card h2 { font-size: 24pt; color: #FFFFFF; font-weight: 700; }
  .card p  { font-size: 14pt; color: rgba(255,255,255,0.85); }
</style>
</head>
<body>

  <!-- Title block: outer div positions, inner text tags -->
  <div style="position: absolute; top: 40pt; left: 60pt; right: 60pt;">
    <h1 style="font-size: 36pt; color: #1A1A1A; font-weight: 700;">Use an assertion as the title, not a topic phrase</h1>
    <p style="font-size: 16pt; color: #555555; margin-top: 10pt;">Subtitle elaborates</p>
  </div>

  <!-- Content card: div carries background, h2/p carry text -->
  <div class="card" style="top: 130pt; left: 60pt; width: 240pt; height: 160pt;">
    <h2>Point one</h2>
    <p>Brief explanatory text</p>
  </div>

  <!-- List: use ul/li, don't hand-type • bullets -->
  <div style="position: absolute; top: 320pt; left: 60pt; width: 540pt;">
    <ul style="font-size: 16pt; color: #1A1A1A; padding-left: 24pt; list-style: disc;">
      <li>First point</li>
      <li>Second point</li>
      <li>Third point</li>
    </ul>
  </div>

  <!-- Illustration: use <img>, not background-image -->
  <img src="illustration.png" style="position: absolute; right: 60pt; top: 110pt; width: 320pt; height: 240pt;" />

</body>
</html>
```

---

## Common-Error Cheat Sheet

| Error message | Cause | Fix |
|---------|------|---------|
| `DIV element contains unwrapped text "XXX"` | bare text inside a div | wrap the text in `<p>` or `<h1>`-`<h6>` |
| `CSS gradients are not supported` | used linear/radial-gradient | switch to solid color, or use flex children for segments |
| `Text element <p> has background` | `<p>` tag has a background color | wrap with `<div>` to carry the background; let `<p>` carry only text |
| `Background images on DIV elements are not supported` | div used background-image | switch to an `<img>` tag |
| `HTML content overflows body by Xpt vertically` | content exceeds 540pt | trim content, shrink font size, or clip with `overflow: hidden` |
| `HTML dimensions don't match presentation layout` | body size doesn't match the pres layout | use `960pt × 540pt` body with `LAYOUT_WIDE`; or `defineLayout` for a custom size |
| `Text box "XXX" ends too close to bottom edge` | a large `<p>` sits < 0.5 inch from the body's bottom edge | move it up, leave bottom margin; the PPT bottom gets partially hidden anyway |

---

## Basic Workflow (PPTX in 3 Steps)

### Step 1: Write One Standalone HTML per Slide, Following the Constraints

```
MyDeck/
├── slides/
│   ├── 01-cover.html    # each file is a complete 960×540pt HTML
│   ├── 02-agenda.html
│   └── ...
└── illustration/        # images referenced by every <img>
    ├── chart1.png
    └── ...
```

### Step 2: Write a build.js That Calls `html2pptx.js`

```js
const pptxgen = require('pptxgenjs');
const html2pptx = require('../scripts/html2pptx.js');  // script from this skill

(async () => {
  const pres = new pptxgen();
  pres.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch, matching the HTML's 960×540pt

  const slides = ['01-cover.html', '02-agenda.html', '03-content.html'];
  for (const file of slides) {
    await html2pptx(`./slides/${file}`, pres);
  }

  await pres.writeFile({ fileName: 'deck.pptx' });
})();
```

### Step 3: Open and Verify

- Open the exported PPTX in PowerPoint / Keynote
- Double-clicking any text should let you edit directly (if it's an image, Rule 1 was violated)
- Verify overflow: each page should sit within the body, nothing clipped

---

## This Path vs. Other Options (When to Pick What)

| Need | Pick |
|------|------|
| Coworkers will edit the text inside the PPTX / sending to non-technical people who keep editing | **This doc's path** (editable, requires writing HTML under the 4 constraints from scratch) |
| Just for presenting / archiving, no further edits | `export_deck_pdf.mjs` (multi-file) or `export_deck_stage_pdf.mjs` (single-file deck-stage) — vector PDF |
| Visual freedom comes first (animation, web component, CSS gradients, complex SVG), willing to give up editability | **PDF** (same as above) — PDF is fidelity-preserving and cross-platform; better than an "image-only PPTX" |

**Never run html2pptx on HTML written for visual freedom** — empirically, visually-driven HTML has <30% pass rate, and patching the remainder page by page is slower than rewriting from scratch. That scenario should ship as PDF, not crammed into PPTX.

---

## Fallback: Visual Mock Already Exists But the User Insists on Editable PPTX

Occasionally you hit this case: you (or the user) already wrote a visually-driven HTML (gradients, web components, complex SVG, the works). PDF would be the right output, but the user explicitly says "no, it has to be an editable PPTX."

**Don't hammer `html2pptx` and pray it passes** — empirically, visually-driven HTML hits <30% pass rate on html2pptx; the other 70% errors or renders wrong. The correct fallback is:

### Step 1 · Surface the Limitation First (Transparent Communication)

In one breath, tell the user three things:

> "Your current HTML uses [list specifically: gradients / web components / complex SVG / ...]. Converting it directly to editable PPTX will fail. I have two options:
> - A. **Ship as PDF** (recommended) — 100% visual fidelity preserved; recipients can view and print, but cannot edit the text
> - B. **Rewrite an editable HTML based on the visual mock** (keeping the design decisions in colors / layout / copy, but reorganizing the HTML structure under the 4 hard constraints, **sacrificing** gradients, web components, complex SVG, and similar visual capabilities) → then export editable PPTX
>
> Which do you want?"

Don't sugarcoat option B — spell out **what gets lost**. Make the user own the trade-off.

### Step 2 · If the User Picks B: the AI Rewrites, Don't Ask the User To

The doctrine here: **the user gives design intent; you translate it into compliant implementation**. Don't make the user learn the 4 hard constraints and rewrite themselves.

Principles when rewriting:
- **Preserve**: color system (primary / secondary / neutrals), information hierarchy (title / subtitle / body / caption), core copy, layout skeleton (top-middle-bottom / two-column / grid), page rhythm
- **Downgrade**: CSS gradients → solid colors or flex segments, web components → paragraph-level HTML, complex SVG → simplified `<img>` or solid-color geometry, shadows → removed or weakened to nearly nothing, custom fonts → fall back to system fonts
- **Rewrite**: bare text → wrapped in `<p>` / `<h*>`, `background-image` → `<img>` tag, background/border on `<p>` → outer div carries it

### Step 3 · Produce a Before/After Diff (Transparent Delivery)

Once rewritten, give the user a before/after table so they know which visual details got simplified:

```
Original design → editable version adjustment
- Title-area purple gradient → primary color #5B3DE8 solid background
- Data card shadow → removed (2pt outline replaces it for separation)
- Complex SVG line chart → simplified to <img> PNG (generated by screenshotting the HTML)
- Hero-area web component animation → static first frame (web components can't be translated)
```

### Step 4 · Export & Dual-Format Delivery

- `editable` HTML → run `scripts/export_deck_pptx.mjs` for the editable PPTX
- **Strongly recommend keeping** the original visual mock → run `scripts/export_deck_pdf.mjs` for a high-fidelity PDF
- Deliver both to the user: PDF of the visual mock + editable PPTX, each playing its own role

### When to Refuse Option B Outright

In some cases the rewrite cost is too high and you should talk the user out of editable PPTX:
- The HTML's core value is animation or interaction (rewriting leaves only a static first frame, losing 50%+ of the information)
- More than 30 pages, rewrite cost exceeds 2 hours
- The visual design depends heavily on precise SVG / custom filters (the rewrite would barely resemble the original)

In these cases tell the user: "This deck is too expensive to rewrite — I recommend PDF over PPTX. If the recipient really insists on pptx format, accept that the visuals will be drastically plainer — want to switch to PDF?"

---

## Why the 4 Constraints Aren't a Bug But a Physical Constraint

These 4 rules aren't `html2pptx.js`'s author being lazy — they are **constraints of the PowerPoint file format (OOXML) itself** projected onto HTML:

- Text in PPTX must live inside a text frame (`<a:txBody>`), which corresponds to paragraph-level HTML elements
- A PPTX shape and a text frame are two separate objects; you can't draw a background and write text on the same element
- PPTX shape fill has limited gradient support (only certain preset gradients; no arbitrary-angle CSS gradients)
- A PPTX picture object must reference a real image file, not a CSS property

Once you understand this, **stop expecting the tool to get smarter** — the HTML must adapt to the PPTX format, not the other way around.
