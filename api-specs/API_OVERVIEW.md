# Talent Mesh API Overview

## Introduction

This document provides an overview of the Talent Mesh API design principles, conventions, and guidelines.

---

## API Design Principles

### 1. RESTful Design
- Resources are nouns, actions are HTTP verbs
- Predictable URL structure
- Stateless interactions

### 2. Consistency
- Consistent naming conventions
- Standard response formats
- Predictable error handling

### 3. Versioning
- URL-based versioning: `/api/v1/`
- Backward-compatible changes within version
- Deprecation policy with migration paths

### 4. Security First
- JWT authentication
- Role-based access control
- Rate limiting per user/IP

---

## Base URL

| Environment | Base URL |
|-------------|----------|
| Development | `http://localhost:3001/api/v1` |
| Staging | `https://api.staging.talentmesh.io/api/v1` |
| Production | `https://api.talentmesh.io/api/v1` |

---

## Authentication

### JWT Bearer Token

```http
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Token Structure

```json
{
  "sub": "user-uuid",
  "linkedin_id": "abc123xyz",
  "org_id": "org-uuid",
  "roles": ["recruiter"],
  "iat": 1704067200,
  "exp": 1704070800
}
```

### Token Lifecycle

| Token Type | Expiration | Refresh |
|------------|------------|---------|
| Access Token | 1 hour | Via refresh token |
| Refresh Token | 7 days | On use (rotation) |

---

## Request Format

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| Authorization | Yes* | Bearer JWT token |
| Content-Type | Yes | `application/json` |
| X-Request-ID | No | Client-provided correlation ID |
| Accept-Language | No | Preferred language |

*Except for public endpoints (LinkedIn OAuth)

### Request Body

```json
{
  "field_name": "value",
  "nested_object": {
    "nested_field": "value"
  },
  "array_field": ["item1", "item2"]
}
```

---

## Response Format

### Success Response

```json
{
  "data": {
    "id": "uuid",
    "type": "user",
    "attributes": {
      "email": "user@example.com",
      "name": "John Doe"
    }
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

### Collection Response

```json
{
  "data": [
    { "id": "uuid1", "type": "assessment", "attributes": {...} },
    { "id": "uuid2", "type": "assessment", "attributes": {...} }
  ],
  "meta": {
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 150,
      "total_pages": 8
    },
    "request_id": "uuid"
  }
}
```

### Error Response

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Email is required"
      }
    ]
  },
  "meta": {
    "request_id": "uuid",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

---

## HTTP Status Codes

### Success Codes

| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | Successful GET, PUT |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |

### Client Error Codes

| Code | Meaning | Use Case |
|------|---------|----------|
| 400 | Bad Request | Validation error |
| 401 | Unauthorized | Missing/invalid token |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource |
| 422 | Unprocessable Entity | Semantic error |
| 429 | Too Many Requests | Rate limit exceeded |

### Server Error Codes

| Code | Meaning | Use Case |
|------|---------|----------|
| 500 | Internal Server Error | Unexpected error |
| 502 | Bad Gateway | Upstream error |
| 503 | Service Unavailable | Maintenance/overload |

---

## Error Codes

### Authentication Errors

| Code | HTTP | Description |
|------|------|-------------|
| AUTH_TOKEN_MISSING | 401 | No token provided |
| AUTH_TOKEN_INVALID | 401 | Token malformed or expired |
| AUTH_TOKEN_EXPIRED | 401 | Token has expired |
| AUTH_LINKEDIN_FAILED | 401 | LinkedIn OAuth failed or denied |
| AUTH_ACCOUNT_LOCKED | 403 | Too many failed attempts |

### Authorization Errors

| Code | HTTP | Description |
|------|------|-------------|
| AUTHZ_INSUFFICIENT_ROLE | 403 | Role doesn't permit action |
| AUTHZ_ORG_ACCESS_DENIED | 403 | Not a member of organization |
| AUTHZ_RESOURCE_DENIED | 403 | Cannot access this resource |

### Validation Errors

| Code | HTTP | Description |
|------|------|-------------|
| VALIDATION_REQUIRED | 400 | Required field missing |
| VALIDATION_FORMAT | 400 | Invalid format |
| VALIDATION_RANGE | 400 | Value out of range |
| VALIDATION_UNIQUE | 409 | Value must be unique |

### Resource Errors

| Code | HTTP | Description |
|------|------|-------------|
| RESOURCE_NOT_FOUND | 404 | Resource doesn't exist |
| RESOURCE_DELETED | 410 | Resource was deleted |
| RESOURCE_CONFLICT | 409 | State conflict |

---

## Pagination

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Page number |
| per_page | integer | 20 | Items per page (max: 100) |
| sort | string | created_at | Sort field |
| order | string | desc | Sort direction (asc/desc) |

### Example

```http
GET /api/v1/assessments?page=2&per_page=50&sort=score&order=desc
```

### Response Headers

```http
X-Total-Count: 150
X-Total-Pages: 3
Link: <...?page=1>; rel="first", <...?page=3>; rel="last"
```

---

## Filtering

### Simple Filters

```http
GET /api/v1/assessments?status=completed&type=devops
```

### Range Filters

```http
GET /api/v1/assessments?score_min=70&score_max=90
GET /api/v1/assessments?created_after=2024-01-01&created_before=2024-01-31
```

### Search

```http
GET /api/v1/users?search=john
```

---

## Rate Limiting

### Limits

| Endpoint Category | Limit | Window |
|-------------------|-------|--------|
| Authentication | 10 | 1 minute |
| Read operations | 100 | 1 minute |
| Write operations | 30 | 1 minute |

### Response Headers

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1704067260
```

### Rate Limit Exceeded

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30

{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Try again in 30 seconds."
  }
}
```

---

## API Services

### Platform APIs

| Service | Base Path | Description |
|---------|-----------|-------------|
| Auth Service | `/api/v1/auth` | Authentication, sessions |
| User Service | `/api/v1/users` | Profiles, CV, skills |
| Assessment Service | `/api/v1/assessments` | Templates, configurations |
| Scheduling Service | `/api/v1/scheduling` | Slots, bookings |

### Intelligence APIs

| Service | Base Path | Description |
|---------|-----------|-------------|
| Scoring Service | `/api/v1/scores` | Assessment scores |
| Report Service | `/api/v1/reports` | Assessment reports |

### Internal APIs (Not Public)

| Service | Description |
|---------|-------------|
| Agent Service | AI Agent Pod lifecycle management |
| Signaling Service | WebRTC signaling (Rust) |
| AI Agent Pod | STT/TTS/LLM processing (pod-internal) |

---

## OpenAPI Specifications

- [auth-service.yaml](openapi/auth-service.yaml)
- [user-service.yaml](openapi/user-service.yaml)
- [assessment-service.yaml](openapi/assessment-service.yaml)
- [scheduling-service.yaml](openapi/scheduling-service.yaml)
- [matching-service.yaml](openapi/matching-service.yaml)
- [agent-service.yaml](openapi/agent-service.yaml)
- [llm-gateway.yaml](openapi/llm-gateway.yaml)

## AsyncAPI Specifications

- [events.yaml](asyncapi/events.yaml)

---

## SDK Support

### Official SDKs (Planned)

| Language | Package | Status |
|----------|---------|--------|
| JavaScript/TypeScript | @talentmesh/sdk | Planned |
| Python | talentmesh-sdk | Planned |

### SDK Example (Future)

```typescript
import { TalentMeshClient } from '@talentmesh/sdk';

const client = new TalentMeshClient({
  apiKey: 'your-api-key',
  environment: 'production'
});

const assessment = await client.assessments.get('assessment-id');
console.log(assessment.score);
```

---

## Webhooks

See [INTEGRATION_PATTERNS.md](../03-architecture/INTEGRATION_PATTERNS.md#webhook-integration) for webhook documentation.

### Webhook Events

| Event | Description |
|-------|-------------|
| `assessment.completed` | Assessment finished |
| `assessment.scored` | Scoring complete |
| `candidate.created` | New candidate registered |
| `booking.created` | Assessment scheduled |

---

## Changelog

### v1.0.0 (2024-01-01)
- Initial API release
- Authentication endpoints
- User management
- Assessment configuration
- Scheduling

---

*Document Version: 3.0*
*Last Updated: 2026-01-04*
*Owner: API Team*
