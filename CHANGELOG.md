# Changelog

All notable changes to the TrueNAS Bangsar Homelab project are documented in this file.

This project follows Semantic Versioning:

MAJOR.MINOR.PATCH

- MAJOR → Infrastructure-level changes (hardware migration, RAID redesign)
- MINOR → New services or significant feature additions
- PATCH → Small configuration updates or documentation improvements

## v2.0.0 - 2026-05-08

### Added

#### Cloud Sync
- OneDrive cloud sync via TrueNAS Cloud Sync tasks
- Four datasets synced daily at 12:00 AM:
  - `/mnt/pool/LM/Immich`
  - `/mnt/pool/LMApps`
  - `/mnt/pool/LMApps2`
  - `/mnt/pool/scripts`
- Implemented as a pre-migration data protection measure before any hardware changes

---

### Changed

#### Hardware Platform
- Migrated from HP Pavilion AC (i5-5200U, 16GB DDR3L) to MSI GE66 Raider (i7-10875H, 32GB DDR4 3200MHz)
- Boot drive unchanged: 256GB M.2 SATA SSD via USB Type-C enclosure
- Thermal management stack added: thermal pad replacement, supplemental fan, cooling pad

#### Storage Pool
- Replaced 2 × 500GB HGST HDD RAID 0 (~1TB) with 2 × 1TB NVMe M.2 SSD RAID 0 (~2TB) using the laptop's internal M.2 slots
- Storage capacity doubled; significant read/write performance improvement over spinning HDD

#### Backup Scheduling
- REST API configuration backup script: weekly, Sunday 12:00 AM
- OneDrive cloud sync (all four datasets): daily, 12:00 AM
- TrueNAS built-in config backup: Sunday 12:15 AM

---

### Notes

#### Intermediate Storage Migration — Toshiba HDD Failure
- Purchased 2 × 1TB Toshiba HDD (second-hand) as an intermediate storage upgrade
- Replaced the 2 × 500GB HGST HDDs with the Toshiba drives in RAID 0; rsynced all data from HGST to Toshiba (Toshiba connected via USB enclosure during transfer)
- One Toshiba drive failed after approximately 1 day of operation, taking the entire RAID 0 pool with it
- Data was not lost at this stage because the OneDrive cloud sync had already completed before the failure

#### Data Recovery to NVMe Pool
- Rsynced data from the partially-failed Toshiba drives to the new NVMe pool
- rsync failed mid-transfer on the Plex Movies library (~500GB) when the second Toshiba drive stopped responding
- Completed the Plex Movies migration by manually copying each movie file individually to the NVMe pool
- All data successfully recovered; no permanent data loss

#### Roadmap Supersession
- The previously planned Xeon server migration (documented in earlier roadmap entries) was superseded by the MSI GE66 Raider platform upgrade

---

## v1.2.0 - 2026-04-08

### Added

#### Services
- RustDesk Relay (dedicated self-hosted relay server)

#### Storage
- New dataset: `pool/LMApps/Rustdesk_Relay`

---

### Changed

#### RustDesk Deployment
- Updated RustDesk to v1.7.1
- Relay functionality separated into dedicated RustDesk Relay app
- Configured Additional Relay Servers to point to Tailscale IP ([YOUR-TAILSCALE-IP]:21117)
- Web client relay accessible at [YOUR-TAILSCALE-IP]:21119 over Tailscale

#### Architecture Evolution
- RustDesk relay now runs as an independent service
- Remote access fully self-hosted via Tailscale overlay network

---

### Notes
- Relay split was a breaking architectural change introduced in RustDesk v1.7.1
- No data loss during transition
- Both native app and web browser client access supported

---

## v1.1.0 - 2026-03-26

### Added

#### Services
- Jackett (torrent indexer aggregator for automation workflows)
- Transmission (torrent client for automated downloads)
- FlareSolverr (Cloudflare bypass service for indexers)

#### Storage
- New dataset: `pool/LMApps/Jackett`
- New dataset: `pool/LMApps/Transmission`
- New dataset: `pool/LMApps/Flaresolverr`
- New dataset: `pool/LMApps/Immich/db_new` (Postgres 18)

---

### Changed

#### Immich Deployment
- Rebuilt Immich due to PostgreSQL version incompatibility
- Migrated from Postgres 14 → Postgres 18
- Created new database dataset (`db_new`)
- Recreated database user
- Reinstalled Immich application

#### Architecture Evolution
- Transitioned from basic homelab services to automation-capable stack
- Introduced torrent automation pipeline (Jackett + Transmission + FlareSolverr)

---

### Notes

- Legacy database dataset (`pool/LMApps/Immich/db`) preserved for recovery/reference
- Media dataset (`uploads`) remained intact during rebuild
- Database metadata not migrated automatically

---

## v1.0.0 - 2026-03-02

### Initial Release

#### Infrastructure
- Deployed TrueNAS SCALE on HP Pavilion AC (i5-5200U, 16GB RAM)
- Configured 256GB Kingston M.2 SATA SSD (USB enclosure) as boot device
- Created 1TB RAID 0 storage pool using 2 × 500GB HGST HDD

#### Network
- Integrated NAS into CelcomDigi 300 Mbps fiber network
- Routed via Xiaomi AX3000 mesh router
- Connected through TP-Link 8-Port unmanaged switch

#### Services Deployed
- Plex
- Jellyfin
- Immich
- Nextcloud
- AdGuard Home
- Tailscale
- RustDesk

#### Automation
- Implemented REST API-based configuration backup script
- Enabled Secret Seed and root key preservation
- Configured 14-day rolling retention policy
- Designed for cron-based scheduled execution

---

End of documented changes
