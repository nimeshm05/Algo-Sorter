# Externalize Database Credentials via Environment Variables: Database Connection Strings

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The service requires database connectivity using PostgreSQL (pg library) with connection parameters that vary across deployment environments
- Hardcoding credentials in source code creates security vulnerabilities and prevents environment-specific configuration
- The application runs in multiple contexts (development, production) requiring different connection strings and port bindings
- The service handles visitor tracking with persistent state requiring secure database access across request boundaries

## Problem Statement

Services that embed database credentials or configuration directly in source code cannot be deployed securely across multiple environments, expose sensitive connection strings in version control, and require code changes for environment-specific configuration adjustments.

## Decision

1. MUST: Database connection strings MUST be sourced from process.env.DATABASE_URL rather than hardcoded in application code

## Policy Block

- MUST Database connection strings MUST be sourced from process.env.DATABASE_URL rather than hardcoded in application code

In scope:
- All database connection configuration including connection strings, credentials, and SSL settings
- Server runtime configuration such as port bindings and host addresses
- Any configuration value that differs between development, staging, and production environments

Out of scope:
- Application-level constants that are identical across all environments
- Public API endpoint paths and route definitions
- Static CORS allowed origins that are environment-independent

Exceptions:
- EXC-001: Local development environments may use .env files with non-production credentials for convenience

## Rationale

- Evidence shows process.env.DATABASE_URL and process.env.PORT are the sole runtime configuration sources, establishing environment variable pattern
- The PostgreSQL Pool initialization directly consumes process.env.DATABASE_URL, demonstrating externalized credential management at service boundary
- This pattern enables the same codebase to operate across multiple deployment targets without code modification
- Separating configuration from code aligns with twelve-factor app principles and reduces credential exposure risk

## Consequences

Positive:
- Credentials remain external to version control, reducing security exposure and audit surface
- The same application artifact can be deployed to multiple environments by changing environment variables only
- Platform providers can inject configuration at deployment time without application code awareness
- Configuration changes do not require code commits, reducing deployment friction

Negative:
- Runtime failures occur if required environment variables are missing, requiring explicit validation logic
- Environment variable management becomes a deployment concern requiring documentation and tooling
- Debugging configuration issues requires access to deployment environment rather than inspecting code
- Local development setup requires additional steps to configure environment variables

## Alternatives

- Hardcode database credentials directly in source code (rejected)
  Rejected because: Exposes credentials in version control, prevents multi-environment deployment, violates security best practices
  When valid: Never valid for production systems; only acceptable for throwaway prototypes
- Use configuration files (e.g., config.json) committed to repository with environment-specific values (rejected)
  Rejected because: Still requires credentials in version control unless files are gitignored, complicating deployment
  When valid: Valid for non-sensitive configuration like feature flags or public API endpoints
- Use secrets management service (e.g., HashiCorp Vault, AWS Secrets Manager) with runtime fetching (deferred)
  Rejected because: Adds infrastructure complexity and dependencies not justified by current evidence of single-service architecture
  When valid: Valid for multi-service architectures requiring secret rotation, audit trails, and centralized secret management

## Risks

- Missing or misconfigured environment variables cause runtime failures that are not caught until deployment
  Mitigation: Implement startup validation that checks for required environment variables and fails fast with clear error messages
  Owner: Engineering team
- Environment variables may be logged or exposed through error messages, leaking credentials
  Mitigation: Implement logging filters to redact environment variable values and avoid echoing configuration in error responses
  Owner: Engineering team
- Developers may accidentally commit .env files containing real credentials to version control
  Mitigation: Add .env to .gitignore, use pre-commit hooks to scan for credential patterns, provide .env.example template
  Owner: Engineering team

## Implementation Notes

- Create a .env.example file documenting all required environment variables with placeholder values for developer onboarding
- Add startup validation that checks process.env.DATABASE_URL and process.env.PORT are defined before initializing the connection pool
- Document environment variable requirements in deployment runbooks and README with clear examples for each environment
- Consider using a library like dotenv for local development to load .env files, but ensure .env is in .gitignore

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' server/src/ | grep -E '(DATABASE_URL|PORT)' # Verify environment variable usage
- grep -r 'postgresql://\|postgres://\|password.*=\|user.*=' server/src/ | grep -v process.env # Detect hardcoded credentials
- test -f .gitignore && grep -q '\.env' .gitignore # Verify .env is gitignored

Accept when:
- All database connection parameters are sourced from process.env.DATABASE_URL with no hardcoded credentials in source files
- Server port binding uses process.env.PORT and no port numbers are hardcoded in application initialization
- No credential strings or connection URLs appear in version control history or current source files

## Enforcement

- Verified by: Pre-commit hooks scanning for credential patterns and hardcoded connection strings
- Verified by: Code review checklist requiring verification that configuration is externalized
- Verified by: Static analysis tools detecting process.env usage patterns and flagging hardcoded secrets
- Violation handling: Automated CI pipeline fails if credential patterns are detected in source code
- Violation handling: Security team notification triggered for commits containing potential credential exposure
- Violation handling: Immediate credential rotation required if credentials are committed to version control
- Exception process: Exceptions require written justification and security team approval
- Exception process: Exception approval must document compensating controls and time-bound remediation plan
- Exception process: All exceptions logged in security exception register with quarterly review