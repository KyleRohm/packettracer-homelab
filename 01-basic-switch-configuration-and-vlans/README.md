
# Lab 01 — Basic Switch Configuration and VLAN Assignment

## Overview

This lab uses Packet Tracer to build a simple switched network with two VLANs. Three PCs connect to the switch, with one PC assigned to VLAN 2 and two PCs assigned to VLAN 3.

The lab demonstrates basic Cisco IOS CLI navigation, VLAN creation, and connectivity verification.

## Objectives

- Build a basic switched network in Packet Tracer.
- Configure static IPv4 addresses on three PCs.
- Navigate Cisco IOS command modes.
- Create and name VLANs.
- Configure switch access ports and assign VLAN membership.
- Save the running configuration to startup configuration.
- Verify VLAN membership using show commands.
- Test connectivity within and between VLANs.

## Lab Environment

| Component | Specification |
|---|---|
| Simulator | Cisco Packet Tracer |
| Switch | Cisco Catalyst 2960 |
| End devices | Three PCs |
| Switch interfaces | FastEthernet0/1, FastEthernet0/2, FastEthernet0/3 |
| Addressing | Static IPv4 |
| VLANs | VLAN 2 and VLAN 3 |

## Network Topology

The planned topology consists of one Cisco 2960 switch connected to three PCs.

| Device | Switch Port | IPv4 Address | Subnet Mask | VLAN |
|---|---|---|---|---|
| USER-PC01 — User | Fa0/1 | 192.168.1.1 | 255.255.255.0 | 2 |
| USER-PC02 — Accounting | Fa0/2 | 192.168.1.2 | 255.255.255.0 | 3 |
| USER-PC03 — Accounting | Fa0/3 | 192.168.1.3 | 255.255.255.0 | 3 |

No default gateway is required for the intended same-VLAN connectivity tests.

## Prerequisites

- Cisco Packet Tracer is installed.
- Basic familiarity with IPv4 addressing and subnet masks.

---

## Step 1 — Build the Topology

1. Added one Cisco Catalyst 2960 switch.
2. Added two PCs
3. Connected USER-PC01 to FastEthernet0/1.
4. Connected USER-PC02 to FastEthernet0/2.

> Note: The third PC will come later.

5. Used copper straight-through cables.
6. Waited for the connections to initialize.

![Network Creation](screenshots/initial_network_setup.png)

---

## Step 2 — Configure PC IPv4 Addresses

For each PC:

1. Opened the PC.
2. Selected Desktop.
3. Opened IP Configuration.
4. Selected Static.
5. Entered an IP address and subnet mask.
6. Left the default gateway blank.

**Verification:** Opened cmd and ran ipconfig:

![USER-PC01 IP Address](screenshots/PC_1_IP.png)
![USER-PC02 IP Address](screenshots/PC_2_IP.png)

Verified connectivity by successfully pinging one PC from the other.

![USER-PC02 IP Address](screenshots/pingable.png)

---

## Step 3 — Access the Switch CLI and Create VLANs

Opened the switch and selected the CLI tab.

Created VLAN 2 & 3 for Users and Accounting:

1. Entered `en` to access privileged mode and then `config t` to configure the terminal.
2. Typed `vlan 2` and named it **User-VLAN** → `Exit`.
3. Typed `vlan 3` and named it **Accounting-VLAN** → `Exit` → `Exit`.
4. Entered `sh vlan` to show the newly created vlans.

![VLAN Creation](screenshots/show_vlan.png)

---

## Step 4 — Assign Ports to VLANs

Assigned each computer to their respective VLANs

1. Opened the switch CLI and entered privileged mode.
2. Typed `conf t` to configure the terminal.
3. Typed `int fa0/2` to select the port.
4. Entered `switchport mode access` and then `switchport access vlan 3` to assign the fa/02 port to VLAN 3.
5. Saved the configuration to memory by typing `do wr`

> Note: This process was repeated by assigning fa0/1 to VLAN 2.

![Assigned Ports To VLANs](screenshots/switchport_access.png)

**Verification:** The configuration was verified by attempting (and failing) to ping one computer from the other.

![Unable To Ping](screenshots/cannot_reach.png)

---

## Step 5 — Create a Third PC and Assign to VLAN 3

A second accounting PC was created, assigned a static IP, and configured to VLAN 3.

```cisco
Switch>
Switch>en
Switch#conf t
Switch(config)#int fa0/3
Switch (config-if)#switchport mode access
Switch (config-if)#switchport access vlan 3
Switch (config-if)#do wr
Building configuration...
[OK]
```

**Verification**:

![3rd PC](screenshots/third_pc.png)

## Verification Checklist

- [✔] Three PCs are connected to the Cisco 2960 switch.
- [✔] Each PC is assigned a static IPv4 address.
- [✔] VLAN 2 exists and is named USER-VLAN.
- [✔] VLAN 3 exists and is named ACCOUNTING-VLAN.
- [✔] Fa0/1 belongs to VLAN 2.
- [✔] Fa0/2 belongs to VLAN 3.
- [✔] Fa0/3 belongs to VLAN 3.
- [✔] USER-PC02 can ping USER-PC03.
- [✔] USER-PC01 cannot ping USER-PC02 or USER-PC03.
- [✔] The running configuration is saved to startup configuration.

---
