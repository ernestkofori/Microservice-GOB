# Local Deployment of Google Microservices Online Boutique

## 📋 Overview
This project demonstrates the local orchestration and deployment of the **Google Online Boutique**. It is a cloud-native, multi-service e-commerce application comprising 10 distinct microservices (including a frontend, product catalog, cart management, and payment processing). 

By migrating this microservices framework to a local Kubernetes environment, this project validates foundational DevOps core competencies in container orchestration, microservices communication paradigms, declarative resource budgeting, and cluster self-healing configurations.

## 🏗️ Architecture
The application runs as a collection of decoupled microservices that communicate using high-performance **gRPC protocols**. A single web **Frontend** acts as the ingress point for public HTTP traffic, while a localized **Redis** instance acts as an ephemeral state cache for application data tracking (such as cart storage).



### Tech Stack Used
* **Containerization:** Docker
* **Orchestration & Control Plane:** Kubernetes (via Minikube)
* **Version Control:** Git / GitHub
* **Communication Protocols:** gRPC, tcpSocket


---

## 🚀 Key Features & Implementation

### 1. High Availability & Scalability Configurations
* **Replica Redundancy:** Implemented **3x replica redundancy** across all core stateless microservices (e.g., `checkoutservice`, `productcatalogservice`, `frontend`), ensuring fault tolerance and high availability inside the cluster topology.
* **State Isolation:** Isolated volatile state management by deploying a single-node **Redis instance** dedicated to handling `cartservice` data transactions using an ephemeral `emptyDir` volume mount layout.

### 2. Service Discovery & Cluster Networking
* **Internal Security:** Configured secure, internal-only **ClusterIP Services** for all backing gRPC microservices to tightly control and lock down communication inside the private virtual network.
* **Decoupled Configuration:** Leveraged Kubernetes internal CoreDNS naming conventions inside environment variables (e.g., setting `PRODUCT_CATALOG_SERVICE_ADDR` to `productcatalogservice:3550`) to connect services dynamically without hardcoding IP addresses.
* **External Access:** Exposed the web application to external host traffic by utilizing a **NodePort Service** on the `frontend` component, routing incoming external HTTP traffic (Port 80) to the container’s listening application port (Port 8080).

### 3. Reliability & Proactive Resource Management
* **Self-Healing Mechanics:** Engineered explicit `livenessProbe` and `readinessProbe` health checks utilizing native microservice gRPC ports. This ensures the cluster automatically restarts unhealthy containers and blocks traffic from reaching pods that are still initializing.
* **Resource Constraints:** Implemented deterministic CPU and Memory budgeting (allocating a standard request of `100m` CPU / `128Mi` memory with rigid ceilings of `200m` CPU / `256Mi` memory) across all microservices to guarantee predictable scheduling and completely eliminate "noisy-neighbor" issues over a shared local hardware plane.

---

## 🛠️ Prerequisites & Setup

To replicate this deployment in a local environment, ensure your host machine has at least **4 Cores** and **8GB of RAM** allocated to your container runtime driver.



### 1. Start the local Cluster environments with enough resouces.
Ensure you have Docker and Minikube downloaded and installed.
Launch your local Minikube cluster with explicit resource configurations to handle the microservice overhead:
```bash

# 1. Start docker and minikube
sudo systemctl start docker
minikube start --cpus=4 --memory=8192 --driver=docker

# 2. Clone the documentation repository
git clone git@github.com:ernestkofori/Microservice-GOB.git
cd Microservice-GOB


# 3. Apply the combined manifest file
kubectl apply -f kubernetes-manifest.yaml

# 4. Verify the health and status of the Clusters and ensure all pods are all running.
kubectl get deployments
kubectl get pods
kubectl get svc

# 4. Access the frontend application interface via Minikube
minikube service frontend --url


```


## 💡 Challenges Faced & Lessons Learned

### Challenge 1: Local Node Resource Exhaustion (Pending Pods)
* **The Blocker:** Spinning up over 28 pods simultaneously (3 replicas across 9 microservices + Redis) on a standard host engine initially caused several pods to hang indefinitely in a `Pending` state, accompanied by heavy CPU throttling across the master node.
* **The Fix:** Diagnosed the cluster bottlenecks using `kubectl describe nodes` and identified resource request oversaturation. Solved this by profiling the microservices, optimizing individual manifest container requests down to a strict `100m` CPU minimum, and explicitly expanding the host resource allocation pool (`--cpus=4 --memory=8192`) at cluster initialization.

### Challenge 2: Service Inter-connectivity Port Misalignments
* **The Blocker:** During early functional testing, dependent systems like the `checkoutservice` were failing to establish gRPC connections to backend layers like the `paymentservice`, causing partial checkout processing crashes.
* **The Fix:** Analyzed application environment variables and cross-referenced them with the networking manifests. Mastered the isolation properties of Kubernetes Services by ensuring that environment variable addresses pointed precisely to the Service's incoming exposure port (e.g., `8083`), which then safely redirected the payload downstream via the designated internal container `targetPort` (`50051`).

---

## 📈 Future Enhancements
- [ ] **Config Decoupling:** Externalize environmental parameters and connection strings out of raw inline manifests and into dedicated Kubernetes `ConfigMaps` and `Secrets`.
- [ ] **Ingress Routing:** Implement an NGINX Ingress Controller to replace standard `NodePort` exposures for unified path-based domain routing (`/`, `/cart`, `/checkout`).
- [ ] **Observability Engine:** Integrate an automated Prometheus and Grafana monitoring stack inside a separate namespace to track live memory/CPU consumption metrics and track gRPC traffic behavior.