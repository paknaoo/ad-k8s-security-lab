# ☸️ Kubernetes Cluster Setup (kubeadm)

## Overview

A Kubernetes cluster was deployed in the SERVERS network to simulate a modern application platform within a segmented enterprise environment.

The cluster consists of:

* 1 control-plane node
* 2 worker nodes

It was built using **kubeadm**, which provides a more transparent and hands-on setup compared to managed Kubernetes solutions.

---

## 🧱 Environment

| Node        | Role          | IP Address    |
| ----------- | ------------- | ------------- |
| k8s-master  | Control Plane | 192.168.20.20 |
| k8s-worker1 | Worker        | 192.168.20.21 |
| k8s-worker2 | Worker        | 192.168.20.22 |

All nodes run **Ubuntu Server** and are located in the SERVERS network.

---

## ⚙️ Control Plane Setup

### 1. Base System Preparation

The control-plane node was prepared with standard Kubernetes prerequisites:

* system update
* swap disabled
* swap entry removed from `/etc/fstab`

```bash
swapoff -a
```

---

### 2. Kernel Modules

Required kernel modules were enabled:

```bash
modprobe overlay
modprobe br_netfilter
```

Persisted configuration:

```bash
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

---

### 3. Sysctl Configuration

Networking parameters required by Kubernetes:

```bash
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```

---

### 4. Container Runtime

**containerd** was installed and configured as the container runtime.

---

### 5. Kubernetes Components

Installed:

* kubeadm
* kubelet
* kubectl

---

### 6. Cluster Initialisation

```bash
kubeadm init
```

---

### 7. kubectl Configuration

```bash
mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## 🧩 Worker Nodes Setup

Both worker nodes were prepared using the same steps:

### 1. Base Preparation

* system update
* swap disabled
* kernel modules enabled
* sysctl configured

---

### 2. Runtime and Components

* containerd installed
* kubeadm installed
* kubelet installed
* kubectl installed

---

### 3. Join Cluster

```bash
kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash <HASH>
```

---

## 🌐 Cluster Networking

A Container Network Interface (CNI) plugin is required for pod networking.

The final configuration uses **Calico**:

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### Networking Note

Only a single CNI plugin should be active in a cluster.

During initial setup, multiple networking solutions were tested, which led to connectivity issues between nodes. The final configuration uses **Calico only**, providing stable pod networking and better alignment with production environments.

---

## 🚀 Application Deployment

Applications were deployed using **declarative YAML manifests**, which aligns with standard Kubernetes practices.

### NGINX Deployment

The deployment uses multiple replicas to ensure the workload is distributed across worker nodes and remains available in case of node failure.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Applied with:

```bash
kubectl apply -f deployment.yaml
```

---

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

---

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: nginx.corp.lab
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

---

## 🔄 DNS Integration

An internal DNS record was created in Active Directory:

* `nginx.corp.lab` → Kubernetes service / ingress

This allows access using a domain name instead of an IP address.

---

## 🔍 Validation

```bash
kubectl get nodes
kubectl get pods -A
kubectl get svc
kubectl get ingress
```

Verified:

* nodes joined successfully ✔
* pods running on workers ✔
* service reachable ✔
* ingress resolving via DNS ✔

---

## 🔗 Integration with Lab

### pfSense

* controls traffic between CLIENTS and Kubernetes

### Active Directory

* provides DNS resolution

### Clients

* access applications via domain name

---

## 🧠 Key Design Decisions

* kubeadm used for full control and learning
* containerd as lightweight runtime
* Calico as final CNI
* declarative YAML configuration
* multi-replica deployment for basic high availability
* cluster size aligned with hardware constraints

---

## ⚠️ Limitations

* single control-plane (no HA at control level)
* no persistent storage
* no CI/CD pipeline yet

---

## 📌 Summary

This Kubernetes setup provides a practical and realistic environment for running containerised workloads.

It integrates with the wider lab (pfSense + Active Directory) and demonstrates how Kubernetes fits into a segmented enterprise network.
