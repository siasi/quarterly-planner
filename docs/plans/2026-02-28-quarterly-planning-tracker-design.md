# Quarterly Planning Tracker - Design Document

**Date:** 2026-02-28
**Status:** Approved

## Overview

A simple single-page web application to track initiatives and team commitments across multiple teams for quarterly planning. The application helps visualize team workload, surface overcommitment, and track which initiatives have full team buy-in.

## Problem Statement

Currently, Jira initiatives don't provide easy visualization of:
- Which initiatives impact which teams
- How many initiatives each team is contributing to
- Whether teams are overcommitted for a quarter
- The commitment status of each team per initiative
- Overall initiative commitment readiness

This application provides a lightweight quarterly planning tool to address these visibility gaps.

## Goals

- Track initiatives and their team commitments for a single quarter
- Visualize team workload and flag overcommitment
- Capture commitment status per team per initiative
- Determine overall initiative commitment (committed when all teams have committed)
- Enable export/import for sharing and archiving
- Keep it simple as an MVP for future extension (Jira integration, database, roles/permissions)

## Non-Goals (Future Enhancements)

- Jira integration
- User authentication and role-based permissions
- Backend database
- Multi-quarter tracking in a single view
- Real-time collaboration

## Technical Approach

**Architecture:** Single self-contained HTML file (Approach 1)
- All HTML structure, CSS styling, and JavaScript in one file
- No build step, no dependencies, no server required
- Portable and shareable via email or file sharing

**Technology Stack:**
- Plain HTML/CSS/JavaScript
- HTML5 Drag and Drop API
- localStorage for auto-save
- File API for export/import

**Rationale:** Maximizes simplicity and portability. Can be opened anywhere, easily shared, and requires zero setup.

## Data Model

```json
{
  "quarter": "Q1 2026",
  "maxInitiativesPerTeam": 5,
  "teams": [
    {"id": "team-1", "name": "Backend Team"},
    {"id": "team-2", "name": "Frontend Team"}
  ],
  "initiatives": [
    {
      "id": "init-1",
      "name": "Customer Dashboard Redesign",
      "teamStates": {
        "team-1": "committed",
        "team-2": "uncommitted"
      }
    }
  ]
}
```

**Entities:**

- **Quarter:** String label (e.g., "Q1 2026")
- **MaxInitiativesPerTeam:** Configurable threshold (default: 5)
- **Teams:** Array of team objects with unique ID and name
- **Initiatives:** Array of initiative objects with unique ID, name, and teamStates
- **TeamStates:** Object mapping team IDs to their state for that initiative

**Team States:**
- `uncommitted` - Team is being asked but hasn't committed yet
- `committed` - Team has committed to work on this initiative
- `completed` - Team has finished their work on this initiative

**Design Decision:** teamStates includes all teams for every initiative. When a new initiative is added, all teams default to "uncommitted" state.

## UI Layout and Components

### Header Section
- Quarter name (inline editable text input)
- Max initiatives threshold (number input)
- Action buttons: Export JSON, Import JSON, Clear All Data

### Initiative List (Primary View - Top)
Display format: Table/grid with columns:

**Columns:**
1. **Initiative Name** - Editable inline, with delete button
2. **Uncommitted** - Team tags in uncommitted state
3. **Committed** - Team tags in committed state
4. **Completed** - Team tags in completed state

**Features:**
- Each team appears as a draggable tag in one of the three state columns
- Drag and drop to move teams between states
- Overall commitment badge per initiative:
  - "✓ All Committed" (green) - all teams are Committed or Completed
  - "⚠ Pending Commitment" (yellow) - at least one team is Uncommitted
- "Add Initiative" button at bottom

### Team Summary Panel (Below)
Display format: Columns layout

**Columns:**
- One column per team
- Column header shows team name and capacity status: "Backend Team (3/5)" in green or "Backend Team (6/5)" in red

**Content per column:**
- List of initiatives where team is "Committed" OR "Uncommitted" (shows the total ask)
- Does NOT show "Completed" initiatives (work already done)
- Visual differentiation:
  - Committed initiatives: Solid colored tags
  - Uncommitted initiatives: Outlined/hollow tags

**Purpose:** Shows total quarterly ask per team and highlights overcommitment.

## Core Functionality

### Adding/Editing Initiatives
- "Add Initiative" button creates new row with all teams in "Uncommitted" state
- Initiative name editable via inline text input
- Delete button removes entire initiative
- Changes auto-save to localStorage

### Managing Team States
- Drag and drop team tags between Uncommitted/Committed/Completed columns
- Uses HTML5 Drag and Drop API
- Visual feedback during drag (highlight valid drop zones)
- Real-time updates to Team Summary Panel and commitment badges

### Team Capacity Management
- Teams are managed by editing the JSON directly (no UI for add/edit/delete teams)
- Rationale: Teams change infrequently, manual JSON editing is acceptable for MVP

### Quarter Settings
- Quarter name: Click to edit inline
- Max threshold: Number input, changes apply immediately to all team warnings

## Calculation Logic

### Team Capacity Warning
- **Count:** Number of initiatives where team is in "Committed" state
- **Compare:** Against maxInitiativesPerTeam threshold
- **Display:** Red indicator if count >= threshold, green otherwise
- **Rationale:** Only committed work counts toward capacity; uncommitted is under discussion, completed is done

### Initiative Commitment Status
- **Logic:** If ALL teams are "Committed" OR "Completed" → show "✓ All Committed" (green)
- **Otherwise:** If ANY team is "Uncommitted" → show "⚠ Pending Commitment" (yellow)
- **Updates:** Real-time as teams are dragged between states

### Team Summary Display
- **Show:** Initiatives where team is "Committed" OR "Uncommitted"
- **Hide:** Initiatives where team is "Completed" (work is done)
- **Visual:** Different styling for committed vs uncommitted tags
- **Purpose:** Display total quarterly ask to each team

## Data Persistence

### Auto-save with localStorage
- Every user action triggers save to browser localStorage
- Key: `quarterly-planning-data`
- Persists between sessions
- No manual save button needed

### Export Functionality
- "Export JSON" button downloads current state
- Filename format: `quarterly-planning-Q1-2026.json` (uses quarter name)
- Uses Blob API and temporary download link
- Exported file contains complete data model

### Import Functionality
- "Import JSON" button opens file picker
- Validates JSON structure before loading
- Shows error message if invalid
- Warns user before replacing current data
- After import, saves to localStorage

### Clear All Data
- Button to reset to empty state
- Confirmation dialog before clearing
- Useful for starting new quarter

### Initial State
- If localStorage is empty (first visit), load sample data:
  - Example quarter: "Q1 2026"
  - 3-4 sample teams
  - 2-3 sample initiatives with varied states
- Helps users understand the interface immediately

## User Workflow

### Quarterly Planning Meeting
1. Open the application (loads from localStorage or shows sample data)
2. Edit quarter name to current planning period
3. Adjust max initiatives threshold if needed
4. Add initiatives for the quarter
5. Drag teams through states as planning discussion progresses:
   - Start: All teams "Uncommitted"
   - Discussion: Move to "Committed" as teams agree
   - Review: Check Team Summary Panel for overcommitment
6. Review which initiatives have "✓ All Committed" status
7. Export JSON to share with stakeholders or archive

### Tracking During Quarter
1. Import previous quarter's JSON file
2. Move teams to "Completed" as work finishes
3. Team Summary shows current obligations (Committed + Uncommitted)
4. Export updated state periodically

### Starting New Quarter
1. Either import previous quarter and modify, or Clear All Data
2. Add new initiatives
3. Repeat planning workflow

## Visual Design Principles

- **Clean and minimal:** Focus on content, not decoration
- **Color coding:** Green (good), Yellow (pending), Red (overcommitted)
- **Immediate feedback:** All changes update UI in real-time
- **Drag-and-drop:** Intuitive for state management
- **Inline editing:** No modals, everything editable in place

## Future Extensions

When ready to scale beyond MVP:

1. **Database backend:** Replace localStorage with API calls
2. **Jira integration:** Auto-sync initiatives and team assignments
3. **Authentication:** User roles and permissions
4. **Multi-quarter view:** Track multiple quarters in one session
5. **Advanced reporting:** Charts, burndown, velocity metrics
6. **Team management UI:** Add/edit/delete teams without JSON editing
7. **Collaboration:** Real-time multi-user editing

## Success Criteria

The application succeeds if it:
- Surfaces team overcommitment during quarterly planning
- Makes initiative commitment status obvious at a glance
- Enables easy export/import for sharing across the organization
- Requires minimal training (intuitive drag-and-drop interface)
- Loads and runs without any setup or dependencies
