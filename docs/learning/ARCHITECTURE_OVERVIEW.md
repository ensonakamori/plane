# Architecture Overview - System Design & Patterns

**Welcome to Plane's Architecture!** This guide provides a comprehensive overview of how Plane is designed, how its components interact, and the architectural patterns that power the system.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 45-60 minutes
**📊 Difficulty Level:** Intermediate
**🎯 Target Audience:** Mid-level React developers learning full-stack architecture

---

## Table of Contents

1. [The Big Picture](#the-big-picture)
2. [Architecture Philosophy](#architecture-philosophy)
3. [System Components](#system-components)
4. [Monorepo Architecture](#monorepo-architecture)
5. [Frontend Architecture](#frontend-architecture)
6. [Backend Architecture](#backend-architecture)
7. [Data Architecture](#data-architecture)
8. [Communication Patterns](#communication-patterns)
9. [State Management Strategy](#state-management-strategy)
10. [Caching Layers](#caching-layers)
11. [Real-Time Features](#real-time-features)
12. [Background Processing](#background-processing)
13. [Scalability Considerations](#scalability-considerations)
14. [Security Architecture](#security-architecture)
15. [Development Architecture](#development-architecture)
16. [Deployment Architecture](#deployment-architecture)
17. [Trade-offs & Design Decisions](#trade-offs--design-decisions)
18. [Next Steps](#next-steps)

---

## The Big Picture

### 🧠 Mental Model: Think of Plane as a City

**If Plane were a city:**

- **Frontend Apps** = Neighborhoods (web, admin, space, live)
- **Backend API** = City Hall (central coordination & decision-making)
- **Database** = City Records (persistent storage of everything)
- **Shared Packages** = Public Utilities (water, power - used everywhere)
- **Redis** = City Bulletin Board (fast, temporary information)
- **Celery** = Department of Public Works (background jobs)
- **WebSockets** = Emergency Alert System (real-time notifications)

Just like a city has different zones working together, Plane has distinct components that communicate to create a cohesive system.

---

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         PRESENTATION LAYER                       │
├──────────────┬──────────────┬──────────────┬───────────────────┤
│   Web App    │  Admin App   │  Space App   │    Live App       │
│ (Main UI)    │ (Settings)   │ (Public)     │  (Real-time)      │
│ Port 3000    │ Port 3001    │ Port 3002    │  Port 3003        │
└──────┬───────┴──────┬───────┴──────┬───────┴───────┬───────────┘
       │              │              │               │
       └──────────────┴──────────────┴───────────────┘
                              │
                    ┌─────────▼─────────┐
                    │   Shared Packages  │
                    │  (types, ui, etc.) │
                    └─────────┬─────────┘
                              │
       ┌──────────────────────┴───────────────────────┐
       │                                              │
┌──────▼──────────────────────────────────────────────▼──────┐
│                     APPLICATION LAYER                       │
│                                                              │
│  ┌────────────────┐    ┌─────────────────────────────────┐ │
│  │  Django API    │◄──►│      Django REST Framework      │ │
│  │  (Business     │    │     (Serializers, Views,        │ │
│  │   Logic)       │    │      Permissions, Auth)         │ │
│  └────────┬───────┘    └─────────────────────────────────┘ │
│           │                                                  │
│  ┌────────▼──────────────────────────────────────────────┐  │
│  │              Middleware Layer                          │  │
│  │  (Auth, CORS, Logging, Request Size, CSRF)           │  │
│  └────────┬──────────────────────────────────────────────┘  │
└───────────┼──────────────────────────────────────────────────┘
            │
    ┌───────┴────────┬──────────────┬────────────────┐
    │                │              │                │
┌───▼────────┐  ┌───▼──────┐  ┌───▼──────┐  ┌─────▼──────┐
│ PostgreSQL │  │  Redis   │  │ Celery   │  │  MongoDB   │
│  (Main DB) │  │ (Cache)  │  │(Workers) │  │(Analytics) │
│            │  │          │  │          │  │            │
└────────────┘  └──────────┘  └──────────┘  └────────────┘

┌────────────────────────────────────────────────────────────┐
│                     INFRASTRUCTURE LAYER                    │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  Docker  │  │   Nginx  │  │   S3/    │  │  Sentry  │  │
│  │Container │  │  Proxy   │  │ Storage  │  │ Logging  │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
└────────────────────────────────────────────────────────────┘
```

---

## Architecture Philosophy

### Core Principles

Plane's architecture follows these guiding principles:

#### 1. **Separation of Concerns** ✅ CURRENT
- Frontend handles presentation and user interaction
- Backend handles business logic and data validation
- Database handles data persistence and integrity
- Each layer has clear responsibilities

**🌉 React Analogy:**
```typescript
// Just like separating concerns in React:
const IssueCard = () => {
  const { issue } = useIssue();           // Data layer
  const { updateIssue } = useActions();   // Business logic
  return <div>{issue.title}</div>;        // Presentation
};
```

#### 2. **Monorepo for Code Sharing** ✅ CURRENT
- Shared packages eliminate code duplication
- Type safety across frontend and backend boundaries
- Single source of truth for constants and types
- Easier refactoring and updates

**Why Monorepo?**
- Before: Copy-paste types between projects ❌
- After: Import from `@plane/types` ✅
- Result: Changes propagate automatically, type-safe everywhere

#### 3. **API-First Design** ✅ CURRENT
- Backend exposes RESTful APIs
- Frontend consumes APIs (never direct DB access)
- APIs are versioned and documented
- Multiple clients can consume the same API

**🎯 Remember This:**
> "Frontend talks to Backend through APIs, never directly to the Database"

#### 4. **Reactive State Management** ✅ CURRENT
- MobX for reactive, observable state
- Automatic UI updates when data changes
- Predictable state flow
- Performance optimizations built-in

#### 5. **Progressive Enhancement** ✅ CURRENT
- Core features work without JavaScript
- Real-time features enhance the experience
- Graceful degradation for offline scenarios

#### 6. **Security by Default** ✅ CURRENT
- Authentication on every API request
- Authorization checks at multiple layers
- CSRF protection built-in
- Input validation and sanitization

---

## System Components

### 1. Frontend Applications

Plane has **4 distinct frontend applications**, each serving different purposes:

#### **Web App** (`/apps/web/`) - Main Application
- **Purpose:** Primary user interface for project management
- **Port:** 3000
- **Tech:** React 18.3 + React Router 7.9 + TypeScript 5.8
- **Users:** Team members, project managers, developers
- **Features:** Issues, projects, workspaces, boards, analytics

**Key File:** `/apps/web/app/root.tsx`

#### **Admin App** (`/apps/admin/`) - Administration Portal
- **Purpose:** Instance administration and configuration
- **Port:** 3001
- **Tech:** React 18.3 + React Router 7.9 + TypeScript 5.8
- **Users:** System administrators
- **Features:** User management, instance settings, licensing

#### **Space App** (`/apps/space/`) - Public Interface
- **Purpose:** Public-facing project views (no auth required)
- **Port:** 3002
- **Tech:** React 18.3 + React Router 7.9 + TypeScript 5.8
- **Users:** Anonymous visitors, stakeholders
- **Features:** Public project boards, read-only issue views

#### **Live App** (`/apps/live/`) - Real-time Collaboration
- **Purpose:** Real-time collaborative features
- **Port:** 3003
- **Tech:** React 18.3 + WebSockets
- **Users:** Team members in active collaboration
- **Features:** Live cursors, real-time updates, presence

**🧠 Mental Model:**
Think of these apps as different "views" into the same data:
- **Web** = Full access dashboard
- **Admin** = Control panel
- **Space** = Public window
- **Live** = Collaboration room

---

### 2. Backend Application

#### **Django API** (`/apps/api/plane/`)
- **Purpose:** Central business logic and data management
- **Tech:** Django 4.2 + Django REST Framework 3.15 + Python 3.12
- **Architecture:** MTV (Model-Template-View) pattern
- **API Style:** RESTful with some GraphQL endpoints

**Key Components:**

```
/apps/api/plane/
├── db/
│   └── models/          # Database models (your "schema")
├── app/
│   ├── views/           # API endpoints
│   ├── serializers/     # Data transformation
│   └── permissions/     # Authorization logic
├── authentication/      # Auth logic
├── bgtasks/            # Celery tasks
├── settings/           # Django configuration
└── urls.py             # API routing
```

**🌉 React to Django Bridge:**

```typescript
// React Component (Frontend)
const IssueList = () => {
  const { data } = useSWR('/api/issues/');
  return <div>{data.map(issue => ...)}</div>;
};
```

```python
# Django View (Backend)
class IssueViewSet(BaseViewSet):
    def list(self, request):
        issues = Issue.objects.filter(project=project)
        return Response(IssueSerializer(issues, many=True).data)
```

**See it in action:**
- Models: `/apps/api/plane/db/models/issue.py`
- Views: `/apps/api/plane/app/views/issue.py`
- Serializers: `/apps/api/plane/app/serializers/issue.py`

---

### 3. Shared Packages

The monorepo architecture provides **15+ shared packages** that all apps can use:

#### **Core Packages:**

| Package | Purpose | Used By |
|---------|---------|---------|
| `@plane/types` | TypeScript type definitions | All frontend apps |
| `@plane/ui` | Reusable UI components | All frontend apps |
| `@plane/constants` | Shared constants (status codes, etc.) | Frontend + Backend |
| `@plane/utils` | Utility functions | All apps |
| `@plane/hooks` | Custom React hooks | Frontend apps |
| `@plane/services` | API service layer | Frontend apps |
| `@plane/shared-state` | MobX stores | Frontend apps |
| `@plane/editor` | Rich text editor | Web + Space |
| `@plane/i18n` | Internationalization | All frontend apps |

**✅ CURRENT PATTERN: Workspace Imports**

```typescript
// Import shared types
import type { TIssue } from "@plane/types";

// Import shared UI components
import { Button, Avatar } from "@plane/ui";

// Import shared constants
import { ISSUE_PRIORITIES } from "@plane/constants";

// Import shared hooks
import { useLocalStorage } from "@plane/hooks";
```

**Location:** `/packages/`

**💡 Aha Moment:**
> Shared packages are like React's component library, but for the entire application stack. Change once, update everywhere!

---

### 4. Database Layer

#### **PostgreSQL** - Primary Database
- **Purpose:** Persistent storage of all application data
- **Version:** 14+
- **Use Cases:** Issues, projects, users, workspaces, comments
- **Access Pattern:** Through Django ORM only

**Key Tables:**
- `plane_issue` - Core issue data
- `plane_project` - Projects
- `plane_workspace` - Workspaces
- `plane_user` - User accounts
- `plane_state` - Issue states

**See schema:** `/apps/api/plane/db/models/`

#### **Redis** - Cache & Session Store
- **Purpose:** Fast temporary storage
- **Version:** 7+
- **Use Cases:**
  - Session storage
  - Cache frequently accessed data
  - Rate limiting
  - Pub/Sub for real-time features

#### **MongoDB** - Analytics (Optional)
- **Purpose:** Analytics data storage
- **Use Cases:** Event tracking, usage metrics

---

### 5. Background Processing

#### **Celery** - Asynchronous Task Queue
- **Purpose:** Handle long-running operations outside request/response cycle
- **Version:** 5.4+
- **Broker:** Redis

**Common Tasks:**
- Email notifications
- Export generation (CSV, PDF)
- Data synchronization
- Scheduled reports
- Webhook delivery

**Example Task:**

```python
# /apps/api/plane/bgtasks/issue_automation.py
from celery import shared_task

@shared_task
def send_issue_notification(issue_id):
    """Background task to send notifications"""
    issue = Issue.objects.get(id=issue_id)
    notify_users(issue)
```

**🌉 React Analogy:**
Celery is like `useEffect` with cleanup - it runs "in the background" without blocking the UI:

```typescript
// React: Non-blocking async operation
useEffect(() => {
  fetchData().then(setData); // Doesn't block render
}, []);

// Celery: Non-blocking async task
create_issue(data)
send_notifications.delay(issue_id)  # Doesn't block response
```

---

## Monorepo Architecture

### What is a Monorepo?

**🧠 Mental Model:**
A monorepo is like having all your React components in one `/components/` folder instead of scattered across multiple projects.

```
Plane Monorepo Structure:
/
├── apps/              # Applications (like pages in Next.js)
│   ├── web/          # Main app
│   ├── admin/        # Admin app
│   ├── space/        # Public app
│   ├── live/         # Real-time app
│   └── api/          # Backend API
├── packages/         # Shared code (like /lib/ or /utils/)
│   ├── types/
│   ├── ui/
│   ├── hooks/
│   └── ...
├── turbo.json        # Build orchestration
├── pnpm-workspace.yaml # Workspace config
└── package.json      # Root dependencies
```

### Benefits of Monorepo

#### 1. **Single Source of Truth** ✅
```typescript
// Define once in @plane/types
export type TIssue = {
  id: string;
  title: string;
  // ...
};

// Use everywhere
import type { TIssue } from "@plane/types"; // web app
import type { TIssue } from "@plane/types"; // admin app
import type { TIssue } from "@plane/types"; // space app
```

#### 2. **Atomic Changes** ✅
```bash
# One PR can update interface + all consumers
git commit -m "refactor: rename Issue.name to Issue.title"
# Updates:
# - @plane/types
# - web app
# - admin app
# - space app
# All in sync!
```

#### 3. **Build Optimization** ✅
Turborepo intelligently caches builds:
```bash
pnpm build  # First time: 5 minutes
# Change one file in web app
pnpm build  # Second time: 30 seconds (cached packages!)
```

### Turborepo Configuration

**File:** `/turbo.json`

```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],        // Build dependencies first
      "outputs": ["dist/**"]           // Cache these outputs
    },
    "dev": {
      "cache": false,                  // Don't cache dev
      "persistent": true               // Keep running
    }
  }
}
```

**🎯 Remember This:**
> Turborepo is like Webpack for your entire monorepo - it knows what changed and only rebuilds what's needed.

---

## Frontend Architecture

### React Router 7 Framework Mode ✅ CURRENT

Plane uses **React Router 7 in framework mode**, which provides:

1. **File-based routing** (like Next.js App Router)
2. **Data loaders** (fetch before render)
3. **Server-side rendering** (SSR)
4. **Optimistic UI** patterns
5. **Form actions**

**Example Route:**

```typescript
// /apps/web/app/routes/projects.$projectId.issues.tsx
import type { LoaderFunctionArgs } from "react-router";

// 1. Loader runs on server
export async function loader({ params }: LoaderFunctionArgs) {
  const issues = await fetchIssues(params.projectId);
  return json({ issues });
}

// 2. Component renders with data
export default function IssuesPage() {
  const { issues } = useLoaderData<typeof loader>();

  return (
    <div>
      {issues.map(issue => <IssueCard key={issue.id} issue={issue} />)}
    </div>
  );
}
```

**🌉 Coming from Create React App?**

| Old Pattern (CRA) | New Pattern (React Router 7) |
|-------------------|------------------------------|
| `useEffect` + fetch | `loader` function |
| Client-side routing | Server-side routing |
| Single bundle | Route-based code splitting |
| Client-only rendering | Server + client rendering |

**⚠️ OUTDATED PATTERN:**
```typescript
// ❌ Don't do this anymore
const Issues = () => {
  const [issues, setIssues] = useState([]);

  useEffect(() => {
    fetch('/api/issues').then(r => r.json()).then(setIssues);
  }, []);

  return <div>...</div>;
};
```

**✅ CURRENT PATTERN:**
```typescript
// ✅ Do this instead
export async function loader() {
  return json({ issues: await fetchIssues() });
}

export default function Issues() {
  const { issues } = useLoaderData<typeof loader>();
  return <div>...</div>;
}
```

---

### MobX State Management ✅ CURRENT

Plane uses **MobX 6.12** for reactive state management.

**🧠 Mental Model:**
MobX is like Excel spreadsheets:
- Change a cell → All formulas recalculate automatically
- Change MobX state → All components re-render automatically

**Example Store:**

```typescript
// /packages/shared-state/src/store/issue.store.ts
import { makeObservable, observable, action, computed } from "mobx";

class IssueStore {
  // Observable state (like useState)
  issues = new Map<string, TIssue>();

  constructor() {
    makeObservable(this, {
      issues: observable,
      addIssue: action,
      issueCount: computed,
    });
  }

  // Actions (like setState)
  addIssue = (issue: TIssue) => {
    this.issues.set(issue.id, issue);
  };

  // Computed values (like useMemo)
  get issueCount() {
    return this.issues.size;
  }
}
```

**Using in Components:**

```typescript
import { observer } from "mobx-react";
import { useIssueStore } from "@plane/shared-state";

// observer() makes component reactive
const IssueList = observer(() => {
  const store = useIssueStore();

  // Automatically re-renders when store.issues changes
  return <div>Total: {store.issueCount}</div>;
});
```

**🌉 MobX vs React State:**

| React State | MobX |
|-------------|------|
| `useState` | `observable` |
| `setState` | `action` |
| `useMemo` | `computed` |
| `useCallback` | Not needed (MobX handles) |
| Manual re-renders | Automatic re-renders |

**See stores:** `/packages/shared-state/src/store/`

---

### TypeScript Patterns ✅ CURRENT

Plane uses **TypeScript 5.8** with strict mode enabled.

**Type Safety Example:**

```typescript
// Define in @plane/types
export type TIssue = {
  id: string;
  title: string;
  description: string | null;
  priority: "urgent" | "high" | "medium" | "low" | "none";
  state: TState;
  assignees: string[];
  created_at: string;
  updated_at: string;
};

// Use in components
const IssueCard = ({ issue }: { issue: TIssue }) => {
  // TypeScript knows all properties
  return (
    <div>
      <h3>{issue.title}</h3>
      <span>{issue.priority}</span> {/* Autocomplete! */}
    </div>
  );
};
```

**💡 Aha Moment:**
> TypeScript catches bugs at compile-time that would otherwise crash your app at runtime!

---

### Data Fetching with SWR ✅ CURRENT

Plane uses **SWR 2.2** for data fetching and caching.

**Example:**

```typescript
import useSWR from "swr";
import { IssueService } from "@plane/services";

const IssueList = () => {
  const { data, error, mutate } = useSWR(
    `/api/issues/`,
    () => IssueService.getIssues()
  );

  if (error) return <div>Error loading issues</div>;
  if (!data) return <div>Loading...</div>;

  return (
    <div>
      {data.map(issue => <IssueCard key={issue.id} issue={issue} />)}
    </div>
  );
};
```

**SWR Features Used:**
- **Automatic Caching** - Fetches once, caches result
- **Automatic Revalidation** - Refetches on focus/reconnect
- **Optimistic Updates** - Update UI before API confirms
- **Error Retry** - Automatically retries failed requests
- **Pagination** - Built-in pagination support

---

## Backend Architecture

### Django MTV Pattern

Django follows the **MTV (Model-Template-View)** pattern:

**🌉 React to Django Bridge:**

| React | Django | Purpose |
|-------|--------|---------|
| Component | Template | Presentation (but we use React!) |
| Props/State | View | Business logic |
| API Response | Serializer | Data transformation |
| TypeScript Type | Model | Data structure |

**Django Request Flow:**

```
1. URL Router      →  /api/issues/
2. View            →  IssueViewSet.list()
3. Permission      →  Check user can view issues
4. Business Logic  →  Filter, sort, paginate
5. Serializer      →  Transform to JSON
6. Response        →  Return to frontend
```

### Django Models (Database Schema)

**🧠 Mental Model:**
Django models are like TypeScript interfaces that CREATE database tables.

```typescript
// TypeScript (describes structure)
type Issue = {
  id: string;
  title: string;
};
```

```python
# Django Model (creates table)
class Issue(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid4)
    title = models.CharField(max_length=255)

    class Meta:
        db_table = "plane_issue"
```

**Key Models:**
- **User** - `/apps/api/plane/db/models/user.py`
- **Workspace** - `/apps/api/plane/db/models/workspace.py`
- **Project** - `/apps/api/plane/db/models/project.py`
- **Issue** - `/apps/api/plane/db/models/issue.py`
- **State** - `/apps/api/plane/db/models/state.py`

---

### Django REST Framework (DRF)

DRF provides the API layer on top of Django.

**ViewSet Example:**

```python
# /apps/api/plane/app/views/issue.py
from rest_framework import status
from rest_framework.response import Response
from plane.app.views import BaseViewSet

class IssueViewSet(BaseViewSet):
    # GET /api/workspaces/:workspace/projects/:project/issues/
    def list(self, request, workspace_slug, project_id):
        issues = Issue.objects.filter(
            project_id=project_id,
            workspace__slug=workspace_slug
        )

        serializer = IssueSerializer(issues, many=True)
        return Response(serializer.data, status=status.HTTP_200_OK)

    # POST /api/workspaces/:workspace/projects/:project/issues/
    def create(self, request, workspace_slug, project_id):
        serializer = IssueSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save(project_id=project_id)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

**Serializer Example:**

```python
# /apps/api/plane/app/serializers/issue.py
from rest_framework import serializers
from plane.db.models import Issue

class IssueSerializer(serializers.ModelSerializer):
    # Custom fields
    assignee_details = UserLiteSerializer(source='assignees', many=True)

    class Meta:
        model = Issue
        fields = [
            'id',
            'title',
            'description',
            'priority',
            'state',
            'assignee_details',
            'created_at',
            'updated_at',
        ]
        read_only_fields = ['id', 'created_at', 'updated_at']
```

**🌉 React to Serializer Bridge:**

```typescript
// React: Transform data for display
const displayIssue = {
  ...issue,
  assigneeNames: issue.assignees.map(a => a.name).join(", ")
};

// Django: Transform data for API
class IssueSerializer:
    assignee_names = serializers.SerializerMethodField()

    def get_assignee_names(self, obj):
        return ", ".join([a.name for a in obj.assignees.all()])
```

---

## Data Architecture

### Database Design Principles

#### 1. **Normalized Schema** ✅
- Minimize data duplication
- Use foreign keys for relationships
- Enforce referential integrity

**Example:**
```python
class Issue(models.Model):
    project = models.ForeignKey(Project, on_delete=models.CASCADE)
    state = models.ForeignKey(State, on_delete=models.CASCADE)
    assignees = models.ManyToManyField(User, related_name="assigned_issues")
```

#### 2. **Soft Deletion** ✅
- Never hard-delete data
- Mark as deleted, keep in database
- Allows recovery and audit trails

```python
class Issue(models.Model):
    deleted_at = models.DateTimeField(null=True, blank=True)

    objects = IssueManager()  # Returns non-deleted
    all_objects = models.Manager()  # Returns all
```

#### 3. **Timestamps** ✅
- Track when records created/updated
- Essential for sync and audit

```python
class BaseModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True
```

---

### Database Relationships

**🧠 Mental Model:**
Database relationships are like TypeScript references:

```typescript
// TypeScript (by reference)
type Issue = {
  id: string;
  project: Project;  // Reference to Project object
  state: State;      // Reference to State object
};
```

```python
# Django (foreign keys)
class Issue(models.Model):
    project = models.ForeignKey(Project)  # Reference to Project record
    state = models.ForeignKey(State)      # Reference to State record
```

**Relationship Types:**

1. **One-to-Many** (ForeignKey)
   - One Project has many Issues
   - One Issue belongs to one Project

2. **Many-to-Many** (ManyToManyField)
   - One Issue has many Labels
   - One Label can be on many Issues

3. **One-to-One** (OneToOneField)
   - One User has one Profile
   - One Profile belongs to one User

**See relationships:** `/apps/api/plane/db/models/`

---

## Communication Patterns

### Frontend → Backend Communication

#### 1. **REST API** (Primary) ✅ CURRENT

```typescript
// Frontend: Make API request
import { APIService } from "@plane/services";

const createIssue = async (data: Partial<TIssue>) => {
  const response = await APIService.post('/api/issues/', data);
  return response.data;
};
```

```python
# Backend: Handle API request
@api_view(['POST'])
def create_issue(request):
    serializer = IssueSerializer(data=request.data)
    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data, status=201)
    return Response(serializer.errors, status=400)
```

#### 2. **WebSockets** (Real-time) ✅ CURRENT

Used for real-time features:
- Live cursors
- Presence indicators
- Real-time updates
- Collaborative editing

```typescript
// Frontend: Subscribe to updates
const socket = new WebSocket('ws://localhost:8000/ws/issues/');

socket.onmessage = (event) => {
  const update = JSON.parse(event.data);
  // Update UI in real-time
};
```

#### 3. **Server-Sent Events (SSE)** (Notifications)

Used for one-way server → client updates:
- Notifications
- Progress updates
- System alerts

---

### API Design Patterns

#### RESTful Resource Naming ✅ CURRENT

```
GET    /api/workspaces/                          # List all workspaces
GET    /api/workspaces/:workspace/               # Get one workspace
POST   /api/workspaces/                          # Create workspace
PATCH  /api/workspaces/:workspace/               # Update workspace
DELETE /api/workspaces/:workspace/               # Delete workspace

GET    /api/workspaces/:workspace/projects/      # List projects
GET    /api/workspaces/:workspace/projects/:id/  # Get project
```

**🎯 Remember This:**
> REST URLs are nouns (resources), not verbs (actions). Use HTTP methods for actions.

**✅ Good:**
- `POST /api/issues/` (create issue)
- `PATCH /api/issues/:id/` (update issue)
- `DELETE /api/issues/:id/` (delete issue)

**❌ Bad:**
- `POST /api/createIssue/`
- `POST /api/updateIssue/`
- `POST /api/deleteIssue/`

---

## State Management Strategy

### Three Levels of State

Plane manages state at three levels:

#### 1. **Server State** (Database)
- **What:** Data persisted in PostgreSQL
- **Examples:** Issues, projects, users
- **Managed By:** Django ORM
- **Access Via:** API endpoints

#### 2. **Client State** (Frontend Cache)
- **What:** Cached server data in browser
- **Examples:** Fetched issues, user profile
- **Managed By:** SWR + MobX
- **Access Via:** Custom hooks

#### 3. **UI State** (Local Component State)
- **What:** Ephemeral UI-only state
- **Examples:** Modal open/closed, form values
- **Managed By:** React useState
- **Access Via:** Component props/state

**🧠 Mental Model:**

```typescript
// 1. Server State (in PostgreSQL)
const issue = { id: "123", title: "Bug", ... }; // Persisted

// 2. Client State (in browser memory)
const { data } = useSWR('/api/issues/123'); // Cached

// 3. UI State (in component)
const [isModalOpen, setIsModalOpen] = useState(false); // Ephemeral
```

---

### State Synchronization

**How state stays in sync:**

```
1. User updates issue in UI
   ↓
2. Optimistic update in MobX store (instant feedback)
   ↓
3. API request sent to backend
   ↓
4. Django validates & saves to PostgreSQL
   ↓
5. API response confirms success
   ↓
6. SWR revalidates cache
   ↓
7. MobX updates from cache
   ↓
8. UI reflects final state
```

**Example:**

```typescript
const updateIssue = async (id: string, data: Partial<TIssue>) => {
  // 1. Optimistic update
  issueStore.updateIssue(id, data);

  try {
    // 2. Send to backend
    const response = await IssueService.update(id, data);

    // 3. Update with server response
    issueStore.updateIssue(id, response);

    // 4. Revalidate SWR cache
    mutate(`/api/issues/${id}`);
  } catch (error) {
    // 5. Rollback on error
    issueStore.rollback(id);
  }
};
```

---

## Caching Layers

Plane uses multiple caching layers for performance:

### 1. **Browser Cache** (Client-side)
- **What:** Browser caches static assets (JS, CSS, images)
- **Duration:** Based on cache headers
- **Invalidation:** On deployment (new build hash)

### 2. **SWR Cache** (Client-side)
- **What:** Cached API responses in memory
- **Duration:** Until page refresh or manual invalidation
- **Invalidation:** On mutation, focus, or interval

```typescript
// SWR auto-caches this request
const { data } = useSWR('/api/issues/', fetcher);

// Manual cache invalidation
mutate('/api/issues/');
```

### 3. **Redis Cache** (Server-side)
- **What:** Frequently accessed data
- **Duration:** Configurable (e.g., 5 minutes)
- **Invalidation:** On write or TTL expiry

```python
# Cache frequently accessed data
@cache_decorator(timeout=300)  # 5 minutes
def get_project(project_id):
    return Project.objects.get(id=project_id)
```

### 4. **Database Query Cache** (Database-level)
- **What:** PostgreSQL query results
- **Duration:** Automatic (Postgres manages)
- **Invalidation:** On data change

---

### Cache Invalidation Strategy

**🎯 Remember This:**
> "There are only two hard things in Computer Science: cache invalidation and naming things." - Phil Karlton

**Plane's Strategy:**

1. **Write-through caching**
   - Update database first
   - Then invalidate cache
   - Ensures data consistency

2. **Stale-while-revalidate**
   - Return cached data immediately
   - Fetch fresh data in background
   - Update cache when fresh data arrives

**Example:**

```typescript
// SWR implements stale-while-revalidate
const { data } = useSWR('/api/issues/', fetcher, {
  revalidateOnFocus: true,      // Revalidate when window focuses
  revalidateOnReconnect: true,  // Revalidate when reconnects
  refreshInterval: 30000,       // Poll every 30 seconds
});
```

---

## Real-Time Features

### WebSocket Architecture

**Use Cases:**
- Presence indicators (who's online)
- Live cursors (collaborative editing)
- Real-time notifications
- Instant issue updates

**Flow:**

```
1. User opens issue page
   ↓
2. Frontend establishes WebSocket connection
   ↓
3. Backend sends initial state
   ↓
4. Another user updates issue
   ↓
5. Backend broadcasts update via WebSocket
   ↓
6. All connected clients receive update
   ↓
7. UI updates in real-time
```

**Implementation:**

```typescript
// Frontend: Connect to WebSocket
const socket = new WebSocket(`ws://localhost:8000/ws/issues/${issueId}/`);

socket.onmessage = (event) => {
  const { type, data } = JSON.parse(event.data);

  switch (type) {
    case 'issue_updated':
      issueStore.updateIssue(data);
      break;
    case 'user_joined':
      presenceStore.addUser(data);
      break;
  }
};
```

---

## Background Processing

### Celery Task Queue

**Why Background Processing?**

Some operations take too long for HTTP request/response cycle:
- Sending emails
- Generating reports
- Processing uploads
- Running analytics

**🌉 React Analogy:**

```typescript
// React: Async doesn't block render
const handleClick = async () => {
  setLoading(true);
  await someSlowOperation();
  setLoading(false);
};

// Django: Celery doesn't block response
def create_issue(request):
    issue = Issue.objects.create(**data)
    send_notifications.delay(issue.id)  # Background task
    return Response(IssueSerializer(issue).data)  # Immediate response
```

**Common Tasks:**

```python
# /apps/api/plane/bgtasks/notification.py
@shared_task
def send_issue_notification(issue_id, user_ids):
    """Send notifications asynchronously"""
    issue = Issue.objects.get(id=issue_id)
    for user_id in user_ids:
        send_email(user_id, f"New issue: {issue.title}")

# /apps/api/plane/bgtasks/export.py
@shared_task
def export_issues_to_csv(project_id):
    """Generate CSV export in background"""
    issues = Issue.objects.filter(project_id=project_id)
    csv_file = generate_csv(issues)
    return csv_file.url
```

**Task Workflow:**

```
1. User clicks "Export Issues"
   ↓
2. API creates export task: export_issues_to_csv.delay(project_id)
   ↓
3. Task queued in Redis
   ↓
4. API returns immediately: "Export started"
   ↓
5. Celery worker picks up task
   ↓
6. Worker generates CSV (takes 2 minutes)
   ↓
7. Worker uploads to S3
   ↓
8. Worker updates export record: status = "completed"
   ↓
9. Frontend polls API for completion
   ↓
10. User gets download link
```

---

## Scalability Considerations

### Horizontal Scaling

**🧠 Mental Model:**
Scaling is like adding more checkout lanes at a grocery store.

**Components that scale horizontally:**

1. **Frontend Servers** ✅
   - Multiple instances behind load balancer
   - Stateless (no session storage)
   - Easy to scale

2. **Backend API Servers** ✅
   - Multiple Django instances
   - Stateless (sessions in Redis)
   - Scale based on request volume

3. **Celery Workers** ✅
   - Add more workers for more throughput
   - Each worker processes tasks independently

**Components that DON'T scale horizontally easily:**

1. **PostgreSQL** ⚠️
   - Primary-replica setup
   - Read replicas for read-heavy loads
   - Write operations limited to primary

2. **Redis** ⚠️
   - Can cluster, but adds complexity
   - Usually single instance is sufficient

---

### Performance Optimization

#### Database Level
- **Indexes** on frequently queried fields
- **Query optimization** (select_related, prefetch_related)
- **Connection pooling** (reuse DB connections)
- **Database vacuuming** (PostgreSQL maintenance)

```python
# ✅ Optimized query
issues = Issue.objects.select_related('project', 'state').prefetch_related('assignees')

# ❌ N+1 query problem
issues = Issue.objects.all()
for issue in issues:
    print(issue.project.name)  # Separate query for each!
```

#### API Level
- **Pagination** (limit results per page)
- **Field filtering** (return only needed fields)
- **Response compression** (gzip)
- **Rate limiting** (prevent abuse)

```python
# Pagination
class IssueViewSet(BaseViewSet):
    pagination_class = PageNumberPagination
    page_size = 50
```

#### Frontend Level
- **Code splitting** (load only needed code)
- **Lazy loading** (defer non-critical components)
- **Image optimization** (WebP, lazy loading)
- **Memoization** (cache expensive calculations)

```typescript
// Code splitting
const IssueDetail = lazy(() => import('./IssueDetail'));

// Memoization
const sortedIssues = useMemo(
  () => issues.sort((a, b) => a.title.localeCompare(b.title)),
  [issues]
);
```

---

## Security Architecture

### Defense in Depth

Plane uses multiple security layers:

#### 1. **Network Layer**
- HTTPS only (TLS 1.2+)
- CORS configuration
- Rate limiting
- DDoS protection

#### 2. **Application Layer**
- Authentication required for all endpoints
- CSRF protection
- SQL injection prevention (ORM)
- XSS prevention (input sanitization)

#### 3. **Data Layer**
- Encrypted at rest (database)
- Encrypted in transit (HTTPS)
- Access controls (row-level security)
- Audit logs

---

### Authentication & Authorization

**Authentication:** Who are you?
**Authorization:** What can you do?

#### Session-Based Authentication ✅ CURRENT

```python
# Django creates session on login
def login(request):
    user = authenticate(username=username, password=password)
    django_login(request, user)  # Creates session
    return Response(UserSerializer(user).data)

# Session stored in Redis
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "default"  # Redis
```

```typescript
// Frontend: Session cookie automatically sent
const { data } = await axios.get('/api/issues/');
// Browser includes session cookie in request
```

#### Permission Checks ✅ CURRENT

```python
# /apps/api/plane/app/permissions/project.py
class ProjectPermission(BasePermission):
    def has_permission(self, request, view):
        # Check user is member of workspace
        workspace = get_workspace(request)
        return WorkspaceMember.objects.filter(
            workspace=workspace,
            member=request.user
        ).exists()

    def has_object_permission(self, request, view, obj):
        # Check user can access this specific project
        return ProjectMember.objects.filter(
            project=obj,
            member=request.user
        ).exists()
```

**🌉 React to Permissions Bridge:**

```typescript
// Frontend: Check permissions in UI
const IssueActions = ({ issue }: { issue: TIssue }) => {
  const { user } = useUser();
  const canEdit = issue.created_by === user.id || user.role === "admin";

  return (
    <div>
      {canEdit && <Button onClick={handleEdit}>Edit</Button>}
    </div>
  );
};
```

```python
# Backend: Enforce permissions in API
class IssueViewSet(BaseViewSet):
    permission_classes = [ProjectPermission]

    def update(self, request, pk=None):
        # Permission check happens automatically
        issue = self.get_object()  # Returns 403 if no permission
        serializer = IssueSerializer(issue, data=request.data)
        serializer.save()
        return Response(serializer.data)
```

---

## Development Architecture

### Local Development Setup

**Services Required:**

```yaml
# docker-compose.dev.yml
services:
  postgres:
    image: postgres:14
    ports: ["5432:5432"]

  redis:
    image: redis:7
    ports: ["6379:6379"]

  api:
    build: ./apps/api
    ports: ["8000:8000"]
    depends_on: [postgres, redis]

  web:
    build: ./apps/web
    ports: ["3000:3000"]

  celery:
    build: ./apps/api
    command: celery -A plane worker
    depends_on: [postgres, redis]
```

**Start Development:**

```bash
# 1. Start backend services
docker-compose up -d postgres redis

# 2. Start Django API
cd apps/api
python manage.py runserver

# 3. Start Celery worker
celery -A plane worker

# 4. Start frontend (in separate terminal)
cd apps/web
pnpm dev
```

---

### Development Tools

#### Hot Module Replacement (HMR) ✅
- Frontend changes reflect instantly
- No page reload needed
- State preserved during updates

#### Django Debug Toolbar
- SQL query profiling
- Template rendering time
- Cache hit/miss ratios

#### React DevTools
- Component hierarchy inspection
- Props/state debugging
- Performance profiling

#### MobX DevTools
- Store state inspection
- Action tracking
- Computed value debugging

---

## Deployment Architecture

### Production Setup

```
┌──────────────────────────────────────────────────┐
│              Load Balancer (Nginx)                │
│              SSL Termination                      │
└───────┬──────────────────────────┬────────────────┘
        │                          │
┌───────▼──────────┐      ┌────────▼───────────┐
│  Frontend Servers │      │   Backend Servers   │
│   (Static Files)  │      │  (Django + Gunicorn)│
│   Multiple nodes  │      │   Multiple nodes    │
└───────┬───────────┘      └────────┬───────────┘
        │                           │
        │    ┌──────────────────────┼──────┐
        │    │                      │      │
┌───────▼────▼───┐  ┌──────────────▼──┐ ┌─▼─────────┐
│   PostgreSQL    │  │     Redis       │ │  Celery   │
│   (Primary)     │  │    (Cache)      │ │ (Workers) │
└─────────────────┘  └─────────────────┘ └───────────┘
```

### Deployment Process

```bash
# 1. Build frontend
cd apps/web
pnpm build

# 2. Build backend
cd apps/api
pip install -r requirements.txt
python manage.py collectstatic

# 3. Run database migrations
python manage.py migrate

# 4. Restart services
systemctl restart gunicorn
systemctl restart celery
nginx -s reload
```

---

## Trade-offs & Design Decisions

### Why React Router 7 instead of Next.js?

**✅ Pros:**
- More flexible routing
- Better TypeScript integration
- Smaller bundle size
- No vendor lock-in

**⚠️ Cons:**
- Less mature ecosystem
- Fewer built-in optimizations
- Smaller community

**Decision:** React Router 7 provides more control and better fits Plane's architecture.

---

### Why MobX instead of Redux?

**✅ Pros:**
- Less boilerplate
- Automatic reactivity
- Simpler mental model
- Better performance for large state

**⚠️ Cons:**
- More "magic" (less explicit)
- Easier to make mistakes
- Smaller community

**Decision:** MobX's reactivity and simplicity outweigh Redux's explicitness.

---

### Why Django instead of Node.js?

**✅ Pros:**
- Batteries-included (admin, ORM, auth)
- Strong type system (Python type hints)
- Mature ecosystem
- Excellent ORM

**⚠️ Cons:**
- Different language from frontend
- Potentially slower than Node.js
- More memory usage

**Decision:** Django's maturity and built-in features accelerate development.

---

### Why Monorepo instead of Polyrepo?

**✅ Pros:**
- Single source of truth
- Atomic changes across packages
- Easier refactoring
- Shared tooling

**⚠️ Cons:**
- Larger repository
- Longer clone time
- More complex CI/CD

**Decision:** Benefits of code sharing outweigh complexity.

---

## Next Steps

### Continue Learning

Now that you understand the architecture, dive deeper:

1. **[PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md)** - Detailed codebase structure
2. **[TECH_STACK_GUIDE.md](./TECH_STACK_GUIDE.md)** - Deep dive into each technology
3. **[DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md)** - How data flows through the system
4. **[FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md)** - React best practices
5. **[BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)** - Django deep dive

---

### Quick Self-Check

✅ **Test your understanding:**

1. Can you explain the difference between server state, client state, and UI state?
2. What are the four frontend applications and their purposes?
3. How does data flow from PostgreSQL to a React component?
4. What's the difference between MobX and SWR?
5. Why does Plane use Celery for background tasks?

**If you can answer these, you're ready to move on!**

---

### Get Hands-On

Ready to see the architecture in action?

1. **[CODE_TOURS.md](./CODE_TOURS.md)** - Guided walkthroughs
2. **[HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)** - Build your first feature
3. **[EXERCISES.md](./EXERCISES.md)** - Coding challenges

---

**🎉 Congratulations!** You now have a solid understanding of Plane's architecture. Time to build something amazing!

---

**Questions or Feedback?**
- Check **[FAQ.md](./FAQ.md)** for common questions
- Join the Plane community discussions
- Open an issue if you find gaps in this documentation

---

**Last Updated:** November 19, 2025
**Maintainers:** Plane Learning Docs Team
**License:** AGPL-3.0
