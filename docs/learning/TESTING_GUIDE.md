# Testing Guide - Frontend & Backend Testing Strategies

**Write reliable tests!** This guide covers testing strategies for React components, Django models/views, integration testing, and E2E testing in Plane.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 1 hour
**📊 Difficulty Level:** Intermediate to Advanced
**🎯 Target Audience:** Developers writing tests

---

## Table of Contents

1. [Testing Philosophy](#testing-philosophy)
2. [Frontend Testing](#frontend-testing)
3. [Backend Testing](#backend-testing)
4. [Integration Testing](#integration-testing)
5. [E2E Testing](#e2e-testing)
6. [Test Coverage](#test-coverage)
7. [Best Practices](#best-practices)

---

## Testing Philosophy

### Testing Pyramid

```
        /\
       /E2E\          Few, slow, expensive
      /------\
     /Integra-\       Some, medium speed
    /tion Tests\
   /------------\
  /  Unit Tests  \    Many, fast, cheap
 /________________\
```

### What to Test

**✅ DO test:**
- Business logic
- User interactions
- Error handling
- Edge cases
- Critical paths

**❌ DON'T test:**
- Third-party libraries
- Implementation details
- Trivial code (getters/setters)

### Test Structure (AAA Pattern)

```typescript
it('updates issue priority', async () => {
  // Arrange - Set up test data
  const issue = createMockIssue({ priority: 'low' });

  // Act - Perform the action
  await updateIssuePriority(issue.id, 'high');

  // Assert - Verify the result
  expect(issue.priority).toBe('high');
});
```

---

## Frontend Testing

### Tech Stack ✅ CURRENT

- **Jest** - Test runner
- **React Testing Library** - Component testing
- **MSW (Mock Service Worker)** - API mocking
- **Playwright** - E2E testing

### Component Testing

**Test user interactions, not implementation**

```typescript
// components/issues/issue-card.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { IssueCard } from './issue-card';

describe('IssueCard', () => {
  it('renders issue name and priority', () => {
    const issue = {
      id: '1',
      name: 'Fix bug',
      priority: 'high'
    };

    render(<IssueCard issue={issue} />);

    expect(screen.getByText('Fix bug')).toBeInTheDocument();
    expect(screen.getByText('High')).toBeInTheDocument();
  });

  it('calls onUpdate when edit button is clicked', async () => {
    const issue = { id: '1', name: 'Fix bug', priority: 'high' };
    const onUpdate = jest.fn();

    render(<IssueCard issue={issue} onUpdate={onUpdate} />);

    fireEvent.click(screen.getByRole('button', { name: /edit/i }));

    expect(onUpdate).toHaveBeenCalledWith(issue);
  });

  it('shows loading state while saving', async () => {
    const issue = { id: '1', name: 'Fix bug', priority: 'high' };

    render(<IssueCard issue={issue} isSaving={true} />);

    expect(screen.getByText(/saving/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /edit/i })).toBeDisabled();
  });

  it('displays error message when save fails', async () => {
    const issue = { id: '1', name: 'Fix bug', priority: 'high' };
    const error = 'Failed to save';

    render(<IssueCard issue={issue} error={error} />);

    expect(screen.getByText(error)).toBeInTheDocument();
  });
});
```

### Testing Hooks

```typescript
// hooks/use-issues.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { useIssues } from './use-issues';

describe('useIssues', () => {
  it('fetches issues on mount', async () => {
    const { result } = renderHook(() => useIssues('project-1'));

    expect(result.current.isLoading).toBe(true);

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
      expect(result.current.issues).toHaveLength(3);
    });
  });

  it('handles fetch errors', async () => {
    // Mock API error
    server.use(
      rest.get('/api/issues/', (req, res, ctx) => {
        return res(ctx.status(500));
      })
    );

    const { result } = renderHook(() => useIssues('project-1'));

    await waitFor(() => {
      expect(result.current.error).toBeTruthy();
      expect(result.current.issues).toHaveLength(0);
    });
  });
});
```

### Mocking API Requests

```typescript
// setup-tests.ts
import { setupServer } from 'msw/node';
import { rest } from 'msw';

// Define request handlers
const handlers = [
  rest.get('/api/issues/', (req, res, ctx) => {
    return res(
      ctx.json([
        { id: '1', name: 'Bug fix', priority: 'high' },
        { id: '2', name: 'Feature', priority: 'medium' }
      ])
    );
  }),

  rest.post('/api/issues/', (req, res, ctx) => {
    const body = req.body as any;
    return res(
      ctx.status(201),
      ctx.json({ id: '3', ...body })
    );
  }),

  rest.patch('/api/issues/:id/', (req, res, ctx) => {
    return res(
      ctx.json({ id: req.params.id, ...req.body })
    );
  })
];

// Setup server
export const server = setupServer(...handlers);

// Start server before tests
beforeAll(() => server.listen());

// Reset handlers after each test
afterEach(() => server.resetHandlers());

// Stop server after tests
afterAll(() => server.close());
```

### Testing MobX Stores

```typescript
// stores/issue.store.test.ts
import { IssueStore } from './issue.store';

describe('IssueStore', () => {
  let store: IssueStore;

  beforeEach(() => {
    store = new IssueStore();
  });

  it('fetches and stores issues', async () => {
    await store.fetchIssues('workspace-1', 'project-1');

    expect(store.isLoading).toBe(false);
    expect(store.issues.size).toBe(2);
  });

  it('adds issue to store', () => {
    const issue = { id: '1', name: 'Test', priority: 'high' };

    store.addIssue(issue);

    expect(store.issues.get('1')).toEqual(issue);
  });

  it('updates existing issue', () => {
    const issue = { id: '1', name: 'Test', priority: 'low' };
    store.addIssue(issue);

    store.updateIssue('1', { priority: 'high' });

    expect(store.issues.get('1')?.priority).toBe('high');
  });

  it('handles fetch errors', async () => {
    // Mock error
    server.use(
      rest.get('/api/issues/', (req, res, ctx) => {
        return res(ctx.status(500));
      })
    );

    await store.fetchIssues('workspace-1', 'project-1');

    expect(store.error).toBeTruthy();
    expect(store.issues.size).toBe(0);
  });
});
```

### Snapshot Testing

```typescript
// components/issue-card.test.tsx
import { render } from '@testing-library/react';
import { IssueCard } from './issue-card';

it('matches snapshot', () => {
  const issue = {
    id: '1',
    name: 'Fix bug',
    priority: 'high',
    assignees: [
      { id: 'user-1', name: 'Alice' }
    ]
  };

  const { container } = render(<IssueCard issue={issue} />);

  expect(container).toMatchSnapshot();
});
```

---

## Backend Testing

### Django Test Setup

```python
# plane/app/tests/test_issues.py
from django.test import TestCase
from rest_framework.test import APITestCase, APIClient
from rest_framework import status
from plane.db.models import User, Workspace, Project, Issue

class IssueTestCase(APITestCase):
    def setUp(self):
        """Set up test data before each test"""
        # Create user
        self.user = User.objects.create_user(
            email='test@example.com',
            password='testpass123',
            username='testuser'
        )

        # Create workspace
        self.workspace = Workspace.objects.create(
            name='Test Workspace',
            slug='test-workspace',
            created_by=self.user
        )

        # Create project
        self.project = Project.objects.create(
            name='Test Project',
            workspace=self.workspace,
            created_by=self.user
        )

        # Authenticate client
        self.client = APIClient()
        self.client.force_authenticate(user=self.user)
```

### Testing Models

```python
from django.test import TestCase
from plane.db.models import Issue, Project

class IssueModelTest(TestCase):
    def setUp(self):
        self.project = Project.objects.create(name='Test Project')

    def test_create_issue(self):
        """Test creating an issue"""
        issue = Issue.objects.create(
            project=self.project,
            name='Test Issue',
            priority='high'
        )

        self.assertEqual(issue.name, 'Test Issue')
        self.assertEqual(issue.priority, 'high')
        self.assertIsNotNone(issue.id)

    def test_issue_sequence_id(self):
        """Test issue sequence ID auto-increments"""
        issue1 = Issue.objects.create(project=self.project, name='Issue 1')
        issue2 = Issue.objects.create(project=self.project, name='Issue 2')

        self.assertEqual(issue1.sequence_id, 1)
        self.assertEqual(issue2.sequence_id, 2)

    def test_issue_str_method(self):
        """Test __str__ method"""
        issue = Issue.objects.create(
            project=self.project,
            name='Test Issue'
        )

        self.assertEqual(str(issue), 'Test Issue')

    def test_soft_delete(self):
        """Test soft delete functionality"""
        issue = Issue.objects.create(project=self.project, name='Test')

        issue.delete()  # Soft delete

        self.assertIsNotNone(issue.deleted_at)
        self.assertEqual(Issue.objects.all().count(), 0)  # Filtered out
        self.assertEqual(Issue.all_objects.count(), 1)  # Still exists
```

### Testing API Endpoints

```python
from rest_framework.test import APITestCase
from rest_framework import status

class IssueAPITest(APITestCase):
    def setUp(self):
        # Setup from previous example
        pass

    def test_list_issues(self):
        """Test GET /api/issues/"""
        # Create test issues
        Issue.objects.create(project=self.project, name='Issue 1')
        Issue.objects.create(project=self.project, name='Issue 2')

        url = f'/api/workspaces/{self.workspace.slug}/projects/{self.project.id}/issues/'
        response = self.client.get(url)

        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data), 2)

    def test_create_issue(self):
        """Test POST /api/issues/"""
        url = f'/api/workspaces/{self.workspace.slug}/projects/{self.project.id}/issues/'
        data = {
            'name': 'New Issue',
            'priority': 'high'
        }

        response = self.client.post(url, data, format='json')

        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.data['name'], 'New Issue')
        self.assertEqual(Issue.objects.count(), 1)

    def test_update_issue(self):
        """Test PATCH /api/issues/{id}/"""
        issue = Issue.objects.create(project=self.project, name='Old Name')

        url = f'/api/workspaces/{self.workspace.slug}/projects/{self.project.id}/issues/{issue.id}/'
        data = {'name': 'New Name'}

        response = self.client.patch(url, data, format='json')

        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['name'], 'New Name')

        issue.refresh_from_db()
        self.assertEqual(issue.name, 'New Name')

    def test_delete_issue(self):
        """Test DELETE /api/issues/{id}/"""
        issue = Issue.objects.create(project=self.project, name='Test')

        url = f'/api/workspaces/{self.workspace.slug}/projects/{self.project.id}/issues/{issue.id}/'
        response = self.client.delete(url)

        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
        self.assertEqual(Issue.objects.count(), 0)

    def test_create_issue_unauthorized(self):
        """Test creating issue without authentication"""
        self.client.force_authenticate(user=None)  # Logout

        url = f'/api/workspaces/{self.workspace.slug}/projects/{self.project.id}/issues/'
        data = {'name': 'New Issue'}

        response = self.client.post(url, data, format='json')

        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)

    def test_create_issue_validation_error(self):
        """Test validation errors"""
        url = f'/api/workspaces/{self.workspace.slug}/projects/{self.project.id}/issues/'
        data = {}  # Missing required field 'name'

        response = self.client.post(url, data, format='json')

        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
        self.assertIn('name', response.data)
```

### Testing Serializers

```python
from django.test import TestCase
from plane.app.serializers import IssueSerializer

class IssueSerializerTest(TestCase):
    def test_serialize_issue(self):
        """Test serializing issue to JSON"""
        issue = Issue.objects.create(name='Test', priority='high')
        serializer = IssueSerializer(issue)

        self.assertEqual(serializer.data['name'], 'Test')
        self.assertEqual(serializer.data['priority'], 'high')

    def test_deserialize_issue(self):
        """Test deserializing JSON to issue"""
        data = {'name': 'Test', 'priority': 'high'}
        serializer = IssueSerializer(data=data)

        self.assertTrue(serializer.is_valid())
        issue = serializer.save()

        self.assertEqual(issue.name, 'Test')

    def test_validation_error(self):
        """Test serializer validation"""
        data = {'name': ''}  # Empty name
        serializer = IssueSerializer(data=data)

        self.assertFalse(serializer.is_valid())
        self.assertIn('name', serializer.errors)
```

### Running Django Tests

```bash
# Run all tests
python manage.py test

# Run specific app tests
python manage.py test plane.app.tests

# Run specific test file
python manage.py test plane.app.tests.test_issues

# Run specific test case
python manage.py test plane.app.tests.test_issues.IssueAPITest

# Run specific test method
python manage.py test plane.app.tests.test_issues.IssueAPITest.test_create_issue

# Run with coverage
coverage run --source='.' manage.py test
coverage report
coverage html  # Generate HTML report
```

---

## Integration Testing

### Frontend ↔ Backend Integration

```typescript
// integration/issue-flow.test.ts
describe('Issue Creation Flow', () => {
  beforeAll(async () => {
    // Start backend server
    await startTestServer();
  });

  afterAll(async () => {
    // Stop backend server
    await stopTestServer();
  });

  it('creates issue end-to-end', async () => {
    // 1. Login
    const { token } = await loginUser('test@example.com', 'password');

    // 2. Create issue
    const response = await fetch('/api/issues/', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        name: 'Test Issue',
        priority: 'high'
      })
    });

    expect(response.status).toBe(201);

    const issue = await response.json();
    expect(issue.name).toBe('Test Issue');

    // 3. Verify in database
    const dbIssue = await Issue.objects.get(id=issue.id);
    expect(dbIssue.name).toBe('Test Issue');
  });
});
```

---

## E2E Testing

### Playwright Tests ✅ CURRENT

```typescript
// e2e/issue-creation.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Issue Creation', () => {
  test.beforeEach(async ({ page }) => {
    // Login
    await page.goto('http://localhost:3000/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'password');
    await page.click('button[type="submit"]');

    // Navigate to issues
    await page.goto('http://localhost:3000/workspace/test/projects/proj-1/issues');
  });

  test('creates new issue', async ({ page }) => {
    // Click "Create Issue" button
    await page.click('text=Create Issue');

    // Fill form
    await page.fill('[name="name"]', 'E2E Test Issue');
    await page.selectOption('[name="priority"]', 'high');

    // Submit
    await page.click('button:has-text("Create")');

    // Wait for issue to appear
    await page.waitForSelector('text=E2E Test Issue');

    // Verify issue appears in list
    expect(await page.textContent('.issue-list')).toContain('E2E Test Issue');
  });

  test('shows validation error for empty name', async ({ page }) => {
    await page.click('text=Create Issue');

    // Submit without filling name
    await page.click('button:has-text("Create")');

    // Verify error message
    await expect(page.locator('text=Name is required')).toBeVisible();
  });

  test('filters issues by priority', async ({ page }) => {
    // Apply filter
    await page.selectOption('[name="priority-filter"]', 'high');

    // Wait for filtered results
    await page.waitForTimeout(500);

    // Verify only high priority issues shown
    const issues = await page.$$('.issue-card');
    for (const issue of issues) {
      const priority = await issue.$eval('.priority', el => el.textContent);
      expect(priority).toBe('High');
    }
  });
});
```

### Running E2E Tests

```bash
# Run all tests
pnpm playwright test

# Run specific test file
pnpm playwright test e2e/issue-creation.spec.ts

# Run in headed mode (see browser)
pnpm playwright test --headed

# Run in debug mode
pnpm playwright test --debug

# Generate test report
pnpm playwright show-report
```

---

## Test Coverage

### Measuring Coverage

```bash
# Frontend coverage
pnpm test --coverage

# Backend coverage
coverage run --source='.' manage.py test
coverage report
coverage html  # Open htmlcov/index.html
```

### Coverage Goals

- **Unit tests:** 70-80% coverage
- **Integration tests:** Critical paths covered
- **E2E tests:** User flows covered

**Don't aim for 100% coverage** - focus on critical code!

---

## Best Practices

### DO

- ✅ Test behavior, not implementation
- ✅ Write descriptive test names
- ✅ Keep tests independent (no shared state)
- ✅ Use factories/fixtures for test data
- ✅ Test edge cases and errors
- ✅ Mock external dependencies
- ✅ Test critical user flows

### DON'T

- ❌ Test third-party code
- ❌ Test trivial code
- ❌ Depend on test execution order
- ❌ Use production database
- ❌ Hardcode IDs or dates
- ❌ Test implementation details
- ❌ Write slow tests

### Quick Tips

**1. Use factories for test data**

```python
# factories.py
import factory
from plane.db.models import Issue

class IssueFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = Issue

    name = factory.Sequence(lambda n: f'Issue {n}')
    priority = 'medium'

# In tests
issue = IssueFactory()
issues = IssueFactory.create_batch(5)
```

**2. Reset state between tests**

```typescript
afterEach(() => {
  // Clear store
  issueStore.issues.clear();

  // Clear localStorage
  localStorage.clear();

  // Reset mocks
  jest.clearAllMocks();
});
```

**3. Use meaningful assertions**

```typescript
// ❌ BAD
expect(issues.length > 0).toBe(true);

// ✅ GOOD
expect(issues).toHaveLength(3);
expect(issues[0].name).toBe('Expected Name');
```

---

## Next Steps

### You've Learned Testing! 🎉

You now understand:
- ✅ Testing philosophy and pyramid
- ✅ Frontend component testing
- ✅ Backend API testing
- ✅ Integration testing
- ✅ E2E testing
- ✅ Test coverage

### Continue Learning

1. **[DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)** - Debug failing tests
2. **[HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)** - Write tests for your features

---

**You're now ready to write reliable tests in Plane! 🚀**
