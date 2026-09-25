

## 1. Initial Interface Configuration

The first step is to establish the connection between the external network, router, switch, and internal network segments.

R1 G0/0 connects to the GNS3 NAT network and obtains its external address through DHCP. R1 G0/1 is used as the trunk connection toward SW1 and does not require a physical IP address because the internal VLAN gateways are configured as subinterfaces.

### R1 — External Interface

```text
enable
configure terminal

interface GigabitEthernet0/0
 ip address dhcp
 no shutdown
exit
````

### R1 — Internal Trunk Interface

```text
interface GigabitEthernet0/1
 no ip address
 no shutdown
exit
```

---

## 2. VLAN and IP Network Configuration

The café requires three separate logical networks. Each functional area receives its own VLAN and /24 subnet so that traffic can be controlled independently.

|VLAN|Name|Network|Gateway|
|---|---|---|---|
|10|STAFF|192.168.10.0/24|192.168.10.1|
|20|POS|192.168.20.0/24|192.168.20.1|
|30|GUEST|192.168.30.0/24|192.168.30.1|

### SW1 — VLAN Creation

```text
enable
configure terminal

vlan 10
 name STAFF
exit

vlan 20
 name POS
exit

vlan 30
 name GUEST
exit
```

### SW1 — R1 Trunk

The R1-facing link must carry all three VLANs so that R1 can provide the Layer-3 gateways through router subinterfaces.

```text
interface GigabitEthernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit
```

### SW1 — Staff Access Ports

```text
interface range GigabitEthernet1/1 - 2
 switchport mode access
 switchport access vlan 10
 no shutdown
exit
```

### SW1 — POS Access Ports

```text
interface range GigabitEthernet2/1 - 3
 switchport mode access
 switchport access vlan 20
 no shutdown
exit
```

### SW1 — Guest/OpenWrt Connection

The OpenWrt connection carries the Guest VLAN toward the Guest network.

```text
interface GigabitEthernet3/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 30
 no shutdown
exit
```

---

## 3. Inter-VLAN Routing

R1 provides the default gateway for each internal VLAN. Router-on-a-Stick is used because a single physical interface can carry multiple VLANs through 802.1Q subinterfaces.

Each subinterface is assigned the first usable address of its corresponding /24 network.

### R1 — VLAN Subinterfaces

```text
configure terminal

interface GigabitEthernet0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/1.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 no shutdown
exit
```

---

## 4. DHCP Configuration

R1 provides DHCP services for all three internal networks. Separate pools are used so that each VLAN receives addresses from its own subnet.

The first addresses are reserved for gateways and infrastructure, while clients receive addresses from the remaining range.

### DHCP Exclusions

```text
ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp excluded-address 192.168.20.1 192.168.20.20
ip dhcp excluded-address 192.168.30.1 192.168.30.20
```

### Staff DHCP Pool

```text
ip dhcp pool STAFF
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 domain-name cafe.com
exit
```

### POS DHCP Pool

```text
ip dhcp pool POS
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
 domain-name cafe.com
exit
```

### Guest DHCP Pool

```text
ip dhcp pool GUEST
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8
 domain-name cafe.com
exit
```

---

## 5. NAT/PAT and Internet Access

The internal VLANs use private IPv4 addresses and therefore require translation before reaching the external GNS3 NAT network.

PAT is used so that all three internal networks can share the external address assigned to R1 G0/0.

### NAT ACL

```text
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.30.0 0.0.0.255
```

### NAT Inside / Outside

```text
interface GigabitEthernet0/0
 ip nat outside
exit

interface GigabitEthernet0/1.10
 ip nat inside
exit

interface GigabitEthernet0/1.20
 ip nat inside
exit

interface GigabitEthernet0/1.30
 ip nat inside
exit
```

### PAT

```text
ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

---

## 6. SSH Remote Management

R1 is the management endpoint for the lab. SSH version 2 is used with local authentication so that remote management is encrypted and does not rely on the insecure Telnet protocol.

### SSH Dependencies

- Hostname/domain configuration
    
- Local administrative account
    
- RSA key pair
    
- SSH version 2
    
- Local VTY authentication
    
- SSH-only inbound VTY access
    

### R1 — SSH Configuration

```text
configure terminal

ip domain name cafe.com
username admin privilege 15 secret cafepass

crypto key generate rsa modulus 2048
ip ssh version 2

line vty 0 4
 login local
 transport input ssh
exit

line vty 5 15
 login local
 transport input ssh
exit
```

The modern SSH client used during testing required legacy Cisco IOSv-compatible cryptographic options. This was a client-side compatibility requirement and was not part of the R1 configuration.

SW1 does not provide remote management in the final design. Its VTY lines are therefore restricted.

### SW1 — Remote VTY Restriction

```text
configure terminal

line vty 0 4
 transport input none
exit

line vty 5 14
 transport input none
exit
```

---

## 7. Guest Network

The Guest network uses VLAN 30 and is kept logically separate from the Staff and POS networks.

OpenWrt is used as the Guest wireless component. Its Guest-side connection is associated with VLAN 30, allowing Guest traffic to reach R1 without exposing the Staff or POS VLANs.

The GNS3 environment did not provide a reliable way to test an actual wireless client over RF. Therefore, Guest connectivity was validated through the available wired/test path while the wireless limitation is documented separately.

---

## 8. Network Isolation — ACL Design

The ACL design was based on the required traffic flow rather than simply blocking entire networks.

The required policy is:

|Source|Destination|Required Result|
|---|---|---|
|Staff|POS|Block|
|Staff|Guest|Block|
|POS|Staff|Block|
|POS|Guest|Block|
|Guest|Staff|Block|
|Guest|POS|Block|
|Staff|Internet|Allow|
|POS|Internet|Allow|
|Guest|Internet|Allow|
|DHCP clients|DHCP server|Allow|

Because the ACLs are applied inbound on the respective VLAN subinterfaces, each network's policy controls traffic as it enters R1 from that VLAN.

### Staff Policy

```text
ip access-list extended STAFF_POLICY
 permit udp any eq bootpc any eq bootps
 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip 192.168.10.0 0.0.0.255 any
exit
```

### POS Policy

```text
ip access-list extended POS_POLICY
 permit udp any eq bootpc any eq bootps
 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip 192.168.20.0 0.0.0.255 any
exit
```

### Guest Policy

```text
ip access-list extended GUEST_POLICY
 permit udp any eq bootpc any eq bootps
 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 permit ip 192.168.30.0 0.0.0.255 any
exit
```

### ACL Application

```text
interface GigabitEthernet0/1.10
 ip access-group STAFF_POLICY in
exit

interface GigabitEthernet0/1.20
 ip access-group POS_POLICY in
exit

interface GigabitEthernet0/1.30
 ip access-group GUEST_POLICY in
exit
```

The DHCP permit statements are placed before the deny statements so that clients can obtain addresses from the R1 DHCP service.

---

## 9. Layer-2 Security

Basic Layer-2 security is applied to the Staff and POS access ports.

The objective is to limit each access port to a single learned device, protect edge ports from unexpected spanning-tree BPDUs, and disable unused interfaces.

### Staff and POS Access Port Security

```text
interface range GigabitEthernet1/1 - 2
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 spanning-tree portfast
 spanning-tree bpduguard enable
exit

interface range GigabitEthernet2/1 - 3
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 spanning-tree portfast
 spanning-tree bpduguard enable
exit
```

Unused switch interfaces are administratively disabled.

```text
interface range GigabitEthernet0/1 - 3
 shutdown
exit
```

Additional unused interfaces are also kept administratively disabled according to the final SW1 port assignment.

---

## 10. Device Hardening

The final configuration includes basic hardening appropriate for the lab environment.

Unnecessary HTTP-based management services are disabled on SW1, while R1 management remains available through SSH.

```text
no ip http server
no ip http secure-server
```

The final configuration is saved after all changes are completed.

```text
end
copy running-config startup-config
```

---

## 11. Final Configuration Verification

The final configuration is verified from both functional and security perspectives.

### Layer 2

```text
show vlan brief
show interfaces trunk
show interfaces status
show port-security
show spanning-tree
```

### Layer 3 / DHCP

```text
show ip interface brief
show ip route
show ip dhcp binding
show ip dhcp pool
```

### NAT

```text
show ip nat translations
show ip nat statistics
```

### ACL

```text
show access-lists
```

ACL counters are checked to confirm that the intended deny and permit entries are being used.

### SSH

```text
show ip ssh
show running-config | section line vty
```

The final functional tests confirm that:

- Staff, POS, and Guest clients receive the correct addresses.
    
- Required Internet connectivity works.
    
- NAT/PAT translations are created.
    
- Staff/POS/Guest isolation policies are enforced.
    
- Layer-2 security features are operational.
    
- R1 can be managed through SSH.
    
