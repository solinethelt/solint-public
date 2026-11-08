# SOLINT PUBLIC — Temporary Authentication Policy

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-SEC-AUTH-002 |
| Version | 1.0 |
| Status | Draft — Architecture and Security Review Required |
| Scope | SOLINT PUBLIC |
| Policy type | Temporary authentication policy |
| Related documents | `docs/architecture/SOLINT_PUBLIC_MASTER_ARCHITECTURE.md`; `docs/architecture/SOLINT_PUBLIC_PHASE_0_TECHNICAL_VALIDATION_AND_SECURITY_THREAT_MODEL.md`; `docs/security/SOLINT_PUBLIC_SECURE_IDENTITY_AND_OFFLINE_ARCHITECTURE.md` |

## 1. Purpose and status

This document records the product decision for temporary password-based authentication and defines requirements for subsequent architecture and security review. It is a specification only: it does not assert that any control is implemented, technically approved, or production-ready.

The temporary policy supersedes the earlier requirement to use SMS OTP for every online login only while this policy is active. SMS OTP remains a future capability and must not be described as currently operational or available. A change to this policy requires a documented decision and approval by the product, architecture, and security owners.

## 2. Temporary product policy

- Users register with an email address and mobile phone number.
- Each user chooses an individual password for their own account. A password is not shared, universal, or system-assigned; different users may independently choose the same password.
- Temporary online login uses the registered mobile number and that account's password.
- Passwords must be hashed using Argon2id before persistent storage. Plaintext or reversibly encrypted passwords are prohibited.
- Login and abuse-prevention decisions are enforced server-side.
- Users can sign out; sessions can be revoked server-side.
- Password recovery is a separate workflow and is blocked until its channel and security rules are approved.
- SMS OTP may be introduced in the future through an approved server-side authentication policy without replacing the entire Authentication capability.

## 3. Registration and verification

Email ownership verification and mobile-number ownership verification are separate states. Email verification must not be described as legal identity verification.

The mechanism for verifying mobile-number ownership is unresolved until an approved mechanism and provider exist. SMS verification must not be represented as operational before the service and flow have been selected, configured, and validated. The account activation rules, required verification states, and access permitted to pending accounts require explicit approval.

The server must consistently normalize and enforce uniqueness for account identifiers. Registration and authentication responses must avoid disclosing whether an account exists where that could enable enumeration.

## 4. Password, login, and abuse prevention

Use a maintained Argon2id implementation and its safe password-verification API. Each password must have an independently generated salt. Hash parameters, upgrade strategy, and any pepper/key-management approach require a separate security decision; this document approves no production parameter values.

Passwords, password hashes, session credentials, and recovery secrets must not be logged, placed in analytics, returned by APIs, or included in error/trace output. Password length and user guidance must be decided in the security design; avoid arbitrary composition rules that encourage predictable passwords.

The server must enforce rate and abuse controls for login and recovery. Thresholds, time windows, temporary throttling or lockout behavior, distributed-attempt handling, monitoring, and safe behavior when a rate-limiting dependency fails remain pending review. Controls must consider denial-of-service risk against legitimate users; permanent or easily triggered lockout must not be introduced without review.

Authentication establishes identity, not authorization. Every protected operation must enforce account status, valid session, resource ownership, permissions, consent, and tenant boundaries as applicable on the server.

## 5. Sessions and sign-out

Session creation, validation, expiry, renewal, sign-out, and revocation must be server-controlled. Sign-out must invalidate the server-side session; clearing browser state alone is insufficient. Session mechanism, credential storage, lifetime, idle timeout, renewal/rotation, CSRF protections where applicable, and revocation after password change or reset are unresolved architecture decisions.

Session credentials must not be stored in insecure browser storage or written to logs. This temporary policy does not itself implement or guarantee the separate one-active-trusted-device requirement.

## 6. Password recovery — implementation blocker

Password recovery is independent of login. Its channel, identity-assurance requirements, expiry, one-time-use enforcement, notification, rate limits, and session-revocation behavior are unresolved and must be approved before recovery is implemented.

Email must not automatically become a recovery method merely because it was collected or verified. SMS-based recovery must not be represented as available until the provider and flow are approved. Recovery must not disclose account existence or bypass account verification, authorization, audit, or applicable device restrictions. Recovery credentials must be random, single-use, time-limited, securely stored in non-recoverable form, and invalidated after use or replacement.

## 7. Future SMS OTP migration

Authentication must be a modular capability that separates account identity, authentication policy, authentication mechanism, session management, trusted-device management, authorization, offline access, and notification-provider adapters. These are logical boundaries and do not require microservices or a second independent authentication system.

SMS OTP activation requires an explicit, reviewed server-side policy. Provider selection, OTP generation and storage, expiration, single-use behavior, attempt and request limits, delivery failures, recovery, and SIM-swap risks require separate decisions and tests. Do not silently enable password and OTP as simultaneous methods. A separate decision must determine whether password remains available for recovery or as an alternative after OTP is introduced.

## 8. Trusted-device, offline, and system boundaries

The product requirement remains a maximum of one active trusted device per account. Device registration, replacement, revocation, and the invariant must be authorized and enforced server-side. Device replacement must be securely authorized and atomic. This policy does not claim that device management is implemented.

The maximum offline authorization period remains 72 hours. Local unlocking may use a PIN or supported biometrics only after technical feasibility and security review. Offline access must not bypass account authorization, session/device restrictions, expiry, or server-side policy. The 72-hour rule, local encryption/key management, PWA limitations, and residual access on a fully offline device remain subject to separate technical validation; no security guarantee is implied here.

SOLINT PUBLIC and SOLINT NETWORK remain independent systems. PUBLIC must never access NETWORK's internal database directly. Cross-product communication must use approved, authenticated, authorized, and documented APIs.

## 9. Data and API design gates

This document does not approve a database schema, migration, authentication provider, API implementation, or production configuration. Before implementation, review and approve:

- Account identifier normalization and uniqueness.
- Password hash format/version and upgrade strategy.
- Email/mobile verification and account-activation states.
- Rate-limit and lockout representation and transaction boundaries.
- Session credential storage, expiry, renewal, and revocation model.
- Recovery challenge lifecycle and one-time-use enforcement.
- Audit event minimization, access, and retention.
- API contracts, validation, error behavior, authorization, versioning, and relevant idempotency.

No example field list or conceptual data model elsewhere should be treated as an approved schema without this review.

## 10. Audit and privacy

Record only necessary security events, such as registration/verification state changes, authentication outcomes, abuse-control events, password changes/recovery, session lifecycle, and authentication-policy changes. Never log passwords, password hashes, OTPs, reset secrets, session credentials, private keys, or unnecessary health content.

Any collection of IP addresses, user-agent data, or derived identifiers requires a documented purpose, access policy, retention period, and privacy review. Hashing an identifier does not automatically make it anonymous.

## 11. Required tests and acceptance gates

Before a related implementation task is accepted, its approved test plan must cover:

- Argon2id hashing and verification through the selected maintained library; plaintext is never persisted or logged.
- Correct and incorrect password behavior, identifier normalization, and duplicate handling.
- Non-enumerating login and recovery responses.
- Server-side rate limiting, repeated and concurrent failed attempts, and documented dependency-failure behavior.
- Session creation, sign-out, expiry, revocation, and password-change/reset consequences.
- Recovery single-use, expiry, replay rejection, rate limiting, and approved session revocation.
- Protected-route authorization and resource ownership on the server.
- Absence of secrets in logs, traces, analytics, errors, and insecure browser storage.
- No direct PUBLIC database access to NETWORK.
- Authentication configuration cannot silently enable an unapproved method.

A product decision recorded, architecture reviewed, security reviewed, implementation completed, tests passed, and production approved are distinct statuses and must be tracked separately. Passing this document's review does not itself approve implementation or production deployment.

## 12. Unresolved decisions — do not guess

1. Mobile-number ownership verification mechanism before an SMS service is approved.
2. Account activation and permissions for unverified/pending accounts.
3. Password recovery channel and identity-assurance process.
4. Authentication provider and backend architecture.
5. Session mechanism, storage, expiry, renewal, and revocation rules.
6. Argon2id production parameters and hash-upgrade strategy.
7. Login/recovery rate-limit thresholds and lockout behavior.
8. Database schema, migration tooling, and API contracts.
9. Phase numbering and exact scope: existing documents place OTP, trusted-device enforcement, and offline capabilities in different phases.
10. How temporary password authentication interacts with trusted-device and offline authorization requirements.

## 13. Status

- **Product decision:** Recorded as temporary.
- **Architecture review:** Pending.
- **Security review:** Pending.
- **Mobile verification:** Unresolved.
- **Password recovery:** Unresolved and blocked from implementation.
- **Argon2id parameters:** Pending security approval.
- **Session design:** Pending architecture and security approval.
- **Database/API/provider selection:** Pending.
- **Implementation:** Not asserted by this document.
- **Production approval:** Not granted.
