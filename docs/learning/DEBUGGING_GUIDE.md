# Debugging Guide - Tools & Techniques for Frontend & Backend

**Debug like a pro!** This guide covers debugging tools, techniques, and common issues for both React frontend and Django backend in Plane.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 1 hour
**📊 Difficulty Level:** Intermediate to Advanced
**🎯 Target Audience:** Developers debugging issues

---

## Table of Contents

1. [Debugging Mindset](#debugging-mindset)
2. [Frontend Debugging](#frontend-debugging)
3. [Backend Debugging](#backend-debugging)
4. [Database Debugging](#database-debugging)
5. [Network Debugging](#network-debugging)
6. [Performance Debugging](#performance-debugging)
7. [Common Issues & Solutions](#common-issues--solutions)

---

## Debugging Mindset

### The Scientific Method

1. **Observe** - What's the symptom?
2. **Hypothesize** - What could cause this?
3. **Test** - How can I verify my hypothesis?
4. **Verify** - Did my fix work?
5. **Document** - Record the solution

### Before You Debug

- ✅ Can you reproduce the issue?
- ✅ What changed recently?
- ✅ Does it happen in dev/staging/production?
- ✅ Does it happen for all users?
- ✅ What do the error logs say?

### Debugging Techniques

1. **Read the error message** (seriously, read it carefully!)
2. **Use console.log / print()** (simple but effective)
3. **Use debugger / breakpoints** (step through code)
4. **Binary search** (comment out half the code)
5. **Rubber duck debugging** (explain problem out loud)
6. **Ask for help** (after trying the above)

---

## Frontend Debugging

### Browser DevTools ✅ CURRENT

**Chrome DevTools (F12 or Cmd+Option+I)**

**1. Console Tab**

```typescript
// Log values
console.log('Issue:', issue);
console.log('User:', user);

// Log objects in table format
console.table(issues);

// Group logs
console.group('Issue Creation');
console.log('Form data:', formData);
console.log('Response:', response);
console.groupEnd();

// Measure performance
console.time('fetch-issues');
await fetchIssues();
console.timeEnd('fetch-issues');

// Stack trace
console.trace();

// Conditional logging
if (process.env.NODE_ENV === 'development') {
  console.debug('Debug info:', data);
}
```

**2. Sources Tab (Debugger)**

```typescript
// Add debugger statement
function handleSubmit(data) {
  debugger;  // Execution pauses here
  const issue = await createIssue(data);
  return issue;
}

// Or set breakpoints:
// - Click line number in Sources tab
// - Execution pauses when line is reached
// - Inspect variables in Scope panel
// - Step through code with F10 (step over), F11 (step into)
```

**3. Network Tab**

- See all HTTP requests
- Check request/response headers
- View request payload and response body
- Check status codes and timing

**4. React DevTools**

```bash
# Install React DevTools extension
# Chrome: https://chrome.google.com/webstore (search "React Developer Tools")
```

**Features:**
- Inspect component tree
- View props and state
- See hooks values
- Track renders
- Edit props/state in real-time

**Usage:**

```typescript
// In Components tab:
// 1. Select component
// 2. View props, state, hooks in sidebar
// 3. Edit values to test different scenarios

// In Profiler tab:
// 1. Click record
// 2. Interact with app
// 3. Stop recording
// 4. See which components rendered and why
```

### MobX DevTools

```bash
# Install MobX DevTools extension
```

**Features:**
- View store state
- Track state changes
- See which components observe which stores
- Time-travel debugging

### Common Frontend Issues

**Issue: Component not re-rendering**

```typescript
// ❌ BAD - Mutating state directly
issue.priority = 'high';  // MobX won't detect this

// ✅ GOOD - Use action
@action
updatePriority(issueId, priority) {
  const issue = this.issues.get(issueId);
  issue.priority = priority;  // Or use runInAction()
}
```

**Issue: Infinite render loop**

```typescript
// ❌ BAD - useEffect without dependencies
useEffect(() => {
  fetchIssues();  // Runs on every render!
});

// ✅ GOOD - Add dependencies
useEffect(() => {
  fetchIssues();
}, [projectId]);  // Only runs when projectId changes
```

**Issue: State not updating**

```typescript
// ❌ BAD - Setting state with same reference
const handleUpdate = () => {
  issues.push(newIssue);
  setIssues(issues);  // Same reference!
};

// ✅ GOOD - Create new array
const handleUpdate = () => {
  setIssues([...issues, newIssue]);
};
```

**Issue: "Cannot read property of undefined"**

```typescript
// ❌ BAD - No null checking
const userName = user.name;  // Error if user is null

// ✅ GOOD - Optional chaining
const userName = user?.name ?? 'Unknown';

// ✅ GOOD - Early return
if (!user) return <div>Loading...</div>;
return <div>{user.name}</div>;
```

### React Error Boundaries

```tsx
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo);
    // Send to error tracking service (Sentry, etc.)
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
  <App />
</ErrorBoundary>
```

---

## Backend Debugging

### Django Debug Toolbar ✅ CURRENT

**Install and configure:**

```python
# settings.py
INSTALLED_APPS = [
    'debug_toolbar',
    # ...
]

MIDDLEWARE = [
    'debug_toolbar.middleware.DebugToolbarMiddleware',
    # ...
]

INTERNAL_IPS = ['127.0.0.1']
```

**Features:**
- SQL queries (count, time, EXPLAIN)
- Cache hits/misses
- Template rendering time
- Signals fired
- Request/response headers

**Usage:**
- Open app in browser
- Toolbar appears on right side
- Click panels to see details

### Python Debugger (pdb)

```python
# Add breakpoint
def my_view(request):
    import pdb; pdb.set_trace()  # Execution pauses here

    issues = Issue.objects.all()
    return Response({'issues': issues})

# Commands:
# n - next line
# s - step into function
# c - continue execution
# l - list code
# p variable - print variable
# pp variable - pretty print
# q - quit
```

### Python Debugger (ipdb) - Enhanced

```bash
pip install ipdb
```

```python
# More features than pdb
import ipdb; ipdb.set_trace()

# Features:
# - Syntax highlighting
# - Tab completion
# - Better introspection
```

### Logging

```python
# Configure logging
import logging
logger = logging.getLogger(__name__)

# In views
def create_issue(request):
    logger.info(f'Creating issue with data: {request.data}')

    try:
        issue = Issue.objects.create(**request.data)
        logger.info(f'Created issue: {issue.id}')
        return Response({'id': issue.id}, status=201)

    except Exception as e:
        logger.error(f'Error creating issue: {str(e)}', exc_info=True)
        return Response({'error': str(e)}, status=500)

# Log levels:
# - DEBUG: Detailed info for debugging
# - INFO: General info
# - WARNING: Warning messages
# - ERROR: Error messages
# - CRITICAL: Critical errors
```

### Django Shell

```bash
# Start shell
python manage.py shell

# Or use shell_plus (more features)
python manage.py shell_plus
```

```python
# Test queries
>>> from plane.db.models import Issue, User
>>> issues = Issue.objects.all()
>>> issues.count()
42

>>> issue = issues.first()
>>> issue.name
'Fix bug'

>>> issue.assignees.all()
[<User: alice>, <User: bob>]

# Test serializers
>>> from plane.app.serializers import IssueSerializer
>>> serializer = IssueSerializer(issue)
>>> serializer.data
{'id': '123', 'name': 'Fix bug', ...}

# Test views
>>> from plane.app.views import IssueViewSet
>>> viewset = IssueViewSet()
>>> queryset = viewset.get_queryset()
>>> queryset.count()
```

### Common Backend Issues

**Issue: N+1 Query Problem**

```python
# ❌ BAD - N+1 queries
issues = Issue.objects.all()  # 1 query
for issue in issues:
    print(issue.project.name)  # N queries

# ✅ GOOD - Use select_related
issues = Issue.objects.select_related('project').all()  # 1 query with JOIN
for issue in issues:
    print(issue.project.name)  # No extra query

# ✅ GOOD - Use prefetch_related for many-to-many
issues = Issue.objects.prefetch_related('assignees').all()
for issue in issues:
    for assignee in issue.assignees.all():  # No extra queries
        print(assignee.name)
```

**Issue: 500 Internal Server Error**

```python
# Check Django logs
# Terminal running `python manage.py runserver`
# Look for stack trace

# Add logging
import logging
logger = logging.getLogger(__name__)

def my_view(request):
    try:
        # Your code
        pass
    except Exception as e:
        logger.error(f'Error: {str(e)}', exc_info=True)
        raise
```

**Issue: Permission Denied (403)**

```python
# Check permissions
from plane.app.permissions import allow_permission, ROLE

class IssueViewSet(viewsets.ModelViewSet):
    @allow_permission([ROLE.ADMIN, ROLE.MEMBER])
    def create(self, request):
        # Check if user is project member
        member = ProjectMember.objects.filter(
            project_id=request.data.get('project_id'),
            member=request.user
        ).first()

        logger.debug(f'User: {request.user}, Member: {member}')

        if not member:
            logger.warning(f'User {request.user} not a project member')
            return Response({'error': 'Not a member'}, status=403)
```

**Issue: Serializer Validation Error**

```python
# Check serializer validation
serializer = IssueSerializer(data=request.data)

if not serializer.is_valid():
    logger.error(f'Validation errors: {serializer.errors}')
    return Response(serializer.errors, status=400)

# Add custom validation
class IssueSerializer(serializers.ModelSerializer):
    def validate_name(self, value):
        logger.debug(f'Validating name: {value}')
        if len(value) < 3:
            raise serializers.ValidationError('Name too short')
        return value
```

---

## Database Debugging

### Query Debugging

```python
# See SQL query
queryset = Issue.objects.filter(priority='high')
print(queryset.query)

# Output:
# SELECT * FROM db_issue WHERE priority = 'high'

# Use EXPLAIN
from django.db import connection
with connection.cursor() as cursor:
    cursor.execute('EXPLAIN SELECT * FROM db_issue WHERE priority = %s', ['high'])
    print(cursor.fetchall())
```

### Django Debug Toolbar SQL Panel

**Features:**
- See all queries
- Execution time
- Duplicate queries
- EXPLAIN plans
- Similar queries

**Usage:**
1. Load page
2. Click "SQL" panel
3. See queries executed
4. Click query to see EXPLAIN
5. Optimize slow queries

### PostgreSQL Logs

```bash
# Enable query logging
# postgresql.conf
log_statement = 'all'
log_duration = on

# Tail logs
tail -f /var/log/postgresql/postgresql.log
```

### Common Database Issues

**Issue: Slow queries**

```python
# Add index
class Issue(models.Model):
    priority = models.CharField(max_length=30, db_index=True)  # Add index

    class Meta:
        indexes = [
            models.Index(fields=['project', 'priority']),  # Composite index
        ]

# Create migration
python manage.py makemigrations
python manage.py migrate
```

**Issue: Deadlocks**

```python
# Use transactions
from django.db import transaction

with transaction.atomic():
    issue = Issue.objects.select_for_update().get(id=issue_id)
    issue.status = 'completed'
    issue.save()
```

---

## Network Debugging

### Browser Network Tab

**1. Check request details**
- URL
- Method (GET, POST, etc.)
- Status code
- Request headers (Authorization token?)
- Request payload
- Response body

**2. Filter by type**
- XHR/Fetch (API requests)
- JS, CSS, Images
- WS (WebSockets)

**3. Timing**
- See how long requests take
- Identify slow requests

### Common Network Issues

**Issue: 401 Unauthorized**

```typescript
// Check token is included
const token = localStorage.getItem('access_token');
console.log('Token:', token);  // Is it null?

// Check token is valid
// Decode JWT at jwt.io
// Is exp (expiration) in the past?

// Check Authorization header
fetch('/api/issues/', {
  headers: {
    'Authorization': `Bearer ${token}`  // Correct format?
  }
});
```

**Issue: CORS errors**

```python
# Backend: Configure CORS
INSTALLED_APPS = [
    'corsheaders',
    # ...
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    # ...
]

CORS_ALLOWED_ORIGINS = [
    'http://localhost:3000',  # Frontend URL
]
```

**Issue: 500 errors from API**

```typescript
// Check response body for error message
try {
  const response = await fetch('/api/issues/', { method: 'POST', body: data });

  if (!response.ok) {
    const errorData = await response.json();
    console.error('API error:', errorData);
    // Check errorData.error, errorData.details, etc.
  }
} catch (error) {
  console.error('Network error:', error);
}
```

---

## Performance Debugging

### React Performance

**1. React DevTools Profiler**

- Click "Profiler" tab
- Click record (circle)
- Interact with app
- Stop recording
- See which components rendered
- See render duration
- Identify slow components

**2. Why did component render?**

```typescript
// Use React DevTools
// Select component
// See "Rendered by" in sidebar

// Or use custom hook
function useWhyDidYouUpdate(name, props) {
  const previousProps = useRef();

  useEffect(() => {
    if (previousProps.current) {
      const changedProps = Object.entries(props).filter(
        ([key, val]) => previousProps.current[key] !== val
      );

      if (changedProps.length > 0) {
        console.log(`[${name}] Changed props:`, Object.fromEntries(changedProps));
      }
    }

    previousProps.current = props;
  });
}

// Usage
const MyComponent = (props) => {
  useWhyDidYouUpdate('MyComponent', props);
  // ...
};
```

**3. Optimization techniques**

```typescript
// Memoize expensive computations
const sortedIssues = useMemo(() => {
  return issues.sort((a, b) => a.priority.localeCompare(b.priority));
}, [issues]);

// Memoize callbacks
const handleClick = useCallback(() => {
  console.log('Clicked');
}, []);

// Memoize components
const IssueCard = React.memo(({ issue }) => {
  return <div>{issue.name}</div>;
});
```

### Backend Performance

**1. Django Debug Toolbar**

- Check "Time" panel
- See which code took longest
- Check SQL panel for slow queries

**2. Profiling with cProfile**

```python
# Profile a function
import cProfile
import pstats

def profile_view(request):
    profiler = cProfile.Profile()
    profiler.enable()

    # Your code here
    result = my_slow_function()

    profiler.disable()
    stats = pstats.Stats(profiler)
    stats.sort_stats('cumulative')
    stats.print_stats(10)  # Top 10 slowest

    return Response(result)
```

**3. Optimize queries**

```python
# Use select_related, prefetch_related
# Add indexes
# Use .only() to fetch fewer columns
# Use .values() for dictionary results (faster than model instances)

# Example
issues = Issue.objects.only('id', 'name').values('id', 'name')
```

---

## Common Issues & Solutions

### Frontend Issues

**1. "Module not found"**

```bash
# Check import path
import { Button } from '@/components/ui/button';  # Correct?

# Check file exists
ls apps/web/components/ui/button.tsx

# Clear cache and reinstall
rm -rf node_modules
pnpm install
```

**2. "Maximum update depth exceeded"**

```typescript
// ❌ BAD - Updates state in render
function MyComponent() {
  const [count, setCount] = useState(0);
  setCount(count + 1);  // Infinite loop!
  return <div>{count}</div>;
}

// ✅ GOOD - Update in event handler or useEffect
function MyComponent() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1);
  };

  return <button onClick={handleClick}>{count}</button>;
}
```

### Backend Issues

**1. "No such table"**

```bash
# Run migrations
python manage.py migrate

# If that doesn't work, check migration files exist
ls plane/db/migrations/

# Create migration if missing
python manage.py makemigrations
```

**2. "Object does not exist"**

```python
# ❌ BAD - Crashes if not found
issue = Issue.objects.get(id=issue_id)

# ✅ GOOD - Handle not found
try:
    issue = Issue.objects.get(id=issue_id)
except Issue.DoesNotExist:
    return Response({'error': 'Issue not found'}, status=404)

# ✅ GOOD - Return None if not found
issue = Issue.objects.filter(id=issue_id).first()
if not issue:
    return Response({'error': 'Issue not found'}, status=404)
```

---

## Debugging Checklist

**When stuck:**

- [ ] Read error message carefully
- [ ] Check browser console for errors
- [ ] Check Network tab for failed requests
- [ ] Check Django logs for backend errors
- [ ] Add console.log / logger.debug statements
- [ ] Use debugger / breakpoints
- [ ] Check recent changes (git diff)
- [ ] Try in incognito mode (clears cache)
- [ ] Restart dev servers
- [ ] Clear cache (browser, SWR, localStorage)
- [ ] Ask for help (with details!)

---

## Next Steps

### You've Learned Debugging! 🎉

You now understand:
- ✅ Browser DevTools (Console, Network, Debugger)
- ✅ React DevTools and MobX DevTools
- ✅ Django Debug Toolbar and pdb
- ✅ Database query debugging
- ✅ Network debugging
- ✅ Performance profiling
- ✅ Common issues and solutions

### Continue Learning

1. **[TESTING_GUIDE.md](./TESTING_GUIDE.md)** - Write tests to catch bugs early
2. **[HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)** - Apply debugging techniques

---

**You're now ready to debug issues efficiently in Plane! 🚀**
