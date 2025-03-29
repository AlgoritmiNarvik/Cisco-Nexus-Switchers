# Rack Switch Identification and Access Log (Quanta & Cisco)

## Overview
This document summarizes all observed details, actions, and findings related to the investigation of switches available in the lab/server room.
We have identified:
1.  A **Quanta LB6M switch (running FASTPATH OS)** positioned at the top of the rack. Console access to this switch has been established (details below). Specific identification details are in `private-network-details.md`.
2.  Two **Cisco Nexus 5600 Series (N5K-C56128P)** switches labeled "Telin 1" and "Telin 2", located below the Quanta switch. These are pending investigation. Specific identification details (if found) should be stored in `private-network-details.md`.

## Switch Images

### Front View
![Front view of Cisco switches](image-front.png)

### Back View
![Back view of Cisco switches](image-back.png)

## Table of Contents
- [Hardware Details](#hardware-details)
  - [Top Switch (Accessed via Console)](#top-switch-accessed-via-console)
  - [Nexus Switches ("Telin 1" & "Telin 2")](#nexus-switches-telin-1--telin-2)
    - [Technical Specifications (Nexus 5600 Series)](#technical-specifications-nexus-5600-series)
- [Connection Attempts and Findings](#connection-attempts-and-findings)
  - [1. Management Port Connection (Attempt via Ethernet Adapter)](#1-management-port-connection-attempt-via-ethernet-adapter)
  - [2. Console Connection (Successful - Top Quanta LB6M Switch)](#2-console-connection-successful---top-quanta-lb6m-switch)
  - [3. Console Connection Tips for Future Sessions (FASTPATH on Quanta)](#3-console-connection-tips-for-future-sessions-fastpath-on-quanta)
  - [4. Useful Quanta FASTPATH Switch Commands (Top Switch)](#4-useful-quanta-fastpath-switch-commands-top-switch)
- [Glossary of Terms](#glossary-of-terms)
- [Next Steps](#next-steps)
  - [Quanta Switch (Top) Investigation](#quanta-switch-top-investigation)
  - [Cisco Nexus Switches ("Telin 1" / "Telin 2") Investigation](#cisco-nexus-switches-telin-1--telin-2-investigation)
  - [Network Clarification (Management Network)](#network-clarification-management-network)
  - [Server (`basefarm-6`) Preparation](#server-basefarm-6-preparation)
  - [Documentation](#documentation)
- [Additional Notes](#additional-notes)
- [Linux Server Connection (to Quanta Console)](#linux-server-connection-to-quanta-console)
  - [Server Configuration](#server-configuration)
  - [Serial Port Details](#serial-port-details)
  - [Connection Challenges](#connection-challenges)
  - [Technical Findings](#technical-findings)
  - [Security Considerations](#security-considerations)
  - [Troubleshooting Details](#troubleshooting-details)
  - [Alternative Connection Methods](#alternative-connection-methods)
- [Contact](#Contact)

## Hardware Details

### Top Switch (Accessed via Console)
- **Vendor**: Quanta Computer Inc.
- **Model**: LB6M
- **Operating System**: FASTPATH (Version details in `private-network-details.md`)
- **Chipset**: Broadcom BCM56820_B0
- **Serial Number**: `<serial-number>` (See `private-network-details.md`)
- **Base MAC Address**: `<mac-address>` (See `private-network-details.md`)
- **Position**: Mounted at the top of the rack above the two Nexus switches.
- **Current Status**: Active with multiple lit SFP ports observed.
- **Connections**: Mixture of fiber and copper connections via SFP modules observed.
- **Console Port**: RJ45 port used with standard "rollover" cable (9600 8N1).
- **Management Port**: Likely a separate RJ45 Ethernet port exists. An IP address (`<ip-address>/<subnet-mask>`) is configured on interface `0/24`, however this port was observed to be **DOWN**. Default gateway also configured (`<gateway-ip>`). (See `private-network-details.md` for IPs).

### Nexus Switches ("Telin 1" & "Telin 2")
- **Vendor**: Cisco
- **Model**: Nexus 5600 Series (N5K-C56128P)
  - **Quantity**: 2 units
  - **Labels**: "Telin 1" and "Telin 2"
- **Position**: Below the top Quanta switch.
- **Current Status**: Assumed active, pending investigation.

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

---

## Connection Attempts and Findings

### 1. Management Port Connection (Attempt via Ethernet Adapter)
- **Equipment Used**: 
  - Exibel USB-C to Gigabit Ethernet adapter (AX88179A chipset)
  - Standard Ethernet cable
- **Connection Status**:
  - Successfully connected to *a* management network segment.
  - Received DHCP Address: `<dhcp-ip-address>/<subnet-mask>` (Network details in `private-network-details.md`)
  - Connection active at 1000baseT full-duplex
- **Test Results**:
  - Unable to ping potential switch management IPs (target IPs may have been incorrect).
  - No response to SSH connection attempts.
  - ARP table showed no devices on this management subnet.
- **Conclusion**: 
  - Physical connectivity established to a `<network-range>` network.
  - **Relevance Unclear**: It is currently unknown if this network relates to the Quanta switch (which uses a different IP range) or the Cisco Nexus switches. Further investigation needed. (See `private-network-details.md` for network details).

### 2. Console Connection (Successful - Top Quanta LB6M Switch)
- **Equipment Used**:
  - Cisco console cable (light-blue RJ45) connected between server's DB9 serial port (`/dev/ttyS0`) and the Quanta switch's RJ45 console port.
  - Terminal Emulator: `minicom` on Linux server (`<server-hostname>`, see `private-network-details.md`).
- **Connection Status**:
  - Device: `/dev/ttyS0` on the Linux server.
  - Settings: 9600 baud, 8 data bits, no parity, 1 stop bit (8N1).
  - Command: `sudo minicom -D /dev/ttyS0 -b 9600`.
  - Successfully connected, bypassed FASTPATH boot menu, logged in as `<admin-username>` (see `private-network-details.md`), and used `enable` (requires password) to reach privileged mode (`#`).
- **Observed Behavior**:
  - OS identified as FASTPATH (Version details in `private-network-details.md`) on a Quanta LB6M.
  - Standard CLI interaction using `?` for help and `Tab` for completion.
  - **LLDP is currently DISABLED** on all interfaces by default configuration.
  - Ports `0/16`, `0/22`, `0/23` were observed to be **UP** (link active). Other ports 0/1-0/28 were DOWN. Port `0/24` (with configured IP) was DOWN.
  - MAC addresses learned on active ports (Details in `private-network-details.md`). ARP table was empty. No Port Channels active.
- **Challenges**:
  - Initial session required navigating the FASTPATH Startup Menu (selected option 1).
  - Cisco commands are invalid; FASTPATH commands must be used.
- **Conclusion**:
  - Successful console access established to the **top Quanta LB6M switch**.
  - Basic device information, configuration snippets, and current port/protocol status obtained.

### 3. Console Connection Tips for Future Sessions (FASTPATH on Quanta)
For the next console connection attempt to the **Quanta switch**:

- **Proper Screen/Minicom Usage**:
  - Start with `screen /dev/ttyS0 9600` or `sudo minicom -D /dev/ttyS0 -b 9600`.
  - Press Enter 2-3 times to get a prompt (`(FASTPATH Routing) >` or `#`).
  - Use `?` frequently to find commands. Use `Tab` for command completion.
  - To exit screen properly: Press `Ctrl+A`, `Ctrl+\`, then 'y'.
  - To exit minicom: Press `Ctrl+A`, then `X`.

- **Troubleshooting Tips**:
  - If garbage characters appear, verify terminal settings (9600 8N1).
  - If no response, check cable seating and `/dev/ttyS0` permissions (`dialout` group).

- **Logging Console Output**:
  - Use the `script` command before starting screen/minicom:
    ```bash
    script quanta_console_output_$(date +%F).txt
    screen /dev/ttyS0 9600 # or minicom
    ```
    (Type `exit` in the shell after finishing to save the log)
  - Minicom logging: `Ctrl+A`, `L`.

### 4. Useful Quanta FASTPATH Switch Commands (Top Switch)
These commands are relevant for the **Quanta LB6M** switch:

#### Basic Information & Status
- `show version` - Hardware/software versions, MAC, Serial.
- `show sysinfo` - System name, location, uptime.
- `show running-config` - Current active configuration.
- `show lldp interface all` - **Shows Link Status (Up/Down)** and LLDP Tx/Rx status per interface (Note: `show interfaces status` is *not* valid).
- `show interface ethernet <slot/port>` - Shows detailed statistics/counters for a specific physical interface (e.g., `show interface ethernet 0/16`).
- `show interface ethernet switchport` - Shows CPU port statistics.
- `show ip interface brief` / `show ip interface` - IP interface status (May show little if interface is down or unconfigured).
- `show arp` - IP-to-MAC address mappings (ARP cache).
- `show mac-addr-table` - MAC forwarding table (MACs learned per port/VLAN).
- `show vlan` - VLAN summary.
- `show lldp remote-device all` - **Shows LLDP neighbors** (Requires LLDP to be enabled first).
- `show lldp statistics all` - Shows LLDP Tx/Rx counters.
- `show port-channel brief` / `show port-channel all` - Shows status of Link Aggregation groups (Port Channels).
- `show environment` - (**Untested**) Hardware status (temp, fans, power).
- `show logging` / `show eventlog` - (**Untested**) System logs.
- `show clock` - System time.
- `show history` - (**Untested**) Command history.

#### Configuration Commands (Enter `configure` first)
- `lldp run` - Enable LLDP globally.
- `interface <type/number>` (e.g., `interface 0/1`) - Enter interface configuration mode.
  - `lldp transmit` - Enable LLDP sending on this interface.
  - `lldp receive` - Enable LLDP receiving on this interface.
- `hostname <name>` - Set the switch hostname.
- `exit` - Exit current configuration mode.
- `write` - Save running-config to startup-config (run in `#` mode after exiting `configure`).
- `copy <source> <destination>` - File operations.

**(Note:** For the Cisco Nexus switches ("Telin 1", "Telin 2"), standard Cisco NX-OS commands apply.)

## Linux Server Connection (to Quanta Console)

### Server Configuration
- **Hardware Connection**: 
  - Linux server (`<server-hostname>`, see `private-network-details.md`) connected to the **Top Quanta LB6M** switch console port.
  - Using Cisco console cable (light-blue) with RJ45 connector to switch and DB9 connector to server's serial port (`/dev/ttyS0`).

### Serial Port Details
- **Available Serial Ports**:
  - `/dev/ttyS0` and `/dev/ttyS1` identified on the server
  - Both are 16550A UART hardware
  - Default configuration: 115200 baud (though 9600 baud is standard for Cisco console)
  
### Connection Challenges
- **Access Requirements**:
  - Serial port access requires membership in the `dialout` group on `<server-hostname>`.
  - Terminal emulation software required (screen or minicom)
  - Proper serial settings: 9600 baud, 8 data bits, no parity, 1 stop bit, no flow control
  
### Technical Findings
- Confirmed connection between server (`/dev/ttyS0`) and **Quanta switch console port**.
- Serial ports identified and verified in system logs.
- Standard connection procedure for Quanta console (run on `<server-hostname>`):
  ```bash
  # Ensure user is in the dialout group
  # sudo usermod -a -G dialout $USER
  # Log out and back in if group was just added
  screen /dev/ttyS0 9600
  # OR
  sudo minicom -D /dev/ttyS0 -b 9600
  ```

### Security Considerations
- Console access provides privileged access to switch configuration
- Authentication required at login
- Avoid storing credentials in scripts or logs

### Troubleshooting Details
- **Network Connectivity Issues**:
  - Server (`<server-hostname>`) has limited network connectivity now
  - Package installation fails with connection timeouts to archive.ubuntu.com
  - Consider downloading packages on another system and transferring them

- **Permission Resolution**:
  - Add user to dialout group: `sudo usermod -a -G dialout <username>` (on `<server-hostname>`)
  - Log out and log back in for group changes to take effect
  - Check group membership with: `groups <username>`

- **Serial Connection Debugging**:
  - Check system logs: `dmesg | grep -i ttyS`
  - Verify port existence: `ls -l /dev/ttyS*`
  - Set port parameters: `sudo stty -F /dev/ttyS0 9600 cs8 -cstopb -parenb`
  - Test basic connection: `sudo cat /dev/ttyS0` (expect garbage characters)

### Alternative Connection Methods
- **If screen/minicom can't be installed**:
  - Try direct TTY interaction: `sudo -S stty -F /dev/ttyS0 9600 cs8 -cstopb -parenb raw -echo; sudo cat /dev/ttyS0 > /dev/null`
  - Use basic utilities: `echo "show version" | sudo tee /dev/ttyS0 > /dev/null`
  - Script for interactive session:
    ```bash
    #!/bin/bash
    # Save as console.sh and make executable with: chmod +x console.sh
    stty raw -echo
    sudo stty -F /dev/ttyS0 9600 cs8 -cstopb -parenb raw -echo
    sudo cat /dev/ttyS0 & CAT_PID=$!
    while IFS= read -r -n1 char; do
      sudo echo -n "$char" > /dev/ttyS0
    done
    kill $CAT_PID
    stty -raw echo
    ```

---

## Glossary of Terms

*   **ARP (Address Resolution Protocol):** A protocol used to map an IP address (Layer 3) to a physical MAC address (Layer 2) on a local network segment. The `show arp` command displays the switch's table of these mappings.
*   **CLI (Command Line Interface):** A text-based interface used for interacting with the switch's operating system to execute commands and configure the device (e.g., via console, SSH, or Telnet).
*   **DHCP (Dynamic Host Configuration Protocol):** A protocol used to automatically assign IP addresses and other network configuration parameters (like gateway, DNS servers) to devices on a network.
*   **FASTPATH:** A networking software stack/SDK developed by Broadcom, often used as the basis for the operating system on switches from various manufacturers like Quanta, Edge-Core, Dell, etc.
*   **FCoE (Fibre Channel over Ethernet):** A protocol that allows Fibre Channel (storage) traffic to be encapsulated and transported over Ethernet networks. Specific feature of the Cisco Nexus switches.
*   **IP (Internet Protocol):** The main network layer protocol used for addressing devices and routing data across networks.
*   **LACP (Link Aggregation Control Protocol):** A protocol used to dynamically bundle multiple physical network links together into a single logical link (a Port Channel or LAG) for increased bandwidth and redundancy.
*   **LLDP (Link Layer Discovery Protocol):** A vendor-neutral protocol used by network devices to advertise their identity, capabilities, and neighbors on a local Ethernet network. Commands like `show lldp remote-device all` use this protocol to discover directly connected devices, but it needs to be enabled first.
    *   **LLDP-MED (Media Endpoint Discovery):** An extension to LLDP specifically for voice/video devices (like IP phones) to provide additional configuration details (e.g., voice VLAN, QoS settings).
*   **MAC Address (Media Access Control Address):** A unique hardware identifier assigned to a network interface card (NIC). Switches use MAC addresses to forward traffic at Layer 2. The `show mac-addr-table` command displays the switch's learned MAC addresses and associated ports.
*   **NIC (Network Interface Card):** The hardware component that connects a computer or server to a network.
*   **NX-OS (Nexus Operating System):** Cisco's operating system specifically designed for their Nexus line of data center switches (like the Telin 1 / Telin 2 devices).
*   **QSFP+ (Quad Small Form-factor Pluggable Plus):** A type of compact, hot-pluggable transceiver used for high-speed data communications, typically 40 Gigabit Ethernet (can often be broken out into 4x10GbE).
*   **SFP (Small Form-factor Pluggable) / SFP+ (Enhanced SFP):** Compact, hot-pluggable transceivers used for data communications. SFP typically supports 1 Gigabit Ethernet or Fibre Channel, while SFP+ supports 10 Gigabit Ethernet. The Quanta and Nexus switches use these for their fiber or copper ports.
*   **SSH (Secure Shell):** A cryptographic network protocol for operating network services securely over an unsecured network. Commonly used for secure remote CLI access.
*   **Tx/Rx:** Abbreviations for Transmit (sending data) and Receive (receiving data), often seen in interface statistics.
*   **VLAN (Virtual Local Area Network):** A method for logically segmenting a physical network into multiple broadcast domains. Devices within the same VLAN can communicate directly at Layer 2, while communication between VLANs requires a Layer 3 router.
*   **vPC (virtual PortChannel):** A Cisco Nexus feature allowing two separate Nexus switches to appear as a single logical switch to a downstream device connected via a Port Channel. Provides device-level redundancy.

---

## Next Steps

This section outlines the planned actions for further investigation and configuration.

### Quanta Switch (Top) Investigation
1.  **Reconnect:** Establish console connection via `/dev/ttyS0` on `<server-hostname>` (9600 8N1).
2.  **Enable LLDP:** Configure the switch to enable LLDP globally and on relevant interfaces (especially the UP ports `0/16`, `0/22`, `0/23`) to discover neighbors.
    ```bash
    # Example Configuration Commands:
    configure
    lldp run
    interface 0/16
    lldp transmit
    lldp receive
    exit
    interface 0/22
    lldp transmit
    lldp receive
    exit
    interface 0/23
    lldp transmit
    lldp receive
    exit
    # Add other interfaces if necessary
    exit
    write 
    ```
3.  **Identify Neighbors:** After enabling LLDP, wait ~60 seconds and run `show lldp remote-device all` to identify connected devices (e.g., confirm connection to `basefarm-6` and identify devices on other UP ports).
4.  **Gather Interface Details:** Run `show interface ethernet <slot/port>` for the UP interfaces (`0/16`, `0/22`, `0/23`) to check detailed statistics, speed, duplex, and potential errors.
5.  **Check System Status:** Execute `show logging`, `show eventlog`, and `show environment` to check for system events, errors, or hardware issues.
6.  **Set Hostname:** If not already done, configure a descriptive hostname (e.g., `Quanta-Top-SW1`).
    ```bash
    configure
    hostname Quanta-Top-SW1 
    exit
    write
    ```
7.  **Verify IP Connectivity:** Investigate the configured IP on interface `0/24`. If the interface can be brought UP, test if the switch is reachable via this IP. Check the role of the `158.39.93.49/50` network.
8.  **Check Remote Access:** Examine the running configuration (`show running-config`) for `ssh` or `telnet` settings (`show network` might also be relevant).

### Cisco Nexus Switches ("Telin 1" / "Telin 2") Investigation
1.  **Identify Ports:** Locate the Console and `mgmt0` ports on both Nexus switches.
2.  **Attempt Console Access:** Move the console cable from the Quanta switch to one of the Nexus switches and attempt connection via `/dev/ttyS0` (9600 8N1). Use standard NX-OS commands (e.g., `show version`, `show running-config`).
3.  **Attempt Management Access:** Connect a laptop or the server to the `mgmt0` port of a Nexus switch. If the management network (`<network-range>`, see `private-network-details.md`) is known or suspected, configure a static IP on the laptop/server in that subnet and attempt SSH/HTTPS access to the Nexus default IPs or IPs found via scanning.

### Network Clarification (Management Network)
1.  **Investigate `<network-range>`:** Connect a device to the network segment where the DHCP address `<dhcp-ip-address>` was received.
2.  **Scan for Devices:** If possible, use tools like `arp-scan` or `nmap` from the connected device to identify active hosts on this network, potentially revealing the Nexus management IPs.

### Server (`basefarm-6`) Preparation
1.  **Verify Console Access Tools:** Ensure `screen` or `minicom` is installed and functional on `<server-hostname>`.
2.  **Confirm Permissions:** Double-check that the user account used for console access (`<username>`) is part of the `dialout` group.

### Documentation
1.  **Update `private-network-details.md`:** Record all new findings, including LLDP neighbor details, confirmed interface connections, management IPs discovered, and any credentials obtained.
2.  **Update `README.md`:** Refine command lists, observations, and next steps based on progress.
3.  **Save Configurations:** After making changes (like enabling LLDP or setting hostname), ensure configurations are saved on the switches (`write` on Quanta, `copy running-config startup-config` on Nexus).

## Additional Notes
- These are high-performance data center switches commonly used for top-of-rack deployment.
- Support various Layer 2/3 protocols and virtualization features.
- Can be used in a Virtual Port Channel (vPC) setup for redundancy.
- The third switch at the top of the rack appears to be in active production use with multiple connected ports.

---

## Contact
Documented by: Almaz @ UiT Narvik, 2025
