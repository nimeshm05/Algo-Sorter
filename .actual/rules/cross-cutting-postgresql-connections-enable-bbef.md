# Use PostgreSQL Connection Pooling with SSL for Primary Datastore Access: Postgresql Connections Enable

These rules are ALWAYS ACTIVE for all PostgreSQL database connections in Node.js/TypeScript server applications, including visitor tracking and stateful data persistence operations, Express.js route handlers performing database queries, and cloud-hosted or containerized deployments requiring SSL database connections.

### Rules

- **R-PGPOOL-001** MUST: PostgreSQL connections MUST enable SSL with rejectUnauthorized set to false for cloud-hosted database compatibility.
- **R-PGPOOL-002** MUST: All PostgreSQL connections use Pool instances with connectionString sourced from process.env.DATABASE_URL.
- **R-PGPOOL-003** MUST: All database queries use parameterized statements with separate text and values properties to prevent SQL injection.
- **R-PGPOOL-004** SHOULD: Initialize the Pool instance at module level before defining route handlers to ensure single pool reuse.
- **R-PGPOOL-005** SHOULD: Configure pool size limits explicitly using max and min parameters based on expected concurrent request volume and database connection limits.
- **R-PGPOOL-006** SHOULD: Implement graceful shutdown handling to close the pool on application termination using pool.end() in process signal handlers.
- **R-PGPOOL-007** SHOULD: Add startup validation to verify DATABASE_URL is set and test database connectivity before binding to PORT to fail fast on misconfiguration.

### Verify

```bash
# Verify Pool instances use connectionString from process.env.DATABASE_URL
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'connectionString.*process.env.DATABASE_URL'

# Verify all database queries use parameterized statements
grep -r 'pool.query' server/src --include='*.ts' | grep -q 'text:.*values:'

# Verify SSL configuration is present with rejectUnauthorized setting
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'ssl.*rejectUnauthorized'
```

**Accept when:**
- All PostgreSQL connections use Pool instances with connectionString sourced from process.env.DATABASE_URL
- All database queries use parameterized statements with separate text and values properties
- SSL configuration is present in Pool initialization with rejectUnauthorized explicitly set
- No direct pg.Client instances are used for database operations
- Pool instance is initialized at module level before route handlers are defined

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to PostgreSQL connection patterns. Violations block merge and require security team review for any SSL configuration changes.
</enforcement>