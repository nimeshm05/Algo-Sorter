# Use PostgreSQL Connection Pool for Primary Datastore Access: Postgresql Connections Enforce

These rules are ALWAYS ACTIVE for all Node.js/Express application code that accesses PostgreSQL databases, particularly in `server/src/` and related datastore integration points.

### Rules

- **R-PGPOOL-001** MUST: PostgreSQL connections MUST enforce SSL with rejectUnauthorized: false for cloud-hosted databases.
- **R-PGPOOL-002** MUST: All database queries MUST use the shared Pool instance with parameterized statements (using $1, $2, $3 placeholders).
- **R-PGPOOL-003** MUST: Pool initialization MUST include DATABASE_URL from environment variables and SSL configuration.
- **R-PGPOOL-004** MUST: No direct Client connections are permitted for request handling; only the shared Pool instance is allowed.
- **R-PGPOOL-005** SHOULD: Implement graceful shutdown by calling pool.end() to drain connections before process termination.
- **R-PGPOOL-006** SHOULD: Use try-catch blocks around all pool.query() calls to handle database errors and prevent connection leaks.

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
- Connection pool is initialized once at application startup before registering Express routes
- Graceful shutdown includes pool.end() call to drain connections

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting code changes that modify database connection patterns.
</enforcement>