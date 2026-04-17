# 01_Hardware.md

## Hardware Architecture Overview

This TrueNAS homelab is deployed on repurposed consumer-grade hardware. The design intentionally balances cost efficiency, learning value, and functional performance.

---

## 1. Host System

### Device
HP Pavilion AC (Laptop-based NAS)

### CPU
Intel Core i5-5200U
- 2 Cores / 4 Threads
- Base Clock: 2.2 GHz
- Turbo Boost up to 2.7 GHz
- 15W TDP

### RAM
16GB DDR3L

---

## 2. Design Rationale

### Why a Laptop-Based NAS?

- Cost-effective (no additional motherboard purchase)
- Low power consumption (15W mobile CPU)
- Built-in UPS buffer via internal battery
- Suitable for light virtualization and container workloads

This setup prioritizes practical learning and resource optimization over enterprise scalability.

---

## 3. Boot Device

### Configuration
- 256GB Kingston M.2 SATA SSD
- Connected via USB enclosure

### Reasoning
- Internal SATA slots reserved for storage pool
- Allows easy OS replacement or migration
- Dedicated OS drive separation from data pool

### Limitation
- USB boot introduces potential reliability concerns
- Not recommended for enterprise production environments

---

## 4. Storage Drives

### Pool Configuration
- 2 x 500GB HGST 2.5" HDD
- Configured as RAID 0 (Striped)
- ~1TB usable capacity

### Rationale
- Maximizes available storage
- Improves sequential read/write performance
- Suitable for media-heavy workloads (Plex, Jellyfin)

### Tradeoff
- No redundancy
- Single disk failure results in total data loss
- Mitigated via configuration backups and non-critical dataset usage

---

## 5. Hardware Constraints & Considerations

| Component   | Limitation              | Impact                          |
|-------------|-------------------------|---------------------------------|
| Mobile CPU  | Limited cores           | Not ideal for heavy virtualization |
| DDR3L RAM   | Older memory standard   | No ECC support                  |
| USB Boot    | Lower reliability       | Requires monitoring             |
| RAID 0      | No fault tolerance      | Backup strategy essential       |

---

## 6. Operational Suitability

This hardware configuration is suitable for:

- Media streaming
- Personal cloud storage
- DNS filtering
- Secure remote access
- Lightweight containerized services

It is not designed for:

- Enterprise-grade uptime
- Large-scale virtualization
- High-availability clusters
- Critical production workloads

---

## 7. Engineering Assessment

This build demonstrates:

- Resource optimization under hardware constraints
- Practical storage architecture implementation
- Understanding of RAID tradeoffs
- Separation of OS and data layers
- Risk awareness and mitigation planning

---

End of Hardware Documentation
