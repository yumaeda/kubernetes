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
export CLUSTER_NAME="prod-$REGION-yumaeda"
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

&nbsp;

## Cleanup
### Delete the LimitRange for the ramen-mania Namespace
```zsh
kubectl delete -f limitrange.yaml
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
gcloud container node-pools create ramen-mania-pool \
  --cluster ${CLUSTER_NAME} --zone ${ZONE} \
  --machine-type=e2-small --num-nodes=2 --max-nodes=3 \
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

## Datadog
### Set environment variable for API Key
```zsh
export DATADOG_API_KEY=xxxx
```
### Install Datadog Agent
```zsh
helm repo add datadog https://helm.datadoghq.com
helm repo update
helm install datadog-agent -f datadog-values.yaml -n ramen-mania --set datadog.site='datadoghq.com' --set datadog.apiKey=${DATADOG_API_KEY} datadog/datadog
```
### Warning
```zsh
###################################################################################
####   WARNING: Cluster-Agent should be deployed in high availability mode     ####
###################################################################################

The Cluster-Agent should be in high availability mode because the following features
are enabled:
* Admission Controller

To run in high availability mode, our recommendation is to update the chart
configuration with:
* set clusterAgent.replicas value to 2 replicas .
* set clusterAgent.createPodDisruptionBudget to true.
```
### Uinstall Datadog Operator
```zsh
helm uninstall datadog-agent -n ramen-mania
kubectl delete configmap datadog-agent-leader-election -n ramen-mania
kubectl delete configmap datadog-agenttoken -n ramen-mania
kubectl delete configmap datadog-cluster-id -n ramen-mania
```

&nbsp;

## Istio
### Preparation
```zsh
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```
### Install
```zsh
kubectl create namespace istio-system
```
```zsh
helm install istio-base istio/base -n istio-system
```
```zsh
helm install istiod istio/istiod -n istio-system --wait
```
```zsh
kubectl create namespace istio-ingress
```
```zsh
kubectl label namespace istio-ingress istio-injection=enabled
```
```zsh
helm install istio-ingress istio/gateway -n istio-ingress --wait
```
```zsh
helm list --namespace=istio-system
```
```zsh
kubectl get pods --namespace=istio-system
```
### Uinstall
```zsh
helm uninstall istio-ingress --namespace=istio-system
```
```zsh
helm uninstall istiod --namespace=istio-system
```
```zsh
helm uninstall istio-base --namespace=istio-system
```
```zsh
kubectl delete namespace istio-system
```
```zsh
kubectl delete namespace istio-ingress
```
```zsh
kubectl get crd -oname | grep --color=never 'istio.io' | xargs kubectl delete
```

&nbsp;

## Reference
- https://docs.datadoghq.com/containers/kubernetes/installation/?tab=operator
- https://istio.io/latest/docs/setup/install/helm/
- https://cloud.google.com/load-balancing/docs/ssl-certificates/troubleshooting?&_ga=2.28635196.-1210616006.1659620185#certificate-managed-status
- https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs
- https://qiita.com/tontoko/items/33faead6bb14370ecb17
