# Use PostgreSQL Connection Pooling with SSL for Primary Datastore Access: Database Connection Strings

These rules are ALWAYS ACTIVE for all Node.js/TypeScript server applications using PostgreSQL, particularly Express.js route handlers performing database queries in cloud-hosted or containerized deployments.

### Rules

- **R-DBCONN-001** MUST: Database connection strings MUST be sourced from process.env.DATABASE_URL at runtime.
- **R-DBCONN-002** MUST: All PostgreSQL connections MUST use Pool instances with connectionString sourced from process.env.DATABASE_URL.
- **R-DBCONN-003** MUST: All database queries MUST use parameterized statements with separate text and values properties to prevent SQL injection.
- **R-DBCONN-004** MUST: SSL configuration MUST be present in Pool initialization with rejectUnauthorized explicitly set.
- **R-DBCONN-005** SHOULD: Pool instances SHOULD be initialized at module level before defining route handlers to ensure single pool reuse.
- **R-DBCONN-006** SHOULD: Pool size limits SHOULD be configured explicitly using max and min parameters based on expected concurrent request volume.
- **R-DBCONN-007** SHOULD: Graceful shutdown handling SHOULD be implemented to close the pool on application termination using pool.end() in process signal handlers.
- **R-DBCONN-008** SHOULD: Startup validation SHOULD verify DATABASE_URL is set and test database connectivity before binding to PORT.

### Verify

```bash
# Verify Pool instances use connectionString from process.env.DATABASE_URL
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'connectionString.*process.env.DATABASE_URL'

# Verify all queries use parameterized statements
grep -r 'pool.query' server/src --include='*.ts' | grep -q 'text:.*values:'

# Verify SSL configuration is present with rejectUnauthorized set
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'ssl.*rejectUnauthorized'
```

**Accept when:**
- All PostgreSQL connections use Pool instances with connectionString sourced from process.env.DATABASE_URL
- All database queries use parameterized statements with separate text and values properties
- SSL configuration is present in Pool initialization with rejectUnauthorized explicitly set
- No direct pg.Client instances are used for database operations
- Pool initialization occurs at module level before route handler definitions

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting code changes affecting database connection patterns.
</enforcement>