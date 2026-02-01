---
name: codebase-rule-discovery
description: Analyzes TypeScript applications to discover patterns and propose coding conventions for shirokuma-docs lint rules. Use when "ルール発見", "rule discovery", "規約提案", "convention proposal", "パターン分析", or when investigating codebases to extract patterns or propose new conventions.
allowed-tools: Read, Grep, Glob, Bash
---

# Codebase Rule Discovery

Analyzes TypeScript applications in the monorepo for two purposes:
1. **Pattern Discovery**: Extract existing patterns across apps
2. **Convention Proposal**: Propose new conventions that enable mechanical checks

## When to Use

- User requests "ルール発見", "rule discovery"
- User says "パターン分析", "pattern analysis"
- User mentions "規約提案", "convention proposal"
- User asks "もっとチェックできるようにしたい"
- User wants to "統一感を上げたい", "機械的チェックを増やしたい"

## Two Modes

### Mode 1: Pattern Discovery (既存パターン発見)

**Goal**: Find what patterns already exist across apps

```
現状のコードを分析 → 共通パターン発見 → ルール化提案
```

### Mode 2: Convention Proposal (規約提案)

**Goal**: Propose conventions that ENABLE more mechanical checks

```
チェック可能性分析 → 規約提案 → 採用後にルール実装
```

**Key Question**: 「どう書けば機械的にチェックできるか？」

## Target Applications

| App | Path | Description |
|-----|------|-------------|
| Blog CMS (admin) | `nextjs-tdd-blog-cms/apps/admin/` | CMS管理画面 |
| Blog CMS (public) | `nextjs-tdd-blog-cms/apps/public/` | 公開ブログ |
| shirokuma-docs | `shirokuma-docs/src/` | ドキュメント生成CLI |

## Workflow: Pattern Discovery

### Step 1-7: (既存のワークフロー)

See [workflows/analyze-codebase.md](workflows/analyze-codebase.md)

## Workflow: Convention Proposal

### Step 1: Identify Check Opportunities

Analyze what COULD be checked if conventions existed:

| Category | Current State | If Standardized |
|----------|--------------|-----------------|
| File placement | 混在 | ドメイン別配置チェック可能 |
| Naming | 一部一貫 | 自動リネーム提案可能 |
| i18n keys | 自由形式 | キー形式検証可能 |

### Step 2: Analyze Current Structure

```bash
# File structure analysis
find apps/ -name "*.ts" -o -name "*.tsx" | head -100

# Directory patterns
ls -la apps/admin/lib/
ls -la apps/public/lib/

# Naming conventions
find apps/ -name "*.tsx" | xargs basename -a | sort | uniq -c | sort -rn
```

### Step 3: Propose Conventions

For each opportunity, document:

1. **Current State**: How it's done now (with variations)
2. **Proposed Convention**: Specific rule
3. **Migration Cost**: How much code needs changing
4. **Check Enabled**: What lint rule becomes possible
5. **Benefits**: Why it's worth standardizing

### Step 4: Generate Convention Proposal

Use [templates/convention-proposal.md](templates/convention-proposal.md)

### Step 5: Save Report

Save to GitHub Discussions (Research category):

```bash
shirokuma-docs gh-discussions create --category Research --title "[Research] convention-{category}" --body "..."
```

## Convention Categories

### 1. File Placement Conventions

| Area | Convention | Enables |
|------|------------|---------|
| Server Actions | `lib/actions/{domain}.ts` | ドメイン完全性チェック |
| Components | `components/{Domain}/` | コンポーネント依存チェック |
| Hooks | `hooks/use{Name}.ts` | Hook命名チェック |
| Types | `types/{domain}.ts` | 型定義重複チェック |

### 2. Naming Conventions

| Target | Convention | Example |
|--------|------------|---------|
| Server Action files | `{domain}-actions.ts` | `post-actions.ts` |
| Component files | `{Name}.tsx` (PascalCase) | `PostCard.tsx` |
| Hook files | `use{Name}.ts` | `useAuth.ts` |
| Test files | `{name}.test.ts` | `post-actions.test.ts` |

### 3. Code Structure Conventions

| Area | Convention | Enables |
|------|------------|---------|
| Server Action order | 認証→CSRF→検証→処理 | 順序チェック |
| Export style | Named exports優先 | 未使用export検出精度向上 |
| i18n keys | `{domain}.{action}.{element}` | キー形式チェック |

### 4. Annotation Conventions

| Tag | Required For | Enables |
|-----|-------------|---------|
| `@serverAction` | Server Actions | 自動ドキュメント生成 |
| `@screen` | Page components | 画面一覧生成 |
| `@usedComponents` | Screens | 依存グラフ生成 |

## Existing Rules (Reference)

| Rule | Status |
|------|--------|
| server-action-structure | Implemented |
| annotation-required | Implemented |
| testdoc-* | Implemented |

Check `shirokuma-docs/src/lint/rules/` for current implementations.

## Quick Reference

```bash
# Pattern discovery (Mode 1)
"discover patterns in blog-cms"

# Convention proposal (Mode 2)
"propose conventions for better checking"
"どうすればもっとチェックできる？"

# Specific areas
"propose file placement conventions"
"propose naming conventions"
"propose i18n key conventions"
```

## Related Resources

- [patterns/discovery-categories.md](patterns/discovery-categories.md) - What to look for
- [templates/rule-proposal.md](templates/rule-proposal.md) - Rule proposal format
- [templates/convention-proposal.md](templates/convention-proposal.md) - Convention proposal format
- [workflows/analyze-codebase.md](workflows/analyze-codebase.md) - Detailed workflow

## Output

Convention proposals should answer:

1. **What**: 具体的な規約内容
2. **Why**: なぜこの規約が必要か
3. **Check**: どんなチェックが可能になるか
4. **Migration**: 既存コードの移行コスト
5. **Priority**: P0/P1/P2
