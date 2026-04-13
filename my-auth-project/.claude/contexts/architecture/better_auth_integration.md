# Better Auth Integration & RBAC Strategy

## 1. Technical Overview
Better Auth is used as the unified authentication framework for this monorepo, providing session management, database adapters, and extensible plugin support.

### Key Advantages for this Project:
- **Framework Agnostic**: Works seamlessly with both Next.js (App Router) and NestJS (via custom guards).
- **Database-First**: Uses Prisma adapter for persistent session management.
- **Built-in RBAC**: Simplifies role management through user metadata and plugins.

## 2. Shared Configuration (`libs/auth`)
To maintain consistency, all Better Auth logic is centralized:
- **Adapter**: Prisma adapter connected to `@proj/database`.
- **Secret Management**: Shared `BETTER_AUTH_SECRET` and `BETTER_AUTH_URL` managed via `@proj/config`.
- **Plugin - Admin**: Enable the `admin` plugin to handle specialized user management tasks.

## 3. Integration Strategy

### A. NextJS (Frontend Implementation)
- **Client Side**: Use `createAuthClient` for login, register, and logout hooks.
- **Server Side**: Use `auth.getSession(headers())` in Server Components and Middleware to protect routes.
- **Middleware**: Intercept requests to `/admin/**` and redirect users if their role is not `admin`.

### B. NestJS (Backend Implementation)
Better Auth session data must be verified in NestJS:
1. **Auth Guard**: A global or controller-level guard that extracts the session token from the `Authorization` header or cookies.
2. **Session Verification**: Use the Better Auth instance within NestJS to validate the token against the database.
3. **Request Decorator**: A custom `@User()` decorator to inject the authenticated user's session and roles into controller methods.

## 4. RBAC Implementation (Permission System)
As per assignment requirements:
- **Roles**: `admin` and `user`.
- **Database Schema**: The `User` table includes a `role` field.
- **Permissions**:
    - **Admin**: Has full access to `/api/users` CRUD operations.
    - **User**: Restricted to `/api/users/me` (viewing own profile only).
- **Backend Enforcement**: Use a `@Roles('admin')` decorator on NestJS controllers to trigger role-based logic in the `RolesGuard`.

## 5. Session Strategies
- **Persistence**: Sessions are stored in the PostgreSQL/SQLite database to ensure security and allow for remote logout.
- **Synchronization**: Use cross-domain cookies or shared headers to ensure the `api` (NestJS) and `web` (Next.js) stay in sync regarding the user's state.

## 6. Comparison Note (For Research Report)
- **NextAuth**: Limited primarily to Next.js.
- **Better Auth**: More flexible for Monorepos, offers better Type-safety, and provides an easier path to extend schemas for RBAC.