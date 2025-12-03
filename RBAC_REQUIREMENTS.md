# Role-Based Access Control (RBAC) Requirements

## Overview

This document outlines the RBAC requirements for all API endpoints when the API Gateway is implemented. The JWT token contains a `role` claim that will be used for authorization.

**JWT Token Structure**:
```json
{
  "sub": "username",
  "role": "STUDENT|FACULTY|ADMIN",
  "userId": 1,
  "iat": 1701600000,
  "exp": 1701603600
}
```

**Available Roles**:
- `STUDENT` - Basic user, can create bookings and view own data
- `FACULTY` - Extended privileges, may have longer booking durations
- `ADMIN` - Full system access, can manage users, resources, and policies

---

## Authorization Levels

### Public Endpoints
- **No authentication required**
- Health checks and authentication endpoints

### Authenticated Endpoints
- **Any logged-in user** (STUDENT, FACULTY, or ADMIN)
- Requires valid JWT token

### Role-Specific Endpoints
- **STUDENT** - Basic user operations
- **FACULTY** - Extended user operations
- **ADMIN** - Administrative operations

### Resource Ownership
- Users can only access/modify their own resources
- Admins can access all resources
- Additional checks: `userId` from JWT must match resource owner

---

## 1. Authentication Service (`/api/auth`)

### 1.1 Register User
**POST** `/api/auth/register`

**Authorization**: `PUBLIC` (No authentication required)

**Notes**: 
- Anyone can register
- Default role is `STUDENT`
- Only admins should be able to register with `FACULTY` or `ADMIN` roles (consider adding admin-only endpoint for user creation)

---

### 1.2 Login
**POST** `/api/auth/login`

**Authorization**: `PUBLIC` (No authentication required)

**Notes**: 
- Returns JWT token with user role

---

### 1.3 Health Check
**GET** `/api/auth/health`

**Authorization**: `PUBLIC` (No authentication required)

---

## 2. User Service (`/api/users`)

### 2.1 Get Current User
**GET** `/api/users/me`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Returns user data for the authenticated user (from JWT `userId`)

---

### 2.2 Get User by ID
**GET** `/api/users/{id}`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can view their own profile
- Admins can view any user profile
- **Enforcement**: Check if `userId` from JWT matches `{id}` OR role is `ADMIN`

---

### 2.3 Get User by Username
**GET** `/api/users/username/{username}`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can view their own profile
- Admins can view any user profile
- **Enforcement**: Check if username from JWT matches `{username}` OR role is `ADMIN`

---

### 2.4 Get All Users
**GET** `/api/users`

**Authorization**: `ADMIN` only

**Notes**: 
- Comment in code: "admin only in production"
- Should return 403 Forbidden for non-admin users

---

### 2.5 Restrict User
**POST** `/api/users/{id}/restrict`

**Authorization**: `ADMIN` only

**Notes**: 
- Comment in code: "admin only"
- Should return 403 Forbidden for non-admin users

---

### 2.6 Unrestrict User
**POST** `/api/users/{id}/unrestrict`

**Authorization**: `ADMIN` only

**Notes**: 
- Comment in code: "admin only"
- Should return 403 Forbidden for non-admin users

---

### 2.7 Check if User is Restricted
**GET** `/api/users/{id}/restricted`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can check their own restriction status
- Admins can check any user's restriction status
- **Enforcement**: Check if `userId` from JWT matches `{id}` OR role is `ADMIN`

---

## 3. Catalog Service (`/api/resources`)

### 3.1 Create Resource
**POST** `/api/resources`

**Authorization**: `ADMIN` only

**Notes**: 
- Only admins should be able to create resources (rooms, equipment, books)
- Should return 403 Forbidden for non-admin users

---

### 3.2 Get Resource by ID
**GET** `/api/resources/{id}`

**Authorization**: `AUTHENTICATED` (Any role)

**Notes**: 
- All authenticated users can view resource details
- Public information (availability, capacity, etc.)

---

### 3.3 Get All Resources
**GET** `/api/resources`

**Authorization**: `AUTHENTICATED` (Any role)

**Query Parameters**: 
- `type`, `floor`, `status`, `search` - all accessible to authenticated users

**Notes**: 
- All authenticated users can browse/search resources
- Public catalog information

---

### 3.4 Update Resource
**PUT** `/api/resources/{id}`

**Authorization**: `ADMIN` only

**Notes**: 
- Only admins should be able to update resource information
- Should return 403 Forbidden for non-admin users

---

### 3.5 Delete Resource
**DELETE** `/api/resources/{id}`

**Authorization**: `ADMIN` only

**Notes**: 
- Only admins should be able to delete resources
- Should return 403 Forbidden for non-admin users

---

### 3.6 Health Check
**GET** `/api/resources/health`

**Authorization**: `PUBLIC` (No authentication required)

---

## 4. Booking Service (`/api/bookings`)

### 4.1 Create Booking
**POST** `/api/bookings`

**Authorization**: `AUTHENTICATED` (STUDENT, FACULTY, or ADMIN)

**Resource Ownership**: 
- Users can only create bookings for themselves
- **Enforcement**: `userId` from JWT must be used (not from request body)
- **Additional**: Check if user is restricted (should not be able to book if restricted)

**Notes**: 
- Policy validation should occur before booking creation
- Different booking limits may apply based on role (e.g., FACULTY may have longer durations)

---

### 4.2 Get All Bookings
**GET** `/api/bookings`

**Authorization**: `ADMIN` only

**Notes**: 
- Only admins should see all bookings system-wide
- Regular users should use `/api/bookings/user/{userId}` to see their own bookings
- Should return 403 Forbidden for non-admin users

---

### 4.3 Get Booking by ID
**GET** `/api/bookings/{id}`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can view their own bookings
- Admins can view any booking
- **Enforcement**: Check if booking's `userId` matches JWT `userId` OR role is `ADMIN`

---

### 4.4 Get Bookings by User ID
**GET** `/api/bookings/user/{userId}`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can only view their own bookings
- Admins can view any user's bookings
- **Enforcement**: Check if `userId` from JWT matches `{userId}` OR role is `ADMIN`

---

### 4.5 Get Bookings by Resource ID
**GET** `/api/bookings/resource/{resourceId}`

**Authorization**: `AUTHENTICATED` (Any role)

**Notes**: 
- All authenticated users can see bookings for a resource
- Useful for checking availability
- Public information (doesn't expose sensitive user data)

---

### 4.6 Update Booking
**PUT** `/api/bookings/{id}`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can only update their own bookings
- Admins can update any booking
- **Enforcement**: Check if booking's `userId` matches JWT `userId` OR role is `ADMIN`
- **Additional**: Check if booking can still be modified (e.g., not already checked in or past start time)

---

### 4.7 Cancel Booking
**DELETE** `/api/bookings/{id}`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can only cancel their own bookings
- Admins can cancel any booking
- **Enforcement**: Check if booking's `userId` matches JWT `userId` OR role is `ADMIN`
- **Additional**: Check if booking can still be cancelled (e.g., not already checked in)

---

### 4.8 Check-In
**POST** `/api/bookings/checkin`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can check in to their own bookings
- Admins can check in to any booking
- **Enforcement**: Validate QR code and check if booking's `userId` matches JWT `userId` OR role is `ADMIN`

**Notes**: 
- QR code validation should verify the booking belongs to the user
- May be used by library staff (consider adding STAFF role or allowing FACULTY/ADMIN)

---

### 4.9 Health Check
**GET** `/api/bookings/health`

**Authorization**: `PUBLIC` (No authentication required)

---

## 5. Policy Service (`/api/policies`)

### 5.1 Create Policy
**POST** `/api/policies`

**Authorization**: `ADMIN` only

**Notes**: 
- Only admins should be able to create booking policies
- Should return 403 Forbidden for non-admin users

---

### 5.2 Get Policy by ID
**GET** `/api/policies/{id}`

**Authorization**: `AUTHENTICATED` (Any role)

**Notes**: 
- All authenticated users can view policy details
- Public information (rules, limits, etc.)

---

### 5.3 Get All Policies
**GET** `/api/policies`

**Authorization**: `AUTHENTICATED` (Any role)

**Query Parameters**: 
- `active=true` - filter for active policies

**Notes**: 
- All authenticated users can view policies
- Public information

---

### 5.4 Update Policy
**PUT** `/api/policies/{id}`

**Authorization**: `ADMIN` only

**Notes**: 
- Only admins should be able to update policies
- Should return 403 Forbidden for non-admin users

---

### 5.5 Delete Policy
**DELETE** `/api/policies/{id}`

**Authorization**: `ADMIN` only

**Notes**: 
- Only admins should be able to delete policies
- Should return 403 Forbidden for non-admin users

---

### 5.6 Validate Booking
**POST** `/api/policies/validate`

**Authorization**: `AUTHENTICATED` (Any role)

**Notes**: 
- All authenticated users can validate booking requests
- Used before creating bookings
- May return different validation results based on user role (e.g., FACULTY may have different limits)

---

### 5.7 Health Check
**GET** `/api/policies/health`

**Authorization**: `PUBLIC` (No authentication required)

---

## 6. Notification Service (`/api/notifications`)

### 6.1 Get Notifications by User ID
**GET** `/api/notifications/user/{userId}`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can only view their own notifications
- Admins can view any user's notifications
- **Enforcement**: Check if `userId` from JWT matches `{userId}` OR role is `ADMIN`

---

### 6.2 Get Unread Notifications
**GET** `/api/notifications/user/{userId}/unread`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can only view their own unread notifications
- Admins can view any user's unread notifications
- **Enforcement**: Check if `userId` from JWT matches `{userId}` OR role is `ADMIN`

---

### 6.3 Get Unread Count
**GET** `/api/notifications/user/{userId}/unread/count`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can only view their own unread count
- Admins can view any user's unread count
- **Enforcement**: Check if `userId` from JWT matches `{userId}` OR role is `ADMIN`

---

### 6.4 Mark Notification as Read
**PUT** `/api/notifications/{id}/read`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can only mark their own notifications as read
- Admins can mark any notification as read
- **Enforcement**: Check if notification's `userId` matches JWT `userId` OR role is `ADMIN`

---

### 6.5 Mark All as Read
**PUT** `/api/notifications/user/{userId}/read-all`

**Authorization**: `AUTHENTICATED` (Any role)

**Resource Ownership**: 
- Users can only mark their own notifications as read
- Admins can mark any user's notifications as read
- **Enforcement**: Check if `userId` from JWT matches `{userId}` OR role is `ADMIN`

---

### 6.6 Health Check
**GET** `/api/notifications/health`

**Authorization**: `PUBLIC` (No authentication required)

---

## 7. Analytics Service (`/api/analytics`)

### 7.1 Get Utilization Statistics
**GET** `/api/analytics/utilization`

**Authorization**: `ADMIN` or `FACULTY`

**Notes**: 
- Analytics data should be restricted to administrators and faculty
- Students should not have access to system-wide analytics
- Should return 403 Forbidden for STUDENT role

---

### 7.2 Get Peak Hours
**GET** `/api/analytics/peak-hours`

**Authorization**: `ADMIN` or `FACULTY`

**Notes**: 
- Analytics data should be restricted to administrators and faculty
- Should return 403 Forbidden for STUDENT role

---

### 7.3 Get Overall Statistics
**GET** `/api/analytics/overall`

**Authorization**: `ADMIN` only

**Notes**: 
- System-wide statistics should be admin-only
- Most sensitive analytics endpoint
- Should return 403 Forbidden for non-admin users

---

### 7.4 Health Check
**GET** `/api/analytics/health`

**Authorization**: `PUBLIC` (No authentication required)

---

## Summary Table

| Service | Endpoint | Method | Authorization Level | Resource Ownership Check |
|---------|----------|--------|---------------------|-------------------------|
| **Auth Service** |
| `/api/auth/register` | POST | PUBLIC | N/A |
| `/api/auth/login` | POST | PUBLIC | N/A |
| `/api/auth/health` | GET | PUBLIC | N/A |
| **User Service** |
| `/api/users/me` | GET | AUTHENTICATED | Self only |
| `/api/users/{id}` | GET | AUTHENTICATED | Self or Admin |
| `/api/users/username/{username}` | GET | AUTHENTICATED | Self or Admin |
| `/api/users` | GET | ADMIN | N/A |
| `/api/users/{id}/restrict` | POST | ADMIN | N/A |
| `/api/users/{id}/unrestrict` | POST | ADMIN | N/A |
| `/api/users/{id}/restricted` | GET | AUTHENTICATED | Self or Admin |
| **Catalog Service** |
| `/api/resources` | POST | ADMIN | N/A |
| `/api/resources/{id}` | GET | AUTHENTICATED | N/A |
| `/api/resources` | GET | AUTHENTICATED | N/A |
| `/api/resources/{id}` | PUT | ADMIN | N/A |
| `/api/resources/{id}` | DELETE | ADMIN | N/A |
| `/api/resources/health` | GET | PUBLIC | N/A |
| **Booking Service** |
| `/api/bookings` | POST | AUTHENTICATED | Self only |
| `/api/bookings` | GET | ADMIN | N/A |
| `/api/bookings/{id}` | GET | AUTHENTICATED | Self or Admin |
| `/api/bookings/user/{userId}` | GET | AUTHENTICATED | Self or Admin |
| `/api/bookings/resource/{resourceId}` | GET | AUTHENTICATED | N/A |
| `/api/bookings/{id}` | PUT | AUTHENTICATED | Self or Admin |
| `/api/bookings/{id}` | DELETE | AUTHENTICATED | Self or Admin |
| `/api/bookings/checkin` | POST | AUTHENTICATED | Self or Admin |
| `/api/bookings/health` | GET | PUBLIC | N/A |
| **Policy Service** |
| `/api/policies` | POST | ADMIN | N/A |
| `/api/policies/{id}` | GET | AUTHENTICATED | N/A |
| `/api/policies` | GET | AUTHENTICATED | N/A |
| `/api/policies/{id}` | PUT | ADMIN | N/A |
| `/api/policies/{id}` | DELETE | ADMIN | N/A |
| `/api/policies/validate` | POST | AUTHENTICATED | N/A |
| `/api/policies/health` | GET | PUBLIC | N/A |
| **Notification Service** |
| `/api/notifications/user/{userId}` | GET | AUTHENTICATED | Self or Admin |
| `/api/notifications/user/{userId}/unread` | GET | AUTHENTICATED | Self or Admin |
| `/api/notifications/user/{userId}/unread/count` | GET | AUTHENTICATED | Self or Admin |
| `/api/notifications/{id}/read` | PUT | AUTHENTICATED | Self or Admin |
| `/api/notifications/user/{userId}/read-all` | PUT | AUTHENTICATED | Self or Admin |
| `/api/notifications/health` | GET | PUBLIC | N/A |
| **Analytics Service** |
| `/api/analytics/utilization` | GET | ADMIN or FACULTY | N/A |
| `/api/analytics/peak-hours` | GET | ADMIN or FACULTY | N/A |
| `/api/analytics/overall` | GET | ADMIN | N/A |
| `/api/analytics/health` | GET | PUBLIC | N/A |

---

## Implementation Notes

### 1. JWT Token Validation
- API Gateway should validate JWT token signature and expiration
- Extract `role` and `userId` from token claims
- Pass `userId` and `role` to downstream services via headers (e.g., `X-User-Id`, `X-User-Role`)

### 2. Role-Based Checks
- Implement role checks in API Gateway before routing to services
- Return `403 Forbidden` for unauthorized role access
- Example: `if (role !== 'ADMIN') return 403`

### 3. Resource Ownership Checks
- Services should validate that `userId` from JWT matches resource owner
- Admins should bypass ownership checks
- Return `403 Forbidden` for unauthorized resource access

### 4. User Restriction Checks
- Before allowing booking creation/updates, check if user is restricted
- Query user-service to verify user status
- Return `403 Forbidden` if user is restricted

### 5. Policy-Based Authorization
- Different booking limits may apply based on role
- FACULTY may have extended booking durations
- Implement in policy-service validation logic

### 6. Error Responses
- `401 Unauthorized`: Missing or invalid JWT token
- `403 Forbidden`: Valid token but insufficient permissions (wrong role or not resource owner)
- `404 Not Found`: Resource doesn't exist or user doesn't have access

### 7. Gateway Implementation
- Validate JWT token on all protected routes
- Extract and forward `userId` and `role` to services
- Implement role-based routing/blocking
- Log authorization failures for security monitoring

---

## Role Permissions Matrix

| Permission | STUDENT | FACULTY | ADMIN |
|------------|---------|---------|-------|
| Register/Login | ✅ | ✅ | ✅ |
| View own profile | ✅ | ✅ | ✅ |
| View own bookings | ✅ | ✅ | ✅ |
| Create bookings | ✅ | ✅ | ✅ |
| Update own bookings | ✅ | ✅ | ✅ |
| Cancel own bookings | ✅ | ✅ | ✅ |
| View resources catalog | ✅ | ✅ | ✅ |
| View policies | ✅ | ✅ | ✅ |
| View own notifications | ✅ | ✅ | ✅ |
| View all users | ❌ | ❌ | ✅ |
| Restrict/unrestrict users | ❌ | ❌ | ✅ |
| Create/update/delete resources | ❌ | ❌ | ✅ |
| View all bookings | ❌ | ❌ | ✅ |
| Create/update/delete policies | ❌ | ❌ | ✅ |
| View analytics (utilization/peak) | ❌ | ✅ | ✅ |
| View overall statistics | ❌ | ❌ | ✅ |
| Extended booking duration | ❌ | ✅ | ✅ |

---

**Last Updated**: December 2025  
**Status**: Ready for API Gateway Implementation
