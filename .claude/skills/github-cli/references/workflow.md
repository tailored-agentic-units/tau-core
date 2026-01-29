# gh workflow / gh run

## Workflows

```bash
# List workflows
gh workflow list

# Trigger a workflow
gh workflow run <workflow-name>
```

## Runs

```bash
# View recent runs
gh run list --limit 10

# View specific run
gh run view <run-id>

# Watch a run in progress
gh run watch <run-id>

# Download run artifacts
gh run download <run-id>
```
