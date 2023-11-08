# Automating Kubernetes with GitOps: Introduction

- [Automating Kubernetes with GitOps: Introduction](#automating-kubernetes-with-gitops-introduction)
- [Module 1: Understanding the Fundamentals](#module-1-understanding-the-fundamentals)
    - [Lesson 1: Understanding DevOps and GitOps](#lesson-1-understanding-devops-and-gitops)
    - [Lesson 2: Using Pipelines](#lesson-2-using-pipelines)
- [Module 2: From Code to Image](#module-2-from-code-to-image)
    - [Lesson 3: Using Git](#lesson-3-using-git)
    - [Lesson 4: Building Container Images](#lesson-4-building-container-images)
- [Module 3: Automating Infrastructure](#module-3-automating-infrastructure)
    - [Lesson 5: Infrastructure as Code](#lesson-5-infrastructure-as-code)
    - [Lesson 6: Configuration as Code](#lesson-6-configuration-as-code)
- [Module 4: From Image to Application](#module-4-from-image-to-application)
    - [Lesson 7: Running Applications in Kubernetes](#lesson-7-running-applications-in-kubernetes)
    - [Lesson 8: Setting up Kubernetes for GitOps](#lesson-8-setting-up-kubernetes-for-gitops)
    - [Lesson 9: Controllers and Operators](#lesson-9-controllers-and-operators)
    - [Lesson 10: Kubernetes CI/CD](#lesson-10-kubernetes-cicd)
    - [Lesson 11: Managing Kubernetes Applications the GitOps Way](#lesson-11-managing-kubernetes-applications-the-gitops-way)
    - [Lesson 12: Using Secrets](#lesson-12-using-secrets)
    - [Lesson 13: Using GitOps to Provide Zero-downtime Application Updates](#lesson-13-using-gitops-to-provide-zero-downtime-application-updates)
    - [Lesson 14: Running a GitOps Project](#lesson-14-running-a-gitops-project)
- [Module 5: Using Kubernetes Ecosystem Solutions](#module-5-using-kubernetes-ecosystem-solutions)
    - [Lesson 15: Implementing Observability](#lesson-15-implementing-observability)
    - [Lesson 16: Integrating Tekton Pipelines](#lesson-16-integrating-tekton-pipelines)
    - [Lesson 17: Automatically Updating Code to Applications](#lesson-17-automatically-updating-code-to-applications)


# Module 1: Understanding the Fundamentals

### [Lesson 1: Understanding DevOps and GitOps](lessons/Lesson-1-Understanding-DevOps-and-GitOps.md)

- 1.1 Understanding DevOps
- 1.2 Exploring GitOps
- 1.3 Kubernetes and GitOps
- 1.4 Deploying Everything as Code
- 1.5 DevOps and GitOps Core Components
- 1.6 DevOps Environments
- 1.7 DevOps Stages
- 1.8 Webhooks and Operators

### [Lesson 2: Using Pipelines](lessons/Lesson-2-Using-Pipelines.md)

- 2.1 What is a Pipeline
- 2.2 Creating Pipelines for DevOps
- 2.3 Creating Pipelines for GitOps
- 2.4 Integrating DevOps and GitOps Pipelines
- 2.5 Getting Started with Jenkins
- 2.6 Exploring Pipelines in Jenkins

# Module 2: From Code to Image

- Module introduction

### [Lesson 3: Using Git](lessons/Lesson-3-Using-Git.md)

- 3.1 Understanding Git
- 3.2 Git Fundamentals
- 3.3 Using Git Advanced Authentication
- 3.4 Working with Branches and Merges
- 3.5 Organizing Git Repositories for GitOps Environments

### [Lesson 4: Building Container Images](lessons/Lesson-4-Building-Container-Images.md)

- 4.1 Understanding Image Formats
- 4.2 Using Dockerfile
- 4.3 Creating a GitOps Container Image
- 4.4 Using Webhooks to Automate Container Image Updates

# Module 3: Automating Infrastructure

- Module introduction

### [Lesson 5: Infrastructure as Code](lessons/Lesson-5-Infrastructure-as-Code.md)

- 5.1 Understanding Infrastructure as Code
- 5.2 Exploring Terraform
- 5.3 Using Terraform for Infrastructure as Code
- 5.4 Using Terraform to Manage Kubernetes

### [Lesson 6: Configuration as Code](lessons/Lesson-6-Configuration-as-Code.md)

- 6.1 Ansible and GitOps
- 6.2 Setting up Ansible
- 6.3 Managing Configuration as Code with Ansible
- 6.4 Setting up AWX
- 6.5 Configuring Webhooks on AWX

# Module 4: From Image to Application

- Module introduction

### [Lesson 7: Running Applications in Kubernetes](lessons/Lesson-7-Running-Applications-in-Kubernetes.md)

- 7.1 Using Kubernetes
- 7.2 Using Minikube
- 7.3 Kubernetes Resources
- 7.4 Running Applications the Declarative Way
- 7.5 Providing Access to Applications

### [Lesson 8: Setting up Kubernetes for GitOps](lessons/Lesson-8-Setting-up-Kubernetes-for-GitOps.md)

- 8.1 Using NameSpaces to Represent GitOps Environments
- 8.2 Labels and Annotations
- 8.3 Using ConfigMaps to Provide Application Data
- 8.4 Kubernetes Storage
- 8.5 Using Services
- 8.6 Using Ingress
- 8.7 Ingress Access to Services in Specific Namespaces
- 8.8 Using NetworkPolicy to Isolate GitOps Environments

### [Lesson 9: Controllers and Operators](lessons/Lesson-9-Controllers-and-Operators.md)

- 9.1 Custom Resources
- 9.2 Providing Operator API Access
- 9.3 Understanding Controllers and Operators
- 9.4 Creating a Custom Operator

### [Lesson 10: Kubernetes CI/CD](lessons/Lesson-10-Kubernetes-CI-CD.md)

10.1 Understanding Kubernetes GitOps CI/CD
10.2 Implementing a CI Pipeline in Kubernetes
10.3 Implementing CD with a Kubernetes GitOps Operator

### [Lesson 11: Managing Kubernetes Applications the GitOps Way](lessons/Lesson-11-Managing-Kubernetes-Applications-the-GitOps-Way.md)

- 11.1 Using the Helm Package Manager
- 11.2 Exploring Kustomize
- 11.3 Using Kustomize to Handle Application Updates the GitOps Way

### [Lesson 12: Using Secrets](lessons/Lesson-12-Using-Secrets.md)

- 12.1 Providing Configuration
- 12.2 Using Secrets
- 12.3 Secrets in GitOps
- 12.4 Bitnami SealedSecrets

### [Lesson 13: Using GitOps to Provide Zero-downtime Application Updates](lessons/Lesson-13-Using-GitOps-to-Provide-Zero-downtime-Application-Updates.md)

- 13.1 Using Deployment Rolling Updates
- 13.2 Applying Blue/Green Deployment Updates
- 13.3 Using Canary Deployments
- 13.4 Service-based Canary Deployments

### [Lesson 14: Running a GitOps Project](lessons/Lesson-14-Running-a-GitOps-Project.md)

- 14.1 Understanding the Project
- 14.2 Preparation: Setting up Git
- 14.3 Preparation: Creating a Worker Image
- 14.4 Preparation: Setting up Storage
- 14.5 Preparation: Creating the YAML Files
- 14.6 Implementing the CI Process
- 14.7 Implementing the CD Process
- 14.8 Performing the Blue/Green Application Update

# Module 5: Using Kubernetes Ecosystem Solutions

- Module introduction

### [Lesson 15: Implementing Observability](lessons/Lesson-15-Implementing-Observability.md)

- 15.1 Understanding Observability
- 15.2 Using Kubernetes Observability Solutions
- 15.3 Using Metrics Server
- 15.4 Using Prometheus
- 15.5 GitOps Observability

### [Lesson 16: Integrating Tekton Pipelines](lessons/Lesson-16-Integrating-Tekton-Pipelines.md)

- 16.1 Understanding Tekton Objects
- 16.2 Running Tekton Tasks
- 16.3 Running Tekton Pipelines
- 16.4 Running Tekton Triggers

### [Lesson 17: Automatically Updating Code to Applications](lessons/Lesson-17-Automatically-Updating-Code-to-Applications.md)

- 17.1 Introducing CI/CD Solutions
- 17.2 Setting up Flux
- 17.3 Using Flux
- 17.4 Exploring OpenShift
- 17.5 Using OpenShift Source to Image
- 17.6 Understanding Argo CD
- 17.7 Using Argo CD
