# Lesson 14: Running a GitOps Project

- [Lesson 14: Running a GitOps Project](#lesson-14-running-a-gitops-project)
    - [14.1 Understanding the Project](#141-understanding-the-project)
      - [Understanding the Project](#understanding-the-project)
      - [Project Resources](#project-resources)
      - [Understanding the GitOps Operator](#understanding-the-gitops-operator)
    - [14.2 Preparation: Setting up Git](#142-preparation-setting-up-git)
      - [Demo: Setting up Git](#demo-setting-up-git)
    - [14.3 Preparation: Creating a Worker Image](#143-preparation-creating-a-worker-image)
      - [Demo: Preparing a Workder Image](#demo-preparing-a-workder-image)
      - [Demo: Fetching Images from Private Registries](#demo-fetching-images-from-private-registries)
    - [14.4 Preparation: Setting up Storage](#144-preparation-setting-up-storage)
      - [Demo: Setting up Storage](#demo-setting-up-storage)
    - [14.5 Preparation: Creating the YAML Files](#145-preparation-creating-the-yaml-files)
      - [Demo: Creating YAML Files](#demo-creating-yaml-files)
      - [Demo: Committing the Files to git](#demo-committing-the-files-to-git)
    - [14.6 Implementing the CI Process](#146-implementing-the-ci-process)
      - [Demo: Implementing the CI Process](#demo-implementing-the-ci-process)
    - [14.7 Implementing the CD Process](#147-implementing-the-cd-process)
      - [Demo: Implementing the CD Process](#demo-implementing-the-cd-process)
    - [14.8 Performing the Blue/Green Application Update](#148-performing-the-bluegreen-application-update)

### 14.1 Understanding the Project

#### Understanding the Project

- In this project, you'll create a simple application that runs an Nginx webserver.
- The GitOps procedure is using 2 environments: preprod and prod
- While in preprod, the application has access to preprod data. This data is provided by a Hostpath type PV, giving access to an index.html file with the content "Welcome to preproduction".
- For each environment, a Git repository is created.

#### Project Resources

- The PersistentVolumes need to be created on beforehand and should be available before starting the GitOps project.
- In each project, you will need the following.
  - A Deployment, running the Nginx application.
  - A PersistentVolumeClaim, connecting to the environment storage using a StorageClass.
  - A NodePort Service, providing access to the Deployment.
  - An Ingress resource, providing access to the Service.
- The resources need to be created on beforehand in the Git repositories.

#### Understanding the GitOps Operator

The GitOps Operator should do the following.
- Create or update resources if files are updated in the Git repository.
- Test the preproduction application. In the simplified test, it should check if the application provides a web page that has the text "Welcome to preproduction"
- If it does not, the GitOps process should stop. If it does, it should copy the preprod YAML files to the prod Git repository.
- After copying the files, all occurences of "preprod" should be replaced with "prod".
- Next, the changes in the Git repository should be committed.
- This completes the CI procedure.
- Next, the files should be deployed using **kubectl apply**
- Finally, a Blue/Green migration should be performed.

### 14.2 Preparation: Setting up Git

#### Demo: Setting up Git

- On GitHub: create two Git repos **preprod** and **prod**
- On the Linux machine: create the Git repository directories **/prod** and **/preprod**
```bash
mkdir preprod prod
cd preprod
git init
echo hello > README.md
echo "welcome to preproduction" > index.html # repeat for /prod
git add *
git commit -m 'initial commit'
git branch -m main
# git remote add origin https://github.com/myname/preprod 
git config --global user.email "your@email.com"
git config --global user.name "yourusername"
git push -u origin main # When prompted for a password, use your personal access token.
```

### 14.3 Preparation: Creating a Worker Image

#### Demo: Preparing a Workder Image

```bash
docker run --name gitops -it anilitblr/gitops sh
wget https://github.com/cli/cli/releases/download/v2.23.0/gh_2.23.0_linux_amd64.tar.gz (verify version)
tar xvf gh_2.23.0_linux_amd64.tar.gz
sudo mv gh_2.23.0_linux_amd64/bin/gh /usr/local/bin
gh auth login
git config --global user.email "your@email.com"
git config --global user.name "yourusername"
git push -u origin main # When prompted for a password, use your personal access token.
exit
docker commit gitops yourdockernameher/gittools
docker login
docker push yourdockernamehere/gittools
```

#### Demo: Fetching Images from Private Registries

```bash
ls ~/.docker/config.json

kubectl create scret generic dockercreds --from-file=.dockerconfigjson=/home/anil/.docker/config.json --type=kubernetes.io/dockerconfigjson

docker login
```

### 14.4 Preparation: Setting up Storage

#### Demo: Setting up Storage

```bash
# On minikube host
minikube ssh
sudo mkdir /prod /preprod --mode=777
ls -ld /prod /preprod
exit;

# All files are in labs/losson/14
cd labs/lesson14/
kubectl apply -f preprod-pv.yaml
kubectl apply -f prod-pv.yaml
kubectl get pv

# Preparing for Ingress
sudo sh -c "echo 192.168.1.200 preprod >> /host/hosts"
sudo sh -c "echo 192.168.1.200 prod >> /host/hosts"
ping prod;
ping preprod;
```

### 14.5 Preparation: Creating the YAML Files

#### Demo: Creating YAML Files

```bash
kubectl create ns preprod
kubectl create ns prod
cp pre-prod-pvc.yaml ~/preprod/
cd ~/preprod
kbuectl apply -f preprod-pvc.yaml
kubectl get pv,pvc
kubectl create deploy -n preprod preprod --image=nginx --replicas=3
cat pvc-patch.yaml
kubectl patch -n preprod deploy preprod --patch-file pvc-patch.yaml
kubectl get -n preprod deploy preprod -o yaml | less # to verify
cd ~/preprod/
kubectl get -n preprod deploy preprod -o yaml > preprod-deploy.yaml
vim preprod-deploy.yaml
# Note delete the lines:
# 1. from annotations: to generation: 2
# 2. resourceVersion and uuid
# 3. from status: everything all the way down.
# save and close the file.

kubectl expose deploy -n preprod preprod --type=NodePort --port=80 --dry-run=client -o yaml > preprod-svc.yaml

kubectl apply -f preprod-svc.yaml
kubectl get all -n preprod

kubectl create ing -n preprod preprod-ing --rule="/=preprod:80" --dry-run=client -o yaml > preprod-ing.yaml

kubectl apply -f preprod-ing.yaml
kubectl get all,ing -n preprod
minikube addons enable ingress;
```
- At this poing we have deploy and svc files in git repo and we can clean up the temporary resources that were needed to generate the files. Before doing so, test that all works! `curl preprod` should do.
```bash
kubectl delete -f preprod-ing.yaml
kubectl delete -f preprod-svc.yaml
kubectl delete -f preprod-deploy.yaml
kubectl delete -f preprod.yaml
```

#### Demo: Committing the Files to git

- Now all the files are in the /preprod directory, we just need to commit them to Git.
- We have all files in the Git repo. Time for the git operator to work.
```bash
git add *
git commit -m "YAML files uploaded"
git push
```

### 14.6 Implementing the CI Process

#### Demo: Implementing the CI Process

```bash
kubectl apply -f gitops-preprod-operator.yaml
# Wait for 3 minutes. Did it work? No1
kubectl get all,ing -n preprod
kubeclt describe -n preprod jobs gitops-preprod[Tab]
kubectl create sa gitops -n preprod
kubectl create sa gitops -n prod
kubectl create clusterrolebinding gitops-admin --clusterrole=cluster-admin --serviceaccount preprod:gitops --serviceaccount prod:gitops
# And now you should see completed on the preprod operator pod!
minikube ssh;
cd /prod && ls
cd /preprod && ls
```

### 14.7 Implementing the CD Process

#### Demo: Implementing the CD Process

```bash
vim gitops-prod-operator.yaml
minikube ssh
cd /prod
rm -f preprod-ing.yaml
exit
kubectl apply -f gitops-prod-operator.yaml
```

### 14.8 Performing the Blue/Green Application Update

- **Note:** Look in Lesson 13 for more details.
