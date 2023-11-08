# Lesson 5: Infrastructure as Code 

- [Lesson 5: Infrastructure as Code](#lesson-5-infrastructure-as-code)
    - [5.1 Understanding Infrastructure as Code](#51-understanding-infrastructure-as-code)
      - [IaC Tools](#iac-tools)
    - [5.2 Exploring Terraform](#52-exploring-terraform)
      - [Terraform Components](#terraform-components)
    - [5.3 Using Terraform for Infrastructure as Code](#53-using-terraform-for-infrastructure-as-code)
      - [Taking a TEstdrive](#taking-a-testdrive)
      - [Demo: Using Terraform](#demo-using-terraform)
      - [Changing Desired State](#changing-desired-state)
    - [5.4 Using Terraform to Manage Kubernetes](#54-using-terraform-to-manage-kubernetes)
      - [Understanding Kubernetes Management](#understanding-kubernetes-management)
      - [Demo: Managing Kubernetes Resources](#demo-managing-kubernetes-resources)

### 5.1 Understanding Infrastructure as Code

- Infrastructure as Code (IaC) is about creating infrastructure the DevOps way.
- To do so, the IaC tool communicates to the infrastructure API.
- Source code of the infrastructure is tored in a version control system, which makes deploying and updating infrastructure the GitOps way easy.
- Notice there is overlap between Infrastructure as Code and Configuration as Code.

#### IaC Tools

- Different tools exist, of which some have a focus on IaC, and some have a more generic approach and can be used to define IaC.
  - Terraform
  - Ansible
  - AWS CloudFormation
  - Azure Resource Manager
  - Google Cloud Deployment Manager
  - Chef
  - Puppet
  - SaltStack

### 5.2 Exploring Terraform

#### Terraform Components

- Terraform exists as a free, open-source solution, as well as a paid commercial solution.
- It can be used to deploy IaC on any platform that has an addressible API, mainly used in cloud.
- To work with Terraform, you will need to install the Terraform binary.
- This binary communicates to the infrastructure API.
- To define infrastructure resources, Terraform works with mainfest files that allows for creating or updating resources.

### 5.3 Using Terraform for Infrastructure as Code

#### Taking a TEstdrive

- To test working with Terraform, you can try building an AWS instance.
- To do so, fetch the AWS_ACCESS_KEY_ID as well as the AWS_SECRET_ACCESS_KEY from the AWS IAM console.
- Also, make sure you use an AMI that has **ENA enabled: yes** (the AMI in the example in the Terraform documentation doesn't work)

#### Demo: Using Terraform

- Fetch the binary from Terraform.io and extract and copy to /usr/local/bin/
- Make sure the colud-specific access credentials are available as instructed.
```bash
vi ~/.aws_credentials
# Update below 2 lines.
export AWS_ACCESS_KEY_ID=xxxxxx
export AWS_SECRET_ACCESS_KEY=xxxxx
cd labs
vim main.tf
terraform init # ensures that all required modules are installed.
terraform plan
terraform apply
```

#### Changing Desired State

- After deploying infrastructure, the current state is in the local project directory in a file with the name terraform.tfstate
- Change this file according to your needs.
- Use **terraform validate** to validate the state file.
- Use **terraform plan; terraform apply** to plan and apply the state file.
- Use **terraform destroy** to delete the resources created.

### 5.4 Using Terraform to Manage Kubernetes

#### Understanding Kubernetes Management

- To manage Kubernetes with Terraform, a Kubernetes provider is needed.
- Different providers are available to connect to Kubernetes in cloud as well as through local client configuration.
- After making contact wth Kubernetes,API resources can be created using Terraform.
- When stored in Git repositories, GitOps methodology can be used to manage the Kubernetes application resource lifecycle.
- Notice that Kubernetes internally provides similar solutions, like Kustomization.
- Alternatively, other IaC tools can be used.

#### Demo: Managing Kubernetes Resources

- This demo requires a running instance of Minikube.
- The kubernetes.tf example file is provided in the Git repository.
```bash
cat kubernetes.tf
terraform init
terraform plan
terraform apply
kubectl get all
```
