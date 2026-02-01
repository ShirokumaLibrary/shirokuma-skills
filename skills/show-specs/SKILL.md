---
name: show-specs
description: Spec Discussion list display skill from Ideas category. Use when "/show-specs", "仕様一覧", "設計ドキュメント一覧", "list specs", "RFC一覧"
disable-model-invocation: true
allowed-tools: Bash
---

# Show Specs

GitHub Discussions の仕様/設計ドキュメントを一覧表示します。

## Usage

```
/show-specs                  # All specs
/show-specs --recent         # Last 5 specs
/show-specs "keyword"        # Search specs
```

## Workflow

### Step 1: Get Repository Info

```bash
OWNER=$(gh repo view --json owner -q '.owner.login')
REPO=$(gh repo view --json name -q '.name')
```

### Step 2: Fetch Specs from Ideas Category

```bash
gh api graphql -f query='{
  repository(owner: "'$OWNER'", name: "'$REPO'") {
    discussions(first: 20, categoryId: "'$IDEAS_CATEGORY_ID'", orderBy: {field: CREATED_AT, direction: DESC}) {
      nodes {
        number
        title
        createdAt
        author { login }
        comments { totalCount }
        url
      }
    }
  }
}'
```

### Step 3: Display Results

```markdown
## Specifications

**Category:** Ideas
**Total:** {count} specs

| # | Title | Author | Comments | Created |
|---|-------|--------|----------|---------|
| #10 | [Spec] Authentication Flow | @user | 5 | 1w ago |
| #8 | [Spec] API Design | @user | 3 | 2w ago |

---

**Quick actions:**
- View spec: `gh discussion view 10`
- Create new: `/create-spec`
- Comment: `gh discussion comment 10 --body "..."`
```

## Filtering

| Filter | Description |
|--------|-------------|
| `[Spec]` prefix | Only show items with [Spec] in title |
| `[RFC]` prefix | Only show RFCs |
| `[Draft]` prefix | Only show drafts |

## Status Indicators

Parse spec body for status:

```
📝 Draft - Under development
🔍 Review - Ready for feedback
✅ Approved - Accepted
❌ Rejected - Not proceeding
🚧 Implementing - In progress
```

## Error Handling

| Error | Action |
|-------|--------|
| No Ideas category | "No Ideas category found. Specs are stored in Ideas." |
| No specs found | "No specifications found. Create one with `/create-spec`" |
| Discussions disabled | "Discussions not enabled for this repository." |

## Notes

- Specs are stored in "Ideas" category by convention
- [Spec] prefix helps identify specification documents
- Comments count indicates engagement level
