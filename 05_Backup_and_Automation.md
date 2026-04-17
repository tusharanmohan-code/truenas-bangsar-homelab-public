# 05_Backup_and_Automation.md

## Backup & Automation Overview

Due to the non-redundant RAID 0 storage configuration, automated configuration backups are critical to ensure system recoverability.

This homelab implements a REST API-based backup mechanism to preserve TrueNAS system configuration, including sensitive authentication components.

---

## 1. Backup Strategy

### Objective
Ensure recoverability of:
- System configuration
- Secret Seed
- Root authorized keys

Even if the storage pool fails.

### Scope
This backup protects:
- System settings
- Networking configuration
- User accounts
- App configurations (logical state)

It does not protect:
- Media content
- Application upload data
- Large datasets

---

## 2. Backup Location

Dataset:
pool/boot_config

Backups are stored as:

truenas-config-YYYYMMDD-HHMMSS.tar

This keeps configuration backups logically separated from media and application datasets.

---

## 3. Implementation Method

The backup process uses:

- TrueNAS SCALE REST API
- API key authentication
- curl for API interaction
- Cron for scheduled execution

---

## 4. Script Behavior

The script performs the following steps:

1. Authenticates using a locally stored API key
2. Sends POST request to the TrueNAS config/save endpoint
3. Requests inclusion of Secret Seed and root authorized keys
4. Saves output as .tar archive
5. Validates tar integrity
6. Applies 14-day rolling retention policy

---

## 5. Security Considerations

- API key stored in a restricted local file
- File permissions restricted to root only
- Backup files set to chmod 600
- No public network exposure

This prevents unauthorized access to system configuration.

---

## 6. Retention Policy

Retention Window:
14 days

Automatic cleanup removes archives older than 14 days, ensuring storage space control with no manual intervention required.

---

## 7. Scheduling

Designed for cron-based execution.

Recommended frequency:
Daily execution during low-usage hours.

Automation ensures no manual backup dependency.

---

## 8. Why This Matters (Especially with RAID 0)

RAID 0 provides zero fault tolerance.

If one disk fails:
- Storage pool is lost
- Services must be redeployed

With configuration backups:
- TrueNAS can be reinstalled
- Config archive can be restored
- Services and network settings can be reconstructed quickly

This significantly reduces rebuild time.

---

## 9. Engineering Assessment

This automation demonstrates:

- REST API integration
- Secure credential handling
- Shell scripting capability
- Scheduled task implementation
- Disaster recovery planning
- Risk mitigation awareness

The backup system reflects operational maturity despite hardware limitations.

---

End of Backup & Automation Documentation
