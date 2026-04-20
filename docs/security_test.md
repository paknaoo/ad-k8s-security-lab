# 🛡️ Security & Network Testing

## Overview

This lab includes a dedicated ATTACK network used to simulate basic offensive security scenarios and validate network segmentation.

The goal is not full penetration testing, but rather:

* verifying firewall rules
* testing network exposure
* understanding how services are visible across segments

Kali Linux is used as the primary testing host.

---

## 🧱 Security Model

The environment follows a segmented architecture:

* **MGMT** → trusted (administration)
* **SERVERS** → critical infrastructure
* **CLIENTS** → user layer
* **ATTACK** → untrusted / hostile
* **SECURITY** → monitoring (planned)
* **DMZ** → external services (planned)

All traffic between these networks is controlled by pfSense.

---

## 🎯 Testing Objectives

Security testing focused on:

* verifying that only required services are exposed
* confirming that sensitive networks are isolated
* validating firewall rules in real conditions
* identifying potential attack paths

---

## ⚔️ Attack Simulation Environment

### Kali Linux (ATTACK Network)

* IP: `192.168.10.100`
* Role: simulated attacker

This host is treated as an untrusted system with limited permissions.

---

## 🔍 Network Scanning (Nmap)

Basic reconnaissance was performed against the SERVERS network.

---

### 1. Host Discovery

```bash id="7wt3j8"
nmap -sn 192.168.20.0/24
```

Purpose:

* identify active hosts
* map the network

---

### 2. Service Detection

```bash id="3l1p0p"
nmap -sV 192.168.20.0/24
```

Purpose:

* detect running services
* identify open ports

---

### 3. OS Detection

```bash id="4p2rcp"
nmap -O 192.168.20.10
```

Purpose:

* identify operating system (e.g. Windows Server)

---

### 4. Full Scan

```bash id="jjhkhf"
nmap -A 192.168.20.0/24
```

Purpose:

* combined scan (services, OS, scripts)
* deeper reconnaissance

---

### 5. Full Port Scan (Kubernetes Node)

```bash id="l0n4hj"
nmap -p- 192.168.20.20
```

Purpose:

* detect all open ports
* identify exposed services on cluster nodes

---

### 6. Vulnerability Scan (NSE)

```bash id="u7k3ka"
nmap --script vuln 192.168.20.0/24
```

Purpose:

* run vulnerability detection scripts
* identify known weaknesses and misconfigurations

---

### 7. HTTP Enumeration (NSE)

```bash id="y1mkcb"
nmap --script http-title,http-enum,http-headers nginx.corp.lab
```

Purpose:

* retrieve webpage titles
* enumerate common directories and endpoints
* analyse HTTP response headers

---

### 8. Web Service Testing

```bash id="m5l1o2"
nmap -p 80,443 nginx.corp.lab
```

Purpose:

* verify application exposure
* confirm DNS resolution works correctly

---

## 🌐 DNS Testing (dig)

DNS resolution was tested directly against internal hosts using `dig`.

### Domain Controller

```bash id="9k7r5o"
dig @192.168.20.10 corp.lab
```

### Kubernetes Node

```bash id="3q0p9m"
dig @192.168.20.20 corp.lab
```

### Observations

* Domain Controller correctly responds to DNS queries ✔
* Kubernetes node does not act as a DNS server (expected behaviour) ✔
* Internal domain resolution is handled centrally by Active Directory

This confirms that DNS is properly centralised and not distributed across infrastructure nodes.

---

## 🔐 Findings & Observations

### 1. Network Segmentation

* MGMT network is not reachable from ATTACK ✔
* CLIENTS network is isolated ✔
* Access is only possible where explicitly allowed

---

### 2. Service Exposure

* Only the Domain Controller exposes expected services (e.g. DNS – port 53)

* Other hosts in the SERVERS network either:

  * do not expose services externally
  * or are filtered by firewall rules

* No unnecessary open ports detected across the network

This confirms that service exposure is minimal and aligned with the security design.

---

### 3. DNS Behaviour

* Internal domain (`corp.lab`) resolves correctly
* Services accessible via DNS instead of IP

---

### 4. Controlled Access

* ATTACK network can scan SERVERS (intentionally allowed)
* Sensitive management interfaces remain protected

---

## 🧠 Security Approach

The lab follows basic security principles:

### Network Segmentation

Each network has a clearly defined role and trust level.

---

### Least Privilege

Traffic is denied by default and only allowed when required.

---

### Controlled Exposure

Only necessary services are accessible between networks.

---

### Separation of Concerns

Infrastructure, users, and testing environments are isolated.

---

## 🚧 Future Improvements

Planned enhancements:

* Wazuh (SIEM) deployment
* Zabbix monitoring
* centralised logging
* alerting and incident detection

---

## 📌 Summary

Security in this lab is based on segmentation, controlled access, and validation through testing.

By combining pfSense firewall rules with active scanning from the ATTACK network, the environment provides a practical way to understand how infrastructure behaves under basic reconnaissance and how well it is protected.
