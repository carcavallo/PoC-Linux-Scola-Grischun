# FOSS-Schul-IT Shell Commands
## **1. Boot Server Commands**
### 1.1 Test PXE Boot Functionality
Check if the PXE server is running:
  ```bash
  sudo systemctl status isc-dhcp-server
  sudo systemctl status tftpd-hpa
  ```
Verify PXE boot files:
  ```bash
  ls /srv/tftp/
  ```
Test TFTP connectivity:
  ```bash
  tftp <PXE_SERVER_IP>
  tftp> get pxelinux.0
  tftp> quit
  ```
### 1.2 Check DHCP Configuration
Test DHCP server:
  ```bash
  sudo dhcpd -t
  ```

### 1.3 Restart PXE and DHCP Services
Restart the PXE server:
  ```bash
  sudo systemctl restart tftpd-hpa
  ```
Restart the DHCP server:
  ```bash
  sudo systemctl restart isc-dhcp-server
  ```
---

## **2. LDAP Commands**
### 2.1 Test LDAP Connectivity
Search LDAP database:
  ```bash
  ldapsearch -x -H ldap://ldap.school.local -b "dc=school,dc=local" -D "cn=admin,dc=school,dc=local" -w <admin_password>
  ```
Check specific user:
  ```bash
  ldapsearch -x -H ldap://ldap.school.local -b "dc=school,dc=local" "(uid=johndoe)"
  ```
### 2.2 Add a New User to LDAP
Generate a password hash:
  ```bash
  slappasswd
  ```

Create a user LDIF file (example johndoe.ldif):
  ```bash
  dn: uid=johndoe,ou=People,dc=school,dc=local
  objectClass: inetOrgPerson
  objectClass: posixAccount
  objectClass: shadowAccount
  cn: John Doe
  sn: Doe
  uid: johndoe
  uidNumber: 1000
  gidNumber: 1000
  homeDirectory: /home/johndoe
  loginShell: /bin/bash
  userPassword: {SSHA}<password_hash>
  ```
Add the user:
  ```bash
  ldapadd -x -D "cn=admin,dc=school,dc=local" -w <admin_password> -H ldap://localhost -f johndoe.ldif
  ```
### 2.3 List Users in LDAP
Search all users:
  ```bash
  ldapsearch -x -H ldap://ldap.school.local -b "dc=school,dc=local" "(objectClass=posixAccount)"
  ```
### 2.4 Delete a User from LDAP
Remove a user:
  ```bash
  ldapdelete -x -D "cn=admin,dc=school,dc=local" -w <admin_password> "uid=johndoe,ou=People,dc=school,dc=local"
  ```

---

## **3. Kerberos Commands**
### 3.1 Add a User to Kerberos
Create a new principal:
  ```bash
  sudo kadmin.local -q "addprinc johndoe"
  ```
### 3.2 Verify Kerberos Principal
List all principals:
  ```bash
  sudo kadmin.local -q "listprincs"
  ```
### 3.3 Test Kerberos Authentication
Authenticate a user:
  ```bash
  kinit johndoe
  ```
Verify the ticket:
  ```bash
  klist
  ```
Destroy the ticket:
  ```bash
  kdestroy
  ```

## **4. Troubleshooting Commands**
### 4.1 SSSD Logs
View SSSD logs:
  ```bash
  sudo tail -f /var/log/sssd/sssd_*.log
  ```
### 4.2 LDAP Logs
View LDAP logs:
  ```bash
  sudo journalctl -u slapd
  ```
### 4.3 DNS Configuration
Test DNS resolution:
  ```bash
  nslookup ldap.school.local
  ```
