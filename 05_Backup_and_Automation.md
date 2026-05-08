# 05_Backup_and_Automation.md

## Backup & Automation Overview

Due to the non-redundant RAID 0 storage configuration, automated configuration backups are critical to ensure system recoverability.

This homelab implements two complementary backup mechanisms:

1. **REST API-based configuration backup** — preserves TrueNAS system state (settings, credentials, network, app config) to the local pool.
2. **Cloud Sync to Microsoft OneDrive** — pushes key datasets (app data, uploads, scripts) off-site for data recoverability.

Together these address two distinct failure scenarios: the ability to rebuild the system (config backup) and the ability to recover actual data (cloud sync).

---

## 1. Backup Strategy

### Objective
Ensure recoverability of:
- System configuration
- Secret Seed
- Root authorized keys
- Application upload data
- App configuration datasets
- Operational scripts

Even if the storage pool fails.

### Scope

**Config backup protects (system recoverability):**
- System settings
- Networking configuration
- User accounts
- App configurations (logical state)

**Cloud Sync to OneDrive protects (data recoverability):**
- Immich photo/video uploads (`/mnt/pool/LM/Immich`)
- App config and data volumes (`/mnt/pool/LMApps`, `/mnt/pool/LMApps2`)
- Operational scripts (`/mnt/pool/scripts`)

**Not protected (intentionally excluded):**
- Media content (Plex movies, TV shows) — these are replaceable and too large for cloud sync
- Raw large datasets — cost and volume make cloud sync impractical

### Design Rationale

The exclusion of media libraries from cloud sync is intentional. Large media files (movies, TV shows) are either re-acquirable or replaceable at low cost; syncing them to cloud would consume significant storage quota and bandwidth for minimal recovery benefit. By contrast, Immich photo uploads are irreplaceable personal data, and app/script datasets represent operational effort that is time-consuming to reconstruct.

---

## 2. Backup Location

### Config Backup

Dataset:
pool/boot_config

Backups are stored as:

truenas-config-YYYYMMDD-HHMMSS.tar

This keeps configuration backups logically separated from media and application datasets.

### Cloud Sync

Destination: Microsoft OneDrive (personal or business account, configured in TrueNAS UI)

The TrueNAS Cloud Sync task handles destination path management via rclone. Files are pushed to the configured OneDrive remote as set up in the TrueNAS SCALE Cloud Credentials and Cloud Sync task UI.

---

## 3. Implementation Method

### Config Backup

The backup process uses:

- TrueNAS SCALE REST API
- API key authentication
- curl for API interaction
- Cron for scheduled execution

### Cloud Sync to OneDrive

The cloud sync process uses:

- TrueNAS SCALE built-in Cloud Sync task (UI-configured)
- rclone under the hood (managed by TrueNAS — no manual rclone installation required)
- Microsoft OneDrive as the cloud credential target
- Push direction: local datasets → OneDrive

TrueNAS SCALE natively supports Cloud Sync tasks via the web UI under **Data Protection → Cloud Sync Tasks**. The OneDrive credential is registered once under **Credentials → Backup Credentials → Cloud Credentials**, and referenced by each sync task.

---

## 4. Script Behavior

### Config Backup Script

The script performs the following steps:

1. Authenticates using stored API key
2. Sends POST request to:
   /api/v2.0/config/save
3. Requests:
   - secretseed: true
   - root_authorized_keys: true
4. Saves output as .tar archive
5. Validates tar integrity
6. Applies 14-day rolling retention policy

### Cloud Sync Task Behavior

Each Cloud Sync task is configured and executed by TrueNAS SCALE internally. The task:

1. Connects to the registered OneDrive cloud credential
2. Compares the local dataset contents against the OneDrive destination
3. Pushes new and changed files to OneDrive (push direction)
4. Logs sync results accessible via TrueNAS UI (Tasks → Cloud Sync Tasks → View Logs)

No custom scripting is required — TrueNAS manages the rclone invocation and error handling.

---

## 5. Security Considerations

### Config Backup

- API key stored in:
  [API-KEY-PATH]
- File permissions restricted
- Backup files chmod 600
- No public network exposure

This prevents unauthorized access to system configuration.

### Cloud Sync to OneDrive

- OAuth credentials for OneDrive are stored in TrueNAS's encrypted credential store (not in plaintext scripts)
- TrueNAS SCALE handles token refresh automatically via the rclone OneDrive backend
- Data is transmitted over HTTPS/TLS to Microsoft OneDrive endpoints
- No credentials are embedded in cron jobs or shell scripts

---

## 6. Retention Policy

### Config Backup Retention

Retention Window:
14 days

Command Used:
find pool/boot_config -mtime +14 -delete

This ensures:
- Storage space control
- Automatic cleanup
- No manual intervention required

### Cloud Sync Retention

OneDrive retention follows Microsoft's standard versioning and recycle bin policies for the account type in use. The TrueNAS Cloud Sync task itself does not enforce a local retention window — it syncs the current state of the dataset to OneDrive.

---

## 7. Scheduling

### REST API Config Backup Script

The custom curl/shell script that calls `/api/v2.0/config/save` runs once per week.

Schedule:
12:00 AM every Sunday

Cron expression:
0 0 * * 0

Weekly frequency is appropriate for system configuration because TrueNAS settings change infrequently. Running it more often would provide negligible additional protection while unnecessarily consuming resources during the night window.

### Cloud Sync to OneDrive

All four Cloud Sync tasks (Immich, LMApps, LMApps2, scripts) run on the same daily schedule.

Schedule:
12:00 AM every day

Cron expression:
0 0 * * *

Daily sync ensures that recent personal data (photos, app state, scripts) is always off-site by the following morning. Because these datasets can change frequently — Immich in particular receives new uploads regularly — a weekly schedule would leave an unacceptable window of potential data loss.

### TrueNAS Built-in Config Backup

TrueNAS SCALE includes a native periodic configuration save separate from the custom REST API script. This is configured under **System → Advanced → Periodic Config Backup** in the web UI.

Schedule:
12:15 AM every Sunday

Cron expression:
15 0 * * 0

The 15-minute offset from the REST API script (00:00) is intentional — it ensures the two config backup operations do not run concurrently, avoiding resource contention during the backup window. Both run on Sunday only, matching the weekly cadence appropriate for system configuration changes.

### Schedule Summary

| Task | Schedule | Cron | Rationale |
|---|---|---|---|
| REST API config backup script | 12:00 AM Sunday | `0 0 * * 0` | Weekly; system config changes infrequently |
| Cloud Sync — Immich | 12:00 AM daily | `0 0 * * *` | Daily; irreplaceable personal data |
| Cloud Sync — LMApps | 12:00 AM daily | `0 0 * * *` | Daily; app state changes regularly |
| Cloud Sync — LMApps2 | 12:00 AM daily | `0 0 * * *` | Daily; app state changes regularly |
| Cloud Sync — scripts | 12:00 AM daily | `0 0 * * *` | Daily; small dataset, high recovery value |
| TrueNAS built-in config backup | 12:15 AM Sunday | `15 0 * * 0` | Weekly + 15 min offset to avoid contention with script |

---

## 8. Why This Matters (Especially with RAID 0)

RAID 0 provides zero fault tolerance.

If one disk fails:
- Storage pool is lost
- Services must be redeployed

### Layer 1 — Config Backup: Rebuild the System

With configuration backups:
- TrueNAS can be reinstalled
- Config .tar can be restored
- Services and network settings can be reconstructed quickly

This significantly reduces rebuild time.

### Layer 2 — Cloud Sync: Recover the Data

Even after a successful system rebuild, application data and personal uploads would be lost without an off-site copy. Cloud Sync to OneDrive closes this gap:

- Immich photo and video uploads are restored from OneDrive
- App datasets (LMApps, LMApps2) containing container configs and persistent data are restored
- Operational scripts are immediately available post-rebuild

Together, the two layers address the full recovery surface: the system can be rebuilt (Layer 1) and the data it serves can be restored (Layer 2).

---

## 9. Cloud Sync — Datasets and Selection Rationale

### Synced Datasets

| Dataset | OneDrive Sync | Rationale |
|---|---|---|
| `/mnt/pool/LM/Immich` | Yes | Irreplaceable personal photos and videos — primary reason for cloud sync |
| `/mnt/pool/LMApps` | Yes | App config and persistent data volumes; time-consuming to reconstruct |
| `/mnt/pool/LMApps2` | Yes | Secondary app dataset — same rationale as LMApps |
| `/mnt/pool/scripts` | Yes | Operational scripts; small size, high value for post-rebuild recovery |
| Media (Plex, Movies, TVShows) | No | Replaceable content; volume makes cloud sync impractical |
| `pool/boot_config` | No | Already handled by the config backup mechanism |

### Why Push Direction

Push (local → OneDrive) is the correct sync direction for backup purposes. It ensures that the cloud copy reflects the current state of the local dataset. A bidirectional or pull sync would risk overwriting local data with stale cloud content.

---

## 10. Engineering Assessment

This automation demonstrates:

- REST API integration
- Secure credential handling
- Shell scripting capability
- Scheduled task implementation
- Disaster recovery planning
- Risk mitigation awareness
- Multi-layer backup architecture (system recoverability + data recoverability)
- Cloud integration via TrueNAS SCALE native tooling (rclone/Cloud Sync)
- Intentional scope design — distinguishing irreplaceable data from replaceable content

The backup system has matured from a single-layer config snapshot into a two-layer strategy that addresses both system rebuild time and data loss risk. The config backup handles system recoverability; the OneDrive cloud sync handles data recoverability. This reflects a more complete understanding of the failure modes introduced by a RAID 0 configuration.

---

End of Backup & Automation Documentation
