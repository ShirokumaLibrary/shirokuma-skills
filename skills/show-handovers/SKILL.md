---
name: show-handovers
description: Display past handover information. Use when "/show-handovers", "引き継ぎ確認", "handover list", "前回の作業", "履歴確認"
disable-model-invocation: true
argument-hint: "[count: 5|10|20]"
allowed-tools: Bash, Read, Glob
---

# Show Handovers

過去のセッション引き継ぎ情報を表示します。

## Usage

```
/show-handovers       # Last 5 handovers
/show-handovers 10    # Last 10 handovers
/show-handovers all   # All handovers
```

## Workflow

### Step 1: Check Data Sources

Try in order:
1. GitHub Discussions (Handovers category)
2. Local files (`.claude/sessions/*.md`)

### Step 2: Fetch from Discussions

```bash
gh api graphql -f query='{
  repository(owner: "{owner}", name: "{repo}") {
    discussions(first: {count}, categoryId: "{handovers_category_id}", orderBy: {field: CREATED_AT, direction: DESC}) {
      nodes { number title body createdAt url }
    }
  }
}'
```

**Note**: `gh discussion` command doesn't exist. Use GraphQL API instead.

### Step 3: Fetch from Local Files

If Discussions not available or as supplement:

```bash
ls -t .claude/sessions/*-handover.md 2>/dev/null | head -{count}
```

### Step 4: Display Results

```markdown
## Recent Handovers

📋 **2025-01-25** - Blog CMS 記事管理機能
   Summary: Implemented post CRUD with draft/publish workflow
   Next: Add category filtering and tag management
   Issues: #10, #12

📋 **2025-01-24** - session-management スキル作成
   Summary: Created /start-session and /end-session skills
   Next: Test with Discussions enabled
   Issues: #1, #2

---
Source: {GitHub Discussions | Local files | Both}
Showing: {count} of {total}

View full handover: `/show-handovers #{number}`
```

### Step 5: If Specific Handover

If argument is a number or date (e.g., `/show-handovers 1` or `/show-handovers 2025-01-24`):

Show full handover content:

```markdown
# Handover: 2025-01-24 - session-management スキル作成

## Summary
Created /start-session and /end-session skills for GitHub-centric workflow.

## Related Issues
- #1 - session-management スキル作成 (in progress)
- #2 - GitHub Issues/Discussions操作機能 (not started)

## Key Decisions
- Use gh CLI instead of GitHub API
- Separate skills for each command

## Next Steps
- [ ] Enable Discussions in repo
- [ ] Test /end-session with Discussion creation

## Modified Files
- .claude/skills/start-session/SKILL.md
- .claude/skills/end-session/SKILL.md
```

## Parsing Handover Content

Extract key sections from handover body:

| Section | Pattern |
|---------|---------|
| Summary | Text after `## Summary` |
| Issues | Lines matching `#\d+` after `## Related Issues` |
| Next Steps | Checkbox items after `## Next Steps` |
| Decisions | List items after `## Key Decisions` |

## Error Handling

| Error | Action |
|-------|--------|
| No Discussions category | Check local files |
| No local files | "No handovers found. Start with `/end-session`" |
| Both empty | "No handover history. This appears to be a new project." |

## Notes

- Combine Discussion and local handovers if both exist
- Sort by date (newest first)
- Show source indicator for each handover
- Truncate long summaries in list view
