# Operations runbook

**Status:** Design for the future pilot. Not executable. 27 October 2026.

The commands below describe the intended operational interface so implementation has a target. Running them today will fail. There is no Compose file, no image, and Docker was not installed on the workstation that this document was written on. Nothing here has been rehearsed. AC-012 is unmet.

When the software exists, update this file so every command matches the files that shipped. Delete any step that the software does not actually support.

## Roles

| Role | Does |
| --- | --- |
| Host operator | Installs Docker, disks, firewall, backups, TLS certificates |
| Customer administrator | Manages users and cameras inside the product |
| Vendor | Provides software images and this procedure. Does not receive a standing login |

On a customer-hosted pilot, the host operator is the customer or their installer.

## Before installation

Collect, in writing:

- Authorization for each camera.
- Server OS, CPU, RAM, disk, and whether a GPU exists (OD-003, OD-019).
- Camera subnet ranges that the allowlist will contain.
- Retention choices (OD-008) and backup location (OD-020).
- Confirmation that the server is not exposed to the public internet, or a recorded exception with TLS and a firewall (OD-002).
- A decision on disk encryption (PRIV-017).

Do not continue an external customer install while those are blank.

## Configuration

Configuration is environment variables and mounted files. Names to implement, with placeholders only:

| Name | Purpose |
| --- | --- |
| `CCTV_DATABASE_URL` | PostgreSQL connection for the API role |
| `CCTV_WORKER_DATABASE_URL` | Connection for the worker role |
| `CCTV_SESSION_SECRET` | Session signing or hashing secret |
| `CCTV_DATA_ENCRYPTION_KEY` | Encrypts camera passwords |
| `CCTV_EVIDENCE_ROOT` | Private directory for snapshots |
| `CCTV_CAMERA_ALLOWLIST` | CIDR ranges the worker may open |
| `CCTV_PUBLIC_BASE_URL` | Browser-facing origin |
| `CCTV_LOG_LEVEL` | Log verbosity |
| `CCTV_TLS_CERT_FILE` and `CCTV_TLS_KEY_FILE` | Used when the proxy terminates TLS |

Rules:

- Real values live in a file that git ignores.
- The example file ships with obvious placeholders such as `change-me`.
- Do not put these values in shell history screenshots that get emailed.
- The encryption key is not stored in PostgreSQL.

## Installation shape

Intended layout after Milestone 0:

1. Install the container engine on the pilot OS.
2. Copy the Compose project and the env file onto the server.
3. Start the stack.
4. Run migrations as the migrator role.
5. Create the first customer administrator through a one-time bootstrap command that prints the initial password once. There is no baked-in password.
6. Sign in through HTTPS and confirm a ready state.
7. Add cameras only after the allowlist matches the camera VLAN.

Intended process names: `proxy`, `api`, `worker`, `postgres`.

PostgreSQL listens on the Compose network only (SEC-023). The published port is the proxy.

## Health checks

Intended endpoints:

| Endpoint | Healthy means |
| --- | --- |
| API liveness | The API process responds |
| API readiness | The API can query the database |
| Worker liveness | The supervisor process responds on its private health port |
| Worker readiness | The supervisor heartbeat is fresh |

Unauthenticated responses stay minimal. Camera status is on the authenticated dashboard, not on the public health URL.

A camera showing Online is healthy only under the definition in the architecture: fresh frames and a live processing loop. A green container with a red camera is a camera incident, not a healthy site.

## Daily checks during a pilot

- Cameras not in Online, and how long they have been there.
- Disk free space for database and evidence.
- Evidence failure count.
- Worker restarts.
- Time synchronization.
- Backup job success for the previous night.

There is no paging integration in Release 1. Someone has to look.

## Incidents

Record time, what was observed, what was done, and what data was touched. Do not attach snapshots to a ticket unless the customer approved that transfer.

### Camera down

1. Read the camera error category in the dashboard.
2. Check power and the camera VLAN before changing product settings.
3. Use reconnect after the network path is restored.
4. If only one camera is affected, leave the others running.
5. Tell the operator the gap exists. Do not describe the gap as monitored.

### Suspected password leak

1. Disable the camera in the product.
2. Change the password on the camera itself.
3. Enter the new password in the product.
4. Review audit and proxy logs for the period.
5. Treat existing snapshots as sensitive.

### Suspected cross-tenant access

1. If more than one customer is on the server, stop the API so the issue does not continue.
2. Preserve database logs and application logs.
3. Identify the account and the object ids.
4. Notify according to the contract and counsel. Do not invent a regulatory timeline in the moment.

### Disk full

1. Expect evidence status failed and a visible camera or job error.
2. Free space by the retention job or by expanding disk. Do not delete the database directory by hand.
3. Confirm new events still exist for the period when snapshots failed.

### Host compromise or ransomware

1. Isolate the machine from the camera VLAN and the internet.
2. Restore onto replacement hardware from a backup that was stored on different media.
3. Rotate session secret, encryption key, database passwords, and camera passwords.
4. If the only backup was on the encrypted disk that was lost, the data is gone. That is why OD-020 asks for a second copy.

## Backup

Intended content:

- A logical dump of PostgreSQL.
- A copy of the evidence directory.
- A copy of the env file stored separately from the public documentation, because it contains secrets.

Intended frequency for a pilot, if OD-020 is accepted as proposed: daily, retained only as long as the retention decision allows.

The backup user is an operating-system procedure. It is not the API database role.

Test restore, do not merely test that the dump file is non-empty:

1. Use a separate machine or a separate Compose project.
2. Restore the database and the evidence directory.
3. Start the stack with the restored encryption key.
4. Sign in and open one known event and one snapshot.
5. Write the date, the build, and whether it worked into the operations log.

A backup that has never been restored is not yet a backup.

## Upgrade

1. Read the release notes for migrations and configuration changes.
2. Take a backup and complete a restore test if this is the first upgrade of that site.
3. Pin the new image tags. Do not use a floating `latest` tag.
4. Run migrations.
5. Start the new API and worker.
6. Check readiness and one camera's last-frame time.
7. Watch error logs for secret-redaction mistakes and camera failures.

## Rollback

1. Stop the new API and worker.
2. Restore the pre-upgrade database backup if the new migration wrote data the old code cannot read.
3. Start the previous image tags.
4. Confirm health and a camera frame time.
5. Record that the new version was rejected and why.

If a migration destroyed data and there is no backup, rollback cannot recover it. That is the reason the backup step is first.

## Secret rotation

| Secret | When |
| --- | --- |
| Customer administrator password | On staff change, and if compromise is suspected |
| Session secret | If sessions may have been forged. Users must sign in again |
| Data encryption key | Re-encrypt camera secrets with the new key version, then retire the old key |
| Camera password | On staff change of people who knew it, and after any leak |
| Database passwords | On staff change of host operators |

## Retention operations

The application job deletes according to policy. The host operator additionally:

- Deletes backup generations that are older than the agreed backup lifetime.
- Does not keep a "just in case" full disk image forever after a customer offboarding.
- Confirms orphan reconciliation has no repeated failures.

Offboarding a pilot organization, once the feature exists: run the audited purge, destroy or return media, and record completion. Do not keep a private copy for demos.

## What operators must not do

- Publish port 5432.
- Mount the Docker socket into the API or worker.
- Email `.env` files or snapshot folders.
- Set the camera allowlist to `0.0.0.0/0`.
- Describe the system as fully monitored while a camera is in Error, Reconnecting, or Degraded.
- Patch containers on the server without recording the image id. Rebuild and deploy a known image instead.

## Evidence this runbook is unfinished

- No command in this file has been executed against this product.
- No backup exists.
- No certificate is specified.
- No on-call rota is defined.
- AC-012 has not been run.

Milestone 8 replaces the "intended" wording with the commands that worked, and it records the rehearsal.
