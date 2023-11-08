# Lesson 7: Running Applications in Kubernetes

- [Lesson 7: Running Applications in Kubernetes](#lesson-7-running-applications-in-kubernetes)
    - [7.1 Using Kubernetes](#71-using-kubernetes)
      - [Understanding Kubernetes](#understanding-kubernetes)
    - [7.2 Using Minikube](#72-using-minikube)
      - [Installing Minikube on Linux](#installing-minikube-on-linux)
      - [Installing Minikube on Windows](#installing-minikube-on-windows)
      - [Installing Minikube on MacOS](#installing-minikube-on-macos)
    - [7.3 Kubernetes Resources](#73-kubernetes-resources)
      - [Understanding Kubernetes Applications](#understanding-kubernetes-applications)
    - [7.4 Running Applications the Declarative Way](#74-running-applications-the-declarative-way)
      - [GitOps Code Requirements](#gitops-code-requirements)
      - [Running Applications the Declarative way](#running-applications-the-declarative-way)
      - [Using kubectl apply or kubectl create](#using-kubectl-apply-or-kubectl-create)
      - [Dynamic Updates](#dynamic-updates)
      - [Demo: Running Applications](#demo-running-applications)
    - [7.5 Providing Access to Applications](#75-providing-access-to-applications)
      - [Accessing Applications](#accessing-applications)
      - [Demo: using kubectl port-forward](#demo-using-kubectl-port-forward)

### 7.1 Using Kubernetes

#### Understanding Kubernetes

- Kubernetes is the standard open-source platform for container orchestration.
- It can be used in different ways.
  - Managed in cloud.
  - Installed on-premise.
  - In minimized learning environments like Minikube.
  - For the purpose of this course, we will be using Minikube.

### 7.2 Using Minikube

#### Installing Minikube on Linux

- Minikube can run as a virtual machine or as a container.
- Using the container driver is recommended.
- To run Minikube, you need at least 2 GB of available RAM and 2vCPUs
- Easy setup on Ubuntu Desktop is provided through the **minikube-docker-setup.sh** script in the course Git repository.

```bash
cd labs
./minikube-docker-setup.sh
minikube start --memory=6g --cpus=4 --vm-driver=docker --cni=calico
kubectl get all
```

#### Installing Minikube on Windows

- Minikube can run as a virtual machine or as a container.
- Hyper-V virtualization is recommended.
- Only supported on Windows Enterprise, Pro, or Education.
- To run Minikube, you need at least 2 GB of available RAM and 2vCPUs

```bash
# Enable Hyper-V virtualization. From an Administrator PowerShell session.
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All

# Download and install .exe installer from.
link: https://minikube.sigs.k8s.io

# Use to start.
minikube start --driver=hyperv

# Should show some current confguration
minikube kubectl get all
```

#### Installing Minikube on MacOS

- Installing Minikube on MacOS Intel platform is easy - just follow the insructions at https://minikube.sigs.k8s.io
- Installing Minikube on MacOS with Arm CPU is more challenging.
  - Arm CPU needs Minikube compiled for ARM architecture.
  - Virtualization for ARM architecture is ore complicated.
- Recommended
  - Use **brew** to install podman and Minikube.
  - Use **podman machine** to install a podman virual machine, which is required to run containers on a non-Linux OS.
  - Use **minikube start --driver=podman --cni=calico** to start Minikube

    ```bash
    # Install homebrew
    brew insdtall podman
    brew install minikube
    podman machine init --cpus 2 --memory 6144 --rootful
    podman machine start
    minikube start --driver=podman --cni=calico
    ```

### 7.3 Kubernetes Resources

#### Understanding Kubernetes Applications

Kubernetes uses different resources to specify the desired state of the cluster.
- **Pod**: one or more containers that run the application.
- **Deployment**: used to run a scalable application.
- **Volume**: represents storage, either as a part of the Pod definition, or as its own resource.
- **Service**: a policy that provides access to the application Pods.
- **Jobs**: tasks that run as a Pod in the Kubernetes cluster.
- **CronJobs**: used to run Jobs on a specific schedule.

### 7.4 Running Applications the Declarative Way

#### GitOps Code Requirements

- To manage applications in a GitOps environment, there are a few requirements.
  - Application updates should happen through the same code as application creation.
  - Code must be idempotent.
- To meet these requirements in a Kubernetes environment, changes should be applied the declarative way.
- The kubectl apply command is what should be used throughout.

#### Running Applications the Declarative way

- An easy way to run an application in Kubernetes is **kubectl create deploy myapp --image=myimage --replicase=3**
- Although useful, this way doesn't work well in a GitOps environment.
- In GitOps the application is defined in a YAML manitest file, and Kubernetes uses and operator to pick up and apply changes in the YAML file automatically.
- To generate the YAML manifest file, use **kubectl create deploy myapp --image=myimage --replicas=3 --dry-run=client -o yaml > myapp.yaml**
- Next, push the YAML file to the Git repository and have the Kubernetes operator use **kubectl apply -f myapp.yaml** to apply the code to the cluster.

#### Using kubectl apply or kubectl create

- After defining YAML files, **kubectl create -f myapp.yaml** can be used to create the application.
  - This command only works if the application doesn't yet exist.
- **kubectl apply -f myapp.yaml** is recommended in GitOps.
  - If the application doesn't yet exists, it will be updated.
  - If the application already exist, modifications will be applied.
  - To preview modifications that will be applied, use **kubectl diff -f myapp.yaml**
  - Each time the application is updated, **kubectl apply** stores the confguration in the last-applied-configuration annotation, whih allows **kubectl diff** to see any change in the manifest file.

#### Dynamic Updates

- Managing applications the declarative way is the foundation for GitOps in a Kubernetes environment.
- Apart from that, application updates can be applied dynamically in some cases.
- HorizontalPodAutoscaler is one solution to apply updates dynamically: when resource usage requires it, it automatically adjusts the number of application instances.

#### Demo: Running Applications

```bash
kubectl create deploy myserver --image=nginx
kubectl create deploy myserver --image=nginx
kubectl delete deploy myserver
kubectl create deploy myserver --image=nginx --dry-run=client -o yaml > myserver.yaml
kubectl apply -f myserver.yaml
kubectl apply -f myserver.yaml
```

### 7.5 Providing Access to Applications

#### Accessing Applications

- After running an application, it doesn't automatically become accessible.
- To access it, different solutions can be used.
  - **kubeclt port-forward** is for testing purposes only, and exposes a port on the host that runs the **kubectl** client.
  - Services are Kubernetes objects that provide load balancing to the different Pod instances.
  - Ingress is an additional Kubernetes object that provides access to HTTP and HTTPS resources.

#### Demo: using kubectl port-forward

```bash
kubectl get all --selector --app=webserver
kubectl port-forward webserver-xxxxx 8080:80
# Open new terminal
curl localhost:8080
```
