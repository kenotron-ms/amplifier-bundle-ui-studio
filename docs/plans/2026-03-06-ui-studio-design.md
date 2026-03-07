# amplifier-bundle-ui-studio — Design

**Date:** 2026-03-06
**Status:** Approved

---

## Vision

A guided design-to-code pipeline expressed as four progressive modes. The human shows up, describes an idea, and the bundle walks them from concept to production code — asking the right questions, generating the right artifacts, and knowing when to hand off to the next phase.

The fundamental unit of every mode is a **convergence loop**: a source input fed through a multi-step function, judged by either a human or an agent, producing a defined output. The human never needs to understand the pipeline — the modes guide them forward.

---

## Bundle Identity

```yaml
bundle: ui-studio
version: 1.0.0
description: Guided design-to-code pipeline via four progressive modes

dependencies:
  - source: git+https://github.com/kenotron-ms/amplifier-module-tool-nano-banana@main
```

**Hero dependency:** `tool-nano-banana` (Gemini 3 Pro Image) — used in every mode for generation, annotation, and visual scoring.

**Reference:** The existing `amplifier-bundle-ui-mockup` bundle contains proven patterns for screenshot comparison, font matching, icon finding, and detail refinement. These are carried forward as skills.

---

## The Convergence Model

Every mode is the same shape:

```
input → [convergence function] → output
              ↑
           judge
       (human or agent)
```

The judge determines when to stop iterating. Human judges are used where taste is required (Loops 1–2). Agent judges are used where quality can be measured (Loops 3–4).

---

## Phase 0: Brainstorm (optional upstream)

Before any mode runs, the concept must be clear enough to generate visuals. This phase is covered by the existing `/brainstorm` mode from the superpowers bundle.

If the human arrives at `/storyboard` with a clear brief, Phase 0 is skipped. If not, `/storyboard`'s sufficiency check gathers what it needs inline — it does not require a separate brainstorm session.

The design-specific questions `/storyboard` needs that generic brainstorming may not cover:
- How many screens?
- What is the primary user journey?
- What is the visual language / aesthetic direction?

---

## The 4 Modes

### `/storyboard`
> Generate a consistent UX flow across all screens

**Convergence function:**
```
Input:     app brief (gathered via sufficiency check)
Judge:     HUMAN (taste)
Steps:     generate full multi-screen flow (all screens at once, nano-banana)
           → present → human reacts → refine → repeat
Stops:     human approves
Output:    storyboard image + named screen inventory
```

**Key design decision:** All screens are generated simultaneously in one nano-banana call. This is how visual consistency is enforced — the model sees the full picture, not individual screens in isolation.

**Entry behavior:** Sufficiency check evaluates whether the brief is detailed enough to generate a meaningful storyboard. Asks targeted questions (one at a time) until the bar is met. If human arrives with a complete brief, proceeds immediately.

**Handoff:** *"Happy with the flow? → `/frame` to refine individual screens"*

---

### `/frame`
> Refine individual screens to pixel-level designs

**Convergence function:**
```
Input:     storyboard + selected screen name
Judge:     HUMAN (taste)
Steps:     generate screen design (nano-banana)
           → present → human reacts → refine → repeat
           (applies contradiction detection: verify each issue against
            original before applying — never trust comparative VLM analysis blindly)
Stops:     human approves screen
Output:    approved screen PNG
```

**Entry behavior:** Needs a storyboard and a screen to work on. If arriving from `/storyboard`, the screen inventory is already available — human just picks which screen to start with.

**Handoff:** *"Screen locked in? → `/blueprint` to extract the component spec"*

---

### `/blueprint`
> Extract the component spec — no pixel without a container

**Convergence function:**
```
Input:     approved screen PNG
Judge:     AGENT (completeness — every pixel has a container?)
Steps:     generate containment overlay (nano-banana)
           → check coverage → fill gaps
           → extract design tokens
           → generate all assets present in mockup
             (backgrounds, thumbnails, illustrations, icons)
           → validate component spec
Stops:     agent judges spec is complete
Output:    component spec, token file, asset inventory
```

**Key design decision:** The containment model is the spec. Every pixel must belong to a component before the spec is considered complete. The agent self-judges this — not the human.

**Entry behavior:** Needs an approved screen PNG. Checks for one before starting.

**Handoff:** *"Spec complete? → `/forge` to generate code"*

---

### `/forge`
> Converge code to match the mockup — automated until done

**Convergence function:**
```
Input:     component spec + token file + asset inventory
Judge:     AGENT (nano-banana visual similarity score)
Steps:     generate/update code
           → Playwright screenshot
           → ImageMagick side-by-side (original LEFT | current RIGHT)
           → nano-banana annotate differences
           → interpret annotations → apply fix
           → repeat
Stops:     match % ≥ threshold (default 95%)
Output:    production code
```

**Entry behavior:** Needs a complete blueprint. Checks for one before starting.

**Handoff:** *"Converged at X% match — implementation ready"*

---

## The Macro Loop

The mode pipeline is not strictly linear. Modes are re-enterable with new instructions:

```
[brainstorm]  (optional, if concept is fuzzy)
      ↓
/storyboard → /frame → /blueprint → /forge
                              ↑____________|
                  (forge stuck? refine blueprint and retry)
        ↑____________________|
      (blueprint wrong? back to frame for screen adjustment)
```

When re-entering a mode, it checks what has changed and continues from the new state — it does not start from scratch.

**Mode progression is guided:** Each mode suggests the next step when its convergence function completes. The human never needs to know the pipeline sequence.

---

## Agents

One specialist per mode. Each agent owns its convergence function entirely.

| Agent | Mode | Judge | Key capability |
|-------|------|-------|----------------|
| `storyboard-agent` | `/storyboard` | Human | Multi-screen simultaneous generation for consistency enforcement |
| `frame-agent` | `/frame` | Human | Per-screen refinement with contradiction detection |
| `blueprint-agent` | `/blueprint` | Agent (self) | Containment model overlay + completeness judgment + asset extraction |
| `forge-agent` | `/forge` | Agent (nano-banana score) | Screenshot comparison convergence loop |

---

## Skills

Skills load methodology into agents. Mix of carried forward from `ui-mockup` and new.

| Skill | Status | Purpose |
|-------|--------|---------|
| `storyboard` | New | Multi-screen consistency enforcement; how to instruct nano-banana to see the whole flow |
| `detail-refinement` | Carry forward | Contradiction detection pattern for per-screen iteration |
| `blueprint-extraction` | New | Containment model methodology; completeness criteria; asset identification |
| `screenshot-comparison` | Carry forward | Three-stage visual annotation pattern (ImageMagick → nano-banana annotate → VLM analyze) |
| `font-matching` | Carry forward | Systematic Google Fonts search (1,600+ families, batch VLM ranking) |
| `icon-finding` | Carry forward | Topology-aware icon selection (Lucide → Heroicons → Feather → FA) |

---

## Mode Transition Design

Each mode's closing behavior:

1. **Detects** when its convergence function has reached a satisfying output
2. **Summarizes** what was produced (storyboard screens, approved PNG path, spec location, match %)
3. **Suggests** the next mode with enough context that the human can act immediately
4. **Passes forward** relevant artifacts as context (paths, screen names, spec files)

Example transition from `/storyboard` → `/frame`:
> *"Storyboard complete — 6 screens: Home, Discover, Article, Profile, Settings, Onboarding. Which screen do you want to refine first? Type `/frame` and name the screen."*

---

## Implementation Notes

- The existing `ui-mockup` bundle's `screenshot-comparison-loop.yaml` recipe is the reference implementation for `/forge`'s convergence function — but in `ui-studio` this logic lives inside `forge-agent`, not a recipe
- Skills are loaded on demand (not always in context) — each agent loads its relevant skills at the start of its work
- The `tool-nano-banana` `reference_image_path` parameter is the key primitive for visual annotation — the agent passes an existing image to get an overlay rather than generating from scratch
