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

# Example RayCluster Configurations

Below are example configurations illustrating different scenarios when using the Kyverno policies for `RayCluster`.



## 1. **Configuration That Passes Both Policies**

This configuration meets all the requirements of both the `enforce-tenant-isolation` and `generate-network-policies` rules.

```yaml
apiVersion: ray.io/v1
kind: RayCluster
metadata:
  name: ray-tenant-team-a
  namespace: ray-namespace
  labels:
    tenant: "team-a"
  annotations:
    maxCpu: "4"
    maxMemory: "8Gi"
    minCpu: "1"
    minMemory: "2Gi"
spec:
  rayVersion: "2.9.0"
  headGroupSpec:
    rayStartParams:
      dashboard-host: "0.0.0.0"
      block: "true"
    template:
      metadata:
        labels:
          tenant: "team-a"
      spec:
        nodeSelector:
          tenant: "team-a"
        tolerations:
        - key: "tenant"
          operator: "Equal"
          value: "team-a"
        containers:
        - name: ray-head
          image: rayproject/ray:2.9.0
          ports:
          - containerPort: 6379
          - containerPort: 8265
          - containerPort: 10001
          resources:
            limits:
              cpu: "4"
              memory: "8Gi"
            requests:
              cpu: "1"
              memory: "2Gi"
  workerGroupSpecs:
  - groupName: small-workers
    replicas: 2
    rayStartParams:
      block: "true"
    template:
      metadata:
        labels:
          tenant: "team-a"
      spec:
        nodeSelector:
          tenant: "team-a"
        tolerations:
        - key: "tenant"
          operator: "Equal"
          value: "team-a"
        containers:
        - name: ray-worker
          image: rayproject/ray:2.9.0
          resources:
            limits:
              cpu: "4"
              memory: "8Gi"
            requests:
              cpu: "1"
              memory: "2Gi"
```
## 2. **Configuration That Fails enforce-tenant-isolation**

```yaml
apiVersion: ray.io/v1
kind: RayCluster
metadata:
  name: ray-test-invalid
  namespace: ray-namespace
  labels:
    tenant: "team-a"
  annotations:
    maxCpu: "4"
    maxMemory: "8Gi"
    minCpu: "1"
    minMemory: "2Gi"
spec:
  rayVersion: "2.9.0"
  headGroupSpec:
    rayStartParams:
      dashboard-host: "0.0.0.0"
      block: "true"
    template:
      spec:
        containers:
        - name: ray-head
          image: rayproject/ray:2.9.0
  workerGroupSpecs:
  - groupName: small-workers
    replicas: 2
    rayStartParams:
      block: "true"
    template:
      spec:
        containers:
        - name: ray-worker
          image: rayproject/ray:2.9.0
```
## 3. **Configuration That Fails generate-network-policies**
```yaml
apiVersion: ray.io/v1
kind: RayCluster
metadata:
  name: ray-test-no-tenant
  namespace: ray-namespace
  annotations:
    maxCpu: "4"
    maxMemory: "8Gi"
    minCpu: "1"
    minMemory: "2Gi"
spec:
  rayVersion: "2.9.0"
  headGroupSpec:
    rayStartParams:
      dashboard-host: "0.0.0.0"
      block: "true"
    template:
      metadata:
        labels:
          app: "ray-head"  # Missing tenant label
      spec:
        nodeSelector:
          app: "ray-node"
        tolerations:
        - key: "app"
          operator: "Equal"
          value: "ray-node"
        containers:
        - name: ray-head
          image: rayproject/ray:2.9.0
  workerGroupSpecs:
  - groupName: small-workers
    replicas: 2
    rayStartParams:
      block: "true"
    template:
      metadata:
        labels:
          app: "ray-worker"  # Missing tenant label
      spec:
        nodeSelector:
          app: "ray-node"
        tolerations:
        - key: "app"
          operator: "Equal"
          value: "ray-node"
        containers:
        - name: ray-worker
          image: rayproject/ray:2.9.0
```
## 4. **Configuration That Fails Both Policies**
```yaml
apiVersion: ray.io/v1
kind: RayCluster
metadata:
  name: ray-test-fail-both
  namespace: ray-namespace
spec:
  rayVersion: "2.9.0"
  headGroupSpec:
    rayStartParams:
      dashboard-host: "0.0.0.0"
      block: "true"
    template:
      spec:
        containers:
        - name: ray-head
          image: rayproject/ray:2.9.0
  workerGroupSpecs:
  - groupName: small-workers
    replicas: 2
    rayStartParams:
      block: "true"
    template:
      spec:
        containers:
        - name: ray-worker
          image: rayproject/ray:2.9.0
```
