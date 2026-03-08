---
mode:
  name: moodboard
  description: Collect design inspirations and synthesize a visual collage and aesthetic brief
  shortcut: moodboard
  tools:
    safe:
      - read_file
      - glob
      - grep
      - load_skill
      - delegate
      - nano-banana
      - write_file
      - edit_file
      - bash
  default_action: block
---

MOODBOARD MODE: Collect design references — screenshots, images, text snippets, URLs — and synthesize them into a visual collage and a structured aesthetic brief that feeds directly into `/storyboard`.

The loop is: drop items → agent saves and acknowledges → add more → say "synthesize" when ready → collage + brief generated → review → refine or done.

## On Entry

Immediately delegate to the moodboard agent. The agent will:

1. Scan `ui-studio/moodboard/items/` for any pre-dropped files
2. Report what it found (or prompt for first input if the folder is empty)
3. Enter the collection loop

```
delegate(
  agent="ui-studio:moodboard-agent",
  instruction="Enter moodboard collection mode. Scan ui-studio/moodboard/items/ for pre-dropped files, report what you find, and then invite the user to add more items. Accept text, image paths, URLs, and file references.",
  context_depth="recent",
  context_scope="conversation"
)
```

## Accepting Items

Each item the user provides — in conversation or pre-dropped — gets saved to `ui-studio/moodboard/items/` with a sequential number prefix:

```
ui-studio/moodboard/items/
├── 001-screenshot.png
├── 002-style-note.md
├── 003-reference-url.md
└── ...
```

The agent acknowledges each item and asks if more are coming or if synthesis should begin.

## Synthesis Trigger

When the user says "synthesize", "done", "that's enough", or similar — delegate the synthesis step:

```
delegate(
  agent="ui-studio:moodboard-agent",
  instruction="Synthesize the moodboard. Generate the visual collage and aesthetic brief from all items in ui-studio/moodboard/items/.",
  context_depth="recent",
  context_scope="conversation"
)
```

Minimum items required before synthesis: 2. If only 1 item exists, ask for at least one more reference.

## Outputs

After synthesis, the agent produces:

- `ui-studio/moodboard/collage.png` — visual reference image (used as nano-banana input for storyboard)
- `ui-studio/moodboard/aesthetic-brief.md` — structured aesthetic brief (auto-loaded by `/storyboard`)

## Handoff

When the brief is approved, suggest the next step:

> **Moodboard complete. Your aesthetic brief and visual collage are ready. Type `/storyboard` — the brief will be loaded automatically as your aesthetic direction.**
