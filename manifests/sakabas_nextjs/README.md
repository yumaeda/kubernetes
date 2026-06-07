# Sakabas Next.js Deployment

Kubernetes manifests for deploying the Sakabas Next.js application.

## Overview

This directory contains Kubernetes manifests for deploying a Next.js application to a GKE cluster using nginx as the ingress controller.

## Prerequisites

- Kubernetes cluster (GKE recommended)
- kubectl configured to access the cluster
- nginx-ingress controller installed in the cluster

## Namespace

The application is deployed to the `sakabas-nextjs` namespace.

```bash
kubectl apply -f namespace.yaml
```

## Deployment

### Apply All Resources

```bash
kubectl apply -f .
```

### Apply Individual Resources

```bash
kubectl apply -f namespace.yaml
kubectl apply -f configmap.yaml
kubectl apply -f secrets.yaml
kubectl apply - deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

## Configuration

### Environment Variables

Configure environment variables in the deployment:

- `NODE_ENV`: Set to `production`
- `PORT`: Application port (default: 3000)

For secrets (DATABASE_URL, API keys, etc.), use the secrets.yaml file:

```bash
kubectl create secret generic sakabas-nextjs-secrets \
  --from-literal=DATABASE_URL="postgresql://..." \
  --from-literal=API_SECRET_KEY="..." \
  -n sakabas-nextjs
```

### Ingress Configuration

Update the following in `ingress.yaml`:

- `your-domain.com`: Replace with your actual domain
- `sakabas-nextjs-tls`: Update with your TLS secret name

TLS is automatically configured via nginx ingress.

### Resource Limits

Current resource allocation:

- **Requests**: 256Mi memory, 250m CPU
- **Limits**: 512Mi memory, 500m CPU

Adjust these values based on your application's needs.

## Scaling

To scale the deployment:

```bash
kubectl scale deployment sakabas-nextjs -n sakabas-nextjs --replicas=3
```

## Monitoring

### Check Deployment Status

```bash
kubectl get pods -n sakabas-nextjs
kubectl get deployment -n sakabas-nextjs
```

### View Logs

```bash
kubectl logs -f deployment/sakabas-nextjs -n sakabas-nextjs
```

### Access Application

The application is accessible at `https://your-domain.com`

## Security

- Container runs as non-root user
- Read-only root filesystem enabled
- Privilege escalation disabled
- Secrets managed via Kubernetes Secrets

## Troubleshooting

### Check Events

```bash
kubectl describe pod -n sakabas-nextjs
kubectl get events -n sakabas-nextjs --sort-by='.lastTimestamp'
```

### Debug Configuration

```bash
kubectl get configmap sakabas-nextjs-config -n sakabas-nextjs -o yaml
kubectl get secrets -n sakabas-nextjs
```

## Uninstall

```bash
kubectl delete -f .
```
