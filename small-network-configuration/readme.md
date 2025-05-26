# Network Reconfiguration Lab (Cisco Packet Tracer)
## Lab Overview
### Objective
Lab Overview
This lab modifies an existing network by:
- Migrating from Class C to Class A IP addressing (except Management VLAN)
- Configuring VLAN isolation (VLAN 40 can ping others, but not vice versa)
- Implementing DHCP, EIGRP, and ACLs for maximum security

### Why This Matters:
- Demonstrates real-world network redesign
- Shows how to scale IP addressing
- Teaches secure VLAN segmentation
## Topology
![Screenshot 2025-05-25 170342](https://github.com/user-attachments/assets/e9fba26a-39d6-4341-91db-fbe5d7f781c1)
## Configuration Steps
### 1. Switch (S1) Configuration
Create VLANs
```bash
enable
conf t
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
exit
```
2. Assign Ports
```bash
int range f0/1-2
 switchport mode access
 switchport access vlan 10
int range f0/3-4
 switchport mode access
 switchport access vlan 20
int range f0/5-6
 switchport mode access
 switchport access vlan 30
int f0/7
 switchport mode access
 switchport access vlan 40
exit
```
3. Disable Unused Ports (best security practice)
```bash
int range f0/8-23, g0/1-2
 shutdown
 switchport mode access
 switchport access vlan 999
exit
```
4. Configure Trunk
```bash
int f0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40
 switchport trunk native vlan 40
 no shutdown
end
```
### 5. R1 Configuration
```bash
enable
conf t
int g0/0.10
 encapsulation dot1Q 10
 ip address 10.10.0.1 255.255.255.0
int g0/0.20
 encapsulation dot1Q 20
 ip address 10.20.0.1 255.255.255.0
int g0/0.30
 encapsulation dot1Q 30
 ip address 10.30.0.1 255.255.255.0
int g0/0.40
 encapsulation dot1Q 40 native
 ip address 192.168.40.1 255.255.255.248
 no shutdown
exit
```
6. DHCP Configuration
```bash
ip dhcp excluded-address 10.10.0.1
ip dhcp excluded-address 10.20.0.1
ip dhcp excluded-address 10.30.0.1

ip dhcp pool VLAN_10
 network 10.10.0.0 255.255.255.0
 default-router 10.10.0.1
ip dhcp pool VLAN_20
 network 10.20.0.0 255.255.255.0
 default-router 10.20.0.1
ip dhcp pool VLAN_30
 network 10.30.0.0 255.255.255.0
 default-router 10.30.0.1
exit
```
7. ACL Configuration for VLAN isolation
```bash
ip access-list extended MANAGEMENT_FILTER
 permit icmp 192.168.40.0 0.0.0.7 10.10.0.0 0.0.0.255 echo
 permit icmp 192.168.40.0 0.0.0.7 10.20.0.0 0.0.0.255 echo
 permit icmp 192.168.40.0 0.0.0.7 10.30.0.0 0.0.0.255 echo
 permit icmp 10.10.0.0 0.0.0.255 192.168.40.0 0.0.0.7 echo-reply
 permit icmp 10.20.0.0 0.0.0.255 192.168.40.0 0.0.0.7 echo-reply
 permit icmp 10.30.0.0 0.0.0.255 192.168.40.0 0.0.0.7 echo-reply
 deny icmp any 192.168.40.0 0.0.0.7 echo
 permit ip 10.10.0.0 0.0.0.255 10.20.0.0 0.0.0.255
 permit ip 10.10.0.0 0.0.0.255 10.30.0.0 0.0.0.255
 permit ip 10.20.0.0 0.0.0.255 10.30.0.0 0.0.0.255
 deny ip any any
exit
int g0/0.40
 ip access-group MANAGEMENT_FILTER out
end
```
This configuration allows pings from VLAN 40 to all other VLANs, but blocks 10,20,30 from pinging VLAN 40. Still allows 10,20,30 to communicate with each other. 
### 8. Outer Router Configuration
```bash
enable
conf t
interface Serial0/0/0
 ip address 172.16.0.1 255.255.255.252
 no shutdown
exit
interface Loopback0
 ip address 209.165.200.1 255.255.255.255
exit
```
9. Static Routes
```bash
ip route 10.10.0.0 255.255.255.0 172.16.0.2
ip route 10.20.0.0 255.255.255.0 172.16.0.2
ip route 10.30.0.0 255.255.255.0 172.16.0.2
ip route 192.168.40.0 255.255.255.248 172.16.0.2
end
```
Simulated internet connectivity through Loopback, and routes traffic to R1 for internal VLANs. 
### 9. Test Connection
10. Ping from VLAN 40 (PC6) to any other VLAN (10.10.0.2 for instance)... This should work.
11. Ping from VLAN 10 to VLAN 40 (192.168.40.3) this should fail.
The ACL for this lab was specific for ping requests to the visible in and out of VLAN 40, this was just the specification for this lab... A more real world use of ACL would look like this:
```bash
ip access-list extended MANAGEMENT_FILTER
 ! ALLOWED SERVICES (VLAN 40 → OTHERS)
 ! ICMP (Ping)
 permit icmp 192.168.40.0 0.0.0.7 10.10.0.0 0.0.0.255 echo
 permit icmp 192.168.40.0 0.0.0.7 10.20.0.0 0.0.0.255 echo
 permit icmp 192.168.40.0 0.0.0.7 10.30.0.0 0.0.0.255 echo
 ! SSH (Port 22)
 permit tcp 192.168.40.0 0.0.0.7 10.10.0.0 0.0.0.255 eq 22 log
 permit tcp 192.168.40.0 0.0.0.7 10.20.0.0 0.0.0.255 eq 22 log
 permit tcp 192.168.40.0 0.0.0.7 10.30.0.0 0.0.0.255 eq 22 log
 ! RDP (Port 3389)
 permit tcp 192.168.40.0 0.0.0.7 10.10.0.0 0.0.0.255 eq 3389 log
 permit tcp 192.168.40.0 0.0.0.7 10.20.0.0 0.0.0.255 eq 3389 log
 permit tcp 192.168.40.0 0.0.0.7 10.30.0.0 0.0.0.255 eq 3389 log
 ! DNS (Port 53)
 permit udp 192.168.40.0 0.0.0.7 any eq 53 log
 ! ALLOW REPLIES TO VLAN 40
 permit icmp 10.10.0.0 0.0.0.255 192.168.40.0 0.0.0.7 echo-reply
 permit icmp 10.20.0.0 0.0.0.255 192.168.40.0 0.0.0.7 echo-reply
 permit icmp 10.30.0.0 0.0.0.255 192.168.40.0 0.0.0.7 echo-reply
 permit tcp 10.10.0.0 0.0.0.255 192.168.40.0 0.0.0.7 established log
 permit tcp 10.20.0.0 0.0.0.255 192.168.40.0 0.0.0.7 established log
 permit tcp 10.30.0.0 0.0.0.255 192.168.40.0 0.0.0.7 established log

 ! (PROTECT VLAN 40)
 ! Block common attacks
 deny tcp any 192.168.40.0 0.0.0.7 eq 22 log
 deny tcp any 192.168.40.0 0.0.0.7 eq 3389 log
 deny udp any 192.168.40.0 0.0.0.7 eq 161 log  # SNMP
 deny tcp any 192.168.40.0 0.0.0.7 eq 80 log   # HTTP
 deny tcp any 192.168.40.0 0.0.0.7 eq 443 log  # HTTPS

 !ALLOW INTER-VLAN COMMUNICATION
 permit ip 10.10.0.0 0.0.0.255 10.20.0.0 0.0.0.255
 permit ip 10.10.0.0 0.0.0.255 10.30.0.0 0.0.0.255
 permit ip 10.20.0.0 0.0.0.255 10.30.0.0 0.0.0.255
 deny ip any any log
```
