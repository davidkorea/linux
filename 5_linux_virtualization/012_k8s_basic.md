

# 1. Understanding the architecture of a Kubernetes cluster
- The master node, which hosts the Kubernetes Control Plane that controls and manages the whole Kubernetes system
- Worker nodes that run the actual applications you deploy

  ![](https://i.loli.net/2019/05/16/5cdcc7c71a8c343699.png)

### THE CONTROL PLANE - MASTER
The Control Plane is what controls the cluster and makes it function. It consists of multiple components that can run on a single master node or be split across multiple nodes and replicated to ensure high availability. These components are
1. The Kubernetes **API Server**, which you and the other Control Plane components communicate with
2. The **Scheduler**, which schedules your apps (assigns a worker node to each deployable component of your application)
3. The **Controller Manager**, which performs cluster-level functions, such as replicating components, keeping track of worker nodes, handling node failures, and so on
4. **etcd**, a reliable distributed data store that persistently stores the cluster configuration.
### THE NODES - WORKER
The worker nodes are the machines that run your containerized applications. The task of running, monitoring, and providing services to your applications is done by the following components: 
1. Docker, rkt, or another container runtime, which runs your containers
2. The **Kubelet**, which talks to the API server and manages containers on its node
3. The **Kubernetes Service Proxy (kube-proxy)**, which load-balances network traffic between application components



# 2. UNDERSTANDING WHAT HAPPENED BEHIND THE SCENES

![](https://i.loli.net/2019/05/16/5cdcf28883a4b99093.png)
-----
 ![](https://i.loli.net/2019/05/16/5cdcf39d00c5783414.png)

To help you visualize what transpired, look at figure. It shows both steps you had toperform to get a container image running inside Kubernetes. 
- First, you built the image and pushed it to Docker Hub. This was necessary because building the image on your local machine only makes it available on your local machine, but you needed to make it accessible to the Docker daemons running on your worker nodes.
- When you ran the kubectl command, it created a new ReplicationController object in the cluster by sending a REST HTTP request to the Kubernetes API server. The ReplicationController then created a new pod, which was then scheduled to one of the worker nodes by the Scheduler. 
- The Kubelet on that node saw that the pod was scheduled to it and instructed Docker to pull the specified image from the registry
because the image wasn’t available locally. After downloading the image, Docker created and ran the container.
- The other two nodes are displayed to show context. They didn’t play any role in the process, because the pod wasn’t scheduled to them.

**With your pod running, how do you access it?**

We mentioned that each pod gets itsown IP address, but this address is internal to the cluster and isn’t accessible from outside of it. To make the pod accessible from the outside, you’ll expose it through a Service object. 
- You’ll create a **special service** of type LoadBalancer, because if you create a **regular service** (a ClusterIP service), like the pod, it would also only be accessible from inside the cluster. 
- By creating a LoadBalancer-type service, an external load balancer will be created and you can connect to the pod through the load balancer’s public IP.

## 2.1 UNDERSTANDING HOW THE REPLICATIONCONTROLLER, THE POD, AND THE SERVICE FIT TOGETHER
![](https://i.loli.net/2019/05/16/5cdcf50b7616568882.png)

As already explained, you’re not creating and working with containers directly. Instead, the basic building block in Kubernetes is the pod. But, you didn’t really create any pods either, at least not directly. 
- By running the `kubectl run` command you created a ReplicationController, and this ReplicationController is what created the actual Pod object. 
- To make that pod accessible from outside the cluster, you told Kubernetes to expose all the pods managed by that ReplicationController as a single Service. 

A rough picture of all three elements is presented in figure

### POD AND ITS CONTAINER
The main and most important component in your system is the pod. It contains only a single container, but generally a pod can contain as many containers as you want.

### REPLICATIONCONTROLLER
The next component is the ReplicationController. It makes sure there’s always exactly one instance of your pod running. Generally, ReplicationControllers are used to replicate pods (that is, create multiple copies of a pod) and keep them running. If your pod were to disappear for any reason, the ReplicationController would create a new pod to replace the missing one.

### SERVICE
The third component of your system is the kubia-http service. To understand why you need services, you need to learn a key detail about pods. They’re ephemeral短暂的. Apod may disappear at any time—because the node it’s running on has failed, because someone deleted the pod, or because the pod was evicted from an otherwise healthy node. When any of those occurs, 
  - a missing pod is replaced with a new one by the ReplicationController, as described previously. 
  - This new pod gets a different IP address from the pod it’s replacing. 

This is where services come in—to solve the problem of ever-changing pod IP addresses, as well as exposing multiple pods at a single constant IP and port pair. When a service is created, it gets a static IP, which never changes during the lifetime of the service.
- Instead of connecting to pods directly, clients should connect to the service through its constant IP address. 
- The service makes sure one of the pods receives the connection, regardless of where the pod is currently running (and what its IP address is).

Services represent a static location for a group of one or more pods that all provide the same service. Requests coming to the IP and port of the service will be forwarded to the IP and port of one of the pods belonging to the service at that moment.

## 2.2 Horizontally scaling the application
```
$ kubectl scale rc kubia --replicas=3
replicationcontroller "kubia" scaled
```
- Notice that you didn’t instruct Kubernetes what action to take. You didn’t tell it to add two more pods. 
- You only set the new desired number of instances and let Kubernetes determine what actions it needs to take to achieve the requested state.
```
$ kubectl get pods
NAME READY STATUS RESTARTS AGE
kubia-hczji 1/1 Running 0 7s
kubia-iq9y6 0/1 Pending 0 7s
kubia-4jfyf 1/1 Running 0 18m
```
Because you now have multiple instances of your app running, let’s see what happens if you hit the service URL again. Will you always hit the same app instance or not?
```
$ curl 104.155.74.57:8080
You’ve hit kubia-hczji
$ curl 104.155.74.57:8080
You’ve hit kubia-iq9y6
$ curl 104.155.74.57:8080
You’ve hit kubia-iq9y6
$ curl 104.155.74.57:8080
You’ve hit kubia-4jfyf
```
Requests are hitting different pods randomly. This is what services in Kubernetes do when more than one pod instance backs them. They act as a **load balancer** standing in front of multiple pods. 
- When there’s only one pod, services provide a static address for the single pod. 
- Whether a service is backed by a single pod or a group of pods, those pods come and go as they’re moved around the cluster, which means their IP addresses change, but the service is always there at the same address. 
- This makes it easy for clients to connect to the pods, regardless of how many exist and how often they change location.

![](https://i.loli.net/2019/05/16/5cdcfcc5712d825144.png)
