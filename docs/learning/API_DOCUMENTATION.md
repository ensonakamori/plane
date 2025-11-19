# API Documentation

> **Last Updated:** November 2025
> **Status:** ✅ CURRENT - Complete REST API reference for Plane

A comprehensive API reference for Plane's REST API, including authentication, endpoints, request/response formats, and real examples from the codebase.

---

## Table of Contents

1. [API Overview](#api-overview)
2. [Authentication](#authentication)
3. [Common Patterns](#common-patterns)
4. [User Endpoints](#user-endpoints)
5. [Workspace Endpoints](#workspace-endpoints)
6. [Project Endpoints](#project-endpoints)
7. [Issue Endpoints](#issue-endpoints)
8. [State Endpoints](#state-endpoints)
9. [Module & Cycle Endpoints](#module--cycle-endpoints)
10. [Error Handling](#error-handling)
11. [Rate Limiting](#rate-limiting)
12. [Pagination](#pagination)
13. [Filtering & Sorting](#filtering--sorting)

---

## API Overview

### Base URL

```
Production:  https://api.plane.so/api/v1
Development: http://localhost:8000/api/v1
```

### API Versioning

Plane uses URL-based versioning:

```
/api/v1/  - Current stable version
/api/v2/  - Next version (when available)
```

### Content Type

All API endpoints accept and return JSON:

```http
Content-Type: application/json
Accept: application/json
```

### HTTP Methods

| Method | Description | Idempotent |
|--------|-------------|------------|
| GET | Retrieve resources | ✅ Yes |
| POST | Create new resources | ❌ No |
| PUT | Replace entire resource | ✅ Yes |
| PATCH | Partial update | ❌ No |
| DELETE | Remove resource | ✅ Yes |

### Response Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created successfully |
| 204 | No Content | Delete successful (no body) |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Permission denied |
| 404 | Not Found | Resource doesn't exist |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |

---

## Authentication

### Authentication Methods

Plane supports multiple authentication methods:

1. **Session-based** (Web application)
2. **JWT tokens** (Mobile/API clients)
3. **OAuth 2.0** (Google, GitHub, GitLab)
4. **API Keys** (Programmatic access)

### Session Authentication

**Endpoint:** `POST /api/auth/sign-in/`

Used for web application login. Returns HTTP-only cookie.

**Request:**

```http
POST /api/auth/sign-in/
Content-Type: application/x-www-form-urlencoded

email=user@example.com&password=secure_password&next_path=/
```

**Response:**

```http
HTTP/1.1 302 Found
Set-Cookie: sessionid=abc123...; HttpOnly; Secure; SameSite=Lax
Location: https://app.plane.so/
```

**Subsequent Requests:**

```http
GET /api/users/me/
Cookie: sessionid=abc123...
```

### Magic Link Authentication

**Endpoint:** `POST /api/auth/magic-sign-in/`

Sends magic link to user's email.

**Request:**

```json
POST /api/auth/magic-sign-in/
Content-Type: application/json

{
  "email": "user@example.com"
}
```

**Response:**

```json
{
  "key": "check-email"
}
```

**Verify Magic Code:**

```http
GET /api/auth/magic-sign-in/?code=MAGIC_CODE&email=user@example.com
```

### OAuth 2.0 Authentication

**Supported Providers:**
- Google (`/api/auth/google/`)
- GitHub (`/api/auth/github/`)
- GitLab (`/api/auth/gitlab/`)

**Flow:**

```
1. GET /api/auth/google/
   → Redirects to Google OAuth consent

2. User approves

3. Google redirects back: /api/auth/google/callback/?code=...

4. Backend exchanges code for token and creates session

5. Redirects to app with session cookie
```

**Example:**

```http
GET /api/auth/google/?next_path=/workspaces

# User approves at Google

# Google redirects:
GET /api/auth/google/callback/?code=4/P7q7W91...&state=...

# Backend processes and redirects:
HTTP/1.1 302 Found
Set-Cookie: sessionid=...; HttpOnly; Secure
Location: https://app.plane.so/workspaces
```

### API Key Authentication

**Create API Key:**

```http
POST /api/users/me/api-keys/
Content-Type: application/json

{
  "name": "CI/CD Integration",
  "expires_in_days": 90
}
```

**Response:**

```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "key": "plane_sk_1234567890abcdef",
  "name": "CI/CD Integration",
  "created_at": "2025-11-19T10:00:00Z",
  "expires_at": "2026-02-19T10:00:00Z"
}
```

**Use API Key:**

```http
GET /api/workspaces/
Authorization: Bearer plane_sk_1234567890abcdef
```

### Check Authentication Status

**Endpoint:** `GET /api/users/me/session/`

```http
GET /api/users/me/session/
```

**Response (Authenticated):**

```json
{
  "is_authenticated": true,
  "user": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "email": "user@example.com",
    "display_name": "John Doe",
    "avatar": "https://..."
  }
}
```

**Response (Unauthenticated):**

```json
{
  "is_authenticated": false
}
```

### Sign Out

**Endpoint:** `POST /api/auth/sign-out/`

```http
POST /api/auth/sign-out/
```

**Response:**

```http
HTTP/1.1 302 Found
Set-Cookie: sessionid=; Expires=Thu, 01 Jan 1970 00:00:00 GMT
Location: https://app.plane.so/
```

---

## Common Patterns

### Resource URL Structure

Plane uses hierarchical URL structure:

```
/api/{resource}/
/api/{resource}/{id}/
/api/workspaces/{slug}/{resource}/
/api/workspaces/{slug}/projects/{project_id}/{resource}/
```

**Examples:**

```
GET    /api/users/me/
GET    /api/workspaces/
GET    /api/workspaces/acme/projects/
GET    /api/workspaces/acme/projects/123/issues/
PATCH  /api/workspaces/acme/projects/123/issues/456/
```

### Standard CRUD Operations

Every resource follows RESTful conventions:

```http
# List all
GET /api/workspaces/{slug}/projects/

# Get one
GET /api/workspaces/{slug}/projects/{id}/

# Create new
POST /api/workspaces/{slug}/projects/

# Full update
PUT /api/workspaces/{slug}/projects/{id}/

# Partial update
PATCH /api/workspaces/{slug}/projects/{id}/

# Delete
DELETE /api/workspaces/{slug}/projects/{id}/
```

### Timestamps

All resources include audit timestamps:

```json
{
  "created_at": "2025-11-19T10:00:00Z",
  "updated_at": "2025-11-19T12:30:00Z",
  "created_by": "123e4567-e89b-12d3-a456-426614174000",
  "updated_by": "123e4567-e89b-12d3-a456-426614174000"
}
```

### Soft Deletion

Resources are soft-deleted (marked deleted, not removed):

```json
{
  "deleted_at": "2025-11-19T15:00:00Z"
}
```

To permanently delete, use `?permanent=true` (admin only).

---

## User Endpoints

### Get Current User

**Endpoint:** `GET /api/users/me/`

Returns the authenticated user's profile.

**Request:**

```http
GET /api/users/me/
Authorization: Bearer {token}
```

**Response:**

```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "email": "john.doe@example.com",
  "username": "john.doe",
  "display_name": "John Doe",
  "first_name": "John",
  "last_name": "Doe",
  "avatar": "https://avatars.plane.so/john.jpg",
  "cover_image": null,
  "user_timezone": "America/New_York",
  "theme": {
    "theme": "dark",
    "accent": "blue"
  },
  "is_email_verified": true,
  "is_onboarded": true,
  "is_tour_completed": true,
  "last_workspace_id": "workspace-uuid",
  "created_at": "2024-01-15T10:00:00Z",
  "updated_at": "2025-11-19T12:00:00Z"
}
```

### Update Current User

**Endpoint:** `PATCH /api/users/me/`

**Request:**

```json
PATCH /api/users/me/
Content-Type: application/json

{
  "display_name": "John Smith",
  "first_name": "John",
  "last_name": "Smith",
  "avatar": "https://new-avatar.jpg",
  "user_timezone": "America/Los_Angeles"
}
```

**Response:**

```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "display_name": "John Smith",
  "first_name": "John",
  "last_name": "Smith",
  "avatar": "https://new-avatar.jpg",
  "user_timezone": "America/Los_Angeles",
  "updated_at": "2025-11-19T13:00:00Z"
}
```

**Validation:**

- `display_name`: 1-255 characters
- `email`: Valid email format (read-only after creation)
- `user_timezone`: Valid timezone from pytz.common_timezones

### Get User Settings

**Endpoint:** `GET /api/users/me/settings/`

Returns user preferences and settings.

**Response:**

```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "email": "john.doe@example.com",
  "workspace": {
    "last_workspace_id": "workspace-uuid",
    "fallback_workspace_id": "workspace-uuid",
    "fallback_workspace_slug": "acme"
  },
  "theme": {
    "theme": "dark",
    "accent": "blue"
  }
}
```

### Deactivate Account

**Endpoint:** `DELETE /api/users/me/deactivate/`

Deactivates the user account (soft delete).

**Request:**

```http
DELETE /api/users/me/deactivate/
```

**Response:**

```http
HTTP/1.1 204 No Content
```

**Effects:**
- User is logged out
- All sessions are invalidated
- User is removed from all workspaces and projects
- Account can be reactivated by logging in again

---

## Workspace Endpoints

### List Workspaces

**Endpoint:** `GET /api/workspaces/`

Returns all workspaces the user is a member of.

**Request:**

```http
GET /api/workspaces/
```

**Response:**

```json
[
  {
    "id": "workspace-uuid-1",
    "name": "Acme Corp",
    "slug": "acme",
    "logo": "https://logos.plane.so/acme.png",
    "owner": {
      "id": "user-uuid",
      "display_name": "Jane Doe",
      "avatar": "https://avatars.plane.so/jane.jpg"
    },
    "total_members": 15,
    "total_projects": 8,
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2025-11-19T10:00:00Z"
  },
  {
    "id": "workspace-uuid-2",
    "name": "Personal Projects",
    "slug": "john-personal",
    "logo": null,
    "owner": {
      "id": "user-uuid-2",
      "display_name": "John Doe"
    },
    "total_members": 1,
    "total_projects": 3,
    "created_at": "2024-06-15T00:00:00Z",
    "updated_at": "2025-11-18T08:00:00Z"
  }
]
```

### Get Workspace Details

**Endpoint:** `GET /api/workspaces/{slug}/`

**Request:**

```http
GET /api/workspaces/acme/
```

**Response:**

```json
{
  "id": "workspace-uuid",
  "name": "Acme Corp",
  "slug": "acme",
  "logo": "https://logos.plane.so/acme.png",
  "logo_props": {
    "in_use": "emoji",
    "emoji": {
      "value": "🚀",
      "url": "https://..."
    }
  },
  "owner": {
    "id": "user-uuid",
    "email": "owner@acme.com",
    "display_name": "Jane Doe",
    "avatar": "https://avatars.plane.so/jane.jpg"
  },
  "organization_size": "50-200",
  "timezone": "America/New_York",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2025-11-19T10:00:00Z",
  "created_by": "user-uuid"
}
```

### Create Workspace

**Endpoint:** `POST /api/workspaces/`

**Request:**

```json
POST /api/workspaces/
Content-Type: application/json

{
  "name": "My New Workspace",
  "slug": "my-workspace",
  "organization_size": "10-50"
}
```

**Response:**

```json
{
  "id": "new-workspace-uuid",
  "name": "My New Workspace",
  "slug": "my-workspace",
  "owner": {
    "id": "user-uuid",
    "display_name": "John Doe"
  },
  "organization_size": "10-50",
  "timezone": "UTC",
  "created_at": "2025-11-19T14:00:00Z"
}
```

**Validation:**
- `name`: Required, 1-80 characters
- `slug`: Required, unique, 3-48 characters, alphanumeric + hyphens
- `organization_size`: Optional, choices: "1-10", "10-50", "50-200", "200-500", "500+"

### Update Workspace

**Endpoint:** `PATCH /api/workspaces/{slug}/`

**Request:**

```json
PATCH /api/workspaces/acme/
Content-Type: application/json

{
  "name": "Acme Corporation",
  "logo": "https://new-logo.png",
  "timezone": "America/Los_Angeles"
}
```

**Response:**

```json
{
  "id": "workspace-uuid",
  "name": "Acme Corporation",
  "slug": "acme",
  "logo": "https://new-logo.png",
  "timezone": "America/Los_Angeles",
  "updated_at": "2025-11-19T14:30:00Z"
}
```

### Delete Workspace

**Endpoint:** `DELETE /api/workspaces/{slug}/`

Soft deletes the workspace. Only workspace owner can delete.

**Request:**

```http
DELETE /api/workspaces/acme/
```

**Response:**

```http
HTTP/1.1 204 No Content
```

---

## Project Endpoints

### List Projects

**Endpoint:** `GET /api/workspaces/{slug}/projects/`

Returns all projects in a workspace that the user has access to.

**Request:**

```http
GET /api/workspaces/acme/projects/
```

**Response:**

```json
[
  {
    "id": "project-uuid-1",
    "name": "Website Redesign",
    "identifier": "WEB",
    "description": "Complete redesign of company website",
    "network": 2,
    "workspace": "workspace-uuid",
    "workspace_detail": {
      "name": "Acme Corp",
      "slug": "acme"
    },
    "project_lead": {
      "id": "user-uuid",
      "display_name": "Jane Doe",
      "avatar": "https://..."
    },
    "emoji": "🎨",
    "icon_prop": {
      "name": "palette",
      "color": "blue"
    },
    "cover_image": "https://covers.plane.so/web-project.jpg",
    "module_view": true,
    "cycle_view": true,
    "issue_views_view": true,
    "page_view": true,
    "intake_view": false,
    "member_role": 20,
    "is_favorite": true,
    "total_members": 8,
    "total_cycles": 3,
    "total_modules": 5,
    "is_member": true,
    "sort_order": 1000,
    "archived_at": null,
    "created_at": "2024-03-01T00:00:00Z",
    "updated_at": "2025-11-19T10:00:00Z"
  }
]
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `per_page` | integer | Items per page (default: 20) |
| `cursor` | string | Pagination cursor |
| `order_by` | string | Sort field (default: "sort_order") |
| `fields` | string | Comma-separated fields to return |

**Example with Pagination:**

```http
GET /api/workspaces/acme/projects/?per_page=10&cursor=next_cursor_token
```

### Get Project Details

**Endpoint:** `GET /api/workspaces/{slug}/projects/{project_id}/`

**Request:**

```http
GET /api/workspaces/acme/projects/project-uuid/
```

**Response:**

```json
{
  "id": "project-uuid",
  "name": "Website Redesign",
  "identifier": "WEB",
  "description": "Complete redesign of company website",
  "description_html": "<p>Complete redesign of company website</p>",
  "network": 2,
  "workspace": "workspace-uuid",
  "project_lead": {
    "id": "user-uuid",
    "email": "lead@acme.com",
    "display_name": "Jane Doe",
    "avatar": "https://..."
  },
  "default_assignee": null,
  "default_state": {
    "id": "state-uuid",
    "name": "Backlog",
    "color": "#858E96"
  },
  "estimate": {
    "id": "estimate-uuid",
    "name": "Fibonacci",
    "type": "points"
  },
  "emoji": "🎨",
  "icon_prop": null,
  "cover_image": "https://covers.plane.so/web-project.jpg",
  "module_view": true,
  "cycle_view": true,
  "issue_views_view": true,
  "page_view": true,
  "intake_view": false,
  "is_time_tracking_enabled": false,
  "is_issue_type_enabled": true,
  "guest_view_all_features": false,
  "archive_in": 0,
  "close_in": 0,
  "timezone": "America/New_York",
  "archived_at": null,
  "created_at": "2024-03-01T00:00:00Z",
  "updated_at": "2025-11-19T10:00:00Z",
  "created_by": "user-uuid",
  "updated_by": "user-uuid"
}
```

### Create Project

**Endpoint:** `POST /api/workspaces/{slug}/projects/`

**Request:**

```json
POST /api/workspaces/acme/projects/
Content-Type: application/json

{
  "name": "Mobile App",
  "identifier": "MOB",
  "description": "iOS and Android mobile application",
  "network": 2,
  "project_lead": "user-uuid",
  "emoji": "📱"
}
```

**Response:**

```json
{
  "id": "new-project-uuid",
  "name": "Mobile App",
  "identifier": "MOB",
  "description": "iOS and Android mobile application",
  "workspace": "workspace-uuid",
  "project_lead": {
    "id": "user-uuid",
    "display_name": "Jane Doe"
  },
  "emoji": "📱",
  "network": 2,
  "created_at": "2025-11-19T15:00:00Z"
}
```

**Validation:**
- `name`: Required, 1-255 characters, unique per workspace
- `identifier`: Required, 1-12 characters, uppercase letters/numbers, unique per workspace
- `description`: Optional, max 5000 characters
- `network`: 0 (Secret) or 2 (Public)
- `emoji`: Optional, valid emoji character

### Update Project

**Endpoint:** `PATCH /api/workspaces/{slug}/projects/{project_id}/`

**Request:**

```json
PATCH /api/workspaces/acme/projects/project-uuid/
Content-Type: application/json

{
  "name": "Mobile App v2",
  "description": "Updated description",
  "module_view": true,
  "cycle_view": true
}
```

**Response:**

```json
{
  "id": "project-uuid",
  "name": "Mobile App v2",
  "description": "Updated description",
  "module_view": true,
  "cycle_view": true,
  "updated_at": "2025-11-19T15:30:00Z",
  "updated_by": "user-uuid"
}
```

### Archive Project

**Endpoint:** `PATCH /api/workspaces/{slug}/projects/{project_id}/archive/`

**Request:**

```http
PATCH /api/workspaces/acme/projects/project-uuid/archive/
```

**Response:**

```json
{
  "id": "project-uuid",
  "archived_at": "2025-11-19T16:00:00Z"
}
```

### Unarchive Project

**Endpoint:** `PATCH /api/workspaces/{slug}/projects/{project_id}/unarchive/`

**Request:**

```http
PATCH /api/workspaces/acme/projects/project-uuid/unarchive/
```

**Response:**

```json
{
  "id": "project-uuid",
  "archived_at": null
}
```

### Delete Project

**Endpoint:** `DELETE /api/workspaces/{slug}/projects/{project_id}/`

**Request:**

```http
DELETE /api/workspaces/acme/projects/project-uuid/
```

**Response:**

```http
HTTP/1.1 204 No Content
```

---

## Issue Endpoints

### List Issues

**Endpoint:** `GET /api/workspaces/{slug}/projects/{project_id}/issues/`

Returns all issues in a project with filtering and sorting.

**Request:**

```http
GET /api/workspaces/acme/projects/project-uuid/issues/
```

**Response:**

```json
[
  {
    "id": "issue-uuid-1",
    "name": "Implement user authentication",
    "description_html": "<p>Add JWT-based authentication</p>",
    "description_stripped": "Add JWT-based authentication",
    "sequence_id": 1,
    "project": "project-uuid",
    "workspace": "workspace-uuid",
    "state": {
      "id": "state-uuid",
      "name": "In Progress",
      "color": "#3A9EC2",
      "group": "started"
    },
    "priority": "high",
    "assignees": [
      {
        "id": "user-uuid-1",
        "display_name": "John Doe",
        "avatar": "https://..."
      }
    ],
    "labels": [
      {
        "id": "label-uuid-1",
        "name": "backend",
        "color": "#FF6B6B"
      }
    ],
    "start_date": "2025-11-19",
    "target_date": "2025-11-25",
    "estimate_point": {
      "id": "point-uuid",
      "key": 1,
      "value": "5"
    },
    "parent": null,
    "sub_issues_count": 2,
    "attachment_count": 1,
    "link_count": 2,
    "cycle_id": "cycle-uuid",
    "module_ids": ["module-uuid-1"],
    "sort_order": 10000,
    "completed_at": null,
    "archived_at": null,
    "is_draft": false,
    "created_at": "2025-11-15T10:00:00Z",
    "updated_at": "2025-11-19T14:00:00Z",
    "created_by": "user-uuid",
    "updated_by": "user-uuid"
  }
]
```

### Filter & Sort Issues

**Query Parameters:**

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `priority` | string | Comma-separated priorities | `urgent,high` |
| `state` | string | Comma-separated state IDs | `state-1,state-2` |
| `assignees` | string | Comma-separated user IDs | `user-1,user-2` |
| `labels` | string | Comma-separated label IDs | `label-1,label-2` |
| `created_by` | string | User ID who created | `user-uuid` |
| `search` | string | Search in name/description | `authentication` |
| `start_date` | string | Filter by start date | `2025-11-01;after` |
| `target_date` | string | Filter by target date | `2025-12-31;before` |
| `order_by` | string | Sort field | `-created_at` |
| `group_by` | string | Group results by field | `state` |
| `sub_group_by` | string | Sub-group by field | `priority` |

**Examples:**

```http
# High and urgent priority issues
GET /api/workspaces/acme/projects/proj/issues/?priority=urgent,high

# Issues assigned to specific users
GET /api/workspaces/acme/projects/proj/issues/?assignees=user-1,user-2

# Issues in specific states
GET /api/workspaces/acme/projects/proj/issues/?state=state-1,state-2

# Search issues
GET /api/workspaces/acme/projects/proj/issues/?search=authentication

# Sort by priority (descending) then created date
GET /api/workspaces/acme/projects/proj/issues/?order_by=-priority,-created_at

# Group by state, sub-group by priority
GET /api/workspaces/acme/projects/proj/issues/?group_by=state&sub_group_by=priority
```

### Get Issue Details

**Endpoint:** `GET /api/workspaces/{slug}/projects/{project_id}/issues/{issue_id}/`

**Request:**

```http
GET /api/workspaces/acme/projects/proj/issues/issue-uuid/
```

**Response:**

```json
{
  "id": "issue-uuid",
  "name": "Implement user authentication",
  "description": {
    "type": "doc",
    "content": [...]
  },
  "description_html": "<p>Add JWT-based authentication</p>",
  "description_stripped": "Add JWT-based authentication",
  "sequence_id": 1,
  "project": "project-uuid",
  "project_detail": {
    "id": "project-uuid",
    "name": "Website Redesign",
    "identifier": "WEB"
  },
  "workspace": "workspace-uuid",
  "workspace_detail": {
    "name": "Acme Corp",
    "slug": "acme"
  },
  "state": {
    "id": "state-uuid",
    "name": "In Progress",
    "color": "#3A9EC2",
    "group": "started",
    "sequence": 2
  },
  "priority": "high",
  "assignees": [
    {
      "id": "user-uuid-1",
      "email": "john@acme.com",
      "display_name": "John Doe",
      "avatar": "https://..."
    }
  ],
  "labels": [
    {
      "id": "label-uuid-1",
      "name": "backend",
      "color": "#FF6B6B",
      "parent": null
    }
  ],
  "start_date": "2025-11-19",
  "target_date": "2025-11-25",
  "estimate_point": {
    "id": "point-uuid",
    "key": 1,
    "value": "5",
    "description": "Medium complexity task"
  },
  "parent": null,
  "sub_issues": [
    {
      "id": "sub-issue-1",
      "name": "Design auth API",
      "sequence_id": 2
    },
    {
      "id": "sub-issue-2",
      "name": "Implement JWT generation",
      "sequence_id": 3
    }
  ],
  "attachments": [
    {
      "id": "attachment-uuid",
      "asset": "https://attachments.plane.so/file.pdf",
      "attributes": {
        "name": "requirements.pdf",
        "size": 102400
      }
    }
  ],
  "links": [
    {
      "id": "link-uuid",
      "title": "OAuth 2.0 Spec",
      "url": "https://oauth.net/2/"
    }
  ],
  "reactions": [
    {
      "id": "reaction-uuid",
      "reaction": "👍",
      "actor": {
        "id": "user-uuid",
        "display_name": "Jane Doe"
      }
    }
  ],
  "is_subscribed": true,
  "cycle_id": "cycle-uuid",
  "module_ids": ["module-uuid-1"],
  "sort_order": 10000,
  "completed_at": null,
  "archived_at": null,
  "is_draft": false,
  "external_source": null,
  "external_id": null,
  "created_at": "2025-11-15T10:00:00Z",
  "updated_at": "2025-11-19T14:00:00Z",
  "created_by": "user-uuid",
  "updated_by": "user-uuid"
}
```

### Create Issue

**Endpoint:** `POST /api/workspaces/{slug}/projects/{project_id}/issues/`

**Request:**

```json
POST /api/workspaces/acme/projects/proj/issues/
Content-Type: application/json

{
  "name": "Fix login bug",
  "description_html": "<p>Users can't login with email</p>",
  "state": "state-uuid",
  "priority": "urgent",
  "assignees": ["user-uuid-1", "user-uuid-2"],
  "labels": ["label-uuid-1"],
  "start_date": "2025-11-19",
  "target_date": "2025-11-20",
  "estimate_point": "point-uuid-1",
  "parent": null
}
```

**Response:**

```json
{
  "id": "new-issue-uuid",
  "name": "Fix login bug",
  "description_html": "<p>Users can't login with email</p>",
  "sequence_id": 47,
  "project": "project-uuid",
  "state": {
    "id": "state-uuid",
    "name": "Backlog"
  },
  "priority": "urgent",
  "assignees": [
    {"id": "user-uuid-1", "display_name": "John Doe"},
    {"id": "user-uuid-2", "display_name": "Jane Smith"}
  ],
  "labels": [
    {"id": "label-uuid-1", "name": "bug", "color": "#FF0000"}
  ],
  "start_date": "2025-11-19",
  "target_date": "2025-11-20",
  "created_at": "2025-11-19T16:00:00Z",
  "created_by": "user-uuid"
}
```

**Validation:**
- `name`: Required, 1-255 characters
- `priority`: "urgent", "high", "medium", "low", "none" (default: "none")
- `state`: Optional, defaults to project's default state
- `start_date`, `target_date`: ISO 8601 date format (YYYY-MM-DD)
- `assignees`: Array of user UUIDs (must be project members)
- `labels`: Array of label UUIDs

### Update Issue

**Endpoint:** `PATCH /api/workspaces/{slug}/projects/{project_id}/issues/{issue_id}/`

**Request:**

```json
PATCH /api/workspaces/acme/projects/proj/issues/issue-uuid/
Content-Type: application/json

{
  "name": "Fix critical login bug",
  "priority": "urgent",
  "state": "in-progress-state-uuid",
  "assignees": ["user-uuid-1"]
}
```

**Response:**

```json
{
  "id": "issue-uuid",
  "name": "Fix critical login bug",
  "priority": "urgent",
  "state": {
    "id": "in-progress-state-uuid",
    "name": "In Progress"
  },
  "assignees": [
    {"id": "user-uuid-1", "display_name": "John Doe"}
  ],
  "updated_at": "2025-11-19T16:30:00Z",
  "updated_by": "user-uuid"
}
```

### Delete Issue

**Endpoint:** `DELETE /api/workspaces/{slug}/projects/{project_id}/issues/{issue_id}/`

**Request:**

```http
DELETE /api/workspaces/acme/projects/proj/issues/issue-uuid/
```

**Response:**

```http
HTTP/1.1 204 No Content
```

### Bulk Operations

**Bulk Update Issues:**

```http
PATCH /api/workspaces/{slug}/projects/{project_id}/issues/bulk/
Content-Type: application/json

{
  "issue_ids": ["issue-1", "issue-2", "issue-3"],
  "properties": {
    "state": "state-uuid",
    "priority": "high",
    "assignees": ["user-uuid-1"]
  }
}
```

**Bulk Delete Issues:**

```http
DELETE /api/workspaces/{slug}/projects/{project_id}/issues/bulk/
Content-Type: application/json

{
  "issue_ids": ["issue-1", "issue-2", "issue-3"]
}
```

### Issue Comments

**List Comments:**

```http
GET /api/workspaces/{slug}/projects/{proj}/issues/{issue}/comments/
```

**Create Comment:**

```json
POST /api/workspaces/{slug}/projects/{proj}/issues/{issue}/comments/

{
  "comment_html": "<p>Great work on this!</p>",
  "access": "INTERNAL"
}
```

**Update Comment:**

```json
PATCH /api/workspaces/{slug}/projects/{proj}/issues/{issue}/comments/{comment_id}/

{
  "comment_html": "<p>Updated comment</p>"
}
```

**Delete Comment:**

```http
DELETE /api/workspaces/{slug}/projects/{proj}/issues/{issue}/comments/{comment_id}/
```

### Issue Reactions

**Add Reaction:**

```json
POST /api/workspaces/{slug}/projects/{proj}/issues/{issue}/reactions/

{
  "reaction": "👍"
}
```

**Remove Reaction:**

```http
DELETE /api/workspaces/{slug}/projects/{proj}/issues/{issue}/reactions/{reaction_id}/
```

### Issue Attachments

**Upload Attachment:**

```http
POST /api/workspaces/{slug}/projects/{proj}/issues/{issue}/attachments/
Content-Type: multipart/form-data

asset: [file]
```

**Delete Attachment:**

```http
DELETE /api/workspaces/{slug}/projects/{proj}/issues/{issue}/attachments/{attachment_id}/
```

### Issue Links

**Add Link:**

```json
POST /api/workspaces/{slug}/projects/{proj}/issues/{issue}/links/

{
  "title": "Related PR",
  "url": "https://github.com/makeplane/plane/pull/123"
}
```

**Delete Link:**

```http
DELETE /api/workspaces/{slug}/projects/{proj}/issues/{issue}/links/{link_id}/
```

### Issue Relations

**Add Relation:**

```json
POST /api/workspaces/{slug}/projects/{proj}/issues/{issue}/relations/

{
  "related_issue": "related-issue-uuid",
  "relation_type": "blocked_by"
}
```

**Relation Types:**
- `blocked_by` / `blocking`
- `relates_to` (symmetric)
- `duplicate` (symmetric)
- `start_before` / `start_after`
- `finish_before` / `finish_after`
- `implemented_by` / `implements`

---

## State Endpoints

### List States

**Endpoint:** `GET /api/workspaces/{slug}/projects/{project_id}/states/`

**Response:**

```json
[
  {
    "id": "state-uuid-1",
    "name": "Backlog",
    "color": "#858E96",
    "group": "backlog",
    "sequence": 1,
    "is_triage": false,
    "default": true,
    "project": "project-uuid",
    "created_at": "2025-01-01T00:00:00Z"
  },
  {
    "id": "state-uuid-2",
    "name": "In Progress",
    "color": "#3A9EC2",
    "group": "started",
    "sequence": 2,
    "is_triage": false,
    "default": false,
    "project": "project-uuid",
    "created_at": "2025-01-01T00:00:00Z"
  },
  {
    "id": "state-uuid-3",
    "name": "Done",
    "color": "#16B364",
    "group": "completed",
    "sequence": 3,
    "is_triage": false,
    "default": false,
    "project": "project-uuid",
    "created_at": "2025-01-01T00:00:00Z"
  }
]
```

**State Groups:**
- `backlog`: Not started
- `unstarted`: Ready to start
- `started`: In progress
- `completed`: Finished
- `cancelled`: Won't do

### Create State

**Request:**

```json
POST /api/workspaces/{slug}/projects/{proj}/states/

{
  "name": "Under Review",
  "color": "#FFA500",
  "group": "started",
  "sequence": 3
}
```

### Update State

**Request:**

```json
PATCH /api/workspaces/{slug}/projects/{proj}/states/{state_id}/

{
  "name": "Code Review",
  "color": "#FFD700"
}
```

### Delete State

```http
DELETE /api/workspaces/{slug}/projects/{proj}/states/{state_id}/
```

Note: Cannot delete state if issues are assigned to it. Must reassign issues first.

---

## Module & Cycle Endpoints

### Modules

**List Modules:**

```http
GET /api/workspaces/{slug}/projects/{project_id}/modules/
```

**Create Module:**

```json
POST /api/workspaces/{slug}/projects/{proj}/modules/

{
  "name": "Authentication Module",
  "description": "All auth-related features",
  "start_date": "2025-11-01",
  "target_date": "2025-11-30",
  "status": "in-progress",
  "lead": "user-uuid"
}
```

**Add Issue to Module:**

```json
POST /api/workspaces/{slug}/projects/{proj}/modules/{module_id}/issues/

{
  "issues": ["issue-uuid-1", "issue-uuid-2"]
}
```

### Cycles

**List Cycles:**

```http
GET /api/workspaces/{slug}/projects/{project_id}/cycles/
```

**Create Cycle:**

```json
POST /api/workspaces/{slug}/projects/{proj}/cycles/

{
  "name": "Sprint 23",
  "description": "Two-week sprint",
  "start_date": "2025-11-18",
  "end_date": "2025-12-01"
}
```

**Add Issue to Cycle:**

```json
POST /api/workspaces/{slug}/projects/{proj}/cycles/{cycle_id}/issues/

{
  "issues": ["issue-uuid-1", "issue-uuid-2"]
}
```

---

## Error Handling

### Error Response Format

All errors follow a consistent format:

```json
{
  "error": "Error message for display",
  "error_code": "SPECIFIC_ERROR_CODE",
  "details": {
    "field": ["Field-specific error"]
  }
}
```

### Common Errors

**400 Bad Request:**

```json
{
  "error": "Invalid input provided",
  "details": {
    "name": ["This field is required."],
    "priority": ["Invalid choice. Must be one of: urgent, high, medium, low, none"]
  }
}
```

**401 Unauthorized:**

```json
{
  "error": "Authentication credentials were not provided.",
  "error_code": "AUTHENTICATION_REQUIRED"
}
```

**403 Forbidden:**

```json
{
  "error": "You do not have permission to perform this action.",
  "error_code": "PERMISSION_DENIED"
}
```

**404 Not Found:**

```json
{
  "error": "Not found.",
  "error_code": "RESOURCE_NOT_FOUND"
}
```

**429 Too Many Requests:**

```json
{
  "error": "Rate limit exceeded. Please try again later.",
  "error_code": "RATE_LIMIT_EXCEEDED",
  "retry_after": 60
}
```

**500 Internal Server Error:**

```json
{
  "error": "An error occurred processing your request."
}
```

---

## Rate Limiting

### Rate Limits

| Endpoint Type | Limit (Authenticated) | Limit (Anonymous) |
|---------------|----------------------|-------------------|
| Authentication | 30 requests/minute | 30 requests/minute |
| Read (GET) | 1000 requests/hour | 100 requests/hour |
| Write (POST/PATCH/DELETE) | 100 requests/hour | N/A |
| Bulk operations | 10 requests/hour | N/A |

### Rate Limit Headers

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1700410800
```

### Rate Limit Exceeded

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1700410800

{
  "error": "Rate limit exceeded. Please try again later.",
  "error_code": "RATE_LIMIT_EXCEEDED",
  "retry_after": 60
}
```

---

## Pagination

### Cursor-based Pagination

Plane uses cursor-based pagination for efficient large dataset traversal.

**Request:**

```http
GET /api/workspaces/acme/projects/?per_page=20
```

**Response:**

```json
{
  "next": "https://api.plane.so/api/workspaces/acme/projects/?per_page=20&cursor=cD0yMDI1LTEx",
  "prev": null,
  "results": [
    {
      "id": "project-1",
      "name": "Project 1"
    }
  ]
}
```

**Next Page:**

```http
GET /api/workspaces/acme/projects/?per_page=20&cursor=cD0yMDI1LTEx
```

### Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `per_page` | integer | Items per page (max 100) | 20 |
| `cursor` | string | Pagination cursor from previous response | null |

---

## Filtering & Sorting

### Filtering

**Multiple Values (OR):**

```http
GET /api/.../issues/?priority=urgent,high
# Returns issues with priority = urgent OR priority = high
```

**Date Filters:**

```http
# After date
GET /api/.../issues/?start_date=2025-11-01;after

# Before date
GET /api/.../issues/?target_date=2025-12-31;before

# Between dates
GET /api/.../issues/?created_at=2025-11-01;2025-11-30;between
```

**Search:**

```http
GET /api/.../issues/?search=authentication
# Searches in name and description_stripped fields
```

### Sorting

**Single Field:**

```http
GET /api/.../issues/?order_by=created_at
# Ascending order

GET /api/.../issues/?order_by=-created_at
# Descending order (prefix with -)
```

**Multiple Fields:**

```http
GET /api/.../issues/?order_by=-priority,-created_at
# Sort by priority desc, then created_at desc
```

**Available Sort Fields:**
- `created_at` / `-created_at`
- `updated_at` / `-updated_at`
- `priority` / `-priority`
- `sequence_id` / `-sequence_id`
- `sort_order` / `-sort_order`
- `target_date` / `-target_date`

### Grouping

**Group By:**

```http
GET /api/.../issues/?group_by=state
```

Returns issues grouped by state:

```json
{
  "backlog": [/* issues in backlog state */],
  "in_progress": [/* issues in progress */],
  "done": [/* completed issues */]
}
```

**Sub-Group By:**

```http
GET /api/.../issues/?group_by=state&sub_group_by=priority
```

Returns issues grouped by state, then sub-grouped by priority:

```json
{
  "backlog": {
    "urgent": [/* urgent issues in backlog */],
    "high": [/* high priority issues in backlog */]
  },
  "in_progress": {
    "urgent": [/* urgent issues in progress */]
  }
}
```

**Available Group Fields:**
- `state` - Group by issue state
- `priority` - Group by priority level
- `assignees` - Group by assigned users
- `labels` - Group by labels
- `created_by` - Group by creator
- `project` - Group by project

---

## React Analogy: API Client Pattern

**Backend (Django):**
```python
# File: apps/api/plane/app/views/issue/base.py

class IssueViewSet(BaseViewSet):
    @allow_permission([ROLE.ADMIN, ROLE.MEMBER])
    def list(self, request, slug, project_id):
        issues = Issue.objects.filter(project_id=project_id)
        return Response(IssueSerializer(issues, many=True).data)
```

**Frontend (React):**
```typescript
// File: apps/web/services/issue.service.ts

class IssueService {
  async list(workspaceSlug: string, projectId: string, params?: IssueFilters) {
    const response = await apiClient.get(
      `/workspaces/${workspaceSlug}/projects/${projectId}/issues/`,
      { params }
    );
    return response.data;
  }
}

// Usage in component:
const issues = await issueService.list('acme', 'proj-123', {
  priority: 'urgent,high',
  state: 'in-progress',
  order_by: '-created_at'
});
```

**Key Insight:** Backend defines the API contract (serializers, validators), frontend consumes it with type-safe services!

---

## Status: ✅ CURRENT (November 2025)

All API endpoints documented here reflect the current implementation as of:

- ✅ Django 4.2
- ✅ Django REST Framework 3.15
- ✅ Plane API v1

---

## Further Reading

### Related Documentation
- [SECURITY_GUIDE.md](./SECURITY_GUIDE.md) - API security best practices
- [TESTING_GUIDE.md](./TESTING_GUIDE.md) - Testing API endpoints
- [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md) - Integrating with Plane API

### External Resources
- [Django REST Framework](https://www.django-rest-framework.org/)
- [REST API Best Practices](https://restfulapi.net/)
- [HTTP Status Codes](https://httpstatuses.com/)

---

**Happy coding!** If you find issues or have questions about the API, check the [FAQ](./FAQ.md) or open an issue on GitHub.
