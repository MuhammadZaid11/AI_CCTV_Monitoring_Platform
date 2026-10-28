# Software requirements specification

**Product:** AI CCTV Monitoring Platform  
**Document version:** 0.1  
**Date:** 27 October 2026  
**Status:** Draft for product-owner review. Not baseline-approved. Not implemented.

This specification is the proposed requirements authority for a commercial product. It replaces the unpublished scope of `docs/AI_CCTV_Monitoring_Agent_SRS_v1.0.docx` for tenancy, licensing, releases, and technology selection. That Word file is retained as history. Its own text says it was not frozen.

If a requirement here disagrees with [product-scope.md](product-scope.md), stop and resolve it before building. The traceability matrix is [requirements-traceability.md](requirements-traceability.md).

## 1. Purpose

This document defines what the platform must do, the constraints on how it is built and operated, and the acceptance criteria for a controlled pilot and later releases.

The business objective is a product that can be piloted and, after the gates in [product-scope.md](product-scope.md), sold to authorized customers. Those customers include CCTV installers, security companies, warehouses, factories, offices, retail sites, and commercial buildings.

The product receives authorized CCTV streams, detects and tracks people, evaluates configurable rules, stores reviewable events and permitted evidence, and helps authorized operators investigate events. It reduces the need to watch every screen continuously. It does not replace security staff and it does not detect every incident.

## 2. Scope

In scope, out of scope, and release names are defined in [product-scope.md](product-scope.md).

Release 1 is a controlled pilot. Release 1.1 is commercial operations. Release 2 is future AI. Release 3 is enterprise deployment. Later releases are specified here so the Release 1 data model does not block them. They are not authorization to build them.

## 3. Repository facts at the time of writing

Verified on 27 October 2026:

- No application code, tests, containers, or git history.
- Python, Node.js, Docker, and FFmpeg were not on the workstation PATH.
- The only prior project file was the Word SRS named above.

Therefore every requirement status in this document is **Not implemented**, and every verification-evidence cell is **None**.

## 4. Users

| Class | Need | Release |
| --- | --- | --- |
| Customer administrator | Manage users, cameras, zones, and retention for one organization | R1 |
| Security operator | See camera health, review events, mark outcomes | R1 |
| Auditor | Read events, permitted evidence, and the audit log | R1 |
| Platform administrator | Operate a hosted deployment without standing access to customer footage | R1 control; hosted sale is later |
| Partner user | Installer or reseller acting only inside a customer grant | R1.1 |
| Worker service | Process assigned cameras through a database role, not a browser session | R1 |
| Host operator | Install, back up, and upgrade the server. This is an operating-system duty, not an application role | R1 procedures |

A person may hold one role per organization. Platform administration is not implied by customer administration.

## 5. Assumptions

Unverified, and listed so they can be challenged:

- The customer has authority to provide every configured stream.
- The first live deployment has a server on a trusted network with the cameras, not a public RTSP endpoint.
- English is an acceptable pilot language.
- One organization and one site are enough for the first pilot's user interface, while the database still stores organization and site on every relevant row.
- Human reviewers are available. The product is not the only control.

Verified facts and unresolved choices are in [open-decisions.md](open-decisions.md).

## 6. Constraints

- No facial recognition, biometric identification, emotion recognition, or cross-camera person re-identification in the product unless a future requirement explicitly overturns this and passes a new privacy review. No such requirement exists.
- No automatic contact with emergency services and no automatic control of doors or alarms.
- Detection output is a signal for a person to review.
- Credentials, tenant identifiers, storage paths, and URLs are configuration, not source code.
- Prices are not a software requirement. Commercial packaging is in [commercial-plan.md](commercial-plan.md).
- Legal compliance is not claimed. [data-governance.md](data-governance.md) lists questions for counsel.
- The pilot architecture is a modular monolith and worker processes, as recommended in [architecture.md](architecture.md), unless the owner changes OD-014, OD-015, or OD-016.

## 7. External interfaces

| Interface | Requirement |
| --- | --- |
| Cameras | RTSP or RTSPS with a tested codec profile. The profile is empty until Milestone 3 records it. |
| Browser | Current desktop and mobile browsers the owner names in Milestone 7. The UI is a website. |
| API | HTTPS JSON under `/api/v1` for any non-loopback deployment. Loopback HTTP is a development setting only. |
| Database | PostgreSQL on a private network. |
| Evidence | Private volume behind a storage interface. |
| Time | UTC stored. Site timezone used for display. The host clock is synchronized by the operator. |
| Notifications | No external channel in Release 1. |
| Identity federation | Not in Release 1. |
| Model file | Loaded from a configured path with a recorded checksum. Not downloaded at runtime from an arbitrary URL. |

## 8. Functional requirements

Priority P0 is required for that release's gate. P1 is required for the release but can follow the core path inside the same release. P2 is conditional.

Verification methods: Test (automated), Demonstration (observed scenario), Inspection (review of design or configuration), Analysis (measurement report).

### 8.1 Release 1

| ID | Pri | Rel | Statement | Depends | Verify | AC | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| FR-001 | P0 | R1 | An authorized administrator can add, edit, disable, reconnect, and remove a camera in their organization. Remove stops processing and deletes the stored secret. Historical events remain until retention or an explicit purge. | FR-013, FR-027 | Test, Demonstration | AC-001 | Not implemented |
| FR-002 | P0 | R1 | Each camera has a unique id, display name, site, enabled flag, connection fields, status, last-frame time, processing delay, and safe error text. | FR-001 | Test | AC-001, AC-002 | Not implemented |
| FR-003 | P0 | R1 | The worker connects to an enabled RTSP or RTSPS camera, detects loss of frames, and reconnects with bounded exponential backoff and jitter. Retry limits are configurable. | FR-001 | Test, Demonstration | AC-002 | Not implemented |
| FR-004 | P0 | R1 | Person detection is performed only through a replaceable adapter. Each event stores the model version and inference configuration used. | FR-009 | Test | AC-003, AC-015 | Not implemented |
| FR-005 | P0 | R1 | An authorized administrator can define one or more restricted polygons per camera in normalized image coordinates. | FR-001, FR-019 | Test | AC-016 | Not implemented |
| FR-006 | P0 | R1 | The tracker maintains identities across frames well enough to tell a new entry from continued presence. Track loss behavior is configurable and documented, including the limitation that a lost track can hide a second person. | FR-004 | Test | AC-003, AC-004 | Not implemented |
| FR-007 | P0 | R1 | The system creates a person-entry or inward boundary-crossing event when a track's reference point meets the active rule's confidence and persistence conditions. The reference point definition is shown to the zone editor. | FR-005, FR-006 | Test, Demonstration | AC-003 | Not implemented |
| FR-008 | P0 | R1 | The system suppresses duplicate events for the same entry using track state and a configurable per-zone cooldown that survives a camera-process restart. | FR-007 | Test | AC-004 | Not implemented |
| FR-009 | P0 | R1 | Each event stores id, organization, site, camera, zone, rule version, type, UTC times, confidence, local track id, model version, inference configuration, review state, and evidence status. | FR-007, FR-027 | Test | AC-003, AC-005 | Not implemented |
| FR-010 | P0 | R1 | An authorized user can filter, sort, paginate, and open events by time, site, camera, zone, type, and review state. Page size is capped by the server. | FR-009, FR-028 | Test, Demonstration | AC-013 | Not implemented |
| FR-011 | P0 | R1 | Camera status is one of Disabled, Connecting, Online, Reconnecting, Degraded, or Error, using the definitions in the architecture. Online requires fresh processed frames. | FR-003 | Test | AC-002 | Not implemented |
| FR-012 | P0 | R1 | The system appends an audit record for login success and failure, logout, role changes, camera and zone changes, entitlement changes, event review transitions, evidence reads, support-grant changes, and retention changes. The record has time, actor, action, object, outcome, and request id. | FR-013 | Test | AC-017 | Not implemented |
| FR-013 | P0 | R1 | Users authenticate with a server-side session. Passwords are stored only as strong hashes. The server enforces the role matrix in section 8.5 on every protected operation. | None | Test | AC-006 | Not implemented |
| FR-014 | P0 | R1 | An authorized reviewer can move an event among New, Acknowledged, Resolved, and False Alert, including a note. Transitions are stored with actor and time. Acknowledged means a person looked, not that the activity was malicious. Resolved and False Alert can be reopened by a customer administrator, and the reopen is audited. | FR-009, FR-013 | Test, Demonstration | AC-005, AC-006 | Not implemented |
| FR-015 | P0 | R1 | The system deletes expired evidence bytes and expired metadata according to the configured retention, records the outcome, retries failed file deletes, and reconciles orphan files. | FR-022 | Test | AC-009 | Not implemented |
| FR-016 | P0 | R1 | Liveness and readiness endpoints exist for the API and the worker. Unauthenticated health output does not include secrets, tenant names, or camera URLs. | None | Test, Inspection | AC-012 | Not implemented |
| FR-017 | P0 | R1 | Camera targets accept only RTSP or RTSPS, reject other schemes, and connect only to addresses in the configured allowlist after DNS resolution. Link-local, loopback, and metadata addresses are denied except a non-customer test switch. | FR-001 | Test | AC-008 | Not implemented |
| FR-018 | P0 | R1 | Loss of a stream, stalled inference, database unavailability, or evidence-store failure moves the affected camera or subsystem to a visible degraded or error condition. The UI does not present that gap as a period of successful monitoring. | FR-011 | Test, Demonstration | AC-010 | Not implemented |
| FR-019 | P0 | R1 | A customer administrator can draw and edit a zone on an authorized still image from that camera in a desktop browser. The saved polygon is the polygon the rule engine uses. | FR-005, FR-032 | Demonstration, Test | AC-016 | Not implemented |
| FR-020 | P0 | R1 | Each camera has a configurable inference sample rate and a cap on in-flight frames. When the cap is reached, the worker drops frames, counts them, and continues. | FR-003 | Test | AC-011 | Not implemented |
| FR-021 | P0 | R1 | A fault or restart of one camera process does not stop processing of another camera on the same supervisor. | FR-003 | Test | AC-018 | Not implemented |
| FR-022 | P0 | R1 | When a rule creates an event and policy allows a snapshot, the worker stores one. If capture or storage fails, the event remains with evidence status failed and a safe reason. | FR-009 | Test | AC-005 | Not implemented |
| FR-023 | P0 | R1 | Camera passwords are write-only in the API, encrypted at rest, and absent from event records and client responses. | SEC-005 | Test | AC-008 | Not implemented |
| FR-024 | P0 | R1 | The dashboard provides camera health, event history, event detail, evidence review, user management, and settings, including loading, empty, error, offline, and degraded states. | FR-010, FR-011 | Demonstration | AC-013 | Not implemented |
| FR-025 | P0 | R1 | Health and event review workflows function in a desktop browser and at a mobile viewport width. Zone drawing is required on desktop widths and may be unavailable on small screens if the limitation is shown. | FR-024 | Demonstration | AC-013 | Not implemented |
| FR-026 | P0 | R1 | Saving a zone or rule creates a new version. Existing events keep the version that fired. | FR-005, FR-009 | Test | AC-003 | Not implemented |
| FR-027 | P0 | R1 | Cameras, zones, events, evidence, and memberships belong to an organization. Cameras and events belong to a site. A pilot can operate with one organization and a default site. | None | Test, Inspection | AC-007 | Not implemented |
| FR-028 | P0 | R1 | The server rejects access to an object whose organization the caller cannot access, including when the caller supplies another tenant's identifier. | FR-013, FR-027 | Test | AC-007 | Not implemented |
| FR-029 | P0 | R1 | Camera detail shows last-frame time and processing delay derived from worker timestamps, not from a client clock. | FR-002 | Test | AC-002 | Not implemented |
| FR-030 | P0 | R1 | Customer operations are under `/api/v1`. Errors return a stable code, a safe message, and a request id, without stack traces or secrets. | FR-013 | Test, Inspection | AC-006 | Not implemented |
| FR-031 | P0 | R1 | Enabling a camera checks a server-side entitlement for the maximum number of enabled cameras. The check cannot be skipped by the client. Release 1 uses a locally stored entitlement. | FR-001 | Test | AC-014 | Not implemented |
| FR-032 | P0 | R1 | The zone-editor reference image is stored as confidential evidence, is served only through an authorized request, and is deleted or replaced according to policy. | FR-019, SEC-008 | Test | AC-008, AC-016 | Not implemented |
| FR-033 | P0 | R1 | The product has no default vendor account that can read customer evidence. Cross-organization support access exists only as a time-bounded grant with an audit record. A single-customer on-prem install can ship with zero grants. | FR-012, FR-028 | Test, Inspection | AC-007, AC-017 | Not implemented |
| FR-034 | P0 | R1 | Camera error text tells an operator the category of failure, such as authentication, network, decode, or configuration, and omits passwords and raw URLs. | FR-011 | Test | AC-002 | Not implemented |

### 8.2 Release 1.1

| ID | Pri | Rel | Statement | Depends | Verify | AC | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| FR-040 | P0 | R1.1 | A platform administrator can create and suspend customer organizations without receiving evidence access. | FR-033 | Test | AC-014 | Not implemented |
| FR-041 | P0 | R1.1 | A customer administrator can create and edit multiple sites, and cameras are placed in a site. | FR-027 | Test | AC-007 | Not implemented |
| FR-042 | P0 | R1.1 | A customer administrator can grant a partner user a defined permission set on that organization until a stated time, and can revoke it. The partner cannot grant themselves another customer. | FR-028 | Test | AC-006, AC-007 | Not implemented |
| FR-043 | P0 | R1.1 | Plan limits on cameras and features are enforced on the server for every relevant API and worker action. | FR-031 | Test | AC-014 | Not implemented |
| FR-044 | P0 | R1.1 | License activation, renewal, expiry, suspension, and transfer follow a documented lifecycle. Offline behavior is documented. Expiry pauses new processing as OD-017 decides and does not delete evidence. | FR-043 | Test, Inspection | AC-014 | Not implemented |
| FR-045 | P0 | R1.1 | Authorized users can configure notification rules for one approved channel. Each attempt stores delivery status. A failed send does not mark the event as delivered. | FR-009 | Test | AC-010 | Not implemented |
| FR-046 | P1 | R1.1 | Authorized users can export event metadata they can already read. The export is audited. Evidence files are excluded unless a separate permission is on. | FR-010, FR-012 | Test | AC-006 | Not implemented |
| FR-047 | P1 | R1.1 | An operator of the vendor or partner can follow a documented onboarding checklist for a new organization without using a shared customer password. | FR-042 | Demonstration, Inspection | AC-012 | Not implemented |
| FR-048 | P2 | R1.1 | Short evidence clips can be enabled per organization after a privacy review. Clip failure preserves the event, as for snapshots. | FR-022, OD-010 | Test | AC-005 | Not implemented |
| FR-049 | P2 | R1.1 | If OD and pilot feedback approve a PWA, the web client can be installed and must document offline, push, and background limits. Live video is not an offline feature. | FR-025 | Demonstration | AC-013 | Not implemented |

### 8.3 Release 2

| ID | Pri | Rel | Statement | Depends | Verify | AC | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| FR-050 | P0 | R2 | Authorized users can import historical recordings they are allowed to use, and search them by organization, site, camera, and time. | FR-027 | Test | AC-007 | Not implemented |
| FR-051 | P0 | R2 | Authorized reviewers can label footage as correct detection, false alert, missed detection, or ambiguous, with the reviewer and time stored. | FR-050 | Test | AC-015 | Not implemented |
| FR-052 | P0 | R2 | Datasets are versioned, record source provenance, run quality checks, and keep training, validation, and test material from sharing the same incident. | FR-051 | Test, Inspection | AC-015 | Not implemented |
| FR-053 | P0 | R2 | Training and evaluation run in isolated workers with separate credentials and resource limits. The registry stores dataset version, model version, code version, configuration, and metrics. | FR-052 | Inspection, Test | AC-015 | Not implemented |
| FR-054 | P0 | R2 | A new model is deployed only after an explicit approval that compares it with the current model. Rollback is possible. The approval and the rollback are audited. Quality is reviewed after deployment. | FR-053, AI-010 | Demonstration, Inspection | AC-015 | Not implemented |
| FR-055 | P1 | R2 | Anomaly detection, if built, is limited to a named use case, emits a score distinct from a rule event, and requires a threshold and human review. | FR-014 | Test, Inspection | AC-020 | Not implemented |
| FR-056 | P1 | R2 | Natural-language queries return only stored events and evidence the caller may access, each with source event, time, and camera. If footage is not indexed or the answer is unreliable, the response says so and does not invent events. | FR-010, FR-028 | Test | AC-007 | Not implemented |

### 8.4 Release 3

| ID | Pri | Rel | Statement | Depends | Verify | AC | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| FR-060 | P1 | R3 | Inference can run on a customer-site edge computer using the same event model as the central API. | FR-004 | Demonstration | AC-012 | Not implemented |
| FR-061 | P1 | R3 | A documented offline period queues synchronization and makes conflicts visible. It does not silently drop events. | FR-009 | Test | AC-010 | Not implemented |
| FR-062 | P1 | R3 | The customer can authenticate with an enterprise identity provider. Application roles remain in this product. | FR-013 | Test | AC-006 | Not implemented |
| FR-063 | P2 | R3 | Native mobile clients are built only after a written decision that a PWA cannot meet a named need. They use `/api/v1` and the same authorization. | FR-030 | Inspection | AC-006 | Not implemented |
| FR-064 | P1 | R3 | Enterprise deployment has a documented, rehearsed upgrade and a backup and recovery objective agreed in the contract. | NFR-007, NFR-020 | Demonstration | AC-012 | Not implemented |

### 8.5 Role matrix for Release 1

| Action | Customer admin | Operator | Auditor | Platform admin |
| --- | --- | --- | --- | --- |
| Manage users in the organization | Yes | No | No | No |
| Manage cameras and zones | Yes | No | No | No |
| View health and events | Yes | Yes | Yes | Only with a support grant |
| Review event state | Yes | Yes | No | Only if the grant includes it |
| View evidence | Yes | Yes | Yes | Only if the grant includes it |
| Read audit | Yes | No | Yes | Own admin actions; customer audit only with grant |
| Change entitlements | No | No | No | Yes |
| Read camera passwords | No | No | No | No |

Partner columns are added when FR-042 is implemented. Until then, partner behavior is unspecified and must not be improvised in the client.

### 8.6 Event and camera states

Event states: New, Acknowledged, Resolved, False Alert. Definitions and reopen behavior are in FR-014.

Camera states are in FR-011 and [architecture.md](architecture.md).

## 9. Non-functional requirements

| ID | Pri | Rel | Statement | Depends | Verify | AC | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| NFR-001 | P0 | R1 | The team measures a profile of five cameras at 720p and 10 FPS input on named hardware, with sampling configured explicitly. The profile is a test, not a capacity warranty. | AI-004 | Analysis | AC-011 | Not implemented |
| NFR-002 | P0 | R1 | The team measures p95 event commit latency against the boundary in the AI evaluation plan. Under two seconds is a provisional target to test, not a promise. | NFR-001 | Analysis | AC-011 | Not implemented |
| NFR-003 | P0 | R1 | After a recoverable worker or stream failure, the supervisor restarts the affected camera. Downtime is visible. No uptime percentage is committed. | FR-018, FR-021 | Test | AC-010 | Not implemented |
| NFR-004 | P0 | R1 | Before a site is told that detection quality is acceptable, precision, recall, misses, and false alerts are reported on a labelled set using the evaluation plan. | AI-005 | Analysis | AC-011 | Not implemented |
| NFR-005 | P0 | R1 | The codebase is modular, with pinned dependencies, formatting, linting, and automated tests for new behavior. | None | Inspection | AC-012 | Not implemented |
| NFR-006 | P0 | R1 | Logs are structured and include request ids and camera ids where relevant. Operators can see per-camera status, frame age, reconnects, and drop counts. | FR-016 | Inspection, Test | AC-010 | Not implemented |
| NFR-007 | P0 | R1 | Backup and restore of the database and evidence are documented and have been executed once on a disposable environment. | None | Demonstration | AC-012 | Not implemented |
| NFR-008 | P0 | R1 | Supported operating system, Docker version, codecs, and CPU or GPU mode are written down after tests. Unsupported combinations are listed as untested. | None | Inspection | AC-011 | Not implemented |
| NFR-009 | P0 | R1 | A person who has been shown the pilot UI can find a camera, filter today's events, and mark an event, without using a shell. | FR-024 | Demonstration | AC-013 | Not implemented |
| NFR-010 | P0 | R1 | A clean machine can be brought to a running pilot stack from the repository instructions and Compose files. | None | Demonstration | AC-012 | Not implemented |
| NFR-011 | P0 | R1 | Schema changes ship as versioned migrations. Hand-edited production schemas are not the upgrade path. | None | Inspection | AC-012 | Not implemented |
| NFR-012 | P0 | R1 | Queues, camera counts, and container resources are bounded. Overload drops or delays work in a visible way. | FR-020 | Test | AC-010, AC-011 | Not implemented |
| NFR-013 | P0 | R1 | The versioned HTTP API is sufficient for a future client to perform the operator tasks without screen-scraping. | FR-030 | Inspection | AC-006 | Not implemented |
| NFR-014 | P0 | R1 | Layouts used for monitoring remain usable at the mobile and desktop widths named in the test plan. | FR-025 | Demonstration | AC-013 | Not implemented |
| NFR-015 | P0 | R1 | Environment-specific values come from configuration. A new tenant or path does not require a code change. | None | Inspection | AC-012 | Not implemented |
| NFR-016 | P0 | R1 | Stored timestamps are UTC. The UI renders them in the site timezone. | None | Test | AC-003 | Not implemented |
| NFR-017 | P1 | R1 | The pilot user interface is English. Other languages wait for a product decision. | None | Inspection | AC-013 | Not implemented |
| NFR-018 | P0 | R1 | Install, upgrade, rollback, backup, restore, and incident steps exist in the runbook and match the software that was actually built. | NFR-010 | Inspection | AC-012 | Not implemented |
| NFR-019 | P0 | R1 | Event metadata remains queryable when snapshot storage fails. | FR-022 | Test | AC-005 | Not implemented |
| NFR-020 | P0 | R1 | Upgrade and rollback have been rehearsed using a pinned previous and next image and a database backup. | NFR-007, NFR-011 | Demonstration | AC-012 | Not implemented |

## 10. Security requirements

Detail and residual risk are in [threat-model.md](threat-model.md).

| ID | Pri | Rel | Statement | Depends | Verify | AC | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| SEC-001 | P0 | R1 | Authentication uses hashed passwords, expiring revocable sessions, and logout. There is no default account password in the shipped configuration. | FR-013 | Test | AC-006 | Not implemented |
| SEC-002 | P0 | R1 | Every protected read and write checks authorization on the server. | FR-013 | Test | AC-006 | Not implemented |
| SEC-003 | P0 | R1 | Tenant isolation is enforced in queries and covered by automated cross-tenant tests. Row-level security is included if OD-014 stays at the recommendation. | FR-028 | Test | AC-007 | Not implemented |
| SEC-004 | P0 | R1 | Secrets are supplied by the environment or a secret file. The repository contains an example file with placeholders and ignore rules for real secret files. | None | Inspection | AC-008 | Not implemented |
| SEC-005 | P0 | R1 | RTSP credentials are encrypted at rest, excluded from logs and API bodies, and not placed in browser storage. | FR-023 | Test | AC-008 | Not implemented |
| SEC-006 | P0 | R1 | Camera configuration cannot be used to request arbitrary URLs, files, or non-camera networks. | FR-017 | Test | AC-008 | Not implemented |
| SEC-007 | P0 | R1 | Any deployment that is not loopback-only uses TLS. | NFR-010 | Inspection | AC-012 | Not implemented |
| SEC-008 | P0 | R1 | Evidence bytes are returned only after an authorization check. Files are outside the public web root. | FR-022 | Test | AC-008 | Not implemented |
| SEC-009 | P0 | R1 | The API validates types, lengths, identifiers, polygons, and body size, and rejects unexpected fields that would change role or organization. | FR-030 | Test | AC-006 | Not implemented |
| SEC-010 | P0 | R1 | Authentication attempts are rate limited. The limit slows repeated failures and leaves a recovery path for the legitimate administrator. | SEC-001 | Test | AC-006 | Not implemented |
| SEC-011 | P0 | R1 | Dependencies and container images are pinned, and a vulnerability scan is run in CI before a pilot build is offered. | NFR-005 | Inspection | AC-012 | Not implemented |
| SEC-012 | P0 | R1 | Audit records are append-only for the application database role and are readable only by authorized roles. | FR-012 | Test | AC-017 | Not implemented |
| SEC-013 | P0 | R1 | Backup files are protected at the same sensitivity as the database and evidence. The runbook names who can read them. | NFR-007 | Inspection | AC-012 | Not implemented |
| SEC-014 | P0 | R1 | Application containers run as non-root, with resource limits, dropped capabilities where the base image allows, and without the Docker socket. | NFR-010 | Inspection | AC-012 | Not implemented |
| SEC-015 | P0 | R1 | The runbook contains initial steps for credential compromise, suspected cross-tenant access, and disk loss. | NFR-018 | Inspection | AC-010 | Not implemented |
| SEC-016 | P0 | R1 | Cookie-authenticated mutations are protected against cross-site request forgery. Session cookies are HttpOnly. | SEC-001 | Test | AC-006 | Not implemented |
| SEC-017 | P0 | R1 | The worker's database role can process cameras and events and cannot change memberships or read other administrative secrets it does not need. | FR-013 | Inspection, Test | AC-008 | Not implemented |
| SEC-018 | P0 | R1 | Support access matches FR-033. | FR-033 | Test | AC-007 | Not implemented |
| SEC-019 | P0 | R1.1 | A partner token or session cannot read a customer who has not granted access, including after grant expiry. | FR-042 | Test | AC-007 | Not implemented |
| SEC-020 | P0 | R1 | The Release 1 build has no feature that enrolls a face, estimates emotion, or links a track to a real-world identity. | PRIV-002 | Inspection, Test | AC-019 | Not implemented |
| SEC-021 | P0 | R1 | If OD-007 is accepted, customer administrators and platform administrators must enroll a second factor before they can change users or cameras. | SEC-001 | Test | AC-006 | Not implemented |
| SEC-022 | P0 | R1 | The proxy sets baseline browser security headers for the dashboard. | SEC-007 | Inspection | AC-012 | Not implemented |
| SEC-023 | P0 | R1 | PostgreSQL is not published on the host network in the Compose file. Only the proxy port is published, bound as the deployment guide states. | NFR-010 | Inspection | AC-012 | Not implemented |
| SEC-024 | P0 | R1.1 | Customers cannot raise their own camera limit by editing the browser. Server enforcement is authoritative. On-prem owners who control the database can still edit rows; the administrator guide says that this is outside the license control. | FR-043 | Test, Inspection | AC-014 | Not implemented |
| SEC-025 | P0 | R1 | Automated tests fail if logs from a camera connection contain a password or userinfo. | SEC-005 | Test | AC-008 | Not implemented |
| SEC-026 | P1 | R1 | A documented cadence exists for reviewing dependency advisories. The pilot does not require a claimed response time the team has not staffed. | SEC-011 | Inspection | AC-012 | Not implemented |

## 11. Privacy and data-governance requirements

| ID | Pri | Rel | Statement | Depends | Verify | AC | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| PRIV-001 | P0 | R1 | The product is used only with footage the customer is authorized to process. The pilot agreement records that assertion per site. | None | Inspection | AC-012 | Not implemented |
| PRIV-002 | P0 | R1 | The system does not provide facial recognition, biometric identity matching, or emotion recognition. | SEC-020 | Inspection | AC-019 | Not implemented |
| PRIV-003 | P0 | R1 | Persistent images are limited to event snapshots and a single current reference frame per camera, unless a later approved feature adds clips. | FR-022 | Inspection | AC-005 | Not implemented |
| PRIV-004 | P0 | R1 | Retention periods exist for evidence, event metadata, audit, and logs, and are the periods the owner approved under OD-008. | FR-015 | Test, Inspection | AC-009 | Not implemented |
| PRIV-005 | P0 | R1 | An organization can be offboarded by an audited purge of secrets and evidence. Deletion is by time, event, camera, site, or organization. | FR-015 | Test | AC-009 | Not implemented |
| PRIV-006 | P0 | R1 | Evidence reads are access-controlled and audited. | FR-012, SEC-008 | Test | AC-008, AC-017 | Not implemented |
| PRIV-007 | P0 | R1 | Secrets, footage, events, and logs are handled according to the classes in the data-governance document. | SEC-004 | Inspection | AC-008 | Not implemented |
| PRIV-008 | P0 | R1 | Retention and purge jobs affect only the organization they are run for, except a platform operation that names its target. | FR-027, FR-015 | Test | AC-007, AC-009 | Not implemented |
| PRIV-009 | P0 | R2 | Footage is copied into a training set only with a recorded permission for that organization and that purpose. | FR-052 | Inspection | AC-015 | Not implemented |
| PRIV-010 | P0 | R1 | The system does not contact emergency services or actuate physical security equipment. | None | Inspection | AC-019 | Not implemented |
| PRIV-011 | P0 | R1 | The product presents events for human review before any consequential decision. | FR-014 | Inspection | AC-020 | Not implemented |
| PRIV-012 | P0 | R1 | Documentation and UI do not claim compliance with a law until counsel has completed the review in the data-governance document and the owner records the result. | None | Inspection | AC-020 | Not implemented |
| PRIV-013 | P0 | R1 | Before any vendor-hosted pilot, counsel records who determines the purposes of processing. | OD-012 | Inspection | AC-012 | Not implemented |
| PRIV-014 | P0 | R1 | The default deployment does not transfer footage to a vendor cloud or another country. | OD-002 | Inspection | AC-012 | Not implemented |
| PRIV-015 | P0 | R1 | Customer-facing materials state that notices and lawful placement are the customer's duty. | PRIV-001 | Inspection | AC-020 | Not implemented |
| PRIV-016 | P0 | R1 | Backup handling is included in retention so deleted evidence is not kept indefinitely on backup media without a decision. | PRIV-004, SEC-013 | Inspection | AC-009 | Not implemented |
| PRIV-017 | P0 | R1 | TLS is used for the dashboard outside loopback, and camera-password encryption is on. Disk encryption is an install requirement recorded for the pilot, or the agreement states that it is absent. | SEC-007, SEC-005 | Inspection | AC-008 | Not implemented |
| PRIV-018 | P0 | R1 | User documentation states that the product cannot erase one person's images selectively, because it does not identify people. | PRIV-002, PRIV-005 | Inspection | AC-019 | Not implemented |

## 12. AI and model-lifecycle requirements

| ID | Pri | Rel | Statement | Depends | Verify | AC | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| AI-001 | P0 | R1 | A single adapter interface is the only detection path used by the rule engine. | FR-004 | Test, Inspection | AC-003 | Not implemented |
| AI-002 | P0 | R1 | Events store model version and inference configuration. | FR-004, FR-009 | Test | AC-015 | Not implemented |
| AI-003 | P0 | R1 | The product and its docs state measured results only when an evaluation report exists for that build and site. | NFR-004 | Inspection | AC-011, AC-020 | Not implemented |
| AI-004 | P0 | R1 | CPU, memory, and GPU use are measured on the benchmark sheet before any camera-capacity sentence is used with a customer. | NFR-001 | Analysis | AC-011 | Not implemented |
| AI-005 | P0 | R1 | The team can produce precision, recall, false alerts per camera-hour, misses, latency, and dropped-frame counts as defined in the evaluation plan, when labels exist. | NFR-004 | Analysis | AC-011 | Not implemented |
| AI-006 | P0 | R1 | Tracking limitations, including occlusion and ID switches, are written next to the tracker the release uses. | FR-006 | Inspection | AC-004 | Not implemented |
| AI-007 | P0 | R1 | The Release 1 detector is configured for the person class only. | SEC-020 | Test | AC-019 | Not implemented |
| AI-008 | P0 | R1 | Numeric quality gates are set by the owner after a baseline report. They are not part of this draft. | OD-018 | Inspection | AC-011 | Not implemented |
| AI-009 | P0 | R1 | The labelled set and match window are defined before the confirmation numbers are computed. | AI-005 | Inspection | AC-011 | Not implemented |
| AI-010 | P0 | R2 | Model promotion requires a comparison, an approval record, rollback, and a check that evaluation splits did not leak the same incident. | FR-054 | Inspection | AC-015 | Not implemented |
| AI-011 | P0 | R2 | Anomaly scores are stored and displayed separately from rule events and cannot be described as deterministic violations. | FR-055 | Inspection | AC-020 | Not implemented |
| AI-012 | P0 | R1 | Every benchmark records hardware, codec, model checksum, sample rate, and network path. | AI-004 | Inspection | AC-011 | Not implemented |
| AI-013 | P0 | R1 | No model weight or inference library is added until its license is recorded against the commercial use in OD-004. | OD-004 | Inspection | AC-015 | Not implemented |
| AI-014 | P0 | R1 | Integration testing starts with two authorized cameras before any attempt to judge the five-camera profile. | NFR-001 | Demonstration | AC-001, AC-011 | Not implemented |
| AI-015 | P0 | R1 | The worker counts frames received, sampled, inferred, and dropped per camera. | FR-020 | Test | AC-011 | Not implemented |
| AI-016 | P1 | R1 | During a pilot, false-alert marks and camera-down time can be reviewed weekly. The product does not retrain itself from those marks. | FR-014 | Inspection | AC-011 | Not implemented |

## 13. Data requirements

Logical entities are specified in [architecture.md](architecture.md). Minimum rules:

- Identifiers are UUIDs.
- Tenant-owned rows include `organization_id`.
- Secrets live in a separate table from camera list columns.
- Rule versions are append-only.
- Evidence is a key and metadata in the database, and bytes in the store.
- Audit rows are not updated by the application role.
- Migrations are the only schema change path (NFR-011).

## 14. Acceptance criteria

These criteria are gates. None have been executed. "Provisional" targets are experiments.

| ID | Criterion | Release | Verify | Status |
| --- | --- | --- | --- | --- |
| AC-001 | On a clean pilot configuration, an administrator adds two authorized RTSP cameras. Both reach a truthful health state. Credentials do not appear in the browser responses or logs. | R1 | Demonstration, Test | Not run |
| AC-002 | Disconnecting one camera shows Reconnecting or Error for that camera, with last-frame time no longer advancing. The camera returns to Online after the stream returns, within the documented backoff, without a manual container restart. Processing delay is visible. | R1 | Demonstration, Test | Not run |
| AC-003 | On the agreed labelled set, a qualifying person entry creates an event with the correct organization, site, camera, zone, rule version, model version, and server time. | R1 | Analysis, Test | Not run |
| AC-004 | Remaining inside the zone does not create a new event per frame. A second entry follows the written cooldown and re-entry rule. A process restart during cooldown does not immediately duplicate the alert. | R1 | Test | Not run |
| AC-005 | An authorized user opens event detail and, when a snapshot exists, sees it. Another role without evidence permission is denied. When snapshot storage fails, the event is still present with evidence status failed. | R1 | Test, Demonstration | Not run |
| AC-006 | Unauthenticated requests are denied. Each Release 1 role is allowed the actions in the role matrix and denied the others, including by direct API call. | R1 | Test | Not run |
| AC-007 | A user of organization A who substitutes organization B's identifiers cannot read or change B's cameras, zones, events, evidence, or users. The response does not leak B's data. | R1 | Test | Not run |
| AC-008 | Passwords, session secrets, and evidence bytes are protected as specified in SEC-005 and SEC-008, including log redaction and cross-tenant file access. | R1 | Test | Not run |
| AC-009 | A test retention period deletes expired evidence bytes and then the corresponding metadata. Deletion is visible in audit or job history. A purge of a test organization removes its secrets and evidence. | R1 | Test | Not run |
| AC-010 | Killing a camera child, stopping the database briefly, and filling the evidence volume each produce the documented visible failure. When the dependency returns, processing recovers as the runbook describes. Metadata is not silently discarded. | R1 | Demonstration, Test | Not run |
| AC-011 | The benchmark sheet in the AI evaluation plan is filled for the two-camera test and for the five-camera profile on named hardware. Results are reported with limitations. No accuracy or capacity claim exceeds that report. The provisional two-second p95 target is recorded as met or not met, and either result is an input to OD-018, not a retroactive pass. | R1 | Analysis | Not run |
| AC-012 | A person who was not the author follows the install document on a clean environment and reaches a healthy stack. Backup, restore, upgrade, and rollback are each rehearsed once and the outcome is written down, including failures. | R1 | Demonstration | Not run |
| AC-013 | An operator completes health review and event review on a desktop browser and at a mobile viewport, including empty and error screens. Zone editing is demonstrated on a desktop viewport. | R1 | Demonstration | Not run |
| AC-014 | Release 1: enabling a camera beyond the entitlement is rejected by the API. Release 1.1: suspension and expiry follow the documented license behavior and cannot be overridden in the UI. Data remains after expiry. | R1 and R1.1 | Test | Not run |
| AC-015 | Release 1: every event created by the worker includes the model version that ran. Release 2: a model cannot be marked current without an approval record, and rollback restores the previous version id for new events. | R1 and R2 | Test, Inspection | Not run |
| AC-016 | A polygon drawn in the zone editor is stored and is the geometry used for a synthetic track that enters that polygon. | R1 | Test, Demonstration | Not run |
| AC-017 | The actions listed in FR-012 create audit rows that contain no secrets or image bytes. | R1 | Test | Not run |
| AC-018 | Inducing a crash in camera one's process leaves camera two's last-frame time advancing. | R1 | Test | Not run |
| AC-019 | A review of the Release 1 user interface and API schema finds no facial recognition, emotion recognition, identity enrollment, emergency-dispatch, or door-control feature. | R1 | Inspection | Not run |
| AC-020 | The event view states that the detection may be wrong or missed and that a person must review it before acting. Documentation does not claim legal compliance or perfect detection. | R1 | Inspection | Not run |

## 15. Verification

| Level | What it covers | When it starts |
| --- | --- | --- |
| Unit | Geometry, cooldown, validation, entitlement arithmetic, redaction | With each module |
| API and database | Auth, roles, tenancy, retention, audit | Milestone 1 onward |
| Worker | Reconnect, isolation, fake detector, evidence failure | Milestones 3 to 6 |
| Browser | AC-013 journeys | Milestone 7 |
| Security | Section 10 tests and the threat-model test plan | Before pilot |
| AI evaluation | [ai-evaluation-plan.md](ai-evaluation-plan.md) | After two cameras work, before any quality claim |
| Operations | AC-012 rehearsal | Before pilot |

A requirement is verified only when the traceability matrix points at a test or a dated report. Updating the requirement text without a test does not verify it.

## 16. Release roadmap

The ordered build is [implementation-plan.md](implementation-plan.md). Scope boundaries are [product-scope.md](product-scope.md).

Release 1 implementation starts only after the approval block below is signed by the product owner and the owner has either accepted the recommended defaults in the open-decisions list or recorded a change.

## 17. Traceability

Every ID in this document appears in [requirements-traceability.md](requirements-traceability.md) with component, test reference, and evidence.

## 18. Approval

This draft is not approved. Signatures below are empty on purpose.

| Role | Name | Decision | Date |
| --- | --- | --- | --- |
| Product owner | | Approve / revise | |
| Person accountable for security review before pilot | | Not this draft; a later review | |
| Person accountable for privacy questions | | Counsel review not started | |

Approval means engineering may start Milestone 0. It does not mean the software is safe, accurate, lawful, or ready to sell.

## 19. Change control

After approval, a change to a P0 requirement updates this document's version, the traceability row, and the tests. The Word SRS is not edited in parallel. New decisions go to [open-decisions.md](open-decisions.md) until they are accepted, then the affected requirement is edited.
