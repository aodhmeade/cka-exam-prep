| **Storage  10%**                                                          |
|---------------------------------------------------------------------------|
| 1.  Understand storage classes, persistent volumes                        |
| 2.  Understand volume mode, access modes and reclaim policies for volumes |
| 3.  Understand persistent volume claims primitive                         |
| 4.  Know how to configure applications with persistent storage            |

- Container engines have traditionally not offered storage that outlives the
  container.  Containers in essence are considered transient.  Therefore you
  need a place to persistently store information.
- Kubernetes provides several methods for handling storage: ephemeral options
  and persistent options, based on Volumes as the central abstraction.
    - ephemeral volumes: data exists only as longs as the pod lasts (e.g. stdout, stderr
      logs, cache data)
    - persistent volumes: data that outlives the pod.
- Volumes are the basic entity containers use to access storage in Kubernetes.
  The storage types they support range from local, to NFS, to cloud storage
  services.
- Initially Kubernetes used a volume plugin system.  This was 'in-tree', meaning
  it shipped with the core Kubernetes binaries and made adding support for new
  storage systems challenging.  The Container Storage Interface (CSI) is a
  standard that Kubernetes now uses to expose block and file storage to
  containerised workloads on container orchestration systems like Kubernetes.
- Two important volumes types to note are 'emptyDir' and 'hostPath' volume
  types.  The storage provider in each case is the Localhost and each has its own use case.

- emptyDir: is first created when a pod is assigned to a node, and exists as
  long as that pod is running on that node.  It is initially empty and all
  containers in the same pod, can r/w in the same emptyDir volume.  When a pod
  is deleted or restarted the data is lost.  Depending on your environment,
  emptyDir vols are stored on whatever medium that backs the node, such as disk,
  or network storage.  You can also set emptyDir to run off RAM (Kubernetes
  mounts a tmpfs in this instance).

      ```
      apiVersion: v1
      kind: Pod
      metadata:
        name: test-pd
      spec:
        containers:
        - image: k8s.gcr.io/test-webserver
          name: test-container
          volumeMounts:
          - mountPath: /cache
            name: cache-volume
        volumes:
        - name: cache-volume
        emptyDir: {}
      ```
    - verify by opening an interactive terminal, locating and listing the
      directory

      ```
      kubectl exec -it test-pd bash
      
      root@test-pd:/# ls

      root@test-pd:/# ls cache/
      root@test-pd:/# 
      ```


- hostPath: volume type mounts a file or directory from the host node's
  filesystem into your pod.  Though generally not recommended, they can be used
  for some use cases.  When configuring, note that the 'path' property is
  required.  You also can specify an optional 'type'.  Some supported types are:
  DirectoryOrCreate, Directory, FileOrCreate, File, Socket, CharDevice,
  BlockDevice.

    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: test-pd
    spec:
      containers:
      - image: k8s.gcr.io/test-webserver
        name: test-container
        volumeMounts:
        - mountPath: /test-pd
          name: test-volume
      volumes:
      - name: test-volume
        hostPath:
          # directory location on host
          path: /data
          # this field is optional
          type: Directory
   ``` 

# **1.  Understand storage classes, persistent volumes**

- [https://kubernetes.io/docs/concepts/storage/persistent-volumes/]
- [https://kubernetes.io/docs/concepts/storage/storage-classes/]

## persistent volumes
- a 'PersistentVolume' (pv) is a storage abstraction used to retain data longer
  then the Pod using it.  The PersistentVolume API provides an API for users &
  administrators that abstract details of how storage is provided from how it is
  consumed. It is a resource in the cluster just like a node is a cluster
  resource.
    - a 'PersistentVolumeClaim' (pvc) is a request for storage by a user.  It can
      be thought of in a similar vain to a Pod. Pods consume node resources and
      pvc's consume pv resources.
    - use a pvc in a pod configuration.
    - the pod will request the claim --> the pvc will try find the volume --> the
      volume contains the actual storage.
    - persistent volumes are not namespaced, they are available to the entire
      cluster.
    - pvc's are namespaced & must exist in the same namespace of the pod using them.
    - volumes are mounted into the container.
    - ConfigMap and Secret are considered local volumes but are not created via
      pv and pvc.

```
kubectl get pv
kubectl get pvc
```

## storage classes
- Kubernetes can dynamically provision volumes for a pvc.  To enable this - it
  needs to know what type of storage is required.  It uses the concept of a
  StorageClass to complete this.
- Example StorageClass:
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type:
```
- Note: the provisioner and parameter fields.
- 'provisioner' will call the AWS api to create the Elastic Block Store volume
- 'parameters' allow you to define the options that the provisioner will use to
  create the volume.  The options you provide will depend on the provisioner
  (GCP, Azure, etc).
- A StorageClass object will do nothing unless you specifically call it.



# **2. Understand volume mode, access modes and reclaim policies for volumes**

- [https://kubernetes.io/docs/concepts/storage/persistent-volumes/]

### volumeMode
- Kubernetes supports two volumeModes of PersistentVolumes: Filesystem and
  Block.
- volumeMode is an optional API parameter. Filesystem is the default mode used
  when volumeMode parameter is omitted.


### Access Modes

The access modes are:

    ReadWriteOnce -- the volume can be mounted as read-write by a single node
    ReadOnlyMany -- the volume can be mounted read-only by many nodes
    ReadWriteMany -- the volume can be mounted as read-write by many nodes

In the CLI, the access modes are abbreviated to:

    RWO - ReadWriteOnce
    ROX - ReadOnlyMany
    RWX - ReadWriteMany

### Reclaim Policies
When a user is done with their volume, they can delete the PVC objects from the
API that allows reclamation of the resource.  The reclaim policy for a
PersistentVolume tells the cluster what to do with the volume after it has been
released of its claim.  Currently, volumes can either be Retained, Recycled, or
Deleted.

    Retain -- manual reclamation 
    Recycle -- basic scrub (rm -rf /thevolume/*)
    Delete -- associated storage asset such as AWS EBS, GCE PD, Azure Disk, or
    OpenStack Cinder volume is deleted

Currently, only NFS and HostPath support recycling. AWS EBS, GCE PD, Azure Disk,
and Cinder volumes support deletion.


# **3.  Understand persistent volume claims primitive**

As per no.1




# **4.  Know how to configure applications with persistent storage**









