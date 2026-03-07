---
skill:
  name: detail-refinement
  version: 1.0.0
  description: Fine-tuning with verification loops and contradiction detection. Prevents going backwards through VLM validation.
  keywords: [refinement, validation, vlm, contradiction-detection, verification]
  author: Ken
  license: MIT
---

# Detail Refinement: VLM Verification with Contradiction Detection

## The Critical Problem

**VLMs can contradict themselves when comparing images.**

When you ask "what's different between original and implementation?", the VLM might give suggestions that **contradict the actual mockup**.

**Real example from our session:**
```
VLM Checklist: "Make hero card narrower than list cards"
Applied the change ❌
User: "You went backwards"
VLM (looking at original only): "Hero is WIDER than list cards"
```

**The mistake:** Applied comparative analysis without verifying against ground truth.

---

## The Solution: Verification Loop with Contradiction Detection

### ⚠️ NEVER Apply VLM Suggestions Without Verification

**Wrong process:**
```
VLM says: "Make hero narrower"
  ↓
Apply the change
  ↓
Result: Worse than before ❌
```

**Correct process:**
```
VLM says: "Make hero narrower"
  ↓
ASK VLM: "Look at ORIGINAL ONLY - is hero wider or narrower than list cards?"
  ↓
VLM: "Hero is wider"
  ↓
CONTRADICTION DETECTED - Don't apply!
  ↓
Present both options to USER for decision
```

---

## The Detailed Refinement Process

### Step 1: Comparative Checklist

**Generate issues list by comparing implementation to original:**

```
Prompt VLM with BOTH images:
"Compare the original mockup (left) to the current implementation (right).

List differences in these categories:
1. Layout & spacing
2. Typography (sizes, weights, fonts)
3. Colors & backgrounds
4. Component sizing
5. Visual hierarchy

For each difference:
- What you see in original
- What you see in implementation
- Specific fix needed

Be precise with measurements where possible."
```

**Output: Checklist of potential issues**

---

### Step 2: Verification Against Original (CRITICAL)

**For EACH issue in the checklist, verify against original ONLY:**

```
Prompt VLM with ORIGINAL ONLY:
"Look at this original mockup.

Issue to verify: 'Hero card appears narrower than list cards'

Measure:
- Hero card margin from screen edges
- List card margins from screen edges

Which is wider? Provide pixel measurements if possible."
```

**Output: Ground truth measurement**

---

### Step 3: Contradiction Detection

**Compare checklist suggestion with verification:**

```python
# Pseudo-code
checklist_says = "Make hero narrower"
verification_says = "Hero has 20px margins, list has 30px margins → Hero is wider"

if checklist_says != verification_says:
    # CONTRADICTION DETECTED
    flag_for_human_approval(
        checklist=checklist_says,
        verification=verification_says,
        current_state=current_implementation
    )
else:
    # Safe to apply
    apply_fix(checklist_says)
```

---

### Step 4: Human Approval for Contradictions

**When contradiction detected, present to user:**

```
⚠️ VLM Contradiction Detected

Issue: Hero card width

Comparative Analysis Says:
  "Make hero card narrower than list cards"

Original Verification Says:
  "Hero has 20px margins (wider than list's 30px)"

Current Implementation:
  Hero margins: 16px

Options:
  A) Keep comparative analysis suggestion (make narrower)
  B) Follow original verification (keep wider)
  C) Different approach (specify)

Which should we use?
```

**Wait for user decision before proceeding.**

---

### Step 5: Apply Only Verified Fixes

**Apply changes in order:**

1. **No contradiction** → Apply immediately
2. **Contradiction** → Wait for human approval, then apply
3. **Re-validate** → Screenshot and check if fix worked
4. **Iterate** → Move to next fix

---

## Why VLM Checklists Can Be Wrong

### Problem 1: Comparative Analysis Errors

When comparing two images, VLM might:
- Misinterpret perspective differences
- Get confused by different content
- Make measurement errors
- Apply generic design rules instead of observing actual design

### Problem 2: Context Confusion

```
Checklist prompt: "Is hero narrower than list?"
VLM might think: "Heroes are usually wider (design convention)"
Instead of: "What do I actually SEE in this specific mockup?"
```

### Solution: Always verify measurements from original in isolation

---

## The Verification Pattern

### Before Applying ANY VLM-Suggested Fix:

**Step 1: Get ground truth from original**
```
Prompt: "Look at the ORIGINAL mockup ONLY.

Measure:
- Hero card margins from screen edges
- List card margins from screen edges

Which is wider? Provide exact measurements."
```

**Step 2: Compare with suggestion**
```
VLM checklist said: "Make hero narrower"
VLM original measurement: "Hero has less margin (is wider)"

→ CONTRADICTION!
→ Don't apply automatically
→ Ask user or get more evidence
```

**Step 3: If no contradiction, apply with confidence**

---

## The Complete Verification Workflow

```
┌─────────────────────────────────────────────┐
│ VLM Detail Checklist (Comparison)           │
│ → Flags N potential issues                  │
└────────────────┬────────────────────────────┘
                 ▼
┌─────────────────────────────────────────────┐
│ Verification Loop (For each issue):         │
│                                              │
│ 1. VLM looks at ORIGINAL ONLY               │
│    → What is ground truth?                  │
│                                              │
│ 2. Compare with checklist suggestion        │
│    → Match or contradiction?                │
│                                              │
│ 3. If MATCH: Apply fix                      │
│    If CONTRADICTION: Flag for human         │
└────────────────┬────────────────────────────┘
                 ▼
┌─────────────────────────────────────────────┐
│ Human Approval (For contradictions only)    │
│                                              │
│ Present:                                     │
│ - What checklist suggests                   │
│ - What original shows                       │
│ - What current has                          │
│                                              │
│ User decides: Apply, Skip, or Third Option  │
└────────────────┬────────────────────────────┘
                 ▼
┌─────────────────────────────────────────────┐
│ Apply Approved Changes                      │
│ → Re-validate                               │
│ → Iterate                                   │
└─────────────────────────────────────────────┘
```

---

## Key Principles

1. **VLMs can be wrong** - Always verify against ground truth
2. **Ground truth = original mockup** - Not VLM's interpretation
3. **Contradictions = human decision** - Don't guess
4. **Measurements > opinions** - "20px margins" not "looks narrower"
5. **Isolate the original** - Look at it alone, not in comparison
6. **When in doubt, ask the user** - Their eyes are the final validator

---

## Real-World Example: Hero Card Width Issue

### Issue Flagged by Checklist:
```json
{
  "item": "Hero card width",
  "status": "DIFFERENT",
  "original": "~350px (narrower than list cards)",
  "fix_needed": "Reduce hero card width or add horizontal margin"
}
```

### Verification (Original Only):
```
Hero margins: 20px
List card margins: 30px
→ Hero is WIDER
```

### Contradiction Detected!
- Checklist: "Make narrower"
- Original measurement: "Hero is wider"

### What Should Have Happened:
- NOT apply the fix automatically
- Present both to user
- Ask: "Should hero be wider or narrower?"

### What Actually Happened:
- Applied the fix blindly ❌
- User caught the error
- Had to revert

---

## Success Metrics

### Good Sign:
- VLM checklist identifies issue
- VLM verification confirms issue  
- Fix applied
- Re-validation shows improvement
- User confirms it looks better

### Bad Sign:
- VLM checklist identifies issue
- VLM verification contradicts it
- Change applied anyway
- User says "you went backwards"

---

## Implementation Guide

### Tool: Comparative Analysis
```python
def comparative_checklist(original_img, current_img):
    """
    Generate list of potential issues by comparing images.
    
    WARNING: These suggestions may be wrong!
    MUST verify each against original before applying.
    """
    prompt = f"""
    Compare these images and list differences.
    
    Original: {original_img}
    Current: {current_img}
    
    Output JSON array of issues.
    """
    return vlm_analyze(prompt, [original_img, current_img])
```

### Tool: Verification
```python
def verify_against_original(original_img, issue):
    """
    Verify a suggested fix against the original mockup only.
    
    This is ground truth. If it contradicts the checklist,
    the checklist is WRONG.
    """
    prompt = f"""
    Look at ONLY this original mockup.
    
    Issue to verify: {issue['description']}
    
    Measure the actual state in the original.
    Provide specific measurements.
    """
    return vlm_analyze(prompt, [original_img])
```

### Tool: Contradiction Detection
```python
def detect_contradiction(checklist_item, verification_result):
    """
    Compare checklist suggestion with verification.
    Flag if they contradict.
    """
    if suggests_opposite_change(checklist_item, verification_result):
        return {
            "contradiction": True,
            "checklist": checklist_item,
            "verification": verification_result,
            "action": "REQUIRE_HUMAN_APPROVAL"
        }
    return {"contradiction": False, "action": "SAFE_TO_APPLY"}
```

---

## Summary

**The Golden Rule:** Never blindly apply VLM comparative analysis. Always verify against original mockup.

**The Process:**
1. Comparative checklist identifies potential issues
2. Verify EACH issue against original only
3. Detect contradictions
4. Apply verified fixes automatically
5. Get human approval for contradictions

**Critical Insight:** VLMs are excellent at visual analysis but can contradict themselves when doing comparative analysis. The original mockup is the only source of truth. When checklist and verification disagree, the verification is correct.
