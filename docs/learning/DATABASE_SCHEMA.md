# Database Schema

> **Last Updated:** November 2025
> **Status:** ✅ CURRENT - Complete PostgreSQL schema reference for Plane

A comprehensive database schema reference for Plane, documenting all tables, relationships, indexes, and constraints.

---

## Table of Contents

1. [Schema Overview](#schema-overview)
2. [Entity Relationship Diagram](#entity-relationship-diagram)
3. [Core Models](#core-models)
4. [User & Authentication](#user--authentication)
5. [Workspace Models](#workspace-models)
6. [Project Models](#project-models)
7. [Issue Models](#issue-models)
8. [Workflow Models](#workflow-models)
9. [Collaboration Models](#collaboration-models)
10. [System Models](#system-models)
11. [Indexes & Performance](#indexes--performance)
12. [Migrations](#migrations)

---

## Schema Overview

### Database Technology

- **Database**: PostgreSQL 14+
- **ORM**: Django 4.2 ORM
- **Extensions**: UUID-OSSP, pg_trgm (for text search)

### Design Principles

1. **Soft Deletion**: Most tables use `deleted_at` for soft deletes
2. **Audit Trail**: All tables include created_at, updated_at, created_by, updated_by
3. **UUIDs**: Primary keys use UUID v4 for distributed systems
4. **Hierarchical Data**: Workspace → Project → Issue relationship
5. **Many-to-Many**: Through tables for complex relationships

### Table Count

```
Total Tables: 65+
Core Tables: 15
Join Tables: 20
System Tables: 10
```

---

## Entity Relationship Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                      PLANE DATABASE SCHEMA                        │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                         USERS & AUTH                             │
└─────────────────────────────────────────────────────────────────┘

        ┌──────────┐
        │  users   │◄──────────┐
        └──────────┘           │
             │                 │
             │ 1               │ N
             │                 │
    ┌────────┼────────┐  ┌──────────────┐
    │        │        │  │  accounts    │
    │        │        │  │ (OAuth)      │
    │        │        │  └──────────────┘
    │        │        │
    │        │        │  ┌──────────────┐
    │        │        └──┤  sessions    │
    │        │           └──────────────┘
    │        │
    │        │           ┌──────────────┐
    │        └───────────┤  profiles    │
    │                    └──────────────┘
    │

┌──────────────────────────────────────────────────────────────────┐
│                         WORKSPACES                                │
└──────────────────────────────────────────────────────────────────┘

    │
    │ N                   ┌────────────────┐
    └────────────────────►│  workspaces    │
                          └────────────────┘
                                 │
                                 │ 1
                                 │
                    ┌────────────┼───────────┐
                    │            │           │
                    │ N          │ N         │ N
                    │            │           │
          ┌─────────────┐  ┌────────────┐  ┌──────────────┐
          │ workspace_  │  │  labels    │  │   projects   │
          │ members     │  │            │  │              │
          └─────────────┘  └────────────┘  └──────────────┘
                                                   │
                                                   │ 1
                                                   │

┌──────────────────────────────────────────────────────────────────┐
│                           PROJECTS                                │
└──────────────────────────────────────────────────────────────────┘

                    ┌──────────────────────────────┐
                    │                              │
            ┌───────┼──────────┬──────────┬────────┼───────┐
            │       │          │          │        │       │
            │ N     │ N        │ N        │ N      │ N     │ N
            │       │          │          │        │       │
    ┌─────────┐  ┌─────┐  ┌────────┐  ┌──────┐  ┌────────┐  ┌─────────┐
    │ project │  │state│  │ cycles │  │module│  │estimate│  │  issue  │
    │ members │  │     │  │        │  │      │  │        │  │  types  │
    └─────────┘  └─────┘  └────────┘  └──────┘  └────────┘  └─────────┘
                    │                     │
                    │                     │

┌──────────────────────────────────────────────────────────────────┐
│                            ISSUES                                 │
└──────────────────────────────────────────────────────────────────┘

                    │                     │
                    │ 1                   │ 1
                    │                     │
                 ┌────────┐               │
                 │ issues │◄──────────────┘
                 └────────┘
                     │
        ┌────────────┼────────────┬─────────────┬──────────────┐
        │            │            │             │              │
        │ N          │ N          │ N           │ N            │ N
        │            │            │             │              │
 ┌──────────┐  ┌─────────┐  ┌─────────┐  ┌──────────┐  ┌──────────┐
 │  issue   │  │  issue  │  │  issue  │  │  issue   │  │  issue   │
 │assignees │  │ labels  │  │comments │  │attachmnts│  │ activity │
 └──────────┘  └─────────┘  └─────────┘  └──────────┘  └──────────┘

        │            │            │             │              │
        │            │            │             │              │
 ┌──────────┐  ┌─────────┐  ┌─────────┐  ┌──────────┐  ┌──────────┐
 │  issue   │  │  issue  │  │  issue  │  │  cycle   │  │  module  │
 │  links   │  │reactions│  │relations│  │  issues  │  │  issues  │
 └──────────┘  └─────────┘  └─────────┘  └──────────┘  └──────────┘

┌──────────────────────────────────────────────────────────────────┐
│                       ADDITIONAL MODELS                           │
└──────────────────────────────────────────────────────────────────┘

    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │  pages   │    │ webhooks │    │  intakes │    │favorites │
    └──────────┘    └──────────┘    └──────────┘    └──────────┘

    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │  views   │    │analytics │    │  deploy  │    │  assets  │
    │          │    │          │    │  boards  │    │   (S3)   │
    └──────────┘    └──────────┘    └──────────┘    └──────────┘
```

---

## Core Models

### Base Model Pattern

All models inherit from base classes providing common fields:

```python
# BaseModel - All models inherit from this
class BaseModel(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid4)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    created_by = models.ForeignKey(User, related_name='+', null=True)
    updated_by = models.ForeignKey(User, related_name='+', null=True)
    deleted_at = models.DateTimeField(null=True)  # Soft delete

# WorkspaceBaseModel - Workspace-scoped models
class WorkspaceBaseModel(BaseModel):
    workspace = models.ForeignKey(Workspace, on_delete=models.CASCADE)
    project = models.ForeignKey(Project, null=True)  # Optional project scope

# ProjectBaseModel - Project-scoped models
class ProjectBaseModel(BaseModel):
    workspace = models.ForeignKey(Workspace, on_delete=models.CASCADE)
    project = models.ForeignKey(Project, on_delete=models.CASCADE)
```

**Common Fields (All Tables)**:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key (UUID v4) |
| `created_at` | TIMESTAMP | Creation timestamp |
| `updated_at` | TIMESTAMP | Last modification timestamp |
| `created_by` | UUID | User who created (FK to users) |
| `updated_by` | UUID | User who last updated (FK to users) |
| `deleted_at` | TIMESTAMP | Soft deletion timestamp (NULL = active) |

---

## User & Authentication

### `users` Table

Core user model with authentication and profile data.

**Table Name**: `users`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | User unique identifier |
| `username` | VARCHAR(128) | UNIQUE, NOT NULL | Username (typically email prefix) |
| `email` | VARCHAR(255) | UNIQUE, NOT NULL | Email address (login identifier) |
| `password` | VARCHAR(128) | NOT NULL | Hashed password (PBKDF2) |
| `display_name` | VARCHAR(255) | NOT NULL | Display name for UI |
| `first_name` | VARCHAR(255) | | First name |
| `last_name` | VARCHAR(255) | | Last name |
| `avatar` | TEXT | | Avatar URL |
| `avatar_asset` | UUID | FK → file_assets | Uploaded avatar |
| `cover_image` | VARCHAR(800) | | Cover image URL |
| `cover_image_asset` | UUID | FK → file_assets | Uploaded cover image |
| `mobile_number` | VARCHAR(255) | | Phone number |
| `user_timezone` | VARCHAR(255) | DEFAULT 'UTC' | User timezone |
| `is_active` | BOOLEAN | DEFAULT TRUE | Account active status |
| `is_staff` | BOOLEAN | DEFAULT FALSE | Django admin access |
| `is_superuser` | BOOLEAN | DEFAULT FALSE | Superuser status |
| `is_email_verified` | BOOLEAN | DEFAULT FALSE | Email verification status |
| `is_password_autoset` | BOOLEAN | DEFAULT FALSE | Password auto-generated (OAuth) |
| `is_password_expired` | BOOLEAN | DEFAULT FALSE | Force password change |
| `is_bot` | BOOLEAN | DEFAULT FALSE | Bot account flag |
| `bot_type` | VARCHAR(30) | | Bot type identifier |
| `is_managed` | BOOLEAN | DEFAULT FALSE | Managed by system |
| `token` | VARCHAR(64) | | API token |
| `token_updated_at` | TIMESTAMP | | Token generation time |
| `last_active` | TIMESTAMP | | Last activity timestamp |
| `last_login_time` | TIMESTAMP | | Last login timestamp |
| `last_logout_time` | TIMESTAMP | | Last logout timestamp |
| `last_login_ip` | VARCHAR(255) | | Last login IP address |
| `last_logout_ip` | VARCHAR(255) | | Last logout IP address |
| `last_login_medium` | VARCHAR(20) | DEFAULT 'email' | Login method |
| `last_login_uagent` | TEXT | | User agent string |
| `last_location` | VARCHAR(255) | | Last known location |
| `created_location` | VARCHAR(255) | | Signup location |
| `masked_at` | TIMESTAMP | | GDPR masking timestamp |
| `date_joined` | TIMESTAMP | AUTO | Account creation date |
| `created_at` | TIMESTAMP | AUTO | Record creation |
| `updated_at` | TIMESTAMP | AUTO | Last update |

**Indexes**:
- PRIMARY KEY on `id`
- UNIQUE INDEX on `username`
- UNIQUE INDEX on `email`
- INDEX on `is_active`
- INDEX on `email` (for lookups)

**Example Row**:

```sql
id                  | 123e4567-e89b-12d3-a456-426614174000
username            | john.doe
email               | john.doe@example.com
password            | pbkdf2_sha256$600000$xyz...
display_name        | John Doe
first_name          | John
last_name           | Doe
user_timezone       | America/New_York
is_active           | true
is_email_verified   | true
is_staff            | false
last_login_time     | 2025-11-19 10:00:00+00
created_at          | 2024-01-15 08:00:00+00
```

### `profiles` Table

Extended user profile information and preferences.

**Table Name**: `profiles`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Profile ID |
| `user` | UUID | FK → users, UNIQUE | One-to-one with user |
| `theme` | JSONB | DEFAULT '{}' | UI theme preferences |
| `is_app_rail_docked` | BOOLEAN | DEFAULT TRUE | UI layout preference |
| `is_tour_completed` | BOOLEAN | DEFAULT FALSE | Onboarding tour status |
| `is_onboarded` | BOOLEAN | DEFAULT FALSE | Onboarding completion |
| `onboarding_step` | JSONB | | Current onboarding step |
| `is_mobile_onboarded` | BOOLEAN | DEFAULT FALSE | Mobile onboarding status |
| `mobile_onboarding_step` | JSONB | | Mobile onboarding progress |
| `mobile_timezone_auto_set` | BOOLEAN | DEFAULT FALSE | Auto timezone detection |
| `use_case` | TEXT | | User's use case |
| `role` | VARCHAR(300) | | Job role |
| `last_workspace_id` | UUID | FK → workspaces | Last visited workspace |
| `language` | VARCHAR(255) | DEFAULT 'en' | UI language |
| `start_of_the_week` | SMALLINT | DEFAULT 0 | Week start day (0=Sunday) |
| `goals` | JSONB | DEFAULT '{}' | User goals |
| `background_color` | VARCHAR(255) | | Profile background color |
| `is_smooth_cursor_enabled` | BOOLEAN | DEFAULT FALSE | Smooth cursor animation |
| `billing_address_country` | VARCHAR(255) | DEFAULT 'INDIA' | Billing country |
| `billing_address` | JSONB | | Full billing address |
| `has_billing_address` | BOOLEAN | DEFAULT FALSE | Billing address set |
| `company_name` | VARCHAR(255) | | Company name |
| `has_marketing_email_consent` | BOOLEAN | DEFAULT FALSE | Marketing emails opt-in |

**Example Row**:

```sql
user                | 123e4567-e89b-12d3-a456-426614174000
theme               | {"theme": "dark", "accent": "blue"}
is_onboarded        | true
is_tour_completed   | true
last_workspace_id   | workspace-uuid
language            | en
start_of_the_week   | 0
```

### `accounts` Table

OAuth provider accounts linked to users.

**Table Name**: `accounts`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Account ID |
| `user` | UUID | FK → users | User owning this account |
| `provider` | VARCHAR | NOT NULL | OAuth provider (google, github, gitlab) |
| `provider_account_id` | VARCHAR(255) | NOT NULL | ID from OAuth provider |
| `access_token` | TEXT | NOT NULL | OAuth access token |
| `access_token_expired_at` | TIMESTAMP | | Token expiry time |
| `refresh_token` | TEXT | | OAuth refresh token |
| `refresh_token_expired_at` | TIMESTAMP | | Refresh token expiry |
| `id_token` | TEXT | | OpenID Connect ID token |
| `metadata` | JSONB | DEFAULT '{}' | Additional OAuth metadata |
| `last_connected_at` | TIMESTAMP | DEFAULT NOW | Last OAuth connection |

**Constraints**:
- UNIQUE (`provider`, `provider_account_id`)

**Example Row**:

```sql
user                     | 123e4567-e89b-12d3-a456-426614174000
provider                 | google
provider_account_id      | 1234567890
access_token            | ya29.a0AfH6SMB...
access_token_expired_at | 2025-11-19 12:00:00+00
last_connected_at       | 2025-11-19 10:00:00+00
```

### `sessions` Table

User sessions for authentication tracking.

**Table Name**: `sessions`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Session ID |
| `user` | UUID | FK → users | User owning session |
| `ip_address` | VARCHAR(255) | | Client IP address |
| `user_agent` | TEXT | | Client user agent |
| `last_activity` | TIMESTAMP | AUTO | Last activity time |
| `expires_at` | TIMESTAMP | NOT NULL | Session expiration |

**Indexes**:
- INDEX on (`user`, `expires_at`)
- INDEX on `expires_at` (for cleanup)

---

## Workspace Models

### `workspaces` Table

Top-level organizational container.

**Table Name**: `workspaces`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Workspace ID |
| `name` | VARCHAR(80) | NOT NULL | Workspace name |
| `slug` | VARCHAR(48) | UNIQUE, NOT NULL | URL-friendly identifier |
| `owner` | UUID | FK → users, NOT NULL | Workspace owner |
| `logo` | TEXT | | Logo URL |
| `logo_asset` | UUID | FK → file_assets | Uploaded logo |
| `organization_size` | VARCHAR(20) | | Company size category |
| `timezone` | VARCHAR(255) | DEFAULT 'UTC' | Workspace timezone |
| `background_color` | VARCHAR(255) | | Workspace accent color |
| `created_at` | TIMESTAMP | AUTO | Creation timestamp |
| `updated_at` | TIMESTAMP | AUTO | Last update |
| `created_by` | UUID | FK → users | Creator |
| `updated_by` | UUID | FK → users | Last updater |
| `deleted_at` | TIMESTAMP | | Soft deletion |

**Indexes**:
- PRIMARY KEY on `id`
- UNIQUE INDEX on `slug`
- INDEX on `owner`

**Example Row**:

```sql
id                  | workspace-uuid-1
name                | Acme Corp
slug                | acme
owner               | user-uuid-1
organization_size   | 50-200
timezone            | America/New_York
created_at          | 2024-01-01 00:00:00+00
```

### `workspace_members` Table

Workspace membership with roles.

**Table Name**: `workspace_members`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Membership ID |
| `workspace` | UUID | FK → workspaces, NOT NULL | Workspace |
| `member` | UUID | FK → users, NOT NULL | User member |
| `role` | INTEGER | NOT NULL | Role level (5=Guest, 15=Member, 20=Admin) |
| `is_active` | BOOLEAN | DEFAULT TRUE | Membership active |
| `view_props` | JSONB | DEFAULT '{}' | View preferences |
| `default_props` | JSONB | DEFAULT '{}' | Default properties |
| `sort_order` | FLOAT | DEFAULT 65535 | Display sort order |

**Constraints**:
- UNIQUE (`workspace`, `member`) WHERE `deleted_at` IS NULL

**Indexes**:
- INDEX on (`workspace`, `member`, `is_active`)
- INDEX on `role`

**Roles**:
- `20` - Admin (full permissions)
- `15` - Member (standard permissions)
- `5` - Guest (read-only)

**Example Row**:

```sql
workspace   | workspace-uuid-1
member      | user-uuid-1
role        | 20
is_active   | true
```

---

## Project Models

### `projects` Table

Projects within workspaces.

**Table Name**: `projects`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Project ID |
| `workspace` | UUID | FK → workspaces, NOT NULL | Parent workspace |
| `name` | VARCHAR(255) | NOT NULL | Project name |
| `identifier` | VARCHAR(12) | NOT NULL | Short code (e.g., "PROJ") |
| `description` | TEXT | | Project description |
| `description_text` | JSONB | | Rich text description |
| `description_html` | JSONB | | HTML description |
| `network` | SMALLINT | DEFAULT 2 | 0=Secret, 2=Public |
| `project_lead` | UUID | FK → users | Project lead |
| `default_assignee` | UUID | FK → users | Default assignee for issues |
| `default_state` | UUID | FK → states | Default state for issues |
| `estimate` | UUID | FK → estimates | Estimation system |
| `emoji` | VARCHAR(255) | | Project emoji icon |
| `icon_prop` | JSONB | | Icon properties |
| `cover_image` | TEXT | | Cover image URL |
| `cover_image_asset` | UUID | FK → file_assets | Uploaded cover |
| `module_view` | BOOLEAN | DEFAULT FALSE | Enable modules feature |
| `cycle_view` | BOOLEAN | DEFAULT FALSE | Enable cycles feature |
| `issue_views_view` | BOOLEAN | DEFAULT FALSE | Enable issue views |
| `page_view` | BOOLEAN | DEFAULT TRUE | Enable pages feature |
| `intake_view` | BOOLEAN | DEFAULT FALSE | Enable intake/triage |
| `is_time_tracking_enabled` | BOOLEAN | DEFAULT FALSE | Enable time tracking |
| `is_issue_type_enabled` | BOOLEAN | DEFAULT FALSE | Enable issue types |
| `guest_view_all_features` | BOOLEAN | DEFAULT FALSE | Guest feature access |
| `archive_in` | INTEGER | DEFAULT 0 | Auto-archive after N months |
| `close_in` | INTEGER | DEFAULT 0 | Auto-close after N months |
| `logo_props` | JSONB | DEFAULT '{}' | Logo properties |
| `timezone` | VARCHAR(255) | DEFAULT 'UTC' | Project timezone |
| `archived_at` | TIMESTAMP | | Archive timestamp |
| `external_source` | VARCHAR(255) | | Import source |
| `external_id` | VARCHAR(255) | | External ID |

**Constraints**:
- UNIQUE (`workspace`, `identifier`) WHERE `deleted_at` IS NULL
- UNIQUE (`workspace`, `name`) WHERE `deleted_at` IS NULL

**Indexes**:
- INDEX on (`workspace`, `archived_at`)
- INDEX on `identifier`
- INDEX on `project_lead`

**Example Row**:

```sql
id              | project-uuid-1
workspace       | workspace-uuid-1
name            | Website Redesign
identifier      | WEB
network         | 2
project_lead    | user-uuid-1
module_view     | true
cycle_view      | true
created_at      | 2024-03-01 00:00:00+00
```

### `project_members` Table

Project membership with role-based permissions.

**Table Name**: `project_members`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Membership ID |
| `workspace` | UUID | FK → workspaces | Workspace (denormalized) |
| `project` | UUID | FK → projects, NOT NULL | Project |
| `member` | UUID | FK → users, NOT NULL | User member |
| `role` | INTEGER | NOT NULL | Role (5=Guest, 15=Member, 20=Admin) |
| `is_active` | BOOLEAN | DEFAULT TRUE | Membership active |
| `view_props` | JSONB | DEFAULT '{}' | View preferences |
| `default_props` | JSONB | DEFAULT '{}' | Default properties |
| `sort_order` | FLOAT | DEFAULT 65535 | Sort order |

**Constraints**:
- UNIQUE (`project`, `member`) WHERE `deleted_at` IS NULL

**Indexes**:
- INDEX on (`project`, `member`, `is_active`)
- INDEX on (`workspace`, `member`)

---

## Issue Models

### `issues` Table

Core issue/task model.

**Table Name**: `issues`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Issue ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `sequence_id` | INTEGER | NOT NULL | Sequential number per project |
| `name` | VARCHAR(255) | NOT NULL | Issue title |
| `description` | JSONB | DEFAULT '{}' | Rich text description (TipTap) |
| `description_html` | TEXT | DEFAULT '<p></p>' | HTML description |
| `description_stripped` | TEXT | | Plain text description |
| `description_binary` | BYTEA | | Binary description format |
| `state` | UUID | FK → states | Current state |
| `priority` | VARCHAR(30) | DEFAULT 'none' | urgent, high, medium, low, none |
| `start_date` | DATE | | Planned start date |
| `target_date` | DATE | | Due date |
| `estimate_point` | UUID | FK → estimate_points | Story points |
| `point` | INTEGER | | Legacy point value (0-12) |
| `parent` | UUID | FK → issues | Parent issue (for sub-issues) |
| `type` | UUID | FK → issue_types | Issue type |
| `sort_order` | FLOAT | DEFAULT 65535 | Sort order within state |
| `completed_at` | TIMESTAMP | | Completion timestamp |
| `archived_at` | DATE | | Archive date |
| `is_draft` | BOOLEAN | DEFAULT FALSE | Draft status |
| `external_source` | VARCHAR(255) | | Import source |
| `external_id` | VARCHAR(255) | | External ID |

**Relationships**:
- Many-to-Many with `users` through `issue_assignees`
- Many-to-Many with `labels` through `issue_labels`
- Many-to-Many with `cycles` through `cycle_issues`
- Many-to-Many with `modules` through `module_issues`

**Constraints**:
- CHECK (`point` BETWEEN 0 AND 12)

**Indexes**:
- PRIMARY KEY on `id`
- INDEX on (`project`, `sequence_id`)
- INDEX on (`project`, `state`)
- INDEX on (`project`, `priority`)
- INDEX on `target_date`
- INDEX on `parent` (for sub-issues)
- FULL TEXT INDEX on (`name`, `description_stripped`) using `gin(to_tsvector())`

**Example Row**:

```sql
id                  | issue-uuid-1
project             | project-uuid-1
sequence_id         | 123
name                | Implement user authentication
state               | state-uuid-1
priority            | high
start_date          | 2025-11-19
target_date         | 2025-11-25
sort_order          | 10000
completed_at        | null
is_draft            | false
created_at          | 2025-11-15 10:00:00+00
```

### `issue_sequences` Table

Tracks issue sequence numbers per project.

**Table Name**: `issue_sequences`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Sequence record ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `issue` | UUID | FK → issues, nullable | Issue (NULL if deleted) |
| `sequence` | BIGINT | NOT NULL | Sequence number |
| `deleted` | BOOLEAN | DEFAULT FALSE | Deleted flag |

**Purpose**: Maintains atomic sequence generation for issues. Even if an issue is deleted, its sequence number is preserved.

**Indexes**:
- INDEX on (`project`, `sequence`)

### `issue_assignees` Table

Join table for issue-user assignments.

**Table Name**: `issue_assignees`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Assignment ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `issue` | UUID | FK → issues | Issue |
| `assignee` | UUID | FK → users | Assigned user |

**Constraints**:
- UNIQUE (`issue`, `assignee`) WHERE `deleted_at` IS NULL

**Indexes**:
- INDEX on (`issue`, `assignee`)
- INDEX on (`assignee`, `project`)

### `issue_labels` Table

Join table for issue-label associations.

**Table Name**: `issue_labels`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Label assignment ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `issue` | UUID | FK → issues | Issue |
| `label` | UUID | FK → labels | Label |

**Indexes**:
- INDEX on (`issue`, `label`)
- INDEX on (`label`, `project`)

### `issue_comments` Table

Comments on issues.

**Table Name**: `issue_comments`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Comment ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `issue` | UUID | FK → issues | Issue |
| `actor` | UUID | FK → users, nullable | Comment author (NULL = system) |
| `comment_json` | JSONB | DEFAULT '{}' | Rich text comment |
| `comment_html` | TEXT | DEFAULT '<p></p>' | HTML comment |
| `comment_stripped` | TEXT | | Plain text |
| `attachments` | TEXT ARRAY | DEFAULT '[]' | Attachment URLs |
| `access` | VARCHAR(100) | DEFAULT 'INTERNAL' | INTERNAL or EXTERNAL |
| `external_source` | VARCHAR(255) | | Import source |
| `external_id` | VARCHAR(255) | | External ID |
| `edited_at` | TIMESTAMP | | Last edit time |

**Indexes**:
- INDEX on (`issue`, `created_at`)
- INDEX on `actor`

### `issue_attachments` Table

File attachments for issues.

**Table Name**: `issue_attachments`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Attachment ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `issue` | UUID | FK → issues | Issue |
| `asset` | FILE | NOT NULL | Uploaded file |
| `attributes` | JSONB | DEFAULT '{}' | File metadata (name, size, type) |
| `external_source` | VARCHAR(255) | | Import source |
| `external_id` | VARCHAR(255) | | External ID |

**File Storage**: Uses Django's FileField, typically stored in S3.

**Indexes**:
- INDEX on `issue`

### `issue_links` Table

External links associated with issues.

**Table Name**: `issue_links`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Link ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `issue` | UUID | FK → issues | Issue |
| `title` | VARCHAR(255) | | Link title |
| `url` | TEXT | NOT NULL | URL |
| `metadata` | JSONB | DEFAULT '{}' | Open Graph metadata |

**Indexes**:
- INDEX on `issue`

### `issue_relations` Table

Relationships between issues.

**Table Name**: `issue_relations`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Relation ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `issue` | UUID | FK → issues | Source issue |
| `related_issue` | UUID | FK → issues | Target issue |
| `relation_type` | VARCHAR(20) | NOT NULL | Relation type |

**Relation Types**:
- `blocked_by` / `blocking`
- `relates_to` (symmetric)
- `duplicate` (symmetric)
- `start_before` / `start_after`
- `finish_before` / `finish_after`
- `implemented_by` / `implements`

**Constraints**:
- UNIQUE (`issue`, `related_issue`) WHERE `deleted_at` IS NULL

### `issue_reactions` Table

Emoji reactions on issues.

**Table Name**: `issue_reactions`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Reaction ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `issue` | UUID | FK → issues | Issue |
| `actor` | UUID | FK → users | User who reacted |
| `reaction` | TEXT | NOT NULL | Emoji |

**Constraints**:
- UNIQUE (`issue`, `actor`, `reaction`) WHERE `deleted_at` IS NULL

### `issue_activity` Table

Audit trail for issue changes.

**Table Name**: `issue_activities`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Activity ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `issue` | UUID | FK → issues | Issue |
| `actor` | UUID | FK → users, nullable | User who made change |
| `verb` | VARCHAR(255) | DEFAULT 'created' | Action type |
| `field` | VARCHAR(255) | | Changed field name |
| `old_value` | TEXT | | Previous value |
| `new_value` | TEXT | | New value |
| `old_identifier` | UUID | | Old related object ID |
| `new_identifier` | UUID | | New related object ID |
| `comment` | TEXT | | Activity comment |
| `attachments` | TEXT ARRAY | DEFAULT '[]' | Attachment URLs |
| `issue_comment` | UUID | FK → issue_comments | Related comment |
| `epoch` | FLOAT | | Unix timestamp |

**Indexes**:
- INDEX on (`issue`, `created_at DESC`)
- INDEX on `actor`

---

## Workflow Models

### `states` Table

Issue workflow states.

**Table Name**: `states`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | State ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `name` | VARCHAR(255) | NOT NULL | State name (e.g., "In Progress") |
| `description` | TEXT | | State description |
| `color` | VARCHAR(255) | NOT NULL | Hex color code |
| `slug` | VARCHAR(100) | | URL-friendly name |
| `sequence` | FLOAT | DEFAULT 65535 | Display order |
| `group` | VARCHAR(20) | DEFAULT 'backlog' | State group |
| `is_triage` | BOOLEAN | DEFAULT FALSE | Triage state flag |
| `default` | BOOLEAN | DEFAULT FALSE | Default state for project |
| `external_source` | VARCHAR(255) | | Import source |
| `external_id` | VARCHAR(255) | | External ID |

**State Groups**:
- `backlog` - Not started
- `unstarted` - Ready to start
- `started` - In progress
- `completed` - Done
- `cancelled` - Won't do
- `triage` - Needs triage

**Constraints**:
- UNIQUE (`project`, `name`) WHERE `deleted_at` IS NULL

**Indexes**:
- INDEX on (`project`, `sequence`)
- INDEX on `group`

**Example Rows**:

```sql
project | name        | color     | group      | sequence
--------|-------------|-----------|------------|----------
proj-1  | Backlog     | #858E96   | backlog    | 10000
proj-1  | Todo        | #858E96   | unstarted  | 20000
proj-1  | In Progress | #3A9EC2   | started    | 30000
proj-1  | Done        | #16B364   | completed  | 40000
```

### `cycles` Table

Time-boxed sprints/iterations.

**Table Name**: `cycles`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Cycle ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `name` | VARCHAR(255) | NOT NULL | Cycle name (e.g., "Sprint 23") |
| `description` | TEXT | | Cycle description |
| `start_date` | TIMESTAMP | | Start date/time |
| `end_date` | TIMESTAMP | | End date/time |
| `owned_by` | UUID | FK → users | Cycle owner |
| `view_props` | JSONB | DEFAULT '{}' | View properties |
| `sort_order` | FLOAT | DEFAULT 65535 | Sort order |
| `progress_snapshot` | JSONB | DEFAULT '{}' | Progress metrics cache |
| `archived_at` | TIMESTAMP | | Archive timestamp |
| `logo_props` | JSONB | DEFAULT '{}' | Logo properties |
| `timezone` | VARCHAR(255) | DEFAULT 'UTC' | Cycle timezone |
| `version` | INTEGER | DEFAULT 1 | Version number |

**Indexes**:
- INDEX on (`project`, `start_date`, `end_date`)
- INDEX on `owned_by`

### `cycle_issues` Table

Join table for cycle-issue association.

**Table Name**: `cycle_issues`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Association ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `cycle` | UUID | FK → cycles | Cycle |
| `issue` | UUID | FK → issues | Issue |

**Constraints**:
- UNIQUE (`cycle`, `issue`) WHERE `deleted_at` IS NULL

**Indexes**:
- INDEX on (`cycle`, `issue`)
- INDEX on `issue`

### `modules` Table

Feature modules or milestones.

**Table Name**: `modules`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Module ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `name` | VARCHAR(255) | NOT NULL | Module name |
| `description` | TEXT | | Description |
| `description_text` | JSONB | | Rich text description |
| `description_html` | JSONB | | HTML description |
| `start_date` | DATE | | Start date |
| `target_date` | DATE | | Target completion |
| `status` | VARCHAR(20) | DEFAULT 'planned' | Module status |
| `lead` | UUID | FK → users, nullable | Module lead |
| `view_props` | JSONB | DEFAULT '{}' | View properties |
| `sort_order` | FLOAT | DEFAULT 65535 | Sort order |
| `archived_at` | TIMESTAMP | | Archive timestamp |
| `logo_props` | JSONB | DEFAULT '{}' | Logo properties |

**Module Statuses**:
- `backlog` - Not started
- `planned` - Planned
- `in-progress` - Active
- `paused` - Paused
- `completed` - Finished
- `cancelled` - Cancelled

**Constraints**:
- UNIQUE (`project`, `name`) WHERE `deleted_at` IS NULL

**Indexes**:
- INDEX on (`project`, `status`)
- INDEX on (`project`, `target_date`)

### `module_issues` Table

Join table for module-issue association.

**Table Name**: `module_issues`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Association ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects | Project |
| `module` | UUID | FK → modules | Module |
| `issue` | UUID | FK → issues | Issue |

**Constraints**:
- UNIQUE (`module`, `issue`) WHERE `deleted_at` IS NULL

**Indexes**:
- INDEX on (`module`, `issue`)
- INDEX on `issue`

---

## Collaboration Models

### `labels` Table

Tags for categorizing issues.

**Table Name**: `labels`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Label ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects, nullable | Project (NULL = workspace-wide) |
| `parent` | UUID | FK → labels, nullable | Parent label (for nesting) |
| `name` | VARCHAR(255) | NOT NULL | Label name |
| `description` | TEXT | | Description |
| `color` | VARCHAR(255) | | Hex color code |
| `sort_order` | FLOAT | DEFAULT 65535 | Sort order |

**Constraints**:
- UNIQUE (`project`, `name`) WHERE `deleted_at` IS NULL AND `project` IS NOT NULL
- UNIQUE (`workspace`, `name`) WHERE `deleted_at` IS NULL AND `project` IS NULL

**Indexes**:
- INDEX on (`workspace`, `project`)
- INDEX on `parent`

### `favorites` Table

User favorites (workspaces, projects, issues, etc.).

**Table Name**: `user_favorites`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Favorite ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects, nullable | Project (if entity is project-scoped) |
| `user` | UUID | FK → users | User who favorited |
| `entity_type` | VARCHAR(50) | NOT NULL | Type (workspace, project, issue, etc.) |
| `entity_identifier` | UUID | NOT NULL | ID of favorited entity |

**Constraints**:
- UNIQUE (`user`, `entity_type`, `entity_identifier`) WHERE `deleted_at` IS NULL

**Indexes**:
- INDEX on (`user`, `entity_type`)
- INDEX on `entity_identifier`

### `notifications` Table

User notifications.

**Table Name**: `notifications`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Notification ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects, nullable | Project |
| `sender` | UUID | FK → users | User who triggered notification |
| `receiver` | UUID | FK → users | User receiving notification |
| `entity_type` | VARCHAR(50) | | Entity type (issue, comment, etc.) |
| `entity_identifier` | UUID | | Entity ID |
| `title` | TEXT | | Notification title |
| `data` | JSONB | DEFAULT '{}' | Notification data |
| `message` | TEXT | | Notification message |
| `message_html` | TEXT | DEFAULT '<p></p>' | HTML message |
| `message_stripped` | TEXT | | Plain text message |
| `read_at` | TIMESTAMP | | Read timestamp |
| `archived_at` | TIMESTAMP | | Archive timestamp |
| `snoozed_till` | TIMESTAMP | | Snooze until time |

**Indexes**:
- INDEX on (`receiver`, `read_at`, `created_at DESC`)
- INDEX on (`workspace`, `entity_type`, `entity_identifier`)

---

## System Models

### `file_assets` Table

Uploaded files (avatars, attachments, etc.).

**Table Name**: `file_assets`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Asset ID |
| `workspace` | UUID | FK → workspaces, nullable | Workspace (if scoped) |
| `attributes` | JSONB | DEFAULT '{}' | File metadata |
| `asset` | FILE | NOT NULL | File path (S3 or local) |
| `entity_type` | VARCHAR(50) | | Entity type (issue_attachment, avatar, etc.) |
| `entity_identifier` | UUID | | Entity ID |
| `is_deleted` | BOOLEAN | DEFAULT FALSE | Deletion flag |

**Indexes**:
- INDEX on (`entity_type`, `entity_identifier`)
- INDEX on `workspace`

### `webhooks` Table

Outgoing webhook configurations.

**Table Name**: `webhooks`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Webhook ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects, nullable | Project (NULL = workspace-wide) |
| `url` | TEXT | NOT NULL | Webhook URL |
| `is_active` | BOOLEAN | DEFAULT TRUE | Active status |
| `secret_key` | VARCHAR(255) | | HMAC secret |
| `events` | TEXT ARRAY | DEFAULT '[]' | Subscribed events |

**Indexes**:
- INDEX on (`workspace`, `is_active`)
- INDEX on `project`

### `recent_visits` Table

Tracks recently viewed entities.

**Table Name**: `user_recent_visits`

**Columns**:

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | UUID | PK | Visit ID |
| `workspace` | UUID | FK → workspaces | Workspace |
| `project` | UUID | FK → projects, nullable | Project |
| `user` | UUID | FK → users | User |
| `entity_type` | VARCHAR(50) | | Entity type |
| `entity_identifier` | UUID | | Entity ID |
| `view_props` | JSONB | DEFAULT '{}' | View properties |

**Indexes**:
- INDEX on (`user`, `workspace`, `updated_at DESC`)

---

## Indexes & Performance

### Primary Indexes

Every table has:
- PRIMARY KEY on `id` (UUID)
- INDEX on `created_at`
- INDEX on `updated_at`

### Foreign Key Indexes

All foreign keys have indexes:
- `workspace_id`
- `project_id`
- `user_id`
- `created_by`
- `updated_by`

### Composite Indexes

Performance-critical composite indexes:

```sql
-- Issues by project and state
CREATE INDEX idx_issues_project_state ON issues(project_id, state_id, sort_order);

-- Issues by project and assignee
CREATE INDEX idx_issues_project_assignee ON issue_assignees(project_id, assignee_id);

-- Issues by project and label
CREATE INDEX idx_issues_project_label ON issue_labels(project_id, label_id);

-- Activities by issue
CREATE INDEX idx_activities_issue_created ON issue_activities(issue_id, created_at DESC);

-- Comments by issue
CREATE INDEX idx_comments_issue_created ON issue_comments(issue_id, created_at DESC);
```

### Full-Text Search Indexes

```sql
-- Issue search
CREATE INDEX idx_issues_search ON issues
USING gin(to_tsvector('english', name || ' ' || COALESCE(description_stripped, '')));

-- Project search
CREATE INDEX idx_projects_search ON projects
USING gin(to_tsvector('english', name || ' ' || COALESCE(description, '')));
```

### Soft Delete Filtering

Most queries filter out soft-deleted records:

```sql
-- Typical query pattern
SELECT * FROM issues
WHERE project_id = ?
  AND deleted_at IS NULL
ORDER BY created_at DESC;
```

Indexes include `deleted_at` for efficient filtering:

```sql
CREATE INDEX idx_issues_project_not_deleted
ON issues(project_id)
WHERE deleted_at IS NULL;
```

---

## Migrations

### Migration Strategy

Plane uses Django migrations for schema changes.

**Migration Files**: `apps/api/plane/db/migrations/`

**Apply Migrations**:

```bash
# Check migration status
python manage.py showmigrations

# Run migrations
python manage.py migrate

# Create new migration
python manage.py makemigrations
```

### Critical Migrations

**0001_initial**: Creates all base tables

**0050_add_uuid_primary_keys**: Migrated from integer to UUID primary keys

**0100_add_soft_deletion**: Added `deleted_at` to all tables

**0150_add_audit_fields**: Added `created_by`, `updated_by`

### Rollback Strategy

**Never rollback in production!** Instead:
1. Create a new migration to fix issues
2. Use data migrations for data fixes
3. Keep historical migrations for audit trail

### Zero-Downtime Migrations

For large tables, use:
- `CONCURRENTLY` for index creation
- Backfill in batches
- Blue-green deployments for breaking changes

**Example**:

```python
# apps/api/plane/db/migrations/0200_add_index_concurrently.py

from django.db import migrations

class Migration(migrations.Migration):
    atomic = False  # Required for CONCURRENTLY

    operations = [
        migrations.RunSQL(
            "CREATE INDEX CONCURRENTLY idx_issues_target_date ON issues(target_date);",
            reverse_sql="DROP INDEX CONCURRENTLY idx_issues_target_date;"
        )
    ]
```

---

## React Analogy: Data Flow

**Backend (Django Models):**
```python
# Define schema
class Issue(models.Model):
    name = models.CharField(max_length=255)
    state = models.ForeignKey(State, on_delete=models.CASCADE)
```

**Frontend (TypeScript Types):**
```typescript
// Mirror schema in TypeScript
interface Issue {
  id: string;
  name: string;
  state: State;
}

// Fetch from API
const issues = await api.get<Issue[]>('/issues/');
```

**Key Insight**: Backend defines the source of truth (database schema), frontend mirrors it with TypeScript types. Changes to schema require updating both!

---

## Status: ✅ CURRENT (November 2025)

This schema documentation reflects the current database structure as of:

- ✅ PostgreSQL 14+
- ✅ Django 4.2
- ✅ UUID primary keys
- ✅ Soft deletion pattern
- ✅ Audit fields on all tables

---

## Further Reading

### Related Documentation
- [API_DOCUMENTATION.md](./API_DOCUMENTATION.md) - API endpoints for these models
- [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md) - Django ORM patterns
- [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md) - Database design principles

### External Resources
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Django Models Documentation](https://docs.djangoproject.com/en/4.2/topics/db/models/)
- [Database Design Best Practices](https://www.postgresql.org/docs/current/ddl.html)

---

**Happy querying!** If you need to understand relationships between tables, check the ERD diagram at the top of this document.
