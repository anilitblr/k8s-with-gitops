# Lesson 6: Configuration as Code

- [Lesson 6: Configuration as Code](#lesson-6-configuration-as-code)
    - [6.1 Ansible and GitOps](#61-ansible-and-gitops)
    - [6.2 Setting up Ansible](#62-setting-up-ansible)
      - [Ansible Lab Requirements](#ansible-lab-requirements)
      - [Demo: Setting up Ansible](#demo-setting-up-ansible)
    - [6.3 Managing Configuration as Code with Ansible](#63-managing-configuration-as-code-with-ansible)
      - [Using Ad-hoc Commands](#using-ad-hoc-commands)
      - [Using Playbooks](#using-playbooks)
    - [6.4 Setting up AWX](#64-setting-up-awx)
      - [AWX Setup Requirements](#awx-setup-requirements)
      - [Demo part 1: Installing AWX](#demo-part-1-installing-awx)
      - [Demo part 2: Installing AWX](#demo-part-2-installing-awx)
      - [Demo part 3: Installing AWX](#demo-part-3-installing-awx)
    - [6.5 Configuring Webhooks on AWX](#65-configuring-webhooks-on-awx)
      - [Running Automated Tasks through Webhooks](#running-automated-tasks-through-webhooks)

### 6.1 Ansible and GitOps

- Ansible is used to manage Configuration as Code.
- Configuration as Code is not GitOps by itself, but it's a common part of a GitOps environment.
- If the Ansible command line is used, configuration manifests in Git repositories are started manually.
- If Ansible AWS/Tower/Automation Platform is used, Webooks can be configured on tasks.

### 6.2 Setting up Ansible

#### Ansible Lab Requirements

- On control hosts
  - Use latest version of CentOS.
  - Enable host name resolving for all managed nodes.
  - Generate SSH keys and copy over to managed hosts (optional)
  - Install Ansible software.
  - Create an inventory file.
- On managed hosts
  - Use latest version of Ubuntu Server LTS.
  - Ensure Python is installed.
  - Enable (key-based) SSH access.
  - Make sure you have a user with (passwordless) sudo privileges.

#### Demo: Setting up Ansible

```bash
# On the Ubuntu-based managed hosts.
sudo apt install openssh-server

# On the control host.
sudo dnf install -y ansible-core
sudo sh -c 'echo ip_address ubuntu.example.com ubuntu' >> /etc/hosts
ssh-keygen
ssh-copy-id ubuntu
ssh ubuntu
echo ubuntu >> inventroy
ansible ubuntu -m ping -i inventroy -u student
```

### 6.3 Managing Configuration as Code with Ansible

#### Using Ad-hoc Commands

- Ansible provides 3000+ different modules.
- Modern Ansible versions provide minimal modules in ansible-core additional modules are insetalled using collections.
- See galaxy.ansible.com for collections.
- Modules provide specific functionality and run as Python scripts on managed nodes.
- Use **ansible-doc -l** for a list of modules.
- Modules can be used in ad-hoc commands.
  - **ansible ubuntu -i inventroy -u student -b -K -m user -a "name=anil"**
  - **ansible ubuntu -i inventroy -u student -b -K -m package -a "name=nmap"**

#### Using Playbooks

- Playbooks provide a DevOps way for working with Ansible.
- In a playbook the desired state is defined in YAML.
- The **ansible-playbook** command is used to compare the current state of the managed machine with the desired state, and if they don't match the desired state is implemented.
- **ansible-playbook -i inventroy -u student -K install_and_start_web.yaml**

### 6.4 Setting up AWX

#### AWX Setup Requirements

To set up AWX according to the instructions found here, make sure you have the following.
- One Ubuntu Workstation using the most recent Ubuntu TLS version.
  - 8 GB RAM
  - 4 vCPUs
  - 40 GB disk space
- At least one server that is in a manageable state (see Lession 6.2)

#### Demo part 1: Installing AWX

```bash
sudo apt install git vim -y
cd labs/tower/
./minikube-docker-setup.sh
minikube start --cpus=4 --memory=6g --addons=ingress --vm-driver=docker
minikube addons list
minikube addons enable ingress

curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash

sudo mv kustomize /usr/local/bin/
# Note: check version number by reading kustomization.yaml and then run.
kustomize build . | kubectl apply -f -
kubetl get pods -n awx # Verify using.
kubectl config set-context --current --namespace=awx
kubetl get pods
```

#### Demo part 2: Installing AWX

```bash
vi tower/awx-demo.yaml
# Modify the kustomization.yaml file to add the following extra line below the resources:
vi kustomization.yaml
...
resources:
  - github.com/ansible/awx-operator/config/default?ref=1.1.3
  - awx-demo.yaml
...
kustomize build . | kubectl apply -f -
kubectl get pods,svc # Verify that you have the AWX Pods and Services running (will take a few minutes)
```

#### Demo part 3: Installing AWX

```bash
# Use this command to get the Minikube service URL.
minikube service -n awx awx-demo-service --url
# Get the AWX admin password using.
kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode # Copy the string that is printed, it is your admin password.

# SSH into Minikube
minikube ssh

# Set up host name reslolution in Minikube for the managed hosts.
sudo vim /etc/hosts

# Note: Make sure the managed hosts are in a manageable state (see Lesson 6.2)
```

### 6.5 Configuring Webhooks on AWX

#### Running Automated Tasks through Webhooks

- To run tasks on AWX, a web resources must be configured.
  - Inventroy: contains a list of all managed hosts.
  - Credentials: indentifies the credentials (user accounts) used by Ansible.
  - Project: refers to a Git repository that contains specific resources.
  - Templates: configures the Ansible playbook that should be executed.
- Connection to Git is made on projects and templates.
  - The project is configuted to sync a specific branch.
  - The template can be configured with a webhook, so that it can be triggered when changes in Git are applied.
- Naviage to **https://minikube:port**
  - username: admin
  - passowrd: password
- **Inventories** 
  - Inventories 
    - click on **Add**
    - Choose **Add inventory**
    - Name: **demo-inventory**
    - Click **Save**
  - Hosts
    - Click **Add**
    - Name: **centos**
    - Click **Save**
- **Credentails**
  - Credentails
  - Click **Add**
    - **Create New Credentails**
      - Name: **demo-credentials**
      - Credentails Type: choose **Machine** 
      - **Type Details**
        - Username: **ansible**
        - Password: **password**
        - Privilege Eecalation Method: **sudo**  
        - Privilege Eecalation Username: **root**
        - Privilege Eecalation Password: **password**
        - Click **Save**
- **Projects** 
  - Click **Add**
  - Name: **demo-project**
  - Source Control Type: Git
    - Source Control URL: https://github.com/anilitblr/tower
    - Click **Save**
- **Templates**
  - Click **Add** icon
  - Choose **Add job template**
  - Create New Job Templates
    - Name: **demo-template**
    - Job Type: **Run**
    - Inventory: **demo-inventory**
      - Choose **Select**
    - Project: **demo-project**
      - Choose **Select**
    - Playbook: Choose **vsftpd.yml**
    - Credentials: **demo-credentials**
      - Choose **Select**  
    - Click **Save**
- Click **Templates** in left navigation.
- Click **Lauch Template icon** of **demo-template**
