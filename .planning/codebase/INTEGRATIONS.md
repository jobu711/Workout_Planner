# External Integrations

**Analysis Date:** 2026-02-02

## APIs & External Services

**Exercise Content:**
- LiftManual.com - Exercise reference content and demonstration videos
  - SDK/Client: Hotlinked images and guide links (no API client)
  - Usage: Exercise images from `https://liftmanual.com/wp-content/uploads/2023/04/[exercise-name].webp`
  - Guides: Links to `https://liftmanual.com/[exercise-slug]/` in "Guide" buttons
  - Example files using this: `workout_plan.html` (300+ image references, lines 2000-3600+)
  - No authentication required

**Social Media Video Links:**
- Instagram Reels - Embedded exercise demonstration videos
  - Example: `https://www.instagram.com/reel/DLSP_76Au0m/?hl=en` (Kettlebell Swing)
  - Usage: "Guide" buttons linking to Instagram
  - No API integration, simple links

- Facebook - Embedded exercise demonstration videos
  - Example: `https://www.facebook.com/reel/1575007680399450` (various exercises)
  - Usage: "Guide" buttons linking to Facebook
  - No API integration, simple links

**Exercise Images Secondary Source:**
- GymVisual.com - Backup exercise image source
  - URL: `https://gymvisual.com/img/p/3/8/2/5/1/38251.gif`
  - Usage: One or more exercise demonstration images
  - No API client, direct image URL linking

## Data Storage

**Databases:**
- None - No backend database

**File Storage:**
- Local filesystem only (user's machine)
- Exported workout JSON files stored in `workouts/` directory
- Files: `push_day_Jan_31.json`, `Leg_day_Feb_1_2026.json`, `Shoulders_2026-02-02.json`

**Browser Storage:**
- localStorage (in-browser key-value storage)
  - `workout_routines` - User's custom routines
  - `workout_history` - Imported workout sessions and tracking data
  - `plateau_dismissals` - Dismissed plateau alerts (expires 7 days)
  - Implementation: `localStorage.getItem()`, `localStorage.setItem()` (lines 4874, 4888, 5208, 5219, etc.)

**Caching:**
- JavaScript in-memory cache: `analyticsCache` (line 5363) - Caches computed analytics statistics
- JavaScript in-memory cache: `prCache` (line 5364) - Caches personal record calculations
- Cleared on new session import (line 5247)

## Authentication & Identity

**Auth Provider:**
- Custom/Manual - No external auth
- No user login system
- No accounts or authentication
- All data stored locally in browser localStorage
- Session ID concept: `appId: "workout-planner-session"` used to validate exported JSON files (validation in `importWorkoutSession()`, line 5231)

**Authorization:**
- None - No permission system
- All features accessible without login

## Monitoring & Observability

**Error Tracking:**
- None - No external error tracking service (Sentry, LogRocket, etc.)
- Basic try/catch blocks in JSON parsing (lines 4876-4881, 5228-5229, 5261-5262)
- Error notifications shown via `showNotification()` function with error flag (lines 4879, 5232, 5240, 5262)

**Logs:**
- Browser console only
  - `console.error('Error loading routines:', e)` (line 4879)
  - No persistent logging
  - No centralized logging service

## CI/CD & Deployment

**Hosting:**
- Static file hosting (any web server)
- No backend infrastructure required
- Git repository for version control (`.git/` directory present)

**CI Pipeline:**
- None detected

**Version Control:**
- Git (GitHub or similar)
- Recent commits visible in git history: branch `master`

## Environment Configuration

**Required env vars:**
- None - No environment variables required
- App configuration is hardcoded

**Secrets location:**
- No secrets in codebase
- No API keys, tokens, or credentials required
- No `.env` or `.env.local` files

## Webhooks & Callbacks

**Incoming:**
- None - No webhook endpoints

**Outgoing:**
- None - No outgoing webhooks
- No external HTTP calls made by the application
- All external integration is read-only (fetching images, embedding links)

## File Import/Export Functionality

**Import:**
- JSON file import via FileReader API (lines 5222-5267)
- Format validation: Checks `appId === 'workout-planner-session'` (line 5231)
- Duplicate detection: Checks timestamp against existing sessions (line 5238)
- Function: `importWorkoutSession(inputEl)` - Reads `.json` file selected by user
- Error handling: Shows notifications for invalid files, duplicates, JSON parse errors

**Export:**
- iPhone/iOS static HTML export: `exportForIPhone()` (lines 4990-5174)
  - Generates complete HTML with all routines, exercises, forms
  - Escapes HTML entities using `esc()` function (line 5020)
  - Includes weight/rep suggestion logic using `getSuggestion()` (line 5044)
  - Creates collapsible exercise sections with input fields for tracking
  - Generates JSON download for iPhone workout tracker app

**Data Flow:**
1. User exports routine from web app → JSON file saved locally
2. User transfers JSON to iPhone (via email, iCloud, etc.)
3. User opens `.json` file in Workout Planner app
4. App imports JSON and validates appId
5. Workout data merged into history with analytics calculated

## Connected Systems

**Workout History Analytics:**
- Progressive overload tracking: `getAllExerciseAnalytics()` (line 5568)
- One-rep-max estimation: `estimate1RM(weight, reps)` using Epley formula (lines 5390-5393)
- Trend detection: Linear regression slope `calculateTrend(values)` (lines 5396-5408)
- Plateau detection: Identifies stalls/fatigue from session data `detectPRs()` (lines 5485-5524)
- Personal record detection: Compares new sessions to history `detectNewPRs(session)` (lines 5525-5564)

**Smart Suggestions:**
- Weight progression algorithm: `getSuggestion(exerciseName)` (lines 5461-5483)
- Double-progression logic: Increases weight when rep target met, resets reps when weight increased
- Exercise classification: `isLowerBodyExercise(name)` determines weight increment strategy (lines 5450-5459)

---

*Integration audit: 2026-02-02*
