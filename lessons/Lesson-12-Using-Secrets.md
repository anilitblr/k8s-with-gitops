# Lesson 12: Using Secrets

- [Lesson 12: Using Secrets](#lesson-12-using-secrets)
    - [12.1 Providing Configuration](#121-providing-configuration)
      - [Providing Configuration](#providing-configuration)
      - [Demo: Using CongigMap](#demo-using-congigmap)
    - [12.2 Using Secrets](#122-using-secrets)
      - [Secrets are not Secret!](#secrets-are-not-secret)
      - [Use Case for Secrets](#use-case-for-secrets)
      - [Using Secrets](#using-secrets)
      - [Using Secrets for Environment Variables](#using-secrets-for-environment-variables)
      - [Retrieving Secrets from the Kubernetes API](#retrieving-secrets-from-the-kubernetes-api)
      - [Demo: Retrieving Secrets from the K8s API](#demo-retrieving-secrets-from-the-k8s-api)
    - [12.3 Secrets in GitOps](#123-secrets-in-gitops)
      - [Keeping Secrets Secret in GitOps](#keeping-secrets-secret-in-gitops)
      - [Using Secrets in a GitOps Environment](#using-secrets-in-a-gitops-environment)
    - [12.4 Bitnami SealedSecrets](#124-bitnami-sealedsecrets)
      - [Using Bitnami SealedSecrets](#using-bitnami-sealedsecrets)
      - [kubeseal and Namespaces](#kubeseal-and-namespaces)
      - [Demo: Using Bitnami Sealed Secrets](#demo-using-bitnami-sealed-secrets)

### 12.1 Providing Configuration

#### Providing Configuration

- To guarantee the flexibility in the GitOps environment, configuration should be stored in external objects.
- This allows you to use different configurations in different GitOps environments, while still deeping the same application manifests.
- ConfigMaps are provided by Kubernetes to store contents in ASCII format.
- Secrets are provided to store contents in a base64 encoded format.
- ConfigMaps and Secrets are used to store environment variables, startup parameters, congiguration files, TLS keys and other keys and passwords.

#### Demo: Using CongigMap

```bash
kubectl create cm dbvars --from-literal=MARIADB_ROOT_PASSWORD=password
kubectl describe cm dbvars
kubectl get deploy
kubectl get pods
kubectl create deploy mydb --image=mariadb
kubectl set env --from=configmap/dbvars deploy/mydb
kubectl get all --selector app=mydb
kubectl get pods mydb-xxxxx-xxxxx
```

### 12.2 Using Secrets

#### Secrets are not Secret!

- A Secret is Base64 encoded.
- Base64 encoding is NOT encryption.
- Using Base64 encoding on a Secret allows for storage of binary data as ASCII strings.
- Without Base64 encoding it is not possible to store binary configuration data such as TLS certificates.

#### Use Case for Secrets

- A Secret is used to separate configuration from the container images.
- The main reason for using Secrets is to encode binary data in a storable format.
- For the rest of it, Secrets are as insecure as ConfigMaps.

#### Using Secrets

Secrets are used in three different ways.
- Mounted as volumes in a Pod.
- Set as environment variables.
- Retrieved directly from the Kubernetes API.

#### Using Secrets for Environment Variables

- If used to provide environment variables, the environment variables are better stored on a volume and mounted as such.
- If set as an environment variable, the environment variable in the Secret is exposed to all applications running in the Pod.
- Secrets that are provided as environment varibles cannot be updated without restarting the Pod.

#### Retrieving Secrets from the Kubernetes API

- From the Pod, the Secret value can be accessed directly from the Kubernetes API instead of mounting it, or accessing it as an environment variable.
- This has the benefit that the Secret is never stored anywhere on the Pod.
- It does however involve additional work, while still not making the Secret secure.
  - It does require a ServiceAccount that allows the Pod to obtain the Secret from the API.
  - It requires the **kubectl** utility to be present within the Pod.

#### Demo: Retrieving Secrets from the K8s API

```bash
kubectl create secret generic mypassword --from-literal password=password
kubectl get secret mypassword -o jsonpath='{.data.password}' | base64 --decode
kubectl create sa admin
kubectl get clusterroles | less
kubectl create clusterrolebinding newadmin --clusterrole=cluster-admin --serviceaccount=default:admin 
kubectl run gittools --image=anil/gitops --dry-run=client -o yaml -- sleep infinity > gittools.yaml
vim gittools.yaml # add spec.serviceAccount: admin above status: {}
kubectl apply -f gittools.yaml
kubectl exec -it gittools -- kubectl get secret mypassword -o jsonpath='{.data.password}' | base64 --decode
```

### 12.3 Secrets in GitOps

#### Keeping Secrets Secret in GitOps

- As Secrets are not so very Secret, they deserve a special treatment in GitOps.
- They should NOT be stored inm Git, repositories, a they are easily decoded.

#### Using Secrets in a GitOps Environment

- Building Secrets in the container image is not a good idea.
  - It would make the container image sensitive.
  - It would take away the separation of configuration and code.
- Manual application weaknens the overall GitOps performance.
- The Kustomize Secret generator might be the best Kubernetes native solution.
- Bitnami Sealed Secret uses a CustomResourceDefinition to create really secure secrets and extract Kubernetes Secrets from those.
- External Secret managers like Hashicorp Vault could also be used.

### 12.4 Bitnami SealedSecrets

#### Using Bitnami SealedSecrets

- Bitnami offers a solutioni that encrypts Kubernets Secrets.
- This solution creates a controller that is based on a CRD, and which takes a Kubernetes Secret YAML manifest file as input.
- It generates a SealedSecret, which can next be used, and safely stored in a Git repository.
- The SealedSecret is encrypted with the private key of the SealedSecret controller, and for that reason will dynamically be decrypted when used in the same cluster.
- When used, the SealedSecret creates a Kubernetes Secret, which in the Kubernetes cluster still isn't encrypted, but now doesn't need to be stored in a Git repository anymore.

#### kubeseal and Namespaces

- By default, the **kubeseal** generated encryptedData in the SealedSecret is valid in the current Namespace only.
- This adds protection, but in a GitOps environment, limits the usability as it complicates using the SealedSecret in different Namespaces that represent the GitOps environments.
- Use **kubeseal --scope cluster-wide** to make the SealedSecret available in the entire cluster.

#### Demo: Using Bitnami Sealed Secrets

```bash
# Downlaod kubeseal
https://github.com/bitnami-labs/sealed-secrets/releases

tar xvf kubeceal-xxxx
sudo mv kubeseal /usr/local/bin/
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.19.5/controller.yaml
kubectl create secret generic mypassword --from-literal=password=password --dry-run=client -o yaml > /tmp/mypassword.yaml
kubeseal -o yaml --scope cluster-wide < /tmp/mypassword.yaml > mysealedpassword.yaml
# Store mysealedpassword.yaml in Git, and remove /tmp/mypassword.yaml
kubectl get secret
kubectl delete secret mypassword
kubectl apply -f mysealedpassword.yaml
kubectl get sealedsecret,secret
```
