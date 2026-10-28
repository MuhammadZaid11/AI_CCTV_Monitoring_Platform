# Product scope

**Status:** Draft 0.1, 27 October 2026. Not approved. Not implemented.

This file is the scope boundary for the AI CCTV Monitoring Platform. Requirements in [SRS.md](SRS.md) use the release names defined here. If this file and the SRS disagree, the newer approved version wins. Until approval, the SRS is the more detailed proposal and this file is the boundary statement.

## Product intent

Sell a maintainable monitoring product to CCTV installation companies, security service providers, warehouses, factories, offices, retail stores, commercial buildings, and similar customers who are allowed to provide the footage.

The product creates reviewable events. It does not guarantee detection, and it does not replace human security personnel.

Initial commercial focus is Pakistan. The same product design must not block a later market, but international sale requires a separate review of privacy law, tax, hosting location, support hours, and contract terms. See [commercial-plan.md](commercial-plan.md).

## Commercial motions the design must allow

These are product capabilities, not a price list.

| Motion | Release treatment |
| --- | --- |
| Subscription hosted by the vendor | Architecture must allow it. Do not make remote cloud ingest of RTSP the pilot default. Offer hosting only after residency, bandwidth, and support are accepted. |
| Customer-hosted installation | Primary pilot path. Same build as hosted, usually one organization. |
| Installer or partner hosts several customers on one server | Treated as multi-tenant, even though it is on the partner's hardware. Isolation requirements apply in full. |
| Installation and configuration fees | Commercial offer. No special product module in Release 1. |
| Camera-count licensing | Entitlement hook in Release 1. Enforcement, plans, and lifecycle in Release 1.1. |
| Site or organization plans | Organization and site exist in the Release 1 data model. Plan packaging is Release 1.1. |
| Installer and reseller partners | Data-model awareness in Release 1. Delegated access product in Release 1.1. |
| Enterprise deployment and support contracts | Release 3, sold only with an explicit operational scope. |

No prices are set. [commercial-plan.md](commercial-plan.md) describes how prices should be validated.

## Release 1 — Controlled pilot

Release 1 is a commercial MVP for a small number of authorized pilots. It is not a general-availability product and must not be labelled production-ready.

### In scope

- Authorized RTSP cameras: add, edit, disable, reconnect, remove.
- Connection settings, encrypted credentials, connection state, last-frame time, processing delay, and actionable errors.
- Bounded reconnect with backoff. One camera failing must not stop the others.
- Configurable frame sampling and resource limits.
- A replaceable person-detection adapter, tracking sufficient for rules, and model version recorded on each event.
- Restricted zones drawn on an authorized camera image.
- Person-entry and inward boundary-crossing rules, with geometry, tracking-loss behavior, cooldown, and duplicate suppression.
- Versioned rules.
- Events with time, camera, site, zone, rule, model version, and processing metadata.
- Review states: New, Acknowledged, Resolved, False Alert.
- Event history, filters, pagination, and detail.
- Evidence snapshots where permitted. A failed snapshot must not discard the event.
- Configurable retention and deletion.
- Authentication, server-side authorization, and tenant ownership.
- Roles used in the pilot: customer administrator, security operator, auditor, and a service identity for workers. Platform administrator exists for a hosted deployment and has no standing access to customer evidence.
- Audit trail for security-sensitive actions.
- Responsive web dashboard for desktop and mobile browsers. Zone drawing is a desktop workflow.
- Docker Compose deployment, versioned migrations, automated tests, structured logs, health endpoints, secret management, and documented backup, restore, upgrade, and rollback.
- Organization and site records so later multi-site and multi-customer work does not require a new core model. The pilot may use one organization and one default site.
- A local entitlement record checked when cameras are added, so limits are not a later retrofit. Billing and license lifecycle are not part of Release 1.

### Release 1 acceptance gates

All of the following must be true before a pilot is offered to a customer:

1. AC-001 through AC-013, AC-016 through AC-020 in [SRS.md](SRS.md) pass, or a named waiver is signed by the product owner.
2. AC-011 has a written benchmark report on named hardware. Meeting a numeric accuracy target is not required until the owner sets one after seeing that report.
3. Cross-tenant denial tests pass even if the pilot itself has one organization.
4. Credential and evidence tests pass.
5. Backup and restore have been rehearsed on a disposable environment.
6. Known limitations, including missed detections, are written in the pilot agreement.
7. No facial recognition, biometric identification, emotion recognition, or automatic emergency dispatch exists in the build.

Passing these gates still does not by itself justify the words "production-ready", "secure", or "compliant".

## Release 1.1 — Commercial operations

Build this only after Release 1 is approved as a separate phase.

- Multi-customer organization management.
- Multiple sites per organization in the administration UI.
- Installer and reseller organizations, with access delegated by the customer, time-bounded and audited.
- Server-side plan entitlements: camera limits and feature limits.
- License activation, renewal, expiry, suspension, transfer, and a documented offline behavior.
- Reports and export permissions.
- Notification rules, delivery tracking, and one approved channel chosen after cost and privacy review.
- Customer onboarding and support administration.
- Optional short evidence clips, if storage and privacy review accept them.
- Progressive Web App behavior only if pilot users show that they need it.

Release 1.1 gate: AC-014 passes, partner isolation tests pass, notification failure cannot create a false "delivered" state, and license expiry cannot be bypassed from the browser. Expiry must not delete customer data.

## Release 2 — Future AI

Not implemented and not part of the pilot.

- Import of authorized historical recordings, indexed by tenant, site, camera, and time.
- Authorized labelling: correct detection, false alert, missed detection, ambiguous.
- Versioned datasets, provenance, quality checks, and separation of training, validation, and test data.
- Isolated training workers and a model registry.
- Comparison against the current model, explicit approval, rollback, and an audit trail.
- Monitoring of model quality after deployment.
- Anomaly detection only for named use cases, with a score that is distinct from a deterministic rule, a threshold, and human review.
- Natural-language search over stored events and available evidence, inside the caller's permissions, with source event, time, and camera shown. The system must say when footage is not indexed or the question cannot be answered. It must not invent events.

Release 2 gate: AC-015, plus dataset leakage tests and a signed model-promotion record. No model is deployed because a notebook run looked better.

## Release 3 — Enterprise and advanced deployment

Planned separately. Not scheduled by this document.

- Edge inference on customer premises.
- Hybrid cloud: local processing with a remote control plane.
- Offline operation and controlled synchronization.
- Enterprise identity federation.
- Higher-grade monitoring, backup, recovery, and upgrade automation.
- Approved external integrations beyond the single notification channel.
- Native Android or iOS applications, only if a PWA cannot meet a demonstrated need.

## Explicitly out of scope

Unless a later approved requirement adds them:

- Facial recognition, face enrollment, identity matching, person re-identification across cameras, age estimation, gender estimation, or emotion recognition.
- License-plate recognition, audio surveillance, weapon detection, or fall detection.
- Automatic calls to police, medical, or fire services.
- Automatic control of doors, alarms, or other physical systems.
- Using the product as the only security control.
- Public-internet exposure of RTSP cameras.
- A consumer home-surveillance marketplace.
- Covert monitoring of people who were not disclosed under the customer's own legal duties. The product cannot make an unlawful deployment lawful.
- Native mobile apps in the pilot.
- Microservices as the initial architecture.
- Continuous recording of every camera as the default retention mode.
- Guaranteed accuracy, uptime, or detection of every intrusion.
- Invented prices, testimonials, or compliance badges.

## Mobile boundary

Release 1 is a responsive website. The HTTP API is versioned so a later mobile client can use the same authorization and event model.

A PWA is a Release 1.1 candidate, not a commitment. Native applications are Release 3 candidates.

Documented limits of a browser client: unreliable background execution, limited push notifications, incomplete offline access, restricted media autoplay, and weak polygon editing on small touch screens. Zone editing is therefore specified for desktop browsers. Monitoring and event review must work at mobile width.

## Relationship to the earlier SRS

`AI_CCTV_Monitoring_Agent_SRS_v1.0.docx` remains in `docs/`. It was a single-site, local-first draft. It did not define tenants, partners, licensing, or a pilot-versus-production boundary. Do not implement from that file alone. Do not delete it; it is the record of the previous proposal.

## Release governance

A feature moves from planned to implemented only when code exists, tests exist, and the traceability matrix cites the evidence. A release moves from draft to approved only when the product owner signs the approval block in [SRS.md](SRS.md). A pilot starts only after the Release 1 gates above. A public sale starts only after a later decision that names support, legal review, backup objectives, and accepted limitations.
