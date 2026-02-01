---
name: add-issue-comment
description: Add comments to Issues/PRs. Use when "/add-issue-comment", "Issueにコメント", "PRにコメント", "comment on issue", "add comment"
disable-model-invocation: true
allowed-tools: Bash, AskUserQuestion
---

# Add Issue Comment

GitHub Issue または Pull Request にコメントを追加します。

## Usage

```
/add-issue-comment 123 "コメント内容"
/add-issue-comment #123                  # Interactive mode
/add-issue-comment                       # List recent issues first
```

## Workflow

### Step 1: Identify Target

If issue number not provided:

```bash
# List recent issues
gh issue list --limit 10 --json number,title,state \
  | jq -r '.[] | "#\(.number) [\(.state)] \(.title)"'
```

Ask user which issue to comment on.

### Step 2: Get Comment Content

If comment not provided as argument, ask user:

```
What would you like to comment?
```

### Step 3: Add Comment

```bash
gh issue comment $ISSUE_NUMBER --body "$COMMENT"
```

Or for PR:

```bash
gh pr comment $PR_NUMBER --body "$COMMENT"
```

### Step 4: Display Result

```markdown
## Comment Added

**Issue:** #123 - {title}
**Comment:** {first 100 chars}...

[View on GitHub]({issue_url})
```

## Comment Templates

Offer templates for common comments:

| Template | Content |
|----------|---------|
| LGTM | "LGTM! :+1:" |
| WIP | "Work in progress, will update soon." |
| Question | "Question: {user input}" |
| Blocker | ":warning: Blocked by: {user input}" |
| Done | "Completed in {commit/PR ref}" |

## Error Handling

| Error | Action |
|-------|--------|
| Issue not found | "Issue #X not found. Check the number." |
| No permission | "You don't have permission to comment." |
| Empty comment | "Comment cannot be empty." |

## Notes

- Works with both Issues and Pull Requests
- Supports markdown in comments
- Can mention users with @username
