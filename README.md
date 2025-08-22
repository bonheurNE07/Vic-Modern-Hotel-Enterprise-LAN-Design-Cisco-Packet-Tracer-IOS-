# Vic Modern Hotel – Enterprise LAN Design (Cisco Packet Tracer / IOS)

This repository documents the design and configuration of **Vic Modern Hotel’s multi-floor enterprise LAN**. The project demonstrates how to implement a professional **campus-style network** in Cisco Packet Tracer using **Cisco IOS routers and switches**.

The network spans **three floors**, each with multiple departments, VLAN segmentation, DHCP per VLAN, OSPF dynamic routing, secure SSH management, and port-security for sensitive endpoints.

---

## Repository structure

```
Vic-Modern-Hotel-Enterprise-LAN-Design-Cisco-Packet-Tracer-IOS/
├─ configs/
│  ├─ routers/
│  │  ├─ F1-RT.cfg   ← Router for 1st floor
│  │  ├─ F2-RT.cfg   ← Router for 2nd floor
│  │  └─ F3-RT.cfg   ← Router for 3rd floor (IT/Admin)
│  ├─ switches/
│  │  ├─ F1-SW.cfg   ← Switch for 1st floor
│  │  ├─ F2-SW.cfg   ← Switch for 2nd floor
│  │  └─ F3-SW.cfg   ← Switch for 3rd floor
│  └─ topology-diagram.png  ← exported Packet Tracer diagram
|  └─ topology-diagram.png  ← exported Cisco Packet Tracer file
└─ README.md  ← this documentation
```

Each **.cfg file** contains the Cisco IOS configuration of the respective device. You can obtain them by running `show running-config` in Packet Tracer and saving the output to a text file.

---

## Overview of technologies used

* **VLANs (10–80)** – segmentation by department
* **Router-on-a-stick** – subinterfaces per VLAN
* **DHCP per VLAN** – automatic IP assignment
* **OSPF (Area 0)** – dynamic routing between routers/floors
* **SSH** – secure remote access to routers
* **Port-security** – restrict IT Test-PC port using sticky MAC
* **Wireless Access Points** – WiFi connectivity per floor

---

---
