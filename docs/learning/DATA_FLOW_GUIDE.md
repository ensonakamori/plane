# Data Flow Guide - Request to Response Lifecycle

**Follow the Data!** This guide traces how data flows through Plane from a user click to a database update and back to the UI.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 50-60 minutes
**📊 Difficulty Level:** Intermediate
**🎯 Target Audience:** Developers who want to understand system-wide data flow

---

## Table of Contents

1. [The Complete Journey](#the-complete-journey)
2. [Frontend Data Flow](#frontend-data-flow)
3. [Backend Data Flow](#backend-data-flow)
4. [Database Operations](#database-operations)
5. [Real-Time Updates](#real-time-updates)
6. [Caching Strategy](#caching-strategy)
7. [Error Handling](#error-handling)
8. [Performance Optimizations](#performance-optimizations)
9. [Next Steps](#next-steps)

---

## The Complete Journey

### 🧠 Mental Model: Restaurant Analogy

Think of data flow like ordering food at a restaurant:

1. **Customer (User)** → Orders food (clicks button)
2. **Waiter (Frontend)** → Takes order to kitchen (sends API request)
3. **Chef (Backend)** → Prepares food (processes request)
4. **Pantry (Database)** → Provides ingredients (fetches/stores data)
5. **Waiter (Frontend)** → Brings food back (displays response)
6. **Customer (User)** → Enjoys meal (sees updated UI)

---

### Example: Creating an Issue

Let's trace a complete example: **User creates a new issue**

```
┌─────────────────────────────────────────────────────────────┐
│ STEP 1: User Interaction (Browser)                          │
└─────────────────────────────────────────────────────────────┘
User clicks "Create Issue" button in React component
           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 2: Event Handler (Frontend)                            │
└─────────────────────────────────────────────────────────────┘
Component calls hook → Hook calls MobX store action
           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 3: Optimistic Update (MobX Store)                      │
└─────────────────────────────────────────────────────────────┘
Store adds temporary issue → UI updates immediately
           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 4: API Request (Service Layer)                         │
└─────────────────────────────────────────────────────────────┘
Service sends HTTP POST → Request goes over network
           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 5: API Receives Request (Django)                       │
└─────────────────────────────────────────────────────────────┘
URL router → View → Permission check → Serializer validation
           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 6: Database Write (PostgreSQL)                         │
└─────────────────────────────────────────────────────────────┘
Django ORM → SQL INSERT → PostgreSQL commits transaction
           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 7: Background Tasks (Celery)                           │
└─────────────────────────────────────────────────────────────┘
Queue notifications → Worker sends emails (async)
           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 8: API Response (Django)                               │
└─────────────────────────────────────────────────────────────┘
Serializer formats response → JSON returned → HTTP 201 Created
           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 9: Update Frontend State (Service Layer)               │
└─────────────────────────────────────────────────────────────┘
Service receives response → Store updates with real data
           ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 10: UI Update (React)                                  │
└─────────────────────────────────────────────────────────────┘
MobX triggers re-render → User sees created issue
```

**Total Time:** ~200-500ms (user perspective: instant!)

---

## Frontend Data Flow

### Phase 1: User Interaction

```typescript
// /apps/web/core/components/issues/create-issue-modal.tsx
import { observer } from "mobx-react";
import { useIssueActions } from "~/hooks/use-issue-actions";

const CreateIssueModal = observer(() => {
  const { createIssue } = useIssueActions();
  const [title, setTitle] = useState("");

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    // STEP 1: Call action from hook
    await createIssue({
      title,
      project_id: "abc123",
      priority: "medium",
    });

    // STEP 2: Close modal
    onClose();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={title}
        onChange={e => setTitle(e.target.value)}
        placeholder="Issue title"
      />
      <button type="submit">Create Issue</button>
    </form>
  );
});
```

**Timeline:**
- **0ms** - User clicks submit
- **0-1ms** - Event handler fires
- **1ms** - Call hook function

---

### Phase 2: Hook Layer

```typescript
// /apps/web/core/hooks/use-issue-actions.tsx
import { useCallback } from "react";
import { useIssueStore } from "~/stores";

export const useIssueActions = () => {
  const issueStore = useIssueStore();

  const createIssue = useCallback(
    async (data: Partial<TIssue>) => {
      // STEP 3: Delegate to store
      return issueStore.createIssue(data);
    },
    [issueStore]
  );

  return { createIssue };
};
```

**Timeline:**
- **1-2ms** - Hook calls store method

**🧠 Mental Model:**
> Hooks are like translators - they translate React patterns (callbacks) into store operations.

---

### Phase 3: MobX Store

```typescript
// /apps/web/core/store/issue/issue.store.ts
import { makeObservable, observable, action, runInAction } from "mobx";
import { IssueService } from "@plane/services";
import type { TIssue } from "@plane/types";

export class IssueStore {
  issues = new Map<string, TIssue>();
  isCreating = false;

  constructor() {
    makeObservable(this, {
      issues: observable,
      isCreating: observable,
      createIssue: action,
    });
  }

  createIssue = async (data: Partial<TIssue>) => {
    // STEP 4: Optimistic update
    const tempId = `temp-${Date.now()}`;
    const tempIssue: TIssue = {
      id: tempId,
      ...data,
      created_at: new Date().toISOString(),
    } as TIssue;

    this.issues.set(tempId, tempIssue);  // UI updates NOW!
    this.isCreating = true;

    try {
      // STEP 5: Call service (network request)
      const issue = await IssueService.create(data);

      // STEP 6: Replace temp with real issue
      runInAction(() => {
        this.issues.delete(tempId);
        this.issues.set(issue.id, issue);
        this.isCreating = false;
      });

      return issue;
    } catch (error) {
      // STEP 7: Rollback on error
      runInAction(() => {
        this.issues.delete(tempId);
        this.isCreating = false;
      });
      throw error;
    }
  };
}
```

**Timeline:**
- **2ms** - Create temp issue
- **2ms** - Add to store (triggers React re-render)
- **3ms** - Start API call
- **3-200ms** - Network request in flight
- **200ms** - Response received
- **201ms** - Update store with real data (triggers re-render)

**💡 Aha Moment:**
> Optimistic updates make the UI feel instant! The user sees their change at 2ms, even though the server doesn't confirm until 200ms.

---

### Phase 4: Service Layer

```typescript
// /packages/services/src/issue.service.ts
import { APIService } from "./api.service";
import type { TIssue } from "@plane/types";

export class IssueService extends APIService {
  constructor() {
    super("/api/v1");
  }

  async create(
    workspaceSlug: string,
    projectId: string,
    data: Partial<TIssue>
  ): Promise<TIssue> {
    // STEP 8: Make HTTP request
    return this.post(
      `/workspaces/${workspaceSlug}/projects/${projectId}/issues/`,
      data
    );
  }
}

// Base service
export abstract class APIService {
  protected async post<T>(url: string, data: any): Promise<T> {
    // STEP 9: Send request
    const response = await axios.post(url, data, {
      headers: {
        "Content-Type": "application/json",
      },
      withCredentials: true,  // Include session cookie
    });

    // STEP 10: Return data
    return response.data;
  }
}
```

**Timeline:**
- **3ms** - Prepare request (serialize JSON)
- **4ms** - Send HTTP POST
- **4-200ms** - Network + server processing
- **200ms** - Receive response
- **201ms** - Parse JSON
- **202ms** - Return to store

---

### Phase 5: UI Update

```typescript
// React component (automatically re-renders)
const IssueList = observer(() => {
  const issueStore = useIssueStore();

  // STEP 11: Component re-renders when store.issues changes
  const issues = Array.from(issueStore.issues.values());

  return (
    <div>
      {issues.map(issue => (
        <IssueCard key={issue.id} issue={issue} />
      ))}
    </div>
  );
});
```

**Timeline:**
- **2ms** - First render (with temp issue)
- **202ms** - Second render (with real issue)

**🌉 React to MobX Bridge:**

```typescript
// WITHOUT MobX (manual updates)
const [issues, setIssues] = useState([]);

const createIssue = async (data) => {
  setIssues(prev => [...prev, tempIssue]);  // Manual update #1
  const issue = await API.create(data);
  setIssues(prev => [...prev.filter(i => i.id !== tempId), issue]);  // Manual update #2
};

// WITH MobX (automatic updates)
const issues = issueStore.issues;  // Automatic updates!

const createIssue = async (data) => {
  await issueStore.createIssue(data);
  // UI updates automatically - no setState needed!
};
```

---

## Backend Data Flow

### Phase 1: HTTP Request Arrives

```python
# Django middleware processes request
┌────────────────────────────────────┐
│ 1. CorsMiddleware                  │  Check CORS headers
├────────────────────────────────────┤
│ 2. SecurityMiddleware              │  Security headers
├────────────────────────────────────┤
│ 3. SessionMiddleware               │  Load session from Redis
├────────────────────────────────────┤
│ 4. AuthenticationMiddleware        │  Load user from session
├────────────────────────────────────┤
│ 5. Request reaches view            │
└────────────────────────────────────┘
```

**Timeline:**
- **0ms** - Request arrives at Django
- **1-5ms** - Middleware processing
- **5ms** - Reaches view function

---

### Phase 2: URL Routing

```python
# /apps/api/plane/urls.py
urlpatterns = [
    path('api/v1/', include('plane.app.urls')),
]

# /apps/api/plane/app/urls.py
urlpatterns = [
    path(
        'workspaces/<str:workspace_slug>/projects/<uuid:project_id>/issues/',
        IssueViewSet.as_view({'post': 'create'}),
        name='issue-create'
    ),
]
```

**Timeline:**
- **5-6ms** - URL matching
- **6ms** - Route to IssueViewSet.create

---

### Phase 3: Permission Check

```python
# /apps/api/plane/app/views/issue/issue.py
from rest_framework import status
from rest_framework.response import Response
from plane.app.views import BaseViewSet
from plane.app.permissions import ProjectPermission

class IssueViewSet(BaseViewSet):
    permission_classes = [ProjectPermission]

    def create(self, request, workspace_slug, project_id):
        # STEP 1: Permission check (automatic via DRF)
        self.check_permissions(request)

        # STEP 2: Validate data
        serializer = IssueSerializer(data=request.data)
        if not serializer.is_valid():
            return Response(
                serializer.errors,
                status=status.HTTP_400_BAD_REQUEST
            )

        # STEP 3: Save to database
        issue = serializer.save(
            project_id=project_id,
            created_by=request.user
        )

        # STEP 4: Background tasks
        send_notifications.delay(issue.id)

        # STEP 5: Return response
        return Response(
            IssueSerializer(issue).data,
            status=status.HTTP_201_CREATED
        )
```

**Timeline:**
- **6-10ms** - Permission check (database query)
- **10-12ms** - Validation
- **12-20ms** - Database write
- **20-21ms** - Queue background task
- **21-25ms** - Serialize response
- **25ms** - Send HTTP response

---

### Phase 4: Serializer Validation

```python
# /apps/api/plane/app/serializers/issue.py
from rest_framework import serializers
from plane.db.models import Issue, State, Project

class IssueSerializer(serializers.ModelSerializer):
    state = serializers.PrimaryKeyRelatedField(
        queryset=State.objects.all()
    )

    class Meta:
        model = Issue
        fields = [
            'id',
            'name',
            'description',
            'priority',
            'state',
            'project',
            'created_by',
            'created_at',
        ]
        read_only_fields = ['id', 'created_at', 'created_by']

    def validate_name(self, value):
        """Custom validation for name"""
        if len(value) < 3:
            raise serializers.ValidationError(
                "Issue name must be at least 3 characters"
            )
        return value

    def validate(self, attrs):
        """Validate state belongs to project"""
        state = attrs.get('state')
        project = attrs.get('project')

        if state.project_id != project.id:
            raise serializers.ValidationError(
                "State must belong to the project"
            )

        return attrs
```

**Timeline:**
- **10-11ms** - Field validation
- **11-12ms** - Custom validation
- **12ms** - Validation complete

**🧠 Mental Model:**
> Serializers are like TypeScript type guards - they ensure data is valid before it reaches the database.

---

### Phase 5: Database Write

```python
# Django ORM generates SQL
issue = serializer.save(
    project_id=project_id,
    created_by=request.user
)

# Translates to SQL:
INSERT INTO plane_issue (
    id,
    name,
    description,
    priority,
    state_id,
    project_id,
    created_by_id,
    created_at,
    updated_at
) VALUES (
    uuid_generate_v4(),
    'Fix login bug',
    '{"type": "doc", ...}',
    'high',
    'state-uuid',
    'project-uuid',
    'user-uuid',
    NOW(),
    NOW()
) RETURNING *;
```

**Timeline:**
- **12-13ms** - Generate SQL
- **13-18ms** - Execute INSERT
- **18-20ms** - PostgreSQL commits transaction
- **20ms** - Return new issue object

**💡 Aha Moment:**
> Django ORM protects you from SQL injection by using parameterized queries. You write Python, get safe SQL!

---

### Phase 6: Background Tasks

```python
# /apps/api/plane/bgtasks/notification.py
from celery import shared_task
from plane.db.models import Issue
from plane.utils.email import send_email

@shared_task
def send_notifications(issue_id):
    """Send notifications to assignees (async)"""
    issue = Issue.objects.select_related('project', 'created_by').get(id=issue_id)

    # Get users to notify
    assignees = issue.assignees.all()
    watchers = issue.project.members.filter(receive_notifications=True)

    for user in set(list(assignees) + list(watchers)):
        send_email(
            to=user.email,
            subject=f"New issue: {issue.name}",
            template="issue_created",
            context={"issue": issue, "user": user}
        )

# In view:
send_notifications.delay(issue.id)  # Returns immediately!
```

**Timeline:**
- **20ms** - Queue task in Redis (instant)
- **Later** - Celery worker picks up task (1-5 seconds)
- **Later** - Emails sent (5-10 seconds)

**🎯 Remember This:**
> Background tasks don't block the API response. The user gets their issue immediately, notifications send later.

---

## Database Operations

### Read Operations

#### Simple Query

```python
# Django ORM
issue = Issue.objects.get(id=issue_id)

# Generated SQL
SELECT * FROM plane_issue WHERE id = 'uuid-here' LIMIT 1;
```

**Timeline:** 1-5ms

---

#### Complex Query with Joins

```python
# Django ORM
issues = Issue.objects.filter(
    project_id=project_id
).select_related(
    'state',
    'project',
    'created_by'
).prefetch_related(
    'assignees',
    'labels'
)

# Generated SQL (optimized with joins)
SELECT
    i.*,
    s.*,
    p.*,
    u.*
FROM plane_issue i
INNER JOIN plane_state s ON i.state_id = s.id
INNER JOIN plane_project p ON i.project_id = p.id
INNER JOIN plane_user u ON i.created_by_id = u.id
WHERE i.project_id = 'project-uuid';

# Separate query for many-to-many
SELECT * FROM plane_user
WHERE id IN (
    SELECT user_id FROM plane_issue_assignee WHERE issue_id IN (...)
);
```

**Timeline:** 5-20ms (depending on data size)

**⚠️ Common Pitfall: N+1 Queries**

```python
# ❌ BAD: N+1 queries
issues = Issue.objects.all()  # 1 query
for issue in issues:
    print(issue.state.name)   # N queries (one per issue!)

# ✅ GOOD: 1 query with join
issues = Issue.objects.select_related('state')  # 1 query with JOIN
for issue in issues:
    print(issue.state.name)   # No additional queries!
```

---

### Write Operations

#### Create

```python
# Django ORM
issue = Issue.objects.create(
    name="Fix bug",
    project_id=project_id,
    state_id=state_id
)

# SQL
INSERT INTO plane_issue (...) VALUES (...) RETURNING *;
```

**Timeline:** 5-15ms

---

#### Update

```python
# Django ORM
issue.priority = "high"
issue.save(update_fields=['priority'])

# SQL
UPDATE plane_issue SET priority = 'high', updated_at = NOW()
WHERE id = 'uuid';
```

**Timeline:** 3-10ms

---

#### Delete (Soft)

```python
# Soft delete (Plane's pattern)
issue.deleted_at = timezone.now()
issue.save(update_fields=['deleted_at'])

# SQL
UPDATE plane_issue SET deleted_at = NOW() WHERE id = 'uuid';
```

**Timeline:** 3-10ms

**💡 Aha Moment:**
> Plane uses soft deletes - data is never truly deleted, just marked as deleted. This allows recovery and audit trails!

---

### Transactions

```python
from django.db import transaction

@transaction.atomic
def create_issue_with_comment(issue_data, comment_text):
    # All or nothing
    issue = Issue.objects.create(**issue_data)
    comment = Comment.objects.create(
        issue=issue,
        text=comment_text
    )

    # If comment fails, issue creation is rolled back
    return issue

# SQL
BEGIN;
INSERT INTO plane_issue (...) VALUES (...);
INSERT INTO plane_comment (...) VALUES (...);
COMMIT;  -- or ROLLBACK if error
```

**Timeline:** 10-30ms

---

## Real-Time Updates

### WebSocket Flow

```
User A updates issue
       ↓
Django saves to database
       ↓
Django publishes to Redis Pub/Sub
       ↓
WebSocket server receives message
       ↓
Server broadcasts to connected clients
       ↓
User B's browser receives update
       ↓
Frontend updates UI (no page refresh!)
```

---

### Implementation

```typescript
// Frontend: Subscribe to updates
const socket = new WebSocket(`ws://localhost:8000/ws/issues/${issueId}/`);

socket.onmessage = (event) => {
  const { type, data } = JSON.parse(event.data);

  switch (type) {
    case 'issue.updated':
      // Update store
      issueStore.updateIssue(data.id, data);
      break;

    case 'comment.created':
      // Add comment
      commentStore.addComment(data);
      break;

    case 'user.joined':
      // Show presence
      presenceStore.addUser(data.user_id);
      break;
  }
};
```

```python
# Backend: Broadcast update
from channels.layers import get_channel_layer
from asgiref.sync import async_to_sync

def broadcast_issue_update(issue):
    channel_layer = get_channel_layer()
    async_to_sync(channel_layer.group_send)(
        f"issue_{issue.id}",
        {
            "type": "issue.updated",
            "data": IssueSerializer(issue).data
        }
    )

# After saving issue:
issue.save()
broadcast_issue_update(issue)  # Real-time!
```

**Timeline:**
- **0ms** - Issue updated
- **1-2ms** - Broadcast to Redis
- **2-10ms** - All connected clients receive update
- **10ms** - UIs update

---

## Caching Strategy

### Multi-Layer Caching

```
┌─────────────────────────────────────────────┐
│ Browser Cache (Static Assets)               │
│ Duration: Until deployment (~hours/days)    │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│ SWR Cache (API Responses)                   │
│ Duration: Until mutation or focus (~seconds)│
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│ Redis Cache (Frequently Accessed Data)      │
│ Duration: 5 minutes (configurable)          │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│ PostgreSQL (Source of Truth)                │
│ Duration: Forever (until deleted)           │
└─────────────────────────────────────────────┘
```

---

### Cache Invalidation

```typescript
// Frontend: Mutate cache
import { useSWRConfig } from "swr";

const updateIssue = async (id: string, data: Partial<TIssue>) => {
  const { mutate } = useSWRConfig();

  // Optimistic update
  mutate(
    `/api/issues/${id}/`,
    (current: TIssue) => ({ ...current, ...data }),
    false  // Don't revalidate yet
  );

  // Send to server
  await IssueService.update(id, data);

  // Invalidate related caches
  mutate(`/api/issues/${id}/`);              // Single issue
  mutate(`/api/projects/${projectId}/issues/`);  // Issue list
};
```

```python
# Backend: Invalidate Redis cache
from django.core.cache import cache

def update_issue(issue_id, data):
    issue = Issue.objects.get(id=issue_id)

    # Update issue
    for key, value in data.items():
        setattr(issue, key, value)
    issue.save()

    # Invalidate caches
    cache.delete(f"issue:{issue_id}")
    cache.delete(f"project:{issue.project_id}:issues")

    return issue
```

---

## Error Handling

### Frontend Error Handling

```typescript
// Component with error handling
const IssueList = () => {
  const { data, error, isLoading } = useSWR('/api/issues/', fetcher);

  if (isLoading) return <Loading />;

  if (error) {
    // Network error
    if (error.code === 'NETWORK_ERROR') {
      return <ErrorState message="No internet connection" />;
    }

    // 404 Not Found
    if (error.response?.status === 404) {
      return <NotFound />;
    }

    // 403 Forbidden
    if (error.response?.status === 403) {
      return <PermissionDenied />;
    }

    // Generic error
    return <ErrorState message="Something went wrong" />;
  }

  return <div>{data.map(...)}</div>;
};
```

---

### Backend Error Handling

```python
# Django view with error handling
from rest_framework.exceptions import ValidationError, PermissionDenied

class IssueViewSet(BaseViewSet):
    def create(self, request, workspace_slug, project_id):
        try:
            # Validate permissions
            if not self.has_project_access(request.user, project_id):
                raise PermissionDenied("You don't have access to this project")

            # Validate data
            serializer = IssueSerializer(data=request.data)
            if not serializer.is_valid():
                raise ValidationError(serializer.errors)

            # Create issue
            issue = serializer.save(
                project_id=project_id,
                created_by=request.user
            )

            return Response(
                IssueSerializer(issue).data,
                status=status.HTTP_201_CREATED
            )

        except PermissionDenied as e:
            return Response(
                {"error": str(e)},
                status=status.HTTP_403_FORBIDDEN
            )

        except ValidationError as e:
            return Response(
                {"error": e.detail},
                status=status.HTTP_400_BAD_REQUEST
            )

        except Exception as e:
            # Log unexpected errors
            logger.error(f"Error creating issue: {e}")
            return Response(
                {"error": "Internal server error"},
                status=status.HTTP_500_INTERNAL_SERVER_ERROR
            )
```

---

## Performance Optimizations

### Frontend Optimizations

#### 1. Memoization

```typescript
// Expensive computation
const sortedIssues = useMemo(
  () => issues.sort((a, b) => a.title.localeCompare(b.title)),
  [issues]  // Only recompute when issues change
);
```

#### 2. Lazy Loading

```typescript
// Code splitting
const IssueDetail = lazy(() => import('./IssueDetail'));

<Suspense fallback={<Loading />}>
  <IssueDetail issueId={id} />
</Suspense>
```

#### 3. Virtual Scrolling

```typescript
// Only render visible items
import { useVirtualizer } from '@tanstack/react-virtual';

const IssueList = ({ issues }: { issues: TIssue[] }) => {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: issues.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 60,  // Row height
  });

  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map(item => (
          <IssueCard
            key={item.key}
            issue={issues[item.index]}
            style={{ height: item.size }}
          />
        ))}
      </div>
    </div>
  );
};
```

---

### Backend Optimizations

#### 1. Database Indexes

```python
class Issue(models.Model):
    project = models.ForeignKey(Project, on_delete=models.CASCADE)
    state = models.ForeignKey(State, on_delete=models.CASCADE)
    priority = models.CharField(max_length=30)

    class Meta:
        indexes = [
            models.Index(fields=['project', 'state']),  # Composite index
            models.Index(fields=['priority']),          # Single field
            models.Index(fields=['-created_at']),       # Descending order
        ]
```

#### 2. Query Optimization

```python
# ✅ GOOD: Optimized query
issues = Issue.objects.filter(
    project_id=project_id
).select_related(
    'state',
    'project'
).prefetch_related(
    'assignees',
    'labels'
).only(
    'id',
    'name',
    'priority',
    'state__name',
    'project__name'
)[:50]  # Limit results

# ❌ BAD: Unoptimized query
issues = Issue.objects.all()  # Fetches everything!
```

#### 3. Pagination

```python
from rest_framework.pagination import PageNumberPagination

class IssuePagination(PageNumberPagination):
    page_size = 50
    page_size_query_param = 'page_size'
    max_page_size = 100

class IssueViewSet(BaseViewSet):
    pagination_class = IssuePagination
```

---

## Next Steps

### Continue Learning

1. **[FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md)** - Deep dive into React patterns
2. **[BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)** - Deep dive into Django
3. **[INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md)** - Frontend ↔ Backend communication

---

### Practice

Try tracing data flow for:
- Updating an issue's priority
- Adding a comment
- Deleting an issue
- Filtering issues by assignee

---

**🎉 You now understand how data flows through Plane!**

---

**Last Updated:** November 19, 2025
**Maintainers:** Plane Learning Docs Team
**License:** AGPL-3.0
