# How-To Guide - Common Development Tasks

**Step-by-step recipes for everyday tasks!** This guide provides practical, copy-paste-ready solutions for common development scenarios in Plane.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 1 hour
**📊 Difficulty Level:** All levels
**🎯 Target Audience:** Developers implementing features

---

## Table of Contents

1. [Frontend Tasks](#frontend-tasks)
2. [Backend Tasks](#backend-tasks)
3. [Database Tasks](#database-tasks)
4. [Full-Stack Features](#full-stack-features)
5. [Common Workflows](#common-workflows)

---

## Frontend Tasks

### How to Add a New React Component

**Goal:** Create a reusable component following Plane's conventions

**Step 1: Create component file**

```bash
# Location: apps/web/components/issues/
touch apps/web/components/issues/issue-priority-badge.tsx
```

**Step 2: Write component**

```tsx
// apps/web/components/issues/issue-priority-badge.tsx
import { cn } from '@/lib/utils';

interface IssuePriorityBadgeProps {
  priority: 'low' | 'medium' | 'high' | 'urgent';
  className?: string;
}

const PRIORITY_CONFIG = {
  low: { label: 'Low', color: 'bg-gray-500' },
  medium: { label: 'Medium', color: 'bg-yellow-500' },
  high: { label: 'High', color: 'bg-orange-500' },
  urgent: { label: 'Urgent', color: 'bg-red-500' }
};

export const IssuePriorityBadge = ({ priority, className }: IssuePriorityBadgeProps) => {
  const config = PRIORITY_CONFIG[priority];

  return (
    <span className={cn('px-2 py-1 rounded text-xs font-medium', config.color, className)}>
      {config.label}
    </span>
  );
};

IssuePriorityBadge.displayName = 'IssuePriorityBadge';
```

**Step 3: Use component**

```tsx
import { IssuePriorityBadge } from '@/components/issues/issue-priority-badge';

const IssueCard = ({ issue }: { issue: Issue }) => {
  return (
    <div>
      <h3>{issue.name}</h3>
      <IssuePriorityBadge priority={issue.priority} />
    </div>
  );
};
```

---

### How to Add a New Route

**Goal:** Create a new page in the app

**Step 1: Create route file**

```bash
# Location: apps/web/app/(all)/[workspaceSlug]/(projects)/projects/(detail)/[projectId]/
mkdir -p apps/web/app/(all)/[workspaceSlug]/(projects)/projects/(detail)/[projectId]/analytics
touch apps/web/app/(all)/[workspaceSlug]/(projects)/projects/(detail)/[projectId]/analytics/page.tsx
```

**Step 2: Create page component**

```tsx
// page.tsx
import { useParams } from 'react-router';
import { ProjectAnalytics } from '@/components/analytics/project-analytics';

export default function ProjectAnalyticsPage() {
  const { workspaceSlug, projectId } = useParams();

  return (
    <div>
      <h1>Project Analytics</h1>
      <ProjectAnalytics workspaceSlug={workspaceSlug} projectId={projectId} />
    </div>
  );
}
```

**Step 3: Add navigation link**

```tsx
// In project sidebar
<Link to={`/${workspaceSlug}/projects/${projectId}/analytics`}>
  Analytics
</Link>
```

---

### How to Fetch Data with SWR

**Goal:** Fetch and cache API data

```tsx
import useSWR from 'swr';

// Define fetcher
const fetcher = async (url: string) => {
  const token = localStorage.getItem('access_token');
  const response = await fetch(url, {
    headers: { Authorization: `Bearer ${token}` }
  });
  if (!response.ok) throw new Error('API error');
  return response.json();
};

// Component
const IssueList = ({ projectId }: { projectId: string }) => {
  const { data: issues, error, isLoading, mutate } = useSWR(
    `/api/projects/${projectId}/issues/`,
    fetcher,
    {
      revalidateOnFocus: false,  // Don't refetch on window focus
      refreshInterval: 30000      // Refresh every 30s
    }
  );

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <button onClick={() => mutate()}>Refresh</button>
      {issues.map(issue => (
        <IssueCard key={issue.id} issue={issue} />
      ))}
    </div>
  );
};
```

---

### How to Use MobX Store

**Goal:** Access and update global state

**Step 1: Access store**

```tsx
import { observer } from 'mobx-react-lite';
import { useStore } from '@/core/hooks/use-store';

export const IssueList = observer(() => {
  const { issueStore } = useStore();
  const { workspaceSlug, projectId } = useParams();

  useEffect(() => {
    issueStore.fetchIssues(workspaceSlug, projectId);
  }, [workspaceSlug, projectId]);

  if (issueStore.isLoading) return <div>Loading...</div>;

  return (
    <div>
      {Array.from(issueStore.issues.values()).map(issue => (
        <IssueCard key={issue.id} issue={issue} />
      ))}
    </div>
  );
});
```

**Step 2: Update store**

```tsx
const handleCreateIssue = async (data: IssueFormData) => {
  await issueStore.createIssue(workspaceSlug, projectId, data);
  // Store automatically updates, component re-renders
};
```

---

### How to Handle Form Submission

**Goal:** Create a form that submits to the API

```tsx
import { useState } from 'react';
import { useNavigate } from 'react-router';
import { toast } from '@/components/ui/toast';

const IssueCreateForm = ({ projectId }: { projectId: string }) => {
  const navigate = useNavigate();
  const [formData, setFormData] = useState({
    name: '',
    priority: 'medium' as const
  });
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [errors, setErrors] = useState<Record<string, string>>({});

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setErrors({});
    setIsSubmitting(true);

    try {
      const issue = await issueService.create(projectId, formData);
      toast.success('Issue created!');
      navigate(`/issues/${issue.id}`);
    } catch (error) {
      if (error instanceof ValidationError) {
        setErrors(error.errors);
      } else {
        toast.error('Failed to create issue');
      }
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          value={formData.name}
          onChange={(e) => setFormData({ ...formData, name: e.target.value })}
          placeholder="Issue name"
          required
        />
        {errors.name && <span className="error">{errors.name}</span>}
      </div>

      <div>
        <select
          value={formData.priority}
          onChange={(e) => setFormData({ ...formData, priority: e.target.value as any })}
        >
          <option value="low">Low</option>
          <option value="medium">Medium</option>
          <option value="high">High</option>
          <option value="urgent">Urgent</option>
        </select>
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Creating...' : 'Create Issue'}
      </button>
    </form>
  );
};
```

---

## Backend Tasks

### How to Create a New Django Model

**Goal:** Add a new database table

**Step 1: Create model file**

```bash
# Location: apps/api/plane/db/models/
touch apps/api/plane/db/models/comment.py
```

**Step 2: Define model**

```python
# apps/api/plane/db/models/comment.py
from django.db import models
from django.conf import settings
from .base import BaseModel

class Comment(BaseModel):
    """Comments on issues, pages, etc."""

    # Foreign keys
    issue = models.ForeignKey(
        'db.Issue',
        on_delete=models.CASCADE,
        related_name='comments'
    )
    author = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name='authored_comments'
    )

    # Content
    content = models.TextField()
    content_html = models.TextField(blank=True)

    # Metadata
    is_edited = models.BooleanField(default=False)
    edited_at = models.DateTimeField(null=True, blank=True)

    class Meta:
        db_table = 'db_comment'
        ordering = ['-created_at']

    def __str__(self):
        return f"Comment by {self.author.username} on {self.issue.name}"
```

**Step 3: Register model**

```python
# apps/api/plane/db/models/__init__.py
from .comment import Comment

__all__ = [
    # ... existing models
    'Comment',
]
```

**Step 4: Create migration**

```bash
cd apps/api
python manage.py makemigrations
# Creates: plane/db/migrations/0XXX_create_comment.py

python manage.py migrate
# Applies migration to database
```

---

### How to Add a Model Field

**Goal:** Add a new field to existing model

**Step 1: Add field to model**

```python
# apps/api/plane/db/models/issue.py
class Issue(BaseModel):
    # ... existing fields ...

    # New field
    estimated_hours = models.DecimalField(
        max_digits=6,
        decimal_places=2,
        null=True,
        blank=True,
        help_text="Estimated time in hours"
    )
```

**Step 2: Create migration**

```bash
python manage.py makemigrations
# Django asks: "You are trying to add a non-nullable field without a default"
# Choose option 2: "Provide a one-off default now"
# Enter: None (since we set null=True)

python manage.py migrate
```

**Step 3: Update serializer**

```python
# apps/api/plane/app/serializers/issue.py
class IssueSerializer(BaseSerializer):
    class Meta:
        model = Issue
        fields = [
            # ... existing fields ...
            'estimated_hours',  # Add new field
        ]
```

---

### How to Create an API Endpoint

**Goal:** Add a custom endpoint to a ViewSet

**Step 1: Add method to ViewSet**

```python
# apps/api/plane/app/views/issue.py
from rest_framework.decorators import action
from rest_framework.response import Response

class IssueViewSet(BaseViewSet):
    # ... existing methods ...

    @action(detail=True, methods=['POST'])
    def duplicate(self, request, workspace_slug, project_id, pk):
        """
        Duplicate an issue.
        POST /api/workspaces/{slug}/projects/{id}/issues/{pk}/duplicate/
        """
        # Get original issue
        original = self.get_object()

        # Create duplicate
        duplicate = Issue.objects.create(
            project=original.project,
            workspace=original.workspace,
            name=f"{original.name} (Copy)",
            description=original.description,
            priority=original.priority,
            created_by=request.user
        )

        # Copy assignees
        duplicate.assignees.set(original.assignees.all())

        # Serialize and return
        serializer = self.get_serializer(duplicate)
        return Response(serializer.data, status=201)
```

**Step 2: Use endpoint from frontend**

```typescript
// services/issue.service.ts
async duplicateIssue(
  workspaceSlug: string,
  projectId: string,
  issueId: string
): Promise<Issue> {
  return this.api.post(
    `/workspaces/${workspaceSlug}/projects/${projectId}/issues/${issueId}/duplicate/`
  );
}
```

---

### How to Add Query Filtering

**Goal:** Allow filtering issues by custom parameters

**Step 1: Add filter to ViewSet**

```python
from rest_framework import filters
from django_filters import rest_framework as django_filters

class IssueFilterSet(django_filters.FilterSet):
    # Exact match
    priority = django_filters.CharFilter(field_name='priority')

    # Contains (case-insensitive)
    name = django_filters.CharFilter(
        field_name='name',
        lookup_expr='icontains'
    )

    # Date range
    created_after = django_filters.DateFilter(
        field_name='created_at',
        lookup_expr='gte'
    )

    # Multiple choice
    priority__in = django_filters.MultipleChoiceFilter(
        field_name='priority',
        choices=Issue.PRIORITY_CHOICES
    )

    class Meta:
        model = Issue
        fields = ['priority', 'name', 'created_after', 'priority__in']

class IssueViewSet(BaseViewSet):
    filterset_class = IssueFilterSet
    filter_backends = [django_filters.DjangoFilterBackend, filters.OrderingFilter]
    ordering_fields = ['created_at', 'priority', 'name']
```

**Step 2: Use from frontend**

```typescript
// GET /api/issues/?priority=high&name=bug&ordering=-created_at
const issues = await issueService.getAll({
  priority: 'high',
  name: 'bug',
  ordering: '-created_at'
});
```

---

## Database Tasks

### How to Write a Data Migration

**Goal:** Populate or update existing data

**Step 1: Create empty migration**

```bash
python manage.py makemigrations --empty plane --name populate_default_priorities
```

**Step 2: Write migration**

```python
# plane/db/migrations/0XXX_populate_default_priorities.py
from django.db import migrations

def populate_priorities(apps, schema_editor):
    Issue = apps.get_model('db', 'Issue')

    # Update all issues with None priority to 'medium'
    Issue.objects.filter(priority__isnull=True).update(priority='medium')

def reverse_priorities(apps, schema_editor):
    # Optional: reverse the migration
    pass

class Migration(migrations.Migration):
    dependencies = [
        ('db', '0XXX_previous_migration'),
    ]

    operations = [
        migrations.RunPython(populate_priorities, reverse_priorities),
    ]
```

**Step 3: Apply migration**

```bash
python manage.py migrate
```

---

### How to Create a Database Index

**Goal:** Speed up queries on specific fields

**Step 1: Add index in model**

```python
class Issue(BaseModel):
    name = models.CharField(max_length=255, db_index=True)  # Single-column index

    class Meta:
        indexes = [
            # Composite index
            models.Index(fields=['project', 'priority'], name='idx_project_priority'),

            # Partial index (PostgreSQL only)
            models.Index(
                fields=['created_at'],
                name='idx_active_created',
                condition=models.Q(deleted_at__isnull=True)
            ),
        ]
```

**Step 2: Create migration**

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## Full-Stack Features

### How to Implement a Complete Feature (End-to-End)

**Goal:** Add "Issue Comments" feature

**Step 1: Backend - Create model**

```python
# apps/api/plane/db/models/comment.py
class Comment(BaseModel):
    issue = models.ForeignKey('db.Issue', on_delete=models.CASCADE, related_name='comments')
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    content = models.TextField()
```

**Step 2: Backend - Create serializer**

```python
# apps/api/plane/app/serializers/comment.py
class CommentSerializer(BaseSerializer):
    author = UserLiteSerializer(read_only=True)

    class Meta:
        model = Comment
        fields = ['id', 'content', 'author', 'created_at']
        read_only_fields = ['author', 'created_at']
```

**Step 3: Backend - Create view**

```python
# apps/api/plane/app/views/comment.py
class CommentViewSet(BaseViewSet):
    serializer_class = CommentSerializer
    model = Comment

    def get_queryset(self):
        return Comment.objects.filter(
            issue__project_id=self.kwargs['project_id']
        ).select_related('author', 'issue')

    def perform_create(self, serializer):
        serializer.save(
            issue_id=self.kwargs['issue_id'],
            author=self.request.user
        )
```

**Step 4: Backend - Add URL**

```python
# apps/api/plane/app/urls.py
router.register(
    r'workspaces/(?P<workspace_slug>[\w-]+)/projects/(?P<project_id>[^/.]+)/issues/(?P<issue_id>[^/.]+)/comments',
    CommentViewSet,
    basename='comment'
)
```

**Step 5: Frontend - Create service**

```typescript
// services/comment.service.ts
class CommentService {
  async getAll(projectId: string, issueId: string): Promise<Comment[]> {
    return this.api.get(`/projects/${projectId}/issues/${issueId}/comments/`);
  }

  async create(projectId: string, issueId: string, content: string): Promise<Comment> {
    return this.api.post(`/projects/${projectId}/issues/${issueId}/comments/`, { content });
  }
}
```

**Step 6: Frontend - Create component**

```tsx
// components/comments/comment-list.tsx
export const CommentList = ({ projectId, issueId }: Props) => {
  const { data: comments, mutate } = useSWR(
    `/api/projects/${projectId}/issues/${issueId}/comments/`,
    fetcher
  );

  const handleSubmit = async (content: string) => {
    await commentService.create(projectId, issueId, content);
    mutate(); // Refresh comments
  };

  return (
    <div>
      <CommentForm onSubmit={handleSubmit} />
      {comments?.map(comment => (
        <CommentCard key={comment.id} comment={comment} />
      ))}
    </div>
  );
};
```

---

## Common Workflows

### How to Debug a Failed Request

**Step 1: Check browser Network tab**

- Open DevTools → Network tab
- Find failed request
- Check status code, headers, response

**Step 2: Check Django logs**

```bash
# Terminal running Django server
# Look for error stack traces
```

**Step 3: Add logging**

```python
# Backend
import logging
logger = logging.getLogger(__name__)

def my_view(request):
    logger.info(f"Request data: {request.data}")
    logger.error(f"Error occurred: {str(e)}")
```

```typescript
// Frontend
console.log('Request data:', data);
console.error('API error:', error);
```

---

### How to Test Changes Locally

**Step 1: Start backend**

```bash
cd apps/api
python manage.py runserver
# Backend runs on http://localhost:8000
```

**Step 2: Start frontend**

```bash
cd apps/web
pnpm dev
# Frontend runs on http://localhost:3000
```

**Step 3: Test in browser**

- Open http://localhost:3000
- Open DevTools → Console and Network tabs
- Test your feature
- Check for errors

---

### How to Reset Database

**Goal:** Start with a fresh database

```bash
cd apps/api

# Delete database
rm db.sqlite3  # If using SQLite
# OR
dropdb plane  # If using PostgreSQL
createdb plane

# Rerun migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser
```

---

## Next Steps

### You've Learned Common Tasks! 🎉

You now know how to:
- ✅ Add React components and routes
- ✅ Create Django models and endpoints
- ✅ Write migrations
- ✅ Implement full-stack features
- ✅ Debug issues

### Continue Learning

1. **[CODE_TOURS.md](./CODE_TOURS.md)** - Follow features through the codebase
2. **[TESTING_GUIDE.md](./TESTING_GUIDE.md)** - Test your changes
3. **[DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)** - Advanced debugging

---

**You're now ready to build features in Plane! 🚀**
