---
name: create-item
description: Project item creation skill for Issues or DraftIssues. Use when "/create-item", "アイテム作成", "タスク追加", "create item", "add task", "新規アイテム"
disable-model-invocation: true
allowed-tools: Bash, AskUserQuestion
---

# Create Item

GitHub Project にアイテム（Issue または DraftIssue）を作成します。

> **Note**: Project は自動検出（リポジトリ名と同名）。詳細は `session-management` スキル参照。

## Usage

```
/create-item                    # Interactive mode
/create-item "タイトル"          # With title
/create-item --type feature     # Specify type
/create-item --issue            # Force Issue creation
```

## DraftIssue vs Issue

| 特徴 | DraftIssue | Issue |
|------|-----------|-------|
| `#番号` | なし | あり (`#123`) |
| 外部参照 | 不可 | 可能 |
| コメント | 不可 | 可能 |
| 作成コマンド | `gh-projects create` | `gh-issues create` |
| 用途 | 軽量メモ | 本格タスク |

**推奨**: 基本的に `gh-issues create` を使用。`#123` で参照可能。

## Workflow

### Step 1: Gather Item Details

If title not provided as argument, ask user:

```
What's the title of this item?
```

Then ask for type:

| Type | Description |
|------|-------------|
| Feature | New functionality |
| Bug | Bug fix |
| Chore | Maintenance, refactoring |
| Docs | Documentation |
| Research | Investigation, spike |

### Step 2: Generate Body Template

Based on type, generate body:

```markdown
## 概要
{Ask user for description}

## タスク
- [ ] {task 1}

## Deliverable
{What "done" looks like}
```

### Step 3: Set Fields

Ask user for:

| Field | Options | Default |
|-------|---------|---------|
| Priority | Critical / High / Medium / Low | Medium |
| Size | XS / S / M / L / XL | S |
| Status | Backlog / Ready | Backlog |

### Step 4: Create Item with shirokuma-docs

**Create Issue (推奨)**:

```bash
shirokuma-docs gh-issues create \
  --title "Item Title" \
  --body "Description" \
  --labels feature \
  --status "Backlog" \
  --priority "Medium" \
  --type "Feature" \
  --size "M"
```

**Create DraftIssue (軽量)**:

```bash
shirokuma-docs gh-projects create \
  --title "Item Title" \
  --body "Description" \
  --status "Backlog" \
  --priority "Medium" \
  --type "Feature" \
  --size "M"
```

CLI features:
- Auto-detects project by repository name
- Handles field ID lookups automatically
- Sets all fields in single operation
- Issue creation includes automatic Project linking

### Step 5: Display Result

```markdown
## Item Created

**Issue:** #123 (or DraftIssue)
**Title:** {title}
**Type:** {type}
**Priority:** {priority}
**Size:** {size}
**Status:** {status}

---
View in project: {project URL}

**Quick actions:**
- Start working: `/start-session`
- Update status: `shirokuma-docs gh-issues update 123 --status "In Progress"`
- Show all items: `/show-items`
```

## CLI Commands Reference

```bash
# Create Issue + add to Project (推奨)
shirokuma-docs gh-issues create \
  --title "Title" --body "Description" \
  --labels feature bug \
  --status "Backlog" --priority "Medium" --type "Feature" --size "M"

# Create DraftIssue
shirokuma-docs gh-projects create \
  --title "Title" --body "Description" \
  --status "Backlog" --priority "Medium" --type "Feature" --size "M"

# Add existing Issue to Project
shirokuma-docs gh-projects add-issue 123 \
  --status "Backlog" --priority "Medium"

# Show available field options
shirokuma-docs gh-projects fields
```

## Error Handling

| Error | Action |
|-------|--------|
| No project found | "No GitHub Project found matching repository name. Run `/project-setup` to create one." |
| gh not authenticated | "Please run `gh auth login` first." |
| Field not found | Use default values, warn user |
| Label not found | Create label or warn user |

## Notes

- Always set all required fields (Status, Priority, Type)
- Size is recommended but optional
- XL items should prompt user to consider splitting
- Project is auto-detected by repository name
- Prefer `gh-issues create` over `gh-projects create` for `#number` support
