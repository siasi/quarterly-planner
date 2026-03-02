# Priority Inversion Signals Design

**Date:** 2026-03-02
**Status:** Approved

## Overview

Add visual indicators to the Next and Quarter Commitment sections that signal when higher priority initiatives are blocked while lower priority initiatives are progressing. This highlights resource allocation issues where teams are working on lower priority work while higher priority work waits.

## Problem Statement

During quarterly planning, it's critical to identify priority inversions:
- High priority initiatives are blocked or partially committed
- Lower priority initiatives have full team commitment and are progressing
- Teams needed for high priority work are busy on lower priority initiatives

These situations require immediate attention and reallocation decisions, but are currently difficult to spot without manually comparing initiative priorities and team commitments.

## User Requirements

### Detection Criteria

A priority inversion occurs when:
1. **Higher priority initiative** (earlier in list):
   - Has at least one uncommitted team
   - Can be either "⏸ Blocked" or "✓ Can Proceed" status

2. **Lower priority initiative** (later in list):
   - Has "✓ Can Proceed" status (at least one team committed)
   - At least one of its committed teams is uncommitted on the higher priority initiative

### Visual Indicators

- Show ⚠️ warning icon before initiative names
- Higher priority initiatives: "⚠️ Blocked by X lower priority initiatives"
- Lower priority initiatives: "⚠️ Blocking X higher priority initiatives"
- Click indicator to see list: "Blocked by: Initiative A, Initiative B"

### Scope

- Show indicators in **Next section** (for blocked/partially committed initiatives)
- Show indicators in **Quarter Commitment section** (for fully committed initiatives)
- Do NOT show in Initiatives section (too noisy, not actionable there)

## Design Approach

### Approach Selected: Pre-Compute Conflict Map (Approach 2)

**Rationale:** Separates conflict detection logic from rendering logic, making both easier to understand, test, and maintain. While slightly more complex than computing during render, the separation of concerns is worth the trade-off.

**How it works:**
1. Create `analyzePriorityConflicts()` function that examines all initiatives
2. Build conflict map: `{initiativeId: {blockedBy: [ids], blocking: [ids]}}`
3. Call analysis function after any data changes
4. Pass conflict map to rendering functions
5. Use conflict data to add warning indicators during render

## Technical Design

### 1. Conflict Detection Logic

**New Function: `analyzePriorityConflicts()`**

```javascript
function analyzePriorityConflicts() {
  const conflictMap = {};

  // Initialize map for all initiatives
  appState.initiatives.forEach(initiative => {
    conflictMap[initiative.id] = { blockedBy: [], blocking: [] };
  });

  // Check each initiative against higher priority initiatives
  appState.initiatives.forEach((currentInit, currentIndex) => {
    // Skip if current initiative is blocked (not progressing)
    if (getInitiativeStatus(currentInit) === 'blocked') {
      return;
    }

    const currentCommittedTeams = getCommittedTeams(currentInit);

    // Check all higher priority initiatives (lower indices)
    for (let i = 0; i < currentIndex; i++) {
      const higherPriorityInit = appState.initiatives[i];
      const uncommittedTeams = getUncommittedTeams(higherPriorityInit);

      // Check for team overlap
      const hasOverlap = hasTeamOverlap(currentCommittedTeams, uncommittedTeams);

      if (hasOverlap) {
        // Record the conflict
        conflictMap[higherPriorityInit.id].blockedBy.push(currentInit.id);
        conflictMap[currentInit.id].blocking.push(higherPriorityInit.id);
      }
    }
  });

  return conflictMap;
}
```

**Helper Functions:**

```javascript
function getInitiativeStatus(initiative) {
  const activeStates = getActiveTeamStates(initiative);
  const hasCommitted = activeStates.some(state =>
    state === TEAM_STATES.COMMITTED || state === TEAM_STATES.COMPLETED
  );
  return hasCommitted ? 'can-proceed' : 'blocked';
}

function getCommittedTeams(initiative) {
  return Object.keys(initiative.teamStates).filter(teamId => {
    const state = initiative.teamStates[teamId];
    return state === TEAM_STATES.COMMITTED || state === TEAM_STATES.COMPLETED;
  });
}

function getUncommittedTeams(initiative) {
  return Object.keys(initiative.teamStates).filter(teamId => {
    const state = initiative.teamStates[teamId];
    return state === TEAM_STATES.UNCOMMITTED;
  });
}

function hasTeamOverlap(array1, array2) {
  return array1.some(item => array2.includes(item));
}
```

### 2. Data Flow

**Trigger Points:**

Call `analyzePriorityConflicts()` after:
- Initial data load (`loadFromLocalStorage()`)
- Initiative reordering (`handleRowDrop()`)
- Team state changes (team tag drop handler)
- Adding/deleting initiatives
- JSON import

**Integration with Render:**

```javascript
function render() {
  const teamColorCache = buildTeamColorCache();
  const conflictMap = analyzePriorityConflicts(); // Compute here

  renderInitiatives(teamColorCache);
  renderInitiativesSummary(teamColorCache, conflictMap); // Pass to sections that need it
  renderTeamSummary(teamColorCache);
}

function renderInitiativesSummary(teamColorCache, conflictMap) {
  // Update quarter section title...

  const { committed, uncommitted } = splitInitiativesByCommitment();
  renderQuarterSection(committed, teamColorCache, conflictMap);
  renderNextSection(uncommitted, teamColorCache, conflictMap);
}
```

**Function Signature Updates:**
- `renderNextSection(initiatives, teamColorCache, conflictMap)`
- `renderQuarterSection(committed, teamColorCache, conflictMap)`

### 3. Visual Components

**Warning Indicator Styling:**

```css
.priority-conflict-indicator {
  cursor: pointer;
  margin-right: 6px;
  color: #f39c12; /* Orange/amber warning color */
  font-size: 16px;
  user-select: none;
}

.priority-conflict-indicator:hover {
  color: #e67e22; /* Darker orange on hover */
}
```

**Adding Indicator to Initiative Name:**

```javascript
function addConflictIndicator(nameCell, initiative, conflictMap) {
  const conflicts = conflictMap[initiative.id];

  // Check if this initiative has any conflicts
  if (!conflicts || (conflicts.blockedBy.length === 0 && conflicts.blocking.length === 0)) {
    return;
  }

  const indicator = document.createElement('span');
  indicator.className = 'priority-conflict-indicator';
  indicator.textContent = '⚠️';
  indicator.title = 'Click to see priority conflicts';

  // Add click handler
  indicator.addEventListener('click', (e) => {
    e.stopPropagation();
    showConflictDetails(indicator, initiative, conflicts);
  });

  // Prepend to name cell
  nameCell.insertBefore(indicator, nameCell.firstChild);
}
```

### 4. User Interaction

**Tooltip Content Generation:**

```javascript
function showConflictDetails(element, initiative, conflicts) {
  let messages = [];

  // Check if blocking higher priority initiatives
  if (conflicts.blocking.length > 0) {
    const names = conflicts.blocking
      .map(id => getInitiativeName(id))
      .join(', ');
    messages.push(`Blocking: ${names}`);
  }

  // Check if blocked by lower priority initiatives
  if (conflicts.blockedBy.length > 0) {
    const names = conflicts.blockedBy
      .map(id => getInitiativeName(id))
      .join(', ');
    messages.push(`Blocked by: ${names}`);
  }

  const message = messages.join(' | ');

  // Use existing tooltip system (click-triggered)
  attachTooltip(element, message, { triggerOnClick: true });
}

function getInitiativeName(initiativeId) {
  const initiative = appState.initiatives.find(i => i.id === initiativeId);
  return initiative ? initiative.name : 'Unknown Initiative';
}
```

**Tooltip Behavior:**
- Click ⚠️ to show (not hover)
- Click outside or click again to dismiss
- Reuse existing `attachTooltip()` function with click trigger option

### 5. Edge Cases

**Initiative Both Blocks and Is Blocked:**
- Show both messages separated by ` | `
- Example: "Blocking: Initiative A | Blocked by: Initiative B"

**No Active Teams (All N/A):**
- Use existing `getActiveTeamStates()` to exclude N/A teams
- If no active teams, no conflicts possible

**Empty or Single Initiative:**
- Return empty conflict map
- No indicators shown

**Performance:**
- Algorithm complexity: O(n²) where n = number of initiatives
- Acceptable for typical usage (< 50 initiatives)
- Can optimize later if needed

**Long Initiative Names:**
- Tooltip will wrap naturally
- Existing tooltip system handles long content

## Implementation Notes

### Files to Modify

- `quarterly-planner.html` - All changes in single file

### Functions to Add

- `analyzePriorityConflicts()` - Main conflict detection
- `getInitiativeStatus(initiative)` - Helper
- `getCommittedTeams(initiative)` - Helper
- `getUncommittedTeams(initiative)` - Helper
- `hasTeamOverlap(array1, array2)` - Helper
- `addConflictIndicator(nameCell, initiative, conflictMap)` - Visual component
- `showConflictDetails(element, initiative, conflicts)` - Interaction
- `getInitiativeName(initiativeId)` - Helper

### Functions to Modify

- `render()` - Call conflict analysis, pass map to render functions
- `renderInitiativesSummary()` - Accept and pass conflictMap parameter
- `renderNextSection()` - Accept conflictMap, add indicators
- `renderQuarterSection()` - Accept conflictMap, add indicators

### CSS to Add

- `.priority-conflict-indicator` - Warning icon styling

## Testing Scenarios

1. **High priority blocked, low priority progressing**
   - Initiative #1: Blocked, needs Backend Team
   - Initiative #3: Can Proceed, Backend Team committed
   - Expected: Both show conflict indicators

2. **High priority can-proceed, low priority fully committed**
   - Initiative #1: Can Proceed (Frontend committed), needs Backend Team
   - Initiative #4: All teams committed, Backend Team included
   - Expected: Both show conflict indicators

3. **Multiple conflicts on single initiative**
   - Initiative #1: Blocked, needs Backend + Frontend
   - Initiative #3: Can Proceed, Backend committed
   - Initiative #5: Can Proceed, Frontend committed
   - Expected: #1 shows "Blocked by 2 lower priority initiatives"

4. **No conflicts (normal case)**
   - All high priority initiatives fully committed
   - Or low priority initiatives don't have overlapping teams
   - Expected: No indicators shown

5. **Reordering changes conflicts**
   - Drag #3 above #1
   - Expected: Conflict indicators update/disappear

6. **Team state changes resolve conflicts**
   - Backend Team moves from #3 to #1
   - Expected: Conflict indicators disappear

## Success Criteria

- ⚠️ indicators appear next to initiative names when priority inversions exist
- Clicking indicator shows list of conflicting initiatives
- Indicators appear in both Next and Quarter Commitment sections
- Indicators update automatically when data changes (reorder, team state changes)
- No performance degradation with typical initiative counts (< 50)
- Code is readable, maintainable, and follows existing patterns

## Future Enhancements

- Option to sort/filter by conflicts (show conflicts first)
- Visual connection lines between conflicting initiatives
- Conflict resolution suggestions ("Move Backend Team from #3 to #1")
- Summary count in section headers ("Next (3 conflicts)")
- Color-coded severity (red for high priority blocked, yellow for can-proceed)
