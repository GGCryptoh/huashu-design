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

**为什么这个模板对**：

| 环节 | 为什么必须这样 |
|------|-------------|
| `lastTick = null` + 首帧 `return` | 避免「脚本加载到 tick 首次执行」的 L 秒被算进动画时间 |
| `playing = false` 默认 | 字体加载期间 `tick` 即使运行也不推进 time，避免渲染错位 |
| `__ready` 在 tick 首帧设 | 录屏脚本此刻开始计时，对应的画面是动画真正的 t=0 |
| `document.fonts.ready.then(...)` 里才启动 tick | 规避字体 fallback 宽度测量、避免首帧字体跳变 |
| `window.__seek` 存在 | 让 `render-video.js` 可以主动矫正——第二道防线 |

**录屏脚本端的对应防御**：
1. `addInitScript` 注入 `window.__recording = true`（先于 page goto）
2. `waitForFunction(() => window.__ready === true)`，记录此刻偏移作为 ffmpeg trim
3. **额外**：`__ready` 之后主动 `page.evaluate(() => window.__seek && window.__seek(0))`，把 HTML 可能的 time 偏差强制归零——这是第二道防线，对付不严格遵守 starter 模板的 HTML

**验证方法**：导出 MP4 后
```bash
ffmpeg -i video.mp4 -ss 0 -vframes 1 frame-0.png
ffmpeg -i video.mp4 -ss $DURATION-0.1 -vframes 1 frame-end.png
```
首帧必须是动画 t=0 的初始状态（不是中段，不是黑），末帧必须是动画终态（不是第二轮 loop 的某个时刻）。

**参考实现**：`assets/animations.jsx` 的 Stage 组件、`scripts/render-video.js` 都已按此协议实现。手写 HTML 必须套 starter tick 模板——每一行都是防过具体 bug。

## 13. 录制时禁止 loop —— `window.__recording` 信号

**踩的坑**：动画 Stage 默认 `loop=true`（浏览器里方便看效果）。`render-video.js` 录完 duration 秒还多等 300ms 缓冲才停止，这 300ms 让 Stage 进入下一循环。ffmpeg `-t DURATION` 截取时，最后 0.5-1s 落入下一循环——视频结尾突然回到第一帧（Scene 1），观众以为视频出 bug。

**根因**：录制脚本和 HTML 之间没有"我在录制"的握手协议。HTML 不知道自己被录，依然按浏览器交互场景循环。

**规则**：

1. **录制脚本**：在 `addInitScript` 里注入 `window.__recording = true`（先于 page goto）：
   ```js
   await recordCtx.addInitScript(() => { window.__recording = true; });
   ```

2. **Stage 组件**：识别这个信号，强制 loop=false：
   ```js
   const effectiveLoop = (typeof window !== 'undefined' && window.__recording) ? false : loop;
   // ...
   if (next >= duration) return effectiveLoop ? 0 : duration - 0.001;
   //                                                       ↑ 留 0.001 防止 Sprite end=duration 被关掉
   ```

3. **结尾 Sprite 的 fadeOut**：录制场景下应设 `fadeOut={0}`，否则视频末尾会渐变到透明/暗色——用户期望停在清晰的最后一帧，不是淡出。手写 HTML 时建议结尾 Sprite 都用 `fadeOut={0}`。

**参考实现**：`assets/animations.jsx` 的 Stage / `scripts/render-video.js` 都已内置握手。手写 Stage 必须实现 `__recording` 检测——否则录制必踩这个坑。

**验证**：导出 MP4 后 `ffmpeg -ss 19.8 -i video.mp4 -frames:v 1 end.png`，检查倒数 0.2 秒是否还是预期最后一帧，没有突然切换到另一个 scene。

## 14. 60fps 视频默认用帧复制 —— minterpolate 兼容性差

**踩的坑**：`convert-formats.sh` 用 `minterpolate=fps=60:mi_mode=mci...` 生成的 60fps MP4，在 macOS QuickTime / Safari 部分版本下无法打开（一片黑或直接拒打）。VLC / Chrome 能打开。

**根因**：minterpolate 输出的 H.264 elementary stream 包含某些播放器解析有问题的 SEI / SPS 字段。

**规则**：

- 默认 60fps 用简单 `fps=60` filter（帧复制），兼容性广（QuickTime/Safari/Chrome/VLC 都能开）
- 高质量插帧用 `--minterpolate` flag 显式启用——但**必须本地测过**目标播放器再交付
- 60fps 标签价值是**上传平台的算法识别**（Bilibili / YouTube 上 60fps 标记会优先推流），实际感知流畅度对 CSS 动画来说提升微弱
- 加 `-profile:v high -level 4.0` 提升 H.264 通用兼容性

**`convert-formats.sh` 已默认改成兼容模式**。如果你需要插帧高质量，加 `--minterpolate` flag：
```bash
bash convert-formats.sh input.mp4 --minterpolate
```

## 15. `file://` + 外部 `.jsx` 的 CORS 陷阱 —— 单文件交付必须内联引擎

**踩的坑**：动画 HTML 里用 `<script type="text/babel" src="animations.jsx"></script>` 外部加载引擎。本机双击打开（`file://` 协议）→ Babel Standalone 走 XHR 拉 `.jsx` → Chrome 报 `Cross origin requests are only supported for protocol schemes: http, https, chrome, chrome-extension...` → 整页黑屏，不报 `pageerror` 只报 console error，很容易当"动画没触发"误诊。

启 HTTP server 也未必救得了——本机有全局代理时 `localhost` 也会走代理，返回 502 / 连接失败。

**规则**：

- **单文件交付（双击打开即用的 HTML）** → `animations.jsx` 必须**内联**到 `<script type="text/babel">...</script>` 标签内，不要用 `src="animations.jsx"`
- **多文件项目（起 HTTP server 演示）** → 可以外部加载，但交付时明确写清 `python3 -m http.server 8000` 命令
- 判断标准：交付给用户的是"HTML 文件"还是"带 server 的项目目录"？前者用内联
- Stage 组件 / animations.jsx 经常 200+ 行——贴进 HTML `<script>` 块完全可接受，别怕体积

**最小验证**：双击你生成的 HTML，**不要**通过任何 server 打开。如果 Stage 正常显示动画首帧，才算通过。

## 16. 跨 scene 反色上下文 —— 画面内元素不要硬编码颜色

**踩的坑**：做多场景动画时，`ChapterLabel` / `SceneNumber` / `Watermark` 等**跨 scene 都出现**的元素，在组件里写死 `color: '#1A1A1A'`（深色文字）。前 4 个 scene 浅底 OK，到第 5 个黑底 scene 时"05"和水印直接消失——不报错、不触发任何检查、关键信息隐形。

**规则**：

- **跨多 scene 复用的画面内元素**（chapter 标签 / scene 编号 / 时间码 / 水印 / 版权条）**禁止硬编码颜色值**
- 改用三种方式之一：
  1. **`currentColor` 继承**：元素只写 `color: currentColor`，父 scene 容器设 `color: 计算值`
  2. **invert prop**：组件接受 `<ChapterLabel invert />` 手动切换深浅
  3. **基于底色自动计算**：`color: contrast-color(var(--scene-bg))`（CSS 4 新 API，或 JS 判断）
- 交付前用 Playwright 抽**每个 scene 的代表帧**，人眼过一遍"跨 scene 元素"是否都可见

这条坑的隐蔽性在于——**没有 bug 报警**。只有人眼或 OCR 能发现。

## 快速自查清单（开工前 5 秒）

- [ ] 每个 `position: absolute` 的父元素都有 `position: relative`？
- [ ] 动画里的特殊字符（`␣` `⌘` `emoji`）都在字体里存在？
- [ ] Grid/Flex 模板的 count 和 JS 数据的 length 一致？
- [ ] 场景切换之间有 cross-fade，没有 >0.3s 的纯空白？
- [ ] DOM 测量代码包在 `document.fonts.ready.then()` 里？
- [ ] `render(t)` 是 pure 的，或有明确的 reset 机制？
- [ ] 第 0 帧是完整初始状态，不是空白？
- [ ] 画面内没有「伪 chrome」装饰（进度条/时间码/底部署名条与 Stage scrubber 撞车）？
- [ ] 动画 tick 第一帧同步设 `window.__ready = true`？（用 animations.jsx 自带；手写 HTML 自己加）
- [ ] Stage 检测 `window.__recording` 强制 loop=false？（手写 HTML 必加）
- [ ] 结尾 Sprite 的 `fadeOut` 设为 0（视频末尾停清晰帧）？
- [ ] 60fps MP4 默认用帧复制模式（兼容性），高质量插帧才加 `--minterpolate`？
- [ ] 导出后抽第 0 帧 + 末帧验证是动画初始/最终状态？
- [ ] 涉及具体品牌（Stripe/Anthropic/Lovart/...）：走完了「品牌资产协议」（SKILL.md §1.a 五步）？有没有写 `brand-spec.md`？
- [ ] 单文件交付的 HTML：`animations.jsx` 是内联的，不是 `src="..."`？（file:// 下 external .jsx 会 CORS 黑屏）
- [ ] 跨 scene 出现的元素（chapter 标签/水印/scene 编号）没有硬编码颜色？在每个 scene 底色下都可见？
