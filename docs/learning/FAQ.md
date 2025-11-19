# Frequently Asked Questions

**Quick Answers!** Common questions from developers learning the Plane codebase, with detailed explanations.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 30-40 minutes
**🎯 Target Audience:** All developers working with Plane

---

## Table of Contents

### Getting Started
1. [How do I set up the development environment?](#how-do-i-set-up-the-development-environment)
2. [What do I need to know before contributing?](#what-do-i-need-to-know-before-contributing)
3. [Which documentation should I read first?](#which-documentation-should-i-read-first)

### Architecture & Design
4. [Why did Plane choose React Router 7 over Next.js?](#why-react-router-7-over-nextjs)
5. [Why MobX instead of Redux or Zustand?](#why-mobx-instead-of-redux)
6. [Why Django instead of Node.js for the backend?](#why-django-instead-of-nodejs)
7. [What's the difference between `core/` and `ee/` directories?](#core-vs-ee-directories)

### Development Workflow
8. [How do I add a new React component?](#how-do-i-add-a-new-component)
9. [How do I create a new API endpoint?](#how-do-i-create-a-new-api-endpoint)
10. [How do I add a new field to an existing model?](#how-do-i-add-a-new-field-to-a-model)
11. [How do I test my changes?](#how-do-i-test-my-changes)

### Code Organization
12. [Where should I put new code?](#where-should-i-put-new-code)
13. [How do imports work in the monorepo?](#how-do-imports-work)
14. [When should I create a shared package vs app-specific code?](#shared-vs-app-specific-code)

### Data Flow
15. [How does data flow from frontend to backend?](#data-flow-frontend-to-backend)
16. [When should I use SWR vs MobX vs React state?](#swr-vs-mobx-vs-react-state)
17. [How do real-time updates work?](#how-do-real-time-updates-work)

### Performance
18. [How can I optimize slow components?](#optimize-slow-components)
19. [How do I prevent unnecessary re-renders?](#prevent-unnecessary-re-renders)
20. [What causes N+1 database queries and how do I fix them?](#n1-queries)

### Common Issues
21. [TypeScript errors after updating a type](#typescript-errors-after-updating-type)
22. [CORS errors when calling the API](#cors-errors)
23. [Database migration conflicts](#migration-conflicts)
24. [Build failures in CI/CD](#build-failures-ci-cd)

---

## Getting Started

### How do I set up the development environment?

**Quick Answer:** Follow [GETTING_STARTED.md](./GETTING_STARTED.md)

**Detailed Steps:**

1. **Prerequisites:**
   ```bash
   node >= 22.18.0
   pnpm >= 10.21.0
   python >= 3.12
   postgresql >= 14
   redis >= 7
   ```

2. **Clone and Install:**
   ```bash
   git clone https://github.com/makeplane/plane.git
   cd plane
   pnpm install
   ```

3. **Setup Backend:**
   ```bash
   cd apps/api
   cp .env.example .env
   python -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   python manage.py migrate
   python manage.py runserver
   ```

4. **Setup Frontend:**
   ```bash
   cd apps/web
   cp .env.example .env
   pnpm dev
   ```

**Common Issues:**
- **Port already in use:** Change port in `.env`
- **Database connection failed:** Check PostgreSQL is running: `pg_isready`
- **Redis connection failed:** Check Redis is running: `redis-cli ping`

---

### What do I need to know before contributing?

**Required Knowledge:**

**Frontend:**
- ✅ React 18 (hooks, context, suspense)
- ✅ TypeScript (types, generics, discriminated unions)
- ✅ Modern CSS (Tailwind preferred)

**Backend (if contributing to API):**
- ✅ Python basics
- ✅ REST API concepts
- ✅ Database fundamentals (SQL)

**Helpful but not required:**
- MobX (can learn while contributing)
- Django (has great documentation)
- React Router 7 (similar to Next.js)

**Read These First:**
1. [README.md](./README.md) - Overview
2. [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md) - System design
3. [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md) - Where things live

---

### Which documentation should I read first?

**Learning Paths by Role:**

**Frontend Developer (React focus):**
1. [GETTING_STARTED.md](./GETTING_STARTED.md)
2. [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md)
3. [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md)
4. [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)

**Backend Developer (Django focus):**
1. [GETTING_STARTED.md](./GETTING_STARTED.md)
2. [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)
3. [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md)
4. [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)

**Full-Stack Developer:**
1. [GETTING_STARTED.md](./GETTING_STARTED.md)
2. [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)
3. [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md)
4. [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md)

---

## Architecture & Design

### Why React Router 7 over Next.js?

**Short Answer:** More flexibility, better TypeScript support, no vendor lock-in.

**Detailed Comparison:**

| Feature | Next.js 15 | React Router 7 | Winner |
|---------|-----------|----------------|---------|
| File-based routing | ✅ | ✅ | Tie |
| TypeScript inference | Good | Excellent | RR7 |
| Server components | ✅ | ❌ | Next.js |
| Loaders/Actions | ❌ | ✅ | RR7 |
| Bundle size | Large | Small | RR7 |
| Learning curve | Medium | Low | RR7 |
| Flexibility | Medium | High | RR7 |
| Community size | Huge | Growing | Next.js |

**Key Reasons:**

1. **Better TypeScript:** `useLoaderData<typeof loader>` gives perfect type inference
2. **Simpler Mental Model:** No server/client component confusion
3. **More Control:** Less magic, more explicit
4. **Smaller Bundles:** React Router 7 generates smaller bundles
5. **No Lock-in:** Can switch to other frameworks easily

**Trade-offs:**
- No server components (yet) - but Plane doesn't need them
- Smaller ecosystem - but growing fast
- Less built-in optimization - but Vite handles this

**See Also:** [ARCHITECTURE_OVERVIEW.md#why-react-router-7](./ARCHITECTURE_OVERVIEW.md)

---

### Why MobX instead of Redux?

**Short Answer:** Less boilerplate, automatic reactivity, better DX.

**Code Comparison:**

**Redux (verbose):**
```typescript
// Action types
const ADD_ISSUE = 'ADD_ISSUE';

// Action creators
const addIssue = (issue) => ({ type: ADD_ISSUE, payload: issue });

// Reducer
const issuesReducer = (state = [], action) => {
  switch (action.type) {
    case ADD_ISSUE:
      return [...state, action.payload];
    default:
      return state;
  }
};

// Component
const Component = () => {
  const dispatch = useDispatch();
  const issues = useSelector(state => state.issues);

  return <button onClick={() => dispatch(addIssue(newIssue))}>Add</button>;
};
```

**MobX (concise):**
```typescript
// Store
class IssueStore {
  issues = [];

  addIssue(issue) {
    this.issues.push(issue);
  }
}

// Component
const Component = observer(() => {
  const store = useIssueStore();
  return <button onClick={() => store.addIssue(newIssue)}>Add</button>;
});
```

**Pros of MobX:**
- ✅ 70% less boilerplate
- ✅ Automatic reactivity (like Vue)
- ✅ Better performance (fine-grained updates)
- ✅ Easier to learn

**Cons of MobX:**
- ⚠️ More "magic" (less explicit)
- ⚠️ Easier to make mistakes
- ⚠️ Smaller community than Redux

**When Redux Wins:**
- Time-travel debugging needed
- Redux DevTools required
- Team prefers explicit actions

**When MobX Wins (Plane's case):**
- Complex nested state
- Frequent updates
- Developer productivity priority

---

### Why Django instead of Node.js?

**Short Answer:** Batteries included, mature ORM, faster development.

**Comparison:**

| Feature | Django | Node.js/Express | Winner |
|---------|--------|-----------------|---------|
| ORM | Built-in (excellent) | TypeORM/Prisma | Django |
| Admin UI | Built-in | None | Django |
| Authentication | Built-in | DIY | Django |
| Type Safety | Python 3.12+ | TypeScript | Tie |
| Performance | Good | Excellent | Node.js |
| Learning Curve | Medium | Low | Node.js |
| Ecosystem | Mature | Huge | Node.js |

**Key Reasons for Django:**

1. **Django Admin:** Free admin UI for internal tools
2. **ORM:** Best-in-class ORM (better than TypeORM/Prisma)
3. **Batteries Included:** Auth, sessions, migrations, etc.
4. **Mature:** 18+ years of production use
5. **Type Safety:** Python 3.12+ type hints are excellent

**Trade-offs:**
- Different language from frontend (JavaScript vs Python)
- Slightly slower than Node.js (but fast enough)
- More memory usage

**See Also:** [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)

---

### Core vs EE Directories?

**Short Answer:**
- `core/` = Community Edition (open source, AGPL-3.0)
- `ee/` = Enterprise Edition (paid features)

**File Structure:**
```
apps/web/
├── core/              # Open source features
│   ├── components/    # Base components
│   ├── hooks/         # Base hooks
│   └── store/         # Base stores
│
└── ee/                # Enterprise features
    ├── components/    # EE-only components
    ├── hooks/         # EE-only hooks
    └── store/         # EE-only stores
```

**Examples:**

**Community Edition (core/):**
- Basic issue management
- Projects and workspaces
- User authentication
- Kanban boards

**Enterprise Edition (ee/):**
- Advanced analytics
- SAML SSO
- Custom workflows
- Advanced permissions

**Import Pattern:**

```typescript
// Import from core (always available)
import { IssueCard } from "~/core/components/issues/issue-card";

// Import from ee (only in EE builds)
import { AdvancedAnalytics } from "~/ee/components/analytics";
```

**When Contributing:**
- New basic features → `core/`
- Enterprise features → `ee/`
- Unsure? → Ask in PR

---

## Development Workflow

### How do I add a new component?

**Location:** `/apps/web/core/components/<feature>/`

**Step-by-Step:**

1. **Create Component File:**
   ```typescript
   // /apps/web/core/components/issues/issue-priority-badge.tsx
   import type { TIssuePriority } from "@plane/types";

   type Props = {
     priority: TIssuePriority;
     onChange?: (priority: TIssuePriority) => void;
   };

   export const IssuePriorityBadge = ({ priority, onChange }: Props) => {
     const colors = {
       urgent: "bg-red-500",
       high: "bg-orange-500",
       medium: "bg-yellow-500",
       low: "bg-green-500",
       none: "bg-gray-500",
     };

     return (
       <span className={`px-2 py-1 rounded ${colors[priority]}`}>
         {priority}
       </span>
     );
   };
   ```

2. **Add to Index (if creating package):**
   ```typescript
   // /apps/web/core/components/issues/index.ts
   export { IssuePriorityBadge } from "./issue-priority-badge";
   ```

3. **Use Component:**
   ```typescript
   import { IssuePriorityBadge } from "~/components/issues";

   <IssuePriorityBadge priority="high" />
   ```

**Best Practices:**
- ✅ Use TypeScript
- ✅ Export as named export
- ✅ Add prop types
- ✅ Keep components small (< 200 lines)
- ✅ Use Tailwind for styling

---

### How do I create a new API endpoint?

**Location:** `/apps/api/plane/app/views/<feature>/`

**Step-by-Step:**

1. **Create View:**
   ```python
   # /apps/api/plane/app/views/issue/issue_comment.py
   from rest_framework import status
   from rest_framework.response import Response
   from plane.app.views import BaseViewSet
   from plane.app.serializers import CommentSerializer
   from plane.db.models import Comment

   class IssueCommentViewSet(BaseViewSet):
       def list(self, request, workspace_slug, project_id, issue_id):
           """GET /api/workspaces/:slug/projects/:id/issues/:id/comments/"""
           comments = Comment.objects.filter(issue_id=issue_id)
           serializer = CommentSerializer(comments, many=True)
           return Response(serializer.data)

       def create(self, request, workspace_slug, project_id, issue_id):
           """POST /api/workspaces/:slug/projects/:id/issues/:id/comments/"""
           serializer = CommentSerializer(data=request.data)
           if serializer.is_valid():
               serializer.save(
                   issue_id=issue_id,
                   created_by=request.user
               )
               return Response(serializer.data, status=status.HTTP_201_CREATED)
           return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
   ```

2. **Add URL Route:**
   ```python
   # /apps/api/plane/app/urls.py
   from plane.app.views import IssueCommentViewSet

   urlpatterns = [
       path(
           'workspaces/<str:workspace_slug>/projects/<uuid:project_id>/issues/<uuid:issue_id>/comments/',
           IssueCommentViewSet.as_view({
               'get': 'list',
               'post': 'create'
           }),
           name='issue-comments'
       ),
   ]
   ```

3. **Create Service (Frontend):**
   ```typescript
   // /apps/web/core/services/comment.service.ts
   import { APIService } from "@plane/services";
   import type { TComment } from "@plane/types";

   export class CommentService extends APIService {
     async getComments(workspaceSlug: string, projectId: string, issueId: string): Promise<TComment[]> {
       return this.get(`/workspaces/${workspaceSlug}/projects/${projectId}/issues/${issueId}/comments/`);
     }

     async createComment(workspaceSlug: string, projectId: string, issueId: string, data: Partial<TComment>): Promise<TComment> {
       return this.post(`/workspaces/${workspaceSlug}/projects/${projectId}/issues/${issueId}/comments/`, data);
     }
   }
   ```

---

### How do I add a new field to a model?

**Step-by-Step:**

1. **Update Django Model:**
   ```python
   # /apps/api/plane/db/models/issue.py
   class Issue(models.Model):
       # ... existing fields ...

       # Add new field
       estimate_hours = models.IntegerField(null=True, blank=True)
   ```

2. **Create Migration:**
   ```bash
   cd apps/api
   python manage.py makemigrations
   # Creates: plane/db/migrations/0123_issue_estimate_hours.py
   ```

3. **Run Migration:**
   ```bash
   python manage.py migrate
   ```

4. **Update Serializer:**
   ```python
   # /apps/api/plane/app/serializers/issue.py
   class IssueSerializer(serializers.ModelSerializer):
       class Meta:
           model = Issue
           fields = [
               # ... existing fields ...
               'estimate_hours',
           ]
   ```

5. **Update TypeScript Type:**
   ```typescript
   // /packages/types/src/issue.d.ts
   export type TIssue = {
     // ... existing fields ...
     estimate_hours: number | null;
   };
   ```

6. **Rebuild Types:**
   ```bash
   pnpm --filter @plane/types build
   ```

**TypeScript will now catch errors if you forget to handle the new field!**

---

### How do I test my changes?

**Frontend Testing:**

```typescript
// /apps/web/core/components/issues/__tests__/issue-card.test.tsx
import { render, screen } from "@testing-library/react";
import { IssueCard } from "../issue-card";

describe("IssueCard", () => {
  it("renders issue title", () => {
    const issue = {
      id: "123",
      title: "Test Issue",
      priority: "high",
    };

    render(<IssueCard issue={issue} />);

    expect(screen.getByText("Test Issue")).toBeInTheDocument();
  });
});
```

**Run Tests:**
```bash
pnpm --filter web test
```

**Backend Testing:**

```python
# /apps/api/plane/tests/unit/test_issue.py
from django.test import TestCase
from plane.db.models import Issue, Project

class IssueTestCase(TestCase):
    def setUp(self):
        self.project = Project.objects.create(name="Test Project")

    def test_create_issue(self):
        issue = Issue.objects.create(
            name="Test Issue",
            project=self.project
        )
        self.assertEqual(issue.name, "Test Issue")
        self.assertEqual(issue.project, self.project)
```

**Run Tests:**
```bash
cd apps/api
python manage.py test
```

---

## Code Organization

### Where should I put new code?

**Decision Tree:**

```
Is it used by multiple apps?
├─ YES → /packages/<package-name>/
│   ├─ Types? → /packages/types/
│   ├─ UI components? → /packages/ui/
│   ├─ Utilities? → /packages/utils/
│   └─ Hooks? → /packages/hooks/
│
└─ NO → Which app?
    ├─ Main UI → /apps/web/core/
    ├─ Admin → /apps/admin/core/
    ├─ Public → /apps/space/core/
    └─ API → /apps/api/plane/
```

**Examples:**

```typescript
// ✅ Shared type (multiple apps use)
// /packages/types/src/issue.d.ts
export type TIssue = { ... };

// ✅ App-specific component (only web uses)
// /apps/web/core/components/issues/issue-card.tsx
export const IssueCard = () => { ... };

// ✅ Shared UI component (web + admin use)
// /packages/ui/src/components/button.tsx
export const Button = () => { ... };
```

---

### How do imports work?

**Import Patterns:**

```typescript
// 1. Shared packages (workspace:)
import type { TIssue } from "@plane/types";
import { Button } from "@plane/ui";
import { useDebounce } from "@plane/hooks";

// 2. App-internal (~/or absolute)
import { IssueCard } from "~/components/issues/issue-card";
import { useIssueStore } from "~/stores";

// 3. Relative (same directory)
import { IssueActions } from "./issue-actions";
import type { Props } from "./types";

// 4. External packages
import { useState } from "react";
import axios from "axios";
```

**Path Aliases:**

```json
// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "~/*": ["./app/*"],                    // App root
      "@plane/*": ["../../packages/*/src"]    // Shared packages
    }
  }
}
```

---

### Shared vs App-Specific Code?

**Create Shared Package When:**
- ✅ Used by 2+ apps
- ✅ Generic/reusable logic
- ✅ Stable API (won't change often)

**Keep in App When:**
- ✅ Used by only one app
- ✅ Specific to app's domain
- ✅ Still experimental/changing

**Example:**

```typescript
// ❌ Don't create shared package for this
// Only used in web app, very specific
/packages/web-specific-feature/

// ✅ Create shared package for this
// Used in web + admin, generic
/packages/hooks/src/use-debounce.ts
```

---

## Data Flow

### Data Flow Frontend to Backend?

**Complete Flow:**

```
1. User clicks button
   ↓
2. Event handler calls hook
   ↓
3. Hook calls MobX store action
   ↓
4. Store makes optimistic update (UI updates NOW)
   ↓
5. Store calls service layer
   ↓
6. Service sends HTTP request (axios)
   ↓
7. Request hits Django URL router
   ↓
8. Router calls ViewSet method
   ↓
9. ViewSet checks permissions
   ↓
10. ViewSet validates with serializer
   ↓
11. ViewSet saves to database (Django ORM)
   ↓
12. PostgreSQL commits transaction
   ↓
13. ViewSet serializes response
   ↓
14. HTTP response sent back
   ↓
15. Service receives response
   ↓
16. Store updates with real data
   ↓
17. MobX triggers React re-render
   ↓
18. UI reflects final state
```

**See:** [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md) for detailed explanation

---

### SWR vs MobX vs React State?

**Decision Matrix:**

| Data Type | Tool | Example |
|-----------|------|---------|
| Server data (remote) | SWR | Issues from API |
| Client state (shared) | MobX | Filters, selected items |
| UI state (local) | React useState | Modal open/closed |

**Examples:**

```typescript
// Server data → SWR
const { data: issues } = useSWR('/api/issues/', fetcher);

// Client state (shared across components) → MobX
const issueStore = useIssueStore();
const selectedIssues = issueStore.selectedIssues;

// UI state (component-only) → React state
const [isModalOpen, setIsModalOpen] = useState(false);
```

**Rule of Thumb:**
- Data from API? → **SWR**
- Shared across components? → **MobX**
- Component-only? → **React state**

---

### How do real-time updates work?

**WebSocket Flow:**

```
User A updates issue
       ↓
Django saves to database
       ↓
Django publishes to Redis Pub/Sub
       ↓
WebSocket server receives message
       ↓
Server broadcasts to all connected clients
       ↓
User B's browser receives WebSocket message
       ↓
Frontend updates MobX store
       ↓
MobX triggers React re-render
       ↓
User B sees update (no refresh needed!)
```

**Code Example:**

```typescript
// Frontend: Subscribe to updates
const socket = new WebSocket(`ws://localhost:8000/ws/issues/${issueId}/`);

socket.onmessage = (event) => {
  const { type, data } = JSON.parse(event.data);

  if (type === 'issue.updated') {
    // Update MobX store
    issueStore.updateIssue(data.id, data);
    // UI updates automatically via MobX!
  }
};
```

---

## Performance

### Optimize Slow Components?

**Debugging Steps:**

1. **Use React DevTools Profiler**
   ```
   Open DevTools → Profiler tab → Record → Interact → Stop
   Look for components taking > 10ms
   ```

2. **Common Causes & Fixes:**

   **Cause: Expensive calculation in render**
   ```typescript
   // ❌ BAD: Recalculates on every render
   const sortedIssues = issues.sort(...);

   // ✅ GOOD: Only recalculates when issues change
   const sortedIssues = useMemo(() => issues.sort(...), [issues]);
   ```

   **Cause: Creating new function on every render**
   ```typescript
   // ❌ BAD: New function every render
   <Button onClick={() => handleClick(id)} />

   // ✅ GOOD: Stable function reference
   const onClick = useCallback(() => handleClick(id), [id]);
   <Button onClick={onClick} />
   ```

   **Cause: Unnecessary re-renders**
   ```typescript
   // ❌ BAD: Re-renders when parent updates
   const ChildComponent = ({ name }) => <div>{name}</div>;

   // ✅ GOOD: Only re-renders when name changes
   const ChildComponent = memo(({ name }) => <div>{name}</div>);
   ```

---

### Prevent Unnecessary Re-renders?

**Strategies:**

1. **Use memo for expensive components:**
   ```typescript
   const IssueCard = memo(({ issue }: { issue: TIssue }) => {
     return <div>{issue.title}</div>;
   });
   ```

2. **Split components to isolate updates:**
   ```typescript
   // ❌ BAD: Entire list re-renders when count changes
   const IssueList = () => {
     const [count, setCount] = useState(0);
     return (
       <div>
         <h1>Issues ({count})</h1>
         {issues.map(issue => <IssueCard issue={issue} />)}
       </div>
     );
   };

   // ✅ GOOD: Only header re-renders
   const IssueList = () => {
     return (
       <div>
         <IssueListHeader />
         <IssueCards />
       </div>
     );
   };
   ```

3. **Use MobX observer granularly:**
   ```typescript
   // ❌ BAD: Entire component observes entire store
   const App = observer(() => {
     const store = useRootStore();
     return <div>...</div>;
   });

   // ✅ GOOD: Only components that use specific data observe
   const IssueCount = observer(() => {
     const issueStore = useIssueStore();
     return <div>{issueStore.count}</div>;
   });
   ```

---

### N+1 Queries?

**Problem:** Multiple database queries when one would suffice

**Example:**
```python
# ❌ BAD: N+1 queries (1 + N)
issues = Issue.objects.all()  # 1 query
for issue in issues:
    print(issue.state.name)   # N queries (1 per issue!)
    print(issue.project.name) # N more queries!
```

**Solution: Use select_related:**
```python
# ✅ GOOD: 1 query with JOINs
issues = Issue.objects.select_related('state', 'project')  # 1 query with JOIN
for issue in issues:
    print(issue.state.name)   # No query!
    print(issue.project.name) # No query!
```

**For Many-to-Many, use prefetch_related:**
```python
# ❌ BAD: N+1 queries
issues = Issue.objects.all()
for issue in issues:
    for assignee in issue.assignees.all():  # N queries!
        print(assignee.name)

# ✅ GOOD: 2 queries total
issues = Issue.objects.prefetch_related('assignees')  # 2 queries
for issue in issues:
    for assignee in issue.assignees.all():  # No additional queries!
        print(assignee.name)
```

---

## Common Issues

### TypeScript Errors After Updating Type?

**Problem:** Type updated in `@plane/types` but components show errors

**Solution:**

1. **Rebuild types package:**
   ```bash
   pnpm --filter @plane/types build
   ```

2. **Restart TypeScript server in VSCode:**
   ```
   Cmd/Ctrl + Shift + P → "TypeScript: Restart TS Server"
   ```

3. **If still broken, clear cache:**
   ```bash
   pnpm clean
   pnpm install
   pnpm --filter @plane/types build
   ```

---

### CORS Errors?

**Problem:** API requests fail with CORS error

**Check Django CORS settings:**
```python
# /apps/api/plane/settings/common.py
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",  # Web app
    "http://localhost:3001",  # Admin app
]

CORS_ALLOW_CREDENTIALS = True
```

**Check frontend is using correct API URL:**
```bash
# /apps/web/.env
VITE_API_BASE_URL=http://localhost:8000
```

**Common Causes:**
- ❌ Wrong API URL in `.env`
- ❌ CORS origin not in `CORS_ALLOWED_ORIGINS`
- ❌ Missing `withCredentials: true` in axios

---

### Migration Conflicts?

**Problem:** `python manage.py migrate` fails with conflict

**Solution:**

1. **List migrations:**
   ```bash
   python manage.py showmigrations
   ```

2. **Resolve conflict:**
   ```bash
   python manage.py migrate --merge
   ```

3. **If that fails, reset migrations (DANGER: loses data):**
   ```bash
   python manage.py migrate <app> zero
   python manage.py migrate
   ```

**Prevention:**
- Always pull latest before creating migration
- Communicate with team about schema changes

---

### Build Failures CI/CD?

**Common Causes:**

1. **TypeScript errors:**
   ```bash
   pnpm --filter web check:types
   ```

2. **Linting errors:**
   ```bash
   pnpm --filter web check:lint
   ```

3. **Tests failing:**
   ```bash
   pnpm --filter web test
   ```

4. **Dependency issues:**
   ```bash
   pnpm install --frozen-lockfile
   ```

**Fix Locally Before Pushing:**
```bash
pnpm check        # Runs all checks
pnpm fix          # Fixes auto-fixable issues
pnpm test         # Runs tests
```

---

## Next Steps

### Continue Learning

**By Role:**
- **Frontend:** [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md)
- **Backend:** [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)
- **Full-Stack:** [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md)

**By Goal:**
- **Understand architecture:** [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)
- **Start contributing:** [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)
- **Build features:** [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)

---

## Still Have Questions?

- **Join Discord:** [plane.so/discord](https://plane.so/discord)
- **GitHub Discussions:** [github.com/makeplane/plane/discussions](https://github.com/makeplane/plane/discussions)
- **Submit Issue:** [github.com/makeplane/plane/issues](https://github.com/makeplane/plane/issues)

---

**🎉 Hope this helped!**

---

**Last Updated:** November 19, 2025
**Maintainers:** Plane Learning Docs Team
**License:** AGPL-3.0
