1. R1 — Router-on-a-Stick

Verify:
show ip interface brief
show running-config | section interface GigabitEthernet0/1

- [ ] G0/1.10 → 192.168.10.1 → up/up
- [ ] G0/1.20 → 192.168.20.1 → up/up
- [ ] G0/1.30 → 192.168.30.1 → up/up


2. R1 — Routing

Verify:
show ip route

- [ ] 192.168.10.0/24 is connected
- [ ] 192.168.20.0/24 is connected
- [ ] 192.168.30.0/24 is connected


3. R1 — DHCP

Verify:
show ip dhcp pool
show ip dhcp binding
show ip dhcp conflict
show running-config | section ip dhcp

- [ ] STAFF pool is operational
- [ ] POS pool is operational
- [ ] GUEST pool is operational
- [ ] Clients receive DHCP addresses
- [ ] No unexpected DHCP conflicts


4. R1 — NAT/PAT

Verify:
show ip nat statistics
show ip nat translations
show access-lists 1

- [ ] VLAN 10 included in NAT
- [ ] VLAN 20 included in NAT
- [ ] VLAN 30 included in NAT
- [ ] PAT uses G0/0
- [ ] Internet traffic creates translations


5. R1 — ACL Configuration

Verify:
show access-lists STAFF_POLICY
show access-lists POS_POLICY
show access-lists GUEST_POLICY

- [ ] DHCP permit entry is present
- [ ] Staff → POS is blocked
- [ ] Staff → Guest is blocked
- [ ] POS → Staff is blocked
- [ ] POS → Guest is blocked
- [ ] Guest → Staff is blocked
- [ ] Guest → POS is blocked
- [ ] Internet traffic is permitted


6. R1 — ACL Application

Verify:
show running-config | section interface GigabitEthernet0/1

- [ ] STAFF_POLICY applied inbound on G0/1.10
- [ ] POS_POLICY applied inbound on G0/1.20
- [ ] GUEST_POLICY applied inbound on G0/1.30


7. R1 — ACL Hit Counters

Verify:
show access-lists STAFF_POLICY
show access-lists POS_POLICY
show access-lists GUEST_POLICY

- [ ] Staff deny counters increase
- [ ] POS deny counters increase
- [ ] Guest deny counters increase


8. R1 — SSH Management

Verify:
show ip ssh
show ssh
show users
show running-config | section line vty
show running-config | include username
show crypto key mypubkey rsa

- [ ] SSH is enabled
- [ ] RSA keys are present
- [ ] VTY is configured for SSH
- [ ] Remote SSH access to R1 works


9. SW1 — VLANs

Verify:
show vlan brief

- [ ] VLAN 10 — Staff
- [ ] VLAN 20 — POS
- [ ] VLAN 30 — Guest
- [ ] VLAN 99 is absent


10. SW1 — Trunks

Verify:
show interfaces trunk

- [ ] Gi0/0 → R1 trunk
- [ ] VLAN 10, 20, 30 allowed toward R1
- [ ] Gi3/0 → OpenWrt
- [ ] VLAN 30 allowed toward OpenWrt


11. SW1 — Access Ports

Verify:
show interfaces status

- [ ] Staff ports → VLAN 10
- [ ] POS ports → VLAN 20
- [ ] Required unused ports are disabled


12. SW1 — Port Security

Verify:
show port-security

- [ ] Port Security enabled on Staff/POS access ports
- [ ] Maximum MAC = 1
- [ ] Violation mode = Shutdown
- [ ] Required ports are Secure-up
- [ ] Violations = 0


13. SW1 — STP Protection

Verify:
show spanning-tree vlan 10
show spanning-tree vlan 20
show spanning-tree vlan 30
show running-config | include spanning-tree portfast
show running-config | include spanning-tree bpduguard

- [ ] PortFast configured
- [ ] BPDU Guard configured
- [ ] Required access ports are forwarding


14. SW1 — Management State

Verify:
show ip interface brief
show running-config | section line vty
show running-config | include ip http

- [ ] No management SVI
- [ ] VLAN 99 management is absent
- [ ] HTTP is disabled
- [ ] HTTPS is disabled
- [ ] Remote VTY access is disabled


15. OpenWrt — Guest Bridge

Verify:
cat /etc/config/network
uci show network
ip link show br-lan
ip link show br-lan.30
bridge vlan show

- [ ] br-lan exists
- [ ] VLAN 30 exists
- [ ] eth0 → VLAN 30 tagged
- [ ] eth1 → VLAN 30 untagged
- [ ] OpenWrt operates as an L2 bridge
- [ ] OpenWrt is not the Guest gateway


16. OpenWrt — DHCP and Routing

Verify:
uci show dhcp
uci show firewall
ip route

- [ ] OpenWrt does not provide Guest DHCP
- [ ] OpenWrt does not perform Guest NAT
- [ ] OpenWrt does not provide Guest gateway
- [ ] R1 remains the Guest gateway


17. Alpine — Guest DHCP and Connectivity

Verify:
ip addr show eth0
ip route
ping -c 3 192.168.30.1
ping -c 3 8.8.8.8

- [ ] DHCP address received from R1
- [ ] Address belongs to 192.168.30.0/24
- [ ] Default gateway = 192.168.30.1
- [ ] Gateway ping successful
- [ ] Internet ping successful


18. Staff — Isolation and Internet

Test:
ping 192.168.20.21
ping 192.168.30.21
ping 8.8.8.8

- [ ] Staff → POS = BLOCKED
- [ ] Staff → Guest = BLOCKED
- [ ] Staff → Internet = ALLOWED


19. POS — Isolation and Internet

Test:
ping 192.168.10.21
ping 192.168.30.21
ping 8.8.8.8

- [ ] POS → Staff = BLOCKED
- [ ] POS → Guest = BLOCKED
- [ ] POS → Internet = ALLOWED


20. Guest — Isolation and Internet

Test:
ping 192.168.10.21
ping 192.168.20.21
ping 192.168.30.1
ping 8.8.8.8

- [ ] Guest → Staff = BLOCKED
- [ ] Guest → POS = BLOCKED
- [ ] Guest → Gateway = ALLOWED
- [ ] Guest → Internet = ALLOWED


21. Final Evidence

Verify:
show access-lists STAFF_POLICY
show access-lists POS_POLICY
show access-lists GUEST_POLICY
show ip nat translations
show ip nat statistics

- [ ] ACL deny counters confirm blocked traffic
- [ ] NAT translations are present
- [ ] Internet translations are verified
- [ ] Final lab behavior matches project objectives