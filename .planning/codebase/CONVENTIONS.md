# Coding Conventions

**Analysis Date:** 2026-02-02

## Naming Patterns

**Files:**
- Single HTML file: `workout_plan.html` (no modularization)
- Exported workout files: `[Routine names]_[Month]_[Date].html` (e.g., `Push_day_Jan_31.html`)
- Saved session files: `Workout_YYYY-MM-DD.json`

**Functions:**
- camelCase for all functions: `showTab()`, `loadRoutines()`, `extractExerciseData()`, `renderAnalytics()`
- Descriptive verb-noun pattern: `generateExerciseId()`, `dismissPlateau()`, `detectNewPRs()`, `toggleExerciseHistoryPanel()`
- Init/load functions: `addRoutineButtons()`, `loadHistory()`, `loadRoutines()`
- Render functions: `renderRoutine()`, `renderAllRoutines()`, `renderHistory()`, `renderAnalytics()`
- Toggle/show/close patterns: `toggleHistorySession()`, `showNotification()`, `closeRoutineModal()`

**Variables:**
- camelCase for all variables: `currentExerciseData`, `workoutHistory`, `routines`, `analyticsCache`
- Storage keys: UPPERCASE_WITH_UNDERSCORES: `ROUTINES_STORAGE_KEY`, `HISTORY_STORAGE_KEY`, `PLATEAU_DISMISSALS_KEY`
- Temporary variables: short names in inner functions: `ex`, `s`, `vol`, `e1rm`
- Collections plural: `routines`, `sessions`, `exercises`, `alerts`

**Types/Objects:**
- Objects use SNAKE_CASE keys in JSON: `dateDisplay`, `imageUrl`, `setsReps`, `exerciseId`
- Routine IDs: kebab-case: `push`, `pull`, `legs-core`, `shoulders`
- Data attributes: kebab-case: `data-routine-id`, `data-exercise-name`, `data-timestamp`

**CSS Classes:**
- kebab-case: `.exercise-card`, `.routine-modal-overlay`, `.add-to-routine-btn`, `.analytics-summary`
- Modifier classes: `.active`, `.expanded`, `.open`, `.saved`, `.error`, `.incomplete`, `.empty`
- State classes: `.show`, `.error` (added/removed dynamically)
- Variant classes: `.shoulder-card`, `.highlight` (for styling variants)

## Code Style

**Formatting:**
- No external formatter (no Prettier/ESLint config)
- 4-space indentation in HTML/CSS
- Mixed quote style: single quotes in HTML attributes, template literals in JS
- No semicolon consistency (both present and missing)
- Long lines are common (some exceed 200 characters)

**Linting:**
- No linting tools configured (no .eslintrc, .prettierrc, biome.json)
- Code style varies between sections (some minified, some formatted)

**Spacing:**
- Consistent blank lines between function definitions
- Comments separated with blank lines above (section headers use `// ========...========`)
- Functions grouped by feature with comment blocks:
  - `// ==========================================`
  - `// ROUTINES FUNCTIONALITY`
  - `// ==========================================`

## Import Organization

**Not Applicable:** Single HTML file with inline JavaScript. No import statements or module system.

**Global Scope Variables:**
- Declared at top of script section: `ROUTINES_STORAGE_KEY`, `currentExerciseData`, `routines`, `HISTORY_STORAGE_KEY`, `workoutHistory`, `analyticsCache`, `PLATEAU_DISMISSALS_KEY`, `currentHistoryView`
- All mutable state lives in global scope
- No module pattern or closure-based isolation

## Error Handling

**Patterns:**
- Try-catch for JSON parsing:
  ```javascript
  try {
      routines = JSON.parse(stored);
  } catch (e) {
      console.error('Error loading routines:', e);
      routines = { push: [], pull: [], 'legs-core': [], shoulders: [] };
  }
  ```

- Silent failure fallback:
  ```javascript
  try {
      const stored = localStorage.getItem(HISTORY_STORAGE_KEY);
      if (stored) {
          workoutHistory = JSON.parse(stored);
      }
  } catch(e) {
      workoutHistory = [];
  }
  ```

- User-facing error notifications:
  ```javascript
  showNotification('This exercise is already in this routine!', true);
  showNotification('Invalid file: not a workout session export.', true);
  ```

- Validation checks before operations:
  ```javascript
  if (!currentExerciseData) return;
  const exists = routines[routineId].some(ex => ex.name === currentExerciseData.name);
  if (exists) {
      showNotification('Already in routine!', true);
      return;
  }
  ```

**No explicit error objects:** Errors are caught but not propagated. Failures are logged to console or shown as toasts.

## Logging

**Framework:** `console.error()` only (no logging library)

**Patterns:**
- Used only for JSON parse errors: `console.error('Error loading routines:', e);`
- No info/debug/warn logging
- User feedback via `showNotification(message, isError)` instead of logs

## Comments

**When to Comment:**
- Section headers for major feature blocks (rare)
- Complex logic explanations in analytics functions
- TODO/FIXME markers not found in codebase

**JSDoc/TSDoc:**
- No JSDoc annotations used
- No type hints or parameter documentation
- Comments are minimal and informal

**Example comment styles:**
```javascript
// Storage key for localStorage
const ROUTINES_STORAGE_KEY = 'workout_routines';

// Current exercise being added (stored temporarily when modal opens)
let currentExerciseData = null;

// Sorted newest-first (workoutHistory is already sorted that way)
return sessions;
```

## Function Design

**Size:**
- Small functions: 5-15 lines (e.g., `closeRoutineModal()`, `deleteHistorySession()`)
- Medium functions: 20-80 lines (e.g., `addToRoutine()`, `detectPlateau()`)
- Large functions: 100+ lines (e.g., `exportForIPhone()` with ~1000 lines, `renderAnalytics()` with ~500 lines)
- Mega functions: `renderHistory()`, `detectNewPRs()` perform complex multi-step transformations

**Parameters:**
- Minimal: 0-2 parameters per function
- IDs and names passed as simple strings: `removeFromRoutine(routineId, exerciseId)`
- No destructuring or options objects
- Callbacks passed inline: `.forEach(function(session) { ... })`

**Return Values:**
- Objects returned as structured data: `{ id, name, setsReps, targets, imageUrl }`
- Arrays for collections: `getExerciseSessions()` returns array of session objects
- Null/undefined for missing data: `getExerciseStats()` returns `null` if no sessions
- No explicit return (undefined) for side-effect functions: `saveRoutines()`, `showNotification()`

## Module Design

**Exports:**
- No exports (single HTML file)
- All functions available in global scope
- Functions called directly: `showTab('tab-shoulder')`, `addToRoutine('push')`

**Scope:**
- Global variables for state: `routines`, `workoutHistory`, `analyticsCache`
- No data hiding or encapsulation
- Event listeners attached to DOM elements directly

## DOM Manipulation

**Pattern:**
- Direct DOM access via `document.getElementById()`, `document.querySelector()`
- innerHTML for bulk rendering: `container.innerHTML = html;`
- String concatenation for HTML building:
  ```javascript
  var html = '';
  html += '<div class="history-session-card">';
  html += '<span class="date">' + escapeHtml(dateDisplay) + '</span>';
  ```
- Template literals in newer code sections:
  ```javascript
  btn.innerHTML = `
      <svg>...</svg>
      Add to Routine
  `;
  ```
- Event listeners via `addEventListener()` and inline `onclick` attributes

**HTML Escaping:**
- Custom function `escapeHtml(str)` for HTML entities: `replace(/&/g,'&amp;')` etc.
- Used consistently in user-generated content rendering
- Inline escape function in export: `function esc(s) { ... }`

## Storage

**LocalStorage Keys:**
- `workout_routines` - Stringified routine objects
- `workout_history` - Stringified workout session array
- `plateau_dismissals` - Object mapping exercise names to dismissal timestamps

**JSON Structure Conventions:**
```javascript
// Routine exercise object
{ id, name, setsReps, targets, imageUrl }

// Workout session object (exported)
{
  version: 1,
  appId: "workout-planner-session",
  date: "YYYY-MM-DD",
  dateDisplay: "Mon DD, YYYY",
  timestamp: milliseconds,
  routines: [ { id, name, subtitle, exercises: [...] } ]
}
```

## Conditional Patterns

**Ternary for simple checks:**
```javascript
const exercises = `${exercises.length} exercise${exercises.length !== 1 ? 's' : ''}`;
```

**Guard clauses at function start:**
```javascript
function importWorkoutSession(inputEl) {
    const file = inputEl.files[0];
    if (!file) return;
    // ... proceed
}
```

**Early returns to avoid nesting:**
```javascript
if (!currentExerciseData) return;
const exists = routines[routineId].some(ex => ex.name === currentExerciseData.name);
if (exists) {
    showNotification(...);
    return;
}
```

## Array/Object Operations

**forEach for iteration** (not map/filter unless transformation needed):
```javascript
workoutHistory.forEach(function(session) {
    session.routines.forEach(function(routine) {
        routine.exercises.forEach(function(ex) { ... });
    });
});
```

**filter for removal:**
```javascript
routines[routineId] = routines[routineId].filter(ex => ex.id !== exerciseId);
```

**some/every for existence checks:**
```javascript
const exists = routines[routineId].some(ex => ex.name === currentExerciseData.name);
const allAtTop = completedSets.every(function(s) { return s.reps >= targetMaxReps; });
```

**map for transformation (less common):**
```javascript
const routineNames = session.routines.map(r => r.name).join(', ');
```

---

*Convention analysis: 2026-02-02*
