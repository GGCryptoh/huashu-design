# Design Critique Deep Guide

> Detailed reference for Phase 7. Provides scoring criteria, scenario priorities, and a common-issues checklist.

---

## Scoring Criteria in Detail

### 1. Philosophy Alignment

| Score | Criteria |
|------|------|
| 9-10 | Design perfectly embodies the core spirit of the chosen philosophy; every detail has a philosophical basis |
| 7-8 | Overall direction is correct, core characteristics are in place, individual details drift |
| 5-6 | Intent is visible, but execution mixes in elements from other styles — not pure enough |
| 3-4 | Imitates only the surface, doesn't grasp the philosophical core |
| 1-2 | Essentially unrelated to the chosen philosophy |

**Review checklist**:
- Are the signature techniques of that designer/institution being used?
- Do colors, typography, and layout align with that philosophy's system?
- Are there any "self-contradictory" elements? (e.g., picked Kenya Hara but stuffed it with content)

### 2. Visual Hierarchy

| Score | Criteria |
|------|------|
| 9-10 | The user's gaze flows naturally along the designer's intent; zero friction in absorbing information |
| 7-8 | Primary/secondary relationships are clear, with 1–2 spots of fuzzy hierarchy |
| 5-6 | You can distinguish title from body, but the middle layers are muddled |
| 3-4 | Information is laid out flat, no clear visual entry point |
| 1-2 | Chaotic — the user doesn't know where to look first |

**Review checklist**:
- Is the size contrast between heading and body sufficient? (At least 2.5×)
- Do color / weight / size establish 3–4 clear levels?
- Is whitespace guiding the eye?
- "Squint test": squint your eyes — is the hierarchy still clear?

### 3. Craft Quality

| Score | Criteria |
|------|------|
| 9-10 | Pixel-perfect; no flaws in alignment, spacing, or color |
| 7-8 | Polished overall, with 1–2 tiny alignment / spacing issues |
| 5-6 | Mostly aligned, but spacing isn't unified and color use isn't systematic |
| 3-4 | Obvious alignment errors, messy spacing, too many colors |
| 1-2 | Crude — looks like a draft |

**Review checklist**:
- Is a unified spacing system used (e.g., 8pt grid)?
- Is spacing consistent across same-type elements?
- Is the number of colors controlled? (Usually no more than 3–4)
- Is the font family unified? (Usually no more than 2)
- Are edges aligned precisely?

### 4. Functionality

| Score | Criteria |
|------|------|
| 9-10 | Every design element serves the goal; zero redundancy |
| 7-8 | Function-oriented, with a small amount of decoration that could be cut |
| 5-6 | Usable, but obvious decorative elements distract |
| 3-4 | Form over function; the user has to work to find information |
| 1-2 | Completely drowned by decoration; loses the ability to communicate |

**Review checklist**:
- If you delete any one element, does the design get worse? (If not, delete it.)
- Is the CTA / key info in the most prominent position?
- Are there elements added "because they look nice"?
- Does the information density match the medium? (PPT shouldn't be too dense; PDF can be denser)

### 5. Originality

| Score | Criteria |
|------|------|
| 9-10 | Refreshing — finds a unique expression within the philosophy's framework |
| 7-8 | Has its own ideas, not a simple template application |
| 5-6 | Conventional, looks like a template |
| 3-4 | Heavy use of clichés (e.g., gradient orbs as a stand-in for AI) |
| 1-2 | Pure template or stock-asset assembly |

**Review checklist**:
- Have common clichés been avoided? (See "Common Issues Checklist" below.)
- Is there personal expression while still respecting the design philosophy?
- Are there any "unexpected but reasonable" design decisions?

---

## Scenario-specific Critique Priorities

Different output types have different review priorities:

| Scenario | Most important | Secondary | Can relax |
|------|-----------|--------|--------|
| Public-account cover / image | Originality, visual hierarchy | Philosophy alignment | Functionality (single image — no interaction) |
| Infographic | Functionality, visual hierarchy | Craft quality | Originality (accuracy first) |
| PPT / Keynote | Visual hierarchy, functionality | Craft quality | Originality (clarity first) |
| PDF / whitepaper | Craft quality, functionality | Visual hierarchy | Originality (professionalism first) |
| Landing page / marketing site | Functionality, visual hierarchy | Originality | — (full requirements) |
| App UI | Functionality, craft quality | Visual hierarchy | Philosophy alignment (usability first) |
| Xiaohongshu image | Originality, visual hierarchy | Philosophy alignment | Craft quality (mood first) |

---

## Top 10 Common Design Issues

### 1. AI tech cliché
**Issue**: gradient orbs, digital rain, blue circuit boards, robot faces
**Why it's an issue**: Users are visually fatigued by these — you can't differentiate yourself
**Fix**: Use abstract metaphors instead of literal symbols (e.g., a "conversation" metaphor instead of a chat-bubble icon)

### 2. Insufficient size hierarchy
**Issue**: Title and body are too close (<2.5×)
**Why it's an issue**: Users can't quickly locate key information
**Fix**: Title should be at least 3× the body (e.g., body 16px → title 48–64px)

### 3. Too many colors
**Issue**: Five or more colors used with no primary/secondary structure
**Why it's an issue**: Visually chaotic; weak brand feel
**Fix**: Limit to 1 primary + 1 secondary + 1 accent + grayscale

### 4. Inconsistent spacing
**Issue**: Element spacing is arbitrary; no system
**Why it's an issue**: Looks unprofessional; visual rhythm is broken
**Fix**: Build an 8pt grid (only use 8/16/24/32/48/64px spacing)

### 5. Insufficient whitespace
**Issue**: Every space is filled with content
**Why it's an issue**: Crowded info causes reading fatigue, lowering communication efficiency
**Fix**: Whitespace should occupy at least 40% of the total area (60%+ for minimal styles)

### 6. Too many fonts
**Issue**: Three or more fonts used
**Why it's an issue**: Visual noise; weakens unity
**Fix**: Two fonts max (1 heading + 1 body); use weight and size for variation

### 7. Inconsistent alignment
**Issue**: Some elements left-aligned, some center, some right
**Why it's an issue**: Breaks visual order
**Fix**: Pick one alignment (left is recommended) and apply globally

### 8. Decoration over content
**Issue**: Background patterns / gradients / shadows steal attention from the main content
**Why it's an issue**: Backwards — users come for the information, not the decoration
**Fix**: Ask "Would the design get worse if I deleted this decoration?" If no, delete it.

### 9. Cyber-neon overuse
**Issue**: Deep blue background (#0D1117) + neon glow effects
**Why it's an issue**: A default aesthetic no-go zone (this skill's taste baseline) and one of the biggest clichés — the user can override per their brand
**Fix**: Choose a more distinctive color scheme (refer to the color systems in the 20 styles)

### 10. Information density doesn't match the medium
**Issue**: A whole page of text in a PPT / 10 elements stuffed into a cover image
**Why it's an issue**: Different media have different optimal information densities
**Fix**:
- PPT: 1 core point per page
- Cover image: 1 visual focus
- Infographic: layered presentation
- PDF: can be denser, but needs clear navigation

---

## Critique Output Template

```
## Design Critique Report

**Overall score**: X.X/10 [Excellent (8+) / Good (6–7.9) / Needs work (4–5.9) / Fails (<4)]

**Sub-scores**:
- Philosophy alignment: X/10 [one-line note]
- Visual hierarchy: X/10 [one-line note]
- Craft quality: X/10 [one-line note]
- Functionality: X/10 [one-line note]
- Originality: X/10 [one-line note]

### Strengths (Keep)
- [Specifically call out what's done well, in design language]

### Issues (Fix)
[Sorted by severity]

**1. [Issue name]** — Critical / Important / Polish
- Current: [describe state]
- Issue: [why it's a problem]
- Fix: [concrete action with values]

### Quick Wins
If you only have 5 minutes, do these 3 first:
- [ ] [Highest-impact fix]
- [ ] [Second most important fix]
- [ ] [Third most important fix]
```

---

**Version**: v1.0
**Last updated**: 2026-02-13
