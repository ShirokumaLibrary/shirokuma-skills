---
name: show-issues
description: GitHub Issues list display skill with filtering. Use when "/show-issues", "Issue一覧", "バグ一覧", "list issues", "open issues"
disable-model-invocation: true
allowed-tools: Bash
---

# Show Issues

GitHub Issues を一覧表示します。

## Usage

```
/show-issues                 # All open issues
/show-issues --all           # Include closed
/show-issues --label bug     # Filter by label
/show-issues --assignee @me  # My issues
```

## Workflow

### Step 1: Build Query

Parse arguments to build gh command:

| Argument | shirokuma-docs flag |
|----------|---------------------|
| --all | --all |
| --closed | --state closed |
| --label X | --label X |
| --assignee X | --assignee X |
| --author X | --author X |

### Step 2: Fetch Issues

```bash
gh issue list \
  --state open \
  --json number,title,state,labels,assignees,createdAt,updatedAt \
  --limit 20
```

### Step 3: Display Results

```markdown
## Issues

**Filter:** {filter description}
**Total:** {count} issues

| # | Title | Labels | Assignee | Updated |
|---|-------|--------|----------|---------|
| #123 | Fix login bug | `bug` | @user | 2d ago |
| #124 | Add dark mode | `feature` | - | 1w ago |

---

**Quick actions:**
- View issue: `gh issue view 123`
- Add comment: `/add-issue-comment 123`
- Add label: `/manage-labels add 123 bug`
```

## Label Colors

Display labels with visual indicators:

```
🔴 bug
🟢 feature
🔵 docs
🟡 help wanted
⚪ wontfix
```

## Error Handling

| Error | Action |
|-------|--------|
| No issues found | "No issues match the filter." |
| Invalid label | "Label '{name}' not found." |
| API error | Show error message |

## Notes

- Default shows open issues only
- Sorted by updated date (most recent first)
- Maximum 50 issues per query
