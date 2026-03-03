# Quarterly Planning Tracker

A simple single-page web application for tracking team commitments across initiatives during quarterly planning. It helps define initiatives and the status of initiatives based on team commitment, identify priority reversals and bottlenecks 

## Features

- **Drag-and-drop interface** - for reordering initiatives and managing team states (Uncommitted → Committed → Completed)
- **Priority-based sorting** - all sections list initiatives by priority
- **Color Code** - visual indicators show initiative and team status at a glance
- **Priority inversion warnings** - alerts when teams work on lower priority initiatives while higher priority work waits
- **Capacity warning** - visual warnings when teams are overcommitted
- **Auto-save** to browser localStorage
- **Export/Import** JSON files for sharing and archiving
- **Zero dependencies** - runs entirely in the browser

## Color Code

### Initiative Status (Left Border + Badges)

The app uses a 4-color system to show initiative status:

- 🟢 **Green - Done**: All participating teams have completed their work
- 🔵 **Blue - Could Complete**: Teams are committed, no teams waiting (could finish this quarter)
- 🟡 **Yellow - Can Proceed**: Some teams committed, some still uncommitted (partial progress)
- 🔴 **Red - Blocked**: No teams committed yet, work cannot proceed

**Where you see this:**
- **Initiative left border** (_Initiatives and Team Commitment_ section) - colored border shows status
- **Status badges** (_Next_ and _Quarter Commitment_ sections) - explicit status labels

### Team Capacity (Left Border + Team badges)

The app uses a 4-color system to show capacity issue based on number of committed initiatives vs. threshold:

- 🟢 **Green**: No commitments (ready to contribute)
- 🟡 **Yellow**: Some commitments, below threshold
- 🔴 **Red**: At or over capacity threshold

**Where you see this:**
- **Team left border** (_Team Summary_ section) - colored border shows status
- **Team badges** (_Next_ and _Quarter Commitment_ sections) - explicit status labels

You can configure the threshold in the header. 

### Priority Inversion Warnings

⚠ **Orange warning icons** appear when:
- Higher priority initiatives are blocked or partially committed
- Lower priority initiatives have teams that higher priority initiatives need
- Hover over ⚠ to see which initiatives are in conflict

## Page Organization

1. **Header** - Quarter name, capacity threshold, export/import controls
2. **Initiatives and Team Commitment** - Main workspace to manage initiatives and team states
3. **Quarter Commitment** - Initiatives that could complete this quarter (all teams committed or completed)
4. **Next** - Initiatives that won't complete this quarter (some teams still uncommitted)
5. **Team Summary** - View by team showing their committed initiatives

## Getting Started

1. Open `quarterly-planner.html` in any modern web browser
2. The app loads with sample data to demonstrate functionality
3. Use tooltips (ⓘ icons) to learn about each section
4. Start managing your initiatives

## Basic Usage

### Managing Initiatives

- **Add**: Click "+ Add Initiative" button
- **Reorder**: Drag the ⋮⋮ grip handle to change priority
- **Edit Name**: Click the initiative name to edit inline
- **Delete**: Click the × button

### Managing Team States

Drag team tags between columns in the Initiatives section:

- **Uncommitted**: Team is being asked but hasn't committed
- **Committed**: Team has committed to work on this initiative
- **Completed**: Team has finished their work
- **Not Participating**: Team is not involved in this initiative

Changes save automatically to browser localStorage.

### Managing Teams

Teams are managed by editing the exported JSON file:

1. Click "Export Data" to download JSON
2. Edit the `teams` array (add/remove/rename teams)
3. Click "Import Data" to load the modified JSON

## Data Model

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
      },
      "eta": "End of Q1"
    }
  ]
}
```

**Team States:**
- `uncommitted` - Being asked, not committed
- `committed` - Committed to work this quarter
- `completed` - Work finished
- `not-applicable` - Not involved in this initiative

## Browser Compatibility

Works in all modern browsers that support:
- HTML5 Drag and Drop API
- localStorage
- ES6 JavaScript

Tested in Chrome, Firefox, Safari, and Edge.

## License

MIT
