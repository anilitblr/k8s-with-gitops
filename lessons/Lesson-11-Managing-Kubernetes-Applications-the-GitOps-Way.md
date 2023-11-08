# Lesson 11: Managing Kubernetes Applications the GitOps Way

- [Lesson 11: Managing Kubernetes Applications the GitOps Way](#lesson-11-managing-kubernetes-applications-the-gitops-way)
    - [11.1 Using the Helm Package Manager](#111-using-the-helm-package-manager)
      - [Understanding Helom Usability in GitOps](#understanding-helom-usability-in-gitops)
    - [11.2 Exploring Kustomize](#112-exploring-kustomize)
      - [Exploring Kustomize](#exploring-kustomize)
      - [Understanding Kustomize](#understanding-kustomize)
      - [Generating Configuration](#generating-configuration)
      - [Demo: Using Kustomize](#demo-using-kustomize)
    - [11.3 Using Kustomize to Handle Application Updates the GitOps Way](#113-using-kustomize-to-handle-application-updates-the-gitops-way)
      - [Uisng Kustomization in GitOps](#uisng-kustomization-in-gitops)

### 11.1 Using the Helm Package Manager

#### Understanding Helom Usability in GitOps

- Helm is the standard Kubernetes package manager.
- It is not a configuration took, but is can be used as a configuration tool.
- Specific configuration is provided y a values.yaml file.
- In a GitOps environment, multiple values.yaml files can be used, one for each environment.
- Using Helm charts in GitOps is not recommended for the following reasons.
  - Helm charts and values files are not declarative by nature.
  - Helm charts and values files are hard to read.
  - Much additional scripting would be required to make them fit different environment.

### 11.2 Exploring Kustomize

#### Exploring Kustomize

- Kustomize was developed with declarative application management in mind.
- Kustomize functionality is integrated in **kubectl**, which makes it easy to use.
- Kustomize makes it easy to merge patches and specific parameters to a base templated version of an application.

#### Understanding Kustomize

- kustomize is a Kubernetes feature that uses a file with the name **kustomization.yaml** to store instructions for the changes a user wants to make to a set of resources.
- This is convenient for applying changes to input files that the user does not cotril himself, and which contents may change because of new versions appearning in Git.
- So, it's a way to modify existing configuration files, and thus implement changes in a declarative way.
- Use **kubectl apply -k ./** in the directory with the kustomization.yaml and the files it refers to apply changes.
- Use **kubectl delete -k ./** in the same directory to delete all that was created by the Kustomization.

#### Generating Configuration

**kustomize** can also be used to generate configuration.
- Use **secretGenerator** to generate a Secret.
- Use **configMapGenerator** to generate a ConfigMap.

#### Demo: Using Kustomize

```bash
cd labs/kustomize-demo
cat deployment.yaml
cat service.yaml
kubectl apply -f deployment.yaml service.yaml
kubectl apply -f service.yaml
kubectl get deploy,svc
cat kustomization.yaml
kubectl apply -k .

cd labs/kustomize-microservice/
cat kustomization.yaml
kubectl apply -k .
```

### 11.3 Using Kustomize to Handle Application Updates the GitOps Way

#### Uisng Kustomization in GitOps

- In a GitOps environment, an application template should be provided with its kustomization.yaml file in a base directory.
  - This application template contains all resources that are common to all environments.
  - The kustomization.yaml file in the base directory is very simple and just creates the application.
- Apart, for each environment a separate directory should be created, containing its own kustomization.yaml file and all additional configuration that will be patched to the base configuration.
- See the example in the Git repository **labs/kustomization-envs**
- Use **kubectl kustomize .** from this directory.
```bash
cd labs/kustomization-envs
tree
cd base
cat deployment.yaml
cat kustomization.yaml
cd ..
cd envs/qa
ls
cat kustomization.yaml
cat patch.yaml
kubectl apply -k .
kubectl get deploy demo-app -o yaml
```
