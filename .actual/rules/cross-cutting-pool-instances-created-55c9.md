# Use PostgreSQL Connection Pooling with SSL for Primary Datastore Access: Pool Instances Created

These rules are ALWAYS ACTIVE for all Node.js/TypeScript server applications using PostgreSQL for persistent storage, particularly Express.js route handlers performing database queries in cloud-hosted or containerized deployments.

### Rules

- **R-POOL-001** MUST: Pool instances MUST be created once at application startup and reused across all request handlers.
- **R-POOL-002** MUST: All PostgreSQL connections MUST use Pool instances with connectionString sourced from process.env.DATABASE_URL.
- **R-POOL-003** MUST: All database queries MUST use parameterized statements with separate text and values properties to prevent SQL injection.
- **R-POOL-004** MUST: SSL configuration MUST be present in Pool initialization with rejectUnauthorized explicitly set.
- **R-POOL-005** SHOULD: Pool size limits SHOULD be configured explicitly using max and min parameters based on expected concurrent request volume and database connection limits.
- **R-POOL-006** SHOULD: Graceful shutdown handling SHOULD be implemented to close the pool on application termination using pool.end() in process signal handlers.
- **R-POOL-007** SHOULD: Startup validation SHOULD verify DATABASE_URL is set and test database connectivity before binding to PORT to fail fast on misconfiguration.

### Verify

```bash
# Verify Pool instances are created with environment-based connection strings
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'connectionString.*process.env.DATABASE_URL'

# Verify all database queries use parameterized statements
grep -r 'pool.query' server/src --include='*.ts' | grep -q 'text:.*values:'

# Verify SSL configuration is present in Pool initialization
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'ssl.*rejectUnauthorized'
```

**Accept when:**
- All PostgreSQL connections use Pool instances with connectionString sourced from process.env.DATABASE_URL
- All database queries use parameterized statements with separate text and values properties
- SSL configuration is present in Pool initialization with rejectUnauthorized explicitly set
- Pool instance is initialized at module level before defining route handlers to ensure single pool reuse
- No direct pg.Client instances are used for request handling

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting code changes. Violations block CI pipeline merge and require code review checklist sign-off.
</enforcement>