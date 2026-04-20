# 🪟 Active Directory & DNS Configuration

## Overview

Active Directory (AD) is deployed as the central identity service for the lab environment.

It provides:

* authentication (users and devices)
* authorisation (group-based access control)
* DNS for internal name resolution

The setup is designed to reflect a small but realistic enterprise environment, with structured organisation and centralised management.

---

## 🧱 Environment Setup

### Server Details

* Hostname: `DC01`
* Role: Domain Controller + DNS Server
* IP Address: `192.168.20.10`
* Network: SERVERS (OPT2)

---

### Domain Configuration

* Domain Name: `corp.lab`
* NetBIOS Name: `CORP`

The domain acts as the central authority for:

* user authentication
* device management
* internal DNS

---

## ⚙️ Active Directory Installation

### Steps Performed

1. Installed Windows Server
2. Assigned static IP address
3. Installed **Active Directory Domain Services (AD DS)** role
4. Promoted server to Domain Controller
5. Created a new forest: `corp.lab`

---

## 🌍 DNS Configuration

DNS is integrated with Active Directory and acts as the primary name resolution system.

### Configured Components

* Forward Lookup Zone:

  * `corp.lab`

* Reverse Lookup Zone:

  * `192.168.20.0/24`

* DNS Forwarder:

  * external DNS (via pfSense / WAN)

---

### Example Records

* `dc01.corp.lab` → 192.168.20.10
* `nginx.corp.lab` → Kubernetes service / ingress

---

### Behaviour

* All domain-joined clients use AD DNS
* Internal services are resolved using domain names
* External queries are forwarded upstream

---

## 🏢 Organisational Unit (OU) Structure

The directory is structured to reflect typical enterprise practices.

### OUs Created

* **Admins**
* **Users**
* **Clients**
* **Servers**
* **Groups**

---

### Default Object Placement (Redirection)

By default, Active Directory places new objects in:

* `CN=Users`
* `CN=Computers`

This was changed to improve organisation and policy management.

### Configuration Applied

* New **user accounts** are automatically placed in:

  * `OU=Users`

* New **computer objects** are automatically placed in:

  * `OU=Clients`

This ensures that:

* Group Policies can be applied consistently
* Objects are organised from the moment they are created
* No manual cleanup is required

---

## 👤 Users & Groups

### Users

* Test user accounts created for authentication and validation

### Groups

* Used to organise permissions and simulate role-based access

This allows:

* cleaner access control
* easier policy assignment

---

## 💻 Domain Join Process

### Windows 11 Client

* Host renamed
* Joined to domain: `corp.lab`

### Validation

* Login using domain credentials ✔
* DNS resolution working ✔
* Communication with Domain Controller verified ✔

---

## 🔐 Group Policy Configuration

### WinRM Configuration

WinRM was enabled via Group Policy to allow remote management.

Configured settings:

* Enable WinRM service
* Allow remote connections
* Configure trusted hosts

---

## 🔄 Integration with Network

Active Directory is integrated with the rest of the lab:

### pfSense

* Forwards DNS queries to AD

### Clients

* Authenticate against AD
* Use AD DNS for name resolution

### Kubernetes

* Internal services registered in DNS (e.g. `nginx.corp.lab`)

---

## 🔍 Validation & Testing

### Authentication

* Domain login successful ✔

### DNS

* Internal name resolution working ✔

### Connectivity

* Client ↔ Domain Controller communication verified ✔

---

## 🧠 Key Design Decisions

* Single Domain Controller (kept simple due to hardware limits)
* AD-integrated DNS for realistic behaviour
* Organised OU structure from the start
* Automatic placement of users and computers into OUs
* Centralised authentication model

---

## ⚠️ Limitations

* Single DC (no redundancy)
* No replication or failover
* Simplified Group Policy setup

These trade-offs were made due to hardware constraints.

---

## 📌 Summary

Active Directory acts as the identity backbone of the lab.

It centralises authentication, provides DNS resolution, and enables structured management of users and devices — closely reflecting how enterprise environments are designed and operated.
