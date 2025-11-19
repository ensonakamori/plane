# First Contributions - Beginner-Friendly Tasks

**Start Contributing!** This guide provides easy, beginner-friendly tasks to help you make your first contributions to Plane.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 20-30 minutes
**📊 Difficulty Level:** Beginner
**🎯 Target Audience:** New contributors wanting to get started

---

## Table of Contents

1. [Before You Start](#before-you-start)
2. [Good First Issues](#good-first-issues)
3. [Documentation Improvements](#documentation-improvements)
4. [UI/UX Enhancements](#uiux-enhancements)
5. [Code Quality Tasks](#code-quality-tasks)
6. [Testing Tasks](#testing-tasks)
7. [How to Submit a PR](#how-to-submit-a-pr)
8. [Getting Help](#getting-help)

---

## Before You Start

### Prerequisites

✅ **You should have:**
- Completed [GETTING_STARTED.md](./GETTING_STARTED.md) setup
- Read [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md)
- Browsed [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md)
- Development environment running locally

✅ **Skills needed:**
- Basic Git knowledge (clone, commit, push, PR)
- React fundamentals (if doing frontend)
- Basic Python (if doing backend)
- Willingness to learn!

---

### Find an Issue

**GitHub Labels for Beginners:**
- `good first issue` - Perfect for first-time contributors
- `documentation` - Documentation improvements
- `help wanted` - Community help needed
- `beginner friendly` - Easier tasks

**Browse Issues:**
1. Go to [github.com/makeplane/plane/issues](https://github.com/makeplane/plane/issues)
2. Filter by label: `good first issue`
3. Read issue description
4. Comment "I'd like to work on this"
5. Wait for maintainer approval

---

## Good First Issues

### 1. Fix TypeScript `any` Types ⭐ Beginner

**Difficulty:** ⭐ Easy
**Time:** 15-30 minutes
**Skills:** TypeScript basics

**Task:** Replace `any` types with proper types

**Example:**
```typescript
// Find this (BAD):
const handleClick = (data: any) => {
  console.log(data.name);
};

// Replace with (GOOD):
import type { TIssue } from "@plane/types";

const handleClick = (data: TIssue) => {
  console.log(data.name);
};
```

**Where to Look:**
```bash
# Find files with 'any' type
grep -r ": any" apps/web/core/components/
```

**Steps:**
1. Find component with `any` type
2. Determine correct type from `@plane/types`
3. Replace `any` with proper type
4. Run `pnpm check:types` to verify
5. Submit PR

**Impact:** Improves type safety, catches bugs earlier

---

### 2. Add Loading States ⭐ Beginner

**Difficulty:** ⭐ Easy
**Time:** 20-40 minutes
**Skills:** React basics

**Task:** Add loading spinners to components that fetch data

**Example:**
```typescript
// Find this (MISSING LOADING):
const IssueList = () => {
  const { data: issues } = useSWR('/api/issues/');

  return (
    <div>
      {issues?.map(issue => <IssueCard key={issue.id} issue={issue} />)}
    </div>
  );
};

// Add loading state (BETTER):
import { Loading } from "@plane/ui";

const IssueList = () => {
  const { data: issues, isLoading } = useSWR('/api/issues/');

  if (isLoading) return <Loading />;

  return (
    <div>
      {issues?.map(issue => <IssueCard key={issue.id} issue={issue} />)}
    </div>
  );
};
```

**Where to Look:**
- Components using `useSWR` without `isLoading` check
- Components using `useLoaderData` without suspense

**Steps:**
1. Find component missing loading state
2. Add loading check
3. Import and use `<Loading />` from `@plane/ui`
4. Test in browser
5. Submit PR

---

### 3. Add Error Boundaries ⭐⭐ Intermediate

**Difficulty:** ⭐⭐ Medium
**Time:** 30-60 minutes
**Skills:** React, error handling

**Task:** Wrap components in error boundaries to catch errors

**Example:**
```typescript
// apps/web/core/components/common/error-boundary.tsx
import React from "react";

type Props = {
  children: React.ReactNode;
  fallback?: React.ReactNode;
};

type State = {
  hasError: boolean;
  error: Error | null;
};

export class ErrorBoundary extends React.Component<Props, State> {
  constructor(props: Props) {
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
        this.props.fallback || (
          <div className="p-4 bg-red-50 border border-red-200 rounded">
            <h2 className="text-red-800 font-semibold">Something went wrong</h2>
            <pre className="text-sm text-red-600 mt-2">
              {this.state.error?.message}
            </pre>
          </div>
        )
      );
    }

    return this.props.children;
  }
}

// Usage:
<ErrorBoundary>
  <IssueList />
</ErrorBoundary>
```

**Where to Apply:**
- Around route components
- Around third-party components
- Around complex features

---

### 4. Improve Button Accessibility ⭐ Beginner

**Difficulty:** ⭐ Easy
**Time:** 15-30 minutes
**Skills:** HTML, accessibility basics

**Task:** Add ARIA labels and keyboard support to buttons

**Example:**
```typescript
// Find this (MISSING ACCESSIBILITY):
<button onClick={handleDelete}>
  <TrashIcon />
</button>

// Improve (BETTER):
<button
  onClick={handleDelete}
  aria-label="Delete issue"
  title="Delete issue"
>
  <TrashIcon />
</button>
```

**Checklist:**
- ✅ Add `aria-label` for icon-only buttons
- ✅ Add `title` for tooltips
- ✅ Ensure keyboard navigation works (Tab key)
- ✅ Add `disabled` state styling
- ✅ Test with screen reader (optional but great!)

---

## Documentation Improvements

### 1. Fix Typos ⭐ Super Easy

**Difficulty:** ⭐ Super Easy
**Time:** 5-10 minutes
**Skills:** English reading/writing

**Task:** Find and fix typos in documentation

**Where to Look:**
- `/docs/learning/*.md`
- Code comments
- README files

**Steps:**
1. Read documentation
2. Note typos or grammar issues
3. Fix in editor
4. Submit PR with descriptive message

**Example PR Title:**
- ✅ "docs: fix typo in ARCHITECTURE_OVERVIEW.md (recieve → receive)"
- ❌ "fix typo"

---

### 2. Add Code Examples ⭐⭐ Easy-Medium

**Difficulty:** ⭐⭐ Medium
**Time:** 30-60 minutes
**Skills:** React or Django knowledge

**Task:** Add code examples to documentation

**Example:**
```markdown
<!-- BEFORE (explanation only) -->
## Using MobX Stores

MobX stores manage application state. Import the store and use it in your component.

<!-- AFTER (with code example) -->
## Using MobX Stores

MobX stores manage application state. Import the store and use it in your component.

**Example:**
```typescript
import { observer } from "mobx-react";
import { useIssueStore } from "~/stores";

const IssueList = observer(() => {
  const issueStore = useIssueStore();

  return (
    <div>
      <h1>Issues ({issueStore.count})</h1>
      {issueStore.issues.map(issue => (
        <IssueCard key={issue.id} issue={issue} />
      ))}
    </div>
  );
});
```
\`\`\`

**Where to Add:**
- Sections with no examples
- Complex concepts that need illustration
- API documentation

---

### 3. Create Diagrams ⭐⭐⭐ Medium-Hard

**Difficulty:** ⭐⭐⭐ Hard
**Time:** 1-3 hours
**Skills:** System understanding, diagram tools

**Task:** Create visual diagrams for documentation

**Tools:**
- [Excalidraw](https://excalidraw.com/) - Simple diagrams
- [Mermaid](https://mermaid.js.org/) - Code-based diagrams
- [Draw.io](https://draw.io/) - Detailed diagrams

**Example Topics:**
- Component hierarchy
- Data flow diagrams
- Database schema
- API request flow
- Authentication flow

---

## UI/UX Enhancements

### 1. Improve Button States ⭐⭐ Medium

**Difficulty:** ⭐⭐ Medium
**Time:** 30-60 minutes
**Skills:** React, Tailwind CSS

**Task:** Add hover, focus, active, and disabled states to buttons

**Example:**
```typescript
// BEFORE (basic button)
<button className="px-4 py-2 bg-blue-600 text-white rounded">
  Click me
</button>

// AFTER (with states)
<button
  className={`
    px-4 py-2 rounded
    bg-blue-600 text-white
    hover:bg-blue-700
    focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
    active:bg-blue-800
    disabled:opacity-50 disabled:cursor-not-allowed
    transition-colors duration-200
  `}
  disabled={isLoading}
>
  {isLoading ? "Loading..." : "Click me"}
</button>
```

**Where to Apply:**
- Primary action buttons
- Form submit buttons
- Delete/destructive buttons

---

### 2. Add Keyboard Shortcuts ⭐⭐⭐ Medium-Hard

**Difficulty:** ⭐⭐⭐ Hard
**Time:** 2-4 hours
**Skills:** React, event handling

**Task:** Add keyboard shortcuts for common actions

**Example:**
```typescript
import { useEffect } from "react";

const useKeyboardShortcut = (key: string, callback: () => void) => {
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === key && (e.metaKey || e.ctrlKey)) {
        e.preventDefault();
        callback();
      }
    };

    window.addEventListener("keydown", handleKeyDown);
    return () => window.removeEventListener("keydown", handleKeyDown);
  }, [key, callback]);
};

// Usage:
const IssueList = () => {
  const [isModalOpen, setIsModalOpen] = useState(false);

  // Cmd/Ctrl + K to open create modal
  useKeyboardShortcut("k", () => setIsModalOpen(true));

  return (
    <div>
      <button onClick={() => setIsModalOpen(true)}>
        Create Issue (⌘K)
      </button>
      {/* ... */}
    </div>
  );
};
```

**Common Shortcuts to Add:**
- `Cmd/Ctrl + K` - Quick search
- `Cmd/Ctrl + N` - Create new item
- `Cmd/Ctrl + S` - Save
- `Esc` - Close modal
- `/` - Focus search

---

## Code Quality Tasks

### 1. Extract Repeated Code ⭐⭐ Medium

**Difficulty:** ⭐⭐ Medium
**Time:** 30-90 minutes
**Skills:** React, refactoring

**Task:** Find repeated code and extract to reusable components/hooks

**Example:**
```typescript
// BEFORE (repeated code in multiple components)
const IssueCard1 = () => {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleDelete = async () => {
    setIsLoading(true);
    try {
      await deleteIssue(id);
    } catch (e) {
      setError(e.message);
    } finally {
      setIsLoading(false);
    }
  };
};

const IssueCard2 = () => {
  // Same pattern repeated!
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleDelete = async () => {
    setIsLoading(true);
    try {
      await deleteIssue(id);
    } catch (e) {
      setError(e.message);
    } finally {
      setIsLoading(false);
    }
  };
};

// AFTER (extracted to hook)
const useAsyncAction = <T,>(action: () => Promise<T>) => {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const execute = async () => {
    setIsLoading(true);
    setError(null);
    try {
      return await action();
    } catch (e) {
      setError(e.message);
      throw e;
    } finally {
      setIsLoading(false);
    }
  };

  return { execute, isLoading, error };
};

// Usage (much cleaner!)
const IssueCard = () => {
  const { execute: handleDelete, isLoading, error } = useAsyncAction(
    () => deleteIssue(id)
  );

  return (
    <div>
      <button onClick={handleDelete} disabled={isLoading}>
        {isLoading ? "Deleting..." : "Delete"}
      </button>
      {error && <div className="text-red-500">{error}</div>}
    </div>
  );
};
```

---

### 2. Add PropTypes/TypeScript ⭐ Easy

**Difficulty:** ⭐ Easy
**Time:** 15-30 minutes per component
**Skills:** TypeScript

**Task:** Add proper TypeScript types to components

**Example:**
```typescript
// BEFORE (no types)
const IssueCard = ({ issue, onUpdate, onDelete }) => {
  return <div>{issue.title}</div>;
};

// AFTER (with types)
import type { TIssue } from "@plane/types";

type Props = {
  issue: TIssue;
  onUpdate: (id: string, data: Partial<TIssue>) => void;
  onDelete: (id: string) => void;
};

const IssueCard = ({ issue, onUpdate, onDelete }: Props) => {
  return <div>{issue.title}</div>;
};
```

---

## Testing Tasks

### 1. Add Unit Tests ⭐⭐ Medium

**Difficulty:** ⭐⭐ Medium
**Time:** 30-60 minutes per component
**Skills:** React Testing Library

**Task:** Add tests for untested components

**Example:**
```typescript
// /apps/web/core/components/issues/__tests__/issue-card.test.tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { IssueCard } from "../issue-card";

describe("IssueCard", () => {
  const mockIssue = {
    id: "123",
    title: "Test Issue",
    priority: "high",
    state: { id: "456", name: "In Progress" },
  };

  it("renders issue title", () => {
    render(<IssueCard issue={mockIssue} />);
    expect(screen.getByText("Test Issue")).toBeInTheDocument();
  });

  it("renders priority badge", () => {
    render(<IssueCard issue={mockIssue} />);
    expect(screen.getByText("high")).toBeInTheDocument();
  });

  it("calls onDelete when delete button clicked", () => {
    const onDelete = jest.fn();
    render(<IssueCard issue={mockIssue} onDelete={onDelete} />);

    fireEvent.click(screen.getByRole("button", { name: /delete/i }));
    expect(onDelete).toHaveBeenCalledWith("123");
  });
});
```

**Run Tests:**
```bash
pnpm --filter web test
```

---

### 2. Add Backend Tests ⭐⭐⭐ Medium-Hard

**Difficulty:** ⭐⭐⭐ Hard
**Time:** 1-2 hours per endpoint
**Skills:** Django, Python testing

**Example:**
```python
# /apps/api/plane/tests/unit/test_issue_api.py
from django.test import TestCase
from rest_framework.test import APIClient
from plane.db.models import Issue, Project, Workspace, User

class IssueAPITestCase(TestCase):
    def setUp(self):
        self.client = APIClient()
        self.user = User.objects.create_user(
            email="test@example.com",
            password="password123"
        )
        self.workspace = Workspace.objects.create(
            name="Test Workspace",
            owner=self.user
        )
        self.project = Project.objects.create(
            name="Test Project",
            workspace=self.workspace
        )
        self.client.force_authenticate(user=self.user)

    def test_create_issue(self):
        url = f"/api/workspaces/{self.workspace.slug}/projects/{self.project.id}/issues/"
        data = {
            "name": "Test Issue",
            "priority": "high"
        }

        response = self.client.post(url, data, format="json")

        self.assertEqual(response.status_code, 201)
        self.assertEqual(response.data["name"], "Test Issue")
        self.assertEqual(Issue.objects.count(), 1)

    def test_list_issues(self):
        # Create test issues
        Issue.objects.create(name="Issue 1", project=self.project)
        Issue.objects.create(name="Issue 2", project=self.project)

        url = f"/api/workspaces/{self.workspace.slug}/projects/{self.project.id}/issues/"
        response = self.client.get(url)

        self.assertEqual(response.status_code, 200)
        self.assertEqual(len(response.data), 2)
```

**Run Tests:**
```bash
cd apps/api
python manage.py test
```

---

## How to Submit a PR

### 1. Fork and Clone

```bash
# Fork on GitHub first, then:
git clone https://github.com/YOUR_USERNAME/plane.git
cd plane
git remote add upstream https://github.com/makeplane/plane.git
```

---

### 2. Create Branch

```bash
# Create feature branch
git checkout -b fix/improve-button-accessibility

# Good branch names:
# - fix/issue-card-loading-state
# - feat/add-keyboard-shortcuts
# - docs/improve-architecture-guide
# - test/add-issue-api-tests
```

---

### 3. Make Changes

```bash
# Make your changes
# Test locally
pnpm check        # Run all checks
pnpm test         # Run tests

# Commit with good message
git add .
git commit -m "feat: add loading state to IssueCard component

- Add isLoading check to prevent empty state
- Import Loading component from @plane/ui
- Add tests for loading state

Fixes #123"
```

**Good Commit Message Format:**
```
<type>: <short description>

<longer description (optional)>
<why this change was needed>
<what was changed>

Fixes #<issue number>
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation
- `test` - Adding tests
- `refactor` - Code refactoring
- `style` - Code style changes
- `chore` - Build/tooling changes

---

### 4. Push and Create PR

```bash
# Push to your fork
git push origin fix/improve-button-accessibility

# Go to GitHub and create PR
# Fill in PR template
# Link to issue number
# Add screenshots if UI change
```

**Good PR Description:**

```markdown
## What does this PR do?

Adds loading state to IssueCard component to prevent showing empty state while data is loading.

## Before / After

**Before:**
[Screenshot showing empty state flash]

**After:**
[Screenshot showing loading spinner]

## How to test

1. Open issue list page
2. Observe loading spinner appears while fetching
3. Loading spinner disappears when data loads

## Checklist

- [x] Tests added
- [x] TypeScript types updated
- [x] Documentation updated
- [x] Local testing completed

Fixes #123
```

---

## Getting Help

### Stuck? Ask for Help!

**Where to Ask:**
- 💬 **Discord:** [plane.so/discord](https://plane.so/discord) - Fastest response
- 💬 **GitHub Discussions:** [github.com/makeplane/plane/discussions](https://github.com/makeplane/plane/discussions)
- 📧 **Email:** support@plane.so

**When Asking:**
1. Describe what you're trying to do
2. Show what you've tried
3. Include error messages
4. Share relevant code snippets
5. Mention which docs you've read

**Example Good Question:**
```
I'm trying to add a loading state to IssueCard but getting this error:

"Cannot find module '@plane/ui'"

I've read FRONTEND_ARCHITECTURE.md and tried:
- pnpm install
- Restarting VS Code

My import:
import { Loading } from "@plane/ui";

Any ideas?
```

---

## Next Steps

After your first PR is merged:

1. ✅ **Celebrate!** 🎉 You're a contributor!
2. ✅ **Find another issue** - Build momentum
3. ✅ **Read more docs** - Deepen understanding
4. ✅ **Help others** - Answer questions in Discord

**More Advanced Tasks:**
- [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md) - Build features
- [CODE_TOURS.md](./CODE_TOURS.md) - Understand complex features
- [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md) - Professional workflow

---

**🎉 Welcome to the Plane community!**

We're excited to have you contributing. Every contribution, no matter how small, makes Plane better for everyone.

---

**Last Updated:** November 19, 2025
**Maintainers:** Plane Learning Docs Team
**License:** AGPL-3.0
