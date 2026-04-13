# Database Architecture (Prisma & PostgreSQL)

## 1. Overview
This project uses **Prisma ORM** with **PostgreSQL** to manage the data layer. The database logic is centralized in a shared library to ensure consistency across the monorepo.

## 2. Library Location (`libs/database`)
- **Schema Path**: `libs/database/prisma/schema.prisma`
- **Responsibility**: 
    - Defining data models.
    - Managing migrations.
    - Exporting a pre-configured `PrismaClient`.

## 3. Data Modeling Rules
- **Naming Convention**: 
    - Models: `PascalCase` (e.g., `User`, `UserRole`).
    - Fields: `camelCase` (e.g., `createdAt`, `updatedAt`).
    - Tables: Map to snake_case if required by DB standards (e.g., `@@map("users")`).
- **Audit Fields**: Every table MUST have `createdAt` and `updatedAt`.
- **Enums**: Use Prisma `enum` for fixed values like `Role` or `Status`.

## 4. Layer Isolation & Access
- **Infrastructure Layer**: Direct Prisma access is ONLY allowed in the `infrastructure` layer of NestJS modules.
- **No Direct UI Access**: Next.js (apps/web) SHOULD NOT query Prisma directly for business logic. It must call the NestJS API to ensure RBAC enforcement.
- **Exception**: Server Actions in Next.js may use Prisma for simple read-only operations if performance is critical, but must manually verify sessions.

## 5. Better Auth Integration
- The `User`, `Session`, `Account`, and `Verification` models must follow the Better Auth schema requirements.
- Any custom fields (like `role`) added to the `User` model must be reflected in the Better Auth configuration.

---

