# k8s-configs
A central repository for Kubernetes YAML manifests, configuration files, and common setup patterns.

Goal: This repo serves as a "parts bin" for Kubernetes clusters. Instead of rewriting standard deployments or service configurations from scratch every time, I store modular configs here to be pulled down as needed.

Contents:

Base cluster configurations.

Common application manifests (Ingress, Monitoring, Logging).

Reusable service and deployment templates.

Usage: Browse the folders, find the component you need, and ```kubectl apply -f``` (or copy into your project's specific repo).
