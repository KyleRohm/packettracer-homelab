
# Lab 02: Small Office Network — DHCP, Wireless Connectivity, and Network Printer

---

## Overview

This lab builds on the basic switching and VLAN concepts introduced in Lab 01 by simulating a small office network in Cisco Packet Tracer.

The network includes three wired PCs, a network printer, a Cisco router, a wireless router, and a laptop connected over Wi-Fi.

The lab focuses on IPv4 addressing, subnetting, router interface configuration, DHCP, wireless security, and end-to-end connectivity testing.

---

## Objectives

By completing this lab, I will demonstrate the ability to:

- Build a small office network topology in Cisco Packet Tracer.
- Connect end devices, switches, and routers using appropriate cables.
- Configure Cisco router interfaces with static IPv4 addresses.
- Configure DHCP pools on a Cisco IOS router.
- Configure a wireless router's SSID and WPA2-Personal security.
- Connect a laptop to a wireless network.
- Assign a static IPv4 address to a network printer.
- Verify addressing and connectivity using `ipconfig` and `ping`.
- Save and verify network device configurations.
- Troubleshoot common addressing and connectivity problems.

---

## Network Topology

### Devices

| Device | Quantity | Purpose |
|---|---:|---|
| Cisco 2911 router | 1 | Routes traffic both LANs and provides DHCP for the wired LAN |
| Cisco 2960 switch | 1 | Connects the wired office devices |
| WRT300N wireless router | 1 | Provides wireless connectivity and DHCP for the wireless LAN |
| PCs | 3 | Wired client devices |
| Network printer | 1 | Printer with a static IPv4 address |
| Laptop | 1 | Wireless client |

### Topology design

The network will use two IPv4 subnets:

1. **Wired LAN:** `192.168.1.0/24`
3. **Wireless LAN:** `192.168.2.0/24`

### Connections

| Source device | Destination device | Cable | Port |
|---|---|---|---|
| PC1 | Switch | Copper straight-through | PC FastEthernet0 → Switch Fa0/1 |
| PC2 | Switch | Copper straight-through | PC FastEthernet0 → Switch Fa0/2 |
| PC3 | Switch | Copper straight-through | PC FastEthernet0 → Switch Fa0/3 |
| Printer | Switch | Copper straight-through | Printer FastEthernet0 → Switch Fa0/4 |
| Switch | Cisco 2911 router | Copper straight-through | Switch GigabitEthernet0/1 → Router GigabitEthernet0/0 |
| Cisco 2911 router | WRT300N | Copper straight-through | Router GigabitEthernet0/1 → WRT300N Internet port |
| Laptop | WRT300N | Wireless | N/A |

---

## Step 1: Add the Devices

1. Added a Cisco 2911 router, Cisco 2960 switch and a WRT300N wireless router.
2. Added three PCs, one printer, and one laptop.
3. Arranged the devices so that the wired LAN and wireless LAN are easy to identify.

---

## Step 2: Connect the Wired Devices

1. Used copper straight-through cables to connect the three PCs and printer to the switch.
2. Connected the switch to the Cisco 2911 router.
3. Connected the Cisco 2911 router's `GigabitEthernet0/1` interface to the WRT300N's **Internet** port.

---

## Step 3: Configure the Cisco Router Interfaces

Accessed the Cisco 2911 router's CLI and configured the wired LAN interface

```text
enable
configure terminal
interface gigabitEthernet 0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
exit
```

![Wired LAN Interface Setup](screenshots/wired_lan_setup.png)

**Explanation:**

- `interface gigabitEthernet 0/0` selects the interface connected to the switch.
- `ip address` assigns the router's address on the wired LAN.
- `no shutdown` enables the router.
- `exit` returns to global configuration mode.

> The router's `192.168.1.1` address will serve as the wired clients' default gateway.

---

## Step 4: Configure DHCP for the Wired LAN

Excluded the printer's static address and a small range of addresses reserved for infrastructure.

```text
configure terminal
ip dhcp excluded-address 192.168.1.1 192.168.1.20
```

Created the wired DHCP pool and saved the configuration:

```text
ip dhcp pool WIRED-LAN
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
exit
end
copy running-config startup-config
```

> When prompted for the destination filename, pressed Enter to accept the default.

**Explanation:**

- `ip dhcp excluded-address` prevents the router from leasing reserved addresses to DHCP clients.
- `ip dhcp pool WIRED-LAN` creates a named DHCP pool.
- `network` specifies the subnet served by the pool.
- `default-router` supplies the default gateway to DHCP clients.

> The excluded range leaves `192.168.1.21` through `192.168.1.254` available for DHCP leases.

![Wired LAN and DCHP Configuration](screenshots/LAN_DHCP_Config.png)

### Configure the wired PCs for DHCP

For each PC:

* Opened the PC, selected **Desktop**, opened **IP Configuration**, and selected **DHCP**.
* Confirmed that an IPv4 address, subnet mask, and default gateway were displayed.

> The PCs should receive addresses in `192.168.1.0/24`, excluding the reserved addresses.

### Verify the DHCP configuration

On the Cisco router, ran:

```text
show ip dhcp binding
show ip dhcp pool
show running-config
```

Confirm that the pool exists and that client leases appear after the PCs request DHCP addresses.

![Verified DHCP Config](screenshots/Verified_DHCP_Config.png)

---

## Step 5: Configure the Wireless Network

The WRT300N has separate Internet-facing and internal LAN interfaces. A cross over cable was used to connect the WRT300N Internet interface connects to the Cisco 2911 router's Gigabit Ethernet 0/1 interface.

### Configure the SSID, security, and IP address

1. Opened the WRT300N's **GUI** tab and set the SSID as `Office WiFi`.

![WiFi Name](screenshots/WiFi_name.png)

2. Set the Security Mode to WPA2 Personal and created a password.

![Security](screenshots/Security.png)

3. Opened the **Setup** tab and entered the following values:

| Setting | Value |
|---|---|
| Internet IP address | 192.168.2.1 |
| Subnet mask | 255.255.255.0 |

> The Cisco 2911 will be serving as the network's default gateway.

![IP Address](screenshots/IP_address.png)

---

## Step 6: Connect the Laptop to Wi-Fi

### Installed Wireless module

1. Opened the laptop and selected **Physical**.
2. Powered the laptop off.
3. Remove the existing module and replaced with a `WPC300N` module.
5. Powered the laptop back on.

### Connected the Laptop to Wi-Fi

1. Selected the **Desktop** tab.
2. Clicked the **Connect** tab and refreshed the list of available networks.
3. Connected to `Office-WiFi` and entered the password.

![Wireless Signal](screenshots/Wireless_Signal.png)

### Verified Laptop is Using DHCP

![Laptop DHCP](screenshots/Laptop_DHCP.png)

---

## Step 7: Configure a Static IP Address for the Printer

Network printers are often assigned a reserved address so that client computers can consistently locate them.

Open the printer in Packet Tracer.

1. Select **Config**.
2. Select its wired Ethernet interface, typically **FastEthernet0**.
3. Configure the static IPv4 address:

| Setting | Value |
|---|---|
| IPv4 address | 192.168.1.10 |
| Subnet mask | 255.255.255.0 |
| Default gateway | 192.168.1.1 |

If the interface screen provides a default-gateway field, enter `192.168.1.1`.

The printer address is within the wired LAN but outside the excluded DHCP reservation range and the available DHCP lease range configured above.

**Why this matters:** A stable address makes it easier for users and client devices to locate a shared network printer. In a production network, DHCP reservations or centralized IP address management can also be used to prevent address conflicts.

---

## Step 10: Verify IPv4 Addressing

On each wired PC, open **Desktop → Command Prompt** and run:

```text
ipconfig
```

Confirm that each PC has:

- An address in `192.168.1.0/24`.
- Subnet mask `255.255.255.0`.
- Default gateway `192.168.1.1`.

On the laptop, run `ipconfig` and confirm that it has an address in `192.168.2.0/24` with gateway `192.168.2.1`.

On the Cisco router, verify:

```text
show ip interface brief
show ip dhcp binding
show ip route
```

The routing table should include connected routes for `192.168.1.0/24` and `192.168.10.0/24` when the interfaces are up.

The Cisco router does not automatically learn the WRT300N's internal wireless subnet through a dynamic routing protocol in this lab. The WRT300N uses NAT by default in its typical gateway configuration.

---

## Step 11: Test Connectivity

### Test 1: Wired PC to default gateway

From a wired PC's command prompt:

```text
ping 192.168.1.1
```

**Expected result:** Replies from the Cisco router's wired interface.

### Test 2: Wired PC to printer

From a wired PC:

```text
ping 192.168.1.10
```

**Expected result:** Replies from the printer, provided its interface is correctly configured and online.

### Test 3: Laptop to wireless default gateway

From the laptop:

```text
ping 192.168.2.1
```

**Expected result:** Replies from the WRT300N's internal LAN interface.

### Test 4: Wireless-to-wired connectivity

From the laptop, test:

```text
ping 192.168.10.1
ping 192.168.1.10
```

These tests can help identify whether the wireless router's WAN connection and upstream network are reachable. Results depend on the WRT300N's NAT behavior, Packet Tracer implementation, and routing configuration. Do not assume that successful Wi-Fi association alone proves end-to-end access to the wired LAN.

If the goal is to permit or prevent wireless clients from accessing the wired LAN, implement and test an explicit network security policy rather than treating subnet separation or NAT as a complete security control.

---

## Step 12: Save the Configuration

On the Cisco router, save the running configuration:

```text
copy running-config startup-config
```

Press Enter to accept the default destination filename.

Save the Packet Tracer project using **File → Save As**. Use a descriptive filename such as:

```text
02-small-office-network-dhcp-and-wireless.pkt
```

Save screenshots of the completed topology, router interfaces, DHCP leases, wireless settings, and successful ping tests in a screenshots folder for this lab.

---

## Troubleshooting

| Symptom | What to check |
|---|---|
| Router interface is down | Confirm the cable, connected port, and `no shutdown` command |
| PC receives no DHCP address | Check the PC's DHCP setting, router interface status, DHCP pool network, and excluded addresses |
| PC has an incorrect default gateway | Verify the `default-router` statement in the relevant DHCP pool |
| Printer cannot be pinged | Check the printer's static IP, subnet mask, cabling, switch port, and possible address conflicts |
| Laptop cannot see the SSID | Check the SSID, wireless module, wireless settings, and laptop power state |
| Laptop cannot associate with Wi-Fi | Verify the security mode and passphrase on both devices |
| Wireless laptop has no IPv4 address | Confirm the WRT300N LAN IP and DHCP server settings |
| Two devices have the same address | Check the addressing plan, static assignments, and DHCP exclusions |
| Wireless client cannot reach a wired device | Check the WRT300N WAN configuration, NAT behavior, upstream connectivity, and routing design |

---

## Key Concepts Learned

### DHCP

DHCP automatically supplies IPv4 addressing information to clients. It reduces manual configuration and makes it easier to manage a network with multiple devices.

### Default gateway

A default gateway provides a path to destinations outside a device's local subnet. A client generally needs the correct gateway to communicate with other routed networks.

### Static IP addressing

A static address can make infrastructure devices such as printers easier to locate. Static assignments must be coordinated with DHCP so that two devices do not receive the same address.

### Wireless security

An SSID identifies a wireless network, while wireless security settings control how clients authenticate and protect their connection.

### Network segmentation

Using separate IPv4 subnets helps organize a network and creates boundaries for routing and policy enforcement. Different subnets alone do not guarantee security; access controls must be configured and tested.

### Verification and troubleshooting

Commands such as `ipconfig`, `ping`, `show ip interface brief`, `show ip dhcp binding`, and `show ip route` help identify whether addressing, interfaces, DHCP, and connectivity are working as expected.

---

## Lab Completion Checklist

- [ ] Added the router, switch, wireless router, three PCs, printer, and laptop.
- [ ] Connected the wired devices and router interfaces.
- [ ] Configured the Cisco router's wired and transit interfaces.
- [ ] Enabled the router interfaces and verified their status.
- [ ] Configured DHCP for the wired LAN.
- [ ] Confirmed that all three wired PCs received valid DHCP addresses.
- [ ] Configured the WRT300N Internet and LAN interfaces with separate subnets.
- [ ] Configured the SSID and wireless security.
- [ ] Connected the laptop to Wi-Fi.
- [ ] Confirmed that the laptop received a valid wireless DHCP address.
- [ ] Assigned the printer a static IPv4 address.
- [ ] Tested wired gateway and printer connectivity.
- [ ] Tested wireless gateway and upstream connectivity.
- [ ] Saved the router configuration and Packet Tracer project.
- [ ] Captured screenshots documenting the completed configuration and test results.

---

## Evidence and Screenshots

Add screenshots here after performing and verifying the configurations in Packet Tracer.

Suggested screenshots:

1. `01-topology.png` — Full network topology.
2. `02-router-interfaces.png` — Router interface configuration and status.
3. `03-dhcp-bindings.png` — DHCP bindings for the wired clients.
4. `04-wireless-security.png` — SSID and wireless security settings, with the passphrase hidden.
5. `05-laptop-ipconfig.png` — Laptop wireless addressing.
6. `06-printer-ipconfig.png` — Printer static IP configuration.
7. `07-connectivity-tests.png` — Successful ping tests.

---

## Conclusion

This lab demonstrates how a small office network can use a switch for wired connectivity, a router for Layer 3 communication, DHCP for automatic client addressing, and a wireless router for Wi-Fi connectivity.

The lab also reinforces the importance of a consistent IP addressing plan, static addressing for selected infrastructure devices, configuration backups, and systematic verification.

The next step is to build on this foundation with additional switching, VLAN configuration, access controls, and more advanced routing and security concepts.
