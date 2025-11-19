# Security Guide

> **Last Updated:** November 2025
> **Status:** ✅ CURRENT - All patterns reflect React Router 7 + Django 4.2 best practices

A comprehensive security guide for Plane, covering OWASP Top 10 vulnerabilities, authentication best practices, and security patterns for React/Django applications.

---

## Table of Contents

1. [Security Overview](#security-overview)
2. [OWASP Top 10 for Plane](#owasp-top-10-for-plane)
3. [Authentication Security](#authentication-security)
4. [Authorization & Permissions](#authorization--permissions)
5. [Input Validation & Sanitization](#input-validation--sanitization)
6. [SQL Injection Prevention](#sql-injection-prevention)
7. [XSS Prevention](#xss-prevention)
8. [CSRF Protection](#csrf-protection)
9. [API Security](#api-security)
10. [Secret Management](#secret-management)
11. [Common Vulnerabilities](#common-vulnerabilities)
12. [Security Checklist](#security-checklist)

---

## Security Overview

### Security Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Frontend (React)                   │
│  • XSS Prevention (React auto-escaping)             │
│  • CSRF Token handling                               │
│  • Secure HTTP-only cookies                          │
│  • Content Security Policy                           │
└─────────────────┬───────────────────────────────────┘
                  │ HTTPS Only
                  │
┌─────────────────▼───────────────────────────────────┐
│              API Gateway / Nginx                     │
│  • Rate Limiting                                     │
│  • DDoS Protection                                   │
│  • TLS/SSL Termination                              │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│              Django Backend                          │
│  • JWT/Session Authentication                        │
│  • RBAC Authorization                                │
│  • ORM SQL Injection Protection                      │
│  • CSRF Middleware                                   │
│  • Input Validation                                  │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│              PostgreSQL Database                     │
│  • Encrypted at rest                                 │
│  • Parameterized queries only                        │
│  • Row-level security                                │
└─────────────────────────────────────────────────────┘
```

### Security Layers

1. **Frontend Security**: XSS prevention, CSRF tokens, secure storage
2. **Network Security**: HTTPS, rate limiting, CORS
3. **Application Security**: Authentication, authorization, input validation
4. **Data Security**: Encryption, SQL injection prevention, secure queries

---

## OWASP Top 10 for Plane

### 1. Broken Access Control (A01:2021)

**Vulnerability**: Users accessing resources they shouldn't have access to.

**Plane's Protection**:

```python
# ✅ CORRECT: Role-based access control in Django
# File: apps/api/plane/app/permissions.py

from rest_framework.permissions import BasePermission

class ProjectMemberPermission(BasePermission):
    def has_permission(self, request, view):
        # Check workspace membership
        workspace_slug = view.kwargs.get("slug")
        return WorkspaceMember.objects.filter(
            workspace__slug=workspace_slug,
            member=request.user,
            is_active=True
        ).exists()

    def has_object_permission(self, request, view, obj):
        # Check project-level permissions
        return ProjectMember.objects.filter(
            project=obj,
            member=request.user,
            is_active=True,
            role__in=[ROLE.ADMIN, ROLE.MEMBER]
        ).exists()
```

**Real Example from Plane**:

```python
# File: apps/api/plane/app/views/project/base.py

class ProjectViewSet(BaseViewSet):
    @allow_permission(allowed_roles=[ROLE.ADMIN, ROLE.MEMBER, ROLE.GUEST], level="WORKSPACE")
    def list(self, request, slug):
        # Only return projects user has access to
        projects = Project.objects.filter(
            workspace__slug=slug,
            project_projectmember__member=request.user,
            project_projectmember__is_active=True
        )
        return Response(ProjectListSerializer(projects, many=True).data)
```

**❌ DANGER - Never do this**:

```python
# ❌ WRONG: No permission check
def get_project(request, project_id):
    project = Project.objects.get(id=project_id)  # Any user can access!
    return Response(ProjectSerializer(project).data)

# ❌ WRONG: Trusting client-side data
def update_project(request, project_id):
    if request.data.get('is_admin'):  # Client can set this!
        # Dangerous operation
        pass
```

**Best Practices**:

- ✅ Always check permissions at the API level (never trust frontend)
- ✅ Use Django's permission system with custom permissions
- ✅ Implement row-level security for sensitive data
- ✅ Log access attempts for audit trails
- ✅ Use `@allow_permission` decorator consistently

---

### 2. Cryptographic Failures (A02:2021)

**Vulnerability**: Exposing sensitive data due to weak or missing encryption.

**Plane's Protection**:

```python
# ✅ CORRECT: Secure password hashing in Django
# File: apps/api/plane/db/models/user.py

from django.contrib.auth.models import AbstractBaseUser

class User(AbstractBaseUser):
    # Django automatically uses PBKDF2 with SHA256
    password = models.CharField(max_length=128)

    def set_password(self, raw_password):
        # Uses Django's make_password with strong hashing
        super().set_password(raw_password)
```

**Secret Management**:

```python
# ✅ CORRECT: Using environment variables
# File: apps/api/plane/settings/common.py

SECRET_KEY = os.environ.get("SECRET_KEY")
DATABASE_PASSWORD = os.environ.get("POSTGRES_PASSWORD")
AWS_SECRET_ACCESS_KEY = os.environ.get("AWS_SECRET_ACCESS_KEY")

# ❌ DANGER: Never hardcode secrets
# SECRET_KEY = "django-insecure-hardcoded-key-123"  # NEVER DO THIS!
```

**Secure Token Storage (Frontend)**:

```typescript
// ✅ CORRECT: HTTP-only cookies (set by backend)
// Tokens stored in HTTP-only cookies are inaccessible to JavaScript

// ❌ DANGER: Never store tokens in localStorage
// localStorage.setItem('token', jwt);  // Vulnerable to XSS!

// ✅ CORRECT: Access tokens via secure cookies
const response = await fetch('/api/users/me', {
  credentials: 'include',  // Include HTTP-only cookies
});
```

**HTTPS Enforcement**:

```python
# ✅ CORRECT: Force HTTPS in production
# File: apps/api/plane/settings/production.py

SECURE_SSL_REDIRECT = True
SECURE_HSTS_SECONDS = 31536000  # 1 year
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

**Best Practices**:

- ✅ Use HTTPS everywhere in production
- ✅ Store passwords with bcrypt/PBKDF2 (Django default)
- ✅ Use HTTP-only cookies for tokens
- ✅ Encrypt sensitive data at rest
- ✅ Rotate secrets regularly
- ✅ Use strong encryption algorithms (AES-256, RSA-2048+)

---

### 3. Injection (A03:2021)

**Vulnerability**: SQL injection, NoSQL injection, command injection.

**Plane's Protection (SQL Injection)**:

```python
# ✅ CORRECT: Django ORM parameterized queries
# File: apps/api/plane/app/views/issue/base.py

def get_issues(request, workspace_slug, project_id):
    # Django ORM automatically parameterizes queries
    issues = Issue.objects.filter(
        project__workspace__slug=workspace_slug,
        project_id=project_id,
        name__icontains=request.GET.get('search', '')  # Safe!
    )
    return Response(IssueSerializer(issues, many=True).data)

# ❌ DANGER: Raw SQL without parameterization
def get_issues_unsafe(request, project_id):
    query = f"SELECT * FROM issues WHERE project_id = {project_id}"  # SQL INJECTION!
    cursor.execute(query)  # NEVER DO THIS!

# ✅ CORRECT: If you must use raw SQL, parameterize it
def get_issues_raw_safe(request, project_id):
    query = "SELECT * FROM issues WHERE project_id = %s"
    cursor.execute(query, [project_id])  # Safe - parameterized
```

**Real Example from Plane**:

```python
# File: apps/api/plane/app/views/issue/base.py

class IssueViewSet(BaseViewSet):
    def list(self, request, slug, project_id):
        # ✅ CORRECT: Using Django ORM filters
        filters = {}

        # Filter by priority
        if request.GET.get('priority'):
            filters['priority__in'] = request.GET.get('priority').split(',')

        # Filter by assignee
        if request.GET.get('assignees'):
            filters['assignees__in'] = request.GET.get('assignees').split(',')

        # Safe - Django parameterizes everything
        issues = Issue.objects.filter(**filters)
        return Response(IssueSerializer(issues, many=True).data)
```

**Command Injection Prevention**:

```python
# ❌ DANGER: Command injection via shell
import os

def export_data(request, filename):
    # NEVER use user input in shell commands!
    os.system(f"export_tool --file {filename}")  # COMMAND INJECTION!

# ✅ CORRECT: Use subprocess with argument list
import subprocess

def export_data_safe(request, filename):
    # Validate filename first
    if not filename.isalnum():
        return Response({"error": "Invalid filename"}, status=400)

    # Use argument list (not shell=True)
    subprocess.run(
        ['export_tool', '--file', filename],
        shell=False,  # Important!
        check=True
    )
```

**Best Practices**:

- ✅ Always use Django ORM (never raw SQL)
- ✅ If raw SQL is necessary, use parameterized queries
- ✅ Never concatenate user input into queries
- ✅ Validate and sanitize all input
- ✅ Use subprocess with argument lists (not shell commands)
- ✅ Escape special characters in user input

---

### 4. Insecure Design (A04:2021)

**Vulnerability**: Missing or ineffective security controls by design.

**Plane's Secure Design Patterns**:

```python
# ✅ CORRECT: Rate limiting on authentication endpoints
# File: apps/api/plane/authentication/rate_limit.py

from rest_framework.throttling import AnonRateThrottle

class AuthenticationThrottle(AnonRateThrottle):
    rate = "30/minute"  # Prevent brute force attacks
    scope = "authentication"

    def throttle_failure_view(self, request, *args, **kwargs):
        raise AuthenticationException(
            error_code="RATE_LIMIT_EXCEEDED",
            error_message="Too many authentication attempts"
        )

# Apply to authentication views
class SignInAuthEndpoint(View):
    throttle_classes = [AuthenticationThrottle]
```

**Secure Password Reset Flow**:

```python
# ✅ CORRECT: Time-limited password reset tokens
# File: apps/api/plane/authentication/views/app/password_management.py

def request_password_reset(request):
    email = request.data.get('email')
    user = User.objects.filter(email=email).first()

    if user:
        # Generate secure token with expiry
        token = generate_reset_token(user)  # Expires in 1 hour
        send_password_reset_email(user, token)

    # Always return success (don't leak user existence)
    return Response({"message": "If email exists, reset link sent"})

def reset_password(request, token):
    # Validate token hasn't expired
    user = validate_reset_token(token)
    if not user:
        return Response({"error": "Invalid or expired token"}, status=400)

    # Validate new password strength
    password = request.data.get('password')
    if len(password) < 8:
        return Response({"error": "Password too weak"}, status=400)

    user.set_password(password)
    user.save()

    # Invalidate all sessions
    Session.objects.filter(user=user).delete()
```

**Secure Session Management**:

```python
# ✅ CORRECT: Session security settings
# File: apps/api/plane/settings/common.py

SESSION_COOKIE_HTTPONLY = True  # Prevent XSS access
SESSION_COOKIE_SECURE = True     # HTTPS only
SESSION_COOKIE_SAMESITE = 'Lax'  # CSRF protection
SESSION_EXPIRE_AT_BROWSER_CLOSE = False
SESSION_COOKIE_AGE = 86400       # 24 hours

# Track session security
class Session(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    ip_address = models.CharField(max_length=255)
    user_agent = models.TextField()
    last_activity = models.DateTimeField(auto_now=True)
    expires_at = models.DateTimeField()
```

**Best Practices**:

- ✅ Implement rate limiting on all public endpoints
- ✅ Use time-limited tokens for sensitive operations
- ✅ Never leak information about user existence
- ✅ Enforce strong password policies
- ✅ Implement account lockout after failed attempts
- ✅ Use secure session management
- ✅ Design APIs to fail securely

---

### 5. Security Misconfiguration (A05:2021)

**Vulnerability**: Insecure default configurations, unnecessary features enabled.

**Plane's Secure Configuration**:

```python
# ✅ CORRECT: Production security settings
# File: apps/api/plane/settings/production.py

DEBUG = False  # NEVER True in production!

ALLOWED_HOSTS = [
    'plane.so',
    '.plane.so',  # Allow subdomains
]

# Security headers
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'

# CORS configuration
CORS_ALLOWED_ORIGINS = [
    "https://app.plane.so",
]
CORS_ALLOW_CREDENTIALS = True

# Remove sensitive headers
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

# ❌ DANGER: Development settings in production
# DEBUG = True  # Exposes stack traces!
# ALLOWED_HOSTS = ['*']  # Allows host header injection!
# CORS_ALLOW_ALL_ORIGINS = True  # Allows any origin!
```

**Secure Error Handling**:

```python
# ✅ CORRECT: Generic error messages for users
# File: apps/api/plane/app/views/base.py

class BaseAPIView(APIView):
    def handle_exception(self, exc):
        # Log detailed error internally
        logger.error(f"API Error: {exc}", exc_info=True)

        # Return generic message to user
        if isinstance(exc, ValidationError):
            return Response(
                {"error": "Invalid input provided"},
                status=status.HTTP_400_BAD_REQUEST
            )

        # Don't expose internal details
        return Response(
            {"error": "An error occurred processing your request"},
            status=status.HTTP_500_INTERNAL_SERVER_ERROR
        )

# ❌ DANGER: Exposing detailed errors
def unsafe_view(request):
    try:
        # some operation
        pass
    except Exception as e:
        # Never send detailed errors to client!
        return Response({"error": str(e), "stack": traceback.format_exc()})
```

**Security Headers**:

```python
# ✅ CORRECT: Security middleware
# File: apps/api/plane/settings/common.py

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',  # Security headers
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',      # CSRF protection
    'django.middleware.clickjacking.XFrameOptionsMiddleware',  # Clickjacking
    # ... other middleware
]

# Content Security Policy (in nginx or middleware)
CSP_DEFAULT_SRC = ["'self'"]
CSP_SCRIPT_SRC = ["'self'", "'unsafe-inline'"]  # Be restrictive!
CSP_STYLE_SRC = ["'self'", "'unsafe-inline'"]
CSP_IMG_SRC = ["'self'", "data:", "https:"]
```

**Best Practices**:

- ✅ Disable DEBUG in production
- ✅ Restrict ALLOWED_HOSTS
- ✅ Configure CORS properly (no wildcards)
- ✅ Remove unnecessary features and endpoints
- ✅ Use security headers
- ✅ Implement Content Security Policy
- ✅ Keep dependencies updated
- ✅ Regular security audits

---

### 6. Vulnerable and Outdated Components (A06:2021)

**Vulnerability**: Using libraries with known security vulnerabilities.

**Plane's Dependency Management**:

```bash
# ✅ CORRECT: Regular dependency updates

# Backend (Python)
pip install --upgrade pip
pip list --outdated
pip-audit  # Check for known vulnerabilities

# Frontend (JavaScript)
npm audit
npm audit fix
npm outdated
```

**Lock Files**:

```bash
# ✅ CORRECT: Use lock files for reproducible builds
# Backend: requirements.txt with exact versions
Django==4.2.7
djangorestframework==3.15.1
psycopg2-binary==2.9.9

# Frontend: package-lock.json (auto-generated)
# Commit this to git!
```

**Automated Security Scanning**:

```yaml
# ✅ CORRECT: GitHub Actions security scan
# File: .github/workflows/security.yml

name: Security Scan
on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      # Python security scan
      - name: Run Safety
        run: |
          pip install safety
          safety check --file requirements.txt

      # JavaScript security scan
      - name: Run npm audit
        run: npm audit --production

      # SAST scanning
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
```

**Best Practices**:

- ✅ Run `npm audit` and `pip-audit` regularly
- ✅ Update dependencies monthly (security patches immediately)
- ✅ Use Dependabot or Renovate for automated PRs
- ✅ Review security advisories for your stack
- ✅ Pin exact versions in production
- ✅ Test updates in staging before production

---

### 7. Identification and Authentication Failures (A07:2021)

**Vulnerability**: Weak authentication allowing unauthorized access.

**Plane's Authentication Implementation**:

```python
# ✅ CORRECT: Secure email/password authentication
# File: apps/api/plane/authentication/views/app/email.py

class SignInAuthEndpoint(View):
    throttle_classes = [AuthenticationThrottle]  # Rate limiting

    def post(self, request):
        email = request.POST.get('email', '').strip().lower()
        password = request.POST.get('password', '')

        # Validate email format
        try:
            validate_email(email)
        except ValidationError:
            raise AuthenticationException(
                error_code="INVALID_EMAIL_SIGN_IN",
                error_message="Invalid email format"
            )

        # Check user exists
        user = User.objects.filter(email=email).first()
        if not user:
            # Use same error as wrong password (don't leak existence)
            raise AuthenticationException(
                error_code="INVALID_CREDENTIALS",
                error_message="Invalid email or password"
            )

        # Verify password
        if not user.check_password(password):
            # Track failed attempts
            track_failed_login(user)
            raise AuthenticationException(
                error_code="INVALID_CREDENTIALS",
                error_message="Invalid email or password"
            )

        # Check account status
        if not user.is_active:
            raise AuthenticationException(
                error_code="ACCOUNT_DEACTIVATED",
                error_message="Account is deactivated"
            )

        # Create secure session
        session = create_secure_session(user, request)

        return Response({"user": UserSerializer(user).data})
```

**Multi-Factor Authentication (MFA)**:

```python
# ✅ CORRECT: TOTP-based MFA
import pyotp

class MFASetupView(APIView):
    def post(self, request):
        # Generate secret for user
        secret = pyotp.random_base32()

        # Store encrypted secret
        user = request.user
        user.mfa_secret = encrypt(secret)
        user.save()

        # Generate QR code data
        totp = pyotp.TOTP(secret)
        provisioning_uri = totp.provisioning_uri(
            name=user.email,
            issuer_name='Plane'
        )

        return Response({
            "secret": secret,
            "qr_code_uri": provisioning_uri
        })

class MFAVerifyView(APIView):
    def post(self, request):
        user = request.user
        code = request.data.get('code')

        # Decrypt and verify TOTP code
        secret = decrypt(user.mfa_secret)
        totp = pyotp.TOTP(secret)

        if totp.verify(code, valid_window=1):  # Allow 30s window
            user.mfa_enabled = True
            user.save()
            return Response({"status": "MFA enabled"})

        return Response(
            {"error": "Invalid code"},
            status=status.HTTP_400_BAD_REQUEST
        )
```

**OAuth 2.0 Integration**:

```python
# ✅ CORRECT: Secure OAuth flow
# File: apps/api/plane/authentication/provider/oauth/google.py

class GoogleOAuthProvider:
    def authenticate(self, code, state):
        # Verify state parameter (CSRF protection)
        if not verify_state(state):
            raise AuthenticationException("Invalid state parameter")

        # Exchange code for token
        token_response = requests.post(
            'https://oauth2.googleapis.com/token',
            data={
                'code': code,
                'client_id': settings.GOOGLE_CLIENT_ID,
                'client_secret': settings.GOOGLE_CLIENT_SECRET,
                'redirect_uri': settings.GOOGLE_REDIRECT_URI,
                'grant_type': 'authorization_code'
            }
        )

        if token_response.status_code != 200:
            raise AuthenticationException("OAuth token exchange failed")

        # Verify ID token
        id_token = token_response.json()['id_token']
        user_info = verify_google_id_token(id_token)

        # Get or create user
        user, created = User.objects.get_or_create(
            email=user_info['email'],
            defaults={
                'username': user_info['email'],
                'display_name': user_info['name'],
                'avatar': user_info['picture'],
                'is_email_verified': True
            }
        )

        return user
```

**Session Security**:

```python
# ✅ CORRECT: Secure session management
# File: apps/api/plane/authentication/session.py

def create_secure_session(user, request):
    # Create session with security metadata
    session = Session.objects.create(
        user=user,
        ip_address=get_client_ip(request),
        user_agent=request.META.get('HTTP_USER_AGENT', ''),
        expires_at=timezone.now() + timedelta(days=7)
    )

    # Set secure cookie
    response.set_cookie(
        'session_id',
        session.id,
        max_age=60*60*24*7,  # 7 days
        httponly=True,       # Prevent XSS
        secure=True,         # HTTPS only
        samesite='Lax'       # CSRF protection
    )

    return session

def validate_session(request):
    session_id = request.COOKIES.get('session_id')
    if not session_id:
        raise AuthenticationException("No session")

    session = Session.objects.filter(
        id=session_id,
        expires_at__gt=timezone.now()
    ).first()

    if not session:
        raise AuthenticationException("Invalid or expired session")

    # Check for session hijacking
    if session.ip_address != get_client_ip(request):
        # IP changed - potential hijacking
        logger.warning(f"Session IP mismatch for user {session.user.id}")
        # Optionally require re-authentication

    # Update last activity
    session.last_activity = timezone.now()
    session.save()

    return session.user
```

**Best Practices**:

- ✅ Use strong password hashing (Django's default PBKDF2)
- ✅ Implement rate limiting on auth endpoints
- ✅ Use same error message for invalid user/password
- ✅ Implement MFA for sensitive accounts
- ✅ Use secure session management
- ✅ Track and alert on suspicious auth activity
- ✅ Implement account lockout after failed attempts
- ✅ Use HTTPS-only, HTTP-only cookies

---

### 8. Software and Data Integrity Failures (A08:2021)

**Vulnerability**: Using untrusted sources, insecure CI/CD pipelines.

**Plane's Integrity Protection**:

```python
# ✅ CORRECT: Verify file uploads
# File: apps/api/plane/app/views/asset/base.py

from django.core.files.uploadedfile import UploadedFile
import magic

class AssetUploadView(APIView):
    def post(self, request):
        file = request.FILES.get('file')
        if not file:
            return Response({"error": "No file provided"}, status=400)

        # Verify file size
        max_size = 5 * 1024 * 1024  # 5MB
        if file.size > max_size:
            return Response({"error": "File too large"}, status=400)

        # Verify MIME type (don't trust client!)
        mime = magic.from_buffer(file.read(1024), mime=True)
        file.seek(0)  # Reset file pointer

        allowed_types = ['image/png', 'image/jpeg', 'image/gif', 'application/pdf']
        if mime not in allowed_types:
            return Response({"error": "Invalid file type"}, status=400)

        # Sanitize filename
        filename = secure_filename(file.name)

        # Scan for malware (if configured)
        if settings.MALWARE_SCANNING_ENABLED:
            if not scan_file(file):
                return Response({"error": "File failed security scan"}, status=400)

        # Upload to secure storage
        asset = Asset.objects.create(
            file=file,
            filename=filename,
            mime_type=mime,
            size=file.size,
            uploaded_by=request.user
        )

        return Response(AssetSerializer(asset).data)
```

**Secure Package Management**:

```json
// ✅ CORRECT: Lock file with integrity hashes
// File: package-lock.json

{
  "dependencies": {
    "react": {
      "version": "18.3.1",
      "resolved": "https://registry.npmjs.org/react/-/react-18.3.1.tgz",
      "integrity": "sha512-wS+hAgJShR0KhEvPJArfuPVN1+Hz1t0Y6n5jLrGQbkb4urgPE/0Rve+1kMB1v/oWgHgm4WIcV+i7F2pTVj+2iQ=="
    }
  }
}
```

**CI/CD Security**:

```yaml
# ✅ CORRECT: Secure GitHub Actions workflow
# File: .github/workflows/deploy.yml

name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    # Use specific versions (not @latest)
    steps:
      - uses: actions/checkout@v4.1.0

      # Verify dependencies
      - name: Verify dependencies
        run: |
          npm ci  # Use ci, not install (verifies lock file)
          npm audit --production --audit-level=high

      # Run tests
      - name: Run tests
        run: npm test

      # Build with integrity checks
      - name: Build
        run: npm run build

      # Deploy with secrets from vault
      - name: Deploy
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          # Deploy only if tests pass
          ./deploy.sh
```

**Best Practices**:

- ✅ Use package lock files (package-lock.json, requirements.txt)
- ✅ Verify file uploads (type, size, content)
- ✅ Use checksums/hashes for integrity verification
- ✅ Implement malware scanning for uploads
- ✅ Secure CI/CD pipelines
- ✅ Use signed commits for critical repos
- ✅ Regularly audit dependencies

---

### 9. Security Logging and Monitoring Failures (A09:2021)

**Vulnerability**: Insufficient logging prevents detection of breaches.

**Plane's Security Logging**:

```python
# ✅ CORRECT: Comprehensive security logging
# File: apps/api/plane/utils/security_logger.py

import logging
from django.utils import timezone

security_logger = logging.getLogger('security')

class SecurityEventLogger:
    @staticmethod
    def log_authentication_attempt(user_email, success, ip_address, user_agent):
        security_logger.info(
            f"Authentication attempt",
            extra={
                'event_type': 'authentication',
                'user_email': user_email,
                'success': success,
                'ip_address': ip_address,
                'user_agent': user_agent,
                'timestamp': timezone.now().isoformat()
            }
        )

    @staticmethod
    def log_permission_denied(user, resource, action, reason):
        security_logger.warning(
            f"Permission denied",
            extra={
                'event_type': 'permission_denied',
                'user_id': user.id,
                'user_email': user.email,
                'resource': resource,
                'action': action,
                'reason': reason,
                'timestamp': timezone.now().isoformat()
            }
        )

    @staticmethod
    def log_suspicious_activity(user, activity_type, details):
        security_logger.warning(
            f"Suspicious activity detected",
            extra={
                'event_type': 'suspicious_activity',
                'user_id': user.id,
                'activity_type': activity_type,
                'details': details,
                'timestamp': timezone.now().isoformat()
            }
        )

    @staticmethod
    def log_data_access(user, model, object_id, action):
        security_logger.info(
            f"Data access",
            extra={
                'event_type': 'data_access',
                'user_id': user.id,
                'model': model,
                'object_id': object_id,
                'action': action,
                'timestamp': timezone.now().isoformat()
            }
        )
```

**Audit Trail Implementation**:

```python
# ✅ CORRECT: Audit trail for sensitive operations
# File: apps/api/plane/db/models/base.py

class AuditedModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, related_name='+')
    updated_at = models.DateTimeField(auto_now=True)
    updated_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, related_name='+')

    class Meta:
        abstract = True

# Track all changes to sensitive models
class AuditLog(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    action = models.CharField(max_length=50)  # CREATE, UPDATE, DELETE
    model_name = models.CharField(max_length=100)
    object_id = models.UUIDField()
    changes = models.JSONField()  # Old and new values
    ip_address = models.CharField(max_length=45)
    user_agent = models.TextField()
    timestamp = models.DateTimeField(auto_now_add=True)

    class Meta:
        db_table = 'audit_logs'
        indexes = [
            models.Index(fields=['user', 'timestamp']),
            models.Index(fields=['model_name', 'object_id']),
        ]
```

**Security Monitoring**:

```python
# ✅ CORRECT: Detect suspicious patterns
# File: apps/api/plane/utils/security_monitoring.py

from django.core.cache import cache

class SecurityMonitor:
    @staticmethod
    def check_failed_login_attempts(user_email, ip_address):
        # Track failed attempts per email
        email_key = f"failed_login:email:{user_email}"
        email_attempts = cache.get(email_key, 0)

        if email_attempts >= 5:
            # Lock account temporarily
            SecurityEventLogger.log_suspicious_activity(
                None,
                'account_lockout',
                f'Too many failed attempts for {user_email}'
            )
            raise AuthenticationException("Account temporarily locked")

        # Track failed attempts per IP
        ip_key = f"failed_login:ip:{ip_address}"
        ip_attempts = cache.get(ip_key, 0)

        if ip_attempts >= 10:
            SecurityEventLogger.log_suspicious_activity(
                None,
                'ip_lockout',
                f'Too many failed attempts from {ip_address}'
            )
            raise AuthenticationException("Too many failed attempts")

    @staticmethod
    def detect_session_hijacking(session, request):
        current_ip = get_client_ip(request)

        # Check for IP change
        if session.ip_address != current_ip:
            # Check if IPs are in same network (allow mobile switching)
            if not are_ips_in_same_network(session.ip_address, current_ip):
                SecurityEventLogger.log_suspicious_activity(
                    session.user,
                    'session_hijacking_attempt',
                    f'IP changed from {session.ip_address} to {current_ip}'
                )
                # Invalidate session and require re-authentication
                session.delete()
                raise AuthenticationException("Session invalidated for security")

    @staticmethod
    def detect_unusual_api_usage(user, endpoint, request):
        # Track API call rate per user
        rate_key = f"api_rate:{user.id}:{endpoint}"
        calls = cache.get(rate_key, 0)

        if calls > 100:  # 100 calls per minute is suspicious
            SecurityEventLogger.log_suspicious_activity(
                user,
                'unusual_api_usage',
                f'High API call rate for {endpoint}'
            )
            # Consider rate limiting or CAPTCHA
```

**Log Configuration**:

```python
# ✅ CORRECT: Structured logging configuration
# File: apps/api/plane/settings/common.py

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {message}',
            'style': '{',
        },
        'json': {
            '()': 'pythonjsonlogger.jsonlogger.JsonFormatter',
            'format': '%(asctime)s %(levelname)s %(name)s %(message)s'
        }
    },
    'handlers': {
        'security_file': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': '/var/log/plane/security.log',
            'maxBytes': 10485760,  # 10MB
            'backupCount': 10,
            'formatter': 'json'
        },
        'siem': {
            'level': 'WARNING',
            'class': 'logging.handlers.SysLogHandler',
            'address': ('siem.company.com', 514),
            'formatter': 'json'
        }
    },
    'loggers': {
        'security': {
            'handlers': ['security_file', 'siem'],
            'level': 'INFO',
            'propagate': False,
        }
    }
}
```

**Best Practices**:

- ✅ Log all authentication attempts (success and failure)
- ✅ Log permission denied events
- ✅ Log sensitive data access
- ✅ Use structured logging (JSON format)
- ✅ Send security logs to SIEM system
- ✅ Set up alerts for suspicious patterns
- ✅ Implement audit trails for compliance
- ✅ Never log sensitive data (passwords, tokens)
- ✅ Rotate logs regularly
- ✅ Monitor logs in real-time

---

### 10. Server-Side Request Forgery (SSRF) (A10:2021)

**Vulnerability**: Attacker makes server fetch malicious URLs.

**Plane's SSRF Protection**:

```python
# ✅ CORRECT: Validate and restrict URLs
# File: apps/api/plane/utils/url_validator.py

from urllib.parse import urlparse
import ipaddress
import requests

class SSRFProtection:
    BLOCKED_SCHEMES = ['file', 'gopher', 'data', 'dict']
    BLOCKED_DOMAINS = ['169.254.169.254', 'metadata.google.internal']  # Cloud metadata

    @staticmethod
    def is_safe_url(url):
        try:
            parsed = urlparse(url)

            # Block dangerous schemes
            if parsed.scheme in SSRFProtection.BLOCKED_SCHEMES:
                return False

            # Only allow http/https
            if parsed.scheme not in ['http', 'https']:
                return False

            # Resolve hostname
            hostname = parsed.hostname
            if not hostname:
                return False

            # Block private IPs
            try:
                ip = ipaddress.ip_address(hostname)
                if ip.is_private or ip.is_loopback or ip.is_link_local:
                    return False
            except ValueError:
                # Not an IP, check DNS
                import socket
                try:
                    ip = socket.gethostbyname(hostname)
                    ip_obj = ipaddress.ip_address(ip)
                    if ip_obj.is_private or ip_obj.is_loopback:
                        return False
                except socket.gaierror:
                    return False

            # Block cloud metadata endpoints
            if hostname in SSRFProtection.BLOCKED_DOMAINS:
                return False

            return True
        except Exception:
            return False

    @staticmethod
    def fetch_url(url, timeout=5):
        if not SSRFProtection.is_safe_url(url):
            raise ValueError("URL blocked by SSRF protection")

        # Use timeout to prevent hanging
        response = requests.get(
            url,
            timeout=timeout,
            allow_redirects=False  # Don't follow redirects
        )

        return response

# ✅ CORRECT: Webhook URL validation
# File: apps/api/plane/app/views/webhook/base.py

class WebhookView(APIView):
    def create(self, request, slug):
        url = request.data.get('url')

        # Validate URL before saving
        if not SSRFProtection.is_safe_url(url):
            return Response(
                {"error": "Invalid webhook URL"},
                status=status.HTTP_400_BAD_REQUEST
            )

        webhook = Webhook.objects.create(
            workspace_id=slug,
            url=url,
            events=request.data.get('events', [])
        )

        return Response(WebhookSerializer(webhook).data)

# ❌ DANGER: No URL validation
def unsafe_webhook_create(request):
    url = request.data.get('url')
    # Attacker can use: file:///etc/passwd or http://169.254.169.254/
    requests.get(url)  # SSRF VULNERABILITY!
```

**Link Preview Protection**:

```python
# ✅ CORRECT: Safe link preview fetching
# File: apps/api/plane/app/views/issue/link_preview.py

class LinkPreviewView(APIView):
    def post(self, request):
        url = request.data.get('url')

        # Validate URL
        if not SSRFProtection.is_safe_url(url):
            return Response(
                {"error": "Invalid URL for preview"},
                status=status.HTTP_400_BAD_REQUEST
            )

        try:
            # Fetch with timeout and size limit
            response = requests.get(
                url,
                timeout=5,
                stream=True,
                headers={'User-Agent': 'Plane-Bot/1.0'}
            )

            # Limit response size
            max_size = 1024 * 1024  # 1MB
            content = b''
            for chunk in response.iter_content(chunk_size=8192):
                content += chunk
                if len(content) > max_size:
                    raise ValueError("Response too large")

            # Parse HTML and extract metadata
            from bs4 import BeautifulSoup
            soup = BeautifulSoup(content, 'html.parser')

            preview = {
                'title': soup.find('meta', property='og:title')['content'] if soup.find('meta', property='og:title') else None,
                'description': soup.find('meta', property='og:description')['content'] if soup.find('meta', property='og:description') else None,
                'image': soup.find('meta', property='og:image')['content'] if soup.find('meta', property='og:image') else None,
            }

            return Response(preview)

        except Exception as e:
            logger.error(f"Link preview error: {e}")
            return Response(
                {"error": "Failed to fetch link preview"},
                status=status.HTTP_400_BAD_REQUEST
            )
```

**Best Practices**:

- ✅ Validate all user-provided URLs
- ✅ Block private IP ranges (10.0.0.0/8, 192.168.0.0/16, 127.0.0.0/8)
- ✅ Block cloud metadata endpoints (169.254.169.254)
- ✅ Only allow http/https schemes
- ✅ Don't follow redirects blindly
- ✅ Use timeouts for external requests
- ✅ Limit response sizes
- ✅ Maintain a blocklist of dangerous domains

---

## Authentication Security

### JWT Best Practices

```python
# ✅ CORRECT: Secure JWT configuration
# File: apps/api/plane/settings/common.py

import datetime

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': datetime.timedelta(minutes=15),  # Short-lived
    'REFRESH_TOKEN_LIFETIME': datetime.timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,  # Issue new refresh token each time
    'BLACKLIST_AFTER_ROTATION': True,  # Blacklist old refresh tokens
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': settings.SECRET_KEY,
    'VERIFYING_KEY': None,
    'AUTH_HEADER_TYPES': ('Bearer',),
    'USER_ID_FIELD': 'id',
    'USER_ID_CLAIM': 'user_id',
    'AUTH_TOKEN_CLASSES': ('rest_framework_simplejwt.tokens.AccessToken',),
    'TOKEN_TYPE_CLAIM': 'token_type',
}

# ❌ DANGER: Insecure JWT settings
# 'ACCESS_TOKEN_LIFETIME': datetime.timedelta(days=365),  # Too long!
# 'ROTATE_REFRESH_TOKENS': False,  # Reuse refresh tokens - insecure!
```

### Session vs JWT

| Feature | Session-based | JWT |
|---------|--------------|-----|
| **State** | Server-side | Stateless |
| **Scalability** | Requires sticky sessions | Easily scalable |
| **Revocation** | Easy (delete session) | Requires blacklist |
| **Size** | Small cookie | Larger token |
| **Security** | HTTP-only cookie | Bearer token |
| **Use Case** | Web apps | APIs, mobile apps |

**Plane uses both:**
- Session-based for web app (HTTP-only cookies)
- JWT for mobile/API access

---

## Authorization & Permissions

### Role-Based Access Control (RBAC)

```python
# ✅ CORRECT: RBAC implementation
# File: apps/api/plane/db/models/project.py

from enum import Enum

class ROLE(Enum):
    ADMIN = 20
    MEMBER = 15
    GUEST = 5

class ProjectMember(models.Model):
    project = models.ForeignKey(Project, on_delete=models.CASCADE)
    member = models.ForeignKey(User, on_delete=models.CASCADE)
    role = models.IntegerField(choices=ROLE_CHOICES)
    is_active = models.BooleanField(default=True)

# Permission decorator
def allow_permission(allowed_roles, level="PROJECT"):
    def decorator(view_func):
        def wrapper(self, request, *args, **kwargs):
            user = request.user

            if level == "WORKSPACE":
                workspace_slug = kwargs.get('slug')
                member = WorkspaceMember.objects.filter(
                    workspace__slug=workspace_slug,
                    member=user,
                    is_active=True
                ).first()
            elif level == "PROJECT":
                project_id = kwargs.get('project_id')
                member = ProjectMember.objects.filter(
                    project_id=project_id,
                    member=user,
                    is_active=True
                ).first()

            if not member:
                return Response(
                    {"error": "You don't have access to this resource"},
                    status=status.HTTP_403_FORBIDDEN
                )

            if member.role not in [role.value for role in allowed_roles]:
                return Response(
                    {"error": "You don't have permission for this action"},
                    status=status.HTTP_403_FORBIDDEN
                )

            return view_func(self, request, *args, **kwargs)

        return wrapper
    return decorator

# Usage
class IssueViewSet(BaseViewSet):
    @allow_permission(allowed_roles=[ROLE.ADMIN, ROLE.MEMBER], level="PROJECT")
    def create(self, request, slug, project_id):
        # Only admins and members can create issues
        pass

    @allow_permission(allowed_roles=[ROLE.ADMIN, ROLE.MEMBER, ROLE.GUEST], level="PROJECT")
    def list(self, request, slug, project_id):
        # All members can view issues
        pass

    @allow_permission(allowed_roles=[ROLE.ADMIN], level="PROJECT")
    def destroy(self, request, slug, project_id, pk):
        # Only admins can delete issues
        pass
```

### Permission Matrix

| Action | Guest | Member | Admin |
|--------|-------|--------|-------|
| View issues | ✅ | ✅ | ✅ |
| Create issues | ❌ | ✅ | ✅ |
| Edit issues | ❌ | ✅ | ✅ |
| Delete issues | ❌ | ❌ | ✅ |
| Manage members | ❌ | ❌ | ✅ |
| Project settings | ❌ | ❌ | ✅ |

---

## Input Validation & Sanitization

### Django Form Validation

```python
# ✅ CORRECT: Comprehensive input validation
# File: apps/api/plane/app/serializers/issue.py

from rest_framework import serializers
from django.core.validators import MinLengthValidator, MaxLengthValidator

class IssueSerializer(serializers.ModelSerializer):
    # Name validation
    name = serializers.CharField(
        min_length=1,
        max_length=255,
        required=True,
        error_messages={
            'required': 'Issue name is required',
            'max_length': 'Issue name cannot exceed 255 characters',
            'blank': 'Issue name cannot be blank'
        }
    )

    # Priority validation
    priority = serializers.ChoiceField(
        choices=['urgent', 'high', 'medium', 'low', 'none'],
        default='none'
    )

    # Date validation
    start_date = serializers.DateField(required=False, allow_null=True)
    target_date = serializers.DateField(required=False, allow_null=True)

    # URL validation
    issue_link = serializers.URLField(
        required=False,
        allow_blank=True,
        validators=[SSRFValidator()]  # Custom SSRF validation
    )

    class Meta:
        model = Issue
        fields = '__all__'

    def validate(self, data):
        # Cross-field validation
        if data.get('start_date') and data.get('target_date'):
            if data['start_date'] > data['target_date']:
                raise serializers.ValidationError({
                    'target_date': 'Target date must be after start date'
                })

        return data

    def validate_name(self, value):
        # Custom name validation
        if value.strip() == '':
            raise serializers.ValidationError('Issue name cannot be empty')

        # Check for malicious patterns
        if '<script>' in value.lower():
            raise serializers.ValidationError('Invalid characters in name')

        return value.strip()

# ❌ DANGER: No validation
class UnsafeIssueSerializer(serializers.ModelSerializer):
    class Meta:
        model = Issue
        fields = '__all__'

    # Accepts anything! SQL injection, XSS, etc.
```

### HTML Sanitization

```python
# ✅ CORRECT: Sanitize HTML content
# File: apps/api/plane/utils/html_processor.py

from bs4 import BeautifulSoup
import bleach

def sanitize_html(html_content):
    """
    Sanitize HTML to prevent XSS attacks
    """
    # Allowed tags and attributes
    allowed_tags = [
        'p', 'br', 'strong', 'em', 'u', 'a', 'ul', 'ol', 'li',
        'h1', 'h2', 'h3', 'h4', 'h5', 'h6', 'blockquote',
        'code', 'pre', 'img'
    ]

    allowed_attributes = {
        'a': ['href', 'title'],
        'img': ['src', 'alt', 'title'],
        '*': ['class']
    }

    # Clean HTML
    clean = bleach.clean(
        html_content,
        tags=allowed_tags,
        attributes=allowed_attributes,
        strip=True  # Remove disallowed tags
    )

    # Additional sanitization
    clean = bleach.linkify(clean)  # Convert URLs to links safely

    return clean

def strip_tags(html_content):
    """
    Remove all HTML tags
    """
    soup = BeautifulSoup(html_content, 'html.parser')
    return soup.get_text()

# Usage in models
class Issue(models.Model):
    description_html = models.TextField()
    description_stripped = models.TextField()

    def save(self, *args, **kwargs):
        # Sanitize HTML before saving
        self.description_html = sanitize_html(self.description_html)

        # Store plain text version
        self.description_stripped = strip_tags(self.description_html)

        super().save(*args, **kwargs)
```

### Frontend Validation (React)

```typescript
// ✅ CORRECT: Client-side validation (defense in depth)
// File: apps/web/components/issues/create-issue-form.tsx

import { z } from 'zod';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

const issueSchema = z.object({
  name: z.string()
    .min(1, 'Issue name is required')
    .max(255, 'Issue name too long')
    .refine(val => val.trim().length > 0, 'Issue name cannot be empty'),

  priority: z.enum(['urgent', 'high', 'medium', 'low', 'none']),

  start_date: z.string().optional().nullable(),

  target_date: z.string().optional().nullable(),

  description: z.string().optional(),
}).refine(data => {
  // Cross-field validation
  if (data.start_date && data.target_date) {
    return new Date(data.start_date) <= new Date(data.target_date);
  }
  return true;
}, {
  message: 'Target date must be after start date',
  path: ['target_date']
});

type IssueFormData = z.infer<typeof issueSchema>;

export const CreateIssueForm = () => {
  const { register, handleSubmit, formState: { errors } } = useForm<IssueFormData>({
    resolver: zodResolver(issueSchema)
  });

  const onSubmit = async (data: IssueFormData) => {
    try {
      // Submit to API (backend validates again!)
      const response = await fetch('/api/issues/', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
        credentials: 'include'  // Include cookies
      });

      if (!response.ok) {
        // Handle backend validation errors
        const errors = await response.json();
        // Show errors to user
      }
    } catch (error) {
      console.error('Failed to create issue:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('name')}
        maxLength={255}  // Browser enforced
      />
      {errors.name && <span>{errors.name.message}</span>}

      <select {...register('priority')}>
        <option value="none">None</option>
        <option value="low">Low</option>
        <option value="medium">Medium</option>
        <option value="high">High</option>
        <option value="urgent">Urgent</option>
      </select>

      <button type="submit">Create Issue</button>
    </form>
  );
};

// ❌ DANGER: No validation
const UnsafeForm = () => {
  const [name, setName] = useState('');

  const handleSubmit = async () => {
    // No validation! Accepts anything!
    await fetch('/api/issues/', {
      method: 'POST',
      body: JSON.stringify({ name })  // Could be XSS payload!
    });
  };
};
```

---

## SQL Injection Prevention

### Django ORM Safety

```python
# ✅ CORRECT: Django ORM automatically prevents SQL injection
issues = Issue.objects.filter(
    project_id=project_id,
    name__icontains=search_term  # Safe! Django parameterizes
)

# ✅ CORRECT: Complex queries with Q objects
from django.db.models import Q

issues = Issue.objects.filter(
    Q(name__icontains=search) | Q(description__icontains=search)
).filter(
    project_id=project_id,
    priority__in=priorities
)

# ✅ CORRECT: Raw SQL with parameterization (if necessary)
from django.db import connection

with connection.cursor() as cursor:
    cursor.execute(
        "SELECT * FROM issues WHERE project_id = %s AND name LIKE %s",
        [project_id, f'%{search}%']  # Parameterized - safe!
    )
    rows = cursor.fetchall()

# ❌ DANGER: String formatting with raw SQL
cursor.execute(
    f"SELECT * FROM issues WHERE project_id = {project_id}"  # SQL INJECTION!
)

# ❌ DANGER: String concatenation
query = "SELECT * FROM issues WHERE name = '" + user_input + "'"  # SQL INJECTION!
cursor.execute(query)
```

### Real Example from Plane

```python
# ✅ CORRECT: Complex filtering with Django ORM
# File: apps/api/plane/app/views/issue/base.py

class IssueViewSet(BaseViewSet):
    def list(self, request, slug, project_id):
        filters = {}

        # Priority filter
        if request.GET.get('priority'):
            filters['priority__in'] = request.GET.get('priority').split(',')

        # State filter
        if request.GET.get('state'):
            filters['state__in'] = request.GET.get('state').split(',')

        # Assignee filter
        if request.GET.get('assignees'):
            filters['assignees__in'] = request.GET.get('assignees').split(',')

        # Search filter
        search = request.GET.get('search')
        if search:
            issues = Issue.objects.filter(
                project_id=project_id,
                **filters
            ).filter(
                Q(name__icontains=search) | Q(description_stripped__icontains=search)
            )
        else:
            issues = Issue.objects.filter(
                project_id=project_id,
                **filters
            )

        # All filters are safely parameterized by Django ORM!
        return Response(IssueSerializer(issues, many=True).data)
```

---

## XSS Prevention

### React Auto-Escaping

```tsx
// ✅ CORRECT: React automatically escapes content
const IssueTitle = ({ issue }: { issue: Issue }) => {
  // Safe! React escapes {issue.name}
  return <h1>{issue.name}</h1>;

  // This renders as text, not HTML:
  // If issue.name = "<script>alert('xss')</script>"
  // React renders: "&lt;script&gt;alert('xss')&lt;/script&gt;"
};

// ❌ DANGER: dangerouslySetInnerHTML
const UnsafeComponent = ({ html }: { html: string }) => {
  // DANGER! Renders raw HTML
  return <div dangerouslySetInnerHTML={{ __html: html }} />;

  // If html contains <script>, it will execute!
};

// ✅ CORRECT: Sanitize before using dangerouslySetInnerHTML
import DOMPurify from 'dompurify';

const SafeHTMLComponent = ({ html }: { html: string }) => {
  const sanitized = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'a'],
    ALLOWED_ATTR: ['href']
  });

  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
};
```

### Content Security Policy (CSP)

```python
# ✅ CORRECT: CSP headers
# File: apps/api/plane/settings/common.py

# CSP Middleware or nginx configuration
CSP_DEFAULT_SRC = ["'self'"]
CSP_SCRIPT_SRC = [
    "'self'",
    "https://cdn.plane.so"
    # No 'unsafe-inline' or 'unsafe-eval'!
]
CSP_STYLE_SRC = ["'self'", "'unsafe-inline'"]  # CSS typically needs inline
CSP_IMG_SRC = ["'self'", "data:", "https:"]
CSP_FONT_SRC = ["'self'", "https://fonts.gstatic.com"]
CSP_CONNECT_SRC = ["'self'", "https://api.plane.so"]
CSP_FRAME_ANCESTORS = ["'none'"]  # Prevent clickjacking

# In nginx:
# add_header Content-Security-Policy "default-src 'self'; script-src 'self' https://cdn.plane.so; ...";
```

### XSS Prevention Checklist

- ✅ Use React's default escaping (don't use dangerouslySetInnerHTML)
- ✅ Sanitize HTML on backend before storing
- ✅ Implement Content Security Policy
- ✅ Validate and sanitize all user input
- ✅ Use HTTP-only cookies for tokens
- ✅ Escape data in URLs and attributes
- ✅ Validate JSON responses before rendering

---

## CSRF Protection

### Django CSRF Middleware

```python
# ✅ CORRECT: CSRF protection enabled
# File: apps/api/plane/settings/common.py

MIDDLEWARE = [
    'django.middleware.csrf.CsrfViewMiddleware',  # Must be enabled!
    # ... other middleware
]

CSRF_COOKIE_HTTPONLY = False  # Must be False for JS access
CSRF_COOKIE_SECURE = True     # HTTPS only
CSRF_COOKIE_SAMESITE = 'Lax'  # Prevent cross-site requests
CSRF_TRUSTED_ORIGINS = [
    'https://app.plane.so',
    'https://*.plane.so',
]
```

### Frontend CSRF Token Handling

```typescript
// ✅ CORRECT: Include CSRF token in requests
// File: apps/web/lib/api-client.ts

function getCookie(name: string): string | null {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) return parts.pop()?.split(';').shift() || null;
  return null;
}

export async function apiRequest(
  url: string,
  options: RequestInit = {}
): Promise<Response> {
  const csrfToken = getCookie('csrftoken');

  const headers = {
    'Content-Type': 'application/json',
    ...(csrfToken && { 'X-CSRFToken': csrfToken }),
    ...options.headers,
  };

  return fetch(url, {
    ...options,
    headers,
    credentials: 'include',  // Include cookies
  });
}

// Usage
await apiRequest('/api/issues/', {
  method: 'POST',
  body: JSON.stringify({ name: 'New Issue' })
});

// ❌ DANGER: No CSRF token
fetch('/api/issues/', {
  method: 'POST',
  body: JSON.stringify(data)
  // Missing CSRF token! Request will be rejected
});
```

### CSRF Exemption (Carefully!)

```python
# ⚠️ CAUTION: Exempting CSRF for specific views
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt  # Only for webhook endpoints!
class WebhookReceiver(View):
    def post(self, request):
        # Verify webhook signature instead
        signature = request.headers.get('X-Webhook-Signature')
        if not verify_webhook_signature(request.body, signature):
            return HttpResponse(status=403)

        # Process webhook
        return HttpResponse(status=200)
```

---

## API Security

### Rate Limiting

```python
# ✅ CORRECT: Rate limiting configuration
# File: apps/api/plane/settings/common.py

REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/hour',       # Anonymous users
        'user': '1000/hour',      # Authenticated users
        'authentication': '30/minute',  # Auth endpoints
        'api_write': '100/hour',  # Write operations
    }
}

# Custom throttle for specific endpoints
from rest_framework.throttling import UserRateThrottle

class IssueCreateThrottle(UserRateThrottle):
    rate = '100/hour'
    scope = 'api_write'

class IssueViewSet(BaseViewSet):
    throttle_classes = [IssueCreateThrottle]
```

### CORS Configuration

```python
# ✅ CORRECT: Restrictive CORS configuration
# File: apps/api/plane/settings/common.py

CORS_ALLOWED_ORIGINS = [
    "https://app.plane.so",
    "https://staging.plane.so",
]

CORS_ALLOW_CREDENTIALS = True  # Allow cookies

CORS_ALLOWED_METHODS = [
    'GET',
    'POST',
    'PUT',
    'PATCH',
    'DELETE',
    'OPTIONS',
]

CORS_ALLOWED_HEADERS = [
    'accept',
    'accept-encoding',
    'authorization',
    'content-type',
    'origin',
    'user-agent',
    'x-csrftoken',
    'x-requested-with',
]

# ❌ DANGER: Permissive CORS
# CORS_ALLOW_ALL_ORIGINS = True  # Allows any website to call your API!
# CORS_ALLOW_CREDENTIALS = True  # With wildcard - dangerous!
```

### API Versioning

```python
# ✅ CORRECT: API versioning for backward compatibility
# File: apps/api/plane/urls.py

urlpatterns = [
    path('api/v1/', include('plane.app.urls.v1')),
    path('api/v2/', include('plane.app.urls.v2')),  # New version
]

# Deprecation warning in old API
class IssueViewSet(BaseViewSet):
    def list(self, request):
        # Add deprecation header
        response = Response(...)
        response['Warning'] = '299 - "API v1 is deprecated, migrate to v2"'
        return response
```

### API Documentation

```python
# ✅ CORRECT: OpenAPI/Swagger documentation
# File: apps/api/plane/urls.py

from drf_yasg.views import get_schema_view
from drf_yasg import openapi

schema_view = get_schema_view(
    openapi.Info(
        title="Plane API",
        default_version='v1',
        description="Plane project management API",
        terms_of_service="https://plane.so/terms/",
        contact=openapi.Contact(email="support@plane.so"),
        license=openapi.License(name="AGPL-3.0"),
    ),
    public=False,  # Require authentication
)

urlpatterns = [
    path('api/docs/', schema_view.with_ui('swagger', cache_timeout=0)),
]
```

---

## Secret Management

### Environment Variables

```bash
# ✅ CORRECT: .env file (never commit this!)
# File: .env

SECRET_KEY=django-insecure-NEVER-commit-this-to-git-123456789
DATABASE_URL=postgresql://user:pass@localhost:5432/plane
REDIS_URL=redis://localhost:6379/0

AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

GOOGLE_CLIENT_ID=123456789.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=SECRET_HERE

# Email
EMAIL_HOST=smtp.gmail.com
EMAIL_HOST_USER=noreply@plane.so
EMAIL_HOST_PASSWORD=app_specific_password_here
```

```python
# ✅ CORRECT: Load from environment
# File: apps/api/plane/settings/common.py

import os
from dotenv import load_dotenv

load_dotenv()

SECRET_KEY = os.environ.get("SECRET_KEY")
if not SECRET_KEY:
    raise ValueError("SECRET_KEY must be set in environment")

DATABASES = {
    'default': dj_database_url.parse(os.environ.get("DATABASE_URL"))
}

AWS_ACCESS_KEY_ID = os.environ.get("AWS_ACCESS_KEY_ID")
AWS_SECRET_ACCESS_KEY = os.environ.get("AWS_SECRET_ACCESS_KEY")
```

### .gitignore

```bash
# ✅ CORRECT: .gitignore
# File: .gitignore

# Environment variables
.env
.env.local
.env.production

# Secrets
secrets/
*.pem
*.key
credentials.json

# Django
*.pyc
__pycache__/
db.sqlite3

# Node
node_modules/
.next/
```

### Secrets in CI/CD

```yaml
# ✅ CORRECT: GitHub Actions secrets
# File: .github/workflows/deploy.yml

name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Deploy
        env:
          SECRET_KEY: ${{ secrets.SECRET_KEY }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        run: |
          # Secrets are injected as environment variables
          ./deploy.sh
```

### Secret Rotation

```python
# ✅ CORRECT: Support multiple API keys for rotation
# File: apps/api/plane/authentication/api_key.py

class APIKey(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    key = models.CharField(max_length=64, unique=True)
    name = models.CharField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)
    expires_at = models.DateTimeField()
    is_active = models.BooleanField(default=True)
    last_used_at = models.DateTimeField(null=True)

    def is_valid(self):
        return (
            self.is_active and
            self.expires_at > timezone.now()
        )

# Allow users to have multiple keys
# When rotating:
# 1. Create new key
# 2. Update applications to use new key
# 3. Deactivate old key after grace period
```

---

## Common Vulnerabilities

### 1. Mass Assignment

```python
# ❌ DANGER: Mass assignment vulnerability
class UserUpdateView(APIView):
    def patch(self, request):
        user = request.user
        # Never do this! User can set any field!
        for key, value in request.data.items():
            setattr(user, key, value)  # User can set is_superuser=True!
        user.save()

# ✅ CORRECT: Explicitly allow fields
class UserUpdateView(APIView):
    def patch(self, request):
        user = request.user
        serializer = UserSerializer(
            user,
            data=request.data,
            partial=True
        )
        serializer.is_valid(raise_exception=True)
        serializer.save()

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['display_name', 'avatar', 'bio']  # Only these!
        read_only_fields = ['is_superuser', 'is_staff']  # Protected
```

### 2. Insecure Direct Object Reference (IDOR)

```python
# ❌ DANGER: IDOR vulnerability
class IssueDetailView(APIView):
    def get(self, request, issue_id):
        # No permission check!
        issue = Issue.objects.get(id=issue_id)
        return Response(IssueSerializer(issue).data)

# ✅ CORRECT: Check permissions
class IssueDetailView(APIView):
    def get(self, request, issue_id):
        issue = Issue.objects.get(id=issue_id)

        # Check user has access to this issue's project
        if not ProjectMember.objects.filter(
            project=issue.project,
            member=request.user,
            is_active=True
        ).exists():
            return Response(
                {"error": "Not found"},  # Don't reveal it exists!
                status=status.HTTP_404_NOT_FOUND
            )

        return Response(IssueSerializer(issue).data)
```

### 3. Path Traversal

```python
# ❌ DANGER: Path traversal vulnerability
def download_file(request, filename):
    # User can request: ../../../../etc/passwd
    file_path = f'/uploads/{filename}'
    with open(file_path, 'rb') as f:
        return HttpResponse(f.read())

# ✅ CORRECT: Validate filename
import os

def download_file(request, filename):
    # Remove path components
    filename = os.path.basename(filename)

    # Validate filename
    if not filename.isalnum():
        return Response({"error": "Invalid filename"}, status=400)

    # Build safe path
    file_path = os.path.join('/uploads/', filename)

    # Ensure path is within /uploads/
    if not file_path.startswith('/uploads/'):
        return Response({"error": "Invalid path"}, status=400)

    if not os.path.exists(file_path):
        return Response({"error": "File not found"}, status=404)

    with open(file_path, 'rb') as f:
        return HttpResponse(f.read())
```

### 4. Information Disclosure

```python
# ❌ DANGER: Leaking sensitive information
class UserListView(APIView):
    def get(self, request):
        users = User.objects.all()
        # Returns passwords, tokens, etc!
        return Response(UserSerializer(users, many=True).data)

# ✅ CORRECT: Only return necessary data
class UserListView(APIView):
    def get(self, request):
        users = User.objects.all()
        serializer = UserPublicSerializer(users, many=True)
        return Response(serializer.data)

class UserPublicSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'display_name', 'avatar']  # Only public fields!
        # Never include: password, email, tokens, etc.
```

### 5. Race Conditions

```python
# ❌ DANGER: Race condition
def transfer_credits(from_user, to_user, amount):
    if from_user.credits >= amount:
        from_user.credits -= amount
        from_user.save()
        to_user.credits += amount
        to_user.save()
    # Two requests can both check credits before either saves!

# ✅ CORRECT: Use database transactions and locks
from django.db import transaction
from django.db.models import F

@transaction.atomic
def transfer_credits(from_user, to_user, amount):
    # Lock rows for update
    from_user = User.objects.select_for_update().get(id=from_user.id)
    to_user = User.objects.select_for_update().get(id=to_user.id)

    if from_user.credits >= amount:
        # Use F() expressions for atomic updates
        User.objects.filter(id=from_user.id).update(
            credits=F('credits') - amount
        )
        User.objects.filter(id=to_user.id).update(
            credits=F('credits') + amount
        )
        return True
    return False
```

---

## Security Checklist

### Pre-Deployment Checklist

#### Django Settings

- [ ] `DEBUG = False` in production
- [ ] `SECRET_KEY` from environment (not hardcoded)
- [ ] `ALLOWED_HOSTS` configured (no wildcards)
- [ ] `SECURE_SSL_REDIRECT = True`
- [ ] `SESSION_COOKIE_SECURE = True`
- [ ] `CSRF_COOKIE_SECURE = True`
- [ ] `SECURE_HSTS_SECONDS` set (31536000 for 1 year)
- [ ] CORS configured restrictively
- [ ] Security middleware enabled
- [ ] Rate limiting configured

#### Authentication

- [ ] Strong password policy enforced
- [ ] Password hashing algorithm up-to-date
- [ ] Session timeout configured
- [ ] MFA available for sensitive accounts
- [ ] Account lockout after failed attempts
- [ ] Password reset tokens expire (1 hour max)
- [ ] HTTP-only cookies for session tokens

#### Authorization

- [ ] RBAC implemented throughout app
- [ ] Permission checks on all endpoints
- [ ] Row-level security for sensitive data
- [ ] API keys can be rotated
- [ ] Audit logs for sensitive operations

#### Input Validation

- [ ] All endpoints validate input
- [ ] File uploads restricted (type, size)
- [ ] HTML sanitized before storage/display
- [ ] SQL injection prevented (use ORM)
- [ ] URL validation for webhooks/links

#### API Security

- [ ] Rate limiting on all endpoints
- [ ] CORS configured properly
- [ ] API versioning in place
- [ ] Deprecation warnings for old APIs
- [ ] API documentation up-to-date

#### Frontend Security

- [ ] Content Security Policy configured
- [ ] XSS protection (use React defaults)
- [ ] CSRF tokens included in requests
- [ ] No sensitive data in localStorage
- [ ] Sanitize before dangerouslySetInnerHTML

#### Infrastructure

- [ ] HTTPS enforced everywhere
- [ ] TLS 1.2+ only
- [ ] Security headers configured
- [ ] DDoS protection in place
- [ ] Web Application Firewall (WAF)
- [ ] Regular security updates

#### Monitoring

- [ ] Security event logging
- [ ] Failed authentication tracking
- [ ] Suspicious activity alerts
- [ ] Audit trail for sensitive data
- [ ] Logs sent to SIEM system
- [ ] Regular log review

#### Compliance

- [ ] GDPR compliance (if applicable)
- [ ] Data retention policies
- [ ] Right to deletion implemented
- [ ] Privacy policy up-to-date
- [ ] Cookie consent banner

### Code Review Security Checklist

When reviewing pull requests:

- [ ] No hardcoded secrets or credentials
- [ ] Input validation on all user input
- [ ] Authorization checks on endpoints
- [ ] SQL injection prevention
- [ ] XSS prevention
- [ ] CSRF protection maintained
- [ ] Sensitive data not logged
- [ ] Error messages don't leak information
- [ ] Dependencies updated (no known vulnerabilities)
- [ ] Tests include security scenarios

---

## React Analogy: Frontend Security

| Backend (Django) | Frontend (React) | Security Concern |
|-----------------|------------------|------------------|
| Django ORM | Query parameters | SQL Injection |
| `bleach.clean()` | React auto-escaping | XSS |
| CSRF middleware | CSRF token in headers | CSRF |
| Permission classes | Route guards | Authorization |
| Form validation | Zod schema | Input validation |
| HTTP-only cookies | `credentials: 'include'` | Token storage |
| HTTPS redirect | Fetch with https:// | Man-in-the-middle |
| Rate limiting | Debounce/throttle | DoS |

**Key Insight**: Frontend security is defense-in-depth. Never trust client-side validation alone—always validate on the backend!

---

## Status: ✅ CURRENT (November 2025)

All security patterns in this guide reflect current best practices for:

- ✅ Django 4.2 security features
- ✅ React 18.3 security patterns
- ✅ OWASP Top 10 (2021 edition)
- ✅ Modern authentication (JWT, OAuth 2.0)
- ✅ API security standards

---

## Further Reading

### Official Documentation
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Django Security](https://docs.djangoproject.com/en/4.2/topics/security/)
- [React Security](https://react.dev/learn/security)
- [Django REST Framework Security](https://www.django-rest-framework.org/topics/security/)

### Security Resources
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [Django Security Checklist](https://docs.djangoproject.com/en/4.2/howto/deployment/checklist/)
- [Mozilla Web Security Guidelines](https://infosec.mozilla.org/guidelines/web_security)

### Tools
- [Bandit](https://github.com/PyCQA/bandit) - Python security linter
- [Safety](https://github.com/pyupio/safety) - Check dependencies for vulnerabilities
- [npm audit](https://docs.npmjs.com/cli/v8/commands/npm-audit) - JavaScript security audit
- [OWASP ZAP](https://www.zaproxy.org/) - Security testing tool
- [Semgrep](https://semgrep.dev/) - Static analysis for security

---

**Remember**: Security is not a one-time task—it's an ongoing process. Stay updated with security advisories, conduct regular audits, and always assume breach!
