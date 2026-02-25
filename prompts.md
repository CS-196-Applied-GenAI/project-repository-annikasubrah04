# prompts.md

Below is a series of step-by-step prompts for a code-generation LLM to implement the "Personal Health & Fitness App — Project Blueprint" in a test-driven, best-practice-oriented, and incremental way. Each prompt builds on the previous, ensures tests are in place from the start, and avoids introducing non-integrated or “dangling” code. Work in the order specified and ensure that at the end of each sequence, all code is connected and working together.

---

## 1. Initial Project Setup & Foundations

### 1.1 Initialize Repositories and Tooling

```
Set up the project’s root directory (monorepo or multiple subrepos as needed) with a minimal README.md describing the vision and goals. Add a .gitignore suitable for iOS (Swift), Web (React), and Backend (Node/Go). Scaffold the folder structure (e.g., /ios, /web, /backend). For each subproject, initialize its respective package manager (SwiftPM for iOS, npm for backend). Write a test to confirm that the project structure exists and that each subproject can pass an empty test.
```

### 1.2 Set Up CI/CD

```
Add basic CI/CD pipeline definitions for iOS (GitHub Actions or Bitrise) and backend (GitHub Actions). Each pipeline should run basic checks and tests for its subproject only. Add a test workflow for each pipeline that runs on every commit and PR. Create an integration test which fails if any subproject’s test fails. Ensure all jobs succeed when code and tests are passing.
```

### 1.3 Branch Protection and Contribution Guidelines

```
Configure branch protection rules and merge policies (e.g., pull request required, passing tests required). Add a CONTRIBUTING.md with clear instructions for running tests, submitting PRs, and agreeing to code standards. Write a test that lints the code according to the project’s coding standards and fails if violations are found.
```

---

## 2. Core Backend (OAuth Relay)

### 2.1 Scaffold OAuth Backend API

```
Draft an OpenAPI YAML spec for the backend’s OAuth relay endpoints: /oauth/start and /oauth/callback (for Fitbit and Oura). Scaffold these endpoints in your backend project (Node/Go). Implement stubbed handlers that return 501 Not Implemented. Add automated tests confirming correct routing and OpenAPI schema conformance. Tests should confirm the endpoints exist and return the expected status codes.
```

### 2.2 Secret Management and Security Stubs

```
Add logic for stubbed secret management (using environment variables or a dummy secret manager). Introduce logging (with levels and structured logs) and middleware for basic security (input validation, request logging). Write tests covering error handling, unauthorized access, and correct logging output for both endpoints.
```

### 2.3 Implement In-Memory Token Relay Logic

```
Implement ephemeral, in-memory token handling logic for relaying OAuth tokens securely (support Fitbit and Oura). Enforce zero-persistence (no disk/database writes). Integrate logging/audit hooks for every token event (start, relay, delete). Add unit and integration tests for each code path: handling, relaying, and secure destruction of tokens. Fuzz tests simulate edge conditions (e.g., bad tokens, unexpected input).
```

---

## 3. iOS App — Foundational Flows

### 3.1 Scaffold iOS App Skeleton

```
Create the initial iOS app project (Swift) with basic files. Scaffold empty SwiftUI screens for Onboarding, Dashboard, Settings, and Data Entry. Add navigation logic between these screens using NavStack. Write a suite of UI unit tests to verify each screen is present, loads, and basic navigation works.
```

### 3.2 Onboarding Flow: Account Creation

```
Implement the account creation UI with form fields for email and password, including client-side validation. Add functional tests that verify form validation and correct navigation upon successful creation. Wire this into the main app navigation. Do not include backend logic yet; mock success and error responses in tests.
```

### 3.3 Legal and Profile Setup Flows

```
Add onboarding screens for ToS and HIPAA/GDPR consent. Enforce that users cannot continue without accepting. Add profile setup (required/optional fields) with field validation. Write integration tests for linear flows and skips; ensure unenforced paths cannot be accessed if prerequisites aren’t met.
```

### 3.4 Device Connection and Baseline Assessment Flows

```
Implement UI for device connection selection (Apple Health, Fitbit, Oura, Manual). Add baseline assessment entry screen. Write tests for each branch of device selection and assessment input, and test the whole onboarding process end-to-end (happy path and alternate flows).
```

---

## 4. HealthKit and OAuth Integration

### 4.1 Integrate HealthKit (iOS)

```
Set up HealthKit entitlements and implement permission request flow. Add read logic for steps, heart rate, sleep, and weight. Handle permission denied and error states in the UI. Write tests for permission requests, error cases, and sample reads on simulator/mock data.
```

### 4.2 Implement OAuth Relay App Flows

```
Implement PKCE/ASWebAuthenticationSession flow for Fitbit/Oura device connections, using the backend relay endpoints. Securely store resulting tokens in Keychain. Include logic for ephemeral token handling and deletion. Unit and integration test all flows: successful connection, error cases, and disconnect logic. Use mocks for relay/backend responses.
```

---

## 5. Data Storage, Privacy, Security

### 5.1 Secure Storage Logic

```
Implement on-device secure storage of tokens using Keychain, and encrypted file storage for sensitive user data (PHI). Add logic for export (PDF, CSV), with password protection. Write automated tests to verify encryption, key management, and correct access controls on exports.
```

### 5.2 Data Lifecycle and Privacy Compliance

```
Implement comprehensive account deletion logic and 6-month data lifecycle autodelete policy. Run static analysis tools to verify no GPS/raw sensor data is uploaded and that no PHI leaves the device. Write tests for all permitted and forbidden flows. Test that deletion and export functions work end to end.
```

---

## 6. Dashboard & KPI Engine

### 6.1 Implement KPI Dashboard and Card UI

```
Design and build the dashboard view with five KPI cards (e.g., steps, sleep). Color code each card according to metric state. Wire up tap logic for details and quick actions (log meal, start workout). Write UI and logic tests for each KPI, edge cases, and navigational flows.
```

---

## 7. Nutrition Engine

### 7.1 Implement BMR/TDEE & Macro Calculators

```
Implement Mifflin-St Jeor BMR and TDEE calculators, and logic for macro splits and safe constraint enforcement. Write tests for all formulas, safety floors/ceilings, and adaptive rule logic. Integrate calculations into onboarding/profile flow as appropriate.
```

### 7.2 Food Entry and Adaptive Targets

```
Build manual and device-driven food entry flows. Implement adaptive targets and adjustment logic. Write integration tests for all user input scenarios and ensure device, manual, and adaptive paths are covered.
```

---

## 8. Workout Engine

### 8.1 Assessment, Program Generator, and Logging

```
Implement baseline fitness assessment logic with data storage. Build goal-configurable program generator and session planner UI. Add logic for progression/autoregulation, track workout sessions, and log exercises. Test all scenarios, including with/without equipment, injury modes, and edge progression cases.
```

---

## 9. Alerts, Notifications, and Safety

### 9.1 Implement Critical Alert and Notifications Engine

```
Develop logic to monitor critical health KPIs (heart rate, calorie, SpO2, rapid weight change, etc.). Wire alert generation to local push notifications. Design DND-aware, privacy-first alert UX. Test all alert scenarios; verify no PHI leaks to notifications, and DND/push control functions as intended.
```

---

## 10. PDF Export & Sharing

### 10.1 Implement PDF Summary Export

```
Add engine for generating password-protected PDF summaries of health data, with structured content and optional CSV attachments. Include scheduling logic for monthly/on-demand exports. Add a pre-export consent checklist. Write export and sharing tests, including accessibility of password-protected and unprotected exports.
```

---

## 11. Final QA, Compliance, Accessibility, Legal Copy

### 11.1 Security, Accessibility, and Compliance

```
Run a complete security and privacy review, including penetration tests and static code analysis. Perform accessibility audits for all app screens and flows. Add automated and manual QA coverage for edge and regressions. Ensure all legal copy/consent text meets compliance, is localized, and tested within all main and export flows. Write a final release-readiness test verifying all previous tests pass, compliance checklists are met, and release blockers are flagged if found.
```

---

# Final Integration Prompt

```
Verify that all components and flows (backend, iOS, integrations, dashboards, exports) are wired together and work seamlessly as a whole. Run a full suite of integration and regression tests simulating both typical and edge-case user behaviors from onboarding to data export, device connect/disconnect, and alerts. Confirm the absence of any orphaned or non-integrated code. Confirm every feature is accessible through the UI or API as appropriate and that the app is MVP-ready.
```