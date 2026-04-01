# AWS Infrastructure Setup for Dynamic EBS Storage

Before running the dynamic `StorageClass` setup provided in this folder, you must ensure that your AWS EKS cluster has the following infrastructure components configured.

## 1. Amazon EBS CSI Driver (EKS Add-on)
To allow Kubernetes to automatically create and manage AWS EBS volumes, you need to install the **Amazon EBS CSI Driver**.

### Installation Methods:
- **AWS Console:** Go to your EKS Cluster -> **Add-ons** tab -> **Get more add-ons** -> Search for **Amazon EBS CSI Driver**.
- **AWS CLI:**
    ```bash
    aws eks create-addon --cluster-name <your-cluster-name> --addon-name aws-ebs-csi-driver
    ```

## 2. IAM Role for the CSI Driver (Permissions)
The driver needs permission to talk to the AWS API to create/attach EBS volumes.

### Required Steps:
1.  **Enable IAM OIDC Provider:** Your EKS cluster must have an IAM OIDC provider enabled.
2.  **Create IAM Role:** Create an IAM role specifically for the EBS CSI Driver.
3.  **Attach Policy:** Attach the AWS-managed policy named `AmazonEBSCSIDriverPolicy` to that role.
4.  **Trust Relationship:** Ensure the IAM role's trust policy allows the `ebs-csi-controller-sa` ServiceAccount (in the `kube-system` namespace) to assume the role.

## 3. Storage Compatibility Checklist
Before applying the manifests (`storageclass.yaml`, `pvc.yaml`):
- [ ] Your EKS cluster version should be **1.23+** (since the CSI driver is mandatory for later versions).
- [ ] The **EBS CSI Driver PODS** should be in a `Running` state:
    ```bash
    kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-ebs-csi-driver
    ```

## Summary of the Workflow:
1.  Set up AWS Infrastructure (IAM and EBS CSI Driver).
2.  Apply `storageclass.yaml`.
3.  Apply `pvc.yaml` (Check status with `kubectl get pvc -n three-tier`).
4.  Apply `deployment.yaml`.

---
**Note:** Without the **EBS CSI Driver**, your PersistentVolumeClaims (PVCs) will remain in a `Pending` state.
