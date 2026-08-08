# React + Vite

This template provides a minimal setup to get React working in Vite with HMR and some ESLint rules.

Currently, two official plugins are available:

- [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react) uses [Oxc](https://oxc.rs)
- [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react-swc) uses [SWC](https://swc.rs/)

## React Compiler

The React Compiler is not enabled on this template because of its impact on dev & build performances. To add it, see [this documentation](https://react.dev/learn/react-compiler/installation).

## Expanding the ESLint configuration

If you are developing a production application, we recommend using TypeScript with type-aware lint rules enabled. Check out the [TS template](https://github.com/vitejs/vite/tree/main/packages/create-vite/template-react-ts) for information on how to integrate TypeScript and [`typescript-eslint`](https://typescript-eslint.io) in your project.


## 1. Introduction to Kubernetes

## What is Kubernetes?

Kubernetes (K8s) is an open-source container orchestration platform used to deploy, manage, scale, and monitor containerized applications automatically. It helps organizations run applications reliably across multiple servers by managing containers efficiently.
Why Kubernetes
As applications grow, managing containers manually becomes difficult. Kubernetes solves these challenges by automating container operations.
It provides:
•	Automated deployment of applications 
•	Automatic scaling based on workload 
•	Self-healing by restarting failed containers 
•	Load balancing to distribute traffic 
•	Rolling updates with zero downtime 
•	Easy rollback to previous versions 
•	Service discovery between applications 
•	High availability and fault tolerance

## Problems Before Kubernetes

## 1. Manual Deployment
Deploying applications manually takes time and increases the chance of human errors.
Problem:
•	Manual installation 
•	Manual configuration 
•	Time-consuming deployments 
•	Higher risk of mistakes

## 2. Problems Solved by Kubernetes

Kubernetes solves the common challenges of running containerized applications by automating deployment, scaling, networking, and recovery.
Problems & Solutions
•	Manual Deployment → Automates application deployment using YAML files. 
•	No Auto Scaling → Automatically scales Pods based on workload. 
•	Application Failure → Restarts failed Pods automatically (Self-Healing). 
•	Uneven Traffic → Distributes traffic using Load Balancing. 
•	Downtime During Updates → Performs Rolling Updates with zero downtime. 
•	Failed Deployment → Supports Rollback to the previous stable version. 
•	Changing Pod IPs → Provides stable networking through Services. 
•	Data Loss → Uses Persistent Volumes (PV) and Persistent Volume Claims (PVC). 
•	Configuration Management → Stores configuration in ConfigMaps. 
•	Secret Management → Securely stores passwords, API keys, and tokens using Secrets.


## 3. Kubernetes Architecture

Kubernetes architecture consists of two main components:

1.	Control Plane (Master Node) – Manages the entire Kubernetes cluster. 

2.	Worker Node – Runs the containerized applications (Pods).

![Kubernetes Architecture](./images/Copilot_20260803_232806.png)

Control Plane (Master Node)
The Control Plane manages the entire Kubernetes cluster.

## 1. API Server
•	Entry point of Kubernetes. 
•	Receives all requests from kubectl. 
•	Communicates with all cluster components. 

## 2. etcd
•	Key-value database. 
•	Stores cluster configuration and state. 
3. Scheduler
•	Selects the best Worker Node for new Pods. 
•	Places Pods based on available CPU and Memory. 

## 4. Controller Manager

•	Maintains the desired state. 
•	Restarts failed Pods automatically. 
•	Ensures replicas are always running.

## 5. Cloud Controller Manager

•	Integrates Kubernetes with cloud providers like AWS. 
•	Creates cloud resources such as: 
o	Load Balancer 
o	Volumes 
o	Routes 
o	Nodes 

## Worker Node (Data Plane)
Each Worker Node runs the application containers.

## 1. kubelet
•	Agent running on every Worker Node. 
•	Receives instructions from the API Server. 
•	Starts and monitors Pods. 

## 2. kube-proxy
•	Handles networking. 
•	Routes traffic to the correct Pods. 
•	Provides Service networking and load balancing.

## 3. Container Runtime
## Runs the containers.
## Examples:
•	containerd 
•	CRI-O 
•	Docker (older Kubernetes versions) 

## 4. Pods
•	Smallest deployable unit. 
•	Runs one or more containers.

Kubernetes Installation Using Minikube (Step-by-Step)
	Minikube is the best option for beginners because it creates a single-node Kubernetes cluster on your local machine.
	Prerequisites
	Ubuntu 22.04 / 24.04 
	2 CPU 
	4 GB RAM 
	Internet Connection 
	Sudo User
Step 1: Update the System
sudo apt update
sudo apt upgrade -y
Verify:
sudo apt update



Step 2: Install Docker

sudo apt install docker.io -y
Start Docker
sudo systemctl start docker
Enable Docker
sudo systemctl enable docker
Check Status
sudo systemctl status docker
Check Version
docker --version
Example Output
Docker version 28.x.x

Step 3: Add User to Docker Group
sudo usermod -aG docker $USER
Apply Changes
newgrp docker
Test Docker
docker run hello-world

Step 4: Install kubectl

Download kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
Install kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
Verify
kubectl version --client

Step 5: Install Minikube
Download Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
Install
sudo install minikube-linux-amd64 /usr/local/bin/minikube
Verify Installation
minikube version
Example Output
minikube version: v1.38.1


Step 6: Start Minikube Cluster
Using Docker Driver
minikube start --driver=docker
If Docker is the default driver, you can also run:
minikube start


Step 7: Verify Cluster
Cluster Information
kubectl cluster-info
Check Nodes
kubectl get nodes
Example Output
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   2m    v1.35.x
Check Pods
kubectl get pods -A

Step 8: Create a Deployment

kubectl create deployment nginx --image=nginx

Verify

kubectl get deployments

Check Pods

kubectl get pods

Step 9: Expose the Deployment

kubectl expose deployment nginx --type=NodePort --port=80

Check Service

kubectl get svc

Example Output
NAME         TYPE       CLUSTER-IP      PORT(S)
nginx        NodePort   10.96.120.25    80:30080/TCP

Step 10: Access the Application
Open the service in your browser:

minikube service nginx

Or get the URL:

minikube service nginx --url


Step 11: Scale the Deployment
 	Increase replicas to 3:

kubectl scale deployment nginx --replicas=3

Verify
kubectl get pods

Step 12: View Logs

      kubectl logs <pod-name>
Example
kubectl logs nginx-6f7d9f9f7d-abcde


Step 13: Describe Resources
Describe Pod
kubectl describe pod <pod-name>
Describe Deployment
kubectl describe deployment nginx
Describe Service
kubectl describe svc nginx
Step 14: Delete Resources
Delete Deployment
kubectl delete deployment nginx
Delete Service
kubectl delete svc nginx

Step 15: Stop Minikube
minikube stop

Step 16: Delete Minikube Cluster

	minikube delete

Useful Commands

kubectl cluster-info

kubectl get nodes

kubectl get pods

kubectl get deployments

kubectl get svc

kubectl get namespaces

kubectl describe pod <pod-name>

kubectl logs <pod-name>

kubectl delete pod <pod-name>

minikube status

minikube dashboard


Note:
This project uses Minikube, which is suitable for learning, development, and testing on a local machine. It helps beginners understand Kubernetes concepts such as Pods, Deployments, Services, and Scaling before moving to a production environment. In production, organizations typically use managed Kubernetes services such as Amazon EKS, Azure AKS, or Google GKE.

