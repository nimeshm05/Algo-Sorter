# Configure CORS Middleware with Explicit Origin Allowlist for Cross-Origin API Access: Development Origins Localhost

Status: proposed
Date: 2025-01-20
Deciders: Detection Pipeline (automated)

## Context

- The application exposes HTTP endpoints that must be accessible from browser-based clients hosted on different origins, requiring Cross-Origin Resource Sharing (CORS) configuration
- Express.js middleware stack processes incoming requests before they reach route handlers, providing a centralized point for security policy enforcement
- The visitor tracking API at server/src/index.ts serves both a production frontend (https://suraj-gov.github.io/sorter) and local development environment (localhost:3000)
- Without explicit CORS configuration, browsers block cross-origin requests by default, preventing legitimate client applications from accessing the API
- The cors middleware is applied globally via app.use(), establishing a boundary-level security control that affects all downstream route handlers

## Problem Statement

API endpoints must be accessible to specific trusted origins while preventing unauthorized cross-origin access from untrusted domains. The default browser same-origin policy blocks all cross-origin requests, but a blanket permissive CORS policy would expose the API to cross-site request forgery and data exfiltration attacks from malicious origins.

## Decision

1. SHOULD: Development origins (localhost) SHOULD be included in the allowlist only when necessary for local testing workflows

## Policy Block

- SHOULD Development origins (localhost) SHOULD be included in the allowlist only when necessary for local testing workflows

In scope:
- All HTTP endpoints exposed by the Express.js application
- Cross-origin requests from browser-based clients
- Production and development environment configurations
- Middleware stack initialization in server/src/index.ts

Out of scope:
- Same-origin requests (not subject to CORS policy)
- Server-to-server API calls (not browser-initiated)
- WebSocket connections (governed by separate origin policies)
- Static asset serving that does not require CORS headers

Exceptions:
- EXC-001: Public API endpoints explicitly designed for third-party consumption may use more permissive CORS policies

## Rationale

- The evidence shows explicit CORS configuration with allowedHeaders containing two specific origins (https://suraj-gov.github.io/sorter and localhost:3000), demonstrating an allowlist-based approach to cross-origin access control
- Applying CORS middleware at the app.use() level establishes a security boundary that protects all downstream routes, including the visitor tracking endpoints that perform database operations
- The pattern coordinates with the Express.js middleware architecture to enforce security policy before request processing reaches business logic, separating security concerns from application logic
- The allowlist approach balances accessibility for legitimate clients against the security risk of unrestricted cross-origin access, which could enable CSRF attacks or unauthorized data access

## Consequences

Positive:
- Legitimate browser-based clients from trusted origins can successfully make cross-origin requests to the API
- Unauthorized origins are blocked by browser CORS enforcement, reducing the attack surface for cross-site request forgery
- Centralized CORS configuration in the middleware stack provides a single point of control for cross-origin access policy
- The explicit allowlist makes security policy visible and auditable in the codebase

Negative:
- Hardcoded origin allowlist in application code requires code changes and redeployment to add new trusted origins
- Development origins (localhost:3000) in the allowlist may inadvertently be deployed to production if not properly managed
- The current implementation uses allowedHeaders parameter which may be a misconfiguration (should likely be origin parameter for CORS origin control)
- Maintenance overhead increases as the number of trusted client applications grows

## Alternatives

- Use wildcard CORS policy (Access-Control-Allow-Origin: *) to permit all origins (rejected)
  Rejected because: Wildcard policy exposes the API to cross-site request forgery attacks and unauthorized data access from any malicious origin, violating security requirements for visitor tracking data
  When valid: Only appropriate for truly public, read-only APIs with no sensitive data or state-changing operations
- Implement dynamic origin validation with pattern matching or database-driven allowlist (deferred)
  Rejected because: Adds complexity and potential performance overhead for the current single-client use case, but may be necessary as the number of trusted origins scales
  When valid: When the application must support multiple client applications with dynamic origin registration requirements
- Deploy API and client application on the same origin to avoid CORS entirely (rejected)
  Rejected because: The client application is hosted on GitHub Pages (suraj-gov.github.io) while the API requires database connectivity and server-side logic, making same-origin deployment architecturally infeasible
  When valid: When both client and server can be deployed as a monolithic application on a single domain

## Risks

- Misconfiguration of CORS parameters (using allowedHeaders instead of origin) may result in ineffective origin validation, allowing unintended cross-origin access
  Mitigation: Verify CORS middleware configuration against cors package documentation, implement integration tests that validate origin blocking behavior, and conduct security review of CORS implementation
  Owner: Engineering team with security team review
- Hardcoded localhost:3000 in production deployment would allow any attacker running a local server to bypass origin restrictions
  Mitigation: Externalize CORS configuration to environment variables, implement environment-specific configuration files, and add deployment validation checks to prevent development origins in production
  Owner: DevOps and engineering team
- Adding new trusted client applications requires code changes and redeployment, creating operational friction and potential downtime
  Mitigation: Migrate to configuration-driven allowlist stored in environment variables or configuration service, implement hot-reload capability for CORS configuration updates without redeployment
  Owner: Engineering team

## Implementation Notes

- Verify that the cors middleware configuration uses the 'origin' parameter rather than 'allowedHeaders' for origin validation - the current evidence suggests a potential misconfiguration
- Extract the origin allowlist to environment variables (e.g., CORS_ALLOWED_ORIGINS) with comma-separated values, parsed at application startup
- Implement separate configuration files for development and production environments, ensuring localhost origins are excluded from production builds
- Add integration tests that verify CORS headers are correctly set for allowed origins and blocked for unauthorized origins
- Consider implementing CORS preflight request handling for endpoints that accept non-simple HTTP methods or custom headers

## Continuation Context


Verify commands:
- grep -r "cors({" server/src/ | grep -E "(origin|allowedHeaders)" | grep -v "\*"
- curl -H "Origin: https://suraj-gov.github.io/sorter" -I http://localhost:${PORT:-3000}/ | grep -i "access-control-allow-origin"
- grep -r "localhost" server/src/ | grep -i cors && echo "WARNING: localhost found in CORS config"

Accept when:
- CORS middleware is configured with explicit origin allowlist (not wildcard) and registered before route handlers in the Express.js middleware stack
- Verification commands confirm that Access-Control-Allow-Origin headers are set correctly for allowed origins and requests from unauthorized origins are blocked
- Production configuration excludes development origins (localhost) and all allowed origins use HTTPS protocol

## Enforcement

- Verified by: Code review process validates CORS configuration against security requirements before merge
- Verified by: Integration tests verify CORS behavior for both allowed and blocked origins in CI pipeline
- Verified by: Security scanning tools check for wildcard CORS configurations or missing origin validation
- Verified by: Manual security review for any changes to CORS middleware configuration or allowlist
- Violation handling: CI pipeline fails if wildcard CORS configuration or missing origin validation is detected
- Violation handling: Code review blocks merge requests that introduce insecure CORS configurations
- Violation handling: Security team is notified of CORS policy violations detected in production monitoring
- Violation handling: Incidents involving unauthorized cross-origin access trigger immediate security review and remediation
- Exception process: Exception requests must document the specific API endpoints requiring permissive CORS policy and business justification
- Exception process: Security team reviews exception requests and assesses risk of cross-origin access for the specific use case
- Exception process: Approved exceptions are documented in the ADR with specific scope limitations and compensating controls
- Exception process: Exceptions are reviewed quarterly and revoked when no longer necessary