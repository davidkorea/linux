

# Understanding the architecture of a Kubernetes cluster
- The master node, which hosts the Kubernetes Control Plane that controls and manages the whole Kubernetes system
- Worker nodes that run the actual applications you deploy

## THE CONTROL PLANE
The Control Plane is what controls the cluster and makes it function. It consists of multiple components that can run on a single master node or be split across multiple nodes and replicated to ensure high availability. These components are
1. The Kubernetes API Server, which you and the other Control Plane components communicate with
2. The Scheduler, which schedules your apps (assigns a worker node to each deployable component of your application)
3. The Controller Manager, which performs cluster-level functions, such as replicating components, keeping track of worker nodes, handling node failures, and so on
4. etcd, a reliable distributed data store that persistently stores the cluster configuration.
