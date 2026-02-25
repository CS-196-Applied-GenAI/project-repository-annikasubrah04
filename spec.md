# Personal Health & Fitness App — Specification (spec.md)
Date: 2026-02-25
User: annikasubrah04

This document captures the complete product, privacy, onboarding, data, nutrition, workout, KPI, alerting, and minimal-backend OAuth design agreed during the conversation. It is intended to be handed to engineering, product, and legal teams as the canonical spec for the MVP.

---

## 1. High-level goals & approach
- Primary goals supported:
  - Weight loss (user-selectable exact target; default guideline: 12 lbs in 12 weeks, ≈1 lb/week)
  - General fitness (scheduled workouts, progressive overload)
  - Sleep optimization (target 8 hours/night with sleep quality metrics)
  - Healthy eating (daily calories & macros, protein priority)
  - Muscle gain (progressive resistance training + protein target)
- Personalization approach:
  - Hybrid model with a heavier weight on data-driven models that learn from the user's historical data.
  - Recommendations are continuously adaptive (plans update over time based on data and engagement).
  - Where appropriate, rules-based health guardrails are enforced (e.g., calorie floors, BMI-based overrides).

---

## 2. Platforms & deployment
- MVP target platforms:
  - iOS (native) — highest priority
  - Web (responsive / PWA)
  - Android: possible later if codebase is easily transferable
- Data storage policy:
  - Default: ALL data is on-device only.
  - The only allowed off-device component is a minimal backend used strictly as an in-memory OAuth relay for Fitbit/Oura (no persistent storage of tokens or PHI).
  - No cloud sync by default; optional encrypted cloud sync is NOT implemented in this spec (all data remains on-device).
- Backend OAuth design for Fitbit/Oura:
  - Minimal backend performs the OAuth exchange and relays data to the app strictly in-memory, then immediately zeroizes tokens/data.
  - Provider authorization remains valid at the provider (user stays connected until they disconnect at provider or in-app).
  - Backend must not persist tokens or device data to disk at any time.

---

## 3. Privacy, security, compliance
- Regulatory/compliance: Comply with HIPAA and relevant regulations.
- Data storage: On-device only (with the exception of the in-memory OAuth relay for Fitbit/Oura described above).
- Encryption:
  - In-transit: TLS 1.2+ required.
  - At-rest: device storage encrypted (use platform-secure storage: iOS Keychain / Secure Enclave).
  - End-to-end: required for highly sensitive items where applicable (e.g., PDF exports can be password-protected).
- Authentication:
  - Email/password required.
  - 2FA optional but highly recommended (show prompt encouraging users to enable).
- Data retention & deletion:
  - Users can export and fully delete data at any time.
  - Policy: data associated with an account will be deleted after the user deletes the app and 6 months have elapsed (explicit deletion lifecycle).
- Model training / analytics:
  - No data leaves the device by default for centralized model training.
  - If any remote model-improvements are desired in the future, that must be a separate explicit opt-in, with clear consent.
- Sharing:
  - Exports: downloadable, password-protected PDF reports (manual export only).
  - No automatic sharing to clinicians/coaches unless user manually exports and transmits the file.
- Third-party analytics: NOT permitted in the MVP.
- Breach notifications / logging / audit trails:
  - Breach notifications are required. Minimal operational logs only; do not include PHI.
- Other constraints:
  - Do not store raw GPS location.
  - Keep raw sleep/device sensor data only on-device.
  - If any server interacts with PHI (the in-memory relay), treat it as a HIPAA-covered process (audit, secure infra, BAA as required).

---

## 4. Onboarding flow & items
Ordering, Required/Skippable, Connect-later options, and UX notes:

1. Account creation (email/password)
   - Required. 2FA optional.
   - Connect later: N/A (account creation is first step).
2. Terms of Service + HIPAA/GDPR consent + opt-in for de-identified model training
   - Required. (Given on-device-only default, de-identified model training remote usage is disabled by default; opt-in would be an explicit separate agreement.)
   - Will block progress until accepted.
3. Basic profile fields (REQUIRED)
   - DOB/date-of-birth, calculated age, sex/gender, height, current weight, allergies, dietary preferences, medical conditions, target goals.
   - Required, cannot be skipped (essential for safe personalization).
4. Optional profile fields
   - Typical sleep schedule, time availability, preferred workout types, gym access, grocery budget.
   - Skippable (Connect later: Yes).
5. Device connections: Apple Health / HealthKit, Fitbit, Oura
   - Requirement: User must choose either device connection OR manual entry for core features to operate ("REQUIRE EITHER THIS OR MANUAL ENTRY").
   - For each provider, request read/write permission scopes (explain scopes in plain language).
   - Fitbit/Oura route uses the minimal in-memory OAuth relay backend (see OAuth section).
   - Connect later: Yes.
6. Manual food/exercise entry setup
   - REQUIRE EITHER device connection or manual entry — app should offer both, and the user can enable manual entry now or later.
7. Optional encrypted cloud sync opt-in
   - Skippable (not used in MVP).
8. Onboarding baseline assessments (REQUIRED)
   - Bodyweight tests, 1RM (or estimated 1RM) from a few lifts, and/or AMRAP on a specified weight; movement screen.
9. Initial training plan preview & acceptance
   - Required. User can accept now or “customize later”.

Sequencing notes:
- Block progression until account + TOS/HIPAA consent + basic profile + onboarding baseline assessments are complete.
- Device connections may be deferred until after profile completion; show them as high-priority next steps.
- During device connect, clearly display required scopes and short consent language.

---

## 5. User profile fields (collected)
Required:
- Date of birth (DOB) → Age
- Sex/Gender
- Height
- Current weight
- Allergies
- Dietary preferences (e.g., vegetarian, vegan, dairy-free)
- Medical conditions (cardiac conditions, diabetes, pregnancy, etc.)
- Target goals (weight loss, general fitness, sleep optimization, healthy eating, muscle gain) with concrete metrics/timeframes (user selects per-goal numeric objectives)

Optional:
- Typical sleep schedule
- Time availability for workouts
- Preferred workout types (strength/cardio/hybrid)
- Gym access (equipment available)
- Grocery budget

---

## 6. Goals & success metrics (user-provided and app-tracked)
User-selected goals can include:
- Weight loss: default guideline target selectable by user (example default: 12 lb in 12 weeks). Weekly rate: preferred ~1 lb/week, acceptable 0.5–2 lb/week.
- General fitness: number of workouts per week, reps completed for target exercise(s), weight lifted for specific exercises.
- Sleep optimization: 8 hours/night target, sleep latency, sleep timing consistency, sleep efficiency.
- Healthy eating: daily calories vs goal, grams of protein, food variety (servings vegetables/fruit), micronutrient flags.
- Muscle gain: reps/weights for target exercises; protein intake in g.

Success metrics the user can track (examples):
- Body weight (single value + trend)
- Body fat % (if available)
- Daily calories & macros (protein grams emphasized)
- Number of workouts/week and adherence
- Sleep hours, sleep efficiency
- Workout-specific metrics (sets/reps/weight)
- Steps, active minutes (as provided by device)

Safety constraints / medical overrides:
- If BMI < 18.5, weight-loss programs are overridden / paused; prioritize health & physician consult.
- If BMI >= 30, special health guidance is applied (priority on health & safe progress).
- Calorie floors: BMR-derived floors but with hard minimums: women minimum 1200 kcal, men minimum 1500 kcal — app must enforce these.

---

## 7. Nutrition — formulas & adaptive rules
1. BMR / TDEE
   - BMR: Mifflin–St Jeor formula is used.
   - Activity multipliers:
     - Sedentary: 1.2
     - Light: 1.375
     - Moderate: 1.55
     - Active: 1.725–1.9
   - Maintenance = BMR × activity multiplier.
2. Initial calorie target for weight loss:
   - Target = maintenance − 500 kcal/day (default).
3. Macro splits:
   - Weight loss: protein 30–35% of calories (but also enforced as 1 g/kg bodyweight), carbs 30–40%, fats 20–30%.
   - Muscle gain: protein 30–35% (user typed "3-35%" earlier — assumed 30–35%), carbs 40–50%, fats 20–30%.
   - Protein absolute minimum: 1 g per kg bodyweight (enforced).
4. Adaptive adjustment rules:
   - Evaluate progress monthly.
   - If actual weight change < target after 1 month: reduce calories by 100–200 kcal.
   - If weight loss > target after 1 month: increase calories by ~100 kcal.
   - Never set calorie target below the hard minimum (1200 women / 1500 men).
5. Refeed / diet break:
   - Allow a higher-calorie day (cheat or refeed) once every 2 weeks.
   - Refeed day should NOT exceed 150% of daily maintenance and target a max of 2000–2500 kcal for that day (user constraint).
6. Dietary constraints:
   - Do not force animal protein. Allow vegetarian/vegan protein sources (soy, eggs, legumes).
   - Recipes and meal suggestions must be flagged for dietary preferences and allergies.
7. Calorie floor enforcement:
   - Calorie floors are BMR-derived but always respect the hard floors (even if BMR < floor, enforce the floor).

---

## 8. Workout program generation & progression
Defaults and user customization:

- Training emphasis:
  - User picks one initially: strength / cardio / hybrid.
  - App can recommend a different emphasis after ~1 month of collected data.
- Sessions per week & session duration:
  - Default: 4–5 sessions/week, ideally 60 minutes each.
  - User-configurable.
- Baseline / assessment:
  - Bodyweight tests, estimated 1RMs from lifts or AMRAP tests, movement screens.
- Program structure & progression:
  - Autoregulated progression focusing on progressive overload (RPE-friendly where appropriate).
  - Progression rules:
    - Arm workouts: increase weight by 2.5–5% after completing 10–12 reps across sets (or once target rep ranges are consistently hit).
    - Leg workouts: increase weight by 2.5–5% after completing 8 reps across sets.
  - Periodization: usable but default is autoregulated micro-progression for the MVP.
- Exercise selection & equipment:
  - For muscle gain prioritize compound lifts; for other goals, include more isolation as appropriate.
  - Substitutions:
    - No dumbbells: kettlebells or bodyweight alternatives.
    - No barbell: heavy dumbbells or machines/leg press.
- Warm-up, mobility, recovery:
  - Included in each session by default. Users may skip with a warning about increased injury risk.
- Time/complexity limits:
  - For college students: respect user's "max workout time" preference; filter the number of exercises to match session-duration limit.
- Injury handling:
  - Exclude exercises if user reports an injury; show alternatives.
  - For serious conditions, display physician sign-off prompt in-app (informational; not a substitute for medical advice).
- Tracking rules:
  - Require user confirmation of performed weights/reps.
  - If user does not input performed sets/reps, use auto-increment predicted 1RM behavior (but prefer user confirmation for accuracy).
- Push guidance:
  - Users can select to receive daily workout plan, weekly plan refreshed every 7 days, and in-workout nudges.

---

## 9. KPI dashboard (priority order & display)
Primary KPIs (in order of priority) and display type:
1. Workouts this week — Single-value card (e.g., "5/7")
2. Protein — Single-value card (consumed / goal)
3. Sleep last night — Single-value card (hours:minutes)
4. Weekly training adherence — Single-value card (sessions attended / target)
5. Calories remaining — Single-value card (calories left for day)

Tap actions and alerting rules for each KPI:

- Workouts this week
  - Green: 5+ workouts completed
  - Yellow: 3–4 workouts
  - Red: <3 workouts
  - Tap: Quick action: "Start workout" + recommended workouts
  - Push alert: Yes — message example: "Start a workout to hit your weekly goal!"
- Protein
  - Target: 1 g / kg bodyweight
  - Status:
    - Green: ≥75% of daily protein goal
    - Yellow: 50–74%
    - Red: <50%
  - Tap: Show comparison to target; quick actions: "Log meal"
  - Push alert: Yes — prompt to log a meal.
- Sleep last night
  - Target: 8.0 hours
  - Green: ≥8 hrs
  - Yellow: 6–7 hrs
  - Red: <6 hrs
  - Tap: Show comparison to target and sleep details
  - Push alert: No
- Weekly training adherence
  - Same thresholds as "Workouts this week" (green for 5+, yellow 3–4, red <3)
  - Tap: "Start workout" + recommended training plan for the day
  - Push alert: Yes — "Follow your workout schedule and start a workout now!"
- Calories remaining
  - Compare consumed calories to daily target (BMR-derived target with the hard floors)
    - Status mapping relative to BMR:
      - Green: ≥75% of BMR-derived target
      - Yellow: 50–74%
      - Red: <50% of BMR-derived target OR below hard floor (women <1200 / men <1500) → automatic red
  - Tap: Show breakdown, quick actions: "Log meal"
  - Push alert: Yes — prompt to log a meal.

Notification privacy rules:
- Notifications must not include sensitive health data content in clear text.
- Respect device DND / quiet hours and do not bypass them.
- Critical alerts are opt-in (user may opt in to high-priority critical alerts).

---

## 10. Critical health thresholds & immediate alert behavior
If any of these thresholds are met, the app issues a high-priority push alert and recommends pausing activity and contacting a clinician. The app does not attempt to replace emergency services guidance but will advise contacting clinician/911.

- Resting heart rate
  - Thresholds: >100 bpm (high), <60 bpm (low)
  - Alert persistence: Alert on single reading if confirmed by connected device. If using intermittent readings, consider requiring 2 consecutive readings in a short window (implementation detail left to device integration).
  - Device requirement: connected device (watch/HR sensor).
- Heart rate during exercise
  - Thresholds: >170 bpm OR exceeding 85–90% of estimated HRmax
  - Alert persistence: Single reading sufficient if device reports it reliably during exercise.
  - Device requirement: connected exercise-capable HR device.
  - Action: push high-priority alert and advise to stop exercise.
- Rapid weight loss
  - Threshold: >2 pounds (≈0.9 kg) lost per week
  - Alert persistence: Trigger when average weekly loss exceeds threshold.
  - Device requirement: weight/history from user/device.
- Calorie intake below safe floor
  - Threshold: daily intake below calorie floor (BMR-derived but hard-min enforced)
  - Alert persistence: Immediate alert when daily intake < floor for 2 consecutive days (i.e., persistent under-eating).
  - Device requirement: nutrition logging or estimate from app.
- Signs of severe sleep apnea
  - Detection rules:
    - Repeated nightly O2 desaturations and extreme spikes in HR during the night.
    - Threshold: blood oxygen levels below 95% for more than 3 days and/or repeated severe fragmentation / extreme HR spikes at night.
  - Alert persistence: >3 consecutive nights with low SpO2 or fragmentation metrics.
  - Device requirement: device-native SPO2/HR/respiratory metrics (Oura/Fitbit/Apple devices).
- SPO2 (general)
  - Threshold: <95% for more than 3 days
  - Alert persistence: same as above (3 days)
  - Device requirement: device-provided SPO2 metric.
- Arrhythmia / irregular heart rhythm
  - Detection rules: rely on device-native arrhythmia detection and algorithms (e.g., irregular intervals consistent with AFib).
  - Alert persistence: device-native detection rules (do not attempt to implement custom detection without device certification).
  - Device requirement: must be a device with native arrhythmia detection. Do not include arrhythmia detection from non-certified sources.

Alert actions:
- Push high-priority notification (visible on lock-screen) advising to stop exercise and contact clinician as appropriate.
- Do not automatically call emergency services; instruct user to contact clinician/911.
- For critical alerts, optionally present full-screen emergency guidance within the app if user taps the notification (but do not auto-escalate).

---

## 11. Merging / source selection for overlapping device data
- User picks a preferred source per metric during onboarding (explicit choice): e.g., user chooses Apple Health for steps, Oura for sleep, Fitbit for active calories, etc.
- App displays an acknowledgement that some sources may be less accurate for certain metrics.
- Default UX: prompt recommended mapping but allow user override per metric.
- Main metrics to map: steps, active calories, total calories, sleep, resting heart rate, HRV, SpO2.

---

## 12. PDF reports & exports
- File properties:
  - Password-protected PDF (user sets password at export time).
  - File naming convention: firstname_lastname_YYYY-MM-DD (date of report).
- Default time ranges:
  - Monthly (user can schedule monthly reports).
  - Allow weekly/custom date ranges manually.
- Content:
  - Sleep vs target (summary & trend)
  - Protein intake vs target & daily breakdown
  - Calories intake vs daily goals & trend
  - Weekly workouts completed, session adherence vs plan
  - Adherence summary (high level)
- Attachments:
  - Optionally include CSV exports as separate files (user choice).
- Clinician-friendly summary:
  - Include high-level safety flags and whether any high-priority alerts occurred in the period.
- Metadata:
  - Include device-source names (e.g., Apple Watch, Oura) for sections that rely on device data.
  - Include a short user consent statement in the report footer: "This report was exported by the user from the Personal Health App on [date]."
- Scheduling:
  - Users may schedule monthly automatic report generation. (Note: reports are generated on-device.)
- Storage & privacy:
  - When scheduled, the PDF must be stored only on-device and/or downloaded by the user; if shared, user handles transmission.
  - Reports are password-protected by the user on export.

---

## 13. Export / sharing policy
- Sharing must be manual — app does not automatically share or push user data to third parties.
- Exports available: PDF (password-protected), CSV (optional).
- Recommended: show a brief checklist before export reminding the user that exported files may contain sensitive health information and they should handle sharing carefully.

---

## 14. Optional cloud sync decision & Fitbit/Oura integration
- Final decision: All data remains on-device by default.
- To support Fitbit/Oura:
  - Implement a minimal backend that:
    - Performs the OAuth token exchange (server-side) so client_secret remains off the client.
    - Immediately relays data/tokens to the app in the same TLS session.
    - Zeroizes tokens and any fetched health data from memory immediately after relaying.
    - Does NOT persist tokens or health data on disk or long-term storage.
  - Provider authorization remains valid at the provider (user continues to be connected until they disconnect).
  - Option chosen: Option 1 for lifecycle: provider authorization persists at provider; the backend does NOT persist tokens. The app stores tokens locally in secure storage (Keychain).
  - Backend in-memory behavior: strictly run in-memory; zero out tokens and fetched payloads; do not write to logs or files containing PHI.
  - If provider requires server-side refresh that cannot be performed by the app without client_secret, the backend may perform an ephemeral refresh on-demand and relay data, then zeroize.

---

## 15. Minimal backend OAuth API (developer summary)
- Endpoints (minimal):
  - POST /oauth/start
    - Request: { provider: "fitbit" | "oura", pkce_challenge }
    - Response: { authUrl }
  - GET /oauth/callback
    - Provider redirects here with code & state
    - Backend does token exchange; either:
      - Option A (recommended): returns tokens via secure deep-link to the app (app stores tokens locally in Keychain) — backend then deletes tokens from memory.
      - Option B: backend fetches a one-time summary payload and sends it to the app and deletes tokens/data.
  - POST /oauth/relay-data (optional)
    - For ephemeral refresh & relay (backend uses in-memory tokens only).
  - POST /disconnect
    - Requests provider revoke and returns success.
- Operational rules:
  - No persistence of tokens or PHI.
  - No PHI in logs. Minimal non-PHI operational logs allowed.
  - All communications TLS-only.
- Recommended flow:
  1. App POST /oauth/start (PKCE).
  2. Open authUrl in external browser / ASWebAuthenticationSession.
  3. Provider redirects to /oauth/callback.
  4. Backend exchanges code → immediately returns tokens or payload to app via deep link.
  5. Backend zeroizes memory and returns 200.
  6. App stores tokens in Keychain and performs future provider sync directly when possible.

---

## 16. Backend security & HIPAA notes
- Use secret manager / HSM for provider client_secrets.
- Rotate secrets regularly.
- Avoid any disk writes of tokens or PHI. If temporary disk caching absolutely needed in an extreme failure mode, encrypt and delete immediately (this is discouraged).
- Maintain minimal operational logs (no PHI).
- Implement intrusion detection, breach procedures, and required breach notification flows.
- For legal/regulatory purposes, treat the relay as touching PHI and follow BAA and HIPAA operational controls.

---

## 17. UX copy - device connect consent (suggested)
"Connect your device
We use OAuth to connect to Fitbit or Oura so your device can share sleep and heart-rate data with this app. To protect your privacy:
- We only request the minimum permissions needed (sleep, heart rate, steps).
- Our backend performs the initial connection but does NOT store your health data or tokens. Data is relayed directly to your device and then immediately deleted from memory.
- Your device authorization stays active until you disconnect. You can disconnect anytime in Settings."

---

## 18. KPI / Dashboard & Notifications: summary of behavior
- Show five primary KPI cards on the main dashboard (Workouts this week, Protein, Sleep last night, Weekly training adherence, Calories remaining).
- Each KPI has color-coded status (green/yellow/red) based on thresholds defined in Section 9.
- Tapping a KPI opens the relevant detail view and offers quick actions (log meal, start workout, view full chart).
- Notifications:
  - Daily push notifications: morning targets (calories, protein, workout plan).
  - Real-time nudges during/after workouts or after poor sleep or other events.
  - Weekly training plan refreshed every 7 days and optionally pushed as a summary.
  - No sensitive details shown in push notifications; they should be prompts to open the app.

---

## 19. Data retention, export, deletion & user control
- Users can fully delete or export their data.
- When a user deletes the app / account, data will be cleared after a 6-month retention window (user data deletion lifecycle).
- The user can immediately request account/data deletion or export at any time.
- No cloud backups by default — exports are user-driven and stored/transmitted externally at the user’s discretion.

---

## 20. Next steps for engineering / product
Suggested immediate artifacts to produce from this spec:
- Generate a developer-ready OpenAPI spec for the minimal OAuth relay endpoints and backend behavior.
- Implement the iOS client flows:
  - Onboarding screens
  - PKCE + ASWebAuthenticationSession flow integration with backend endpoints
  - Keychain storage for provider tokens
  - HealthKit/Apple Health integration
- Implement on-device persistence & model pipelines (local adaptive models that learn from user data).
- Implement dashboard UI with the five KPI cards and the tap/quick-action behaviors.
- Implement safety engine that monitors critical thresholds and fires alerts per Section 10.
- Produce legal / consent copy for review by counsel and privacy team (include the device-connect consent copy above).
- Implement PDF export generation (password-protected) and monthly scheduling on-device.

---

## 21. Appendices / Reference values (quick)
- Calorie hard floors:
  - Women: 1200 kcal/day (minimum enforced)
  - Men: 1500 kcal/day (minimum enforced)
- Protein: 1 g per kg bodyweight (daily minimum target)
- TDEE multipliers:
  - Sedentary: 1.2
  - Light: 1.375
  - Moderate: 1.55
  - Active: 1.725–1.9
- Initial weight-loss calorie adjustment: maintenance − 500 kcal/day
- Macro ranges:
  - Weight loss: protein 30–35% (1 g/kg floor), carbs 30–40%, fat 20–30%
  - Muscle gain: protein 30–35%, carbs 40–50%, fat 20–30%
- Critical thresholds:
  - Resting HR: >100 bpm or <60 bpm
  - Exercise HR: >170 bpm or >85–90% HRmax
  - Rapid weight loss: >2 lb/week
  - SPO2: <95% for >3 days → alert
  - Calorie floor violations: daily < floor for 2 days → alert
  - Arrhythmia: device-native detection only

---

End of spec.md