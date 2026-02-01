---
name: start-session
description: Session start skill showing GitHub Projects and previous handovers. Use when "/start-session", "セッション開始", "作業開始", "start session", "begin work"
disable-model-invocation: true
argument-hint: "[project-item-id]"
allowed-tools: Bash, Read, Grep
---

# Start Session

新しい作業セッションを開始し、コンテキストを表示します。

## Workflow

Execute the following steps in order:

### Step 1: Get Repository Info

```bash
REPO=$(gh repo view --json nameWithOwner -q '.nameWithOwner' 2>/dev/null || echo "unknown")
OWNER=$(gh repo view --json owner -q '.owner.login' 2>/dev/null)
echo "Repository: $REPO"
```

### Step 2: Check for Previous Handover

Get the latest handover from Discussions:

```bash
shirokuma-docs gh-discussions list --category Handovers --limit 1
```

If handover found, get details:

```bash
shirokuma-docs gh-discussions get <number>
```

If Discussions are not enabled or category doesn't exist, skip this step gracefully.

### Step 3: List Project Items

Use shirokuma-docs CLI for efficient fetching:

```bash
shirokuma-docs gh-issues list
```

Features:
- **GraphQL-based**: Efficient API calls
- **Pagination**: Handles 100+ items automatically
- **JSON output**: Easy to parse and display

#### CLI Commands

```bash
# List active items (open issues)
shirokuma-docs gh-issues list

# List all items including closed
shirokuma-docs gh-issues list --all

# Filter by status
shirokuma-docs gh-issues list --status "In Progress" --status "Ready"

# Get item details (includes body)
shirokuma-docs gh-issues get 123

# Show available field options (Projects fields)
shirokuma-docs gh-projects fields

# Create new item (Issue + Project)
shirokuma-docs gh-issues create \
  --title "Title" --body "Description" \
  --labels feature \
  --status "Backlog" --priority "Medium" --type "Feature" --size "M"

# Update item
shirokuma-docs gh-issues update 123 --status "In Progress"
```

Group items by Status for display:
- **In Progress**: Items currently being worked on
- **Ready**: Items ready to start
- **Backlog**: Items prioritized for future work
- **Icebox**: Low priority items (shown separately)

### Step 4: Display Session Context

Format the output as:

```markdown
## Session Started

**Repository:** {repo}
**Project:** {project name}
**Time:** {current time}

### Previous Handover
{title or "None found"}
- Summary: {summary from handover body}
- Next Steps: {next steps from handover}

### Project Items

**In Progress:**
- {title} (Status: In Progress)

**Ready ({count}):**
- {title}
- ...

**Backlog ({count}):**
- {title}
- ...
```

### Step 5: Ask for Direction

Ask the user:

```
Which item would you like to work on today?
Or describe what you'd like to accomplish.

If selecting an item, I'll move it to "In Progress" status.
```

## If Item ID Provided as Argument

If user provides a project item (e.g., `/start-session {item-title}`):

1. Find the item in the project
2. Move status to "In Progress"
3. Start working on that item

## Project Status Updates

Use shirokuma-docs CLI to update item status:

```bash
# Update status via gh-issues (recommended)
shirokuma-docs gh-issues update 123 --status "In Progress"

# Or via gh-projects
shirokuma-docs gh-projects update 123 --status "In Progress"
```

## Error Handling

| Error | Action |
|-------|--------|
| `gh: command not found` | Tell user to install: `brew install gh` or `sudo apt install gh` |
| `not logged in` | Tell user to run: `gh auth login` |
| No project found | Say "No GitHub Project found. Run `/project-setup` to create one." |
| Discussions error | Skip handover display, continue with project items |

## Notes

- Always show the current time in the session header
- Parse handover body to extract key information (Summary, Next Steps)
- Group project items by Status
- Show items in priority order within each status
