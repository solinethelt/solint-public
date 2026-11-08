# SOLINT PUBLIC

Independent public-facing SOLINT application. This repository is separate from SOLINT NETWORK.

## Current status

Phase 1 foundation scaffold is in progress. The initial Vite + React + TypeScript application shell is intentionally minimal. It contains no production authentication, health-record data, offline medical-data support, or live NETWORK integration.

## Requirements

- Node.js 20 LTS (or a currently supported Node.js LTS release)
- npm

## Run locally

```bash
npm install
npm run dev
```

The development server binds to `127.0.0.1` by default.

## Validation

```bash
npm run typecheck
npm run build
```

These commands are documented but have not yet been executed in this environment. Add the generated `package-lock.json` from a local `npm install` in the next implementation commit to make installs reproducible.

## Environment configuration

Copy `.env.example` to `.env.local` if local configuration is needed. Values prefixed with `VITE_` are exposed to browser code and must never contain secrets. The API base URL is intentionally unset until an approved API contract and deployment endpoint exist.

## Security and scope

- No secrets belong in frontend source or Vite environment variables.
- No real patient information should be used in development or tests.
- The current service worker/PWA data-caching behavior is not implemented; no medical data is supported offline.
- Authentication UI and server integration are not implemented.
- SOLINT NETWORK integration must use an approved, documented API; direct database access is prohibited.
