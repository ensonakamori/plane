# Database Architecture - PostgreSQL for Frontend Developers

**Master databases as a React developer!** This guide explains PostgreSQL, data modeling, and SQL using concepts you already understand from JavaScript and React.

**📅 Documentation Date:** November 19, 2025
**⏱️ Estimated Reading Time:** 1.5-2 hours
**📊 Difficulty Level:** Intermediate
**🎯 Target Audience:** Frontend developers learning databases for the first time

---

## Table of Contents

1. [Introduction - Why Databases?](#introduction---why-databases)
2. [Mental Model: localStorage vs Database](#mental-model-localstorage-vs-database)
3. [PostgreSQL Basics](#postgresql-basics)
4. [Tables and Columns](#tables-and-columns)
5. [Data Types](#data-types)
6. [Primary Keys and Foreign Keys](#primary-keys-and-foreign-keys)
7. [Relationships Explained](#relationships-explained)
8. [Indexes - Making Queries Fast](#indexes---making-queries-fast)
9. [Migrations - Version Control for Schema](#migrations---version-control-for-schema)
10. [Constraints and Validation](#constraints-and-validation)
11. [Query Optimization](#query-optimization)
12. [Real Schema Examples from Plane](#real-schema-examples-from-plane)
13. [Database Design Patterns](#database-design-patterns)
14. [Common Pitfalls](#common-pitfalls)
15. [Next Steps](#next-steps)

---

## Introduction - Why Databases?

### The Problem with Frontend State

As a React developer, you manage state:

```typescript
// React state (lost on page refresh)
const [issues, setIssues] = useState<Issue[]>([]);

// localStorage (limited to ~5MB, per-browser)
localStorage.setItem('issues', JSON.stringify(issues));

// In-memory (fast but lost on refresh)
const issuesCache = new Map<string, Issue>();
```

**Problems:**
- ❌ Lost on page refresh (unless localStorage)
- ❌ Not shared across users
- ❌ No querying capabilities
- ❌ No relationships between data
- ❌ No validation
- ❌ Limited storage

### The Database Solution

**Databases provide:**
- ✅ **Persistent storage** - Data survives server restarts
- ✅ **Shared state** - All users see the same data
- ✅ **Powerful queries** - Find, filter, sort, aggregate
- ✅ **Relationships** - Link related data together
- ✅ **Constraints** - Enforce data integrity
- ✅ **Transactions** - Atomic operations (all-or-nothing)
- ✅ **Scalability** - Handle millions of records

### 🧠 Mental Model

```
React State      = Temporary, per-user, in-browser
localStorage     = Persistent, per-user, in-browser
Database         = Persistent, all-users, on-server
```

Think of a database as **localStorage on steroids** - shared across all users, with querying superpowers!

---

## Mental Model: localStorage vs Database

### localStorage (What You Know)

```typescript
// Store data
localStorage.setItem('user', JSON.stringify({
  id: '123',
  name: 'Alice',
  email: 'alice@example.com'
}));

// Retrieve data
const user = JSON.parse(localStorage.getItem('user'));

// Search (INEFFICIENT - must parse all items)
const allUsers = Object.keys(localStorage)
  .filter(key => key.startsWith('user:'))
  .map(key => JSON.parse(localStorage.getItem(key)));

const alice = allUsers.find(u => u.name === 'Alice');
```

**Problems:**
- Must parse all items to search
- No relationships between items
- Limited to ~5MB
- Per-browser, not shared
- No validation

### PostgreSQL (What You'll Learn)

```sql
-- Store data (INSERT)
INSERT INTO users (id, name, email)
VALUES ('123', 'Alice', 'alice@example.com');

-- Retrieve data (SELECT)
SELECT * FROM users WHERE id = '123';

-- Search (EFFICIENT - indexed lookup)
SELECT * FROM users WHERE name = 'Alice';

-- Relationships (JOIN)
SELECT issues.*, users.name as assignee_name
FROM issues
JOIN users ON issues.assignee_id = users.id
WHERE users.name = 'Alice';
```

**Advantages:**
- Indexed searches (milliseconds for millions of rows)
- Relationships via foreign keys
- Unlimited storage (terabytes)
- Shared across all users
- Built-in validation (constraints)

---

## PostgreSQL Basics

### What is PostgreSQL?

**PostgreSQL (Postgres) is a relational database** - it stores data in tables (like Excel spreadsheets).

**Key Concepts:**

| Concept | JavaScript Equivalent |
|---------|----------------------|
| **Database** | Object with many keys |
| **Table** | Array of objects |
| **Row** | Single object in array |
| **Column** | Property of object |
| **Schema** | TypeScript interface definition |

### Database Structure

```typescript
// JavaScript (in-memory)
const database = {
  users: [
    { id: '1', name: 'Alice', email: 'alice@example.com' },
    { id: '2', name: 'Bob', email: 'bob@example.com' }
  ],
  issues: [
    { id: 'a', title: 'Bug fix', assignee_id: '1' },
    { id: 'b', title: 'Feature', assignee_id: '2' }
  ]
};
```

```sql
-- PostgreSQL (persistent storage)
Database: plane
├── Table: users
│   ├── Row: { id: '1', name: 'Alice', email: 'alice@example.com' }
│   └── Row: { id: '2', name: 'Bob', email: 'bob@example.com' }
└── Table: issues
    ├── Row: { id: 'a', title: 'Bug fix', assignee_id: '1' }
    └── Row: { id: 'b', title: 'Feature', assignee_id: '2' }
```

### SQL Basics

**SQL (Structured Query Language)** is like JavaScript for databases.

```sql
-- SELECT = Array.filter + Array.map
SELECT name, email FROM users WHERE name = 'Alice';

-- INSERT = Array.push
INSERT INTO users (name, email) VALUES ('Charlie', 'charlie@example.com');

-- UPDATE = Array.map + mutation
UPDATE users SET email = 'newemail@example.com' WHERE id = '1';

-- DELETE = Array.filter (inverse)
DELETE FROM users WHERE id = '1';
```

**JavaScript Equivalents:**

```typescript
// SELECT
const result = users
  .filter(u => u.name === 'Alice')
  .map(u => ({ name: u.name, email: u.email }));

// INSERT
users.push({ name: 'Charlie', email: 'charlie@example.com' });

// UPDATE
users = users.map(u =>
  u.id === '1' ? { ...u, email: 'newemail@example.com' } : u
);

// DELETE
users = users.filter(u => u.id !== '1');
```

---

## Tables and Columns

### What is a Table?

**A table is like a TypeScript interface + an array of instances.**

```typescript
// TypeScript interface (schema)
interface User {
  id: string;
  username: string;
  email: string;
  created_at: Date;
}

// Array of users (data)
const users: User[] = [
  { id: '1', username: 'alice', email: 'alice@example.com', created_at: new Date() },
  { id: '2', username: 'bob', email: 'bob@example.com', created_at: new Date() }
];
```

```sql
-- PostgreSQL table (schema + data combined)
CREATE TABLE users (
    id UUID PRIMARY KEY,
    username VARCHAR(128) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Data lives in the same table
INSERT INTO users VALUES ('1', 'alice', 'alice@example.com', NOW());
INSERT INTO users VALUES ('2', 'bob', 'bob@example.com', NOW());
```

### Plane's User Table

**Actual schema from Plane:**

```sql
CREATE TABLE db_user (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(128) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE,
    display_name VARCHAR(255) DEFAULT '',
    first_name VARCHAR(255),
    last_name VARCHAR(255),
    avatar TEXT,
    cover_image VARCHAR(800),

    -- Timestamps
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    date_joined TIMESTAMP NOT NULL DEFAULT NOW(),

    -- Booleans
    is_active BOOLEAN DEFAULT TRUE,
    is_email_verified BOOLEAN DEFAULT FALSE,
    is_superuser BOOLEAN DEFAULT FALSE,

    -- Activity
    last_active TIMESTAMP DEFAULT NOW(),
    last_login_time TIMESTAMP
);
```

**TypeScript Equivalent:**

```typescript
interface User {
  id: string;
  username: string;
  email: string | null;
  display_name: string;
  first_name: string;
  last_name: string;
  avatar: string;
  cover_image: string | null;
  created_at: Date;
  updated_at: Date;
  date_joined: Date;
  is_active: boolean;
  is_email_verified: boolean;
  is_superuser: boolean;
  last_active: Date | null;
  last_login_time: Date | null;
}
```

### Column Properties

```sql
CREATE TABLE issues (
    -- PRIMARY KEY = unique identifier (like id in JavaScript)
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    -- NOT NULL = required field (no null/undefined)
    title VARCHAR(255) NOT NULL,

    -- NULL allowed = optional field (can be null)
    description TEXT,

    -- DEFAULT = default value (like function parameter default)
    priority VARCHAR(30) DEFAULT 'none',

    -- UNIQUE = no duplicates allowed (like Set in JavaScript)
    sequence_id INTEGER UNIQUE NOT NULL,

    -- CHECK = validation constraint
    point INTEGER CHECK (point >= 0 AND point <= 12),

    -- Timestamp with auto-update
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

---

## Data Types

### PostgreSQL → TypeScript Mapping

| PostgreSQL | TypeScript | Example | Notes |
|-----------|------------|---------|-------|
| `VARCHAR(n)` | `string` | `'Hello'` | Max n characters |
| `TEXT` | `string` | `'Long text...'` | Unlimited length |
| `INTEGER` | `number` | `42` | Whole numbers |
| `BIGINT` | `number` | `999999999` | Large integers |
| `DECIMAL(10,2)` | `number` | `99.99` | Exact decimals |
| `BOOLEAN` | `boolean` | `true` | True/false |
| `TIMESTAMP` | `Date` | `2025-11-19T10:00:00Z` | Date + time |
| `DATE` | `Date` | `2025-11-19` | Date only |
| `UUID` | `string` | `'550e8400-...'` | Unique ID |
| `JSONB` | `object` | `{"key": "value"}` | JSON object |
| `ARRAY` | `Array<T>` | `{1,2,3}` | Array of values |

### String Types

```sql
-- VARCHAR(n) - Variable length, max n characters
username VARCHAR(128)  -- Max 128 chars

-- TEXT - Unlimited length
description TEXT       -- Any length

-- CHAR(n) - Fixed length (padded with spaces)
country_code CHAR(2)   -- Always 2 chars: 'US', 'UK'
```

**JavaScript:**

```typescript
username: string;      // VARCHAR(128)
description: string;   // TEXT
country_code: string;  // CHAR(2)
```

### Number Types

```sql
-- INTEGER - Whole numbers (-2B to +2B)
age INTEGER

-- BIGINT - Large whole numbers
user_count BIGINT

-- DECIMAL(precision, scale) - Exact decimals
price DECIMAL(10, 2)  -- 99999999.99 (10 digits, 2 after decimal)

-- REAL/FLOAT - Approximate decimals (like JavaScript number)
latitude FLOAT
```

**JavaScript:**

```typescript
age: number;           // INTEGER
user_count: number;    // BIGINT
price: number;         // DECIMAL
latitude: number;      // FLOAT
```

### Date/Time Types

```sql
-- TIMESTAMP - Date + time with timezone
created_at TIMESTAMP DEFAULT NOW()
-- Stores: 2025-11-19 10:30:45.123+00

-- DATE - Date only
birth_date DATE
-- Stores: 2025-11-19

-- TIME - Time only
meeting_time TIME
-- Stores: 14:30:00
```

**JavaScript:**

```typescript
created_at: Date;      // TIMESTAMP
birth_date: Date;      // DATE
meeting_time: string;  // TIME (no native Time type in JS)
```

### JSON Type ✅ CURRENT

**PostgreSQL can store JSON directly!**

```sql
CREATE TABLE issues (
    id UUID PRIMARY KEY,
    description JSONB  -- JSON Binary (fast + indexed)
);

INSERT INTO issues (id, description) VALUES (
    gen_random_uuid(),
    '{"ops": [{"insert": "Hello world"}]}'::jsonb
);

-- Query JSON fields
SELECT * FROM issues WHERE description->>'ops' IS NOT NULL;
SELECT * FROM issues WHERE description @> '{"ops": []}';
```

**JavaScript:**

```typescript
interface Issue {
  id: string;
  description: object;  // JSONB
}

// Django automatically parses JSON
issue.description  // Already a JavaScript object!
```

---

## Primary Keys and Foreign Keys

### Primary Keys

**Primary Key = Unique identifier for each row** (like `id` in JavaScript objects).

```typescript
// JavaScript - id uniquely identifies each user
const users = [
  { id: '1', name: 'Alice' },  // id is unique
  { id: '2', name: 'Bob' }     // id is unique
];
```

```sql
-- SQL - id is the primary key
CREATE TABLE users (
    id UUID PRIMARY KEY,  -- PRIMARY KEY = unique + not null
    name VARCHAR(255)
);
```

**Rules:**
- ✅ Must be UNIQUE
- ✅ Cannot be NULL
- ✅ One primary key per table
- ✅ Never changes (immutable)

**Common Types:**

```sql
-- UUID (Plane uses this) ✅ CURRENT
id UUID PRIMARY KEY DEFAULT gen_random_uuid()
-- Example: 550e8400-e29b-41d4-a716-446655440000

-- Auto-incrementing integer (older pattern)
id SERIAL PRIMARY KEY
-- Example: 1, 2, 3, 4, ...

-- String (rare)
id VARCHAR(50) PRIMARY KEY
-- Example: 'user-alice', 'user-bob'
```

### Foreign Keys

**Foreign Key = Reference to another table's primary key** (like object references in JavaScript).

```typescript
// JavaScript - reference by ID
const users = [
  { id: '1', name: 'Alice' }
];

const issues = [
  {
    id: 'a',
    title: 'Bug fix',
    assignee_id: '1'  // References users[0].id
  }
];

// To get assignee name, you must lookup:
const issue = issues[0];
const assignee = users.find(u => u.id === issue.assignee_id);
console.log(assignee.name);  // 'Alice'
```

```sql
-- SQL - foreign key enforces relationship
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE issues (
    id UUID PRIMARY KEY,
    title VARCHAR(255),
    assignee_id UUID REFERENCES users(id)  -- Foreign key!
);

-- Database ensures assignee_id always points to valid user
INSERT INTO issues (id, title, assignee_id)
VALUES ('a', 'Bug fix', '999');  -- ERROR: user '999' doesn't exist!

-- Join to get assignee name (no manual lookup needed)
SELECT issues.title, users.name as assignee_name
FROM issues
JOIN users ON issues.assignee_id = users.id;
```

### on_delete Behavior

**What happens when you delete a user who has issues assigned?**

```sql
-- CASCADE - Delete all related issues
assignee_id UUID REFERENCES users(id) ON DELETE CASCADE

-- SET NULL - Set assignee_id to NULL
assignee_id UUID REFERENCES users(id) ON DELETE SET NULL

-- RESTRICT - Prevent deletion if issues exist
assignee_id UUID REFERENCES users(id) ON DELETE RESTRICT

-- SET DEFAULT - Set to default value
assignee_id UUID REFERENCES users(id) ON DELETE SET DEFAULT DEFAULT '...'
```

**JavaScript Equivalent:**

```typescript
// CASCADE
users = users.filter(u => u.id !== '1');
issues = issues.filter(i => i.assignee_id !== '1');  // Delete related issues

// SET NULL
users = users.filter(u => u.id !== '1');
issues = issues.map(i => i.assignee_id === '1' ? { ...i, assignee_id: null } : i);

// RESTRICT
if (issues.some(i => i.assignee_id === '1')) {
  throw new Error("Cannot delete user with assigned issues");
}
```

---

## Relationships Explained

### One-to-Many (Most Common)

**"One project has many issues"**

```typescript
// JavaScript
interface Project {
  id: string;
  name: string;
}

interface Issue {
  id: string;
  title: string;
  project_id: string;  // Reference to project
}

// Data
const projects = [{ id: 'proj-1', name: 'Plane' }];
const issues = [
  { id: 'issue-1', title: 'Bug', project_id: 'proj-1' },
  { id: 'issue-2', title: 'Feature', project_id: 'proj-1' }
];
```

```sql
-- PostgreSQL
CREATE TABLE projects (
    id UUID PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE issues (
    id UUID PRIMARY KEY,
    title VARCHAR(255),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE
);

-- Query: Get all issues for a project
SELECT * FROM issues WHERE project_id = 'proj-1';

-- Query: Get project with all its issues (JOIN)
SELECT projects.name, issues.title
FROM projects
LEFT JOIN issues ON projects.id = issues.project_id
WHERE projects.id = 'proj-1';
```

**Django Model:**

```python
class Project(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4)
    name = models.CharField(max_length=255)

class Issue(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4)
    title = models.CharField(max_length=255)
    project = models.ForeignKey(
        Project,
        on_delete=models.CASCADE,
        related_name='issues'  # Reverse relationship
    )

# Usage
project = Project.objects.get(id='proj-1')
issues = project.issues.all()  # Get all issues for project
```

### Many-to-Many

**"Many issues can have many assignees, and many assignees can have many issues"**

```typescript
// JavaScript (manually maintain relationships)
interface Issue {
  id: string;
  title: string;
}

interface User {
  id: string;
  name: string;
}

// Junction table to track relationships
interface IssueAssignee {
  issue_id: string;
  user_id: string;
}

const issues = [{ id: 'issue-1', title: 'Bug' }];
const users = [
  { id: 'user-1', name: 'Alice' },
  { id: 'user-2', name: 'Bob' }
];

// Relationships
const issue_assignees = [
  { issue_id: 'issue-1', user_id: 'user-1' },
  { issue_id: 'issue-1', user_id: 'user-2' }
];

// To get assignees for an issue:
const issueId = 'issue-1';
const assigneeIds = issue_assignees
  .filter(ia => ia.issue_id === issueId)
  .map(ia => ia.user_id);
const assignees = users.filter(u => assigneeIds.includes(u.id));
```

```sql
-- PostgreSQL automatically creates junction table
CREATE TABLE issues (
    id UUID PRIMARY KEY,
    title VARCHAR(255)
);

CREATE TABLE users (
    id UUID PRIMARY KEY,
    name VARCHAR(255)
);

-- Junction table (stores relationships)
CREATE TABLE issue_assignees (
    id SERIAL PRIMARY KEY,
    issue_id UUID REFERENCES issues(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE(issue_id, user_id)  -- Prevent duplicates
);

-- Insert relationships
INSERT INTO issue_assignees (issue_id, user_id)
VALUES ('issue-1', 'user-1'), ('issue-1', 'user-2');

-- Query: Get all assignees for an issue
SELECT users.*
FROM users
JOIN issue_assignees ON users.id = issue_assignees.user_id
WHERE issue_assignees.issue_id = 'issue-1';

-- Query: Get all issues for a user
SELECT issues.*
FROM issues
JOIN issue_assignees ON issues.id = issue_assignees.issue_id
WHERE issue_assignees.user_id = 'user-1';
```

**Django Model (Django creates junction table automatically!):**

```python
class Issue(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4)
    title = models.CharField(max_length=255)
    assignees = models.ManyToManyField(
        User,
        related_name='assigned_issues'
    )  # Django creates issue_assignees table automatically!

# Usage (super easy)
issue = Issue.objects.get(id='issue-1')

# Add assignees
issue.assignees.add(user1, user2)

# Get all assignees
assignees = issue.assignees.all()

# Remove assignee
issue.assignees.remove(user1)

# Reverse: Get all issues assigned to user
user = User.objects.get(id='user-1')
assigned_issues = user.assigned_issues.all()
```

### Self-Referencing (Parent/Child)

**"An issue can have a parent issue (sub-issues)"**

```sql
CREATE TABLE issues (
    id UUID PRIMARY KEY,
    title VARCHAR(255),
    parent_id UUID REFERENCES issues(id) ON DELETE CASCADE
);

-- Insert parent issue
INSERT INTO issues (id, title, parent_id)
VALUES ('issue-1', 'Epic: Auth System', NULL);

-- Insert child issues
INSERT INTO issues (id, title, parent_id)
VALUES
    ('issue-2', 'Subtask: Login form', 'issue-1'),
    ('issue-3', 'Subtask: Signup form', 'issue-1');

-- Query: Get all sub-issues for parent
SELECT * FROM issues WHERE parent_id = 'issue-1';

-- Query: Get parent for sub-issue
SELECT parent.*
FROM issues as child
JOIN issues as parent ON child.parent_id = parent.id
WHERE child.id = 'issue-2';
```

**Django Model:**

```python
class Issue(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4)
    title = models.CharField(max_length=255)
    parent = models.ForeignKey(
        'self',  # Self-reference!
        on_delete=models.CASCADE,
        null=True,
        related_name='children'
    )

# Usage
parent_issue = Issue.objects.get(id='issue-1')
sub_issues = parent_issue.children.all()  # Get all sub-issues

sub_issue = Issue.objects.get(id='issue-2')
parent = sub_issue.parent  # Get parent issue
```

---

## Indexes - Making Queries Fast

### What Are Indexes?

**Indexes are like book indexes** - they help find data quickly without scanning every row.

```typescript
// JavaScript - no index (O(n) search)
const users = [
  { id: '1', name: 'Alice', email: 'alice@example.com' },
  { id: '2', name: 'Bob', email: 'bob@example.com' },
  // ... 1 million more users
];

// Must scan ALL users to find one email
const user = users.find(u => u.email === 'alice@example.com');
// Time: O(n) - slow for large arrays

// With Map index (O(1) lookup)
const usersByEmail = new Map(users.map(u => [u.email, u]));
const user = usersByEmail.get('alice@example.com');
// Time: O(1) - instant!
```

```sql
-- PostgreSQL - no index (sequential scan)
SELECT * FROM users WHERE email = 'alice@example.com';
-- Scans all 1,000,000 rows - SLOW (500ms)

-- Create index
CREATE INDEX idx_users_email ON users(email);

-- Now query uses index (fast lookup)
SELECT * FROM users WHERE email = 'alice@example.com';
-- Uses index - FAST (5ms)
```

### When to Add Indexes

**Add indexes to columns you frequently search/filter by:**

```sql
-- Frequently searched fields
CREATE INDEX idx_issues_project_id ON issues(project_id);
CREATE INDEX idx_issues_assignee_id ON issues(assignee_id);
CREATE INDEX idx_users_email ON users(email);

-- Frequently sorted fields
CREATE INDEX idx_issues_created_at ON issues(created_at);
CREATE INDEX idx_issues_priority ON issues(priority);

-- Composite indexes (multiple columns)
CREATE INDEX idx_issues_project_priority ON issues(project_id, priority);
```

**Django automatically creates indexes for:**
- ✅ Primary keys
- ✅ Foreign keys
- ✅ Fields with `unique=True`
- ✅ Fields with `db_index=True`

```python
class Issue(models.Model):
    id = models.UUIDField(primary_key=True)  # Auto-indexed
    project = models.ForeignKey(Project)     # Auto-indexed
    email = models.EmailField(unique=True)   # Auto-indexed
    title = models.CharField(max_length=255, db_index=True)  # Indexed!
```

### Trade-offs

**Indexes make reads faster but writes slower:**

```
Without Index:
- SELECT (read): Slow (500ms)
- INSERT (write): Fast (5ms)

With Index:
- SELECT (read): Fast (5ms)
- INSERT (write): Slower (10ms) - must update index
```

**Rule of thumb:** Index fields you search/filter by often.

---

## Migrations - Version Control for Schema

### What Are Migrations?

**Migrations are like Git for your database schema** - they track changes over time.

```typescript
// Git for code
git commit -m "Add user authentication"
git commit -m "Add issue comments"
git push

// Migrations for database
python manage.py makemigrations  # Creates migration file
python manage.py migrate         # Applies migration to database
```

### How Migrations Work

**Step 1: Create/modify a Django model**

```python
# Initial model
class Issue(models.Model):
    title = models.CharField(max_length=255)
    description = models.TextField()
```

**Step 2: Generate migration**

```bash
python manage.py makemigrations
```

**Django creates a migration file:**

```python
# plane/db/migrations/0001_initial.py
from django.db import migrations, models

class Migration(migrations.Migration):
    initial = True

    operations = [
        migrations.CreateModel(
            name='Issue',
            fields=[
                ('id', models.UUIDField(primary_key=True)),
                ('title', models.CharField(max_length=255)),
                ('description', models.TextField()),
            ],
        ),
    ]
```

**Step 3: Apply migration**

```bash
python manage.py migrate
```

**Django executes SQL:**

```sql
CREATE TABLE db_issue (
    id UUID PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL
);
```

### Adding a Field

**Step 1: Add field to model**

```python
class Issue(models.Model):
    title = models.CharField(max_length=255)
    description = models.TextField()
    priority = models.CharField(max_length=30, default='none')  # New field!
```

**Step 2: Generate migration**

```bash
python manage.py makemigrations
```

**Django creates migration:**

```python
# plane/db/migrations/0002_add_priority.py
class Migration(migrations.Migration):
    dependencies = [
        ('db', '0001_initial'),  # Depends on previous migration
    ]

    operations = [
        migrations.AddField(
            model_name='issue',
            name='priority',
            field=models.CharField(default='none', max_length=30),
        ),
    ]
```

**Step 3: Apply migration**

```bash
python manage.py migrate
```

**Django executes SQL:**

```sql
ALTER TABLE db_issue
ADD COLUMN priority VARCHAR(30) DEFAULT 'none' NOT NULL;

-- Updates existing rows with default value
UPDATE db_issue SET priority = 'none';
```

### Reverting Migrations

```bash
# Revert last migration
python manage.py migrate plane 0001

# Django executes:
# ALTER TABLE db_issue DROP COLUMN priority;

# Revert all migrations
python manage.py migrate plane zero
```

### 💡 Aha Moment

**Migrations are Git commits for your database schema!**

```
Code Changes (Git)          Database Changes (Migrations)
---------------------       -----------------------------
git add .                   python manage.py makemigrations
git commit -m "..."         (creates migration file)
git push                    python manage.py migrate
git revert <hash>           python manage.py migrate <number>
```

---

## Constraints and Validation

### Database-Level Validation

**Constraints enforce data integrity at the database level** (unlike JavaScript validation which can be bypassed).

```typescript
// JavaScript validation (can be bypassed)
function createUser(data) {
  if (!data.email) {
    throw new Error("Email required");
  }
  if (users.some(u => u.email === data.email)) {
    throw new Error("Email already exists");
  }
  users.push(data);
}

// But you can bypass it:
users.push({ email: null });  // Oops! Invalid data in array
```

```sql
-- PostgreSQL constraints (CANNOT be bypassed)
CREATE TABLE users (
    email VARCHAR(255) NOT NULL UNIQUE,  -- Database enforces this
    age INTEGER CHECK (age >= 0 AND age <= 150)
);

-- This will FAIL at database level:
INSERT INTO users (email, age) VALUES (NULL, -5);
-- ERROR: null value in column "email" violates not-null constraint
-- ERROR: new row violates check constraint "users_age_check"
```

### Common Constraints

```sql
-- NOT NULL - Field is required
email VARCHAR(255) NOT NULL

-- UNIQUE - No duplicates allowed
email VARCHAR(255) UNIQUE
username VARCHAR(128) UNIQUE

-- CHECK - Custom validation
age INTEGER CHECK (age >= 0 AND age <= 150)
priority VARCHAR(30) CHECK (priority IN ('low', 'medium', 'high'))

-- DEFAULT - Default value if not provided
is_active BOOLEAN DEFAULT TRUE
created_at TIMESTAMP DEFAULT NOW()

-- FOREIGN KEY - Must reference existing row
project_id UUID REFERENCES projects(id)

-- PRIMARY KEY - Unique identifier (combines NOT NULL + UNIQUE)
id UUID PRIMARY KEY
```

### Django Model Constraints

```python
from django.db import models
from django.core.validators import MinValueValidator, MaxValueValidator

class Issue(models.Model):
    # NOT NULL (required field)
    title = models.CharField(max_length=255)

    # NULL allowed (optional field)
    description = models.TextField(null=True, blank=True)

    # UNIQUE constraint
    sequence_id = models.IntegerField(unique=True)

    # CHECK constraint (validators)
    point = models.IntegerField(
        validators=[MinValueValidator(0), MaxValueValidator(12)],
        null=True
    )

    # FOREIGN KEY with on_delete behavior
    project = models.ForeignKey(
        Project,
        on_delete=models.CASCADE  # Delete issue when project deleted
    )

    # DEFAULT value
    priority = models.CharField(max_length=30, default='none')
    created_at = models.DateTimeField(auto_now_add=True)

    # Multi-field unique constraint
    class Meta:
        unique_together = [['project', 'sequence_id']]
        # Each project can have issue #1, #2, etc.
        # But no duplicate sequence_ids within same project
```

### Check Constraints

```python
from django.db import models
from django.db.models import Q, CheckConstraint

class Issue(models.Model):
    start_date = models.DateField(null=True)
    target_date = models.DateField(null=True)

    class Meta:
        constraints = [
            # Ensure start_date <= target_date
            CheckConstraint(
                check=Q(start_date__lte=models.F('target_date')),
                name='start_before_target'
            ),
        ]
```

---

## Query Optimization

### The N+1 Query Problem

**Problem: Making one query per row (SLOW)**

```python
# BAD - N+1 queries
issues = Issue.objects.all()  # 1 query
for issue in issues:
    print(issue.project.name)  # N queries (one per issue!)

# If you have 100 issues, this makes 101 queries!
```

**Solution 1: select_related (for ForeignKey)**

```python
# GOOD - 1 query with JOIN
issues = Issue.objects.select_related('project').all()
for issue in issues:
    print(issue.project.name)  # No extra query!

# SQL:
# SELECT * FROM issues
# JOIN projects ON issues.project_id = projects.id
```

**Solution 2: prefetch_related (for ManyToMany)**

```python
# GOOD - 2 queries total
issues = Issue.objects.prefetch_related('assignees').all()
for issue in issues:
    for assignee in issue.assignees.all():  # No extra queries!
        print(assignee.name)

# SQL:
# SELECT * FROM issues
# SELECT * FROM users WHERE id IN (...)
```

### Only Select What You Need

```python
# BAD - Selects all columns
issues = Issue.objects.all()
# SELECT * FROM issues (includes description, description_html, etc.)

# GOOD - Only select specific columns
issues = Issue.objects.only('id', 'title', 'priority')
# SELECT id, title, priority FROM issues

# GOOD - Exclude large columns
issues = Issue.objects.defer('description_html', 'description_binary')
# SELECT id, title, ... FROM issues (excludes heavy columns)
```

### Use Aggregation in Database

```python
# BAD - Load all data, aggregate in Python
issues = Issue.objects.all()
high_priority_count = len([i for i in issues if i.priority == 'high'])

# GOOD - Aggregate in database
from django.db.models import Count
high_priority_count = Issue.objects.filter(priority='high').count()
# SELECT COUNT(*) FROM issues WHERE priority = 'high'
```

### Explain Query Plans

```python
# See what SQL Django generates
issues = Issue.objects.filter(priority='high').select_related('project')
print(issues.query)

# PostgreSQL explain plan (shows indexes used)
# SELECT * FROM issues WHERE priority = 'high'
# EXPLAIN: Seq Scan on issues (cost=0.00..100.00 rows=50)
```

---

## Real Schema Examples from Plane

### Issue Table

```sql
CREATE TABLE db_issue (
    -- Primary key
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    -- Foreign keys (relationships)
    workspace_id UUID NOT NULL REFERENCES db_workspace(id) ON DELETE CASCADE,
    project_id UUID NOT NULL REFERENCES db_project(id) ON DELETE CASCADE,
    parent_id UUID REFERENCES db_issue(id) ON DELETE CASCADE,  -- Self-reference
    state_id UUID REFERENCES db_state(id) ON DELETE CASCADE,
    created_by_id UUID REFERENCES db_user(id) ON DELETE SET NULL,
    updated_by_id UUID REFERENCES db_user(id) ON DELETE SET NULL,

    -- Basic fields
    name VARCHAR(255) NOT NULL,
    description JSONB DEFAULT '{}'::jsonb,
    description_html TEXT DEFAULT '<p></p>',
    sequence_id INTEGER NOT NULL,
    priority VARCHAR(30) DEFAULT 'none',

    -- Dates
    start_date DATE,
    target_date DATE,

    -- Metadata
    sort_order DECIMAL DEFAULT 65535,
    is_draft BOOLEAN DEFAULT FALSE,

    -- Timestamps
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    archived_at TIMESTAMP,
    deleted_at TIMESTAMP,

    -- Indexes
    CONSTRAINT unique_project_sequence UNIQUE (project_id, sequence_id)
);

-- Additional indexes
CREATE INDEX idx_issue_project ON db_issue(project_id);
CREATE INDEX idx_issue_workspace ON db_issue(workspace_id);
CREATE INDEX idx_issue_state ON db_issue(state_id);
CREATE INDEX idx_issue_parent ON db_issue(parent_id);
CREATE INDEX idx_issue_created_at ON db_issue(created_at);
```

### Many-to-Many: Issue Assignees

```sql
-- Junction table (automatically created by Django)
CREATE TABLE issue_assignees (
    id SERIAL PRIMARY KEY,
    issue_id UUID REFERENCES db_issue(id) ON DELETE CASCADE,
    user_id UUID REFERENCES db_user(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(issue_id, user_id)  -- Prevent duplicate assignments
);

CREATE INDEX idx_issue_assignees_issue ON issue_assignees(issue_id);
CREATE INDEX idx_issue_assignees_user ON issue_assignees(user_id);
```

### Project Hierarchy

```sql
-- Workspace (top level)
CREATE TABLE db_workspace (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Project (belongs to workspace)
CREATE TABLE db_project (
    id UUID PRIMARY KEY,
    workspace_id UUID REFERENCES db_workspace(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    identifier VARCHAR(10) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(workspace_id, identifier)
);

-- Issue (belongs to project, belongs to workspace)
CREATE TABLE db_issue (
    id UUID PRIMARY KEY,
    workspace_id UUID REFERENCES db_workspace(id) ON DELETE CASCADE,
    project_id UUID REFERENCES db_project(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL
);

-- Relationship diagram:
-- Workspace (1) → Projects (many) → Issues (many)
```

---

## Database Design Patterns

### Pattern 1: Soft Delete

**Don't actually delete - mark as deleted instead**

```sql
CREATE TABLE issues (
    id UUID PRIMARY KEY,
    title VARCHAR(255),
    deleted_at TIMESTAMP  -- NULL = active, NOT NULL = deleted
);

-- "Delete" an issue (soft delete)
UPDATE issues SET deleted_at = NOW() WHERE id = '123';

-- Query only active issues
SELECT * FROM issues WHERE deleted_at IS NULL;

-- Actually delete (hard delete)
DELETE FROM issues WHERE id = '123';
```

**Why?**
- ✅ Can recover deleted data
- ✅ Maintain audit trail
- ✅ Preserve relationships
- ❌ Database grows larger

### Pattern 2: Audit Columns

**Track who created/updated and when**

```sql
CREATE TABLE issues (
    id UUID PRIMARY KEY,
    title VARCHAR(255),

    -- Audit columns
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    created_by_id UUID REFERENCES users(id),
    updated_by_id UUID REFERENCES users(id)
);

-- Django automatically sets these!
```

### Pattern 3: Denormalization for Performance

**Store computed/aggregated data to avoid expensive queries**

```sql
-- Normalized (recalculate every time)
CREATE TABLE projects (
    id UUID PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE issues (
    id UUID PRIMARY KEY,
    project_id UUID REFERENCES projects(id)
);

-- Query issue count (slow on large datasets)
SELECT projects.*, COUNT(issues.id) as issue_count
FROM projects
LEFT JOIN issues ON projects.id = issues.project_id
GROUP BY projects.id;

-- Denormalized (store count)
CREATE TABLE projects (
    id UUID PRIMARY KEY,
    name VARCHAR(255),
    issue_count INTEGER DEFAULT 0  -- Store count here!
);

-- Update count when issue created/deleted (via trigger or Django signal)
-- Query is now instant:
SELECT * FROM projects;  -- issue_count is already there!
```

### Pattern 4: Polymorphic Relationships

**One table references multiple different tables**

```sql
-- Comments can be on issues, pages, or other models
CREATE TABLE comments (
    id UUID PRIMARY KEY,
    content TEXT,
    entity_type VARCHAR(50),  -- 'issue', 'page', 'doc'
    entity_id UUID            -- ID of the entity
);

-- Get comments for an issue
SELECT * FROM comments
WHERE entity_type = 'issue' AND entity_id = 'issue-123';
```

---

## Common Pitfalls

### 1. Not Using Indexes

```sql
-- SLOW - no index on email
SELECT * FROM users WHERE email = 'alice@example.com';
-- Scans all rows

-- FAST - with index
CREATE INDEX idx_users_email ON users(email);
SELECT * FROM users WHERE email = 'alice@example.com';
-- Uses index
```

### 2. N+1 Queries

```python
# BAD - 101 queries for 100 issues
issues = Issue.objects.all()
for issue in issues:
    print(issue.project.name)  # Query per issue

# GOOD - 1 query
issues = Issue.objects.select_related('project').all()
for issue in issues:
    print(issue.project.name)  # No extra query
```

### 3. Selecting All Columns

```python
# BAD - selects huge description field
issues = Issue.objects.all()

# GOOD - only select what you need
issues = Issue.objects.only('id', 'title', 'priority')
```

### 4. Not Using Transactions

```python
# BAD - partial updates on error
issue = Issue.objects.create(...)
issue.assignees.add(user1)  # Succeeds
issue.assignees.add(user2)  # Fails - first assignment still happened!

# GOOD - all-or-nothing
from django.db import transaction

with transaction.atomic():
    issue = Issue.objects.create(...)
    issue.assignees.add(user1, user2)
    # If either fails, both are rolled back
```

### 5. Exposing Database IDs

```sql
-- BAD - predictable IDs
id SERIAL PRIMARY KEY  -- 1, 2, 3, 4...
-- Users can guess: /api/issues/1, /api/issues/2, /api/issues/3

-- GOOD - UUIDs (unpredictable)
id UUID PRIMARY KEY DEFAULT gen_random_uuid()
-- /api/issues/550e8400-e29b-41d4-a716-446655440000
```

---

## Next Steps

### You've Learned Database Fundamentals! 🎉

You now understand:
- ✅ Tables, rows, columns (like TypeScript interfaces + arrays)
- ✅ Primary keys and foreign keys (relationships)
- ✅ One-to-Many and Many-to-Many relationships
- ✅ Indexes (making queries fast)
- ✅ Migrations (version control for schema)
- ✅ Constraints (data validation)
- ✅ Query optimization (avoiding N+1 queries)

### Continue Learning

1. **[BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)** - Django models and querysets
2. **[INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md)** - React → Django → Database flow
3. **[HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)** - Add fields, create models, write migrations
4. **[DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)** - Debug database queries

### Practice Exercises

1. **Design a schema** - Create tables for a blog (posts, comments, authors)
2. **Write migrations** - Add/remove fields from existing models
3. **Optimize queries** - Find N+1 queries and fix them with select_related
4. **Use Django shell** - Practice querying data
5. **Read Plane's schema** - Study `/home/user/plane/apps/api/plane/db/models/`

### Resources

- **PostgreSQL Docs:** https://www.postgresql.org/docs/
- **Django ORM:** https://docs.djangoproject.com/en/4.2/topics/db/
- **Plane Models:** `/home/user/plane/apps/api/plane/db/models/`

---

**You're now ready to understand Plane's database architecture! 🚀**

**Next:** [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md) - Learn how React, Django, and PostgreSQL work together.
