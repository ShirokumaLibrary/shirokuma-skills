---
name: ending-session
description: End a work session saving handover info and updating project items. Use when "/end-session", "セッション終了", "作業終了", "end session", "finish work", "引き継ぎ保存"
allowed-tools: Bash, Read, Write, Grep, Glob
---

# Ending Session

End the current session and auto-save handover information.

> **Reference**: See `showing-github/common/reference/github-operations.md` for CLI commands.
> **Template**: See `common/templates/handover-template.md` for handover format.

## Workflow

### Step 1: Gather Session Summary

Analyze conversation to extract:
1. **Summary**: What was accomplished (1-2 sentences)
2. **Related Items**: Project items worked on
3. **Key Decisions**: Important decisions with rationale
4. **Blockers**: Any blockers encountered
5. **Next Steps**: Actionable tasks for next session
6. **Modified Files**: From `git status --short`

### Step 2: Get Modified Files

```bash
git status --short | head -20
```

### Step 3: Update Project Items

```bash
# Mark completed items
shirokuma-docs gh-issues update {number} --status "Done"

# Or mark for review
shirokuma-docs gh-issues update {number} --status "Review"
```

### Step 4: Save Handover (auto-save, no confirmation)

**To GitHub Discussions:**

```bash
shirokuma-docs gh-discussions create \
  --category Handovers \
  --title "$(date +%Y-%m-%d) - {brief summary}" \
  --body "$HANDOVER_BODY"
```

**Local fallback** (if Discussions unavailable):

```bash
mkdir -p .claude/sessions
echo "$HANDOVER_BODY" > .claude/sessions/$(date +%Y-%m-%d-%H%M%S)-handover.md
```

### Step 5: Display Summary

```markdown
## Session Ended

**Saved to:** {Discussion URL or local path}

### Accomplishments
{summary}

### Completed Items
- {item} → Done

### Next Steps
- [ ] {task 1}
- [ ] {task 2}
```

## Handover Body Format

```markdown
## Summary
{What was accomplished}

## Related Items
- #{number} - {title} - {status}

## Key Decisions
- {decision with rationale}

## Blockers
- {blocker or "None"}

## Next Steps
- [ ] {actionable task}

## Commits (this session)
| Hash | Description |
|------|-------------|
| {hash} | {message} |

## Modified Files
- `path/file.ts` - {change description}

## Notes
{additional context}
```

## Error Handling

| Error | Action |
|-------|--------|
| Discussion creation fails | Save to local file |
| No "Handovers" category | Save to local file |
| gh not authenticated | Save to local file |
| No changes in session | Still save a brief handover |

## Notes

- Auto-save without confirmation for faster workflow
- Always generate a summary even if brief
- Local fallback ensures handover is never lost
