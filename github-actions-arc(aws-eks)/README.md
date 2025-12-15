# GitHub Actions ARC Runner Configuration (AWS EKS)

This directory contains Kubernetes configurations for GitHub Actions Runner Controller (ARC) on AWS EKS, with an optimized pod scheduling strategy for cost efficiency.

## Directory Structure

```
github-actions-arc(aws-eks)/
├── Custom-Resources/
│   ├── Arc-Runner-Set.yaml           # Main runner scale set configuration
│   ├── Arc-Runner-Set-Listener.yaml  # Listener that polls for new jobs
│   └── Ephemeral-Runner-Set.yaml     # Ephemeral runner pod template
└── Deployments/
    ├── arc-gha-rs-controller.yaml    # ARC controller manager
    ├── aws-cluster-autoscaler.yaml   # Scales node groups based on demand
    ├── coredns.yaml                  # DNS for the cluster
    └── metrics-server.yaml           # Metrics for autoscaling decisions
```

## Before You Apply

Replace these placeholders in the configuration files:

| Placeholder | Description | Files |
|-------------|-------------|-------|
| `{organization-name-here}` | Your GitHub org or org/repo | Arc-Runner-Set.yaml, Arc-Runner-Set-Listener.yaml, Ephemeral-Runner-Set.yaml |
| `{your-region}` | AWS region (e.g., `us-east-1`) | metrics-server.yaml, coredns.yaml, aws-cluster-autoscaler.yaml |
| `{your-cluster-name}` | Your EKS cluster name | aws-cluster-autoscaler.yaml |

You'll also need to create a GitHub secret for authentication:

```bash
kubectl create secret generic arc-runner-set-gha-rs-github-secret \
  --namespace=arc-runners \
  --from-literal=github_token=<YOUR_GITHUB_PAT>
```

## Workload Scheduling Strategy

The configuration uses node affinity and tolerations to separate workloads based on their resource requirements and runtime characteristics.

### Runner Pods (`workload-type: build-runner`)

The ephemeral runner pods are scheduled on nodes labeled with `workload-type: build-runner`. These nodes are:

- **Larger instance types** with more CPU and memory (runners request 15 CPU / 26Gi memory)
- **Ephemeral** — they scale to zero when no jobs are running
- **Cost-optimized** — only running when actively processing CI/CD jobs

Runner pods also have pod anti-affinity rules to spread across different nodes, preventing multiple runners from landing on the same host.

### Listener & System Pods (`workload-type: system`)

The listener pod (which monitors for new GitHub Actions jobs) and other system components are scheduled on nodes labeled with `workload-type: system`. These nodes are:

- **Smaller instance types** with minimal resources
- **Always running (24/7)** to continuously poll for new workflow jobs
- **Lower cost** — sized appropriately for lightweight, long-running processes

System deployments on these nodes include:
- ARC Controller Manager
- ARC Listener
- CoreDNS
- Metrics Server
- Cluster Autoscaler

## Node Labels Required

Ensure your Kubernetes nodes are labeled appropriately:

```bash
# For build runner nodes (larger instances)
kubectl label node <node-name> workload-type=build-runner

# For system/listener nodes (smaller instances)
kubectl label node <node-name> workload-type=system
```

## Node Taints (Optional)

The configurations include tolerations for a `workload-type=system:NoSchedule` taint. If you want to reserve system nodes exclusively for these workloads:

```bash
kubectl taint node <node-name> workload-type=system:NoSchedule
```

## Resource Sizing

The runner pod resource limits (15 CPU / 26Gi memory) are tuned for AWS EC2 c8i.4xlarge instances (16 vCPU, 32GB RAM). Adjust these values in the Custom-Resources files based on your node instance types, or remove them to use defaults.
