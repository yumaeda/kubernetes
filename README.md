# kubernetes
## Preparations
### [Setup GCP](https://github.com/yumaeda/gcp/blob/main/README.md)
### Install packages
```sh
brew install \
  helm \
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
```
```zsh
export REGION=us-central1 &&
export ZONE=us-central1-a &&
export CLUSTER_NAME="prod-$REGION-sakabas"
```
### Execute below command
```zsh
gcloud container clusters create \
  --preemptible \
  --project=${PROJECT_ID} \
  --machine-type=e2-micro \
  --num-nodes=2 \
  --max-nodes=3 \
  --disk-size=10 \
  --network=sakabas-vpc \
  --subnetwork=subnet-us-central-192 \
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

### Create Namespace
```zsh
kubectl apply -f namespace.yaml
```
### Get a list of Namespaces
```zsh
kubectl get namespaces
```
### Create LimitRange
```zsh
kubectl apply -f limitrange.yaml
```
### Get the LimitRange for the sakabas Namespace
```zsh
kubectl describe limitranges --namespace=sakabas
```
## Create Ingress
```zsh
kubectl apply -f basic-ingress.yaml
```
## Get External IP of the load balancer
```zsh
kubectl describe ingress basic-ingress --namespace=sakabas
```

&nbsp;

## Cleanup
### Delete the LimitRange for the ramen-mania Namespace
```zsh
kubectl delete -f limitrange.yaml
```
## Delete Ingress
```zsh
kubectl delete -f basic-ingress.yaml
```
### Delete the Namespace
```zsh
kubectl delete -f namespace.yaml
```
### Delete GKE cluster
```zsh
gcloud container clusters delete ${CLUSTER_NAME}
```

&nbsp;

## Misc
### Reduce the number of nodes in the k8s cluster
```zsh
gcloud container node-pools create sakabas-pool \
  --cluster ${CLUSTER_NAME} --zone ${ZONE} \
  --machine-type=e2-micro --num-nodes=2 --max-nodes=3 \
  --disk-size=10 --preemptible
```
```zsh
gcloud container node-pools delete default-pool --cluster ${CLUSTER_NAME} --zone ${ZONE}
```
```zsh
kubectl get pod --all-namespaces
```
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
### See current resource usage
```zsh
kubectl top pod -n sakabas
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
### Delete all pods
```zsh
kubectl delete --all pods --all-namespaces
```

&nbsp;

## Reference
- https://docs.datadoghq.com/containers/kubernetes/installation/?tab=operator
- https://istio.io/latest/docs/setup/install/helm/
- https://cloud.google.com/load-balancing/docs/ssl-certificates/troubleshooting?&_ga=2.28635196.-1210616006.1659620185#certificate-managed-status
- https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs
- https://qiita.com/tontoko/items/33faead6bb14370ecb17
