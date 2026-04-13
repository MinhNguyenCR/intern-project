# NX Monorepo Architecture Guidelines (Assignment Compliance)

## 1. Executive Summary
This project utilizes **NX** as the primary build system and monorepo manager. The architecture is designed to fulfill the requirements of the "Auth + User Management System," ensuring a clear separation between apps and shared logic while maintaining a high-performance task pipeline.

## 2. Workspace Structure
Following the "Expected Monorepo Structure" from the assignment, the workspace is organized as follows:

### Applications (`apps/`)
- **`web/`**: 
    - **Framework**: Next.js (App Router).
    - **Responsibility**: Frontend UI, Server Components (RSC), Client Components, Middleware-based route protection, and Better Auth client integration.
- **`api/`**: 
    - **Framework**: NestJS.
    - **Responsibility**: Backend REST API, Dependency Injection (DI) core, Guards for RBAC enforcement, and Prisma-based business logic.

### Shared Libraries (`libs/`)
To maximize code reuse and maintainability, the logic is decoupled into specific libraries:
- **`database/`**: Contains the Prisma schema and generated Client. This is the single source of truth for the PostgreSQL/SQLite schema.
- **`auth/`**: Contains Better Auth configurations, server-side session handlers, and shared authentication constants.
- **`shared-types/`**: Contains TypeScript interfaces, DTOs (Data Transfer Objects), and RBAC Role Enums used by both `web` and `api`.
- **`config/`**: Centralized environment variable management and workspace-wide constants.
- **`utils/`**: Shared helper functions (e.g., validation logic, date formatting).

## 3. RBAC & Data Flow Implementation

### Backend Enforcement (NestJS)
- **Guards**: Every protected endpoint in `api/` must use a custom `RolesGuard`.
- **Session**: Access sessions via Better Auth adapters to verify roles at the server level.

### Frontend Enforcement (Next.js)
- **Middleware**: Used for global route protection (e.g., redirecting unauthenticated users).
- **Server-Side Checks**: Permission checks must be performed in Server Components before rendering sensitive data.

## 4. Scaling by Domain (Advanced Architecture)
As the project grows, we follow the **Domain-Driven Design (DDD)** approach within NX:
1. **Grouping**: Libraries should be prefixed by domain (e.g., `libs/user/feature-management`, `libs/user/ui-card`).
2. **Boundary Rules**: We use Nx tags (`type:app`, `type:lib`) to prevent circular dependencies.

## 5. Task Pipeline & Performance
- **Caching**: Local and Remote caching are enabled for `build`, `test`, and `lint` targets.
- **Dependency Graph**: Use `npx nx graph` to visualize the relationship between the Auth library and the consumer apps.
- **Affected Commands**: Use `nx affected` to only run tests or builds on code that has changed, optimizing CI/CD time.