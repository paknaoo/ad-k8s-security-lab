# 🔥 pfSense Configuration (Step-by-Step)

## Overview

pfSense is the central networking component of this lab and acts as the primary control plane for all network communication.

It is responsible for:

* routing between network segments
* enforcing firewall policies
* providing DHCP services
* forwarding DNS traffic to Active Directory

All inter-network communication is inspected and controlled here, making pfSense the core security layer of the environment.

---

## 🧱 Initial Setup

### 1. Installation

* pfSense deployed as a virtual machine in VMware Workstation
* Multiple virtual NICs assigned:

  * WAN (external access)
  * LAN (management)
  * OPT1–OPT5 (segmented internal networks)

---

### 2. Interface Assignment

Each VMware virtual network is mapped to a dedicated pfSense interface.

| Interface | Role            | Subnet          |
| --------- | --------------- | --------------- |
| WAN       | Internet access | DHCP            |
| LAN       | MGMT            | 192.168.0.0/24  |
| OPT1      | ATTACK          | 192.168.10.0/24 |
| OPT2      | SERVERS         | 192.168.20.0/24 |
| OPT3      | CLIENTS         | 192.168.30.0/24 |
| OPT4      | SECURITY        | 192.168.40.0/24 |
| OPT5      | DMZ             | 192.168.50.0/24 |

Due to VMware Workstation limitations, segmentation is implemented using **separate virtual networks instead of VLAN tagging**.

---

## 🌐 Interface Configuration

Each internal interface is configured with a static gateway IP:

* LAN → 192.168.0.254
* OPT1 → 192.168.10.254
* OPT2 → 192.168.20.254
* OPT3 → 192.168.30.254
* OPT4 → 192.168.40.254
* OPT5 → 192.168.50.254

WAN interface obtains IP via DHCP.

---

## 🔐 Management Access Control

Administrative access to the pfSense web interface is restricted exclusively to the MGMT network.

* Allowed: `192.168.0.0/24`
* Denied: all other networks

This enforces a dedicated management plane and reduces the attack surface.

---

## 📡 DHCP Configuration

### Dynamic Address Pools

#### ATTACK (OPT1)

* Range: `192.168.10.100–200`
* Client: Kali Linux

#### CLIENTS (OPT3)

* Range: `192.168.30.100–200`
* Client: Windows 11

---

### Static Mappings (Servers)

Critical infrastructure uses static addressing:

* Domain Controller (AD/DNS) → `192.168.20.10`
* Kubernetes Control Plane → `192.168.20.20`
* Kubernetes Workers → `192.168.20.21–22`

This ensures consistent service discovery and stable DNS records.

---

## 🌍 DNS Configuration

pfSense acts as a DNS forwarder and delegates resolution to Active Directory.

* Domain: `corp.lab`
* DNS Server: `192.168.20.10`

### Behavior

* DHCP clients receive:

  * DNS server (AD)
  * domain suffix (`corp.lab`)

* Internal name resolution is handled entirely by AD DNS

This mirrors real enterprise environments where identity and DNS are tightly coupled.

---

## 🔥 Firewall Policy Design

### Default Policy

**Deny all traffic by default**

All communication between networks must be explicitly allowed.

---

## 🔐 Interface Rules

### 🟢 MGMT (LAN)

* Full administrative access to all networks
* Allowed protocols:

  * SSH (22)
  * RDP (3389)
  * HTTPS (443)
* Used as the only trusted management zone

---

### 🟡 SERVERS (OPT2)

* Accept inbound traffic only from:

  * MGMT
  * CLIENTS (limited services)

* Block unsolicited traffic from ATTACK network

---

### 🔵 CLIENTS (OPT3)

Allowed access to core infrastructure services:

* **Active Directory services:**

  * DNS (53) – domain resolution
  * Kerberos (88) – authentication
  * LDAP (389) – directory services
  * SMB (445) – domain resources and policies

* Limited access to internal applications (e.g. Kubernetes services)

---

### 🔴 ATTACK (OPT1)

Designed as an untrusted testing zone:

* Allowed:

  * outbound traffic for reconnaissance and testing

* Restricted:

  * no access to MGMT network
  * no direct access to CLIENTS

* Controlled access to SERVERS for security testing (e.g. Nmap scans)

---

### 🟣 SECURITY (OPT4)

* Reserved for future monitoring stack
* No active rules yet

---

### ⚫ DMZ (OPT5)

* Fully isolated by default
* Intended for future public-facing services

---

## 🔄 Connectivity Validation

After configuration, network functionality was verified.

### Network Reachability

* MGMT → all networks ✔
* CLIENTS → Domain Controller ✔
* ATTACK → SERVERS (controlled) ✔

---

### DNS Resolution

* `corp.lab` successfully resolved via AD DNS ✔

---

## 📸 Snapshot Strategy

Snapshots were created after each major configuration step:

* initial installation
* interface setup
* DHCP configuration
* firewall rule implementation

This enables safe rollback during testing and experimentation.

---

## 🧠 Key Design Decisions

* Segmentation enforced at interface level (no VLANs)
* Centralized traffic control via pfSense
* Default deny security model
* Dedicated management network
* Active Directory used as authoritative DNS

---

## ⚠️ Summary

pfSense acts as the backbone of the lab's network architecture.

It enforces segmentation, controls traffic flow, and ensures that all communication between systems is intentional, auditable, and aligned with security best practices.
