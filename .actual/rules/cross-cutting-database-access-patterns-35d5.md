# Use PostgreSQL Connection Pooling with SSL for Primary Datastore Access: Database Access Patterns

These rules are ALWAYS ACTIVE for all Node.js/TypeScript server applications using PostgreSQL, particularly Express.js route handlers performing database queries in cloud-hosted or containerized deployments.

### Rules

- **R-PGPOOL-001** SHOULD: Database access patterns (SELECT, INSERT, UPDATE) SHOULD be encapsulated within async request handlers to properly manage connection lifecycle.
- **R-PGPOOL-002** MUST: All PostgreSQL connections MUST use Pool instances with connectionString sourced from process.env.DATABASE_URL.
- **R-PGPOOL-003** MUST: All database queries MUST use parameterized statements with separate text and values properties to prevent SQL injection.
- **R-PGPOOL-004** MUST: SSL configuration MUST be present in Pool initialization with rejectUnauthorized explicitly set.
- **R-PGPOOL-005** SHOULD: Pool size limits SHOULD be configured explicitly using max and min parameters based on expected concurrent request volume.
- **R-PGPOOL-006** SHOULD: Graceful shutdown handling SHOULD be implemented to close the pool on application termination using pool.end() in process signal handlers.
- **R-PGPOOL-007** SHOULD: Startup validation SHOULD verify DATABASE_URL is set and test database connectivity before binding to PORT.

### Verify

```bash
# Verify Pool instantiation with environment-based connection string
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'connectionString.*process.env.DATABASE_URL'

# Verify all queries use parameterized statements
grep -r 'pool.query' server/src --include='*.ts' | grep -q 'text:.*values:'

# Verify SSL configuration is present
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'ssl.*rejectUnauthorized'
```

**Accept when:**
- All PostgreSQL connections use Pool instances with connectionString sourced from process.env.DATABASE_URL
- All database queries use parameterized statements with separate text and values properties
- SSL configuration is present in Pool initialization with rejectUnauthorized explicitly set
- Pool instance is initialized at module level before defining route handlers
- Graceful shutdown handling is implemented for pool.end() on process termination
- Startup validation checks for DATABASE_URL presence and database connectivity

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting code changes. CI pipeline MUST block merges for violations of R-PGPOOL-002, R-PGPOOL-003, and R-PGPOOL-004. Code review checklist MUST explicitly verify connection pooling and SSL configuration.
</enforcement>