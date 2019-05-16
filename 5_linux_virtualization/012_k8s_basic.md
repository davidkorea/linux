

# 1. Understanding the architecture of a Kubernetes cluster
- The master node, which hosts the Kubernetes Control Plane that controls and manages the whole Kubernetes system
- Worker nodes that run the actual applications you deploy

  ![](https://i.loli.net/2019/05/16/5cdcc7c71a8c343699.png)

## 1.1 THE CONTROL PLANE - MASTER
The Control Plane is what controls the cluster and makes it function. It consists of multiple components that can run on a single master node or be split across multiple nodes and replicated to ensure high availability. These components are
### 1. etcd
a reliable distributed data store that persistently stores the cluster configuration.
### 2. Kubernetes API Server
The Kubernetes API Server which you and the other Control Plane components communicate with
### 3. Scheduler
The Scheduler which schedules your apps (assigns a worker node to each deployable component of your application)
### 4. Controller Manager
The Controller Manager which performs cluster-level functions, such as replicating components, keeping track of worker nodes, handling node failures, and so on. The single Controller Manager process currently combines a multitude of controllers performing various reconciliation tasks. Eventually those controllers will be split up into separate processes, enabling you to replace each one with a custom implementation if necessary. The list of these controllers includes:
- **Replication Manager** (a controller for ReplicationController resources)
  - ReplicationController could be thought of as an **infinite loop**无限循环, where in each iteration, the controller finds the number of pods matching its pod selector and compares the number to the desired replica count.
    ![](https://i.loli.net/2019/05/16/5cdd11a1f073128616.png)
- **ReplicaSet**, **DaemonSe**t, and **Job controllers**
- **Deployment controller
  - The Deployment controller takes care of keeping the actual state of a deployment in sync with the desired state specified in the corresponding Deployment API object.
- **StatefulSet controller**
- **Node controller**
  - Most resources belong to a specific namespace. When a Namespace resource is deleted, all the resources in that namespace must also be deleted. This is what the Namespace controller does. When it’s notified of the deletion of a Namespace object, it deletes all the resources belonging to the namespace through the API server
- **Service controller**
  -  a few different types exist. One of them was the 'LoadBalancer' service, which requests a load balancer from the infrastructure to make the service available externally. The Service controller is the one requesting and releasing a load balancer from the infrastructure, when a LoadBalancer-type Service is created or deleted.
- **Endpoints controller**
  - Services aren’t linked directly to pods, but instead contain a list of endpoints (IPs and ports), which is created and updated either manually or automatically according to the pod selector defined on the Service. 
  - The Endpoints controller is the active component that keeps the endpoint list constantly updated with the IPs and ports of pods matching the label selector.
    ![](https://i.loli.net/2019/05/16/5cdd11a5d413987461.png)
- **Namespace controller**
- **PersistentVolume controller**
- **Others**

Controllers never talk to each other directly. They don’t even know any other controllers exist. Each controller connects to the API server and, through the watch mechanism asks to be notified when a change occurs in the list of resources of any type the controller is responsible for.

## 1.2 THE NODES - WORKER
The worker nodes are the machines that run your containerized applications. The task of running, monitoring, and providing services to your applications is done by the following components: 
### 1. Docker, rkt, or another container runtime, which runs your containers
### 2. Kubelet
The Kubelet, which talks to the API server and manages containers on its node. Kubelet is the component responsible for everything running on a worker node.
- Its initial job is to register the node it’s running on by creating a Node resource in the API server. 
- Then it needs to continuously monitor the API server for Pods that have been scheduled to the node, and start the pod’s containers. It does this by telling the configured container runtime (which is Docker, CoreOS’ rkt, or something else) to run a container from a specific container image. 
- The Kubelet then constantly monitors running containers and reports their status, events, and resource consumption to the API server.
- RUNNING STATIC PODS WITHOUT THE API SERVER
  - Kubelet can also run pods based on pod manifest files in a specific local directory as shown in figure
  - This feature is used to run the containerized versions of the Control Plane components as pods. Instead of running Kubernetes system components natively, you can put their pod manifests into the Kubelet’s manifest directory and have the Kubelet run and manage them. You can also use the same method to run your custom system containers, but doing it through a DaemonSet is the recommended method.
  ![](https://i.loli.net/2019/05/16/5cdd1606a1fd960354.png)
### 3. Kubernetes Service Proxy (kube-proxy)
The Kubernetes Service Proxy (kube-proxy), which load-balances network traffic between application components.



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
