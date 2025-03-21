# Cisco Switch Identification and Access Log

## Overview
This document summarizes all observed details, actions, and findings related to the investigation of Cisco switches available in the lab/server room. We have identified two Cisco N5K-C56128P switches labeled "Telin 1" and "Telin 2", as well as a third Cisco switch positioned at the top of the rack.

## Switch Images

### Front View
![Front view of Cisco switches](image-front.png)

### Back View
![Back view of Cisco switches](image-back.png)

## Hardware Details

### Switch Identification:
- **Nexus Switches**:
  - **Model**: Cisco Nexus 5600 Series (N5K-C56128P)
  - **Quantity**: 2 units
  - **Labels**: "Telin 1" and "Telin 2"

- **Additional Top Switch**:
  - **Vendor**: Cisco
  - **Possible Models**: Could be a Nexus 9000 series or possibly a Catalyst model (exact identification pending)
  - **Current Status**: Active with multiple lit SFP ports
  - **Connections**: Mixture of fiber and copper connections via SFP modules
  - **Position**: Mounted at the top of the rack above the two Nexus switches

### Technical Specifications (Nexus 5600 Series):
- **Form Factor**: 1RU (1.75 inches) rack mount
- **Switching Capacity**: Up to 1.28 Tbps
- **Forwarding Rate**: Up to 947 Mpps
- **Ports**:
  - 48 fixed 10-Gigabit SFP+ ports
  - 4 fixed 40-Gigabit QSFP+ ports (or 16 10-Gigabit ports through breakout cables)
  - Management ports: 1 RJ-45 port, 1 RS-232 console port, 1 USB port

### Key Features:
- Layer 2 and 3 switching capabilities
- Low-latency cut-through architecture
- Virtual Port Channel (vPC) technology
- Fibre Channel over Ethernet (FCoE) support
- Advanced quality of service (QoS)
- Comprehensive security features

### Physical Features:
- **Console Port**:
  -  **Light-blue cable was used through RJ45 port** (Cisco standard)
  - Used for low-level CLI access for configuration and diagnostics
- **Management Port**: RJ-45 Ethernet port

---

## Connection Attempts and Findings

### 1. Management Port Connection
- **Equipment Used**: 
  - Exibel USB-C to Gigabit Ethernet adapter (AX88179A chipset)
  - Standard Ethernet cable
- **Connection Status**:
  - Successfully connected to management network
  - Received IP address: <internal-mgmt-ip>
  - Connection active at 1000baseT full-duplex
- **Test Results**:
  - Unable to ping potential switch management IPs (potential gateway and device addresses)
  - No response to SSH connection attempts
  - ARP table showed no devices on the management subnet
- **Conclusion**: 
  - Physical connectivity established
  - Management network detected but no successful device communication

### 2. Console Connection 
- **Equipment Used**:
  - Cisco console cable (light-blue RJ45)
  - GenesysLogic USB3.1 Hub adapter
- **Connection Status**:
  - Device detected as /dev/tty.usbserial-2110
  - Connection established using `screen /dev/tty.usbserial-2110 9600`
- **Observed Behavior**:
  - Terminal displayed random symbols when moving trackpad/mouse
  - Characters appeared in response to terminal interaction
  - Standard terminal behaviors (history, arrow keys) were non-functional
  - Screen session did not save terminal history
- **Challenges**:
  - Difficulty interpreting console output
  - Terminal emulation issues (screen showing "Sorry, could not find a PTY" in some attempts)
    - **Cause**: Previous screen sessions not properly terminated, creating resource contention
    - **Solution**: Kill existing screen processes with `sudo lsof | grep tty.usbserial` and `sudo kill <PID>`
  - Resource busy errors when attempting multiple connections
- **Conclusion**:
  - Console port physical connection succeeded
  - Basic communication with switch was established
  - Further practice with console interaction needed for effective management

### 3. Console Connection Tips for Future Sessions
For the next console connection attempt:

- **Proper Screen Usage**:
  - After starting `screen /dev/tty.usbserial-2110 9600`:
    - Press Enter 2-3 times to get a clean prompt
    - Type commands carefully (no arrow keys or history)
    - If you see a prompt like `Switch>` or `hostname#`, you're successfully connected
  - To exit screen properly: Press Ctrl+A followed by Ctrl+\ (then 'y' to confirm)

- **Troubleshooting Tips**:
  - If garbage characters appear, try "resetting" the terminal with `Ctrl+A` then `k`
  - If no response, ensure console cable is fully seated
  - Try a different baud rate if standard 9600 doesn't work (115200 is common alternative)

- **Logging Console Output**:
  - Use the `script` command before starting screen:
    ```
    script console_output.txt
    screen /dev/tty.usbserial-2110 9600
    ```
    (After disconnecting, type `exit` to save the log)
  - Consider using a dedicated serial terminal application like Serial (from Mac App Store)

### 4. Useful Cisco Switch Commands
For future reference, these commands will be valuable once console access is established:

#### Basic Information
- `show version` - Display switch model, OS version, uptime
- `show running-config` - View current configuration
- `show interfaces status` - View all interface statuses
- `show ip interface brief` - Show IP addresses on interfaces
- `show cdp neighbors` - Display connected Cisco devices

#### Configuration Commands
- `configure terminal` - Enter configuration mode
- `interface <type/number>` - Configure a specific interface
- `copy running-config startup-config` - Save configuration

---

## Next Steps

1. **Console Access Troubleshooting**:
   - Try different terminal emulation software (Serial, CoolTerm, ZTerm)
   - Test different baud rates (115200, 57600, 38400)
   - Check console cable and adapter functionality on another device
   
2. **Alternative Management Access**:
   - Try connection via web interface (https://<mgmt-ip>)
   - Attempt telnet connection if SSH is disabled
   - Test different management IP subnets (alternative private networks)

3. **Physical Reset Option**:
   - If necessary, consider password recovery procedure
   - Document factory reset process as last resort

---

## Additional Notes
- These are high-performance data center switches commonly used for top-of-rack deployment.
- Support various Layer 2/3 protocols and virtualization features.
- Can be used in a Virtual Port Channel (vPC) setup for redundancy.
- The third switch at the top of the rack appears to be in active production use with multiple connected ports.

---

## Linux Server Connection

### Server Configuration
- **Hardware Connection**: 
  - Linux server in the server room connected to Cisco switch console port
  - Using Cisco console cable (light-blue) with RJ45 connector to switch and DB9 connector to server
  - The cable is sometimes referred to as a "rollover cable" due to its pin configuration

### Serial Port Details
- **Available Serial Ports**:
  - `/dev/ttyS0` and `/dev/ttyS1` identified on the server
  - Both are 16550A UART hardware
  - Default configuration: 115200 baud (though 9600 baud is standard for Cisco console)
  
### Connection Challenges
- **Access Requirements**:
  - Serial port access requires membership in the `dialout` group
  - Terminal emulation software required (screen or minicom)
  - Proper serial settings: 9600 baud, 8 data bits, no parity, 1 stop bit, no flow control
  
### Technical Findings
- Confirmed connection between server and switch console port
- Serial ports identified and verified in system logs
- Standard connection procedure should use:
  ```
  screen /dev/ttyS0 9600
  ```
  (After user is added to dialout group and screen is installed)

### Security Considerations
- Console access provides privileged access to switch configuration
- Authentication required at login
- Avoid storing credentials in scripts or logs

### Troubleshooting Details
- **Network Connectivity Issues**:
  - Server has limited network connectivity now
  - Package installation fails with connection timeouts to archive.ubuntu.com
  - Consider downloading packages on another system and transferring them

- **Permission Resolution**:
  - Add user to dialout group: `sudo usermod -a -G dialout <username>`
  - Log out and log back in for group changes to take effect
  - Check group membership with: `groups <username>`

- **Serial Connection Debugging**:
  - Check system logs: `dmesg | grep -i ttyS`
  - Verify port existence: `ls -l /dev/ttyS*`
  - Set port parameters: `sudo stty -F /dev/ttyS0 9600 cs8 -cstopb -parenb`
  - Test basic connection: `sudo cat /dev/ttyS0` (expect garbage characters)

### Alternative Connection Methods
- **If screen/minicom can't be installed**:
  - Try direct TTY interaction: `sudo -S stty -F /dev/ttyS0 9600 cs8 -cstopb -parenb raw -echo; sudo cat /dev/ttyS0`
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

### Next Steps
1. **Configuration Priorities**:
   - Add <username> user to dialout group
   - Install screen package (when network is available)
   - Create a simple script for console access if package installation fails
   
2. **Initial Switch Configuration**:
   - Document hostname and IP configuration
   - Map out physical port connections
   - Check CDP neighbors to identify connected devices
   
3. **Documentation Tasks**:
   - Record successful connection method
   - Document switch passwords (securely)
   - Save running-config to startup-config after changes

## Credits
Documented by: Almaz @ UiT Narvik, 2025
