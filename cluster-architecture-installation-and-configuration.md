| **Cluster Architecture, Installation and Configuration   25%**         |   |
|------------------------------------------------------------------------|---|
| 1.  Manage role based access control (RBAC)                            |   |
| 2.  Use Kubeadm to install a basic cluster                             |   |
| 3.  Manage a highly-available Kubernetes cluster                       |   |
| 4.  Provision underlying infrastructure to deploy a Kubernetes cluster |   |
| 5.  Perform a version upgrade on a Kubernetes cluster using Kubeadm    |   |
| 6.  Implement etcd backup and restore                                  |   |

## **1. Manage role based access control (RBAC)**               

- [official - kubernetes.io](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [other - bitnami tutorials](https://docs.bitnami.com/tutorials/configure-rbac-in-your-kubernetes-cluster/)

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


# **2. Use Kubeadm to install a basic cluster**

[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/]
[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/]
[https://kubernetes.io/docs/setup/production-environment/container-runtimes/]

- kubeadm is a cli tool that you can use to set up a K8s cluster from scratch.
- kubeadm is not the only tool. There are other vendor specific tools. 
- Bootstrapping a Kubernetes cluster with kubeadm essentially involves 3 steps: 
    - run 'kubeadm init' (to initialise the cp/head node)
    - apply a network plugin (a CNI plugin),
    - run 'kubeadm join' (on a worker node).
- There are other commands such as kubeadm upgrade, kubeadm config, kubeadm
  token, kubeadm reset.
- Before you begin, check to ensure you have a container runtime installed on
  your nodes.
- Once your cluster is up and running, you would use the 'kubectl' command to
  interact with your cluster. kubectl uses $HOME/.kube/config as a configuration
  file.  This contains all the Kubernetes endpoints that you may end up using,
  as well as other items such as credentials, contexts, definitions, etc.
- Note: A context is a combination of a cluster and user credentials.  You can
  switch contexts with kubectl.  This is useful when switching between clusters,
  e.g. going from local to the cloud.
```
$ kubectl config use-context name-of-new-context
```

## install in two steps: 1 - install a control plane, 2 - grow the cluster by
adding a worker node.

- Note: steps performed using two GCP virtual machines running Ubuntu 18.04.

### Step 1 - install a control plane node
```
1. Switch to root
# sudo -i

1.1 Disable swap on all nodes (note why exactly ...). Optional: update the file
system table file to ensure it is off on all reboots.
```
# swapoff -a
# sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

2 Start by updating and upgrading software packages
# sudo apt update && sudo apt upgrade -y 

3 Next, install a container runtime engine, example - docker.
# sudo apt install -y docker.io

3.1 Configure Cgroup drivers
[https://kubernetes.io/docs/setup/production-environment/container-runtimes/]

```
cat << EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

```
3.2 Use systemd to restart docker and enable on boot
```
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

4 Add a new repo for Kubernetes. Create and edit as follows:
```
# vim /etc/apt/sources.list.d/kubernetes.list
'deb    http://apt.kubernetes.io/   kubernetes-xenial   main'
```

5. Add a GPG key for the packages. 
```
# curl -s \
# https://packages.cloud.google.com/apt/doc/apt-key.gpg \
# | apt-key add -
```
6. Update the new repo
```
# apt-get update
```

7. Install the necessary software (kubeadm kubelet kubectl).  
- Note: tried installing all three
- the Kubelet version may never exceed the API server version.  For example, the
  kubelet running v.1.20.0 should be fully compatible with a 1.21.0 API server,
  but not vica versa.
```
# apt-get install kubeadm 
```

8. Mark the relevant packages so they are not upgraded:
```
# apt-mark hold kubeadm kubelet kubectl
```

9. Get cp IP address and add to /etc/hosts
```
hostname -i
vim /etc/hosts
10.128.0.3 k8scp #<-- add this line (change ip to match output from hostname -i)
```

10. To start using your cluster, you need to make the following config changes:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config # copy the default
config to your home directory 
```


11. At this point we could copy and paste the join command from the cp node. That
command only works for 2 hours, so we will build our own join should we want to
add nodes in the future. Find the token on the cp node. The token lasts 2 hours
by default. If it has been longer, and no token is present you can generate a
new one with the sudo kubeadm token create command, seen in the following
command. On the cp node:
```
# kubeadm token list
# kubeadm token create --print-join-command
```

11. Use the above output on the worker node:

```
kubeadm join k8scp:6443 --token qsqg38.g9pohmf6pjdw0hkw \
--discovery-token-ca-cert-hash \
sha256:246ac43b7feb35d39340e1930c263e910f6f95f53be8b983a5b011a19c9e100c
```

12. To verify if node has joined, run this on the cp:
```
$ kubectl get nodes
```


13. To start using your cluster, you need to make the following config changes:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config # copy the default
config to your home directory 
```

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

## **3.  Manage a highly-available Kubernetes cluster**                       




## **4. Provision underlying infrastructure to deploy a Kubernetes cluster**
- cloud, multi-cloud, on-premises, hybrid, combinaton thereof.
- possibly not tested in the exam ... review





## **5.  Perform a version upgrade on a Kubernetes cluster using Kubeadm**
- if you build your cluster with kubeadm, you also have the option to upgrade
  the cluster using the kubeadm upgrade command.
- skipping minor versions when upgrading is unsupported.
- the upgrade workflow at a high level is as follows:
    1. upgrade a primary control plane node.
    2. upgrade additional control plane nodes.
    3. upgrade worker nodes.
- Note: real world scenario requires certain precautions to be taken into
  consideration, such as disabling swap in linux.  See the official kubernetes.io
  guidance for more detailed information.
- determine which version to upgrade to.  Find the latest stable kubernetes
  version using OS package manager ('apt' as the exam is based on Ubuntu):
- useful to mark the versions of kubeadm, kubectl and kubelet before you begin.
```
sudo apt-mark hold kubeadm kubelet kubectl
```

```
apt-update
apt-cache kubeadm
# find the latest 1.21 version in the list
# it should look like 1.21.x-00, where x is the latest patch.
```
- upgrading control plane nodes should be done one node at a time
- the control plane must have the /etc/kubernetes/admin.conf file.

- update kubeadm
```
# replace x in the 1.21.x-00 with latest patch version
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubernetes=1.21.x-00 && \
apt-mark hold kubeadm
```
- verify that the download works and has the expected version
```
kubeadm version
# might be good practice to mark this newer version again with sudo apt-mark
# hold kubeadm
```
- drain first node and execute upgrade
```
kubectl drain <node> --ignore-daemonsets
```




- verify the upgrade plan. This command checks that your cluster can be upgrade,
  and fetches versions you can upgrade to.
```
kubeadm upgrade plan
```
- Note: kubeadm upgrade also automatically renews the certificates that it
  manages on the node.
- Note: if the kubeadm upgrade plan output shows any component configs that
  require manual upgrade, you must provide a config file with replacement
  configs to kubeam upgrade apply via the --config cli flag.
- Next, choose a version to upgrade to:
```
sudo kubeadm upgrade apply v1.21.x
-
# once the command finishes you should see:
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.21.x". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with
upgrading your kubelets if you haven't already done so.
```
- upgrade kubelet
```
sudo apt-get install --only-upgrade kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```





## 6.  Implement etcd backup and restore                                  
- etcd is an open source distributed key-value store used as Kubernetes' backing store for all cluster
  data (i.e. cluster state data, metadata, cluster config data).
- containerised workloads (distributed) have complex management requirements as
  they grow.  Kubernetes simplifies the process of managing these workloads by
  co-ordinating tasks which run on mulitple machines in multiple locations.  It
  achieves this co-ordination by using a data store that provides a single
  source of truth about the status of the system at any given point in time.  etcd is this data store.
- etcd is built on the Raft consensus algorithm to ensure data consistency.
- Kubernetes uses etcd's "watch" function to monitor cluster state data, compare
  against ideal state and to reconfigure itself when changes occur.  
- the 'etc' in etcd references the UNIX/Linux config directory /etc. The 'd' in
  etcd stands for distributed.
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




