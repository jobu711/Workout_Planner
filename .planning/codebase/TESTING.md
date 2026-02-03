# Testing Patterns

**Analysis Date:** 2026-02-02

## Test Framework

**Runner:**
- Not configured
- No test files present
- No package.json or build tooling

**Assertion Library:**
- None (no testing framework)

**Run Commands:**
- N/A - no test runner configured

## Test File Organization

**Location:**
- No test files present in codebase
- No `__tests__`, `tests/`, `.test.js`, or `.spec.js` files

**Naming:**
- Not applicable

**Structure:**
- Not applicable

## Test Structure

**Suite Organization:**
- No test suites exist

**Patterns:**
- No setup/teardown patterns
- No test fixtures or factories
- No mocking infrastructure

## Mocking

**Framework:**
- None

**Patterns:**
- No mocking implemented
- All state managed in global variables: `routines`, `workoutHistory`, `analyticsCache`
- Manual state management during development

**What to Mock (if tests were added):**
- `localStorage` - Use sinon.stub or custom mock for getItem/setItem
- `Date.now()` - For timestamp-based testing of plateau detection and trend calculation
- `FileReader` - For testing JSON import functionality
- `document.createElement()`, `querySelector()` - DOM manipulation tests

**What NOT to Mock:**
- Core calculation functions: `estimate1RM()`, `calculateTrend()`, `detectPlateau()`
- Data transformation: `extractExerciseData()`, `getExerciseStats()`, `getAllExerciseAnalytics()`
- Business logic: `addToRoutine()`, `detectNewPRs()`, `getSuggestion()`

## Fixtures and Factories

**Test Data:**
- No fixtures present
- Manually created workout sessions exported from the app
- Sample files in `workouts/` directory (committed manually)

**Location:**
- `workouts/` - Contains sample .json exported sessions:
  - `Leg_day_Feb_1_2026.json`
  - `push_day_Jan_31.json`
  - `Shoulders_2026-02-02.json`

**Data Structure Example:**
```javascript
{
  "version": 1,
  "appId": "workout-planner-session",
  "date": "2026-01-31",
  "dateDisplay": "Jan 31, 2026",
  "timestamp": 1738350000000,
  "routines": [
    {
      "id": "push",
      "name": "Push Day",
      "subtitle": "Chest, Shoulders, Triceps",
      "exercises": [
        {
          "name": "Bench Press",
          "targets": "Chest, Triceps",
          "imageUrl": "https://...",
          "setsReps": "3 sets x 10-12 reps",
          "sets": [
            { "weight": 135, "reps": 12, "completed": true },
            { "weight": 135, "reps": 11, "completed": true },
            { "weight": 135, "reps": 10, "completed": true }
          ]
        }
      ]
    }
  ]
}
```

## Coverage

**Requirements:**
- No coverage metrics enforced
- No coverage reporting

**View Coverage:**
- Not applicable

## Test Types

**Unit Tests:**
- Should test calculation functions:
  - `estimate1RM(weight, reps)` - Epley formula
  - `calculateTrend(values)` - Linear regression slope
  - `detectPlateau(recentSessions)` - Plateau detection logic
  - `isLowerBodyExercise(name)` - Exercise classification
  - `getSuggestion(exerciseName)` - Weight/rep progression logic

- Should test data extraction:
  - `extractExerciseData(card)` - DOM parsing, level tag removal
  - `getExerciseSessions(exerciseName)` - Session filtering by name
  - `getExerciseStats(exerciseName)` - Aggregation and PR detection

**Integration Tests:**
- Should test workflow:
  - Add exercise → Save to routines → Persist to localStorage → Load on page reload
  - Import JSON → Validate appId → Detect duplicates → Render history
  - Export routines → Generate HTML → Include weight suggestions
  - Detect new PRs → Show modal → Update analytics

- Should test localStorage round-trip:
  - Save and load `workout_routines`
  - Save and load `workout_history`
  - Save and load `plateau_dismissals`

**E2E Tests:**
- Not configured
- Manual testing required:
  - Tab switching works for all 12 tabs
  - Exercise cards render with images and metadata
  - "Add to Routine" modal opens, prevents duplicates, saves
  - History import shows PR celebration modal when new PRs detected
  - Analytics view shows correct summary stats, plateaus, and PR hall of fame
  - Plateau dismissal persists for 7 days

## Common Patterns to Test

**Async Testing:**
```javascript
// FileReader is asynchronous
function importWorkoutSession(inputEl) {
    const file = inputEl.files[0];
    const reader = new FileReader();
    reader.onload = function(e) {
        const data = JSON.parse(e.target.result);
        // Test: verify JSON parsing errors show notification
        // Test: verify appId validation (must be "workout-planner-session")
        // Test: verify duplicate detection by timestamp
    };
    reader.readAsText(file);
}
```

**Error Testing:**
- Invalid JSON import should show error notification
- Duplicate timestamp should show error notification
- Missing appId should show error notification
- localStorage access failures should fall back to defaults

**Example test cases (if test suite existed):**

```javascript
describe('estimate1RM', () => {
    test('returns weight for 1 rep', () => {
        expect(estimate1RM(100, 1)).toBe(100);
    });
    test('uses Epley formula for multiple reps', () => {
        expect(estimate1RM(100, 10)).toBe(133); // 100 * (1 + 10/30)
    });
    test('returns 0 for invalid input', () => {
        expect(estimate1RM(0, 10)).toBe(0);
        expect(estimate1RM(100, 0)).toBe(0);
    });
});

describe('detectPlateau', () => {
    test('returns null for < 3 sessions', () => {
        expect(detectPlateau([session1, session2])).toBeNull();
    });
    test('detects stall when same weight for 3 sessions', () => {
        // 3 sessions with max weight 135 lbs but rep count not increasing
        const result = detectPlateau(last3Sessions);
        expect(result.type).toBe('stall');
        expect(result.suggestions).toContain('Try adding 1-2 reps per set');
    });
    test('detects fatigue when completion < 80%', () => {
        // 3 sessions with mostly incomplete sets
        const result = detectPlateau(lowCompletionSessions);
        expect(result.type).toBe('fatigue');
    });
});

describe('addToRoutine', () => {
    test('prevents duplicate exercises in same routine', () => {
        routines.push = [{ name: 'Bench Press', ... }];
        addToRoutine('push'); // Already has Bench Press
        expect(showNotification).toHaveBeenCalledWith(
            'This exercise is already in this routine!',
            true
        );
    });
    test('saves to localStorage after adding', () => {
        addToRoutine('push');
        expect(localStorage.setItem).toHaveBeenCalledWith(
            'workout_routines',
            expect.any(String)
        );
    });
});

describe('importWorkoutSession', () => {
    test('rejects file with wrong appId', () => {
        const invalidData = { appId: 'wrong-app', ... };
        // FileReader mock
        expect(showNotification).toHaveBeenCalledWith(
            'Invalid file: not a workout session export.',
            true
        );
    });
    test('prevents duplicate imports by timestamp', () => {
        const session1 = { timestamp: 1738350000000, ... };
        workoutHistory.push(session1);
        // Attempt to import same timestamp
        expect(showNotification).toHaveBeenCalledWith(
            'This workout session has already been imported.',
            true
        );
    });
    test('triggers PR detection on import', () => {
        const newSessionWithPR = { ... };
        importWorkoutSession(newSessionWithPR);
        expect(detectNewPRs).toHaveBeenCalled();
        expect(showPRModal).toHaveBeenCalled();
    });
});

describe('calculateTrend', () => {
    test('returns 0 for < 2 values', () => {
        expect(calculateTrend([100])).toBe(0);
    });
    test('calculates linear regression slope', () => {
        // [100, 105, 110, 115] = positive trend
        const trend = calculateTrend([100, 105, 110, 115]);
        expect(trend).toBeGreaterThan(0);
    });
});
```

## Manual Testing Checklist

**Core Features:**
- [ ] Tab switching: all 12 tabs load content and hide others
- [ ] Exercise card rendering: images load, text displays correctly
- [ ] "Add to Routine" buttons appear on all cards except routines tab
- [ ] Routine modal opens, prevents duplicates, adds exercise, saves
- [ ] localStorage persistence: refresh page, routines still present
- [ ] Remove from routine: count updates, exercise removed

**History & Analytics:**
- [ ] Export routine as HTML: includes weight suggestions, can save on iOS
- [ ] Import .json file: validates appId, prevents duplicates, renders history
- [ ] History sessions: expand/collapse, show exercise details and set badges
- [ ] Delete session: removes from history, updates localStorage
- [ ] Analytics toggle: switches between sessions and analytics views
- [ ] PR detection: new PR modal shows on import, correct calculations
- [ ] Plateau alerts: detected over 3 sessions, dismissible for 7 days
- [ ] PR Hall of Fame: shows recent PRs sorted by date
- [ ] Weight suggestions: pre-filled based on history and progression logic

**Mobile Responsive:**
- [ ] Tab nav scrolls horizontally on mobile
- [ ] Exercise cards stack vertically at 768px
- [ ] Modal dialog fits viewport
- [ ] Touch-friendly button sizes (min 44px)
- [ ] History panel slides in/out smoothly
- [ ] Sticky tab nav with backdrop blur

## Known Testing Gaps

**No tests for:**
- DOM manipulation and rendering logic
- localStorage serialization/deserialization edge cases
- Event handling (click, keyboard escape, etc.)
- Mobile responsiveness (viewport-based CSS)
- Image loading and external resource links
- Browser API usage (FileReader, Blob, URL.createObjectURL)
- Undo/redo or state recovery
- Performance regression in analytics with large datasets

---

*Testing analysis: 2026-02-02*
