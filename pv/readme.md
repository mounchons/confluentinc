Yes, you can create a PersistentVolume in Kubernetes that uses storage from your own VPS, including a Linux EC2 instance on AWS. One common way to do this is by using NFS (Network File System). Below are the steps to set this up and a sample manifest code for creating a PersistentVolume and a PersistentVolumeClaim in Kubernetes.

### Steps to Create PersistentVolume using NFS on Your VPS

1. **Set up NFS Server on Your Linux EC2 Instance:**

   - Install NFS server:
     ```sh
     sudo apt update
     sudo apt install nfs-kernel-server
     ```

   - Create a directory to share:
     ```sh
     sudo mkdir -p /mnt/nfs_share
     sudo chown nobody:nogroup /mnt/nfs_share
     sudo chmod 777 /mnt/nfs_share
     ```

   - Add the directory to the NFS exports file:
     ```sh
     echo "/mnt/nfs_share *(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports
     ```

   - Apply the changes:
     ```sh
     sudo exportfs -a
     sudo systemctl restart nfs-kernel-server
     ```

   - Allow NFS traffic through the firewall (if applicable):
     ```sh
     sudo ufw allow from any to any port nfs
     ```
2. **Create a StorageClass:** 

   - Create a file named `00-nfs-storage-class.yaml` with the following content:
     ```yaml
     apiVersion: storage.k8s.io/v1
     kind: StorageClass
     metadata:
       name: nfs
     provisioner: kubernetes.io/no-provisioner
     volumeBindingMode: Immediate

3. **Create a PersistentVolume Manifest:**

   - Create a file named `01-nfs-pv.yaml` with the following content:
     ```yaml
     apiVersion: v1
     kind: PersistentVolume
     metadata:
       name: nfs-pv
     spec:
       capacity:
         storage: 1Gi
       accessModes:
         - ReadWriteMany
       nfs:
         path: /mnt/nfs_share
         server: <your-nfs-server-ip>
       persistentVolumeReclaimPolicy: Retain
       storageClassName: nfs
     ```

4. **Create a PersistentVolumeClaim Manifest:**

   - Create a file named `02-nfs-pvc.yaml` with the following content:
     ```yaml
     apiVersion: v1
     kind: PersistentVolumeClaim
     metadata:
       name: nfs-pvc
     spec:
       accessModes:
         - ReadWriteMany
       resources:
         requests:
           storage: 1Gi
       storageClassName: nfs
     ```

5. **Apply the Manifests to Your Kubernetes Cluster:**

   - Apply the PersistentVolume:
     ```sh
     kubectl apply -f 00-nfs-storage-class.yaml
     ```

   - Apply the PersistentVolume:
     ```sh
     kubectl apply -f 01-nfs-pv.yaml
     ```

   - Apply the PersistentVolumeClaim:
     ```sh
     kubectl apply -f 02-nfs-pvc.yaml --namespace dev
     ```

### Using the PersistentVolumeClaim in a Pod

To use the `PersistentVolumeClaim` in a pod, create a manifest for your pod like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-test-pod
spec:
  containers:
    - name: test-container
      image: busybox
      command: ["/bin/sh", "-c", "sleep 3600"]
      volumeMounts:
        - mountPath: "/mnt/nfs"
          name: nfs-volume
  volumes:
    - name: nfs-volume
      persistentVolumeClaim:
        claimName: nfs-pvc
```

Apply this manifest:

```sh
kubectl apply -f nfs-test-pod.yaml
```

This will create a pod that mounts the NFS share at `/mnt/nfs` inside the container. You can verify the setup by accessing the pod and checking the contents of the mounted directory.

By following these steps, you can create a PersistentVolume in Kubernetes using storage from your own VPS. Adjust the IP address, paths, and other configurations as needed for your specific environment.