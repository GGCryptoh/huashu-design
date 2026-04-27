# Gallery Ripple + Multi-Focus · Scene Choreography Philosophy

> A **reusable visual choreography structure** distilled from the huashu-design hero animation v9 (25 seconds, 8 scenes).
> Not an animation production pipeline — instead, **which scenarios make this choreography "right"**.
> Reference in practice: [demos/hero-animation-v9.mp4](../demos/hero-animation-v9.mp4) · [https://www.huasheng.ai/huashu-design-hero/](https://www.huasheng.ai/huashu-design-hero/)

## One-Liner First

> **When you have 20+ homogeneous visual assets and the scene needs to "express scale and depth," reach for Gallery Ripple + Multi-Focus before stacking layouts.**

Generic SaaS feature animations, product launches, skill promos, portfolio collection showcases — as long as you have enough assets and consistent style, this structure almost always lands.

---

## What This Technique Is Actually Expressing

It's not "showing off the assets" — it's telling a narrative through **two rhythm shifts**:

**Beat 1 · Ripple expansion (~1.5s)**: 48 cards radiate outward from the center; the audience gets hit by "quantity" — "oh, this thing produces a *lot*."

**Beat 2 · Multi-Focus (~8s, 4 loops)**: while the camera slowly pans, 4 times the background dims + desaturates, lifting one card to the center of the screen — the audience switches from "the impact of quantity" to "the gaze of quality," each at a steady 1.7s cadence.

**Core narrative structure**: **Scale (Ripple) → Gaze (Focus × 4) → Fade out (Walloff)**. Combined, these three beats express *Breadth × Depth* — not just that it can produce a lot, but that each piece is worth stopping for.

Compare the counter-examples:

| Approach | Audience perception |
|------|---------|
| 48 cards in a static grid (no Ripple) | Pretty but no narrative — looks like a grid screenshot |
| Quick-cut one card at a time (no Gallery context) | Feels like a slideshow, loses the sense of "scale" |
| Ripple alone, no Focus | Stunning, but no specific card sticks |
| **Ripple + Focus × 4 (this recipe)** | **First the awe of quantity, then the gaze of quality, then a calm fade — a full emotional arc** |

---

## Prerequisites (All Four Required)

This choreography is **not universal**. The 4 conditions below are all required:

1. **Asset count ≥ 20, ideally 30+**
   Fewer than 20 and the Ripple feels "empty" — every cell of the 48 must be in motion to give the density. v9 used 48 cells × 32 images (cycled to fill).

2. **Consistent visual style across assets**
   All 16:9 slide previews / all app screenshots / all cover designs — aspect ratio, color, layout must look like "a single set." Mixing styles makes the gallery look like a clipboard pile.

3. **Each asset is still readable when zoomed in**
   Focus enlarges a card to 960px wide; if the original goes blurry or has thin information at that size, the Focus beat is wasted. Inverse test: can you pick 4 out of the 48 as "most representative"? If not, the assets aren't of even quality.

4. **The scene is landscape or square, not portrait**
   The Gallery's 3D tilt (`rotateX(14deg) rotateY(-10deg)`) needs lateral spread; portrait orientation makes the tilt look narrow and awkward.

**Fallback paths when conditions are missing**:

| What's missing | Degrade to |
|-------|-----------|
| Fewer than 20 assets | Switch to "3–5 static side-by-side + focus one at a time" |
| Inconsistent style | Switch to keynote-style "cover + 3 chapter hero images" |
| Thin information | Switch to "data-driven dashboard" or "money quote + big type" |
| Portrait orientation | Switch to "vertical scroll + sticky cards" |

---

## Technical Recipe (v9 production parameters)

### 4-Layer Structure

```
viewport (1920×1080, perspective: 2400px)
  └─ canvas (4320×2520, oversized overflow) → 3D tilt + pan
      └─ 8×6 grid = 48 cards (gap 40px, padding 60px)
          └─ img (16:9, border-radius 9px)
      └─ focus-overlay (absolute center, z-index 40)
          └─ img (matches selected slide)
```

**Key**: the canvas is 2.25× larger than the viewport, so the pan creates a "peeking into a larger world" feeling.

### Ripple Expansion (Distance-Based Delay Algorithm)

```js
// Each card's entry time = distance from center × 0.8s delay
const col = i % 8, row = Math.floor(i / 8);
const dc = col - 3.5, dr = row - 2.5;       // offset from center
const dist = Math.hypot(dc, dr);
const maxDist = Math.hypot(3.5, 2.5);
const delay = (dist / maxDist) * 0.8;       // 0 → 0.8s
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
const opacity = expoOut(Math.min(1, localT));
```

**Core parameters**:
- Total duration 1.7s (`T.s3_ripple: [8.3, 10.0]`)
- Max delay 0.8s (center fires earliest, corners last)
- Per-card entry duration 0.7s
- Easing: `expoOut` (burst feel, not smooth)

**Concurrently**: canvas scale goes from 1.25 → 0.94 (zoom out to reveal) — pairing the appearance with a synchronous pull-back.

### Multi-Focus (4 Beats)

```js
T.focuses = [
  { start: 11.0, end: 12.7, idx: 2  },  // 1.7s
  { start: 13.3, end: 15.0, idx: 3  },  // 1.7s
  { start: 15.6, end: 17.3, idx: 10 },  // 1.7s
  { start: 17.9, end: 19.6, idx: 16 },  // 1.7s
];
```

**Cadence**: each focus 1.7s, with 0.6s breathing room between. Total 8s (11.0–19.6s).

**Inside each focus**:
- In ramp: 0.4s (`expoOut`)
- Hold: 0.9s middle (`focusIntensity = 1`)
- Out ramp: 0.4s (`easeOut`)

**Background change (this is the key)**:

```js
if (focusIntensity > 0) {
  const dimOp = entryOp * (1 - 0.6 * focusIntensity);  // dim to 40%
  const brt = 1 - 0.32 * focusIntensity;                // brightness 68%
  const sat = 1 - 0.35 * focusIntensity;                // saturate 65%
  card.style.filter = `brightness(${brt}) saturate(${sat})`;
}
```

**It's not just opacity — it's desaturate + darken at the same time**. That makes the foreground overlay's colors "pop out" instead of merely "brightening a bit."

**Focus overlay size animation**:
- From 400×225 (entry) → 960×540 (hold state)
- Surrounded by 3 layers of shadow + a 3px accent-color outline ring, giving it the feel of "being framed"

### Pan (Continuous Motion Keeps Stillness from Going Boring)

```js
const panT = Math.max(0, t - 8.6);
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;
```

- Sine wave + linear drift, two layers of motion — not a pure loop, every moment's position is different
- X and Y use different frequencies (0.12 vs. 0.09) to avoid the eye spotting a "regular cycle"
- Clamp at ±900/500px to prevent drifting off-stage

**Why not a pure linear pan**: with pure linear, the audience "predicts" where it'll be a second from now. Sine + drift makes every second new; combined with the 3D tilt it produces a faint sense of motion-sickness (the good kind) that holds attention.

---

## 5 Reusable Patterns (Distilled From the v6 → v9 Iteration)

### 1. **`expoOut` as the primary easing, not `cubicOut`**

`easeOut = 1 - (1-t)³` (smooth) vs. `expoOut = 1 - 2^(-10t)` (bursts then converges fast).

**Why pick it**: `expoOut`'s first 30% reaches 90%, mimicking physical damping — the intuition of "something heavy landing." Particularly suited to:
- Card entry (weight)
- Ripple expansion (shockwave)
- Brand float-up (settling in)

**When to still use `cubicOut`**: focus out ramps, symmetric micro-animations.

### 2. **Paper-feel base + terracotta-orange accent (Anthropic lineage)**

```css
--bg: #F7F4EE;        /* warm paper */
--ink: #1D1D1F;       /* near black */
--accent: #D97757;    /* terracotta orange */
--hairline: #E4DED2;  /* warm line */
```

**Why**: a warm base still "breathes" after GIF compression — pure white reads as "screeny." Terracotta orange as the sole accent threads through the terminal prompt, dir-card selection, cursor, brand hyphen, focus ring — every visual anchor strung onto a single color.

**v5 lesson**: added a noise overlay to simulate paper grain — GIF frame compression got destroyed (every frame was different). v6 switched to "base color + warm shadow only," keeping 90% of the paper feel and shrinking GIF size by 60%.

### 3. **Two-Tier Shadow to Fake Depth, Not Real 3D**

```css
.gallery-card.depth-near { box-shadow: 0 32px 80px -22px rgba(60,40,20,0.22), ... }
.gallery-card.depth-far  { box-shadow: 0 14px 40px -16px rgba(60,40,20,0.10), ... }
```

Use a deterministic `sin(i × 1.7) + cos(i × 0.73)` algorithm to assign each card a near / mid / far shadow tier — **visually you get a "3D stacking" feel, but the per-frame transform never changes; GPU cost is 0**.

**The cost of real 3D**: per-card `translateZ` makes the GPU compute 48 transforms + shadow blurs every frame. v4 tried it and Playwright struggled to record at 25fps. The two-tier shadow in v6 is <5% visually different but 10× cheaper.

### 4. **Weight Change (`font-variation-settings`) Reads More Cinematic Than Size Change**

```js
const wght = 100 + (700 - 100) * morphP;  // 100 → 700 over 0.9s
wordmark.style.fontVariationSettings = `"wght" ${wght.toFixed(0)}`;
```

The brand wordmark transitions Thin → Bold over 0.9s, with a letter-spacing tweak (-0.045 → -0.048em).

**Why it beats scale up/down**:
- Audiences have seen scale up/down too many times; expectations are baked in
- Weight change reads as "inner fullness," like a balloon inflating, instead of "being pushed forward"
- Variable fonts only became common after 2020+, so the audience unconsciously reads it as "modern"

**Constraint**: must use a variable-capable font (Inter / Roboto Flex / Recursive, etc.). Static fonts can only fake it (switching among a few fixed weights produces visible jumps).

### 5. **Low-Intensity Corner Brand as a Persistent Signature**

During the Gallery scene, the top-left carries a tiny `HUASHU · DESIGN` mark — 16% opacity, 12px size, wide letter-spacing.

**Why it's there**:
- After the Ripple burst, the audience tends to "lose focus" and forget what they're watching; a soft top-left mark anchors them
- More elegant than a full-screen logo — anyone in branding knows a signature doesn't need to shout
- Leaves an attribution signal even when the GIF gets screenshotted and reshared

**Rule**: appears only mid-piece (when the frame is busy); off at the open (don't cover the terminal), off at the close (brand reveal is the lead).

---

## Counter-Examples: When NOT to Use This Choreography

**❌ Product demo (showing features)**: the Gallery makes everything flash by; the audience won't remember a single feature. Use "single-screen focus + tooltip annotations" instead.

**❌ Data-driven content**: viewers need to read numbers, and the Gallery's fast cadence doesn't give them time. Use "data charts + reveal item-by-item" instead.

**❌ Story narrative**: the Gallery is a "parallel" structure; stories need "causation." Use keynote-style chapter transitions instead.

**❌ Only 3–5 assets**: the Ripple lacks density and looks "patchy." Use "static layout + highlight one card at a time" instead.

**❌ Portrait (9:16)**: the 3D tilt needs lateral spread; portrait makes the tilt feel "crooked" instead of "unfolding."

---

## How to Decide if Your Task Fits This Choreography

A three-step quick check:

**Step 1 · Asset count**: count how many same-class visual assets you have. <15 → stop; 15–25 → marginal; 25+ → just use it.

**Step 2 · Consistency test**: place 4 random assets side-by-side. Does it look like "a set"? If not → unify the style first, or switch approaches.

**Step 3 · Narrative match**: are you trying to express *Breadth × Depth* (quantity × quality)? Or "process," "features," "story"? If it's not the first, don't force the fit.

If all three are yes, fork the v6 HTML, change the `SLIDE_FILES` array and timeline — you've got it. Swap the palette via `--bg / --accent / --ink` and you've reskinned without touching the bones.

---

## Related References

- Full technical workflow: [references/animations.md](animations.md) · [references/animation-best-practices.md](animation-best-practices.md)
- Animation export pipeline: [references/video-export.md](video-export.md)
- Audio config (BGM + SFX dual-track): [references/audio-design-rules.md](audio-design-rules.md)
- Apple gallery style sibling reference: [references/apple-gallery-showcase.md](apple-gallery-showcase.md)
- Source HTML (v6 + audio integration version): `www.huasheng.ai/huashu-design-hero/index.html`
