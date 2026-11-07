# SOLINT PUBLIC

# PHASE 0 — TECHNICAL VALIDATION & SECURITY THREAT MODELING

| Field | Value |
|---|---|
| Document ID | SOLINT-PUBLIC-PHASE-000 |
| Version | 1.0 |
| Status | DRAFT — Pre-Implementation Review |
| Date | November 2026 |
| Parent Document | `docs/architecture/SOLINT_PUBLIC_MASTER_ARCHITECTURE.md` |
| Related Documents | `docs/security/SOLINT_PUBLIC_SECURE_IDENTITY_AND_OFFLINE_ARCHITECTURE.md`; `docs/security/SOLINT_PUBLIC_SECURITY_IMPLEMENTATION_SPEC.md` |
| Project | SOLINT PUBLIC |
| Platform | Secure Offline-First PWA |

---

## 1. PURPOSE

Phase 0 establishes the technical, architectural, and security foundation required before implementation of SOLINT PUBLIC.

The objectives are:

1. Validate the proposed application architecture.
2. Identify technical dependencies.
3. Evaluate security risks.
4. Define trust boundaries.
5. Validate authentication and device-management requirements.
6. Evaluate offline-first PWA feasibility.
7. Define database and API design principles.
8. Establish data protection requirements.
9. Define implementation and testing gates.
10. Prevent architectural decisions that could require expensive redesign.

This phase is a design-time validation process.

It does not claim that the application, database, APIs, or security controls have already been implemented or tested.

## 2. SCOPE

Phase 0 covers:

- SOLINT PUBLIC architecture
- Authentication
- Identity verification
- Single trusted device
- OTP delivery
- Session management
- Offline authorization
- Local data encryption
- Data synchronization
- Database security
- API security
- PWA limitations
- Threat modeling
- Technical decision-making
- Development readiness

This phase does not implement the complete patient application.

It does not modify SOLINT NETWORK.

## 3. CONFIRMED PRODUCT REQUIREMENTS

The following requirements are approved by the product owner.

| ID | Requirement | Status |
|---|---|---|
| P0-001 | Email is used for registration and initial verification | CONFIRMED |
| P0-002 | Normal online login uses registered mobile number | CONFIRMED |
| P0-003 | Every online login requires SMS OTP | CONFIRMED |
| P0-004 | One active trusted device per account | CONFIRMED |
| P0-005 | Simultaneous trusted devices are prohibited | CONFIRMED |
| P0-006 | Maximum offline authorization is 72 hours | CONFIRMED |
| P0-007 | Offline unlocking uses PIN or supported biometrics | CONFIRMED |
| P0-008 | Permitted records can be created offline | CONFIRMED |
| P0-009 | Records synchronize after internet connectivity returns | CONFIRMED |
| P0-010 | Server remains the authoritative data source | CONFIRMED |
| P0-011 | PUBLIC cannot access NETWORK’s internal database directly | CONFIRMED |

These requirements must not be changed silently during implementation.

## 4. ARCHITECTURAL VALIDATION

### 4.1 Recommended Architecture

SOLINT PUBLIC should initially use a Modular Monolith architecture.

Proposed structure:

```text
SOLINT PUBLIC
│
├── PWA FRONTEND
│   ├── Authentication UI
│   ├── Patient Dashboard
│   ├── Health Records
│   ├── AI Health
│   ├── My Care
│   ├── Family Health
│   └── Offline Data Layer
│
├── BACKEND APPLICATION
│   ├── Identity Module
│   ├── Authentication Module
│   ├── Device Trust Module
│   ├── Patient Module
│   ├── Health Record Module
│   ├── AI Services Module
│   ├── Offline Authorization Module
│   ├── Synchronization Module
│   ├── Notification Module
│   └── Audit Module
│
├── API LAYER
│   ├── Authentication APIs
│   ├── Patient APIs
│   ├── Health Record APIs
│   ├── Device APIs
│   └── Synchronization APIs
│
└── DATA LAYER
    ├── PostgreSQL
    ├── Private File Storage
    ├── Audit Storage
    └── Background Job Processing
```

This is a logical architecture.

The final technology selection must be based on actual compatibility, security, cost, operational complexity, and integration requirements.

### 4.2 Architecture Principles

- Independent PUBLIC frontend.
- Independent application development lifecycle.
- Modular backend.
- Server-side authorization.
- API-based communication.
- No direct access to NETWORK database.
- Strict separation of clinical data and authentication secrets.
- Explicit data ownership.
- Auditability.
- Minimum necessary data storage.

Microservices are not required for the initial implementation.

## 5. TECHNOLOGY VALIDATION

The following technology options must be evaluated before selection.

| Layer | Candidate Technologies | Validation Requirement |
|---|---|---|
| Frontend | React, TypeScript, Vite | Lovable compatibility |
| UI | Tailwind CSS, component library | Accessibility and responsive design |
| PWA | Service Worker, Web App Manifest | iOS and Android support |
| Local Storage | IndexedDB | Offline reliability |
| Encryption | Web Crypto API | Key protection and browser compatibility |
| Backend | Node.js, TypeScript | Security and maintainability |
| Backend Framework | NestJS or Fastify | Authentication and API requirements |
| Database | PostgreSQL | Data integrity and access control |
| API Contracts | OpenAPI | Versioning and validation |
| Input Validation | Zod or equivalent | Server-side schema validation |
| Authentication | Dedicated identity/authentication service | OTP and session support |
| SMS | Replaceable SMS provider integration | Delivery, security and operational reliability |
| File Storage | Private object storage | Medical document protection |

No technology should be selected solely because it is the default option in Lovable.

The selected stack must support the security requirements of SOLINT PUBLIC.

### 5.1 Technology Selection Criteria

Each candidate must be evaluated according to:

- Security capabilities
- Lovable compatibility
- Integration complexity
- Operational cost
- Scalability
- Vendor lock-in
- Maintenance requirements
- Documentation quality
- Testing capabilities
- International deployment readiness

**Deliverable:** `docs/architecture/SOLINT_PUBLIC_TECH_STACK_DECISIONS.md`

## 6. IDENTITY AND AUTHENTICATION VALIDATION

### 6.1 Identity Model

The system must distinguish:

1. Account creation
2. Email ownership verification
3. Mobile number verification
4. Online authentication
5. Device trust
6. Offline local unlocking

These are separate processes.

Email verification must not be described as legal identity verification.

### 6.2 Online Authentication

Required workflow:

```text
User
  |
  v
Registered Mobile Number
  |
  v
Account Lookup
  |
  v
OTP Generation
  |
  v
SMS Delivery
  |
  v
OTP Verification
  |
  v
Device Authorization Check
  |
  v
Session Creation
```

All authentication decisions must be enforced by the backend.

### 6.3 OTP Validation

The technical review must verify:

- Secure random generation
- OTP expiration
- Single-use behavior
- Secure verification storage
- Rate limiting
- Brute-force prevention
- Resend restrictions
- Account enumeration protection
- SIM Swap risk
- SMS provider failure handling

A short numeric OTP must not be protected merely by an ordinary unsalted hash.

The implementation should use an appropriately protected verification mechanism, such as a server-side keyed HMAC, together with strict attempt limits.

### 6.4 Authentication Provider Selection

Evaluate:

- Custom OTP authentication
- Managed authentication provider
- Supabase Auth or equivalent, if compatible with the required policy

The selected provider must support:

- Mobile OTP
- Account verification
- Session management
- Secure token handling
- Device-associated authorization
- Account recovery
- Server-side integration

The system must not build a second independent authentication mechanism on top of an existing provider without a clear technical justification.

**Deliverable:** `docs/security/SOLINT_PUBLIC_AUTH_PROVIDER_EVALUATION.md`

## 7. SINGLE DEVICE SECURITY VALIDATION

### 7.1 Required Invariant

For each user:

```text
COUNT(active_trusted_devices) <= 1
```

This rule must be enforced at the database and backend levels.

### 7.2 Validation Requirements

The technical review must establish:

- How devices are registered.
- How device keys are generated.
- How device identity is verified.
- How device replacement is authorized.
- How old sessions are revoked.
- How the one-device rule is enforced transactionally.
- How simultaneous replacement requests are handled.

### 7.3 Device Replacement

The backend must guarantee atomic replacement.

Two simultaneous requests must not produce two active devices.

### 7.4 Offline Revocation Limitation

A completely offline device cannot receive an immediate revocation command.

The previous device may retain local access until its existing offline authorization expires.

**Maximum residual offline authorization: 72 hours.**

This is an accepted product limitation that must be documented.

The technical review must assess:

- Risk of lost-device access.
- Risk of stolen-device access.
- Encryption effectiveness.
- Local PIN protection.
- Automatic locking.
- Data minimization.
- Pending record recovery.

**Deliverable:** `docs/security/SOLINT_PUBLIC_DEVICE_TRUST_THREAT_MODEL.md`

## 8. OFFLINE-FIRST FEASIBILITY VALIDATION

### 8.1 Offline Requirements

The application must:

- Open without internet on a previously trusted device.
- Authenticate locally within the permitted offline period.
- Display permitted cached information.
- Allow permitted record creation.
- Preserve locally created records.
- Synchronize after connectivity returns.
- Lock sensitive information after authorization expiration.

### 8.2 PWA Limitations

The technical validation must specifically evaluate:

- iOS Safari storage behavior.
- Android Chrome storage behavior.
- IndexedDB persistence.
- Service Worker lifecycle.
- Background Sync availability.
- Application termination and restart.
- Browser data eviction.
- Device clock manipulation.
- Local cryptographic key protection.

The implementation must not depend exclusively on Background Sync.

### 8.3 Offline Authorization

The maximum validity period is 72 hours.

The system must evaluate how to enforce this duration despite:

- Device clock changes.
- Application restarts.
- Browser process termination.
- Offline device operation.
- Local storage modification attempts.

A browser-only timestamp must not be treated as an independently trusted clock.

If secure expiration cannot be sufficiently enforced on a supported platform, the architecture must use a more conservative policy or restrict offline access on that platform.

### 8.4 Offline Storage

The system must distinguish:

- Application shell cache
- Public application assets
- Encrypted health records
- Local synchronization queue
- Authentication credentials
- Offline authorization metadata

Sensitive health records must not be placed in ordinary Cache Storage as plaintext.

**Deliverable:** `docs/security/SOLINT_PUBLIC_OFFLINE_FEASIBILITY_REPORT.md`

## 9. ENCRYPTION AND KEY MANAGEMENT VALIDATION

### 9.1 Key Categories

The architecture must distinguish:

- Device Identity Key
- Local Data Encryption Key
- Session Credentials
- Offline Authorization Metadata

These keys must have separate purposes.

### 9.2 Technical Review

The review must establish:

- Key generation
- Key storage
- Key access
- Key rotation
- Key invalidation
- Recovery behavior
- Encryption scope
- Data decryption lifecycle

### 9.3 PWA Security Limitation

Browser-based key storage does not automatically provide hardware-backed protection.

Web Crypto API provides cryptographic operations but does not independently solve the problem of protecting keys against a compromised browser environment.

The implementation must evaluate WebAuthn and platform-specific capabilities where supported.

It must also assess the risk of malicious JavaScript and XSS.

### 9.4 Data Protection Rule

Sensitive information must be encrypted before persistent local storage.

The system must not store an unprotected master key alongside encrypted medical records.

**Deliverable:** `docs/security/SOLINT_PUBLIC_KEY_MANAGEMENT_DESIGN.md`

## 10. OFFLINE DATA AND SYNCHRONIZATION VALIDATION

### 10.1 Local Record Lifecycle

Required states:

```text
LOCAL_DRAFT
     |
     v
PENDING_SYNC
     |
     v
SYNCING
   /   \
  v     v
SYNCED  SYNC_FAILED
          |
          v
       RETRY
```

A separate state must exist for records requiring conflict review.

### 10.2 Data Provenance

Supported sources:

- `CLINICALLY_VERIFIED`
- `PATIENT_REPORTED`
- `IMPORTED`
- `AI_GENERATED`
- `OFFLINE_PATIENT_ENTRY`

Offline patient records must not automatically become clinically verified.

### 10.3 Synchronization Validation

The technical review must establish:

- Unique local record identifiers.
- Idempotency.
- Duplicate prevention.
- Record ownership.
- Versioning.
- Conflict detection.
- Retry behavior.
- Interrupted upload recovery.
- Server acknowledgment.
- Audit logging.

### 10.4 Device Replacement and Pending Records

The system must explicitly define what happens to unsynchronized records on the old device.

Mandatory rules:

- Revoked devices cannot synchronize.
- Local records must not be silently deleted.
- Unsynchronized records must not be automatically treated as server records.
- The user must be warned before device replacement if pending records exist and the old device is accessible.
- Recovery of records from an inaccessible old device must be treated as a separate security-controlled process.

Automatic transfer of local encryption keys or health records to the new device is prohibited unless a separately validated secure recovery mechanism is designed.

**Deliverable:** `docs/architecture/SOLINT_PUBLIC_SYNC_AND_CONFLICT_RESOLUTION_SPEC.md`

## 11. THREAT MODELING

A formal threat model must be completed before implementation approval.

The following categories must be evaluated.

### 11.1 STRIDE Threat Categories

| Category | Potential Threat | Required Control |
|---|---|---|
| Spoofing | Unauthorized user impersonation | OTP and device trust |
| Tampering | Modification of local health data | Authenticated encryption and integrity checks |
| Repudiation | Denial of important actions | Audit logging |
| Information Disclosure | Exposure of medical information | Encryption and access control |
| Denial of Service | OTP abuse or API flooding | Rate limiting and monitoring |
| Elevation of Privilege | Unauthorized access to another patient’s data | Server-side authorization and isolation |

### 11.2 Threat Scenarios

The threat model must include:

1. SIM Swap.
2. OTP interception.
3. Brute-force OTP attempts.
4. Account enumeration.
5. Lost device.
6. Stolen unlocked device.
7. Device replacement abuse.
8. Offline authorization manipulation.
9. Device clock rollback.
10. IndexedDB extraction.
11. Browser storage deletion.
12. XSS.
13. Malicious dependency.
14. Unauthorized API access.
15. IDOR vulnerabilities.
16. Cross-tenant data access.
17. Session theft.
18. Refresh-token reuse.
19. Duplicate synchronization.
20. Clinical record tampering.
21. Malicious file upload.
22. Unauthorized access through administrator privileges.

Each threat must include:

- Asset affected
- Attack surface
- Attack scenario
- Likelihood
- Impact
- Risk level
- Mitigation
- Residual risk
- Validation method

**Deliverable:** `docs/security/SOLINT_PUBLIC_THREAT_REGISTER.md`

## 12. TRUST BOUNDARIES

The architecture must explicitly define the following boundaries:

**Boundary A — User Device**

The browser and local storage must be treated as potentially compromised.

**Boundary B — Internet Transport**

All communication must use HTTPS and authenticated API requests.

**Boundary C — Backend**

The backend is responsible for authentication, authorization, validation, and business rules.

**Boundary D — Database**

The database must enforce integrity and access restrictions.

**Boundary E — External Providers**

SMS providers, AI services, file storage providers, and other integrations must receive only necessary information.

**Boundary F — SOLINT NETWORK**

PUBLIC must communicate with NETWORK through controlled APIs.

No direct access to NETWORK’s internal database is allowed.

## 13. DATABASE AND API VALIDATION

### 13.1 Database Requirements

The database design must cover:

- User accounts
- Verified email addresses
- Verified mobile numbers
- Authentication challenges
- Trusted devices
- User sessions
- Offline authorization
- Patient records
- Local synchronization references
- Audit events
- Data consent
- Device replacement history

The exact schema must be reviewed before migrations are created.

### 13.2 API Requirements

All APIs must support:

- Authentication
- Authorization
- Input validation
- Output validation
- Rate limiting
- Error handling
- API versioning
- Audit events
- Idempotency where applicable

API contracts must be documented before implementation.

Proposed location: `docs/api/`

### 13.3 PUBLIC and NETWORK Integration

The technical validation must determine:

- Which data is owned by PUBLIC.
- Which data is owned by NETWORK.
- Which data may be exchanged.
- Which API services are required.
- How patient consent is represented.
- How identity matching is performed.
- How external records are verified.

The existing NETWORK implementation must be inspected before finalizing integration contracts.

No assumptions about its actual database structure or authentication provider may be treated as verified facts.

## 14. DATA PRIVACY AND GOVERNANCE

The system must define:

- Data classification
- Data minimization
- User consent
- Data retention
- Account deletion
- Medical record retention
- Export procedures
- Data access history
- Third-party processing
- Data breach response

Medical records and account security information must have separate access policies.

AI services must not receive unnecessary personal information.

AI-generated information must remain distinguishable from clinically verified information.

## 15. PLATFORM COMPATIBILITY TESTING

The technical validation must evaluate:

| Platform | Required Validation |
|---|---|
| iOS Safari | PWA installation, storage, encryption, offline access |
| Android Chrome | PWA installation, storage, encryption, offline access |
| Desktop Chrome | PWA and offline behavior |
| Desktop Safari | Storage and service worker behavior |
| Desktop Edge | PWA and synchronization behavior |

Testing must cover:

- First installation
- First online authentication
- Application restart
- Offline reopening
- Offline record creation
- Connectivity restoration
- Synchronization
- Offline expiration
- Device replacement
- Storage loss

A feature must not be declared supported merely because it works in a desktop development environment.

## 16. TECHNICAL DECISION REGISTER

The following decisions must be finalized.

| ID | Decision | Status |
|---|---|---|
| TD-001 | Modular Monolith for initial backend | PROPOSED |
| TD-002 | React and TypeScript frontend | PROPOSED |
| TD-003 | PWA as initial delivery platform | CONFIRMED |
| TD-004 | PostgreSQL database | PROPOSED |
| TD-005 | SMS OTP for every online login | CONFIRMED |
| TD-006 | One active trusted device | CONFIRMED |
| TD-007 | Maximum 72-hour offline authorization | CONFIRMED |
| TD-008 | IndexedDB for structured offline data | PROPOSED |
| TD-009 | Web Crypto API for local encryption | PROPOSED |
| TD-010 | AES-GCM authenticated encryption | PROPOSED |
| TD-011 | API-first NETWORK integration | CONFIRMED |
| TD-012 | Direct PUBLIC database access to NETWORK | PROHIBITED |
| TD-013 | Authentication provider | PENDING |
| TD-014 | SMS provider | PENDING |
| TD-015 | Key management strategy | PENDING |
| TD-016 | Offline expiration enforcement | PENDING |
| TD-017 | Backend framework | PENDING |
| TD-018 | File storage provider | PENDING |

Every decision must have:

- Rationale
- Alternatives considered
- Security implications
- Cost implications
- Operational implications
- Approval status

## 17. PHASE 0 DELIVERABLES

Phase 0 is complete only when the following documents exist and have passed review.

| ID | Deliverable | Proposed Repository Path |
|---|---|---|
| D-001 | Technology Stack Decisions | `docs/architecture/SOLINT_PUBLIC_TECH_STACK_DECISIONS.md` |
| D-002 | Authentication Provider Evaluation | `docs/security/SOLINT_PUBLIC_AUTH_PROVIDER_EVALUATION.md` |
| D-003 | Device Trust Threat Model | `docs/security/SOLINT_PUBLIC_DEVICE_TRUST_THREAT_MODEL.md` |
| D-004 | Offline Feasibility Report | `docs/security/SOLINT_PUBLIC_OFFLINE_FEASIBILITY_REPORT.md` |
| D-005 | Key Management Design | `docs/security/SOLINT_PUBLIC_KEY_MANAGEMENT_DESIGN.md` |
| D-006 | Synchronization Specification | `docs/architecture/SOLINT_PUBLIC_SYNC_AND_CONFLICT_RESOLUTION_SPEC.md` |
| D-007 | Threat Register | `docs/security/SOLINT_PUBLIC_THREAT_REGISTER.md` |
| D-008 | Database Architecture | `docs/database/SOLINT_PUBLIC_DATABASE_ARCHITECTURE.md` |
| D-009 | API Contracts | `docs/api/` |
| D-010 | Phase 0 Validation Report | `docs/roadmap/SOLINT_PUBLIC_PHASE_0_VALIDATION_REPORT.md` |

These are deliverables to be produced during Phase 0, not existing completed documents.

## 18. PHASE 0 ACCEPTANCE GATES

### Gate 1 — Architecture

Requirements:

- Modular boundaries defined.
- PUBLIC and NETWORK separation validated.
- Technology dependencies identified.
- API strategy documented.

### Gate 2 — Authentication

Requirements:

- OTP provider evaluated.
- OTP security model documented.
- Single-device policy defined.
- Device replacement model validated.

### Gate 3 — Offline Security

Requirements:

- 72-hour policy technically evaluated.
- Local encryption model reviewed.
- Key management approach documented.
- PWA limitations documented.
- Data recovery risks assessed.

### Gate 4 — Data and Synchronization

Requirements:

- Database ownership model defined.
- API contracts reviewed.
- Idempotency designed.
- Conflict resolution defined.
- Data provenance rules documented.

### Gate 5 — Security

Requirements:

- Threat register completed.
- Critical risks mitigated or explicitly blocked.
- Privacy requirements defined.
- Security test plan approved.

### Gate 6 — Development Readiness

Requirements:

- Technology choices approved.
- Database design reviewed.
- API contracts approved.
- Dependencies mapped.
- Acceptance tests defined.
- No unresolved critical architectural issue.

Only after all gates pass may Phase 1 implementation begin.

## 19. RISK CLASSIFICATION

All identified risks must be classified as follows:

**CRITICAL**

A risk that could result in unauthorized medical data access, account takeover, serious data loss, or violation of core architecture.

Action: Implementation must stop until resolved.

**HIGH**

A risk that could compromise security, reliability, or data integrity.

Action: Must be resolved before the affected feature is implemented.

**MEDIUM**

A risk that can be mitigated through implementation controls or documented limitations.

Action: Must have an owner and mitigation plan.

**LOW**

A limited risk with acceptable impact.

Action: Document and monitor.

## 20. IMPLEMENTATION GOVERNANCE

Before any Lovable implementation prompt is prepared:

1. Verify the architecture decisions.
2. Review relevant technical dependencies.
3. Validate the selected authentication provider.
4. Review the database model.
5. Review the API contracts.
6. Evaluate security risks.
7. Review the migration plan.
8. Define expected code changes.
9. Define acceptance tests.
10. Confirm no conflict with SOLINT NETWORK.

Lovable must not be allowed to make major architectural decisions implicitly.

Every implementation step must have a defined scope and rollback strategy.

## 21. FINAL PHASE 0 STATUS

- **Phase:** 0 — Technical Validation & Security Threat Modeling
- **Product Requirements:** CONFIRMED
- **Architecture:** DRAFT
- **Technology Selection:** PENDING
- **Threat Model:** PENDING
- **Database Design:** PENDING
- **API Contracts:** PENDING
- **Security Validation:** PENDING
- **Implementation:** NOT STARTED
- **Production Deployment:** NOT APPROVED
- **Next Phase:** Phase 1 — Application Foundation & Secure Identity

---

**END OF DOCUMENT**
