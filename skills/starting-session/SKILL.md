---
name: starting-session
description: Start a work session showing project status and previous handovers. Use when "/start-session", "セッション開始", "作業開始", "start session", "begin work"
allowed-tools: Bash, Read, Grep
---

# Starting Session

Start a new work session and display project context.

> **Reference**: See `showing-github/common/reference/github-operations.md` for CLI commands and error handling.

## Workflow

### Step 1: Get Repository Info

```bash
REPO=$(gh repo view --json nameWithOwner -q '.nameWithOwner' 2>/dev/null || echo "unknown")
```

### Step 2: Check for Previous Handover

```bash
shirokuma-docs gh-discussions list --category Handovers --limit 1
```

If found, get details:

```bash
shirokuma-docs gh-discussions get <number>
```

If Discussions not enabled or category missing, skip gracefully.

### Step 3: List Project Items

```bash
shirokuma-docs gh-issues list
```

Group by Status: In Progress → Ready → Backlog → Icebox

### Step 4: Display Session Context

```markdown
## Session Started

**Repository:** {repo}
**Project:** {project name}
**Time:** {current time}

### Previous Handover
{title or "None found"}
- Summary: {summary}
- Next Steps: {next steps}

### Project Items

**In Progress:**
- {title}

**Ready ({count}):**
- {title}

**Backlog ({count}):**
- {title}
```

### Step 5: Ask for Direction

```
Which item would you like to work on today?
Or describe what you'd like to accomplish.

If selecting an item, I'll move it to "In Progress" status.
```

## If Item ID Provided as Argument

1. Find the item in the project
2. Move status to "In Progress": `shirokuma-docs gh-issues update {number} --status "In Progress"`
3. Start working on that item

## Error Handling

| Error | Action |
|-------|--------|
| `gh: command not found` | Install: `brew install gh` or `sudo apt install gh` |
| `not logged in` | Run: `gh auth login` |
| No project found | "Run `/project-setup` to create one." |
| Discussions error | Skip handover, continue with items |

## Notes

- Always show current time in session header
- Parse handover body for Summary and Next Steps
- Show items in priority order within each status
