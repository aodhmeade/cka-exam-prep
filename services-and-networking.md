| **Services and Networking  20%**                                             |
|------------------------------------------------------------------------------|
| 1.  Understand host networking configuration on the cluster nodes            |
| 2.  Understand connectivity between Pods                                     |
| 3.  Understand ClusterIP, NodePort, LoadBalancer service types and endpoints |
| 4.  Know how to use Ingress controllers and Ingress resources                |
| 5.  Know how to configure and use CoreDNS                                    |
| 6.  Choose an appropriate container network interface plugin                 |


[https://cloud.google.com/kubernetes-engine/docs/concepts/network-overview]
[https://github.com/IBM/kubernetes-networking/blob/master/pdf/KubernetesNetworking-Lecture.pdf]


**Note**
The Kubernetes networking model relies heavily on IP addresses.  Services, Pods,
containers, and nodes communicate using IP addresses and ports.  Kubernetes
provides different types of load balancing to direct traffic to the correct
Pods.

**Terms**
- ClusterIP: The IP address assigned to a Service. In other documents, it may be
  called the "Cluster IP". This address is stable for the lifetime of the
  Service, as discussed in the Services section in this topic.

- Pod IP: The IP address assigned to a given Pod. This is ephemeral, as
  discussed in the Pods section in this topic.

- Node IP: The IP address assigned to a given node.

- In Kubernetes, you can assign arbitrary key-value pairs called labels to any
  Kubernetes resource. Kubernetes uses labels to group multiple related Pods
  into a logical unit called a Service. A Service has a stable IP address and
  ports, and provides load balancing among the set of Pods whose labels match
  all the labels you define in the label selector when you create the Service.

- Kubernetes manages connectivity among Pods and Services using the kube-proxy
  component. This is deployed as a static Pod on each node by default.

- When you configure a Service, you can optionally remap its listening port by
  defining values for port and targetPort.

    - The port is where clients reach the application.
    - The targetPort is the port where the application is actually listening for
      traffic within the Pod.










# **1.  Understand host networking configuration on the cluster nodes**











# **2.  Understand connectivity between Pods**
[https://kubernetes.io/docs/concepts/cluster-administration/networking/]

Kubernetes tries to address 4 distinct networking problems:

1. Highly-coupled container-to-container communications: this is solved by Pods and
localhost communications.
2. Pod-to-Pod communicationn: 
3. Pod-to-Service communications: this is covered by services.
4. External-to-Service communications: this is covered by services.

Kubernetes imposes the following fundamental requirements on any networking
implementation (barring any intentional network segmentation policies):

- pods on a node can communicate with all pods on all nodes without NAT
- agents on a node (e.g. system daemons, kubelet) can communicate with all pods on
that node

Note: For those platforms that support Pods running in the host network (e.g.
Linux):

- pods in the host network of a node can communicate with all pods on all nodes
without NAT

There are a number of ways this networking model can be implemented.  See
section 6 below.


















n ** 3.  Understand ClusterIP, NodePort, LoadBalancer service types and
# endpoints **

## ClusterIP
- this is the default approach when creating a Kubernetes service.  The service
  is allocated an internal IP that other components can use to access the pods.

### Target Port
- allows you to separate the port the service is available on from the port the
  application is listening on

## Node Port
While the TargetPort and ClusterIP make the pod available to inside the cluster,
the NodePort exposes the service on each Node's IP via the defined static port.

## External IPs


## LoadBalancer











# **4.  Know how to use Ingress controllers and Ingress resources**      

[https://kubernetes.io/docs/concepts/services-networking/ingress/] 
[https://www.ibm.com/cloud/blog/kubernetes-ingress]

- Ingress is another way for Kubernetes to tackle routing.

- Kubernetes Ingress is an API object that provides routing rules to manage
  external users' access to the services in a Kubernetes cluster, typically via
  http and https.  Traffic routing is controlled by rules defined on the Ingress
  resource.

  The difference between Services (clusterIP, NodePort, LoadBalancer) and
  Ingress is in the level of efficiency.  Instead of using multiple Services,
  you can route traffic based on the request host or path which allows for
  centralisation of many services to a single point.

- Kubernetes Ingress is an API object that describes the desired state for
  exposing services to the outside of the Kubernetes cluster.  An Ingress
  Controller is the actual implementation of the Ingress API. An Ingress
  Controller reads and processes the Ingress Resource information and usually
  runs as a daemon in a Pod, which watches the /ingresses endpoint on the API
  server, which is found under the 'networking.k8s.io/v1beta1' group for new
  objects.

- An Ingress provides the following:
    - externally reachable URLs for applications deployed in Kubernetes clusters
    - name-based virtual host and URI-based routing support
    - load balancing rules and traffic, as well as SSL termination

- Ingress Controllers are built using reverse proxies.  The proxy gives it Layer
  7 of the OSI model routing and load balancing capabilities.

- This load balancer can be a software load balancer running in a cluster or
  hardware-based, or a cloud load balancer running externally. Different load
  balancers require different Ingress Controller implementations.  There are
  various available on the market.  Kubernetes as a project supports and
  maintains AWS, GCE, and nginx ingress controllers.  They are other offerings
  on the market too.

- The Ingress Controller is responsible for reading the Ingress Resource
  information and processing it.  Being inside the cluster themselves, you need
  to expose them to the outside via a Service with a type of either NodePort or
  LoadBalancer. Service --> Ingress --> Pod(s)

- The Ingress Controller inspects HTTP requests and based on the characteristics
  it finds, such as URL path or domain name, it directs a client to the correct
  Pod.

- For exam: know how to set up an Ingress Resource and Ingress Controller.

- Sample Ingress Resource manifest:
'''
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
spec:
  rules:
  - http:
      paths:
      - path: /test-path-service-one
        backend:
          serviceName: service-one
          servicePort: 80
  - http:
      paths:
      - path: /test-path-service-two
        backend:
          serviceName: service-two
          servicePort: 80
'''

- You can manage ingress resources like you do Pods, Services, etc.:
'''
$ kubectl get ingress
$ kubectl describe ingress <ingress-name>
$ kubectl edit ingress <ingress-name>
$ kubectl delete ingress <ingress-name>
'''
- To deploy an Ingress Controller, you can create one with 'kubectl'.

- You can use Helm to install an ingress controller:
'''
helm search hub ingress
'''

- Test the host by using curl

# **5.  Know how to configure and use CoreDNS**
[https://coredns.io/]

- Kubernetes creates DNS records for Services and Pods. You can contact services
  with consistent DNS names instead of IP addresses.
- Kubernetes DNS schedules a DNS Pod and Service on the cluster, and configures
  the kubelets to tell individual containers to use the DNS Services's IP to
  resolve DNS names.
- CoreDNS is an open source DNS server, written in Go, and hosted by CNCF.
- It can act as the Kubernetes cluster DNS, as an alternative to kube-dns (since
  v1.12).
- Troubleshooting: use tools such as dig, nslookup.  Check /etc/resolv.conf of
  the container.

```
kubectl get pods -n kube-system

kubectl exec -it <pod-name> -- /bin/bash

apt-get update; apt-get install curl dnsutils -y

dig

cat /etc/resolv.conf

kubectl -n kube-system get svc kube-dns -o yaml

```

- CoreDNS is modular and pluggable - each plugin adds new functionality to
  CoreDNS. This can be configured by maintaining a Corefile (which is CoreDNS'
  configuration file.  It defines:
    - what servers listen on what ports and which protocol
    - for which zone each servier is authoritative
    - which plugins are loaded in a server
- In Kubernetes, you modify the ConfigMap for the CoreDNS Corefile to change how
  DNS service discovery behaves for the cluster.
- To view/modify the ConfigMap:

```
kubectl -n kube-system get configmap coredns -o yaml
```


# **6.  Choose an appropriate container network interface plugin**

[https://github.com/containernetworking/cni]
[https://www.redhat.com/sysadmin/cni-kubernetes]
[https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy]
[https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/]

- To provide container networking, Kubernetes uses a CNI plugin.  It is
  mandatory for the cluster to be operational.

- From kubernetes.io (Installing a Pod network add-on): 'You must deploy a
  Container Network Interface (CNI) based Pod network add-on so that your Pods
  can communicate with each other. Cluster DNS (CoreDNS) will not start up
  before a network is installed.'

- Its role is to set up network connectivity so Pods can communicate with each
  other.

- CNI is a separate framework.  From github: 'CNI (Container Network Interface),
  a Cloud Native Computing Foundation project, consists of a specification and
  libraries for writing plugins to configure network interfaces in Linux
  containers, along with a number of supported plugins. CNI concerns itself only
  with network connectivity of containers and removing allocated resources when
  the container is deleted. Because of this focus, CNI has a wide range of
  support and the specification is simple to implement.'

- CNI is used by other container runtimes - Podman, CRI-O, Mesos, OpenShift, etc.

- A CNI plugin is responsible for:
       - inserting a network interface into the container network namespace
       - assigning an IP to this interface
       - removing the interface when the container is deleted

- In the CNI architecture, the container runtime calls the CNI plugin with verbs
  such as 'ADD, DEL, CHECK, VERSION (and more ...)'. For example, ADD creates a new
  network interface for the container, and details of what is to be added are
  passed to CNI via JSON.
 
- Note: Kubernetes also uses kubenet, which is not a CNI plugin.  It has a
  limited set of functionalities.  If you use a public cloud provider and one of
  their managed Kubernetes services, the CNI is already chosen for you.

- There are a number of 3rd-party network plugins implementing the CNI
  specification.  Examples: Calico, Flannel, WeaveNet.

- The CNI plugin is normally run as a daemonset - as a Pod. Once the plugin is
  installed, a network config file is created at /etc/cni/net.d  The CNI plugin
  referenced in this file should be located in /opt/cni/bin.  Kubelet is then
  responsible for reading the config file and invoking the relevant plugin
  binary in order to set up the network for each Pod.

- Get a CNI manifest through kubernetes.io

- Apply with 'kubectl apply -f <name.yaml>'

'''
$ systemctl status kubelet.service
$ ps -aux | grep kubelet
$ ls /etc/cni/net.d/
$ ls /opt/cni/bin
''' 



