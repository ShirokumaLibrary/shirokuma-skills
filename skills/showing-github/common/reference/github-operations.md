# GitHub Operations Reference

Shared reference for all session/GitHub skills. Single source of truth for CLI commands, workflows, and conventions.

## Architecture: Issues + Projects Hybrid

| Component | Purpose |
|-----------|---------|
| **Issues** | Task management, `#123` references, history |
| **Projects** | Status/Priority/Type/Size field management |
| **Labels** | Type identification only (`feature`, `bug`, `chore`) |
| **Discussions** | Handovers, Specs, Decisions, Q&A |

**Status is managed via Projects fields** (not Labels).

Project naming convention: Project name = repository name (e.g., `blogcms` repo → `blogcms` project).

## Prerequisites

- `gh` CLI installed and authenticated
- GitHub Project configured (run `/project-setup` if not)
- Discussions enabled with categories: Handovers, Ideas, Q&A (optional)

## DraftIssue vs Issue

| Feature | DraftIssue | Issue |
|---------|-----------|-------|
| `#number` | No | Yes (`#123`) |
| External reference | No | Yes |
| Comments | No | Yes |
| Create command | `gh-projects create` | `gh-issues create` |
| Use case | Lightweight memo | Full task |

**Recommendation**: Use `gh-issues create` by default for `#number` support.

## shirokuma-docs CLI Reference

Prefer shirokuma-docs CLI over direct `gh` commands. Config in `shirokuma-docs.config.yaml`.

### Issues (Primary Interface)

```bash
shirokuma-docs gh-issues list                          # Open issues
shirokuma-docs gh-issues list --all                    # Include closed
shirokuma-docs gh-issues list --status "In Progress"   # Filter by status
shirokuma-docs gh-issues get {number}                  # Details
shirokuma-docs gh-issues create \
  --title "Title" --body "Body" \
  --labels feature \
  --status "Backlog" --priority "Medium" --type "Feature" --size "M"
shirokuma-docs gh-issues update {number} --status "In Progress"
shirokuma-docs gh-issues comment {number} --body "..."
shirokuma-docs gh-issues close {number}
shirokuma-docs gh-issues reopen {number}
```

### Projects (Low-level Access)

```bash
shirokuma-docs gh-projects list                        # Project items
shirokuma-docs gh-projects fields                      # Show field options
shirokuma-docs gh-projects add-issue {number}          # Add issue to project
shirokuma-docs gh-projects create \
  --title "Title" --body "Body" \
  --status "Backlog" --priority "Medium"               # DraftIssue
shirokuma-docs gh-projects get PVTI_xxx                # By item ID
shirokuma-docs gh-projects update {number} --status "Done"
```

### Discussions

```bash
shirokuma-docs gh-discussions list --category Handovers --limit 5
shirokuma-docs gh-discussions get {number}
shirokuma-docs gh-discussions create \
  --category Handovers \
  --title "$(date +%Y-%m-%d) - Summary" \
  --body "Content"
```

### Repository

```bash
shirokuma-docs gh-repo info
shirokuma-docs gh-repo labels
```

### Cross-repo Operations

```bash
shirokuma-docs gh-issues list --repo docs
shirokuma-docs gh-issues create --repo docs --title "Title" --body "Body"
```

### gh Fallback (CLI unsupported only)

```bash
# Labels
gh issue edit {number} --add-label "label"
gh issue edit {number} --remove-label "label"
gh label list
gh label create "name" --color "0E8A16" --description "Desc"

# Pull Requests
gh pr list --state open
gh pr view {number}
gh pr comment {number} --body "..."

# Repository info
gh repo view --json nameWithOwner -q '.nameWithOwner'

# Authentication
gh auth login
gh auth status
```

## Status Workflow

```
Icebox → Backlog → Spec Review → Ready → In Progress → Review → Testing → Done → Released
                                              ↓
                                          Pending (blocked)
```

| Status | Description |
|--------|-------------|
| Icebox | Low priority, not yet planned |
| Backlog | Planned for future work |
| Spec Review | Requirements being reviewed |
| Ready | Ready to start |
| In Progress | Currently working on |
| Pending | Blocked (document reason) |
| Review | Code review |
| Testing | QA testing |
| Done | Completed |
| Released | Deployed to production |

## Labels Convention

Labels are used for **Type identification only** (Status is via Projects fields):

| Label | Purpose |
|-------|---------|
| `feature` | New functionality |
| `bug` | Bug fix |
| `chore` | Maintenance |
| `docs` | Documentation |
| `research` | Investigation |

Optional priority labels: `priority:critical`, `priority:high`

## Common Error Handling

| Error | Action |
|-------|--------|
| `shirokuma-docs: command not found` | Install: `npm i -g @shirokuma-library/shirokuma-docs` |
| `gh: command not found` | Install: `brew install gh` or `sudo apt install gh` |
| `not logged in` / `not authenticated` | Run: `gh auth login` |
| No project found | Run `/project-setup` to create one |
| Discussions disabled/category not found | Use local file fallback |
| `HTTP 404` | Check repository name and permissions |
| API rate limit | Show cached/partial data |
