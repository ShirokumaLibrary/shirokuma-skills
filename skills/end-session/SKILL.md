---
name: end-session
description: Session end skill saving handover info and updating Project items. Use when "/end-session", "セッション終了", "作業終了", "end session", "finish work", "引き継ぎ保存"
disable-model-invocation: true
allowed-tools: Bash, Read, Write, Grep, Glob
---

# End Session

現在のセッションを終了し、引き継ぎ情報を自動保存します。

> **Note**: Project は自動検出（リポジトリ名と同名）。詳細は `session-management` スキル参照。

## Workflow

Execute the following steps in order:

### Step 1: Gather Session Summary

Analyze the current conversation to extract:

1. **Summary**: What was accomplished (1-2 sentences)
2. **Related Items**: Project items worked on
3. **Key Decisions**: Important decisions made during the session
4. **Blockers**: Any blockers encountered
5. **Next Steps**: What should be done next
6. **Modified Files**: Files that were created or modified (from git status)

### Step 2: Get Modified Files

```bash
git status --short | head -20
```

### Step 3: Update Project Items (if applicable)

If items were completed, update their status using shirokuma-docs:

```bash
# Update item status to Done
shirokuma-docs gh-issues update 123 --status "Done"

# Or mark for review
shirokuma-docs gh-issues update 123 --status "Review"
```

### Step 4: Save Handover (Auto-save, no confirmation)

Create a Discussion using shirokuma-docs:

```bash
shirokuma-docs gh-discussions create \
  --category Handovers \
  --title "$(date +%Y-%m-%d) - {brief summary}" \
  --body "$HANDOVER_BODY"
```

If Discussions are not available, save to local file:

```bash
mkdir -p .claude/sessions
echo "$HANDOVER_BODY" > .claude/sessions/$(date +%Y-%m-%d-%H%M%S)-handover.md
```

### Step 5: Display Summary

After saving, display the result:

```markdown
## セッション終了

**保存先:** {Discussion URL or local file path}

### 今回の成果
{summary}

### 完了したアイテム
- {item 1} → Done
- {item 2} → Done

### 次のステップ
- [ ] {task 1}
- [ ] {task 2}

---
お疲れさまでした！
```

## Handover Format

Use this format for the handover body:

```markdown
## Summary
{What was accomplished}

## Related Items
- {project item title} - {status}

## Key Decisions
- {decision with rationale}

## Blockers
- {blocker or "None"}

## Next Steps
- [ ] {actionable task}

## Modified Files
- `path/file.ts` - {change description}

## Notes
{additional context}
```

## CLI Commands Reference

```bash
# List current items (for reference)
shirokuma-docs gh-issues list

# Get item details (supports #number)
shirokuma-docs gh-issues get 9       # By Issue number

# Update item status
shirokuma-docs gh-issues update 9 --status "Done"
shirokuma-docs gh-issues update 9 --status "Review"

# For Projects-only operations (DraftIssue etc)
shirokuma-docs gh-projects get PVTI_xxx
shirokuma-docs gh-projects update PVTI_xxx --status "Done"
```

## Issue Number Support

Issue 番号でアイテムを操作できます：

| 操作 | コマンド例 |
|------|-----------|
| 詳細取得 | `gh-issues get 9` |
| ステータス更新 | `gh-issues update 9 --status "Done"` |
| コメント追加 | `gh-issues comment 9 --body "..."` |

**Note**: DraftIssue は Issue 番号を持たないため、`gh-projects` で item ID (`PVTI_xxx`) を使用。

## Error Handling

| Error | Action |
|-------|--------|
| Discussion creation fails | Save to local file instead |
| No category "Handovers" | Save to local file |
| gh not authenticated | Save to local file |
| No changes in session | Still save a brief handover |

## Local Fallback

If GitHub Discussions are not available:

1. Create `.claude/sessions/` directory if not exists
2. Save handover as Markdown: `YYYY-MM-DD-HHMMSS-handover.md`
3. Inform user of local file path

## Notes

- Auto-save without confirmation (faster workflow)
- Always generate a summary even if brief
- Extract project item references from conversation context
- Use git status to find modified files
- Local fallback ensures handover is never lost
- Project is auto-detected by repository name
