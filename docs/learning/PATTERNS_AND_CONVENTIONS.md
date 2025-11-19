# Patterns and Conventions - Code Standards in Plane

**Write clean, consistent code!** This guide covers naming conventions, file organization, and code patterns used throughout the Plane codebase.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 45-60 minutes
**📊 Difficulty Level:** Intermediate
**🎯 Target Audience:** Developers wanting to write consistent, maintainable code

---

## Table of Contents

1. [TypeScript Conventions](#typescript-conventions)
2. [React Component Patterns](#react-component-patterns)
3. [File Organization](#file-organization)
4. [Naming Conventions](#naming-conventions)
5. [Import Order](#import-order)
6. [Python/Django Conventions](#pythondjango-conventions)
7. [API Design Patterns](#api-design-patterns)
8. [State Management Patterns](#state-management-patterns)
9. [Testing Patterns](#testing-patterns)
10. [Comments and Documentation](#comments-and-documentation)

---

## TypeScript Conventions

### Type Definitions ✅ CURRENT

```typescript
// ✅ GOOD - Use interfaces for objects
interface User {
  id: string;
  email: string;
  display_name: string;
}

// ✅ GOOD - Use type for unions, intersections
type Priority = 'low' | 'medium' | 'high' | 'urgent';
type IssueStatus = 'open' | 'in_progress' | 'completed';

// ✅ GOOD - Use enums for constants with behavior
enum IssueType {
  Bug = 'bug',
  Feature = 'feature',
  Task = 'task'
}

// ❌ BAD - Don't use any
function process(data: any) { }  // NO!

// ✅ GOOD - Use specific types or unknown
function process(data: Issue | Project) { }
function process(data: unknown) { }
```

### Null vs Undefined

```typescript
// ✅ GOOD - Use undefined for optional properties
interface Issue {
  id: string;
  parent_id?: string;  // undefined if not set
}

// ✅ GOOD - Use null for API responses (explicit absence)
interface IssueResponse {
  id: string;
  parent_id: string | null;  // null from backend
}

// ✅ GOOD - Use optional chaining
const parentName = issue.parent?.name ?? 'No parent';
```

### Type Guards

```typescript
// ✅ GOOD - Type guard functions
function isIssue(item: Issue | Project): item is Issue {
  return 'sequence_id' in item;
}

if (isIssue(item)) {
  console.log(item.sequence_id);  // TypeScript knows it's Issue
}
```

### Generics

```typescript
// ✅ GOOD - Generic service
class APIService<T> {
  async get(id: string): Promise<T> {
    const response = await fetch(`/api/${id}`);
    return response.json();
  }
}

const issueService = new APIService<Issue>();
const issue = await issueService.get('123');  // Type is Issue
```

---

## React Component Patterns

### Component Structure ✅ CURRENT

```tsx
// ✅ GOOD - Function component with TypeScript
import { useState, useEffect } from 'react';
import { observer } from 'mobx-react-lite';

interface IssueCardProps {
  issue: Issue;
  onUpdate?: (issue: Issue) => void;
  className?: string;
}

export const IssueCard = observer(({ issue, onUpdate, className }: IssueCardProps) => {
  const [isEditing, setIsEditing] = useState(false);

  useEffect(() => {
    // Side effects here
  }, [issue.id]);

  const handleSave = async () => {
    // Handle save
    onUpdate?.(issue);
  };

  return (
    <div className={cn('issue-card', className)}>
      {/* Component JSX */}
    </div>
  );
});

IssueCard.displayName = 'IssueCard';
```

### Props Naming

```tsx
// ✅ GOOD - Clear, descriptive prop names
interface Props {
  issue: Issue;                    // Entity names
  onUpdate: (issue: Issue) => void;  // Event handlers (on*)
  isLoading: boolean;                // Boolean (is*, has*, can*)
  className?: string;                // Optional styling
  children?: React.ReactNode;        // Child elements
}

// ❌ BAD - Unclear names
interface Props {
  data: any;           // Too vague
  click: () => void;   // Missing 'on' prefix
  loading: boolean;    // Missing 'is' prefix
}
```

### Component Organization

```tsx
// ✅ GOOD - Organized component file
// 1. Imports
import { useState } from 'react';
import { observer } from 'mobx-react-lite';

// 2. Types/Interfaces
interface Props {
  issue: Issue;
}

// 3. Constants
const PRIORITY_COLORS = {
  low: '#green',
  high: '#red'
};

// 4. Component
export const IssueCard = observer(({ issue }: Props) => {
  // 4a. Hooks
  const [isOpen, setIsOpen] = useState(false);
  const { issueStore } = useStore();

  // 4b. Computed values
  const priorityColor = PRIORITY_COLORS[issue.priority];

  // 4c. Event handlers
  const handleClick = () => {
    setIsOpen(true);
  };

  // 4d. Effects
  useEffect(() => {
    // Effect logic
  }, []);

  // 4e. Render helpers
  const renderPriority = () => (
    <span style={{ color: priorityColor }}>{issue.priority}</span>
  );

  // 4f. Return JSX
  return (
    <div onClick={handleClick}>
      {renderPriority()}
    </div>
  );
});
```

### Custom Hooks

```tsx
// ✅ GOOD - Extract reusable logic to hooks
function useIssues(projectId: string) {
  const [issues, setIssues] = useState<Issue[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchIssues = async () => {
      setIsLoading(true);
      try {
        const data = await issueService.getAll(projectId);
        setIssues(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setIsLoading(false);
      }
    };

    fetchIssues();
  }, [projectId]);

  return { issues, isLoading, error };
}

// Usage
const { issues, isLoading, error } = useIssues(projectId);
```

### Conditional Rendering

```tsx
// ✅ GOOD - Early returns
if (isLoading) return <Spinner />;
if (error) return <ErrorMessage error={error} />;
if (!issues.length) return <EmptyState />;

return <IssueList issues={issues} />;

// ✅ GOOD - Ternary for simple conditions
{isEditing ? <EditForm /> : <DisplayView />}

// ✅ GOOD - && for show/hide
{hasPermission && <DeleteButton />}

// ❌ BAD - Nested ternaries
{isLoading ? <Spinner /> : error ? <Error /> : issues ? <List /> : <Empty />}
```

---

## File Organization

### Directory Structure ✅ CURRENT

```
apps/web/
├── app/                          # React Router routes
│   ├── (all)/                   # Route group
│   │   └── [workspaceSlug]/
│   │       └── projects/
│   │           └── [projectId]/
│   │               ├── issues/
│   │               │   ├── page.tsx      # Route component
│   │               │   ├── layout.tsx    # Layout
│   │               │   └── header.tsx    # Header
│   │               └── settings/
│   └── routes.ts                # Route config
│
├── components/                   # Shared components
│   ├── ui/                      # UI primitives
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   └── modal.tsx
│   ├── issues/                  # Feature components
│   │   ├── issue-card.tsx
│   │   ├── issue-list.tsx
│   │   └── issue-form.tsx
│   └── common/                  # Common components
│       ├── header.tsx
│       └── sidebar.tsx
│
├── core/                        # Core functionality
│   ├── hooks/                   # Custom hooks
│   │   ├── use-store.ts
│   │   └── use-issues.ts
│   ├── services/                # API services
│   │   ├── api.service.ts
│   │   └── issue.service.ts
│   └── store/                   # MobX stores
│       ├── root.store.ts
│       └── issue.store.ts
│
├── lib/                         # Utilities
│   ├── utils.ts
│   └── constants.ts
│
└── types/                       # TypeScript types
    └── issue.d.ts
```

### File Naming

```
// ✅ GOOD - Kebab-case for files
issue-card.tsx
issue-list.tsx
use-issues.ts

// ✅ GOOD - Match component name
export const IssueCard  →  issue-card.tsx
export const IssueList  →  issue-list.tsx

// ✅ GOOD - Descriptive names
user-profile-settings.tsx    // Not user.tsx
issue-create-form.tsx        // Not form.tsx
```

---

## Naming Conventions

### TypeScript/React

```typescript
// PascalCase - Components, Interfaces, Types, Classes
class IssueService { }
interface Issue { }
type IssueStatus = 'open' | 'closed';
const IssueCard = () => { };

// camelCase - Variables, functions, methods, props
const issueCount = 10;
function getIssue() { }
const handleClick = () => { };

// UPPER_SNAKE_CASE - Constants
const API_BASE_URL = 'https://api.plane.so';
const MAX_FILE_SIZE = 10_000_000;

// kebab-case - File names, CSS classes
issue-card.tsx
<div className="issue-card" />
```

### Python/Django

```python
# snake_case - Everything except classes
def get_issue():
    issue_count = 10
    return issue_count

# PascalCase - Classes
class IssueSerializer:
    pass

class IssueViewSet:
    pass

# UPPER_SNAKE_CASE - Constants
API_VERSION = 'v1'
MAX_ISSUES = 100
```

### Boolean Names

```typescript
// ✅ GOOD - Use is/has/can/should prefixes
isLoading
hasPermission
canEdit
shouldUpdate

// ❌ BAD - Unclear
loading  // Is it loading or the loading component?
permission  // Boolean or permission object?
```

### Event Handlers

```typescript
// ✅ GOOD - Use handle* prefix
const handleClick = () => { };
const handleSubmit = () => { };
const handleIssueUpdate = (issue: Issue) => { };

// ✅ GOOD - Props use on* prefix
interface Props {
  onClick: () => void;
  onSubmit: (data: FormData) => void;
  onIssueUpdate: (issue: Issue) => void;
}
```

---

## Import Order

### React Import Order ✅ CURRENT

```tsx
// 1. External libraries (React, third-party)
import { useState, useEffect } from 'react';
import { observer } from 'mobx-react-lite';
import { useParams, useNavigate } from 'react-router';

// 2. Internal core (services, stores, hooks)
import { useStore } from '@/core/hooks/use-store';
import { issueService } from '@/core/services/issue.service';

// 3. Components
import { Button } from '@/components/ui/button';
import { IssueCard } from '@/components/issues/issue-card';

// 4. Types
import type { Issue, IssueFormData } from '@/types/issue';

// 5. Utilities & constants
import { cn } from '@/lib/utils';
import { ISSUE_PRIORITIES } from '@/lib/constants';

// 6. Styles
import './issue-list.css';
```

### Python Import Order

```python
# 1. Python standard library
import json
import uuid
from datetime import datetime

# 2. Django imports
from django.db import models
from django.contrib.auth import get_user_model

# 3. Third-party imports
from rest_framework import serializers
from rest_framework.response import Response

# 4. Local imports
from plane.db.models import Issue, Project
from plane.app.serializers import IssueSerializer
```

---

## Python/Django Conventions

### Model Naming

```python
# ✅ GOOD - Singular names
class Issue(models.Model):
    pass

class User(models.Model):
    pass

# ❌ BAD - Plural names
class Issues(models.Model):  # NO!
    pass
```

### Field Naming

```python
class Issue(models.Model):
    # ✅ GOOD - Descriptive snake_case names
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    sequence_id = models.IntegerField()

    # ✅ GOOD - Boolean fields use is_/has_ prefix
    is_draft = models.BooleanField(default=False)
    is_completed = models.BooleanField(default=False)

    # ✅ GOOD - Foreign key naming
    project = models.ForeignKey(Project)      # Field name
    project_id  # Django creates this automatically

    # ✅ GOOD - Many-to-many naming (plural)
    assignees = models.ManyToManyField(User)
    labels = models.ManyToManyField(Label)
```

### ViewSet Methods

```python
class IssueViewSet(viewsets.ModelViewSet):
    # Standard CRUD methods
    def list(self, request):       # GET /issues/
        pass

    def create(self, request):     # POST /issues/
        pass

    def retrieve(self, request, pk):  # GET /issues/{pk}/
        pass

    def update(self, request, pk):    # PUT /issues/{pk}/
        pass

    def partial_update(self, request, pk):  # PATCH /issues/{pk}/
        pass

    def destroy(self, request, pk):   # DELETE /issues/{pk}/
        pass

    # Custom actions use @action decorator
    @action(detail=False, methods=['POST'])
    def bulk_create(self, request):   # POST /issues/bulk_create/
        pass

    @action(detail=True, methods=['GET'])
    def sub_issues(self, request, pk):  # GET /issues/{pk}/sub_issues/
        pass
```

---

## API Design Patterns

### RESTful URL Structure ✅ CURRENT

```
# ✅ GOOD - Hierarchical, resource-based URLs
GET    /api/workspaces/{slug}/projects/
POST   /api/workspaces/{slug}/projects/
GET    /api/workspaces/{slug}/projects/{id}/
PATCH  /api/workspaces/{slug}/projects/{id}/
DELETE /api/workspaces/{slug}/projects/{id}/

GET    /api/workspaces/{slug}/projects/{id}/issues/
POST   /api/workspaces/{slug}/projects/{id}/issues/
GET    /api/workspaces/{slug}/projects/{id}/issues/{issue_id}/
PATCH  /api/workspaces/{slug}/projects/{id}/issues/{issue_id}/
DELETE /api/workspaces/{slug}/projects/{id}/issues/{issue_id}/

# ❌ BAD - Flat, action-based URLs
POST   /api/createIssue/
GET    /api/getIssues/
POST   /api/updateIssue/123/
```

### Request/Response Formats

```typescript
// ✅ GOOD - Consistent response format
interface APIResponse<T> {
  data: T;
  meta?: {
    page: number;
    total: number;
  };
}

// ✅ GOOD - Error response format
interface APIError {
  error: string;
  code?: string;
  details?: Record<string, string[]>;
}

// Example
{
  "error": "Validation failed",
  "code": "VALIDATION_ERROR",
  "details": {
    "name": ["This field is required"],
    "priority": ["Invalid choice"]
  }
}
```

---

## State Management Patterns

### MobX Store Structure ✅ CURRENT

```typescript
import { makeObservable, observable, action, computed } from 'mobx';

class IssueStore {
  // Observable state
  issues: Map<string, Issue> = new Map();
  isLoading = false;
  error: string | null = null;

  constructor() {
    makeObservable(this, {
      // Observables
      issues: observable,
      isLoading: observable,
      error: observable,

      // Actions
      fetchIssues: action,
      addIssue: action,
      updateIssue: action,

      // Computed
      issueCount: computed,
      highPriorityIssues: computed
    });
  }

  // Computed values
  get issueCount() {
    return this.issues.size;
  }

  get highPriorityIssues() {
    return Array.from(this.issues.values())
      .filter(i => i.priority === 'high');
  }

  // Actions (mutations)
  async fetchIssues() {
    this.isLoading = true;
    try {
      const issues = await issueService.getAll();
      runInAction(() => {
        issues.forEach(issue => this.issues.set(issue.id, issue));
        this.isLoading = false;
      });
    } catch (error) {
      runInAction(() => {
        this.error = error.message;
        this.isLoading = false;
      });
    }
  }

  addIssue(issue: Issue) {
    this.issues.set(issue.id, issue);
  }

  updateIssue(id: string, updates: Partial<Issue>) {
    const issue = this.issues.get(id);
    if (issue) {
      this.issues.set(id, { ...issue, ...updates });
    }
  }
}
```

---

## Testing Patterns

### Test Naming

```typescript
// ✅ GOOD - Descriptive test names
describe('IssueCard', () => {
  it('renders issue title and priority', () => { });
  it('calls onUpdate when edit button is clicked', () => { });
  it('shows loading state while saving', () => { });
  it('displays error message when save fails', () => { });
});

// ❌ BAD - Vague test names
describe('IssueCard', () => {
  it('works', () => { });
  it('test', () => { });
});
```

### Test Structure (AAA Pattern)

```typescript
it('updates issue priority', async () => {
  // Arrange - Set up test data
  const issue = { id: '1', priority: 'low' };
  const onUpdate = jest.fn();

  // Act - Perform action
  render(<IssueCard issue={issue} onUpdate={onUpdate} />);
  fireEvent.click(screen.getByText('Set High Priority'));

  // Assert - Check results
  await waitFor(() => {
    expect(onUpdate).toHaveBeenCalledWith(
      expect.objectContaining({ priority: 'high' })
    );
  });
});
```

---

## Comments and Documentation

### When to Comment

```typescript
// ✅ GOOD - Explain WHY, not WHAT
// We use exponential backoff to avoid overwhelming the server
// during high traffic periods
const delay = Math.pow(2, retryCount) * 1000;

// ✅ GOOD - Document complex logic
/**
 * Calculates issue sort order using lexicographic midpoint algorithm.
 * This allows efficient reordering without updating all issues.
 *
 * @param prevSortOrder - Sort order of previous issue (or null if first)
 * @param nextSortOrder - Sort order of next issue (or null if last)
 * @returns New sort order value between prev and next
 */
function calculateSortOrder(
  prevSortOrder: number | null,
  nextSortOrder: number | null
): number {
  // Implementation...
}

// ❌ BAD - State the obvious
// Increment count by 1
count++;

// ❌ BAD - Commented-out code
// const oldImplementation = () => { ... };
```

### JSDoc for Functions

```typescript
/**
 * Creates a new issue in the specified project.
 *
 * @param projectId - The project ID where issue will be created
 * @param data - Issue data including name, priority, assignees
 * @returns Promise resolving to the created issue
 * @throws {ValidationError} If issue data is invalid
 * @throws {PermissionError} If user lacks create permission
 *
 * @example
 * ```ts
 * const issue = await createIssue('proj-1', {
 *   name: 'Fix bug',
 *   priority: 'high'
 * });
 * ```
 */
async function createIssue(
  projectId: string,
  data: IssueFormData
): Promise<Issue> {
  // Implementation
}
```

---

## Best Practices Summary

### TypeScript
- ✅ Use strict TypeScript (no `any`)
- ✅ Prefer interfaces over types for objects
- ✅ Use type guards for narrowing
- ✅ Leverage utility types (`Partial`, `Pick`, `Omit`)

### React
- ✅ Use function components with hooks
- ✅ Extract reusable logic to custom hooks
- ✅ Keep components small and focused
- ✅ Use `observer` from mobx-react-lite

### File Organization
- ✅ Group by feature, not by type
- ✅ Colocate related files
- ✅ Use index files for clean imports

### Code Quality
- ✅ Write self-documenting code
- ✅ Follow consistent naming
- ✅ Keep functions small
- ✅ Avoid deep nesting

### Testing
- ✅ Write tests for business logic
- ✅ Test user interactions, not implementation
- ✅ Use descriptive test names

---

## Next Steps

### You've Learned Plane's Code Standards! 🎉

You now understand:
- ✅ TypeScript conventions
- ✅ React component patterns
- ✅ File organization
- ✅ Naming conventions
- ✅ Import order
- ✅ Python/Django standards
- ✅ API design patterns

### Continue Learning

1. **[HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)** - Apply these patterns in practice
2. **[CODE_TOURS.md](./CODE_TOURS.md)** - See patterns in real code
3. **[TESTING_GUIDE.md](./TESTING_GUIDE.md)** - Testing patterns and strategies

---

**You're now ready to write clean, consistent code in Plane! 🚀**
