# 🧱 NestJS Architecture Structure (Standard - Clean Architecture)

## 1. Overview
This document defines a **standard module-based architecture** for NestJS projects following:
- Clean Architecture principles
- Domain-Driven Design (DDD-lite)
- Scalable and maintainable structure

---

## 2. High-Level Principles

### 2.1 Layer Separation
The system MUST be divided into 4 layers:

- **Domain** → Core business logic (framework-independent)
- **Application** → Use cases (orchestration layer)
- **Infrastructure** → External systems (DB, APIs, services)
- **Interface** → Delivery layer (HTTP, GraphQL, etc.)

### 2.2 Dependency Rule
Dependencies MUST follow this direction:

Interface → Application → Domain  
Infrastructure → Application / Domain (via interfaces)

- Domain MUST NOT depend on NestJS or any framework
- Application MUST NOT depend on Infrastructure implementations

---

## 3. Project Structure

```
src/
├── modules/
│   └── <module-name>/
│       ├── domain/
│       │   ├── entities/
│       │   ├── value-objects/
│       │   ├── repositories/
│       │   └── services/
│       │
│       ├── application/
│       │   ├── use-cases/
│       │   ├── dto/
│       │   └── mappers/
│       │
│       ├── infrastructure/
│       │   ├── database/
│       │   ├── external/ 
│       │   └── mappers/
│       │
│       ├── interface/
│       │   ├── controllers/
│       │   ├── guards/
│       │   ├── pipes/
│       │   └── interceptors/
│       │
│       └── <module-name>.module.ts
│
├── shared/
│   ├── domain/
│   ├── infrastructure/
│   └── utils/
│
├── config/
├── main.ts
└── app.module.ts
```

---

## 4. Layer Responsibilities

### 4.1 Domain Layer

**Purpose:** Contains core business logic.

**Rules:**
- MUST NOT import NestJS
- MUST NOT access database directly

**Includes:**
- Entities
- Value Objects
- Repository Interfaces
- Domain Services
- Domain Errors

---

### 4.2 Application Layer

**Purpose:** Executes business use cases.

**Rules:**
- MUST depend only on Domain
- MUST use repository interfaces (not implementations)

**Includes:**
- Use Cases (1 file per use case)
- DTOs (input/output)
- Mappers

---

### 4.3 Infrastructure Layer

**Purpose:** Implements external dependencies.

**Includes:**
- Database repositories
- Third-party API clients
- Persistence models

**Rules:**
- MUST implement interfaces from Domain
- SHOULD NOT contain business logic

---

### 4.4 Interface Layer

**Purpose:** Handles incoming requests.

**Includes:**
- Controllers
- Guards
- Pipes
- Interceptors

**Rules:**
- MUST NOT contain business logic
- MUST call Application layer only

---

## 5. Naming Conventions

| Component        | Convention                          | Example                         |
|-----------------|------------------------------------|---------------------------------|
| Module          | kebab-case                         | `user-profile`                  |
| Entity          | PascalCase                         | `UserEntity`                    |
| Use Case        | PascalCase + UseCase               | `CreateUserUseCase`             |
| Repository      | Interface suffix                   | `UserRepository`                |
| Implementation  | Impl suffix                        | `UserRepositoryImpl`            |
| DTO             | Suffix DTO                         | `CreateUserDto`                 |

---

## 6. Dependency Injection Rules

- MUST use interface tokens instead of concrete classes

Example:

```ts
{
  provide: 'UserRepository',
  useClass: UserRepositoryImpl,
}
```

- Use cases MUST receive dependencies via constructor

---

## 7. Use Case Standard

Each use case MUST:
- Be a separate class
- Have a single responsibility
- Expose an `execute()` method

Example:

```ts
export class CreateUserUseCase {
  constructor(private readonly userRepo: UserRepository) {}

  async execute(input: CreateUserDto) {
    // business logic
  }
}
```

---

## 8. DTO Rules

- MUST use class-based DTOs
- MUST use validation decorators (class-validator)
- MUST be separated from domain entities

---

## 9. Mapper Rules

- MUST convert:
  - DTO ↔ Domain
  - Domain ↔ Persistence

- MUST NOT leak DB schema into Domain

---

## 10. Error Handling

- Domain MUST define custom errors
- Application MAY map errors to response codes
- Interface MUST handle HTTP transformation

---

## 11. Module Design Rules

- Each module = one bounded context
- Modules MUST be independent
- Cross-module communication via interfaces only

---

## 12. Anti-Patterns (MUST AVOID)

- ❌ Business logic inside controllers
- ❌ Direct DB access in use cases
- ❌ Sharing entities between modules
- ❌ Using `any` type in DTOs
- ❌ Tight coupling between modules

---

## 13. Optional Extensions

For large-scale systems:

- CQRS pattern
- Event-driven architecture
- Saga / workflow orchestration
- Multi-tenant support
- Caching layer (Redis)

---

## 14. Example Module (User)

```
user/
├── domain/
├── application/
├── infrastructure/
├── interface/
└── user.module.ts
```

---

## 15. Summary

This architecture ensures:
- High scalability
- Clear separation of concerns
- Testability
- Maintainability

ALL contributors MUST follow this structure.
