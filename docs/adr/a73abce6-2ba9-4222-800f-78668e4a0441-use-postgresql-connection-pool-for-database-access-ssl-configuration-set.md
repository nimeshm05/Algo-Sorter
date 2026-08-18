# Use PostgreSQL Connection Pool for Database Access: Ssl Configuration Set

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application requires persistent storage for visitor tracking data including IP addresses, visit counts, and timestamps
- Database connections are expensive resources that require management to avoid exhaustion and performance degradation
- The application runs in a cloud environment where DATABASE_URL is provided via environment variables with SSL requirements
- Multiple concurrent HTTP requests need to execute SQL queries (SELECT, INSERT, UPDATE) against the same database instance

## Problem Statement

The application needs a reliable, performant mechanism to manage database connections for visitor tracking operations while handling concurrent requests without exhausting connection limits or introducing connection overhead on every query.

## Decision

1. MUST: SSL configuration MUST set rejectUnauthorized to false for the connection pool

## Policy Block

- MUST SSL configuration MUST set rejectUnauthorized to false for the connection pool

## Rationale

- The evidence shows a Pool instance created with DATABASE_URL and SSL configuration, indicating connection pooling is the established pattern for database access
- Multiple query operations (SELECT SUM, SELECT with WHERE, INSERT, UPDATE) are executed through the same pool instance, demonstrating reuse across different operations
- Parameterized queries with $1, $2 placeholders are consistently used across all data access patterns, establishing a security-conscious approach
- The pattern supports the visitor tracking use case which requires aggregate queries, lookups by IP address, and conditional insert/update logic

## Consequences

Positive:
- Connection pooling reduces overhead by reusing established database connections across multiple requests
- Parameterized queries prevent SQL injection vulnerabilities by separating query structure from user-provided data
- SSL configuration enables secure communication with the database in production environments
- The pool abstraction simplifies connection management and automatically handles connection lifecycle

Negative:
- SSL configuration with rejectUnauthorized: false reduces security by accepting self-signed certificates without validation
- Connection pool configuration is not explicitly tuned (no max connections, timeout settings visible), potentially leading to resource exhaustion under high load
- Database credentials are managed through environment variables without evidence of rotation or secrets management integration
- No connection error handling or retry logic is evident in the query patterns, which may lead to unhandled failures

## Alternatives

- Create a new database connection for each HTTP request (rejected)
  Rejected because: Creating connections per request introduces significant overhead and can exhaust database connection limits under concurrent load
  When valid: Only appropriate for very low-traffic applications with infrequent database access
- Use an ORM (Object-Relational Mapper) with built-in connection pooling (rejected)
  Rejected because: The application uses direct SQL queries with the pg library, indicating a preference for explicit query control over ORM abstractions
  When valid: Valid for applications requiring complex object mapping, migrations management, or cross-database compatibility
- Use a single persistent connection without pooling (rejected)
  Rejected because: A single connection cannot handle concurrent requests and creates a bottleneck for parallel query execution
  When valid: Only suitable for single-threaded, sequential processing scenarios

## Risks

- Connection pool exhaustion under high concurrent load due to lack of explicit pool size configuration
  Mitigation: Configure explicit maxConnections, connectionTimeoutMillis, and idleTimeoutMillis parameters on the Pool instance
  Owner: engineering team
- SSL certificate validation is disabled (rejectUnauthorized: false), potentially exposing connections to man-in-the-middle attacks
  Mitigation: Obtain proper SSL certificates for the database and enable full certificate validation, or use certificate pinning
  Owner: security team
- Database connection failures are not explicitly handled, which may result in unhandled promise rejections and application crashes
  Mitigation: Implement try-catch blocks around pool.query() calls and add connection error event handlers on the pool instance
  Owner: engineering team

## Implementation Notes

- Initialize the Pool instance once at application startup before registering route handlers to ensure connection reuse
- Use the pg library's Pool class with connectionString, ssl, max, idleTimeoutMillis, and connectionTimeoutMillis configuration options
- Structure parameterized queries with the {text: string, values: array} format to separate SQL structure from dynamic values
- Implement graceful shutdown by calling pool.end() when the application terminates to close all connections cleanly

## Continuation Context


Verify commands:
- grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'
- grep -r 'pool\.query' server/src/ | grep -q '\$1'
- grep -r 'ssl.*rejectUnauthorized' server/src/ | grep -q 'false'

Accept when:
- A Pool instance is created with DATABASE_URL from environment variables and SSL configuration
- All database queries use parameterized syntax with $1, $2 placeholders for dynamic values
- The same pool instance is reused across multiple route handlers and query operations

## Enforcement

- Verified by: Code review checking for Pool instantiation pattern and parameterized query usage
- Verified by: Static analysis scanning for direct string concatenation in SQL queries
- Verified by: Integration tests verifying database connectivity and query execution through the pool
- Violation handling: Pull requests introducing non-parameterized queries are rejected during code review
- Violation handling: Static analysis failures block CI pipeline progression
- Violation handling: Runtime connection errors are logged and monitored for investigation
- Exception process: Exceptions require architectural review and documentation of security implications
- Exception process: Alternative connection patterns must demonstrate equivalent or superior connection management
- Exception process: Approved exceptions are documented in ADR amendments with time-bound review periods