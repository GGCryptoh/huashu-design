# Slide Decks: HTML Slide Deck Spec

Building slide decks is a high-frequency design job. This doc explains how to do HTML slide decks well — from architecture choice and per-slide design through to the full PDF/PPTX export path.

**What this skill covers**:
- **HTML presentation (the base artifact, always the default, always required)** → one HTML per slide + `assets/deck_index.html` aggregator, keyboard paging in the browser, fullscreen presenting
- HTML → PDF export → `scripts/export_deck_pdf.mjs` / `scripts/export_deck_stage_pdf.mjs`
- HTML → editable PPTX export → `references/editable-pptx.md` + `scripts/html2pptx.js` + `scripts/export_deck_pptx.mjs` (requires HTML written against 4 hard constraints)

> **⚠️ HTML is the base, PDF/PPTX are derivatives.** Whatever the final delivery format, you **must** first build the HTML aggregator (`index.html` + `slides/*.html`) — that is the "source" of the slide deck. PDF/PPTX are one-command snapshots exported from the HTML.
>
> **Why HTML first**:
> - Best for live presenting (projector / screen-share goes straight to fullscreen, keyboard paging, no dependency on Keynote/PPT software)
> - During development each page can be opened standalone (just double-click) to verify, no need to re-run the export every time
> - It's the only upstream for PDF/PPTX export (avoids the "exported then realized HTML needs changes, then re-export" death loop)
> - Delivery can be "HTML + PDF" or "HTML + PPTX" as a pair — receiver picks whichever they prefer
>
> 2026-04-22 moxt brochure verified: after building 13 HTML pages + the index.html aggregator, `export_deck_pdf.mjs` exported the PDF in one shot, zero changes needed. The HTML version is itself a deliverable you can present directly from the browser.

---

## 🛑 Confirm the delivery format before starting (the hardest checkpoint)

**This decision comes before "single file vs multi file".** 2026-04-20 options private-board project verified: **not confirming delivery format before you start = 2-3 hours of rework.**

### Decision tree (HTML-first architecture)

Every delivery starts from the same HTML aggregator (`index.html` + `slides/*.html`). The delivery format only changes the **HTML authoring constraints** and the **export command**:

```
[ALWAYS DEFAULT · REQUIRED] HTML aggregator (index.html + slides/*.html)
   │
   ├── Browser presenting only / local HTML archive  → done at this point, max visual freedom
   │
   ├── Also need PDF (print / share / archive)        → run export_deck_pdf.mjs, one shot
   │                                                     HTML authoring is free, no visual constraints
   │
   └── Also need editable PPTX (teammate edits text)  → write HTML against 4 hard constraints from line one
                                                         run export_deck_pptx.mjs, one shot
                                                         sacrifices gradients / web components / complex SVG
```

### Kickoff script (copy-paste ready)

> Whether the final delivery is HTML, PDF, or PPTX, I'll first build an HTML aggregator you can flip through and present in the browser (`index.html` with keyboard paging) — that's always the default base artifact. On top of that I'll ask whether you also need a PDF / PPTX snapshot.
>
> Which export format do you need?
> - **HTML only** (present/archive) → full visual freedom
> - **Also PDF** → same as above, plus one export command
> - **Also editable PPTX** (teammate edits text in PPT) → I must write HTML against 4 hard constraints from line one, sacrificing some visual capability (no gradients, no web components, no complex SVG).

### Why "if you want PPTX you must follow the 4 hard constraints from the start"

PPTX editability depends on `html2pptx.js` translating the DOM element-by-element into PowerPoint objects. It needs **4 hard constraints**:

1. body fixed at 960pt × 540pt (matches `LAYOUT_WIDE`, 13.333″ × 7.5″, NOT 1920×1080px)
2. All text wrapped in `<p>`/`<h1>`-`<h6>` (no bare text in div, no `<span>` carrying primary text)
3. `<p>`/`<h*>` themselves cannot have background/border/shadow (move those to a wrapping div)
4. `<div>` cannot use `background-image` (use an `<img>` tag)
5. No CSS gradients, no web components, no complex decorative SVG

**This skill's default HTML enjoys high visual freedom** — heavy span use, nested flex, complex SVG, web components (like `<deck-stage>`), CSS gradients — **almost none of it naturally passes the html2pptx constraints** (in practice, throwing visual-driven HTML at html2pptx gives < 30% pass rate).

### Two real paths, cost comparison (2026-04-20 real war story)

| Path | Approach | Result | Cost |
|------|----------|--------|------|
| ❌ **Write HTML freely first, retrofit PPTX later** | Single-file deck-stage + heavy SVG/span decoration | To get editable PPTX, only two options remain:<br>A. Hand-write hundreds of lines of pptxgenjs with hardcoded coordinates<br>B. Rewrite all 17 HTML pages into Path A format | 2-3 hours of rework, and the hand-written version has **perpetual maintenance cost** (change one word in HTML, manually re-sync the PPTX) |
| ✅ **Write to Path A constraints from step one** | One HTML per slide + 4 hard constraints + 960×540pt | One command produces 100% editable PPTX, and you can also fullscreen-present in the browser (Path A HTML is just standard browser-playable HTML) | Spend 5 extra minutes when writing HTML thinking "how does this text fit inside `<p>`", zero rework |

### What about mixed delivery

User says "I want HTML for presenting **and** editable PPTX" — **this isn't mixed**, the PPTX requirement subsumes the HTML one. HTML written to Path A is itself fullscreen-presentable in the browser (just add the `deck_index.html` aggregator). **No extra cost.**

User says "I want PPTX **and** animations / web components" — **this is a real contradiction.** Tell the user: editable PPTX means giving up these visual capabilities. Make them choose, don't quietly resort to hand-writing pptxgenjs (that becomes a perpetual maintenance debt).

### What if you only learn PPTX is needed afterward (emergency fallback)

Rare case: HTML is already written and you only then discover PPTX is required. Use the **fallback flow** (full details in `references/editable-pptx.md`, "Fallback: existing visual draft but user insists on editable PPTX"):

1. **First choice: ship a PDF instead** (100% visual fidelity, cross-platform, receiver can view and print) — if the receiver's real need is "presenting/archiving", PDF is the best delivery
2. **Second choice: have AI rewrite an editable HTML using the visual draft as a blueprint** → export editable PPTX — preserves the color / layout / copy design decisions, sacrifices gradients, web components, complex SVG, etc.
3. **Not recommended: rebuild via hand-written pptxgenjs** — every position, font, and alignment must be hand-tuned, maintenance cost is high, and every word change in HTML later requires another manual re-sync

Always present the choices to the user and let them decide. **Never reflexively start hand-writing pptxgenjs** — that's the last-resort fallback.

---

## 🛑 Before batch production: build a 2-page showcase to lock the grammar

**As soon as the deck is ≥ 5 pages, never go straight from page 1 to the last page.** The right order, validated on the 2026-04-22 moxt brochure:

1. Pick **the 2 most visually different page types** and build them as a showcase first (e.g. "cover" + "emotion/quote page", or "cover" + "product showcase page")
2. Screenshot and have the user confirm the grammar (masthead / fonts / color / spacing / structure / Chinese-English bilingual ratio)
3. Once the direction is approved, batch out the remaining N-2 pages, each reusing the established grammar
4. After everything is done, assemble the HTML aggregator + PDF / PPTX derivatives together

**Why**: writing 13 pages straight through → user says "wrong direction" = 13 reworks. Doing a 2-page showcase first → wrong direction = 2 reworks. Once the visual grammar is locked, the decision space for the remaining N pages collapses to "how do I fit the content in".

**Showcase page selection principle**: pick the two pages with the most different visual structure. If those two pass = every intermediate page also passes.

| Deck type | Recommended showcase page pair |
|-----------|--------------------------------|
| B2B brochure / product launch | Cover + content page (philosophy/emotion page) |
| Brand reveal | Cover + product feature page |
| Data report | Big data chart page + analysis-conclusion page |
| Tutorial / courseware | Chapter cover + a specific knowledge-point page |

---

## 📐 Publication grammar template (moxt-tested, reusable)

Fits B2B brochure / product launch / long-report decks. Reuse this structure on every page = 13 pages visually consistent, zero rework.

### Per-page skeleton

```
┌─ masthead (top strip + horizontal rule)────┐
│  [logo 22-28px] · A Product Brochure                Issue · Date · URL │
├──────────────────────────────────────────┤
│                                          │
│  ── kicker (green short bar + uppercase) │
│  CHAPTER XX · SECTION NAME               │
│                                          │
│  H1 (Chinese Noto Serif SC 900)          │
│  Keyword in brand primary color          │
│                                          │
│  English subtitle (Lora italic)          │
│  ─────────── divider ──────────          │
│                                          │
│  [content: 60/40 two-col / 2x2 grid / list] │
│                                          │
├──────────────────────────────────────────┤
│ section name                     XX / total │
└──────────────────────────────────────────┘
```

### Style conventions (copy-paste ready)

- **H1**: Chinese Noto Serif SC 900, size 80-140px depending on information density, keyword in brand primary color (don't pile color across the whole line)
- **English sub**: Lora italic 26-46px, brand signature words (e.g. "AI team") bold + primary color italic
- **Body**: Noto Serif SC 17-21px, line-height 1.75-1.85
- **Accent highlight**: bold + primary color on keywords inside body text, max 3 per page (more and they lose their anchoring effect)
- **Background**: warm beige #FAFAFA + very faint radial-gradient noise (`rgba(33,33,33,0.015)`) to add a paper feel

### The visual protagonist must vary

If 13 pages are all "text + one screenshot", it's monotonous. **Rotate the visual-protagonist type each page**:

| Visual type | Suited section |
|-------------|----------------|
| Cover typography (big type + masthead + pillar) | Title page / chapter cover |
| Single-character portrait (one giant momo, etc.) | Introducing a single concept/character |
| Group portrait / avatar cards in a row | Team / customer case |
| Timeline cards in progression | Showing "long-term relationship" / "evolution" |
| Knowledge graph / connected node diagram | Showing "collaboration" / "flow" |
| Before/After comparison cards + center arrow | Showing "change" / "difference" |
| Product UI screenshot + outlined device frame | Specific feature showcase |
| Big-quote (half-page giant type) | Emotion page / problem page / quotation page |
| Real avatar + quote card (2×2 or 1×4) | Customer testimonial / use case |
| Big-type back cover + URL pill button | CTA / closing |

---

## ⚠️ Common pitfalls (moxt war-story summary)

### 1. Emoji don't render in Chromium / Playwright export

Chromium has no color-emoji font by default, so `page.pdf()` or `page.screenshot()` shows emoji as empty boxes.

**Fix**: substitute Unicode glyphs (`✦` `✓` `✕` `→` `·` `—`), or just go plain text ("Email · 23" instead of "📧 23 emails").

### 2. `export_deck_pdf.mjs` errors with `Cannot find package 'playwright'`

Cause: ESM module resolution walks up from the script's location looking for `node_modules`. The script lives at `~/.claude/skills/huashu-design/scripts/`, where there are no deps.

**Fix**: copy the script into the deck project (e.g. `brochure/build-pdf.mjs`), run `npm install playwright pdf-lib` from the project root, then `node build-pdf.mjs --slides slides --out output/deck.pdf`.

### 3. Screenshot fires before Google Fonts finish loading → Chinese text renders in the system default sans-serif

Before Playwright screenshot/PDF, set at least `wait-for-timeout=3500` so the webfont downloads and paints. Or self-host fonts into `shared/fonts/` to reduce network dependency.

### 4. Information-density imbalance: content pages stuffed too tight

moxt philosophy page v1 had 2×2 = 4 paragraphs + 3 tenets at the bottom = 7 chunks of content, cramped and repetitive. Changed to 1×3 = 3 paragraphs and the breathing room came right back.

**Fix**: keep each page to "1 core message + 3-4 supporting points + 1 visual protagonist"; if you exceed that, split into a new page. **Less is more** — the audience looks at one page for 10 seconds; one memorable point lands better than four.

---

## 🛑 Pick the architecture first: single file or multi file?

**This is the first decision when building a deck — get it wrong and you'll keep stepping on the same rakes. Read this section before doing anything.**

### Architecture comparison

| Dimension | Single file + `deck_stage.js` | **Multi file + `deck_index.html` aggregator** |
|-----------|-------------------------------|-----------------------------------------------|
| Code structure | One HTML, every slide is a `<section>` | One HTML per slide, `index.html` stitches them via iframe |
| CSS scope | ❌ Global; one page's style can leak to all | ✅ Naturally isolated — every iframe is its own world |
| Verification grain | ❌ Need a JS goTo to switch to a given page | ✅ Just double-click a single-page file to view in the browser |
| Parallel development | ❌ One file, multiple agents conflict | ✅ Multiple agents work in parallel on different pages, zero merge conflicts |
| Debug difficulty | ❌ One CSS bug breaks the whole deck | ✅ A broken page only affects itself |
| Inline interactivity | ✅ Sharing state across pages is trivial | 🟡 iframes need postMessage |
| Print to PDF | ✅ Built-in | ✅ Aggregator iterates iframes on beforeprint |
| Keyboard navigation | ✅ Built-in | ✅ Built into the aggregator |

### Which one? (decision tree)

```
│ Q: how many pages will the deck have?
├── ≤10 pages, needs in-deck animation or cross-page interaction, pitch deck → single file
└── ≥10 pages, academic talk, courseware, long deck, multi-agent parallel work → multi file (recommended)
```

**Default to the multi-file path.** It's not a "backup" — it's the **main path for long decks and team collaboration**. Reason: every advantage of the single-file architecture (keyboard nav, print, scale) is also there in multi-file, while multi-file's scope isolation and verifiability are things single-file can't claw back.

### Why is this rule so firm? (real-incident log)

The single-file architecture once stepped on four rakes back-to-back during the AI-psychology talk deck:

1. **CSS specificity override**: `.emotion-slide { display: grid }` (specificity 10) trampled `deck-stage > section { display: none }` (specificity 2), causing every page to render and stack at once.
2. **Shadow DOM slot rule overridden by outer CSS**: `::slotted(section) { display: none }` couldn't beat the outer rule, so sections refused to hide.
3. **localStorage + hash navigation race**: after refresh, instead of jumping to the hash, it stuck on the old localStorage-recorded position.
4. **High verification cost**: had to `page.evaluate(d => d.goTo(n))` to screenshot a given page — twice as slow as `goto(file://.../slides/05-X.html)` and prone to errors.

All root causes trace to **a single global namespace** — multi-file architecture eliminates these problems at the physical level.

---

## Path A (default): multi-file architecture

### Directory structure

```
my-deck/
├── index.html              # Copy from assets/deck_index.html, edit MANIFEST
├── shared/
│   ├── tokens.css          # Shared design tokens (palette / type scale / common chrome)
│   └── fonts.html          # <link> for Google Fonts (each page includes)
└── slides/
    ├── 01-cover.html       # Each file is a complete 1920×1080 HTML
    ├── 02-agenda.html
    ├── 03-problem.html
    └── ...
```

### Per-slide template skeleton

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>P05 · Chapter Title</title>
<link href="https://fonts.googleapis.com/css2?family=..." rel="stylesheet">
<link rel="stylesheet" href="../shared/tokens.css">
<style>
  /* Styles unique to this page. Any class name you pick will never pollute another page. */
  body { padding: 120px; }
  .my-thing { ... }
</style>
</head>
<body>
  <!-- 1920×1080 content (body width/height locked in tokens.css) -->
  <div class="page-header">...</div>
  <div>...</div>
  <div class="page-footer">...</div>
</body>
</html>
```

**Key constraints**:
- `<body>` is the canvas — lay things out directly on it. Don't wrap with a `<section>` or any other wrapper.
- `width: 1920px; height: 1080px` is locked by the `body` rule in `shared/tokens.css`.
- Pull in `shared/tokens.css` for shared design tokens (palette, type scale, page-header/footer, etc.).
- Each page writes its own font `<link>` (font imports are cheap individually, and it guarantees each page opens standalone).

### Aggregator: `deck_index.html`

**Copy it straight from `assets/deck_index.html`.** You only need to change one thing — the `window.DECK_MANIFEST` array, listing every slide filename in order with a human-readable label:

```js
window.DECK_MANIFEST = [
  { file: "slides/01-cover.html",    label: "Cover" },
  { file: "slides/02-agenda.html",   label: "Agenda" },
  { file: "slides/03-problem.html",  label: "Problem statement" },
  // ...
];
```

The aggregator ships with: keyboard navigation (←/→/Home/End/number keys/P for print), scale + letterbox, bottom-right counter, localStorage memory, hash-based jump, print mode (iterates iframes to emit per-page PDF).

### Single-page verification (the multi-file architecture's killer feature)

Every slide is standalone HTML. **Finish a page, double-click it in the browser to look**:

```bash
open slides/05-personas.html
```

Playwright screenshots likewise just `goto(file://.../slides/05-personas.html)` — no JS paging needed, and no other page's CSS can interfere. This drives the "change a little, verify a little" workflow cost to near zero.

### Parallel development

Hand each slide's task to a different agent and run them concurrently — the HTML files are mutually independent, so there are no merge conflicts. Long decks built in parallel like this can compress production time to 1/N.

### What belongs in `shared/tokens.css`

Only **things that are genuinely shared across pages**:

- CSS variables (palette, type scale, spacing scale)
- Canvas locks like `body { width: 1920px; height: 1080px; }`
- Chrome that's identical on every page, like `.page-header` / `.page-footer`

**Don't** dump per-page layout classes in here — that regresses you to the single-file architecture's global pollution problem.

---

## Path B (small deck): single file + `deck_stage.js`

Use this for ≤10 pages, when you need cross-page shared state (e.g. a React Tweaks panel that drives every page), or for an ultra-compact pitch-deck demo.

### Basic usage

1. Read the content of `assets/deck_stage.js` and embed it in the HTML `<script>` (or `<script src="deck_stage.js">`)
2. Wrap slides in `<deck-stage>` inside body
3. 🛑 **The script tag must come after `</deck-stage>`** (see hard constraint below)

```html
<body>

  <deck-stage>
    <section>
      <h1>Slide 1</h1>
    </section>
    <section>
      <h1>Slide 2</h1>
    </section>
  </deck-stage>

  <!-- ✅ Correct: script is after deck-stage -->
  <script src="deck_stage.js"></script>

</body>
```

### 🛑 Script-position hard constraint (2026-04-20 real war story)

**Don't put `<script src="deck_stage.js">` in `<head>`.** Even though placing it in `<head>` does define `customElements`, the parser fires `connectedCallback` the moment it hits the opening `<deck-stage>` tag — at which point the child `<section>` elements haven't been parsed yet, `_collectSlides()` gets an empty array, the counter shows `1 / 0`, and every page renders stacked at once.

**Three compliant patterns** (pick any one):

```html
<!-- ✅ Most recommended: script after </deck-stage> -->
</deck-stage>
<script src="deck_stage.js"></script>

<!-- ✅ Also fine: script in head with defer -->
<head><script src="deck_stage.js" defer></script></head>

<!-- ✅ Also fine: module scripts are naturally deferred -->
<head><script src="deck_stage.js" type="module"></script></head>
```

`deck_stage.js` already ships with a `DOMContentLoaded` deferred-collection defense, so even script-in-head won't blow up entirely — but `defer` or placing it at the bottom of body is still the cleaner path; don't rely on the defensive branch.

### ⚠️ The CSS trap of single-file architecture (must read)

The most common pitfall of single-file architecture — **the `display` property gets stolen by per-page styles**.

Common wrong pattern 1 (writing display: flex directly on section):

```css
/* ❌ External CSS specificity 2 overrides shadow DOM's ::slotted(section){display:none} (also 2) */
deck-stage > section {
  display: flex;            /* All pages render stacked at once! */
  flex-direction: column;
  padding: 80px;
  ...
}
```

Common wrong pattern 2 (section has a higher-specificity class):

```css
.emotion-slide { display: grid; }   /* Specificity: 10, even worse */
```

Both cause **every slide to render stacked at once** — the counter might show `1 / 10` and look normal, but visually page 1 is covering page 2 covering page 3.

### ✅ Starter CSS (copy this verbatim, don't step on the rake)

**The section itself** only handles "visible / not visible"; **layout (flex/grid, etc.) goes on `.active`**:

```css
/* Section gets only non-display generic styles */
deck-stage > section {
  background: var(--paper);
  padding: 80px 120px;
  overflow: hidden;
  position: relative;
  /* ⚠️ Don't write display here! */
}

/* Lock "inactive = hidden" with both specificity and weight */
deck-stage > section:not(.active) {
  display: none !important;
}

/* Only the active page gets its display + layout */
deck-stage > section.active {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* Print mode: every page must show, override :not(.active) */
@media print {
  deck-stage > section { display: flex !important; }
  deck-stage > section:not(.active) { display: flex !important; }
}
```

Alternative: **put the per-page flex/grid on an inner wrapper `<div>`**, so the section itself is always nothing but a `display: block/none` switch. Cleanest approach:

```html
<deck-stage>
  <section>
    <div class="slide-content flex-layout">...</div>
  </section>
</deck-stage>
```

### Custom dimensions

```html
<deck-stage width="1080" height="1920">
  <!-- 9:16 portrait -->
</deck-stage>
```

---

## Slide Labels

Both deck_stage and deck_index label every page (shown in the counter). Give them **meaningful** labels:

**Multi file**: in `MANIFEST` write `{ file, label: "04 Problem Statement" }`
**Single file**: on the section add `<section data-screen-label="04 Problem Statement">`

**Key: slide numbers start at 1, not 0.**

When the user says "slide 5", they mean the 5th slide, never array index `[4]`. Humans don't speak 0-indexed.

---

## Speaker Notes

**Off by default** — only add them when the user explicitly asks.

Once you have speaker notes you can strip the slide text down to a minimum and focus on impactful visuals — the notes carry the full script.

### Format

**Multi file**: write inside `index.html`'s `<head>`:

```html
<script type="application/json" id="speaker-notes">
[
  "Script for slide 1...",
  "Script for slide 2...",
  "..."
]
</script>
```

**Single file**: same location.

### Notes-writing guidelines

- **Complete**: it's not an outline, it's what you actually plan to say
- **Conversational**: how you talk, not how you write
- **Aligned**: array item N corresponds to slide N
- **Length**: 200-400 characters works best
- **Emotional beats**: mark stresses, pauses, and emphasis points

---

## Slide design patterns

### 1. Establish a system (mandatory)

After exploring the design context, **state the system you'll use, out loud**:

```markdown
Deck system:
- Backgrounds: at most 2 (90% white + 10% dark section divider)
- Type: display in Instrument Serif, body in Geist Sans
- Rhythm: section dividers are full-bleed color + white text; normal slides are white-on-white
- Imagery: hero slides use full-bleed photos, data slides use charts

I'll work to this system — flag anything off.
```

Wait for the user to confirm before going further.

### 2. Common slide layouts

- **Title slide**: solid background + giant title + subtitle + author/date
- **Section divider**: colored background + chapter number + chapter title
- **Content slide**: white background + title + 1-3 bullet points
- **Data slide**: title + large chart/number + short caption
- **Image slide**: full-bleed photo + small caption at the bottom
- **Quote slide**: whitespace + giant quote + attribution
- **Two-column**: left-right comparison (vs / before-after / problem-solution)

Use at most 4-5 layouts in one deck.

### 3. Scale (saying it again)

- Body min **24px**, ideal 28-36px
- Titles **60-120px**
- Hero type **180-240px**
- Slides are read from 10 meters away — the type must be big enough

### 4. Visual rhythm

A deck needs **intentional variety**:

- Color rhythm: mostly white-on-white + occasional colored section dividers + occasional dark passages
- Density rhythm: a few text-heavy + a few image-heavy + a few whitespace-quote pages
- Type-size rhythm: normal titles + occasional giant hero type

**Don't make every slide look the same** — that's a PPT template, not design.

### 5. Spatial breathing (must read for data-dense pages)

**The most common rookie mistake**: cramming every possible piece of info onto one page.

Information density ≠ effective information delivery. Academic/lecture decks especially demand restraint:

- List/matrix pages: don't draw N elements all at the same size. Use **figure-ground hierarchy** — enlarge the 5 you're talking about today as the main act, shrink the remaining 16 as background hints.
- Big-number pages: the number itself is the visual protagonist. Keep surrounding caption to under 3 lines or the audience's eyes ping-pong.
- Quote pages: leave whitespace between the quote and the attribution; don't squish them together.

Self-audit against two checks: "is the data the protagonist?" and "is the text crammed together?". Iterate until the whitespace makes you a little uneasy.

---

## Print to PDF

**Multi file**: `deck_index.html` already handles the `beforeprint` event and emits the PDF page-by-page.

**Single file**: `deck_stage.js` does the same.

Print styles are wired up — you don't need to write extra `@media print` CSS.

---

## Export to PPTX / PDF (self-serve scripts)

HTML is the first-class citizen. But users frequently need PPTX/PDF delivery. Two general-purpose scripts ship under `scripts/` and **work on any multi-file deck**:

### `export_deck_pdf.mjs` — export a vector PDF (multi-file architecture)

```bash
node scripts/export_deck_pdf.mjs --slides <slides-dir> --out deck.pdf
```

**Features**:
- Text **stays vector** (selectable, searchable)
- 100% visual fidelity (Playwright's bundled Chromium renders, then prints)
- **No HTML changes required**
- Each slide gets its own `page.pdf()`, then `pdf-lib` merges them

**Dependencies**: `npm install playwright pdf-lib`

**Limitation**: text in PDF can't be edited — go back to HTML to change anything.

### `export_deck_stage_pdf.mjs` — dedicated to single-file deck-stage architecture ⚠️

**When to use**: the deck is a single HTML + `<deck-stage>` web component wrapping N `<section>` (Path B architecture). The `export_deck_pdf.mjs` "one `page.pdf()` per HTML" approach doesn't work here, so use this dedicated script.

```bash
node scripts/export_deck_stage_pdf.mjs --html deck.html --out deck.pdf
```

**Why you can't reuse export_deck_pdf.mjs** (2026-04-20 real war story):

1. **Shadow DOM beats `!important`**: deck-stage's shadow CSS has `::slotted(section) { display: none }` (only the active one is `display: block`). Even if you write `@media print { deck-stage > section { display: block !important } }` in light DOM, you can't push past — once `page.pdf()` triggers print media, Chromium's final render only contains the active slide, so **the entire PDF is 1 page** (a duplicate of the current active slide).

2. **Looping goto still emits only 1 page each**: the intuitive fix "navigate to each `#slide-N` then `page.pdf({pageRanges:'1'})`" also fails — because the outer-DOM `deck-stage > section { display: block }` print rule gets overridden, the final render is always the first item in the section list (not the page you navigated to). Result: 17 iterations, 17 copies of P01 cover.

3. **Absolute children spill to the next page**: even if you do get every section to render, when the section itself is `position: static`, its absolute-positioned `cover-footer`/`slide-footer` resolves against the initial containing block — when print forces the section to 1080px height, the absolute footer can be pushed onto the next page (manifests as PDF having 1 more page than there are sections, with that extra page being an orphaned footer).

**Fix strategy** (implemented in the script):

```js
// After opening the HTML, use page.evaluate to lift sections out of the deck-stage slot,
// drop them into a plain div directly under body, and inline-style them to position:relative + fixed size
await page.evaluate(() => {
  const stage = document.querySelector('deck-stage');
  const sections = Array.from(stage.querySelectorAll(':scope > section'));
  document.head.appendChild(Object.assign(document.createElement('style'), {
    textContent: `
      @page { size: 1920px 1080px; margin: 0; }
      html, body { margin: 0 !important; padding: 0 !important; }
      deck-stage { display: none !important; }
    `,
  }));
  const container = document.createElement('div');
  sections.forEach(s => {
    s.style.cssText = 'width:1920px!important;height:1080px!important;display:block!important;position:relative!important;overflow:hidden!important;page-break-after:always!important;break-after:page!important;background:#F7F4EF;margin:0!important;padding:0!important;';
    container.appendChild(s);
  });
  // Disable page-break on the last section to avoid a trailing blank page
  sections[sections.length - 1].style.pageBreakAfter = 'auto';
  sections[sections.length - 1].style.breakAfter = 'auto';
  document.body.appendChild(container);
});

await page.pdf({ width: '1920px', height: '1080px', printBackground: true, preferCSSPageSize: true });
```

**Why this works**:
- Lifting sections out of the shadow DOM slot into a plain light-DOM div completely bypasses the `::slotted(section) { display: none }` rule
- Inline `position: relative` makes absolute children resolve against the section, no overflow
- `page-break-after: always` gives every section its own page when the browser prints
- The last child has page-break disabled to avoid a trailing blank page

**A note on verifying with `mdls -name kMDItemNumberOfPages`**: macOS Spotlight metadata is cached — after rewriting a PDF, run `mdimport file.pdf` to force-refresh, otherwise it shows the old page count. The real count comes from `pdfinfo` or counting files via `pdftoppm`.

---

### `export_deck_pptx.mjs` — export editable PPTX

```bash
# Only mode: text frames are natively editable (fonts fall back to system fonts)
node scripts/export_deck_pptx.mjs --slides <dir> --out deck.pptx
```

How it works: `html2pptx` reads computedStyle element-by-element and translates the DOM into PowerPoint objects (text frame / shape / picture). Text becomes real text frames you can double-click to edit in PPT.

**Hard constraints** (the HTML must satisfy these or the page is skipped — full details in `references/editable-pptx.md`):
- All text must live in `<p>`/`<h1>`-`<h6>`/`<ul>`/`<ol>` (no bare-text div)
- `<p>`/`<h*>` themselves cannot have background/border/shadow (move them to an outer div)
- No `::before`/`::after` for decorative text (pseudo-elements can't be lifted out)
- Inline elements (span/em/strong) cannot have margin
- No CSS gradients (can't be rendered)
- div cannot use `background-image` (use `<img>`)

The script ships with an **automatic preprocessor** — it wraps "bare text in a leaf div" as `<p>` automatically (preserving classes). This handles the most common violation (bare text). Other violations (border on p, margin on span, etc.) still need to be fixed at the HTML source.

**Font-fallback caveat**:
- Playwright uses the webfont to measure text-box dimensions; PowerPoint/Keynote uses local fonts to render
- When they differ you get **overflow or misalignment** — eyeball every page
- Recommend installing the HTML-used fonts on the target machine, or fall back to `system-ui`

**Don't use this path for visual-first decks** → use `export_deck_pdf.mjs` to ship a PDF. A PDF is 100% visually faithful, vector, cross-platform, text-searchable — that's the real home for visual-first decks, not some "non-editable compromise".

### Make HTML export-friendly from day one

Most stable decks: **write HTML to the editable 4 hard constraints from the start.** Then `export_deck_pptx.mjs` passes everything cleanly. The extra cost is small:

```html
<!-- ❌ Bad -->
<div class="title">Key finding</div>

<!-- ✅ Good (p wrapper, class carried over) -->
<p class="title">Key finding</p>

<!-- ❌ Bad (border on p) -->
<p class="stat" style="border-left: 3px solid red;">41%</p>

<!-- ✅ Good (border on outer div) -->
<div class="stat-wrap" style="border-left: 3px solid red;">
  <p class="stat">41%</p>
</div>
```

### When to use which

| Scenario | Recommended |
|----------|-------------|
| For the host / archival storage | **PDF** (universal, high-fidelity, text-searchable) |
| Sending to collaborators so they can tweak copy | **PPTX editable** (accept font fallback) |
| Live presenting, no content edits | **PDF** (vector, faithful, cross-platform) |
| HTML is the primary medium | Play in the browser — export is just a backup |

## Deep path to editable PPTX (long-term projects only)

If your deck will be maintained long-term, edited repeatedly, and shared across a team — **write HTML to the html2pptx constraints from the start** so `export_deck_pptx.mjs` passes the lot. See `references/editable-pptx.md` (4 hard constraints + HTML templates + common-error reference + fallback flow when a visual draft already exists).

---

## FAQ

**Multi file: a page in an iframe won't open / blank screen**
→ Check whether the `MANIFEST` `file` path is correct relative to `index.html`. Use browser DevTools to confirm the iframe `src` opens directly.

**Multi file: a page's styles seem to conflict with another page**
→ Not possible (iframes are isolated). If it feels like a conflict, it's cache — Cmd+Shift+R for a hard refresh.

**Single file: multiple slides stacked at once**
→ CSS-specificity problem. See the "CSS trap of single-file architecture" section above.

**Single file: scale looks wrong**
→ Check that every slide hangs directly under `<deck-stage>` as a `<section>`. No `<div>` in between.

**Single file: want to jump to a specific slide**
→ Append a hash to the URL: `index.html#slide-5` jumps to slide 5.

**Both architectures: text positions inconsistent across screens**
→ Use fixed dimensions (1920×1080) and `px` units, not `vw`/`vh` or `%`. Let the scaling layer handle it uniformly.

---

## Verification checklist (must pass when the deck is done)

1. [ ] Open `index.html` (or main HTML) directly in the browser — first page renders, no broken images, fonts loaded
2. [ ] Press → through every page — no blank pages, no layout breakage
3. [ ] Press P for print preview — each page is exactly one A4 (or 1920×1080), no clipping
4. [ ] Pick 3 pages at random, Cmd+Shift+R hard-refresh — localStorage memory still works
5. [ ] Batch screenshot with Playwright (multi-file: iterate `slides/*.html`; single-file: use goTo) and eyeball every page
6. [ ] grep for any `TODO` / `placeholder` leftovers and confirm they're all cleaned up
