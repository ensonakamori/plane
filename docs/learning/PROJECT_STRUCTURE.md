# Project Structure - Navigating the Plane Codebase

**Master the Monorepo!** This guide provides a comprehensive map of the Plane codebase, showing you where everything lives and how to find what you need.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 40-50 minutes
**📊 Difficulty Level:** Beginner to Intermediate
**🎯 Target Audience:** Developers joining the Plane project

---

## Table of Contents

1. [Quick Overview](#quick-overview)
2. [Root-Level Structure](#root-level-structure)
3. [Applications (`/apps/`)](#applications-apps)
4. [Shared Packages (`/packages/`)](#shared-packages-packages)
5. [Web App Structure](#web-app-structure)
6. [Admin App Structure](#admin-app-structure)
7. [Space App Structure](#space-app-structure)
8. [Live App Structure](#live-app-structure)
9. [API App Structure](#api-app-structure)
10. [Configuration Files](#configuration-files)
11. [Naming Conventions](#naming-conventions)
12. [Finding What You Need](#finding-what-you-need)
13. [File Organization Patterns](#file-organization-patterns)
14. [Next Steps](#next-steps)

---

## Quick Overview

### 🧠 Mental Model: Library Organization

Think of the Plane monorepo as a library:

- **`/apps/`** = Main reading rooms (different sections for different users)
- **`/packages/`** = Reference section (shared by all reading rooms)
- **`/docs/`** = Help desk (documentation and guides)
- **Root config files** = Library rules and policies

---

### Visual Structure

```
/home/user/plane/
├── apps/                    # 🏢 Applications (the products)
│   ├── web/                # Main user interface
│   ├── admin/              # Administration portal
│   ├── space/              # Public interface
│   ├── live/               # Real-time collaboration
│   ├── api/                # Backend API
│   └── proxy/              # Nginx proxy config
│
├── packages/               # 📦 Shared packages (reusable code)
│   ├── types/              # TypeScript type definitions
│   ├── ui/                 # Shared UI components
│   ├── hooks/              # Custom React hooks
│   ├── services/           # API service layer
│   ├── shared-state/       # MobX stores
│   ├── constants/          # Shared constants
│   ├── utils/              # Utility functions
│   ├── editor/             # Rich text editor
│   ├── i18n/               # Internationalization
│   └── ... (more packages)
│
├── docs/                   # 📚 Documentation
│   ├── learning/           # Learning guides (you are here!)
│   └── api/                # API documentation
│
├── docker/                 # 🐳 Docker configurations
├── .github/                # GitHub workflows and templates
│
├── package.json            # Root package config
├── pnpm-workspace.yaml     # pnpm workspace config
├── turbo.json              # Turborepo build config
├── tsconfig.json           # TypeScript config
└── .eslintrc.js            # ESLint config
```

---

## Root-Level Structure

### Essential Files

#### **`package.json`** - Root Package Configuration
```json
{
  "name": "plane",
  "private": true,
  "workspaces": ["apps/*", "packages/*"],
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "test": "turbo test"
  }
}
```

**What it does:**
- Defines workspace structure
- Provides top-level scripts
- Manages dev dependencies

**📍 Location:** `/home/user/plane/package.json`

---

#### **`pnpm-workspace.yaml`** - Workspace Configuration
```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

**What it does:**
- Tells pnpm which directories are packages
- Enables workspace: protocol for internal dependencies

**📍 Location:** `/home/user/plane/pnpm-workspace.yaml`

**🌉 Coming from npm?**
```bash
# npm (old)
npm install lodash

# pnpm workspaces (new)
pnpm add lodash --filter web
pnpm add @plane/types --filter admin  # Internal dependency
```

---

#### **`turbo.json`** - Build Orchestration
```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

**What it does:**
- Orchestrates builds across packages
- Caches build outputs
- Manages task dependencies

**📍 Location:** `/home/user/plane/turbo.json`

**💡 Aha Moment:**
> Turborepo is like `make` for modern JavaScript - it knows what depends on what and only rebuilds what changed!

---

## Applications (`/apps/`)

### Overview

```
/apps/
├── web/        # Main application (port 3000)
├── admin/      # Admin portal (port 3001)
├── space/      # Public interface (port 3002)
├── live/       # Real-time features (port 3003)
├── api/        # Django backend (port 8000)
└── proxy/      # Nginx configuration
```

---

### Common Patterns Across Frontend Apps

All frontend apps (`web/`, `admin/`, `space/`, `live/`) share this structure:

```
<app>/
├── app/                    # React Router 7 routes
│   ├── root.tsx           # Root component
│   ├── routes.ts          # Route configuration
│   ├── routes/            # Route components
│   ├── entry.client.tsx   # Client entry point
│   └── provider.tsx       # App providers
│
├── core/                   # Core application code
│   ├── components/        # React components
│   ├── hooks/             # Custom hooks (app-specific)
│   ├── store/             # MobX stores (app-specific)
│   ├── services/          # API services (app-specific)
│   ├── layouts/           # Layout components
│   ├── lib/               # Utilities
│   └── constants/         # Constants (app-specific)
│
├── ee/                     # Enterprise Edition features
│   ├── components/        # EE-only components
│   ├── hooks/             # EE-only hooks
│   └── store/             # EE-only stores
│
├── public/                 # Static assets
│   ├── favicon/           # Favicons
│   ├── icons/             # Icon files
│   └── images/            # Images
│
├── helpers/                # Helper functions
├── nginx/                  # Nginx config for this app
├── package.json           # App dependencies
├── tsconfig.json          # TypeScript config
└── react-router.config.ts # React Router config
```

**🎯 Remember This:**
> `core/` = Community Edition features (open source)
> `ee/` = Enterprise Edition features (paid)

---

## Web App Structure

**Primary Application** - Main project management interface

**📍 Location:** `/home/user/plane/apps/web/`
**Port:** 3000
**Purpose:** Complete project management experience

### Key Directories

#### **`/app/`** - React Router 7 Application
```
app/
├── root.tsx                 # Root component (wraps everything)
├── routes.ts                # Route definitions
├── layout.tsx               # Main layout
├── provider.tsx             # Providers (MobX, theme, auth)
│
├── routes/                  # File-based routes
│   ├── (all)/               # Public routes
│   │   └── invitations.$invitation.tsx
│   ├── (home)/              # Home section
│   │   ├── layout.tsx
│   │   └── route.tsx
│   └── compat/              # Compatibility routes
│
├── assets/                  # Static assets
├── error/                   # Error boundaries
└── types/                   # App-specific types
```

**Route Example:**

```typescript
// /apps/web/app/routes/(home)/route.tsx
import type { LoaderFunctionArgs } from "react-router";

export async function loader({ request }: LoaderFunctionArgs) {
  // Load data on server
  const user = await getUser(request);
  return json({ user });
}

export default function HomePage() {
  const { user } = useLoaderData<typeof loader>();
  return <Dashboard user={user} />;
}
```

---

#### **`/core/components/`** - React Components
```
core/components/
├── issues/                  # Issue-related components
│   ├── issue-layouts/       # List, board, calendar views
│   ├── issue-detail/        # Issue detail view
│   ├── issue-modal/         # Create/edit modals
│   ├── peek-overview/       # Quick preview
│   └── filters.tsx          # Filter UI
│
├── project/                 # Project components
│   ├── create-project-modal.tsx
│   ├── project-card.tsx
│   └── settings/            # Project settings
│
├── workspace/               # Workspace components
│   ├── sidebar/
│   ├── settings/
│   └── members/
│
├── ui/                      # Generic UI components
│   ├── button.tsx
│   ├── modal.tsx
│   └── input.tsx
│
└── common/                  # Common components
    ├── loader.tsx
    ├── empty-state.tsx
    └── error-boundary.tsx
```

**Component Organization Pattern:**

```typescript
// Feature-based organization ✅ CURRENT
components/
  issues/
    issue-card.tsx           # Small, focused component
    issue-list.tsx           # Composition of issue-cards
    issue-board.tsx          # Different view of issues
    filters.tsx              # Related functionality

// NOT like this ❌ OUTDATED
components/
  IssueCard.tsx
  IssueList.tsx
  IssueBoard.tsx
  IssueFilters.tsx
  ProjectCard.tsx
  ProjectList.tsx
  ...all mixed together
```

**💡 Aha Moment:**
> Components are organized by feature (issues, projects) not by type (cards, lists). This makes related code easier to find!

---

#### **`/core/store/`** - MobX State Management
```
core/store/
├── root.store.ts            # Root store (composes all stores)
├── user.store.ts            # User state
├── workspace.store.ts       # Workspace state
├── project/                 # Project stores
│   ├── project.store.ts
│   └── project-member.store.ts
├── issue/                   # Issue stores
│   ├── issue.store.ts
│   ├── issue-filter.store.ts
│   └── issue-kanban.store.ts
└── cycle/
    ├── cycle.store.ts
    └── active-cycle.store.ts
```

**Store Example:**

```typescript
// /apps/web/core/store/issue/issue.store.ts
import { makeObservable, observable, action, runInAction } from "mobx";
import { IssueService } from "@plane/services";
import type { TIssue } from "@plane/types";

export class IssueStore {
  // Observable state
  issues = new Map<string, TIssue>();
  isLoading = false;

  constructor() {
    makeObservable(this, {
      issues: observable,
      isLoading: observable,
      fetchIssues: action,
      addIssue: action,
    });
  }

  // Actions
  fetchIssues = async (projectId: string) => {
    this.isLoading = true;
    try {
      const response = await IssueService.getIssues(projectId);
      runInAction(() => {
        response.forEach(issue => this.issues.set(issue.id, issue));
        this.isLoading = false;
      });
    } catch (error) {
      runInAction(() => {
        this.isLoading = false;
      });
    }
  };

  addIssue = (issue: TIssue) => {
    this.issues.set(issue.id, issue);
  };
}
```

**🌉 React to MobX Bridge:**

| React Pattern | MobX Pattern | File Location |
|---------------|--------------|---------------|
| `useState` | `observable` | `core/store/*.store.ts` |
| `useContext` | Root store injection | `core/store/root.store.ts` |
| Custom hooks | Store methods | `core/hooks/use-issue.ts` |

---

#### **`/core/hooks/`** - Custom React Hooks
```
core/hooks/
├── use-local-storage.tsx    # localStorage wrapper
├── use-debounce.tsx         # Debounce hook
├── use-timer.tsx            # Timer utilities
├── use-platform-os.tsx      # OS detection
│
├── use-issues-actions.tsx   # Issue actions
├── use-workspace-invitation.tsx
│
└── context/                 # Context-based hooks
    ├── use-issue-modal.tsx
    └── app-rail-context.tsx
```

**Hook Example:**

```typescript
// /apps/web/core/hooks/use-issues-actions.tsx
import { useCallback } from "react";
import { useIssueStore } from "./store";
import { IssueService } from "@plane/services";

export const useIssueActions = () => {
  const store = useIssueStore();

  const createIssue = useCallback(async (data: Partial<TIssue>) => {
    // Optimistic update
    const tempIssue = { id: "temp", ...data };
    store.addIssue(tempIssue);

    try {
      // API call
      const issue = await IssueService.create(data);
      // Update with real issue
      store.updateIssue("temp", issue);
    } catch (error) {
      // Rollback on error
      store.removeIssue("temp");
      throw error;
    }
  }, [store]);

  return { createIssue };
};
```

---

#### **`/core/services/`** - API Service Layer
```
core/services/
├── issue/
│   ├── issue.service.ts     # Issue CRUD operations
│   ├── issue-filter.service.ts
│   └── issue-comment.service.ts
│
├── project/
│   ├── project.service.ts
│   └── project-member.service.ts
│
└── workspace/
    ├── workspace.service.ts
    └── workspace-member.service.ts
```

**Service Example:**

```typescript
// /apps/web/core/services/issue/issue.service.ts
import { APIService } from "@plane/services";
import type { TIssue } from "@plane/types";

export class IssueService extends APIService {
  constructor() {
    super("/api/v1/workspaces");
  }

  async getIssues(workspaceSlug: string, projectId: string): Promise<TIssue[]> {
    return this.get(`/${workspaceSlug}/projects/${projectId}/issues/`);
  }

  async create(
    workspaceSlug: string,
    projectId: string,
    data: Partial<TIssue>
  ): Promise<TIssue> {
    return this.post(`/${workspaceSlug}/projects/${projectId}/issues/`, data);
  }

  async update(
    workspaceSlug: string,
    projectId: string,
    issueId: string,
    data: Partial<TIssue>
  ): Promise<TIssue> {
    return this.patch(
      `/${workspaceSlug}/projects/${projectId}/issues/${issueId}/`,
      data
    );
  }
}
```

**🎯 Remember This:**
> Services are the ONLY place that should make API calls. Components use stores, stores use services, services use APIs.

**Data Flow:**
```
Component → Hook → Store → Service → API → Backend
```

---

#### **`/core/layouts/`** - Layout Components
```
core/layouts/
├── default-layout/          # Default app layout
│   ├── index.tsx
│   ├── sidebar.tsx
│   └── header.tsx
│
├── auth-layout/             # Authenticated layout
│   ├── workspace-wrapper.tsx
│   └── project-wrapper.tsx
│
└── settings-layout/         # Settings layout
    └── index.tsx
```

**Layout Pattern:**

```typescript
// /apps/web/core/layouts/default-layout/index.tsx
export const DefaultLayout = ({ children }: { children: React.ReactNode }) => {
  return (
    <div className="flex h-screen">
      <Sidebar />
      <div className="flex-1 flex flex-col">
        <Header />
        <main className="flex-1 overflow-auto">
          {children}
        </main>
      </div>
    </div>
  );
};
```

---

## Admin App Structure

**Administration Portal** - Instance and user management

**📍 Location:** `/home/user/plane/apps/admin/`
**Port:** 3001
**Purpose:** System administration and configuration

### Structure (Similar to Web)

```
admin/
├── app/                     # Routes (like web)
├── core/                    # Admin-specific components
│   ├── components/
│   │   ├── users/           # User management
│   │   ├── instance/        # Instance settings
│   │   └── licensing/       # License management
│   ├── hooks/
│   └── store/
├── public/
└── package.json
```

**Key Differences from Web:**
- Focus on admin UI (tables, forms, dashboards)
- Different permissions (admin-only)
- Instance-level operations

---

## Space App Structure

**Public Interface** - No authentication required

**📍 Location:** `/home/user/plane/apps/space/`
**Port:** 3002
**Purpose:** Public project boards and issue views

### Structure

```
space/
├── app/                     # Routes
├── core/
│   ├── components/
│   │   ├── issues/          # Public issue views (read-only)
│   │   └── board/           # Public boards
│   └── services/            # Public API endpoints
└── public/
```

**Key Differences:**
- No authentication required
- Read-only views
- Simplified UI
- Public API endpoints only

---

## Live App Structure

**Real-time Collaboration** - WebSocket-based features

**📍 Location:** `/home/user/plane/apps/live/`
**Port:** 3003
**Purpose:** Real-time collaboration features

### Structure

```
live/
├── app/
├── core/
│   ├── components/
│   │   ├── cursors/         # Live cursors
│   │   ├── presence/        # User presence
│   │   └── collaborative-editing/
│   └── lib/
│       └── websocket/       # WebSocket client
└── package.json
```

**Key Features:**
- WebSocket connections
- Real-time state sync
- Presence indicators
- Collaborative editing

---

## API App Structure

**Django Backend** - Business logic and data management

**📍 Location:** `/home/user/plane/apps/api/`
**Port:** 8000 (development)
**Tech:** Django 4.2 + Python 3.12

### Structure

```
api/
├── plane/                   # Main Django app
│   ├── db/                  # Database layer
│   │   ├── models/          # Django models (database schema)
│   │   └── migrations/      # Database migrations
│   │
│   ├── app/                 # Application layer
│   │   ├── views/           # API endpoints (ViewSets)
│   │   ├── serializers/     # Data serialization
│   │   ├── permissions/     # Authorization logic
│   │   └── urls.py          # URL routing
│   │
│   ├── authentication/      # Auth layer
│   │   ├── middleware/      # Auth middleware
│   │   ├── views/           # Login, logout, signup
│   │   └── adapter/         # SSO adapters
│   │
│   ├── bgtasks/             # Background tasks (Celery)
│   │   ├── notification.py  # Notification tasks
│   │   └── export.py        # Export tasks
│   │
│   ├── middleware/          # Custom middleware
│   │   ├── request_body_size.py
│   │   └── logger.py
│   │
│   ├── settings/            # Django settings
│   │   ├── common.py        # Common settings
│   │   ├── local.py         # Local dev settings
│   │   └── production.py    # Production settings
│   │
│   ├── utils/               # Utility functions
│   ├── analytics/           # Analytics features
│   ├── space/               # Space app endpoints
│   ├── web/                 # Web app endpoints
│   ├── license/             # Licensing logic
│   │
│   ├── urls.py              # Root URL configuration
│   ├── wsgi.py              # WSGI entry point
│   └── celery.py            # Celery configuration
│
├── requirements.txt         # Python dependencies
├── requirements.dev.txt     # Dev dependencies
├── manage.py                # Django management script
├── Dockerfile               # Docker image
└── .env.example             # Environment variables template
```

---

### Django Models (`/plane/db/models/`)

**🧠 Mental Model:**
Models define your database schema in Python code.

```
db/models/
├── __init__.py              # Exports all models
├── base.py                  # Base model (timestamps, etc.)
├── user.py                  # User model
├── workspace.py             # Workspace model
├── project.py               # Project model
├── issue.py                 # Issue model ⭐ Most important
├── state.py                 # Issue states
├── cycle.py                 # Cycles (sprints)
├── module.py                # Modules
├── page.py                  # Pages (wiki)
├── notification.py          # Notifications
├── view.py                  # Saved views
├── webhook.py               # Webhooks
└── ...
```

**Model Example:**

```python
# /apps/api/plane/db/models/issue.py
from django.db import models
from plane.db.models import ProjectBaseModel

class Issue(ProjectBaseModel):
    """Core issue model"""
    project = models.ForeignKey("Project", on_delete=models.CASCADE)
    state = models.ForeignKey("State", on_delete=models.CASCADE)
    parent = models.ForeignKey("self", on_delete=models.CASCADE, null=True)

    name = models.CharField(max_length=255)
    description = models.JSONField(default=dict, blank=True)
    description_html = models.TextField(blank=True)

    priority = models.CharField(max_length=30, choices=PRIORITY_CHOICES)
    start_date = models.DateField(null=True, blank=True)
    target_date = models.DateField(null=True, blank=True)

    assignees = models.ManyToManyField("User", related_name="assigned_issues")
    labels = models.ManyToManyField("Label", related_name="issues")

    estimate_point = models.IntegerField(null=True, blank=True)

    class Meta:
        db_table = "plane_issue"
        verbose_name = "Issue"
        verbose_name_plural = "Issues"
        ordering = ["-created_at"]

    def __str__(self):
        return f"{self.project.identifier}-{self.sequence_id}: {self.name}"
```

**🌉 TypeScript to Django Bridge:**

```typescript
// Frontend type (/packages/types/src/issue.d.ts)
export type TIssue = {
  id: string;
  project: string;
  name: string;
  description: any;
  priority: "urgent" | "high" | "medium" | "low" | "none";
  state: string;
  assignees: string[];
  created_at: string;
};
```

```python
# Backend model (/apps/api/plane/db/models/issue.py)
class Issue(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid4)
    project = models.ForeignKey(Project, on_delete=models.CASCADE)
    name = models.CharField(max_length=255)
    description = models.JSONField(default=dict)
    priority = models.CharField(max_length=30)
    state = models.ForeignKey(State, on_delete=models.CASCADE)
    assignees = models.ManyToManyField(User)
    created_at = models.DateTimeField(auto_now_add=True)
```

---

### Django Views (`/plane/app/views/`)

**API Endpoints** - Handle HTTP requests

```
app/views/
├── __init__.py
├── base.py                  # Base views
├── issue/
│   ├── issue.py             # IssueViewSet (CRUD)
│   ├── issue_comment.py     # Comment endpoints
│   ├── issue_attachment.py  # Attachment endpoints
│   └── issue_activity.py    # Activity log
├── project/
│   ├── project.py           # Project CRUD
│   └── project_member.py    # Member management
├── workspace/
│   ├── workspace.py
│   └── workspace_member.py
└── ...
```

**ViewSet Example:**

```python
# /apps/api/plane/app/views/issue/issue.py
from rest_framework import status
from rest_framework.response import Response
from plane.app.views import BaseViewSet
from plane.app.serializers import IssueSerializer
from plane.app.permissions import ProjectPermission
from plane.db.models import Issue

class IssueViewSet(BaseViewSet):
    """Issue CRUD operations"""
    permission_classes = [ProjectPermission]
    serializer_class = IssueSerializer

    def get_queryset(self):
        return Issue.objects.filter(
            workspace__slug=self.kwargs["workspace_slug"],
            project_id=self.kwargs["project_id"]
        )

    def list(self, request, workspace_slug, project_id):
        """GET /api/workspaces/:slug/projects/:id/issues/"""
        issues = self.get_queryset()
        serializer = IssueSerializer(issues, many=True)
        return Response(serializer.data)

    def create(self, request, workspace_slug, project_id):
        """POST /api/workspaces/:slug/projects/:id/issues/"""
        serializer = IssueSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(
                project_id=project_id,
                created_by=request.user
            )
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

---

### Django Serializers (`/plane/app/serializers/`)

**Data Transformation** - Convert between DB and JSON

```
app/serializers/
├── __init__.py
├── base.py                  # Base serializers
├── issue.py                 # Issue serializers
├── project.py               # Project serializers
├── workspace.py             # Workspace serializers
└── user.py                  # User serializers
```

**Serializer Example:**

```python
# /apps/api/plane/app/serializers/issue.py
from rest_framework import serializers
from plane.db.models import Issue

class IssueSerializer(serializers.ModelSerializer):
    # Related fields
    state_detail = StateSerializer(source='state', read_only=True)
    project_detail = ProjectLiteSerializer(source='project', read_only=True)
    assignee_details = UserLiteSerializer(source='assignees', many=True, read_only=True)

    # Computed fields
    sub_issues_count = serializers.IntegerField(read_only=True)

    class Meta:
        model = Issue
        fields = [
            'id',
            'name',
            'description',
            'description_html',
            'priority',
            'state',
            'state_detail',
            'project',
            'project_detail',
            'assignees',
            'assignee_details',
            'start_date',
            'target_date',
            'estimate_point',
            'sub_issues_count',
            'created_at',
            'updated_at',
        ]
        read_only_fields = [
            'id',
            'created_at',
            'updated_at',
            'created_by',
        ]
```

**🌉 Serializer to TypeScript:**

```python
# Backend serializer output
{
  "id": "123",
  "name": "Fix bug",
  "priority": "high",
  "state_detail": {
    "id": "456",
    "name": "In Progress",
    "color": "#3b82f6"
  },
  "assignee_details": [
    {
      "id": "789",
      "display_name": "John Doe",
      "avatar": "..."
    }
  ]
}
```

```typescript
// Frontend type
type TIssue = {
  id: string;
  name: string;
  priority: string;
  state_detail: {
    id: string;
    name: string;
    color: string;
  };
  assignee_details: {
    id: string;
    display_name: string;
    avatar: string;
  }[];
};
```

---

### Background Tasks (`/plane/bgtasks/`)

**Celery Tasks** - Async operations

```
bgtasks/
├── __init__.py
├── notification.py          # Send notifications
├── export.py                # Export issues/projects
├── issue_automation.py      # Automated issue actions
└── webhook.py               # Webhook delivery
```

**Task Example:**

```python
# /apps/api/plane/bgtasks/notification.py
from celery import shared_task
from plane.db.models import Issue, User
from plane.utils.email import send_email

@shared_task
def send_issue_notification(issue_id: str, user_ids: list[str]):
    """Send email notification about issue update"""
    try:
        issue = Issue.objects.get(id=issue_id)
        users = User.objects.filter(id__in=user_ids)

        for user in users:
            send_email(
                to=user.email,
                subject=f"Issue updated: {issue.name}",
                template="issue_updated",
                context={"issue": issue, "user": user}
            )
    except Exception as e:
        # Log error but don't crash
        logger.error(f"Failed to send notification: {e}")
```

**Usage:**

```python
# In a view
def update_issue(request, issue_id):
    issue = Issue.objects.get(id=issue_id)
    issue.name = request.data["name"]
    issue.save()

    # Send notifications in background (doesn't block response)
    send_issue_notification.delay(
        issue_id=issue.id,
        user_ids=[a.id for a in issue.assignees.all()]
    )

    return Response(IssueSerializer(issue).data)
```

---

## Shared Packages (`/packages/`)

**Reusable Code** - Shared across all apps

### Package Overview

```
packages/
├── types/              # TypeScript types
├── ui/                 # UI component library
├── hooks/              # React hooks
├── services/           # API services
├── shared-state/       # MobX stores
├── constants/          # Shared constants
├── utils/              # Utility functions
├── editor/             # Rich text editor
├── i18n/               # Translations
├── decorators/         # TypeScript decorators
├── logger/             # Logging utilities
├── propel/             # Propel integration
├── eslint-config/      # Shared ESLint config
├── typescript-config/  # Shared TS config
└── tailwind-config/    # Shared Tailwind config
```

---

### **`@plane/types`** - TypeScript Definitions

```
types/src/
├── index.d.ts           # Exports all types
├── auth.d.ts            # Auth types
├── issue.d.ts           # Issue types ⭐ Most used
├── project.d.ts         # Project types
├── workspace.d.ts       # Workspace types
├── user.d.ts            # User types
├── state.d.ts           # State types
├── cycle.d.ts           # Cycle types
└── ...
```

**Type Example:**

```typescript
// /packages/types/src/issue.d.ts
export type TIssue = {
  id: string;
  sequence_id: number;
  name: string;
  description_html: string;
  priority: TIssuePriority;
  state: string;
  state_detail: TState;
  project: string;
  project_detail: TProject;
  workspace: string;
  parent: string | null;
  assignees: string[];
  assignee_details: TUser[];
  labels: string[];
  label_details: TLabel[];
  start_date: string | null;
  target_date: string | null;
  estimate_point: number | null;
  created_at: string;
  updated_at: string;
  created_by: string;
};

export type TIssuePriority = "urgent" | "high" | "medium" | "low" | "none";
```

**Usage in Apps:**

```typescript
// Any app can import
import type { TIssue, TIssuePriority } from "@plane/types";

const issue: TIssue = {
  id: "123",
  name: "Fix bug",
  priority: "high", // TypeScript validates this!
  // ...
};
```

---

### **`@plane/ui`** - Component Library

```
ui/src/
├── components/
│   ├── button/
│   │   ├── button.tsx
│   │   └── button.stories.tsx  # Storybook story
│   ├── input/
│   ├── modal/
│   ├── dropdown/
│   ├── avatar/
│   ├── tooltip/
│   └── ...
│
├── icons/               # Icon components
│   ├── lucide.tsx
│   └── custom-icons.tsx
│
└── index.ts             # Exports
```

**Component Example:**

```typescript
// /packages/ui/src/components/button/button.tsx
import * as React from "react";
import { cva, type VariantProps } from "class-variance-authority";

const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md font-medium transition-colors",
  {
    variants: {
      variant: {
        primary: "bg-blue-600 text-white hover:bg-blue-700",
        secondary: "bg-gray-200 text-gray-900 hover:bg-gray-300",
        outline: "border border-gray-300 hover:bg-gray-100",
      },
      size: {
        sm: "h-8 px-3 text-sm",
        md: "h-10 px-4 text-base",
        lg: "h-12 px-6 text-lg",
      },
    },
    defaultVariants: {
      variant: "primary",
      size: "md",
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={buttonVariants({ variant, size, className })}
        {...props}
      />
    );
  }
);
```

**Usage:**

```typescript
// In any app
import { Button } from "@plane/ui";

<Button variant="primary" size="lg">
  Click me
</Button>
```

---

### **`@plane/hooks`** - Shared React Hooks

```
hooks/src/
├── use-debounce.ts
├── use-local-storage.ts
├── use-outside-click.ts
├── use-copy-to-clipboard.ts
├── use-intersection-observer.ts
└── index.ts
```

**Hook Example:**

```typescript
// /packages/hooks/src/use-debounce.ts
import { useState, useEffect } from "react";

export function useDebounce<T>(value: T, delay: number = 500): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
}
```

---

### **`@plane/services`** - API Service Layer

```
services/src/
├── api.service.ts       # Base API service
├── auth.service.ts      # Authentication
├── issue.service.ts     # Issue operations
├── project.service.ts   # Project operations
├── workspace.service.ts # Workspace operations
└── index.ts
```

**Service Example:**

```typescript
// /packages/services/src/api.service.ts
import axios, { AxiosInstance } from "axios";

export abstract class APIService {
  protected baseURL: string;
  protected instance: AxiosInstance;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
    this.instance = axios.create({
      baseURL,
      withCredentials: true, // Include cookies
      headers: {
        "Content-Type": "application/json",
      },
    });
  }

  protected async get<T>(url: string): Promise<T> {
    const response = await this.instance.get(url);
    return response.data;
  }

  protected async post<T>(url: string, data: any): Promise<T> {
    const response = await this.instance.post(url, data);
    return response.data;
  }

  protected async patch<T>(url: string, data: any): Promise<T> {
    const response = await this.instance.patch(url, data);
    return response.data;
  }

  protected async delete<T>(url: string): Promise<T> {
    const response = await this.instance.delete(url);
    return response.data;
  }
}
```

---

### **`@plane/shared-state`** - Shared MobX Stores

```
shared-state/src/
├── store/
│   ├── user.store.ts        # User state (used across apps)
│   ├── workspace.store.ts   # Workspace state
│   ├── rich-filters/        # Filter state
│   └── work-item-filters/   # Work item filters
└── index.ts
```

**Shared Store Example:**

```typescript
// /packages/shared-state/src/store/user.store.ts
import { makeObservable, observable, action, computed } from "mobx";
import type { TUser } from "@plane/types";

export class UserStore {
  currentUser: TUser | null = null;
  isLoading = false;

  constructor() {
    makeObservable(this, {
      currentUser: observable,
      isLoading: observable,
      setUser: action,
      isAuthenticated: computed,
    });
  }

  setUser = (user: TUser | null) => {
    this.currentUser = user;
  };

  get isAuthenticated() {
    return this.currentUser !== null;
  }
}
```

---

### **`@plane/constants`** - Shared Constants

```
constants/src/
├── issues.ts            # Issue-related constants
├── projects.ts          # Project constants
├── workspace.ts         # Workspace constants
├── colors.ts            # Color palette
├── state.ts             # State constants
└── index.ts
```

**Constants Example:**

```typescript
// /packages/constants/src/issues.ts
export const ISSUE_PRIORITIES = [
  { key: "urgent", label: "Urgent", color: "#ef4444" },
  { key: "high", label: "High", color: "#f97316" },
  { key: "medium", label: "Medium", color: "#eab308" },
  { key: "low", label: "Low", color: "#22c55e" },
  { key: "none", label: "None", color: "#64748b" },
] as const;

export const ISSUE_STATE_GROUPS = [
  { key: "backlog", label: "Backlog" },
  { key: "unstarted", label: "Unstarted" },
  { key: "started", label: "Started" },
  { key: "completed", label: "Completed" },
  { key: "cancelled", label: "Cancelled" },
] as const;
```

---

### **`@plane/utils`** - Utility Functions

```
utils/src/
├── date.ts              # Date utilities
├── string.ts            # String utilities
├── validation.ts        # Validation helpers
├── format.ts            # Formatting functions
└── index.ts
```

**Utility Example:**

```typescript
// /packages/utils/src/date.ts
import { format, formatDistanceToNow } from "date-fns";

export const formatDate = (date: string | Date): string => {
  return format(new Date(date), "MMM dd, yyyy");
};

export const formatDateTime = (date: string | Date): string => {
  return format(new Date(date), "MMM dd, yyyy HH:mm");
};

export const formatRelativeTime = (date: string | Date): string => {
  return formatDistanceToNow(new Date(date), { addSuffix: true });
};
```

---

## Configuration Files

### TypeScript Configuration

**Root Config** - `/tsconfig.json`
```json
{
  "extends": "@plane/typescript-config/base.json",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@plane/*": ["packages/*/src"]
    }
  }
}
```

**App Config** - `/apps/web/tsconfig.json`
```json
{
  "extends": "@plane/typescript-config/react.json",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "~/*": ["./app/*"],
      "@plane/*": ["../../packages/*/src"]
    }
  }
}
```

---

### ESLint Configuration

**Shared Config** - `/packages/eslint-config/package.json`
```javascript
module.exports = {
  extends: [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
  ],
  rules: {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn",
    "react/prop-types": "off",
  },
};
```

**App Config** - `/apps/web/.eslintrc.js`
```javascript
module.exports = {
  extends: ["@plane/eslint-config"],
  // App-specific overrides
};
```

---

## Naming Conventions

### File Naming

#### ✅ CURRENT PATTERNS

**Components:** `kebab-case.tsx`
```
issue-card.tsx         # Component
issue-list.tsx         # Component
create-issue-modal.tsx # Component
```

**Stores:** `kebab-case.store.ts`
```
issue.store.ts         # MobX store
project.store.ts       # MobX store
root.store.ts          # Root store
```

**Services:** `kebab-case.service.ts`
```
issue.service.ts       # API service
project.service.ts     # API service
auth.service.ts        # API service
```

**Hooks:** `use-kebab-case.ts`
```
use-issue.ts           # Custom hook
use-debounce.ts        # Custom hook
use-local-storage.ts   # Custom hook
```

**Types:** `kebab-case.d.ts`
```
issue.d.ts             # Type definitions
project.d.ts           # Type definitions
```

**Python Files:** `snake_case.py`
```
issue.py               # Django model
issue_view.py          # Django view
issue_serializer.py    # DRF serializer
```

---

### Variable Naming

#### TypeScript/React

```typescript
// Components: PascalCase
const IssueCard = () => {};
const CreateIssueModal = () => {};

// Variables: camelCase
const issueId = "123";
const currentUser = getCurrentUser();

// Constants: SCREAMING_SNAKE_CASE
const MAX_ISSUE_TITLE_LENGTH = 255;
const API_BASE_URL = "/api/v1";

// Types: PascalCase with T prefix
type TIssue = {};
type TProject = {};
type TUser = {};

// Interfaces: PascalCase with I prefix (less common)
interface IIssueStore {};
```

#### Python/Django

```python
# Classes: PascalCase
class Issue(models.Model):
    pass

class IssueViewSet(BaseViewSet):
    pass

# Functions/methods: snake_case
def get_issue(issue_id):
    pass

def create_issue(data):
    pass

# Variables: snake_case
issue_id = "123"
current_user = get_current_user()

# Constants: SCREAMING_SNAKE_CASE
MAX_ISSUE_TITLE_LENGTH = 255
API_VERSION = "v1"
```

---

### Directory Naming

```
kebab-case/            # ✅ Preferred
  issue-layouts/
  project-settings/
  workspace-members/

camelCase/             # ❌ Avoid
  issueLayouts/
  projectSettings/
```

---

## Finding What You Need

### Common Tasks → File Locations

| Task | Location | File Path |
|------|----------|-----------|
| Add new issue field (backend) | Django model | `/apps/api/plane/db/models/issue.py` |
| Add new issue field (frontend) | TypeScript type | `/packages/types/src/issue.d.ts` |
| Create new API endpoint | Django view | `/apps/api/plane/app/views/issue/` |
| Create new React component | Component folder | `/apps/web/core/components/issues/` |
| Add new route | Route file | `/apps/web/app/routes/` |
| Add shared UI component | UI package | `/packages/ui/src/components/` |
| Add utility function | Utils package | `/packages/utils/src/` |
| Add custom hook | Hooks package | `/packages/hooks/src/` |
| Add constant | Constants package | `/packages/constants/src/` |
| Add background task | Celery tasks | `/apps/api/plane/bgtasks/` |

---

### Search Strategies

#### Find by Feature

```bash
# Find all issue-related files
find . -name "*issue*" -type f

# Find in specific directory
find ./apps/web/core/components -name "*issue*"
```

#### Find by Type

```bash
# Find all React components
find . -name "*.tsx" -not -path "*/node_modules/*"

# Find all Django models
find ./apps/api -name "*.py" -path "*/models/*"

# Find all TypeScript types
find ./packages/types -name "*.d.ts"
```

#### Search Content

```bash
# Find files containing "IssueViewSet"
grep -r "IssueViewSet" apps/api/

# Find TypeScript files using TIssue type
grep -r "TIssue" apps/web/ --include="*.tsx" --include="*.ts"
```

---

## File Organization Patterns

### Component Co-location ✅ CURRENT

**Pattern:** Keep related files together

```
components/issues/
├── issue-card.tsx           # Component
├── issue-card.stories.tsx   # Storybook story
├── issue-card.test.tsx      # Tests
├── issue-list.tsx           # Related component
├── issue-board.tsx          # Related component
└── use-issue-actions.ts     # Related hook
```

**Benefits:**
- Easy to find related files
- Clear feature boundaries
- Easier to refactor/delete features

---

### Service Layer Pattern ✅ CURRENT

**Pattern:** Separate API logic from UI logic

```
Frontend:
  Component → Hook → Store → Service → API

Backend:
  API → View → Serializer → Model → Database
```

**Example:**

```typescript
// Component (UI)
const IssueCard = ({ issueId }: { issueId: string }) => {
  const { issue } = useIssue(issueId);  // Hook
  return <div>{issue.name}</div>;
};

// Hook (Glue code)
const useIssue = (id: string) => {
  const store = useIssueStore();  // Store
  return { issue: store.getIssue(id) };
};

// Store (State management)
class IssueStore {
  async fetchIssue(id: string) {
    const issue = await IssueService.get(id);  // Service
    this.issues.set(id, issue);
  }
}

// Service (API calls)
class IssueService extends APIService {
  async get(id: string): Promise<TIssue> {
    return this.get(`/api/issues/${id}/`);  // API
  }
}
```

---

### Barrel Exports Pattern ✅ CURRENT

**Pattern:** Export multiple items from `index.ts`

```typescript
// /packages/ui/src/components/index.ts
export { Button } from "./button";
export { Input } from "./input";
export { Modal } from "./modal";
export { Dropdown } from "./dropdown";

// Usage: Import from single location
import { Button, Input, Modal } from "@plane/ui";
```

**Benefits:**
- Cleaner imports
- Easy to reorganize internals
- Clear public API

---

## Next Steps

### Continue Learning

Now that you understand the structure, dive deeper:

1. **[TECH_STACK_GUIDE.md](./TECH_STACK_GUIDE.md)** - Understand each technology
2. **[ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)** - See how it all fits together
3. **[PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md)** - Learn coding standards
4. **[HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)** - Start building features

---

### Quick Self-Check

✅ **Test your understanding:**

1. Where would you add a new React component for issues?
2. Where is the Django model for issues located?
3. What's the difference between `/apps/web/core/` and `/apps/web/ee/`?
4. How do you import shared types in a component?
5. Where would you add a background task for sending emails?

**Answers:**
1. `/apps/web/core/components/issues/`
2. `/apps/api/plane/db/models/issue.py`
3. `core/` = Community Edition, `ee/` = Enterprise Edition
4. `import type { TIssue } from "@plane/types";`
5. `/apps/api/plane/bgtasks/notification.py`

---

### Get Hands-On

Ready to navigate the codebase?

1. Clone the repository
2. Open in your IDE
3. Use "Go to Definition" (Cmd/Ctrl + Click) to explore connections
4. Try finding files for a feature you use in the UI

---

**🎉 Congratulations!** You can now navigate the Plane codebase with confidence!

---

**Questions or Feedback?**
- Check **[FAQ.md](./FAQ.md)** for common questions
- Join the Plane community discussions
- Open an issue if you find gaps in this documentation

---

**Last Updated:** November 19, 2025
**Maintainers:** Plane Learning Docs Team
**License:** AGPL-3.0
