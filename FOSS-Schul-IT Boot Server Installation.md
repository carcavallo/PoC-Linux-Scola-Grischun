# PXE Boot and Network Setup Guide on Debian 12 in VirtualBox

## Prepositions
1. **Disable DHCP on Your Router**:
   - Your router's DHCP server must be disabled to avoid conflicts, even with correct IP ranges, as it can cause issues.
2. **ISO Used**:
   - Tested with `debian-12.8.0-amd64-netinst.iso`. Other ISOs should also work.

---

## 1. VirtualBox Settings
- **Network**: 
  - Adapter 1: **Bridged Adapter**
  - Promiscuous Mode: **Allow All**
  - Cable Connected: **Checked**

---

## 2. Configure VM (First Boot)
1. Default configuration for Debian: 
   - Use all default and guided settings.
   - Install the bootloader on the same disk.
   - Choose **GNOME Desktop Environment** and **Minimal Tools** for ease of use (can be uninstalled later).
2. Reboot and log in with your configured user.

---

## 3. Install VirtualBox Guest Additions
1. Load guest tools from **Devices → Add Guest Tools**.
2. Install required tools:
   ```bash
   su
   apt update -y && apt upgrade -y
   cd /media/cdrom0 # Adjust to your actual media directory
   apt install build-essential linux-headers-$(uname -r)
   sh ./VBoxLinuxAdditions.run
   sudo reboot now
3. Enable bidirectional clipboard:
   **Go to Devices → Shared Clipboard → Bidirectional**.

---

## 4. Configure User
1. Enter the terminal and switch to the root user:
   ```bash
   su
2. Install sudo
   ```bash
   apt install sudo
3. Update the PATH environment variable:
   ```bash
   export PATH=$PATH:/usr/sbin:/usr/bin
4. Add the user to the sudo group:
   ```bash
   usermod -aG sudo %%%username%%%
5. Switch to the user:
   ```bash
   su %%%username%%%

---

## 5. Setup VM Network
1. Configure a static IP:
   ```bash
   sudo nano /etc/network/interfaces
   ```
   Add the following configuration:
   ```bash
   # The loopback network interface
   auto lo
   iface lo inet loopback

   auto enp0s3
   iface enp0s3 inet static
       address <YOUR_IPV4_ADDRESS>; # Replace <YOUR_IPV4_ADDRESS> with your machine's IPv4 address
       netmask 255.255.255.0
       gateway 192.168.0.1
       dns-nameservers 192.168.0.1
   ```

2. Restart networking and reboot:
   ```bash
   sudo systemctl restart networking
   sudo reboot now
   ```
3. Check the new configuration:
   ```bash
   ip addr
   ```

---

## 6. DHCP Configuration
1. Install DHCP Server (will produce errors):
   ```bash
   sudo apt install isc-dhcp-server
   ```
2. Configure DHCP:
   ```bash
   sudo nano /etc/dhcp/dhcpd.conf
   ```
   Add the following configuration:
   ```bash
   default-lease-time 600;
   max-lease-time 7200;
   ddns-update-style none;
   authoritative;

   subnet 192.168.0.0 netmask 255.255.255.0 {
       range 192.168.0.30 192.168.0.50;
       option routers 192.168.0.1;
       option broadcast-address 192.168.0.255;
       option domain-name-servers 192.168.0.1;
       next-server <YOUR_IPV4_ADDRESS>; # Replace <YOUR_IPV4_ADDRESS> with your machine's IPv4 address

       if option architecture-type = 00:07 {
           filename "debian-installer/amd64/bootnetx64.efi";
       } else {
           filename "pxelinux.0";
       }
   }

   ```
3. Edit interfaces for DHCP:
   ```bash
   sudo nano /etc/default/isc-dhcp-server
   ```
   Update:
   ```bash
   INTERFACESv4="enp0s3"
   INTERFACESv6=""
   ```
4. Check for errors:
   ```bash
   sudo dhcpd -t
   ```
5. Restart DHCP Server:
   ```bash
   sudo systemctl restart isc-dhcp-server
   ```

---

## 7. Install DHCP-Relay-Agent
1. Install using:
   ```bash
   sudo apt install isc-dhcp-relay
   ```
2. Edit the configuration::
   ```bash
   sudo nano /etc/default/isc-dhcp-relay
   ```
   Add the following configuration:
   ```bash
   SERVERS="192.168.1.1"        # Replace <YOUR_DEFAULT_GATEWAY> with your the routers default gateway
   INTERFACES="eth0"            # Your VM-Network interface
   OPTIONS="-option 66 <TFTP-IP>"
   ```
3. Starten des DHCP-Relay-Dienstes
   ```bash
   sudo systemctl restart isc-dhcp-relay
   sudo systemctl enable isc-dhcp-relay
   ```
---

## 8. Check DHCP Connection
1. Verify using:
   ```bash
   sudo dhclient -v enp0s3
   ```
---

## 9. Install TFTP Server
1. Install TFTP:
   ```bash
   sudo apt install tftpd-hpa
2. Configure TFTP:
   ```bash
   sudo nano /etc/default/tftpd-hpa
   ```
   Add:
   ```bash
   TFTP_USERNAME="tftp"
   TFTP_DIRECTORY="/srv/tftp"
   TFTP_ADDRESS="0.0.0.0:69"
   TFTP_OPTIONS="--secure"
   ```
3. Restart TFTP Server:
   ```bash
   sudo systemctl restart tftpd-hpa
   ```
   
---

## 10. Install Linux Image Files
1. Download and extract files:
   ```bash
   cd /srv/tftp
   sudo wget https://deb.debian.org/debian/dists/bookworm/main/installer-amd64/current/images/netboot/netboot.tar.gz
   sudo tar -xvzf netboot.tar.gz
   sudo rm netboot.tar.gz
   ```

---

## 12. Set File Permissions
1. Update permissions:
   ```bash
   sudo chown tftp:tftp /srv/tftp
   sudo chmod 755 /srv/tftp
   sudo chown -R tftp:tftp /srv/tftp/*
   sudo chmod -R 755 /srv/tftp/*
   ```

---

## 13. Install Syslog for Error Handling
1. Install syslog:
   ```bash
   sudo apt install rsyslog -y
   ```
2. Enable and use syslog:
   ```bash
   sudo systemctl enable rsyslog
   sudo tail /var/log/syslog
   ```
   
---


## 14. (Optional) Remove GUI
1. Completely uninstall GUI:
   ```bash
   sudo apt purge gnome* x11* -y
   sudo apt autoremove -y
   ```
2. Reboot into command-line interface.
   
---



