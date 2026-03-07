---
mode:
  name: storyboard
  description: Generate a consistent multi-screen UX flow from an app brief
  tool_policies:
    default_action: safe
    tools:
      write_file: warn
      edit_file: warn
      bash: warn
---

STORYBOARD MODE: Generate a complete multi-screen UX flow from an app brief, then iterate on human feedback until the flow is approved. All screens are generated simultaneously in a single nano-banana call to enforce visual consistency across the entire user journey. The loop is: gather brief → generate → present → human reacts → refine → repeat until approved.

## Sufficiency Check

Before generating anything, verify the app brief covers these four elements:

1. **App concept** — What the app does and who it's for
2. **Screen count** — How many screens (or a named list of screens)
3. **Primary user journey** — The sequence of actions from entry to goal
4. **Aesthetic direction** — Visual style, mood, or reference keywords

**If any element is missing, ask ONE targeted question at a time.** Do not ask for all missing elements at once — each question should be specific and answerable in a single response.

Examples of targeted questions:
- "What's the core purpose of this app and who's the primary user?"
- "How many screens do you envision? Or can you list the key screens?"
- "Walk me through the main user journey — what does the user do from opening the app to completing their goal?"
- "What visual direction are you thinking? (e.g., dark/light, minimal/rich, playful/professional, any reference apps?)"

**If all four elements are present in the initial message, skip questions and proceed immediately to generation.**

## Generation

Delegate to `ui-studio:storyboard-agent` to generate the storyboard. The agent handles:

- Loading the storyboard skill for prompt structure and methodology
- Invoking nano-banana with ALL screens in a single call (never one screen at a time)
- Producing a versioned storyboard image and screen inventory
- Presenting results back for review

```
delegate(
  agent="ui-studio:storyboard-agent",
  instruction="Generate a multi-screen storyboard for the following app brief: [full brief with all four elements]. This is version 1.",
  context_depth="recent",
  context_scope="conversation"
)
```

## Iteration

After the agent returns the storyboard:

1. **Present** — Show the generated storyboard image and screen inventory to the human
2. **Await feedback** — Let the human react (this is a taste judgment, not an automated check)
3. **Refine** — If the human requests changes, delegate back to the storyboard-agent with the specific feedback and the previous version as reference
4. **Repeat** — Continue the generate → present → feedback → refine loop until the human approves

When re-delegating for refinement:
```
delegate(
  agent="ui-studio:storyboard-agent",
  instruction="Refine the storyboard based on this feedback: [human's feedback]. Use the previous version as reference. This is version N.",
  context_depth="recent",
  context_scope="conversation"
)
```

## Handoff

When the human approves the storyboard (signals like "looks good", "that works", "let's move on"):

1. Output the final named screen inventory
2. Suggest the next step with this exact message format:

> **Storyboard complete — [N] screens: [Screen1], [Screen2], ... Which screen do you want to refine first? Type `/frame` and name the screen.**

This transitions the user from storyboard (flow-level) to frame (screen-level) refinement.
