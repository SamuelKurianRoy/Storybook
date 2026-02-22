# Debug Log: Left Page Visibility Issue

## Problem Description
In a 3D CSS flip book animation, the left side pages (backs of flipped pages) are not displaying correctly. Only the first page back shows, but subsequent page backs don't appear as the user navigates through the book.

**Expected Behavior:**
- When viewing page spread (e.g., pages 9-10)
- Left side should show page 9 (the back of page-4 div)
- Right side should show page 10 (the front of page-5 div)

**Actual Behavior:**
- Only the first page back (page 1) appears on the left
- After page 1, the left side remains blank as you flip through the book

---

## Technical Context

### Book Structure
- 13 pages total (page-0 through page-12)
- Each page div contains:
  - `.page-front` - Shows when page is unflipped (right side)
  - `.page-back` - Should show when page is flipped (left side)
- Pages use `transform: rotateY(-180deg)` for flip animation
- All pages positioned with `right: 0` and `transform-origin: left center`

### JavaScript Logic
- `currentPage` variable tracks progress (starts at -1)
- Pages with `index <= currentPage` get `.flipped` class
- `updateBookState()` function controls visibility

---

## Attempted Solutions (All Failed)

### Attempt 1: Removed `backface-visibility: hidden`
**What was tried:**
- Removed CSS property `backface-visibility: hidden` from `.page-side` rule
- Theory: This property was preventing page backs from being visible after rotation

**Result:** ❌ Failed
- Removing this caused ALL page backs to show simultaneously
- Pages stacked on top of each other, creating visual chaos
- Issue: Needed to control which backs are visible, not show all of them

---

### Attempt 2: Added JavaScript Visibility Control
**What was tried:**
- Modified `updateBookState()` function to explicitly set opacity/visibility on `.page-back` elements
- Logic: Only show back when `index === currentPage`
- Hide backs for all other flipped pages

```javascript
if (index === currentPage) {
    pageBack.style.opacity = '1';
    pageBack.style.visibility = 'visible';
} else {
    pageBack.style.opacity = '0';
    pageBack.style.visibility = 'hidden';
}
```

**Result:** ❌ Failed
- Only first page back showed, then subsequent pages had blank left side
- Issue: CSS rules may have been conflicting with inline styles

---

### Attempt 3: Added Display Property Control
**What was tried:**
- Added `display: none/block` to JavaScript visibility control
- Theory: `display` might be more forceful than opacity/visibility

```javascript
pageBack.style.display = index === currentPage ? 'block' : 'none';
```

**Result:** ❌ Failed
- Same issue persisted
- Left side still blank after first page

---

### Attempt 4: Repositioned Flipped Pages to Left Side
**What was tried:**
- Changed positioning logic: flipped pages use `left: 0`, unflipped use `right: 0`
- Changed `transform-origin` to `right center` for flipped pages
- Changed rotation to `rotateY(180deg)` instead of `-180deg`

**Result:** ❌ Complete Failure
- Entire flip animation broke
- Even the first page disappeared
- Pages no longer visible at all on either side
- Had to revert immediately

**Lesson Learned:**
- Pages MUST stay positioned on right side (`right: 0`)
- `transform-origin: left center` is essential for proper flip mechanics
- Moving pages to left side fundamentally breaks the 3D transform

---

### Attempt 5: Explicit Front/Back Visibility Control
**What was tried:**
- Extended JavaScript to control BOTH page-front AND page-back visibility
- For flipped pages: hide front (`opacity: 0`), show only current back
- For unflipped pages: show front, hide back

**Result:** ❌ Failed
- Issue reverted to previous state
- Only first left page visible, rest blank

---

### Attempt 6: Removed Conflicting CSS Rules
**What was tried:**
- Identified duplicate CSS rules for `.page.flipped .page-back`
- Removed rules that set `opacity: 1; visibility: visible` on all flipped backs
- Added `pointer-events` control to JavaScript
- Added `display` property control

**JavaScript changes:**
```javascript
if (index === currentPage) {
    pageBack.style.opacity = '1';
    pageBack.style.visibility = 'visible';
    pageBack.style.display = 'block';
    pageBack.style.pointerEvents = 'auto';
} else {
    pageBack.style.opacity = '0';
    pageBack.style.visibility = 'hidden';
    pageBack.style.display = 'none';
    pageBack.style.pointerEvents = 'none';
}
```

**CSS changes:**
- Removed `.page.flipped .page-back { opacity: 1; visibility: visible; }`
- Left only `.page:not(.flipped) .page-back { opacity: 0; visibility: hidden; }`

**Result:** ❌ Failed
- Same behavior continues
- Left side still not updating after first page

---

## Current State

### What Works
✅ First page back displays correctly (page 1 on left side)
✅ Right side pages display correctly throughout the book
✅ Page flip animation works smoothly
✅ Z-index stacking appears correct

### What Doesn't Work
❌ Left side pages after the first one don't show
❌ When currentPage > 0, only the first flipped page back is visible

### HTML State (When on Page 12-13 spread)
From browser inspection showing all flipped page backs have:
```html
style="opacity: 0; visibility: hidden;"
```
Only page-6's back shows: `style="opacity: 1; visibility: visible;"`

But visually: nothing appears on the left side.

---

## Hypotheses for Root Cause

### Hypothesis 1: Z-Index Stacking Issue
- Perhaps flipped pages have z-index that places them behind something
- Z-index formula: `10 + index` for flipped pages
- This should make higher index pages stack on top

### Hypothesis 2: 3D Transform Perspective Issue
- The `rotateY(-180deg)` might be positioning backs in a way that's not visible
- Perspective or transform-style might need adjustment
- Parent container has `transform-style: preserve-3d`

### Hypothesis 3: CSS Specificity War
- Despite removing duplicate rules, other CSS might override inline styles
- Possible cascade issue where inline styles aren't winning
- Browser might be applying styles in unexpected order

### Hypothesis 4: Page-back Transform Issue
- `.page-back` elements have `transform: rotateY(180deg)` 
- Combined with parent's `rotateY(-180deg)` might cause unexpected positioning
- Double rotation might place content off-screen or behind other elements

### Hypothesis 5: Browser Rendering Bug
- 3D CSS transforms can have rendering glitches
- Might need hardware acceleration hints
- Could require `will-change` or `translateZ` hacks

---

## Code Locations

### Key Files
- **Main File:** `index_new.html`

### Key Code Sections
- **updateBookState() function:** Lines ~1035-1085
- **CSS .page-back rules:** Lines ~168-188
- **CSS .page.flipped rules:** Lines ~99-111
- **Page flip mechanics:** Lines ~89-103

---

## Next Steps to Try

### Option 1: Debug Logging
Add console.log statements to verify:
- Which page is currentPage
- Inline styles being set
- Computed styles after JavaScript runs
- Whether page-back elements exist

### Option 2: Force z-index Override
Set very high z-index on current page back:
```javascript
pageBack.style.zIndex = '9999';
```

### Option 3: Force Rendering with translateZ
Add transform property to force GPU rendering:
```javascript
pageBack.style.transform = 'rotateY(180deg) translateZ(1px)';
```

### Option 4: Simplify CSS Reset
Set all properties via JavaScript:
```javascript
pageBack.style.cssText = 'opacity: 1 !important; visibility: visible !important; display: block !important; transform: rotateY(180deg); position: absolute; width: 100%; height: 100%; top: 0; left: 0;';
```

### Option 5: Check Computed Styles
Inspect what the browser actually computes after all CSS/JS applied
May reveal hidden override or inherited property

### Option 6: Test Without CSS
Temporarily disable ALL page-back CSS rules
Control everything via JavaScript inline styles
Would confirm if CSS is the blocker

---

## Timeline Summary
1. ✅ Initial working state (all backs hidden by `backface-visibility`)
2. ❌ Removed backface-visibility → all backs showed
3. ❌ Added JS visibility control → only first back showed
4. ❌ Added display property → same issue
5. ❌ Tried repositioning → complete failure
6. ❌ Controlled front+back visibility → same issue  
7. ❌ Removed CSS conflicts + added full property control → **current state, still failing**

---

## Conclusion
After 7 different approaches, the issue persists. The problem appears to be deeper than simple CSS/JavaScript visibility control. It may involve:
- 3D transform rendering pipeline
- Browser-specific rendering behavior
- Complex interaction between CSS transforms and inline style properties
- Possible need for completely different architectural approach

The fact that the first page back works but subsequent ones don't suggests the logic is partially correct, but something in the rendering layer is preventing updates from taking effect.

**Date:** February 13, 2026  
**Status:** Unresolved  
**Priority:** High - Book is unusable without visible left pages
