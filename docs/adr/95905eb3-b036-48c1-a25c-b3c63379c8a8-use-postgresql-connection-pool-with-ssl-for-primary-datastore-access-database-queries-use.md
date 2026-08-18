# Use PostgreSQL Connection Pool with SSL for Primary Datastore Access: Database Queries Use

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application requires persistent storage for visitor tracking data with IP addresses, visit counts, and timestamps
- Database credentials are provided via environment variables (DATABASE_URL) requiring secure connection handling
- The service exposes HTTP endpoints that perform synchronous database queries requiring connection management
- The pg library is used as the PostgreSQL client driver, requiring explicit connection pool configuration
- SSL connections with rejectUnauthorized: false indicate connection to a managed database service with self-signed certificates

## Problem Statement

The application needs a reliable, secure method to connect to a PostgreSQL database for visitor tracking operations while managing connection lifecycle, handling SSL requirements for remote database services, and supporting concurrent HTTP requests without connection exhaustion.

## Decision

1. MUST: All database queries MUST use parameterized statements with $1, $2, etc. placeholders to prevent SQL injection

## Policy Block

- MUST All database queries MUST use parameterized statements with $1, $2, etc. placeholders to prevent SQL injection

## Rationale

- The pg Pool provides automatic connection pooling, preventing connection exhaustion under concurrent HTTP request load
- SSL configuration with rejectUnauthorized: false enables connectivity to managed PostgreSQL services (e.g., Heroku Postgres) that use self-signed certificates
- Parameterized queries with $1, $2 placeholders protect against SQL injection while maintaining query readability
- Environment-based configuration (DATABASE_URL) follows twelve-factor app principles for deployment portability

## Consequences

Positive:
- Connection pooling automatically manages connection lifecycle, reducing overhead and preventing connection leaks
- SSL encryption protects data in transit between application and database service
- Parameterized queries eliminate SQL injection vulnerabilities in visitor tracking operations
- Environment variable configuration enables seamless deployment across development, staging, and production environments

Negative:
- Setting rejectUnauthorized: false disables certificate validation, creating vulnerability to man-in-the-middle attacks
- Pool configuration is global and cannot be easily mocked or replaced for testing without environment manipulation
- No explicit connection pool sizing configuration may lead to default limits being insufficient under high load
- Direct pool.query calls throughout route handlers create tight coupling between HTTP layer and data access layer

## Alternatives

- Use direct Client connections instead of Pool for each request (rejected)
  Rejected because: Direct Client connections require manual connection lifecycle management and would create connection exhaustion under concurrent load
  When valid: Only valid for single-threaded batch scripts or CLI tools with sequential operations
- Use an ORM like TypeORM or Prisma for database access (rejected)
  Rejected because: The simple visitor tracking schema with three basic queries does not justify ORM complexity and overhead
  When valid: Valid when schema complexity grows beyond simple CRUD operations or when type-safe query building becomes necessary
- Implement a repository pattern to abstract database access from route handlers (deferred)
  Rejected because: Not rejected but not implemented; would improve testability and separation of concerns
  When valid: Should be adopted when adding additional database tables or when unit testing route handlers becomes necessary

## Risks

- SSL certificate validation is disabled (rejectUnauthorized: false), creating MITM attack surface
  Mitigation: Obtain proper CA certificate bundle for the managed database service and enable full certificate validation, or restrict network access to trusted networks only
  Owner: engineering team
- No connection pool size limits are configured, potentially allowing pool exhaustion or excessive connections
  Mitigation: Add explicit pool configuration with max connection limit based on database service tier and expected concurrent load
  Owner: engineering team
- Database credentials in DATABASE_URL environment variable could be exposed through logs or error messages
  Mitigation: Ensure error handling does not log connection strings and use secret management systems for credential rotation
  Owner: engineering team

## Implementation Notes

- Initialize the Pool instance once at application startup in server/src/index.ts before registering middleware or routes
- Use pool.query({ text: '...', values: [...] }) format for all parameterized queries to maintain consistency
- Consider extracting database access into a separate module (e.g., db.ts) to isolate pool configuration and query functions
- Add connection pool event listeners (pool.on('error', ...)) to handle unexpected connection errors gracefully
- Document the expected DATABASE_URL format (postgresql://user:password@host:port/database) in deployment documentation

## Continuation Context


Verify commands:
- grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'
- grep -r 'pool\.query' server/src/ | grep -q '\$1'
- grep -r 'ssl:' server/src/ | grep -q 'rejectUnauthorized: false'

Accept when:
- Pool initialization with DATABASE_URL and SSL configuration is present in server/src/index.ts
- All database queries use parameterized statements with $1, $2, etc. placeholders
- SSL configuration includes rejectUnauthorized setting for managed database compatibility

## Enforcement

- Verified by: Code review verification of Pool initialization and SSL configuration
- Verified by: Automated grep-based checks in CI pipeline for parameterized query patterns
- Verified by: Security scanning for hardcoded credentials or connection strings
- Violation handling: CI pipeline fails if non-parameterized queries are detected
- Violation handling: Code review blocks merge if Pool configuration deviates from standard
- Violation handling: Security alerts trigger for any hardcoded database credentials
- Exception process: Exceptions require architecture review approval with documented justification
- Exception process: Alternative connection patterns must demonstrate equivalent security and reliability
- Exception process: Temporary exceptions must include remediation timeline and tracking ticket