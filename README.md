# Three-Tier Chat Application — Kubernetes Deployment

A three-tier chat application deployed on **Kubernetes using Minikube**, with containerized frontend and backend services and MongoDB running inside the Kubernetes cluster.

This project demonstrates containerization, Kubernetes deployments, services, persistent storage, secrets, and local Kubernetes deployment using Minikube.

---

## Tech Stack

- Docker
- Docker Hub
- Kubernetes
- Minikube
- kubectl
- MongoDB
- Node.js / Express
- React
- Socket.IO

---

## Project Structure

```text
kubernetes/
│
├── backend/
│   ├── Dockerfile
│   └── ...
│
├── frontend/
│   ├── Dockerfile
│   └── ...
│
├── k8s/
│   ├── namespace.yml
│   ├── secrets.yml
│   ├── deployment-frontend.yml
│   ├── frontend-service.yml
│   ├── deployment-backend.yml
│   ├── backend-service.yml
│   ├── mongodb-deployment.yml
│   ├── mongodb-service.yml
│   ├── mongodb-pv.yml
│   └── mongodb-pvc.yml
│
├── docker-compose.yml
├── Jenkinsfile
├── .env.example
├── .gitignore
└── README.md
```

---

## 1. Prerequisites

Make sure the following tools are installed.

- Git
- Docker
- kubectl
- Minikube
- A Docker Hub account

### Windows

**Install Docker Desktop**

Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop/).

Verify:

```bash
docker --version
```

**Install kubectl**

```bash
winget install Kubernetes.kubectl
```

Verify:

```bash
kubectl version --client
```

**Install Minikube**

```bash
winget install Kubernetes.minikube
```

Verify:

```bash
minikube version
```

### Linux

**Install Docker**

For Ubuntu/Debian:

```bash
sudo apt update
sudo apt install docker.io -y
```

Start Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Verify:

```bash
docker --version
```

Optional — allow the current user to run Docker without `sudo`:

```bash
sudo usermod -aG docker $USER
```

Log out and back in after running this command.

**Install kubectl**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
```

Verify:

```bash
kubectl version --client
```

**Install Minikube**

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Verify:

```bash
minikube version
```

---

## 2. Clone the Repository

```bash
git clone https://github.com/harshitsaxena214/kubernetes.git
cd kubernetes
```

---

## 3. Start Minikube

Start a local Kubernetes cluster using Docker as the driver:

```bash
minikube start --driver=docker
```

Check the cluster status:

```bash
minikube status
```

Verify that Kubernetes can communicate with the cluster:

```bash
kubectl get nodes
```

Expected output should show a node in the `Ready` state.

---

## 4. Build Docker Images

```bash
docker build -t full-stack-frontend:latest ./frontend
docker build -t full-stack-backend:latest ./backend
```

Verify the images:

```bash
docker images
```

---

## 5. Push Images to Docker Hub

Log in:

```bash
docker login
```

Tag the images (replace `<dockerhub-username>` with your Docker Hub username):

```bash
docker tag full-stack-frontend:latest <dockerhub-username>/full-stack-frontend:latest
docker tag full-stack-backend:latest <dockerhub-username>/full-stack-backend:latest
```

Push them:

```bash
docker push <dockerhub-username>/full-stack-frontend:latest
docker push <dockerhub-username>/full-stack-backend:latest
```

The images will now be available in your Docker Hub repositories.

---

## 6. Configure Kubernetes Images

Update the Kubernetes deployment files to use your Docker Hub images.

Frontend:

```yaml
containers:
  - name: frontend
    image: <dockerhub-username>/full-stack-frontend:latest
```

Backend:

```yaml
containers:
  - name: backend
    image: <dockerhub-username>/full-stack-backend:latest
```

Replace `<dockerhub-username>` with your actual Docker Hub username in both files.

---

## 7. Kubernetes Namespace

The application uses a dedicated namespace, `chat-app`, defined in `k8s/namespace.yml`.

```bash
kubectl apply -f k8s/namespace.yml
```

Verify:

```bash
kubectl get namespaces
```

---

## 8. Kubernetes Secrets

The application uses Kubernetes Secrets for sensitive configuration, defined in `k8s/secrets.yml`.

```bash
kubectl apply -f k8s/secrets.yml
```

Verify:

```bash
kubectl get secrets -n chat-app
```

> **Important:** Do not commit real passwords, API keys, database credentials, JWT secrets, or other sensitive values to GitHub. `secrets.yml` in a public repository should contain only safe/example values, or be excluded from Git via `.gitignore`.

---

## 9. Deploy MongoDB

MongoDB runs inside the Kubernetes cluster. The project includes:

- `k8s/mongodb-pv.yml`
- `k8s/mongodb-pvc.yml`
- `k8s/mongodb-deployment.yml`
- `k8s/mongodb-service.yml`

Apply them in order:

```bash
kubectl apply -f k8s/mongodb-pv.yml
kubectl apply -f k8s/mongodb-pvc.yml
kubectl apply -f k8s/mongodb-deployment.yml
kubectl apply -f k8s/mongodb-service.yml
```

Check MongoDB resources:

```bash
kubectl get pods -n chat-app
kubectl get svc -n chat-app
```

Check persistent storage:

```bash
kubectl get pv
kubectl get pvc -n chat-app
```

---

## 10. Deploy Backend

```bash
kubectl apply -f k8s/deployment-backend.yml
kubectl apply -f k8s/backend-service.yml
```

Verify:

```bash
kubectl get pods -n chat-app
kubectl get svc -n chat-app
```

---

## 11. Deploy Frontend

```bash
kubectl apply -f k8s/deployment-frontend.yml
kubectl apply -f k8s/frontend-service.yml
```

Verify:

```bash
kubectl get pods -n chat-app
kubectl get svc -n chat-app
```

---

## 12. Access the Application via Port Forwarding

```bash
kubectl port-forward service/frontend 8080:80 -n chat-app
```

You should see:

```text
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

Open your browser at [http://localhost:8080](http://localhost:8080).

Flow:

```text
Browser (localhost:8080)
        │
        ▼
kubectl port-forward
        │
        ▼
Frontend Service :80
        │
        ▼
Frontend Pod :80
```

Keep the port-forwarding terminal running while using the app. Stop it with `Ctrl + C` (works on both Windows and Linux).

---

## 13. Deploy Everything at Once

Once the manifests are configured, deploy all resources in one command:

```bash
kubectl apply -f k8s/
```

Check the resources:

```bash
kubectl get all -n chat-app
kubectl get pv
kubectl get pvc -n chat-app
```

---

## 14. Verify the Deployment

```bash
kubectl get pods -n chat-app
kubectl get svc -n chat-app
kubectl get deployments -n chat-app
kubectl get all -n chat-app
```

A healthy deployment should show application Pods `Running` with `READY 1/1`.

---

## 15. Scaling Deployments

```bash
kubectl get deployments -n chat-app
kubectl scale deployment/<deployment-name> --replicas=3 -n chat-app
kubectl get pods -n chat-app
```

---

## 16. Updating Docker Images

After making changes to the application, rebuild:

```bash
docker build -t full-stack-frontend:latest ./frontend
docker build -t full-stack-backend:latest ./backend
```

Tag and push the updated images:

```bash
docker tag full-stack-frontend:latest <dockerhub-username>/full-stack-frontend:latest
docker tag full-stack-backend:latest <dockerhub-username>/full-stack-backend:latest

docker push <dockerhub-username>/full-stack-frontend:latest
docker push <dockerhub-username>/full-stack-backend:latest
```

Then update the Kubernetes deployments as required.

---

## 17. Stop the Application

Remove all Kubernetes resources:

```bash
kubectl delete -f k8s/
```

Or delete just the namespace (this removes everything inside it):

```bash
kubectl delete namespace chat-app
```

---

## 18. Stop Minikube

```bash
minikube stop
```

Start it again later with:

```bash
minikube start --driver=docker
```

---

## 19. Delete Minikube

To completely remove the cluster:

```bash
minikube delete
```

A new cluster can be created later with:

```bash
minikube start --driver=docker
```

---

## 20. Quick Start

For an already configured environment:

```bash
git clone https://github.com/harshitsaxena214/kubernetes.git
cd kubernetes

minikube start --driver=docker

docker build -t full-stack-frontend:latest ./frontend
docker build -t full-stack-backend:latest ./backend

docker tag full-stack-frontend:latest <dockerhub-username>/full-stack-frontend:latest
docker tag full-stack-backend:latest <dockerhub-username>/full-stack-backend:latest

docker push <dockerhub-username>/full-stack-frontend:latest
docker push <dockerhub-username>/full-stack-backend:latest

kubectl apply -f k8s/
kubectl get all -n chat-app

kubectl port-forward service/frontend 8080:80 -n chat-app
```

Open [http://localhost:8080](http://localhost:8080).

---

## Kubernetes Architecture

```text
                         Kubernetes Cluster
                              │
                         chat-app namespace
                              │
              ┌───────────────┼────────────────┐
              │               │                │
              ▼               ▼                ▼
        Frontend Service   Backend Service   MongoDB Service
             :80              :5001
              │                 │                 │
              ▼                 ▼                 ▼
        Frontend Pods     Backend Pods       MongoDB Pod
                                                  │
                                                  ▼
                                             PVC / PV
```

The frontend is accessed externally through port forwarding:

```text
localhost:8080
      │
      ▼
kubectl port-forward
      │
      ▼
Frontend Service :80
      │
      ▼
Frontend Pod :80
```

The backend stays internal to the cluster:

```text
Frontend → Backend Service → Backend Pods
```

MongoDB is also deployed inside the cluster, backed by persistent storage.

---

## DevOps Concepts Demonstrated

- Containerization with Docker
- Docker image creation
- Docker Hub image registry
- Kubernetes namespaces
- Kubernetes Deployments
- Kubernetes Services
- ClusterIP networking
- Kubernetes Secrets
- Persistent Volumes and Persistent Volume Claims
- MongoDB deployment
- Minikube
- kubectl
- Port forwarding
- Scaling
- Container orchestration
