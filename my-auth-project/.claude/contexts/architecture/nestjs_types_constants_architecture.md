# 🧩 NestJS Types & Constants Architecture (Standard)

## 1. 🎯 Objective

Define a **consistent, scalable, and clean architecture** for managing:

- Types
- Constants

Aligned with:
- Clean Architecture
- DDD-lite
- Existing NestJS module structure

---

## 2. 🏗️ Folder Structure

### 2.1 Module Level

```
modules/
└── user/
    ├── domain/
    │   ├── types/
    │   │   ├── user.type.ts
    │   │   └── user-status.type.ts
    │   │
    │   ├── constants/
    │   │   └── user.constants.ts
    │
    ├── application/
    │   ├── types/
    │   │   ├── create-user.input.ts
    │   │   └── create-user.output.ts
    │
    ├── infrastructure/
    │   ├── types/
    │   │   └── user.persistence.ts
    │
    └── interface/
        ├── types/
        │   └── user.response.ts
```

---

### 2.2 Shared Level

```
shared/
├── types/
│   ├── common.type.ts
│   ├── pagination.type.ts
│   └── api-response.type.ts
│
├── constants/
│   ├── error-code.constants.ts
│   ├── system.constants.ts
│   └── header.constants.ts
```

---

## 3. 🧠 Core Principles

### 3.1 Layer Isolation (MANDATORY)

| Layer          | Allowed Types |
|----------------|--------------|
| Domain         | Business types only |
| Application    | Input / Output contracts |
| Infrastructure | DB / external models |
| Interface      | HTTP response types |

❗ NEVER reuse types across incorrect layers

---

### 3.2 No Type Leakage

- Domain MUST NOT know:
  - DTO
  - Database schema
  - HTTP response

---

### 3.3 Avoid `any`

- `any` is strictly forbidden
- Use generics or union types instead

---

## 4. 🧾 Type Design Rules

### 4.1 Domain Types

```ts
export type UserId = string;

export type UserStatus = 'ACTIVE' | 'INACTIVE';
```

Rules:
- Prefer **union types** over enums
- Keep framework-independent

---

### 4.2 Branded Types (Recommended ⭐)

```ts
export type UserId = string & { readonly brand: unique symbol };
```

Purpose:
- Prevent mixing IDs
- Improve type safety

---

### 4.3 Application Types

```ts
export type CreateUserInput = {
  email: string;
  name: string;
};

export type CreateUserOutput = {
  id: string;
};
```

Rules:
- Represent use-case contracts
- MUST NOT contain business logic

---

### 4.4 Infrastructure Types

```ts
export type UserPersistence = {
  PK: string;
  SK: string;
  email: string;
};
```

Rules:
- Reflect database schema
- MUST NOT leak to domain

---

### 4.5 Interface Types

```ts
export type UserResponse = {
  id: string;
  email: string;
};
```

Rules:
- Represent API response shape only

---

## 5. 🔢 Constants Architecture

### 5.1 Principles

- No magic strings
- Centralized definition
- Group by domain or concern

---

### 5.2 Domain Constants

```ts
export const USER_STATUS = {
  ACTIVE: 'ACTIVE',
  INACTIVE: 'INACTIVE',
} as const;
```

---

### 5.3 Shared Constants

```ts
export const API_HEADER = {
  TENANT_ID: 'x-tenant-id',
  REQUEST_ID: 'x-request-id',
};
```

---

### 5.4 Error Codes (Critical ⭐)

```ts
export const ERROR_CODE = {
  USER_NOT_FOUND: 'USER_NOT_FOUND',
  UNAUTHORIZED: 'UNAUTHORIZED',
} as const;
```

Rules:
- Must be consistent across all layers
- Used for:
  - Domain errors
  - API responses

---

## 6. 🔄 Mapping Rules

| From → To            | Required |
|---------------------|--------|
| DTO → Application   | ✅ |
| Application → Domain| ✅ |
| Domain → Persistence| ✅ |
| Domain → Response   | ✅ |

❗ NEVER reuse types directly across layers

---

## 7. 🚫 Anti-patterns

### ❌ Dump Types File

```
types.ts
```

---

### ❌ Reusing DTO in Domain

```
CreateUserDto inside domain
```

---

### ❌ Global Enum Abuse

```ts
enum Status {}
```

---

### ❌ Magic Strings

```ts
if (status === 'ACTIVE') {}
```

---

## 8. 🧩 Advanced Patterns

### 8.1 Result Type

```ts
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };
```

---

### 8.2 Pagination Generic

```ts
export type Pagination<T> = {
  items: T[];
  total: number;
};
```

---

### 8.3 API Response Standard

```ts
export type ApiResponse<T> = {
  data: T;
  meta?: {
    pagination?: {
      total: number;
    };
  };
};
```

---

## 9. 🔥 Summary

This architecture ensures:

- Strong type safety
- Clear separation of concerns
- No cross-layer leakage
- Scalable and maintainable codebase

---

## 10. 📌 Enforcement (Recommended)

- ESLint rules for:
  - No cross-layer imports
  - No `any`
- Strict TypeScript config
- Code review checklist for types & constants

---

ALL contributors MUST follow this standard.

