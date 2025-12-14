# k8s-configs

A central repository for Kubernetes YAML manifests, configuration files, and common setup patterns.

### 🎯 Goal
This repo serves as a "parts bin" for Kubernetes clusters. Instead of rewriting standard deployments or service configurations from scratch every time, I store modular configs here to be pulled down as needed.

### 📦 Contents
* **Base cluster configurations:** Essential setups for new clusters.
* **Common application manifests:** Templates for Ingress, Monitoring, and Logging.
* **Reusable templates:** Standard Service and Deployment definitions.

### 🚀 Usage
Browse the folders, find the component you need, and run:

```bash
kubectl apply -f <filename>.yaml
