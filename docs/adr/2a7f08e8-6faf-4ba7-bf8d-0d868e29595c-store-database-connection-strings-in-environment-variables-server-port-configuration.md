# Store Database Connection Strings in Environment Variables: Server Port Configuration

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application requires database connectivity configuration that varies across deployment environments (development, staging, production)
- Hardcoding database credentials in source code creates security vulnerabilities and prevents environment-specific configuration
- The PostgreSQL connection pool initialization requires a connection string containing authentication credentials and endpoint information
- Runtime configuration must support SSL/TLS settings for secure database connections with configurable certificate validation

## Problem Statement

Database connection strings contain sensitive credentials that must not be committed to version control, while the application needs a consistent mechanism to configure database access across different deployment environments without code changes.

## Decision

1. SHOULD: Server port configuration SHOULD be sourced from process.env.PORT environment variable

## Policy Block

- SHOULD Server port configuration SHOULD be sourced from process.env.PORT environment variable

In scope:
- PostgreSQL database connection configuration
- Connection pool initialization parameters
- SSL/TLS certificate validation settings
- Server runtime port configuration

Out of scope:
- Application-level secrets unrelated to database connectivity
- Client-side configuration or API keys
- Build-time configuration that does not contain credentials
- Non-sensitive feature flags or application settings

## Rationale

- The evidence shows process.env.DATABASE_URL and process.env.PORT are used as runtime configuration sources in server/src/index.ts
- PostgreSQL Pool initialization explicitly references process.env.DATABASE_URL with SSL configuration, demonstrating environment-based secrets handling
- This pattern enables deployment across multiple environments without code modification while keeping credentials out of version control
- The pattern aligns with twelve-factor app methodology for configuration management and secrets handling

## Consequences

Positive:
- Database credentials remain outside version control, reducing security risk of credential exposure
- Environment-specific configuration enables seamless deployment across development, staging, and production
- Configuration changes do not require code modifications or redeployment
- SSL settings can be adjusted per environment without touching application code

Negative:
- Application startup fails if required environment variables are not set, requiring operational documentation
- Debugging connection issues requires access to environment configuration outside the codebase
- Local development setup requires additional steps to configure environment variables
- No compile-time validation of environment variable presence or format

## Alternatives

- Hardcode database connection strings directly in source code (rejected)
  Rejected because: Exposes credentials in version control and prevents environment-specific configuration without code changes
  When valid: Never valid for production systems with sensitive credentials
- Use configuration files (e.g., config.json) with credentials stored separately (rejected)
  Rejected because: Adds complexity of file management and still requires environment-specific file handling; environment variables are simpler for containerized deployments
  When valid: May be valid for legacy systems with established file-based configuration patterns
- Use dedicated secrets management service (e.g., HashiCorp Vault, AWS Secrets Manager) (deferred)
  Rejected because: Adds infrastructure complexity and dependencies; may be warranted as application scales
  When valid: Valid for large-scale production systems requiring secret rotation, audit trails, and centralized secrets management

## Risks

- Environment variables may be logged or exposed through error messages or monitoring tools
  Mitigation: Implement logging filters to redact DATABASE_URL values; configure monitoring tools to exclude environment variable dumps
  Owner: engineering team
- Missing or malformed DATABASE_URL causes runtime failures that may not be caught until deployment
  Mitigation: Add startup validation to check for required environment variables; implement health checks that verify database connectivity
  Owner: engineering team
- SSL configuration with rejectUnauthorized: false may accept invalid certificates in production
  Mitigation: Document SSL configuration requirements per environment; use rejectUnauthorized: true in production with proper certificate management
  Owner: engineering team

## Implementation Notes

- Create .env.example file documenting required environment variables (DATABASE_URL, PORT) without actual credentials
- Use dotenv or similar library for local development to load environment variables from .env file (excluded from version control)
- Document DATABASE_URL format: postgresql://username:password@host:port/database?sslmode=require
- Implement startup validation that checks for DATABASE_URL presence and attempts connection before accepting requests

## Continuation Context


Verify commands:
- grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'
- grep -r 'postgresql://' server/src/ | grep -v 'process.env' | grep -v 'node_modules' && echo 'FAIL: Hardcoded credentials found' || echo 'PASS: No hardcoded credentials'
- grep -r 'new Pool' server/src/ | grep -A 5 'connectionString' | grep 'process.env'

Accept when:
- All database connection strings are sourced from process.env.DATABASE_URL with no hardcoded credentials in source files
- PostgreSQL Pool initialization includes SSL configuration and references environment variables
- No grep matches for hardcoded postgresql:// connection strings outside of environment variable references

## Enforcement

- Verified by: Automated code review checks scanning for hardcoded connection strings or credentials
- Verified by: CI pipeline grep-based verification commands checking for process.env.DATABASE_URL usage
- Verified by: Security scanning tools detecting secrets in source code
- Violation handling: CI pipeline fails if hardcoded credentials are detected in source code
- Violation handling: Pull requests blocked until credentials are moved to environment variables
- Violation handling: Security alerts generated for any committed secrets requiring immediate remediation
- Exception process: No exceptions permitted for production code containing hardcoded database credentials
- Exception process: Test fixtures may use hardcoded test database URLs only for isolated unit tests with clearly marked test data
- Exception process: Documentation examples must use placeholder values (e.g., postgresql://user:pass@host:5432/db) not real credentials