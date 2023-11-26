# Deploy
## Create Namespace
```zsh
kubectl apply -f namespace.yaml
```
## Get a list of Namespaces
```zsh
kubectl get namespaces
```
## Create LimitRange
```zsh
kubectl apply -f limitrange.yaml
```
## Get the LimitRange for the sakabas Namespace
```zsh
kubectl describe limitranges --namespace=sakabas-nginx
```
## Build simplified manifest
```zsh
kustomize build . > deploy.yaml
```
## Create Deployment & Service
```zsh
kubectl apply -f deploy.yaml
```
## List Pods
```zsh
kubectl get pods --namespace=sakabas-nginx
```
## Create managed certificate
```zsh
kubectl apply -f certificate.yaml
```
## Check the status of the certificate
```zsh
gcloud beta compute ssl-certificates list
```
## Create Ingress
```zsh
kubectl apply -f ingress.yaml
```
## Get External IP of the load balancer
```zsh
kubectl describe ingress web-ingress --namespace=sakabas-nginx
```

&nbsp;

# Delete
## Delete Ingress
```zsh
kubectl delete -f ingress.yaml
```
## Delete Google-managed certificate
```zsh
kubectl delete -f certificate.yaml
```
## Delete Service
```zsh
kubectl delete service web --namespace=sakabas-nginx
```
## Delete Deployment
```zsh
kubectl delete deployment web --namespace=sakabas-nginx
```
## Delete LimitRange
```zsh
kubectl delete -f limitrange.yaml
```
## Delete the Namespace
```zsh
kubectl delete -f namespace.yaml
```
