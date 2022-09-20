# kubernetes
## Preparations
### [Setup GCP](https://github.com/yumaeda/gcp/blob/main/README.md)
### Install packages
```sh
brew install \
  kubectl \
  kubectx \
  kustomize
```
### Enable `container.googleapis.com`
```zsh
gcloud services enable container.googleapis.com
```

&nbsp;

## Create GKE cluster
### Set environment variables
```zsh
export PROJECT_ID=xxxx
export REGION=us-central1
export ZONE=us-central1-a
export CLUSTER_NAME="prod-$REGION-yumaeda"
```
### Execute below command
- From GKE 1.12.0, [`f1-micro` machine type is no longer supported](https://stackoverflow.com/questions/61357217/gcloud-kubernetes-in-f1-micro-results-in-node-pools-of-f1-micro-machines-are-no).

```zsh
gcloud container clusters create \
  --preemptible \
  --project=${PROJECT_ID} \
  --machine-type g1-small \
  --num-nodes 3 \
  --disk-size 10 \
  --zone ${ZONE} \
  --cluster-version latest \
  ${CLUSTER_NAME}
```
### Get a list of GKE cluster
```zsh
gcloud container clusters list
```
### Populate `kubeconfig` file (Add a new kubectl context)
```zsh
gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${ZONE} --project ${PROJECT_ID}
```
### Show list of kubectl contexts
```zsh
kubectx
```
### Show namespace for the current kubectl context
```zsh
kubens -c
```
### Switch kubectl context
```zsh
kubectx ${KUBECTL_CONTEXT}
```

&nbsp;

## Deploy
### Build simplified manifest
```zsh
kustomize build . > deploy.yaml
```
### Create Deployment & Service
```zsh
kubectl apply -f deploy.yaml
```
### List Pods
```sh
kubectl get pods
```
### Create Ingress
```zsh
kubectl apply -f basic-ingress.yaml
```
### Get External IP of the load balancer
```zsh
kubectl get ingress basic-ingress
```

&nbsp;

## Delete
### Delete Ingress
```zsh
kubectl delete ingress basic-ingress
```
### Delete Service
```zsh
kubectl delete service web
```
### Delete Deployment
```zsh
kubectl delete deployment web
```
### Delete GKE cluster
```zsh
gcloud container clusters delete ${CLUSTER_NAME}
```

&nbsp;

## Misc
### Port-forward service to localhost
```zsh
kubectl port-forward svc/web 8080:8080
```
- Access http://localhost:8080/ to make sure that port-forwarding works.
### Execute command within pods
```zsh
kubectl exec --stdin --tty ${POD_NAME} -- /bin/sh
```
### Show logs for the pod
```zsh
kubectl logs ${POD_NAME}
```
### Delete the current pod
```zsh
kubectl delete pod ${POD_NAME}
```
- New pod with different name is automatically created.
### Rollout restarted Deployment
```zsh
kubectl rollout restart deployment/web
```
- Old pod is replaced by new pod.

&nbsp;

## Reference
- https://github.com/socialwifi/docker-nginx-gcs-proxy
