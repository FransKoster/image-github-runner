# GitHub Actions Self-Hosted Runner Container Image

Build and test a container image for self-hosted GitHub runners designed for Kubernetes deployment with the [Actions Runner Controller](https://github.com/actions/actions-runner-controller).

## Features

- 🐳 Based on the official [GitHub Actions Runner Dockerfile](https://github.com/actions/runner/blob/main/images/Dockerfile)
- 📦 Easy package customization via `packages.txt`
- 🔧 Multi-architecture support (linux/amd64, linux/arm64)
- ☸️ Optimized for Actions Runner Controller on Kubernetes
- 🔐 Includes Docker-in-Docker support
- ✅ Automated build and test workflows

## Quick Start

### Building the Image

```bash
docker build -t github-runner:latest \
  --build-arg RUNNER_VERSION=2.321.0 \
  --build-arg TARGETOS=linux \
  .
```

### Using with Actions Runner Controller

Deploy the runner on Kubernetes using Actions Runner Controller:

```yaml
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: custom-runner
spec:
  replicas: 1
  template:
    spec:
      image: ghcr.io/franskoster/image-github-runner:latest
      dockerdWithinRunnerContainer: true
      repository: your-org/your-repo
```

## Customizing Packages

To add custom packages to your runner image:

1. Edit the `packages.txt` file
2. Add one package name per line
3. Lines starting with `#` are treated as comments
4. Rebuild the image

Example `packages.txt`:
```
# Build tools
build-essential
make

# Python
python3
python3-pip

# Node.js (if needed)
nodejs
npm

# Additional utilities
wget
zip
unzip
```

## Build Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `RUNNER_VERSION` | Required | GitHub Actions runner version (e.g., `2.321.0`) |
| `TARGETOS` | `linux` | Target operating system |
| `TARGETARCH` | Auto-detected | Target architecture (amd64, arm64) |
| `RUNNER_CONTAINER_HOOKS_VERSION` | `0.7.0` | Runner container hooks version for k8s volume mode |
| `RUNNER_CONTAINER_HOOKS_VERSION_NOVOLUME` | `0.8.0` | Runner container hooks version for k8s novolume mode |
| `DOCKER_VERSION` | `29.0.1` | Docker version to install |
| `BUILDX_VERSION` | `0.30.0` | Docker Buildx version to install |

## GitHub Workflows

This repository includes two automated workflows:

### Build Workflow (`.github/workflows/build.yml`)

- Triggers on push to `main`/`develop` branches, tags, and PRs
- Builds multi-architecture images (amd64, arm64)
- Pushes images to GitHub Container Registry (ghcr.io)
- Creates artifact attestations
- Supports manual dispatch with custom runner version

### Test Workflow (`.github/workflows/test.yml`)

- Tests image build process
- Validates image structure and components
- Checks custom package installation
- Runs security vulnerability scans with Trivy
- Lints Dockerfile with hadolint

## Development

### Local Testing

Build and test the image locally:

```bash
# Build the image
docker build -t runner-test:latest \
  --build-arg RUNNER_VERSION=2.321.0 \
  --build-arg TARGETOS=linux \
  .

# Test the image
docker run --rm runner-test:latest id runner
docker run --rm runner-test:latest git --version
docker run --rm runner-test:latest python3 --version
```

### Running Tests

The test workflow can be run locally or via GitHub Actions:

```bash
# Run specific tests
docker run --rm runner-test:latest which python3
docker run --rm runner-test:latest which make
docker run --rm runner-test:latest sudo echo "Sudo works"
```

## Architecture

The image is built in two stages:

1. **Build Stage**: Downloads and extracts the GitHub Actions runner, container hooks, and Docker CLI
2. **Runtime Stage**: Sets up Ubuntu 22.04 (Jammy) with necessary dependencies and custom packages

### Included Components

- GitHub Actions Runner
- Runner Container Hooks (for Kubernetes integration)
- Docker CLI with Buildx plugin
- Git (from git-core/ppa)
- Custom packages from `packages.txt`

## Security

- Regular vulnerability scanning with Trivy
- Minimal base image (dotnet/runtime-deps:8.0-jammy)
- Non-root user execution
- Sudoers configured for CI/CD tasks
- Build provenance attestation

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add/update tests as needed
5. Submit a pull request

## License

This project is based on the official GitHub Actions Runner and follows similar licensing.

## References

- [GitHub Actions Runner](https://github.com/actions/runner)
- [Actions Runner Controller](https://github.com/actions/actions-runner-controller)
- [Runner Container Hooks](https://github.com/actions/runner-container-hooks)
