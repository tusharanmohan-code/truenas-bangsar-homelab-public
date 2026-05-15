# 04_Services_Deployment.md

## Services Deployment Overview

The TrueNAS homelab runs multiple self-hosted services to provide media streaming, cloud storage, DNS filtering, secure remote access, and workflow automation.

Applications are deployed via TrueNAS SCALE Apps (containerized workloads) with persistent storage mapped to dedicated ZFS datasets.

Current System Version: v2.1.0

---

## 1. Plex

### Purpose
Media streaming server for Movies and TV Shows.

### Media Datasets
pool/LM/Plex/Movies
pool/LM/Plex/TVShows

### Config Dataset
pool/LMApps/Plex_Config

### Deployment Notes
- Media stored separately from configuration
- Structured library separation for indexing efficiency
- Wired Ethernet connection ensures stable streaming

---

## 2. Jellyfin

### Purpose
Open-source alternative media server.

### Config Datasets
pool/LMApps/Jellyfin/Config
pool/LMApps/Jellyfin/Cache

### Media Source
Shared from pool/LM

### Deployment Notes
- Used for comparison testing with Plex
- Configuration isolated for easier rebuild
- Cache separated from config dataset

---

## 3. Immich

### Purpose
Self-hosted photo backup and management platform, replacing Google Photos.

### Datasets
pool/LM/Immich
pool/LMApps/Immich/db (legacy - Postgres 14)
pool/LMApps/Immich/db_new (Postgres 18 - active)
pool/LMApps/Immich/uploads

### Mobile Backup
- Android phone configured for automatic backup via Immich mobile app
- App uploads direct to NAS over local network; Tailscale used when remote

### Google Photos Migration
- Migration completed May 2026
- Source: Google Takeout export
- Import method: Immich CLI with sidecar metadata handling
- Albums recreated from Google Takeout folder structure using `--album` flag
- Import order: phone app backup first, then Takeout CLI upload

### Incident
- Original deployment used PostgreSQL v14
- During update, required Postgres version became unavailable
- Immich failed to start due to database incompatibility

### Resolution
- Created new dataset: pool/LMApps/Immich/db_new
- Deployed new PostgreSQL instance (v18)
- Recreated database user
- Reinstalled Immich with new database configuration

### Notes
- Original database dataset (db) preserved for reference/recovery
- Media uploads dataset remained unchanged
- Metadata tied to old database not automatically migrated
- Highlights importance of database backup before upgrades
- See 06_Lessons_Learned.md §16 for detailed Google Photos migration notes

---

## 4. Nextcloud

### Purpose
Private cloud storage and file synchronization.

### Deployment Notes
- Used for internal file storage
- Accessible within LAN
- Remote access via Tailscale

---

## 5. AdGuard Home

### Purpose
Network-wide DNS filtering and ad blocking.

### Dataset
pool/LMApps/adguard_certs

### Deployment Notes
- Handles DNS queries within LAN
- Reduces tracking and unwanted traffic
- Operates behind router without public exposure

---

## 6. Tailscale

### Purpose
Secure remote access to homelab environment.

### Dataset
pool/LMApps/Tailscale/Data

### Deployment Notes
- Eliminates need for port forwarding
- Encrypted peer-to-peer connectivity
- Reduces external attack surface

---

## 7. RustDesk

### Purpose
Self-hosted remote desktop management.

### Dataset
pool/LMApps/Rustdesk

### Deployment Notes
- Enables secure remote desktop sessions
- Remote access routed via Tailscale overlay network
- Relay functionality moved to dedicated RustDesk Relay app (v1.7.1 architecture change)
- Configured to use self-hosted relay via Tailscale

---

## 8. RustDesk Relay

### Purpose
Dedicated self-hosted relay server for RustDesk remote desktop connections.

### Dataset
pool/LMApps/Rustdesk_Relay

### Deployment Notes
- Handles relay and web client relay traffic on dedicated ports
- Accessed via Tailscale overlay network
- Separated from main RustDesk app as of v1.7.1 architecture change
- Required for both native app and web browser client access

---

## 9. Jackett

### Purpose
Indexer aggregator providing a unified API across multiple sources for automation workflows.

### Dataset
pool/LMApps/Jackett

### Deployment Notes
- Provides unified API for content source indexing
- Integrated with FlareSolverr for Cloudflare-protected sources

---

## 10. Transmission

### Purpose
Download client for automated media acquisition workflows.

### Dataset
pool/LMApps/Transmission

### Deployment Notes
- Integrated with Jackett as part of automation pipeline
- Lightweight and stable download client

---

## 11. FlareSolverr

### Purpose
Bypasses Cloudflare challenge pages for indexers used within automation workflows.

### Dataset
pool/LMApps/Flaresolverr

### Deployment Notes
- Required for Cloudflare-protected indexer sources used by Jackett
- Acts as a proxy to resolve challenges and enable reliable automation

---

## 12. Deployment Architecture Principles

| Principle              | Implementation                                    |
|------------------------|---------------------------------------------------|
| Persistent Storage     | All services mapped to dedicated datasets         |
| Media Separation       | Media stored under LM, configs under LMApps       |
| Container Isolation    | Each service deployed independently               |
| No Direct WAN Exposure | Remote access handled via Tailscale               |
| Automation Ready       | Services integrated with external n8n workflows   |

---

## 13. Operational Considerations

- Services restart automatically via TrueNAS Apps
- Configurations stored within ZFS datasets
- App data separation simplifies migration
- All services operate within single LAN subnet

---

## Engineering Assessment

This service deployment demonstrates:

- Containerized workload management
- Persistent storage mapping strategy
- Logical separation of stateful vs media data
- Secure remote access configuration
- Multi-service orchestration within a constrained environment
- Integration of automation-ready services

The system reflects structured service planning with increasing operational maturity.

---

End of Services Deployment Documentation
