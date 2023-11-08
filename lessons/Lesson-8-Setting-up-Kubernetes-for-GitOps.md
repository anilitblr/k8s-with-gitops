# Lesson 8: Setting up Kubernetes for GitOps

- [Lesson 8: Setting up Kubernetes for GitOps](#lesson-8-setting-up-kubernetes-for-gitops)
    - [8.1 Using NameSpaces to Represent GitOps Environments](#81-using-namespaces-to-represent-gitops-environments)
      - [NameSpaces and GitOps](#namespaces-and-gitops)
      - [Namespaces and Environments](#namespaces-and-environments)
      - [Representing Environments](#representing-environments)
    - [8.2 Labels and Annotations](#82-labels-and-annotations)
      - [Understanding Labels and Annotations](#understanding-labels-and-annotations)
      - [Using Annotations](#using-annotations)
      - [Showing Annotations](#showing-annotations)
      - [Demo: Managing Annotations](#demo-managing-annotations)
    - [8.3 Using ConfigMaps to Provide Application Data](#83-using-configmaps-to-provide-application-data)
      - [Providing Configuration](#providing-configuration)
    - [8.4 Kubernetes Storage](#84-kubernetes-storage)
      - [Organizing Storage](#organizing-storage)
    - [8.5 Using Services](#85-using-services)
      - [Demo: Using Services](#demo-using-services)
    - [8.6 Using Ingress](#86-using-ingress)
      - [Understanding Ingress](#understanding-ingress)
      - [Configuring Minikube Ingress](#configuring-minikube-ingress)
      - [Demo: Running the Minikube Ingress Addon](#demo-running-the-minikube-ingress-addon)
      - [Demo: Creating Ingress](#demo-creating-ingress)
    - [8.7 Ingress Access to Services in Specific Namespaces](#87-ingress-access-to-services-in-specific-namespaces)
      - [Understanding Ingress Access to Services](#understanding-ingress-access-to-services)
      - [Demo: Connecting Ingress to Staged Services](#demo-connecting-ingress-to-staged-services)
    - [8.8 Using NetworkPolicy to Isolate GitOps Environments](#88-using-networkpolicy-to-isolate-gitops-environments)
      - [Why NetworkPolicy is Needed](#why-networkpolicy-is-needed)
      - [NetworkPolicy Considerations](#networkpolicy-considerations)
      - [Demo 1: Using NetworkPolicy](#demo-1-using-networkpolicy)
      - [Demo 1: Using NetworkPolicy](#demo-1-using-networkpolicy-1)

### 8.1 Using NameSpaces to Represent GitOps Environments

#### NameSpaces and GitOps

- A NameSpaces is used to create an isolated Environment in Kubernetes.
- Namespaces allow delegation of administration tasks using Role Based Access Control.
- Namespaces also allow for configuration of Quota, as well as network access using NetworkPolicy.
- In GitOps, Namespaces can be used to define the GitOps environment.

#### Namespaces and Environments

- A Namespace should contain all that is required for running an application.
  - The application Pod.
  - Configuration, stored in ConfigMap and Secret.
  - Additional dependencies.
- The Namespace is configured with RBAC for the right access contril.
- NetworkPolicy is taking care of incoming and outgoing network traffic.

#### Representing Environments

- In a GitOps configuration, each environment may be implemented by a Namespace.
- When the application runs in a specific Namespace it has its own dataset, and while approved, it can be moved to the next environment.
- To ensure that only the intended clients can talk to the application while it is in a specific namespaced environment, NetworkPolicy should be added to restrict network traffic.

### 8.2 Labels and Annotations

#### Understanding Labels and Annotations

- Labels and annotations are key/value pairs that are set on Kubernetes resources.
- Labels are used by Kubernetes to connect resources.
- Labels are essential in connecting resources, like Deployments to Pods and Services to Pods.
- Annotations are used by Kubernetes to add an additional descrition.
- Annotations have no specivid meaning to Kubernetes, but can be used by external utilities.
- While labels are managed by Kubernetes, Annotations can be very useful in a GitOps environment.

#### Using Annotations

Annotations can be used in the following cases.
- To allow the GitOps pipelines to leave identifying information when changes have been applied.
- To add specific build information, like image hashes, Git repository SHA, and more.
- Pointers to identify related objects.
- Contact details.

#### Showing Annotations

- To list resources that have a specific annotation, you will need a jsonpath expression.
- When using jsonpath expressions, think about the following.
  - Kubernetes resources are stored in an array, use @ to refer to all items in the array.
  - If an annotation has dots in the name (kubectl.kubernetes.io), you need to escape the dots (kubectl\.kubernetes\.io)

#### Demo: Managing Annotations

```bash
kubectl create deploy annotated --image=nginx
kubectl annotate deploy annotated environment=qa
sudo apt install jq
kubectl get deploy -A -o jsonpath='{.items[0].metadata.annotations}' | jq 

# This prints all deployments that have environment=="qa" in the information.
kubectl get deploy -A -o jsonpath='{.items[?(@.metadata.annotations.environment=="qa")].metadata.name}'

# This prints all deployments that have "last-applied-configuration" annotation.
kubectl get deploy -A -o jsonpath='{.items[?(@.metadata.annotations.kubectl\.kubernetes\.io/last-applied-configuration)].metadata.name}'
```

### 8.3 Using ConfigMaps to Provide Application Data

#### Providing Configuration

Configuration information can be stored in different locations.
- **Container image**: Not recommended, because you would need to recreate the image after each configuration chagne.
- **ConfigMap**: Kubernetes native resources, but Pods need to berestarted if the ConfigMaps is updated.
- **Configuration repository**: may be less integrated, but configuration can be picked up without restarting Pods.

```bash
kubeclt create cm -h | less
kubectl create cm color --from-literal=color=red
kubectl describe cm color
kubectl get deploy
kubectl set env --help | less
kubectl set env --from=configmap/color --prefix=MYSQL_ deployment/webserver
kubectl get all
kubectl exec webserver-xxxxx-xxxxx -- env
```

### 8.4 Kubernetes Storage

#### Organizing Storage

- Container storage by nature is ephemeral.
- Kubernetes Pods may be congigured with volumes.
- These volumes can refer to different types of external storage.
- Alternatively, Persistent Volumes can be used as dedicated objects to point to the specific storage.
- Pods can then be configured with a Persistent Volume Clain (PVC)
- Using PVCs allows for an increased level of flexibility.
- In a GitOps environment, using PVCs is recommended as it allows you to create specific Persistent Volumes in each environment and point to these a a flexible way.

```bash
vim labs/pv.yaml
kubectl create apply -f labs/pv.yaml

kubectl get pv
vim labs/pvc.yaml
kubectl create apply -f labs/pvc.yaml
kubectl get pv,pvc
kubectl get storageclass

vim labs/pv-pod.yaml
kubectl create apply -f labs/pv-pod.yaml
kubectl exec pv-pod -- touch /usr/share/nginx/html/moon.txt
kubectl get pv
kubectl describe pv pv-xxxx-xxxx-xxxx

minikube ssh;
ls -l /tmp/host-path-provisioner/awx/pv-claim
```

### 8.5 Using Services

#### Demo: Using Services

```bash
kubectl create deploy webapp --image=nginx --replicas=3
kubectl expose deploy webapp --port=80 --type=NodePort
kubectl get svc
curl IP-Address:<NodePort>
```

### 8.6 Using Ingress

#### Understanding Ingress

- Ingress can be used as an advanced load balancer to provide access to applications.
- Ingress provides rules that allow for redirection of traffic in a specific direction.
- It takes care of HTTP/HTTPS traffic only.
- To implement Ingess, an Ingress controller is needed.
- Ingress controllers are provided by the echosystem.
- The nginx Ingress controller is commonly used.

#### Configuring Minikube Ingress

- Specific instructions exist to use Ingress in a full Kubernetes cluster.
- Minikube provides easy Ingress access using a Minikube addon.
- Use `minikube addon enable ingress` to enable it.
- Without it, there will be no working Ingress.

#### Demo: Running the Minikube Ingress Addon

```bash
minikube addons list;
minikube addons enable ingress
kubectl get ns
kubectl get pods -n ingress-nginx
# Note: After installing, a patch must be added to the ingress-nginx-controller Deployment configuration.
vim labs/ingress-patch.yaml
kubectl get cm -n ingress-nginx ingress-nginx-controller -o yaml | less
kubectl patch -n ingress-nginx cm ingress-nginx-controller --patch-file ingress-patch.yaml
kubectl get cm -n ingress-nginx ingress-nginx-controller -o yaml | less
kubectl delete pod -n ingress-nginx ingress-nginx-controller-xxxx-xxxx
kubectl get pods -n ingress-nginx
kubeclt config set set-context --current --namespace=default
```

#### Demo: Creating Ingress

```bash
kubectl create deployment nginxsvc --image=nginx
kubectl expose deploy nginxsvc --port=80 --type=NodePort
kubectl get svc
curl http://$(minikube ip):<nodeport>
kubectl create ingress nginxsvc-ingress --rule="/=nginxsvc:80" --rule="/hello=newdep:8080"
sudo vim /etc/hosts
192.168.1.200 nginxsvc.info

kubectl get ingress # wait until it shows an IP Address.
curl nginxsvc.info

kubectl create deployment newdep --image=gcr.io/google-sample/hello-app:2.0
kubectl expose deployment newdep --port=8080
curl nginxsvc.info/hello
```

### 8.7 Ingress Access to Services in Specific Namespaces

#### Understanding Ingress Access to Services

- Ingress can only access Service in the same Namespace
- In a GitOps environment where applications are running in a specific Namespace according to the stage that they care in, direct access from Ingress to these Services is not possible.
- Use an ExternalName Service type to connect to a Service in a different Namespace to make Ingress cross-Namespace access possible.

#### Demo: Connecting Ingress to Staged Services

```bash
kubectl get ns
kubectl get deploy -n preprod
kubectl create deploy preprod --image=nginx -n preprod
kubectl expose deploy preprd -n preprod --port=80
kubectl create svc externalname --help | less
kubectl create svc externalname preprod --external-name preprod.preprod.svc.cluster.local
kubectl describe svc preprod
kubectl create ingress simple --rule="foo.com/=preprod:80"
sudo su -c "echo 192.168.1.200  foo.com >> /etc/hosts"
curl foo.com
```

### 8.8 Using NetworkPolicy to Isolate GitOps Environments

#### Why NetworkPolicy is Needed

- When the application moves through the different environments in the CI/CD pipeline, it is important to implement strict separation.
- If by accident a client from one environment might access the data used by the application in a different environments, it may lead to data corruption or loss.
- This is why NetworkPolicy should be used to block access from outside the environment, which is represented by the Namespace.
- Look for the sample code in `labs/nwp.yaml` file.

#### NetworkPolicy Considerations

- The default Kubernetes (ane Minikube) CNI does not support NetworkPolicy.
- To use a NetworkPolicy, it must be supported by the network plugin; Calico is commonly used.
- If Minikube has been started with the **--cni=calico** option, the NetworkPolicy will work.

#### Demo 1: Using NetworkPolicy

```bash
kubectl create ns prod
kubectl run web --image=nginx -n prod
kubectl expose -n prod pod web --port=80
kubectl get pods,svc -n prod
kubectl run prodpod --image=anil/gitops -n prod -- sleep infinity
kubectl exec -n prod prodpod -- curl web
kubectl run prodpod --image=anil/gitops -n default -- sleep infinity
kubectl exec -n default defaultpod -- curl web.prod.svc.cluster.local
# Up to here, access is unrestricted.
```

#### Demo 1: Using NetworkPolicy

```bash
kubectl apply -f labs/nwp.yaml
kubectl exec -n default defaultpod -- curl web.prod.svc.cluster.local
kubectl exec -n prod prodpod -- curl web
# Note: Now the NetworkPolicy is effective and blocks access to anything happening in the prod Namespace.
```
