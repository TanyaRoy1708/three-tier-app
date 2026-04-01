# Dynamic Storage Setup (AWS EKS)

This folder contains a **Dynamic Provisioning** setup. This is the standard way to handle storage in a production AWS EKS cluster.

## Files included:

1. **`storageclass.yaml`**: Defines a new StorageClass named `ebs-sc` using the Amazon EBS CSI driver.
   - `provisioner: ebs.csi.aws.com`: Tells AWS to automatically create an EBS volume.
   - `volumeBindingMode: WaitForFirstConsumer`: Ensures the volume is created in the same Availability Zone where your pod is scheduled.

2. **`pvc.yaml`**: This PersistentVolumeClaim (PVC) refers to the `ebs-sc` storage class. 
   - Note the `storageClassName: ebs-sc`.
   - Kubernetes will automatically create a `PersistentVolume` (PV) for this claim.

3. **`deployment.yaml`**: A database deployment configured to use the dynamic PVC.

## Key Benefits
- **Automatic:** No need to manually create `pv.yaml` files or manage node paths.
- **Node-Independent:** EBS volumes can move between nodes, so your database stays available even if a node fails.
- **Standard Practice:** This is how production clusters handle stateful data.
