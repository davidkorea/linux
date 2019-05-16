

# Understanding the architecture of a Kubernetes cluster
- The master node, which hosts the Kubernetes Control Plane that controls and manages the whole Kubernetes system
- Worker nodes that run the actual applications you deploy

  ![](https://i.loli.net/2019/05/16/5cdcc7c71a8c343699.png)

## THE CONTROL PLANE - MASTER
The Control Plane is what controls the cluster and makes it function. It consists of multiple components that can run on a single master node or be split across multiple nodes and replicated to ensure high availability. These components are
1. The Kubernetes **API Server**, which you and the other Control Plane components communicate with
2. The **Scheduler**, which schedules your apps (assigns a worker node to each deployable component of your application)
3. The **Controller Manager**, which performs cluster-level functions, such as replicating components, keeping track of worker nodes, handling node failures, and so on
4. **etcd**, a reliable distributed data store that persistently stores the cluster configuration.
## THE NODES - WORKER
The worker nodes are the machines that run your containerized applications. The task of running, monitoring, and providing services to your applications is done by the following components: 
1. Docker, rkt, or another container runtime, which runs your containers
2. The **Kubelet**, which talks to the API server and manages containers on its node
3. The **Kubernetes Service Proxy (kube-proxy)**, which load-balances network traffic between application components
