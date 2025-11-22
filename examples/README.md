# Example Kubernetes Deployment with Actions Runner Controller

This directory contains examples for deploying the custom GitHub Actions runner image with Actions Runner Controller.

## Prerequisites

- Kubernetes cluster (1.21+)
- [Actions Runner Controller](https://github.com/actions/actions-runner-controller) installed
- GitHub Personal Access Token (PAT) or GitHub App credentials

## Installing Actions Runner Controller

```bash
# Add the Actions Runner Controller Helm repository
helm repo add actions-runner-controller https://actions-runner-controller.github.io/actions-runner-controller

# Install Actions Runner Controller
kubectl create namespace actions-runner-system
helm install actions-runner-controller actions-runner-controller/actions-runner-controller \
  --namespace actions-runner-system \
  --set authSecret.create=true \
  --set authSecret.github_token=YOUR_GITHUB_PAT
```

## Deployment Examples

### 1. Repository-Scoped Runner

Deploy a runner for a specific repository:

```yaml
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: example-runner
  namespace: default
spec:
  replicas: 2
  template:
    spec:
      repository: your-org/your-repo
      image: ghcr.io/franskoster/image-github-runner:latest
      dockerdWithinRunnerContainer: true
      resources:
        limits:
          cpu: "2.0"
          memory: "4Gi"
        requests:
          cpu: "1.0"
          memory: "2Gi"
```

### 2. Organization-Scoped Runner

Deploy a runner for all repositories in an organization:

```yaml
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: org-runner
  namespace: default
spec:
  replicas: 3
  template:
    spec:
      organization: your-org
      image: ghcr.io/franskoster/image-github-runner:latest
      dockerdWithinRunnerContainer: true
      labels:
        - custom-runner
        - docker-enabled
```

### 3. Autoscaling Runner

Deploy with horizontal pod autoscaling:

```yaml
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: autoscaling-runner
  namespace: default
spec:
  template:
    spec:
      repository: your-org/your-repo
      image: ghcr.io/franskoster/image-github-runner:latest
      dockerdWithinRunnerContainer: true
---
apiVersion: actions.summerwind.dev/v1alpha1
kind: HorizontalRunnerAutoscaler
metadata:
  name: autoscaling-runner-autoscaler
  namespace: default
spec:
  scaleTargetRef:
    name: autoscaling-runner
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: TotalNumberOfQueuedAndInProgressWorkflowRuns
    repositoryNames:
    - your-org/your-repo
```

### 4. Runner with Custom Environment Variables

```yaml
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: custom-env-runner
  namespace: default
spec:
  replicas: 1
  template:
    spec:
      repository: your-org/your-repo
      image: ghcr.io/franskoster/image-github-runner:latest
      dockerdWithinRunnerContainer: true
      env:
      - name: CUSTOM_VAR
        value: "custom-value"
      - name: DOCKER_HOST
        value: "unix:///var/run/docker.sock"
```

### 5. Runner with Volume Mounts

```yaml
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: volume-runner
  namespace: default
spec:
  replicas: 1
  template:
    spec:
      repository: your-org/your-repo
      image: ghcr.io/franskoster/image-github-runner:latest
      dockerdWithinRunnerContainer: true
      volumeMounts:
      - name: cache
        mountPath: /home/runner/_work/_cache
      volumes:
      - name: cache
        persistentVolumeClaim:
          claimName: runner-cache-pvc
```

## Using the Runner in Workflows

Once deployed, you can use the runner in your workflows:

```yaml
name: Example Workflow
on: [push]

jobs:
  build:
    runs-on: self-hosted  # Use default label
    # OR specify custom labels:
    # runs-on: [self-hosted, custom-runner, docker-enabled]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build with Docker
        run: docker build -t myapp:latest .
      
      - name: Run Python script
        run: python3 --version
```

## Monitoring and Troubleshooting

### Check Runner Status

```bash
# List runners
kubectl get runners -n default

# Check runner logs
kubectl logs -f <runner-pod-name> -n default

# Describe runner
kubectl describe runner <runner-name> -n default
```

### Common Issues

1. **Runner not appearing in GitHub**: Check authentication and network connectivity
2. **Docker build failures**: Ensure `dockerdWithinRunnerContainer: true` is set
3. **Resource limits**: Adjust CPU and memory based on your workload

## Security Considerations

- Use namespace isolation for different teams/projects
- Implement Pod Security Policies or Pod Security Standards
- Rotate GitHub tokens regularly
- Use private image registries for custom images
- Limit runner permissions using RBAC

## References

- [Actions Runner Controller Documentation](https://github.com/actions/actions-runner-controller)
- [GitHub Self-Hosted Runners](https://docs.github.com/en/actions/hosting-your-own-runners)
