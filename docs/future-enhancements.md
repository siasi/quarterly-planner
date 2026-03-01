# Future Enhancements - Quarterly Planning Tracker

**Date:** 2026-02-28

## Planned Changes

### 1. ✅ Revisit Initiative Status in Next Section (COMPLETED)
- ~~Review the current status badges (⚠ Partially Committed, ✕ Not Started)~~
- ~~Consider whether the current status representation is optimal~~
- ~~Evaluate if additional status levels or different indicators would be more useful~~
- **Implemented:** Changed to "✓ Can Proceed" (green) and "⏸ Blocked" (red)
- **Commit:** 53bde6c - Action-focused labels distinguish actionable vs blocked initiatives

### 2. Initiative Priority Ordering in Next Section
- Add ability to reorder initiatives in the Next section
- Ordering should indicate priority
- Consider drag-and-drop reordering or up/down buttons
- Priority should be saved and persisted

### 3. Team Non-Contribution Concept
- Introduce the idea that a team may not be contributing to an initiative at all
- This is different from uncommitted/committed/completed states
- Consider use cases: some initiatives don't need certain teams
- May need a new state or way to indicate "not applicable" for a team

### 4. Revisit Initiative Name Field
- Review the current inline editing approach
- Consider improvements to UX/UI
- Evaluate if there are any issues with the current implementation

### 5. Revisit Delete Button Color
- Review current delete button styling (red background, white ×)
- Consider alternative color schemes or visual treatments
- Ensure good contrast and visibility

### 6. Team Management UI
- Add ability to create new teams via UI (currently requires JSON editing)
- Add ability to rename existing teams
- Consider ability to delete teams (with appropriate warnings)
- Design UI for team management (modal, inline, or separate section)

## Implementation Notes

- Each item should be addressed individually
- User feedback and iteration expected for each change
- Maintain backward compatibility with existing data where possible
- Update README.md as features are added
