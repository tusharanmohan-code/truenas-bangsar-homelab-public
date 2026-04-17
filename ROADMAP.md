# ROADMAP.md

## Project Roadmap

This document outlines the planned evolution of the TrueNAS Home Homelab.

This roadmap reflects forward-looking improvements.
Completed upgrades are recorded in CHANGELOG.md.

Current System Version: v1.2.0

---

## Strategic Direction

The long-term objective is to evolve from a resource-constrained laptop-based NAS into a scalable, reliable, and automation-capable infrastructure platform.

Core focus areas:

- Hardware scalability
- Storage resilience
- Offsite backup
- Workflow automation
- Service stability
- Operational maturity

---

## 1. Infrastructure Upgrade - Xeon NAS Migration

### Target Specification (Locked)

#### CPU
Intel Xeon E-2246G
- 6 Cores / 12 Threads
- 80W TDP
- UHD P630 iGPU (Quick Sync support)

#### Motherboard
BKHD C246 NAS Mini-ITX
- 8x SATA
- 2x M.2 NVMe
- 4x 2.5G LAN
- Up to 64GB DDR4

#### RAM
- 16GB DDR4 (initial)
- Upgrade path to 32GB+

#### Storage
- 4 x 1TB 2.5" HDD
- Planned layout: RAIDZ1 or RAID10 (final decision pending)
- Future expansion: up to 8 drives

#### Boot
- 128GB NVMe (existing)

#### Cooling
Low-profile 4-heatpipe cooler (92mm or 120mm fan required for 80W CPU stability)

---

### Objectives

- Eliminate laptop thermal constraints
- Increase compute capacity for container workloads
- Enable stable hardware transcoding (Plex / Jellyfin)
- Improve storage scalability
- Support 2.5G networking
- Provide 4-6 year platform longevity

---

### Status
Planned - Not Yet Implemented

---

## 2. Offsite Backup Implementation

### Planned

- Integrate cloud storage (1TB plan)
- Implement rclone-based dataset sync
- Schedule automated cloud backup jobs

### Objective

Provide offsite redundancy to mitigate:

- Disk failure
- Pool corruption
- Physical hardware loss

---

## 3. Service Stabilization

### RustDesk

Current Status: Resolved - Fully operational as of v1.2.0

Actions Taken:
- Updated to v1.7.1
- Deployed dedicated RustDesk Relay app
- Configured self-hosted relay via Tailscale overlay network

---

## 4. Workflow Automation Expansion

### n8n Deployment

Current Status: Resolved - n8n hosted on external VPS

Decision:
- n8n deployment moved to external VPS instead of TrueNAS Apps
- Offloads automation workload from resource-constrained laptop hardware
- TrueNAS integrates with external n8n instance via API/webhooks

Potential Integrations:
- Backup notifications
- Immich event triggers
- Nextcloud activity automation
- Infrastructure alerts

---

## 5. Documentation & Optimization

Planned Improvements:

- Storage benchmarking
- Snapshot scheduling documentation
- Dataset structure refinement
- Migration documentation (when Xeon build begins)

---

## Roadmap Lifecycle Policy

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
