---
name: code-reviewing
description: Comprehensive code review workflow with specialized roles. Use when "レビューして", "review", "セキュリティチェック", "security audit", "テストレビュー", "test quality", "Next.js review", or when checking code quality, security, or testing patterns.
allowed-tools: Read, Grep, Glob, Bash, WebSearch, WebFetch
---

# Code Reviewing Skill

Comprehensive code review workflow with specialized roles for different review types.

## When to Use

Automatically invoke when the user:
- Requests "review", "レビューして", "コードレビュー"
- Says "security review", "セキュリティ", "audit"
- Mentions "test review", "テストレビュー", "test quality"
- Asks for "Next.js review", "プロジェクトレビュー"

## Design Philosophy

**Check and report both "Do" and "Don't" rules**

- **Do**: Verify via Review Checklist in each role
- **Don't**: Detect via Anti-patterns to Detect in each role

## Architecture

- `SKILL.md` - This file (core workflow)
- `common/` - Reusable across projects (shareable/publishable)
  - `patterns/` - Generic patterns (drizzle-orm, better-auth, server-actions, etc.)
  - `criteria/` - Quality criteria (code-quality, security, testing)
  - `roles/` - Review role definitions (code, security, testing, nextjs)
  - `templates/` - Report templates
- `.claude/rules/` - Project-specific conventions (auto-loaded)
  - `tech-stack.md` - Versions and patterns
  - `lib-structure.md` - lib/ directory rules
  - `server-actions.md` - Server Action conventions
  - `shirokuma-annotations.md` - JSDoc annotations
  - `known-issues.md` - CVEs and bugs
  - `testing.md` - Test conventions

## Available Roles

| Role | Focus | Trigger |
|------|-------|---------|
| **code** | Quality, patterns, style | "review", "コードレビュー" |
| **code+annotation** | JSDoc annotations | "annotation review", "アノテーションレビュー" |
| **security** | OWASP, CVEs, auth | "security review", "セキュリティ" |
| **testing** | TDD, coverage, mocks | "test review", "テストレビュー" |
| **nextjs** | Framework, patterns | "Next.js review", "プロジェクト" |

## Workflow

```
Role Selection → Load Knowledge → Run Lints → Analyze Code → Generate Report → Save Report
```

**6 Steps**: Select Role → Load → **Lint** → Analyze → Report → Save

### 1. Role Selection

Based on user request, select appropriate role:

| Keyword | Role | Files to Load |
|---------|------|---------------|
| "review", "レビュー" | code | common/criteria/code-quality, common/criteria/coding-conventions, common/patterns/server-actions, common/patterns/drizzle-orm, common/patterns/jsdoc |
| "annotation", "アノテーション" | code+annotation | common/roles/code.md (+ rules: shirokuma-annotations.md auto-loaded) |
| "security", "セキュリティ" | security | common/criteria/security, common/patterns/better-auth (+ rules: known-issues.md auto-loaded) |
| "test", "テスト" | testing | common/criteria/testing, common/patterns/e2e-testing (+ rules: testing.md auto-loaded) |
| "Next.js", "nextjs" | nextjs | ALL common files |

### 2. Load Knowledge

Read required knowledge files based on role:

```
1. Auto-loaded: .claude/rules/*.md (based on file paths)
2. Role-specific: common/roles/{role}.md
3. Criteria: common/criteria/{relevant}.md
4. Patterns: common/patterns/{relevant}.md
```

**Note**: Project-specific rules are auto-loaded from `.claude/rules/` - no manual loading needed.

### 3. Run shirokuma-docs Lints (REQUIRED)

**Execute automated checks before manual review:**

```bash
# Test documentation (@testdoc, @skip-reason)
shirokuma-docs lint-tests -p . -f terminal

# Implementation-test coverage
shirokuma-docs lint-coverage -p . -f summary

# Code structure (Server Actions, annotations)
shirokuma-docs lint-code -p . -f terminal

# Project structure (directories, naming)
shirokuma-docs lint-structure -p . -f terminal

# Annotation consistency (@usedComponents, @screen)
shirokuma-docs lint-annotations -p . -f terminal
```

**Key rules to check:**

| Rule | Description |
|------|-------------|
| `skipped-test-report` | Reports `.skip` tests (ensure `@skip-reason` present) |
| `testdoc-required` | All tests need `@testdoc` |
| `lint-coverage` | Source files need corresponding tests |
| `annotation-required` | Server Actions need `@serverAction` |

See project-specific workflow documentation for detailed fix instructions.

### 4. Analyze Code

1. Read target files
2. Apply criteria from loaded knowledge
3. Check against known issues
4. Cross-reference with shirokuma-docs lint results
5. Identify violations and improvements

### 5. Generate Report

Use `common/templates/report.md` format:

1. Summary (include shirokuma-docs lint summary)
2. Critical Issues
3. Improvements
4. Best Practices
5. Recommendations

### 6. Save Report

**Create Discussion in Reports category:**

```bash
shirokuma-docs gh-discussions create \
  --category Reports \
  --title "[Review] {role}: {target}" \
  --body "$(cat report.md)"
```

Report the Discussion URL to the user.

> See `rules/output-destinations.md` for output destination policy.

## Role Details

### Code Review (`common/roles/code.md`)

Focus areas:
- TypeScript best practices
- Error handling
- Async patterns
- Coding conventions (naming, imports, structure)
- Code smells detection
- Documentation quality (JSDoc)

### Security Review (`common/roles/security.md`)

Focus areas:
- OWASP Top 10 2025
- Authentication/Authorization
- Input validation
- Injection prevention
- CVE awareness

### Test Review (`common/roles/testing.md`)

Focus areas:
- TDD compliance
- Test coverage
- Mock patterns
- E2E quality
- Anti-patterns

### Next.js Review (`common/roles/nextjs.md`)

Focus areas:
- App Router patterns
- Server/Client components
- Tailwind CSS v4
- shadcn/ui integration
- next-intl configuration

## Knowledge Update

When user requests `--update`:

1. Web search for latest:
   - Next.js releases and CVEs
   - React updates
   - Tailwind CSS changes
   - Better Auth updates
   - OWASP updates

2. Update relevant files:
   - `.claude/rules/tech-stack.md` - Versions
   - `.claude/rules/known-issues.md` - CVEs

## Progressive Disclosure

For token efficiency:

1. **Auto-loaded**: `.claude/rules/*.md` based on file paths being reviewed
2. **On Demand**: Load common/ files based on role/findings
3. **Minimal Output**: Summary first, details on request

## Quick Reference

```bash
# Code quality review
"review lib/actions/"

# Annotation consistency review
"annotation review components/"
"アノテーションレビュー components/"
"check usedComponents in nav-tags.tsx"

# Security review
"security review lib/actions/"

# Test review
"test review"

# Next.js project review
"Next.js review"

# Update knowledge base
"reviewer --update"
```

## Notes

- **Reports saved**: Create Discussion in Reports category (see `rules/output-destinations.md`)
- **Role-based**: Load only relevant knowledge files
- **Progressive**: Summary first, details on request
- **Updateable**: Use `--update` to refresh knowledge
- **Rules auto-loaded**: Project conventions from `.claude/rules/`

## Reference Documents

### Project Rules (Auto-loaded from .claude/rules/)

| Rule | Content | Auto-loads when |
|------|---------|-----------------|
| `tech-stack.md` | Versions, patterns | Always |
| `lib-structure.md` | lib/ directory rules | Reviewing lib/**/*.ts |
| `server-actions.md` | Server Action conventions | Reviewing lib/actions/** |
| `shirokuma-annotations.md` | JSDoc annotations | Reviewing tests, actions, pages |
| `known-issues.md` | CVEs and bugs | Always |
| `testing.md` | Test conventions | Reviewing *.test.ts |

### Common (Reusable)

| Document | Content | When to Read |
|----------|---------|--------------|
| [common/criteria/code-quality.md](common/criteria/code-quality.md) | Quality standards | Code review |
| [common/criteria/security.md](common/criteria/security.md) | Security checklist | Security review |
| [common/criteria/testing.md](common/criteria/testing.md) | Test quality | Test review |
| [nextjs-vibe-coding/common/patterns/tailwind-v4.md](../nextjs-vibe-coding/common/patterns/tailwind-v4.md) | CSS variable issues | Tailwind styling |
| [nextjs-vibe-coding/common/patterns/radix-ui-hydration.md](../nextjs-vibe-coding/common/patterns/radix-ui-hydration.md) | Hydration errors | Radix UI |
