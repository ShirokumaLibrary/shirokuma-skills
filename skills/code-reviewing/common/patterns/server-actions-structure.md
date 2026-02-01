# Server Actions Module Structure Standard

Related: [server-actions.md](./server-actions.md) (security patterns), [jsdoc.md](./jsdoc.md)

## Purpose

Server Actions モジュールの **構造と JSDoc** の標準ルール。
shirokuma-docs による機械的チェックと、レビューの効率化を目的とする。

---

## File Naming Convention

```
lib/actions/
├── types.ts              # 共通型（ActionResult, PaginatedResult）
├── {module}.ts           # Server Actions（メインファイル）
├── {module}-types.ts     # モジュール固有型+定数
└── {module}-utils.ts     # ヘルパー関数（必要時のみ）
```

| ファイル | 役割 | 必須タグ |
|---------|------|---------|
| `{module}.ts` | Server Actions | `@serverAction`, `@feature` |
| `{module}-types.ts` | 型定義+定数 | `@skip-test` |
| `{module}-utils.ts` | ヘルパー関数 | なし（テスト対象） |

---

## Module Structure Template

```typescript
"use server"

/**
 * [モジュール名]サーバーアクション / [Module Name] Server Actions
 *
 * [1-2行の概要説明]
 *
 * @serverAction
 * @feature [FeatureName]
 * @layer Application - Server Actions
 *
 * @usedInScreen [Screen1], [Screen2]
 * @usedComponents [Component1], [Component2]
 *
 * @dbTables [table1], [table2]
 * @dbOperations SELECT, INSERT, UPDATE, DELETE
 *
 * @authPattern
 *   1. verifyAuth() - Read operations
 *   2. verifyAuthMutation() - Write operations
 *
 * @category Server Actions - [カテゴリ名]
 */

// ============================================================
// Imports
// ============================================================
import { revalidatePath } from "next/cache"
import { eq, desc, and } from "drizzle-orm"
import { db, table1, table2 } from "@repo/database"
import { verifyAuth, verifyAuthMutation } from "@/lib/auth-utils"
import { z } from "zod"
import type { ActionResult } from "./types"
import { CONSTANTS, type ModuleType } from "./{module}-types"

// ============================================================
// Validation Schemas
// ============================================================

/**
 * [スキーマ名]バリデーションスキーマ
 *
 * @validation [SchemaName]
 */
const CreateSchema = z.object({
  name: z.string().min(1).max(100),
  // ...
})

// ============================================================
// Server Actions (Read)
// ============================================================

/**
 * [関数説明] / [Function description]
 *
 * @description [詳細説明]
 *
 * @serverAction
 * @feature [FeatureName]
 * @dbTables [tables]
 *
 * @param id - [パラメータ説明]
 * @returns [戻り値説明]
 * @throws 認証エラー
 *
 * @example
 * ```ts
 * const result = await getItem("uuid")
 * ```
 */
export async function getItem(id: string): Promise<ItemType | null> {
  await verifyAuth()
  // implementation
}

// ============================================================
// Server Actions (Write)
// ============================================================

/**
 * [関数説明] / [Function description]
 *
 * @description [詳細説明]
 *
 * @serverAction
 * @feature [FeatureName]
 * @dbTables [tables]
 *
 * @param formData - フォームデータ
 * @returns 成功: `{ success: true }` / 失敗: `{ success: false, error }`
 *
 * @example
 * ```tsx
 * <form action={createItem}>
 *   <input name="name" />
 * </form>
 * ```
 */
export async function createItem(formData: FormData): Promise<ActionResult> {
  await verifyAuthMutation()
  // implementation
}
```

---

## Types File Template

```typescript
/**
 * [モジュール名]型定義 / [Module Name] Types
 *
 * [モジュール名]関連の型定義と定数
 *
 * @skip-test 型定義のみ - ランタイムロジックなし
 */
import type { table } from "@repo/database"

// ============================================================
// Constants
// ============================================================

/**
 * [定数名]の選択肢
 */
export const STATUSES = ["active", "inactive", "archived"] as const

/**
 * [定数名]の選択肢
 */
export const TYPES = ["type1", "type2", "type3"] as const

// ============================================================
// Types
// ============================================================

/**
 * [型名] - ステータス
 */
export type Status = (typeof STATUSES)[number]

/**
 * [型名] - 基本型（スキーマ推論）
 */
export type Item = typeof table.$inferSelect

/**
 * [型名] - 詳細情報付き
 */
export type ItemWithDetails = Item & {
  relation1: Relation1Type
  relation2?: Relation2Type
}

/**
 * [型名] - 新規作成用
 */
export type NewItem = typeof table.$inferInsert
```

---

## Required JSDoc Tags

### File Header (モジュールヘッダー)

| タグ | 必須 | 説明 |
|-----|-----|------|
| `@serverAction` | Yes | Server Action モジュールを示す |
| `@feature` | Yes | 機能分類（feature-map 用） |
| `@dbTables` | Yes | 使用するDBテーブル |
| `@category` | No | ドキュメント分類 |
| `@usedInScreen` | No | 使用される画面 |
| `@usedComponents` | No | 使用されるコンポーネント |

### Function JSDoc

| タグ | 必須 | 説明 |
|-----|-----|------|
| `@serverAction` | Yes | Server Action 関数を示す |
| `@feature` | Yes | 機能分類 |
| `@param` | If params | パラメータ説明 |
| `@returns` | Yes | 戻り値説明 |
| `@description` | No | 詳細説明 |
| `@example` | No | 使用例 |
| `@throws` | No | 例外説明 |

---

## Section Separators

セクション区切りコメントを使用して構造を明確化：

```typescript
// ============================================================
// [Section Name]
// ============================================================
```

**推奨セクション順序**:
1. Imports
2. Validation Schemas
3. Helper Functions (internal, not exported)
4. Server Actions (Read)
5. Server Actions (Write)

---

## Validation Checklist

### File Structure
- [ ] `"use server"` がファイル先頭にある
- [ ] モジュールヘッダー JSDoc がある
- [ ] `@serverAction` タグがモジュールヘッダーにある
- [ ] `@feature` タグがモジュールヘッダーにある
- [ ] `@dbTables` タグがモジュールヘッダーにある
- [ ] セクション区切りコメントがある

### Function Documentation
- [ ] 全 public 関数に JSDoc がある
- [ ] 全 public 関数に `@serverAction` タグがある
- [ ] 全 public 関数に `@feature` タグがある
- [ ] 全 public 関数に `@returns` タグがある
- [ ] パラメータがあれば `@param` タグがある

### Types File
- [ ] `*-types.ts` ファイル名になっている
- [ ] `@skip-test` タグがヘッダーにある
- [ ] 定数と型が分離されている

---

## shirokuma-docs Integration

このルールは shirokuma-docs の以下の機能で活用される：

1. **feature-map**: `@serverAction`, `@feature` タグで自動分類
2. **details**: モジュール詳細ページに Types, Utilities セクション表示
3. **lint-code** (予定): 構造・タグの機械的チェック

### lint-code 検証ルール (予定)

```yaml
# shirokuma-docs.config.yaml
lintCode:
  serverActions:
    filePattern: "apps/*/lib/actions/*.ts"
    excludePattern: "*-types.ts"
    requiredFileHeader:
      - "@serverAction"
      - "@feature"
      - "@dbTables"
    requiredFunctionTags:
      - "@serverAction"
      - "@feature"
      - "@returns"
    sectionSeparators: true
```
