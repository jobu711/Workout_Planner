# Technology Stack

**Analysis Date:** 2026-02-02

## Languages

**Primary:**
- HTML5 - Main markup language (~1,783 lines in `workout_plan.html`)
- CSS3 - Inline styles in `<style>` block (~1,042 lines, lines 7-1042 in `workout_plan.html`)
- JavaScript (ES6+) - Vanilla JS without transpilation (~2,500+ lines, lines 4830-end in `workout_plan.html`)

**Secondary:**
- JSON - Workout session export/import format (`.json` files in `workouts/` directory)

## Runtime

**Environment:**
- Browser-based (no server-side runtime)
- Target: Modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile: iOS Safari (exported static HTML), Android browsers
- No Node.js or server runtime required

**Package Manager:**
- None - No npm, yarn, or similar package management
- No external dependencies beyond browser APIs
- No build step required

## Frameworks

**Core:**
- No framework (vanilla HTML/CSS/JS only)
- Single-page application (SPA) pattern implemented manually with DOM manipulation

**Testing:**
- No testing framework in place
- No unit test, integration test, or e2e test infrastructure

**Build/Dev:**
- No build tools (no webpack, Vite, Rollup, etc.)
- No transpiler (no Babel, TypeScript compiler, etc.)
- No dev server or hot reload
- No minification or bundling in deployment

## Key Dependencies

**Critical:**
- Browser localStorage API - Persists routines (`workout_routines`), history (`workout_history`), and plateau dismissals (`plateau_dismissals`) across sessions
- FileReader API - Reads imported `.json` workout session files in `importWorkoutSession()` function (line 5222)
- JSON parsing - Core to session import/export (lines 5229, 5245)

**Infrastructure:**
- External image CDN: `liftmanual.com/wp-content/uploads/` - Hotlinked exercise images (300+ image URLs)
- External guide URLs: `liftmanual.com/[exercise-slug]/` - Links to exercise guides
- External video links: Instagram Reels, Facebook shares (e.g., `instagram.com/reel/DLSP_76Au0m/`)
- GymVisual CDN: `gymvisual.com/img/p/` - One exercise image (Sit-ups)

## Configuration

**Environment:**
- No environment variables required
- No config files (`.env`, `.env.local`, etc.)
- App ID hardcoded: `workout-planner-session` (used for validation in `importWorkoutSession()`, line 5231)

**Build:**
- No build configuration files
- Single `workout_plan.html` file is the complete application
- All CSS and JS inlined (no external stylesheets or scripts)

## Platform Requirements

**Development:**
- Text editor (no IDE required)
- Git for version control (`.git/` directory present)
- No build tools or development servers needed
- Browser for testing

**Production:**
- Static file hosting (any web server can serve `.html` files)
- HTTPS recommended (for secure localStorage access on some browsers)
- CDN access to `liftmanual.com` and `gymvisual.com` for exercise images
- iOS: Exported HTML file viewable via Files app or Quick Look (lines 5090-5174 handle iOS export)
- Browser APIs required: localStorage, FileReader, JSON

## Styling Architecture

**CSS Locations:**
- All styles inline in `<style>` block (lines 7-1042 in `workout_plan.html`)
- No CSS preprocessors (no Sass, Less)
- No CSS frameworks (no Bootstrap, Tailwind, etc.)

**Design System:**
- Colors: Theme dark blue/navy (`#1a1a2e`, `#16213e`), orange accent (`#ff6b35`), teal accent (`#4ecdc4`)
- Responsive breakpoints: 768px (mobile layout), 480px (compact spacing)
- CSS Grid for exercise cards layout (min 300px, auto-fit columns)
- Backdrop blur on sticky tab navigation (lines 765-775)

## Data Format

**Local Storage:**
- `workout_routines` - JSON object with routine arrays: `{ push: [], pull: [], 'legs-core': [], shoulders: [] }`
- `workout_history` - JSON array of workout session objects
- `plateau_dismissals` - JSON object tracking dismissed plateau alerts (expires after 7 days)

**Export Format:**
- JSON with schema: `{ version: 1, appId: "workout-planner-session", date: string, dateDisplay: string, timestamp: number, routines: [...] }`
- HTML static file for iOS export (fully self-contained, all scripts and styles inlined)

---

*Stack analysis: 2026-02-02*
