# TrueNAS Bangsar Homelab

A self-hosted infrastructure project built on TrueNAS SCALE to provide media streaming, private cloud storage, network-wide DNS filtering, and secure remote access.

This homelab is designed as a structured learning environment focused on storage architecture, networking fundamentals, automation, and infrastructure documentation.

Current System Version: v2.0.0

---

# Project Objectives

- Deploy and manage a functional NAS environment
- Implement structured ZFS storage architecture
- Deploy containerized self-hosted services
- Automate system configuration backups
- Maintain version-controlled infrastructure documentation
- Practice risk-aware system design under hardware constraints

---

# Hardware Architecture (Current)

## Host System
MSI GE66 Raider (Gaming laptop repurposed as NAS)
Intel Core i7-10875H — 8 Cores / 16 Threads, 2.3 GHz base / 5.1 GHz boost, 45W TDP
32GB DDR4 3200MHz SO-DIMM

## Boot Device
256GB M.2 SATA SSD
Connected via USB Type-C enclosure

## Storage Pool
2 x 1TB NVMe M.2 SSD
Configured as Striped vdev (RAID 0 equivalent)
~2TB usable capacity

Note: This configuration prioritizes capacity and performance over redundancy.

## Thermal Management
Thermal pad replacement + supplemental fan + cooling pad
Required to sustain 24/7 duty cycle at 45W TDP within a gaming laptop chassis.

---

# Network Architecture

ISP: CelcomDigi 300 Mbps Fiber

Network Flow:

GPON Modem
↓
Xiaomi AX3000 Mesh Router
↓
TP-Link 8-Port Unmanaged Switch
↓
- TrueNAS Server
- Smart TV
- PlayStation 5

Design Focus:
- Wired NAS connection for stability
- Centralized routing via AX3000
- Layer 2 switching for LAN expansion
- Remote access secured via Tailscale (no port forwarding)

---

# Storage Design

Pool Name: `pool`
Filesystem: ZFS

Dataset Structure Highlights:

- pool/LM (Media)
  - Plex/Movies
  - Plex/TVShows
  - Immich
- pool/LMApps (Application data)
  - Plex_Config
  - Jellyfin
  - Immich (db / uploads)
  - Tailscale
  - Rustdesk
  - Rustdesk_Relay
  - Jackett
  - Transmission
  - Flaresolverr
- pool/LMApps2 (Reserved for expansion)
- pool/boot_config (System configuration backups)
- pool/scripts (Automation scripts)

Storage Characteristics:

- No redundancy (RAID 0)
- ZFS checksumming enabled
- Logical separation of media and application state
- Configuration backups automated

---

# Deployed Services

| Service          | Purpose |
|-----------------|---------|
| Plex            | Media streaming |
| Jellyfin        | Alternative media server |
| Immich          | Self-hosted photo backup |
| Nextcloud       | Private cloud storage |
| AdGuard Home    | Network-wide DNS filtering |
| Tailscale       | Secure remote VPN access |
| RustDesk        | Remote desktop management |
| RustDesk Relay  | Self-hosted relay server for RustDesk |
| Jackett         | Torrent indexer aggregator |
| Transmission    | Torrent download client |
| FlareSolverr    | Cloudflare bypass proxy |
| n8n             | Workflow automation (hosted on external Coolify VPS, integrates via API/webhooks) |

All TrueNAS-hosted services use persistent ZFS dataset mapping.

---

# Backup & Automation

Two-layer backup strategy protecting configuration state and critical data.

## Layer 1 — Config Backup Script
A custom bash script automates TrueNAS configuration backups using the REST API.

- API key authentication
- Includes Secret Seed and root authorized keys
- 14-day rolling retention policy
- Scheduled Sunday 12:00 AM
- Stores configuration archives in `pool/boot_config`

## Layer 2 — Cloud Sync to OneDrive
TrueNAS SCALE Cloud Sync task replicates critical datasets to Microsoft OneDrive daily.

- Scheduled daily at 12:00 AM
- Datasets synced: `/mnt/pool/LM/Immich`, `/mnt/pool/LMApps`, `/mnt/pool/LMApps2`, `/mnt/pool/scripts`
- Media (Plex library) intentionally excluded — too large, recoverable from source

## TrueNAS Built-in Config Backup
Scheduled Sunday 12:15 AM (offsets from Layer 1 to avoid conflicts).

---

# Design Tradeoffs

| Decision | Reason | Risk |
|----------|--------|------|
| RAID 0 | Maximize usable storage | No redundancy |
| USB Boot | Hardware limitation | Lower reliability |
| Laptop Host | Hardware repurposing | 45W TDP requires active thermal management |
| Thermal Management | Gaming laptop chassis repurposed for 24/7 duty | Requires active cooling stack |

This environment prioritizes learning and architecture experience over enterprise-grade uptime.

---

# Future Direction

See `ROADMAP.md` for planned upgrades including:

- Migration to Xeon-based NAS platform (on hold — HDD out of stock, ECC RAM cost)
- Service stabilization
- Workflow automation expansion

---

# Skills Demonstrated

- ZFS storage design
- RAID tradeoff analysis
- Network topology planning
- Containerized service deployment
- REST API automation
- Cron job scheduling
- Cloud sync integration (TrueNAS Cloud Sync / rclone)
- Multi-layer backup architecture
- Hardware migration and data recovery (including manual file recovery from a failing drive)
- Thermal management for repurposed hardware
- Risk mitigation strategy
- Infrastructure documentation lifecycle management

---

For historical changes, see `CHANGELOG.md`.
