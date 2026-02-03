# Architecture

**Analysis Date:** 2026-02-02

## Pattern Overview

**Overall:** Single-page HTML application (SPA) with embedded CSS/JS and localStorage-based state management.

**Key Characteristics:**
- All code consolidated in one `workout_plan.html` file (~6,280 lines)
- No build tools, frameworks, or external dependencies
- Client-side state persisted via localStorage
- Tabbed interface with progressive enhancement (tab switching, modals)
- Exercise reference data hardcoded in HTML; user data (routines, history) stored in localStorage

## Layers

**Presentation Layer:**
- Purpose: Render UI tabs, exercise cards, forms, modals, analytics views
- Location: `workout_plan.html` lines 1-1780 (CSS), lines 1963-4770 (HTML content)
- Contains: Exercise cards (`.exercise-card`), tab navigation (`.tab-nav`), modal overlays, analytics components
- Depends on: JavaScript DOM manipulation functions
- Used by: User interactions (clicks, form submissions)

**State Management Layer:**
- Purpose: Manage routines, workout history, analytics calculations, dismissal preferences
- Location: `workout_plan.html` lines 4859-4889 (routines), lines 5206-5220 (history), lines 5644-5680 (analytics)
- Contains: `routines` object, `workoutHistory` array, `plateau_dismissals` map
- Depends on: localStorage API
- Used by: All user-facing features (routine building, history tracking, analytics)

**Business Logic Layer:**
- Purpose: Exercise extraction, data validation, calculations, suggestions, PR detection
- Location: `workout_plan.html` lines 4897-5998 (routine operations), lines 5390-5762 (analytics)
- Contains: Functions for extracting exercise data, calculating e1RM, trend detection, plateau detection, PR detection, weight suggestions
- Depends on: State management layer
- Used by: Presentation layer event handlers

**Data Access Layer:**
- Purpose: Read/write to localStorage, parse/serialize JSON
- Location: `workout_plan.html` lines 4873-4889 (routines), lines 5206-5276 (history), lines 6272-6277 (initialization)
- Contains: `loadRoutines()`, `saveRoutines()`, `loadHistory()`, `saveHistory()`, file import/export
- Depends on: Native browser APIs (localStorage, FileReader, Blob)
- Used by: State management layer

## Data Flow

**Add Exercise to Routine:**

1. User clicks "Add to Routine" button on exercise card
2. `openRoutineModal(card)` extracts exercise data via `extractExerciseData(card)`
3. Modal displays routine options with `addToRoutine(routineId)` callbacks
4. User selects routine → `addToRoutine()` validates (duplicate check), creates exercise object with unique ID, pushes to `routines[routineId]`
5. `saveRoutines()` serializes to localStorage
6. `renderRoutine(routineId)` rebuilds DOM
7. `showNotification()` confirms action

**Import Workout Session:**

1. User selects .json file from `workouts/` directory via file input
2. `importWorkoutSession(inputEl)` reads file via FileReader
3. Validates appId and timestamp against existing history
4. Detects new PRs via `detectNewPRs(session)`
5. Saves session to `workoutHistory` array
6. `saveHistory()` persists to localStorage
7. If new PRs detected, `showPRModal()` displays celebration
8. `renderHistory()` or `renderAnalytics()` shows updated data

**View Analytics:**

1. User clicks "Analytics" toggle in History tab
2. `switchHistoryView('analytics')` hides sessions, shows analytics container
3. `renderAnalytics()` calls `getAllExerciseAnalytics()` to aggregate all history data
4. For each exercise: `getExerciseStats()` calculates e1RM, volume, trend, plateau status
5. `renderAnalyticsSummary()`, `renderPlateauAlerts()`, `renderPRHallOfFame()`, `renderExerciseAnalyticsCards()` build nested DOM
6. Sparklines generated via `buildSparkline()`
7. User can expand cards via `toggleAnalyticsCard()` to see per-exercise history panel

**State Management:**

- Routines: `routines` object (JavaScript in-memory) synchronized with localStorage via `loadRoutines()`/`saveRoutines()`
- History: `workoutHistory` array synchronized with localStorage via `loadHistory()`/`saveHistory()`
- Plateau dismissals: stored per exercise name in localStorage `plateau_dismissals`, checked during `detectPlateau()`
- Temporary state: `currentExerciseData` variable holds exercise being added, cleared on modal close

## Key Abstractions

**Exercise Object (Routine):**
- Purpose: Represents an exercise added to a user routine
- Examples: `workout_plan.html` lines 4897-4935 (extractExerciseData), 4953-4980 (addToRoutine)
- Pattern: Plain object with `id`, `name`, `setsReps`, `targets`, `imageUrl` properties
- Unique ID generated via `generateExerciseId()` to track same exercise added multiple times

**Workout Session Object (History Entry):**
- Purpose: Represents a complete workout imported from iPhone app
- Examples: `workouts/push_day_Jan_31.json`
- Pattern: JSON with `version`, `appId`, `date`, `dateDisplay`, `timestamp`, `routines` array
- Each routine contains exercises with `sets` array tracking weight/reps/completion

**Exercise Stats Object (Analytics):**
- Purpose: Aggregated metrics for a single exercise across all history
- Pattern: Generated by `getExerciseStats()` (lines 5572-5643), contains:
  - `sessions`: array of all workout sessions containing exercise
  - `currentWeight`: most recent weight
  - `maxWeight`: PR weight
  - `estimated1RM`: Epley formula calculation
  - `volume`: total reps × weight
  - `trend`: linear regression slope of e1RM over time
  - `plateau`: boolean or plateau type ('stall'/'fatigue'/'improved')
  - `lastSetBest`: best rep count in most recent session

**Analytics Summary Object:**
- Purpose: Top-level stats across all history
- Pattern: Generated by `getAllExerciseAnalytics()` (lines 5644-5674), contains:
  - `sessionCount`: number of imported sessions
  - `exerciseCount`: distinct exercises trained
  - `newPRs`: PRs from most recent import
  - `frequencyMap`: exercises trained per week estimate
  - All exercise stats keyed by name

## Entry Points

**Page Load:**
- Location: `workout_plan.html` lines 6272-6277 (DOMContentLoaded event)
- Triggers: Browser parses and renders HTML
- Responsibilities: `addRoutineButtons()` injects buttons, `loadRoutines()` restores user routines, `loadHistory()` restores sessions, `addHistoryButtons()` injects history view buttons

**Tab Navigation:**
- Location: `workout_plan.html` lines 4831-4852 (showTab function)
- Triggers: User clicks tab button (onclick handler)
- Responsibilities: Toggle active CSS class, show/hide tab content div, smooth scroll on mobile

**Routine Modal:**
- Location: `workout_plan.html` lines 4940-4952 (openRoutineModal)
- Triggers: User clicks "Add to Routine" button on exercise card
- Responsibilities: Scrape exercise data, display modal with routine options, capture selection

**Export (iPhone):**
- Location: `workout_plan.html` lines 4990-5201 (exportForIPhone)
- Triggers: User clicks "Save Workout" button in Routines tab
- Responsibilities: Scrape selected routine exercises with set data from DOM, generate JSON, download as file

**History Import:**
- Location: `workout_plan.html` lines 5222-5268 (importWorkoutSession)
- Triggers: User selects .json file from file input in History tab
- Responsibilities: Parse JSON, validate structure, check duplicates, detect PRs, save to localStorage

## Error Handling

**Strategy:** Defensive programming with try-catch for JSON parsing, null checks for DOM queries, duplicate prevention via timestamp/appId validation.

**Patterns:**
- localStorage parsing (line 4876): try-catch wraps JSON.parse, falls back to empty object
- Exercise data extraction (lines 4897-4935): checks for element existence before accessing properties
- File import (lines 5222-5268): validates appId matches 'workout-planner-session' and timestamp not in history
- DOM queries: mostly assume elements exist; some use optional chaining (?.querySelector)

**No explicit error modals:** Validation failures silently prevent action or show toast notifications via `showNotification(message, true)` with isError flag.

## Cross-Cutting Concerns

**Logging:**
- Console.error() for unexpected failures (e.g., JSON parse error on line 4879)
- No persistent logging

**Validation:**
- Duplicate routine exercise: checked by `addToRoutine()` before push (lines 4960-4980)
- Duplicate history session: checked by appId + timestamp match (line 5249)
- File structure: validates JSON has `appId === 'workout-planner-session'` and `routines` array (line 5245)

**Authentication:**
- Not applicable; client-side only, no backend

**Responsive Design:**
- CSS media queries at 768px (tablet) and 480px (mobile)
- Tab nav scrolls horizontally on mobile with `-webkit-overflow-scrolling: touch`
- Grid layouts reflow from 3 columns → 2 columns → 1 column
- Exercise cards stack vertically on mobile

---

*Architecture analysis: 2026-02-02*
