# SOLINT PUBLIC

# Secure Identity, Single Trusted Device & Offline-First Architecture

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-SEC-001 |
| Version | 1.1 |
| Status | DRAFT — Architecture Baseline Updated |
| Date | November 2026 |
| Parent Document | `docs/architecture/SOLINT_PUBLIC_MASTER_ARCHITECTURE.md` |
| Scope | SOLINT PUBLIC |
| Platform | Progressive Web Application (PWA) |

---

## 1. PURPOSE

This document defines the security architecture for:

- User registration and identity verification
- Mobile-based OTP authentication
- Single trusted device management
- Secure offline application access
- Local health data protection
- Offline record creation
- Secure synchronization with the central server
- Device replacement and account recovery

The primary objective is to ensure that each SOLINT PUBLIC account is accessible through only one active trusted device while supporting controlled offline operation.

Security and privacy are foundational requirements of SOLINT PUBLIC.

## 2. CONFIRMED PRODUCT REQUIREMENTS

The following decisions have been confirmed by the product owner.

| Requirement | Confirmed Policy |
|---|---|
| Email usage | Registration and initial identity verification only |
| Subsequent login | Mobile number + SMS OTP |
| Password-based login | Not required in the initial authentication model |
| Active devices | Exactly one active trusted device per account |
| Additional simultaneous devices | Not permitted |
| Offline access | Maximum 72 hours |
| Offline authentication | Local PIN or supported device biometrics |
| Offline record creation | Supported |
| Server synchronization | After internet connectivity is restored |
| Server as source of truth | Mandatory |

These requirements must be reflected in the database, authentication services, API contracts, frontend, and testing strategy.

## 3. USER REGISTRATION AND IDENTITY VERIFICATION

### 3.1 Registration

Registration requires:

- Email address
- Mobile phone number
- Required identity information
- Acceptance of terms and privacy policy

The registration process must include:

1. Email format validation.
2. Email ownership verification.
3. Mobile number validation.
4. SMS OTP verification of the mobile number.
5. Account creation.
6. Initial trusted-device registration.

Email and mobile verification are separate steps.

### 3.2 Email Usage Policy

After successful registration:

- Email must not be required for normal login.
- Email must not be requested for every login.
- Email must not replace mobile OTP authentication.
- Email must remain associated with the verified account.
- Changes to the registered email must require a separate verification process.

Email-based account recovery must not be introduced automatically without a separately approved security policy.

## 4. MOBILE OTP AUTHENTICATION

### 4.1 Login Flow

All subsequent online logins must use the registered mobile number and SMS OTP.

Proposed flow:

1. User opens SOLINT PUBLIC.
2. User enters the registered mobile number.
3. Server identifies the account.
4. System checks account status and device registration.
5. System generates a one-time password.
6. OTP is sent to the registered mobile number.
7. User enters the OTP.
8. Server validates OTP.
9. Server checks the device authorization policy.
10. Access is granted if all security conditions are satisfied.

Email must not be required in this flow.

### 4.2 OTP Security Requirements

OTP must:

- Be single-use.
- Have a short configurable expiration period.
- Be securely generated.
- Be securely validated on the server.
- Have a maximum attempt limit.
- Have request-rate restrictions.
- Be invalidated after successful verification.
- Be protected against brute-force attacks.
- Be protected against repeated message requests.

OTP values must never be stored in application logs.

### 4.3 SIM Swap and Phone Number Security

SMS OTP has security limitations.

The system must include:

- OTP request rate limiting.
- Suspicious login detection.
- Repeated failed-attempt monitoring.
- Device replacement restrictions.
- Security event logging.
- Secure phone number replacement procedures.
- Additional verification for suspicious account recovery.

Future authentication upgrades may include Passkeys or other stronger authentication mechanisms.

They must not be added in a way that silently changes the confirmed login experience.

## 5. SINGLE TRUSTED DEVICE POLICY

### 5.1 Core Rule

Each SOLINT PUBLIC account may have only one active trusted device.

Simultaneous active access from multiple trusted devices is prohibited.

The system must distinguish between:

- Current trusted device
- New unregistered device
- Previously trusted but revoked device
- Device undergoing replacement

### 5.2 Device Registration

After successful registration and initial authentication, the device must be registered.

Suggested data model:

**`trusted_devices`**

- `id`
- `user_id`
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

A unique server-side constraint or equivalent transactional enforcement must ensure that each user has no more than one active trusted device.

This restriction must be enforced by the backend, not merely by the frontend.

### 5.3 Device Identity

Device trust must not depend solely on:

- IP address
- Browser fingerprint
- User-agent
- Device name
- Cookies

The architecture should use:

- Web Crypto API
- Device-specific asymmetric cryptographic keys
- Server-side public-key registration
- WebAuthn/Passkeys where supported
- Secure session management

Private keys must never be transmitted to the server.

The system must document the limitations of browser-based key storage and must not claim hardware-backed protection unless technically verified.

### 5.4 Device Replacement

If a user wants to move their account to a new device:

1. User starts the device replacement process.
2. User verifies access through the registered mobile number.
3. Server checks account and device status.
4. The previous device is marked for revocation.
5. The new device completes registration.
6. The previous device loses authorization for future online access and synchronization.
7. The new device becomes the only active trusted device.

The replacement operation must be atomic.

At no point may two devices remain simultaneously active as trusted devices.

The replacement process must also address the possibility that the previous device is lost, stolen, or inaccessible.

For suspicious replacement requests, additional verification or a temporary security hold must be possible.

## 6. OFFLINE ACCESS POLICY

### 6.1 Confirmed Offline Duration

Maximum offline authorization period:

**72 hours**

This is the maximum time after the last successful online authentication during which the trusted device may access permitted offline information.

The 72-hour period must not be extended silently.

### 6.2 Initial Offline Authorization

Offline access becomes available only after:

1. Successful online authentication.
2. Successful OTP verification.
3. Trusted-device registration.
4. Successful establishment of local security credentials.
5. Completion of required offline data preparation.

A new device cannot obtain offline access without completing online authentication.

### 6.3 Offline Unlocking

After the initial online authentication, the user may reopen the application without internet access.

Local unlocking may use:

- A locally configured PIN
- Supported device biometrics

This is local device unlocking, not a new online login.

OTP cannot be delivered or verified while the device has no internet connection.

### 6.4 Offline Expiration

After 72 hours:

- Sensitive offline data must be locked.
- The user must reconnect to the internet.
- Mobile OTP authentication must be completed.
- Offline authorization may then be renewed.

The application must not silently bypass the expiration rule.

The expiration policy must be enforced through protected local authorization state and server-side authorization.

## 7. SECURE OFFLINE DATA STORAGE

### 7.1 Storage Architecture

Recommended components:

- IndexedDB
- Web Crypto API
- AES-GCM authenticated encryption
- Device-specific cryptographic keys
- Service Worker
- Local synchronization queue

IndexedDB alone is not sufficient to protect medical information.

Sensitive data must be encrypted before persistent local storage.

### 7.2 Local Data Protection

Requirements:

- No plaintext persistence of protected health data.
- No authentication tokens in localStorage.
- Secure key handling.
- Automatic locking after inactivity.
- Local authorization expiration.
- Secure logout behavior.
- Restricted caching of sensitive information.
- Data minimization.
- Protection against unauthorized local access.

If the application detects that the local security state has been invalidated, sensitive data must remain inaccessible.

### 7.3 Offline Data Categories

**Category A — Essential Information**

Potentially available offline:

- Basic patient profile
- Active medication list
- Critical allergies
- Active care plans
- Selected emergency information
- Essential appointment details

**Category B — Optional Information**

Available only when explicitly enabled:

- Selected laboratory results
- Selected medical documents
- Health record summaries
- Selected historical information

**Category C — Online-Only Information**

Not available offline by default:

- Financial information
- Sensitive administrative records
- Data requiring current server authorization
- Unnecessary historical information

The application must allow explicit control over which permitted information is available offline.

## 8. OFFLINE RECORD CREATION

Users must be able to create permitted health-related information without internet access.

Examples:

- Symptom records
- Health observations
- Personal health updates
- Medication-related notes
- Measurements
- Care-related notes
- Selected document attachments

Every locally created record must include:

- Local record ID
- User ID
- Device ID
- Creation timestamp
- Modification timestamp
- Record type
- Synchronization status
- Data provenance
- Version number

### 8.1 Record Status

Supported states:

- `LOCAL_DRAFT`
- `PENDING_SYNC`
- `SYNCING`
- `SYNCED`
- `SYNC_FAILED`
- `CONFLICT_REQUIRES_REVIEW`

The application must clearly distinguish between local records and records confirmed by the central server.

### 8.2 Data Provenance

Supported provenance values:

- `CLINICALLY_VERIFIED`
- `PATIENT_REPORTED`
- `IMPORTED`
- `AI_GENERATED`
- `OFFLINE_PATIENT_ENTRY`

Offline patient-entered information must never automatically become clinically verified information.

AI-generated content must remain distinguishable from physician-approved content.

## 9. SECURE SYNCHRONIZATION

### 9.1 Synchronization Workflow

When internet connectivity returns:

1. Detect connectivity.
2. Validate the current session.
3. Validate the active device.
4. Check device revocation status.
5. Check authorization expiry.
6. Retrieve pending local records.
7. Validate each record.
8. Submit records through authenticated APIs.
9. Apply server-side authorization.
10. Check for duplicate submissions.
11. Detect conflicts.
12. Save accepted records.
13. Confirm successful synchronization.
14. Update local synchronization status.

### 9.2 Idempotency

Every synchronization request must include an idempotency key.

Repeated submissions must not create duplicate health records.

### 9.3 Conflict Resolution

Conflicting health information must never be silently overwritten.

The system must support:

- Version comparison
- Timestamp comparison
- Ownership verification
- Conflict detection
- User notification
- Manual review when necessary
- Audit logging

### 9.4 Background Synchronization

Background Sync may be used where supported.

However, it must not be a mandatory dependency.

Synchronization must also be triggered:

- When the application opens.
- When it returns to the foreground.
- When connectivity is restored.
- When the user requests manual synchronization.

This is especially important for iOS and Safari environments.

## 10. SERVICE WORKER AND PWA SECURITY

The Service Worker must:

- Cache necessary application assets only.
- Avoid caching sensitive API responses by default.
- Never cache authentication secrets.
- Apply explicit cache-control policies.
- Support secure application updates.
- Remove expired and unnecessary cached data.

Additional requirements:

- Content Security Policy
- XSS protection
- Secure HTTPS
- Secure session handling
- CSRF protection where applicable
- Dependency integrity
- Strict API authorization
- No authentication tokens in localStorage
- Secure logout and local-data locking

## 11. ACCOUNT RECOVERY AND SECURITY EVENTS

The system must support:

- Lost-device reporting
- Device replacement
- Device revocation
- Phone number replacement
- Session termination
- Suspicious login detection
- Account recovery
- Security notifications
- Audit history

Changing a registered mobile number must require additional identity verification.

Email must not automatically become an alternative login method.

Recovery must not bypass the single-device policy.

## 12. SERVER-SIDE ARCHITECTURE

The server remains the authoritative source of truth.

Proposed architecture:

```text
SOLINT PUBLIC PWA
        │
        ▼
Mobile OTP Authentication
        │
        ▼
Device Trust Service
        │
        ▼
API Gateway
        │
        ▼
Authorization Layer
        │
        ▼
Application Services
        │
        ▼
PostgreSQL Database
```

All sensitive operations must validate:

- User identity
- Session validity
- Active device
- Device authorization
- Data ownership
- Tenant access
- Record permissions
- Consent requirements

SOLINT PUBLIC must never access the internal SOLINT NETWORK database directly.

## 13. AUDIT AND SECURITY MONITORING

The following events must be recorded:

- Registration
- Email verification
- Mobile verification
- OTP generation
- OTP validation
- Successful login
- Failed login
- Device registration
- Device replacement
- Device revocation
- Offline authorization
- Offline authorization expiration
- Synchronization
- Synchronization failure
- Account recovery
- Phone number replacement
- Security-related data access

Audit logs must not contain:

- OTP values
- Passwords
- Private cryptographic keys
- Unnecessary sensitive medical content

## 14. DEVELOPMENT PHASES

### Phase 0 — Security Architecture

Tasks:

- Finalize authentication policy.
- Define single-device enforcement.
- Define device replacement.
- Define OTP security.
- Define offline authorization.
- Define encryption and key management.
- Define offline-data classification.
- Define synchronization contracts.
- Prepare threat model.

Deliverables:

- Authentication Specification
- Device Trust Specification
- Offline Data Policy
- Threat Model
- API Contracts
- Security Test Plan

### Phase 1 — Registration and OTP

Tasks:

- Email registration
- Email verification
- Mobile verification
- SMS OTP login
- OTP rate limiting
- Session management
- Secure account creation

Acceptance:

- Email is not required for subsequent login.
- Mobile OTP is mandatory for online login.
- OTP is single-use and expires.
- OTP abuse is restricted.
- Unverified users cannot access protected information.

### Phase 2 — Single Trusted Device

Tasks:

- Device registration
- Device public-key registration
- Single-device enforcement
- Device management
- Device replacement
- Device revocation
- Session invalidation

Acceptance:

- Each account has only one active trusted device.
- A new device cannot become active without authorized replacement.
- The previous device loses future online authorization.
- Device state is enforced server-side.

### Phase 3 — Secure Offline Foundation

Tasks:

- PWA installation
- Service Worker
- IndexedDB
- Local encryption
- Secure local unlocking
- 72-hour offline authorization
- Automatic application lock

Acceptance:

- Trusted device can reopen the application offline.
- Unknown devices cannot access local account data.
- Expired offline authorization blocks sensitive access.
- Protected data is not persisted in plaintext.

### Phase 4 — Offline Records

Tasks:

- Offline record creation
- Local record IDs
- Record provenance
- Synchronization queue
- Status indicators

Acceptance:

- Permitted information can be recorded offline.
- Records retain their original timestamps.
- Local records remain distinguishable from server-confirmed records.

### Phase 5 — Secure Synchronization

Tasks:

- Authenticated synchronization APIs
- Idempotency
- Conflict detection
- Retry logic
- Manual synchronization
- Synchronization monitoring

Acceptance:

- Repeated uploads do not create duplicates.
- Conflicts are not silently overwritten.
- Revoked devices cannot synchronize.
- Failed synchronization preserves local records safely.

### Phase 6 — Security Validation

Tasks:

- Penetration testing
- OTP security testing
- Device replacement testing
- Offline access testing
- Encryption testing
- XSS testing
- Session testing
- Synchronization integrity testing
- iOS Safari testing
- Android Chrome testing
- Desktop browser testing

Acceptance:

- No unresolved critical security vulnerability.
- No unauthorized access to another user’s information.
- No plaintext persistence of protected health data.
- All critical synchronization scenarios pass.

## 15. MANDATORY TEST SCENARIOS

1. New account registration.
2. Email verification.
3. Mobile OTP verification.
4. Login using mobile number and OTP.
5. Login without email.
6. Incorrect OTP.
7. Expired OTP.
8. Repeated OTP requests.
9. Login from the existing trusted device.
10. Login from a new device.
11. Attempt to activate two devices simultaneously.
12. Authorized device replacement.
13. Lost-device replacement.
14. Device revocation.
15. Opening the application offline.
16. Offline authorization expiration after 72 hours.
17. Local PIN verification.
18. Biometric unlocking where supported.
19. Offline record creation.
20. Closing the application before synchronization.
21. Restoring connectivity.
22. Synchronizing pending records.
23. Repeated synchronization requests.
24. Conflicting updates.
25. Clearing browser storage.
26. Changing the registered phone number.
27. Attempting to access another user’s information.
28. Attempting synchronization from a revoked device.

## 16. TECHNICAL LIMITATIONS

The implementation team must account for:

- PWA does not provide identical security guarantees to native applications.
- Browser storage may be cleared.
- Background synchronization is not consistently supported.
- Biometric support varies by platform.
- Browser cryptographic key storage is not automatically hardware-backed.
- Remote revocation cannot immediately reach a fully offline device.
- SMS OTP is vulnerable to some phone-number takeover attacks.
- Local encryption does not eliminate XSS and malicious-code risks.

If the required security guarantees exceed PWA capabilities, native application development or platform-specific secure storage must be evaluated.

No security guarantee may be claimed without technical validation.

## 17. RELATIONSHIP WITH MASTER ARCHITECTURE

This document supplements:

`docs/architecture/SOLINT_PUBLIC_MASTER_ARCHITECTURE.md`

The master architecture must reference this document in:

- Security & Privacy
- Authentication
- PWA Architecture
- Data Architecture
- API Architecture
- Testing & Quality Assurance

The architecture must remain compatible with the independent design of SOLINT PUBLIC and SOLINT NETWORK.

## 18. IMPLEMENTATION GOVERNANCE

Before sending any implementation prompt to Lovable:

1. Review the actual project configuration.
2. Verify the authentication provider and SMS delivery mechanism.
3. Review database tables and access policies.
4. Check device registration and uniqueness enforcement.
5. Review encryption and key-management design.
6. Verify offline-data policies.
7. Review API contracts.
8. Identify security risks.
9. Define migration and rollback requirements.
10. Prepare acceptance tests.
11. Correct all critical issues.
12. Submit the final reviewed implementation plan.

No implementation must silently change existing SOLINT NETWORK functionality.

## 19. FINAL ARCHITECTURAL DECISIONS

| Decision | Final Status |
|---|---|
| Email for initial registration | CONFIRMED |
| Email verification | CONFIRMED |
| Email for subsequent login | NOT USED |
| Mobile number as login identifier | CONFIRMED |
| SMS OTP for every online login | CONFIRMED |
| Password-based login | NOT REQUIRED INITIALLY |
| One active trusted device per account | CONFIRMED |
| Simultaneous trusted devices | PROHIBITED |
| Secure device replacement | REQUIRED |
| Offline access | CONFIRMED |
| Maximum offline authorization | 72 HOURS |
| Local PIN | REQUIRED AS OFFLINE UNLOCK OPTION |
| Device biometrics | SUPPORTED WHERE AVAILABLE |
| IndexedDB | PROPOSED |
| Encryption before local persistence | REQUIRED |
| Server as source of truth | REQUIRED |
| Automatic synchronization | REQUIRED |
| Conflict detection | REQUIRED |
| Direct PUBLIC access to NETWORK database | PROHIBITED |

## FINAL STATUS

- **Document Status:** DRAFT — Updated Architecture Baseline
- **Product Decisions:** CONFIRMED
- **Technical Validation:** PENDING
- **Implementation Status:** NOT STARTED
- **Next Required Step:** Phase 0 — Security Threat Modeling and Technical Validation
- **Production Deployment:** PROHIBITED UNTIL SECURITY VALIDATION IS COMPLETED

---

## Review Notes

The document records the intended product policy, but several security properties depend on browser and operating-system capabilities and must be validated before implementation. In particular, offline access on a revoked or lost device cannot be remotely disabled until that device reconnects; PWA key storage and biometric unlocking vary by platform; and encryption at rest does not protect against active same-origin script compromise such as XSS. The stated single-device and 72-hour policies should therefore be treated as server-enforced online policies plus bounded local controls, not as guarantees that a fully offline device can be remotely controlled. Phase 0 should define and test these limitations, key lifecycle and recovery, threat model, jurisdiction-specific privacy obligations, and behavior when browser storage is cleared or corrupted.
