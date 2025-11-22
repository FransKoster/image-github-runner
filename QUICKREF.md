# Quick Reference

## Common Commands

### Build Image Locally

```bash
docker build -t github-runner:latest \
  --build-arg RUNNER_VERSION=2.321.0 \
  --build-arg TARGETOS=linux \
  .
```

### Test Image

```bash
# Check installed packages
docker run --rm github-runner:latest python3 --version
docker run --rm github-runner:latest git --version
docker run --rm github-runner:latest docker --version

# Interactive shell
docker run --rm -it github-runner:latest /bin/bash
```

### Deploy to Kubernetes

```bash
# Apply runner deployment
kubectl apply -f examples/runner-deployment.yaml

# Check status
kubectl get runners
kubectl logs -f <runner-pod-name>
```

### Customize Packages

Edit `packages.txt` and rebuild:

```bash
# Add packages to packages.txt
echo "terraform" >> packages.txt
echo "kubectl" >> packages.txt

# Rebuild
docker build -t github-runner:custom \
  --build-arg RUNNER_VERSION=2.321.0 \
  .
```

## Useful GitHub Workflow Examples

### Use Self-Hosted Runner

```yaml
jobs:
  build:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      - run: make build
```

### With Custom Labels

```yaml
jobs:
  build:
    runs-on: [self-hosted, docker, linux]
    steps:
      - uses: actions/checkout@v4
      - run: docker build .
```

## Kubernetes Commands

```bash
# Install Actions Runner Controller
helm install actions-runner-controller \
  actions-runner-controller/actions-runner-controller \
  --namespace actions-runner-system \
  --set authSecret.create=true \
  --set authSecret.github_token=YOUR_TOKEN

# Scale runners
kubectl scale runnerdeployment github-runner --replicas=5

# Delete runners
kubectl delete runnerdeployment github-runner
```

## Troubleshooting

### Runner Not Appearing

```bash
# Check logs
kubectl logs <runner-pod-name>

# Check runner status
kubectl describe runner <runner-name>
```

### Build Failures

```bash
# Run hadolint
docker run --rm -i hadolint/hadolint < Dockerfile

# Check syntax
docker build --no-cache .
```

### Permission Issues

```bash
# Check user
docker run --rm github-runner:latest id

# Test sudo
docker run --rm github-runner:latest sudo echo "test"
```
