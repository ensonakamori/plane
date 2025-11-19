# Getting Started with Plane

**Welcome to Plane!** This guide will get you from zero to running the application locally.

**Estimated Time:** 1-2 hours (depending on your internet speed)
**Difficulty:** Beginner
**Prerequisites:** Basic command line knowledge

**Documentation Date:** November 19, 2025
**Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [System Requirements](#system-requirements)
- [Installation Steps](#installation-steps)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

---

## 🎯 What You'll Accomplish

By the end of this guide, you will have:

- ✅ All required tools installed (Node.js, Python, PostgreSQL, Redis)
- ✅ Plane codebase cloned and dependencies installed
- ✅ Local development environment running
- ✅ Frontend accessible at `http://localhost:3000`
- ✅ Backend API accessible at `http://localhost:8000`
- ✅ Confidence to start exploring the code

---

## Prerequisites

### Required Knowledge

- Basic command line usage (cd, ls, running commands)
- Basic Git usage (clone, status, pull)
- Text editor or IDE (VS Code recommended)

### Required Tools (We'll install these)

- **Node.js 22.18+** - JavaScript runtime for frontend
- **Python 3.12+** - Backend runtime
- **pnpm 10+** - Package manager (faster than npm)
- **PostgreSQL 14+** - Primary database
- **Redis 5+** - Caching and message broker
- **Git** - Version control (you probably have this)

---

## System Requirements

### Minimum

- **RAM:** 8GB
- **Disk:** 5GB free space
- **OS:** macOS, Linux, or Windows (with WSL2)
- **Internet:** For downloading dependencies

### Recommended

- **RAM:** 16GB (better dev experience)
- **CPU:** 4+ cores
- **SSD:** For faster builds

---

## Installation Steps

### Step 1: Install Node.js 22+

**Why Node.js?** The frontend is built with React, which requires Node.js to run the development server and build tools.

#### macOS (using Homebrew)
```bash
# Install Homebrew if you don't have it
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Node.js 22
brew install node@22

# Verify installation
node --version  # Should show v22.x.x
```

#### Linux (Ubuntu/Debian)
```bash
# Install Node.js 22 LTS
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version  # Should show v22.x.x
```

#### Windows (using WSL2)
Follow Linux instructions above in WSL2 terminal.

**🎯 Remember:** Node 22 is required (not Node 18 or 20) as specified in package.json.

---

### Step 2: Install pnpm 10+

**Why pnpm?** Faster than npm, more reliable than yarn, perfect for monorepos.

```bash
# Install pnpm globally
npm install -g pnpm@10

# Verify installation
pnpm --version  # Should show 10.x.x
```

**🧠 Mental Model:** Think of pnpm like npm, but it creates a single shared store for all packages across all projects, saving disk space and installation time.

---

### Step 3: Install Python 3.12+

**Why Python?** The backend API is built with Django, a Python web framework.

#### macOS (using Homebrew)
```bash
# Install Python 3.12
brew install python@3.12

# Verify installation
python3.12 --version  # Should show 3.12.x

# Create a symlink for convenience
ln -s /opt/homebrew/bin/python3.12 /usr/local/bin/python3
```

#### Linux (Ubuntu/Debian)
```bash
# Add deadsnakes PPA for newer Python versions
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update

# Install Python 3.12
sudo apt install python3.12 python3.12-venv python3.12-dev

# Verify installation
python3.12 --version  # Should show 3.12.x
```

#### Windows (WSL2)
Follow Linux instructions above.

---

### Step 4: Install PostgreSQL

**Why PostgreSQL?** The primary database for storing all application data (users, projects, issues, etc.).

#### macOS (using Homebrew)
```bash
# Install PostgreSQL
brew install postgresql@14

# Start PostgreSQL service
brew services start postgresql@14

# Verify installation
psql --version  # Should show 14.x
```

#### Linux (Ubuntu/Debian)
```bash
# Install PostgreSQL
sudo apt install postgresql postgresql-contrib

# Start PostgreSQL service
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Verify installation
psql --version  # Should show 14.x or higher
```

**🧠 Mental Model for Frontend Devs:**
Think of PostgreSQL as a giant, well-organized JSON store with powerful query capabilities. Instead of storing data in variables or state, the backend stores it in tables that persist forever.

---

### Step 5: Install Redis

**Why Redis?** Used for caching (making things fast) and as a message broker for background tasks.

#### macOS (using Homebrew)
```bash
# Install Redis
brew install redis

# Start Redis service
brew services start redis

# Verify installation
redis-cli ping  # Should respond with "PONG"
```

#### Linux (Ubuntu/Debian)
```bash
# Install Redis
sudo apt install redis-server

# Start Redis service
sudo systemctl start redis-server
sudo systemctl enable redis-server

# Verify installation
redis-cli ping  # Should respond with "PONG"
```

**🧠 Mental Model for Frontend Devs:**
Redis is like localStorage on the backend - fast, in-memory storage. But unlike localStorage:
- It can be shared between multiple backend servers
- It can expire automatically
- It's used for caching database queries and session data

---

### Step 6: Clone the Repository

```bash
# Clone the repo
git clone https://github.com/makeplane/plane.git

# Navigate into the project
cd plane

# Check your current branch
git branch  # You should be on a branch
```

---

### Step 7: Install Frontend Dependencies

```bash
# Install all frontend dependencies (this may take 5-10 minutes)
pnpm install

# Verify installation
ls node_modules  # Should see many packages
```

**What just happened?**
- pnpm read `package.json` and `pnpm-workspace.yaml`
- It downloaded all dependencies for all apps and packages
- It created symlinks for workspace dependencies
- Total size: ~500MB of node_modules

**⚠️ Common Pitfall:** Don't use `npm install` or `yarn install` - this project uses pnpm specifically for monorepo support and performance.

---

### Step 8: Set Up Backend Environment

#### Create Python Virtual Environment

**Why virtual environment?** Isolates Python dependencies for this project from your system Python.

```bash
# Navigate to API directory
cd apps/api

# Create virtual environment
python3.12 -m venv venv

# Activate virtual environment
source venv/bin/activate  # On macOS/Linux
# OR
venv\Scripts\activate  # On Windows

# Your prompt should now show (venv)
```

**🧠 Mental Model:**
Virtual environment is like `node_modules` for Python - a project-specific folder containing all Python packages.

#### Install Python Dependencies

```bash
# Make sure you're in apps/api and venv is activated
pip install --upgrade pip
pip install -r requirements.txt

# This will install:
# - Django 4.2 (web framework)
# - Django REST Framework (API tools)
# - Celery (background tasks)
# - psycopg (PostgreSQL driver)
# - and many more...

# Verify installation
python -m django --version  # Should show 4.2.x
```

---

### Step 9: Set Up Database

#### Create PostgreSQL Database

```bash
# Connect to PostgreSQL as superuser
# macOS:
psql postgres

# Linux:
sudo -u postgres psql
```

In the PostgreSQL prompt:
```sql
-- Create database
CREATE DATABASE plane_db;

-- Create user
CREATE USER plane_user WITH PASSWORD 'plane_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE plane_db TO plane_user;

-- Exit
\q
```

**🧠 Mental Model for Frontend Devs:**
This is like setting up a new localStorage namespace. The database (`plane_db`) is where all data lives, and the user (`plane_user`) is like an API key with permissions to access it.

---

### Step 10: Configure Environment Variables

#### Frontend Environment

```bash
# Navigate to web app
cd apps/web

# Copy example env file
cp .env.example .env

# Edit .env file
```

Key variables in `apps/web/.env`:
```bash
# API endpoint (backend URL)
VITE_API_BASE_URL=http://localhost:8000

# Enable dev mode
VITE_DEV_MODE=true
```

#### Backend Environment

```bash
# Navigate to API
cd apps/api

# Copy example env file
cp .env.example .env

# Edit .env file
```

Key variables in `apps/api/.env`:
```bash
# Database
DATABASE_URL=postgresql://plane_user:plane_password@localhost:5432/plane_db

# Redis
REDIS_URL=redis://localhost:6379/

# Django secret key (generate a random string)
SECRET_KEY=your-secret-key-here

# Debug mode (only for development!)
DEBUG=True

# Allowed hosts
ALLOWED_HOSTS=localhost,127.0.0.1
```

**🎯 Remember:** Never commit `.env` files! They contain secrets.

---

### Step 11: Run Database Migrations

**What are migrations?** Think of them as "Git commits" for your database schema. They create and update database tables.

```bash
# Make sure you're in apps/api with venv activated
cd apps/api
source venv/bin/activate  # If not already activated

# Run migrations
python manage.py migrate

# You should see output like:
# Operations to perform:
#   Apply all migrations: ...
# Running migrations:
#   Applying contenttypes.0001_initial... OK
#   Applying auth.0001_initial... OK
#   ... (many more) ...
```

**What just happened?**
- Django read migration files in `apps/api/plane/*/migrations/`
- It created tables in PostgreSQL (users, projects, issues, etc.)
- Your database now has the schema matching the Django models

**🧠 Mental Model:**
Migrations are like TypeScript interfaces being written to the database. Each migration adds/modifies tables, just like updating a type definition.

---

### Step 12: Create Superuser (Admin)

```bash
# Create admin account
python manage.py createsuperuser

# Follow prompts:
# Email: your@email.com
# Password: ******** (make it strong!)
# Password (again): ********
```

This creates your first user account with admin privileges.

---

### Step 13: Start Backend Server

```bash
# Make sure you're in apps/api with venv activated
cd apps/api
source venv/bin/activate

# Start Django development server
python manage.py runserver

# You should see:
# Starting development server at http://127.0.0.1:8000/
# Quit the server with CONTROL-C.
```

**✅ Checkpoint:** Visit http://localhost:8000/api/ - you should see a browsable API interface!

Leave this terminal running and open a new terminal for frontend.

---

### Step 14: Start Frontend Server

In a **new terminal**:

```bash
# Navigate to web app
cd apps/web

# Start development server
pnpm dev

# You should see:
#  ➜  Local:   http://localhost:3000/
#  ➜  press h + enter to show help
```

**✅ Checkpoint:** Visit http://localhost:3000/ - you should see the Plane app!

---

### Step 15: Start Celery Worker (Optional)

For background tasks like sending emails, processing uploads:

In a **new terminal**:

```bash
# Navigate to API
cd apps/api
source venv/bin/activate

# Start Celery worker
celery -A plane worker -l info

# You should see:
# [tasks]
#   . plane.bgtasks.issue_task
#   ... (many more tasks)
# [worker] ready.
```

---

## Verification

### ✅ Everything Working?

Check each service:

1. **PostgreSQL:** `redis-cli ping` → "PONG"
2. **Redis:** `psql -U plane_user -d plane_db -c "SELECT 1;"` → returns 1
3. **Backend API:** http://localhost:8000/api/ → Browsable API page
4. **Frontend:** http://localhost:3000/ → Plane login page
5. **Celery (optional):** Worker terminal shows "ready"

### 🎉 Success!

If all checks pass, congratulations! Your development environment is ready.

---

## Troubleshooting

### Issue: `pnpm install` fails

**Solution:**
```bash
# Clear pnpm cache
pnpm store prune

# Remove node_modules
rm -rf node_modules

# Try again
pnpm install
```

---

### Issue: `python manage.py migrate` fails

**Common causes:**

1. **Database not running:**
   ```bash
   # macOS
   brew services list  # Check if postgresql is started
   brew services restart postgresql@14

   # Linux
   sudo systemctl status postgresql
   sudo systemctl start postgresql
   ```

2. **Wrong credentials:**
   - Check `DATABASE_URL` in `apps/api/.env`
   - Verify database and user exist:
     ```bash
     psql postgres -c "\l"  # List databases
     ```

3. **Port already in use:**
   - PostgreSQL default port is 5432
   - Check if something else is using it:
     ```bash
     lsof -i :5432
     ```

---

### Issue: `pnpm dev` shows errors

**Common causes:**

1. **Wrong Node version:**
   ```bash
   node --version  # Must be 22.x.x
   ```

2. **Missing environment variables:**
   - Check `apps/web/.env` exists
   - Verify `VITE_API_BASE_URL` is set

3. **Port 3000 already in use:**
   ```bash
   lsof -i :3000  # Find process using port
   kill -9 <PID>  # Kill the process
   ```

---

### Issue: Backend returns 500 errors

**Debug steps:**

1. **Check backend terminal** for error messages
2. **Check database connection:**
   ```bash
   cd apps/api
   python manage.py dbshell  # Should open PostgreSQL prompt
   ```
3. **Verify Redis is running:**
   ```bash
   redis-cli ping  # Should return PONG
   ```
4. **Check Django logs:**
   ```bash
   # In apps/api
   tail -f logs/django.log  # If logging is configured
   ```

---

### Issue: Frontend can't reach backend

**Symptoms:** Network errors, 404s, CORS errors

**Solution:**
1. Verify backend is running: http://localhost:8000/api/
2. Check `VITE_API_BASE_URL` in `apps/web/.env`:
   ```bash
   # Should be:
   VITE_API_BASE_URL=http://localhost:8000
   ```
3. Restart frontend server after changing env vars

---

### Issue: Hot Module Replacement (HMR) not working

**Solution:**
```bash
# Clear Vite cache
cd apps/web
rm -rf .vite

# Restart dev server
pnpm dev
```

---

## Development Tips

### 💡 **Pro Tip #1: Use tmux or split terminals**

Instead of opening multiple terminals, use terminal multiplexer:

```bash
# Install tmux (macOS)
brew install tmux

# Start tmux session
tmux new -s plane

# Split panes:
# Ctrl+b then " (horizontal split)
# Ctrl+b then % (vertical split)
# Ctrl+b then arrow keys (navigate)

# Now run backend in one pane, frontend in another!
```

---

### 💡 **Pro Tip #2: Create startup scripts**

Create `dev.sh` in project root:

```bash
#!/bin/bash

# Start all services
echo "Starting Redis..."
redis-server &

echo "Starting PostgreSQL..."
# Add your OS-specific command

echo "Starting Backend..."
cd apps/api
source venv/bin/activate
python manage.py runserver &

echo "Starting Frontend..."
cd ../web
pnpm dev &

echo "All services started!"
```

Make it executable:
```bash
chmod +x dev.sh
./dev.sh
```

---

### 💡 **Pro Tip #3: VS Code Extensions**

Recommended extensions:
- **Python** (Microsoft) - Python language support
- **Pylance** - Python type checking
- **ESLint** - JavaScript linting
- **Prettier** - Code formatting
- **TypeScript Vue Plugin (Volar)** - Better TypeScript support
- **GitLens** - Git blame and history
- **Thunder Client** - API testing (like Postman)

---

## Next Steps

### 🎯 You're ready to explore!

Now that everything is running:

1. **Explore the Frontend:**
   - Log in with your superuser account
   - Create a workspace
   - Create a project
   - Create an issue
   - Watch the browser network tab - see API calls!

2. **Explore the Backend:**
   - Visit http://localhost:8000/api/
   - Browse the API endpoints
   - Try making API calls with the browsable interface
   - Watch your backend terminal for request logs

3. **Make a Small Change:**
   - Edit a React component in `apps/web/core/components/`
   - Save and watch HMR update the page!
   - Edit some text, change a color

4. **Read the Architecture:**
   - [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md) - Understand the big picture
   - [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md) - Navigate the codebase
   - [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md) - React patterns

---

## Quick Reference

### Common Commands

```bash
# Frontend (apps/web)
pnpm install          # Install dependencies
pnpm dev              # Start dev server
pnpm build            # Build for production
pnpm check:lint       # Run linter
pnpm check:types      # Type check

# Backend (apps/api)
source venv/bin/activate       # Activate virtual env
python manage.py runserver     # Start server
python manage.py migrate       # Run migrations
python manage.py makemigrations  # Create migrations
python manage.py shell         # Open Django shell
python manage.py test          # Run tests

# Database
psql -U plane_user -d plane_db  # Connect to database
python manage.py dbshell       # Django database shell

# Celery
celery -A plane worker -l info  # Start worker
celery -A plane beat -l info    # Start scheduler

# Git
git status                      # Check status
git pull                        # Pull latest changes
git checkout -b feature/my-feature  # Create branch
```

---

## Useful URLs

- **Frontend:** http://localhost:3000/
- **Backend API:** http://localhost:8000/api/
- **Admin Panel:** http://localhost:8000/admin/
- **API Docs (Swagger):** http://localhost:8000/api/schema/swagger-ui/
- **API Docs (Redoc):** http://localhost:8000/api/schema/redoc/

---

## Understanding What's Running

When you have everything running, here's what each service does:

```
Port 3000: Frontend (React Router + Vite)
├─ Serves the web app
├─ Hot Module Replacement (HMR)
└─ Proxies API requests to :8000

Port 8000: Backend (Django)
├─ REST API endpoints
├─ Handles authentication
├─ Database queries
└─ Business logic

Port 5432: PostgreSQL
├─ Stores all persistent data
├─ Tables for users, projects, issues, etc.
└─ ACID transactions

Port 6379: Redis
├─ Caches database queries
├─ Stores session data
├─ Message broker for Celery
└─ In-memory, very fast

Celery Worker (no port)
├─ Processes background tasks
├─ Sends emails
├─ Generates reports
└─ Scheduled jobs
```

---

## Architecture at a Glance

```
YOU (Browser)
    ↓
[Frontend: React @ localhost:3000]
    ↓ HTTP/HTTPS
[Backend: Django @ localhost:8000]
    ↓ SQL
[Database: PostgreSQL @ localhost:5432]

[Cache: Redis @ localhost:6379]
    ↑
[Celery Workers] ← Background tasks
```

**🧠 Mental Model:**

Think of it like a restaurant:
- **Frontend** = Dining room (what customers see)
- **Backend** = Kitchen (where food is prepared)
- **Database** = Pantry/Storage (ingredients and supplies)
- **Redis** = Chef's mise en place (quick-access prepped items)
- **Celery** = Delivery drivers (tasks that happen later)

---

## ✅ Checklist

Before moving on, make sure:

- [ ] Node.js 22.x installed and verified
- [ ] Python 3.12.x installed and verified
- [ ] pnpm 10.x installed and verified
- [ ] PostgreSQL running and database created
- [ ] Redis running
- [ ] Frontend dependencies installed (`pnpm install`)
- [ ] Backend dependencies installed (`pip install -r requirements.txt`)
- [ ] Environment files configured (`.env` in both frontend and backend)
- [ ] Database migrations run (`python manage.py migrate`)
- [ ] Superuser created (`python manage.py createsuperuser`)
- [ ] Frontend runs at http://localhost:3000/
- [ ] Backend runs at http://localhost:8000/api/
- [ ] Can log in to the app

---

## 🎉 Congratulations!

You've set up a complex, modern, full-stack application!

**What you've learned:**
- ✅ How to set up a monorepo project
- ✅ How frontend and backend services interact
- ✅ What each technology does (Node, Python, PostgreSQL, Redis)
- ✅ How to run multiple services simultaneously
- ✅ Basic troubleshooting skills

**You're now ready to:**
- Explore the codebase
- Make your first changes
- Understand the architecture
- Contribute to the project

---

## What's Next?

### Beginner Path
1. [ARCHITECTURE_OVERVIEW.md](./ARCHITECTURE_OVERVIEW.md) - Understand the big picture
2. [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md) - Make your first PR

### Intermediate Path
1. [PROJECT_STRUCTURE.md](./PROJECT_STRUCTURE.md) - Navigate the codebase
2. [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md) - React deep dive
3. [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md) - Code standards

### Advanced Path
1. [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md) - Learn Django
2. [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md) - Data modeling
3. [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md) - Full-stack development

---

**Happy Coding! 🚀**

[← Back to Learning Hub](./README.md) | [Next: Architecture Overview →](./ARCHITECTURE_OVERVIEW.md)
