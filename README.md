# ADK Agent Kubernetes Architecture

This repository contains the Kubernetes configuration for deploying the ADK (AI Development Kit) Agent in a production-ready, scalable architecture.

## Architecture Overview

![Architecture Diagram](adk-complex.png)

The architecture consists of multiple interconnected components designed for scalability, reliability, and security.

## Components

### Core Applications

#### ADK Agent
- Main application deployment with auto-scaling capabilities
- Configured with health checks and resource management
- Exposes HTTP (8080) and metrics (9090) ports
- Uses persistent storage for data retention

#### Cache Layer
- Redis deployment for high-performance caching
- Single replica for session management
- Resource-limited for cost optimization

#### Database
- PostgreSQL StatefulSet with 3 replicas
- Persistent storage for data durability
- Secure password management through Kubernetes secrets

### Storage

#### Persistent Storage
- **PersistentVolume (PV)**: 10GB storage allocation
- **PersistentVolumeClaim (PVC)**: 5GB storage request
- Used by ADK Agent for data persistence

### Configuration Management

#### ConfigMap
Contains non-sensitive configuration:
- `ADK_LOG_LEVEL`: Logging level configuration
- `ADK_SERVER_PORT`: HTTP server port
- `ADK_METRICS_PORT`: Metrics port for monitoring

#### Secrets
Manages sensitive information:
- API keys
- Database credentials
- Encrypted using Kubernetes secrets

### Networking

#### Services
- **ADK Agent Service**: LoadBalancer type, exposes HTTP and metrics
- **Redis Cache Service**: Internal ClusterIP service
- **Database Service**: Headless service for StatefulSet

#### Ingress
- Nginx Ingress Controller
- Path-based routing (/api)
- SSL termination (configurable)

#### Network Policies
Enforces secure communication between pods:
- Restricts ADK Agent to Redis cache access
- Controls database access
- Implements principle of least privilege

### Scaling and Reliability

#### HorizontalPodAutoscaler (HPA)
- Automatic scaling based on CPU utilization
- Minimum: 1 replica
- Maximum: 10 replicas
- Target CPU utilization: 80%

#### Health Checks
- **Liveness Probe**: Ensures container is running
- **Readiness Probe**: Confirms service availability
- Configurable timing and thresholds

## Resource Requirements

### ADK Agent
```yaml
resources:
  limits:
    memory: "128Mi"
    cpu: "500m"
    ephemeral-storage: "128Mi"
  requests:
    memory: "128Mi"
    cpu: "500m"
    ephemeral-storage: "128Mi"
```

### Redis Cache
```yaml
resources:
  limits:
    memory: "256Mi"
    cpu: "200m"
```

## Deployment

### Prerequisites
- Kubernetes cluster (GKE recommended)
- kubectl configured
- Required secrets and configurations

### Environment Variables
Set the following variables:
```bash
export GOOGLE_CLOUD_PROJECT="your-project"
export GOOGLE_CLOUD_LOCATION="your-location"
export GOOGLE_GENAI_USE_VERTEXAI="true/false"
```

### Deployment Steps
1. Create the namespace:
   ```bash
   kubectl create namespace adk-system
   ```

2. Apply the configuration:
   ```bash
   kubectl apply -f adk.yaml
   ```

3. Verify the deployment:
   ```bash
   kubectl get all -n adk-system
   ```

## Monitoring

### Metrics
- Application metrics available on port 9090
- Compatible with Prometheus monitoring
- Default Kubernetes metrics for HPA

### Logging
- Container logs available through kubectl
- Configurable log levels via ConfigMap
- Integration with Cloud Logging (when on GKE)

## Security

### Authentication
- Service Account based access
- API key management through secrets
- Network policy enforcement

### Network Security
- Pod-to-pod communication restrictions
- Ingress traffic control
- Egress limitations

## Maintenance

### Scaling
- Automatic horizontal scaling via HPA
- Manual scaling possible through kubectl
- StatefulSet scaling for database

### Updates
- Rolling updates configured
- Zero-downtime deployments
- Configurable update strategies

## Troubleshooting

### Common Issues
1. Pod startup failures
   - Check logs: `kubectl logs <pod-name>`
   - Verify resource limits
   - Check health probe configuration

2. Network connectivity
   - Verify network policies
   - Check service endpoints
   - Validate ingress configuration

### Debug Commands
```bash
# Check pod status
kubectl get pods -n adk-system

# View pod logs
kubectl logs -f deployment/adk-agent

# Check service endpoints
kubectl get endpoints

# Verify network policies
kubectl get networkpolicies
```

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details. 