# Homelab Portfolio

**Linux · Networking · Virtualization · Infrastructure Operations**

Hi, I'm Gregor. I'm building practical skills in Linux, networking, virtualization and infrastructure operations through my personal homelab.

My goal is to move into a professional IT infrastructure role and continue developing towards cloud infrastructure engineering.

This repository documents selected projects, technical decisions, troubleshooting experiences and lessons learned. It focuses on work I have actually carried out and distinguishes implemented or tested work from future plans.

---

## Homelab Overview

My homelab spans two separate sites.

| Site | Device | Role |
| --- | --- | --- |
| **Site A — NAB9 network** | **Minisforum NAB9** | Main Proxmox host for virtual machines, LXC containers and infrastructure services. |
| Site A | **Fire TV Stick** | Independent alternative Tailscale access path if the NAB9 is unavailable. |
| **Site B — HP Omen network** | **HP Omen Laptop** | Original Proxmox host, still running services and storage during the migration. |
| Site B | **Raspberry Pi** | Runs smaller self-hosted services and provides an independent alternative Tailscale access path. |

The environment is used to explore how virtualization, networking, storage and self-hosted services interact, while providing practical experience with troubleshooting and infrastructure changes.

---

## Featured Projects

### 01 · [Proxmox Homelab & Service Migration](projects/proxmox-homelab/README.md)

Built Proxmox environments on an HP Omen laptop and later on a dedicated Minisforum NAB9. The project covers virtual machines, LXC containers, storage, scheduled local Proxmox backups and the staged migration of services and data between the two systems.

**Status:** Migration in progress. Both Proxmox hosts remain in use while workloads and storage are moved and validated. A backup restore has not yet been tested.

**Technologies:** Proxmox VE · Linux · virtual machines · LXC · storage · SMB/CIFS · backups

---

### 02 · [Multi-Site Networking with Tailscale](projects/multi-site-tailscale/README.md)

Connected two local networks using Tailscale subnet routing to provide cross-site access to selected devices, services and SMB storage.

Each site also has an independent alternative Tailscale access device. The Fire TV Stick provided access during an actual NAB9 outage, while the Raspberry Pi path was verified by deliberately shutting down the HP Omen.

**Status:** Operational. Cross-site access and both alternative access paths have been tested in practice.

**Technologies:** Tailscale · Linux · subnet routing · IP networking · SMB/CIFS · hardware troubleshooting

---

### 03 · [Identity & Secure Access](projects/identity-secure-access/README.md)

Deployed Authentik and Pangolin in separate virtualized environments and integrated them using OpenID Connect (OIDC).

Troubleshooting covered reverse proxy routing, DNS resolution, Cloudflare DNS-01 certificate validation and the authentication flow. Pangolin is publicly reachable, and the Authentik login and return flow has been tested from outside the homelab networks.

**Status:** Initial integration tested. Extending the access architecture to additional homelab services and external VPS infrastructure is planned.

**Technologies:** Authentik · Pangolin · Traefik · Cloudflare · DNS · TLS · OIDC

---

## Additional Hands-On Experience

| Area | Experience |
| --- | --- |
| **Docker and Docker Compose** | Running and managing self-hosted services in a dedicated Debian virtual machine. |
| **Jellyfin** | Configured an LXC container and tested 4K-to-720p transcoding with media accessed through a cross-site SMB share. Jellyfin reported 188 fps during the test; this reading alone does not verify hardware acceleration. |
| **Home Assistant** | Operating an existing smart home installation and preparing a separate environment for a future rebuild. |
| **Nextcloud AIO** | Installed a test deployment using the project's documentation to evaluate it for personal use. |
| **Raspberry Pi** | Running smaller self-hosted services and an independent Tailscale access path. |

---

## Current Focus

I am continuing to develop my Linux administration, networking and infrastructure skills while documenting selected homelab projects.

Current work includes completing the staged migration to the NAB9 and preparing a network infrastructure upgrade focused on segmentation, faster internal connectivity and centralized management.

My longer-term learning path includes infrastructure automation, cloud infrastructure and related engineering practices.

This portfolio is a work in progress. Project pages are expanded as implementations, tests and troubleshooting experiences are reviewed and documented.
