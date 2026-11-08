# SOLINT PUBLIC — Phase 1: Foundation & Application Shell

| Field | Value |
|---|---|
| Version | 1.0 |
| Status | IN PROGRESS — conditional; scaffold committed, validation pending |
| Repository | `solinethelt/solint-public` |
| Scope | Independent web application foundation |

## Purpose and guardrails

Build the independent PUBLIC application shell without implementing healthcare business workflows. Keep PUBLIC deployment, configuration, and lifecycle separate from SOLINT NETWORK. Any future cross-product integration must use an approved API; direct database access and reuse of NETWORK credentials are prohibited.

This phase does not implement real authentication, OTP, trusted-device enforcement, health records, AI features, live NETWORK integration, medical-data caching, or offline medical-record synchronization. Unfinished routes must remain clearly marked placeholders when they are added.

## Repository inspection and coordination

The existing repository initially contained architecture and planning documents but no application scaffold. The Phase 0 deliverables are in a separate documentation branch and are design artifacts, not evidence of builds, provider tests, or production approval. A separate temporary-password policy in another branch conflicts with the confirmed product requirement of mobile SMS OTP for every online login. Phase 1 therefore does not select or implement either login mechanism; authentication remains an integration boundary pending an explicit, reconciled decision.

The draft OpenAPI file on the Phase 0 deliverables branch uses an example domain and proposed endpoints. It is not treated as an approved NETWORK contract or a production endpoint.

## Execution sequence

### Step 1 — Resolve design gate (before production-oriented functionality)

- [ ] Product/security owners reconcile the temporary-password branch with the SMS OTP requirement; record which policy governs and its scope.
- [ ] Approve the application stack, runtime, hosting/deployment strategy, and ownership of environment configuration.
- [ ] Review Phase 0 artifacts and mark each decision as approved, pending, or blocked based on evidence; do not infer provider approvals from draft recommendations.
- [ ] Confirm an API contract and integration owner before any live NETWORK connectivity.

### Step 2 — Foundation scaffold (this implementation slice)

- [x] Add a Vite + React + TypeScript app entry point and strict TypeScript configuration.
- [x] Add a minimal Persian-first shell and global accessibility focus style.
- [x] Add a PWA manifest containing no health data or credentials.
- [x] Add `.gitignore`, a safe environment policy, and development/validation README guidance.
- [ ] Run `npm install`, commit the generated lockfile, then run typecheck and production build in CI or a local Node.js environment.
- [ ] Add automated tests and CI; no test result is claimed by this commit.

### Step 3 — Shell and navigation

- [ ] Add responsive header and navigation for Home, Health Record, AI Health, My Care, Family, Discovery, Community, and Profile.
- [ ] Add explicit not-yet-available placeholders and a not-found route; do not fabricate patient content.
- [ ] Add loading, error, empty, and online/offline network-status states.

### Step 4 — Localization and design system

- [ ] Add translation resources for `fa`, `en`, `ar`, and `fr`; Persian is default.
- [ ] Ensure Persian/Arabic use RTL and English/French use LTR, including runtime language switching.
- [ ] Add locale-aware date/number formatting and reusable design tokens and components.
- [ ] Validate keyboard access, labels, contrast, responsive behavior, and screen-reader navigation.

### Step 5 — Safe PWA and API boundary

- [ ] Add installability assets and service-worker registration/update behavior after defining an explicit shell-only cache allowlist.
- [ ] Never cache tokens, private documents, plaintext medical responses, or authenticated API responses.
- [ ] Add network state detection without implying offline access to health records.
- [ ] Add a typed API client with configurable, unset-by-default public base URL, timeouts, cancellation, and safe error mapping. Do not invent NETWORK endpoints.

### Step 6 — Evidence and final gate

- [ ] Record actual install, typecheck, build, and automated-test results.
- [ ] Validate manifest and service-worker behavior in supported browsers, including real iOS Safari and Android Chrome checks before claiming platform support.
- [ ] Review browser storage/cache for absence of sensitive data and inspect the built bundle for accidental secrets.
- [ ] Publish repository structure, build/test, security, PWA, localization, and known-limitations reports.
- [ ] Do not mark Phase 1 complete or production-approved without evidence and review.

## Scaffold details

The initial commit establishes `src/app`, `src/styles`, `public`, Vite configuration, TypeScript settings, and npm scripts. Other feature modules are intentionally not created empty. The API base URL is not set; Vite `VITE_` values are public client configuration and never secrets. The current manifest does not constitute verified installability, and a service worker is intentionally not registered until its cache policy and tests are added.

## Acceptance criteria

Phase 1 is complete only after the user-provided acceptance criteria are evidenced by passing builds/tests and review. This scaffold alone does not satisfy the phase completion gate.
