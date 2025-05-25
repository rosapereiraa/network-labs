# Enterprise VLAN Segmentation Lab: Isolating Management Traffic with ACLs

## Objective
Implement one-way traffic control between VLANs to:
- Isolate management devices from client networks (PCI DSS/HIPAA compliance)
- Demonstrate secure network segmentation
- Prevent lateral movement during breaches

## Technologies
- Cisco Packet Tracer / GNS3
- VLANs, 802.1Q Trunking
- Extended ACLs (Cisco IOS)

## Lab Steps
1. VLAN Configuration (Switch)
```bash
vlan 10
 name SERVERS
interface range fa0/1-2
 switchport access vlan 10
```
2. Inter-VLAN Routing (Router)
```bash
interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```
3. ACL for One Way Isolation
```bash
access-list 100 deny icmp 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255 echo
access-list 100 permit ip any any
```
4. In order to test findings, try to ping management from a client PC, then the other way around:
```bash
ping 192.168.30.2 # should fail from client pc
ping 192.168.20.2 # should succeed from the management pc
```
## Topology and Results
Here is the final topology, in the right bottom corner we can appreciate the failed/successful ICMP messages depending on which VLAN is the one trying to communicate. 
![Screenshot 2025-05-25 163745](https://github.com/user-attachments/assets/1372a6fb-18a3-47b7-8d60-4f6878525340)
