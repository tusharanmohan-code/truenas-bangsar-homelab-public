# 06_Lessons_Learned.md

## Lessons Learned – TrueNAS Bangsar (v1.0.0 – v2.1.0)

This document captures key technical insights gained during the design, deployment, operation, and hardware migration of the TrueNAS Bangsar homelab.

The focus is on architectural decisions, tradeoffs, operational maturity, and hard lessons from a multi-failure hardware migration.

---

# 1. Hardware Constraints Matter

Running TrueNAS on a laptop platform highlighted:

- Limited CPU headroom under container workloads
- Thermal constraints under sustained load
- Lack of ECC memory support
- Reduced scalability (limited SATA expansion)

Lesson:
Infrastructure design must align with long-term scalability goals. Hardware constraints directly influence system reliability and performance.

---

# 2. RAID 0 Is a Risky Tradeoff

Using a striped vdev (RAID 0 equivalent) provided:

- Full usable storage capacity
- Improved sequential throughput

However:

- Zero fault tolerance
- Single disk failure destroys entire pool

Lesson:
Capacity and performance should never be chosen without a clear recovery strategy. RAID is not backup.

---

# 3. ZFS Provides Integrity Even Without Redundancy

Even in RAID 0 configuration, ZFS provided:

- Checksumming
- Copy-on-write safety
- Dataset isolation
- Snapshot capability

Lesson:
Filesystem choice significantly impacts data integrity and system flexibility.

---

# 4. Dataset Structure Is Critical

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

# 5. Automation Reduces Risk

Implementing REST API-based configuration backups:

- Reduced rebuild complexity
- Preserved authentication keys
- Created disaster recovery capability

Lesson:
Automation is essential, especially when operating non-redundant storage.

Manual processes do not scale.

---

# 6. Secure Remote Access Is Better Than Port Forwarding

Using Tailscale instead of exposing services:

- Reduced attack surface
- Avoided router-level port forwarding
- Simplified remote connectivity

Lesson:
Secure overlay networking is preferable to direct WAN exposure.

---

# 7. Documentation Changes How You Think

Structuring this project into:

- README
- Hardware documentation
- Network architecture
- Storage design
- Services deployment
- Automation
- Roadmap
- Changelog

Revealed gaps in planning and design assumptions.

Lesson:
Writing documentation exposes architectural weaknesses and forces clarity.

---

# 8. Planning Before Upgrading Is Critical

The Xeon migration plan reinforced:

- Importance of workload forecasting
- Cooling and airflow considerations
- Storage redundancy design
- Lifecycle planning

Lesson:
Upgrades should be intentional and documented before implementation.

---

# 9. Infrastructure Is a Lifecycle, Not a Setup

This project evolved from:

- Basic NAS experimentation
- To structured service deployment
- To automation and roadmap planning

Lesson:
Infrastructure management is continuous iteration, not a one-time build.

---

# 10. Never Use Second-Hand Drives as an Intermediate Migration Target

During the v2.0.0 hardware migration, two used Toshiba 1TB HDDs were acquired second-hand as a temporary intermediate storage hop between the original HGST drives and the final NVMe target. One drive failed after approximately one day of operation, destroying the entire RAID 0 pool on those drives.

Used and refurbished drives carry an unknown wear history. SMART data may appear healthy, but hidden sector degradation, reallocated sectors accumulated over prior use, and drive fatigue are not always surfaced before failure. Placing a migration-critical dataset on used drives with no redundancy and no prior burn-in is a high-risk gamble.

Lesson:
Intermediate migration targets should use new or known-good drives. Second-hand drives may pass initial checks but fail under sustained write load. If budget prevents new drives, reduce the number of migration hops instead.

---

# 11. Implement Cloud Backup Before Hardware Changes, Not After

OneDrive cloud sync was configured and running before the hardware migration began — an intentional decision made as a data protection step. When the Toshiba drives failed and the intermediate RAID 0 pool was destroyed, all critical datasets were already replicated off-site.

The cloud sync acted as a silent safety net throughout the entire migration failure chain. Without it, the Toshiba failure would have resulted in data loss rather than an inconvenience.

Lesson:
Cloud or off-site backup must be in place and verified before any hardware change begins. Establishing backup after a migration is too late. If the migration fails before backup is set up, the data is gone.

---

# 12. rsync Fails Hard on a Dying Drive — Have a Fallback Plan

When the Toshiba drive began failing mid-migration, an rsync job copying the Plex Movies library (~500GB) stopped entirely when the source drive stopped responding. rsync does not retry failing files, skip unreadable sectors, or resume gracefully from mid-file I/O errors by default. The transfer halted, leaving the destination in an incomplete state.

The resolution required manually copying each Plex movie file individually using cp, monitoring for errors per file, and verifying each copy before moving to the next.

Lesson:
Know your tool's failure behaviour before depending on it for a live migration. rsync is not designed for degraded-drive recovery. For failing media, ddrescue or manual per-file copy with error checking is more appropriate. Have this plan ready before it is needed.

---

# 13. Manual File-by-File Copy Is a Valid Last Resort

When rsync failed on the degraded Toshiba drive, manual per-file copying completed the transfer successfully. It was slower and more labour-intensive, but it provided control that automated tooling could not: each file could be verified individually, errors were immediately visible, and partial transfers could be identified and retried.

The Plex Movies library was fully recovered through this method despite the source drive being in a partially failed state.

Lesson:
Automated tooling is the right default, but manual file-by-file copying with immediate verification is a legitimate recovery technique when drives are degraded. It should be a known option in any migration runbook, not a last-resort improvisation.

---

# 14. RAID 0 Across Second-Hand Drives Amplifies Risk

The intermediate Toshiba RAID 0 pool represented the highest-risk storage configuration possible: no redundancy, unknown drive health, and all data striped across both drives. A single drive failure — which occurred — destroyed the entire pool instantly.

RAID 0 across new drives with known health is already an accepted risk. RAID 0 across used drives with unknown history is compounding two sources of uncertainty simultaneously.

Lesson:
If redundancy is not possible, reduce the migration hop count instead of accepting a RAID 0 across unknown drives. A single healthy drive in a non-redundant configuration is safer than a RAID 0 of two uncertain ones. Capacity is not worth the amplified failure risk.

---

# 15. Platform Upgrade and Data Migration Are Two Separate Risks — Do Not Combine Them

The v2.0.0 migration combined three changes in a single operation: new drives (Toshiba HDDs), new hardware platform (MSI GE66 Raider), and new storage medium (NVMe SSD). Each transition represented an independent failure point with its own failure mode.

When the Toshiba drives failed, it was unclear initially whether the failure was drive-related, enclosure-related, or related to the new hardware environment. Diagnosing the root cause was harder because multiple variables changed simultaneously. Each additional concurrent change increased the blast radius of any single failure.

Lesson:
Separate platform upgrades from data migrations wherever possible. Migrate data first, verify integrity, then change hardware. Each step in isolation is diagnosable. Combined, failures become ambiguous and recovery becomes more complex.

---

# 16. Google Photos to Immich Migration — Metadata and CLI Gotchas

Migrating a Google Photos library to Immich via Google Takeout introduced several non-obvious failure modes that are not well-documented, and some that reflect recent changes to Google's export format and the Immich CLI.

### Google Takeout Sidecar Filename Format Changed

Google Takeout exports metadata as `.supplemental-metadata.json` sidecar files, not `.json` as older guides and community posts suggest. This format change is not prominently documented. Running exiftool without accounting for this pattern caused it to silently skip a large number of photos — they imported with wrong timestamps (defaulting to the file system modification date) rather than the original photo date.

Correct exiftool command to apply dates from supplemental metadata sidecars:

```
exiftool -r -d "%s" -tagsfromfile "%d/%f.%e.supplemental-metadata.json" "-DateTimeOriginal<PhotoTakenTimeTimestamp" "-FileModifyDate<PhotoTakenTimeTimestamp" -ext jpg -ext jpeg -ext png -ext mp4 -ext mov -overwrite_original "/path/to/Takeout/Google Photos/"
```

For files that have no sidecar at all, fall back to extracting the date from the filename:

```
exiftool -r -if 'not $DateTimeOriginal' "-DateTimeOriginal<filename" -overwrite_original "/path/to/Takeout/Google Photos/"
```

### Immich CLI Authentication Uses API Keys, Not Credentials

The Immich CLI `login` command does not accept an email and password. Authentication requires an API key generated from within the Immich web interface: Account Settings → API Keys. The login command is then:

```
immich login <server-url> <api-key>
```

### The `--google-photos` Flag Was Removed

Older versions of the Immich CLI had a `--google-photos` flag to handle Takeout sidecar metadata. This flag was removed in newer releases. The current upload command handles sidecar metadata automatically — no flag is needed. Passing the old flag will cause the command to fail.

### Album Recreation

The `--album` flag recreates Google Photos albums based on the folder structure of the Takeout export. Each subfolder becomes an album. This is the correct approach for preserving album organisation without manual work.

### Do Not Stop a CLI Upload Partway Through

Interrupting a Takeout upload mid-run leaves partially-imported assets in Immich with incorrect dates or missing metadata — because sidecar association happens during the import pass. Cleaning up partial imports is significantly more time-consuming than the original upload. Run the full upload in a screen or tmux session and let it complete.

### Recommended Import Order

1. Phone backup via Immich mobile app (establishes the baseline library)
2. Google Takeout via Immich CLI (adds historical library on top)

Reversing this order risks duplicate detection conflicts where Takeout versions overwrite or conflict with already-imported phone originals.

Lesson:
Google Takeout is a moving target. Verify sidecar filename format before running any metadata tools. Authenticate the Immich CLI with an API key. Do not interrupt bulk imports. Run exiftool on the export before uploading — fixing timestamps after import is harder than fixing them before.

---

# Final Reflection

TrueNAS Bangsar has evolved from an initial single-machine NAS experiment into a platform that has been stress-tested by a real multi-failure hardware migration.

v1.0.0 established the architectural foundations: ZFS dataset structure, service deployment, automation, and documentation discipline. The lessons from that phase were largely theoretical — tradeoffs understood on paper before they were experienced in practice.

The v2.0.0 migration delivered those lessons in full. Used drives failed. A RAID 0 pool was destroyed. rsync halted on a dying drive mid-transfer. Each failure was contained because the groundwork from v1.0.0 was in place: cloud sync was already running, datasets were cleanly separated, and documentation made the recovery path clear.

v2.0.0 running on the MSI GE66 Raider with NVMe storage represents a meaningfully more capable and better-understood platform — earned through the experience of recovering from each failure along the way.

v2.1.0 added the Google Photos migration to Immich, completing the transition away from cloud-dependent photo storage. The migration surfaced new lessons around metadata tooling, CLI authentication, and import sequencing — each added to this document as a record of what was learned in practice rather than theory.

---

End of Lessons Learned
