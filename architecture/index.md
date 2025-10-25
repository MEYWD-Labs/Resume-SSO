# Resume-SSO - Authentication Service Architecture

## Overview

Resume-SSO is the central authentication service that provides JWT token-based authentication to all other services in the Resume Builder platform. This service handles user registration, login, OAuth integration, token generation, validation, and refresh operations.

## Architecture Documentation

This directory contains the architecture documentation relevant to the SSO authentication service:

### Core Authentication Documents

1. **[authentication-flow.md](./authentication-flow.md)** - MOST IMPORTANT
   - JWT token flow across all services
   - Token structure and lifecycle
   - Token validation and refresh flows
   - OAuth integration flow
   - Cross-service authentication patterns

2. **[security-architecture.md](./security-architecture.md)**
   - Defense in depth security layers
   - Authentication and authorization mechanisms
   - JWT signing and token rotation
   - Role-Based Access Control (RBAC)
   - Multi-Factor Authentication (MFA)

3. **[cloudflare-serverless-architecture.md](./cloudflare-serverless-architecture.md)**
   - Cloudflare Workers infrastructure
   - D1 database for user credentials
   - KV store for refresh tokens and sessions
   - Serverless deployment context

### Supporting Documents

4. **[api-contracts.md](./api-contracts.md)**
   - Resume-SSO API endpoints
   - Authentication endpoints (register, login, logout, refresh)
   - OAuth endpoints (Google, GitHub)
   - Token validation endpoints
   - API response formats

5. **[deployment-architecture.md](./deployment-architecture.md)**
   - CI/CD pipeline for SSO service
   - Staging and production deployment flows
   - Health checks and rollback procedures
   - Infrastructure as code configuration

## Service Responsibilities

The Resume-SSO service is responsible for:

- User registration and credential management
- Username/password authentication
- OAuth provider integration (Google, GitHub, etc.)
- JWT access token generation and signing
- Refresh token management and rotation
- Token validation for all platform services
- Session management via Cloudflare KV
- Security event logging and audit trails

All other services in the platform depend on Resume-SSO for authentication and rely on the JWT tokens it provides to authorize user requests.
