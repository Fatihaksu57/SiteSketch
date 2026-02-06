# CLAUDE.md - SiteSketch

## Project Overview

SiteSketch ("Trassenaufnahme") is a single-file web application for managing infrastructure survey projects with photo documentation, annotation tools, and geolocation mapping. It targets German-speaking users working in construction planning and route surveying.

The entire application lives in **`SiteSketch 2.5.html`** (~1572 lines). There is no build system, no package manager, no external dependencies, and no backend server.

## Tech Stack

- **Pure HTML5 / CSS3 / JavaScript** (ES6+ classes, no frameworks)
- **Canvas API** for image annotation/drawing and map rendering
- **IndexedDB** for client-side persistence (localStorage fallback)
- **OpenStreetMap Nominatim API** for geocoding
- **OpenStreetMap tile service** for map rendering

## File Structure

```
SiteSketch/
├── SiteSketch 2.5.html   # The entire application (HTML + CSS + JS)
├── README.md              # Minimal readme
└── CLAUDE.md              # This file
```

## Architecture

### Class Organization (in `SiteSketch 2.5.html`)

| Class | Lines | Purpose |
|---|---|---|
| `Database` | ~449-515 | IndexedDB wrapper with localStorage fallback. Object stores: `projects`, `photos` |
| `GeocodingService` | ~518-541 | Nominatim geocoding with rate limiting (1100ms) and caching |
| `MapRenderer` | ~544-669 | Custom OSM tile-based map with Canvas rendering, drag/zoom, touch support |
| `Editor` | ~672-1128 | Photo annotation: drawing tools, undo/redo (50 levels), layer management |
| `App` | ~1131-1565 | Main controller: project CRUD, photo management, tabs, PDF export, JSON import/export |

### Constants & Configuration (lines ~393-446)

- **`SURFACES`**: Surface types (Unbefestigt, Gehwegplatte, Asphalt, Beton, Pflaster, Geschl. Bauweise)
- **`DNS`**: Pipe diameters (DN50, DN100)
- **`TOOL_GROUPS`**: Hierarchical tool definitions (SELECT, TRASSE, INSTALLATIONSROHR, SCHACHT, BOHRUNG, TEXT_CALL_OUT)
- **`TOOLS`**: Flattened lookup map built from `TOOL_GROUPS`

### Data Models

**Project**: `id`, `projectNumber`, `projectName`, `customer`, `status` (offen/in-arbeit/abgeschlossen), `description`, address fields (`street`, `houseNumber`, `postalCode`, `city`, `country`), `creator`, `contacts[]`, `lat`/`lon`, `createdAt`, `updatedAt`

**Photo**: `id`, `projectId`, `name`, `dataUrl` (base64), `thumbnail`, `annotations[]`, `isMapSnapshot`, `mapMetadata`

**Annotation**: `tool`, `points[]` (for lines), `point` (for single points), `text`, `meta` (surface/DN for TRASSE)

**Contact**: `name`, `role`, `phone`, `email`

### View System

No router. Views are toggled via CSS class `.active` on `.view` elements:
- `projectListView` - Project listing with search/filter
- `projectFormView` - Create/edit project form
- `projectDetailView` - Project detail with tabs (Photos, Map, Quantities)
- `editorContainer` - Photo annotation editor (full-screen canvas)

Navigation is managed by `App.showView(viewId)` and `App.goBack()`.

### State Management

- Single global `app` instance (App class)
- All persistent data in IndexedDB (database name: `SiteSketchDB`)
- Temporary state held as instance properties (`currentProject`, `currentPhoto`, `tempContacts`, `editingId`)
- No external state library

### Styling

- CSS custom properties (variables) in `:root` for theming
- Primary color: `#0066FF`
- Mobile-responsive with media queries at `768px` breakpoint
- BEM-like class naming (`.project-card`, `.form-input`, `.status-badge`)

## Development Workflow

### Running the Application

Open `SiteSketch 2.5.html` directly in any modern browser. No server required. No build step.

### Making Changes

All code is in the single HTML file. When editing:

1. **CSS** is in the `<style>` block (lines ~7-390)
2. **HTML** structure is in the `<body>` (lines ~390-390)
3. **JavaScript** starts at `<script>` (line ~391) and contains all classes and logic

### Testing

There is no automated test suite. Test manually in the browser after changes:
- Create/edit/delete projects
- Take/import photos and annotate them
- Test map geocoding and rendering
- Export/import JSON
- Generate PDF reports
- Test on mobile viewports for touch interactions

### Deployment

Serve the HTML file statically or open it as a local file. No build artifacts needed.

## Key Conventions

### JavaScript Patterns

- **Class-based OOP** with ES6 classes and `async/await`
- **Inline event handlers** (`onclick="app.method()"`) for UI actions
- **Direct DOM manipulation** with `innerHTML` and `document.getElementById`
- **Promise-wrapped IndexedDB** operations in the Database class
- **JSON serialization** for undo/redo snapshots
- **Template literals** for HTML rendering in JavaScript

### Naming Conventions

- Tool IDs: `UPPER_SNAKE_CASE` (e.g., `SCHACHT_AZK_NEU`, `BOHRUNG_HAUSEINFUEHRUNG`)
- CSS classes: `kebab-case` (e.g., `project-card-header`, `status-badge`)
- JavaScript: `camelCase` for methods and variables, `PascalCase` for classes
- Database store names: lowercase (`projects`, `photos`)
- LocalStorage keys prefixed with `ss_`

### Error Handling

- `try/catch` around JSON import and external API calls
- Graceful fallback from IndexedDB to localStorage
- User feedback via toast notifications (`app.toast(message, type)`)

### HTML Escaping

Use `App.esc()` method when inserting user-provided text into HTML via template literals to prevent XSS.

## External APIs

| API | Usage | Rate Limit |
|---|---|---|
| Nominatim (OpenStreetMap) | Address geocoding | 1 request per 1.1 seconds (enforced in code) |
| OpenStreetMap Tiles | Map tile images | Standard OSM tile usage policy |

No API keys required. No secrets to manage.

## Browser Requirements

- ES6+ support (classes, async/await, template literals)
- HTML5 Canvas API
- IndexedDB (or localStorage as fallback)
- Fetch API
- Touch events for mobile support

## Language

All UI text is in **German**. Date formatting uses `de-DE` locale. There is no i18n system.

## Important Notes for AI Assistants

- This is a **single-file application**. Do not split it into multiple files unless explicitly asked.
- There are **no dependencies** to install and **no build process** to run.
- The file has a space in its name: `SiteSketch 2.5.html` - quote the path when referencing it in shell commands.
- When modifying the JavaScript, respect the existing class structure (Database, GeocodingService, MapRenderer, Editor, App).
- The global `app` instance is created at the bottom of the script and initialized with `app.init()`.
- Canvas drawing operations in the Editor class use a `scale` factor for responsive rendering - account for this when adding drawing features.
- Map snapshots use the **Haversine formula** for real-world distance calculations on annotations.
