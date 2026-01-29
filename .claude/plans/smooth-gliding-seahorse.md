# Plan: Create `project-management` Skill

## Summary

Create a new Claude Code skill for managing GitHub Projects v2, phases (cross-repo grouping), and backlog items using the `gh project` CLI.

### Three-Tier Hierarchy

| Tier | GitHub Construct | Scope | Example |
|------|-----------------|-------|---------|
| **Project** | GitHub Projects v2 | Org/user | "TAU Platform" |
| **Phase** | SINGLE_SELECT custom field | Per project | "MVP Phase 1", "v0.0.2" |
| **Item** | Project item (issue/PR/draft) | Per project, any repo | Issue #42 from tau-core |

Phases replace repo-scoped milestones with a cross-repo construct via a custom field.

## Files to Create/Modify

### 1. Create: `.claude/skills/project-management/SKILL.md` (~485 lines)

**Frontmatter:**
- `name: project-management`
- `allowed-tools` — read-only `gh` commands only (no approval needed for context gathering):
  - `Bash(gh project list*)` — list projects
  - `Bash(gh project view*)` — view project details / get project ID
  - `Bash(gh project field-list*)` — list fields / resolve field and option IDs
  - `Bash(gh project item-list*)` — list backlog items / filter by phase
  - `Bash(gh issue list*)` — list issues for adding to projects
  - `Bash(gh issue view*)` — view issue details
  - `Bash(gh auth status*)` — verify token scope
- All mutating commands (`create`, `edit`, `delete`, `close`, `link`, `item-add`, `item-edit`, `field-create`, etc.) require user approval
- Description with triggers: project, phase, backlog, board, sprint, milestone, roadmap, cross-repo

**Sections:**

| Section | Content |
|---------|---------|
| When This Skill MUST Be Used | Trigger scenarios (create/list/view projects, manage phases, backlog ops, bootstrap, cross-repo) |
| Concepts | 3-tier hierarchy table, Phase explanation, ID chain diagram |
| Project Lifecycle | Create, list, view, edit, close/reopen, delete |
| Repository Linking | Link/unlink repos and teams to projects |
| Phase Management | Create Phase field (SINGLE_SELECT), list fields/options, add options (delete+recreate workaround with data-loss warning) |
| Backlog Management | Add issues by URL, create drafts, list items, assign phase (5-step ID chain), clear phase, archive, delete |
| Composite Workflows | Bootstrap project, bulk-add issues, view phase progress (jq grouping/counting/filtering), move items between phases, cross-repo overview |
| ID Resolution Patterns | Project ID, field ID, option ID, item ID resolution commands, optimized 2-call helper |
| Best Practices | 10 items: structured output, cache fields, limits, ID opacity, naming convention, scope boundary, token scope, web fallback, drafts, archive over delete |

**Scope boundary:** Covers `gh project *` commands only. Delegates issue/PR CRUD, releases, search, and actions to the existing `github-cli` skill.

### 2. Modify: `.claude/settings.json`

Add `"Skill(project-management)"` to `permissions.allow` in alphabetical order (between `go-patterns` and `skill-creator`).

### 3. Modify: `.claude/CLAUDE.md`

Add row to the Skills table:
```
| project-management | GitHub Projects v2, phases, cross-repo backlogs |
```

## Key Design Decisions

1. **Read-only `allowed-tools`** — Pre-approves `list`, `view`, `field-list`, `item-list`, and `auth status` commands; all mutating commands still require user approval
2. **No `references/` directory** — Content fits in a single SKILL.md under 500 lines
3. **ID Resolution as dedicated section** — The trickiest `gh project` aspect (chaining opaque IDs for `item-edit`) gets its own reference section with an optimized 2-call pattern
4. **Phase field deletion warning** — CLI cannot add individual options to an existing SINGLE_SELECT field; document the delete+recreate workaround and its data-loss risk
5. **`--jq` over `--template`** — Simpler and more composable with shell pipelines

## Verification

1. Load the skill: invoke `/project-management` or trigger via "create a project"
2. Verify no loader errors (the pattern that broke `skill-creator` must not appear)
3. Confirm `gh project --help` works (token scope: `project`)
4. Test a representative workflow: create project, link repo, create Phase field, add an issue, assign phase
