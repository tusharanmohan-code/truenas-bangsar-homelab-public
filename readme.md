# TrueNAS Home Homelab

A self-hosted infrastructure project built on TrueNAS SCALE to provide media streaming, private cloud storage, network-wide DNS filtering, and secure remote access.

This homelab is designed as a structured learning environment focused on storage architecture, networking fundamentals, automation, and infrastructure documentation.

Current System Version: v1.2.0

---

## Project Objectives

- Deploy and manage a functional NAS environment
- Implement structured ZFS storage architecture
- Deploy containerized self-hosted services
- Automate system configuration backups
- Maintain version-controlled infrastructure documentation
- Practice risk-aware system design under hardware constraints

---

## Hardware Architecture (Current)

### Host System
HP Pavilion AC (Laptop-based NAS)
Intel Core i5-5200U (2 Cores / 4 Threads)
16GB DDR3L RAM

### Boot Device
256GB Kingston M.2 SATA SSD
Connected via USB enclosure

### Storage Pool
2 x 500GB HGST 2.5" HDD
Configured as Striped vdev (RAID 0 equivalent)
~1TB usable capacity

Note: This configuration prioritizes capacity and performance over redundancy.

---

## Network Architecture

ISP: Fiber (300 Mbps)

Network Flow:

GPON Modem
> Mesh Router
> Unmanaged Switch
> TrueNAS Server / Smart TV / PlayStation 5

Design Focus:
- Wired NAS connection for stability
- Centralized routing
- Layer 2 switching for LAN expansion
- Remote access secured via Tailscale (no port forwarding)

---

## Storage Design

Pool Name: pool
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
- pool/boot_config (System configuration backups)
- pool/scripts (Automation scripts)

Storage Characteristics:

- No redundancy (RAID 0)
- ZFS checksumming enabled
- Logical separation of media and application state
- Configuration backups automated

---

## Deployed Services

| Service       | Purpose                        |
|---------------|--------------------------------|
| Plex          | Media streaming                |
| Jellyfin      | Alternative media server       |
| Immich        | Self-hosted photo backup       |
| Nextcloud     | Private cloud storage          |
| AdGuard Home  | Network-wide DNS filtering     |
| Tailscale     | Secure remote VPN access       |
| RustDesk      | Remote desktop management      |
| RustDesk Relay| Self-hosted relay server       |

All services use persistent ZFS dataset mapping.

---

## Backup & Automation

A custom bash script automates TrueNAS configuration backups using the REST API.

Features:

- API key authentication
- Includes Secret Seed and root authorized keys
- 14-day rolling retention policy
- Cron-based scheduled execution
- Stores configuration archives in pool/boot_config

This mitigates rebuild time in case of storage pool failure.

---

## Design Tradeoffs

| Decision    | Reason                    | Risk               |
|-------------|---------------------------|--------------------|
| RAID 0      | Maximize usable storage   | No redundancy      |
| USB Boot    | Hardware limitation       | Lower reliability  |
| Laptop Host | Cost efficiency           | Limited scalability|

This environment prioritizes learning and architecture experience over enterprise-grade uptime.

---

## Future Direction

See ROADMAP.md for planned upgrades including:

- Migration to Xeon-based NAS platform
- Offsite backup implementation
- Service stabilization
- Workflow automation expansion

---

## Skills Demonstrated

- ZFS storage design
- RAID tradeoff analysis
- Network topology planning
- Containerized service deployment
- REST API automation
- Cron job scheduling
- Risk mitigation strategy
- Infrastructure documentation lifecycle management

---

For historical changes, see CHANGELOG.md.
