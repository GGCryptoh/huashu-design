# Apple Gallery Showcase · Gallery-Wall Animation Style

> Inspiration: the Claude Design site's hero video + Apple product pages' "wall of work" arrangement
> Origin in practice: huashu-design's release hero v5
> Where it fits: **product launch hero animations, skill capability demos, portfolio showcases** — anywhere you need to display "multiple high-quality outputs" at once and steer the audience's attention

---

## Trigger Test: When to Use This Style

**Good fit**:
- 10+ real outputs need to share the same screen (PPTs, Apps, web pages, infographics)
- Audience is professional (developers, designers, PMs), sensitive to "craft"
- The vibe you want to convey is "restrained, exhibition-style, high-end, with spatial depth"
- You need focus and the full picture simultaneously (detail without losing the whole)

**Bad fit**:
- Single-product focus (use the frontend-design product hero template instead)
- Emotion-heavy / narrative-strong animation (use the timeline-storytelling template instead)
- Small screens / portrait (the tilted perspective gets muddy on small frames)

---

## Core Visual Tokens

```css
:root {
  /* Light gallery palette */
  --bg:         #F5F5F7;   /* main canvas base — Apple-site gray */
  --bg-warm:    #FAF9F5;   /* warm off-white variant */
  --ink:        #1D1D1F;   /* primary text */
  --ink-80:     #3A3A3D;
  --ink-60:     #545458;
  --muted:      #86868B;   /* secondary text */
  --dim:        #C7C7CC;
  --hairline:   #E5E5EA;   /* card 1px border */
  --accent:     #D97757;   /* terracotta orange — Claude brand */
  --accent-deep:#B85D3D;

  --serif-cn: "Noto Serif SC", "Songti SC", Georgia, serif;
  --serif-en: "Source Serif 4", "Tiempos Headline", Georgia, serif;
  --sans:     "Inter", -apple-system, "PingFang SC", system-ui;
  --mono:     "JetBrains Mono", "SF Mono", ui-monospace;
}
```

**Key principles**:
1. **Never use a pure-black background**. Black makes the work look like a movie, not "work output you'd actually adopt"
2. **Terracotta orange is the sole hue accent**; everything else is grayscale + white
3. **Three-stack font system** (English serif + Chinese serif + sans + mono) creates a "publication" feel rather than an "internet product" feel

---

## Core Layout Patterns

### 1. Floating Card (the atomic unit of the entire style)

```css
.gallery-card {
  background: #FFFFFF;
  border-radius: 14px;
  padding: 6px;                          /* the padding is the "matting paper" */
  border: 1px solid var(--hairline);
  box-shadow:
    0 20px 60px -20px rgba(29, 29, 31, 0.12),   /* primary shadow, soft and long */
    0 6px 18px -6px rgba(29, 29, 31, 0.06);     /* secondary near-light, creates float */
  aspect-ratio: 16 / 9;                  /* uniform slide ratio */
  overflow: hidden;
}
.gallery-card img {
  width: 100%; height: 100%;
  object-fit: cover;
  border-radius: 9px;                    /* slightly smaller corner than the card — visual nesting */
}
```

**Anti-pattern**: don't tile edge-to-edge (no padding, no border, no shadow) — that's infographic-density expression, not exhibition.

### 2. 3D-Tilted Work Wall

```css
.gallery-viewport {
  position: absolute; inset: 0;
  overflow: hidden;
  perspective: 2400px;                   /* deeper perspective so the tilt isn't exaggerated */
  perspective-origin: 50% 45%;
}
.gallery-canvas {
  width: 4320px;                         /* canvas = 2.25× viewport */
  height: 2520px;                        /* leave pan room */
  transform-origin: center center;
  transform: perspective(2400px)
             rotateX(14deg)              /* tilt back */
             rotateY(-10deg)             /* turn left */
             rotateZ(-2deg);             /* slight skew, breaks "too neat" */
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 40px;
  padding: 60px;
}
```

**Parameter sweet spots**:
- rotateX: 10–15deg (more than that and it looks like a VIP-event backdrop)
- rotateY: ±8–12deg (left-right symmetry)
- rotateZ: ±2–3deg (the "not-arranged-by-a-machine" human touch)
- perspective: 2000–2800px (under 2000 fish-eyes, over 3000 approaches orthographic)

### 3. 2×2 Four-Corner Convergence (Selection Scene)

```css
.grid22 {
  display: grid;
  grid-template-columns: repeat(2, 800px);
  gap: 56px 64px;
  align-items: start;
}
```

Each card slides in toward the center from its corresponding corner (tl / tr / bl / br) + fades in. The matching `cornerEntry` vectors:

```js
const cornerEntry = {
  tl: { dx: -700, dy: -500 },
  tr: { dx:  700, dy: -500 },
  bl: { dx: -700, dy:  500 },
  br: { dx:  700, dy:  500 },
};
```

---

## Five Core Animation Patterns

### Pattern A · Four-Corner Convergence (0.8–1.2s)

Four elements slide in from the viewport corners while scaling 0.85 → 1.0, ease-out. Good for an opening that "shows multi-directional choices."

```js
const inP = easeOut(clampLerp(t, start, end));
card.style.transform = `translate3d(${(1-inP)*ce.dx}px, ${(1-inP)*ce.dy}px, 0) scale(${0.85 + 0.15*inP})`;
card.style.opacity = inP;
```

### Pattern B · Selected Card Zooms + Others Slide Out (0.8s)

The selected card scales 1.0 → 1.28; the others fade out + blur + drift back toward their corners:

```js
// selected
card.style.transform = `translate3d(${cellDx*outP}px, ${cellDy*outP}px, 0) scale(${1 + 0.28*easeOut(zoomP)})`;
// not selected
card.style.opacity = 1 - outP;
card.style.filter = `blur(${outP * 1.5}px)`;
```

**Key**: the unselected ones should blur, not just fade. Blur mimics depth of field and visually "pushes out" the selected card.

### Pattern C · Ripple Expansion (1.7s)

From the center outward, each card fades in based on its distance-delay and scales from 1.25× → 0.94× ("camera pulling back"):

```js
const col = i % COLS, row = Math.floor(i / COLS);
const dc = col - (COLS-1)/2, dr = row - (ROWS-1)/2;
const dist = Math.sqrt(dc*dc + dr*dr);
const delay = (dist / maxDist) * 0.8;
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
card.style.opacity = easeOut(Math.min(1, localT));

// concurrently the whole canvas scales 1.25→0.94
const galleryScale = 1.25 - 0.31 * easeOut(rippleProgress);
```

### Pattern D · Sinusoidal Pan (Continuous Drift)

Combine a sine wave with linear drift to avoid the "has a start and an end" feeling of a marquee loop:

```js
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;    // drifts left horizontally
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;    // drifts up vertically
const clampedX = Math.max(-900, Math.min(900, panX));   // prevent exposing edges
```

**Parameters**:
- Sine period `0.09–0.15 rad/s` (slow, about 30–50s per cycle)
- Linear drift `5–8 px/s` (slower than a blink)
- Amplitude `120–220 px` (big enough to feel, small enough not to nauseate)

### Pattern E · Focus Overlay (Focus Switching)

**Key design**: the focus overlay is a **flat element** (not tilted) floating above the tilted canvas. The selected slide scales from its tile position (~400×225) to screen-center (960×540); the background canvas's tilt doesn't change but **dims to 45%**:

```js
// Focus overlay (flat, centered)
focusOverlay.style.width = (startW + (endW - startW) * focusIntensity) + 'px';
focusOverlay.style.height = (startH + (endH - startH) * focusIntensity) + 'px';
focusOverlay.style.opacity = focusIntensity;

// background cards dim but stay visible (critical — never 100% mask)
card.style.opacity = entryOp * (1 - 0.55 * focusIntensity);   // 1 → 0.45
card.style.filter = `brightness(${1 - 0.3 * focusIntensity})`;
```

**Sharpness rules of iron**:
- The focus overlay's `<img>` must `src` straight to the original image — **do not reuse the compressed thumbnail from the gallery**
- Preload every original into a `new Image()[]` array up front
- Compute the overlay's `width/height` per frame; the browser resamples the original each frame

---

## Timeline Architecture (Reusable Skeleton)

```js
const T = {
  DURATION: 25.0,
  s1_in: [0.0, 0.8],    s1_type: [1.0, 3.2],  s1_out: [3.5, 4.0],
  s2_in: [3.9, 5.1],    s2_hold: [5.1, 7.0],  s2_out: [7.0, 7.8],
  s3_hold: [7.8, 8.3],  s3_ripple: [8.3, 10.0],
  panStart: 8.6,
  focuses: [
    { start: 11.0, end: 12.7, idx: 2  },
    { start: 13.3, end: 15.0, idx: 3  },
    { start: 15.6, end: 17.3, idx: 10 },
    { start: 17.9, end: 19.6, idx: 16 },
  ],
  s4_walloff: [21.1, 21.8], s4_in: [21.8, 22.7], s4_hold: [23.7, 25.0],
};

// core easing
const easeOut = t => 1 - Math.pow(1 - t, 3);
const easeInOut = t => t < 0.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2, 3)/2;
function lerp(time, start, end, fromV, toV, easing) {
  if (time <= start) return fromV;
  if (time >= end) return toV;
  let p = (time - start) / (end - start);
  if (easing) p = easing(p);
  return fromV + (toV - fromV) * p;
}

// a single render(t) function reads the timestamp and writes every element
function render(t) { /* ... */ }
requestAnimationFrame(function tick(now) {
  const t = ((now - startMs) / 1000) % T.DURATION;
  render(t);
  requestAnimationFrame(tick);
});
```

**Architectural essence**: **every piece of state is derived from the timestamp t** — no state machines, no setTimeout. This buys you:
- Jump to any moment with `window.__setTime(12.3)` (easy frame-by-frame Playwright capture)
- Seamless looping for free (t mod DURATION)
- Freeze any frame during debugging

---

## Texture Details (Easily Skipped, But Make-or-Break)

### 1. SVG noise texture

A light background's worst failure mode is "too flat." Layer on a barely-there fractalNoise:

```html
<style>
.stage::before {
  content: '';
  position: absolute; inset: 0;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='200' height='200'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 0.078  0 0 0 0 0.078  0 0 0 0 0.074  0 0 0 0.035 0'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>");
  opacity: 0.5;
  pointer-events: none;
  z-index: 30;
}
</style>
```

It looks the same on screen — remove it and you'll feel it was there.

### 2. Corner Brand Mark

```html
<div class="corner-brand">
  <div class="mark"></div>
  <div>HUASHU · DESIGN</div>
</div>
```

```css
.corner-brand {
  position: absolute; top: 48px; left: 72px;
  font-family: var(--mono);
  font-size: 12px;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: var(--muted);
}
```

Only shown during the work-wall scene, fades in and out. Like a museum exhibit label.

### 3. Brand Reveal Wordmark

```css
.brand-wordmark {
  font-family: var(--sans);
  font-size: 148px;
  font-weight: 700;
  letter-spacing: -0.045em;   /* negative letter-spacing is the key — tightens the type into a logo */
}
.brand-wordmark .accent {
  color: var(--accent);
  font-weight: 500;           /* the accent character is actually thinner — visual contrast */
}
```

`letter-spacing: -0.045em` is the standard treatment for big type on Apple's product pages.

---

## Common Failure Modes

| Symptom | Cause | Fix |
|---|---|---|
| Looks like a PPT template | Cards have no shadow / hairline | Add two-layer box-shadow + 1px border |
| The tilt looks cheap | Only `rotateY`, no `rotateZ` | Add ±2–3deg `rotateZ` to break the rigidity |
| The pan feels "stuttery" | Used setTimeout or CSS keyframe loop | Use rAF + sin/cos as continuous functions |
| Focus text is unreadable | Reused the low-res tile from the gallery | Separate overlay + `src` straight to the original |
| Background is too empty | Solid `#F5F5F7` | Layer on SVG fractalNoise at 0.5 opacity |
| Type feels too "internet" | Only Inter | Add a serif stack (one English + one Chinese) + mono — three stacks |

---

## References

- Full implementation sample: `/Users/alchain/Documents/Writing/01-Official-Account/Projects/2026.04-huashu-design-release/Assets/hero-animation-v5.html`
- Original inspiration: claude.ai/design hero video
- Aesthetic reference: Apple product pages, Dribbble shot collection pages

When you hit an animation request that means "lay out multiple high-quality outputs side by side," copy the skeleton from this file, swap the content, tune timing — done.
