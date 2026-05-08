# ROADMAP.md

## Project Roadmap

This document outlines the planned evolution of the TrueNAS Bangsar Homelab.

This roadmap reflects forward-looking improvements.
Completed upgrades are recorded in CHANGELOG.md.

Current System Version: v2.0.0

---

# Strategic Direction

The current platform is the MSI GE66 Raider (i7-10875H, 8c/16t, 32GB DDR4, 2×1TB NVMe RAID 0), serving as a capable interim NAS. It handles all current workloads without meaningful constraint.

The long-term objective is to migrate to a purpose-built Xeon NAS for true enterprise-grade storage redundancy, physical drive expandability, and multi-year platform longevity — when hardware availability and pricing conditions are right.

Core focus areas:

- Hardware scalability
- Storage resilience
- Offsite backup
- Workflow automation
- Service stability
- Operational maturity

---

# 1. Infrastructure Upgrade – Xeon NAS Migration

## Target Specification

### CPU
Intel Xeon E-2246G
- 6 Cores / 12 Threads
- 80W TDP
- UHD P630 iGPU (Quick Sync support)

### Motherboard
BKHD C246 NAS Mini-ITX
- 8× SATA
- 2× M.2 NVMe
- 4× 2.5G LAN
- Up to 64GB DDR4

### RAM
- 16GB DDR4 ECC (initial)
- Upgrade path to 32GB+

### Storage
- 4 × 1TB 2.5" HDD
- Planned layout: RAIDZ1 or RAID10 (final decision pending)
- Future expansion: up to 8 drives

### Boot
- 128GB NVMe (existing)

### Cooling
Low-profile 4-heatpipe cooler (92mm or 120mm fan required for 80W CPU stability)

---

### Objectives

- Eliminate laptop thermal constraints
- Increase compute capacity for container workloads
- Enable stable hardware transcoding (Plex / Jellyfin)
- Improve storage scalability
- Support 2.5G networking
- Provide 4–6 year platform longevity

---

### Status
On Hold — Pending hardware availability and pricing

### Hold Rationale

- HDDs currently out of stock (4× 1TB 2.5" HDDs for RAIDZ1/RAID10 pool)
- DDR4 ECC RAM pricing currently too high
- Current platform (MSI GE66 Raider) meets workload needs in the interim
- Will be revisited when market conditions improve

---

# 2. Offsite Backup Implementation

## Status: Completed — v2.0.0

Implemented via TrueNAS SCALE Cloud Sync tasks (TrueNAS manages rclone internally).

- Microsoft OneDrive target (1TB plan)
- 4 datasets configured for cloud sync
- Daily sync schedule: 12:00 AM
- Automated and managed natively through TrueNAS UI

### Objective

Provide offsite redundancy to mitigate:

- Disk failure
- Pool corruption
- Physical hardware loss

See CHANGELOG.md v2.0.0 for full implementation details.

---

# 3. Service Stabilization

## RustDesk

Current Status:
Resolved - Fully operational as of v1.2.0

Actions Taken:
- Updated to v1.7.1
- Deployed dedicated RustDesk Relay app
- Configured self-hosted relay via Tailscale
- Web client access enabled on port 21119

---

# 4. Workflow Automation Expansion

## n8n Deployment

Current Status:
Resolved - n8n hosted on external Coolify VPS (not TrueNAS)

Decision:
- n8n deployment moved to Coolify VPS instead of TrueNAS Apps
- Offloads automation workload from resource-constrained laptop hardware
- TrueNAS integrates with external n8n instance via API/webhooks

Potential Integrations:
- Backup notifications
- Immich event triggers
- Nextcloud activity automation
- Infrastructure alerts

---

# 5. Documentation & Optimization

Planned Improvements:

- Storage benchmarking
- Snapshot scheduling documentation
- Dataset structure refinement

---

# Roadmap Lifecycle Policy

When a roadmap item is completed:

1. Record completion in CHANGELOG.md (new version release)
2. Update README.md to reflect the new current system
3. Update this roadmap section to "Completed" or remove it

This ensures separation between:
- Planning (Roadmap)
- History (Changelog)
- Current State (README)

---

End of Roadmap
