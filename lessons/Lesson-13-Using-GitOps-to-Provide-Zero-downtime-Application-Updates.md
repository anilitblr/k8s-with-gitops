# Lesson 13: Using GitOps to Provide Zero###downtime Application Updates

- [Lesson 13: Using GitOps to Provide Zero###downtime Application Updates](#lesson-13-using-gitops-to-provide-zerodowntime-application-updates)
    - [13.1 Using Deployment Rolling Updates](#131-using-deployment-rolling-updates)
      - [Understanding Updates](#understanding-updates)
      - [Accessing Applications while Updating](#accessing-applications-while-updating)
      - [Controlling the Update Procedure](#controlling-the-update-procedure)
    - [13.2 Applying Blue/Green Deployment Updates](#132-applying-bluegreen-deployment-updates)
      - [Understanding Blue/Green](#understanding-bluegreen)
      - [Service versus Ingress Blue/Green](#service-versus-ingress-bluegreen)
      - [Demo 1: Ingress Based Blue/Green Deployments](#demo-1-ingress-based-bluegreen-deployments)
    - [13.3 Using Canary Deployments](#133-using-canary-deployments)
      - [Understanding Canary](#understanding-canary)
      - [Service versus Ingress Canary](#service-versus-ingress-canary)
      - [Demo: Ingress Canary Deployments](#demo-ingress-canary-deployments)
    - [13.4 Service-based Canary Deployments](#134-service-based-canary-deployments)
      - [Understanding Service-based Canary](#understanding-service-based-canary)

### 13.1 Using Deployment Rolling Updates

#### Understanding Updates

- While using a Deployment, the update strategy type is set to rolling Update.
- This means that in a Development Pods are upgraded gradually, thus ensuring that there will always be Pods to service user requests.
- The result is that updates are scheduled without downtime.
- Rolling updates work fine for applications that support having multiple versions active at the same time.
- For applications that don't support having multiple versions simultaneously, Recreate can be used as the update strategy type.
- If Recreate is used, old Pods are killed before new Pods are started.

#### Accessing Applications while Updating

- Service and optionally, Ingress is used to provide access to applications.
- The Service object uses a label to connect to Pods with the same label.
- As updatedPods will have the same label as the original Pods, access to the application while updating is guaranteed.

#### Controlling the Update Procedure

- The default application update mechanisms don't provide much control over the update procedure.
- For more control, use Blue/Green or Canary Deployments.
- In Blue/Green the new version is brought online while the old version is still active, which allows for testing the new version without exposing it.
- In Canary, a limited number of new version Pods runs beside the old version Pods, such that a limited amount of users is exposed.

```bash
kubectl create deploy oldnginx --image=nginx:1.13 --replicas=5 --dry-run=client -o yaml > oldnginx.yaml

cat oldnginx.yaml
kubeclt explain deploy.spec.strategy
kubeclt explain deploy.spec.strategy.rollingUpdate

vim oldnginx.yaml
# Update this under strategy
strategy: 
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 3

kubectl apply -f oldnginx.yaml
kubectl get all --selector app=oldnginx

vim oldnginx.yaml
# Change the nginx image version from 1.13 to 1.17
# save and close the file.
kubectl diff -f oldnginx.yaml
kubectl apply -f oldnginx.yaml
kubectl get all --selector app=oldnginx
```

### 13.2 Applying Blue/Green Deployment Updates

#### Understanding Blue/Green

- In a Blue/Green Deployment scenario, the old version Deployment and the new version Deployment run side-by-side.
- While running side-by-side, the new version of the application is not yet exposed.
- After fully testing and verifying the new version of the application, traffic can be swapped.
- To implement Blue/Green Deployment updates, native Kubernetes resources can be used.
- Ecosystem solutions like Argo Rollouts or Istio can be used as well, and offer additional options.

#### Service versus Ingress Blue/Green

- Service Blue/Green is happening at a lower level.
- Existing client connections may get disturbed.
- For better support of existing connections, Ingress can be used.
- Notive that Ingress only works for HTTP/HTTPS-based applications.

#### Demo 1: Ingress Based Blue/Green Deployments

- Part 1: Creating the Blue application

    ```bash
    cd labs/kustomize-bluegreen/blue/
    cat *
    sudo sh -c "echo $(minikube ip) myapp.local >> /etc/hosts"
    kubectl apply -k .
    kubectl get deploy,pods,svc,cm,ing
    curl http://myapp.local
    ```

- Part 2: Creating the Green application

    ```bash
    cd labs/kustomize-bluegreen/green/
    cat *
    kubectl apply -k .
    ```

- Part 3: Making the switch.

    ```bash
    sed -i -e 's/blue-svc/green-svc/' myapp-ing.yaml
    kubectl apply -f myapp-ing.yaml
    curl http://myapp.local
    kubectl get all
    kubectl scale deploy blue-deploy --replicas=0
    ```

### 13.3 Using Canary Deployments

#### Understanding Canary

- A Canary Deployment upgrade strategy will expose a new version of the application to a limited number of users before completing the migration to the version.
- This allows user exposure with a minimized risk.
- If things don't work out will, it's easy to revert to the previous situation by just removing the new application instances.

#### Service versus Ingress Canary

- Canary Deployments can be configured based on Services or Ingress.
- Using Ingress is preferred as teh application picks up the change without reconnecting.
- Service-based Canary Deployments are congigured to use a common selector label on the old as well as the new applications.
- Ingress-based Canary Deployments are using two Ingress resources pointing to the same Ingress virtual host.

#### Demo: Ingress Canary Deployments

```bash
echo new-version > index.html
kubectl create cm new-version --from-file=index.html

echo old-version > index.html
kubectl create cm old-version --from-file=index.html

kubectl get cm
vim canary.yaml
kubectl apply -f canary.yaml
kubectl expose deploy old-version --port=80 --type=NodePort
sed -i -e 's/old/new' canary.yaml
kubectl apply -f canary.yaml

kubectl expose deploy new-version --port=80 --type=NodePort
kubectl get deploy,pods,svc 

sudo sh -c "$(minikube ip) theapp.info >> /etc/hosts"
cat /etc/hosts
kubectl create ing old-version --rule="theapp.info/=old-version:80"
curl theapp.info # all traffic goes to old-version only.
cat new-ing.yaml # notice the annotations
kubectl apply -f new-ing.yaml
curl theapp.info # repeat at least 15 times.
```

### 13.4 Service-based Canary Deployments

#### Understanding Service-based Canary

- Ingress-based Canary Deployments are based on annotations added to the metadata section of the Ingress resource.
- As Ingress works for HTTP/HTTPS applications only, you can't always use it.
- Service-based Canary Deployments are using a selector label.
- The label used as the Service selector must be set on the old as well as the new versions of the application.
- The service will address all Pods that match the selector label.
- Use Replicas to determine how much traffic goes to which version of the application.
- Once complete, use **kubectl scale** to scale the old application down to zero replicas, and the new application to the desired number of replicas.

```bash
# Start by deleting all deploy,svc and ing from the previous demo.
kubectl get svc,deploy,ing
for i in $(kubectl get svc,deploy,ing) | awk '{ print $1}'); do kubectl delete $i; done

sed -i -e 's/new/old' canary.yaml
vim canary.yaml
kubectl apply -f canary.yaml

sed -i -e 's/old/new/' canary.yaml
sed -i -e 's/replicas: 3/replicas: 1/' canary.yaml
kubectl apply -f canary.yaml
kubectl get all
kubectl expose deploy old-version --name=theapp --port=80 --selector type=canary --type=NodePort
kubectl get svc
kubectl describe svc theapp
curl $(minikube ip):<nodeport> # repeat at least 10 times.

# Switch to a new version
kubectl scale deploy new-version --replicas=3
kubectl scale deploy old-version --replicas=0
curl $(minikube ip):<nodeport> # Every request must go to the new-version.
```
