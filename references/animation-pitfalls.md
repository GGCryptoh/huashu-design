# Animation Pitfalls: HTML Animation Bugs We Hit and the Rules That Came Out of Them

The bugs we hit most often when making animation, and how to avoid them. Every
rule here comes from a real failure case.

Reading this through before you start writing animation saves a round of
iteration.

## 1. Stacking Layout — `position: relative` Is the Default Obligation

**The bug we hit**: a sentence-wrap element wrapped 3 bracket-layers
(`position: absolute`). The sentence-wrap didn't have `position: relative` set,
so the absolute brackets used `.canvas` as their coordinate system and floated
off 200px below the screen.

**Rules**:
- Any container that holds `position: absolute` children **must** explicitly set `position: relative`
- Even if visually you don't need an "offset", write `position: relative` as the coordinate-system anchor
- If you're writing `.parent { ... }` and its children include `.child { position: absolute }`, instinctively give the parent `relative`

**Quick check**: every time you see a `position: absolute`, walk up its
ancestors and make sure the nearest positioned ancestor is the coordinate system
you *intended*.

## 2. Character Trap — Don't Rely on Rare Unicode

**The bug we hit**: we tried to use `␣` (U+2423 OPEN BOX) to visualize a
"space token". Neither Noto Serif SC nor Cormorant Garamond ships this glyph,
so it rendered as blank / a tofu box and the viewer couldn't see it at all.

**Rules**:
- **Every character that appears in your animation must exist in the typeface you've chosen**
- Common rare-character blacklist: `␣ ␀ ␐ ␋ ␨ ↩ ⏎ ⌘ ⌥ ⌃ ⇧ ␦ ␖ ␛`
- To express meta-characters like "space / return / tab", use a **CSS-constructed semantic box**:
  ```html
  <span class="space-key">Space</span>
  ```
  ```css
  .space-key {
    display: inline-flex;
    padding: 4px 14px;
    border: 1.5px solid var(--accent);
    border-radius: 4px;
    font-family: monospace;
    font-size: 0.3em;
    letter-spacing: 0.2em;
    text-transform: uppercase;
  }
  ```
- Verify emoji too: some emoji fall back to a gray square outside Noto Emoji, so prefer the `emoji` font-family or an SVG

## 3. Data-Driven Grid/Flex Templates

**The bug we hit**: code had `const N = 6` tokens, but CSS was hardcoded as
`grid-template-columns: 80px repeat(5, 1fr)`. The 6th token had no column and
the whole matrix was misaligned.

**Rules**:
- When the count comes from a JS array (`TOKENS.length`), the CSS template should also be data-driven
- Option A: inject from JS via a CSS variable
  ```js
  el.style.setProperty('--cols', N);
  ```
  ```css
  .grid { grid-template-columns: 80px repeat(var(--cols), 1fr); }
  ```
- Option B: use `grid-auto-flow: column` and let the browser expand automatically
- **Ban the "fixed number + JS constant" combo** — when N changes, CSS won't update with it

## 4. Transition Gap — Scene Cuts Must Be Continuous

**The bug we hit**: between zoom1 (13-19s) and zoom2 (19.2-23s), the main
sentence was already hidden. zoom1 fade out (0.6s) + zoom2 fade in (0.6s) +
stagger delay (0.2s+) added up to roughly 1 second of pure blank screen. The
viewer thought the animation had frozen.

**Rules**:
- When cutting between scenes continuously, fade out and fade in must **cross-overlap** — don't make the previous one fully disappear before the next one starts
  ```js
  // Bad:
  if (t >= 19) hideZoom('zoom1');      // 19.0s out
  if (t >= 19.4) showZoom('zoom2');    // 19.4s in → 0.4s blank in the middle

  // Good:
  if (t >= 18.6) hideZoom('zoom1');    // start fade out 0.4s earlier
  if (t >= 18.6) showZoom('zoom2');    // fade in at the same time (cross-fade)
  ```
- Or use an "anchor element" (e.g. the main sentence) as a visual link between scenes — let it briefly reappear during the zoom switch
- Match the CSS transition durations carefully so the next transition isn't triggered before the previous one finishes

## 5. The Pure Render Principle — Animation State Should Be Seekable

**The bug we hit**: we used `setTimeout` + `fireOnce(key, fn)` to chain-trigger
animation state. Normal playback was fine, but when frame-by-frame recording or
seeking to arbitrary time points, setTimeouts that had already fired couldn't
"go back in time".

**Rules**:
- The `render(t)` function should ideally be a **pure function**: given t, output a unique DOM state
- If you must use side effects (like toggling classes), pair a `fired` set with an explicit reset:
  ```js
  const fired = new Set();
  function fireOnce(key, fn) { if (!fired.has(key)) { fired.add(key); fn(); } }
  function reset() { fired.clear(); /* clear every .show class */ }
  ```
- Expose `window.__seek(t)` for Playwright / debugging:
  ```js
  window.__seek = (t) => { reset(); render(t); };
  ```
- Animation-related setTimeouts should not span >1 second, or seek-rewind will go haywire

## 6. Measuring Before Fonts Load = Wrong Measurements

**The bug we hit**: we called `charRect(idx)` to measure bracket positions on
DOMContentLoaded. Fonts hadn't loaded yet, so each character's width was the
fallback font's width — every position was wrong. Once fonts loaded (about
500ms later), the bracket's `left: Xpx` was still the old value, permanently
offset.

**Rules**:
- Any layout code that depends on DOM measurement (`getBoundingClientRect`, `offsetWidth`) **must** be wrapped in `document.fonts.ready.then()`
  ```js
  document.fonts.ready.then(() => {
    requestAnimationFrame(() => {
      buildBrackets(...);  // fonts are ready now, measurement is accurate
      tick();              // animation starts
    });
  });
  ```
- The extra `requestAnimationFrame` gives the browser one frame to commit layout
- If using Google Fonts CDN, `<link rel="preconnect">` to speed up first load

## 7. Recording Prep — Reserve Hooks for Video Export

**The bug we hit**: Playwright `recordVideo` defaults to 25fps and starts
recording the moment the context is created. The first 2 seconds of page load
and font load got recorded. The delivered video had a 2-second blank / white
flash up front.

**Rules**:
- Use a `render-video.js` tool that handles: warmup navigate → reload to restart the animation → wait duration → ffmpeg trim head + transcode to H.264 MP4
- **Frame 0** of the animation should be the complete initial state with final layout already in place (not blank, not loading)
- Want 60fps? Post-process with ffmpeg `minterpolate`. Don't depend on the browser's source frame rate
- Want GIF? Two-stage palette (`palettegen` + `paletteuse`) — can compress a 30s 1080p animation down to 3MB

See `video-export.md` for the full script invocation.

## 8. Batch Export — tmp Directories Must Carry the PID to Avoid Concurrency Collisions

**The bug we hit**: we ran `render-video.js` in 3 parallel processes to record
3 HTMLs. Because TMP_DIR was named only with `Date.now()`, when all 3 processes
started in the same millisecond they shared the same tmp directory. The first
process to finish cleaned up the tmp dir, the other two hit `ENOENT` reading
the directory, and everything crashed.

**Rules**:
- Any temporary directory that multiple processes might share must include **a PID or random suffix** in its name:
  ```js
  const TMP_DIR = path.join(DIR, '.video-tmp-' + Date.now() + '-' + process.pid);
  ```
- If you really want to parallelize across files, use the shell's `&` + `wait` rather than forking inside a single node script
- When batch-recording multiple HTMLs, the conservative move is: run them **serially** (up to 2 in parallel is fine; 3 or more — just queue them up)

## 9. Progress Bars / Replay Buttons in the Recording — Chrome Elements Polluting the Video

**The bug we hit**: the animation HTML had a `.progress` bar, a `.replay`
button, and a `.counter` timestamp added so humans could debug playback. When
recorded as MP4 and delivered, those elements appeared at the bottom of the
video — looked like dev tools had been screenshotted into the frame.

**Rules**:
- Manage the "chrome elements" in HTML that exist for humans (progress bar / replay button / footer / masthead / counter / phase labels) separately from the actual video content
- **Convention class name** `.no-record`: any element with this class is automatically hidden by the recording script
- The script side (`render-video.js`) injects CSS by default to hide common chrome class names:
  ```
  .progress .counter .phases .replay .masthead .footer .no-record [data-role="chrome"]
  ```
- Inject with Playwright's `addInitScript` (takes effect before every navigate, survives reload)
- When you want to see the raw HTML (with chrome), add the `--keep-chrome` flag

## 10. Animation Repeats in the First Few Seconds of the Recording — Warmup Frames Leaking In

**The bug we hit**: the old `render-video.js` flow was
`goto → wait fonts 1.5s → reload → wait duration`. Recording started the moment
the context was created, so during warmup the animation had already played a
chunk; after reload it restarted from 0. The first few seconds of the video
became "mid-animation + transition + animation from 0" — a strong sense of
repetition.

**Rules**:
- **Warmup and Record must use separate contexts**:
  - Warmup context (no `recordVideo` option): just loads the url, waits for fonts, then closes
  - Record context (with `recordVideo`): fresh state, animation recorded from t=0
- ffmpeg `-ss trim` can only shave off Playwright's tiny startup latency (~0.3s); it **cannot** be used to cover up warmup frames — the source has to be clean
- Closing the record context = the webm file gets written to disk. That's a Playwright constraint.
- The corresponding code pattern:
  ```js
  // Phase 1: warmup (throwaway)
  const warmupCtx = await browser.newContext({ viewport });
  const warmupPage = await warmupCtx.newPage();
  await warmupPage.goto(url, { waitUntil: 'networkidle' });
  await warmupPage.waitForTimeout(1200);
  await warmupCtx.close();

  // Phase 2: record (fresh)
  const recordCtx = await browser.newContext({ viewport, recordVideo });
  const page = await recordCtx.newPage();
  await page.goto(url, { waitUntil: 'networkidle' });
  await page.waitForTimeout(DURATION * 1000);
  await page.close();
  await recordCtx.close();
  ```

## 11. Don't Draw "Fake Chrome" Inside the Frame — Decorative Player UI Collides With Real Chrome

**The bug we hit**: the animation used the `Stage` component, which already
ships scrubber + timecode + pause button (which belong to `.no-record` chrome
and are hidden automatically on export). I then drew a "magazine-page-number-style
decorative progress bar" along the bottom of the frame —
`00:60 ──── CLAUDE-DESIGN / ANATOMY` — and felt great about it. **Result**:
the user saw two progress bars — one was the Stage controller, the other was my
decoration. Visually they collided completely; the user flagged it as a bug.
"Why is there another progress bar inside the video?"

**Rules**:

- Stage already provides: scrubber + timecode + pause/replay button. **Don't draw inside the frame** progress indicators, current timecode, copyright bylines, or chapter counters — they either collide with chrome or they're filler slop (violating the "earn its place" principle).
- "Magazine page-number feel", "magazine vibe", "bottom byline" — these **decorative urges** are high-frequency filler that AI adds automatically. Every time one shows up, be on guard: does it really carry information that nothing else could? Or is it just filling empty space?
- If you really believe a bottom strip must exist (e.g. the animation's subject is literally the player UI itself), it must be **narratively necessary** and **visually clearly distinct from the Stage scrubber** (different position, different form, different tone).

**Element ownership test** (every element you draw into the canvas must be able to answer):

| What it belongs to | Action |
|------------|------|
| The narrative content of a specific scene | OK, keep it |
| Global chrome (control / debug use) | Add the `.no-record` class, hidden on export |
| **Belongs to neither a scene nor chrome** | **Delete it.** This is a homeless element — necessarily filler slop. |

**Self-check (3 seconds before delivery)**: take a still and ask yourself —

- Is there anything in the frame "that looks like video player UI" (a horizontal progress bar, a timecode, a control-button shape)?
- If yes, would deleting it hurt the narrative? If not, delete it.
- Does the same kind of information (progress / time / byline) appear twice? Consolidate it into chrome in one place.

**Anti-examples**: drawing `00:42 ──── PROJECT NAME` along the bottom, drawing a "CH 03 / 06" chapter counter at the bottom-right, drawing a version number "v0.3.1" along the edge — all fake-chrome filler.

## 12. Leading Blank Frames + Start-Point Offset in Recordings — The `__ready` × tick × lastTick Triple Trap

**Bug A · Leading blank**: a 60-second animation exported to MP4 had 2-3
seconds of blank page at the front. `ffmpeg --trim=0.3` couldn't cut it out.

**Bug B · Start-point offset (real incident, 2026-04-20)**: a 24-second video
exported, but to the user it felt like "the first frame doesn't start until
second 19 of the video". What actually happened: recording started at t=5,
went through to t=24, then looped back to t=0 and recorded another 5 seconds
to the end — so the final 5 seconds of the video were the animation's true
beginning.

**Root cause** (both bugs share one root cause):

Playwright `recordVideo` starts writing the WebM the instant `newContext()` is
called. At that moment Babel / React / font loading collectively take L seconds
(2-6s). The recording script waits on `window.__ready = true` as the "animation
starts here" anchor — and it must be strictly paired with animation `time = 0`.
Two common mistakes:

| Mistake | Symptom |
|------|------|
| Setting `__ready` inside `useEffect` or synchronous setup (before tick's first frame) | Recording script thinks the animation started, but the WebM is still recording the blank page → **leading blank** |
| Initializing tick's `lastTick = performance.now()` at the **top level** of the script | The L seconds of font loading get counted into the first frame's `dt`, and `time` jumps instantly to L → the whole recording lags by L seconds → **start-point offset** |

**The correct full starter tick template** (hand-written animations must use this skeleton):

```js
// ━━━━━━ state ━━━━━━
let time = 0;
let playing = false;   // ❗ don't play by default; wait until fonts are ready
let lastTick = null;   // ❗ sentinel — on tick's first frame dt is forced to 0 (don't use performance.now())
const fired = new Set();

// ━━━━━━ tick ━━━━━━
function tick(now) {
  if (lastTick === null) {
    lastTick = now;
    window.__ready = true;   // ✅ pair: "recording start" and "animation t=0" on the same frame
    render(0);               // render once more to ensure DOM is ready (fonts are ready by now)
    requestAnimationFrame(tick);
    return;
  }
  const dt = (now - lastTick) / 1000;   // dt only starts advancing after the first frame
  lastTick = now;

  if (playing) {
    let t = time + dt;
    if (t >= DURATION) {
      t = window.__recording ? DURATION - 0.001 : 0;  // don't loop while recording; keep 0.001s to preserve the final frame
      if (!window.__recording) fired.clear();
    }
    time = t;
    render(time);
  }
  requestAnimationFrame(tick);
}

// ━━━━━━ boot ━━━━━━
// Don't fire rAF immediately at the top level — wait for fonts to load
document.fonts.ready.then(() => {
  render(0);                 // draw the initial frame first (fonts are ready)
  playing = true;
  requestAnimationFrame(tick);  // the first tick pairs __ready + t=0
});

// ━━━━━━ seek interface (for render-video defensive correction) ━━━━━━
window.__seek = (t) => { fired.clear(); time = t; lastTick = null; render(t); };
```

**Why this template is correct**:

| Mechanism | Why it has to be this way |
|------|-------------|
| `lastTick = null` + first-frame `return` | Prevents the L seconds between "script loaded" and "tick first executed" from being counted into animation time |
| `playing = false` by default | While fonts are loading, even if `tick` runs it doesn't advance time, avoiding render misalignment |
| `__ready` set on tick's first frame | The recording script starts its clock here, and the corresponding frame is the animation's real t=0 |
| Tick is only kicked off inside `document.fonts.ready.then(...)` | Avoids font-fallback width measurement and first-frame font-swap jumps |
| `window.__seek` exists | Lets `render-video.js` proactively correct — a second line of defense |

**Corresponding defenses on the recording-script side**:
1. `addInitScript` injects `window.__recording = true` (before page goto)
2. `waitForFunction(() => window.__ready === true)`, record the offset at this moment as the ffmpeg trim
3. **Extra**: after `__ready`, proactively call `page.evaluate(() => window.__seek && window.__seek(0))` to force any time drift in the HTML back to zero — second line of defense, for HTML that doesn't strictly follow the starter template

**Verification**: after exporting the MP4
```bash
ffmpeg -i video.mp4 -ss 0 -vframes 1 frame-0.png
ffmpeg -i video.mp4 -ss $DURATION-0.1 -vframes 1 frame-end.png
```
The first frame must be the animation's initial state at t=0 (not a mid section, not black). The last frame must be the animation's final state (not some moment from a second loop).

**Reference implementation**: `assets/animations.jsx`'s Stage component and `scripts/render-video.js` both already implement this protocol. Hand-written HTML must apply the starter tick template — every line in it defends against a specific bug.

## 13. No Looping During Recording — The `window.__recording` Signal

**The bug we hit**: Stage defaults to `loop=true` (handy in the browser for
previewing). `render-video.js` waits an extra 300ms buffer after the duration
finishes before stopping, and those 300ms let Stage roll into the next loop.
When ffmpeg `-t DURATION` clipped, the last 0.5-1s fell into the next loop —
the end of the video suddenly returned to the first frame (Scene 1) and the
viewer thought the video was buggy.

**Root cause**: there's no "I'm recording" handshake protocol between the
recording script and the HTML. The HTML doesn't know it's being recorded, so
it loops as if it were a browser interactive context.

**Rules**:

1. **Recording script**: inject `window.__recording = true` via `addInitScript` (before page goto):
   ```js
   await recordCtx.addInitScript(() => { window.__recording = true; });
   ```

2. **Stage component**: recognize this signal and force loop=false:
   ```js
   const effectiveLoop = (typeof window !== 'undefined' && window.__recording) ? false : loop;
   // ...
   if (next >= duration) return effectiveLoop ? 0 : duration - 0.001;
   //                                                       ↑ keep 0.001 so a Sprite with end=duration isn't turned off
   ```

3. **The trailing Sprite's fadeOut**: when recording, set `fadeOut={0}` — otherwise the end of the video fades to transparent/dark, but the user expects to land on a clear final frame, not a fade-out. When hand-writing HTML, prefer `fadeOut={0}` for trailing Sprites.

**Reference implementation**: `assets/animations.jsx`'s Stage / `scripts/render-video.js` both have the handshake built in. A hand-written Stage must implement `__recording` detection — otherwise recording will hit this bug for sure.

**Verification**: after exporting the MP4, `ffmpeg -ss 19.8 -i video.mp4 -frames:v 1 end.png`. Check whether the last 0.2 seconds is still the expected final frame, with no sudden switch to another scene.

## 14. Default 60fps Should Use Frame Duplication — minterpolate Has Poor Compatibility

**The bug we hit**: 60fps MP4 produced by `convert-formats.sh` with
`minterpolate=fps=60:mi_mode=mci...` couldn't be opened in some versions of
macOS QuickTime / Safari (all black, or refused to open). VLC / Chrome could
open it.

**Root cause**: minterpolate's H.264 elementary stream output contains certain
SEI / SPS fields that some players have trouble parsing.

**Rules**:

- Default 60fps uses the simple `fps=60` filter (frame duplication) for broad compatibility (works on QuickTime/Safari/Chrome/VLC)
- For high-quality interpolation, opt in explicitly with the `--minterpolate` flag — but **you must test the target players locally** before delivery
- The value of the 60fps tag is **the platform's algorithmic recognition on upload** (Bilibili / YouTube prioritize the 60fps tag in distribution); actual perceived smoothness gains for CSS animation are minor
- Add `-profile:v high -level 4.0` to improve H.264 universal compatibility

**`convert-formats.sh` now defaults to compatibility mode.** If you need
high-quality interpolation, add the `--minterpolate` flag:
```bash
bash convert-formats.sh input.mp4 --minterpolate
```

## 15. The `file://` + External `.jsx` CORS Trap — Single-File Delivery Must Inline the Engine

**The bug we hit**: the animation HTML loaded the engine externally with
`<script type="text/babel" src="animations.jsx"></script>`. Open it locally by
double-click (`file://` protocol) → Babel Standalone tries to XHR the `.jsx` →
Chrome throws `Cross origin requests are only supported for protocol schemes:
http, https, chrome, chrome-extension...` → entire page goes black. It doesn't
fire `pageerror`, only a console error, and is easily misdiagnosed as
"animation didn't trigger".

Spinning up an HTTP server doesn't necessarily save you either — when there's a
global proxy on the machine, `localhost` may also route through the proxy and
come back as 502 / connection failed.

**Rules**:

- **Single-file delivery (HTML that works on double-click)** → `animations.jsx` must be **inlined** inside a `<script type="text/babel">...</script>` tag — don't use `src="animations.jsx"`
- **Multi-file project (running an HTTP server for demo)** → external loading is fine, but spell out `python3 -m http.server 8000` clearly at delivery
- Decision criterion: are you delivering "an HTML file" or "a project directory with a server"? The former uses inlining
- The Stage component / animations.jsx is often 200+ lines — pasting it into an HTML `<script>` block is totally fine, don't worry about size

**Minimal verification**: double-click the HTML you generated. **Do not** open it through any server. If Stage shows the animation's first frame correctly, it passes.

## 16. Cross-Scene Inverted-Color Context — Don't Hardcode Colors on In-Frame Elements

**The bug we hit**: in a multi-scene animation, elements that **appear across
all scenes** like `ChapterLabel` / `SceneNumber` / `Watermark` had
`color: '#1A1A1A'` (dark text) hardcoded inside the component. The first 4
scenes had light backgrounds, fine. By scene 5 the background was black and
"05" and the watermark just vanished — no error, no check tripped, key
information invisible.

**Rules**:

- **In-frame elements reused across multiple scenes** (chapter labels / scene numbers / timecodes / watermarks / copyright bars) **must not hardcode color values**
- Use one of three approaches instead:
  1. **`currentColor` inheritance**: the element only writes `color: currentColor`, and the parent scene container sets `color: <computed value>`
  2. **invert prop**: the component accepts `<ChapterLabel invert />` to flip light/dark manually
  3. **Auto-computed from background**: `color: contrast-color(var(--scene-bg))` (the new CSS 4 API, or judge in JS)
- Before delivery, use Playwright to capture a **representative frame per scene** and eyeball whether the cross-scene elements are visible in all of them

The insidiousness of this pitfall is that **there's no bug alarm**. Only the
human eye (or OCR) can catch it.

## Quick Self-Check Checklist (5 Seconds Before You Start)

- [ ] Does every parent of a `position: absolute` element have `position: relative`?
- [ ] Do all special characters in the animation (`␣` `⌘` `emoji`) exist in the chosen typeface?
- [ ] Does the Grid/Flex template count match the length of the JS data?
- [ ] Are scene transitions cross-faded with no pure-blank gap >0.3s?
- [ ] Is DOM measurement code wrapped in `document.fonts.ready.then()`?
- [ ] Is `render(t)` pure, or does it have an explicit reset mechanism?
- [ ] Is frame 0 the complete initial state, not blank?
- [ ] No "fake chrome" decorations in the frame (progress bar / timecode / bottom byline colliding with the Stage scrubber)?
- [ ] Does the animation tick set `window.__ready = true` synchronously on the first frame? (Built into animations.jsx; for hand-written HTML you add it yourself)
- [ ] Does Stage detect `window.__recording` and force loop=false? (Mandatory for hand-written HTML)
- [ ] Is the trailing Sprite's `fadeOut` set to 0 (so the video ends on a clear frame)?
- [ ] Does the 60fps MP4 default to frame-duplication mode (compatibility), with `--minterpolate` only added for high-quality interpolation?
- [ ] After export, did you sample frame 0 and the final frame to verify they are the animation's initial / final states?
- [ ] When dealing with a specific brand (Stripe / Anthropic / Lovart / ...), did you complete the "Brand Asset Protocol" (SKILL.md §1.a, five steps)? Did you write `brand-spec.md`?
- [ ] For single-file HTML delivery: is `animations.jsx` inlined, not `src="..."`? (External `.jsx` over file:// blackscreens via CORS)
- [ ] Do elements that appear across scenes (chapter labels / watermarks / scene numbers) avoid hardcoded colors? Are they visible against each scene's background?
