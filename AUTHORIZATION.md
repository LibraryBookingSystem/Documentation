# Authorization & RBAC Implementation

## Overview

All microservices implement JWT-based authentication with AOP (Aspect-Oriented Programming) role-based access control. Authorization is declarative via annotations.

## Authentication

All protected endpoints require JWT Bearer token:

```
Authorization: Bearer <token>
```

JWT tokens contain: `userId`, `role`, `username` - automatically extracted by services.

## Roles

| Role      | Description                                             |
| --------- | ------------------------------------------------------- |
| `STUDENT` | Basic user - can create/manage own bookings             |
| `FACULTY` | Extended privileges - can approve users, view analytics |
| `ADMIN`   | Full access - manage all resources, users, policies     |

## Authorization Annotations

### @RequiresRole

```java
@RequiresRole({"ADMIN"})           // Admin only
@RequiresRole({"ADMIN", "FACULTY"}) // Admin or Faculty
@RequiresRole                       // Any authenticated user
```

### @RequiresOwnership

```java
@RequiresOwnership(resourceIdParam = "id", byUserId = true)  // User owns resource
@RequiresOwnership(resourceIdParam = "username", byUserId = false)  // By username
```

### Service-Specific Ownership

- `@RequiresBookingOwnership` - Validates booking belongs to user
- `@RequiresNotificationOwnership` - Validates notification belongs to user

## Endpoint Authorization Summary

### Auth Service (`/api/auth`)

| Endpoint    | Method | Auth   |
| ----------- | ------ | ------ |
| `/register` | POST   | PUBLIC |
| `/login`    | POST   | PUBLIC |
| `/health`   | GET    | PUBLIC |

### User Service (`/api/users`)

| Endpoint               | Method | Auth          | Ownership     |
| ---------------------- | ------ | ------------- | ------------- |
| `/me`                  | GET    | AUTHENTICATED | Self          |
| `/{id}`                | GET    | AUTHENTICATED | Self or Admin |
| `/username/{username}` | GET    | AUTHENTICATED | Self or Admin |
| `/`                    | GET    | ADMIN         | -             |
| `/{id}/restrict`       | POST   | ADMIN         | -             |
| `/{id}/unrestrict`     | POST   | ADMIN         | -             |
| `/{id}/restricted`     | GET    | AUTHENTICATED | Self or Admin |
| `/pending`             | GET    | FACULTY/ADMIN | -             |
| `/rejected`            | GET    | FACULTY/ADMIN | -             |
| `/{id}/approve`        | POST   | FACULTY/ADMIN | -             |
| `/{id}/reject`         | POST   | FACULTY/ADMIN | -             |
| `/{id}`                | DELETE | ADMIN         | -             |

### Catalog Service (`/api/resources`)

| Endpoint  | Method | Auth          |
| --------- | ------ | ------------- |
| `/`       | POST   | ADMIN         |
| `/`       | GET    | AUTHENTICATED |
| `/{id}`   | GET    | AUTHENTICATED |
| `/{id}`   | PUT    | ADMIN         |
| `/{id}`   | DELETE | ADMIN         |
| `/health` | GET    | PUBLIC        |

### Booking Service (`/api/bookings`)

| Endpoint                 | Method | Auth          | Ownership              |
| ------------------------ | ------ | ------------- | ---------------------- |
| `/`                      | POST   | AUTHENTICATED | Self (userId from JWT) |
| `/`                      | GET    | ADMIN         | -                      |
| `/{id}`                  | GET    | AUTHENTICATED | Self or Admin          |
| `/user/{userId}`         | GET    | AUTHENTICATED | Self or Admin          |
| `/resource/{resourceId}` | GET    | AUTHENTICATED | -                      |
| `/{id}`                  | PUT    | AUTHENTICATED | Self or Admin          |
| `/{id}`                  | DELETE | AUTHENTICATED | Self or Admin          |
| `/checkin`               | POST   | AUTHENTICATED | Self or Admin          |
| `/health`                | GET    | PUBLIC        |

### Policy Service (`/api/policies`)

| Endpoint    | Method | Auth          |
| ----------- | ------ | ------------- |
| `/`         | POST   | ADMIN         |
| `/`         | GET    | AUTHENTICATED |
| `/{id}`     | GET    | AUTHENTICATED |
| `/{id}`     | PUT    | ADMIN         |
| `/{id}`     | DELETE | ADMIN         |
| `/validate` | POST   | AUTHENTICATED |
| `/health`   | GET    | PUBLIC        |

### Notification Service (`/api/notifications`)

| Endpoint                      | Method | Auth          | Ownership     |
| ----------------------------- | ------ | ------------- | ------------- |
| `/user/{userId}`              | GET    | AUTHENTICATED | Self or Admin |
| `/user/{userId}/unread`       | GET    | AUTHENTICATED | Self or Admin |
| `/user/{userId}/unread/count` | GET    | AUTHENTICATED | Self or Admin |
| `/{id}/read`                  | PUT    | AUTHENTICATED | Self or Admin |
| `/user/{userId}/read-all`     | PUT    | AUTHENTICATED | Self or Admin |
| `/health`                     | GET    | PUBLIC        |

### Analytics Service (`/api/analytics`)

| Endpoint       | Method | Auth          |
| -------------- | ------ | ------------- |
| `/utilization` | GET    | FACULTY/ADMIN |
| `/peak-hours`  | GET    | FACULTY/ADMIN |
| `/overall`     | GET    | ADMIN         |
| `/health`      | GET    | PUBLIC        |

## Configuration

All services use the same JWT secret (set via environment variable in production):

```yaml
jwt:
  secret: ${JWT_SECRET:my-super-secret-jwt-key-for-library-booking-system-2024}
  expiration: ${JWT_EXPIRATION:3600000}
```

## Error Responses

| Code | Description                              |
| ---- | ---------------------------------------- |
| 401  | Missing or invalid JWT token             |
| 403  | Valid token but insufficient permissions |
| 404  | Resource not found or no access          |

---

**Status**: ✅ Fully Implemented  
**Last Updated**: December 2025
