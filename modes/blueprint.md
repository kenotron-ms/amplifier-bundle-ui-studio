---
mode:
  name: blueprint
  description: Extract the component spec from an approved screen — every pixel must belong to a component
  shortcut: blueprint
  tools:
    safe:
      - read_file
      - glob
      - grep
      - load_skill
      - delegate
      - recipes
      - nano-banana
      - write_file
      - edit_file
      - bash
  default_action: block
---

BLUEPRINT MODE: Fully automated component spec extraction from an approved screen.

This mode takes an approved screen PNG and extracts a complete component specification, design tokens, and asset inventory. The agent self-judges completeness using the containment model — every pixel must belong to a named component. No human approval is needed during extraction; the agent loops until coverage is 100%.

## Sufficiency Check

This mode requires an **approved screen PNG path** to begin.

- If arriving from `/frame`, the approved screen path should already be in context. Confirm it and proceed.
- If starting fresh, ask the user: "Which approved screen PNG should I extract the blueprint from? Provide the file path."

Do NOT proceed without a valid PNG path.

## Agent Delegation

This mode is **fully automated**. Immediately delegate all work to the blueprint agent:

```
delegate(
  agent="ui-studio:blueprint-agent",
  instruction="""Extract a complete blueprint from the approved screen.

Screen PNG path: {path_to_approved_screen}

Generate the containment overlay, self-judge coverage until 100%, then extract the component spec, design tokens, and asset inventory.

Write all outputs to blueprints/{screen-name}/""",
  context_depth="recent",
  context_scope="conversation"
)
```

You are the orchestrator in this mode — the agent does all the work. Wait for it to complete, then present the results to the user.

## Output Location

All output files are written to `blueprints/{screen-name}/`:

| File | Purpose |
|------|---------|
| `containment-overlay.png` | Visual proof of 100% pixel coverage |
| `component-spec.md` | One section per component with type, hierarchy, tokens, content |
| `tokens.json` | All design tokens: colors, typography, spacing, border-radius |
| `assets.md` | Inventory of icons, images, backgrounds with source info |

## Handoff

When the agent reports completion, present the results and suggest the next step:

```
Blueprint complete — component spec, tokens, and assets extracted.

Files written to blueprints/{screen-name}/:
- component-spec.md ({N} components)
- tokens.json
- assets.md

Type `/forge blueprints/{screen-name}/` to generate code.
```

## Announcement

When entering this mode, announce:
"Entering blueprint mode. I'll extract a complete component spec from the approved screen — every pixel will be assigned to a named component."

## Transitions

**Done when:** Agent confirms 100% coverage and all three output files are written.

**Golden path:** `/forge blueprints/{screen-name}/`
- Tell user: "Blueprint complete — component spec, tokens, and assets extracted. Type `/forge blueprints/{screen-name}/` to generate code."
