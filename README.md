# 🔧 Enterprise Homelab: AD + Kubernetes + Security

## 🧠 Overview

This project is a fully segmented enterprise-style homelab designed to simulate a realistic IT infrastructure. It integrates networking, system administration, security, and container orchestration into one cohesive environment.

The lab is built around a central firewall/router (pfSense) and multiple isolated network segments, each representing a different role within an organization.

It serves as a hands-on platform for learning, testing, and validating real-world scenarios without impacting production systems.

---

## 🎯 Purpose

The primary goal of this lab is to move beyond theory and build practical, job-ready skills.

This environment simulates how modern infrastructures actually operate:

* segmented networks with controlled traffic flow
* centralized identity management (Active Directory)
* containerized workloads (Kubernetes)
* attack simulation and network auditing
* foundations for monitoring and security operations

It allows safe experimentation with:

* misconfigurations
* attack scenarios
* service deployments
* troubleshooting complex systems

---

## 🌐 Network Topology

```
WAN
 │
pfSense
 ├── LAN   (MGMT - Ubuntu)
 ├── OPT1  (ATTACK - Kali)
 ├── OPT2  (SERVERS - AD + Kubernetes)
 ├── OPT3  (CLIENT - Windows)
 ├── OPT4  (SECURITY - Wazuh/Zabbix)
 └── OPT5  (DMZ - Load Balancer)
```

- [Full Network Topology](docs/topology.md)

---

## 📡 IP Plan

| Network         | Purpose                     | Gateway        |
| --------------- | --------------------------- | -------------- |
| LAN (MGMT)      | Management                  | 192.168.0.254  |
| OPT1 (ATTACK)   | Attack / testing            | 192.168.10.254 |
| OPT2 (SERVERS)  | AD + Kubernetes             | 192.168.20.254 |
| OPT3 (CLIENTS)  | End users                   | 192.168.30.254 |
| OPT4 (SECURITY) | Monitoring (planned)        | 192.168.40.254 |
| OPT5 (DMZ)      | External services (planned) | 192.168.50.254 |

### Addressing Scheme

* `.1–.9` → infrastructure
* `.10–.50` → servers (static IP)
* `.100–200` → DHCP clients
* `.254` → gateway (pfSense)

---

## 🖥️ Lab Components

### 🟢 Management

* Ubuntu Desktop (DHCP)
* Central administration host

### 🔴 Attack Network

* Kali Linux
* Used for scanning, enumeration, and testing security posture

### 🟡 Servers

* Windows Server (Active Directory + DNS)
* Kubernetes cluster:

  * 1x control-plane
  * 2x workers

### 🔵 Clients

* Windows 11
* Domain joined (`corp.lab`)

### 🟣 Security (Planned)

* Wazuh (SIEM)
* Zabbix (monitoring)

### ⚫ DMZ (Planned)

* Load balancer / reverse proxy

---

## 🔥 pfSense Configuration

* WAN: DHCP
* GUI is restricted to the MGMT network only
* Segmented interfaces (LAN + OPT1–OPT5)
* Firewall rules controlling inter-network traffic
* DHCP:

  * enabled for clients and attack network
  * static mappings for servers
* DNS integrated with Active Directory (`corp.lab`)

---

## 🪟 Active Directory

* Domain: `corp.lab`
* DNS:

  * forwarder configured
  * reverse lookup zone added

### Organizational Structure

* Admins
* Users
* Clients
* Servers
* Groups

### Additional Configuration

* WinRM enabled via Group Policy
* Internal DNS records (e.g. `nginx.corp.lab`)

---

## ☸️ Kubernetes Cluster

### Control Plane

* Swap disabled

* Kernel and sysctl tuning

* Container runtime: containerd

* Installed:

  * kubeadm
  * kubelet
  * kubectl

* Cluster initialized with `kubeadm init`

* Networking:

  * Calico

### Workloads

* NGINX Deployment
* Service exposure
* Ingress configured

### Worker Nodes

* Prepared and joined via `kubeadm join`

---

## 🛠️ Attack & Auditing

Basic reconnaissance performed using Kali Linux:

```bash
# Host discovery
nmap -sn 192.168.20.0/24

# Service detection
nmap -sV 192.168.20.0/24

# OS detection
nmap -O 192.168.20.10

# Full scan
nmap -A 192.168.20.0/24

# Full port scan (Kubernetes node)
nmap -p- 192.168.20.20

# Web services
nmap -p 80,443 nginx.corp.lab
```

---

## 📸 Snapshots

Snapshots were created at each major step:

* base installations
* network configuration
* Active Directory setup
* Kubernetes deployment

This allows fast rollback and safe experimentation.

---

## 🚧 Future Improvements

* Wazuh deployment (SIEM)
* Zabbix monitoring
* DMZ reverse proxy / load balancer
* VPN access to lab
* CI/CD pipelines for Kubernetes

---

## 🧠 Skills Demonstrated

* Network segmentation & firewalling
* Active Directory & DNS management
* Kubernetes cluster deployment
* Linux & Windows administration
* Basic offensive security (recon, scanning)
* Infrastructure troubleshooting

---

## ⚠️ Disclaimer

This lab is intended for educational purposes only.
Do not expose it to the public internet without proper hardening and security controls.
