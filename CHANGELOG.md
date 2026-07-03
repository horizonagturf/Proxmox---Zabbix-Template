# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Added

- **Cluster aggregate items** — nodes online/offline, CPU total/utilization, memory total/used/utilization, VMs running/stopped, LXCs running/stopped (all derived from `proxmox.cluster.resources`).
- **Node certificate monitoring** — per-node HTTP agent fetches `/nodes/{node}/certificates/info`; dependent items expose certificate count and soonest expiry; triggers fire on expiring-soon and expired certificates.
- **Node physical-disk monitoring** — per-node HTTP agent fetches `/nodes/{node}/disks/list`; dependent items expose disk count and inventory; a script item queries SMART status for each disk via `/nodes/{node}/disks/smart`.
- New macro `{$PVE.CERT.EXPIRY.WARN}` (default 30 days) controls certificate expiry warning threshold.
- New value map "Disk SMART status" (1 = Healthy, 0 = Unhealthy).
- Documented additional API paths (`/nodes/{node}/certificates/info`, `/nodes/{node}/disks/list`, `/nodes/{node}/disks/smart`) in the template description.

### Fixed

- Removed the `DOES_NOT_EXIST` filter on `{#QEMU.TAGS}` in QEMU discovery so VMs without any tags are no longer silently excluded.
- The "has been restarted" trigger dependency for QEMU now also requires `onboot=1`, consistent with the hybrid monitoring behaviour (guests with `onboot=0` no longer produce spurious alerts).
