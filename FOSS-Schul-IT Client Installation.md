# FOSS-Schul-IT Client Installation ||*TODO: AUTOMATE THIS*||

## 1. Add hosts entry
1. Edit the file:
   ```bash
   nano /etc/hosts
   ```
2. Add:
   ```bash
   192.168.1.48                ldap.school.local
   ```
   
---

## 2. Install Required Packages
1. Install sssd, libnss-sss, libpam-sss, and other required packages:
    ```bash
    sudo apt update
    sudo apt install sssd libnss-sss libpam-sss libpam-krb5
    ```
2. Create or edit the SSSD configuration file /etc/sssd/sssd.conf:
    ```bash
    sudo nano /etc/sssd/sssd.conf
    ```
   Add the following content:
    ```bash
    [sssd]
    services = nss, pam
    config_file_version = 2
    domains = school.local
   
    [domain/school.local]
    id_provider = ldap
    auth_provider = krb5
    chpass_provider = krb5
    ldap_uri = ldap://ldap.school.local
    ldap_search_base = dc=school,dc=local
    krb5_realm = SCHOOL.LOCAL
    krb5_server = ldap.school.local
    ```
3. Set the correct permissions and restart the service:
    ```bash
    sudo chmod 600 /etc/sssd/sssd.conf
    sudo systemctl restart sssd
    ```
---

## 3. Update PAM and NSS Configuration
1. Update PAM and NSS Configuration:
   Update /etc/nsswitch.conf:
    ```bash
    sudo sed -i 's/^passwd:.*/passwd:         compat sss/' /etc/nsswitch.conf
    sudo sed -i 's/^group:.*/group:          compat sss/' /etc/nsswitch.conf
   ```
2. Enable SSSD for PAM:
    ```bash
    sudo pam-auth-update
   ```
    Select "SSSD" and "Kerberos authentication".

---

## 4. Test LDAP and Kerberos
1. Test LDAP integration:
     ```bash
     getent passwd johndoe
     ```
   Test Kerberos authentication:
     ```bash
     kinit johndoe
     klist
     ```
