# Requirements traceability

**Status:** Draft 0.1, 27 October 2026.

This matrix lists every requirement and acceptance-criterion ID in [SRS.md](SRS.md). Full statements live in the SRS. If a short description here and the SRS ever disagree, the SRS statement is the one to fix against, and then this row should be updated.

| Column | Meaning |
| --- | --- |
| Implementation | `Not implemented` until code and a passing test exist |
| Test reference | Name of an automated test or a dated manual script. `None` today |
| Evidence | Command output, report, or review record. `None` today |

No row is verified. Creating this file is not evidence that a requirement is met.

## Functional

| ID | Description | Priority | Release | Dependencies | Component | Implementation | Verification | Acceptance | Test reference | Evidence |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| FR-001 | Administrator can add, edit, disable, reconnect, and remove a camera in their organization; removal stops processing and deletes the secret; history remains until retention or purge | P0 | R1 | FR-013, FR-027 | Cameras | Not implemented | Test, Demonstration | AC-001 | None | None |
| FR-002 | Camera record includes id, name, site, enabled flag, connection fields, status, last-frame time, processing delay, and safe error text | P0 | R1 | FR-001 | Cameras, Worker | Not implemented | Test | AC-001, AC-002 | None | None |
| FR-003 | Worker connects to RTSP or RTSPS, detects frame loss, and reconnects with bounded exponential backoff and jitter | P0 | R1 | FR-001 | Worker | Not implemented | Test, Demonstration | AC-002 | None | None |
| FR-004 | Person detection goes through a replaceable adapter; events store model version and inference configuration | P0 | R1 | FR-009 | Inference, Events | Not implemented | Test | AC-003, AC-015 | None | None |
| FR-005 | Administrator can define one or more restricted polygons per camera in normalized coordinates | P0 | R1 | FR-001, FR-019 | Zones | Not implemented | Test | AC-016 | None | None |
| FR-006 | Tracker distinguishes a new entry from continued presence; track-loss behavior is configurable and its limits are documented | P0 | R1 | FR-004 | Tracking | Not implemented | Test | AC-003, AC-004 | None | None |
| FR-007 | Person-entry or inward boundary crossing creates an event when confidence and persistence are met; the reference point is explained in the editor | P0 | R1 | FR-005, FR-006 | Rules | Not implemented | Test, Demonstration | AC-003 | None | None |
| FR-008 | Duplicate events are suppressed by track state and a per-zone cooldown that survives a camera-process restart | P0 | R1 | FR-007 | Rules, Events | Not implemented | Test | AC-004 | None | None |
| FR-009 | Event stores organization, site, camera, zone, rule version, type, UTC times, confidence, local track id, model version, inference configuration, review state, and evidence status | P0 | R1 | FR-007, FR-027 | Events | Not implemented | Test | AC-003, AC-005 | None | None |
| FR-010 | Authorized user can filter, sort, paginate, and open events by time, site, camera, zone, type, and state, with a server page cap | P0 | R1 | FR-009, FR-028 | Events, Web | Not implemented | Test, Demonstration | AC-013 | None | None |
| FR-011 | Camera status is Disabled, Connecting, Online, Reconnecting, Degraded, or Error; Online requires fresh processed frames | P0 | R1 | FR-003 | Worker, Web | Not implemented | Test | AC-002 | None | None |
| FR-012 | Audit record for login, logout, role, camera, zone, entitlement, review, evidence read, support grant, and retention changes, without secrets | P0 | R1 | FR-013 | Audit | Not implemented | Test | AC-017 | None | None |
| FR-013 | Server-side session authentication and role checks on every protected operation; passwords stored only as hashes | P0 | R1 | None | Identity | Not implemented | Test | AC-006 | None | None |
| FR-014 | Reviewer can move events among New, Acknowledged, Resolved, and False Alert, with notes; reopen by customer administrator is audited | P0 | R1 | FR-009, FR-013 | Events | Not implemented | Test, Demonstration | AC-005, AC-006 | None | None |
| FR-015 | Retention job deletes expired evidence bytes and metadata, retries failed deletes, and reconciles orphans | P0 | R1 | FR-022 | Evidence, Jobs | Not implemented | Test | AC-009 | None | None |
| FR-016 | API and worker liveness and readiness exist and do not reveal secrets, tenant names, or camera URLs | P0 | R1 | None | API, Worker | Not implemented | Test, Inspection | AC-012 | None | None |
| FR-017 | Camera targets allow only RTSP or RTSPS to allowlisted addresses after DNS resolution; link-local, loopback, and metadata addresses are denied | P0 | R1 | FR-001 | Cameras, Worker | Not implemented | Test | AC-008 | None | None |
| FR-018 | Stream, inference, database, or evidence failure is shown as degraded or error and is not displayed as successful monitoring | P0 | R1 | FR-011 | Worker, Web | Not implemented | Test, Demonstration | AC-010 | None | None |
| FR-019 | Administrator can draw a zone on an authorized camera still in a desktop browser; the saved polygon is the one evaluated | P0 | R1 | FR-005, FR-032 | Web, Zones | Not implemented | Demonstration, Test | AC-016 | None | None |
| FR-020 | Each camera has a sample rate and an in-flight frame cap; overflow drops frames, counts them, and continues | P0 | R1 | FR-003 | Worker | Not implemented | Test | AC-011 | None | None |
| FR-021 | A fault in one camera process does not stop other cameras on the same supervisor | P0 | R1 | FR-003 | Worker | Not implemented | Test | AC-018 | None | None |
| FR-022 | A permitted snapshot is stored for an event; if capture fails, the event remains with evidence status failed | P0 | R1 | FR-009 | Evidence, Worker | Not implemented | Test | AC-005 | None | None |
| FR-023 | Camera passwords are write-only, encrypted at rest, and absent from events and API responses | P0 | R1 | SEC-005 | Cameras | Not implemented | Test | AC-008 | None | None |
| FR-024 | Dashboard includes health, events, evidence review, users, and settings, with loading, empty, error, offline, and degraded states | P0 | R1 | FR-010, FR-011 | Web | Not implemented | Demonstration | AC-013 | None | None |
| FR-025 | Health and event review work on desktop and mobile widths; zone drawing is a desktop workflow when unavailable on small screens | P0 | R1 | FR-024 | Web | Not implemented | Demonstration | AC-013 | None | None |
| FR-026 | Zone or rule save creates a new version; existing events keep the version that fired | P0 | R1 | FR-005, FR-009 | Zones, Events | Not implemented | Test | AC-003 | None | None |
| FR-027 | Cameras, zones, events, evidence, and memberships belong to an organization; cameras and events belong to a site | P0 | R1 | None | Tenancy | Not implemented | Test, Inspection | AC-007 | None | None |
| FR-028 | The server rejects access when the caller supplies another organization's object id | P0 | R1 | FR-013, FR-027 | Identity, Tenancy | Not implemented | Test | AC-007 | None | None |
| FR-029 | Last-frame time and processing delay come from worker timestamps | P0 | R1 | FR-002 | Worker, Web | Not implemented | Test | AC-002 | None | None |
| FR-030 | Customer API is `/api/v1` and errors return a stable code, safe message, and request id | P0 | R1 | FR-013 | API | Not implemented | Test, Inspection | AC-006 | None | None |
| FR-031 | Enabling a camera checks a server-side local entitlement for the maximum enabled cameras | P0 | R1 | FR-001 | Licensing | Not implemented | Test | AC-014 | None | None |
| FR-032 | Zone-editor reference image is confidential, authorized on read, and replaced or deleted by policy | P0 | R1 | FR-019, SEC-008 | Evidence, Web | Not implemented | Test | AC-008, AC-016 | None | None |
| FR-033 | No default vendor account can read evidence; support access is a time-bounded audited grant | P0 | R1 | FR-012, FR-028 | Tenancy | Not implemented | Test, Inspection | AC-007, AC-017 | None | None |
| FR-034 | Camera errors use categories such as authentication, network, decode, or configuration, without passwords or raw URLs | P0 | R1 | FR-011 | Worker, Web | Not implemented | Test | AC-002 | None | None |
| FR-040 | Platform administrator can create and suspend organizations without receiving evidence access | P0 | R1.1 | FR-033 | Tenancy | Not implemented | Test | AC-014 | None | None |
| FR-041 | Customer administrator can manage multiple sites and place cameras in a site | P0 | R1.1 | FR-027 | Tenancy | Not implemented | Test | AC-007 | None | None |
| FR-042 | Customer administrator can grant and revoke a time-bounded partner permission; the partner cannot self-grant another customer | P0 | R1.1 | FR-028 | Tenancy | Not implemented | Test | AC-006, AC-007 | None | None |
| FR-043 | Plan limits on cameras and features are enforced on the server | P0 | R1.1 | FR-031 | Licensing | Not implemented | Test | AC-014 | None | None |
| FR-044 | License activation, renewal, expiry, suspension, transfer, and offline behavior are documented; expiry does not delete evidence | P0 | R1.1 | FR-043 | Licensing | Not implemented | Test, Inspection | AC-014 | None | None |
| FR-045 | Notification rules use one approved channel and record delivery status; failure is not shown as delivered | P0 | R1.1 | FR-009 | Notifications | Not implemented | Test | AC-010 | None | None |
| FR-046 | Metadata export is limited to events the user can read, is audited, and excludes evidence files unless a separate permission is on | P1 | R1.1 | FR-010, FR-012 | Events, Audit | Not implemented | Test | AC-006 | None | None |
| FR-047 | Onboarding checklist can be followed without a shared customer password | P1 | R1.1 | FR-042 | Tenancy, Ops | Not implemented | Demonstration, Inspection | AC-012 | None | None |
| FR-048 | Optional short clips, if enabled after privacy review, preserve the event when capture fails | P2 | R1.1 | FR-022, OD-010 | Evidence | Not implemented | Test | AC-005 | None | None |
| FR-049 | PWA, if approved, documents offline, push, and background limits and does not offer offline live video | P2 | R1.1 | FR-025 | Web | Not implemented | Demonstration | AC-013 | None | None |
| FR-050 | Authorized historical recordings can be imported and indexed by organization, site, camera, and time | P0 | R2 | FR-027 | Footage archive | Not implemented | Test | AC-007 | None | None |
| FR-051 | Reviewers can label correct, false, missed, and ambiguous cases with reviewer and time | P0 | R2 | FR-050 | Dataset | Not implemented | Test | AC-015 | None | None |
| FR-052 | Datasets are versioned, carry provenance, and separate training, validation, and test incidents | P0 | R2 | FR-051 | Dataset | Not implemented | Test, Inspection | AC-015 | None | None |
| FR-053 | Training runs in isolated workers; the registry stores dataset, model, code, configuration, and metrics | P0 | R2 | FR-052 | Training | Not implemented | Inspection, Test | AC-015 | None | None |
| FR-054 | Model deployment requires comparison, explicit approval, rollback, and audit | P0 | R2 | FR-053, AI-010 | Model registry | Not implemented | Demonstration, Inspection | AC-015 | None | None |
| FR-055 | Anomaly detection is limited to a named use case, with a score separate from rule events, a threshold, and human review | P1 | R2 | FR-014 | Rules, Inference | Not implemented | Test, Inspection | AC-020 | None | None |
| FR-056 | Natural-language search returns only permitted stored events and evidence, with source, time, and camera, and states when it cannot answer | P1 | R2 | FR-010, FR-028 | Search | Not implemented | Test | AC-007 | None | None |
| FR-060 | Edge inference uses the same event model as the API | P1 | R3 | FR-004 | Worker, Deploy | Not implemented | Demonstration | AC-012 | None | None |
| FR-061 | Offline synchronization queues events and shows conflicts instead of dropping them silently | P1 | R3 | FR-009 | Sync | Not implemented | Test | AC-010 | None | None |
| FR-062 | Enterprise identity provider can authenticate users while application roles stay in this product | P1 | R3 | FR-013 | Identity | Not implemented | Test | AC-006 | None | None |
| FR-063 | Native mobile clients are built only after a written PWA gap, and they use `/api/v1` | P2 | R3 | FR-030 | Mobile | Not implemented | Inspection | AC-006 | None | None |
| FR-064 | Enterprise upgrade and recovery meet a contracted, rehearsed objective | P1 | R3 | NFR-007, NFR-020 | Ops | Not implemented | Demonstration | AC-012 | None | None |

## Non-functional

| ID | Description | Priority | Release | Dependencies | Component | Implementation | Verification | Acceptance | Test reference | Evidence |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| NFR-001 | Measure five cameras at 720p and 10 FPS input on named hardware; the profile is a test, not a warranty | P0 | R1 | AI-004 | Worker, Eval | Not implemented | Analysis | AC-011 | None | None |
| NFR-002 | Measure p95 event-commit latency; under two seconds is a provisional target, not a promise | P0 | R1 | NFR-001 | Worker, Eval | Not implemented | Analysis | AC-011 | None | None |
| NFR-003 | Supervisor restarts the affected camera after a recoverable failure and shows the downtime; no uptime percentage is committed | P0 | R1 | FR-018, FR-021 | Worker | Not implemented | Test | AC-010 | None | None |
| NFR-004 | Site acceptance of detection quality uses a labelled report of precision, recall, misses, and false alerts | P0 | R1 | AI-005 | Eval | Not implemented | Analysis | AC-011 | None | None |
| NFR-005 | Modular code, pinned dependencies, formatting, lint, and automated tests for new behavior | P0 | R1 | None | Repo | Not implemented | Inspection | AC-012 | None | None |
| NFR-006 | Structured logs with request and camera ids; operators can see status, frame age, reconnects, and drops | P0 | R1 | FR-016 | Observability | Not implemented | Inspection, Test | AC-010 | None | None |
| NFR-007 | Backup and restore of database and evidence are documented and executed once on a disposable environment | P0 | R1 | None | Ops | Not implemented | Demonstration | AC-012 | None | None |
| NFR-008 | OS, Docker, codecs, and CPU or GPU mode are documented after test; other combinations stay untested | P0 | R1 | None | Deploy | Not implemented | Inspection | AC-011 | None | None |
| NFR-009 | A trained operator can find a camera, filter today's events, and mark an event without a shell | P0 | R1 | FR-024 | Web | Not implemented | Demonstration | AC-013 | None | None |
| NFR-010 | A clean machine reaches a running stack from the repository instructions and Compose files | P0 | R1 | None | Deploy | Not implemented | Demonstration | AC-012 | None | None |
| NFR-011 | Schema changes are versioned migrations | P0 | R1 | None | Database | Not implemented | Inspection | AC-012 | None | None |
| NFR-012 | Queues, camera counts, and container resources are bounded; overload is visible | P0 | R1 | FR-020 | Worker, Deploy | Not implemented | Test | AC-010, AC-011 | None | None |
| NFR-013 | `/api/v1` supports future clients for operator tasks | P0 | R1 | FR-030 | API | Not implemented | Inspection | AC-006 | None | None |
| NFR-014 | Monitoring layouts work at the named mobile and desktop widths | P0 | R1 | FR-025 | Web | Not implemented | Demonstration | AC-013 | None | None |
| NFR-015 | Environment-specific values come from configuration | P0 | R1 | None | Deploy | Not implemented | Inspection | AC-012 | None | None |
| NFR-016 | Timestamps are stored in UTC and shown in the site timezone | P0 | R1 | None | API, Web | Not implemented | Test | AC-003 | None | None |
| NFR-017 | Pilot UI language is English | P1 | R1 | None | Web | Not implemented | Inspection | AC-013 | None | None |
| NFR-018 | Runbook steps match the software that was built | P0 | R1 | NFR-010 | Ops | Not implemented | Inspection | AC-012 | None | None |
| NFR-019 | Event metadata stays queryable when snapshot storage fails | P0 | R1 | FR-022 | Events, Evidence | Not implemented | Test | AC-005 | None | None |
| NFR-020 | Upgrade and rollback are rehearsed with pinned images and a database backup | P0 | R1 | NFR-007, NFR-011 | Ops | Not implemented | Demonstration | AC-012 | None | None |

## Security

| ID | Description | Priority | Release | Dependencies | Component | Implementation | Verification | Acceptance | Test reference | Evidence |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SEC-001 | Hashed passwords, expiring revocable sessions, logout, and no default shipped password | P0 | R1 | FR-013 | Identity | Not implemented | Test | AC-006 | None | None |
| SEC-002 | Server-side authorization on every protected read and write | P0 | R1 | FR-013 | Identity | Not implemented | Test | AC-006 | None | None |
| SEC-003 | Tenant isolation in queries, automated cross-tenant tests, and row-level security if OD-014 stands | P0 | R1 | FR-028 | Tenancy, Database | Not implemented | Test | AC-007 | None | None |
| SEC-004 | Secrets come from the environment or a secret file; the example file has placeholders | P0 | R1 | None | Deploy | Not implemented | Inspection | AC-008 | None | None |
| SEC-005 | RTSP credentials encrypted at rest and excluded from logs, API bodies, and browser storage | P0 | R1 | FR-023 | Cameras | Not implemented | Test | AC-008 | None | None |
| SEC-006 | Camera configuration cannot request arbitrary URLs, files, or non-camera networks | P0 | R1 | FR-017 | Cameras, Worker | Not implemented | Test | AC-008 | None | None |
| SEC-007 | Non-loopback deployments use TLS | P0 | R1 | NFR-010 | Deploy | Not implemented | Inspection | AC-012 | None | None |
| SEC-008 | Evidence bytes require authorization and are outside the public web root | P0 | R1 | FR-022 | Evidence | Not implemented | Test | AC-008 | None | None |
| SEC-009 | API validates input and rejects fields that would let a client set role or organization | P0 | R1 | FR-030 | API | Not implemented | Test | AC-006 | None | None |
| SEC-010 | Authentication attempts are rate limited without a permanent lockout of the only administrator | P0 | R1 | SEC-001 | Identity | Not implemented | Test | AC-006 | None | None |
| SEC-011 | Dependencies and images are pinned and scanned before a pilot build | P0 | R1 | NFR-005 | CI | Not implemented | Inspection | AC-012 | None | None |
| SEC-012 | Audit rows are append-only for the application role and readable only by authorized roles | P0 | R1 | FR-012 | Audit, Database | Not implemented | Test | AC-017 | None | None |
| SEC-013 | Backups are protected at the same sensitivity as database and evidence | P0 | R1 | NFR-007 | Ops | Not implemented | Inspection | AC-012 | None | None |
| SEC-014 | Containers run as non-root, with resource limits, and without the Docker socket | P0 | R1 | NFR-010 | Deploy | Not implemented | Inspection | AC-012 | None | None |
| SEC-015 | Runbook includes initial steps for credential compromise, cross-tenant access, and disk loss | P0 | R1 | NFR-018 | Ops | Not implemented | Inspection | AC-010 | None | None |
| SEC-016 | Cookie mutations are protected against CSRF; session cookies are HttpOnly | P0 | R1 | SEC-001 | Identity, Web | Not implemented | Test | AC-006 | None | None |
| SEC-017 | Worker database role cannot change memberships or read administrative secrets it does not need | P0 | R1 | FR-013 | Database | Not implemented | Inspection, Test | AC-008 | None | None |
| SEC-018 | Support access matches FR-033 | P0 | R1 | FR-033 | Tenancy | Not implemented | Test | AC-007 | None | None |
| SEC-019 | Partner access ends at grant expiry and cannot reach a customer who did not grant it | P0 | R1.1 | FR-042 | Tenancy | Not implemented | Test | AC-007 | None | None |
| SEC-020 | Release 1 has no face enrollment, emotion estimation, or real-world identity link | P0 | R1 | PRIV-002 | Inference, Web | Not implemented | Inspection, Test | AC-019 | None | None |
| SEC-021 | If OD-007 is accepted, administrators enroll a second factor before changing users or cameras | P0 | R1 | SEC-001 | Identity | Not implemented | Test | AC-006 | None | None |
| SEC-022 | Proxy sets baseline browser security headers | P0 | R1 | SEC-007 | Deploy | Not implemented | Inspection | AC-012 | None | None |
| SEC-023 | PostgreSQL is not published on the host; only the proxy port is published | P0 | R1 | NFR-010 | Deploy | Not implemented | Inspection | AC-012 | None | None |
| SEC-024 | Camera limits are enforced on the server; on-prem database owners can still edit rows, and the guide says so | P0 | R1.1 | FR-043 | Licensing | Not implemented | Test, Inspection | AC-014 | None | None |
| SEC-025 | Tests fail if a camera-connection log contains a password or userinfo | P0 | R1 | SEC-005 | Worker, Test | Not implemented | Test | AC-008 | None | None |
| SEC-026 | Dependency-advisory review cadence is documented without an unstaffed response-time claim | P1 | R1 | SEC-011 | Ops | Not implemented | Inspection | AC-012 | None | None |

## Privacy

| ID | Description | Priority | Release | Dependencies | Component | Implementation | Verification | Acceptance | Test reference | Evidence |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| PRIV-001 | Pilot agreement records that the customer asserts authority for each site's footage | P0 | R1 | None | Commercial | Not implemented | Inspection | AC-012 | None | None |
| PRIV-002 | No facial recognition, biometric identity matching, or emotion recognition | P0 | R1 | SEC-020 | Inference | Not implemented | Inspection | AC-019 | None | None |
| PRIV-003 | Persistent images are event snapshots and one current reference frame per camera, unless clips are later approved | P0 | R1 | FR-022 | Evidence | Not implemented | Inspection | AC-005 | None | None |
| PRIV-004 | Retention periods for evidence, events, audit, and logs are the periods approved under OD-008 | P0 | R1 | FR-015 | Evidence, Jobs | Not implemented | Test, Inspection | AC-009 | None | None |
| PRIV-005 | Organization offboarding is an audited purge; deletion is by time, event, camera, site, or organization | P0 | R1 | FR-015 | Evidence, Tenancy | Not implemented | Test | AC-009 | None | None |
| PRIV-006 | Evidence reads are authorized and audited | P0 | R1 | FR-012, SEC-008 | Evidence, Audit | Not implemented | Test | AC-008, AC-017 | None | None |
| PRIV-007 | Data classes in the data-governance document are followed | P0 | R1 | SEC-004 | Ops | Not implemented | Inspection | AC-008 | None | None |
| PRIV-008 | Retention and purge affect only the named organization | P0 | R1 | FR-027, FR-015 | Jobs | Not implemented | Test | AC-007, AC-009 | None | None |
| PRIV-009 | Training copies require a recorded per-organization permission | P0 | R2 | FR-052 | Dataset | Not implemented | Inspection | AC-015 | None | None |
| PRIV-010 | The system does not contact emergency services or actuate physical equipment | P0 | R1 | None | Product | Not implemented | Inspection | AC-019 | None | None |
| PRIV-011 | Events are presented for human review before consequential decisions | P0 | R1 | FR-014 | Web | Not implemented | Inspection | AC-020 | None | None |
| PRIV-012 | No legal-compliance claim until counsel's review is recorded | P0 | R1 | None | Docs | Not implemented | Inspection | AC-020 | None | None |
| PRIV-013 | Counsel records processing roles before any vendor-hosted pilot | P0 | R1 | OD-012 | Commercial | Not implemented | Inspection | AC-012 | None | None |
| PRIV-014 | Default deployment does not transfer footage to a vendor cloud or another country | P0 | R1 | OD-002 | Deploy | Not implemented | Inspection | AC-012 | None | None |
| PRIV-015 | Customer materials state that notices and lawful placement are the customer's duty | P0 | R1 | PRIV-001 | Docs, Web | Not implemented | Inspection | AC-020 | None | None |
| PRIV-016 | Backup lifetime is part of the retention decision | P0 | R1 | PRIV-004, SEC-013 | Ops | Not implemented | Inspection | AC-009 | None | None |
| PRIV-017 | TLS and camera-password encryption are on; disk encryption is recorded or its absence is stated in the agreement | P0 | R1 | SEC-007, SEC-005 | Deploy | Not implemented | Inspection | AC-008 | None | None |
| PRIV-018 | Documentation states the product cannot selectively erase one person | P0 | R1 | PRIV-002, PRIV-005 | Docs | Not implemented | Inspection | AC-019 | None | None |

## AI lifecycle

| ID | Description | Priority | Release | Dependencies | Component | Implementation | Verification | Acceptance | Test reference | Evidence |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AI-001 | The rule engine calls person detection only through the adapter | P0 | R1 | FR-004 | Inference, Rules | Not implemented | Test, Inspection | AC-003 | None | None |
| AI-002 | Events store model version and inference configuration | P0 | R1 | FR-004, FR-009 | Events | Not implemented | Test | AC-015 | None | None |
| AI-003 | Measured results are stated only when an evaluation report exists for that build and site | P0 | R1 | NFR-004 | Docs, Eval | Not implemented | Inspection | AC-011, AC-020 | None | None |
| AI-004 | CPU, memory, and GPU use are measured before any capacity sentence is used with a customer | P0 | R1 | NFR-001 | Eval | Not implemented | Analysis | AC-011 | None | None |
| AI-005 | Precision, recall, false alerts per camera-hour, misses, latency, and dropped frames can be reported when labels exist | P0 | R1 | NFR-004 | Eval | Not implemented | Analysis | AC-011 | None | None |
| AI-006 | Tracking limits, including occlusion and ID switches, are documented for the shipped tracker | P0 | R1 | FR-006 | Tracking | Not implemented | Inspection | AC-004 | None | None |
| AI-007 | The Release 1 detector is configured for the person class only | P0 | R1 | SEC-020 | Inference | Not implemented | Test | AC-019 | None | None |
| AI-008 | Numeric quality gates are set by the owner after a baseline, not in this draft | P0 | R1 | OD-018 | Eval | Not implemented | Inspection | AC-011 | None | None |
| AI-009 | The labelled set and match window are frozen before confirmation numbers are computed | P0 | R1 | AI-005 | Eval | Not implemented | Inspection | AC-011 | None | None |
| AI-010 | Release 2 promotion requires comparison, approval, rollback, and a split-leakage check | P0 | R2 | FR-054 | Model registry | Not implemented | Inspection | AC-015 | None | None |
| AI-011 | Anomaly scores are stored and displayed separately from rule events | P0 | R2 | FR-055 | Events, Web | Not implemented | Inspection | AC-020 | None | None |
| AI-012 | Benchmarks record hardware, codec, model checksum, sample rate, and network path | P0 | R1 | AI-004 | Eval | Not implemented | Inspection | AC-011 | None | None |
| AI-013 | Model weights and inference libraries are added only after the commercial license is recorded | P0 | R1 | OD-004 | Inference | Not implemented | Inspection | AC-015 | None | None |
| AI-014 | Integration starts with two authorized cameras before the five-camera profile is judged | P0 | R1 | NFR-001 | Eval, Worker | Not implemented | Demonstration | AC-001, AC-011 | None | None |
| AI-015 | Worker counts frames received, sampled, inferred, and dropped per camera | P0 | R1 | FR-020 | Worker | Not implemented | Test | AC-011 | None | None |
| AI-016 | Pilot review uses false-alert marks and camera-down time; the product does not retrain from them | P1 | R1 | FR-014 | Eval | Not implemented | Inspection | AC-011 | None | None |

## Acceptance criteria

| ID | Description | Priority | Release | Dependencies | Component | Implementation | Verification | Acceptance | Test reference | Evidence |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| AC-001 | Two authorized RTSP cameras reach truthful health without credentials in responses or logs | P0 | R1 | FR-001, FR-003, SEC-005 | Worker, Cameras | Not run | Demonstration, Test | AC-001 | None | None |
| AC-002 | One camera disconnects and recovers visibly without stopping the documented backoff behavior or hiding delay | P0 | R1 | FR-003, FR-011, FR-029 | Worker | Not run | Demonstration, Test | AC-002 | None | None |
| AC-003 | A qualifying labelled entry creates an event with organization, site, camera, zone, rule version, model version, and server time | P0 | R1 | FR-007, FR-009 | Rules, Events | Not run | Analysis, Test | AC-003 | None | None |
| AC-004 | Continued presence and restart-during-cooldown follow the written duplicate rules | P0 | R1 | FR-008 | Rules | Not run | Test | AC-004 | None | None |
| AC-005 | Authorized evidence loads, unauthorized evidence is denied, and a failed snapshot keeps the event | P0 | R1 | FR-014, FR-022 | Evidence, Events | Not run | Test, Demonstration | AC-005 | None | None |
| AC-006 | Unauthenticated and wrong-role API calls are denied for the Release 1 role matrix | P0 | R1 | FR-013, SEC-002 | Identity | Not run | Test | AC-006 | None | None |
| AC-007 | Organization A cannot read or change organization B's objects by substituting identifiers | P0 | R1 | FR-028, SEC-003 | Tenancy | Not run | Test | AC-007 | None | None |
| AC-008 | Passwords, session secrets, and evidence bytes meet SEC-005 and SEC-008, including log redaction | P0 | R1 | SEC-005, SEC-008, SEC-025 | Cameras, Evidence | Not run | Test | AC-008 | None | None |
| AC-009 | Expired evidence bytes and metadata are deleted; a test organization purge removes secrets and evidence | P0 | R1 | FR-015, PRIV-005 | Evidence, Jobs | Not run | Test | AC-009 | None | None |
| AC-010 | Child crash, database interruption, and full evidence volume produce the documented visible failures and recovery | P0 | R1 | FR-018, NFR-003 | Worker, Ops | Not run | Demonstration, Test | AC-010 | None | None |
| AC-011 | Benchmark sheets for two and five cameras are filled and claims do not exceed them; the two-second target is reported as met or unmet | P0 | R1 | NFR-001, NFR-002, AI-012 | Eval | Not run | Analysis | AC-011 | None | None |
| AC-012 | A clean install, backup, restore, upgrade, and rollback are each rehearsed and written up | P0 | R1 | NFR-007, NFR-010, NFR-020 | Ops | Not run | Demonstration | AC-012 | None | None |
| AC-013 | Health and event review work on desktop and mobile viewports, including empty and error screens; zone editing works on desktop | P0 | R1 | FR-024, FR-025 | Web | Not run | Demonstration | AC-013 | None | None |
| AC-014 | Camera entitlement is enforced in Release 1; Release 1.1 expiry and suspension match the documented behavior and do not delete data | P0 | R1, R1.1 | FR-031, FR-044 | Licensing | Not run | Test | AC-014 | None | None |
| AC-015 | Release 1 events store model version; Release 2 promotion requires approval and rollback changes the version used by new events | P0 | R1, R2 | AI-002, AI-010 | Events, Model registry | Not run | Test, Inspection | AC-015 | None | None |
| AC-016 | The polygon drawn in the editor is the polygon a synthetic entering track uses | P0 | R1 | FR-005, FR-019 | Zones, Rules, Web | Not run | Test, Demonstration | AC-016 | None | None |
| AC-017 | FR-012 actions create audit rows with no secrets or image bytes | P0 | R1 | FR-012 | Audit | Not run | Test | AC-017 | None | None |
| AC-018 | Crashing camera one's process leaves camera two's last-frame time advancing | P0 | R1 | FR-021 | Worker | Not run | Test | AC-018 | None | None |
| AC-019 | Release 1 UI and API contain no facial recognition, emotion recognition, identity enrollment, emergency dispatch, or door control | P0 | R1 | SEC-020, PRIV-002, PRIV-010 | Product | Not run | Inspection | AC-019 | None | None |
| AC-020 | Event view and docs state that detection can be wrong or missed and that a person must review it; they do not claim compliance or perfect detection | P0 | R1 | PRIV-011, PRIV-012 | Web, Docs | Not run | Inspection | AC-020 | None | None |

## Coverage check

Counts in this file, which must match [SRS.md](SRS.md):

| Prefix | IDs | Count |
| --- | --- | --- |
| FR | 001–034, 040–049, 050–056, 060–064 | 56 |
| NFR | 001–020 | 20 |
| SEC | 001–026 | 26 |
| PRIV | 001–018 | 18 |
| AI | 001–016 | 16 |
| AC | 001–020 | 20 |

Total rows: 156. All implementation and evidence cells are empty of results.
