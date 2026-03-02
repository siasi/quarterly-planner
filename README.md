# Quarterly Planning Tracker

A simple single-page web application for tracking team commitments across initiatives during quarterly planning.

## How the page is organised
- The header has controls to setup the context, to import and export the data
- The *Initiatives and Teams Commitment* section is where you operate and where you can capture initiatives priorities (sorting them) and committment per team and per initiatives. 
- The *Teams Summary* page is where you have the view per team with initiatives they committed and they don't. Color code indicate teams that are requested too many contributions (see later color code). 
- The *Commitment* section is where you can see the initiatives that can continue or start in the quarter and that could potentially be completed, based on team commitment. 
- The *Next* section is where you see the initiatives that don't have commitment from all required teams and that completion has to be postponed. 

## Color Code

## Features

- **Drag-and-drop interface** for managing team states (Uncommitted → Committed → Completed)
- **Visual capacity warnings** when teams are overcommitted
- **Initiative commitment tracking** shows which initiatives have full team buy-in
- **Priority inversion warnings** automatically detect when teams are working on lower priority initiatives while higher priority work waits
- **Auto-save** to browser localStorage
- **Export/Import** JSON files for sharing and archiving
- **Zero dependencies** - runs entirely in the browser

### Color Code for Initiatives

Capture the possibility to complete the initiative, based on teams commitment
- 🟢 Green: All teams committed
- 🟡 Yellow: Partially committed
- 🔴 Red: No team committed

### Color Code for Teams

Would flag the risk that a team is requested to much in terms of contribution to multiple initiatives. This is visualised comparing the number of initiatives the team commit, compared to a configurable threshold.
- 🟢 Green: Team has no committment
- 🟡 Yellow: Team has some committment, below threshold
- 🔴 Red: Team has some committment, at/over threshold

### Priority Inversion Indicators

⚠ Warning: Indicates priority conflicts
- **In Next section:** Higher priority initiatives blocked while lower priority initiatives progress
- **In Quarter Commitment:** Lower priority initiatives consuming resources needed by higher priority work
- Hover over the ⚠ icon to see which initiatives are involved in the conflict 

## Getting Started

1. Open `quarterly-planner.html` in any modern web browser
2. The app loads with sample data to demonstrate functionality
3. Start editing for your needs

## Usage

All tooltips (info icons plus hover tooltips) are designed to help first-time users understand the app without external documentation.

### Header Controls

### Managing Initiatives

- **Add Initiative**: Click "+ Add Initiative" button (all teams start as Uncommitted)
- **Edit Name**: Click the initiative name to edit inline
- **Delete**: Click "Delete" button (with confirmation)

Drag team tags between three columns:
- **Uncommitted**: Team is being asked but hasn't committed
- **Committed**: Team has committed to work on this initiative
- **Completed**: Team has finished their work

Changes save automatically (use the local storage).

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
