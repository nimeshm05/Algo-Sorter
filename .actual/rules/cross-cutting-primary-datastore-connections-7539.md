# Use PostgreSQL Connection Pooling with SSL for Primary Datastore Access: Primary Datastore Connections

These rules are ALWAYS ACTIVE for all Node.js/TypeScript server applications using PostgreSQL as the primary datastore, particularly Express.js route handlers performing database queries in cloud-hosted or containerized deployments.

### Rules

- **R-POOL-001** MUST: Primary datastore connections MUST use the pg Pool class for connection pooling rather than direct Client instances.
- **R-POOL-002** MUST: All database queries MUST use parameterized statements with separate text and values properties to prevent SQL injection.
- **R-POOL-003** MUST: Pool initialization MUST source connectionString from process.env.DATABASE_URL environment variable.
- **R-POOL-004** MUST: SSL configuration MUST be present in Pool initialization with rejectUnauthorized explicitly set.
- **R-POOL-005** SHOULD: Pool instance SHOULD be initialized at module level before defining route handlers to ensure single pool reuse.
- **R-POOL-006** SHOULD: Explicit pool size limits SHOULD be configured using max and min parameters based on expected concurrent request volume.
- **R-POOL-007** SHOULD: Graceful shutdown handling SHOULD be implemented to close the pool on application termination using pool.end() in process signal handlers.
- **R-POOL-008** SHOULD: Startup validation SHOULD verify DATABASE_URL is set and test database connectivity before binding to PORT.

### Verify

```bash
# Verify Pool usage with environment variable sourcing
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'connectionString.*process.env.DATABASE_URL'

# Verify parameterized queries with separate text and values
grep -r 'pool.query' server/src --include='*.ts' | grep -q 'text:.*values:'

# Verify SSL configuration in Pool initialization
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'ssl.*rejectUnauthorized'
```

**Accept when:**
- All PostgreSQL connections use Pool instances with connectionString sourced from process.env.DATABASE_URL
- All database queries use parameterized statements with separate text and values properties
- SSL configuration is present in Pool initialization with rejectUnauthorized explicitly set
- Pool instance is initialized at module level before route handler definitions
- Startup validation checks for required environment variables before attempting database connection
- Graceful shutdown handling is implemented for pool.end() on process termination

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-POOL-001 through R-POOL-004 rules are mandatory and must be verified before accepting code changes. R-POOL-005 through R-POOL-008 are strongly recommended best practices.
</enforcement>