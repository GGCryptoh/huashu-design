# Video Export: HTML Animation → MP4 / GIF

After the animation HTML is done, users often ask "can you export it as video?" This guide gives the full flow.

## When to Export

**Export timing**:
- The animation runs cleanly start-to-finish; visual verification is done (Playwright screenshots confirm correct state at each timestamp)
- The user has watched it in the browser at least once and is OK with it
- **Don't** export while there are unfixed animation bugs — fixing things in video is more expensive

**Trigger phrases the user might say**:
- "Can you export it as video?"
- "Convert to MP4"
- "Make a GIF"
- "60fps"

## Output Specs

By default, deliver three formats and let the user choose:

| Format | Spec | Best for | Typical size (30s) |
|---|---|---|---|
| MP4 25fps | 1920×1080 · H.264 · CRF 18 | WeChat embed, Channels, YouTube | 1–2 MB |
| MP4 60fps | 1920×1080 · minterpolate · H.264 · CRF 18 | High-frame-rate showcase, Bilibili, portfolio | 1.5–3 MB |
| GIF | 960×540 · 15fps · palette-optimized | Twitter/X, README, Slack preview | 2–4 MB |

## Toolchain

Two scripts in `scripts/`:

### 1. `render-video.js` — HTML → MP4

Records a baseline 25fps MP4. Depends on a global Playwright install.

```bash
NODE_PATH=$(npm root -g) node /path/to/claude-design/scripts/render-video.js <html-file>
```

Optional flags:
- `--duration=30` animation length (seconds)
- `--width=1920 --height=1080` resolution
- `--trim=2.2` seconds to cut from the start (drops reload + font-load time)
- `--fontwait=1.5` font-load wait (seconds); raise it if there are many fonts

Output: same directory as the HTML, with the same basename plus `.mp4`.

### 2. `add-music.sh` — MP4 + BGM → MP4

Mixes BGM into a silent MP4. Pick by mood from the built-in BGM library, or supply your own. Auto-matches duration; adds fade in/out.

```bash
bash add-music.sh <input.mp4> [--mood=<name>] [--music=<path>] [--out=<path>]
```

**Built-in BGM library** (in `assets/bgm-<mood>.mp3`):

| `--mood=` | Style | Best for |
|-----------|------|---------|
| `tech` (default) | Apple Silicon / Apple-keynote — minimal synth + piano | Product launches, AI tools, skill promo |
| `ad` | Upbeat modern electronic with build + drop | Social ads, product teasers, promo videos |
| `educational` | Warm, bright, light guitar / electric piano, inviting | Explainer, tutorial intros, course teasers |
| `educational-alt` | Same family, alternative track | Same |
| `tutorial` | Lo-fi ambient, almost imperceptible | Software demo, coding tutorials, long demos |
| `tutorial-alt` | Same family, alternative track | Same |

**Behavior**:
- Music is trimmed to video length
- 0.3s fade in + 1s fade out (no hard cut)
- Video stream `-c:v copy` (no re-encode); audio AAC 192k
- `--music=<path>` overrides `--mood`; lets you specify any external audio
- Wrong mood name lists all valid options; doesn't fail silently

**Typical pipeline** (animation export trio + music):
```bash
node render-video.js animation.html                        # record
bash convert-formats.sh animation.mp4                      # derive 60fps + GIF
bash add-music.sh animation-60fps.mp4                      # add default tech BGM
# Or scenario-specific:
bash add-music.sh tutorial-demo.mp4 --mood=tutorial
bash add-music.sh product-promo.mp4 --mood=ad --out=promo-final.mp4
```

### 3. `convert-formats.sh` — MP4 → 60fps MP4 + GIF

Generates a 60fps version and a GIF from an existing MP4.

```bash
bash /path/to/claude-design/scripts/convert-formats.sh <input.mp4> [gif_width] [--minterpolate]
```

Outputs (alongside input):
- `<name>-60fps.mp4` — defaults to `fps=60` frame duplication (broad compatibility); add `--minterpolate` to enable high-quality interpolation
- `<name>.gif` — palette-optimized GIF (default 960 wide, configurable)

**60fps mode selection**:

| Mode | Command | Compatibility | When to use |
|---|---|---|---|
| Frame duplication (default) | `convert-formats.sh in.mp4` | Plays in QuickTime / Safari / Chrome / VLC | General delivery, platform uploads, social |
| minterpolate | `convert-formats.sh in.mp4 --minterpolate` | macOS QuickTime / Safari may refuse to open it | Bilibili and other showcases that need real interpolation; **always test in the target player before delivery** |

Why default to frame duplication? minterpolate's H.264 elementary stream has a known compat bug — we hit "macOS QuickTime won't open it" multiple times when minterpolate was the default. See `animation-pitfalls.md` §14.

`gif_width` parameter:
- 960 (default) — universal for social
- 1280 — sharper but bigger
- 600 — Twitter/X loads it faster

## Standard Recommended Flow

After the user says "export the video":

```bash
cd <project-dir>

# Assume $SKILL points to this skill's root (replace with your install path)

# 1. Record the 25fps base MP4
NODE_PATH=$(npm root -g) node "$SKILL/scripts/render-video.js" my-animation.html

# 2. Derive 60fps MP4 and GIF
bash "$SKILL/scripts/convert-formats.sh" my-animation.mp4

# Output manifest:
# my-animation.mp4         (25fps · 1-2 MB)
# my-animation-60fps.mp4   (60fps · 1.5-3 MB)
# my-animation.gif         (15fps · 2-4 MB)
```

## Technical Details (for debugging)

### Playwright recordVideo gotchas

- Frame rate is locked at 25fps; cannot record 60fps directly (Chromium headless compositor cap)
- Recording starts from context creation, so you must `trim` to drop the loading time
- Default format is webm; ffmpeg conversion to H.264 MP4 needed for general playback

`render-video.js` already handles all of this.

### ffmpeg minterpolate parameters

Current config: `minterpolate=fps=60:mi_mode=mci:mc_mode=aobmc:me_mode=bidir:vsbmc=1`

- `mi_mode=mci` — motion compensation interpolation
- `mc_mode=aobmc` — adaptive overlapped block motion compensation
- `me_mode=bidir` — bidirectional motion estimation
- `vsbmc=1` — variable-size block motion compensation

Works well for CSS **transform animations** (translate / scale / rotate).
For **pure fades** it can produce mild ghosting — if the user dislikes it, fall back to simple frame duplication:

```bash
ffmpeg -i input.mp4 -r 60 -c:v libx264 ... output.mp4
```

### Why GIF palette uses two passes

GIF is limited to 256 colors. A single-pass GIF squashes the whole animation onto one generic 256-color palette — washing out subtle palettes like beige base + orange.

Two passes:
1. `palettegen=stats_mode=diff` — scan the entire clip and generate **an optimal palette specific to this animation**
2. `paletteuse=dither=bayer:bayer_scale=5:diff_mode=rectangle` — encode with that palette; `rectangle` diff only updates changing regions, drastically shrinking the file

For fade transitions, `dither=bayer` is smoother than `none`, but the file is slightly bigger.

## Pre-flight Check (before export)

30-second self-check before export:

- [ ] HTML has been run start-to-finish in a browser; no console errors
- [ ] Frame 0 is the complete initial state (not a blank loading screen)
- [ ] Final frame is a stable closing state (not cut off mid-animation)
- [ ] Fonts / images / emoji all render correctly (see `animation-pitfalls.md`)
- [ ] `duration` parameter matches the actual animation length in HTML
- [ ] Stage in HTML detects `window.__recording` and forces `loop=false` (must verify on hand-rolled Stage; built into `assets/animations.jsx`)
- [ ] Final Sprite has `fadeOut={0}` (last video frame doesn't fade)
- [ ] Includes "Created by Huashu-Design" watermark (mandatory for animation scenes; for third-party brand work prepend "Unofficial · ". See SKILL.md §"Skill Promo Watermark")

## Delivery Note

Standard format for the note you give the user after export:

```
**Full delivery**

| File | Format | Spec | Size |
|---|---|---|---|
| foo.mp4 | MP4 | 1920×1080 · 25fps · H.264 | X MB |
| foo-60fps.mp4 | MP4 | 1920×1080 · 60fps (motion-interpolated) · H.264 | X MB |
| foo.gif | GIF | 960×540 · 15fps · palette-optimized | X MB |

**Notes**
- 60fps uses minterpolate for motion-estimation interpolation; great for transform animations
- GIF uses palette optimization; a 30s animation comes out around 3MB

Want a different size or frame rate? Tell me.
```

## Common Follow-up Requests

| User says | Response |
|---|---|
| "Too big" | MP4: bump CRF to 23–28; GIF: drop resolution to 600 or fps to 10 |
| "GIF is too blurry" | Bump `gif_width` to 1280; or suggest MP4 (WeChat Moments supports it) |
| "I want vertical 9:16" | Change the HTML source to `--width=1080 --height=1920` and re-record |
| "Add a watermark" | ffmpeg with `-vf "drawtext=..."` or `overlay=` a PNG |
| "Transparent background" | MP4 doesn't support alpha; use WebM VP9 + alpha, or APNG |
| "Lossless" | Set CRF 0 + preset veryslow (file becomes 10× larger) |
