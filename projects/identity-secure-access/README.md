# Identity & Secure Remote Access

## Overview

This project explores the integration of Authentik and Pangolin in my personal homelab, with a focus on centralized authentication, remote access, and troubleshooting across multiple infrastructure components.

Before starting this project, I already had practical experience with domains, DNS, HTTPS, TLS certificates, and reverse proxies. I had used these technologies to make self-hosted applications accessible, including an HTTPS setup for Immich. I had also previously used a WireGuard connection to my FRITZ!Box and an authenticated Home Assistant entry point to access selected services remotely.

My motivation for trying Authentik and Pangolin was primarily curiosity and hands-on learning. I had heard about both technologies, particularly their integration, and wanted to understand what they could offer beyond my existing setup.

The current deployment is operational for its intended test: Pangolin is publicly reachable through a domain, and its Authentik login flow has been successfully tested from outside my home networks. Pangolin is not yet the access gateway for my other homelab applications.

The longer-term objective is to move the public access layer away from my home networks and onto external VPS infrastructure, while retaining private connectivity to selected homelab services.

## 1. Goals and Motivation

My existing password-management workflow with Vaultwarden already works well for individual application logins. Single sign-on was therefore an interesting potential benefit, but not the main reason for this project.

The main goals are to:

- Explore Authentik and Pangolin through a real deployment.
- Understand how Authentik can act as an identity provider for Pangolin.
- Configure and test an OpenID Connect (OIDC) authentication flow.
- Apply my existing DNS, TLS, and reverse proxy experience to a different access architecture.
- Investigate how external access to self-hosted services can be managed more deliberately.
- Prepare to move the public access layer away from the homelab.
- Reduce the need for inbound port forwarding on my home router for the planned access path.

The intended design should allow users to access selected applications through public domains without connecting directly to my home internet connection.

This is an architectural goal, not a claim that the home IP address would be invisible under all circumstances or that the resulting setup would be automatically secure.

## 2. Current Architecture

My homelab currently spans two physical locations connected through Tailscale. During the transition to the newer NAB9 host, public inbound traffic still enters through the original network.

The current external access path to Pangolin is:

**Internet → Cloudflare → Nginx Proxy Manager at the original site → Tailscale connection → Pangolin/Traefik on the NAB9**

Authentik also runs on the NAB9 and handles authentication for Pangolin.

| Component | Current role |
|---|---|
| Authentik | Identity provider for Pangolin |
| Pangolin | Publicly reachable remote-access platform under evaluation |
| Traefik | Reverse proxy component associated with the Pangolin deployment |
| Nginx Proxy Manager | Existing public-facing reverse proxy at the original site |
| Tailscale | Private connection between the two homelab locations |
| Cloudflare | Part of the public domain access and certificate-validation setup |

This is a transitional architecture. Pangolin is not yet configured to publish and protect my other homelab services.

For the wider two-site network design and alternative access paths, see the [Multi-Site Tailscale project](../multi-site-tailscale/README.md).

## 3. Authentik and Pangolin Integration

I deployed Authentik and Pangolin in separate virtualized environments on the NAB9 and configured Authentik as the identity provider for Pangolin.

The main new learning area was the OIDC integration: understanding how the two applications communicate during authentication and how their configuration affects the login and return flow.

### Tested external login flow

I successfully tested the following sequence from outside my home networks, using the public domain rather than a private network address:

1. Open Pangolin using its public domain.
2. Get redirected to Authentik.
3. Sign in through Authentik.
4. Return to Pangolin after successful authentication.

This confirms that the external authentication and return flow works for Pangolin.

It does not establish that additional applications are already protected by Pangolin, that every failure scenario has been tested, or that the deployment has undergone a security audit.

## 4. Troubleshooting and Lessons Learned

The most demanding part of this project was getting several individually familiar technologies to work together in a new architecture.

The difficulties were distributed across identity configuration, DNS resolution, certificate issuance, reverse proxy routing, and connectivity between the two homelab locations.

### Distinguishing application availability from integration failures

An internal request to Authentik returned an HTTP 302 redirect to its login page.

This demonstrated that Authentik itself was responding, even while the wider authentication and proxy setup was not yet working as intended.

The important distinction was between an available application and a functioning end-to-end authentication flow.

### Clarifying the proxy topology

The initial setup involved multiple reverse proxy components, services at two locations, and a Tailscale connection between them.

I had to establish which component received incoming traffic, where it forwarded requests, and how Pangolin and Authentik could reach each other.

An incorrect endpoint was identified and corrected during troubleshooting. Connectivity between the two homelab networks was subsequently verified.

### Resolving DNS problems

The Pangolin environment initially used a Tailscale DNS resolver.

DNS resolution problems interfered with reliable access to external resources required by the Traefik setup, including plugin retrieval.

The resolver configuration was changed to use external DNS resolvers.

This was not my first experience configuring DNS. The lesson was how resolver selection inside one component could affect a larger application deployment.

### Fixing TLS certificate issuance

Traefik initially presented its default certificate instead of the intended trusted certificate.

HTTP-01 certificate validation did not work reliably in the existing Cloudflare and proxy arrangement. The setup was changed to Cloudflare DNS-01 validation.

A missing or empty Cloudflare DNS API token in the service configuration also had to be corrected.

After these changes, Traefik obtained a valid Let's Encrypt certificate.

I had worked with HTTPS and certificates before this project. The new challenge was adapting certificate issuance to this particular combination of Cloudflare, Traefik, and the transitional proxy architecture.

### Investigating external gateway errors

During setup, the public access path intermittently returned HTTP 504 errors even though individual internal components responded.

Troubleshooting required checking the route across Cloudflare, Nginx Proxy Manager, the inter-site connection, Pangolin/Traefik, and Authentik rather than relying on a single internal connectivity test.

The later successful external login demonstrates that the complete access path works for the tested authentication scenario. It is not a claim of continuously measured availability.

### Completing the OIDC login flow

The Authentik integration initially required further OIDC configuration work.

Possible client-configuration issues were investigated, but the exact cause of the earlier failure was not conclusively recorded. I therefore do not present a suspected client ID, client secret, or client type issue as a confirmed root cause.

The current result is verified: opening Pangolin through its public domain redirects to Authentik, and successful authentication returns the browser to Pangolin.

## 5. Planned Architecture

The current Pangolin deployment on the NAB9 is a learning and transitional setup, not the intended permanent hosting arrangement.

The long-term goal is to separate the public access layer from the homelab by moving the relevant components to external VPS infrastructure.

### Intended access path

**User → public domain / Cloudflare → Pangolin and Traefik on VPS infrastructure → Tailscale → selected homelab application**

Authentik is also intended to run externally and provide authentication for the access platform.

Cloudflare remains part of the planned public access path. The VPS infrastructure would provide the public-facing entry point, while Tailscale would connect it to the homelab without requiring inbound web port forwarding on the home router for this access path.

The intention is to avoid exposing the home internet connection directly to visitors of the published applications and to reduce unnecessary public exposure of the home IP address.

This does not guarantee complete IP-address anonymity. The VPS provider and relevant network services may still have visibility into connection information.

### Hosting decisions still open

The following decisions have not yet been finalized:

- Whether Authentik and Pangolin will share a VPS or run on separate external systems.
- The exact deployment and routing design for the VPS-based architecture.
- Which homelab applications will be published through Pangolin.
- Which access controls and validation procedures will be required before the new architecture replaces the current setup.

The existing Nginx Proxy Manager arrangement is intended to be replaced, but it remains part of the current deployment.

### Independent monitoring and alerting

I also intend to introduce externally hosted monitoring and alerting.

The purpose is to retain an independent way to detect and report outages, including a complete power or internet outage at home.

Uptime Kuma is one possible solution, but the monitoring product has not been selected. The exact hosting arrangement is also still open.

External monitoring and alerting are planned, not implemented as part of this project.

## 6. Current Status

| Capability | Status |
|---|---|
| Existing experience with DNS, HTTPS, TLS, and reverse proxies | Established before this project |
| Authentik deployed on the NAB9 | Implemented |
| Pangolin deployed on the NAB9 | Implemented |
| Pangolin reachable through a public domain | Tested |
| External Authentik login and return to Pangolin | Tested |
| Other homelab applications published and protected through Pangolin | Planned |
| Replacement of the existing Nginx Proxy Manager setup | Planned |
| Pangolin and Authentik hosted externally | Planned; VPS allocation undecided |
| VPS-to-homelab access through Tailscale | Planned |
| External monitoring and alerting | Planned; product not selected |

## Key Takeaway

This project extends my existing experience with DNS, HTTPS, certificates, and reverse proxies into identity integration and a more complex remote-access architecture.

The central learning experience has been understanding and troubleshooting the complete authentication path: an available application, correct DNS resolution, valid TLS certificates, working proxy routing, and a successful OIDC login are separate requirements that need to be verified individually.

The result so far is a working, externally tested Authentik login for Pangolin. The next architectural step is to move the public access and identity components onto external VPS infrastructure while retaining Cloudflare and using Tailscale for private connectivity to the homelab.