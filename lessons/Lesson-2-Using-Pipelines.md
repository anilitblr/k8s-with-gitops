# Lesson 2: Using Pipelines

- [Lesson 2: Using Pipelines](#lesson-2-using-pipelines)
    - [2.1 What is a Pipeline](#21-what-is-a-pipeline)
      - [Understanding Pipeline](#understanding-pipeline)
    - [2.2 Creating Pipelines for DevOps](#22-creating-pipelines-for-devops)
      - [CI/CD Pipelines](#cicd-pipelines)
      - [Common DevOps CI/CD Stages](#common-devops-cicd-stages)
    - [2.3 Creating Pipelines for GitOps](#23-creating-pipelines-for-gitops)
      - [GitOps Pipeline](#gitops-pipeline)
      - [Implementing GitOps CI](#implementing-gitops-ci)
    - [2.4 Integrating DevOps and GitOps Pipelines](#24-integrating-devops-and-gitops-pipelines)
      - [DevOps and GitOps Overlap](#devops-and-gitops-overlap)
    - [2.5 Getting Started with Jenkins](#25-getting-started-with-jenkins)
      - [Why Jenkins?](#why-jenkins)
      - [Using Jenkins](#using-jenkins)
      - [Jenkins Agents](#jenkins-agents)
      - [Running Jenkins as a Sustemd Service](#running-jenkins-as-a-sustemd-service)
    - [2.6 Exploring Pipelines in Jenkins](#26-exploring-pipelines-in-jenkins)
      - [Creating a Jenkins Pipeline](#creating-a-jenkins-pipeline)
      - [Examining a Pipeline Script](#examining-a-pipeline-script)
      - [Demo: Running the Pipeline script](#demo-running-the-pipeline-script)

### 2.1 What is a Pipeline

#### Understanding Pipeline

- Insstructions on computers are processed in pipelines
  - A pipeline is a set of connected data processing elements.
  - The ouput of one element is used a sinput of the next element.
  - Elements of a pipeline can be executed in order or in parallel, depending on the underlying architecture.
- An instruction pipeline defines hwo CPUs are handling their instructions and move them forwrd to the next CPU component.
- A software pipeline defines how the output of one command can be used an input of the next command.
- A CI/CD pipeline moves the software build process through its different stages.

### 2.2 Creating Pipelines for DevOps

#### CI/CD Pipelines

- In DevOps, CI/CD is organized in pipelines.
- The pipeline automates all steps in CI/CD according to the needs of a specific environment.
- In DevOps CI/CD, at the end of the pipeline, updated software is provided.
- In a Kubernetes-based environment, the updated software is delivered using images.

#### Common DevOps CI/CD Stages

In DevOps CI/CD, different stages are commonly used.
- **Pipeline trigger**: the pipeline is triggered automatically when new code is committed. Webhooks are commonly used.
- **Code checkout**: where the code is fetched from the source repository.
- **Code compile**: creates a running application.
- **Run Unit tests**: verifies that application parts work as expected.
- **Code Packaging**: build the container iamge.
- **Automated testing**: ensures the application works as expected.
- **Delivery**: produces artifacts that are ready to be deployed i production.
- **Deployment**: results in a running, updated application without having any human intervention.

### 2.3 Creating Pipelines for GitOps

#### GitOps Pipeline

- DevOps pipelines can also be used for GitOps.
- A main addition to the GitOps pipeline is the working of the GitOps operator.
- In Kubernetes-integrated GitOps, the GitOps operator is a Kubernetes resources that polls the Git repository for changes, and automatically applies updates if changes have been detected.

#### Implementing GitOps CI

- Common phases in GitOps CI include.
  - Build (may occur in DevOps pipeline)
  - Test (may occur in DevOps pipeline)
  - Update application container image (may occur in DevOps pipeline)
  - Clone Git configuration repository
  - Change Kubernetes manifest files
  - Commit changed manifest files back to the Git repository
- Don't forget that steps may be added or skipped according to the needs in a specific environment.
- The mission of GitOps CD is to apply changed manifest to the Kubernetes infrastructure.
- To do so, a GitOps operator may be used to discover changes and apply changed manifests to the Kubernetes cluster.
- Alternatively, generic CI/CD pipeline utilities may be used to discover changes and apply changed manifests to the Kubernetes cluster.

### 2.4 Integrating DevOps and GitOps Pipelines

#### DevOps and GitOps Overlap

- The goal of DevOps is to deliver software.
- The goal of GitOps is to deliver running application.
- These could be organized in one signle pipeline.
- It's more common to organize the DevOps procedure separately in its own pipeline.
- DevOps
  - CI
    1. Code commit.
    2. Code checkout.
    3. Package.
  - CD
    1. Image.
- GitOps
  - CI
    1. Clone GitHub repository.
    2. Change the Kubernetes manifests.
    3. Commit changes to the Git.
  - CD
    1. GitOps Operator (Deploys to the working Kubernetes.)

### 2.5 Getting Started with Jenkins

#### Why Jenkins?

- Jenkins is a software that makes it easier to build pipelines.
- It provides an automated solution that allows you to shape all stages using different steps, using the Git repository as input.
- Using Jenkins is optional, there are other ways to perform GitOps CI/CD if Jenkins is not used.
- Also, alternative solutions are abvailable.
  - AWS CodePipeline
  - Gitlab CI
  - TeamCity
  - and more...

#### Using Jenkins

- A Jenkins job represents a pipeline.
- Jenkins ues Jenkinsfiles to define jobs.
- For easy access in a GitOps environment, Jenkinsfiles are stored i Git repositories.

#### Jenkins Agents

- Different agents are available for Jenkins.
- You don't have to call a specific agent to run generic commands in a Jenkins pipeline.
- Make sure that commands needed in the Jenkins are installed on the Jenkins host.

#### Running Jenkins as a Sustemd Service

- Install Docker, as the sample Jenkinsfile will run Docker commands https://docs.docker.com/engine/install/ubuntu
    ```bash
    # 1. Set up Docker's Apt repository.

    # Add Docker's official GPG key:
    sudo apt-get update
    sudo apt-get install ca-certificates curl gnupg
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg

    # Add the repository to Apt sources:
    echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update

    # 2. Install the Docker packages.
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

    # 3. Verify that the Docker Engine installation is successful by running the hello-world image
    sudo docker run hello-world
    ```
- Install Java
    ```bash
    sudo apt install default-jre
    ```
- Install Jenkins according to the instructions found here https://www.jenkins.io/doc/book/installing/linux/
    ```bash
    curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null

    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    
    sudo apt-get update
    sudo apt-get install jenkins
    sudo usermod -aG docker jenkins
    id jenkins
    sudo systemctl restart jenkins
    ```
- Ensure that optional additional programs that run from the Jenkins are installed as well.

### 2.6 Exploring Pipelines in Jenkins

#### Creating a Jenkins Pipeline

- At this poing, you are ready to create a Jenkins CI/CD pipeline.
- A pipeline is a script that is written in the Groovy syntax.
- In this script, different stages are defined, and each stage consists of a couple of steps.
- These couuespond to the DevOps stages used.
- Jenkins pi;elines can use specific agents to perform tasks.
- When no generic agents are used, shell commands can be executed.
- Each command that is executed as a shell command must be installed on the machine or container that runs Jenkins.

#### Examining a Pipeline Script

- A simple pipeline script, containing different stages.
- Notice the Git repository that is used in this file.
  - Use **git branch:'branchname'** to indicate which branch you want to use.
  - Add **url:** to refer to the specific Git repository.
  - The git repository will be cloned to the local Jenkins environment, and it will be updated each time the Jenkins pipeline runs.

#### Demo: Running the Pipeline script

- Connect to Jenkins Dashboard at http://localhost:8080
- Click **New Item**
- Enter a name: **Jenkinspipeline**
- Select **Pipeline** and click **OK**
- Select **Pipeline** and set **Definition** to **Pipeline Script**
- In the script code section, manually copy contents of **labs/Jenkinspipeline**
- Click **Save**
- In the menu on the left, select **Build Now** and observe what is happening.
- For more details, click the build name and date, and from there select **Console Output** to see messages generat4ed on the console.
