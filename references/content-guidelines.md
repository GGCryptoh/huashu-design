# Content Guidelines: Anti-AI-slop, Content Rules, Scale Specs

The traps you fall into most easily in AI design. This is a "what NOT to do" list — more important than "what to do," because AI slop is the default and you'll produce it unless you actively avoid it.

## Full AI-slop Blacklist

### Visual traps

**Aggressive gradient backgrounds**
- Purple → pink → blue full-screen gradient (signature smell of AI-generated webpages)
- Rainbow gradient in any direction
- Mesh gradient covering the whole background
- If you must use gradient: subtle, monochromatic, deployed with intent (e.g., button hover)

**Rounded card + left border-accent color**
```css
/* The signature of AI-flavored cards */
.card {
  border-radius: 12px;
  border-left: 4px solid #3b82f6;
  padding: 16px;
}
```
This card design is everywhere in AI-generated dashboards. Want emphasis? Use a more designed approach: background color contrast, weight / size contrast, plain dividers, or skip cards entirely.

**Emoji decoration**
Unless the brand itself uses emoji (Notion, Slack), don't put emoji in the UI. **Especially avoid**:
- 🚀 ⚡️ ✨ 🎯 💡 in front of titles
- ✅ in feature lists
- → inside CTA buttons (a standalone arrow is OK; an emoji arrow is not)

If you don't have icons, use a real icon library (Lucide / Heroicons / Phosphor), or use placeholders.

**SVG-drawn imagery**
Don't try to draw with SVG: people, scenes, devices, objects, abstract art. AI-drawn SVG imagery is recognizable in one glance — childish and cheap. **A gray rectangle + the text label "illustration slot 1200×800" beats a clumsy SVG hero illustration 100×.**

The only places SVG is allowed:
- Real icons (16×16 to 32×32 size)
- Geometric shapes as decoration
- Charts in data viz

**Excessive iconography**
Not every title / feature / section needs an icon. Overusing icons makes the interface look like a toy. Less is more.

**"Data slop"**
Fabricated stats as decoration:
- "10,000+ happy customers" (you don't even know if it's true)
- "99.9% uptime" (don't write it without real data)
- Decorative "metric cards" composed of icon + number + label
- Mock tables stuffed with fake data and ornamental flourishes

If you don't have real data, leave a placeholder or ask the user.

**"Quote slop"**
Fabricated user reviews / celebrity quotes decorating the page. Leave a placeholder; ask the user for real quotes.

### Typography traps

**Avoid these clichéd fonts**:
- Inter (default for AI-generated webpages)
- Roboto
- Arial / Helvetica
- Pure system font stack
- Fraunces (AI discovered it and now overuses it)
- Space Grotesk (the recent AI favorite)

**Use distinctive display + body pairings.** Direction options:
- Serif display + sans body (editorial feel)
- Mono display + sans body (technical feel)
- Heavy display + light body (contrast)
- Variable font for hero weight animation

Font sources:
- Underused good options on Google Fonts (Instrument Serif, Cormorant, Bricolage Grotesque, JetBrains Mono)
- Open-source font sites (Fraunces sister fonts, Adobe Fonts)
- Don't invent font names from memory

### Color traps

**Inventing colors from scratch**
Don't design a whole unfamiliar color system from zero. It usually doesn't harmonize.

**Strategy**:
1. Have a brand color → use it; fill missing color tokens by interpolating in oklch
2. No brand color but have a reference → eyedropper from the reference product's screenshots
3. Fully from zero → pick a known color system (Radix Colors / Tailwind default palette / Anthropic brand); don't tune your own

**Defining colors with oklch** is the modern approach:
```css
:root {
  --primary: oklch(0.65 0.18 25);      /* warm terracotta */
  --primary-light: oklch(0.85 0.08 25); /* same hue, lighter */
  --primary-dark: oklch(0.45 0.20 25);  /* same hue, darker */
}
```
oklch keeps the hue stable when you adjust lightness, unlike hsl.

**Casually flipping colors for dark mode**
It's not a simple invert. Good dark mode requires re-tuning saturation, contrast, and accent. If you don't want to do dark mode, don't.

### Layout traps

**Bento-grid overuse**
Every AI-generated landing page wants to bento. Unless your information structure genuinely fits bento, use a different layout.

**Big hero + 3-column features + testimonials + CTA**
This landing page template is worn out. Want to be original? Actually be original.

**Every card in a card grid identical**
Asymmetric, varied sizes, some with images and some text-only, some spanning columns — that's what real designers do.

## Content Guidelines

### 1. Don't add filler content

Every element must earn its place. Whitespace is a design problem solved by **composition** (contrast, rhythm, breathing room) — **not** by stuffing in content.

**How to spot filler**:
- If you removed this content, would the design get worse? If "no," remove it.
- What real problem does this element solve? If it's "to make the page less empty," delete it.
- Does this stat / quote / feature have real data backing it? If not, don't fabricate.

"One thousand no's for every yes."

### 2. Ask before adding material

You think adding a paragraph / page / section would be better? Ask the user first; don't add unilaterally.

Reasons:
- The user knows their audience better than you
- Adding content has a cost; the user may not want it
- Unilateral additions violate the "junior designer reporting work" relationship

### 3. Create a system up front

After exploring design context, **state the system you'll use out loud** and let the user confirm:

```markdown
My design system:
- Color: #1A1A1A body + #F0EEE6 background + #D97757 accent (from your brand)
- Type: Instrument Serif for display + Geist Sans for body
- Rhythm: section titles use full-bleed colored backgrounds + white text; ordinary sections use white background
- Imagery: hero uses a full-bleed photo; feature sections use placeholders pending your assets
- At most 2 background colors — avoid clutter

Confirm this direction and I'll start.
```

Wait for confirmation before starting. This check-in prevents "halfway in, realized direction was wrong."

## Scale Specs

### Slides (1920×1080)

- Body min **24px**, ideal 28–36px
- Title 60–120px
- Section title 80–160px
- Hero headline can use 180–240px huge type
- Never use <24px text in slides

### Print documents

- Body min **10pt** (~13.3px), ideal 11–12pt
- Title 18–36pt
- Caption 8–9pt

### Web and mobile

- Body min **14px** (use 16px for senior-friendly)
- Mobile body **16px** (avoid iOS auto-zoom)
- Hit target (clickable element) min **44×44px**
- Line height 1.5–1.7 (Chinese 1.7–1.8)

### Contrast

- Body vs background **at least 4.5:1** (WCAG AA)
- Large type vs background **at least 3:1**
- Use Chrome DevTools accessibility tooling to check

## CSS Power Tools

**Modern CSS features** are designers' best friends — use them boldly:

### Typography

```css
/* Make headings break naturally — no orphan word on the last line */
h1, h2, h3 { text-wrap: balance; }

/* Body wrap; avoid widows and orphans */
p { text-wrap: pretty; }

/* Chinese-typography killer: punctuation kerning, line-start/end control */
p { 
  text-spacing-trim: space-all;
  hanging-punctuation: first;
}
```

### Layout

```css
/* CSS Grid + named areas = max readability */
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
}

/* Subgrid aligns card contents */
.card { display: grid; grid-template-rows: subgrid; }
```

### Visual effects

```css
/* Designed scrollbar */
* { scrollbar-width: thin; scrollbar-color: #666 transparent; }

/* Glassmorphism (use sparingly) */
.glass {
  backdrop-filter: blur(20px) saturate(150%);
  background: color-mix(in oklch, white 70%, transparent);
}

/* View transitions API for smooth page changes */
@view-transition { navigation: auto; }
```

### Interaction

```css
/* :has() makes conditional styling easy */
.card:has(img) { padding-top: 0; } /* cards with an image have no top padding */

/* Container queries make components actually responsive */
@container (min-width: 500px) { ... }

/* New color-mix function */
.button:hover {
  background: color-mix(in oklch, var(--primary) 85%, black);
}
```

## Decision Cheat Sheet: When in Doubt

- Want to add a gradient? → probably don't
- Want to add an emoji? → don't
- Want to add rounded corners + border-left accent to a card? → don't; use a different approach
- Want to draw a hero illustration in SVG? → don't; use a placeholder
- Want to add a decorative quote? → ask the user if they have a real one first
- Want to add a row of icon-features? → ask whether icons are needed; probably not
- Use Inter? → swap for something more distinctive
- Use a purple gradient? → swap for a palette with reasoning behind it

**When you feel "adding this would look better," that's usually an AI-slop tell.** Build the simplest version first; add only when the user asks.
