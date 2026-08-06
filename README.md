# Pipeline Helper Docker Image

A pre-built Docker image containing essential DevOps tools for CI/CD pipelines. Designed for use across multiple organizations and projects.

**Publicly available at:** `ghcr.io/ujam-dev/pipeline-helper`

## What's Included

- **Node.js 24** - JavaScript runtime
- **Terraform** - Infrastructure as Code
- **AWS CLI** - Amazon Web Services command-line tools
- **curl** - Data transfer tool
- **jq** - JSON query processor
- **wget, unzip** - File utilities

## Available Versions

| Tag | Terraform | AWS CLI | Node.js | Status |
|-----|-----------|---------|---------|--------|
| `v1.0` | 1.14.5 | 2.13.x | 24 | Latest stable |
| `v1.0-tf-1.14.5-aws-2.13.x` | 1.14.5 | 2.13.x | 24 | Latest stable |
| `v1.1-tf-1.15.0-aws-2.15.x` | 1.15.0 | 2.15.x | 24 | Upcoming |
| `latest` | 1.14.5 | 2.13.x | 24 | Points to v1.0 |

## Quick Start

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
├── .github/
│   └── workflows/
│       └── publish.yml       # GitHub Actions: build & publish
├── README.md                 # This file
└── CHANGELOG.md              # Version history
```

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

### Test the Image

```bash
docker run --rm pipeline-helper:dev terraform --version
docker run --rm pipeline-helper:dev aws --version
docker run --rm pipeline-helper:dev jq --version
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
