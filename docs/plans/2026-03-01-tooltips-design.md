# Tooltip System - Design Document

**Date:** 2026-03-01
**Status:** Approved

## Overview

Add a comprehensive tooltip system to make the Quarterly Planning Tracker self-descriptive for first-time users. Tooltips will explain UI sections, interactive elements, and visual indicators throughout the application.

## Goals

- Make the application self-explanatory for new users
- Explain complex concepts (team states, capacity, commitment status)
- Provide contextual help without cluttering the interface
- Maintain accessibility (keyboard navigation, screen readers)
- Desktop-focused (mobile not a priority)

## Non-Goals

- Mobile-optimized tooltips
- Interactive content within tooltips
- User preferences/settings for tooltips
- Analytics on tooltip usage

## Architecture & Approach

### Hybrid Tooltip System

We'll implement three types of tooltips:

1. **Info Icon Tooltips (ⓘ)** - For section headers and complex concepts
   - Clickable/hoverable icons
   - Multi-line formatted text
   - Custom styled popovers

2. **Inline Tooltips** - For simple UI elements
   - Native HTML `title` attribute
   - Browser default styling
   - Minimal implementation

3. **Visual Indicator Tooltips** - For color-coded elements
   - Hover-triggered explanations
   - Compact, color-matched styling

### Technical Stack

- **Pure CSS** for tooltip styling (no external libraries)
- **Vanilla JavaScript** for interaction behavior
- **Native HTML5** accessibility attributes
- **Zero dependencies** (consistent with app philosophy)

## Tooltip Locations

### Section Headers (4 tooltips)
1. "Initiatives and Team Commitment" ⓘ
2. "Team Summary" ⓘ
3. "[Quarter] Commitment" ⓘ (dynamic quarter name)
4. "Next" ⓘ

### State Column Headers (4 tooltips)
5. "Uncommitted" ⓘ
6. "Committed" ⓘ
7. "Completed" ⓘ
8. "Not Participating" ⓘ

### Header Controls (5 tooltips)
9. Quarter input ⓘ
10. Max Initiatives per Team input ⓘ
11. Export JSON button ⓘ
12. Import JSON button ⓘ
13. Clear All Data button ⓘ

### Visual Indicators (hover only)
- Initiative colored borders (green/yellow/red)
- Team capacity colors (green/yellow/red)
- Status badges in Next section

### Simple Elements (native title attribute)
- Drag handles (⋮⋮)
- Delete buttons (×)
- Team tags

**Total:** 13 info icon tooltips + multiple hover tooltips

## Visual Design

### Info Icon (ⓘ) Style

**Appearance:**
- Unicode character: ⓘ (U+24D8)
- Size: 14px
- Default color: `#95a5a6` (subtle gray)
- Hover color: `#2c3e50` (prominent dark gray)
- Cursor: pointer
- Margin: 0 0 0 6px (spacing from text)

**Positioning:**
- Inline with text (to the right of headers/labels)
- Vertically aligned with text baseline

### Tooltip Popover Style

**Container:**
- Background: `#2c3e50` (matches app header)
- Text color: `#ffffff` (white)
- Padding: 12px 16px
- Border-radius: 6px
- Max-width: 300px
- Font-size: 13px
- Line-height: 1.5
- Box-shadow: `0 4px 12px rgba(0,0,0,0.15)`
- Z-index: 1000

**Arrow/Pointer:**
- Small triangle pointing to info icon
- Same color as tooltip background (`#2c3e50`)
- Position: Depends on tooltip placement

**Animation:**
- Fade in: 0.2s ease-in
- Fade out: 0.2s ease-out

**Positioning Logic:**
- Default: Below and centered on icon
- Fallback: Above or to side if near screen edge
- Smart positioning to stay within viewport

### Visual Indicator Tooltips

**Style:**
- Compact, single-line format
- Max-width: 200px
- Font-size: 12px
- Match color of indicator being explained
- Simpler styling than info tooltips

## Interaction Behavior

### Info Icon Tooltips (ⓘ)

**Mouse Interaction:**
- **Hover**: Show tooltip after 300ms delay
- **Click**: Toggle tooltip (show/hide)
- **Click outside**: Hide tooltip
- **Mouse leave**: Keep visible (user may want to read)

**Keyboard Interaction:**
- **Tab**: Focus on info icon
- **Enter/Space**: Toggle tooltip
- **ESC**: Close tooltip
- **Tab out**: Close tooltip

**Multiple Tooltips:**
- Only one info tooltip visible at a time
- Opening new tooltip closes previous one

### Simple Hover Tooltips (native title)

**Mouse Interaction:**
- **Hover**: Browser default (instant show)
- **Leave**: Browser default (instant hide)

### Visual Indicator Tooltips

**Mouse Interaction:**
- **Hover**: Show immediately (no delay)
- **Leave**: Hide immediately

## Accessibility

### Keyboard Navigation
- Info icons are focusable (`tabindex="0"`)
- Enter/Space key opens tooltips
- ESC key closes tooltips
- Focus returns to info icon after closing
- Visible focus indicator on info icons

### Screen Readers
- Info icons have `aria-label="More information"`
- Tooltips have `role="tooltip"`
- Tooltip content is associated via `aria-describedby`
- All tooltip text is screen reader accessible

### Focus Management
- Focus indicator visible on all focusable elements
- Logical tab order maintained
- No focus traps (tooltips are non-interactive)

## Implementation Structure

### HTML Structure

**Info Icon + Tooltip:**
```html
<h2>
  Section Title
  <span class="info-icon" tabindex="0" aria-label="More information">ⓘ
    <span class="tooltip" role="tooltip">
      Tooltip content goes here...
    </span>
  </span>
</h2>
```

**State Column Header:**
```html
<div class="state-column-header">
  State Name
  <span class="info-icon" tabindex="0" aria-label="More information">ⓘ
    <span class="tooltip" role="tooltip">
      Explanation of this state...
    </span>
  </span>
</div>
```

**Header Control:**
```html
<label>
  Label Text
  <span class="info-icon" tabindex="0" aria-label="More information">ⓘ
    <span class="tooltip" role="tooltip">
      What this control does...
    </span>
  </span>
  <input type="text" />
</label>
```

### CSS Classes

**New classes:**
- `.info-icon` - Icon styling and positioning
- `.info-icon:hover` - Hover state
- `.info-icon:focus` - Focus state for accessibility
- `.tooltip` - Tooltip popover container (hidden by default)
- `.tooltip.visible` - Visible state (toggled by JavaScript)
- `.tooltip::before` - Arrow/pointer
- `.tooltip-compact` - Compact style for visual indicators

### JavaScript Behavior

**Event Handlers:**
- Click handler on `.info-icon` to toggle tooltips
- Document click handler to close tooltips when clicking outside
- ESC key handler to close open tooltips
- Focus/blur handlers for keyboard navigation
- Mouseenter/mouseleave for hover behavior

**State Management:**
- Track currently open tooltip (only one at a time)
- Close previous tooltip when opening new one
- Clean up event listeners properly

**Positioning Logic:**
- Calculate available space around icon
- Position tooltip below by default
- Reposition if near viewport edge
- Add arrow pointing to icon

## Tooltip Content

See `docs/tooltip-content.md` for the complete list of tooltip text. Key principles:

- **Concise**: Short, actionable explanations
- **Helpful**: Focus on what users need to know
- **Consistent**: Similar tone and structure across tooltips
- **Accurate**: Reflect current app behavior

## Success Criteria

The tooltip system succeeds if:
- New users can understand the app without external documentation
- Tooltips explain complex concepts (team states, capacity, commitment)
- Interface remains clean and uncluttered
- Keyboard navigation works seamlessly
- Screen readers can access all tooltip content
- Tooltips don't interfere with normal workflow

## Future Enhancements (Not in Scope)

- User preference to hide/show tooltips permanently
- Rich media in tooltips (images, videos)
- Interactive tooltips with links or actions
- Mobile-optimized touch interactions
- Contextual help based on user actions
- Tutorial mode with sequential tooltips
