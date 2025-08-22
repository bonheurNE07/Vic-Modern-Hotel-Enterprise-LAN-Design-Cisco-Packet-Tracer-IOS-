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
# 1. IP Plan & VLANs

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
# 2. Routers Configurations

## Router Configuration – Floor 1 (F1-RT)

### Purpose
- Acts as the default gateway for VLANs 10, 20, and 30 (Reception, Restaurant, Kitchen).
- Provides inter-VLAN routing using **Router-on-a-Stick**.
- Runs **OSPF** to advertise connected networks.
- Provides **DHCP pools** per VLAN.
- Allows **SSH remote access**.

---

## Base Configuration (F1-RT)
```
hostname F1-RT
no ip domain-lookup
!
! Enable secret
enable secret xpass10admin
! Configure SSH
ip domain-name vicmodern.local
crypto key generate rsa modulus 1024
ip ssh version 2
username admin-38 privilege 15 secret xpass8admin
!
line vty 0 4
 login local
 transport input ssh
!
```

## Interfaces to other routers (Serial DCE/DTE links)
```
interface Serial0/0/0
ip address 10.10.10.1 255.255.255.252
clock rate 64000
no shutdown
!
interface Serial0/0/1
ip address 10.10.10.5 255.255.255.252
no shutdown
```
---

## Subinterfaces for VLANs
```
interface g0/0
 no shutdown

interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface g0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
```

---

## DHCP Pools
```
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10

ip dhcp pool VLAN10-POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8

ip dhcp pool VLAN20-POOL
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8

ip dhcp pool VLAN30-POOL
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8
```

---

## OSPF Configuration
```
router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 10.10.10.4 0.0.0.3 area 0
 network 10.10.10.0 0.0.0.3 area 0
```

---

# Router Configuration – Floor 2 (F2-RT)

## Purpose

* Acts as the default gateway for VLANs 40, 50, and 60 (Rooms, Conference Hall, Gym).
* Provides inter-VLAN routing using **Router-on-a-Stick**.
* Runs **OSPF** to advertise connected networks.
* Provides **DHCP pools** per VLAN.
* Allows **SSH remote access**.

---

## Base Configuration (F2-RT)

```
hostname F2-RT
no ip domain-lookup

! Enable secret
enable secret xpass10admin
!
ip domain-name vicmodern.local
crypto key generate rsa modulus 1024
ip ssh version 2
username admin-38 privilege 15 secret xpass8admin
!
line vty 0 4
 login local
 transport input ssh
!
```

## Interfaces to other routers (Serial DCE/DTE links)
```
interface Serial0/0/0
ip address 10.10.10.6 255.255.255.252
clock rate 64000
no shutdown
!
interface Serial0/0/1
ip address 10.10.10.9 255.255.255.252
no shutdown
```
---

## Subinterfaces for VLANs

```
interface g0/0
 no shutdown

interface g0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0

interface g0/0.50
 encapsulation dot1Q 50
 ip address 192.168.50.1 255.255.255.0

interface g0/0.60
 encapsulation dot1Q 60
 ip address 192.168.60.1 255.255.255.0
```

---

## DHCP Pools

```
ip dhcp excluded-address 192.168.40.1 192.168.40.10
ip dhcp excluded-address 192.168.50.1 192.168.50.10
ip dhcp excluded-address 192.168.60.1 192.168.60.10

ip dhcp pool VLAN40-POOL
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.1
 dns-server 8.8.8.8

ip dhcp pool VLAN50-POOL
 network 192.168.50.0 255.255.255.0
 default-router 192.168.50.1
 dns-server 8.8.8.8

ip dhcp pool VLAN60-POOL
 network 192.168.60.0 255.255.255.0
 default-router 192.168.60.1
 dns-server 8.8.8.8
```

---

## OSPF Configuration

```
router ospf 1
 router-id 2.2.2.2
 network 192.168.40.0 0.0.0.255 area 0
 network 192.168.50.0 0.0.0.255 area 0
 network 192.168.60.0 0.0.0.255 area 0
 network 10.10.10.4 0.0.0.3 area 0
 network 10.10.10.8 0.0.0.3 area 0
```

---

# Router Configuration – Floor 3 (F3-RT)

## Purpose

* Acts as the default gateway for VLANs 70 and 80 (IT Department and Admin Offices).
* Provides inter-VLAN routing using **Router-on-a-Stick**.
* Runs **OSPF** to advertise connected networks.
* Provides **DHCP pools** per VLAN.
* Allows **SSH remote access**.
* Enforces **port-security** on IT Test-PC (VLAN 70).

---

## Base Configuration (F3-RT)

```cisco
hostname F3-RT
no ip domain-lookup

! Enable secret
enable secret xpass10admin
!
ip domain-name vicmodern.local
crypto key generate rsa modulus 1024
ip ssh version 2
username admin-38 privilege 15 secret xpass8admin
!
line vty 0 4
 login local
 transport input ssh
!
```

---

## Interfaces to Other Routers (Serial DCE/DTE links)

```cisco
interface Serial0/0/0
 ip address 10.10.10.2 255.255.255.252
 no shutdown
!
interface Serial0/0/1
 ip address 10.10.10.10 255.255.255.252
 clock rate 64000
 no shutdown
```

---

## Subinterfaces for VLANs

```cisco
interface g0/0
 no shutdown

interface g0/0.70
 encapsulation dot1Q 70
 ip address 192.168.70.1 255.255.255.0

interface g0/0.80
 encapsulation dot1Q 80
 ip address 192.168.80.1 255.255.255.0

interface g0/0.90
 encapsulation dot1Q 90
 ip address 192.168.90.1 255.255.255.0
```

---

## DHCP Pools

```cisco
ip dhcp excluded-address 192.168.70.1 192.168.70.10
ip dhcp excluded-address 192.168.80.1 192.168.80.10
ip dhcp excluded-address 192.168.90.1 192.168.80.10

ip dhcp pool VLAN70-POOL
 network 192.168.70.0 255.255.255.0
 default-router 192.168.70.1
 dns-server 8.8.8.8

ip dhcp pool VLAN80-POOL
 network 192.168.80.0 255.255.255.0
 default-router 192.168.80.1
 dns-server 8.8.8.8

ip dhcp pool VLAN90-POOL
 network 192.168.90.0 255.255.255.0
 default-router 192.168.90.1
 dns-server 8.8.8.8
```

---

## OSPF Configuration

```cisco
router ospf 1
 router-id 3.3.3.3
 network 192.168.70.0 0.0.0.255 area 0
 network 192.168.80.0 0.0.0.255 area 0
network 192.168.90.0 0.0.0.255 area 0
 network 10.10.10.8 0.0.0.3 area 0
 network 10.10.10.12 0.0.0.3 area 0
```

---

## Port-Security (on F3-SW for IT Test-PC)

```cisco
interface fa0/1
  switchport mode access
  switchport access vlan 70
  switchport port-security
  switchport port-security maximum 1
  switchport port-security mac-address sticky
  switchport port-security violation restrict
```

---
