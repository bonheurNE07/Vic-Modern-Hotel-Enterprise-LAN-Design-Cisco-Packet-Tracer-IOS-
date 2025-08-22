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
## 1. IP Plan & VLANs

Each floor has its own departments, with dedicated VLAN IDs and subnets. We use **192.168.X.0/24** addressing, where **X = VLAN ID** for simplicity.

| Floor | Department         | VLAN ID | Subnet          | Gateway IP   |
| ----- | ------------------ | ------- | --------------- | ------------ |
| 1     | Reception          | 10      | 192.168.10.0/24 | 192.168.10.1 |
| 1     | Restaurant         | 20      | 192.168.20.0/24 | 192.168.20.1 |
| 1     | Kitchen            | 30      | 192.168.30.0/24 | 192.168.30.1 |
| 2     | Finance            | 40      | 192.168.40.0/24 | 192.168.40.1 |
| 2     | HR                 | 50      | 192.168.50.0/24 | 192.168.50.1 |
| 2     | Guest Rooms (WiFi) | 60      | 192.168.60.0/24 | 192.168.60.1 |
| 3     | IT Department      | 70      | 192.168.70.0/24 | 192.168.70.1 |
| 3     | Admin Offices      | 80      | 192.168.80.0/24 | 192.168.80.1 |

### Notes

* Each VLAN has its **own DHCP pool** served by the floor router.
* Default gateways are configured on router subinterfaces.
* Wireless clients (VLAN 60) connect through access points mapped to their VLAN.
* VLAN 70 (IT) will have **port-security** enabled on its Test-PC port.

---
