# Data governance

**Status:** Draft 0.1, 27 October 2026. Not a privacy policy, not a legal opinion, and not an implemented control.

The product processes video of people. That is the point of CCTV and the reason this file exists. Counsel must review the questions in the last section before an external customer is onboarded. Nothing here claims that the product complies with any statute.

## Roles

| Deployment | Who decides why cameras run | Who operates the software | Notes |
| --- | --- | --- | --- |
| Customer-hosted pilot | Customer | Customer, possibly with an installer | The vendor is supplying software. Do not copy footage into a vendor laptop "for debugging" unless the agreement allows that specific copy. |
| Installer-hosted, several customers | Each end customer for their own cameras | Installer | The installer is handling other people's footage. Contracts must say so. The software must isolate tenants. |
| Vendor-hosted | Customer | Vendor | Controller and processor, or the local equivalent, must be written by counsel before this mode is sold. OD-012. |

The software cannot determine whether the customer is allowed to point a camera at a place. The administrator who adds a camera is asserting that authorization. The pilot agreement should make that assertion explicit.

## Classification

| Class | Examples | Handling |
| --- | --- | --- |
| Restricted | Camera passwords, session secrets, password hashes, encryption keys, license secrets | Encrypted or hashed, never logged, never returned to the browser, not in backups that are casually emailed |
| Confidential | Video frames in memory, snapshots, events, zones, rules, audit entries that name people or sites, exports | Tenant scoped, access controlled, retained for a defined period, included in deletion |
| Internal | Service logs without secrets, performance counters, model version strings | Still not public. May contain camera ids |
| Public | Marketing pages that do not exist yet | Must not include customer footage |

Zones are confidential because they show which door the customer cares about.

## Data inventory for Release 1

| Data | Where | Created by | Retention owner |
| --- | --- | --- | --- |
| User account and password hash | Database | Administrator | Life of the account, then delete or disable per offboarding |
| Session | Database | Login | Hours, then delete. Hashed id |
| Organization, site | Database | Administrator | Life of the tenant |
| Camera configuration | Database | Administrator | Life of the camera configuration; history may remain in audit |
| Camera password | Database, encrypted | Administrator | Until rotated or camera removed. Ciphertext deleted with the camera secret row |
| Reference frame for the zone editor | Evidence store | Zone editor | Replace on new capture. Delete with the camera. It can show people, so it is confidential |
| Sampled frames in worker memory | RAM | Worker | Not written to disk except the chosen snapshot |
| Event row | Database | Worker | Metadata retention period |
| Snapshot | Evidence store | Worker | Evidence retention period, proposed shorter than metadata |
| Audit row | Database | API | Longer than operational logs |
| Application log | Container logs | All processes | Short. Proposed 30 days. Not a second evidence store |
| Backup | Operator-chosen media | Operator | Must not outlive the policy the customer thinks they have. OD-020 |
| Entitlement | Database | Platform or local admin | Life of the contract |

Release 1 does not store continuous video, audio, face templates, or a gallery of identities.

The product cannot delete "all frames of one visitor" without building biometric search, and that feature is forbidden. Deletion is by time, event, camera, site, or organization. Say this in the customer documentation so nobody promises selective erasure of a person.

## Minimization

- Infer on a sample of frames, not in order to keep every frame.
- Persist a snapshot only when a rule creates an event, plus a single replaceable reference image for drawing zones.
- Store the box and confidence, not a cropped face dataset.
- Detector output contract has no biometric attributes. See [architecture.md](architecture.md).
- Do not send frames to a third-party API in Release 1. Detection runs locally.
- Logs record ids and error codes, not images and not URLs with passwords.

Optional clips are out of the pilot unless OD-010 changes. If they are added, they inherit evidence classification and a short retention.

## Access

| Data | Customer admin | Operator | Auditor | Platform admin | Worker process |
| --- | --- | --- | --- | --- | --- |
| Camera password | Write only, never read back | No | No | No | Decrypt in memory to connect |
| Live configuration | Yes | Read status only | Read status only | No, unless a support grant says so | Read assigned cameras |
| Events | Yes | Yes | Yes | Only with grant | Insert |
| Snapshots | Yes | Yes | Yes if the role includes evidence | Only with grant | Write |
| Audit | Yes, own organization | No | Yes | Own administrative actions; customer audit only with grant | No |
| Other organizations | No | No | No | No standing access | No |

Evidence reads are audited. The audit entry records who and which evidence id, not the image.

Support grants are time-bounded and empty by default. An on-prem customer install should not contain a vendor user.

## Retention and deletion

Proposed defaults, **not approved** (OD-008):

| Data | Proposed period |
| --- | --- |
| Snapshots and reference frames | 30 days, reference frame earlier if replaced |
| Event metadata | 180 days |
| Audit | 365 days |
| Application logs | 30 days |
| Sessions | Absolute expiry well under 24 hours for a pilot, exact value set in Milestone 1 |
| Backups | Decide with OD-020 so they do not keep snapshots for a year after the product deleted them |

The customer administrator may set a shorter evidence retention within bounds chosen at install. They may not set "forever" through the UI in the pilot. A longer period is a deliberate configuration change, recorded in audit.

Deletion behavior:

1. Expiry job removes bytes, then marks the evidence row, then eventually removes expired event metadata according to its own schedule.
2. A failed file delete leaves the row in a visible error state and retries. It does not pretend success.
3. Removing a camera stops processing and deletes the secret. Events remain until retention or purge.
4. Organization offboarding is an audited purge: disable logins, delete secrets, delete evidence bytes, delete or anonymize operational rows as the agreement requires, and record completion. Audit of the purge itself may need to remain for the vendor's own account; counsel should say how long.
5. Orphan files are reconciled.
6. Restore of an old backup can resurrect deleted evidence. Operators must know that. After a purge, old backups that contain the tenant are either destroyed or kept under a named legal hold. There is no silent middle state.

## Encryption

| Layer | Intent | Limit |
| --- | --- | --- |
| Browser to proxy | TLS in every pilot that is not loopback development | Not implemented until certificates exist |
| Worker to camera | `rtsps` when the camera supports it. Many cameras will only offer RTSP | Passwords on plain RTSP can be sniffed on the camera VLAN. Say so in the install notes |
| Database and evidence at rest | Disk encryption provided by the host, plus application encryption for camera passwords | Application-level encryption of every JPEG is optional and not required for the first pilot if disk encryption is on and documented. If disk encryption is off, the pilot agreement must say the files are protected only by permissions |
| Backups | Same sensitivity as the source. Encrypt backup files if the tool supports it | A backup on an unlocked USB drive is a second breach |

Key for camera-password encryption is not stored in the database it encrypts. Rotation is a runbook item: new key version, re-encrypt, keep the old key until complete.

## Tenant isolation

Every confidential row is scoped by organization. Tests in AC-007 are a gate even for a one-customer pilot, because the next deployment may be an installer server.

Exports, when they exist in Release 1.1, are authorized, audited, and limited to metadata unless a separate evidence-export permission exists. Default export does not zip every snapshot.

Partner users see only organizations that delegated access, and only until expiry.

## Training data, later

Release 2 may copy authorized recordings into a dataset. That copy is a new use. Requirements before any copy:

- A recorded decision that this customer allowed this footage to be used for training.
- No mixing of organizations in one training set unless each organization agreed and the purpose allows it.
- Provenance: source event or file, time, camera, who labelled it.
- Labels for correct, false, missed, and ambiguous.
- Train, validation, and test splits that do not share the same incident.
- Storage separated from the live evidence volume so a model-training job cannot use the API's database role.
- Deletion: if the customer revokes training permission, the dataset objects from that customer are removed, including derived files where the agreement says so. Model weights that already absorbed the data are a counsel question; the product must be able to identify which training run used the data.

None of this pipeline exists. Do not start it informally by copying a pilot folder onto a developer machine.

## Backups

Backups are for recovery, not for a second analytics environment. They must not be restored into a developer laptop that also holds unrelated customer data.

Minimum design, to be rehearsed under AC-012:

- Database dump on a schedule the owner accepts.
- Evidence volume copied with the same schedule, or an explicit decision that evidence is rebuildable and is not backed up. That decision has not been made. Default intent is to back up evidence because investigations depend on it.
- A restore onto a clean machine, followed by a login and an event read, recorded in writing.
- Backup files stored somewhere that a ransomware event on the same disk does not automatically destroy. A second disk or an offline copy is an operational requirement for a real pilot. The product does not magically provide the second disk.

## Logging and support

Support staff do not receive a bundle that contains snapshots unless the customer has approved that incident and the transfer. Release 1 has no "send diagnostics to vendor" button, which avoids building an accidental exfiltration feature.

## Legal and privacy questions for counsel

These questions are unresolved. The product owner should take them to a qualified adviser. Answers will change contracts and may change retention defaults.

1. What is the current Pakistani legal framework that applies to CCTV in workplaces, commercial buildings, and private security companies, including any personal-data statute actually in force on the deployment date, and the Prevention of Electronic Crimes Act where computer access and data are concerned?
2. What notice is required for employees, contractors, and visitors, and does the customer or the installer give it?
3. Is after-hours monitoring of a staff entrance treated differently from a public shop floor?
4. Who is the decision-maker for the footage in a customer-hosted install, and who is that person if an installer hosts several clients on one server?
5. What is the breach-notification duty, and to whom, if snapshots are copied without authorization?
6. How long may event images be kept for a security purpose before the period is hard to justify?
7. What must happen to data at the end of a pilot and at the end of a paid contract?
8. Can footage be transferred to a vendor engineer in another city, or to a cloud region outside Pakistan, for support or hosting?
9. Which sales-tax and invoicing rules apply to software licenses and to installation services in the seller's province?
10. If a later customer is in a jurisdiction with a comprehensive privacy law, which product controls (deletion, access log, processing record, subprocessors) are still missing?
11. Are there sector rules for the first design partner, for example a factory, a school, or a hospital? Schools and hospitals are not the recommended first pilot.
12. Does the contract need to forbid the customer from using the product for covert workplace surveillance or for any biometric purpose?

Until those answers exist, the honest external statement is: the software is being designed for authorized security monitoring with human review, and compliance has not been assessed.

## Customer responsibilities the product will not absorb

- Lawful placement of cameras and a defined purpose.
- Signs or notices, if required.
- Not aiming cameras at areas they are not allowed to monitor.
- Deciding what a guard does when an alert appears.
- Their own personnel access to the dashboard.
- Providing the server, power, and a place for backups in the customer-hosted model.

The dashboard must still show, in the product, that detection can be wrong and that a quiet screen does not prove an area is empty.
