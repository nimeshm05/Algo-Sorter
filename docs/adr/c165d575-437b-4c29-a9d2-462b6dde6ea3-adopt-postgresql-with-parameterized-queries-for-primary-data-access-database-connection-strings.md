# Adopt PostgreSQL with Parameterized Queries for Primary Data Access: Database Connection Strings

Status: proposed
Date: 2024-01-15
Deciders: Detection Pipeline (automated)

## Context

- The application requires persistent storage for visitor tracking data including IP addresses, visit counts, and timestamps
- PostgreSQL connection pooling is configured via environment variables with SSL enforcement for secure remote database access
- The system performs three distinct SQL operations: SELECT for aggregation and lookup, INSERT for new visitor records, and UPDATE for incrementing visit counts
- Database credentials and connection strings are managed through environment variables (DATABASE_URL) following twelve-factor app principles

## Problem Statement

The application needs a reliable, secure method for persisting and querying visitor data while preventing SQL injection attacks and managing database connections efficiently across concurrent requests in a Node.js Express environment.

## Decision

1. MUST: Database connection strings MUST be sourced from environment variables, never hardcoded in application code

## Policy Block

- MUST Database connection strings MUST be sourced from environment variables, never hardcoded in application code

## Rationale

- PostgreSQL with the pg library provides ACID compliance and mature connection pooling necessary for concurrent visitor tracking operations
- Parameterized queries prevent SQL injection attacks by separating query structure from user-supplied data (IP addresses, timestamps)
- Environment-based configuration enables deployment flexibility across development, staging, and production environments without code changes
- The detected pattern shows consistent use of pool.query with text/values structure across all data access points, establishing a proven implementation baseline

## Consequences

Positive:
- SQL injection vulnerabilities are eliminated through mandatory parameterized query usage
- Connection pooling reduces database connection overhead and improves performance under concurrent load
- Environment-based configuration supports secure credential management and deployment portability
- Consistent query patterns across SELECT, INSERT, and UPDATE operations simplify maintenance and code review

Negative:
- SSL configuration with rejectUnauthorized: false reduces security by accepting self-signed certificates, creating potential MITM vulnerability
- Direct pool.query usage without abstraction layer couples business logic tightly to PostgreSQL-specific syntax
- No connection retry logic or error handling patterns are evident in the detected code
- Lack of query result type safety may lead to runtime errors when accessing result.rows properties

## Alternatives

- Use an ORM like TypeORM or Prisma for type-safe database access with migration management (rejected)
  Rejected because: Evidence shows direct pg library usage without ORM abstractions; introducing an ORM would require significant refactoring of existing query patterns
  When valid: Valid for greenfield projects or when type safety and schema migration management become critical requirements
- Implement a repository pattern with a dedicated data access layer abstracting PostgreSQL specifics (rejected)
  Rejected because: Current implementation uses direct pool.query calls in route handlers; no abstraction layer is present in the evidence
  When valid: Valid when the application scales beyond visitor tracking and requires database portability or complex query composition
- Use prepared statements with explicit statement names for frequently executed queries (deferred)
  Rejected because: Not rejected but not currently implemented; would optimize repeated query execution
  When valid: Valid optimization when query performance profiling identifies parameterized query parsing as a bottleneck

## Risks

- SSL configuration with rejectUnauthorized: false accepts invalid certificates, enabling potential man-in-the-middle attacks on database connections
  Mitigation: Obtain valid SSL certificates for production database instances and set rejectUnauthorized: true, or use certificate pinning
  Owner: Engineering team and infrastructure team
- No connection pool error handling or retry logic may cause application crashes on transient database connectivity issues
  Mitigation: Implement connection pool error event handlers and query retry logic with exponential backoff
  Owner: Engineering team
- Direct query result property access (rows[0].sum, rows[0].count) without null checks may cause runtime errors on unexpected query results
  Mitigation: Add defensive null checks and result validation before accessing nested properties; consider using optional chaining
  Owner: Engineering team

## Implementation Notes

- Initialize the Pool instance once at application startup and reuse across all route handlers to maintain connection pooling efficiency
- Always structure queries with separate text and values properties when incorporating user-supplied data (req.ip, timestamps)
- Access query results via the rows array property and validate array length before accessing indexed elements
- Store DATABASE_URL in environment-specific configuration files (.env for development) and secure secret management systems for production

## Continuation Context


Verify commands:
- grep -r 'pool\.query' server/src/ | grep -v '\$[0-9]' | grep -v 'SELECT SUM' && echo 'Found unparameterized queries' || echo 'All queries use parameters'
- grep -r 'new Pool' server/src/ | grep -q 'process\.env\.DATABASE_URL' && echo 'Pool uses env config' || echo 'Pool config not from env'
- grep -r 'pool\.query' server/src/ | grep -E '(INSERT|UPDATE)' | grep -q 'values:' && echo 'DML queries use parameterization' || echo 'DML queries missing parameters'

Accept when:
- All grep commands for unparameterized queries return no matches, confirming parameterized query usage
- Pool initialization verification confirms DATABASE_URL is sourced from process.env
- All INSERT and UPDATE statements include values arrays with parameterized placeholders

## Enforcement

- Verified by: Code review checklist requiring parameterized query verification for all database access code
- Verified by: Static analysis with grep-based CI checks scanning for unparameterized query patterns
- Verified by: Security review of SSL configuration and environment variable usage during deployment approval
- Violation handling: CI pipeline fails if unparameterized queries are detected in modified files
- Violation handling: Pull requests containing direct string concatenation in SQL queries are automatically flagged for security review
- Violation handling: Production deployments are blocked if DATABASE_URL is hardcoded rather than environment-sourced
- Exception process: Exceptions for dynamic query construction must be approved by security team with documented SQL injection risk assessment
- Exception process: Alternative SSL configurations require infrastructure team approval with documented certificate validation strategy
- Exception process: All exceptions must be documented in ADR amendments with time-bound review periods