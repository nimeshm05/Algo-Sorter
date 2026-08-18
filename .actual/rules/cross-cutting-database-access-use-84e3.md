# Use PostgreSQL Connection Pool for Database Access: Database Access Use

These rules are ALWAYS ACTIVE for all database access code in the application, particularly files implementing visitor tracking data persistence and SQL query execution.

### Rules

- **R-DB-001** MUST: Database access MUST use a connection pool initialized with connectionString from process.env.DATABASE_URL
- **R-DB-002** MUST: All database queries MUST use parameterized syntax with $1, $2 placeholders for dynamic values to prevent SQL injection
- **R-DB-003** MUST: The same pool instance MUST be reused across multiple route handlers and query operations
- **R-DB-004** SHOULD: Pool instance SHOULD be initialized once at application startup before registering route handlers
- **R-DB-005** SHOULD: Graceful shutdown SHOULD call pool.end() when the application terminates to close all connections cleanly
- **R-DB-006** SHOULD: Pool configuration SHOULD include explicit maxConnections, connectionTimeoutMillis, and idleTimeoutMillis parameters
- **R-DB-007** SHOULD: Database connection failures SHOULD be explicitly handled with try-catch blocks and connection error event handlers

### Verify

```bash
# Verify Pool instance is created with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify all database queries use parameterized syntax
grep -r 'pool\.query' server/src/ | grep -q '\$1'

# Verify SSL configuration is present
grep -r 'ssl.*rejectUnauthorized' server/src/ | grep -q 'false'

# Verify no direct string concatenation in SQL queries
! grep -r "pool\.query.*+" server/src/ | grep -v '\$[0-9]'
```

**Accept when:**
- A Pool instance is created with DATABASE_URL from environment variables and SSL configuration
- All database queries use parameterized syntax with $1, $2 placeholders for dynamic values
- The same pool instance is reused across multiple route handlers and query operations
- No direct string concatenation is used in SQL query construction

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-DB rules marked MUST are non-negotiable; SHOULD rules represent best practices that should be followed unless explicitly documented exceptions are approved through architectural review.
</enforcement>