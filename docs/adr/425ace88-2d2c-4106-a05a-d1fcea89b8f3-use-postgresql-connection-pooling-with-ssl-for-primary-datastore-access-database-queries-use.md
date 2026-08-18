# Use PostgreSQL Connection Pooling with SSL for Primary Datastore Access: Database Queries Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application requires persistent storage for visitor tracking data with operations including SELECT, INSERT, and UPDATE queries against a relational database
- Database connection credentials are sourced from runtime environment variables (process.env.DATABASE_URL) rather than hardcoded configuration files
- The pg library Pool class is instantiated with SSL configuration requiring encrypted connections to the PostgreSQL database
- Multiple concurrent HTTP requests to endpoints like '/' and '/hello' necessitate efficient database connection management to avoid connection exhaustion
- The application runs in an environment where DATABASE_URL and PORT are externally configured, suggesting a cloud or containerized deployment model

## Problem Statement

Without a standardized approach to database connection management and configuration sourcing, applications risk connection pool exhaustion, inconsistent security posture across environments, and tight coupling between application code and infrastructure-specific connection details. The pattern must balance connection efficiency, security requirements, and environment portability.

## Decision

1. MUST: All database queries MUST use parameterized statements with $1, $2, etc. placeholders and separate values arrays

## Policy Block

- MUST All database queries MUST use parameterized statements with $1, $2, etc. placeholders and separate values arrays

In scope:
- All PostgreSQL database connections in Node.js/TypeScript server applications
- Visitor tracking and stateful data persistence operations
- Express.js route handlers performing database queries
- Cloud-hosted or containerized deployments requiring SSL database connections

Out of scope:
- In-memory caching or session storage mechanisms
- Non-PostgreSQL database systems (MySQL, MongoDB, etc.)
- Client-side data storage or browser-based persistence
- Static file serving or stateless API endpoints with no database interaction

Exceptions:
- EXC-001: Development or local testing environments where SSL overhead is unnecessary
- EXC-002: Migration scripts or one-time administrative tasks requiring direct Client connections

## Rationale

- The pg Pool class provides automatic connection management, preventing resource exhaustion under concurrent load as evidenced by multiple async route handlers accessing the same pool instance
- Environment-based configuration (process.env.DATABASE_URL) enables deployment portability across development, staging, and production without code changes
- SSL with rejectUnauthorized: false accommodates cloud database providers (like Heroku Postgres) that use self-signed certificates while maintaining encrypted transport
- Parameterized queries with separate values arrays prevent SQL injection vulnerabilities as demonstrated in all INSERT and UPDATE operations in the evidence

## Consequences

Positive:
- Connection pooling reduces database connection overhead and improves throughput for concurrent requests
- Environment-based configuration enables seamless deployment across multiple environments without code modification
- SSL encryption protects sensitive data in transit between application and database servers
- Parameterized queries eliminate SQL injection attack vectors while maintaining query readability

Negative:
- Setting rejectUnauthorized: false reduces SSL security by accepting self-signed certificates, creating potential MITM vulnerability
- Pool configuration defaults may not be optimal for all workload patterns, requiring tuning for high-concurrency scenarios
- Environment variable dependency creates runtime configuration complexity and potential startup failures if DATABASE_URL is misconfigured
- Single pool instance becomes a shared resource requiring careful error handling to prevent connection leaks

## Alternatives

- Use direct pg.Client instances for each database operation without connection pooling (rejected)
  Rejected because: Direct Client connections create excessive overhead for each request and risk connection exhaustion under concurrent load, as the application handles multiple simultaneous HTTP requests
  When valid: Only appropriate for single-operation scripts or administrative tools with no concurrency requirements
- Use an ORM like TypeORM or Prisma for database abstraction with built-in connection management (rejected)
  Rejected because: The evidence shows direct SQL queries with pool.query() calls, indicating a preference for lightweight database access without ORM abstraction overhead
  When valid: Valid for applications requiring complex entity relationships, migrations, or type-safe query builders
- Store database credentials in configuration files or hardcoded constants (rejected)
  Rejected because: Hardcoded credentials prevent environment portability and create security risks through credential exposure in version control
  When valid: Never valid for production systems; only acceptable in isolated local development with dummy data

## Risks

- SSL configuration with rejectUnauthorized: false exposes the application to man-in-the-middle attacks if certificate validation is bypassed
  Mitigation: Use proper CA certificates in production environments and restrict rejectUnauthorized: false to cloud providers with verified infrastructure
  Owner: Security team and infrastructure team
- Missing DATABASE_URL environment variable causes application startup failure with unclear error messages
  Mitigation: Implement startup validation to check for required environment variables and provide clear error messages before attempting database connection
  Owner: Engineering team
- Connection pool exhaustion under high load if pool size limits are not configured appropriately
  Mitigation: Configure explicit pool size limits based on database server capacity and implement connection timeout handling with proper error responses
  Owner: Engineering team and database administrator

## Implementation Notes

- Initialize the Pool instance at module level in server/src/index.ts before defining route handlers to ensure single pool reuse
- Configure pool size limits explicitly using max and min parameters based on expected concurrent request volume and database connection limits
- Implement graceful shutdown handling to close the pool on application termination using pool.end() in process signal handlers
- Add startup validation to verify DATABASE_URL is set and test database connectivity before binding to PORT to fail fast on misconfiguration

## Continuation Context


Verify commands:
- grep -r 'new Pool' server/src --include='*.ts' | grep -q 'connectionString.*process.env.DATABASE_URL'
- grep -r 'pool.query' server/src --include='*.ts' | grep -q 'text:.*values:'
- grep -r 'new Pool' server/src --include='*.ts' | grep -q 'ssl.*rejectUnauthorized'

Accept when:
- All PostgreSQL connections use Pool instances with connectionString sourced from process.env.DATABASE_URL
- All database queries use parameterized statements with separate text and values properties
- SSL configuration is present in Pool initialization with rejectUnauthorized explicitly set

## Enforcement

- Verified by: Automated code review checks scanning for direct Client usage or non-parameterized queries
- Verified by: CI pipeline verification commands checking for Pool instantiation patterns and SSL configuration
- Verified by: Runtime monitoring of database connection pool metrics to detect connection leaks or exhaustion
- Violation handling: CI build failures block merge for code using direct Client instances or missing parameterization
- Violation handling: Code review checklist requires explicit verification of connection pooling and SSL configuration
- Violation handling: Production monitoring alerts trigger incident response for connection pool exhaustion or SSL handshake failures
- Exception process: Submit exception request to tech lead with justification for alternative database access pattern
- Exception process: Document exception in ADR amendments with specific scope and time-bound validity
- Exception process: Require security team review for any SSL configuration changes affecting production environments