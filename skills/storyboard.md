---
skill:
  name: storyboard
  version: 1.0.0
  description: Use when generating a multi-screen UX flow from an app brief using nano-banana — ensures visual consistency by generating all screens simultaneously in a single call
  keywords: [storyboard, multi-screen, UX flow, consistency, nano-banana, simultaneous generation]
  author: kenotron-ms
  license: MIT
---

# Storyboard: Multi-Screen UX Flow Generation

Generate a complete multi-screen application flow from an app brief using a single nano-banana call. All screens are produced simultaneously to enforce visual consistency across the entire user journey.

## Overview

The storyboard is the first visual artifact in the design pipeline. It answers:
1. **What screens exist?** — The complete screen inventory
2. **How do they connect?** — Navigation flow and transitions
3. **Do they feel like one app?** — Consistent visual language across every screen

**Result:** A single storyboard image showing all screens in a flow layout, plus a named screen inventory with labeled transitions.

---

## Why Simultaneous Generation

This is the core principle of the storyboard skill. **All screens must be generated together in one nano-banana call.**

### The Consistency Problem

When screens are generated one at a time (sequentially), each generation is an independent event. The model has no memory of what it produced before. This causes:

- **Visual drift** — Screen 1 uses rounded cards with soft shadows; Screen 5 shifts to flat cards with hard borders
- **Navigation inconsistency** — The tab bar has 4 items on one screen and 5 on another
- **Component divergence** — Buttons, headers, and list items look subtly different on every screen
- **Color palette drift** — Blues shift across screens because each generation picks a slightly different hue

### The Simultaneous Solution

When all screens are generated in a single call, the model sees the **full picture at once**. It enforces:

- **Consistent design language** — The same card style, button shape, and spacing rules apply everywhere because the model is rendering them side by side
- **Shared navigation patterns** — Tab bars, headers, and back buttons are visually identical across screens because they exist in the same image
- **Component reuse** — The model naturally reuses visual elements when it can see all screens together
- **Coherent color palette** — One generation means one palette decision, applied uniformly

**Think of it this way:** Asking an artist to paint six canvases one at a time in separate sessions produces six related but inconsistent paintings. Asking them to paint all six on one large canvas produces a cohesive series — they can see the whole and keep it unified.

---

## When to Use This Skill

Use this pattern when:
- You have an app brief (description of what the app does, who it's for, and the core user journey)
- You need to establish the screen inventory and navigation flow before refining individual screens
- You want a visual artifact to discuss with stakeholders before committing to detailed design

**Don't use for:**
- Refining a single screen to pixel-level detail (use `/frame` instead)
- Generating code from an approved design (use `/blueprint` then `/forge`)
- Comparing implementation screenshots against mockups (use `screenshot-comparison`)

---

## Prerequisites

**Tools required:**
- Nano Banana tool (`tool-nano-banana`) — Gemini image generation
- `GOOGLE_API_KEY` set in environment

**Inputs required:**
- App brief with enough detail to generate meaningful screens. At minimum:
  - What the app does (core purpose)
  - Who uses it (target user)
  - The primary user journey (sequence of actions)
  - Visual direction (aesthetic, mood, style keywords)
  - Approximate screen count or list of screen names

If the brief is incomplete, run a sufficiency check first — ask targeted questions one at a time until these are covered.

---

## Structuring the Nano-Banana Prompt

The prompt structure is critical. It must give the model everything it needs to generate all screens coherently in a single pass.

### Prompt Structure

Follow this order:

1. **Visual style declaration** (once, applies to all screens)
2. **Complete screen list** (all names upfront)
3. **User journey narrative** (the flow between screens)
4. **Per-screen descriptions** (content and purpose of each)
5. **Layout and labeling instructions** (how to arrange the output)

### Worked Example

For a podcast discovery app with 5 screens:

```bash
amplifier tool invoke nano-banana \
  operation=generate \
  output_path=storyboard.png \
  'prompt=Generate a complete multi-screen UX storyboard for a mobile podcast discovery app.

VISUAL STYLE (applies to ALL screens):
- Modern, clean iOS aesthetic with a dark theme
- Rounded cards with subtle depth shadows
- SF Pro-style typography
- Accent color: warm coral (#FF6B6B)
- Bottom tab navigation on every main screen
- Status bar visible on all screens

SCREENS (generate ALL of these together in one image):
1. Home
2. Discover
3. Episode Player
4. Library
5. Profile

USER JOURNEY:
Home is the landing screen showing personalized recommendations.
User taps a category on Home → arrives at Discover with filtered results.
User taps a podcast on Discover → opens Episode Player with playback controls.
User saves a podcast from Episode Player → it appears in Library.
User taps their avatar on any screen → opens Profile with settings and history.

SCREEN DESCRIPTIONS:
1. Home — Hero featured podcast at top, "Continue Listening" row, "Recommended For You" grid. Bottom tab: Home (active), Discover, Library, Profile.
2. Discover — Search bar at top, category pills (Comedy, Tech, News, Health), trending podcast grid below. Bottom tab: Discover (active).
3. Episode Player — Large album art, episode title, podcast name, progress bar, play/pause/skip controls, volume slider, "Save to Library" button.
4. Library — Saved podcasts list with cover art thumbnails, episode count, last played date. Segmented control: All / In Progress / Downloaded. Bottom tab: Library (active).
5. Profile — User avatar and name, listening stats (hours, episodes, streak), settings list (Notifications, Audio Quality, Theme, Account). Bottom tab: Profile (active).

OUTPUT LAYOUT:
- Arrange all 5 screens in a clear flow layout (grid or row)
- Label each screen with its name above or below the frame
- Draw arrows or lines between screens showing navigation transitions
- Label each transition with the action that triggers it (e.g., "Tap category", "Tap podcast", "Save", "Tap avatar")
- Show entry point clearly (Home = start)
- Low-to-mid fidelity: focus on layout, hierarchy, and flow — not pixel-perfect details
- All screens should be at mobile aspect ratio (roughly 9:19.5)' \
  aspect_ratio=16:9 \
  resolution=2K \
  use_thinking=true \
  number_of_images=1
```

### Key Prompt Principles

| Principle | Why |
|-----------|-----|
| Declare visual style **once at the top** | The model applies it uniformly to all screens |
| List **all screen names upfront** | The model plans the full layout before drawing |
| Describe the **user journey as a narrative** | Creates logical flow the model can follow |
| Give **per-screen content descriptions** | Ensures each screen has the right elements |
| Request **labels and transition arrows** | Makes the output self-documenting |
| Specify **low-to-mid fidelity** | Keeps focus on flow and structure, not polish |

---

## Describing Screen Transitions and State Connections

Transitions are the connective tissue of the storyboard. They answer: "What does the user do to get from Screen A to Screen B?"

### Transition Format

In the prompt, describe transitions as action-triggered movements:

```
User [action] on [Screen A] → arrives at [Screen B]
```

Examples:
```
User taps a podcast card on Discover → opens Episode Player
User swipes right on Episode Player → returns to Discover
User taps "Save" on Episode Player → podcast appears in Library
User taps the profile tab → opens Profile from any screen
```

### Entry and Exit States

Always specify:
- **Entry point** — Which screen appears first when the app launches
- **Exit states** — Screens that lead to external actions (logout, share, deep link out)
- **Return paths** — How the user gets back (back button, tab bar, swipe)

### Visual Representation in Output

Request that the storyboard shows:
- **Arrows** between screens labeled with the triggering action
- **Screen names** clearly labeled above or below each frame
- **Entry marker** on the starting screen (e.g., "START" label or a distinct arrow)
- **Bidirectional arrows** where navigation goes both ways (e.g., tab bar screens)

---

## Representing Navigation Flow in Output

After the storyboard image is generated, produce a structured **screen inventory** alongside it.

### Screen Inventory Format

```
Screen Inventory (5 screens):
1. Home — Personalized landing with recommendations and continue listening
2. Discover — Browse and search podcasts by category
3. Episode Player — Playback controls and episode details
4. Library — Saved and downloaded podcasts
5. Profile — User settings and listening history
```

### Transition Map

```
Transitions:
- Home → Discover: Tap category or search
- Home → Episode Player: Tap "Continue Listening" episode
- Discover → Episode Player: Tap podcast card
- Episode Player → Library: Tap "Save to Library"
- Any screen → Profile: Tap profile tab
- Tab bar (persistent): Home, Discover, Library, Profile
```

This inventory is the **handoff artifact** to `/frame`. When the human picks a screen to refine, they reference it by name from this list.

---

## Fidelity Guidance

The storyboard stage operates at **low-to-mid fidelity**. Getting the fidelity right is important — too low and there's nothing to react to; too high and you waste iterations on details that will change.

### What Storyboard Fidelity Means

| Aspect | Storyboard Fidelity | NOT Storyboard (too detailed) |
|--------|---------------------|-------------------------------|
| **Layout** | Block-level placement of elements | Pixel-precise padding values |
| **Typography** | Relative hierarchy (large title, body, caption) | Exact font name, weight, and size |
| **Color** | Directional palette (dark theme with coral accent) | Exact hex values for every element |
| **Icons** | Recognizable placeholders (house, search, play) | Final icon assets from a specific library |
| **Images** | Representative placeholders or abstract shapes | Real photography or final illustrations |
| **Spacing** | Proportional and balanced | Measured in exact dp/pt values |
| **Interactions** | Labeled transitions between screens | Microinteractions and animations |

### What to Focus On

During storyboard review, the human should evaluate:
- **Does the flow make sense?** Can you trace the user journey from start to finish?
- **Are all screens accounted for?** Is anything missing from the journey?
- **Does the navigation model work?** Tab bar vs. stack navigation vs. modals — is the pattern clear?
- **Is the hierarchy right?** Does each screen emphasize the right content?
- **Does it feel like one app?** Consistent visual language across all screens?

### What to Ignore (for now)

- Exact colors (directional is enough)
- Typography details (hierarchy matters, specific fonts don't yet)
- Icon accuracy (the right concept matters, not the right asset)
- Content fidelity (placeholder text is fine)
- Pixel alignment (close enough is good enough)

These are all refined in `/frame` and `/blueprint`.

---

## What a Complete Storyboard Output Looks Like

A storyboard is **done** when it has all three components:

### 1. Storyboard Image

A single image (typically 16:9 or wider) showing:
- All screens arranged in a flow layout (grid, row, or diagram)
- Each screen at mobile aspect ratio
- Screen names labeled
- Transition arrows with action labels
- Consistent visual style across all screens
- Entry point clearly marked

### 2. Screen Inventory

A numbered list of every screen with a one-line purpose:

```
Podcast App — 5 screens:
1. Home — Personalized landing with recommendations and continue listening
2. Discover — Browse and search podcasts by category
3. Episode Player — Playback controls and episode details
4. Library — Saved and downloaded podcasts
5. Profile — User settings and listening history
```

### 3. Transition Labels

A list of actions connecting screens:

```
Transitions:
- Home → Discover: Tap category pill
- Home → Episode Player: Tap continue listening item
- Discover → Episode Player: Tap podcast card
- Episode Player → Library: Tap save button
- Any main screen ↔ Any main screen: Bottom tab bar
```

---

## When Is It Done? (Human Approval Criteria)

The storyboard loop is judged by a **human**, not an agent. This is a taste decision.

### The Loop

```
generate storyboard → present to human → human reacts → refine → repeat
```

### Approval Signals

The storyboard is **approved** when the human says something like:
- "Looks good"
- "That works, let's move on"
- "Happy with the flow"
- "Yes, this captures what I want"
- Any affirmative that they accept the screen inventory and flow

### Refinement Signals

The storyboard needs **another iteration** when the human says:
- "Missing a screen for [X]" → Add the screen and regenerate
- "Screen A and Screen B should be combined" → Merge and regenerate
- "The flow doesn't make sense between [X] and [Y]" → Revise transitions and regenerate
- "The style is too [dark/colorful/minimal/etc]" → Adjust style declaration and regenerate
- "Can we try a different layout?" → Adjust arrangement and regenerate

### Handoff

When approved, present the transition to the next mode:

> *"Storyboard complete — [N] screens: [Screen 1], [Screen 2], ... Which screen do you want to refine first? Type `/frame` and name the screen."*

---

## Iteration Tips

### First generation is rarely final

Expect 2-4 iterations. The first generation establishes a baseline for the human to react to. Reactions are more productive than descriptions — it's easier to say "make the cards bigger" than to describe the ideal card size from scratch.

### Refine, don't restart

When iterating, modify the prompt — don't rewrite it. Change the specific element the human flagged:
- Adjust a screen description
- Add or remove a screen from the list
- Change the style declaration
- Modify a transition

Keep everything else identical so the model produces a minimal diff, not a completely different storyboard.

### Use the reference image for refinement

After the first generation, subsequent iterations pass the previous storyboard as `reference_image_path`. This sends the image as an **inline image block** in the multimodal call — Gemini literally sees your previous storyboard and applies targeted changes to it, rather than generating from scratch. This produces tight diffs instead of full regenerations.

```bash
amplifier tool invoke nano-banana \
  operation=generate \
  reference_image_path=ui-studio/storyboards/storyboard_v1.png \
  output_path=ui-studio/storyboards/storyboard_v2.png \
  'prompt=TAKE THIS STORYBOARD and make these changes:
  - Add a 6th screen: Onboarding (shown before Home for new users)
  - Make the cards larger on the Home screen
  - Keep everything else the same' \
  aspect_ratio=16:9 \
  resolution=2K \
  use_thinking=true \
  number_of_images=1
```

This produces tighter iterations because the model starts from the existing layout rather than generating from scratch.

---

## Standard Output Location

**Convention:** Storyboard artifacts go in `.amplifier/storyboard/`

```
.amplifier/storyboard/
  storyboard_v1.png        # First generation
  storyboard_v2.png        # After refinement
  storyboard_final.png     # Approved version
  screen_inventory.md      # Named screen list + transitions
```

---

## Related Skills

- **detail-refinement** — Per-screen refinement with contradiction detection (used in `/frame`)
- **screenshot-comparison** — Three-stage visual annotation for implementation validation (used in `/forge`)
- **font-matching** — Systematic font identification for design token extraction
- **icon-finding** — Topology-aware icon selection from standard libraries

---

## Summary

**Core principle:** Generate all screens simultaneously in one nano-banana call to enforce visual consistency.

**Output:** Storyboard image + screen inventory + transition labels.

**Fidelity:** Low-to-mid. Layout, hierarchy, and flow — not pixel-perfect details.

**Judge:** Human. Approved when the human says the flow works and all screens are accounted for.

**Handoff:** Approved storyboard feeds into `/frame` where individual screens are refined to full fidelity.
