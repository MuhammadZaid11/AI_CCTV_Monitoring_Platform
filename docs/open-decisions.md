# Open decisions

**Status:** Open as of 27 October 2026. Recommendations are proposals, not owner decisions.

Implementation should not start until OD-001 is resolved by an explicit approval of the implementation plan. Other rows may be accepted at their recommended default in that same approval, or changed first.

| ID | Decision | Why it matters | Options | Recommendation | Owner | Needed before | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| OD-001 | Adopt this documentation set as the working baseline | Two scope documents now exist. Engineering needs one authority. | A. Approve this markdown set and keep the Word SRS as history. B. Revise selected sections first. C. Continue from the Word SRS only. | A. The Word file has no tenancy, licensing, or pilot gates. | Product owner | Any implementation | Open |
| OD-002 | Where the first pilot runs | Camera networks should stay local. Public RTSP to a cloud is a security and bandwidth risk. | A. Server on the customer LAN. B. Installer NOC with a private link to the site. C. Vendor-hosted cloud ingest. | A for the first site. B only with a private link and multi-tenant controls if more than one customer shares the server. C is deferred. | Product owner | Pilot deployment design | Open |
| OD-003 | Pilot server operating system | Media decoding, GPU drivers, and file permissions differ by OS. | A. Linux server. B. Windows Server or Windows desktop with Docker Desktop. | A for any pilot that is left running. Windows is acceptable for development only after Docker is installed and tested. | Engineering | Milestone 0 | Open |
| OD-004 | Detection model and license | Ultralytics YOLO code is AGPL-3.0. Weight licenses are separate. Using the wrong license can block a closed-source sale or force source disclosure. | A. ONNX Runtime plus a model with a commercial-use weight license. B. Buy a commercial Ultralytics license and record its terms. C. Defer model choice and block the inference milestone. | A, with written license review before weights are added to the product. Do not vendor AGPL code by default. | Product owner and engineering | Milestone 4 | Open |
| OD-005 | FFmpeg and codec policy | A GPL build of FFmpeg, or a GPL-linked encoder, is a distribution risk. H.265 also has patent exposure. | A. LGPL FFmpeg build, H.264 decode, JPEG snapshots, no GPL libraries. B. Unrestricted FFmpeg. | A. Record the exact build and codecs after a compatibility test. | Engineering | Milestone 3 | Open |
| OD-006 | First design-partner segment | Warehouses, retail, factories, offices, and installers do not share the same zones, lighting, or buying process. | Pick one primary segment for the first two pilots. | CCTV installers as the channel, with one warehouse or factory site as the first end customer, because restricted zones and after-hours entry are easier to define than crowded retail floors. | Product owner | Pilot recruiting | Open |
| OD-007 | MFA in Release 1 | A stolen administrator password exposes cameras and evidence. | A. TOTP required for customer administrators in Release 1. B. Password-only in the pilot, MFA in 1.1. | A for customer administrators and any platform administrator. Operators may use passwords in the pilot if the account cannot change cameras or users. | Product owner | Milestone 1 | Open |
| OD-008 | Retention defaults | Storage cost, investigations, and privacy pull in opposite directions. | Owner sets days for snapshots, event rows, audit rows, and logs. | Proposed starting point only: snapshots 30 days, event metadata 180 days, audit 365 days, application logs 30 days. Not approved. | Product owner, with legal advice | Before pilot data is stored | Open |
| OD-009 | Zone cooldown default | Too short creates alert floods after a tracker restart. Too long hides a second entry. | Configurable. Default 0, 30, or 60 seconds, plus track-level suppression. | Track-level one-alert-per-entry, plus a proposed 30 second zone cooldown. Both configurable per rule. | Engineering, confirmed by pilot user | Milestone 5 | Open |
| OD-010 | Evidence clips in the pilot | Clips increase storage, privacy surface, and failure modes. | A. Snapshots only. B. Clips required for pilot. | A. Keep a storage interface that can add clips later. | Product owner | Milestone 6 | Open |
| OD-011 | Notification channel | Email, SMS, and WhatsApp differ in cost, delivery proof, and privacy. | Dashboard only; then one external channel. | Dashboard only in Release 1. Choose a channel in Release 1.1 after interviewing operators and pricing the provider. | Product owner | Release 1.1 start | Open |
| OD-012 | Legal entity and data roles | Contracts, tax, and breach duty depend on who sells and who hosts. | Customer-hosted: customer is the decision maker for the footage. Vendor-hosted: roles must be written with counsel. | Do not host customer footage until counsel defines controller and processor duties. On-prem pilot: customer controls the footage; vendor supplies software under a written pilot agreement. | Product owner and counsel | Before any customer footage | Open |
| OD-013 | Pakistan legal and tax review | Workplace CCTV, visitor notice, retention, PECA duties, and provincial sales tax are not decided by this repository. | Commission a review, or limit the pilot to the owner's own premises under a written authorization. | Written authorization for every pilot camera, plus counsel before the first external customer. Do not claim compliance. | Product owner and counsel | Before external pilot | Open |
| OD-014 | Database row security | Application bugs are the usual cause of cross-tenant leaks. Postgres row-level security is a second barrier. | A. Application checks only. B. Application checks plus row-level security on tenant tables. | B in Release 1, before any deployment that can hold two organizations. | Engineering | Milestone 1 | Open |
| OD-015 | API session mechanism | Affects logout, revocation, and future mobile clients. | A. Server-side sessions, HttpOnly cookies. B. Long-lived JWTs in the browser. | A for the web application. Add revocable refresh tokens later for mobile, without changing authorization rules. | Engineering | Milestone 1 | Open |
| OD-016 | Queue technology | Extra services increase on-prem install failure. Redis licensing changed after the 7.2 BSD line. | A. PostgreSQL job table for short jobs; processes for camera streams. B. Celery plus Redis or Valkey. | A. Introduce Valkey only when a measured need for shared rate limits or fan-out appears. | Engineering | Milestone 0 | Open |
| OD-017 | Behavior when a license expires | A hard lock can trap the customer out of their own evidence. A soft lock can be ignored. | Delete nothing. Choose whether ingestion pauses immediately or after a grace period, and whether the dashboard stays readable. | Pause new ingestion at expiry. Keep read access to existing events. Never delete data because a subscription lapsed. | Product owner | Release 1.1 | Open |
| OD-018 | Numeric detection thresholds | False-alert tolerance depends on the site. Invented precision numbers would be false advertising. | Set precision, recall, and false alerts per camera-hour after the first labelled baseline. | Do not set numbers in Release 1 approval. Run the evaluation plan, then set pilot thresholds in writing. | Product owner | After first labelled baseline, before any accuracy claim | Open |
| OD-019 | Benchmark hardware and real cameras | The provisional profile of five 720p cameras cannot be promised without a host and codecs. | Record CPU, RAM, GPU, disk, camera models, codecs, and network. | Start with two authorized test streams. Do not publish a camera-capacity number before the report exists. | Engineering | Milestone 3 and 4 | Open |
| OD-020 | Backup objectives | Restore design depends on acceptable loss and downtime. | Owner states recovery point and recovery time for metadata and for evidence. | Proposed pilot target to test, not an SLA: metadata loss under 24 hours, evidence loss under 24 hours, restore rehearsed within one working day on a single server. | Product owner | Before pilot | Open |
| OD-021 | Who may edit zones | Bad zones create permanent false alerts or blind spots. | Customer administrator only, or administrator plus a delegated installer. | Customer administrator in Release 1. Installer edits only under Release 1.1 delegation. | Product owner | Milestone 5 | Open |
| OD-022 | Repository and CI host | There is no git remote today. | Git host chosen by the owner, with private visibility. | Create a private git repository before Milestone 0. No public footage or secrets. | Product owner | Milestone 0 | Open |

## Assumptions used while these remain open

These are not verified facts.

- The owner wants a product that can be piloted with real customers, not a demo.
- The owner has, or will obtain, the right to process every camera used in development and pilot.
- The first deployment has one organization, while the schema still supports many.
- English is the pilot user-interface language.
- Continuous full-time recording is not required for the pilot.
- No customer has been contracted by anything present in this repository.
- Team size, budget, and target date are unknown. Effort notes in the implementation plan are relative, not a schedule.

## Verified facts

These were checked on 27 October 2026. They are not assumptions.

- `D:\Ai` contains `docs\AI_CCTV_Monitoring_Agent_SRS_v1.0.docx` and, after this task, the markdown files named in the README.
- The Word file's package metadata says it was generated by `python-docx`. Its document text is titled version 1.0, status "Draft for stakeholder review", dated 27 October 2026. The file's core timestamps are the `python-docx` default of 2013-12-23, so those timestamps are not evidence of when it was written.
- The Word draft says requirements are not frozen and that camera count, performance, retention, notifications, and environment still need confirmation.
- The directory is not a git repository.
- No application source, tests, or dependency files were present.
- `git` is available as a command, but there is no repository. `python`, `py`, `docker`, `node`, and `ffmpeg` were not available on the PATH of this workstation.

## Decisions that look settled but are not

The recommended stack in [architecture.md](architecture.md) is an engineering recommendation. OD-003, OD-004, OD-005, OD-014, OD-015, and OD-016 can overturn parts of it. Do not describe those choices as already adopted in a customer document.
