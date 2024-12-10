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
   usermod -aG sudo <YOUR_USERNAME>
5. Switch to the user:
   ```bash
   su <YOUR_USERNAME>

---

## 5. Setup VM Network
1. Configure a static IP:
   ```bash
   sudo nano /etc/network/interfaces
   ```
   Add the following configuration:
   ```bash
   auto lo
   iface lo inet loopback

   auto enp0s3
   iface enp0s3 inet static
       address <DESIRED_IPV4_ADDRESS>
       netmask 255.255.255.0
       gateway 192.168.1.1
       dns-nameservers 8.8.8.8
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

   subnet 192.168.1.0 netmask 255.255.255.0 {
       range 192.168.1.150 192.168.1.200;
       option routers 192.168.1.1;
       option broadcast-address 192.168.1.255;
       option domain-name-servers 192.168.1.1;
       next-server <YOUR_IPV4_ADDRESS>;
       filename "pxelinux.0";
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

## 7. Install TFTP Server
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
   TFTP_ADDRESS=":69"
   TFTP_OPTIONS="--secure"
   ```
3. Restart TFTP Server:
   ```bash
   sudo systemctl restart tftpd-hpa
   ```
   
---

## 8. Install Linux Image Files
1. Download and extract files:
   ```bash
   cd /srv/tftp
   sudo wget https://deb.debian.org/debian/dists/bookworm/main/installer-amd64/current/images/netboot/netboot.tar.gz
   sudo tar -xvzf netboot.tar.gz
   sudo rm netboot.tar.gz
   ```

---

## 9. Set File Permissions
1. Update permissions:
   ```bash
   sudo chown tftp:tftp /srv/tftp
   sudo chmod 755 /srv/tftp
   sudo chown -R tftp:tftp /srv/tftp/*
   sudo chmod -R 755 /srv/tftp/*
   ```

---

### 10. Prepare the Preseed File
1. Create a file named preseed.cfg and include the necessary configurations. Below is an example for installing Debian with a desktop environment:

   Sample preseed.cfg:
   ```bash
   ### Localization
   d-i debian-installer/locale string en_US.UTF-8
   d-i console-setup/ask_detect boolean false
   d-i console-setup/layoutcode string ch
   d-i keyboard-configuration/xkb-keymap select ch
   
   ### Network Configuration
   d-i netcfg/choose_interface select auto
   d-i netcfg/get_hostname string debian
   d-i netcfg/get_domain string local
   
   ### Mirror Configuration
   d-i mirror/country string manual
   d-i mirror/http/hostname string ftp.ch.debian.org
   d-i mirror/http/directory string /debian
   d-i mirror/http/proxy string
   
   ### Disk Partitioning
   d-i partman-auto/method string regular
   d-i partman-auto/disk string /dev/sda
   d-i partman-auto/expert_recipe string \
       boot-root :: \
           1000 1000 1000 ext4 \
               $primary{ } $bootable{ } method{ format } format{ } \
               use_filesystem{ } filesystem{ ext4 } mountpoint{ /boot } \
           . \
           10000 10000 10000 ext4 \
               $primary{ } method{ format } format{ } \
               use_filesystem{ } filesystem{ ext4 } mountpoint{ / } \
           . \
           5000 10000 10000 linux-swap \
               $primary{ } method{ swap } format{ } \
           .
   d-i partman-partitioning/confirm_write_new_label boolean true
   d-i partman/choose_partition select finish
   d-i partman/confirm boolean true
   d-i partman/confirm_nooverwrite boolean true
   
   ### User Accounts
   d-i passwd/root-login boolean true
   d-i passwd/root-password password debian
   d-i passwd/root-password-again password debian
   d-i passwd/make-user boolean true
   d-i passwd/user-fullname string Debian User
   d-i passwd/username string debian
   d-i passwd/user-password password debian
   d-i passwd/user-password-again password debian
   d-i passwd/user-default-groups string sudo
   d-i user-setup/encrypt-home boolean false
   
   ### Task Selection
   tasksel tasksel/first multiselect standard
   d-i pkgsel/include string gnome-core sssd libpam-sss libnss-sss ldap-utils nscd libsss-sudo
   
   ### LDAP
   
   ### Bootloader Installation
   d-i grub-installer/only_debian boolean true
   d-i grub-installer/with_other_os boolean true
   d-i grub-installer/bootdev string /dev/sda
   
   ### Finishing Up
   d-i finish-install/reboot_in_progress note
   d-i debian-installer/exit/poweroff boolean true
   
   ```
2. Configure the PXE Server to Use the Preseed File
   Modify the PXE boot menu file (e.g., pxelinux.cfg/default) to append the preseed file to the kernel arguments.

   Example PXE Boot Menu Entry:
   ```bash
   label Install Debian with Preseed
       menu label ^Install Debian with Preseed
       kernel debian-installer/amd64/linux
       append initrd=debian-installer/amd64/initrd.gz auto=true priority=critical preseed/url=http://<YOUR_IPV4_ADDRESS>/preseed.cfg

   ```
3. Provide the Preseed File via HTTP
   Ensure your preseed file is available to clients during installation:

   Use a web server (e.g., Apache or Nginx) to serve the preseed file.
   Place the preseed.cfg file in the web server's document root (e.g., /var/www/html).

   Test the PXE Boot!
   ---

## 11. (Optional) Install Syslog for Error Handling
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

## 12. (Optional) Remove GUI
1. Completely uninstall GUI:
   ```bash
   sudo apt purge gnome* x11* -y
   sudo apt autoremove -y
   ```
2. Reboot into command-line interface.
   
---



