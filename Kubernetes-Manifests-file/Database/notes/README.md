# Kubernetes Storage Setup Explanation

This document explains the storage configuration for the database pods in this project.

## Summary of the Storage Setup

The current setup uses **Static Provisioning** with **`hostPath`** storage. It is **not** using dynamic provisioning.

### 1. PersistentVolume (PV) - The Actual Storage
The `pv.yaml` file defines the actual storage in the cluster. In this case, it uses a `hostPath` which points to a folder on the worker node's local disk:
- **Path:** `/data/db`
- **Capacity:** `1Gi`
- **Access Mode:** `ReadWriteOnce` (Only one pod can write to this at a time)

### 2. PersistentVolumeClaim (PVC) - The Storage Request
The `pvc.yaml` file is used by the database pod to request storage. Since it has `storageClassName: ""` (an empty string), it tells Kubernetes to find a manually created PV instead of automatically creating a new one (Dynamic Provisioning).

### 3. Database Deployment - Connecting to Storage
The MongoDB `deployment.yaml` refers to the `mongo-volume-claim` (the PVC) in its `volumes` section. This mounts the storage provided by the PV into the pod at the `/data/db` location inside the container.

---

### Important Distinction for Beginners: Static vs. Dynamic Provisioning

| Feature | Static Provisioning (Current Setup) | Dynamic Provisioning (StorageClass) |
| :--- | :--- | :--- |
| **Who creates the PV?** | The Cluster Administrator (Manually) | The `StorageClass` (Automatically) |
| **Files Needed** | `pv.yaml`, `pvc.yaml` | Only `pvc.yaml` |
| **Typical Use Case** | Local development, single-node clusters | Cloud environments (AWS EBS, GCP Disk), production |
| **Reliability** | Low (Data is tied to the node's local disk) | High (Disks can shift between nodes if one fails) |

---
**Note:** If you move this project to a multi-node EKS cluster, you would likely replace the `pv.yaml` and `hostPath` with an AWS-backed `StorageClass` for persistent data that survives node failures.
