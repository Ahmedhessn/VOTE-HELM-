# Vote App Helm Chart

A Helm chart for deploying a complete Vote Application stack on Kubernetes, including PostgreSQL, Redis, Vote frontend, Result display, and Worker services.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- kubectl configured to access your cluster
- Ingress controller installed (e.g., NGINX Ingress Controller)

## Installation

### Quick Start

```bash
# Add the chart repository (if using a Helm repository)
helm repo add vote-app https://github.com/Ahmedhessn/VOTE-HELM-
helm repo update

# Install the chart
helm install vote-app ./vote-app-chart

# Or install from local directory
helm install vote-app ./vote-app-chart
```

### Install with Custom Values

```bash
# Create a custom values file
cat > my-values.yaml <<EOF
vote:
  config:
    optionA: "Python"
    optionB: "JavaScript"
postgres:
  storage:
    size: 5Gi
ingress:
  hosts:
    - host: vote.example.com
      paths:
        - path: /
          pathType: Prefix
          service: vote
          port: 80
        - path: /result
          pathType: Prefix
          service: result
          port: 4000
EOF

# Install with custom values
helm install vote-app ./vote-app-chart -f my-values.yaml
```

## Configuration

The following table lists the configurable parameters and their default values:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `postgres.enabled` | Enable PostgreSQL deployment | `true` |
| `postgres.image.repository` | PostgreSQL image repository | `01061875164/postgres15-alpine` |
| `postgres.image.tag` | PostgreSQL image tag | `v3` |
| `postgres.replicas` | Number of PostgreSQL replicas | `1` |
| `postgres.port` | PostgreSQL port | `5432` |
| `postgres.database` | PostgreSQL database name | `postgres` |
| `postgres.storage.size` | PostgreSQL PVC size | `1Gi` |
| `postgres.storage.accessMode` | PostgreSQL PVC access mode | `ReadWriteOnce` |
| `postgres.secret.user` | PostgreSQL username | `postgres` |
| `postgres.secret.password` | PostgreSQL password | `postgres` |
| `redis.enabled` | Enable Redis deployment | `true` |
| `redis.image.repository` | Redis image repository | `01061875164/redis` |
| `redis.image.tag` | Redis image tag | `v1` |
| `redis.replicas` | Number of Redis replicas | `1` |
| `redis.port` | Redis port | `6379` |
| `vote.enabled` | Enable Vote service | `true` |
| `vote.image.repository` | Vote image repository | `01061875164/vote-app` |
| `vote.image.tag` | Vote image tag | `v1` |
| `vote.replicas` | Number of Vote replicas | `1` |
| `vote.port` | Vote service port | `80` |
| `vote.service.type` | Vote service type | `NodePort` |
| `vote.service.nodePort` | Vote NodePort | `30080` |
| `vote.config.optionA` | Vote option A | `Cats` |
| `vote.config.optionB` | Vote option B | `Dogs` |
| `result.enabled` | Enable Result service | `true` |
| `result.image.repository` | Result image repository | `01061875164/result` |
| `result.image.tag` | Result image tag | `v3` |
| `result.replicas` | Number of Result replicas | `1` |
| `result.port` | Result service port | `4000` |
| `result.service.type` | Result service type | `NodePort` |
| `result.service.nodePort` | Result NodePort | `30081` |
| `worker.enabled` | Enable Worker service | `true` |
| `worker.image.repository` | Worker image repository | `01061875164/worker` |
| `worker.image.tag` | Worker image tag | `v2` |
| `worker.replicas` | Number of Worker replicas | `1` |
| `ingress.enabled` | Enable Ingress | `true` |
| `ingress.className` | Ingress class name | `nginx` |
| `ingress.hosts` | Ingress hosts configuration | See values.yaml |

## Usage

### Accessing the Application

#### Using Ingress (Recommended)

1. Add the hostname to your `/etc/hosts` file (or Windows `C:\Windows\System32\drivers\etc\hosts`):
   ```
   <INGRESS_IP> vote.local
   ```

2. Access the application:
   - Vote page: `http://vote.local/`
   - Results page: `http://vote.local/result`

#### Using NodePort

If Ingress is not enabled, you can access the services via NodePort:
- Vote service: `<NODE_IP>:30080`
- Result service: `<NODE_IP>:30081`

### Upgrading the Release

```bash
# Upgrade with new values
helm upgrade vote-app ./vote-app-chart -f my-values.yaml

# Upgrade with new chart version
helm upgrade vote-app ./vote-app-chart
```

### Uninstalling the Chart

```bash
helm uninstall vote-app
```

**Note:** This will delete all resources including the PostgreSQL PVC. To preserve data, backup the PVC before uninstalling.

### Scaling Services

```bash
# Scale vote service to 3 replicas
helm upgrade vote-app ./vote-app-chart --set vote.replicas=3

# Scale result service to 2 replicas
helm upgrade vote-app ./vote-app-chart --set result.replicas=2
```

## Architecture

The application consists of:

- **PostgreSQL**: Database for storing vote data
- **Redis**: Cache/message queue for vote processing
- **Vote**: Frontend service for voting
- **Result**: Service for displaying voting results
- **Worker**: Background worker for processing votes

## Troubleshooting

### Check Pod Status

```bash
kubectl get pods -l app=vote
kubectl get pods -l app=result
kubectl get pods -l app=db
kubectl get pods -l app=redis
kubectl get pods -l app=worker
```

### View Logs

```bash
# Vote service logs
kubectl logs -l app=vote

# Result service logs
kubectl logs -l app=result

# Worker logs
kubectl logs -l app=worker

# PostgreSQL logs
kubectl logs -l app=db
```

### Check Services

```bash
kubectl get svc
kubectl get ingress
```

### Check Persistent Volume

```bash
kubectl get pvc
kubectl describe pvc postgres-pvc
```

## Security Considerations

⚠️ **Important**: This chart uses default credentials. For production:

1. **Change default passwords**: Update `postgres.secret.password` in values.yaml or use a Secret management solution
2. **Use TLS**: Enable TLS in the ingress configuration
3. **Network Policies**: Implement NetworkPolicies to restrict pod-to-pod communication
4. **Resource Limits**: Add resource requests and limits to all containers
5. **Security Context**: Configure security contexts to run containers as non-root users

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License.

