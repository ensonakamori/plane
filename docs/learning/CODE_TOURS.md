# Code Tours - Guided Feature Walkthroughs

**Follow the code!** This guide walks you through complete features in Plane, showing how frontend, backend, and database work together.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 1.5-2 hours
**📊 Difficulty Level:** Intermediate to Advanced
**🎯 Target Audience:** Developers learning the codebase architecture

---

## Table of Contents

1. [Tour 1: Creating an Issue](#tour-1-creating-an-issue)
2. [Tour 2: User Authentication](#tour-2-user-authentication)
3. [Tour 3: Real-Time Notifications](#tour-3-real-time-notifications)
4. [Tour 4: Issue State Changes](#tour-4-issue-state-changes)

---

## Tour 1: Creating an Issue

**Journey:** User clicks "Create Issue" → Issue appears in database → UI updates

### Step 1: User Interaction (Frontend)

**File:** `/home/user/plane/apps/web/app/(all)/[workspaceSlug]/(projects)/projects/(detail)/[projectId]/issues/(list)/page.tsx`

```tsx
// User clicks "Create Issue" button
export default function IssuesPage() {
  const [isCreateModalOpen, setIsCreateModalOpen] = useState(false);

  return (
    <div>
      <Button onClick={() => setIsCreateModalOpen(true)}>
        Create Issue
      </Button>

      {isCreateModalOpen && (
        <IssueCreateModal
          onClose={() => setIsCreateModalOpen(false)}
          onSuccess={(issue) => {
            setIsCreateModalOpen(false);
            // Navigate to new issue or refresh list
          }}
        />
      )}
    </div>
  );
}
```

### Step 2: Form Component

**File:** `/home/user/plane/apps/web/components/issues/issue-create-modal.tsx` (conceptual)

```tsx
import { useState } from 'react';
import { useParams, useNavigate } from 'react-router';
import { observer } from 'mobx-react-lite';
import { useStore } from '@/core/hooks/use-store';

export const IssueCreateModal = observer(({ onClose, onSuccess }) => {
  const { workspaceSlug, projectId } = useParams();
  const { issueStore } = useStore();
  const navigate = useNavigate();

  const [formData, setFormData] = useState({
    name: '',
    description_html: '<p></p>',
    priority: 'medium',
    assignee_ids: [],
    state_id: null
  });

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    try {
      // Call MobX store action
      const issue = await issueStore.createIssue(
        workspaceSlug,
        projectId,
        formData
      );

      onSuccess(issue);
      navigate(`/${workspaceSlug}/projects/${projectId}/issues/${issue.id}`);

    } catch (error) {
      console.error('Failed to create issue:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.name}
        onChange={(e) => setFormData({ ...formData, name: e.target.value })}
        placeholder="Issue name"
        required
      />
      {/* More form fields... */}
      <button type="submit">Create</button>
    </form>
  );
});
```

### Step 3: MobX Store

**File:** `/home/user/plane/apps/web/core/store/issue/issue.store.ts` (conceptual)

```typescript
import { makeObservable, observable, action, runInAction } from 'mobx';
import { issueService } from '@/core/services/issue.service';

class IssueStore {
  issues: Map<string, Issue> = new Map();
  isLoading = false;

  constructor() {
    makeObservable(this, {
      issues: observable,
      isLoading: observable,
      createIssue: action
    });
  }

  async createIssue(
    workspaceSlug: string,
    projectId: string,
    data: IssueFormData
  ): Promise<Issue> {
    this.isLoading = true;

    try {
      // Call API service
      const issue = await issueService.create(workspaceSlug, projectId, data);

      runInAction(() => {
        // Add to store
        this.issues.set(issue.id, issue);
        this.isLoading = false;
      });

      return issue;

    } catch (error) {
      runInAction(() => {
        this.isLoading = false;
      });
      throw error;
    }
  }
}
```

### Step 4: API Service

**File:** `/home/user/plane/apps/web/core/services/issue.service.ts` (conceptual)

```typescript
class IssueService {
  private api: APIService;

  async create(
    workspaceSlug: string,
    projectId: string,
    data: IssueFormData
  ): Promise<Issue> {
    const response = await this.api.post(
      `/workspaces/${workspaceSlug}/projects/${projectId}/issues/`,
      data
    );

    return response;
  }
}

// Base API service handles auth, errors
class APIService {
  async post<T>(path: string, data: any): Promise<T> {
    const token = localStorage.getItem('access_token');

    const response = await fetch(`${this.baseURL}${path}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify(data)
    });

    if (!response.ok) {
      throw new APIError(response.status, await response.json());
    }

    return response.json();
  }
}
```

### Step 5: HTTP Request Travels to Backend

```
Browser sends:
POST https://api.plane.so/api/workspaces/acme/projects/proj-1/issues/
Headers:
  Authorization: Bearer eyJhbGci...
  Content-Type: application/json
Body:
  {
    "name": "Fix login bug",
    "priority": "high",
    "assignee_ids": ["user-1"]
  }
```

### Step 6: Django URL Router

**File:** `/home/user/plane/apps/api/plane/app/urls.py`

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from plane.app.views import IssueViewSet

router = DefaultRouter()
router.register(
    r'workspaces/(?P<workspace_slug>[\w-]+)/projects/(?P<project_id>[^/.]+)/issues',
    IssueViewSet,
    basename='issue'
)

urlpatterns = [
    path('api/', include(router.urls)),
]

# URL pattern matches:
# POST /api/workspaces/{workspace_slug}/projects/{project_id}/issues/
# → IssueViewSet.create()
```

### Step 7: Authentication Middleware

**Django automatically verifies JWT token:**

```python
# DRF Authentication (automatic)
# 1. Extract token from Authorization header
# 2. Verify token signature
# 3. Decode payload: { user_id: "abc-123", exp: 1637856000 }
# 4. Load User from database
# 5. Set request.user = User object
```

### Step 8: ViewSet Create Method

**File:** `/home/user/plane/apps/api/plane/app/views/issue.py` (conceptual)

```python
from rest_framework import viewsets
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from plane.app.serializers import IssueCreateSerializer
from plane.db.models import Issue, Project

class IssueViewSet(viewsets.ModelViewSet):
    serializer_class = IssueCreateSerializer
    permission_classes = [IsAuthenticated]

    def create(self, request, workspace_slug, project_id):
        # 1. Check permissions
        project = Project.objects.get(
            id=project_id,
            workspace__slug=workspace_slug,
            members=request.user  # User must be project member
        )

        # 2. Validate data
        serializer = self.get_serializer(
            data=request.data,
            context={'project_id': project_id}
        )
        serializer.is_valid(raise_exception=True)

        # 3. Save to database
        issue = serializer.save(
            project=project,
            workspace=project.workspace,
            created_by=request.user
        )

        # 4. Queue background task
        from plane.bgtasks.issue_activities_task import issue_activity
        issue_activity.delay(
            issue_id=str(issue.id),
            action='created',
            user_id=str(request.user.id)
        )

        # 5. Return response
        return Response(serializer.data, status=201)
```

### Step 9: Serializer

**File:** `/home/user/plane/apps/api/plane/app/serializers/issue.py`

```python
from rest_framework import serializers
from plane.db.models import Issue, User, State

class IssueCreateSerializer(serializers.ModelSerializer):
    # Write-only fields for relationships
    assignee_ids = serializers.ListField(
        child=serializers.UUIDField(),
        write_only=True,
        required=False
    )
    state_id = serializers.UUIDField(write_only=True, required=False)

    class Meta:
        model = Issue
        fields = ['id', 'name', 'description_html', 'priority', 'assignee_ids', 'state_id']

    def validate(self, attrs):
        """Custom validation"""
        # Validate assignees are project members
        if attrs.get('assignee_ids'):
            valid_assignees = ProjectMember.objects.filter(
                project_id=self.context['project_id'],
                member_id__in=attrs['assignee_ids'],
                is_active=True
            ).values_list('member_id', flat=True)

            attrs['assignee_ids'] = list(valid_assignees)

        return attrs

    def create(self, validated_data):
        """Create issue with relationships"""
        assignee_ids = validated_data.pop('assignee_ids', [])

        # Create issue
        issue = Issue.objects.create(**validated_data)

        # Set assignees (many-to-many)
        if assignee_ids:
            issue.assignees.set(assignee_ids)

        return issue
```

### Step 10: Database INSERT

**PostgreSQL executes:**

```sql
-- Start transaction
BEGIN;

-- Insert issue
INSERT INTO db_issue (
    id, name, description_html, priority,
    project_id, workspace_id, created_by_id,
    created_at, updated_at, sequence_id
) VALUES (
    gen_random_uuid(),
    'Fix login bug',
    '<p>Description...</p>',
    'high',
    'proj-1',
    'ws-1',
    'user-123',
    NOW(),
    NOW(),
    (SELECT COALESCE(MAX(sequence_id), 0) + 1 FROM db_issue WHERE project_id = 'proj-1')
);

-- Insert assignees (many-to-many)
INSERT INTO issue_assignees (issue_id, user_id)
VALUES ('issue-abc', 'user-1');

-- Commit transaction
COMMIT;
```

### Step 11: Response Returned

**Django sends:**

```json
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Fix login bug",
  "description_html": "<p>Description...</p>",
  "priority": "high",
  "sequence_id": 42,
  "assignees": [
    {
      "id": "user-1",
      "username": "alice",
      "avatar": "https://..."
    }
  ],
  "created_at": "2025-11-19T10:30:00Z",
  "updated_at": "2025-11-19T10:30:00Z"
}
```

### Step 12: Frontend Receives Response

**Back to service layer:**

```typescript
// APIService.post() resolves with data
const issue = await response.json();
// issue = { id: "550e...", name: "Fix login bug", ... }

// IssueService.create() returns issue
return issue;

// IssueStore.createIssue() adds to store
runInAction(() => {
  this.issues.set(issue.id, issue);
});

// Component receives issue
const issue = await issueStore.createIssue(...);

// Navigate to issue detail page
navigate(`/issues/${issue.id}`);
```

### Step 13: Background Task (Async)

**Celery worker picks up task:**

```python
# plane/bgtasks/issue_activities_task.py
from celery import shared_task

@shared_task
def issue_activity(issue_id, action, user_id):
    """Log issue activity (runs in background)"""
    from plane.db.models import Issue, IssueActivity, User

    issue = Issue.objects.get(id=issue_id)
    user = User.objects.get(id=user_id)

    # Create activity log
    IssueActivity.objects.create(
        issue=issue,
        actor=user,
        action=action,
        created_at=timezone.now()
    )

    # Send notifications to assignees
    for assignee in issue.assignees.all():
        send_email_notification(assignee, f"{user.username} created issue: {issue.name}")
```

### Complete Flow Diagram

```
User clicks button
  → React Component renders form
  → Form submitted
  → MobX Store action called
  → API Service makes HTTP POST
  → Request sent to Django
  ↓
Django URL router matches URL
  → Authentication middleware verifies JWT
  → ViewSet.create() called
  → Permission check
  → Serializer validates data
  → Issue saved to PostgreSQL
  → Background task queued (Celery)
  → Response serialized to JSON
  → HTTP 201 response sent
  ↓
Frontend receives response
  → API Service returns data
  → MobX Store updates
  → Component re-renders
  → User sees new issue
  → Navigate to issue detail
  ↓
Background (async):
  → Celery worker processes task
  → Activity log created
  → Notifications sent
```

**Total request time: ~100ms**
**Background task time: ~2-5 seconds (doesn't block response)**

---

## Tour 2: User Authentication

**Journey:** Login form → JWT token → Authenticated requests

### Frontend: Login Component

```tsx
const LoginPage = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const navigate = useNavigate();

  const handleLogin = async (e: React.FormEvent) => {
    e.preventDefault();

    try {
      const response = await fetch('/api/auth/sign-in/', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });

      const data = await response.json();

      // Store tokens
      localStorage.setItem('access_token', data.access_token);
      localStorage.setItem('refresh_token', data.refresh_token);

      // Store user
      localStorage.setItem('user', JSON.stringify(data.user));

      navigate('/');

    } catch (error) {
      console.error('Login failed');
    }
  };

  return (
    <form onSubmit={handleLogin}>
      <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} />
      <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} />
      <button>Sign In</button>
    </form>
  );
};
```

### Backend: Authentication View

**File:** `/home/user/plane/apps/api/plane/app/views/auth.py` (conceptual)

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework_simplejwt.tokens import RefreshToken
from django.contrib.auth import authenticate

@api_view(['POST'])
def sign_in(request):
    email = request.data.get('email')
    password = request.data.get('password')

    # Authenticate user
    user = authenticate(email=email, password=password)

    if user is None:
        return Response({'error': 'Invalid credentials'}, status=401)

    # Generate JWT tokens
    refresh = RefreshToken.for_user(user)

    return Response({
        'access_token': str(refresh.access_token),
        'refresh_token': str(refresh),
        'user': {
            'id': str(user.id),
            'email': user.email,
            'display_name': user.display_name,
            'avatar': user.avatar
        }
    })
```

### JWT Token Structure

**Access token payload:**

```json
{
  "token_type": "access",
  "exp": 1637856000,
  "iat": 1637769600,
  "jti": "abc123",
  "user_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**How token is verified on subsequent requests:**

```python
# DRF automatically does this for protected endpoints
from rest_framework_simplejwt.authentication import JWTAuthentication

class IssueViewSet(viewsets.ModelViewSet):
    authentication_classes = [JWTAuthentication]

    def list(self, request):
        # request.user is automatically set from JWT token
        user = request.user  # User object loaded from database
```

---

## Tour 3: Real-Time Notifications

**Journey:** Issue updated → WebSocket message → UI updates instantly

### Backend: WebSocket Consumer

```python
# plane/app/consumers/notification.py
from channels.generic.websocket import AsyncJsonWebsocketConsumer

class NotificationConsumer(AsyncJsonWebsocketConsumer):
    async def connect(self):
        self.workspace_id = self.scope['url_route']['kwargs']['workspace_id']
        self.group_name = f'workspace_{self.workspace_id}'

        # Join workspace group
        await self.channel_layer.group_add(self.group_name, self.channel_name)
        await self.accept()

    async def disconnect(self, code):
        # Leave workspace group
        await self.channel_layer.group_discard(self.group_name, self.channel_name)

    async def issue_updated(self, event):
        # Send notification to WebSocket
        await self.send_json({
            'type': 'issue.updated',
            'data': event['data']
        })
```

### Backend: Signal Triggers WebSocket

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from channels.layers import get_channel_layer
from asgiref.sync import async_to_sync

@receiver(post_save, sender=Issue)
def issue_saved(sender, instance, created, **kwargs):
    """Send WebSocket notification when issue is saved"""
    if not created:  # Only on update
        channel_layer = get_channel_layer()
        group_name = f'workspace_{instance.workspace_id}'

        async_to_sync(channel_layer.group_send)(
            group_name,
            {
                'type': 'issue_updated',
                'data': {
                    'id': str(instance.id),
                    'name': instance.name,
                    'priority': instance.priority
                }
            }
        )
```

### Frontend: WebSocket Client

```typescript
class WebSocketService {
  private socket: WebSocket | null = null;

  connect(workspaceId: string) {
    const token = localStorage.getItem('access_token');
    const url = `wss://api.plane.so/ws/${workspaceId}/?token=${token}`;

    this.socket = new WebSocket(url);

    this.socket.onmessage = (event) => {
      const data = JSON.parse(event.data);

      if (data.type === 'issue.updated') {
        this.handleIssueUpdate(data.data);
      }
    };
  }

  private handleIssueUpdate(issueData: any) {
    // Update MobX store
    issueStore.updateIssue(issueData.id, issueData);

    // Show toast notification
    toast.info(`Issue "${issueData.name}" was updated`);
  }
}
```

### Component Subscribes to Updates

```tsx
const IssueDetail = ({ issueId }: { issueId: string }) => {
  const { issueStore } = useStore();
  const issue = issueStore.issues.get(issueId);

  useEffect(() => {
    // Connect WebSocket
    websocketService.connect(workspaceId);

    return () => websocketService.disconnect();
  }, [workspaceId]);

  // Component automatically re-renders when issueStore updates
  return (
    <div>
      <h1>{issue?.name}</h1>
      <span>{issue?.priority}</span>
    </div>
  );
};
```

### Flow

```
User A updates issue
  → Django saves to database
  → post_save signal fires
  → Sends message to WebSocket channel
  → All connected clients in workspace receive message
  → User B's browser updates automatically
  → UI re-renders with new data
```

---

## Tour 4: Issue State Changes

**Journey:** Drag issue to different column → State updates → Backend synced

### Frontend: Drag and Drop

```tsx
import { DragDropContext, Droppable, Draggable } from 'react-beautiful-dnd';

const KanbanBoard = observer(() => {
  const { issueStore } = useStore();

  const handleDragEnd = async (result) => {
    if (!result.destination) return;

    const issueId = result.draggableId;
    const newStateId = result.destination.droppableId;

    // Optimistic update (immediate UI change)
    issueStore.updateIssue(issueId, { state_id: newStateId });

    try {
      // Sync with backend
      await issueService.update(workspaceSlug, projectId, issueId, {
        state_id: newStateId
      });
    } catch (error) {
      // Revert on error
      issueStore.revertIssue(issueId);
      toast.error('Failed to update issue');
    }
  };

  return (
    <DragDropContext onDragEnd={handleDragEnd}>
      {states.map(state => (
        <Droppable key={state.id} droppableId={state.id}>
          {(provided) => (
            <div ref={provided.innerRef} {...provided.droppableProps}>
              <h3>{state.name}</h3>
              {getIssuesForState(state.id).map((issue, index) => (
                <Draggable key={issue.id} draggableId={issue.id} index={index}>
                  {(provided) => (
                    <div
                      ref={provided.innerRef}
                      {...provided.draggableProps}
                      {...provided.dragHandleProps}
                    >
                      <IssueCard issue={issue} />
                    </div>
                  )}
                </Draggable>
              ))}
              {provided.placeholder}
            </div>
          )}
        </Droppable>
      ))}
    </DragDropContext>
  );
});
```

### Backend: State Update

```python
class IssueViewSet(viewsets.ModelViewSet):
    def partial_update(self, request, workspace_slug, project_id, pk):
        issue = self.get_object()
        old_state = issue.state

        serializer = self.get_serializer(issue, data=request.data, partial=True)
        serializer.is_valid(raise_exception=True)
        serializer.save()

        # Log state change
        if 'state_id' in request.data and old_state.id != issue.state_id:
            IssueActivity.objects.create(
                issue=issue,
                actor=request.user,
                action='state_changed',
                old_value=old_state.name,
                new_value=issue.state.name
            )

        return Response(serializer.data)
```

---

## Key Takeaways

### What You Learned

- ✅ **Complete request flow** - Frontend → Backend → Database → Response
- ✅ **Authentication** - JWT tokens, middleware, permissions
- ✅ **Real-time updates** - WebSockets, signals, automatic UI updates
- ✅ **Optimistic updates** - Instant UI feedback, sync with backend
- ✅ **State management** - MobX stores, React components, data flow

### Next Steps

1. **[HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)** - Implement your own features
2. **[TESTING_GUIDE.md](./TESTING_GUIDE.md)** - Test these flows
3. **[DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)** - Debug issues in the flow

---

**You now understand how Plane's features work end-to-end! 🚀**
