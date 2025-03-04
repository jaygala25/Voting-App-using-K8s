# Voting Application on Kubernetes

## Overview

Welcome to the Demo Voting Application, a distributed application deployed on Kubernetes that allows users to cast votes and view real-time results. This application demonstrates the power of container orchestration, microservices architecture, and scalable deployment using Kubernetes. It consists of multiple components, including a voting interface, a result interface, a worker, a Redis database for temporary storage, and a PostgreSQL database for persistent storage.

This README provides an overview of the application, its architecture, deployment instructions, and guidelines for running and contributing to the project.

---

## Table of Contents

1. [Architecture](#architecture)
2. [Components](#components)
3. [Prerequisites](#prerequisites)
4. [Deployment](#deployment)
5. [Accessing the Application](#accessing-the-application)
6. [Scaling](#scaling)
7. [Contributing](#contributing)
8. [License](#license)

---

## Architecture

The Demo Voting Application is deployed in a Kubernetes namespace called `vote`. It follows a microservices-based architecture with the following components:

- **Voting User Interface**: Where users cast their votes.
- **Result User Interface**: Displays real-time voting results.
- **Voting Service**: Exposes the voting interface to users.
- **Result Service**: Exposes the result interface to users.
- **Worker**: Processes votes and updates the databases.
- **Redis**: An in-memory data store for temporary vote storage and inter-component communication.
- **PostgreSQL**: A persistent database for storing final vote results.

The architecture can be visualized as follows:

```
[Diagram: Voting Application Architecture]
- Namespace: `vote`
- Voting User -> Voting Service (NodePort: 30004) -> Voting Deployment
- Result User -> Result Service (NodePort: 30005) -> Result Deployment
- Voting Deployment <-> Redis Service <-> Redis Deployment
- Voting Deployment <-> Worker Deployment <-> PostgreSQL Service <-> PostgreSQL Deployment
```

This diagram illustrates the data flow and interactions between components, with services exposed via NodePorts and deployments managed by Kubernetes.

---

## Components

The application includes the following Kubernetes resources:

### 1. Voting Application
- **Deployment**: `voting-app-deploy`
  - Replicas: 1
  - Pod Name: `voting-app-pod`
  - Container: `voting-app`
    - Image: `kodekloud/examplevotingapp_vote:v1`
    - Port: 80
- **Service**: `voting-service`
  - Type: NodePort
  - Port: 80 (Target Port: 80, NodePort: 30004)
  - Selector: `name: voting-app-pod, app: demo-voting-app`

### 2. Result Application
- **Deployment**: `result-app-deploy`
  - Replicas: 1
  - Pod Name: `result-app-pod`
  - Container: `result-app`
    - Image: `kodekloud/examplevotingapp_result:v1`
    - Port: 80
- **Service**: `result-service`
  - Type: NodePort
  - Port: 80 (Target Port: 80, NodePort: 30005)
  - Selector: `name: result-app-pod, app: demo-voting-app`

### 3. Worker Application
- **Deployment**: `worker-app-deploy`
  - Replicas: 1
  - Pod Name: `worker-app-pod`
  - Container: `worker-app`
    - Image: `kodekloud/examplevotingapp_worker:v1`

### 4. Redis
- **Deployment**: `redis-deploy`
  - Replicas: 1
  - Pod Name: `redis-pod`
  - Container: `redis`
    - Image: `redis`
    - Port: 6379
- **Service**: `redis`
  - Port: 6379 (Target Port: 6379)
  - Selector: `name: redis-pod, app: demo-voting-app`

### 5. PostgreSQL
- **Deployment**: `postgres-deploy`
  - Replicas: 1
  - Pod Name: `postgres-pod`
  - Container: `postgres`
    - Image: `postgres:11`
    - Port: 5432
    - Environment Variables:
      - `POSTGRES_USER`: "postgres"
      - `POSTGRES_PASSWORD`: "postgres"
      - `POSTGRES_HOST_AUTH_METHOD`: "md5"
- **Service**: `db`
  - Port: 5432 (Target Port: 5432)
  - Selector: `name: postgres-pod, app: demo-voting-app`

All components are labeled with `app: demo-voting-app` and organized under the `vote` namespace for isolation and management.

---

## Prerequisites

Before deploying the Demo Voting Application, ensure you have:

- **Kubernetes Cluster**: A running cluster (e.g., Minikube, Kind, or a cloud provider like GKE, EKS, or AKS).
- **kubectl**: Installed and configured to communicate with your cluster.
- **Docker**: Installed (optional, for building custom images locally).

---

## Deployment

Follow these steps to deploy the application on Kubernetes:

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/your-username/demo-voting-app.git
   cd demo-voting-app
   ```

2. **Apply Kubernetes Manifests**:
   Assuming the manifests are in a `k8s/` directory, apply them in this order:
   ```bash
   kubectl apply -f k8s/namespace.yaml
   kubectl apply -f k8s/postgres-deployment.yaml
   kubectl apply -f k8s/postgres-service.yaml
   kubectl apply -f k8s/redis-deployment.yaml
   kubectl apply -f k8s/redis-service.yaml
   kubectl apply -f k8s/worker-deployment.yaml
   kubectl apply -f k8s/voting-deployment.yaml
   kubectl apply -f k8s/voting-service.yaml
   kubectl apply -f k8s/result-deployment.yaml
   kubectl apply -f k8s/result-service.yaml
   ```

3. **Verify Deployment**:
   Ensure all resources are running:
   ```bash
   kubectl get pods -n vote
   kubectl get deployments -n vote
   kubectl get services -n vote
   ```

4. **Monitor Logs** (Optional):
   Check logs for troubleshooting:
   ```bash
   kubectl logs <pod-name> -n vote
   ```

---

## Accessing the Application

After deployment, access the application via NodePorts:

- **Voting Interface**:
  - URL: `http://<node-ip>:30004`
  - Cast votes here.

- **Result Interface**:
  - URL: `http://<node-ip>:30005`
  - View real-time results.

Replace `<node-ip>` with your Kubernetes node's IP address.

---

## Scaling

Each deployment starts with `replicas: 1`. To scale a component (e.g., `voting-app-deploy`), use:

```bash
kubectl scale deployment <deployment-name> --replicas=<number> -n vote
```

For example, to scale the voting application to 3 replicas:
```bash
kubectl scale deployment voting-app-deploy --replicas=3 -n vote
```

---

## Contributing

We welcome contributions! To get started:

1. Fork the repository.
2. Create a branch for your feature or fix.
3. Commit your changes with clear messages.
4. Push your branch and submit a pull request.

Follow Kubernetes best practices and include tests where applicable.

---

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

---

This README offers a complete guide to understanding, deploying, and managing the Demo Voting Application on Kubernetes, based on the provided architecture and manifests. Enjoy exploring this demo!
