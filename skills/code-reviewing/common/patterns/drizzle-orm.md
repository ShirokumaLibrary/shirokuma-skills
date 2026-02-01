# Drizzle ORM Patterns

Related: [security.md](../criteria/security.md) (A05 Injection), [code-quality.md](../criteria/code-quality.md)

## Query Patterns

### Select with Relations

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

### Pagination

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

### Search with ILIKE (Escape Wildcards)

```typescript
// CRITICAL: Must escape wildcards to prevent ILIKE injection
function escapeLikePattern(query: string): string {
  return query.replace(/[%_\\]/g, "\\$&")
}

const pattern = `%${escapeLikePattern(userInput)}%`
db.select().from(posts).where(ilike(posts.title, pattern))
```

## Anti-Patterns

### N+1 Queries

```typescript
// Bad: N+1 queries (one query per item)
for (const category of categories) {
  const posts = await getPostsByCategory(category.id)
}

// Good: Batch query with inArray()
const allPosts = await db
  .select()
  .from(posts)
  .where(inArray(posts.categoryId, categoryIds))

const postsByCategory = allPosts.reduce((acc, post) => {
  (acc[post.categoryId] ||= []).push(post)
  return acc
}, {})
```

### Mass Assignment

```typescript
// Bad: Direct spread allows unintended fields
await db.update(posts).set({ ...formData })

// Good: Explicit validated fields only
const validated = Schema.parse(formData)
await db.update(posts).set({
  title: validated.title,
  content: validated.content,
})
```

## Migration Strategy

- **Development**: `drizzle-kit push` for rapid iteration
- **Production**: `drizzle-kit generate` + `migrate` for versioned migrations

## Deprecated APIs

- `InferModel` -> Use `InferSelectModel` and `InferInsertModel`

## Type Safety

```typescript
// Define types from schema
type Post = InferSelectModel<typeof posts>
type NewPost = InferInsertModel<typeof posts>

// Use in functions
async function createPost(data: NewPost): Promise<Post> {
  const [result] = await db.insert(posts).values(data).returning()
  return result
}
```
