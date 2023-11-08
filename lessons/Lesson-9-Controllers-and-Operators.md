# Lesson 9: Controllers and Operators

- [Lesson 9: Controllers and Operators](#lesson-9-controllers-and-operators)
    - [9.1 Custom Resources](#91-custom-resources)
      - [Custom Resources Definitions](#custom-resources-definitions)
    - [9.2 Providing Operator API Access](#92-providing-operator-api-access)
      - [Understanding Access Needs](#understanding-access-needs)
      - [Demo: Configuring a ServiceAccount](#demo-configuring-a-serviceaccount)
    - [9.3 Understanding Controllers and Operators](#93-understanding-controllers-and-operators)
      - [Understanding Controllers](#understanding-controllers)
      - [Understanding Operators](#understanding-operators)
      - [Using Community Operators](#using-community-operators)
    - [9.4 Creating a Custom Operator](#94-creating-a-custom-operator)
      - [What the Operator Needs to do](#what-the-operator-needs-to-do)
      - [How Operators Monitor for Changes](#how-operators-monitor-for-changes)
      - [Demo: Uing kubectl get --watch](#demo-uing-kubectl-get---watch)
      - [Implementing a Custom Operator](#implementing-a-custom-operator)

### 9.1 Custom Resources

#### Custom Resources Definitions

- The Kubernetes API is extensible.
- CustomResourceDefinitions (CRDs) are provided to make API extension easy.
- Using a CustomResourceDefinition, any resource can be added to the API easily.
- While adding applications to Kubernetes, using Operators and Controllers, CustomResourceDefinitions are commonly used.

```bash
kubectl api-resources | less
vim labs/crd-object.yaml
kubectl apply -f labs/crd-object.yaml
kubectl api-resources | grep backup
vim labs/crd-backup.yaml
kubectl apply -f labs/crd-backup.yaml
kubectl get backup
```

### 9.2 Providing Operator API Access

#### Understanding Access Needs

- In a Kubernetes GitOps environment we want to use operators that automatically create and update applications in Kubernetes.
- To do so, the operator needs permissions.
- To allow Kubernetes applications to manage other Kubernetes resources, Kubernetes provides the ServiceAccount resource.
- Each application uses a default ServiceAccount, providing limited access to the cluster.
- For the GitOps operator to work, the ServiceAccount needs additional permissions.
- To obtain these permissions, a RoleBinding or ClusterRoleBinding is needed.

```bash
kubectl get pods
kubectl get pods defaultpod -o yaml | less
kubectl get sa
kubectl get sa -n kube-system
```

#### Demo: Configuring a ServiceAccount

```bash
kubectl create sa gitops
kubectl get clusterroles -A | less
kubectl create clusterrolebinding admin --clusterrole=admin --serviceaccount=default:gitops
kubectl get clusterrolebinding admin -o yaml
```

### 9.3 Understanding Controllers and Operators

#### Understanding Controllers

- A Kubernetes controller monitors Kubernetes resources to ensure that the current state matches the desired state.
- Each resource type has its own controller.
- When Kubernetes APIs are extended using CRDs, new controllers can be added to realize functionality.
- Each controller runs an infitite loop where each iteration reconciles the desired state as described in the **spec** with the current state in the **status** of the resources definition.
- The **kube-controller-manager** is a core Kubernetes process that takes care of all resource specific controllers.

#### Understanding Operators 

- A Kubernetes operator is an application specific controller that manages the desired state of complex applications on behalf of a user.
- Opeators may use CRDs to do their work.
- All operators are controllers, the difference is that controllers are a part of the core Kubernetes code whereas operators are created by Kubernetes users to do specific work.
- Controllers relate to resources types as defined in the API, operators relate to specific applications.

#### Using Community Operators

- Community provided operators are available, providing much additional functionality.
- Operators can easily be installed using helm chars.
- Alternatively, operators can be added through https://operatorhub.io

### 9.4 Creating a Custom Operator

#### What the Operator Needs to do

- The operator needs to automatically perform actios on specific events.
- Any event can be monitored, such as resource creation, modfication and more.
- This allows the operator to set up an environemtn for running an application, to move an application forward to the next stage or anything else that you could imagine.

#### How Operators Monitor for Changes

- To run an operator, the operator needs to be able to detect changes to Kubernetes resources.
- If the operator runs in a loop and automatically monitors resources state changes, it can be configured to act upon a state change.
- The foundation for the working of a custom operator is the **kubectl get --watch --ouput-watch-events** command.
- This commands shows ouput as ADDED, MODIFIED, DELETED
- The operator can be configured to react when one of these occurs.
- Bash scripts are a nice way of doing so.
- To only see modifications, and not the resources already existing, use **kubectl get --watch --watch-only --output-watch-events** 

#### Demo: Uing kubectl get --watch
```bash
# From one terminal.
kubectl get --watch --ouput-watch-events pod

# From another terminal
kubectl run testpod --image=nginx

# Observe the output of the below command 
kubectl get --watch 
```

#### Implementing a Custom Operator

- If an operator is an automated task that runs continuously, it can be implemented by running a simple shell script.
- To make it a bit moreadvance, the shell script can be included in a Kubernetes CronJob which will make it less reactive.
- The mission of the operator is to react on events, and perform specific actions if a specific event is happening.
- Check **exposenginx.yaml** in the repository.
- This operator exposes Deployments but only if they're using an Nginx container image.
```bash
vim labs/exposenginx.yaml
chmod +x labs/exposenginx.yaml
./labs/exposenginx.yaml

# From another terminal
kubectl create deploy operatornginx --image=nginx --replicas=3
kubectl get all --selector app=operatornginx
kubectl describe svc operatornginx
``` 
