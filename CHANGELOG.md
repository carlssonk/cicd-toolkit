# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial release of GitHub Workflows Library
- Deploy to S3 workflow with versioning support
- Rollback S3 workflow with automatic and manual modes
- Create Release workflow with changelog generation
- Deployment Router workflow supporting multiple branching strategies
- Notification workflow for Slack, Discord, Teams, and GitHub Discussions
- Comprehensive documentation for all workflows
- Example workflows for common use cases
- Quick start guide
- Contributing guidelines

### Features

#### Deploy to S3 (`deploy-s3.yml`)
- Versioned deployments with commit hashes
- Dual-path strategy (versioned + main)
- Metadata tracking for audit trail
- Support for npm, yarn, and pnpm
- Concurrency control
- OIDC authentication
- Comprehensive outputs

#### Rollback S3 (`rollback-s3.yml`)
- Automatic rollback to previous version
- Manual rollback to specific commit
- Validation before rollback
- Metadata updates with rollback tracking
- Safety checks and error handling

#### Create Release (`create-release.yml`)
- Automatic git tagging
- GitHub release creation
- Changelog generation from conventional commits
- Customizable tag prefixes
- Pre-release support
- Idempotent operation

#### Deployment Router (`deployment-router-trunk.yml`)
- Trunk-based development strategy
- Terraform Action determination

#### Notify (`notify.yml`)
- Slack notifications
- Discord notifications
- Microsoft Teams notifications
- GitHub Discussions
- Status-based formatting
- Rich context inclusion
- Mention on failure support

## [1.0.0] - YYYY-MM-DD

### Added
- Initial stable release

---

## Version History

- **v1.0.0** - Initial stable release
- **v0.1.0** - Initial development version

## Migration Guides

### Upgrading to v1.0.0

No breaking changes - this is the initial release.

## Support

For questions and support:
- 📖 [Documentation](./README.md)
- 💬 [GitHub Discussions](https://github.com/carlssonk/cicd-toolkit/discussions)
- 🐛 [Issue Tracker](https://github.com/carlssonk/cicd-toolkit/issues)

