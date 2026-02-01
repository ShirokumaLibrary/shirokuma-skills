---
name: manage-labels
description: Issue/PR label management skill. Use when "/manage-labels", "ラベル管理", "ラベル追加", "add label", "remove label", "list labels"
disable-model-invocation: true
allowed-tools: Bash, AskUserQuestion
---

# Manage Labels

GitHub Issue/PR のラベルを管理します。

## Usage

```
/manage-labels                           # List all labels
/manage-labels add 123 bug              # Add label to issue
/manage-labels remove 123 bug           # Remove label from issue
/manage-labels create "new-label" "ff0000"  # Create new label
```

## Workflow

### List Labels

```bash
gh label list --json name,color,description \
  | jq -r '.[] | "- \(.name) (#\(.color)): \(.description // "No description")"'
```

### Add Label to Issue

```bash
gh issue edit $ISSUE_NUMBER --add-label "$LABEL_NAME"
```

### Remove Label from Issue

```bash
gh issue edit $ISSUE_NUMBER --remove-label "$LABEL_NAME"
```

### Create New Label

```bash
gh label create "$LABEL_NAME" --color "$COLOR" --description "$DESCRIPTION"
```

### Delete Label

```bash
gh label delete "$LABEL_NAME" --yes
```

## Standard Labels

Suggest these labels for new projects:

| Label | Color | Description |
|-------|-------|-------------|
| bug | d73a4a | Something isn't working |
| feature | 0075ca | New feature or request |
| docs | 0052cc | Documentation |
| good first issue | 7057ff | Good for newcomers |
| help wanted | 008672 | Extra attention needed |
| wontfix | ffffff | Will not be worked on |
| duplicate | cfd3d7 | Duplicate issue |
| priority:high | b60205 | High priority |
| priority:low | c5def5 | Low priority |

## Commands

| Command | Action |
|---------|--------|
| `list` | Show all labels |
| `add <issue> <label>` | Add label to issue |
| `remove <issue> <label>` | Remove label from issue |
| `create <name> <color>` | Create new label |
| `delete <name>` | Delete label |
| `setup` | Create standard labels |

## Interactive Mode

If no arguments provided:

```
What would you like to do?
1. List all labels
2. Add label to issue
3. Remove label from issue
4. Create new label
5. Setup standard labels
```

## Error Handling

| Error | Action |
|-------|--------|
| Label not found | "Label '{name}' not found. Create it first?" |
| Issue not found | "Issue #X not found." |
| Label exists | "Label '{name}' already exists." |
| No permission | "You don't have permission to manage labels." |

## Notes

- Colors are hex without # (e.g., "ff0000" for red)
- Label names are case-insensitive
- Can add multiple labels: `--add-label "bug,priority:high"`
