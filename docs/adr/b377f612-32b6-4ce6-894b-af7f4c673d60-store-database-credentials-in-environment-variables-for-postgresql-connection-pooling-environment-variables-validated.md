# Store Database Credentials in Environment Variables for PostgreSQL Connection Pooling: Environment Variables Validated

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application uses PostgreSQL as its primary datastore with connection pooling via the pg library
- Database connection strings contain sensitive credentials that must not be hardcoded in source code
- The application requires runtime configuration for deployment flexibility across environments (development, staging, production)
- Port configuration needs to be externalized to support different deployment targets and container orchestration

## Problem Statement

Applications integrating with external databases require secure credential management that prevents exposure in version control while maintaining deployment flexibility across multiple environments. Hardcoding connection strings creates security vulnerabilities and reduces portability.

## Decision

1. SHOULD: Environment variables SHOULD be validated at application startup to fail fast on misconfiguration

## Policy Block

- SHOULD Environment variables SHOULD be validated at application startup to fail fast on misconfiguration

In scope:
- PostgreSQL database connections using the pg library
- Server port configuration for Express applications
- Runtime configuration requiring environment-specific values
- Credentials and connection strings for external service integration

Out of scope:
- Build-time configuration that does not vary by environment
- Public API endpoints and non-sensitive configuration
- Client-side configuration or browser environment variables
- Development-only mock data or test fixtures

Exceptions:
- EXC-001: Local development using .env files with .gitignore exclusion
- EXC-002: Integration tests using test-specific environment variables or in-memory databases

## Rationale

- The evidence shows process.env.DATABASE_URL and process.env.PORT being accessed for runtime configuration, indicating environment-based secrets management
- Using environment variables aligns with twelve-factor app methodology for configuration management and enables secure deployment across cloud platforms
- The pattern prevents credential exposure in version control while maintaining deployment flexibility
- Connection pooling with environment-sourced credentials supports horizontal scaling and multi-environment deployments

## Consequences

Positive:
- Credentials remain external to source code, reducing security risk from version control exposure
- Deployment flexibility across environments without code changes
- Compatibility with container orchestration platforms (Docker, Kubernetes) and cloud providers
- Simplified credential rotation without application redeployment

Negative:
- Runtime failures if environment variables are not properly configured
- Additional operational complexity in managing environment-specific configuration
- Potential for environment variable leakage through logging or error messages
- Dependency on deployment platform's secrets management capabilities

## Alternatives

- Hardcode database credentials directly in source code (rejected)
  Rejected because: Creates severe security vulnerability by exposing credentials in version control and prevents environment-specific configuration
  When valid: Never valid for production systems
- Use dedicated secrets management service (HashiCorp Vault, AWS Secrets Manager) (deferred)
  Rejected because: Adds infrastructure complexity and operational overhead; environment variables provide sufficient security for current scale
  When valid: When managing hundreds of secrets across multiple services or requiring advanced features like automatic rotation and audit logging
- Store credentials in configuration files excluded from version control (rejected)
  Rejected because: Requires file system access and complicates containerized deployments; less portable than environment variables
  When valid: Legacy systems with file-based configuration requirements

## Risks

- Environment variables may be logged or exposed through error messages or process inspection
  Mitigation: Implement logging filters to redact sensitive values; use secure error handling that does not expose environment details
  Owner: Engineering team
- Missing or misconfigured environment variables cause runtime failures
  Mitigation: Implement startup validation that checks for required environment variables and fails fast with clear error messages
  Owner: DevOps team
- Environment variable injection attacks in containerized environments
  Mitigation: Use platform-native secrets management; restrict container runtime permissions; validate environment variable formats
  Owner: Security team

## Implementation Notes

- Create .env.example template file documenting required environment variables without actual values
- Implement startup validation that checks for DATABASE_URL and PORT before initializing connection pool
- Use dotenv or similar library for local development; rely on platform-provided environment variables in production
- Document environment variable requirements in deployment runbooks and README files
- Consider using typed configuration objects that parse and validate environment variables at startup

## Continuation Context


Verify commands:
- grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'
- grep -r 'process\.env\.PORT' server/src/ | grep -v 'node_modules'
- ! grep -rE '(postgresql://|postgres://).*:.*@' server/src/ --include='*.ts' --include='*.js' | grep -v 'process.env'

Accept when:
- All database connection strings are sourced from process.env.DATABASE_URL
- Server port configuration is sourced from process.env.PORT
- No hardcoded database credentials or connection strings appear in source code

## Enforcement

- Verified by: Static code analysis scanning for hardcoded credentials patterns
- Verified by: Code review checklist requiring environment variable usage for secrets
- Verified by: Pre-commit hooks detecting credential patterns in staged files
- Violation handling: Automated CI pipeline failure on detection of hardcoded credentials
- Violation handling: Security team notification for credential exposure incidents
- Violation handling: Immediate credential rotation if exposure is detected in version control
- Exception process: Submit exception request documenting specific use case and security justification
- Exception process: Security team review and approval required for any credential handling exceptions
- Exception process: Approved exceptions must be documented in code comments with ticket reference