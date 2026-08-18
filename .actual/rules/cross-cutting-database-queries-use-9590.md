# Use PostgreSQL Connection Pool with SSL for Primary Datastore Access: Database Queries Use

These rules are ALWAYS ACTIVE for all database query code in the application, particularly in route handlers and data access layers that interact with PostgreSQL via the pg library.

### Rules

- **R-DB-001** MUST: All database queries MUST use parameterized statements with $1, $2, etc. placeholders to prevent SQL injection.
- **R-DB-002** MUST: Pool initialization with DATABASE_URL and SSL configuration (rejectUnauthorized: false) MUST be present in server/src/index.ts.
- **R-DB-003** MUST: All pool.query calls MUST use the { text: '...', values: [...] } format for consistency and security.
- **R-DB-004** SHOULD: Database access SHOULD be extracted into a separate module (e.g., db.ts) to isolate pool configuration and query functions.
- **R-DB-005** SHOULD: Connection pool event listeners (pool.on('error', ...)) SHOULD be added to handle unexpected connection errors gracefully.

### Verify

```bash
# Verify Pool initialization with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify all database queries use parameterized statements
grep -r 'pool\.query' server/src/ | grep -q '\$1'

# Verify SSL configuration includes rejectUnauthorized setting
grep -r 'ssl:' server/src/ | grep -q 'rejectUnauthorized: false'
```

**Accept when:**
- Pool initialization with DATABASE_URL and SSL configuration is present in server/src/index.ts
- All database queries use parameterized statements with $1, $2, etc. placeholders
- SSL configuration includes rejectUnauthorized setting for managed database compatibility
- No hardcoded credentials or connection strings are present in source code
- Error handling does not log connection strings or sensitive database information

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting code that modifies database query patterns or pool initialization. Violations of R-DB-001 and R-DB-002 are blocking and MUST be remediated before merge.
</enforcement>