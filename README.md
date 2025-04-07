# Algoritmi AI-SafeSpace: Rack Switch & Server Configuration Log

**Project:** Algoritmi-AI-SafeSpace
**Goal:** Establish a compute resource ("compute som er åpent for studentene") using legacy server hardware (HP ProLiant DL360 G5/G6) and newly acquired components (including a custom Epyc/GPU pc) for student use. This involves configuring network switches and servers for internal and external connectivity.

> **DISCLAIMER:** This documentation is shared for academic and educational purposes. All sensitive information (exact IP addresses, serial numbers, etc.) is hidden or written as placeholders. The information contained here represents standard network configuration practices that don't pose any security risks. This project is intended as a learning resource for students and network enthusiasts at UiT Narvik.

TL:DR; This document summarizes the configuration journey, focusing on the network infrastructure. Specific sensitive details like exact IPs, MAC addresses, serial numbers, and credentials are stored in a separate, private file, contact for more detailes if needed.

## Switch Images

### Front View
![Front view of Cisco switches](image-front.png)

### Back View
![Back view of Cisco switches](image-back.png)

---

## Table of Contents

- [Algoritmi AI-SafeSpace: Rack Switch \& Server Configuration Log](#algoritmi-ai-safespace-rack-switch--server-configuration-log)
  - [Switch Images](#switch-images)
    - [Front View](#front-view)
    - [Back View](#back-view)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Hardware Inventory](#hardware-inventory)
    - [Quanta LB6M Switch (Primary Switch)](#quanta-lb6m-switch-primary-switch)
    - [Nexus Switches ("Telin 1" \& "Telin 2")](#nexus-switches-telin-1--telin-2)
      - [Technical Specifications (Nexus 5600 Series):](#technical-specifications-nexus-5600-series)
    - [Management Switch ("Switch 1")](#management-switch-switch-1)
    - [Servers](#servers)
  - [Network Topology (Conceptual)](#network-topology-conceptual)
  - [Quanta LB6M Configuration Summary](#quanta-lb6m-configuration-summary)
    - [Basic Setup](#basic-setup)
    - [Link Aggregation (Static LAGs)](#link-aggregation-static-lags)
    - [Access Ports](#access-ports)
    - [Layer 3 Configuration](#layer-3-configuration)
    - [Uplink Configuration](#uplink-configuration)
  - [Server Configuration Notes](#server-configuration-notes)
    - [Internal Network](#internal-network)
    - [Bonding](#bonding)
    - [Gateway](#gateway)
  - [Switch 1 (Management) Configuration Summary](#switch-1-management-configuration-summary)
  - [Current Status \& Issues](#current-status--issues)
  - [Console Access Guide (Quanta LB6M)](#console-access-guide-quanta-lb6m)
    - [Connection Details](#connection-details)
    - [Useful Quanta FASTPATH Commands](#useful-quanta-fastpath-commands)
  - [Glossary of Terms](#glossary-of-terms)
  - [Next Steps](#next-steps)
  - [Contact](#contact)

---

## Overview

This repository documents the process of identifying, accessing, and configuring network switches and servers for the Algoritmi AI-SafeSpace project. The primary focus has been on configuring a Quanta LB6M switch to connect several HP ProLiant servers ("Basefarm" 2-6) and a new custom-built server ("Algoritmi-1"). A secondary Ethernet switch ("Switch 1") is used for management port access.

The configuration involved console access, setting basic security, enabling discovery protocols (LLDP), and configuring link aggregation (Static LAG) due to limitations found in the switch's LACP command-line interface (pending investigation though). Layer 3 routing has been enabled, and an IP address configured for the internal server network.

---

## Hardware Inventory

### Quanta LB6M Switch (Primary Switch)
- **Vendor**: Quanta Computer Inc.
- **Model**: LB6M
- **OS**: FASTPATH `<version>` based on Linux `<kernel-version>`
- **Role**: Primary Layer 2/3 switch connecting servers and providing potential internet gateway.
- **Management IP**: `<internal-gateway-ip>` / `<subnet-mask>` associated with VLAN 1 (configured via `network parms`).
- **Uplink IP**: `<uit-assigned-ip>` / `<subnet-mask>` configured on physical port `0/25`.
- **Default Gateway (for Switch)**: `<uit-gateway-ip>`
- **Base MAC Address**: `<quanta-mac-address>`
- **Console Access**: Via RJ45 console port (9600 8N1) using a rollover light-blue cable connected to `<server-hostname>` (`/dev/ttyS0`).
- **Status**: Configured for internal L2/L3 operation with Static LAGs. **UiT Uplink port `0/25` is currently DOWN.**.

### Nexus Switches ("Telin 1" & "Telin 2")
- **Vendor**: Cisco
- **Model**: Nexus 5600 Series (N5K-C56128P)
  - **Quantity**: 2 units
  - **Labels**: "Telin 1" and "Telin 2"
- **Position**: Below the top Quanta switch.
- **Current Status**: active, pending investigation.

#### Technical Specifications (Nexus 5600 Series):
- **Form Factor**: 1RU (1.75 inches) rack mount
- **Switching Capacity**: Up to 1.28 Tbps
- **Forwarding Rate**: Up to 947 Mpps
- **Ports**:
  - 48 fixed 10-Gigabit SFP+ ports
  - 4 fixed 40-Gigabit QSFP+ ports (or 16 10-Gigabit ports through breakout cables)
  - Management ports: 1 RJ-45 port, 1 RS-232 console port, 1 USB port
- **Key Features**: Layer 2/3, vPC, FCoE, QoS, NX-OS operating system.
- **Console Port**: Standard Cisco RJ45 console port (usually 9600 8N1).
- **Management Port**: Dedicated RJ45 Ethernet port (`mgmt0`).

### Management Switch ("Switch 1")
- **Vendor/Model**: TBC (Black Ethernet switch, patch-panel style)
- **Role**: Provides Layer 2 access for server management ports (iLO/DRAC/BMC).
- **Connections**:
    - Downlinks: Ethernet ports `1` (to BF1-OFF), `2` (BF2), `3` (BF3), `4` (BF4), `5` (BF5), `6` (BF6), `11` (Algoritmi-1) connect to server management NICs.
    - Uplinks: SFP ports `Uplink 1` & `Uplink 2` connect via SFP cables to Quanta ports `0/22` & `0/23` (configured as Static LAG `1/1`).
- **Status**: Assumed operational; configuration unknown.

### Servers
- **Basefarm 1**: HP ProLiant DL360 G5. **Status: Decommissioned** (Hardware issues). Connected to Switch 1 Port 1 (Mgmt only).
- **Basefarm 2-4**: HP ProLiant DL360 G5. **Status: Active**. Running Proxmox VE. Internal IP: `10.0.0.x`. Connected to Quanta via SFP (BF2: single link; BF3/4: dual links for LAG) and Switch 1 (Mgmt).
- **Basefarm 5-6**: HP ProLiant DL360 G6. **Status: Active**. Running Proxmox VE. Internal IP: `10.0.0.x`. Connected to Quanta via dual SFP links (for LAG) and Switch 1 (Mgmt).
- **Algoritmi-1**: Custom Build (AMD Epyc 7C13, RTX 3090). **Status: Active**. OS TBD. Internal IP: TBD (`10.0.0.x`?). Connected to Quanta port `0/26` (Ethernet, Link UP, status uncertain) and Switch 1 Port 11 (Mgmt).

---

## Network Topology (Conceptual)

```
+-----------------+        +-----------------------+        +-----------------+
|   UiT Network   |<----X--|  Quanta LB6M Switch   |--------| Switch 1 (Mgmt) |
| (External Link) |  (DOWN)|                       | (LAG   | (Ports 1-6, 11) |
+-----------------+ (0/25) |  Mgmt IP: 10.0.0.x?   |  1/1)  +-------+---------+
                           |  Uplink IP: x.x.x.x   | (Static)       | Ethernet
                           +--+-+-+--------+-+-+---+                |
                              | | |        | | |                    |
      +------------+ (StaticLAG)| |        | | |(Static LAG)  +------------+ ... etc
      | Basefarm 6 | ----(1/12)-+ |        | +-----(1/6)------| Basefarm 3 |
      | (10.0.0.x?)|  (0/12,0/13) |        |   (0/6,0/7)      | (10.0.0.x?)|
      +-----+------+              |        |                  +-----+------+
            | (Mgmt)              |        |                        | (Mgmt)
            +---------------------+--------+------------------------+
                                  |        |
      +------------+ (Static LAG) |        |(Static LAG) +------------+ ... etc
      | Basefarm 4 |-----(1/8)--+        +----(1/10)-----| Basefarm 5 |
      | (10.0.0.x?)|  (0/8,0/9)            (0/10,0/11)   | (10.0.0.x?)|
      +-----+------+                                     +-----+------+
            | (Mgmt)                                           | (Mgmt)
            +--------------------------------------------------+
                              |        +------------+
                              +--------| Basefarm 2 | (0/2, Single Link)
                              |        | (10.0.0.x?)|
      +-------------+         |        +-----+------+
      | Algoritmi-1 |---------+              | (Mgmt)
      | (10.0.0.x?) | (0/26, Ethernet)       |
      +-----+-------+                        |
            | (Mgmt)                         |
            +--------------------------------+
```

---

## Quanta LB6M Configuration Summary

Configuration applied via console access as of April 2025.

### Basic Setup
- **Admin User**: Password secured.
- **LLDP**: Enabled (transmit/receive) on most active ports (`0/2`, `0/6`-`0/13`, `0/22`, `0/23`, `0/26`) and the uplink port (`0/25`).
- **Unused Ports**: `0/1`, `0/24` are `shutdown`.

### Link Aggregation (Static LAGs)
Static LAGs were configured (Dynamic LACP pending investigation):
- **LAG 1/1 (to Switch 1)**: Members `0/22`, `0/23`. Type: Static. Status: UP. Configured for VLAN 1 access (implicit).
- **LAG 1/6 (to Basefarm 3)**: Members `0/6`, `0/7`. Type: Static (`port-channel static`). Status: UP. Configured for VLAN 1 access (`vlan pvid 1`, `vlan participation include 1`).
- **LAG 1/8 (to Basefarm 4)**: Members `0/8`, `0/9`. Type: Static (`port-channel static`). Status: UP. Configured for VLAN 1 access.
- **LAG 1/10 (to Basefarm 5)**: Members `0/10`, `0/11`. Type: Static (`port-channel static`). Status: UP. Configured for VLAN 1 access.
- **LAG 1/12 (to Basefarm 6)**: Members `0/12`, `0/13`. Type: Static (`port-channel static`). Status: UP. Configured for VLAN 1 access.

### Access Ports
- **Port 0/2 (to Basefarm 2)**: Configured for VLAN 1 access (`vlan pvid 1`, `vlan participation include 1`). Status: UP.
- **Port 0/26 (to Algoritmi-1)**: Configured for VLAN 1 access (`vlan pvid 1`, `vlan participation include 1`). Status: UP.

### Layer 3 Configuration
- **IP Routing**: Enabled globally (`ip routing`).
- **Management/Gateway IP**: `10.0.0.x` configured via `network parms` and associated with `Management VLAN ID 1`. This allows the switch to act as the gateway for the `10.0.0.x` network.
- **Switch Default Gateway**: Set to `<uit-gateway-ip>` via `ip default-gateway`. (Only active when uplink is UP).

### Uplink Configuration
- **Port 0/25**: Configured with static IP `<uit-assigned-ip>` / `<subnet-mask>`. Set to `no shutdown`. **Status: DOWN**.

---

## Server Configuration Notes

### Internal Network
- Servers operate on a private `10.0.0.x` network (subnet mask `255.255.255.0`).
- IPs observed: BF2 (`10.0.0.x?`), BF3 (`10.0.0.x?`), BF4 (`10.0.0.x?`), BF5 (`10.0.0.x?`), BF6 (`10.0.0.x?`). Algoritmi-1 IP TBD.

### Bonding
- **Requirement:** Servers connected to Static LAGs on the Quanta switch (Basefarm 3, 4, 5, 6) **must** be configured for a compatible **Static Bonding mode** in their OS (e.g., `/etc/network/interfaces`).
- **Recommended Mode:** `balance-xor` (Mode 2 in Linux bonding). Using `802.3ad` (LACP) will cause a mismatch (or choos dynamic LACP).
- **Basefarm 2:** Uses a single link connection to Quanta port `0/2`. No bonding needed/configured for this link.

### Gateway
- Servers should be configured to use the Quanta switch's management IP (`<internal-gateway-ip>`, e.g., `10.0.0.1`) as their default gateway.

---

## Switch 1 (Management) Configuration Summary

- **Role**: Provides Layer 2 connectivity for server management ports.
- **Connection to Quanta**: Uses Static LAG `1/1` (Ports `0/22`, `0/23`) configured for access in VLAN 1.
- **Internal Config**: Unknown (Make/Model TBC). Assumed to be performing basic Layer 2 switching.

---

## Current Status & Issues

- **Working:**
    - Internal Layer 2 connectivity between servers, Quanta, and Switch 1 via VLAN 1.
    - Static LAGs between Quanta and servers (BF3-6) and Switch 1 are UP and passing traffic.
    - Quanta switch is configured with the intended gateway IP (`10.0.0.1`) for the server network.
    - `ip routing` is enabled on the Quanta switch.
    - **Server Bonding Modes:** Requires verification/configuration on Basefarm 3, 4, 5, 6 to use `balance-xor` (or another compatible static mode) to match the switch's Static LAGs.
    - **Algoritmi-1 Link (0/26):** Link is UP, but no MAC/LLDP learned by Quanta. Needs investigation on Algoritmi-1's network config and activity.
    - **Switch 1 LLDP:** No LLDP neighbor seen by Quanta from Switch 1. Check if LLDP is enabled on Switch 1.
    - **Switch 1 Identity:** Make/Model/MAC still TBC.

---

## Console Access Guide (Quanta LB6M)

### Connection Details
- **Cable**: Standard Rollover Console Cable (RJ45 to DB9).
- **Switch Port**: RJ45 Console Port.
- **Server Port**: DB9 Serial Port (e.g., `/dev/ttyS0` on `<server-hostname>`).
- **Settings**: 9600 baud, 8 data bits, No parity, 1 stop bit (9600 8N1).
- **Software (Linux)**: `minicom` or `screen`.
  ```bash
  # Example using minicom (requires user in 'dialout' group or run with sudo)
  sudo minicom -D /dev/ttyS0 -b 9600

  # Example using screen (requires user in 'dialout' group)
  screen /dev/ttyS0 9600
  ```
- **Login**: Use configured admin username and password. Enter `enable` and provide password for privileged mode (`#`).

### Useful Quanta FASTPATH Commands
*(Run at `#` prompt unless noted)*
- `show version`: Hardware/software details.
- `show sysinfo`: Hostname, uptime.
- `show network`: Management IP configuration status.
- `show running-config`: Current active configuration.
- `show lldp interface all`: Link status and LLDP port status.
- `show lldp remote-device all`: Discovered LLDP neighbors.
- `show mac-addr-table`: Learned MAC addresses per port/LAG.
- `show vlan`: VLAN database summary.
- `show port-channel brief`: Summary status of all LAGs.
- `show port-channel <id>`: Detailed status of a specific LAG (e.g., `show port-channel 1/6`).
- `show ip interface brief`: Status of configured IP interfaces (e.g., loopback).
- `show arp`: ARP table (IP-to-MAC mappings known by switch).
- `show environment`: Hardware status (temp, fans, power).
- `show logging` / `show eventlog`: System logs.
- `write memory`: **Save configuration changes.**
- `configure`: Enter global configuration mode.
- `exit`: Exit current configuration mode.
- `?`: Display available commands in current context.

---

## Glossary of Terms

*   **ARP (Address Resolution Protocol):** A protocol used to map an IP address (Layer 3) to a physical MAC address (Layer 2) on a local network segment. The `show arp` command displays the switch's table of these mappings.
*   **CLI (Command Line Interface):** A text-based interface used for interacting with the switch's operating system to execute commands and configure the device (e.g., via console, SSH, or Telnet).
*   **DHCP (Dynamic Host Configuration Protocol):** A protocol used to automatically assign IP addresses and other network configuration parameters (like gateway, DNS servers) to devices on a network.
*   **FASTPATH:** A networking software stack/SDK developed by Broadcom, often used as the basis for the operating system on switches from various manufacturers like Quanta, Edge-Core, Dell, etc.
*   **FCoE (Fibre Channel over Ethernet):** A protocol that allows Fibre Channel (storage) traffic to be encapsulated and transported over Ethernet networks. Specific feature of the Cisco Nexus switches.
*   **IP (Internet Protocol):** The main network layer protocol used for addressing devices and routing data across networks.
*   **LACP (Link Aggregation Control Protocol):** A protocol used to dynamically bundle multiple physical network links together into a single logical link (a Port Channel or LAG) for increased bandwidth and redundancy.
*   **LAG (Link Aggregation Group):** Multiple physical ports combined to act as a single logical port, for increased bandwidth and redundancy.
*   **LLDP (Link Layer Discovery Protocol):** A vendor-neutral protocol used by network devices to advertise their identity, capabilities, and neighbors on a local Ethernet network. Commands like `show lldp remote-device all` use this protocol to discover directly connected devices, but it needs to be enabled first.
    *   **LLDP-MED (Media Endpoint Discovery):** An extension to LLDP specifically for voice/video devices (like IP phones) to provide additional configuration details (e.g., voice VLAN, QoS settings).
*   **MAC Address (Media Access Control Address):** A unique hardware identifier assigned to a network interface card (NIC). Switches use MAC addresses to forward traffic at Layer 2. The `show mac-addr-table` command displays the switch's learned MAC addresses and associated ports.
*   **NIC (Network Interface Card):** The hardware component that connects a computer or server to a network.
*   **NX-OS (Nexus Operating System):** Cisco's operating system specifically designed for their Nexus line of data center switches (like the Telin 1 / Telin 2 devices).
*   **QSFP+ (Quad Small Form-factor Pluggable Plus):** A type of compact, hot-pluggable transceiver used for high-speed data communications, typically 40 Gigabit Ethernet (can often be broken out into 4x10GbE).
*   **SFP (Small Form-factor Pluggable) / SFP+ (Enhanced SFP):** Compact, hot-pluggable transceivers used for data communications. SFP typically supports 1 Gigabit Ethernet or Fibre Channel, while SFP+ supports 10 Gigabit Ethernet. The Quanta and Nexus switches use these for their fiber or copper ports.
*   **Static LAG:** Link Aggregation without a dynamic negotiation protocol like LACP. Requires manual configuration consistency on both ends.
*   **SVI (Switched Virtual Interface):** A logical Layer 3 interface on a switch associated with a specific VLAN, allowing the switch to route traffic for that VLAN.
*   **SSH (Secure Shell):** A cryptographic network protocol for operating network services securely over an unsecured network. Commonly used for secure remote CLI access.
*   **Tx/Rx:** Abbreviations for Transmit (sending data) and Receive (receiving data), often seen in interface statistics.
*   **VLAN (Virtual Local Area Network):** A method for logically segmenting a physical network into multiple broadcast domains. Devices within the same VLAN can communicate directly at Layer 2, while communication between VLANs requires a Layer 3 router.
*   **vPC (virtual PortChannel):** A Cisco Nexus feature allowing two separate Nexus switches to appear as a single logical switch to a downstream device connected via a Port Channel. Provides device-level redundancy.

---

## Next Steps

1.  **Verify/Configure Server Bonding (or choose dynamic LACP):** Log into Basefarm 3, 4, 5, and 6. Check `/etc/network/interfaces`. Ensure `bond-mode` is set to `balance-xor` (or another compatible static mode). Restart networking if changes are made.
2.  **Investigate Quanta PSU 1:** Check physical connections and status lights for Power Supply 1. Determine why it's reported as "Not powered".
3.  **Investigate Anomalies:**
    *   Check network configuration and activity on Algoritmi-1 to understand why no MAC/LLDP is seen on Quanta `0/26`.
    *   Check Switch 1 configuration to see if LLDP can be enabled.
4.  **Identify Switch 1:** Attempt to identify the Make/Model of "Switch 1" for better documentation and potential configuration access.
5.  **Documentation:** Keep this README and `private-network-details.md` updated with findings.

---

## Contact
Documented by: Almaz @ UiT Narvik, Algoritmi Student Society, April 2025
