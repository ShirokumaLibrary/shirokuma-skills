---
name: session-management
description: GitHub-integrated session management skill. Use when "/start-session", "/end-session", "セッション開始", "セッション終了", "引き継ぎ", "handover", "進捗報告", "show handover"
allowed-tools: Bash, Read, Write, Grep, Glob
---

# Session Management Skill

GitHub (Projects + Issues + Discussions) を活用したセッション管理。

## Project Naming Convention

Project 名 = リポジトリ名（例: `blogcms` リポジトリ → `blogcms` プロジェクト）

これにより、同一オーガニゼーション内の複数リポジトリが各自のプロジェクトを持てる。

## Architecture: Issues + Projects Hybrid

| コンポーネント | 用途 |
|--------------|------|
| **Issues** | タスク管理、`#123` 参照、履歴追跡 |
| **Projects** | Status/Priority/Type/Size フィールド管理 |
| **Labels** | Type 識別のみ (`feature`, `bug`, `chore`) |

**Status は Projects フィールドで管理**（Labels ではない）

## Prerequisites

- `gh` CLI installed and authenticated
- GitHub Project configured (run `/project-setup` if not)
- Repository with Discussions enabled (optional but recommended)
- Discussions categories configured: Handovers, Specs, Decisions, Q&A

## Commands

### /start-session

セッションを開始し、作業コンテキストを表示。

**Workflow:**

1. Get repository info from git remote
2. Fetch latest handover from Discussions (Handovers category)
3. List project items by Status (In Progress / Ready / Backlog)
4. Display session context with choices

**Output Format:**

```markdown
## Session Started

**Repository:** {owner/repo}
**Project:** {project name}
**Time:** {timestamp}

### Previous Handover
- Task: {currentTask}
- Next Steps: {nextSteps}
- Blockers: {blockers}

### Project Items

**In Progress:**
- #123 {title}

**Ready ({count}):**
- #124 {title}
- ...

---
Which item would you like to work on?
Or describe what you'd like to do today.
```

### /end-session

セッション終了、引き継ぎ情報を保存。

**Workflow:**

1. Summarize current session from conversation context
2. Update project item status if applicable
3. Create handover Discussion post (if Discussions enabled)
4. Display summary

### /show-items [filter]

プロジェクトアイテム一覧を表示。

**Examples:**
- `/show-items` - Active items (excludes Done/Released)
- `/show-items all` - All items including Done
- `/show-items ready` - Items with Ready status

### /create-item [title]

新しいプロジェクトアイテムを作成（DraftIssue または Issue）。

### /show-handovers [count]

過去の引き継ぎ一覧を表示。

## shirokuma-docs CLI Reference

Unified CLI tool for GitHub operations. 設定は `shirokuma-docs.config.yaml` の `github` セクションで管理。

### Configuration (`shirokuma-docs.config.yaml`)

All options have sensible defaults. Uncomment only to customize:

```yaml
# GitHub CLI integration (all values are defaults)
# github:
#   discussionsCategory: "Handovers"
#   listLimit: 20
#   labels:
#     feature: "feature"
#     bug: "bug"
#     chore: "chore"
```

**Note**: Done/Released items are filtered automatically:
- `gh-issues list`: Uses `--state open` by default (Done items are typically closed)
- `gh-projects list`: Excludes Done/Released by default (use `--all` to include)

### Commands

```bash
# ===== Issues (Primary Interface) =====
shirokuma-docs gh-issues list              # Active items (auto-excludes Done/Released)
shirokuma-docs gh-issues list --all        # Include closed
shirokuma-docs gh-issues get 123           # Get details
shirokuma-docs gh-issues update 123 --status "In Progress"
shirokuma-docs gh-issues comment 123 --body "..."

# Create with project fields
shirokuma-docs gh-issues create \
  --title "Title" --body "Description" \
  --labels feature --status "Backlog" --priority "Medium"

# ===== Projects (Low-level Access) =====
shirokuma-docs gh-projects list            # Project items
shirokuma-docs gh-projects fields          # Show field options
shirokuma-docs gh-projects add-issue 123   # Add issue to project

# ===== Discussions =====
shirokuma-docs gh-discussions list         # Uses default category from config
shirokuma-docs gh-discussions get 5
shirokuma-docs gh-discussions create --title "Title" --body "Body"  # Uses default category

# ===== Repository Info =====
shirokuma-docs gh-repo info
shirokuma-docs gh-repo labels
```

Features:
- **Config-based defaults**: `shirokuma-docs.config.yaml` の `github` セクション
- **Auto-detect**: Project by repository name
- **Zero config**: 全オプションにデフォルト値あり
- **JSON output**: All commands output JSON for scripting

## DraftIssue vs Issue

| 特徴 | DraftIssue | Issue |
|------|-----------|-------|
| `#番号` | なし | あり |
| 外部参照 | 不可 | 可能 (`#123`) |
| コメント | 不可 | 可能 |
| 作成 | `gh-projects create` | `gh-issues create` |
| 用途 | 軽量メモ | 本格タスク |

**推奨**: 基本的に `gh-issues create` を使用。`#123` で参照可能。

## Discussions Commands (Simplified)

`shirokuma-docs.config.yaml` に `github.discussionsCategory` があれば `--category` 省略可能:

```bash
# List handovers (uses discussionsCategory from config)
shirokuma-docs gh-discussions list --limit 5

# Create handover (uses default category)
shirokuma-docs gh-discussions create \
  --title "$(date +%Y-%m-%d) - Session Summary" \
  --body "Handover content..."
```

## Error Handling

| Error | Action |
|-------|--------|
| gh not installed | Show: `brew install gh` or `sudo apt install gh` |
| Not authenticated | Show: `gh auth login` |
| No project found | Show: `/project-setup` to create one |
| Discussions disabled | Skip handover post, save to local file |
| Category not found | List available categories, suggest creation |

## Status Workflow Reference

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

Labels は **Type 識別のみ** に使用（Status は Projects フィールド）：

| Label | Purpose |
|-------|---------|
| `feature` | New functionality |
| `bug` | Bug fix |
| `chore` | Maintenance |
| `docs` | Documentation |
| `research` | Investigation |

**Priority Labels** (オプション):
- `priority:critical`
- `priority:high`
