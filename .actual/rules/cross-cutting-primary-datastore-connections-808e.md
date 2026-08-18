# Use PostgreSQL Connection Pool for Primary Datastore Access: Primary Datastore Connections

These rules are ALWAYS ACTIVE for all Node.js Express application code that accesses the PostgreSQL primary datastore.

### Rules

- **R-POOL-001** MUST: Primary datastore connections MUST use the pg Pool class initialized with connectionString from process.env.DATABASE_URL
- **R-POOL-002** MUST: All database queries MUST use parameterized statements with $1, $2, $3 placeholders and values arrays to prevent SQL injection
- **R-POOL-003** MUST: Pool initialization MUST include SSL configuration from the DATABASE_URL environment variable
- **R-POOL-004** MUST: No direct Client connections are permitted for request handling; all queries MUST use the shared Pool instance
- **R-POOL-005** SHOULD: Pool instance SHOULD be initialized once at application startup before registering Express routes
- **R-POOL-006** SHOULD: All pool.query() calls SHOULD be wrapped in try-catch blocks to handle database errors and prevent connection leaks
- **R-POOL-007** SHOULD: Graceful shutdown SHOULD call pool.end() to drain connections before process termination

### Verify

```bash
# Verify Pool is initialized with DATABASE_URL
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify parameterized queries are used
grep -r 'pool\.query' server/src/ | grep -q '\$[0-9]'

# Verify SSL configuration is present
grep -r 'ssl:' server/src/ | grep -q 'rejectUnauthorized'
```

**Accept when:**
- All database queries use the shared Pool instance with parameterized statements
- Pool initialization includes DATABASE_URL from environment and SSL configuration
- No direct Client connections are created for request handling
- All pool.query() invocations use $1, $2, $3 placeholders with values arrays
- Pool instance is initialized once at application startup

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All pull requests introducing direct Client connections, non-parameterized queries, or deviating from the Pool pattern MUST be rejected.
</enforcement>