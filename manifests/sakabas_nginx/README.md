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
kubectl describe limitranges --namespace=sakabas
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
kubectl get pods --namespace=sakabas
```
## Create managed certificate
```zsh
kubectl apply -f web-cert.yaml
```
## Create Ingress
```zsh
kubectl apply -f basic-ingress.yaml
```
## Get External IP of the load balancer
```zsh
kubectl describe ingress basic-ingress --namespace=sakabas
```
## Check the status of the certificate
```zsh
gcloud beta compute ssl-certificates list
```
## Create a ServiceAccount
```zsh
kubectl apply -f serviceaccount.yaml
```
## Get a list of ServiceAccounts
```zsh
kubectl get serviceaccounts --namespace=ramen-mania
```
## Create CronJob
```zsh
kubectl apply -f cronjob.yaml
```
## Check a CronJob's configuration
```zsh
kubectl describe cronjob terminated-pod-cleaner --namespace=ramen-mania
```

&nbsp;

# Delete
## Delete Ingress
```zsh
kubectl delete -f basic-ingress.yaml
```
## Delete Google-managed certificate
```zsh
kubectl delete -f web-cert.yaml
```
## Delete Service
```zsh
kubectl delete service web --namespace=sakabas
```
## Delete Deployment
```zsh
kubectl delete deployment web --namespace=sakabas
```
## Delete a CronJob
```zsh
kubectl delete -f cronjob.yaml
```
## Delete the ServiceAccount
```zsh
kubectl delete -f serviceaccount.yaml
```
## Delete the LimitRange for the ramen-mania Namespace
```zsh
kubectl delete -f limitrange.yaml
```
## Delete the Namespace
```zsh
kubectl delete -f namespace.yaml
```
