# Use PostgreSQL Connection Pool for Primary Datastore Access: Services Execute Select

These rules are ALWAYS ACTIVE for all Node.js Express application code that accesses PostgreSQL databases for visitor tracking data and HTTP endpoint operations.

### Rules

- **R-POOL-001** SHOULD: Services SHOULD execute SELECT, INSERT, and UPDATE operations through the shared pool instance rather than creating new connections.
- **R-POOL-002** MUST: Pool initialization MUST include DATABASE_URL from environment variables and SSL configuration.
- **R-POOL-003** MUST: All database queries MUST use parameterized statements with $1, $2, $3 placeholders and values arrays to prevent SQL injection.
- **R-POOL-004** MUST: No direct Client connections SHALL be created for request handling; all datastore access MUST route through the shared Pool instance.
- **R-POOL-005** SHOULD: Applications SHOULD implement graceful shutdown by calling pool.end() to drain connections before process termination.
- **R-POOL-006** SHOULD: Try-catch blocks SHOULD wrap all pool.query() calls to handle database errors and prevent connection leaks.

### Verify

```bash
# Verify Pool instance is created with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify parameterized queries are used consistently
grep -r 'pool\.query' server/src/ | grep -q '\$[0-9]'

# Verify SSL configuration is present
grep -r 'ssl:' server/src/ | grep -q 'rejectUnauthorized'
```

**Accept when:**
- All database queries use the shared Pool instance with parameterized statements
- Pool initialization includes DATABASE_URL from environment and SSL configuration
- No direct Client connections are created for request handling
- Connection pool monitoring and error handling are implemented in async route handlers

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting code changes that introduce or modify database access patterns.
</enforcement>