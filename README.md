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
