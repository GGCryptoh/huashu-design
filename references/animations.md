# Animations: Timeline Animation Engine

Read this when building animation / motion-design HTML. Principles, usage, and typical patterns.

## Core Pattern: Stage + Sprite

Our animation system (`assets/animations.jsx`) provides a timeline-driven engine:

- **`<Stage>`**: the container for the whole animation; auto-provides auto-scale (fit viewport) + scrubber + play/pause/loop controls
- **`<Sprite start end>`**: a time slice. A Sprite only renders during `start`–`end`. Inside, you read your own local progress `t` (0→1) via the `useSprite()` hook
- **`useTime()`**: reads the current global time (seconds)
- **`Easing.easeInOut` / `Easing.easeOut` / ...`**: easing functions
- **`interpolate(t, from, to, easing?)`**: interpolates by t

This pattern borrows from Remotion / After Effects — but lightweight, zero dependencies.

## Getting Started

```html
<script type="text/babel" src="animations.jsx"></script>
<script type="text/babel">
  const { Stage, Sprite, useTime, useSprite, Easing, interpolate } = window.Animations;

  function Title() {
    const { t } = useSprite();  // local progress 0→1
    const opacity = interpolate(t, [0, 1], [0, 1], Easing.easeOut);
    const y = interpolate(t, [0, 1], [40, 0], Easing.easeOut);
    return (
      <h1 style={{ 
        opacity, 
        transform: `translateY(${y}px)`,
        fontSize: 120,
        fontWeight: 900,
      }}>
        Hello.
      </h1>
    );
  }

  function Scene() {
    return (
      <Stage duration={10}>  {/* 10-second animation */}
        <Sprite start={0} end={3}>
          <Title />
        </Sprite>
        <Sprite start={2} end={5}>
          <SubTitle />
        </Sprite>
        {/* ... */}
      </Stage>
    );
  }

  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<Scene />);
</script>
```

## Common Animation Patterns

### 1. Fade In / Fade Out

```jsx
function FadeIn({ children }) {
  const { t } = useSprite();
  const opacity = interpolate(t, [0, 0.3], [0, 1], Easing.easeOut);
  return <div style={{ opacity }}>{children}</div>;
}
```

**Range note**: `[0, 0.3]` means the fade-in completes in the first 30% of the sprite, and stays opacity=1 after that.

### 2. Slide In

```jsx
function SlideIn({ children, from = 'left' }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, 0.4], [0, 1], Easing.easeOut);
  const offset = (1 - progress) * 100;
  const directions = {
    left: `translateX(-${offset}px)`,
    right: `translateX(${offset}px)`,
    top: `translateY(-${offset}px)`,
    bottom: `translateY(${offset}px)`,
  };
  return (
    <div style={{
      transform: directions[from],
      opacity: progress,
    }}>
      {children}
    </div>
  );
}
```

### 3. Character-by-character typewriter

```jsx
function Typewriter({ text }) {
  const { t } = useSprite();
  const charCount = Math.floor(text.length * Math.min(t * 2, 1));
  return <span>{text.slice(0, charCount)}</span>;
}
```

### 4. Count up

```jsx
function CountUp({ from = 0, to = 100, duration = 0.6 }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, duration], [0, 1], Easing.easeOut);
  const value = Math.floor(from + (to - from) * progress);
  return <span>{value.toLocaleString()}</span>;
}
```

### 5. Phased explanation (typical educational animation)

```jsx
function Scene() {
  return (
    <Stage duration={20}>
      {/* Phase 1: present the problem */}
      <Sprite start={0} end={4}>
        <Problem />
      </Sprite>

      {/* Phase 2: present the approach */}
      <Sprite start={4} end={10}>
        <Approach />
      </Sprite>

      {/* Phase 3: present the result */}
      <Sprite start={10} end={16}>
        <Result />
      </Sprite>

      {/* Caption visible the entire time */}
      <Sprite start={0} end={20}>
        <Caption />
      </Sprite>
    </Stage>
  );
}
```

## Easing Functions

Preset easing curves:

| Easing | Behavior | Use for |
|--------|------|------|
| `linear` | constant speed | scrolling captions, sustained motion |
| `easeIn` | slow → fast | exits / disappears |
| `easeOut` | fast → slow | entrances / appearances |
| `easeInOut` | slow → fast → slow | position changes |
| **`expoOut`** ⭐ | **exponential ease-out** | **Anthropic-grade primary easing** (physical weight) |
| **`overshoot`** ⭐ | **bounce-back** | **toggles / button pops / emphasized interactions** |
| `spring` | spring | interactive feedback, geometry settling |
| `anticipation` | reverse first, then forward | emphasis motion |

**Default primary easing is `expoOut`** (not `easeOut`) — see `animation-best-practices.md` §2.
Entrance with `expoOut`, exit with `easeIn`, toggle with `overshoot` — the foundational rule of Anthropic-grade animation.

## Pacing and Duration Guide

### Micro-interactions (0.1–0.3s)
- Button hover
- Card expand
- Tooltip appear

### UI transitions (0.3–0.8s)
- Page change
- Modal appear
- List item enter

### Narrative animation (2–10s per phase)
- One phase of a concept explanation
- A data-chart reveal
- A scene transition

### Single-narrative animation: max 10s per phase
Human attention is finite. Ten seconds for one thing — when it's done, move to the next.

## Order of Thinking When Designing an Animation

### 1. Content / story first, animation second

**Wrong**: think "I want a fancy animation" first, then stuff content into it
**Right**: figure out what message you're communicating, then use animation to serve it

Animation is a **signal**, not **decoration**. A fade-in says "this is important, look here" — if everything fades in, the signal goes flat.

### 2. Write the timeline by scene

```
0:00 - 0:03   Problem appears (fade in)
0:03 - 0:06   Problem zooms / unfolds (zoom + pan)
0:06 - 0:09   Solution appears (slide in from right)
0:09 - 0:12   Solution explained (typewriter)
0:12 - 0:15   Result demo (counter up + chart reveal)
0:15 - 0:18   One-line summary (static, hold 3 seconds)
0:18 - 0:20   CTA or fade out
```

Write the timeline first, then write components.

### 3. Assets first

Images / icons / fonts the animation needs — prepare them **before** building. Don't go hunting for assets halfway through; it kills the rhythm.

## Common Issues

**Animation stutters**
→ Mostly layout thrashing. Use `transform` and `opacity`; don't animate `top` / `left` / `width` / `height` / `margin`. Browsers GPU-accelerate `transform`.

**Animation too fast, can't read it**
→ Reading a Chinese character takes 100–150ms; a word 300–500ms. If you're telling a story with text, give each line at least 3 seconds.

**Animation too slow, viewer gets bored**
→ Interesting visual changes should be dense. A static frame longer than 5 seconds gets boring.

**Multiple animations interfere**
→ Tell the browser an element will animate via `will-change: transform` to reduce reflow.

**Recording to video**
→ Use the skill's built-in toolchain (one command, three formats): see `video-export.md`
- `scripts/render-video.js` — HTML → 25fps MP4 (Playwright + ffmpeg)
- `scripts/convert-formats.sh` — 25fps MP4 → 60fps MP4 + optimized GIF
- Need more precise frame rendering? Make `render(t)` a pure function — see `animation-pitfalls.md` item 5

## Pairing With Video Tools

This skill builds **HTML animations** (running in a browser). If the final output is video material:

- **Short animations / concept demos**: use the methods here for HTML animation → screen-record
- **Long video / narrative**: this skill focuses on HTML animation; for long video use an AI video-generation skill or pro video software
- **Motion graphics**: pro tools (After Effects / Motion Canvas) are a better fit

## On Popmotion etc.

If you really need physics-based animation (spring, decay, keyframes with precise timing) and our engine can't handle it, you can fall back to Popmotion:

```html
<script src="https://unpkg.com/popmotion@11.0.5/dist/popmotion.min.js"></script>
```

But **try our engine first**. It's enough 90% of the time.
