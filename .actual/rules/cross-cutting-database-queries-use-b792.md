# Use PostgreSQL Connection Pool for Primary Datastore Access: Database Queries Use

These rules are ALWAYS ACTIVE for all database query code in Node.js/Express application files that interact with PostgreSQL through the pg library.

### Rules

- **R-DB-001** MUST: Database queries MUST use parameterized statements with $1, $2 placeholders and values arrays to prevent SQL injection.
- **R-DB-002** MUST: All database connections MUST use a shared Pool instance initialized with DATABASE_URL from environment variables.
- **R-DB-003** MUST: Pool initialization MUST include SSL configuration with appropriate certificate validation settings.
- **R-DB-004** MUST: All pool.query() calls MUST be wrapped in try-catch blocks to handle database errors and prevent connection leaks.
- **R-DB-005** SHOULD: Connection pool size limits SHOULD be configured based on expected concurrent request volume and database connection limits.
- **R-DB-006** SHOULD: Graceful shutdown SHOULD call pool.end() to drain connections before process termination.

### Verify

```bash
# Verify Pool instance is created with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify all pool.query() calls use parameterized statements
grep -r 'pool\.query' server/src/ | grep -q '\$[0-9]'

# Verify SSL configuration is present
grep -r 'ssl:' server/src/ | grep -q 'rejectUnauthorized'
```

**Accept when:**
- All database queries use the shared Pool instance with parameterized statements ($1, $2, $3 placeholders with values arrays)
- Pool initialization includes DATABASE_URL from environment and SSL configuration
- No direct Client connections are created for request handling
- All pool.query() invocations are wrapped in error handling
- Connection pooling is the centralized data access mechanism across all route handlers

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting code changes that interact with the PostgreSQL database.
</enforcement>