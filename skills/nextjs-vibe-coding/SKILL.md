---
name: nextjs-vibe-coding
description: TDD implementation workflow for Next.js projects. Use when "実装して", "機能追加", "コンポーネント作成", "ページ作成", "機能を作って", implementing features, creating components, or building pages.
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, WebSearch, WebFetch
---

# Next.js Vibe Coding Skill

Test-first implementation workflow for Next.js projects with modern tech stack.

## When to Use

Automatically invoke when the user:
- Requests "implement feature", "機能追加", "実装して"
- Wants "create component", "コンポーネント作成"
- Says "build page", "ページ作成", "画面を作って"
- Mentions TDD implementation, test-first development
- Describes a feature in natural language (vibe coding)

## Core Philosophy

**Vibe Coding**: Transform natural language descriptions into working code
**Test-First**: ALWAYS write tests BEFORE implementation - NO EXCEPTIONS

```
User Request → Understand → Plan → WRITE TESTS → Verify Tests Exist → Implement → Run Tests → Verify Docs → Refine → Report → Portal
```

**10 Steps**: Understand → Plan → **Write Tests** → **Verify Tests** → Implement → Run Tests → **Verify Docs** → Refine → Report → **Portal** (optional)

> **CRITICAL RULE**: You MUST NOT proceed to Step 5 (Implement) until test files are created and verified. If you skip tests, you are violating the core contract of this skill.

## Architecture

- `SKILL.md` - This file (core workflow)
- `common/` - Reusable across projects (shareable/publishable)
  - `patterns/` - Generic patterns (testing, drizzle-orm, better-auth, etc.)
  - `reference/` - Checklists, large-scale rules
  - `templates/` - Code templates for Server Actions, components, pages
- `.claude/rules/` - Project-specific conventions (auto-loaded)
  - `tech-stack.md` - Versions and patterns
  - `lib-structure.md` - lib/ directory rules
  - `server-actions.md` - Server Action conventions
  - `shirokuma-annotations.md` - JSDoc annotations
  - `testing.md` - Test conventions

## Before Starting

1. Rules in `.claude/rules/` are auto-loaded based on file paths
2. Check project's `CLAUDE.md` for project-specific conventions
3. Use templates from `common/templates/` directory as starting points

## Workflow

### Step 1: Understand Request

Parse the user's natural language request:

- **What**: Feature/component/page to build
- **Where**: Which app and path
- **Why**: User-facing behavior expected
- **Constraints**: Performance, accessibility, i18n requirements

If unclear, ask ONE focused question:
```
To implement this feature, I need to clarify:
- Should this be a Server or Client Component?
- Which app should this go in?
- Should this support i18n (Japanese/English)?
```

### Step 2: Plan Implementation

Create a checklist:

```markdown
## Implementation Plan

### Files to Create
- [ ] `lib/actions/feature.ts` - Server Actions
- [ ] `app/[locale]/(dashboard)/feature/page.tsx` - Page
- [ ] `components/feature-form.tsx` - Form component
- [ ] `__tests__/lib/actions/feature.test.ts` - Action tests
- [ ] `__tests__/components/feature-form.test.tsx` - Component tests

### Files to Modify
- [ ] `messages/ja/*.json` - Japanese translations
- [ ] `messages/en/*.json` - English translations

### Dependencies (if needed)
- [ ] `pnpm add package-name`
- [ ] `npx shadcn@latest add component`
```

### Step 3: Write Tests First (MANDATORY)

**THIS STEP IS MANDATORY - DO NOT SKIP**

Create test files BEFORE any implementation code:

1. **Read templates first**:
   ```bash
   cat .claude/skills/nextjs-vibe-coding/common/templates/server-action.test.ts.template
   cat .claude/skills/nextjs-vibe-coding/common/templates/component.test.tsx.template
   ```

2. **Create test files using templates**:
   - `__tests__/lib/actions/{{name}}.test.ts` - Server Action tests
   - `__tests__/components/{{name}}-form.test.tsx` - Component tests

3. **Add @testdoc comments (REQUIRED)**:
   Each test MUST have a JSDoc comment with Japanese description:

   ```typescript
   /**
    * @testdoc 新しいユーザーを作成できる
    * @purpose ユーザー作成APIの正常系確認
    * @precondition 有効なユーザーデータが提供されている
    * @expected ユーザーがDBに保存され、IDが返される
    */
   it("should create a new user", async () => {
     // test implementation
   });
   ```

4. **Minimum test coverage required**:
   - Server Actions: Create, Read (list + single), Update, Delete
   - Components: Render, Form submission, Validation errors, Loading states

See [common/patterns/testing.md](common/patterns/testing.md) for mock setup.

### Step 4: Verify Tests Exist (GATE)

**CHECKPOINT - DO NOT PROCEED WITHOUT PASSING THIS GATE**

Before implementing, verify test files exist:

```bash
# Verify test files were created
ls -la __tests__/lib/actions/{{name}}.test.ts
ls -la __tests__/components/{{name}}-form.test.tsx
```

**If test files do not exist, GO BACK TO STEP 3.**

Only after confirming test files exist, proceed to implementation.

### Step 5: Implement

Use templates from `common/templates/` directory:
- `server-action.ts.template` - Server Action implementation
- `form-component.tsx.template` - Form component
- `page-list.tsx.template` - List page
- `page-new.tsx.template` - Create page
- `page-edit.tsx.template` - Edit page
- `delete-button.tsx.template` - Delete button with dialog

See [common/patterns/code-patterns.md](common/patterns/code-patterns.md) for tech-specific patterns.

### Step 6: Run Tests (REQUIRED)

**ALL tests must pass before completing**

```bash
# Lint & Type Check
pnpm --filter {app} lint
pnpm --filter {app} tsc --noEmit

# Run Unit Tests - MUST PASS
pnpm --filter {app} test

# E2E if applicable
pnpm test:e2e --grep "feature"
```

**If tests fail:**
1. Fix the implementation (not the tests)
2. Re-run tests until all pass
3. Only then proceed to Step 6.5

### Step 6.5: shirokuma-docs Verification (REQUIRED)

**Verify test documentation quality before completing**

```bash
# Test documentation lint (@testdoc, @skip-reason)
shirokuma-docs lint-tests -p . -f terminal

# Implementation-test coverage check
shirokuma-docs lint-coverage -p . -f summary

# Code structure check (Server Actions, annotations)
shirokuma-docs lint-code -p . -f terminal
```

**Required checks:**

| Check | Pass Criteria | Fix |
|-------|---------------|-----|
| `skipped-test-report` | All `.skip` have `@skip-reason` | Add `@skip-reason` annotation |
| `testdoc-required` | All tests have `@testdoc` | Add Japanese description |
| `lint-coverage` | New files have tests | Create test or add `@skip-test` |

**If issues found:**
1. Add missing `@testdoc` comments
2. Add `@skip-reason` for any `.skip` tests
3. Re-run lint commands until clean
4. Only then proceed to Step 7

### Step 7: Refine

Based on test results and lint feedback:
1. Add edge case tests
2. Improve UX (loading states, error handling)
3. Optimize (reduce re-renders, improve queries)
4. Update documentation if needed

### Step 8: Generate Report

**Create Discussion in Reports category:**

1. Write report with structure from [reference/report-template.md](reference/report-template.md)
2. Create Discussion:
   ```bash
   shirokuma-docs gh-discussions create \
     --category Reports \
     --title "[Implementation] {feature-name}" \
     --body "$(cat report.md)"
   ```
3. Report the Discussion URL to the user

> See `rules/output-destinations.md` for output destination policy.

### Step 9: Update Portal (For Significant Changes)

**When to run**: After significant implementations (new features, multiple files, architectural changes)

```bash
# Build documentation portal
shirokuma-docs portal -p . -o docs/portal

# Or use the shirokuma-md skill
/shirokuma-md build
```

**Triggers for portal update**:
- New Server Actions added
- New screens/pages created
- Database schema changes
- New components with `@usedComponents` annotations

**Skip if**: Minor fixes, single file changes, test-only updates

## Reference Documents

### Project Rules (Auto-loaded from .claude/rules/)

| Rule | Content | Auto-loads when |
|------|---------|-----------------|
| `tech-stack.md` | Versions, patterns | Always |
| `lib-structure.md` | lib/ directory rules | Editing lib/**/*.ts |
| `server-actions.md` | Server Action conventions | Editing lib/actions/** |
| `shirokuma-annotations.md` | JSDoc annotations | Editing tests, actions, pages |
| `testing.md` | Test conventions | Editing *.test.ts |

### Common (Reusable)

| Document | Content | When to Read |
|----------|---------|--------------|
| [common/reference/checklists.md](common/reference/checklists.md) | Quality checklists | After implementation |
| [common/templates/README.md](common/templates/README.md) | Template list | When generating code |
| [common/reference/large-scale.md](common/reference/large-scale.md) | File split rules | Large features |
| [common/patterns/tailwind-v4.md](common/patterns/tailwind-v4.md) | CSS variable issues | Tailwind styling |
| [common/patterns/radix-ui-hydration.md](common/patterns/radix-ui-hydration.md) | Hydration errors | Radix UI |

## Quick Commands

```bash
# Lint & Format (推奨: 一括修正)
pnpm --filter {app} fix          # ESLint + Prettier 一括修正

# Lint & Type Check
pnpm --filter {app} lint         # ESLintチェック
pnpm --filter {app} lint:fix     # ESLint自動修正
pnpm --filter {app} tsc --noEmit # 型チェック

# Format
pnpm --filter {app} format       # Prettier整形
pnpm --filter {app} format:check # Prettier差分確認

# Test
pnpm --filter {app} test
pnpm --filter {app} test --watch

# Build
pnpm --filter {app} build

# Dev
pnpm dev:{app}
```

## Notes

- **TESTS ARE NOT OPTIONAL** - No exceptions, no excuses
- **REPORTS ARE REQUIRED** - Create Discussion in Reports category (see `rules/output-destinations.md`)
- **CONVENTIONS ARE MANDATORY** - Rules in `.claude/rules/` are auto-loaded
- Use templates as starting points, customize as needed
- If you cannot write tests, explain why and stop
