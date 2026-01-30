# CLAUDE.md - AI Assistant Guide

## Project Overview

**Preventive Maintenance Log** - A single-page web application for Helena Agri-Enterprises to track and manage agricultural equipment maintenance.

- **Type**: Single-file SPA (Single Page Application)
- **Stack**: HTML5, CSS3, Vanilla JavaScript (ES6+), Firebase Firestore
- **Purpose**: Equipment maintenance tracking, scheduling, and issue management

## Architecture

### Single-File Structure

This project uses a monolithic architecture where all code resides in `index.html`:

```
/maintenance-tracker/
└── index.html (~2,500 lines)
    ├── HTML structure (lines 1-1214)
    │   ├── Header with branding
    │   ├── Navigation tabs (5 tabs)
    │   ├── Tab content sections
    │   └── Modal dialogs
    ├── CSS styles (lines 7-900+)
    │   └── Embedded in <style> tag
    └── JavaScript (lines 1215-end)
        └── ES6 module in <script type="module">
```

### Firebase Backend

The application uses Firebase Firestore for data persistence. Configuration is embedded in the JavaScript section (lines 1219-1226).

**Collections**:
- `equipment` - Equipment inventory with maintenance items
- `maintenanceTemplates` - Reusable maintenance task templates
- `schedules` - Scheduled maintenance events
- `issues` - Equipment problems/issues tracking

## Key Features / Tabs

1. **Equipment** (default) - Equipment cards with status indicators, filtering by type
2. **Dashboard** - Statistics overview (Past Due, Up to Date, Out of Service, Minor Issues)
3. **Templates** - Maintenance template management by equipment type
4. **Schedule** - Interactive calendar for maintenance scheduling
5. **Issues** - Issue logging with severity levels (Critical/Minor)

## Code Conventions

### CSS

- Uses CSS custom properties (variables) in `:root` for theming
- BEM-like naming for classes (e.g., `.equipment-card`, `.equipment-header`)
- Mobile-responsive design with flexbox and CSS grid

**Brand Colors** (defined in `:root`):
```css
--helena-green: #215530      /* Primary */
--helena-green-light: #2d6e3f
--helena-green-dark: #1a4426
--accent-gold: #c5a562       /* Accent */
--accent-light: #e8dcc4
```

**Status Colors**:
```css
--success: #28a745
--warning: #ffc107
--danger: #dc3545
```

### JavaScript

- ES6 modules with Firebase SDK imports from CDN
- Global state arrays: `equipment`, `templates`, `schedules`, `issues`
- Functions exposed to `window` object for HTML onclick handlers
- Async/await pattern for Firebase operations

**Key Patterns**:
```javascript
// Data loading pattern
async function loadData() {
  const snapshot = await getDocs(collection(db, 'collectionName'));
  data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
  renderFunction();
}

// Form submission pattern
window.functionName = async function(event) {
  event.preventDefault();
  // ... gather form data
  await addDoc(collection(db, 'collectionName'), data);
  await loadData();
  closeModal('modalId');
}
```

### HTML

- Tab navigation with `data-tab` attributes
- Modal dialogs with `.active` class toggling
- Form inputs with semantic IDs (e.g., `equipmentId`, `equipmentName`)

## Firebase Operations

**Available operations** (imported from Firebase SDK):
- `getDocs(collection(db, name))` - Fetch all documents
- `addDoc(collection(db, name), data)` - Create document
- `updateDoc(doc(db, name, id), data)` - Update document
- `deleteDoc(doc(db, name, id))` - Delete document

## Data Models

### Equipment
```javascript
{
  id: string,           // Firestore document ID
  equipmentId: string,  // User-defined equipment ID
  name: string,
  type: string,         // Equipment type category
  location: string,
  maintenanceItems: [{
    id: number,
    name: string,
    interval: number,   // Days between maintenance
    description: string,
    lastCompleted: string | null,
    history: []
  }],
  createdAt: string     // ISO date string
}
```

### Maintenance Template
```javascript
{
  id: string,
  equipmentType: string,
  name: string,
  interval: number,
  description: string,
  createdAt: string
}
```

### Schedule
```javascript
{
  id: string,
  equipmentId: string,
  scheduledDate: string,
  notes: string
}
```

### Issue
```javascript
{
  id: string,
  equipmentId: string,
  title: string,
  severity: 'Critical' | 'Minor',
  notes: string,
  completed: boolean,
  createdAt: string
}
```

## Maintenance Status Logic

Status calculation (`getMaintenanceStatus` function):
- **danger** (Past Due): `daysUntil < 0` or `lastCompleted === null`
- **warning** (Coming Due): `daysUntil <= 7`
- **good** (On Schedule): `daysUntil > 7`

## Development Workflow

### Making Changes

1. All changes are made directly to `index.html`
2. No build process required - static file deployment
3. Test locally by opening `index.html` in a browser
4. Firebase backend is live - changes affect production data

### Common Modification Areas

| Task | Location in index.html |
|------|------------------------|
| Add CSS styles | `<style>` section (lines 7-900+) |
| Modify HTML structure | Before `<script>` tag (lines 1-1214) |
| Add/modify JavaScript | `<script type="module">` section |
| Update Firebase config | Lines 1219-1226 |
| Add new modal | Add HTML before `</body>`, add JS handlers |

### Adding New Features

1. **New Tab**: Add nav button with `data-tab`, add `tab-content` div, add render function
2. **New Modal**: Add modal HTML structure, add open/close functions to `window`
3. **New Collection**: Add to `loadData()`, create render function, add CRUD operations

## External Dependencies

Loaded via CDN (no npm/package.json):
- Firebase SDK v10.7.1 (firebase-app.js, firebase-firestore.js)
- Google Fonts (Roboto, Roboto Condensed)

## Testing

No automated testing framework is configured. Manual testing approach:
1. Open `index.html` in browser
2. Check browser console for errors
3. Test CRUD operations for each collection
4. Verify responsive design at different viewport sizes

## Important Notes for AI Assistants

1. **Single File**: All code is in `index.html` - do not create separate files unless explicitly requested
2. **No Build System**: Changes take effect immediately, no compilation needed
3. **Live Database**: Firebase is production - be careful with data operations
4. **Global Functions**: Use `window.functionName = function()` for onclick handlers
5. **Modal Pattern**: Always pair with `closeModal('modalId')` after successful operations
6. **State Management**: Remember to call `loadData()` after modifying Firestore
7. **CSS Variables**: Use existing CSS custom properties for consistent theming
8. **Error Handling**: Use try/catch with console.error and user alerts for Firebase operations

## Quick Reference

```javascript
// Open a modal
document.getElementById('modalId').classList.add('active');

// Close a modal
closeModal('modalId');

// Reload all data
await loadData();

// Get equipment by ID
const eq = equipment.find(e => e.id === equipmentId);

// Firebase document reference
const docRef = doc(db, 'collectionName', documentId);
```
