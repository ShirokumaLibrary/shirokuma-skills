# Project Items Rule

## Required Fields

Every project item MUST have:

| Field | Required | Options |
|-------|----------|---------|
| Status | Yes | See workflow below |
| Priority | Yes | Critical / High / Medium / Low |
| Type | Yes | Feature / Bug / Chore / Docs / Research |
| Size | Recommended | XS / S / M / L / XL |

## Status Workflow

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

## Size Estimation

| Size | Time | Example |
|------|------|---------|
| XS | ~1h | Typo fix, config change |
| S | ~4h | Small feature, bug fix |
| M | ~1d | Medium feature |
| L | ~3d | Large feature |
| XL | 3d+ | Epic (should be split) |

## Body Template

```markdown
## 概要
{What this item does}

## 背景
{Why this is needed - optional}

## タスク
- [ ] Task 1
- [ ] Task 2

## Deliverable
{What "done" looks like}

## 優先度
{Priority justification - optional}
```

## Creating Items

When creating new items:

1. Set all required fields immediately
2. Use the body template
3. XL items should be split into smaller items
4. Link related items in body if applicable
