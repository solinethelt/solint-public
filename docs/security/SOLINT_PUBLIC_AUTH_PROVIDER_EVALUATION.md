# SOLINT PUBLIC
# Authentication Provider Evaluation

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-AUTH-001 |
| Version | 1.0 |
| Status | COMPLETE FOR PHASE 0 BASELINE |
| Date | 2026-11-08 |
| Parent Document | `docs/architecture/SOLINT_PUBLIC_PHASE_0_TECHNICAL_VALIDATION_AND_SECURITY_THREAT_MODEL.md` |

---

## 1. Objective

This document evaluates the authentication model required for SOLINT PUBLIC and defines the minimum capabilities needed to satisfy the confirmed product policy.

## 2. Confirmed policy

The product owner has confirmed the following requirements:

- Email is used for registration and initial identity verification only.
- Subsequent online login uses a mobile number and SMS OTP.
- Password login is not required in the initial model.
- Exactly one active trusted device is allowed per account.
- Device replacement requires explicit authorization.
- Server remains the source of truth.

## 3. Options evaluated

### Option A — Custom OTP implementation

**Description**
- Custom OTP generator, verification logic, challenge storage, and rate limiting implemented inside the SOLINT PUBLIC backend.

**Pros**
- Full control over flows and policy.
- Easy to enforce one-device invariant and custom session logic.
- Easier to keep auth rules aligned with product policy.

**Cons**
- High engineering and security burden.
- Requires careful handling of SMS provider integration, brute-force protections, and abuse detection.
- Must still be reviewed for SIM-swap and account takeover risks.

**Verdict**
- Viable, but only if a strong engineering team is assigned to secure implementation and audits.

### Option B — Managed authentication provider

**Description**
- Using a recognized external identity provider for session management and OTP delivery, with custom business logic layered on top.

**Pros**
- Faster to implement securely.
- Stronger baseline for rate limiting, token management, and session controls.
- Easier to keep security controls updated.

**Cons**
- Some providers may not support the exact business invariant without custom server-side logic.
- Requires careful integration with custom trusted-device checks.
- Can create confusion if the provider is treated as the sole source of authorization.

**Verdict**
- Recommended baseline for production implementation.

### Option C — Supabase Auth or equivalent provider

**Description**
- Managed auth with OTP, session tokens, and backend integration.

**Pros**
- Fast to implement.
- Good developer experience.
- Works well for modern PWA and serverless or Node-based applications.

**Cons**
- The product requires a stricter device trust boundary than a generic auth provider exposes out of the box.
- One-device enforcement must still be implemented as a server-side invariant.
- It must not be treated as a replacement for the trusted-device module.

**Verdict**
- Acceptable if the provider meets the project’s security, session, and device requirements.

## 4. Recommended baseline

The recommended model is:

- Use a managed identity/authentication provider for session management and OTP issuance.
- Keep device trust, offline authorization, and device replacement as custom server-side business logic.
- Treat the provider as an infrastructure primitive, not as the sole determinant of account trust.

### Key design rule

Authentication and authorization are separate concerns.

The provider manages login sessions and token lifetimes, but the SOLINT PUBLIC backend must still enforce:

- exactly one active trusted device
- server-side device authorization
- OTP rate limits
- offline authorization window
- account recovery and replacement controls

## 5. Minimum required capabilities

The selected provider must support:

- mobile number lookup
- one-time password generation and validation
- secure token issuance and expiry
- session invalidation
- rate limiting and attempt throttling
- audit logging for auth events
- integration with custom backend authorization checks

## 6. Security requirements to enforce outside the provider

The following must be enforced by SOLINT PUBLIC itself, not delegated to the provider alone:

- one trusted device per account
- atomic device replacement
- revoked-device rejection
- device state transitions
- offline authorization metadata
- secure audit trail for login and recovery events
- phone number change verification
- suspicious login and device replacement review

## 7. Decision

The approved Phase 0 decision is:

- Recommended: managed auth provider for identity/session operations.
- Required: custom SOLINT PUBLIC trusted-device module and device authorization layer.
- Prohibited: treating an auth provider as a full replacement for product-level device trust decisions.

## 8. Risk note

Even with a managed provider, SMS OTP remains exposed to SIM-swap and phone-number takeover risks. This is acceptable only if the architecture adds server-side rate limits, suspicious activity monitoring, and a robust device replacement process.

---

End of document.
