# Threat model

**Status:** Draft 0.1, 27 October 2026. This is a design analysis. It is not a penetration test, and no control listed here is implemented.

Method: identify assets and trust boundaries, then abuse cases. Severity is qualitative. Revisit this file before the first external pilot and after any deployment change, especially hosted multi-tenant service or partner access.

Related requirements are SEC-* in [SRS.md](SRS.md). Planned tests are at the end. There are no test results.

## Assets

| Asset | Why it matters | Classification |
| --- | --- | --- |
| Live and sampled video | Shows people and the layout of a protected site | Confidential |
| Evidence snapshots | Same content, kept on disk | Confidential |
| Camera passwords and RTSP URLs | Use of a password can move a camera or expose other devices | Restricted |
| Session secrets and password hashes | Account takeover | Restricted |
| Events, zones, and rules | Reveal what the customer considers sensitive | Confidential |
| Audit log | Needed after an incident; also sensitive | Confidential |
| Entitlement and license material | Revenue and access limits | Restricted |
| Model weights | License and integrity | Internal or restricted, depending on the vendor terms |
| Backups | A full copy of the above | Same as the source |
| Host and Docker credentials | Full compromise | Restricted |

Availability of monitoring is an asset even though it is not a file. A silent outage can be worse than a loud failure because the customer believes the area is watched.

## Trust boundaries

```mermaid
flowchart TB
  subgraph untrusted [Outside the product]
    browser[Browser and operator]
    cameraNet[Camera subnet]
    internet[Any wider network]
  end
  subgraph host [Customer or vendor server]
    proxy[Proxy]
    api[API]
    worker[Worker processes]
    db[(Database)]
    disk[Evidence volume]
  end
  browser --- proxy
  proxy --- api
  api --- db
  api --- disk
  worker --- db
  worker --- disk
  worker --- cameraNet
  internet --- proxy
```

Boundaries that matter:

1. Browser to proxy.
2. Proxy to API.
3. API and worker to database.
4. API and worker to evidence files.
5. Worker to camera network.
6. Administrator to another organization's data.
7. Partner to a customer.
8. Vendor staff to customer footage.
9. Backup media leaving the host.
10. Build pipeline to released images.

The camera network is not a trusted peer. Cameras are often default-password devices. The product should assume a camera, or a device pretending to be one, may be hostile.

## Actors

| Actor | Starting power |
| --- | --- |
| Anonymous internet or LAN client | Can send requests to whatever port is published |
| Authenticated operator | Can act inside one organization according to role |
| Customer administrator | Can change cameras, users, and zones in that organization |
| Partner user | Should have only delegated access. Feature is Release 1.1 |
| Platform administrator | High risk on a hosted system. Must not be omnipotent over footage by default |
| Person with a foothold on a camera or the camera VLAN | Can attempt to reach the worker or poison the stream |
| Person with a stolen laptop of an operator | Browser session if still valid |
| Malicious or compromised dependency | Build and runtime |
| Insider with server SSH | Outside the application. Operating-system controls apply |

## Abuse cases

### Authentication and sessions

| ID | Threat | What goes wrong | Mitigation in the design | Residual risk |
| --- | --- | --- | --- | --- |
| T-001 | Password guessing | Attacker learns a weak password | Argon2id hashes, minimum length, login rate limit that slows rather than permanently locking the only admin, no default passwords | Credential stuffing of reused passwords remains. MFA is OD-007 |
| T-002 | Stolen session cookie | Attacker replays a session | HttpOnly, Secure, SameSite, hashed session id at rest, expiry, logout revocation, CSRF token on mutations | Malware on the operator PC can still act as the user while the session is valid |
| T-003 | Session fixation or token in a URL | Session leaks via logs and referrers | Cookie session only; never put session or RTSP secrets in query strings | Misimplementation |
| T-004 | JWT that cannot be revoked | Stolen token lasts until expiry | Do not use long-lived browser JWTs for Release 1 | A future mobile token design could reintroduce this if done carelessly |

### Authorization and tenant isolation

| ID | Threat | What goes wrong | Mitigation in the design | Residual risk |
| --- | --- | --- | --- | --- |
| T-010 | Broken object-level authorization | Changing an id returns another customer's event or image | Organization predicate on every query, membership check, RLS backstop, tests that log in as org A and request org B ids | A new endpoint forgets the check. Code review and tests have to treat this as a release blocker |
| T-011 | UI-only authorization | Hidden button still has a working API | Server checks on every mutating and reading operation, including evidence bytes | None if tests are incomplete |
| T-012 | Platform-admin overreach | Vendor staff browse customer video | No standing evidence permission. Time-bounded support grant, audited. On-prem installs ship without a vendor account | A grant left open too long. Process risk |
| T-013 | Partner keeps access after the job | Installer reads a former customer | Expiring grants, customer can revoke, tested in Release 1.1 | Clock skew on expiry |
| T-014 | Privilege change without audit | Administrator role added quietly | Audit membership changes | Audit log tampering by someone with database access. See T-040 |
| T-015 | Insecure direct object reference on files | Guessable path returns a JPEG | Opaque keys, authorization before read, files outside the web root, no directory listing | Backup copies and local server access |

### Camera credentials, SSRF, and the camera network

| ID | Threat | What goes wrong | Mitigation in the design | Residual risk |
| --- | --- | --- | --- | --- |
| T-020 | RTSP password in the browser, logs, or events | Password reused against the camera | Separate secret table, encryption, redaction, API never returns the password, connection test does not echo the URL | Memory dumps and database theft still expose ciphertext. Key management matters |
| T-021 | SSRF | Admin, or an attacker with admin, points the worker at the database, host agent, or `169.254.169.254` | Scheme allowlist, DNS resolution checks, deny link-local, loopback, and metadata ranges, allowlist of camera subnets configured at install | An allowed subnet may still contain non-camera hosts. The allowlist must be tight |
| T-022 | Credential-bearing URL stored whole | Logs and exceptions print `rtsp://user:pass@host` | Store fields separately. Redact userinfo if a library builds a URL | Third-party libraries that log the URL. Test for this |
| T-023 | Worker used as a pivot | Compromised process scans the LAN | Worker can reach only configured camera subnets at the firewall, not the whole LAN, where the customer can enforce that. Container has no need for Docker socket | Depends on the customer's firewall. The product cannot fix a flat network |
| T-024 | Hostile camera or RTSP server | Malformed stream exploits the decoder | Process isolation per camera, resource limits, pinned decoder, drop privileges, run as non-root | Native decoder vulnerabilities remain a real residual risk |
| T-025 | Default camera passwords | Attacker reaches the camera independently of this product | Product does not ship or require default camera passwords. Install guide tells the customer to change them. Out of product control | Customer may ignore it |

### Evidence, privacy, and backups

| ID | Threat | What goes wrong | Mitigation in the design | Residual risk |
| --- | --- | --- | --- | --- |
| T-030 | Unauthorized evidence download | Operator or outsider saves images | Role check, organization check, audit of reads, short-lived response, no public bucket | Authorized users can still exfiltrate. That is a personnel issue plus audit |
| T-031 | Snapshots in logs or error reports | Images copied into support tickets automatically | Logs are metadata only. No screenshot upload feature in Release 1 | Humans can still copy images out |
| T-032 | Retention says deleted but backups remain | Data lives on a USB disk | Runbook includes backup rotation in the retention design. OD-008 and OD-020 | Until backups are actually rotated, policy is fiction |
| T-033 | Disk theft | Laptop or server disk read | Full-disk encryption where the operating system provides it. Volume permissions. This must be an install step, not only an application feature | Encryption that is not turned on |
| T-034 | Deletion that misses a file | Orphan JPEGs | Reconciliation job | Bugs in the job. Needs a test |
| T-035 | Training data taken from production later | Release 2 copies customer video into a dataset the customer did not approve | No training pipeline in Release 1. Future copy requires a recorded permission | Future feature pressure |

### Abuse of the API and supply chain

| ID | Threat | What goes wrong | Mitigation in the design | Residual risk |
| --- | --- | --- | --- | --- |
| T-040 | Audit log edited | Attacker covers tracks | Append-only application behavior, database role cannot update audit rows, restricted read | Superuser on the database can still edit. Integrity is not tamper-proof against a host admin. Say so |
| T-041 | Injection and mass assignment | Unexpected fields change role or organization | Explicit input models, server sets organization from the session and path, parameterized SQL | New fields added carelessly |
| T-042 | Resource exhaustion | Huge pages, many cameras, giant polygons | Page caps, body size limits, camera entitlement, polygon point limits, container limits, bounded frame queues | Capacity still depends on hardware |
| T-043 | Login or API flood | Dashboard unusable | Rate limits on authentication and a general request limit | Distributed low-rate attacks. Acceptable residual for a site-local pilot |
| T-044 | Dependency or base-image compromise | Malicious package in the build | Pinned versions, lockfiles, image digest pins when builds exist, vulnerability scan in CI, no runtime package installs | Zero-days and unused scan waivers |
| T-045 | Secrets in git | Repository leak becomes a camera leak | Example env file only, ignore rules, secret scanning when CI exists. This repository currently has no git | A later commit of a real `.env` |
| T-046 | CSRF | Browser is tricked into a state-changing call | SameSite and CSRF token | Mis-set CORS that reflects credentials. CORS should not be a public wildcard |

### Detection, licensing, and operations

| ID | Threat | What goes wrong | Mitigation in the design | Residual risk |
| --- | --- | --- | --- | --- |
| T-050 | Model or rule trusted too much | Guard ignores the floor because the dashboard is quiet, or confronts someone because of a false alert | UI limitation text, review states, no automatic physical action, evaluation plan | Users can still ignore the warning |
| T-051 | Model file swapped | Worse detections or a malicious model | Checksum recorded for the approved file. Release 2 formalizes promotion. Release 1 still records version | Someone with server access can change files and checksums together |
| T-052 | Poisoned training data | Future model learns an attacker's labels | Out of Release 1. Dataset provenance is a Release 2 requirement | Not mitigated because not built |
| T-053 | License check only in the UI | Customer raises the camera cap from the browser | Server-side entitlement check on enable | Local administrator with database access can edit the row. On-prem license enforcement is a commercial control, not a strong security boundary against the machine owner. Do not pretend otherwise |
| T-054 | Clock rollback to extend a license or retention | Expiry never arrives | Release 1.1 documents offline clock behavior. Retention uses database time, not a client clock | A dishonest on-prem owner can change their own clock. That is their data |
| T-055 | Monitoring gap hidden | Status stays green with no frames | Health definition requires fresh frames and a live inference loop | Bugs in the heartbeat. Needs tests |
| T-056 | Backup restore of an old secret | Rotated camera password overwritten by a restore | Runbook says to record which secrets are inside a backup before restore | Operational mistake |

## Security requirements checklist

Use this before a pilot. Every row is currently **not implemented**.

| Control | Requirement | Pilot gate |
| --- | --- | --- |
| Password hashing and session expiry, logout, revocation | SEC-001 | Required |
| Server-side authorization on every protected operation | SEC-002 | Required |
| Tenant isolation and IDOR tests | SEC-003 | Required |
| Secrets out of git, logs, and API responses | SEC-004, SEC-005 | Required |
| SSRF and camera-network allowlist | SEC-006 | Required |
| TLS for any non-loopback pilot | SEC-007 | Required |
| Evidence authorization and private storage | SEC-008 | Required |
| Input validation | SEC-009 | Required |
| Rate limiting on login | SEC-010 | Required |
| Dependency and image review | SEC-011 | Required before pilot, even if the first scan finds accepted issues |
| Audit log of sensitive actions, no secrets in it | SEC-012 | Required |
| Backup access control | SEC-013 | Required as a procedure |
| Non-root containers, resource limits, no Docker socket in the app | SEC-014 | Required |
| Incident notes for credential leak and data exposure | SEC-015 | Procedure required; no claim of a full SOC |
| CSRF and cookie flags | SEC-016 | Required |
| Worker database least privilege | SEC-017 | Required |
| No vendor backdoor; support grants explicit | SEC-018 | Required |
| MFA for customer administrators | SEC-021 | Follow OD-007 |
| Security headers | SEC-022 | Required |
| Database not published to the LAN | SEC-023 | Required |
| Log redaction tests | SEC-025 | Required |

## Incident handling at design level

The runbook will hold the steps once the system exists. The design commitments are:

- Camera credential compromise: disable the camera record, rotate the password on the camera itself, review audit and connection errors, and treat recorded snapshots as sensitive.
- Suspected cross-tenant access: take the API out of service if hosted, preserve logs, identify the query, notify affected customers under the contract and the law that counsel identifies. Do not freelance a notification standard in code.
- Ransomware or disk loss: restore from the last backup that was actually tested, then change secrets. If backups were on the same disk only, say so in the pilot agreement as a limitation.
- Model or dependency vulnerability: pin an updated version, rebuild the image, and record the change. Do not hot-patch a production container by hand without a note.

There is no bug-bounty or external disclosure program defined.

## Security test plan

Write these tests with the features. Do not mark them passed in advance.

| Test | Shows |
| --- | --- |
| Unauthenticated calls to camera, event, evidence, and user routes fail | Authentication is required |
| Operator cannot create users, change camera passwords, or read audit if that role forbids it | Role checks |
| Two organizations seeded; each user receives only their ids; cross ids return 404 or 403 without revealing the other row | Tenant isolation. Prefer 404 when existence itself is sensitive |
| API response and a log capture during a failed and a successful connection contain no password and no `user:pass@` | Redaction |
| Camera URL cases: `file://`, `http://127.0.0.1`, `http://169.254.169.254`, DNS to a denied address | SSRF controls |
| Evidence path traversal and another organization's storage key | File authorization |
| Login rate limit engages on a burst | Abuse control |
| Audit row written for login failure, role change, camera change, acknowledgement, evidence read | Audit coverage |
| Child process killed; the other test camera still updates last-frame time | Isolation |
| Container configuration review: user, capabilities, published ports | Hardening inspection |

A manual review before pilot walks the checklist above and records the commit hash. That review is not a certification.

## Residual risks to tell a customer

Even with the mitigations implemented and tested:

- People will be missed, and harmless motion will alert.
- A person who controls the server can read the database and the files.
- A person who controls an allowed camera subnet may attack the decoder.
- On-prem license limits are not a security control against the server owner.
- This document is not legal advice and not proof of compliance with Pakistani or any other law.
