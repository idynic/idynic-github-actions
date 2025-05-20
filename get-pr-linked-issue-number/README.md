# Get PR Linked Issue Number

This action extracts the first linked issue number from a Pull Request's title, body, or branch name. It's designed to identify the issue that a PR is addressing by looking for common linking patterns.

## Purpose

When working with Pull Requests in GitHub, it's common to reference the issue being addressed using phrases like "Fixes #123" or via branch naming conventions. This action simplifies the process of identifying these references, making it easier to automate workflows that need to know which issue a PR is addressing.

## Inputs

| Name | Description | Required |
|------|-------------|----------|
| `pr_title` | The Pull Request title. | Yes |
| `pr_body` | The Pull Request body content. | Yes |
| `pr_branch` | The name of the PR's head branch. | Yes |

## Outputs

| Name | Description |
|------|-------------|
| `issue_number` | The extracted issue number, if found. Empty otherwise. |

## Extraction Logic

The action searches for an issue number in the following order of priority:

1. **PR Title & Body**: 
   - Looks for common linking phrases: `Closes #123`, `Fixes #123`, `Resolves #123` (case insensitive)
   - If not found, searches for any `#123` reference

2. **Branch Name**:
   - Checks for patterns like `issue/123`, `feature/123`, `fix/123`
   - As a fallback, looks for branches starting with a number, e.g., `123-fix-bug`

The action returns the first issue number found following this priority order. If multiple issues are referenced, only the first one is returned.

## Example Usage

```yaml
name: PR Processing
on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  extract-issue:
    runs-on: ubuntu-latest
    steps:
      - name: Get linked issue
        id: get_issue
        uses: idynic/idynic-github-actions/get-pr-linked-issue-number@main
        with:
          pr_title: ${{ github.event.pull_request.title }}
          pr_body: ${{ github.event.pull_request.body }}
          pr_branch: ${{ github.head_ref }}
          
      - name: Use issue number
        run: |
          if [[ -n "${{ steps.get_issue.outputs.issue_number }}" ]]; then
            echo "This PR is linked to issue #${{ steps.get_issue.outputs.issue_number }}"
          else
            echo "No linked issue found"
          fi
```

## Notes

- The action only returns the first issue number it finds, based on the priority order.
- If no issue number is found, the `issue_number` output will be empty.
- The pattern matching supports the most common reference formats, but custom formats may not be detected.