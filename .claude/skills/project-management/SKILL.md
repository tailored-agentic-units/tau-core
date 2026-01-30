---
name: project-management
description: >
  REQUIRED for GitHub Projects v2 operations including project boards, phases,
  and cross-repo backlog management via the gh CLI.
  Use when the user asks to "create a project", "list projects", "add phase",
  "assign phase", "view backlog", "bootstrap project", "link repo to project",
  "move items between phases", or any gh project operation.
  Triggers: project, phase, backlog, board, project management, sprint, milestone,
  roadmap, cross-repo, project item, project field.

  When this skill is invoked, use the gh project CLI to execute the requested
  operation. Always use --format json with --jq for structured output when
  parsing IDs. For composite workflows, chain commands to resolve IDs.
allowed-tools:
  - "Bash(gh project list*)"
  - "Bash(gh project view*)"
  - "Bash(gh project field-list*)"
  - "Bash(gh project item-list*)"
  - "Bash(gh issue list*)"
  - "Bash(gh issue view*)"
  - "Bash(gh auth status*)"
---

# GitHub Project Management

Manage GitHub Projects v2, phases, and cross-repo backlogs using the `gh project` CLI.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Creating, listing, viewing, editing, closing, or deleting **projects**
- Linking or unlinking **repositories** to/from a project
- Creating or managing **phase fields** (SINGLE_SELECT fields on a project)
- Adding issues/PRs to a project or managing **backlog items**
- Viewing or filtering items **by phase**
- **Bootstrapping** a new project with repos and phases
- Moving items **between phases** or viewing **phase progress**
- Any **cross-repo** project board operation

> **Scope boundary**: This skill covers `gh project *` commands. For issue/PR CRUD,
> releases, search, and actions, use the **github-cli** skill instead.

## Concepts

### Three-Tier Hierarchy

| Tier | GitHub Construct | Scope | Purpose |
|------|-----------------|-------|---------|
| **Project** | GitHub Projects v2 | Org or user | Cross-repo board |
| **Phase** | SINGLE_SELECT custom field | Per project | Group items like milestones |
| **Item** | Project item (issue/PR/draft) | Per project | Backlog entry |

### Phase as Cross-Repo Milestone

Phases use a SINGLE_SELECT field on the project to group items from any linked repo.
Example phases: "Backlog", "Phase 1 - Foundation", "v0.0.2", "Done".

### Milestone Convention

Each non-meta phase (i.e., not "Backlog" or "Done") gets a **corresponding milestone**
on every linked repository. This provides repo-level progress metrics (X of Y closed)
that align with the project phase.

| Construct | Scope | Purpose |
|-----------|-------|---------|
| **Phase** | Cross-repo (project) | Organize items across all repos |
| **Milestone** | Per-repo | Progress tracking within a single repo |

**Rules:**
- Milestone names match phase names exactly (e.g., "Phase 1 - Foundation")
- When assigning a phase to an issue, also assign the corresponding milestone
- "Backlog" phase does not get a milestone (unscheduled work)

**Create milestones when adding a phase:**

```bash
# Create milestone on each linked repo
gh api --method POST /repos/<owner>/<repo>/milestones \
  -f title="Phase 1 - Foundation" \
  -f description="Foundation phase — see TAU Platform project board"
```

**Assign milestone when assigning phase:**

```bash
gh issue edit <number> --repo <owner>/<repo> --milestone "Phase 1 - Foundation"
```

### ID Chain

Many operations require resolved IDs. The chain flows:

```
project number --> project ID       (gh project view --format json)
                   field name --> field ID     (gh project field-list --format json)
                                  option name --> option ID  (from field-list options)
item URL -------> item ID           (gh project item-add --format json)
```

All IDs are opaque strings (e.g., `PVT_...`, `PVTSSF_...`, `PVTSO_...`, `PVTI_...`).

## Label Convention

All TAU Platform repositories use a shared label taxonomy focused on work type
categorization. Labels are general-purpose — no domain-specific ontology.

### Standard Labels

| Label | Description | Color |
|-------|-------------|-------|
| `bug` | Something isn't working correctly | `d73a4a` |
| `feature` | New capability or functionality | `0075ca` |
| `improvement` | Enhancement to existing functionality | `a2eeef` |
| `refactor` | Code restructuring without behavior change | `d4c5f9` |
| `documentation` | Documentation additions or updates | `0e8a16` |
| `testing` | Test additions or improvements | `fbca04` |
| `infrastructure` | CI/CD, build, tooling, project setup | `e4e669` |

### Bootstrap Labels on a New Repo

Replace the default GitHub labels with the standard set:

```bash
# Remove default labels
for label in "bug" "documentation" "duplicate" "enhancement" "good first issue" \
  "help wanted" "invalid" "question" "wontfix"; do
  gh label delete "$label" --repo <owner>/<repo> --yes 2>/dev/null
done

# Create standard labels
gh label create "bug"             --repo <owner>/<repo> --color d73a4a --description "Something isn't working correctly"
gh label create "feature"         --repo <owner>/<repo> --color 0075ca --description "New capability or functionality"
gh label create "improvement"     --repo <owner>/<repo> --color a2eeef --description "Enhancement to existing functionality"
gh label create "refactor"        --repo <owner>/<repo> --color d4c5f9 --description "Code restructuring without behavior change"
gh label create "documentation"   --repo <owner>/<repo> --color 0e8a16 --description "Documentation additions or updates"
gh label create "testing"         --repo <owner>/<repo> --color fbca04 --description "Test additions or improvements"
gh label create "infrastructure"  --repo <owner>/<repo> --color e4e669 --description "CI/CD, build, tooling, project setup"
```

### Clone Labels Across Repos

Once a source repo has the standard labels, clone them to new repos:

```bash
# Clone labels from source repo (overwrites existing)
gh label clone <owner>/tau-platform --repo <owner>/<new-repo> --force
```

## Project Lifecycle

### Create

```bash
# Create a project
gh project create --owner <owner> --title "TAU Platform" --format json

# Create and capture the project number
gh project create --owner <owner> --title "TAU Platform" --format json --jq '.number'
```

### List

```bash
# List open projects
gh project list --owner <owner> --format json

# Include closed projects
gh project list --owner <owner> --closed --format json

# Extract numbers and titles
gh project list --owner <owner> --format json --jq '.projects[] | {number, title}'
```

### View

```bash
# View project details
gh project view <number> --owner <owner> --format json

# Get project ID (needed for item-edit)
gh project view <number> --owner <owner> --format json --jq '.id'

# Open in browser
gh project view <number> --owner <owner> --web
```

### Edit

```bash
# Update title
gh project edit <number> --owner <owner> --title "New Title"

# Update description and visibility
gh project edit <number> --owner <owner> --description "Cross-repo roadmap" --visibility PRIVATE
```

### Close / Reopen

```bash
# Close a completed project
gh project close <number> --owner <owner>

# Reopen
gh project close <number> --owner <owner> --undo
```

### Delete

```bash
# Permanently delete (no undo)
gh project delete <number> --owner <owner>
```

## Repository Linking

Link repositories to make their issues available in the project.

### Link

```bash
# Link a specific repo
gh project link <number> --owner <owner> --repo <owner>/<repo>

# Link the current directory's repo
gh project link <number> --owner <owner>

# Link a team
gh project link <number> --owner <org> --team <team-slug>
```

### Unlink

```bash
# Unlink a repo
gh project unlink <number> --owner <owner> --repo <owner>/<repo>
```

## Phase Management

Phases are SINGLE_SELECT fields on the project, typically named "Phase".

### Create the Phase Field

```bash
# Create with initial options
gh project field-create <number> --owner <owner> \
  --name "Phase" \
  --data-type SINGLE_SELECT \
  --single-select-options "Backlog,MVP Phase 1,v0.0.2,Done"
```

### List Fields and Phase Options

```bash
# List all fields
gh project field-list <number> --owner <owner> --format json

# Get the Phase field ID
gh project field-list <number> --owner <owner> --format json \
  --jq '.fields[] | select(.name == "Phase") | .id'

# Get all Phase option IDs
gh project field-list <number> --owner <owner> --format json \
  --jq '.fields[] | select(.name == "Phase") | .options'

# Get a specific option ID by name
gh project field-list <number> --owner <owner> --format json \
  --jq '.fields[] | select(.name == "Phase") | .options[] | select(.name == "MVP Phase 1") | .id'
```

### Add Phase Options

The CLI does not support adding individual options to an existing SINGLE_SELECT field.
To add options, delete and recreate the field.

```bash
# 1. Get the field ID
FIELD_ID=$(gh project field-list <number> --owner <owner> --format json \
  --jq '.fields[] | select(.name == "Phase") | .id')

# 2. Delete the old field
gh project field-delete --id "$FIELD_ID"

# 3. Recreate with all options (existing + new)
gh project field-create <number> --owner <owner> \
  --name "Phase" \
  --data-type SINGLE_SELECT \
  --single-select-options "Backlog,MVP Phase 1,v0.0.2,Sprint 3,Done"
```

> **Warning**: Deleting the field clears all existing phase assignments on items.
> Use this only when setting up for the first time or when a fresh start is acceptable.
> For non-destructive option management, use the GitHub web UI or the GraphQL API.

### Delete a Phase Field

```bash
FIELD_ID=$(gh project field-list <number> --owner <owner> --format json \
  --jq '.fields[] | select(.name == "Phase") | .id')
gh project field-delete --id "$FIELD_ID"
```

## Backlog Management

### Add Issues to Project

```bash
# Add an issue by URL
gh project item-add <number> --owner <owner> \
  --url https://github.com/<owner>/<repo>/issues/42

# Add and capture the item ID
gh project item-add <number> --owner <owner> \
  --url https://github.com/<owner>/<repo>/issues/42 \
  --format json --jq '.id'
```

### Create Draft Items

```bash
# Create a draft issue directly on the project
gh project item-create <number> --owner <owner> \
  --title "Investigate performance regression" \
  --body "Details here..."
```

### List Items

```bash
# List all items (default 30)
gh project item-list <number> --owner <owner> --format json

# List up to 200 items
gh project item-list <number> --owner <owner> -L 200 --format json

# List with key details
gh project item-list <number> --owner <owner> --format json \
  --jq '.items[] | {id, title: .content.title, type: .content.type, repo: .content.repository}'
```

### Assign a Phase to an Item

Requires three resolved IDs: project ID, field ID, and option ID.

```bash
# Step 1: Get the project ID
PROJECT_ID=$(gh project view <number> --owner <owner> --format json --jq '.id')

# Step 2: Get the Phase field ID
FIELD_ID=$(gh project field-list <number> --owner <owner> --format json \
  --jq '.fields[] | select(.name == "Phase") | .id')

# Step 3: Get the option ID for the target phase
OPTION_ID=$(gh project field-list <number> --owner <owner> --format json \
  --jq '.fields[] | select(.name == "Phase") | .options[] | select(.name == "MVP Phase 1") | .id')

# Step 4: Get the item ID (if not already known)
ITEM_ID=$(gh project item-list <number> --owner <owner> --format json \
  --jq '.items[] | select(.content.title == "Issue title") | .id')

# Step 5: Set the phase
gh project item-edit \
  --id "$ITEM_ID" \
  --project-id "$PROJECT_ID" \
  --field-id "$FIELD_ID" \
  --single-select-option-id "$OPTION_ID"
```

### Clear a Phase Assignment

```bash
gh project item-edit \
  --id "$ITEM_ID" \
  --project-id "$PROJECT_ID" \
  --field-id "$FIELD_ID" \
  --clear
```

### Archive / Unarchive Items

```bash
# Archive a completed item
gh project item-archive <number> --owner <owner> --id <item-id>

# Unarchive
gh project item-archive <number> --owner <owner> --id <item-id> --undo
```

### Delete an Item

```bash
gh project item-delete <number> --owner <owner> --id <item-id>
```

## Composite Workflows

### Bootstrap a New Project

Create a project, link repos, set up the Phase field, and bootstrap labels.

```bash
# 1. Create the project
PROJECT_NUM=$(gh project create --owner <owner> --title "TAU Platform" \
  --format json --jq '.number')

# 2. Link repositories
gh project link "$PROJECT_NUM" --owner <owner> --repo <owner>/tau-core
gh project link "$PROJECT_NUM" --owner <owner> --repo <owner>/tau-skills

# 3. Create the Phase field
gh project field-create "$PROJECT_NUM" --owner <owner> \
  --name "Phase" \
  --data-type SINGLE_SELECT \
  --single-select-options "Backlog,MVP Phase 1,v0.0.2,Done"

# 4. Set visibility
gh project edit "$PROJECT_NUM" --owner <owner> --visibility PRIVATE

# 5. Bootstrap standard labels on linked repos (see Label Convention)
gh label clone <owner>/tau-platform --repo <owner>/tau-core --force
gh label clone <owner>/tau-platform --repo <owner>/tau-skills --force

# 6. Create milestones for non-meta phases on each linked repo (see Milestone Convention)
for repo in <owner>/tau-core <owner>/tau-skills; do
  gh api --method POST "/repos/$repo/milestones" \
    -f title="MVP Phase 1" \
    -f description="Phase 1 — see TAU Platform project board"
done
```

### Bulk Add Issues to Project

```bash
# Add all open issues from a repo
for url in $(gh issue list --repo <owner>/<repo> --state open --json url --jq '.[].url'); do
  gh project item-add <number> --owner <owner> --url "$url"
done
```

### View Phase Progress

```bash
# List items with their phase
gh project item-list <number> --owner <owner> -L 200 --format json \
  --jq '.items[] | {title: .content.title, repo: .content.repository, phase: (.fieldValues[] | select(.field.name == "Phase") | .name) // "Unassigned"}'

# Count items per phase
gh project item-list <number> --owner <owner> -L 200 --format json \
  --jq '[.items[] | (.fieldValues[] | select(.field.name == "Phase") | .name) // "Unassigned"] | group_by(.) | map({phase: .[0], count: length})'

# List items in a specific phase
gh project item-list <number> --owner <owner> -L 200 --format json \
  --jq '.items[] | select(any(.fieldValues[]; .field.name == "Phase" and .name == "MVP Phase 1")) | {title: .content.title, repo: .content.repository}'
```

### Move Items Between Phases

```bash
# Resolve IDs once
PROJECT_ID=$(gh project view <number> --owner <owner> --format json --jq '.id')
FIELD_ID=$(gh project field-list <number> --owner <owner> --format json \
  --jq '.fields[] | select(.name == "Phase") | .id')
NEW_OPTION_ID=$(gh project field-list <number> --owner <owner> --format json \
  --jq '.fields[] | select(.name == "Phase") | .options[] | select(.name == "v0.0.2") | .id')

# Move a single item
gh project item-edit \
  --id <item-id> \
  --project-id "$PROJECT_ID" \
  --field-id "$FIELD_ID" \
  --single-select-option-id "$NEW_OPTION_ID"

# Bulk move all items from one phase to another
OLD_PHASE="MVP Phase 1"
ITEM_IDS=$(gh project item-list <number> --owner <owner> -L 200 --format json \
  --jq ".items[] | select(any(.fieldValues[]; .field.name == \"Phase\" and .name == \"$OLD_PHASE\")) | .id")
for id in $ITEM_IDS; do
  gh project item-edit \
    --id "$id" \
    --project-id "$PROJECT_ID" \
    --field-id "$FIELD_ID" \
    --single-select-option-id "$NEW_OPTION_ID"
done
```

### Cross-Repo Backlog Overview

```bash
# Items grouped by repository
gh project item-list <number> --owner <owner> -L 200 --format json \
  --jq '[.items[] | {title: .content.title, repo: .content.repository, type: .content.type}] | group_by(.repo) | map({repo: .[0].repo, count: length, items: map(.title)})'
```

## ID Resolution Patterns

The `gh project item-edit` command requires opaque IDs, not human-readable names.

### Project ID

```bash
gh project view <number> --owner <owner> --format json --jq '.id'
# Returns: "PVT_kwHOA..."
```

### Field ID

```bash
gh project field-list <number> --owner <owner> --format json \
  --jq '.fields[] | select(.name == "Phase") | .id'
# Returns: "PVTSSF_lAHOA..."
```

### Option ID

```bash
gh project field-list <number> --owner <owner> --format json \
  --jq '.fields[] | select(.name == "Phase") | .options[] | select(.name == "MVP Phase 1") | .id'
# Returns: "PVTSO_..."
```

### Item ID

```bash
# From item-add output
gh project item-add <number> --owner <owner> --url <issue-url> --format json --jq '.id'
# Returns: "PVTI_lAHOA..."

# From item-list by title
gh project item-list <number> --owner <owner> --format json \
  --jq '.items[] | select(.content.title == "Bug: login timeout") | .id'

# From item-list by issue number and repo
gh project item-list <number> --owner <owner> --format json \
  --jq '.items[] | select(.content.number == 42 and .content.repository == "owner/repo") | .id'
```

### Optimized ID Resolution

Resolve all IDs needed for a phase assignment with minimal API calls:

```bash
# 2 API calls instead of 3: project view + field-list (reuse field-list output)
PROJECT_ID=$(gh project view <number> --owner <owner> --format json --jq '.id')
FIELDS_JSON=$(gh project field-list <number> --owner <owner> --format json)
FIELD_ID=$(echo "$FIELDS_JSON" | jq -r '.fields[] | select(.name == "Phase") | .id')
OPTION_ID=$(echo "$FIELDS_JSON" | jq -r '.fields[] | select(.name == "Phase") | .options[] | select(.name == "TARGET_PHASE") | .id')
```

## Best Practices

1. **Structured output**: Always use `--format json` with `--jq` when extracting IDs or
   filtering items programmatically
2. **Cache field data**: Fetch `field-list` once and extract multiple values with `jq`
   rather than making repeated API calls
3. **Limit item-list**: Use `-L` flag to set appropriate limits; default is 30, which
   may miss items in larger projects
4. **ID opacity**: Never hardcode IDs across sessions; always resolve them fresh since
   project items can be reindexed
5. **Phase field convention**: Name the field "Phase" consistently across all projects
   for predictable `--jq` selectors
6. **Scope boundary**: Use this skill for `gh project *` commands. For issue/PR lifecycle
   (create, edit, close, label, assign), use the **github-cli** skill
7. **Token scope**: `gh project` commands require the `project` scope.
   Verify with `gh auth status` and add with `gh auth refresh -s project`
8. **Web fallback**: Use `--web` on `gh project view` or `gh project list` to open in
   browser when detailed visualization is needed
9. **Draft items**: Use `item-create` for quick backlog entries that don't yet need a
   full issue in a specific repository
10. **Archive over delete**: Prefer `item-archive` over `item-delete` to preserve history
