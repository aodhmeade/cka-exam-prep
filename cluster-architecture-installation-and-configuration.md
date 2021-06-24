| **Cluster Architecture, Installation and Configuration   25%**         |   |
|------------------------------------------------------------------------|---|
| 1.  Manage role based access control (RBAC)                            |   |
| 2.  Use Kubeadm to install a basic cluster                             |   |
| 3.  Manage a highly-available Kubernetes cluster                       |   |
| 4.  Provision underlying infrastructure to deploy a Kubernetes cluster |   |
| 5.  Perform a version upgrade on a Kubernetes cluster using Kubeadm    |   |
| 6.  Implement etcd backup and restore                                  |   |


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
kubectl create -f pod-reader.yaml
```

## kubectl

```
kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods
```

[official](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
[other](https://docs.bitnami.com/tutorials/configure-rbac-in-your-kubernetes-cluster/)



## 2.  Use Kubeadm to install a basic cluster                             


## 3.  Manage a highly-available Kubernetes cluster                       


## 4.  Provision underlying infrastructure to deploy a Kubernetes cluster 



## 5.  Perform a version upgrade on a Kubernetes cluster using Kubeadm    


## 6.  Implement etcd backup and restore                                  
