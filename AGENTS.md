# Agent Guidelines for Kubernetes Project

## Overview

This document defines how AI agents should interact with the Kubernetes project, including patterns for manifest modifications, deployment operations, and troubleshooting.

## Manifest Modification Patterns

### Updating Deployment Images

When updating container images, modify `deployment.yaml`:

```yaml
spec:
  template:
    spec:
      containers:
      - name: <service-name>
        image: "us-central1-docker.pkg.dev/hello-world-352201/<service>/sakabas-<service>:latest"
        imagePullPolicy: Always
```

### Scaling Deployments

Use `kubectl scale` for quick scaling:

```bash
kubectl scale deployment <deployment-name> -n <namespace> --replicas=<count>
```

For persistent changes, edit `deployment.yaml` and update the `replicas` field.

### Updating Environment Variables

**ConfigMap values** (non-sensitive):
Edit `configmap.yaml` or use `kubectl edit configmap`:

```bash
kubectl edit configmap <configmap-name> -n <namespace>
```

**Secrets** (sensitive):
Use `kubectl create secret` with `--from-literal`:

```bash
kubectl create secret generic <secret-name> \
  --from-literal=<key>=<value> \
  -n <namespace>
```

Or edit existing secrets:

```bash
kubectl edit secrets <secret-name> -n <namespace>
```

### Resource Adjustments

Modify `resources.requests` and `resources.limits` in `deployment.yaml`:

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

## Service-Specific Guidelines

### sakabas-api

- **Image**: `us-central1-docker.pkg.dev/hello-world-352201/sakabas-api/sakabas-api:latest`
- **Port**: 8080
- **Dependencies**: TiDB, AWS S3
- **Secrets Required**: `tidb-pwd`, `tidb-host`, `tidb-name`, `tidb-user`, `s3-id`, `s3-secret`, `s3-region`, `db-pwd`, `db-host`, `db-name`, `db-user`

### sakabas-nginx

- **Image**: `us-central1-docker.pkg.dev/hello-world-352201/nginx-gcs-proxy/nginx-gcs-proxy:latest`
- **Port**: 80
- **Purpose**: GCS proxy for serving static content

### sakabas-nextjs

- **Image**: `your-registry/sakabas-nextjs:latest` (needs to be configured)
- **Port**: 3000
- **Environment**: NODE_ENV=production
- **Health Checks**: HTTP GET on `/`

## Common Operations

### Rolling Restart

```bash
kubectl rollout restart deployment/<deployment-name> -n <namespace>
```

### Rollback

```bash
kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

### View Deployment History

```bash
kubectl rollout history deployment/<deployment-name> -n <namespace>
```

### Delete and Recreate

```bash
kubectl delete -f <manifest-file>
kubectl apply -f <manifest-file>
```

## Troubleshooting Patterns

### Pod Not Starting

1. Check pod status:
   ```bash
   kubectl get pods -n <namespace>
   ```

2. Describe the pod:
   ```bash
   kubectl describe pod <pod-name> -n <namespace>
   ```

3. Check events:
   ```bash
   kubectl get events -n <namespace> --sort-by='.lastTimestamp'
   ```

4. View logs:
   ```bash
   kubectl logs <pod-name> -n <namespace>
   kubectl logs -f deployment/<deployment-name> -n <namespace>
   ```

### Image Pull Errors

Verify image exists in GCR:
```bash
gcloud container images list --repository=us-central1-docker.pkg.dev/hello-world-352201
```

### Service Connectivity Issues

1. Check service endpoints:
   ```bash
   kubectl get endpoints -n <namespace>
   ```

2. Test with port-forward:
   ```bash
   kubectl port-forward svc/<service-name> 8080:80 -n <namespace>
   ```

3. Check ingress:
   ```bash
   kubectl describe ingress <ingress-name> -n <namespace>
   ```

## Security Best Practices

1. **Never commit secrets**: All sensitive data must use Kubernetes Secrets
2. **Use non-root users**: All containers run as non-root
3. **Read-only filesystem**: Enable `readOnlyRootFilesystem: true`
4. **Resource limits**: Always set both requests and limits
5. **TLS certificates**: Use Google-managed certificates via Ingress

## Agent Task Patterns

### For Deployment Changes

1. Review the current manifest
2. Identify the target service (api, nginx, or nextjs)
3. Make targeted changes to the appropriate file
4. Provide the deployment command

### For Troubleshooting

1. Ask for the specific symptom (pod not starting, connectivity issue, etc.)
2. Request relevant diagnostic commands
3. Interpret the output and suggest fixes

### For Resource Optimization

1. Check current resource usage:
   ```bash
   kubectl top pod -n <namespace>
   ```

2. Review current limits in manifests
3. Suggest adjustments based on actual usage

## Files to Avoid Modifying

- `.gitignore` - Repository configuration
- `README.md` - Main documentation (unless requested)
- Manifest files in other service directories when working on one service

## When to Escalate

- GCP project-level changes (VPC, subnet, firewall)
- Cluster infrastructure changes
- Cross-service dependency changes
- Security-related modifications beyond manifest updates
