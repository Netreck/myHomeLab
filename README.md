#  Personal Homelab

Personal homelab project documentation focused on learning more about computing and using the environment as a server for personal projects.  
I plan to continuously update this repository as the system evolves and matures.

**Note:** The main idea is to keep the **VPS structure as minimal as possible**, using the cheapest option available and avoiding unnecessary costs. Using it only for exposing ipv4.

---
## 👋 About Me

My name is **Gabriel Gonçalves**, and I am a **Computer Science student at UFABC**.

---

## 🔜 Next Updates

Known issues, ideas for future updates, security improvements, and architectural improvements.

- **WAF (Web Application Firewall)** for improved application security.  
  The idea is to place it *before* traffic reaches Nginx inside Proxmox, helping prevent injections and other common attacks.

- **FastNetMon Community** for Layer 3/4 (UDP/TCP) DDoS detection.  
  The initial plan is to start with basic detection and later integrate it with another personal project focused on DDoS detection using Machine Learning.

- **Hardware upgrade**
  - +2 × 16 GB RAM modules  
  - Dedicated GPU for AI processing / training 3090 24GB 

- **Migrate personal project currently running on GCP**
  - *HireMatch AI*: a project that simulates / assists ATS systems in resume analysis.


---

## 📐 Architecture Diagram

<img width="784" height="793" alt="architecture diagram" src="https://github.com/user-attachments/assets/a796926a-b3cf-4b7e-b4f7-03b6314917a6" />

---

## 🧠 Project Overview

The homelab uses **Proxmox OS** as the virtualization platform, hosting virtual machines and LXC containers.  
Since my ISP uses **CGNAT** and charges extra for a public IPv4 address, external access to services is securely handled through a **VPS with a public IP**, using **WireGuard** orchestrated by **Netmaker**.

External traffic reaches the VPS, where **Caddy** manages TLS certificates and forwards connections through the VPN tunnel to the homelab.  
Inside the local environment, **Nginx Proxy Manager** (reverse proxy) routes traffic to internal services, which use **private IPv6 addressing**.  
**Pi-hole** is used only as an internal LAN DNS for ad-blocking and anti-tracking and is **not part of the server network architecture**.

---

## 🧩 Services and Machines (VMs and LXC)

This section briefly describes the main services running in the Proxmox environment.

## Proxmox PVE

- **ProxMenuX**  
  Dashboard used for server monitoring, including logs, resource usage, hardware temperature, and memory allocation.

  Example image:  
  <img width="1546" height="970" alt="ProxMenuX example" src="https://github.com/user-attachments/assets/f36f639f-05b4-40eb-a2c3-77d44d1df22e" />

---

### LXC Containers

- **Pi-hole** (LXC)  
  Internal DNS for the local network, responsible for ad-blocking and DNS resolution for specific devices.

- **Nginx Proxy Manager** (LXC)  
  Internal reverse proxy responsible for routing traffic to the correct services inside the LAN.

- **Netmaker Client** (LXC)  
  Client responsible for connecting the homelab to the WireGuard VPN network managed by Netmaker.

- **Main Monitor [Grafana]** (LXC)  
  Granafa,loki, prometheus setup  to centralize logs visualization in a single point.



### Virtual Machines (VMs)

- **TrueNAS** (VM) -- alocated 100GB
  Used as a NAS server to store files, acting as a private cloud storage solution.

---

## 🔐 Technologies Used

- Proxmox VE  
- LXC and Virtual Machines  
- Netmaker  
- WireGuard  
- Caddy (TLS / HTTPS)  
- Nginx Proxy Manager  
- Pi-hole  
- Cloudflare DNS
- Grafana
- Loki
- Prometheus
---

## 🖥️ Server Hardware Specifications

- **Motherboard:** X99 D4 Atermiter  
- **CPU:** Intel Xeon E5-2680 v4 (LGA 2011-3)  
- **Memory:** 32 GB DDR4 ECC RDIMM (2×16 GB)  
- **Storage:** 512 GB NVMe M.2 SSD  
- **GPU:** NVIDIA RTX 3070 Ti  
- **Power Supply:** 750 W  


