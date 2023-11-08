# Lesson 10: Kubernetes CI/CD

- [Lesson 10: Kubernetes CI/CD](#lesson-10-kubernetes-cicd)
    - [10.1 Understanding Kubernetes GitOps CI/CD](#101-understanding-kubernetes-gitops-cicd)
      - [Understanding the Entire Procedure](#understanding-the-entire-procedure)
      - [Understanding Steps in CI and CD](#understanding-steps-in-ci-and-cd)
      - [How to Handle CI and CD](#how-to-handle-ci-and-cd)
    - [10.2 Implementing a CI Pipeline in Kubernetes](#102-implementing-a-ci-pipeline-in-kubernetes)
      - [Identifying Changed Git Repository](#identifying-changed-git-repository)
      - [Using the Git Commit SHA in Image Tags](#using-the-git-commit-sha-in-image-tags)
      - [Using git diff to Update Git](#using-git-diff-to-update-git)
      - [Using kubectl patch](#using-kubectl-patch)
      - [Demo: Using kubectl patch](#demo-using-kubectl-patch)
      - [Implementing the CI Pipeline](#implementing-the-ci-pipeline)
    - [10.3 Implementing CD with a Kubernetes GitOps Operator](#103-implementing-cd-with-a-kubernetes-gitops-operator)
      - [Understanding GitOps Operator Tasks](#understanding-gitops-operator-tasks)
      - [Running the GitOps Operator](#running-the-gitops-operator)

### 10.1 Understanding Kubernetes GitOps CI/CD

#### Understanding the Entire Procedure

- The complete CI/CD Pipeline should tak care of all tasks from the container image update to the automatic application update.
- It makes sense to have two different Git repositoies.
  - The application reposiory takes care of application code and Dockerfile.
  - The configuration Git repository takes care of Kubernetes manifest files.

#### Understanding Steps in CI and CD

- In Continuous Integration, the following should happen after each push of new application code.
  - Clone the application Git repository.
  - Change the Dockerfile and build it.
  - Push the new image to the image registry.
  - Clone the configuration Git repository.
  - Updat the manifest files.
  - Push the new manifest files to Git.
- In Continuous Deployment, the following should happen.
  - Clone the Git config repostiroy.
  - Discover YAML file.
  - Use **kubectl apply** to update current running applications.

#### How to Handle CI and CD

- There are different options for running the Kubernetes integrated CI/CD parts.
  - You may integrate CI as well as CD in a pipeline system like Jenkins.
  - You may run the CI part manually after application changes (identified by git diff).
  - You may run the Cd part automatically as a Kubernetes operator.
  - You could run the CI as well as the CD part as a Kubernetes operator that is continuously polling to find out if any modifications hav happened.
- In this lession, we'll uss a shell script for the CI parts, and we'll use a Kubernetes operator based on the CronJob for the CD part.

### 10.2 Implementing a CI Pipeline in Kubernetes

#### Identifying Changed Git Repository

- Git repositories are identified by the commit SHA, a unique identifier that is related to a specific commit.
- With each commit, the commit SHA changes.
- Alternatively, use **git diff** to see if there are differences between the current state of the local branch and the Git repository.

#### Using the Git Commit SHA in Image Tags

- Using commit SHA can be used as a unique identifier in the application image tag.
  - Use **git rev-parse HEAD** to print the current commit SHA.
  - Make it a bit shorter so that it is usable in an image tag: **git rev-parse HEAD | cut -c1-8**
  - **docker tag myimage myimage:$(git rev-parse HEAD | cut -c1-8)**
- Whenever the Git commit SHA has changed the CI pipeline should geerate a new image.

    ```bash
    git rev-parse HEAD
    git rev-parse HEAD | cut -c1-8
    docker tag myimage myimage:$(git rev-parse HEAD | cut -c1-8)
    ```

#### Using git diff to Update Git

- The **git diff** command identifies if there are differences between the local branch and the remote branch.
- If any differences are detected, the Git repository can be synchronized.
- To avoid conflicts, it might be samrt to manually **git push** changes, if the local branch is more recent.
- If the remote branch is more recent, the local branch can be updated automatically.
- If you are sure that the remote branch is always more up-to-date, you can use **git pull** in an automated way.

#### Using kubectl patch

- To make changes to an existing Kubernetes manifest file, the** kubectl patch** command comes in handy.
- It allows you to apply a patch, which consists of the parameters that have changed to an existing YAML file.

#### Demo: Using kubectl patch

```bash
cd lab/patch
cat deployment.yaml
kubectl apply -f deployment.yaml
cat patch.yaml
echo hello > index.html
kubectl create cm the --from-file=index.html
kubectl patch deploy the-deploy --patch-file patch.yaml
kubeclt get deploy the-deploy -o yaml
```

#### Implementing the CI Pipeline

- To implement the pipeline, a bach script can be used (alongside other scripted solutions).
- To automatically run the CI pipeline, a Kubernetes GitOps operator can be implemented.

### 10.3 Implementing CD with a Kubernetes GitOps Operator

#### Understanding GitOps Operator Tasks

- The mission of the Kubernetes GitOps Operator, is to fetch YAML files from a Git repository, and apply the content of these files to the cluster.
- It is used to implement the Continuous Deployment part of a CI/CD pipeline.
- To create a shell scritp-based Kubernetes GitOps Operator, use a CronJob to run that script.
- The GitOps reconciliation loop starts by cloning the Git repository.
- Next it will check the Git repository for manifest files to apply to the cluster.
- If necessary, the **kubectl apply** command should run automatically to create or update the resources.

#### Running the GitOps Operator

- Operator should be automatically executed.
- The Kubernetes CronJob resource is an excellent way to do so.
- The gitops-operator.yaml file in the repository uses the anil/gitops image to implement a very basic Kubernetes based GitOps operator.
- Notice that this operator is limited: it only runs **kubectl apply** every 2 minutes.
    ```bash
    vim labs/gitops-operator.yaml
    kubectl create sa gitops
    kubectl create clusterrolebinding --help | less
    kubectl create clusterrolebinding gitops-admin --clusterrole=cluster-admin --serviceaccount default:gitops
    kubectl apply -f labs/gitops-operator.yaml
    kubectl get all
    kubectl get jobs,cronjobs,pods
    ```
