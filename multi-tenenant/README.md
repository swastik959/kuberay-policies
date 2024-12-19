# KubeRay Multi-Tenant Kyverno Policy

This repository contains a Kyverno policy designed to enforce **multi-tenant isolation** and **network security** for KubeRay clusters in a Kubernetes environment.

---

## Policy Overview

### 1. **Enforce Tenant Isolation**
This rule ensures that `RayCluster` resources comply with multi-tenant isolation policies:
- **Tenant Labeling:** Every `RayCluster` must include a `tenant` label in its metadata.
- **Node Affinity and Tolerations:**
  - Pods are restricted to nodes labeled with their corresponding tenant.
  - Pods must tolerate tenant-specific taints.
- **Resource Constraints:** Ensures that CPU and memory usage of containers comply with tenant-specific annotations (`maxCpu`, `maxMemory`, `minCpu`, `minMemory`).

### 2. **Generate Network Policies**
This rule automatically generates a `NetworkPolicy` for each `RayCluster` to enforce network security:
- **Traffic Isolation:** Only pods within the same tenant can communicate with each other.
- **Port-Level Restrictions:** Outbound traffic is limited to specific ports required by Ray:
  - `6379` (Redis)
  - `8265` (Ray Dashboard)
  - `10001` (Workers)
- **Dynamic Policy Generation:** Network policies are dynamically created based on the `RayCluster` configuration.



