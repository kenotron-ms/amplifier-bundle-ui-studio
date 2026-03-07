---
bundle:
  name: ui-studio
  version: 1.0.0
  description: Guided design-to-code pipeline via four progressive modes

includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
  - bundle: ui-studio:behaviors/ui-studio

provider:
  api: anthropic
  model: claude-opus-4-0-20250514

orchestrator:
  type: loop
---

# UI Studio — Design-to-Code Pipeline

Guided design-to-code system that walks you from rough concept to production-ready implementation through four progressive modes: storyboard, frame, blueprint, and forge.

@ui-studio:context/ui-studio-awareness.md

---

@foundation:context/shared/common-system-base.md
