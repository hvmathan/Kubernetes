Replication Controller and Replication Set in Kubernetes Deployment
Harsha Mathan
Harsha Mathan

7 min read
·
Just now





In Kubernetes, there are situations where you might need to scale workloads or update configurations across all running pods. This can be achieved seamlessly using Kubernetes Deployment. In this blog, we will explore how to deploy changes using both the Replication Controller (legacy) and Replica Set (current). This is the 8th blog in the #40DaysofKubernetes series by Piyush Sachdeva. For further details, refer to the embedded video at the end of this blog.


Typical workload being shared across available Pods using rc
Typical Workload Managed by Replication Controller
A Replication Controller ensures that a specified number of pod replicas are running at any given time. If one of the pods fails, the Replication Controller will either create a new pod on the existing node or on a new node if the current node has reached its maximum capacity. In the example below, we have defined replicas: 3, which creates 3 identical pods.

Creating the pods via yaml file

apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
  labels:
    env: demo
spec:
  replicas: 3
  template:
    metadata:
      labels:
        env: demo
    spec:
      containers:
      - image: nginx
        name: nginx
  replicas: 3
This YAML configuration creates 3 pods. Here are some key commands to manage these resources:

kubectl apply -f rc.yml → Deploy the YAML file.
kubectl get po → List the pods.
kubectl get rc → List the Replication Controllers.
kubectl describe po nginx-rc-jdrqx → Describe the pod in detail.
This creates 3 pods as below


Replica Set :

Replica Sets are the current preferred approach for managing pods and scaling workloads. To migrate from Replication Controller to Replica Set, update your YAML file with the following changes:

  selector:
    matchLabels:
      env: demo
Ensure to update Group version & Kind for Replica Set in yaml file as below

apiVersion: apps/v1
kind: ReplicaSet
Now, we can see there are 6 pods running. 3 for replication contoller and 3 for replicaset.


Thats it, we can play around now for example — if you want to increase the number of replication set, update the yaml file which is broadly termed as manifest file.

we can also modify in the live object by using the below command.

kubectlrs/nginx-rs
Next option is to use ‘kubectl’ in imperative way

kubectl scale --replicas=10 rs/nginx-rs

Scaling the pods using imperative way
Deployment
Deployment is a higher-level abstraction that manages Replica Sets, which in turn manage pods. It supports rolling updates, ensuring that changes are applied with no downtime and providing the ability to roll back to previous versions.


This has the feasibly to amend any variable in single shot ensuring no downtime when change is happening in one of the pods.

The deployment happens in rolling update fashion ie. when change is happening to one pod, the traffic will be shifted to the other pods.

Also, this helps to rollback to a particular version if required.

Change the below in yml file for Deployments.

kind: Deployment
Upon executing the yml file, we can see deployment pods running similar to how rs was running.

Use the below command to check all the artifacats running

kubectl get all

get all
We can update an image one single shot as below.


when you perform a Kubectl describe, we can notice the image is being updated but it is to be noted that it is not updated in the local yml file.

Perform the rollback using the below command.

kubectl rollout history deploy/nginx-deploy
To rollback:

kubectl rollout undo deploy/nginx-deploy
Now when we describe the image, we can can the first version that is nginx and not nginx 1.9.1..

We can create a yaml file from command line similar to how we have done in the past by redirecting the bleow command to a yaml file..

kubectl create deploy deploy/nginx-new --image=nginx --dry-run=client 
this will create a yaml file as below..

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: deploy/nginx-new
  name: deploy/nginx-new
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deploy/nginx-new
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deploy/nginx-new
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
Tasks from https://github.com/piyushsachdeva/CKA-2024/blob/main/Resources/Day08/task.md

#1. Create a new Replicaset based on the nginx image with 3 replicas

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-rs
  labels:
    env: demo
spec:
  replicas: 3
  template:
    metadata:
      labels:
        env: demo
    spec:
      containers:
      - image: nginx
        name: nginx
  replicas: 3
  selector:
    matchLabels:
      env: demo

3 rs pods up and running…
#2. Update the replicas to 4 from the YAML - update the below in the above yml

replicas: 4

4 pods up and running
#3. Updating via cmd line

kubectl scale --replicas=6 rs/nginx-rs-85f776c449

6 pods after changing them iteratively.
Deployment — Task :
Create a Deployment named nginx with 3 replicas. The Pods should use the nginx:1.23.0 image and the name nginx. The Deployment uses the label tier=backend. The Pod template should use the label app=v1.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    tier: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: v1
      tier: backend
  template:
    metadata:
      labels:
        app: v1
        tier: backend
    spec:
      containers:
      - name: nginx
        image: nginx:1.23.0
2. List the Deployment and ensure the correct number of replicas is running.

kubectl get deployments
3. Update the image to nginx:1.23.4

    spec:
      containers:
      - name: nginx
        image: nginx:1.23.4
4. Verify that the change has been rolled out to all replicas.

kubectl rollout status deployment/nginx
kubectl rollout status deployment/nginx

Image updated
5. Assign the change cause “Pick up patch version” to the revision.

kubectl annotate deployment/nginx kubernetes.io/change-cause="Pick up patch version"
Validate :


Annotation updated
6. Scale the Deployment to 5 replicas.


Scaling to 5
7. Have a look at the Deployment rollout history.

kubectl rollout history deployment/nginx
8.Revert the Deployment to revision 1.

(base) harshavigneshmathan@Harshavigneshs-MacBook-Air Day08 % kubectl rollout undo deployment/nginx --to-revision=1

deployment.apps/nginx rolled back
(base) harshavigneshmathan@Harshavigneshs-MacBook-Air Day08 % 
9. Ensure that the Pods use the image nginx:1.23.0.

kubectl set image deployment/nginx nginx=nginx:1.23.0
base) harshavigneshmathan@Harshavigneshs-MacBook-Air Day08 % kubectl describe deployment/nginx

Name:                   nginx
Namespace:              default
CreationTimestamp:      Sat, 13 Jul 2024 20:31:15 +0530
Labels:                 tier=backend
Annotations:            deployment.kubernetes.io/revision: 3
Selector:               app=v1,tier=backend
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=v1
           tier=backend
  Containers:
   nginx:
    Image:         nginx:1.23.0
Troubleshooting :

We get the below error upon running the this sample yaml.
apiVersion: v1
kind:  Deployment
metadata:
  name: nginx-deploy
  labels:
    env: demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
  replicas: 3
  selector:
    matchLabels:
      env: demo
(base) harshavigneshmathan@Harshavigneshs-MacBook-Air Day08 % kubectl apply -f troubleshoot1.yml 
error: resource mapping not found for name: "nginx-deploy" namespace: "" from "troubleshoot1.yml": no matches for kind "Deployment" in version "v1"
ensure CRDs are installed first
(base) harshavigneshmathan@Harshavigneshs-MacBook-Air Day08 % 
To fix this —check the deployment API Version the apiVersion should have apps/v1

Troubleshooting 2 :

apiVersion: v1
kind:  Deployment
metadata:
  name: nginx-deploy
  labels:
    env: demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
  replicas: 3
  selector:
    matchLabels:
      env: dev
similar to the previous one, we have to set the right api version. Another error is around

base) harshavigneshmathan@Harshavigneshs-MacBook-Air Day08 % kubectl apply -f troubleshooting2.yml
The Deployment "nginx-deploy" is invalid: 
* spec.template.metadata.labels: Invalid value: map[string]string{"env":"demo"}: `selector` does not match template `labels`
* spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string{"env":"dev"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: field is immutable
(base) harshavigneshmathan@Harshavigneshs-MacBook-Air Day08 % 
ERROR:
(base) harshavigneshmathan@Harshavigneshs-MacBook-Air Day08 % kubectl apply -f troubleshooting2.yml
The Deployment "nginx-deploy" is invalid: spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string{"app":"v1"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: field is immutable
Here we have an error because because you cannot change the selector field of a Deployment after it has been created. This field is immutable. For this, we have delete and redeploy the yaml again

kubectl delete deployment nginx-deploy
Final corrected version:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    tier: backend  # Updated to reflect the required label
spec:
  replicas: 3  # Number of replicas
  selector:
    matchLabels:
      app: v1  # Correct selector to match the Pod labels
  template:
    metadata:
      labels:
        app: v1  # Labels should match the selector
    spec:
      containers:
      - name: nginx
        image: nginx:1.23.0  # Specified version of the nginx image
        ports:
        - containerPort: 80
This concludes the 8th blog in this series. Please refer the below video by Piyush Sachdeva which is used as a reference for this blog.


#Deployment #40DaysofKubernetes.