# Output Structure & Tech Stack Targets — Design

**Date:** 2026-03-06
**Status:** Approved

---

## Problem

Output paths across agents and modes are inconsistent:

- `storyboard-agent` writes to `.amplifier/storyboard/` (wrong — system directory)
- `frame-agent` writes to `frames/{screen-name}/` (CWD-relative, top-level)
- `blueprint-agent` writes to `blueprints/{screen-name}/` (CWD-relative, top-level)
- `forge-agent` writes to `output/` (generic, collides with anything)

The forge target list (HTML, React, Vue, Svelte) doesn't match the intended stack.

---

## Output Folder Structure

All ui-studio artifacts live under a single visible `ui-studio/` root at the project's working directory.

```
ui-studio/
  storyboards/
    storyboard-v1.png
    storyboard-final.png
    screen-inventory.md
  frames/
    {screen-name}/
      v1.png
      v2.png
      approved.png
  blueprints/
    {screen-name}/
      containment-overlay.png
      component-spec.md
      tokens.json
      assets.md
  forge/
    {screen-name}/
      {target}/
        [generated code files]
        current.png        ← latest screenshot
        comparison.png     ← side-by-side diff
        annotated.png      ← nano-banana annotations
```

### Key decisions

- **Visible directory** — `ui-studio/` not `.ui-studio/`. Users should be able to browse outputs naturally.
- **Single root** — all generated artifacts in one place; easy to `.gitignore` if desired.
- **Forge organized by screen → target** — the same screen can be forged to multiple targets independently. Screenshots live inside each target folder because different targets render differently.

---

## Tech Stack Targets

Five supported targets across three categories:

| Category | Target | Frontend Stack | Screenshot mechanism | Token format |
|---|---|---|---|---|
| Web | `react` | React + Tailwind CSS | Playwright | CSS variables + `tailwind.config.js` |
| Web | `svelte` | SvelteKit + Tailwind CSS | Playwright | CSS variables + `tailwind.config.js` |
| Desktop | `electron` | HTML/React frontend + `main.js` | Playwright (built-in Chromium) | CSS variables |
| Desktop | `tauri` | HTML/React frontend + `src-tauri/` scaffold | Playwright (webview) | CSS variables |
| Desktop + Mobile | `flutter` | Dart + Flutter widgets | Flutter SDK (hard requirement) | `ThemeData` / `const Color(0xFF...)` |

### Flutter hard requirement

Flutter is the only target that cannot use Playwright for screenshots. Before attempting any Flutter code generation, the forge-agent **must** verify that `flutter` CLI is available:

```bash
flutter --version
```

If Flutter CLI is not found, fail immediately with a clear message:

```
Flutter CLI not found. Install Flutter SDK before targeting flutter:
  https://docs.flutter.dev/get-started/install
```

Do not proceed to code generation. Do not degrade gracefully. Hard fail.

### Tauri and Electron note

Both use a web frontend (HTML/React + Tailwind). The actual UI code is nearly identical to the `react` target. The difference is the shell scaffolding:
- **Electron**: adds `main.js` + `package.json` with electron dependency
- **Tauri**: adds `src-tauri/` directory with a minimal Rust scaffold and `tauri.conf.json`

Screenshot comparison for both uses Playwright against the web frontend.

---

## .gitignore recommendation

Users who don't want to commit generated artifacts can add:

```
ui-studio/
```

This is not enforced by the bundle — the folder is intentionally visible and commitable.
