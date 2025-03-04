# User Manual: Connecting to a Kubernetes Cluster and Deploying Jupyter Notebook with PyTorch on NVIDIA DGX

Stockholm Medical Artificial Intelligence and Learning Environments (SMAILE) uses a **Kubernetes (K8S) cluster** hosted in the **Karolinska Institutet (KI) data centre**. This cluster is **not accessible through the internet**, meaning users must either:
- Be on the **KI network**, or
- Use the **KI VPN** if accessing remotely.

## 1. Introduction to Kubernetes and Containers

### What is Kubernetes?
Kubernetes (K8s) is an open-source container orchestration platform that automates deploying, managing, and scaling applications. It enables efficient use of resources by distributing workloads across clusters of machines.

### Benefits of Kubernetes
- **Scalability:** Applications can be scaled up or down automatically.
- **Self-Healing:** Kubernetes restarts failed containers and replaces them when needed.
- **Resource Efficiency:** Ensures optimal use of CPU, memory, and GPUs.
- **Load Balancing:** Distributes network traffic across pods efficiently.
- **Declarative Configuration:** Applications are described in YAML manifests, making them reproducible.

### What are Containers?
Containers package applications and their dependencies into isolated environments, making them portable across different computing environments. Popular container runtime environments include Docker.

## 2. Introduction to Docker and Dockerfiles

### What is Docker?
Docker is a platform for developing, shipping, and running applications in containers. Containers ensure that applications run the same way regardless of where they are deployed.

### Using Default vs. Custom Docker Images
For deployments, you can either use a **default container image** such as `nvcr.io/nvidia/pytorch:23.10-py3`, which includes the necessary dependencies, or **create a custom Docker image** tailored to specific development needs.

### Understanding Dockerfiles
A Dockerfile is a script containing a set of instructions to assemble a container image. It defines:
- **Base image:** The starting point (e.g., Ubuntu, NVIDIA PyTorch image).
- **Dependencies:** Software, libraries, and tools required.
- **Commands:** Instructions to run the application (e.g., Jupyter Notebook).

## 3. Introduction to Helm and Helm Charts

### What is Helm?
Helm is a package manager for Kubernetes, similar to apt for Ubuntu or pip for Python. It simplifies the deployment of applications by managing Kubernetes resources using Helm Charts.

### What are Helm Charts?
A Helm Chart is a pre-configured template for deploying applications on Kubernetes. It enables version control, upgrades, and rollbacks.

### Benefits of Using Helm
- **Simplifies deployment** by providing predefined templates.
- **Ensures consistency** across different environments.
- **Facilitates upgrades and rollbacks** of applications.

For more information, visit the [Helm installation guide](https://helm.sh/docs/intro/install/).

## 4. Understanding `values.yaml`
The `values.yaml` file defines configuration options for the Helm chart. Key parameters include:

- **`image.repository`**: Specifies the container image to use (e.g., `nvcr.io/nvidia/pytorch`).
- **`service.port.port`**: Defines the Jupyter Notebook service port (8888 by default).
- **`resources.limits`**: Specifies resource limits such as GPU, CPU, and memory allocation.
- **`persistence`**: Configures storage settings, including mount path and disk size.
- **`probes.liveness.enabled`**: Enables or disables liveness probes for health checks.

Custom configurations can be applied by modifying `values.yaml` or creating an override file.

## 5. Connecting to the SMAILE K8S Cluster
To connect to the **SMAILE Kubernetes Cluster**, you must either:
- Be on the **KI network** or
- Use the **KI VPN** if accessing remotely.

## 6. Managing Kubernetes Credentials
Users receive Kubernetes credentials as a YAML file, typically named `kubeconfig.yaml`. To use it, place it in the appropriate directory:

```sh
mkdir -p ~/.kube
mv kubeconfig.yaml ~/.kube/config
export KUBECONFIG=~/.kube/config
```

For Windows (PowerShell):

```powershell
mkdir $HOME\.kube
Move-Item kubeconfig.yaml $HOME\.kube\config
$env:KUBECONFIG="$HOME\.kube\config"
```

To test that Kubernetes is correctly configured, run:

```sh
kubectl get pods
```

If the connection is successful, it will list the running pods in the cluster.

## 7. Quick Start Guide

### Step 1: Install Chocolatey (Windows only)
Chocolatey is a package manager for Windows, useful for installing software such as kubectl and Helm.

To install Chocolatey, open PowerShell as Administrator and run:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

### Step 2: Install kubectl
Kubectl is the command-line tool for interacting with Kubernetes clusters.

#### Windows:
Follow the official installation guide: [Install kubectl on Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)

#### macOS:
```sh
brew install kubectl
```

#### Ubuntu:
```sh
sudo apt update && sudo apt install -y kubectl
```

After installation, verify it is working by running:

```sh
kubectl version --client
```

### Step 3: Copy Kubernetes Credentials
Once `kubectl` is installed, copy your `kubeconfig.yaml` file to the appropriate directory:

```sh
mkdir -p ~/.kube
mv kubeconfig.yaml ~/.kube/config
export KUBECONFIG=~/.kube/config
```

For Windows:

```powershell
mkdir $HOME\.kube
Move-Item kubeconfig.yaml $HOME\.kube\config
$env:KUBECONFIG="$HOME\.kube\config"
```

To verify that credentials work, run:

```sh
kubectl get pods
```

If the connection is successful, it will list the running pods.

### Step 4: Install Helm
Helm simplifies application deployment in Kubernetes.

#### Windows:
```powershell
choco install kubernetes-helm
```

#### macOS:
```sh
brew install helm
```

#### Ubuntu:
```sh
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Step 5: Add Helm Repository
Before installing applications, add the Helm repository:

```sh
helm repo add nvidia-charts https://ki-smile.github.io/nvidia-charts
helm repo update
```

### Scaling the Deployment

#### Scale Down (Release GPU Resources)
```sh
kubectl scale deployment rel-jupyter --replicas=0
```

#### Scale Up (Resume Work)
```sh
kubectl scale deployment rel-jupyter --replicas=1
```

## 8. Summary
By following this guide, you can efficiently deploy and manage Jupyter Notebook instances on Kubernetes clusters powered by NVIDIA GPUs. The guide provides flexibility to either use prebuilt images or create custom ones as needed.
