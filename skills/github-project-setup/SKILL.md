---
name: github-project-setup
description: Automates GitHub Project initial setup with Status, Priority, Type, and Size fields. Use when "project setup", "プロジェクト作成", "GitHub Project初期設定", or starting a new project with kanban workflow.
---

# GitHub Project Setup

Automates GitHub Project initial setup including Status workflow, Priority, Type, and Size custom fields.

## When to Use

- Creating a new GitHub Project
- Setting up a kanban workflow
- When user says "project setup", "プロジェクト作成", or "GitHub Project"

## Workflow

### Step 1: Check Permissions

```bash
gh auth status
```

If permission is missing, ask user to run:

```bash
gh auth refresh -s project,read:project
```

### Step 2: Get Repository Info

```bash
OWNER=$(gh repo view --json owner -q '.owner.login' 2>/dev/null)
REPO=$(gh repo view --json name -q '.name' 2>/dev/null)
```

### Step 3: Create Project

```bash
PROJECT_NAME="${1:-$REPO Roadmap}"
gh project create --owner $OWNER --title "$PROJECT_NAME" --format json
```

### Step 4: Link to Repository

```bash
gh project link $PROJECT_NUMBER --owner $OWNER --repo $OWNER/$REPO
```

This makes the project accessible from the repository's Projects tab.

### Step 5: Get Field IDs

```bash
PROJECT_NUMBER=$(gh project list --owner $OWNER --format json | jq -r '.projects[0].number')
FIELD_ID=$(gh project field-list $PROJECT_NUMBER --owner $OWNER --format json | jq -r '.fields[] | select(.name=="Status") | .id')
PROJECT_ID=$(gh project view $PROJECT_NUMBER --owner $OWNER --format json | jq -r '.id')
```

### Step 6: Configure All Fields

Use the setup script with language auto-detected from conversation:

```bash
python scripts/setup-project.py \
  --lang={en|ja} \
  --field-id=$FIELD_ID \
  --project-id=$PROJECT_ID
```

**Fields created**:

| Field | Options |
|-------|---------|
| Status | Icebox → Backlog → Spec Review → Ready → In Progress ⇄ Pending → Review → Testing → Done → Released |
| Priority | Critical / High / Medium / Low |
| Type | Feature / Bug / Chore / Docs / Research |
| Size | XS / S / M / L / XL |

See [scripts/setup-project.py](scripts/setup-project.py) for language dictionaries.

### Step 7: Report Results

After completion, display:

- Project name and URL
- Configured Status list
- Added custom fields

## Status Workflow

**Normal Flow**:

Icebox → Backlog → Spec Review → Ready → In Progress → Review → Testing → Done → Released

**Exception Flows**:

| Pattern | Flow | Description |
|---------|------|-------------|
| Requirements unclear | Spec Review → Backlog | Needs reconsideration |
| Blocked | Any → Pending → Original status | Temporary hold (reason required) |
| Review feedback | Review → In Progress | Fix requested changes |
| Test failed | Testing → In Progress | Bug fix needed |
| Simple task | Backlog → Ready | Skip Spec Review if requirements are clear |

**Operational Rules**:

1. One task In Progress per person (WIP limit)
2. Always document reason when moving to Pending
3. Review tasks stuck in same status for over a week
4. Keep Ready queue stocked with actionable tasks

## Error Handling

| Error | Solution |
|-------|----------|
| `missing scopes [project]` | Run `gh auth refresh -s project,read:project` |
| `Project already exists` | Show existing project URL |
| `Owner not found` | Use `--owner` option explicitly |

## Notes

- Permission refresh requires interactive mode (user must run manually)
- Language auto-detected from conversation (Japanese or English)
- For AI development, Size (effort) is more useful than time estimates
- XL-sized tasks should be split into smaller tasks

## Related Resources

- [scripts/setup-project.py](scripts/setup-project.py) - Setup script with language dictionaries
- [reference/status-options.md](reference/status-options.md) - Status workflow and definitions
- [reference/custom-fields.md](reference/custom-fields.md) - Custom field definitions
