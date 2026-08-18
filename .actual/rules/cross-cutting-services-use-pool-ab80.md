# Use PostgreSQL Connection Pool for Primary Datastore Access: Services Use Pool

These rules are ALWAYS ACTIVE for all Node.js Express service code that accesses PostgreSQL databases through the pg library.

### Rules

- **R-POOL-001** MAY: Services MAY use pool.query() with either string queries or query configuration objects containing text and values properties.
- **R-POOL-002** MUST: All database queries use the shared Pool instance with parameterized statements (using $1, $2, $3 placeholders with values arrays).
- **R-POOL-003** MUST: Pool initialization includes DATABASE_URL from environment variables and SSL configuration.
- **R-POOL-004** MUST NOT: Direct Client connections be created for request handling; only the shared Pool instance is permitted.
- **R-POOL-005** MUST: Pool instance be initialized once at application startup before registering Express routes.
- **R-POOL-006** MUST: All pool.query() calls be wrapped in try-catch blocks to handle database errors and prevent connection leaks.
- **R-POOL-007** SHOULD: Implement graceful shutdown by calling pool.end() to drain connections before process termination.
- **R-POOL-008** SHOULD: Configure pool size limits based on expected concurrent request volume and database connection limits.

### Verify

```bash
# Verify Pool instance is created with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify pool.query() uses parameterized queries with placeholders
grep -r 'pool\.query' server/src/ | grep -q '\$[0-9]'

# Verify SSL configuration is present
grep -r 'ssl:' server/src/ | grep -q 'rejectUnauthorized'
```

**Accept when:**
- All database queries use the shared Pool instance with parameterized statements
- Pool initialization includes DATABASE_URL from environment and SSL configuration
- No direct Client connections are created for request handling
- All pool.query() calls are wrapped in error handling
- Pool instance is initialized before route registration

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting code that uses PostgreSQL connections. Pull requests introducing direct Client connections or non-parameterized queries MUST be rejected.
</enforcement>