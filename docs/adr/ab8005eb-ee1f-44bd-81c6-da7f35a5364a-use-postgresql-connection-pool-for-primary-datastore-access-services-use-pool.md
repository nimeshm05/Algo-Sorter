# Use PostgreSQL Connection Pool for Primary Datastore Access: Services Use Pool

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The service requires persistent storage for visitor tracking data including IP addresses, visit counts, and timestamps
- Database connections are established from a Node.js Express application that serves HTTP endpoints requiring real-time data access
- The application runs in a cloud environment where DATABASE_URL is provided via environment variables with SSL requirements
- Multiple concurrent HTTP requests necessitate efficient connection management to avoid exhausting database resources
- The pg library provides connection pooling capabilities for PostgreSQL databases in Node.js environments

## Problem Statement

Services require reliable, performant access to PostgreSQL databases while managing connection lifecycle efficiently across concurrent requests. Without connection pooling, each request would create and destroy database connections, leading to resource exhaustion, increased latency, and potential connection limit violations.

## Decision

1. MAY: Services MAY use pool.query() with either string queries or query configuration objects containing text and values properties

## Policy Block

- MAY Services MAY use pool.query() with either string queries or query configuration objects containing text and values properties

## Rationale

- The evidence shows a Pool instance created with DATABASE_URL and SSL configuration, demonstrating connection pooling as the established pattern for datastore access
- Multiple pool.query() invocations across different route handlers (SELECT SUM, SELECT with WHERE, INSERT, UPDATE) confirm the pool serves as the central data access mechanism
- Parameterized queries with $1, $2, $3 placeholders and values arrays appear consistently, indicating a security-conscious approach to query construction
- The pattern supports the service boundary architecture by centralizing database connection management at the application initialization layer

## Consequences

Positive:
- Connection pooling reduces latency and resource consumption by reusing established database connections across requests
- Parameterized queries prevent SQL injection vulnerabilities by separating query structure from user-supplied values
- Centralized pool configuration simplifies SSL and connection string management through environment variables
- The pg library provides battle-tested connection lifecycle management and error handling for PostgreSQL

Negative:
- Connection pool exhaustion can occur if queries are slow or handlers fail to release connections properly
- SSL configuration with rejectUnauthorized: false reduces security by accepting self-signed certificates
- The pool instance becomes a singleton dependency that must be carefully managed during application shutdown
- Debugging connection-related issues requires understanding pool internals and connection state management

## Alternatives

- Use direct Client connections without pooling for each request (rejected)
  Rejected because: Creates and destroys connections for every request, leading to high latency, resource exhaustion, and potential connection limit violations under concurrent load
  When valid: Only appropriate for single-request scripts or batch jobs where connection reuse is not beneficial
- Use an ORM like TypeORM or Sequelize with built-in connection pooling (rejected)
  Rejected because: Adds abstraction overhead and dependencies when the application only requires simple parameterized queries without complex object mapping
  When valid: Valid for applications with complex domain models, migrations, and relationships requiring ORM features
- Use a connection proxy or external pooler like PgBouncer (deferred)
  Rejected because: Not rejected; represents a complementary approach that could be added for additional connection management at the infrastructure layer
  When valid: Appropriate when multiple application instances require centralized connection pooling or when database connection limits are reached

## Risks

- Connection pool exhaustion if slow queries or unhandled errors prevent connection release
  Mitigation: Implement query timeouts, monitor pool metrics, and ensure proper error handling in all async route handlers
  Owner: engineering team
- SSL configuration with rejectUnauthorized: false exposes the service to man-in-the-middle attacks
  Mitigation: Obtain proper SSL certificates for the database or configure certificate validation with appropriate CA certificates
  Owner: infrastructure team
- Database credentials in environment variables could be exposed through logging or error messages
  Mitigation: Use secrets management systems, sanitize error outputs, and restrict access to environment variable configuration
  Owner: security team

## Implementation Notes

- Initialize the Pool instance once at application startup before registering Express routes to ensure availability
- Configure pool size limits based on expected concurrent request volume and database connection limits
- Implement graceful shutdown by calling pool.end() to drain connections before process termination
- Use try-catch blocks around all pool.query() calls to handle database errors and prevent connection leaks
- Consider adding connection pool monitoring and alerting to detect exhaustion or performance degradation

## Continuation Context


Verify commands:
- grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'
- grep -r 'pool\.query' server/src/ | grep -q '\$[0-9]'
- grep -r 'ssl:' server/src/ | grep -q 'rejectUnauthorized'

Accept when:
- All database queries use the shared Pool instance with parameterized statements
- Pool initialization includes DATABASE_URL from environment and SSL configuration
- No direct Client connections are created for request handling

## Enforcement

- Verified by: Code review checking for Pool usage patterns and parameterized queries
- Verified by: Static analysis scanning for SQL injection vulnerabilities and direct Client usage
- Verified by: Integration tests validating connection pooling behavior under concurrent load
- Violation handling: Pull requests introducing direct Client connections or non-parameterized queries are rejected
- Violation handling: Security scanning tools flag SQL injection risks for immediate remediation
- Violation handling: Performance degradation from connection mismanagement triggers incident response
- Exception process: Exceptions for alternative connection patterns require architecture review approval
- Exception process: Documented justification must demonstrate why pooling is inappropriate for the specific use case
- Exception process: Exception approval includes security review for any deviation from parameterized queries