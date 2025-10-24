# Resume-SSO

**Authentication & Identity Service** for the Resume Builder SaaS platform.

> **Critical Foundation Service** - This service must be built first as all other services depend on it for authentication and authorization.

## Overview

Resume-SSO is the central authentication and identity management service built on Cloudflare Workers. It provides secure user authentication, session management, OAuth integration, and multi-factor authentication for the entire Resume Builder platform.

## Technology Stack

- **Runtime**: Cloudflare Workers (serverless)
- **Database**: Cloudflare D1 (SQLite)
- **Cache**: Cloudflare KV (token blacklist, device memory)
- **Framework**: Hono (lightweight web framework)
- **Authentication**: JWT (RS256), bcrypt password hashing
- **Email**: Cloudflare Email Workers
- **Language**: TypeScript

## Key Features

### MVP-1: Email/Password Authentication
- User registration with email verification
- Secure password hashing (bcrypt, cost factor 12)
- Email verification tokens (24-hour expiration)
- Password management (reset, change)
- JWT token generation (access + refresh)

### MVP-2: Session Management
- JWT validation middleware (RS256)
- Token refresh mechanism (7-day refresh tokens)
- Logout with token blacklisting (Cloudflare KV)
- Current user info endpoint

### MVP-3: OAuth Integration
- Google OAuth 2.0
- LinkedIn OAuth 2.0 (with resume data import)
- Account linking for existing users

### MVP-4: Security & MFA
- Rate limiting (login, registration, password reset)
- Account lockout after 5 failed attempts
- TOTP-based Multi-Factor Authentication
- Backup codes generation
- Device memory (30-day trusted devices)

## Service Dependencies

**This service has NO dependencies** - it is the foundation service.

**Services that depend on Resume-SSO**:
- Resume-API (requires JWT validation)
- Resume-UI (requires authentication)
- Resume-Admin-API (requires admin authentication + MFA)
- Resume-Admin (requires admin authentication + MFA)

## API Endpoints

### Public Endpoints
- `POST /auth/register` - Create account
- `POST /auth/login` - Authenticate user
- `POST /auth/forgot-password` - Request password reset
- `POST /auth/reset-password` - Set new password
- `GET /auth/verify/:token` - Verify email address

### OAuth Endpoints
- `GET /auth/google` - Initiate Google OAuth
- `GET /auth/google/callback` - Handle Google callback
- `GET /auth/linkedin` - Initiate LinkedIn OAuth
- `GET /auth/linkedin/callback` - Handle LinkedIn callback

### Protected Endpoints
- `GET /auth/me` - Get current user info
- `POST /auth/refresh` - Refresh access token
- `POST /auth/logout` - Invalidate session
- `POST /auth/change-password` - Update password

### MFA Endpoints
- `POST /auth/mfa/enable` - Enable MFA
- `POST /auth/mfa/verify` - Verify TOTP code
- `POST /auth/mfa/disable` - Disable MFA

## Development Setup

### Prerequisites
- Node.js 18+
- Wrangler CLI (`npm install -g wrangler`)
- Cloudflare account

### Environment Variables

Create a `.dev.vars` file:

```bash
# JWT Configuration
JWT_SECRET=<your-secret-key>
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

### Quick Start

```bash
# Install dependencies
npm install

# Run database migrations
npm run db:migrate

# Start development server
npm run dev

# Run tests
npm test

# Deploy to production
npm run deploy
```

## Database Schema

```sql
-- Users table
CREATE TABLE users (
  id TEXT PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT,
  is_verified INTEGER DEFAULT 0,
  created_at INTEGER NOT NULL
);

-- OAuth accounts
CREATE TABLE oauth_accounts (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  provider TEXT NOT NULL,
  provider_user_id TEXT NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Verification tokens
CREATE TABLE verification_tokens (
  token TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  type TEXT NOT NULL,
  expires_at INTEGER NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

-- MFA settings
CREATE TABLE mfa_settings (
  user_id TEXT PRIMARY KEY,
  secret TEXT NOT NULL,
  backup_codes TEXT NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

## JWT Payload Structure

```typescript
{
  user_id: string;
  email: string;
  subscription_tier: 'free' | 'pro' | 'premium';
  roles: string[];
  iat: number;
  exp: number;
}
```

## Rate Limiting

- Login attempts: **5 per 15 minutes**
- Registration: **3 per hour per IP**
- Password reset: **3 per hour**
- Failed login lockout: **5 attempts → 30 minute lockout**

## Security Features

- Password requirements: min 8 chars, uppercase, number, special character
- JWT signing: RS256 algorithm
- Token expiration: 15 minutes (access), 7 days (refresh)
- Token blacklisting: Cloudflare KV with automatic cleanup
- TOTP MFA: 6-digit codes with 30-second window
- Backup codes: 10 one-time use codes

## Testing

```bash
# Run all tests
npm test

# Run unit tests
npm run test:unit

# Run integration tests
npm run test:integration

# Run security tests
npm run test:security

# Test coverage
npm run test:coverage
```

## Performance Targets

- Login response time: **< 200ms**
- Registration response time: **< 500ms**
- JWT validation: **< 50ms**
- Token refresh: **< 100ms**

## Development Plan

See [HIGH_LEVEL_PLAN.md](./HIGH_LEVEL_PLAN.md) for the complete dependency-tree based development plan with MVPs, epics, and critical path.

## GitHub Issues

Track development progress in [GitHub Issues](https://github.com/MEYWD-Labs/Resume-SSO/issues):
- [MVP-1 Epics](https://github.com/MEYWD-Labs/Resume-SSO/issues?q=is%3Aissue+is%3Aopen+MVP-1) - Email/Password Authentication
- [MVP-2 Epics](https://github.com/MEYWD-Labs/Resume-SSO/issues?q=is%3Aissue+is%3Aopen+MVP-2) - Session Management
- [MVP-3 Epics](https://github.com/MEYWD-Labs/Resume-SSO/issues?q=is%3Aissue+is%3Aopen+MVP-3) - OAuth Integration
- [MVP-4 Epics](https://github.com/MEYWD-Labs/Resume-SSO/issues?q=is%3Aissue+is%3Aopen+MVP-4) - Security & MFA

## Deployment

```bash
# Deploy to production
wrangler deploy

# Deploy with specific environment
wrangler deploy --env production

# View logs
wrangler tail
```

## Support

For issues or questions, create a GitHub issue in this repository.

## License

Proprietary - All rights reserved
