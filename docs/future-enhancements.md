# Future Enhancements - Quarterly Planning Tracker

**Date:** 2026-02-28

## Planned Changes

### 1. ✅ Revisit Initiative Status in Next Section (COMPLETED)
- ~~Review the current status badges (⚠ Partially Committed, ✕ Not Started)~~
- ~~Consider whether the current status representation is optimal~~
- ~~Evaluate if additional status levels or different indicators would be more useful~~
- **Implemented:** Changed to "✓ Can Proceed" (green) and "⏸ Blocked" (red)
- **Commit:** 53bde6c - Action-focused labels distinguish actionable vs blocked initiatives

### 2. ✅ Initiative Priority Ordering in Next Section (COMPLETED)
- ~~Add ability to reorder initiatives in the Next section~~
- ~~Ordering should indicate priority~~
- ~~Consider drag-and-drop reordering or up/down buttons~~
- ~~Priority should be saved and persisted~~
- **Implemented:** Drag-and-drop with ⋮⋮ grip handle on left side of initiative rows
- **Commit:** 96ef5c6 - Reordering affects all sections (Initiatives, Quarter Commitment, Next, Team Summary)

### 3. ✅ Team Non-Contribution Concept (COMPLETED)
- ~~Introduce the idea that a team may not be contributing to an initiative at all~~
- ~~This is different from uncommitted/committed/completed states~~
- ~~Consider use cases: some initiatives don't need certain teams~~
- ~~May need a new state or way to indicate "not applicable" for a team~~

**Design Decisions:**
- **Visual Representation:** 4th column "Not Applicable" or "N/A" in initiative rows
- **Interaction Method:** Drag team tag to N/A column (consistent with current pattern)
- **Team Summary Impact:** Non-contributing initiatives do NOT appear in team's column
- **Next Section Impact:** Non-contributing teams do NOT appear in "Waiting for..." column
- **Capacity Calculation:** Non-contributing initiatives do NOT count toward team capacity
- **Commitment Status:** Ignore N/A teams when calculating status (only count uncommitted/committed/completed)

**Data Model Change:**
- Add new state value: `'not-applicable'` alongside `'uncommitted'`, `'committed'`, `'completed'`
- Update initiative.teamStates to support: `{teamId: 'not-applicable'}`

**Implementation:** ✅ All completed
- ✅ Add 4th column to initiative row grid
- ✅ Add "N/A" state column with drop zone
- ✅ Update capacity calculation logic to exclude N/A
- ✅ Update Team Summary to filter out N/A initiatives
- ✅ Update Next section to filter out N/A teams
- ✅ Update commitment status calculation to ignore N/A teams
- ✅ Update CSS grid template columns (add 1 more column)
- **Commit:** dcd8961 - Teams can be dragged to N/A column, excluded from all calculations

### 4. ✅ Revisit Initiative Name Field (COMPLETED)
- ~~Review the current inline editing approach~~
- ~~Consider improvements to UX/UI~~
- ~~Evaluate if there are any issues with the current implementation~~
- **Problems Identified:** Names getting cut off (200px limit), hierarchy issue (name looked like data entry field)
- **Implemented:** Restructured as section header with full width
- **Design:** Bold, 18px font, header/body layout, hover/focus states
- **Commit:** a20c7fc - Name now prominent as initiative title, no truncation

### 5. ✅ Revisit Delete Button Color (COMPLETED)
- ~~Review current delete button styling (red background, white ×)~~
- ~~Consider alternative color schemes or visual treatments~~
- ~~Ensure good contrast and visibility~~
- **Problem Identified:** Red confused with team status colors, button too large (40px)
- **Implemented:** Subtle gray button (#95a5a6), reduced to 30px, darker hover state
- **Commit:** 344c7ba - Clear distinction between controls and status indicators

### 6. Team Management UI
- Add ability to create new teams via UI (currently requires JSON editing)
- Add ability to rename existing teams
- Consider ability to delete teams (with appropriate warnings)
- Design UI for team management (modal, inline, or separate section)

### 7. ✅ Revisit Team Names in Team Summary (COMPLETED)
- ~~Review visual treatment of team names in Team Summary section~~
- ~~Current: "Backend Team (3/5)" with colored text and left border~~
- ~~Consider: Font size, weight, prominence, hierarchy~~
- ~~Evaluate: Capacity indicator styling "(3/5)"~~
- ~~Assess: Color treatment (currently text color changes based on capacity)~~
- ~~Goal: Make team names more visually prominent and clear~~
- **Implemented:** Side-by-side layout with visual separation
- **Design:** Team name bold 16px dark gray | Capacity count 14px capacity-colored
- **Result:** Team name always readable, capacity indicator visually distinct
- **Commit:** 7470744 - Flexbox layout separates name from capacity with clear visual hierarchy

### 8. ✅ Priority-Based Visual Signals in Next Section (COMPLETED)
- ~~Add color coding to signal the relationship between initiative priority and status~~
- ~~Show when highest priority initiatives are blocked vs progressing~~
- ~~Show when lower priority initiatives are progressing vs blocked~~
- ~~Consider: Color-coded borders, backgrounds, or status indicators~~
- ~~Goal: Make priority/status conflicts immediately visible~~
- **Implemented:** ⚠ indicators on initiative names with hover-triggered tooltips
- **Design:** Detects when higher priority initiatives need teams that are committed to lower priority work
- **Interaction:** Hover over indicator to see list of conflicting initiatives
- **Commit:** c91b5a7 (and related commits)

## Implementation Notes

- Each item should be addressed individually
- User feedback and iteration expected for each change
- Maintain backward compatibility with existing data where possible
- Update README.md as features are added
