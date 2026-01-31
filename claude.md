# Workout Planner

## Project Overview

Single-page HTML workout companion app ("Complete Kettlebell & Shoulder Health Program") with inline CSS/JS. Dark-themed, mobile-responsive exercise reference with 11 tabbed sections and a custom routine builder backed by localStorage.

## Architecture & Structure

### File: `workout_plan.html` (~3,985 lines)

**Layout:** `<head>` (styles, lines 1-1042) | `<body>` (HTML content, lines 1045-3700) | `<script>` (JS, lines 3702-3982)

### Tab System

| Tab ID | Button Class | Section | Line Range | Exercise Count |
|--------|-------------|---------|------------|----------------|
| `tab-shoulder` | `.shoulder-tab` | Shoulder Exercises | 1068-1256 | 15 (warm-up 5, pressing 7, upright rows 3) |
| `tab-kettlebell` | `.kettlebell-tab` | Kettlebell Training | 1259-1687 | 32 (warm-up 4, posterior 2, legs 7, shoulders 3, more KB 12, KB core 4) |
| `tab-core` | `.core-tab` | Core Strength | 1689-2102 | 25+ |
| `tab-back` | `.back-tab` | Back Training | 2104-2410 | 19 (pull-ups 5, rows 6, pulldowns 6, lower back 2) |
| `tab-biceps` | `.biceps-tab` | Biceps Training | 2412-2660 | 15 (barbell 3, dumbbell 7, cable 3, machine 2) |
| `tab-chest` | `.chest-tab` | Chest Training | 2662-2960 | 18 (barbell 3, dumbbell 3, flies 5, machine 2, push-ups 5) |
| `tab-triceps` | `.triceps-tab` | Triceps Training | 2962-3203 | 14 (pushdowns 3, overhead 3, skull crushers 4, kickbacks 1, dips 3) |
| `tab-legs` | `.legs-tab` | Leg Training | 3205-3498 | 22 (hamstrings 11, quads 8, calves 3) |
| `tab-cardio` | `.cardio-tab` | Cardio Exercises | 3500-3587 | 3 |
| `tab-wrist` | `.wrist-tab` | Wrist Training | 3589-3684 | 5 (kettlebell 3, dumbbell/barbell 1, towel 1) |
| `tab-routines` | `.routines-tab` | My Workout Routines | 3686-3800 | N/A (user-built) |

### CSS Architecture (lines 1-1042)

- **Theme colors:** Background `#1a1a2e`/`#16213e`, Orange accent `#ff6b35`, Teal accent `#4ecdc4`
- **Card variants:** `.exercise-card` (default orange), `.shoulder-card` (teal), `.highlight` (darker orange)
- **Level tags:** `.level-beginner` (green), `.level-intermediate` (yellow), `.level-advanced` (red), `.level-rehab` (blue), `.level-warmup` (gray)
- **Tab button colors:** Each tab has unique active gradient (e.g., `.shoulder-tab.active` = teal, `.chest-tab.active` = red, `.wrist-tab.active` = gold/amber)
- **Responsive breakpoints:** 768px (mobile layout, horizontal tab scroll), 480px (compact spacing)
- **Key components:** `.tab-nav` (sticky, backdrop-blur), `.exercises-grid` (auto-fit grid, min 300px), `.routine-modal-overlay`, `.routine-notification`

### Exercise Card Template

```html
<div class="exercise-card">
    <img src="[liftmanual.com URL]" alt="[name]" class="exercise-image">
    <h3>[Name] <span class="level-tag level-[level]">[Label]</span></h3>
    <p>[Description]</p>
    <div class="exercise-details">
        <span class="detail-badge">[Sets x Reps]</span>
        <a href="[guide URL]" class="watch-link">Guide</a>
    </div>
    <p class="muscles"><strong>Targets:</strong> [Muscle groups]</p>
    <div class="form-tips"><strong>Key:</strong> [Tip text]</div>
    <!-- "Add to Routine" button injected by JS -->
</div>
```

### JavaScript (lines 3702-3982)

| Function | Purpose |
|----------|---------|
| `showTab(tabId)` | Switch active tab, scroll to top on mobile |
| `loadRoutines()` | Parse localStorage into `routines` object |
| `saveRoutines()` | Serialize `routines` to localStorage |
| `extractExerciseData(card)` | Scrape name, sets/reps, targets, image from card DOM |
| `openRoutineModal(card)` | Show modal with routine options for a given exercise |
| `closeRoutineModal()` | Hide modal, clear temp data |
| `addToRoutine(routineId)` | Push exercise to routine array, save, re-render |
| `removeFromRoutine(routineId, exerciseId)` | Filter exercise out, save, re-render |
| `renderRoutine(routineId)` | Build DOM for a routine's exercise list |
| `renderAllRoutines()` | Render all 4 routines |
| `showNotification(message, isError)` | Toast notification with auto-dismiss |
| `addRoutineButtons()` | Inject "Add to Routine" button onto every exercise card |

**localStorage key:** `workout_routines`

**Routine IDs:** `push`, `pull`, `legs-core`, `shoulders`

### External Resources

- Exercise images/GIFs: `liftmanual.com/wp-content/uploads/2023/04/`
- Exercise guides: `liftmanual.com/[exercise-slug]/`
- Video links: Instagram reels, Facebook shares

## Project Rules / Coding Conventions

- All code is inline in a single HTML file (no build tools, no external CSS/JS)
- Exercise images are hotlinked from liftmanual.com
- SVG icons are inline (YouTube icon, plus icon, checkmark icon, close icon)
- Cardio and Wrist tabs use inline styles on elements rather than dedicated CSS classes
- Tab content sections are wrapped in `<div id="tab-[name]" class="tab-content">`
- Workout sections use `<div class="workout-section">` with `<div class="exercises-grid">` for card layout
- Section dividers use `<div class="section-divider [type]">` between major groups

## Agent Preferences

Use workout-tracker-frontend agent for all coding tasks.
After major revisions (new tabs, bulk exercise additions, structural changes), commit changes to the git repository with a descriptive message.

## Key Data Structures

### Routine Exercise Object (JS)
```json
{
    "id": "ex_1234567890_abc123def",
    "name": "Exercise Name",
    "setsReps": "3 sets x 10-12 reps",
    "targets": "Muscle Group 1, Muscle Group 2",
    "imageUrl": "https://liftmanual.com/wp-content/uploads/..."
}
```

### Routines Object (localStorage)
```json
{
    "push": [],
    "pull": [],
    "legs-core": [],
    "shoulders": []
}
```

## Verification Checklist

- [ ] Tab switching works for all 11 tabs
- [ ] Exercise cards render with image, description, sets/reps, targets
- [ ] "Add to Routine" buttons appear on all exercise cards (not in Routines tab)
- [ ] Routine modal opens/closes, adds exercises, prevents duplicates
- [ ] Routines persist across page reloads (localStorage)
- [ ] Remove from routine works and updates count
- [ ] Notification toasts appear for add/remove/duplicate actions
- [ ] Mobile responsive: tabs scroll horizontally, cards stack vertically
- [ ] All external image/guide links load correctly
- [ ] Sticky tab nav with backdrop blur works on scroll
