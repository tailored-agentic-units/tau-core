# Plan: GitHub CLI and Skill Creator Skills

## Overview

Create two Claude Code skills:
1. **github-cli**: GitHub CLI operations skill for issue/PR/release management
2. **skill-creator**: Meta-skill for creating new Claude Code skills

## Design Decisions (Confirmed)

| Decision | Choice |
|----------|--------|
| Skill scope | Single comprehensive `github-cli` skill |
| Invocation | Auto-invocable; `gh` commands require user authorization |
| Location | Both project-level in `.claude/skills/` |
| Permissions | Add to `.claude/settings.json` allow list |

---

## Implementation

### Skill 1: `github-cli`

**Path:** `.claude/skills/github-cli/SKILL.md`

**Frontmatter:**
```yaml
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
```

**Content Structure:**
1. **When This Skill Applies** - Trigger scenarios
2. **Issue Operations**
   - Create: `gh issue create --title "..." --body "..." --label bug`
   - List: `gh issue list --state open --assignee @me`
   - Search: `gh search issues "query" --repo owner/repo`
   - View: `gh issue view <number>`
   - Edit: `gh issue edit <number> --add-label enhancement`
   - Close: `gh issue close <number>`
3. **Pull Request Operations**
   - Create: `gh pr create --title "..." --body "..."`
   - List: `gh pr list --state open`
   - View: `gh pr view <number>` / `gh pr diff`
   - Review: `gh pr review <number> --approve`
   - Merge: `gh pr merge <number> --squash`
   - Comments: `gh api repos/{owner}/{repo}/pulls/{number}/comments`
4. **Release Operations**
   - Create: `gh release create v1.0.0 --title "..." --notes "..."`
   - List: `gh release list`
5. **Search Operations**
   - Issues: `gh search issues "bug" --repo owner/repo --state open`
   - PRs: `gh search prs "fix" --review-requested=@me`
   - Code: `gh search code "function" --repo owner/repo`
6. **API Access**
   - REST: `gh api repos/{owner}/{repo}/issues`
   - GraphQL: `gh api graphql -f query='...'`
7. **Best Practices**
   - Use `--json` for structured output
   - Use HEREDOC for multi-line bodies
   - Reference existing commit message patterns

### Skill 2: `skill-creator`

**Path:** `.claude/skills/skill-creator/SKILL.md`

**Frontmatter:**
```yaml
---
name: skill-creator
description: >
  REQUIRED for creating or modifying Claude Code skills.
  Use when the user asks to "create a skill", "new skill", "add a skill",
  "modify skill", or needs help with SKILL.md format and frontmatter.
  Triggers: create skill, new skill, SKILL.md, skill template, frontmatter,
  slash command, custom command.

  When creating skills, follow this project's patterns: single SKILL.md per directory,
  YAML frontmatter with explicit trigger descriptions, "When This Skill Applies" section,
  organized content with code examples. Keep SKILL.md under 500 lines.
  After creating a skill, add it to .claude/settings.json permissions (alphabetical order).
---
```

**Content Structure:**
1. **When This Skill Applies** - Creating/modifying skills
2. **Skill Directory Structure**
   ```
   .claude/skills/<skill-name>/
   ├── SKILL.md           # Required: main instructions
   └── references/        # Optional: supporting docs
   ```
3. **Frontmatter Reference**
   - `name`: Skill identifier (kebab-case)
   - `description`: When to use + trigger phrases
   - `disable-model-invocation`: User-only if `true`
   - `user-invocable`: Hide from `/` menu if `false`
   - `allowed-tools`: Pre-approved tools
   - `context: fork`: Run in subagent
   - `agent`: Subagent type when forked
4. **Content Best Practices**
   - Start with "When This Skill Applies"
   - Use concrete trigger phrases in description
   - Include code examples
   - Keep SKILL.md under 500 lines
5. **Project Patterns** (reference existing skills)
6. **Settings Integration** - Add to `.claude/settings.json`

**Supporting File:** `.claude/skills/skill-creator/references/frontmatter-reference.md`
- Complete frontmatter field documentation
- Examples for each field
- Common patterns

---

## Settings Update

Update `.claude/settings.json` to add new skills (alphabetical order):

```json
{
  "plansDirectory": "./.claude/plans",
  "permissions": {
    "allow": [
      "Skill(github-cli)",
      "Skill(go-patterns)",
      "Skill(skill-creator)",
      "Skill(tau-core-admin)",
      "Skill(tau-core-dev)"
    ]
  }
}
```

---

## Files to Create/Modify

| File | Action |
|------|--------|
| `.claude/skills/github-cli/SKILL.md` | Create |
| `.claude/skills/skill-creator/SKILL.md` | Create |
| `.claude/skills/skill-creator/references/frontmatter-reference.md` | Create |
| `.claude/settings.json` | Modify (add permissions) |

---

## Verification

1. **github-cli skill:**
   - Invoke `/github-cli` and verify it loads
   - Test `gh issue list` (should prompt for authorization)
   - Test creating an issue with proper formatting

2. **skill-creator skill:**
   - Invoke `/skill-creator` and verify it loads
   - Ask to create a test skill, verify structure matches patterns

3. **Auto-invocation:**
   - Ask "create an issue for..." without `/github-cli`
   - Verify Claude loads the skill automatically
