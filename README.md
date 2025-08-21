[![DelivOps banner](https://raw.githubusercontent.com/delivops/.github/main/images/banner.png?raw=true)](https://delivops.com)

# Python Package CodeArtifact Deployment Action

A reusable GitHub Action for building and deploying Python packages to AWS CodeArtifact.

## Description

This action automates the process of:
- Building the Python package with a specified version
- Configuring AWS credentials
- Uploading the package to AWS CodeArtifact

## Prerequisites

- AWS IAM role with appropriate permissions for CodeArtifact
- GitHub repository with Python package(s)
- Properly configured `pyproject.toml` or `setup.py` that can use the `VERSION` environment variable

## Usage

### Example: Automatic Version Bumping with Push to Main

This approach automatically creates tags and deploys new versions when you push to main:

```yaml
name: Deploy Python Package

on:
  push:
    branches:
      - main
    paths:
      - 'libs/my-package/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Bump version and push tag
        id: tag_version
        uses: mathieudutour/github-tag-action@v6.2
        with:
          github_token: ${{ github.token }}
          tag_prefix: my-package-v
      
      - name: Deploy to CodeArtifact
        uses: your-username/python-codeartifact-action@v1
        with:
          lib-name: 'my-package'
          lib-version: ${{ steps.tag_version.outputs.new_version }}
          python-version: '3.11'
          aws-region: ${{ secrets.AWS_REGION }}
          aws-account-id: ${{ secrets.AWS_ACCOUNT_ID }}
          codeartifact-domain: ${{ vars.CODEARTIFACT_DOMAIN }}
          codeartifact-repository: ${{ vars.CODEARTIFACT_REPOSITORY }}
```

The `mathieudutour/github-tag-action` automatically bumps the version based on your commit messages:
- `feat:` or `feature:` → Minor version bump
- `fix:` → Patch version bump
- `BREAKING CHANGE:` or `major:` → Major version bump

### Alternative: Deploy When a Tag is Manually Created

If you prefer manual control over versioning:

```yaml
name: Deploy Python Package
on:
  push:
    tags:
      - 'my-package-v*'  # Only run for tags matching this pattern

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Extract version from tag (e.g., my-package-v1.2.3 → 1.2.3)
      - name: Extract version from tag
        id: get_version
        run: echo "VERSION=${GITHUB_REF_NAME#my-package-v}" >> $GITHUB_OUTPUT
      
      - name: Deploy to CodeArtifact
        uses: your-username/python-codeartifact-action@v1
        with:
          lib-name: 'my-package'
          lib-version: ${{ steps.get_version.outputs.VERSION }}
          python-version: '3.11'
          aws-region: ${{ secrets.AWS_REGION }}
          aws-account-id: ${{ secrets.AWS_ACCOUNT_ID }}
          codeartifact-domain: ${{ vars.CODEARTIFACT_DOMAIN }}
          codeartifact-repository: ${{ vars.CODEARTIFACT_REPOSITORY }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `lib-name` | Name of the library to be deployed | Yes | - |
| `lib-path` | Path to the library directory | Yes | `libs/$(lib-name)` |
| `lib-version` | Version of the library to be deployed | Yes | - |
| `python-version` | Python version to use | Yes | - |
| `aws-region` | AWS Region | Yes | - |
| `aws-account-id` | AWS Account ID | Yes | - |
| `aws-role-name` | AWS IAM Role to assume | No | `github_libs` |
| `codeartifact-domain` | AWS CodeArtifact domain | Yes | - |
| `codeartifact-repository` | AWS CodeArtifact repository | Yes | - |

## Package Versioning

This action passes the specified version as an environment variable (`VERSION`) during the build step. Your package's build system should use this environment variable to set the version.

For example, in `setup.py`:

```python
import os
from setuptools import setup, find_packages

version = os.environ.get("VERSION", "0.0.0")

setup(
    name="my-package",
    version=version,
    packages=find_packages(),
    # ... other setup parameters
)
```

Or with a dynamic version in `pyproject.toml`:

```toml
[build-system]
requires = ["setuptools>=42", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my-package"
dynamic = ["version"]
```

With a corresponding `setup.py` or other mechanism to read the `VERSION` environment variable.

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.