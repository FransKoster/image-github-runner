# Development and local testing

This repository includes a `pre-commit` configuration and a Dependabot config.

## Pre-commit (local)

1. Install `pre-commit` (choose one):

   - Using pip (recommended if you have Python installed):

     ```bash
     python3 -m pip install --user pre-commit
     ```

   - Or using Homebrew on macOS:

     ```bash
     brew install pre-commit
     ```

2. Install the git hook in this repository:

   ```bash
   cd /path/to/this/repo
   pre-commit install
   ```

3. Run all checks against all files:

   ```bash
   pre-commit run --all-files
   ```

## Dependabot

- Dependabot is configured in `.github/dependabot.yml` to check weekly for updates to GitHub Actions, Dockerfiles and Python (pip) dependencies.
- Dependabot will open update PRs on this repository as updates are found.

## Notes

- If a hook (like `black` or `flake8`) installs additional tooling, `pre-commit` will manage and install those automatically in an isolated environment.
- Adjust the `.pre-commit-config.yaml` if you want to add or remove hooks specific to this project.

## CI (GitHub Actions)

- A GitHub Actions workflow `/.github/workflows/pre-commit.yml` runs `pre-commit` on `pull_request` events targeting `main`.
- The workflow installs Python and `pre-commit`, then runs `pre-commit run --all-files` so CI enforces the same hooks you run locally.
