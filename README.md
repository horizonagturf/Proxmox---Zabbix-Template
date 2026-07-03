
# Proxmox VE by HTTP

## Upstream Origin

This repository contains a customized fork of the upstream `Proxmox VE by HTTP` Zabbix template.

- Upstream project: [Zabbix](https://github.com/zabbix/zabbix)
- Upstream license: `AGPL-3.0-only`
- Upstream template export base: `7.0-3`

This fork is maintained so custom changes can be applied outside the main Zabbix project. It is not an official Zabbix release and is not endorsed or supported by Zabbix.

## Overview

This template is designed for the effortless deployment of Proxmox VE monitoring by Zabbix via HTTP and doesn't require any external scripts.

Proxmox VE uses a REST like API. The concept is described in Resource Oriented Architecture (ROA).

Check the [`API documentation`](https://pve.proxmox.com/pve-docs/api-viewer/index.html) for details.

## Requirements

Zabbix version: 7.0 and higher.

## Tested versions

This template has been tested on:
- Proxmox VE

## Configuration

> Zabbix should be configured according to the instructions in the [Templates out of the box](https://www.zabbix.com/documentation/7.0/manual/config/templates_out_of_the_box) section.

## Setup

1. Create an API token for the monitoring user. Important note: for security reasons, it is recommended to create a separate user (Datacenter - Permissions).

Please provide the necessary access levels for both the User and the Token:

* Check: ["perm","/",["Sys.Audit"]]
* Check: ["perm","/storage",["Datastore.Audit"]]
* Check: ["perm","/vms",["VM.Audit"]]

2. Copy the resulting Token ID and Secret into the host macros `{$PVE.TOKEN.ID}` and `{$PVE.TOKEN.SECRET}`.

3. Set the hostname or IP address of the Proxmox API VE host in the `{$PVE.URL.HOST}` macro. You can also change the API port in the `{$PVE.URL.PORT}` macro if necessary.

### Monitoring behavior

- Guests tagged `do_not_monitor` are excluded from QEMU and LXC discovery.
- Guests with `Start at boot` disabled (`onboot=0`) remain discovered, but the `Not running` and `has been restarted` alerts are suppressed (both QEMU and LXC).
- Tag-based exclusion depends on Proxmox exposing guest `tags` in `/cluster/resources`.
- HA, Ceph, and backup monitoring gracefully degrade (return empty data) if those subsystems are not configured.

### Macros used

|Name|Description|Default|
|----|-----------|-------|
|{$PVE.URL.HOST}|<p>The hostname or IP address of the Proxmox VE API host.</p>|`<SET PVE HOST>`|
|{$PVE.URL.PORT}|<p>The API uses the HTTPS protocol and the server listens to port 8006 by default.</p>|`8006`|
|{$PVE.TOKEN.ID}|<p>API tokens allow stateless access to most parts of the REST API by another system, software or API client.</p>|`USER@REALM!TOKENID`|
|{$PVE.TOKEN.SECRET}|<p>Secret key.</p>|`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`|
|{$PVE.APT.UPDATES.WARN}|<p>Alert threshold for number of pending package updates per node.</p>|`50`|
|{$PVE.BACKUP.NOTBACKEDUP.WARN}|<p>Alert threshold for number of guests not covered by any backup job.</p>|`0`|
|{$PVE.CERT.EXPIRY.WARN}|<p>Certificate expiry warning threshold in seconds (default 30 days).</p>|`2592000`|
|{$PVE.ROOT.PUSE.MAX.WARN}|<p>Maximum used root space in percentage.</p>|`90`|
|{$PVE.MEMORY.PUSE.MAX.WARN}|<p>Maximum used memory in percentage.</p>|`90`|
|{$PVE.CPU.PUSE.MAX.WARN}|<p>Maximum used CPU in percentage.</p>|`90`|
|{$PVE.SWAP.PUSE.MAX.WARN}|<p>Maximum used swap space in percentage.</p>|`90`|
|{$PVE.VM.MEMORY.PUSE.MAX.WARN}|<p>Maximum used memory in percentage.</p>|`90`|
|{$PVE.VM.CPU.PUSE.MAX.WARN}|<p>Maximum used CPU in percentage.</p>|`90`|
|{$PVE.LXC.MEMORY.PUSE.MAX.WARN}|<p>Maximum used memory in percentage.</p>|`90`|
|{$PVE.LXC.CPU.PUSE.MAX.WARN}|<p>Maximum used CPU in percentage.</p>|`90`|
|{$PVE.LXC.DISK.PUSE.MAX.WARN}|<p>Maximum used disk in percentage.</p>|`90`|
|{$PVE.LXC.TAGS.NOT_MATCHES}|<p>Exclude LXC containers whose Proxmox tags contain `do_not_monitor`.</p>|`(^&#124;[;,])do_not_monitor($&#124;[;,])`|
|{$PVE.STORAGE.PUSE.MAX.WARN}|<p>Maximum used storage space in percentage.</p>|`90`|
|{$PVE.VM.TAGS.NOT_MATCHES}|<p>Exclude virtual machines whose Proxmox tags contain `do_not_monitor`.</p>|`(^&#124;[;,])do_not_monitor($&#124;[;,])`|

### Items

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Get cluster resources|<p>Resources index.</p>|HTTP agent|proxmox.cluster.resources|
|Get cluster status|<p>Get cluster status information.</p>|HTTP agent|proxmox.cluster.status|
|API service status|<p>Get API service status.</p>|Script|proxmox.api.available|
|Cluster: Nodes online|<p>Number of online nodes in the cluster.</p>|Dependent item|proxmox.cluster.nodes.online|
|Cluster: Nodes offline|<p>Number of offline nodes in the cluster.</p>|Dependent item|proxmox.cluster.nodes.offline|
|Cluster: CPU total|<p>Total number of CPUs across all nodes.</p>|Dependent item|proxmox.cluster.cpu.total|
|Cluster: CPU utilization|<p>Average CPU utilization across all online nodes.</p>|Dependent item|proxmox.cluster.cpu.utilization|
|Cluster: Memory total|<p>Total memory across all nodes.</p>|Dependent item|proxmox.cluster.memory.total|
|Cluster: Memory used|<p>Total memory used across all nodes.</p>|Dependent item|proxmox.cluster.memory.used|
|Cluster: Memory utilization|<p>Memory utilization percentage across the cluster.</p>|Dependent item|proxmox.cluster.memory.utilization|
|Cluster: VMs running|<p>Number of running QEMU virtual machines.</p>|Dependent item|proxmox.cluster.vms.running|
|Cluster: VMs stopped|<p>Number of stopped QEMU virtual machines.</p>|Dependent item|proxmox.cluster.vms.stopped|
|Cluster: LXCs running|<p>Number of running LXC containers.</p>|Dependent item|proxmox.cluster.lxcs.running|
|Cluster: LXCs stopped|<p>Number of stopped LXC containers.</p>|Dependent item|proxmox.cluster.lxcs.stopped|
|Get guests not backed up|<p>Returns a list of guests not covered by any backup job.</p>|HTTP agent|proxmox.backup.notbackedup.get|
|Backup: Guests not backed up count|<p>Number of guests not covered by any backup job.</p>|Dependent item|proxmox.backup.notbackedup.count|
|Backup: Guests not backed up list|<p>Comma-separated list of guest names/IDs not covered by backup.</p>|Dependent item|proxmox.backup.notbackedup.list|
|Get HA status|<p>Get HA manager status including quorum, master, and LRM info.</p>|HTTP agent|proxmox.ha.status.get|
|Get HA resources|<p>Get list of all HA-managed resources and their states.</p>|HTTP agent|proxmox.ha.resources.get|
|HA: Quorum|<p>Whether the HA cluster currently has quorum.</p>|Dependent item|proxmox.ha.quorum|
|HA: Master node|<p>The node currently acting as HA CRM master.</p>|Dependent item|proxmox.ha.master.node|
|HA: Resources total|<p>Total number of HA-managed resources.</p>|Dependent item|proxmox.ha.resources.total|
|HA: Resources in error state|<p>Number of HA-managed resources currently in error state.</p>|Dependent item|proxmox.ha.resources.error|
|HA: Resources started|<p>Number of HA-managed resources in started state.</p>|Dependent item|proxmox.ha.resources.started|
|HA: Resources stopped|<p>Number of HA-managed resources in stopped state.</p>|Dependent item|proxmox.ha.resources.stopped|
|Get Ceph status|<p>Get Ceph cluster status including health, OSDs, and monitors.</p>|HTTP agent|proxmox.ceph.status.get|
|Ceph: Health status|<p>Ceph cluster health status (HEALTH_OK, HEALTH_WARN, HEALTH_ERR).</p>|Dependent item|proxmox.ceph.health|
|Ceph: OSDs total|<p>Total number of Ceph OSDs in the cluster.</p>|Dependent item|proxmox.ceph.osd.total|
|Ceph: OSDs up|<p>Number of Ceph OSDs currently up.</p>|Dependent item|proxmox.ceph.osd.up|
|Ceph: OSDs in|<p>Number of Ceph OSDs currently marked in.</p>|Dependent item|proxmox.ceph.osd.in|
|Ceph: Monitors|<p>Number of Ceph monitors in the cluster.</p>|Dependent item|proxmox.ceph.mon.count|

### Triggers

|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Proxmox VE: API service not available|<p>The API service is not available.</p>|`last(/Proxmox VE by HTTP/proxmox.api.available) <> 200`|High||
|Proxmox VE: Guests not covered by backup|<p>Guests not included in any backup job.</p>|`last(/Proxmox VE by HTTP/proxmox.backup.notbackedup.count) > {$PVE.BACKUP.NOTBACKEDUP.WARN}`|Warning||
|Proxmox VE: HA quorum lost|<p>The HA cluster has lost quorum.</p>|`last(/Proxmox VE by HTTP/proxmox.ha.quorum) = 0`|Disaster||
|Proxmox VE: HA resource(s) in error state|<p>One or more HA-managed resources are in error.</p>|`last(/Proxmox VE by HTTP/proxmox.ha.resources.error) > 0`|High||
|Proxmox VE: Ceph health WARN|<p>Ceph cluster reporting warning health.</p>|`last(/Proxmox VE by HTTP/proxmox.ceph.health) = "HEALTH_WARN"`|Warning||
|Proxmox VE: Ceph health ERR|<p>Ceph cluster reporting error health.</p>|`last(/Proxmox VE by HTTP/proxmox.ceph.health) = "HEALTH_ERR"`|High||
|Proxmox VE: Ceph OSD(s) down|<p>One or more Ceph OSDs are not running.</p>|`last(/Proxmox VE by HTTP/proxmox.ceph.osd.up) < last(/Proxmox VE by HTTP/proxmox.ceph.osd.total)`|Average||

### LLD rule Cluster discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Cluster discovery||Dependent item|proxmox.cluster.discovery|

### Item prototypes for Cluster discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Cluster [{#RESOURCE.NAME}]: Quorate|<p>Indicates if there is a majority of nodes online to make decisions.</p>|Dependent item|proxmox.cluster.quorate[{#RESOURCE.NAME}]|

### Trigger prototypes for Cluster discovery

|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Proxmox VE: Cluster [{#RESOURCE.NAME}] not quorum|<p>Proxmox VE use a quorum-based technique to provide a consistent state among all cluster nodes.</p>|`last(/Proxmox VE by HTTP/proxmox.cluster.quorate[{#RESOURCE.NAME}]) <> 1`|High||

### LLD rule Node discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Node discovery||Dependent item|proxmox.node.discovery|

### Item prototypes for Node discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Node [{#NODE.NAME}]: Status|<p>Indicates if the node is online or offline.</p>|Dependent item|proxmox.node.online[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Status|<p>Read node status.</p>|HTTP agent|proxmox.node.status[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: RRD statistics|<p>Read node RRD statistics.</p>|HTTP agent|proxmox.node.rrd[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Time|<p>Read server time and time zone settings.</p>|HTTP agent|proxmox.node.time[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Uptime|<p>The system uptime.</p>|Dependent item|proxmox.node.uptime[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: PVE version|<p>PVE manager version.</p>|Dependent item|proxmox.node.pveversion[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Kernel version|<p>Kernel version info.</p>|Dependent item|proxmox.node.kernelversion[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Root filesystem, used|<p>Root filesystem usage.</p>|Dependent item|proxmox.node.rootused[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Root filesystem, total|<p>Root filesystem total.</p>|Dependent item|proxmox.node.roottotal[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Memory, used|<p>Memory usage.</p>|Dependent item|proxmox.node.memused[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Memory, total|<p>Memory total.</p>|Dependent item|proxmox.node.memtotal[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: CPU, usage|<p>CPU usage.</p>|Dependent item|proxmox.node.cpu[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Outgoing data, rate|<p>Network usage.</p>|Dependent item|proxmox.node.netout[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Incoming data, rate|<p>Network usage.</p>|Dependent item|proxmox.node.netin[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: CPU, loadavg|<p>CPU average load.</p>|Dependent item|proxmox.node.loadavg[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: CPU, iowait|<p>CPU iowait time.</p>|Dependent item|proxmox.node.iowait[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Swap filesystem, total|<p>Swap total.</p>|Dependent item|proxmox.node.swaptotal[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Swap filesystem, used|<p>Swap used.</p>|Dependent item|proxmox.node.swapused[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Time zone|<p>Time zone.</p>|Dependent item|proxmox.node.timezone[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Localtime|<p>Seconds since 1970-01-01 00:00:00 (local time).</p>|Dependent item|proxmox.node.localtime[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Time|<p>Seconds since 1970-01-01 00:00:00 UTC.</p>|Dependent item|proxmox.node.utctime[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Get certificates|<p>Retrieves certificate information from the node.</p>|HTTP agent|proxmox.node.certs.get[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Certificate count|<p>Total number of certificates on the node.</p>|Dependent item|proxmox.node.cert.count[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Certificate soonest expiry|<p>Earliest notafter value across all certificates.</p>|Dependent item|proxmox.node.cert.notafter.soonest[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Certificate expiring soonest name|<p>Filename of the certificate expiring soonest.</p>|Dependent item|proxmox.node.cert.expiring.name[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Get disks|<p>Retrieves the list of physical disks from the node.</p>|HTTP agent|proxmox.node.disks.get[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Disk count|<p>Total number of physical disks on the node.</p>|Dependent item|proxmox.node.disk.count[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Disk inventory|<p>Summary of all physical disks (devpath, model, serial, type, vendor).</p>|Dependent item|proxmox.node.disk.inventory[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: SMART status|<p>Checks SMART health for all disks. Returns 1 if all healthy, 0 if any failure.</p>|Script|proxmox.node.disk.smart[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Replication status|<p>Aggregated replication job status.</p>|Script|proxmox.node.replication.get[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Replication jobs|<p>Number of replication jobs configured.</p>|Dependent item|proxmox.node.replication.jobs[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Replication errors|<p>Number of replication jobs with errors.</p>|Dependent item|proxmox.node.replication.errors[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Replication max lag|<p>Maximum replication lag in seconds.</p>|Dependent item|proxmox.node.replication.lag[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Get ZFS pools|<p>Get list of ZFS pools on the node.</p>|HTTP agent|proxmox.node.zfs.get[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: ZFS pool count|<p>Total number of ZFS pools.</p>|Dependent item|proxmox.node.zfs.pool.count[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: ZFS pools unhealthy|<p>Number of ZFS pools not in ONLINE state.</p>|Dependent item|proxmox.node.zfs.pool.unhealthy[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: ZFS pools unhealthy names|<p>Names of ZFS pools not in ONLINE state.</p>|Dependent item|proxmox.node.zfs.pool.unhealthy.names[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Get subscription|<p>Get subscription status for the node.</p>|HTTP agent|proxmox.node.subscription.get[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Subscription status|<p>Subscription status (active, expired, invalid, etc.).</p>|Dependent item|proxmox.node.subscription.status[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Subscription due date|<p>Next due date for the subscription.</p>|Dependent item|proxmox.node.subscription.duedate[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Get pending updates|<p>Get list of available package updates.</p>|HTTP agent|proxmox.node.apt.updates.get[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Pending updates count|<p>Number of pending package updates.</p>|Dependent item|proxmox.node.apt.updates.count[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Get failed tasks|<p>Get recent failed tasks on the node.</p>|HTTP agent|proxmox.node.tasks.failed.get[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Failed tasks in 24h|<p>Number of failed tasks in the last 24 hours.</p>|Dependent item|proxmox.node.tasks.failed.count24h[{#NODE.NAME}]|
|Node [{#NODE.NAME}]: Latest failed task|<p>Most recent failed task type and exit status.</p>|Dependent item|proxmox.node.tasks.failed.latest[{#NODE.NAME}]|

### Trigger prototypes for Node discovery

|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Proxmox VE: Node [{#NODE.NAME}] offline|<p>Node offline.</p>|`last(/Proxmox VE by HTTP/proxmox.node.online[{#NODE.NAME}]) <> 1`|High||
|Proxmox VE: Node [{#NODE.NAME}] has been restarted|<p>Uptime is less than 10 minutes.</p>|`last(/Proxmox VE by HTTP/proxmox.node.uptime[{#NODE.NAME}])<10m`|Info|**Manual close**: Yes<br>**Depends on**:<br><ul><li>Proxmox VE: Node [{#NODE.NAME}] offline</li></ul>|
|Proxmox VE: Node [{#NODE.NAME}]: PVE manager has changed|<p>Firmware version has changed.</p>|`last(/Proxmox VE by HTTP/proxmox.node.pveversion[{#NODE.NAME}],#1)<>last(/Proxmox VE by HTTP/proxmox.node.pveversion[{#NODE.NAME}],#2)`|Info|**Manual close**: Yes|
|Proxmox VE: Node [{#NODE.NAME}]: Kernel version has changed|<p>Kernel version has changed.</p>|`last(/Proxmox VE by HTTP/proxmox.node.kernelversion[{#NODE.NAME}],#1)<>last(/Proxmox VE by HTTP/proxmox.node.kernelversion[{#NODE.NAME}],#2)`|Info|**Manual close**: Yes|
|Proxmox VE: Node [{#NODE.NAME}] high root filesystem space usage|<p>Root filesystem space usage.</p>|`min(...rootused,5m) / last(...roottotal) * 100 > {$PVE.ROOT.PUSE.MAX.WARN}`|Warning||
|Proxmox VE: Node [{#NODE.NAME}] high memory usage|<p>Memory usage.</p>|`min(...memused,5m) / last(...memtotal) * 100 > {$PVE.MEMORY.PUSE.MAX.WARN}`|Warning||
|Proxmox VE: Node [{#NODE.NAME}] high CPU usage|<p>CPU usage.</p>|`min(...cpu,5m) > {$PVE.CPU.PUSE.MAX.WARN}`|Warning||
|Proxmox VE: Node [{#NODE.NAME}] high swap space usage|<p>Swap usage.</p>|`min(...swapused,5m) / last(...swaptotal) * 100 > {$PVE.SWAP.PUSE.MAX.WARN}`|Warning||
|Proxmox VE: Node [{#NODE.NAME}]: Certificate expiring soon|<p>A certificate is expiring within the warning threshold.</p>|`last(...cert.notafter.soonest) < now() + {$PVE.CERT.EXPIRY.WARN}`|Warning||
|Proxmox VE: Node [{#NODE.NAME}]: Certificate expired|<p>A certificate has expired.</p>|`last(...cert.notafter.soonest) < now()`|Average||
|Proxmox VE: Node [{#NODE.NAME}]: SMART status fail|<p>One or more disks have a SMART health failure.</p>|`last(...disk.smart) <> 1`|High||
|Proxmox VE: Node [{#NODE.NAME}]: Replication error|<p>One or more replication jobs have errors.</p>|`last(...replication.errors) > 0`|High||
|Proxmox VE: Node [{#NODE.NAME}]: ZFS pool degraded|<p>One or more ZFS pools not in ONLINE state.</p>|`last(...zfs.pool.unhealthy) > 0`|High||
|Proxmox VE: Node [{#NODE.NAME}]: Subscription not active|<p>Subscription is not active.</p>|`last(...subscription.status) <> "active"`|Info||
|Proxmox VE: Node [{#NODE.NAME}]: Many pending updates|<p>High number of pending package updates.</p>|`last(...apt.updates.count) > {$PVE.APT.UPDATES.WARN}`|Info||
|Proxmox VE: Node [{#NODE.NAME}]: Failed tasks in last 24h|<p>Failed tasks in the last 24 hours.</p>|`last(...tasks.failed.count24h) > 0`|Average||

### LLD rule Storage discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Storage discovery||Dependent item|proxmox.storage.discovery|

### Item prototypes for Storage discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Storage [{#NODE.NAME}/{#STORAGE.NAME}]: Type|<p>More specific type, if available.</p>|Dependent item|proxmox.node.plugintype[{#NODE.NAME},{#STORAGE.NAME}]|
|Storage [{#NODE.NAME}/{#STORAGE.NAME}]: Size|<p>Storage size in bytes.</p>|Dependent item|proxmox.node.maxdisk[{#NODE.NAME},{#STORAGE.NAME}]|
|Storage [{#NODE.NAME}/{#STORAGE.NAME}]: Content|<p>Allowed storage content types.</p>|Dependent item|proxmox.node.content[{#NODE.NAME},{#STORAGE.NAME}]|
|Storage [{#NODE.NAME}/{#STORAGE.NAME}]: Used|<p>Used disk space in bytes.</p>|Dependent item|proxmox.node.disk[{#NODE.NAME},{#STORAGE.NAME}]|

### Trigger prototypes for Storage discovery

|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Proxmox VE: Storage [{#NODE.NAME}/{#STORAGE.NAME}] high filesystem space usage|<p>Storage space usage.</p>|`min(...disk,5m) / last(...maxdisk) * 100 > {$PVE.STORAGE.PUSE.MAX.WARN}`|Warning||

### LLD rule QEMU discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|QEMU discovery||Dependent item|proxmox.qemu.discovery|

### Item prototypes for QEMU discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|VM [{#NODE.NAME}/{#QEMU.NAME} ({#QEMU.ID})]: CPU usage|<p>CPU load.</p>|Dependent item|proxmox.qemu.cpu[{#QEMU.ID}]|
|VM [{#NODE.NAME}/{#QEMU.NAME} ({#QEMU.ID})]: Memory usage|<p>Used memory in bytes.</p>|Dependent item|proxmox.qemu.mem[{#QEMU.ID}]|
|VM [{#NODE.NAME}/{#QEMU.NAME} ({#QEMU.ID})]: Memory total|<p>Total memory in bytes.</p>|Dependent item|proxmox.qemu.maxmem[{#QEMU.ID}]|
|VM [{#NODE.NAME}/{#QEMU.NAME} ({#QEMU.ID})]: Disk write, rate|<p>Disk write.</p>|Dependent item|proxmox.qemu.diskwrite[{#QEMU.ID}]|
|VM [{#NODE.NAME}/{#QEMU.NAME} ({#QEMU.ID})]: Disk read, rate|<p>Disk read.</p>|Dependent item|proxmox.qemu.diskread[{#QEMU.ID}]|
|VM [{#NODE.NAME}/{#QEMU.NAME} ({#QEMU.ID})]: Incoming data, rate|<p>Incoming data rate.</p>|Dependent item|proxmox.qemu.netin[{#QEMU.ID}]|
|VM [{#NODE.NAME}/{#QEMU.NAME} ({#QEMU.ID})]: Outgoing data, rate|<p>Outgoing data rate.</p>|Dependent item|proxmox.qemu.netout[{#QEMU.ID}]|
|VM [{#NODE.NAME}/{#QEMU.NAME}]: Get data|<p>Get VM status data.</p>|HTTP agent|proxmox.qemu.get.data[{#QEMU.ID}]|
|VM [{#NODE.NAME}/{#QEMU.NAME} ({#QEMU.ID})]: Uptime|<p>System uptime.</p>|Dependent item|proxmox.qemu.uptime[{#QEMU.ID}]|
|VM [{#NODE.NAME}/{#QEMU.NAME} ({#QEMU.ID})]: Status|<p>Status of Virtual Machine.</p>|Dependent item|proxmox.qemu.vmstatus[{#QEMU.ID}]|

### Trigger prototypes for QEMU discovery

|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Proxmox VE: VM [{#NODE.NAME}/{#QEMU.NAME} ({#QEMU.ID})] high memory usage|<p>Memory usage.</p>|`min(...mem,5m) / last(...maxmem) * 100 > {$PVE.VM.MEMORY.PUSE.MAX.WARN}`|Warning||
|Proxmox VE: VM [{#NODE.NAME}/{#QEMU.NAME} ({#QEMU.ID})] high CPU usage|<p>CPU usage.</p>|`min(...cpu,5m) > {$PVE.VM.CPU.PUSE.MAX.WARN}`|Warning||
|Proxmox VE: VM [{#NODE.NAME}/{#QEMU.NAME}] has been restarted|<p>Uptime is less than 10 minutes.</p>|`last(...uptime)<10m`|Info|**Manual close**: Yes<br>**Depends on**:<br><ul><li>VM Not running</li></ul>|
|Proxmox VE: VM [{#NODE.NAME}/{#QEMU.NAME} ({#QEMU.ID})]: Not running|<p>VM state is not "running" (only fires when onboot=1).</p>|`last(...vmstatus)<>"running" and last(...onboot)=1`|Average||

### LLD rule LXC discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|LXC discovery||Dependent item|proxmox.lxc.discovery|

### Item prototypes for LXC discovery

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|LXC [{#NODE.NAME}/{#LXC.NAME}]: Get data|<p>Get LXC status data.</p>|HTTP agent|proxmox.lxc.get.data[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME}]: Get config|<p>Get LXC configuration data.</p>|HTTP agent|proxmox.lxc.get.config[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: Start at boot|<p>Whether the container starts during node boot.</p>|Dependent item|proxmox.lxc.onboot[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: Uptime|<p>System uptime.</p>|Dependent item|proxmox.lxc.uptime[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: Status|<p>Status of LXC container.</p>|Dependent item|proxmox.lxc.vmstatus[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: CPU usage|<p>CPU load.</p>|Dependent item|proxmox.lxc.cpu[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: Memory usage|<p>Used memory in bytes.</p>|Dependent item|proxmox.lxc.mem[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: Memory total|<p>Total memory in bytes.</p>|Dependent item|proxmox.lxc.maxmem[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: Disk space usage|<p>Disk space usage.</p>|Dependent item|proxmox.lxc.disk[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: Disk space total|<p>Total disk space.</p>|Dependent item|proxmox.lxc.maxdisk[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: Disk write, rate|<p>Disk write.</p>|Dependent item|proxmox.lxc.diskwrite[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: Disk read, rate|<p>Disk read.</p>|Dependent item|proxmox.lxc.diskread[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: Incoming data, rate|<p>Incoming data rate.</p>|Dependent item|proxmox.lxc.netin[{#LXC.ID}]|
|LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: Outgoing data, rate|<p>Outgoing data rate.</p>|Dependent item|proxmox.lxc.netout[{#LXC.ID}]|

### Trigger prototypes for LXC discovery

|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Proxmox VE: LXC [{#NODE.NAME}/{#LXC.NAME}] has been restarted|<p>Uptime is less than 10 minutes (only fires when onboot=1).</p>|`last(...uptime)<10m and last(...onboot)=1`|Info|**Manual close**: Yes<br>**Depends on**:<br><ul><li>LXC Not running</li></ul>|
|Proxmox VE: LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: Not running|<p>LXC state is not "running" (only fires when onboot=1).</p>|`last(...vmstatus)<>"running" and last(...onboot)=1`|Average||
|Proxmox VE: LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})]: high disk space usage|<p>Disk space usage.</p>|`min(...disk,5m) / last(...maxdisk) * 100 > {$PVE.LXC.DISK.PUSE.MAX.WARN}`|Warning||
|Proxmox VE: LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})] high memory usage|<p>Memory usage.</p>|`min(...mem,5m) / last(...maxmem) * 100 > {$PVE.LXC.MEMORY.PUSE.MAX.WARN}`|Warning||
|Proxmox VE: LXC [{#NODE.NAME}/{#LXC.NAME} ({#LXC.ID})] high CPU usage|<p>CPU usage.</p>|`min(...cpu,5m) > {$PVE.LXC.CPU.PUSE.MAX.WARN}`|Warning||

## Support

Issues and changes specific to this fork should be tracked in this repository.

If a problem is reproducible with the unmodified upstream template, you can also review the upstream resources:

- [Zabbix repository](https://github.com/zabbix/zabbix)
- [Zabbix support](https://support.zabbix.com)
- [Zabbix forums](https://www.zabbix.com/forum/zabbix-suggestions-and-feedback)

## License

The original template content comes from the Zabbix project and remains subject to the upstream `AGPL-3.0-only` license. Modifications in this repository are distributed under the same license unless stated otherwise.
