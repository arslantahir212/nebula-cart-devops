# NebulaCart — DevOps CI/CD Project

> A premium, futuristic electronics eCommerce storefront built with React and Vite, containerised with Docker, automated with Jenkins CI/CD, and deployed to Kubernetes using kind on AWS EC2.

---

## Project Overview

NebulaCart is a single-page electronics storefront designed with a modern dark neon UI and glassmorphism-inspired design.

The application includes:

- 12 premium electronics products
- Product search
- Category filtering
- Shopping cart management
- React Context and useReducer state management
- Responsive user interface
- Docker containerisation
- Kubernetes deployment
- Automated Jenkins CI/CD pipeline

This project was extended into a complete DevOps workflow that automatically builds, versions, publishes, and deploys the application.

---

# DevOps Architecture

```text
                    Developer
                        |
                        v
                  GitHub Repository
                        |
                        v
                     Jenkins
                        |
                 Checkout Source
                        |
                        v
                Build Docker Image
                        |
                        v
            Tag with BUILD_NUMBER
                        |
                        v
                  Docker Hub
                        |
                        v
             Update Kubernetes Image
                        |
                        v
               Kubernetes (kind)
                        |
              +---------+---------+
              |                   |
              v                   v
           Pod 1               Pod 2
         Replica 1            Replica 2
              |                   |
              +---------+---------+
                        |
                        v
               Kubernetes Service
                        |
                        v
                  NodePort 30080
                        |
                        v
                 AWS EC2 Public IP
                        |
                        v
              NebulaCart Application
```

---

# Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 19 |
| Build Tool | Vite |
| Styling | Tailwind CSS |
| Icons | Lucide React |
| State Management | React Context + useReducer |
| Containerisation | Docker |
| Container Registry | Docker Hub |
| CI/CD | Jenkins |
| Orchestration | Kubernetes |
| Kubernetes Environment | kind |
| CLI | kubectl |
| Cloud Environment | AWS EC2 |
| Source Control | Git & GitHub |

---

# Local Development

## Install Dependencies

```bash
npm install
```

## Start Development Server

```bash
npm run dev
```

The development server runs on:

```text
http://localhost:5173
```

## Production Build

```bash
npm run build
```

The production build is generated in:

```text
dist/
```

---

# Docker

The application uses a multi-stage Docker build.

## Build Docker Image

```bash
docker build -t nebula-cart:1.0 .
```

## Run Docker Container

```bash
docker run -d \
  --name nebula-cart \
  -p 5173:5173 \
  nebula-cart:1.0
```

The application can then be accessed at:

```text
http://localhost:5173
```

## Check Running Container

```bash
docker ps
```

## Stop Container

```bash
docker stop nebula-cart
```

## Remove Container

```bash
docker rm nebula-cart
```

---

# Docker Hub

The application Docker images are stored in Docker Hub.

Docker Hub repository:

```text
arslanlinux/nebula-cart
```

The Jenkins pipeline creates versioned Docker images using the Jenkins `BUILD_NUMBER`.

Examples:

```text
arslanlinux/nebula-cart:1
arslanlinux/nebula-cart:2
arslanlinux/nebula-cart:3
```

The pipeline also maintains:

```text
arslanlinux/nebula-cart:latest
```

Versioned image tags allow each Jenkins build to be identified and deployed independently.

---

# Jenkins CI/CD Pipeline

The CI/CD pipeline is defined in:

```text
Jenkinsfile
```

The pipeline automates the process from source code to Kubernetes deployment.

## Pipeline Stages

### 1. Checkout

Jenkins checks out the latest source code from the GitHub `main` branch.

```text
GitHub
   |
   v
Jenkins Checkout
```

---

### 2. Build Docker Image

Jenkins builds the Docker image using the current Jenkins build number.

Example:

```text
arslanlinux/nebula-cart:3
```

The image is also tagged as:

```text
arslanlinux/nebula-cart:latest
```

---

### 3. Push to Docker Hub

Jenkins authenticates with Docker Hub using Jenkins credentials and pushes both image tags.

```text
arslanlinux/nebula-cart:${BUILD_NUMBER}
```

and:

```text
arslanlinux/nebula-cart:latest
```

---

### 4. Deploy to Kubernetes

After successfully pushing the image, Jenkins updates the Kubernetes Deployment.

The deployment command is:

```bash
kubectl set image deployment/nebula-cart \
nebula-cart=arslanlinux/nebula-cart:${BUILD_NUMBER} \
-n nebula-cart
```

Jenkins then waits for the Kubernetes rolling update to complete:

```bash
kubectl rollout status deployment/nebula-cart \
-n nebula-cart
```

---

# Complete CI/CD Workflow

```text
Developer Pushes Code
        |
        v
GitHub
        |
        v
Jenkins
        |
        v
Checkout Source Code
        |
        v
Build Docker Image
        |
        v
Tag Image with BUILD_NUMBER
        |
        v
Push Image to Docker Hub
        |
        v
Update Kubernetes Deployment
        |
        v
Kubernetes Rolling Update
        |
        v
Two New Application Pods
        |
        v
Live NebulaCart Application
```

---

# Kubernetes Deployment

The Kubernetes configuration is stored in:

```text
k8s/
```

The directory contains:

```text
k8s/
├── deployment.yaml
├── kind-config.yaml
├── namespace.yaml
└── service.yaml
```

---

## Kubernetes Namespace

The application runs inside the following namespace:

```text
nebula-cart
```

Create the namespace:

```bash
kubectl apply -f k8s/namespace.yaml
```

Check namespaces:

```bash
kubectl get namespaces
```

---

## Kubernetes Deployment

The application is deployed using a Kubernetes Deployment with two replicas.

Apply the Deployment:

```bash
kubectl apply -f k8s/deployment.yaml
```

Check the Deployment:

```bash
kubectl get deployment -n nebula-cart
```

Expected result:

```text
READY   UP-TO-DATE   AVAILABLE
2/2     2            2
```

Check the Pods:

```bash
kubectl get pods -n nebula-cart
```

Both Pods should be in the `Running` state.

---

## Kubernetes Service

The application is exposed using a Kubernetes NodePort Service.

Apply the Service:

```bash
kubectl apply -f k8s/service.yaml
```

Check the Service:

```bash
kubectl get service -n nebula-cart
```

The application is exposed through:

```text
NodePort: 30080
```

---

# kind Kubernetes Cluster

The Kubernetes cluster was created using kind.

The cluster configuration is stored in:

```text
k8s/kind-config.yaml
```

The configuration maps port `30080` from the kind node to the EC2 host.

Create the cluster:

```bash
kind create cluster \
  --name nebula-cart \
  --config k8s/kind-config.yaml
```

Check the cluster:

```bash
kind get clusters
```

Check Kubernetes nodes:

```bash
kubectl get nodes
```

Expected result:

```text
nebula-cart-control-plane   Ready
```

---

# Kubernetes Verification

Check all resources:

```bash
kubectl get all -n nebula-cart
```

Check Pods:

```bash
kubectl get pods -n nebula-cart
```

Check Deployment:

```bash
kubectl get deployment nebula-cart -n nebula-cart
```

Check Service:

```bash
kubectl get service -n nebula-cart
```

Check the deployed Docker image:

```bash
kubectl describe deployment nebula-cart \
  -n nebula-cart | grep Image
```

Example:

```text
Image: arslanlinux/nebula-cart:3
```

---

# Successful CI/CD Deployment

The complete CI/CD pipeline was successfully tested.

During Jenkins Build #3:

```text
Jenkins Build #3
        |
        v
Build arslanlinux/nebula-cart:3
        |
        v
Push to Docker Hub
        |
        v
Update Kubernetes Deployment
        |
        v
Kubernetes Rolling Update
        |
        v
2 New Pods Running
        |
        v
NebulaCart Live
```

The Kubernetes Deployment was successfully updated to:

```text
arslanlinux/nebula-cart:3
```

The two application replicas were confirmed to be running.

The application was also successfully accessed through the AWS EC2 public IP using NodePort `30080`.

---

# Application Access

The deployed application is accessible using:

```text
http://YOUR_EC2_PUBLIC_IP:30080
```

Example:

```text
http://54.91.122.176:30080
```

The application was successfully tested through the EC2 public IP and confirmed to be running from Kubernetes.

---

# Project Structure

```text
nebula_cart/
├── public/
├── src/
│   ├── components/
│   ├── context/
│   ├── data/
│   ├── hooks/
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
├── k8s/
│   ├── deployment.yaml
│   ├── kind-config.yaml
│   ├── namespace.yaml
│   └── service.yaml
├── Dockerfile
├── docker-compose.yml
├── Jenkinsfile
├── package.json
├── package-lock.json
└── README.md
```

---

# Key DevOps Concepts Demonstrated

This project provided practical experience with:

- Git and GitHub
- Docker containerisation
- Docker multi-stage builds
- Docker image tagging
- Docker Hub
- Jenkins CI/CD
- Jenkins credentials
- Jenkins build numbers
- Automated Docker image publishing
- Kubernetes Namespaces
- Kubernetes Deployments
- Kubernetes ReplicaSets
- Kubernetes Pods
- Kubernetes Services
- Kubernetes NodePort
- Kubernetes rolling updates
- kind Kubernetes clusters
- kubectl
- AWS EC2
- End-to-end CI/CD automation

---

# Project Outcome

This project demonstrates a complete end-to-end DevOps CI/CD workflow:

```text
Code Change
     |
     v
GitHub
     |
     v
Jenkins
     |
     v
Docker Build
     |
     v
Versioned Docker Image
     |
     v
Docker Hub
     |
     v
Kubernetes Deployment
     |
     v
Rolling Update
     |
     v
2 Running Replicas
     |
     v
Live Application
```

A new code change can be processed through the CI/CD pipeline, packaged into a versioned Docker image, pushed to Docker Hub, and automatically deployed to Kubernetes.

---

# Author

**Muhammad Arslan Tahir**

DevOps Engineering Learner

GitHub:

```text
https://github.com/arslantahir212
```

---

## License

This project is provided as-is for demonstration and learning purposes.
