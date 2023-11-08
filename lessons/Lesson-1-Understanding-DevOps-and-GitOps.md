# Lesson 1: Understanding DevOps and GitOps

- [Lesson 1: Understanding DevOps and GitOps](#lesson-1-understanding-devops-and-gitops)
    - [1.1 Understanding DevOps](#11-understanding-devops)
      - [From Ops to DevOps](#from-ops-to-devops)
      - [Continuous Integration](#continuous-integration)
      - [Continuous Delivery/Deployment](#continuous-deliverydeployment)
      - [DevOps](#devops)
      - [Microservices](#microservices)
      - [DevOps Benefits](#devops-benefits)
    - [1.2 Exploring GitOps](#12-exploring-gitops)
      - [What is GitOps](#what-is-gitops)
      - [Why GitOps](#why-gitops)
      - [What to Manage with GitOps](#what-to-manage-with-gitops)
      - [Managing a Systems Desired State](#managing-a-systems-desired-state)
    - [1.3 Kubernetes and GitOps](#13-kubernetes-and-gitops)
    - [1.4 Deploying Everything as Code](#14-deploying-everything-as-code)
      - [Everything as Code](#everything-as-code)
      - [Benefits](#benefits)
    - [1.5 DevOps and GitOps Core Components](#15-devops-and-gitops-core-components)
    - [1.6 DevOps Environments](#16-devops-environments)
      - [Environments](#environments)
    - [1.7 DevOps Stages](#17-devops-stages)
      - [Understanding Stages](#understanding-stages)
      - [Common DevOps Stages](#common-devops-stages)
      - [Common GitOps Stages](#common-gitops-stages)
    - [1.8 Webhooks and Operators](#18-webhooks-and-operators)
      - [Webhooks](#webhooks)
      - [GitOps Operators](#gitops-operators)

### 1.1 Understanding DevOps

#### From Ops to DevOps

- To underdstand DevOps, you need to underdstand traditional IT operations.
- In traditional IT operations, the developersperiodically deliver new software versions to the Qunlity Assurance (QA) team.
- The QA team tests the software and provides it to the operations team for deployment.
- To implement this way of working, ticket systems are commonly used.
- This model works for infrequent release cycles, but not for environments where new releases are needed all the time.

#### Continuous Integration

- While developing software, developer teams often use automated systems to compile, test and produce deployable artifacts.
- The process implemented by these systems is referred to as Continuous Integration (CI).
- Deploying the artifacts requires cooperation with the development team.
- Because different teams are involved, this can be a lengthy process.
- While transitioning from on-premise IT systems to clod-based systems, a new way of working emeerged.
- This was facilitated by programmable APIs provided by these systems.
- Combining CI and Cloud was an essential reason for the start of the DevOps mothodologies.

#### Continuous Delivery/Deployment

- After the DevOps phase of Continuous Integration, the artifacts need to be present for use.
- In Continuous Delivery, the artifacts are delivered to the appropriate location, from where a person can move it into production.
- In Continuous Deployment, the artifacts are delivered to the appropriate location, frmo where is is automatically moved into production.
- Both Continuous Delivery and Continuous Deployment are referred to as the CD in CI/CD.

#### DevOps

- In DevOps, developer and operator pratices are combined into a system that shortens the development life cycle.
- DevOps has a technological aspect, where different technical solutions are combined.
- DevOps also has an organizationl aspect, where developer and operator teams are integrated.
- DevOps is a sttae of mind, and the exact implementation depends on the specific DevOps team.
- You will notice that many ways exist in whichs DevOps is realized.

#### Microservices

- In DevOps, microservices are common.
- A microservices is an application that consists of different components that are independently developed and maintained.
- Microservices are efficient, as components can be reused in different applications.
- Also, because development teams can focus on small, individual components, the development life cycle for microservices can be faster.
- In a containerized cloud world, the components often are provided as container images.

#### DevOps Benefits

- Better collaboration between developers and operators.
- More frequent releases and updates.
- Improved quality.
- Focus on microservice types of applications.
- Decreased cost.

### 1.2 Exploring GitOps

#### What is GitOps

- GitOps provides tools that use DevOps practices and applies them to infrastructure automation and containerized application deployment.
- GitOps applies versison control on Infrasturcture as Code (IaC) and Configuration as Code (CaC) while being stored in Git repositories.
- CI/CD tools and additional management tools are used to deploy that infrasturcture as well as that code into a Kubernetes environment in a fully automated way.
- In GitOps, the Git revision control system is used to track and approve changes.

#### Why GitOps

- By storing code in Git repositorues, the entire application, configuration, and infrasturcture lifecycle is traced.
- Also, as Git allows for storing versions, it is easy to track changes and revert if so required.
- Git-based revision and change control makes it easy to manage every aspect of the environment.

#### What to Manage with GitOps

- Different aspects of the IT environment can be managed using GitOps solutions.
  - Infrastructure
  - Configuration
  - Applications
- Kubernetes is a perfect target for GitOps, as it uses declarative manifest files that can be managed in GitOps.

#### Managing a Systems Desired State

- In GitOps, the desired state of the managed system is stored in Git.
- Changes are first applied to Git branches.
- While in the branch, the change canb e reviewed and approved.
- When changes are approved, they will be merged in the main branch.
- From there, an operator process is used to change the systems current state to the new desired state.

### 1.3 Kubernetes and GitOps

- Running containerized applications is the standard in cloud environments.
- Kubernetes is the solution to orchestrate containerized applications.
- Kubernetes offers different resources to represent any stage of the current configuration.
- Kubernetes offers deployment strategies that integrate perfectly in pipelines that are defined in the GitOps environment.
- Kubernetes has a declarative API, that allows application changes to be pushed to the environment easily.
- Kubernetes operators can automate the process of updating application code that is ready to be updated.

### 1.4 Deploying Everything as Code

#### Everything as Code

- In a GitOps environment, everything is code.
  - Infrastructure as Code: defining which resources to create.
  - configuration as Code: defining how to configure these resources.
  - Application manifest files: defining which applications to run in which way.
- By managing everything as code, managing the lifecycle of everythig is becoming easier.
- As a result, you might not have one git repository containing everything, but multiple projects to define everything as code in your Git environment.

#### Benefits

Running everything as code provides common benefits:
- It makes it easy to repeat the same operations without making erros.
- It's more reliable, as specific knowledge is implemented as a standardized solution.
- Specific solutions provided by the development team and the infrastructure team can be implemented in an automated way, using GitOps operators.

### 1.5 DevOps and GitOps Core Components

- Applications in DevOps as well as infrastructure configuration in GitOps are implemented through environments and stages.
- The environment is where the software is used before it is moved into production.
- Once the software is cleared in one environment, it is cleared to be used in the next environment.
- To manage the software implementation in a specific environment, stages are normally implemented.

### 1.6 DevOps Environments

#### Environments

- Environments define the different phases applied in software development.
- Each environment can have itw own CI/CD pipeline.
  - QA: Quality Acceptance, which is where it is ensured that the software runs on the specific hardware and other sppecific dependencies.
  - E2E: End-toEnd testing, which is about testing the workflow from beginning to end.
  - Staging: This is where all dependencies that exist in the production environment can be tested.
- The pre-production environment consists of QA and E2E, the production environment consists of staging and testing.
- Environments
  - QA
  - E2E
  - Staging
    - Code submission
    - Code review
    - Build
    - Unit test
    - Image build
    - Image push
  - Production

### 1.7 DevOps Stages

#### Understanding Stages

- An environment defines a generic phase in the software lifecycle.
- Each environment consists of different stages, which are steps that lead to the desired results in a specific environment.
- These stages are not carved in stone, but depend on commonly used methodology within a specific department.

#### Common DevOps Stages

- Code submission
- Code review
- Build
- Unit test
- Container image build
- Container image push 

#### Common GitOps Stages

- Build application and deliver as container images (DevOps)
- Write Kubernetes manifest files that use these images and store in Git.
- Apply from Git to pre-release environments.
- Use Kubernetes deployment solutions like canary and blue/green to move from pre-release to production.

### 1.8 Webhooks and Operators

#### Webhooks

- A webhook is a communications method between two APIs.
- The purpose is to update one application when parts of the other application have changed.
- For instance, you can configure webhooks to update your container image after commiting changes to your code in Git.
- Using webhooks is convenient, but in a fully Kubernetes-integrated GitOps environment, the webhook tasks are taken over by the Kubernetes GitOps operator.

#### GitOps Operators

- Kubernetes has operators that can do what webhooks can be doing in a mor eefficient way.
- Such an operator can update the Kubernetes environment by continuously performing a few steps.
  - Use `git clone` to get the latest version of the configuration.
  - Use `kubectl apply` to apply changed manifest code to the Kubernetes cluster.
