# User Manual: Connecting to a Kubernetes Cluster and Deploying Jupyter Notebook with PyTorch on NVIDIA DGX

Stockholm Medical Artificial Intelligence and Learning Environments (SMAILE) uses a **Kubernetes (K8S) cluster** hosted in the **Karolinska Institutet (KI) data centre**. This cluster is **not accessible through the internet**, meaning users must either:
- Be on the **KI network**, or
- Use the **KI VPN** if accessing remotely.

## Table of Contents

1. [Introduction to Kubernetes and Containers](#1-introduction-to-kubernetes-and-containers)
2. [Introduction to Docker and Dockerfiles](#2-introduction-to-docker-and-dockerfiles)
3. [Introduction to Helm and Helm Charts](#3-introduction-to-helm-and-helm-charts)
4. [Understanding values.yaml](#4-understanding-valuesyaml)
5. [Connecting to the SMAILE K8S Cluster](#5-connecting-to-the-smaile-k8s-cluster)
6. [Managing Kubernetes Credentials](#6-managing-kubernetes-credentials)
7. [Installation and Setup Guide](#7-installation-and-setup-guide)
8. [Deploying Jupyter Notebook with PyTorch](#8-deploying-jupyter-notebook-with-pytorch)
9. [Deploying MATLAB](#9-deploying-matlab)
10. [Managing Persistent Storage](#10-managing-persistent-storage)
11. [Common Kubernetes Commands](#11-common-kubernetes-commands)
12. [Troubleshooting](#12-troubleshooting)
13. [Kubernetes Ecosystem and Relationships](#13-kubernetes-ecosystem-and-relationships)
14. [Summary](#14-summary)

## 1. Introduction to Kubernetes and Containers

### What is Kubernetes?
Kubernetes (K8s) is an open-source container orchestration platform that automates deploying, managing, and scaling applications. It enables efficient use of resources by distributing workloads across clusters of machines.

### Key Kubernetes Concepts
- **Pods:** The smallest deployable units in Kubernetes that contain one or more containers
- **Deployments:** Controllers that manage pod lifecycles and enable rolling updates
- **Services:** Network endpoints that expose pods to the network
- **Nodes:** Worker machines (physical or virtual) that run pods
- **Namespaces:** Virtual clusters for resource isolation and organization

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

### Container vs. Virtual Machine
Containers share the host OS kernel but have isolated processes and filesystems. This makes them more lightweight than virtual machines, which include a full OS instance.

### Using Default vs. Custom Docker Images
For deployments, you can either use a **default container image** such as `nvcr.io/nvidia/pytorch:23.10-py3`, which includes the necessary dependencies, or **create a custom Docker image** tailored to specific development needs.

### Understanding Dockerfiles
A Dockerfile is a script containing a set of instructions to assemble a container image. It defines:
- **Base image:** The starting point (e.g., Ubuntu, NVIDIA PyTorch image).
- **Dependencies:** Software, libraries, and tools required.
- **Commands:** Instructions to run the application (e.g., Jupyter Notebook).

### Example Dockerfile for PyTorch with Jupyter
```Dockerfile
FROM nvcr.io/nvidia/pytorch:23.10-py3

# Install additional packages
RUN pip install --no-cache-dir \
    matplotlib \
    pandas \
    scikit-learn \
    seaborn \
    jupyterlab

# Set working directory
WORKDIR /workspace

# Expose Jupyter port
EXPOSE 8888

# Start Jupyter when container launches
CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", "--NotebookApp.token=''", "--NotebookApp.password=''"]
```

### Building and Pushing Docker Images
```sh
# Build the Docker image
docker build -t your-registry/pytorch-jupyter:v1 .

# Push the image to a registry
docker push your-registry/pytorch-jupyter:v1
```

## 3. Introduction to Helm and Helm Charts

### What is Helm?
Helm is a package manager for Kubernetes, similar to apt for Ubuntu or pip for Python. It simplifies the deployment of applications by managing Kubernetes resources using Helm Charts.

### What are Helm Charts?
A Helm Chart is a pre-configured template for deploying applications on Kubernetes. It enables version control, upgrades, and rollbacks.

### Structure of a Helm Chart
```
my-chart/
  ├── Chart.yaml          # Chart metadata
  ├── values.yaml         # Default configuration values
  ├── templates/          # Directory of templates
  │   ├── deployment.yaml
  │   ├── service.yaml
  │   └── ...
  └── charts/             # Optional dependent charts
```

### Benefits of Using Helm
- **Simplifies deployment** by providing predefined templates.
- **Ensures consistency** across different environments.
- **Facilitates upgrades and rollbacks** of applications.

For more information, visit the [Helm installation guide](https://helm.sh/docs/intro/install/).

### Common Helm Commands

```sh
# Install a Helm chart
helm install [RELEASE_NAME] [CHART]

# Install a chart with custom values
helm install [RELEASE_NAME] [CHART] -f [VALUES_FILE]

# Upgrade an existing release
helm upgrade [RELEASE_NAME] [CHART]

# List all releases
helm list

# Uninstall a release
helm uninstall [RELEASE_NAME]
```

## 4. Understanding `values.yaml`
The `values.yaml` file defines configuration options for the Helm chart. Key parameters include:

- **`image.repository`**: Specifies the container image to use (e.g., `nvcr.io/nvidia/pytorch`).
- **`service.port`**: Defines the service port (8888 by default for Jupyter).
- **`resources.limits`**: Specifies resource limits such as GPU, CPU, and memory allocation.
- **`persistence`**: Configures storage settings, including mount path and disk size.
- **`nodeSelector`**: Specifies which nodes the pod should be scheduled on.

Custom configurations can be applied by modifying `values.yaml` or creating an override file.

## 5. Connecting to the SMAILE K8S Cluster
To connect to the **SMAILE Kubernetes Cluster**, you must either:
- Be on the **KI network** or
- Use the **KI VPN** if accessing remotely.

### Setting up VPN Access
For detailed instructions on setting up and using the KI VPN service, please refer to the official KI documentation:
[KI VPN Service Documentation](https://staff.ki.se/tools-and-support/it-and-telephony/tools-for-working-off-campus/vpn-service-ki-vpn)

The documentation covers:
- How to request VPN access
- Installation of the VPN client
- Connection instructions
- Troubleshooting common connection issues

## 6. Managing Kubernetes Credentials
Users receive Kubernetes credentials as a YAML file, typically named `kubeconfig.yaml`. To use it, place it in the appropriate directory:

### Linux/macOS
```sh
mkdir -p ~/.kube
mv kubeconfig.yaml ~/.kube/config
export KUBECONFIG=~/.kube/config
```

### Windows (PowerShell)
```powershell
mkdir -Force $HOME\.kube
Move-Item kubeconfig.yaml $HOME\.kube\config
$env:KUBECONFIG="$HOME\.kube\config"
```

To make the KUBECONFIG environment variable persistent on Windows, you can set it in the system environment variables:
1. Search for "Edit the system environment variables" in Windows search
2. Click "Environment Variables"
3. Under "User variables", click "New"
4. Variable name: `KUBECONFIG`
5. Variable value: `%USERPROFILE%\.kube\config`

To test that Kubernetes is correctly configured, run:

```sh
kubectl get pods
```

If the connection is successful, it will list the running pods in the cluster.

## 7. Installation and Setup Guide

### Step 1: Install Chocolatey (Windows only)
Chocolatey is a package manager for Windows, useful for installing software such as kubectl and Helm.

To install Chocolatey, open PowerShell as Administrator and run:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

### Step 2: Install kubectl
Kubectl is the command-line tool for interacting with Kubernetes clusters.

#### Windows:
Using Chocolatey (recommended):
```powershell
choco install kubernetes-cli
```

Alternatively, follow the official installation guide: [Install kubectl on Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)

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
mkdir -Force $HOME\.kube
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

Verify Helm installation:
```sh
helm version
```

### Step 5: Add Helm Repository
Before installing applications, add the SMAILE Helm repository:

```sh
helm repo add nvidia-charts https://ki-smile.github.io/nvidia-charts
helm repo update
```

## 8. Deploying Jupyter Notebook with PyTorch

### Step 1: Create a Configuration File
Create a custom values file (e.g., `jupyter-values.yaml`) to customize your deployment:

```yaml
# Basic configuration for Jupyter with PyTorch
image:
  repository: nvcr.io/nvidia/pytorch
  tag: 23.10-py3
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 8888

resources:
  limits:
    nvidia.com/gpu: 1
    memory: 16Gi
    cpu: 4
  requests:
    memory: 4Gi
    cpu: 2

persistence:
  enabled: true
  storageClass: "standard"  # Use appropriate storage class
  size: 50Gi
  mountPath: /workspace

# Optional: Set specific node selectors
nodeSelector: {}
```

### Step 2: Deploy Using Helm
Install the Jupyter application using Helm:

```sh
helm install my-jupyter nvidia-charts/jupyter -f jupyter-values.yaml
```

### Step 3: Verify Deployment
Check the status of your deployment:

```sh
kubectl get pods
kubectl get services
```

Wait until the pod status shows `Running`.

### Step 4: Access Jupyter Notebook
Set up port forwarding to access Jupyter Notebook:

```sh
kubectl port-forward service/my-jupyter 8888:8888
```

Access Jupyter Notebook in your browser at `http://localhost:8888`

### Step 5: Managing Your Deployment

#### Scale Down (Release GPU Resources)
When you're not using the notebook, release resources by scaling down:

```sh
kubectl scale deployment my-jupyter --replicas=0
```

#### Scale Up (Resume Work)
When you want to resume work:

```sh
kubectl scale deployment my-jupyter --replicas=1
```

## 9. Deploying MATLAB

### Step 1: Create a Configuration File
Create a custom values file (e.g., `matlab-values.yaml`) to customize your MATLAB deployment:

```yaml
# Basic configuration for MATLAB
image:
  repository: mathworks/matlab
  tag: r2023b
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 8888

resources:
  limits:
    nvidia.com/gpu: 1
    memory: 16Gi
    cpu: 4
  requests:
    memory: 8Gi
    cpu: 2

persistence:
  enabled: true
  storageClass: "standard"  # Use appropriate storage class
  size: 50Gi
  mountPath: /home/matlab

# License server configuration (if needed)
license:
  server: "flexlm.ki.se"  # Replace with your actual license server
  port: 27000

# Optional: Set specific node selectors
nodeSelector: {}
```

### Step 2: Deploy Using Helm
Install the MATLAB application using Helm:

```sh
helm install my-matlab nvidia-charts/matlab -f matlab-values.yaml
```

### Step 3: Verify Deployment
Check the status of your deployment:

```sh
kubectl get pods
kubectl get services
```

Wait until the pod status shows `Running`.

### Step 4: Access MATLAB
Set up port forwarding to access MATLAB:

```sh
kubectl port-forward service/my-matlab 8888:8888
```

Access MATLAB in your browser at `http://localhost:8888`

### Step 5: Managing Your Deployment

#### Scale Down (Release Resources)
When you're not using MATLAB, release resources by scaling down:

```sh
kubectl scale deployment my-matlab --replicas=0
```

#### Scale Up (Resume Work)
When you want to resume work:

```sh
kubectl scale deployment my-matlab --replicas=1
```

## 10. Managing Persistent Storage

### Understanding Persistent Volumes in Kubernetes
Persistent Volumes (PVs) and Persistent Volume Claims (PVCs) provide storage that persists beyond the lifecycle of pods. This is crucial for saving work in Jupyter Notebooks and MATLAB.

### Types of Storage Classes
The SMAILE cluster offers different storage classes, such as:
- **standard**: General-purpose storage
- **ssd**: SSD-backed storage for better performance
- **shared**: Storage accessible from multiple pods simultaneously

You can view available storage classes with:
```sh
kubectl get storageclass
```

### Setting Up Persistent Storage
In your `values.yaml`, ensure persistence is enabled and specify the appropriate storage class:

```yaml
persistence:
  enabled: true
  storageClass: "standard"  # Or "ssd" or "shared" as needed
  size: 50Gi
  mountPath: /workspace  # For Jupyter
  # mountPath: /home/matlab  # For MATLAB
```

### Checking Storage Status
To view your Persistent Volume Claims:
```sh
kubectl get pvc
```

To see detailed information about a specific PVC:
```sh
kubectl describe pvc <pvc-name>
```

### 10.1 Copying Files to a Persistent Volume Claim (PVC)
In Kubernetes, **Persistent Volume Claims (PVCs)** provide storage that persists beyond the lifecycle of pods. If you need to copy files from your local machine to a PVC-mounted directory inside a pod, you can use the `kubectl cp` command.

#### Using `kubectl cp`
The `kubectl cp` command allows you to copy files between your local machine and a running pod.

#### Step 1: Find the Pod Name
To locate the pod using your PVC, run:

```sh
kubectl get pods
```

If you need more details about the storage mounts in a specific pod:

```sh
kubectl describe pod <pod-name>
```

#### Step 2: Copy Files to the Pod
To copy a local file to a pod that has access to the PVC:

```sh
kubectl cp /path/to/local/file <pod-name>:/path/in/pod
```

For example, if your pod is named `my-jupyter` and your PVC is mounted at `/workspace`, you can copy a local dataset like this:

```sh
kubectl cp dataset.csv my-jupyter:/workspace/dataset.csv
```

To copy an entire directory:

```sh
kubectl cp /path/to/local/dir <pod-name>:/workspace/
```

#### Step 3: Verify the File
After copying, log into the pod to verify:

```sh
kubectl exec -it <pod-name> -- ls -l /workspace
```

#### Troubleshooting
* **Permission Issues:** Ensure the target directory in the pod allows write access.
* **File Overwrites:** Existing files may be overwritten without warning.
* **Slow Transfers:** Large files may take time due to Kubernetes network overhead.

This method enables easy file transfers when working with Kubernetes **Persistent Volumes**, improving workflows for data science and research environments.

## 11. Common Kubernetes Commands

### Pod Management
```sh
# List all pods
kubectl get pods

# Get detailed information about a pod
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>

# Execute a command in a pod
kubectl exec -it <pod-name> -- /bin/bash
```

### Deployment Management
```sh
# List all deployments
kubectl get deployments

# Scale a deployment
kubectl scale deployment <deployment-name> --replicas=<number>

# Edit a deployment
kubectl edit deployment <deployment-name>

# Delete a deployment
kubectl delete deployment <deployment-name>
```

### Service Management
```sh
# List all services
kubectl get services

# Get detailed information about a service
kubectl describe service <service-name>

# Access a service locally via port forwarding
kubectl port-forward service/<service-name> <local-port>:<service-port>
```

### Persistent Volume Management
```sh
# List all persistent volume claims
kubectl get pvc

# Get detailed information about a PVC
kubectl describe pvc <pvc-name>

# Delete a PVC (caution: this may delete data)
kubectl delete pvc <pvc-name>
```

### Helm Management
```sh
# List all Helm releases
helm list

# Get information about a release
helm status <release-name>

# Upgrade a release with new values
helm upgrade <release-name> nvidia-charts/<chart-name> -f <values-file>

# Uninstall a release
helm uninstall <release-name>
```

## 12. Troubleshooting

### Common Issues and Solutions

#### Issue: Cannot connect to the cluster
- **Check VPN connection** - Ensure you are connected to the KI VPN
- **Verify kubeconfig** - Make sure your kubeconfig file is correctly located at `~/.kube/config`
- **Check credentials** - Run `kubectl config view` to ensure your configuration is correct
- **Test connectivity** - Try `kubectl cluster-info` to verify cluster connection

#### Issue: Pod stays in "Pending" state
- **Check resource availability** - GPUs might be fully utilized
- **Inspect pod events** - Run `kubectl describe pod <pod-name>` to see the events
- **Verify resource requests** - Your pod might be requesting more resources than available
- **Check node status** - Run `kubectl get nodes` to see if any nodes are available

#### Issue: Cannot access Jupyter or MATLAB
- **Verify port forwarding** - Ensure port forwarding is active
- **Check service status** - Run `kubectl get service <service-name>`
- **Inspect pod logs** - Run `kubectl logs <pod-name>` to check for application errors
- **Check network connectivity** - Make sure no firewall is blocking the connection

#### Issue: Lost data after pod restart
- **Verify persistence configuration** - Ensure persistence is enabled in your values.yaml
- **Check PVC status** - Run `kubectl get pvc` to verify the persistent volume claim exists
- **Inspect mount paths** - Verify the mount path in your container matches your configuration
- **Examine events** - Run `kubectl get events` to see if there were any storage-related issues

### Getting Help
For issues not covered in this troubleshooting guide, please contact your system administrator or refer to the official Kubernetes documentation at [kubernetes.io](https://kubernetes.io/docs/home/).

## 13. Kubernetes Ecosystem and Relationships

The following diagram illustrates the relationships between different components of the Kubernetes ecosystem and the tools used to interact with it:

![Kubernetes Ecosystem and Relationships](https://raw.githubusercontent.com/ki-smile/nvidia-charts/main/k8s-concepts-diagram.svg)


### Key Relationships Explained

- **Docker** builds container images used by pods
- **Helm** simplifies deployment of applications to Kubernetes using pre-configured charts
- **kubectl** is the command-line tool for interacting directly with the cluster
- **Deployments** manage multiple pod replicas and handle rolling updates
- **Services** expose pods to the network via stable endpoints
- **Persistent Volumes** provide storage that persists beyond pod lifecycles

## 14. Summary
By following this guide, you can efficiently deploy and manage Jupyter Notebook and MATLAB instances on the SMAILE Kubernetes cluster powered by NVIDIA GPUs.

Key points to remember:
- Connect to the KI VPN when accessing the cluster remotely
- Use Helm charts from the nvidia-charts repository for simplified deployment
- Properly configure persistent storage to avoid data loss
- Scale down deployments when not in use to free up resources for other users
- Understand basic Kubernetes commands for managing your deployment

For more information, refer to the official documentation:
- [SMAILE Helm Charts Repository](https://github.com/ki-smile/nvidia-charts)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Helm Documentation](https://helm.sh/docs/)
- [NVIDIA GPU Operator](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/overview.html)
