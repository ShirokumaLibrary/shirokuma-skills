---
name: show-dashboard
description: Project dashboard display skill aggregating GitHub data. Use when "/show-dashboard", "ダッシュボード", "プロジェクト状況", "project status", "overview", "統計"
disable-model-invocation: true
allowed-tools: Bash
---

# Show Dashboard

GitHub からデータを取得し、プロジェクトの統計ダッシュボードを表示します。

## Usage

```
/show-dashboard              # Full dashboard
/show-dashboard --quick      # Quick summary only
```

## Workflow

### Step 1: Get Repository Info

```bash
OWNER=$(gh repo view --json owner -q '.owner.login')
REPO=$(gh repo view --json name -q '.name')
echo "Repository: $OWNER/$REPO"
```

### Step 2: Gather Statistics

Run these commands in parallel:

```bash
# Project items by status
gh project item-list 1 --owner $OWNER --format json | jq -r '
  .items | group_by(.status) |
  map({status: .[0].status, count: length}) |
  sort_by(.count) | reverse | .[]'

# Recent commits (last 7 days)
gh api repos/$OWNER/$REPO/commits?per_page=10 | jq -r '.[].commit.message' | head -5

# Open issues count
gh issue list --state open --json number | jq length

# Open PRs count
gh pr list --state open --json number | jq length

# Recent discussions (handovers)
gh api graphql -f query='{
  repository(owner: "'$OWNER'", name: "'$REPO'") {
    discussions(first: 3, orderBy: {field: CREATED_AT, direction: DESC}) {
      nodes { title createdAt category { name } }
    }
  }
}'
```

### Step 3: Display Dashboard

```markdown
# 📊 Project Dashboard

**Repository:** {owner}/{repo}
**Generated:** {timestamp}

---

## Project Items

| Status | Count | Bar |
|--------|-------|-----|
| Done | 6 | ██████████ |
| In Progress | 1 | ██ |
| Backlog | 2 | ████ |
| Icebox | 2 | ████ |

**Total:** {total} items
**Completion:** {done/total * 100}%

---

## Activity

| Metric | Count |
|--------|-------|
| Open Issues | {count} |
| Open PRs | {count} |
| Commits (7d) | {count} |

---

## Recent Commits

1. {commit message 1}
2. {commit message 2}
3. {commit message 3}

---

## Recent Handovers

| Date | Title |
|------|-------|
| {date} | {title} |
| {date} | {title} |

---

## Quick Actions

- `/show-project-items` - View all items
- `/show-handovers` - View handover history
- `/create-item` - Add new item
```

## Quick Mode (--quick)

Simplified output:

```markdown
## Quick Status

**Items:** 6 Done / 1 In Progress / 2 Backlog
**Issues:** 3 open | **PRs:** 1 open
**Last commit:** {message} ({time ago})
```

## Visual Elements

Use these for visual representation:

```
Progress bar: ██████████░░░░░░ 60%
Status icons: ✅ Done | 🔄 In Progress | 📋 Backlog | 🧊 Icebox
Trend: ↑ Up | ↓ Down | → Stable
```

## Error Handling

| Error | Action |
|-------|--------|
| No project found | Show only Issues/PRs/Commits |
| API rate limit | Show cached/partial data |
| gh not authenticated | Prompt to run `gh auth login` |

## Notes

- Dashboard data is fetched on-demand (not cached)
- All data comes from GitHub API via gh CLI
- Works offline with reduced functionality (git log only)
