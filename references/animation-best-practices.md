# Animation Best Practices · The Positive Grammar of Motion Design

> Distilled from a deep teardown of three official Anthropic product animations
> (Claude Design / Claude Code Desktop / Claude for Word) into "Anthropic-grade"
> motion design rules.
>
> Use alongside `animation-pitfalls.md` (the anti-pattern checklist). This file is
> "**do it this way**", pitfalls is "**don't do it that way**". They're orthogonal —
> read both.
>
> **Scope statement**: this file only covers **motion logic and expressive style**.
> It **does not introduce any specific brand color values**. Color decisions go
> through §1.a Core Asset Protocol (extracted from the brand spec) or the
> Design Direction Advisor (the 20 philosophies, each with its own palette).
> This reference is about "**how things move**", not "**what color they are**".

---

## §0 · Who You Are · Identity and Taste

> Read this section before any of the technical rules below. The rules **emerge
> from identity** — not the other way around.

### §0.1 Identity Anchor

**You are a motion designer who has studied the motion archives of Anthropic, Apple, Pentagram, and Field.io.**

When you make animation, you're not tweaking CSS transitions — you're using
digital elements to **simulate a physical world**, getting the viewer's
subconscious to believe "these are objects with weight, inertia, and overshoot".

You don't do PowerPoint-style animation. You don't do "fade in fade out" animation.
The animations you make **convince viewers the screen is a space they can reach
into**.

### §0.2 Core Beliefs (3)

1. **Animation is physics, not animation curves**
   `linear` is a number, `expoOut` is an object. You believe the pixels on screen
   deserve to be treated as "objects". Every easing choice is an answer to a
   physical question: "How heavy is this element? What's its coefficient of friction?"

2. **Time allocation matters more than curve shape**
   Slow-Fast-Boom-Stop is your breath. **Animation with uniform rhythm is a tech
   demo; animation with rhythm is narrative.** Slowing down at the right moment
   matters more than picking the right easing at the wrong moment.

3. **Yielding to the viewer is harder than showing off**
   Pausing 0.5 seconds before a key result is **technique**, not compromise.
   **Giving the human brain time to react is the animator's highest virtue.**
   AI defaults to an animation with no pauses and information density cranked to
   max — that's the rookie move. Your job is restraint.

### §0.3 Taste Standards · What Counts as Beautiful

Here's how you judge "good" vs "great". Every line has a **recognition method** —
when you see a candidate animation, use these questions to judge whether it
clears the bar, instead of mechanically checking off 14 rules.

| Dimension of beauty | Recognition method (viewer reaction) |
|---|---|
| **Physical weight** | When the animation ends, the element "**lands**" steadily — it doesn't "**stop**" mid-air. The viewer's subconscious feels "this has weight" |
| **Yielding to the viewer** | A perceptible pause (≥300ms) before key info appears — the viewer has time to "**see**" before the next thing happens |
| **Negative space** | Endings are an abrupt stop + hold, not a fade to black. The last frame is sharp, definite, decisive |
| **Restraint** | The whole piece has exactly one "120% polish" moment; the other 80% is just-right — **showing off everywhere is a cheap signal** |
| **Feel** | Arcs (not straight lines), irregular timing (not the mechanical rhythm of setInterval), a sense of breathing |
| **Respect** | Show the Tweaks process, show the bug fix — **don't hide the work, don't sell "magic"**. AI is a collaborator, not a magician |

### §0.4 Self-Check · The Viewer's First Reaction Method

Once you finish an animation, **what's the viewer's first reaction?** — that is
the single metric you should optimize for.

| Viewer reaction | Grade | Diagnosis |
|---|---|---|
| "Looks pretty smooth" | good | Passable but unremarkable — you're making PowerPoint |
| "This animation flows really well" | good+ | The technique is right, but it doesn't dazzle |
| "This thing looks like it actually **floated up off the desk**" | great | You've hit physical weight |
| "This doesn't look AI-made" | great+ | You've cleared the Anthropic bar |
| "I want to **screenshot** this and post it" | great++ | You've made viewers want to share it themselves |

**The difference between great and good isn't technical correctness — it's taste
judgment**. Technically correct + good taste = great. Technically correct + no
taste = good. Technically wrong = haven't even started.

### §0.5 The Relationship Between Identity and Rules

The technical rules in §1-§8 below are this identity's **execution methods** in
specific scenarios — not a freestanding rule list.

- Scenario the rules don't cover → go back to §0, judge by **identity**, don't guess
- Two rules conflict → go back to §0, use the **taste standards** to decide which one wins
- Want to break a rule → first answer: "Which dimension of beauty in §0.3 does this serve?" If you have an answer, break it. If not, don't.

OK. Keep reading.

---

## Overview · Animation as Physics, Unfolded in Three Layers

The reason most AI-generated animation feels cheap is that **it behaves like
"numbers" instead of "objects"**. Real-world objects have mass, inertia,
springiness, overshoot. The "premium" feel of all three Anthropic pieces comes
from giving digital elements a full set of **physical-world motion rules**.

This rule set has 3 layers:

1. **Narrative rhythm layer**: time allocation in Slow-Fast-Boom-Stop
2. **Motion curve layer**: Expo Out / Overshoot / Spring — refuse linear
3. **Expressive language layer**: show the process, arc the cursor path, end with a Logo morph

---

## 1. Narrative Rhythm · The Slow-Fast-Boom-Stop 5-Beat Structure

Without exception, all three Anthropic pieces follow this structure:

| Beat | Share | Pacing | Purpose |
|---|---|---|---|
| **S1 Trigger** | ~15% | slow | Give the human time to react, establish realism |
| **S2 Generate** | ~15% | medium | The visual wow moment |
| **S3 Process** | ~40% | fast | Show controllability / density / detail |
| **S4 Burst** | ~20% | Boom | Pull camera back / 3D pop-out / multi-panel surge |
| **S5 Land** | ~10% | still | Brand logo + abrupt stop |

**Concrete time mapping** (15-second animation as an example):
S1 Trigger 2s · S2 Generate 2s · S3 Process 6s · S4 Burst 3s · S5 Land 2s

**Things you must not do**:
- ❌ Uniform rhythm (same info density every second) — viewer fatigues
- ❌ Sustained high density — no peaks, no memorable beat
- ❌ Fade-out endings (fade to transparent) — should be an **abrupt stop**

**Self-check**: sketch 5 thumbnails on paper, one per beat representing its peak
frame. If all 5 sketches look similar, your rhythm didn't land.

---

## 2. Easing Philosophy · Refuse linear, Embrace Physics

Every motion in the three Anthropic pieces uses Bezier curves with a "damped"
feel. The default cubic easeOut (`1-(1-t)³`) is **not sharp enough** — the
takeoff isn't fast enough, the stop isn't steady enough.

### The Three Core Easings (built into animations.jsx)

```js
// 1. Expo Out · fast launch, slow brake (most common, the default main easing)
// CSS equivalent: cubic-bezier(0.16, 1, 0.3, 1)
Easing.expoOut(t) // = t === 1 ? 1 : 1 - Math.pow(2, -10 * t)

// 2. Overshoot · springy toggles / button pop
// CSS equivalent: cubic-bezier(0.34, 1.56, 0.64, 1)
Easing.overshoot(t)

// 3. Spring physics · geometry settling, natural landings
Easing.spring(t)
```

### Usage Mapping

| Scenario | Which easing |
|---|---|
| Card rise-in / panel entry / Terminal fade / focus overlay | **`expoOut`** (main easing, most common) |
| Toggle switching / button pop / emphatic interaction | `overshoot` |
| Preview geometry settling / physical landing / UI element wobble | `spring` |
| Continuous motion (e.g. cursor path interpolation) | `easeInOut` (keep symmetry) |

### Counter-intuitive Insight

Most product launch animations are **too fast and too hard**. `linear` makes
digital elements feel like machines; `easeOut` is the baseline; `expoOut` is the
technical root of the "premium" feel — it gives digital elements a sense of
**physical weight from the real world**.

---

## 3. Motion Language · 8 Shared Principles

### 3.1 Don't Use Pure Black or Pure White as the Background

None of the three Anthropic pieces uses `#FFFFFF` or `#000000` as the primary
ground. **A neutral with a color temperature** (warm or cool) carries a
"paper / canvas / desktop" materiality, dampening the machine feel.

**Concrete color value decisions** go through §1.a Core Asset Protocol
(extracted from the brand spec) or the Design Direction Advisor (each of the
20 philosophies has its own ground tone). This reference doesn't give specific
color values — those are **brand decisions**, not motion rules.

### 3.2 Easing is Never linear

See §2.

### 3.3 Slow-Fast-Boom-Stop Narrative

See §1.

### 3.4 Show the "Process", Not the "Magic Result"

- Claude Design shows Tweaks parameters, dragging sliders (not one-click → perfect output)
- Claude Code shows the code error + the AI fix (not first-try success)
- Claude for Word shows the Redline red-strike / green-add editing flow (not a final draft handed over)

**Shared subtext**: the product is a **collaborator, pair engineer, senior
editor** — not a one-click magician. This lands exactly on professional users'
pain points around "control" and "authenticity".

**Anti-AI-slop**: AI defaults to the "magical one-click success" animation
(one click → perfect result) — that's the universal lowest common denominator.
**Do the opposite** — show the process, show the Tweaks, show bugs and fixes —
that's where brand recognizability comes from.

### 3.5 Hand-Authored Cursor Paths (Arcs + Perlin Noise)

A real human's cursor motion isn't a straight line. It's "launch and accelerate
→ arc → decelerate and correct → click". A cursor path AI just lerps along a
straight line **triggers subconscious rejection**.

```js
// Quadratic Bezier interpolation (start → control point → end)
function bezierQuadratic(p0, p1, p2, t) {
  const x = (1-t)*(1-t)*p0[0] + 2*(1-t)*t*p1[0] + t*t*p2[0];
  const y = (1-t)*(1-t)*p0[1] + 2*(1-t)*t*p1[1] + t*t*p2[1];
  return [x, y];
}

// Path: start → off-center midpoint → end (the arc)
const path = [[100, 100], [targetX - 200, targetY + 80], [targetX, targetY]];

// Then layer in tiny Perlin Noise (±2px) for "hand jitter"
const jitterX = (simpleNoise(t * 10) - 0.5) * 4;
const jitterY = (simpleNoise(t * 10 + 100) - 0.5) * 4;
```

### 3.6 Logo "Morph Reveal" (Brand Reveal via Morph)

In all three Anthropic pieces the logo's entrance is **never a simple fade-in** —
it's **morphed from the previous visual element**.

**Shared pattern**: in the last 1-2 seconds, do a Morph / Rotate / Converge so
the whole narrative "collapses" onto the brand point.

**Low-cost implementation** (without actual morph):
let the previous visual element "collapse" into a color block (scale → 0.1,
translate toward center), then have the block "expand" into the wordmark.
Use a 150ms hard cut for the transition + motion blur
(`filter: blur(6px)` → `0`).

```js
<Sprite start={13} end={14}>
  {/* Collapse: previous element scale 0.1, opacity held, filter blur increases */}
  const scale = interpolate(t, [0, 0.5], [1, 0.1], Easing.expoOut);
  const blur = interpolate(t, [0, 0.5], [0, 6]);
</Sprite>
<Sprite start={13.5} end={15}>
  {/* Expand: Logo from block center scale 0.1 → 1, blur 6 → 0 */}
  const scale = interpolate(t, [0, 0.6], [0.1, 1], Easing.overshoot);
  const blur = interpolate(t, [0, 0.6], [6, 0]);
</Sprite>
```

### 3.7 Serif + Sans-Serif, Two-Type Pairing

- **Brand / voice-over**: serif (carries "academic / publication / taste" feel)
- **UI / code / data**: sans-serif + monospace

**A single typeface is always wrong.** Serif delivers "taste"; sans delivers
"function".

The actual typeface picks go through the brand spec (brand-spec.md's
Display / Body / Mono three-stack) or the Design Direction Advisor's 20
philosophies. This reference doesn't pick specific fonts — those are
**brand decisions**.

### 3.8 Focus Switching = Background Damping + Foreground Sharpening + Flash Guide

Focus switching is **not just** dropping opacity. The full recipe is:

```js
// Filter combo for unfocused elements
tile.style.filter = `
  brightness(${1 - 0.5 * focusIntensity})
  saturate(${1 - 0.3 * focusIntensity})
  blur(${focusIntensity * 4}px)        // ← key: only blur actually makes it "recede"
`;
tile.style.opacity = 0.4 + 0.6 * (1 - focusIntensity);

// After the focus lands, run a 150ms flash highlight at the focus point to lead the eye back
focusOverlay.animate([
  { background: 'rgba(255,255,255,0.3)' },
  { background: 'rgba(255,255,255,0)' }
], { duration: 150, easing: 'ease-out' });
```

**Why blur is mandatory**: with only opacity + brightness, unfocused elements
are still "sharp" and visually they don't "step into the background". blur(4-8px)
is what actually pushes non-focus a depth layer back.

---

## 4. Concrete Motion Techniques (Code Snippets You Can Copy Directly)

### 4.1 FLIP / Shared Element Transition

A button "expands" into an input field — it's **not** the button disappearing
plus a new panel appearing. The core is **the same DOM element** transitioning
between two states, not two elements cross-fading.

```jsx
// Using Framer Motion's layoutId
<motion.div layoutId="design-button">Design</motion.div>
// ↓ same layoutId after click
<motion.div layoutId="design-button">
  <input placeholder="Describe your design..." />
</motion.div>
```

For a native implementation see https://aerotwist.com/blog/flip-your-animations/

### 4.2 "Breathing" Expansion (width→height)

A panel doesn't open by **pulling width and height at the same time**. Instead:
- First 40% of the time: only pull width (keep height small)
- Last 60% of the time: hold width, push height open

This simulates the physical-world feel of "unroll first, then pour water in".

```js
const widthT = interpolate(t, [0, 0.4], [0, 1], Easing.expoOut);
const heightT = interpolate(t, [0.3, 1], [0, 1], Easing.expoOut);
style.width = `${widthT * targetW}px`;
style.height = `${heightT * targetH}px`;
```

### 4.3 Staggered Fade-up (30ms stagger)

When table rows, card columns, or list items enter, **delay each element by 30ms**
and animate `translateY` from 10px back to 0.

```js
rows.forEach((row, i) => {
  const localT = Math.max(0, t - i * 0.03);  // 30ms stagger
  row.style.opacity = interpolate(localT, [0, 0.3], [0, 1], Easing.expoOut);
  row.style.transform = `translateY(${
    interpolate(localT, [0, 0.3], [10, 0], Easing.expoOut)
  }px)`;
});
```

### 4.4 Non-Linear Breathing · Hover 0.5s Before the Key Result

Machines execute fast and continuously, but **hover for 0.5s before key results
appear** to give the viewer's brain reaction time.

```jsx
// Typical scenario: AI finishes generating → 0.5s hover → result surfaces
<Sprite start={8} end={8.5}>
  {/* 0.5s pause — nothing moves, let the viewer stare at the loading state */}
  <LoadingState />
</Sprite>
<Sprite start={8.5} end={10}>
  <ResultAppear />
</Sprite>
```

**Anti-example**: AI finishes and instantly seam-cuts to the result — no
reaction time for the viewer, information is lost.

### 4.5 Chunk Reveal · Simulate Token Streaming

When AI generates text, **don't use `setInterval` to pop in one character at a
time** (looks like old-movie subtitles). Use **chunk reveal** instead — 2-5
characters at a time, irregular intervals, simulating real token streaming.

```js
// Chunk by chunk, not character by character
const chunks = text.split(/(\s+|,\s*|\.\s*|;\s*)/);  // split by word + punctuation
let i = 0;
function reveal() {
  if (i >= chunks.length) return;
  element.textContent += chunks[i++];
  const delay = 40 + Math.random() * 80;  // irregular 40-120ms
  setTimeout(reveal, delay);
}
reveal();
```

### 4.6 Anticipation → Action → Follow-through

Three of Disney's 12 principles. Anthropic uses them very explicitly:

- **Anticipation**: a small reverse motion before the action begins (a button slightly shrinks before it pops)
- **Action**: the main motion itself
- **Follow-through**: a settling resonance after the action ends (a card lands then slightly bounces)

```js
// All three stages of a card entry
const anticip = interpolate(t, [0, 0.2], [1, 0.95], Easing.easeIn);     // anticipation
const action  = interpolate(t, [0.2, 0.7], [0.95, 1.05], Easing.expoOut); // action
const settle  = interpolate(t, [0.7, 1], [1.05, 1], Easing.spring);       // settle
// Final scale = product of the three or applied piecewise
```

**Anti-example**: animation with only Action and no Anticipation + Follow-through
looks like "PowerPoint animation".

### 4.7 3D Perspective + translateZ Layering

To get the "tilted 3D + floating cards" feel, add perspective to the container
and give individual elements different translateZ values:

```css
.stage-wrap {
  perspective: 2400px;
  perspective-origin: 50% 30%;  /* slight high-angle view */
}
.card-grid {
  transform-style: preserve-3d;
  transform: rotateX(8deg) rotateY(-4deg);  /* the golden ratio */
}
.card:nth-child(3n) { transform: translateZ(30px); }
.card:nth-child(5n) { transform: translateZ(-20px); }
.card:nth-child(7n) { transform: translateZ(60px); }
```

**Why rotateX 8° / rotateY -4° is the golden ratio**:
- Greater than 10° → elements feel too distorted, like they're "tipping over"
- Less than 5° → looks like "skew" rather than "perspective"
- The asymmetric 8° × -4° simulates the natural angle of "the camera looking down from the top-left corner of a desk"

### 4.8 Diagonal Pan · Move X and Y Together

Camera motion isn't pure up/down or pure left/right — it's **moving X and Y at
the same time** to simulate diagonal travel:

```js
const panX = Math.sin(flowT * 0.22) * 40;
const panY = Math.sin(flowT * 0.35) * 30;
stage.style.transform = `
  translate(-50%, -50%)
  rotateX(8deg) rotateY(-4deg)
  translate3d(${panX}px, ${panY}px, 0)
`;
```

**Key point**: X and Y use different frequencies (0.22 vs 0.35) to avoid the
Lissajous loop becoming regular.

---

## 5. Scenario Recipes (Three Narrative Templates)

The three reference videos correspond to three product personalities. **Pick the
one that fits your product best** — don't mix them.

### Recipe A · Apple Keynote Dramatic (Claude Design type)

**Best for**: major version launches, hero animations, visual-wow-first
**Rhythm**: Slow-Fast-Boom-Stop with a strong arc
**Easing**: `expoOut` throughout + a sprinkle of `overshoot`
**SFX density**: high (~0.4/s), SFX pitches tuned to the BGM scale
**BGM**: IDM / minimalist tech-electronic — calm and precise
**Closer**: hard camera pull-back → drop → Logo morph → ethereal single tone → abrupt stop

### Recipe B · One-Shot Tool Style (Claude Code type)

**Best for**: developer tools, productivity apps, flow-state scenarios
**Rhythm**: continuous, steady flow, no obvious peaks
**Easing**: `spring` physics + `expoOut`
**SFX density**: **0** (rely entirely on BGM to drive the edit rhythm)
**BGM**: Lo-fi Hip-hop / Boom-bap, 85-90 BPM
**Core technique**: land key UI actions on the BGM kick/snare transients —
"**the musical groove is the interaction sound effect**"

### Recipe C · Office Productivity Narrative (Claude for Word type)

**Best for**: enterprise software, document / spreadsheet / calendar tools,
professionalism-first
**Rhythm**: multi-scene hard cuts + Dolly In/Out
**Easing**: `overshoot` (toggles) + `expoOut` (panels)
**SFX density**: medium (~0.3/s), mainly UI clicks
**BGM**: Jazzy Instrumental, minor key, BPM 90-95
**Core highlight**: one scene must have a "whole-film peak" — 3D pop-out /
floating off the plane

---

## 6. Anti-Examples · This Is AI Slop

| Anti-pattern | Why it's wrong | Correct approach |
|---|---|---|
| `transition: all 0.3s ease` | `ease` is a cousin of linear, every element moves at the same rate | `expoOut` + per-element stagger |
| Every entry is `opacity 0→1` | No sense of motion direction | Pair with `translateY 10→0` + Anticipation |
| Logo fade-in | No narrative closing impact | Morph / Converge / collapse-and-expand |
| Cursor moves in a straight line | Subconscious "machine" feel | Bezier arc + Perlin Noise |
| Typing pops one character at a time (setInterval) | Looks like old-movie subtitles | Chunk Reveal with random intervals |
| No hover before key result | Viewer has no reaction time | 0.5s hover before the result |
| Focus switching only changes opacity | Out-of-focus elements still look sharp | opacity + brightness + **blur** |
| Pure black / pure white background | Cyber feel / reflective fatigue | Neutral with color temperature (per brand spec) |
| All animation moves at the same speed | No rhythm | Slow-Fast-Boom-Stop |
| Fade-out endings | No sense of decision | Abrupt stop (hold the last frame) |

---

## 7. Self-Check Checklist (60 Seconds Before Delivery)

- [ ] Is the narrative structure Slow-Fast-Boom-Stop, not uniform rhythm?
- [ ] Is the default easing `expoOut`, not `easeOut` or `linear`?
- [ ] Are toggles / button pops using `overshoot`?
- [ ] Do card / list entries have a 30ms stagger?
- [ ] Is there a 0.5s hover before key results?
- [ ] Does typing use Chunk Reveal, not setInterval per character?
- [ ] Does focus switching add blur (not just opacity)?
- [ ] Does the logo land via Morph reveal, not fade-in?
- [ ] Is the background not pure black / pure white (has color temperature)?
- [ ] Do you have a serif + sans-serif typography hierarchy?
- [ ] Does it end with an abrupt stop, not a fade-out?
- [ ] (If there's a cursor) is the cursor path an arc, not a straight line?
- [ ] Does SFX density match the product personality (see Recipes A/B/C)?
- [ ] Is there a 6-8dB loudness gap between BGM and SFX? (see `audio-design-rules.md`)

---

## 8. Relationship to Other References

| reference | Role | Relationship |
|---|---|---|
| `animation-pitfalls.md` | Technical pitfalls (16 entries) | "**Don't do it this way**" · the inverse of this file |
| `animations.md` | Stage / Sprite engine usage | The foundation of **how to write** animation |
| `audio-design-rules.md` | Two-track audio rules | The rules for **scoring** animation |
| `sfx-library.md` | The 37-SFX catalog | The SFX **asset library** |
| `apple-gallery-showcase.md` | Apple Gallery showcase style | A focused study of one specific motion style |
| **This file** | The positive grammar of motion design | "**Do it this way**" |

**Invocation order**:
1. First go through SKILL.md Step 3's four positioning questions (settle the narrative role and visual temperature)
2. Once you've chosen a direction, read this file to lock in the **motion language** (Recipes A/B/C)
3. When writing code, consult `animations.md` and `animation-pitfalls.md`
4. When exporting video, go through `audio-design-rules.md` + `sfx-library.md`

---

## Appendix · Sources for This File

- Anthropic official animation teardown: `reference-animations/BEST-PRACTICES.md` in Huashu's project directory
- Anthropic audio teardown: `AUDIO-BEST-PRACTICES.md` in the same directory
- 3 reference videos: `ref-{1,2,3}.mp4` plus the matching `gemini-ref-*.md` / `audio-ref-*.md`
- **Strict filtering**: this reference does not include any specific brand color values, type names, or product names.
  Color / type decisions go through §1.a Core Asset Protocol or the 20 design philosophies.
