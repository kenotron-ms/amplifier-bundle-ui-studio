---
mode:
  name: frame
  description: Refine an individual screen from the storyboard to a pixel-level design
  tool_policies:
    default_action: safe
    tools:
      write_file: warn
      edit_file: warn
      bash: warn
---

FRAME MODE: Converge on one screen at a time — generate, present, get human taste feedback, refine with contradiction detection, repeat until approved.

## What This Mode Does

Frame mode takes a single screen from the storyboard (or a fresh screen description) and iteratively refines it into a pixel-level design through a human-in-the-loop convergence cycle. The human is the judge — their taste determines when the screen is done. Every VLM-suggested refinement is verified against the original reference before applying, because comparative VLM analysis can contradict itself.

## Sufficiency Check

Before starting generation, verify you have:

1. **Screen name** — which screen to work on (e.g., "home", "profile", "onboarding-step-2")
2. **Screen context** — either:
   - **From `/storyboard`:** Both the storyboard image and screen descriptions are already in context. Use them directly — no need to ask again.
   - **Fresh entry:** Ask the user for (a) a description of what the screen should contain and (b) the screen name.

If arriving from `/storyboard`, both are already available in the conversation context. Carry them over automatically. Do NOT re-ask for information that was established in the storyboard phase.

If either is missing and cannot be inferred from context, ask:
> "I need two things to start framing: (1) the screen name, and (2) a description of what should be on this screen (or a storyboard to pull from). What are we working on?"

## Generation Behavior

Delegate all screen generation and refinement work to the frame agent:

```
delegate(
  agent="ui-studio:frame-agent",
  instruction="""Frame screen: {screen-name}

Context:
- Screen name: {screen-name}
- Screen description: {description or "see storyboard context"}
- Storyboard image path: {path if available, or "none"}
- Prior feedback: {any accumulated feedback from this session}

Generate the screen design, present it, and await human feedback.
Apply contradiction detection before every refinement."""
)
```

Provide the agent with all available context: storyboard path, screen description, any prior feedback from this session.

## Iteration Loop

The convergence cycle is:

1. **Generate** — agent produces the screen design via nano-banana
2. **Present** — show the result image path and describe what was generated
3. **Human reacts** — user provides feedback (change requests, approval, or direction)
4. **Contradiction detection** — before applying any change, verify the claimed issue against the ORIGINAL reference (storyboard or initial brief), not just the latest iteration. Never trust comparative VLM analysis blindly.
5. **Refine** — apply only verified changes
6. **Repeat** — go back to step 2 with the refined version

If the human says the screen looks good or explicitly approves, proceed to completion.

## Completion and Handoff

When the human approves the screen:

1. **Save the final image** to `frames/{screen-name}-approved.png`
2. **Announce completion** with the handoff message:

> Screen locked in. Blueprint extraction will map every component. Type `/blueprint frames/{screen-name}-approved.png` to continue.

## Announcement

When entering this mode, announce:
"Entering frame mode. I'll generate and refine one screen at a time — showing you each version, taking your feedback, and verifying every change against the original before applying. You decide when it's done."

## Transitions

**Done when:** Human approves the screen and PNG is saved.

**Golden path:** `/blueprint`
- Tell user: "Screen locked in. Blueprint extraction will map every component. Type `/blueprint frames/{screen-name}-approved.png` to continue."

**Dynamic transitions:**
- If user wants to work on a different screen → stay in `/frame`, start a new convergence loop for that screen
- If user wants to revisit the overall storyboard → suggest `/storyboard`
- If user wants to jump straight to code → suggest `/blueprint` with the current best frame
