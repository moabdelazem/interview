# Kubernetes Interview Questions

**Q1: What is Kubernetes?**

Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It provides a way to manage and run containerized applications in a distributed environment, making it easier to build, deploy, and manage containerized applications at scale.

**Q2: Explain the kubernetes architecture**

Kubernetes follow master-worker architecture.

Master architecture (Control Plane):

- `API Server`: The API server is the entry point for the control plane.
- `etcd`: The etcd is the key-value store that stores the state of the cluster.
- `Controller Manager`: The controller manager is the component that manages the controllers.
- `Scheduler`: The scheduler is the component that assigns pods to nodes.

Worker architecture (Data Plane):

- `Kubelet`: The kubelet is the agent that runs on each node and manages the containers.
- `Container Runtime`: The container runtime is the runtime that runs the containers.
- `Kube Proxy`: The kube proxy is the network proxy that runs on each node and manages the network policies.

**Q3: What is the difference between a `Pod` and a container?**

A `Pod` is a group of containers that are scheduled together on the same node. A `container` is an instance of an image that is running in a `Pod`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: web-container
      image: nginx:alpine
```

**Q4: Why do we need `Services` if pod can be reached by IP address?**

Pods are ephemeral; their IPs change when they restart. A Service provides a stable IP and DNS name that load-balances traffic across a set of Pods.

**Q5: Explain the `ReplicaSet` and `Deployment`**

A `Deployment` is the high-level object you interact with. It manages the `ReplicaSet`, which in turn ensures the exact number of Pods are running. Deployments allow for rolling updates and rollbacks.

**Q6: Explain the `Node` and `Master`**

A `Node` is a worker machine in Kubernetes. A `Master` is the control plane that manages the cluster.

**Q7: What are `Namespaces`, any why do we need them?**

They are virtual clusters within a physical cluster. They provide logical isolation for different teams, environments (Prod/Dev), or projects and help apply RBAC and resource quotas.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

---

**Q8: Explain the kubernetes `Control Loop`**

```
Observe the current state of the cluster (API Server) -> compare it to the desired state (YAML) -> take actions to make the current state match the desired state through controllers.
```

**Q9: Explain the difference between `ConfigMap` and `Secret`**

Both store configuration data, but `Secret` are specifically for sensitive data (passwords, tokens, etc), Secrets are only Base64 encoded by default and should be used with encryption-at-rest or external providers like HashiCorp Vault, Sealed Secrets, etc.

**Q10: What is difference between `Deployment` and `StatefulSet`**

`Deployment` is for stateless applications, while `StatefulSet` is for stateful applications (Databases, etc).

**Q11: StatefulSets vs. Deployments: When do you choose?**

Use `StatefulSets` for apps requiring stable network identities and ordered persistent storage (e.g., MongoDB, Kafka). Use `Deployments` for stateless web apps where any replica is identical to another.

**Q12: What is Headless Service?**

A Service with `clusterIP: None`. It doesn't load-balance; instead, it returns the individual IP addresses of the Pods via DNS. Often used for StatefulSets (Databases) where Pods need to talk to each other directly.

**Q13: Explain PV, PVC, and StorageClass**

These three components work together to manage persistent storage in Kubernetes:

- **PersistentVolume (PV)**: A piece of storage in the cluster that has been provisioned by an administrator or dynamically using a StorageClass. Think of it as the actual disk or storage resource (e.g., an AWS EBS volume, NFS share, or local disk). PVs have a lifecycle independent of any Pod.

- **PersistentVolumeClaim (PVC)**: A request for storage by a user. It's like a "ticket" that says "I need 10GB of fast storage." Kubernetes matches the PVC to an available PV that meets the requirements. Pods use PVCs to access storage.

- **StorageClass**: Defines the "type" of storage available (e.g., fast SSD, slow HDD, replicated, etc.). It enables **dynamic provisioning** - when a PVC is created, Kubernetes automatically creates a matching PV using the StorageClass.

**Simple Analogy:**

- `StorageClass` = The menu of storage options (SSD, HDD, etc.)
- `PVC` = Your order ("I want 10GB of SSD storage")
- `PV` = The actual storage you receive

**Q14: What is Ingress?**

Ingress is a Kubernetes API object that manages external HTTP/HTTPS access to services within a cluster. It acts as a smart router that sits at the edge of your cluster.

**Why use Ingress instead of LoadBalancer Services?**

- A LoadBalancer Service creates one external IP per service (expensive!)
- Ingress provides a single entry point for multiple services with path-based or host-based routing

**Key Components:**

- **Ingress Resource**: The YAML configuration that defines routing rules (e.g., `/api` goes to service A, `/web` goes to service B)
- **Ingress Controller**: The actual implementation that makes Ingress work (e.g., NGINX, Traefik, HAProxy). Without a controller, Ingress resources do nothing.

**What Ingress provides:**

- Path-based routing (`example.com/api` → api-service)
- Host-based routing (`api.example.com` → api-service)
- SSL/TLS termination
- Load balancing

**Q15: How do Liveness, Readiness, and Startup probes differ?**

**Startup**: Runs first; disables other probes until the app finishes starting up.
**Readiness**: Tells K8s when the app is ready to receive traffic (added to Service endpoints).
**Liveness**: Tells K8s if the app is alive. If it fails, K8s restarts the container.

---

**Q16: What are the kubernetes services?**

- `NodePort`: Exposes the service on a static port on each node's IP address.
- `ClusterIP`: Exposes the service on a cluster-internal IP address.
- `LoadBalancer`: Exposes the service using a cloud provider's load balancer.

**Q17: What are Taints and Tolerations?**

Taints and Tolerations work together to **repel pods from nodes** unless they explicitly allow it.

**Taints** (applied to Nodes):
A taint marks a node as "restricted." Pods won't be scheduled on tainted nodes unless they tolerate the taint.

```bash
# Add a taint to a node
kubectl taint nodes node1 key=value:NoSchedule
```

**Taint Effects:**

- `NoSchedule`: Pods won't be scheduled on this node (existing pods stay)
- `PreferNoSchedule`: Scheduler will _try_ to avoid this node, but may still schedule if needed
- `NoExecute`: Existing pods are evicted, new pods won't be scheduled

**Tolerations** (applied to Pods):
A toleration allows a pod to be scheduled on nodes with matching taints.

```yaml
tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```

**Use Cases:**

- Dedicated nodes for specific workloads (GPU nodes, high-memory nodes)
- Preventing regular workloads from running on master nodes
- Evicting pods during node maintenance

**Q18: What is Node Affinity and Pod Affinity?**

Affinity rules **attract pods to specific nodes or other pods**. They're the opposite of taints (which repel).

**Node Affinity** (Pod → Node attraction):
Tells the scheduler to place pods on nodes with specific labels.

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution: # HARD requirement
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
    preferredDuringSchedulingIgnoredDuringExecution: # SOFT preference
      - weight: 1
        preference:
          matchExpressions:
            - key: zone
              operator: In
              values:
                - us-east-1a
```

**Two Types:**

- `requiredDuringScheduling...`: Pod **must** be scheduled on matching nodes (hard)
- `preferredDuringScheduling...`: Scheduler will **try** to use matching nodes (soft)

**Pod Affinity / Anti-Affinity** (Pod → Pod attraction/repulsion):
Controls pod placement relative to other pods.

- **Pod Affinity**: "Schedule me on the same node/zone as pods with label X" (co-location)
- **Pod Anti-Affinity**: "Don't schedule me on the same node as pods with label X" (spread)

```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: cache
        topologyKey: kubernetes.io/hostname # Same node
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: web
          topologyKey: kubernetes.io/hostname # Different nodes
```

**Use Cases:**

- Co-locate frontend and cache pods for low latency (affinity)
- Spread replicas across nodes/zones for high availability (anti-affinity)

**Q19: How do Taints/Tolerations and Node Affinity work together?**

They complement each other for fine-grained scheduling control:

| Mechanism              | Direction            | Purpose                                            |
| ---------------------- | -------------------- | -------------------------------------------------- |
| **Taints/Tolerations** | Node → Pod (repel)   | "Keep pods **away** from this node unless allowed" |
| **Node Affinity**      | Pod → Node (attract) | "Place this pod **on** specific nodes"             |

**How they work together:**

1. **Taints block by default** - A tainted node rejects all pods
2. **Tolerations allow entry** - A pod with matching toleration CAN be scheduled there
3. **Node Affinity directs placement** - The pod actively WANTS to go to specific nodes

**Example: Dedicated GPU Nodes**

```bash
# Step 1: Taint GPU nodes (repel regular workloads)
kubectl taint nodes gpu-node-1 gpu=true:NoSchedule
```

```yaml
# Step 2: GPU workload with both toleration AND affinity
spec:
  tolerations: # "I'm allowed on GPU nodes"
    - key: "gpu"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"
  affinity:
    nodeAffinity: # "I WANT to be on GPU nodes"
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: gpu
                operator: In
                values:
                  - "true"
```

**Why use both?**

- Toleration alone: Pod _can_ go to GPU node, but might land elsewhere
- Affinity alone: Pod _wants_ GPU node, but can't bypass the taint
- **Both together**: Pod is _allowed_ AND _directed_ to GPU nodes

**Q20: Can you walk me through what happens behind the scenes when I execute `kubectl apply -f deployment.yaml`?**

1. The Request (Client Side)

`kubectl`: It validates the YAML file locally, converts it to JSON, and sends a `POST` request to the API Server.

2. The Gatekeeper (API Server)

Auth & Checks: The API Server receives the request and performs Authentication (Who are you?), Authorization (Can you do this?), and runs Admission Controllers (Mutating/Validating webhooks to enforce policies).

Persistence: If everything passes, the API Server writes the Deployment object to Etcd (the cluster's database).

3. The Brain (Controller Manager)

Watch Mechanism: The Deployment Controller (inside the Controller Manager) is constantly watching the API Server. It detects the new Deployment object.

ReplicaSet Creation: It creates a ReplicaSet specifically for that version of the Deployment.

Pod Creation: The ReplicaSet Controller sees the new ReplicaSet and notices the current pod count (0) doesn't match the desired replicas (e.g., 3). It creates 3 Pod objects in the API Server. Crucially, these Pods are currently in a "Pending" state with no Node assigned.

4. The Decision (Scheduler)

Scheduling: The kube-scheduler watches for Pods with no assigned node.

Filter & Score: It filters out nodes that don't have enough resources (CPU/RAM) or don't match taints/tolerations. It then scores the remaining nodes to find the best fit.

Binding: It updates the API Server, setting the nodeName field in the Pod object to the chosen Node.

5. The Execution (Kubelet)

Action: The Kubelet running on that specific Node is watching for Pods assigned to itself.

Runtime: It instructs the Container Runtime (like containerd or Docker) to pull the image and start the container.

Status Loop: The Kubelet reports the status (e.g., "Running") back to the API Server, which updates the Etcd.

**Q21: What is Horizontal Pod Autoscaler (HPA)?**

The Horizontal Pod Autoscaler (HPA) automatically scales the number of pod replicas in a Deployment, ReplicaSet, or StatefulSet based on observed metrics (like CPU/memory usage or custom metrics).

**How it works:**

1. HPA continuously monitors metrics via the Metrics Server
2. Compares current metric values against the target threshold
3. Calculates the desired number of replicas
4. Adjusts the replica count automatically (scale up or down)

**Example HPA:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70 # Scale when CPU > 70%
```

**Key Configuration:**

- `minReplicas` / `maxReplicas`: Bounds for scaling
- `scaleTargetRef`: The workload to scale
- `metrics`: What to monitor (CPU, memory, custom metrics)

**Important Notes:**

- Requires **Metrics Server** to be installed in the cluster
- Pods must have **resource requests** defined for CPU/memory scaling to work
- HPA checks metrics every 15 seconds by default
- There's a cooldown period to prevent thrashing (scale-up: 0s, scale-down: 5min by default)

**Q22: What are Network Policies?**

Network Policies are Kubernetes resources that control **pod-to-pod traffic** at the IP/port level. By default, all pods can communicate with each other freely. Network Policies allow you to restrict this based on labels, namespaces, and IP blocks.

**Key Concepts:**

- **Ingress Rules**: Control incoming traffic TO pods
- **Egress Rules**: Control outgoing traffic FROM pods
- **Default Deny**: Once a NetworkPolicy selects a pod, all traffic not explicitly allowed is denied

**Example: Allow traffic only from specific pods**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-network-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api # Apply to pods with label app=api
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend # Only allow from frontend pods
        - namespaceSelector:
            matchLabels:
              name: monitoring # Allow from monitoring namespace
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: database
      ports:
        - protocol: TCP
          port: 5432
```

**Common Use Cases:**

- **Default Deny All**: Block all traffic, then whitelist specific flows
- **Namespace Isolation**: Pods in `dev` can't talk to pods in `prod`
- **Database Protection**: Only backend pods can reach the database
- **Egress Control**: Restrict which external services pods can reach

**Default Deny All Example:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {} # Applies to ALL pods in namespace
  policyTypes:
    - Ingress
    - Egress
  # No ingress/egress rules = deny all
```

**Important Notes:**

- Network Policies require a **CNI plugin** that supports them (Calico, Cilium, Weave Net). Basic CNIs like Flannel don't enforce Network Policies
- Policies are **additive**: multiple policies combining = union of all allowed traffic
- Empty `podSelector: {}` means the policy applies to all pods in the namespace
