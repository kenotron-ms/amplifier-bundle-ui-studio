---
name: blueprint-extraction
description: Use when extracting a component spec from an approved screen PNG — applies the containment model to ensure every pixel belongs to a component before the spec is considered complete
version: 1.0.0
license: MIT
---

# Blueprint Extraction: No Pixel Without a Container

Extract a complete component spec, design tokens, and asset inventory from an approved screen PNG. The spec is not done until every pixel belongs to a named component.

## Overview

Blueprint extraction turns a visual design into a structured specification using the **containment model**: every pixel in the approved screen must be accounted for inside a named component. The agent self-judges completeness — no human input is needed to determine when the spec is done.

**Input:** Approved screen PNG
**Output:** Component spec, token file, asset inventory
**Judge:** Agent (self) — "Are there uncovered pixels?"

---

## The Containment Model Principle

> Every pixel must belong to a named component. The spec is NOT complete until coverage is 100%.

The containment model is the core idea behind blueprint extraction. It works like this:

- The screen is a rectangle of pixels
- Every pixel must be assigned to exactly one component in a hierarchy
- A "component" is a named, bounded region: a card, a heading, an icon, a background
- Parent components contain child components — the hierarchy is a tree
- The root component is the screen itself

**Why this matters:** During code generation (`/forge`), every component in the spec becomes a real element in code. If a visual element was never captured in the spec, it will be missing from the generated code. The containment model prevents this class of error entirely — nothing is missed because nothing is allowed to be uncovered.

**What "covered" means:**
- The pixel falls inside the bounding box of a named component
- That component has a type, a position in the hierarchy, and associated tokens
- Decorative space (margins, padding, backgrounds) is covered by the parent container

**What "uncovered" means:**
- A region of pixels exists that no component claims
- This is a gap — the agent must create a new component or expand an existing one to cover it

---

## How to Generate Containment Overlays with Nano Banana

The containment overlay is a visual annotation layer drawn on top of the approved screen PNG. It shows labeled bounding boxes for every component the agent has identified.

### Step 1: Generate the Initial Overlay

```bash
amplifier tool invoke nano-banana \
  operation=generate \
  reference_image_path=approved-screen.png \
  output_path=containment-overlay.png \
  'prompt=TAKE THIS IMAGE and draw a CONTAINMENT OVERLAY on top of it.

For EVERY visual element in the screen, draw:
- A labeled bounding box showing the component boundary
- A text label with the component name (e.g., "HeaderBar", "ArticleCard", "HeroBackground")
- Hierarchy indicators: nest child boxes inside parent boxes
- Use distinct colors for different hierarchy levels:
  - Level 0 (root/screen): no box needed
  - Level 1 (major sections): RED borders
  - Level 2 (cards, groups): BLUE borders
  - Level 3 (individual elements): GREEN borders

Every pixel must fall inside at least one labeled box.
Leave NO region unlabeled. If an area is a background, label it (e.g., "ScreenBackground").
If an area is whitespace/padding, it belongs to its parent container.

Use semi-transparent fills so the original design is visible beneath.
Make labels bold with drop shadows for readability.' \
  aspect_ratio=preserve \
  resolution=2K \
  use_thinking=true \
  number_of_images=1
```

**Critical parameter:** `reference_image_path` tells nano-banana to annotate the existing image — not generate a new one from scratch.

**Output:** `containment-overlay.png` — the approved screen with labeled bounding boxes for every component.

### What the Overlay Shows

- **Component boundaries:** Colored rectangles around every element
- **Component names:** Text labels identifying each element (e.g., "NavBar", "SearchInput", "ArticleThumbnail")
- **Hierarchy:** Nested boxes showing parent-child relationships
- **Full coverage:** Every region of the screen is inside at least one box

---

## Self-Judgment Loop: How the Agent Decides When Coverage Is Complete

The agent runs an autonomous loop. No human input is required.

### The Loop

```
┌─────────────────────────────────────────────────────────────┐
│  1. Generate containment overlay             │
│  2. Analyze: "Any pixels NOT in a box?"      │
│  3. If gaps → define new components           │
│  4. Regenerate overlay with additions         │
│  5. Repeat from step 2                        │
│  6. Stop when: no uncovered pixels remain     │
└─────────────────────────────────────────────────────────────┘
```

### Step-by-Step

**Step 1 — Generate overlay:**
Create the initial containment overlay using nano-banana (see above).

**Step 2 — Check for gaps:**
Analyze the overlay image. Ask: *"Are there any pixels in this screen that are NOT covered by a labeled bounding box?"*

```bash
amplifier tool invoke nano-banana \
  operation=analyze \
  image_path=containment-overlay.png \
  'prompt=Analyze this containment overlay carefully.

The overlay shows labeled bounding boxes drawn on top of a screen design.
Your job: find any region of the screen that is NOT inside a labeled box.

Look for:
- Gaps between boxes where the original design is visible but no box covers it
- Small elements (icons, dividers, badges) that were missed
- Background regions that have no container assigned
- Edges or corners of the screen that fall outside all boxes

Output JSON:
{
  "coverage_complete": true/false,
  "uncovered_regions": [
    {
      "description": "What is visible in the uncovered area",
      "location": "Approximate position (e.g., top-right corner, between CardA and CardB)",
      "suggested_component": "Proposed name and type for a component to cover it"
    }
  ],
  "total_components_identified": N,
  "confidence": "high/medium/low"
}'
```

**Step 3 — Fill gaps:**
For each uncovered region, define a new component:
- Give it a name that describes its role (e.g., `StatusBarBackground`, `DividerLine`, `FloatingActionButton`)
- Assign it a type (container, text, image, icon, etc.)
- Place it in the hierarchy under the correct parent
- Add it to the component spec

**Step 4 — Regenerate overlay:**
Generate a new overlay that includes the additional components. Pass the updated component list in the prompt so nano-banana draws boxes for the new components too.

**Step 5 — Repeat:**
Go back to Step 2. Check coverage again. Continue until the analysis returns `"coverage_complete": true`.

**Step 6 — Done:**
When no uncovered pixels remain, the containment model is satisfied. Proceed to token extraction and asset identification.

### Convergence

Typically takes 1–3 iterations:
- Iteration 1: Captures 80–90% of components (major sections, obvious elements)
- Iteration 2: Catches small elements (dividers, badges, status bar items)
- Iteration 3: Edge cases (background regions, decorative spacing)

---

## Design Token Extraction

Once the containment model is complete, extract design tokens from the approved screen. Tokens are the atomic visual values used by components.

### What to Extract

**Colors:**
- Background colors for each container
- Text colors for each text element
- Border colors where visible
- Accent/highlight colors (buttons, links, active states)
- Format: hex codes (e.g., `#1A1A2E`)

**Typography:**
- Font families (describe visually: serif/sans-serif, geometric/humanist, stroke contrast — do not guess names)
- Font sizes in px for each distinct text style
- Font weights (regular, medium, semibold, bold)
- Line heights where distinguishable
- Group into named styles: heading, subheading, body, caption, label

**Spacing:**
- Padding inside containers (top, right, bottom, left) in px
- Margins between sibling components in px
- Gap values in flex/grid layouts in px
- Measure from the approved screen at actual resolution

**Borders:**
- Border radius per component (e.g., 8px, 12px, fully round)
- Border width where visible (e.g., 1px, 2px)
- Border color (reference color tokens)

### Token Extraction: 4-Phase Process

Token extraction is a four-phase process: analyze → write tokens.json → generate spec overlay → verify.

#### Phase 1: Raw Token Analysis

```bash
amplifier tool invoke nano-banana \
  operation=analyze \
  image_path=approved-screen.png \
  'prompt=Extract all design tokens from this screen. For each, provide exact values:

COLORS: Every distinct color as hex. Name by role:
  color-primary, color-secondary, color-surface, color-background,
  color-text-primary, color-text-secondary, color-accent, color-border, etc.

TYPOGRAPHY: Every distinct text style. Describe font-family visually
  (serif/sans-serif, geometric/humanist, stroke contrast — do not guess names).
  Include font-size (px), font-weight (400/500/600/700), line-height.
  Name each style: heading-lg, heading-md, body, caption, label, etc.

SPACING: Measure padding, margin, and gap values in px.
  Name each: spacing-xs, spacing-sm, spacing-md, spacing-lg, spacing-xl

BORDERS: Border-radius, border-width, and border-color per component.
  Name each: radius-sm, radius-md, radius-lg, radius-full

SHADOWS: Any drop shadows or elevation — offset, blur, spread, color.

Output as structured JSON with all five categories.'
```

#### Phase 2: Write tokens.json

Save the extracted values as `tokens.json` (see [Token File Format](#token-file-format) above).

#### Phase 3: Generate Token Spec Overlay

Generate a professional redline/handoff sheet — the original screen centered, surrounded by callout annotations for every token. Populate the prompt with the **actual values from Phase 1**:

```bash
amplifier tool invoke nano-banana \
  operation=generate \
  reference_image_path=approved-screen.png \
  output_path=blueprint/token-overlay.png \
  'prompt=Create a professional design specification redline sheet.

Reproduce the original UI faithfully in the CENTER of the image (preserve exact
proportions, colors, typography, and layout). Surround it with token annotations.

ANNOTATION STYLE:
- Label boxes: dark navy (#0F172A bg, #F8FAFC text, 1px #334155 border, 6px radius)
- Callout lines: thin amber (#F59E0B) connecting labels to UI elements
- Color swatches: 14x14px filled squares inline with each color label
- Spacing arrows: thin double-headed arrows (#38BDF8) with px measurements

[LEFT SIDE — Spacing & Layout]
Double-headed arrows annotating: {spacing values from Phase 1}

[RIGHT SIDE — Typography & Colors]
TYPOGRAPHY group: {font-family descriptions, sizes, weights from Phase 1}
COLOR group: {hex values with filled swatches from Phase 1}
BORDER RADIUS group: {radius values from Phase 1}
SHADOWS (below UI if present): {shadow specs from Phase 1}

OUTPUT: Wide format 1600x1000px, white background, UI centered,
professional Figma-style handoff sheet aesthetic.' \
  resolution=2K \
  use_thinking=true \
  number_of_images=1
```

#### Phase 4: Verify the Overlay

```bash
amplifier tool invoke nano-banana \
  operation=analyze \
  image_path=blueprint/token-overlay.png \
  'prompt=Evaluate this design specification sheet:
1. Is the UI faithfully reproduced in the center?
2. Are annotations legible (label text, callout lines, color swatches)?
3. Any obvious inaccuracies (wrong colors, empty swatches, misidentified fonts)?
Report any issues found.'
```

If the verify step flags inaccuracies, note them in `tokens.json` as comments and proceed — `tokens.json` is the authoritative record; the overlay is visual evidence.

### Token File Format

```json
{
  "color-primary": "#1A1A2E",
  "color-secondary": "#16213E",
  "color-surface": "#FFFFFF",
  "color-surface-elevated": "#F5F5F5",
  "color-background": "#0F0F1A",
  "color-text-primary": "#FFFFFF",
  "color-text-secondary": "#A0A0B0",
  "color-accent": "#E94560",
  "color-border": "#2A2A3E",
  "font-family-primary": "Inter",
  "font-family-secondary": "Georgia",
  "font-size-heading-lg": "28px",
  "font-size-heading-md": "20px",
  "font-size-body": "16px",
  "font-size-caption": "12px",
  "font-weight-regular": "400",
  "font-weight-medium": "500",
  "font-weight-bold": "700",
  "line-height-tight": "1.2",
  "line-height-normal": "1.5",
  "spacing-xs": "4px",
  "spacing-sm": "8px",
  "spacing-md": "16px",
  "spacing-lg": "24px",
  "spacing-xl": "32px",
  "radius-sm": "4px",
  "radius-md": "8px",
  "radius-lg": "16px",
  "radius-full": "9999px",
  "border-width-default": "1px"
}
```

Save as `tokens.json` alongside the component spec.

---

## Asset Identification and Generation

Every non-text, non-solid-color visual element in the screen is an asset. Identify all of them.

### Asset Categories

**Backgrounds:**
- Full-bleed images behind content (hero images, splash backgrounds)
- Gradient backgrounds (CSS-reproducible — note the gradient stops)
- Textured or patterned backgrounds (need generation or placeholder)

**Thumbnails:**
- Content images within cards or lists (article photos, product images)
- Avatar images (circular or rounded profile photos)
- Preview images (video thumbnails, gallery items)

**Illustrations:**
- Decorative graphics (onboarding illustrations, empty states)
- Diagrams or infographics
- Brand graphics or logos

**Icons:**
- Individual UI icons (navigation, actions, status indicators)
- Use the `icon-finding` skill to select from Lucide, Heroicons, or Feather
- Note: prefer existing icon libraries over generating custom icons

### For Each Asset, Record

| Field | Description |
|-------|-------------|
| **name** | Descriptive identifier (e.g., `hero-background`, `author-avatar`) |
| **type** | `background` / `thumbnail` / `illustration` / `icon` |
| **dimensions** | Width x height in px as measured from the design |
| **description** | What the asset depicts (e.g., "dark abstract texture", "person photo") |
| **role** | Where it appears in the component hierarchy |
| **source** | `generate` (nano-banana), `placeholder` (solid color/gradient), `icon-library` (Lucide/etc.) |

### Asset Identification Prompt

```bash
amplifier tool invoke nano-banana \
  operation=analyze \
  image_path=approved-screen.png \
  'prompt=Identify every visual asset in this screen design.

An "asset" is any visual element that is NOT:
- Solid color fills (those are tokens)
- Text content (that is component content)
- UI chrome like borders or shadows (those are tokens)

For each asset found, provide:
- name: descriptive identifier (kebab-case)
- type: background | thumbnail | illustration | icon
- dimensions: width x height in px
- description: what it depicts visually
- role: what it does in the design (hero image, avatar, nav icon, etc.)
- source: how to obtain it (generate, placeholder, icon-library)

Output as JSON array.'
```

### Asset Inventory Format

```
hero-background:     background,   390x844px,  dark abstract texture with subtle gradient
author-avatar:       thumbnail,    40x40px,    circular person photo placeholder
article-thumbnail:   thumbnail,    120x80px,   content photo for article card
search-icon:         icon,         24x24px,    magnifying glass — Lucide: Search
bookmark-icon:       icon,         24x24px,    bookmark outline — Lucide: Bookmark
back-arrow:          icon,         24x24px,    left-pointing arrow — Lucide: ArrowLeft
onboarding-graphic:  illustration, 300x200px,  welcome illustration with abstract shapes
```

Save as `assets.md` or `assets.json` alongside the component spec.

---

## Output Formats

Blueprint extraction produces three files. All three must be complete before the blueprint is considered done.

### 1. Component Spec (`component-spec.md`)

A hierarchical list of every component, with type, children, token references, content, and asset references.

```
Screen
  - type: container
  - children: [StatusBar, HeaderBar, HeroSection, ContentArea, BottomNav]
  - tokens: { background: color-background }

  StatusBar
    - type: container
    - children: [TimeDisplay, SignalIcons, BatteryIcon]
    - tokens: { background: transparent, padding: spacing-sm }

    TimeDisplay
      - type: text
      - content: "9:41"
      - tokens: { color: color-text-primary, font-size: font-size-caption, font-weight: font-weight-medium }

  HeaderBar
    - type: container
    - children: [BackButton, Title, SearchButton]
    - tokens: { padding: spacing-md, gap: spacing-sm }

    BackButton
      - type: icon
      - asset: back-arrow
      - tokens: { color: color-text-primary }

    Title
      - type: text
      - content: "Discover"
      - tokens: { color: color-text-primary, font-size: font-size-heading-lg, font-weight: font-weight-bold }

    SearchButton
      - type: icon
      - asset: search-icon
      - tokens: { color: color-text-primary }

  HeroSection
    - type: container
    - children: [HeroImage, HeroOverlay, HeroTitle, HeroSubtitle]
    - tokens: { border-radius: radius-lg, overflow: hidden }

    HeroImage
      - type: image
      - asset: hero-background
      - tokens: { width: 100%, height: 200px }

    HeroTitle
      - type: text
      - content: [dynamic]
      - tokens: { color: color-text-primary, font-size: font-size-heading-md, font-weight: font-weight-bold }

  ContentArea
    - type: container
    - children: [ArticleCard, ArticleCard, ArticleCard]
    - tokens: { padding: spacing-md, gap: spacing-md }

    ArticleCard
      - type: container
      - children: [ArticleThumbnail, ArticleInfo]
      - tokens: { background: color-surface, border-radius: radius-md, padding: spacing-sm }

      ArticleThumbnail
        - type: image
        - asset: article-thumbnail
        - tokens: { border-radius: radius-sm }

      ArticleInfo
        - type: container
        - children: [ArticleTitle, ArticleAuthor]
        - tokens: { gap: spacing-xs }

        ArticleTitle
          - type: text
          - content: [dynamic]
          - tokens: { color: color-text-primary, font-size: font-size-body, font-weight: font-weight-medium }

        ArticleAuthor
          - type: container
          - children: [AuthorAvatar, AuthorName]
          - tokens: { gap: spacing-xs }

          AuthorAvatar
            - type: image
            - asset: author-avatar
            - tokens: { width: 40px, height: 40px, border-radius: radius-full }

          AuthorName
            - type: text
            - content: [dynamic]
            - tokens: { color: color-text-secondary, font-size: font-size-caption }

  BottomNav
    - type: container
    - children: [NavHome, NavDiscover, NavBookmarks, NavProfile]
    - tokens: { background: color-surface, padding: spacing-sm }
```

**Component type vocabulary:**
- `container` — Groups other components; no visual content of its own beyond background/border
- `text` — Displays text content (static or dynamic)
- `image` — Displays a raster image (references an asset)
- `icon` — Displays an icon (references an asset from icon library)
- `button` — Interactive tap target (contains text, icon, or both)
- `input` — Text input field

### 2. Token File (`tokens.json`)

Flat key-value map of all design tokens. See [Token File Format](#token-file-format) above.

### 3. Asset Inventory (`assets.md`)

List of every non-token visual asset. See [Asset Inventory Format](#asset-inventory-format) above.

---

## Complete Blueprint Extraction Session

```bash
# Working directory: project root
# Input: approved-screen.png (from /frame mode)

# 1. Generate initial containment overlay
amplifier tool invoke nano-banana \
  operation=generate \
  reference_image_path=approved-screen.png \
  output_path=blueprint/containment-overlay.png \
  'prompt=Draw containment overlay with labeled bounding boxes for every component...'

# 2. Check coverage (self-judgment)
amplifier tool invoke nano-banana \
  operation=analyze \
  image_path=blueprint/containment-overlay.png \
  'prompt=Find any uncovered pixels...'
# → If gaps found: define new components, regenerate overlay, check again
# → If no gaps: proceed

# 3. Extract design tokens (raw analysis)
amplifier tool invoke nano-banana \
  operation=analyze \
  image_path=approved-screen.png \
  'prompt=Extract all design tokens: colors (hex), typography (visual description + size/weight),
spacing (px), border-radius (px), shadows. Output structured JSON.'
# → Save as blueprint/tokens.json

# 4. Generate token spec overlay (redline sheet)
amplifier tool invoke nano-banana \
  operation=generate \
  reference_image_path=approved-screen.png \
  output_path=blueprint/token-overlay.png \
  'prompt=Professional redline sheet: UI centered, token callouts surrounding it.
  Left: spacing arrows. Right: typography, colors with swatches, border-radius, shadows.
  Amber callout lines, navy label boxes, 1600x1000px white background.'
# → Verify with nano-banana analyze afterward

# 5. Identify and inventory assets
amplifier tool invoke nano-banana \
  operation=analyze \
  image_path=approved-screen.png \
  'prompt=Identify every visual asset...'
# → Save as blueprint/assets.md

# 6. Compile component spec
# → Using the containment overlay component list + tokens + assets
# → Save as blueprint/component-spec.md

# 7. Final validation
# → Verify: every component has tokens, every asset is referenced,
#    every token is used, no orphans in any file
```

### Output Files

```
blueprint/
├── containment-overlay.png   # Visual proof of 100% pixel coverage
├── component-spec.md         # Hierarchical component specification
├── tokens.json               # Design token values (authoritative record)
├── token-overlay.png         # Redline handoff sheet with all tokens annotated
└── assets.md                 # Asset inventory
```

---

## Definition of Done

The blueprint is **complete** when ALL of the following are true:

1. **100% pixel coverage** — Every pixel in the approved screen belongs to a named component
2. **Component spec is a valid tree** — Single root, no orphan components, every child has a parent
3. **All tokens extracted** — Colors, typography, spacing, and borders are captured with exact values
4. **All assets inventoried** — Every non-text, non-solid-color visual element is listed with name, type, dimensions, and source
5. **Cross-references are valid** — Every `tokens: {}` reference in the component spec points to a real key in `tokens.json`; every `asset:` reference points to a real entry in `assets.md`
6. **No uncovered regions** — The agent's self-judgment loop returned `"coverage_complete": true`

If any condition is not met, the agent continues working. The human is not asked to judge — the agent owns completeness.

---

## When to Use This Skill

Use this pattern when:
- You have an approved screen PNG from `/frame`
- You need to produce a structured spec before code generation
- You want to guarantee nothing in the design is missed

**Don't use for:**
- Storyboard-level work (use `/storyboard` mode)
- Screen refinement (use `/frame` mode)
- Code generation (use `/forge` mode — it consumes this skill's output)

---

## Related Skills

- **screenshot-comparison** — Visual diff for `/forge` convergence loop (consumes this skill's output)
- **icon-finding** — Select icons from Lucide/Heroicons/Feather during asset identification
- **detail-refinement** — Per-screen refinement before blueprint extraction

---

## Summary

**The containment model is the spec.** Every pixel gets a container. The agent judges its own work. Four output files — component spec, tokens.json, token-overlay.png (redline handoff sheet), and assets — give `/forge` everything it needs to generate code with nothing missing.
