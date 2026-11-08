# SOLINT PUBLIC
# Synchronization and Conflict Resolution Specification

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-SYNC-001 |
| Version | 1.0 |
| Status | COMPLETE FOR PHASE 0 BASELINE |
| Date | 2026-11-08 |
| Parent Document | `docs/architecture/SOLINT_PUBLIC_MASTER_ARCHITECTURE.md` |

---

## 1. Purpose

This specification defines the required synchronization model for offline-first records in SOLINT PUBLIC. The server remains the source of truth and must never silently overwrite a user’s data.

## 2. Record lifecycle

### States

```text
LOCAL_DRAFT
  -> PENDING_SYNC
  -> SYNCING
     -> SYNCED
     -> SYNC_FAILED
          -> RETRY
     -> CONFLICT_REQUIRES_REVIEW
```

### Required metadata

Every local record must include:

- `local_record_id`
- `user_id`
- `device_id`
- `created_at`
- `updated_at`
- `record_type`
- `provenance`
- `version`
- `sync_status`
- `idempotency_key`
- `source_system`

## 3. Provenance model

The following provenance values are approved:

- `CLINICALLY_VERIFIED`
- `PATIENT_REPORTED`
- `IMPORTED`
- `AI_GENERATED`
- `OFFLINE_PATIENT_ENTRY`

Offline patient-entered records must remain distinguishable from server-verified clinical records.

## 4. Synchronization rules

1. The app may create local records while offline.
2. All synchronization requests require an idempotency key.
3. The server applies authorization before acceptance.
4. Duplicate submissions must not create duplicate medical records.
5. Conflicts require explicit resolution, never silent overwrite.
6. Revoked devices cannot synchronize.
7. The local queue must persist safely across restarts and connectivity interruptions.

## 5. Conflict resolution model

### Conflict categories

| Type | Behavior |
|---|---|
| Same record same version | Accept idempotently |
| Same record different version, same owner | Review required or server merge policy |
| Same semantic record by different device | Conflict review |
| Device revocation before sync | Reject and mark as security event |
| Server authoritative record changed | Prefer server version with audit trail |

### Required server behavior

- compare version and timestamp
- require ownership validation
- reject silent overwrite
- create conflict review task when needed
- record audit entry for all conflict decisions

## 6. Retry and failure handling

The system must support:

- exponential backoff or scheduled retry
- manual synchronization trigger
- retry after internet recovery
- queued records with failure state
- preserving data if the app closes before sync completion

## 7. API semantics

The synchronization API should support:

- `POST /api/public/v1/sync/records`
- `GET /api/public/v1/sync/status`
- `POST /api/public/v1/sync/retry`
- `POST /api/public/v1/sync/conflicts/review`

Each request must include:

- request id
- user id
- device id
- idempotency key
- hash or checksum for payload integrity
- authorization token

## 8. Final status

This specification is a Phase 0 baseline and must be validated against actual workflow requirements, provider APIs, and the final storage design before implementation starts. The essential rule remains unchanged: the server is the source of truth, and conflicts are never silently lost.

---

End of document.
