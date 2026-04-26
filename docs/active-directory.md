# 🪟 Active Directory & DNS

## Overview

Active Directory is deployed as the central identity and DNS service in the lab.

It provides:

- authentication (users and devices)
- DNS for internal name resolution
- basic group-based access control

---

## 🧱 Environment

### Domain Controller

- **Hostname:** `WinServerLAB`
- **Role:** Domain Controller and DNS Server
- **IP Address:** `192.168.20.10`
- **Network:** `SERVERS`

### Domain

- **Domain Name:** `corp.lab`
- **NetBIOS Name:** `CORP`

---

## ⚙️ Setup Summary

The Active Directory environment was configured with the following steps:

1. Installed Windows Server.
2. Assigned a static IP address.
3. Installed the Active Directory Domain Services role.
4. Promoted the server to a Domain Controller.
5. Created a new forest: `corp.lab`.

---

## 🌐 DNS Configuration

DNS is integrated with Active Directory and provides internal name resolution for the lab.

### Zones

- **Forward Lookup Zone:** `corp.lab`
- **Reverse Lookup Zone:** `192.168.20.0/24`

### Example Records

| Record | Target |
|---|---|
| `WinServerLAB.corp.lab` | `192.168.20.10` |
| `nginx.corp.lab` | Kubernetes ingress / service endpoint |

### Behaviour

- Domain clients use Active Directory DNS.
- Internal services are resolved using domain names.
- External DNS queries are forwarded upstream through pfSense.

---

## 🏢 OU Structure

The directory is organised into a simple enterprise-style structure:

- `Admins`
- `Users`
- `Clients`
- `Servers`
- `Groups`

### Default Object Placement

Default object placement was adjusted to keep the directory organised:

- New users are placed in `OU=Users`.
- New computers are placed in `OU=Clients`.

This makes Group Policy assignment and object management easier.

---

## 👤 Users and Groups

- Test users were created for authentication validation.
- Groups were created for basic role-based access control.

---

## 💻 Domain Join

### Windows Client

A Windows client was joined to the domain:

- **Domain:** `corp.lab`
- **Validation:** login using domain credentials works

---

## 🔐 Group Policy: WinRM

WinRM was enabled through Group Policy to support remote management.

Configured items:

- enable WinRM service
- allow remote connections
- configure trusted hosts

---

## 🔗 Integration with the Lab

Active Directory is integrated with other lab components:

- **pfSense** forwards DNS queries to Active Directory.
- **Clients** authenticate against Active Directory and use AD DNS.
- **Kubernetes** services can be accessed using internal DNS records such as `nginx.corp.lab`.

---

## 🔍 Validation and Testing

### Client to Domain Controller Communication

#### 1. DNS Resolution

~~~powershell
Resolve-DnsName corp.lab
Resolve-DnsName nginx.corp.lab
~~~

This verifies that the client can resolve internal domain records.

#### 2. Connectivity to Key AD Services

~~~powershell
Test-NetConnection 192.168.20.10 -Port 53
Test-NetConnection 192.168.20.10 -Port 389
Test-NetConnection 192.168.20.10 -Port 445
~~~

This validates communication with key services:

| Port | Service | Purpose |
|---:|---|---|
| 53 | DNS | name resolution |
| 389 | LDAP | directory communication |
| 445 | SMB | domain communication and policies |

#### 3. Domain Trust / Secure Channel

~~~powershell
Test-ComputerSecureChannel -Verbose
~~~

This verifies that the Windows client is correctly joined to the domain.

---

## 📸 Screenshots

### Domain Controller

![AD setup](screenshots/ad/ad-dc.png)

### DNS Configuration

![DNS records](screenshots/ad/dns.png)

### Client Validation

![client validation](screenshots/ad/client-validation.png)

---

## 🧠 Key Design Decisions

- Single Domain Controller for lab simplicity.
- AD-integrated DNS for realistic internal name resolution.
- Structured OU layout from the beginning.
- Centralised authentication for users and devices.

---

## ⚠️ Limitations

- Single Domain Controller, no redundancy.
- No replication or failover.
- Minimal Group Policy setup.

---

## 📌 Summary

Active Directory acts as the central identity and DNS system in the lab.

Validation confirms that:

- domain authentication works
- DNS resolution is correct
- client to Domain Controller communication is functional

The setup reflects a simplified but realistic enterprise environment.
