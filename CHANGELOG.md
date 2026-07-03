# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Added

- **Cluster aggregate items** — nodes online/offline, CPU total/utilization, memory total/used/utilization, VMs running/stopped, LXCs running/stopped (all derived from `proxmox.cluster.resources`).
- **Backup monitoring** — fetches `/cluster/backup-info/not-backed-up`; items expose the count and list of guests not covered by any backup job; trigger fires when count exceeds `{$PVE.BACKUP.NOTBACKEDUP.WARN}`.
- **HA (High Availability) monitoring** — fetches `/cluster/ha/status/current` and `/cluster/ha/resources`; items expose HA quorum, master node, resources total/started/stopped/error; triggers on quorum loss (Disaster) and resources in error state (High).
- **Ceph monitoring** — fetches `/cluster/ceph/status`; items expose health status, OSD total/up/in, and monitor count; triggers on `HEALTH_WARN`, `HEALTH_ERR`, and OSD(s) down.
- **Node replication monitoring** — script item aggregates per-job status from `/nodes/{node}/replication`; items expose job count, error count, and max lag (seconds); trigger on replication errors.
- **Node ZFS pool monitoring** — fetches `/nodes/{node}/disks/zfs`; items expose pool count, unhealthy pool count, and unhealthy pool names; trigger on degraded/faulted pools.
- **Node subscription monitoring** — fetches `/nodes/{node}/subscription`; items expose status and due date; trigger when subscription is not active.
- **Node APT update monitoring** — fetches `/nodes/{node}/apt/update`; items expose pending update count; trigger when count exceeds `{$PVE.APT.UPDATES.WARN}`.
- **Node failed-task monitoring** — fetches `/nodes/{node}/tasks?errors=1`; items expose 24h failed count and latest failure description; trigger on any failures in the last 24 hours.
- **Node certificate monitoring** — per-node HTTP agent fetches `/nodes/{node}/certificates/info`; dependent items expose certificate count, soonest expiry, and expiring cert name; triggers fire on expiring-soon and expired certificates.
- **Node physical-disk monitoring** — per-node HTTP agent fetches `/nodes/{node}/disks/list`; dependent items expose disk count and inventory; a script item queries SMART status for each disk via `/nodes/{node}/disks/smart`.
- **LXC onboot-gated alerts** — LXC containers now fetch config to determine `onboot` status; "Not running" and "has been restarted" triggers only fire when `onboot=1`.
- New macro `{$PVE.APT.UPDATES.WARN}` (default 50) controls pending-updates alert threshold.
- New macro `{$PVE.BACKUP.NOTBACKEDUP.WARN}` (default 0) controls not-backed-up guest alert threshold.
- New macro `{$PVE.CERT.EXPIRY.WARN}` (default 30 days) controls certificate expiry warning threshold.
- New value maps: "Disk SMART status", "HA quorate", "ZFS pool health".
- Documented all additional API paths in the template description.

### Changed

- **Subscription trigger** — added `manual_close: YES` so the "Subscription not active" problem can be manually closed when a paid subscription is not applicable.
- **Backup not covered trigger** — added `manual_close: YES` so the "Guests not covered by backup" problem can be manually closed when specific guests are intentionally unprotected.

### Fixed

- Removed the `DOES_NOT_EXIST` filter on `{#QEMU.TAGS}` in QEMU discovery so VMs without any tags are no longer silently excluded.
- The "has been restarted" trigger dependency for QEMU now also requires `onboot=1`, consistent with the hybrid monitoring behaviour (guests with `onboot=0` no longer produce spurious alerts).
- SMART check now skips drives reporting `UNKNOWN` health (e.g. behind RAID controllers or lacking SMART support).
