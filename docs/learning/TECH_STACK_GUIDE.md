# Tech Stack Guide - Deep Dive into Plane's Technologies

**Master Every Technology!** This guide provides comprehensive explanations of each technology in Plane's stack, with React analogies to help frontend developers understand backend concepts.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 60-75 minutes
**📊 Difficulty Level:** Intermediate to Advanced
**🎯 Target Audience:** Mid-level React developers learning the full stack

---

## Table of Contents

### Frontend Technologies
1. [React 18.3](#react-183)
2. [React Router 7.9](#react-router-79)
3. [TypeScript 5.8](#typescript-58)
4. [MobX 6.12](#mobx-612)
5. [SWR 2.2](#swr-22)
6. [Vite 7.1](#vite-71)
7. [Tailwind CSS](#tailwind-css)

### Backend Technologies
8. [Django 4.2](#django-42)
9. [Django REST Framework 3.15](#django-rest-framework-315)
10. [Python 3.12](#python-312)
11. [Celery 5.4](#celery-54)

### Databases
12. [PostgreSQL 14+](#postgresql-14)
13. [Redis 7+](#redis-7)
14. [MongoDB](#mongodb)

### Build & Development Tools
15. [Turborepo 2.6](#turborepo-26)
16. [pnpm 10.21](#pnpm-1021)
17. [Node.js 22.18+](#nodejs-2218)

### Supporting Technologies
18. [Docker](#docker)
19. [Nginx](#nginx)
20. [WebSockets](#websockets)

---

## React 18.3

### What is React?

**Official Description:** A JavaScript library for building user interfaces

**🧠 Mental Model for Beginners:**
React is like building with LEGO blocks - you create small, reusable pieces (components) and snap them together to build complex UIs.

---

### Why Plane Uses React 18.3

**Key Features Used:**

#### 1. **Concurrent Rendering** ✅ CURRENT
```typescript
// React 18 automatically optimizes rendering
const IssueList = () => {
  const [issues, setIssues] = useState([]);

  // This won't block the UI even with 1000+ issues
  const filteredIssues = issues.filter(issue => issue.priority === 'high');

  return <div>{filteredIssues.map(...)}</div>;
};
```

#### 2. **Automatic Batching**
```typescript
// React 18 batches these into a single re-render
const handleClick = () => {
  setCount(c => c + 1);     // ✓
  setFlag(f => !f);         // ✓ All batched together
  setUser({ name: "John" }); // ✓
  // Only ONE re-render!
};
```

#### 3. **Suspense for Data Fetching**
```typescript
// Plane uses Suspense for code splitting
const IssueDetail = lazy(() => import('./IssueDetail'));

<Suspense fallback={<Loading />}>
  <IssueDetail issueId={id} />
</Suspense>
```

---

### React Patterns in Plane

#### Component Composition ✅ CURRENT
```typescript
// Small, focused components
const IssueCard = ({ issue }: { issue: TIssue }) => (
  <div className="issue-card">
    <IssueTitle title={issue.title} />
    <IssuePriority priority={issue.priority} />
    <IssueAssignees assignees={issue.assignees} />
  </div>
);

// Compose into larger features
const IssueList = ({ issues }: { issues: TIssue[] }) => (
  <div className="issue-list">
    {issues.map(issue => <IssueCard key={issue.id} issue={issue} />)}
  </div>
);
```

**🎯 Remember This:**
> "Composition over inheritance" - Build complex UIs from simple components, don't create complex inheritance hierarchies.

---

### Version Highlights (React 18 vs 17)

| Feature | React 17 | React 18 | Plane Uses? |
|---------|----------|----------|-------------|
| Concurrent Rendering | ❌ | ✅ | ✅ Yes |
| Automatic Batching | Partial | Full | ✅ Yes |
| Suspense (data) | Experimental | Stable | ✅ Yes |
| `startTransition` | ❌ | ✅ | ✅ Yes |
| `useId` hook | ❌ | ✅ | ✅ Yes |

**📍 Docs:** [React 18 Documentation](https://react.dev/)

---

## React Router 7.9

### What is React Router 7?

**Framework Mode:** React Router 7 isn't just a router - it's a full-stack framework like Next.js, built on React Router.

**🌉 Coming from Next.js?**

| Next.js | React Router 7 |
|---------|----------------|
| `/pages/` folder | `/app/routes/` folder |
| `getServerSideProps` | `loader` function |
| API Routes | `action` functions |
| `_app.tsx` | `root.tsx` |
| File-based routing | File-based routing |

---

### Why Plane Uses React Router 7

#### 1. **Type-Safe Loaders** ✅ CURRENT

```typescript
// /apps/web/app/routes/projects.$projectId.tsx
import type { LoaderFunctionArgs } from "react-router";

// Loader runs on SERVER before rendering
export async function loader({ params }: LoaderFunctionArgs) {
  const project = await getProject(params.projectId);
  if (!project) throw new Response("Not Found", { status: 404 });
  return json({ project });
}

// Component gets typed data
export default function ProjectPage() {
  const { project } = useLoaderData<typeof loader>();
  //      ^ TypeScript knows exact shape!

  return <div>{project.name}</div>;
}
```

**💡 Aha Moment:**
> Loaders eliminate the "fetch-in-useEffect" pattern! Data is ready before the component renders.

---

#### 2. **Actions for Mutations** ✅ CURRENT

```typescript
// Handle form submissions on the server
export async function action({ request, params }: ActionFunctionArgs) {
  const formData = await request.formData();
  const title = formData.get("title");

  const issue = await createIssue({
    projectId: params.projectId,
    title: String(title),
  });

  return redirect(`/issues/${issue.id}`);
}

// Use in component
export default function CreateIssue() {
  return (
    <Form method="post">
      <input name="title" />
      <button type="submit">Create</button>
    </Form>
  );
}
```

---

#### 3. **File-Based Routing** ✅ CURRENT

```
app/routes/
├── projects.$projectId.tsx       → /projects/:projectId
├── projects.$projectId.issues.tsx → /projects/:projectId/issues
├── projects.$projectId.settings.tsx → /projects/:projectId/settings
└── issues.$issueId.tsx           → /issues/:issueId
```

**Route Parameters:**
- `$projectId` = Dynamic segment (`:projectId`)
- `.` = Nested route
- `_` prefix = Layout route (no URL segment)

---

#### 4. **Optimistic UI** ✅ CURRENT

```typescript
import { useFetcher } from "react-router";

const IssuePriority = ({ issue }: { issue: TIssue }) => {
  const fetcher = useFetcher();

  // Show optimistic update immediately
  const priority = fetcher.formData?.get("priority") ?? issue.priority;

  return (
    <fetcher.Form method="post" action={`/issues/${issue.id}/priority`}>
      <select name="priority" value={priority} onChange={e => fetcher.submit(e.currentTarget.form)}>
        <option value="high">High</option>
        <option value="medium">Medium</option>
        <option value="low">Low</option>
      </select>
    </fetcher.Form>
  );
};
```

**🎯 Remember This:**
> `useFetcher` for mutations without navigation, `Form` for mutations with navigation.

---

### React Router 7 vs Other Frameworks

| Feature | Next.js 15 | Remix | React Router 7 | Plane Uses? |
|---------|-----------|-------|----------------|-------------|
| File-based routing | ✅ | ✅ | ✅ | ✅ |
| Server components | ✅ | ❌ | ❌ | ❌ |
| Loaders | ❌ | ✅ | ✅ | ✅ |
| Actions | ❌ | ✅ | ✅ | ✅ |
| TypeScript inference | Good | Excellent | Excellent | ✅ |
| Bundle size | Large | Small | Small | ✅ |

**Why React Router 7?**
- More flexible than Next.js
- Better TypeScript support than Next.js
- Simpler mental model (no server components confusion)
- Smaller bundle size

**📍 Docs:** [React Router 7 Documentation](https://reactrouter.com/)

---

## TypeScript 5.8

### What is TypeScript?

**In One Sentence:** JavaScript with types that catch bugs before you run the code.

**🧠 Mental Model:**
TypeScript is like having a smart linter that understands your entire codebase and prevents you from making mistakes.

---

### Why Plane Uses TypeScript 5.8

#### 1. **Strict Type Safety** ✅ CURRENT

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,              // All strict checks
    "noUncheckedIndexedAccess": true,  // Array/object access safety
    "noImplicitAny": true,       // No implicit any
    "strictNullChecks": true     // Catch null/undefined errors
  }
}
```

**Example:**

```typescript
// ✅ Type-safe
const getIssue = (id: string): TIssue | undefined => {
  return issues.find(i => i.id === id);
};

const issue = getIssue("123");
if (issue) {
  console.log(issue.title);  // OK - checked for undefined
}

// ❌ TypeScript error
console.log(issue.title);  // Error: issue might be undefined
```

---

#### 2. **Type Inference** ✅ CURRENT

```typescript
// TypeScript infers types automatically
const issue = {
  id: "123",
  title: "Fix bug",
  priority: "high" as const,  // Literal type, not string
};

// TypeScript knows:
// issue.id is string
// issue.title is string
// issue.priority is "high" (not generic string)

issue.priority = "medium";  // ❌ Error: Type '"medium"' is not assignable to type '"high"'
```

---

#### 3. **Discriminated Unions** ✅ CURRENT

```typescript
// Common pattern in Plane
type TApiResponse<T> =
  | { status: "loading" }
  | { status: "error"; error: string }
  | { status: "success"; data: T };

const handleResponse = (response: TApiResponse<TIssue>) => {
  // TypeScript narrows type based on status
  if (response.status === "loading") {
    return <Loading />;
  }

  if (response.status === "error") {
    return <Error message={response.error} />;
    //                      ^ TypeScript knows error exists here
  }

  return <IssueList issues={[response.data]} />;
  //                         ^ TypeScript knows data exists here
};
```

---

#### 4. **Template Literal Types** (TS 5.8)

```typescript
// Type-safe route generation
type TWorkspaceRoute = `/${string}/projects`;
type TProjectRoute = `/${string}/projects/${string}`;
type TIssueRoute = `/${string}/projects/${string}/issues/${string}`;

const navigateToIssue = (workspace: string, project: string, issue: string) => {
  const route: TIssueRoute = `/${workspace}/projects/${project}/issues/${issue}`;
  navigate(route);
};
```

---

### TypeScript 5.8 Features Used

| Feature | Description | Plane Uses? |
|---------|-------------|-------------|
| `satisfies` operator | Type checking without widening | ✅ Yes |
| Decorator support | Class decorators (MobX) | ✅ Yes |
| `const` type parameters | Preserve literal types | ✅ Yes |
| `using` declaration | Resource management | 🔜 Coming |
| Isolated declarations | Faster builds | ✅ Yes |

**Example: `satisfies`**

```typescript
// Ensure object matches type without widening
const ISSUE_PRIORITIES = {
  urgent: { label: "Urgent", color: "#ef4444" },
  high: { label: "High", color: "#f97316" },
  medium: { label: "Medium", color: "#eab308" },
  low: { label: "Low", color: "#22c55e" },
} satisfies Record<string, { label: string; color: string }>;

// TypeScript knows exact keys
type TPriority = keyof typeof ISSUE_PRIORITIES;  // "urgent" | "high" | "medium" | "low"
```

**📍 Docs:** [TypeScript 5.8 Documentation](https://www.typescriptlang.org/docs/)

---

## MobX 6.12

### What is MobX?

**In One Sentence:** Reactive state management that automatically updates your UI when data changes.

**🧠 Mental Model:**
MobX is like Excel spreadsheets - change a cell, and all formulas recalculate automatically. Change state, and all components re-render automatically.

---

### MobX vs Other State Management

| Library | Learning Curve | Boilerplate | Performance | Plane Uses? |
|---------|----------------|-------------|-------------|-------------|
| MobX | Low | Minimal | Excellent | ✅ Yes |
| Redux | High | Lots | Good | ❌ No |
| Zustand | Low | Minimal | Good | ❌ No |
| Jotai | Medium | Some | Excellent | ❌ No |

**Why MobX?**
- Less boilerplate than Redux
- Automatic reactivity (no manual subscriptions)
- Excellent performance (fine-grained reactivity)
- Great TypeScript support

---

### MobX Core Concepts

#### 1. **Observable State** ✅ CURRENT

```typescript
import { makeObservable, observable } from "mobx";

class IssueStore {
  // Observable state (auto-tracked)
  issues = new Map<string, TIssue>();

  constructor() {
    makeObservable(this, {
      issues: observable,  // Make it observable
    });
  }
}
```

**🌉 React Comparison:**

```typescript
// React useState
const [issues, setIssues] = useState(new Map());

// MobX observable
issues = new Map();  // No setter needed, change directly!
```

---

#### 2. **Actions** ✅ CURRENT

```typescript
import { action } from "mobx";

class IssueStore {
  issues = new Map<string, TIssue>();

  constructor() {
    makeObservable(this, {
      issues: observable,
      addIssue: action,      // Mark as action
      removeIssue: action,
    });
  }

  // Actions modify state
  addIssue = (issue: TIssue) => {
    this.issues.set(issue.id, issue);
  };

  removeIssue = (id: string) => {
    this.issues.delete(id);
  };
}
```

**🎯 Remember This:**
> Actions are the ONLY way to modify observable state (in strict mode).

---

#### 3. **Computed Values** ✅ CURRENT

```typescript
import { computed } from "mobx";

class IssueStore {
  issues = new Map<string, TIssue>();

  constructor() {
    makeObservable(this, {
      issues: observable,
      issueCount: computed,    // Cached, auto-updates
      highPriorityIssues: computed,
    });
  }

  // Computed values (cached)
  get issueCount() {
    return this.issues.size;
  }

  get highPriorityIssues() {
    return Array.from(this.issues.values())
      .filter(issue => issue.priority === "high");
  }
}
```

**🌉 React Comparison:**

```typescript
// React useMemo
const highPriorityIssues = useMemo(
  () => issues.filter(i => i.priority === "high"),
  [issues]  // Manual dependency!
);

// MobX computed
get highPriorityIssues() {
  return issues.filter(i => i.priority === "high");
  // Automatic dependency tracking!
}
```

---

#### 4. **Reactions** ✅ CURRENT

```typescript
import { reaction } from "mobx";

class IssueStore {
  issues = new Map<string, TIssue>();

  constructor() {
    // React to changes
    reaction(
      () => this.issues.size,  // Watch this
      (size) => {               // Run this when it changes
        console.log(`Issue count: ${size}`);
      }
    );
  }
}
```

**🌉 React Comparison:**

```typescript
// React useEffect
useEffect(() => {
  console.log(`Issue count: ${issues.length}`);
}, [issues.length]);  // Manual dependency

// MobX reaction
reaction(
  () => issues.size,  // Auto-tracked
  (size) => console.log(`Issue count: ${size}`)
);
```

---

### Using MobX in React

```typescript
import { observer } from "mobx-react";
import { useIssueStore } from "./stores";

// Observer makes component reactive
const IssueList = observer(() => {
  const store = useIssueStore();

  // Automatically re-renders when store.issues changes
  return (
    <div>
      <h1>Issues ({store.issueCount})</h1>
      {Array.from(store.issues.values()).map(issue => (
        <IssueCard key={issue.id} issue={issue} />
      ))}
    </div>
  );
});
```

**⚠️ Common Pitfall:**

```typescript
// ❌ Doesn't work - not an observer
const IssueCount = () => {
  const store = useIssueStore();
  return <div>{store.issueCount}</div>;  // Won't update!
};

// ✅ Works - wrapped in observer
const IssueCount = observer(() => {
  const store = useIssueStore();
  return <div>{store.issueCount}</div>;  // Updates automatically!
});
```

**📍 Docs:** [MobX Documentation](https://mobx.js.org/)

---

## SWR 2.2

### What is SWR?

**SWR = Stale-While-Revalidate**

**In One Sentence:** A data fetching library that returns cached data (stale) while fetching fresh data in the background (revalidate).

**🧠 Mental Model:**
SWR is like your phone's email app - it shows you cached emails instantly, then checks for new ones in the background.

---

### Why Plane Uses SWR

#### 1. **Automatic Caching** ✅ CURRENT

```typescript
import useSWR from "swr";

const IssueList = () => {
  // First call: fetches from API
  const { data } = useSWR("/api/issues/", fetcher);

  // Second call (elsewhere): returns from cache!
  const { data: sameData } = useSWR("/api/issues/", fetcher);

  return <div>{data?.map(...)}</div>;
};
```

---

#### 2. **Automatic Revalidation** ✅ CURRENT

```typescript
const { data } = useSWR("/api/issues/", fetcher, {
  revalidateOnFocus: true,      // Refetch when window focuses
  revalidateOnReconnect: true,  // Refetch when internet reconnects
  refreshInterval: 30000,       // Poll every 30 seconds
});
```

**💡 Aha Moment:**
> SWR keeps your UI in sync with the server automatically - no manual refetching needed!

---

#### 3. **Optimistic Updates** ✅ CURRENT

```typescript
import { useSWRConfig } from "swr";

const updateIssue = async (id: string, data: Partial<TIssue>) => {
  const { mutate } = useSWRConfig();

  // Optimistic update (instant feedback)
  mutate(
    "/api/issues/",
    (current: TIssue[]) =>
      current.map(i => i.id === id ? { ...i, ...data } : i),
    false  // Don't revalidate yet
  );

  try {
    // Send to server
    await IssueService.update(id, data);

    // Revalidate to ensure accuracy
    mutate("/api/issues/");
  } catch (error) {
    // Rollback on error
    mutate("/api/issues/");
  }
};
```

---

#### 4. **Pagination Support** ✅ CURRENT

```typescript
import useSWRInfinite from "swr/infinite";

const IssueList = () => {
  const { data, size, setSize } = useSWRInfinite(
    (index) => `/api/issues/?page=${index + 1}`,
    fetcher
  );

  const issues = data ? data.flat() : [];
  const loadMore = () => setSize(size + 1);

  return (
    <div>
      {issues.map(issue => <IssueCard key={issue.id} issue={issue} />)}
      <button onClick={loadMore}>Load More</button>
    </div>
  );
};
```

---

### SWR Best Practices in Plane

#### ✅ CURRENT: Separate keys for different queries

```typescript
// Good - separate keys
useSWR(`/api/projects/${projectId}/issues/`, fetcher);
useSWR(`/api/projects/${projectId}/issues/?priority=high`, fetcher);

// Bad - same key for different queries
useSWR('/api/issues/', () => fetcher({ projectId }));
```

#### ✅ CURRENT: Use SWR for server state, MobX for client state

```typescript
// Server state (from API) - Use SWR
const { data: issues } = useSWR('/api/issues/', fetcher);

// Client state (UI only) - Use MobX or useState
const [selectedIssue, setSelectedIssue] = useState<string | null>(null);
```

**📍 Docs:** [SWR Documentation](https://swr.vercel.app/)

---

## Vite 7.1

### What is Vite?

**In One Sentence:** A build tool that's 10-100x faster than Webpack because it uses ESM and esbuild.

**🧠 Mental Model:**
- **Webpack**: Bundles everything upfront (slow cold start)
- **Vite**: Serves files on-demand (instant cold start)

---

### Why Plane Uses Vite

#### 1. **Instant Server Start** ✅ CURRENT

```bash
# Webpack (old)
npm run dev  # Wait 30-60 seconds...

# Vite (new)
pnpm dev     # Starts in <1 second!
```

---

#### 2. **Hot Module Replacement (HMR)** ✅ CURRENT

```typescript
// Edit this component
export const IssueCard = ({ issue }: { issue: TIssue }) => {
  return <div>{issue.title}</div>;  // Change this
};

// Vite updates INSTANTLY (< 100ms)
// State preserved, no full reload!
```

---

#### 3. **Optimized Build** ✅ CURRENT

```bash
# Build for production
pnpm build

# Vite automatically:
# - Code splits by route
# - Tree-shakes unused code
# - Minifies with esbuild
# - Generates source maps
```

---

### Vite Configuration in Plane

```typescript
// /apps/web/react-router.config.ts
import { defineConfig } from "@react-router/dev/config";

export default defineConfig({
  // Server options
  server: {
    port: 3000,
    hmr: true,
  },

  // Build options
  build: {
    target: "esnext",
    sourcemap: true,
  },

  // Optimizations
  optimizeDeps: {
    include: ["mobx", "mobx-react", "react", "react-dom"],
  },
});
```

**📍 Docs:** [Vite Documentation](https://vitejs.dev/)

---

## Django 4.2

### What is Django?

**In One Sentence:** A Python web framework that includes everything you need to build a backend (ORM, admin, auth, etc.).

**🧠 Mental Model for React Devs:**
- **React** = UI library (bring your own router, state, etc.)
- **Django** = Full framework (batteries included)

---

### Why Plane Uses Django

#### 1. **Built-in ORM** ✅ CURRENT

```python
# No SQL required!
issues = Issue.objects.filter(
    project_id=project_id,
    priority="high"
).select_related('state', 'project')

# Django generates SQL:
# SELECT * FROM plane_issue
# JOIN plane_state ON plane_issue.state_id = plane_state.id
# JOIN plane_project ON plane_issue.project_id = plane_project.id
# WHERE project_id = %s AND priority = %s
```

**🌉 React Comparison:**

```typescript
// React (manual filtering)
const highPriorityIssues = issues.filter(i =>
  i.project_id === projectId && i.priority === "high"
);

// Django ORM (database filtering)
issues = Issue.objects.filter(project_id=project_id, priority="high")
```

---

#### 2. **Built-in Admin** ✅ CURRENT

```python
# /apps/api/plane/db/admin.py
from django.contrib import admin
from plane.db.models import Issue

@admin.register(Issue)
class IssueAdmin(admin.ModelAdmin):
    list_display = ['name', 'project', 'state', 'priority']
    list_filter = ['priority', 'state']
    search_fields = ['name', 'description']

# Automatically generates admin UI at /admin/
```

**💡 Aha Moment:**
> Django admin gives you a free CRUD interface for all models - no custom UI needed for internal tools!

---

#### 3. **Built-in Authentication** ✅ CURRENT

```python
# Authentication built-in
from django.contrib.auth import authenticate, login

def login_view(request):
    user = authenticate(username=username, password=password)
    if user:
        login(request, user)  # Creates session
        return Response({"success": True})
    return Response({"error": "Invalid credentials"}, status=401)
```

---

#### 4. **Migration System** ✅ CURRENT

```python
# Change model
class Issue(models.Model):
    priority = models.CharField(max_length=30)
    # Add new field
    estimate_point = models.IntegerField(null=True, blank=True)

# Generate migration
$ python manage.py makemigrations

# Apply migration
$ python manage.py migrate
# Django automatically updates database schema!
```

**🌉 React Comparison:**

```typescript
// React: Change type manually everywhere
type TIssue = {
  // ...
  estimate_point?: number;  // Add manually
};

// Update 50+ files using TIssue...

// Django: Change model once, run migration
class Issue(models.Model):
    estimate_point = models.IntegerField(null=True)
# Django updates database + generates Python code
```

---

### Django Architecture (MTV)

**MTV = Model-Template-View** (not MVC!)

```python
# Model (Database)
class Issue(models.Model):
    name = models.CharField(max_length=255)

# View (Business Logic)
def get_issue(request, issue_id):
    issue = Issue.objects.get(id=issue_id)
    return Response(IssueSerializer(issue).data)

# Template (Presentation)
# Plane uses React instead of Django templates!
```

**🌉 React Comparison:**

| Django | React | Purpose |
|--------|-------|---------|
| Model | TypeScript type | Data structure |
| View | API call | Business logic |
| Template | Component | Presentation |

**📍 Docs:** [Django Documentation](https://docs.djangoproject.com/)

---

## PostgreSQL 14+

### What is PostgreSQL?

**In One Sentence:** A powerful, open-source relational database that stores your data in tables with relationships.

**🧠 Mental Model:**
PostgreSQL is like Excel on steroids:
- **Tables** = Spreadsheets
- **Columns** = Column headers
- **Rows** = Data rows
- **Relationships** = VLOOKUP between sheets

---

### Why Plane Uses PostgreSQL

#### 1. **ACID Compliance** ✅

**ACID = Atomicity, Consistency, Isolation, Durability**

```sql
-- All or nothing
BEGIN;
  INSERT INTO plane_issue (name, project_id) VALUES ('Bug fix', 123);
  UPDATE plane_project SET issue_count = issue_count + 1 WHERE id = 123;
COMMIT;  -- Both succeed or both fail
```

---

#### 2. **Rich Data Types** ✅

```python
# PostgreSQL supports JSON, arrays, UUIDs, etc.
class Issue(models.Model):
    id = models.UUIDField(primary_key=True)  # UUID type
    description = models.JSONField()          # JSON type
    labels = models.ArrayField(              # Array type
        models.CharField(max_length=50)
    )
```

---

#### 3. **Advanced Indexing** ✅

```python
class Issue(models.Model):
    name = models.CharField(max_length=255, db_index=True)  # Index

    class Meta:
        indexes = [
            # Composite index for faster queries
            models.Index(fields=['project', 'state']),
            # GIN index for JSON fields
            models.Index(fields=['description'], name='issue_desc_gin'),
        ]
```

---

### PostgreSQL in Plane

**Connection:**

```python
# /apps/api/plane/settings/common.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('POSTGRES_DB'),
        'USER': os.environ.get('POSTGRES_USER'),
        'PASSWORD': os.environ.get('POSTGRES_PASSWORD'),
        'HOST': os.environ.get('POSTGRES_HOST'),
        'PORT': os.environ.get('POSTGRES_PORT', 5432),
    }
}
```

**Key Tables:**

- `plane_user` - Users
- `plane_workspace` - Workspaces
- `plane_project` - Projects
- `plane_issue` - Issues (largest table)
- `plane_state` - Issue states
- `plane_issue_assignee` - Many-to-many relationship

**📍 Docs:** [PostgreSQL Documentation](https://www.postgresql.org/docs/)

---

## Redis 7+

### What is Redis?

**In One Sentence:** An in-memory key-value store used for caching and sessions (think: super-fast dictionary).

**🧠 Mental Model:**
Redis is like JavaScript's `Map` object, but shared across all servers and persisted to disk.

---

### Why Plane Uses Redis

#### 1. **Session Storage** ✅ CURRENT

```python
# Store session in Redis (fast!)
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "default"  # Redis

# When user logs in:
login(request, user)
# Session stored in Redis:
# Key: session:abc123
# Value: {user_id: 789, ...}
```

---

#### 2. **Caching** ✅ CURRENT

```python
from django.core.cache import cache

# Cache expensive query
issues = cache.get(f"project:{project_id}:issues")
if not issues:
    issues = Issue.objects.filter(project_id=project_id)
    cache.set(f"project:{project_id}:issues", issues, timeout=300)  # 5 min
```

---

#### 3. **Celery Broker** ✅ CURRENT

```python
# Celery uses Redis as message queue
CELERY_BROKER_URL = os.environ.get('REDIS_URL')
CELERY_RESULT_BACKEND = os.environ.get('REDIS_URL')

# When you call:
send_email.delay(user_id)
# Task stored in Redis queue, worker picks it up
```

---

### Redis Data Structures

```python
from django.core.cache import cache

# String
cache.set("user:123:name", "John Doe")

# List
cache.lpush("recent_issues", issue_id)  # Add to list

# Set
cache.sadd("project:123:members", user_id)  # Add to set

# Hash
cache.hset("user:123", "email", "john@example.com")

# Expiration
cache.expire("temp_data", 60)  # Expire in 60 seconds
```

**📍 Docs:** [Redis Documentation](https://redis.io/docs/)

---

## Celery 5.4

### What is Celery?

**In One Sentence:** A distributed task queue for running background jobs asynchronously.

**🧠 Mental Model:**
Celery is like hiring workers to do tasks in the background while your main app stays responsive.

---

### Why Plane Uses Celery

#### 1. **Async Email Sending** ✅ CURRENT

```python
# /apps/api/plane/bgtasks/notification.py
from celery import shared_task

@shared_task
def send_email_notification(user_id: str, message: str):
    user = User.objects.get(id=user_id)
    send_email(to=user.email, subject="Notification", body=message)

# In view:
def create_issue(request):
    issue = Issue.objects.create(...)
    # Don't block response waiting for email
    send_email_notification.delay(user_id, "New issue created!")
    return Response(IssueSerializer(issue).data)
```

**🌉 React Comparison:**

```typescript
// React (blocks render)
const handleClick = async () => {
  await slowOperation();  // UI frozen!
  setDone(true);
};

// Celery (doesn't block)
def create_issue(request):
    issue = Issue.objects.create(...)
    slow_operation.delay()  # Background task
    return Response(...)    # Immediate response!
```

---

#### 2. **Scheduled Tasks** ✅ CURRENT

```python
# /apps/api/plane/settings/common.py
from celery.schedules import crontab

CELERY_BEAT_SCHEDULE = {
    'daily-digest': {
        'task': 'plane.bgtasks.send_daily_digest',
        'schedule': crontab(hour=9, minute=0),  # 9 AM daily
    },
    'cleanup-old-data': {
        'task': 'plane.bgtasks.cleanup_old_data',
        'schedule': crontab(hour=2, minute=0),  # 2 AM daily
    },
}
```

---

#### 3. **Export Generation** ✅ CURRENT

```python
@shared_task
def export_issues_csv(project_id: str, user_id: str):
    # Long-running task (2-5 minutes)
    issues = Issue.objects.filter(project_id=project_id)
    csv_file = generate_csv(issues)
    upload_to_s3(csv_file)
    notify_user(user_id, csv_file.url)

# In view:
def export_issues(request, project_id):
    # Start background task
    task = export_issues_csv.delay(project_id, request.user.id)
    return Response({"task_id": task.id})  # Immediate response
```

---

### Celery Architecture

```
┌─────────────┐
│  Django API │
└──────┬──────┘
       │ task.delay()
       ▼
┌─────────────┐
│    Redis    │ ← Message queue
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Celery    │
│   Worker    │ ← Processes tasks
└─────────────┘
```

**📍 Docs:** [Celery Documentation](https://docs.celeryq.dev/)

---

## Turborepo 2.6

### What is Turborepo?

**In One Sentence:** A build orchestrator for monorepos that caches builds and runs tasks in parallel.

**🧠 Mental Model:**
Turborepo is like a smart build system that remembers what it built before and only rebuilds what changed.

---

### Why Plane Uses Turborepo

#### 1. **Incremental Builds** ✅ CURRENT

```bash
# First build (everything)
turbo build
# Builds: types, ui, hooks, services, web, admin, space (7 packages)
# Time: 2-3 minutes

# Change one file in web app
turbo build
# Only builds: web (cached: types, ui, hooks, services)
# Time: 10 seconds!
```

---

#### 2. **Parallel Execution** ✅ CURRENT

```bash
# Without Turborepo (sequential)
pnpm build --filter types  # 10s
pnpm build --filter ui     # 20s
pnpm build --filter web    # 30s
# Total: 60s

# With Turborepo (parallel)
turbo build
# All run in parallel!
# Total: 30s (limited by slowest)
```

---

#### 3. **Smart Caching** ✅ CURRENT

```json
// turbo.json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", "build/**"]
    }
  }
}
```

**How it works:**
1. Hash inputs (source files, dependencies)
2. Check cache for this hash
3. If found, restore from cache (instant!)
4. If not found, run build and cache result

**📍 Docs:** [Turborepo Documentation](https://turbo.build/repo/docs)

---

## pnpm 10.21

### What is pnpm?

**In One Sentence:** A fast, disk-efficient package manager that shares dependencies across projects.

**🧠 Mental Model:**

```
npm/yarn:
  project1/node_modules/react/  (50 MB)
  project2/node_modules/react/  (50 MB)
  project3/node_modules/react/  (50 MB)
  Total: 150 MB

pnpm:
  ~/.pnpm-store/react/           (50 MB)
  project1/node_modules/react/  → symlink
  project2/node_modules/react/  → symlink
  project3/node_modules/react/  → symlink
  Total: 50 MB
```

---

### Why Plane Uses pnpm

#### 1. **Workspace Support** ✅ CURRENT

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

```bash
# Install for specific package
pnpm add react --filter web

# Install for all packages
pnpm install

# Run script in specific package
pnpm --filter web dev
```

---

#### 2. **Disk Space Savings** ✅

```bash
# npm (traditional)
$ du -sh node_modules/
1.2 GB

# pnpm (with sharing)
$ du -sh node_modules/
300 MB (symlinks to shared store)
```

---

#### 3. **Strict Dependencies** ✅

```typescript
// ❌ npm allows this (phantom dependency)
import _ from "lodash";  // Not in package.json, but works!

// ✅ pnpm prevents this
import _ from "lodash";  // Error: lodash not in package.json
```

**📍 Docs:** [pnpm Documentation](https://pnpm.io/)

---

## Quick Reference

### Frontend Stack Summary

| Technology | Version | Purpose | Alternatives |
|------------|---------|---------|--------------|
| React | 18.3 | UI library | Vue, Svelte |
| React Router | 7.9 | Routing + SSR | Next.js, Remix |
| TypeScript | 5.8 | Type safety | JavaScript, Flow |
| MobX | 6.12 | State management | Redux, Zustand |
| SWR | 2.2 | Data fetching | React Query, RTK Query |
| Vite | 7.1 | Build tool | Webpack, esbuild |
| Tailwind | 3.4 | CSS framework | CSS Modules, Styled Components |

---

### Backend Stack Summary

| Technology | Version | Purpose | Alternatives |
|------------|---------|---------|--------------|
| Django | 4.2 | Web framework | FastAPI, Express |
| DRF | 3.15 | REST API | GraphQL, tRPC |
| Python | 3.12 | Language | Node.js, Go |
| Celery | 5.4 | Task queue | Bull, BullMQ |
| PostgreSQL | 14+ | Database | MySQL, MongoDB |
| Redis | 7+ | Cache/Sessions | Memcached, Valkey |

---

### Development Stack Summary

| Technology | Version | Purpose | Alternatives |
|------------|---------|---------|--------------|
| Turborepo | 2.6 | Monorepo builds | Nx, Lerna |
| pnpm | 10.21 | Package manager | npm, yarn |
| Node.js | 22.18+ | Runtime | Bun, Deno |
| Docker | Latest | Containerization | Podman, containerd |

---

## Next Steps

### Deep Dive by Role

**Frontend Focus:**
1. [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md) - React patterns
2. [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md) - Code standards

**Backend Focus:**
1. [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md) - Django deep dive
2. [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md) - Database design

**Full-Stack:**
1. [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md) - How data flows
2. [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md) - Frontend ↔ Backend

---

**🎉 You now understand every technology in Plane's stack!**

---

**Last Updated:** November 19, 2025
**Maintainers:** Plane Learning Docs Team
**License:** AGPL-3.0
