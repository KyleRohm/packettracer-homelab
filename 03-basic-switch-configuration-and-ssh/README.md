# Lab 03: Basic Switch Configuration and SSH

## Overview

This lab covers the initial configuration of a Cisco Catalyst 2960 switch using Cisco Packet Tracer.

The lab simulates the process of setting up a new switch or preparing a replacement switch for deployment. Initial configuration is performed through a console connection, after which the switch is configured for remote management using Secure Shell (SSH).

The lab covers basic Cisco IOS commands, hostname configuration, password protection, management IP addressing, console access, SSH authentication, and configuration verification.

## Objectives

- Establish a console connection to a Cisco switch.
- Inspect the initial switch configuration using Cisco IOS show commands.
- Configure a hostname and basic security settings.
- Configure console access with password authentication.
- Assign a management IP address to the switch.
- Configure a default gateway for remote management from other subnets.
- Create a local administrator account.
- Generate RSA keys and enable SSH version 2.
- Restrict remote management access to SSH.
- Verify connectivity and test remote SSH access.
- Save the running configuration to startup configuration.

## Prerequisites

- Cisco Packet Tracer installed.
- Basic familiarity with Cisco IOS command-line interface (CLI) navigation.
- Understanding of IPv4 addresses and subnet masks.

## Network Topology

### Devices

| Device | Quantity | Purpose |
|---|---:|---|
| Cisco Catalyst 2960 switch | 1 | Provides Layer 2 switching and a management interface |
| PC | 1 | Provides console access and tests network connectivity and SSH |

### Topology design

This lab uses a single switch and a PC to demonstrate initial switch configuration and remote management.

The PC initially connects to the switch through a console cable. After configuration, an Ethernet connection is used to test IP connectivity and SSH access.

### IP addressing

| Device | Interface | IPv4 address | Subnet mask | Default gateway |
|---|---|---|---|---|
| Switch01 | VLAN 1 (management SVI) | `192.168.1.5` | `255.255.255.0` | `192.168.1.1` |
| PC | FastEthernet0 | `192.168.1.3` | `255.255.255.0` | Not required for local testing |

**Important:** The switch's default gateway, `192.168.1.1`, is an example address. A router or Layer 3 device must actually use that address on the management subnet for remote-subnet management to work.

The switch and PC use addresses in the same subnet, `192.168.1.0/24`, allowing them to communicate directly without a router.

> **Lab isolation:** This lab uses the same address range as Lab 02. If you combine the labs in one Packet Tracer topology, ensure that the IP addresses do not conflict with existing devices. Otherwise, keep this lab in its own Packet Tracer file.

---

## Step 1: Add the Devices

1. Open Cisco Packet Tracer.
2. Add one Cisco Catalyst 2960 switch.
3. Add one PC.
4. Arrange the devices in the workspace.

The switch will initially be configured through a console connection before Ethernet connectivity is tested.

---

## Step 2: Establish a Console Connection

### Connect the PC to the switch

1. Open the **Connections** menu.
2. Select the light-blue console cable.
3. Connect the PC's `RS-232` port to the switch's `Console` port.

### Open the switch terminal

1. Click the PC.
2. Open the **Desktop** tab.
3. Select **Terminal**.
4. Leave the default serial settings unchanged and click **OK**.
5. Press Enter to access the switch CLI.

The console connection provides out-of-band management, meaning the switch can be configured without an existing working IP connection.

This is particularly useful when configuring a new switch before its management interface has been assigned an IP address.

---

## Step 3: Inspect the Initial Switch Configuration

Enter privileged EXEC mode:

```text
enable
```

Privileged EXEC mode provides access to administrative commands and configuration inspection.

### View the running configuration

```text
show running-config
```

Displays the configuration currently active in memory.

### View the startup configuration

```text
show startup-config
```

Displays the configuration saved in NVRAM for use when the switch starts.

A new switch may report that the startup configuration is not present. This is normal before an initial configuration has been saved.

### Check the IOS version

```text
show version
```

Displays information about the switch, including its IOS version, hardware platform, and uptime.

### View VLAN information

```text
show vlan brief
```

Displays the VLANs configured on the switch and their associated access ports.

A switch with its default configuration will typically contain VLAN 1 and the default VLAN entries.

These commands establish a baseline for comparing the switch's configuration before and after the lab.

---

## Step 4: Configure Basic Switch Settings

Enter global configuration mode:

```text
configure terminal
```

Global configuration mode allows changes to the switch's settings.

### Configure the hostname

```text
hostname Switch01
```

Changes the switch's hostname from the default `Switch` to `Switch01`.

A meaningful hostname makes it easier to identify devices when managing multiple switches and routers.

### Enable password encryption for supported configuration passwords

```text
service password-encryption
```

This command obscures supported plaintext passwords in the configuration display.

**Security note:** This command does not provide strong password protection. The Type 7 encryption used for many line passwords is reversible and should not be treated as secure cryptographic storage. Use secret-based authentication where supported.

### Configure the privileged EXEC password

```text
enable secret LabEnable-ChangeMe
```

Sets the password required to enter privileged EXEC mode.

The `enable secret` command stores the secret in a protected form, unlike the legacy `enable password` command.

### Disable unwanted DNS lookups

```text
no ip domain-lookup
```

Prevents the switch from attempting to resolve mistyped commands as domain names.

Without this setting, an incorrectly entered command may cause the CLI to pause while the switch attempts a DNS lookup.

### Configure a Message of the Day banner

```text
banner motd #Unauthorized access is strictly prohibited.#
```

Displays a warning message when someone connects to the switch.

The `#` characters mark the beginning and end of the banner message. Another delimiter can be used as long as it does not appear within the message itself.

Banners are commonly used to communicate authorized-use policies and security warnings.

> Use unique lab-only passwords. Do not reuse these example values on production devices or real accounts.

---

## Step 5: Configure Console Access

Configure password authentication for the console line:

```text
line console 0
password LabConsole-ChangeMe
login
logging synchronous
exit
```

### Explanation

| Command | Purpose |
|---|---|
| `line console 0` | Enters console-line configuration mode. |
| `password LabConsole-ChangeMe` | Sets the console line password. |
| `login` | Requires the configured line password when accessing the console. |
| `logging synchronous` | Keeps asynchronous console messages from disrupting command entry. |
| `exit` | Returns to global configuration mode. |

The `login` command is important because setting a line password alone does not necessarily enable password authentication.

### Verify console authentication

After configuring the console line, disconnect from the terminal session and reconnect, or reopen the terminal session as appropriate in Packet Tracer.

The switch should display the configured banner and prompt for the console password.

After successful authentication, enter:

```text
enable
```

Enter the privileged EXEC password configured in Step 4.

This verifies that console authentication and privileged EXEC authentication are separate controls.

---

## Step 6: Configure the Switch Management IP Address

A Layer 2 switch forwards Ethernet frames between devices, but administrators also need an IP address to manage the switch remotely.

On a Layer 2 switch, the management IP address is typically configured on a switched virtual interface (SVI), rather than on an individual physical switch port.

### Inspect the current interface status

```text
end
show ip interface brief
```

This command summarizes the IP addresses and operational status of the switch's interfaces.

Before configuration, VLAN 1 may not have an IP address and may be administratively down.

### Configure VLAN 1

```text
configure terminal
interface vlan 1
ip address 192.168.1.5 255.255.255.0
no shutdown
exit
```

### Explanation

| Command | Purpose |
|---|---|
| `interface vlan 1` | Enters configuration mode for the VLAN 1 management SVI. |
| `ip address 192.168.1.5 255.255.255.0` | Assigns the switch's management IP address and subnet mask. |
| `no shutdown` | Administratively enables the SVI. |
| `exit` | Returns to global configuration mode. |

The subnet mask `255.255.255.0` corresponds to a `/24` network.

The switch management address is `192.168.1.5`, and the network address is `192.168.1.0`.

**Important:** `no shutdown` does not guarantee that the SVI will be operationally up. VLAN 1 must also be active, and at least one switch port in VLAN 1 must have an active link for the SVI to become operational on a typical Layer 2 switch.

### Verify the management interface

```text
end
show ip interface brief
```

Confirm that VLAN 1 has the address `192.168.1.5`.

Ideally, its status and protocol should both show `up` after the Ethernet connection is established and an active port belongs to VLAN 1.

---

## Step 7: Connect the PC Using Ethernet

The console cable is used for initial configuration. To test IP connectivity and SSH, connect the PC to a switch Ethernet port.

### Connect the Ethernet cable

1. Open **Connections**.
2. Select a copper straight-through cable.
3. Connect the PC's `FastEthernet0` interface to the switch's `FastEthernet0/1` interface.

The PC's Ethernet interface and the switch port must be connected and operational for network communication to work.

### Configure the PC's IPv4 address

1. Click the PC.
2. Open **Desktop → IP Configuration**.
3. Select **Static**.
4. Enter the following values:

| Setting | Value |
|---|---|
| IPv4 address | `192.168.1.3` |
| Subnet mask | `255.255.255.0` |
| Default gateway | Leave blank for this local test |

The PC and switch now have addresses in the same subnet.

No default gateway is necessary for communication between these two devices because they are on the same local network.

### Test connectivity

Open **Desktop → Command Prompt** on the PC and run:

```text
ping 192.168.1.5
```

A successful test should return replies from the switch management IP address.

The first ping may time out while ARP resolution and interface connectivity are being established. If that happens, run the ping again and investigate persistent failures.

### Verify the switch port

On the switch CLI, run:

```text
show interfaces status
show vlan brief
```

Confirm that the connected switch port is connected and belongs to VLAN 1.

---

## Step 8: Configure the Switch's Default Gateway

Configure the default gateway from global configuration mode:

```text
configure terminal
ip default-gateway 192.168.1.1
end
```

The default gateway allows a Layer 2 switch to send management traffic to destinations outside its local IP subnet.

For example, an administrator on a different subnet may need to connect to the switch using SSH. The switch needs a valid gateway to return traffic to that administrator's network.

**Important:** This configuration does not create a router or make `192.168.1.1` reachable by itself. A router or Layer 3 device must be configured with that address and provide the appropriate routing.

If this standalone lab has no router, local ping and SSH testing can still work without a configured default gateway.

---

## Step 9: Configure a Local Administrator Account

Create a local user for SSH authentication:

```text
configure terminal
username admin privilege 15 secret LabSSH-ChangeMe
```

### Explanation

| Command component | Purpose |
|---|---|
| `username admin` | Creates a local account named `admin`. |
| `privilege 15` | Assigns the highest standard Cisco IOS privilege level. |
| `secret LabSSH-ChangeMe` | Configures the account's password using secret-based storage. |

Privilege level 15 provides extensive administrative access. For production environments, use least privilege and individual administrator accounts wherever possible.

The username and password configured here will be used to authenticate SSH sessions.

---

## Step 10: Configure the Domain Name and Generate RSA Keys

SSH requires cryptographic keys on the switch. On this IOS platform, a hostname and domain name are used when generating RSA keys.

### Configure the domain name

```text
ip domain-name homelab.local
```

The domain name can be a lab-only name; it does not have to be a publicly registered Internet domain.

### Generate RSA keys

```text
crypto key generate rsa
```

When prompted for the modulus size, enter `2048` if the Packet Tracer switch model and IOS image support it.

Some Packet Tracer device images support only a limited set of key sizes. If `2048` is not accepted, use a supported size for the simulation and document that limitation.

RSA keys enable the switch to establish encrypted SSH sessions.

**Security note:** Use modern key sizes and cryptographic settings on real equipment. A Packet Tracer simulation may not support every feature available on production Cisco IOS devices.

---

## Step 11: Enable SSH Version 2 and Restrict Remote Access

### Set the SSH version

```text
ip ssh version 2
```

SSH version 2 is preferred over SSH version 1 because it provides improved security.

### Configure the virtual terminal lines

```text
line vty 0 15
login local
transport input ssh
exit
end
```

### Explanation

| Command | Purpose |
|---|---|
| `line vty 0 15` | Enters configuration mode for the virtual terminal lines used for remote sessions. |
| `login local` | Uses the switch's local username database for authentication. |
| `transport input ssh` | Allows SSH for incoming remote sessions and excludes Telnet on these lines. |
| `exit` | Returns to the previous configuration mode. |
| `end` | Returns to privileged EXEC mode. |

The `login local` command links remote access to the local administrator account created in Step 9.

The `transport input ssh` command helps reduce exposure by disabling Telnet access through the configured VTY lines. Telnet does not protect credentials and session traffic with SSH-style encryption.

### Verify the SSH configuration

Run:

```text
show ip ssh
show ip interface brief
show running-config
```

Check that:

- SSH version 2 is enabled.
- VLAN 1 has the intended management IP address.
- The local username exists.
- The VTY lines use local authentication.
- The VTY lines accept SSH rather than Telnet.

The precise output can vary with the Packet Tracer device model and IOS image.

---

## Step 12: Test SSH Remote Management

Once the switch has an operational management SVI and the PC can reach it, test SSH from the PC.

### Method 1: Use the PC Command Prompt

Open **PC → Desktop → Command Prompt** and enter:

```text
ssh -l admin 192.168.1.5
```

The `-l admin` option specifies the username to use for authentication.

When prompted, enter the password configured for the local `admin` account.

If the connection succeeds, the switch should display its login banner and provide a CLI session.

### Method 2: Use the Packet Tracer SSH Client

1. Open the PC.
2. Select **Desktop**.
3. Open **SSH Client**, if available in your Packet Tracer version.
4. Enter the switch IP address: `192.168.1.5`.
5. Enter the username: `admin`.
6. Connect and provide the configured password.

A successful SSH session confirms that the management IP address is reachable and that SSH authentication is functioning.

### Verify the session

After logging in, run:

```text
enable
show ip interface brief
show running-config
```

These commands confirm that the SSH session provides access to the switch CLI and its configuration.

---

## Step 13: Save the Configuration

After verifying the configuration, save it:

```text
copy running-config startup-config
```

When prompted for the destination filename, press Enter to accept the default.

Alternatively, some IOS images support:

```text
write memory
```

### Why save the configuration?

The running configuration is stored in memory and can be lost when the device restarts. The startup configuration is saved for use when the switch boots.

Saving the configuration ensures that the initial settings, management IP address, passwords, and SSH configuration persist after a restart.

### Verify the saved configuration

```text
show startup-config
```

Confirm that the expected configuration is present.

---

## Step 14: Final Verification Checklist

Use this checklist to confirm that the lab is complete.

- [ ] Connected to the switch through the console cable.
- [ ] Inspected the initial configuration with `show running-config` and `show startup-config`.
- [ ] Configured the hostname `Switch01`.
- [ ] Configured `enable secret`.
- [ ] Disabled unwanted DNS lookups.
- [ ] Configured a Message of the Day banner.
- [ ] Configured console-line password authentication.
- [ ] Assigned `192.168.1.5/24` to VLAN 1.
- [ ] Connected the PC to the switch with Ethernet.
- [ ] Assigned `192.168.1.3/24` to the PC.
- [ ] Verified local connectivity with `ping 192.168.1.5`.
- [ ] Configured the default gateway where appropriate.
- [ ] Created a local SSH administrator account.
- [ ] Configured the domain name and generated RSA keys.
- [ ] Enabled SSH version 2.
- [ ] Configured the VTY lines to use local authentication and allow SSH only.
- [ ] Successfully tested SSH from the PC.
- [ ] Saved the running configuration to startup configuration.

---

## Troubleshooting

### The VLAN 1 interface is down

Run:

```text
show ip interface brief
show vlan brief
show interfaces status
```

Confirm that VLAN 1 is active and that at least one port assigned to VLAN 1 has an active Ethernet link.

Check the PC's Ethernet cable and the switch port status.

### The PC cannot ping the switch

Verify that:

- The PC uses `192.168.1.3` with subnet mask `255.255.255.0`.
- The switch management SVI uses `192.168.1.5` with subnet mask `255.255.255.0`.
- Both devices are connected to the same local network.
- The PC's switch port is active and assigned to VLAN 1.
- VLAN 1 is operational.

Try the ping again after the link becomes active.

### SSH is not working

Run the following commands on the switch:

```text
show ip interface brief
show ip ssh
show running-config
```

Check that:

- VLAN 1 is up and has the correct IP address.
- RSA keys have been generated.
- SSH version 2 is enabled.
- The local username and secret are configured.
- The VTY lines use `login local`.
- The VTY lines allow SSH through `transport input ssh`.

Also confirm that the username and password entered on the PC match the local account.

### The switch cannot reach a device on another subnet

Check the switch's default gateway:

```text
show running-config
```

Confirm that the configured gateway is the IP address of a reachable router or Layer 3 device on the management subnet.

A default gateway alone does not provide routing; the upstream network must also be configured appropriately.

### The switch prompts for a password unexpectedly

The console-line password and the privileged EXEC password serve different purposes:

- The console-line password controls access through the console connection.
- The `enable secret` password controls entry into privileged EXEC mode.
- The local username and secret authenticate SSH access.

Use the credentials associated with the prompt you are seeing.

---

## Key Concepts Learned

### Console access

Console access provides a way to configure and troubleshoot a network device without relying on IP connectivity.

### Management SVI

A switched virtual interface provides an IP endpoint for managing a Layer 2 switch. It is separate from the physical switch ports that forward Ethernet traffic.

### Privileged EXEC mode

Privileged EXEC mode provides access to administrative commands. The `enable secret` setting protects entry into this mode.

### Local authentication

A local username and secret can be used to authenticate remote management sessions without requiring an external authentication server.

### SSH

SSH provides encrypted remote CLI access. Generating cryptographic keys, configuring local authentication, and restricting VTY access to SSH are important parts of enabling secure remote management.

### Default gateway

A Layer 2 switch needs a default gateway to communicate with management destinations outside its own IP subnet. It does not need a gateway to communicate with a PC on the same subnet.

### Running and startup configurations

The running configuration reflects the active settings in memory. The startup configuration stores settings that should persist after a reboot.

---

## Screenshots

Add screenshots from your completed Packet Tracer lab to the `screenshots` folder in this repository.

Suggested screenshots:

| Screenshot | What it should demonstrate |
|---|---|
| `Initial_Layout.png` | PC and Cisco 2960 switch connected through the console cable |
| `Initial_Configuration.png` | Initial `show running-config` or `show version` output |
| `Basic_Switch_Config.png` | Hostname, banner, and basic security settings |
| `Management_SVI.png` | VLAN 1 management IP configuration |
| `Management_Ping.png` | Successful PC ping to `192.168.1.5` |
| `SSH_Configuration.png` | SSH version and VTY configuration |
| `SSH_Success.png` | Successful SSH session to `Switch01` |
| `Saved_Configuration.png` | Startup configuration after saving |

Only add screenshots that reflect the configuration and results you actually observed in Packet Tracer.

---

## Conclusion

In this lab, a Cisco Catalyst 2960 switch was configured through a simulated console connection and prepared for remote management.

The configuration covers basic device identification, password protection, management IP addressing, local authentication, SSH version 2, and configuration persistence.

These skills provide a foundation for more advanced switching and networking tasks, including VLAN management, switch port security, remote administration, and troubleshooting.

**Next steps:** Build on this lab by configuring additional VLANs, assigning access ports, and testing connectivity between devices in different VLANs.
