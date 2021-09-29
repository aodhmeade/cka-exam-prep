| **cluster architecture, installation and configuration   25%**         |
|------------------------------------------------------------------------|
| 1.  manage role based access control (rbac)                            |
| 2.  use kubeadm to install a basic cluster                             |
| 3.  manage a highly-available kubernetes cluster                       |
| 4.  provision underlying infrastructure to deploy a kubernetes cluster |
| 5.  perform a version upgrade on a kubernetes cluster using kubeadm    |
| 6.  [implement etcd backup and restore](#6-implement-etcd-backup-and-restore)|

# **1. manage role based access control (rbac)**

- [https://kubernetes.io/docs/reference/access-authn-authz/rbac/]
- [https://docs.bitnami.com/tutorials/configure-rbac-in-your-kubernetes-cluster/]

- to perform any action in a cluster, you need to access the api.
- three steps are performed when accessing the api: authentication,
  authorisation (abac, rbac, webhook, global deny/allow settings), and admission
  control. 
- rbac falls under authorisation.  it uses the rbac.authorization.k8s.io api
  group to manage authorisation decisions.  with the resources in this group,
  you can define roles and associate users to these roles.

summary of rbac process:
- determine or create namespace
- create certificate credentials for user
- set the credentials for the user to the namespace using a context
- create a role for the expected task set
- bind the user to the role
- verify the user has limited access

there are four resources in this api group: role, clusterrole, rolebinding, and
clusterrolebinding.
- to manage rbac, you need:
  - rules: this is an operation (verb) that can be carried out on a resource.
  - role & clusterrole: represent a set of permissions (rules).  they are additive
    (i.e. there are no "deny" rules).  a role sets permissions within a particular
    namespace.  clusterrole by contrast is non-namespaced.
  - subjects: is the entity that attempts an operation, of which there are 3:
      - user accounts: (human or otherwise) - external to the cluster.  these are
        not api objects.
      - service accounts:
      - groups:
  - rolebindings and clusterrolebindings: these bind subjects to roles.
    difference here is similiar to role versus clusterrole.  rolebindings
    applies to a namespace, whereas clusterrolebinding applies to all
    namespaces.

example yaml manifest for a 'role':
```yaml
apiversion: rbac.authorization.k8s.io/v1
kind: role
metadata:
  namespace: default
    name: pod-reader
    rules:
    - apigroups: [""] # "" indicates the core api group
      resources: ["pods"]
        verbs: ["get", "watch", "list"]
```
- to create the above manifest, save the yaml and create as follows:
`kubectl create -f role-pod-reader-example.yaml`.

example yaml manifest for a 'rolebinding':
```
kind: rolebinding
apiversion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: pod-reader-binding
  namespace: default
subjects:
- kind: user
  name: employee
  apigroup: ""
roleref:
  kind: role
  name: pod-reader 
  apigroup: ""
```
- to create the above manifest, save the yaml and create as follows:
`kubectl create -f rolebinding-pod-reader-example.yaml`.

### roles can be created imperatively by using kubectl
- create a role named "pod-reader" that allows a user to perform get, watch and
  list on pods `kubectl create role pod-reader --verb=get --verb=list
  --verb=watch --resource=pods`.

- create a role named "foo" with apigroups specified `kubectl create role foo
  --verb=get,list,watch --resource=replicasets.apps`. 

- to test the rbac setup you can use `kubectl auth can-i`.  run `kubectl auth can-i -h` to see some examples.  

# **2. use kubeadm to install a basic cluster**

- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/]
- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/]
- [https://kubernetes.io/docs/setup/production-environment/container-runtimes/]

- kubeadm is a cli tool that you can use to set up a k8s cluster from scratch.
- kubeadm is not the only tool. there are other vendor specific tools. 
- bootstrapping a kubernetes cluster with kubeadm essentially involves 3 steps: 
  - run 'kubeadm init' (to initialise the cp/head node)
  - apply a network plugin (a cni plugin),
  - run 'kubeadm join' (on a worker node).
- there are other commands such as kubeadm upgrade, kubeadm config, kubeadm
  token, kubeadm reset.
- before you begin, check to ensure you have a container runtime installed on
  your nodes.
- once your cluster is up and running, you would use the 'kubectl' command to
  interact with your cluster. kubectl uses $home/.kube/config as a configuration
  file.  this contains all the kubernetes endpoints that you may end up using,
  as well as other items such as credentials, contexts, definitions, etc.
- note: a context is a combination of a cluster and user credentials.  you can
  switch contexts with kubectl.  this is useful when switching between clusters,
  e.g. going from local to the cloud, `kubectl config use-context
  name-of-new-context`.

### installation steps for a basic cluster (one control plane node and one
worker node). steps performed using two virtual machines running ubuntu 18.04 on gce.

#### section 1 - install a control plane node
- switch to root, `sudo -i`.

- disable swap on all nodes, `swapoff -a`. with swap enabled, it's problematic for disk io
mangement, isolation management, etc. see github issues for some of the discussion around this area
e.g. [https://github.com/kubernetes/kubernetes/issues/53533]. 

- optional: update the file system table file to ensure it is off on all
  reboots. requires reboot to take effect, 
  `sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab`

- start by updating and upgrading software packages, `sudo apt update && sudo apt upgrade -y`

- next, install a container runtime engine, example - docker, `sudo apt install -y docker.io`

- configure cgroup drivers.  both the container runtime and the kubelet have a
property called "cgroup driver", which is important with respect to the
management of cgroups on linux. note: matching the container runtime and the
kubelet cgroup drivers is required, otherwise the kubelet process will fail.

    - [https://kubernetes.io/docs/setup/production-environment/container-runtimes/]
    - [https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/]

`cat << eof | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
eof`

- use systemd to restart docker and enable on boot
`sudo systemctl enable docker`
`sudo systemctl daemon-reload`
`sudo systemctl restart docker`

- add a new repo for kubernetes. create and edit as follows: 
`vim /etc/apt/sources.list.d/kubernetes.list`

`deb    http://apt.kubernetes.io/   kubernetes-xenial   main   #<-- add this line to the file`

- add a gpg key for the packages. 
`curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -`

- update the new repo, `apt-get update`.

- install the necessary software, `sudo apt-get install kubeadm`. 
    - the kubelet version may never exceed the api server version.  for
    example, the kubelet running v.1.20.0 should be fully compatible with a
    1.21.0 api server, but not vica versa.

- mark & hold the relevant packages so they are not upgraded: `apt-mark hold
kubeadm kubelet kubectl`. confirm: `apt-mark showhold`.

- decide which pod network to use for container networking interface (cni).
for this example, using calico. just wget for now.
`wget https://docs.projectcalico.org/manifests/calico.yaml`

- get the control plane node ip address and add to /etc/hosts
`hostname -i`
`vim /etc/hosts`
`10.128.0.3 k8scp #<-- add this line (change ip to match output from hostname -i)`

- initialise the control plane.
`kubeadmin init --config=kubeadm-config.yaml --upload-certs | tee kubeadmin-init.out`

- to start using your cluster, you need to make the following config changes:
`mkdir -p $home/.kube`
`sudo cp -i /etc/kubernetes/admin.conf $home/.kube/config # copy the default config to your home directory`

- apply the cni plugin
`kubectl apply -f calico.yaml #<-- using the file obtained via wget above`

#### section 2 - grow the cluser - add worker node(s)
- follow all the steps above except initialising, i.e. don't run `kubeadm
init`.

- at this point we could copy and paste the join command from the cp node. that
command only works for 2 hours, so we will build our own join should we want to
add nodes in the future. find the token on the cp node. the token lasts 2 hours
by default. if it has been longer, and no token is present you can generate a
new one with the sudo kubeadm token create command, seen in the following
command. on the cp node:

`kubeadm token list`
`kubeadm token create --print-join-command`

- use the above output on the worker node:

`kubeadm join k8scp:6443 --token qsqg38.g9pohmf6pjdw0hkw --discovery-token-ca-cert-hash sha256:w43jlj2l4htofuu4tl4ry9340wgslkgf9732097448r028280220j`

- to verify if node has joined, run this on the cp: `kubectl get nodes`.

# **3. manage a highly-available kubernetes cluster**

- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/]
- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/]

- you can set up a highly-available kubernetes cluster as follows:
    - with stacked control plane nodes, where etcd nodes are colocated with
      control plane nodes
    - with external etcd nodes, where etcd runs on separate nodes from the
      control plane

- note: for both methods you need this infrastructure:

    - three machines that meet kubeadm's minimum requirements for the control-plane nodes
    - three machines that meet kubeadm's minimum requirements for the workers
    - full network connectivity between all machines in the cluster (public or private network)
    - sudo privileges on all machines
    - ssh access from one device to all nodes in the system
    - kubeadm and kubelet installed on all machines. kubectl is optional.

- for the external etcd cluster only, you also need:
    - three additional machines for etcd members
    - first steps for both method 

- one way to gain HA is to use the 'kubeadm' command and join at least 2 control
  plane servers to the cluster. the command is similar to joining a worker node
  to the cluster except it includes some additional flags (--control-plane flag
  and a certificate-key flag). note: the key will likely need to be regenerated
  if the control plane nodes are added 2 hours after the cluster in initialised.

### join control plane nodes
- edit /etc/hosts on all nodes to ensure alias is set ... come back to this (if
  including load balancer step ... maybe not required for exam)
- on the first cp, create the tokens and hashes that are required to join the
  cluster, `sudo kubeadm token create`.
- create a new ssl hash
```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin
-outform der 2>/dev/null | openssl dgst -sha256 -hex | sed's/ˆ.* //'
```
- create a new cp certificate to join as a cp instead of a worker
`sudo kubeadm init phase upload-certs --upload-certs`

- on the second cp use the previous output to build the kubeadm join command
`sudo kubeadm join k8scp:6443`

- to verify, run the following on the first cp node:
`kubectl get nodes`

- copy over the config files as suggested in the output

### simulate a node failure
- shutdown docker on node that shows 'is leader' set to true
`sudo systemctl stop docker.service`

- check the logs to see updates referring to leader loss.

- view the status using etcdctl

# **4. provision underlying infrastructure to deploy a kubernetes cluster**

- cloud, multi-cloud, on-premises, hybrid, sbc's, etc., combinaton thereof.
- possibly not tested in the exam ... review
- public cloud providers - managed kubernetes services - if you do not
  want to manage the cluster yourself (e.g. elastic kubernetes service from aws
  or google kubernetes engine from google).
- for critical workloads to run on a production-grade set up see
  [https://kubernetes.io/docs/setup/production-environment/]

# **5.  perform a version upgrade on a kubernetes cluster using kubeadm**
- [https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/]

- if you build your cluster with kubeadm, you also have the option to upgrade
  the cluster using the kubeadm upgrade command.
- skipping minor versions when upgrading is unsupported.
- the upgrade workflow at a high level is as follows:
    1. upgrade a primary control plane node.
    2. upgrade additional control plane nodes.
    3. upgrade worker nodes.
- note: real world scenario requires certain precautions to be taken into
  consideration, such as disabling swap in linux.  see the official kubernetes.io
  guidance for more detailed information.
- determine which version to upgrade to.  find the latest stable kubernetes
  version using os package manager ('apt' as the exam is based on ubuntu):

- `kubectl get node`: will give you the version your nodes are running on.

- useful to mark the versions of kubeadm, kubectl and kubelet before you begin:
  `sudo apt-mark hold kubeadm kubelet kubectl`

- update the package data for apt, `sudo apt update`

- check your current versions,`sudo kubeadm version`, `sudo kubelet --version`,`kubectl version --short` 

- find the latest kubernetes package available. it should look like 1.21.x-00, where x is the latest patch.
`sudo apt-cache madison kubeadm`

- remove the hold placed on kubeadm,`sudo apt-mark unhold kubeadm`

- update the package. replace the x in the 1.21.x-00 with latest patch version
`sudo apt-get install -y kubeadm=1.21.x-00`

- place a hold on the package again to prevent further updates:
`sudo apt-mark hold kubeadm`

- verify the version of 'kubeadm' installed:
`sudo kubeadm version`

- Note: upgrading control plane nodes should be done one node at a time
- Note: the control plane must have the /etc/kubernetes/admin.conf file.

- to prepare the control plane for an update, you first need to evict as many
  pods as possible.  you can ignore daemonsets (calico). 
`kubectl drain k8scp --ignore-daemonsets`

- you can use the 'upgrade plan' argument to check the existing cluster, to see
  if it can be upgraded: `sudo kubeadm upgrade plan`

- Note: kubeadm upgrade also automatically renews the certificates that it
  manages on the node.

- Note: if the kubeadm upgrade plan output shows any component configs that
  require manual upgrade, you must provide a config file with replacement
  configs to kubeam upgrade apply via the --config cli flag.

- next, choose a version to upgrade to:
`sudo kubeadm upgrade apply 1.21.x`

- once the command finishes you should see:
```
[upgrade/successful] success! your cluster was upgraded to "v1.21.x". enjoy!

[upgrade/kubelet] now that your control plane is upgraded, please proceed with
upgrading your kubelets if you haven't already done so.
```
- check the status of the nodes:
`kubectl get nodes`

- release the hold on kubelet and kubectl:
`sudo apt-mark unhold kubelet kubectl`

- upgrade both packages to the same version as kubeadm:
`sudo apt-get install -y kubelet=1.21.x-00 kubectl=1.21.x-00`

- re-apply the hold so other updates don't update the kubernetes version.
`sudo apt-mark hold kubelet kubectl`

- restart daemons & kubelet
`sudo systemctl daemon-reload` and `sudo systemctl restart kubelet`.

- verify that the control plane node has been updated to the new version.  then
  update the other control plane nodes using the same process as above. note:
  instead of using 'kubeadm upgrade apply', you will use 'kubeadm upgrade node'.

- make the control plane availabe for the scheduler:
`kubectl uncordon k8scp`

- verify that the control plane is in a ready status:
`kubectl get nodes`


# 6 implement etcd backup and restore

- [https://etcd.io/]
- [https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster]

- etcd is an open source distributed key-value store used as kubernetes' backing store for all cluster
  data (i.e. state data, metadata, config data). it provides a single source of truth about the status of the system at any given point in time. 
- etcd is built on the raft consensus algorithm to ensure data consistency.
- kubernetes uses etcd's "watch" function to monitor cluster state data, compare
  against ideal state and to reconfigure itself when changes occur.  
- the 'etc' in etcd references the unix/linux config directory /etc. the 'd' in
  etcd stands for distributed.
- periodically backing up the etcd cluster data is important to recover
  kubernetes clusters in a dr event.
- etcdctl: is a command line client for etcd.
- official documentation advises to stop kube-apiservers before restore to
  ensure there is no reliance on stale data. as regards the exam ...?
- backup can be accomplished in two ways: etcd built-in snapshot or volume
  snapshot.

### built-in snapshot method:
- Restore steps will depend on how etcd is deployed i.e. whether it has been set
  up as a stacked etcd service, as an external service (running as a daemon), or
  as a static pod.  In this case, we are backing up and restoring etcd which is running as a static pod.

- Find the 'staticPodPath'. It can be found in the kubelet config file at
  '/var/lib/kubelet/config.yaml': `staticPodPath=/etc/kubernetes/manifests`.
  There you should find the 'etcd.yaml' file.

- Locate the etcd pods on the control plane node: `kubectl get pods -A | grep etcd`

- You can interact with etcd either from the control node, or by using etcdctl
  in the container inside an etcd Pod:
  - `apt install etcd-client` if not available on the main node.
  - `kubectl -n kube-system exec -it etcd-<Tab> -- sh -c "<commands here>"` if using
the etcd client on the etcd pod itself.

    - `etcdctl -h`  #<-- view options and arguments available to ectdctl
    - `etcdctl version`  #<-- to get version

- In order to take a snapshot, you need to authenticate via certificates (if
  --client-cert-auth is set to true in /etc/kubernetes/manifests/etcd.yaml).  Check
  the configuration file on the control plane node for the 3 required files
  (trusted-ca-file, cert-file, and key-file): `cat /etc/kubernetes/manifests/etcd.yaml`

- To check the health of etcd (checking from the control node):

```bash
ETCDCTL_API=3 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
etcdctl endpoint health
```
- output should be similar to:
`127.0.0.1:2379 is healthy: successfully committed proposal: took = 79.649196m`

- You can also determine how many db's are part of the etcd cluster. 3 to 5 are common in a
  production environment to provide 50%+1 quorum

```bash
ETCDCTL_API=3 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
etcdctl --endpoints=https://127.0.0.1:2379 member list -w table
```
- if you see 'IS LEARNER' equal to true, it means one of the etcd db's has not
  caught up with the leader db.  If it is false, it means it's ok.  

- Take the snapshot.
```
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file>
```
- Verify the snapshot just taken
```
ETCDCTL_API=3 etcdctl --write-out=table snapshot status /tmp/etcd-backup.db"
```
- you should see output similar to:
```
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| fe01cf57 |    34018 |       968  | 3.1 MB     |
+----------+----------+------------+------------+
```

- At this point, you could run a new test pod.  Once you perform a restore, this
  test pod should no longer be running as it was created after the snapshot was
  taken: `kubectl run testpod --image=nginx -- /bin/sh -c 'sleep 3600'`

Step 3: restore snapshot db to a specific directory using the '--data-dir'
argument. The data directory that etcd is using can be found by: `grep -i
data-dir /etc/kubernetes/manifests/etcd.yaml`. Output should be similar to: `-
--data-dir=/var/lib/etcd`. In the restore, we will set the '--data-dir' to a new
directory (you don't need to create this, it will be created for you). A number
of other commands are issued (most of which are copied from the etcd.yaml
manifest).
   - --name ip-172.18.3.48 #<-- 
   - --initial-advertise-peer-urls=https://[IP]:2380 #<-- 
   - --initial-cluster=ip-172-31-5-141=https://[IP]:2380 #<-- 
   - --initial-cluster-token=etcd-cluster-1  #<-- 
   - --skip-hash-check=true #<-- 

```bash
export ETCDCTL_API=3 \
etcdctl snapshot restore /tmp/etcd-backup.db \
--data-dir /var/lib/etcd-backup \
--name ip-172.18.3.48 \
--initial-advertise-peer-urls=https://[IP]:2380 \
--initial-cluster=ip-172-31-5-141=https://[IP]:2380 \
--initial-cluster-token=etcd-cluster-1 \
--skip-hash-check=true 
```

- you should see out put similar to (if doing so from the control node):
```
2021-09-28 10:05:25.730702 I | mvcc: restore compact to 113090
2021-09-28 10:05:25.737697 I | etcdserver/membership: added member
8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32
```

- now tell etcd to use that directory by updating
  `/etc/kubernetes/manifests/etcd.yaml` and update the hostPath for etcd-data:

```bash
  volumes:
  - hostPath:
    path: /var/lib/etcd-backup
    type: DirectoryOrCreate
  name: etcd-data
```
- it will take a minute or two for etcd and the api-server restart/reconnect.  
- you should **not** see your test pod (created after snapshot taken).
- if you revert the hostPath change back to /var/lib/etcd you should see the
  test pod created earlier.
- you may have to restart kubelet and docker:
  - `system daemon reload` 
  - `systemctl restart docker`
  - `systemctl restart kubelet`
