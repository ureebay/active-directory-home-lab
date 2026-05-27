# Active Directory Home Lab

A fully functional Windows Active Directory environment built on Oracle VirtualBox, simulating a small corporate network with a Domain Controller, DHCP, DNS, NAT routing, and a Windows 10 client joined to the domain. Over 1,000 user accounts were bulk-provisioned via PowerShell automation.

---

## Environment Overview

| Component | Details |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Domain Controller OS | Windows Server 2019 Evaluation |
| Client OS | Windows 10 Pro |
| Domain Name | mydomain.com |
| Internal IP Scheme | 172.16.0.0/24 |
| DC Internal IP | 172.16.0.1 |
| DHCP Scope | 172.16.0.100 - 172.16.0.200 |

---

## Network Architecture

```
Internet
    |
[NAT Adapter] <-- DC External NIC (DHCP from home router)
    |
[Domain Controller - Windows Server 2019]
    - Active Directory Domain Services
    - DNS Server
    - DHCP Server (172.16.0.100-200)
    - RAS / NAT (routes internal traffic to internet)
    |
[Internal VirtualBox Network]
    |
[CLIENT1 - Windows 10 Pro]
    - Joined to mydomain.com
    - Gets IP from DC DHCP
    - Uses DC as DNS and default gateway
```

---

## Lab Build Steps

### Step 1: VM Creation and Server 2019 Installation

Created the Domain Controller VM in VirtualBox with two network adapters:
- Adapter 1: NAT (connects to home internet)
- Adapter 2: Internal Network (connects to private lab network)

Allocated 2048 MB RAM and 4 CPU cores, then booted from the Windows Server 2019 ISO and completed a clean install with the Desktop Experience option.

![DC VM Setup](screenshots/dc-virtualbox-settings.png)
![Server 2019 Installing](screenshots/dc-server2019-installing.png)
![Admin Password Setup](screenshots/server2019-admin-password-setup.png)

---

### Step 2: Network Adapter Identification and IP Configuration

After boot, identified the two NICs by checking their details in Network Connections. The adapter showing a 10.x.x.x address was the external (internet-facing) NIC, renamed to _INTERNET_. The adapter showing a 169.254.x.x autoconfiguration address was the internal NIC, renamed to X_Internal_X.

Manually assigned a static IP to the internal NIC:
- IP: 172.16.0.1
- Subnet Mask: 255.255.255.0
- Default Gateway: (none, DC is the gateway)
- DNS: 127.0.0.1 (loopback, DC serves as its own DNS after AD install)

![External NIC - Internet](screenshots/nic-external-internet-details.png)
![Internal NIC - 169.254 Autoconfig](screenshots/nic-internal-169-autoconfig.png)

---

### Step 3: Active Directory Domain Services Installation

Installed the Active Directory Domain Services role via Server Manager > Add Roles and Features. After installation completed, ran the post-deployment configuration to promote the server to a Domain Controller and created a new forest with the domain name mydomain.com.

![AD DS Role Installation](screenshots/adds-role-installation.png)
![AD DS Configuration Wizard](screenshots/adds-configuration-wizard.png)

---

### Step 4: Dedicated Domain Admin Account Creation

Instead of using the built-in Administrator account, created a dedicated domain admin account following a real-world naming convention (a-suribe), placed inside a custom _ADMINS Organizational Unit. Added the account to the Domain Admins security group via Member Of > Add.

![Creating Domain Admin Account](screenshots/ad-create-domain-admin-user.png)

---

### Step 5: RAS / NAT Configuration

Installed the Remote Access role (with the Routing sub-role) to enable NAT, allowing the internal Windows 10 client to reach the internet through the Domain Controller. Configured the _INTERNET_ adapter as the public interface.

![Remote Access Role Installation](screenshots/remote-access-role-installation.png)
![NAT Interface Selection](screenshots/nat-interface-selection.png)

---

### Step 6: DHCP Server Setup

Installed the DHCP Server role and configured a scope to automatically assign IP addresses to clients on the internal network:

- Scope Name: 172.16.0.0
- Range: 172.16.0.100 to 172.16.0.200
- Subnet Mask: 255.255.255.0
- Default Gateway (Router): 172.16.0.1 (the DC)
- DNS Server: 172.16.0.1 (the DC)

Authorized the DHCP server in Active Directory after setup.

![DHCP Scope Configured](screenshots/dhcp-scope-configured.png)

---

### Step 7: PowerShell Bulk User Creation

Executed a PowerShell script to automatically create over 1,000 user accounts in Active Directory, pulling names from a plain text file (names.txt). Personal account suribe was added to the top of the names list.

**Script breakdown:**

```powershell
$PASSWORD_FOR_USERS   = "Password!2"
$USER_FIRST_LAST_LIST = Get-Content .\names.txt

$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first    = $n.Split(" ")[0].ToLower()
    $last     = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()

    New-AdUser -AccountPassword $password `
               -GivenName $first `
               -Surname $last `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_USERS,$((([ADSI]'').distinguishedName))" `
               -Enabled $true
}
```

Before running, set execution policy to unrestricted:
```powershell
Set-ExecutionPolicy Unrestricted
```

![PowerShell Script Open in ISE](screenshots/powershell-script-open.png)
![Users Being Created in Output](screenshots/powershell-users-being-created.png)
![AD Users and Computers - _USERS OU Populated](screenshots/ad-users-ou-populated.png)

---

### Step 8: Windows 10 Client VM Creation

Created a second VM (CLIENT1) in VirtualBox configured with only one network adapter set to Internal Network, so it receives its IP address from the DC's DHCP server. Installed Windows 10 Pro (required for domain join; Home edition cannot join a domain).

![Windows 10 Installing](screenshots/client1-windows10-installing.png)
![Local User Setup](screenshots/client1-local-user-setup.png)

---

### Step 9: Domain Join and Connectivity Verification

Renamed the machine to CLIENT1 and joined it to mydomain.com simultaneously via System > Rename this PC (Advanced). Verified full connectivity from CLIENT1 via Command Prompt:

```
ipconfig        --> 172.16.0.100, gateway 172.16.0.1, DNS suffix mydomain.com
ping google.com --> successful (confirms NAT routing working end-to-end)
ping mydomain.com --> successful (confirms DNS resolving through DC)
whoami          --> mydomain\suribe (confirms domain login working)
```

![ipconfig and ping google.com](screenshots/client1-ipconfig-ping-google.png)
![ping mydomain.com](screenshots/client1-ping-mydomain.png)
![whoami - mydomain\suribe](screenshots/client1-whoami-domain-user.png)

---

## Key Concepts Demonstrated

**Active Directory** - Created a domain, Organizational Units, security groups, and bulk user provisioning via PowerShell automation.

**DNS** - DC serves as the authoritative DNS server for the internal domain. Clients resolve both internal hostnames and external domains through it.

**DHCP** - Scoped address assignment for internal clients with correct gateway and DNS options pushed automatically.

**NAT / Routing** - Internal clients on a private 172.16.0.0/24 network reach the public internet through the DC acting as a router, with the external IP NATed to the home network.

**PowerShell Automation** - Bulk provisioned 1,000+ AD user accounts programmatically from a flat text file, demonstrating real-world IT automation patterns.

**Domain Join** - CLIENT1 joined to mydomain.com, allowing any of the 1,000+ provisioned domain accounts to authenticate on the machine.

---

## Tools and Technologies

- Oracle VirtualBox
- Windows Server 2019
- Windows 10 Pro
- Active Directory Domain Services (AD DS)
- DNS Server
- DHCP Server
- Remote Access / NAT
- Windows PowerShell ISE

---

## Acknowledgments

Lab environment based on the Active Directory home lab tutorial by [Josh Madakor](https://www.youtube.com/@JoshMadakor). All configuration, documentation, and screenshots are my own work following his guided project.

---

## Author

Sebastian Uribe | [LinkedIn](https://www.linkedin.com/in/s1uribe/) | MIS Student, George Mason University
