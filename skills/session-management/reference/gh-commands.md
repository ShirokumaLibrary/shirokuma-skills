# CLI Commands Reference

Quick reference for commands used in session management. **Prefer shirokuma-docs CLI** over direct `gh` commands.

## Issues (shirokuma-docs CLI)

```bash
# List open issues
shirokuma-docs gh-issues list

# List all (including closed)
shirokuma-docs gh-issues list --all

# Filter by status
shirokuma-docs gh-issues list --status "In Progress" --status "Ready"

# View issue details
shirokuma-docs gh-issues get {number}

# Create issue with project fields
shirokuma-docs gh-issues create \
  --title "Title" --body "Body" \
  --labels "bug" \
  --status "Backlog" --priority "Medium" --type "Bug" --size "S"

# Update project fields
shirokuma-docs gh-issues update {number} --status "In Progress"

# Add comment
shirokuma-docs gh-issues comment {number} --body "Comment text"

# Close issue
shirokuma-docs gh-issues close {number}

# Reopen issue
shirokuma-docs gh-issues reopen {number}

# Cross-repo operations
shirokuma-docs gh-issues list --repo docs
shirokuma-docs gh-issues create --repo docs --title "Title" --body "Body"
```

## Discussions (shirokuma-docs CLI)

```bash
# List by category
shirokuma-docs gh-discussions list --category Handovers --limit 5

# Get discussion details
shirokuma-docs gh-discussions get {number}

# Create discussion
shirokuma-docs gh-discussions create \
  --category Handovers \
  --title "2025-01-31 - Summary" \
  --body "## Summary ..."
```

## Projects (shirokuma-docs CLI)

```bash
# List project items
shirokuma-docs gh-projects list

# Show available fields
shirokuma-docs gh-projects fields

# Update project fields
shirokuma-docs gh-projects update {number} --status "Done"
```

## gh Fallback (CLI未対応のみ)

```bash
# Labels (CLI未対応 - shirokuma-docs#11 で対応予定)
gh issue edit {number} --add-label "in-progress"
gh issue edit {number} --remove-label "blocked"

# Pull Requests
gh pr list --state open
gh pr view {number}
gh pr create --title "Title" --body "Body"

# Labels management
gh label list
gh label create "label-name" --color "0E8A16" --description "Description"

# Repository info
gh repo view --json nameWithOwner -q '.nameWithOwner'

# Authentication
gh auth login
gh auth status
```

## Useful Patterns

### Get latest handover

```bash
shirokuma-docs gh-discussions list --category Handovers --limit 1
```

### Count open issues by label

```bash
gh issue list --state open --json labels | jq '[.[].labels[].name] | group_by(.) | map({label: .[0], count: length})'
```

### Get issues updated today

```bash
gh issue list --state all --json number,title,updatedAt | jq --arg today "$(date +%Y-%m-%d)" '[.[] | select(.updatedAt | startswith($today))]'
```

## Error Handling

| Error | Solution |
|-------|----------|
| `shirokuma-docs: command not found` | Install: `npm i -g @shirokuma-library/shirokuma-docs` |
| `gh: command not found` | Install: `brew install gh` or `sudo apt install gh` |
| `not logged in` | Run: `gh auth login` |
| `Category not found` | Check Discussions categories are set up |
| `HTTP 404` | Check repository name and permissions |

## Notes

- **Always prefer shirokuma-docs CLI** over direct gh commands
- Fallback to `gh` only when shirokuma-docs doesn't support the operation
- JSON output is the default for shirokuma-docs commands
- Cross-repo aliases: `docs`, `blogcms`, `infra`, `skills`
