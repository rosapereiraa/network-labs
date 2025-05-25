# Network Reconfiguration Lab (Cisco Packet Tracer)
## Lab Overview
### Objective
Modified an existing network to:
- Convert IP addressing from **Class C (192.168.x.x)** to **Class A (10.x.x.x)**
- Implement VLAN segmentation (Office/Public/Corporate/Management)
- Enforce security via ACLs and port security

### Key Features
- DHCP server per VLAN
- One-way traffic isolation
- EIGRP routing between R1 and Outer router
- Black Hole VLAN for unused ports

## Topology
![Screenshot 2025-05-25 170342](https://github.com/user-attachments/assets/e9fba26a-39d6-4341-91db-fbe5d7f781c1)

## Configuration Steps
1. VLAN Configuration (S1)
```bash
vlan 10
 name Office
vlan 20
 name Public
vlan 30
 name Corporate
vlan 40
 name Management
vlan 999
 name Black_Hole
```
2. Port Assignment (S1)
```bash
# Black Hole VLAN for unused ports
int range f0/8-23,g0/1-2
 shutdown
 switchport mode access
 switchport access vlan 999
# Active ports with port security
int range f0/1-7
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
```
3. Router Subinterfaces (R1)
```bash
interface g0/0.10
 encapsulation dot1Q 10
 ip address 10.10.0.1 255.255.255.0
```
4. DHCP Configuration (R1)
```bash
ip dhcp excluded-address 10.10.0.1
ip dhcp pool VLAN_10
 network 10.10.0.0 255.255.255.0
 default-router 10.10.0.1
```
5. Security ACLs
```bash
ip access-list extended MANAGEMENT_FILTER
 permit ip 192.168.40.0 0.0.0.7 any
 deny ip any any
interface g0/0.40
 ip access-group MANAGEMENT_FILTER out
```
6. Verify connection by pinging from different VLANs. VLAN 10 devices should be able to ping VLAN 20 devices but not VLAN 40 devices.

This configuration implements scalable addressing that aims to support future expansion, zero trust principles, and meets PCI DSS/HIPAA segmentation requirements.
