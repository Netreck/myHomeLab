# Personal Homelab

Personal infrastructure project used to study virtualization, networking, observability, security, automation, and self-hosting while running services for my personal projects.

This repository documents the environment as it evolves. The current architecture prioritizes network segmentation, private administration, and a minimal public VPS footprint.

> [!NOTE]
> My ISP uses CGNAT. The VPS exists mainly to provide a public IPv4 entry point and is intentionally kept small and inexpensive. Compute workloads remain in the homelab.

---

## About Me

My name is **Gabriel Gonçalves**, and I study Computer Science at **UFABC**.

---

## Project Goals

* Run personal applications and infrastructure services locally.
* Practice network segmentation, routing, firewall policies, and private administration.
* Build hands-on experience with Proxmox, pfSense, LXC, VPNs, reverse proxies, observability, and automation.
* Keep public cloud costs low by using the VPS only as the public edge.
* Maintain the environment as a realistic platform engineering and security laboratory.

---

## Architecture Diagram

<img width="1045" height="867" alt="image" src="https://github.com/user-attachments/assets/3e9d6413-9c02-48e7-8f75-c32dd41763f8" />

The editable source is stored in [`docs/homelab.drawio`](docs/homelab.drawio).

The diagram represents a static snapshot of the environment. It is not generated automatically from the live infrastructure.

---

## Architecture Overview

The homelab runs on **Proxmox VE**, which hosts the pfSense virtual firewall and the LXC containers used by the platform.

Because the residential connection is behind CGNAT, public traffic first reaches a small VPS with public IPv4 address `201.23.79.145`. **Caddy**, running in Docker on the VPS, accepts HTTP/HTTPS traffic and forwards it through a direct **WireGuard** tunnel to the homelab.

The WireGuard tunnel terminates on `CT111 (wg-edge)` in the isolated EDGE network. Traffic then crosses **pfSense**, reaches **Nginx Proxy Manager** in the DMZ, and is forwarded through pfSense again to the authorized application in the SERVERS network.

Administrative access uses a separate path through **Tailscale**. My administrative device connects to `CT113 (tailscale-vpn)`, and pfSense controls access from the SERVERS network to the MGMT network. The Proxmox interface is available only at `10.10.30.2:8006` in MGMT.

### Public Traffic Flow

```text
gabriel-goncalves.com
        │
        ▼
Public VPS — 201.23.79.145
Docker Caddy — ports 80/443
        │
        ▼
WireGuard — UDP 51822
Overlay network — 10.255.255.0/30
        │
        ▼
CT111 wg-edge — EDGE 10.10.40.2
        │
        ▼
VM107 pfSense
        │
        ▼
CT103 Nginx Proxy Manager — DMZ 10.10.10.11
        │
        ▼
VM107 pfSense
        │
        ▼
Authorized application — SERVERS 10.10.20.0/24
```

### Private Administration Flow

```text
Mac / administrative device
        │
        ▼
Tailscale
        │
        ▼
CT113 tailscale-vpn — VPNADMIN 10.10.20.18
        │
        ▼
VM107 pfSense
        │
        ▼
MGMT 10.10.30.0/24
        │
        ▼
Proxmox VE — 10.10.30.2:8006
```

---

## Network Segmentation

pfSense is the central Layer 3 router and firewall between the internal networks. Proxmox bridges provide Layer 2 connectivity and do not route traffic between zones.

| Zone         | Proxmox bridge | Subnet            | Purpose                                                                   |
| ------------ | -------------- | ----------------- | ------------------------------------------------------------------------- |
| DMZ          | `dmznet`       | `10.10.10.0/24`   | Reverse proxy and public-facing internal services                         |
| SERVERS      | `srvnet`       | `10.10.20.0/24`   | Applications, monitoring, and internal workloads                          |
| MGMT         | `mgmtnet`      | `10.10.30.0/24`   | Proxmox and infrastructure management interfaces                          |
| EDGE         | `edgenet`      | `10.10.40.0/24`   | WireGuard entry point from the public VPS                                 |
| VPNADMIN     | `vpnadmin`     | `10.10.50.0/24`   | Reserved administrative VPN zone; |

Private networks use `.1` as their pfSense gateway.

---

## Services and Machines

### Virtual Machines

#### VM107 — pfSense

Central router and firewall for the segmented networks. It controls traffic between EDGE, DMZ, SERVERS, MGMT, VPNADMIN, and the upstream network.

### LXC Containers

| ID    | Name                | Network / IP            | Purpose                                                                | 
| ----- | ------------------- | ----------------------- | ---------------------------------------------------------------------- |
| CT100 | OpenClaw            | SERVERS — `10.10.20.17` | Application workload                                                   | 
| CT101 | NetBox              | MGMT — `10.10.30.10`    | Network source of truth and documentation                              | 
| CT103 | Nginx Proxy Manager | DMZ — `10.10.10.11`     | Internal reverse proxy; HTTP/HTTPS on `80/443`, administration on `81` | 
| CT104 | Main-Monitor        | SERVERS — `10.10.20.12` | Grafana, Loki, and Prometheus observability stack                      |
| CT108 | Portfolio-Prod      | SERVERS — `10.10.20.15` | Portfolio production workload                                          | 
| CT109 | Portfolio-Prod-1    | SERVERS — `10.10.20.16` | Additional portfolio production workload                               | 
| CT111 | wg-edge             | EDGE — `10.10.40.2`     | WireGuard endpoint for the VPS tunnel                                  | 
| CT112 | segment-pilot-web   | SERVERS — `10.10.20.10` | Network segmentation pilot workload                                    | 
| CT113 | tailscale-vpn       | VPNADMIN — `10.10.20.18` | Private administrative access through Tailscale                        | 

### Proxmox VE

* Management address: `10.10.30.2`
* Web interface: `https://10.10.30.2:8006`
* Management zone: MGMT
* No Proxmox host address is used on the old WAN / LEGACY network.

### ProxMenuX

Dashboard used for monitoring server logs, resource usage, hardware temperature, and memory allocation.

<img width="1546" height="970" alt="ProxMenuX dashboard" src="https://github.com/user-attachments/assets/f36f639f-05b4-40eb-a2c3-77d44d1df22e" />

### Local LAN Services

**Pi-hole** provides DNS filtering, ad blocking, and anti-tracking for selected devices on the residential LAN. It is not part of the segmented server traffic path documented above.

---

## Security Model

* The residential connection does not expose application ports directly to the internet.
* The public VPS provides the IPv4 entry point and forwards traffic through WireGuard.
* Public traffic enters the homelab through the isolated EDGE zone.
* Nginx Proxy Manager is isolated in the DMZ.
* Application backends remain in SERVERS.
* Proxmox and infrastructure management interfaces remain in MGMT.
* Administrative access is performed through Tailscale and routed by pfSense.
* Nginx Proxy Manager port `81` is intended for private administration only.
* Inter-zone access is controlled by pfSense rules and should be limited to required destinations and ports.

---

## Observability

`CT104 (Main-Monitor)` centralizes infrastructure monitoring and log visualization with:

* **Grafana** for dashboards and visualization.
* **Prometheus** for metrics collection and storage.
* **Loki** for centralized log aggregation.

---

## Technologies Used

* Proxmox VE
* pfSense
* LXC and virtual machines
* WireGuard
* Tailscale
* Docker
* Caddy
* Nginx Proxy Manager
* Cloudflare DNS
* NetBox
* Grafana
* Prometheus
* Loki
* Pi-hole

---

## Server Hardware

* **Motherboard:** X99 D4 Atermiter
* **CPU:** Intel Xeon E5-2680 v4, 14 cores / 28 threads
* **Memory:** 32 GB DDR4 ECC RDIMM, 2 × 16 GB
* **Storage:** 512 GB NVMe M.2 SSD
* **GPU:** NVIDIA GeForce RTX 3070 Ti
* **Power supply:** 750 W

---

## Planned Improvements

* **DDoS visibility:** evaluate FastNetMon Community for Layer 3/4 UDP and TCP detection and integrate telemetry with a personal machine-learning DDoS detection project.
* **Hardware expansion:** add another 32 GB of RAM and evaluate an RTX 3090 24 GB for local AI inference and training.
* **Recovery:** document and test recovery procedures for Proxmox and the virtualized pfSense dependency.

---

## Current Limitations

* The environment runs on a single Proxmox host and does not provide high availability.
* pfSense is virtualized on the same host that it protects, creating a management dependency during failures.
* The public path contains two reverse-proxy layers: Caddy on the VPS and Nginx Proxy Manager in the DMZ.
* The architecture diagram and inventory are maintained manually.

---

## Repository Status

This is an active personal project. Addresses, services, firewall policies, and diagrams may change as the environment is tested and improved.
