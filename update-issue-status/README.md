# Update Issue Status Action

This composite action updates the status of GitHub issues in project boards using the GraphQL API.

## Prerequisites

- GitHub token with `project` scope
- Issue must be added to a GitHub Projects V2 board
- Project must have a "Status" field configured

## Usage

```yaml
- uses: idynic/idynic-github-actions/update-issue-status@v1
  with:
    issue-number: ${{ github.event.issue.number }}
    status: 'In Progress'
    token: ${{ secrets.PROJECT_TOKEN }}
    organization: 'idynic'
    project-name: 'My Project'
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `issue-number` | The issue number to update | Yes | - |
| `status` | The new status to set (e.g., 'Todo', 'In Progress', 'Done') | Yes | - |
| `token` | GitHub token with project permissions | Yes | - |
| `organization` | Organization name | No | `idynic` |
| `project-name` | Project name to search for | No | `{repo} project` |
| `project-number` | (Deprecated) Project number in the organization | No | - |

## Permissions Required

The GitHub token must have:
- `project` scope for organization projects
- `repo` scope for repository access
- `read:org` scope to query organization data

## Examples

### Basic Usage

```yaml
name: Update Issue Status
on:
  issues:
    types: [opened, reopened, closed]

jobs:
  update-status:
    runs-on: ubuntu-latest
    steps:
      - name: Update status
        uses: idynic/idynic-github-actions/update-issue-status@v1
        with:
          issue-number: ${{ github.event.issue.number }}
          status: ${{ github.event.action == 'closed' && 'Done' || 'In Progress' }}
          token: ${{ secrets.PROJECT_TOKEN }}
```

### Using with Pull Requests

```yaml
name: Update Issue Status from PR
on:
  pull_request:
    types: [opened, ready_for_review, closed]

jobs:
  update-linked-issue:
    runs-on: ubuntu-latest
    steps:
      - name: Extract issue number
        id: extract
        run: |
          # Extract issue number from PR title/body
          ISSUE_NUMBER=$(echo "${{ github.event.pull_request.title }}" | grep -o '#[0-9]\+' | head -1 | tr -d '#')
          echo "issue_number=$ISSUE_NUMBER" >> $GITHUB_OUTPUT
      
      - name: Update issue status
        if: steps.extract.outputs.issue_number
        uses: idynic/idynic-github-actions/update-issue-status@v1
        with:
          issue-number: ${{ steps.extract.outputs.issue_number }}
          status: ${{ github.event.action == 'closed' && 'Done' || 'In Review' }}
          token: ${{ secrets.PROJECT_TOKEN }}
```

### Custom Project Name

```yaml
- uses: idynic/idynic-github-actions/update-issue-status@v1
  with:
    issue-number: 123
    status: 'In Progress'
    token: ${{ secrets.PROJECT_TOKEN }}
    organization: 'myorg'
    project-name: 'Development Board'
```

## Status Values

Common status values (must match exactly with your project's status field options):
- `Todo`
- `In Progress`
- `In Review`
- `Done`
- `Blocked`

## Error Handling

The action will fail with detailed error messages if:
- Required inputs are missing
- Project is not found
- Issue is not in the specified project
- Status value doesn't match project options
- Token lacks required permissions

## Troubleshooting

### Common Issues

1. **"Project not found" error**
   - Verify the project name matches exactly
   - Check organization name is correct
   - Ensure project is accessible with the token

2. **"Status not found" error**
   - Status must match exactly (case-sensitive)
   - Check available statuses in your project settings
   - Action will list available statuses in error message

3. **"Issue not found in project" error**
   - Ensure issue is added to the project board
   - Check issue number is correct
   - Verify token has access to both repo and project

4. **Permission denied errors**
   - Token needs `project` scope
   - For org projects, token needs org-level access
   - Personal access tokens work better than GITHUB_TOKEN

### Debug Mode

Enable debug logging:
```yaml
- uses: idynic/idynic-github-actions/update-issue-status@v1
  with:
    issue-number: 123
    status: 'In Progress'
    token: ${{ secrets.PROJECT_TOKEN }}
  env:
    ACTIONS_STEP_DEBUG: true
```

## Limitations

- Only works with GitHub Projects V2 (not classic projects)
- Requires issues to be already added to the project
- Status field must be a single-select field
- Project name search is case-sensitive

## Contributing

See [Contributing Guide](../CONTRIBUTING.md) for development setup and guidelines.

## License

MIT License - see [LICENSE](../LICENSE) file for details.