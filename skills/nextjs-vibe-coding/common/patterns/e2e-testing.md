# E2E Testing Best Practices

## Directory Organization (App-Based)

Organize E2E tests by app in monorepo projects:

```
tests/e2e/
├── admin/           # Admin app tests
│   ├── auth/
│   │   ├── login.test.ts
│   │   └── logout.test.ts
│   ├── posts/
│   │   ├── create.test.ts
│   │   └── list.test.ts
│   └── settings/
├── public/          # Public app tests
│   ├── blog/
│   ├── comments/
│   └── search/
├── shared/          # Cross-app tests (rare)
│   └── seo.test.ts
└── helpers/         # Shared test utilities
    ├── database.ts
    ├── fixtures.ts
    └── auth.ts
```

**Benefits**:
- Clear ownership: Which tests belong to which app
- Parallel execution: Run app tests independently
- Easier maintenance: Changes in one app don't affect others
- Better CI/CD: Run affected app tests only on changes

**Playwright Configuration**:
```typescript
// playwright.config.ts
export default defineConfig({
  projects: [
    {
      name: "admin",
      testDir: "./tests/e2e/admin",
      use: { baseURL: "https://admin-test.local.test" },
    },
    {
      name: "public",
      testDir: "./tests/e2e/public",
      use: { baseURL: "https://public-test.local.test" },
    },
  ],
})
```

**Run specific app tests**:
```bash
npx playwright test --project=admin
npx playwright test --project=public
```

## Test Environment Isolation

Use separate test database and hosts:

| Environment | Host | Database |
|-------------|------|----------|
| Development | admin.local.test | blogcms_dev |
| E2E Test | admin-test.local.test | blogcms_test |

## Database Fixtures (TestDatabaseClient)

Direct database access for E2E test isolation:

```typescript
// tests/helpers/database.ts
import { Pool, PoolClient } from "pg"

const TEST_DATABASE_URL = "postgresql://postgres:postgres@localhost:5432/blogcms_test"
const pool = new Pool({ connectionString: TEST_DATABASE_URL, max: 5 })

export class TestDatabaseClient {
  private client: PoolClient | null = null

  async connect(): Promise<void> { this.client = await pool.connect() }
  async disconnect(): Promise<void> { this.client?.release(); this.client = null }

  async query<T = any>(sql: string, params?: any[]): Promise<T[]> {
    if (!this.client) throw new Error("Not connected")
    const result = await this.client.query(sql, params)
    return result.rows as T[]
  }

  async clearTable(tableName: string): Promise<void> {
    await this.query(`TRUNCATE TABLE "${tableName}" CASCADE`)
  }

  async clearAllPosts(): Promise<void> {
    await this.query("TRUNCATE TABLE posts CASCADE")
  }
}

export const testDb = new TestDatabaseClient()
```

## Empty State Testing Pattern

```typescript
import { testDb, seedTestDatabase } from "../helpers/database"

test.describe("Post List Empty State", () => {
  test.beforeAll(async () => {
    await testDb.connect()
    await testDb.clearAllPosts()
  })

  test.afterAll(async () => {
    await testDb.disconnect()
    await seedTestDatabase()  // Restore data for other tests
  })

  test("should display empty state when no posts exist", async ({ page }) => {
    await page.goto("/posts")
    await expect(page.getByText(/投稿がありません|No posts/i)).toBeVisible()
  })
})
```

## Serial Test Execution for State Dependencies

```typescript
// Use serial for tests that depend on each other's state
test.describe.serial("Comment Moderation", () => {
  test.beforeEach(async ({ page, context }) => {
    await context.clearCookies()
    await redisHelper.clearRateLimits("ratelimit:*")
    // Login before each test
  })

  test("should approve a pending comment", async ({ page }) => {
    // Goes first - approves a comment
  })

  test("should reject an approved comment", async ({ page }) => {
    // Goes second - needs approved comments from previous test
  })
})
```

## Icon Button Testing

For buttons with icons only (using `title` attribute):

```typescript
// Find icon button by title attribute (both languages)
const restoreButton = page.locator("button[title='Restore'], button[title='復元']").first()
await expect(restoreButton).toBeVisible()
await restoreButton.click()

// Or use screen reader text
const button = page.getByRole("button").filter({ has: page.locator("text=Restore") })
```

## Rate Limiting for Tests

Relax rate limits in development to prevent test failures:

```typescript
// lib/auth.ts
rateLimit: {
  window: 15 * 60,
  max: process.env.NODE_ENV === "production" ? 5 : 1000,
}
```

## Multi-Step Auth Flow Timeouts

Admin login includes multiple API calls (signIn → get-session → check-admin):

```typescript
// Increase timeout for multi-step flows
await expect(page).toHaveURL(/\/$/, { timeout: 15000 })
```

## Loading State Testing

Use Promise-based blocking to reliably test loading states:

```typescript
let resolveRequest: () => void
const requestPromise = new Promise<void>((r) => { resolveRequest = r })

await page.route("**/api/auth/sign-in/**", async (route) => {
  await requestPromise
  await route.continue()
})

// Click triggers blocked request, loading state stays visible
await submitButton.click()

// Verify loading state (button text changes!)
const loadingButton = page.getByRole("button", { name: /Logging in/i })
await expect(loadingButton).toBeVisible()

resolveRequest!()  // Release to complete
```

## Button Text During Loading

Button locators need to account for text changes:

```typescript
// Before click: "Log in"
// During loading: "Logging in..."

// WRONG: Original locator fails during loading
await expect(submitButton).toBeDisabled()

// CORRECT: Use new locator for loading state
const loadingButton = page.getByRole("button", { name: /Logging in/i })
await expect(loadingButton).toBeDisabled()
```
