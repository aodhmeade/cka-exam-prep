| **Services and Networking  20%**                                             |
|------------------------------------------------------------------------------|
| 1.  Understand host networking configuration on the cluster nodes            |
| 2.  Understand connectivity between Pods                                     |
| 3.  Understand ClusterIP, NodePort, LoadBalancer service types and endpoints |
| 4.  Know how to use Ingress controllers and Ingress resources                |
| 5.  Know how to configure and use CoreDNS                                    |
| 6.  Choose an appropriate container network interface plugin                 |




1.  Understand host networking configuration on the cluster nodes            

2.  Understand connectivity between Pods                                     

# ** 3.  Understand ClusterIP, NodePort, LoadBalancer service types and
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

4.  Know how to use Ingress controllers and Ingress resources                

An Ingress enables inbound connections to the cluster, allowing external traffic
to reach the correct Pod.  Ingress enables externally-reachable urls, load balance traffic, terminate SSL,
offer name based virtual hosting for a Kubernetes cluster.



5.  Know how to configure and use CoreDNS                                    

6.  Choose an appropriate container network interface plugin                 
