# github-actions-validate-release

A GitHub composite action that validates release deployments by checking:
1. Tag format follows semantic versioning
2. Git tag exists in the repository
3. Docker image with the tag exists in AWS ECR

This action is designed to be used as a preflight check before deploying releases, ensuring all prerequisites are met before deployment begins.


## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `tag` | Git/image tag to validate (must follow semantic versioning: v*.*.* ) | Yes | - |
| `ECR_REGISTRY` | Full ECR registry URL including repository name<br/>Example: `123456789.dkr.ecr.us-east-1.amazonaws.com/my-app` | Yes | - |
| `aws-account` | AWS account name to assume role for ECR access | No | `bisnow` |
| `aws-region` | AWS region where ECR repository is located | No | `us-east-1` |

## Usage

### Basic Example

```yaml
jobs: 
  validate-release:
    name: Validate tag and check if image exists
    runs-on: arc-runners-bisnow
    steps:
      - name: Validate Release
        uses: bisnow/github-actions-validate-release@v1.0
        with: 
          tag: ${{ inputs.tag }}
          ECR_REGISTRY: ${{ env.ECR_REGISTRY }}
```

### With Custom AWS Account and Region

```yaml
- name: Validate Release
  uses: bisnow/github-actions-validate-release@v1.0
  with:
    tag: ${{ inputs.tag }}
    ECR_REGISTRY: ${{ env.ECR_REGISTRY }}
    aws-account: production
    aws-region: us-west-2
```

## How It Works

This action performs validation in the following order:

1. **Validate Tag Format**
   - Checks that the tag starts with `v` and follows semantic versioning pattern
   - Pattern: `v[MAJOR].[MINOR].[PATCH][optional-suffix]`
   - Examples: `v1.0.0`, `v2.3.4-beta`, `v1.0.0-rc.1`

2. **Check Git Tag Exists**
   - Checks if the tag exists in the local repository
   - If not found locally, fetches tags from remote and checks again
   - Fails if tag doesn't exist in either location

3. **AWS Authentication**
   - Assumes the specified AWS role for ECR access
   - Uses `bisnow/github-actions-assume-role-for-environment`

4. **Check Docker Image Exists**
   - Extracts repository name from the ECR_REGISTRY URL
   - Queries AWS ECR to verify an image with the specified tag exists
   - Fails if image is not found

   ## Versioning

This action uses rolling major version tags. You can pin to:

- A specific version: `@v3.1.0` (exact, never changes)
- A major version: `@v3` (recommended, gets bug fixes and new features)

When a new semantic version tag (e.g., `v3.2.0`) is pushed, a GitHub Actions workflow automatically updates the corresponding major version tag (`v3`) to point to the new release.