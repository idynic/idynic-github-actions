# Create Issue Branch Action

A GitHub composite action that creates and pushes new branches for issues when work begins, with built-in conflict handling and timestamp-based naming.

## Features

- 🌿 **Automated Branch Creation**: Creates branches with consistent naming patterns
- 🔧 **Git Configuration**: Automatically configures git with bot credentials
- ⏰ **Timestamp Support**: Optional timestamp suffix to avoid naming conflicts
- 🔁 **Existing Branch Handling**: Handles scenarios where branches already exist
- 💬 **Issue Comments**: Automatically adds informative comments to issues
- 🎨 **Customizable Prefixes**: Support for custom branch prefixes

## Usage

```yaml
- name: Create issue branch
  uses: idynic/idynic-github-actions/create-issue-branch@v1
  with:
    issue-number: ${{ github.event.issue.number }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    branch-prefix: 'issue'  # Optional, defaults to 'issue'
    timestamp: 'true'      # Optional, defaults to 'true'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `issue-number` | The issue number for which to create a branch | Yes | - |
| `github-token` | GitHub token with repo permissions | Yes | - |
| `repository` | Repository in owner/name format | Yes | - |
| `branch-prefix` | Prefix for the branch name | No | `issue` |
| `timestamp` | Whether to append timestamp to branch name | No | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `branch-name` | The name of the created branch |

## Branch Naming

Branches are created with the following format:
- With timestamp (default): `<prefix>-<issue-number>-<timestamp>`
- Without timestamp: `<prefix>-<issue-number>`

Examples:
- `issue-123-1703123456`
- `feature-456`
- `bugfix-789-1703123457`

## Example Workflows

### Basic Usage

```yaml
name: Create Branch on Issue Command
on:
  issue_comment:
    types: [created]

jobs:
  create-branch:
    if: github.event.comment.body == '/start'
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Create issue branch
        uses: idynic/idynic-github-actions/create-issue-branch@v1
        with:
          issue-number: ${{ github.event.issue.number }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          repository: ${{ github.repository }}
```

### With Custom Prefix

```yaml
- name: Create feature branch
  uses: idynic/idynic-github-actions/create-issue-branch@v1
  with:
    issue-number: ${{ github.event.issue.number }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    branch-prefix: 'feature'
```

### Without Timestamp

```yaml
- name: Create stable branch name
  uses: idynic/idynic-github-actions/create-issue-branch@v1
  with:
    issue-number: ${{ github.event.issue.number }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    timestamp: 'false'
```

## Error Handling

The action handles several error scenarios:
- Missing required inputs
- Existing branches (when timestamp is disabled)
- Git operation failures
- Network connectivity issues

## Git Configuration

The action automatically configures git with GitHub Actions bot credentials:
- Name: `github-actions[bot]`
- Email: `41898282+github-actions[bot]@users.noreply.github.com`

## Security Considerations

- Always use a GitHub token with appropriate permissions
- The action requires push access to create branches
- Be cautious when using custom branch prefixes to avoid conflicts
- Consider using branch protection rules on your main branch

## Contributing

Contributions are welcome! Please see the [main repository's CONTRIBUTING.md](../CONTRIBUTING.md) for guidelines.

## License

This action is licensed under the same terms as the main repository. See [LICENSE](../LICENSE) for details.
