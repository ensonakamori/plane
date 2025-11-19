# Backend Architecture - Django for React Developers

**Master Django as a React Developer!** This guide translates backend concepts into terms you already understand, making Django feel familiar and approachable.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 2-3 hours
**📊 Difficulty Level:** Intermediate to Advanced
**🎯 Target Audience:** React developers learning backend for the first time

---

## Table of Contents

1. [Introduction - React Meets Django](#introduction---react-meets-django)
2. [Mental Model: React vs Django](#mental-model-react-vs-django)
3. [Django MTV Pattern Explained](#django-mtv-pattern-explained)
4. [Models - Your Database Schema as Code](#models---your-database-schema-as-code)
5. [Views - API Endpoints Like Express Routes](#views---api-endpoints-like-express-routes)
6. [Serializers - Data Transformers](#serializers---data-transformers)
7. [Django REST Framework (DRF)](#django-rest-framework-drf)
8. [Request Lifecycle - From Browser to Database](#request-lifecycle---from-browser-to-database)
9. [Authentication & Permissions](#authentication--permissions)
10. [Database Relationships](#database-relationships)
11. [Querysets - SQL Made Easy](#querysets---sql-made-easy)
12. [Celery - Background Jobs](#celery---background-jobs)
13. [Real Code Examples from Plane](#real-code-examples-from-plane)
14. [Common Patterns](#common-patterns)
15. [Debugging Django](#debugging-django)
16. [Next Steps](#next-steps)

---

## Introduction - React Meets Django

### Why Django?

You're a React developer. You build UIs, manage state, fetch data from APIs. But where does that data come from? How do APIs work? What happens when you `POST` to `/api/issues/`?

**This guide answers those questions** by teaching you Django through the lens of React.

### What You'll Learn

By the end of this guide, you'll understand:

- ✅ How Django models create database tables (like TypeScript interfaces, but with superpowers)
- ✅ How views handle HTTP requests (like Express.js route handlers)
- ✅ How serializers transform data (like JSON.stringify, but smarter)
- ✅ How authentication works (JWT tokens, permissions, middleware)
- ✅ How to read and write Django code confidently
- ✅ How a request flows from frontend → Django → database → response

### Prerequisites

**Required:**
- ✅ Basic Python syntax (functions, classes, decorators)
- ✅ React knowledge (components, props, state)
- ✅ REST API concepts (GET, POST, PUT, DELETE)

**Not Required (We'll Teach You):**
- ❌ Django experience
- ❌ Database/SQL knowledge
- ❌ Backend development

---

## Mental Model: React vs Django

### The Big Picture

Think of Django as **React for your backend**:

| React (Frontend) | Django (Backend) |
|------------------|------------------|
| **Components** render UI | **Views** return data |
| **Props** pass data down | **Serializers** transform data |
| **State** holds UI data | **Models** hold database data |
| **useEffect** fetches data | **Querysets** fetch data |
| **React Router** handles URLs | **Django URLs** route requests |
| **TypeScript types** ensure safety | **Models** enforce schema |

### Key Differences

**React is STATELESS (usually):**
```tsx
// React component - no persistent state
function IssueCard({ issue }: { issue: Issue }) {
  return <div>{issue.title}</div>;
}
// When page refreshes, data is gone unless refetched
```

**Django is STATEFUL (database):**
```python
# Django model - persistent state
class Issue(models.Model):
    title = models.CharField(max_length=255)
    # Data lives in PostgreSQL forever
```

### 🧠 Mental Model

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND (React)                      │
│  User clicks button → Component updates → State changes     │
│                           ↓                                  │
│                    fetch('/api/issues/')                     │
└─────────────────────────┬───────────────────────────────────┘
                          │ HTTP Request (JSON)
┌─────────────────────────┴───────────────────────────────────┐
│                       BACKEND (Django)                       │
│  URL Router → View → Serializer → Model → Database          │
│                           ↓                                  │
│  Database → Model → Serializer → View → JSON Response       │
└─────────────────────────┬───────────────────────────────────┘
                          │ HTTP Response (JSON)
┌─────────────────────────┴───────────────────────────────────┐
│                        FRONTEND (React)                      │
│  Response → setState → Component re-renders → UI updates    │
└─────────────────────────────────────────────────────────────┘
```

---

## Django MTV Pattern Explained

### What is MTV?

Django uses **MTV (Model-Template-View)**:
- **M**odel = Database schema
- **T**emplate = API responses (we don't use HTML templates)
- **V**iew = Request handler

**For APIs (like Plane), think of it as MVS:**
- **M**odel = Database schema
- **V**iew = API endpoint logic
- **S**erializer = Data transformer

### React Analogy

```tsx
// React Pattern
interface Issue {          // TypeScript type
  id: string;
  title: string;
}

function IssueList() {     // Component (renders UI)
  const [issues, setIssues] = useState<Issue[]>([]);

  useEffect(() => {
    fetch('/api/issues/')  // Fetch from backend
      .then(r => r.json())
      .then(setIssues);
  }, []);

  return issues.map(i => <IssueCard key={i.id} issue={i} />);
}
```

```python
# Django Pattern (backend)

# Model - Database schema (like TypeScript interface + database)
class Issue(models.Model):
    id = models.UUIDField(primary_key=True)
    title = models.CharField(max_length=255)

# Serializer - Data transformer (JSON ↔ Model)
class IssueSerializer(serializers.ModelSerializer):
    class Meta:
        model = Issue
        fields = ['id', 'title']

# View - API endpoint (like Express route handler)
class IssueViewSet(viewsets.ModelViewSet):
    queryset = Issue.objects.all()
    serializer_class = IssueSerializer

    def list(self, request):
        # GET /api/issues/
        issues = self.queryset.all()
        serializer = self.serializer_class(issues, many=True)
        return Response(serializer.data)
```

### 💡 Aha Moment

**React components** receive data and render UI.
**Django views** receive requests and return data.

Both are just functions that transform inputs into outputs!

---

## Models - Your Database Schema as Code

### What Are Models?

**Models are TypeScript interfaces that create actual database tables.**

### React Developer Translation

```typescript
// TypeScript interface (frontend only - no persistence)
interface User {
  id: string;
  username: string;
  email: string;
  created_at: Date;
}
```

```python
# Django model (backend - creates actual database table)
class User(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4)
    username = models.CharField(max_length=128, unique=True)
    email = models.CharField(max_length=255, unique=True)
    created_at = models.DateTimeField(auto_now_add=True)
```

When you run `python manage.py migrate`, Django creates this SQL:

```sql
CREATE TABLE "db_user" (
    "id" uuid PRIMARY KEY,
    "username" varchar(128) UNIQUE NOT NULL,
    "email" varchar(255) UNIQUE NOT NULL,
    "created_at" timestamp NOT NULL
);
```

### 🧠 Mental Model

```
TypeScript Interface  →  Only exists in code, no persistence
Django Model         →  Exists in code + creates database table + provides API to interact with data
```

### Real Example: User Model

**Location:** `/home/user/plane/apps/api/plane/db/models/user.py`

```python
from django.db import models
from django.contrib.auth.models import AbstractBaseUser
import uuid

class User(AbstractBaseUser):
    # Primary key (like id in TypeScript)
    id = models.UUIDField(
        default=uuid.uuid4,      # Auto-generate UUID
        unique=True,
        editable=False,
        db_index=True,           # Index for fast lookups
        primary_key=True
    )

    # String fields (like TypeScript string)
    username = models.CharField(max_length=128, unique=True)
    email = models.CharField(max_length=255, unique=True, null=True, blank=True)
    display_name = models.CharField(max_length=255, default="")
    first_name = models.CharField(max_length=255, blank=True)
    last_name = models.CharField(max_length=255, blank=True)

    # File/URL fields
    avatar = models.TextField(blank=True)
    cover_image = models.URLField(blank=True, null=True, max_length=800)

    # Timestamps (auto-managed)
    date_joined = models.DateTimeField(auto_now_add=True)  # Set once on creation
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)       # Updates on every save

    # Boolean fields
    is_active = models.BooleanField(default=True)
    is_email_verified = models.BooleanField(default=False)
    is_superuser = models.BooleanField(default=False)

    # Activity tracking
    last_active = models.DateTimeField(default=timezone.now, null=True)
    last_login_time = models.DateTimeField(null=True)
```

**TypeScript Equivalent:**

```typescript
interface User {
  id: string;
  username: string;
  email: string | null;
  display_name: string;
  first_name: string;
  last_name: string;
  avatar: string;
  cover_image: string | null;
  date_joined: Date;
  created_at: Date;
  updated_at: Date;
  is_active: boolean;
  is_email_verified: boolean;
  is_superuser: boolean;
  last_active: Date | null;
  last_login_time: Date | null;
}
```

### Field Types Reference

| Django Field | TypeScript Type | Purpose |
|--------------|-----------------|---------|
| `CharField(max_length=255)` | `string` | Short text (max 255 chars) |
| `TextField()` | `string` | Long text (unlimited) |
| `IntegerField()` | `number` | Integer |
| `FloatField()` | `number` | Decimal number |
| `BooleanField()` | `boolean` | True/False |
| `DateField()` | `Date` | Date only (YYYY-MM-DD) |
| `DateTimeField()` | `Date` | Date + time |
| `UUIDField()` | `string` | UUID |
| `JSONField()` | `object` | JSON object |
| `URLField()` | `string` | URL (validated) |
| `EmailField()` | `string` | Email (validated) |

### Field Options

```python
class Issue(models.Model):
    # Required field
    title = models.CharField(max_length=255)

    # Optional field (null in DB, blank in forms)
    description = models.TextField(null=True, blank=True)

    # Default value
    priority = models.CharField(max_length=30, default="none")

    # Unique constraint
    sequence_id = models.IntegerField(unique=True)

    # Auto-set on creation
    created_at = models.DateTimeField(auto_now_add=True)

    # Auto-update on every save
    updated_at = models.DateTimeField(auto_now=True)

    # Database index for fast lookups
    name = models.CharField(max_length=255, db_index=True)

    # Choices (like TypeScript union types)
    PRIORITY_CHOICES = (
        ("urgent", "Urgent"),
        ("high", "High"),
        ("medium", "Medium"),
        ("low", "Low"),
        ("none", "None"),
    )
    priority = models.CharField(max_length=30, choices=PRIORITY_CHOICES, default="none")
```

**TypeScript Equivalent:**

```typescript
type Priority = "urgent" | "high" | "medium" | "low" | "none";

interface Issue {
  title: string;                    // required
  description?: string | null;      // optional
  priority: Priority;               // union type with default
  sequence_id: number;              // unique
  created_at: Date;                 // auto-set
  updated_at: Date;                 // auto-update
  name: string;                     // indexed
}
```

### Real Example: Issue Model

**Location:** `/home/user/plane/apps/api/plane/db/models/issue.py`

```python
from django.db import models
from django.conf import settings
from .project import ProjectBaseModel

class Issue(ProjectBaseModel):
    PRIORITY_CHOICES = (
        ("urgent", "Urgent"),
        ("high", "High"),
        ("medium", "Medium"),
        ("low", "Low"),
        ("none", "None"),
    )

    # Self-referencing foreign key (parent issue)
    parent = models.ForeignKey(
        "self",                      # Reference to same model
        on_delete=models.CASCADE,    # Delete children when parent deleted
        null=True,
        blank=True,
        related_name="parent_issue"
    )

    # Foreign key to State model
    state = models.ForeignKey(
        "db.State",                  # Reference to State model
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name="state_issue"
    )

    # Basic fields
    name = models.CharField(max_length=255, verbose_name="Issue Name")
    description = models.JSONField(blank=True, default=dict)
    description_html = models.TextField(blank=True, default="<p></p>")

    # Choice field (enum)
    priority = models.CharField(
        max_length=30,
        choices=PRIORITY_CHOICES,
        default="none"
    )

    # Date fields
    start_date = models.DateField(null=True, blank=True)
    target_date = models.DateField(null=True, blank=True)

    # Many-to-Many relationship (an issue can have multiple assignees)
    assignees = models.ManyToManyField(
        settings.AUTH_USER_MODEL,    # Reference to User model
        blank=True,
        related_name="assignee"
    )

    # Integer with validation
    point = models.IntegerField(
        validators=[MinValueValidator(0), MaxValueValidator(12)],
        null=True,
        blank=True
    )
```

**TypeScript Equivalent:**

```typescript
type Priority = "urgent" | "high" | "medium" | "low" | "none";

interface Issue {
  parent: Issue | null;              // Self-reference
  state: State | null;               // Foreign key
  name: string;
  description: object;
  description_html: string;
  priority: Priority;
  start_date: Date | null;
  target_date: Date | null;
  assignees: User[];                 // Many-to-many
  point: number | null;              // 0-12
}
```

### 💡 Aha Moment

**Models are living, breathing TypeScript interfaces!**

- They define the shape of your data (like TypeScript)
- They create database tables (SQL magic)
- They provide methods to query data (`.all()`, `.filter()`, `.get()`)
- They handle relationships (foreign keys, many-to-many)
- They validate data before saving

---

## Views - API Endpoints Like Express Routes

### What Are Views?

**Views are functions that receive HTTP requests and return HTTP responses.**

Think of them as **Express.js route handlers** or **React Router loaders/actions**.

### React Developer Translation

```typescript
// React Router loader (frontend)
export async function loader({ params }: LoaderFunctionArgs) {
  const issues = await fetch(`/api/issues/`).then(r => r.json());
  return json({ issues });
}
```

```javascript
// Express.js route (backend)
app.get('/api/issues/', async (req, res) => {
  const issues = await Issue.find({});
  res.json({ issues });
});
```

```python
# Django view (backend)
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(['GET'])
def issue_list(request):
    issues = Issue.objects.all()
    serializer = IssueSerializer(issues, many=True)
    return Response(serializer.data)
```

### Types of Views

Django has **3 types of views**:

1. **Function-Based Views (FBV)** - Simple functions
2. **Class-Based Views (CBV)** - Reusable classes
3. **ViewSets (DRF)** - REST API superpowers ✅ **CURRENT (Plane uses this)**

### Function-Based Views (Simple)

```python
# apps/api/plane/app/views/custom.py
from rest_framework.decorators import api_view, permission_classes
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated

@api_view(['GET', 'POST'])
@permission_classes([IsAuthenticated])
def issue_list(request, project_id):
    if request.method == 'GET':
        # Handle GET request
        issues = Issue.objects.filter(project_id=project_id)
        serializer = IssueSerializer(issues, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        # Handle POST request
        serializer = IssueSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=201)
        return Response(serializer.errors, status=400)
```

**Express.js Equivalent:**

```javascript
app.get('/api/projects/:projectId/issues', authenticate, async (req, res) => {
  const issues = await Issue.find({ project_id: req.params.projectId });
  res.json(issues);
});

app.post('/api/projects/:projectId/issues', authenticate, async (req, res) => {
  const issue = new Issue(req.body);
  await issue.save();
  res.status(201).json(issue);
});
```

### ViewSets (DRF - Plane's Approach) ✅ CURRENT

**ViewSets are like React Router's resource routes** - they handle all CRUD operations automatically.

**Location:** `/home/user/plane/apps/api/plane/app/views/cycle/issue.py`

```python
from rest_framework import viewsets
from rest_framework.response import Response
from rest_framework.decorators import action

class CycleIssueViewSet(viewsets.ModelViewSet):
    serializer_class = CycleIssueSerializer
    model = CycleIssue

    # Define queryset (like a SQL query)
    def get_queryset(self):
        return self.filter_queryset(
            super()
            .get_queryset()
            .filter(workspace__slug=self.kwargs.get("slug"))
            .filter(project_id=self.kwargs.get("project_id"))
            .filter(cycle_id=self.kwargs.get("cycle_id"))
            .select_related("project", "workspace", "cycle", "issue")
            .prefetch_related("issue__assignees", "issue__labels")
            .distinct()
        )

    # GET /api/workspaces/{slug}/projects/{project_id}/cycles/{cycle_id}/issues/
    def list(self, request, *args, **kwargs):
        issues = self.get_queryset()
        serializer = self.get_serializer(issues, many=True)
        return Response(serializer.data)

    # POST /api/workspaces/{slug}/projects/{project_id}/cycles/{cycle_id}/issues/
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data, status=201)

    # GET /api/.../issues/{pk}/
    def retrieve(self, request, pk=None, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance)
        return Response(serializer.data)

    # PATCH /api/.../issues/{pk}/
    def partial_update(self, request, pk=None, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance, data=request.data, partial=True)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data)

    # DELETE /api/.../issues/{pk}/
    def destroy(self, request, pk=None, *args, **kwargs):
        instance = self.get_object()
        instance.delete()
        return Response(status=204)

    # Custom action: POST /api/.../issues/bulk_create/
    @action(detail=False, methods=['POST'])
    def bulk_create(self, request, *args, **kwargs):
        issues = request.data.get('issues', [])
        serializer = self.get_serializer(data=issues, many=True)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data, status=201)
```

### ViewSet Methods

| ViewSet Method | HTTP Method | URL Pattern | Purpose |
|----------------|-------------|-------------|---------|
| `list()` | GET | `/issues/` | Get all issues |
| `create()` | POST | `/issues/` | Create new issue |
| `retrieve()` | GET | `/issues/{id}/` | Get one issue |
| `update()` | PUT | `/issues/{id}/` | Replace issue |
| `partial_update()` | PATCH | `/issues/{id}/` | Update fields |
| `destroy()` | DELETE | `/issues/{id}/` | Delete issue |
| `@action` | Custom | `/issues/custom/` | Custom endpoint |

### 🧠 Mental Model

```typescript
// React Router (frontend)
<Route path="/issues" element={<IssueList />} loader={loader} action={action} />

// Django ViewSet (backend)
class IssueViewSet(viewsets.ModelViewSet):
    def list():     # GET /issues/
    def create():   # POST /issues/
    def retrieve(): # GET /issues/:id/
    def update():   # PUT /issues/:id/
    def destroy():  # DELETE /issues/:id/
```

Both map URLs to handlers - React for pages, Django for APIs!

### Request Object

```python
def my_view(request):
    # Like Express req object
    request.method          # "GET", "POST", etc.
    request.data            # Request body (parsed JSON)
    request.query_params    # URL query string (?page=1)
    request.user            # Authenticated user
    request.headers         # HTTP headers
    request.FILES           # Uploaded files
```

**React/Fetch Equivalent:**

```typescript
// Frontend
const response = await fetch('/api/issues/', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ title: 'New Issue' })
});

// Backend receives:
// request.method = 'POST'
// request.data = { 'title': 'New Issue' }
// request.headers = { 'content-type': 'application/json' }
```

### Response Object

```python
from rest_framework.response import Response

# Return JSON
return Response({'message': 'Success'})                    # 200 OK
return Response(data, status=201)                          # 201 Created
return Response({'error': 'Not found'}, status=404)        # 404 Not Found
return Response(serializer.errors, status=400)             # 400 Bad Request
```

**TypeScript Equivalent:**

```typescript
// Express.js
res.json({ message: 'Success' });              // 200 OK
res.status(201).json(data);                    // 201 Created
res.status(404).json({ error: 'Not found' }); // 404 Not Found
res.status(400).json(errors);                  // 400 Bad Request

// React Router action
return json({ message: 'Success' });
return json(data, { status: 201 });
return json({ error: 'Not found' }, { status: 404 });
```

---

## Serializers - Data Transformers

### What Are Serializers?

**Serializers convert between Python objects and JSON** (and vice versa).

Think of them as:
- `JSON.stringify()` + `JSON.parse()` combined
- Type validators
- Data transformers

### React Developer Translation

```typescript
// Frontend - manual transformation
interface IssueDTO {
  id: string;
  title: string;
  created_at: string;  // ISO string
}

interface Issue {
  id: string;
  title: string;
  created_at: Date;    // Actual Date object
}

// Transform API response → App state
function deserialize(dto: IssueDTO): Issue {
  return {
    id: dto.id,
    title: dto.title,
    created_at: new Date(dto.created_at)
  };
}

// Transform App state → API request
function serialize(issue: Issue): IssueDTO {
  return {
    id: issue.id,
    title: issue.title,
    created_at: issue.created_at.toISOString()
  };
}
```

```python
# Backend - Django does this automatically
from rest_framework import serializers

class IssueSerializer(serializers.ModelSerializer):
    class Meta:
        model = Issue
        fields = ['id', 'title', 'created_at']

    # Django automatically:
    # - Converts Issue model → JSON (serialize)
    # - Converts JSON → Issue model (deserialize)
    # - Validates data
    # - Handles relationships
```

### Simple Serializer

**Location:** `/home/user/plane/apps/api/plane/app/serializers/issue.py`

```python
from rest_framework import serializers
from plane.db.models import Issue

class IssueFlatSerializer(serializers.ModelSerializer):
    class Meta:
        model = Issue
        fields = [
            "id",
            "name",
            "description",
            "priority",
            "start_date",
            "target_date",
            "sequence_id",
        ]
```

**Usage:**

```python
# Model → JSON (serialize)
issue = Issue.objects.get(id="123")
serializer = IssueFlatSerializer(issue)
print(serializer.data)
# Output: {
#   "id": "123",
#   "name": "Fix bug",
#   "priority": "high",
#   ...
# }

# JSON → Model (deserialize)
data = {"name": "New issue", "priority": "medium"}
serializer = IssueFlatSerializer(data=data)
if serializer.is_valid():
    issue = serializer.save()  # Creates Issue in database
```

### Nested Serializers

```python
from rest_framework import serializers

class UserLiteSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'avatar']

class StateLiteSerializer(serializers.ModelSerializer):
    class Meta:
        model = State
        fields = ['id', 'name', 'color']

class IssueDetailSerializer(serializers.ModelSerializer):
    # Nested serializers (like populating references)
    assignees = UserLiteSerializer(many=True, read_only=True)
    state = StateLiteSerializer(read_only=True)

    # Write-only fields for creating/updating
    assignee_ids = serializers.ListField(
        child=serializers.UUIDField(),
        write_only=True,
        required=False
    )
    state_id = serializers.UUIDField(write_only=True, required=False)

    class Meta:
        model = Issue
        fields = [
            'id', 'name', 'description',
            'assignees', 'assignee_ids',  # Read/write separately
            'state', 'state_id',
            'created_at', 'updated_at'
        ]
```

**API Response (GET):**

```json
{
  "id": "abc-123",
  "name": "Fix login bug",
  "assignees": [
    {"id": "user-1", "username": "alice", "avatar": "..."},
    {"id": "user-2", "username": "bob", "avatar": "..."}
  ],
  "state": {
    "id": "state-1",
    "name": "In Progress",
    "color": "#ff0000"
  },
  "created_at": "2025-11-19T10:00:00Z"
}
```

**API Request (POST):**

```json
{
  "name": "New issue",
  "assignee_ids": ["user-1", "user-2"],
  "state_id": "state-1"
}
```

### 💡 Aha Moment

**Serializers are like React's prop validation + data transformation combined!**

```typescript
// React prop types (validation only)
interface IssueProps {
  id: string;
  title: string;
  assignees: User[];
}

// Django serializer (validation + transformation + persistence)
class IssueSerializer(serializers.ModelSerializer):
    # Validates, transforms, AND saves to database
```

### Validation

```python
class IssueCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = Issue
        fields = '__all__'

    def validate(self, attrs):
        # Custom validation (like form validation in React)
        if attrs.get('start_date') and attrs.get('target_date'):
            if attrs['start_date'] > attrs['target_date']:
                raise serializers.ValidationError(
                    "Start date cannot be after target date"
                )
        return attrs

    def validate_title(self, value):
        # Field-specific validation
        if len(value) < 3:
            raise serializers.ValidationError("Title too short")
        return value
```

**TypeScript Equivalent (Zod):**

```typescript
import { z } from 'zod';

const IssueSchema = z.object({
  title: z.string().min(3, "Title too short"),
  start_date: z.date().optional(),
  target_date: z.date().optional(),
}).refine(data => {
  if (data.start_date && data.target_date) {
    return data.start_date <= data.target_date;
  }
  return true;
}, { message: "Start date cannot be after target date" });
```

---

## Django REST Framework (DRF)

### What is DRF?

**Django REST Framework (DRF)** is like Express.js for Django - it makes building REST APIs easy.

**Without DRF (plain Django):**
```python
from django.http import JsonResponse
import json

def issue_list(request):
    if request.method == 'GET':
        issues = list(Issue.objects.values())
        return JsonResponse({'issues': issues})
    elif request.method == 'POST':
        data = json.loads(request.body)
        issue = Issue.objects.create(**data)
        return JsonResponse({'issue': model_to_dict(issue)})
```

**With DRF:**
```python
from rest_framework import viewsets

class IssueViewSet(viewsets.ModelViewSet):
    queryset = Issue.objects.all()
    serializer_class = IssueSerializer
    # DRF handles GET, POST, PUT, PATCH, DELETE automatically!
```

### DRF Features

1. **Automatic CRUD endpoints** - Define model + serializer, get full API
2. **Authentication** - JWT, session, token auth built-in
3. **Permissions** - Role-based access control
4. **Pagination** - Automatic page/limit handling
5. **Filtering** - Query parameters → database filters
6. **Browsable API** - Web UI for testing (like Postman built-in)
7. **Throttling** - Rate limiting
8. **Versioning** - API version management

### Browsable API ✅ CURRENT

Visit `http://localhost:8000/api/issues/` in your browser:

```
Issues
GET /api/issues/

[
  {
    "id": "abc-123",
    "title": "Fix bug",
    "priority": "high"
  }
]

POST form below:
Title: [_________]
Priority: [high ▼]
[Submit]
```

**This is DRF's killer feature** - you can test APIs directly in the browser!

### Permissions

```python
from rest_framework import permissions

class IssueViewSet(viewsets.ModelViewSet):
    queryset = Issue.objects.all()
    serializer_class = IssueSerializer
    permission_classes = [permissions.IsAuthenticated]

    def get_permissions(self):
        if self.action in ['list', 'retrieve']:
            # Anyone can view
            return [permissions.AllowAny()]
        else:
            # Only authenticated users can create/update/delete
            return [permissions.IsAuthenticated()]
```

**Plane's Custom Permissions:**

```python
from plane.app.permissions import allow_permission, ROLE

class ProjectIssueViewSet(viewsets.ModelViewSet):
    @allow_permission([ROLE.ADMIN, ROLE.MEMBER])
    def create(self, request, *args, **kwargs):
        # Only admins and members can create issues
        pass

    @allow_permission([ROLE.ADMIN])
    def destroy(self, request, *args, **kwargs):
        # Only admins can delete issues
        pass
```

---

## Request Lifecycle - From Browser to Database

### The Journey of a Request

Let's trace what happens when you create an issue from the React app.

**Step 1: User clicks "Create Issue" in React**

```tsx
// Frontend: apps/web/components/IssueCreateForm.tsx
const handleSubmit = async (data: IssueFormData) => {
  const response = await fetch('/api/workspaces/acme/projects/proj-1/issues/', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    },
    body: JSON.stringify({
      name: data.title,
      description_html: data.description,
      priority: data.priority,
      assignee_ids: data.assignees,
      state_id: data.state
    })
  });

  if (response.ok) {
    const issue = await response.json();
    console.log('Created issue:', issue);
  }
};
```

**Step 2: Request travels to Django**

```
Browser
  ↓ HTTP POST /api/workspaces/acme/projects/proj-1/issues/
  ↓ Headers: { Authorization: Bearer <token>, Content-Type: application/json }
  ↓ Body: { name: "Fix bug", priority: "high", ... }
Django Server
```

**Step 3: Django URL Router**

```python
# apps/api/plane/urls.py
urlpatterns = [
    path('api/', include('plane.app.urls')),
]

# apps/api/plane/app/urls.py
urlpatterns = [
    path('workspaces/<slug>/projects/<project_id>/issues/',
         IssueViewSet.as_view({'post': 'create'})),
]
```

Django matches URL and routes to `IssueViewSet.create()`.

**Step 4: Authentication Middleware**

```python
# Middleware checks Authorization header
if 'Authorization' in request.headers:
    token = request.headers['Authorization'].replace('Bearer ', '')
    user = verify_jwt_token(token)  # Decode JWT
    request.user = user
else:
    request.user = AnonymousUser()
```

**Step 5: Permission Check**

```python
class IssueViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated, ProjectMemberPermission]

    def create(self, request, *args, **kwargs):
        # DRF checks permissions before calling this method
        # If user not authenticated → 401 Unauthorized
        # If user not project member → 403 Forbidden
```

**Step 6: View Method (create)**

```python
def create(self, request, *args, **kwargs):
    # request.data = { name: "Fix bug", priority: "high", ... }

    # 1. Initialize serializer with request data
    serializer = self.get_serializer(data=request.data)

    # 2. Validate data
    serializer.is_valid(raise_exception=True)

    # 3. Save to database
    self.perform_create(serializer)

    # 4. Return response
    return Response(serializer.data, status=201)
```

**Step 7: Serializer Validation**

```python
class IssueCreateSerializer(serializers.ModelSerializer):
    def validate(self, attrs):
        # Check start_date < target_date
        if attrs.get('start_date') > attrs.get('target_date'):
            raise serializers.ValidationError("Invalid dates")

        # Validate assignees are project members
        if attrs.get('assignee_ids'):
            valid_assignees = ProjectMember.objects.filter(
                project_id=self.context['project_id'],
                member_id__in=attrs['assignee_ids']
            ).values_list('member_id', flat=True)
            attrs['assignee_ids'] = list(valid_assignees)

        return attrs
```

**Step 8: Save to Database**

```python
def perform_create(self, serializer):
    # serializer.save() creates Issue in database
    issue = serializer.save(
        project_id=self.kwargs['project_id'],
        workspace_id=self.kwargs['workspace_id'],
        created_by=self.request.user
    )

    # Database INSERT query:
    # INSERT INTO db_issue (id, name, priority, project_id, ...)
    # VALUES ('uuid', 'Fix bug', 'high', 'proj-1', ...)
```

**Step 9: Database Transaction**

```sql
-- PostgreSQL executes:
BEGIN;

INSERT INTO db_issue (
    id, name, description_html, priority,
    project_id, workspace_id, created_by_id,
    created_at, updated_at
) VALUES (
    gen_random_uuid(),
    'Fix bug',
    '<p>Description</p>',
    'high',
    'proj-1',
    'ws-1',
    'user-123',
    NOW(),
    NOW()
);

-- If assignees provided, create many-to-many relationships
INSERT INTO issue_assignees (issue_id, user_id)
VALUES ('issue-abc', 'user-1'), ('issue-abc', 'user-2');

COMMIT;
```

**Step 10: Serializer Response**

```python
# Serializer converts saved Issue back to JSON
serializer.data
# Output:
{
    "id": "issue-abc",
    "name": "Fix bug",
    "description_html": "<p>Description</p>",
    "priority": "high",
    "assignees": [
        {"id": "user-1", "username": "alice"},
        {"id": "user-2", "username": "bob"}
    ],
    "created_at": "2025-11-19T10:30:00Z",
    "updated_at": "2025-11-19T10:30:00Z"
}
```

**Step 11: HTTP Response**

```python
return Response(serializer.data, status=201)

# HTTP response:
# HTTP/1.1 201 Created
# Content-Type: application/json
#
# {
#   "id": "issue-abc",
#   "name": "Fix bug",
#   ...
# }
```

**Step 12: React receives response**

```tsx
const response = await fetch('/api/workspaces/acme/projects/proj-1/issues/', {
  method: 'POST',
  body: JSON.stringify(data)
});

if (response.ok) {
  const issue = await response.json();
  // issue = { id: "issue-abc", name: "Fix bug", ... }

  // Update React state
  setIssues(prev => [...prev, issue]);

  // Show success toast
  toast.success('Issue created!');

  // Navigate to issue detail page
  navigate(`/issues/${issue.id}`);
}
```

### 🧠 Mental Model

```
User Action (React)
  ↓
HTTP Request (JSON)
  ↓
Django URL Router
  ↓
Authentication Middleware (verify JWT)
  ↓
Permission Check (is user allowed?)
  ↓
View Method (business logic)
  ↓
Serializer Validation (validate data)
  ↓
Database Save (PostgreSQL INSERT)
  ↓
Serializer Response (model → JSON)
  ↓
HTTP Response (JSON)
  ↓
React State Update (re-render UI)
```

**Total time: 50-200ms** ⚡

---

## Authentication & Permissions

### JWT Authentication ✅ CURRENT

**Plane uses JWT (JSON Web Tokens)** for authentication.

### How It Works

**Step 1: User logs in**

```tsx
// Frontend
const login = async (email: string, password: string) => {
  const response = await fetch('/api/auth/login/', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password })
  });

  const data = await response.json();
  // data = {
  //   access_token: "eyJhbGciOiJIUzI1...",
  //   refresh_token: "eyJhbGciOiJIUzI1...",
  //   user: { id: "...", email: "...", ... }
  // }

  // Store token
  localStorage.setItem('access_token', data.access_token);
  localStorage.setItem('refresh_token', data.refresh_token);
};
```

**Step 2: Include token in requests**

```tsx
// Frontend
const fetchIssues = async () => {
  const token = localStorage.getItem('access_token');

  const response = await fetch('/api/issues/', {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });

  return response.json();
};
```

**Step 3: Django verifies token**

```python
# Backend (automatic via DRF)
from rest_framework_simplejwt.authentication import JWTAuthentication

class IssueViewSet(viewsets.ModelViewSet):
    authentication_classes = [JWTAuthentication]

    def list(self, request):
        # DRF automatically:
        # 1. Extracts token from Authorization header
        # 2. Verifies token signature
        # 3. Decodes token payload
        # 4. Sets request.user to User object

        user = request.user  # Authenticated User instance
        issues = Issue.objects.filter(workspace__members=user)
        ...
```

### JWT Token Structure

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoiMTIzIiwiZXhwIjoxNjM...
│                                      │                                    │
│        Header (algorithm)            │      Payload (user data)           │  Signature
│                                      │                                    │
└─ {"alg":"HS256","typ":"JWT"}        └─ {"user_id":"123","exp":1637...}  └─ Hash
```

**Decoded Payload:**

```json
{
  "user_id": "abc-123",
  "email": "alice@example.com",
  "exp": 1637856000,  // Expiration timestamp
  "iat": 1637769600,  // Issued at timestamp
  "type": "access"
}
```

### Permission System

**Plane's custom permissions:**

```python
from plane.app.permissions import allow_permission, ROLE

class ProjectViewSet(viewsets.ModelViewSet):
    @allow_permission([ROLE.ADMIN])
    def destroy(self, request, *args, **kwargs):
        # Only admins can delete projects
        project = self.get_object()
        project.delete()
        return Response(status=204)

    @allow_permission([ROLE.ADMIN, ROLE.MEMBER])
    def update(self, request, *args, **kwargs):
        # Admins and members can update projects
        pass

    @allow_permission([ROLE.ADMIN, ROLE.MEMBER, ROLE.VIEWER])
    def retrieve(self, request, *args, **kwargs):
        # Everyone can view projects
        pass
```

**Role Hierarchy:**

```python
ROLE = {
    'ADMIN': 20,     # Full access
    'MEMBER': 15,    # Read/write
    'VIEWER': 10,    # Read-only
    'GUEST': 5       # Limited read
}
```

### Checking Permissions in Views

```python
def get_queryset(self):
    # Only return issues user has access to
    return Issue.objects.filter(
        project__project_projectmember__member=self.request.user,
        project__project_projectmember__is_active=True
    )

def perform_create(self, serializer):
    # Check if user has permission to create in this project
    project = Project.objects.get(id=self.kwargs['project_id'])
    member = ProjectMember.objects.get(
        project=project,
        member=self.request.user
    )

    if member.role < ROLE.MEMBER:
        raise PermissionDenied("You don't have permission to create issues")

    serializer.save(created_by=self.request.user)
```

### 💡 Aha Moment

**JWT tokens are like session cookies, but stateless!**

```
Session Auth (Old Way):
  User logs in → Server stores session in database
  Every request → Server looks up session in database
  Pros: Can revoke immediately
  Cons: Database lookup on every request

JWT Auth (Modern Way):
  User logs in → Server creates signed token
  Every request → Server verifies signature (no DB lookup)
  Pros: Fast, scalable, stateless
  Cons: Can't revoke until expiration (use short expiry + refresh tokens)
```

---

## Database Relationships

### Types of Relationships

Django supports 3 types of relationships:

1. **ForeignKey (Many-to-One)** - Like TypeScript reference
2. **ManyToManyField (Many-to-Many)** - Like array of references
3. **OneToOneField (One-to-One)** - Like unique reference

### ForeignKey (Many-to-One)

**"Many issues belong to one project"**

```python
class Issue(models.Model):
    project = models.ForeignKey(
        Project,
        on_delete=models.CASCADE,  # Delete issues when project deleted
        related_name='issues'      # Access from other side: project.issues.all()
    )
```

**TypeScript Equivalent:**

```typescript
interface Issue {
  id: string;
  project: Project;  // Reference to one project
}

interface Project {
  id: string;
  issues: Issue[];   // Reverse relationship
}
```

**Database Schema:**

```sql
CREATE TABLE issues (
    id UUID PRIMARY KEY,
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE
);

-- One project can have many issues
-- One issue belongs to one project
```

**Usage:**

```python
# Get issue's project
issue = Issue.objects.get(id='123')
project = issue.project
print(project.name)

# Get project's issues (reverse relationship)
project = Project.objects.get(id='456')
issues = project.issues.all()  # Related name: 'issues'
for issue in issues:
    print(issue.name)
```

### ManyToManyField

**"Many issues can have many assignees, and many assignees can have many issues"**

```python
class Issue(models.Model):
    assignees = models.ManyToManyField(
        User,
        blank=True,
        related_name='assigned_issues'
    )
```

**TypeScript Equivalent:**

```typescript
interface Issue {
  id: string;
  assignees: User[];  // Array of users
}

interface User {
  id: string;
  assigned_issues: Issue[];  // Reverse relationship
}
```

**Database Schema:**

```sql
-- Django creates a "through" table automatically
CREATE TABLE issue_assignees (
    id SERIAL PRIMARY KEY,
    issue_id UUID REFERENCES issues(id),
    user_id UUID REFERENCES users(id),
    UNIQUE(issue_id, user_id)
);
```

**Usage:**

```python
# Add assignees to issue
issue = Issue.objects.get(id='123')
user1 = User.objects.get(id='user-1')
user2 = User.objects.get(id='user-2')

issue.assignees.add(user1, user2)

# Get all assignees
assignees = issue.assignees.all()

# Check if user is assigned
if user1 in issue.assignees.all():
    print("User is assigned")

# Remove assignee
issue.assignees.remove(user1)

# Clear all assignees
issue.assignees.clear()

# Reverse: get all issues assigned to user
user_issues = user1.assigned_issues.all()
```

### Real Example: Issue Relationships

```python
class Issue(ProjectBaseModel):
    # ForeignKey (Many-to-One)
    project = models.ForeignKey(
        Project,
        on_delete=models.CASCADE,
        related_name='project_issue'
    )

    state = models.ForeignKey(
        State,
        on_delete=models.CASCADE,
        related_name='state_issue'
    )

    parent = models.ForeignKey(
        'self',  # Self-referencing (sub-issues)
        on_delete=models.CASCADE,
        null=True,
        related_name='parent_issue'
    )

    # ManyToManyField
    assignees = models.ManyToManyField(
        User,
        related_name='assignee'
    )

    labels = models.ManyToManyField(
        Label,
        related_name='labels'
    )
```

**Relationship Diagram:**

```
Project (1) ──┬─→ Issue (many)
              │
State (1) ────┼─→ Issue (many)
              │
Issue (1) ────┼─→ Issue (many)  [parent/children]
              │
User (many) ←─┴─→ Issue (many)  [assignees]
Label (many) ←──→ Issue (many)  [labels]
```

### on_delete Options

```python
class Issue(models.Model):
    # CASCADE: Delete issues when project deleted
    project = models.ForeignKey(Project, on_delete=models.CASCADE)

    # SET_NULL: Set to NULL when user deleted
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)

    # PROTECT: Prevent deletion if issues exist
    state = models.ForeignKey(State, on_delete=models.PROTECT)

    # SET_DEFAULT: Set to default value
    priority = models.ForeignKey(Priority, on_delete=models.SET_DEFAULT, default='none')
```

---

## Querysets - SQL Made Easy

### What Are Querysets?

**Querysets are lazy SQL query builders** - they don't hit the database until you need the data.

Think of them as **SQL wrapped in Python**.

### React Developer Translation

```typescript
// Frontend - filtering in JavaScript
const issues = [
  { id: 1, title: "Bug", priority: "high", assignee: "alice" },
  { id: 2, title: "Feature", priority: "low", assignee: "bob" },
  { id: 3, title: "Fix", priority: "high", assignee: "alice" }
];

// Filter in memory (INEFFICIENT for large datasets)
const highPriorityIssues = issues.filter(i => i.priority === 'high');
const aliceIssues = issues.filter(i => i.assignee === 'alice');
const sortedIssues = [...issues].sort((a, b) =>
  a.title.localeCompare(b.title)
);
```

```python
# Backend - filtering in database (EFFICIENT)
issues = Issue.objects.all()  # Returns queryset (NOT executed yet)

# Filter (adds WHERE clause)
high_priority = issues.filter(priority='high')  # Still not executed

# Multiple filters (chaining)
alice_high_priority = (
    issues
    .filter(priority='high')
    .filter(assignees__username='alice')
)  # Still lazy!

# Order (adds ORDER BY clause)
sorted_issues = issues.order_by('name')

# Only when you iterate, it executes the SQL query
for issue in high_priority:
    print(issue.name)  # NOW it hits database
```

### Basic Queryset Methods

```python
# Get all objects
Issue.objects.all()
# SQL: SELECT * FROM issues

# Filter (WHERE clause)
Issue.objects.filter(priority='high')
# SQL: SELECT * FROM issues WHERE priority = 'high'

# Exclude (WHERE NOT)
Issue.objects.exclude(priority='low')
# SQL: SELECT * FROM issues WHERE priority != 'low'

# Get single object (raises error if not found or multiple)
Issue.objects.get(id='123')
# SQL: SELECT * FROM issues WHERE id = '123' LIMIT 1

# Filter + Get first
Issue.objects.filter(priority='high').first()
# SQL: SELECT * FROM issues WHERE priority = 'high' LIMIT 1

# Count
Issue.objects.filter(priority='high').count()
# SQL: SELECT COUNT(*) FROM issues WHERE priority = 'high'

# Exists (returns bool)
Issue.objects.filter(name='Bug').exists()
# SQL: SELECT 1 FROM issues WHERE name = 'Bug' LIMIT 1

# Order by
Issue.objects.order_by('created_at')  # Ascending
Issue.objects.order_by('-created_at')  # Descending (minus sign)
# SQL: SELECT * FROM issues ORDER BY created_at DESC

# Limit (slicing)
Issue.objects.all()[:10]  # First 10
# SQL: SELECT * FROM issues LIMIT 10

Issue.objects.all()[10:20]  # Next 10
# SQL: SELECT * FROM issues LIMIT 10 OFFSET 10
```

### Advanced Filtering

```python
from django.db.models import Q

# OR queries
Issue.objects.filter(Q(priority='high') | Q(priority='urgent'))
# SQL: WHERE priority = 'high' OR priority = 'urgent'

# Complex queries
Issue.objects.filter(
    Q(priority='high') & (Q(state__name='In Progress') | Q(state__name='Todo'))
)
# SQL: WHERE priority = 'high' AND (state.name = 'In Progress' OR state.name = 'Todo')

# NOT queries
Issue.objects.filter(~Q(priority='low'))
# SQL: WHERE NOT priority = 'low'

# Field lookups
Issue.objects.filter(name__icontains='bug')  # Case-insensitive LIKE
# SQL: WHERE name ILIKE '%bug%'

Issue.objects.filter(created_at__gte='2025-01-01')  # Greater than or equal
# SQL: WHERE created_at >= '2025-01-01'

Issue.objects.filter(priority__in=['high', 'urgent'])  # IN
# SQL: WHERE priority IN ('high', 'urgent')

Issue.objects.filter(assignees__isnull=True)  # IS NULL
# SQL: WHERE assignees IS NULL
```

### Relationship Queries

```python
# Filter by related field (JOIN)
Issue.objects.filter(project__name='Plane')
# SQL: SELECT * FROM issues
#      JOIN projects ON issues.project_id = projects.id
#      WHERE projects.name = 'Plane'

# Filter by many-to-many relationship
Issue.objects.filter(assignees__username='alice')
# SQL: SELECT * FROM issues
#      JOIN issue_assignees ON issues.id = issue_assignees.issue_id
#      JOIN users ON issue_assignees.user_id = users.id
#      WHERE users.username = 'alice'

# Reverse relationship
project = Project.objects.get(id='123')
project.issues.filter(priority='high')
# SQL: SELECT * FROM issues
#      WHERE project_id = '123' AND priority = 'high'
```

### Select Related & Prefetch Related (N+1 Query Prevention)

**Problem (N+1 queries):**

```python
# This causes N+1 queries!
issues = Issue.objects.all()  # 1 query
for issue in issues:
    print(issue.project.name)  # N queries (one per issue)
```

**Solution 1: select_related (for ForeignKey)**

```python
# Only 1 query with JOIN
issues = Issue.objects.select_related('project', 'state').all()
for issue in issues:
    print(issue.project.name)  # No extra query!
    print(issue.state.name)     # No extra query!

# SQL: SELECT * FROM issues
#      JOIN projects ON issues.project_id = projects.id
#      JOIN states ON issues.state_id = states.id
```

**Solution 2: prefetch_related (for ManyToMany)**

```python
# 2 queries total (1 for issues, 1 for all assignees)
issues = Issue.objects.prefetch_related('assignees').all()
for issue in issues:
    for assignee in issue.assignees.all():  # No extra queries!
        print(assignee.username)

# SQL: SELECT * FROM issues
#      SELECT * FROM users WHERE id IN (...)
```

### Real Example from Plane

**Location:** `/home/user/plane/apps/api/plane/app/views/cycle/issue.py`

```python
def get_queryset(self):
    return self.filter_queryset(
        super()
        .get_queryset()
        # Filter by URL parameters
        .filter(workspace__slug=self.kwargs.get("slug"))
        .filter(project_id=self.kwargs.get("project_id"))
        .filter(cycle_id=self.kwargs.get("cycle_id"))

        # Optimize queries with JOINs
        .select_related("project")
        .select_related("workspace")
        .select_related("cycle")
        .select_related("issue", "issue__state", "issue__project")

        # Optimize many-to-many queries
        .prefetch_related("issue__assignees", "issue__labels")

        # Remove duplicates
        .distinct()
    )
```

**This generates ONE optimized SQL query instead of hundreds!**

### 💡 Aha Moment

**Querysets are like React Query or SWR, but for the backend!**

```typescript
// Frontend (React Query)
const { data: issues } = useQuery(['issues'], async () => {
  const response = await fetch('/api/issues/');
  return response.json();
});

// Backend (Django Queryset)
issues = Issue.objects.all()  # Lazy - only executes when needed
```

Both are **lazy** and **cached** - they only fetch when necessary!

---

## Celery - Background Jobs

### What is Celery?

**Celery is a task queue** for running slow operations in the background.

Think of it as **setTimeout on steroids** - but distributed, reliable, and persistent.

### Why Background Jobs?

Some operations are too slow for HTTP requests:

- ❌ Sending emails (2-5 seconds)
- ❌ Generating reports (10-60 seconds)
- ❌ Processing images (5-30 seconds)
- ❌ Importing data from CSV (minutes to hours)

**HTTP requests should respond in < 500ms!**

### React Developer Translation

```typescript
// Frontend - blocking (BAD)
const sendEmail = async () => {
  setLoading(true);
  await fetch('/api/send-email/');  // User waits 5 seconds
  setLoading(false);
};

// Frontend - non-blocking (GOOD)
const sendEmail = async () => {
  // Triggers background job, returns immediately
  await fetch('/api/send-email/', { method: 'POST' });
  toast.success('Email queued!');  // Returns in 50ms
};
```

### How Celery Works

```
Django View
  ↓
Queue Task (Redis) ─→ Return response immediately
  ↓
Celery Worker (separate process)
  ↓
Execute task in background
  ↓
Update database / send notification
```

### Defining a Task

```python
from celery import shared_task
from plane.db.models import Issue, IssueActivity

@shared_task
def issue_activity(issue_id, action, user_id):
    """
    Background task to log issue activity.
    Runs asynchronously in Celery worker.
    """
    issue = Issue.objects.get(id=issue_id)
    user = User.objects.get(id=user_id)

    IssueActivity.objects.create(
        issue=issue,
        actor=user,
        action=action,
        created_at=timezone.now()
    )

    # Send notification (slow operation)
    send_notification_to_assignees(issue, action)
```

### Calling a Task

```python
from plane.bgtasks.issue_activities_task import issue_activity

class IssueViewSet(viewsets.ModelViewSet):
    def update(self, request, pk=None):
        issue = self.get_object()
        serializer = self.get_serializer(issue, data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()

        # Queue background task (returns immediately)
        issue_activity.delay(
            issue_id=issue.id,
            action='updated',
            user_id=request.user.id
        )

        # Return response without waiting for task
        return Response(serializer.data)
```

**Without Celery (blocking):**
```python
def update(self, request, pk=None):
    issue = self.get_object()
    serializer.save()

    # This blocks for 2-3 seconds
    IssueActivity.objects.create(...)
    send_notification_to_assignees(...)  # Slow!

    return Response(serializer.data)  # User waits 3 seconds
```

**With Celery (non-blocking):**
```python
def update(self, request, pk=None):
    issue = self.get_object()
    serializer.save()

    # Queue task - returns in microseconds
    issue_activity.delay(issue.id, 'updated', request.user.id)

    return Response(serializer.data)  # User gets response in 50ms
```

### 🧠 Mental Model

```typescript
// Frontend - Promise.all (runs in browser)
Promise.all([
  fetchIssues(),
  fetchProjects(),
  fetchUsers()
]);

// Backend - Celery tasks (runs in worker processes)
celery.group([
  send_email.delay(user_id),
  generate_report.delay(project_id),
  process_image.delay(file_id)
]);
```

---

## Real Code Examples from Plane

### Example 1: Issue Model

**Location:** `/home/user/plane/apps/api/plane/db/models/issue.py`

```python
class Issue(ProjectBaseModel):
    """
    Issue model - represents a work item in a project.

    Inherits from ProjectBaseModel which provides:
    - id (UUID)
    - project (ForeignKey)
    - workspace (ForeignKey)
    - created_at, updated_at
    - created_by, updated_by
    """

    # Priority choices (like TypeScript union type)
    PRIORITY_CHOICES = (
        ("urgent", "Urgent"),
        ("high", "High"),
        ("medium", "Medium"),
        ("low", "Low"),
        ("none", "None"),
    )

    # Relationships
    parent = models.ForeignKey("self", on_delete=models.CASCADE, null=True, blank=True)
    state = models.ForeignKey("db.State", on_delete=models.CASCADE, null=True)
    assignees = models.ManyToManyField(User, blank=True, related_name="assignee")

    # Basic fields
    name = models.CharField(max_length=255)
    description = models.JSONField(blank=True, default=dict)
    description_html = models.TextField(blank=True, default="<p></p>")
    priority = models.CharField(max_length=30, choices=PRIORITY_CHOICES, default="none")

    # Dates
    start_date = models.DateField(null=True, blank=True)
    target_date = models.DateField(null=True, blank=True)

    # Metadata
    sequence_id = models.IntegerField(default=1)
    sort_order = models.FloatField(default=65535)
    is_draft = models.BooleanField(default=False)

    class Meta:
        db_table = "db_issue"
        verbose_name = "Issue"
        verbose_name_plural = "Issues"
        ordering = ["-created_at"]
```

### Example 2: Issue Serializer

**Location:** `/home/user/plane/apps/api/plane/app/serializers/issue.py`

```python
class IssueCreateSerializer(BaseSerializer):
    """
    Serializer for creating/updating issues.
    Handles nested relationships (assignees, labels).
    """

    # Write-only fields (for POST/PATCH requests)
    state_id = serializers.PrimaryKeyRelatedField(
        source="state",
        queryset=State.objects.all(),
        required=False,
        allow_null=True
    )

    assignee_ids = serializers.ListField(
        child=serializers.PrimaryKeyRelatedField(queryset=User.objects.all()),
        write_only=True,
        required=False,
    )

    label_ids = serializers.ListField(
        child=serializers.PrimaryKeyRelatedField(queryset=Label.objects.all()),
        write_only=True,
        required=False,
    )

    # Read-only fields (for GET responses)
    project_id = serializers.UUIDField(source="project.id", read_only=True)
    workspace_id = serializers.UUIDField(source="workspace.id", read_only=True)

    class Meta:
        model = Issue
        fields = "__all__"
        read_only_fields = ["workspace", "project", "created_by", "updated_by"]

    def validate(self, attrs):
        """Custom validation logic"""
        # Validate date range
        if attrs.get("start_date") and attrs.get("target_date"):
            if attrs["start_date"] > attrs["target_date"]:
                raise serializers.ValidationError(
                    "Start date cannot exceed target date"
                )

        # Validate HTML content for security
        if "description_html" in attrs and attrs["description_html"]:
            is_valid, error_msg, sanitized = validate_html_content(attrs["description_html"])
            if not is_valid:
                raise serializers.ValidationError({"error": "Invalid HTML content"})
            attrs["description_html"] = sanitized

        # Validate assignees are project members
        if attrs.get("assignee_ids"):
            valid_assignees = ProjectMember.objects.filter(
                project_id=self.context["project_id"],
                member_id__in=attrs["assignee_ids"],
                is_active=True
            ).values_list("member_id", flat=True)
            attrs["assignee_ids"] = list(valid_assignees)

        return attrs

    def create(self, validated_data):
        """Create issue with relationships"""
        assignee_ids = validated_data.pop("assignee_ids", [])
        label_ids = validated_data.pop("label_ids", [])

        # Create issue
        issue = Issue.objects.create(**validated_data)

        # Set many-to-many relationships
        if assignee_ids:
            issue.assignees.set(assignee_ids)
        if label_ids:
            issue.labels.set(label_ids)

        return issue
```

### Example 3: Issue ViewSet

**Location:** `/home/user/plane/apps/api/plane/app/views/cycle/issue.py`

```python
class CycleIssueViewSet(BaseViewSet):
    """
    API endpoints for issues within a cycle.

    URLs:
    - GET    /api/workspaces/{slug}/projects/{project_id}/cycles/{cycle_id}/issues/
    - POST   /api/workspaces/{slug}/projects/{project_id}/cycles/{cycle_id}/issues/
    - GET    /api/.../issues/{pk}/
    - PATCH  /api/.../issues/{pk}/
    - DELETE /api/.../issues/{pk}/
    """

    serializer_class = CycleIssueSerializer
    model = CycleIssue

    def get_queryset(self):
        """
        Optimize query with select_related and prefetch_related.
        Filter by workspace, project, and cycle from URL parameters.
        """
        return self.filter_queryset(
            super()
            .get_queryset()
            # Filter by URL params
            .filter(workspace__slug=self.kwargs.get("slug"))
            .filter(project_id=self.kwargs.get("project_id"))
            .filter(cycle_id=self.kwargs.get("cycle_id"))
            # Filter by user permissions
            .filter(
                project__project_projectmember__member=self.request.user,
                project__project_projectmember__is_active=True,
            )
            # Optimize queries
            .select_related("project", "workspace", "cycle")
            .select_related("issue", "issue__state", "issue__project")
            .prefetch_related("issue__assignees", "issue__labels")
            .distinct()
        )

    def list(self, request, *args, **kwargs):
        """GET /issues/ - List all issues in cycle"""
        queryset = self.get_queryset()

        # Apply filters from query params
        filters = request.query_params
        if filters.get("priority"):
            queryset = queryset.filter(issue__priority=filters["priority"])

        # Paginate
        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)

    def create(self, request, *args, **kwargs):
        """POST /issues/ - Create new issue in cycle"""
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)

        # Save issue
        issue = serializer.save()

        # Queue background task for activity logging
        issue_activity.delay(
            issue_id=str(issue.id),
            action="created",
            user_id=str(request.user.id)
        )

        return Response(serializer.data, status=201)
```

---

## Common Patterns

### Pattern 1: Soft Delete

**Don't actually delete data - mark as deleted instead.**

```python
class SoftDeleteModel(models.Model):
    deleted_at = models.DateTimeField(null=True, blank=True)

    class Meta:
        abstract = True

    def delete(self, *args, **kwargs):
        """Override delete to soft delete"""
        self.deleted_at = timezone.now()
        self.save()

    def hard_delete(self):
        """Actually delete from database"""
        super().delete()

class Issue(SoftDeleteModel):
    name = models.CharField(max_length=255)
    # Inherits deleted_at field

# Usage
issue.delete()  # Sets deleted_at, keeps in database
issue.hard_delete()  # Actually removes from database

# Filter out deleted issues
Issue.objects.filter(deleted_at__isnull=True)
```

### Pattern 2: Audit Fields

**Track who created/updated and when.**

```python
class AuditModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, related_name='+')
    updated_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, related_name='+')

    class Meta:
        abstract = True

# Every model inherits these fields
class Issue(AuditModel):
    name = models.CharField(max_length=255)
```

### Pattern 3: Manager for Common Queries

```python
class IssueManager(models.Manager):
    def active(self):
        """Get only active (non-deleted, non-archived) issues"""
        return self.filter(
            deleted_at__isnull=True,
            archived_at__isnull=False,
            is_draft=False
        )

    def high_priority(self):
        """Get high priority issues"""
        return self.filter(priority__in=['high', 'urgent'])

class Issue(models.Model):
    # ... fields ...
    objects = IssueManager()

# Usage
Issue.objects.active()  # Only active issues
Issue.objects.high_priority()  # Only high/urgent
Issue.objects.active().high_priority()  # Chainable!
```

### Pattern 4: Signals (Like useEffect for Models)

```python
from django.db.models.signals import post_save, pre_delete
from django.dispatch import receiver

@receiver(post_save, sender=Issue)
def issue_created(sender, instance, created, **kwargs):
    """Run after issue is saved"""
    if created:
        # New issue created
        send_notification(f"New issue: {instance.name}")
    else:
        # Existing issue updated
        send_notification(f"Issue updated: {instance.name}")

@receiver(pre_delete, sender=Issue)
def issue_deleted(sender, instance, **kwargs):
    """Run before issue is deleted"""
    log_deletion(instance)
```

**React useEffect Equivalent:**

```typescript
useEffect(() => {
  // Run after issue changes
  if (issue) {
    sendNotification(`Issue: ${issue.name}`);
  }
}, [issue]);
```

---

## Debugging Django

### Django Debug Toolbar ✅ CURRENT

**Visual debugger for Django apps** - shows SQL queries, templates, cache, signals, etc.

**Enable it:**

```python
# settings.py
INSTALLED_APPS = [
    'debug_toolbar',
    ...
]

MIDDLEWARE = [
    'debug_toolbar.middleware.DebugToolbarMiddleware',
    ...
]
```

**Open app in browser, see toolbar on right side:**

```
SQL Queries (23 queries in 45ms)
  ↓
SELECT * FROM issues WHERE project_id = '123'  [5ms]
SELECT * FROM users WHERE id IN (...)          [3ms]
SELECT * FROM states WHERE ...                 [2ms]

Tips: Use select_related to reduce queries!
```

### Python Debugger (pdb)

```python
def my_view(request):
    import pdb; pdb.set_trace()  # Breakpoint

    issues = Issue.objects.all()
    # Shell opens here, you can inspect variables:
    # >>> issues
    # >>> issues.count()
    # >>> issues.first().name
```

### Logging

```python
import logging
logger = logging.getLogger(__name__)

def my_view(request):
    logger.info(f"User {request.user} accessed issues")
    logger.debug(f"Query params: {request.query_params}")

    try:
        issue = Issue.objects.get(id='123')
    except Issue.DoesNotExist:
        logger.error("Issue not found", exc_info=True)
        return Response({'error': 'Not found'}, status=404)
```

### Django Shell

```bash
# Start Django shell
python manage.py shell

# Interactive Python with Django loaded
>>> from plane.db.models import Issue, User
>>> issues = Issue.objects.all()
>>> issues.count()
42
>>> issue = issues.first()
>>> issue.name
"Fix bug"
>>> issue.assignees.all()
[<User: alice>, <User: bob>]
```

**Like Node.js REPL but with your entire Django app loaded!**

---

## Next Steps

### You've Learned Django Fundamentals! 🎉

You now understand:
- ✅ Models (database schema as code)
- ✅ Views (API endpoints)
- ✅ Serializers (data transformers)
- ✅ Querysets (SQL made easy)
- ✅ Authentication (JWT tokens)
- ✅ Relationships (ForeignKey, ManyToMany)
- ✅ Background jobs (Celery)

### Continue Learning

1. **[DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md)** - Deep dive into PostgreSQL, migrations, and schema design
2. **[INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md)** - How React and Django work together
3. **[HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)** - Step-by-step recipes for common tasks
4. **[CODE_TOURS.md](./CODE_TOURS.md)** - Follow real features through the codebase
5. **[TESTING_GUIDE.md](./TESTING_GUIDE.md)** - Test Django models, views, and APIs

### Practice Exercises

1. **Read an existing model** - Pick a model in `/apps/api/plane/db/models/` and understand it
2. **Trace a request** - Follow an API call from React → Django → Database → Response
3. **Create a simple model** - Design a new feature with models, serializers, and views
4. **Query the database** - Use Django shell to practice querysets
5. **Add a field** - Add a field to an existing model and create a migration

### Resources

- **Django Docs:** https://docs.djangoproject.com/
- **DRF Docs:** https://www.django-rest-framework.org/
- **Plane Codebase:** `/home/user/plane/apps/api/plane/`

---

**You're now ready to read and understand Plane's Django backend! 🚀**

**Next:** [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md) - Learn how data is structured in PostgreSQL.
