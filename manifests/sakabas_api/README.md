# Deploy
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
kubectl get pods --namespace=sakabas
```
## Create managed certificate
```zsh
kubectl apply -f certificate.yaml
```
## Check the status of the certificate
```zsh
kubectl get managedcertificate --namespace=sakabas
```

&nbsp;

# Delete
## Delete Google-managed certificate
```zsh
kubectl delete -f certificate.yaml
```
## Delete Service
```zsh
kubectl delete service api --namespace=sakabas
```
## Delete Deployment
```zsh
kubectl delete deployment api --namespace=sakabas
```
