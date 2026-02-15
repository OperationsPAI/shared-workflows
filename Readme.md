# Shared Workflows

A collection of reusable GitHub Actions workflows and actions designed to simplify CI/CD processes across multiple projects.

## 📦 Contents

### Workflows

#### 1. Helm Release Workflow

Path: `.github/workflows/release-helm.yml`

Automates the Helm chart release process, packaging and publishing charts to GitHub Pages.

**Features:**

- Automatically extracts version from Git tags
- Updates Helm Chart `appVersion`
- Packages and publishes chart to gh-pages branch
- Supports dependency updates and index merging

**Usage:**

```yaml
name: Release Helm Chart

on:
  push:
    tags:
      - "v*"

jobs:
  release:
    uses: <your-org>/shared-workflows/.github/workflows/release-helm.yml@main
    with:
      main-branch: "main" # Optional, default "main"
      chart_dir: "helm" # Optional, default "helm"
      repo_name: "my-helm-repo" # Required
      repo_url: "https://example.com/charts/" # Required
```

**Input Parameters:**

| Parameter     | Type   | Required | Default | Description          |
| ------------- | ------ | -------- | ------- | -------------------- |
| `main-branch` | string | No       | `main`  | Main branch name     |
| `chart_dir`   | string | No       | `helm`  | Helm chart directory |
| `repo_name`   | string | Yes      | -       | Helm repository name |
| `repo_url`    | string | Yes      | -       | Helm repository URL  |

#### 2. Docker Build and Push Workflow

Path: `.github/workflows/release-docker.yml`

Automates building and pushing Docker images to a private registry (Volces) with version tagging.

**Features:**

- Automatically extracts version from Git tags
- Builds multi-platform Docker images
- Pushes to private registry with version and latest tags
- Supports custom build arguments
- Utilizes GitHub Actions cache for faster builds
- Runs on self-hosted runners

**Usage:**

```yaml
name: Release Docker Image

on:
  push:
    tags:
      - "v*"

jobs:
  release:
    uses: <your-org>/shared-workflows/.github/workflows/release-docker.yml@main
    with:
      main-branch: "main" # Optional, default "main"
      volces_name: "myapp/service" # Required
      build_args: | # Optional
        NODE_ENV=production
        APP_VERSION=${{ github.ref_name }}
    secrets:
      VOLCES_USERNAME: ${{ secrets.VOLCES_USERNAME }}
      VOLCES_PASSWORD: ${{ secrets.VOLCES_PASSWORD }}
    vars:
      VOLCES_REGISTRY: ${{ vars.VOLCES_REGISTRY }}
```

**Input Parameters:**

| Parameter     | Type   | Required | Default | Description                                |
| ------------- | ------ | -------- | ------- | ------------------------------------------ |
| `main-branch` | string | No       | `main`  | Main branch to check tag ancestry          |
| `volces_name` | string | Yes      | -       | Image name without registry prefix         |
| `build_args`  | string | No       | `""`    | Multiline string of Docker build arguments |

**Required Secrets:**

- `VOLCES_USERNAME`: Username for the private registry
- `VOLCES_PASSWORD`: Password for the private registry

**Required Variables:**

- `VOLCES_REGISTRY`: Registry URL (e.g., `registry.example.com`)

**Generated Tags:**

- `${VOLCES_REGISTRY}/${volces_name}:${version}`
- `${VOLCES_REGISTRY}/${volces_name}:latest`

### Actions

#### 1. Extract Tag Version

Path: `actions/extract-tag-version/`

Extracts and validates semantic version numbers (SemVer) from Git tags, ensuring proper format and merge status to the main branch.

**Features:**

- Validates tags against SemVer format
- Checks if tag commit is merged to main branch
- Outputs version without `v` prefix
- Customizable SemVer regex pattern

**Usage:**

```yaml
- name: Extract Tag Version
  uses: <your-org>/shared-workflows/actions/extract-tag-version@main
  id: extract_version
  with:
    semver-regex: "^v([0-9]+)\\.([0-9]+)\\.([0-9]+)(?:-([0-9A-Za-z.-]+))?(?:\\+([0-9A-Za-z.-]+))?$"
    main-branch: "main"

- name: Use extracted version
  run: echo "Version is ${{ steps.extract_version.outputs.version }}"
```

**Input Parameters:**

| Parameter      | Type   | Required | Default               | Description                    |
| -------------- | ------ | -------- | --------------------- | ------------------------------ |
| `semver-regex` | string | No       | Standard SemVer regex | Regex pattern to validate tags |
| `main-branch`  | string | No       | `main`                | Main branch name               |

**Outputs:**

| Name      | Description                          |
| --------- | ------------------------------------ |
| `version` | Extracted version without `v` prefix |

**Validation Rules:**

- Tags must follow SemVer format (e.g., `v1.0.0`, `v1.0.0-alpha.1`)
- Tag commit must be merged to the specified main branch
- If validation fails, the action will exit with an error message

## 🚀 Quick Start

1. Reference workflows or actions from this repository in your repository
2. Configure input parameters as needed
3. Ensure your GitHub token has the necessary permissions

## 📋 Prerequisites

- GitHub Actions must be enabled
- For Helm workflow: GitHub Pages must be enabled (gh-pages branch)
- For Docker workflow: Self-hosted runner configured, private registry access
- For version extraction: Tags must follow SemVer format

## 🔧 Permission Requirements

**Helm Release Workflow:**

```yaml
permissions:
  contents: write
```

**Docker Build and Push Workflow:**

```yaml
permissions:
  contents: read
  packages: write
```

## 📝 Examples

For complete usage examples, please refer to the documentation sections for each workflow and action.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
