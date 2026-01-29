---
name: github-cli
description: >
  REQUIRED for ANY GitHub repository operations via the gh CLI.
  Use when the user asks to "create an issue", "list issues", "create PR",
  "search issues", "create release", "view PR comments", "check CI status",
  or any gh CLI operation. Triggers: issue, PR, pull request, release, GitHub,
  gh command, repository, milestone, label, assignee, workflow, actions, gist.

  When this skill is invoked, use the gh CLI to execute the requested operation.
  Always use --json flag for structured output when parsing is needed.
  For multi-line content (issue/PR bodies), use HEREDOC syntax.
---

# GitHub CLI Operations

Perform GitHub repository operations using the `gh` CLI.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Creating, listing, searching, viewing, editing, or closing **issues**
- Creating, listing, viewing, reviewing, or merging **pull requests**
- Creating or listing **releases**
- Searching for issues, PRs, or code across repositories
- Viewing or triggering **GitHub Actions workflows**
- Creating **gists**
- Direct **GitHub API** calls (REST or GraphQL)
- Managing **labels**, **milestones**, or **assignees**

## Issue Operations

### Create

```bash
# Simple issue
gh issue create --title "Bug: login fails on timeout" --body "Steps to reproduce..."

# With labels and assignee
gh issue create --title "Add retry logic" --label enhancement --label "good first issue" --assignee @me

# Multi-line body using HEREDOC
gh issue create --title "Feature request" --body "$(cat <<'EOF'
## Summary
Description of the feature.

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2

## Context
Additional context here.
EOF
)"
```

### List

```bash
# Open issues assigned to me
gh issue list --state open --assignee @me

# Filter by label
gh issue list --label bug --state open

# JSON output for parsing
gh issue list --json number,title,labels,state --limit 50

# Filter by milestone
gh issue list --milestone "v1.0"
```

### Search

```bash
# Search in current repo
gh search issues "timeout error" --repo $(gh repo view --json nameWithOwner -q .nameWithOwner)

# Search with filters
gh search issues "bug" --repo owner/repo --state open --label critical

# Search across an organization
gh search issues "memory leak" --owner my-org --state open
```

### View

```bash
# View issue details
gh issue view 42

# JSON output
gh issue view 42 --json title,body,labels,assignees,comments

# View in browser
gh issue view 42 --web
```

### Edit

```bash
# Add labels
gh issue edit 42 --add-label enhancement --add-label "help wanted"

# Remove label
gh issue edit 42 --remove-label bug

# Reassign
gh issue edit 42 --add-assignee username

# Update title and body
gh issue edit 42 --title "Updated title" --body "Updated description"

# Set milestone
gh issue edit 42 --milestone "v1.0"
```

### Close

```bash
# Close with comment
gh issue close 42 --comment "Fixed in #45"

# Close as not planned
gh issue close 42 --reason "not planned"
```

## Pull Request Operations

### Create

```bash
# Basic PR
gh pr create --title "Fix timeout bug" --body "Resolves #42"

# PR with HEREDOC body
gh pr create --title "Add retry logic" --body "$(cat <<'EOF'
## Summary
- Added exponential backoff retry logic to HTTP client
- Configurable max retries and initial backoff

## Test Plan
- [ ] Unit tests pass
- [ ] Integration tests with Ollama
EOF
)"

# Draft PR
gh pr create --title "WIP: new feature" --draft

# Target specific base branch
gh pr create --base develop --title "Feature branch merge"

# With reviewers
gh pr create --title "Ready for review" --reviewer user1,user2

# With labels
gh pr create --title "Bug fix" --label bug --label urgent
```

### List

```bash
# Open PRs
gh pr list --state open

# PRs authored by me
gh pr list --author @me

# PRs requesting my review
gh pr list --search "review-requested:@me"

# JSON output
gh pr list --json number,title,headRefName,state,reviewDecision
```

### View

```bash
# View PR details
gh pr view 45

# View diff
gh pr diff 45

# View only changed file names
gh pr diff 45 --name-only

# View PR comments
gh api repos/{owner}/{repo}/pulls/45/comments

# JSON output
gh pr view 45 --json title,body,commits,files,reviews
```

### Review

```bash
# Approve
gh pr review 45 --approve

# Request changes
gh pr review 45 --request-changes --body "Please fix the error handling"

# Comment
gh pr review 45 --comment --body "Looks good, minor suggestions"
```

### Merge

```bash
# Squash merge (preferred for clean history)
gh pr merge 45 --squash

# Merge commit
gh pr merge 45 --merge

# Rebase
gh pr merge 45 --rebase

# Auto-merge when checks pass
gh pr merge 45 --auto --squash

# Delete branch after merge
gh pr merge 45 --squash --delete-branch
```

### Check Status

```bash
# View CI checks status
gh pr checks 45

# Wait for checks to complete
gh pr checks 45 --watch
```

## Release Operations

### Create

```bash
# Create release from tag
gh release create v1.0.0 --title "v1.0.0" --notes "Initial release"

# Generate release notes automatically
gh release create v1.0.0 --generate-notes

# Draft release
gh release create v1.0.0 --draft --title "v1.0.0 Release Candidate"

# With HEREDOC notes
gh release create v1.0.0 --title "v1.0.0" --notes "$(cat <<'EOF'
## What's New
- Feature A
- Feature B

## Bug Fixes
- Fixed issue #42
EOF
)"

# Upload release assets
gh release create v1.0.0 ./dist/binary-linux ./dist/binary-darwin
```

### List

```bash
# List releases
gh release list

# JSON output
gh release list --json tagName,publishedAt,isPrerelease
```

### View

```bash
gh release view v1.0.0
```

## Search Operations

### Issues

```bash
gh search issues "timeout" --repo owner/repo --state open
gh search issues "bug" --label critical --state open --owner my-org
gh search issues --assignee @me --state open
```

### Pull Requests

```bash
gh search prs "fix" --review-requested=@me --state open
gh search prs --author @me --merged --repo owner/repo
gh search prs --checks failure --state open
```

### Code

```bash
gh search code "func NewAgent" --repo owner/repo
gh search code "TODO" --owner my-org --language go
gh search code "interface Provider" --extension go
```

## GitHub Actions / Workflows

```bash
# List workflows
gh workflow list

# View recent runs
gh run list --limit 10

# View specific run
gh run view <run-id>

# Watch a run in progress
gh run watch <run-id>

# Trigger a workflow
gh workflow run <workflow-name>

# Download run artifacts
gh run download <run-id>
```

## API Access

### REST

```bash
# Get repository details
gh api repos/{owner}/{repo}

# List issue comments
gh api repos/{owner}/{repo}/issues/42/comments

# Create a comment
gh api repos/{owner}/{repo}/issues/42/comments -f body="Comment text"

# Paginated results
gh api repos/{owner}/{repo}/issues --paginate
```

### GraphQL

```bash
gh api graphql -F owner='{owner}' -F name='{repo}' -f query='
  query($name: String!, $owner: String!) {
    repository(owner: $owner, name: $name) {
      releases(last: 3) {
        nodes { tagName }
      }
    }
  }
'
```

## Gist Operations

```bash
# Create a gist from a file
gh gist create myfile.go --desc "Example code"

# Create public gist
gh gist create myfile.go --public

# Create from stdin
echo "code snippet" | gh gist create --filename snippet.go

# List gists
gh gist list

# View a gist
gh gist view <gist-id>
```

## Best Practices

1. **Structured output**: Use `--json` flag when you need to parse or filter results
2. **HEREDOC for bodies**: Always use HEREDOC syntax for multi-line issue/PR bodies
3. **Current repo context**: `gh` auto-detects the repo from the current git directory
4. **Rate limiting**: Use `--limit` to control result counts for search/list operations
5. **Web fallback**: Use `--web` flag to open in browser when CLI output is insufficient
6. **Labels**: Use `--label` multiple times or comma-separated for multiple labels
7. **Templates**: Reference project issue/PR templates when creating content
