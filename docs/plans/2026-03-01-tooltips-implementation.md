# Tooltip System Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a comprehensive tooltip system to make the Quarterly Planning Tracker self-descriptive for new users.

**Architecture:** Implement three types of tooltips: (1) Info icon tooltips with custom popovers for complex concepts, (2) Native title attribute tooltips for simple elements, (3) Visual indicator tooltips for color-coded elements. Pure CSS + vanilla JavaScript, zero dependencies, fully accessible.

**Tech Stack:** HTML5, CSS3, Vanilla JavaScript, ARIA attributes

---

## Task 1: Add CSS for Info Icon and Tooltip Styling

**Files:**
- Modify: `quarterly-planner.html:441-442` (after existing styles, before `</style>`)

**Step 1: Add info icon CSS styles**

Add these styles in the `<style>` section, after line 441 (before `</style>`):

```css
/* === TOOLTIP SYSTEM === */

.info-icon {
    display: inline-block;
    font-size: 14px;
    color: #95a5a6;
    margin-left: 6px;
    cursor: pointer;
    user-select: none;
    transition: color 0.2s;
    position: relative;
}

.info-icon:hover {
    color: #2c3e50;
}

.info-icon:focus {
    outline: 2px solid #3498db;
    outline-offset: 2px;
    border-radius: 50%;
    color: #2c3e50;
}

.tooltip {
    display: none;
    position: absolute;
    background: #2c3e50;
    color: #ffffff;
    padding: 12px 16px;
    border-radius: 6px;
    max-width: 300px;
    font-size: 13px;
    line-height: 1.5;
    box-shadow: 0 4px 12px rgba(0,0,0,0.15);
    z-index: 1000;
    white-space: normal;
    text-align: left;
    font-weight: normal;
    left: 50%;
    transform: translateX(-50%);
    top: calc(100% + 10px);
    opacity: 0;
    transition: opacity 0.2s ease-in;
}

.tooltip.visible {
    display: block;
    opacity: 1;
}

.tooltip::before {
    content: '';
    position: absolute;
    bottom: 100%;
    left: 50%;
    transform: translateX(-50%);
    border: 6px solid transparent;
    border-bottom-color: #2c3e50;
}

.tooltip-compact {
    max-width: 200px;
    font-size: 12px;
    padding: 8px 12px;
}
```

**Step 2: Verify CSS renders correctly**

Open `quarterly-planner.html` in browser, inspect element styles.
Expected: No visual changes yet (CSS added but no HTML using it)

**Step 3: Commit CSS changes**

```bash
git add quarterly-planner.html
git commit -m "feat: add tooltip system CSS styling

- Add .info-icon styles (gray, hover dark, focus outline)
- Add .tooltip popover styles (dark background, white text)
- Add .tooltip.visible for show/hide state
- Add .tooltip::before for arrow pointer
- Add .tooltip-compact for visual indicators

Part of tooltip system implementation"
```

---

## Task 2: Add Tooltip Helper Function in JavaScript

**Files:**
- Modify: `quarterly-planner.html:570-572` (in HELPER FUNCTIONS section, after formatStateName)

**Step 1: Add createInfoIcon helper function**

Add this function after `formatStateName()` function (around line 570):

```javascript
/**
 * Create an info icon with tooltip
 * @param {string} tooltipText - The tooltip content
 * @param {boolean} compact - Use compact styling for visual indicators
 * @returns {HTMLElement} The info icon element with tooltip
 */
function createInfoIcon(tooltipText, compact = false) {
    const icon = document.createElement('span');
    icon.className = 'info-icon';
    icon.tabIndex = 0;
    icon.textContent = 'ⓘ';
    icon.setAttribute('aria-label', 'More information');

    const tooltip = document.createElement('span');
    tooltip.className = compact ? 'tooltip tooltip-compact' : 'tooltip';
    tooltip.setAttribute('role', 'tooltip');
    tooltip.textContent = tooltipText;

    icon.appendChild(tooltip);
    return icon;
}
```

**Step 2: Test helper function in browser console**

Open browser console and run:
```javascript
const icon = createInfoIcon('Test tooltip');
document.body.appendChild(icon);
```
Expected: Info icon appears with ⓘ symbol

**Step 3: Commit helper function**

```bash
git add quarterly-planner.html
git commit -m "feat: add createInfoIcon helper function

Creates info icon (ⓘ) with tooltip element:
- Accepts tooltip text and optional compact styling
- Sets up accessibility attributes (tabIndex, aria-label, role)
- Returns complete icon+tooltip element

Part of tooltip system implementation"
```

---

## Task 3: Add Tooltip Event Handling System

**Files:**
- Modify: `quarterly-planner.html:920-925` (in INITIALIZATION section, before initializeApp)

**Step 1: Add tooltip interaction handlers**

Add these functions before `initializeApp()`:

```javascript
// === TOOLTIP INTERACTION ===

let currentOpenTooltip = null;
let tooltipHoverTimeout = null;

/**
 * Show a tooltip
 * @param {HTMLElement} icon - The info icon element
 */
function showTooltip(icon) {
    const tooltip = icon.querySelector('.tooltip');
    if (!tooltip) return;

    // Close any currently open tooltip
    hideAllTooltips();

    // Show this tooltip
    tooltip.classList.add('visible');
    currentOpenTooltip = tooltip;
}

/**
 * Hide a specific tooltip
 * @param {HTMLElement} icon - The info icon element
 */
function hideTooltip(icon) {
    const tooltip = icon.querySelector('.tooltip');
    if (tooltip) {
        tooltip.classList.remove('visible');
        if (currentOpenTooltip === tooltip) {
            currentOpenTooltip = null;
        }
    }
}

/**
 * Hide all open tooltips
 */
function hideAllTooltips() {
    if (currentOpenTooltip) {
        currentOpenTooltip.classList.remove('visible');
        currentOpenTooltip = null;
    }
    if (tooltipHoverTimeout) {
        clearTimeout(tooltipHoverTimeout);
        tooltipHoverTimeout = null;
    }
}

/**
 * Toggle tooltip visibility
 * @param {HTMLElement} icon - The info icon element
 */
function toggleTooltip(icon) {
    const tooltip = icon.querySelector('.tooltip');
    if (tooltip && tooltip.classList.contains('visible')) {
        hideTooltip(icon);
    } else {
        showTooltip(icon);
    }
}

/**
 * Initialize tooltip event handlers
 */
function initializeTooltips() {
    // Click handler for toggling tooltips
    document.addEventListener('click', (event) => {
        const icon = event.target.closest('.info-icon');
        if (icon) {
            event.stopPropagation();
            toggleTooltip(icon);
        } else {
            // Click outside - close all tooltips
            hideAllTooltips();
        }
    });

    // Keyboard handler for ESC key
    document.addEventListener('keydown', (event) => {
        if (event.key === 'Escape') {
            hideAllTooltips();
        }
    });

    // Hover handlers for info icons
    document.addEventListener('mouseenter', (event) => {
        const icon = event.target.closest('.info-icon');
        if (icon) {
            tooltipHoverTimeout = setTimeout(() => {
                showTooltip(icon);
            }, 300);
        }
    }, true);

    document.addEventListener('mouseleave', (event) => {
        const icon = event.target.closest('.info-icon');
        if (icon && tooltipHoverTimeout) {
            clearTimeout(tooltipHoverTimeout);
            tooltipHoverTimeout = null;
        }
    }, true);

    // Keyboard navigation (Enter/Space to toggle)
    document.addEventListener('keydown', (event) => {
        if (event.target.classList.contains('info-icon')) {
            if (event.key === 'Enter' || event.key === ' ') {
                event.preventDefault();
                toggleTooltip(event.target);
            }
        }
    });

    // Focus/blur handlers
    document.addEventListener('focus', (event) => {
        if (event.target.classList.contains('info-icon')) {
            // Focus shows tooltip after delay (like hover)
            tooltipHoverTimeout = setTimeout(() => {
                showTooltip(event.target);
            }, 300);
        }
    }, true);

    document.addEventListener('blur', (event) => {
        if (event.target.classList.contains('info-icon')) {
            if (tooltipHoverTimeout) {
                clearTimeout(tooltipHoverTimeout);
                tooltipHoverTimeout = null;
            }
            hideTooltip(event.target);
        }
    }, true);
}
```

**Step 2: Call initializeTooltips in initializeApp**

Modify `initializeApp()` function (around line 930) to call `initializeTooltips()`:

```javascript
function initializeApp() {
    const hasStoredData = loadFromLocalStorage();
    if (!hasStoredData) {
        initializeSampleData();
        saveToLocalStorage();
    }
    initializeEventListeners();
    initializeTooltips(); // Add this line
    render();
}
```

**Step 3: Test tooltip interactions**

Open browser, test:
- Click info icon → tooltip appears
- Click outside → tooltip disappears
- Hover icon → tooltip appears after 300ms
- Press ESC → tooltip disappears

**Step 4: Commit tooltip interaction system**

```bash
git add quarterly-planner.html
git commit -m "feat: add tooltip interaction and event handling

Implements complete tooltip behavior:
- Click to toggle tooltips (show/hide)
- Hover shows tooltip after 300ms delay
- Click outside closes all tooltips
- ESC key closes tooltips
- Keyboard navigation (Enter/Space to toggle)
- Focus/blur handlers for accessibility
- Only one tooltip visible at a time

Part of tooltip system implementation"
```

---

## Task 4: Add Tooltips to Section Headers

**Files:**
- Modify: `quarterly-planner.html:464-505` (HTML section headers)

**Step 1: Add tooltip to "Initiatives and Team Commitment" header**

Modify the Initiatives section header (around line 465):

```html
<section class="initiative-section">
    <h2>Initiatives and Team Commitment</h2>
    <div id="initiativeList"></div>
    <button id="addInitiativeBtn" class="add-btn">+ Add Initiative</button>
</section>
```

Change to add info icon inline with JavaScript in render function. We'll do this in the next step instead to make it dynamic.

Actually, for section headers, we need to add them dynamically since they're static HTML. Let's create a helper to add tooltips to existing elements.

**Step 2: Add helper function to attach tooltip to existing element**

Add this helper function after `createInfoIcon()`:

```javascript
/**
 * Attach an info icon with tooltip to an existing element
 * @param {HTMLElement} element - The element to attach tooltip to
 * @param {string} tooltipText - The tooltip content
 */
function attachTooltip(element, tooltipText) {
    const icon = createInfoIcon(tooltipText);
    element.appendChild(icon);
}
```

**Step 3: Add function to initialize section header tooltips**

Add this function in the INITIALIZATION section:

```javascript
/**
 * Initialize tooltips for section headers
 */
function initializeSectionHeaderTooltips() {
    // Initiatives and Team Commitment
    const initiativeHeader = document.querySelector('.initiative-section h2');
    attachTooltip(initiativeHeader, 'Manage your quarterly initiatives and track which teams are involved. Drag team tags between columns to update their commitment status. The colored left border shows overall initiative status: green (all teams committed), yellow (partially committed), red (not started).');

    // Team Summary
    const teamSummaryHeader = document.querySelector('.team-summary-section h2');
    attachTooltip(teamSummaryHeader, 'Shows the current workload for each team. The number (e.g., "3/5") indicates committed initiatives versus your maximum threshold. Green headers mean the team has capacity, red means they\'re at or over capacity. Only committed and uncommitted initiatives are shown here.');

    // Next section
    const nextHeader = document.querySelector('.next-section h2');
    attachTooltip(nextHeader, 'Initiatives that still have uncommitted teams. These are being discussed but not yet fully committed. The status shows whether work can proceed (some teams committed) or is blocked (no teams committed).');
}
```

**Step 4: Call initializeSectionHeaderTooltips in initializeApp**

Modify `initializeApp()`:

```javascript
function initializeApp() {
    const hasStoredData = loadFromLocalStorage();
    if (!hasStoredData) {
        initializeSampleData();
        saveToLocalStorage();
    }
    initializeEventListeners();
    initializeTooltips();
    initializeSectionHeaderTooltips(); // Add this line
    render();
}
```

**Step 5: Test section header tooltips**

Open browser and verify:
- All three section headers have ⓘ icons
- Clicking/hovering shows tooltips
- Tooltip text is correct

**Step 6: Commit section header tooltips**

```bash
git add quarterly-planner.html
git commit -m "feat: add tooltips to section headers

Added info icon tooltips to:
- Initiatives and Team Commitment section
- Team Summary section
- Next section

Tooltips explain the purpose of each section.

Part of tooltip system implementation"
```

---

## Task 5: Add Dynamic Tooltip to Quarter Commitment Section

**Files:**
- Modify: `quarterly-planner.html:1330-1335` (renderInitiativesSummary function)

**Step 1: Modify renderInitiativesSummary to add tooltip**

In the `renderInitiativesSummary()` function, modify the code that updates the quarter section title (around line 1330):

Change:
```javascript
function renderInitiativesSummary(teamColorCache) {
    // Update quarter section title
    const quarterTitle = document.getElementById('quarterSectionTitle');
    quarterTitle.textContent = appState.quarter + ' Commitment';
```

To:
```javascript
function renderInitiativesSummary(teamColorCache) {
    // Update quarter section title with tooltip
    const quarterTitle = document.getElementById('quarterSectionTitle');
    quarterTitle.textContent = appState.quarter + ' Commitment';

    // Remove existing tooltip if any
    const existingIcon = quarterTitle.querySelector('.info-icon');
    if (existingIcon) {
        existingIcon.remove();
    }

    // Add tooltip
    attachTooltip(quarterTitle, 'Initiatives where all teams have committed or completed their work. These are the confirmed deliverables for this quarter. Add target completion dates in the ETA column.');
```

**Step 2: Test quarter section tooltip**

Open browser and verify:
- Quarter Commitment header has ⓘ icon
- Tooltip shows correct text
- Tooltip updates when quarter name changes

**Step 3: Commit quarter section tooltip**

```bash
git add quarterly-planner.html
git commit -m "feat: add dynamic tooltip to Quarter Commitment section

- Adds info icon tooltip to quarter section header
- Tooltip updates when quarter name changes
- Removes old icon before adding new one to prevent duplicates

Part of tooltip system implementation"
```

---

## Task 6: Add Tooltips to State Column Headers

**Files:**
- Modify: `quarterly-planner.html:1004-1012` (createStateColumn function)

**Step 1: Add tooltip to state column header**

Modify the `createStateColumn()` function to add tooltip to header:

Change:
```javascript
function createStateColumn(state, initiative, teamColorCache) {
    const col = document.createElement('div');
    col.className = 'state-column';
    col.dataset.state = state;
    col.dataset.initiativeId = initiative.id;

    const header = document.createElement('div');
    header.className = 'state-column-header';
    header.textContent = formatStateName(state);
    col.appendChild(header);
```

To:
```javascript
function createStateColumn(state, initiative, teamColorCache) {
    const col = document.createElement('div');
    col.className = 'state-column';
    col.dataset.state = state;
    col.dataset.initiativeId = initiative.id;

    const header = document.createElement('div');
    header.className = 'state-column-header';
    header.textContent = formatStateName(state);

    // Add tooltip based on state
    const tooltips = {
        [TEAM_STATES.UNCOMMITTED]: 'Teams that are being asked to participate but haven\'t committed yet. Drag teams here during initial planning discussions.',
        [TEAM_STATES.COMMITTED]: 'Teams that have committed to work on this initiative this quarter. This counts toward their capacity.',
        [TEAM_STATES.COMPLETED]: 'Teams that have finished their work on this initiative. Completed work doesn\'t count toward capacity.',
        [TEAM_STATES.NOT_APPLICABLE]: 'Teams that are not involved in this initiative at all. These teams are excluded from capacity calculations and commitment status.'
    };

    if (tooltips[state]) {
        attachTooltip(header, tooltips[state]);
    }

    col.appendChild(header);
```

**Step 2: Test state column tooltips**

Open browser and verify:
- All four state columns (Uncommitted, Committed, Completed, Not Participating) have ⓘ icons
- Tooltips show correct explanations
- Tooltips work on all initiative rows

**Step 3: Commit state column tooltips**

```bash
git add quarterly-planner.html
git commit -m "feat: add tooltips to state column headers

Added info icon tooltips to all state columns:
- Uncommitted: explains teams being asked
- Committed: explains committed teams and capacity
- Completed: explains finished work
- Not Participating: explains excluded teams

Tooltips appear on all initiative rows.

Part of tooltip system implementation"
```

---

## Task 7: Add Tooltips to Header Controls

**Files:**
- Modify: `quarterly-planner.html:449-460` (header controls HTML)

**Step 1: Update header controls HTML to include data attributes**

We'll add tooltips to header controls dynamically. Modify the header controls section (around line 449):

Change the header controls to add IDs for easier targeting:

```html
<div class="header-controls">
    <label id="quarterLabel">
        Quarter: <input type="text" id="quarterInput" placeholder="Q1 2026" aria-label="Quarter name">
    </label>
    <label id="maxInitiativesLabel">
        Max Initiatives per Team: <input type="number" id="maxInitiativesInput" min="1" value="2" aria-label="Maximum initiatives per team threshold">
    </label>
    <button id="exportBtn" aria-label="Export data as JSON file">Export JSON</button>
    <button id="importBtn" aria-label="Import data from JSON file">Import JSON</button>
    <input type="file" id="importFile" accept=".json" style="display: none;" aria-label="Select JSON file to import">
    <button id="clearBtn" aria-label="Clear all data and reset to sample data">Clear All Data</button>
</div>
```

**Step 2: Add function to initialize header control tooltips**

Add this function in the INITIALIZATION section:

```javascript
/**
 * Initialize tooltips for header controls
 */
function initializeHeaderControlTooltips() {
    // Quarter input
    const quarterLabel = document.getElementById('quarterLabel');
    attachTooltip(quarterLabel, 'The planning period (e.g., "Q1 2026"). This appears in section headers and ETA suggestions. Click to edit.');

    // Max Initiatives input
    const maxInitiativesLabel = document.getElementById('maxInitiativesLabel');
    attachTooltip(maxInitiativesLabel, 'The maximum number of committed initiatives before a team is considered overcommitted. Used to highlight capacity issues in the Team Summary.');

    // Export button
    const exportBtn = document.getElementById('exportBtn');
    attachTooltip(exportBtn, 'Download your current planning data as a JSON file. Use this to share with stakeholders or back up your work.');

    // Import button
    const importBtn = document.getElementById('importBtn');
    attachTooltip(importBtn, 'Load a previously exported planning file. This will replace all current data.');

    // Clear button
    const clearBtn = document.getElementById('clearBtn');
    attachTooltip(clearBtn, 'Reset the application to sample data. This will erase all your current initiatives and team commitments.');
}
```

**Step 3: Call initializeHeaderControlTooltips in initializeApp**

Modify `initializeApp()`:

```javascript
function initializeApp() {
    const hasStoredData = loadFromLocalStorage();
    if (!hasStoredData) {
        initializeSampleData();
        saveToLocalStorage();
    }
    initializeEventListeners();
    initializeTooltips();
    initializeSectionHeaderTooltips();
    initializeHeaderControlTooltips(); // Add this line
    render();
}
```

**Step 4: Test header control tooltips**

Open browser and verify:
- Quarter label has ⓘ icon
- Max Initiatives label has ⓘ icon
- All three buttons (Export, Import, Clear) have ⓘ icons
- Tooltips show correct text

**Step 5: Commit header control tooltips**

```bash
git add quarterly-planner.html
git commit -m "feat: add tooltips to header controls

Added info icon tooltips to all header controls:
- Quarter input: explains planning period
- Max Initiatives input: explains capacity threshold
- Export JSON button: explains data download
- Import JSON button: explains data load
- Clear All Data button: explains reset

Part of tooltip system implementation"
```

---

## Task 8: Add Native Title Tooltips to Simple Elements

**Files:**
- Modify: `quarterly-planner.html:1052-1060` (createInitiativeHeader function)
- Modify: `quarterly-planner.html:1118-1125` (createTeamTag function)

**Step 1: Add title attribute to drag handle**

In `createInitiativeHeader()` function, modify the grip element:

Change:
```javascript
const grip = document.createElement('div');
grip.className = 'initiative-grip-col';
grip.innerHTML = '⋮⋮';
grip.draggable = true;
grip.setAttribute('aria-label', 'Drag to reorder initiative');
```

To:
```javascript
const grip = document.createElement('div');
grip.className = 'initiative-grip-col';
grip.innerHTML = '⋮⋮';
grip.draggable = true;
grip.setAttribute('aria-label', 'Drag to reorder initiative');
grip.title = 'Drag to reorder initiatives'; // Add this line
```

**Step 2: Add title attribute to delete button**

In `createInitiativeHeader()` function, modify the delete button:

Change:
```javascript
const deleteBtn = document.createElement('button');
deleteBtn.className = 'initiative-delete-btn';
deleteBtn.innerHTML = '×';
deleteBtn.title = 'Delete initiative';
deleteBtn.setAttribute('aria-label', `Delete initiative: ${initiative.name}`);
```

To (already has title, just verify):
```javascript
const deleteBtn = document.createElement('button');
deleteBtn.className = 'initiative-delete-btn';
deleteBtn.innerHTML = '×';
deleteBtn.title = 'Delete this initiative'; // Update text
deleteBtn.setAttribute('aria-label', `Delete initiative: ${initiative.name}`);
```

**Step 3: Add title attribute to team tags**

In `createTeamTag()` function, add title attribute:

Change:
```javascript
function createTeamTag(team, initiativeId, teamColorCache = null) {
    const tag = document.createElement('div');
    tag.className = 'team-tag';
    tag.textContent = team.name;
    tag.draggable = true;
    tag.tabIndex = 0;
    tag.dataset.teamId = team.id;
    tag.dataset.initiativeId = initiativeId;
    tag.style.backgroundColor = getTeamColor(team.id, teamColorCache);
```

To:
```javascript
function createTeamTag(team, initiativeId, teamColorCache = null) {
    const tag = document.createElement('div');
    tag.className = 'team-tag';
    tag.textContent = team.name;
    tag.draggable = true;
    tag.tabIndex = 0;
    tag.dataset.teamId = team.id;
    tag.dataset.initiativeId = initiativeId;
    tag.style.backgroundColor = getTeamColor(team.id, teamColorCache);
    tag.title = 'Drag to change commitment status'; // Add this line
```

**Step 4: Test native tooltips**

Open browser and hover over:
- Drag handles (⋮⋮) → shows "Drag to reorder initiatives"
- Delete buttons (×) → shows "Delete this initiative"
- Team tags → shows "Drag to change commitment status"

**Step 5: Commit native title tooltips**

```bash
git add quarterly-planner.html
git commit -m "feat: add native title tooltips to interactive elements

Added browser-native tooltips using title attribute:
- Drag handles: 'Drag to reorder initiatives'
- Delete buttons: 'Delete this initiative'
- Team tags: 'Drag to change commitment status'

Simple tooltips for straightforward interactions.

Part of tooltip system implementation"
```

---

## Task 9: Add Visual Indicator Tooltips (Colored Borders and Badges)

**Files:**
- Modify: `quarterly-planner.html:1015-1020` (createInitiativeRow function)
- Modify: `quarterly-planner.html:1415-1420` (createTeamColumn function)

**Step 1: Add tooltip to initiative colored border**

In `createInitiativeRow()` function, modify the row element to add title:

Change:
```javascript
function createInitiativeRowContainer(initiative) {
    const row = document.createElement('div');
    row.className = 'initiative-row';
    row.dataset.initiativeId = initiative.id;

    // Set colored left border based on status
    const statusColor = getInitiativeStatusColor(initiative);
    row.style.borderLeft = `5px solid ${statusColor}`;
```

To:
```javascript
function createInitiativeRowContainer(initiative) {
    const row = document.createElement('div');
    row.className = 'initiative-row';
    row.dataset.initiativeId = initiative.id;

    // Set colored left border based on status
    const statusColor = getInitiativeStatusColor(initiative);
    row.style.borderLeft = `5px solid ${statusColor}`;

    // Add tooltip explaining color
    const statusTooltips = {
        [COLORS.GREEN]: '🟢 Green: All teams committed',
        [COLORS.YELLOW]: '🟡 Yellow: Partially committed',
        [COLORS.RED]: '🔴 Red: Not started'
    };
    row.title = statusTooltips[statusColor] || '';
```

**Step 2: Add tooltip to team capacity colors**

In `createTeamColumn()` function, modify the capacity span:

Change:
```javascript
// Capacity count (colored based on capacity)
const capacitySpan = document.createElement('span');
capacitySpan.className = 'team-capacity';
capacitySpan.textContent = `${committedCount}/${appState.maxInitiativesPerTeam}`;
capacitySpan.style.color = getTeamColor(team.id, teamColorCache);
```

To:
```javascript
// Capacity count (colored based on capacity)
const capacitySpan = document.createElement('span');
capacitySpan.className = 'team-capacity';
capacitySpan.textContent = `${committedCount}/${appState.maxInitiativesPerTeam}`;
const teamColor = getTeamColor(team.id, teamColorCache);
capacitySpan.style.color = teamColor;

// Add tooltip explaining color
const capacityTooltips = {
    [COLORS.GREEN]: '🟢 Green: Team has capacity',
    [COLORS.YELLOW]: '🟡 Yellow: Team has some commits',
    [COLORS.RED]: '🔴 Red: Team at/over capacity'
};
capacitySpan.title = capacityTooltips[teamColor] || '';
```

**Step 3: Add tooltips to status badges in Next section**

In `renderNextSection()` function, add titles to status badges:

Change:
```javascript
if (hasCommitted || hasCompleted) {
    statusCell.innerHTML = '<span class="initiative-commitment-badge badge-committed">✓ Can Proceed</span>';
} else {
    statusCell.innerHTML = '<span class="initiative-commitment-badge badge-blocked">⏸ Blocked</span>';
}
```

To:
```javascript
if (hasCommitted || hasCompleted) {
    statusCell.innerHTML = '<span class="initiative-commitment-badge badge-committed" title="Some teams are committed, work can start">✓ Can Proceed</span>';
} else {
    statusCell.innerHTML = '<span class="initiative-commitment-badge badge-blocked" title="No teams committed yet">⏸ Blocked</span>';
}
```

**Step 4: Test visual indicator tooltips**

Open browser and hover over:
- Initiative colored borders → shows color meaning
- Team capacity numbers → shows color meaning
- Status badges in Next section → shows badge meaning

**Step 5: Commit visual indicator tooltips**

```bash
git add quarterly-planner.html
git commit -m "feat: add tooltips to visual indicators

Added hover tooltips to color-coded elements:
- Initiative colored borders: green/yellow/red status
- Team capacity numbers: green/yellow/red capacity
- Status badges: Can Proceed vs Blocked

Uses native title attribute for simple hover tooltips.

Part of tooltip system implementation"
```

---

## Task 10: Test Complete Tooltip System and Fix Any Issues

**Files:**
- Modify: `quarterly-planner.html` (various locations if bugs found)

**Step 1: Comprehensive manual testing**

Test all tooltip types:
- [ ] Section headers (3 tooltips)
- [ ] Quarter Commitment header (dynamic)
- [ ] State columns (4 tooltips per initiative)
- [ ] Header controls (5 tooltips)
- [ ] Drag handles (native)
- [ ] Delete buttons (native)
- [ ] Team tags (native)
- [ ] Initiative borders (hover)
- [ ] Capacity numbers (hover)
- [ ] Status badges (hover)

**Step 2: Test interactions**

Test behaviors:
- [ ] Click info icon → tooltip appears
- [ ] Click outside → tooltip disappears
- [ ] Hover icon 300ms → tooltip appears
- [ ] ESC key → tooltip disappears
- [ ] Tab to icon → can focus
- [ ] Enter/Space on icon → tooltip toggles
- [ ] Only one tooltip open at a time

**Step 3: Test accessibility**

Test with keyboard only:
- [ ] Tab through all info icons
- [ ] Focus indicators visible
- [ ] Enter/Space opens tooltips
- [ ] ESC closes tooltips
- [ ] Screen reader announces icons (use NVDA/JAWS if available)

**Step 4: Fix any bugs found**

If bugs are found, fix them and commit:
```bash
git add quarterly-planner.html
git commit -m "fix: [describe the fix]

[Explain what was broken and how it was fixed]"
```

**Step 5: Final commit if all tests pass**

```bash
git add quarterly-planner.html
git commit -m "test: verify complete tooltip system functionality

Tested all tooltip types and interactions:
- All 13 info icon tooltips working
- All native title tooltips working
- All visual indicator tooltips working
- Keyboard navigation functional
- Accessibility features verified
- Single tooltip at a time enforced

Tooltip system implementation complete"
```

---

## Task 11: Update Documentation

**Files:**
- Modify: `README.md:20-44` (Usage section)

**Step 1: Add tooltip information to README**

Add a new subsection in the Usage section:

```markdown
### Understanding the Interface

The application includes helpful tooltips throughout:
- **Info icons (ⓘ)**: Click or hover on these icons next to section headers and controls for detailed explanations
- **Hover tooltips**: Hover over colored indicators and interactive elements for quick help
- **Keyboard navigation**: Press Tab to navigate, Enter/Space to open tooltips, ESC to close

All tooltips are designed to help first-time users understand the app without external documentation.
```

**Step 2: Test documentation clarity**

Read through the updated README and verify it makes sense for new users.

**Step 3: Commit documentation update**

```bash
git add README.md
git commit -m "docs: add tooltip system to README

Added section explaining tooltip system:
- Info icon tooltips for detailed help
- Hover tooltips for quick reference
- Keyboard navigation instructions

Helps users discover and use the tooltip system."
```

---

## Task 12: Final Cleanup and Optimization

**Files:**
- Review: `quarterly-planner.html` (entire file)

**Step 1: Review code for any console.log statements**

Search for debug statements and remove any that were added during development.

**Step 2: Verify no duplicate code**

Check that tooltip initialization functions aren't duplicated or called multiple times unnecessarily.

**Step 3: Test performance**

Open browser DevTools → Performance tab:
- Record while interacting with tooltips
- Verify no performance issues (should be <16ms for 60fps)

**Step 4: Verify file size**

Check file size hasn't grown excessively:
```bash
wc -l quarterly-planner.html
```
Expected: Around 1700-1800 lines (added ~200-300 lines)

**Step 5: Final commit**

```bash
git add quarterly-planner.html
git commit -m "chore: final cleanup of tooltip system

- Removed any debug statements
- Verified no code duplication
- Performance tested and optimized
- File organization maintained

Tooltip system implementation complete and production-ready"
```

---

## Success Criteria

The tooltip system implementation is complete when:

✅ All 13 info icon tooltips are functional
✅ All native title tooltips are present
✅ All visual indicator tooltips work
✅ Click to toggle tooltips works
✅ Hover shows tooltips after 300ms
✅ ESC closes tooltips
✅ Only one tooltip visible at a time
✅ Keyboard navigation works (Tab, Enter, Space)
✅ Accessibility attributes present
✅ Documentation updated
✅ No performance issues
✅ All commits pushed to GitHub

---

## Notes

- All tooltip text is defined in `docs/tooltip-content.md` for easy editing
- Tooltips use pure CSS and vanilla JavaScript (zero dependencies)
- System is fully accessible (keyboard + screen reader)
- Desktop-focused (mobile not optimized but functional)
- Follows existing code style and organization
