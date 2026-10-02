

This project simulates a small café network designed to provide separate and controlled network access for different operational areas of the café.

The network is divided into three logical segments:

| Network | Purpose                                                |
| ------- | ------------------------------------------------------ |
| Staff   | Internal access for café employees and staff devices   |
| POS     | Isolated network for billing and Point-of-Sale systems |
| Guest   | Internet access for café customers                     |

Each network is assigned a separate VLAN and IP subnet. The lab implements inter-VLAN routing, DHCP, NAT/PAT, Internet connectivity, and ACL-based traffic isolation. The design also includes a Guest Wi-Fi concept using OpenWrt as the wireless component.

### Implementation Notes

The original design included additional management and wireless considerations. During implementation, some adjustments were required because of GNS3 limitations and the availability of hardware/resources.

- OpenWrt is included to represent the Guest wireless infrastructure, but actual Wi-Fi/RF client connectivity was not tested in GNS3.
- The dedicated Management VLAN was removed from the final implementation to keep the lab aligned with the available environment and project scope.
- The final configuration therefore focuses on the core café requirements: Staff, POS, Guest, Internet access, segmentation, isolation, and basic device security.


## Network Addressing

The café network uses a single private address block, `192.168.0.0/16`, which is divided into separate /24 subnets for each functional network.

### Final Network Allocation

| VLAN | Network / Subnet | Default Gateway | Primary Interface | Purpose |
|---|---|---|---|---|
| VLAN 10 | 192.168.10.0/24 | 192.168.10.1 | R1 G0/1.10 | Staff Network |
| VLAN 20 | 192.168.20.0/24 | 192.168.20.1 | R1 G0/1.20 | POS / Billing Network |
| VLAN 30 | 192.168.30.0/24 | 192.168.30.1 | R1 G0/1.30 | Guest Network |
| — | DHCP / External | DHCP | R1 G0/0 | Internet / GNS3 NAT |

The three internal networks are routed through R1 using 802.1Q subinterfaces. DHCP services are provided by R1, while Internet access is provided through the GNS3 NAT connection on R1 G0/0.

### Removed Management Subnet

An additional Management VLAN was included in the initial design but was removed from the final implementation.

| VLAN | Original Subnet | Original Purpose | Final Status |
|---|---|---|---|
| VLAN 99 | 192.168.99.0/24 | Device Management | Removed |

The Management VLAN was removed before the final configuration. The final lab therefore uses VLANs 10, 20, and 30 for the café's operational networks without a dedicated management subnet.

# Café Network — Lab Configuration Objectives

## Objective 1 — Physical & Logical Network Setup

Establish the topology and prepare the basic Layer-2 and Layer-3 network structure.

### 1.1 Interface Connections

| Connection | Interface | Interface | Type / Purpose |
|---|---|---|---|
| GNS3 NAT → R1 | — | R1 G0/0 | External / DHCP |
| R1 → SW1 | R1 G0/1 | SW1 G0/0 | 802.1Q Trunk |
| SW1 → Staff | SW1 Gi1/1–Gi1/2 | Staff devices | Access — VLAN 10 |
| SW1 → POS | SW1 Gi2/1–Gi2/3 | POS devices | Access — VLAN 20 |
| SW1 → OpenWrt | SW1 Gi3/0 | OpenWrt eth0 | Guest VLAN 30 |

### 1.2 VLAN and IP Network

Create the required VLANs and assign the corresponding IP networks and gateway interfaces.

| VLAN | Name | Network | Gateway |
|---|---|---|---|
| 10 | STAFF | 192.168.10.0/24 | 192.168.10.1 |
| 20 | POS | 192.168.20.0/24 | 192.168.20.1 |
| 30 | GUEST | 192.168.30.0/24 | 192.168.30.1 |

Configure R1 G0/0 as a DHCP client to obtain the external IP address from the GNS3 NAT network.

Configure R1 subinterfaces for VLANs 10, 20, and 30 and establish the required trunk between R1 and SW1.

Assign the corresponding access ports on SW1 to the Staff and POS VLANs.

---

## Objective 2 — Network Services & Protocol Configuration

Configure the core services required for the café network.

### 2.1 DHCP

Configure DHCP on R1 for the three internal networks.

- Create separate DHCP pools for:
  - STAFF — `192.168.10.0/24`
  - POS — `192.168.20.0/24`
  - GUEST — `192.168.30.0/24`
- Reserve the gateway and required infrastructure addresses.
- Provide the appropriate default gateway and DNS information to clients.

### 2.2 Inter-VLAN Routing

Use R1 router subinterfaces to provide Layer-3 gateways for VLANs 10, 20, and 30.

Verify that the required VLAN interfaces are operational and routing is functioning.

### 2.3 NAT/PAT

Configure NAT overload on R1 so that the internal Staff, POS, and Guest networks can access the Internet through the GNS3 NAT connection.

### 2.4 SSH

Configure secure remote management for R1.

Required dependencies:

- Hostname/domain configuration
- Local administrative user
- RSA key pair
- SSH version 2
- VTY local authentication
- SSH-only VTY access

SW1 remote management is not required in the final design.

---

## Objective 3 — Guest Network

Configure the Guest network as a separate network using VLAN 30.

- Connect OpenWrt to the Guest VLAN.
- Provide Guest clients with DHCP addresses through R1.
- Ensure Guest traffic can reach the Internet.
- Prevent Guest traffic from accessing Staff and POS networks.

Actual wireless/RF client testing is outside the final GNS3 validation because of the available environment.

---

## Objective 4 — Network Isolation & Security Policies

Implement the required traffic restrictions between the internal networks.

- Staff → POS: Block
- Staff → Guest: Block
- POS → Staff: Block
- POS → Guest: Block
- Guest → Staff: Block
- Guest → POS: Block
- Required Internet access: Allow

Use extended ACLs on the appropriate R1 VLAN interfaces to enforce these policies.

---

## Objective 5 — Layer-2 Security

Apply basic switch security controls to the Staff and POS access ports.

- Port Security
- Maximum one MAC address per access port
- Violation action: shutdown
- PortFast
- BPDU Guard
- Unused switch ports administratively disabled

Verify that the configured security features are operational.

---

## Objective 6 — Final Hardening

Apply basic device hardening appropriate for the lab.

- Disable unnecessary HTTP/HTTPS management services.
- Restrict SW1 VTY access.
- Keep R1 management accessible through SSH only.
- Remove unnecessary configuration left from earlier design iterations.
- Save the final running configuration to startup configuration.

---

## Objective 7 — Verification & Documentation

Perform final functional testing of the complete network.

Verify:

- VLAN and trunk operation
- DHCP address assignment
- Inter-VLAN routing
- Internet connectivity
- NAT/PAT translations
- ACL isolation
- Guest network connectivity
- Port Security and Layer-2 protections
- R1 SSH access
- Final device configuration

Document the final topology, addressing, implemented features, limitations, and verification results.
