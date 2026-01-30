---
name: github-cli
description: >
  REQUIRED for ANY GitHub repository operations via the gh CLI.
  Use when the user asks to "create an issue", "list issues", "create PR",
  "search issues", "create release", "view PR comments", "check CI status",
  "manage labels", "set secret", "set variable", "create discussion", or any
  gh CLI operation.
  Triggers: issue, PR, pull request, release, GitHub, gh command, repository,
  milestone, label, assignee, workflow, actions, gist, secret, variable,
  discussion.

  When this skill is invoked, use the gh CLI to execute the requested operation.
  Always use --json flag for structured output when parsing is needed.
  For multi-line content (issue/PR bodies), use HEREDOC syntax.
---

# GitHub CLI Operations

Perform GitHub repository operations using the `gh` CLI.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Creating, listing, viewing, updating, or closing **discussions** (via GraphQL)
- Creating, listing, searching, viewing, editing, or closing **issues**
- Creating, listing, viewing, reviewing, or merging **pull requests**
- Creating, listing, or deleting **releases**
- Searching for issues, PRs, or code across repositories
- Viewing or triggering **GitHub Actions workflows**
- Managing **secrets** or **variables** for GitHub Actions
- Creating, editing, or deleting **labels**
- Creating **gists**
- Direct **GitHub API** calls (REST or GraphQL)
- Managing **labels**, **milestones**, or **assignees**

> **Scope boundary**: For GitHub Projects v2 (project boards, phases, cross-repo backlogs),
> use the **project-management** skill instead.

## Command Reference

Each `gh` subcommand has detailed documentation in its reference file:

| Command | Operations | Reference |
|---------|------------|-----------|
| `gh issue` | Create, list, search, view, edit, close | [issue.md](references/issue.md) |
| `gh pr` | Create, list, view, review, merge, status, ready, reopen | [pr.md](references/pr.md) |
| `gh release` | Create, list, view, delete | [release.md](references/release.md) |
| `gh search` | Search issues, PRs, code across repos | [search.md](references/search.md) |
| `gh workflow` / `gh run` | List, view, trigger, watch, download artifacts | [workflow.md](references/workflow.md) |
| `gh api` | REST and GraphQL API calls | [api.md](references/api.md) |
| `gh gist` | Create, list, view gists | [gist.md](references/gist.md) |
| `gh label` | Create, list, edit, delete, clone labels | [label.md](references/label.md) |
| Discussions | Create, list, view, update, close, comment (via GraphQL) | [discussion.md](references/discussion.md) |
| `gh secret` | Set, list, delete secrets (repo, env, org) | [secret.md](references/secret.md) |
| `gh variable` | Set, get, list, delete variables (repo, env, org) | [variable.md](references/variable.md) |

## Best Practices

1. **Structured output**: Use `--json` flag when you need to parse or filter results
2. **HEREDOC for bodies**: Always use HEREDOC syntax for multi-line issue/PR bodies
3. **Current repo context**: `gh` auto-detects the repo from the current git directory
4. **Rate limiting**: Use `--limit` to control result counts for search/list operations
5. **Web fallback**: Use `--web` flag to open in browser when CLI output is insufficient
6. **Labels**: Use `--label` multiple times or comma-separated for multiple labels
7. **Templates**: Reference project issue/PR templates when creating content
8. **Secrets vs Variables**: Use secrets for sensitive data (tokens, passwords); use variables for non-sensitive config (URLs, feature flags)
9. **Cross-repo targeting**: Use `-R owner/repo` to target a different repository
10. **Scoped operations**: Secrets, variables, and labels support repo, environment, and org scopes via `-e`, `-o` flags
