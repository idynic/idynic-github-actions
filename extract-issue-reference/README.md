# Extract Issue Reference

This action extracts issue references from pull requests following GitHub best practices.

## Features

- Extracts issue numbers from PR title, body, and branch name
- Handles special characters and quotes in PR bodies
- Uses a robust pattern matching approach to find various issue reference formats
- Determines whether issue status should be updated based on PR event type
- Follows GitHub best practices for issue tracking

## Usage

This action is designed to be used in conjunction with the `update-issue-status` action to manage issues linked to pull requests.

```yaml
name: PR Issue Status Update

on:
  pull_request:
    types: [opened, synchronize, ready_for_review, closed]

permissions:
  issues: write
  pull-requests: read

jobs:
  update-linked-issue:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout GitHub Actions
        uses: actions/checkout@v4
        with:
          repository: idynic/idynic-github-actions
          path: .github/actions
          token: ${{ secrets.PROJECT_TOKEN }}

      # Extract issue number from PR and determine status update strategy
      - name: Extract issue reference
        id: extract-issue
        uses: ./.github/actions/extract-issue-reference
        with:
          pr-title: ${{ github.event.pull_request.title }}
          pr-body: ${{ github.event.pull_request.body }}
          pr-branch: ${{ github.event.pull_request.head.ref }}
          pr-action: ${{ github.event.action }}
          check-update-status-tag: "true"

      # Update issue status if needed
      - name: Update issue status
        if: steps.extract-issue.outputs.should-update-status == 'true'
        uses: ./.github/actions/update-issue-status
        with:
          issue-number: ${{ steps.extract-issue.outputs.issue-number }}
          status: ${{ steps.extract-issue.outputs.update-status }}
          token: ${{ secrets.PROJECT_TOKEN }}
          organization: ${{ github.repository_owner }}

      # Add appropriate comment to issue
      - name: Add PR opened comment
        if: steps.extract-issue.outputs.should-comment == 'true' && steps.extract-issue.outputs.comment-type == 'opened'
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: ${{ steps.extract-issue.outputs.issue-number }},
              body: `📋 Pull Request #${{ github.event.pull_request.number }} opened for this issue: ${{ github.event.pull_request.html_url }}\n\n✅ Issue status updated to: **Waiting for Review**`
            });

      - name: Add review ready comment
        if: steps.extract-issue.outputs.should-comment == 'true' && steps.extract-issue.outputs.comment-type == 'ready'
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: ${{ steps.extract-issue.outputs.issue-number }},
              body: `📋 Pull Request #${{ github.event.pull_request.number }} is ready for review: ${{ github.event.pull_request.html_url }}\n\n✅ Issue status updated to: **Waiting for Review**`
            });
            
      - name: Add status update comment
        if: steps.extract-issue.outputs.should-comment == 'true' && steps.extract-issue.outputs.comment-type == 'sync'
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: ${{ steps.extract-issue.outputs.issue-number }},
              body: `🔄 PR #${{ github.event.pull_request.number }} updated with new commits: ${{ github.event.pull_request.html_url }}\n\n✅ Issue status updated to: **Waiting for Review**`
            });

      - name: Add merged comment
        if: steps.extract-issue.outputs.should-comment == 'true' && steps.extract-issue.outputs.comment-type == 'merged'
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: ${{ steps.extract-issue.outputs.issue-number }},
              body: `🎉 Pull Request #${{ github.event.pull_request.number }} has been merged!\n\n✅ Issue status updated to: **Done**`
            });
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `pr-title` | The PR title to extract issue numbers from | Yes | N/A |
| `pr-body` | The PR body to extract issue numbers from | Yes | N/A |
| `pr-branch` | The branch name to extract issue numbers from | Yes | N/A |
| `pr-action` | The PR action (opened, synchronize, ready_for_review, closed) | Yes | N/A |
| `check-update-status-tag` | Whether to check for [update-status] tag in PR title for synchronize events | No | true |

## Outputs

| Name | Description |
|------|-------------|
| `issue-number` | The extracted issue number (if found) |
| `should-update-status` | Whether the issue status should be updated based on PR event type |
| `update-status` | The status to update the issue to (if should-update-status is true) |
| `should-comment` | Whether a comment should be added to the issue |
| `comment-type` | The type of comment to add (opened, ready, sync, merged) |

## How It Works

This action follows GitHub best practices for issue tracking:

1. **Issue Extraction**: Searches for issue references in PR title, body, and branch name.
   - Identifies various formats like: #123, fix #123, fixes #123, closes #123, etc.
   - Handles branch names in the format: issue-123 or issue-123-description

2. **Status Update Strategy**:
   - PR opened → Issue moves to "Waiting for Review"
   - PR ready for review → Issue moves to "Waiting for Review"
   - PR synchronized (pushed commits) → Only update if "[update-status]" is in PR title
   - PR merged → Issue moves to "Done"
   - PR closed without merging → No status update

3. **Comment Strategy**:
   - Adds comments only on significant state transitions
   - Avoids comment spam on regular code pushes
   - Provides opt-in mechanism for updates on code pushes

## Special Note

To request a status update on a code push (synchronize event), add `[update-status]` to your PR title before pushing. This allows for status updates when explicitly desired without spamming on every minor push.