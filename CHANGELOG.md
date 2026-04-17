# Changelog

All notable changes to the TrueNAS Home Homelab project are documented in this file.

This project follows Semantic Versioning:

MAJOR.MINOR.PATCH

- MAJOR: Infrastructure-level changes (hardware migration, RAID redesign)
- MINOR: New services or significant feature additions
- PATCH: Small configuration updates or documentation improvements

---

## v1.2.0 - 2026-04-08

### Added

#### Services
- RustDesk Relay (dedicated self-hosted relay server)

#### Storage
- New dataset: pool/LMApps/Rustdesk_Relay

---

### Changed

#### RustDesk Deployment
- Updated RustDesk to v1.7.1
- Relay functionality separated into dedicated RustDesk Relay app
- Configured Additional Relay Servers to point to Tailscale overlay network
- Web client relay accessible via Tailscale

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
- Jackett (torrent indexer aggregator)
- Transmission (download client)
- FlareSolverr (Cloudflare bypass service for indexers)

#### Storage
- New dataset: pool/LMApps/Jackett
- New dataset: pool/LMApps/Transmission
- New dataset: pool/LMApps/Flaresolverr
- New dataset: pool/LMApps/Immich/db_new (Postgres 18)

---

### Changed

#### Immich Deployment
- Rebuilt Immich due to PostgreSQL version incompatibility
- Migrated from Postgres 14 to Postgres 18
- Created new database dataset (db_new)
- Recreated database user
- Reinstalled Immich application

#### Architecture Evolution
- Transitioned from basic homelab services to automation-capable stack

---

### Notes

- Legacy database dataset (pool/LMApps/Immich/db) preserved for recovery/reference
- Media dataset (uploads) remained intact during rebuild
- Database metadata not migrated automatically

---

## v1.0.0 - 2026-03-02

### Initial Release

#### Infrastructure
- Deployed TrueNAS SCALE on HP Pavilion AC (i5-5200U, 16GB RAM)
- Configured 256GB Kingston M.2 SATA SSD (USB enclosure) as boot device
- Created 1TB RAID 0 storage pool using 2 x 500GB HGST HDD

#### Network
- Integrated NAS into home fiber network
- Routed via mesh router
- Connected through unmanaged switch

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
