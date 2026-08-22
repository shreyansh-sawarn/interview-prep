# deploy docker container to kubernetes

To deploy a Docker container to Kubernetes, you must build your Docker image, push it to a centralized container registry, write a Kubernetes Deployment manifest, and apply it to your cluster using the kubectl tool. Kubernetes does not run raw Docker containers directly; instead, it coordinates pods that pull your predefined Docker images from a registry.
Here is the complete step-by-step workflow to get your Docker application running on Kubernetes.
## 📋 Prerequisites

* Docker installed locally.
* Kubectl command-line tool installed.
* Access to a Kubernetes Cluster (e.g., [Docker Desktop Kubernetes](https://www.docker.com/blog/how-to-set-up-a-kubernetes-cluster-on-docker-desktop/), Minikube, or cloud providers like AWS EKS / Google GKE).
* An account on a Container Registry (e.g., [Docker Hub](https://hub.docker.com/) or Amazon ECR).

------------------------------
## Step 1: Build and Tag Your Docker Image
Before Kubernetes can use your application, it must be packaged into a Docker image with a distinct version tag.

   1. Navigate to the directory containing your Dockerfile.
   2. Build and tag the image using your registry username:
   
   docker build -t your-registry-username/myapp:v1.0.0 .
   

(Note: Avoid using the :latest tag for production workloads, as it creates ambiguity during automated rollbacks.)
## Step 2: Push the Image to a Registry
Kubernetes nodes need access to an online registry to download and run your container image. 

   1. Log in to your registry from your terminal:
   
   docker login
   
   2. Push your freshly built image:
   
   docker push your-registry-username/myapp:v1.0.0
   

## Step 3: Create the Kubernetes Manifest
Kubernetes relies on declarative YAML files to define your application's desired state. Create a file named deployment.yaml and paste the following configuration:

apiVersion: apps/v1kind: Deploymentmetadata:
  name: myapp-deployment
  labels:
    app: myappspec:
  replicas: 3 # Tells Kubernetes to run 3 identical instances of your container
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: your-registry-username/myapp:v1.0.0 # Your hosted Docker image
        ports:
        - containerPort: 8080 # The port your application listens on inside the container

## Step 4: Deploy to the Kubernetes Cluster
Use the kubectl CLI to pass your manifest file to your active cluster.

   1. Apply the configuration manifest:
   
   kubectl apply -f deployment.yaml
   
   2. Verify that your deployment was successfully created:
   
   kubectl get deployments
   
   3. Check the real-time status of your running pods:
   
   kubectl get pods
   

## Step 5: Expose Your App to Traffic (Optional)
By default, your deployment is only accessible internally within the cluster. To open it to external traffic, create a Service manifest named service.yaml: 

apiVersion: v1kind: Servicemetadata:
  name: myapp-servicespec:
  type: LoadBalancer # Provisions an external IP address to access your app
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80         # Port accessible from outside the cluster
      targetPort: 8080 # Routes traffic directly to your container's internal port

Apply this service manifest to finalize the network routing:

kubectl apply -f service.yaml

You can discover your external IP address by running kubectl get services.
------------------------------

# what does etcd do? what comprises master node? does worker nodes only have pods?

## What does etcd do?
etcd is the primary database of a Kubernetes cluster. It is a highly available, distributed, and consistent key-value store. 

* Stores Cluster State: Every configuration file, manifest, secret, and policy is saved inside etcd.
* Tracks Live Data: It keeps track of the real-time status of every node, pod, and container running in the cluster.
* Single Source of Truth: If something is not written to etcd, it does not exist in the cluster.
* Enables Recovery: If the cluster crashes, you can completely restore it using a backup of the etcd data. 

------------------------------
## What comprises the Master Node (Control Plane)?
The master node acts as the "brain" of the Kubernetes cluster. It runs components that make global decisions, detect events, and respond to cluster changes. It comprises four core components:

* kube-apiserver: The entry point and communication hub for the entire cluster. It exposes the Kubernetes API and validates all commands (like kubectl apply).
* etcd: The key-value store that acts as the cluster's database (as described above). 
* kube-scheduler: The matchmaker. It looks at newly created pods and decides which worker node they should run on based on available resources.
* kube-controller-manager: The regulator. It runs background loops to ensure the current state of the cluster matches your desired state (e.g., if a node goes down, it notices and orders a pod replacement). 

------------------------------
## Do Worker Nodes only have Pods?
No. While running pods is the main purpose of a worker node, the node itself must run several critical system utilities to manage those pods and communicate with the master node.  
Every worker node contains three essential system components alongside your user pods:
* Kubelet: An agent tracking tool. It acts as the "captain" of the node, taking instructions from the master node to ensure that the containers listed in the pods are running and healthy.
* Kube-proxy: The network supervisor. It runs on each node to maintain network rules, allowing your pods to talk to each other and handle network traffic from the outside world. 
* Container Runtime: The engine. This is the underlying software that actually runs the containers (e.g., containerd or CRI-O; note that Kubernetes uses these standard runtimes instead of direct Docker engines now). 

------------------------------

# why do pods go down?

Pods go down because Kubernetes detects a failure, resource shortage, or explicit configuration mismatch. When a pod fails, it enters a specific status phase (like Failed, Evicted, or CrashLoopBackOff) which points directly to the underlying cause.
The primary reasons pods go down can be categorized into four main areas.
## 1. Application and Code Runtime Errors
The software running inside your container crashes, causing the pod to fail.

* CrashLoopBackOff: The container starts, fails, restarts, and fails again in a continuous loop. This is usually caused by code bugs, missing environment variables, unhandled exceptions, or database connection failures. 
* Health Check Failures: If you configure Liveness Probes, Kubernetes regularly checks if your app is alive. If your app freezes, deadlocks, or stops responding, Kubernetes will intentionally kill the pod and start a new one.

## 2. Resource Starvation (The OOMKilled Error)
The node runs out of physical hardware resources to support the pod.

* OOMKilled (Exit Code 137): Out Of Memory. Your container tried to consume more RAM than the limits set in your YAML manifest, or the entire worker node ran out of RAM, forcing the operating system to terminate the pod to save the node. 
* CPU Throttling: While CPU shortages rarely kill a pod outright, extreme starvation can make the app so slow that it fails its health check probes and gets killed. 

## 3. Cluster Maintenance and Evictions
The master node intentionally shuts down the pod due to structural cluster changes.

* Node Pressure Eviction: If a worker node runs dangerously low on disk space or memory, the master node will evict lower-priority pods and reschedule them on healthier nodes.
* Cluster Scaling or Upgrades: During automated node upgrades or cloud [Horizontal Pod Autoscaling (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/), Kubernetes gracefully terminates pods to balance the infrastructure workload.
* Node Failure: If a worker node loses power, breaks its network connection, or stops communicating with etcd, the master node marks it as NotReady and recreates its pods on an online node.

## 4. Configuration and Registry Issues
The pod cannot successfully pull down or configure its dependencies.

* ErrImagePull / ImagePullBackOff: The pod goes down or fails to start because Kubernetes cannot download the Docker image. This happens if the image tag is deleted, typed incorrectly, or if the private registry credentials expired.
* Missing ConfigMaps or Secrets: If your deployment manifest references a [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/) or ConfigMap that does not exist, the pod will fail to initialize.

------------------------------
## How to Diagnose a Dead Pod
If your pod went down, you can find the exact error code by running these two diagnostic commands in your terminal:

# 1. Shows general status, restart counts, and error states (like OOMKilled)
kubectl get pods
# 2. Shows the full lifecycle event log and exact reasons for termination
kubectl describe pod <your-pod-name>

# what is kubectl get rs?

kubectl get rs is the shorthand command used to list and check the status of ReplicaSets inside your Kubernetes cluster. 
A ReplicaSet is a background controller that ensures a stable set of identical pod replicas are running at any given time. While you rarely manage ReplicaSets directly—because the higher-level Deployment object manages them for you—checking them is crucial for understanding application scaling and deployments.
------------------------------
## Understanding the Command Output
When you run kubectl get rs, you will see a table that looks like this:

NAME                          DESIRED   CURRENT   READY   AGE
myapp-deployment-55649fd747   3         3         3       15m

Here is what each column means:

* NAME: The unique identifier of the ReplicaSet. It combines your deployment name with a unique hash generated from your pod template settings.
* DESIRED: The number of identical pod instances you want running (defined by replicas in your YAML).
* CURRENT: The number of pod instances that are currently created and physically exist on the nodes.
* READY: The number of pods that are fully functional, healthy, and actively passing their readiness network checks.
* AGE: How long this specific ReplicaSet has been active in the cluster. 

------------------------------
## The Relationship Between Deployments and ReplicaSets
When you update an application, Deployments use ReplicaSets to manage Rolling Updates without downtime: 

   1. Initial State: You deploy Version 1. The Deployment creates ReplicaSet-V1 to run 3 pods.
   2. The Update: You update your Docker image tag to Version 2 and run kubectl apply.
   3. The Rollout: The Deployment does not modify the old ReplicaSet. Instead, it leaves ReplicaSet-V1 alone and creates a brand new ReplicaSet-V2.
   4. The Shift: The Deployment slowly scales ReplicaSet-V2 up (from 0 to 3 pods) while scaling ReplicaSet-V1 down (from 3 to 0 pods).

If you run kubectl get rs during or after an update, you will often see multiple ReplicaSets for the same app. The old, inactive ones are kept with a DESIRED count of 0 so you can instantly roll back to a previous working version if your new update crashes.
------------------------------

# kubectl get events?

kubectl get events is the primary diagnostic command used to view the cluster's internal timeline of recent activity. It displays chronological system logs, lifecycle changes, and error warnings for all objects (pods, nodes, configurations) within your active namespace. [1, 2, 3, 4, 5] 
While kubectl get pods tells you what state your application is currently in, kubectl get events tells you why and how it got there.
------------------------------
## 📋 Understanding the Command Output
When you run kubectl get events, you will see a structured table formatted like this:

LAST SEEN   TYPE      REASON              OBJECT                        MESSAGE
2m          Normal    Scheduled           pod/myapp-7c6f-abc12          Successfully assigned default/myapp-7c6f-abc12 to node-1
1m          Normal    Pulling             pod/myapp-7c6f-abc12          Pulling image "nginx:latest"
45s         Normal    Pulled              pod/myapp-7c6f-abc12          Successfully pulled image "nginx:latest" in 15s
30s         Normal    Created             pod/myapp-7c6f-abc12          Created container myapp
30s         Normal    Started             pod/myapp-7c6f-abc12          Started container myapp
10s         Warning   Unhealthy           pod/myapp-7c6f-abc12          Liveness probe failed: HTTP probe failed with statuscode: 500

Here is how to decode each column:

* LAST SEEN: How long ago the event occurred (or the last time a recurring event repeated).
* TYPE: The severity of the event.
* Normal: Routine operations (e.g., pulling images, scaling pods, scheduling workloads).
   * Warning: Failures or potential problems (e.g., node down, crashed containers, failing health checks).
* REASON: A single-word programmatic identifier summarizing the action (e.g., Scheduled, FailedMount, Killing, OOMKilled). 
* OBJECT: The specific resource name that triggered the log entry. 
* MESSAGE: A detailed human-readable explanation providing context on exactly what failed or succeeded. 

------------------------------
## ⚡ Power-User Command Variations
By default, standard kubectl get events dumps an unsorted list of every event in the current namespace, which can quickly become overwhelming. Use these flags to filter out the noise:  
1. Watch Events in Real Time
Stream events live as they happen in your cluster (excellent for monitoring a failing deployment):

kubectl get events --watch

2. Sort by Time (Most Recent Last)
Forces the output to display chronologically so you can see the immediate trail of breadcrumbs leading to a crash:

kubectl get events --sort-by='.metadata.creationTimestamp'

3. Target Only Error Warnings
Hides all "Normal" routine logs so you only see failures, system errors, and warning events: [24, 25] 

kubectl get events --field-selector type=Warning

4. Check a Specific Resource
To view events tied strictly to a single pod or service without scanning the entire namespace roster:

kubectl describe pod <pod-name>

(The describe command automatically appends an isolated "Events" section at the very bottom of its text output.)
------------------------------
## ⚠️ A Critical Catch: Event Lifespans
Kubernetes events are ephemeral and heavily throttled. By default, the cluster deletes all events after 1 hour to prevent the etcd database from filling up with old log entries. If an error happened yesterday, kubectl get events will be blank; you will need to check your centralized logging system or container stdout logs instead. 
------------------------------


# what are kubectl namespace commands

## What is a Namespace?
A Namespace is a virtual cluster partition within your physical Kubernetes cluster. It allows you to isolate environments (e.g., development, staging, production), manage distinct team access, and prevent naming conflicts between identical application components.
Here are the essential kubectl commands to manage, use, and filter by namespaces.
------------------------------
## 1. View Namespaces
To see the existing namespaces configured inside your cluster:

kubectl get ns

(Note: ns is the universal short-cut alias for namespaces.)
By default, Kubernetes auto-generates a few standard namespaces:

* default: Where resources go if you don’t specify a namespace.
* kube-system: Reserved for internal cluster infrastructure components (like the API server or DNS).
* kube-public: Accessible by anyone (even unauthenticated users), rarely used.
* kube-node-lease: Holds heartbeat objects to track node health. 

------------------------------
## 2. Create and Delete Namespaces
To isolate a new environment or clean up an old one:

* Create a new namespace:

kubectl create ns development

* Delete a namespace (and everything inside it):

kubectl delete ns development

⚠️ Warning: Deleting a namespace instantly and permanently deletes every pod, service, deployment, and configuration stored inside it. 

------------------------------
## 3. Target a Namespace in Commands
By default, kubectl only interacts with the default namespace. To interact with resources elsewhere, you must use flags:

* Target a specific namespace (-n):

kubectl get pods -n kube-system

* List resources across ALL namespaces (-A or --all-namespaces):

kubectl get deployments -A

* Deploy a YAML manifest into a specific namespace:

kubectl apply -f deployment.yaml -n staging

(Note: If you explicitly declare a namespace: name inside the YAML file's metadata: block, that hardcoded value will override your command-line -n flag). 

------------------------------
## 4. Switch the Default Namespace (Context)
Typing -n <namespace> on every single command becomes tedious. You can permanently switch your context so your terminal defaults to your active working directory.

* Check your active context namespace:

kubectl config view --minify | grep namespace

* Switch your default namespace workspace:

kubectl config set-context --current --namespace=development

Now, running kubectl get pods will automatically fetch data from development instead of default. 

------------------------------
