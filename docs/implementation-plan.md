# Implementation plan

**Status:** Proposed sequence, 27 October 2026. Not started. Do not write application code until the product owner approves this plan (OD-001).

Estimates are relative points for one engineer who already knows Python and React. One point is about a focused day. They are not a delivery date, a staffing plan, or a quotation. The team size is unknown. Risk can erase the points, especially around cameras and licenses.

## Phase 0 findings

Completed as documentation only. No product code was written.

| Fact | Evidence |
| --- | --- |
| The workspace had one file, the Word SRS draft | Directory listing of `D:\Ai` on 27 October 2026 |
| That draft is unapproved and has no tenancy or commercial licensing model | Text extracted from the docx |
| There is no git repository | `git status` reported that `D:\Ai` is not a repository |
| Python, Node.js, Docker, and FFmpeg are not installed on this workstation | Commands were not found on PATH |
| Nothing is tested | No test files exist |

The Word file was not modified or deleted.

Assumptions and open decisions are separated in [open-decisions.md](open-decisions.md).

## Phase 1 through 3

These phases are this documentation set:

- Requirements: [SRS.md](SRS.md), [product-scope.md](product-scope.md), [requirements-traceability.md](requirements-traceability.md).
- Architecture and security: [architecture.md](architecture.md), [threat-model.md](threat-model.md), [data-governance.md](data-governance.md).
- Evaluation, commercial, and operations intent: [ai-evaluation-plan.md](ai-evaluation-plan.md), [commercial-plan.md](commercial-plan.md), [operations-runbook.md](operations-runbook.md).

Phase 4 in the product brief (implementation) is the milestone list below. It is blocked on approval. Phase 5 (pilot readiness) is the last gate in this file and has not been entered.

## Definition of done for a milestone

- The milestone's tests exist and have been run. The command output is recorded in the milestone note, including failures.
- New behavior is reflected in the traceability matrix with a test name. Status stays "not verified" until the run is recorded.
- Authorization, tenant scope, and failure behavior for that milestone are included, not postponed as a batch.
- Logs from the new path do not contain secrets.
- Schema changes are migrations.
- The owner has not been told a feature works if the test failed.

## Milestone 0 — Scaffold

**Goal:** a runnable empty stack and a private git history.  
**Points:** 3. **Risk:** Low, after tools are installed.  
**Blocked by:** OD-001, OD-022, and installation of Git, Docker, Python, and Node.

Expected files:

- `backend/pyproject.toml`, package layout in `backend/src/cctv_platform/`
- `backend/src/cctv_platform/main.py` with live and ready endpoints
- `backend/src/cctv_platform/settings.py`
- `backend/alembic/` initial empty migration
- `frontend/` Vite React TypeScript shell
- `deploy/compose.yml`, `deploy/nginx/`, `.env.example`, `.gitignore`
- CI definition that lints and runs unit tests

Gate: Compose starts API, worker placeholder, proxy, and PostgreSQL. Ready fails when the database is stopped. The example env file has no real secrets. CI runs once.

## Milestone 1 — Identity, tenancy, audit

**Goal:** sign-in and a data model that cannot forget organization.  
**Points:** 8. **Risk:** High, because a shortcut here becomes a cross-tenant bug.

Expected modules: `modules/identity`, `modules/tenancy`, `modules/audit`, session middleware, RLS policies if OD-014 remains recommended, membership tables, support-grant table with no default row.

Tests: password hashing, login rate limit, logout revocation, role matrix denials, two-organization IDOR tests, audit rows, CSRF rejection.

Gate: AC-006 and AC-007 pass for the endpoints that exist. FR-033 is true by inspection: no vendor backdoor user.

## Milestone 2 — Cameras without video

**Goal:** configuration and secrets, still no decode.  
**Points:** 5. **Risk:** Medium.

Expected modules: `modules/cameras`, `modules/licensing` with a local entitlement row.

Tests: CRUD inside the organization, password not returned, encryption round-trip, SSRF cases from the threat model, entitlement rejection, audit of changes.

Gate: FR-001, FR-017, FR-023, FR-031, and SEC-025's log assertion for the connection-test stub.

## Milestone 3 — RTSP supervisor

**Goal:** two test streams show truthful health and isolation.  
**Points:** 13. **Risk:** High. Codecs and network behavior dominate.

Expected modules: `workers/supervisor.py`, `workers/camera_process.py`, PyAV decode behind an interface, lease columns, frame counters.

Tests: MediaMTX or an equivalent local RTSP fixture, backoff, child crash isolation, degraded state when frames stop, redaction of the fixture URL.

Gate: AC-001, AC-002, and AC-018 on the fixture. A real camera is not required for this gate and is required before pilot.

## Milestone 4 — Detector and tracker

**Goal:** a licensed adapter and a tracker, measured enough to know the cost.  
**Points:** 8. **Risk:** High, mostly license and speed.

Blocked by OD-004 and OD-005.

Expected modules: `modules/inference`, `modules/tracking`, benchmark sheet template filled for the development machine.

Tests: fake detector, person-class-only assertion, checksum recorded, counters for sampled and dropped frames. A smoke clip if the license allows it.

Gate: AI-013 recorded in the repo. AC-015's Release 1 half can be completed in Milestone 5 when events exist. No accuracy claim.

## Milestone 5 — Rules and events

**Goal:** polygons create versioned events and suppress duplicates.  
**Points:** 8. **Risk:** Medium.

Expected modules: `modules/zones`, `modules/rules`, `modules/events`.

Tests: synthetic tracks for entry, stay, exit, re-entry, cooldown across a restarted state, rule version immutability, tenant scope on events.

Gate: AC-003 on synthetic tracks, AC-004, AC-016's geometry half.

## Milestone 6 — Evidence and retention

**Goal:** snapshots and deletion.  
**Points:** 5. **Risk:** Medium.

Expected modules: `modules/evidence`, `workers/jobs.py`, filesystem store.

Tests: event survives a failed write, authorized download, cross-tenant download denied, expiry deletes bytes, orphan reconciliation.

Gate: AC-005 and AC-009.

## Milestone 7 — Dashboard

**Goal:** an operator can work in the browser.  
**Points:** 10. **Risk:** Medium.

Expected areas: sign-in, camera list and detail, desktop zone editor, event list and detail, users, settings, empty and error states, limitation text on the event.

Tests: Playwright journeys for AC-013 and AC-020. API tests remain the authorization gate.

Gate: AC-013, AC-016's drawing half, AC-020.

## Milestone 8 — Pilot hardening

**Goal:** rehearse the runbook and close the security checklist.  
**Points:** 8. **Risk:** Medium, or high if the first real cameras differ from the fixture.

Work: dependency scan, container review, backup and restore, upgrade and rollback, two real authorized cameras, overnight soak, evaluation report using [ai-evaluation-plan.md](ai-evaluation-plan.md), update traceability with evidence links, pilot agreement draft and customer responsibility list.

Gate: AC-010, AC-011, AC-012, AC-017, AC-019, and the Release 1 gates in [product-scope.md](product-scope.md).

## Later phases, not scheduled

| Phase | Starts when | First milestones |
| --- | --- | --- |
| Release 1.1 | Owner approves a commercial-operations phase after at least one pilot | Organizations UI, partner grants, license lifecycle, one notification channel |
| Release 2 | A customer need and a training permission exist | Import and labels before any training job |
| Release 3 | A named enterprise requirement | Edge and identity, each with its own design note |

## Suggested build order inside a week

The order is the milestone order. Do not start the zone editor before events exist in the API. Do not start the detector before the license decision. The dashboard can show camera health as soon as Milestone 3 is done, and gain events in Milestone 7.

## Top risks

| # | Risk | Area | Why it is serious | Mitigation in this plan |
| --- | --- | --- | --- | --- |
| 1 | Model or FFmpeg license blocks commercial distribution | Business, legal, technical | AGPL or GPL components can conflict with a closed-source product | OD-004 and OD-005 before Milestone 4. No weights committed earlier |
| 2 | Real RTSP cameras do not behave like the test fixture | Technical | The pilot fails on site even if CI is green | Milestone 3 fixture, then real cameras in Milestone 8 before any customer |
| 3 | Cross-tenant data leak | Security | An installer-hosted server with two customers becomes a breach | Organization column, tests, and RLS in Milestone 1, before cameras |
| 4 | Camera passwords or snapshots exposed | Security, privacy | Reused camera passwords and images of people | Write-only secrets, redaction tests, private files, audited downloads |
| 5 | Missed people and false alerts | AI, business | The buyer thinks the area is protected, or they stop trusting every alert | Evaluation plan, human review, limitation text, no accuracy marketing |
| 6 | The provisional five-camera profile does not fit the hardware | Technical, commercial | A capacity claim would be false | Measure before speaking. Sell the tested configuration only |
| 7 | Live video sent across the public internet by default | Security, privacy, operations | Exposure and fragile links | On-prem or private-link pilot. OD-002 |
| 8 | Unreviewed lawful basis for CCTV | Privacy, business | A pilot on someone else's staff or visitors without notice | Counsel questions, written authorization, no external pilot before that |
| 9 | Support and upgrades overwhelm a small team | Business, operations | On-prem software fails quietly after the installer leaves | Runbook rehearsal, no 24/7 promise, camera limits, visible health |
| 10 | Operators treat a green dashboard as proof | Safety, commercial | A quiet screen during an outage or a missed detection | Health definitions, gap visibility, AC-020 wording, contract limitations |

Further risks that did not make this list: no git history yet, this workstation has no toolchain, backups that sit on the same disk, and a partner grant that nobody revokes. They are real and are covered by OD-022, Milestone 0, OD-020, and FR-042.

## Traceability maintenance

When a milestone lands, update the rows it implements: status, test reference, and a pointer to the command output or report. Do not mark a row verified because the file exists.

## Approval to start

The next engineering action after approval is Milestone 0 only. Milestone 1 does not start in the same change set as the scaffold unless the owner asks for that.
