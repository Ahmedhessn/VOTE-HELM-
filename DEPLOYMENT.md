# Deployment Guide

## Quick Deployment Steps

### 1. Clone or Navigate to Chart Directory

```bash
cd vote-app-chart
```

### 2. Push to GitHub Repository

```bash
# Set the remote (if not already set)
git remote add origin https://github.com/Ahmedhessn/VOTE-HELM-.git

# Push to main branch
git branch -M main
git push -u origin main
```

### 3. Install from GitHub

#### Option A: Install directly from GitHub

```bash
helm install vote-app https://github.com/Ahmedhessn/VOTE-HELM-/archive/refs/heads/main.tar.gz
```

#### Option B: Clone and Install

```bash
# Clone the repository
git clone https://github.com/Ahmedhessn/VOTE-HELM-.git
cd VOTE-HELM-

# Install the chart
helm install vote-app .
```

#### Option C: Add as Helm Repository (if configured)

```bash
# Add repository
helm repo add vote-app https://raw.githubusercontent.com/Ahmedhessn/VOTE-HELM-/main/

# Update repository
helm repo update

# Install
helm install vote-app vote-app/vote-app
```

### 4. Verify Installation

```bash
# Check all resources
kubectl get all

# Check pods
kubectl get pods

# Check services
kubectl get svc

# Check ingress
kubectl get ingress
```

### 5. Access the Application

#### Using Ingress

1. Get the ingress IP:
   ```bash
   kubectl get ingress
   ```

2. Add to `/etc/hosts` (Linux/Mac) or `C:\Windows\System32\drivers\etc\hosts` (Windows):
   ```
   <INGRESS_IP> vote.local
   ```

3. Access:
   - Vote: http://vote.local/
   - Results: http://vote.local/result

#### Using NodePort

- Vote: `<NODE_IP>:30080`
- Results: `<NODE_IP>:30081`

## Customization

### Update Values

Edit `values.yaml` or create a custom values file:

```bash
# Install with custom values
helm install vote-app . -f custom-values.yaml

# Upgrade with new values
helm upgrade vote-app . -f custom-values.yaml
```

### Common Customizations

```yaml
# Scale services
vote:
  replicas: 3
result:
  replicas: 2

# Change vote options
vote:
  config:
    optionA: "Python"
    optionB: "JavaScript"

# Increase PostgreSQL storage
postgres:
  storage:
    size: 10Gi

# Change ingress host
ingress:
  hosts:
    - host: vote.example.com
      paths:
        - path: /
          pathType: Prefix
          service: vote
          port: 80
```

## Troubleshooting

### Check Chart Syntax

```bash
helm lint .
```

### Dry Run Installation

```bash
helm install vote-app . --dry-run --debug
```

### View Generated Manifests

```bash
helm template vote-app . > output.yaml
```

### Uninstall

```bash
helm uninstall vote-app
```

