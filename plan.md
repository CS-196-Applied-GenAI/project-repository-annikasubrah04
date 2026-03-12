# Personal Health & Fitness App — Project Blueprint (plan.md)

This document provides a detailed, step-by-step implementation plan for building the MVP as specified in spec.md. It follows an iterative approach, breaking down complex domains into manageable, testable, and logically ordered steps. Each round of decomposition further splits the chunks to ensure strong testability and safe progress, while maintaining forward momentum.

---

## 1. Initial Project Setup & Foundations
1. **Repository initialization**
2. **Set up monorepo or multiple subrepos (iOS, Web, Backend)**
3. **Define CI/CD pipelines**
4. **Select base tech stacks (iOS: Swift, Web: React/PWA, Backend: minimal relay e.g., Node/Go)**
5. **Write project README.md with vision, goals, architecture overview, and contribution guidelines**

## 2. Core Backend (OAuth Relay)
- **2.1. Scaffold minimal OAuth backend API structure**
  - Draft OpenAPI spec for endpoints
  - Set up endpoint scaffolding and secret manager stubs
  - Write in-memory token relay logic (Fitbit, Oura)
  - Enforce zero-persistence policy, strong logging, and audit hooks
  - Implement automated security/hygiene checks
- **2.2. Integration and security tests for backend**
  - Write automated functional tests for endpoint flows
  - Fuzz edge-cases; simulate breach scenarios

## 3. iOS App — Foundational Flows
- **3.1. App skeleton and navigation setup**
  - Scaffold main app screens
  - Configure navigation (onboarding, dashboard, settings, data entry)
- **3.2. Core onboarding flows**
  - Account creation (email/password)
  - Legal screens (ToS, HIPAA/GDPR consent)
  - Profile setup (required/optional fields, validation)
  - Device connection/mode selection UI
  - Baseline assessment entry
  - Accept initial training plan
- **3.3. HealthKit integration**
  - Read/write access to key metrics (steps, heart rate, sleep, weight)
  - Handle permissions, onboarding explanation, and error UI

## 4. OAuth Relay Backend <-> iOS App Flows
- PKCE/ASWebAuthenticationSession flow for Fitbit/Oura (w/deep-link and Keychain token storage)
- Ephemeral relay and deletion of tokens/data
- Connection status and disconnect flows

## 5. Data Storage, Privacy, Security
- On-device secure storage (Keychain for tokens, encrypted files for data)
- Implement data export/import logic; account deletion/6mo-lifecycle
- Enforce no GPS/raw sensor uploads
- Automated unit and integration tests

## 6. Dashboard & KPI Engine (iOS)
- Render 5 KPI cards
- Color-coded state logic per metric
- Tap for detail, quick action flows (log meal, start workout, view chart)

## 7. Nutrition Engine
- Mifflin–St Jeor BMR, TDEE, macro calculators
- Dynamic targets, adaptive adjustment rules, safety constraints
- Manual and device-driven food input flows

## 8. Workout Engine
- Program/plan generator (goal-based)
- Session builder, tracking, baseline test logic
- Progression/autoregulation, workout logs, injury mods

## 9. Alerts, Notifications & Safety Logic
- Critical threshold monitoring (HR, calorie, SpO2, rapid weight loss, etc.)
- Local push notification engine (privacy, DND-aware)
- Alert UX (full-screen, tap-through guidance)

## 10. PDF Export & Sharing
- Summary PDF export engine (password protect, content structure, CSV attachments)
- Scheduling logic
- Consent reminders pre-export

## 11. Final QA, Compliance, Accessibility, Legal Copy
- Security review, penetration test, privacy legal review
- Accessibility validation
- Translate and QA all legal/consent copy in app/on export

---

## Iterative Decomposition (Chunk → Steps → Micro-Steps)
Below, each of the above domains is broken into incrementally smaller, strongly testable steps, grouped in recommended order of execution:

### 1. Initial Project Setup — Micro-Steps
1. Create repo(s); initialize with README, .gitignore
2. Set up CI/CD for iOS (GitHub Actions/Bitrise) and backend (GitHub Actions)
3. Define branch protection, merge policies
4. Decide on package managers (SwiftPM for iOS, npm for backend)
5. Base folder and code structure for each app/subrepo
6. Create first commit: initial app scaffolds (iOS: App, Web: placeholder, Backend: minimal relay folder)

### 2. Backend OAuth Relay — Micro-Steps
1. Author OpenAPI yaml for endpoints
2. Scaffold endpoints with dummy logic (Node or Go)
3. Set up logging, secrets stubs, security middlewares
4. Implement /oauth/start (Fitbit, Oura)
5. Implement /oauth/callback for PKCE exchange (deep-link return to app)
6. Add memory zeroization, no-token-persistence logic
7. End-to-end test flow with mock data
8. Security/fuzz tests on auth flows

### 3. Onboarding Flow — Micro-Steps
1. Design wireframes/screens for onboarding in Figma/Sketch
2. Implement screen navigation logic (SwiftUI/NavStack)
3. Add account creation UI + validation
4. Add ToS/HIPAA screen + enforcement logic
5. Profile fields (required first, optional next) + validation
6. Device connect selector (Apple Health/Fitbit/Oura/manual)
7. Baseline assessment UI and storage
8. Accept initial plan screen
9. End-to-end onboarding test (happy path and skips)

### 4. Health Data Integration — Micro-Steps
1. Set up HealthKit project capability/entitlements
2. Implement permission flow and error states
3. Read steps, heart rate, sleep, weight
4. Write back data (if allowed)
5. Unit, integration, permissions regression tests

### 5. Data Privacy & Storage — Micro-Steps
1. Integrate Keychain for tokens
2. Encrypted file storage for PHI
3. Implement export (PDF, CSV) logic with password protection
4. Account deletion and 6mo lifecycle delete logic
5. Test PHI never leaves device—static analysis/test coverage

### 6. KPI Dashboard — Micro-Steps
1. Design and implement dashboard/card UI
2. Wire up sample KPI logic (e.g., workouts this week)
3. Color status logic for each card
4. Tap logic for detail/quick actions
5. Edge case and notification tests

### 7. Nutrition Engine — Micro-Steps
1. Implement BMR/TDEE calculators
2. Macro split and target logic (safe constraints)
3. Manual food entry flow
4. Device-driven nutrition integration
5. Adaptive progression and adjustment rule implementation
6. Safety edge tests (floor/bounds enforcement)

### 8. Workout Engine — Micro-Steps
1. Assessment logic, store initial test data
2. Build program generator (goal-configurable)
3. Session planner UI
4. Autoregulation/progression logic
5. Exercise tracking flows, workout logging
6. Test with and without equipment/injury scenarios

### 9. Critical Alerts & Notification — Micro-Steps
1. Implement alert-engine for each critical threshold
2. Wire to local push notification engine
3. Design alert modals/screens
4. Safety constraint test cases
5. Privacy (no PHI in notifs), DND-guarding

### 10. PDF Export — Micro-Steps
1. PDF summary generator (data templating)
2. Add password prompt flow
3. Attach CSV (if requested)
4. Scheduling (monthly, on-demand)
5. Export/share UX, consent checklist

### 11. Compliance & Final QA — Micro-Steps
1. Security & privacy review checklist
2. Accessibility audits
3. QA testing (manual and automated)
4. Final copy/consent/legal review
5. Release criteria checklist

---

This granular breakdown will be further iterated and sized by engineering as implementation progresses; steps combine safe isolation, direct testability, and continuous product progress toward MVP.

_End of plan.md_
