# Codebase Concerns

**Analysis Date:** 2026-02-02

## Tech Debt

**Monolithic HTML File:**
- Issue: All code (CSS, HTML, JavaScript) bundled in single 6,280-line HTML file (328KB)
- Files: `workout_plan.html`
- Impact: Difficult to maintain, test, and refactor; no code reusability; performance overhead on initial load; version control shows entire app changing even for small updates
- Fix approach: Extract CSS to separate stylesheet, split JavaScript into modules (routines, analytics, history, UI), create separate exercise data file. Use build tool or dynamic imports to maintain single-file delivery for iOS PWA compatibility.

**Hardcoded String Interpolation in HTML Generation:**
- Issue: Export function builds HTML by string concatenation with minified CSS embedded; analytics rendering uses raw HTML string assembly
- Files: `workout_plan.html` lines 5020-5196 (exportForIPhone), 5283-5356 (renderHistory), 5773-6007 (renderAnalytics)
- Impact: Brittle concatenation prone to typos; CSS duplication; hard to maintain styles; no templating prevents consistency
- Fix approach: Use template literals or a lightweight templating engine for HTML generation. Create CSS constant files. Consider using Handlebars or Nunjucks for complex templates.

**Inline SVG & Image Repetition:**
- Issue: SVG icons repeated throughout code (chevron, plus, checkmark, delete, etc.); no centralized icon system
- Files: `workout_plan.html` lines 4773, 5313, 5848, 5924, 6034, etc.
- Impact: Code bloat; inconsistent styling; hard to update icon appearance globally
- Fix approach: Create icon utility function or constants object for SVG definitions. Use CSS sprite or base64-encoded icons.

**Mixed ES6 & ES5 Syntax:**
- Issue: Modern arrow functions, template literals, const/let coexist with `var`, `function` declarations, and var-style concatenation
- Files: `workout_plan.html` lines 4831-6280 (entire script section)
- Impact: Inconsistent code style hurts readability; harder to enforce standards as codebase grows
- Fix approach: Standardize on ES6+. Use const by default, arrow functions for callbacks, template literals for strings. Add ESLint config.

**Global State Variables:**
- Issue: `routines`, `workoutHistory`, `currentExerciseData`, `analyticsCache`, `currentHistoryView`, `prCache` defined globally at module scope
- Files: `workout_plan.html` lines 4859-4870, 5200, 5363-5365, 5760
- Impact: Difficult to test in isolation; name collisions risk; unpredictable state mutations
- Fix approach: Encapsulate in module pattern or class. Use getter/setter functions. Add state validation.

## Known Bugs

**Duplicate History Buttons on Tab Switch:**
- Symptoms: "View History" button may appear multiple times on same exercise card if user switches tabs and returns
- Files: `workout_plan.html` lines 6018-6048 (addHistoryButtons function)
- Trigger: Navigate away from exercise tab, then back; addHistoryButtons() called again on DOMContentLoaded and after import
- Workaround: Button checks `if (card.querySelector('.view-history-btn')) return;` but repeated DOM manipulations can cause race conditions
- Fix approach: Debounce addHistoryButtons(), or check DOM state before appending. Use data attribute flag to mark processed cards.

**Analytics Cache Not Invalidated on Modal Close:**
- Symptoms: If user dismisses PR modal after import, then immediately switches to Analytics view, cache may show stale data momentarily
- Files: `workout_plan.html` lines 5363-5364 (analyticsCache), 5782 (force refresh but only in renderAnalytics)
- Trigger: Import workout, close PR modal, switch to Analytics tab quickly
- Workaround: None; eventual consistency as cache is invalidated on next history change
- Fix approach: Invalidate cache immediately after successful import, before showing PR modal.

**Exercise Name Case Sensitivity in Stats:**
- Symptoms: "Bench Press" and "bench press" treated as different exercises in analytics
- Files: `workout_plan.html` line 5654 (exerciseMap uses lowercase key but preserves original case for display)
- Trigger: User manually enters exercise with different casing in exported workout, or copy-paste variation
- Workaround: Consistent naming in exports mitigates
- Fix approach: Normalize exercise names to lowercase for all lookups (getExerciseSessions, detectNewPRs, etc.). Preserve original case only for display.

**Missing Image URLs in Exports:**
- Symptoms: Exercise images not included in exported HTML file; imageUrl fields are empty strings in JSON exports
- Files: `workout_plan.html` line 16 in sample workouts (empty imageUrl)
- Trigger: Exported HTML references images via src attribute, but if image fails to load, no fallback
- Workaround: User must manually add images or app fetches them from liftmanual.com on demand
- Fix approach: Include image URLs or data URIs in export. Add image preloading in exported HTML with fallback placeholder.

**Plateau Detection Logic Fragile:**
- Symptoms: Plateau detection may fire incorrectly if user logs incomplete sets or varies format significantly
- Files: `workout_plan.html` lines 5411-5488 (detectPlateau function)
- Trigger: Inconsistent data entry (some sets completed, others not); missing weight data
- Workaround: Filter for completed sets only before analysis
- Fix approach: Add validation layer before plateau detection. Document expected data structure. Add logging for edge cases.

## Security Considerations

**Stored XSS via Exercise Names in Analytics:**
- Risk: If exercise name contains HTML/JavaScript, innerHTML assignment could execute arbitrary code
- Files: `workout_plan.html` line 5826, 5853, 5963 (innerHTML = html; escapeHtml used but incomplete)
- Current mitigation: escapeHtml() function (lines 5202-5204) escapes &<>"
- Recommendations: Ensure ALL dynamic text inserted via innerHTML is escaped. Consider using textContent instead of innerHTML where possible. Audit all innerHTML assignments (lines 5288, 5356, 5775-5778, 5826, 5863, 6007, 6140, 6158, 6162). Add Content Security Policy headers if deployed to server.

**localStorage No Expiry/Validation:**
- Risk: Malformed or corrupted localStorage data could crash app; old data never pruned
- Files: `workout_plan.html` lines 4874-4881 (loadRoutines), 5206-5214 (loadHistory), 5831-5834 (loadDismissals)
- Current mitigation: Try-catch wraps JSON.parse; falls back to empty structure
- Recommendations: Add version/schema validation to stored objects. Implement cleanup of old dismissal records (older than 30 days). Consider size limits to prevent quota issues.

**File Upload No Validation:**
- Risk: Malicious JSON file could contain arbitrary data, causing unexpected behavior or consuming memory
- Files: `workout_plan.html` lines 5222-5267 (importWorkoutSession)
- Current mitigation: appId check (line 5231), timestamp duplicate check (line 5238)
- Recommendations: Validate all required fields exist and are correct types. Cap array lengths (max exercises per routine, max sets per exercise). Validate numeric ranges (weight, reps > 0). Reject files > size limit.

**Inline JavaScript in onclick Handlers:**
- Risk: onclick="dismissPlateau('...')" uses string interpolation; if name not properly escaped, could break
- Files: `workout_plan.html` line 5859 (`escapeHtml(item.name.replace(/'/g, "\\'"))`)
- Current mitigation: escapeHtml + manual single-quote escape
- Recommendations: Use addEventListener instead of onclick attributes. Store data in data-* attributes, retrieve via event.target.dataset. Avoid inline event handlers.

**External Image Hotlinks:**
- Risk: liftmanual.com images could change, disappear, or be replaced with malicious content; CORS issues on some browsers
- Files: All exercise cards reference `https://liftmanual.com/wp-content/uploads/2023/04/...`
- Current mitigation: None
- Recommendations: Cache images locally or use CDN with version pinning. Add image error handlers to show fallback. Consider downloading images at build time. Validate image URLs before storing.

## Performance Bottlenecks

**O(n²) or O(n³) Detection in Analytics:**
- Problem: detectNewPRs() iterates through entire history for each exercise in import; getExerciseSessions() scans all sessions for each name lookup
- Files: `workout_plan.html` lines 5675-5754 (detectNewPRs), 5367-5388 (getExerciseSessions)
- Cause: Nested loops with string comparisons; no indexing
- Improvement path: Pre-build exercise index when history loads. Use Map<exerciseName, sessions[]> instead of scanning every time. Cache results during render pass.

**Full Analytics Recalculation on Every View Switch:**
- Problem: renderAnalytics() calls getAllExerciseAnalytics() which reprocesses entire history (1000+ sessions would be slow)
- Files: `workout_plan.html` lines 5644-5673 (getAllExerciseAnalytics), 5773-5789 (renderAnalytics)
- Cause: Cache is invalidated on each render; no lazy evaluation
- Improvement path: Implement persistent cache keyed by history length + timestamp. Invalidate only on actual data change (import, delete). Use Web Worker for async analytics computation.

**DOM Manipulation in Loops:**
- Problem: renderRoutine() and renderExerciseAnalyticsCards() create DOM elements in forEach loops; layout thrashing
- Files: `workout_plan.html` lines 6171-6189 (renderRoutine), 5960-6005 (renderExerciseAnalyticsCards)
- Cause: Each appendChild triggers reflow; no batching
- Improvement path: Build HTML string first, then single innerHTML assignment. Or use DocumentFragment to batch appends.

**String Concatenation in Tight Loops:**
- Problem: renderHistory(), renderAnalytics(), renderExerciseAnalyticsCards() build HTML by += concatenation in forEach
- Files: `workout_plan.html` lines 5297-5357, 5820-5826, 5960-6005
- Cause: Each += creates new string; no array join optimization
- Improvement path: Use array.push() in loop, then join(). Template literals with arrays for cleaner code.

**No Pagination/Virtualization:**
- Problem: History view renders ALL sessions at once; Analytics view renders ALL exercises; no scrolling optimization
- Files: `workout_plan.html` lines 5283-5357 (renderHistory), 5954-6007 (renderExerciseAnalyticsCards)
- Cause: Linear rendering
- Improvement path: Implement virtual scrolling (render only visible items). Add pagination controls. Lazy-load analytics on tab switch.

**Heavy Sparkline Generation:**
- Problem: buildSparkline() called for every exercise card in analytics; Map.apply() for max calculation
- Files: `workout_plan.html` lines 5932-5952
- Cause: Inefficient max calculation; repeated SVG generation for same data
- Improvement path: Batch sparklines at render time. Cache min/max per exercise. Use CSS bar chart instead of DOM elements.

## Fragile Areas

**Modal State Management:**
- Files: `workout_plan.html` lines 4939-4950 (routine modal), 6115-6146 (PR modal)
- Why fragile: Single global currentExerciseData variable; no state validation; clicking outside modal may not close if overlay not properly positioned; nested modals could overlap
- Safe modification: Add state validation after close. Test overlay click handlers. Consider state machine pattern (open, closing, closed) instead of boolean.
- Test coverage: No unit tests for modal open/close edge cases; no test for rapid open/close clicks.

**Exercise Data Extraction:**
- Files: `workout_plan.html` lines 4896-4937 (extractExerciseData)
- Why fragile: Assumes specific DOM structure (h3 with level-tag, .exercise-details .detail-badge, .muscles with strong, .exercise-image); breaks if HTML structure changes
- Safe modification: Add defensive checks for missing elements. Use data attributes instead of DOM selectors. Add logging if extraction fails.
- Test coverage: No tests for missing elements or malformed cards.

**History Rendering with Set Badges:**
- Files: `workout_plan.html` lines 5329-5346
- Why fragile: Set badge styling depends on .completed and specific weight/reps being present; empty sets (0, 0) shown as dashes but styling assumes completed=true/false
- Safe modification: Validate set data structure before rendering. Add explicit state for incomplete/empty sets. Test all combinations.
- Test coverage: No tests for edge cases (0 weight, 0 reps, null completed flag).

**Plateau Detection Heuristics:**
- Files: `workout_plan.html` lines 5411-5488
- Why fragile: Three independent checks (weight stall, volume decline, low completion) can trigger false positives if user logs weight in pounds but has metric weights; or if one workout is deload by design
- Safe modification: Add user-configurable thresholds. Document assumptions (weight units, what counts as "stall"). Add telemetry to measure false positive rate.
- Test coverage: No tests with synthetic data for edge cases (very light weight, very high reps, perfect compliance, etc.).

**Exercise Name Fuzzy Matching:**
- Files: `workout_plan.html` lines 5490-5494 (isLowerBodyExercise), 5367-5388 (getExerciseSessions)
- Why fragile: isLowerBodyExercise() uses indexOf on lowercase name; would misclassify "Leg Curl" as lower body (correct) but "Shoulder Leg Press" would match "leg" (correct by accident). getExerciseSessions() uses exact string match after normalization; synonyms (Barbell Bench = Bench Press) counted as different
- Safe modification: Use structured exercise database with canonical names. Implement edit-distance matching or alias lookup. Test against exercise name variations.
- Test coverage: No tests for synonym handling.

## Scaling Limits

**localStorage Quota (5-10MB typical):**
- Current capacity: ~10,000 completed workouts (1KB per session) before hitting 10MB limit
- Limit: At 100 workouts/month, would fill in ~100 months. App performance degrades as JSON stringification gets slow.
- Scaling path: Implement IndexedDB for larger datasets (no practical limit). Add export/archive old sessions. Implement automatic cleanup of sessions > 1 year old.

**DOM Node Count:**
- Current capacity: ~1,000 exercises in history view renders smoothly; at 5,000+ sessions, rendering becomes sluggish
- Limit: Browser max DOM nodes is 10,000-100,000 depending on device; typical mobile phones slow down at 5,000+
- Scaling path: Virtual scrolling for history. Pagination in analytics. Lazy-load exercise cards.

**Analytics Computation Time:**
- Current capacity: ~500 exercises, ~2,000 sets of data computable in < 100ms on modern device
- Limit: At 1,000+ exercises or 10,000+ sets, getAllExerciseAnalytics() becomes noticeably slow; blocking the main thread
- Scaling path: Web Worker for background computation. Incremental updates instead of full recalculation. Cache at granular level (per-exercise stats).

**Array Allocation in detectNewPRs:**
- Current capacity: Import works fine up to ~500 exercises per session
- Limit: O(n²) comparison (tempHistory × session.exercises) becomes slow; memory allocation for intermediate arrays
- Scaling path: Pre-index tempHistory by exercise name. Use Set for O(1) lookup.

## Dependencies at Risk

**No External Dependencies (Intentional):**
- Risk: Browser compatibility; missing polyfills for older browsers (no Promise, fetch, etc.)
- Impact: App works only on modern browsers (ES6+ support required); iOS Safari must be recent
- Migration plan: If compatibility required, add polyfill bundle. Otherwise, document minimum browser version.

**Hotlinked liftmanual.com Images:**
- Risk: Domain could go offline, change URL structure, or block hotlinks with referrer policy
- Impact: All exercise images broken; exported HTML files reference dead URLs
- Migration plan: Download and host images locally. Store image data URIs in localStorage as fallback. Cache buster strategy.

**Hardcoded Routine/Tab Names:**
- Risk: Changing routine ID ("push" to "strength") would require renaming throughout code
- Impact: Refactoring requires manual string replacement
- Migration plan: Create constants object for routine definitions. Use computed properties instead of hardcoding routineNames objects.

## Missing Critical Features

**No Data Backup/Export Beyond JSON:**
- Problem: User data exists only in localStorage and manually exported JSON files; no automatic backup
- Blocks: Cloud sync, data recovery, cross-device access
- Severity: Medium (workaround: manual exports; data not lost but inaccessible if browser data cleared)

**No Undo/Redo:**
- Problem: Deleting a workout session or exercise from routine is permanent; no history of changes
- Blocks: Accident recovery
- Severity: Low (workaround: import backup JSON)

**No Exercise Search/Filter:**
- Problem: 200+ exercise cards require scrolling to find specific exercise
- Blocks: User efficiency on first-time discovery
- Severity: Low (workaround: browser find feature, but won't work in routines tab)

**No Bulk Operations:**
- Problem: Adding 5 exercises to routine requires 5 clicks + 5 modal confirmations
- Blocks: Quick routine building
- Severity: Low (workaround: manually edit exported JSON and re-import)

**No Set Templates:**
- Problem: User must re-enter set counts/reps for new exercises even if following same protocol
- Blocks: Faster routine creation
- Severity: Low

**No Dark Mode Toggle:**
- Problem: Dark theme is hardcoded; no system preference detection
- Blocks: Energy savings on OLED devices; accessibility
- Severity: Low

## Test Coverage Gaps

**Untested Area: Export HTML Generation:**
- What's not tested: Generated HTML validity, image fallbacks, CSS minification correctness, JSON stringification of workout data in embedded script
- Files: `workout_plan.html` lines 5020-5196 (exportForIPhone)
- Risk: Exported file could have broken HTML/CSS, invalid data in embedded script, orphaned quotes
- Priority: High (broken export ruins user experience; no way to track workouts)

**Untested Area: Analytics Algorithms:**
- What's not tested: Trend calculation with various dataset sizes, plateau detection edge cases, PR comparison logic with identical values
- Files: `workout_plan.html` lines 5396-5487 (calculateTrend, detectPlateau), 5675-5754 (detectNewPRs)
- Risk: False plateaus, missed PRs, incorrect trend direction
- Priority: High (incorrect fitness advice undermines app value)

**Untested Area: localStorage Corruption Handling:**
- What's not tested: JSON.parse errors on corrupted data, quota exceeded errors, data recovery
- Files: `workout_plan.html` lines 4874-4881 (loadRoutines), 5206-5214 (loadHistory)
- Risk: Corruption causes app to fail silently; user loses data
- Priority: Medium (fallback to empty state exists, but data loss is severe)

**Untested Area: File Import Validation:**
- What's not tested: Malformed JSON, missing fields, oversized files, version mismatches
- Files: `workout_plan.html` lines 5222-5267 (importWorkoutSession)
- Risk: Malicious or corrupted file crashes import; no graceful error recovery
- Priority: Medium (user-facing feature; error messages should guide recovery)

**Untested Area: Mobile Responsiveness Edge Cases:**
- What's not tested: Tab scrolling on narrow devices, modal positioning on landscape, input focus behavior on keyboard popup
- Files: CSS media queries lines 680-768, modal styling lines 1404-1450
- Risk: Unusable UI on some devices
- Priority: Medium (app is mobile-first; responsiveness is critical)

**Untested Area: Race Conditions:**
- What's not tested: Rapid import, delete, analytics switch in sequence; concurrent history and analytics updates
- Files: `workout_plan.html` lines 5245-5260 (import), 5269-5273 (delete), 5762-5771 (switch view)
- Risk: Stale data displayed; analytics cache out of sync; duplicate renders
- Priority: Low (hard to trigger, but async operations could be safer)

---

*Concerns audit: 2026-02-02*
