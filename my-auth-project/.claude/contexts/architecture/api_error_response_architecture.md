# 📦 API Response & Error Handling Architecture (Full Standard)

## 1. Overview

This document defines a **complete architecture standard** for handling API responses and errors in backend systems (NestJS-ready, framework-agnostic design).

### Goals
- Unified response format
- Consistent error handling across all layers
- Clear contract between backend and frontend
- Scalable for large systems (IAM, multi-tenant, microservices)

---

## 2. Design Principles

- DO NOT return `null` fields
- Response MUST be minimal and predictable
- Error MUST be structured and machine-readable
- Business errors MUST be distinguishable from system errors
- Logging and tracing MUST be supported

---

## 3. Response Architecture

### 3.1 Success Response

```json
{
  "success": true,
  "data": {}
}
```

### Optional Fields

```json
{
  "success": true,
  "data": {},
  "meta": {
    "requestId": "req-123",
    "timestamp": "2026-04-08T10:00:00Z"
  }
}
```

---

### 3.2 Pagination Response

```json
{
  "success": true,
  "data": [],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 100,
      "totalPages": 10
    }
  }
}
```

---

## 4. Error Response

### 4.1 Standard Error

```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User not found",
    "type": "BUSINESS"
  }
}
```

---

### 4.2 Error with Details

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "type": "VALIDATION",
    "details": {
      "email": "Invalid email format"
    }
  }
}
```

---

### 4.3 Error Types

| Type       | Description                          |
|------------|--------------------------------------|
| BUSINESS   | Business logic violations            |
| VALIDATION | Input validation errors              |
| SYSTEM     | Internal server or infrastructure    |
| AUTH       | Authentication / authorization       |

---

## 5. Error Code System

### Format

```
<DOMAIN>_<ERROR_NAME>
```

### Examples

- USER_NOT_FOUND
- AUTH_INVALID_TOKEN
- TENANT_ACCESS_DENIED
- VALIDATION_ERROR

### Rules

- MUST be stable (do not change once released)
- MUST be unique per domain
- MUST be machine-readable (UPPER_SNAKE_CASE)

---

## 6. Layered Error Handling

### 6.1 Domain Layer

- Defines domain errors
- MUST NOT depend on framework

Example:

```ts
export class UserNotFoundError extends Error {
  code = 'USER_NOT_FOUND';
}
```

---

### 6.2 Application Layer

- Handles use case logic
- MAY transform domain errors

---

### 6.3 Infrastructure Layer

- Handles DB / external errors
- MUST map external errors to internal error format

---

### 6.4 Interface Layer

- Converts errors to HTTP response
- MUST NOT leak internal errors

---

## 7. HTTP Mapping

| HTTP | Use Case                  |
|------|---------------------------|
| 200  | Success                   |
| 201  | Created                   |
| 400  | Validation error          |
| 401  | Unauthorized              |
| 403  | Forbidden                 |
| 404  | Not found                 |
| 500  | Internal error            |

---

## 8. NestJS Implementation

### 8.1 Response Interceptor

```ts
@Injectable()
export class ResponseInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    return next.handle().pipe(
      map((data) => {
        const res: any = {
          success: true,
          data,
        };

        if (data?.meta) {
          res.meta = data.meta;
        }

        return res;
      }),
    );
  }
}
```

---

### 8.2 Exception Filter

```ts
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const res = host.switchToHttp().getResponse();

    res.status(exception.status || 500).json({
      success: false,
      error: {
        code: exception.code || 'INTERNAL_ERROR',
        message: exception.message,
        type: exception.type || 'SYSTEM',
        ...(exception.details && { details: exception.details }),
      },
    });
  }
}
```

---

## 9. Validation Handling

- MUST map class-validator errors

Example:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "type": "VALIDATION",
    "details": {
      "email": "Invalid email"
    }
  }
}
```

---

## 10. Observability

### Meta Fields

```json
{
  "meta": {
    "requestId": "req-123",
    "traceId": "trace-xyz",
    "timestamp": "ISO_DATE"
  }
}
```

### Rules

- requestId MUST be generated per request
- traceId SHOULD integrate with logging system

---

## 11. Anti-Patterns

- ❌ Returning raw exceptions
- ❌ Exposing stack traces
- ❌ Using dynamic error messages without code
- ❌ Returning inconsistent response shapes

---

## 12. Summary

All APIs MUST:

- Return consistent response structure
- Avoid null fields
- Use structured error codes
- Separate error handling by layers
- Support observability and tracing

This architecture is REQUIRED for all backend services.

