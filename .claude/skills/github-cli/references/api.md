# gh api

## REST

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

## GraphQL

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
