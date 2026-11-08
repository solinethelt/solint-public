در ادامه، متن آمادهٔ فایل `docs/security/SOLINT_PUBLIC_TEMPORARY_AUTHENTICATION_POLICY.md` آمده است. وضعیت سند عمداً «پیش‌نویس برای بازبینی معماری و امنیت» ثبت شده؛ تصمیم محصول دربارهٔ رمز عبور موقت است، اما جزئیات فنی و مسیر بازیابی هنوز باید پیش از پیاده‌سازی تأیید شوند.

````markdown name=docs/security/SOLINT_PUBLIC_TEMPORARY_AUTHENTICATION_POLICY.md
# SOLINT PUBLIC — Temporary Authentication Policy

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-SEC-AUTH-002 |
| Version | 1.0 |
| Status | Draft — Product decision recorded; architecture and security review required |
| Scope | SOLINT PUBLIC |
| Policy type | Temporary authentication policy |
| Related documents | `docs/architecture/SOLINT_PUBLIC_MASTER_ARCHITECTURE.md`; `docs/architecture/SOLINT_PUBLIC_PHASE_0_TECHNICAL_VALIDATION_AND_SECURITY_THREAT_MODEL.md`; `docs/security/SOLINT_PUBLIC_SECURE_IDENTITY_AND_OFFLINE_ARCHITECTURE.md` |

---

## 1. Purpose

This document records the temporary SOLINT PUBLIC authentication policy approved by the product owner and defines the security and architecture requirements that must be resolved before implementation.

This is a policy specification. It does not claim that authentication, password storage, account recovery, sessions, or any related security control has been implemented or tested.

The policy is temporary because SMS OTP is the intended future online authentication method once an SMS provider and the associated security and operational controls have been approved.

## 2. Decision Summary

For the temporary authentication phase:

1. Each user registers with an email address and mobile phone number.
2. Each user chooses an individual password for their own account.
3. The user signs in with their mobile phone number and password.
4. Passwords must be hashed using Argon2id before persistent storage.
5. The system must enforce server-side controls against repeated failed attempts and password guessing.
6. Users must be able to sign out, and the server must be able to revoke sessions.
7. Password recovery must use a separate, secure workflow.
8. The authentication architecture must support enabling SMS OTP later through an explicitly approved server-side policy, without replacing the whole authentication subsystem.

A user-chosen password is unique to that account in the system’s account model. “Fixed password” does not mean a shared, universal, or default password. Users may independently choose the same password; the system must never assign a shared password.

## 3. Policy Status and Precedence

This temporary policy supersedes the earlier requirement that every subsequent online login must use SMS OTP **only for the period in which this temporary policy is approved and active**.

It does not silently remove or redefine the following product requirements:

- SOLINT PUBLIC and SOLINT NETWORK remain independent systems.
- SOLINT PUBLIC must not directly access SOLINT NETWORK’s internal database.
- The one-active-trusted-device policy remains a separate product requirement and must be implemented only in its approved phase.
- Offline access and the 72-hour authorization policy remain subject to separate technical validation and approval.
- SMS OTP remains a future authentication option, not an implemented or available capability under this temporary policy.

Any change to this policy requires a documented decision and approval by the relevant product, architecture, and security owners.

## 4. Registration and Verification

### 4.1 Registration data

Registration is expected to collect:

- Email address
- Mobile phone number
- A password chosen by the user
- Any additional profile or consent data required by the approved product scope

The exact set of identity/profile fields must be defined by the approved product requirements and data-minimization review. This document does not authorize collecting government identity data or other unnecessary sensitive information.

### 4.2 Contact verification

Email ownership and mobile-number ownership are separate verification states.

- Email verification must not be described as legal identity verification.
- A mobile number must not be considered verified unless an approved verification mechanism has successfully verified its ownership.
- SMS verification must not be represented as operational until an SMS provider and verification flow have been selected, configured, and validated.
- An account must not receive access to protected resources until the account’s required verification and activation conditions have been satisfied.

The verification method, activation rules, expiry periods, resend limits, and recovery implications must be documented before implementation.

### 4.3 Account identifiers

The implementation design must define canonicalization and uniqueness rules for email addresses and phone numbers. These rules must be applied consistently at the server and persistence layers.

Authentication responses must not reveal whether an email address or phone number is registered when doing so could enable account enumeration.

## 5. Password Requirements

### 5.1 Password selection and handling

- The user chooses their own password during registration or an approved password-setting flow.
- The application must never use a shared, universal, hard-coded, or default password.
- Passwords must be transmitted only over an approved secure transport.
- Passwords must never be logged, placed in analytics, included in error messages, or returned by an API.
- Passwords must not be stored in plaintext or in a reversibly encrypted form.
- Passwords must be redacted from tracing, debugging, and telemetry.

Password length, allowed characters, breached-password screening, and user-facing password guidance must be decided in the security design. The policy must avoid arbitrary composition rules that encourage weak, predictable passwords.

### 5.2 Password hashing

- Password hashes must use Argon2id through a maintained, reviewed implementation.
- Hashing parameters must be selected through a security review and performance testing on the target deployment environment.
- The implementation must use a unique salt for each password hash, as supported by the selected Argon2id library.
- Parameters must be versioned or otherwise identifiable so hashes can be upgraded when the password is next verified.
- Password verification must use the chosen library’s safe verification function; application code must not implement a custom hash comparison.
- A pepper, if adopted, must be stored and managed separately from the password database using an approved secret-management mechanism. A pepper is not a substitute for Argon2id or access controls.
- Plaintext passwords, reset tokens, and session credentials must not be written to database records, logs, backups as application data, or test fixtures.

No Argon2id parameter values are mandated by this document. The selected values must be recorded in an approved technical decision and periodically reviewed.

## 6. Login and Abuse Prevention

### 6.1 Login flow

The temporary online login flow is:

1. The user submits a mobile number and password.
2. The server validates and normalizes the request.
3. The server applies rate and abuse controls.
4. The server verifies the account state and password hash.
5. If authentication succeeds, the server creates a session according to the approved session design.
6. If authentication fails, the response does not disclose whether the account exists, is unverified, or has an incorrect password.
7. Security-relevant outcomes are recorded without storing the password or other authentication secrets.

All authentication decisions must be enforced by the server. Client-side checks are for user experience only and must not be treated as security controls.

### 6.2 Rate limiting and failed attempts

Protection against password guessing must be enforced server-side and must consider, as appropriate:

- Account or normalized identifier
- Source network or other abuse signals
- Time window
- Repeated requests and concurrent attempts
- Distributed attempts across multiple sources

The design must balance abuse prevention with the risk of denial-of-service against legitimate users. A permanent or easily triggered account lockout must not be introduced without security review.

The policy must define:

- Thresholds and time windows
- Temporary throttling or lockout behavior
- Progressive delays, if used
- User-facing responses
- Monitoring and alerting
- How legitimate users regain access
- How controls behave when the rate-limiting dependency is unavailable

Exact thresholds and durations must be established through a security and operational decision rather than assumed in application code.

## 7. Sessions, Sign-Out, and Revocation

- Session creation, validation, expiry, sign-out, and revocation must be controlled by the server.
- Sign-out must invalidate the current session server-side; merely deleting client-side state is insufficient.
- The implementation must define session lifetime, renewal, idle timeout, and behavior after password change or password reset.
- The implementation must define whether password changes revoke the current session, all sessions, or require re-authentication.
- The session credential must not be stored in an insecure browser location.
- If browser cookies are selected, their `Secure`, `HttpOnly`, and `SameSite` settings and any applicable CSRF protections must be defined.
- If bearer tokens are selected instead, storage, renewal, rotation, revocation, and replay protections must be explicitly reviewed.
- Session identifiers and credentials must never be written to logs.

The session mechanism is not selected by this document. It requires an architecture decision before implementation.

This temporary policy does not itself implement or guarantee the separate one-active-trusted-device requirement.

## 8. Password Recovery

Password recovery is a separate security-sensitive workflow and must not be treated as ordinary login.

The recovery channel and its verification method are **not decided by this document**. In particular:

- Email must not automatically become a password-recovery method merely because it is collected or verified.
- SMS-based recovery must not be claimed as available until the SMS provider and verification flow have been approved and implemented.
- Recovery must not bypass account verification, authorization, audit, or the one-active-trusted-device policy where that policy applies.
- Recovery responses must not disclose whether an account exists.
- Recovery requests and completion attempts must be rate-limited and monitored.
- Reset credentials must be random, single-use, short-lived, securely stored in non-recoverable form, and invalidated after use or replacement.
- A successful reset must trigger the approved session revocation policy and a security notification where supported.
- Recovery events must be audited without recording passwords, reset secrets, or unnecessary personal data.

**Implementation blocker:** The recovery channel, identity assurance requirements, expiry rules, and session-revocation behavior must be approved before password recovery is released. Until then, a secure recovery flow must not be invented by implementation teams.

## 9. Future Migration to SMS OTP

The architecture must allow SMS OTP to be introduced through an approved authentication policy without rewriting the entire authentication subsystem.

The design should separate:

- Authentication policy selection
- Authentication strategy or mechanism
- Account identity and verification state
- Session issuance and validation
- Notification/provider integrations
- Authorization and protected-resource checks

Future SMS OTP requirements:

1. OTP login must not become active merely because a provider is configured; activation requires explicit server-side policy and security approval.
2. OTP generation, expiry, single-use behavior, attempt limits, request rate limits, delivery failures, and account-enumeration protections must be defined and tested.
3. The SMS provider must be integrated behind an approved, replaceable boundary.
4. Session and authorization behavior must remain consistent with the approved session design.
5. The migration plan must define whether passwords are disabled, retained for recovery, or retained as an alternative. Password retention as a recovery or alternative method requires a separate security decision.
6. The system must not silently enable password and SMS as simultaneous methods unless that combined policy is explicitly approved.
7. Migration must not weaken existing account verification, session revocation, audit, or device policies.

The future design must not assume that SMS OTP is stronger than all password-based authentication in every threat scenario. Its SIM-swap and phone-number takeover risks must be documented.

## 10. Authorization and SOLINT NETWORK Boundary

Authentication establishes an identity; it does not by itself authorize access to a resource.

Every protected server operation must enforce the applicable:

- User identity and account status
- Session validity
- Resource ownership
- Role or permission requirements
- Consent and tenant boundaries, where applicable

SOLINT PUBLIC must not connect directly to SOLINT NETWORK’s internal database. Any future integration must use approved, authenticated, authorized, and documented APIs.

## 11. Audit, Logging, and Privacy

Security events may include:

- Registration and verification state changes
- Successful and failed login outcomes
- Rate-limit or temporary-throttle events
- Password changes and recovery events
- Session creation, sign-out, expiry, and revocation
- Changes to authentication policy

Audit and operational logs must not contain:

- Passwords or password hashes
- OTP or reset-token values
- Session credentials
- Private cryptographic keys
- Unnecessary health information

Collection of IP addresses, user-agent data, or derived identifiers must have a documented security purpose, access policy, retention period, and privacy review. Hashing an identifier does not automatically make it anonymous.

## 12. Data and API Design Requirements

This document defines requirements, not a final database schema.

Before creating migrations, the design must specify:

- Account identifiers and their normalization/uniqueness rules
- Password hash format/version and upgrade strategy
- Email and phone verification states
- Account activation and suspension states
- Rate-limit and lockout representation, if persisted
- Session storage and revocation model
- Password-reset challenge representation and one-time-use enforcement
- Audit event minimization, access, and retention
- Database constraints and transaction boundaries

Before implementing endpoints, an API contract must define:

- Registration and verification operations
- Login and logout operations
- Session behavior
- Password change and recovery operations, once recovery is approved
- Validation and error response shapes
- Rate-limit behavior
- Authentication and authorization requirements
- Versioning and idempotency where appropriate

No field list in this document should be treated as an approved schema. In particular, authentication state must not be implemented as unreviewed counters or security decisions enforced only in the frontend.

## 13. Testing Requirements

The approved implementation plan must include, at minimum:

### Password and account tests

- Password hashing uses Argon2id through the approved library.
- Plaintext passwords are never persisted or logged.
- Password verification succeeds for a valid password and fails for an incorrect password.
- Hash parameters/version can be identified and upgraded according to the approved migration strategy.
- Duplicate or non-canonical account identifiers are handled consistently.

### Login and abuse-control tests

- Valid credentials create a session.
- Invalid credentials do not disclose account existence or verification state.
- Repeated and distributed failed attempts are controlled according to the approved policy.
- Rate-limit dependency failures follow a documented safe behavior.
- Concurrent requests cannot bypass relevant attempt controls.

### Session tests

- Sign-out invalidates the server-side session.
- Expired and revoked sessions are rejected.
- Password reset/change applies the approved session-revocation policy.
- Session credentials are absent from logs and insecure client storage.

### Recovery tests

Once the recovery policy is approved:

- Unknown and known account requests return non-enumerating responses.
- Reset credentials are single-use, expire, and cannot be replayed.
- Rate limits apply to recovery requests and attempts.
- Successful recovery follows the approved session-revocation and notification rules.

### Architecture and privacy tests

- Protected API routes enforce authorization server-side.
- No direct database access to SOLINT NETWORK is introduced.
- Logs, errors, analytics, and traces do not expose passwords, session credentials, or reset secrets.
- Authentication-policy configuration accepts only approved values and cannot silently enable an unapproved method.

## 14. Acceptance Criteria Before Implementation Approval

This policy is ready to guide implementation only when:

- The temporary password-based policy and its precedence over the earlier SMS-login requirement are acknowledged in the governing architecture/security documents.
- The Phase 1 scope and phase numbering are consistent across the roadmap and security documents.
- Backend, database, session, and deployment architecture decisions are approved.
- Password policy and Argon2id configuration strategy are reviewed and documented.
- Rate-limiting and lockout behavior are approved, including denial-of-service considerations.
- The password-recovery channel and security requirements are approved.
- Session expiry, renewal, sign-out, revocation, and password-change behavior are approved.
- Account verification and activation rules are approved, including mobile verification while SMS is unavailable.
- API contracts, data ownership, and migration requirements are reviewed.
- Test cases and acceptance criteria are assigned to implementation tasks.
- The implementation plan is explicitly approved before coding begins.

A policy decision or specification being approved does not mean its implementation is complete or secure.

## 15. Open Decisions and Blockers

The following must be resolved before the affected functionality is implemented:

1. Which email service will deliver verification and security notifications?
2. How will mobile-number ownership be verified before an SMS provider is available?
3. Which recovery channel and identity-assurance process will be used?
4. Which session mechanism will be used, and what are its expiry, renewal, and revocation rules?
5. What password length and user guidance will be required?
6. What Argon2id parameters are appropriate for the production environment?
7. What rate-limit thresholds, temporary-throttle rules, and lockout behavior are approved?
8. Which server-side authentication policy configuration is permitted, and who can change it?
9. How will account verification and recovery interact with the single trusted device policy?
10. Which database, migration tooling, backend framework, and API contract approach are approved?

No implementation team should silently decide these items during coding.

## 16. Implementation Status

- **Temporary password policy:** Product decision recorded.
- **Architecture review:** Pending.
- **Security review:** Pending.
- **Recovery policy:** Pending.
- **Technology/provider selection:** Pending.
- **Implementation:** Not started or not verified by this document.
- **Production approval:** Not granted by this document.

---
````
