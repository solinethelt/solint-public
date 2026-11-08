# SOLINT PUBLIC
# Key Management Design

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-KEY-001 |
| Version | 1.0 |
| Status | COMPLETE FOR PHASE 0 BASELINE |
| Date | 2026-11-08 |
| Parent Document | `docs/security/SOLINT_PUBLIC_SECURE_IDENTITY_AND_OFFLINE_ARCHITECTURE.md` |

---

## 1. Purpose

This design defines the minimum approved pattern for local key handling in the PWA and for the server-side authorization model.

## 2. Design principles

- Local health records must never be stored in plaintext.
- Keys are separated by purpose.
- A master key must never be stored next to encrypted data.
- The server is authoritative for account and device trust.
- Browser storage is not a trusted secure enclave.

## 3. Key categories

| Key | Purpose | Storage | Notes |
|---|---|---|---|
| Device identity key | Device binding and attestation | Browser keystore or protected local storage | Not sufficient alone for trust |
| Local data encryption key | Encrypt health records in IndexedDB | Device-local secure key material | Must be protected from browser extraction |
| Session credential | Secure online session | Backend-issued token store | Never in localStorage |
| Offline authorization key | Enforce 72-hour local access window | Local authenticated metadata + server record | Must be revocable |

## 4. Local encryption design

### Recommended algorithm

- AES-GCM for authenticated encryption
- HKDF or equivalent key derivation for local key material
- unique nonce per record or batch
- encrypted payload includes integrity tag and metadata

### Storage pattern

- Data is encrypted before IndexedDB write.
- Ciphertext is stored alongside metadata fields.
- Metadata should not contain plaintext sensitive records.

## 5. Key lifecycle

### Key generation

- Generate per-device key material after successful registration.
- Generate a derived local data key at first secure offline bootstrap.
- Bind the key to the verified active device via server-side registration.

### Key rotation

- Rotate local data keys when the device is re-registered.
- Rotate on explicit security event, suspicious activity, or device replacement.
- Ensure stale encrypted records remain inaccessible until re-keying is complete.

### Key invalidation

- Invalidate on revocation.
- Invalidate on account recovery events.
- Invalidate on evidence of compromise.
- Require onboarding flow to regenerate device trust after invalidation.

## 6. Key protection limitations

The approved Phase 0 position is:

- Web Crypto API is required for browser-side cryptographic operations.
- Web Crypto API does not automatically provide hardware-backed protection.
- Local browser storage can be cleared or accessed by malicious JavaScript.
- The system must not state that data is protected by a hardware root of trust unless tested and verified.

## 7. Server-side companion controls

The backend must not allow a local key to override server authorization. The server must still validate:

- active device state
- record ownership
- sync authorization
- device replacement status
- offline expiry

## 8. Final design decision

The approved baseline is:

- local data encryption before persistence
- server-side authorization remains the real trust boundary
- device-bound local key material with explicit rotation and invalidation
- no plaintext medical data in browser storage
- no secret or master key stored alongside encrypted records

---

End of document.
