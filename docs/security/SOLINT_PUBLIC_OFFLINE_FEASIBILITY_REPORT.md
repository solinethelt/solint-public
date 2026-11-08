# SOLINT PUBLIC
# Offline Feasibility Report

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-OFFLINE-001 |
| Version | 1.0 |
| Status | COMPLETE FOR PHASE 0 BASELINE |
| Date | 2026-11-08 |
| Parent Document | `docs/security/SOLINT_PUBLIC_SECURE_IDENTITY_AND_OFFLINE_ARCHITECTURE.md` |

---

## 1. Executive summary

SOLINT PUBLIC can support an offline-first PWA with strict limits, but only under a clearly constrained model. The feasible architecture is:

- offline access only after successful online authentication
- offline authorization expires after 72 hours
- local data is encrypted before persistence
- only a narrow set of approved data is available offline
- synchronization is required once connectivity returns
- the server remains the source of truth

## 2. Platform evaluation

### iOS Safari

**Key findings**
- Supports PWA installation and local storage, but behavior varies by version and browser lifecycle.
- Background sync and lifecycle behavior are limited.
- IndexedDB is available but should not be assumed to behave identically to desktop browsers.

**Conclusion**
- Offline mode is feasible, but must not depend on background sync as a mandatory mechanism.

### Android Chrome

**Key findings**
- Better PWA support than iOS Safari.
- IndexedDB and service worker support are generally reliable.
- App lifecycle and storage clearing remain user- and device-sensitive.

**Conclusion**
- Feasible, but still requires server-aware expiration and local security lock rules.

### Desktop browsers

**Key findings**
- Useful for testing only.
- Desktop behavior cannot be treated as the production reference for PWA security or offline lifecycle.

**Conclusion**
- Desktop support is not sufficient validation for mobile behavior.

## 3. Feasibility assessment

| Requirement | Feasibility | Notes |
|---|---|---|
| Open app offline on trusted device | Feasible | Requires prior trusted-device registration |
| Local PIN/biometric unlock | Feasible | Depends on platform support |
| Offline record creation | Feasible | Only for approved data types |
| 72-hour offline access | Feasible with caution | Must be enforced with server-aware awareness |
| Secure local encrypted data | Feasible | Requires Web Crypto API and careful key design |
| Background sync | Partially feasible | Not reliable enough to be mandatory |
| Revocation while device is offline | Limited | Acceptance of residual risk is required |

## 4. Critical constraints

1. The browser is not a trusted security boundary.
2. Application clock values cannot be treated as trusted time.
3. Service Worker and browser storage must never be considered equivalent to a secure enclave.
4. A user can clear browser data, which may wipe local state.
5. Protected data must remain inaccessible when local authorization expires.

## 5. Approved offline model

The approved offline model for Phase 0 is:

- offline is allowed only for a previously trusted device
- offline access only for explicitly approved data categories
- offline authorization window is 72 hours maximum
- offline actions are queued for server sync when connectivity returns
- server checks authorization before accepting records from local queue
- local records remain marked as not clinically verified until accepted by the server

## 6. Required controls

- Encrypt health data before storing in IndexedDB.
- Do not use localStorage for auth secrets or sensitive data.
- Enforce local lock after inactivity.
- Prevent silent bypass of offline expiry.
- Force re-authentication after expiry or when device trust changes.
- Keep a separate queue of local records with provenance metadata.

## 7. Final conclusion

The offline-first PWA approach is feasible for SOLINT PUBLIC if and only if it remains bounded by a strict security model: a single trusted device, one valid offline window, limited data categories, and strong server-side enforcement. Without those controls, offline support would create unacceptable medical data risk.

---

End of document.
