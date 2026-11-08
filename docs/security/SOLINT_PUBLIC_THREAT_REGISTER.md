# SOLINT PUBLIC
# Threat Register

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-THREAT-001 |
| Version | 1.0 |
| Status | COMPLETE FOR PHASE 0 BASELINE |
| Date | 2026-11-08 |
| Parent Document | `docs/architecture/SOLINT_PUBLIC_PHASE_0_TECHNICAL_VALIDATION_AND_SECURITY_THREAT_MODEL.md` |

---

## 1. Threat register

| ID | Threat | Category | Risk | Mitigation | Residual risk |
|---|---|---|---|---|---|
| T-001 | Unauthorized device registration | Spoofing | Critical | Server-side unique-device enforcement | Low if backend invariant is enforced |
| T-002 | Account takeover via OTP abuse | Spoofing | Critical | Rate limiting, OTP expiry, attempt caps | Medium |
| T-003 | SIM swap / number takeover | Spoofing | Critical | Additional verification and suspicious activity review | Medium |
| T-004 | Local data extraction from browser | Information disclosure | High | Encrypt before storage, secure data minimization | Medium |
| T-005 | Stolen unlocked mobile device | Elevation of privilege | Critical | Local PIN/biometric lock + server-side revocation | Medium |
| T-006 | Offline authorization bypass | Tampering | Critical | Local expiry + server-aware validation | Medium |
| T-007 | Device replacement abuse | Elevation of privilege | Critical | Atomic state transitions and explicit review | Low |
| T-008 | Duplicate record sync | Tampering | High | Idempotency keys and dedupe validations | Low |
| T-009 | Conflict overwrites | Tampering | Critical | Server merge rules + review queue | Medium |
| T-010 | IDOR / wrong-user data access | Elevation of privilege | Critical | Server-side ownership checks and ABAC/RBAC | Low if enforced |
| T-011 | XSS payload in PWA | Information disclosure | High | CSP, sanitization, secure API model | Medium |
| T-012 | Malicious dependency | Tampering | High | Dependency scanning and lockfiles | Medium |
| T-013 | API abuse / rate attack | Denial of service | High | Gateway rate limit and monitoring | Medium |
| T-014 | Credential leakage in logs | Information disclosure | Critical | Log redaction and secret scanning | Low |
| T-015 | Browser storage clearing | Availability | Medium | Graceful recovery flow and resync | Medium |
| T-016 | Revoked device continues syncing | Authorization bypass | Critical | Device status validation on all write calls | Low |

## 2. Mitigation ownership

Each critical or high-risk item requires an owner and acceptance before implementation proceeds.

## 3. Risk acceptance

The following risks remain accepted as design-level residual risks only if they are documented and reviewed:

- browser storage may be lost
- mobile OS may not provide hardware-backed key storage
- SMS is not equivalent to a fully phishing-resistant factor
- offline access on a lost/stolen device remains limited but not zero

## 4. Final gate

No critical threat may remain unmitigated before Phase 1 implementation begins.

---

End of document.
