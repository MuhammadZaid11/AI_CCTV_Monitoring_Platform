# AI CCTV Monitoring Platform

Commercial software for authorized CCTV monitoring. The platform is intended to receive customer-authorized camera streams, detect and track people, evaluate configurable security rules, store reviewable events and permitted evidence, and help authorized operators investigate those events.

This repository is the start of that product. It is not a finished system.

## Current status

| Area | Status |
| --- | --- |
| Application code | Not started |
| Database, containers, CI, tests | Not started |
| Requirements and architecture | Draft 0.1, dated 27 October 2026, **not approved** |
| Production readiness, security certification, detection accuracy, legal compliance | **Not established.** Nothing in this repository is evidence of those claims. |

The platform is meant to reduce continuous manual watching. It is not a replacement for security staff, and it must not be described as detecting every incident.

A human must review events before any consequential security decision. The product must not call emergency services, unlock doors, or identify people by face, biometric match, or emotion.

## What is in this repository

Verified by inspection on 27 October 2026:

- `docs/AI_CCTV_Monitoring_Agent_SRS_v1.0.docx` — an earlier draft SRS. It is preserved and is not an approved baseline.
- The markdown files listed below — the proposed commercial baseline, written after that inspection.
- No git repository, source code, dependency manifests, tests, Docker files, or CI configuration.

The earlier Word SRS assumed a local Windows deployment and named specific libraries (FastAPI, OpenCV, YOLO, React, PostgreSQL) as if they were already selected. This documentation set keeps the useful product intent from that draft and replaces the unpublished commercial, tenancy, licensing, and release model. Technology choices here are recommendations, not installed components. See [docs/open-decisions.md](docs/open-decisions.md).

## Planned and not built

Release 1 is a controlled pilot: cameras, person detection and tracking, restricted-zone rules, reviewable events, evidence snapshots, authentication and tenant isolation, a responsive web dashboard, and operable deployment.

Release 1.1 is commercial operations: organizations, sites, installer partners, licensing, reports, and notifications.

Release 2 is future AI: labelled footage, model promotion, specific anomaly detection, and natural-language search over real stored events.

Release 3 is enterprise deployment: edge and hybrid operation, offline sync, enterprise identity, and native mobile apps only if customers justify them.

Details and exclusions are in [docs/product-scope.md](docs/product-scope.md).

## Documentation

Read in this order:

1. [docs/product-scope.md](docs/product-scope.md) — what each release includes and what gates it.
2. [docs/SRS.md](docs/SRS.md) — requirements, acceptance criteria, and approvals.
3. [docs/requirements-traceability.md](docs/requirements-traceability.md) — requirement IDs, tests, and evidence. All evidence cells are empty because nothing is implemented.
4. [docs/architecture.md](docs/architecture.md) — recommended modular monolith, data flow, and deployment.
5. [docs/threat-model.md](docs/threat-model.md) — assets, threats, mitigations, and planned security tests.
6. [docs/data-governance.md](docs/data-governance.md) — classification, retention, deletion, and tenant isolation.
7. [docs/ai-evaluation-plan.md](docs/ai-evaluation-plan.md) — how detection quality will be measured before any accuracy claim.
8. [docs/commercial-plan.md](docs/commercial-plan.md) — customers, channel, pilots, and pricing validation. No prices.
9. [docs/implementation-plan.md](docs/implementation-plan.md) — build order after approval. Implementation has not started.
10. [docs/operations-runbook.md](docs/operations-runbook.md) — intended operations. These procedures are not executable yet.
11. [docs/open-decisions.md](docs/open-decisions.md) — decisions that still need an owner.

## Development prerequisites

Not installed on the workstation inspected on 27 October 2026: Git, Python, Node.js, Docker, and FFmpeg. None of these were required to write the documents. They will be required before Milestone 0 in the implementation plan, and versions should be pinned at that time rather than assumed from this README.

When implementation is approved, development should use a clean environment, versioned dependencies, and no real camera passwords in the repository.

## How to approve the next step

Implementation of the camera engine, model integration, API, or frontend should not start until the product owner reviews this set and explicitly approves the implementation plan.

Approving the plan can accept the recommended defaults in [docs/open-decisions.md](docs/open-decisions.md), or it can change specific decisions first. Approval of the documents does not mean the product is ready to sell.

## Non-claims

This repository does not show completed tests, customer pilots, benchmark numbers, certifications, or a security assessment. Those may be added only when they have actually been run, and the results must be stored as evidence rather than written in advance.
