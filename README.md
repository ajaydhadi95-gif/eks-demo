# Kubernetes (K8s) – Complete Learning & AWS EKS CI/CD Guide

![Kubernetes](https://img.shields.io/badge/Kubernetes-Container%20Orchestration-326CE5?logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Container-2496ED?logo=docker&logoColor=white)
![AWS EKS](https://img.shields.io/badge/AWS-EKS-FF9900?logo=amazonaws&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?logo=jenkins&logoColor=white)

## Table of Contents

1. [Introduction to Kubernetes](#1-introduction-to-kubernetes)
2. [Why Kubernetes?](#2-why-kubernetes)
3. [Problems Solved by Kubernetes](#3-problems-solved-by-kubernetes)
4. [Kubernetes Architecture](#4-kubernetes-architecture)
5. [Core Kubernetes Components](#5-core-kubernetes-components)
6. [Kubernetes Installation Using Minikube](#6-kubernetes-installation-using-minikube)
7. [Deploy an NGINX Application on Minikube](#7-deploy-an-nginx-application-on-minikube)
8. [Scaling, Logs and Troubleshooting](#8-scaling-logs-and-troubleshooting)
9. [Cleanup](#9-cleanup)
10. [Useful Kubernetes Commands](#10-useful-kubernetes-commands)
11. [Minikube vs Production Kubernetes](#11-minikube-vs-production-kubernetes)
12. [AWS EKS Production-Style CI/CD Architecture](#12-aws-eks-production-style-cicd-architecture)
13. [GitHub to Jenkins CI/CD Flow](#13-github-to-jenkins-cicd-flow)
14. [Build Docker Image](#14-build-docker-image)
15. [Push Image to Amazon ECR](#15-push-image-to-amazon-ecr)
16. [Deploy Application to Amazon EKS](#16-deploy-application-to-amazon-eks)
17. [Kubernetes Service](#17-kubernetes-service)
18. [Ingress and AWS Load Balancer Controller](#18-ingress-and-aws-load-balancer-controller)
19. [OIDC and IRSA](#19-oidc-and-irsa)
20. [Application Load Balancer](#20-application-load-balancer)
21. [Route 53](#21-route-53)
22. [End-to-End Request Flow](#22-end-to-end-request-flow)
23. [Production Notes](#23-production-notes)

---

# 1. Introduction to Kubernetes

## What is Kubernetes?

**Kubernetes (K8s)** is an open-source container orchestration platform used to deploy, manage, scale, and operate containerized applications.

It helps organizations run applications reliably across multiple machines by automating container scheduling, networking, scaling, service discovery, and recovery.

## Why Kubernetes?

As applications grow, manually managing containers becomes difficult. Kubernetes automates common operational tasks such as:

- Automated application deployment
- Scaling of workloads
- Self-healing of failed Pods
- Service discovery and networking
- Load balancing
- Rolling updates
- Rollbacks
- High availability
- Configuration and secret management

---

# 2. Why Kubernetes?

Before Kubernetes, containerized applications were often managed manually.

### Common problems

- Manual installation and configuration
- Difficult application deployments
- No automatic recovery
- Difficult scaling
- Changing container IP addresses
- Traffic distribution problems
- Downtime during application updates
- Configuration management challenges
- Secret management challenges

Kubernetes addresses these problems by allowing the desired state of an application to be defined declaratively, usually through YAML manifests.

---

# 3. Problems Solved by Kubernetes

| Problem | Kubernetes Solution |
|---|---|
| Manual deployment | Deployments and YAML manifests |
| No automatic scaling | Horizontal Pod Autoscaler and replica management |
| Application failure | Self-healing and controller reconciliation |
| Uneven traffic | Services and load-balancing mechanisms |
| Downtime during updates | Rolling updates |
| Failed deployment | Rollout history and rollback |
| Changing Pod IPs | Kubernetes Services |
| Persistent application data | Persistent Volumes and Persistent Volume Claims |
| Application configuration | ConfigMaps |
| Sensitive configuration | Secrets |

> **Note:** Kubernetes provides the mechanisms for high availability and rolling updates, but the actual availability and zero-downtime behavior depend on the application design, replica count, readiness probes, infrastructure, and deployment strategy.

---

# 4. Kubernetes Architecture

Kubernetes can be understood as two major areas:

```text
                    Kubernetes Cluster
                           |
             +-------------+-------------+
             |                           |
       Control Plane                  Worker Nodes
       (Management)                   (Data Plane)
             |                           |
    +--------+--------+            +-----+------+
    |        |        |            |            |
 API Server  etcd  Scheduler    kubelet     kube-proxy
    |                 |              |
    |           Controller Manager   Pods
    |
 Cloud Controller Manager
```

## Control Plane

The Control Plane manages the Kubernetes cluster.

### 1. API Server

- Main entry point to Kubernetes.
- Receives requests from `kubectl`, controllers, and other clients.
- Validates and processes Kubernetes API requests.
- Communicates with other cluster components.

### 2. etcd

- Distributed key-value store.
- Stores Kubernetes cluster state and configuration.
- The API Server uses etcd as the cluster's persistent state store.

### 3. Scheduler

- Selects a suitable Worker Node for newly created Pods.
- Considers resource availability and scheduling constraints.

### 4. Controller Manager

- Runs controllers that continuously compare actual state with desired state.
- Helps maintain the desired number of replicas.
- Detects and responds to changes in cluster resources.

### 5. Cloud Controller Manager

Used in cloud environments to integrate Kubernetes with cloud-provider APIs.

For AWS-based Kubernetes deployments, cloud integrations can manage resources such as:

- Load balancers
- Cloud volumes
- Cloud networking resources
- Node-related cloud resources

---

# 5. Core Kubernetes Components

## Worker Node

A Worker Node runs application workloads.

### kubelet

- Agent running on each Worker Node.
- Communicates with the Kubernetes API Server.
- Ensures containers described by Pod specifications are running.

### kube-proxy

- Provides networking support for Kubernetes Services.
- Helps route network traffic to appropriate Pods.

### Container Runtime

Runs containers through the Kubernetes Container Runtime Interface (CRI).

Common runtimes include:

- `containerd`
- `CRI-O`

> Docker Engine was commonly used with older Kubernetes setups through integrations such as Docker Shim. Modern Kubernetes installations generally use CRI-compatible runtimes such as containerd or CRI-O.

### Pod

A **Pod** is the smallest deployable unit in Kubernetes.

A Pod can contain one or more containers that share networking and storage resources.

---

# 6. Kubernetes Installation Using Minikube

Minikube is useful for learning, local development, and testing Kubernetes concepts on a local machine.

## Prerequisites

Recommended starting requirements:

- Ubuntu 22.04 / 24.04
- At least 2 CPUs
- At least 4 GB RAM
- Internet connection
- Sudo privileges
- Docker installed

> Resource requirements can vary depending on the workloads and Minikube driver.

---

## Step 1: Update the System

```bash
sudo apt update
sudo apt upgrade -y
```

Verify:

```bash
sudo apt update
```

---

## Step 2: Install Docker

```bash
sudo apt install docker.io -y
```

Start Docker:

```bash
sudo systemctl start docker
```

Enable Docker at boot:

```bash
sudo systemctl enable docker
```

Check Docker:

```bash
sudo systemctl status docker
docker --version
```

Example:

```text
Docker version 28.x.x
```

---

## Step 3: Add User to Docker Group

This allows the current user to use Docker without `sudo`.

```bash
sudo usermod -aG docker $USER
```

Apply the group change:

```bash
newgrp docker
```

Test Docker:

```bash
docker run hello-world
```

---

## Step 4: Install kubectl

Download the current stable Linux AMD64 client:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Install:

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Verify:

```bash
kubectl version --client
```

---

## Step 5: Install Minikube

Download Minikube:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

Install:

```bash
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Verify:

```bash
minikube version
```

Example:

```text
minikube version: v1.38.1
```

The exact version may change because the installation uses the latest available release.

---

# 7. Deploy an NGINX Application on Minikube

## Step 1: Start Minikube

Using Docker as the driver:

```bash
minikube start --driver=docker
```

If Docker is already configured as the default driver:

```bash
minikube start
```

---

## Step 2: Verify the Cluster

Check cluster information:

```bash
kubectl cluster-info
```

Check Nodes:

```bash
kubectl get nodes
```

Example:

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   2m    v1.35.x
```

Check all Pods:

```bash
kubectl get pods -A
```

---

## Step 3: Create an NGINX Deployment

```bash
kubectl create deployment nginx --image=nginx
```

Verify Deployment:

```bash
kubectl get deployments
```

Check Pods:

```bash
kubectl get pods
```

Expected state:

```text
STATUS
Running
```

---

## Step 4: Expose the Deployment

Create a NodePort Service:

```bash
kubectl expose deployment nginx --type=NodePort --port=80
```

Check the Service:

```bash
kubectl get svc
```

Example:

```text
NAME    TYPE       CLUSTER-IP      PORT(S)
nginx   NodePort   10.96.120.25    80:30080/TCP
```

> The NodePort value is dynamically assigned unless explicitly specified. Do not assume it will always be `30080`.

---

## Step 5: Access the Application

Open the application automatically:

```bash
minikube service nginx
```

Or retrieve the URL:

```bash
minikube service nginx --url
```

Open the returned URL in a browser.

---

# 8. Scaling, Logs and Troubleshooting

## Scale the Deployment

Increase replicas to 3:

```bash
kubectl scale deployment nginx --replicas=3
```

Verify:

```bash
kubectl get pods
```

You should see three NGINX Pods managed by the Deployment.

---

## View Pod Logs

First get the Pod name:

```bash
kubectl get pods
```

Then:

```bash
kubectl logs <pod-name>
```

Example:

```bash
kubectl logs nginx-6f7d9f9f7d-abcde
```

---

## Describe Resources

Describe a Pod:

```bash
kubectl describe pod <pod-name>
```

Describe Deployment:

```bash
kubectl describe deployment nginx
```

Describe Service:

```bash
kubectl describe svc nginx
```

These commands are especially useful when troubleshooting:

- Pending Pods
- CrashLoopBackOff
- ImagePullBackOff
- Failed scheduling
- Service connectivity
- Configuration problems

---

# 9. Cleanup

Delete the Deployment:

```bash
kubectl delete deployment nginx
```

Delete the Service:

```bash
kubectl delete svc nginx
```

Stop Minikube:

```bash
minikube stop
```

Delete the Minikube cluster completely:

```bash
minikube delete
```

---

# 10. Useful Kubernetes Commands

## Cluster

```bash
kubectl cluster-info
kubectl get nodes
kubectl get namespaces
```

## Workloads

```bash
kubectl get pods
kubectl get deployments
kubectl get replicasets
```

## Services

```bash
kubectl get svc
kubectl describe svc <service-name>
```

## Troubleshooting

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events
kubectl delete pod <pod-name>
```

## Minikube

```bash
minikube status
minikube dashboard
```

---

# 11. Minikube vs Production Kubernetes

Minikube is primarily intended for local learning and development.

For production workloads, organizations commonly use managed Kubernetes services such as:

- Amazon EKS
- Azure AKS
- Google GKE

This project therefore uses Minikube to learn the Kubernetes fundamentals and then demonstrates a production-style deployment flow using Amazon EKS.

---

# 12. AWS EKS Production-Style CI/CD Architecture

The production-style flow covered in this project is:

```text
Developer
    |
    | git push
    v
GitHub Repository
    |
    | Webhook
    v
Jenkins
    |
    | Build & Test
    v
Docker Image
    |
    | Push
    v
Amazon ECR
    |
    | Pull Image
    v
Amazon EKS
    |
    +--> Deployment
    |       |
    |       v
    |     Pods
    |
    +--> Service
    |
    +--> Ingress
             |
             v
AWS Load Balancer Controller
             |
             v
Application Load Balancer
             |
             v
Route 53
             |
             v
Users
```

A multi-tier application can extend this architecture:

```text
Users
  |
  v
Route 53
  |
  v
Application Load Balancer
  |
  v
Ingress
  |
  +----------------------+
  |                      |
  v                      v
Frontend Service     Backend Service
  |                      |
  v                      v
Frontend Pods         Backend Pods
                         |
                         v
                    Amazon RDS
```

---

# 13. GitHub to Jenkins CI/CD Flow

## Step 1: Developer Pushes Code

```bash
git add .
git commit -m "Initial Commit"
git push origin main
```

A configured GitHub webhook can trigger Jenkins automatically after the push.

---

## Step 2: Jenkins Pipeline Starts

A simplified Jenkinsfile structure:

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/username/repository.git'
            }
        }

        // Build, test, image push and deployment stages
        // can be added here.
    }
}
```

> Replace the repository URL with your actual GitHub repository URL. In production, credentials and repository configuration should be managed securely through Jenkins credentials and pipeline configuration.

---

# 14. Build Docker Image

Jenkins builds a Docker image using the project's Dockerfile.

Example:

```bash
docker build -t frontend:v1 .
```

Verify:

```bash
docker images
```

For CI/CD, using an immutable tag such as the Jenkins `BUILD_NUMBER` or Git commit SHA is preferable to relying only on `v1` or `latest`.

---

# 15. Push Image to Amazon ECR

## Step 1: Authenticate Docker with ECR

Replace `<ACCOUNT_ID>` with the AWS account ID.

```bash
aws ecr get-login-password --region ap-south-1 | \
docker login --username AWS --password-stdin \
<ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com
```

## Step 2: Tag the Image

```bash
docker tag frontend:v1 \
<ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/frontend:v1
```

## Step 3: Push the Image

```bash
docker push \
<ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/frontend:v1
```

Verify the image in the Amazon ECR repository.

> The ECR repository must exist before pushing unless your CI/CD pipeline creates it automatically.

---

# 16. Deploy Application to Amazon EKS

## Connect kubectl to EKS

Example cluster:

```text
Cluster: demo-cluster
Region:  ap-south-1
```

Update kubeconfig:

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name demo-cluster
```

Verify:

```bash
kubectl get nodes
```

---

## Deploy the Application

If the repository contains Kubernetes manifests:

```bash
kubectl apply -f deployment.yaml
```

Verify:

```bash
kubectl get deployments
kubectl get pods
```

For a production project, the Deployment should normally specify:

- Container image
- Replica count
- Container port
- Resource requests/limits
- Readiness probe
- Liveness probe
- Image pull policy
- Environment/configuration references

---

# 17. Kubernetes Service

A Service provides a stable network endpoint for a set of Pods.

Apply the Service:

```bash
kubectl apply -f service.yaml
```

Verify:

```bash
kubectl get svc
```

Typical Service types include:

- `ClusterIP` – internal cluster communication
- `NodePort` – exposes a port on each node
- `LoadBalancer` – integrates with a cloud load balancer
- `ExternalName` – maps a Service to an external DNS name

For the EKS architecture in this project, the Service is used as the stable backend for the Ingress/controller routing layer.

---

# 18. Ingress and AWS Load Balancer Controller

## What is Ingress?

Ingress defines HTTP/HTTPS routing rules for applications inside a Kubernetes cluster.

For example:

```text
example.com/
       |
       v
Frontend Service

example.com/api
       |
       v
Backend Service
```

Create the Ingress:

```bash
kubectl apply -f ingress.yaml
```

Verify:

```bash
kubectl get ingress
```

> **Important:** An Ingress resource is a set of routing rules. It does not itself process network traffic. An Ingress Controller watches the Ingress resource and implements those rules. In this AWS architecture, the AWS Load Balancer Controller provisions and configures an AWS Application Load Balancer.

---

# 19. OIDC and IRSA

## What is OIDC?

Amazon EKS can use an OIDC identity provider to establish trust between Kubernetes workloads and AWS IAM.

This enables **IAM Roles for Service Accounts (IRSA)**.

IRSA allows a Kubernetes ServiceAccount to assume an AWS IAM role so that Pods can access AWS resources without storing long-lived AWS access keys inside the container.

---

## Associate the IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --region ap-south-1 \
  --cluster demo-cluster \
  --approve
```

Verify the OIDC association using AWS/EKS tooling if required.

---

# 20. AWS Load Balancer Controller

The AWS Load Balancer Controller watches Kubernetes resources such as Ingress and Service resources and creates/configures AWS load-balancing resources.

## Step 1: Create IAM Policy

Create the AWS Load Balancer Controller IAM policy according to the current AWS Load Balancer Controller documentation.

Example policy name:

```text
AWSLoadBalancerControllerIAMPolicy
```

The policy ARN used below assumes the policy already exists.

---

## Step 2: Create IAM Service Account

```bash
eksctl create iamserviceaccount \
  --cluster demo-cluster \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

Replace:

```text
<ACCOUNT_ID>
```

with your AWS account ID.

---

## Step 3: Install Helm Repository

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

---

## Step 4: Install AWS Load Balancer Controller

```bash
helm install aws-load-balancer-controller \
  eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

---

## Step 5: Verify Controller

```bash
kubectl get pods -n kube-system
```

Check the Deployment:

```bash
kubectl get deployment \
  aws-load-balancer-controller \
  -n kube-system
```

The Controller Pods should eventually reach `Running` and become Ready.

---

# 21. Application Load Balancer

Once:

1. The AWS Load Balancer Controller is installed,
2. IAM/OIDC configuration is correct, and
3. A valid AWS Ingress resource is applied,

the controller can provision/configure an Application Load Balancer according to the Ingress configuration.

Check the Ingress:

```bash
kubectl get ingress
```

For more details:

```bash
kubectl describe ingress <ingress-name>
```

The ALB DNS name will normally appear in the Ingress status.

---

# 22. Route 53

Route 53 can be used to provide a friendly domain name for the application.

Example:

```text
www.example.com
       |
       v
Application Load Balancer
```

Create an appropriate Route 53 record, commonly an **Alias A/AAAA record**, pointing to the ALB.

The exact record configuration depends on whether the application uses:

- Root domain
- Subdomain
- IPv4/IPv6
- HTTPS
- ACM certificate

---

# 23. End-to-End Request Flow

The complete request flow for the production-style architecture is:

```text
User
 |
 | HTTPS request
 v
Route 53
 |
 | DNS resolution
 v
Application Load Balancer
 |
 | Listener / target routing
 v
AWS Load Balancer Controller-managed routing
 |
 v
Kubernetes Service
 |
 v
Frontend Pods
 |
 | Internal Kubernetes networking
 v
Backend Service
 |
 v
Backend Pods
 |
 | Database connection
 v
Amazon RDS
 |
 v
Response
```

### Explanation

1. The user accesses the application using a domain name.
2. Route 53 resolves the domain to the Application Load Balancer.
3. The ALB receives the HTTP/HTTPS request.
4. The AWS Load Balancer Controller implements the Kubernetes Ingress configuration on AWS.
5. The request is routed to the appropriate Kubernetes Service.
6. The Service selects the appropriate application Pods.
7. Frontend Pods can communicate with Backend Pods through Kubernetes networking.
8. Backend Pods can communicate with Amazon RDS when the application requires database access.
9. The response returns through the appropriate networking path to the user.

### Security

OIDC + IRSA allows the AWS Load Balancer Controller to obtain AWS permissions through an IAM role associated with its Kubernetes ServiceAccount rather than embedding long-lived AWS access keys in the Pod.

---

# 24. Production Notes

The Minikube portion of this README is designed for learning and local testing.

For a production EKS implementation, consider adding:

## Kubernetes

- Multiple replicas
- Readiness probes
- Liveness probes
- Startup probes where required
- Resource requests and limits
- Horizontal Pod Autoscaler
- Pod Disruption Budgets
- Namespace separation
- Network Policies
- ConfigMaps
- Secrets
- Persistent storage where required

## AWS

- Multi-AZ architecture
- Private worker-node subnets
- Public subnets for internet-facing load balancers where appropriate
- IAM least privilege
- Security Groups
- VPC and subnet design
- NAT Gateway where required
- Amazon RDS Multi-AZ where appropriate
- CloudWatch monitoring and logging
- AWS Certificate Manager for HTTPS
- Route 53 DNS
- ECR image scanning and lifecycle policies

## CI/CD

Recommended pipeline stages:

```text
Checkout
   |
   v
Build
   |
   v
Unit / Integration Tests
   |
   v
Docker Build
   |
   v
Security / Image Scan
   |
   v
Push to ECR
   |
   v
Deploy to EKS
   |
   v
Rollout Verification
```

For Jenkins, avoid storing AWS access keys directly in the Jenkinsfile. Use Jenkins credentials or, preferably, an appropriate AWS identity mechanism with least-privilege permissions.

---

# Project Summary

This project covers Kubernetes from fundamentals to a production-style AWS EKS workflow.

### Local Kubernetes

- Kubernetes fundamentals
- Control Plane and Worker Nodes
- Pods
- Deployments
- Services
- Scaling
- Logs
- Troubleshooting
- Minikube installation
- NGINX deployment

### AWS EKS

- Amazon EKS
- Amazon ECR
- IAM
- OIDC
- IRSA
- AWS Load Balancer Controller
- Ingress
- Application Load Balancer
- Route 53

### CI/CD

- GitHub
- Jenkins
- Docker
- Amazon ECR
- Kubernetes deployment
- EKS-based application delivery

---

# Final Architecture

```text
                         +----------------+
                         |     Users      |
                         +-------+--------+
                                 |
                                 v
                         +---------------+
                         |   Route 53    |
                         +-------+-------+
                                 |
                                 v
                    +-------------------------+
                    | Application Load        |
                    | Balancer (ALB)          |
                    +-----------+-------------+
                                |
                                v
                    +-------------------------+
                    | Kubernetes Ingress      |
                    | Rules                   |
                    +-----------+-------------+
                                |
                                v
                    +-------------------------+
                    | Kubernetes Service      |
                    +-----------+-------------+
                                |
                       +--------+--------+
                       |                 |
                       v                 v
                 Frontend Pods      Backend Pods
                                         |
                                         v
                                  +-------------+
                                  | Amazon RDS  |
                                  +-------------+

CI/CD:

Developer
   |
   v
GitHub
   |
   | Webhook
   v
Jenkins
   |
   v
Docker Build
   |
   v
Amazon ECR
   |
   v
Amazon EKS
```

---

