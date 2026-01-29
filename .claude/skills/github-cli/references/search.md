# gh search

## Issues

```bash
gh search issues "timeout" --repo owner/repo --state open
gh search issues "bug" --label critical --state open --owner my-org
gh search issues --assignee @me --state open
```

## Pull Requests

```bash
gh search prs "fix" --review-requested=@me --state open
gh search prs --author @me --merged --repo owner/repo
gh search prs --checks failure --state open
```

## Code

```bash
gh search code "func NewAgent" --repo owner/repo
gh search code "TODO" --owner my-org --language go
gh search code "interface Provider" --extension go
```
