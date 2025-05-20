# Update Project V2 Item Status

This action sets the status of an item (linked to an issue or pull request) on a GitHub Project V2 board. It handles all the necessary GraphQL calls to find the project, locate the item on the project board (or add it if not present), identify the status field and options, and perform the update.

## Purpose

GitHub Project V2 boards use a GraphQL API that requires multiple sequential calls to:
1. Find the project by owner and number
2. Identify the status field and target option values
3. Check if the item is already on the project board
4. Add the item to the project if needed
5. Update the item's status field

This action encapsulates this complex workflow into a simple, reusable component.

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `github_token` | A Personal Access Token (PAT) with `repo`, `read:org`, and `project` scopes. **Note: GITHUB_TOKEN will NOT work for organization projects.** | Yes | |
| `owner_login` | The login of the organization or user that owns the GitHub Project (e.g., 'idynic'). | Yes | |
| `project_number` | The number of the Project V2 (e.g., 5 if the URL is /orgs/owner/projects/5). | Yes | |
| `content_node_id` | The GraphQL Node ID of the content (Issue or Pull Request) whose status is to be updated on the project board. | Yes | |
| `target_status_value` | The exact string value of the desired status (e.g., 'In Progress', 'Done'). This MUST match an existing option in your project's status field. | Yes | |
| `status_field_name` | The name of the custom 'Status' field in your Project V2. This is case-sensitive. | No | `Status` |

## Outputs

| Name | Description |
|------|-------------|
| `project_item_id` | The GraphQL Node ID of the project item that was created or updated. |

## Prerequisites

- The `gh` CLI is installed in the GitHub Actions runner environment (pre-installed on GitHub-hosted runners).
- `jq` is installed by this action if not already present.

## Critical Requirements

### Token Requirements

The `github_token` input can be one of the following:
- **GITHUB_TOKEN**: The default token provided by GitHub Actions, with `project: write` permission added in your workflow file
- **PAT**: A Personal Access Token with the following scopes:
  - `repo` - For repository access
  - `read:org` - For organization access (finding projects)
  - `project` - For project board modifications

Example workflow permission configuration:
```yaml
permissions:
  issues: write       # Required for commenting on issues
  pull-requests: read # Required for reading PR details (if needed)
  project: write      # Required for project board access
```

> **Important**: If using the default `GITHUB_TOKEN`, you must add `project: write` permission in your workflow file as shown above.

### Configuration Values

The following values must **exactly** match your GitHub Project V2 configuration (case-sensitive):

- `owner_login`: The organization or user name that owns the project 
  - For organization projects: The organization name (e.g., 'idynic')
  - For user projects: The username (e.g., 'octocat')

- `project_number`: The number in your project URL
  - For organization projects: If URL is `https://github.com/orgs/idynic/projects/5`, use `5`
  - For user projects: If URL is `https://github.com/users/octocat/projects/1`, use `1`

- `status_field_name`: The exact name of your status field (default is `Status`)
  - This is case-sensitive and must match the field name in your Project V2 settings

- `target_status_value`: The exact option value you want to set
  - This is case-sensitive and must match one of the options in your status field

### Content Node ID

The `content_node_id` must be the GraphQL Node ID of the issue or PR, not the issue or PR number. It typically looks like:
- For issues: `MDU6SXNzdWUxMjM0NTY3ODk=`
- For PRs: `MDExOlB1bGxSZXF1ZXN0MTIzNDU2Nzg5`

You can obtain this using the `get-pr-linked-issue-number` action or by querying GitHub's GraphQL API.

## Example Usage

```yaml
- name: Update Project Status
  uses: idynic/idynic-github-actions/update-project-v2-item-status@main
  with:
    github_token: ${{ secrets.PROJECT_TOKEN }}
    owner_login: idynic
    project_number: 3
    content_node_id: ${{ steps.extract_ref.outputs.node_id }}
    target_status_value: "In Progress"
```

## Error Handling

This action includes comprehensive error checking at each step of the GraphQL workflow. It will:

- Verify the project exists and is accessible with your PAT
- Check that the status field exists and is a single select field
- Verify the target status option is valid
- Ensure the content item can be found or added to the project
- Validate that the status update was successful

If any step fails, the action will exit with a detailed error message to help troubleshoot the issue, including:

- Available fields when a status field is not found
- Available options when a status option is not found
- GraphQL errors when API calls fail
- Permission issues with your PAT

## Troubleshooting Common Issues

1. **"Error: Project V2 not found"**
   - Verify the `project_number` and `owner_login` are correct
   - Ensure your PAT has the `read:org` scope for organization projects

2. **"Error: Status field not found"**
   - Check that `status_field_name` matches exactly (case-sensitive)
   - Look at the list of available fields in the error message

3. **"Error: Status option not found"**
   - Verify `target_status_value` matches exactly (case-sensitive)
   - Check the list of available options in the error message

4. **GraphQL Permission Errors**
   - Ensure your PAT has all required scopes: `repo`, `read:org`, and `project`
   - For organization projects, you may need to ensure the PAT owner has adequate permissions in the organization