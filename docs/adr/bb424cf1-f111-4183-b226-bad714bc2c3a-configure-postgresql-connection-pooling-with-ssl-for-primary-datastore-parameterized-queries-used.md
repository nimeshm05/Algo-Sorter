# Configure PostgreSQL Connection Pooling with SSL for Primary Datastore: Parameterized Queries Used

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application uses PostgreSQL as its primary datastore, accessed through the 'pg' library with connection pooling configured via environment variables
- Database connections require SSL/TLS encryption to protect data in transit, though certificate validation is disabled (rejectUnauthorized: false)
- The visitor tracking service performs multiple database operations (SELECT, INSERT, UPDATE) requiring connection reuse and efficient resource management
- Database credentials and connection strings are externalized through environment variables (DATABASE_URL) following twelve-factor app principles
- The application exposes HTTP endpoints that trigger database queries, necessitating secure and performant database access patterns

## Problem Statement

Without standardized connection pooling and SSL configuration for PostgreSQL access, applications risk connection exhaustion, inconsistent security postures, and exposure of sensitive data in transit. The pattern observed shows SSL enabled but with certificate validation disabled, creating a security-convenience tradeoff that must be explicitly governed.

## Decision

1. MUST: Parameterized queries MUST be used for all database operations involving user input or dynamic values to prevent SQL injection

## Policy Block

- MUST Parameterized queries MUST be used for all database operations involving user input or dynamic values to prevent SQL injection

In scope:
- All PostgreSQL database connections in Node.js/TypeScript applications
- Primary datastore access patterns including connection initialization and query execution
- Database operations triggered by HTTP request handlers or background jobs
- Environment-based configuration for database credentials and connection parameters

Out of scope:
- Non-PostgreSQL databases (MySQL, MongoDB, Redis, etc.)
- Database migration scripts or schema management tools
- Database administration tools or direct psql client usage
- Read replicas or secondary database connections with different security requirements

Exceptions:
- EXC-001: Connecting to managed PostgreSQL services (Heroku, AWS RDS, etc.) that provide SSL but use self-signed certificates
- EXC-002: Local development environments where SSL is not configured on the PostgreSQL instance

## Rationale

- Connection pooling prevents resource exhaustion by reusing database connections across multiple requests, critical for applications handling concurrent HTTP traffic
- SSL encryption protects sensitive data (visitor IP addresses, timestamps) in transit between application and database, meeting baseline security compliance requirements
- Parameterized queries with the pg library's text/values pattern prevent SQL injection vulnerabilities by separating query structure from user-supplied data
- Environment-based configuration enables secure credential management and supports different database instances across development, staging, and production environments

## Consequences

Positive:
- Prevents connection exhaustion and improves application performance through efficient connection reuse
- Protects data in transit with SSL/TLS encryption, reducing risk of man-in-the-middle attacks
- Eliminates SQL injection vulnerabilities through consistent use of parameterized queries
- Enables secure credential management and environment-specific database configuration without code changes

Negative:
- Disabling SSL certificate validation (rejectUnauthorized: false) creates vulnerability to man-in-the-middle attacks if network is compromised
- Connection pooling adds complexity to application lifecycle management (pool initialization, graceful shutdown)
- Environment variable dependencies create runtime configuration requirements that must be validated before application startup
- SSL overhead adds latency to database operations, though typically negligible compared to query execution time

## Alternatives

- Use direct PostgreSQL connections without pooling (new Client() per request) (rejected)
  Rejected because: Creates connection exhaustion under load and degrades performance due to connection establishment overhead for each request
  When valid: Only valid for single-use scripts or batch jobs that execute one query and exit
- Disable SSL entirely for database connections (rejected)
  Rejected because: Exposes all database traffic in plaintext, violating security compliance requirements and creating unacceptable risk for production environments
  When valid: Only acceptable in isolated local development environments with no sensitive data
- Use ORM (TypeORM, Sequelize, Prisma) with built-in connection pooling and SSL configuration (deferred)
  Rejected because: Not rejected but not chosen; adds abstraction layer and dependencies that may not be needed for simple query patterns
  When valid: Valid for applications with complex data models, migrations, and relationships requiring ORM features

## Risks

- SSL certificate validation disabled (rejectUnauthorized: false) allows man-in-the-middle attacks if network security is compromised
  Mitigation: Document the specific hosting provider requirement, implement network-level security controls, and periodically review whether proper certificate validation can be enabled
  Owner: Security team and infrastructure team
- Missing or incorrect DATABASE_URL environment variable causes application startup failure with unclear error messages
  Mitigation: Implement explicit environment variable validation at startup with clear error messages, and document required configuration in deployment guides
  Owner: Engineering team
- Connection pool exhaustion if pool size is not tuned for application concurrency and database connection limits
  Mitigation: Configure pool size based on expected concurrent requests and database connection limits, implement connection timeout monitoring and alerting
  Owner: Engineering team and operations team

## Implementation Notes

- Initialize the Pool instance once at application startup and reuse it across all request handlers to avoid creating multiple pools
- Configure pool size using environment variables (e.g., PGMAXCONNECTIONS) to tune for specific deployment environments and database limits
- Implement graceful shutdown by calling pool.end() to drain connections before process termination
- Use async/await with pool.query() to handle database operations and ensure proper error handling for connection failures
- For managed database services requiring rejectUnauthorized: false, add inline comments explaining the hosting provider's certificate configuration

## Continuation Context


Verify commands:
- grep -r 'new Pool' --include='*.ts' --include='*.js' | grep -E 'ssl.*rejectUnauthorized'
- grep -r 'pool\.query.*\$[0-9]' --include='*.ts' --include='*.js' | wc -l
- grep -r 'process\.env\.DATABASE_URL' --include='*.ts' --include='*.js'

Accept when:
- All PostgreSQL Pool instantiations include ssl configuration object with explicit rejectUnauthorized setting
- All database queries with dynamic values use parameterized queries (text and values properties) with no string concatenation
- DATABASE_URL is sourced from process.env and not hardcoded in any source files

## Enforcement

- Verified by: Static code analysis scanning for Pool instantiation patterns and SSL configuration
- Verified by: Code review checklist requiring verification of parameterized query usage
- Verified by: Security scanning tools detecting hardcoded credentials or connection strings
- Violation handling: CI pipeline fails if grep verification commands detect missing SSL configuration or non-parameterized queries
- Violation handling: Code review blocks merge if database connections lack proper pooling or SSL settings
- Violation handling: Security team notified for manual review if violations are detected in production code
- Exception process: Submit exception request to security team with justification for specific hosting environment or development scenario
- Exception process: Document approved exceptions in code comments and deployment configuration files
- Exception process: Review exceptions quarterly to determine if proper SSL certificate validation can be enabled