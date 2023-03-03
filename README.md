# Kubernetes-Mastery
Advanced Techniques and Best Practices for Efficient Container Orchestration

## 1. Install Docker on *Ubuntu:AWS Lightsail* and verify it works:

#### Update the package index:

```bash
sudo apt-get update
```

#### Install the required packages to allow apt to use a repository over HTTPS:
```bash
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

#### Add Docker's official GPG key:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

#### Add the Docker repository to APT sources:
```bash
sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
```

#### Update the package index again:
```bash
sudo apt-get update
```

#### Install Docker:
```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

#### This should download and run a test container that prints a "Hello from Docker!" message. Verify that Docker is installed and running:
```bash
sudo docker run hello-world
```

#### You can also verify the Docker version:
```bash
docker version
```

## 2. Install Kubernetes on a *Ubuntu:AWS Lightsail* machine and verify its installation:

#### Update the system packages:
```bash
sudo apt-get update
```

#### Install dependencies:
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```

#### Add the Kubernetes signing key:
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

#### Add the Kubernetes repository:
```bash
sudo add-apt-repository "deb https://apt.kubernetes.io/ kubernetes-xenial main"
```

#### Update the system packages again:
```bash
sudo apt-get update
```

#### Install Kubernetes:
```bash
sudo apt-get install -y kubelet kubeadm kubectl
```

#### Initialize the cluster:
```bash
sudo kubeadm init
```

#### Set up Kubernetes networking:
```bash
sudo kubectl apply -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml
```

#### Verify that Kubernetes is running:
```bash
sudo kubectl get pods --all-namespaces
```

## 3. Some commonly used commands to monitor the state of Kubernetes:
```bash
kubectl get nodes: This command displays the status of all the nodes in the Kubernetes cluster.
```

```bash
kubectl get pods: This command displays the status of all the pods in the Kubernetes cluster.
```

```bash
kubectl get services: This command displays the status of all the services in the Kubernetes cluster.
```

```bash
kubectl get deployments: This command displays the status of all the deployments in the Kubernetes cluster.
```

```bash
kubectl get events: This command displays the recent events in the Kubernetes cluster.
```

```bash
kubectl logs <pod-name>: This command displays the logs of a specific pod.
```

```bash
kubectl describe <object-type> <object-name>: This command provides a detailed description of a specific Kubernetes object, such as a pod, service, deployment, or node.
```

These commands help to monitor the state of Kubernetes and ensure that everything is working as expected.
