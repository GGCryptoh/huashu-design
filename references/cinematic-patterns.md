# Cinematic Patterns · Best Practices for Workflow Demos

> 5 key patterns for upgrading from "PPT animation" to "keynote-grade cinematic."
> Distilled from the two cinematic demos in the 2026-04 "Let's Talk Skill" deck (Nuwa workflow + Darwin workflow); production-tested and reproducible.

---

## 0 · What problem does this doc solve?

When you need to make a "demo animation that walks through a workflow" (typical scenarios: skill workflows, product onboarding, API call flows, agent task execution), there are two common approaches:

| Paradigm | What it looks like | Outcome |
|---|---|---|
| **PPT animation** (bad) | step 1 fade in → step 2 fade in → step 3 fade in; 4 boxes laid out on the same screen | Viewers feel "this is just a PPT with fade effects" — no wow moment |
| **Cinematic** (good) | Scene-based; only one thing in focus at a time; transitions between scenes are dissolve / focus pull / morph | Viewers feel "this is a clip from a product launch" and want to screenshot-share it |

The root of the difference is **not animation tech** — it's **narrative paradigm**. This doc explains how to upgrade from the former to the latter.

---

## 1 · Five Core Patterns

### Pattern A · Dashboard + Cinematic Overlay double-layer structure

**Problem**: a pure cinematic defaults to a black screen + a ▶ button. If a user lands on this slide and doesn't click, they see nothing.

**Solution**:
```
DEFAULT state (always visible): the complete static workflow dashboard
  └── Viewers can immediately see how the skill / workflow runs

POINT ▶ trigger (overlay floats in): 22-second cinematic
  └── Plays once; auto fades back to DEFAULT when done
```

**Implementation**:
- `.dash` is visible by default, `.cinema` is `opacity: 0; pointer-events: none`
- `.play-cta` is a small gold button at the bottom-right (not a giant central overlay)
- Click → `cinema.classList.add('show')` + `dash.classList.add('hide')`
- Run once with `requestAnimationFrame` (not in a loop); on end, `endCinematic()` reverses state

**Anti-pattern**: default = central giant ▶ overlay covering everything; the page is blank until clicked.

---

### Pattern B · Scene-based, NOT step-based

**Problem**: splitting the animation into "step 1 appears → step 2 appears → ..." is PPT thinking.

**Solution**: split into 5 scenes, each one an **independent shot**, full-screen, focused on a single thing:

| Scene type | Role | Length |
|---|---|---|
| 1 · Invoke | User input triggers (terminal typewriter) | 3–4s |
| 2 · Process | Visualization of the core workflow (unique visual language) | 5–6s |
| 3 · Result/Insight | The key extracted output (visualized) | 4–5s |
| 4 · Output | Real artifact display (file / diff / number) | 3–4s |
| 5 · Hero Reveal | Closing hero moment (huge type + value prop) | 4–5s |

**Total ≈ 22 seconds** — tested as the golden length:
- Under 18s: PMs haven't entered focus mode before it ends
- Over 25s: lose patience
- 22s is just enough to "hook → unfold → close → leave an impression"

**Implementation**:
- `T = { DURATION: 22.0, s1_in: [0, 0.7], s2_in: [3.8, 4.6], ... }` global timeline
- A single `requestAnimationFrame(render)` runs opacity / transform calc for all scenes
- Don't use `setTimeout` chains (easy to break, hard to debug)
- Easing must be `expoOut` / `easeOut` / cubic-bezier — **never linear**

---

### Pattern C · Each demo's visual language must be independent

**Problem**: after building the first cinematic, you get lazy and reuse the same template for the second (same orbit + pentagon + typewriter + hero big-type) and only swap the copy.

**Consequence**: viewers notice the two skills "look identical" — which is equivalent to telling them "these two skills are not different."

**Solution**: each workflow has a different core metaphor, so the visual language must be different.

**Comparison**:

| Dimension | Nuwa (distill a person) | Darwin (optimize a skill) |
|---|---|---|
| Core metaphor | Collect → distill → write | Loop → evaluate → ratchet |
| Visual motion | Floating / radiating / pentagon | Looping / rising / contrasting |
| Scene 2 | 3D Orbit · 8 archive items floating in a perspective ellipse | Spin Loop · token runs 5 laps along a 6-node ring |
| Scene 3 | Pentagon · 5 tokens radiate from the center | v1 vs v5 · side-by-side diff (red v1 vs gold v5) |
| Scene 4 | SKILL.md typewriter | Hill-Climb · full-screen curve drawn |
| Scene 5 hero | "21 minutes" serif italic huge text | Spinning gear ⚙ + "KEPT +1.1" gold tag |

**Test**: cover up the copy; just looking at the visuals, can you tell which demo this is? If not, it's lazy.

---

### Pattern D · Use real AI-generated assets, not emoji or hand-drawn SVG

**Problem**: a 3D orbit / gallery needs floating asset fragments. Emoji (📚🎤) is ugly and brand-less; hand-drawn SVG book spines never look like real books.

**Solution**: use `huashu-gpt-image` to generate one 4×2 grid (8 themed objects · white background · 60px breathing space · unified style), then use `extract_grid.py --mode bbox` to cut out 8 transparent PNGs.

**Prompt principles** (full prompt patterns in the `huashu-gpt-image` skill):
- IP anchor ("1960s Caltech archive aesthetic" / "Hearthstone-style consistent treatment")
- White background (easy to cut; gray adds atmosphere but transparent extraction is hard)
- 4×2, not 5×5 (avoid the last-row compression bug)
- Persona finishing ("You are a Wired magazine curator preparing an exhibition photo")

**Anti-pattern**: emoji as icons; CSS silhouettes substituting for product shots.

---

### Pattern E · Dual-bus BGM + SFX

**Problem**: only animation, no audio — viewers subconsciously feel "this looks like a poor demo."

**Solution**: BGM long bed + 11 SFX cues.

**General SFX cue recipe** (works for workflow demos):

| Time | SFX | Trigger scenario |
|---|---|---|
| 0.10s | whoosh | Terminal rises from below |
| 3.0s | enter | Typewriter finishes; press enter |
| 4.0s | slide-in | Scene 2 elements enter |
| 5–9s × 5 times | sparkle | Key process beats (each generation / each token / each datapoint) |
| 14s | click | Switch to output scene |
| 17.8s | logo-reveal | Hero reveal moment |
| typewriter | type | One trigger every 2 characters (don't go too dense) |

**Frequency separation**: BGM volume 0.32 (low-frequency floor noise), SFX volume 0.55 (mid/high-frequency punch), sparkle 0.7 (must stand out), logo-reveal 0.85 (the strongest hero moment).

**User control**:
- Must have a ▶ start overlay (browser autoplay restrictions)
- A small mute button in the upper-right (user can mute anytime)
- Don't make it "force-play when user opens this slide"

---

## 2 · Designing the Static Dashboard

The dashboard is layer 1 of the dual structure — a PM can grok the skill without ever clicking ▶.

**Layout**: 3-column grid (or 1 large + 2 small); each panel solves one question:

| Panel type | Question it answers | Example |
|---|---|---|
| **Pipeline / Flow Diagram** | "What's the workflow of this skill?" | Nuwa 4-stage pipeline · Darwin autoresearch loop |
| **Snapshot / State** | "What does the actual run output look like?" | Darwin 8-dim rubric snapshot |
| **Trajectory / Evolution** | "How does it change across multiple runs?" | Darwin 5-generation hill-climb curve |
| **Examples / Gallery** | "What has it produced before?" | Nuwa 21 personas gallery |
| **Strip · Example I/O** | "Input → output" | Nuwa example strip: `› nuwa distill feynman → feynman.skill (21 min)` |

**Key constraints**:
- Information density must be sufficient (each panel carries differentiated info)
- But no data slop (every number must be meaningful)
- Color scheme consistent with the cinematic (same palette, transitions feel seamless)

---

## 3 · Debug & Dev Tools

Any long animation needs three dev tools, otherwise debugging will explode.

### Tool 1 · `?seek=N` freezes at second N

```js
const seek = parseFloat(params.get('seek'));
if (!isNaN(seek)) {
  started = true; muted = true;
  frozenT = seek;  // render() uses this t instead of elapsed
  cinema.classList.add('show'); dash.classList.add('hide');
}

// inside render():
let t = frozenT !== null ? frozenT : (elapsed % T.DURATION);
```

Usage: `http://.../slide.html?seek=12` jumps directly to the frame at 12s — no need to wait for playback.

### Tool 2 · `?autoplay=1` skips the ▶ overlay

Useful for Playwright auto-screenshots and for force-launching when embedded as iframe.

### Tool 3 · Manual REPLAY button

Small button at the upper-right; user / debug can replay arbitrarily. CSS:

```css
.replay{position:absolute;top:18px;right:18px;background:rgba(212,165,116,0.1);
  border:1px solid rgba(212,165,116,0.3);color:#D4A574;
  font-family:monospace;font-size:10px;letter-spacing:.28em;text-transform:uppercase;
  padding:6px 12px;border-radius:1px;cursor:pointer;backdrop-filter:blur(6px);z-index:6}
```

---

## 4 · iframe Embedding Pitfalls (when cinematic is embedded in a deck)

### Pitfall 1 · Parent window's click zone intercepts the iframe's button

If the deck's index.html added "left/right 22vw transparent click zones for paging," they will **cover the ▶ play button inside the iframe** — clicks get swallowed as "next slide."

**Fix**: give the click zones `top: 12vh; bottom: 25vh`, leaving 25% of the top and bottom free, so the iframe's central ▶ and bottom-right ▶ both remain clickable.

### Pitfall 2 · iframe steals focus and keyboard events are lost

Once the user clicks inside the iframe, focus is in the iframe; the parent window misses ←/→ keyboard events.

**Fix**:
```js
iframe.addEventListener('load', () => {
  // inject keyboard forwarder
  const doc = iframe.contentDocument;
  doc.addEventListener('keydown', (e) => {
    window.dispatchEvent(new KeyboardEvent('keydown', { key: e.key, ... }));
  });
  // pull focus back to parent after click
  doc.addEventListener('click', () => setTimeout(() => window.focus(), 0));
});
```

### Pitfall 3 · file:// vs https:// behavioral differences

A cinematic that works locally on file:// may break after deployment, because:
- file:// — iframe contentDocument is same-origin
- https:// — also same-origin (if same host), but audio autoplay restrictions are stricter

**Fix**:
- Before deploying, run `python3 -m http.server` to test on local HTTP
- BGM must `bgm.play()` only after user clicks ▶; don't play immediately on page-load

---

## 5 · Anti-pattern Quick Reference

| Anti-pattern | Correct pattern |
|---|---|
| Default = black screen ▶ overlay | Default = static dashboard; ▶ is auxiliary |
| 4 steps laid out in a row, fade in | 5 full-screen scene switches; each scene focuses on one thing |
| Reuse template, swap copy, call it a different demo | Each demo has independent visual language (cover the copy, can still tell apart) |
| emoji / hand-drawn SVG as assets | gpt-image-2 grid + extract_grid extraction |
| No BGM, no SFX | BGM + 11 SFX cues, dual-bus |
| Schedule with `setTimeout` chain | `requestAnimationFrame` + global timeline `T` object |
| Linear animation | Expo / cubic-bezier easing |
| No dev tools | `?seek=N` + `?autoplay=1` + REPLAY button |
| Buttons inside iframe swallowed by parent click zone | Click zone gets top/bottom margin to make room |

---

## 6 · Time Budget

Following these patterns, a complete cinematic demo (with dashboard):

| Task | Time |
|---|---|
| Design 5-scene narrative + visual language | 30 min (deliberate — determines independence) |
| Dashboard static layout + content | 1 hr |
| Cinematic 5-scene implementation | 1.5 hr |
| Audio cue timing + replay button | 30 min |
| Playwright screenshots verifying 5 key moments | 15 min |
| **Single-demo total** | **3–4 hours** |

The second demo reuses the framework but **the visual language must be independent** — about 2–3 hours.
