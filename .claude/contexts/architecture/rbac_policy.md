# Role-Based Access Control (RBAC) Policy

## 1. Role Definitions
This project implements a strict hierarchical role system to manage access levels:

| Role | Description | Code Constant |
|------|-------------|---------------|
| **ADMIN** | Full system access, managing all users, and global configurations. | `Role.ADMIN` |
| **USER** | Standard access to personal profile and own data only. | `Role.USER` |

## 2. Permission Matrix (Core Requirements)
The following matrix defines the enforcement rules for both Backend and Frontend:

| Feature | Action | USER | ADMIN | Enforcement Point |
|---------|--------|:----:|:-----:|-------------------|
| **Auth** | Login / Register / Logout | ✅ | ✅ | Shared Auth Lib |
| **Profile** | View/Edit Own Profile | ✅ | ✅ | `api/users/me` |
| **Users** | List All Users | ❌ | ✅ | `api/users` |
| **Users** | Create/Update/Delete Users | ❌ | ✅ | `api/users/:id` |
| **Admin UI**| Access `/admin` Dashboard | ❌ | ✅ | Next.js Middleware |

## 3. Backend Enforcement (NestJS)
To ensure security, permissions must be enforced at the server level:

- **Mechanism**: Use a custom `RolesGuard` in conjunction with a `@Roles()` decorator.
- **Session Extraction**: Role information must be retrieved from the **Better Auth** session stored in the database.
- **Default State**: Deny by default. If a route has no decorator, it should require at least a valid session.

### Implementation Pattern:
```typescript
@Controller('users')
@UseGuards(AuthGuard, RolesGuard)
export class UserController {
  @Get()
  @Roles('admin') // Strictly enforced for ADMIN only
  async getAllUsers() {
    return this.userService.findAll();
  }

  @Get('me')
  @Roles('admin', 'user') // Both roles can access their own profile
  async getProfile(@User() user: UserEntity) {
    return user;
  }
}