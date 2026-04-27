# Verification: Output Verification Workflow

Some design-agent native environments (like Claude.ai Artifacts) ship with a built-in `fork_verifier_agent` that spawns a subagent to inspect via iframe screenshots. Most agent environments (Claude Code / Codex / Cursor / Trae / etc.) don't have this built-in capability — using Playwright manually covers the same verification scenarios.

## Verification Checklist

Every time you produce an HTML, run through this checklist:

### 1. Browser Render Check (Mandatory)

The most basic question: **does the HTML open at all?** On macOS:

```bash
open -a "Google Chrome" "/path/to/your/design.html"
```

Or use Playwright screenshots (next section).

### 2. Console Error Check

The most common issue in HTML files is JS errors causing a blank page. Run Playwright once:

```bash
python ~/.claude/skills/claude-design/scripts/verify.py path/to/design.html
```

This script:
1. Opens the HTML in headless chromium
2. Saves a screenshot to the project directory
3. Captures console errors
4. Reports status

See `scripts/verify.py` for details.

### 3. Multi-viewport Check

For responsive designs, capture multiple viewports:

```bash
python verify.py design.html --viewports 1920x1080,1440x900,768x1024,375x667
```

### 4. Interaction Check

Tweaks, animations, button toggles — static screenshots can't see them. **Recommend the user open a browser and click through themselves**, or record with Playwright:

```python
page.video.record('interaction.mp4')
```

### 5. Slide-by-slide Check

For deck-style HTML, capture each slide:

```bash
python verify.py deck.html --slides 10  # capture the first 10
```

Generates `deck-slide-01.png`, `deck-slide-02.png`... for quick browsing.

## Playwright Setup

First-time setup:

```bash
# If not already installed
npm install -g playwright
npx playwright install chromium

# Or the Python version
pip install playwright
playwright install chromium
```

If the user already has Playwright installed globally, just use it.

## Screenshot Best Practices

### Capture full page

```python
page.screenshot(path='full.png', full_page=True)
```

### Capture viewport

```python
page.screenshot(path='viewport.png')  # default captures only the visible area
```

### Capture a specific element

```python
element = page.query_selector('.hero-section')
element.screenshot(path='hero.png')
```

### High-resolution screenshot

```python
page = browser.new_page(device_scale_factor=2)  # retina
```

### Wait for animation to settle before capturing

```python
page.wait_for_timeout(2000)  # wait 2s for the animation to settle
page.screenshot(...)
```

## Sending Screenshots to the User

### Open the local screenshot directly

```bash
open screenshot.png
```

The user opens it in their own Preview / Figma / VSCode / browser.

### Upload to an image host and share a link

If a remote collaborator needs to see it (Slack / Lark / WeChat), have the user use their own image-host tool or MCP to upload:

```bash
python ~/Documents/writing/tools/upload_image.py screenshot.png
```

Returns a permanent ImgBB link you can paste anywhere.

## When Verification Fails

### Blank page

There's definitely a console error. Check first:

1. Whether the integrity hash on the React+Babel script tag is correct (see `react-setup.md`)
2. Whether `const styles = {...}` has a naming collision
3. Whether cross-file components are exported to `window`
4. JSX syntax errors (babel.min.js doesn't report them — switch to the unminified babel.js)

### Animation stutters

- Record a clip with the Chrome DevTools Performance tab
- Look for layout thrashing (frequent reflows)
- Prefer `transform` and `opacity` for motion (GPU-accelerated)

### Wrong fonts

- Check that the `@font-face` URL is reachable
- Check the fallback font
- Slow Chinese-font load: show fallback first, swap once loaded

### Layout misalignment

- Check whether `box-sizing: border-box` is applied globally
- Check the `* margin: 0; padding: 0` reset
- Open gridlines in Chrome DevTools to inspect the actual layout

## Verification = the Designer's Second Pair of Eyes

**Always go through it yourself.** Common AI-generated bugs:

- Looks correct but interaction is broken
- Static screenshot is fine but layout breaks on scroll
- Looks great wide but collapses narrow
- Forgot to test dark mode
- After Tweaks switch, some components don't react

**One minute of verification at the end saves an hour of rework.**

## Common Verification Script Commands

```bash
# Basic: open + screenshot + capture errors
python verify.py design.html

# Multi-viewport
python verify.py design.html --viewports 1920x1080,375x667

# Multi-slide
python verify.py deck.html --slides 10

# Output to a specific directory
python verify.py design.html --output ./screenshots/

# headless=false, opens a real browser for you to watch
python verify.py design.html --show
```
