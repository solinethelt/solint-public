# SOLINT PUBLIC
# Device Trust Threat Model

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-DEVICE-001 |
| Version | 1.0 |
| Status | COMPLETE FOR PHASE 0 BASELINE |
| Date | 2026-11-08 |
| Parent Document | `docs/security/SOLINT_PUBLIC_SECURE_IDENTITY_AND_OFFLINE_ARCHITECTURE.md` |

---

## 1. Scope

This threat model focuses on trusted-device enforcement, online/offline authentication boundaries, and device replacement risk in SOLINT PUBLIC.

## 2. Security invariants

The following invariants are mandatory:

1. `COUNT(active_trusted_devices) <= 1` per user.
2. The backend enforces the invariant, not just the app UI.
3. A revoked device cannot synchronize or obtain new authorization.
4. A lost or stolen device cannot bypass the 72-hour offline maximum.
5. Device replacement must be atomic.
6. Audit logs must record device lifecycle events.

## 3. Assets at risk

- user account identity
- offline authorization metadata
- local encryption keys
- health records created offline
- device trust state and inverse trust relationships
- synchronization queue

## 4. Threat scenarios and control strategy

| Threat | Likelihood | Impact | Control |
|---|---|---|---|
| Unauthorized device registration | Medium | Critical | Server-side unique active-device constraint |
| Simultaneous replacement requests | Medium | Critical | Transactional state machine with locking |
| Lost/stolen device access | Medium | Critical | Revocation, offline expiry, secure local lock |
| Device clock rollback | Medium | High | Server-aware expiry validation and conservative policy |
| Browser storage extraction | Medium | High | Encrypt sensitive data before local persistence |
| Offline authorization abuse | Medium | High | Local expiry enforcement + server-side authorization checks |
| Session theft / token reuse | Medium | Critical | Secure tokens + session invalidation + device binding |
| Duplicate synchronization | Medium | High | Idempotency keys and dedupe checks |
| Device spoofing | Low-Medium | Critical | Public-key verification and server-side trust relation |
| Phone number takeover | Medium | Critical | Additional verification for number change and recovery |

## 5. Device trust model

### 5.1 Trusted device lifecycle

```text
UNREGISTERED -> PENDING_VERIFICATION -> ACTIVE -> REPLACEMENT_PENDING -> REVOKED
```

### 5.2 Required server-side fields

- `user_id`
- `device_id`
- `device_public_key`
- `device_name`
- `device_platform`
- `device_status`
- `registered_at`
- `last_authenticated_at`
- `last_seen_at`
- `revoked_at`
- `offline_access_expiry`
- `security_metadata`

### 5.3 Storage rule

A unique constraint must ensure that only one row per user has `device_status = ACTIVE`.

If the implementation uses a database with transactional semantics, the invariant must be enforced in one transaction.

## 6. Device replacement flow

1. User requests replacement and re-authenticates through current trusted device or verified OTP.
2. Backend validates the request and checks suspicious activity.
3. Backend marks current active device as `REPLACEMENT_PENDING`.
4. New device completes registration and public-key binding.
5. Backend atomically swaps active device.
6. Old device receives revocation status and cannot synchronize further.
7. Offline authorization on old device is allowed only until the remaining allowed window expires.

## 7. Residual risk and mitigations

### Residual risk: offline device still active after revocation

- Mitigation: 72-hour maximum offline period, server-aware expiry, and local lock after expiry.
- Residual risk: limited but accepted if clearly documented.

### Residual risk: browser key storage not hardware-backed

- Mitigation: keep local keys encrypted, rely on server-side enforcement, and avoid over-claiming security guarantees.
- Residual risk: must be documented in product and security review.

### Residual risk: user device compromise

- Mitigation: local PIN/biometric lock, secure logout, short session expiry, and device revocation flows.
- Residual risk: unavoidable in browser-based environments without OS-level trust anchors.

## 8. Required audit events

- registration
- device registration
- online login success/failure
- device replacement requested
- device replacement completed
- device revocation
- offline authorization granted
- offline authorization expired
- suspicious login flagged
- phone number replacement

## 9. Phase 0 conclusion

The device trust model is feasible only if it is enforced at the backend, persisted in a transaction-safe model, and constrained by the 72-hour offline limit. A purely frontend-only approach is not acceptable.

---

End of document.
