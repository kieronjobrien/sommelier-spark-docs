# API Specification — Sommelier Spark

## Document Information

| Field | Value |
|-------|-------|
| **Document ID** | SS-WS3-API |
| **Version** | 1.0 |
| **Date** | 2026-01-21 |
| **Author** | Obi Wan |
| **Status** | DRAFT |
| **Classification** | CONFIDENTIAL |
| **Related Documents** | SS-WS3-HLD, SS-WS3-LLD, SS-WS3-EARS, SS-WS3.0-CDM, SS-WS3.0-ORG |

---

## CONFIDENTIALITY NOTICE

This document contains proprietary and confidential API specifications for the Sommelier Spark platform. Distribution is restricted to authorised personnel only.

---

## Table of Contents

1. [API Overview](#1-api-overview)
2. [Endpoint Summary](#2-endpoint-summary)
3. [Authentication Endpoints](#3-authentication-endpoints)
4. [User Endpoints](#4-user-endpoints)
5. [Organisation Endpoints](#5-organisation-endpoints)
6. [Wine Endpoints](#6-wine-endpoints)
7. [Module Endpoints](#7-module-endpoints)
8. [Quiz Endpoints](#8-quiz-endpoints)
9. [Scenario Endpoints](#9-scenario-endpoints)
10. [Progress Endpoints](#10-progress-endpoints)
11. [Import Endpoints](#11-import-endpoints)
12. [Report Endpoints](#12-report-endpoints)
13. [Curriculum Endpoints](#13-curriculum-endpoints)
14. [Common Schemas](#14-common-schemas)
15. [Error Handling](#15-error-handling)
16. [Webhooks](#16-webhooks)
17. [Appendices](#17-appendices)

---

## 1. API Overview

### 1.1 Base URL

| Environment | Base URL |
|-------------|----------|
| **Production** | `https://api.sommelierspark.com/api/v1` |
| **Staging** | `https://api-staging.sommelierspark.com/api/v1` |
| **Development** | `http://localhost:3000/api/v1` |

### 1.2 Authentication

All API endpoints (except authentication endpoints) require a valid JWT access token.

**Token Format:** Bearer Token in Authorization header

```http
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

**JWT Claims:**

| Claim | Description |
|-------|-------------|
| `sub` | User ID (UUID) |
| `email` | User email address |
| `organisationId` | Organisation ID (UUID) |
| `role` | User role (LEARNER, ADMIN, OWNER, SYSTEM_ADMIN) |
| `iat` | Issued at timestamp |
| `exp` | Expiration timestamp |
| `iss` | Issuer: `sommelier-spark` |
| `aud` | Audience: `sommelier-spark-api` |

**Token Lifetimes:**

| Token Type | Lifetime |
|------------|----------|
| Access Token | 1 hour |
| Refresh Token | 30 days |
| Session Timeout | 8 hours inactivity |

### 1.3 Rate Limiting

| Scope | Limit | Window |
|-------|-------|--------|
| Per User | 100 requests | 1 minute |
| Per Organisation | 1,000 requests | 1 minute |
| Enterprise Tier | 10,000 requests | 1 minute |

**Rate Limit Headers:**

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1705750200
Retry-After: 60
```

### 1.4 Versioning Strategy

- **Method:** URL path versioning (`/api/v1/`, `/api/v2/`)
- **Current Version:** v1
- **Deprecation Policy:** 6 months notice before version sunset
- **Breaking Changes:** Major version increment only
- **Non-Breaking Changes:** Additive changes within current version

### 1.5 Standard Response Format

**Success Response:**

```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "type": "wine",
    "attributes": { }
  },
  "meta": {
    "timestamp": "2026-01-21T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

**Paginated Response:**

```json
{
  "data": [ ],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "totalPages": 5,
    "totalItems": 100
  },
  "links": {
    "self": "/api/v1/wines?page=1",
    "first": "/api/v1/wines?page=1",
    "prev": null,
    "next": "/api/v1/wines?page=2",
    "last": "/api/v1/wines?page=5"
  }
}
```

**Error Response:**

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "name",
        "message": "Name is required"
      }
    ],
    "requestId": "req_abc123",
    "timestamp": "2026-01-21T10:30:00Z"
  }
}
```

### 1.6 Content Types

| Type | Media Type |
|------|------------|
| JSON (default) | `application/json` |
| File Upload | `multipart/form-data` |
| CSV Export | `text/csv` |
| PDF Export | `application/pdf` |

---

## 2. Endpoint Summary

### 2.1 Authentication

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/auth/login` | Authenticate user | None |
| POST | `/auth/logout` | Invalidate session | Required |
| POST | `/auth/refresh` | Refresh access token | Refresh Token |
| POST | `/auth/forgot-password` | Request password reset | None |
| POST | `/auth/reset-password` | Reset password with token | None |

### 2.2 Users

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/users` | List users | Manager |
| GET | `/users/:id` | Get user by ID | Manager |
| GET | `/users/me` | Get current user | User |
| POST | `/users` | Create user | Admin |
| PUT | `/users/:id` | Update user | Admin |
| DELETE | `/users/:id` | Delete user | Admin |
| POST | `/users/invite` | Invite user via email | Manager |
| POST | `/users/:id/activate` | Activate invited user | None |
| POST | `/users/:id/suspend` | Suspend user | Admin |
| POST | `/users/:id/reactivate` | Reactivate user | Admin |

### 2.3 Organisations

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/organisations/current` | Get current organisation | User |
| PUT | `/organisations/current` | Update organisation | Owner |
| PUT | `/organisations/current/settings` | Update settings | Owner |
| PUT | `/organisations/current/branding` | Update branding | Owner |

### 2.4 Wines

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/wines` | List wines | User |
| GET | `/wines/:id` | Get wine by ID | User |
| POST | `/wines` | Create wine | Author |
| PUT | `/wines/:id` | Update wine | Author |
| DELETE | `/wines/:id` | Delete wine | Admin |
| POST | `/wines/:id/submit` | Submit for review | Author |
| POST | `/wines/:id/approve` | Approve wine | Expert |
| POST | `/wines/:id/reject` | Reject wine | Expert |
| POST | `/wines/:id/publish` | Publish wine | Admin |
| POST | `/wines/:id/archive` | Archive wine | Admin |

### 2.5 Modules

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/modules` | List modules | User |
| GET | `/modules/:id` | Get module by ID | User |
| POST | `/modules` | Create module | Author |
| PUT | `/modules/:id` | Update module | Author |
| DELETE | `/modules/:id` | Delete module | Admin |
| GET | `/modules/:id/lessons` | List module lessons | User |
| POST | `/modules/:id/lessons` | Create lesson | Author |
| PUT | `/modules/:id/lessons/:lessonId` | Update lesson | Author |
| DELETE | `/modules/:id/lessons/:lessonId` | Delete lesson | Admin |
| PUT | `/modules/:id/lessons/reorder` | Reorder lessons | Author |

### 2.6 Quizzes

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/quizzes` | List quizzes | User |
| GET | `/quizzes/:id` | Get quiz by ID | User |
| POST | `/quizzes` | Create quiz | Author |
| PUT | `/quizzes/:id` | Update quiz | Author |
| DELETE | `/quizzes/:id` | Delete quiz | Admin |
| GET | `/quizzes/:id/preview` | Preview quiz | Author |
| POST | `/quizzes/:id/questions` | Add question | Author |
| PUT | `/quizzes/:id/questions/:questionId` | Update question | Author |
| DELETE | `/quizzes/:id/questions/:questionId` | Delete question | Author |
| POST | `/quizzes/:id/attempts` | Start quiz attempt | User |
| GET | `/quizzes/:id/attempts/:attemptId` | Get attempt details | User |
| POST | `/quizzes/:id/attempts/:attemptId/submit` | Submit quiz attempt | User |

### 2.7 Scenarios

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/scenarios` | List scenarios | User |
| GET | `/scenarios/:id` | Get scenario by ID | User |
| POST | `/scenarios` | Create scenario | Author |
| PUT | `/scenarios/:id` | Update scenario | Author |
| DELETE | `/scenarios/:id` | Delete scenario | Admin |
| GET | `/scenarios/:id/preview` | Preview scenario | Author |
| POST | `/scenarios/:id/attempts` | Start scenario attempt | User |
| POST | `/scenarios/:id/attempts/:attemptId/choices` | Submit choice | User |
| GET | `/scenarios/:id/attempts/:attemptId` | Get attempt status | User |

### 2.8 Progress

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/progress` | Get own progress | User |
| GET | `/progress/summary` | Get progress summary | User |
| GET | `/progress/users/:id` | Get user progress | Manager |
| POST | `/progress/lessons/:id/complete` | Mark lesson complete | User |
| GET | `/certifications` | Get certifications | User |

### 2.9 Imports

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/imports/wines` | Upload wine import | Manager |
| GET | `/imports/:id` | Get import status | Manager |
| GET | `/imports/:id/preview` | Preview import data | Manager |
| GET | `/imports/:id/errors` | Get import errors | Manager |
| POST | `/imports/:id/confirm` | Confirm import | Manager |
| POST | `/imports/:id/cancel` | Cancel import | Manager |
| GET | `/imports` | List imports | Manager |
| GET | `/templates/wines.csv` | Download wine template | Manager |

### 2.10 Reports

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/reports/individual/:userId` | Individual report | Manager |
| GET | `/reports/team` | Team report | Manager |
| GET | `/reports/organisation` | Organisation report | Admin |
| GET | `/reports/content` | Content report | Admin |
| POST | `/reports/export` | Export report | Manager |

### 2.11 Curriculum

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/curriculum/generate` | Generate curriculum | Admin |
| GET | `/curriculum/status` | Get generation status | Admin |
| GET | `/curriculum` | Get current curriculum | User |

---

## 3. Authentication Endpoints

### 3.1 POST /auth/login

Authenticate user with email and password.

**Request:**

```http
POST /api/v1/auth/login
Content-Type: application/json
```

```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!"
}
```

**Response (200 OK):**

```json
{
  "data": {
    "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...",
    "expiresIn": 3600,
    "tokenType": "Bearer",
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "email": "user@example.com",
      "name": "John Smith",
      "role": "LEARNER",
      "organisationId": "660e8400-e29b-41d4-a716-446655440001",
      "certificationLevel": "BRONZE"
    }
  }
}
```

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 400 | VALIDATION_ERROR | Invalid request body |
| 401 | INVALID_CREDENTIALS | Email or password incorrect |
| 403 | ACCOUNT_LOCKED | Account locked after failed attempts |
| 403 | ACCOUNT_SUSPENDED | Account has been suspended |

---

### 3.2 POST /auth/logout

Invalidate current session and blacklist tokens.

**Request:**

```http
POST /api/v1/auth/logout
Authorization: Bearer <access_token>
```

**Response (204 No Content):**

No response body.

---

### 3.3 POST /auth/refresh

Refresh an expired access token using a valid refresh token.

**Request:**

```http
POST /api/v1/auth/refresh
Content-Type: application/json
```

```json
{
  "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4..."
}
```

**Response (200 OK):**

```json
{
  "data": {
    "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "bmV3IHJlZnJlc2ggdG9rZW4...",
    "expiresIn": 3600,
    "tokenType": "Bearer"
  }
}
```

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 401 | INVALID_REFRESH_TOKEN | Refresh token is invalid or expired |
| 401 | TOKEN_REVOKED | Refresh token has been revoked |

---

### 3.4 POST /auth/forgot-password

Request a password reset email.

**Request:**

```http
POST /api/v1/auth/forgot-password
Content-Type: application/json
```

```json
{
  "email": "user@example.com"
}
```

**Response (200 OK):**

```json
{
  "data": {
    "message": "If an account exists with this email, a reset link has been sent."
  }
}
```

> Note: Always returns success to prevent email enumeration.

---

### 3.5 POST /auth/reset-password

Reset password using a valid reset token.

**Request:**

```http
POST /api/v1/auth/reset-password
Content-Type: application/json
```

```json
{
  "token": "reset_token_from_email",
  "password": "NewSecurePassword123!",
  "confirmPassword": "NewSecurePassword123!"
}
```

**Response (200 OK):**

```json
{
  "data": {
    "message": "Password has been reset successfully."
  }
}
```

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 400 | INVALID_TOKEN | Reset token is invalid or expired |
| 400 | PASSWORD_MISMATCH | Passwords do not match |
| 400 | PASSWORD_TOO_WEAK | Password does not meet requirements |

---

## 4. User Endpoints

### 4.1 GET /users

List users in the organisation.

**Authorization:** Manager, Admin, Owner

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Page number |
| pageSize | integer | 20 | Items per page (max: 100) |
| sort | string | createdAt:desc | Sort field and direction |
| status | string | - | Filter by status (ACTIVE, INVITED, SUSPENDED) |
| role | string | - | Filter by role |
| search | string | - | Search by name or email |

**Request:**

```http
GET /api/v1/users?page=1&pageSize=20&status=ACTIVE
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "email": "john.smith@example.com",
      "name": "John Smith",
      "firstName": "John",
      "lastName": "Smith",
      "role": "LEARNER",
      "jobTitle": "Server",
      "status": "ACTIVE",
      "certificationLevel": "BRONZE",
      "lastLoginAt": "2026-01-20T15:30:00Z",
      "createdAt": "2026-01-01T10:00:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "totalPages": 3,
    "totalItems": 45
  },
  "links": {
    "self": "/api/v1/users?page=1&pageSize=20",
    "first": "/api/v1/users?page=1&pageSize=20",
    "prev": null,
    "next": "/api/v1/users?page=2&pageSize=20",
    "last": "/api/v1/users?page=3&pageSize=20"
  }
}
```

---

### 4.2 GET /users/:id

Get a specific user by ID.

**Authorization:** Manager, Admin, Owner (own org only)

**Request:**

```http
GET /api/v1/users/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "john.smith@example.com",
    "name": "John Smith",
    "firstName": "John",
    "lastName": "Smith",
    "role": "LEARNER",
    "jobTitle": "Server",
    "department": "Restaurant",
    "hireDate": "2025-06-15",
    "status": "ACTIVE",
    "certificationLevel": "BRONZE",
    "avatarUrl": "https://cdn.sommelierspark.com/avatars/user123.jpg",
    "lastLoginAt": "2026-01-20T15:30:00Z",
    "createdAt": "2026-01-01T10:00:00Z",
    "updatedAt": "2026-01-15T12:00:00Z"
  }
}
```

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 404 | USER_NOT_FOUND | User does not exist |
| 403 | FORBIDDEN | Cannot access user from different organisation |

---

### 4.3 GET /users/me

Get the current authenticated user.

**Authorization:** Any authenticated user

**Request:**

```http
GET /api/v1/users/me
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "john.smith@example.com",
    "name": "John Smith",
    "firstName": "John",
    "lastName": "Smith",
    "role": "LEARNER",
    "jobTitle": "Server",
    "department": "Restaurant",
    "status": "ACTIVE",
    "certificationLevel": "BRONZE",
    "avatarUrl": "https://cdn.sommelierspark.com/avatars/user123.jpg",
    "organisation": {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "name": "The Ivy Collection",
      "subscriptionTier": "PROFESSIONAL"
    },
    "preferences": {
      "emailNotifications": true,
      "weeklyDigest": true,
      "timezone": "Europe/London"
    },
    "lastLoginAt": "2026-01-20T15:30:00Z"
  }
}
```

---

### 4.4 POST /users/invite

Invite a new user to the organisation.

**Authorization:** Manager, Admin, Owner

**Request:**

```http
POST /api/v1/users/invite
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "email": "newuser@example.com",
  "firstName": "Jane",
  "lastName": "Doe",
  "role": "LEARNER",
  "jobTitle": "Server",
  "department": "Restaurant"
}
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "email": "newuser@example.com",
    "name": "Jane Doe",
    "firstName": "Jane",
    "lastName": "Doe",
    "role": "LEARNER",
    "jobTitle": "Server",
    "department": "Restaurant",
    "status": "INVITED",
    "invitedAt": "2026-01-21T10:30:00Z",
    "invitedBy": "550e8400-e29b-41d4-a716-446655440000"
  },
  "meta": {
    "message": "Invitation email sent successfully"
  }
}
```

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 400 | VALIDATION_ERROR | Invalid request body |
| 409 | USER_EXISTS | User with this email already exists |
| 403 | USER_LIMIT_REACHED | Organisation user limit exceeded |

---

### 4.5 POST /users/:id/activate

Activate an invited user (called when user clicks invitation link).

**Authorization:** None (token-based)

**Request:**

```http
POST /api/v1/users/770e8400-e29b-41d4-a716-446655440002/activate
Content-Type: application/json
```

```json
{
  "invitationToken": "invitation_token_from_email",
  "password": "SecurePassword123!",
  "confirmPassword": "SecurePassword123!"
}
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "email": "newuser@example.com",
    "name": "Jane Doe",
    "status": "ACTIVE",
    "activatedAt": "2026-01-21T11:00:00Z"
  },
  "meta": {
    "message": "Account activated successfully. Please log in."
  }
}
```

---

### 4.6 PUT /users/:id

Update user details.

**Authorization:** Admin, Owner (for any user), User (for own profile)

**Request:**

```http
PUT /api/v1/users/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "firstName": "John",
  "lastName": "Smith-Jones",
  "jobTitle": "Head Server",
  "department": "Fine Dining"
}
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "john.smith@example.com",
    "name": "John Smith-Jones",
    "firstName": "John",
    "lastName": "Smith-Jones",
    "role": "LEARNER",
    "jobTitle": "Head Server",
    "department": "Fine Dining",
    "status": "ACTIVE",
    "updatedAt": "2026-01-21T10:30:00Z"
  }
}
```

---

### 4.7 DELETE /users/:id

Remove a user from the organisation.

**Authorization:** Admin, Owner

**Request:**

```http
DELETE /api/v1/users/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer <access_token>
```

**Response (204 No Content):**

No response body.

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 404 | USER_NOT_FOUND | User does not exist |
| 403 | CANNOT_DELETE_OWNER | Cannot delete organisation owner |
| 403 | CANNOT_DELETE_SELF | Cannot delete your own account |

---

## 5. Organisation Endpoints

### 5.1 GET /organisations/current

Get the current user's organisation.

**Authorization:** Any authenticated user

**Request:**

```http
GET /api/v1/organisations/current
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "name": "The Ivy Collection",
    "slug": "the-ivy-collection",
    "type": "HOSPITALITY_GROUP",
    "subscriptionTier": "PROFESSIONAL",
    "subscriptionStatus": "ACTIVE",
    "status": "ACTIVE",
    "settings": {
      "features": {
        "scenariosEnabled": true,
        "customContentEnabled": true,
        "apiAccessEnabled": false
      },
      "training": {
        "requiredLevel": "BRONZE",
        "deadlineDays": 30,
        "autoEnrollNewUsers": true
      },
      "notifications": {
        "emailEnabled": true,
        "weeklyDigest": true,
        "progressReminders": true
      }
    },
    "branding": {
      "logoUrl": "https://cdn.sommelierspark.com/logos/ivy.png",
      "primaryColor": "#1a365d",
      "secondaryColor": "#e2e8f0"
    },
    "userCount": 45,
    "maxUsers": 50,
    "createdAt": "2025-06-01T00:00:00Z"
  }
}
```

---

### 5.2 PUT /organisations/current

Update organisation details.

**Authorization:** Owner

**Request:**

```http
PUT /api/v1/organisations/current
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "name": "The Ivy Collection UK",
  "billingEmail": "billing@ivycollection.com"
}
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "name": "The Ivy Collection UK",
    "billingEmail": "billing@ivycollection.com",
    "updatedAt": "2026-01-21T10:30:00Z"
  }
}
```

---

### 5.3 PUT /organisations/current/settings

Update organisation settings.

**Authorization:** Owner

**Request:**

```http
PUT /api/v1/organisations/current/settings
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "training": {
    "requiredLevel": "SILVER",
    "deadlineDays": 45,
    "autoEnrollNewUsers": true,
    "mandatoryModules": ["wine-fundamentals", "food-pairing"]
  },
  "notifications": {
    "emailEnabled": true,
    "weeklyDigest": false,
    "progressReminders": true,
    "deadlineWarnings": true
  }
}
```

**Response (200 OK):**

```json
{
  "data": {
    "settings": {
      "training": {
        "requiredLevel": "SILVER",
        "deadlineDays": 45,
        "autoEnrollNewUsers": true,
        "mandatoryModules": ["wine-fundamentals", "food-pairing"]
      },
      "notifications": {
        "emailEnabled": true,
        "weeklyDigest": false,
        "progressReminders": true,
        "deadlineWarnings": true
      }
    },
    "updatedAt": "2026-01-21T10:30:00Z"
  }
}
```

---

### 5.4 PUT /organisations/current/branding

Update organisation branding.

**Authorization:** Owner (Professional/Enterprise tier only)

**Request:**

```http
PUT /api/v1/organisations/current/branding
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

```
logo: [binary file]
primaryColor: #1a365d
secondaryColor: #e2e8f0
```

**Response (200 OK):**

```json
{
  "data": {
    "branding": {
      "logoUrl": "https://cdn.sommelierspark.com/logos/ivy-new.png",
      "primaryColor": "#1a365d",
      "secondaryColor": "#e2e8f0"
    },
    "updatedAt": "2026-01-21T10:30:00Z"
  }
}
```

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 403 | FEATURE_NOT_AVAILABLE | Branding not available on Starter tier |
| 400 | INVALID_IMAGE | Invalid image format or size |

---

## 6. Wine Endpoints

### 6.1 GET /wines

List wines in the organisation.

**Authorization:** Any authenticated user

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Page number |
| pageSize | integer | 20 | Items per page (max: 100) |
| sort | string | name:asc | Sort field and direction |
| status | string | - | Filter by status (DRAFT, REVIEW, PUBLISHED, ARCHIVED) |
| wineType | string | - | Filter by wine type |
| region | string | - | Filter by region |
| country | string | - | Filter by country |
| priceTier | string | - | Filter by price tier |
| search | string | - | Full-text search |

**Request:**

```http
GET /api/v1/wines?page=1&pageSize=20&wineType=red&status=PUBLISHED
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "880e8400-e29b-41d4-a716-446655440003",
      "name": "Château Margaux 2015",
      "producer": "Château Margaux",
      "vintage": 2015,
      "region": "Margaux",
      "country": "FR",
      "wineType": "red",
      "grapeVarieties": ["Cabernet Sauvignon", "Merlot", "Petit Verdot"],
      "priceTier": "luxury",
      "status": "PUBLISHED",
      "version": 1,
      "createdAt": "2026-01-01T10:00:00Z",
      "updatedAt": "2026-01-15T12:00:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "totalPages": 5,
    "totalItems": 92
  },
  "links": {
    "self": "/api/v1/wines?page=1&pageSize=20",
    "first": "/api/v1/wines?page=1&pageSize=20",
    "prev": null,
    "next": "/api/v1/wines?page=2&pageSize=20",
    "last": "/api/v1/wines?page=5&pageSize=20"
  }
}
```

---

### 6.2 GET /wines/:id

Get a specific wine by ID.

**Authorization:** Any authenticated user

**Request:**

```http
GET /api/v1/wines/880e8400-e29b-41d4-a716-446655440003
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "880e8400-e29b-41d4-a716-446655440003",
    "name": "Château Margaux 2015",
    "producer": "Château Margaux",
    "vintage": 2015,
    "region": "Margaux",
    "country": "FR",
    "wineType": "red",
    "grapeVarieties": ["Cabernet Sauvignon", "Merlot", "Petit Verdot"],
    "priceTier": "luxury",
    "price": 450.00,
    "tastingNotes": "Deep ruby colour with aromas of blackcurrant, violet, and subtle oak.",
    "foodPairings": ["Beef Wellington", "Lamb", "Aged Cheese"],
    "servingTemperature": "16-18°C",
    "decantingTime": "2-3 hours",
    "quickFacts": {
      "tastingNotes": "Deep ruby colour with aromas of blackcurrant",
      "pairings": ["Red meat", "Game"],
      "keyPoints": ["First Growth Bordeaux", "Left Bank"],
      "pronunciation": "sha-TOH mar-GOH",
      "studyTips": "Remember: Margaux is known for elegance"
    },
    "detailedProfile": {
      "appearance": "Deep ruby with violet reflections",
      "nose": "Complex bouquet of blackcurrant, violet, graphite",
      "palate": "Full-bodied with silky tannins",
      "production": "Traditional Bordeaux methods"
    },
    "expertInsights": {
      "history": "First classified in 1855",
      "terroir": "Gravel soils on the left bank",
      "aging": "Drink 2025-2050"
    },
    "imageUrl": "https://cdn.sommelierspark.com/wines/margaux2015.jpg",
    "status": "PUBLISHED",
    "version": 1,
    "createdAt": "2026-01-01T10:00:00Z",
    "updatedAt": "2026-01-15T12:00:00Z",
    "publishedAt": "2026-01-10T09:00:00Z",
    "createdBy": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "John Smith"
    },
    "_links": {
      "self": "/api/v1/wines/880e8400-e29b-41d4-a716-446655440003",
      "quizzes": "/api/v1/wines/880e8400-e29b-41d4-a716-446655440003/quizzes",
      "scenarios": "/api/v1/wines/880e8400-e29b-41d4-a716-446655440003/scenarios"
    }
  }
}
```

---

### 6.3 POST /wines

Create a new wine.

**Authorization:** Content Author, Content Admin, Org Admin

**Request:**

```http
POST /api/v1/wines
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "name": "Cloudy Bay Sauvignon Blanc 2024",
  "producer": "Cloudy Bay",
  "vintage": 2024,
  "region": "Marlborough",
  "country": "NZ",
  "wineType": "white",
  "grapeVarieties": ["Sauvignon Blanc"],
  "priceTier": "premium",
  "price": 28.00,
  "tastingNotes": "Vibrant citrus and tropical fruit aromas with a crisp finish.",
  "foodPairings": ["Seafood", "Salads", "Goat Cheese"],
  "servingTemperature": "8-10°C"
}
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440004",
    "name": "Cloudy Bay Sauvignon Blanc 2024",
    "producer": "Cloudy Bay",
    "vintage": 2024,
    "region": "Marlborough",
    "country": "NZ",
    "wineType": "white",
    "grapeVarieties": ["Sauvignon Blanc"],
    "priceTier": "premium",
    "price": 28.00,
    "status": "DRAFT",
    "version": 1,
    "createdAt": "2026-01-21T10:30:00Z",
    "createdBy": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "John Smith"
    }
  }
}
```

**Validation Rules:**

| Field | Rule |
|-------|------|
| name | Required, 1-255 characters |
| region | Required, 1-100 characters |
| country | Required, ISO 3166-1 alpha-2 code |
| wineType | Required, enum: red, white, rosé, sparkling, dessert, fortified |
| grapeVarieties | Required, 1-10 items |
| priceTier | Required, enum: budget, moderate, premium, luxury |
| vintage | Optional, 1900-current year |

---

### 6.4 PUT /wines/:id

Update a wine.

**Authorization:** Content Author (own draft), Content Admin

**Request:**

```http
PUT /api/v1/wines/990e8400-e29b-41d4-a716-446655440004
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "tastingNotes": "Vibrant citrus, passion fruit and grapefruit with a crisp, mineral finish.",
  "foodPairings": ["Seafood", "Salads", "Goat Cheese", "Thai Cuisine"],
  "quickFacts": {
    "tastingNotes": "Crisp and refreshing with tropical notes",
    "pairings": ["Seafood", "Salads"],
    "keyPoints": ["New Zealand Sauvignon Blanc benchmark", "Cool climate"],
    "pronunciation": "CLOW-dee BAY",
    "studyTips": "Classic New World Sauvignon Blanc style"
  }
}
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440004",
    "name": "Cloudy Bay Sauvignon Blanc 2024",
    "tastingNotes": "Vibrant citrus, passion fruit and grapefruit with a crisp, mineral finish.",
    "foodPairings": ["Seafood", "Salads", "Goat Cheese", "Thai Cuisine"],
    "quickFacts": {
      "tastingNotes": "Crisp and refreshing with tropical notes",
      "pairings": ["Seafood", "Salads"],
      "keyPoints": ["New Zealand Sauvignon Blanc benchmark", "Cool climate"],
      "pronunciation": "CLOW-dee BAY",
      "studyTips": "Classic New World Sauvignon Blanc style"
    },
    "status": "DRAFT",
    "version": 1,
    "updatedAt": "2026-01-21T11:00:00Z"
  }
}
```

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 404 | WINE_NOT_FOUND | Wine does not exist |
| 403 | CANNOT_EDIT_PUBLISHED | Cannot edit published wine (create revision instead) |
| 409 | WINE_IN_REVIEW | Cannot edit wine while in review |

---

### 6.5 DELETE /wines/:id

Soft delete a wine.

**Authorization:** Content Admin, Org Admin

**Request:**

```http
DELETE /api/v1/wines/990e8400-e29b-41d4-a716-446655440004
Authorization: Bearer <access_token>
```

**Response (204 No Content):**

No response body.

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 404 | WINE_NOT_FOUND | Wine does not exist |
| 409 | WINE_HAS_REFERENCES | Wine is referenced by published quizzes or scenarios |

---

### 6.6 POST /wines/:id/submit

Submit a wine for review.

**Authorization:** Content Author, Content Admin

**Request:**

```http
POST /api/v1/wines/990e8400-e29b-41d4-a716-446655440004/submit
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440004",
    "name": "Cloudy Bay Sauvignon Blanc 2024",
    "status": "REVIEW",
    "reviewRequestedAt": "2026-01-21T11:30:00Z",
    "reviewRequestedBy": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "John Smith"
    }
  },
  "meta": {
    "message": "Wine submitted for review"
  }
}
```

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 409 | INVALID_STATUS | Wine is not in DRAFT status |
| 400 | INCOMPLETE_CONTENT | Required fields missing |

---

### 6.7 POST /wines/:id/approve

Approve a wine (domain expert review).

**Authorization:** Domain Expert, Content Admin

**Request:**

```http
POST /api/v1/wines/990e8400-e29b-41d4-a716-446655440004/approve
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "feedback": "Content is accurate and well-written."
}
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440004",
    "name": "Cloudy Bay Sauvignon Blanc 2024",
    "status": "APPROVED",
    "approvedAt": "2026-01-21T12:00:00Z",
    "approvedBy": {
      "id": "aa0e8400-e29b-41d4-a716-446655440005",
      "name": "Sarah Expert"
    }
  }
}
```

---

### 6.8 POST /wines/:id/reject

Reject a wine and return to draft.

**Authorization:** Domain Expert, Content Admin

**Request:**

```http
POST /api/v1/wines/990e8400-e29b-41d4-a716-446655440004/reject
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "feedback": "Tasting notes need more detail. Please expand on the palate description."
}
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440004",
    "name": "Cloudy Bay Sauvignon Blanc 2024",
    "status": "DRAFT",
    "rejectedAt": "2026-01-21T12:00:00Z",
    "rejectionFeedback": "Tasting notes need more detail. Please expand on the palate description."
  }
}
```

---

### 6.9 POST /wines/:id/publish

Publish an approved wine.

**Authorization:** Content Admin, Org Admin

**Request:**

```http
POST /api/v1/wines/990e8400-e29b-41d4-a716-446655440004/publish
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440004",
    "name": "Cloudy Bay Sauvignon Blanc 2024",
    "status": "PUBLISHED",
    "version": 1,
    "publishedAt": "2026-01-21T14:00:00Z",
    "publishedBy": {
      "id": "bb0e8400-e29b-41d4-a716-446655440006",
      "name": "Admin User"
    }
  }
}
```

---

### 6.10 POST /wines/:id/archive

Archive a published wine.

**Authorization:** Content Admin, Org Admin

**Request:**

```http
POST /api/v1/wines/880e8400-e29b-41d4-a716-446655440003/archive
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "reason": "Wine no longer available on our list"
}
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "880e8400-e29b-41d4-a716-446655440003",
    "name": "Château Margaux 2015",
    "status": "ARCHIVED",
    "archivedAt": "2026-01-21T15:00:00Z",
    "archiveReason": "Wine no longer available on our list"
  }
}
```

---

## 7. Module Endpoints

### 7.1 GET /modules

List learning modules.

**Authorization:** Any authenticated user

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Page number |
| pageSize | integer | 20 | Items per page (max: 100) |
| sort | string | sortOrder:asc | Sort field and direction |
| status | string | - | Filter by status |
| tier | string | - | Filter by tier (bronze, silver, gold) |
| category | string | - | Filter by category |

**Request:**

```http
GET /api/v1/modules?tier=bronze&status=PUBLISHED
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "cc0e8400-e29b-41d4-a716-446655440007",
      "title": "Wine Fundamentals",
      "description": "Introduction to wine basics for new staff members.",
      "category": "wine_fundamentals",
      "tier": "bronze",
      "sortOrder": 1,
      "estimatedMinutes": 45,
      "lessonCount": 5,
      "imageUrl": "https://cdn.sommelierspark.com/modules/fundamentals.jpg",
      "status": "PUBLISHED",
      "progress": {
        "completed": false,
        "completedLessons": 2,
        "percentComplete": 40
      }
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "totalPages": 2,
    "totalItems": 25
  }
}
```

---

### 7.2 GET /modules/:id

Get a specific module with lessons.

**Authorization:** Any authenticated user

**Request:**

```http
GET /api/v1/modules/cc0e8400-e29b-41d4-a716-446655440007
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "cc0e8400-e29b-41d4-a716-446655440007",
    "title": "Wine Fundamentals",
    "description": "Introduction to wine basics for new staff members.",
    "category": "wine_fundamentals",
    "tier": "bronze",
    "sortOrder": 1,
    "estimatedMinutes": 45,
    "imageUrl": "https://cdn.sommelierspark.com/modules/fundamentals.jpg",
    "status": "PUBLISHED",
    "lessons": [
      {
        "id": "dd0e8400-e29b-41d4-a716-446655440008",
        "title": "What is Wine?",
        "sortOrder": 1,
        "estimatedMinutes": 10,
        "completed": true
      },
      {
        "id": "ee0e8400-e29b-41d4-a716-446655440009",
        "title": "Wine Types",
        "sortOrder": 2,
        "estimatedMinutes": 10,
        "completed": true
      },
      {
        "id": "ff0e8400-e29b-41d4-a716-446655440010",
        "title": "Reading a Wine Label",
        "sortOrder": 3,
        "estimatedMinutes": 10,
        "completed": false
      }
    ],
    "quiz": {
      "id": "gg0e8400-e29b-41d4-a716-446655440011",
      "title": "Wine Fundamentals Quiz",
      "tier": "bronze",
      "passingScore": 70,
      "questionCount": 10
    },
    "progress": {
      "completed": false,
      "completedLessons": 2,
      "totalLessons": 5,
      "percentComplete": 40,
      "lastAccessedAt": "2026-01-20T15:30:00Z"
    }
  }
}
```

---

### 7.3 POST /modules

Create a new module.

**Authorization:** Content Author, Content Admin

**Request:**

```http
POST /api/v1/modules
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "title": "French Wine Regions",
  "description": "Explore the major wine regions of France.",
  "category": "regions_appellations",
  "tier": "silver",
  "imageUrl": "https://cdn.sommelierspark.com/modules/french-regions.jpg"
}
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "hh0e8400-e29b-41d4-a716-446655440012",
    "title": "French Wine Regions",
    "description": "Explore the major wine regions of France.",
    "category": "regions_appellations",
    "tier": "silver",
    "sortOrder": 10,
    "estimatedMinutes": 0,
    "lessonCount": 0,
    "status": "DRAFT",
    "createdAt": "2026-01-21T10:30:00Z"
  }
}
```

---

### 7.4 POST /modules/:id/lessons

Add a lesson to a module.

**Authorization:** Content Author, Content Admin

**Request:**

```http
POST /api/v1/modules/hh0e8400-e29b-41d4-a716-446655440012/lessons
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "title": "Bordeaux: An Overview",
  "content": "<h2>Introduction to Bordeaux</h2><p>Bordeaux is one of the world's most famous wine regions...</p>",
  "estimatedMinutes": 15
}
```

**Response (201 Created):**

```json
{
  "data": {
    "id": "ii0e8400-e29b-41d4-a716-446655440013",
    "moduleId": "hh0e8400-e29b-41d4-a716-446655440012",
    "title": "Bordeaux: An Overview",
    "content": "<h2>Introduction to Bordeaux</h2><p>Bordeaux is one of the world's most famous wine regions...</p>",
    "sortOrder": 1,
    "estimatedMinutes": 15,
    "createdAt": "2026-01-21T10:45:00Z"
  }
}
```

---

### 7.5 PUT /modules/:id/lessons/reorder

Reorder lessons within a module.

**Authorization:** Content Author, Content Admin

**Request:**

```http
PUT /api/v1/modules/hh0e8400-e29b-41d4-a716-446655440012/lessons/reorder
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "lessonOrder": [
    "ii0e8400-e29b-41d4-a716-446655440013",
    "jj0e8400-e29b-41d4-a716-446655440014",
    "kk0e8400-e29b-41d4-a716-446655440015"
  ]
}
```

**Response (200 OK):**

```json
{
  "data": {
    "moduleId": "hh0e8400-e29b-41d4-a716-446655440012",
    "lessons": [
      { "id": "ii0e8400-e29b-41d4-a716-446655440013", "sortOrder": 1 },
      { "id": "jj0e8400-e29b-41d4-a716-446655440014", "sortOrder": 2 },
      { "id": "kk0e8400-e29b-41d4-a716-446655440015", "sortOrder": 3 }
    ]
  }
}
```

---

## 8. Quiz Endpoints

### 8.1 GET /quizzes

List quizzes.

**Authorization:** Any authenticated user

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Page number |
| pageSize | integer | 20 | Items per page |
| tier | string | - | Filter by tier |
| category | string | - | Filter by category |
| status | string | - | Filter by status |

**Request:**

```http
GET /api/v1/quizzes?tier=bronze&status=PUBLISHED
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "gg0e8400-e29b-41d4-a716-446655440011",
      "title": "Wine Fundamentals Quiz",
      "description": "Test your basic wine knowledge.",
      "tier": "bronze",
      "category": "wine_fundamentals",
      "passingScore": 70,
      "timeLimit": 15,
      "questionCount": 10,
      "status": "PUBLISHED",
      "moduleId": "cc0e8400-e29b-41d4-a716-446655440007",
      "bestScore": 85,
      "passed": true,
      "attemptCount": 2
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "totalPages": 1,
    "totalItems": 12
  }
}
```

---

### 8.2 GET /quizzes/:id

Get quiz details with questions (for learners, answers are hidden).

**Authorization:** Any authenticated user

**Request:**

```http
GET /api/v1/quizzes/gg0e8400-e29b-41d4-a716-446655440011
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "gg0e8400-e29b-41d4-a716-446655440011",
    "title": "Wine Fundamentals Quiz",
    "description": "Test your basic wine knowledge.",
    "tier": "bronze",
    "category": "wine_fundamentals",
    "passingScore": 70,
    "timeLimit": 15,
    "questionCount": 10,
    "status": "PUBLISHED",
    "unlocked": true,
    "userProgress": {
      "bestScore": 85,
      "passed": true,
      "attemptCount": 2,
      "lastAttemptAt": "2026-01-19T14:30:00Z"
    }
  }
}
```

---

### 8.3 GET /quizzes/:id/preview

Preview quiz with answers (for authors).

**Authorization:** Content Author, Content Admin

**Request:**

```http
GET /api/v1/quizzes/gg0e8400-e29b-41d4-a716-446655440011/preview
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "gg0e8400-e29b-41d4-a716-446655440011",
    "title": "Wine Fundamentals Quiz",
    "questions": [
      {
        "id": "ll0e8400-e29b-41d4-a716-446655440016",
        "question": "What colour is a typical Chardonnay wine?",
        "questionType": "multiple_choice",
        "difficulty": "easy",
        "sortOrder": 1,
        "options": [
          { "id": "opt1", "text": "Red", "isCorrect": false },
          { "id": "opt2", "text": "White", "isCorrect": true },
          { "id": "opt3", "text": "Rosé", "isCorrect": false },
          { "id": "opt4", "text": "Orange", "isCorrect": false }
        ],
        "explanation": "Chardonnay is a white grape variety that produces white wine."
      }
    ]
  }
}
```

---

### 8.4 POST /quizzes/:id/attempts

Start a quiz attempt.

**Authorization:** Any authenticated user

**Request:**

```http
POST /api/v1/quizzes/gg0e8400-e29b-41d4-a716-446655440011/attempts
Authorization: Bearer <access_token>
```

**Response (201 Created):**

```json
{
  "data": {
    "attemptId": "mm0e8400-e29b-41d4-a716-446655440017",
    "quizId": "gg0e8400-e29b-41d4-a716-446655440011",
    "startedAt": "2026-01-21T10:30:00Z",
    "expiresAt": "2026-01-21T10:45:00Z",
    "timeLimit": 15,
    "questions": [
      {
        "id": "ll0e8400-e29b-41d4-a716-446655440016",
        "question": "What colour is a typical Chardonnay wine?",
        "questionType": "multiple_choice",
        "sortOrder": 1,
        "options": [
          { "id": "opt1", "text": "Red" },
          { "id": "opt2", "text": "White" },
          { "id": "opt3", "text": "Rosé" },
          { "id": "opt4", "text": "Orange" }
        ]
      }
    ]
  }
}
```

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 403 | QUIZ_LOCKED | Quiz is not unlocked (prerequisites not met) |
| 429 | COOLDOWN_ACTIVE | Must wait before retrying (24h after failed attempt) |

---

### 8.5 POST /quizzes/:id/attempts/:attemptId/submit

Submit a completed quiz attempt.

**Authorization:** Attempt owner

**Request:**

```http
POST /api/v1/quizzes/gg0e8400-e29b-41d4-a716-446655440011/attempts/mm0e8400-e29b-41d4-a716-446655440017/submit
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "answers": [
    { "questionId": "ll0e8400-e29b-41d4-a716-446655440016", "selectedOptions": ["opt2"] },
    { "questionId": "nn0e8400-e29b-41d4-a716-446655440018", "selectedOptions": ["opt1"] }
  ]
}
```

**Response (200 OK):**

```json
{
  "data": {
    "attemptId": "mm0e8400-e29b-41d4-a716-446655440017",
    "quizId": "gg0e8400-e29b-41d4-a716-446655440011",
    "score": 80,
    "passingScore": 70,
    "passed": true,
    "correctCount": 8,
    "totalQuestions": 10,
    "timeSpent": 720,
    "completedAt": "2026-01-21T10:42:00Z",
    "results": [
      {
        "questionId": "ll0e8400-e29b-41d4-a716-446655440016",
        "correct": true,
        "selectedOptions": ["opt2"],
        "correctOptions": ["opt2"],
        "explanation": "Chardonnay is a white grape variety that produces white wine."
      }
    ],
    "certificateEarned": null,
    "nextQuiz": {
      "id": "oo0e8400-e29b-41d4-a716-446655440019",
      "title": "Silver Wine Knowledge Quiz",
      "tier": "silver"
    }
  }
}
```

---

## 9. Scenario Endpoints

### 9.1 GET /scenarios

List scenarios.

**Authorization:** Any authenticated user

**Request:**

```http
GET /api/v1/scenarios?difficulty=bronze&status=PUBLISHED
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "pp0e8400-e29b-41d4-a716-446655440020",
      "title": "First-Time Wine Buyer",
      "description": "Help a customer choose their first bottle of wine.",
      "category": "WINE_PAIRING",
      "difficulty": "bronze",
      "estimatedTime": 10,
      "customerName": "Sarah",
      "status": "PUBLISHED",
      "bestScore": 90,
      "passed": true,
      "attemptCount": 1
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "totalPages": 1,
    "totalItems": 8
  }
}
```

---

### 9.2 GET /scenarios/:id

Get scenario details.

**Authorization:** Any authenticated user

**Request:**

```http
GET /api/v1/scenarios/pp0e8400-e29b-41d4-a716-446655440020
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "pp0e8400-e29b-41d4-a716-446655440020",
    "title": "First-Time Wine Buyer",
    "description": "Help a customer choose their first bottle of wine.",
    "category": "WINE_PAIRING",
    "difficulty": "bronze",
    "estimatedTime": 10,
    "customerName": "Sarah",
    "customerPersona": "A young professional looking to buy wine for a dinner party. She's not very familiar with wine but wants to impress her guests.",
    "situation": "Sarah approaches you at the wine counter looking a bit overwhelmed by the selection.",
    "status": "PUBLISHED",
    "wines": [
      {
        "id": "990e8400-e29b-41d4-a716-446655440004",
        "name": "Cloudy Bay Sauvignon Blanc 2024",
        "talkingPoints": ["Crowd-pleaser", "Great with appetizers"]
      }
    ],
    "userProgress": {
      "bestScore": 90,
      "passed": true,
      "attemptCount": 1
    }
  }
}
```

---

### 9.3 POST /scenarios/:id/attempts

Start a scenario attempt.

**Authorization:** Any authenticated user

**Request:**

```http
POST /api/v1/scenarios/pp0e8400-e29b-41d4-a716-446655440020/attempts
Authorization: Bearer <access_token>
```

**Response (201 Created):**

```json
{
  "data": {
    "attemptId": "qq0e8400-e29b-41d4-a716-446655440021",
    "scenarioId": "pp0e8400-e29b-41d4-a716-446655440020",
    "startedAt": "2026-01-21T10:30:00Z",
    "currentStep": {
      "id": "rr0e8400-e29b-41d4-a716-446655440022",
      "stepOrder": 1,
      "stepType": "DIALOGUE",
      "customerText": "Hi, I'm looking for a wine for a dinner party this weekend. I really don't know much about wine though. Can you help me?",
      "customerMood": "curious",
      "choices": [
        {
          "id": "ch1",
          "choiceText": "Of course! What kind of food will you be serving?",
          "sortOrder": 1
        },
        {
          "id": "ch2",
          "choiceText": "Red or white?",
          "sortOrder": 2
        },
        {
          "id": "ch3",
          "choiceText": "How much are you looking to spend?",
          "sortOrder": 3
        }
      ]
    }
  }
}
```

---

### 9.4 POST /scenarios/:id/attempts/:attemptId/choices

Submit a choice in a scenario.

**Authorization:** Attempt owner

**Request:**

```http
POST /api/v1/scenarios/pp0e8400-e29b-41d4-a716-446655440020/attempts/qq0e8400-e29b-41d4-a716-446655440021/choices
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "choiceId": "ch1"
}
```

**Response (200 OK):**

```json
{
  "data": {
    "attemptId": "qq0e8400-e29b-41d4-a716-446655440021",
    "choiceMade": {
      "id": "ch1",
      "choiceText": "Of course! What kind of food will you be serving?",
      "isOptimal": true,
      "points": 10,
      "feedback": "Great question! Understanding the food pairing helps narrow down options."
    },
    "customerReaction": "Oh, that's a good question! We're having grilled salmon.",
    "currentScore": 10,
    "maxPossibleScore": 10,
    "isComplete": false,
    "nextStep": {
      "id": "ss0e8400-e29b-41d4-a716-446655440023",
      "stepOrder": 2,
      "stepType": "DIALOGUE",
      "customerText": "We're having grilled salmon with lemon butter sauce. What would go well with that?",
      "customerMood": "engaged",
      "choices": [
        {
          "id": "ch4",
          "choiceText": "I'd recommend a crisp white wine like Sauvignon Blanc.",
          "sortOrder": 1
        },
        {
          "id": "ch5",
          "choiceText": "A light red like Pinot Noir could work well.",
          "sortOrder": 2
        },
        {
          "id": "ch6",
          "choiceText": "How about a nice Champagne?",
          "sortOrder": 3
        }
      ]
    }
  }
}
```

---

### 9.5 Scenario Completion Response

When the scenario is complete:

```json
{
  "data": {
    "attemptId": "qq0e8400-e29b-41d4-a716-446655440021",
    "scenarioId": "pp0e8400-e29b-41d4-a716-446655440020",
    "score": 85,
    "passingScore": 70,
    "passed": true,
    "totalPoints": 34,
    "maxPoints": 40,
    "isComplete": true,
    "completedAt": "2026-01-21T10:45:00Z",
    "timeSpent": 900,
    "feedback": "Excellent work! You asked good discovery questions and made appropriate recommendations.",
    "pathTaken": [
      {
        "stepId": "rr0e8400-e29b-41d4-a716-446655440022",
        "choiceId": "ch1",
        "points": 10,
        "isOptimal": true
      },
      {
        "stepId": "ss0e8400-e29b-41d4-a716-446655440023",
        "choiceId": "ch4",
        "points": 10,
        "isOptimal": true
      }
    ]
  }
}
```

---

## 10. Progress Endpoints

### 10.1 GET /progress

Get current user's progress.

**Authorization:** Any authenticated user

**Request:**

```http
GET /api/v1/progress
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "certificationLevel": "BRONZE",
    "overallProgress": {
      "modulesCompleted": 3,
      "totalModules": 10,
      "lessonsCompleted": 12,
      "totalLessons": 45,
      "quizzesPassed": 2,
      "totalQuizzes": 12,
      "scenariosPassed": 1,
      "totalScenarios": 8,
      "percentComplete": 30
    },
    "tierProgress": {
      "bronze": {
        "modulesCompleted": 3,
        "totalModules": 4,
        "quizzesPassed": 2,
        "totalQuizzes": 4,
        "percentComplete": 75,
        "certified": true,
        "certifiedAt": "2026-01-15T10:00:00Z"
      },
      "silver": {
        "modulesCompleted": 0,
        "totalModules": 3,
        "quizzesPassed": 0,
        "totalQuizzes": 4,
        "percentComplete": 0,
        "certified": false,
        "unlocked": true
      },
      "gold": {
        "modulesCompleted": 0,
        "totalModules": 3,
        "quizzesPassed": 0,
        "totalQuizzes": 4,
        "percentComplete": 0,
        "certified": false,
        "unlocked": false
      }
    },
    "recentActivity": [
      {
        "type": "LESSON_COMPLETED",
        "contentId": "dd0e8400-e29b-41d4-a716-446655440008",
        "contentTitle": "What is Wine?",
        "timestamp": "2026-01-20T15:30:00Z"
      },
      {
        "type": "QUIZ_PASSED",
        "contentId": "gg0e8400-e29b-41d4-a716-446655440011",
        "contentTitle": "Wine Fundamentals Quiz",
        "score": 85,
        "timestamp": "2026-01-19T14:30:00Z"
      }
    ]
  }
}
```

---

### 10.2 GET /progress/summary

Get progress summary (for dashboard).

**Authorization:** Any authenticated user

**Request:**

```http
GET /api/v1/progress/summary
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "certificationLevel": "BRONZE",
    "percentComplete": 30,
    "streak": 5,
    "totalTimeSpent": 7200,
    "nextMilestone": {
      "type": "SILVER_CERTIFICATION",
      "progress": 25,
      "remaining": "Complete 3 more Silver modules"
    },
    "weeklyGoal": {
      "target": 60,
      "current": 45,
      "percentComplete": 75
    }
  }
}
```

---

### 10.3 GET /progress/users/:id

Get a specific user's progress (for managers).

**Authorization:** Manager, Admin, Owner

**Request:**

```http
GET /api/v1/progress/users/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer <access_token>
```

**Response (200 OK):**

Same structure as GET /progress.

---

### 10.4 POST /progress/lessons/:id/complete

Mark a lesson as complete.

**Authorization:** Any authenticated user

**Request:**

```http
POST /api/v1/progress/lessons/dd0e8400-e29b-41d4-a716-446655440008/complete
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "lessonId": "dd0e8400-e29b-41d4-a716-446655440008",
    "completed": true,
    "completedAt": "2026-01-21T10:30:00Z",
    "moduleProgress": {
      "moduleId": "cc0e8400-e29b-41d4-a716-446655440007",
      "completedLessons": 3,
      "totalLessons": 5,
      "percentComplete": 60,
      "moduleCompleted": false
    }
  }
}
```

---

### 10.5 GET /certifications

Get user's certifications.

**Authorization:** Any authenticated user

**Request:**

```http
GET /api/v1/certifications
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": [
    {
      "id": "tt0e8400-e29b-41d4-a716-446655440024",
      "level": "BRONZE",
      "title": "Bronze Wine Certification",
      "description": "Demonstrated foundational wine knowledge",
      "earnedAt": "2026-01-15T10:00:00Z",
      "certificateNumber": "SS-BRONZE-2026-001234",
      "certificateUrl": "https://cdn.sommelierspark.com/certificates/SS-BRONZE-2026-001234.pdf"
    }
  ]
}
```

---

## 11. Import Endpoints

### 11.1 POST /imports/wines

Upload a wine import file.

**Authorization:** Manager, Admin, Owner

**Request:**

```http
POST /api/v1/imports/wines
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

```
file: [wines.csv]
duplicateHandling: skip | update | error
```

**Response (201 Created):**

```json
{
  "data": {
    "importId": "uu0e8400-e29b-41d4-a716-446655440025",
    "status": "VALIDATING",
    "fileName": "wines.csv",
    "fileSize": 15234,
    "rowCount": 50,
    "createdAt": "2026-01-21T10:30:00Z"
  }
}
```

---

### 11.2 GET /imports/:id

Get import status.

**Authorization:** Manager, Admin, Owner

**Request:**

```http
GET /api/v1/imports/uu0e8400-e29b-41d4-a716-446655440025
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "importId": "uu0e8400-e29b-41d4-a716-446655440025",
    "status": "AWAITING_CONFIRMATION",
    "fileName": "wines.csv",
    "rowCount": 50,
    "validRows": 48,
    "errorRows": 2,
    "duplicateRows": 5,
    "createdAt": "2026-01-21T10:30:00Z",
    "validatedAt": "2026-01-21T10:30:15Z"
  }
}
```

---

### 11.3 GET /imports/:id/preview

Preview import data.

**Authorization:** Manager, Admin, Owner

**Request:**

```http
GET /api/v1/imports/uu0e8400-e29b-41d4-a716-446655440025/preview?page=1&pageSize=10
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "importId": "uu0e8400-e29b-41d4-a716-446655440025",
    "items": [
      {
        "rowNumber": 1,
        "status": "valid",
        "wine": {
          "name": "Château Margaux 2018",
          "region": "Margaux",
          "country": "FR",
          "wineType": "red"
        }
      },
      {
        "rowNumber": 2,
        "status": "error",
        "wine": {
          "name": "Invalid Wine",
          "region": "",
          "country": "XX"
        },
        "errors": [
          { "field": "region", "message": "Region is required" },
          { "field": "country", "message": "Invalid country code" }
        ]
      }
    ]
  },
  "meta": {
    "page": 1,
    "pageSize": 10,
    "totalPages": 5,
    "totalItems": 50
  }
}
```

---

### 11.4 GET /imports/:id/errors

Get import errors.

**Authorization:** Manager, Admin, Owner

**Request:**

```http
GET /api/v1/imports/uu0e8400-e29b-41d4-a716-446655440025/errors
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "importId": "uu0e8400-e29b-41d4-a716-446655440025",
    "errors": [
      {
        "rowNumber": 2,
        "errors": [
          { "field": "region", "message": "Region is required" },
          { "field": "country", "message": "Invalid country code" }
        ]
      },
      {
        "rowNumber": 15,
        "errors": [
          { "field": "wineType", "message": "Invalid wine type: 'pink'" }
        ]
      }
    ],
    "totalErrors": 2
  }
}
```

---

### 11.5 POST /imports/:id/confirm

Confirm and execute import.

**Authorization:** Manager, Admin, Owner

**Request:**

```http
POST /api/v1/imports/uu0e8400-e29b-41d4-a716-446655440025/confirm
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "importId": "uu0e8400-e29b-41d4-a716-446655440025",
    "status": "COMPLETED",
    "imported": 48,
    "skipped": 2,
    "errors": 0,
    "completedAt": "2026-01-21T10:35:00Z"
  }
}
```

---

### 11.6 POST /imports/:id/cancel

Cancel an import.

**Authorization:** Manager, Admin, Owner

**Request:**

```http
POST /api/v1/imports/uu0e8400-e29b-41d4-a716-446655440025/cancel
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "importId": "uu0e8400-e29b-41d4-a716-446655440025",
    "status": "CANCELLED",
    "cancelledAt": "2026-01-21T10:35:00Z"
  }
}
```

---

### 11.7 GET /templates/wines.csv

Download wine import template.

**Authorization:** Manager, Admin, Owner

**Request:**

```http
GET /api/v1/templates/wines.csv
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```
Content-Type: text/csv
Content-Disposition: attachment; filename="wine-import-template.csv"

name,producer,vintage,region,country,wineType,grapeVarieties,priceTier,price,tastingNotes,foodPairings,servingTemperature
"Example Wine","Producer Name",2020,"Region","FR","red","Grape1;Grape2","premium",45.00,"Tasting notes here","Food1;Food2","16-18°C"
```

---

## 12. Report Endpoints

### 12.1 GET /reports/individual/:userId

Get individual learner report.

**Authorization:** Manager, Admin, Owner (or own report)

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| startDate | date | Report start date |
| endDate | date | Report end date |

**Request:**

```http
GET /api/v1/reports/individual/550e8400-e29b-41d4-a716-446655440000?startDate=2026-01-01&endDate=2026-01-31
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "userName": "John Smith",
    "period": {
      "startDate": "2026-01-01",
      "endDate": "2026-01-31"
    },
    "summary": {
      "lessonsCompleted": 12,
      "quizzesAttempted": 5,
      "quizzesPassed": 4,
      "averageQuizScore": 82,
      "scenariosCompleted": 2,
      "averageScenarioScore": 85,
      "totalTimeSpent": 14400,
      "certificationLevel": "BRONZE"
    },
    "moduleProgress": [
      {
        "moduleId": "cc0e8400-e29b-41d4-a716-446655440007",
        "title": "Wine Fundamentals",
        "percentComplete": 100,
        "completedAt": "2026-01-10T14:30:00Z"
      }
    ],
    "quizResults": [
      {
        "quizId": "gg0e8400-e29b-41d4-a716-446655440011",
        "title": "Wine Fundamentals Quiz",
        "bestScore": 85,
        "attempts": 2,
        "passed": true
      }
    ],
    "strengths": ["Wine Types", "Food Pairing"],
    "areasForImprovement": ["French Regions", "Tasting Notes"]
  }
}
```

---

### 12.2 GET /reports/team

Get team progress report.

**Authorization:** Manager, Admin, Owner

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| startDate | date | Report start date |
| endDate | date | Report end date |
| department | string | Filter by department |

**Request:**

```http
GET /api/v1/reports/team?startDate=2026-01-01&endDate=2026-01-31
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "period": {
      "startDate": "2026-01-01",
      "endDate": "2026-01-31"
    },
    "summary": {
      "totalUsers": 45,
      "activeUsers": 38,
      "certifiedBronze": 25,
      "certifiedSilver": 10,
      "certifiedGold": 2,
      "averageProgress": 65,
      "averageQuizScore": 78
    },
    "topPerformers": [
      {
        "userId": "550e8400-e29b-41d4-a716-446655440000",
        "name": "John Smith",
        "progress": 95,
        "certificationLevel": "SILVER"
      }
    ],
    "atRiskLearners": [
      {
        "userId": "vv0e8400-e29b-41d4-a716-446655440026",
        "name": "Jane Doe",
        "progress": 15,
        "daysSinceLastActivity": 14
      }
    ],
    "progressByDepartment": [
      {
        "department": "Restaurant",
        "userCount": 20,
        "averageProgress": 70
      },
      {
        "department": "Bar",
        "userCount": 10,
        "averageProgress": 55
      }
    ]
  }
}
```

---

### 12.3 GET /reports/organisation

Get organisation-wide report.

**Authorization:** Admin, Owner

**Request:**

```http
GET /api/v1/reports/organisation?startDate=2026-01-01&endDate=2026-01-31
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "period": {
      "startDate": "2026-01-01",
      "endDate": "2026-01-31"
    },
    "summary": {
      "totalUsers": 45,
      "activeUsers": 38,
      "newUsersThisPeriod": 5,
      "completionRate": 65,
      "certificationRate": 82
    },
    "certificationDistribution": {
      "none": 8,
      "bronze": 25,
      "silver": 10,
      "gold": 2
    },
    "contentUsage": {
      "mostPopularModules": [
        { "title": "Wine Fundamentals", "views": 150 }
      ],
      "mostAttemptedQuizzes": [
        { "title": "Wine Fundamentals Quiz", "attempts": 89 }
      ]
    },
    "trends": {
      "weeklyActiveUsers": [32, 35, 38, 36],
      "weeklyCompletions": [45, 52, 48, 55]
    }
  }
}
```

---

### 12.4 GET /reports/content

Get content analytics report.

**Authorization:** Admin

**Request:**

```http
GET /api/v1/reports/content
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "summary": {
      "totalWines": 92,
      "totalModules": 10,
      "totalQuizzes": 12,
      "totalScenarios": 8,
      "totalQuestions": 150
    },
    "contentByStatus": {
      "DRAFT": 15,
      "REVIEW": 3,
      "PUBLISHED": 89,
      "ARCHIVED": 5
    },
    "contentGaps": {
      "winesMissingLevel2": 12,
      "winesMissingLevel3": 25,
      "modulesWithoutQuiz": 2,
      "quizzesWithFewQuestions": 1
    },
    "quizPerformance": [
      {
        "quizId": "gg0e8400-e29b-41d4-a716-446655440011",
        "title": "Wine Fundamentals Quiz",
        "attempts": 89,
        "passRate": 78,
        "averageScore": 75
      }
    ],
    "questionPerformance": {
      "easiest": [
        { "questionId": "q1", "passRate": 95 }
      ],
      "hardest": [
        { "questionId": "q5", "passRate": 45 }
      ]
    }
  }
}
```

---

### 12.5 POST /reports/export

Export a report.

**Authorization:** Manager, Admin, Owner

**Request:**

```http
POST /api/v1/reports/export
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "reportType": "team",
  "format": "csv",
  "startDate": "2026-01-01",
  "endDate": "2026-01-31",
  "filters": {
    "department": "Restaurant"
  }
}
```

**Response (200 OK):**

```json
{
  "data": {
    "exportId": "ww0e8400-e29b-41d4-a716-446655440027",
    "status": "PROCESSING",
    "estimatedTime": 30
  }
}
```

Or if immediately available:

```json
{
  "data": {
    "exportId": "ww0e8400-e29b-41d4-a716-446655440027",
    "status": "COMPLETED",
    "downloadUrl": "https://cdn.sommelierspark.com/exports/team-report-2026-01.csv",
    "expiresAt": "2026-01-22T10:30:00Z"
  }
}
```

---

## 13. Curriculum Endpoints

### 13.1 POST /curriculum/generate

Trigger curriculum generation from wine list.

**Authorization:** Admin, Owner

**Request:**

```http
POST /api/v1/curriculum/generate
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "options": {
    "includeQuizzes": true,
    "includeScenarios": true,
    "targetTiers": ["bronze", "silver", "gold"]
  }
}
```

**Response (202 Accepted):**

```json
{
  "data": {
    "jobId": "xx0e8400-e29b-41d4-a716-446655440028",
    "status": "PENDING",
    "estimatedTime": 180,
    "createdAt": "2026-01-21T10:30:00Z"
  }
}
```

---

### 13.2 GET /curriculum/status

Get curriculum generation status.

**Authorization:** Admin, Owner

**Request:**

```http
GET /api/v1/curriculum/status
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "jobId": "xx0e8400-e29b-41d4-a716-446655440028",
    "status": "PROCESSING",
    "progress": 45,
    "currentStep": "Generating Silver tier modules",
    "steps": [
      { "name": "Analysing wine list", "status": "completed" },
      { "name": "Generating Bronze modules", "status": "completed" },
      { "name": "Generating Silver modules", "status": "in_progress" },
      { "name": "Generating Gold modules", "status": "pending" },
      { "name": "Generating quizzes", "status": "pending" },
      { "name": "Generating scenarios", "status": "pending" }
    ],
    "startedAt": "2026-01-21T10:30:00Z",
    "estimatedCompletion": "2026-01-21T10:35:00Z"
  }
}
```

---

### 13.3 GET /curriculum

Get current curriculum structure.

**Authorization:** Any authenticated user

**Request:**

```http
GET /api/v1/curriculum
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "data": {
    "id": "yy0e8400-e29b-41d4-a716-446655440029",
    "version": 2,
    "generatedAt": "2026-01-15T10:00:00Z",
    "stats": {
      "wineCount": 92,
      "moduleCount": 10,
      "lessonCount": 45,
      "quizCount": 12,
      "scenarioCount": 8
    },
    "paths": [
      {
        "tier": "bronze",
        "title": "Bronze Certification Path",
        "modules": [
          {
            "id": "cc0e8400-e29b-41d4-a716-446655440007",
            "title": "Wine Fundamentals",
            "order": 1
          }
        ],
        "estimatedTime": 180,
        "quizId": "bronze-cert-quiz"
      },
      {
        "tier": "silver",
        "title": "Silver Certification Path",
        "modules": [],
        "estimatedTime": 300,
        "prerequisite": "bronze",
        "quizId": "silver-cert-quiz"
      },
      {
        "tier": "gold",
        "title": "Gold Certification Path",
        "modules": [],
        "estimatedTime": 420,
        "prerequisite": "silver",
        "quizId": "gold-cert-quiz"
      }
    ]
  }
}
```

---

## 14. Common Schemas

### 14.1 Pagination

```typescript
interface PaginationParams {
  page: number;      // Page number (1-indexed)
  pageSize: number;  // Items per page (default: 20, max: 100)
  sort: string;      // Sort field and direction (e.g., "name:asc")
}

interface PaginationMeta {
  page: number;
  pageSize: number;
  totalPages: number;
  totalItems: number;
}

interface PaginationLinks {
  self: string;
  first: string;
  prev: string | null;
  next: string | null;
  last: string;
}
```

### 14.2 Error Response

```typescript
interface ErrorResponse {
  error: {
    code: string;           // Machine-readable error code
    message: string;        // Human-readable message
    details?: ErrorDetail[];// Field-level errors
    requestId: string;      // Request correlation ID
    timestamp: string;      // ISO 8601 timestamp
  };
}

interface ErrorDetail {
  field: string;
  message: string;
}
```

### 14.3 Enumerations

**Wine Types:**
```
red | white | rosé | sparkling | dessert | fortified
```

**Price Tiers:**
```
budget | moderate | premium | luxury
```

**Content Status:**
```
DRAFT | REVIEW | APPROVED | PUBLISHED | ARCHIVED
```

**User Roles:**
```
LEARNER | MANAGER | ADMIN | OWNER | SYSTEM_ADMIN
```

**Content Author Roles:**
```
CONTENT_AUTHOR | DOMAIN_EXPERT | CONTENT_ADMIN
```

**Certification Levels:**
```
NONE | BRONZE | SILVER | GOLD
```

**User Status:**
```
ACTIVE | INVITED | SUSPENDED | DEACTIVATED
```

**Organisation Status:**
```
ACTIVE | TRIAL | SUSPENDED | CANCELLED
```

**Subscription Tiers:**
```
STARTER | PROFESSIONAL | ENTERPRISE
```

**Import Status:**
```
UPLOADED | VALIDATING | VALIDATION_FAILED | AWAITING_CONFIRMATION | PROCESSING | COMPLETED | COMPLETED_WITH_ERRORS | FAILED | CANCELLED
```

**Question Types:**
```
multiple_choice | multiple_select | true_false | matching
```

**Scenario Categories:**
```
WINE_PAIRING | CUSTOMER_SERVICE | UPSELLING | DIETARY_ALLERGIES | WINE_FAULTS | SPECIAL_OCCASIONS
```

### 14.4 Wine Schema

```typescript
interface Wine {
  id: string;
  name: string;
  producer: string | null;
  vintage: number | null;
  region: string;
  country: string;
  wineType: WineType;
  grapeVarieties: string[];
  priceTier: PriceTier;
  price: number | null;
  tastingNotes: string | null;
  foodPairings: string[];
  servingTemperature: string | null;
  decantingTime: string | null;
  quickFacts: QuickFacts | null;
  detailedProfile: DetailedProfile | null;
  expertInsights: ExpertInsights | null;
  imageUrl: string | null;
  status: ContentStatus;
  version: number;
  createdAt: string;
  updatedAt: string;
  publishedAt: string | null;
  createdBy: UserSummary;
}
```

### 14.5 Module Schema

```typescript
interface Module {
  id: string;
  title: string;
  description: string;
  category: string;
  tier: Tier;
  sortOrder: number;
  estimatedMinutes: number;
  imageUrl: string | null;
  status: ContentStatus;
  lessons: Lesson[];
  quiz: QuizSummary | null;
  createdAt: string;
  updatedAt: string;
}

interface Lesson {
  id: string;
  moduleId: string;
  title: string;
  content: string;
  sortOrder: number;
  estimatedMinutes: number;
}
```

### 14.6 Quiz Schema

```typescript
interface Quiz {
  id: string;
  title: string;
  description: string | null;
  tier: Tier;
  category: string;
  passingScore: number;
  timeLimit: number | null;
  questionCount: number;
  status: ContentStatus;
  moduleId: string | null;
  createdAt: string;
  updatedAt: string;
}

interface Question {
  id: string;
  quizId: string;
  question: string;
  questionType: QuestionType;
  options: Option[];
  explanation: string | null;
  difficulty: Difficulty;
  sortOrder: number;
  relatedWineIds: string[];
}

interface Option {
  id: string;
  text: string;
  isCorrect: boolean;
  sortOrder: number;
}
```

### 14.7 Scenario Schema

```typescript
interface Scenario {
  id: string;
  title: string;
  description: string;
  category: ScenarioCategory;
  difficulty: Tier;
  estimatedTime: number;
  customerName: string;
  customerPersona: string;
  situation: string;
  status: ContentStatus;
  steps: ScenarioStep[];
  wines: ScenarioWine[];
  createdAt: string;
  updatedAt: string;
}

interface ScenarioStep {
  id: string;
  stepOrder: number;
  stepType: StepType;
  customerText: string;
  customerMood: string | null;
  context: string | null;
  hint: string | null;
  choices: ScenarioChoice[];
}

interface ScenarioChoice {
  id: string;
  choiceText: string;
  isOptimal: boolean;
  points: number;
  feedback: string;
  customerReaction: string | null;
  nextStepId: string | null;
  sortOrder: number;
}
```

### 14.8 User Schema

```typescript
interface User {
  id: string;
  email: string;
  name: string;
  firstName: string | null;
  lastName: string | null;
  role: UserRole;
  jobTitle: string | null;
  department: string | null;
  organisationId: string;
  status: UserStatus;
  certificationLevel: CertificationLevel;
  avatarUrl: string | null;
  lastLoginAt: string | null;
  createdAt: string;
  updatedAt: string;
}

interface UserSummary {
  id: string;
  name: string;
}
```

### 14.9 Organisation Schema

```typescript
interface Organisation {
  id: string;
  name: string;
  slug: string;
  type: OrganisationType;
  subscriptionTier: SubscriptionTier;
  subscriptionStatus: SubscriptionStatus;
  status: OrganisationStatus;
  settings: OrganisationSettings;
  branding: Branding | null;
  billingEmail: string;
  userCount: number;
  maxUsers: number;
  createdAt: string;
  updatedAt: string;
}
```

---

## 15. Error Handling

### 15.1 HTTP Status Codes

| Status | Meaning | Usage |
|--------|---------|-------|
| 200 | OK | Successful request |
| 201 | Created | Resource created successfully |
| 202 | Accepted | Async operation started |
| 204 | No Content | Successful deletion |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Resource conflict |
| 422 | Unprocessable Entity | Validation failed |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |
| 503 | Service Unavailable | Service temporarily unavailable |

### 15.2 Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| VALIDATION_ERROR | 400 | Request validation failed |
| INVALID_JSON | 400 | Malformed JSON body |
| AUTHENTICATION_REQUIRED | 401 | No valid token provided |
| INVALID_CREDENTIALS | 401 | Login credentials invalid |
| TOKEN_EXPIRED | 401 | Access token has expired |
| TOKEN_REVOKED | 401 | Token has been revoked |
| PERMISSION_DENIED | 403 | Insufficient permissions |
| ACCOUNT_LOCKED | 403 | Account is locked |
| ACCOUNT_SUSPENDED | 403 | Account is suspended |
| FEATURE_NOT_AVAILABLE | 403 | Feature not in subscription tier |
| USER_LIMIT_REACHED | 403 | Organisation user limit reached |
| RESOURCE_NOT_FOUND | 404 | Requested resource not found |
| USER_NOT_FOUND | 404 | User does not exist |
| WINE_NOT_FOUND | 404 | Wine does not exist |
| MODULE_NOT_FOUND | 404 | Module does not exist |
| QUIZ_NOT_FOUND | 404 | Quiz does not exist |
| CONFLICT | 409 | Resource state conflict |
| USER_EXISTS | 409 | User with email already exists |
| INVALID_STATUS | 409 | Invalid status transition |
| WINE_HAS_REFERENCES | 409 | Wine is referenced by other content |
| RATE_LIMITED | 429 | Too many requests |
| INTERNAL_ERROR | 500 | Unexpected server error |
| SERVICE_UNAVAILABLE | 503 | Service temporarily unavailable |

---

## 16. Webhooks

### 16.1 Overview

Webhooks are available for Enterprise tier organisations to receive real-time notifications of events.

**Webhook Endpoint Configuration:**

```json
{
  "url": "https://your-server.com/webhooks/sommelier-spark",
  "secret": "whsec_...",
  "events": ["user.certified", "import.completed"]
}
```

### 16.2 Event Types

| Event | Description |
|-------|-------------|
| `user.created` | New user added to organisation |
| `user.activated` | User accepted invitation |
| `user.certified` | User achieved certification |
| `content.published` | Content item published |
| `content.archived` | Content item archived |
| `import.completed` | Bulk import completed |
| `curriculum.generated` | Curriculum generation complete |

### 16.3 Webhook Payload

```json
{
  "id": "evt_zz0e8400-e29b-41d4-a716-446655440030",
  "type": "user.certified",
  "created": "2026-01-21T10:30:00Z",
  "data": {
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "userName": "John Smith",
    "certificationLevel": "SILVER",
    "certifiedAt": "2026-01-21T10:30:00Z"
  },
  "organisationId": "660e8400-e29b-41d4-a716-446655440001"
}
```

### 16.4 Webhook Security

All webhooks include an HMAC-SHA256 signature in the `X-Sommelier-Signature` header:

```
X-Sommelier-Signature: sha256=abc123...
```

Verify the signature using your webhook secret:

```typescript
const crypto = require('crypto');

function verifySignature(payload, signature, secret) {
  const expected = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  return `sha256=${expected}` === signature;
}
```

### 16.5 Retry Policy

| Attempt | Delay |
|---------|-------|
| 1 | Immediate |
| 2 | 1 minute |
| 3 | 5 minutes |
| 4 | 30 minutes |
| 5 | 2 hours |
| 6 | 24 hours |

Webhooks are disabled after 6 consecutive failures.

---

## 17. Appendices

### 17.1 Glossary

| Term | Definition |
|------|------------|
| **Tenant** | An organisation using the platform |
| **Curriculum** | Complete set of learning content for an organisation |
| **Tier** | Certification level (Bronze, Silver, Gold) |
| **Progressive Disclosure** | Content layering by depth level |
| **Distractor** | Plausible incorrect answer in quiz question |

### 17.2 Country Codes

Use ISO 3166-1 alpha-2 codes for country fields:

| Code | Country |
|------|---------|
| FR | France |
| IT | Italy |
| ES | Spain |
| DE | Germany |
| PT | Portugal |
| US | United States |
| AU | Australia |
| NZ | New Zealand |
| AR | Argentina |
| CL | Chile |
| ZA | South Africa |
| GB | United Kingdom |

### 17.3 Rate Limit Details by Endpoint

| Endpoint Category | Standard | Enterprise |
|-------------------|----------|------------|
| Read operations | 100/min | 1000/min |
| Write operations | 50/min | 500/min |
| Import operations | 10/hour | 100/hour |
| Report exports | 20/hour | 200/hour |
| Curriculum generation | 5/day | 50/day |

### 17.4 Related Documents

| Document ID | Title |
|-------------|-------|
| SS-WS3-HLD | High-Level Design |
| SS-WS3-LLD | Low-Level Design |
| SS-WS3-EARS | Requirements Specification |
| SS-WS3.0-CDM | Content Domain Model |
| SS-WS3.0-ORG | Organization Model |
| SS-WS3.0-CLS | Content Lifecycle Specification |

### 17.5 Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-21 | Obi Wan | Initial draft |

---

*End of API Specification*

**CONFIDENTIAL — Sommelier Spark**
