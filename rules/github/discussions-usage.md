# GitHub Discussions Usage

## Purpose

Discussions store human-readable knowledge; Rules store AI-readable extracts.

| Layer | Audience | Language | Content |
|-------|----------|----------|---------|
| Discussions | Human | User's language | Context, rationale, details |
| Rules/Skills | AI | English | Concise patterns, commands |

## Categories

| Category | When to Use |
|----------|-------------|
| Handovers | Session end - create via `/end-session` |
| ADR | Architecture decisions confirmed |
| Knowledge | Patterns/solutions confirmed |
| Research | Investigation needed |

## Workflow

```
Research → ADR (if decision) → Knowledge → Rule extract
```

## Cross-Reference

- Discussions share number space with Issues (#1, #2, ...)
- Reference in commits: "See Discussion #30"
- Add to Projects for tracking

## AI Behavior

1. **Read**: Check Discussions for context when researching
2. **Write**: Create Discussions for significant findings
3. **Extract**: Propose Rule when pattern is confirmed
4. **Reference**: Link Discussion # in Rule comments

## Title Formats

| Category | Format |
|----------|--------|
| Handovers | `YYYY-MM-DD - {summary}` |
| ADR | `ADR-{NNN}: {title}` |
| Knowledge | `{Topic Name}` |
| Research | `[Research] {topic}` |
