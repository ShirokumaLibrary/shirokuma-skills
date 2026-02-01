---
name: managing-github-items
description: Create and manage GitHub project items, specs, comments, and labels. Use when "/create-item", "/create-spec", "/add-issue-comment", "/manage-labels", "アイテム作成", "仕様作成", "コメント追加", "ラベル管理"
allowed-tools: Bash, AskUserQuestion, Read, Write
---

# Managing GitHub Items

Create and manage GitHub project items, spec Discussions, issue comments, and labels.

> **Reference**: See `showing-github/common/reference/github-operations.md` for CLI commands, DraftIssue vs Issue comparison, and error handling.

## /create-item [title]

Create a GitHub Project item (Issue or DraftIssue).

```
/create-item                    # Interactive mode
/create-item "Title"            # With title
/create-item --type feature     # Specify type
```

### Workflow

**Step 1: Gather Details**

If title not provided, ask user. Then ask for type:

| Type | Label |
|------|-------|
| Feature | `feature` |
| Bug | `bug` |
| Chore | `chore` |
| Docs | `docs` |
| Research | `research` |

**Step 2: Generate Body**

```markdown
## 概要
{description}

## タスク
- [ ] {task 1}

## Deliverable
{what "done" looks like}
```

**Step 3: Set Fields**

| Field | Options | Default |
|-------|---------|---------|
| Priority | Critical / High / Medium / Low | Medium |
| Size | XS / S / M / L / XL | S |
| Status | Backlog / Ready | Backlog |

**Step 4: Create**

```bash
# Issue (recommended - supports #number)
shirokuma-docs gh-issues create \
  --title "Title" --body "Body" \
  --labels feature \
  --status "Backlog" --priority "Medium" --type "Feature" --size "M"

# DraftIssue (lightweight)
shirokuma-docs gh-projects create \
  --title "Title" --body "Body" \
  --status "Backlog" --priority "Medium"
```

**Step 5: Display Result**

```markdown
## Item Created
**Issue:** #123 | **Type:** Feature | **Priority:** Medium | **Status:** Backlog
```

---

## /create-spec [title]

Create a spec/design Discussion in Ideas category.

```
/create-spec "Feature Name"
/create-spec                    # Interactive mode
```

### Workflow

**Step 1: Get Category ID**

```bash
OWNER=$(gh repo view --json owner -q '.owner.login')
REPO=$(gh repo view --json name -q '.name')
CATEGORY_ID=$(gh api graphql -f query='{
  repository(owner: "'$OWNER'", name: "'$REPO'") {
    discussionCategories(first: 10) { nodes { id name } }
  }
}' | jq -r '.data.repository.discussionCategories.nodes[] | select(.name == "Ideas") | .id')
```

**Step 2: Gather Details**

Ask user for: Title, Summary, Problem Statement, Proposed Solution, Alternatives (optional)

**Step 3: Generate Body**

```markdown
## Summary
{summary}

## Problem Statement
{problem}

## Proposed Solution
{proposal}

## Alternatives Considered
{alternatives or "None"}

## Open Questions
- [ ] {question}

---
Status: Draft
```

**Step 4: Create Discussion**

```bash
gh api graphql \
  -f query='mutation($repoId: ID!, $catId: ID!, $title: String!, $body: String!) {
    createDiscussion(input: {repositoryId: $repoId, categoryId: $catId, title: $title, body: $body}) {
      discussion { url number }
    }
  }' \
  -f repoId="$REPO_ID" -f catId="$CATEGORY_ID" \
  -f title="[Spec] $TITLE" -f body="$BODY"
```

### Local Fallback

If Discussions unavailable:

```bash
mkdir -p .claude/specs
echo "$BODY" > .claude/specs/$(date +%Y-%m-%d)-{slug}.md
```

---

## /add-issue-comment [#] [text]

Add comments to Issues or Pull Requests.

```
/add-issue-comment 123 "Comment"
/add-issue-comment #123              # Interactive
/add-issue-comment                   # List recent issues first
```

### Workflow

**Step 1: Identify Target**

If number not provided, list recent issues:

```bash
gh issue list --limit 10 --json number,title,state \
  | jq -r '.[] | "#\(.number) [\(.state)] \(.title)"'
```

**Step 2: Get Content**

If comment not provided, ask user. Offer templates:

| Template | Content |
|----------|---------|
| LGTM | "LGTM! :+1:" |
| WIP | "Work in progress, will update soon." |
| Blocker | ":warning: Blocked by: {input}" |
| Done | "Completed in {commit/PR ref}" |

**Step 3: Post Comment**

```bash
gh issue comment $NUMBER --body "$COMMENT"
# For PRs:
gh pr comment $NUMBER --body "$COMMENT"
```

---

## /manage-labels [action] [#] [label]

Manage GitHub Issue/PR labels.

```
/manage-labels                           # List all labels
/manage-labels add 123 bug              # Add label
/manage-labels remove 123 bug           # Remove label
/manage-labels create "name" "ff0000"   # Create label
/manage-labels setup                    # Create standard labels
```

### Commands

```bash
# List
gh label list --json name,color,description

# Add/Remove
gh issue edit $NUMBER --add-label "$LABEL"
gh issue edit $NUMBER --remove-label "$LABEL"

# Create/Delete
gh label create "$NAME" --color "$COLOR" --description "$DESC"
gh label delete "$NAME" --yes
```

### Standard Labels

| Label | Color | Description |
|-------|-------|-------------|
| bug | d73a4a | Something isn't working |
| feature | 0075ca | New feature or request |
| docs | 0052cc | Documentation |
| chore | cccccc | Maintenance |
| research | 7057ff | Investigation |
| priority:high | b60205 | High priority |
| priority:low | c5def5 | Low priority |

---

## Error Handling

| Error | Action |
|-------|--------|
| No project found | "Run `/project-setup` to create one." |
| gh not authenticated | "Run `gh auth login` first." |
| Field/label not found | Use defaults, warn user |
| Issue not found | "Issue #{n} not found. Check the number." |
| No Ideas category | Create in General, suggest adding Ideas |
| Discussions disabled | Save spec to `.claude/specs/` |
| Empty required field | Prompt user for input |

## Notes

- Always set required fields (Status, Priority, Type) on new items
- XL items should prompt user to consider splitting
- Prefix spec titles with "[Spec]" for filtering
- Label colors are hex without # (e.g., "ff0000")
