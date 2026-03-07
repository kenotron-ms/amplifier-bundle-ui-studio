---
skill:
  name: screenshot-comparison
  version: 1.0.0
  description: Three-stage before/after screenshot comparison with visual annotations for implementation validation
  keywords: [screenshot, comparison, validation, visual-diff, annotations, nano-banana]
  author: Ken
  license: MIT
---

# Screenshot Comparison: Before & After Analysis

A systematic three-stage approach to comparing mockup vs implementation screenshots using visual annotations.

## Overview

This skill captures a proven pattern for validating implementations against mockups:
1. **Create clean side-by-side comparison** (original left, current right)
2. **Generate visual overlay annotations** using Nano Banana Pro
3. **VLM analysis** reads annotations and prioritizes fixes

**Result:** Clear visual markup with prioritized action items.

---

## When to Use This Skill

Use this pattern when:
- ✅ You have an original mockup/design
- ✅ You have a current implementation screenshot
- ✅ You need to identify visual differences systematically
- ✅ You want prioritized fixes based on visual impact

**Don't use for:**
- Initial mockup analysis (use mockup-analyzer instead)
- Text-only changes
- Non-visual comparisons

---

## Prerequisites

**Tools required:**
- ImageMagick (`magick montage` command)
- Nano Banana Pro tool (`tool-nano-banana`)
- Screenshots at same viewport size

**Environment:**
- `GOOGLE_API_KEY` set for Nano Banana Pro

---

## The Three-Stage Process

### Stage 1: Create Side-by-Side Comparison

Use ImageMagick to create a clean left-right comparison:

```bash
cd /path/to/project

# Simple side-by-side
magick montage mockups/original.png screenshots/current.png \
  -geometry +10+10 \
  -background white \
  compare_sxs.png
```

**Output:** `compare_sxs.png` - Original (left) | Current (right)

**Alternative if PIL available:**
```python
from PIL import Image

left = Image.open("mockups/original.png")
right = Image.open("screenshots/current.png")

# Ensure same height
if left.height != right.height:
    aspect = right.width / right.height
    right = right.resize((int(left.height * aspect), left.height))

# Create side-by-side
combined = Image.new('RGB', (left.width + 20 + right.width, left.height), 'white')
combined.paste(left, (0, 0))
combined.paste(right, (left.width + 20, 0))
combined.save("compare_sxs.png")
```

---

### Stage 2: Generate Visual Annotations

Use Nano Banana Pro to add overlay markup:

```bash
amplifier tool invoke nano-banana \
  operation=generate \
  reference_image_path=compare_sxs.png \
  output_path=compare_annotated.png \
  'prompt=TAKE THIS IMAGE showing side-by-side comparison (left=original, right=current) and ADD OVERLAY ANNOTATIONS:

- RED circles/arrows pointing to areas that DO NOT MATCH (with text labels)
- GREEN checkmarks on areas that DO MATCH well
- Yellow warning markers for areas that are CLOSE but not exact
- Text callouts explaining each difference (e.g., "Missing blur", "Wrong icon", "Font too small")
- Percentage indicators showing match quality per region (e.g., "Weather Card: 60% match")
- Draw lines connecting related elements if helpful

Make annotations:
- CLEARLY VISIBLE with semi-transparent overlays where needed
- Bold text with contrasting colors and drop shadows for readability
- Bright saturated RED for problems, GREEN for matches, YELLOW for warnings
- Professional design QA review style

Focus on key visual areas:
- Background and blur effects
- Typography (sizes, weights, colors)
- Icon accuracy
- Spacing and alignment
- Color accuracy
- Border and shadow effects' \
  aspect_ratio=preserve \
  resolution=2K \
  use_thinking=true \
  number_of_images=1
```

**Critical parameter:** `reference_image_path` tells Nano Banana to TAKE the existing image and add annotations to it (not generate from scratch).

**Output:** `compare_annotated.png` - Side-by-side WITH visual markup overlay

---

### Stage 3: VLM Analysis of Annotations

Use VLM to read the annotated image and extract findings:

```bash
amplifier tool invoke nano-banana \
  operation=analyze \
  image_path=compare_annotated.png \
  'prompt=You are analyzing an ANNOTATED comparison image showing:
- LEFT: Original mockup (target design)
- RIGHT: Current implementation
- OVERLAID: Visual annotations (RED=problems, GREEN=matches, YELLOW=warnings)

Read the visual annotations and provide detailed analysis.

Output JSON:
{
  "overall_match_percentage": 70,
  "red_marked_issues": [
    "List each RED-marked problem with its label"
  ],
  "green_marked_matches": [
    "List each GREEN-marked matching element"
  ],
  "yellow_warnings": [
    "List any yellow warning markers"
  ],
  "top_3_priority_fixes": [
    {
      "priority": 1,
      "component": "Component from annotations",
      "problem": "What annotation shows as problem",
      "original_state": "Left side",
      "current_state": "Right side",
      "fix_needed": "Specific actionable fix",
      "impact": "High/Medium/Low"
    }
  ],
  "progress_summary": "What is working well and what needs most attention"
}

Focus on reading the visual annotations to identify marked differences.'
```

**Output:** Structured JSON with prioritized fixes based on visual annotations

---

## Standard Output Location

**Convention:** All comparison artifacts go in `.amplifier/screenshots/`

This keeps your workspace clean and provides a standard location for:
- Screenshot captures
- Side-by-side comparisons
- Annotated analysis images
- Iteration history

```bash
.amplifier/screenshots/
├── iteration_1/
│   ├── screenshot.png      # Captured implementation
│   ├── sxs.png            # Side-by-side comparison
│   └── annotated.png      # With visual annotations
├── iteration_2/
│   └── ...
└── MANIFEST.md            # Summary of results (generated by cleanup)
```

## Complete Example Session

```bash
# Working directory: /Users/ken/workspace/myapp
# Output location: .amplifier/screenshots/

# 1. Create side-by-side
mkdir -p .amplifier/screenshots/iteration_1
magick montage mockups/design-v1.png .amplifier/screenshots/iteration_1/screenshot.png \
  -geometry +10+10 -background white .amplifier/screenshots/iteration_1/sxs.png

# 2. Generate annotations
amplifier tool invoke nano-banana \
  operation=generate \
  reference_image_path=.amplifier/screenshots/iteration_1/sxs.png \
  output_path=.amplifier/screenshots/iteration_1/annotated.png \
  'prompt=TAKE THIS IMAGE and ADD OVERLAY ANNOTATIONS: RED for differences, GREEN for matches...'

# 3. Analyze annotations
amplifier tool invoke nano-banana \
  operation=analyze \
  image_path=.amplifier/screenshots/iteration_1/annotated.png \
  'prompt=Read the annotations and report differences with priorities...'
```

**Result files:**
- `.amplifier/screenshots/iteration_1/sxs.png` - Clean side-by-side
- `.amplifier/screenshots/iteration_1/annotated.png` - With visual markup
- JSON output with prioritized fixes

---

## Output Structure

### Stage 2 Output (Generate)
```json
{
  "generated_images": ["compare_annotated.png"],
  "count": 1
}
```

### Stage 3 Output (Analyze)
```json
{
  "overall_match_percentage": 72,
  "red_marked_issues": [
    "Missing blur effect on weather card",
    "Wrong icon (drop vs. rain cloud)",
    "Missing daily forecast details"
  ],
  "green_marked_matches": [
    "Status bar time matches",
    "Bottom card layout alignment correct"
  ],
  "yellow_warnings": [
    "Font weight slightly too bold",
    "Extra navigation bar present"
  ],
  "top_3_priority_fixes": [
    {
      "priority": 1,
      "component": "Weather Card Background",
      "problem": "Missing glassmorphism blur",
      "original_state": "Semi-transparent with backdrop blur",
      "current_state": "Solid opaque background",
      "fix_needed": "Add backdrop-filter: blur(10px) and reduce opacity",
      "impact": "High"
    }
  ],
  "progress_summary": "Basic structure matches (70%), but core styling missing..."
}
```

---

## Integration with Iteration Loops

This skill fits into the validation workflow:

```
1. mockup-analyzer → Generate blueprint (0-50% match)
2. implementation → Build components (50-70% match)
3. screenshot-comparison → Identify gaps (THIS SKILL)
4. Apply Priority 1 fix
5. Re-run screenshot-comparison → Measure improvement
6. Repeat until >95% match
```

**Convergence pattern:**
- Iteration 1: 60% → Fix background blur → 72%
- Iteration 2: 72% → Fix icons → 80%
- Iteration 3: 80% → Fix typography → 88%
- Iteration 4: 88% → Fine-tune spacing → 94%

### Automated Loop Recipe

For automated convergence, use the `screenshot-comparison-loop` recipe:

```bash
amplifier tool invoke recipes operation=execute \
  recipe_path=@ui-studio:recipes/screenshot-comparison-loop.yaml \
  'context={
    "original_mockup_path": "mockups/design.png",
    "implementation_url": "http://localhost:5173",
    "viewport_width": "390",
    "viewport_height": "844",
    "target_match_percentage": "95"
  }'
```

The recipe runs this three-stage pattern in a loop:
1. Captures screenshot
2. Generates annotated comparison
3. Analyzes and reports top priority fix
4. **Pauses** for you to apply the fix
5. Press ENTER to continue to next iteration
6. Repeats until match ≥ target or max iterations

See `@ui-studio:recipes/screenshot-comparison-loop.yaml` for full details.

---

## Critical Success Factors

### 1. Same Viewport Size
```
Original mockup: 390x844
Screenshot MUST be: 390x844

Different sizes = invalid comparison!
```

Use browser DevTools device mode or Playwright to match dimensions.

### 2. Reference Image Path (Stage 2)
```bash
# ✓ CORRECT - Takes existing image and annotates it
reference_image_path=compare_sxs.png

# ✗ WRONG - Would generate new image from scratch
reference_image=["original.png", "current.png"]
```

The `reference_image_path` parameter is critical for overlay annotations.

### 3. Clear Annotation Prompt
Be specific about what to mark:
- RED for mismatches
- GREEN for correct elements
- YELLOW for close-but-not-exact
- Text labels explaining each issue
- Percentage indicators per region

---

## Troubleshooting

### Issue: Generated image doesn't look annotated

**Cause:** Used `reference_image` instead of `reference_image_path`

**Fix:**
```bash
# Use this parameter specifically
reference_image_path=compare_sxs.png
```

### Issue: Annotations not visible

**Cause:** Prompt didn't specify visibility requirements

**Fix:** Add to prompt:
```
Make annotations CLEARLY VISIBLE with:
- Semi-transparent overlays where needed
- Bold text with drop shadows
- Bright saturated colors (RED, GREEN, YELLOW)
```

### Issue: VLM can't read annotations

**Cause:** Annotations too subtle or wrong colors

**Fix:** Regenerate Stage 2 with more explicit contrast requirements

---

## Advanced Patterns

### Multiple Comparison Views

Create different comparison types:

```bash
# Side-by-side
magick montage original.png current.png -geometry +10+10 compare_sxs.png

# Overlay blend
magick original.png current.png -compose blend -define compose:args=50 compare_overlay.png

# Difference heatmap
magick compare original.png current.png -compose difference compare_diff.png
```

Annotate each view separately for different insights.

### Region-Specific Analysis

Focus annotations on specific areas:

```bash
# Crop to weather card region first
magick compare_sxs.png -crop 800x400+0+100 weather_card_compare.png

# Then annotate just that region
amplifier tool invoke nano-banana \
  operation=generate \
  reference_image_path=weather_card_compare.png \
  output_path=weather_card_annotated.png \
  'prompt=Focus annotations on this weather card only...'
```

## Cleanup Strategy

After convergence, the workflow automatically cleans up intermediate files:

**What's kept:**
- ✅ `iteration_1/` - Baseline (first state)
- ✅ `iteration_N/` - Final state (where N is last iteration)
- ✅ All `annotated.png` files - Visual record of each iteration

**What's removed:**
- 🗑️ Intermediate `screenshot.png` files (iterations 2 to N-1)
- 🗑️ Intermediate `sxs.png` files (iterations 2 to N-1)

**Result:** Preserved visual history with minimal disk usage.

The cleanup step generates a `MANIFEST.md` file documenting:
- Date and final match percentage
- Total iterations run
- Files preserved
- Cleanup actions taken

This keeps your workspace clean while maintaining a complete audit trail.

---

## Related Skills

- **vlm-iteration** - Complete iteration loop pattern with convergence tracking
- **detail-refinement** - Fine-tuning after reaching 80% match
- **font-matching** - Systematic font identification
- **icon-finding** - Icon asset selection

---

## Summary

**Three stages:**
1. Create clean side-by-side comparison (ImageMagick)
2. Generate visual overlay annotations (Nano Banana Pro with `reference_image_path`)
3. VLM reads annotations and prioritizes fixes

**Key insight:** Visual annotations make differences obvious and enable systematic prioritization.

**Use in iteration loops:** Compare → Annotate → Analyze → Fix #1 → Repeat until >95% match.
