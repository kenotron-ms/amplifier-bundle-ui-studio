---
name: moodboard
description: Use when collecting design inspirations and synthesizing a visual moodboard collage and aesthetic brief — covers item intake, collage generation via nano-banana, and brief extraction
version: 1.0.0
license: MIT
---

# Moodboard: From Scattered References to Aesthetic Signal

Collect disparate design references — screenshots, images, URLs, text descriptions — and synthesize them into a visual collage and structured aesthetic brief. The brief feeds directly into `/storyboard` as the aesthetic direction input.

## Overview

**Input:** Anything the user provides: image files, screenshots, URL references, text snippets describing styles, adjectives, reference apps
**Output:** `collage.png` (visual reference for nano-banana) + `aesthetic-brief.md` (structured text for storyboard prompts)
**Judge:** Human — they approve the brief before the moodboard is considered done

---

## Item Storage Convention

Every item saved to `ui-studio/moodboard/items/` uses a sequential number prefix:

```
001-{slug}.{ext}
002-{slug}.{ext}
...
```

**Naming rules:**
- Slugify the source: `screenshot-of-linear-app.png` → `001-linear-screenshot.png`
- Text snippets → `.md` files: `002-style-note.md`
- URL references → `.md` files with the URL and any description: `003-dribbble-ref.md`
- Keep slugs short (≤30 chars), lowercase, hyphens only

**Metadata file:** After saving each item, append one line to `ui-studio/moodboard/manifest.md`:

```
001 | linear-screenshot.png | image | Dark app UI with high-contrast typography
002 | style-note.md         | text  | "Generous whitespace, editorial feel, SF Pro"
003 | dribbble-ref.md       | url   | https://... — payment dashboard reference
```

This manifest is used at synthesis time to know what's in the collection.

---

## Input Type Handling

### Image files / screenshots
- Save as-is to `items/` with the numbered filename
- Note: if the user provides a path to an existing file, copy it:
  ```bash
  cp {source} ui-studio/moodboard/items/{NNN}-{slug}.png
  ```
- Add to manifest with a brief 1-line description of what's visible

### Text snippets / style descriptions
- Save as `.md` file
- Content: exactly what the user wrote, verbatim. No summarizing.
- Example:
  ```md
  # Style Note
  Dark background, almost black. Accent color is electric violet.
  Typography: Plus Jakarta Sans, generous line-height, editorial weight contrast.
  Mood: focused, premium, calm — not playful.
  ```

### URLs
- Save a `.md` file with the URL and any description the user added:
  ```md
  # Reference URL
  https://dribbble.com/shots/12345-payment-dashboard
  
  User note: "The card shadows and subtle grid lines here"
  ```
- Do NOT attempt to fetch or screenshot the URL — the user will provide what's needed

---

## Synthesis: Generating the Collage

When synthesis is triggered, read the manifest to understand what's in `items/`. Then generate the collage with nano-banana.

### Collage prompt structure

The collage prompt must:
1. Instruct nano-banana to compose all available images into a coherent moodboard layout
2. Incorporate text snippets as typographic specimens overlaid on the composition
3. Apply an overall dark-studio aesthetic (dark background, labels)

```bash
amplifier tool invoke nano-banana \
  operation=generate \
  output_path=ui-studio/moodboard/collage.png \
  'prompt=Create a cohesive designer moodboard composition. Dark slate background (#0D1117).

REFERENCE IMAGES to incorporate (compose them organically, not in a rigid grid):
{list each image item with its description}

TEXT SPECIMENS to include as typographic overlays:
{list each text/note item, rendered as editorial type in white on dark}

COMPOSITION RULES:
- Arrange images at varied scales with slight overlaps (±3° rotation on some)
- Leave breathing room — not a cramped collage
- Add thin white hairline borders on image tiles
- Color palette strip extracted from the images, shown along one edge
- Small annotation labels in monospace 9px white: category tags on each tile
- Overall: authentic Figma/Notion moodboard feel, professional design tool

Label: top-left pill badge "ui-studio/moodboard — {N} references"'
```

---

## Synthesis: Extracting the Aesthetic Brief

After the collage is generated, analyze all items and produce `aesthetic-brief.md`. This is the structured output that `/storyboard` reads as its aesthetic direction.

### Analysis approach

Work through each item type:

**From images:** Look for (or infer from descriptions): dominant background color, accent colors, typography density, border-radius style, icon style, component density
**From text notes:** Extract direct style preferences verbatim — these are the user's explicit intent
**From URL notes:** Use the user's annotation to understand which specific aspect they're referencing

### Brief format

```md
# Aesthetic Brief

> Generated from {N} references. Review and edit before using in /storyboard.

## Palette Impression
- Background: {dominant bg color, with approximate hex if identifiable}
- Accent: {primary accent color}
- Text: {primary text treatment}
- Surface: {card/panel color if distinct}

## Typography Feel
- Style: {humanist sans / geometric sans / serif / monospace — pick the closest}
- Weight distribution: {light-heavy contrast / uniform medium / bold-dominant}
- Density: {tight and compact / balanced / airy and editorial}

## Layout Tendencies
- Spatial rhythm: {tight / balanced / generous whitespace}
- Grid style: {strict grid / organic / full-bleed / card-based}
- Navigation pattern: {bottom tab / sidebar / top nav / minimal}

## Mood Keywords
{5–8 single words or short phrases directly from references, e.g.: "minimal", "premium", "focused", "editorial", "calm", "high-contrast"}

## Reference Summary
{2–3 sentences synthesizing what the collection is saying as a whole. Write as if briefing a designer: "The references point toward a dark, editorial aesthetic with strong typographic hierarchy..."}

## For /storyboard
Paste these keywords into your aesthetic direction:
> {one-line distillation suitable for direct use in a storyboard prompt}
```

---

## Collage-as-Reference-Image for Storyboard

When `/storyboard` reads the aesthetic brief and finds `ui-studio/moodboard/collage.png`, it should pass the collage as `reference_image_path` to nano-banana. This gives screen generation **real visual signal** — not just words.

The storyboard agent checks for `collage.png` automatically. No user action needed.

---

## Re-synthesis

The user can add more items and re-synthesize at any time. Each synthesis overwrites `collage.png` and `aesthetic-brief.md`. Previous versions are not archived unless the user asks.

If the user adds items after the brief was already approved, re-run synthesis and present the updated brief for review again.

---

## Minimum Viable Collection

Before synthesizing, the collection should have at least **2 items**. A single reference is too narrow to synthesize from. With 1 item, ask:

> "One reference is a starting point. Add at least one more — a color palette, a style description, another screenshot — and I'll synthesize from there."

A collection of 5–10 items typically produces the richest brief. Beyond 15, diminishing returns — synthesis quality plateaus.
