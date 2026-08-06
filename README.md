# Pipeline Helper Docker Image

A pre-built Docker image containing essential DevOps tools for CI/CD pipelines. Designed for use across multiple organizations and projects.

**Publicly available at:** `ghcr.io/ujam-dev/pipeline-helper`

## What's Included

- **Node.js 24** - JavaScript runtime
- **Terraform** - Infrastructure as Code
- **AWS CLI** - Amazon Web Services command-line tools
- **curl** - Data transfer tool
- **jq** - JSON query processor
- **git** - Source control tooling for commit metadata and scripting workflows
- **wget, unzip, zip** - File utilities and archive packaging support
- **send-slack** - Built-in helper for posting messages to a Slack webhook

## Available Versions

| Tag | Terraform | AWS CLI | Node.js | Status |
|-----|-----------|---------|---------|--------|
| `v1.0` | 1.14.5 | 2.13.x | 24 | Latest stable |
| `v1.0-tf-1.14.5-aws-2.13.x` | 1.14.5 | 2.13.x | 24 | Latest stable |
| `v1.1-tf-1.15.0-aws-2.15.x` | 1.15.0 | 2.15.x | 24 | Upcoming |
| `latest` | 1.14.5 | 2.13.x | 24 | Points to v1.0 |

## Quick Start

### Architecture Support

- Published images include both `linux/amd64` and `linux/arm64` variants.
- On Intel/AMD Linux runners (for example GitHub-hosted Ubuntu), Docker pulls `linux/amd64` automatically.
- On Apple Silicon hosts (M1/M2), Docker pulls `linux/arm64` automatically.

### Use in Your Pipeline

**Bitbucket Pipelines:**
```yaml
image: ghcr.io/ujam-dev/pipeline-helper:v1.0

pipelines:
  branches:
    main:
      - step:
          name: Deploy
          script:
            - terraform --version
            - aws --version
            - jq --version
```

**GitHub Actions:**
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/ujam-dev/pipeline-helper:v1.0
    steps:
      - run: terraform --version
      - run: aws --version
```

### No Authentication Required

This image is public and can be pulled by any CI/CD pipeline without credentials.

## Repository Structure

```
pipeline-helper/
├── Dockerfile.v1.0           # Base image v1.0
├── Dockerfile.v2.0           # Base image v2.0 (future)
├── keys/
│   └── aws-cli-team-public.gpg.asc # AWS CLI signing key used for verification
├── .github/
│   └── workflows/
│       ├── smoke-test.yml    # CI smoke tests across amd64/arm64 and tool versions
│       ├── publish.yml       # GitHub Actions: build & publish
│       └── enforce-pinned-actions.yml # CI check: require SHA-pinned actions
├── README.md                 # This file
└── CHANGELOG.md              # Version history
```

## Supply-Chain Security

This repository includes multiple hardening controls to reduce supply-chain risk:

- **Pinned base image digest**: Dockerfiles use immutable `FROM ...@sha256:` references.
- **Terraform integrity verification**: downloaded Terraform archives are verified with published SHA256 checksums before installation.
- **AWS CLI authenticity verification**: AWS CLI archives are verified with AWS's published PGP signature and trusted fingerprint.
- **Pinned GitHub Actions**: third-party actions in workflows are pinned to full commit SHAs (not floating tags).
- **Policy enforcement in CI**: `.github/workflows/enforce-pinned-actions.yml` fails if a workflow introduces unpinned actions.
- **Build provenance and SBOM**: publish workflow emits provenance attestations and SBOM metadata.
- **Pre-publish smoke tests**: `.github/workflows/smoke-test.yml` builds both image variants on `linux/amd64` and `linux/arm64` and validates Node.js, Terraform, AWS CLI, and `jq` versions.

## Publishing a New Version

### Step 1: Update Dockerfile

Create a new Dockerfile for the version:
```bash
cp Dockerfile.v1.0 Dockerfile.v2.0
# Edit Dockerfile.v2.0 with new tool versions
```

### Step 2: Commit and Tag

```bash
git add Dockerfile.v2.0
git commit -m "chore: add base image v2.0 with Terraform 1.15.0"
git tag -a v2.0 -m "Release v2.0"
git push origin main
git push origin v2.0
```

### Step 3: GitHub Actions Publishes Automatically

The `.github/workflows/publish.yml` workflow will:
- Detect the tag
- Run smoke tests for `linux/amd64` and `linux/arm64` (Node.js, Terraform, AWS CLI, and `jq` version checks)
- Build the image
- Push to `ghcr.io/ujam-dev/pipeline-helper:v2.0`
- Push to `ghcr.io/ujam-dev/pipeline-helper:latest`

## Versioning Strategy

We follow **semantic versioning**: `vMAJOR.MINOR.PATCH`

- **MAJOR** - Breaking changes (e.g., Node.js major upgrade)
- **MINOR** - New tools added or minor version upgrades
- **PATCH** - Bug fixes, security patches

### Tag Naming
- `v1.0` - Full semantic version (main tag)
- `v1.0-tf-1.14.5-aws-2.13.x` - Specific tool versions for clarity and reproducibility
- `latest` - Always points to the most recent release

## Future Variants

As the base image matures, we may introduce specialized variants:

- **`v1.0-cloudflare-tf-1.14.5-aws-2.13.x`** - Adds Wrangler CLI for Cloudflare Workers deployments
- **`v1.0-full-tf-1.14.5-aws-2.13.x`** - Includes additional DevOps tools for extended use cases

These will be published as separate image tags to keep the main image lean and reduce pull sizes for projects that don't need them.

## Maintenance

### Security Updates

When vulnerabilities are discovered in included tools:

1. Update the Dockerfile with patched versions
2. Release as a PATCH version: `v1.0.1`
3. Update `latest` tag

### Adding New Tools

To add new tools to the image:

1. Update the appropriate `Dockerfile.vX.Y`
2. Test locally with: `docker build -f Dockerfile.vX.Y .`
3. Release as MINOR version: `v2.0`
4. Document in `CHANGELOG.md`

## Local Development

### Build Locally

```bash
docker build -f Dockerfile.v1.0 -t pipeline-helper:dev .
```

Force a specific architecture when needed:

```bash
# Build amd64 image (useful on Apple Silicon for compatibility testing)
docker buildx build --platform linux/amd64 -f Dockerfile.v1.0 -t pipeline-helper:dev-amd64 .

# Build arm64 image
docker buildx build --platform linux/arm64 -f Dockerfile.v1.0 -t pipeline-helper:dev-arm64 .
```

### Test the Image

```bash
docker run --rm pipeline-helper:dev terraform --version
docker run --rm pipeline-helper:dev aws --version
docker run --rm pipeline-helper:dev jq --version
```

## Sending Slack Notifications

The image ships `/usr/local/bin/send-slack` for posting pipeline notifications to Slack.

### Requirements

- Set `SLACK_WEBHOOK_URL` as a CI secret (Incoming Webhook URL from your Slack app).

### Usage

```bash
send-slack -message ":rocket: Deployment of my-service to prod was successful!"
```

### Bitbucket Pipelines example

```yaml
image: ghcr.io/ujam-dev/pipeline-helper:v1.0

pipelines:
  branches:
    main:
      - step:
          name: Deploy
          script:
            - # ... your deploy steps ...
            - |
              send-slack -message ":rocket: *${BITBUCKET_REPO_SLUG}* deployed to prod.
              Commit: <https://bitbucket.org/${BITBUCKET_WORKSPACE}/${BITBUCKET_REPO_SLUG}/commits/${BITBUCKET_COMMIT}|${BITBUCKET_COMMIT:0:7}>"
```

### GitHub Actions example

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/ujam-dev/pipeline-helper:v1.0
    env:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    steps:
      - name: Deploy
        run: | # ... your deploy steps ...
      - name: Notify Slack
        run: |
          send-slack -message ":rocket: *${{ github.repository }}* deployed to prod.
          Commit: <https://github.com/${{ github.repository }}/commit/${{ github.sha }}|${GITHUB_SHA:0:7}>"
```

## Troubleshooting

### Authentication Issues

This image is **public** - no authentication should be required. If you encounter auth errors:

- Ensure you're using the correct registry: `ghcr.io/ujam-dev/pipeline-helper`
- Verify the tag exists (check [GitHub Container Registry](https://github.com/orgs/ujam-dev/packages))
- Try pulling manually: `docker pull ghcr.io/ujam-dev/pipeline-helper:v1.0`

### Image Not Found

- Check that the version tag exists in GHCR
- Ensure tag follows the format: `v1.0`, `v2.0`, etc.
- Latest stable: use `latest` tag or specific version like `v1.0`

## License

MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

**THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.**

## Contacts

- **Maintainer:** [UJAM Music Technology GmbH](https://github.com/ujam-dev)/[Thomas Göllner](https://github.com/ujam-tgoellner)

## FAQ

**Q: Can I use this in production?**  
A: Yes. Pin to a specific version (e.g., `v1.0`) for stability.

**Q: Does this work with other CI/CD systems?**  
A: Yes - any system that supports Docker images from GHCR (GitHub Actions, GitLab CI, Jenkins, etc.)
