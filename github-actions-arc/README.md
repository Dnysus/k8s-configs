# GitHub Actions ARC Runner Configuration

This directory contains the Kubernetes configuration for GitHub Actions Runner Controller (ARC) with an optimized pod scheduling strategy for cost efficiency.

## Workload Scheduling Strategy

The configuration uses node affinity and tolerations to separate workloads based on their resource requirements and runtime characteristics.

### Runner Pods (`workload-type: build-runner`)

The ephemeral runner pods are scheduled on nodes labeled with `workload-type: build-runner`. These nodes are:

- **Larger instance types** with more CPU and memory (runners request 15 CPU / 26Gi memory)
- **Ephemeral** — they scale to zero when no jobs are running
- **Cost-optimized** — only running when actively processing CI/CD jobs

Runner pods also have pod anti-affinity rules to spread across different nodes, preventing multiple runners from landing on the same host.

### Listener & System Pods (`workload-type: system`)

The listener pod (which monitors for new GitHub Actions jobs) is scheduled on nodes labeled with `workload-type: system`. These nodes are:

- **Smaller instance types** with minimal resources
- **Always running (24/7)** to continuously poll for new workflow jobs
- **Lower cost** — sized appropriately for lightweight, long-running processes

## Node Labels Required

Ensure your Kubernetes nodes are labeled appropriately:

```bash
# For build runner nodes (larger instances)
kubectl label node <node-name> workload-type=build-runner

# For system/listener nodes (smaller instances)
kubectl label node <node-name> workload-type=system
```

## Node Taints (Optional)

The listener template includes tolerations for a `workload-type=system:NoSchedule` taint. If you want to reserve system nodes exclusively for these workloads:

```bash
kubectl taint node <node-name> workload-type=system:NoSchedule
```

