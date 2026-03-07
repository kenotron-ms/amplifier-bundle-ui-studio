---
mode:
  name: forge
  description: Converge code to match the mockup through an automated screenshot comparison loop
  shortcut: forge
  tools:
    safe:
      - read_file
      - glob
      - grep
      - load_skill
      - delegate
      - recipes
      - write_file
      - edit_file
      - bash
      - apply_patch
  default_action: block
---

FORGE MODE: Automated visual convergence. The agent does the work — you watch it converge.

This mode runs a fully automated screenshot comparison loop that generates code from a blueprint, captures screenshots, compares them against the approved mockup using nano-banana, and iteratively fixes differences — one at a time, highest-priority first — until the implementation visually matches the design at ≥95% similarity.

## Sufficiency Check

Before delegating, verify a complete blueprint directory is available containing:

- `component-spec.md` — full component specification
- `tokens.json` — design tokens (colors, spacing, typography)
- `assets.md` — asset inventory (images, icons, dimensions)
- `approved-screen.png` — the approved mockup screenshot to converge toward

**If arriving from `/blueprint`:** the blueprint directory path is already in context. Confirm the four required files exist and proceed.

**If entering fresh:** ask the user for the blueprint directory path:
```
"To start forging, I need the path to your blueprint directory.
It should contain: component-spec.md, tokens.json, assets.md, and approved-screen.png.

What's the path?"
```

Do NOT proceed until all four files are confirmed present.

## Framework Selection

If the target framework has not been specified (by the user or in a previous mode), ask:

```
"Which target framework?
1. HTML + Tailwind CSS (default)
2. React + Tailwind CSS
3. Vue + Tailwind CSS
4. Svelte + Tailwind CSS

Press enter for default, or choose 1-4."
```

If the user just presses enter or doesn't specify, use **HTML + Tailwind CSS**.

## Agent Delegation

Once the blueprint is confirmed and framework selected, delegate the entire convergence loop to the forge agent:

```
delegate(
  agent="ui-studio:forge-agent",
  instruction="Blueprint directory: [path]. Target framework: [framework]. Approved mockup: [path/to/approved-screen.png]. Converge to ≥95% match.",
  context_depth="recent"
)
```

This is **fully automated** — the agent generates code, captures screenshots, compares, fixes, and repeats without human intervention. Do not interrupt the agent unless it surfaces a stuck state.

## Progress Reporting

The forge agent reports match percentage at each iteration:
```
Iteration 1: 62% match. Fixed: missing navigation bar background gradient.
Iteration 2: 71% match. Fixed: hero card border-radius too small.
...
```

Surface these reports to the user as they arrive so progress is visible.

## Exit Behavior

When the agent reports convergence (match ≥ 95%):
- Display the final message: **"Converged at X% match after N iterations."**
- List all generated files in `output/`
- Show the final comparison image path

## Stuck Behavior

If the agent surfaces a stuck state (match percentage did not improve over 3 consecutive iterations):
- Display the agent's stuck report including the current match percentage
- Show the side-by-side comparison image to the user
- Ask the user for direction: "The forge loop is stuck at X%. Here's the current comparison. What should it try differently?"
- Pass the user's guidance back to the agent and let it resume the loop