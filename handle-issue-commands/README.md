# Handle Issue Commands Action

A GitHub composite action that parses and handles slash commands in issue comments, managing command authorization and execution.

## Features

- 🔐 **Authorization Checking**: Verifies users have write permissions before executing commands
- ✅ **Command Validation**: Validates commands and provides helpful error messages
- 🚀 **Extensible Command Handling**: Supports multiple commands with easy extension
- 📝 **Automatic Acknowledgment**: Adds reactions and comments to acknowledge valid commands
- 🛡️ **Security First**: Prevents unauthorized users from executing commands

## Usage

```yaml
- name: Handle issue command
  uses: idynic/idynic-github-actions/handle-issue-commands@v1
  with:
    command: 'start'
    issue-number: ${{ github.event.issue.number }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    comment-user: ${{ github.event.comment.user.login }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `command` | The command to handle (e.g., start, done, review, close) | Yes | - |
| `issue-number` | The issue number where the command was posted | Yes | - |
| `github-token` | GitHub token with repo and issue permissions | Yes | - |
| `repository` | Repository in owner/name format | Yes | - |
| `comment-user` | Username who posted the command | Yes | - |

## Outputs

| Output | Description |
|--------|-------------|
| `authorized` | Whether the user is authorized to run the command (true/false) |
| `processed` | Whether the command was processed successfully (true/false) |

## Supported Commands

| Command | Description | Action |
|---------|-------------|--------|
| `/start` | Start working on an issue | Assigns issue to the commenter |
| `/done` | Mark issue as complete | Adds completion comment |
| `/review` | Request review | Adds review request comment |
| `/close` | Close the issue | Closes issue with comment |

## Authorization

This action checks if the commenting user has write permissions to the repository. Users without write permissions will receive an error message and the command will not be executed.

## Example Workflow

Here's a complete example of using this action in a workflow:

```yaml
name: Handle Issue Commands
on:
  issue_comment:
    types: [created]

jobs:
  handle-command:
    if: startsWith(github.event.comment.body, '/')
    runs-on: ubuntu-latest
    
    steps:
      - name: Extract command
        id: extract
        run: |
          COMMAND=$(echo "${{ github.event.comment.body }}" | awk '{print $1}' | tr -d '/')
          echo "command=$COMMAND" >> $GITHUB_OUTPUT

      - name: Handle command
        id: handle
        uses: idynic/idynic-github-actions/handle-issue-commands@v1
        with:
          command: ${{ steps.extract.outputs.command }}
          issue-number: ${{ github.event.issue.number }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          repository: ${{ github.repository }}
          comment-user: ${{ github.event.comment.user.login }}

      - name: Create branch if /start command
        if: steps.extract.outputs.command == 'start' && steps.handle.outputs.authorized == 'true'
        uses: idynic/idynic-github-actions/create-issue-branch@v1
        with:
          issue-number: ${{ github.event.issue.number }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          repository: ${{ github.repository }}
```

## Error Handling

The action provides clear error messages for:
- Unauthorized users attempting to use commands
- Invalid or unknown commands
- API failures during command processing

## Security Considerations

- Always use a GitHub token with appropriate permissions
- The action validates user permissions before executing any commands
- Commands are normalized to prevent injection attacks
- All user inputs are properly escaped when used in API calls

## Contributing

Contributions are welcome! Please see the [main repository's CONTRIBUTING.md](../CONTRIBUTING.md) for guidelines.

## License

This action is licensed under the same terms as the main repository. See [LICENSE](../LICENSE) for details.
