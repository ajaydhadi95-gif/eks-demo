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
