# Using the Idynic GitHub Actions

This repository contains reusable GitHub Actions and workflow templates to standardize issue management across Idynic repositories.

## Quick Setup

To add these workflows to a repository:

1. Create the workflow files in your repository:

```bash
# Create .github/workflows directory if it doesn't exist
mkdir -p .github/workflows

# Copy workflow templates
curl -o .github/workflows/issue-comment-commands.yml https://raw.githubusercontent.com/idynic/idynic-github-actions/main/issue-comment-commands-template.yml

curl -o .github/workflows/pr-issue-status-update.yml https://raw.githubusercontent.com/idynic/idynic-github-actions/main/pr-issue-status-update-template.yml
```

2. Add the PROJECT_TOKEN secret to your repository:

```bash
# Set PROJECT_TOKEN for your repository
export PROJECT_TOKEN="your-github-token"
gh secret set PROJECT_TOKEN -R idynic/your-repo
```

## Workflow Features

### Issue Comment Commands (`issue-comment-commands.yml`)

- Handles the `/start` command on issues
- Creates a branch for the issue (format: `issue-{number}-{timestamp}`)
- Updates issue status to "In Progress"

#### Usage

Comment `/start` on any issue to:
1. Create a new branch for the issue
2. Update the issue status

### PR Issue Status Update (`pr-issue-status-update.yml`)

- Updates linked issues when PRs are opened, made ready for review, or merged
- Extracts issue number from PR title/body (format: `Fix #123`, `Closes #456`, etc.)
- Updates issue status based on PR state:
  - PR opened or ready: "Waiting for Review" 
  - PR merged: "Done"

#### Usage

1. Create PRs with titles or descriptions that reference issues:
   - "Fix #123: Add new feature"
   - "Implement XYZ (closes #456)"

2. The workflow will automatically:
   - Link the PR to the referenced issue
   - Update the issue status
   - Add comments to the issue

## Actions

The workflows use the following reusable actions:

1. `extract-issue-reference`: Extracts issue numbers from PR title, body, or branch name
2. `handle-issue-commands`: Processes slash commands in issue comments
3. `create-issue-branch`: Creates branches for issues
4. `update-issue-status`: Updates issue status via comments

## Contributing

To make changes to these workflows, please create a PR with your changes.