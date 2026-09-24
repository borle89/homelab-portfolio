# Proxmox Homelab & Service Migration

## Overview

This project documents the evolution of my personal homelab from a laptop-based Proxmox environment to a more capable mini-PC platform.

The homelab provides a practical environment for learning infrastructure engineering through real services, virtualization, containers, networking, storage, and operational troubleshooting. Rather than rebuilding everything at once, I am approaching the transition as a staged migration: document the existing environment, establish the new platform, validate selected workloads, and move the remaining services only when their dependencies are understood.

The project is ongoing. This page distinguishes the infrastructure already deployed and tested from the migration and backup work still planned.

## 1. Background and Goals

My original homelab runs on an HP Omen laptop with an Intel Core i7-6700. It has provided a useful platform for learning Proxmox and self-hosting, but I want to return the laptop to everyday Linux use once its infrastructure role is no longer needed.

I selected a Minisforum NAB9 as the new primary Proxmox host. Its Intel Core i9-12900HK offers more processing capacity and additional cores and threads for running multiple virtual machines and containers. The system also provides dual 2.5 GbE interfaces for a future network upgrade.

The main goals are to:

- Establish the NAB9 as the primary virtualization host.
- Separate workloads into appropriate virtual machines and LXC containers.
- Migrate services incrementally instead of relying on a single large cutover.
- Preserve existing media data and filesystems during the storage move.
- Improve backup and recovery arrangements as a separate, verifiable stage.
- Build practical experience with infrastructure documentation, dependencies, migration planning, and validation.

The NAB9 currently has 32 GB of RAM. An upgrade to 64 GB is a future consideration, not part of the completed migration.

## 2. Infrastructure Architecture

The environment currently spans two physical locations connected through Tailscale.

**Network addressing lesson:** Both locations initially used the same private IP subnet because their home routers had the same default network configuration. This created overlapping address ranges when connecting the sites through Tailscale. I changed the subnet at one location so that each site has a distinct address range and cross-site routing works without this conflict.

| Component | Current role |
|---|---|
| Minisforum NAB9 | New primary Proxmox host and destination for selected workloads |
| HP Omen laptop | Existing Proxmox host, still running services that have not been migrated |
| Tailscale | Private connectivity between the two locations |
| Existing media drives | Media storage currently attached to the original environment |

On the NAB9, Proxmox hosts both virtual machines and LXC containers. Docker runs inside a dedicated virtual machine, providing a separate environment for Compose-based applications.

A 1 TB SSD connected to the NAB9 via USB is used as the destination for scheduled Proxmox backups. This is a local backup arrangement, separate from the planned Proxmox Backup Server deployment.

This is an incremental transition, not a completed replacement of the original host. Both systems remain relevant while workloads and storage are moved.

For the separately documented network architecture and fallback connectivity, see the [Multi-Site Tailscale project](../multi-site-tailscale/README.md).

## 3. Workloads on the NAB9

The NAB9 already hosts a mix of infrastructure services and applications.

### Virtual machines

| VM | Purpose |
|---|---|
| Home Assistant | New Home Assistant environment; not yet the productive instance |
| Docker | Dedicated host for containerized applications |
| Nextcloud | Nextcloud environment |
| Authentik | Identity and access management environment |

### LXC containers

| Container | Purpose | Current state |
|---|---|---|
| Pangolin | Remote-access infrastructure | Running |
| Jellyfin | Media server | Running; selected playback tested |
| Immich | Photo management | Stopped |
| Frigate | Video processing | Stopped |
| Tailscale | Site connectivity | Running |

A running VM or container does not, by itself, demonstrate that its application is fully configured or operational. The table records the guest inventory, not a blanket claim that every service is production-ready.

### Docker applications

The dedicated Docker VM currently contains nine running containers, covering:

- Wanderer, including its application, database, and search components.
- AdventureLog and its PostGIS database.
- Nginx Proxy Manager.
- Mosquitto MQTT broker.
- Portainer.
- Vaultwarden.

The Docker environment uses both bind mounts and named Docker volumes for persistent application data. These storage dependencies have been identified as part of the current inventory, but a complete restore test has not yet been performed.

## 4. Migration Approach

The migration is being organized around dependencies and validation rather than simply moving containers from one host to another.

### Phase 1: Establish and document the new platform

**Status: In progress**

The NAB9 is running Proxmox with separate VMs and LXC containers. Its existing workloads and the Docker VM's persistent storage locations have been inventoried.

The original HP Omen environment remains available while the new host is built and tested.

### Phase 2: Validate selected services across locations

**Status: Partially completed**

Jellyfin has been installed on the NAB9 and tested against media storage still located at the original site. The connection uses the existing Tailscale link and an SMB share.

Movie playback was successfully tested. This demonstrates a working cross-site access path for that scenario, but it does not establish that every Jellyfin feature, media format, or failure scenario has been validated.

The SMB connection is a transitional arrangement, not the intended permanent media-storage architecture.

### Phase 3: Migrate remaining workloads and storage

**Status: Planned**

The productive Home Assistant instance and remaining media and automation services still need to move from the HP Omen environment.

The existing media drives are intended to move physically to the NAB9 and connect through USB enclosures. The goal is to preserve the existing data and filesystems rather than rebuild the media library from scratch.

The migration will need to account for application configuration, persistent data, service dependencies, and post-migration functional checks.

### Phase 4: Establish a dedicated backup role

**Status: Planned**

After the required workloads have been migrated, I intend to use the HP Omen temporarily as a Proxmox Backup Server.

**Proxmox Backup Server is not currently installed.** A successful backup configuration or tested restore must not be inferred from this plan.

The longer-term goal is dedicated backup hardware, allowing the HP Omen to return to use as a Linux laptop. A separate storage server is also a future consideration, not part of the current architecture.

## 5. Practical Experience and Validation

This project has provided hands-on experience with:

| Area | Work performed |
|---|---|
| Virtualization | Deploying and organizing workloads across Proxmox VMs and LXC containers |
| Containerization | Operating a dedicated Docker VM with multiple applications |
| Persistent storage | Identifying Docker bind mounts and named volumes relevant to backup and migration |
| Networking | Connecting services across two locations through Tailscale |
| File sharing | Accessing remote media through SMB during the transition |
| Functional testing | Confirming selected Jellyfin movie playback from the new host |
| Migration planning | Separating completed work from dependencies, risks, and future tasks |

These are practical activities carried out in my own environment. They should not be read as claims of enterprise-scale operations, completed disaster-recovery testing, or a finished migration.

## 6. Current Status and Next Steps

The NAB9 is operational and already runs a range of virtualized and containerized workloads. The original HP Omen remains in service because the migration is not complete.

The next project milestones are:

1. Complete the documentation needed for the actual migration.
2. Move the remaining required services and their persistent data.
3. Relocate the existing media drives while preserving their data.
4. Validate migrated applications and their dependencies.
5. Implement the planned backup environment and perform restore tests.
6. Retire the HP Omen from its primary infrastructure role.

## Key Takeaway

The main learning objective is not simply to run more self-hosted applications. It is to understand how infrastructure components depend on one another, document the current state, make deliberate architectural choices, and verify changes before treating a migration as complete.