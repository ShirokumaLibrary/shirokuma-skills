---
name: show-project-items
description: Display GitHub Project items with Status filter and issue numbers. Use when "/show-items", "アイテム確認", "項目一覧", "タスク確認", "project items"
disable-model-invocation: true
argument-hint: "[filter: ready|in-progress|backlog|done|all]"
allowed-tools: Bash
---

# Show Project Items

GitHub Project アイテムを一覧表示します。Statusフィルタ、#番号対応。

> **Note**: Project は自動検出（リポジトリ名と同名）。詳細は `session-management` スキル参照。

## Usage

```
/show-items              # Active items (excludes Done/Released)
/show-items all          # All items including Done
/show-items ready        # Items with "Ready" status
/show-items in-progress  # Items with "In Progress" status
/show-items backlog      # Items in Backlog
```

## Workflow

### Step 1: Fetch Project Items

Use shirokuma-docs CLI for efficient fetching:

```bash
# Active items (open issues)
shirokuma-docs gh-issues list

# All items including closed
shirokuma-docs gh-issues list --all

# Filter by status (supports multiple)
shirokuma-docs gh-issues list --status Ready
shirokuma-docs gh-issues list --status "In Progress" --status Ready --status Backlog

# Get specific item details (by Issue number)
shirokuma-docs gh-issues get 9

# For DraftIssue or item ID
shirokuma-docs gh-projects get PVTI_xxx
```

### Step 2: Parse Status Filter

Check `$ARGUMENTS` for filter type:

| Argument | CLI Flag |
|----------|----------|
| (empty) | (default - open issues) |
| `all` | `--all` |
| `ready` | `--status Ready` |
| `in-progress` | `--status "In Progress"` |
| `backlog` | `--status Backlog` |
| `icebox` | `--status Icebox` |

### Step 3: Display Results

**Grouped View (default):**

```markdown
## Project Items

**In Progress (1):**
- #9 shirokuma-docs: GitHub CLI コマンド群追加 (XL, Medium)

**Backlog (2):**
- #10 Feature A (M, High)
- - Draft item B (S, Medium)

**Icebox (1):**
- #8 Future enhancement (L, Low)

---
Total: 4 active items
Project: blogcms

**Quick actions:**
- Start working: `/start-session`
- Create item: `/create-item`
- Get details: `shirokuma-docs gh-issues get 9`
```

**Filtered View:**

```markdown
## Ready Items (2)

| # | Title | Priority | Type | Size |
|---|-------|----------|------|------|
| #10 | Feature A | High | Feature | M |
| - | Draft item | Medium | Bug | S |

---
Filter: Ready
```

## Issue Number Support

Issue 番号でアイテムを操作できます:

```bash
# 詳細取得
shirokuma-docs gh-issues get 9

# ステータス更新
shirokuma-docs gh-issues update 9 --status "In Progress"

# コメント追加
shirokuma-docs gh-issues comment 9 --body "Comment text"
```

## CLI Commands Reference

```bash
# Issues (primary interface)
shirokuma-docs gh-issues list                       # Open issues
shirokuma-docs gh-issues list --all           # All issues
shirokuma-docs gh-issues list --status Ready        # Filter by status
shirokuma-docs gh-issues get 9                      # Get by number
shirokuma-docs gh-issues update 9 --status "In Progress"

# Projects (low-level access)
shirokuma-docs gh-projects list                     # Project items
shirokuma-docs gh-projects get PVTI_xxx             # By item ID
shirokuma-docs gh-projects delete 9                 # Remove from project
shirokuma-docs gh-projects fields                   # Show field options

# Debug mode
shirokuma-docs gh-issues list --verbose             # Show debug output
```

## Status Workflow Reference

```
Icebox → Backlog → Spec Review → Ready → In Progress → Review → Testing → Done → Released
                                              ↓
                                          Pending (blocked)
```

| Status | Description |
|--------|-------------|
| Icebox | Low priority, not yet planned |
| Backlog | Planned for future |
| Spec Review | Requirements being reviewed |
| Ready | Ready to start |
| In Progress | Currently working on |
| Pending | Blocked (needs reason) |
| Review | Code review |
| Testing | QA testing |
| Done | Completed |
| Released | Deployed |

## Error Handling

| Error | Action |
|-------|--------|
| No project found | "No GitHub Project found matching repository name. Run `/project-setup` to create one." |
| No items match filter | "No items with status '{filter}'. Try `/show-items` to see all." |
| Invalid status | Script shows available options |
| gh error | Show error message from gh |

## Notes

- Default view shows open issues only
- Items sorted by Priority within each status
- Shows custom fields (Priority, Type, Size)
- Project is auto-detected by repository name
- Use `gh-issues` for Issue operations, `gh-projects` for DraftIssue/item ID operations
