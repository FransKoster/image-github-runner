# Contributing to GitHub Actions Runner Image

Thank you for your interest in contributing! This guide will help you customize and build the runner image.

## Quick Start

1. Fork this repository
2. Clone your fork
3. Make your changes
4. Test locally
5. Submit a pull request

## Adding Custom Packages

The easiest way to customize the runner is by editing the `packages.txt` file:

```bash
# Edit packages.txt
vim packages.txt

# Add packages, one per line
# Example:
nodejs
npm
terraform
kubectl
```

### Package Format

- One package name per line
- Lines starting with `#` are comments
- Empty lines are ignored
- Use standard Ubuntu package names

## Building Locally

### Prerequisites

- Docker or Podman
- Docker Buildx (for multi-arch builds)

### Build Commands

```bash
# Build for your platform (amd64)
docker build -t github-runner:test \
  --build-arg RUNNER_VERSION=2.321.0 \
  --build-arg TARGETOS=linux \
  .

# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --build-arg RUNNER_VERSION=2.321.0 \
  --build-arg TARGETOS=linux \
  -t github-runner:test \
  .
```

### Testing Your Build

```bash
# Run the container
docker run --rm -it github-runner:test /bin/bash

# Test installed packages
docker run --rm github-runner:test python3 --version
docker run --rm github-runner:test git --version
docker run --rm github-runner:test docker --version

# Check runner installation
docker run --rm github-runner:test ls -la /home/runner/bin
```

## Advanced Customization

### Adding Custom Scripts

Create a custom script and copy it in the Dockerfile:

```dockerfile
# Add after the package installation
COPY scripts/setup.sh /tmp/setup.sh
RUN chmod +x /tmp/setup.sh && /tmp/setup.sh && rm /tmp/setup.sh
```

### Installing from Source

Add custom installation steps in the Dockerfile:

```dockerfile
# Install a tool from source
RUN cd /tmp \
    && curl -LO https://example.com/tool.tar.gz \
    && tar xzf tool.tar.gz \
    && cd tool \
    && make install \
    && cd / \
    && rm -rf /tmp/tool*
```

### Modifying Base Image

To use a different base image, update the `FROM` line:

```dockerfile
FROM mcr.microsoft.com/dotnet/runtime-deps:8.0-noble
```

## Build Arguments

Available build arguments:

| Argument | Default | Description |
|----------|---------|-------------|
| `RUNNER_VERSION` | Required | GitHub Actions runner version |
| `TARGETOS` | `linux` | Target OS |
| `TARGETARCH` | Auto | Target architecture |
| `RUNNER_CONTAINER_HOOKS_VERSION` | `0.7.0` | Container hooks version |
| `DOCKER_VERSION` | `29.0.1` | Docker CLI version |
| `BUILDX_VERSION` | `0.30.0` | Buildx version |

Example with custom versions:

```bash
docker build \
  --build-arg RUNNER_VERSION=2.320.0 \
  --build-arg DOCKER_VERSION=28.0.0 \
  -t github-runner:custom \
  .
```

## Testing Changes

### Lint the Dockerfile

```bash
docker run --rm -i hadolint/hadolint < Dockerfile
```

### Run Tests Locally

```bash
# Build the image
docker build -t runner-test:latest \
  --build-arg RUNNER_VERSION=2.321.0 \
  .

# Test basic functionality
docker run --rm runner-test:latest which python3
docker run --rm runner-test:latest which git
docker run --rm runner-test:latest git --version
```

### Security Scanning

```bash
# Scan for vulnerabilities with Trivy
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image runner-test:latest
```

## Updating Runner Version

To update to a new runner version:

1. Check [GitHub Actions Runner Releases](https://github.com/actions/runner/releases)
2. Update the default version in `.github/workflows/build.yml`
3. Update examples in README.md
4. Test the build
5. Submit a PR

## Pull Request Guidelines

### Before Submitting

- [ ] Test your changes locally
- [ ] Run hadolint on the Dockerfile
- [ ] Update README if adding features
- [ ] Add examples if relevant
- [ ] Update version numbers if applicable

### PR Description

Include:
- What changed and why
- How to test the changes
- Any breaking changes
- Related issues

### Example PR Description

```markdown
## Changes
- Added Node.js 20 to default packages
- Updated README with Node.js example

## Testing
- Built image successfully
- Verified Node.js installation: `node --version` returns v20.x
- Ran sample workflow with Node.js

## Breaking Changes
None
```

## Code Style

### Dockerfile

- Use `apt-get` instead of `apt`
- Include `--no-install-recommends` for packages
- Clean up apt lists: `rm -rf /var/lib/apt/lists/*`
- Group related commands with `&&`
- Add comments for complex operations

### YAML Files

- Use 2 spaces for indentation
- Add comments for complex configurations
- Follow Kubernetes naming conventions

## Release Process

Releases are automated via GitHub Actions:

1. Tag a commit: `git tag v1.0.0`
2. Push the tag: `git push origin v1.0.0`
3. GitHub Actions builds and publishes the image
4. Image is available at `ghcr.io/franskoster/image-github-runner:v1.0.0`

## Getting Help

- Open an [issue](https://github.com/franskoster/image-github-runner/issues) for bugs
- Start a [discussion](https://github.com/franskoster/image-github-runner/discussions) for questions
- Check existing issues and PRs first

## License

By contributing, you agree that your contributions will be licensed under the same license as this project.
