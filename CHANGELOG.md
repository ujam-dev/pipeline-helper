# Changelog

All notable changes to this project will be documented in this file.

The format is based on Keep a Changelog,
and this project adheres to Semantic Versioning.

## [1.0.2] - 2026-09-24

### Added
- Added the Docker CLI for connecting to a mounted host Docker socket.

## [1.0.1] - 2026-08-06

### Added
- Added `send-slack` helper script at `/usr/local/bin/send-slack` for posting Slack notifications from pipeline steps.

## [1.0.0] - 2026-08-06

### Added
- Initial release of pipeline-helper container image.
- Node.js 24 base runtime.
- Terraform 1.14.5.
- AWS CLI 2.13.x line.
- Common utilities: curl, git, jq, unzip, wget, zip.
- GitHub Actions workflow for publishing tagged releases to GHCR.
