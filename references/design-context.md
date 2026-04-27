# Design Context: Start From What Already Exists

**This is the single most important thing in this skill.**

Good hi-fi design always grows out of an existing design context. **Hi-fi from scratch is a last resort and will produce generic work.** So at the start of every design task, ask: is there anything to reference?

## What Is Design Context

In priority order, high to low:

### 1. The user's Design System / UI Kit
The component library, color tokens, typography spec, and icon system that already exist in the user's product. **The ideal scenario.**

### 2. The user's codebase
If the user gave you a codebase, the components live there in working form. Read those files:
- `theme.ts` / `colors.ts` / `tokens.css` / `_variables.scss`
- Specific components (Button.tsx, Card.tsx)
- Layout scaffold (App.tsx, MainLayout.tsx)
- Global stylesheets

**Read the code and copy exact values**: hex codes, spacing scale, font stack, border radius. Don't redraw from memory.

### 3. The user's shipped product
If the user has a live product but no codebase, use Playwright or have them provide screenshots.

```bash
# Use Playwright to screenshot a public URL
npx playwright screenshot https://example.com screenshot.png --viewport-size=1920,1080
```

So you can see the actual visual vocabulary.

### 4. Brand guidelines / logo / existing assets
The user might have: logo files, brand color spec, marketing collateral, slide templates. All of it is context.

### 5. Competitor references
The user says "like XX site does it" — ask for the URL or a screenshot. **Don't** work from a vague memory baked into your training data.

### 6. Known design systems (fallback)
If none of the above exists, use a recognized design system as a base:
- Apple HIG
- Material Design 3
- Radix Colors (palette)
- shadcn/ui (components)
- Tailwind's default palette

Tell the user explicitly which one you're using so they know it's a starting point, not a final spec.

## Workflow for Acquiring Context

### Step 1: Ask the user

The mandatory questions at the start of a task (from `workflow.md`):

```markdown
1. Do you have an existing design system / UI kit / component library? Where?
2. Brand guidelines, color / typography specs?
3. Can you give me screenshots or a URL of the existing product?
4. Is there a codebase I can read?
```

### Step 2: When the user says "no", help them find it

Don't give up immediately. Try:

```markdown
Let me see if there's any thread to pull on:
- Any related designs from your previous projects?
- What colors / typography does your company's marketing site use?
- What style is your product's logo? Can you send one?
- Any product you admire that we should reference?
```

### Step 3: Read every piece of context you can find

If the user gives you a codebase path, you read:
1. **List the file structure first**: find files related to style/theme/components
2. **Read theme/token files**: lift the actual hex/px values
3. **Read 2–3 representative components**: see the visual vocabulary (hover state, shadow, border, padding patterns)
4. **Read the global stylesheet**: base resets, font loading
5. **If there's a Figma link / screenshots**: look at them, but **trust the code more**

**Important**: **Don't** glance once and rebuild from impression. You haven't really lifted the system until you have 30+ concrete values.

### Step 4: Vocalize the system you're going to use

After reading the context, tell the user the system you're using:

```markdown
Based on your codebase and product screenshots, the design system I extracted:

**Color**
- Primary: #C27558 (from tokens.css)
- Background: #FDF9F0
- Text: #1A1A1A
- Muted: #6B6B6B

**Typography**
- Display: Instrument Serif (from @font-face in global.css)
- Body: Geist Sans
- Mono: JetBrains Mono

**Spacing** (from your scale)
- 4, 8, 12, 16, 24, 32, 48, 64

**Shadow patterns**
- `0 1px 2px rgba(0,0,0,0.04)` (subtle card)
- `0 10px 40px rgba(0,0,0,0.1)` (elevated modal)

**Border-radius**
- Small components 4px, cards 12px, buttons 8px

**Component vocabulary**
- Button: filled primary, outlined secondary, ghost tertiary, all 8px corners
- Card: white background, subtle shadow, no border

I'm starting with this system. Confirm OK?
```

Wait for confirmation before starting.

## Designing From Scratch (the no-context fallback)

**Strong warning**: output quality drops noticeably in this case. Tell the user explicitly.

```markdown
You don't have a design context, so I can only work from generic intuition.
The output will be "looks OK but lacks distinctiveness".
Want to keep going, or supply some references first?
```

If the user insists, make decisions in this order:

### 1. Pick an aesthetic direction
Don't ship a generic result. Pick a clear direction:
- brutally minimal
- editorial / magazine
- brutalist / raw
- organic / natural
- luxury / refined
- playful / toy
- retro-futuristic
- soft / pastel

Tell the user which one you picked.

### 2. Pick a known design system as the skeleton
- Use Radix Colors for the palette (https://www.radix-ui.com/colors)
- Use shadcn/ui for component vocabulary (https://ui.shadcn.com)
- Use Tailwind's spacing scale (multiples of 4)

### 3. Pick distinctive font pairings

Don't use Inter / Roboto. Recommended pairings (free off Google Fonts):
- Instrument Serif + Geist Sans
- Cormorant Garamond + Inter Tight
- Bricolage Grotesque + Söhne (paid)
- Fraunces + Work Sans (note: Fraunces is already AI-overused)
- JetBrains Mono + Geist Sans (technical feel)

### 4. Every key decision has reasoning

Don't decide silently. Write it in the HTML comment:

```html
<!--
Design decisions:
- Primary color: warm terracotta (oklch 0.65 0.18 25) — fits the "editorial" direction
- Display: Instrument Serif for humanist, literary feel
- Body: Geist Sans for cleanness contrast
- No gradients — committed to minimal, no AI slop
- Spacing: 8px base, golden-ratio friendly (8/13/21/34)
-->
```

## Import Strategy (when the user gives you a codebase)

If the user says "import this codebase as reference":

### Small (<50 files)
Read everything; internalize the context.

### Medium (50–500 files)
Focus on:
- `src/components/` or `components/`
- All style/token/theme-related files
- 2–3 representative full-page components (Home.tsx, Dashboard.tsx)

### Large (>500 files)
Have the user point you to the focus area:
- "I want to do the settings page" → read existing settings stuff
- "I want a new feature" → read overall shell + nearest reference
- Don't aim for completeness; aim for accuracy

## Working with Figma / design mocks

If the user gives you a Figma link:

- **Don't** expect to "convert Figma to HTML" directly — that needs extra tooling
- Figma links are usually not publicly accessible
- Have the user: export as **screenshots** + tell you the specific color / spacing values

If they only give a Figma screenshot, tell them:
- I can see the visuals, but I can't extract precise values
- Please give me the key numbers (hex, px), or export as code (Figma supports this)

## Final Reminder

**The ceiling on a project's design quality is determined by the quality of context you start with.**

Spending 10 minutes collecting context is worth more than an hour of hi-fi from scratch.

**When there's no context, prioritize asking the user, not pushing through.**
