| **Troubleshooting 30%**                   |   
|--------------------------------------------|
| 1.  Evaluate cluster and node logging      |  
| 2.  Understand how to monitor applications |  
| 3.  Manage container stdout & stderr logs  |  
| 4.  Troubleshoot application failure       |  
| 5.  Troubleshoot cluster component failure |  
| 6.  Troubleshoot networking                |  

#### Useful to understand the pod lifecycle as a starting point:
[https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/]

#### An observation from kubernetes' practitioners:
Kubernetes relies on API calls and is sensitive to network issues.  Therefore,
standard linux tools (e.g. dig, tcpdump) can be very useful in troubleshooting your cluster.

# 1.  Evaluate cluster and node logging
https://kubernetes.io/docs/concepts/cluster-administration/logging/

### Node level logging architecture:
- a container engine handles & redirects any output generated to a containerised
  applications stdout and stderr streams.  For example, the Docker container
  engine redirects those two streams to a logging driver, which is configured in
  Kubernetes to write to a file in JSON format.
- by default, if a container restarts, the kubelet keeps one terminated
  container with its logs. If a pod is evicted from the node, the logs are also
  evicted.
- real world consideration in node-level logging is implementing log rotation.
  Kubernetes is not responsible for this.
- there are two types of system components: those that run in a container and
  those that do not. For example:
    - kubernetes scheduler & kube-proxy run in a container.
    - kubelet & container runtime do not run in containers.
- on machines with systemd, the kubelet & container runtime write to journald.
  if systemd is not present, the kubelet & container runtime write to .log files
  in /var/log directory.  System components inside containers always write to
  the /var/log directory, bypassing the default logging mechanism.
    Control Plane
    - /var/log/kube-apiserver.log # responsible for serving the API
    - /var/log/kube-scheduler.log # responsible for scheduling decisions
    - /var/log/kube-controller-manager.log # responsible for managing
      replication controllers
    Worker Nodes
    - /var/log/kubelet.log # responsible for running containers on the node
    - /var/log/kube-proxy.log # responsible for service load-balancing
    Various Container Logs
    - /var/log/containers
    More log files for current pods
    - /var/log/pods

### Cluster level logging architectures:
- Kubernetes does not provide an out-of-the-box solution for cluser-level
  logging.
- Common options include:
    - use a node-level logging agent that runs on every node.
    - include a dedicated sidecar container for logging in an application pod.
    - push logs directly to a backend from within the application.

#### streaming sidecar container and sidecar container with a logging agent:
https://kubernetes.io/docs/concepts/cluster-administration/logging/

- a sidecar container can provide some additional help/assistance to the main
  application container.  Where this construct sees a lot of use is in the
  provision of logging capabilities.

# 2.  Understand how to monitor applications
- if your container image includes debugging utilities, you can run commands
  inside a container with `kubectl exec`.

`kubectl exec --stdin --tty <pod-name> -- /bin/bash` if only one container
running

`kubectl exec -i -t <pod-name> --container <container-name> -- /bin/bash` # if pod
has more than one container. Note: the short options -i and -t are the same as
the long options --stdin and --tty.

- to get container count on a node, run: `sudo docker ps | wc -l`

#### debugging with an ephemeral debug container
- https://kubernetes.io/docs/tasks/debug-application-cluster/debug-running-pod/#ephemeral-container-example

- Ephemeral containers can be used for interactive troubleshooting when 'kubectl
exec' won't work because the container is no longer running or the container
image does not contain debugging utilities (no shell).  Examples might include
distroless images.

- Note: You need to enable the 'EphemeralContainers' feature gate in your
  cluster and you require kubectl version 1.22 or later.   
  - Step 1 - add the following to the files below: `-
    --feature-gates=EphemeralContainers=true`
    - /etc/kubernetes/manifests/kube-apiserver.yaml
    - /etc/kubernetes/manifests/kube-scheduler.yaml
  - Step 2 - you need to pass the same parameter to
    '/etc/systemd/system/kubelet.service.d/10-kubeadm.conf' as follows:
    `ExecStart=/usr/bin/kubelet --feature-gates=EphemeralContainers=true $KUBELET ...`
    Then restart the kubelet service: `sudo systemctl daemon-reload` and `sudo
    systemctl restart kubelet`
  - Step 3 - you will need to perform step two on each worker node as well.

- First create a pod with a minimalist container image (no debugging utilities):
  `kubectl run ephem-demo --image=k8s.gcr.io/pause:3.1 --restart=Never`

- Try `kubectl exec -it ephem-demo -- sh`. This will fail because there is no
  shell in this container.

- Add a debugging container using `kubectl debug -it ephem-demo
  --image=busybox --target=ephem-demo`

- You should be attached to the console of the container you are looking to
  debug:
`Defaulting debug container name to debugger-9brzt`

- In a new terminal, you can get the details of the container and your
  ephem-demo container with: `kubectl describe pod ephem-demo`

### liveness probes, readiness probes and startup probes
- [https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/]

- kubelet will use 'liveness probes' to know when to restart a container.
- kubelet will use 'readiness probes' to know when a container is ready to start
  handling traffic.
- kubelet will use 'startup probes' to know when a container application has
  started (if this is used, liveness & readiness probes are disabled until it
  succeeds).

#### liveness probe
- you can run a shell command, a TCP probe, or a http GET.
- you must specify the type of probe and pass some parameters to it
    - initialDelaySeconds: how long to wait before starting checks.
    - periodSeconds: how often to run the check.

   ```yaml
   ...
     livenessProbe:
       exec:
         command:
         - cat
         - /tmp/healthy
       initialDelaySeconds: 10
       periodSeconds: 5
   ```
#### readiness probe
- similar to liveness probes.

#### startup probes
- adds a `failureThreshold` to the manifest.
- this is multiplied with `periodSeconds` to work out the time allowed before a
  startup failure should be reported.

# 3.  Manage container stdout & stderr logs
- container standard out can be seen via the 'kubectl logs' command.
- for example, `kubectl -n <a-namespace> logs <pod-name>


# 4.  Troubleshoot application failure

- Inevitably, you will need to debug problems with your application.
- First step is to check if a pod is running: `kubectl get pods`
- If the Pod is not ready, use 'describe' to get more specific information from
  a pod. This provides information about the Pod as well as a list of events
  that describe the initialisation process which may show issues: `kubectl describe pod <pod-name>`
- To get extra detail, pass the '-o yaml' output format flag to the 'kubectl
  get' pod command: `kubectl get pod <pod-name> -o yaml`
- To obtain the stdout & stderr logs from the containers in the pod (the level
  of detail will depend on how the application has been configured): `kubectl
  logs <pod-name>`. If more than one container in a Pod, `kubectl logs -f <pod-name> -c <containername> -n <namespace>`
- If the container has crashed, you can try and check the logs using the
  'previous' flag: `kubectl logs <pod-name> --previous -c <containername> -n <namespace>`

### some example Pod states
Pending: maybe insufficient resources, requesting more than what is available.
    - Try: free up resources, add more nodes to the cluster
Waiting: cannot start.
    - Try: checking image name is correct, if image is available
ImagePullBackOff: image problem.
    - Try: checking image name is correct and is available.
CrashLoopBackOff: error in the container that causes Pod to restart.
    - Try: use 'describe' to try and ascertain problem, export spec and review.

- kubectl explain pods

# 5.  Troubleshoot cluster component failure
If you have ruled out your app as the root cause of the problem you are
experiencing then you can troubleshoot your cluster components.  

Control Plane Components
    kube-apiserver
    etcd
    kube-scheduler
    kube-controller-manager
    cloud-controller-manager
Node Components
    kubelet
    kube-proxy
    container runtime (kubernetes supports several - docker, containerd, cri-o)
    CNI
Addons, DNS, Web UI (Dashboard)

Container Resource Monitoring

- first thing to check is if all cluster nodes are registered correctly and in a ready state
`kubectl get nodes`
- to get more information on a node: `kubectl describe node <node-name>`
- to get extra detail on a node, pass the output to yaml: `kubectl get node
  <node-name> -o yaml`
- check for event errors with: `kubectl get events
  --namespace=<namespace-name>` or `kubectl get events --all-namespaces'
- to get information about the overall health of a cluster, you can run:
  `kubectl cluster-info dump`
- to get the status of a particular service, e.g.: `service kube-apiserver status`

### kube-apiserver
`journalctl -u kube-apiserver`
or
`cat /var/log/kube-apiserver.log`

### kube-scheduler 
`journalctl -u kube-scheduler`
or
`cat /var/log/kube-scheduler.log`

### kube-controller-manager
`journalctl -u kube-controller-manager`
or
`cat /var/log/kube-controller-manger.log`

- The next step would be to look deeper into the relevant machines hosting the
  services.

### Node Components
`sudo journalctl -u kubelet`
`sudo systemctl status kubelet`
`sudo systemctl enable kubelet`
`sudo systemctl start kubelet`

- kubectl get componentstatus
- kubectl config view

# 6. Troubleshoot networking
- [https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/]

- first step is to check the service status, `kubectl get service`
- for more information, describe the service, `kubectl describe service
  <service-name>`
- check if 'kube-proxy' is running, `sudo systemctl status kube-proxy`.
- check journald logs, `sudo journalctl logs kube-proxy`



