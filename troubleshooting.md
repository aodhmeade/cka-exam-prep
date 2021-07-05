
| ** Troubleshooting 30%**                   |   |
|--------------------------------------------|---|
| 1.  Evaluate cluster and node logging      |   |
| 2.  Understand how to monitor applications |   |
| 3.  Manage container stdout & stderr logs  |   |
| 4.  Troubleshoot application failure       |   |
| 5.  Troubleshoot cluster component failure |   |
| 6.  Troubleshoot networking                |   |

Seems to make more sense to group the sections as follows:

1 and 6: cover Kubernetes Components 
    - Control Plane Components: kube-apiserver, etcd, kube-scheduler,
    - kube-controller-manager
    - Node Components: kubelet, kube-proxy, container runtime
    - Addons: namely DNS

2, 3 and 4: cover Applications

6: cover Networking (Services CNI)

Some observations:
Kubernetes relies on API calls and is sensitive to network issues.  Therefore,
standard linux tools (e.g. dig, tcpdump) can be very useful in troubleshooting your cluster.

# ** 1.  Evaluate cluster and node logging  **
## Node level logging architecture:
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

## Cluster level logging architectures:
- Kubernetes does not provide an out-of-the-box solution for cluser-level
  logging.
- Common options include:
    - use a node-level logging agent that runs on every node.
    - include a dedicated sidecar container for logging in an application pod.
    - push logs directly to a backend from within the application.

# ** 2.  Understand how to monitor applications **
```
kubectl exec --stdin --tty <pod-name> -- /bin/bash # if only one container
running

kubectl exec -i -t <pod-name> --container <container-name> -- /bin/bash # if pod
has more than one container. Note: the short options -i and -t are the same as
the long options --stdin and --tty.

exit # enter 'exit' when you are finished with the shell

```

# ** 3.  Manage container stdout & stderr logs  **




# ** 4.  Troubleshoot application failure **
Inevitably, you will need to debug problems with your application.
```
# to start, check if a pod is running
kubectl get pods

# to get more specific information from a pod, use describe
kubectl describe pod <pod-name>

# to get extra detail, pass the -o yaml output format flag to the kubectl get
# pod command
kubectl get pod <pod-name> -o yaml

# to obtain the stdout & stderr logs from the containers in the pod (the level of detail will
# depend on how the application has been configured.
kubectl logs <pod-name>

# to check the logs of the previous pod
kubectl logs <pod-name> -f --previous

```


# ** 5.  Troubleshoot cluster component failure **
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
Addons

DNS

Web UI (Dashboard)

Container Resource Monitoring
```
# first thing to check is if all cluster nodes are registered correctly and in a ready state
kubectl get nodes

# to get more information on a node
kubectl describe node <node-name>

# to get extra detail on a node, pass the output to yaml
kubectl get node <node-name> -o yaml

# check for event errors with
kubectl get events --namespace=<namespace-name>
kubectl get events --all-namespaces

# to get information about the overall health of a cluster, you can run
kubectl cluster-info dump

# 
service kube-apiserver status
service kube-controller-manager status
service kube-scheduler status

```

```
** kube-apiserver **
# for systemd based
journalctl -u kube-apiserver
or
cat /var/log/kube-apiserver.log
```

```
** kube-scheduler **
# for systemd based
journalctl -u kube-scheduler
or
cat /var/log/kube-scheduler.log
```

```
** kube-controller-manager **
# for systemd based
journalctl -u kube-controller-manager
or
cat /var/log/kube-controller-manger.log
```



The next step would be to look deeper into the relevant machines hosting the
services.




# ** 6.  Troubleshoot networking  **

```
# first step is to check the service status
kubectl get service

# for more information, describe the service
kubectl describe service <service-name>
```
