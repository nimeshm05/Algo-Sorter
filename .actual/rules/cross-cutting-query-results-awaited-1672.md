# Use PostgreSQL Connection Pool for Primary Datastore Access: Query Results Awaited

These rules are ALWAYS ACTIVE for all Node.js Express application files that access PostgreSQL databases, including route handlers, middleware, and data access layers.

### Rules

- **R-POOL-001** MUST: Initialize a single Pool instance at application startup with DATABASE_URL from environment variables and SSL configuration before registering Express routes.
- **R-POOL-002** MUST: Use the shared Pool instance for all database queries; do not create direct Client connections for request handling.
- **R-POOL-003** MUST: Use parameterized queries with $1, $2, $3 placeholders and values arrays to prevent SQL injection vulnerabilities.
- **R-POOL-004** SHOULD: Query results SHOULD be awaited using async/await patterns within Express route handlers.
- **R-POOL-005** SHOULD: Wrap all pool.query() calls in try-catch blocks to handle database errors and prevent connection leaks.
- **R-POOL-006** SHOULD: Implement graceful shutdown by calling pool.end() to drain connections before process termination.
- **R-POOL-007** MAY: Configure pool size limits based on expected concurrent request volume and database connection limits.

### Verify

```bash
# Verify Pool instance is created with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify all queries use parameterized statements
grep -r 'pool\.query' server/src/ | grep -q '\$[0-9]'

# Verify SSL configuration is present
grep -r 'ssl:' server/src/ | grep -q 'rejectUnauthorized'
```

**Accept when:**
- All database queries use the shared Pool instance with parameterized statements
- Pool initialization includes DATABASE_URL from environment and SSL configuration
- No direct Client connections are created for request handling
- Query results are awaited in async route handlers
- Error handling is implemented around pool.query() calls

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-POOL rules must be validated before approving code changes that interact with PostgreSQL databases.
</enforcement>