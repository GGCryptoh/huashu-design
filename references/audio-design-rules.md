# Audio Design Rules · huashu-design

> The audio recipe applied to every animation demo. Pairs with `sfx-library.md` (the asset list).
> Hammered out in production: huashu-design hero v1–v9 iterations · Gemini deep dissection of three Anthropic official films · 8000+ A/B comparisons

---

## Core Principle · Dual-bus Audio (Iron Rule)

Animation audio **must be designed in two independent layers**, not one:

| Layer | Role | Time scale | Relation to visuals | Frequency band |
|---|---|---|---|---|
| **SFX (beat layer)** | Marks every visual beat | 0.2–2s short | **Strong sync** (frame-aligned) | **High freq 800Hz+** |
| **BGM (ambient bed)** | Emotional bed, sound field | Continuous 20–60s | Loose sync (paragraph-level) | **Mid/low freq <4kHz** |

**An animation with only BGM is crippled** — viewers subconsciously feel "stuff is moving but not making sound" — and that's the root of the cheap feel.

---

## Gold Standard · Golden Ratios

These numbers are **engineering-grade hard parameters** measured from Anthropic's three official films + our own v9 final cut. Apply them directly:

### Volume
- **BGM volume**: `0.40–0.50` (relative to full scale 1.0)
- **SFX volume**: `1.00`
- **Loudness gap**: BGM **-6 to -8 dB** below SFX peak (don't make SFX absolute-loud — make the gap)
- **amix parameter**: `normalize=0` (never `normalize=1`, which flattens dynamic range)

### Frequency separation (P1 hard optimization)
Anthropic's secret isn't "loud SFX"; it's **frequency-band separation**:

```bash
[bgm_raw]lowpass=f=4000[bgm]      # Limit BGM to <4kHz (mid/low)
[sfx_raw]highpass=f=800[sfx]      # Push SFX to 800Hz+ (mid/high)
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]
```

Why: human ears are most sensitive to 2–5kHz (the "presence band"). If SFX live in this band and BGM covers the full spectrum, **BGM's high-frequency content masks SFX**. Highpass on SFX + lowpass on BGM puts each in its own region of the spectrum, and SFX clarity jumps a tier.

### Fades
- BGM in: `afade=in:st=0:d=0.3` (0.3s — avoid hard cut)
- BGM out: `afade=out:st=N-1.5:d=1.5` (1.5s long tail — sense of closure)
- SFX has its own envelope; no extra fade needed

---

## SFX Cue Design Rules

### Density (how many SFX per 10s)
The three Anthropic films have three density tiers:

| Film | SFX per 10s | Product personality | Scenario |
|---|---|---|---|
| Artifacts (ref-1) | **~9 / 10s** | Feature-dense, info-heavy | Complex tool demo |
| Code Desktop (ref-2) | **0** | Pure ambient, meditative | Dev tool flow state |
| Word (ref-3) | **~4 / 10s** | Balanced, office rhythm | Productivity tool |

**Heuristic**:
- Calm / focused product personality → low SFX density (0–3 / 10s); BGM-driven
- Lively / info-heavy personality → high SFX density (6–9 / 10s); SFX-driven rhythm
- **Don't fill every visual beat** — restraint is more sophisticated than density. **Cutting 30–50% of cues makes the rest more dramatic.**

### Cue selection priority
Not every visual beat needs SFX. Pick by this priority:

**P0 mandatory** (omitting feels off):
- Typing (terminal / input)
- Click / select (user decision moments)
- Focus shift (visual lead change)
- Logo reveal (brand close)

**P1 recommended**:
- Element entrance / exit (modal / card)
- Completion / success feedback
- AI generation start / end
- Major transitions (scene change)

**P2 optional** (overuse muddies things):
- hover / focus-in
- Progress ticks
- Decorative ambient

### Timestamp alignment precision
- **Same-frame alignment** (0ms error): click / focus shift / logo landing
- **Lead by 1–2 frames** (-33ms): fast whoosh (gives the viewer a psychological cue)
- **Lag by 1–2 frames** (+33ms): object landing / impact (matches real physics)

---

## BGM Selection Decision Tree

The huashu-design skill ships with 6 BGM tracks (`assets/bgm-*.mp3`):

```
What's the animation's personality?
├─ Product launch / tech demo → bgm-tech.mp3 (minimal synth + piano)
├─ Tutorial / tool walkthrough → bgm-tutorial.mp3 (warm, instructional)
├─ Education / explanation → bgm-educational.mp3 (curious, thoughtful)
├─ Marketing / brand promo → bgm-ad.mp3 (upbeat, promotional)
└─ Same vibe, need a variant → bgm-*-alt.mp3 (alt version of each)
```

### When no BGM (worth considering)
See Anthropic Code Desktop (ref-2): **0 SFX + pure lo-fi BGM** can also feel premium.

**When to pick no BGM**:
- Animation length <10s (BGM doesn't have time to establish)
- Product personality is "focused / meditative"
- The scene already has ambient / voiceover audio
- Very high SFX density (avoid auditory overload)

---

## Scene Recipes (Plug-and-play)

### Recipe A · Product launch hero (huashu-design v9 same-spec)
```
Length: 25s
BGM: bgm-tech.mp3 · 45% · band <4kHz
SFX density: ~6 / 10s

Cues:
  Terminal typing → type × 4 (0.6s spacing)
  Enter           → enter
  Cards converge  → card × 4 (staggered 0.2s)
  Select          → click
  Ripple          → whoosh
  4 focuses       → focus × 4
  Logo            → thud (1.5s)

Volume: BGM 0.45 / SFX 1.0 · amix normalize=0
```

### Recipe B · Tool feature demo (reference: Anthropic Code Desktop)
```
Length: 30–45s
BGM: bgm-tutorial.mp3 · 50%
SFX density: 0–2 / 10s (very few)

Strategy: let BGM + voiceover drive; SFX only at **decisive moments** (file save / command-execute completion)
```

### Recipe C · AI generation demo
```
Length: 15–20s
BGM: bgm-tech.mp3, or no BGM
SFX density: ~8 / 10s (high)

Cues:
  User input → type + enter
  AI starts processing → magic/ai-process (1.2s loop)
  Generation complete → feedback/complete-done
  Result appears → magic/sparkle
  
Highlight: ai-process can loop 2–3 times spanning the whole generation
```

### Recipe D · Pure ambient long take (reference: Artifacts)
```
Length: 10–15s
BGM: none
SFX: use 3–5 carefully designed cues alone

Strategy: each SFX is the lead — no BGM "muddying" issue.
Good for: single-product slow-motion, close-up showcase
```

---

## ffmpeg Composition Templates

### Template 1 · Single SFX onto video
```bash
ffmpeg -y -i video.mp4 -itsoffset 2.5 -i sfx.mp3 \
  -filter_complex "[0:a][1:a]amix=inputs=2:normalize=0[a]" \
  -map 0:v -map "[a]" output.mp4
```

### Template 2 · Multi-SFX timeline composition (cue-aligned)
```bash
ffmpeg -y \
  -i sfx-type.mp3 -i sfx-enter.mp3 -i sfx-click.mp3 -i sfx-thud.mp3 \
  -filter_complex "\
[0:a]adelay=1100|1100[a0];\
[1:a]adelay=3200|3200[a1];\
[2:a]adelay=7000|7000[a2];\
[3:a]adelay=21800|21800[a3];\
[a0][a1][a2][a3]amix=inputs=4:duration=longest:normalize=0[mixed]" \
  -map "[mixed]" -t 25 sfx-track.mp3
```
**Key params**:
- `adelay=N|N`: first is left-channel delay (ms), second is right; write twice to keep stereo aligned
- `normalize=0`: preserves dynamic range — critical!
- `-t 25`: truncates to a specific length

### Template 3 · Video + SFX track + BGM (with frequency separation)
```bash
ffmpeg -y -i video.mp4 -i sfx-track.mp3 -i bgm.mp3 \
  -filter_complex "\
[2:a]atrim=0:25,afade=in:st=0:d=0.3,afade=out:st=23.5:d=1.5,\
     lowpass=f=4000,volume=0.45[bgm];\
[1:a]highpass=f=800,volume=1.0[sfx];\
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]" \
  -map 0:v -map "[a]" -c:v copy -c:a aac -b:a 192k final.mp4
```

---

## Failure Mode Cheat Sheet

| Symptom | Root cause | Fix |
|---|---|---|
| Can't hear SFX | BGM high-frequency content masks it | Add `lowpass=f=4000` to BGM + `highpass=f=800` to SFX |
| SFX too loud / harsh | SFX absolute volume too high | Drop SFX volume to 0.7 and BGM to 0.3 — preserve the gap |
| BGM and SFX rhythm clash | Wrong BGM (used music with strong beat) | Switch to ambient / minimal-synth BGM |
| BGM cuts off abruptly at end | No fade-out | `afade=out:st=N-1.5:d=1.5` |
| SFX overlap into mush | Cues too dense + each SFX too long | Keep each SFX under 0.5s; cue spacing ≥0.2s |
| WeChat mp4 has no audio | WeChat sometimes mutes auto-play | No worries — clicking gets audio; gifs never have audio anyway |

---

## Linking with Visuals (Advanced)

### SFX timbre must match the visual style
- Warm beige / paper-like visuals → use **wooden / soft** SFX timbres (Morse, paper snap, soft click)
- Cold black-tech visuals → use **metallic / digital** SFX timbres (beep, pulse, glitch)
- Hand-drawn / playful visuals → use **cartoon / exaggerated** timbres (boing, pop, zap)

The current beige base in `apple-gallery-showcase.md` → pair with `keyboard/type.mp3` (mechanical) + `container/card-snap.mp3` (soft) + `impact/logo-reveal-v2.mp3` (cinematic bass).

### SFX can drive the visual rhythm
Advanced trick: **design the SFX timeline first, then adjust the visual animation to align with the SFX** (not the other way around).
Each SFX cue is a "clock tick"; visuals adapted to SFX rhythm feel rock-solid. The opposite — chasing visuals with SFX — has a ±1-frame mismatch problem that always feels off.

---

## Quality Checklist (Pre-release self-check)

- [ ] Loudness gap: SFX peak − BGM peak = -6 to -8 dB?
- [ ] Frequency: BGM lowpass 4kHz + SFX highpass 800Hz?
- [ ] amix normalize=0 (preserve dynamic range)?
- [ ] BGM fade-in 0.3s + fade-out 1.5s?
- [ ] SFX count appropriate (density per scenario personality)?
- [ ] Each SFX same-frame aligned with the visual beat (within ±1 frame)?
- [ ] Logo-reveal SFX long enough (1.5s recommended)?
- [ ] Listen with BGM off: do the SFX alone have rhythm?
- [ ] Listen with SFX off: does BGM alone have emotional flow?

Each layer must stand on its own. If only the combined mix sounds good, you didn't design well.

---

## References

- SFX asset list: `sfx-library.md`
- Visual style reference: `apple-gallery-showcase.md`
- Deep audio analysis of three Anthropic films: `/Users/alchain/Documents/writing/01-wechat-public-account/projects/2026.04-huashu-design-launch/reference-animations/AUDIO-BEST-PRACTICES.md`
- huashu-design v9 production case: `/Users/alchain/Documents/writing/01-wechat-public-account/projects/2026.04-huashu-design-launch/figures/hero-animation-v9-final.mp4`
