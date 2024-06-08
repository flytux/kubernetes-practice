# NFS volume snapshot with CSI driver

- Rocky 8, k3s v1.27.6
- Install NFS server
- Install k3s
- Install CSI crd
- Install external CSI snapshotter
- Install nfs-csi driver
- Install storageclass, snapshotclass
- Test dynamic pv provisioning
- Test volumesnapshot create and recovery

- Velero CSI setup

---

```bash

# Install NFS server
$ dnf install nfs-utils -y
$ mkdir -p /var/nfs/general
$ chown -R 65534:65534 /var/nfs/general

$ cat <<EOF > /etc/exports
/var/nfs/general    10.10.10.11(rw,sync,no_subtree_check)
EOF

$ systemctl enable nfs-server
$ systemctl start nfs-server
$ systemctl status nfs-server

$ mkdir -p /nfs/general
$ mount /var/nfs/general /nfs/general
$ df -h | grep nfs

# Install NFS CSI driver
$ curl -skSL https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/v4.5.0/deploy/install-driver.sh | bash -s v4.5.0 --

$ kubectl -n kube-system get pod -o wide -l app=csi-nfs-controller
$ kubectl -n kube-system get pod -o wide -l app=csi-nfs-node

# Create storage class
$ cat <<EOF > nfs-sc.yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi
provisioner: nfs.csi.k8s.io
parameters:
  server: 10.10.10.11
  share: /var/nfs/general
  mountPermissions: "0777"
  # csi.storage.k8s.io/provisioner-secret is only needed for providing mountOptions in DeleteVolume
  # csi.storage.k8s.io/provisioner-secret-name: "mount-options"
  # csi.storage.k8s.io/provisioner-secret-namespace: "default"
reclaimPolicy: Delete
volumeBindingMode: Immediate
mountOptions:
  - nfsvers=4.1
EOF

$ kubectl apply -f nfs-sc.yml

# Test PVC
$ kubectl create -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/example/pvc-nfs-csi-dynamic.yaml
$ kubectl get pvc

# Install CSI CRD

$ git clone https://github.com/kubernetes-csi/external-snapshotter.git
$ kubectl apply -k external-snapshotter/client/config/crd/

# Create nginx with volume
cat <<EOF > nfs-nginx.yml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-deployment-nfs
spec:
  accessModes:
    - ReadWriteMany  # In this example, multiple Pods consume the same PVC.
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs-csi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-nfs
spec:
  replicas: 1
  selector:
    matchLabels:
      name: deployment-nfs
  template:
    metadata:
      name: deployment-nfs
      labels:
        name: deployment-nfs
    spec:
      nodeSelector:
        "kubernetes.io/os": linux
      containers:
        - name: deployment-nfs
          image: mcr.microsoft.com/oss/nginx/nginx:1.19.5
          command:
            - "/bin/bash"
            - "-c"
            - set -euo pipefail; while true; do echo $(hostname) $(date) >> /mnt/nfs/outfile; sleep 1; done
          volumeMounts:
            - name: nfs
              mountPath: "/mnt/nfs"
              readOnly: false
      volumes:
        - name: nfs
          persistentVolumeClaim:
            claimName: pvc-deployment-nfs
EOF

# Create snapshot
cat <<EOF > volumesnapshot.yml
---
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: test-nfs-snapshot
spec:
  volumeSnapshotClassName: csi-nfs-snapclass
  source:
    persistentVolumeClaimName: pvc-nfs-dynamic
EOF

cat <<EOF > pvc-restored.yml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs-snapshot-restored
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs-csi
  dataSource:
    name: test-nfs-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
EOF

# Create nginx-restored
cat <<EOF > nginx-restored.yml
---
kind: Pod
apiVersion: v1
metadata:
  name: nginx-nfs-restored-snapshot
spec:
  nodeSelector:
    "kubernetes.io/os": linux
  containers:
    - image: mcr.microsoft.com/oss/nginx/nginx:1.19.5
      name: nginx-nfs
      command:
        - "/bin/bash"
        - "-c"
        - set -euo pipefail; while true; do echo $(date) >> /mnt/nfs/outfile; sleep 1; done
      volumeMounts:
        - name: persistent-storage
          mountPath: "/mnt/nfs"
          readOnly: false
  volumes:
    - name: persistent-storage
      persistentVolumeClaim:
        claimName: pvc-nfs-snapshot-restored
EOF

```
