# Next.js 16 (App Router) – Architecture & Naming Convention Guide

## 🎯 Purpose
Provide a **production-ready, scalable, and consistent structure** for Next.js 16 projects, including:
- Folder architecture
- Naming conventions
- Component structure
- Error handling strategy

---

# 📦 1. Project Structure

```
src/
├── app/
│   ├── (public)/
│   ├── (auth)/
│   ├── (dashboard)/
│   │   └── users/
│   │       ├── page.tsx
│   │       ├── loading.tsx
│   │       ├── error.tsx
│   │       ├── not-found.tsx
│   │       └── _components/
│   │
│   ├── api/
│   │   └── users/
│   │       └── route.ts
│   │
│   ├── layout.tsx
│   └── globals.css
│
├── components/
│   ├── ui/
│   ├── common/
│   └── layout/
│
├── modules/
│   └── user/
│       ├── components/
│       ├── hooks/
│       ├── services/
│       ├── types/
│       └── utils/
│
├── lib/
│   ├── fetcher.ts
│   ├── error.ts
│   └── constants.ts
│
├── hooks/
├── types/
├── utils/
└── config/
```

---

# 🧭 2. Architecture Principles

## 2.1 app/ (Routing Layer Only)

- Contains:
  - page.tsx
  - layout.tsx
  - loading.tsx
  - error.tsx
  - not-found.tsx
- ❌ No business logic

---

## 2.2 modules/ (Domain-driven)

Each domain is isolated:

```
modules/user/
├── components/
├── hooks/
├── services/
├── types/
└── utils/
```

### Rules
- One domain = one module
- No cross-module coupling
- Business logic lives here

---

## 2.3 components/ (Shared UI)

```
components/
├── ui/        # design system
├── common/    # reusable components
└── layout/    # layout components
```

---

# 🧱 3. Naming Convention (IMPORTANT)

## 3.1 File Naming Rules

| Type        | Convention   | Example            |
|------------|-------------|--------------------|
| Component  | kebab-case  | user-card.tsx      |
| Hook       | camelCase   | useUsers.ts        |
| Service    | kebab-case  | user.service.ts    |
| Utils      | kebab-case  | format-date.ts     |
| Types      | kebab-case  | user.type.ts       |
| Folder     | kebab-case  | user-management/   |

---

## 3.2 Component Naming

```tsx
export function UserCard() {}
```

- Use PascalCase for component names
- File remains kebab-case

---

## 3.3 Hook Naming

```ts
export function useUsers() {}
```

- Must start with `use`
- Use camelCase

---

## 3.4 Private Folder Convention

```
_components/
_hooks/
```

- `_` prefix = private scope
- Not reusable outside module/route

---

## 3.5 Next.js Reserved Files

```
page.tsx
layout.tsx
loading.tsx
error.tsx
not-found.tsx
route.ts
```

- ❗ Do NOT rename

---

# ⚡ 4. Page Structure

```
app/(dashboard)/users/
├── page.tsx
├── loading.tsx
├── error.tsx
├── not-found.tsx
└── _components/
```

---

## 4.1 page.tsx (Orchestration only)

```tsx
import { UsersList } from "./_components/users-list";

export default function UsersPage() {
  return <UsersList />;
}
```

---

## 4.2 loading.tsx

```tsx
export default function Loading() {
  return <div>Loading...</div>;
}
```

---

## 4.3 error.tsx

```tsx
"use client";

export default function Error({ error, reset }) {
  return (
    <div>
      <p>Something went wrong</p>
      <button onClick={reset}>Retry</button>
    </div>
  );
}
```

---

## 4.4 not-found.tsx

```tsx
export default function NotFound() {
  return <div>404 - Not Found</div>;
}
```

---

# 🚨 5. Error Handling Architecture

## 5.1 Centralized Error Model

```ts
export class AppError extends Error {
  constructor(
    public code: string,
    public message: string,
    public status: number = 500,
    public details?: unknown
  ) {
    super(message);
  }
}
```

---

## 5.2 Error Codes

```
USER_NOT_FOUND
UNAUTHORIZED
FORBIDDEN
VALIDATION_ERROR
INTERNAL_ERROR
```

---

## 5.3 API Route Handling

```ts
import { NextResponse } from "next/server";
import { AppError } from "@/lib/error";

export async function GET() {
  try {
    return NextResponse.json({ data: [] });
  } catch (err) {
    if (err instanceof AppError) {
      return NextResponse.json(
        {
          error: {
            code: err.code,
            message: err.message,
            details: err.details,
          },
        },
        { status: err.status }
      );
    }

    return NextResponse.json(
      {
        error: {
          code: "INTERNAL_ERROR",
          message: "Something went wrong",
        },
      },
      { status: 500 }
    );
  }
}
```

---

## 5.4 Fetcher Wrapper

```ts
export async function fetcher(url: string) {
  const res = await fetch(url);
  const json = await res.json();

  if (!res.ok) {
    throw new Error(json.error?.message || "Unknown error");
  }

  return json.data;
}
```

---

# 🔁 6. Data Flow

```
UI (Component)
   ↓
Hook (useX)
   ↓
Service
   ↓
Fetcher
   ↓
API Route
```

---

# 🧠 7. Best Practices

## ✅ DO
- Separate UI / Logic / Data
- Use domain-based modules
- Centralize error handling
- Keep page.tsx clean

## ❌ DON'T
- Put business logic in page.tsx
- Fetch directly in large UI components
- Hardcode error messages
- Cross-import between modules

---

# 🔥 8. TL;DR

- app/ = routing only
- modules/ = business logic
- components/ = shared UI
- naming = depends on file type (NOT all kebab-case)
- error = structured with code

---

# 🚀 Optional Enhancements

- TanStack Query / SWR
- Zod validation
- ESLint rules for naming enforcement

---

**Author:** Internal Engineering Standard
**Version:** 1.0.0

