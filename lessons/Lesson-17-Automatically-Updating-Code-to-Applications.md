# Lesson 17: Automatically Updating Code to Applications

- [Lesson 17: Automatically Updating Code to Applications](#lesson-17-automatically-updating-code-to-applications)
    - [17.1 Introducing CI/CD Solutions](#171-introducing-cicd-solutions)
      - [Overview](#overview)
    - [17.2 Setting up Flux](#172-setting-up-flux)
      - [Understanding Flux](#understanding-flux)
      - [Using Flux Sources](#using-flux-sources)
      - [Demo: Setting up Flux](#demo-setting-up-flux)
    - [17.3 Using Flux](#173-using-flux)
      - [Demo: Using Flux CD](#demo-using-flux-cd)
      - [Demo: Using Flux CD](#demo-using-flux-cd-1)
      - [Demo: Updating Applications](#demo-updating-applications)
    - [17.4 Exploring OpenShift](#174-exploring-openshift)
      - [Understanding OpenShift](#understanding-openshift)
      - [Running Applications in OpenShift](#running-applications-in-openshift)
    - [17.5 Using OpenShift Source to Image](#175-using-openshift-source-to-image)
      - [Understanding Source to Image](#understanding-source-to-image)
      - [Understanding S2i Resources](#understanding-s2i-resources)
      - [Understanding ImageStream](#understanding-imagestream)
      - [Demo: Running S2i](#demo-running-s2i)
    - [17.6 Understanding Argo CD](#176-understanding-argo-cd)
      - [Understanding Argo CD](#understanding-argo-cd)
      - [Argo CD Core Concepts Applications](#argo-cd-core-concepts-applications)
      - [Application Sync and Health Status](#application-sync-and-health-status)
      - [Argo CD Core Concepts: Projects](#argo-cd-core-concepts-projects)
      - [Argo CD Working](#argo-cd-working)
    - [17.7 Using Argo CD](#177-using-argo-cd)
      - [Demo: Installing Argo CD](#demo-installing-argo-cd)
      - [Demo: Accessing Argo CD](#demo-accessing-argo-cd)
      - [Understanding Argo CD Working](#understanding-argo-cd-working)
      - [Demo: Synchronizing an Application](#demo-synchronizing-an-application)
      - [Demo: Updating a Changed Application](#demo-updating-a-changed-application)
      - [Using the Argo CD Web Interface](#using-the-argo-cd-web-interface)

### 17.1 Introducing CI/CD Solutions

#### Overview

- Many solutions exist to implement CI/CD in Kubernetes.
- **Flux:** an open source solution that keeps Kubernetes in sync with source code in Git repositories.
- **OpenShift S2I:** OpenShift is the Red Hat Kubernetes distribution. It contains a solution to automatically update running applications when the source code has changed.
- **Argo CD:** Argo CD is an open source solution that implements a Kubernetes operator to automatically synchronize and update applications. 

### 17.2 Setting up Flux

#### Understanding Flux

- Flux implements GitOps continuous deployment in Kubernetes.
- It fetches manifest files from a Git repository, applies a kustomization.yaml to that to apply patches (if needed), and creates the corresponding resources in Kubernetes.
- It uses different custom resources to keep an eye on any changes happening in the Git repository, then applies these changes in the Kubernetes cluster.
- To use it, Flux offers its own CLI, **flux**
- Past releases used **fluxctl**, this is no longer supported.

#### Using Flux Sources

- Flux works with different sources which specify where the application should be obtained.
  - **flux create source git ...**
  - **flux create source helm ...**
- These sources are monitored by the controllers running in the flux-system Namespace.
  - Use **kubectl get pods -n flux-system** to show.

#### Demo: Setting up Flux

```bash
curl -s https://fluxcd.io/install.sh | sudo bash
. <(flux completion bash)
flux check --pre
```

### 17.3 Using Flux

#### Demo: Using Flux CD

```bash
# First, have Flux create a private Git repository.
flux bootstrap github --owner=anilitblr --repository=svvflux --branch=main --path=./clusters/my-cluster --personal
gh auth login
git config --global user.email "mail@mydomain.local"
git config --global user.name "myusername"
export GITHUB_USER=<myusername>
source <(flux completion bash)
```

#### Demo: Using Flux CD

```bash
git clone https://github.com/$GITHUB_USER/svvflux
cd svvflux
flux create source git kube-app --url=https://github.com/anilitblr/kube-app --branch=main --interval=30s --export > clusters/my-cluster/kube-app-source.yaml
git add -A && git commit -m "initial commit"
git push
flux create kustomization kube-app --target-namespace=default --source=kube-app --patch="./" --prune=true --interval=30s --export > clusters/my-cluster/kube-app-kustomization.yaml
git add -A && git commit -m "adding file" && git push

flux get kustomization --watch # give it 2 minutes.
kubectl get all
```

#### Demo: Updating Applications

```bash
cat labs/flux-patch.yaml >> ~/svvflux/clusters/my-cluster/kube-app-kustomization.yaml
git add -A && git commit -m "Update"
git push
flux get kustomization --watch
kubectl get all
```

### 17.4 Exploring OpenShift

#### Understanding OpenShift

- OpenShift is the Red Hat Kubernetes distribution
- Expressed in main functionality, OpenShift is a Kubernetes distribution where developer options are integrated in an automated way.
  - Source 2 Image
  - Pipelines
  - More developed authentication and RBAC.
- OpenShift adds operators to vanilla Kubernetes.
- OpenShift adds many extensions to the Kubernetes APIs.
- To use OpenShift, get OpenShift Local from developers.redhat.com

#### Running Applications in OpenShift

```bash
# OpenShift uses the oc utility as the default tool.
oc new-project demo;
oc new-app --docker-image=mariadb
oc set -h
oc adm -h

# Managing a running environment is very similar to Kubernetes.
oc get all
oc logs
oc describe
oc explain
```

### 17.5 Using OpenShift Source to Image

#### Understanding Source to Image

- Source to Image (S2i) allows you to run an application directly from source code.
- **oc new-app** allows you to work with S2i directly.
- S2i connects source code to an S2i image stream builder image to create a temporary builder Pod that writes an application image to the internal image registry.
- Based on this custom image, a deployment is created.
- S2i takes away the need for the developer to know anything about Dockerfile and related items.
- S2i also allows for continuous patching as updates can be triggered using webhooks.

#### Understanding S2i Resources

- **ImageStream:** defines the interpreter needed to create the custom image.
- **BuildConfig:** defines all that is needed to convert source code into an image (Git repo, imagestream)
- **DeploymentConfig/Deployment:** defines how to run the container in the cluster; contains the Pod template that refers to the custom build iamge.
- **Service:** defines how the application running in the deployment is exposed.

#### Understanding ImageStream

- The ImageStream is an API resource offered by the internal image repository to provide different versiosn of images.
- Use **oc get is -n openshift** for a list.
- ImageStreams are managed by Red Hat through OpenShift. If a new version of the image becomes available, it will automatically trigger a new build of application code.
- Custom ImageStreams can also be integrated.

#### Demo: Running S2i

```bash
oc new-app php~https://githum.com/anilitblr/simpleapp --name=simple-app
oc get all
oc get is -n openshift
oc get builds # allows for monitoring the build process.
oc get buildconfig -o yaml | less # shows the buildconfig used.
oc get deployment # shows the resulting deployment.
oc start-build simple1 # runs a manual application update. 
```

### 17.6 Understanding Argo CD

#### Understanding Argo CD

- Argo CD is a declarative GitOps continuous delivery tool for Kubernetes.
- It's an open source GitOps operator.
- Argo Cd integrates to Kubernetes to synchronize applications in Git.
- It comes with its CLI or web-based management interface that allows you to manage application state.
- Argo CD was designed to be used in very big multitenant Kubernetes clusters.

#### Argo CD Core Concepts Applications

- The Application connects to a source (Git repository) to create Kubernetes objects in the destination.
  - It connects to a source Git repo where GitOps environment can be implemented.
  - It defines a Kubernetes destination (which can be a GitOps environment)
  - It uses sync and health statuses to keep the Kubernetes objects up-to-date
- The Application source doesn't define how to provide the applications.
  - YAML manifest files, helm charts, and Kustomize overlays are all supported.
- The Applicatino defines in which Kubernetes cluster and in which Namespace the Kubernetes objects should be created.

#### Application Sync and Health Status

- The Argo CD application sync status uses **kubectl diff** to verify that the application desired state as defined in the (Git) source meets the current state of the running Kubernetes objects.
- The health status summarizes if the application meets its requirements.
- The aggregated application status as shown by the **argocd apps list** command indicates the current state of the entire application. If only one component is degraded, the entire application shows as degraded.

#### Argo CD Core Concepts: Projects

- The **Project** provides a logical grouping of applications where teams within an organization are isolated from each other by fine tuning access control.
- The purpose of the project is to provide security, and to isolate teams from each other.
- It restricts which Kubernetes objects may be created by an application.

#### Argo CD Working

- The essence of Argo CD Working is based on two features.
  - Use **kubectl diff** to compare current file contents with a running application.
  - If there's a difference, use **kubectl apply** to update the application.
- To implement this procedure, different phases are processed.
  - **Retrieve resource manifests:** done by argocd-repo-server.
  - **Detect and fix deviations:** done by argocd-application-controller.
  - **Present the result to end usres;** done by argocd-server.

### 17.7 Using Argo CD

#### Demo: Installing Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get all -n argocd
# Download latest version of the argocd binary from here.
# https://github.com/argoproj/argo-cd/releases/
sudo mv ~/Downloads/argocd-linux-amd64 /usr/local/bin/argocd
```

#### Demo: Accessing Argo CD

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
kubectl port-forward svc/argocd-server -n argocd 8080:443
argocd login localhost:8080
    un: admin
    pw: as printed above
argocd account update-password
```

#### Understanding Argo CD Working

- While creating an app with **argocd create app**, the application is created but not automatically synced.
- To sync the Kubernetes resources with the code in the Git repository you have two options.
  - Manually synchronize, using **argocd app sync**
  - Configure a sync policy to synchronize automatically: **argocd app set <appname> --sync-policy automated**

#### Demo: Synchronizing an Application

```bash
argocd app create kube-app --repo https://github.com/anilitblr/kube-app --path . --dest-server https://kubernetes.default.svc --dest-namespace default
argocd app get kube-app
argocd app list
argocd app sync kube-app
kubectl get all
```

#### Demo: Updating a Changed Application

- Apply a minor change to the source code in Git, and push the changes.
- Use **argocd app diff kube-app --refresh**
- **argocd app get kube-app** will show current status as OutOfSync.
- **argocd app sync kube-app** will trigger a synchronization.

#### Using the Argo CD Web Interface

- Argo CD comes with a web interface
- This interface can be accessed on the Argo CD port that is exposed.
- For easy access, use **kubectl port-forward** to define a port on the kubectl client machine.
- Alternatively, use the Nodeport Service type to define a port on the cluster nodes.
