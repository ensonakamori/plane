# Frontend Architecture - React Best Practices & Patterns

**Master React in Plane!** This guide covers React best practices, architectural patterns, and modern development techniques used in the Plane codebase.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 60-75 minutes
**📊 Difficulty Level:** Intermediate to Advanced
**🎯 Target Audience:** React developers wanting to master production-grade patterns

---

## Table of Contents

1. [React Router 7 Framework](#react-router-7-framework)
2. [Component Architecture](#component-architecture)
3. [State Management with MobX](#state-management-with-mobx)
4. [Data Fetching Strategies](#data-fetching-strategies)
5. [TypeScript Best Practices](#typescript-best-practices)
6. [Performance Optimization](#performance-optimization)
7. [Code Organization](#code-organization)
8. [Testing Strategies](#testing-strategies)
9. [Common Patterns](#common-patterns)
10. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
11. [Next Steps](#next-steps)

---

## React Router 7 Framework

### File-Based Routing ✅ CURRENT

**Location:** `/apps/web/app/routes/`

```
routes/
├── (all)/                          # Public routes group
│   ├── layout.tsx                 # Shared layout for public routes
│   └── invitations.$invitation.tsx  # /invitations/:invitation
│
├── (home)/                        # Home section
│   ├── layout.tsx                # Home layout
│   ├── route.tsx                 # /home
│   └── _index.tsx                # /home (index route)
│
└── projects.$projectId/          # Project routes
    ├── layout.tsx                # Project layout (sidebar, header)
    ├── route.tsx                 # /projects/:projectId
    ├── issues.tsx                # /projects/:projectId/issues
    ├── issues.$issueId.tsx       # /projects/:projectId/issues/:issueId
    └── settings.tsx              # /projects/:projectId/settings
```

**File Naming Conventions:**

- `$param` = Dynamic segment (e.g., `$projectId` → `:projectId`)
- `.` = Nested route
- `_` prefix = Layout route (no URL segment)
- `_index.tsx` = Index route

---

### Loaders - Data Before Render ✅ CURRENT

**Pattern:** Fetch data on the server before component renders

```typescript
// /apps/web/app/routes/projects.$projectId.issues.tsx
import type { LoaderFunctionArgs } from "react-router";
import { json } from "react-router";
import { IssueService } from "~/services/issue.service";

// Loader runs on SERVER (or client during navigation)
export async function loader({ params, request }: LoaderFunctionArgs) {
  const { projectId } = params;

  // Fetch data
  const issues = await IssueService.getIssues(projectId);
  const states = await StateService.getStates(projectId);

  // Return as JSON
  return json({
    issues,
    states,
    meta: {
      total: issues.length,
      fetched_at: new Date().toISOString(),
    },
  });
}

// Component gets typed data
export default function IssuesPage() {
  const { issues, states, meta } = useLoaderData<typeof loader>();
  //      ^ TypeScript knows exact shape!

  return (
    <div>
      <h1>Issues ({meta.total})</h1>
      <IssueList issues={issues} states={states} />
    </div>
  );
}
```

**🎯 Benefits:**

1. **No Loading States** - Data ready when component mounts
2. **Type Safety** - `useLoaderData<typeof loader>` is fully typed
3. **Server Rendering** - Can render on server with data
4. **Better UX** - No flash of loading spinner

---

### Actions - Handle Mutations ✅ CURRENT

**Pattern:** Handle form submissions and mutations on the server

```typescript
// /apps/web/app/routes/projects.$projectId.issues.tsx
import type { ActionFunctionArgs } from "react-router";
import { redirect } from "react-router";

export async function action({ request, params }: ActionFunctionArgs) {
  const { projectId } = params;
  const formData = await request.formData();

  const intent = formData.get("intent");

  switch (intent) {
    case "create": {
      const title = String(formData.get("title"));
      const description = String(formData.get("description"));

      const issue = await IssueService.create(projectId, {
        title,
        description,
      });

      // Redirect to new issue
      return redirect(`/projects/${projectId}/issues/${issue.id}`);
    }

    case "delete": {
      const issueId = String(formData.get("issueId"));
      await IssueService.delete(issueId);

      // Stay on same page
      return json({ success: true });
    }

    default:
      return json({ error: "Invalid intent" }, { status: 400 });
  }
}

// Component uses Form
export default function IssuesPage() {
  return (
    <Form method="post">
      <input type="hidden" name="intent" value="create" />
      <input name="title" placeholder="Issue title" required />
      <textarea name="description" placeholder="Description" />
      <button type="submit">Create Issue</button>
    </Form>
  );
}
```

**🎯 Benefits:**

1. **Progressive Enhancement** - Works without JavaScript
2. **Type Safety** - Form data validated on server
3. **Automatic Revalidation** - Loaders rerun after actions
4. **Optimistic UI** - Use `useFetcher` for instant feedback

---

### Optimistic UI with Fetchers ✅ CURRENT

**Pattern:** Update UI immediately, sync with server in background

```typescript
import { useFetcher } from "react-router";

const IssuePrioritySelect = ({ issue }: { issue: TIssue }) => {
  const fetcher = useFetcher();

  // Get optimistic value (what user clicked) or current value
  const priority =
    (fetcher.formData?.get("priority") as TPriority) ?? issue.priority;

  const isUpdating = fetcher.state !== "idle";

  return (
    <fetcher.Form
      method="post"
      action={`/api/issues/${issue.id}/priority`}
    >
      <select
        name="priority"
        value={priority}
        onChange={(e) => fetcher.submit(e.currentTarget.form)}
        disabled={isUpdating}
      >
        <option value="urgent">🔴 Urgent</option>
        <option value="high">🟠 High</option>
        <option value="medium">🟡 Medium</option>
        <option value="low">🟢 Low</option>
        <option value="none">⚪ None</option>
      </select>
    </fetcher.Form>
  );
};
```

**Timeline:**
- **0ms** - User selects new priority
- **0ms** - UI updates immediately (optimistic)
- **0-200ms** - Request sent to server
- **200ms** - Server confirms, UI stays the same (already updated!)

**💡 Aha Moment:**
> Fetchers give you optimistic UI for free! The UI updates instantly, even though the server hasn't responded yet.

---

## Component Architecture

### Component Hierarchy ✅ CURRENT

```
Page (Route Component)
  ├── Layout
  │   ├── Header
  │   ├── Sidebar
  │   └── Content
  │       └── Feature Component
  │           ├── List Component
  │           │   └── Item Component
  │           │       ├── Actions
  │           │       └── Details
  │           └── Filters
  └── Modals
```

**Example:**

```typescript
// Page Level
const IssuesPage = () => {
  return (
    <ProjectLayout>
      <IssuesFeature projectId={projectId} />
    </ProjectLayout>
  );
};

// Feature Level
const IssuesFeature = ({ projectId }: { projectId: string }) => {
  const { issues } = useIssues(projectId);

  return (
    <div>
      <IssueFilters />
      <IssueList issues={issues} />
      <CreateIssueModal />
    </div>
  );
};

// List Level
const IssueList = ({ issues }: { issues: TIssue[] }) => {
  return (
    <div>
      {issues.map((issue) => (
        <IssueCard key={issue.id} issue={issue} />
      ))}
    </div>
  );
};

// Item Level
const IssueCard = ({ issue }: { issue: TIssue }) => {
  return (
    <div className="issue-card">
      <IssueTitle title={issue.title} />
      <IssuePriority priority={issue.priority} />
      <IssueAssignees assignees={issue.assignees} />
      <IssueActions issue={issue} />
    </div>
  );
};
```

**🎯 Remember This:**
> Break components down until each has a single responsibility. Small components are easier to test, reuse, and understand.

---

### Component Patterns

#### 1. Compound Components ✅ CURRENT

**Pattern:** Related components that work together

```typescript
// /packages/ui/src/components/dropdown/dropdown.tsx
const Dropdown = ({ children }: { children: React.ReactNode }) => {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <DropdownContext.Provider value={{ isOpen, setIsOpen }}>
      <div className="dropdown">{children}</div>
    </DropdownContext.Provider>
  );
};

const DropdownTrigger = ({ children }: { children: React.ReactNode }) => {
  const { setIsOpen } = useDropdownContext();
  return <button onClick={() => setIsOpen((v) => !v)}>{children}</button>;
};

const DropdownContent = ({ children }: { children: React.ReactNode }) => {
  const { isOpen } = useDropdownContext();
  if (!isOpen) return null;
  return <div className="dropdown-content">{children}</div>;
};

// Export as compound component
Dropdown.Trigger = DropdownTrigger;
Dropdown.Content = DropdownContent;

export { Dropdown };

// Usage
<Dropdown>
  <Dropdown.Trigger>
    <Button>Open Menu</Button>
  </Dropdown.Trigger>
  <Dropdown.Content>
    <MenuItem>Option 1</MenuItem>
    <MenuItem>Option 2</MenuItem>
  </Dropdown.Content>
</Dropdown>
```

---

#### 2. Render Props ✅ CURRENT

**Pattern:** Share logic via render function

```typescript
const DataLoader = <T,>({
  url,
  children,
}: {
  url: string;
  children: (data: T | null, loading: boolean, error: Error | null) => React.ReactNode;
}) => {
  const { data, error, isLoading } = useSWR<T>(url);

  return <>{children(data, isLoading, error)}</>;
};

// Usage
<DataLoader url="/api/issues/">
  {(issues, loading, error) => {
    if (loading) return <Loading />;
    if (error) return <Error error={error} />;
    return <IssueList issues={issues} />;
  }}
</DataLoader>
```

---

#### 3. Higher-Order Components (HOC) ⚠️ LESS COMMON

**Pattern:** Wrap component to add functionality

```typescript
// withAuth HOC
const withAuth = <P extends object>(
  Component: React.ComponentType<P>
) => {
  return (props: P) => {
    const { user, isLoading } = useAuth();

    if (isLoading) return <Loading />;
    if (!user) return <Redirect to="/login" />;

    return <Component {...props} />;
  };
};

// Usage
const ProtectedPage = withAuth(DashboardPage);
```

**⚠️ Note:** Hooks are preferred over HOCs in modern React. Use HOCs only when hooks don't fit.

---

### Component Best Practices ✅ CURRENT

#### 1. Keep Components Pure

```typescript
// ✅ GOOD: Pure component
const IssueCard = ({ issue }: { issue: TIssue }) => {
  return (
    <div>
      <h3>{issue.title}</h3>
      <span>{issue.priority}</span>
    </div>
  );
};

// ❌ BAD: Side effects in render
const IssueCard = ({ issue }: { issue: TIssue }) => {
  // DON'T mutate props!
  issue.title = issue.title.toUpperCase();

  // DON'T make API calls in render!
  fetch('/api/log').then(...);

  return <div>{issue.title}</div>;
};
```

---

#### 2. Use Composition Over Props Drilling

```typescript
// ❌ BAD: Props drilling
const App = () => {
  const user = useUser();
  return <Layout user={user} />;
};

const Layout = ({ user }) => <Sidebar user={user} />;
const Sidebar = ({ user }) => <UserMenu user={user} />;
const UserMenu = ({ user }) => <div>{user.name}</div>;

// ✅ GOOD: Context
const UserContext = createContext<TUser | null>(null);

const App = () => {
  const user = useUser();
  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
};

const UserMenu = () => {
  const user = useContext(UserContext);
  return <div>{user.name}</div>;
};
```

---

#### 3. Extract Reusable Logic to Hooks

```typescript
// ✅ GOOD: Custom hook
const useIssueActions = (issueId: string) => {
  const store = useIssueStore();

  const updatePriority = useCallback(
    (priority: TPriority) => {
      return store.updateIssue(issueId, { priority });
    },
    [store, issueId]
  );

  const deleteIssue = useCallback(() => {
    return store.deleteIssue(issueId);
  }, [store, issueId]);

  return { updatePriority, deleteIssue };
};

// Usage in multiple components
const IssueCard = ({ issue }: { issue: TIssue }) => {
  const { updatePriority, deleteIssue } = useIssueActions(issue.id);

  return (
    <div>
      <PrioritySelect onChange={updatePriority} />
      <button onClick={deleteIssue}>Delete</button>
    </div>
  );
};
```

---

## State Management with MobX

### Store Architecture ✅ CURRENT

**Location:** `/apps/web/core/store/`

```
store/
├── root.store.ts               # Root store (combines all stores)
├── user.store.ts               # User state
├── workspace.store.ts          # Workspace state
├── project/
│   ├── project.store.ts       # Project CRUD
│   └── project-member.store.ts # Project members
├── issue/
│   ├── issue.store.ts         # Issue CRUD
│   ├── issue-filter.store.ts  # Filter state
│   └── issue-kanban.store.ts  # Kanban view state
└── cycle/
    ├── cycle.store.ts
    └── active-cycle.store.ts
```

---

### Store Pattern ✅ CURRENT

```typescript
// /apps/web/core/store/issue/issue.store.ts
import { makeObservable, observable, action, computed, runInAction } from "mobx";
import { IssueService } from "~/services/issue.service";
import type { TIssue } from "@plane/types";

export class IssueStore {
  // Observable state
  issues = new Map<string, TIssue>();
  isLoading = false;
  error: string | null = null;

  // Root store reference
  rootStore;

  constructor(rootStore: RootStore) {
    this.rootStore = rootStore;

    makeObservable(this, {
      // Observables
      issues: observable,
      isLoading: observable,
      error: observable,

      // Actions
      fetchIssues: action,
      addIssue: action,
      updateIssue: action,
      deleteIssue: action,

      // Computed
      issueCount: computed,
      sortedIssues: computed,
    });
  }

  // Computed values (cached)
  get issueCount() {
    return this.issues.size;
  }

  get sortedIssues() {
    return Array.from(this.issues.values()).sort(
      (a, b) => new Date(b.created_at).getTime() - new Date(a.created_at).getTime()
    );
  }

  // Actions (modify state)
  fetchIssues = async (projectId: string) => {
    this.isLoading = true;
    this.error = null;

    try {
      const issues = await IssueService.getIssues(projectId);

      runInAction(() => {
        issues.forEach((issue) => this.issues.set(issue.id, issue));
        this.isLoading = false;
      });
    } catch (error) {
      runInAction(() => {
        this.error = error.message;
        this.isLoading = false;
      });
    }
  };

  addIssue = (issue: TIssue) => {
    this.issues.set(issue.id, issue);
  };

  updateIssue = (id: string, data: Partial<TIssue>) => {
    const issue = this.issues.get(id);
    if (issue) {
      this.issues.set(id, { ...issue, ...data });
    }
  };

  deleteIssue = (id: string) => {
    this.issues.delete(id);
  };
}
```

---

### Using Stores in Components ✅ CURRENT

```typescript
import { observer } from "mobx-react";
import { useIssueStore } from "~/stores";

// Observer makes component reactive
const IssueList = observer(() => {
  const issueStore = useIssueStore();

  useEffect(() => {
    issueStore.fetchIssues(projectId);
  }, [issueStore, projectId]);

  // Automatically re-renders when store changes
  return (
    <div>
      {issueStore.isLoading && <Loading />}
      {issueStore.error && <Error message={issueStore.error} />}

      <h1>Issues ({issueStore.issueCount})</h1>

      {issueStore.sortedIssues.map((issue) => (
        <IssueCard key={issue.id} issue={issue} />
      ))}
    </div>
  );
});
```

**⚠️ Common Pitfall:**

```typescript
// ❌ Doesn't work - not wrapped in observer
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

---

## Data Fetching Strategies

### SWR for Server State ✅ CURRENT

**Pattern:** Use SWR for data that comes from the server

```typescript
import useSWR from "swr";
import { IssueService } from "~/services/issue.service";

const IssueList = ({ projectId }: { projectId: string }) => {
  const { data: issues, error, mutate } = useSWR(
    `/api/projects/${projectId}/issues/`,
    () => IssueService.getIssues(projectId),
    {
      revalidateOnFocus: true,      // Refetch on window focus
      revalidateOnReconnect: true,  // Refetch on reconnect
      dedupingInterval: 2000,       // Dedupe requests within 2s
    }
  );

  if (error) return <Error error={error} />;
  if (!issues) return <Loading />;

  return (
    <div>
      {issues.map((issue) => (
        <IssueCard key={issue.id} issue={issue} onUpdate={() => mutate()} />
      ))}
    </div>
  );
};
```

---

### Optimistic Updates with SWR ✅ CURRENT

```typescript
import { useSWRConfig } from "swr";

const useUpdateIssue = () => {
  const { mutate } = useSWRConfig();

  const updateIssue = async (issueId: string, data: Partial<TIssue>) => {
    const key = `/api/issues/${issueId}/`;

    // Optimistic update
    mutate(
      key,
      (current: TIssue) => ({ ...current, ...data }),
      false  // Don't revalidate yet
    );

    try {
      // Send to server
      const updated = await IssueService.update(issueId, data);

      // Update with server response
      mutate(key, updated);

      return updated;
    } catch (error) {
      // Rollback on error
      mutate(key);
      throw error;
    }
  };

  return { updateIssue };
};
```

---

### React Router Loaders vs SWR

**When to use Loaders:**
- Initial page data
- Data needed before render
- SEO-critical data

**When to use SWR:**
- Client-side updates
- Polling data
- Optional data
- Data that changes frequently

**✅ Best Practice: Use Both**

```typescript
// Loader: Initial data
export async function loader({ params }: LoaderFunctionArgs) {
  const issues = await IssueService.getIssues(params.projectId);
  return json({ issues });
}

// Component: SWR for real-time updates
export default function IssuesPage() {
  const initialData = useLoaderData<typeof loader>();

  // SWR with initial data from loader
  const { data: issues } = useSWR(
    `/api/projects/${projectId}/issues/`,
    fetcher,
    {
      fallbackData: initialData.issues,  // Use loader data initially
      refreshInterval: 30000,            // Poll every 30s
    }
  );

  return <IssueList issues={issues} />;
}
```

---

## TypeScript Best Practices

### Strict Type Safety ✅ CURRENT

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictPropertyInitialization": true
  }
}
```

---

### Type Definitions ✅ CURRENT

```typescript
// /packages/types/src/issue.d.ts
export type TIssue = {
  id: string;
  name: string;
  description_html: string;
  priority: TIssuePriority;
  state: string;
  state_detail: TState;
  project: string;
  assignees: string[];
  assignee_details: TUser[];
  created_at: string;
  updated_at: string;
};

export type TIssuePriority = "urgent" | "high" | "medium" | "low" | "none";

// Use in components
const IssueCard = ({ issue }: { issue: TIssue }) => {
  // TypeScript validates all properties
  return (
    <div>
      <h3>{issue.name}</h3>
      <span>{issue.priority}</span>  {/* Autocomplete! */}
    </div>
  );
};
```

---

### Generic Components ✅ CURRENT

```typescript
// Generic list component
type TListProps<T> = {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string;
};

const List = <T,>({ items, renderItem, keyExtractor }: TListProps<T>) => {
  return (
    <div>
      {items.map((item) => (
        <div key={keyExtractor(item)}>{renderItem(item)}</div>
      ))}
    </div>
  );
};

// Usage with type inference
<List
  items={issues}
  renderItem={(issue) => <IssueCard issue={issue} />}
  keyExtractor={(issue) => issue.id}
/>
```

---

### Discriminated Unions ✅ CURRENT

```typescript
type TApiState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "error"; error: string }
  | { status: "success"; data: T };

const IssueList = () => {
  const [state, setState] = useState<TApiState<TIssue[]>>({ status: "idle" });

  // TypeScript narrows type based on status
  switch (state.status) {
    case "idle":
      return <div>Click to load issues</div>;

    case "loading":
      return <Loading />;

    case "error":
      return <Error message={state.error} />;  // TS knows error exists

    case "success":
      return <IssueCards issues={state.data} />;  // TS knows data exists
  }
};
```

---

## Performance Optimization

### 1. Memoization ✅ CURRENT

```typescript
import { memo, useMemo, useCallback } from "react";

// Memo component (skip render if props unchanged)
const IssueCard = memo(({ issue }: { issue: TIssue }) => {
  return <div>{issue.title}</div>;
});

// useMemo for expensive calculations
const ExpensiveComponent = ({ issues }: { issues: TIssue[] }) => {
  const sortedAndFiltered = useMemo(() => {
    return issues
      .filter((i) => i.priority === "high")
      .sort((a, b) => a.title.localeCompare(b.title));
  }, [issues]);

  return <List items={sortedAndFiltered} />;
};

// useCallback for stable function references
const IssueList = ({ issues }: { issues: TIssue[] }) => {
  const handleUpdate = useCallback((id: string, data: Partial<TIssue>) => {
    IssueService.update(id, data);
  }, []);  // Stable reference

  return (
    <div>
      {issues.map((issue) => (
        <IssueCard key={issue.id} issue={issue} onUpdate={handleUpdate} />
      ))}
    </div>
  );
};
```

**⚠️ Warning:** Don't over-optimize!

```typescript
// ❌ BAD: Premature optimization
const SimpleComponent = memo(({ name }: { name: string }) => {
  const uppercased = useMemo(() => name.toUpperCase(), [name]);
  return <div>{uppercased}</div>;
});

// ✅ GOOD: Simple and fast
const SimpleComponent = ({ name }: { name: string }) => {
  return <div>{name.toUpperCase()}</div>;
};
```

---

### 2. Code Splitting ✅ CURRENT

```typescript
import { lazy, Suspense } from "react";

// Lazy load components
const IssueDetail = lazy(() => import("./IssueDetail"));
const ProjectSettings = lazy(() => import("./ProjectSettings"));

const App = () => {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/issues/:id" element={<IssueDetail />} />
        <Route path="/settings" element={<ProjectSettings />} />
      </Routes>
    </Suspense>
  );
};
```

---

### 3. Virtual Scrolling ✅ CURRENT

```typescript
import { useVirtualizer } from "@tanstack/react-virtual";
import { useRef } from "react";

const IssueList = ({ issues }: { issues: TIssue[] }) => {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: issues.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 60,  // Row height
    overscan: 5,             // Render 5 extra rows
  });

  return (
    <div ref={parentRef} style={{ height: "600px", overflow: "auto" }}>
      <div style={{ height: `${virtualizer.getTotalSize()}px` }}>
        {virtualizer.getVirtualItems().map((virtualRow) => (
          <div
            key={virtualRow.key}
            style={{
              position: "absolute",
              top: 0,
              left: 0,
              width: "100%",
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`,
            }}
          >
            <IssueCard issue={issues[virtualRow.index]} />
          </div>
        ))}
      </div>
    </div>
  );
};
```

---

## Common Patterns

### 1. Controlled Components ✅ CURRENT

```typescript
const CreateIssueForm = () => {
  const [title, setTitle] = useState("");
  const [description, setDescription] = useState("");

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    await createIssue({ title, description });
    setTitle("");
    setDescription("");
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Title"
      />
      <textarea
        value={description}
        onChange={(e) => setDescription(e.target.value)}
        placeholder="Description"
      />
      <button type="submit">Create</button>
    </form>
  );
};
```

---

### 2. Portal Pattern ✅ CURRENT

```typescript
import { createPortal } from "react-dom";

const Modal = ({ children, isOpen }: { children: React.ReactNode; isOpen: boolean }) => {
  if (!isOpen) return null;

  return createPortal(
    <div className="modal-overlay">
      <div className="modal-content">{children}</div>
    </div>,
    document.body  // Render at body level
  );
};
```

---

### 3. Error Boundaries ✅ CURRENT

```typescript
class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { hasError: boolean; error: Error | null }
> {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error("Error caught by boundary:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h1>Something went wrong</h1>
          <pre>{this.state.error?.message}</pre>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <IssueList />
</ErrorBoundary>
```

---

## Anti-Patterns to Avoid

### ❌ Don't Mutate State

```typescript
// ❌ BAD
const handleUpdate = () => {
  issues.push(newIssue);  // Mutates array!
  setIssues(issues);
};

// ✅ GOOD
const handleUpdate = () => {
  setIssues([...issues, newIssue]);  // New array
};
```

---

### ❌ Don't Use Index as Key

```typescript
// ❌ BAD
{issues.map((issue, index) => (
  <IssueCard key={index} issue={issue} />
))}

// ✅ GOOD
{issues.map((issue) => (
  <IssueCard key={issue.id} issue={issue} />
))}
```

---

### ❌ Don't Fetch in useEffect Everywhere

```typescript
// ❌ OLD WAY: useEffect for data fetching
const IssueList = () => {
  const [issues, setIssues] = useState([]);

  useEffect(() => {
    fetch("/api/issues/").then(setIssues);
  }, []);

  return <div>{issues.map(...)}</div>;
};

// ✅ NEW WAY: React Router loader
export async function loader() {
  return json({ issues: await fetchIssues() });
}

export default function IssueList() {
  const { issues } = useLoaderData<typeof loader>();
  return <div>{issues.map(...)}</div>;
}
```

---

## Next Steps

1. **[PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md)** - Code standards
2. **[HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)** - Build features
3. **[TESTING_GUIDE.md](./TESTING_GUIDE.md)** - Test your code

---

**🎉 You're now a React expert in the Plane codebase!**

---

**Last Updated:** November 19, 2025
**Maintainers:** Plane Learning Docs Team
**License:** AGPL-3.0
