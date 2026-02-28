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
