# Jira Ticket Check GitHub Action

This GitHub Action validates that all commits in a Pull Request have valid Jira tickets that are in an acceptable status (open, in progress, or done/closed).

## Features

- ✅ Extracts Jira ticket IDs from commit messages (format: `AAAA-NNNN`)
- ✅ Validates tickets exist in Jira
- ✅ Checks ticket status against allowed statuses
- ✅ Fails the PR if any commit lacks a valid ticket
- ✅ Provides detailed output for each commit
- ✅ Pure bash implementation - no dependencies required

## Setup

### 1. Create Jira API Token

1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
2. Click "Create API token"
3. Give it a name (e.g., "GitHub PR Validation")
4. **Required Scopes**: The token needs the following read-only permissions:
   - `read:jira-work` - Read issues and project data
   - `read:account` - Read account information
   - `read:jira-user` - Read user information
   - `read:me` - Read your own user details
5. Copy the token (you won't be able to see it again)

> **Note**: These are read-only scopes - the action does not modify any Jira data.

### 2. Configure GitHub Variables and Secrets

Add the following to your repository (Settings → Secrets and variables → Actions):

**Variables** (Settings → Secrets and variables → Actions → Variables tab):
- `JIRA_BASE_URL`: Your Jira instance URL (e.g., `https://your-domain.atlassian.net` or just `your-domain.atlassian.net`)
  - The action will automatically add `https://` if not provided
  - Trailing slashes are automatically removed
- `JIRA_EMAIL`: Email address for Jira API authentication

**Secrets** (Settings → Secrets and variables → Actions → Secrets tab):
- `JIRA_API_TOKEN`: Jira API token created above

### 3. Add the Workflow File

The workflow file is already created at `.github/workflows/jira-ticket-check.yml`

## Usage

### Commit Message Format

Include a Jira ticket ID in your commit messages. The pattern matches any uppercase letters followed by a hyphen and numbers: `[A-Z]+-[0-9]+`

Examples:

```
PROJ-123: Add new feature

This commit adds a new feature for user authentication.
```

Or:

```
Fix bug in login flow (TEAM-456)
```

Multiple tickets in one commit:

```
PROJ-123 PROJ-124: Update authentication system
```

### Valid Ticket Statuses

The action considers the following statuses as valid:
- In Progress
- In Development
- In Review
- Done
- Closed
- Resolved

You can modify the `is_valid_status` function in the workflow file to customize this list.

## How It Works

1. **Trigger**: The action runs on PR open, synchronize, or reopen events
2. **Commit Analysis**: Fetches all commits in the PR
3. **Ticket Extraction**: Searches commit messages for Jira ticket IDs using pattern `[A-Z]+-[0-9]+`
4. **Validation**: For each ticket:
   - Checks if the ticket exists in Jira using the REST API
   - Verifies the ticket status is in the allowed list
5. **Result**: 
   - ✅ Passes if all commits have valid tickets
   - ❌ Fails if any commit lacks a ticket or has an invalid ticket

## Example Output

```
🔍 Checking Jira tickets in PR commits...

📋 Ticket Pattern: [A-Z]+-[0-9]+
✅ Valid Statuses: In Progress, In Development, In Review, Done, Closed, Resolved

Found 3 commit(s) to check

📝 Commit a1b2c3d: Add user authentication
   🎫 Found ticket(s): PROJ-123
   ✅ Ticket PROJ-123 is valid (Status: In Progress)
   🔗 https://your-domain.atlassian.net/browse/PROJ-123

📝 Commit e4f5g6h: Fix login bug
   🎫 Found ticket(s): PROJ-456
   ✅ Ticket PROJ-456 is valid (Status: Done)
   🔗 https://your-domain.atlassian.net/browse/PROJ-456

📝 Commit i7j8k9l: Update README
   ❌ No Jira ticket found in commit message

============================================================
📊 SUMMARY
============================================================

❌ PR validation FAILED

Invalid commits:
  - i7j8k9l: No Jira ticket found
```

## Customization

### Modify Valid Statuses

Edit the `is_valid_status` function in `.github/workflows/jira-ticket-check.yml`:

```bash
is_valid_status() {
  local status=$(echo "$1" | tr '[:upper:]' '[:lower:]')
  case "$status" in
    "in progress"|"in development"|"in review"|"done"|"closed"|"resolved")
      return 0
      ;;
    # Add more statuses here like:
    # "custom status"|"another status")
    #   return 0
    #   ;;
    *)
      return 1
      ;;
  esac
}
```

### Change Ticket Pattern

The current pattern matches `[A-Z]+-[0-9]+`. To modify it, edit the grep command in the workflow:

```bash
# Current pattern
tickets=$(echo "$commit_msg" | grep -oE '[A-Z]+-[0-9]+' | sort -u)

# Example: Match lowercase too
tickets=$(echo "$commit_msg" | grep -oE '[A-Za-z]+-[0-9]+' | sort -u)
```

## Troubleshooting

### "No commits found in this PR"
- Ensure the workflow has `fetch-depth: 0` to fetch all git history
- Check that the PR has commits

### "Jira credentials not configured"
- Verify the variable `JIRA_BASE_URL` and `JIRA_EMAIL` are set
- Verify the secret `JIRA_API_TOKEN` is set
- Ensure the variables and secret are available to the workflow

### "Failed to fetch Jira ticket"
- Check the API token is valid and not expired
- Verify the email has access to the Jira project
- Ensure the Jira base URL is correct (no trailing slash)

### Lint Warnings about Context Access

The lint warnings about "Context access might be invalid" are informational and can be ignored. They appear because the linter cannot verify if the secrets exist in your repository, but the workflow will work correctly once you add the secrets.

## License

MIT
