# Integration Guide - Frontend ↔ Backend Communication

**Connect React to Django like a pro!** This guide shows you exactly how the frontend and backend communicate in Plane, from authentication to real-time updates.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 1-1.5 hours
**📊 Difficulty Level:** Intermediate
**🎯 Target Audience:** Full-stack developers connecting React to Django

---

## Table of Contents

1. [Introduction - The Full Stack](#introduction---the-full-stack)
2. [HTTP Communication Basics](#http-communication-basics)
3. [Authentication Flow](#authentication-flow)
4. [Making API Requests from React](#making-api-requests-from-react)
5. [Service Layer Pattern](#service-layer-pattern)
6. [Error Handling](#error-handling)
7. [State Management Integration](#state-management-integration)
8. [Real-Time Updates (WebSockets)](#real-time-updates-websockets)
9. [File Uploads](#file-uploads)
10. [Optimistic Updates](#optimistic-updates)
11. [Complete Examples](#complete-examples)
12. [Debugging Integration Issues](#debugging-integration-issues)
13. [Best Practices](#best-practices)
14. [Next Steps](#next-steps)

---

## Introduction - The Full Stack

### The Complete Picture

```
┌────────────────────────────────────────────────────┐
│                    BROWSER                         │
│  ┌──────────────────────────────────────────┐     │
│  │  React Component                         │     │
│  │  - User clicks "Create Issue"            │     │
│  │  - Component calls IssueService          │     │
│  └──────────────┬───────────────────────────┘     │
│                 │                                  │
│  ┌──────────────▼───────────────────────────┐     │
│  │  Service Layer (services/issue.ts)       │     │
│  │  - Prepares request                      │     │
│  │  - Adds auth headers                     │     │
│  │  - Sends HTTP POST                       │     │
│  └──────────────┬───────────────────────────┘     │
└─────────────────┼────────────────────────────────┘
                  │ HTTP Request
                  │ POST /api/workspaces/acme/projects/proj-1/issues/
                  │ Headers: { Authorization: Bearer <token> }
                  │ Body: { name: "Fix bug", priority: "high" }
┌─────────────────▼────────────────────────────────┐
│                    SERVER                         │
│  ┌──────────────────────────────────────────┐    │
│  │  Django URL Router                       │    │
│  │  - Matches URL pattern                   │    │
│  │  - Routes to IssueViewSet                │    │
│  └──────────────┬───────────────────────────┘    │
│                 │                                 │
│  ┌──────────────▼───────────────────────────┐    │
│  │  Authentication Middleware               │    │
│  │  - Verifies JWT token                    │    │
│  │  - Sets request.user                     │    │
│  └──────────────┬───────────────────────────┘    │
│                 │                                 │
│  ┌──────────────▼───────────────────────────┐    │
│  │  IssueViewSet.create()                   │    │
│  │  - Validates permissions                 │    │
│  │  - Validates data                        │    │
│  │  - Saves to database                     │    │
│  └──────────────┬───────────────────────────┘    │
│                 │                                 │
│  ┌──────────────▼───────────────────────────┐    │
│  │  PostgreSQL Database                     │    │
│  │  - INSERT INTO issues ...                │    │
│  └──────────────┬───────────────────────────┘    │
│                 │                                 │
│  ┌──────────────▼───────────────────────────┐    │
│  │  Response                                │    │
│  │  - Serialize Issue → JSON                │    │
│  │  - Return HTTP 201 Created               │    │
│  └──────────────┬───────────────────────────┘    │
└─────────────────┼────────────────────────────────┘
                  │ HTTP Response
                  │ Status: 201 Created
                  │ Body: { id: "...", name: "Fix bug", ... }
┌─────────────────▼────────────────────────────────┐
│                    BROWSER                        │
│  ┌──────────────────────────────────────────┐    │
│  │  Service Layer                           │    │
│  │  - Receives response                     │    │
│  │  - Returns data to component             │    │
│  └──────────────┬───────────────────────────┘    │
│                 │                                 │
│  ┌──────────────▼───────────────────────────┐    │
│  │  React Component                         │    │
│  │  - Updates state                         │    │
│  │  - Re-renders UI                         │    │
│  │  - Shows success message                 │    │
│  └──────────────────────────────────────────┘    │
└──────────────────────────────────────────────────┘
```

**Total time: 50-200ms** ⚡

---

## HTTP Communication Basics

### REST API Fundamentals

**REST (Representational State Transfer)** uses HTTP methods for CRUD operations:

| HTTP Method | CRUD Operation | Example |
|-------------|---------------|---------|
| `GET` | Read | Get issues list |
| `POST` | Create | Create new issue |
| `PUT` | Replace | Replace entire issue |
| `PATCH` | Update | Update issue fields |
| `DELETE` | Delete | Delete issue |

### HTTP Status Codes

| Code | Meaning | When to Use |
|------|---------|------------|
| `200` | OK | Successful GET, PUT, PATCH |
| `201` | Created | Successful POST (new resource) |
| `204` | No Content | Successful DELETE |
| `400` | Bad Request | Validation error |
| `401` | Unauthorized | Not authenticated |
| `403` | Forbidden | Not authorized |
| `404` | Not Found | Resource doesn't exist |
| `500` | Server Error | Backend crashed |

### Request/Response Structure

**Request:**

```typescript
// HTTP Request
POST /api/workspaces/acme/projects/proj-1/issues/
Host: api.plane.so
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

{
  "name": "Fix login bug",
  "priority": "high",
  "assignee_ids": ["user-1", "user-2"]
}
```

**Response:**

```typescript
// HTTP Response
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Fix login bug",
  "priority": "high",
  "assignees": [
    { "id": "user-1", "username": "alice", "avatar": "..." },
    { "id": "user-2", "username": "bob", "avatar": "..." }
  ],
  "created_at": "2025-11-19T10:30:00Z",
  "updated_at": "2025-11-19T10:30:00Z"
}
```

---

## Authentication Flow

### JWT Token Authentication ✅ CURRENT

**Plane uses JWT (JSON Web Tokens)** for authentication.

### Login Flow

**Step 1: User submits login form**

```tsx
// Frontend: Login component
const handleLogin = async (email: string, password: string) => {
  try {
    // Call login API
    const response = await fetch('/api/auth/sign-in/', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    });

    if (!response.ok) {
      throw new Error('Login failed');
    }

    const data = await response.json();
    // data = {
    //   access_token: "eyJhbGci...",
    //   refresh_token: "eyJhbGci...",
    //   user: { id: "...", email: "...", ... }
    // }

    // Store tokens
    localStorage.setItem('access_token', data.access_token);
    localStorage.setItem('refresh_token', data.refresh_token);

    // Redirect to dashboard
    navigate('/workspace');

  } catch (error) {
    console.error('Login failed:', error);
    toast.error('Invalid credentials');
  }
};
```

**Step 2: Backend verifies credentials**

```python
# Backend: Authentication view
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework_simplejwt.tokens import RefreshToken
from django.contrib.auth import authenticate

@api_view(['POST'])
def sign_in(request):
    email = request.data.get('email')
    password = request.data.get('password')

    # Verify credentials
    user = authenticate(email=email, password=password)

    if user is None:
        return Response(
            {'error': 'Invalid credentials'},
            status=401
        )

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

### Making Authenticated Requests

**Step 3: Include token in all API requests**

```tsx
// Frontend: API service with auth
const getIssues = async (projectId: string) => {
  const token = localStorage.getItem('access_token');

  const response = await fetch(`/api/projects/${projectId}/issues/`, {
    method: 'GET',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    }
  });

  if (response.status === 401) {
    // Token expired - refresh or redirect to login
    await refreshToken();
    // Retry request...
  }

  return response.json();
};
```

**Step 4: Backend verifies token**

```python
# Backend: Automatic via DRF authentication
from rest_framework.permissions import IsAuthenticated
from rest_framework_simplejwt.authentication import JWTAuthentication

class IssueViewSet(viewsets.ModelViewSet):
    authentication_classes = [JWTAuthentication]
    permission_classes = [IsAuthenticated]

    def list(self, request):
        # request.user is automatically set from token
        user = request.user  # User object from JWT

        # Only return issues user has access to
        issues = Issue.objects.filter(
            project__members=user
        )

        serializer = IssueSerializer(issues, many=True)
        return Response(serializer.data)
```

### Token Refresh Flow

```tsx
// Frontend: Refresh expired access token
const refreshToken = async () => {
  const refresh = localStorage.getItem('refresh_token');

  const response = await fetch('/api/auth/refresh/', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ refresh })
  });

  const data = await response.json();

  // Store new access token
  localStorage.setItem('access_token', data.access_token);

  return data.access_token;
};
```

### 🧠 Mental Model

```
Login → Get Tokens → Store Tokens → Include in Headers → Backend Verifies → Access Granted

Token expires → Refresh Token → Get New Access Token → Continue

Refresh expires → Logout → Redirect to Login
```

---

## Making API Requests from React

### Using Fetch API

```tsx
// Basic fetch request
const getIssues = async () => {
  const response = await fetch('/api/issues/');
  const data = await response.json();
  return data;
};

// POST with auth and body
const createIssue = async (issueData: IssueFormData) => {
  const token = localStorage.getItem('access_token');

  const response = await fetch('/api/issues/', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    },
    body: JSON.stringify(issueData)
  });

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }

  return response.json();
};

// PATCH (partial update)
const updateIssue = async (issueId: string, updates: Partial<Issue>) => {
  const token = localStorage.getItem('access_token');

  const response = await fetch(`/api/issues/${issueId}/`, {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    },
    body: JSON.stringify(updates)
  });

  return response.json();
};

// DELETE
const deleteIssue = async (issueId: string) => {
  const token = localStorage.getItem('access_token');

  const response = await fetch(`/api/issues/${issueId}/`, {
    method: 'DELETE',
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });

  if (response.status !== 204) {
    throw new Error('Delete failed');
  }
};
```

### Using SWR (Plane's Approach) ✅ CURRENT

**SWR provides caching, revalidation, and automatic refetching.**

```tsx
import useSWR from 'swr';

// Fetcher function
const fetcher = async (url: string) => {
  const token = localStorage.getItem('access_token');

  const response = await fetch(url, {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });

  if (!response.ok) {
    throw new Error('API error');
  }

  return response.json();
};

// Component using SWR
const IssueList = ({ projectId }: { projectId: string }) => {
  const { data: issues, error, isLoading, mutate } = useSWR(
    `/api/projects/${projectId}/issues/`,
    fetcher
  );

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading issues</div>;

  return (
    <div>
      {issues.map(issue => (
        <IssueCard key={issue.id} issue={issue} />
      ))}
    </div>
  );
};
```

**SWR Features:**

```tsx
// Automatic revalidation
const { data } = useSWR('/api/issues/', fetcher, {
  refreshInterval: 30000  // Refetch every 30 seconds
});

// Manual revalidation
const { data, mutate } = useSWR('/api/issues/', fetcher);
mutate();  // Refetch now

// Optimistic updates
const { data, mutate } = useSWR('/api/issues/', fetcher);

const createIssue = async (newIssue) => {
  // Update UI immediately (optimistic)
  mutate([...data, newIssue], false);

  // Send request
  const created = await IssueService.create(newIssue);

  // Update with real data
  mutate();
};
```

---

## Service Layer Pattern

### Why Service Layer?

**Centralize API logic** instead of fetching directly in components.

```tsx
// ❌ BAD - Fetch directly in component
const IssueList = () => {
  const [issues, setIssues] = useState([]);

  useEffect(() => {
    fetch('/api/issues/')
      .then(r => r.json())
      .then(setIssues);
  }, []);

  // Code is hard to reuse, test, and maintain
};
```

```tsx
// ✅ GOOD - Use service layer
const IssueList = () => {
  const { data: issues } = useSWR('/api/issues/', IssueService.getAll);
  // Service handles auth, errors, caching
};
```

### Service Class Example

**Location:** `/home/user/plane/apps/web/services/issue.service.ts` (conceptual)

```typescript
// services/issue.service.ts
import { APIService } from './api.service';

interface Issue {
  id: string;
  name: string;
  priority: string;
  assignee_ids: string[];
}

class IssueService {
  private api: APIService;

  constructor() {
    this.api = new APIService('/api');
  }

  // GET /api/workspaces/:workspaceSlug/projects/:projectId/issues/
  async getAll(workspaceSlug: string, projectId: string): Promise<Issue[]> {
    return this.api.get(
      `/workspaces/${workspaceSlug}/projects/${projectId}/issues/`
    );
  }

  // GET /api/.../issues/:issueId/
  async getById(
    workspaceSlug: string,
    projectId: string,
    issueId: string
  ): Promise<Issue> {
    return this.api.get(
      `/workspaces/${workspaceSlug}/projects/${projectId}/issues/${issueId}/`
    );
  }

  // POST /api/.../issues/
  async create(
    workspaceSlug: string,
    projectId: string,
    data: Partial<Issue>
  ): Promise<Issue> {
    return this.api.post(
      `/workspaces/${workspaceSlug}/projects/${projectId}/issues/`,
      data
    );
  }

  // PATCH /api/.../issues/:issueId/
  async update(
    workspaceSlug: string,
    projectId: string,
    issueId: string,
    data: Partial<Issue>
  ): Promise<Issue> {
    return this.api.patch(
      `/workspaces/${workspaceSlug}/projects/${projectId}/issues/${issueId}/`,
      data
    );
  }

  // DELETE /api/.../issues/:issueId/
  async delete(
    workspaceSlug: string,
    projectId: string,
    issueId: string
  ): Promise<void> {
    return this.api.delete(
      `/workspaces/${workspaceSlug}/projects/${projectId}/issues/${issueId}/`
    );
  }

  // Custom endpoint: POST /api/.../issues/bulk-create/
  async bulkCreate(
    workspaceSlug: string,
    projectId: string,
    issues: Partial<Issue>[]
  ): Promise<Issue[]> {
    return this.api.post(
      `/workspaces/${workspaceSlug}/projects/${projectId}/issues/bulk-create/`,
      { issues }
    );
  }
}

export const issueService = new IssueService();
```

### Base API Service

```typescript
// services/api.service.ts
class APIService {
  private baseURL: string;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
  }

  private getHeaders(): HeadersInit {
    const token = localStorage.getItem('access_token');

    return {
      'Content-Type': 'application/json',
      ...(token && { 'Authorization': `Bearer ${token}` })
    };
  }

  private async request<T>(
    path: string,
    options: RequestInit
  ): Promise<T> {
    const url = `${this.baseURL}${path}`;

    const response = await fetch(url, {
      ...options,
      headers: {
        ...this.getHeaders(),
        ...options.headers
      }
    });

    // Handle token refresh
    if (response.status === 401) {
      await this.refreshToken();
      // Retry request
      return this.request(path, options);
    }

    // Handle errors
    if (!response.ok) {
      const error = await response.json();
      throw new APIError(response.status, error);
    }

    // Handle empty responses (DELETE)
    if (response.status === 204) {
      return null as T;
    }

    return response.json();
  }

  async get<T>(path: string): Promise<T> {
    return this.request<T>(path, { method: 'GET' });
  }

  async post<T>(path: string, data: any): Promise<T> {
    return this.request<T>(path, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }

  async patch<T>(path: string, data: any): Promise<T> {
    return this.request<T>(path, {
      method: 'PATCH',
      body: JSON.stringify(data)
    });
  }

  async delete<T>(path: string): Promise<T> {
    return this.request<T>(path, { method: 'DELETE' });
  }

  private async refreshToken(): Promise<void> {
    // Refresh token logic
  }
}
```

---

## Error Handling

### Frontend Error Handling

```tsx
// Component with error handling
const IssueForm = () => {
  const [error, setError] = useState<string | null>(null);
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (data: IssueFormData) => {
    setError(null);
    setIsLoading(true);

    try {
      const issue = await issueService.create(workspaceSlug, projectId, data);

      toast.success('Issue created!');
      navigate(`/issues/${issue.id}`);

    } catch (err) {
      if (err instanceof APIError) {
        switch (err.status) {
          case 400:
            // Validation error
            setError(err.message);
            break;
          case 401:
            // Unauthorized
            navigate('/login');
            break;
          case 403:
            // Forbidden
            setError('You don\'t have permission to create issues');
            break;
          case 404:
            // Not found
            setError('Project not found');
            break;
          case 500:
            // Server error
            setError('Server error. Please try again later.');
            break;
          default:
            setError('An error occurred');
        }
      } else {
        setError('Network error');
      }
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {error && <Alert variant="error">{error}</Alert>}
      {/* Form fields */}
      <button disabled={isLoading}>
        {isLoading ? 'Creating...' : 'Create Issue'}
      </button>
    </form>
  );
};
```

### Backend Error Responses

```python
# Django view error handling
from rest_framework.response import Response
from rest_framework.exceptions import ValidationError, PermissionDenied

class IssueViewSet(viewsets.ModelViewSet):
    def create(self, request, *args, **kwargs):
        try:
            serializer = self.get_serializer(data=request.data)
            serializer.is_valid(raise_exception=True)

            # Check permissions
            if not self.has_create_permission(request):
                raise PermissionDenied("You don't have permission to create issues")

            issue = serializer.save()

            return Response(serializer.data, status=201)

        except ValidationError as e:
            # 400 Bad Request
            return Response({'error': str(e)}, status=400)

        except PermissionDenied as e:
            # 403 Forbidden
            return Response({'error': str(e)}, status=403)

        except Exception as e:
            # 500 Internal Server Error
            logger.error(f"Error creating issue: {str(e)}")
            return Response(
                {'error': 'An unexpected error occurred'},
                status=500
            )
```

---

## State Management Integration

### MobX Store Integration

**Plane uses MobX for state management.**

```typescript
// stores/issue.store.ts
import { makeObservable, observable, action, runInAction } from 'mobx';
import { issueService } from '@/services/issue.service';

class IssueStore {
  issues: Map<string, Issue> = new Map();
  isLoading = false;
  error: string | null = null;

  constructor() {
    makeObservable(this, {
      issues: observable,
      isLoading: observable,
      error: observable,
      fetchIssues: action,
      createIssue: action,
      updateIssue: action,
      deleteIssue: action
    });
  }

  async fetchIssues(workspaceSlug: string, projectId: string) {
    this.isLoading = true;
    this.error = null;

    try {
      const issues = await issueService.getAll(workspaceSlug, projectId);

      runInAction(() => {
        issues.forEach(issue => {
          this.issues.set(issue.id, issue);
        });
        this.isLoading = false;
      });

    } catch (error) {
      runInAction(() => {
        this.error = 'Failed to load issues';
        this.isLoading = false;
      });
    }
  }

  async createIssue(
    workspaceSlug: string,
    projectId: string,
    data: Partial<Issue>
  ) {
    try {
      const issue = await issueService.create(workspaceSlug, projectId, data);

      runInAction(() => {
        this.issues.set(issue.id, issue);
      });

      return issue;

    } catch (error) {
      runInAction(() => {
        this.error = 'Failed to create issue';
      });
      throw error;
    }
  }

  async updateIssue(
    workspaceSlug: string,
    projectId: string,
    issueId: string,
    updates: Partial<Issue>
  ) {
    const existing = this.issues.get(issueId);

    if (!existing) return;

    try {
      const updated = await issueService.update(
        workspaceSlug,
        projectId,
        issueId,
        updates
      );

      runInAction(() => {
        this.issues.set(issueId, updated);
      });

    } catch (error) {
      runInAction(() => {
        this.error = 'Failed to update issue';
      });
    }
  }

  async deleteIssue(
    workspaceSlug: string,
    projectId: string,
    issueId: string
  ) {
    try {
      await issueService.delete(workspaceSlug, projectId, issueId);

      runInAction(() => {
        this.issues.delete(issueId);
      });

    } catch (error) {
      runInAction(() => {
        this.error = 'Failed to delete issue';
      });
    }
  }
}

export const issueStore = new IssueStore();
```

### Using Store in Components

```tsx
import { observer } from 'mobx-react-lite';
import { useStore } from '@/hooks/use-store';

const IssueList = observer(() => {
  const { issueStore } = useStore();
  const { workspaceSlug, projectId } = useParams();

  useEffect(() => {
    issueStore.fetchIssues(workspaceSlug, projectId);
  }, [workspaceSlug, projectId]);

  if (issueStore.isLoading) return <div>Loading...</div>;
  if (issueStore.error) return <div>Error: {issueStore.error}</div>;

  return (
    <div>
      {Array.from(issueStore.issues.values()).map(issue => (
        <IssueCard key={issue.id} issue={issue} />
      ))}
    </div>
  );
});
```

---

## Real-Time Updates (WebSockets)

### WebSocket Connection

**Plane uses WebSockets for real-time updates.**

```typescript
// services/websocket.service.ts
class WebSocketService {
  private socket: WebSocket | null = null;
  private handlers: Map<string, Set<Function>> = new Map();

  connect(workspaceSlug: string) {
    const token = localStorage.getItem('access_token');
    const url = `wss://api.plane.so/ws/${workspaceSlug}/?token=${token}`;

    this.socket = new WebSocket(url);

    this.socket.onopen = () => {
      console.log('WebSocket connected');
    };

    this.socket.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleMessage(data);
    };

    this.socket.onerror = (error) => {
      console.error('WebSocket error:', error);
    };

    this.socket.onclose = () => {
      console.log('WebSocket disconnected');
      // Reconnect after delay
      setTimeout(() => this.connect(workspaceSlug), 5000);
    };
  }

  private handleMessage(data: any) {
    const { type, payload } = data;

    // Notify all handlers for this message type
    const handlers = this.handlers.get(type);
    if (handlers) {
      handlers.forEach(handler => handler(payload));
    }
  }

  subscribe(type: string, handler: Function) {
    if (!this.handlers.has(type)) {
      this.handlers.set(type, new Set());
    }
    this.handlers.get(type)!.add(handler);

    // Return unsubscribe function
    return () => {
      this.handlers.get(type)?.delete(handler);
    };
  }

  send(type: string, payload: any) {
    if (this.socket && this.socket.readyState === WebSocket.OPEN) {
      this.socket.send(JSON.stringify({ type, payload }));
    }
  }

  disconnect() {
    if (this.socket) {
      this.socket.close();
      this.socket = null;
    }
  }
}

export const websocketService = new WebSocketService();
```

### Using WebSockets in Components

```tsx
import { useEffect } from 'react';
import { websocketService } from '@/services/websocket.service';

const IssueDetail = ({ issueId }: { issueId: string }) => {
  const [issue, setIssue] = useState<Issue | null>(null);

  useEffect(() => {
    // Subscribe to issue updates
    const unsubscribe = websocketService.subscribe('issue.updated', (data) => {
      if (data.id === issueId) {
        // Real-time update!
        setIssue(data);
      }
    });

    return unsubscribe;
  }, [issueId]);

  return (
    <div>
      <h1>{issue?.name}</h1>
      {/* UI automatically updates when other users edit */}
    </div>
  );
};
```

---

## File Uploads

### Frontend File Upload

```tsx
const FileUpload = () => {
  const [file, setFile] = useState<File | null>(null);
  const [isUploading, setIsUploading] = useState(false);

  const handleUpload = async () => {
    if (!file) return;

    setIsUploading(true);

    const formData = new FormData();
    formData.append('file', file);
    formData.append('entity_type', 'issue_attachment');

    try {
      const token = localStorage.getItem('access_token');

      const response = await fetch('/api/assets/', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${token}`
          // Don't set Content-Type - browser sets it with boundary
        },
        body: formData
      });

      const asset = await response.json();
      console.log('Uploaded:', asset.url);

    } catch (error) {
      console.error('Upload failed:', error);
    } finally {
      setIsUploading(false);
    }
  };

  return (
    <div>
      <input
        type="file"
        onChange={(e) => setFile(e.target.files?.[0] || null)}
      />
      <button onClick={handleUpload} disabled={isUploading}>
        {isUploading ? 'Uploading...' : 'Upload'}
      </button>
    </div>
  );
};
```

### Backend File Handling

```python
# Django view
from rest_framework.parsers import MultiPartParser
from rest_framework.response import Response

class FileAssetViewSet(viewsets.ModelViewSet):
    parser_classes = [MultiPartParser]

    def create(self, request):
        file = request.FILES.get('file')
        entity_type = request.data.get('entity_type')

        # Validate file
        if not file:
            return Response({'error': 'No file provided'}, status=400)

        # Save file
        asset = FileAsset.objects.create(
            file=file,
            entity_type=entity_type,
            created_by=request.user
        )

        return Response({
            'id': str(asset.id),
            'url': asset.file.url,
            'name': file.name,
            'size': file.size
        }, status=201)
```

---

## Optimistic Updates

### What Are Optimistic Updates?

**Update UI immediately, then sync with server.**

```tsx
const IssueCard = ({ issue }: { issue: Issue }) => {
  const [localIssue, setLocalIssue] = useState(issue);

  const toggleComplete = async () => {
    // Optimistic update (immediate UI change)
    setLocalIssue({ ...localIssue, is_completed: !localIssue.is_completed });

    try {
      // Sync with server
      const updated = await issueService.update(
        workspaceSlug,
        projectId,
        issue.id,
        { is_completed: !localIssue.is_completed }
      );

      // Update with server data
      setLocalIssue(updated);

    } catch (error) {
      // Revert on error
      setLocalIssue(issue);
      toast.error('Failed to update issue');
    }
  };

  return (
    <div className={localIssue.is_completed ? 'completed' : ''}>
      <input
        type="checkbox"
        checked={localIssue.is_completed}
        onChange={toggleComplete}
      />
      {localIssue.name}
    </div>
  );
};
```

**Benefits:**
- ✅ Instant feedback
- ✅ Feels fast
- ✅ Handles errors gracefully

---

## Complete Examples

### Example 1: Creating an Issue (End-to-End)

**Frontend: Component**

```tsx
import { useState } from 'react';
import { useNavigate } from 'react-router';
import { issueService } from '@/services/issue.service';
import { toast } from '@/components/ui/toast';

const IssueCreateForm = ({ workspaceSlug, projectId }: Props) => {
  const navigate = useNavigate();
  const [formData, setFormData] = useState({
    name: '',
    priority: 'medium' as const,
    assignee_ids: [] as string[]
  });
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);

    try {
      const issue = await issueService.create(
        workspaceSlug,
        projectId,
        formData
      );

      toast.success('Issue created!');
      navigate(`/issues/${issue.id}`);

    } catch (error) {
      toast.error('Failed to create issue');
    } finally {
      setIsSubmitting(false);
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

      <select
        value={formData.priority}
        onChange={(e) => setFormData({ ...formData, priority: e.target.value as any })}
      >
        <option value="low">Low</option>
        <option value="medium">Medium</option>
        <option value="high">High</option>
        <option value="urgent">Urgent</option>
      </select>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Creating...' : 'Create Issue'}
      </button>
    </form>
  );
};
```

**Frontend: Service**

```typescript
// services/issue.service.ts
class IssueService {
  async create(
    workspaceSlug: string,
    projectId: string,
    data: Partial<Issue>
  ): Promise<Issue> {
    const token = localStorage.getItem('access_token');

    const response = await fetch(
      `/api/workspaces/${workspaceSlug}/projects/${projectId}/issues/`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify(data)
      }
    );

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    return response.json();
  }
}
```

**Backend: View**

```python
# views/issue.py
from rest_framework import viewsets
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated

class IssueViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]
    serializer_class = IssueSerializer

    def create(self, request, workspace_slug, project_id):
        # Validate user has access to project
        project = Project.objects.get(
            id=project_id,
            workspace__slug=workspace_slug,
            members=request.user
        )

        # Create issue
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)

        issue = serializer.save(
            project=project,
            workspace=project.workspace,
            created_by=request.user
        )

        # Queue background task
        issue_activity.delay(
            issue_id=str(issue.id),
            action='created',
            user_id=str(request.user.id)
        )

        return Response(serializer.data, status=201)
```

**Backend: Serializer**

```python
# serializers/issue.py
class IssueSerializer(serializers.ModelSerializer):
    assignee_ids = serializers.ListField(write_only=True, required=False)

    class Meta:
        model = Issue
        fields = ['id', 'name', 'priority', 'assignee_ids']

    def create(self, validated_data):
        assignee_ids = validated_data.pop('assignee_ids', [])

        issue = Issue.objects.create(**validated_data)

        if assignee_ids:
            issue.assignees.set(assignee_ids)

        return issue
```

---

## Best Practices

### 1. Centralize API Logic

```tsx
// ✅ GOOD - Service layer
const issue = await issueService.create(data);

// ❌ BAD - Direct fetch in component
const response = await fetch('/api/issues/', ...);
```

### 2. Handle Errors Gracefully

```tsx
try {
  await issueService.create(data);
  toast.success('Created!');
} catch (error) {
  toast.error('Failed to create');
  console.error(error);
}
```

### 3. Use TypeScript Types

```typescript
interface Issue {
  id: string;
  name: string;
  priority: 'low' | 'medium' | 'high' | 'urgent';
}

// Type-safe API calls
const issue: Issue = await issueService.getById(id);
```

### 4. Cache Responses

```tsx
// Use SWR for automatic caching
const { data: issues } = useSWR('/api/issues/', fetcher);
```

### 5. Retry Failed Requests

```typescript
const fetchWithRetry = async (url: string, retries = 3) => {
  for (let i = 0; i < retries; i++) {
    try {
      return await fetch(url);
    } catch (error) {
      if (i === retries - 1) throw error;
      await delay(1000 * Math.pow(2, i));  // Exponential backoff
    }
  }
};
```

---

## Next Steps

### You've Learned Frontend ↔ Backend Integration! 🎉

You now understand:
- ✅ HTTP communication (REST API, status codes)
- ✅ Authentication (JWT tokens, refresh flow)
- ✅ Service layer pattern
- ✅ Error handling
- ✅ State management integration (MobX)
- ✅ Real-time updates (WebSockets)
- ✅ File uploads
- ✅ Optimistic updates

### Continue Learning

1. **[HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)** - Add new endpoints and features
2. **[CODE_TOURS.md](./CODE_TOURS.md)** - Follow real features through the stack
3. **[TESTING_GUIDE.md](./TESTING_GUIDE.md)** - Test integration points
4. **[DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)** - Debug API issues

---

**You're now ready to build full-stack features in Plane! 🚀**
