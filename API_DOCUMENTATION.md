# Library System API Documentation

## Base URL
- **API Gateway**: `http://localhost:8080`
- **Direct Service Access** (if gateway is disabled):
  - Auth Service: `http://localhost:3002`
  - User Service: `http://localhost:3001`
  - Catalog Service: `http://localhost:3003`
  - Booking Service: `http://localhost:3004`
  - Policy Service: `http://localhost:3005`
  - Notification Service: `http://localhost:3006`
  - Analytics Service: `http://localhost:3007`

## Authentication
Currently, the system uses `X-User-Id` header for authentication instead of JWT tokens. Include this header in protected endpoints.

---

## 1. Authentication Service (`/api/auth`)

### 1.1 Register User
**POST** `/api/auth/register`

**Description**: Register a new user account

**Headers**: None required

**Request Body**:
```json
{
  "username": "john_doe",
  "email": "john.doe@example.com",
  "password": "password123",
  "role": "STUDENT"
}
```

**Valid Roles**: `STUDENT`, `FACULTY`, `ADMIN`

**Response** (201 Created):
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "username": "john_doe",
    "email": "john.doe@example.com",
    "role": "STUDENT",
    "restricted": false
  }
}
```

**Postman Setup**:
- Method: `POST`
- URL: `http://localhost:8080/api/auth/register`
- Headers: `Content-Type: application/json`
- Body: Select `raw` → `JSON`, paste the request body above

---

### 1.2 Login
**POST** `/api/auth/login`

**Description**: Authenticate user and receive JWT token

**Headers**: None required

**Request Body**:
```json
{
  "username": "john_doe",
  "password": "password123"
}
```

**Response** (200 OK):
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "username": "john_doe",
    "email": "john.doe@example.com",
    "role": "STUDENT",
    "restricted": false
  }
}
```

**Postman Setup**:
- Method: `POST`
- URL: `http://localhost:8080/api/auth/login`
- Headers: `Content-Type: application/json`
- Body: Select `raw` → `JSON`, paste the request body above

---

### 1.3 Health Check
**GET** `/api/auth/health`

**Response** (200 OK):
```
Auth Service is running!
```

---

## 2. User Service (`/api/users`)

### 2.1 Get Current User
**GET** `/api/users/me`

**Description**: Get current authenticated user information

**Headers**:
- `X-User-Id`: `1` (required)

**Response** (200 OK):
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john.doe@example.com",
  "role": "STUDENT",
  "restricted": false
}
```

**Postman Setup**:
- Method: `GET`
- URL: `http://localhost:8080/api/users/me`
- Headers: `X-User-Id: 1`

---

### 2.2 Get User by ID
**GET** `/api/users/{id}`

**Description**: Get user information by ID

**Headers**: `Authorization` header required (currently checks for presence only)

**Path Parameters**:
- `id`: User ID (e.g., `1`)

**Response** (200 OK):
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john.doe@example.com",
  "role": "STUDENT",
  "restricted": false
}
```

**Postman Setup**:
- Method: `GET`
- URL: `http://localhost:8080/api/users/1`
- Headers: `Authorization: Bearer <token>` (or any value)

---

### 2.3 Get User by Username
**GET** `/api/users/username/{username}`

**Description**: Get user information by username

**Headers**: `Authorization` header required

**Path Parameters**:
- `username`: Username (e.g., `john_doe`)

**Response** (200 OK):
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john.doe@example.com",
  "role": "STUDENT",
  "restricted": false
}
```

---

### 2.4 Get All Users
**GET** `/api/users`

**Description**: Get all users (admin only in production)

**Headers**: `Authorization` header required

**Response** (200 OK):
```json
[
  {
    "id": 1,
    "username": "john_doe",
    "email": "john.doe@example.com",
    "role": "STUDENT",
    "restricted": false
  },
  {
    "id": 2,
    "username": "jane_smith",
    "email": "jane.smith@example.com",
    "role": "FACULTY",
    "restricted": false
  }
]
```

---

### 2.5 Restrict User
**POST** `/api/users/{id}/restrict`

**Description**: Restrict a user account (admin only)

**Headers**: `Authorization` header required

**Path Parameters**:
- `id`: User ID

**Request Body**:
```json
{
  "reason": "Violation of library policies"
}
```

**Response** (200 OK):
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john.doe@example.com",
  "role": "STUDENT",
  "restricted": true,
  "restrictionReason": "Violation of library policies"
}
```

---

### 2.6 Unrestrict User
**POST** `/api/users/{id}/unrestrict`

**Description**: Remove restriction from a user account (admin only)

**Headers**: `Authorization` header required

**Path Parameters**:
- `id`: User ID

**Response** (200 OK):
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john.doe@example.com",
  "role": "STUDENT",
  "restricted": false
}
```

---

### 2.7 Check if User is Restricted
**GET** `/api/users/{id}/restricted`

**Description**: Check if a user account is restricted

**Headers**: `Authorization` header required

**Path Parameters**:
- `id`: User ID

**Response** (200 OK):
```json
{
  "restricted": false
}
```

---

## 3. Catalog Service (`/api/resources`)

### 3.1 Create Resource
**POST** `/api/resources`

**Description**: Create a new library resource (room, equipment, etc.)

**Headers**: `Authorization` header required

**Request Body**:
```json
{
  "name": "Study Room 101",
  "description": "Quiet study room with whiteboard",
  "type": "ROOM",
  "floor": 1,
  "capacity": 4,
  "status": "AVAILABLE"
}
```

**Valid Types**: `ROOM`, `EQUIPMENT`, `BOOK`
**Valid Status**: `AVAILABLE`, `OCCUPIED`, `MAINTENANCE`, `RESERVED`

**Response** (201 Created):
```json
{
  "id": 1,
  "name": "Study Room 101",
  "description": "Quiet study room with whiteboard",
  "type": "ROOM",
  "floor": 1,
  "capacity": 4,
  "status": "AVAILABLE",
  "createdAt": "2025-12-03T10:00:00"
}
```

---

### 3.2 Get Resource by ID
**GET** `/api/resources/{id}`

**Description**: Get resource information by ID

**Headers**: `Authorization` header required (except for health check)

**Path Parameters**:
- `id`: Resource ID

**Response** (200 OK):
```json
{
  "id": 1,
  "name": "Study Room 101",
  "description": "Quiet study room with whiteboard",
  "type": "ROOM",
  "floor": 1,
  "capacity": 4,
  "status": "AVAILABLE",
  "createdAt": "2025-12-03T10:00:00"
}
```

---

### 3.3 Get All Resources
**GET** `/api/resources`

**Description**: Get all resources with optional filters

**Headers**: `Authorization` header required (except for health check)

**Query Parameters** (all optional):
- `type`: Filter by type (`ROOM`, `EQUIPMENT`, `BOOK`)
- `floor`: Filter by floor number
- `status`: Filter by status (`AVAILABLE`, `OCCUPIED`, `MAINTENANCE`, `RESERVED`)
- `search`: Search by resource name

**Examples**:
- `GET /api/resources` - Get all resources
- `GET /api/resources?type=ROOM` - Get all rooms
- `GET /api/resources?floor=1&status=AVAILABLE` - Get available resources on floor 1
- `GET /api/resources?search=Study` - Search for resources with "Study" in name

**Response** (200 OK):
```json
[
  {
    "id": 1,
    "name": "Study Room 101",
    "type": "ROOM",
    "floor": 1,
    "capacity": 4,
    "status": "AVAILABLE"
  }
]
```

---

### 3.4 Update Resource
**PUT** `/api/resources/{id}`

**Description**: Update resource information

**Headers**: `Authorization` header required

**Path Parameters**:
- `id`: Resource ID

**Request Body**:
```json
{
  "name": "Study Room 101 - Updated",
  "description": "Updated description",
  "status": "MAINTENANCE"
}
```

**Response** (200 OK):
```json
{
  "id": 1,
  "name": "Study Room 101 - Updated",
  "description": "Updated description",
  "type": "ROOM",
  "floor": 1,
  "capacity": 4,
  "status": "MAINTENANCE"
}
```

---

### 3.5 Delete Resource
**DELETE** `/api/resources/{id}`

**Description**: Delete a resource

**Headers**: `Authorization` header required

**Path Parameters**:
- `id`: Resource ID

**Response** (204 No Content)

---

### 3.6 Health Check
**GET** `/api/resources/health`

**Response** (200 OK):
```
Catalog Service is running!
```

---

## 4. Booking Service (`/api/bookings`)

### 4.1 Create Booking
**POST** `/api/bookings`

**Description**: Create a new booking

**Headers**:
- `X-User-Id`: `1` (required)
- `Authorization`: Bearer token (required by gateway)

**Request Body**:
```json
{
  "resourceId": 1,
  "startTime": "2025-12-04T10:00:00",
  "endTime": "2025-12-04T12:00:00"
}
```

**Response** (201 Created):
```json
{
  "id": 1,
  "userId": 1,
  "resourceId": 1,
  "resourceName": "Study Room 101",
  "startTime": "2025-12-04T10:00:00",
  "endTime": "2025-12-04T12:00:00",
  "status": "CONFIRMED",
  "qrCode": "BOOKING-1-ABC123",
  "createdAt": "2025-12-03T10:00:00"
}
```

**Postman Setup**:
- Method: `POST`
- URL: `http://localhost:8080/api/bookings`
- Headers:
  - `X-User-Id: 1`
  - `Authorization: Bearer <any-token>`
  - `Content-Type: application/json`
- Body: Select `raw` → `JSON`, paste the request body above

---

### 4.2 Get All Bookings
**GET** `/api/bookings`

**Description**: Get all bookings

**Headers**: `Authorization` header required

**Response** (200 OK):
```json
[
  {
    "id": 1,
    "userId": 1,
    "resourceId": 1,
    "resourceName": "Study Room 101",
    "startTime": "2025-12-04T10:00:00",
    "endTime": "2025-12-04T12:00:00",
    "status": "CONFIRMED"
  }
]
```

---

### 4.3 Get Booking by ID
**GET** `/api/bookings/{id}`

**Description**: Get booking information by ID

**Headers**: `Authorization` header required

**Path Parameters**:
- `id`: Booking ID

**Response** (200 OK):
```json
{
  "id": 1,
  "userId": 1,
  "resourceId": 1,
  "resourceName": "Study Room 101",
  "startTime": "2025-12-04T10:00:00",
  "endTime": "2025-12-04T12:00:00",
  "status": "CONFIRMED",
  "qrCode": "BOOKING-1-ABC123"
}
```

---

### 4.4 Get Bookings by User ID
**GET** `/api/bookings/user/{userId}`

**Description**: Get all bookings for a specific user

**Headers**: `Authorization` header required

**Path Parameters**:
- `userId`: User ID

**Response** (200 OK):
```json
[
  {
    "id": 1,
    "userId": 1,
    "resourceId": 1,
    "resourceName": "Study Room 101",
    "startTime": "2025-12-04T10:00:00",
    "endTime": "2025-12-04T12:00:00",
    "status": "CONFIRMED"
  }
]
```

---

### 4.5 Get Bookings by Resource ID
**GET** `/api/bookings/resource/{resourceId}`

**Description**: Get all bookings for a specific resource

**Headers**: `Authorization` header required

**Path Parameters**:
- `resourceId`: Resource ID

**Response** (200 OK):
```json
[
  {
    "id": 1,
    "userId": 1,
    "resourceId": 1,
    "resourceName": "Study Room 101",
    "startTime": "2025-12-04T10:00:00",
    "endTime": "2025-12-04T12:00:00",
    "status": "CONFIRMED"
  }
]
```

---

### 4.6 Update Booking
**PUT** `/api/bookings/{id}`

**Description**: Update booking details

**Headers**:
- `X-User-Id`: `1` (required)
- `Authorization`: Bearer token (required)

**Path Parameters**:
- `id`: Booking ID

**Request Body**:
```json
{
  "startTime": "2025-12-04T11:00:00",
  "endTime": "2025-12-04T13:00:00"
}
```

**Response** (200 OK):
```json
{
  "id": 1,
  "userId": 1,
  "resourceId": 1,
  "startTime": "2025-12-04T11:00:00",
  "endTime": "2025-12-04T13:00:00",
  "status": "CONFIRMED"
}
```

---

### 4.7 Cancel Booking
**DELETE** `/api/bookings/{id}`

**Description**: Cancel a booking

**Headers**:
- `X-User-Id`: `1` (required)
- `Authorization`: Bearer token (required)

**Path Parameters**:
- `id`: Booking ID

**Response** (204 No Content)

---

### 4.8 Check-In
**POST** `/api/bookings/checkin`

**Description**: Check-in to a booking using QR code

**Headers**: `Authorization` header required

**Request Body**:
```json
{
  "qrCode": "BOOKING-1-ABC123"
}
```

**Response** (200 OK):
```json
{
  "id": 1,
  "userId": 1,
  "resourceId": 1,
  "status": "CHECKED_IN",
  "checkedInAt": "2025-12-04T10:05:00"
}
```

---

### 4.9 Health Check
**GET** `/api/bookings/health`

**Response** (200 OK):
```
Booking Service is running!
```

---

## 5. Policy Service (`/api/policies`)

### 5.1 Create Policy
**POST** `/api/policies`

**Description**: Create a new booking policy

**Headers**: `Authorization` header required

**Request Body**:
```json
{
  "name": "Maximum Booking Duration",
  "description": "Users can book resources for maximum 4 hours",
  "ruleType": "MAX_DURATION_HOURS",
  "ruleValue": "4",
  "active": true
}
```

**Response** (201 Created):
```json
{
  "id": 1,
  "name": "Maximum Booking Duration",
  "description": "Users can book resources for maximum 4 hours",
  "ruleType": "MAX_DURATION_HOURS",
  "ruleValue": "4",
  "active": true,
  "createdAt": "2025-12-03T10:00:00"
}
```

---

### 5.2 Get Policy by ID
**GET** `/api/policies/{id}`

**Description**: Get policy information by ID

**Headers**: `Authorization` header required (except for health check)

**Path Parameters**:
- `id`: Policy ID

**Response** (200 OK):
```json
{
  "id": 1,
  "name": "Maximum Booking Duration",
  "description": "Users can book resources for maximum 4 hours",
  "ruleType": "MAX_DURATION_HOURS",
  "ruleValue": "4",
  "active": true
}
```

---

### 5.3 Get All Policies
**GET** `/api/policies`

**Description**: Get all policies

**Headers**: `Authorization` header required (except for health check)

**Query Parameters** (optional):
- `active`: Filter by active status (`true`/`false`)

**Examples**:
- `GET /api/policies` - Get all policies
- `GET /api/policies?active=true` - Get only active policies

**Response** (200 OK):
```json
[
  {
    "id": 1,
    "name": "Maximum Booking Duration",
    "ruleType": "MAX_DURATION_HOURS",
    "ruleValue": "4",
    "active": true
  }
]
```

---

### 5.4 Update Policy
**PUT** `/api/policies/{id}`

**Description**: Update policy information

**Headers**: `Authorization` header required

**Path Parameters**:
- `id`: Policy ID

**Request Body**:
```json
{
  "name": "Updated Policy Name",
  "description": "Updated description",
  "active": false
}
```

**Response** (200 OK):
```json
{
  "id": 1,
  "name": "Updated Policy Name",
  "description": "Updated description",
  "active": false
}
```

---

### 5.5 Delete Policy
**DELETE** `/api/policies/{id}`

**Description**: Delete a policy

**Headers**: `Authorization` header required

**Path Parameters**:
- `id`: Policy ID

**Response** (204 No Content)

---

### 5.6 Validate Booking
**POST** `/api/policies/validate`

**Description**: Validate a booking request against policies

**Headers**: `Authorization` header required

**Request Body**:
```json
{
  "userId": 1,
  "resourceId": 1,
  "startTime": "2025-12-04T10:00:00",
  "endTime": "2025-12-04T14:00:00"
}
```

**Response** (200 OK):
```json
{
  "valid": true,
  "message": "Booking request is valid"
}
```

Or if invalid:
```json
{
  "valid": false,
  "message": "Booking duration exceeds maximum allowed duration of 4 hours"
}
```

---

### 5.7 Health Check
**GET** `/api/policies/health`

**Response** (200 OK):
```
Policy Service is running!
```

---

## 6. Notification Service (`/api/notifications`)

### 6.1 Get Notifications by User ID
**GET** `/api/notifications/user/{userId}`

**Description**: Get all notifications for a user

**Headers**: `Authorization` header required (except for health check)

**Path Parameters**:
- `userId`: User ID

**Response** (200 OK):
```json
[
  {
    "id": 1,
    "userId": 1,
    "type": "BOOKING_CONFIRMED",
    "title": "Booking Confirmed",
    "message": "Your booking for Study Room 101 has been confirmed",
    "read": false,
    "createdAt": "2025-12-03T10:00:00"
  }
]
```

---

### 6.2 Get Unread Notifications
**GET** `/api/notifications/user/{userId}/unread`

**Description**: Get unread notifications for a user

**Headers**: `Authorization` header required

**Path Parameters**:
- `userId`: User ID

**Response** (200 OK):
```json
[
  {
    "id": 1,
    "userId": 1,
    "type": "BOOKING_CONFIRMED",
    "title": "Booking Confirmed",
    "message": "Your booking has been confirmed",
    "read": false
  }
]
```

---

### 6.3 Get Unread Count
**GET** `/api/notifications/user/{userId}/unread/count`

**Description**: Get count of unread notifications

**Headers**: `Authorization` header required

**Path Parameters**:
- `userId`: User ID

**Response** (200 OK):
```json
{
  "count": 3
}
```

---

### 6.4 Mark Notification as Read
**PUT** `/api/notifications/{id}/read`

**Description**: Mark a notification as read

**Headers**: `Authorization` header required

**Path Parameters**:
- `id`: Notification ID

**Response** (200 OK):
```json
{
  "id": 1,
  "userId": 1,
  "type": "BOOKING_CONFIRMED",
  "title": "Booking Confirmed",
  "message": "Your booking has been confirmed",
  "read": true
}
```

---

### 6.5 Mark All as Read
**PUT** `/api/notifications/user/{userId}/read-all`

**Description**: Mark all notifications as read for a user

**Headers**: `Authorization` header required

**Path Parameters**:
- `userId`: User ID

**Response** (204 No Content)

---

### 6.6 Health Check
**GET** `/api/notifications/health`

**Response** (200 OK):
```
Notification Service is running!
```

---

## 7. Analytics Service (`/api/analytics`)

### 7.1 Get Utilization Statistics
**GET** `/api/analytics/utilization`

**Description**: Get resource utilization statistics

**Headers**: `Authorization` header required (except for health check)

**Query Parameters** (required):
- `startDate`: Start date in ISO format (e.g., `2025-12-01`)
- `endDate`: End date in ISO format (e.g., `2025-12-31`)
- `resourceId`: Resource ID (optional)

**Example**:
```
GET /api/analytics/utilization?startDate=2025-12-01&endDate=2025-12-31&resourceId=1
```

**Response** (200 OK):
```json
[
  {
    "resourceId": 1,
    "resourceName": "Study Room 101",
    "totalBookings": 45,
    "totalHours": 180,
    "utilizationRate": 0.75
  }
]
```

---

### 7.2 Get Peak Hours
**GET** `/api/analytics/peak-hours`

**Description**: Get peak usage hours

**Headers**: `Authorization` header required

**Query Parameters** (required):
- `startTime`: Start datetime in ISO format (e.g., `2025-12-01T00:00:00`)
- `endTime`: End datetime in ISO format (e.g., `2025-12-31T23:59:59`)

**Example**:
```
GET /api/analytics/peak-hours?startTime=2025-12-01T00:00:00&endTime=2025-12-31T23:59:59
```

**Response** (200 OK):
```json
{
  "peakHours": [10, 11, 14, 15],
  "peakDay": "MONDAY",
  "averageBookingsPerHour": 12.5
}
```

---

### 7.3 Get Overall Statistics
**GET** `/api/analytics/overall`

**Description**: Get overall system statistics

**Headers**: `Authorization` header required

**Query Parameters** (required):
- `startTime`: Start datetime in ISO format
- `endTime`: End datetime in ISO format

**Example**:
```
GET /api/analytics/overall?startTime=2025-12-01T00:00:00&endTime=2025-12-31T23:59:59
```

**Response** (200 OK):
```json
{
  "totalBookings": 1250,
  "totalUsers": 150,
  "totalResources": 25,
  "averageBookingDuration": 2.5,
  "mostBookedResource": "Study Room 101"
}
```

---

### 7.4 Health Check
**GET** `/api/analytics/health`

**Response** (200 OK):
```
Analytics Service is running!
```

---

## Complete Testing Flow

### Step-by-Step Postman Flow

#### **Phase 1: Setup and Authentication**

1. **Register a New User**
   - Method: `POST`
   - URL: `http://localhost:8080/api/auth/register`
   - Body:
     ```json
     {
       "username": "testuser",
       "email": "test@example.com",
       "password": "password123",
       "role": "STUDENT"
     }
     ```
   - **Save the response**: Note the `user.id` and `token` from response
   - **Create Postman Variable**: 
     - `userId` = `1` (from response)
     - `token` = token from response

2. **Login** (Alternative to registration)
   - Method: `POST`
   - URL: `http://localhost:8080/api/auth/login`
   - Body:
     ```json
     {
       "username": "testuser",
       "password": "password123"
     }
     ```
   - **Update variables**: Save `userId` and `token` from response

#### **Phase 2: Create Resources (Admin/Setup)**

3. **Create a Resource (Room)**
   - Method: `POST`
   - URL: `http://localhost:8080/api/resources`
   - Headers:
     - `Authorization: Bearer {{token}}`
   - Body:
     ```json
     {
       "name": "Study Room 101",
       "description": "Quiet study room with whiteboard",
       "type": "ROOM",
       "floor": 1,
       "capacity": 4,
       "status": "AVAILABLE"
     }
     ```
   - **Save**: Note the `resourceId` from response (e.g., `1`)
   - **Create Variable**: `resourceId` = `1`

4. **Create Another Resource (Equipment)**
   - Method: `POST`
   - URL: `http://localhost:8080/api/resources`
   - Headers:
     - `Authorization: Bearer {{token}}`
   - Body:
     ```json
     {
       "name": "Projector 1",
       "description": "HD Projector",
       "type": "EQUIPMENT",
       "floor": 2,
       "capacity": 1,
       "status": "AVAILABLE"
     }
     ```

5. **Get All Resources**
   - Method: `GET`
   - URL: `http://localhost:8080/api/resources`
   - Headers:
     - `Authorization: Bearer {{token}}`
   - Verify resources were created

#### **Phase 3: Create Policies**

6. **Create a Booking Policy**
   - Method: `POST`
   - URL: `http://localhost:8080/api/policies`
   - Headers:
     - `Authorization: Bearer {{token}}`
   - Body:
     ```json
     {
       "name": "Maximum Booking Duration",
       "description": "Users can book resources for maximum 4 hours",
       "ruleType": "MAX_DURATION_HOURS",
       "ruleValue": "4",
       "active": true
     }
     ```

7. **Get Active Policies**
   - Method: `GET`
   - URL: `http://localhost:8080/api/policies?active=true`
   - Headers:
     - `Authorization: Bearer {{token}}`

#### **Phase 4: Create Bookings**

8. **Validate Booking Request**
   - Method: `POST`
   - URL: `http://localhost:8080/api/policies/validate`
   - Headers:
     - `Authorization: Bearer {{token}}`
   - Body:
     ```json
     {
       "userId": 1,
       "resourceId": 1,
       "startTime": "2025-12-04T10:00:00",
       "endTime": "2025-12-04T12:00:00"
     }
     ```
   - Verify validation passes

9. **Create a Booking**
   - Method: `POST`
   - URL: `http://localhost:8080/api/bookings`
   - Headers:
     - `X-User-Id: {{userId}}`
     - `Authorization: Bearer {{token}}`
   - Body:
     ```json
     {
       "resourceId": 1,
       "startTime": "2025-12-04T10:00:00",
       "endTime": "2025-12-04T12:00:00"
     }
     ```
   - **Save**: Note the `bookingId` and `qrCode` from response
   - **Create Variables**: 
     - `bookingId` = booking ID from response
     - `qrCode` = QR code from response

10. **Get User's Bookings**
    - Method: `GET`
    - URL: `http://localhost:8080/api/bookings/user/{{userId}}`
    - Headers:
      - `Authorization: Bearer {{token}}`
    - Verify your booking appears

11. **Get Booking by ID**
    - Method: `GET`
    - URL: `http://localhost:8080/api/bookings/{{bookingId}}`
    - Headers:
      - `Authorization: Bearer {{token}}`

#### **Phase 5: Check-In and Notifications**

12. **Check-In to Booking**
    - Method: `POST`
    - URL: `http://localhost:8080/api/bookings/checkin`
    - Headers:
      - `Authorization: Bearer {{token}}`
    - Body:
      ```json
      {
        "qrCode": "{{qrCode}}"
      }
      ```
    - Verify status changes to `CHECKED_IN`

13. **Get User Notifications**
    - Method: `GET`
    - URL: `http://localhost:8080/api/notifications/user/{{userId}}`
    - Headers:
      - `Authorization: Bearer {{token}}`
    - Should see booking confirmation notification

14. **Get Unread Notification Count**
    - Method: `GET`
    - URL: `http://localhost:8080/api/notifications/user/{{userId}}/unread/count`
    - Headers:
      - `Authorization: Bearer {{token}}`

15. **Mark Notification as Read**
    - Method: `PUT`
    - URL: `http://localhost:8080/api/notifications/1/read`
    - Headers:
      - `Authorization: Bearer {{token}}`

#### **Phase 6: Update and Cancel**

16. **Update Booking**
    - Method: `PUT`
    - URL: `http://localhost:8080/api/bookings/{{bookingId}}`
    - Headers:
      - `X-User-Id: {{userId}}`
      - `Authorization: Bearer {{token}}`
    - Body:
      ```json
      {
        "startTime": "2025-12-04T11:00:00",
        "endTime": "2025-12-04T13:00:00"
      }
      ```

17. **Cancel Booking**
    - Method: `DELETE`
    - URL: `http://localhost:8080/api/bookings/{{bookingId}}`
    - Headers:
      - `X-User-Id: {{userId}}`
      - `Authorization: Bearer {{token}}`

#### **Phase 7: Analytics**

18. **Get Utilization Statistics**
    - Method: `GET`
    - URL: `http://localhost:8080/api/analytics/utilization?startDate=2025-12-01&endDate=2025-12-31`
    - Headers:
      - `Authorization: Bearer {{token}}`

19. **Get Overall Statistics**
    - Method: `GET`
    - URL: `http://localhost:8080/api/analytics/overall?startTime=2025-12-01T00:00:00&endTime=2025-12-31T23:59:59`
    - Headers:
      - `Authorization: Bearer {{token}}`

#### **Phase 8: User Management**

20. **Get Current User**
    - Method: `GET`
    - URL: `http://localhost:8080/api/users/me`
    - Headers:
      - `X-User-Id: {{userId}}`

21. **Restrict User** (Admin only)
    - Method: `POST`
    - URL: `http://localhost:8080/api/users/{{userId}}/restrict`
    - Headers:
      - `Authorization: Bearer {{token}}`
    - Body:
      ```json
      {
        "reason": "Test restriction"
      }
      ```

22. **Unrestrict User** (Admin only)
    - Method: `POST`
    - URL: `http://localhost:8080/api/users/{{userId}}/unrestrict`
    - Headers:
      - `Authorization: Bearer {{token}}`

---

## Postman Collection Setup

### Creating Environment Variables

1. Create a new environment in Postman named "Library System Local"
2. Add these variables:
   - `baseUrl`: `http://localhost:8080`
   - `userId`: `1` (will be updated after registration/login)
   - `token`: (will be updated after login)
   - `resourceId`: `1` (will be updated after creating resource)
   - `bookingId`: (will be updated after creating booking)
   - `qrCode`: (will be updated after creating booking)

### Using Variables in Requests

- URL: `{{baseUrl}}/api/auth/login`
- Headers: `Authorization: Bearer {{token}}`
- Body: Use `{{userId}}` in JSON where needed

### Pre-request Scripts (Optional)

For automatic token extraction after login:
```javascript
if (pm.response.code === 200) {
    var jsonData = pm.response.json();
    pm.environment.set("token", jsonData.token);
    pm.environment.set("userId", jsonData.user.id);
}
```

---

## Error Responses

### 400 Bad Request
```json
{
  "error": "Validation failed",
  "message": "Resource ID is required"
}
```

### 401 Unauthorized
```json
{
  "error": "Unauthorized",
  "message": "Authorization header is required"
}
```

### 404 Not Found
```json
{
  "error": "Not Found",
  "message": "Resource with ID 999 not found"
}
```

### 409 Conflict
```json
{
  "error": "Conflict",
  "message": "Username or email already exists"
}
```

### 500 Internal Server Error
```json
{
  "error": "Internal Server Error",
  "message": "An unexpected error occurred"
}
```

---

## Notes

1. **Authentication**: Currently uses `X-User-Id` header. In production, this should be extracted from JWT token.

2. **CORS**: All endpoints support CORS with `Access-Control-Allow-Origin: *`

3. **Health Checks**: All services have `/health` endpoints that don't require authentication

4. **Date Formats**: 
   - Dates: `YYYY-MM-DD` (e.g., `2025-12-04`)
   - DateTimes: `YYYY-MM-DDTHH:mm:ss` (e.g., `2025-12-04T10:00:00`)

5. **Gateway**: If API Gateway is running, use port `8080`. Otherwise, use direct service ports.

6. **Testing Tips**:
   - Start with health checks to verify services are running
   - Register/login first to get user ID and token
   - Create resources before creating bookings
   - Check notifications after creating bookings
   - Use Postman's Collection Runner for automated testing

---

## Quick Reference

| Service | Base Path | Port (Direct) |
|---------|-----------|---------------|
| Auth | `/api/auth` | 3002 |
| User | `/api/users` | 3001 |
| Catalog | `/api/resources` | 3003 |
| Booking | `/api/bookings` | 3004 |
| Policy | `/api/policies` | 3005 |
| Notification | `/api/notifications` | 3006 |
| Analytics | `/api/analytics` | 3007 |

---

**Last Updated**: December 2025
