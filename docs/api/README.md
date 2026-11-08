# SOLINT PUBLIC
# API Contracts Overview

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-API-001 |
| Version | 1.0 |
| Status | COMPLETE FOR PHASE 0 BASELINE |
| Date | 2026-11-08 |
| Project | SOLINT PUBLIC |

---

## 1. Design principles

- all PUBLIC and NETWORK communication is API-based
- no direct database access
- all endpoints require authentication and authorization
- all writes are audited
- versioning is mandatory
- responses use minimal data and explicit error codes

## 2. Base paths

```text
/api/public/v1
```

## 3. Core endpoints

### Authentication

- `POST /api/public/v1/auth/register`
- `POST /api/public/v1/auth/verify-email`
- `POST /api/public/v1/auth/request-otp`
- `POST /api/public/v1/auth/verify-otp`
- `POST /api/public/v1/auth/login`
- `POST /api/public/v1/auth/logout`

### Devices

- `POST /api/public/v1/devices/register`
- `GET /api/public/v1/devices/me`
- `POST /api/public/v1/devices/replace`
- `POST /api/public/v1/devices/revoke`

### Synchronization

- `POST /api/public/v1/sync/records`
- `GET /api/public/v1/sync/status`
- `POST /api/public/v1/sync/retry`
- `POST /api/public/v1/sync/conflicts/review`

### Patient data

- `GET /api/public/v1/patient/profile`
- `POST /api/public/v1/patient/records`
- `GET /api/public/v1/patient/records`

## 4. Response envelope

```json
{
  "success": true,
  "data": {},
  "error": null,
  "traceId": "uuid"
}
```

## 5. Standard error codes

- `400` invalid request
- `401` unauthenticated
- `403` forbidden
- `409` conflict or duplicate
- `410` revoked or expired state
- `422` validation failure
- `429` rate limited
- `500` server error

## 6. Security requirements

- all requests over HTTPS only
- rate limiting on OTP and login requests
- idempotency keys for writes
- server-side device and user ownership checks on every write
- no sensitive data in logs

## 7. Contract status

This is the initial contract baseline; the final contract must be validated against real NETWORK integration APIs and the production data model before implementation approval.

---

End of document.
