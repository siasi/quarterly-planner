# Development Guidelines for This Project

## Developer Preferences

### Development Process
- **Structured approach**: Brainstorming → Design → Implementation Plan → Task-by-task execution with reviews
- **Incremental progress**: Prefer completing one task fully before moving to the next
- **User testing**: Developer actively tests changes and provides specific feedback
- **Iterative refinement**: Willing to try multiple approaches until the solution works correctly

### Code Quality & Review
- **Test thoroughly**: Issues found during testing must be fixed before proceeding
- **Spec compliance**: All implementations should match approved specifications exactly
- **Content separation**: Keep editable content (like tooltips) in separate markdown files for easy editing
- **Real validation**: Don't assume changes work - developer will verify in browser

### UX & Design Philosophy
- **User-centric**: Features should make the app self-descriptive for first-time users
- **Accessibility matters**: Keyboard navigation, screen readers, proper ARIA attributes
- **Simplicity over complexity**: Remove conflicting features rather than adding complex workarounds
- **Visual clarity**: Proper spacing, readable text, clear visual indicators

### Problem-Solving Approach
- **Pragmatic**: If solution doesn't work after 2-3 attempts, try a fundamentally different approach
- **Debugging**: Browser DevTools inspection is acceptable for troubleshooting
- **Communication**: Ask for clarification when unclear ("what specifically is uppercase?")
- **Rollback willingness**: Okay to revert changes if they solve the wrong problem

### Technical Preferences
- **Zero dependencies**: Pure HTML/CSS/JavaScript, no frameworks
- **Single-file app**: Keep it simple with inline styles and scripts
- **Browser compatibility**: Test in modern browsers, desktop-focused
- **Git hygiene**: Clear, descriptive commit messages with Co-Authored-By tags

### Documentation
- **README matters**: Keep documentation current as features are added
- **Design docs**: Create and get approval on design documents before implementation
- **Content files**: Separate files for editable content (tooltip-content.md)
- **Plan files**: Detailed implementation plans with task breakdown

## Common Patterns

### When Issues Are Found
1. User reports specific issue with clear description
2. Ask clarifying questions if needed
3. Implement fix
4. User tests in browser with hard refresh
5. Iterate until issue is resolved

### For New Features
1. Brainstorm and explore approaches
2. Create design document for approval
3. Create implementation plan
4. Execute task by task with reviews
5. Test thoroughly
6. Update documentation

## Anti-Patterns to Avoid
- ❌ Don't assume CSS changes work without hard refresh verification
- ❌ Don't over-engineer solutions when simple approaches exist
- ❌ Don't skip testing steps
- ❌ Don't make changes to things not requested (e.g., removing uppercase when only tooltip text should change)
- ❌ Don't add features proactively without explicit request

## Notes
- Developer uses both hover and click for tooltips - ensure both work
- Developer tests at page edges - viewport positioning is critical
- Developer edits content files directly - keep content in editable markdown
- Developer values clarity over brevity in documentation
