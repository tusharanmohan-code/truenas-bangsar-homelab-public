# 01_Hardware.md

## Hardware Architecture Overview

This TrueNAS homelab is deployed on repurposed consumer-grade hardware. The design intentionally balances cost efficiency, learning value, and functional performance.

---

## 1. Host System

### Device
MSI GE66 Raider (Gaming Laptop repurposed as NAS)

### CPU
Intel Core i7-10875H (Comet Lake-H)
- 8 Cores / 16 Threads
- Base Clock: 2.3 GHz
- Turbo Boost up to 5.1 GHz (single-core max)
- 45W TDP (Configurable TDP-down: 35W)
- 16MB Intel Smart Cache

### RAM
32GB DDR4 3200MHz (SO-DIMM)

---

## 2. Design Rationale

### Why a Gaming Laptop-Based NAS?

- Cost-effective repurposing of existing hardware
- Significantly higher CPU performance than predecessor (8c/16t vs 2c/4t)
- Built-in UPS buffer via internal battery
- NVMe storage pool eliminates HDD noise and rotational latency
- Suitable for medium virtualization, container workloads, and media transcoding

### Tradeoffs vs. Dedicated NAS Hardware

- Higher TDP (45W) compared to low-power mobile predecessors — requires active thermal management
- Gaming chassis not optimized for 24/7 continuous operation thermals
- Limited internal drive bay expansion typical of laptop form factor

This setup prioritizes practical learning, resource optimization, and meaningful performance headroom over enterprise scalability.

---

## 3. Boot Device

### Configuration
- 256GB M.2 SATA SSD
- Housed in an M.2 SATA USB enclosure connected via USB Type-C port

### Reasoning
- Internal M.2 slots reserved for the NVMe storage pool
- Allows easy OS replacement or migration without touching the data pool
- Dedicated OS drive separation from data pool

### Limitation
- USB boot introduces potential reliability concerns
- Not recommended for enterprise production environments

---

## 4. Storage Drives

### Pool Configuration
- 2 × 1TB NVMe M.2 SSD (internal slots)
- Configured as RAID 0 (Striped)
- ~2TB usable capacity

### Rationale
- NVMe sequential speeds (typically 3,000–7,000 MB/s per drive) far exceed HDD throughput
- No spinning disk: eliminates rotational noise, mechanical heat, and seek latency
- Striping across two NVMe drives maximizes bandwidth for media-heavy workloads (Plex, Jellyfin, arr stack)
- Significant upgrade from prior 2 × 500GB HGST HDD RAID 0 (~1TB)

### Tradeoff
- No redundancy — single drive failure results in total pool loss
- Mitigated via configuration backups, snapshot replication, and non-critical dataset usage
- Backup strategy essential (see 05_Backup_and_Automation.md)

---

## 5. Hardware Constraints & Considerations

| Component | Limitation | Impact |
|-----------|------------|--------|
| Gaming Laptop CPU | 45W TDP requires active cooling | Thermal management essential for 24/7 operation |
| DDR4 RAM | Consumer memory standard | No ECC support; data integrity relies on ZFS checksumming |
| USB Boot | Lower reliability than internal SATA/NVMe boot | Requires monitoring; keep spare enclosure |
| RAID 0 | No fault tolerance | Comprehensive backup strategy mandatory |
| Laptop Chassis | Not designed for continuous load thermals | Cooling pad + thermal pad replacement required |

---

## 5a. Thermal Management

The i7-10875H is a high-performance H-series mobile CPU rated at 45W TDP. Running this chip at sustained load in a gaming laptop chassis — repurposed for always-on NAS duty — requires deliberate thermal mitigation.

### Measures Implemented

- **Thermal pad replacement**: Factory thermal interface material replaced with higher-quality thermal pads to improve heat transfer between die and heatsink
- **Supplemental fan**: External fan directed at chassis vents to assist exhaust airflow
- **Cooling pad**: Laptop seated on an active cooling pad with upward-facing fans targeting the chassis underside intake vents

### Rationale

- TrueNAS SCALE under typical NAS workloads (file serving, containers, scheduled scrubs) does not sustain peak CPU boost clocks, so real-world TDP draw is well below the 45W rated maximum
- The cooling stack is sized to handle periodic peak events (Plex transcoding, ZFS scrub, large rsync jobs) without thermal throttling
- Sustained thermal throttling on ZFS workloads can cause write latency spikes and dataset integrity timing issues

---

## 6. Operational Suitability

This hardware configuration is suitable for:

- Media streaming and transcoding (Plex, Jellyfin)
- Personal cloud storage (Nextcloud, Immich)
- DNS filtering (AdGuard Home)
- Secure remote access (Tailscale, WireGuard)
- Lightweight to medium containerized services
- Medium virtualization workloads (8 cores / 16 threads provides meaningful headroom)
- Automation and orchestration (n8n, Home Assistant)

It is not designed for:

- Enterprise-grade uptime (laptop chassis, USB boot)
- Large-scale virtualization or hyperconverged infrastructure
- High-availability clusters
- Critical production workloads requiring ECC memory

---

## 7. Engineering Assessment

This build demonstrates:

- Hardware repurposing and resource optimization under real-world constraints
- Deliberate upgrade decisions: NVMe pool over HDD for speed and silence, higher-core-count CPU for workload headroom
- Practical storage architecture implementation with known RAID 0 tradeoffs accepted and mitigated
- Thermal engineering awareness — active cooling stack designed for a chassis not intended for 24/7 duty
- Separation of OS and data layers (USB boot + internal NVMe pool)
- Risk awareness and mitigation planning (backup automation, snapshot replication)

---

End of Hardware Documentation
