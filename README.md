# kubernetes
## Install
```sh
brew install kubectl
brew install kubectx
```

## List Clusters
```sh
kubectx
```

## Show Current Context's Cluster
```sh
kubectx -c
```

## Show Current Context's Namespace
```sh
kubens -c
```

## Switch
- To switch your context to one of the clusters displayed by `bubectx`.
```sh
kubectx ${CLUSTER_CONTEXT}
```

## List Pods
```sh
kubectl get pods
```
