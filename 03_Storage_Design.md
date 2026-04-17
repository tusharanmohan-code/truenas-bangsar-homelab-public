# 03_Storage_Design.md

## Storage Architecture Overview

The TrueNAS homelab uses ZFS with a single storage pool named:

pool

The pool is configured as a striped vdev (RAID 0 equivalent) across two 500GB HGST HDDs, providing approximately 1TB of usable capacity.

This configuration prioritizes performance and capacity over redundancy.

Current System Version: v1.2.0

---

## 1. Pool Information

### Pool Name
pool

### Configuration
Striped vdev (RAID 0 equivalent)

### Total Usable Capacity
~1TB

### Current Usage
~783 GB used

### Encryption
Unencrypted

---

## 2. Dataset Structure

The dataset layout is structured to logically separate media, applications, and system configuration.

### Root Level

pool
├── boot_config
├── LM
├── LMApps
├── LMApps2
└── scripts

---

## 3. Media & Library Datasets (LM)

LM is used as the primary media storage hierarchy.

pool/LM
├── Immich
├── Plex
│   ├── Movies
│   └── TVShows

### Purpose

- Plex media libraries separated by content type
- Immich dataset for photo storage
- Clean hierarchy for SMB sharing and service mapping

---

## 4. Application Data (LMApps)

LMApps stores persistent application configuration and container data.

pool/LMApps
├── adguard_certs
├── Immich
│   ├── db (Postgres 14 - legacy)
│   ├── db_new (Postgres 18 - active)
│   └── uploads
├── Jellyfin
│   ├── Cache
│   └── Config
├── Plex_Config
├── Rustdesk
├── Rustdesk_Relay
├── Tailscale
│   └── Data
├── Jackett
├── Transmission
└── Flaresolverr

### Purpose

- Isolates app configuration from media datasets
- Enables easier migration or recreation of services
- Allows selective snapshotting if needed

---

## 5. Additional Datasets

### boot_config
Used for storing automated TrueNAS configuration backups (.tar files).

### scripts
Stores custom automation scripts (e.g., REST API backup script, future automation workflows).

### LMApps2
Reserved for future application expansion, staging, or migration testing.

---

## 6. Design Rationale

| Decision                        | Reason                                               |
|---------------------------------|------------------------------------------------------|
| Separate LM and LMApps          | Clean separation of media vs application state       |
| Dedicated Immich DB datasets    | Allows safe DB migration and rollback                |
| Preserve old DB (db)            | Enables recovery or forensic debugging               |
| Introduced db_new               | Clean upgrade path for Postgres version change       |
| Script storage inside pool      | Centralized automation management                    |
| Dedicated RustDesk Relay dataset| Isolates relay service from main RustDesk config     |

This structure improves clarity, maintainability, and migration flexibility.

---

## 7. RAID 0 Characteristics

| Property             | Behavior                          |
|----------------------|-----------------------------------|
| Redundancy           | None                              |
| Performance          | Improved sequential throughput    |
| Capacity Efficiency  | 100% usable                       |
| Failure Tolerance    | 0 disk failures                   |

If either disk fails, the entire pool becomes unreadable.

---

## 8. Risk Mitigation Strategy

- Automated configuration backups stored in boot_config
- Service deployments are reproducible
- Media considered replaceable
- Legacy database preserved for recovery (db dataset)
- Critical credentials preserved via API backup

This homelab is not used for irreplaceable enterprise data.

---

## 9. Engineering Assessment

This storage implementation demonstrates:

- Practical ZFS dataset organization
- Separation of media and application state
- Safe migration strategy (db > db_new)
- Understanding of RAID tradeoffs
- Structured hierarchical dataset design

The configuration reflects intentional architecture with increasing operational maturity.

---

End of Storage Design Documentation
