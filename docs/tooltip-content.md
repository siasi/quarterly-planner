# Tooltip Content for Quarterly Planning Tracker

This file contains all the tooltip text that will be displayed in the application. Edit the descriptions as needed.

---

## Section Headers

### "Initiatives and Team Commitment" ⓘ
**Tooltip Text:**
```
Manage your quarterly initiatives and track teams commitment. Drag team tags between columns to update their commitment status. The colored left border shows overall initiative status: green (all teams committed), yellow (partially committed), red (no team commited).
```

### "Team Summary" ⓘ
**Tooltip Text:**
```
Shows the current ask for each team. The number (e.g., "3/5") indicates committed initiatives versus your maximum threshold. Green headers mean the team is below threashold, red means they're at or over it. Only committed and uncommitted initiatives are shown here.
```

### "Q1 2026 Commitment" ⓘ (dynamic - uses quarter name)
**Tooltip Text:**
```
Initiatives that can make progress in this quarter and that are the next to be completed, as they have commitment from all requiried teams. You can capture an ETA that depends by estimations. Use this view to communicate where you are putting your effort, to get things done, and what you expect will be completed soon. 
```

### "Next" ⓘ
**Tooltip Text:**
```
Initiatives that still have uncommitted teams. These are being discussed but not yet fully committed. The status shows whether work can proceed, but with no chance to complete yet (only some teams committed) or is blocked (no teams committed).
```

---

## State Columns in Initiatives Section

### "Uncommitted" ⓘ
**Tooltip Text:**
```
Teams that are being asked to participate but haven't committed yet. Drag teams here during initial planning discussions.
```

### "Committed" ⓘ
**Tooltip Text:**
```
Teams that have committed to work on this initiative this quarter. This counts toward their capacity.
```

### "Completed" ⓘ
**Tooltip Text:**
```
Teams that have finished their work on this initiative. Completed work doesn't count toward capacity.
```

### "Not Participating" ⓘ
**Tooltip Text:**
```
Teams that are not involved in this initiative at all. These teams are excluded from capacity calculations and commitment status.
```

---

## Header Controls

### "Quarter" input
**Tooltip Text:**
```
The planning period (e.g., "Q1 2026"). Click to edit.
```

### "Max Initiatives per Team" input
**Tooltip Text:**
```
The maximum number of committed initiatives before to flag a concentration of asks. Used to highlight high requests for a team in the Team Summary.
```

### "Export JSON" button
**Tooltip Text:**
```
Download your current planning data as a JSON file. Use this to share your changes with colleagues (this is an MVP to get feedback about the value of this approach).
```

### "Import JSON" button
**Tooltip Text:**
```
Load a previously exported planning file. This will replace all current data.
```

### "Clear All Data" button
**Tooltip Text:**
```
Reset the application to sample data. This will erase all your current initiatives and team commitments.
```

---

## Visual Indicators

### Colored left border on initiatives (hover tooltip)
**Tooltip Text:**
```
🟢 Green: All teams committed
🟡 Yellow: Partially committed
🔴 Red: No team committed
```

### Team capacity colors (hover tooltip on capacity number)
**Tooltip Text:**
```
🟢 Green: Team has no committment
🟡 Yellow: Team has some committment, below threshold
🔴 Red: Team has some committment, at/over threshold 
```

### Initiative status badges in Next section
**Tooltip Text for "✓ Can Proceed":**
```
Some teams are committed, work can proceed
```

**Tooltip Text for "⏸ Blocked":**
```
No teams committed yet, work is blocked
```

---

## Interactive Elements (Simple tooltips using native 'title' attribute)

### Drag handle (⋮⋮)
**Tooltip Text:**
```
Drag to change initiatives' priority
```

### Delete button (×)
**Tooltip Text:**
```
Delete this initiative
```

### Team tags in state columns
**Tooltip Text:**
```
Drag to change commitment status
```

---

## Notes

- **Info icon tooltips (ⓘ)**: Will be clickable/hoverable with styled popovers
- **Simple tooltips**: Will use native HTML `title` attribute
- **Visual indicators**: Will show on hover with custom styled tooltips
- All tooltips should be concise, actionable, and helpful for first-time users
