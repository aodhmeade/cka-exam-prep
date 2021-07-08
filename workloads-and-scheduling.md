| ** Workloads and Scheduling 15%** |
|--------------------------------------------------------------------------------------------|
| 1.  Understand deployments and how to perform rolling update and rollbacks | 
| 2.  Use ConfigMaps and Secrets to configure applications |
| 3.  Know how to scale applications |
| 4.  Understand the primitives used to create robust, self-healing, application deployments |
| 5.  Understand how resource limits can affect Pod scheduling |
| 6.  Awareness of manifest management and common templating tools |



# **1.  Understand deployments and how to perform rolling update and rollbacks**
[https://kubernetes.io/docs/concepts/workloads/controllers/deployment/]

## Deployments
- A 'Deployment' provides declaritive updates for Pods and ReplicaSets.

- The following is an example of a Deployment.  When applied it will create a
  'ReplicaSet' to bring up 3 nginx Pods:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort:80
```

- To create the above deployment, run the following command:
```
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
```

- Run 'kubectl get deployments', to check if the Deployment was created.
- To see the ReplicaSet created by the Deployment, run 'kubectl get rs'.
- To see the Pods, run 'kubectl get pods'.
- To see the labels automatically generated for each Pod, run 'kubectl get pods
  --show-labels'.

## Rollouts and Rollbacks
- A deployment's rollout is triggered if and only if the deployment's Pod
  template (that is, .spec.template) is changed (e.g. if the labels or container
  images of the template are updated). Other updates, such as scaling requests
  do not trigger a rollout.

- Update the version of the nginx image to 1.16.1

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16.1
        ports:
        - containerPort:80
```

- Method 1: declaratively
update the file manually.

Method 2: imperatively
- 'kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1'

- To see the rollout status, run 'kubectl rollout status
  deployment/nginx-deployment' 

- Deployment ensures that only a certain number of Pods are down while they are
  being updated. By default, it ensures that at least 75% of the desired number
  of Pods are up (25% max unavailable).

- Deployment also ensures that only a certain number of Pods are created above
  the desired number of Pods. By default, it ensures that at most 125% of the
  desired number of Pods are up (25% max surge).

- To get details of your deployment, run 'kubectl describe deployments'.


Rollback
- To check the revisions of a Deployment, 'kubectl rollout history
  deployment.v1.apps/nginx-deployment'.
- To see a specific revision, run 'kubectl rollout history
  deployment.v1.apps/nginx-deployment --revision=2'.
- To undo the current rollout and rollback to the previous revision run,
  'kubectl rollout undo deployment.v1.apps/nginx-deployment'


# **2.  Use ConfigMaps and Secrets to configure applications**
## ConfigMaps

[https://kubernetes.io/docs/concepts/configuration/configmap/] 

- A ConfigMap is an API object used to store non-confidential data in key-value
  pairs. Pods can consume ConfigMaps as environment variables, cli arguments, or
  as configuration files in a volume.  It allows you to decouple
  environment-specific configuration from your container images.  For example,
  using the same code but with different configuration for dev, test, production
  environments.  Note: they do not offer secrecy or encryption.  Data is stored
  in plaintext format.  Use a 'Secret' if this is a requirement.

- Data stored in a ConfigMap cannot exceed 1 MiB.

- Unlike most Kubernetes objects that have a 'spec', a ConfigMap has 'data' and
  'binaryData' fields. These fields accept key-value pairs as their values.

- The Pod and the ConfigMap must be in the same 'namespace' and the order of
  execution matters, i.e. the ConfigMap must be already created for a Pod to
  reference it.

- Imperative: The basic syntax for creating a config map is:
```
kubectl create configmap [configmap-name] [attribute] [source]
```

- Depending on the source the attribute will be: --from-file or --from-literal.
  The configmap-name you give is arbitrary.  

```
kubectl create configmap my-config-map-test --from-file [path/to/yaml/file]
kubectl create configmap my-config-map-test --from-literal=website=kubernetes.io
```

- To review the ConfigMap created, run 'kubectl describe configmap
  my-config-map-test', or 'kubectl get configmaps my-config-map-test -o yaml'

- To refer to this ConfigMap, declare it in the Pod that will be using it.
  Multiple Pods can reference the same ConfigMap.

- There are two ways to configure a Pod to use a specific ConfigMap: either as a
  mounted volume or by using environment variables.


## Secrets

[https://kubernetes.io/docs/concepts/configuration/secret/]

- A 'Secret' is a similar API resource to a 'ConfigMap'.

- Kubernetes 'Secrets' let you store and manage sensitive information, such as
  passwords, OAuth tokens, and ssh keys.  Users can create Secrets and the
  system also creates secrets.

- Kubernetes Secrets are, by default, stored as unencrypted base64-encoded
  strings.  By default they can be retrieved - as plaintext - by anyone with API
  access, or by anyone with access to etcd.  Therefore, it is recommended that
  you (at a minimum):
    1. enable encryption at rest.
    2. enable or configure RBAC rules that restrict the reading of and writing
    to.

- To use a Secret, a Pod needs to reference the Secret.  It can be used with a
  Pod in 3 ways:
    as files in a volume mounted on one or more containers
    as container environment variable
    by the kubelet when pulling the images for the Pod

- The name of the Secret must be a valid DNS subdomain name.

- When creating a Secret, you specify its 'type'.

- There is no limit to the number of Secrets that can be used but there is a 1MB
  limit to their size.

- To check for existing Secrets, run 'kubectl get secrets'.

## Create a Secret using a configuration file (declaratively)

- To store 2 strings in a Secret using the 'data' field, convert the strings to
  base64 as follows:

'echo -n 'admin' | base64'

- The output is similar to:
'YWRtaW4='

'echo -n '1f2d1e2e67df' | base64'

- The output is similar to:

'MWYyZDFlMmU2N2Rm'

- Write a Secret file that looks like:

```
$ vim secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-first-secret
type: Opaque  
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```  

- Now create the Secret using 'kubectl apply'
'kubectl apply -f ./secret.yaml'

- 'kubectl get' and 'kubectl describe' can be used to check the Secret. They
  will avoid showing the contents of the Secret by default - to protect from
  accidental exposure, terminal log store.

### Createing a Secrect using kubectl (Imperatively)
- Example scenario: Pod needs access to a database.  You need somewhere to store
  the db connection string.

- Create a username and password and store them in files.
```
echo -n 'my-db-username' > ./username.txt
echo -n 'my-db-password' > ./password.txt

# note: the -n flag stops a /n char being added to the file and being encoded
# too.
```

```
kubectl create secret generic db-username-pass \
--from-file=./username.txt \
--from-file=./password.txt
```

- Note: the default key name is the filename. You can optionally set the key
  name using the '--from-file=[key=]source'.

```
kubectl create secret generic db-username-pass \
--from-file=username=./username.txt \
--from-file=password=./password.txt
```

- You can also provide Secret data using the '--from-literal=<key>=<value>' tag.
  Note: special characters will be interpreted by your shell and require
  escaping.  Surrounding it with single quotes should be sufficient.

```
kubectl create secret generic dev-db-secret \
  --from-literal=username=devuser \
    --from-literal=password='S!B\*d$zDsb='
```
- To verify, run 'kubectl get secrets'

- To analyse, run 'kubectl describe secret db-username'

- 'kubectl get secret <secret-name> -o jsonpath='{.data}'

- 'echo '<base64-encoded-value>' | base64 --decode'

- 'kubectl delete secret <secret-name>'

- To edit an existing Secret, run 'kubectl edit secrets <secret-name>'. This
  will open up the default editor & allow for updating the base64 encoded Secret
  values in the 'data' field.

### using secrets via Environment Variables
- You can mount secrets as files using a volume definition in a pod manifest.
  This mount path will contain a file whose name will be the key of the secret
  created with the 'kubectl create secret ...' step.

```
...
spec:
  containers:
- image: busybox
command:
- sleep
- "3600"
volumeMounts:
- mountPath: /mysqlpassword
name: mysql
name: busy
volumes:
- name: mysql
secret:
secretName:
mysql
```

- Once the Pod is running, you can verify that the secret is accessible in the
  container:
```
$ kubectl exec -it busybox -- cat /mysqlpassword/password
LFTr@1n
```

### using Secrets via Environment Variables
- A Secret can be used as an environment variable in a Pod. 


```
spec:
containers:
- image: mysql:5.5
env:
- name: MYSQL_ROOT_PASSWORD
valueFrom:
secretKeyRef:
name: mysql
key:password
name: mysql
```

- You can set the file access permission bits for a single Secret key.  If you
  don't specify any permissions, '0644' is used by default.
- Mounted Secrets are updated automatically.  Environment variables are not
  updated after a Secret update.
- Immutable Secrets ... maybe not required for the exam ... 



# **3.  Know how to scale applications**

- You can scale a Deployment by using the following command:
```
kubectl scale deployment nginx-deployment --replicas=10
```
- This command can be used to scale up and down.
- This is ok if you are dealing with a small number of Pods.  If you are dealing
  with a large number and want this implemented in an automated fashion, you can
  use autoscaling.  ... will this come up in the exam ... check
- You can use the following command:
'kubectl autoscale deployment nginx-deployment --min=2 --max=5'


# 4.  Understand the primitives used to create robust, self-healing, application deployments 

- Deployments, ReplicaSets, StatefulSets, DaemonSets, Jobs, CronJobs.





# 5.  Understand how resource limits can affect Pod scheduling
[https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/]

- When talking about managing resources for Containers, there are two terms to
  keep in mind: requests and limits.

- When you create a Pod, you can (it's optional) specify how much of each
  resource a container needs.  The most common to specify are CPU and RAM. When
  you specify the resource request, the scheduler uses this information to
  decide which node to place the Pod on.  When you specify a resoure limit, the
  kubelet enforces those limits (and conversely reserve limit).

- By default, Kubernetes assumes that a container within a Pod requires 0.5 CPU
  and 256Mi of memory.  If the container needs more than this, you need to
  manually set this in Pod definition file.

- By default, Kubernetes sets resource limits to 1 CPU and 512Mi of mem.  If the
  the limits need to be increased, you need to set this in the Pod definition
  file.

- Each container of a Pod can specify one or more of the following:

    - spec.containers[].resources.limits.cpu
    - spec.containers[].resources.limits.memory
    - spec.containers[].resources.limits.hugepages-<size>
    - spec.containers[].resources.requests.cpu
    - spec.containers[].resources.requests.memory
    - spec.containers[].resources.requests.hugepages-<size>

- See the official kubernetes.io link above to understand the meaning of the
  resource units in Kubernetes.

- At the namespace level, you can limit the total sum of compute resources that
  can be requested.











# 6.  Awareness of manifest management and common templating tools
