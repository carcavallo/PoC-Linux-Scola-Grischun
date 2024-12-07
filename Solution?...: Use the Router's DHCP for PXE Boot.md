# Solution: Use the Router's DHCP for PXE Boot

If disabling the router's DHCP service isn't feasible, you can configure the router's DHCP server to work with the PXE server by manually adding the required options.

---

## Steps to Configure Router's DHCP for PXE Boot

### 1. Access Your Router’s Admin Panel
- Use your router's IP address (e.g., `192.168.0.1`) in a web browser to log in.

### 2. Locate the DHCP Settings
- Navigate to the DHCP configuration section of the admin panel.

### 3. Add Custom DHCP Options
- Look for an option to add custom DHCP parameters or advanced settings.
- Add the following options:
  - **Next Server (Option 66)**: Set this to the IP address of your PXE server (e.g., `192.168.0.2`).
  - **Boot Filename (Option 67)**: Set this to the PXE bootloader filename (e.g., `pxelinux.0` or `debian-installer/amd64/bootnetx64.efi`).

### 4. Update the DHCP Range
- Ensure the IP range allocated by the router does not conflict with the PXE server's range if you still use the PXE server’s DHCP.

### 5. Test PXE Boot
- Restart the client machine and check if it successfully gets an IP address and boot instructions from the router’s DHCP server.

---

## Additional Notes

- **Subnet and IP Range Matching**: Ensure that the router's subnet and the PXE server’s IP configuration match (e.g., both are in the `192.168.0.0/24` network).
- **Router Limitations**: Some consumer-grade routers might not support advanced DHCP options (Option 66/67). If so, you'll need to disable the router's DHCP service and rely solely on the PXE server for DHCP.

With these steps, you can leverage your router’s DHCP service while ensuring PXE functionality is intact.
