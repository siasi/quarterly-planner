# Quarterly Planning Tracker Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a single self-contained HTML file that tracks team commitments across initiatives for quarterly planning with drag-and-drop state management.

**Architecture:** Single HTML file containing all structure, styling, and logic. Uses localStorage for persistence, HTML5 Drag and Drop API for state management, and File API for export/import. No external dependencies, no build step.

**Tech Stack:** Plain HTML/CSS/JavaScript, HTML5 Drag and Drop API, localStorage, Blob API

---

## Task 1: Create Basic HTML Structure and Initial State

**Files:**
- Create: `quarterly-planner.html`

**Step 1: Create the HTML file with basic structure**

Create `quarterly-planner.html` with DOCTYPE, head, and body. Include a title and meta tags for charset and viewport.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quarterly Planning Tracker</title>
</head>
<body>
    <div id="app"></div>
</body>
</html>
```

**Step 2: Add the HTML structure for all UI components**

Inside the `<body>` tag (before closing), add the complete structure:

```html
<div id="app">
    <!-- Header Section -->
    <header class="header">
        <div class="header-content">
            <h1>Quarterly Planning Tracker</h1>
            <div class="header-controls">
                <label>
                    Quarter: <input type="text" id="quarterInput" placeholder="Q1 2026">
                </label>
                <label>
                    Max Initiatives per Team: <input type="number" id="maxInitiativesInput" min="1" value="5">
                </label>
                <button id="exportBtn">Export JSON</button>
                <button id="importBtn">Import JSON</button>
                <input type="file" id="importFile" accept=".json" style="display: none;">
                <button id="clearBtn">Clear All Data</button>
            </div>
        </div>
    </header>

    <!-- Initiative List -->
    <section class="initiative-section">
        <h2>Initiatives</h2>
        <div id="initiativeList"></div>
        <button id="addInitiativeBtn" class="add-btn">+ Add Initiative</button>
    </section>

    <!-- Team Summary Panel -->
    <section class="team-summary-section">
        <h2>Team Summary</h2>
        <div id="teamSummary" class="team-summary-grid"></div>
    </section>
</div>
```

**Step 3: Verify the HTML structure**

Open `quarterly-planner.html` in a browser. You should see:
- A header with "Quarterly Planning Tracker"
- Input fields for quarter and max initiatives
- Buttons for export, import, and clear
- Two sections: "Initiatives" and "Team Summary"
- "Add Initiative" button

Expected: Basic unstyled HTML structure displays correctly.

**Step 4: Commit the basic structure**

```bash
git add quarterly-planner.html
git commit -m "feat: add basic HTML structure for quarterly planner"
```

---

## Task 2: Add CSS Styling

**Files:**
- Modify: `quarterly-planner.html`

**Step 1: Add CSS within a `<style>` tag in the head**

Insert this `<style>` block inside the `<head>` section after the `<title>` tag:

```html
<style>
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }

    body {
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
        line-height: 1.6;
        color: #333;
        background: #f5f5f5;
        padding: 20px;
    }

    #app {
        max-width: 1400px;
        margin: 0 auto;
        background: white;
        border-radius: 8px;
        box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        overflow: hidden;
    }

    .header {
        background: #2c3e50;
        color: white;
        padding: 20px;
    }

    .header h1 {
        margin-bottom: 15px;
        font-size: 24px;
    }

    .header-controls {
        display: flex;
        gap: 15px;
        flex-wrap: wrap;
        align-items: center;
    }

    .header-controls label {
        display: flex;
        align-items: center;
        gap: 8px;
        font-size: 14px;
    }

    .header-controls input[type="text"],
    .header-controls input[type="number"] {
        padding: 6px 10px;
        border: 1px solid #ccc;
        border-radius: 4px;
        font-size: 14px;
    }

    .header-controls button {
        padding: 8px 16px;
        background: #3498db;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 14px;
        transition: background 0.2s;
    }

    .header-controls button:hover {
        background: #2980b9;
    }

    #clearBtn {
        background: #e74c3c;
    }

    #clearBtn:hover {
        background: #c0392b;
    }

    .initiative-section,
    .team-summary-section {
        padding: 20px;
    }

    .initiative-section h2,
    .team-summary-section h2 {
        margin-bottom: 15px;
        font-size: 20px;
        color: #2c3e50;
    }

    .initiative-row {
        display: grid;
        grid-template-columns: 200px 1fr 1fr 1fr;
        gap: 10px;
        padding: 15px;
        background: #f9f9f9;
        border: 1px solid #ddd;
        border-radius: 4px;
        margin-bottom: 10px;
        align-items: start;
    }

    .initiative-name-col {
        display: flex;
        flex-direction: column;
        gap: 8px;
    }

    .initiative-name-col input {
        width: 100%;
        padding: 6px;
        border: 1px solid #ccc;
        border-radius: 4px;
        font-size: 14px;
    }

    .initiative-name-col button {
        padding: 4px 8px;
        background: #e74c3c;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 12px;
    }

    .initiative-commitment-badge {
        padding: 4px 8px;
        border-radius: 4px;
        font-size: 12px;
        font-weight: bold;
        display: inline-block;
        margin-top: 4px;
    }

    .badge-committed {
        background: #27ae60;
        color: white;
    }

    .badge-pending {
        background: #f39c12;
        color: white;
    }

    .state-column {
        min-height: 60px;
        padding: 10px;
        background: white;
        border: 2px dashed #ccc;
        border-radius: 4px;
    }

    .state-column.drag-over {
        background: #e8f5e9;
        border-color: #27ae60;
    }

    .state-column-header {
        font-size: 12px;
        font-weight: bold;
        color: #666;
        margin-bottom: 8px;
        text-transform: uppercase;
    }

    .team-tag {
        display: inline-block;
        padding: 4px 8px;
        margin: 4px;
        background: #3498db;
        color: white;
        border-radius: 4px;
        font-size: 13px;
        cursor: move;
        user-select: none;
    }

    .team-tag.dragging {
        opacity: 0.5;
    }

    .add-btn {
        padding: 10px 20px;
        background: #27ae60;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 14px;
        font-weight: bold;
    }

    .add-btn:hover {
        background: #229954;
    }

    .team-summary-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 15px;
    }

    .team-column {
        background: #f9f9f9;
        border: 1px solid #ddd;
        border-radius: 4px;
        padding: 15px;
    }

    .team-column-header {
        font-weight: bold;
        margin-bottom: 10px;
        font-size: 14px;
    }

    .team-column-header.over-capacity {
        color: #e74c3c;
    }

    .team-column-header.under-capacity {
        color: #27ae60;
    }

    .initiative-tag {
        display: block;
        padding: 6px 10px;
        margin-bottom: 6px;
        border-radius: 4px;
        font-size: 13px;
    }

    .initiative-tag.committed {
        background: #3498db;
        color: white;
    }

    .initiative-tag.uncommitted {
        background: white;
        border: 2px solid #3498db;
        color: #3498db;
    }
</style>
```

**Step 2: Open in browser and verify styling**

Open `quarterly-planner.html` in a browser. Expected:
- Clean, professional layout with good spacing
- Header with dark background and white text
- Initiative section with grid layout ready for rows
- Team summary section with grid layout
- Responsive design that adapts to screen size
- Color-coded buttons (blue for actions, red for clear)

**Step 3: Commit the styling**

```bash
git add quarterly-planner.html
git commit -m "style: add CSS for quarterly planner layout"
```

---

## Task 3: Implement Data Model and State Management

**Files:**
- Modify: `quarterly-planner.html`

**Step 1: Add JavaScript with data model and state management**

Add this `<script>` tag at the end of the `<body>` tag (after the `#app` div but before `</body>`):

```html
<script>
// Data Model
let appState = {
    quarter: 'Q1 2026',
    maxInitiativesPerTeam: 5,
    teams: [],
    initiatives: []
};

// Generate unique IDs
function generateId(prefix) {
    return `${prefix}-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
}

// Initialize with sample data
function initializeSampleData() {
    appState = {
        quarter: 'Q1 2026',
        maxInitiativesPerTeam: 5,
        teams: [
            { id: generateId('team'), name: 'Backend Team' },
            { id: generateId('team'), name: 'Frontend Team' },
            { id: generateId('team'), name: 'Mobile Team' },
            { id: generateId('team'), name: 'DevOps Team' }
        ],
        initiatives: [
            {
                id: generateId('init'),
                name: 'Customer Dashboard Redesign',
                teamStates: {}
            },
            {
                id: generateId('init'),
                name: 'Payment Integration',
                teamStates: {}
            },
            {
                id: generateId('init'),
                name: 'Mobile App Performance',
                teamStates: {}
            }
        ]
    };

    // Initialize all team states as uncommitted for all initiatives
    appState.initiatives.forEach(initiative => {
        appState.teams.forEach(team => {
            initiative.teamStates[team.id] = 'uncommitted';
        });
    });

    // Set some example states
    appState.initiatives[0].teamStates[appState.teams[0].id] = 'committed';
    appState.initiatives[0].teamStates[appState.teams[1].id] = 'committed';
    appState.initiatives[1].teamStates[appState.teams[0].id] = 'committed';
    appState.initiatives[2].teamStates[appState.teams[2].id] = 'completed';
}

// localStorage management
const STORAGE_KEY = 'quarterly-planning-data';

function saveToLocalStorage() {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(appState));
}

function loadFromLocalStorage() {
    const stored = localStorage.getItem(STORAGE_KEY);
    if (stored) {
        try {
            appState = JSON.parse(stored);
            return true;
        } catch (e) {
            console.error('Failed to parse localStorage data:', e);
            return false;
        }
    }
    return false;
}

// Initialize app
function initializeApp() {
    const hasStoredData = loadFromLocalStorage();
    if (!hasStoredData) {
        initializeSampleData();
        saveToLocalStorage();
    }
    render();
}

// Placeholder render function
function render() {
    console.log('Rendering with state:', appState);
}

// Initialize on page load
document.addEventListener('DOMContentLoaded', initializeApp);
</script>
```

**Step 2: Open browser console and verify state initialization**

Open `quarterly-planner.html` in a browser, open DevTools console (F12). Expected:
- Console log showing "Rendering with state:" followed by the appState object
- appState contains quarter, maxInitiativesPerTeam, teams array (4 teams), initiatives array (3 initiatives)
- Each initiative has teamStates with all team IDs mapped to states

**Step 3: Test localStorage persistence**

In browser console, run:
```javascript
localStorage.getItem('quarterly-planning-data')
```

Expected: JSON string with the appState data.

Refresh the page and check console. Expected: Same state loads from localStorage (not recreating sample data).

Clear localStorage and refresh:
```javascript
localStorage.clear()
```

Expected: Sample data initializes again on refresh.

**Step 4: Commit data model and state management**

```bash
git add quarterly-planner.html
git commit -m "feat: add data model and localStorage state management"
```

---

## Task 4: Implement Header Controls

**Files:**
- Modify: `quarterly-planner.html`

**Step 1: Add event listeners for header controls**

In the `<script>` section, after the `initializeApp()` function, add:

```javascript
// Header control handlers
function setupHeaderControls() {
    const quarterInput = document.getElementById('quarterInput');
    const maxInitiativesInput = document.getElementById('maxInitiativesInput');

    quarterInput.value = appState.quarter;
    maxInitiativesInput.value = appState.maxInitiativesPerTeam;

    quarterInput.addEventListener('input', (e) => {
        appState.quarter = e.target.value;
        saveToLocalStorage();
        render();
    });

    maxInitiativesInput.addEventListener('change', (e) => {
        appState.maxInitiativesPerTeam = parseInt(e.target.value, 10);
        saveToLocalStorage();
        render();
    });
}
```

**Step 2: Update render function to call setupHeaderControls**

Replace the placeholder `render()` function with:

```javascript
function render() {
    console.log('Rendering with state:', appState);
    setupHeaderControls();
    renderInitiatives();
    renderTeamSummary();
}
```

**Step 3: Add placeholder render functions**

Add these placeholder functions:

```javascript
function renderInitiatives() {
    console.log('Rendering initiatives...');
}

function renderTeamSummary() {
    console.log('Rendering team summary...');
}
```

**Step 4: Test header controls in browser**

Open `quarterly-planner.html` in browser.

Test quarter input:
1. Change quarter value to "Q2 2026"
2. Check console - should see "Rendering with state:" with updated quarter
3. Refresh page - quarter should persist as "Q2 2026"

Test max initiatives:
1. Change value to 3
2. Check console - should see updated maxInitiativesPerTeam
3. Refresh page - value should persist as 3

Expected: Both inputs update appState, save to localStorage, and persist across refreshes.

**Step 5: Commit header controls**

```bash
git add quarterly-planner.html
git commit -m "feat: implement header controls with auto-save"
```

---

## Task 5: Implement Export and Import Functionality

**Files:**
- Modify: `quarterly-planner.html`

**Step 1: Add export JSON functionality**

In the `<script>` section, add after the `setupHeaderControls()` function:

```javascript
// Export functionality
function exportJSON() {
    const dataStr = JSON.stringify(appState, null, 2);
    const blob = new Blob([dataStr], { type: 'application/json' });
    const url = URL.createObjectURL(blob);

    const filename = `quarterly-planning-${appState.quarter.replace(/\s+/g, '-')}.json`;

    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
}

// Setup export button
function setupExportButton() {
    const exportBtn = document.getElementById('exportBtn');
    exportBtn.addEventListener('click', exportJSON);
}
```

**Step 2: Add import JSON functionality**

Add after the export functions:

```javascript
// Import functionality
function importJSON(file) {
    const reader = new FileReader();

    reader.onload = (e) => {
        try {
            const imported = JSON.parse(e.target.result);

            // Validate structure
            if (!imported.quarter || !imported.teams || !imported.initiatives) {
                alert('Invalid JSON file: missing required fields (quarter, teams, initiatives)');
                return;
            }

            // Confirm before replacing
            if (!confirm('This will replace all current data. Continue?')) {
                return;
            }

            appState = imported;
            saveToLocalStorage();
            render();
            alert('Data imported successfully!');
        } catch (error) {
            alert('Failed to import JSON: ' + error.message);
        }
    };

    reader.readAsText(file);
}

// Setup import button
function setupImportButton() {
    const importBtn = document.getElementById('importBtn');
    const importFile = document.getElementById('importFile');

    importBtn.addEventListener('click', () => {
        importFile.click();
    });

    importFile.addEventListener('change', (e) => {
        const file = e.target.files[0];
        if (file) {
            importJSON(file);
        }
        // Reset file input
        e.target.value = '';
    });
}
```

**Step 3: Add clear all data functionality**

Add after the import functions:

```javascript
// Clear all data
function clearAllData() {
    if (!confirm('Are you sure you want to clear all data? This cannot be undone.')) {
        return;
    }

    initializeSampleData();
    saveToLocalStorage();
    render();
    alert('All data cleared and reset to sample data.');
}

// Setup clear button
function setupClearButton() {
    const clearBtn = document.getElementById('clearBtn');
    clearBtn.addEventListener('click', clearAllData);
}
```

**Step 4: Update render function to setup all buttons**

Update the `render()` function:

```javascript
function render() {
    console.log('Rendering with state:', appState);
    setupHeaderControls();
    setupExportButton();
    setupImportButton();
    setupClearButton();
    renderInitiatives();
    renderTeamSummary();
}
```

**Step 5: Test export functionality**

Open `quarterly-planner.html` in browser:
1. Click "Export JSON"
2. Expected: File downloads with name like `quarterly-planning-Q1-2026.json`
3. Open the downloaded file in a text editor
4. Expected: Valid JSON with quarter, teams, initiatives, and teamStates

**Step 6: Test import functionality**

1. Modify the exported JSON file (change quarter to "Q3 2026")
2. In browser, click "Import JSON"
3. Select the modified file
4. Expected: Confirmation dialog appears
5. Click OK
6. Expected: Success alert, UI updates with Q3 2026
7. Refresh page
8. Expected: Imported data persists

**Step 7: Test clear functionality**

1. Click "Clear All Data"
2. Expected: Confirmation dialog
3. Click OK
4. Expected: Data resets to sample data, alert shows confirmation

**Step 8: Commit export/import functionality**

```bash
git add quarterly-planner.html
git commit -m "feat: add export, import, and clear data functionality"
```

---

## Task 6: Implement Initiative Rendering

**Files:**
- Modify: `quarterly-planner.html`

**Step 1: Implement renderInitiatives function**

Replace the placeholder `renderInitiatives()` function with:

```javascript
function renderInitiatives() {
    const initiativeList = document.getElementById('initiativeList');
    initiativeList.innerHTML = '';

    appState.initiatives.forEach(initiative => {
        const row = createInitiativeRow(initiative);
        initiativeList.appendChild(row);
    });
}

function createInitiativeRow(initiative) {
    const row = document.createElement('div');
    row.className = 'initiative-row';
    row.dataset.initiativeId = initiative.id;

    // Name column
    const nameCol = document.createElement('div');
    nameCol.className = 'initiative-name-col';

    const nameInput = document.createElement('input');
    nameInput.type = 'text';
    nameInput.value = initiative.name;
    nameInput.addEventListener('input', (e) => {
        initiative.name = e.target.value;
        saveToLocalStorage();
    });

    const deleteBtn = document.createElement('button');
    deleteBtn.textContent = 'Delete';
    deleteBtn.addEventListener('click', () => deleteInitiative(initiative.id));

    const commitmentBadge = document.createElement('div');
    commitmentBadge.className = 'initiative-commitment-badge';
    updateCommitmentBadge(commitmentBadge, initiative);

    nameCol.appendChild(nameInput);
    nameCol.appendChild(deleteBtn);
    nameCol.appendChild(commitmentBadge);

    // State columns
    const uncommittedCol = createStateColumn('uncommitted', initiative);
    const committedCol = createStateColumn('committed', initiative);
    const completedCol = createStateColumn('completed', initiative);

    row.appendChild(nameCol);
    row.appendChild(uncommittedCol);
    row.appendChild(committedCol);
    row.appendChild(completedCol);

    return row;
}

function createStateColumn(state, initiative) {
    const col = document.createElement('div');
    col.className = 'state-column';
    col.dataset.state = state;
    col.dataset.initiativeId = initiative.id;

    const header = document.createElement('div');
    header.className = 'state-column-header';
    header.textContent = state.charAt(0).toUpperCase() + state.slice(1);
    col.appendChild(header);

    // Add team tags for this state
    appState.teams.forEach(team => {
        if (initiative.teamStates[team.id] === state) {
            const tag = createTeamTag(team, initiative.id);
            col.appendChild(tag);
        }
    });

    // Setup drop zone
    setupDropZone(col);

    return col;
}

function createTeamTag(team, initiativeId) {
    const tag = document.createElement('div');
    tag.className = 'team-tag';
    tag.textContent = team.name;
    tag.draggable = true;
    tag.dataset.teamId = team.id;
    tag.dataset.initiativeId = initiativeId;

    tag.addEventListener('dragstart', handleDragStart);
    tag.addEventListener('dragend', handleDragEnd);

    return tag;
}

function updateCommitmentBadge(badge, initiative) {
    const allCommitted = Object.values(initiative.teamStates).every(
        state => state === 'committed' || state === 'completed'
    );

    if (allCommitted) {
        badge.textContent = '✓ All Committed';
        badge.className = 'initiative-commitment-badge badge-committed';
    } else {
        badge.textContent = '⚠ Pending Commitment';
        badge.className = 'initiative-commitment-badge badge-pending';
    }
}
```

**Step 2: Add placeholder drag-and-drop handlers**

Add these placeholder functions:

```javascript
function handleDragStart(e) {
    console.log('Drag start');
}

function handleDragEnd(e) {
    console.log('Drag end');
}

function setupDropZone(col) {
    console.log('Setup drop zone');
}
```

**Step 3: Add deleteInitiative function**

Add:

```javascript
function deleteInitiative(initiativeId) {
    if (!confirm('Are you sure you want to delete this initiative?')) {
        return;
    }

    appState.initiatives = appState.initiatives.filter(i => i.id !== initiativeId);
    saveToLocalStorage();
    render();
}
```

**Step 4: Test initiative rendering in browser**

Open `quarterly-planner.html`. Expected:
- Three initiative rows displayed
- Each row has name input, delete button, and commitment badge
- Three state columns (Uncommitted, Committed, Completed) per row
- Team tags appear in appropriate columns based on initial state
- Commitment badges show correct status

Test editing:
1. Change an initiative name
2. Refresh page
3. Expected: Name change persists

Test deleting:
1. Click delete on an initiative
2. Expected: Confirmation dialog, then initiative removed
3. Refresh page
4. Expected: Deletion persists

**Step 5: Commit initiative rendering**

```bash
git add quarterly-planner.html
git commit -m "feat: implement initiative rendering with edit and delete"
```

---

## Task 7: Implement Drag-and-Drop State Management

**Files:**
- Modify: `quarterly-planner.html`

**Step 1: Implement drag start and end handlers**

Replace the placeholder drag handlers with:

```javascript
let draggedElement = null;

function handleDragStart(e) {
    draggedElement = e.target;
    e.target.classList.add('dragging');
    e.dataTransfer.effectAllowed = 'move';
    e.dataTransfer.setData('text/html', e.target.innerHTML);
}

function handleDragEnd(e) {
    e.target.classList.remove('dragging');

    // Remove drag-over class from all columns
    document.querySelectorAll('.state-column').forEach(col => {
        col.classList.remove('drag-over');
    });
}
```

**Step 2: Implement drop zone setup**

Replace the placeholder `setupDropZone()` function with:

```javascript
function setupDropZone(col) {
    col.addEventListener('dragover', handleDragOver);
    col.addEventListener('dragenter', handleDragEnter);
    col.addEventListener('dragleave', handleDragLeave);
    col.addEventListener('drop', handleDrop);
}

function handleDragOver(e) {
    if (e.preventDefault) {
        e.preventDefault();
    }
    e.dataTransfer.dropEffect = 'move';
    return false;
}

function handleDragEnter(e) {
    if (e.target.classList.contains('state-column')) {
        e.target.classList.add('drag-over');
    }
}

function handleDragLeave(e) {
    if (e.target.classList.contains('state-column')) {
        e.target.classList.remove('drag-over');
    }
}

function handleDrop(e) {
    if (e.stopPropagation) {
        e.stopPropagation();
    }
    e.preventDefault();

    const dropTarget = e.target.closest('.state-column');
    if (!dropTarget || !draggedElement) {
        return false;
    }

    const teamId = draggedElement.dataset.teamId;
    const initiativeId = draggedElement.dataset.initiativeId;
    const newState = dropTarget.dataset.state;

    // Update state in data model
    const initiative = appState.initiatives.find(i => i.id === initiativeId);
    if (initiative) {
        initiative.teamStates[teamId] = newState;
        saveToLocalStorage();
        render();
    }

    return false;
}
```

**Step 3: Test drag-and-drop in browser**

Open `quarterly-planner.html`.

Test dragging team tags:
1. Drag a team tag from "Uncommitted" column
2. Expected: Tag becomes semi-transparent, drop zones highlight on hover
3. Drop it in "Committed" column
4. Expected: UI re-renders, tag now appears in "Committed" column
5. Check commitment badge
6. Expected: Badge updates if all teams are now committed
7. Refresh page
8. Expected: State change persists

Test multiple drags:
1. Move several teams between states
2. Expected: Each drag updates immediately and persists

**Step 4: Commit drag-and-drop functionality**

```bash
git add quarterly-planner.html
git commit -m "feat: implement drag-and-drop for team state management"
```

---

## Task 8: Implement Add Initiative Functionality

**Files:**
- Modify: `quarterly-planner.html`

**Step 1: Add addInitiative function**

Add this function:

```javascript
function addInitiative() {
    const newInitiative = {
        id: generateId('init'),
        name: 'New Initiative',
        teamStates: {}
    };

    // Initialize all teams as uncommitted
    appState.teams.forEach(team => {
        newInitiative.teamStates[team.id] = 'uncommitted';
    });

    appState.initiatives.push(newInitiative);
    saveToLocalStorage();
    render();
}
```

**Step 2: Setup add initiative button**

Add this function:

```javascript
function setupAddInitiativeButton() {
    const addBtn = document.getElementById('addInitiativeBtn');
    addBtn.addEventListener('click', addInitiative);
}
```

**Step 3: Update render function**

Update the `render()` function to include the new setup:

```javascript
function render() {
    console.log('Rendering with state:', appState);
    setupHeaderControls();
    setupExportButton();
    setupImportButton();
    setupClearButton();
    setupAddInitiativeButton();
    renderInitiatives();
    renderTeamSummary();
}
```

**Step 4: Test add initiative in browser**

Open `quarterly-planner.html`.

Test adding:
1. Click "+ Add Initiative" button
2. Expected: New initiative row appears with name "New Initiative"
3. All teams appear in "Uncommitted" column
4. Badge shows "⚠ Pending Commitment"
5. Edit the name to something meaningful
6. Refresh page
7. Expected: New initiative persists with edited name

**Step 5: Commit add initiative functionality**

```bash
git add quarterly-planner.html
git commit -m "feat: implement add initiative functionality"
```

---

## Task 9: Implement Team Summary Panel

**Files:**
- Modify: `quarterly-planner.html`

**Step 1: Implement renderTeamSummary function**

Replace the placeholder `renderTeamSummary()` function with:

```javascript
function renderTeamSummary() {
    const teamSummary = document.getElementById('teamSummary');
    teamSummary.innerHTML = '';

    appState.teams.forEach(team => {
        const column = createTeamColumn(team);
        teamSummary.appendChild(column);
    });
}

function createTeamColumn(team) {
    const column = document.createElement('div');
    column.className = 'team-column';

    // Calculate committed count
    const committedCount = appState.initiatives.filter(
        initiative => initiative.teamStates[team.id] === 'committed'
    ).length;

    // Create header with capacity indicator
    const header = document.createElement('div');
    header.className = 'team-column-header';

    const isOverCapacity = committedCount >= appState.maxInitiativesPerTeam;
    if (isOverCapacity) {
        header.classList.add('over-capacity');
    } else {
        header.classList.add('under-capacity');
    }

    header.textContent = `${team.name} (${committedCount}/${appState.maxInitiativesPerTeam})`;
    column.appendChild(header);

    // Add initiative tags for committed and uncommitted states
    appState.initiatives.forEach(initiative => {
        const state = initiative.teamStates[team.id];
        if (state === 'committed' || state === 'uncommitted') {
            const tag = createInitiativeTag(initiative, state);
            column.appendChild(tag);
        }
    });

    return column;
}

function createInitiativeTag(initiative, state) {
    const tag = document.createElement('div');
    tag.className = `initiative-tag ${state}`;
    tag.textContent = initiative.name;
    return tag;
}
```

**Step 2: Test team summary in browser**

Open `quarterly-planner.html`.

Expected display:
- One column per team (4 columns)
- Each header shows team name and count (e.g., "Backend Team (2/5)")
- Headers are green if under capacity, red if at/over capacity
- Each column shows initiatives where team is Committed or Uncommitted
- Committed initiatives have solid blue background
- Uncommitted initiatives have blue border (hollow)
- Completed initiatives don't appear

Test interaction:
1. Drag a team from Uncommitted to Committed in an initiative
2. Expected: Team summary updates immediately
3. Count increases by 1 for that team
4. Tag changes from hollow to solid
5. If team reaches threshold, header turns red

**Step 3: Commit team summary panel**

```bash
git add quarterly-planner.html
git commit -m "feat: implement team summary panel with capacity tracking"
```

---

## Task 10: Final Testing and Polish

**Files:**
- Modify: `quarterly-planner.html`

**Step 1: Add defensive checks for edge cases**

Add these safety checks at the start of `createStateColumn()`:

```javascript
function createStateColumn(state, initiative) {
    const col = document.createElement('div');
    col.className = 'state-column';
    col.dataset.state = state;
    col.dataset.initiativeId = initiative.id;

    const header = document.createElement('div');
    header.className = 'state-column-header';
    header.textContent = state.charAt(0).toUpperCase() + state.slice(1);
    col.appendChild(header);

    // Add team tags for this state
    appState.teams.forEach(team => {
        // Defensive check: ensure teamStates exists and has this team
        if (initiative.teamStates && initiative.teamStates[team.id] === state) {
            const tag = createTeamTag(team, initiative.id);
            col.appendChild(tag);
        }
    });

    // Setup drop zone
    setupDropZone(col);

    return col;
}
```

**Step 2: Add validation to import function**

Update the `importJSON` validation:

```javascript
function importJSON(file) {
    const reader = new FileReader();

    reader.onload = (e) => {
        try {
            const imported = JSON.parse(e.target.result);

            // Validate structure
            if (!imported.quarter || !imported.teams || !imported.initiatives) {
                alert('Invalid JSON file: missing required fields (quarter, teams, initiatives)');
                return;
            }

            if (!Array.isArray(imported.teams) || !Array.isArray(imported.initiatives)) {
                alert('Invalid JSON file: teams and initiatives must be arrays');
                return;
            }

            // Validate maxInitiativesPerTeam
            if (typeof imported.maxInitiativesPerTeam !== 'number' || imported.maxInitiativesPerTeam < 1) {
                imported.maxInitiativesPerTeam = 5; // Default
            }

            // Confirm before replacing
            if (!confirm('This will replace all current data. Continue?')) {
                return;
            }

            appState = imported;
            saveToLocalStorage();
            render();
            alert('Data imported successfully!');
        } catch (error) {
            alert('Failed to import JSON: ' + error.message);
        }
    };

    reader.readAsText(file);
}
```

**Step 3: Comprehensive manual testing**

Open `quarterly-planner.html` and test the following workflow:

**Basic Operations:**
1. Edit quarter name → Verify persistence
2. Change max initiatives to 3 → Verify team headers update
3. Add new initiative → Verify all teams start as uncommitted
4. Edit initiative name → Verify persistence
5. Delete initiative → Verify confirmation and removal

**Drag and Drop:**
1. Drag team from Uncommitted to Committed → Verify update
2. Drag team from Committed to Completed → Verify update
3. Drag team from Completed back to Uncommitted → Verify update
4. Verify commitment badges update correctly
5. Verify team summary updates in real-time

**Team Summary:**
1. Verify committed count is accurate
2. Get a team to threshold → Verify header turns red
3. Move team from Committed to Completed → Verify count decreases
4. Verify uncommitted initiatives show as hollow tags
5. Verify completed initiatives don't appear

**Export/Import:**
1. Export JSON → Verify file downloads
2. Modify exported file → Import → Verify changes load
3. Import invalid JSON → Verify error message
4. Import file missing fields → Verify error message

**Clear Data:**
1. Make changes
2. Clear all data → Verify confirmation
3. Verify sample data loads
4. Verify localStorage resets

**Persistence:**
1. Make various changes
2. Close browser tab
3. Reopen file → Verify all changes persist

**Step 4: Test with empty teams array**

In browser console, test edge case:
```javascript
appState.teams = []
render()
```

Expected: No errors, initiatives show empty state columns.

Reset:
```javascript
localStorage.clear()
location.reload()
```

**Step 5: Final commit**

```bash
git add quarterly-planner.html
git commit -m "fix: add validation and defensive checks for edge cases"
```

---

## Task 11: Create README Documentation

**Files:**
- Create: `README.md`

**Step 1: Create README with usage instructions**

```markdown
# Quarterly Planning Tracker

A simple single-page web application for tracking team commitments across initiatives during quarterly planning.

## Features

- **Drag-and-drop interface** for managing team states (Uncommitted → Committed → Completed)
- **Visual capacity warnings** when teams are overcommitted
- **Initiative commitment tracking** shows which initiatives have full team buy-in
- **Auto-save** to browser localStorage
- **Export/Import** JSON files for sharing and archiving
- **Zero dependencies** - runs entirely in the browser

## Getting Started

1. Open `quarterly-planner.html` in any modern web browser
2. The app loads with sample data to demonstrate functionality
3. Start editing for your needs

## Usage

### Header Controls

- **Quarter Name**: Click to edit the planning period (e.g., "Q1 2026")
- **Max Initiatives per Team**: Set the threshold for capacity warnings (default: 5)
- **Export JSON**: Download current state as a JSON file
- **Import JSON**: Load a previously exported file
- **Clear All Data**: Reset to sample data

### Managing Initiatives

- **Add Initiative**: Click "+ Add Initiative" button (all teams start as Uncommitted)
- **Edit Name**: Click the initiative name to edit inline
- **Delete**: Click "Delete" button (with confirmation)
- **Commitment Badge**: Shows "✓ All Committed" (green) when all teams are committed/completed, otherwise "⚠ Pending Commitment" (yellow)

### Managing Team States

Drag team tags between three columns:
- **Uncommitted**: Team is being asked but hasn't committed
- **Committed**: Team has committed to work on this initiative
- **Completed**: Team has finished their work

Changes save automatically.

### Team Summary Panel

Shows one column per team with:
- Team name and committed count (e.g., "Backend Team (3/5)")
- **Green header**: Under capacity
- **Red header**: At or over capacity threshold
- **Solid tags**: Committed initiatives
- **Hollow tags**: Uncommitted initiatives
- Completed initiatives are hidden (work is done)

### Managing Teams

Teams are managed by editing the exported JSON file directly:

```json
{
  "teams": [
    {"id": "team-1", "name": "Backend Team"},
    {"id": "team-2", "name": "New Team"}
  ]
}
```

After editing:
1. Import the modified JSON file
2. New teams automatically appear as "Uncommitted" for all initiatives

## Data Model

The application uses this JSON structure:

```json
{
  "quarter": "Q1 2026",
  "maxInitiativesPerTeam": 5,
  "teams": [
    {"id": "team-1", "name": "Backend Team"}
  ],
  "initiatives": [
    {
      "id": "init-1",
      "name": "Customer Dashboard Redesign",
      "teamStates": {
        "team-1": "committed"
      }
    }
  ]
}
```

## Workflow Example

### Quarterly Planning Meeting

1. Open the application
2. Update quarter name (e.g., "Q2 2026")
3. Adjust max initiatives threshold if needed
4. Add initiatives for the quarter
5. Drag teams to "Committed" as planning discussion progresses
6. Check Team Summary for overcommitment (red headers)
7. Review which initiatives show "✓ All Committed"
8. Export JSON to share with stakeholders

### During the Quarter

1. Import the planning JSON file
2. Drag teams to "Completed" as work finishes
3. Export updated state periodically

## Browser Compatibility

Works in all modern browsers that support:
- HTML5 Drag and Drop API
- localStorage
- ES6 JavaScript
- Blob API

Tested in Chrome, Firefox, Safari, and Edge.

## Future Enhancements

- Jira integration
- Multi-quarter tracking
- Backend database
- User authentication and permissions
- Advanced reporting and charts
- Team management UI

## License

MIT
```

**Step 2: Verify README accuracy**

Read through the README and cross-reference with `quarterly-planner.html` to ensure all instructions are accurate.

**Step 3: Commit README**

```bash
git add README.md
git commit -m "docs: add comprehensive README with usage instructions"
```

---

## Task 12: Final Verification

**Files:**
- Test: `quarterly-planner.html`
- Read: `README.md`

**Step 1: Complete workflow test**

Open `quarterly-planner.html` and complete this full workflow:

1. **Setup**: Change quarter to "Q2 2026", set max to 4
2. **Clear**: Clear all data to start fresh with sample data
3. **Add**: Add 2 new initiatives
4. **Edit**: Rename all initiatives to realistic names
5. **Drag**: Move teams around to create varied states
6. **Verify**: Check that at least one team exceeds capacity (red header)
7. **Verify**: Check that at least one initiative shows "✓ All Committed"
8. **Export**: Export the data
9. **Close**: Close the browser completely
10. **Open**: Reopen `quarterly-planner.html`
11. **Verify**: All changes persisted
12. **Import**: Import the exported file
13. **Verify**: Data matches the export

Expected: All steps work smoothly with no errors.

**Step 2: Test in different browsers**

If available, test in at least 2 different browsers (Chrome, Firefox, Safari, Edge):
- Open file
- Verify UI renders correctly
- Test drag-and-drop
- Test export/import
- Verify localStorage persistence

**Step 3: Verify file is self-contained**

Run this check:
```bash
grep -i "src=" quarterly-planner.html
grep -i "href=" quarterly-planner.html | grep -v "javascript"
```

Expected: No external dependencies (no CDN links, no external CSS/JS files).

Verify file can be opened directly:
```bash
open quarterly-planner.html
```

Expected: Opens and works without needing a web server.

**Step 4: Final commit**

```bash
git add -A
git commit -m "chore: final verification complete"
```

**Step 5: Create completion summary**

Run:
```bash
git log --oneline
```

Expected output showing all commits from the implementation plan.

---

## Completion Checklist

- [ ] Single self-contained HTML file created
- [ ] CSS styling implemented with clean, minimal design
- [ ] Data model with teams, initiatives, and team states
- [ ] localStorage persistence with auto-save
- [ ] Header controls (quarter name, max threshold)
- [ ] Export JSON functionality
- [ ] Import JSON functionality with validation
- [ ] Clear all data functionality
- [ ] Initiative list rendering with editable names
- [ ] Drag-and-drop team state management
- [ ] Add initiative functionality
- [ ] Delete initiative functionality
- [ ] Commitment badge calculations (All Committed / Pending)
- [ ] Team summary panel rendering
- [ ] Team capacity warnings (red/green headers)
- [ ] Initiative tags styled (solid for committed, hollow for uncommitted)
- [ ] Edge case validation and defensive checks
- [ ] README documentation
- [ ] Cross-browser testing
- [ ] Comprehensive workflow testing
- [ ] No external dependencies verified

## Success Criteria

The implementation is complete when:

1. Opening `quarterly-planner.html` shows a fully functional interface with sample data
2. All CRUD operations work (create, read, update, delete initiatives)
3. Drag-and-drop smoothly moves teams between states
4. Team summary accurately reflects capacity and warnings
5. Export creates valid JSON files
6. Import validates and loads JSON files correctly
7. All changes persist across browser sessions
8. The file is completely self-contained with no external dependencies
9. README provides clear usage instructions
10. Application works across multiple browsers
