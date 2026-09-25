

**Objective 1 — Network Segmentation**

1.1 Configure separate VLANs for Staff, POS, and Guest networks.

1.2 Use the approved VLAN IDs and IP addressing plan.

1.3 Configure the appropriate Layer-3 gateway for each VLAN.

---

**Objective 2 — Switch Port Assignment**

2.1 Assign Staff endpoint ports to VLAN 10.

2.2 Assign POS endpoint ports to VLAN 20.

2.3 Provide the required VLAN 30 connectivity for the Guest network.

2.4 Disable unused switch ports.

---

**Objective 3 — Trunk Connectivity**

3.1 Establish the required 802.1Q trunk between R1 and SW1.

3.2 Allow VLANs 10, 20, and 30 across the R1–SW1 trunk.

3.3 Provide VLAN 30 trunk connectivity between SW1 and OpenWrt.

---

**Objective 4 — Inter-VLAN Routing**

4.1 Configure Router-on-a-Stick on R1 for VLANs 10, 20, and 30.

4.2 Provide the designated gateway for each VLAN.

4.3 Apply the required traffic policies to control communication between VLANs.

---

**Objective 5 — DHCP**

5.1 Provide DHCP addressing for Staff devices.

5.2 Provide DHCP addressing for POS devices.

5.3 Provide DHCP addressing for Guest devices.

5.4 Exclude reserved infrastructure addresses from DHCP allocation.

---

**Objective 6 — Guest Network Connectivity**

6.1 Provide a dedicated Guest network using VLAN 30.

6.2 Connect the OpenWrt device to the Guest VLAN as an L2 bridge.

6.3 Deliver Guest client addressing and gateway information through R1.

6.4 Verify Guest client connectivity through the OpenWrt bridge.

---

**Objective 7 — Internet Connectivity**

7.1 Provide Internet access to Staff, POS, and Guest networks.

7.2 Ensure Guest clients can reach the Internet while remaining isolated from internal networks.

7.3 Verify end-to-end Internet connectivity from the required client networks.

---

**Objective 8 — Network Address Translation**

8.1 Configure NAT/PAT for the internal client networks.

8.2 Translate private client addresses to the external interface address.

8.3 Verify that multiple internal clients can access the Internet through PAT.

---

**Objective 9 — Network Isolation**

9.1 Prevent Staff clients from accessing the POS network.

9.2 Prevent Staff clients from accessing the Guest network.

9.3 Prevent POS clients from accessing the Staff network unless specifically permitted by the final policy.

9.4 Prevent POS clients from accessing the Guest network.

9.5 Prevent Guest clients from accessing Staff and POS networks.

10.6 Allow permitted Internet traffic from the isolated networks.

---

**Objective 10 — POS Network Protection**

10.1 Apply the required ACL policy to protect the POS network.

10.2 Prevent Guest users from initiating communication with POS devices.

10.3 Prevent unauthorized communication between POS and other internal VLANs.

10.4 Allow POS Internet access required by the café design.

---

**Objective 11 — Switch Port Security**

11.1 Enable Port Security on designated Staff and POS access ports.

11.2 Limit protected ports to one MAC address.

11.3 Configure the required violation action.

11.4 Verify that protected ports operate in the expected secure state.

---

**Objective 12 — Edge-Port Protection**

12.1 Configure PortFast on designated endpoint access ports.

12.2 Configure BPDU Guard on designated endpoint access ports.

12.3 Verify that protected edge ports remain in the expected forwarding state.

---

**Objective 13 — Secure Device Management**

13.1 Enable secure remote SSH management on R1.

13.2 Use authenticated administrative access for R1.

13.3 Verify RSA-based SSH operation.

13.4 Disable remote management access on SW1 where it is not required.

---

**Objective 14 — Device Hardening**

14.1 Configure appropriate device identification and administrative controls.

14.2 Protect privileged administrative access.

14.3 Restrict unnecessary remote-access methods.

14.4 Disable unnecessary HTTP/HTTPS management services on SW1.

14.5 Disable unused switch interfaces.

---

**Objective 15 — Verification and Documentation**

15.1 Verify VLAN membership and Layer-2 connectivity.

15.2 Verify trunk operation and VLAN propagation.

15.3 Verify IP addressing and DHCP operation.

15.4 Verify Router-on-a-Stick and gateway operation.

15.5 Verify NAT/PAT and Internet connectivity.

15.6 Verify ACL-based Staff, POS, and Guest isolation.

15.7 Verify Port Security, PortFast, and BPDU Guard.

15.8 Verify R1 SSH management.

15.9 Verify OpenWrt VLAN 30 bridge operation.

15.10 Document the final topology, addressing, device roles, configurations, and verification evidence.

15.11 Record significant deviations from the original design and the reason for the change.