
![](image%20ref/cafe%20topology.png)
### Experience Journey

- **OpenWrt in GNS3** — Installed OpenWrt as a virtual network device in GNS3 and explored its integration into the lab network.
- **SSH Testing Environment** — Set up a lightweight Linux virtual machine in GNS3 specifically to test SSH connectivity with the Cisco devices.
- **SSH Troubleshooting** — Encountered an SSH compatibility issue during the initial connection attempt. After identifying and aligning the required SSH settings, a successful SSH connection to **R1** was established.
- **OpenWrt GUI Access Attempt** — Tried to access the OpenWrt web interface from the Linux Mint host by testing different virtual connectivity paths and interfaces. The investigation did not result in reliable browser access, but provided practical experience with virtual networking and troubleshooting.

**Overall:** The project involved more than configuring the final network. It also provided hands-on experience with deploying virtual network systems, testing real connectivity, troubleshooting compatibility issues, and investigating problems across the GNS3 and Linux networking environment.

### Lab Environment
![](image%20ref/gns3_toptology.png)

| Component          | Device / Software | Version                                   |
| ------------------ | ----------------- | ----------------------------------------- |
| Lab platform       | GNS3              | 2.2.61                                    |
| Router             | Cisco IOSv        | IOS version used in the lab               |
| Layer-2 Switch     | Cisco IOSvL2      | IOS version used in the lab               |
| Guest/AP appliance | OpenWrt           | 25.12.0                                   |
| SSH test client    | Alpine Linux      | Lightweight Linux VM used for SSH testing |
| Network clients    | VPCS              | GNS3 built-in VPCS                        |
| External network   | GNS3 NAT          | GNS3 built-in NAT node                    |
| Host OS            | Linux Mint        | 22.3 (Zena)                               |



### ACL config

![](image%20ref/cafe%20acl%20policy.png)

