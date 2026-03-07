---
skill:
  name: font-matching
  version: 1.0.0
  description: Systematic Google Fonts search using VLM validation across 1,600+ font families
  keywords: [fonts, typography, google-fonts, vlm, mockup, design-tokens]
  author: Ken
  license: MIT
---

# Font Matching Skill: VLM-Guided Google Fonts Selection

## The Problem

When implementing mockups, designers may use:
- Commercial fonts (not available for web)
- System fonts (inconsistent across platforms)
- Unlicensed fonts (can't use in production)
- Unknown fonts (can't identify by eye)

**We need:** Find the closest Google Fonts match for production use.

**Challenge:** We can't just ask "what font is this?" because:
- VLM might hallucinate font names
- Original font might not be on Google Fonts
- Need to match visually, not by name

---

## Critical Discovery: Two-Font Typography Systems

**Most mockups use a font pairing:**
- **Display/Title font** - Serif or distinctive sans-serif (headers, titles)
- **Body/UI font** - Clean sans-serif (body text, metadata, labels)

**This is a graphics design convention!**

**Before searching for fonts, identify:**
1. How many fonts are in use? (usually 2)
2. Which text uses which font?
3. What weights of each font are needed?

**Our test case revealed:**
- Serif (Playfair Display): Main title (700), tabs (400), hero title (700), article titles (600)
- Sans-serif (Roboto): Hero metadata (400), descriptions (400), pills (400), nav labels (500)

**You must search for BOTH fonts, not just one!**

---

## The Font Matching Process (VLM-Validated)

### Step 1: VLM Describes Font Characteristics

**Don't ask:** "What font is this?"  
**Do ask:** "Describe what you SEE in typography terms"

**Prompt:**
```
Analyze the typography in this mockup.

For the main title "Meditations":
- Serif or sans-serif?
- Thick or thin strokes?
- High or low contrast (thick/thin variation)?
- Wide or narrow letter spacing?
- Geometric or humanist?
- Elegant, modern, classic, playful?

Describe the VISUAL characteristics, not guessed font names.
```

**Example VLM Response:**
```
Main title "Meditations":
- Serif font
- High stroke contrast (thick verticals, thin horizontals)
- Elegant, refined appearance
- Classic style with modern sensibility
- Medium letter spacing
```

---

### Step 2: Query Google Fonts API

**Get all fonts matching characteristics:**

```bash
# Fetch Google Fonts metadata
curl "https://www.googleapis.com/webfonts/v1/webfonts?key=YOUR_API_KEY" > fonts.json

# Filter by category
jq '.items[] | select(.category == "serif")' fonts.json
```

**Categories available:**
- `serif` - Traditional serifs
- `sans-serif` - Clean, modern
- `display` - Decorative, headlines only
- `handwriting` - Script, cursive
- `monospace` - Code, fixed-width

**For our case:** Filter to `serif` for title font, `sans-serif` for body font.

---

### Step 3: Create Batch Comparison Pages

**Don't compare fonts one-by-one - batch them!**

**Create HTML comparison page:**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Serif Font Comparison - Batch 1</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <!-- Include 20 fonts in one batch -->
  <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700&family=Libre+Baskerville:wght@400;700&..." rel="stylesheet">
</head>
<body>
  <h1>Original Mockup Reference</h1>
  <img src="mockup-title-cropped.png" width="400">
  
  <h1>Font Candidates (Batch 1 of 10)</h1>
  
  <div class="font-sample" style="font-family: 'Playfair Display'">
    <h2>Meditations</h2>
    <p>Font: Playfair Display</p>
  </div>
  
  <div class="font-sample" style="font-family: 'Libre Baskerville'">
    <h2>Meditations</h2>
    <p>Font: Libre Baskerville</p>
  </div>
  
  <!-- ... 18 more fonts -->
</body>
</html>
```

**Why batches?**
- Compare 20 fonts at once (fast visual scanning)
- Create ~10 batches to cover all serif fonts
- VLM picks top 2-3 from each batch
- Final round: compare only the finalists

---

### Step 4: VLM Ranks Candidates in Each Batch

**Prompt for each batch:**
```
Compare these 20 serif fonts to the original mockup title.

Original: [cropped image of "Meditations" title]
Candidates: [screenshot of batch comparison page]

Pick the TOP 2 that best match:
- Stroke contrast (thick/thin variation)
- Letter proportions
- Serif style (bracketed, slab, hairline)
- Overall elegance

Output: Font names of top 2 matches
```

**Example VLM Response:**
```
Batch 1 top matches:
1. Playfair Display - High contrast, elegant serifs, modern refinement
2. Libre Baskerville - Good contrast, classic proportions

(Reject other 18 as not matching closely enough)
```

**Repeat for all batches** → Collect ~20 finalists from 10 batches

---

### Step 5: Finals Comparison

**Create one page with all finalists:**

```html
<h1>Finals - Top 20 Serif Fonts</h1>
<img src="mockup-title-cropped.png" width="600">

<div class="font-sample" style="font-family: 'Playfair Display'; font-weight: 700; font-size: 48px;">
  Meditations
</div>
<div class="font-sample" style="font-family: 'Libre Baskerville'; font-weight: 700; font-size: 48px;">
  Meditations
</div>
<!-- ... all 20 finalists -->
```

**VLM ranks top 3:**
```
1. Playfair Display - Best match (95% confidence)
2. Libre Baskerville - Close second (85%)
3. Cormorant Garamond - Third (75%)
```

---

### Step 6: Weight Verification

**Check if the winning font has needed weights:**

```
Font: Playfair Display
Weights needed: 400 (tabs), 600 (article titles), 700 (main title)

Google Fonts Playfair Display weights available:
- 400, 500, 600, 700, 800, 900 ✓

All weights available! ✓
```

**If missing weights:**
- Try the second-place font
- Or use closest available weight (600 → 700)

---

### Step 7: Fallback Chain

**Define complete font stack:**

```css
font-family: 'Playfair Display', Georgia, 'Times New Roman', serif;
```

**Why fallbacks matter:**
- Google Fonts might fail to load
- User might have network issues
- Accessibility/privacy mode might block web fonts

**Good fallback choices:**
- Serif titles → Georgia (macOS/Windows), Times New Roman
- Sans-serif body → system-ui, -apple-system, Roboto, Arial

---

## The Complete Font Search Workflow

```
1. VLM analyzes mockup typography
   → Identifies 2 fonts (display + body)
   → Lists characteristics for each
   
2. Query Google Fonts API
   → Filter by category (serif, sans-serif)
   → Get all ~300 serif + ~800 sans-serif fonts
   
3. Create batch comparison pages
   → 20 fonts per page
   → Display mockup text in each font
   
4. VLM picks top 2-3 from each batch
   → Compare visually to original
   → Narrow from 300 → ~20 finalists
   
5. Finals comparison page
   → All finalists side-by-side
   → VLM ranks top 3
   
6. Verify weights available
   → Check if winner has all needed weights
   
7. Define fallback chain
   → System fonts as backups
```

---

## Key Insights from Our Session

### 1. **TWO fonts, not one**
Don't search for "the font" - search for the display font AND the body font separately.

### 2. **Search ALL fonts, not just famous ones**
- Don't assume "it's probably Georgia or Times"
- Systematically check all 1,600+ Google Fonts
- The right font might be obscure (Playfair Display wasn't our first guess!)

### 3. **Batching is 10x faster**
- Comparing 20 fonts at once vs 1 at a time
- 10 batches cover all fonts in ~30 minutes
- One-by-one would take hours

### 4. **VLM is better at ranking than naming**
- Can accurately rank visual similarity
- Might hallucinate font names if asked directly
- Use for "which of these 20 is closest?" not "what font is this?"

### 5. **Weight availability matters**
If title needs weight 700 and body needs 400/500, both fonts must support those weights.

---

## Handling Edge Cases

### What if no good match exists?

**Option 1: Use the closest match**
- VLM picks best of what's available
- Might be 70% match instead of 95%
- Document the difference

**Option 2: Use a system font**
- Georgia (serif) or Arial (sans-serif)
- Reliable, no loading time
- Won't match exactly but universally available

**Option 3: Purchase/license the original font**
- If exact match is critical
- Check font licensing for web use

### What if VLM picks different fonts each time?

**This means the fonts are very similar!**
- Use VLM confidence scores
- Pick the one with highest availability (more weights)
- Or ask user to make final call

### What if mockup uses 3+ fonts?

**Still follow the same process:**
1. Identify all distinct fonts
2. Search for each separately
3. More fonts = more complexity (avoid if possible)

---

## Typography System Documentation

**After finding fonts, document:**

```yaml
typography:
  display:
    family: "Playfair Display"
    fallback: "Georgia, 'Times New Roman', serif"
    weights: [400, 600, 700]
    usage:
      - Main screen title (700)
      - Hero card title (700)
      - Article titles (600)
      - Tab labels (400)
  
  body:
    family: "Roboto"
    fallback: "system-ui, -apple-system, Arial, sans-serif"
    weights: [400, 500]
    usage:
      - Body text (400)
      - Metadata (400)
      - Labels (400)
      - Navigation (500)
```

---

## Implementation (Tailwind Config)

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      fontFamily: {
        display: ['Playfair Display', 'Georgia', 'Times New Roman', 'serif'],
        body: ['Roboto', 'system-ui', '-apple-system', 'Arial', 'sans-serif'],
      },
      fontWeight: {
        // Display weights
        'display-regular': 400,
        'display-semibold': 600,
        'display-bold': 700,
        // Body weights
        'body-regular': 400,
        'body-medium': 500,
      }
    }
  }
}
```

**Usage in components:**
```jsx
<h1 className="font-display font-display-bold text-4xl">
  Meditations
</h1>

<p className="font-body font-body-regular text-base">
  Description text
</p>
```

---

## Success Metrics

✓ **Both display and body fonts identified**  
✓ **Searched all relevant fonts (not just famous ones)**  
✓ **VLM confidence >90% for winner**  
✓ **All needed weights available**  
✓ **Fallback chain defined**  
✓ **User confirms visual match**

---

## Summary

**The systematic approach:**
1. Identify 2-font system (display + body)
2. Query Google Fonts API (1,600+ fonts)
3. Batch comparisons (20 fonts per page)
4. VLM narrows 300 → 20 finalists
5. Finals round picks winner
6. Verify weights, define fallbacks

**Why this works:**
- Comprehensive (checks ALL fonts)
- Efficient (batch comparisons)
- Accurate (VLM visual ranking)
- Reliable (weight verification)

**Critical insight:** Font matching is a SEARCH problem, not a recognition problem. Don't ask VLM "what font?" - show it options and ask "which matches best?"
