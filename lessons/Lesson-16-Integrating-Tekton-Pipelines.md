# Lesson 16: Integrating Tekton Pipelines

- [Lesson 16: Integrating Tekton Pipelines](#lesson-16-integrating-tekton-pipelines)
    - [16.1 Understanding Tekton Objects](#161-understanding-tekton-objects)
      - [Understanding Tekton](#understanding-tekton)
      - [Demo: Installing Tekton](#demo-installing-tekton)
    - [16.2 Running Tekton Tasks](#162-running-tekton-tasks)
      - [Understanding Tasks](#understanding-tasks)
      - [Demo: Running Tasks](#demo-running-tasks)
    - [16.3 Running Tekton Pipelines](#163-running-tekton-pipelines)
      - [Understanding Pipelines](#understanding-pipelines)
    - [16.4 Running Tekton Triggers](#164-running-tekton-triggers)
      - [Understanding Triggers](#understanding-triggers)
      - [Demo: Installing Tekton Triggers](#demo-installing-tekton-triggers)
      - [Demo: Using Tekton Triggers](#demo-using-tekton-triggers)

### 16.1 Understanding Tekton Objects

#### Understanding Tekton

- Tekton is used to integrate pipeline execution in Kubernetes.
- The core element to do so is the Task.
- The Tekton Task is an API resource that defines a series of steps to be executed.
- The Taskruns as a Pod, and each step runs in its own container.

#### Demo: Installing Tekton

```bash
# Tekton installs CRDs in an existing Kubernetes cluster.
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

# Installation instructions.
https://tekton.dev/docs/cli

wget https://github.com/tektoncd/cli/releases/download/v0.32.2/tektoncd-cli-0.32.2_Linux-64bit.deb

sudo dpkg -i tektoncd-cli-0.32.2_Linux-64bit.deb

tkn version;
kubectl get pods -n tekton-pipelines
```

### 16.2 Running Tekton Tasks

#### Understanding Tasks

- A Task  defines steps which are executed by a Pod.
- Each step runs a container with a specific command in that Pod.
- The TaskRun is needed to run a Task.

#### Demo: Running Tasks

```bash
cd labs/
cat demotask.yaml
kubectl apply -f demotask.yaml
kubectl get pods,tasks # onlyshows the Task
cat demotask-run.yaml
kubectl apply -f demotask-run.yaml
kubectl get pods,tasks,taskruns # shows a multi-container Pod
kubectl logs --selector=tekton.dev/taskRun=devmotask-run
kubectl logs --selector=tekton.dev/taskRun=demotask-run -c step-steptwo
```

### 16.3 Running Tekton Pipelines

#### Understanding Pipelines

- A Pipeline organizes Tasks to run in a specific order.
- If a Task uses params (which really are variables), these may be defined in the Pipeline spec for further handling by the Task.
- To run a Pipeline, a PipelineRun object is needed.
- The PipelineRun set the values for parameters and runs the Pipeline
- The Pipeline runs the Tasks, and creates one Pod for each Task, with one container for each step in the Tasks.

```bash
cd labs/
kubectl apply -f secondtask.yaml
kubectl apply -f demo-pipeline.yaml
cat demo-pipeline-run.yaml
kubectl apply -f demo-pipeline-run.yaml
kubectl get pipelines,pipelineruns,tasks,tasksruns
tkn pipelinerun logs demopipe-run -f -n default
kubectl get pipeline,pipelinerun,tasks,pods
```

### 16.4 Running Tekton Triggers

#### Understanding Triggers

- A Trigger is an API resource that runs a Pipeline based on an external event.
- It consists of three additional API resources.
  - EventListener: runs a Pod that listens for specific events.
  - TriggerTemplate: configures a PipelineRun when the event occurs.
  - TriggerBinding: passes the data created by the TriggerTemplate to the PipelineRun.
- Because the EventListener manages API objects, a ServiceAccount and RBAC configuration are required to use it.

#### Demo: Installing Tekton Triggers

```bash
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml
kubectl get pods -n tekton-pipelines
```

#### Demo: Using Tekton Triggers

```bash
kubectl apply -f trigger-template.yaml
kubectl apply -f trigger-binding.yaml
kubectl apply -f rbac.yaml
kubectl apply -f event-listener.yaml
kubectl port-forward service/el-demo-listernet 8080
# From another terminal
curl -v -H 'content-Type: application/json' -d '{"username": "anil"}' http://localhost:8080
kubectl get pipelineruns
tkn pipelinerun logs <mypipelinerun> -f
```
