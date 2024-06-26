Welcome to the 5th blog in the #40DaysofKubernetes series! After exploring the basics of containers, it’s time to dive into Kubernetes. In this blog, we’ll cover Kubernetes’ architecture and its core components. I have also demonstrated a simple lifecycle of a command when issued by the user.

This guide is inspired by Piyush Sachdeva excellent explanation in the embedded video.



Kubernetes Architecture
Control Plane (Master Node)
The Control Plane manages the Kubernetes cluster and consists of several key components:

- API Server:The entry point for all REST commands used to control the cluster. It processes and validates REST requests, and updates the state of the objects in ETCD.
- Scheduler: Assigns workloads to specific nodes based on resource availability.
- ControllerManager: Runs various controllers to handle routine tasks, such as node monitoring and lifecycle events.
- etcd: A distributed key-value store used to persist cluster state and configuration data. This is similar to control table in Database architecture

Control Plane :
Basically used for administrative purpose.

The request from the Client is sent to the APIServer and it moves from there on. This is the entry point.

Scheduler — Helps to schedule the workflow as it recevies request from user

Controller Manager — Combination of many controllers (node controller, namespace controller etc..). This keeps monitoring and ensures that everything is up and running

ETCD — A simple JSON that store key value pair of requests / information from APIServer. In database terms, this is the control table.

Worker Node
Worker nodes are responsible for running the actual applications and consist of:

- Kubelet: Communicates with the Control Plane, ensuring containers are running in a Pod. It receives instructions from the API Server and executes them.
- Kube-proxy: Manages network rules, allowing communication to your Pods from network sessions inside or outside of your cluster.

Pods
Pods are the smallest deployable units in Kubernetes. A Pod can hold one or more containers that share the same network namespace and storage.

- Lifecycle of a Kubectl Command: When you issue a `kubectl` command to create a Pod, the API Server processes the request, stores the information in etcd, and the Scheduler assigns the Pod to a suitable node. The Kubelet on that node then ensures the Pod and its containers are running as specified.


Life-cycle of Kubectl command
Thanks for reading! If you have any questions, feel free to ask. Happy to help!
