# Source Runtime Configuration from Environment Variables for Database and Service Binding: Database Connection Pools

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application requires runtime configuration for database connection strings and service port bindings that vary across deployment environments (development, staging, production)
- Hardcoding sensitive connection strings or environment-specific values in source code creates security vulnerabilities and deployment inflexibility
- The server/src/index.ts file demonstrates use of process.env.DATABASE_URL and process.env.PORT to externalize configuration from application code
- PostgreSQL connection pooling requires SSL configuration and connection parameters that must be environment-specific without code changes

## Problem Statement

Applications need a secure, flexible mechanism to obtain runtime configuration values such as database credentials, API endpoints, and service ports without embedding sensitive data in source code or requiring recompilation for different deployment environments.

## Decision

1. SHOULD: Database connection pools SHOULD be initialized with environment-sourced configuration including SSL settings appropriate to the deployment context

## Policy Block

- SHOULD Database connection pools SHOULD be initialized with environment-sourced configuration including SSL settings appropriate to the deployment context

In scope:
- All database connection configuration including connection strings, credentials, and SSL parameters
- Service binding configuration including HTTP ports and host addresses
- Environment-specific feature flags and operational parameters
- Third-party API endpoints and authentication tokens

Out of scope:
- Application constants that are identical across all environments
- Build-time configuration embedded during compilation or bundling
- Static resource paths and internal routing configuration

Exceptions:
- EXC-001: Local development environments where developers use .env files with non-production test credentials

## Rationale

- The evidence shows process.env.DATABASE_URL and process.env.PORT usage in server/src/index.ts, demonstrating externalized configuration for PostgreSQL connection pooling and service binding
- Environment variable sourcing enables the same application binary to operate across multiple deployment contexts without code modification or recompilation
- Separating configuration from code reduces the risk of credential exposure in version control systems and supports secure secret management practices
- The pattern aligns with twelve-factor application principles for configuration management and supports containerized deployment models

## Consequences

Positive:
- Sensitive credentials remain external to source code, reducing exposure risk in version control and code review processes
- Single application artifact can be deployed across multiple environments with environment-specific configuration injection
- Configuration changes do not require application recompilation or redeployment, only process restart
- Integration with secret management systems (Vault, AWS Secrets Manager, Kubernetes Secrets) becomes straightforward through environment variable injection

Negative:
- Runtime failures occur if required environment variables are missing, requiring robust startup validation and error handling
- Debugging configuration issues becomes more complex as values are not visible in source code
- Environment variable management across multiple services and environments introduces operational overhead
- Type safety and validation of configuration values must be implemented explicitly at runtime rather than compile time

## Alternatives

- Hardcode configuration values directly in source code with separate branches or builds per environment (rejected)
  Rejected because: Creates security vulnerabilities by exposing credentials in version control; requires separate builds per environment; violates secure coding practices
  When valid: Never valid for production systems with sensitive credentials
- Use configuration files (JSON, YAML) deployed alongside application with environment-specific values (rejected)
  Rejected because: Configuration files must still be managed outside version control for sensitive values; environment variables provide simpler integration with container orchestration and secret management
  When valid: Valid for complex hierarchical configuration where environment variables become unwieldy, combined with secret injection
- Fetch configuration from remote configuration service at runtime (Consul, etcd, AWS Parameter Store) (deferred)
  Rejected because: Adds complexity and external dependencies for simple use cases; introduces bootstrap problem for service discovery credentials
  When valid: Valid for large-scale distributed systems requiring dynamic configuration updates without restarts

## Risks

- Application fails to start or exhibits runtime errors when required environment variables are undefined or malformed
  Mitigation: Implement explicit configuration validation at application startup; fail fast with clear error messages identifying missing variables; provide .env.example templates
  Owner: Engineering team
- Environment variables may be logged or exposed through error messages, stack traces, or monitoring systems
  Mitigation: Implement sanitization in logging frameworks to redact sensitive values; configure error reporting to exclude environment details in production
  Owner: Security team
- Inconsistent environment variable naming or missing documentation leads to deployment failures and operational confusion
  Mitigation: Establish naming conventions (e.g., SCREAMING_SNAKE_CASE); maintain centralized documentation of required variables; use infrastructure-as-code to enforce consistency
  Owner: DevOps team

## Implementation Notes

- Access process.env.DATABASE_URL and process.env.PORT at application initialization; validate presence and format before establishing database connections or starting HTTP servers
- For PostgreSQL connections, construct Pool configuration objects with connectionString from environment variables, including SSL settings (rejectUnauthorized: false shown in evidence)
- Consider using a configuration validation library to enforce required variables and type constraints at startup rather than discovering issues at runtime
- Document all required environment variables in README or deployment documentation; provide .env.example with placeholder values for local development

## Continuation Context


Verify commands:
- grep -r 'process\.env\.' server/src/ | grep -E '(DATABASE_URL|PORT)' # Verify environment variable usage
- grep -r 'connectionString.*process\.env' server/src/ # Confirm database connection uses env vars
- ! grep -r 'postgresql://.*:.*@' server/src/ --include='*.ts' --include='*.js' # Ensure no hardcoded connection strings

Accept when:
- All database connection strings and service ports are sourced from process.env variables in server/src/index.ts
- No hardcoded credentials or connection strings appear in TypeScript or JavaScript source files
- Application startup code validates presence of required environment variables before initializing database pools or HTTP servers

## Enforcement

- Verified by: Automated code review checks scanning for hardcoded credentials and connection strings in pull requests
- Verified by: Static analysis tools (e.g., git-secrets, truffleHog) integrated into CI pipeline to detect credential patterns
- Verified by: Manual security review of configuration management during architecture reviews
- Violation handling: CI pipeline fails if hardcoded credentials or connection strings are detected in source code
- Violation handling: Pull requests containing configuration violations are blocked from merge until remediated
- Violation handling: Security team notification triggered for credential exposure incidents; immediate credential rotation required
- Exception process: Exception requests must be submitted to security team with justification and risk assessment
- Exception process: Approved exceptions documented in ADR exception log with expiration date and compensating controls
- Exception process: Exceptions reviewed quarterly; expired exceptions revert to standard enforcement