# Use PostgreSQL Connection Pool for Database Access: Pool Instance Reused

These rules are ALWAYS ACTIVE for all database access code in the application, particularly route handlers and data access layers that interact with PostgreSQL through the pg library.

### Rules

- **R-POOL-001** MUST: The pool instance MUST be reused across all route handlers rather than creating new connections per request.
- **R-POOL-002** MUST: All database queries MUST use parameterized syntax with $1, $2 placeholders for dynamic values to prevent SQL injection.
- **R-POOL-003** MUST: The Pool instance MUST be initialized once at application startup before registering route handlers.
- **R-POOL-004** SHOULD: Configure explicit maxConnections, connectionTimeoutMillis, and idleTimeoutMillis parameters on the Pool instance to prevent connection exhaustion.
- **R-POOL-005** SHOULD: Implement try-catch blocks around pool.query() calls and add connection error event handlers on the pool instance.
- **R-POOL-006** SHOULD: Call pool.end() during graceful shutdown to close all connections cleanly.

### Verify

```bash
# Verify Pool instance is created with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify all queries use parameterized syntax
grep -r 'pool\.query' server/src/ | grep -q '\$1'

# Verify SSL configuration is present
grep -r 'ssl.*rejectUnauthorized' server/src/ | grep -q 'false'
```

**Accept when:**
- A Pool instance is created with DATABASE_URL from environment variables and SSL configuration
- All database queries use parameterized syntax with $1, $2 placeholders for dynamic values
- The same pool instance is reused across multiple route handlers and query operations
- No direct string concatenation is found in SQL query construction

<enforcement>
Clause Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to database access patterns. Static analysis scanning for direct string concatenation in SQL queries must be performed. Integration tests must verify database connectivity and query execution through the pool.
</enforcement>