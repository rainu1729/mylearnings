# Kubernetes Local Setup Guide

This guide describes how to install the `kubectl` CLI and set up a local Kubernetes cluster using `minikube` or `kind`.

---

## 1. Install `kubectl` CLI

Download and install the Kubernetes command-line tool.

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Verify installation:
```bash
kubectl version --client
```

---

## 2. Spin up a Local Cluster

### Option A: Using Minikube (Recommended)

`minikube` runs a single-node Kubernetes cluster inside a container or virtual machine.

1. **Install Minikube**:
   ```bash
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   sudo install minikube-linux-amd64 /usr/local/bin/minikube
   ```

2. **Start Cluster**:
   ```bash
   minikube start --driver=docker
   ```

### Option B: Using Kind (Kubernetes in Docker)

`kind` is a tool for running local Kubernetes clusters using Docker container "nodes".

1. **Install Kind**:
   ```bash
   # For AMD64 / x86_64
   [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
   chmod +x ./kind
   sudo mv ./kind /usr/local/bin/kind
   ```

2. **Create Cluster**:
   ```bash
   kind create cluster --name my-cluster
   ```

---

## 3. Verify Cluster Connection

Confirm that `kubectl` can communicate with your local cluster:

```bash
kubectl cluster-info
kubectl get nodes
```
