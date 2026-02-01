---
paths:
  - ".claude/skills/**/*.md"
  - ".claude/agents/**/*.md"
  - ".claude/output-styles/**/*.md"
  - ".claude/commands/**/*.md"
---

# Claude Config Authoring Rules

## Naming Convention

**Format**: `gerund-form` (verb + -ing), lowercase, hyphens, max 64 chars

```yaml
# Good
name: processing-pdfs
name: managing-agents

# Bad
name: PDF-Processor    # Not gerund, uppercase
name: helper           # Too vague
```

## Description (CRITICAL)

**Template**: `[What it does]. Use when [triggers].`

**Requirements**:
- Max 1024 characters
- Third person voice
- Include WHAT (capability) and WHEN (triggers)
- NO angle brackets (`<example>` forbidden)

```yaml
# Good
description: Extract text from PDF files. Use when working with PDFs or user mentions "PDF", "PDF処理".

# Bad
description: Helps with documents  # Too vague, no triggers
```

## File Size Limits

- `SKILL.md` / `AGENT.md`: Under 500 lines
- Challenge every sentence for necessity
- Move detailed content to reference files

## Language Guidelines

- **Primary**: English (model comprehension)
- **Triggers**: Include Japanese in description for bilingual users
- **Avoid**: Heavy table formatting (token-inefficient)

## File Structure

```
skill-name/
├── SKILL.md        # Required, <500 lines
├── reference.md    # Optional, detailed specs
├── examples.md     # Optional, I/O examples
└── scripts/        # Optional, automation
```

## Anti-Patterns

- Deep reference chains (keep ONE level from SKILL.md)
- Windows backslashes (use forward slashes only)
- Too many options without defaults
- Vague naming without specific triggers

## Validation Checklist

Before committing:
- [ ] Name: gerund form, lowercase
- [ ] Description: triggers, third person, no `<>`
- [ ] SKILL.md: under 500 lines
- [ ] Frontmatter: valid YAML (spaces, not tabs)
- [ ] Paths: forward slashes only
