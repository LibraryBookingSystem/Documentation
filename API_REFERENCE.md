# API Reference

## Base URL

- **API Gateway**: `http://localhost:8080` (recommended)
- Direct service access: ports 3001-3007

## Authentication

All protected endpoints require: `Authorization: Bearer <token>`

---

## Auth Service (`/api/auth`)

| Method | Endpoint    | Description    | Auth |
| ------ | ----------- | -------------- | ---- |
| POST   | `/register` | Register user  | No   |
| POST   | `/login`    | Login, get JWT | No   |
| GET    | `/health`   | Health check   | No   |

**Register Request:**

```json
{
  "username": "user",
  "email": "user@example.com",
  "password": "pass123",
  "role": "STUDENT"
}
```

**Login Request:**

```json
{ "username": "user", "password": "pass123" }
```

**Response:**

```json
{
  "token": "eyJ...",
  "user": { "id": 1, "username": "user", "role": "STUDENT" }
}
```

---

## User Service (`/api/users`)

| Method | Endpoint               | Description       | Auth              |
| ------ | ---------------------- | ----------------- | ----------------- |
| GET    | `/me`                  | Current user      | Yes               |
| GET    | `/`                    | All users         | ADMIN             |
| GET    | `/{id}`                | User by ID        | Yes (owner/admin) |
| GET    | `/username/{username}` | User by username  | Yes (owner/admin) |
| GET    | `/{id}/restricted`     | Check restriction | Yes (owner/admin) |
| POST   | `/{id}/restrict`       | Restrict user     | ADMIN             |
| POST   | `/{id}/unrestrict`     | Unrestrict user   | ADMIN             |
| GET    | `/pending`             | Pending users     | FACULTY/ADMIN     |
| GET    | `/rejected`            | Rejected users    | FACULTY/ADMIN     |
| POST   | `/{id}/approve`        | Approve user      | FACULTY/ADMIN     |
| POST   | `/{id}/reject`         | Reject user       | FACULTY/ADMIN     |
| DELETE | `/{id}`                | Delete user       | ADMIN             |

---

## Catalog Service (`/api/resources`)

| Method | Endpoint  | Description     | Auth  |
| ------ | --------- | --------------- | ----- |
| POST   | `/`       | Create resource | ADMIN |
| GET    | `/`       | List resources  | Yes   |
| GET    | `/{id}`   | Get resource    | Yes   |
| PUT    | `/{id}`   | Update resource | ADMIN |
| DELETE | `/{id}`   | Delete resource | ADMIN |
| GET    | `/health` | Health check    | No    |

**Query params for GET `/`:** `type`, `floor`, `status`, `search`

**Create/Update Request:**

```json
{
  "name": "Room 101",
  "description": "Study room",
  "type": "ROOM",
  "floor": 1,
  "capacity": 4,
  "status": "AVAILABLE"
}
```

---

## Booking Service (`/api/bookings`)

| Method | Endpoint                 | Description       | Auth              |
| ------ | ------------------------ | ----------------- | ----------------- |
| POST   | `/`                      | Create booking    | Yes               |
| GET    | `/`                      | All bookings      | ADMIN             |
| GET    | `/{id}`                  | Get booking       | Yes (owner/admin) |
| GET    | `/user/{userId}`         | User's bookings   | Yes (owner/admin) |
| GET    | `/resource/{resourceId}` | Resource bookings | Yes               |
| PUT    | `/{id}`                  | Update booking    | Yes (owner/admin) |
| DELETE | `/{id}`                  | Cancel booking    | Yes (owner/admin) |
| POST   | `/checkin`               | Check-in (QR)     | Yes               |
| GET    | `/health`                | Health check      | No                |

**Create Request:**

```json
{
  "resourceId": 1,
  "startTime": "2025-12-10T10:00:00",
  "endTime": "2025-12-10T12:00:00"
}
```

**Check-in Request:**

```json
{ "qrCode": "BOOKING-1-ABC123" }
```

---

## Policy Service (`/api/policies`)

| Method | Endpoint    | Description      | Auth  |
| ------ | ----------- | ---------------- | ----- |
| POST   | `/`         | Create policy    | ADMIN |
| GET    | `/`         | List policies    | Yes   |
| GET    | `/{id}`     | Get policy       | Yes   |
| PUT    | `/{id}`     | Update policy    | ADMIN |
| DELETE | `/{id}`     | Delete policy    | ADMIN |
| POST   | `/validate` | Validate booking | Yes   |
| GET    | `/health`   | Health check     | No    |

**Query params for GET `/`:** `active` (boolean)

**Validate Request:**

```json
{
  "userId": 1,
  "resourceId": 1,
  "startTime": "2025-12-10T10:00:00",
  "endTime": "2025-12-10T14:00:00"
}
```

---

## Notification Service (`/api/notifications`)

| Method | Endpoint                      | Description          | Auth              |
| ------ | ----------------------------- | -------------------- | ----------------- |
| GET    | `/user/{userId}`              | User notifications   | Yes (owner/admin) |
| GET    | `/user/{userId}/unread`       | Unread notifications | Yes (owner/admin) |
| GET    | `/user/{userId}/unread/count` | Unread count         | Yes (owner/admin) |
| PUT    | `/{id}/read`                  | Mark as read         | Yes (owner/admin) |
| PUT    | `/user/{userId}/read-all`     | Mark all read        | Yes (owner/admin) |
| GET    | `/health`                     | Health check         | No                |

---

## Analytics Service (`/api/analytics`)

| Method | Endpoint       | Description       | Auth          |
| ------ | -------------- | ----------------- | ------------- |
| GET    | `/utilization` | Utilization stats | FACULTY/ADMIN |
| GET    | `/peak-hours`  | Peak hours        | FACULTY/ADMIN |
| GET    | `/overall`     | Overall stats     | ADMIN         |
| GET    | `/health`      | Health check      | No            |

**Query params:**

- `/utilization`: `startDate`, `endDate`, `resourceId` (optional)
- `/peak-hours`: `startTime`, `endTime`
- `/overall`: `startTime`, `endTime`

---

## Status Codes

| Code | Description                          |
| ---- | ------------------------------------ |
| 200  | Success                              |
| 201  | Created                              |
| 204  | No Content                           |
| 400  | Bad Request                          |
| 401  | Unauthorized (missing/invalid token) |
| 403  | Forbidden (insufficient permissions) |
| 404  | Not Found                            |
| 409  | Conflict                             |
| 500  | Server Error                         |

---

**Last Updated**: December 2025
