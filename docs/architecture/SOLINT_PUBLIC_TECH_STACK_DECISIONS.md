# SOLINT PUBLIC
# Technology Stack Decisions

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-TECH-001 |
| Version | 1.0 |
| Status | COMPLETE FOR PHASE 0 BASELINE |
| Date | 2026-11-08 |
| Project | SOLINT PUBLIC |
| Platform | Secure Offline-First PWA |

---

## 1. Decision Summary

This document records the approved technology baseline for the Phase 0 design gate. The goal is to keep the system secure, mobile-first, and easy to maintain while avoiding premature distributed complexity.

### Approved baseline

- Frontend: React + TypeScript + Vite + Tailwind CSS + TanStack Query + React Router
- PWA: Service Worker + Web App Manifest + IndexedDB + Web Crypto API
- Backend: Node.js + TypeScript + NestJS (recommended) or Fastify if justified
- Database: PostgreSQL
- Object storage: Private object storage (S3-compatible or equivalent)
- API contract: REST + OpenAPI + versioning
- Identity: managed authentication provider with SMS OTP capability + custom device trust enforcement
- Audit + monitoring: structured logs, event store, metrics, tracing

## 2. Why this stack

### 2.1 Frontend choice

React and TypeScript are a practical baseline for a PWA with rich form flows, secure local state, and component reuse. The project is mobile-first and requires accessibility support, number-heavy OTP flows, and a robust PWA lifecycle.

### 2.2 Backend choice

A modular monolith is preferred for the first implementation. It keeps boundaries clear without introducing microservice overhead. NestJS provides strong validation, modular structure, and predictable API conventions.

### 2.3 Data choice

PostgreSQL remains the source-of-truth relational database. It supports strict constraints, audit tables, row-level authorization patterns, and reliable transactions for the one-device rule.

### 2.4 Offline first choice

IndexedDB and Web Crypto API are selected because they provide the required local data capabilities for the PWA. However, they do not offer hardware-backed security guarantees by themselves, which is why server-side enforcement remains mandatory.

## 3. Approved standards

| Layer | Decision | Reason |
|---|---|---|
| Frontend | React + TypeScript | Productivity and ecosystem maturity |
| Build | Vite | Fast iteration and good PWA support |
| UI | Tailwind CSS | Design consistency and speed |
| State/data | TanStack Query | Solid API state management |
| Routing | React Router | Standard, predictable navigation |
| PWA | Web App Manifest + SW | Installability and offline support |
| Local persistence | IndexedDB | Structured local storage |
| Encryption | Web Crypto API + AES-GCM | Needed for encrypted local health data |
| Backend | NestJS/Node.js + TypeScript | Secure, modular, maintainable |
| DB | PostgreSQL | Strong relational integrity and auditability |
| File storage | Private object storage | Fits approved document and attachment workflows |
| API | REST + OpenAPI | Compatibility and versioning |
| Validation | Zod | Strong schema validation |

## 4. Non-goals and constraints

- No direct PUBLIC access to SOLINT NETWORK database.
- No hidden architectural drift from the approved modular boundaries.
- No microservices before a real operational requirement exists.
- No storage of OTPs, private keys, or sensitive medical content in logs.
- No claim of hardware-backed key protection unless the chosen browser and platform are technically validated.

## 5. Final decision register

| ID | Decision | Status |
|---|---|---|
| TD-001 | Modular Monolith for initial backend | APPROVED |
| TD-002 | React + TypeScript frontend | APPROVED |
| TD-003 | PWA as initial deployment platform | APPROVED |
| TD-004 | PostgreSQL database | APPROVED |
| TD-005 | SMS OTP for every online login | APPROVED |
| TD-006 | One active trusted device | APPROVED |
| TD-007 | Maximum 72-hour offline authorization | APPROVED |
| TD-008 | IndexedDB for offline structured data | APPROVED |
| TD-009 | Web Crypto API for local encryption | APPROVED |
| TD-010 | AES-GCM authenticated encryption | APPROVED |
| TD-011 | API-first NETWORK integration | APPROVED |
| TD-012 | Direct PUBLIC database access to NETWORK | PROHIBITED |
| TD-013 | Authentication provider | RECOMMENDED: managed provider + custom device trust |
| TD-014 | SMS provider | RECOMMENDED: validated provider with rate limiting |
| TD-015 | Key management strategy | APPROVED: server-controlled key hierarchy and local encrypted storage |
| TD-016 | Offline expiration enforcement | APPROVED: server-aware local authorization and 72h max |
| TD-017 | Backend framework | RECOMMENDED: NestJS |
| TD-018 | File storage provider | RECOMMENDED: private S3-compatible storage |

## 6. Implementation gate

The stack is approved for Phase 0 as a design baseline. It is not an approval for production rollout. Final provider selection and security validation must still be completed in staging before work on Phase 1 begins.

---

End of document.
