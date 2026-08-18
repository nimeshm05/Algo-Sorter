# Use PostgreSQL with Connection Pooling for Data Access: Queries User Supplied

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application requires persistent storage for visitor tracking data including IP addresses, visit counts, and timestamps
- Database connection management is needed for a Node.js Express application handling concurrent HTTP requests
- The deployment environment provides database credentials via environment variables (DATABASE_URL) with SSL requirements
- The application performs basic CRUD operations (SELECT, INSERT, UPDATE) on a visitors table to maintain session state

## Problem Statement

The application needs a reliable, production-ready data access pattern that efficiently manages database connections across multiple concurrent requests while supporting parameterized queries to prevent SQL injection and maintaining connection security through SSL.

## Decision

1. MUST: All queries with user-supplied data MUST use parameterized queries with the text/values pattern to prevent SQL injection

## Policy Block

- MUST All queries with user-supplied data MUST use parameterized queries with the text/values pattern to prevent SQL injection

## Rationale

- The evidence shows consistent use of the pg library's Pool class for connection management, indicating a deliberate choice for production-grade connection pooling
- Parameterized queries with $1, $2, $3 placeholders are used throughout for INSERT and UPDATE operations, demonstrating security-conscious data access patterns
- SSL configuration with rejectUnauthorized: false suggests deployment to a managed database service (likely Heroku Postgres or similar) that uses self-signed certificates
- The pattern supports both simple queries (SELECT SUM) and parameterized queries, providing flexibility for different query complexity levels

## Consequences

Positive:
- Connection pooling reduces overhead by reusing database connections across requests, improving application performance under load
- Parameterized queries eliminate SQL injection vulnerabilities by separating query structure from user data
- SSL enforcement protects data in transit between application and database servers
- The Pool abstraction simplifies connection lifecycle management and automatic connection recovery

Negative:
- Setting rejectUnauthorized to false reduces SSL security by accepting self-signed certificates, creating potential MITM vulnerability
- Connection pool configuration is not explicitly tuned (no max connections, idle timeout), relying on pg library defaults which may not suit all workloads
- No explicit error handling or connection retry logic is visible in the evidence, potentially leading to unhandled promise rejections
- Direct access to process.env at module initialization time prevents runtime configuration changes without application restart

## Alternatives

- Use direct database connections without pooling (new Client() per request) (rejected)
  Rejected because: Direct connections create excessive overhead for each HTTP request and do not scale well under concurrent load, leading to connection exhaustion
  When valid: Only appropriate for single-threaded batch scripts or one-off administrative tasks
- Use an ORM layer (e.g., Sequelize, TypeORM) instead of raw SQL queries (rejected)
  Rejected because: The application's query patterns are simple CRUD operations that do not justify the complexity and performance overhead of a full ORM
  When valid: Valid for applications with complex domain models, relationships, and migrations requiring schema versioning
- Use query builder library (e.g., Knex.js) for SQL construction (deferred)
  Rejected because: Not rejected; could provide better query composition and migration support while maintaining performance
  When valid: Appropriate if query complexity grows or schema migration management becomes necessary

## Risks

- SSL configuration with rejectUnauthorized: false exposes the application to man-in-the-middle attacks if network is compromised
  Mitigation: Obtain proper SSL certificates for the database or use certificate pinning; document why self-signed certificates are accepted in production
  Owner: engineering team
- Unhandled promise rejections from pool.query() calls may crash the Node.js process or leave requests hanging
  Mitigation: Implement try-catch blocks around all async database operations and add global unhandled rejection handlers
  Owner: engineering team
- Default connection pool settings may not handle traffic spikes, leading to connection exhaustion or timeout errors
  Mitigation: Explicitly configure pool size, connection timeout, and idle timeout based on expected load and database connection limits
  Owner: engineering team

## Implementation Notes

- Initialize the Pool instance once at module level and reuse it across all route handlers to maintain connection pooling benefits
- Always use parameterized queries (text/values pattern) when incorporating user input, request parameters, or any external data into SQL statements
- Wrap all pool.query() calls in try-catch blocks and implement appropriate error responses for database failures
- Consider adding connection pool event listeners (pool.on('error')) to log connection issues and monitor pool health

## Continuation Context


Verify commands:
- grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL' && echo 'Pool initialization found'
- grep -r 'pool.query' server/src/ | grep -E '\$[0-9]' && echo 'Parameterized queries detected'
- grep -r 'ssl.*rejectUnauthorized' server/src/ && echo 'SSL configuration present'

Accept when:
- All database queries use the Pool instance rather than direct Client connections
- All queries containing user-supplied data use parameterized queries with positional placeholders ($1, $2, etc.)
- SSL configuration is explicitly defined in Pool initialization options

## Enforcement

- Verified by: Code review checking for Pool usage and parameterized query patterns
- Verified by: Static analysis scanning for SQL injection vulnerabilities and raw string concatenation in queries
- Verified by: Integration tests verifying database connectivity and query execution
- Violation handling: Pull requests introducing direct Client usage or non-parameterized queries must be rejected
- Violation handling: Security scanning tools should flag SQL concatenation patterns for immediate remediation
- Violation handling: Production deployments require passing integration tests that validate connection pool behavior
- Exception process: Exceptions for raw SQL (non-parameterized) queries require security review and documentation of why parameterization is not feasible
- Exception process: Alternative SSL configurations require infrastructure team approval and security assessment
- Exception process: All exceptions must be documented in code comments with ticket references and expiration dates