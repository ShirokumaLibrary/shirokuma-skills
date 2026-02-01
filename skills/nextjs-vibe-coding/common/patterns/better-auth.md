# Better Auth Patterns

Better Auth を使用した認証パターン。セッションはDBに保存され、パスワードは `accounts` テーブルに格納される。

---

## Solution

### Admin Verification (Server)

```typescript
// lib/auth-utils.ts
import { auth } from "@/lib/auth"
import { db, users, eq } from "@repo/database"

export async function verifyAdmin(headers: Headers) {
  const session = await auth.api.getSession({ headers })
  if (!session) return null

  const [user] = await db
    .select({ role: users.role })
    .from(users)
    .where(eq(users.id, session.user.id))

  if (user?.role !== "admin") return null

  return { user: { ...session.user, role: user.role } }
}
```

### Client-side Session

```typescript
"use client"

import { authClient } from "@/lib/auth-client"

export function UserNav() {
  const { data: session } = authClient.useSession()

  const handleLogout = () => {
    authClient.signOut()
    window.location.href = "/login"  // Use location, NOT router.push()
  }

  return (/* ... */)
}
```

---

## Usage

- サーバーサイドで管理者権限を確認する場合 → `verifyAdmin()`
- クライアントサイドでセッション情報を取得する場合 → `authClient.useSession()`
- ログアウト後のリダイレクト → `window.location.href`

---

## Key Points

- パスワードは `users` ではなく `accounts` テーブルに保存される
- ログイン成功後は `router.push()` ではなく `window.location.href` を使う（リダイレクトループ回避）
- セッションはクッキーではなくデータベースに保存される
- CSRF保護は `verifyAdminMutation()` で自動的に適用される（[csrf-protection.md](csrf-protection.md) 参照）
