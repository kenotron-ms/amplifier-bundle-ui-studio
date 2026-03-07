# UI Studio

Guided design-to-code pipeline via four progressive modes.

## Modes

| Mode | Command | Description |
|------|---------|-------------|
| Storyboard | `/storyboard` | Explore concepts and define screen flow narratives |
| Frame | `/frame` | Compose wireframe structure and spatial layout |
| Blueprint | `/blueprint` | Generate precise visual specifications with design tokens |
| Forge | `/forge` | Produce production code with screenshot-driven validation |

## Installation

```bash
amplifier run --bundle git+https://github.com/kenotron-ms/amplifier-bundle-ui-studio@main
```

## Quick Usage

Progress through modes sequentially — each builds on the previous output:

```
/storyboard  →  describe your concept, get a screen flow narrative
/frame       →  turn the narrative into wireframe layouts
/blueprint   →  extract precise visual specs and design tokens
/forge       →  generate code, validate with screenshots, iterate
```

## Dependencies

| Dependency | Purpose |
|------------|---------|
| [tool-nano-banana](https://github.com/kenotron-ms/amplifier-module-tool-nano-banana) | VLM-powered screenshot capture and visual analysis |
| Playwright | Browser automation for screenshot capture |
| ImageMagick | Image processing for visual diff comparison |
| `GOOGLE_API_KEY` | Access to Google Fonts API for font matching |

## Composable Usage

Other bundles can include UI Studio as a behavior layer:

```yaml
includes:
  - bundle: git+https://github.com/kenotron-ms/amplifier-bundle-ui-studio@main
    behavior: ui-studio
```
