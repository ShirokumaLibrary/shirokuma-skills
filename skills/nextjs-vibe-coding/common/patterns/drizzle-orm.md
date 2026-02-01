# Drizzle ORM Patterns

## Schema File Organization (Large Projects)

For large-scale projects, organize schemas by domain in separate files:

| File | Purpose |
|------|---------|
| `schema/index.ts` | Barrel exports + all relations |
| `schema/common.ts` | Shared column definitions |
| `schema/auth.ts` | users, sessions, accounts, verifications |
| `schema/content.ts` | posts, categories, tags, post_tags |
| `schema/comments.ts` | comments |
| `index.ts` | DB client + re-export schema |
| `constants.ts` | Shared constants |

### common.ts - Reusable Columns

```typescript
import { timestamp } from "drizzle-orm/pg-core"

export const timestamps = {
  createdAt: timestamp("created_at").notNull().defaultNow(),
  updatedAt: timestamp("updated_at").notNull().defaultNow(),
}
```

### auth.ts - Domain Tables

```typescript
import { pgTable, uuid, varchar, timestamp, text, boolean } from "drizzle-orm/pg-core"
import { timestamps } from "./common"

export const users = pgTable("user", {
  id: uuid("id").defaultRandom().primaryKey(),
  email: varchar("email", { length: 255 }).notNull().unique(),
  name: varchar("name", { length: 255 }),
  role: varchar("role", { length: 20 }).notNull().default("user"),
  emailVerified: timestamp("email_verified"),
  ...timestamps,
})

export const sessions = pgTable("session", {
  id: uuid("id").defaultRandom().primaryKey(),
  userId: uuid("user_id").notNull().references(() => users.id, { onDelete: "cascade" }),
  token: varchar("token", { length: 255 }).notNull().unique(),
  expiresAt: timestamp("expires_at").notNull(),
  ...timestamps,
})

// accounts, verifications tables...
```

### schema/index.ts - Barrel + Relations

```typescript
// Re-export all tables
export * from "./common"
export * from "./auth"
export * from "./content"
export * from "./comments"

// Define relations in ONE place to avoid circular dependencies
import { relations } from "drizzle-orm"
import { users, sessions, accounts } from "./auth"
import { posts, categories, tags, postTags } from "./content"
import { comments } from "./comments"

export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
  comments: many(comments),
  sessions: many(sessions),
}))

export const postsRelations = relations(posts, ({ one, many }) => ({
  author: one(users, { fields: [posts.authorId], references: [users.id] }),
  category: one(categories, { fields: [posts.categoryId], references: [categories.id] }),
  comments: many(comments),
  postTags: many(postTags),
}))

// ... other relations
```

### drizzle.config.ts

```typescript
export default defineConfig({
  dialect: "postgresql",
  schema: "./src/schema",  // Directory path for all files
  out: "./drizzle",
})
```

### Why Separate Files

- Domain boundaries are clear (Auth / Content / Comments)
- Reduces merge conflicts in team development
- Easier to find and modify related tables
- Scales better as project grows (20+ tables)

### Key Rules

1. Define relations in `schema/index.ts` only (avoids circular imports)
2. Export everything through barrel file
3. Keep common patterns (timestamps, soft delete) in `common.ts`
4. Single file is fine for projects under 20 tables / 500 lines

## Query with Relations

```typescript
const postsWithAuthor = await db
  .select({
    id: posts.id,
    title: posts.title,
    authorName: users.name,
  })
  .from(posts)
  .leftJoin(users, eq(posts.authorId, users.id))
  .where(eq(posts.status, "published"))
```

## Pagination

```typescript
const PAGE_SIZE = 20

export async function getPaginated(page: number = 1) {
  const offset = (page - 1) * PAGE_SIZE

  const [items, countResult] = await Promise.all([
    db.select().from(features).orderBy(desc(features.createdAt)).limit(PAGE_SIZE).offset(offset),
    db.select({ count: count() }).from(features),
  ])

  return {
    items,
    pagination: {
      page,
      pageSize: PAGE_SIZE,
      total: countResult[0]?.count ?? 0,
      totalPages: Math.ceil((countResult[0]?.count ?? 0) / PAGE_SIZE),
    },
  }
}
```

## Search with ILIKE (Escape Wildcards)

```typescript
function escapeLikePattern(query: string): string {
  return query.replace(/[%_\\]/g, "\\$&")
}

const pattern = `%${escapeLikePattern(userInput)}%`
db.select().from(posts).where(ilike(posts.title, pattern))
```

## Schema Documentation (shirokuma-docs)

カラムとインデックスの説明はインラインJSDocコメントで記述する。shirokuma-docsが自動抽出してドキュメント化。

### Column Comments

カラム定義の直前にJSDocコメントを配置：

```typescript
export const organizations = pgTable("organizations", {
  /** 組織ID（UUID） */
  id,
  /** 組織名（表示名） */
  name: text("name").notNull(),
  /** 組織スラッグ（URLに使用、一意制約あり） */
  slug: text("slug").notNull().unique(),
  /** 組織の説明文 */
  description: text("description"),
  ...timestamps,
})
```

### Index Comments

インデックス定義の直前にJSDocコメントを配置：

```typescript
export const organizationMembers = pgTable(
  "organization_members",
  {
    /** メンバーシップID */
    id,
    /** 組織ID */
    organizationId: uuid("organization_id").notNull(),
    /** ユーザーID */
    userId: text("user_id").notNull(),
    ...timestamps,
  },
  (table) => [
    /** 組織とユーザーの組み合わせで一意制約（重複メンバーシップを防止） */
    uniqueIndex("org_members_org_user_idx").on(table.organizationId, table.userId),
  ]
)
```

### ❌ Don't Use

`@columns` や `@indexes` JSDocタグは使用しない（インラインコメントと重複するため）：

```typescript
// ❌ BAD - Don't use these tags
/**
 * @columns
 *   - id: 組織ID
 *   - name: 組織名
 * @indexes
 *   - org_slug_idx: スラッグの一意インデックス
 */

// ✅ GOOD - Use inline comments instead (as shown above)
```
