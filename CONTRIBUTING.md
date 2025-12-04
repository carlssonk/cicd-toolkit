# Contributing to GitHub Workflows Library

Thank you for your interest in contributing! This document provides guidelines and instructions for contributing to this project.

## 🎯 How to Contribute

### Reporting Issues

1. Check if the issue already exists
2. Use the issue template
3. Provide clear reproduction steps
4. Include relevant logs and configuration

### Suggesting Features

1. Open a discussion first to gauge interest
2. Describe the use case clearly
3. Explain why existing workflows don't meet the need
4. Consider backward compatibility

### Submitting Pull Requests

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Test thoroughly
5. Update documentation
6. Commit with clear messages
7. Push to your fork
8. Open a pull request

## 📝 Development Guidelines

### Workflow Design Principles

1. **Reusability**: Workflows should be generic and configurable
2. **Security**: Use OIDC, minimal permissions, no secrets in logs
3. **Reliability**: Include error handling, validation, and timeouts
4. **Observability**: Provide clear outputs, summaries, and logs
5. **Documentation**: Every input/output must be documented

### Code Standards

#### Workflow Structure

```yaml
name: Descriptive Name (Reusable)

on:
  workflow_call:
    inputs:
      # Required inputs first
      required_input:
        description: "Clear description"
        required: true
        type: string
      # Optional inputs after
      optional_input:
        description: "Clear description"
        required: false
        type: string
        default: "sensible-default"
    outputs:
      output_name:
        description: "What this output contains"
        value: ${{ jobs.job-name.outputs.output_name }}

jobs:
  job-name:
    name: Descriptive Job Name
    runs-on: ubuntu-latest
    timeout-minutes: 30
    concurrency:
      group: unique-group-${{ inputs.environment }}
      cancel-in-progress: false
    permissions:
      # Minimal required permissions
    outputs:
      output_name: ${{ steps.step-id.outputs.value }}
    
    steps:
      - name: Descriptive Step Name
        run: |
          # Clear, commented code
```

#### Naming Conventions

- **Workflows**: `kebab-case.yml` (e.g., `deploy-s3.yml`)
- **Jobs**: `kebab-case` (e.g., `deploy-production`)
- **Steps**: Descriptive sentences (e.g., `Configure AWS Credentials`)
- **Inputs/Outputs**: `snake_case` (e.g., `aws_role_arn`)

#### Error Handling

Always validate inputs and provide clear error messages:

```yaml
- name: Validate inputs
  run: |
    if [ -z "${{ inputs.required_input }}" ]; then
      echo "❌ Error: required_input is required"
      exit 1
    fi
```

#### Security

1. Never log secrets or sensitive data
2. Use OIDC for cloud authentication
3. Request minimal permissions
4. Validate all inputs
5. Use concurrency controls

### Documentation Standards

#### Workflow Documentation

Each workflow must have documentation in `docs/` with:

1. **Overview**: What the workflow does
2. **Features**: Key capabilities
3. **Usage**: Basic and advanced examples
4. **Inputs**: All inputs with descriptions and defaults
5. **Outputs**: All outputs with descriptions
6. **Prerequisites**: Required setup
7. **Examples**: Real-world usage examples
8. **Troubleshooting**: Common issues and solutions
9. **Related Workflows**: Links to related workflows

#### Code Comments

- Comment complex logic
- Explain non-obvious decisions
- Document workarounds with context

### Testing

Before submitting:

1. **Test Locally**: Use `act` or similar tools
2. **Test in Fork**: Run workflows in your fork
3. **Test All Scenarios**: Success, failure, edge cases
4. **Test Documentation**: Verify examples work

### Commit Messages

Follow conventional commits:

```
feat: add support for custom cache control headers
fix: resolve issue with Cloudflare invalidation
docs: update deployment router examples
refactor: simplify metadata tracking logic
test: add tests for rollback validation
chore: update dependencies
```

## 🔄 Pull Request Process

### Before Submitting

- [ ] Code follows style guidelines
- [ ] All tests pass
- [ ] Documentation is updated
- [ ] Examples are provided
- [ ] Changelog is updated (if applicable)
- [ ] No breaking changes (or clearly documented)

### PR Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
Describe how you tested the changes

## Checklist
- [ ] Code follows style guidelines
- [ ] Documentation updated
- [ ] Examples provided
- [ ] Tested thoroughly
```

### Review Process

1. Automated checks must pass
2. At least one maintainer approval required
3. Address all review comments
4. Squash commits before merge (if requested)

## 🏷️ Versioning

We use [Semantic Versioning](https://semver.org/):

- **MAJOR**: Breaking changes
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes (backward compatible)

## 📋 Workflow Checklist

When adding a new workflow:

- [ ] Workflow file in `.github/workflows/`
- [ ] Documentation in `docs/`
- [ ] Example usage in `docs/examples/`
- [ ] Entry in main README
- [ ] Tests (if applicable)
- [ ] Security review
- [ ] Performance considerations
- [ ] Backward compatibility check

## 🎨 Documentation Style

### Markdown

- Use ATX-style headers (`#` not `===`)
- Use fenced code blocks with language tags
- Use tables for structured data
- Use emojis sparingly for visual hierarchy
- Include examples for every feature

### Code Examples

- Provide complete, runnable examples
- Include comments for clarity
- Show both basic and advanced usage
- Use realistic variable names

## 🤝 Community

### Code of Conduct

- Be respectful and inclusive
- Welcome newcomers
- Provide constructive feedback
- Focus on what's best for the community

### Getting Help

- 📖 Read the documentation first
- 💬 Use GitHub Discussions for questions
- 🐛 Use Issues for bugs
- 💡 Use Discussions for feature ideas

## 🙏 Recognition

Contributors will be recognized in:
- README contributors section
- Release notes
- GitHub contributors page

## 📞 Contact

- GitHub Discussions: For questions and discussions
- GitHub Issues: For bugs and feature requests
- Email: [maintainer-email] (for security issues only)

---

Thank you for contributing to make this project better! 🎉

