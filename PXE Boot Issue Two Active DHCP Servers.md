# PXE Boot Issue: Two Active DHCP Servers

The issue arises because there are **two active DHCP servers** on your network: one on your **router** and another on your **Debian system**. This creates conflicts when a PXE client tries to request an IP address. Here's a breakdown of the problem:

---

## Problem Details

### 1. Router's DHCP Settings:
- Your router's DHCP is configured with the IP range `192.168.1.20` to `192.168.1.254`.
- It operates on a subnet of `192.168.1.0/24` (subnet mask `255.255.255.0`), which means the router assigns IPs only within this range.

### 2. Debian DHCP Server:
- Your Debian DHCP server also responds to client DHCP requests. From the logs, it's clear that your Debian server is offering IPs like `192.168.0.30`, which is outside the `192.168.1.0/24` range of your router.
- The PXE client ends up receiving conflicting offers:
  - The router DHCP server operates on `192.168.1.0/24`.
  - The Debian DHCP server operates on `192.168.0.0/16`.
- When the client requests an IP like `192.168.1.21`, the Debian server detects it as being on the "wrong network" because the Debian server is configured for a broader subnet (`192.168.0.0/16`).

### 3. Conflict:
- The PXE client alternates between offers from the two DHCP servers, causing failures like `wrong network` and `DHCPNAK`.

---

## Solution Options

### 1. Disable Router's DHCP Server (Preferred for PXE Booting):
If your PXE clients must receive IPs from your Debian DHCP server, you need to disable the DHCP server on your router.

#### Steps:
1. Log in to your router's admin interface.
2. Navigate to the DHCP settings.
3. Disable DHCP.
4. Ensure your Debian DHCP server is configured correctly to handle the PXE clients.

---

### 2. Isolate the PXE Boot Network:
- Use a dedicated VLAN or physical network for PXE booting clients.
- Assign your Debian DHCP server to this network and ensure the router's DHCP server does not overlap with it.

---

### 3. Reconfigure Subnet Settings:
- Change the Debian DHCP server to operate within the same subnet as the router (e.g., `192.168.1.0/24`).
- Update your Debian DHCP configuration file (`/etc/dhcp/dhcpd.conf`) to use the same range as the router (e.g., `192.168.1.20 - 192.168.1.254`).
- Restart the Debian DHCP service:
  ```bash
  sudo systemctl restart isc-dhcp-server
