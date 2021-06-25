| **Cluster Architecture, Installation and Configuration   25%**         |   |
|------------------------------------------------------------------------|---|
| 1.  Manage role based access control (RBAC)                            |   |
| 2.  Use Kubeadm to install a basic cluster                             |   |
| 3.  Manage a highly-available Kubernetes cluster                       |   |
| 4.  Provision underlying infrastructure to deploy a Kubernetes cluster |   |
| 5.  Perform a version upgrade on a Kubernetes cluster using Kubeadm    |   |
| 6.  Implement etcd backup and restore                                  |   |

Each section:
Background snippet: 
Declarative examples: yaml
Imperative: kubectl
Links: to more detail

## 1.  Manage role based access control (RBAC)                           
- To perform any action in a cluster, you need to access the API.
- Three steps are performed when accessing the API: authentication,
  authorisation (ABAC, RBAC, Webhook, Global deny/allow settings), and Admission Control. 
- RBAC falls under authorisation.  It uses the rbac.authorization.k8s.io API
  group to manage authorisation decisions.  With the resources in this group, you can define
  roles and associate users to these roles.

- Summary of RBAC process:
    - determine or create namespace
    - create certificate credentials for user
    - set the credentials for the user to the namespace using a context
    - create a role for the expected task set
    - bind the user to the role
    - verify the user has limited access

- There are four resources in this API group:
    - Role
    - ClusterRole
    - RoleBinding
    - ClusterRoleBinding

To manage RBAC, you need:
- Rules: this is an operation (verb) that can be carried out on a resource.
- Role & ClusterRole: represent a set of permissions (rules).  They are additive
  (i.e. there are no "deny" rules).  A role sets permissions within a particular
  namespace.  ClusterRole by contrast is non-namespaced.
- Subjects: Is the entity that attempts an operation, of which there are 3:
    - user accounts: (human or otherwise) - external to the cluster.  These are
      not API objects.
    - service accounts:
    - groups:
- RoleBindings and ClusterRoleBindings: these bind subjects to roles. Difference
  here is similiar to Role versus ClusterRole.  RoleBindings applies to a
  namespace, whereas ClusterRoleBinding applies to all namespaces.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
    name: pod-reader
    rules:
    - apiGroups: [""] # "" indicates the core API group
      resources: ["pods"]
        verbs: ["get", "watch", "list"]
```

```
kubectl create -f role-pod-reader.yaml
```

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: pod-reader-binding
  namespace: default
subjects:
- kind: User
  name: employee
  apiGroup: ""
roleRef:
  kind: Role
  name: pod-reader 
  apiGroup: ""
```

```
kubectl create -f rolebinding-pod-reader.yaml
```


## kubectl

```
kubectl create role pod-reader --verb=get --verb=list --verb=watch
--resource=pods # creates a role named "pod-reader" that allows a user to
perform get, watch and list on pods.
```

```
kubectl create role foo --verb=get,list,watch --resource=replicasets.apps #
create a role named "foo" with apiGroups specified.
```

- [official - kubernetes.io](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [other - bitnami tutorials](https://docs.bitnami.com/tutorials/configure-rbac-in-your-kubernetes-cluster/)


## 2.  Use Kubeadm to install a basic cluster                             
### background
- The exam list six clusters that are in existence ???
- Maybe the case that you have to add a node to the ik8s cluster.
- kubeadm is a tool that you can use to set up a K8s cluster from scratch.

```
sudo apt update && sudo apt upgrade -y # to update and upgrade packages
sudo install kubelet, kubeadm, kubectl # to install 
sudo kubeadm init # to initialise cluster on control plane. This may take
several minutes.  When it finishes you should see "Your Kubernetes control-plane
has initialized successfully!"
# To start using your cluster, you need to make the following config changes:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config # copy the default
config to your home directory 
sudo chown $(id -u):$(id -g) $HOME/.kube/config
Set up Pod Networking
You will need to set up a Container Network Interface (CNI). 
Know one for the exam.
Weave seems to be recommended.
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl
version | base64 | tr -d '\n')"
kubectl get po -n kube-system # inspect pods to see if weavenet pods running


... come back to thi ... 
```

## 3.  Manage a highly-available Kubernetes cluster                       


## 4.  Provision underlying infrastructure to deploy a Kubernetes cluster 



## 5.  Perform a version upgrade on a Kubernetes cluster using Kubeadm    


## 6.  Implement etcd backup and restore                                  
- etcd is a key-value store used as Kubernetes' backing store for all cluster
  data (i.e. cluster state data and cluster config data).
- all K8s objects are stored on etcd.
- periodically backing up the etcd cluster data is important to recover
  Kubernetes clusters in a DR event.
- etcdctl: is a command line client for etcd.
- official documentation advises to stop kube-apiservers before restore to
  ensure there is no reliance on stale data. As regards the exam ...?
- backup can be accomplished in two ways: etcd built-in snapshot or volume
  snapshot.

kubectl -n kube-system get pods

Built-in snapshot:

```
Step 1: take the snapshot using etcdctl. With etcdctl, you can specify a number
of options, such as endpoint, certificates, etc. Cert/keyfile info can be
obtained by describing the etcd pod or by checking
/etc/kubernetes/manifests/etcd.yaml

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
    snapshot save <backup-file-location>

Step 2: verify the snapshot taken
sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db # you
should see output similar to:

+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| fe01cf57 |       10 |          7 | 2.1 MB     |
+----------+----------+------------+------------+

Step 3: restore, etcd supports restoring from snapshots that are taken from an
etcd process of the major.minor version.

ETCDCTL_API=3 etcdctl snapshot restore snapshot.db




