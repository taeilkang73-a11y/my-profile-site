# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Start

This is a standalone vanilla JavaScript web application with no build process or dependencies. To run it locally:

1. **Direct browser** (simplest):
   - Open `index.html` directly in a web browser

2. **Local server** (recommended for development):
   ```bash
   python -m http.server 8000
   # or
   python3 -m http.server 8000
   ```
   Then visit `http://localhost:8000`

3. **VS Code Live Server**:
   - Install Live Server extension
   - Right-click `index.html` → "Open with Live Server"

## Architecture Overview

This is a **vanilla JavaScript SPA** with a simple two-module architecture:

### Module Breakdown

**`js/storage.js`** - Data Layer
- `BucketStorage` object handles all persistence via LocalStorage
- Single source of truth: `bucketList` array with items containing `{id, title, completed, createdAt, completedAt}`
- Key methods: `load()`, `save()`, `addItem()`, `updateItem()`, `deleteItem()`, `toggleComplete()`, `getStats()`, `getFilteredList()`
- No side effects; pure data management

**`js/app.js`** - UI Layer  
- `BucketListApp` class manages DOM and user interactions
- Caches DOM elements in constructor for performance
- Event handlers call storage methods, then call `render()` to update UI
- Single app instance stored as global `let app` (initialized on DOMContentLoaded)
- Methods follow pattern: user action → storage mutation → `render()`

**`index.html`** - Markup
- Tailwind CSS via CDN (no build step needed)
- Semantic HTML with `data-` attributes for JS selectors
- Modal and empty state elements pre-rendered but hidden

**`css/styles.css`** - Styling  
- Supplements Tailwind with animations (`slideIn`, `fadeIn`, `scaleIn`)
- Filter button states (`.filter-btn`, `.active`)
- Mobile responsive breakpoint at 640px
- Includes dark mode support via `prefers-color-scheme`

## Data Flow

User interaction → `handleX()` in app.js → `BucketStorage.method()` → `this.render()`

The `render()` method:
1. Calls `updateStats()` to refresh statistics
2. Filters list via `BucketStorage.getFilteredList()`
3. Maps items to HTML strings via `createBucketItemHTML()`
4. Writes to DOM in `bucketListContainer`

## Important Implementation Details

### XSS Prevention
- `escapeHtml()` method sanitizes user input before rendering
- Required because item titles are rendered via `innerHTML` from user input
- Applied to both title display and in onclick handlers when passing title to functions

### Data Persistence
- All changes auto-save to LocalStorage via `BucketStorage.save()`
- ID generation uses `Date.now().toString()` (adequate for single-user app)
- Dates stored as ISO strings for consistency

### Event Binding
- Events bound in `init()` via `addEventListener`
- Inline onclick handlers in dynamically generated HTML for item buttons (toggle, edit, delete)
- Modal click handler checks `e.target === this.editModal` to close on backdrop click

### Filter Logic
- Three states: `'all'` (all items), `'active'` (incomplete), `'completed'` (finished)
- Stored in `this.currentFilter`; UI state tracked via `.active` class on filter buttons

## Common Tasks

**Add a new feature to items** (e.g., priority, tags):
1. Extend item object in `BucketStorage.addItem()` (add the new field)
2. Modify `createBucketItemHTML()` in app.js to display it
3. Add a storage method if mutation is needed (e.g., `updatePriority()`)
4. Bind events in `bindEvents()` if user interaction required

**Modify UI styling**:
- Tailwind classes in HTML for most styling (see `index.html`)
- Animation/state overrides in `styles.css`
- Color theme: Blue (#3b82f6), Green (#10b981), Orange (#f97316), Red (#ef4444)

**Change validation or error handling**:
- Input validation in `handleAdd()` and `handleEditSubmit()` (currently just checks empty string)
- Error handling in storage: `load()` and `save()` have try/catch with console.error fallback

## Notes for Future Development

- **No external dependencies**: Keep it that way. Any new feature should use browser APIs.
- **localStorage quota**: ~5-10MB depending on browser. Not a concern for typical bucket lists, but be aware if adding data exports or large fields.
- **Performance**: App re-renders entire list on every action. Fine for reasonable item counts (<1000), but would need virtualization or incremental updates for scale.
- **Browser compatibility**: Modern browsers only (ES6, LocalStorage, querySelector). IE11 not supported.
- **Accessibility**: Currently minimal ARIA labels. Consider adding if expanding UI.
