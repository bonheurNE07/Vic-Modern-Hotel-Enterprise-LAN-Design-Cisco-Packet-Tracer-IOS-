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
| 1     | Wi-Fi              | 35      | 192.168.35.0/24 | 192.168.35.1 |
| 2     | Finance            | 40      | 192.168.40.0/24 | 192.168.40.1 |
| 2     | HR                 | 50      | 192.168.50.0/24 | 192.168.50.1 |
| 2     | Sales/Marketing    | 60      | 192.168.60.0/24 | 192.168.60.1 |
| 2     | Guest Rooms (WiFi) | 65      | 192.168.65.0/24 | 192.168.65.1 |
| 3     | IT Department      | 70      | 192.168.70.0/24 | 192.168.70.1 |
| 3     | Admin Offices      | 80      | 192.168.80.0/24 | 192.168.80.1 |
| 3     | Wi-Fi              | 90      | 192.168.90.0/24 | 192.168.90.1 |

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

interface g0/0.35
 encapsulation dot1Q 35
 ip address 192.168.35.1 255.255.255.0
```  

---

## DHCP Pools
```
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10
ip dhcp excluded-address 192.168.35.1

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

ip dhcp pool VLAN30-POOL
 network 192.168.35.0 255.255.255.0
 default-router 192.168.35.1
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
 network 192.168.35.0 0.0.0.255 area 0
 network 10.10.10.4 0.0.0.3 area 0
 network 10.10.10.0 0.0.0.3 area 0
```

---

# Router Configuration – Floor 2 (F2-RT)

## Purpose

* Acts as the default gateway for VLANs 40, 50, 65, and 60 (Rooms, Conference Hall, Gym).
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

interface g0/0.65
 encapsulation dot1Q 65
 ip address 192.168.65.1 255.255.255.0
```

---

## DHCP Pools

```
ip dhcp excluded-address 192.168.40.1 192.168.40.10
ip dhcp excluded-address 192.168.50.1 192.168.50.10
ip dhcp excluded-address 192.168.60.1 192.168.60.10
ip dhcp excluded-address 192.168.65.1 

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

ip dhcp pool VLAN60-POOL
 network 192.168.60.0 255.255.255.0
 default-router 192.168.65.1
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
 network 192.168.65.0 0.0.0.255 area 0
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

# 3. Switch Configurations

## Switch – Floor 1 (F1-SW)

**Purpose**

* Reception (VLAN 10), Restaurant (VLAN 20), Kitchen (VLAN 30), Wi-Fi(35)
* Trunk to F1-RT on **Gi0/1** with native VLAN 99
* AP on **Gi0/2** → Public Wi‑Fi (VLAN 35)
* Unused ports parked in VLAN 99 and shutdown

```cisco
! ===== Base =====
hostname F1-SW
no ip domain-lookup
service password-encryption
! Set device domain name (needed for SSH keys)
ip domain-name vicmodern.local

! Create RSA key pair for SSH (1024-bit, you can use 2048 if IOS supports)
crypto key generate rsa modulus 1024

! Force SSH version 2
ip ssh version 2

! Create local admin user
username admin-38 privilege 15 secret xpass8admin

! Set enable secret
enable secret xpass10admin

! Configure VTY lines for SSH login
line vty 0 4
 login local
 transport input ssh

spanning-tree mode pvst
spanning-tree extend system-id

! ===== VLANs =====
vlan 10
 name RECEPTION
vlan 20
 name RESTAURANT
vlan 30
 name KITCHEN
vlan 35
 name Wi-Fi
vlan 99
 name NATIVE-BLACKHOLE

! ===== Trunk to Router (Gi0/1) =====
interface GigabitEthernet0/1
 description Uplink to F1-RT
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,35
 switchport mode trunk

! ===== Access ports (examples) =====
interface FastEthernet0/1
 description Reception
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast

interface FastEthernet0/2
 description Reception
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast

interface FastEthernet0/6
 description Restaurant
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast

interface FastEthernet0/7
 description Restaurant
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast

interface FastEthernet0/11
 description Kitchen wired
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast

interface FastEthernet0/12
 description Kitchen wired
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast

! ===== AP port (Gi0/2) → Public Wi‑Fi on VLAN 45 =====
interface GigabitEthernet0/2
 description AP – Public Wi‑Fi
 switchport mode access
 switchport access vlan 45
 spanning-tree portfast

! ===== Park and disable all remaining unused ports =====
interface range FastEthernet0/3-5, FastEthernet0/8-10, FastEthernet0/13-24
 description UNUSED (parked)
 switchport mode access
 switchport access vlan 99
 shutdown

! ===== Disable unused SVI =====
interface Vlan1
 no ip address
 shutdown
!
end
```

---

## Switch – Floor 2 (F2-SW)

**Purpose**

* Finance (VLAN 60), HR (VLAN 50), Sales-Marketing (VLAN 40), Guest Wi‑Fi (VLAN 65)
* Trunk to F2-RT on **Gi0/1** with native VLAN 99
* AP on **Gi0/2** → Guest Wi‑Fi (VLAN 60)
* Unused ports parked in VLAN 99 and shutdown

```cisco
! ===== Base =====
hostname F2-SW
no ip domain-lookup
service password-encryption
! Set device domain name (needed for SSH keys)
ip domain-name vicmodern.local

! Create RSA key pair for SSH (1024-bit, you can use 2048 if IOS supports)
crypto key generate rsa modulus 1024

! Force SSH version 2
ip ssh version 2

! Create local admin user
username admin-38 privilege 15 secret xpass8admin

! Set enable secret
enable secret xpass10admin

! Configure VTY lines for SSH login
line vty 0 4
 login local
 transport input ssh

spanning-tree mode pvst
spanning-tree extend system-id

! ===== VLANs =====
vlan 40
 name FINANCE
vlan 50
 name HR
vlan 60
 name SALES-MARKETING
vlan 65
 name GUEST-WIFI
vlan 99
 name NATIVE-BLACKHOLE

! ===== Trunk to Router (Gi0/1) =====
interface GigabitEthernet0/1
 description Uplink to F2-RT
 switchport trunk native vlan 99
 switchport trunk allowed vlan 40,50,60,65
 switchport mode trunk

! ===== Access ports (examples) =====
interface FastEthernet0/1
 description Finance
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast

interface FastEthernet0/2
 description Finance
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast

interface FastEthernet0/6
 description HR
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast

interface FastEthernet0/7
 description HR
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast

interface FastEthernet0/11
 description Sales Marketing
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast

interface FastEthernet0/12
 description Sales Marketing
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast

! ===== AP port (Gi0/2) → Guest Wi‑Fi on VLAN 65 =====
interface GigabitEthernet0/2
 description AP – Guest Wi‑Fi
 switchport mode access
 switchport access vlan 65
 spanning-tree portfast

! ===== Park and disable all remaining unused ports =====
interface range FastEthernet0/3-5, FastEthernet0/8-10, FastEthernet0/13-24
 description UNUSED (parked)
 switchport mode access
 switchport access vlan 99
 shutdown

! ===== Disable unused SVI =====
interface Vlan1
 no ip address
 shutdown
!
end
```

---

## Switch – Floor 3 (F3-SW)

**Purpose**

* IT (VLAN 70), Admin (VLAN 80), IT Wi‑Fi (VLAN 90)
* Trunk to F3-RT on **Gi0/1** with native VLAN 99
* AP on **Gi0/2** → IT Wi‑Fi (VLAN 90)
* **Port‑security** on IT Test‑PC (Fa0/1) → sticky MAC, violation **shutdown**
* Unused ports parked in VLAN 99 and shutdown

```cisco
! ===== Base =====
hostname F3-SW
no ip domain-lookup
service password-encryption
! Set device domain name (needed for SSH keys)
ip domain-name vicmodern.local

! Create RSA key pair for SSH (1024-bit, you can use 2048 if IOS supports)
crypto key generate rsa modulus 1024

! Force SSH version 2
ip ssh version 2

! Create local admin user
username admin-38 privilege 15 secret xpass8admin

! Set enable secret
enable secret xpass10admin

! Configure VTY lines for SSH login
line vty 0 4
 login local
 transport input ssh

spanning-tree mode pvst
spanning-tree extend system-id

! ===== VLANs =====
vlan 70
 name IT
vlan 80
 name ADMIN
vlan 90
 name IT-WIFI
vlan 99
 name NATIVE-BLACKHOLE

! ===== Trunk to Router (Gi0/1) =====
interface GigabitEthernet0/1
 description Uplink to F3-RT
 switchport trunk native vlan 99
 switchport trunk allowed vlan 70,80,90
 switchport mode trunk

! ===== IT Test-PC with Port-Security (Fa0/1) =====
interface FastEthernet0/1
 description IT Test-PC (sticky)
 switchport mode access
 switchport access vlan 70
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 spanning-tree portfast

! ===== IT general =====
interface FastEthernet0/2
 description IT user
 switchport mode access
 switchport access vlan 70
 spanning-tree portfast

! ===== Admin =====
interface FastEthernet0/6
 description Admin user
 switchport mode access
 switchport access vlan 80
 spanning-tree portfast

interface FastEthernet0/7
 description Admin user
 switchport mode access
 switchport access vlan 80
 spanning-tree portfast

! ===== AP port (Gi0/2) → IT Wi‑Fi on VLAN 90 =====
interface GigabitEthernet0/2
 description AP – IT Wi‑Fi
 switchport mode access
 switchport access vlan 90
 spanning-tree portfast

! ===== Park and disable all remaining unused ports =====
interface range FastEthernet0/3-5, FastEthernet0/8-24
 description UNUSED (parked)
 switchport mode access
 switchport access vlan 99
 shutdown

! ===== Disable unused SVI =====
interface Vlan1
 no ip address
 shutdown
!
end
```


---
