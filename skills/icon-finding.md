---
skill:
  name: icon-finding
  version: 1.0.0
  description: VLM-guided icon selection with topology awareness. NEVER use emoji for UI icons.
  keywords: [icons, svg, emoji, topology, nano-banana, mockup]
  author: Ken
  license: MIT
---

# Icon Finding Skill: VLM-Guided Icon Selection

## The Problem

When implementing UI mockups, we need icons for navigation, actions, and UI elements.

**NEVER use:**
- ❌ Emoji (🧭🧘🎙️) - Tacky, unprofessional, inconsistent across platforms
- ❌ Generated 1K images (1024x1024, 500KB+) - Wasteful, slow, unnecessary

**ALWAYS use:**
- ✅ SVG icons from established libraries (<1KB, scalable, professional)
- ✅ Generate only if no suitable icon exists in libraries

---

## The Icon Finding Process

### Step 1: Use VLM to Describe What Icon LOOKS Like (Not What It Means)

**Prompt Nano Banana Pro:**
```
Look at the bottom navigation in this mockup.

For each icon, describe what you ACTUALLY SEE (mechanical description, not concept):
- What shapes make up the icon?
- How are the shapes arranged?
- What does it visually look like?

Example: "A circle with a dot in the center" NOT "a target icon"
```

**Example VLM Response:**
```
Icon 1: "A cluster of five small interconnected circles - one central, four radiating"
Icon 2: "An open book shape - larger rectangle with smaller offset rectangle"
Icon 3: "Vertical line with horizontal bars at top and bottom, curved top"
```

---

### Step 2: Map Visual Description to Icon Concepts

| VLM Visual Description | Likely Concept | Icon Library Name |
|------------------------|----------------|-------------------|
| "5 interconnected circles" | Network/Share | ⚠️ WRONG - see semantic check below |
| "Open book shape" | Book | `book-open` ✓ |
| "Vertical line with bars and curve" | Microphone | `microphone` or `mic` ✓ |

---

### ⚠️ CRITICAL: Semantic Meaning Check (Context-Dependent!)

**Before using an icon, check BOTH:**
1. **Generic UI meaning** (what it usually means)
2. **App-specific context** (what it means in THIS app)

**Example: The Network/Share Icon - Context Matters!**

```
VLM says: "5 interconnected circles" (network visualization)
Visual match: share icon from Heroicons

Generic meaning: Social sharing, export to other apps
App-specific context: "Explore" section shows a GRAPH OF NODES

✓ CORRECT to use share/network icon here!
Why? Icon is a VISUAL PREVIEW of what users will see when they tap it.
```

**The deeper insight:** Icons can be:
- **Generic symbols** (compass = explore anywhere)
- **Visual previews** (network icon = you'll see a network graph)

**In this meditation app:**
- Network icon for "Explore" = ✓ Shows node graph visualization
- Book icon for "Meditations" = ✓ Shows reading/text content
- Microphone icon for "Voice" = ✓ Shows audio feature

**The icon choice teaches users what to expect!**

---

### The Icon Topology Rule

**Icons don't just represent concepts - they represent STRUCTURE:**

#### Share Icon (Hub-and-Spoke Topology):
```
     ●
    /|\\
   ● ● ●

ONE-to-MANY relationship
Meaning: Broadcasting, distributing FROM one source TO many targets
Use cases: 
  - Social sharing (post to Twitter, Facebook, etc.)
  - Export (send file to multiple destinations)
  - Broadcast messages
```

#### Network/Graph Icon (Mesh Topology):
```
   ●---●
   |\ /|
   | X |
   |/ \|
   ●---●

MANY-to-MANY relationship  
Meaning: Interconnected peers, exploration of relationships
Use cases:
  - Knowledge graphs (concepts connected to concepts)
  - Social networks (people connected to people)
  - Exploration features (navigate between related items)
  - Network topology visualization
```

**Why This Matters:**

The designer chose the network/graph icon for "Explore" because:
1. **Visual preview**: Icon SHOWS the graph structure users will see
2. **Structural match**: Many-to-many topology matches the feature
3. **Semantic accuracy**: Exploration = navigating connections, not broadcasting

**Using share icon would be WRONG** even though it looks similar:
- Visually: Both have circles and connections ✓
- Topology: Share = 1→many, Graph = many↔many ✗
- Meaning: Share = export/broadcast, Graph = explore relationships ✗

---

### The Icon Selection Hierarchy (UPDATED)

```
1. Respect original mockup choice (designer chose for DEEP reasons)
   ↓
2. Use VLM to describe visual appearance
   ↓
3. Identify the TOPOLOGY (one-to-many? many-to-many? hierarchical? mesh?)
   ↓
4. Search libraries matching BOTH visual AND topological structure
   ↓
5. Semantic meaning check (does it match context + structure?)
   ↓
6. If no match found → Generate with VLM's visual + topological description
```

**The critical question:** "What STRUCTURE does this icon represent?"

Not just "what concept" but "what relationship model, what topology, what data structure?"

---

### The Three-Level Semantic Check

**Level 1: Generic UI Convention**
"What does this icon usually mean?"
- Share = social export
- Compass = navigation/explore
- Globe = language/international

**Level 2: App-Specific Context**
"What does this icon mean in THIS app?"
- Share/network = explore section with node graph visualization
- Book = meditation reading library
- Microphone = voice meditation feature

**Level 3: Visual Preview**
"Does the icon SHOW what users will see?"
- Network icon → User taps → Sees network graph ✓ Perfect match!
- Book icon → User taps → Sees text/reading ✓ Makes sense!
- Microphone icon → User taps → Sees audio controls ✓ Clear!

**USE APP CONTEXT, NOT JUST GENERIC CONVENTIONS!**

---

### Step 2b: Semantic Validation (NEW STEP)

After finding a visual match, ask:

**"What does this icon MEAN to users?"**

Check against established patterns:
- Share icon → Social sharing, export
- Compass icon → Explore, navigate, discover
- Globe icon → Language, worldwide, international
- Book icon → Reading, library, documentation
- Magnifying glass → Search (specific query)
- Map pin → Location, place
- Heart → Favorite, like
- Star → Rating, bookmark
- Bell → Notifications, alerts

**If semantic meaning doesn't match context → keep searching or generate!**

---

### Updated Mapping (With Semantic Check):

| VLM Visual Description | First Instinct | ❌ Semantic Issue | ✓ Correct Icon |
|------------------------|----------------|-------------------|----------------|
| "5 interconnected circles" | share icon | Share = social export | **compass** (explore/discover) |
| "Open book shape" | book-open | ✓ Matches reading | **book-open** |
| "Vertical line with bars" | microphone | ✓ Matches voice | **microphone** |

---

### Step 3: Search Popular Icon Libraries

**Recommended libraries (in order):**

1. **Heroicons** (https://heroicons.com/)
   - Made by Tailwind creators
   - Clean, minimal, consistent
   - MIT licensed
   - Available: 24x24 outline, 24x24 solid, 20x20 solid, 16x16 micro

2. **Feather Icons** (https://feathericons..com/)
   - Ultra-minimal stroke icons
   - Consistent 24x24 grid
   - MIT licensed

3. **Lucide** (https://lucide.dev/)
   - Fork of Feather with more icons
   - Same minimal style
   - ISC licensed

4. **Phosphor Icons** (https://phosphoricons.com/)
   - Multiple weights (thin, light, regular, bold, fill)
   - Consistent style
   - MIT licensed

---

### Step 4: Fetch SVG from Library

**Method: Direct from GitHub (Heroicons example)**

```bash
# Outline 24x24 version
https://raw.githubusercontent.com/tailwindlabs/heroicons/master/src/24/outline/ICON-NAME.svg

# Examples:
https://raw.githubusercontent.com/tailwindlabs/heroicons/master/src/24/outline/book-open.svg
https://raw.githubusercontent.com/tailwindlabs/heroicons/master/src/24/outline/microphone.svg
https://raw.githubusercontent.com/tailwindlabs/heroicons/master/src/24/outline/share.svg
```

---

### Step 5: Integrate SVG into HTML

**Inline SVG (recommended for small icon counts):**
```html
<button class="nav-item">
  <svg class="nav-icon" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
    <path d="..." stroke-linecap="round" stroke-linejoin="round"/>
  </svg>
  <span class="nav-label">Explore</span>
</button>
```

**CSS for styling:**
```css
.nav-icon {
  width: 24px;
  height: 24px;
  color: #6C6C6C;  /* stroke="currentColor" uses this */
}

.nav-item.active .nav-icon {
  color: #C66E5B;  /* Active state */
}
```

**Benefits:**
- `stroke="currentColor"` inherits CSS color
- One icon, multiple color states via CSS
- No additional HTTP requests
- Tiny file size (<1KB per icon)

---

### Step 6: Fallback - Generate Only If Not Found

If no library has the right icon:

**Use VLM's mechanical description in generation prompt:**
```python
# Instead of: "Generate a meditation icon"
# Use: "Generate an icon showing [VLM's description]"

result = await generator.generate(
    prompt="""
    Create a minimal 24x24 line icon:
    Visual: [VLM's mechanical description]
    Style: Single-color stroke, no fill, 1.5px stroke width
    Format: Simple, clean, matches Heroicons aesthetic
    """,
    params={
        "resolution": "minimal",  # NOT "1K"!
    }
)
```

---

## When to Use Each Approach

| Scenario | Approach | Reasoning |
|----------|----------|-----------|
| Standard UI icon with established meaning | **Library** first | Semantic meaning is critical |
| Icon matches visually BUT wrong semantic meaning | **Keep searching** or **Generate** | Never compromise on semantics |
| No semantic conflict, visual match found | **Use library** | Fast, professional, tiny |
| Custom brand icon or specialized symbol | **Generate** with VLM | Brand-specific needs |
| Icon carries cultural/UX baggage | **Be extra careful** | Wrong icon = confused users |

---

## The Semantic Meaning Rule

**CRITICAL INSIGHT:** Visual similarity ≠ Correct choice

Even if an icon *looks* like what the VLM described, check:
1. **What does this icon MEAN to users?**
2. **Does that meaning match the context?**
3. **Is there established UI convention around this icon?**

**Real Example from This Session:**

```
Context: "Explore" navigation in meditation app
VLM described: "5 interconnected circles"
Visual match: share icon (looks similar!)

But semantic check reveals:
- Share icon = Social sharing (Twitter, Facebook, send to others)
- Context needs: Exploration, discovery, navigation

Correct icon: compass
- Compass = Explore, discover, navigate
- Matches both visual (circle with directional element) AND semantic meaning
```

**The lesson:** If you use "share" for "explore", users will be confused even though it visually matches the VLM description.

---

## Icon Semantic Meanings (Quick Reference)

### Navigation & Wayfinding
- **Compass** → Explore, discover, navigate
- **Map** → Location, places, geography
- **Globe** → Language selection, worldwide, international
- **Home** → Home screen, main page
- **Arrow** → Direction, navigation, back/forward

### Actions & Interaction
- **Share** → Social sharing, export, send to others (⚠️ Very specific meaning!)
- **Plus/Add** → Create new, add item
- **Heart** → Favorite, like, love
- **Star** → Rating, bookmark, featured
- **Bell** → Notifications, alerts
- **Trash** → Delete, remove

### Content & Communication
- **Book** → Reading, library, documentation
- **Message/Chat** → Messaging, conversations
- **Mail** → Email, messages
- **Microphone** → Voice, audio recording, speech

### System & Tools
- **Search/Magnifying glass** → Search (specific query), zoom
- **Settings/Gear** → Configuration, preferences
- **Filter** → Filter data, refine results
- **Download/Upload** → Transfer files

**When in doubt:** Search for "[icon name] meaning in UI design" to verify established conventions!

---

## The Emoji Rule

**NEVER use emoji for UI elements:**
- Navigation icons ❌
- Action buttons ❌
- Status indicators ❌
- Form controls ❌

**ONLY use emoji for:**
- Colloquial text content ✅
- Reactions/emotions in messaging ✅
- User-generated content ✅

**Why:**
- Emoji are inconsistent across platforms
- Emoji feel unprofessional in UI chrome
- Emoji don't support color styling (can't match brand)
- Emoji scale poorly and have accessibility issues

---

## Our Success Case

**Original approach:**
- Asked Nano Banana to generate icons
- Got 1024x1024 JPEG images (572KB each!)
- Wrong format, wrong size, wasteful

**Improved approach:**
1. VLM described actual icons in mockup
2. Mapped to Heroicons names (share, book-open, microphone)
3. Fetched SVG from GitHub (<1KB each)
4. Inline in HTML with `stroke="currentColor"`
5. CSS controls color for active/inactive states

**Result:**
- Professional, scalable, tiny file size
- Color-controllable via CSS
- Instant loading, no generation time
- Matches the clean, minimal aesthetic

---

## File Size Comparison

| Method | File Size | Load Time | Scalability | Color Control |
|--------|-----------|-----------|-------------|---------------|
| Emoji | N/A | Instant | Poor | None |
| Generated PNG (1K) | 500KB+ | Slow | Pixelated | None |
| Library SVG | <1KB | Instant | Perfect | CSS |

**Winner:** Library SVG by a mile!

---

## The Complete Icon Finding Workflow

```
Input: UI mockup with icons

1. VLM Analysis
   → Describe visual appearance of each icon
   → Mechanical description, not conceptual

2. Concept Mapping  
   → Match visual description to icon concept
   → "5 circles" → share/network

3. Library Search
   → Check Heroicons, Feather, Lucide, Phosphor
   → Search by concept name

4. Fetch SVG
   → Get from library GitHub
   → Verify it matches the visual description

5. Integrate
   → Inline SVG in HTML
   → Use stroke="currentColor" for CSS styling
   → Test active/inactive states

6. Fallback (rare)
   → If no library match, generate small icon
   → Use VLM's visual description in prompt
   → Request SVG format if possible
```

---

## Summary

**The Golden Rule:** NEVER use emoji for UI icons. Always search libraries first.

**The Process:**
1. VLM describes topology
2. Identify semantics from context
3. Search Heroicons → Feather → Lucide → Phosphor
4. Verify both topology and semantic match
5. Generate only if libraries have nothing suitable

**Critical insight:** Icon selection requires BOTH visual matching (topology) AND conceptual understanding (semantics). VLMs handle topology; you handle semantics.
