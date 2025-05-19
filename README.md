# Atriumn GitHub Actions

A comprehensive collection of reusable GitHub Actions composite actions for Atriumn projects.

## Overview

This repository provides standardized, reusable composite actions that help automate common workflows across Atriumn projects. These actions promote consistency, reduce duplication, and simplify CI/CD pipeline implementation.

## Action Catalog

### Issue Management Actions

#### [handle-issue-commands](./handle-issue-commands/)
Parses and handles slash commands in issue comments with authorization checking and command validation.
```yaml
- uses: idynic/idynic-github-actions/handle-issue-commands@main
  with:
    command: 'start'
    issue-number: ${{ github.event.issue.number }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    comment-user: ${{ github.event.comment.user.login }}
```

#### [create-issue-branch](./create-issue-branch/)
Creates and pushes new branches for issues with timestamp-based naming and conflict handling.
```yaml
- uses: idynic/idynic-github-actions/create-issue-branch@main
  with:
    issue-number: ${{ github.event.issue.number }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    branch-prefix: 'issue'  # Optional
    timestamp: 'true'      # Optional
```

#### [update-issue-status](./update-issue-status/)
Updates issue status in GitHub Projects V2 boards using GraphQL API.
```yaml
- uses: idynic/idynic-github-actions/update-issue-status@main
  with:
    issue-number: ${{ github.event.issue.number }}
    status: 'In Progress'
    token: ${{ secrets.PROJECT_TOKEN }}
    organization: 'idynic'
```

### Development Environment Actions

#### [setup-node](./.github/actions/setup-node/)
Sets up Node.js environment with caching.
```yaml
- uses: idynic/idynic-github-actions/.github/actions/setup-node@main
  with:
    node-version: '20'
```

## Quick Start

### Using an Action

1. Add the action to your workflow file:
```yaml
name: My Workflow
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: idynic/idynic-github-actions/setup-node@main
        with:
          node-version: '20'
```

2. Configure inputs as needed (see individual action documentation)

### Best Practices

- Always use a specific version tag instead of `main` for production workflows
- Review action documentation for required inputs and outputs
- Consider using action outputs for workflow orchestration

## Repository Structure

```
idynic-github-actions/
├── .github/
│   ├── actions/        # Standard GitHub Actions location
│   ├── workflows/      # Test and validation workflows
│   └── settings.yml    # Repository configuration
├── create-issue-branch/   # Issue branch creation action
├── handle-issue-commands/ # Issue command processing action
├── update-issue-status/   # Issue status management action
├── docs/                 # Additional documentation
└── README.md            # This file
```

## Documentation

- [Contributing Guidelines](./CONTRIBUTING.md)
- [Development Setup](./docs/development.md)
- [Repository Setup](./docs/SETUP.md)
- [Repository Settings](./docs/repository-settings.md)
- [Versioning Strategy](./VERSIONING.md)
- [Changelog](./CHANGELOG.md)

## Support

- **Issues**: [Report bugs or request features](https://github.com/idynic/idynic-github-actions/issues)
- **Discussions**: [Ask questions or share ideas](https://github.com/idynic/idynic-github-actions/discussions)

## Topics

- `github-actions`
- `composite-actions`
- `automation`
- `ci-cd`
- `devops`

## License

See [LICENSE](./LICENSE) file for details.

---

*Maintained by the Atriumn team*