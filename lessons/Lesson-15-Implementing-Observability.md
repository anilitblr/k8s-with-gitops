# Lesson 15: Implementing Observability

- [Lesson 15: Implementing Observability](#lesson-15-implementing-observability)
    - [15.1 Understanding Observability](#151-understanding-observability)
      - [Why Observability is Needed](#why-observability-is-needed)
      - [Application Logging](#application-logging)
      - [Metrics](#metrics)
      - [Tracing](#tracing)
      - [Observability in GitOps](#observability-in-gitops)
      - [Application Monitoring](#application-monitoring)
    - [15.2 Using Kubernetes Observability Solutions](#152-using-kubernetes-observability-solutions)
      - [Kubernetes Observability](#kubernetes-observability)
      - [Understanding Pod states](#understanding-pod-states)
    - [15.3 Using Metrics Server](#153-using-metrics-server)
      - [Demo: Runnig Metrics Server](#demo-runnig-metrics-server)
    - [15.4 Using Prometheus](#154-using-prometheus)
      - [Understanding Requirements](#understanding-requirements)
      - [Demo: Running Prometheus](#demo-running-prometheus)
      - [Demo: Exposing Prometheus](#demo-exposing-prometheus)
      - [Exploring Kubernetes Resources](#exploring-kubernetes-resources)
    - [15.5 GitOps Observability](#155-gitops-observability)
      - [Monitoring GitOps Processes](#monitoring-gitops-processes)
      - [Monitoring Application Sync Status](#monitoring-application-sync-status)
      - [Monitoring Configuration Drift](#monitoring-configuration-drift)
      - [Demo: Monitoring Configuration Drift](#demo-monitoring-configuration-drift)
      - [Monitoring Git Changes](#monitoring-git-changes)
      - [Demo: Monitoring Git Changes](#demo-monitoring-git-changes)

### 15.1 Understanding Observability

#### Why Observability is Needed

- Observability is about monitoring what is going on in the cluster.
- In a GitOps environment this ia an essential element, as it allows the GitOps tools to apply application updates if the current state of an application doesn't match the desired state.
- Observability includes three system aspects:
  - Event logging
  - Metrics
  - Tracing
- Differrent ecosystem tools as well as Kubernetes and Git native tools re available for observability.

#### Application Logging

- Kubernetes logging starts at the Pod.
- The Pod application generates events which are captured by Kubernetes and stored in the cluster.
- Use **kubectl logs** to read the Pods logs.
- Although logs are useful for checking individual applications, they don't reveal the actual state of the entire system.

#### Metrics

- Metrics are used to measure performance and workings of the application or system.
- Core metrics include CPU, memory, disk and network utilization.
- Application-specific metrics can be gathers metrics, which can next be used by tools like **kubectl top [pods | nodes]**
- In addition, applications may provide their own metrics through a metrics endpoint.
- Metrics should be collected periodically and stored so that metrics historical trends can be analyzed.
- Prometheus is used to store such metrics in a database.
- In addition, Grafana can be used to visualize the metrics.

#### Tracing

- Tracing is about recording information about a program's execution.
- A cloud-based, distributed, tracing framework provides insight in the entire flow, from user request to the applications implemented as microservices.
- Jaeger is the common tracing utility.
- To use tracing, the application must be configured to send data to the tracing application.

#### Observability in GitOps

- In a GitOps environment, there are different aspects that should be monitored using observability.
  - Application health
  - Application sync status
  - Configuration drift
  - Changes to the environment

#### Application Monitoring

- While monitoring applications, different parameters can be observed.
- Common metrics include
  - **Latency:** the time is takes to process a request.
  - **Traffic:** the amount of traffic
  - **Errors:** the amoutn of erros
  - **Saturation:** the percentage of the application capacity that is currently used

### 15.2 Using Kubernetes Observability Solutions

#### Kubernetes Observability

- Kubernetes itself has some basic options for observability.
- **kubectl describe pod ...** gives information about the current Pod status
- Probes can be used to check the readiness or liveness of a Pod

#### Understanding Pod states

- **kubectl describe pod ...** shows the current state of a Pod
  - **Pending:** the Pod is being started but waiting for an external condition such as container image download or resource availability.
  - **Running:** the Pod is operational
  - **Succeeded:** the Pod has the RestartPolicy set such that it isn't automatically restarted and has completed its work.
  - **Failed:** a container application has failed with a non-zero exit status
  - **Unknown:** the status of the Pod could not be obtained
- To understand the state, also check the Pod conditions:
  - **PodScheduled:** the scheduler has assigned the Pod to a node
  - **Initialized:** all init containers have been started
  - **ContainersReady:** all containers in the Pod are ready
  - **Ready:** the Pod is operational
```bash
kubectl describe pod <pod-name>
```

### 15.3 Using Metrics Server

#### Demo: Runnig Metrics Server

```bash
# See https://github.com/kubernetes-sigs/metrics-server.git
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl -n kube-system get pods # look for metrics-server
kubectl -n kube-system edit deployments.apps metrics-server 
    # In spec.template.spec.containers.args, use the following
    - --kubelet-insecure-tls
    - --kubelet-preferrred-address-types=InternalIP,ExternalIP,Hostname
kubectl -n kube-system logs metrics-server<TAB> # should now generating self-signed cert and Serving securely on ::443
kubectl top pods --all-namespaces # will show most active Pods
```

### 15.4 Using Prometheus

#### Understanding Requirements

- To scrape metrics from Kubernetes, it makes sense to run Prometheus as an application (Deployment) in Kubernetes.
- You'll need a service to make it accessible, and optionally an Ingress resource as well.
- You'll need some Role Based Access Contril settings to allow it to get information bout of the Kubernetes etcd.
- You'll need a ConfigMap to provide a custom prometheus.yml that tells Prometheus to scrape data from Kubernetes resources.

#### Demo: Running Prometheus

```bash
minikube ssh;
ls cpu;
exit;
# Make sure the Kube worker node has a CPUs to host the metrics scraper Pods!
git clone https://github.com/prometheus-operator/kube-prometheus
cd kube-prometheus/
kubectl create -f manifests/setup/ # Do not use kubectl apply
kubectl create -f manifests/
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

#### Demo: Exposing Prometheus

- After installing the Prometheus operator, ClusterIP services are started to provide access.
- Before being able to actually access the components, these services need to be exposed.
- Recommended: use **kubectl port-forward** to expose a local port that forwards to the services.
  - kubectl -n monitoring port-forward svc/prometheus-k8s 9090 &
  - kubectl -n monitoring port-forward svc/grafana 3000 &
- As an alternative, the service can be configured as NodePort or an Ingress can be used to provide access.

#### Exploring Kubernetes Resources

- With the basic configuration, different Kubernetes cluster-related resources can be discovered.
- Use the metrics explorer (globe sysbol) for a list of all available resources
- Compose a dashboard using the critical metrics for your system to be observed

### 15.5 GitOps Observability

#### Monitoring GitOps Processes

- In a GitOps environment, specific parameters should be observed
- Performance of the GitOps operator itself
  - Latency
  - Traffic
  - Saturation
  - Errors
- Application sync status
- Configuration drift

#### Monitoring Application Sync Status

- In a GitOps environment, the running application should be in sync with the application sources in Git.
- There are different tools to verify
- **git diff** will show details about uncommitted changes to the Git repository.
- **kubectl diff** will verify if the content of a manifest file meets the current state of the Kubernetes application

#### Monitoring Configuration Drift

- A configuration drift occurs when changes are applied outside of the GitOps process
- Think of a user using any imperative command like **kubectl edit** or **kubectl scale** to change resources
- The GitOps operator might automatically revert this type of change
- To detect differences between manifest and running application, use **kubectl diff -f manifestfile.yaml**

#### Demo: Monitoring Configuration Drift

```bash
cd labs/kube-app
kubectl diff -f sampleginx-deploy.yaml
kubectl apply -f sampleginx-deploy.yaml
kubectl diff -f sampleginx-deploy.yaml
kubectl scale deploy sampleinx --replicas=4
kubectl diff -f sampleginx-deploy.yaml
```

#### Monitoring Git Changes

- To analyze what has been happening, you might want to monitor Git commits.
- Use **git log** to get a log of all events that have occurred on a specific repository.
- This will show the commit message, which should identify the reason of the commit as well as the author of the change.
- Each commit has a specific event.
- Use **git show <commit-id>** to get more information about the event.

#### Demo: Monitoring Git Changes

```bash
# Run this demo in any Git repository.
git log # will show all events.
git show <commit-id>
```

