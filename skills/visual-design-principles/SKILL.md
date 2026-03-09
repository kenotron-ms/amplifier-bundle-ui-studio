---
name: visual-design-principles
description: Use when generating or refining UI screen designs — applies graphic design fundamentals, visual hierarchy, typography, color, spacing, and screen archetype patterns to produce coherent, professional screens
version: 1.0.0
license: MIT
---

# Visual Design Principles for UI Screens

A working reference for applying graphic design thinking when generating storyboards and refining individual frames. The goal: screens that feel *designed*, not assembled.

---

## 1. Visual Hierarchy — The Eye Has a Job

Every screen needs one dominant element. The eye enters somewhere, then follows a path. You design that path deliberately.

**The hierarchy tools, in order of power:**
| Tool | Effect | Use For |
|------|--------|---------|
| **Size** | Biggest thing = most important | Hero titles, primary CTAs |
| **Weight** | Bold = emphasis without size change | Section headers, labels |
| **Color/Contrast** | High contrast = attention | Primary actions, alerts, prices |
| **Whitespace** | Space around = importance | Key messages, primary buttons |
| **Position** | Top-left entry, Z/F scan pattern | Primary content placement |
| **Texture/Depth** | Elevation separates layers | Cards above background |

**The rule of one:** Each screen should have exactly one element at the highest level of the hierarchy — the thing the user *should* look at first. If everything shouts, nothing is heard.

---

## 2. Typography — The Invisible Grid

Typography creates structure. When it's right, you don't notice it. When it's wrong, the screen feels chaotic.

### Scale
Use a modular scale (1.25× or 1.333× step). Approximate mobile scale:

| Role | Size | Weight | Line Height | Use For |
|------|------|--------|-------------|---------|
| Display | 40–48px | 700 | 1.1 | Hero moments, splash screens |
| H1 | 28–36px | 700 | 1.15 | Screen title, primary headline |
| H2 | 22–26px | 600 | 1.2 | Section headers |
| H3 | 17–20px | 600 | 1.3 | Card titles, list headers |
| Body | 14–16px | 400 | 1.5 | Content, descriptions |
| Label | 13–14px | 500 | 1.3 | UI labels, nav items, captions |
| Caption | 11–12px | 400 | 1.4 | Timestamps, metadata |

### Rules
- **Max 2 typefaces** — 1 is better. Pair a geometric sans (headings) with a humanist sans (body) if using 2.
- **Max 3 weights** — Regular (400), Medium/Semibold (500/600), Bold (700).
- **Never justify text** on mobile — ragged right reads better at narrow widths.
- **Line length** — 45–75 characters per line for body text. Short cards: 25–40 chars.
- **Color contrast** — body text needs 4.5:1 against background (WCAG AA); large headings need 3:1.

---

## 3. Color — The 60-30-10 System

Three roles, one rule: **60% background/surface, 30% secondary surfaces/content, 10% accent/action**.

### Semantic Color Roles
| Role | Token Name | Use |
|------|-----------|-----|
| **Background** | `color-background-screen` | Screen fill, behind everything |
| **Surface** | `color-background-card` | Cards, sheets, elevated containers |
| **Surface Elevated** | `color-background-elevated` | Modals, popovers |
| **Text Primary** | `color-text-primary` | Headings, body text |
| **Text Secondary** | `color-text-secondary` | Captions, metadata, inactive |
| **Text Tertiary** | `color-text-tertiary` | Placeholder, disabled |
| **Accent / Primary** | `color-accent-primary` | CTAs, links, active states, key highlights |
| **Accent Secondary** | `color-accent-secondary` | Secondary actions, tags |
| **Outline** | `color-border` | Dividers, input borders |
| **Error** | `color-error` | Validation, destructive actions |
| **Success** | `color-success` | Confirmations, completed states |

### Dark Theme Specifics
- Elevation uses **lighter surfaces** going up, not more saturated color
- Shadows don't work on dark — use surface color stepping instead
- Saturated colors on dark backgrounds vibrate; desaturate accents ~20%
- Surface colors: Background #0F0F1A → Card #1A1A2E → Elevated #242438 (example)

### Light Theme Specifics
- Avoid pure white (#FFFFFF) backgrounds — use off-white (#F7F7F7, #FAFAFA)
- Cards on light: subtle shadow + slight white offset, not heavy borders
- Primary text: #111 or #1A1A1A — pure black (#000) reads harsh

---

## 4. Spacing — The 8pt Grid

All spacing values are **multiples of 4** (preferred: multiples of 8).

```
4px   — micro gap (icon to label, tight related items)
8px   — small gap (between list item lines, internal card padding)
12px  — medium-small (related item groups)
16px  — base unit (standard padding, gaps between cards)
24px  — section gap (between unrelated groups)
32px  — large gap (between major sections)
48px  — section breathing room (top/bottom margins)
64px+ — hero spacing
```

**Mobile content margins:** 16–20px left/right. Never less than 16px.
**Card padding:** 16px standard, 12px compact.
**List item height:** 48–56px for tap targets (min 44px per Apple HIG).
**Bottom padding for scrollable content:** 32px + safe area inset.

---

## 5. Gestalt — How Elements Relate

The brain groups elements automatically. Design with these groupings, not against them.

| Principle | What It Means | Application |
|-----------|---------------|-------------|
| **Proximity** | Close = related | Group label + field + hint together; increase gap before next group |
| **Similarity** | Same style = same function | All primary buttons look identical; all secondary buttons look identical |
| **Continuity** | The eye follows lines and paths | Align elements to an invisible grid; connect related items with consistent left edge |
| **Closure** | The brain completes incomplete shapes | Cards don't need full borders — a shadow or background fill is enough |
| **Figure/Ground** | Content pops from background | Sufficient contrast between card and screen background |

---

## 6. Composition — How the Screen Breathes

**Content hierarchy on a mobile screen (top to bottom):**
1. **Anchor** — hero image, bold title, or key visual (first thing eyes land on)
2. **Orient** — context that explains what this screen is for
3. **Act** — primary action or primary content list
4. **Support** — secondary actions, metadata, navigation

**Avoid:**
- Equal visual weight everywhere — creates visual noise
- Cramming — leave breathing room between sections
- Centering everything — left-aligned text body reads faster
- Mixing alignment — pick left or center for a section, not both

---

## 7. Screen Archetypes

Common screen types and their design signatures. Every screen you design maps to one of these (or is a deliberate hybrid). Match the archetype to the design pattern.

### Feed / Home
**Purpose:** Ongoing content discovery  
**Visual signature:** Repeating card pattern with vertical scroll  
**Key design decisions:**
- Card aspect ratio (16:9 thumbnail or 1:1 avatar — pick one, stick to it)
- Information density (compact list vs. spacious cards)
- Section headers with "See all" links break the feed into categories
- One featured/hero item at the top (larger, different treatment)
- **Common mistake:** All cards the same size with no editorial hierarchy

### Detail / Article
**Purpose:** Deep dive on a single item  
**Visual signature:** Full-bleed hero media → title → metadata → body  
**Key design decisions:**
- Hero image fills the top 35–50% of the screen
- Title is the largest text element after the hero
- Metadata (date, author, read time) in small secondary text below title
- Body uses generous line height (1.6)
- Floating or sticky CTA (save, share, subscribe)
- **Common mistake:** Metadata same size as title, creating equal visual weight

### Profile
**Purpose:** User identity + their content  
**Visual signature:** Avatar + stats row → content grid/list below  
**Key design decisions:**
- Avatar is large (80–120px), centered or top-left, is the anchor
- Stats row (followers, following, posts) uses bold numbers + small labels
- Horizontal tabs switch between content types (Posts, Liked, Saved)
- Content below tabs is a grid (photos) or list (posts)
- **Common mistake:** Equal visual weight across avatar, name, bio, stats — avatar should dominate

### Settings / Preferences
**Purpose:** Configuration  
**Visual signature:** Grouped list rows with labels and controls  
**Key design decisions:**
- Grouped sections with section headers (12px uppercase, secondary color)
- 48–56px row height for tap targets
- Consistent right-alignment of controls (toggle, chevron, value)
- Destructive actions (Delete, Sign Out) isolated in their own group, red text
- **Common mistake:** All settings at the same visual weight — destructive actions need visual separation

### Onboarding / Splash
**Purpose:** First impression + value prop  
**Visual signature:** Full-bleed illustration/graphic + large title + description + CTA  
**Key design decisions:**
- Illustration or graphic takes 50–60% of screen height
- Title is the largest text on screen
- Description is 2–3 lines max (short and punchy)
- Single primary CTA (large, full-width, accent color)
- Progress dots if multi-step
- **Common mistake:** Too much text, small illustration, weak CTA styling

### Form / Input
**Purpose:** Data collection  
**Visual signature:** Labeled fields stacked vertically, submit at bottom  
**Key design decisions:**
- Labels above fields (not inside as placeholder — placeholder disappears on input)
- Group related fields visually (billing info, shipping info in separate sections)
- Error states: red border + red helper text below the field (not a modal)
- Primary submit button: full-width, accent, at the bottom
- **Common mistake:** Placeholder text used as labels — disappears when user types

### Search / Results
**Purpose:** Information retrieval  
**Visual signature:** Sticky search bar → filters → results list  
**Key design decisions:**
- Search input is the most prominent element, at the very top
- Filter chips below the search bar (horizontal scroll)
- Results use consistent card/row pattern
- Empty state with helpful prompts (recent searches, suggestions)
- **Common mistake:** Filters buried below fold, search results inconsistent with browse screens

### Empty State
**Purpose:** No-content moment — doesn't mean no design  
**Visual signature:** Centered illustration → message → action  
**Key design decisions:**
- Illustration is friendly and contextual to the content type
- Title is 1 short line (e.g., "Nothing saved yet")
- Subtitle is 1–2 lines explaining what to do
- Primary CTA button gets the user unstuck
- **Common mistake:** Just showing "No results" text — missed moment to guide the user

### Dashboard / Stats
**Purpose:** Data at a glance  
**Visual signature:** Key metric cards at top → supporting charts → detail lists  
**Key design decisions:**
- Most important metric: largest, boldest number on screen
- Stat cards: number large (H1 weight), label small, trend indicator (↑↓)
- Charts are supplementary — label data points directly (avoid legends)
- **Common mistake:** Equal-weight stats with no primary number

### Modal / Bottom Sheet
**Purpose:** Focused action without leaving context  
**Visual signature:** Handle indicator → title → content → action buttons  
**Key design decisions:**
- Handle (drag indicator) at top for bottom sheets
- Title is clear and action-oriented ("Delete Post?" not "Confirm")
- Content is minimal — don't put a full screen inside a modal
- Primary action prominent (right side or bottom full-width)
- Destructive actions: red text
- **Common mistake:** Too much content forcing scroll inside a modal

---

## 8. Cross-Screen Coherence

What makes a multi-screen app feel like *one thing* rather than a collection of screens:

| Consistency Signal | What It Means |
|-------------------|---------------|
| **Navigation chrome** | Tab bar / nav bar visually identical on every route screen (locked in nav-shell.md) |
| **Card pattern** | Cards for the same content type look identical across screens |
| **Button style** | Primary, secondary, and destructive buttons are identical everywhere |
| **Typography application** | H1 means the same thing visually on every screen |
| **Color roles** | Accent color is used *only* for interactive/primary elements |
| **Spacing rhythm** | The same vertical rhythm between sections on every screen |
| **Icon style** | One icon library, one size rule, one stroke weight |
| **Photo treatment** | All thumbnails same aspect ratio, same corner radius, same object-fit |

**Design coherence test:** Take any two screens side by side. Would a user know they're from the same app? They should. Buttons, type, color, and spacing should feel like they share a system.

---

## 9. Common Anti-Patterns

Things that make screens feel undesigned:

| Anti-Pattern | Fix |
|-------------|-----|
| **Everything is the same size** | Establish hierarchy — something must be dominant |
| **Random spacing** | Commit to the 8pt grid, be consistent |
| **Too many accent colors** | One accent color per screen. More = noise |
| **Centered body text** | Left-align content text. Center only for 1–2 line UI moments |
| **Pure white/black** | Off-white (#FAFAFA) and near-black (#111) |
| **Crowded navigation** | Max 5 tab bar items; max 3 action buttons in headers |
| **Inconsistent corner radius** | Pick one radius for cards, one for buttons, don't mix |
| **Icons without labels** | Below 7 icons, labels help comprehension; always label nav |
| **Placeholder as label** | Labels above fields, always. Placeholders are hints only |
| **Equal-weight lists** | Use typographic and color variation to establish row hierarchy |
| **Missing empty states** | Every list/feed needs a designed empty state |
| **No loading skeleton** | Add skeleton screens for async content |

---

## 10. Translating to Nano-Banana Prompts

When generating screens with nano-banana, embed these principles explicitly in the prompt. Don't assume the model will apply design thinking automatically.

**Hierarchy anchor:**
```
The visual anchor for this screen is [element]. It should be the largest/most prominent element. 
Everything else recedes relative to it.
```

**Typography precision:**
```
Typography hierarchy:
- Primary heading: [size]px, [weight], color-text-primary
- Secondary text: [size]px, [weight], color-text-secondary  
- Body text: [size]px, regular weight, line-height 1.5
- Captions/metadata: 12px, secondary color
```

**Color discipline:**
```
Color usage:
- Backgrounds and surfaces only: color-background-screen, color-background-card
- One accent: [accent color hex] — used ONLY for primary actions and active states
- Text: [primary hex] for headings, [secondary hex] for body, [tertiary hex] for captions
- Do NOT introduce additional colors beyond this palette
```

**Spacing rhythm:**
```
Spacing system (8pt grid):
- Screen margins: 20px left/right
- Between cards: 16px
- Between sections: 32px
- Card internal padding: 16px
```

**Archetype reference:**
```
This screen follows the [FEED / DETAIL / PROFILE / SETTINGS / FORM / etc.] archetype:
[paste the relevant archetype pattern above]
```

**Coherence constraint (for multi-screen storyboards):**
```
All screens in this storyboard share:
- Same card treatment (corner radius, shadow)
- Same button style
- Same typography scale
- Same color palette
Do NOT introduce visual variations between screens.
```
