# Set up Process

This is a simple Access Control List lab that blocks the client server (VLAN 20) from being able to ping the management server (VLAN 30).

The set up for this lab was fairly simple. VLANs 10, 20, 30 correspond to the servers, clients, and management respectively. IPs were assigned according to the corresponding VLAN for each department. For instance, SVR1's IP is 192.168.10.2 and SVR2 was assigned 192.168.10.3 and so on for every device in the diagram. 

## Assigning VLANs

1. To assign VLAN names, we went to the "Switch0" CLI
```bash
enable
config t
vlan 10
name Servers
exit
```
2. To assign interfaces in the same CLI we did (repeat for all VLANs accordingly)
```bash
interface fastethernet0/1
switchport mode access
switchport access vlan 10
exit
```
3. Then we configured trunk mode:
```bash
int fastethernet0/24
switchport mode trunk
switchport trunk allowed vlan 10,20,30
exit
```
4. On the router's CLI:
```bash
enable
configure terminal
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit
interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit
interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
exit
interface FastEthernet0/0
 no shutdown
exit
```
4. To restrict communication between clients and management we created an ACL to deny traffic from VLAN 20 to VLAN 30:
```bash
enable
config t
access-list 100 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 100 permit ip any any
interface FastEthernet0/0.20
 ip access-group 100 in
exit
```
5. Finally, IPs were assigned to the devices as explained above. Making sure that the default gateway for each device matches the corresponding interface for each department, for instance, for all devices in VLAN 10 the gateway is 192.168.10.1, and so on for all other devices in different VLANs.

To test the connectivity, clients are able to ping the servers and other clients, but never the management VLAN.
