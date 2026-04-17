# 06_Lessons_Learned.md

## Lessons Learned

This document captures key technical insights gained during the design, deployment, and operation of the TrueNAS homelab.

The focus is on architectural decisions, tradeoffs, and operational maturity.

---

## 1. Hardware Constraints Matter

Running TrueNAS on a laptop platform highlighted:

- Limited CPU headroom under container workloads
- Thermal constraints under sustained load
- Lack of ECC memory support
- Reduced scalability (limited SATA expansion)

Lesson:
Infrastructure design must align with long-term scalability goals. Hardware constraints directly influence system reliability and performance.

---

## 2. RAID 0 Is a Risky Tradeoff

Using a striped vdev (RAID 0 equivalent) provided:

- Full usable storage capacity
- Improved sequential throughput

However:

- Zero fault tolerance
- Single disk failure destroys entire pool

Lesson:
Capacity and performance should never be chosen without a clear recovery strategy. RAID is not backup.

---

## 3. ZFS Provides Integrity Even Without Redundancy

Even in RAID 0 configuration, ZFS provided:

- Checksumming
- Copy-on-write safety
- Dataset isolation
- Snapshot capability

Lesson:
Filesystem choice significantly impacts data integrity and system flexibility.

---

## 4. Dataset Structure Is Critical

Separating:

- Media datasets (LM)
- Application state (LMApps)
- Configuration backups (boot_config)
- Scripts (scripts)

Improved:

- Maintainability
- Migration flexibility
- Troubleshooting clarity

Lesson:
Logical dataset hierarchy simplifies long-term system evolution.

---

## 5. Automation Reduces Risk

Implementing REST API-based configuration backups:

- Reduced rebuild complexity
- Preserved authentication keys
- Created disaster recovery capability

Lesson:
Automation is essential, especially when operating non-redundant storage. Manual processes do not scale.

---

## 6. Secure Remote Access Is Better Than Port Forwarding

Using Tailscale instead of exposing services:

- Reduced attack surface
- Avoided router-level port forwarding
- Simplified remote connectivity

Lesson:
Secure overlay networking is preferable to direct WAN exposure.

---

## 7. Documentation Changes How You Think

Structuring this project into dedicated documentation files revealed gaps in planning and design assumptions.

Lesson:
Writing documentation exposes architectural weaknesses and forces clarity.

---

## 8. Planning Before Upgrading Is Critical

The Xeon migration plan reinforced:

- Importance of workload forecasting
- Cooling and airflow considerations
- Storage redundancy design
- Lifecycle planning

Lesson:
Upgrades should be intentional and documented before implementation.

---

## 9. Service Architecture Evolves

The RustDesk v1.7.1 upgrade introduced a breaking architectural change, splitting relay functionality into a separate dedicated application. Silent migration script failures required log inspection to diagnose.

Lesson:
Version upgrades on self-hosted services can introduce breaking changes. Always review release notes and validate with logs before assuming success.

---

## 10. Infrastructure Is a Lifecycle, Not a Setup

This project evolved from basic NAS experimentation to structured service deployment to automation and roadmap planning.

Lesson:
Infrastructure management is continuous iteration, not a one-time build.

---

## Final Reflection

This homelab represents:

- Practical infrastructure experience
- Tradeoff awareness
- Risk mitigation planning
- Automation mindset
- Documentation discipline

The next phase will focus on scalability, redundancy, and long-term platform stability.

---

End of Lessons Learned
