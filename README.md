# Training Guide

## Deploy K3s

```bash
k3sup install --ip=10.68.0.143 --user=root --k3s-version=v1.18.8+k3s1
```

## Kubernetes

### Pod

- criar namespace
kubectl create namespace suse-k8s

``` bash
kubectl apply -f pod.yaml
kubectl logs myapp-pod
kubectl get po -w
kubectl delete po myapp-pod
```

### Deployment

- we can launch random stuff, but this isn't repeatable

``` bash
kubectl create deploy nginx --image=nginx:1.16-alpine
kubectl get deploy
kubectl get po
kubectl delete deploy/nginx
```

- launch again using kustomize templates

```
kubectl create deploy nginx --image=nginx:1.16-alpine --dry-run -o yaml > deployment/base/deployment.yaml
kubectl apply -k deployment/base
kubectl get deploy
kubectl get po
```

- describe pod, look at the image
- scale deployment manually

``` bash
kubectl scale deploy/nginx --replicas=3
kubectl rollout status deploy/nginx
kubectl get deploy
kubectl get po
```

- upgrade with bad image

``` bash
kubectl set image deploy/nginx nginx=nginx:1.17-alpne --record
kubectl rollout status deploy/nginx
kubectl get po
kubectl rollout undo deploy/nginx
```

- redo upgrade from manifest

``` bash
kustomize build deployment/base
```

- edit base to change image and then apply

```bash
kubectl apply -k deployment/base
```

- how can we use this for different environments?

```bash
kustomize build deployment/overlay/staging
kustomize build deployment/overlay/production

kubectl apply -k deployment/overlay/staging
kubectl apply -k deployment/overlay/production
kubectl get deploy
kubectl get pods
```

### ConfigMaps

- show ConfigMaps
- explain what they're for
- explain how they're generated by Kustomize
- they'll show up later

### Services

- show services listening as NodePort
- go look at them

``` bash
curl -I training-a:<port>
```

### Ingress

- show `deployment/overlay/ingress/single/ingress.yaml`

``` bash
kubectl apply -k deployment/overlay/ingress/single
kubectl get ingress
```

- visit <https://training-a.cl.monach.us>
- what about multiple apps?

``` bash
kustomize build deployment/overlay/ingress/fanout
kubectl apply -k deployment/overlay/ingress/fanout 
```

- visit <https://training-a.cl.monach.us> (fail)
- visit <https://training.cl.monach.us/nginx> (works)
- deploy rancher-demo application

```bash
kustomize build rancher-demo/base
kubectl apply -k rancher-demo/base
```
- visit <https://training-a.cl.monach.us/>

### Load Balancer (metalLB)
``` bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.10.2/manifests/metallb.yaml
```


## Rancher

### Server Deploy

```
docker run -d --restart=unless-stopped -p 80:80 -p 443:443 -v /opt/rancher:/var/lib/rancher rancher/rancher:v2.4.5
```

### Node Deploy

- Show how we would deploy an RKE cluster
- Import the `training-a` k3s cluster

### Rancher Server Walkthrough

- Clusters
- Authentication & Security
- Storage
- Projects
- Namespaces
- Catalogs
- CLI/API/Kubectl

### Application Deployment

- show workloads on running cluster
- edit them / delete them
- redeploy `monachus/rancher-demo` as a workload
    - expose port 8080
- put an Ingress in front of it
    - use `training-a.cl.monach.us`
- scale it

## Rancher Application Catalog

- use the Hadoop example
