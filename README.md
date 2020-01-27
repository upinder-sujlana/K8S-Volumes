# K8S-Volumes
```

Topology
========================
kmaster  - 192.168.1.80
knode1   - 192.168.1.81
knode2   - 192.168.1.82

The three form a K8S cluster:-
kmaster@kmaster:~$ kubectl get nodes
NAME      STATUS   ROLES    AGE    VERSION
kmaster   Ready    master   233d   v1.14.2
knode1    Ready    <none>   233d   v1.14.2
knode2    Ready    <none>   233d   v1.14.2
kmaster@kmaster:~$

The 3-nodes are OS details are the same:-
kmaster@kmaster:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04 LTS
Release:        18.04
Codename:       bionic
kmaster@kmaster:~$

Additionally, I have a 4th node outside the cluster, but in the same
LAN that I am using as a NFS Server :-
minikube - 192.168.1.85 (NFS Server running here)

On the NFS Server I have exposed the three directories to the above
cluster (permit all) directory names are gold, silver, bronze.
In the PVC I am just using gold as an example.

Created the Persistent Volume
==================================
kmaster@kmaster:~/dockerimagemaker/NFS$ kubectl create -f nfs-pv.yml
persistentvolume/nfs-pv created
kmaster@kmaster:~/dockerimagemaker/NFS$

kmaster@kmaster:~/dockerimagemaker/NFS$ kubectl get pv --show-labels -o wide
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE    LABELS
nfs-pv   5Gi        RWX            Retain           Available                                   100s   volume=nfs-pv-volume
kmaster@kmaster:~/dockerimagemaker/NFS$

kmaster@kmaster:~/dockerimagemaker/NFS$ kubectl describe pv nfs-pv
Name:            nfs-pv
Labels:          volume=nfs-pv-volume
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:
Status:          Available
Claim:
Reclaim Policy:  Retain
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        5Gi
Node Affinity:   <none>
Message:
Source:
    Type:      NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    192.168.1.85
    Path:      /home/minikube/NFSShare/gold
    ReadOnly:  false
Events:        <none>
kmaster@kmaster:~/dockerimagemaker/NFS$
  
 Created the Persistent Volume Claim
 ======================================
 kmaster@kmaster:~/dockerimagemaker/NFS$ kubectl create -f nfs-pvc.yml
persistentvolumeclaim/nfs-pvc created
kmaster@kmaster:~/dockerimagemaker/NFS$

kmaster@kmaster:~/dockerimagemaker/NFS$ kubectl get pvc --show-labels
NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE   LABELS
nfs-pvc   Bound    nfs-pv   5Gi        RWX                           32s   <none>
kmaster@kmaster:~/dockerimagemaker/NFS$

kmaster@kmaster:~/dockerimagemaker/NFS$ kubectl describe pvc nfs-pvc
Name:          nfs-pvc
Namespace:     default
StorageClass:
Status:        Bound
Volume:        nfs-pv
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      5Gi
Access Modes:  RWX
VolumeMode:    Filesystem
Events:        <none>
Mounted By:    <none>
kmaster@kmaster:~/dockerimagemaker/NFS$
  
  
To test the setup I created the below deployment and had the POD write to the gold directory. 

kmaster@kmaster:~$ kubectl create -f nfstester.yml
deployment.apps/nfstester created
kmaster@kmaster:~$

Verify
===================
Went to the NFS server directory and started the tail on the newly created file:
minikube@ubuntu:~/NFSShare/gold$ tail -f hellopod.txt
Wed Jan 15 20:50:56 UTC 2020
Wed Jan 15 20:51:06 UTC 2020
Wed Jan 15 20:51:16 UTC 2020
Wed Jan 15 20:51:26 UTC 2020


  
```
