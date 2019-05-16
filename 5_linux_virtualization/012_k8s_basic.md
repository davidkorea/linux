

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

  ![](https://i.loli.net/2019/05/16/5cdcf39d00c5783414.png)

To help you visualize what transpired, look at figure. It shows both steps you had toperform to get a container image running inside Kubernetes. 
- First, you built the image and pushed it to Docker Hub. This was necessary because building the image on your local machine only makes it available on your local machine, but you needed to make it accessible to the Docker daemons running on your worker nodes.
- When you ran the kubectl command, it created a new ReplicationController object in the cluster by sending a REST HTTP request to the Kubernetes API server. The ReplicationController then created a new pod, which was then scheduled to one of the worker nodes by the Scheduler. 
- The Kubelet on that node saw that the pod was scheduled to it and instructed Docker to pull the specified image from the registry
because the image wasn’t available locally. After downloading the image, Docker created and ran the container.
- The other two nodes are displayed to show context. They didn’t play any role in the process, because the pod wasn’t scheduled to them.
