# SOLINT PUBLIC
# Database Architecture

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-DB-001 |
| Version | 1.0 |
| Status | COMPLETE FOR PHASE 0 BASELINE |
| Date | 2026-11-08 |
| Parent Document | `docs/architecture/SOLINT_PUBLIC_MASTER_ARCHITECTURE.md` |

---

## 1. Database objective

The database for SOLINT PUBLIC must support patient-centered health records, auth and device trust, offline-enabled workflows, and strict server-side authorization boundaries without exposing direct access to SOLINT NETWORK data.

## 2. Database model

### Core ownership model

- `public` schema for PUBLIC-owned data
- no direct access to NETWORK schema
- no shared tables between PUBLIC and NETWORK
- all cross-product access via API layer only

## 3. Core tables

### users

- `id`
- `email`
- `mobile_number`
- `status`
- `created_at`
- `updated_at`
- `deleted_at`

### user_identities

- `id`
- `user_id`
- `identity_type` (`email`, `mobile`, `passkey`, etc.)
- `value`
- `is_verified`
- `verified_at`

### trusted_devices

- `id`
- `user_id`
- `device_id`
- `device_public_key`
- `device_name`
- `platform`
- `status`
- `registered_at`
- `last_authenticated_at`
- `last_seen_at`
- `revoked_at`
- `offline_access_expiry`

### sessions

- `id`
- `user_id`
- `device_id`
- `session_token_hash`
- `issued_at`
- `expires_at`
- `revoked_at`
- `ip_hash`
- `user_agent_hash`

### otp_challenges

- `id`
- `user_id`
- `mobile_number`
- `code_hash`
- `issued_at`
- `expires_at`
- `attempt_count`
- `used_at`
- `status`

### offline_authorization

- `id`
- `user_id`
- `device_id`
- `granted_at`
- `expires_at`
- `status`
- `source`

### patient_records

- `id`
- `user_id`
- `record_type`
- `provenance`
- `version`
- `content_json`
- `server_confirmed`
- `created_at`
- `updated_at`
- `deleted_at`

### local_sync_queue

- `id`
- `user_id`
- `device_id`
- `local_record_id`
- `payload_json`
- `status`
- `attempt_count`
- `last_attempt_at`
- `idempotency_key`

### audit_events

- `id`
- `user_id`
- `device_id`
- `event_type`
- `event_payload`
- `created_at`
- `actor_type`

## 4. Required constraints

- unique active device per user
- OTP challenge single-use and expiry-bound
- revoked sessions disabled on every request
- no plaintext secret fields in logs or audit payloads
- device and record ownership checks in DB or application transaction layer

## 5. Database governance

- schema migrations in code repository
- review before any production migration
- no direct data copy from NETWORK
- all significant writes audited

## 6. Phase 0 conclusion

This database baseline is suitable for the initial phase and provides the core structures needed for secure auth, device trust, and offline synchronization. It must be revalidated when real API contracts and NETWORK integrations are finalized.

---

End of document.
