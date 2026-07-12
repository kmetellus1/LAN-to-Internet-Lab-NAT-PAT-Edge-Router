<!-- PROJECT HEADER -->
<div align="center">

# 🌐 LAN-to-Internet Lab: NAT/PAT Edge Router

**A production-style Cisco Packet Tracer lab that connects a private LAN to a simulated internet and shares a single public IP across every host using NAT/PAT (overload).**
**It turns one of the CCNA's most-tested topics into a working, fully documented network you can open, run, and learn from in minutes.**

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet%20Tracer%208.x-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Cisco IOS](https://img.shields.io/badge/Cisco-IOS-005073?style=for-the-badge&logo=cisco&logoColor=white)
![Focus](https://img.shields.io/badge/Focus-NAT%20%2F%20PAT-2EA44F?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-red?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-success?style=for-the-badge)

</div>

---

## 📑 Table of Contents
- [Overview](#-overview)
- [Network Topology](#️-network-topology)
- [Key Features](#-key-features)
- [Built With](#️-built-with)
- [Installation](#-installation)
- [Quick Start / Usage](#-quick-start--usage)
- [Verification](#-verification)
- [Repository Structure](#-repository-structure)
- [Future Roadmap](#️-future-roadmap)
- [Author](#-author)

---

## 📖 Overview

Every office faces the same constraint: many devices inside, but only **one public IP** from the ISP. This lab solves it the way real edge routers do — with **Port Address Translation (PAT)**, letting an entire LAN reach the internet behind a single public address, tracked by port.

The topology models three realistic segments — a **private company LAN**, a **public WAN link**, and a **public "internet" server** — across two routers, a switch, and a DNS/web server. The result is a self-contained sandbox where you can watch a private host browse a public website *by name*, then inspect the exact translation table that made it happen.

Everything is documented at a beginner-friendly level (plain-English analogies, click-by-click and line-by-line steps, troubleshooting), with a dedicated security lens for those heading toward **SOC / Blue Team** work.

---

## 🗺️ Network Topology

```
 PC0 --+
       |                                          [  Internet  ]
 PC1 --+--[Switch0]--[EDGE  Gig0/1]===WAN===[Gig0/0  ISP  Gig0/1]--[Server0]
    192.168.1.0/24        .225      209.165.200.224/30    .1     209.165.201.10
    (private, NAT inside)           (public link)      209.165.201.0/24 (public)
```

| Segment | Network | Role |
|---|---|---|
| Company LAN | `192.168.1.0/24` | Private hosts (NAT **inside**) |
| WAN link | `209.165.200.224/30` | Customer ↔ ISP (public) |
| Internet server | `209.165.201.0/24` | DNS + Web server (public) |

<img width="2560" height="1430" alt="topology Lan to NAT" src="https://github.com/user-attachments/assets/a1315854-9cf8-451c-82c6-7ac5141f2dc5" />


---

## ✨ Key Features

- **PAT / NAT Overload** — an entire LAN shares one public IP, kept apart by port number.
- **Simulated internet edge** — a two-router design (customer **EDGE** + **ISP**) that mirrors a real service-provider handoff.
- **Public DNS + HTTP server** — browse to `www.example.com` and load a real page, proving name resolution and web access across the "internet."
- **Default routing + public/private addressing** — uses RFC 5737 documentation ranges, the professional (and exam-correct) choice.
- **Beginner-friendly documentation** — analogies, exact steps, and a full troubleshooting section in `docs/`.
- **Verification workflow** — confirm the build with `show ip nat translations` and see NAT working live.
- **Security / SOC angle** — how NAT translation logs answer "which internal host was behind this public IP?" during incident response.

---

## 🛠️ Built With

| Layer | Technology |
|---|---|
| Simulation | Cisco Packet Tracer 8.x |
| Network OS | Cisco IOS |
| Routing | 2× Cisco 2911 ISR |
| Switching | 1× Cisco 2960 |
| Services | DNS + HTTP (Packet Tracer Server) |
| Protocols / Concepts | NAT/PAT, Static & Default Routing, ICMP, DNS, HTTP, ACLs |
| Documentation | Markdown |

---

## 📦 Installation

**Prerequisites:** [Cisco Packet Tracer 8.0+](https://www.netacad.com/courses/packet-tracer) (free with a Cisco Networking Academy account).

```bash
# 1. Clone the repository
git clone https://github.com/kmetellus1/nat-pat-internet-lab.git

# 2. Move into the project folder
cd nat-pat-internet-lab

# 3. Open the lab in Cisco Packet Tracer
#    File  ->  Open  ->  nat-pat-internet-lab.pkt
```

That's it — the topology and base cabling load ready to go. To build it from scratch instead, follow the full walkthrough in `docs/Lab_NAT_PAT_Internet_Step_by_Step.md`.

---

## 🚀 Quick Start / Usage

The heart of the lab is the NAT configuration on the **EDGE** router. Open its CLI and paste:

```bash
enable
configure terminal

! --- Designate the NAT boundary ---
interface gig0/0
 ip nat inside
interface gig0/1
 ip nat outside
 exit

! --- Define which private hosts may be translated ---
access-list 1 permit 192.168.1.0 0.0.0.255

! --- Enable PAT: share the outside interface's public IP ---
ip nat inside source list 1 interface gig0/1 overload

! --- Send all unknown traffic to the ISP ---
ip route 0.0.0.0 0.0.0.0 209.165.200.226
end
write memory
```

Then, from **PC0**, open the web browser and visit:

```
http://www.example.com
```

A private `192.168.1.x` host just reached a public server through a single shared address. ✅

---

## 🔍 Verification

Generate traffic first (browse or ping the server), then on **EDGE**:

```bash
EDGE# show ip nat translations
```

Expected output (abbreviated) — note both PCs sharing one public IP:

```
Pro  Inside global          Inside local        Outside local       Outside global
tcp  209.165.200.225:1025   192.168.1.11:1025   209.165.201.10:80   209.165.201.10:80
tcp  209.165.200.225:1026   192.168.1.12:1026   209.165.201.10:80   209.165.201.10:80
```

| Check | Command / Action | Expected |
|---|---|---|
| Reach server by IP | `ping 209.165.201.10` (PC) | Replies |
| Resolve name | `nslookup www.example.com` (PC) | `209.165.201.10` |
| Load website | `http://www.example.com` (browser) | Page loads |
| Confirm translation | `show ip nat translations` (EDGE) | Inside → single global IP |

<img width="910" height="928" alt="ping 209 365 201 10" src="https://github.com/user-attachments/assets/2c736409-c27c-4f58-be39-a8471859305b" />

<img width="898" height="925" alt="PC0 Web Browser" src="https://github.com/user-attachments/assets/b2997ea8-0ade-4f9f-ae0c-be6fffdb3202" /> 

<img width="906" height="924" alt="Router Confirm tranlation" src="https://github.com/user-attachments/assets/97882283-f140-40cf-b581-095f013b1ffc" />




---

## 📂 Repository Structure

```
nat-pat-internet-lab/
├── README.md
├── LICENSE
├── nat-pat-internet-lab.pkt         # The Packet Tracer project
├── docs/
│   └── Lab_NAT_PAT_Internet_Step_by_Step.md   # Full guided walkthrough
└── screenshots/
    ├── topology.png
    ├── nat-translations.png
    └── browser-success.png
```

---

## 🗺️ Future Roadmap

- [ ] **Static NAT / port forwarding** — expose an inside web server to the internet and evolve the design into a basic **DMZ**.
- [ ] **Edge ACL hardening** — apply an inbound extended ACL on the outside interface to filter unsolicited traffic (a first firewall layer).
- [ ] **DHCP on EDGE** — auto-assign LAN addressing so clients configure themselves.
- [ ] **Syslog integration** — forward EDGE/ISP logs to a logging server as a stepping stone toward SIEM-based monitoring.

---

## 👤 Author

**Kirkland Metellus** — IT Support Technician | SOC Analyst
🔗 GitHub: [github.com/kmetellus1](https://github.com/kmetellus1)  ·  💼 LinkedIn: [linkedin.com/in/kmetellus](https://linkedin.com/in/kmetellus)

<div align="center">

*Part of an ongoing hands-on networking & security lab series. ⭐ Star the repo if it helped you learn NAT/PAT!*

</div>
