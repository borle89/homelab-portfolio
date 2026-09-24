# Homelab Portfolio

**Linux · Networking · Virtualization · Infrastructure Operations**

Hi, I'm Gregor. I'm building practical skills in Linux, networking, virtualization and infrastructure operations through my personal homelab.

My goal is to move into a professional IT infrastructure role and continue developing towards cloud infrastructure engineering.

This repository documents selected projects, technical decisions, troubleshooting experiences and lessons learned. It focuses on work I have actually carried out, rather than presenting planned technologies as completed skills.

---

## Homelab Overview

My homelab spans two separate sites approximately 50 km apart.

| Site | Device | Role |
| --- | --- | --- |
| **Site A — NAB9 network** | **Minisforum NAB9** | The newer main server, hosting virtual machines, LXC containers and infrastructure services. |
| Site A | **Fire TV Stick** | Provides an alternative Tailscale access path if the NAB9 is unavailable. |
| **Site B — HP Omen network** | **HP Omen Laptop** | My original Proxmox learning environment, which continues to host services and storage during the transition to the NAB9. |
| Site B | **Raspberry Pi** | Runs smaller self-hosted services and provides an alternative Tailscale access path if the laptop is unavailable. It is also planned to become an independent monitoring device. |

The two sites are connected through Tailscale subnet routers, enabling cross-site access to devices and SMB storage shares.

I use this environment to learn how infrastructure components work together, investigate failures and improve existing setups.

---

## Featured Projects

### 01 · [Proxmox Homelab & Service Migration](projects/proxmox-homelab/README.md)

Built Proxmox environments on a laptop and later on a dedicated Minisforum server. Created and configured virtual machines and LXC containers, allocated resources, configured storage and set up scheduled backups.

The transition between the two systems is ongoing. It includes preparing services on the new server while existing applications and data remain available on the original system.

**Technologies:** Proxmox VE · Linux · virtual machines · LXC · storage · SMB/CIFS · backups

---

### 02 · [Multi-Site Networking with Tailscale](projects/multi-site-tailscale/README.md)

Connected two local networks approximately 50 km apart using Tailscale subnet routers.

Each network has a primary subnet router and an additional device providing an alternative access path:

| Network | Primary subnet router | Alternative access path |
| --- | --- | --- |
| **Site A — NAB9 network** | Tailscale LXC on the Minisforum NAB9 | Fire TV Stick |
| **Site B — HP Omen network** | Tailscale container on the Proxmox laptop | Raspberry Pi |

The setup enables cross-site access to devices and SMB storage shares.

> **Real-world troubleshooting:** When the NAB9 became unreachable due to a failed power supply, the Fire TV Stick maintained access to the local network. This helped isolate the issue to the server rather than the entire site connection. On-site troubleshooting and a multimeter measurement identified the faulty power supply.

The Fire TV Stick required an additional application to remain awake and available as a subnet router.

**Technologies:** Tailscale · Linux · subnet routing · IP networking · SMB/CIFS · hardware troubleshooting

---

### 03 · [Identity & Secure Access](projects/identity-secure-access/README.md)

Deployed Authentik and Pangolin in separate virtualized environments and configured Authentik for use with Pangolin.

Troubleshooting involved reverse proxy routing, DNS resolution, Cloudflare DNS-01 certificate validation and OIDC configuration. The initial deployment and service installation were carried out independently; I used AI assistance while learning the identity and access configuration.

Authentik currently serves Pangolin. Extending it to additional homelab services is planned.

**Technologies:** Authentik · Pangolin · Traefik · Cloudflare · DNS · TLS · OIDC

---

## Additional Hands-On Experience

| Area | Experience |
| --- | --- |
| **Docker and Docker Compose** | Running and managing self-hosted services in a Debian virtual machine. |
| **Jellyfin** | Configured an LXC container, tested Intel hardware transcoding and accessed media through a cross-site SMB share. |
| **Home Assistant** | Operating an existing smart home installation and preparing a separate environment for a future rebuild. |
| **Nextcloud AIO** | Installed a test deployment using the project's documentation to evaluate it for personal use. |
| **Raspberry Pi** | Running a Tailscale fallback subnet router and smaller self-hosted services. A future role as an independent monitoring and management device is planned. |

---

## Current Focus

I am continuing to develop my Linux administration, networking and infrastructure skills while documenting selected homelab projects. My longer-term learning path includes automation, cloud infrastructure and related engineering practices.

I am also planning a network infrastructure upgrade focused on network segmentation, faster internal connectivity and centralized network management. The design and implementation will be documented as a separate project.

This portfolio is a work in progress. Project pages will be expanded with architecture explanations, implementation details and troubleshooting notes as they are reviewed and documented.
