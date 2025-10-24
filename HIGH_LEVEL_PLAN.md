# High-Level Development Plan - Resume-SSO

**Service Type**: Authentication & Identity Service (Cloudflare Workers)
**Build Approach**: AI-Automated Development
**Priority**: CRITICAL - Foundation service (blocks all other services)

---

## Dependency Tree Overview

```
Resume-SSO (Foundation)
├── MVP-1: Email/Password Auth ⟶ Blocks: All other services
│   ├── Database Schema
│   ├── Registration & Email Verification
│   ├── Login & JWT Generation
│   └── Password Management
│
├── MVP-2: Session Management ⟶ Blocks: API authentication
│   ├── JWT Validation Middleware
│   ├── Token Refresh
│   └── Logout & Blacklisting
│
├── MVP-3: OAuth Integration ⟶ Blocks: Social login features
│   ├── Google OAuth
│   └── LinkedIn OAuth
│
└── MVP-4: Security & MFA ⟶ Blocks: Premium features
    ├── Rate Limiting
    ├── TOTP MFA
    └── Account Security
```

---

## MVP-1: Email/Password Authentication (Foundation)

**Status**: MUST BUILD FIRST - Blocks everything else
**Dependencies**: None
**Blocks**: Resume-API, Resume-UI, Resume-Admin, Resume-Admin-API

### Epic 1.1: Database Schema & Setup
**What**: D1 database structure for user management
**Complexity**: Medium
**Dependencies**: None

**Tasks**:
- [ ] Create D1 database schema (users, oauth_accounts, verification_tokens, sessions)
- [ ] Set up wrangler.toml with D1 binding
- [ ] Create migration scripts
- [ ] Seed initial data structure

**Acceptance Criteria**:
- [ ] D1 database created and accessible
- [ ] Schema matches feature plan specifications
- [ ] Migrations can be applied successfully

---

### Epic 1.2: User Registration Flow
**What**: Allow users to create accounts
**Complexity**: Large
**Dependencies**: Epic 1.1 (Database Schema)

**Tasks**:
- [ ] `POST /auth/register` endpoint
- [ ] Email validation logic
- [ ] Password strength validation (min 8 chars, uppercase, number, special)
- [ ] Password hashing with bcrypt (cost factor 12)
- [ ] Email verification token generation
- [ ] Send verification email via Cloudflare Email Workers

**Acceptance Criteria**:
- [ ] Users can register with email/password
- [ ] Passwords are hashed securely
- [ ] Verification emails sent successfully
- [ ] Duplicate email registrations rejected

---

### Epic 1.3: Email Verification
**What**: Verify user email addresses
**Complexity**: Medium
**Dependencies**: Epic 1.2 (Registration)

**Tasks**:
- [ ] `GET /auth/verify/:token` endpoint
- [ ] Token validation (24-hour expiration)
- [ ] Account activation logic
- [ ] Error handling for expired/invalid tokens

**Acceptance Criteria**:
- [ ] Email verification completes successfully
- [ ] Expired tokens are rejected
- [ ] User account is activated after verification

---

### Epic 1.4: User Login
**What**: Authenticate users and issue JWT tokens
**Complexity**: Large
**Dependencies**: Epic 1.3 (Email Verification)

**Tasks**:
- [ ] `POST /auth/login` endpoint
- [ ] Email + password validation
- [ ] JWT token generation (access + refresh)
- [ ] Token payload structure (user_id, email, subscription_tier, roles)
- [ ] Return user profile on successful login

**Acceptance Criteria**:
- [ ] Users can log in with verified email/password
- [ ] JWT tokens generated with correct payload
- [ ] Unverified accounts cannot log in
- [ ] Invalid credentials return appropriate error

---

### Epic 1.5: Password Management
**What**: Password reset and change functionality
**Complexity**: Medium
**Dependencies**: Epic 1.4 (Login)

**Tasks**:
- [ ] `POST /auth/forgot-password` endpoint (request reset)
- [ ] `POST /auth/reset-password` endpoint (set new password)
- [ ] `POST /auth/change-password` endpoint (authenticated users)
- [ ] Reset token generation and validation (1-hour expiration)
- [ ] Send password reset emails
- [ ] Invalidate all sessions on password change

**Acceptance Criteria**:
- [ ] Users can request password reset via email
- [ ] Reset tokens expire after 1 hour
- [ ] Users can change password when authenticated
- [ ] All sessions invalidated after password reset

---

## MVP-2: Session Management (Critical)

**Status**: Required for API authentication
**Dependencies**: MVP-1
**Blocks**: Resume-API authentication middleware

### Epic 2.1: JWT Validation Middleware
**What**: Validate JWT tokens on protected routes
**Complexity**: Medium
**Dependencies**: Epic 1.4 (Login)

**Tasks**:
- [ ] JWT signature verification using RS256
- [ ] Token expiration validation
- [ ] User context injection from JWT payload
- [ ] `GET /auth/me` endpoint (current user info)

**Acceptance Criteria**:
- [ ] JWT tokens validated correctly
- [ ] Expired tokens rejected
- [ ] User context available in protected routes

---

### Epic 2.2: Token Refresh
**What**: Refresh access tokens without re-login
**Complexity**: Medium
**Dependencies**: Epic 2.1 (JWT Validation)

**Tasks**:
- [ ] `POST /auth/refresh` endpoint
- [ ] Refresh token validation (7-day expiration)
- [ ] Issue new access token
- [ ] Rotate refresh token for security

**Acceptance Criteria**:
- [ ] Expired access tokens can be refreshed
- [ ] Refresh tokens rotate on each use
- [ ] Invalid refresh tokens rejected

---

### Epic 2.3: Logout & Session Blacklisting
**What**: Allow users to log out and invalidate tokens
**Complexity**: Small
**Dependencies**: Epic 2.1 (JWT Validation)

**Tasks**:
- [ ] `POST /auth/logout` endpoint
- [ ] Add JWT to blacklist in Cloudflare KV
- [ ] Check blacklist in validation middleware
- [ ] Clear client-side tokens

**Acceptance Criteria**:
- [ ] Users can log out successfully
- [ ] Blacklisted tokens are rejected
- [ ] KV cleanup for expired blacklist entries

---

## MVP-3: OAuth Integration (User Experience)

**Status**: Enhances user onboarding
**Dependencies**: MVP-2
**Blocks**: Social login features in Resume-UI

### Epic 3.1: Google OAuth
**What**: Allow users to sign in with Google
**Complexity**: Large
**Dependencies**: Epic 2.1 (JWT Validation)

**Tasks**:
- [ ] `GET /auth/google` endpoint (initiate OAuth flow)
- [ ] `GET /auth/google/callback` endpoint (handle callback)
- [ ] Google OAuth client configuration
- [ ] User profile creation/linking
- [ ] JWT generation for OAuth users
- [ ] Handle email conflicts (link accounts)

**Acceptance Criteria**:
- [ ] Users can sign in with Google
- [ ] User profiles created from Google data
- [ ] Existing accounts linked if email matches
- [ ] JWT tokens issued after OAuth

---

### Epic 3.2: LinkedIn OAuth
**What**: Allow users to sign in with LinkedIn (import resume data)
**Complexity**: Large
**Dependencies**: Epic 3.1 (Google OAuth - reuse pattern)

**Tasks**:
- [ ] `GET /auth/linkedin` endpoint (initiate OAuth flow)
- [ ] `GET /auth/linkedin/callback` endpoint (handle callback)
- [ ] LinkedIn OAuth client configuration
- [ ] Extract profile data for resume auto-populate
- [ ] User profile creation/linking
- [ ] JWT generation for OAuth users

**Acceptance Criteria**:
- [ ] Users can sign in with LinkedIn
- [ ] Profile data extracted for resume use
- [ ] Existing accounts linked if email matches

---

## MVP-4: Security & MFA (Premium Features)

**Status**: Required for production and premium users
**Dependencies**: MVP-3
**Blocks**: Premium tier features

### Epic 4.1: Rate Limiting
**What**: Prevent abuse and brute force attacks
**Complexity**: Medium
**Dependencies**: MVP-2

**Tasks**:
- [ ] Implement rate limiting using Cloudflare Rate Limiting
- [ ] Login attempts: 5 per 15 minutes
- [ ] Registration: 3 per hour per IP
- [ ] Password reset: 3 per hour
- [ ] Return appropriate error codes (429)

**Acceptance Criteria**:
- [ ] Rate limits enforced on all auth endpoints
- [ ] Clear error messages for rate-limited requests
- [ ] Limits reset after time window

---

### Epic 4.2: Account Lockout Protection
**What**: Lock accounts after failed login attempts
**Complexity**: Medium
**Dependencies**: Epic 4.1 (Rate Limiting)

**Tasks**:
- [ ] Track failed login attempts in D1
- [ ] Lock account after 5 failed attempts
- [ ] Auto-unlock after 30 minutes
- [ ] Email notification on lockout
- [ ] Manual unlock via email verification

**Acceptance Criteria**:
- [ ] Accounts locked after 5 failed attempts
- [ ] Auto-unlock after 30 minutes
- [ ] Email notifications sent

---

### Epic 4.3: Multi-Factor Authentication (TOTP)
**What**: Two-factor authentication for enhanced security
**Complexity**: Large
**Dependencies**: Epic 4.2

**Tasks**:
- [ ] `POST /auth/mfa/enable` endpoint
- [ ] `POST /auth/mfa/verify` endpoint
- [ ] `POST /auth/mfa/disable` endpoint
- [ ] Generate TOTP secret and QR code
- [ ] Backup codes generation (10 codes)
- [ ] MFA validation during login
- [ ] Remember device option (30 days in KV)

**Acceptance Criteria**:
- [ ] Users can enable TOTP MFA
- [ ] QR codes generated for authenticator apps
- [ ] MFA required during login when enabled
- [ ] Backup codes work when TOTP unavailable

---

## Critical Path (Build Order)

1. **MVP-1.1**: Database Schema ⟶ Foundation for everything
2. **MVP-1.2**: User Registration ⟶ Allows user creation
3. **MVP-1.3**: Email Verification ⟶ Activates accounts
4. **MVP-1.4**: User Login ⟶ Issues JWT tokens
5. **MVP-1.5**: Password Management ⟶ Completes basic auth
6. **MVP-2.1**: JWT Validation ⟶ Enables API authentication
7. **MVP-2.2**: Token Refresh ⟶ Better UX
8. **MVP-2.3**: Logout ⟶ Security requirement
9. **MVP-3.1**: Google OAuth ⟶ Easier onboarding
10. **MVP-3.2**: LinkedIn OAuth ⟶ Resume data import
11. **MVP-4.1**: Rate Limiting ⟶ Production readiness
12. **MVP-4.2**: Account Lockout ⟶ Security hardening
13. **MVP-4.3**: MFA ⟶ Premium security feature

---

## What Blocks What

| This Epic | Blocks These Epics |
|-----------|-------------------|
| MVP-1.1 (Database) | Everything |
| MVP-1.4 (Login) | Resume-API authentication |
| MVP-2.1 (JWT Validation) | All API endpoints requiring auth |
| MVP-3.1 (Google OAuth) | Social login in Resume-UI |
| MVP-4.3 (MFA) | Premium tier security features |

---

## Testing Strategy

### Unit Tests
- Password hashing/validation
- JWT generation/validation
- Token expiration logic
- Rate limiting logic

### Integration Tests
- Full registration → verification → login flow
- OAuth flows (Google, LinkedIn)
- Password reset flow
- MFA enrollment and verification

### Security Tests
- SQL injection attempts
- JWT tampering
- Rate limit bypass attempts
- Weak password rejection

---

## API Endpoints Summary

### Public Endpoints (MVP-1)
- `POST /auth/register` - Create account
- `POST /auth/login` - Authenticate
- `POST /auth/forgot-password` - Request reset
- `POST /auth/reset-password` - Set new password
- `GET /auth/verify/:token` - Verify email

### OAuth Endpoints (MVP-3)
- `GET /auth/google` - Initiate Google OAuth
- `GET /auth/google/callback` - Handle Google callback
- `GET /auth/linkedin` - Initiate LinkedIn OAuth
- `GET /auth/linkedin/callback` - Handle LinkedIn callback

### Protected Endpoints (MVP-2)
- `POST /auth/logout` - Invalidate session
- `POST /auth/refresh` - Refresh access token
- `GET /auth/me` - Get current user
- `POST /auth/change-password` - Update password

### MFA Endpoints (MVP-4)
- `POST /auth/mfa/enable` - Enable MFA
- `POST /auth/mfa/verify` - Verify TOTP code
- `POST /auth/mfa/disable` - Disable MFA

---

## Environment Variables

```bash
# JWT Configuration
JWT_SECRET=<secret-key>
JWT_ACCESS_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d

# OAuth Providers
GOOGLE_CLIENT_ID=<client-id>
GOOGLE_CLIENT_SECRET=<secret>
LINKEDIN_CLIENT_ID=<client-id>
LINKEDIN_CLIENT_SECRET=<secret>

# Email Service
EMAIL_FROM=noreply@resumebuilder.com

# Rate Limiting
RATE_LIMIT_LOGIN=5/15m
RATE_LIMIT_REGISTER=3/1h
```

---

## Success Metrics

- [ ] 100% auth endpoint test coverage
- [ ] < 200ms response time for login
- [ ] Zero JWT security vulnerabilities
- [ ] Rate limiting prevents brute force attacks
- [ ] MFA adoption > 20% for premium users
