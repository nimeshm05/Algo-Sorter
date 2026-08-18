# Use Environment Variables for Runtime Database Configuration: Applications Provide Default

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application requires database connection parameters at runtime to establish connectivity with PostgreSQL
- Configuration values must be externalized from source code to support multiple deployment environments (development, staging, production)
- The pg Pool constructor accepts a connectionString parameter that contains all necessary connection details including host, port, credentials, and database name
- Environment variables provide a standard mechanism for injecting configuration at process startup without code modification
- The PORT configuration enables the application server to bind to different network ports across environments

## Problem Statement

Applications need a secure, environment-agnostic method to configure database connections and server ports without hardcoding sensitive credentials or environment-specific values in source code, while maintaining compatibility with cloud platform deployment patterns that inject configuration through environment variables.

## Decision

1. MAY: Applications MAY provide default values for PORT when the environment variable is not set

## Policy Block

- MAY Applications MAY provide default values for PORT when the environment variable is not set

## Rationale

- The evidence shows explicit use of process.env.DATABASE_URL and process.env.PORT in server/src/index.ts, demonstrating a consistent pattern of runtime configuration sourcing
- The pg Pool instantiation with connectionString parameter and SSL configuration indicates cloud-hosted database connectivity requirements typical of PaaS deployments
- Environment variable-based configuration enables the same codebase to operate across multiple environments without modification, supporting continuous deployment practices
- This pattern aligns with twelve-factor app methodology for configuration management and is widely supported by cloud platforms and container orchestration systems

## Consequences

Positive:
- Database credentials remain external to version control, reducing security exposure from accidental credential commits
- The same application binary can be deployed to multiple environments by changing only environment variables
- Configuration changes do not require code recompilation or redeployment, only process restart
- Cloud platform compatibility is maintained as most PaaS providers inject configuration through environment variables

Negative:
- Environment variable management becomes a deployment concern requiring coordination between development and operations teams
- Missing or misconfigured environment variables cause runtime failures rather than compile-time errors
- Debugging configuration issues requires access to the runtime environment rather than inspecting source code alone
- Local development requires developers to manually set environment variables or use .env files with additional tooling

## Alternatives

- Hardcode database connection strings and port numbers directly in application source code (rejected)
  Rejected because: Hardcoded credentials create security vulnerabilities, prevent environment portability, and require code changes for configuration updates
  When valid: Only acceptable for throwaway prototypes or single-environment applications with no security requirements
- Use configuration files (JSON, YAML, TOML) stored in the repository or mounted at runtime (rejected)
  Rejected because: Configuration files in version control expose credentials; externally mounted files add deployment complexity compared to environment variables
  When valid: Appropriate for complex configuration with nested structures that exceed environment variable capabilities
- Retrieve configuration from a centralized configuration service (Consul, etcd, AWS Parameter Store) (rejected)
  Rejected because: Adds infrastructure dependencies and complexity; the evidence shows a simpler application that does not require distributed configuration management
  When valid: Justified for microservices architectures requiring dynamic configuration updates without process restarts

## Risks

- Missing environment variables at runtime cause application startup failures with unclear error messages
  Mitigation: Implement startup validation that checks for required environment variables and provides clear error messages indicating which variables are missing
  Owner: engineering team
- Environment variables may be logged or exposed through error messages, leaking sensitive credentials
  Mitigation: Implement logging filters to redact environment variable values and avoid echoing configuration in error responses
  Owner: engineering team
- SSL configuration with rejectUnauthorized: false disables certificate validation, creating man-in-the-middle attack vulnerability
  Mitigation: Use proper SSL certificate validation in production environments; only disable for development or when using self-signed certificates with explicit security review
  Owner: security team

## Implementation Notes

- Use a .env file with dotenv library for local development to avoid manually setting environment variables in each shell session
- Document all required environment variables in README.md with example values (using placeholder credentials, not real ones)
- Consider using a configuration validation library like joi or zod to validate environment variables at startup with clear error messages
- For the DATABASE_URL format, follow the PostgreSQL connection string specification: postgresql://user:password@host:port/database

## Continuation Context


Verify commands:
- grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'
- grep -r 'process\.env\.PORT' server/src/ | grep -v 'node_modules'
- grep -rE '(postgresql://|postgres://).*@.*:.*/' server/src/ --include='*.ts' --include='*.js' | grep -v process.env || echo 'No hardcoded connection strings found'

Accept when:
- All database connection strings are sourced from process.env.DATABASE_URL with no hardcoded credentials in source files
- Server port configuration is sourced from process.env.PORT
- No grep matches for hardcoded PostgreSQL connection strings containing credentials in TypeScript or JavaScript source files

## Enforcement

- Verified by: Automated code review checks scanning for hardcoded connection strings or credentials in pull requests
- Verified by: Static analysis tools configured to flag direct credential usage in source code
- Verified by: Manual security review during code review process checking for proper environment variable usage
- Violation handling: Pull requests containing hardcoded credentials are automatically flagged and blocked from merging
- Violation handling: Security team is notified of any credential exposure for immediate rotation
- Violation handling: Developers are required to remediate violations by externalizing configuration to environment variables before merge approval
- Exception process: Exceptions for test fixtures or mock data may be granted if credentials are clearly fake and documented as non-functional
- Exception process: Exception requests must be submitted to security team with justification and risk assessment
- Exception process: Approved exceptions must be documented in code comments explaining why the pattern deviation is necessary