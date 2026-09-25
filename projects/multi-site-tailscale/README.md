# Multi-Site Networking with Tailscale

[← Portfolio overview](../../README.md)

**Two sites · Subnet routing · Cross-site access · Independent fallback devices**

A practical networking project connecting two separate homelab locations.

The setup provides remote access to servers, network devices and selected services. Each site also has an independent Tailscale device that can maintain network access when its primary Proxmox host is unavailable.

## Project Overview

| | Site A — NAB9 network | Site B — HP Omen network |
|---|---|---|
| Primary Proxmox host | Minisforum NAB9 | HP Omen Laptop |
| Primary Tailscale instance | Dedicated Proxmox LXC container | Dedicated Docker container inside a VM running on Proxmox |
| Alternative access device | Fire TV Stick | Raspberry Pi |
| Fallback implementation | Tailscale on Fire TV Stick | Tailscale in Docker |
| Fallback verification | Actual NAB9 outage | HP Omen deliberately shut down |

**Status:** Operational. Automatic fallback to the independent subnet router was observed at both sites without manual route changes.

---

## 1. Background & Goal

Before adopting Tailscale, I used a WireGuard connection through my FRITZ!Box for remote access to my original homelab.

I later switched to Tailscale because I wanted more flexibility in how I made individual devices and services accessible. Initially, I used it primarily to reach Home Assistant and administer services while away from home.

When I added a Minisforum NAB9 at a second location, I needed a way to access devices at the new site from my existing Home Assistant installation.

I started with targeted access to a solar battery system and a smart meter.

My first practical connectivity test involved checking whether a Govee light remained visible in its app and controllable through Home Assistant.

After confirming that the devices I needed were reachable, I expanded the Tailscale subnet route to cover the new site's local network. This made it possible to access the server, router and individual services without configuring a separate route for each device.

I later expanded access at the original site as well.

What began as remote access to individual services gradually developed into a multi-site networking project involving subnet routing, cross-site storage access and alternative access paths for server outages.

---

## 2. Architecture

The two sites use separate local networks. Tailscale subnet routing provides the access paths described below. The documented tests cover remote administration and cross-site SMB access; other traffic paths have not yet been documented.

### Site A — NAB9 network

| Component | Role |
|---|---|
| Minisforum NAB9 | Main Proxmox host |
| Dedicated Proxmox LXC container | Primary Tailscale subnet router |
| Fire TV Stick | Independent alternative Tailscale access path |
| Local devices and services | Accessible through the advertised subnet route |

### Site B — HP Omen network

| Component | Role |
|---|---|
| HP Omen Laptop | Original Proxmox host and storage location |
| Dedicated Tailscale Docker container inside the Docker VM hosted on Proxmox | Primary Tailscale subnet router |
| Raspberry Pi | Independent alternative Tailscale access path |
| SMB storage shares | Used for cross-site media access |

**Architecture note:** The fallback devices operate independently of their respective Proxmox hosts. At each site, the primary and alternative subnet router advertise the same /24 route; the two sites use different prefixes. During the NAB9 power supply failure and the deliberate HP Omen shutdown, access to the respective site remained available without manual route changes.

---

## 3. Implementation

### Proxmox and Tailscale

On the NAB9, I deployed Tailscale in a dedicated Proxmox LXC container. On the HP Omen, I deployed it as a dedicated Docker container inside a VM running on Proxmox. I installed Tailscale using its documentation and enabled the required subnet routes.

The setup developed incrementally:

1. Started with remote access to the original homelab.
2. Made selected devices at the new site accessible, including the solar battery system and smart meter.
3. Tested access to a Govee light through its app and Home Assistant.
4. Expanded the advertised route to the new site's local network.
5. Extended subnet routing to the original site's local network.

This allowed me to move from access to selected devices towards broader access to the resources I needed at both sites.

### Cross-Site Storage Test

After establishing the network connection, I installed Jellyfin on the NAB9 and tested access to media stored at the original site.

The NAB9 accessed the media through an SMB share across the Tailscale-connected networks.

This was a practical test of the connection beyond remote administration: a service at one site could use storage located at the other.

The Jellyfin migration is still in progress. This test does not mean that all media libraries or services have already moved to the NAB9.

---

## 4. Alternative Access Paths

I wanted to retain access to each site's local network even if its primary Proxmox host became unavailable.

### Site A: Fire TV Stick

The Fire TV Stick was my first fallback device.

At the time, the NAB9 was the only available server at the new site. I did not have another computer or a freely configurable router that could provide an independent Tailscale connection.

I therefore configured Tailscale on a Fire TV Stick as an alternative subnet router.

**Practical issue:** The Fire TV Stick entered sleep mode, making it unsuitable as an always-available access device.

I installed an additional Android application through a sideloading process using my smartphone to keep the device awake.

The Fire TV Stick later proved useful during an actual NAB9 outage.

### Site B: Raspberry Pi

The Raspberry Pi was added later in the project.

It already ran smaller self-hosted services and remained powered independently of the HP Omen laptop. I deployed Tailscale in a dedicated Docker container to provide an alternative access path if the laptop became unavailable.

**Fallback test:** After installing Tailscale on the Raspberry Pi, I shut down the HP Omen laptop and verified that I could still access the site's network through the Raspberry Pi. I then restarted the laptop.

This confirmed that the alternative access path remained usable without the primary Proxmox host.

The Raspberry Pi is also planned to take on an independent monitoring role in the future.

---

## 5. Real-World Troubleshooting: NAB9 Power Supply Failure

The alternative access design became useful when the NAB9 unexpectedly became unreachable.

### Symptoms

- The NAB9 was no longer reachable remotely.
- Services hosted on the NAB9 were unavailable.
- The cause was initially unknown.

### Investigation

The Fire TV Stick still provided access to the site's local network.

This helped me establish that the site itself remained reachable while the NAB9 was missing from the network.

Because I could not recover the server remotely, I investigated the hardware on site.

After several checks, I used a multimeter to test the power supply and identified the faulty power adapter as the cause of the server failure.

### Outcome

The Fire TV Stick maintained an independent access path during the outage.

The incident demonstrated the value of having a network access device that does not depend on the primary server remaining operational.

It also showed the limits of remote access: an alternative network connection can help identify and narrow down a problem, but it cannot repair failed hardware.

---

## 6. Lessons Learned

| Area | Practical lesson |
|---|---|
| Subnet routing | Tailscale can provide access to local network resources without installing it on every individual device. |
| Incremental implementation | Starting with selected devices made it possible to test the setup before expanding access to entire local networks. |
| Independent access | A fallback device should remain available when the primary server is powered off or fails. |
| Device behavior | Consumer hardware may require additional configuration to remain suitable for an always-available infrastructure role. |
| Troubleshooting | Continued access through the Fire TV Stick helped distinguish a server outage from loss of access to the entire site. |
| Cross-site services | The Jellyfin and SMB test demonstrated that a service at one site could access storage at the other. |

---

## 7. Verified Failover and Open Tests

Both subnet routers at each site advertise the same /24 route. The alternative device maintained network access automatically when the corresponding Proxmox host became unavailable: once during the unexpected NAB9 power supply failure and once when the HP Omen was deliberately shut down. No manual route change was required in either test.

The tested access paths are remote administration and cross-site SMB storage access. Initiating connections from ordinary devices without Tailscale in both directions has not yet been documented. A route overview can be added later using anonymized example networks.

---

## 8. Current Status & Next Steps

**Implemented and verified**

- Tailscale subnet routing at both sites.
- Remote access to the network resources needed for administration.
- Cross-site access to an SMB media share.
- Fire TV Stick as an alternative access path during an actual NAB9 outage.
- Raspberry Pi as an alternative access path, verified by shutting down the HP Omen.

**Planned**

- Expand the Raspberry Pi's role into independent infrastructure monitoring.
- Continue the migration of services to the NAB9.
- Document the network architecture and operational procedures in more detail.

---

## Technologies

**Tailscale · WireGuard (previous remote-access setup) · Proxmox VE · Linux · LXC · Docker · Subnet Routing · SMB/CIFS · Home Assistant · Jellyfin · Raspberry Pi**
