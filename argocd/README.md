# ArgoCD Configuration

This directory contains ArgoCD Application manifests for deploying the Vote App using GitOps methodology.

## Files

- `application.yaml` - Default application configuration
- `application-staging.yaml` - Staging environment application
- `application-production.yaml` - Production environment application
- `app-of-apps.yaml` - Application of Applications pattern for managing all environments

## Prerequisites

1. ArgoCD installed in your Kubernetes cluster
2. ArgoCD CLI installed (`argocd`)
3. Access to ArgoCD server
4. GitHub repository with Helm chart

## ArgoCD Installation

If ArgoCD is not installed, you can install it using:

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD to be ready
kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

## Setup Instructions

### 1. Login to ArgoCD

```bash
# Port forward ArgoCD server
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Login (default username: admin)
argocd login localhost:8080
```

### 2. Add GitHub Repository to ArgoCD

#### For Public Repository

```bash
argocd repo add https://github.com/Ahmedhessn/VOTE-APP-V-3 --type git
```

#### For Private Repository

```bash
# Using SSH
argocd repo add git@github.com:Ahmedhessn/VOTE-APP-V-3.git --ssh-private-key-path ~/.ssh/id_rsa

# Using HTTPS with credentials
argocd repo add https://github.com/Ahmedhessn/VOTE-APP-V-3 \
  --username <github-username> \
  --password <github-token>
```

### 3. Deploy Applications

#### Option A: Deploy Individual Applications

```bash
# Deploy default application
kubectl apply -f argocd/application.yaml

# Deploy staging application
kubectl apply -f argocd/application-staging.yaml

# Deploy production application
kubectl apply -f argocd/application-production.yaml
```

#### Option B: Deploy Using App-of-Apps Pattern

```bash
# Deploy the app-of-apps application
kubectl apply -f argocd/app-of-apps.yaml

# This will automatically create all child applications
```

### 4. Verify Deployment

```bash
# List applications
argocd app list

# Get application status
argocd app get vote-app

# View application details
argocd app get vote-app-staging
argocd app get vote-app-production
```

## Application Configuration

### Default Application (`application.yaml`)

- Deploys to `default` namespace
- Syncs from `main` branch
- Automated sync enabled with self-healing

### Staging Application (`application-staging.yaml`)

- Deploys to `staging` namespace
- Syncs from `develop` branch
- Uses staging values file
- Automated sync enabled

### Production Application (`application-production.yaml`)

- Deploys to `production` namespace
- Syncs from `main` branch
- Uses production values file
- Automated sync with prune disabled (manual approval recommended)

## Sync Policies

### Automated Sync

- **prune**: Automatically remove resources that are no longer in Git
- **selfHeal**: Automatically sync when cluster state differs from Git
- **allowEmpty**: Allow empty applications

### Manual Sync

For production, you may want to disable automated sync and require manual approval:

```yaml
syncPolicy:
  automated:
    prune: false
    selfHeal: false
```

Then sync manually:

```bash
argocd app sync vote-app-production
```

## Accessing ArgoCD UI

```bash
# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Access in browser
# https://localhost:8080
# Username: admin
# Password: (from initial admin secret)
```

## GitOps Workflow

1. **Develop**: Make changes to Helm chart or values
2. **Commit**: Push changes to GitHub repository
3. **ArgoCD**: Detects changes and syncs automatically (if enabled)
4. **Deploy**: ArgoCD applies changes to Kubernetes cluster

### Example Workflow

```bash
# Make changes to values.yaml
vim values.yaml

# Commit changes
git add values.yaml
git commit -m "Update vote app configuration"
git push origin main

# ArgoCD will automatically detect and sync (if automated sync is enabled)
# Or sync manually
argocd app sync vote-app
```

## Monitoring and Troubleshooting

### Check Application Health

```bash
argocd app get vote-app
```

### View Application Logs

```bash
# ArgoCD application logs
argocd app logs vote-app

# Kubernetes resources
kubectl get pods -n <namespace>
kubectl logs -n <namespace> <pod-name>
```

### Sync Status

```bash
# Get sync status
argocd app get vote-app -o json | jq '.status.sync'

# Get health status
argocd app get vote-app -o json | jq '.status.health'
```

### Common Issues

#### Sync Failed

```bash
# Check sync operation details
argocd app get vote-app --refresh

# Retry sync
argocd app sync vote-app --force
```

#### Application Out of Sync

```bash
# Check diff
argocd app diff vote-app

# Sync application
argocd app sync vote-app
```

## Best Practices

1. **Use separate applications for each environment** - Better isolation and control
2. **Disable automated prune for production** - Prevent accidental deletions
3. **Use App-of-Apps pattern** - Manage multiple applications easily
4. **Enable self-healing** - Keep cluster state in sync with Git
5. **Use value files** - Separate configuration per environment
6. **Monitor sync status** - Set up alerts for sync failures

## Integration with GitHub Actions

You can combine GitHub Actions for CI (testing, building) with ArgoCD for CD (deployment):

1. GitHub Actions: Build and test Helm charts
2. GitHub Actions: Push chart to registry or update Git
3. ArgoCD: Automatically deploy from Git repository

This provides a complete CI/CD pipeline with GitOps methodology.

