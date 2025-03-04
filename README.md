# Voting Application on Kubernetes

## Overview

Welcome to the Demo Voting Application, a distributed application deployed on Kubernetes that allows users to cast votes and view real-time results. This application demonstrates the power of container orchestration, microservices architecture, and scalable deployment using Kubernetes. It consists of multiple components, including a voting interface, a result interface, a worker, a Redis database for temporary storage, and a PostgreSQL database for persistent storage.

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

The Demo Voting Application is deployed in a Kubernetes namespace called `vote`. It follows a microservices-based architecture that showcases the interaction between different services and deployments:

![Screenshot 2025-03-04 at 2 51 09 PM](https://github.com/user-attachments/assets/fd1368e8-8ab0-4cea-ab73-921a59943e4b)

### Detailed Component Interactions

1. **Voting Users**
   - External users interact with the application through the voting interface
   - Connects to the `vote-service`, which routes requests to the `vote-deployment`
   - Allows users to cast their vote between Cats and Dogs

2. **Vote Service (`vote-service`)**
   - A Kubernetes service that exposes the voting interface
   - Acts as a load balancer for the voting application pods
   - Enables external access to the voting interface via NodePort
   - Routes incoming traffic to the `vote-deployment`

3. **Vote Deployment (`vote-deployment`)**
   - Manages the pods running the voting application
   - Ensures the specified number of replicas are running
   - Contains the frontend logic for the voting interface

4. **Redis Service and Deployment (`redis-service` and `redis-deployment`)**
   - Provides a temporary data storage solution
   - Stores incoming votes before they are processed
   - Facilitates communication between the voting interface and the worker

5. **Worker Deployment (`worker-deployment`)**
   - Background process that:
     - Retrieves votes from Redis
     - Processes and transforms vote data
     - Writes processed votes to the PostgreSQL database
   - Acts as a bridge between the voting interface and the persistent database

6. **User Result Check**
   - Users can view real-time voting results
   - Connects to the `result-service`

7. **Result Service (`result-service`)**
   - Exposes the results interface
   - Routes requests to the `result-deployment`
   - Enables external access to voting results

8. **Result Deployment (`result-deployment`)**
   - Manages pods displaying voting results
   - Retrieves and displays vote counts from the database

9. **Database Service and Deployment (`db-service` and `db-deployment`)**
   - Persistent storage for final vote results
   - Uses PostgreSQL to store the aggregated voting data
   - Provides a permanent record of votes processed by the worker

### Data Flow
1. User casts a vote through the voting interface
2. Vote is temporarily stored in Redis
3. Worker processes the vote from Redis
4. Processed vote is written to PostgreSQL
5. Result interface fetches and displays votes from the database

---

## Application User Interface

### Voting Interface
Users can cast their vote between Cats and Dogs:

![Screenshot 2025-03-04 at 2 38 44 PM](https://github.com/user-attachments/assets/1afdabe8-5900-4b40-87e3-503cfeca5d47)

### Real-time Results
The results are updated in real-time:

![Screenshot 2025-03-04 at 2 38 52 PM](https://github.com/user-attachments/assets/be11e491-8e4a-46e9-bbe9-a61db2db018b)

---

## Kubernetes Deployment Status

### Deployment and Pod Status
The application can be verified using `kubectl`:

<img width="905" alt="Screenshot 2025-03-04 at 2 41 11 PM" src="https://github.com/user-attachments/assets/18b421fe-e2b2-41f8-bfd5-275db4a4a751" />

Key observations:
- All deployments are READY and AVAILABLE
- Pods are in 'Running' status
- No restart issues detected

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

- **Kubernetes Cluster**: A running cluster (e.g., Minikube, Kind, or a cloud provider like GKE, EKS, or AKS)
- **kubectl**: Installed and configured to communicate with your cluster
- **Docker**: Installed (optional, for building custom images locally)

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
  - Cast votes here

- **Result Interface**:
  - URL: `http://<node-ip>:30005`
  - View real-time results

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

1. Fork the repository
2. Create a branch for your feature or fix
3. Commit your changes with clear messages
4. Push your branch and submit a pull request

Follow Kubernetes best practices and include tests where applicable.

---

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

---

This README offers a complete guide to understanding, deploying, and managing the Demo Voting Application on Kubernetes. Enjoy exploring this demo!
