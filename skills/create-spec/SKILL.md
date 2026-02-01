---
name: create-spec
description: Create spec Discussions for design documents. Use when "/create-spec", "仕様作成", "設計ドキュメント", "create spec", "design doc", "RFC"
disable-model-invocation: true
allowed-tools: Bash, AskUserQuestion, Read, Write
---

# Create Spec

GitHub Discussions に仕様/設計ドキュメントを作成します。

## Usage

```
/create-spec "機能名"
/create-spec                    # Interactive mode
```

## Workflow

### Step 1: Get Repository Info

```bash
OWNER=$(gh repo view --json owner -q '.owner.login')
REPO=$(gh repo view --json name -q '.name')

# Get Ideas category ID (for specs)
CATEGORY_ID=$(gh api graphql -f query='{
  repository(owner: "'$OWNER'", name: "'$REPO'") {
    discussionCategories(first: 10) { nodes { id name } }
  }
}' | jq -r '.data.repository.discussionCategories.nodes[] | select(.name == "Ideas") | .id')
```

### Step 2: Gather Spec Details

Ask user for:

1. **Title**: Feature/spec name
2. **Summary**: One-line description
3. **Problem**: What problem does this solve?
4. **Proposal**: How will it work?
5. **Alternatives**: Other approaches considered (optional)

### Step 3: Generate Spec Body

```markdown
## Summary
{summary}

## Problem Statement
{problem description}

## Proposed Solution
{detailed proposal}

### API/Interface
{if applicable}

### Data Model
{if applicable}

## Alternatives Considered
{alternatives or "None"}

## Open Questions
- [ ] {question 1}

## Related
- Project Item: {link if applicable}
- Related Specs: {links}

---
Status: Draft
Author: @{username}
Created: {date}
```

### Step 4: Create Discussion

```bash
gh api graphql \
  -f query='mutation($repoId: ID!, $catId: ID!, $title: String!, $body: String!) {
    createDiscussion(input: {repositoryId: $repoId, categoryId: $catId, title: $title, body: $body}) {
      discussion { url number }
    }
  }' \
  -f repoId="$REPO_ID" \
  -f catId="$CATEGORY_ID" \
  -f title="[Spec] $TITLE" \
  -f body="$BODY"
```

### Step 5: Display Result

```markdown
## Spec Created

**Title:** [Spec] {title}
**Category:** Ideas
**URL:** {discussion_url}

---
Share this spec for review and feedback.
```

## Spec Categories

| Category | Use For |
|----------|---------|
| Ideas | Feature specs, RFCs |
| Q&A | Technical questions |
| General | General discussions |

## Error Handling

| Error | Action |
|-------|--------|
| No Ideas category | Create in General, suggest adding Ideas category |
| Discussions disabled | Save to local file `.claude/specs/` |
| Empty fields | Prompt user for required fields |

## Local Fallback

If Discussions unavailable:

```bash
mkdir -p .claude/specs
echo "$BODY" > .claude/specs/$(date +%Y-%m-%d)-{slug}.md
```

## Notes

- Prefix title with "[Spec]" for easy filtering
- Link to related Project items when possible
- Keep specs focused on one feature/change
