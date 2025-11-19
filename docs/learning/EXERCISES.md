# Exercises

> **Last Updated:** November 2025
> **Status:** ✅ CURRENT - Hands-on coding challenges for Plane

Progressive, hands-on coding challenges to build your skills working with Plane's codebase. Each exercise includes objectives, hints, solutions, and self-grading checklists.

---

## Table of Contents

1. [How to Use This Guide](#how-to-use-this-guide)
2. [Beginner Exercises](#beginner-exercises)
3. [Intermediate Exercises](#intermediate-exercises)
4. [Advanced Exercises](#advanced-exercises)
5. [Full-Stack Challenges](#full-stack-challenges)
6. [Performance Optimization](#performance-optimization)
7. [Security Challenges](#security-challenges)
8. [Solutions & Hints](#solutions--hints)

---

## How to Use This Guide

### Exercise Format

Each exercise follows this structure:

```
## Exercise N: Title
**Level**: Beginner/Intermediate/Advanced
**Type**: Frontend/Backend/Full-Stack
**Time**: Estimated time to complete
**Skills**: Technologies/concepts covered

### Objective
What you'll build or implement

### Requirements
Specific requirements and acceptance criteria

### Hints
Guidance to get you started

### Self-Grading Checklist
- [ ] Requirement 1
- [ ] Requirement 2

### Solution
Where to find the solution
```

### Setup Instructions

Before starting exercises:

```bash
# 1. Set up development environment
cd /home/user/plane
make dev_setup

# 2. Start backend
cd apps/api
python manage.py runserver

# 3. Start frontend (in another terminal)
cd apps/web
npm run dev

# 4. Create a test workspace and project
# Use the UI or create via Django shell
```

### Testing Your Solutions

```bash
# Frontend tests
cd apps/web
npm run test

# Backend tests
cd apps/api
python manage.py test

# Linting
npm run lint
python -m black apps/api --check
```

---

## Beginner Exercises

### Exercise 1: Add a Color Picker Component

**Level**: ⭐ Beginner
**Type**: Frontend
**Time**: 30-45 minutes
**Skills**: React components, TypeScript, Props

#### Objective

Create a reusable color picker component that can be used throughout the Plane UI for selecting colors (e.g., for labels, states).

#### Requirements

1. Create a `ColorPicker` component that displays a grid of preset colors
2. Allow selecting a color and returning it to parent component
3. Support custom color input via hex code
4. Show the currently selected color with a visual indicator
5. Make it accessible (keyboard navigation, ARIA labels)

#### Hints

- Look at existing components in `apps/web/components/ui/` for patterns
- Use React's `useState` for managing selected color
- Consider using the existing design system colors
- Check `apps/web/components/labels/` for inspiration

#### Starter Code

```typescript
// apps/web/components/ui/color-picker.tsx

interface ColorPickerProps {
  value: string;
  onChange: (color: string) => void;
  colors?: string[];
}

export const ColorPicker = ({ value, onChange, colors }: ColorPickerProps) => {
  // TODO: Implement color picker
  return <div>Color Picker</div>;
};
```

#### Self-Grading Checklist

- [ ] Component renders a grid of colors
- [ ] Clicking a color calls `onChange` with the color value
- [ ] Currently selected color has a visual indicator
- [ ] Custom hex input works correctly
- [ ] Component is TypeScript type-safe
- [ ] Keyboard navigation works (Tab, Enter)
- [ ] ARIA labels are present for accessibility

#### Where to Look

- `apps/web/components/labels/create-label-modal.tsx` - Label color selection
- `apps/web/components/ui/` - Other UI components
- `apps/web/lib/constants.ts` - Design system colors

---

### Exercise 2: Add Issue Priority Filter

**Level**: ⭐ Beginner
**Type**: Frontend
**Time**: 45-60 minutes
**Skills**: React, State management, Filtering

#### Objective

Add a priority filter to the issue list view that filters issues by priority level.

#### Requirements

1. Add a priority filter dropdown above the issue list
2. Support multi-select (multiple priorities at once)
3. Update URL query params when filter changes
4. Filter should persist on page reload
5. Show count of issues for each priority

#### Hints

- Check `apps/web/components/issues/issue-filter.tsx`
- Use URL search params for persistence: `useSearchParams()`
- Look at existing filters for state, assignee
- Use MobX store to manage filter state

#### Starter Code

```typescript
// apps/web/components/issues/filters/priority-filter.tsx

export const PriorityFilter = () => {
  const priorities = ['urgent', 'high', 'medium', 'low', 'none'];

  // TODO: Implement filter logic

  return (
    <div>
      <h3>Priority</h3>
      {/* TODO: Add checkboxes or multi-select */}
    </div>
  );
};
```

#### Self-Grading Checklist

- [ ] Filter dropdown renders with all priority options
- [ ] Selecting priorities filters the issue list
- [ ] Multiple priorities can be selected at once
- [ ] Filter state is stored in URL params
- [ ] Filter persists on page reload
- [ ] Issue counts are displayed for each priority
- [ ] Clearing filter shows all issues

#### Where to Look

- `apps/web/components/issues/issue-filter.tsx` - Existing filters
- `apps/web/stores/issue.store.ts` - Issue store
- `apps/web/lib/hooks/use-issue-filter.ts` - Filter hooks

---

### Exercise 3: Create a Django Management Command

**Level**: ⭐ Beginner
**Type**: Backend
**Time**: 30-45 minutes
**Skills**: Django, Management commands, Database queries

#### Objective

Create a Django management command that exports all issues from a project to a CSV file.

#### Requirements

1. Command should accept project ID as argument
2. Export issue ID, name, state, priority, assignees
3. Save to a CSV file in the specified location
4. Handle errors gracefully (invalid project ID, permissions)
5. Show progress for large projects

#### Hints

- Management commands go in `apps/api/plane/management/commands/`
- Use Django's `BaseCommand` class
- Use `csv.writer` from Python's csv module
- Query issues with `Issue.objects.filter(project_id=project_id)`

#### Starter Code

```python
# apps/api/plane/management/commands/export_issues.py

from django.core.management.base import BaseCommand
from plane.db.models import Issue, Project

class Command(BaseCommand):
    help = 'Export issues from a project to CSV'

    def add_arguments(self, parser):
        parser.add_argument('project_id', type=str, help='Project UUID')
        parser.add_argument('--output', type=str, default='issues.csv', help='Output file path')

    def handle(self, *args, **options):
        project_id = options['project_id']
        output_path = options['output']

        # TODO: Implement export logic
        self.stdout.write(self.style.SUCCESS('Export complete!'))
```

#### Self-Grading Checklist

- [ ] Command accepts project_id argument
- [ ] Command accepts optional --output argument
- [ ] CSV file is created with correct headers
- [ ] All issues from project are exported
- [ ] Assignees are properly formatted (comma-separated)
- [ ] Invalid project ID shows error message
- [ ] Progress is shown for large projects
- [ ] Command can be run: `python manage.py export_issues <project_id>`

#### Where to Look

- `apps/api/plane/management/commands/` - Other management commands
- `apps/api/plane/db/models/issue.py` - Issue model
- Python `csv` module documentation

---

### Exercise 4: Add Issue Description Word Count

**Level**: ⭐ Beginner
**Type**: Full-Stack
**Time**: 60-90 minutes
**Skills**: Django models, React components, API integration

#### Objective

Add a word count feature that displays the number of words in an issue's description.

#### Requirements

**Backend:**
1. Add a `description_word_count` field to Issue model (computed field)
2. Update field when description changes
3. Include in Issue serializer

**Frontend:**
1. Display word count below issue description editor
2. Update count in real-time as user types
3. Format number with commas (1,234)

#### Hints

**Backend:**
- Add a method to Issue model: `@property def description_word_count`
- Use `description_stripped` field for counting
- Split by whitespace and count tokens

**Frontend:**
- Add to `IssueDescriptionEditor` component
- Use `useMemo` to calculate count efficiently
- Display in a subtle, non-intrusive way

#### Self-Grading Checklist

**Backend:**
- [ ] `description_word_count` property added to Issue model
- [ ] Property returns correct word count
- [ ] Field included in IssueSerializer
- [ ] API returns word count in issue detail endpoint

**Frontend:**
- [ ] Word count displays below description editor
- [ ] Count updates in real-time as user types
- [ ] Number is formatted with commas
- [ ] Count is visible but not distracting
- [ ] Works for both create and edit modes

#### Where to Look

**Backend:**
- `apps/api/plane/db/models/issue.py` - Issue model
- `apps/api/plane/app/serializers/issue.py` - Issue serializer

**Frontend:**
- `apps/web/components/issues/issue-description-editor.tsx`
- `apps/web/lib/utils/format.ts` - Number formatting

---

## Intermediate Exercises

### Exercise 5: Implement Issue Templates

**Level**: ⭐⭐ Intermediate
**Type**: Full-Stack
**Time**: 3-4 hours
**Skills**: Django models, REST API, React forms, State management

#### Objective

Create an issue template feature that allows users to save and reuse issue templates (title, description, labels, etc.).

#### Requirements

**Backend:**
1. Create `IssueTemplate` model with fields: name, description, default_priority, default_labels
2. Create CRUD API endpoints for templates
3. Add template permissions (only project admins can create/edit)
4. Add endpoint to create issue from template

**Frontend:**
1. Add "Templates" section in project settings
2. Create UI to manage templates (create, edit, delete)
3. Add "Use Template" button on create issue modal
4. Pre-fill issue form with template data

#### Starter Code

**Backend Model:**

```python
# apps/api/plane/db/models/issue_template.py

from django.db import models
from .project import ProjectBaseModel

class IssueTemplate(ProjectBaseModel):
    name = models.CharField(max_length=255)
    description_html = models.TextField(default='<p></p>')
    default_priority = models.CharField(
        max_length=30,
        choices=(
            ('urgent', 'Urgent'),
            ('high', 'High'),
            ('medium', 'Medium'),
            ('low', 'Low'),
            ('none', 'None'),
        ),
        default='none'
    )
    default_state = models.ForeignKey('State', null=True, on_delete=models.SET_NULL)
    default_assignee = models.ForeignKey('User', null=True, on_delete=models.SET_NULL)
    default_labels = models.ManyToManyField('Label', blank=True)

    class Meta:
        db_table = 'issue_templates'
        ordering = ('-created_at',)
```

**Frontend Service:**

```typescript
// apps/web/services/issue-template.service.ts

export interface IssueTemplate {
  id: string;
  name: string;
  description_html: string;
  default_priority: string;
  default_state: string | null;
  default_assignee: string | null;
  default_labels: string[];
}

class IssueTemplateService {
  async list(workspaceSlug: string, projectId: string): Promise<IssueTemplate[]> {
    // TODO: Implement API call
  }

  async create(workspaceSlug: string, projectId: string, data: Partial<IssueTemplate>): Promise<IssueTemplate> {
    // TODO: Implement API call
  }

  async createIssueFromTemplate(workspaceSlug: string, projectId: string, templateId: string): Promise<Issue> {
    // TODO: Implement API call
  }
}

export const issueTemplateService = new IssueTemplateService();
```

#### Self-Grading Checklist

**Backend:**
- [ ] IssueTemplate model created with all fields
- [ ] Migration created and applied successfully
- [ ] CRUD endpoints created (`GET`, `POST`, `PATCH`, `DELETE`)
- [ ] Serializer handles many-to-many relationships correctly
- [ ] Permission checks prevent non-admins from creating templates
- [ ] Endpoint to create issue from template works
- [ ] Template labels are properly associated

**Frontend:**
- [ ] Templates list page shows all templates
- [ ] Create template form saves successfully
- [ ] Edit template form pre-fills with existing data
- [ ] Delete template works with confirmation
- [ ] "Use Template" dropdown appears on create issue modal
- [ ] Selecting template pre-fills issue form
- [ ] Template UI is intuitive and follows Plane's design

**API Endpoints:**
- [ ] `GET /api/workspaces/{slug}/projects/{id}/issue-templates/`
- [ ] `POST /api/workspaces/{slug}/projects/{id}/issue-templates/`
- [ ] `PATCH /api/workspaces/{slug}/projects/{id}/issue-templates/{template_id}/`
- [ ] `DELETE /api/workspaces/{slug}/projects/{id}/issue-templates/{template_id}/`
- [ ] `POST /api/workspaces/{slug}/projects/{id}/issue-templates/{template_id}/create-issue/`

#### Where to Look

- `apps/api/plane/db/models/` - Model examples
- `apps/api/plane/app/views/issue/base.py` - Issue view patterns
- `apps/web/components/issues/create-issue-modal.tsx` - Issue creation UI
- `apps/web/components/project/settings/` - Project settings UI

---

### Exercise 6: Add Issue Relationships Graph

**Level**: ⭐⭐ Intermediate
**Type**: Frontend
**Time**: 4-5 hours
**Skills**: React, D3.js/Cytoscape, Graph visualization

#### Objective

Create a visual graph that shows relationships between issues (blocks, relates to, etc.).

#### Requirements

1. Fetch issue relations from API
2. Render graph using a graph library (D3.js, Cytoscape, or React Flow)
3. Show issues as nodes, relationships as edges
4. Color-code by relationship type
5. Make nodes clickable (navigate to issue)
6. Add zoom and pan controls
7. Layout algorithm (force-directed or hierarchical)

#### Hints

- Use React Flow (easier) or D3.js (more control)
- Fetch relations from `/api/.../issues/{id}/relations/`
- Use `useMemo` to compute graph data structure
- Consider performance for large graphs (>100 nodes)

#### Starter Code

```typescript
// apps/web/components/issues/issue-relations-graph.tsx

import ReactFlow, { Node, Edge } from 'reactflow';
import 'reactflow/dist/style.css';

interface IssueRelationsGraphProps {
  issueId: string;
}

export const IssueRelationsGraph = ({ issueId }: IssueRelationsGraphProps) => {
  const [nodes, setNodes] = React.useState<Node[]>([]);
  const [edges, setEdges] = React.useState<Edge[]>([]);

  React.useEffect(() => {
    // TODO: Fetch relations and build graph data
  }, [issueId]);

  return (
    <div style={{ width: '100%', height: '600px' }}>
      <ReactFlow nodes={nodes} edges={edges} />
    </div>
  );
};
```

#### Self-Grading Checklist

- [ ] Graph renders with nodes and edges
- [ ] Nodes represent issues (show issue ID and title)
- [ ] Edges represent relationships with correct direction
- [ ] Relationship types are color-coded
- [ ] Clicking node navigates to issue detail
- [ ] Zoom and pan controls work
- [ ] Layout algorithm positions nodes logically
- [ ] Performance is good for 50+ issues
- [ ] Graph is responsive (adjusts to container size)

#### Where to Look

- `apps/web/components/issues/issue-detail.tsx` - Issue detail view
- React Flow documentation: https://reactflow.dev/
- `apps/api/plane/db/models/issue.py` - IssueRelation model

---

### Exercise 7: Implement Bulk Issue Update

**Level**: ⭐⭐ Intermediate
**Type**: Backend
**Time**: 2-3 hours
**Skills**: Django REST Framework, Transactions, Performance

#### Objective

Create an API endpoint that allows bulk updating of multiple issues at once (e.g., change state, assignee, priority for selected issues).

#### Requirements

1. Create endpoint accepting list of issue IDs and update data
2. Update all issues in a single database transaction
3. Validate that user has permission for all issues
4. Create activity entries for all changes
5. Trigger webhooks for bulk changes
6. Optimize to avoid N+1 queries

#### Starter Code

```python
# apps/api/plane/app/views/issue/bulk.py

from rest_framework import status
from rest_framework.response import Response
from django.db import transaction

from plane.app.views.base import BaseAPIView
from plane.app.permissions import ProjectMemberPermission
from plane.db.models import Issue, IssueActivity

class IssueBulkUpdateView(BaseAPIView):
    permission_classes = [ProjectMemberPermission]

    def patch(self, request, slug, project_id):
        issue_ids = request.data.get('issue_ids', [])
        updates = request.data.get('updates', {})

        if not issue_ids:
            return Response(
                {"error": "issue_ids is required"},
                status=status.HTTP_400_BAD_REQUEST
            )

        # TODO: Implement bulk update with transaction
        # 1. Validate user has access to all issues
        # 2. Update issues in transaction
        # 3. Create activity entries
        # 4. Trigger webhooks

        return Response({"message": "Issues updated"})
```

#### Self-Grading Checklist

- [ ] Endpoint accepts issue_ids and updates payload
- [ ] Validates all issues exist in the project
- [ ] Checks user has permission for all issues
- [ ] Updates are applied in a single transaction
- [ ] Transaction rolls back if any update fails
- [ ] Activity entries created for each issue
- [ ] Supports updating: state, priority, assignees, labels
- [ ] No N+1 queries (use `select_related`, `prefetch_related`)
- [ ] Webhooks are triggered for changes
- [ ] Returns updated issue count

#### Where to Look

- `apps/api/plane/app/views/issue/base.py` - Issue update patterns
- `apps/api/plane/bgtasks/issue_activities_task.py` - Activity creation
- `apps/api/plane/bgtasks/webhook_task.py` - Webhook triggering
- Django transaction documentation

---

### Exercise 8: Add Real-time Presence Indicators

**Level**: ⭐⭐ Intermediate
**Type**: Full-Stack
**Time**: 4-6 hours
**Skills**: WebSockets, Django Channels, React, Real-time updates

#### Objective

Show which users are currently viewing an issue (presence indicators like "John is viewing this issue").

#### Requirements

**Backend:**
1. Set up Django Channels for WebSocket support
2. Create consumer for issue presence
3. Track user joins/leaves on issue view
4. Broadcast presence updates to all viewers

**Frontend:**
1. Connect to WebSocket when viewing issue
2. Display avatars of current viewers
3. Show "X users viewing" count
4. Update in real-time as users join/leave

#### Hints

**Backend:**
- Use Django Channels with Redis for channel layer
- Create consumer in `apps/api/plane/consumers/issue_presence.py`
- Use channel groups per issue: `issue_{issue_id}`

**Frontend:**
- Use WebSocket API or library like `socket.io-client`
- Connect in `useEffect` when issue mounts
- Clean up connection on unmount

#### Self-Grading Checklist

**Backend:**
- [ ] Django Channels installed and configured
- [ ] Redis channel layer configured
- [ ] WebSocket consumer handles connect/disconnect
- [ ] Presence state stored (user ID, issue ID, timestamp)
- [ ] Broadcasts presence updates to issue group
- [ ] Cleans up stale presence data

**Frontend:**
- [ ] WebSocket connects when viewing issue
- [ ] Displays list of current viewers
- [ ] Updates in real-time as users join/leave
- [ ] Shows user avatars
- [ ] Disconnects cleanly on unmount
- [ ] Handles connection errors gracefully

#### Where to Look

- Django Channels documentation: https://channels.readthedocs.io/
- `apps/api/plane/asgi.py` - ASGI configuration
- WebSocket API: https://developer.mozilla.org/en-US/docs/Web/API/WebSocket

---

## Advanced Exercises

### Exercise 9: Implement Custom Field Types

**Level**: ⭐⭐⭐ Advanced
**Type**: Full-Stack
**Time**: 8-12 hours
**Skills**: Django models, Schema migrations, Complex forms, JSON fields

#### Objective

Add support for custom fields on issues (e.g., "Customer Name", "Revenue Impact", "Severity Score") that admins can define per project.

#### Requirements

**Backend:**
1. Create `CustomField` model (name, type, options)
2. Field types: text, number, select, multi-select, date
3. Store values in JSON field on Issue model
4. Validate custom field values based on field type
5. Support filtering/sorting by custom fields

**Frontend:**
1. Admin UI to create/edit custom fields
2. Show custom fields in issue create/edit forms
3. Render appropriate input based on field type
4. Display custom fields in issue detail view
5. Support filtering by custom fields

#### Architecture Hints

**Backend:**
```python
# Custom field model
class CustomField(ProjectBaseModel):
    name = models.CharField(max_length=255)
    field_type = models.CharField(choices=[
        ('text', 'Text'),
        ('number', 'Number'),
        ('select', 'Single Select'),
        ('multi_select', 'Multi Select'),
        ('date', 'Date'),
    ])
    options = models.JSONField(default=dict)  # For select types
    is_required = models.BooleanField(default=False)
    sequence = models.IntegerField(default=0)

# Store values on Issue
class Issue(ProjectBaseModel):
    # ... existing fields
    custom_fields = models.JSONField(default=dict)
    # Structure: {"field_id": "value"}
```

**Frontend:**
```typescript
interface CustomField {
  id: string;
  name: string;
  field_type: 'text' | 'number' | 'select' | 'multi_select' | 'date';
  options?: { value: string; label: string }[];
  is_required: boolean;
}

const CustomFieldInput = ({ field, value, onChange }: CustomFieldInputProps) => {
  switch (field.field_type) {
    case 'text':
      return <input type="text" value={value} onChange={onChange} />;
    case 'number':
      return <input type="number" value={value} onChange={onChange} />;
    case 'select':
      return <select value={value} onChange={onChange}>...</select>;
    // ... other types
  }
};
```

#### Self-Grading Checklist

**Backend:**
- [ ] CustomField model created with all field types
- [ ] Issue model has custom_fields JSON field
- [ ] CRUD endpoints for custom fields
- [ ] Validation logic for each field type
- [ ] Custom field values saved on issue create/update
- [ ] Filtering by custom fields works
- [ ] Serializer includes custom fields in issue response

**Frontend:**
- [ ] Custom fields management UI in project settings
- [ ] Create custom field form with all field types
- [ ] Issue form dynamically shows custom fields
- [ ] Appropriate input rendered for each field type
- [ ] Required custom fields are validated
- [ ] Custom fields display in issue detail view
- [ ] Can filter issues by custom field values

**Bonus:**
- [ ] Support default values for custom fields
- [ ] Support field visibility rules (show if X is Y)
- [ ] Export custom fields in issue export
- [ ] Bulk edit custom field values

#### Where to Look

- `apps/api/plane/db/models/` - Model patterns
- `apps/web/components/project/settings/` - Settings UI
- `apps/web/components/issues/issue-form.tsx` - Issue form
- Django JSONField documentation

---

### Exercise 10: Build Activity Dashboard

**Level**: ⭐⭐⭐ Advanced
**Type**: Full-Stack
**Time**: 10-15 hours
**Skills**: Data aggregation, Charts, Complex queries, Caching

#### Objective

Create a project dashboard showing activity metrics, charts, and insights.

#### Requirements

**Backend:**
1. Aggregate issues by state, priority, assignee
2. Calculate velocity (issues completed per week)
3. Compute burn-down data for cycles
4. Track issue aging (time in each state)
5. Cache expensive aggregations
6. Optimize with database indexes

**Frontend:**
1. Dashboard page with multiple chart sections
2. Bar chart: Issues by state
3. Pie chart: Issues by priority
4. Line chart: Velocity over time
5. Burn-down chart for active cycle
6. Date range filter
7. Real-time updates (WebSocket or polling)

#### Starter Code

**Backend:**

```python
# apps/api/plane/app/views/analytics/project_dashboard.py

from django.db.models import Count, Q, F
from django.utils import timezone
from datetime import timedelta
from rest_framework.response import Response

from plane.app.views.base import BaseAPIView

class ProjectDashboardView(BaseAPIView):
    def get(self, request, slug, project_id):
        # Issues by state
        issues_by_state = Issue.objects.filter(
            project_id=project_id,
            deleted_at__isnull=True
        ).values('state__name', 'state__color').annotate(
            count=Count('id')
        )

        # Issues by priority
        issues_by_priority = Issue.objects.filter(
            project_id=project_id,
            deleted_at__isnull=True
        ).values('priority').annotate(
            count=Count('id')
        )

        # Velocity (last 8 weeks)
        weeks_ago = timezone.now() - timedelta(weeks=8)
        velocity = self._calculate_velocity(project_id, weeks_ago)

        # TODO: Add more metrics

        return Response({
            'issues_by_state': list(issues_by_state),
            'issues_by_priority': list(issues_by_priority),
            'velocity': velocity,
        })

    def _calculate_velocity(self, project_id, start_date):
        # TODO: Group completed issues by week
        pass
```

**Frontend:**

```typescript
// apps/web/components/analytics/project-dashboard.tsx

import { BarChart, PieChart, LineChart } from 'recharts';

export const ProjectDashboard = ({ workspaceSlug, projectId }: Props) => {
  const [data, setData] = React.useState<DashboardData | null>(null);

  React.useEffect(() => {
    fetchDashboardData();
  }, [workspaceSlug, projectId]);

  const fetchDashboardData = async () => {
    // TODO: Fetch from API
  };

  return (
    <div className="space-y-8">
      <section>
        <h2>Issues by State</h2>
        <BarChart data={data?.issues_by_state} />
      </section>

      <section>
        <h2>Issues by Priority</h2>
        <PieChart data={data?.issues_by_priority} />
      </section>

      <section>
        <h2>Velocity</h2>
        <LineChart data={data?.velocity} />
      </section>
    </div>
  );
};
```

#### Self-Grading Checklist

**Backend:**
- [ ] Endpoint returns all required metrics
- [ ] Issues aggregated by state correctly
- [ ] Issues aggregated by priority correctly
- [ ] Velocity calculation groups by week
- [ ] Burn-down calculation for active cycle
- [ ] Date range filtering works
- [ ] Expensive queries are cached (Redis)
- [ ] Database indexes optimize queries
- [ ] Response time < 500ms for typical project

**Frontend:**
- [ ] Dashboard page renders all sections
- [ ] Bar chart displays issues by state
- [ ] Pie chart displays issues by priority
- [ ] Line chart displays velocity trend
- [ ] Burn-down chart for active cycle
- [ ] Date range picker filters data
- [ ] Charts are responsive
- [ ] Loading states shown while fetching
- [ ] Error states handled gracefully
- [ ] Charts use consistent color scheme

**Bonus:**
- [ ] Real-time updates via WebSocket
- [ ] Export dashboard as PDF/PNG
- [ ] Drill-down: click chart to see issues
- [ ] Comparison with previous period
- [ ] Customizable dashboard layout

#### Where to Look

- `apps/api/plane/app/views/analytic/` - Existing analytics
- Chart libraries: Recharts, Victory, or Chart.js
- Django aggregation: https://docs.djangoproject.com/en/4.2/topics/db/aggregation/
- Redis caching patterns

---

## Full-Stack Challenges

### Exercise 11: Build a Comment Mention System

**Level**: ⭐⭐⭐ Advanced
**Type**: Full-Stack
**Time**: 8-12 hours
**Skills**: Rich text editing, Real-time search, Notifications

#### Objective

Implement @mentions in issue comments that notify mentioned users.

#### Requirements

**Backend:**
1. Parse comment HTML for @mentions
2. Extract mentioned user IDs
3. Create notifications for mentioned users
4. Store mentions in `issue_mentions` table
5. Send email to mentioned users (optional)

**Frontend:**
1. Rich text editor with @mention support
2. Autocomplete dropdown when typing @
3. Render mentions with special styling
4. Show who's mentioned in comment

#### Implementation Guide

**Backend:**

```python
# apps/api/plane/utils/mention_parser.py

from bs4 import BeautifulSoup
import re

def extract_mentions(comment_html):
    """Extract user IDs from @mention tags in HTML"""
    soup = BeautifulSoup(comment_html, 'html.parser')
    mentions = soup.find_all('span', class_='mention')

    user_ids = []
    for mention in mentions:
        user_id = mention.get('data-user-id')
        if user_id:
            user_ids.append(user_id)

    return user_ids

# In comment view
def create(self, request, slug, project_id, issue_id):
    comment_html = request.data.get('comment_html')

    # Save comment
    comment = IssueComment.objects.create(
        issue_id=issue_id,
        actor=request.user,
        comment_html=comment_html
    )

    # Extract and save mentions
    mentioned_user_ids = extract_mentions(comment_html)
    for user_id in mentioned_user_ids:
        IssueMention.objects.create(
            issue_id=issue_id,
            mention_id=user_id
        )

        # Create notification
        create_mention_notification(
            sender=request.user,
            receiver_id=user_id,
            issue_id=issue_id,
            comment=comment
        )

    return Response(CommentSerializer(comment).data)
```

**Frontend:**

```typescript
// Use TipTap editor with mention extension

import { useEditor } from '@tiptap/react';
import Mention from '@tiptap/extension-mention';

const CommentEditor = () => {
  const editor = useEditor({
    extensions: [
      StarterKit,
      Mention.configure({
        HTMLAttributes: {
          class: 'mention',
        },
        suggestion: {
          items: async ({ query }) => {
            // Fetch users matching query
            const users = await searchUsers(query);
            return users;
          },
          render: () => {
            // Render autocomplete dropdown
          },
        },
      }),
    ],
  });

  return <EditorContent editor={editor} />;
};
```

#### Self-Grading Checklist

- [ ] @ symbol triggers user autocomplete
- [ ] Autocomplete searches project members
- [ ] Selecting user inserts mention tag
- [ ] Mentions render with special styling
- [ ] Backend parses mentions from HTML
- [ ] IssueMention records created
- [ ] Notifications created for mentioned users
- [ ] Mentioned users receive notification
- [ ] Can mention multiple users in one comment
- [ ] Edge cases handled (invalid users, self-mention)

---

### Exercise 12: Implement Issue Dependencies

**Level**: ⭐⭐⭐ Advanced
**Type**: Full-Stack
**Time**: 12-16 hours
**Skills**: Complex relationships, Validation, Graph algorithms

#### Objective

Add support for issue dependencies: "Issue A depends on Issue B" (A can't be completed until B is done).

#### Requirements

**Backend:**
1. Extend IssueRelation model to support dependency type
2. Validate no circular dependencies
3. Prevent marking issue as done if dependencies incomplete
4. API to add/remove dependencies
5. Compute dependency graph depth

**Frontend:**
1. UI to add dependencies on issue detail page
2. Show dependency tree visualization
3. Show blocking/blocked by issues
4. Warn when trying to complete issue with incomplete dependencies
5. Gantt chart view (bonus)

#### Key Challenge: Circular Dependency Detection

```python
# apps/api/plane/utils/dependency_validator.py

def has_circular_dependency(issue_id, depends_on_id):
    """
    Check if adding dependency creates a cycle.

    Example:
    A depends on B
    B depends on C
    If we try to add "C depends on A", it creates a cycle!
    """
    visited = set()
    stack = [depends_on_id]

    while stack:
        current = stack.pop()
        if current == issue_id:
            return True  # Cycle detected!

        if current in visited:
            continue

        visited.add(current)

        # Get issues that current depends on
        dependencies = IssueRelation.objects.filter(
            issue_id=current,
            relation_type='depends_on'
        ).values_list('related_issue_id', flat=True)

        stack.extend(dependencies)

    return False

# In view
def add_dependency(self, request, slug, project_id, issue_id):
    depends_on_id = request.data.get('depends_on')

    if has_circular_dependency(issue_id, depends_on_id):
        return Response(
            {"error": "Adding this dependency would create a cycle"},
            status=status.HTTP_400_BAD_REQUEST
        )

    IssueRelation.objects.create(
        issue_id=issue_id,
        related_issue_id=depends_on_id,
        relation_type='depends_on'
    )
```

#### Self-Grading Checklist

**Backend:**
- [ ] Can add dependency between issues
- [ ] Circular dependency validation prevents cycles
- [ ] Cannot complete issue with incomplete dependencies
- [ ] API returns dependency tree
- [ ] Efficient queries (no N+1 problems)

**Frontend:**
- [ ] Can add dependency from issue detail page
- [ ] Shows list of dependencies (depends on / blocked by)
- [ ] Dependency tree visualization
- [ ] Warning when trying to complete with incomplete deps
- [ ] Can remove dependencies

**Bonus:**
- [ ] Gantt chart showing dependency timeline
- [ ] Critical path calculation
- [ ] Suggest optimal completion order

---

## Performance Optimization

### Exercise 13: Optimize Slow Issue List Query

**Level**: ⭐⭐⭐ Advanced
**Type**: Backend
**Time**: 4-6 hours
**Skills**: Database optimization, Query analysis, Caching

#### Objective

Optimize a slow issue list query that's taking 5+ seconds for large projects.

#### Scenario

```python
# Current slow query
def list(self, request, slug, project_id):
    issues = Issue.objects.filter(project_id=project_id)

    # This causes N+1 queries!
    for issue in issues:
        issue.assignee_count = issue.assignees.count()
        issue.label_names = [label.name for label in issue.labels.all()]
        issue.comment_count = issue.issue_comments.count()

    return Response(IssueSerializer(issues, many=True).data)
```

#### Requirements

1. Identify N+1 query problems
2. Use `select_related` and `prefetch_related`
3. Add database indexes
4. Use annotations for aggregates
5. Implement pagination
6. Add Redis caching for expensive computations
7. Measure performance improvements

#### Solution Approach

```python
# Optimized query
def list(self, request, slug, project_id):
    issues = Issue.objects.filter(
        project_id=project_id
    ).select_related(
        'state',  # Single join
        'project',
        'parent'
    ).prefetch_related(
        'assignees',  # Batch fetch
        'labels',
        'issue_comments'
    ).annotate(
        assignee_count=Count('assignees'),
        label_count=Count('labels'),
        comment_count=Count('issue_comments')
    )

    # Add pagination
    paginator = self.pagination_class()
    paginated_issues = paginator.paginate_queryset(issues, request)

    return paginator.get_paginated_response(
        IssueSerializer(paginated_issues, many=True).data
    )
```

#### Self-Grading Checklist

- [ ] Django Debug Toolbar installed to analyze queries
- [ ] N+1 queries eliminated with `select_related`/`prefetch_related`
- [ ] Aggregates computed with `annotate` instead of loops
- [ ] Database indexes added for frequently queried fields
- [ ] Pagination implemented (20-50 items per page)
- [ ] Redis caching added for expensive computations
- [ ] Query count reduced from 100+ to <10
- [ ] Response time improved from 5s to <500ms

#### Where to Look

- Django Debug Toolbar: https://django-debug-toolbar.readthedocs.io/
- Django query optimization: https://docs.djangoproject.com/en/4.2/topics/db/optimization/
- `apps/api/plane/app/views/issue/base.py` - Current implementation

---

## Security Challenges

### Exercise 14: Fix IDOR Vulnerability

**Level**: ⭐⭐ Intermediate
**Type**: Backend
**Time**: 2-3 hours
**Skills**: Security, Authorization, Testing

#### Objective

Find and fix an Insecure Direct Object Reference (IDOR) vulnerability in the codebase.

#### Vulnerable Code

```python
# apps/api/plane/app/views/issue/vulnerable.py

class IssueDetailView(APIView):
    def get(self, request, issue_id):
        # VULNERABILITY: No permission check!
        issue = Issue.objects.get(id=issue_id)
        return Response(IssueSerializer(issue).data)

    def patch(self, request, issue_id):
        # VULNERABILITY: Any user can update any issue!
        issue = Issue.objects.get(id=issue_id)
        issue.name = request.data.get('name')
        issue.save()
        return Response(IssueSerializer(issue).data)
```

#### Requirements

1. Identify the vulnerability
2. Add proper permission checks
3. Return 404 (not 403) for unauthorized access
4. Write tests to verify the fix
5. Apply fix to all similar endpoints

#### Solution

```python
# Fixed version
class IssueDetailView(APIView):
    permission_classes = [ProjectMemberPermission]

    def get(self, request, slug, project_id, issue_id):
        # Check user is project member
        issue = Issue.objects.filter(
            id=issue_id,
            project_id=project_id,
            project__project_projectmember__member=request.user,
            project__project_projectmember__is_active=True
        ).first()

        if not issue:
            # Return 404 to not reveal issue existence
            return Response(
                {"error": "Issue not found"},
                status=status.HTTP_404_NOT_FOUND
            )

        return Response(IssueSerializer(issue).data)
```

#### Self-Grading Checklist

- [ ] Permission check added to all issue endpoints
- [ ] Returns 404 (not 403) for unauthorized access
- [ ] Test case verifies user A cannot access user B's issue
- [ ] Test case verifies user can access own issues
- [ ] Applied fix to update, delete endpoints
- [ ] Checked other models for similar vulnerabilities
- [ ] Security audit documented

#### Where to Look

- `apps/api/plane/app/permissions.py` - Permission classes
- `apps/api/plane/app/views/base.py` - Base view classes
- [SECURITY_GUIDE.md](./SECURITY_GUIDE.md) - Security patterns

---

## Solutions & Hints

### Getting Unstuck

If you're stuck on an exercise:

1. **Read the hints section** - They provide guidance on approach
2. **Look at "Where to Look"** - Similar code exists in the codebase
3. **Check related docs** - SECURITY_GUIDE, API_DOCUMENTATION, etc.
4. **Search the codebase** - Use grep/search to find examples
5. **Ask for help** - Open a GitHub discussion or ask the community

### Solution Locations

Full solutions for exercises are available in:

```
/home/user/plane/docs/learning/exercise-solutions/
├── exercise-01-color-picker.md
├── exercise-02-priority-filter.md
├── exercise-03-export-command.md
└── ...
```

(Note: Create these solution files as you complete exercises!)

### Testing Your Solutions

Before marking an exercise complete:

1. **Run tests**: Ensure all tests pass
2. **Manual testing**: Test in the UI/API
3. **Check list**: Complete self-grading checklist
4. **Code review**: Ask someone to review your code
5. **Document**: Write comments explaining your approach

### Progressive Learning Path

**Week 1-2: Beginner Exercises (1-4)**
- Build confidence with small changes
- Learn codebase structure
- Get familiar with React and Django patterns

**Week 3-4: Intermediate Exercises (5-8)**
- Build full features end-to-end
- Learn state management and API integration
- Understand database design

**Week 5-8: Advanced Exercises (9-12)**
- Tackle complex features
- Learn performance optimization
- Master the full stack

**Ongoing: Challenges (13-14)**
- Improve existing code
- Learn security best practices
- Contribute to production code

---

## React Analogy: Learning Progression

**Beginner**: Learning React basics (components, props, state)
**Intermediate**: Building features (forms, routing, API calls)
**Advanced**: Architecting systems (state management, performance, patterns)

Similarly, these exercises progress from simple component changes to complex full-stack features!

---

## Status: ✅ CURRENT (November 2025)

All exercises reflect the current Plane codebase and tech stack:

- ✅ React Router 7 patterns
- ✅ Django 4.2 best practices
- ✅ TypeScript 5.8 features
- ✅ Modern security patterns

---

## Further Reading

### Related Documentation
- [GETTING_STARTED.md](./GETTING_STARTED.md) - Setup guide
- [TECH_STACK_GUIDE.md](./TECH_STACK_GUIDE.md) - Technology overview
- [TESTING_GUIDE.md](./TESTING_GUIDE.md) - Testing patterns
- [SECURITY_GUIDE.md](./SECURITY_GUIDE.md) - Security best practices

### External Resources
- React documentation: https://react.dev/
- Django documentation: https://docs.djangoproject.com/
- TypeScript handbook: https://www.typescriptlang.org/docs/
- PostgreSQL documentation: https://www.postgresql.org/docs/

---

**Happy coding!** Remember: The best way to learn is by doing. Start with Exercise 1 and work your way up. Every exercise builds skills you'll use in production code!
