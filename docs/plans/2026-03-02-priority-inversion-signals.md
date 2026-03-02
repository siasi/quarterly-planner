# Priority Inversion Signals Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add visual warnings when higher priority initiatives are blocked while lower priority initiatives progress, showing which teams are causing the conflict.

**Architecture:** Pre-compute a conflict map during render that identifies priority inversions (higher priority initiatives with uncommitted teams vs lower priority initiatives with those teams committed). Add clickable ⚠️ indicators to initiative names in Next and Quarter Commitment sections that show conflict details in a tooltip.

**Tech Stack:** Vanilla JavaScript, HTML5, CSS3 (single file app, no frameworks)

---

## Task 1: Add Helper Functions for Conflict Detection

**Files:**
- Modify: `quarterly-planner.html` (add after existing helper functions, around line 730)

**Step 1: Add getInitiativeStatus helper**

Add this function after the `formatStateName()` function:

```javascript
/**
 * Get the status of an initiative (blocked or can-proceed)
 * @param {Object} initiative - The initiative object
 * @returns {string} 'blocked' or 'can-proceed'
 */
function getInitiativeStatus(initiative) {
    const activeStates = getActiveTeamStates(initiative);
    const hasCommitted = activeStates.some(state =>
        state === TEAM_STATES.COMMITTED || state === TEAM_STATES.COMPLETED
    );
    return hasCommitted ? 'can-proceed' : 'blocked';
}
```

**Step 2: Add getCommittedTeams helper**

Add after `getInitiativeStatus`:

```javascript
/**
 * Get array of team IDs that are committed or completed on an initiative
 * @param {Object} initiative - The initiative object
 * @returns {Array<string>} Array of team IDs
 */
function getCommittedTeams(initiative) {
    return Object.keys(initiative.teamStates).filter(teamId => {
        const state = initiative.teamStates[teamId];
        return state === TEAM_STATES.COMMITTED || state === TEAM_STATES.COMPLETED;
    });
}
```

**Step 3: Add getUncommittedTeams helper**

Add after `getCommittedTeams`:

```javascript
/**
 * Get array of team IDs that are uncommitted on an initiative
 * @param {Object} initiative - The initiative object
 * @returns {Array<string>} Array of team IDs
 */
function getUncommittedTeams(initiative) {
    return Object.keys(initiative.teamStates).filter(teamId => {
        const state = initiative.teamStates[teamId];
        return state === TEAM_STATES.UNCOMMITTED;
    });
}
```

**Step 4: Add hasTeamOverlap helper**

Add after `getUncommittedTeams`:

```javascript
/**
 * Check if two arrays have any common elements
 * @param {Array} array1 - First array
 * @param {Array} array2 - Second array
 * @returns {boolean} True if arrays have overlap
 */
function hasTeamOverlap(array1, array2) {
    return array1.some(item => array2.includes(item));
}
```

**Step 5: Add getInitiativeName helper**

Add after `hasTeamOverlap`:

```javascript
/**
 * Get initiative name by ID
 * @param {string} initiativeId - The initiative ID
 * @returns {string} Initiative name or 'Unknown Initiative'
 */
function getInitiativeName(initiativeId) {
    const initiative = appState.initiatives.find(i => i.id === initiativeId);
    return initiative ? initiative.name : 'Unknown Initiative';
}
```

**Step 6: Test helpers in browser console**

Open `quarterly-planner.html` in browser, open DevTools console, run:

```javascript
// Test with sample data
const testInit = appState.initiatives[0];
console.log('Status:', getInitiativeStatus(testInit));
console.log('Committed teams:', getCommittedTeams(testInit));
console.log('Uncommitted teams:', getUncommittedTeams(testInit));
console.log('Overlap test:', hasTeamOverlap(['team-1'], ['team-1', 'team-2'])); // Should be true
console.log('Name:', getInitiativeName(testInit.id));
```

Expected: All functions return valid data without errors

**Step 7: Commit**

```bash
git add quarterly-planner.html
git commit -m "feat: add helper functions for conflict detection

Add getInitiativeStatus, getCommittedTeams, getUncommittedTeams,
hasTeamOverlap, and getInitiativeName helpers to support priority
inversion detection.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 2: Add Main Conflict Analysis Function

**Files:**
- Modify: `quarterly-planner.html` (add after helper functions)

**Step 1: Add analyzePriorityConflicts function**

Add after the helper functions:

```javascript
/**
 * Analyze all initiatives and detect priority inversions
 * Returns a map of conflicts for each initiative
 * @returns {Object} Map of {initiativeId: {blockedBy: [], blocking: []}}
 */
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

**Step 2: Test in browser console**

Open browser, refresh page, open console:

```javascript
// Test conflict detection
const conflictMap = analyzePriorityConflicts();
console.log('Conflict map:', conflictMap);

// Check specific initiative
const firstInit = appState.initiatives[0];
console.log('First initiative conflicts:', conflictMap[firstInit.id]);
```

Expected: Returns conflict map object with blockedBy and blocking arrays for each initiative

**Step 3: Commit**

```bash
git add quarterly-planner.html
git commit -m "feat: add priority conflict analysis function

Implement analyzePriorityConflicts to detect when higher priority
initiatives are blocked while lower priority initiatives progress with
overlapping teams.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 3: Add CSS for Warning Indicator

**Files:**
- Modify: `quarterly-planner.html` (add to `<style>` section)

**Step 1: Add priority-conflict-indicator styles**

Find the CSS section (around line 200-300) and add after the `.tooltip` styles:

```css
.priority-conflict-indicator {
    cursor: pointer;
    margin-right: 6px;
    color: #f39c12;
    font-size: 16px;
    user-select: none;
    display: inline-block;
    transition: color 0.2s;
}

.priority-conflict-indicator:hover {
    color: #e67e22;
}
```

**Step 2: Verify CSS loads correctly**

Open browser, refresh page, open DevTools, check computed styles:
- Look for `.priority-conflict-indicator` in Styles panel

Expected: CSS rules are present and accessible

**Step 3: Commit**

```bash
git add quarterly-planner.html
git commit -m "style: add CSS for priority conflict indicator

Add styling for clickable warning icon that indicates priority
inversions. Orange color signals attention needed.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 4: Add Click-Triggered Tooltip System

**Files:**
- Modify: `quarterly-planner.html` (add after tooltip initialization functions, around line 1240)

**Step 1: Add showConflictTooltip function**

Add after the existing `initializeTooltips()` function:

```javascript
/**
 * Show a click-triggered tooltip with conflict details
 * @param {HTMLElement} element - The element to attach tooltip to
 * @param {string} message - The tooltip message
 */
function showConflictTooltip(element, message) {
    // Remove any existing conflict tooltip
    const existingTooltip = document.querySelector('.conflict-tooltip');
    if (existingTooltip) {
        existingTooltip.remove();
    }

    // Create tooltip element
    const tooltip = document.createElement('div');
    tooltip.className = 'tooltip conflict-tooltip';
    tooltip.setAttribute('role', 'tooltip');
    tooltip.textContent = message;
    tooltip.style.position = 'fixed';
    tooltip.style.display = 'block';
    tooltip.style.visibility = 'visible';
    tooltip.style.opacity = '1';
    tooltip.style.zIndex = '10000';

    // Position tooltip near the clicked element
    const rect = element.getBoundingClientRect();
    tooltip.style.left = rect.left + 'px';
    tooltip.style.top = (rect.bottom + 8) + 'px';

    document.body.appendChild(tooltip);

    // Close on click outside
    setTimeout(() => {
        const closeHandler = (e) => {
            if (!tooltip.contains(e.target) && !element.contains(e.target)) {
                tooltip.remove();
                document.removeEventListener('click', closeHandler);
            }
        };
        document.addEventListener('click', closeHandler);
    }, 100);
}
```

**Step 2: Test tooltip manually**

Open browser console and test:

```javascript
// Create a test button
const testBtn = document.createElement('button');
testBtn.textContent = 'Test Tooltip';
testBtn.style.margin = '20px';
document.body.appendChild(testBtn);

testBtn.addEventListener('click', () => {
    showConflictTooltip(testBtn, 'Test message: Initiative A, Initiative B');
});
```

Click the button and verify:
- Tooltip appears below button
- Clicking outside closes tooltip
- Message displays correctly

Expected: Tooltip shows and dismisses correctly

**Step 3: Remove test button**

In console: `testBtn.remove()`

**Step 4: Commit**

```bash
git add quarterly-planner.html
git commit -m "feat: add click-triggered tooltip for conflicts

Implement showConflictTooltip to display priority conflict details
when user clicks the warning indicator.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 5: Add Conflict Indicator Component

**Files:**
- Modify: `quarterly-planner.html` (add after conflict tooltip function)

**Step 1: Add showConflictDetails function**

Add after `showConflictTooltip`:

```javascript
/**
 * Build and show conflict details message
 * @param {HTMLElement} element - The element to attach tooltip to
 * @param {Object} initiative - The initiative object
 * @param {Object} conflicts - The conflicts object {blockedBy: [], blocking: []}
 */
function showConflictDetails(element, initiative, conflicts) {
    let messages = [];

    // Check if blocking higher priority initiatives
    if (conflicts.blocking && conflicts.blocking.length > 0) {
        const names = conflicts.blocking
            .map(id => getInitiativeName(id))
            .join(', ');
        messages.push(`Blocking: ${names}`);
    }

    // Check if blocked by lower priority initiatives
    if (conflicts.blockedBy && conflicts.blockedBy.length > 0) {
        const names = conflicts.blockedBy
            .map(id => getInitiativeName(id))
            .join(', ');
        messages.push(`Blocked by: ${names}`);
    }

    const message = messages.join(' | ');
    showConflictTooltip(element, message);
}
```

**Step 2: Add addConflictIndicator function**

Add after `showConflictDetails`:

```javascript
/**
 * Add conflict indicator to initiative name cell if conflicts exist
 * @param {HTMLElement} nameCell - The table cell containing initiative name
 * @param {Object} initiative - The initiative object
 * @param {Object} conflictMap - The complete conflict map
 */
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
    indicator.setAttribute('aria-label', 'Priority conflict indicator');

    // Add click handler
    indicator.addEventListener('click', (e) => {
        e.stopPropagation();
        showConflictDetails(indicator, initiative, conflicts);
    });

    // Prepend to name cell (before text)
    nameCell.insertBefore(indicator, nameCell.firstChild);
}
```

**Step 3: Test manually with sample conflict**

Open console and create a test scenario:

```javascript
// Simulate conflicts
const testConflictMap = {
    'init-1': { blockedBy: ['init-2', 'init-3'], blocking: [] },
    'init-2': { blockedBy: [], blocking: ['init-1'] }
};

// Test on a Next section row
const nextRows = document.querySelectorAll('#nextBody tr');
if (nextRows.length > 0) {
    const firstRow = nextRows[0];
    const nameCell = firstRow.querySelector('td:first-child');
    const testInit = appState.initiatives.find(i => i.name === nameCell.textContent);
    if (testInit) {
        addConflictIndicator(nameCell, testInit, testConflictMap);
    }
}
```

Expected: ⚠️ appears before initiative name, clicking shows tooltip with conflict names

**Step 4: Refresh page to clear test**

**Step 5: Commit**

```bash
git add quarterly-planner.html
git commit -m "feat: add conflict indicator component

Implement addConflictIndicator to prepend warning icon to initiative
names and showConflictDetails to generate tooltip message.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 6: Integrate Conflict Map into Render Flow

**Files:**
- Modify: `quarterly-planner.html:1313` (render function)

**Step 1: Update render() to compute conflict map**

Find the `render()` function (around line 1313) and modify:

```javascript
function render() {
    // Calculate team colors once for entire render cycle (performance optimization)
    const teamColorCache = calculateTeamColors();
    const conflictMap = analyzePriorityConflicts(); // Add this line

    updateHeaderControls();
    renderInitiatives(teamColorCache);
    renderInitiativesSummary(teamColorCache, conflictMap); // Add conflictMap parameter
    renderTeamSummary(teamColorCache);
}
```

**Step 2: Update renderInitiativesSummary signature**

Find `renderInitiativesSummary` (around line 1874) and update:

```javascript
function renderInitiativesSummary(teamColorCache, conflictMap) {
    // Update quarter section title with tooltip
    const quarterTitle = document.getElementById('quarterSectionTitle');
    quarterTitle.textContent = appState.quarter + ' Commitment';

    // Remove existing tooltip if any
    const existingIcon = quarterTitle.querySelector('.info-icon');
    if (existingIcon) {
        existingIcon.remove();
    }

    // Add tooltip
    attachTooltip(quarterTitle, 'Initiatives that can make progress in this quarter and that are the next to be completed, as they have commitment from all requiried teams. You can capture an ETA that depends by estimations. Use this view to communicate where you are putting your effort, to get things done, and what you expect will be completed soon.');

    // Split initiatives and render both sections
    const { committed, uncommitted } = splitInitiativesByCommitment();
    renderQuarterSection(committed, teamColorCache, conflictMap); // Add conflictMap
    renderNextSection(uncommitted, teamColorCache, conflictMap); // Add conflictMap
}
```

**Step 3: Verify code compiles**

Refresh browser, check console for errors

Expected: No JavaScript errors, page loads normally

**Step 4: Commit**

```bash
git add quarterly-planner.html
git commit -m "refactor: integrate conflict map into render flow

Update render and renderInitiativesSummary to compute and pass
conflict map to section renderers.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 7: Add Conflict Indicators to Next Section

**Files:**
- Modify: `quarterly-planner.html:1828` (renderNextSection function)

**Step 1: Update renderNextSection signature and add indicators**

Find `renderNextSection` (around line 1828) and modify:

```javascript
function renderNextSection(initiatives, teamColorCache, conflictMap) {
    const nextBody = document.getElementById('nextBody');
    nextBody.innerHTML = '';

    initiatives.forEach(initiative => {
        const row = document.createElement('tr');

        // Initiative name cell
        const nameCell = document.createElement('td');
        nameCell.textContent = initiative.name;

        // Add conflict indicator if conflicts exist
        addConflictIndicator(nameCell, initiative, conflictMap); // Add this line

        row.appendChild(nameCell);

        // Status cell with traffic light system
        const statusCell = document.createElement('td');
        const activeStates = getActiveTeamStates(initiative);
        const hasCommitted = activeStates.some(state => state === TEAM_STATES.COMMITTED);
        const hasCompleted = activeStates.some(state => state === TEAM_STATES.COMPLETED);

        if (hasCommitted || hasCompleted) {
            statusCell.innerHTML = '<span class="initiative-commitment-badge badge-committed" title="Some teams are committed, work can proceed">✓ Can Proceed</span>';
        } else {
            statusCell.innerHTML = '<span class="initiative-commitment-badge badge-blocked" title="No teams committed yet, work is blocked">⏸ Blocked</span>';
        }
        row.appendChild(statusCell);

        // Uncommitted teams cell
        const teamsCell = document.createElement('td');
        const uncommittedTeams = getTeamsInState(initiative, TEAM_STATES.UNCOMMITTED);

        uncommittedTeams.forEach(team => {
            const tag = document.createElement('span');
            tag.className = 'team-tag-summary';
            tag.textContent = team.name;
            tag.style.backgroundColor = getTeamColor(team.id, teamColorCache);
            teamsCell.appendChild(tag);
        });

        row.appendChild(teamsCell);
        nextBody.appendChild(row);
    });
}
```

**Step 2: Test in browser**

1. Open `quarterly-planner.html`
2. Create a test scenario:
   - Drag initiative #3 above initiative #1 (if needed)
   - Set Backend Team to "Uncommitted" on initiative #1
   - Set Backend Team to "Committed" on initiative #3
3. Check Next section

Expected: Initiative #1 shows ⚠️ before name, clicking shows "Blocked by: [initiative #3 name]"

**Step 3: Test tooltip click**

- Click ⚠️ indicator
- Verify tooltip shows correct initiative names
- Click outside to dismiss
- Verify tooltip disappears

**Step 4: Commit**

```bash
git add quarterly-planner.html
git commit -m "feat: add conflict indicators to Next section

Show warning icons for initiatives with priority inversions in the
Next section. Clicking indicators reveals conflicting initiative names.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 8: Add Conflict Indicators to Quarter Commitment Section

**Files:**
- Modify: `quarterly-planner.html:1793` (renderQuarterSection function)

**Step 1: Update renderQuarterSection signature and add indicators**

Find `renderQuarterSection` (around line 1793) and modify:

```javascript
function renderQuarterSection(initiatives, teamColorCache, conflictMap) {
    const quarterBody = document.getElementById('quarterBody');
    quarterBody.innerHTML = '';

    initiatives.forEach(initiative => {
        const row = document.createElement('tr');

        // Initiative name cell
        const nameCell = document.createElement('td');
        nameCell.textContent = initiative.name;

        // Add conflict indicator if conflicts exist
        addConflictIndicator(nameCell, initiative, conflictMap); // Add this line

        row.appendChild(nameCell);

        // Contributing teams cell
        const teamsCell = document.createElement('td');
        const contributingTeams = getActiveContributingTeams(initiative);

        contributingTeams.forEach(team => {
            const tag = document.createElement('span');
            tag.className = 'team-tag-summary';
            tag.textContent = team.name;
            tag.style.backgroundColor = getTeamColor(team.id, teamColorCache);
            teamsCell.appendChild(tag);
        });

        row.appendChild(teamsCell);

        // ETA cell
        const etaCell = document.createElement('td');
        const etaInput = document.createElement('input');
        etaInput.type = 'text';
        etaInput.className = 'eta-input';
        etaInput.value = initiative.eta || '';
        etaInput.placeholder = 'Expected completion';
        etaInput.addEventListener('input', (e) => {
            initiative.eta = e.target.value;
            saveToLocalStorage();
        });
        etaCell.appendChild(etaInput);
        row.appendChild(etaCell);

        quarterBody.appendChild(row);
    });
}
```

**Step 2: Test in browser**

1. Refresh page
2. Find an initiative in Quarter Commitment that is blocking higher priority work
3. Check for ⚠️ indicator
4. Click indicator

Expected: Shows "Blocking: [higher priority initiative names]"

**Step 3: Test multiple scenarios**

Create these test scenarios:
- High priority blocked + low priority progressing → Both show indicators
- High priority can-proceed + low priority fully committed → Both show indicators
- No overlapping teams → No indicators

**Step 4: Commit**

```bash
git add quarterly-planner.html
git commit -m "feat: add conflict indicators to Quarter Commitment

Show warning icons for initiatives blocking higher priority work in
the Quarter Commitment section.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 9: Test Priority Inversion Detection

**Files:**
- Manual testing in browser

**Step 1: Test Scenario 1 - High priority blocked, low priority progressing**

Setup:
1. Clear data and start fresh
2. Add 3 initiatives: "Initiative A", "Initiative B", "Initiative C"
3. Add 1 team: "Backend Team"
4. Set Initiative A (highest priority): Backend Team → Uncommitted
5. Set Initiative C (lowest priority): Backend Team → Committed

Expected Results:
- Next section shows Initiative A with ⚠️
- Quarter Commitment shows Initiative C with ⚠️
- Clicking A's ⚠️ shows: "Blocked by: Initiative C"
- Clicking C's ⚠️ shows: "Blocking: Initiative A"

**Step 2: Test Scenario 2 - Multiple conflicts**

Setup:
1. Add 4 initiatives: "Init 1", "Init 2", "Init 3", "Init 4"
2. Add 2 teams: "Backend", "Frontend"
3. Set Init 1: Both teams Uncommitted
4. Set Init 2: Frontend Uncommitted, Backend Committed
5. Set Init 3: Backend Uncommitted, Frontend Committed
6. Set Init 4: Both teams Committed

Expected Results:
- Init 1 shows: "Blocked by: Init 2, Init 4" (or "Blocked by 2 lower priority initiatives")
- Init 2 shows: "Blocking: Init 1"
- Init 4 shows: "Blocking: Init 1"

**Step 3: Test Scenario 3 - Reordering changes conflicts**

Setup:
1. Start with scenario 1 (A blocked, C progressing)
2. Drag Initiative C above Initiative A (reorder)

Expected Results:
- Conflict indicators disappear (C is now higher priority)

**Step 4: Test Scenario 4 - Team state changes resolve conflicts**

Setup:
1. Start with scenario 1 (A blocked, C progressing)
2. Drag Backend Team from C to A

Expected Results:
- Conflict indicators disappear immediately
- A moves to Quarter Commitment (now has a committed team)

**Step 5: Test edge case - Initiative both blocks and is blocked**

Setup:
1. Add 3 initiatives with 2 teams
2. Init 1: Team A uncommitted, Team B committed
3. Init 2: Team A committed, Team B uncommitted
4. Init 3: Both teams committed

Expected Results:
- Init 2 shows: "Blocking: Init 1 | Blocked by: Init 3"

**Step 6: Document test results**

All scenarios pass? If not, note failures for debugging.

---

## Task 10: Test Edge Cases and Performance

**Files:**
- Manual testing in browser

**Step 1: Test with N/A teams**

Setup:
1. Add initiative with 3 teams
2. Set one team to "N/A"
3. Set another team Uncommitted on this initiative, Committed on lower priority

Expected: Conflict detection ignores N/A team, correctly identifies conflict

**Step 2: Test with single initiative**

Setup:
1. Clear all initiatives
2. Add single initiative

Expected: No conflicts shown, no errors in console

**Step 3: Test with many initiatives (performance)**

Setup:
1. Export current data
2. Manually edit JSON to duplicate initiatives (create 30+ initiatives)
3. Import the modified JSON
4. Observe render performance

Expected: Page renders smoothly, no lag when dragging/dropping teams

**Step 4: Test long initiative names**

Setup:
1. Create initiative with very long name (100+ characters)
2. Create conflict scenario
3. Click ⚠️ to show tooltip

Expected: Tooltip wraps text properly, remains readable

**Step 5: Check console for errors**

Review browser console for any warnings or errors

Expected: No errors, no warnings

---

## Task 11: Update Documentation

**Files:**
- Modify: `README.md`
- Modify: `docs/future-enhancements.md`

**Step 1: Update README.md - add to Color Code section**

Find the "Color Code" section and add after "Color Code for Teams":

```markdown
### Priority Inversion Indicators

⚠️ Warning: Indicates priority conflicts
- **In Next section:** Higher priority initiatives blocked while lower priority initiatives progress
- **In Quarter Commitment:** Lower priority initiatives consuming resources needed by higher priority work
- Click the ⚠️ icon to see which initiatives are involved in the conflict
```

**Step 2: Update README.md - add to Features section**

Add to the Features list:

```markdown
- **Priority inversion warnings** automatically detect when teams are working on lower priority initiatives while higher priority work waits
```

**Step 3: Update future-enhancements.md**

Find Task 8 and mark as completed:

```markdown
### 8. ✅ Priority-Based Visual Signals in Next Section (COMPLETED)
- ~~Add color coding to signal the relationship between initiative priority and status~~
- ~~Show when highest priority initiatives are blocked vs progressing~~
- ~~Show when lower priority initiatives are progressing vs blocked~~
- ~~Consider: Color-coded borders, backgrounds, or status indicators~~
- ~~Goal: Make priority/status conflicts immediately visible~~
- **Implemented:** ⚠️ indicators on initiative names with click-triggered tooltips
- **Design:** Detects when higher priority initiatives need teams that are committed to lower priority work
- **Interaction:** Click indicator to see list of conflicting initiatives
- **Commit:** [commit hash will be added]
```

**Step 4: Verify documentation is accurate**

Read through changes and confirm they match implementation

**Step 5: Commit**

```bash
git add README.md docs/future-enhancements.md
git commit -m "docs: update for priority inversion indicators

Add documentation for new conflict detection feature. Update README
with explanation and future-enhancements with completion status.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 12: Final Integration Test

**Files:**
- Manual testing in browser

**Step 1: Full workflow test**

1. Clear data, start fresh
2. Set up realistic quarterly planning scenario:
   - 5 initiatives
   - 3 teams
   - Mix of priorities and commitments
3. Create at least one priority inversion
4. Verify indicators appear correctly
5. Click indicators and verify tooltip messages
6. Reorder initiatives and verify indicators update
7. Move teams and verify conflicts resolve

**Step 2: Test data persistence**

1. Create conflict scenario
2. Export JSON
3. Refresh page
4. Verify indicators still appear (localStorage)
5. Clear storage
6. Import JSON
7. Verify indicators appear correctly

**Step 3: Verify no regressions**

Test existing features still work:
- Add/delete initiatives
- Drag team tags between columns
- Team Summary shows correct capacity
- Quarter Commitment / Next sections split correctly
- Tooltips on info icons still work
- Export/Import functionality

**Step 4: Cross-browser test** (if possible)

Test in:
- Chrome/Edge
- Firefox
- Safari

Expected: Feature works consistently across browsers

**Step 5: Final review**

- Check for console errors
- Verify all commits have proper messages
- Ensure code follows existing style
- Confirm all documentation is updated

---

## Success Criteria

- [ ] ⚠️ indicators appear when priority inversions exist
- [ ] Clicking indicators shows conflict details in tooltip
- [ ] Indicators appear in both Next and Quarter Commitment sections
- [ ] Indicators update when initiatives reordered
- [ ] Indicators disappear when conflicts resolved
- [ ] No performance issues with typical data (< 50 initiatives)
- [ ] No console errors or warnings
- [ ] Existing features continue working
- [ ] Documentation updated
- [ ] Code follows DRY, YAGNI principles

## Testing Checklist

- [ ] High priority blocked + low priority progressing → shows conflicts
- [ ] Multiple conflicts on one initiative → tooltip lists all
- [ ] Reordering changes conflicts → indicators update
- [ ] Team state changes resolve conflicts → indicators disappear
- [ ] Initiative blocks AND is blocked → shows both messages
- [ ] N/A teams excluded from detection
- [ ] Single initiative → no errors
- [ ] Many initiatives → no performance issues
- [ ] Long initiative names → tooltip wraps properly
- [ ] Data persistence works (localStorage + export/import)
- [ ] No regressions in existing features
