# AKS CI/CD Configuration

This directory contains Azure Kubernetes Service (AKS) specific configurations for deploying the Vote App.

## Files

- `values-staging.yaml` - Staging environment values
- `values-production.yaml` - Production environment values

## Prerequisites

1. Azure CLI installed and configured
2. AKS cluster created and configured
3. kubectl configured to access your AKS cluster
4. Helm 3.x installed
5. GitHub Secrets configured (see below)

## GitHub Secrets Required

### For GitHub Actions CI/CD

Add the following secrets to your GitHub repository:

#### Azure Credentials
```
AZURE_CREDENTIALS
```
Format:
```json
{
  "clientId": "<service-principal-client-id>",
  "clientSecret": "<service-principal-secret>",
  "subscriptionId": "<azure-subscription-id>",
  "tenantId": "<azure-tenant-id>"
}
```

#### AKS Cluster Information
```
AKS_STAGING_RESOURCE_GROUP=<your-staging-resource-group>
AKS_STAGING_CLUSTER_NAME=<your-staging-cluster-name>
AKS_PRODUCTION_RESOURCE_GROUP=<your-production-resource-group>
AKS_PRODUCTION_CLUSTER_NAME=<your-production-cluster-name>
```

#### Image Registry (if using private registry)
```
POSTGRES_IMAGE_REPO=<your-postgres-image-repo>
POSTGRES_IMAGE_TAG=<your-postgres-image-tag>
REDIS_IMAGE_REPO=<your-redis-image-repo>
REDIS_IMAGE_TAG=<your-redis-image-tag>
VOTE_IMAGE_REPO=<your-vote-image-repo>
VOTE_IMAGE_TAG=<your-vote-image-tag>
RESULT_IMAGE_REPO=<your-result-image-repo>
RESULT_IMAGE_TAG=<your-result-image-tag>
WORKER_IMAGE_REPO=<your-worker-image-repo>
WORKER_IMAGE_TAG=<your-worker-image-tag>
```

## Setup Instructions

### 1. Create Azure Service Principal

```bash
az ad sp create-for-rbac --name "github-actions-sp" --role contributor --scopes /subscriptions/<subscription-id> --sdk-auth
```

Copy the output and add it as `AZURE_CREDENTIALS` secret in GitHub.

### 2. Grant AKS Cluster Access

```bash
# Get your AKS cluster resource ID
AKS_ID=$(az aks show --resource-group <resource-group> --name <cluster-name> --query id -o tsv)

# Get your service principal client ID
SP_CLIENT_ID=$(az ad sp list --display-name "github-actions-sp" --query [0].appId -o tsv)

# Grant contributor role
az role assignment create --assignee $SP_CLIENT_ID --role "Azure Kubernetes Service Cluster Admin Role" --scope $AKS_ID
```

### 3. Configure GitHub Environments

1. Go to your GitHub repository
2. Navigate to Settings > Environments
3. Create `staging` and `production` environments
4. Add required secrets for each environment

## Deployment

### Using GitHub Actions

The deployment happens automatically when:
- Pushing to `develop` branch → deploys to staging
- Pushing to `main` branch → deploys to production
- Manual trigger via workflow_dispatch

### Manual Deployment

```bash
# Staging
helm upgrade --install vote-app-staging ./vote-app-chart \
  --namespace staging \
  --create-namespace \
  -f ./aks/values-staging.yaml

# Production
helm upgrade --install vote-app-production ./vote-app-chart \
  --namespace production \
  --create-namespace \
  -f ./aks/values-production.yaml
```

## Monitoring

After deployment, check the status:

```bash
# Check pods
kubectl get pods -n staging
kubectl get pods -n production

# Check services
kubectl get svc -n staging
kubectl get svc -n production

# Check ingress
kubectl get ingress -n staging
kubectl get ingress -n production
```

## Troubleshooting

### Authentication Issues

If you encounter authentication errors:

```bash
# Login to Azure
az login

# Get AKS credentials
az aks get-credentials --resource-group <resource-group> --name <cluster-name>
```

### Pod Startup Issues

```bash
# Check pod logs
kubectl logs -n <namespace> <pod-name>

# Describe pod for events
kubectl describe pod -n <namespace> <pod-name>
```

### Ingress Issues

```bash
# Check ingress controller
kubectl get pods -n ingress-nginx

# Check ingress configuration
kubectl describe ingress -n <namespace> <ingress-name>
```

