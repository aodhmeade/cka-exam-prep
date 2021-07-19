| **Cluster Architecture, Installation and Configuration   25%**         |   |
|------------------------------------------------------------------------|---|
| 1.  Manage role based access control (RBAC)                            |   |
| 2.  Use Kubeadm to install a basic cluster                             |   |
| 3.  Manage a highly-available Kubernetes cluster                       |   |
| 4.  Provision underlying infrastructure to deploy a Kubernetes cluster |   |
| 5.  Perform a version upgrade on a Kubernetes cluster using Kubeadm    |   |
| 6.  Implement etcd backup and restore                                  |   |


# 1. Manage role based access control (RBAC)

- [https://kubernetes.io/docs/reference/access-authn-authz/rbac/]
- [https://docs.bitnami.com/tutorials/configure-rbac-in-your-kubernetes-cluster/]

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

- To manage RBAC, you need:
    - Rules: this is an operation (verb) that can be carried out on a resource.
    - Role & ClusterRole: represent a set of permissions (rules).  They are additive
      (i.e. there are no "deny" rules).  A role sets permissions within a particular
      namespace.  ClusterRole by contrast is non-namespaced.
    - Subjects: Is the entity that attempts an operation, of which there are 3:
        - user accounts: (human or otherwise) - external to the cluster.  These are
          not API objects.
        - service accounts:
        - groups:
    - RoleBindings and ClusterRoleBindings: these bind subjects to roles.
      Difference here is similiar to Role versus ClusterRole.  RoleBindings
      applies to a namespace, whereas ClusterRoleBinding applies to all
      namespaces.

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


# 2. Use Kubeadm to install a basic cluster

- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/]
- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/]
- [https://kubernetes.io/docs/setup/production-environment/container-runtimes/]

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

## Cluster installation end goal: one control plane node and one worker node. Steps performed using two virtual machines running Ubuntu 18.04.

### Step 1 - install a control plane node

1. Switch to root
```
sudo -i
```

2. Disable swap on all nodes (note why exactly ...). Optional: update the file
system table file to ensure it is off on all reboots.
```
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

3. Start by updating and upgrading software packages
```
sudo apt update && sudo apt upgrade -y 
```

4. Next, install a container runtime engine, example - docker.
```
 sudo apt install -y docker.io
```

5. Configure Cgroup drivers.  Both the container runtime and the kubelet have a
property called "cgroup driver", which is important with respect to the
management of cgroups on Linux. Note: matching the container runtime and the
kubelet cgroup drivers is required, otherwise the kubelet process will fail.

- [https://kubernetes.io/docs/setup/production-environment/container-runtimes/]
- [https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/]

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

6. Use systemd to restart docker and enable on boot
```
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

7. Add a new repo for Kubernetes. Create and edit as follows:
```
vim /etc/apt/sources.list.d/kubernetes.list
'deb    http://apt.kubernetes.io/   kubernetes-xenial   main'   #<-- add this
line to the file
```

8. Add a GPG key for the packages. 
```
curl -s \
https://packages.cloud.google.com/apt/doc/apt-key.gpg \
| apt-key add -
```

9. Update the new repo
```
# apt-get update
```

10. Install the necessary software (kubeadm kubelet kubectl).  
- Note: tried installing all three
- the Kubelet version may never exceed the API server version.  For example, the
  kubelet running v.1.20.0 should be fully compatible with a 1.21.0 API server,
  but not vica versa.
```
# apt-get install kubeadm 
```

11. Mark & hold the relevant packages so they are not upgraded:
```
# apt-mark hold kubeadm kubelet kubectl
```

12. Get cp IP address and add to /etc/hosts
```
hostname -i
vim /etc/hosts
10.128.0.3 k8scp #<-- add this line (change ip to match output from hostname -i)
```

13. To start using your cluster, you need to make the following config changes:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config # copy the default
config to your home directory 
```

### step 2 - grow the cluser - add worker node(s)

1. At this point we could copy and paste the join command from the cp node. That
command only works for 2 hours, so we will build our own join should we want to
add nodes in the future. Find the token on the cp node. The token lasts 2 hours
by default. If it has been longer, and no token is present you can generate a
new one with the sudo kubeadm token create command, seen in the following
command. On the cp node:

```
# kubeadm token list
# kubeadm token create --print-join-command
```

2. Use the above output on the worker node:

```
kubeadm join k8scp:6443 --token qsqg38.g9pohmf6pjdw0hkw \
--discovery-token-ca-cert-hash \
sha256:W43JLJ2L4HTOFUU4TL4RY9340WGSLKGF9732097448R028280220J
```

3. To verify if node has joined, run this on the cp:
```
$ kubectl get nodes
```

4. To start using your cluster, you need to make the following config changes:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config # copy the default
config to your home directory 
```


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

# 3. Manage a highly-available Kubernetes cluster

- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/]
- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/]

- You can set up a highly-available Kubernetes cluster as follows:
    - with stacked control plane nodes, where etcd nodes are colocated with
      control plane nodes
    - with external etcd nodes, where etcd runs on separate nodes from the
      control plane

- Note: For both methods you need this infrastructure:

    - Three machines that meet kubeadm's minimum requirements for the control-plane nodes
    - Three machines that meet kubeadm's minimum requirements for the workers
    - Full network connectivity between all machines in the cluster (public or private network)
    - sudo privileges on all machines
    - SSH access from one device to all nodes in the system
    - kubeadm and kubelet installed on all machines. kubectl is optional.

- For the external etcd cluster only, you also need:
    - Three additional machines for etcd members
    - First steps for both method 

- One way to gain HA is to use the 'kubeadm' command and join at least 2 control
  plane servers to the cluster. The command is similar to joining a worker node
  to the cluster except it includes some additional flags (--control-plane flag
  and a certificate-key flag). Note: the key will likely need to be regenerated
  if the control plane nodes are added 2 hours after the cluster in initialised.

## Join Control Plane Nodes
- edit /etc/hosts on all nodes to ensure alias is set ... come back to this (if
  including load balancer step ... maybe not required for exam)
- on the first cp, create the tokens and hashes that are required to join the
  cluster
```
sudo kubeadm token create
```
- create a new SSL hash
```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin
-outform der 2>/dev/null | openssl dgst -sha256 -hex | sed's/ˆ.* //'

```
- create a new cp certificate to join as a cp instead of a worker
```
sudo kubeadm init phase upload-certs --upload-certs
```
- on the second cp use the previous output to build the kubeadm join command
```
sudo kubeadm join k8scp:6443

```
- to verify, run the following on the first cp node:
```
kubectl get nodes
```
- copy over the config files as suggested in the output

## Simulate a node failure
- shutdown docker on node that shows 'IS LEADER' set to true
```
sudo systemctl stop docker.service
```
- check the logs to see updates referring to leader loss.

- view the status using etcdctl


# 4. Provision underlying infrastructure to deploy a Kubernetes cluster
- cloud, multi-cloud, on-premises, hybrid, combinaton thereof.
- possibly not tested in the exam ... review

- you can download kubernetes and deploy a Kubernetes cluster on a local
  machine, in your own data centre, into a public cloud.  
- public cloud providers also offer managed Kubernetes services if you do not
  want to manage the cluster yourself (e.g. Elastic Kubernetes Service from AWS
  or Google Kubernetes Engine from google).
- For critical workloads to run on a production-grade set up see
  [https://kubernetes.io/docs/setup/production-environment/]



# 5.  Perform a version upgrade on a Kubernetes cluster using Kubeadm
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

- Firstly, update the package data for apt.
```
sudo apt update
```
- Check your current versions
```
sudo kubeadm version
```
- Find the latest kubernetes package available. It should look like 1.21.x-00, where x is the latest patch.
```
sudo apt-cache kubeadm
```
- Remove the hold placed on kubeadm:
```
sudo apt-mark unhold kubeadm
```
- Update the package. Replace the x in the 1.21.x-00 with latest patch version
```
sudo apt-get install -y kubeadm=1.21.x-00
```
- Place a hold on the package again to prevent further updates:
```
sudo apt-mark hold kubeadm
```
- Verify the version of 'kubeadm' installed:
```
sudo kubeadm version
```
- Note:
- upgrading control plane nodes should be done one node at a time
- the control plane must have the /etc/kubernetes/admin.conf file.

- To prepare the control plane for an update, you first need to evict as many
  pods as possible.  You can ignore daemonsets (Calico). 
```
kubectl drain k8scp --ignore-daemonsets
```
- You can use the 'upgrade plan' argument to check the existing cluster, to see
  if it can be upgraded.
```
sudo kubeadm upgrade plan
```

- Note: kubeadm upgrade also automatically renews the certificates that it
  manages on the node.
- Note: if the kubeadm upgrade plan output shows any component configs that
  require manual upgrade, you must provide a config file with replacement
  configs to kubeam upgrade apply via the --config cli flag.
- Next, choose a version to upgrade to:
```
sudo kubeadm upgrade apply v1.21.x
```
- once the command finishes you should see:
```
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.21.x". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with
upgrading your kubelets if you haven't already done so.
```
- Check the status of the nodes:
```
kubectl get nodes
```
- Release the hold on kubelet and kubectl:
```
sudo apt-mark unhold kubelet kubectl
```
- Upgrade both packages to the same version as kubeadm:
```
sudo apt-get install -y kubelet=1.21.x-00 kubectl=1.21.x-00
```
- Re-apply the hold so other updates don't update the Kubernetes version.
```
sudo apt-mark hold kubelet kubectl
```
- Restart the daemons
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
- Verify that the control plane node has been updated to the new version.  Then
  update the other control plane nodes using the same process as above. Note:
  instead of using 'kubeadm upgrade apply', you will use 'kubeadm upgrade node'.

- Make the control plane availabe for the scheduler:
```
kubectl uncordon k8scp
```
- Verify that the control plane is in a ready status:
```
kubectl get nodes
```



sudo apt-get install --only-upgrade kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```





# 6. Implement etcd backup and restore

- [https://etcd.io/]
- [https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster]

- etcd is an open source distributed key-value store used as Kubernetes' backing store for all cluster
  data (i.e. state data, metadata, config data).
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

## Built-in snapshot method:

- Firstly, locate the etcd pods on the control plane node:
```
kubectl get pods -n kube-system
```

- Interact with etcd by using etcdctl from inside an etcd Pod.
```
kubectl -n kube-system exec -it etcd-<Tab> -- sh
```

```
etcdctl -h  #<-- view options and arguments available to ectdctl
etcdctl version  #<-- to get version
```
- In order to take a snapshot, you need to authenticate via certificates.  Check
  the configuration file on the control plane node for the 3 required files
  (trusted-ca-file, cert-file, and key-file).
```
cat /etc/kubernetes/manifests/etcd.yaml
```

- These files can be viewed on an etcd Pod at:
```
cd /etc/kubernetes/pki/etcd
echo *
```

- Check the health of etcd
```
student@control-plane:~$ kubectl -n kube-system exec -it etcd-control-plane -- sh \
-c "ETCDCTL_API=3 \
ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt \
ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt \
ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key \
etcdctl endpoint health"
```
- output should be similar to:
```
127.0.0.1:2379 is healthy: successfully committed proposal: took = 79.649196m
```
- Determine how many db's are part of the etcd cluster. 3 to 5 are common in a
  production enviroment to provide 50%+1 quorum

```
kubectl -n kube-system exec -it etcd-control-plane -- sh -c "ETCDCTL_API=3 \
ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt \
ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key \
etcdctl --endpoints=https://127.0.0.1:2379 member list -w table"
```
- if you see 'IS LEARNER' equal to true, it means one of the etcd db's has not
  caught up with the leader db.  If it is false, it means it's ok.  


- Take the snapshot.
```
ETCDCTL_API=3 etcdctl snapshot save <backup-file> \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file>
```

- verify the snapshot taken
```
student@control-plane:~$ kubectl -n kube-system exec -it etcd-control-plane --
sh \
-c "ETCDCTL_API=3 etcdctl --write-out=table snapshot status
/var/lib/etcd/snapshot.db"

```
- you should see output similar to:
```
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| fe01cf57 |    34018 |       968  | 3.1 MB     |
+----------+----------+------------+------------+
```


Step 3: restore, etcd supports restoring from snapshots that are taken from an
etcd process of the major.minor version.
```
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db
```




